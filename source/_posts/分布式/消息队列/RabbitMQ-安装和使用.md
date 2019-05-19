---
title: RabbitMQ 安装和使用
toc: true
categories:
  - 分布式
tags:
  - 消息队列
  - RabbitMQ
abbrlink: 29003c34
date: 2018-02-04 17:35:33
---

RabbitMQ 是一个消息队列，主要是用来实现应用程序的异步和解耦，同时也能起到消息缓冲，消息分发的作用。本文介绍RabbitMQ 安装和使用。

<!-- more -->

RabbitMQ 是一个开源的`AMQP`实现，服务器端用`Erlang`语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

可以把消息队列想象成邮局，你的笔友把信件投递到邮局，邮递员源源不断地进出邮局，把笔友的信送到你的手里。此时的笔友就是一个生产者（Product）,邮递员一次送信就是（Queue）,而你收信就像是消费者（Consumer）。

## AMQP

AMQP（Advanced Message Queuing Protocol，高级消息队列协议）的原始用途只是为金融界提供一个可以彼此协作的消息协议，而现在的目标则是为通用消息队列架构提供通用构建工具。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向**消息、队列、路由**（包括点对点和发布/订阅）、可靠性、安全。

RabbitMQ 则是一个开源的 AMQP 实现。

## Rabbit 概念

通常我们谈到队列服务, 会有三个概念： 发消息者、队列、收消息者，RabbitMQ 在这个基本概念之上, 多做了一层抽象, 在发消息者和 队列之间, 加入了交换器 (Exchange)。这样发消息者和队列就没有直接联系, 转而变成发消息者把消息给交换器, 交换器根据调度策略再把消息再给队列。

