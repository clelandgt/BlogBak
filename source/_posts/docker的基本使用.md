---
title: docker的基本使用
date: 2018-04-05 21:44:30
tags:
---
系统环境：mac_sierra
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/figure_captions/waters-3084551_1920.jpg)

<!-- more -->

<!-- MarkdownTOC -->

- 基础配置
    - 配置镜像加速器
- docker的ubuntu镜像
    - 安装一些常用的工具
- 常用命令
    - 镜像
    - 容器操作
        - 新建容器
        - 进入容器
        - 终止容器
        - 删除容器

<!-- /MarkdownTOC -->


## 基础配置
### 配置镜像加速器
国内从DockerHub拉取镜像有时会比较慢，可以使用Docker官网提供的加速器。

打开Perferences...->Daemon->Registry mirrors

添加 https://registry.docker-cn.com

![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/docker的基本使用/docker的基本使用_1.png)

## docker的ubuntu镜像
### 安装一些常用的工具
```
$ apt-get update
$ apt install net-tools  # ifconfig 
$ apt install iputils-ping  # ping
```


## 常用命令
### 镜像
从DockerHub的镜像仓库中获取镜像
```
 docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

```
$ docker pull ubuntu:16.04  # 拉取ubutu16.04到本地
```


```
$ docker search centos # 搜索搜索DockerHub上的镜像
$ docker image ls  # 列出镜像
$ docker image ls -a  # 中间层镜像
$ docker image ls ubuntu  # 根据仓库名列出镜像
$ docker image ls ubuntu:16.04  # 指定特定的某个镜像，也就是指定仓库名和标签
$ docker system df   # 镜像体积
```

### 容器操作
#### 新建容器
```
$ docker run -it --name ubuntu ubuntu:16.04 bash  # 新建名为ubuntu的容器并启动
$ docker container start  # 启动一个已停止的容器
$ docker container ls -a  # 列出容器
```
 
#### 进入容器
```
$ docker attach 容器
```

#### 终止容器
通过 exit 命令或 Ctrl+d 来退出终端 时，所创建的容器立刻终止
```
$ docker container stop
```

#### 删除容器
```
$ docker container rm 容器
```
