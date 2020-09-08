### 1、安装

版本：logstash-7.5.0.tar.gz

[下载链接](https://artifacts.elastic.co/downloads/logstash/logstash-7.5.0.tar.gz)

```
mkdir /home/work/elk
  
tar -zxf logstash-7.5.0.tar.gz
  
## 根据机器配置调整 logstash 的 java 虚拟机堆大小
vim /home/work/elk/logstash-7.5.0/config/jvm.options
  
## 此项目是 logstash 的配置文件，运行脚本的集合
git clone https://github.com/2575688848/logstash.git
```

logstash-7.5.0.tar.gz 是 logstash 本身的项目文件。

git clone 下来的 logstash 是 logstash 运行的配置文件，这里将所有的配置文件放在一起，统一管理。



###  2、启动

启动命令在 logstash 项目的 bin 目录下，每一个配置文件都有一个对应的启动脚本。

启动完之后，可以在 logs 目录下对应的日志文件看到日志信息。