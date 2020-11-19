---
title: Kubernetes集群搭建
tags:
  - docker
  - etcd
  - flannel
  - kubernetes
id: '270'
categories:
  - - container
date: 2017-09-24 22:16:47
---

_献给在墙内尝试搭建最新版kubernetes集群的家伙_

## 1\. Overview

[![cluster_topology](https://www.zmannotes.com/wp-content/uploads/2017/09/cluster_topology-300x248.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/cluster_topology.png)

## 2\. 组件

OS：CentOS7

### 2.1 Master

Apiproxy (必要)

Scheduler (必要)

Controller Manager (必要)

Kubectl (非必要)

### 2.2 Node

Docker（必要）

Kubelet (必要)

Flannel（非必要）

Kube-Proxy (非必要)

Etcd (非必要)

## 3\. Start Trip

### 3.1 下载Kubernetes（所有节点）

1.  创建并进入目录
    
    mkdir ~/kubernetes && cd ~/kubernetes
    
2.  下载Kubernetes.tar.gz ([LINK](https://github.com/kubernetes/kubernetes/releases/tag/v1.7.6))
    
    wget https://github.com/kubernetes/kubernetes/releases/download/v1.7.6/kubernetes.tar.gz
    
3.  解压
    
    tar zxvf kubernetes.tar.gz
    
4.  下载二进制包
    
    ./kubernetes/cluster/get-kube-binaries.sh
    
5.  解压
    
    tar zxvf ./kubernetes/server/kubernetes-server-linux-amd64.tar.gz
    
6.  添加环境变量
    
    cd ./kubernetes/server/bin
    export PATH=\`pwd\`:$PATH
    
7.  检查兼容的etcd版本
    
    \[root@localhost ~\]# grep TAGS kubernetes/cluster/images/etcd/Makefile
    # \[TAGS=2.2.1 2.3.7 3.0.17\] \[REGISTRY=gcr.io/google\_containers\] \[ARCH=amd64\] \[BASEIMAGE=busybox\] make (buildpush)
    
    TAGS的值为兼容的版本，kubernetes1.7.6兼容etcd2.2.1，于是选择etcd2.2.1

### 3.2 搭建etcd集群

1.  下载etcd2.2.1 （[LINK](https://github.com/coreos/etcd/releases/tag/v2.2.1))
    
    curl -L  https://github.com/coreos/etcd/releases/download/v2.2.1/etcd-v2.2.1-darwin-amd64.zip -o etcd-v2.2.1-darwin-amd64.zip
    unzip etcd-v2.2.1-darwin-amd64.zip
    cd etcd-v2.2.1-darwin-amd64
    
2.  搭建集群
    
    ###将export中的ip改为对应机器的ip###
    #host1 ip 为192.168.46.31，执行以下命令
    export HostIP="192.168.46.31"
    nohup \\
    ./etcd \\
        -name=etcd0 \\
        -listen-client-urls=http://0.0.0.0:2379 \\
        -advertise-client-urls=http://${HostIP}:2379 \\
        -listen-peer-urls=http://0.0.0.0:2380 \\
        -initial-advertise-peer-urls=http://${HostIP}:2380 \\
        -initial-cluster-token etcd-cluster-1 \\
         -initial-cluster etcd0=http://192.168.46.31:2380,etcd1=http://192.168.46.32:2380,etcd2=http://192.168.46.33:2380 \\
         -initial-cluster-state new \\
    &
    
    #host2 ip为192.168.46.32，执行以下命令
    export HostIP="192.168.46.32"
    nohup \\
    ./etcd \\
        -name=etcd1 \\
        -listen-client-urls=http://0.0.0.0:2379 \\
        -advertise-client-urls=http://${HostIP}:2379 \\
        -listen-peer-urls=http://0.0.0.0:2380 \\
        -initial-advertise-peer-urls=http://${HostIP}:2380 \\
        -initial-cluster-token etcd-cluster-1 \\
         -initial-cluster etcd0=http://192.168.46.31:2380,etcd1=http://192.168.46.32:2380,etcd2=http://192.168.46.33:2380 \\
         -initial-cluster-state new \\
    &
    
    #host3 ip为192.168.46.33，执行以下命令
    export HostIP="192.168.46.33"
    nohup \\
    ./etcd \\
        -name=etcd2 \\
        -listen-client-urls=http://0.0.0.0:2379 \\
        -advertise-client-urls=http://${HostIP}:2379 \\
        -listen-peer-urls=http://0.0.0.0:2380 \\
        -initial-advertise-peer-urls=http://${HostIP}:2380 \\
        -initial-cluster-token etcd-cluster-1 \\
         -initial-cluster etcd0=http://192.168.46.31:2380,etcd1=http://192.168.46.32:2380,etcd2=http://192.168.46.33:2380 \\
         -initial-cluster-state new \\
    &
    
3.  检查集群状态
    
    etcdctl member list
    etcdctl cluster-health
    
4.  设置flannel配置
    
    etcdctl set /coreos.com/network/config '{"Network": "10.1.0.0/16","SubnetLen": 24,"SubnetMin": "10.1.0.0","SubnetMax": "10.1.9.255","Backend": {"Type":"vxlan"}}'
    

### 3.3 安装Flannel（每个node）

1.  安装flannel
    
    wget https://github.com/coreos/flannel/releases/download/v0.8.0/flanneld-amd64 && chmod +x flanneld-amd64
    
2.  启动flannel
    
    ./flanneld-amd64 --etcd-endpoints=http://192.168.46.31:2379
    
3.  检查flannel运行状态
    
    \[root@localhost ~\]# cat /var/run/flannel/subnet.env
    FLANNEL\_NETWORK=10.1.0.0/16
    FLANNEL\_SUBNET=10.1.5.1/24
    FLANNEL\_MTU=1450
    FLANNEL\_IPMASQ=false
    

### 3.4 安装Docker（每个node）

1.  安装Docker
    
    yum install docker -y
    
2.  配置docker网络
    
    #修改/etc/systemd/system/docker.service文件信息如下
    #其实就是在docker的启动命令中添加--bip和--mtu两个参数，参数信息来自flannel获取的配置
    ...
    \[Service\]
    ...
    EnvironmentFile=-/run/flannel/subnet.env (新增此行)
    ...
    ExecStart=/usr/bin/dockerd ... \\
        ... \\
        --bip=${FLANNEL\_SUBNET} --mtu=${FLANNEL\_MTU} \\ (新增此行)
        ...
    
3.  重启docker并确认网段
    
    systemctl restart docker
    ip a
    #输出信息docker0的网段应当落入flannel归属的网段，即配置文件中/var/run/flannel/subnet.env的FLANNEL\_SUBNET
    
4.  加载gcr必要镜像
    
    wget https://github.com/zman2013/kubernetes-install/raw/master/docker-images/pause-amd64.tar
    docker load -i pause-amd64.tar
    #Loaded image: gcr.io/google\_containers/pause-amd64:3.0
    

### 3.5 启动Apiserver（master节点）

nohup hyperkube apiserver \\
    --etcd-servers=http://192.168.46.31:2379 \\  #etcd集群地址
    --service-cluster-ip-range=10.1.0.0/16 \\
    --storage-backend=etcd2 \\
    --insecure-bind-address=0.0.0.0 \\
    --admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,ResourceQuota \\
    --v=4 > apiserver.log &

### 3.6 启动Scheduler（master节点）

nohup hyperkube scheduler --master=127.0.0.1:8080 > scheduler.log &  #apiserver的地址

### 3.7 启动Controller Manager（master节点）

nohup hyperkube controller-manager --master=127.0.0.1:8080 > controller-manager.log & #apiserver的地址

###  3.8 启动Kubelet（每个Node）

nohup hyperkube kubelet \\
    --api-servers=http://192.168.46.31:8080 \\
    --hostname-override=192.168.46.32 \\       #本机ip
    --cgroup-driver=systemd \\
    --v=4  > kubelet.log &

###  3.9 启动Kube-Proxy（每个Node）

nohup hyperkube kube-proxy --master=http://192.168.46.31:8080 > kube-proxy.log &

 

### Q&A

**1.** Q: Error syncing pod 39d2bd89-a174-11e7-b855-0050568723c9, skipping: failed to "StartContainer" for "POD" with ErrImagePull: "image pull failed for gcr.io/google\_containers/pause-amd64:3.0, this may be because there are no credentials on this request A: 获取pause-amd64镜像失败。参考步骤：3.4中4加载gcr必要镜像。  

### LINKS

1.  [Flannel](https://github.com/coreos/flannel)
2.  [Flanneld配置](https://github.com/coreos/flannel/blob/master/Documentation/configuration.md)
3.  [Etcd](https://github.com/coreos/etcd)
4.  [Kubernetes从零创建](https://kubernetes.io/docs/getting-started-guides/scratch/)
5.  [Kubernetes启动参数](https://kubernetes.io/docs/admin/kube-apiserver/)