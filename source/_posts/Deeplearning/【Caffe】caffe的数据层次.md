---
title: 【Caffe】caffe的数据层次
categories:
  - DeepLearning
tags:
  - deeplearning
  - caffe
abbrlink: 691f534d
date: 2019-06-17 13:21:53
---

![Caffe](https://image.shuiyujie.com/2019-06-17-18-25-09.png)

> Deep networks are compositional models that are naturally represented as a collection of inter-connected layers that work on chunks of data. Caffe defines a net layer-by-layer in its own model schema. The network defines the entire model bottom-to-top from input data to loss. As data and derivatives flow through the network in the [forward and backward passes](https://caffe.berkeleyvision.org/tutorial/forward_backward.html) Caffe stores, communicates, and manipulates the information as *blobs*: the blob is the standard array and unified memory interface for the framework. The layer comes next as the foundation of both model and computation. The net follows as the collection and connection of layers. The details of blob describe how information is stored and communicated in and across layers and nets.
>
> [Solving](https://caffe.berkeleyvision.org/tutorial/solver.html) is configured separately to decouple modeling and optimization.
>
> We will go over the details of these components in more detail.
>
> https://caffe.berkeleyvision.org/tutorial/net_layer_blob.html

<!-- more -->

# blob

![Caffe的数据层次](https://image.shuiyujie.com/2019-09-22-22-08-31.png)

数据表示为四维张量 (N, C, H, W)

- N 是 batch Size 大小
- C 是 channel
- H, W 为图片宽高

![blob](https://image.shuiyujie.com/2019-09-22-22-09-32.png)

- data 存储数据
- diff 存储梯度
- shape 分别为 blob 的尺度
- count 是所有数据数目,即 N*C*H*W

# layer

layer 是基本计算单元,除了输入层没有 bottom,输出层没有 top,每一层都有 bottom 和 top,分别串接前一层和后一层.每一种 layer 都有对应的 layer_param,用于实现该层参数的配置。

![layer](https://image.shuiyujie.com/2019-09-22-22-12-34.png)

- 需要实现 setup, forward, backward 登函数
- 使用 prototxt 配置
- 包含网络层参数,如输入数据层的 data_param

## Caffe 中的序列化

![序列化](https://image.shuiyujie.com/2019-09-22-22-14-01.png)

`/caffe/src/caffe/proto/caffe.proto`使用 protobuf 协议进行数据序列化和解析,实际使用的时候回编译成与所定义的数据结构对应的代码,从而实现数据的读取,解析和存储

**Caffe 中的 LayerParameter 示例**

```protobuf
// NOTE
// Update the next available ID when you add a new LayerParameter field.
//
// LayerParameter next available layer-specific ID: 149 (last added: clip_param)
message LayerParameter {
  optional string name = 1; // the layer name
  optional string type = 2; // the layer type
  repeated string bottom = 3; // the name of each bottom blob
  repeated string top = 4; // the name of each top blob

  // The train / test phase for computation.
  optional Phase phase = 10;

  // The amount of weight to assign each top blob in the objective.
  // Each layer assigns a default value, usually of either 0 or 1,
  // to each top blob.
  repeated float loss_weight = 5;

  // Specifies training parameters (multipliers on global learning constants,
  // and the name and other settings used for weight sharing).
  repeated ParamSpec param = 6;

  // The blobs containing the numeric parameters of the layer.
  repeated BlobProto blobs = 7;

```

> https://github.com/BVLC/caffe/blob/master/src/caffe/proto/caffe.proto

# net

Net 就是一个由 layer 实例组成的完整的 CNN 模型

![net](https://image.shuiyujie.com/2019-09-22-22-20-25.png)

[Caffe中的Net类是如何工作的](https://zhuanlan.zhihu.com/p/21878314)

## Caffe 中的训练参数

| 参数            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| net             | 网络文件路径                                                 |
| test_iter       | 一次测试的 batch 数目，如果它等于 1，就说明支取一个 batch size 的数据来做测试，如果 batch size 太小，那么对于分类任务来说统计出来的指标也不可信，所以一次测试，用到所有测试数据，因为常令 test_iter *test_batchsize = 测试集合的大小 |
| test_interval   | 测试间隔，即每隔多少次迭代进行一次测试                       |
| base_lr         | 学习率                                                       |
| lr_policy       | 学习率策略，base_lr 和 lr_policy 决定了学习率大小如何变化，而不同的学习率变化方法又有不同的参数 学习率的变化方法很多：fixed, step, exp, inv, multistep, poly, sigmoid |
| type            | 优化方法                                                     |
| clip_gradients  | 固定梯度范围                                                 |
| momentum        | 动量项                                                       |
| momentum        | 优化策略 Adam 用到的参数                                     |
| weight_decay    | 权重衰减率                                                   |
| display         | 显示间隔 debug_info                                          |
| max_iter        | 最大迭代次数                                                 |
| snapshot        | 存储模型间隔                                                 |
| snapshot_prefix | 存储路径与前缀                                               |
| solver_mode     | GPU/CPU 开关，可以指定用 GPU 或者 CPU 进行训                 |