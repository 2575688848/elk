### 1、搭建流程

#### 安装

版本：elasticsearch-7.5.0-linux-x86_64.tar.gz

[下载链接](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-linux-x86_64.tar.gz)

```
mkdir /home/work/elk
cd /home/work/elk
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-linux-x86_64.tar.gz
tar -xzf elasticsearch-7.5.0-linux-x86_64.tar.gz
```



#### jdk使用：

1. 使用自己安装的 jdk
2. 使用 es 本身带有的 jdk（也是推荐使用的方式）

第一种方式配置好JAVA_HOME ，es 启动时会自动使用 JAVA_HOME 路径的jdk

第二种方式：需要在 es 的启动脚本中声明使用的是那个 jdk ，在 /home/work/elk/elasticsearch-7.5.0/bin/elasticsearch 行首添加：

```
export JAVA_HOME=/home/work/elk/elasticsearch-7.5.0/jdk
export PATH=$JAVA_HOME/bin:$PATH
  
# 添加jdk判断
if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/home/work/elk/elasticsearch-7.5.0/jdk/bin/java"
else
        JAVA=`which java`
fi
```



### 2、配置文件

注意：⚠️  es 7.xx版本后的配置和之前的有所不同，注意使用新的配置方式

 ```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: "elk-cluster"
 
network.bind_host: 0.0.0.0
 
node.name: "node-01"
network.host: 0.0.0.0
transport.tcp.port: 9300
http.port: 9200
http.max_content_length: 100mb
http.cors.enabled: true
http.cors.allow-origin: "*"
 
# 禁止内存交换
bootstrap.memory_lock: true
 
bootstrap.system_call_filter: false
 
# 支持正则脚本
script.painless.regex.enabled: true
 
cluster.initial_master_nodes: ["node-01","node-02","node-03"]
 
discovery.seed_hosts: ["ip1:9300","ip2:9300","ip3:9300"]

 
======================== 如果有开启 xpack 安全认证的需求，则需要配置以下参数 =========================
  
# 开启 xpack 安全认证
xpack.security.enabled: true
 
# 开启 ssl 认证
xpack.security.transport.ssl.enabled: true
 
xpack.security.transport.ssl.verification_mode: certificate
 
# ca 证书路径
xpack.security.transport.ssl.keystore.path: /home/work/elk/elasticsearch-7.5.0/config/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: /home/work/elk/elasticsearch-7.5.0/config/elastic-certificates.p12
 ```



**如果设置了 xpack 安全认证 需要进行以下操作：**

注⚠️：凡是涉及到执行 bin 下脚本的操作，都需要在执行文件的行首设置启动 jdk 路径

```java
export JAVA_HOME=/home/work/elk/elasticsearch-7.5.0/jdk
export PATH=$JAVA_HOME/bin:$PATH
```

如下面的  `elasticsearch-certutil` 和 `elasticsearch-setup-passwords` 都需要设置

```
cd /home/work/elk/elasticsearch-7.5.0/bin

生产证书文件, bin 目录下执行（一路回车，就行），生成的文件在 es home 目录下

./elasticsearch-certutil ca
./elasticsearch-certutil cert --ca elastic-stack-ca.p12

cd /home/work/elk/elasticsearch-7.5.0

# 移动文件到 config 文件夹下
mv elastic-certificates.p12 elastic-stack-ca.p12  config

cd config
  
# 文件授权
chmod 640 elastic-certificates.p12 elastic-stack-ca.p12
  
# 将证书文件分发至所有节点的 config 目录下
scp elastic-certificates.p12 elastic-stack-ca.p12   work@XX.XX.XX.XX:/XX/XX
  
# 设置密码。 bin 目录下执行，会让你设置系统默认用户密码，按步骤一个个设置
./elasticsearch-setup-passwords interactive
```



**设置 java 堆大小和 垃圾收集器**

/home/work/elk/elasticsearch-7.5.0/config/jvm.options

elasticsearch-7.5.0 使用的是 jdk13 使用的是 CMS ，jdk 14 默认使用的是 G1 收集器

```
-Xms10g
-Xmx10g

################################################################
## Expert settings
################################################################
##
## All settings below this section are considered
## expert settings. Don't tamper with them unless
## you understand what you are doing
##
################################################################

## GC configuration
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
```



### 3、启动、关闭

启动：

/bin/elasticsearch -d

-d : 以后台的形式启动

 

关闭：

ps -ef | grep elasticsearch

kill  进程号



### 4、查看集群启动状态

```
curl -XGET 'http://localhost:9200/_cluster/health?pretty'
```

**如果设置了 xpack 安全认证，需要加上用户名，输入密码才能访问**

```
curl -XGET -u elastic  'http://localhost:9200/_cluster/health?pretty'
```



看到成功启动了 4 个节点，集群状态是 green ，绿色代表健康

```json
{
  "cluster_name" : "elk-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 4,
  "number_of_data_nodes" : 4,
  "active_primary_shards" : 44,
  "active_shards" : 88,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```



### 5、问题解决

#### 启动es的时候报错：

**max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]**

root 用户给 /etc/sysctl.conf 文件添加如下配置项：

```
# 一个进程可以拥有的 VMA(虚拟内存区域)的数量：
vm.max_map_count=262144
```

执行 sysctl -p  使之生效



当开启 bootstrap.memory_lock: true 表示禁用内存交换 会报下面的错误

**memory locking requested for elasticsearch process but memory is not locked**

解决方法：

vim /etc/security/limits.conf 文件尾加入以下几行 

```
# allow user 'work' mlockall
work soft memlock unlimited
work hard memlock unlimited
```

 

#### es集群节点之间无法互相发现

1、先检查配置有没有问题

2、如果配置没有问题，则很可能是data目录的原因。这里的解决方案是：在确定数据可以删除的情况下，删除data目录再重新启动。