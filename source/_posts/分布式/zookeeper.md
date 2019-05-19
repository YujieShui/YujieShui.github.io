---
title: zookeeper 的安装和配置
toc: true
categories:
  - 分布式
tags:
  - zookeeper
abbrlink: 530b5399
date: 2017-11-14 17:18:25
---

ZooKeeper 是一个分布式协调服务。其本身就是一个高可用的分布式程序，只需半数以上节点存活即可继续使用，所以使用中往往配置奇数台主机。他主要能提供主从协调、服务器节点动态上下线、统一配置管理、分布式共享锁、统一名称服务等。

本文演示在 CentOS 7 虚拟机部署和配置 zookeeper。

<!-- more -->

# 下载

首先需要下载安装包。前去[下载地址](http://www.apache.org/dyn/closer.cgi/zookeeper/)下载安装包，本文使用的 zookeeper 版本是`zookeeper-3.4.10`。

下载完成之后将安装包传到服务器，我将其传到 apps 目录下：

> scp zookeeper-3.4.10.tar.gz root@192.168.2.222:/apps/

解压安装包

> sudo tar -zxvf zookeeper-3.4.10.tar.gz -C /apps/

重命名解压文件

> sudo mv zookeeper-3.4.10 zookeeper

# 配置环境变量

修改配置文件

> vi /etc/profile
> 添加如下内容：
> \# zookeeper
> export ZOOKEEPER_HOME=/apps/zookeeper
> export PATH=\$PATH:$ZOOKEEPER_HOME/bin

*注：ZOOKEEPER_HOME 为你自己的文件位置。*

手动加载配置文件：

> source /etc/profile

# 修改 zookeeper 配置文件

配置文件放置在 `zookeeper/conf` 目录下，查看目录文件有三个，其中`zoo_sample.cfg`是范例配置文件：

```
$ls
configuration.xsl  log4j.properties  zoo_sample.cfg
```

复制`zoo_sample.cfg`并重命名`zoo.cfg`,`zoo.cfg`就是需要的配置文件：

> cp zoo_sample.cfg zoo.cfg
> vi zoo.cfg

配置文件中添加或修改如下内容：

```
# 数据目录
dataDir=/home/hadoop/zookeeper/data
# 日志目录
dataLogDir=/home/hadoop/zookeeper/log
# 包括自己在内的所有主机名称
server.1=slave1:2888:3888 (主机名, 心跳端口、数据端口)
server.2=slave2:2888:3888
server.3=slave3:2888:3888
```

*注意：slave1、slave2、slave3 是主机名即 hostname。所以需要配置主机名和 ip 的映射。*

新建对应的文件夹:

> mkdir -m 755 zookeeper/data
> mkdir -m 755 zookeeper/log

配置 myid：

> echo 1 > zookeeper/data/myid

给三台主机的名称分别配置为`1,2,3`。

**最后：在另外两台主机上使用任意方法重复如上配置并修改主机名配置完成**

# 启动 zookeeper

进入启动目录

> cd /apps/zookeeper/bin

启动

> zkServer.sh start

查看状态

> zkServer.sh status

```
Using config: /apps/zookeeper/bin/../conf/zoo.cfg
Mode: leader
----
Using config: /apps/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```

**以上，zookeeper 的基本安装配置完毕。**

