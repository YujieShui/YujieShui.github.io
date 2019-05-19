---
title: Linux 权限
toc: true
categories:
  - Linux
tags: linux
abbrlink: 40efde90
date: 2017-11-13 19:50:41
---

Linux 是多用户的操作系统，同一时间可以有多个用户同时操作同一台计算机。为了让用户和用户之间不相互影响，必须要有一种机制来保障每一个用户的行为不会越界对其他用户造成不必要的影响。

<!-- more -->

# 用户和用户组

了解 Linux 的权限管理首先要了解用户和组的概念。Linux 引入了用户和用户组的概念，一个用户可以拥有多个文件和目录， 用户对这个文件或目录的访问权限拥有控制权。同时用户可以属于一个或者多个用户组，用户组中的用户拥有**同属于这个组的**对文件的访问控制权。

Linux 会给每个用户分配一个 uid（标识用户） 和 gid (标识用户组)，在创建用户的时候默认就会创建一个和用户名同名的用户组，我可以用`id`命令来查看：

>id
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

这些信息都是配置在配置文件中的，我们有必要了解这些信息在哪些配置文件中,他们主要配置在三个文件中`/etc/shadow`,`/etc/passwd`和`/etc/group`。

> 用户帐户 定义在/etc/passwd 文件里面，用户组定义在/etc/group 文件里面。当用户帐户和用户组创建以后， 这些文件随着文件/etc/shadow 的变动而修改，文件/etc/shadow 包含了关于用户密码的信息。 对于每个用户帐号，文件/etc/passwd 定义了用户（登录）名、uid、gid、帐号的真实姓名、家目录 和登录 shell。如果你查看一下文件/etc/passwd 和文件/etc/group 的内容，你会注意到除了普通 用户帐号之外，还有超级用户（uid 0）帐号，和各种各样的系统用户。

# 基本的用户管理

## 创建一个普通用户

添加用户

> useradd  shui

设置密码

> passwd shui  按提示输入密码即可

这样就会创建一个新的用户，我们可以在`/etc/group`中看到新建的默认用户组的信息`shui:x:1000:`也可以再`/etc/shadow`,`/etc/passwd`看到相关信息。

## 为用户配置sudo权限

配置sudo权限就是将永不加入到`sudoers`用户组中，这是一个特殊的用户组，在这个用户组中的用户可以通过在命令前面添加`sudo`临时获取`root`的权限。

用root编辑 vi /etc/sudoers

在文件的如下位置，为hadoop添加一行即可

> root    ALL=(ALL)       ALL     
shui  ALL=(ALL)       ALL

或者使用如下语句也是相同效果

> echo 'shui ALL=(ALL) ALL'>> /etc/sudoers

*然后，hadoop用户就可以用sudo来执行系统级别的指令*

# 文件权限的操作

前面说过用户具有对文件的的访问控制权，访问控制权具有**可读、可写、可操作**三种，我们来具体看一下

## linux 文件权限的描述格式解读

用`ll`命令来看一下根目录下的文件信息,我们复制其中的一部分：

```
dr-xr-xr-x.   5 root root 4096 11月  2 10:34 boot
drwxr-xr-x.  20 root root 3140 11月  2 10:36 dev
drwxr-xr-x.  74 root root 8192 11月  2 20:48 etc
drwxr-xr-x.   3 root root   18 11月  2 19:03 home
```

**每个文件的信息大同小异，关注最前面的十个字符，他就表示文件的操作权限。**

以/etc目录的信息为例`drwxr-xr-x.  74 root root 8192 11月  2 20:48 etc`

第一位表示文件类型,d 表示这是一个文件夹，文件属性还有以下内容

![文件类型](http://upload-images.jianshu.io/upload_images/5430305-a18c76f78238c675.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

后面的九位分别代表用户权限、用户组权限和其他用户权限

![权限属性](http://upload-images.jianshu.io/upload_images/5430305-7f8a457aae28df46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- r:表示可读 read
- w:表示可惜 write
- x:表示可执行 excute

/etc 目录的权限 `rwxr-xr-x` 就表示，root 用户具有可读可写可操作的权限，文件所有者的组成员可以访问该目录，但是不能新建、重命名、删除文件，其他成员可以访问该目录，但是不能新建、重命名、删除文件。

![权限表](http://upload-images.jianshu.io/upload_images/5430305-51db58cc1b768d91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# chmod 更改权限

只有`root`和文件的所有者才能更改文件的权限，更改文件的权限有两种方式，一种是使用符号，另一种是使用八进制数字。

## 符号方式修改文件权限

符号即前面`rwx`对应的含义，`ugoa`则分别表示用户(user)、用户组(group)、其他人(other)和所有(all),一组示例来演示：

```
chmod g-rw haha.dat    表示将haha.dat对所属组的rw权限取消
chmod o-rw haha.dat 	表示将haha.dat对其他人的rw权限取消
chmod u+x haha.dat      表示将haha.dat对所属用户的权限增加x
```

## 八进制的方式来修改权限

一个八进制数字可以表示三个二进制数，二进制的 111 对应八进制的 7，而 111 正好可以表示 rwx 的含义，1 则代表有权限 0 则代表没有权限。

| 八进制 | 二进制 | 符号 |
| --- | --- | --- |
| 0 | 000 | --- |
| 1 | 001 | —-x |
| 2 | 010 | -w- |
| 3 | 011 | -wx |
| 4 | 100 | r-- |
| 5 | 101 | r-x |
| 6 | 110 | rw- |
| 7 | 111 | rwx |


例如：

> chmod 664 haha.dat   
> 修改成   rw-rw-r--

一些人喜欢使用八进制表示法，而另一些人则非常喜欢符号表示法。符号表示法的优点是， 允许你设置文件模式的某个属性，而不影响其他的属性。

# 更多

[权限](http://billie66.github.io/TLCL/book/chap10.html)
[Linux 用户和用户组管理](http://www.runoob.com/linux/linux-user-manage.html)

