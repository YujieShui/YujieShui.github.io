---
title: 【TensorFlow2.0】TensorFlow的安装
toc: true
categories:
  - DeepLearning
tags:
  - tensorflow
  - deeplearning
abbrlink: 7b2629fd
date: 2019-09-05 21:50:06
---

本文的环境是 Ubuntu16.04 + CUDA10 + Anaconda + TensorFlow2.0，现在的 conda 不支持安装 TensorFlow2.0 的包，所以需要用 pip 来安装 TensorFlow2.0。

```shell
pip install tensorflow-gpu==2.0.0-rc0 numpy matplotlib pandas -i https://pypi.tuna.tsinghua.edu.cn/simple
```

截止到现在，TensorFlow 发布了最新的 rc 版本，与以后的正式发行版差别不大了。

<!-- more -->

以下是 TensorFlow1.0 的安装方法：

![](http://image.shuiyujie.com/2019-09-22-22-45-03.png)

![](http://image.shuiyujie.com/2019-09-22-22-45-44.png)