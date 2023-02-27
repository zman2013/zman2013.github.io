---
title: Docker常用命令
tags:
  - docker
id: '276'
categories:
  - - container
date: 2017-09-24 22:24:16
---

```shell
docker pull gcr.io/google_containers/pause-amd64 
docker commit -m \'message\' 
docker push 
docker images 
docker rmi imageId 
docker run -i -t ubuntu /bin/bash 
docker run --name demo -d ubuntu 
docker exec -i -t containerId /bin/bash 
docker ps -a 
docker stop containerId 
docker start containerId 
docker rm containerId 
docker save gcr.io/google_containers/pause-amd64 -o pause.tar 
docker load -i pause.tar
```

# 常用容器

## 1. mongodb
```shell
# 运行容器
docker run -d -p 27017-27019:27017-27019 --name mongodb mongo
# 连接到容器
docker exec -it mongodb bash
```

# Links
https://docs.docker.com/engine/reference/commandline/docker/
