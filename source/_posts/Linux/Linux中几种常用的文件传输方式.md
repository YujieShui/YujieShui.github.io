---
title: Linux中几种常用的文件传输方式
categories:
  - Linux
tags:
  - linux
  - 文件传输
abbrlink: a6fc1711
date: 2018-01-21 20:14:35
---

整理`scp`,`wget`,`ftp`以及其他Linux中常用的文件传输方式的介绍。

<!-- more -->

# scp

当没有安装`web server`和`ftp server`的时候或感觉上面的方法比较麻烦，那么用`scp`命令就会排上用场。

## scp是什么？

`scp`是secure copy的简写，用于在Linux下进行远程拷贝文件的命令，和它类似的命令有`cp`，不过`cp`只是在本机进行拷贝不能跨服务器，而且`scp`传输是加密的。可能会稍微影响一下速度。

## scp有什么用？

1、我们需要获得远程服务器上的某个文件，远程服务器既没有配置ftp服务器，没有开启web服务器，也没有做共享，无法通过常规途径获得文件时，只需要通过scp命令便可轻松的达到目的。

2、我们需要将本机上的文件上传到远程服务器上，远程服务器没有开启ftp服务器或共享，无法通过常规途径上传是，只需要通过scp命令便可以轻松的达到目的。

## scp使用方法

**获取远程服务器上的文件**

> scp root@192.168.0.223:/apps/test/a.md /Users/shui/Desktop/b.md

- root@192.168.0.223 : 使用root用户登录服务器192.168.0.223

- /apps/test/a.md:本机要传递过去的文件

- /Users/shui/Desktop/b.md：远程主机目录

**获取远程服务器上的目录**

> scp -r root@192.168.0.223:/apps/test/a /Users/shui/Desktop/b

**将本地文件上传到服务器上**

> scp test.txt root@192.168.0.223:/apps/test/a.txt

- root@192.168.0.223 : 使用root用户登录远程服务器192.168.0.223
- test.txt:本机要传递过去的文件
- /apps/test/a.txt：远程主机目录和文件名


**将本地目录上传到服务器上**

> scp -r dir1 root@192.168.0.223:/apps/test/a

**当 scp 指定端口时**

> scp -P 2222 -r dir1 root@192.168.0.223:/apps/test/a

**可能有用的几个参数**

-v 和大多数 linux 命令中的 -v 意思一样 , 用来显示进度 . 可以用来查看连接 , 认证 , 或是配置错误 .
-C 使能压缩选项 .
-4 强行使用 IPV4 地址 .
-6 强行使用 IPV6 地址 .

参考：[scp 命令](http://www.runoob.com/linux/linux-comm-scp.html)

---

# Wget

> Linux系统中的wget是一个下载文件的工具，它用在命令行下。对于Linux用户是必不可少的工具，我们经常要下载一些软件或从远程服务器恢复备份到本地服务器。wget支持HTTP，HTTPS和FTP协议，可以使用HTTP代理。所谓的自动下载是指，wget可以在用户退出系统的之后在后台执行。这意味这你可以登录系统，启动一个wget下载任务，然后退出系统，wget将在后台执行直到任务完成，相对于其它大部分浏览器在下载大量数据时需要用户一直的参与，这省去了极大的麻烦。

> wget 可以跟踪HTML页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构。这又常被称作”递归下载”。在递归下载的时候，wget 遵循Robot Exclusion标准(/robots.txt). wget可以在下载的同时，将链接转换成指向本地文件，以方便离线浏览。

> wget 非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性.如果是由于网络的原因下载失败，wget会不断的尝试，直到整个文件下载完毕。如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。这对从那些限定了链接时间的服务器上下载大文件非常有用。

**下载单个文件**

> wget http://www.minjieren.com/wordpress-3.1-zh_CN.zip

**下载并以不同的文件名保存**

> wget -O tomcat.tar.gz http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.24/bin/apache-tomcat-8.5.24.tar.gz

**后台下载** 

> wget -b http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz

参考：[每天一个linux命令（61）：wget命令](http://www.cnblogs.com/peida/archive/2013/03/18/2965369.html)

---

# Ftp

## 功能

> ftp 命令使用文件传输协议（File Transfer Protocol, FTP）在本地主机和远程主机之间或者在两个远程主机之间进行文件传输。

> FTP 协议允许数据在不同文件系统的主机之间传输。尽管这个协议在传输数据上提供了高适应性，但是它并没有尝试去保留一个特定文件系统上的文件属性（例如一个文件的保护模式或者修改次数）。而且 FTP 协议很少对一个文件系统的整体结构作假定，也不提供这样的功能，比如递归的拷贝子目录。在使用 ftp 命令时，需要注意 FTP 协议的这些特性。当需要保留文件属性或者需要递归的拷贝子目录时，可以使用 rcp/scp 等命令。

**语法**

> ftp [-dignv][主机名称或IP地址]

**参数：**

-d 详细显示指令执行过程，便于排错或分析程序执行的情形。
-i 关闭互动模式，不询问任何问题。
-g 关闭本地主机文件名称支持特殊字符的扩充特性。
-n 不使用自动登陆。
-v 显示指令执行过程。

[Linux ftp命令](http://www.runoob.com/linux/linux-comm-ftp.html)

---

# 其他

摘自：[Linux 上的常用文件传输方式介绍与比较](https://www.ibm.com/developerworks/cn/linux/l-cn-filetransfer/)

**综上所述**
各种文件传输方式的特征表现各有千秋，我们从以下几个方面综合对比，更深入地了解它们各自的特性。

**传输性能**
wget 通过支持后台执行及断点续传提高文件传输效率 ； rsync 则以其高效的传输及压缩算法达到快传输的目的。

**配置难度**
rcp 只需进行简单的配置，创建 .rhost 文件以及设置 /etc/hosts 文件中主机名与 IP 地址列表； wget 设置设置方便简单，只需在客户端指定参数执行命令即可； rsync 在使用前需要对服务端 /etc/rsyncd.conf 进行参数设定，配置内容相对复杂。

**安全性能**
ftp、rcp 不保证传输的安全性，scp、rsync 则均可基于 ssh 认证进行传输，提供了较强的安全保障。 wget 也可通过指定安全协议做到安全传输。

**通过上述的对比不难发现，每种文件传输方法基于其自身的特点与优势均有其典型的适用场景：**

- ftp 作为最常用的入门式的文件传输方法，使用简单，易于理解，并且可以实现脚本自动化；
- rcp 相对于 ftp 可以保留文件属性并可递归的拷贝子目录；
- scp 利用 ssh 传输数据，并使用与 ssh 相同的认证模式，相对于 rcp 提供更强的安全保障；
- wget，实现递归下载，可跟踪 HTML 页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构，适合实现远程网站的镜像；
- curl 则适合用来进行自动的文件传输或操作序列，是一个很好的模拟用户在网页浏览器上的行为的工具；
- rsync 更适用于大数据量的每日同步，拷贝的速度很快，相对 wget 来说速度快且安全高效。

