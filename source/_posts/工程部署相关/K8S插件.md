---
title: K8S插件
date: 2020-10-15
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: 工程部署相关
tags:
	- K8S插件
---





## 1 监控插件

监控可帮助您确保Kubernetes应用程序平稳运行并排除可能出现的任何问题。

**Prometheus**是一种流行的开源监视工具，许多公司都使用它来监视其IT基础结构。

还有许多其他监视工具可用：Grafana、cAdvisor、Fluentd、Jaeger、Telepresence、Zabbix



### 1.1 Prometheus结构



![普罗米修斯建筑](K8S%E6%8F%92%E4%BB%B6/architecture.png)



最左边这块就是采集的，一些短周期的任务，或者一些持久性的任务。

中间上面这块是服务发现，有很多被监控端时，需要自动发现新加入的节点。k8s内置了服务发现的机制，也就是它会连接k8s的API，去发现你部署的哪些应用，哪些pod，通通暴露出去，方便监控。

右上角是Prometheus的告警，它告警实现是有一个组件的，Alertmanager,这个组件是接收prometheus发来的告警就是触发了一些预值，会通知Alertmanager,而Alertmanager来处理告警相关的处理，然后发送给接收人，可以是email,也可以是企业微信或者钉钉等。

中间下面这块是Prometheus本身，内部是有一个TSDB的数据库。内部的采集和展示 Prometheus它都可以完成，但是展示这块UI比较low，所以借助于开源的Grafana来展示。

所有的被监控端暴露完指标之后，Prometheus会主动的抓取这些指标，存储到自己TSDB数据库里面，提供给Web UI或者Grafana，或者API clients通过PromQL来调用这些数据，PromQL相当于Mysql的SQL，主要是查询这些数据的。



![image-20201218145702252](K8S%E6%8F%92%E4%BB%B6/image-20201218145702252.png)



### 

组件关系： 

Prometheus会抓取被监控端指标，存储到自己TSDB数据库里面，提供给Grafana。

Node Exporter 负责抓取集群机器本身信息，可被Prometheus获取。



### 1.2 K8S 监控指标

* Kubernetes本身监控

  * Node数量 
  * Node资源利用率 
  * Pods数量
  * 资源对象状态 ：比如pod，service,deployment,job这些资源状态，统计

* Pod监控

  * Pod数量

  * 容器资源利用率 ：每个容器消耗了多少资源，用了多少CPU，用了多少内存

  * 应用程序：偏应用程序本身的指标了，一般运维很难拿到，所以需要开发去暴露出来

    

如果想监控node的资源，就可以放一个node_exporter,这是监控node资源的，node_exporter是Linux上的采集器，你放上去你就能采集到当前节点的CPU、内存、网络IO，等待都可以采集的。

如果想监控容器，k8s内部提供cAdvisor采集器，pod呀，容器都可以采集到这些指标，都是内置的，不需要单独部署，只知道怎么去访问这个Cadvisor就可以了。

如果想监控k8s资源对象，会部署一个kube-state-metrics这个服务，它会定时的API中获取到这些指标，帮你存取到Prometheus里，要是告警的话，通过Alertmanager发送给一些接收方，通过Grafana可视化展示。





## 2 Prometheus+ Grafana部署

### 2.1 Prometheus Server部署

```
cd /root/workspace/visual_file/
download...  https://prometheus.io/download/
tar -zxvf prometheus-2.23.0.linux-amd64.tar.gz
mv prometheus-2.23.0.linux-amd64 prometheus


/root/workspace/visual_file/prometheus/prometheus --config.file=/root/workspace/visual_file/prometheus/prometheus.yml &
```



使用systemd来启停prometheus

```
vim /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus Monitoring System
Documentation=Prometheus Monitoring System
[Service]
ExecStart=/root/workspace/visual_file/prometheus/prometheus --config.file=/root/workspace/visual_file/prometheus/prometheus.yml --web.listen-address=:9090
[Install]
WantedBy=multi-user.target
```



启动 & 停止

```
systemctl daemon-reload
systemctl start prometheus
systemctl status prometheus
systemctl enable prometheus
http://10.60.1.97:9090
```



### 2.2 Grafana部署

#### 2.2.1 安装 & 配置

RPM包方式下载安装和启动（可不看）

```
download...   https://grafana.com/grafana/download?platform=linux
rpm -ivh --nodeps grafana-7.3.5-1.x86_64.rpm

启动
/bin/systemctl daemon-reload
/bin/systemctl enable grafana-server.service
/bin/systemctl start grafana-server.service
http://10.60.1.97:3000
```



二进制包下载安装和启动（推荐）

```
# download...  https://grafana.com/grafana/download
wget https://dl.grafana.com/oss/release/grafana-7.3.6.linux-amd64.tar.gz

tar -zxvf grafana-7.3.6.linux-amd64.tar.gz
mv grafana-7.3.6  grafana
```

```
可修改配置文件
vi /root/workspace/visual_file/grafana/conf/defaults.ini
```



把grafana-server添加到systemd中

