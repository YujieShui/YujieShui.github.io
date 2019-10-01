---
title: 在ubuntu上安装maven
categories:
  - Java
tags:
  - maven
  - ubuntu
abbrlink: 9ec2f732
date: 2018-09-19 22:04:24
---

本文介绍如何在 ubuntu16.04 中安装 maven。

<!-- more -->

在 [Maven 官网](https://maven.apache.org/)下载安装包，然后解压。

解压之后拷贝到到 `/opt` 目录下，并建立软链接：

```shell
mv apache-maven-3.3.9 /opt
ln -s apache-maven-3.3.9 maven
```

配置环境变量

```shell
vim ~/.bashrc

export M2_HOME=/opt/maven
export M2=$M2_HOME/bin
export PATH=$M2:$PATH

source ~/.bashrc
```

测试安装是否成功

```shell
mvn -v
```

修改 maven 的源

```shell
cp /opt/maven/conf/settings.xml ~/.m2
vim ~/.m2/settings.xml
```

在 `<mirrors>`中进行如下配置

```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror> 
```

