---
title: Redis安装
date: 2021-11-28 13:57:08
tags:
- Redis
- 安装
categories:
- Redis
description: Redis在linux环境下安装
sticky:
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_miro_qvwm.png
---

## Redis官方网站下载安装包

1. [Redis官方网站](https://redis.io/)
2. [Redis中文官方网站](http://www.redis.cn/)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128142143.png)

## 安装

1. **下载安装最新版的gcc编译器**

   - 查看是否安装gcc：`gcc --version`

     ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128150331.png)

   - 安装gcc：`yum install gcc`

2. **tar包上传到linux系统上并解压**

   ```bash
   # 解压命令
   tar -zxvf redis-6.2.6.tar.gz
   ```

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128142515.png)

3. **进入解压后的redis目录**

   ```bash
   cd redis-6.2.6
   ```

4. **在redis-6.2.6目录下再次执行make命令（只是编译好）**

5. **如果没有准备好C语言编译环境，make 会报错—`Jemalloc/jemalloc.h`：没有那个文件**

   - 解决方案：运行：`make distclean`

   - 在redis-6.2.6目录下再次执行`make`命令（只是编译好）

6. **跳过make test 继续执行: make install**

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128143714.png)

## 安装目录：`/usr/local/bin`

查看默认安装目录：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128143851.png)

- `redis-benchmark`:性能测试工具，可以在自己本子运行，看看自己本子性能如何
- `redis-check-aof`：修复有问题的AOF文件
- `redis-check-rdb`：修复有问题的RDB文件
- `redis-sentinel`：Redis集群使用
- `redis-server`：Redis服务器启动命令
- `redis-cli`：客户端，操作入口

## 启动

### 前台启动（不推荐）

前台启动，命令行窗口不能关闭，否则服务器停止

```bash
# 命令
redis-server
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128144149.png)

### 后台启动（推荐）

1. **备份`redis.conf`**

   拷贝一份`redis.conf`到其他目录

   ```bash
   mkdir /myredis
   cp /opt/redis/redis-6.2.6/redis.conf /myredis/
   ```

2. **后台启动设置`daemonize no`改成`yes`**

   修改redis.conf(257行)文件将里面的`daemonize no` 改成 yes，让服务在后台启动

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128145044.png)

3. **Redis启动**

   ```bash
   redis-server /myredis/redis.conf
   ```

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128145331.png)

4. **用客户端访问**

   ```bash
   redis-cli
   # 多个端口可以
   redis-cli -p 6379
   ```

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128145436.png)

5. **测试验证：ping**

   ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128145541.png)

6. **Redis关闭**

   1. **单实例关闭**：`redis-cli shutdown`

      ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128150154.png)

      也可以进入终端后再关闭

      ![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211128150021.png)

   2. **多实例关闭，指定端口关闭**：`redis-cli -p 6379 shutdown`