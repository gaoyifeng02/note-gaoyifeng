
这份报告针对 `risk` 表（50个字段）在执行 `count(*)` 时出现的严重性能延迟（76秒）进行分析，提出解决方案，最终验收结果的记录。

### 现状描述 

```python
EXPLAIN (ANALYZE, BUFFERS) 
SELECT count(*) FROM risk 
WHERE ref_account_id = '6745393f14a0673311d11d39' 
  AND happen_time >= '2025-11-23' 
  AND happen_time < '2025-12-23';

Aggregate  (cost=3238030.07..3238030.08 rows=1 width=8) (actual time=73853.319..73853.320 rows=1 loops=1)
   Buffers: shared hit=879847 read=622250 dirtied=114 written=12390
   ->  Index Only Scan using ix_risk_account_happen on risk  (cost=0.57..3229783.33 rows=3298697 width=0) (actual time=0.626..73415.866 rows=2480978 loops=1)
         Index Cond: ((ref_account_id = '6745393f14a0673311d11d39'::text) AND (happen_time >= '2025-11-23 00:00:00'::timestamp without time zone) AND (happen_time < '2025-12-23 00:00:00'::timest
amp without time zone))
         Heap Fetches: 2485841
         Buffers: shared hit=879847 read=622250 dirtied=114 written=12390
 Planning Time: 0.169 ms
 Execution Time: 73853.362 ms
(8 rows)

Time: 73854.254 ms (01:13.854)
```

- **查询语句**：带条件的 `SELECT count(*)` 统计，涉及数据量约 248 万行。
- **性能表现**：执行耗时 **73.8 秒**。
- **资源消耗**：产生约 **622,250** 次磁盘物理读取 (shared read)，折算 IO 吞吐约 **4.7 GB**。

### 根因分析

根据 `EXPLAIN ANALYZE` 提供的真实数据，性能瓶颈并非缺失索引，而是触发了 **“Index Only Scan 陷阱”** ：

1. **无效的“只读索引扫描”** ： 执行计划虽然使用了 `Index Only Scan`，但 **​`Heap Fetches`​** **数值高达 2,485,841**。这意味着数据库每扫描一行索引，都不得不回表（Heap）查询原始行来确认 MVCC 可见性。
2. **MVCC 与 Visibility Map 失效**： 由于该表最近可能存在大量写入或 `VACUUM` 频率不足，导致 **Visibility Map (VM)** 未能覆盖目标数据页。数据库无法通过索引直接判断数据生死，强制触发回表。
3. **宽表带来的 IO 放大效应**： 由于表中有 **50 个字段**，单行物理尺寸巨大，一个 8KB 的 Page 存储行数极少。为了确认 248 万行数据的可见性，数据库被迫读取了超过 62 万个数据页（4.7 GB 数据）。这种**低密度的随机 IO** 是耗时 76 秒的根本原因。

### 性能预估 

- **优化后目标**：将 `Heap Fetches` 降至 **0**。
    
- **IO 降幅预测**：
    
    - **当前**：读取堆表 + 索引 ≈ 622,250 Pages (4.7 GB)。
    - **优化后**：仅读取索引页 ≈ 20,000 Pages (约 150 MB)。
- **预期结果**：查询延迟将从 **76 秒降至 1 秒以内**（实现真正的纯内存/轻量 IO 扫描）。
    

---

### 变更执行方案

#### 手动触发清理

对全表进行一次清理和统计信息更新，以修复 Visibility Map。

```sql
-- 建议在低峰期执行，不锁读写，但会占用一定 I/O
-- 增加成本限制延迟，避免瞬间 IO 抽干
SET vacuum_cost_delay = 10;
VACUUM ANALYZE risk;
```

#### 步骤二：调整自动清理策略（长期有效）

针对 50 字段宽表且数据量大的特性，默认的 5% 触发阈值过于迟钝。建议将该表的 `autovacuum` 触发阈值调低至 1%，以保持 VM 文件的实时性。

```sql
ALTER TABLE risk SET (
  autovacuum_vacuum_scale_factor = 0.01,    -- 变动 1% 即触发清理
);
```

---

### 风险评估 

