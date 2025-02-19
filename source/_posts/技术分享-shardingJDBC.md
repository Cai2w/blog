---
title: 技术分享--shardingJDBC
date: 2021-08-08 14:39:04
tags:
- 分库分表
- 主从复制
- 读写分离
- shardingJDBC
categories:
- Java
cover: https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/undraw_social_grl_562b.png
---

## 为什么要分库分表？

答案很简单：`数据库出现性能瓶颈`
数据库出现性能瓶颈，对外表现有几个方面：
大量请求阻塞

   - 在高并发场景下，大量请求都需要操作数据库，导致连接数不够了，请求处于阻塞状态。
- SQL 操作变慢
   - 如果数据库中存在一张上亿数据量的表，一条 SQL 没有命中索引会全表扫描，这个查询耗时会非常久。
- 存储出现问题
   - 业务量剧增，单库数据量越来越大，给存储造成巨大压力。

## 数据库相关优化方案

数据库优化方案很多，主要分为两大类：软件层面、硬件层面。
软件层面包括：SQL 调优、表结构优化、读写分离、数据库集群、分库分表等；
硬件层面主要是增加机器性能。

### SQL调优
SQL 调优往往是解决数据库问题的第一步，往往投入少部分精力就能获得较大的收益。
SQL 调优主要目的是尽可能的让那些慢 SQL 变快，手段其实也很简单就是让 SQL 执行尽量命中索引。
主要使用explain命令来查看sql语句的执行计划，通过观察执行结果很容易就知道该 SQL 语句是不是全表扫描、有没有命中索引。
我们观察`TYPE`列可以得知,sql语句是否进行了全表扫描,常见取值有:
ALL、index、range、 ref、eq_ref、const、system、NULL（从左到右，性能从差到好）
ALL 代表这条 SQL 语句全表扫描了，需要优化。一般来说需要达到range 级别及以上。

### 表结构优化
拿我们的表举例，现在需要向前端返回落地页的相关信息，包括落地页详情，对应产品分类，公司名称...
怎么样获取数据？
可以通过表的关联 join 最后返回整合后的结果
但是landpage表的数据有很多，通过表的关联比较费力，随着数据量增长，为了取个别的字段要关联查询几十上百万的落地页表，速度肯定会大打折扣。
优化：

> - 先查landpage表，获取主要信息，根据这批数据信息，再去查询其他表，获得其他所需字段，然后在java层面进行组合
>
> - 可以尝试将需要的别的表的字段添加到landpage表中，这种做法通常叫做数据库表冗余字段。这样做的好处是展示落地页信息不需要再关联查询产品分类表、公司名称表...

冗余字段的做法也有一个弊端，如果这个字段更新会同时涉及到多个表的更新，因此在选择冗余字段时要尽量选择不经常更新的字段。
### 架构优化
当单台数据库实例扛不住，我们可以增加实例组成集群对外服务。
当发现读请求明显多于写请求时，我们可以让主实例负责写，从实例对外提供读的能力；
如果读实例压力依然很大，可以在数据库前面加入缓存如 redis，让请求优先从缓存取数据减少数据库访问。
缓存分担了部分压力后，数据库依然是瓶颈，这个时候就可以考虑分库分表的方案了。

### 硬件优化
硬件成本非常高，一般来说不可能遇到数据库性能瓶颈就去升级硬件。
在前期业务量比较小的时候，升级硬件数据库性能可以得到较大提升；但是在后期，升级硬件得到的收益就不那么明显了。
## 项目数据库演变
### 多应用单数据库
我刚进入项目组的时候，我们是有两个客户端的：**portal（前台）、managemen（后台）**
这两个项目是共用一个数据库的。
![image.png](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628230562442-7d7d4145-aa26-4926-b3b0-790ff85424aa.png)

### 多应用多数据库
随着项目的不断推进迭代，我们需要开发新的模块---**创意工坊**。
为了开发新的模块，我们需要创建新的数据库，为了不使数据库更加混乱，我们建了一个新库用来存放创意工坊的数据。这其实就是“分库”了。
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628231267971-8d07ffea-fa59-426f-b328-6ca0dde36dc4.png)
单数据库的能够支撑的并发量是有限的，拆成多个库可以使服务间不用竞争，提升服务的性能。如果只拆分应用不拆分数据库，不能解决根本问题，整个系统也很容易达到瓶颈。

随着数据量的不断增大，读写分离也就在我们考虑的范畴了
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628232370085-c7306f7b-3ac8-4699-91a8-24c928283575.png)
如果随着业务量的逐渐增多，读数据库顶不住压力，我们可以在业务和数据库之间加一层缓存。缓存分担一部分压力后，数据库依然是瓶颈，这时候就可以考虑分库分表了。

