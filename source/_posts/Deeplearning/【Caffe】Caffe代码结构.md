---
title: 【Caffe】Caffe代码结构
toc: true
categories:
  - DeepLearning
tags:
  - deeplearning
  - caffe
abbrlink: c2c24b74
date: 2019-06-17 13:31:53
---

![Caffe](https://image.shuiyujie.com/2019-06-17-18-25-09.png)

> Caffe 安装看 [Installation](http://caffe.berkeleyvision.org/installation.html), 第一个例子可以看 [Training LeNet on MNIST with caffe](http://caffe.berkeleyvision.org/gathered/examples/mnist.html)  的示例.跟着示例做一遍,精确度能有 98% 左右,做完很有成就感.跑完之后可以跑更多的官方示例,它包括 Notebook Example 和 Command Line Example.
>
> [Caffe Tutorial](http://caffe.berkeleyvision.org/tutorial/)介绍了 Caffe 的基础,其中也包括各种数据结构 http://caffe.berkeleyvision.org/tutorial/
>
> [Model Zoo](http://caffe.berkeleyvision.org/model_zoo.html)有很多实现的网络和训练好的模型
>
> 接下来重点看一下 [Caffe Tutorial](http://caffe.berkeleyvision.org/tutorial/) 了解基本数据结构

<!--more -->

# 目录结构

![目录结构](https://image.shuiyujie.com/2019-09-22-22-22-16.png)

# 数据结构类

**src/caffe 目录下**

| blob.hpp / cpp          |                                       |
| ----------------------- | ------------------------------------- |
| layer.hpp / cpp         |                                       |
| net.hpp/cpp             |                                       |
| solver.hpp/cpp          |                                       |
| sgd_solvers.hpp/cpp     | blob , layer , net 的定义             |
|                         |                                       |
| solver _factory.hpp/cpp |                                       |
| layer_factory.hpp/cpp   | 工厂类模板定义和普通 layer 的模板定义 |
|                         |                                       |
| caffe.hpp/cpp           |                                       |
| common.hpp/cpp          | 通用包含文件                          |
|                         |                                       |
| internal thread.hpp/cpp |                                       |
| parallel.hpp/cpp        |                                       |
| syncedmem.hpp/cpp       | gpu 编程和内存等较为底层的文件        |

# IO 类

不同格式的数据读取层

| base_data_layer.hpp/cpp  |                          |
| ------------------------ | ------------------------ |
| data_layer.npp/cpp       |                          |
| window_data_layer.cpp    |                          |
| parameter_layer.cpp      |                          |
| memory_data_layer.cpp    |                          |
| dummy_data_layer.cpp     |                          |
| hdf5_data_layer.cpp      |                          |
| hdf5_output_layer.cpp    |                          |
| image_data_layer.hpp/cpp | 不同格式的数据读取层     |
|                          |                          |
| data_transformer.hpp/cpp | 数据的预处理，增强等变换 |

# 基础函数类

| math.hpp / cpp          | 基本数学操作，加减乘除 |
| ----------------------- | ---------------------- |
|                         |                        |
| absval_layer.hpp / cpp  |                        |
| exp_layer.hpp / cpp     |                        |
| log_layer.hpp / cpp     | 基础数学函数变换       |
|                         |                        |
| power_layer.hpp /cpp    |                        |
| tanh_layer.hpp / cpp    |                        |
| sigmoid_layer.hpp / cpp |                        |
| relu_layer.hpp / cpp    | 若干激活函数           |

# 形状处理类

| flatten_layer.hpp/cpp     |                          |
| ------------------------- | ------------------------ |
| slice_layer.hpp / cpp     |                          |
| split_layer.hpp / cpp     |                          |
| tile_layer.hpp / cpp      |                          |
| concat_layer.hpp / cpp    |                          |
| reduction_layer.hpp / cpp |                          |
| eltwise_layer.hpp/cpp     |                          |
| crop_layer.hpp/cpp        |                          |
| pooling_layer.hpp / cpp   |                          |
| scale_layer.hpp / cpp     | 对 blob 进行各类形状变换 |

# 损失函数类

用于分类，回归等任务的常见损失函数定义

```
multinomial_logistic_loss_layer.hpp/cpp
softmax_loss_layer.hpp/cpp
euclidean_loss_layer.hpp/cpp
sigmoid_cross_entropy_loss_layer.hpp/cpp
contrastive_loss_layer.hpp/cpp
hinge_loss_layer.hpp/cpp
infogain_loss_layer.hpp/cpp
```

# 卷积类

卷积与反卷积定义

```
im2col_layer.cpp
base_conv_layer.cpp
conv_layer.cpp
deconv_layer.cpp
inner_product_layer.cpp
```

