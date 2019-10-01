---
title: 【TensorFlow2.0】手撕前向传播算法
toc: true
categories:
  - DeepLearning
tags:
  - tensorflow
  - deeplearning
abbrlink: c6ed1613
date: 2019-09-05 21:55:06
---

![TensorFlow2.0](https://image.shuiyujie.com/tensorflow2_coming_soon)

本文介绍如何使用 TensorFlow2.0 实现前向传播,先介绍用 TensorFlow 普通的 API 来实现前向传播,将会介绍:如何加载数据集,如何完成参数初始化和构建前向传播网络,如何计算 accuracy.

由于神经网络的训练流程大同小异,就可以使用 tf.keras 封装的 API 来简化模型训练和测试的流程.本文第二部分将会介绍如何使用 tf.keras 来定义神经网络以及优化器,如何用`tf.keras.metrics`来计算 accuracy 和 loss.

<!-- more -->

# 安装 TensorFlow2.0

本文的开发环境是 Ubuntu16.04 + CUDA10 + Anaconda + TensorFlow2.0，目前 conda 不支持安装 TensorFlow2.0 的包，所以需要用 pip 来安装 TensorFlow2.0。

```shell
pip install tensorflow-gpu==2.0.0-rc0 numpy matplotlib pandas -i https://pypi.tuna.tsinghua.edu.cn/simple
```

截止到现在，TensorFlow 发布了最新的 rc 版本，与以后的正式发行版差别不大了。

# 完成一次前向传播

![mnist前向传播](https://image.shuiyujie.com/mnist_front_forward.gif)



## 如何加载数据集

- 第1步: 使用 keras.datasets 加载 mnist 数据集,将会返回两组数据
- 第2步: 将 Numpy 类型的数据转换为 tensor
- 第3步: 将 tensor 转换为 datasets

```python
# 需要引入的包
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import datasets
import os

# 设置 TensorFlow 的日志级别,避免输出过多提示信息
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'

# 第1步: 使用 keras.datasets 加载 mnist 数据集,将会返回两组数据
# x: [60000,28,28], x_test:[10000,28,28]
# y: [60000], y_test:[10000]
(x, y), (x_text, y_text) = datasets.mnist.load_data()

# 第2步: 将 Numpy 类型的数据转换为 tensor
# keras.datasets 加载到的是 Numpy 类型的数据,将其转换为 tensor
# convert to tensor
# x:[0~255] => [0~1]
x = tf.convert_to_tensor(x, dtype=tf.float32) / 255.
y = tf.convert_to_tensor(y, dtype=tf.int32)

x_text = tf.convert_to_tensor(x_text, dtype=tf.float32) / 255.
y_text = tf.convert_to_tensor(y_text, dtype=tf.int32)

# 查看数据分布情况,shape,dype,min,max
print(x.shape, y.shape, x.dtype, y.dtype)
print(tf.reduce_min(x), tf.reduce_max(y))
print(tf.reduce_min(y), tf.reduce_max(y))

# 第3步: 将 tensor 转换为 datasets
# tensorflow 推荐使用 tf.data.Dataset 来加载数据集
# tensor => datasets
# 128个为一个batch返回datasets
train_db = tf.data.Dataset.from_tensor_slices((x,y)).batch(128)
test_db = tf.data.Dataset.from_tensor_slices((x_text,y_text)).batch(128)

train_iter = iter(train_db)
sample = next(train_iter)
print("batch:", sample[0].shape, sample[1].shape)
```

## 前向传播参数初始化

前面创建的 train_db 是 shape = [128, 28, 28],表示 128 张 28x28 的灰度图像.在输入的时候我将其展开成 shape = [128, 28*28],接着用全连接层降维到 shape = 10 的 tensor,对应 mnist 共 10 类的标签.

```python
# [b, 784] => [b, 256] => [b, 128] => [b, 10]
# w:[dim_in, dim_out]; b:[dim_out]
w1 = tf.Variable(tf.random.truncated_normal([784, 256], stddev=0.1))
b1 = tf.Variable(tf.zeros([256]))
w2 = tf.Variable(tf.random.truncated_normal([256, 128], stddev=0.1))
b2 = tf.Variable(tf.zeros([128]))
w3 = tf.Variable(tf.random.truncated_normal([128, 10], stddev=0.1))
b3 = tf.Variable(tf.zeros([10]))
```

## 前向传播 & 自动求导

```python
# 学习率
lr = 1e-3

# train_db 将会被计算 10 次
for epoch in range(10): # iterate db for 10
    # 遍历 train_db
    for step, (x, y) in enumerate(train_db, 1):
        # x:[128,28,28]
        # y:[128]

        # 将28x28的张量展开成784
        # x:[128,28,28] => [128,784]
        x = tf.reshape(x, [-1,28*28])

        # 使用 tf.GradientTape() 自动求导
        with tf.GradientTape() as tape:
            # 构建前向传播的网络
            # x:[b,28*28]
            # h1 = x@w1+b1
            # [b,784]@[784,256]+[256] => [b,256] + [256] => []
            h1 = tf.nn.relu(x@w1 + b1)
            h2 = tf.nn.relu(h1@w2 + b2)
            out = h2@w3 + b3
			
            # y 使用 one_hot 编码,与神经网络的输出对应
            # y: [b] => [b, 10]
            y_onehot = tf.one_hot(y, depth=10)

            # 计算均方误差 mse = mean(sum(y-out)^2)
            # loss:[b,10]
            loss = tf.square(y_onehot-out)
            # 求误差的请平均值
            # loss:[b,10] => scalar
            loss = tf.reduce_mean(loss)

        # 借助于 tensorflow 自动求导
        grads = tape.gradient(loss, [w1, b1, w2, b2, w3, b3])
        
        # 根据梯度更新参数
        # w1 = w1 - lr * w1_grad
        w1.assign_sub(lr * grads[0])
        b1.assign_sub(lr * grads[1])
        w2.assign_sub(lr * grads[2])
        b2.assign_sub(lr * grads[3])
        w3.assign_sub(lr * grads[4])
        b3.assign_sub(lr * grads[5])

        # 每迭代100次输出一次loss
        if step % 100 == 0:
            print(epoch+1, (epoch+1)*step, 'loss:', float(loss))
```

## 计算准确率 accuracy

我们需要通过模型的准确率来评估模型,模型评估需要在 test_db 上进行.

```python
	total_number = 0
    total_correct = 0
    # 遍历 test_db
    for step, (x, y) in enumerate(test_db):
		
        # [b, 28, 28] => [b, 28*28]
        x = tf.reshape(x,[-1,28*28])
        
        # 利用最新的参数完成一次前向传播
        # [b, 784] => [b, 256] => [b, 128] => [b, 10]
        h1 = tf.nn.relu(x@w1 + b1)
        h2 = tf.nn.relu(h1@w2 + b2)
        out = h2@w3 + b3

        # 使用 softmax() 输出每个分类的概率值
        # [b, 10] ~ R
        # [b, 10] ~ [0,1]
        prob = tf.nn.softmax(out, axis=1)
        
        # 概率最大的值就是模型的预测值
        # [b, 10] => [b]
        preb = tf.argmax(prob, axis=1)
        preb = tf.cast(preb, dtype=tf.int32)
        
        # 预测值与真实值比较
        # [b] int32
        # print(y.dtype, preb.dtype)
        correct = tf.cast(tf.equal(y, preb), dtype=tf.int32)
        correct = tf.reduce_sum(correct)
        total_correct += int(correct)
        total_number += x.shape[0]
	
    acc = total_correct / total_number
    print("accuracy:", acc)
```



# 前向传播过程优化

![mnist in csv](https://image.shuiyujie.com/mnist_in_csv.png)

## 构建数据集时添加数据预处理

Tensorflow 鼓励使用 `tf.data.Dataset`加载数据集,它给我封装了许多处理数据集的方法,比如`train_db = train_db.map(preprocess).shuffle(60000).batch(128)`

在`preprocess(x,y)`中进行数据预处理,利用`map(preprocess)`调用预处理函数,再用`shuffle()`随机成对打乱数据集,最后用`batch()`将数据集按照 128 一份来分隔.

```python
def preprocess(x, y):
    # 数据预处理,归一化,类型转换
    x = tf.cast(x, dtype=tf.float32) / 255.
    y = tf.cast(y, dtype=tf.int32)
    return x,y

# x: [60000,28,28], x_test:[10000,28,28]
# y: [60000], y_test:[10000]
(x, y), (x_text, y_text) = datasets.mnist.load_data()

# tensor => datasets
train_db = tf.data.Dataset.from_tensor_slices((x,y))
train_db = train_db.map(preprocess).shuffle(60000).batch(128)

test_db = tf.data.Dataset.from_tensor_slices((x_text,y_text))
test_db = test_db.map(preprocess).shuffle(10000).batch(128)
```

## 使用 tf.keras 来构建模型

TensorFlow2.0 降低了使用者的门槛,利用 `tf.keras` 可以直接使用 Keras 的一系列 API,这样就可以更加方便地定义模型,比如说前面定义参数 [w,b] 的过程就可以简化成下面这样

```python
# 用 Sequential 构建 3 层的全连接
network = Sequential([layers.Dense(256, activation='relu'),
            layers.Dense(128, activation='relu'),
            layers.Dense(10)
            ])

# build 并制定 input
network.build(input_shape=[None, 28*28])
# 查看模型参数
network.summary()

# 指定优化器
optimizer = optimizers.Adam(lr=0.01)
```

## 使用 metrics 自动计算 loss 和 accuracy

```python
# Step1.Build a meter
acc_meter = metrics.Accuracy()
loss_meter = metrics.Mean()

for epoch in range(10): # iterate db for 10
    for step, (x, y) in enumerate(train_db, 1):
        
        x = tf.reshape(x, [-1,28*28])

        with tf.GradientTape() as tape:
         
            # [b,784] => [b,10]
            out = network(x)
            y_onehot = tf.one_hot(y, depth=10)

            loss = tf.reduce_mean(tf.losses.categorical_crossentropy(y_onehot, out, from_logits=True))

            # Step2.Update data
            loss_meter.update_state(loss)

        # compute gradients
        # grads = tape.gradient(loss, [w1, b1, w2, b2, w3, b3])
        grads = tape.gradient(loss, network.trainable_variables)
        optimizer.apply_gradients(zip(grads, network.trainable_variables))

        if step % 100 == 0:
            # Step3.Get Average data
            print(step, 'loss:', loss_meter.result().numpy())
            # Clear buffer
            loss_meter.reset_states()

    total_number = 0
    total_correct = 0
    for step, (x, y) in enumerate(test_db):
        acc_meter.reset_states()

        # [b, 28, 28] => [b, 28*28]
        x = tf.reshape(x,[-1,28*28])
        # [b, 784] => [b, 256] => [b, 128] => [b, 10]
        # h1 = tf.nn.relu(x@w1 + b1)
        # h2 = tf.nn.relu(h1@w2 + b2)
        # out = h2@w3 + b3
        
		# 输出调用 network 就可以获得
        out = network(x)

        # [b, 10] ~ R
        # [b, 10] ~ [0,1]
        prob = tf.nn.softmax(out, axis=1)
        # [b, 10] => [b]
        pred = tf.argmax(prob, axis=1)
        pred = tf.cast(pred, dtype=tf.int32)
        # [b] int32
        # print(y.dtype, preb.dtype)
        # correct = tf.cast(tf.equal(y, pred), dtype=tf.int32)
        # correct = tf.reduce_sum(correct)
        # total_correct += int(correct)
        # total_number += x.shape[0]
        
        # metrics 能快速计算 acc 和 loss
        acc_meter.update_state(y, pred)

    # acc = total_correct / total_number
    print(step, 'Evaluate Acc:', acc_meter.result().numpy())
```

## tf.keras 版

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import datasets, layers, optimizers, Sequential, metrics
import os

os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'

def preprocess(x, y):
    """
    x is a simple image, not a batch
    """
    x = tf.cast(x, dtype=tf.float32) / 255.
    x = tf.reshape(x, [28*28])
    y = tf.cast(y, dtype=tf.int32)
    y = tf.one_hot(y, depth=10)
    return x,y

# x: [60000,28,28], x_test:[10000,28,28]
# y: [60000], y_test:[10000]
(x, y), (x_text, y_text) = datasets.mnist.load_data()

# tensor => datasets
train_db = tf.data.Dataset.from_tensor_slices((x,y))
train_db = train_db.map(preprocess).shuffle(60000).batch(128)

test_db = tf.data.Dataset.from_tensor_slices((x_text,y_text))
test_db = test_db.map(preprocess).shuffle(10000).batch(128)

# 使用 tf.Sequential 定义上面的这些参数
network = Sequential([layers.Dense(256, activation='relu'),
            layers.Dense(128, activation='relu'),
            layers.Dense(10)
            ])

network.build(input_shape=(None, 28*28))
network.summary()

network.compile(optimizer=optimizers.Adam(lr=0.01),
                loss=tf.losses.CategoricalCrossentropy(from_logits=True),
                metrics=['accuracy'])

network.fit(train_db, epochs=10, validation_data=test_db, validation_freq=2)

network.evaluate(test_db)

sample = next(iter(ds_val))
x = sample[0]
y = sample[1] # one-hot
pred = network.predict(x) # [b, 10]
# convert back to number
y = tf.argmax(y, axis=1)
pred = tf.argmax(pred, axis=1)

print(pred)
print(y)

```

