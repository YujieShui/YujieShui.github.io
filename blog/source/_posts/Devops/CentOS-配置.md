---
title: CentOS 7 配置
date: 2017-11-14 09:57:16
categories: 
- 运维
- Linux
tags: 
- linux
- centos
- jdk
- tomcat
- mysql
---

本文说明如何在 Parallels 虚拟机上安装 CentOS 7 以及对其的一些基本配置，几个常用配置文件的说明和 JDK, Tomcat 和 Mysql 的安装。

<!-- more -->

# CentOS 7 基本配置

## 安装 centos 7

首先下载镜像文件，我装的是 *CentOS-7-x86_64-Minimal-1708.iso*。我从网易的镜像下载，下载完后上传了一份到百度云盘：

> 网易镜像地址：http://mirrors.163.com/centos/7/isos/x86_64/
> 百度云盘链接:http://pan.baidu.com/s/1gf7w1uN  密码:5lbx

![第一步](http://upload-images.jianshu.io/upload_images/5430305-bb9bfa40b0c66271.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![第二步](http://upload-images.jianshu.io/upload_images/5430305-19fde3fa988edc5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**之后按照引导即可完成，有问题可以参看 [Mac利用PD虚拟机安装Centos7](http://www.jianshu.com/p/423ba6e48aaa)**。

![桥接模式](http://upload-images.jianshu.io/upload_images/5430305-70c6843b6887d7ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 更换 yum 源

yum 是一种包管理工具，就像 Java 中常用的 maven，或者安卓中用的 Gradle，再或者 Node.js 中的 npm。通过他我们能够统一地下载安装软件。但是由于网络原因下载缓慢，所以要把源换成国内的。更换过程如下：

备份：

> mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

去下面的源下载对应版本repo文件, 放入`/etc/yum.repos.d/`

repo文件下载地址：

> 网易:http://mirrors.163.com/.help/centos.html
> 阿里:http://mirrors.aliyun.com/repo/

运行以下命令生成缓存

> yum clean all
> yum makecache

**yum 源配置完成**

## 通过 yum 安装一些基本软件

net-tools 提供dig, nslookup, ipconfig等，用于配置网络：

> yum install net-tools

添加 wget 下载文件：

> yum install wget

## 创建一个普通用户并赋予 root 权限

用普通账号进行登录可以避免 root 用户进行错误操作，而且用普通用户登录就像给服务器建立了两道墙，必须先用普通用户登录再设置能用 root 账号登录，所以后面还要配置禁止 root 用户用过 SSH 登录。

创建普通用户

> useradd shui
> passwd shui
> 输入密码

这个普通用户有时也需要使用 root 权限，所以讲他加入到`sudoers` 用户组，允许其使用`sudo`临时调用 root 权限

> echo 'shui ALL=(ALL) ALL'>> /etc/sudoers
> tail -1 /etc/sudoers
> shui ALL=(ALL) ALL

## 禁止 root 使用 ssh 登入

进入配置文件：

> /etc/ssh/sshd_config

找到如下语句进行修改

> PermitRootLogin yes

把它改成

> PermitRootLogin no

重启 sshd

> systemctl restart sshd.service

这样别人就要必须要获取普通用户账号密码，然后才能破解 root

## 将防火墙换成 iptables

CentOS 7.0 默认使用的是 firewall 作为防火墙，常用的是 iptables。先关闭 firewall 再安装 iptables。

关闭 firewall

```
#停止firewall
systemctl stop firewalld.service 
#禁止firewall开机启动
systemctl disable firewalld.service 
```
安装 iptables

> yum -y install iptables-services

启动 iptables

```
#重启防火墙使配置生效
systemctl restart iptables.service 
#设置防火墙开机启动
systemctl enable iptables.service
```

## 关闭 SELinux

查看 SELinux 状态

> /usr/sbin/sestatus -v | grep SELinux
> SELinux status:  enabled # 表示为开启状态

永久关闭，重启生效

> vi /etc/selinux/config 
> \#修改内容如下
> SELINUX=disabled

## 更改系统语言

查看当前系统语言

```
localectl

System Locale: LANG=zh_CN.UTF-8
       VC Keymap: cn
      X11 Layout: cn
```

查看系统中存在的语言列表，因为很长通过 grep 来查找需要的语言是否存在

```
localectl list-locales | grep US

en_US
en_US.iso88591
en_US.iso885915
en_US.utf8
.
.
```

设置自己要改的语言

```
localectl set-locale LANG=en_US.UTF-8
```

## 参考：

[CentOS yum 源的配置与使用](http://www.jianshu.com/p/d8573f9d1f96)

[安裝 CentOS 7 後必做的七件事](https://www.hkpug.net/2014/09/18/%E5%AE%89%E8%A3%9D-centos-7-%E5%BE%8C%E5%BF%85%E5%81%9A%E7%9A%84%E4%B8%83%E4%BB%B6%E4%BA%8B/)

[Change system language in centos 7](https://my.9xhost.net/knowledgebase/article/33/change-system-language-in-centos-7/)

# 一些配置文件的修改

## 修改主机名

可以用`hostname`来查看你的主机名，修改主机名配置文件：

> vi /etc/hostname

重启生效

> reboot

## 修改 ip 地址

> vi /etc/sysconfig/network-scripts/ifcfg-eth0

在网卡中配置如下内容可以设置静态 ip

```
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.0.11
NETMASK=255.255.255.0
DNS1=192.168.0.1
DNS2=8.8.8.8
```

配置完之后保存重启网络

> service network restart

# 设置 ip 和 hostname 的映射

> vi /etc/hosts

添加上 ip 和 hostname 的键值对，举例前面设置`hostname`为 main 设置`ip`为 192.168.0.11 在最后追加一行：

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.11 main
```

之后可以用主机名进行访问

# 安装 JDK

将 JDK 安装包上传，上传可以使用 scp,rz 从本机上传，也可以直接下载。

解压 JDK 安装包：

> tar -zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local

配置环境变量：

```
vi /etc/profile

# 追加内容如下
export JAVA_HOME=/usr/local/jdk1.8.0_151
export PATH=$PATH:$JAVA_HOME/bin
```

加载环境变量：

> source /etc/profile

更多

[linux配置java环境变量(详细)](http://www.cnblogs.com/samcn/archive/2011/03/16/1986248.html)

# 安装 Tomcat

[点击此处下载安装包](https://tomcat.apache.org/download-80.cgi)。本文使用的版本是`apache-tomcat-8.5.23.tar.gz`，下载后再安装包传到 Linux 主机。

解压安装包：

> tar -zxvf apache-tomcat-8.5.23.tar.gz -C /usr/local/

启动 tomacat

> /usr/local/apache-tomcat-8.5.23/bin/startup.sh

查看 tomcat 进程

> ps -ef | grep tomcat

访问`http://192.168.2.224:8080/`即主机 ip + tomcat 端口。小猫出现表示成功。

# 安装 Mysql

Mysql 安装之前先留个心，安装过程中可能会需要记录密码，有些默认没有密码，请留心有没有提示记录密码，然而没看见也没关系就是要再折腾一下。本文演示用`yum`安装 mysql 数据库。

## 配置YUM源

```
# 下载mysql源安装包
shell> wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
# 安装mysql源
shell> yum localinstall mysql57-community-release-el7-8.noarch.rpm
# 检查mysql源是否安装成功
yum repolist enabled | grep "mysql.*-community.*"

# 成功显示如下
mysql-connectors-community/x86_64       MySQL Connectors Community            42
mysql-tools-community/x86_64            MySQL Tools Community                 51
mysql57-community/x86_64                MySQL 5.7 Community Server           227
```

## 安装MySQL

> yum install mysql-community-server

## 启动 mysql

> systemctl start mysqld

查看MySQL的启动状态

> systemctl status mysqld


## 修改 root 默认密码

mysql安装完成之后，在`/var/log/mysqld.log`文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：

> grep 'temporary password' /var/log/mysqld.log
> 2017-11-15T16:26:37.970235Z 1 [Note] A temporary password is generated for root@localhost: d54aqgZr69>d

此时默认的密码就是`d54aqgZr69>d`，用 root 身份登录之后修改密码：

> mysql -uroot -p
> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass1!';

*注：mysql 的密码需要有一定复杂度*

**更多请参考:**[CentOS7下安装MySQL5.7安装与配置（YUM）](http://www.centoscn.com/mysql/2016/0626/7537.html)

