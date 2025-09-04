
~~~
docker pull apache/rocketmq:5.1.0
~~~

~~~
# 日志目录
mkdir /docker/rocketmq/nameserver/logs -p
# 脚本目录
mkdir /docker/rocketmq/nameserver/bin -p
~~~

~~~
docker run -d \
--privileged=true \
--name rmqnamesrv \
apache/rocketmq:5.1.0 sh mqnamesrv

~~~

~~~
docker cp rmqnamesrv:/home/rocketmq/rocketmq-5.1.0/bin/runserver.sh /docker/rocketmq/nameserver/bin/runserver.sh
~~~

找到调用calculate_heap_sizes函数的位置注释掉保存

~~~
docker stop rmqnamesrv
docker rm rmqnamesrv
~~~

~~~
docker run -d \
--privileged=true \
--restart=always \
--name rocketmq-nameserver-alone \
-p 9876:9876  \
-v /docker/rocketmq/nameserver/logs:/home/rocketmq/logs \
-v /docker/rocketmq/nameserver/bin/runserver.sh:/home/rocketmq/rocketmq-5.1.0/bin/runserver.sh \
-e "MAX_HEAP_SIZE=1024M" \
-e "HEAP_NEWSIZE=512M" \
apache/rocketmq:5.1.0 sh mqnamesrv
~~~

~~~
# 创建需要的挂载目录
mkdir /docker/rocketmq/broker/logs -p \
mkdir /docker/rocketmq/broker/data -p \
mkdir /docker/rocketmq/broker/conf -p \
mkdir /docker/rocketmq/broker/bin -p
~~~

~~~
# nameServer 地址多个用;隔开 默认值null
# 例：127.0.0.1:6666;127.0.0.1:8888 
namesrvAddr = 117.72.91.125:9876
# 集群名称
brokerClusterName = DefaultCluster
# 节点名称
brokerName = broker-a
# broker id节点ID， 0 表示 master, 其他的正整数表示 slave，不能小于0 
brokerId = 0
# Broker服务地址	String	内部使用填内网ip，如果是需要给外部使用填公网ip
brokerIP1 = 117.72.91.125
# Broker角色
brokerRole = ASYNC_MASTER
# 刷盘方式
flushDiskType = ASYNC_FLUSH
# 在每天的什么时间删除已经超过文件保留时间的 commit log，默认值04
deleteWhen = 04
# 以小时计算的文件保留时间 默认值72小时
fileReservedTime = 72
# 是否允许Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
# 是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
~~~

~~~
docker run -d \
--name rmqbroker \
--privileged=true \
apache/rocketmq:5.1.0 \
sh mqbroker
~~~

~~~
docker cp rmqbroker:/home/rocketmq/rocketmq-5.1.0/bin/runbroker.sh /docker/rocketmq/broker/bin/runbroker.sh
~~~

~~~
docker stop rmqbroker
docker rm rmqbroker
~~~

~~~
docker run -d \
--restart=always \
--name rocketmq-broker-alone \
-p 10911:10911 -p 10909:10909 \
--privileged=true \
-v /docker/rocketmq/broker/logs:/root/logs \
-v /docker/rocketmq/broker/store:/root/store \
-v /docker/rocketmq/broker/conf/broker.conf:/home/rocketmq/broker.conf \
-v /docker/rocketmq/broker/bin/runbroker.sh:/home/rocketmq/rocketmq-5.1.0/bin/runbroker.sh \
-e "MAX_HEAP_SIZE=2048M" \
-e "HEAP_NEWSIZE=1024M" \
apache/rocketmq:5.1.0 \
sh mqbroker -c /home/rocketmq/broker.conf
~~~

完整broker配置：
~~~
# nameServer 地址多个用;隔开 默认值null
# 例：127.0.0.1:6666;127.0.0.1:8888 
namesrvAddr = 127.0.0.1:6666
# 集群名称 单机配置可以随意填写，如果是集群部署在同一个集群中集群名称必须一致类似Nacos的命名空间
brokerClusterName = DefaultCluster
# broker节点名称 单机配置可以随意填写，如果是集群部署在同一个集群中节点名称不要重复
brokerName = broker-a
# broker id节点ID， 0 表示 master, 其他的正整数表示 slave，不能小于0 
brokerId = 0
# Broker 对外服务的监听端口 默认值10911
# 端口（注意：broker启动后，会占用3个端口，分别在listenPort基础上-2，+1，供内部程序使用，所以集群一定要规划好端口，避免冲突）
listenPort=10911
# Broker服务地址	String	内部使用填内网ip，如果是需要给外部使用填公网ip
brokerIP1 = 127.0.0.1
# BrokerHAIP地址，供slave同步消息的地址 内部使用填内网ip，如果是需要给外部使用填公网ip
brokerIP2 = 127.0.0.1

# Broker角色 默认值ASYNC_MASTER
# ASYNC_MASTER 异步复制Master，只要主写成功就会响应客户端成功，如果主宕机可能会出现小部分数据丢失
# SYNC_MASTER 同步双写Master，主和从节点都要写成功才会响应客户端成功，主宕机也不会出现数据丢失
# SLAVE
brokerRole = ASYNC_MASTER
# 刷盘方式
# SYNC_FLUSH（同步刷新）相比于ASYNC_FLUSH（异步处理）会损失很多性能，但是也更可靠，所以需要根据实际的业务场景做好权衡，默认值ASYNC_FLUSH
flushDiskType = ASYNC_FLUSH
# 在每天的什么时间删除已经超过文件保留时间的 commit log，默认值04
deleteWhen = 04
# 以小时计算的文件保留时间 默认值72小时
fileReservedTime = 72

# 消息大小 单位字节 默认1024 * 1024 * 4
maxMessageSize=4194304

# 在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数，默认值4
defaultTopicQueueNums=4
# 是否允许Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
# 是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

# 失败重试时间，默认重试16次进入死信队列，第一次1s第二次5s以此类推。
# 延时队列时间等级默认18个，可以设置多个比如在后面添加一个1d(一天)，使用的时候直接用对应时间等级即可,从1开始到18，如果添加了第19个直接使用等级19即可
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

# 指定TM在20秒内应将最终确认状态发送给TC，否则引发消息回查。默认为60秒
transactionTimeout=20
# 指定最多回查5次，超过后将丢弃消息并记录错误日志。默认15次。
transactionCheckMax=5
# 指定设置的多次消息回查的时间间隔为10秒。默认为60秒。
transactionCheckInterval=10

~~~









