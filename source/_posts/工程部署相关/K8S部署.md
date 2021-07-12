---
title: K8S部署
date: 2020-10-21
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: 工程部署相关
tags:
	- K8S部署
---







## 1 部署概览

平台规划

* 单master，master直接管理多个node节点
* 多master，master和node之间多一个负载均衡组件（高可用）

硬件配置要求

* 测试环境：master-**2核**4G20G，node-4核8G40G

集群部署方式

* kubeadm：一个K8S部署工具

  * 创建一个Master节点： kubeadm init
  * 将node节点加入到当前集群中： kubeadm join <Master的IP和端口>
  * 快速部署，但屏蔽了很多细节，遇到问题较难排查。

* 二进制包

  * 二进制包，手动部署每个组件
  * 手动部署非常麻烦，更可控，也利于维护

  

## 2 虚拟机环境 - 快速搭建

### 2.1 Kubeadm方式

准备环境 - 安装软件 - 部署过程 - 测试

#### 2.1.1 准备环境

准备三个虚拟机，配置好ip

* | 角色   | IP            |
  | ------ | ------------- |
  | PC     | 192.168.10.18 |
  | master | 192.168.10.3  |
  | node1  | 192.168.10.44 |
  | node2  | 192.168.10.55 |

  ```
  # 关闭防火墙
  查看版本： firewall-cmd --version
  显示状态： firewall-cmd --state
  systemctl start firewalld
  systemctl stop firewalld
  systemctl disable firewalld
   systemctl start firewalld
  
  # 关闭selinux（Linux 的一个安全子系统）
  sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
  setenforce 0  # 临时
  
  # 关闭swap
  swapoff -a  # 临时
  sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
  
  # 根据规划设置主机名
  hostnamectl set-hostname <hostname>
  hostnamectl set-hostname Master
  hostnamectl set-hostname Node2
  hostnamectl set-hostname Node3
  
  # 在master添加hosts
  cat >> /etc/hosts << EOF
  192.168.10.3 Master
  192.168.10.44 Node2
  192.168.10.55 Node3
  EOF
  
  # 配置路由参数,防止kubeadm报路由警告
  cat > /etc/sysctl.d/k8s.conf << EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF
  sysctl --system  # 生效
  
  ```

时间同步

```
  yum install ntpdate -y
  ntpdate time.windows.com
```



#### 2.2.2 软件安装

**Docker相关**

安装docker - 使用安装脚本自动安装

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
或者：
curl -sSL https://get.daocloud.io/docker | sh
```

设置开机启动

```
systemctl enable docker && systemctl start docker
docker --version
```

docker镜像源

```
vi /etc/docker/daemon.json 
systemctl restart docker.service


$ cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```

**安装kubernetes源**

```
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

**安装kubeadm，kubelet和kubectl，并开机启动**

```
# 由于版本更新频繁，这里指定版本号部署：
$ yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
$ systemctl enable kubelet
```

#### 2.2.3 部署过程

**部署Kubernetes Master**

在Master 192.168.10.3 执行。

```
$ kubeadm init \
  --apiserver-advertise-address=192.168.10.3 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.18.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

![image-20201020160006708](K8S%E9%83%A8%E7%BD%B2/image-20201020160006708.png)

使用kubectl工具：

```bash
mkdir -p $HOME/.kube
 cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl get nodes
```

![image-20201020160146561](K8S%E9%83%A8%E7%BD%B2/image-20201020160146561.png)

如果装乱了重来：

```
# 卸载服务
kubeadm reset
# 删除rpm包
rpm -qa|grep kube*|xargs rpm --nodeps -e
# 删除容器及镜像
docker images -qa|xargs docker rmi -f
```



加入Kubernetes Node

在要被添加的Node节点虚拟机中执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```
$ kubeadm join 192.168.10.3:6443 --token lgxrim.idkgqpiowuzdbch9 \
    --discovery-token-ca-cert-hash sha256:9266780f8008eb60d6d154d09bd20e71fd50455f1b4d7bb8c6308f0c483a3ef7 
```

![image-20201020161052754](K8S%E9%83%A8%E7%BD%B2/image-20201020161052754.png)

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```
kubeadm token create --print-join-command
```

**部署CNI网络插件**

部署CNI网络插件

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```



默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