```
vi /etc/systemd/system/grafana-server.service

[Unit]
Description=Grafana
After=network.target
[Service]
ExecStart=/root/workspace/visual_file/grafana/bin/grafana-server -homepath  /root/workspace/visual_file/grafana/
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

启动 & 停止

```
systemctl start  grafana-server
systemctl status  grafana-server
systemctl enable  grafana-server
http://10.60.1.97:3000
```



#### 2.2.2 界面配置

启动后的界面：

<img src="K8S%E6%8F%92%E4%BB%B6/QQ%E5%9B%BE%E7%89%8720201216203138.png" alt="QQ图片20201216203138" style="zoom: 80%;" />



添加数据源，选择 **prometheus**

Dashboards页面 import “Prometheus 2.0 Stats”

http://10.60.1.97:9090

![KADG7NZ0LZ8P`6VB8QNEI`6](K8S%E6%8F%92%E4%BB%B6/KADG7NZ0LZ8P%606VB8QNEI%606.png)



查看数据

![KADG7NZ0LZ8P`6VB8QNEI`6](K8S%E6%8F%92%E4%BB%B6/KADG7NZ0LZ8P%606VB8QNEI%606-1608122881403.png)

![image-20201218160927860](K8S%E6%8F%92%E4%BB%B6/image-20201218160927860.png)



### 2.2.3 配置kubernetes数据源

启动插件：`Plugins` → `kubernetes` → `Enable`





## 3 Node Exporter—采集主机运行数据

### 3.1 安装部署

与传统的监控zabbix来对比的话，prometheus-server就像是mysql，负责存储数据。只不过这是时序数据库而不是关系型的数据库。数据的收集还需要其他的客户端，在prometheus中叫做exporter。针对不同的服务，有各种各样的exporter，就好比zabbix的zabbix-agent一样。

这里为了能够采集到主机的运行指标如CPU, 内存，磁盘等信息。我们可以使用 Node Exporter



下载：https://github.com/prometheus/node_exporter/releases

```
cd /root/workspace/visual_file/
被监控的机器安装node-exporter
tar -xf node_exporter-1.0.1.linux-amd64.tar.gz
新建一个目录专门安装各种exporter
mkdir prometheus_exporter
mv node_exporter-1.0.1.linux-amd64 prometheus_exporter/
cd prometheus_exporter/
mv node_exporter-1.0.1.linux-amd64/ node_exporter

```

直接打开node_exporter的可执行文件即可启动 node export，默认会启动9100端口。建议使用systemctl来启动

```
直接启动
/root/workspace/visual_file/node_exporter/node_exporter &

配置systemctl启动
vim /etc/systemd/system/node_exporter.service 

[Unit]
Description=node_exporter
After=network.target
[Service]
Restart=on-failure
ExecStart=/root/workspace/visual_file/prometheus_exporter/node_exporter/node_exporter
[Install]
WantedBy=multi-user.target

加入开机启动
systemctl enable node_exporter
systemctl start node_exporter
```

命令

```
systemctl start node_exporter
systemctl status node_exporter
systemctl stop node_exporter
```







### 3.2 配置Prometheus

配置Prometheus，收集node exporter的数据

node exporter启动后也就是暴露了9100端口，并没有把数据传到prometheus，还需要在prometheus中配置，让prometheus去pull这个接口的数据。

```
vi /root/workspace/visual_file/prometheus/prometheus.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
   #采集node exporter监控数据
  - job_name: 'master'
    static_configs:
    - targets: ['localhost:9100']
  - job_name: 'node1'
    static_configs:
    - targets: ['10.60.1.99:9100']
```

配置好之后，prometheus 9090 就能看到了

![image-20201218144459377](K8S%E6%8F%92%E4%BB%B6/image-20201218144459377.png)

导入模板

从官网查找模板 https://grafana.com/

下载json，

<img src="K8S%E6%8F%92%E4%BB%B6/image-20201218141606045.png" alt="image-20201218141606045" style="zoom:50%;" />

![image-20201218152732797](K8S%E6%8F%92%E4%BB%B6/image-20201218152732797.png)







## 命令综合



prometheus 启动

```
systemctl start prometheus
systemctl start  grafana-server
systemctl start node_exporter


http://10.60.1.97:9090
http://10.60.1.97:3000
lsof -i:9090
lsof -i:9100
lsof -i:3000
```

 状态查看

```
systemctl status prometheus
systemctl status  grafana-server
systemctl status node_exporter
```

停止

```
systemctl stop prometheus
systemctl stop  grafana-server
systemctl stop node_exporter
systemctl daemon-reload
```





## Bugs

level=error ts=2020-12-16T12:38:35.749Z caller=main.go:808 err="error starting web server: listen tcp 0.0.0.0:9090: bind: address already in use"

```
lsof -i:9090
kill pid
```

level=error ts=2020-12-18T02:42:47.814Z caller=node_exporter.go:194 err="listen tcp :9100: bind: address already in use"

```
netstat -pan | grep 9100
kill pid
```

systemd 启动失败 Failed at step EXEC spawning /root/workspace/visual_file/prometheus: Permission denied

```
配置用户名的字段去掉
```

Warning: grafana-server.service changed on disk. Run 'systemctl daemon-reload' to reload units.

```
修改了配置文件需要重新reload
systemctl daemon-reload
```





## Reference

https://www.cnblogs.com/shawhe/p/11833368.html

https://blog.csdn.net/ywd1992/article/details/85989259

https://blog.51cto.com/14143894/2438026

https://blog.csdn.net/vic_qxz/article/details/109347645