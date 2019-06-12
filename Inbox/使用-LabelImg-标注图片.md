---
title: darknet 
toc: true
categories:
  - 深度学习
tags:
  - 数据处理
abbrlink: 85d6b824
date: 2019-06-04 22:18:30
---

![如图](http://image.shuiyujie.com/2019-06-04-22-23-29.png)

数据是机器学习的基础，图像分类、检测的任务都是基于数据集开始的，比如 [mnist](http://yann.lecun.com/exdb/mnist/)、[PASCAL](http://host.robots.ox.ac.uk/pascal/VOC/)、[imagenet](<http://www.image-net.org/>) 等。在实际开发任务中，我们需要自己建立数据集，就要使用标注工具。

不可否认，打标任务是枯燥的，但有一句话经历过就会熟悉在心：**大多数情况下，数据集决定了任务的成败！**

本文介绍一款开源的图片标注工具 LabelImage。

<!-- more -->

# 下载和编译

首先从 Github 下载 [LabelImg](https://github.com/tzutalin/labelImg)，并根据其文档编译运行 LabelImg。

# 文件结构

```
.
├── build-tools
├── CONTRIBUTING.rst
├── data
    |—— predefined_classes.txt
├── labelImg.py
...
```

首先在`data/predefined_classes.txt`文件配置进行图片的类别，比如我有以下 5 个类别就进行如下配置

```
dog
cat
magpie
pigeon
nest
```

`labelImg.py`是 labelimg 的启动文件，启动方式和你的安装方式对应，具体见 [LabelImg说明](https://github.com/tzutalin/labelImg)

# 开始使用

启动 labelimg 之后的界面如下图所示

![labelimg 界面](http://image.shuiyujie.com/labelimg.png)

注意：

1. 存放图片文件夹和存放标记文件的文件夹需要保持一致
2. 正确选择标记文件的格式，是需要 yolo 格式还是 xml 格式，默认为 xml 格式
3. 右侧可以选择默认标签，当密集标注一个类的时候很实用
4. 打完几张标签之后请确认标签是否正确，再去文件目录下确认以下标记文件是否生成且格式正确

# 快捷键

```
+------------+--------------------------------------------+
| Space      | 保存                                        |
+------------+--------------------------------------------+
| w          | 创建矩形框                                   |
+------------+--------------------------------------------+
| d          | 下一张图片                                   |
+------------+--------------------------------------------+
| a          | 上一张图片                                   |
+------------+--------------------------------------------+
```

最常用的快捷键是上面这 4 个，用好快捷键可以调高打标效率。此外按住`ctrl+鼠标滚轮`可以调整图片大小，局部放大图片可以提高打标的精准度。