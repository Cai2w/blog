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

不出意外的话，这里mysql会报错：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/Snipaste_2021-08-11_22-33-46.jpg)

**原因是因为密码设置的过于简单会报错,MySQL有密码设置的规范，具体是与validate_password_policy的值有关,下图表明该值规则**

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20190226144810183.png)

我们可以看一下mysql的初始密码规则

```mysql
SHOW VARIABLES LIKE 'validate_password%';
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/Snipaste_2021-08-11_22-41-28.jpg)

修改密码的验证策略：

```mysql
set global validate_password_policy=0;
```

密码的长度和**validate_password_length**有关，我们将密码长度降低点

```mysql
set global validate_password_length=4;
```

这样就没问题了

## 从实例搭建

> 从实例配置都基本一致，这里只以一个从实例做讲解

- 修改mysql配置文件/etc/my.cnf

```bash
[mysqld]
## 设置server_id，同一局域网中需要唯一
# 另一个从实例可以配置为103
server_id=102  
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
# 另一个从实例可以配置为mall-mysql-slave2-bin 
log-bin=mall-mysql-slave1-bin 
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

- 修改完配置后重启实例：

```
service mysqld restart
```

## 将主从数据库进行连接

- 连接到主数据库的mysql客户端，查看主数据库状态：

```mysql
show master status;
```

- 主数据库状态显示如下：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210811224928.png)

- 连接从数据库的mysql的客户端

```bash
mysql -uroot -proot
```

- 在从数据库中配置主从复制

```
change master to master_host='192.168.200.11', master_user='slave1', master_password='123456', master_port=3306, master_log_file='mall-mysql-bin.000001', master_log_pos=1136, master_connect_retry=30;
```

> slave2 需将master_user、master_password替换为先前配置的第二个账号，做到一个库一个号

- 主从复制命令参数说明：
  - master_host：主数据库的IP地址；
  - master_port：主数据库的运行端口；
  - master_user：在主数据库创建的用于同步数据的用户账号；
  - master_password：在主数据库创建的用于同步数据的用户密码；
  - master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
  - master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
  - master_connect_retry：连接失败重试的时间间隔，单位为秒。

- 查看主从同步状态：

```mysql
show slave status \G;
```

```
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.200.11
                  Master_User: slave2
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 1136
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: No		#表示还没开始同步
            Slave_SQL_Running: No		#表示还没开始同步
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1136
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

```

- 开启主从同步：

```mysql
start slave;
```

- 查看从数据库状态发现仍然没同步：

```
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 192.168.200.11
                  Master_User: slave2
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 1136
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: Connecting	
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1136
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2003
                Last_IO_Error: error connecting to master 'slave2@192.168.200.11:3306' - retry-time: 30  retries: 1
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 210811 22:01:39
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.01 sec)
```

现在插一句，先解决一部分小伙伴遇到的其他情况：**Slave_IO_Running: No**

> 哈哈，这其实是有个坑的，这一部分的小伙伴应该是先安装mysql，之后再进行复制的。

**因为mysql 有个uuid , 然而uuid 是唯一标识的，所以克隆过来的uuid是一样的，只需要修改一下uuid 就ok了，找到auto.cnf 文件修改uuid**

```mysql
#退出mysql
mysql> exit;
#查询auto.cnf
find / -iname "auto.cnf"
#修改 这里写你找到的路径
vim /var/lib/mysql/auto.cnf

# 原内容
[auto]
server-uuid=0661e980-f858-11eb-9073-000c2965b894

# 调整后内容，自定义即可，把最后一位4替换为5
[auto]
server-uuid=0661e980-f858-11eb-9073-000c2965b895
#保存退出
#重启mysql
systemctl restart mysql

#登录mysql
mysql -uroot -p

#重启slave
stop slave;
start slave;
#查看状态
show slave status \G;
```

这样，你可能就是我出现的这种情况了，哈哈哈。（**如果你都是Yes的话就可以跳过这部分了**）

> **Slave_IO_Running: Connecting，Slave_SQL_Running：Yes**出现这种情况，我想到的就是没访问到master虚拟机，难道是防火墙的问题？

查看一下master的防火墙状态

```bash
systemctl status firewalld
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210811231537.png)

果然，防火墙开着的，由于我防火墙没有放行3306端口（虽然别的端口也没放行）。

我就用比较暴力的方法了：**关防火墙**，你们别学我，你们还是老实放行端口吧

```
#关闭防火墙
systemctl stop firewalld
#查看防火墙状态
systemctl status firewalld
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210811231829.png)

防火墙关闭成功。

我们再次进入其中一个slave测试：

```mysql
#重启slave
stop slave;
start slave;
#查看状态
show slave status \G;
```

```
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.200.11
                  Master_User: slave1
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 1136
               Relay_Log_File: mall-mysql-relay-bin.000002
                Relay_Log_Pos: 325
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: Yes		#同步成功
            Slave_SQL_Running: Yes		#同步成功
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1136
              Relay_Log_Space: 537
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 11
                  Master_UUID: f8b4cd3b-f857-11eb-9172-000c293b13fb
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

到此，成功了！！

## 主从复制测试

- 在主实例中创建一个数据库`mall`；

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210811232259.png)

- 在从实例中查看数据库，发现也有一个`mall`数据库，可以判断主从复制已经搭建成功。

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210811232335.png)

至此，我们完成了mysql的主从复制。

如果想配置读写分离，请移步我的另一篇博客，导航栏搜索**shardingjdbc**，下拉文章到读写分离即可