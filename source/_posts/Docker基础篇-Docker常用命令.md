---
title: Docker基础篇-Docker常用命令
date: 2022-07-31 20:09:56
tags:
- Docker
categories:
- Docker
description:
sticky:
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_mobile_interface_wakp.png
---

**看完本文📖**

* 熟悉并会使用Docker的常用命令

## 帮助启动类命令

- 启动docker： `systemctl start docker`

- 停止docker： `systemctl stop docker`

- 重启docker： `systemctl restart docker`

- 查看docker状态： `systemctl status docker`

- 开机启动： `systemctl enable docker`

- 查看docker概要信息： `docker info`

- 查看docker总体帮助文档： `docker --help`

- 查看docker命令帮助文档： `docker 具体命令 --help`

## 镜像命令

### `docker images`

列出本地主机上的镜像。

>  各个选项说明:
>
> - `REPOSITORY`：表示镜像的仓库源
> - `TAG`：镜像的标签版本号
> - `IMAGE ID`：镜像ID
> - `CREATED`：镜像创建时间
> - `SIZE`：镜像大小 
>
> 同一仓库源可以有多个 `TAG`版本，代表这个仓库源的不同个版本，我们使用 `REPOSITORY:TAG` 来定义不同的镜像。如果你不指定一个镜像的版本标签，例如你只使用 `ubuntu`，docker 将默认使用 `ubuntu:latest` 镜像 

**OPTIONS说明：**

- `-a` :列出本地所有的镜像（含历史映像层）
- `-q` :只显示镜像ID。

### `docker search 某个XXX镜像名字`

**网站：**https://hub.docker.com

**命令：**`docker search [OPTIONS] 镜像名字`

**OPTIONS说明：**

- `--limit` : 只列出N个镜像，默认25个

  > docker search --limit 5 redis

### `docker pull 某个XXX镜像名字`

下载镜像

**命令：**`docker pull 镜像名字[:TAG]`

> **docker pull 镜像名字**：没有TAG就是最新版，等价于**`docker pull 镜像名字:latest`**

### `docker system df` 

查看镜像/容器/数据卷所占的空间

### `docker rmi 某个XXX镜像名字ID`

删除镜像

**删除单个**：`docker rmi  -f 镜像ID`

**删除多个**：`docker rmi -f 镜像名1:TAG 镜像名2:TAG` 

**删除全部**：`docker rmi -f $(docker images -qa)`

## 面试题：谈谈docker虚悬镜像是什么？

仓库名、标签都是\<none\>的镜像，俗称虚悬镜像`dangling image`，后续的`Dockerfile`章节会再介绍

## 容器命令

**有镜像才能创建容器，这是根本前提**

### 新建+启动容器

`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

>  **OPTIONS说明**（常用）：有些是一个减号，有些是两个减号 
>
> - `--name="容器新名字"`    为容器指定一个名称；
>
> - `-d:` 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)； 
> - `-i`：以交互模式运行容器，通常与 -t 同时使用；
> - `-t`：为容器重新分配一个伪输入终端，通常与 -i 同时使用；也即启动交互式容器(前台有伪终端，等待交互)； 
> - `-P`: 随机端口映射，大写P
> - `-p`: 指定端口映射，小写p

**案例：启动交互式容器(前台命令行)**

> 使用镜像centos:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
>
> `docker run -it centos /bin/bash` 
>
> 参数说明：
>
> - -i: 交互式操作。
> - -t: 终端。
> - centos : centos 镜像。
> - /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
> - 要退出终端，直接输入 exit

### 列出当前所有正在运行的容器

`docker ps [OPTIONS]`

> **OPTIONS说明（常用）**： 
>
> - `-a` :列出当前所有正在运行的容器+历史上运行过的
>
> - `-l` :显示最近创建的容器。
>
> - `-n`：显示最近n个创建的容器。
>
> - `-q` :静默模式，只显示容器编号。

### 退出容器【重要】

两种退出方式：

- `exit`：run进去容器，exit退出，**容器停止**
- `ctrl+p+q`：run进去容器，ctrl+p+q退出，**容器不停止**

### 启动已停止运行的容器

`docker start 容器ID或者容器名`

### 重启容器

`docker restart 容器ID或者容器名`

### 停止容器

`docker stop 容器ID或者容器名`

### 强制停止容器

`docker kill 容器ID或容器名`

### 删除已停止的容器

`docker rm 容器ID`

一次性删除多个容器实例：

- `docker rm -f $(docker ps -a -q)`
- `docker ps -a -q | xargs docker rm`

### 重要

有镜像才能创建容器，这是根本前提

#### 启动守护式容器(后台服务器)

> 在大部分的场景下，我们希望 docker 的服务是在后台运行的，
> 我们可以使用 `-d` 指定容器的后台运行模式。

`docker run -d 容器名`

>  使用镜像`centos:latest`以后台模式启动一个容器
>
> `docker run -d centos` 
>
> **问题**：然后`docker ps -a` 进行查看, **会发现容器已经退出**
>
> 很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程.容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。 这个是docker的机制问题,比如你的web容器,我们以nginx为例，正常情况下,我们配置启动服务只需要启动相应的service即可。例如`service nginx start`但是,这样做,nginx为后台进程模式运行,就导致docker前台没有运行的应用,这样的容器后台启动后,会立即自杀因为他觉得他没事可做了.所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行，常见就是命令行模式，表示我还有交互操作，别中断

**redis 前后台启动演示case**：

- 前台交互式启动： `docker run -it redis:6.0.8`
- 后台守护式启动：`docker run -d redis:6.0.8`

#### 查看容器日志

`docker logs 容器ID`

#### 查看容器内运行的进程

`docker top 容器ID`

#### 查看容器内部细节

`docker inspect 容器ID`

#### 进入正在运行的容器并以命令行交互【重点】

`docker exec -it 容器ID bashShell`

重新进入：`docker attach 容器ID`

> 区别：
>
> - attach 直接进入容器启动命令的终端，不会启动新的进程
>   用exit退出，会导致容器的停止。
> - exec 是在容器中打开新的终端，并且可以启动新的进程
>   用exit退出，不会导致容器的停止。
>
> **推荐大家使用 docker exec 命令，因为退出容器终端，不会导致容器的停止。**

**用之前的redis容器实例进入试试**

进入redis服务：

- `docker exec -it 容器ID /bin/bash`
- `docker exec -it 容器ID redis-cli`

> 一般用`-d`后台启动的程序，再用`exec`进入对应容器实例

#### 从容器内拷贝文件到主机上

`docker cp  容器ID:容器内路径 目的主机路径`

#### 导入和导出容器

- **export**： 导出容器的内容留作为一个tar归档文件[对应import命令]

- **import** ：从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]

**案例：**

> `docker export 容器ID > 文件名.tar`
>
> `cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号`

## 结语

> 我是菜菜🥬，一个人菜瘾大的互联网从业者🎉
>
> 用功不求太猛，但求有恒🎈