$ kubectl get pods -n kube-system
```

多等一会才会启动完成：

![image-20201020161446118](K8S%E9%83%A8%E7%BD%B2/image-20201020161446118.png)



![image-20201020161726697](K8S%E9%83%A8%E7%BD%B2/image-20201020161726697.png)

#### 2.2.4 部署应用-测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```
$ kubectl create deployment nginx --image=nginx

通过yaml文件来部署
$ kubectl create deployment web --image=nginx --dry-run -o yaml > web.yaml
$ kubectl apply -f web.yaml
```

![image-20201020161900890](K8S%E9%83%A8%E7%BD%B2/image-20201020161900890.png)

![image-20201020162021491](K8S%E9%83%A8%E7%BD%B2/image-20201020162021491.png)

pod创建不成功时，查看日志：

````
kubectl describe pod nginx
````



删除pod和deployment

```
[root@k8smaster ~]# kubectl get  pod
[root@k8smaster ~]# kubectl delete pod nginx-86c57db685-shkkq
[root@k8smaster ~]# kubectl get deployment
[root@k8smaster ~]# kubectl delete deployment nginx
```

```
对外暴露端口
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
```

![image-20201020162225756](K8S%E9%83%A8%E7%BD%B2/image-20201020162225756.png)

访问地址：http://NodeIP:Port  http://192.168.10.44:30695

![image-20201020162447346](K8S%E9%83%A8%E7%BD%B2/image-20201020162447346.png)



### 2.2 二进制包的方式部署和搭建

1. 准备环境
2. 安装软件
3. 为etcd和apiserver自签证书
4. 部署etcd集群
5. 部署master组件
   1. kube-apiserver
   2. kube-controller-manager
   3. kube-scheduler
   4. etcd
6. 部署node组件
   1. kubelet
   2. kube-proxy
   3. docker
   4. etcd
7. 部署集群中网络插件





## 3 生产环境部署

### 3.1 准备环境

各个节点的IP

* | 角色                | IP         |
  | ------------------- | ---------- |
  | K8S_Node1（Master） | 10.60.1.96 |
  | K8S_Node3           | 10.60.1.98 |




防火墙与端口

```
# 生产环境关闭防火墙不安全，所以开放几个端口(未测试是否生效，因为被服务器的网络搞的心态崩了索性把防火墙关了，服务器不连外网所以问题不大)
查看版本： firewall-cmd --version
显示状态： firewall-cmd --state
systemctl stop firewalld
systemctl disable firewalld

Master节点：
TCP	入站	6443*	Kubernetes API 服务器	所有组件
TCP	入站	2379-2380	etcd server client API	kube-apiserver, etcd
TCP	入站	10250	Kubelet API	kubelet 自身、控制平面组件
TCP	入站	10251	kube-scheduler	kube-scheduler 自身
TCP	入站	10252	kube-controller-manager	kube-controller-manager 自身
Node节点：
TCP	入站	10250	Kubelet API	kubelet 自身、控制平面组件
TCP	入站	30000-32767	NodePort 服务**	所有组件

查看版本： firewall-cmd --version
显示状态： firewall-cmd --state     systemctl status firewalld
查看已经开放的端口：firewall-cmd --list-ports
开放端口：firewall-cmd --zone=public --add-port=80/tcp --permanent

 firewall-cmd --zone=public --add-port=6443/tcp --permanent
 firewall-cmd --zone=public --add-port=2379/tcp --permanent
 firewall-cmd --zone=public --add-port=2380/tcp --permanent
 firewall-cmd --zone=public --add-port=10250/tcp --permanent
 firewall-cmd --zone=public --add-port=10251/tcp --permanent
 firewall-cmd --zone=public --add-port=10252/tcp --permanent
 systemctl reload firewalld
重启失败的话可以重新开启   systemctl start firewalld
firewall-cmd --list-ports

```

```
# 关闭selinux（Linux 的一个安全子系统）
 sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
 sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostname 查看主机名
hostname -i：查看本机对应的IP

hostnamectl set-hostname <hostname>
 hostnamectl set-hostname K8SNode1
 hostnamectl set-hostname K8SNode2
 hostnamectl set-hostname K8SNode3
 hostnamectl set-hostname K8SNode4

# 在master添加hosts
 cat >> /etc/hosts << EOF
10.60.1.96 K8SNode1
10.60.1.98 K8SNode3
EOF

# 配置路由参数,防止kubeadm报路由警告
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com

