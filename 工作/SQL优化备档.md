PostgreSQL 宽表 COUNT(*) 慢查询优化实战

一、背景

业务中有两张表 A 和 B，它们是 1:N 的关系。查询条件需要同时用到 A 和 B 的字段，因此原方案是先 JOIN 再 WHERE 过滤。但 JOIN 先行意味着全表关联，资源开销巨大。

为此，我们采用了冗余字段的方案——将 B 表的查询字段冗余到 A 表（即 risk 表），从而避免 JOIN。

冗余之后，JOIN 的问题解决了，但新问题出现了：一条简单的 SELECT count(*) 带条件查询，竟然跑了 76 秒。

二、问题现象

risk 表有 50 个字段，执行以下查询：

    SELECT count(*) FROM risk
    WHERE ref_account_id = '6745393f14a0673311d11d39'
      AND happen_time >= '2025-11-23'
      AND happen_time < '2025-12-23';

执行计划关键信息：

  指标                  	数值                     
  命中数据行数              	~248 万行                
  执行耗时                	73.8 秒                 
  磁盘物理读取 (shared read)	622,250 Pages (~4.7 GB)
  扫描方式                	Index Only Scan        
  Heap Fetches（回表次数）  	2,485,841              

看起来用了 Index Only Scan，应该很快才对，为什么还是慢？

三、根因分析：Index Only Scan 陷阱

3.1 什么是 Index Only Scan？

正常情况下，Index Only Scan 是 PostgreSQL 最高效的扫描方式之一——它只读索引，不碰表数据（Heap），所以速度极快。

但它有一个前提：数据库必须能通过 Visibility Map (VM) 确认索引指向的行是否"可见"。如果 VM 没有覆盖到对应的数据页，数据库就不得不回表（Heap Fetch）去逐行确认 MVCC 可见性。

3.2 本案中发生了什么？

三个因素叠加，导致了灾难性的性能表现：

1) Visibility Map 失效 → 大量回表

由于该表近期有大量写入操作，且 VACUUM 频率不足，导致 VM 文件未能及时更新。结果是：扫描 248 万行索引，每一行都触发了回表（Heap Fetches ≈ 扫描行数）。名义上是 "Index Only Scan"，实际上退化成了 "Index Scan + 全量回表"。

2) 宽表导致 IO 放大

risk 表有 50 个字段，单行物理尺寸很大，一个 8KB 的 Page 能存的行数极少。为确认 248 万行的可见性，数据库被迫读取了 62 万+ 个数据页，共 4.7 GB 数据。

3) 随机 IO 拖慢整体速度

回表读取的是分散在磁盘各处的数据页，属于低密度随机 IO，这是机械磁盘和 SSD 都不擅长的场景。

一句话总结：VACUUM 不及时 → VM 失效 → Index Only Scan 退化为全量回表 → 宽表放大 IO → 76 秒。

四、优化方案

4.1 立即生效：手动 VACUUM

对全表执行一次 VACUUM，修复 Visibility Map：

    -- 建议在低峰期执行
    -- 限速参数，避免瞬间打满磁盘 IO
    SET vacuum_cost_delay = 10;
    VACUUM ANALYZE risk;

说明：

- 标准 VACUUM（非 VACUUM FULL）只申请 SHARE UPDATE EXCLUSIVE 锁，不阻塞业务读写。
- 在 100G+ 的表上执行会产生持续磁盘读取压力，测试环境（21G）实际耗时约 7 分钟。
- 生产环境预估 30 分钟 ~ 1 小时，取决于磁盘 IO 能力。

4.2 长期有效：调低 autovacuum 触发阈值

PostgreSQL 默认的 autovacuum 触发阈值是表数据变动 5%。对于一张数据量巨大的宽表，5% 意味着需要积累非常多的变更才会触发自动清理，VM 很容易过期。

将阈值调低到 1%，让 autovacuum 更积极地维护 VM：

    ALTER TABLE risk SET (
      autovacuum_vacuum_scale_factor = 0.01
    );

五、优化效果预估

  指标          	优化前           	优化后（预估）     
  Heap Fetches	2,485,841     	→ 0         
  读取数据量       	~4.7 GB（堆表+索引）	~150 MB（仅索引）
  查询耗时        	76 秒          	< 1 秒       

原理：VM 修复后，Index Only Scan 无需回表，只扫描索引页即可完成计数。

六、实际验收结果

VACUUM 执行后，对相同查询进行验证：

查询 1：10月数据（~480 万行）

  指标          	数值           
  执行耗时        	4.5 秒        
  Heap Fetches	133,164（大幅下降）
  磁盘读取        	43,682 Pages 
  并行执行        	2 Workers    

查询 2：11月数据（~248 万行）

  指标          	数值          
  执行耗时        	1.6 秒       
  Heap Fetches	22,052（大幅下降）
  磁盘读取        	9,658 Pages 
  并行执行        	2 Workers   

对比优化前的 76 秒，11 月数据查询耗时降至 1.6 秒，提升约 47 倍。

Heap Fetches 未降至 0，说明 VACUUM 期间仍有新的写入产生了未标记的页面，这属于正常现象。配合调低后的 autovacuum 阈值，后续会持续改善。

七、总结

  阶段  	要点                                      
  问题根因	VACUUM 不及时导致 Visibility Map 失效，Index Only Scan 退化为全量回表
  优化手段	手动 VACUUM + 调低 autovacuum 阈值            
  效果  	76 秒 → 1.6 秒，IO 从 4.7 GB 降至 ~75 MB      
  注意事项	低峰期执行 VACUUM，设置限速参数，监控磁盘 IOPS           

