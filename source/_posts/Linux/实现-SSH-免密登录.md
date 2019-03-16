---
title: 实现 SSH 免密登录
categories:
  - Linux
tags:
  - linux
  - ssh
  - 分布式
abbrlink: ea055f0d
date: 2017-12-19 11:57:18
---

SSH (Secure Shell的) 是一种网络协议，用于计算机之间的加密登录。

通过使用SSH，你可以把所有传输的数据进行加密更加安全可靠。使用SSH，还有一个额外的好处就是传输的数据是经过压缩的，所以可以加快传输的速度。SSH 有很多功能，它既可以代替 Telnet，又可以为FTP、Pop、甚至为 PPP 提供一个安全的"通道"。

![SSH](http://upload-images.jianshu.io/upload_images/2791079-879bf1f790d974bc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

主机之间通过 SSH 进行连接的时候需要输入密码进行校验。在部署分布式应用时，主机间要建立良好的通信，首先要做的就是配置 SSH 免密登录。

*以下演示在`centos 7`中配置免密登录。*

# 配置主机间的免密登录

## 配置免密登录

什么是免密登录呢？

通常我们登录 SSH 是通过账号和免密来登录的，输入`ssh username@ip-server`然后输入密码。

如果每次都输入密码会很麻烦，而且要对多台主机进行自动化管理，每次都要输入密码不现实。我们可以配置公钥和密钥进行免密登录。**免密登录做的事情其实就是通过 SSH 的公钥和密钥来校验身份信息。**

首先你要知道每台主机有一份公钥和一份私钥。我们要做的事情可以用一张图来表示：

![ssh 免密登录](http://upload-images.jianshu.io/upload_images/2791079-5f9ccb0e655e4206.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.生成密匙对

> ssh-keygen

之后可以在`/root/.ssh`中看到生成的密匙对

```
[root@main /]# cd /root/.ssh/
[root@main .ssh]# ls
id_rsa id_rsa.pub
```
2.拷贝一份 A 的公钥给 B

> ssh-copy-id 192.168.0.10
> 输入密码

此时在 B 的`authorized_keys`中就会有一份 A 的`id_rsa.pub`公钥信息。

注：第二步操作的做的事情其实就是一个拷贝密钥的工作，也可以手动拷贝，但是用上面的命令更方便。

3.最后我们就可以免密登录,也就是不输入密码 A 就可以登录 B

> ssh root@192.168.0.10
 
*192.168.0.10 为 B 的 ip 地址*

如果要退出登录，输入`exit`即可。

## 给主机起一个别名

192.168.0.10 是 ip 地址，也就是说登录的时候我们还要输入一次 ip。我们可以给每个主机配置一个别名，用`ssh ip-server`的方式登录。

就像人有身份证也有名字一样，我们可以通过 ip 来辨识主机。给他一个别名就是给一个`hostname`。

可以用`hostname`来查看你的主机名，要改主机名改他的配置文件

> vi /etc/hostname
> 在其中替换为你要改的名字，比如 main

重启生效

> reboot

这样主机名已经改掉了，还差一步。我们要让主机名和我们的 ip 关联在一起，修改`/etc/hosts`文件

> vi /etc/hosts
> 添加上 ip 和 hostname 的键值对

例如：

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.11 main
192.168.0.10 slave
```

两边都配置完成可以用`ssh slave`直接连接`slave`。如果你想自己免密连接自己那就按照上面的步骤给自己配置一份密匙就行了，动手试试吧。

# SSH 协议是什么以及更多参考

[SSH 协议介绍](http://blog.csdn.net/macrossdzh/article/details/5691924)

[数字签名是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

[SSH原理与运用（二）：远程操作与端口转发](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html
)

[如何在CentOS 7上修改主机名](http://www.jianshu.com/p/39d7000dfa47)