通过 [RabbitMQ 官网](https://www.rabbitmq.com/getstarted.html) 的示例中看到 RabbitMQ 有六种模式。

![RabbitMQ 六种模式](http://upload-images.jianshu.io/upload_images/2791079-4ea9ba91d680f1a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

官网中有多种语言的实现，本文用 Java 来实现。采用 Springboot 集成 RabbitMQ。

----

# CentOS 安装 RabbitMQ

## 安装 Erlang、Elixir

### 准备

> yum update
> 
> yum install epel-release
> 
> yum install gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git wget wxBase.x86_64

### 安装 Erlang

> wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
> 
> rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
> 
>  yum update
> 
> yum install erlang

验证是否安装成功，输入命令：`erl`

### 安装 Elixir

因为 EPEL 中的 Elixir 版本太老，所以下面是通过源码编译安装的过程：

> git clone https://github.com/elixir-lang/elixir.git
> 
> cd elixir/
> 
> make clean test
> 
> export PATH="$PATH:/usr/local/elixir/bin"

验证是否安装成功，输入命令：`iex`

## 安装 RabbitMQ

> wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
> 
> rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
> 
> yum install rabbitmq-server-3.6.1-1.noarch.rpm

## Rabitmq 管理

至此已经安装完成，下面介绍启动和自动开机启动命令和配置

启动：

> systemctl start rabbitmq-server

开机自动启动：

> systemctl enable rabbitmq-server

查看 rabbitmq-server 状态：

>rabbitmqctl status

关闭：

> systemctl enable rabbitmq-server

可以直接通过配置文件的访问进行管理，也可以通过Web的访问进行管理。

通过Web进行管理,开启 Web 管理:

> rabbitmq-plugins enable rabbitmq_management
> 
> chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/

*注：先启动 RabbitMQ*

访问：`http://192.168.2.223:15672/`，默认用户 guest ，密码 guest。

发现登录失败，由于账号guest具有所有的操作权限，并且又是默认账号，出于安全因素的考虑，guest用户只能通过localhost登陆使用。

我们新增一个用户：

> rabbitmqctl add_user admin  123456
> 
> rabbitmqctl set_user_tags admin administrator
> 
> rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

![RabbitMQ 管理界面](http://upload-images.jianshu.io/upload_images/2791079-a0eac20af580be37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

# Springboot 集成 RabbitMQ

假设现在已经按照前面的步骤完成了 RabbitMQ 的安装，现在开始使用 Springboot 集成 RabbitMQ。

## 基本配置

IDEA 先新建一个 maven 项目，在 pom 文件中添加相关依赖:

### pom 文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.shuiyujie</groupId>
    <artifactId>pom</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>pom</name>

    <!-- Spring Boot 启动父依赖 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Test 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- rabbitmq -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

    </dependencies>
</project>
```

### application.properties

```
# rabbitmq 配置文件
spring.rabbitmq.host=192.168.0.223
# 默认端口
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123456
```

## "Hello World"

![HellowWorld.png](http://upload-images.jianshu.io/upload_images/2791079-16ad9e3ac29862a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我们的目标很简单就是创建一个生产者 P，和一个消费者 C，同时将 P 产生的消息放到队列中供 C 使用。

**Queue**

```
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {
    @Bean
    public Queue helloQueue() {
        return new Queue("hello");
    }
}
```

**HelloSender**

```java
@Controller
public class HelloSender {

    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send() {
        String context = "hello " + new Date();
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("hello", context);
    }
}
```

**HelloReceiver**

```java
@Component
public class HelloReceiver {
    @RabbitHandler
    @RabbitListener(queues = "hello")
    public void process(String hello) {
        System.out.println("Receiver  : " + hello);
    }
}
```

**运行**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = HelloApplication.class)
public class RabbitmqApplicationTests {

    @Autowired
    private HelloSender helloSender;

    @Test
    public void hello() throws Exception {
        helloSender.send();
    }
}
```

**成功接收到消息**

```
Receiver  : hello Thu Feb 01 22:21:39 CST 2018
```

**注意：`HelloReceiver`的`@RabbitListener(queues = "hello")`注解是方法级的，参照别的文章都是类级别的注解导致一直无法正常连接。**

## Work Queues

![Work Queues.png](http://upload-images.jianshu.io/upload_images/2791079-39907ff7eae4effb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Work Queues` 模式在原来的基础上多增加了一个消费者。同理我们可以扩展三个、四个甚至更多的`consumer`。这样做的好处在于，当我们使用一个`consumer`的时候，当它收到一条消息进行处理的时候会发生阻塞。有多个`consumer`时，消息就可以分发给空闲的`consumer`进行处理。

**生产者**

```java
/**
 * Work 模式下的生产者
 * 
 * @author shui
 * @create 2018-02-04
 **/
@Controller
public class WorkSender {

    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send(int i) {
        String context = "work ";
        System.out.println("Sender : " + context + "*****" + i);
        this.rabbitTemplate.convertAndSend("work", context);
    }
}
```

**Queue**

```java
@Configuration
public class WorkConfig {
    @Bean
    public Queue workQueue() {
        return new Queue("work");
    }
}
```

**两个消费者**

```java
@Component
public class WorkReceicer1 {
    @RabbitHandler
    @RabbitListener(queues = "work")
    public void process(String message) {
        System.out.println("Work Receiver1  : " + message);
    }
}

@Component
public class WorkReceicer2 {
    @RabbitHandler
    @RabbitListener(queues = "work")
    public void process(String message) {
        System.out.println("Work Receiver2  : " + message);
    }
}
```

**测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Startup.class)
public class RabbitMQDirectTest {

    @Autowired
    private WorkSender workSender;

    @Test
    public void sendWorkTest() {
        for (int i = 0; i < 20; i++) {
            workSender.send(i);
        }
    }
}
```

**结果**

```
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver2  : work 
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver1  : work 
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver2  : work 
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver1  : work 
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver1  : work 
Work Receiver2  : work 
Work Receiver2  : work 
Work Receiver1  : work 
```

*发现消费得很平均，每个`consumer`处理一半的消息。*

## public/subscribe

![public/subscribe.png](http://upload-images.jianshu.io/upload_images/2791079-4026a587dd2a63ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面的两个例子我们看到`producer`产生的消息直接发送给`queue`，然后`queue`又直接将消息传给`consumer`。RabbitMQ 的亮点就在于改变了上面这种消息传递的方式，`producer`不会将消息直接传给`queue`而是传给`exchanges`再由`exchangers`传给`queue`。然而我们在前面的两个例子中并没有使用`exchanges`，那是因为 RabbitMQ 有默认的`exchanges`，只要我们传的参数是`""`。在默认模式下，不需要将`exchanges`做任何绑定。除此之外`exchanges`有以下几种类型：

> 1. Direct：direct 类型的行为是”先匹配, 再投送”. 即在绑定时设定一个 routing_key, 消息的 routing_key 匹配时, 才会被交换器投送到绑定的队列中去.
> 2. Topic：按规则转发消息（最灵活）
> 3. Headers：设置header attribute参数类型的交换机
> 4. Fanout：转发消息到所有绑定队列

**Queue**

以下使用的是`Fanout Exchange`转发消息到所有绑定队列。这里要配置两个`queue`，并且配置`exchanges`，并把`queue`和`exchanges`绑定。

```java
/**
 *
 * public/subscribe 模式
 *
 * @author shui
 * @create 2018-02-04
 **/
@Configuration
public class FanoutConfig {

    /************************************************************************
     * 新建队列 fanout.A 、fanout.B
************************************************************************/

    @Bean
    public Queue AMessage() {
        return new Queue("fanout.A");
    }

    @Bean
    public Queue BMessage() {
        return new Queue("fanout.B");
    }

    /**
     * 建立一个交换机
     *
     * @return
     */
    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }

    /************************************************************************
     * 将 fanout.A 、 fanout.B 绑定到交换机 fanoutExchange 上
************************************************************************/

    @Bean
    Binding bindingExchangeA(Queue AMessage, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(AMessage).to(fanoutExchange);
    }

    @Bean
    Binding bindingExchangeB(Queue BMessage, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(BMessage).to(fanoutExchange);
    }
}
```

**生产者**

在创建`producter`的时候，要将他和`exchanges`绑定。

```java
@Controller
public class FanoutSender {
    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send() {
        String context = "hi, fanout msg ";
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("fanoutExchange","", context);
    }
}
```

**两个消费者**

```java
@Component
public class FanoutReceiveA {
    @RabbitHandler
    @RabbitListener(queues = "fanout.A")
    public void process(String message) {
        System.out.println("fanout Receiver A  : " + message);
    }
}

@Component
public class FanoutReceiveB {

    @RabbitHandler
    @RabbitListener(queues = "fanout.B")
    public void process(String message) {
        System.out.println("fanout Receiver B  : " + message);
    }
}
```

**测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Startup.class)
public class FanoutTest {
    @Autowired
    private FanoutSender fanoutSender;

    @Test
    public void setFanoutSender() {
        fanoutSender.send();
    }
}
```

**结果**

```
fanout Receiver B  : hi, fanout msg 
fanout Receiver A  : hi, fanout msg 
```

## Routing

![routing.png](http://upload-images.jianshu.io/upload_images/2791079-a7a629c3ddd452a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在前面的`Fanout`模式下，消息会直接**广播**给`queue`。如果我们想让`consumer`处理某些特定的消息，就要让他接收消息的队列中没有其他类型的消息，所以能不能让`queue`只接收某些消息，而不接收另一些消息呢？

RabbitMQ 中有一个 Routingkey 的概念。在队列与交换机的绑定过程中添加`Routingkey`表示`queue`接收的消息需要带有`Routingkey`。

## Topic

![topic.png](http://upload-images.jianshu.io/upload_images/2791079-e6aa0a7f74c58d9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Topic`模式和`Direct`模式类似，`Direct`模式需要`Routingkey`完全匹配而`Topic`模式更加灵活，可以通过通配符进行配置。

> 1. 在这种交换机模式下：路由键必须是一串字符，用句号（.） 隔开，例如：topic.A
> 2. 路由模式必须包含一个星号`*`，主要用于匹配路由键指定位置的一个单词，比如说，一个路由模式是这样子：agreements..b.*，那么就只能匹配路由键是这样子的：第一个单词是 agreements，第四个单词是 b。 井号（#）就表示相当于一个或者多个单词；例如一个匹配模式是agreements.eu.berlin.#，那么，以agreements.eu.berlin开头的路由键都是可以的。

**Queue and exchange**

另个队列分别为 topic.A,topic.B,将他们绑定到 topicExchange 上。并且设置了规则，topic.A 必须是完全匹配的也就是`Direct`模式，topic.B 使用`Topic`模式，只要是`Rouctingkey`为 topic 开头的都可以接收。

```java
@Configuration
public class TopicConfig {

    final static String message = "topic.A";
    final static String messages = "topic.B";

    @Bean
    public Queue queueMessage() {
        return new Queue(TopicConfig.message);
    }

    @Bean
    public Queue queueMessages() {
        return new Queue(TopicConfig.messages);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("topicExchange");
    }

    @Bean
    Binding bindingExchangeMessage(Queue queueMessage, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessage).to(exchange).with("topic.message");
    }

    @Bean
    Binding bindingExchangeMessages(Queue queueMessages, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessages).to(exchange).with("topic.#");
    }
}
```

**生产者**

```java
@Controller
public class TopicSend {
    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send() {
        String context = "hi, i am message 0";
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("topicExchange", "topic.1", context);
    }

    public void send1() {
        String context = "hi, i am message 1";
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("topicExchange", "topic.message", context);
    }

    public void send2() {
        String context = "hi, i am messages 2";
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("topicExchange", "topic.messages", context);
    }
}
```

**消费者**

```java
@Component
@RabbitListener(queues = "topic.A")
public class TopicReceiver {
    @RabbitHandler
    public void process(String message) {
        System.out.println("Topic Receiver1  : " + message);
    }
}

@Component
@RabbitListener(queues = "topic.B")
public class TopicReceiver2 {
    @RabbitHandler
    public void process(String message) {
        System.out.println("Topic Receiver2  : " + message);
    }
}
```

**测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Startup.class)
public class TopicTest {
    @Autowired
    private TopicSend sender;

    @Test
    public void topic() throws Exception {
        sender.send();
    }

    @Test
    public void topic1() throws Exception {
        sender.send1();
    }

    @Test
    public void topic2() throws Exception {
        sender.send2();
    }
}
```

**结果**

```
Sender : hi, i am message 1
Sender : hi, i am messages 2
Sender : hi, i am message 0
Topic Receiver1  : hi, i am message 1
Topic Receiver2  : hi, i am message 1
Topic Receiver2  : hi, i am messages 2
Topic Receiver2  : hi, i am message 0
```

# 总结

掌握 RabbitMQ 的核心在于如何使用好`exchanges`，它有默认模式`""` , `direct` , `topic` , `headers` 和 `fanout` 这几种模式。

通过 RabbitMQ 的 `routingkey` 可以过滤交换机传递给队列的消息。`fanout` 模式下，需要队列和交换机的`routingkey`完全匹配，而在`topic`模式下，可以通过通配符进行配置，变得更加灵活。

## 安装参考：

[Install RabbitMQ server in CentOS 7](https://www.unixmen.com/install-rabbitmq-server-centos-7/)

[CentOS 7 下安装 RabbitMQ
](http://www.cnblogs.com/xueweihan/p/7099641.html)

[Install Erlang and Elixir in CentOS 7](https://www.unixmen.com/install-erlang-elixir-centos-7/)

[rabbitmq——用户管理](https://my.oschina.net/hncscwc/blog/262246)

## Springboot 集成 RabbitMQ 参考

[RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)

[Spring Boot 中使用 RabbitMQ](https://juejin.im/post/59f194e06fb9a0451329ec53)

[springboot rabbitmq整合](https://www.cnblogs.com/xmzJava/p/8036591.html)

[Spring Boot系列(八)：RabbitMQ详解](https://zhuanlan.zhihu.com/p/25069044?refer=dreawer)

