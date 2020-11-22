---
title: Kubernetes常用命令
tags:
  - kubectl
  - kubernetes
id: '280'
categories:
  - - container
date: 2017-09-24 22:30:39
---

### 核心命令
```shell
kubectl get - list resources

kubectl describe - show detailed information about a resource

kubectl logs - print the logs from a container in a pod

kubectl exec - execute a command on a contain in a pod
```
### 1. get

1.1 查看nodes
`kubectl get nodes`

1.2 查看pod

`kubectl get pod`

1.3 查看deployment

`kubectl get deployments`

### 2. describe

2.1 查看详情
```shell
kubectl describe pods

kubectl describe pods $POD_NAME

kubectl describe nodes
```
### 3. exec
3.1 调用pod内部命令

`kubectl exec -ti $POD_NAME curl localhost:8080`

3.2 启动容器bash

`kubectl exec -ti $POD_NAME bash    （退出：exit）`

### 4. 删除service

`kubectl delete service -l run=kubernetes-bootcamp`

### 5. 查看版本
`kubectl version`

### 6. 查看集群信息
`kubectl cluster-info`

### 7. 创建一个deployment
`kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080`

### 8. 暴露服务到外网（即：创建一个service）
8.1 创建service
`kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080    (port: 为pod监听的端口）`
8.2 查看service
```shell
kubectl describe service
kubectl describe services/kubernetes-bootcamp
```

8.3 删除service
```shell
kubectl delete service $SERVICE_NAME

kubectl delete service -l run=kubernetes-bootcamp
```

### 9. 标签的使用（label）

9.1 打标签
`kubectl label pod $POD_NAME app=v1`

9.2 使用标签过滤
`kubectl get pods -l app=v1`

### 10. scale

`kubectl scale deployments/kubernetes-bootcamp --replicas=2`

### 11. 平滑升级、回滚

11.1 **升级**
`kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2`

11.2 **检查状态**
11.2.1 通过service访问服务接口
11.2.2 kubectl rollout status deployments/kubernetes-bootcamp
11.3 **回滚**
`kubectl rollout undo deployments/kubernetes-bootcamp`

### 12. namespace
kubectl create namespace mem-example

kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/memory-request-limit.yaml --namespace=mem-example

`kubectl get pod memory-demo --namespace=mem-example`
 LINK [Kubectl API](https://kubernetes.io/docs/user-guide/kubectl/v1.7/)