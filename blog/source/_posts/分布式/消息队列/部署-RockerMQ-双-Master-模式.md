---
title: 部署 RockerMQ 双 Master 模式
date: 2018-01-25 21:57:27
categories:
- 分布式
tags: 
- RocketMQ
- 消息队列
---

本文介绍搭建双 master 的 RocektMQ 的集群。

<!-- more -->

# RocketMQ集群方式

首先要部署一个 RocketMQ 的集群，以下集群方式摘自网络。

推荐的几种 Broker 集群部署方式，这里的 Slave 不可写但可读，类似于 Mysql 主备方式。

## 单个 Master

这种方式风险较大，一旦Broker 重启或者宕机时，会导致整个服务不可用，不建议线上环境使用。

## 多 Master 模式

一个集群无 Slave，全是 Master，例如 2 个 Master 或者 3 个 Master。

**优点**

配置简单，单个 Master 宕机或重启维护对应用无影响，在磁盘配置为 RAID10 时，即使机器宕机不可恢复情况下，由与 RAID10 磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢） 。性能最高。

**缺点**

单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到受到影响。

**启动顺序**

1. 先启动 NameServer
2. 再机器 A，启动第一个 Master
3. 再机器 B，启动第二个 Master

## 多 Master 多 Slave 模式，异步复制

每个 Master 配置一个 Slave，有多对Master-Slave，HA 采用异步复制方式，主备有短暂消息延迟，毫秒级。

**优点**

即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，因为 Master 宕机后，消费者仍然可以从 Slave 消费，此过程对应用透明。不需要人工干预。性能同多 Master 模式几乎一样。 

**缺点**

Master 宕机，磁盘损坏情况，会丢失少量消息。

**启动顺序**

1. 启动 NameServer
2. 机器 A，启动第一个 Master
3. 机器 B，启动第二个 Master
4. 机器 C，启动第一个 Slave 
5. 机器 D，启动第二个 Slave

## 多 Master 多 Slave 模式，同步双写

每个 Master 配置一个 Slave，有多对Master-Slave，HA 采用同步双写方式，主备都写成功，向应用返回成功。

**优点**

数据与服务都无单点，Master宕机情况下，消息无延迟，服务可用性与 数据可用性都非常高 

**缺点**

性能比异步复制模式略低，大约低 10%左右，发送单个消息的 RT 会略高。目前主宕机后，备机不能自动切换为主机，后续会支持自动切换功能 。

**启动顺序**

1. 启动 NameServer
2. 机器 A，启动第一个 Master
3. 机器 B，启动第二个 Master
4. 机器 C，启动第一个 Slave
5. 机器 D，启动第二个 Slave

<div class="note info icon"><p>以上 Broker 与 Slave 配对是通过指定相同的brokerName 参数来配对，Master 的 BrokerId 必须是 0，Slave 的BrokerId 必须是大与 0 的数。另外一个 Master 下面可以挂载多个 Slave，同一 Master 下的多个 Slave通过指定不同的 BrokerId 来区分。</p></div>

---

# 部署双 master 模式的集群

出于简单和可用的考虑，本文使用双 master 的方式部署 RocketMQ 集群。更多细节看上一篇文章[RocketMQ Start](http://shuiyujie.com/2018/01/21/RocketMQ-Start/#more)，我们用上篇文章中相同的方式再部署一台主机。

## 服务器环境


| 序号 | IP | 用户名 | 角色 | 模式 |
| --- | --- | --- | --- | --- |
| 1 | 192.168.0.222 | root | nameService1,brokerServer1 | Master |
| 2 | 192.168.0.255 | Root | nameService2,brokerServer2 | Master2 |

## 修改 hosts

> vi /etc/hosts

| IP | Name |
| --- | --- |
| 192.168.0.222 | rocketmq-nameserver1 |
| 192.168.0.222 | rocketmq-master1 |
| 192.168.0.225 | rocketmq-nameserver2 |
| 192.168.0.225 | rocketmq-master2 |

## 修改配置文件

修改目标文件：

> /usr/local/rocketmq/conf/2m-noslave

该目录下有两个文件,分别修改一个，与下面的配置文件的 `brokerName` 对应

> broker-a.properties  broker-b.properties

```
#所属集群名字 
brokerClusterName=rocketmq-cluster 
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a|broker-b
#0 表示 Master，>0 表示 Slave 
brokerId=0 
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876 
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4 
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true 
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true 
#Broker 对外服务的监听端口 
listenPort=10911 
#删除文件时间点，默认凌晨 4点 
deleteWhen=04 
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88

#存储路径 
storePathRootDir=/usr/local/rocketmq/store 
#commitLog 存储路径 
storePathCommitLog=/usr/local/rocketmq/store/commitlog 
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小i
maxMessageSize=65536

#flushCommitLogLeastPages=4 
#flushConsumeQueueLeastPages=2 
#flushCommitLogThoroughInterval=10000 
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色 
#- ASYNC_MASTER 异步复制Master 
#- SYNC_MASTER 同步双写Master 
#- SLAVE 
brokerRole=ASYNC_MASTER

#刷盘方式 
#- ASYNC_FLUSH 异步刷盘 
#- SYNC_FLUSH 同步刷盘 
flushDiskType=ASYNC_FLUSH

#checkTransactionMessageEnable=false

#发消息线程池数量 
#sendMessageThreadPoolNums=128 
#拉消息线程池数量 
#pullMessageThreadPoolNums=128
```

## 配置存储路径

从配置文件中可以看到需要配置一些文件的存储位置，只要文件中配置的目录存在即可

```
#存储路径 
storePathRootDir=/usr/local/rocketmq/store 
#commitLog 存储路径 
storePathCommitLog=/usr/local/rocketmq/store/commitlog 
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
```

## 分别修改日志配置文件

> mkdir -p /usr/local/rocketmq/logs 
cd /usr/local/rocketmq/conf && sed -i 's#${user.home}#/usr/local/rocketmq#g' *.xml

## 修改 JVM 参数

runserver.sh

```
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

改成

JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=320m"
```

runbroker.sh

```
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"

改成：

JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```
 
tools.sh

```
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=128m"

改成

JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=128m"
```

## 启动两台主机的 NameServer

>cd /usr/local/rocketmq/bin
> nohup sh mqnamesrv &

## 启动BrokerServer A

> nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 & 
> netstat -ntlp
> tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
> tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log

## 启动BrokerServer A

>nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-b.properties >/dev/null 2>&1 & 
> netstat -ntlp
> tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
> tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log

![启动成功](http://upload-images.jianshu.io/upload_images/2791079-93906a9fdf4db672.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 数据清理

```
cd /usr/local/rocketmq/bin 
sh mqshutdown broker
sh mqshutdown namesrv

*等待停止 *

rm -rf /usr/local/rocketmq/store
mkdir /usr/local/rocketmq/store
mkdir /usr/local/rocketmq/store/commitlog
mkdir /usr/local/rocketmq/store/consumequeue 
mkdir /usr/local/rocketmq/store/index

*重复以上步骤重启NameServer与BrokerServer*
```