遇到：16 Nov 17:07:33 ntpdate[416478]: no server suitable for synchronization found
开放防火墙123端口无效
```





### 3.2 软件安装与部署

Docker 参考之前的

内网服务器不通外网，kubeadm无法拉取docker镜像，很头疼

这里尝试了一些离线部署方式：https://github.com/haiker2011/k8s-offline-install

sealos 需要ssh到root： https://github.com/fanux/sealos 



https://www.cnblogs.com/straycats/p/8506959.html 中分享了一个离线包但是版本太旧了

```
tar -xjvf k8s_images.tar.bz2

 docker load <k8s_images/docker_images/etcd-amd64_v3.1.10.tar
 docker load <k8s_images/docker_images/flannel_v0.9.1-amd64.tar
 docker load <k8s_images/docker_images/k8s-dns-dnsmasq-nanny-amd64_v1.14.7.tar
 docker load <k8s_images/docker_images/k8s-dns-kube-dns-amd64_1.14.7.tar
 docker load <k8s_images/docker_images/k8s-dns-sidecar-amd64_1.14.7.tar
 docker load <k8s_images/docker_images/kube-apiserver-amd64_v1.9.0.tar
 docker load <k8s_images/docker_images/kube-controller-manager-amd64_v1.9.0.tar
 docker load <k8s_images/docker_images/kube-scheduler-amd64_v1.9.0.tar
 docker load <k8s_images/docker_images/kube-proxy-amd64_v1.9.0.tar
 docker load <k8s_images/docker_images/pause-amd64_3.0.tar
 docker load <k8s_images/kubernetes-dashboard_v1.8.1.tar
```

```
cd k8s_images
 rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm
 rpm -ivh kubernetes-cni-0.6.0-0.x86_64.rpm  

 rpm -ivh kubelet-1.9.9-9.x86_64.rpm
 rpm -ivh kubectl-1.9.0-0.x86_64.rpm
 rpm -ivh kubeadm-1.9.0-0.x86_64.rpm
```



yum remove kube*



新找的离线包进行部署：

```
 rpm -ivh  ...
```

遇到的问题：获取的截获离线安装包在安装过程中缺少依赖，离线安装了一些缺少的依赖后：1. 一些依赖不生效； 2. 一些依赖包网上找不到。



总体而言，离线部署的问题：

1. 缺乏服务器环境适配的离线包和依赖包。
2. 没有一个详尽的文档，踩的坑很多，无从下手。





最终要来了root账号，使用sealos来部署。。。



## 4 Sealos 生产环境部署

### 4.1 准备工作

各个节点的IP

* | 角色       | IP         |
  | ---------- | ---------- |
  | K8S_Master | 10.60.1.97 |
  | K8S_Node   | 10.60.1.99 |



```
# 下载并安装sealos, sealos是个golang的二进制工具，直接下载拷贝到bin目录即可, release页面也可下载
$ wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/latest/sealos && \
    chmod +x sealos && cp sealos /usr/bin 

# 下载离线资源包
$ wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/7b6af025d4884fdd5cd51a674994359c-1.18.0/kube1.18.0.tar.gz

# 安装一个单 master的kubernetes集群
$ sealos init --passwd 'dcy2020@ict.com' --master 10.60.1.97  --node 10.60.1.99	    --pkg-url /home/nlper/huotengfei/kube1.18.0.tar.gz 	--version v1.18.0
```





### 4.2 Bugs

```
sealos init 执行无响应
```

重新安装之前的版本（例如3.3.8）解决问题



```
[ERROR SystemVerification]: unsupported graph driver: vfs
```

需要系统文件格式为ext4，需要格式化系统；

或者将docker的文件存储格式改为overlay2，然而docker就启动不起来了，可能是因为linux内核版本为3，需要升级内核

```
vi /etc/docker/daemon.json, 添加如下
 
 "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
