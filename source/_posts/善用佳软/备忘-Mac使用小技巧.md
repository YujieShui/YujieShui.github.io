---
title: 备忘-Mac使用小技巧
toc: true
categories:
  - 善用佳软
tags:
  - mac
abbrlink: ccb5aaa3
date: 2019-06-23 00:39:34
---

![题图: https://pixabay.com/photos/apple-books-computers-desk-desktop-1839046/](http://image.shuiyujie.com/apple-1839046_1920.jpg)

备忘：一些使用 mac 的小 tips。

<!-- more -->

# 如何设置允许任何来源软件安装？

“安全性与隐私” 勾选任何来源，如果没有该选项，在终端输入如下命令：

```shell
sudo spctl --master-disable
```

# 如何设置开机启动项目

![如何设置开机启动项目](http://image.shuiyujie.com/2019-09-23-00-40-46.png)

# 连接不上 Wifi，其他设备可以正常使用

```
以下方法都是打苹果客服的热线得来的，顺序为推荐顺序

1.打开设置-网络-位置，更改为自动，或者顺便新建一个新的位置
2.打开设置-网络-高级，DNS，设置为114.114.114.114和8.8.8.8
3.打开finder-前往-电脑-Macintosh HD-资源库-Perferences，删除这个文件com.apple.wifi.message-tracer
4.关机，立即同时按下control+commmand+R+P, 听到三声响声后松手
```