## 分表
分库说完了，那什么时候分表呢？
拿我们的产品表、素材表等为例，爬虫每天都在爬取数据存放到这些表中，当数据增长到一定阶段后数据库查询效率就会出现明显下降。
因此，当单表数据量过大后，我们就可以考虑分表了。
如何分表呢？
水平切分和垂直拆分
**水平拆分和垂直拆分**
用户表（user）来说，表中有7个字段：id,name,age,sex,nickname,description，如果 nickname 和 description 不常用，我们可以将其拆分为另外一张表：用户详细信息表，这样就由一张用户表拆分为了用户基本信息表+用户详细信息表，两张表结构不一样相互独立。但是从这个角度来看垂直拆分并没有从根本上解决单表数据量过大的问题，因此我们还是需要做一次水平拆分。
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628234873268-199a5a9a-7299-4389-91df-65a8e03ce057.png)
还有一种拆分方法，比如表中有一万条数据，我们拆分为两张表，id 为奇数的：1，3，5，7……放在 user1， id 为偶数的：2，4，6，8……放在 user2中，这样的拆分办法就是水平拆分了。
水平拆分的方式也很多，除了上面说的按照 id 拆表，还可以按照时间维度取拆分，比如订单表，可以按每日、每月等进行拆分。

- 每日表：只存储当天的数据。
- 每月表：可以起一个定时任务将前一天的数据全部迁移到当月表。
- 历史表：同样可以用定时任务把时间超过 30 天的数据迁移到 history表。

按照我们当前的日数据量、月数据量，还不需要按照时间维度进行拆分。
**特点：**

- 垂直切分：基于表或字段划分，表结构不同。
- 水平拆分：基于数据划分，表结构相同，数据不同。
> 总结：分表主要是为了减少单张表的大小，解决单表数据量带来的性能问题。

## 分库分表带来的复杂性

### **跨库关联查询**

在单库未拆分表之前，我们可以很方便使用 join 操作关联多张表查询数据，但是经过分库分表后两张表可能都不在一个数据库中，如何使用 join 呢？
有几种方案可以解决：

- 字段冗余：把需要关联的字段放入主表中，避免 join 操作；
- 数据抽象：通过ETL等将数据汇合聚集，生成新的表；
- 全局表：比如一些基础表可以在每个数据库中都放一份；
- 应用层组装：将基础数据查出来，通过应用程序计算组装；
### **分布式事务**

单数据库可以用本地事务搞定，使用多数据库就只能通过分布式事务解决了。
常用解决方案有：基于可靠消息（MQ）的解决方案、两阶段事务提交、柔性事务等。

### **排序、分页、函数计算问题**

在使用 SQL 时 order by， limit 等关键字需要特殊处理，一般来说采用分片的思路：
先在每个分片上执行相应的函数，然后将各个分片的结果集进行汇总和再次计算，最终得到结果。

### **分布式 ID**

如果使用 Mysql 数据库在单库单表可以使用 id 自增作为主键，分库分表了之后就不行了，会出现id 重复。
常用的分布式 ID 解决方案有：

- UUID
- 基于数据库自增单独维护一张 ID表
- 雪花算法（Snowflake）

### **多数据源**

分库分表之后可能会面临从多个数据库或多个子表中获取数据，一般的解决思路有：客户端适配和代理层适配。
业界常用的中间件有：

- shardingsphere（前身 sharding-jdbc）
- Mycat
## ShardingJdbc概览
### shardingJdbc是什么
官网给出的解释是这样的：
> ### Sharding-JDBC
> 定位为轻量级Java框架，在Java的JDBC层提供的额外服务。 它使用客户端直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架。
> - 适用于任何基于JDBC的ORM框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
> - 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP等。
> - 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer，PostgreSQL以及任何遵循SQL92标准的数据库。

### shardingJdbc能干什么
- **数据分片:**
   - 读写分离
   - 分库分表
   - 分布式主键
- **分布式事务:**
   - XA强一致事务
   - 柔性事务
- **数据库治理:**
   - 配置动态化
   - 熔断&禁用
   - 调用链路追踪

**架构图:**
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628217504904-11991f71-a655-4d7a-bcee-d26381f43336.png)
其中：Registry Center中存放着一份 数据库结构和分片规则

## 主从复制
### 原理

1. 当Master节点进行insert、update、delete操作时，会按顺序写入到binlog(二进制日志)中。
1. salve从库连接master主库，Master有多少个slave就会创建多少个binlog dump线程。
1. 当Master节点的binlog(二进制日志)发生变化时，binlog dump 线程会通知所有的salve节点，并将相应的binlog内容推送给slave节点。
1. I/O线程接收到 binlog 内容后，将内容写入到本地的 relay-log(中继日志)。
1. SQL线程读取I/O线程写入的relay-log，并且根据 relay-log 的内容对从数据库做对应的操作。

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628240013898-b1562474-5ed0-4bac-8608-28339eede028.png)
> 知道了主从复制的原理，我们也可以很清楚的知道，读写分离的写操作是必须要在master节点上进行，因为salve节点实现数据统一根据的是master节点的binlog，我们如果在slave节点进行写操作，在master节点进行读操作，二者数据是不会统一的，主从复制也就失去的意义。

