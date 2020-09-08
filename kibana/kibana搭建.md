### 1、安装

版本：kibana-7.5.0

[下载链接](https://artifacts.elastic.co/downloads/kibana/kibana-7.5.0-linux-x86_64.tar.gz)

```
mkdir /home/work/elk

cd /home/work/elk

tar -zxf kibana-7.5.0-linux-x86_64.tar.gz

cd /home/work/elk/kibana-7.8.0/config

vim kibana.yml
```



编辑 kibana.yml 配置文件

```
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://ip1:9200", "http://ip2:9200", "http://ip3:9200"]
kibana.index: ".kibana"

### kibana7以后支持此配置，设置成中文。
i18n.locale: "zh-CN"

### 如果 es 设置了 xpack 安全认证，需要设置以下参数。（账号密码填写自己之前设置的）
elasticsearch.username: "elastic"
elasticsearch.password: "elastic123"
```



### 2、启动，关闭

**启动 kibana：**

```
nohup ./kibana >/dev/null 2>&1 & 
```

访问此页面：curl -XGET http://localhost:5601/app/kibana  观察是否能出现 html 页面代码

如果连接的 es 设置了 xpack 安全认证

则访问：curl -XGET -u elastic http://localhost:5601/app/kibana  

输入 elastic 用户的密码



**关闭：**

lsof -i:5601  

通过配置的端口找到进程 id

kill 进程id