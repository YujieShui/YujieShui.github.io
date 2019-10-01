---
title: Ubuntu配置JDK
categories:
  - Java
tags:
  - java
  - linux
abbrlink: 1d66a96c
date: 2017-09-03 21:13:12
---

wget下载jdk8

```shell
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
```

解压

```shell
tar xzf jdk-8u141-linux-x64.tar.gz
```

配置环境变量

vim /etc/profile

```shell
JAVA_HOME=/home/shui/java/jdk1.8.0_141
JRE_HOME=/home/shui/java/jdk1.8.0_141/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

使配置生效

```shell
export JAVA_HOME JRE_HOME CLASS_PATH PATH

source /etc/profile
```