```

docker存储驱动改成overlay2后出现的问题： 无法启动docker

```
$ systemctl start docker
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
```

```
查看错误信息
$ systemctl status docker.service -l
failed to start daemon: error initializing graphdriver: overlay2: the backing xfs filesystem is formatted without d_type support, which leads to incorrect behavior. Reformat the filesystem with ftype=1 to enable d_type support. Backing filesystems without d_type support are not supported.
```

```
无效的尝试
vi daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
无效的尝试2
https://www.jianshu.com/p/93518610eea1
Nov 16 15:45:52 k8snode2 dockerd[382923]: time="2020-11-16T15:45:52.943234353+08:00" level=warning msg="could not use snapshotter zfs in metadata plugin" error="path /var/lib/docker/containerd/daemon/io.containerd.snapshotter.v1.zfs must be a zfs filesystem to be used with the zfs snapshotter"
无效的尝试3
https://www.cnblogs.com/ayanmw/p/10258171.html
```

将docker的文件存储格式改为overlay，device-mapper 试试： 也是一样，都不支持



![image-20201118152727459](K8S%E9%83%A8%E7%BD%B2/image-20201118152727459.png)

![image-20201125123127574](K8S%E9%83%A8%E7%BD%B2/image-20201125123127574.png)



## 5 离线包生产环境部署

### 5.1 准备工作

各个节点的IP

* | 角色      | IP         |
  | --------- | ---------- |
  | k8snode2  | 10.60.1.99 |
  | k8smaster | 10.60.1.97 |




防火墙与端口

```
# 生产环境关闭防火墙不安全，所以开放几个端口(未测试是否生效，因为被服务器的网络搞的心态崩了索性把防火墙关了，服务器不连外网所以问题不大)
查看版本： firewall-cmd --version
显示状态： firewall-cmd --state
systemctl stop firewalld
systemctl disable firewalld

Master节点：
TCP	入站	6443*	Kubernetes API 服务器	所有组件
TCP	入站	2379-2380	etcd server client API	kube-apiserver, etcd
TCP	入站	10250	Kubelet API	kubelet 自身、控制平面组件
TCP	入站	10251	kube-scheduler	kube-scheduler 自身
TCP	入站	10252	kube-controller-manager	kube-controller-manager 自身
Node节点：
TCP	入站	10250	Kubelet API	kubelet 自身、控制平面组件
TCP	入站	30000-32767	NodePort 服务**	所有组件

查看版本： firewall-cmd --version
显示状态： firewall-cmd --state     systemctl status firewalld
查看已经开放的端口：firewall-cmd --list-ports
开放端口：firewall-cmd --zone=public --add-port=80/tcp --permanent

 firewall-cmd --zone=public --add-port=6443/tcp --permanent
 firewall-cmd --zone=public --add-port=2379/tcp --permanent
 firewall-cmd --zone=public --add-port=2380/tcp --permanent
 firewall-cmd --zone=public --add-port=10250/tcp --permanent
 firewall-cmd --zone=public --add-port=10251/tcp --permanent
 firewall-cmd --zone=public --add-port=10252/tcp --permanent
 systemctl reload firewalld
重启失败的话可以重新开启   systemctl start firewalld
firewall-cmd --list-ports

```

```
# 关闭selinux（Linux 的一个安全子系统）
 sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时
ls

# 关闭swap
swapoff -a  # 临时
 sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostname 查看主机名
hostname -i：查看本机对应的IP

hostnamectl set-hostname <hostname>
 hostnamectl set-hostname K8SNode2
 hostnamectl set-hostname K8SNode4

# 在master添加hosts
 cat >> /etc/hosts << EOF
10.60.1.96 K8SNode1
10.60.1.98 K8SNode3
EOF

# 配置路由参数,防止kubeadm报路由警告
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com

遇到：16 Nov 17:07:33 ntpdate[416478]: no server suitable for synchronization found
开放防火墙123端口无效
```



配置harbor： 证书、host







### 5.2 软件安装

k8s 离线软件**kubeadm、kubelet、kubectl**

```
 rpm -ivh xxx.rpm
```



load镜像

```
tar -xjvf k8s_images.tar.bz2

 docker load <k8s_images/docker_images/etcd-amd64_v3.1.10.tar

```



修改kubelet的配置文件 - 关于docker



### 5.3 搭建集群



kubeadm init

```
kubeadm init --kubernetes-version=v1.16.4  --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.60.1.97
```







使用kubectl工具在master节点进行如下操作：

```bash
 rm -rf $HOME/.kube
 mkdir -p $HOME/.kube
 cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 chown $(id -u):$(id -g) $HOME/.kube/config

$  kubectl get nodes
```

![image-20201020160146561](K8S%E9%83%A8%E7%BD%B2/image-20201020160146561.png)



**加入Kubernetes Node**

在要被添加的Node节点虚拟机中执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```
kubeadm join 10.60.1.97:6443 --token p1fxnc.p41jgp16q33sw04r \
    --discovery-token-ca-cert-hash sha256:a88cb283e9768f031acbb400f98dd523e10ab8ca1d657b42c672d69fd9cb936e
