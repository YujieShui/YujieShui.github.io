---
title: zookeeper的基本概念
toc: true
categories:
  - 分布式
tags:
  - zookeeper
abbrlink: e7973620
date: 2019-05-03 07:46:38
---

本文介绍 zookeeper 的一些基本概念。

<!-- more -->

# 数据模型

- 分层结构
- 树形结构中的每个节点叫做Znode
- 每个Znode都有数据ͧ(byte[]类型ͨ)，也可以有子节点
- 节点路径
  - 斜线分隔:/Zoo/Duck
  - 没有相对路径

- 通过数据结构stat来孓储数据的变化 ACL的变化和时间戳 
- 数据发生变化时，版本号会递增
- 可以对Znode中的数据进行读写操作

# 数据节点(Znode)

- 不是机器的意思
- Zk树形结构中的数据节点，用于存储数据
- 持久节点：一旦创建,除非主动调用删除操作,否则一直存储在zk上
- 临时节点：与客户端的会话绑定，一旦客户端绘画是小，这个客户端创建的所有临时节点都会被移除
- SEQUENTIAL Znode：创建临时节点时，如果设置属性 SEQUENTIAL，则会自动在节点名后面追加一个整型数字

# 持久节点

启动一个 zkCli 创建一个持久节点，即使创建节点的会话关闭，该节点也不会被删除。

```shell
[zk: localhost:2181(CONNECTED) 10] create /node nodedata
Created /node
[zk: localhost:2181(CONNECTED) 11] ls /
[node, zookeeper]
```

## 临时节点

启动一个 zkCli 之后创建临时节点

```shell
[zk: localhost:2181(CONNECTED) 3] create -e /node1 node1data
Created /node1
[zk: localhost:2181(CONNECTED) 4] ls /
[zookeeper, node1]
```

将创建临时节点的会话关闭之后，临时节点将会消失。(可以再起一个 zkCli 验证)

## 顺序节点

创建数据节点

```
[zk: localhost:2181(CONNECTED) 12] create -s /node/seq 321
Created /node/seq0000000000
[zk: localhost:2181(CONNECTED) 13] create -s /node/seq 322
Created /node/seq0000000001
[zk: localhost:2181(CONNECTED) 15] ls /node
[seq0000000000, seq0000000001]
```

# 集群角色

- Leader: 为客户端提供读和写服务
- Follower: 提供读服务，所有写服务都需要转交给Leader角色，参与选举
- Observer: 提供读服，不参与选举过程，一般是为了增强zk集群的读请求并发能力

# 会发(Session)

- Zk的客户与zk的服务端之间的连接
- 通过心跳检测保持客户端连接的存活
- 接收来自服务端的watch事件通知
- 可以设置超时时间

# 版本

- Version: 当前 Znode 的版本
- Cversion：当前 Znode 的子节点的版本
- Aversion：当前 Znode 的 ANL(访问控制)版本

# Watcher

- 作用于 Znode 节点上
- 多种事件通知：数据更新，子节点状态等

# ACL

- Access Control Lists
- 类似于linux/unix的权限控制
- **CREATE**：创建子节点的权限
- READ:获取节点数据和子节点列表的权限
- WRITE：更新节点数据的权限
- **DELETE**：删除子节点的权限
- ADMIN：设置节点ACL的权限

*CREATE和DELETE是针对子节点的权限控制。*

