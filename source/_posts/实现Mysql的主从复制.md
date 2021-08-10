---
title: 实现Mysql的主从复制
date: 2021-08-08 21:23:23
tags:
- 主从复制
categories:
- java
---

## 准备工作

### 配置网络

1. 通过VMware,复制三台虚拟机：master、slave1、slave2

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/Snipaste_2021-08-08_22-12-53.jpg)

2. 修改IP地址

```bash
# master: 192.168.200.11
# slave1: 192.168.200.12
# slave2: 192.168.200.13
# 打开配置文件
vim /etc/sysconfig/network-scripts/ifcfg-ens33

#IP的配置方法[none|static|bootp|dhcp](引导时不适用协议|静态分配IP|BOOTP协议|DHCP协议)
BOOTPROTO="static"
#IP地址
IPADDR=192.168.200.11
#网关
GATEWAY=192.168.200.2
#域名解析器
DNS1=192.168.200.2
```

3. 重启网络服务或重启系统生效

```bash
#重启网络 
service network restart
#重启系统  
shutdown -r now 
#重启系统  
reboot
#查看ip
ifconfig
```

### 安装mysql

1. 创建目录

```shell
#创建目录
mkdir /opt/software
#进入该目录
cd /opt/software
```

2. 使用xftp将mysql5.7上传到该目录
3. 解压安装

```shell
#解压
tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

```



## 原理

1. 当Master节点进行insert、update、delete操作时，会按顺序写入到binlog(二进制日志)中。
2. salve从库连接master主库，Master有多少个slave就会创建多少个binlog dump线程。
3. 当Master节点的binlog(二进制日志)发生变化时，binlog dump 线程会通知所有的salve节点，并将相应的binlog内容推送给slave节点。
4. I/O线程接收到 binlog 内容后，将内容写入到本地的 relay-log(中继日志)。
5. SQL线程读取I/O线程写入的relay-log，并且根据 relay-log 的内容对从数据库做对应的操作。

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/1628240013898-b1562474-5ed0-4bac-8608-28339eede028.png)

> 知道了主从复制的原理，我们也可以很清楚的知道，读写分离的写操作是必须要在master节点上进行，因为salve节点实现数据统一根据的是master节点的binlog，我们如果在slave节点进行写操作，在master节点进行读操作，二者数据是不会统一的，主从复制也就失去的意义。
>

## 主实例搭建

- 修改mysql配置文件/etc/my.cnf

```bash
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=11
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

- 修改完配置后重启实例：

  ```shell
  service mysqld restart
  ```

- 创建数据同步用户：

```shell
# 连接数据库
mysql -uroot -proot

# 创建数据同步用户
# 为slave1创建同步账号
CREATE USER 'slave1'@'192.168.200.12' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave1'@'192.168.200.12';
# 为slave2创建同步账号
CREATE USER 'slave2'@'192.168.200.13' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave2'@'192.168.200.13';
```