1. **锁竞争**：本次变更使用标准 `VACUUM`（非 `VACUUM FULL`），仅申请 `SHARE UPDATE EXCLUSIVE` 锁，**不阻塞业务的 INSERT/UPDATE/DELETE 和查询**。
    
2. **IO 负载**：在 100G+ 的表上跑 `VACUUM` 会产生持续的磁盘读取压力。（测试环境21G，实际执行时间为7m17s）
    
    - **对策**：通过 `SET vacuum_cost_delay` 限速，并由 SRE 监控磁盘 IOPS 使用率。
3. **执行时间**：预估全表 `VACUUM` 耗时在 30 分钟至 1 小时之间（取决于 IO 能力）。
    

### 6. 执行后结果


```python
dla=# EXPLAIN (ANALYZE, BUFFERS) 
dla-# SELECT count(*) FROM risk
dla-# WHERE ref_account_id = '6745393f14a0673311d11d39'
dla-#   AND happen_time >= '2025-10-23'
dla-#   AND happen_time < '2025-11-23';
                                                                                                              QUERY PLAN                                                                        
                                  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------
 Finalize Aggregate  (cost=176784.93..176784.94 rows=1 width=8) (actual time=4496.741..4521.042 rows=1 loops=1)
   Buffers: shared hit=282131 read=43682 dirtied=134 written=717
   ->  Gather  (cost=176784.72..176784.93 rows=2 width=8) (actual time=4492.624..4521.021 rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         Buffers: shared hit=282131 read=43682 dirtied=134 written=717
         ->  Partial Aggregate  (cost=175784.72..175784.73 rows=1 width=8) (actual time=4487.008..4487.010 rows=1 loops=3)
               Buffers: shared hit=282131 read=43682 dirtied=134 written=717
               ->  Parallel Index Only Scan using ix_risk_account_happen on risk  (cost=0.57..170790.08 rows=1997857 width=0) (actual time=0.590..4361.820 rows=1602420 loops=3)
                     Index Cond: ((ref_account_id = '6745393f14a0673311d11d39'::text) AND (happen_time >= '2025-10-23 00:00:00'::timestamp without time zone) AND (happen_time < '2025-11-23 00:00
:00'::timestamp without time zone))
                     Heap Fetches: 133164
                     Buffers: shared hit=282131 read=43682 dirtied=134 written=717
 Planning:
   Buffers: shared hit=157
 Planning Time: 0.992 ms
 Execution Time: 4521.092 ms
(16 rows)

dla=# EXPLAIN (ANALYZE, BUFFERS) 
dla-# SELECT count(*) FROM risk
dla-# WHERE ref_account_id = '6745393f14a0673311d11d39'
dla-#   AND happen_time >= '2025-11-23'
dla-#   AND happen_time < '2025-12-23';
                                                                                                              QUERY PLAN                                                                        
                                  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------
 Finalize Aggregate  (cost=128414.86..128414.87 rows=1 width=8) (actual time=1558.095..1567.072 rows=1 loops=1)
   Buffers: shared hit=110396 read=9658 dirtied=71
   ->  Gather  (cost=128414.65..128414.86 rows=2 width=8) (actual time=1558.017..1567.049 rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         Buffers: shared hit=110396 read=9658 dirtied=71
         ->  Partial Aggregate  (cost=127414.65..127414.66 rows=1 width=8) (actual time=1553.989..1553.990 rows=1 loops=3)
               Buffers: shared hit=110396 read=9658 dirtied=71
               ->  Parallel Index Only Scan using ix_risk_account_happen on risk  (cost=0.57..123794.48 rows=1448066 width=0) (actual time=0.408..1492.030 rows=828233 loops=3)
                     Index Cond: ((ref_account_id = '6745393f14a0673311d11d39'::text) AND (happen_time >= '2025-11-23 00:00:00'::timestamp without time zone) AND (happen_time < '2025-12-23 00:00
:00'::timestamp without time zone))
                     Heap Fetches: 22052
                     Buffers: shared hit=110396 read=9658 dirtied=71
 Planning Time: 0.195 ms
 Execution Time: 1567.144 ms
(14 rows)
```