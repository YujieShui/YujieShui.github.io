---
title: Hadoop 集群搭建
date: 2017-12-19 18:56:24
categories:
- 大数据
- Hadoop
tags:
- 大数据
- hadoop
- linux
- 分布式
---

HADOOP是apache旗下的一套开源软件平台。HADOOP提供的功能有：利用服务器集群，根据用户的自定义业务逻辑，对海量数据进行分布式处理。HADOOP的核心组件有:HDFS（分布式文件系统）,YARN（运算资源调度系统）, MAPREDUCE（分布式运算编程框架）。

广义上来说，HADOOP通常是指一个更广泛的概念——HADOOP生态圈。**这是大数据学习的第一篇，学习大数据先从搭建一个 Hadoop 的集群开始吧。**

![Hadoop](http://upload-images.jianshu.io/upload_images/2791079-dff93a544c6025d7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

## 目标

十年前是最好的学习时间，次之是现在。学习大数据就从这一刻起，一步一步地学习，总有大成的一天。

第一件能做的事情就是搭建一个小的 Hadoop 集群，在有了集群的基础上就能编写程序来进行进一步的开发了。

用的是虚拟机搭建分布式环境。Hadoop 也可以用一台主机搭建,这里用三台 CentOS 7 mini 的虚拟机搭建一个小的集群。

**最开始的目标小一点，能跑起来就行。**

## 详述

HADOOP 集群具体来说包含两个集群：HDFS集群和YARN集群，两者逻辑上分离，但物理上常在一起:

- HDFS 集群，负责海量数据的存储，集群中的角色主要有NameNode / DataNode

- YARN 集群，负责海量数据运算时的资源调度，集群中的角色主要有 ResourceManager /NodeManager

也就是说接下来我们在三台主机上分别配置 HDFS 集群用于存数据，以及 YARN 集群用来进行资源管理。其中一台作为 NameNode,两台为 DataNode。

*本篇先讲配置，到底是什么用的在后面讲。*

## 准备工作

在安装 Hadoop 集群之前先要做以下几个准备工作：

- 三台主机，本文用的是 centos 7
- Java 环境，本文用的是 java 1.8.0
- 新建 Hadoop 用户，配置相同的密码
- 将 Hadoop 用户添加到 sudo 用户组
- 配置主机名和主机的映射
- 主机之间实现免密登录

<div class="note info"><p>以上配置可以参考：[CentOS 7 配置](http://shuiyujie.com/2017/11/14/linux/CentOS-%E9%85%8D%E7%BD%AE/)，[实现 SSH 免密登录](http://shuiyujie.com/2017/12/19/linux/%E5%AE%9E%E7%8E%B0-SSH-%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95/)</p></div>

我的三台主机如下：

```
192.168.2.222 main
192.168.2.223 slave1
192.168.2.224 slave3
```

## Hadoop 安装

[下载 Hadoop 安装包](http://hadoop.apache.org/releases.html)之后上传到主机并解压到`/home/hadoop/apps/hadoop/hadoop-2.6.5/`

> tar zxvf hadoop-2.6.5.tar.gz

配置`HADOOP_HOME`

> sudo vi /etc/profile
> export HADOOP_HOME = /home/hadoop/apps/hadoop/
> hadoop-2.6.5
> export PATH=$PATH:$HADOOP_HOME/bin 

使配置文件生效

> source /etc/profile

## Hadoop 配置

在`etc`目录下有若干配置文件，三台主机均按照如下配置进行修改。

### hadoop-env.sh

指定 JAVA_HOME

> vi  hadoop-env.sh
> \# The java implementation to use.
export JAVA_HOME=/usr/local/jdk1.8.0_151

### core-site.xml

配置 HDFS 临时文件夹，**注意文件夹要新建好**。以及配置 `NameNode`的主机名和端口,`dataNode`就能够发现`NameNode`。

```
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://main:9000</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/home/hadoop/apps/hadoop/hadoop-2.6.5/tmp</value>
</property>
</configuration>
```

### hdfs-site.xml

主要配置`namenode`和`datanode`保存文件的文件夹路径，也要**注意先把文件夹建立好。**

```
<configuration>
<property>
<name>dfs.namenode.name.dir</name>
<value>/home/hadoop/data/name</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>/home/hadoop/data/data</value>
</property>
 
<property>
<name>dfs.replication</name>
<value>3</value>
</property>
 
<property>
<name>dfs.secondary.http.address</name>
<value>main:50090</value>
</property>
</configuration>
```
### mapred-site.xml

配置使用 yarn 进行资源调度

```
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
</configuration>
```

### yarn-site.xml

```
<configuration>
<property>
<name>yarn.resourcemanager.hostname</name>
<value>main</value>
</property>
 
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
</configuration>
```

### slaves

slaves 文件中的主机就是`datanode`,`main`主机可以既做`namenode`也做`datanode`不过一般分开，这里也配置在了一起。

> vi slaves

```
main
slave1
slave3
```

## 启动集群

初始化HDFS
> bin/hadoop  namenode  -format
 
启动HDFS
> sbin/start-dfs.sh
 
启动YARN
> sbin/start-yarn.sh

浏览器访问 http://192.168.2.222:50070/ 可以看到如下界面

![集群启动成功](http://upload-images.jianshu.io/upload_images/2791079-800ac62674e5758a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动成功，通过浏览器我们就能看看这些信息。

## 总结

本文介绍了 Hadoop 的集群搭建，用手动启动的方式太繁琐，我们可以使用 shell 脚本启动集群，而 Hadoop 已经为我们编写好了 shell 脚本。

如何用 shell 脚本管理 Hadoop 集群？一个集群搭建起来要怎么用呢？集群中各部分的作用又是什么？

**这些在都以后的文章中回答。**

## 文章参考

> [Hadoop 中文文档](https://hadoop.apache.org/docs/r1.0.4/cn/quickstart.html)
[Hadoop 新 MapReduce 框架 Yarn 详解](https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-yarn/)
[Hadoop Yarn详解](https://yq.aliyun.com/articles/5896)




