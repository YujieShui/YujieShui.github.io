---
title: RocketMQ Start
date: 2018-01-21 17:39:45
categories:
- 分布式
- 消息队列
tags: 
- RocketMQ
- 消息队列
---

RocketMQ 是一款分布式、队列模型的消息中间件，具有以下特点：

1. 能够保证严格的消息顺序 
2. 提供丰富的消息拉取模式 
3. 高效的订阅者水平扩展能力 
4. 实时的消息订阅机制 
5. 亿级消息堆积能力

今天开始试着理解她。

<!-- more -->

---

# 快速开始

## 环境要求

1. 64bit OS, Linux/Unix/Mac is recommended;
2. 64bit JDK 1.8+;
3. Maven 3.2.x
4. Git

## 准备工作

下载安装包 [here](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.2.0/rocketmq-all-4.2.0-source-release.zip) 到服务器中，本文使用的是`rocketmq-all-4.2.0`。

> unzip rocketmq-all-4.2.0-source-release.zip
> cd rocketmq-all-4.2.0/
> mvn -Prelease-all -DskipTests clean install -U

<div class="note danger"><p>没有注意环境，maven 版本为 3.0.5，升级 maven 到 3.5.2</p></div>

> cd distribution/target/apache-rocketmq

启动 Name Server 

> nohup sh bin/mqnamesrv &
> tail -f ~/logs/rocketmqlogs/namesrv.log
> The Name Server boot success...

启动 Broker

> nohup sh bin/mqbroker -n localhost:9876 &
> tail -f ~/logs/rocketmqlogs/broker.log 

<div class="note danger"><p>错误：启动不成功，看错误日志</p></div>

```
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 8589934592 bytes for committing reserved memory.
# Possible reasons:
#   The system is out of physical RAM or swap space
#   In 32 bit mode, the process size limit was hit
# Possible solutions:
#   Reduce memory load on the system
#   Increase physical memory or swap space
#   Check if swap backing store is full
#   Use 64 bit Java on a 64 bit OS
#   Decrease Java heap size (-Xmx/-Xms)
#   Decrease number of Java threads
#   Decrease Java thread stack sizes (-Xss)
#   Set larger code cache with -XX:ReservedCodeCacheSize=
# This output file may be truncated or incomplete.
#
#  Out of Memory Error (os_linux.cpp:2640), pid=3437, tid=0x00007f378a5c2700
#
# JRE version:  (8.0_151-b12) (build )
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.151-b12 mixed mode linux-amd64 compressed oops)
# Core dump written. Default location: /apps/rocketmq-all-4.2.0/distribution/core or core.3437
#
```
<div class="note info"><p>原因：rocketmq默认jvm配置较高，导致内存不足
</p></div>

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

再次启动 Broker 成功

> The broker[main, 192.168.0.222:10911] boot success. serializeType=JSON and name server is localhost:9876

查看一下服务 Jps

```
[root@main apache-rocketmq]# jps
4419 BrokerStartup
4636 Jps
4077 NamesrvStartup
```

发送接收消息

```
 > export NAMESRV_ADDR=localhost:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

停止服务

```
sh bin/mqshutdown broker
sh bin/mqshutdown namesrv
```

---

# 使用 RocketMQ Console 监控

在 Tomcat 中部署 RocketMQ Console，可以看到 RocketMQ 的运行情况。

[RocketMQ 扩展项目地址](https://github.com/apache/rocketmq-externals)将项目 clone 到本地，进入`rocketmq-console`，参考项目文档编译。

修改`application.properties`文件

> rocketmq.config.namesrvAddr= 192.168.0.222:9876

编译运行文件

> mvn clean package -Dmaven.test.skip=true
> java -jar target/rocketmq-console-ng-1.0.0.jar

在浏览器访问`192.168.0.222:8080`即可访问监控界面，就能看到 RocketMQ 的基本信息。

---

# 客户端访问

`NameService` 和 `BrokerService`已经启动，现在单机模式下用客户端来实现消息的订阅和发送。

## Producer

```java
public class TestProducer {
    public static void main(String[] args) throws MQClientException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("DefaultCluster");
        producer.setNamesrvAddr("192.168.0.222:9876");
        producer.start();

        for (int i = 0; i < 100; i++) {
            try {
                {
                    Message msg = new Message("TopicTest1",
                            "TagA",
                            "key113",
                            "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
                    SendResult sendResult = producer.send(msg);
                    System.out.printf("%s%n", sendResult);

                    QueryResult queryMessage =
                            producer.queryMessage("TopicTest1", "key113", 10, 0, System.currentTimeMillis());
                    for (MessageExt m : queryMessage.getMessageList()) {
                        System.out.printf("%s%n", m);
                    }
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        producer.shutdown();
    }
}
```

## Consumer

```java
public class PullConsumer {
    private static final Map<MessageQueue, Long> OFFSE_TABLE = new HashMap<MessageQueue, Long>();

    public static void main(String[] args) throws MQClientException {
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("DefaultCluster");
        consumer.setNamesrvAddr("192.168.0.222:9876");

        consumer.start();

        Set<MessageQueue> mqs = consumer.fetchSubscribeMessageQueues("TopicTest1");
        for (MessageQueue mq : mqs) {
            System.out.printf("Consume from the queue: %s%n", mq);
            SINGLE_MQ:
            while (true) {
                try {
                    PullResult pullResult =
                            consumer.pullBlockIfNotFound(mq, null, getMessageQueueOffset(mq), 32);
                    System.out.printf("%s%n", pullResult);
                    putMessageQueueOffset(mq, pullResult.getNextBeginOffset());
                    switch (pullResult.getPullStatus()) {
                        case FOUND:
                            break;
                        case NO_MATCHED_MSG:
                            break;
                        case NO_NEW_MSG:
                            break SINGLE_MQ;
                        case OFFSET_ILLEGAL:
                            break;
                        default:
                            break;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

        consumer.shutdown();
    }

    private static long getMessageQueueOffset(MessageQueue mq) {
        Long offset = OFFSE_TABLE.get(mq);
        if (offset != null)
            return offset;

        return 0;
    }

    private static void putMessageQueueOffset(MessageQueue mq, long offset) {
        OFFSE_TABLE.put(mq, offset);
    }

}
```

![集群信息](http://upload-images.jianshu.io/upload_images/2791079-927f52581cf3151d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从控制台可以看到集群的基本信息，包括消息的产生和消费数量等。

<div class="note info"><p> RocketMQ 单机模式跑通，接下来使用双 master 模式。</p></div>

[部署 RockerMQ 双 Master 模式](http://shuiyujie.com/2018/01/25/%E9%83%A8%E7%BD%B2-RockerMQ-%E5%8F%8C-Master-%E6%A8%A1%E5%BC%8F/#more)

---

# 参考资料

[Apache RocketMQ](https://rocketmq.apache.org/docs/quick-start/)
[Rocket-Externals](https://github.com/apache/rocketmq-externals/tree/master/rocketmq-console)
[安装rocketmq-console](http://www.cnblogs.com/quchunhui/p/7284752.html)
[Maven 升级](https://stackoverflow.com/questions/7532928/how-do-i-install-maven-with-yum)