```

![image-20201020161052754](K8S%E9%83%A8%E7%BD%B2/image-20201020161052754.png)

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下（还没试过）：

```
kubeadm token create --print-join-command
# 永久token?
kubeadm token create --ttl 0
kubeadm token list
```



**部署CNI网络插件**

部署CNI网络插件

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

$ kubectl get pods -n kube-system
```

发现官网给的命令拉镜像失败

<img src="K8S%E9%83%A8%E7%BD%B2/image-20201204111138177.png" alt="image-20201204111138177" style="zoom:67%;" />

查看kube-flannel.yml文件发现本地镜像名字没对应上，于是修改镜像名称：

```
docker tag quay.io/coreos/flannel:v0.11.0-amd64  quay.io/coreos/flannel:v0.13.1-rc1
```

发现节点not ready,在节点查看日志，定位问题

```
journalctl -f -u kubelet
```

Failed to start ContainerManager failed to initialize top level QOS containers:

```
 systemctl stop kubepods.slice
 systemctl restart kubelet

from: http://www.ichenfu.com/2019/12/06/kubelet-failed-to-initialize-top-level-qos-containers/
```



![image-20201204193252438](K8S%E9%83%A8%E7%BD%B2/image-20201204193252438.png)

![image-20201204193304895](K8S%E9%83%A8%E7%BD%B2/image-20201204193304895.png)



### 5.4 部署应用-测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```
$  kubectl create deployment nginx --image=nginx

通过yaml文件来部署
$  kubectl create deployment nginx --image=nginx --dry-run -o yaml > nginx.yaml

修改yaml让其添加本地镜像：
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: hub.ict.ac.cn/wde/nginx:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 80
          securityContext:
            privileged: true

$  kubectl apply -f nginx.yaml
```

![image-20201020161900890](K8S%E9%83%A8%E7%BD%B2/image-20201020161900890.png)

![image-20201020162021491](K8S%E9%83%A8%E7%BD%B2/image-20201020162021491.png)

```
对外暴露端口
$  kubectl expose deployment nginx --port=80 --type=NodePort
$  kubectl get pod,svc
```

![image-20201204193501504](K8S%E9%83%A8%E7%BD%B2/image-20201204193501504.png)

访问地址：http://NodeIP:Port  http://10.60.1.99:32546



![image-20201204193534427](K8S%E9%83%A8%E7%BD%B2/image-20201204193534427.png)













## 5目前的进展与问题

###  11.09 问题



离线部署问题

* 服务器离线环境，即使挂上代理后
  * k8S插件kubelet、 kubeadm、 kubectl 网络or镜像问题yum不下来，配置了k8S阿里源也不行
  * 插件需要的镜像也拉不下来，配置了阿里源也不行

* 使用离线包下载
  * 官方网站不公开离线rpm包
  * k8S节点部署所需要的镜像离线包也没有
  * 在一些博客分享的离线包版本不一致或too old



**请教老师后**

k8S的离线安装受到网络环境限制，体现在yum源插件获取、插件安装过程需要拉取的docker镜像

1. 推荐了sealos离线部署方式
   可行性很大，但是尝试过程中需要ssh到root账号（非）
   不知是否可以提供？或者可以另开2台没有其他服务的服务器，以免影响其他进程


2. 获取了ZK老师截获的离线rpm包和habor上的镜像
   推荐了ansiable来控制集群，也提供了一套3master集群方案。但是需要和系统版本和软件生态对应（例如需要系统版本centos7.6，现有版本为7.2）

   关于版本问题，张凯老师建议我部署到能重装系统的虚拟机

   可以尝试，但结果不可控

   离线包部署非常复杂，时间耗费多
   
3. 张凯老师当时在闲置的windows上的虚拟机上部署





### 11.24 问题



**离线包手动部署：**

过程中的很多问题提issue都不知道该往哪提

安装现有的rpm包过程中，系统提示缺少核心依赖，找到一部分，其中一部分始终没有找到

离线部署并没有一个完整详尽的文档，要踩的坑还很多，终日不断的试错却没有进展让人 心力交瘁，身心折磨。



**离线工具部署 - sealos**

过程中解决了很多问题，issue都提了很多个

https://github.com/fanux/sealos/issues/527 出现存储格式不兼容问题无法继续

解决的办法：1. 将系统文件格式改为ext4，需要格式化系统；2. 将docker的文件存储格式改为overlay2或其他，然而docker就启动不起来了，因为linux内核版本过低且存储系统不支持d_type，需要升级内核。。。



**建议换到一个可升级内核重装系统(可联外网就更好了)的部署环境**or **请天玑熟悉这块的运维老师来部署**

<font size=4 color='red' > ...</font> 



### 12.08 进展

已经搭建好了k8s 集群，目前是单master多node的集群。

总结：

安装工具所需的软件依赖与yum源配置、安装工具所需的docker镜像、集群搭建的各种bug通过查看日志来定向解决（例如CNI网络插件、cgroup配置等)



接下来就是把现有的服务部署一下。



## 6 常用命令

### 6.1 检查日志和错误等

```
 journalctl -xefu kubelet