### 主实例搭建

- 修改mysql配置文件/etc/my.cnf
```
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
```
service mysqld restart
```

- 创建数据同步用户：
```bash
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
### 从实例搭建

- 修改mysql配置文件/etc/my.cnf
```bash
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
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
### 将主从数据库进行连接

- 连接到主数据库的mysql客户端，查看主数据库状态：
```
show master status;
```

- 主数据库状态显示如下：

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628244779856-5f8ae134-12b8-439a-a00d-3c9fe68f4e60.png)

- 连接从数据库的mysql的客户端
```
mysql -uroot -proot
```

- 在从数据库中配置主从复制
```
change master to master_host='192.168.200.11', master_user='slave1', master_password='123456', master_port=3306, master_log_file='mall-mysql-bin.000001', master_log_pos=645, master_connect_retry=30;
```

- 主从复制命令参数说明：

   - master_host：主数据库的IP地址；
   - master_port：主数据库的运行端口；
   - master_user：在主数据库创建的用于同步数据的用户账号；
   - master_password：在主数据库创建的用于同步数据的用户密码；
   - master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
   - master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
   - master_connect_retry：连接失败重试的时间间隔，单位为秒。
- 查看主从同步状态：

```
show slave status \G;
```
```
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.200.11
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 645
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: No			#表示还没开始同步
            Slave_SQL_Running: No			#表示还没开始同步
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 645
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
```
start slave;
```

- 查看从数据库状态发现已经同步：
```
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.200.11
                  Master_User: slave1
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 645
               Relay_Log_File: mall-mysql-relay-bin.000002
                Relay_Log_Pos: 325
        Relay_Master_Log_File: mall-mysql-bin.000001
             Slave_IO_Running: Yes		# 同步成功
            Slave_SQL_Running: Yes		# 同步成功
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 645
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
                  Master_UUID: cabb05d9-d404-11eb-9530-000c299fd1be
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
### 主从复制测试

- 在主实例中创建一个数据库`mall`；

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628245983280-a71697fc-a67b-4e94-ac36-ec63d15e4ffa.png)

- 在从实例中查看数据库，发现也有一个`mall`数据库，可以判断主从复制已经搭建成功。

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628246021302-5c0a86fc-ff64-4bb0-a2ea-8294b7847a5c.png)
## 读写分离
主从复制完成后，我们还需要实现读写分离，master负责写入数据，两台slave负责读取数据。怎么实现呢？
> 解决读写分离的方案有两种：**应用层解决**和**中间件解决**。

```sql
CREATE TABLE `tb_commodity_info` (
  `id` varchar(32) NOT NULL,
  `commodity_name` varchar(512) DEFAULT NULL COMMENT '商品名称',
  `commodity_price` varchar(36) DEFAULT '0' COMMENT '商品价格',
  `number` int(10) DEFAULT '0' COMMENT '商品数量',
  `description` varchar(2048) DEFAULT '' COMMENT '商品描述',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品信息表';
```
### **应用层解决**

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628246313014-65f85aaf-26f3-47b4-910d-53bd7b269f2f.png)
**优点：**

1. 多数据源切换方便，由程序自动完成；

1. 不需要引入中间件；

1. 理论上支持任何数据库；


**缺点：**

1. 由程序员完成，运维参与不到；

1. 不能做到动态增加数据源；

### **中间件解决**

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628246393683-97daf9e0-3b93-4c58-8cef-0f7314234543.png)
**优点：**


1. 源程序不需要做任何改动就可以实现读写分离；
1. 动态添加数据源不需要重启程序；



**缺点：**


1. 程序依赖于中间件，会导致切换数据库变得困难；
1. 由中间件做了中转代理，性能有所下降；



> 应用层是采用AOP的方式，通过方法名判断，方法名中有get、select、query开头的则连接slave，其他的则连接master数据库。
> 但是通过AOP的方式实现起来代码有点繁琐，有没有什么现成的框架呢，答案是有的。
> Apache ShardingSphere 是一套开源的分布式数据库中间件解决方案组成的生态圈，它由 JDBC、Proxy两部分组成。
> ShardingSphere-JDBC定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。
> 读写分离就可以使用ShardingSphere-JDBC实现。

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/1628246430164-0a4ab21b-1990-4798-87e4-d6f66651b8c8.png)
下面演示一下SpringBoot+Mybatis+druid+ShardingSphere-JDBC代码实现。
**项目配置**
版本说明：

```yml
SpringBoot：2.5.3
druid：1.1.20
mybatis-spring-boot-starter:1.3.2
sharding-jdbc-spring-boot-starter:4.0.0-RC1
```
添加sharding-jdbc的maven配置：
```sql
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.0.0-RC1</version>
</dependency>
```
然后在application.yml添加配置：
```sql
# 这是使用druid连接池的配置，其他的连接池配置可能有所不同
spring:
  main:
    allow-bean-definition-overriding: true
  shardingsphere:
    datasource:
      names: master,slave0,slave1
      master:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.200.11:3306/mall?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&useSSL=false&serverTimezone=GMT
        username: root
        password: root
      slave0:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.200.12:3306/mall?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&useSSL=false&serverTimezone=GMT
        username: root
        password: root
      slave1:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.200.13:3306/mall?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&useSSL=false&serverTimezone=GMT
        username: root
        password: root

    props:
      sql.show: true
    masterslave:
      load-balance-algorithm-type: round_robin  #round_robin    random
    sharding:
      master-slave-rules:
        master:
          master-data-source-name: master
          slave-data-source-names: slave1,slave0
```
sharding.master-slave-rules是标明主库和从库，一定不要写错，否则写入数据到从库，就会导致无法同步。
load-balance-algorithm-type是路由策略，round_robin表示轮询策略，random表示随机策略。
启动项目，可以看到以下信息，配置的三个数据源初始化，代表配置成功：
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/20210812145324.png)
编写实体类接口：

```java
import lombok.Data;

/**
 * @author wangpeixu
 * @date 2021/8/12 13:41
 */
@Data
public class Commodity {
    private String id;
    private String commodityName;
    private String commodityPrice;
    private Integer number;
    private String description;
}
```
准备就绪，开始测试！
**测试**
编写测试类

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import wang.cai2.shardingSphere.dao.CommodityMapper;
import wang.cai2.shardingSphere.entity.Commodity;

import java.util.List;
import java.util.Random;

/**
 * @author wangpeixu
 * @date 2021/8/12 13:56
 */
@SpringBootTest
public class masterTest {
    @Autowired
    private CommodityMapper commodityMapper;

    @Test
    void masterT() {
        Commodity commodity = new Commodity();
        commodity.setCommodityName("冬瓜");
        commodity.setCommodityPrice("6");
        commodity.setNumber(10000);
        commodity.setDescription("卖冬瓜");
        commodity.setId(String.valueOf(new Random().nextInt(1000)));
        commodityMapper.addCommodity(commodity);
    }

    @Test
    void queryTest() {
        for (int i = 0; i < 10; i++) {
            List<Commodity> query = commodityMapper.query();
            System.out.println("-------");
        }
    }
}
```

插入数据**masterT()**：
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/20210812220118.png)
查询数据**queryTest()**：
![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/image-20210812220324746.png)
成功

