---
title: zookeepre源码编译
toc: true
categories:
  - 分布式
tags:
  - zookeeper
abbrlink: 5ddebf1a
date: 2019-05-03 07:46:39
---

我们首先 fork 一份 zookeeper 在 github 上的项目，地址为：https://github.com/apache/zookeeper。再将 fork 之后的项目 clone 到本地。

我们不使用最新版本的代码，而是使用 3.4.11 这个版本的，所以我们根据 tag，checkout 一个分支出来：

```shell
git checkout -b zookeeper3.4.11 release-3.4.11
```

由于 zookeeper 是用 ant 构建的，所以要安装 ant。我使用的是 macOS，直接用 homebrew 安装比较方便：

```shell
brew update
brew install ant
```

之后用 ant 构建一个 eclipse 项目

```shell
ant eclipse
```

最后，将导入项目。