---
title: Protobuf 编译 Java
categories:
  - Java
tags:
  - java
  - protobuf
abbrlink: 7e6ba807
date: 2018-01-28 11:06:34
---

Google Protocol Buffers 简称 Protobuf，它提供了一种灵活、高效、自动序列化结构数据的机制，可以联想 XML，但是比 XML 更小、更快、更简单。仅需要自定义一次你所需的数据格式，然后用户就可以使用 Protobuf 编译器自动生成各种语言的源码，方便的读写用户自定义的格式化的数据。与语言无关，与平台无关，还可以在不破坏原数据格式的基础上，依据老的数据格式，更新现有的数据格式。

<!-- more -->

官方下载地址：
> https://github.com/google/protobuf/releases/tag/v2.6.1

protobuf-2.6.1 百度云盘：
>链接:http://pan.baidu.com/s/1hrK7m7a  密码:epnb

**安装过程**

* 解压文件
> unzip protobuf-2.6.1.zip

* 进入目录
> cd protobuf-2.6.1

* 自定义编译目录
> ./configure --prefix= /Users/shui/company/protobuf

* 安装
>  make
>  mak install

* 配置环境变量
> vim .bash_profile

* 添加如下配置
> export PROTOBUF=/Users/shui/company/protobuf
> export PATH=$PROTOBUF/bin:$PATH```

* 使配置生效
> source .bash_profile

* 查看是否生效
> protoc --version

**安装结束，进行编译**

* 指定需要进行编译的 proto 文件
* 指定编译产生的 java 文件存放的位置

假设文件名为 file.proto,文件放置在 filedir 目录下；产生的java文件也位于 filedir 目录下，编译命令如下：

> protoc /filedir/file/proto --java_out=/filedir

如果有很多需要编译的 proto 文件，可以进入项目目录

>protoc *.proto --java_out=/filedir

**参考：**

> https://github.com/google/protobuf/tree/master/java
> http://blog.csdn.net/u013045971/article/details/50592998
> http://my.oschina.NET/KingPan/blog/283881?fromerr=8vajR5S9

