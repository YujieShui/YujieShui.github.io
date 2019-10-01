---
title: 【TensorFlow2.0】TensorFlow简介
toc: true
categories:
  - DeepLearning
tags:
  - tensorflow
  - deeplearning
abbrlink: b5bae26d
date: 2019-09-05 21:45:06
---

![TensorFlow2.0](https://image.shuiyujie.com/tensorflow2_coming_soon)

<!-- more -->

# TensorFlow 里程碑

![TensorFlow里程碑](https://image.shuiyujie.com/2019-09-22-21-50-21.png)

- 2015.9发布0.1版本
- 2017.2发布1.0版本
- 2019春发布2.0版本

# TensorFlow vs Pytorch

TensorFlow1.0 上手困难，经常被诟病，都说 Pytorch。现在 TensorFlow2.0 出了，就容易上手多了。

![TensorFlow1.0 vs PyTorch](https://image.shuiyujie.com/2019-09-22-21-53-34.png)

![TensorFlow2.0 vs PyTorch](https://image.shuiyujie.com/2019-09-22-21-54-20.png)

学习建议就是

- 忘掉 TensorFlow1.x

- PyTorch和TensorFlow选择一个主修

- - 两者都要掌握

- Keras逐渐淡出

- - TF+Keras
  - PyTorch+Caffe2

# TensorFlow 的优点

## GPU 加速

用 CPU 和 GPU 分别测试一下运算速度

```python
import tensorflow as tf
import timeit

with tf.device('/cpu:0'):
    cpu_a = tf.random.normal([10000, 1000])
    cpu_b = tf.random.normal([1000, 2000])
    print(cpu_a.device, cpu_b.device)
with tf.device('/gpu:0'):
    gpu_a = tf.random.normal([10000, 1000])
    gpu_b = tf.random.normal([1000, 2000])
    print(gpu_a.device, gpu_b.device)
def cpu_run():
    with tf.device('/cpu:0'):
        c = tf.matmul(cpu_a, cpu_b)
    return c
def gpu_run():
    with tf.device('/gpu:0'):
        c = tf.matmul(gpu_a, gpu_b)
    return c

# warm up
cpu_time = timeit.timeit(cpu_run, number=10)
gpu_time = timeit.timeit(gpu_run, number=10)
print('warmup:', cpu_time, gpu_time)

cpu_time = timeit.timeit(cpu_run, number=10)
gpu_time = timeit.timeit(gpu_run, number=10)
print('run time:', cpu_time, gpu_time)
```

![](https://image.shuiyujie.com/2019-09-22-21-56-26.png)

## 自动求导

```python
import tensorflow as tf
x = tf.constant(1.)
a = tf.constant(2.)
b = tf.constant(3.)
c = tf.constant(4.)
with tf.GradientTape() as tape:
    tape.watch([a, b, c])
    y = a**2 * x + b * x + c
[dy_da, dy_db, dy_dc] = tape.gradient(y, [a, b, c])
print(dy_da, dy_db, dy_dc)
```

## 神经网络 API

![神经网络 API](https://image.shuiyujie.com/2019-09-22-21-57-14.png)

# TensorFlow 社区

[TensorFlow 社区地址](https://github.com/tensorflow?utf8=✓&q=model&type=&language=)

[TensorFlow Github 地址](https://github.com/tensorflow/tensorflow)

## TensorFlow 生态 - TFX

![TFX](https://image.shuiyujie.com/2019-09-22-21-59-09.png)

TFX - 基于 TensorFlow 的端到端机器学习平台

[TFX Github 地址](https://github.com/tensorflow/tfx)

## TensorFlow 生态 - Kubeflow

![Kubeflow](https://image.shuiyujie.com/2019-09-22-22-00-34.png)

Kubeflow 表示 K8s + Data flow

一个典型的 AI 工作流程是怎么样的？可以划分成 8 个步骤：

1. 从客户获得产品需求
2. 设计我们的产品
3. 数据处理
4. 训练模型
5. 数据/模型可视化
6. 将模型做成服务
7. 模型验证：蓝绿测试、灰度测试
8. 商业上取得成功

[KubeFlow Github 地址](https://github.com/kubeflow/kubeflow)

[kubernetes Githube 地址](https://github.com/kubernetes/kubernetes )