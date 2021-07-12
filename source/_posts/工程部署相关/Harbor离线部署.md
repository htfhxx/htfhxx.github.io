---
title: Harbor离线部署
date: 2021-04-06
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: 工程部署相关
tags:
	- Harbor离线部署
---



## 安装Docker、Docker-compose

docker之前安装参考其他



docker-compose 离线包地址：

https://github.com/docker/compose/releases



给予可执行权限：

```
cp docker-compose-Linux-x86_64 /usr/bin/docker-compose
chmod +x /usr/bin/docker-compose
```

查看版本：

```
docker-compose -v
```

![1619510471441](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619510471441.png)



## Harbor安装

离线包下载：https://github.com/goharbor/harbor/releases

解压：

```
tar -zxf harbor-offline-installer-v2.2.1.tgz -C /usr/local/ 
```

修改harbor.yml

```
cd /usr/local/harbor 
cp harbor.yml.tmpl harbor.yml
vi harbor.yml

```

注释掉https，修改端口号：

![1619682950573](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619682950573.png)



安装

```
docker load -i /usr/local/harbor/harbor.v2.2.1.tar.gz
cd /usr/local/harbor
./prepare 
./install.sh
```

![1619510673967](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619510673967.png)



![1619510736486](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619510736486.png)

![1619510776721](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619510776721.png)

![1619510785441](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619510785441.png)



五）harbor的控制

```text
cd /usr/local/harbor/
docker-compose up -d 启动 
docker-compose stop 停止 
docker-compose restart 重新启动
```

![1619683011292](Harbor%E7%A6%BB%E7%BA%BF%E9%83%A8%E7%BD%B2/1619683011292.png)



## 排查故障

端口问题？

```
查看端口是否开启
netstat -an | grep 748
打开端口
firewall-cmd --zone=public --add-port=748/tcp --permanent
```







## Reference

https://zhuanlan.zhihu.com/p/280751946

https://blog.csdn.net/ywd1992/article/details/106012742

