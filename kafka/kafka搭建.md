### 1、安装kafka

版本：kafka_2.13-2.5.0.tgz

[下载链接](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.5.0/kafka_2.13-2.5.0.tgz)

注：kafka 运行需要 jdk 的支持，如果jdk版本小于1.8 先安装jdk1.8，并修改JAVA_HOME 环境变量。

下载 kafka_2.13-2.5.0.tgz 到 /home/work/kafka 目录下

```bash
mkdir /home/work/kafka
  
mkdir /home/work/kafka/data /home/work/kafka/logs
  
## myid文件的作用是标示 kafka节点的 broker.id，节点 1 就是 1，节点 2 就是 2
echo 1 > /home/work/kafka/data/myid
  
cd /home/work/kafka
  
## 在 /home/kafka 目录下解压 kafka
tar -zxf  kafka_2.13-2.5.0.tgz
```



### 2、自定义 kafka 堆内存

在 kafka 的 bin 目录下的 kafka-server-start.sh 文件，修改以下内容

 

```bash
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx5G -Xms5G"
fi
```

将堆内存修改到 5G



### 3、zookeeper和kafka配置文件

kafka 节点需要 zookeeper 来管理，如果没有 zookeeper 集群的话，可以使用 kafka 自带打包和配置好的Zookeeper。

**1、zookeeper.properties 配置文件**

``` bash
dataDir=/home/work/kafka/data
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
# maxClientCnxns=100
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
# admin.enableServer=false
# admin.serverPort=8080
 
tickTime=2000
initLimit=5
syncLimit=2
 
# cluster mode
server.1=ip:2888:3888
server.2=ip:2888:3888
server.3=ip:2888:3888
```



cluster mode配置端口解释

第一个端口是master和slave之间的通信端口，默认是2888，第二个端口是leader选举的端口，集群刚启动的时候选举或者leader挂掉之后进行新的选举的端口默认是3888

 

**2、server.properties**

```
broker.id=1
host.name=10.20.0.22
 
port=9092
  
############################# Socket Server Settings #############################
 
 
num.network.threads=3
  
# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8
  
# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400
  
# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400
  
# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600
  
  
############################# Log Basics #############################
  
# A comma separated list of directories under which to store log files
log.dirs=/home/work/kafka/logs/
  
#  默认创建 topic 时的分区数
 
num.partitions=3
  
# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1
  
############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
  
############################# Log Flush Policy #############################
  
# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.
  
# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000
  
# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000
  
############################# Log Retention Policy #############################
  
# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.
  
# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168
  
# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824
  
# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824
  
# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000
  
############################# Zookeeper #############################
  
# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=10.20.0.22:2181,10.20.0.19:2181,10.20.0.20:2181
  
# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000
  
  
############################# Group Coordinator Settings #############################
  
# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=3000
```



### 4. 启动 kafka 

kafka启动 先启动zookeeper，再启动kafka。

**./zookeeper-server-start.sh -daemon ../config/zookeeper.properties**

**./kafka-server-start.sh -daemon ../config/server.properties**











