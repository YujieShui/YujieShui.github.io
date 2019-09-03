---
title: 【Caffe】Caffe实战手写数字分类
toc: true
categories:
  - DeepLearning
tags:
  - deeplearning
  - caffe
  - 开源框架
abbrlink: 96cd1245
date: 2019-06-17 18:21:53
---

![Caffe](http://image.shuiyujie.com/2019-06-17-18-25-09.png)

[Caffe](http://caffe.berkeleyvision.org/)是优秀的深度学习开源框架，并具有良好的开源生态，我们可以在[Model zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)中找到许多模型的实现。本文以 Caffe 提供的 [Training LeNet on MNIST with Caffe](https://caffe.berkeleyvision.org/gathered/examples/mnist.html) 为例，介绍 Caffe 使用流程，并且在原文基础上增加可视化 accuracy 和 loss 的内容。

本文的目标是能够了解 Caffe 基本的使用流程。

<!-- more -->

![Caffe 使用流程](http://image.shuiyujie.com/2019-06-17-18-23-18.png)

# 准备数据

在安装和编译 Caffe 之后进入根目录，执行以下命令，将会下载 [MNIST](http://yann.lecun.com/exdb/mnist/) 数据集。

```shell
cd $CAFFE_ROOT
./data/mnist/get_mnist.sh #下载 mnist 数据集
./examples/mnist/create_mnist.sh # 将下载下来的数据集格式调整为 caffe 能够解析的格式
```

数据集包括 mnist_train_lmdb 和 mnist_test_lmdb 两部分。

# 定义 Net

Caffe 提供的示例中使用 lenet_train_test.prototxt 定义 [LeNet](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf) 的网络结构。LetNet 网络结构为`卷积层 -> 池化层 -> 卷积层 -> 池化层 -> 全连接层 -> 全连接层 -> 全连接层`。Caffe 用 layer 来表示表示网络，网络被定义在以`.prototxt`结尾的文件中，因为 Caffe 使用了 [Google Protobuf](https://developers.google.com/protocol-buffers/docs/overview) 作为数据传输的格式。

在 lenet_train_test.prototxt 中我们可看到`name: "LeNet"`表示网络结构的名称为 LeNet，接下来是

```protobuf
layer {
  name: "mnist" # 名称
  type: "Data" # 输入层
  transform_param {
    scale: 0.00390625 # 归一化，范围为 0-1
  }
  data_param {
    source: "mnist_train_lmdb" # 指定要读取的数据
    backend: LMDB
    batch_size: 64
  }
  top: "data"
  top: "label"
}
...
```

我们可以通过 [http://ethereon.github.io/netscope/#/editor](http://ethereon.github.io/netscope/#/editor) 生成网络图像，或者使用 caffe 提供的脚本，它在`caffe/python/draw_net.py`。

# 配置 Solver

Solver 用来指定训练的一些参数，比如使用哪个网络、迭代次数、优化方法、使用 GPU 还是 CPU 等信息。

```shell
# The train/test net protocol buffer definition
net: "examples/mnist/lenet_train_test.prototxt"
# test_iter specifies how many forward passes the test should carry out.
# In the case of MNIST, we have test batch size 100 and 100 test iterations,
# covering the full 10,000 testing images.
test_iter: 100
# Carry out testing every 500 training iterations.
test_interval: 500
# The base learning rate, momentum and the weight decay of the network.
base_lr: 0.01
momentum: 0.9
weight_decay: 0.0005
# The learning rate policy
lr_policy: "inv"
gamma: 0.0001
power: 0.75
# Display every 100 iterations
display: 100
# The maximum number of iterations
max_iter: 10000
# snapshot intermediate results
snapshot: 5000
snapshot_prefix: "examples/mnist/lenet"
# solver mode: CPU or GPU
solver_mode: GPU
```

# 训练

训练的时候指定 Solver，保存权值文件的路径，日志：`./examples/mnist/train_lenet.sh -gpu 1`

```shell
#!/usr/bin/env sh
set -e
SOLVER=examples/mnist/lenet_solver.prototxt
WEIGHTS=./lenet_iter_10000.caffemodel
./build/tools/caffe train --solver=${SOLVER} $@ 2>&1 | tee log.txt
```

通过解析日志，我们能够可视化模型训练过程中的 accuray 和 loss 的情况。

```shell
# 解析日志文件,将 acc 和 loss 输出到 .refine 文件
grep "Test net output #0: accuracy =" log.txt > trainacc.refine
grep "Test net output #1: loss =" log.txt > trainloss.refine

# 生成如下所示的 .refine 文件
# trainacc.refine
I0615 10:16:15.671094 11161 solver.cpp:414] Test net output #0: accuracy = 0.9903
I0615 10:16:16.061339 11161 solver.cpp:414] Test net output #0: accuracy = 0.9903
I0615 10:16:16.451835 11161 solver.cpp:414] Test net output #0: accuracy = 0.988
I0615 10:16:16.850911 11161 solver.cpp:414] Test net output #0: accuracy = 0.9906

# trainloss.refine
I0615 10:16:15.671116 11161 solver.cpp:414] Test net output #1: loss = 0.0298686 (* 1 = 0.0298686 loss)
I0615 10:16:16.061362 11161 solver.cpp:414] Test net output #1: loss = 0.0285139 (* 1 = 0.0285139 loss)
I0615 10:16:16.451859 11161 solver.cpp:414] Test net output #1: loss = 0.0348767 (* 1 = 0.0348767 loss)
I0615 10:16:16.850932 11161 solver.cpp:414] Test net output #1: loss = 0.0284803 (* 1 = 0.0284803 loss)

# 接着用 python 脚本解析 .refine 绘制 acc 和 loss 曲线
python show_acc.py
python show_loss.py
```

Python 脚本可以在 [Github Gist](https://gist.github.com/YujieShui/67933c77054410461cb18c031b36057d) 中查看，需要根据之前生成的日志进行小改动。

![train_loss](http://image.shuiyujie.com/2019-06-17-19-13-31.png)

可以看到迭代 100 次 loss 已经降到很低了,之后略有降低,300次左右基本上就在 0.3, 0.4 左右

![train_accuracy](http://image.shuiyujie.com/2019-06-17-19-14-19.png)

与之相对的，100 次左右准确率就有 0.96 了,缓慢上升,最后准确率稳定在 0.98 。

# 测试

测试的方法和训练的方法基本一致，使用 `./examples/mnist/test_lenet.sh -gpu 1`

```shell
#!/usr/bin/env sh
set -e
MODEL=examples/mnist/lenet_iter_10000.caffemodel
TRAIN_NET=examples/mnist/lenet_train_test.prototxt
./build/tools/caffe test -model ${TRAIN_NET} -weights ${MODEL} $@ 2>&1 | tee test_log.txt
# 结果如下所示
...
I0616 15:29:32.107722 13441 caffe.cpp:304] Batch 49, accuracy = 1
I0616 15:29:32.107739 13441 caffe.cpp:304] Batch 49, loss = 0.00704199
I0616 15:29:32.107748 13441 caffe.cpp:309] Loss: 0.0412109
I0616 15:29:32.107769 13441 caffe.cpp:321] accuracy = 0.9862
I0616 15:29:32.107782 13441 caffe.cpp:321] loss = 0.0412109 (* 1 = 0.0412109 loss)
```

用上面同样的方法能够可视化测试的 loss 和 accuracy。

***

本文介绍了 Caffe 的基本使用流程，这样就可以把一些别人写好的网络用起来了。进一步我们需要了解如何用 Caffe 自定义网络结构，调整网络的参数，完成更加个性化的任务。

参考：

[Training LeNet on MNIST with Caffe](https://caffe.berkeleyvision.org/gathered/examples/mnist.html)

[A step by step guide to Caffe](http://shengshuyang.github.io/A-step-by-step-guide-to-Caffe.html)

[Caffe](http://caffe.berkeleyvision.org/)

[Caffe Model zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo#cnn-object-proposal-models-for-salient-object-detection)