systemctl status kubelet
docker ps | grep NAME
kubectl get cs
 kubectl get nodes #获取集群状态
 kubectl get pods -n kube-system
```



### 6.2 重启与恢复

```
 systemctl daemon-reload 
 systemctl restart kubelet
```



### 6.3 部署相关

重置系统（慎用！！！）

```
# 卸载服务
kubeadm reset
# 删除rpm包
rpm -qa|grep kube*|xargs rpm --nodeps -e
# 删除容器及镜像
docker images -qa|xargs docker rmi -f
```



重置集群（慎用！）

```
 kubeadm reset
 systemctl stop kubelet
 systemctl stop docker
 rm -rf /var/lib/cni/
 rm -rf /var/lib/kubelet/*
 rm -rf /etc/cni/
 ifconfig cni0 down
 ifconfig flannel.1 down
 ifconfig docker0 down
 ip link delete cni0
 ip link delete flannel.1
 systemctl start docker
 systemctl start kubectl

```



删除pod（慎用）

```
 kubectl get  pod
 kubectl delete pod nginx-86c57db685-shkkq
 kubectl get deployment
 kubectl delete deployment nginx
 kubectl get svc -n default
 kubectl delete svc nginx -n default
```







### 常见bug

kubeadm init 报错可能的解决方式

```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
 
systemctl daemon-reload && systemctl restart kubelet
```

[dcy@k8smaster ~]$  kubectl get nodes
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes"

```
 rm -rf $HOME/.kube 部署之前没删干净 from: https://blog.csdn.net/woay2008/article/details/93250137
```

Failed to start ContainerManager failed to initialize top level QOS containers: root container [kube

```
vi /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=--feature-gates SupportPodPidsLimit=false --feature-gates SupportNodePidsLimit=false
再重启 systemctl daemon-reload && systemctl restart kubelet

from: https://blog.csdn.net/weixin_44144334/article/details/103356026
```





Failed to start ContainerManager failed to initialize top level QOS containers: failed to update top level BestEffort QOS cgroup :

```
 systemctl stop kubepods.slice
 systemctl restart kubelet

from: http://www.ichenfu.com/2019/12/06/kubelet-failed-to-initialize-top-level-qos-containers/
```

![image-20201214161200028](K8S%E9%83%A8%E7%BD%B2/image-20201214161200028.png)





Failed to start ContainerManager failed to initialize top level QOS containers: root container [kubepods] doesn't exist:

````
解决办法参考这里，在kubelet 的启动参数中加上下面参数并重启
--cgroups-per-qos=false  --enforce-node-allocatable=""

 vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cgroups-per-qos=false  --enforce-node-allocatable="" "
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cgroups-per-qos=false  --enforce-node-allocatable='' "
````



failed to run Kubelet: no client provided, cannot use webhook authentication

```
上面的配置 vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 语法问题？...
双引号内的"" 改为单引号
```

CNI failed to retrieve network namespace path: cannot find network namespace for the terminated container "ce9f100a2c6d81cde318b8491336a78eaf388af0255fce6885c7729944861f84

```

```



failed to collect filesystem stats - rootDiskErr: <nil>, extraDiskErr: could not stat "/var/lib/docker/containers/e5d4ab897e065554fa1cd14764031589f17d064269cdb81fc6079daa23509441" to get inode usage: stat /var/lib/docker/containers/e5d4ab897e065554fa1cd14764031589f17d064269cdb81fc6079daa23509441: no such file or directory

```

```

failed to collect filesystem stats - rootDiskErr: <nil>, extraDiskErr: could not stat

```

```

[root@k8smaster ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
Unable to connect to the server: unexpected EOF

```
把这个yml文件整到本地去run
```





## Reference

https://www.bilibili.com/video/BV1GT4y1A756?p=2

https://github.com/haiker2011/k8s-offline-install

https://github.com/fanux/sealos

