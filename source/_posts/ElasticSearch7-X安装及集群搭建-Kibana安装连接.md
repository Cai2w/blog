---
title: ElasticSearch7.X安装及集群搭建+Kibana安装连接
date: 2021-11-14 14:23:36
tags:
- ElasticSearch
- Kibana
categories:
- ElasticSearch
description:
sticky:
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_fitting_piece_iilo.png
---

## 下载ES安装包

ElasticSearch下载地址：[点我](https://www.elastic.co/downloads/past-releases#elasticsearch)

我这里是下载的linux已经编译好的包，直接配置启动就行了，因为它自带了JDK等环境。

如果你想要使用你自己配置好的 Java 版本，需要设置 `JAVA_HOME` 环境变量 —— [参考](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211114143304.png)

下载好之后，我们进行`tar`包解压

```bash
# 创建es文件夹
mkdir /mydata/elasticsearch
# 进入创建的文件夹
cd /mydata/elasticsearch
# 解压
tar -zxvf elasticsearch-7.15.1-linux-x86_64.tar.gz
```

解压后的[目录组成](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html#targz-layout)：

```shell
.
├── bin # 二进制脚本存放目录，包括 elasticsearch 来指定运行一个 node，包括 elasticsearch-plugin 来安装 plugins
├── config # 包含了 elasticsearch.yml 配置文件
├── data # 节点上分配的每个 index/分片 的数据文件
├── lib
├── LICENSE.txt
├── logs
├── modules
├── NOTICE.txt
├── plugins # 插键文件存放的位置
└── README.textile
```

## 准备工作

> 不能使用 root 用户启动 es，否则会报错：
>
> `Caused by: java.lang.RuntimeException: can not run elasticsearch as root`

我们需要新建一个操作es的用户：

```bash
# 创建用户
# 给用户分配组：useradd –g 用户组 用户名
useradd -g es es
# 修改密码
passwd es
```

为了让我们的es用户能更好的操作es应用，我们修改es的文件所有者

```bash
# 递归修改es文件所有者
chown -R es:es elasticsearch-7.15.1
```

## ElasticSearch相关配置

### 单节点

```yml
vim config/elasticsearch.yml

# 集群名称
cluster.name: my-es
# 启动地址，如果不配置，只能本地访问
network.host: 0.0.0.0
# 节点名称
node.name: node-130
# 数据存储地址
path.data: /mydata/elasticsearch/elasticsearch-7.15.1/data
# 日志存储地址
path.logs: /mydata/elasticsearch/elasticsearch-7.15.1/logs
# 节点列表
discovery.seed_hosts: ["192.168.200.130"]
# 初始化时master节点的选举列表
cluster.initial_master_nodes: ["node-130"]
# 对外提供服务的端口
http.port: 9200
# 内部服务端口
transport.port: 9300    
# 跨域支持
http.cors.enabled: true
# 跨域访问允许的域名地址（正则）
http.cors.allow-origin: /.*/
```

### 集群

192.168.200.130

```yml
vim config/elasticsearch.yml

# 集群名称
cluster.name: my-es
# 启动地址，如果不配置，只能本地访问
network.host: 0.0.0.0
# 节点名称
node.name: node-130
# 数据存储地址
path.data: /mydata/elasticsearch/elasticsearch-7.15.1/data
# 日志存储地址
path.logs: /mydata/elasticsearch/elasticsearch-7.15.1/logs
# 节点列表
discovery.seed_hosts: ["192.168.200.130","192.168.200.131","192.168.200.132"]
# 初始化时master节点的选举列表
cluster.initial_master_nodes: ["node-130"]
# 对外提供服务的端口
http.port: 9200
# 内部服务端口
transport.port: 9300    
# 跨域支持
http.cors.enabled: true
# 跨域访问允许的域名地址（正则）
http.cors.allow-origin: /.*/
```

192.168.200.131

```yml
vim config/elasticsearch.yml

# 集群名称
cluster.name: my-es
# 启动地址，如果不配置，只能本地访问
network.host: 0.0.0.0
# 节点名称
node.name: node-131
# 数据存储地址
path.data: /mydata/elasticsearch/elasticsearch-7.15.1/data
# 日志存储地址
path.logs: /mydata/elasticsearch/elasticsearch-7.15.1/logs
# 节点列表
discovery.seed_hosts: ["192.168.200.130","192.168.200.131","192.168.200.132"]
# 初始化时master节点的选举列表
cluster.initial_master_nodes: ["node-130"]
# 对外提供服务的端口
http.port: 9200
# 内部服务端口
transport.port: 9300    
# 跨域支持
http.cors.enabled: true
# 跨域访问允许的域名地址（正则）
http.cors.allow-origin: /.*/
```

192.168.200.132

```yml
vim config/elasticsearch.yml

# 集群名称
cluster.name: my-es
# 启动地址，如果不配置，只能本地访问
network.host: 0.0.0.0
# 节点名称
node.name: node-132
# 数据存储地址
path.data: /mydata/elasticsearch/elasticsearch-7.15.1/data
# 日志存储地址
path.logs: /mydata/elasticsearch/elasticsearch-7.15.1/logs
# 节点列表
discovery.seed_hosts: ["192.168.200.130","192.168.200.131","192.168.200.132"]
# 初始化时master节点的选举列表
cluster.initial_master_nodes: ["node-130"]
# 对外提供服务的端口
http.port: 9200
# 内部服务端口
transport.port: 9300    
# 跨域支持
http.cors.enabled: true
# 跨域访问允许的域名地址（正则）
http.cors.allow-origin: /.*/
```

> 这里的配置基本上和单机启动一样，只不过在discovery.seed_hosts中指定了节点列表。然后将每个节点启动即可，然后分别访问各服务的9200端口，查看服务是否正常启动。
>
> 同时，也可以访问任意节点的http://host:9200/_cluster/health查看集群状态：green表示正常，yellow表示警告，red表示异常
>
> **注：并不是每个节点都需要配置cluster.initial_master_nodes**

## JVM 配置

[JVM 参数设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/jvm-options.html#jvm-options)可以通过 `jvm.options` 文件（推荐方式）或者 `ES_JAVA_OPTS` 环境变量来修改。

`jvm.options` 位于

- `$ES_HOME/config/jvm.options` 当通过 `tar` or `zip` 包安装
- `/etc/elasticsearch/jvm.options` 当通过 Debian or RPM packages

官网也介绍了如何[设置堆大小](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)。

默认情况，ES 告诉 JVM 使用一个最小和最大都为 1GB 的堆。但是到了生产环境，这个配置就比较重要了，确保 ES 有足够堆空间可用。

ES 使用 `Xms(minimum heap size)` 和 `Xmx(maxmimum heap size)` 设置堆大小。你应该将这两个值设为同样的大小。

**`Xms` 和 `Xmx` 不能大于你物理机内存的 50%。**

设置的示例：

```shell
-Xms2g 
-Xmx2g
```

## 测试

```bash
#启动es
./bin/elasticsearch &
#测试es启动是否成功
curl -get localhost:9200
```

## ES-FAQ

### Q1：`[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

```shell
echo "vm.max_map_count=262144" > /etc/sysctl.conf
sysctl -p
```

### Q2：`max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]`

```shell
sudo vim /etc/security/limits.conf
# 加入以下内容
* soft nofile 300000
* hard nofile 300000
* soft nproc 102400
* soft memlock unlimited
* hard memlock unlimited
```

修改了limits.conf，不需要重启，重新登录即生效。

查看当前用户的软限制

命令：ulimit -n  等价于 ulimit -S -n

查看当前用户的硬限制

命令：ulimit -H -n

### Q3：`master_not_discovered_exception`

主节点指定的名字要保证存在，别指定了不存在的节点名

## 其他配置

我们使用浏览器连接es可能无法连接上，这时候看一看是不是防火墙端口是否放行

```bash
# 查看防火墙状态
systemctl status firewalld
#查询端口是否开放
firewall-cmd --query-port=9200/tcp
firewall-cmd --query-port=9300/tcp
#开放端口
firewall-cmd --permanent --add-port=9200/tcp
firewall-cmd --permanent --add-port=9300/tcp
#重新载入
firewall-cmd --reload
```

## ik分词器

ElasticSearch应用时，我们常常会使用到中文，这样ElasticSearch原来的分词器就不够用了，需要安装一个中文分词器，用的多的就是IK分词器，下载地址（下载.zip包）：https://github.com/medcl/elasticsearch-analysis-ik/releases

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211114163251.png)

下载好之后，在ElasticSearch主目录下的plugins目录新建一个目录，然后将下载好的ik分词器压缩包放进去，再使用unzip解压:　

```bash
# 在ElasticSearch主目录下的plugins目录新建一个目录：analysis-ik
mkdir plugins/analysis-ik
# 将下载好的ik分词器压缩包放进去
mv elasticsearch-analysis-ik-7.15.1.zip plugins/analysis-ik/
# 进入新建的analysis-ik目录进行解压
cd plugins/analysis-ik
# 解压
unzip elasticsearch-analysis-ik-7.15.1.zip
```

做完之后重启ElasticSearch就可以了

## Kibana安装

Kibana下载地址：https://www.elastic.co/downloads/past-releases#kibana

同样的，下载编译好的包（7.15.1）

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211114152259.png)

然后解压，然后修改config/kibana.yml，添加ElasticSearch的节点配置

```yml
# 服务端口，默认5601
server.port: 5601
# 启动地址，默认localhost，如果不修改，那么远程无法访问
server.host: 0.0.0.0
# elasticsearch集群地址，旧版本是elasticsearch.url
elasticsearch.hosts: ["http://192.168.200.130:9200","http://192.168.200.131:9200","http://192.168.200.132:9200"]
# 如果ES有设置账号密码，则添加下面的账号密码设置
#elasticsearch.username: username
#elasticsearch.password: passwor
```

然后就可以启动了：

```bash
#修改文件所有者
chown -R es:es kibana-7.15.1-linux-x86_64
# 启动
kibana-7.12.0-linux-x86_64/bin/kibana
```

防火墙放行端口

```bash
#开放端口
firewall-cmd --permanent --add-port=5601/tcp
#重新载入
firewall-cmd --reload
```

启动之后，访问http://ip:5601就能访问到kibina了，kibina内部功能功能很多，像绘制图表仪表盘等等，还有很多模拟数据，可自行了解，就是说kibina主要是通过连接ElasticSearch来查询数据，然后将数据统计汇总呈现出图形化的界面来方面我们进行数据分析的。

不过我们开发常用的就是它的Dev-Tools了，用来发送Restfull的请求来访问操作ElasticSearch：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211114152522.png)

进入dev-tools后就可以自行操作了：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20211114152543.png)

