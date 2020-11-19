---
title: 容器技术
tags:
  - container
  - devops
  - docker
  - kubernetes
id: '253'
categories:
  - - container
date: 2017-09-24 10:49:42
---

_介绍容器和容器管理相关技术_

## **Docker**

### 1\. 特性

轻量（相比VM）

隔离 隔离 隔离

交付完整的运行环境

_是的，就是这样几个简单、质朴的特性，颠覆互联网DevOps领域的整个交付流程。_

### 2\. Docker发展历史

[![docker_history](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_history-1024x371.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_history.png)

docker测试版本一经推出便立刻获得业界认可、热捧、支持，公司果断卖掉当前主业，全力发展docker，公司名称也改为Docker。

2015年后Docker公司便一直在努力打造围绕docker容器的商业化的生态圈。至2017年3月推出了EE和CE两个系列。

提及LXC，是因为docker最初的版本就是基于lxc实现的轻量容器化。

### 3\. Docker Archetecture

[![docker_archetecture](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_archetecture-300x157.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_archetecture.png)

docker真正干活的就是Docker daemon，而它的核心任务就是基于镜像创建容器。

SDE通过docker client或者restful API向docker daemon发送指令，docker daemon接收、解析指令，从镜像仓库拉取镜像，基于镜像创建容器。

### 4 镜像分层

[![docker_image](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_image-300x232.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_image.png)

镜像是分层的，docker daemon可以基于任何一层创建容器。且镜像一旦生成就无法被更改。只有在容器内可以修改数据，如果我们想，可以把改动作为新的一层，生成一个镜像。 上层的改动是可以覆盖下层的，举例： 我们在镜像第二层安装了emacs，又在第三层卸载了emacs。 当基于第二层镜像创建容器时，容器中是有emacs的。 当基于第三层镜像创建容器时，容器中就没有emacs了。

### 5 docker基本原理

[![docker_theory](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_theory-300x225.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/docker_theory.png)

Docker自身有些类似拉皮条的，所有的特性都是调用操作系统内核来实现的，而核心依赖的linux技术有namespaces、cgroup、aufs。

我更倾向于将namespaces理解为命名空间的隔离：进程ID、网络隔离、用户空间隔离等

cgroup是纯资源隔离：cpu、memory、device、blkio。在不适合使用容器的场景下，可以很方便的使用cgroup进行资源隔离。

aufs是实现镜像分层的根本技术手段

**_至此可以忘掉docker，如今它已经完全褪去了高大上的光环（虽然一开始就没有神秘的技术），而它的位置亦沦落为容器化的、稳定的、可信赖的工具。_**

## Kubernetes

### 1\. 诱人的标签

Inspired and informed by Google's experience and internal system

automate deploying, automate scaling, automate operating

container orchestration

open-source platform

### 2\. Kubernetes Archetecture

[![kubernetes_archetecture](https://www.zmannotes.com/wp-content/uploads/2017/09/kubernetes_archetecture-300x130.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/kubernetes_archetecture.png)

典型的Master、Node模式，Master无状态可以配置多个、Node亦是可以横向扩展。

Master核心组件：

ApiServer：网关

Scheduler：任务调度服务

Replication Controller：副本管理服务

Node核心组件：

Kubelet：负责节点与apiserver通信和执行指令的全部工作

Kube-Proxy：网络代理和轻量负载均衡，创建service的必要组件

Docker：容器组件

### 3\. Kubernetes术语

**3.1 Pod**

Kubernetes [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) are mortal. Pods in fact have a [lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/). When a worker node dies, the Pods running on the Node are also lost. A [ReplicationController](https://kubernetes.io/docs/user-guide/replication-controller/#what-is-a-replicationcontroller) might then dynamically drive the cluster back to desired state via creation of new Pods to keep your application running.

[https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/)

Each Pod in a Kubernetes cluster has a unique IP address, even Pods on the same Node

**3.2 Service**

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods.

Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service.

**3.3 Deployment**

[https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-intro/](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-intro/)

you create a Kubernetes Deploymentconfiguration. The Deployment instructs Kubernetes how to create and update instances of your application. Once you've created a Deployment, the Kubernetes master schedules mentioned application instances onto individual Nodes in the cluster.

**3.4 Kubelet**

The kubelet daemon is the primary agent that runs on each node. The kubelet daemon watches the master API server and ensures the appropriate local containers are started, remain healthy, and continue to run.

**3.5 Kube-Proxy**

The kube-proxy daemon runs on each node as a simple network proxy and load balancer for the services on that node.

**3.6 Replication Controller**

[https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#what-is-a-replicationcontroller](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#what-is-a-replicationcontroller)

A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.

**3.7 Service**

[https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/)

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods.

Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service. Services allow your applications to receive traffic.

**3.8 Namespace**

[https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)

Create a namespace so that the resources you create in this exercise are isolated from the rest of your cluster.

## 实践

### 1\.  镜像层级划分

[![image_design](https://www.zmannotes.com/wp-content/uploads/2017/09/image_design.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/image_design.png)

## 2\. 集群建设

[![cluster_topology](https://www.zmannotes.com/wp-content/uploads/2017/09/cluster_topology-300x248.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/cluster_topology.png)

基于docker和kubernetes的特性可以很方便搭建互相隔离的集群环境。

底层网络基于coreos贡献的flannel，使得集群内部的容器可以互相通信，简单好用，原理也容易理解。

**3\. DevOps流程**

[![deploy_flow](https://www.zmannotes.com/wp-content/uploads/2017/09/deploy_flow-1024x334.png)](https://www.zmannotes.com/wp-content/uploads/2017/09/deploy_flow.png)

**Link**

1.  [Docker](https://docs.docker.com/)
2.  [Cgroup](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html-single/resource_management_guide/)
3.  [Kubernetes](https://kubernetes.io/docs/home/)
4.  [Flannel](https://github.com/coreos/flannel)

  _感谢读完~_