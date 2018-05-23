---
layout: post
title: docker学习笔记
subtitle: "\"部署环境还得靠自己\""
date: 2018-05-23 22:17:00
tags: 
    - 笔记
    - Docker
---

Docker作为当下非常时髦的软件部署解决方案，其简洁清晰的概念和强大的功能让我忍不住想学习一番。刚好对运维这块的知识也挺感兴趣，趁这两天学习学习docker的相关知识并持续更新学习过程中遇到的问题。

# 学习路线
首先先上
  - [官方文档](https://docs.docker.com/)
  - [中文文档](https://docs.docker-cn.com/)
  - [阮一峰教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

和其他技术文档类似，官网文档有一个[tutorial](https://docs.docker.com/get-started/)，我也是照着这个教程一步步走的。

# 遇到的问题
1. 这个问题真的非常尴尬，在Service这章，需要编写一个docker-compose.yml文件。一开始我是照着中文文档做的，结果中文官网上的这个配置文件的语法有问题，所有的冒号后面都没有跟着空格，之前又对yaml语法不熟悉，于是先去学习了一遍yaml的语法。后来发现官方文档上的yml文件是没问题的。所以看文档还得看英文原版啊...（还是得好好啃英语 Orz）

# 一些笔记
#### 使docker命令具有root权限
1. 创建`docker`用户组
```sh
sudo groupadd docker
```
2.将用户加入`docker`组
```sh
sudo usermod -aG docker $USER
```

#### 开启docker服务
```sh
service docker start
```
or
```sh
systemctl start docker
```

#### 删除镜像文件
```sh
docker irm [iamgeName]
```

#### 从官方仓库拉取镜像
```sh
docker pull [imageNaem]
```
> 默认从'Library'文件组拉取

#### 终止启动中的容器
```sh
docker container kill [containerID]
```
> 终止运行的容器文件，依然会占据硬盘空间，可以使用`docker container rm [containerID]`命令删除

#### 服务相关命令
```sh
docker stack ls              # 列出此 Docker 主机上所有正在运行的应用
docker stack deploy -c <composefile> <appname> # 运行指定的 Compose 文件
docker stack services <appname> # 列出与应用关联的服务
docker stack ps <appname> # 列出与应用关联的正在运行的容器
docker stack rm <appname> # 清除应用
docker node ls # 显示所有节点
docker swarm leave --force # 强制清除swarm
```