## Sharding-Jdbc实现分库分表

shardingJdbc的配置文件如下：

```
# datasource
spring:
  main:
    allow-bean-definition-overriding: true
  # 配置真实数据源
  shardingsphere:
    # 打开sql输出日志
    props:
      sql:
        show: true
    datasource:
      names: db1,db2
      # 第一个配置源
      db1:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/db_1?serverTimezone=UTC
        username: root
        password: root
      # 第二个配置源
      db2:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/db_2?serverTimezone=UTC
        username: root
        password: root
    sharding:
      #库的分片策略
      default-database-strategy:
        inline:
          sharding-column: user_id
          algorithm-expression: db$->{user_id %2 +1}
      tables:
        course:
          actual-data-nodes: db$->{1..2}.course_$->{1..2}
          # 表的分片策略
          table-strategy:
            inline:
              sharding-column: cid
              algorithm-expression: course_$->{cid % 2 + 1}
          # cid 的生成策略
          key-generator:
            column: cid
            type: SNOWFLAKE  #雪花算法
```

测试代码：

```
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import wang.cai2.shardingSphere.dao.CourseMapper;
import wang.cai2.shardingSphere.entity.Course;

import java.util.List;

@SpringBootTest
class ShardingSphereApplicationTests {

    @Autowired
    private CourseMapper courseMapper;

    @Test
    void saveCourse() {
        for (int i = 1; i <= 10; i++) {
            Course course = new Course();
            course.setCname("java" + i);
            course.setUserId(Long.valueOf(i));
            course.setCstatus("Normal" + i);
            courseMapper.saveCourse(course);
        }
    }

    @Test
    void findAll() {
        List<Course> all = courseMapper.findAll();
        for (Course course : all) {
            System.out.println(course);
        }
    }

}
```

![](https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/20210812233553.png)

我们可以看到，数据是按照我们的配置插入了

