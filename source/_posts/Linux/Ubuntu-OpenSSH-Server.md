---
title: Ubuntu OpenSSH Server
categories:
  - Linux
tags:
  - linux
  - ubuntu
abbrlink: cbbc9ca0
date: 2019-03-16 18:41:24
---

本文介绍如何使用`OpenSSH`实现计算机之间的远程控制和数据交换。你将了解到`OpenSSH`的一些配置以及如何在`Ubuntu`中修改这些配置。

`Ubuntu16.04`是目前`Ubuntu`较为稳定的版本，本文使用该版本进行说明。

<!-- more -->

# 介绍

`OpenSSH`基于[SSH协议](https://zh.wikipedia.org/zh-hans/Secure_Shell)，是用于实现计算机之间远程控制和数据交换的工作的工具。

传统的工具实现远程登录(telnet)和rcp等功能的方式不安全的，他们会用明文的方式交换用户密码。`OpenSSH`用后台进程和客户端工具来提高安全性，对远程控制和数据交换操作进行加密，比其他传统工具更加高效。

`OpenSSH`使用`sshd`持续地监听来自各个客户端程序的连接。当客户端发出连接请求，`ssh`根据客户端的连接类型来判断是否建立连接。比如，如果远程计算机是一个`ssh`客户端程序，`OpenSSH`将会在认证之后建立一个控制会话。如果远程用户使用`scp`进行连接，`OpenSSH`在认证之后会与客户端建立连接，并在后台初始化一个安全的文件拷贝。

`OpenSSH`可以使用密码、公钥和 [Kerberos](https://www.varonis.com/blog/kerberos-) 等多种方式进行认证。

# 安装

`OpenSSH`的安装非常简单，分别安装`OpenSSH Sever`和`OpenSSH Client`。

> sudo apt install openssh-client
> sudo apt install openssh-server

# 配置

`OpenSSH`的配置文件是`/etc/ssh/sshd_config`，查看详细配置可以使用

> man sshd_config

在修改配置文件之前我们应当对配置文件进行备份

> sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.original
> sudo chmod a-w /etc/ssh/sshd_config.original

我们可以通过修改配置文件做这些事情：

1. 将`OpenSSH`监听的默认TCP端口从2222改为默认端口22，可以修改`Port 2222`
2. 允许运行使用公式登录，可以使用`PubkeyAuthentication yes`

修改完成之后重启服务使其生效

> sudo systemctl restart sshd.service

# SSH Keys

配置了`ssh key`允许主机之间直接通信而不用输入密码。

首选生成`ssh key`，使用一下命令并一路回车

> ssh-keygen -t rsa

此时会在`~/.ssh`文件夹在生产一个密钥文件和一个公钥文件，我们将公钥拷贝给远程的主机

> ssh-copy-id username@remotehost

之后用相同的方式将远程主机的公钥拷贝给本机，就可以实现双方免密登录。

最后我们还要注意，认证用户需要对用于认证的文件有读写的权限。

> chmod 600 .ssh/authorized_keys

# 引用

[OpenSSH Server
](https://help.ubuntu.com/lts/serverguide/openssh-server.html.en)


