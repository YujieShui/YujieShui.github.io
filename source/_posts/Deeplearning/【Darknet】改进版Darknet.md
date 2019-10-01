---
title: 【Darknet】改进版Darknet
toc: true
categories:
  - DeepLearning
tags:
  - darknet
  - deeplearning
abbrlink: 3054b634
date: 2019-06-13 21:19:45
---

![darknet](https://image.shuiyujie.com/2019-06-13-21-01-25.png)

[AlexeyAB/darknet](AlexeyAB/darknet) 在[原版darknet](http://pjreddie.com/darknet/yolo/
)的基础上做了一些改进，用得最舒服的两个点是：可以实时查看 loss 和 mAP；可以自动计算一些模型评估指标。

下面是对其 README.md 的翻译。

<!-- more -->

- [Requirements (and how to install dependecies)](https://github.com/AlexeyAB/darknet#requirements)
- [Pre-trained models](https://github.com/AlexeyAB/darknet#pre-trained-models)
- [Explanations in issues](https://github.com/AlexeyAB/darknet/issues?q=is%3Aopen+is%3Aissue+label%3AExplanations)
- [Yolo v3 in other frameworks (TensorRT, TensorFlow, PyTorch, OpenVINO, OpenCV-dnn,...)](https://github.com/AlexeyAB/darknet#yolo-v3-in-other-frameworks)
- [Datasets](https://github.com/AlexeyAB/darknet#datasets)

1. [Improvements in this repository](https://github.com/AlexeyAB/darknet#improvements-in-this-repository)

2. [How to use](https://github.com/AlexeyAB/darknet#how-to-use-on-the-command-line)

3. [How to compile on Linux](https://github.com/AlexeyAB/darknet#how-to-compile-on-linux)

4. How to compile on Windows

5. - [Using vcpkg](https://github.com/AlexeyAB/darknet#how-to-compile-on-windows-using-vcpkg)
   - [Legacy way](https://github.com/AlexeyAB/darknet#how-to-compile-on-windows-legacy-way)

6. [How to train (Pascal VOC Data)](https://github.com/AlexeyAB/darknet#how-to-train-pascal-voc-data)

7. [How to train with multi-GPU:](https://github.com/AlexeyAB/darknet#how-to-train-with-multi-gpu)

8. [How to train (to detect your custom objects)](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects)

9. [How to train tiny-yolo (to detect your custom objects)](https://github.com/AlexeyAB/darknet#how-to-train-tiny-yolo-to-detect-your-custom-objects)

10. [When should I stop training](https://github.com/AlexeyAB/darknet#when-should-i-stop-training)

11. [How to calculate mAP on PascalVOC 2007](https://github.com/AlexeyAB/darknet#how-to-calculate-map-on-pascalvoc-2007)

12. [How to improve object detection](https://github.com/AlexeyAB/darknet#how-to-improve-object-detection)

13. [How to mark bounded boxes of objects and create annotation files](https://github.com/AlexeyAB/darknet#how-to-mark-bounded-boxes-of-objects-and-create-annotation-files)

14. [How to use Yolo as DLL and SO libraries](https://github.com/AlexeyAB/darknet#how-to-use-yolo-as-dll-and-so-libraries)

# 如何训练自定义的数据集

使用 yolo，yolov2 的配置文件 yolov2-voc.cfg, yolov2-tiny-voc.cfg, yolo-voc.cfg, yolo-voc.2.0.cfg 进行训练 [click by the link](https://github.com/AlexeyAB/darknet/tree/47c7af1cea5bbdedf1184963355e6418cb8b1b4f#how-to-train-pascal-voc-data)

##Yolo v3

Yolo v3 训练步骤：

1. copy yolov3.cfg to yolo-obj.cfg and:

1. change batch = 64
2. change subdivision = 8
3. change max_batch to (classes*2000), eg 比如 3 类，则 max_batches = 6000
4. max_batches 设置为 max_batch 的 80% 和 90%，接上面的则 step = 4800,5400
5. 修改 [yolo] - layers 的 classes = 3
6. 修改 [convolutions] 的 filters = (classes + 5)x3

- 在 build\darknet\x64\data\ 目录下创建 obj.names,存放检测目标的名称
- 在 build\darknet\x64\data\ 目录下创建 obj.data

1. classes= 2
2. train  = data/train.txt
3. valid  = data/test.txt
4. names = data/obj.names
5. backup = backup/

- build\darknet\x64\data\obj\ 放置图片,并且将 yolo 格式的 txt 文件和图片放在一起
- 在 build\darknet\x64\data\ 放置 train.txt, val.txt, test.txt 用来存放训练集, 验证集和测试集的图片列表

1. 可以用相对路径也可以用绝对路径
2. 更好的做法是将数据集统一放在一个文件夹下,可以用软连接的形式放到 data 目录下,或者直接用绝对路径

- 在 build\darknet\x64 放置供卷积层使用的预训练文件,[点击下载](https://pjreddie.com/media/files/darknet53.conv.74)
- 使用命令进行训练 ./darknet detector train data/obj.data yolo-obj.cfg darknet53.conv.74

1. 每迭代 100 次 yolo-obj_last.weights 会被保存到 build\darknet\x64\backup\ (在前面 obj.data 中可以设定)
2. 每迭代 1000 次 yolo-obj_xxxx.weights 会被保存到 build\darknet\x64\backup\
3. darknet.exe detector train data/obj.data yolo-obj.cfg darknet53.conv.74 -dont_show 可以禁止显示 Loss-Window,比如说我在远程主机上训练不想显示 loss 窗口
4. 不通过 GUI 而是通过浏览器来查看 mAP 和 Loss-chart 可以使用 ./darknet detector train data/obj.data yolo-obj.cfg darknet53.conv.74 -dont_show -mjpeg_port 8090 -map.接着在 [http://ip-address:8090](http://ip-address:8090/)查看

- 训练的时候每 4 个 epoch 计算一次 mAP,需要在 obj.data 中设置好 valid=valid.txtor train.txt,然后使用命令 darknet.exe detector train data/obj.data yolo-obj.cfg darknet53.conv.74 **-map**
- 训练结束之后可以在 build\darknet\x64\backup\ 找到最终的权重文件 yolo-obj_final.weights
- 每迭代 100 次都可以停止训练,之后再重新开始训练, 比如说 2000 次迭代之后暂停并重新开始 bdarknet.exe detector train data/obj.data yolo-obj.cfg backup\yolo-obj_2000.weights

1. darknet 的[原始项目](https://github.com/pjreddie/darknet)中权重文件每 10 000 次迭代才会保存一次(iteration > 1000)
2. 当然也会保存所有之前的中间结果

Note

1. 如果在训练中看到 avg(loss) 字段出现了 nan 表示训练出现了问题;如果其他字段出现 nan 训练依然正常
2. 如果在 cfg-file 中修改 width= 或者 height=,它们必须要是 32 的倍数
3. 测试目标检测效果可以使用 darknet.exe detector test data/obj.data yolo-obj.cfg yolo-obj_8000.weights
4. 如果出现 Out of memory 的错误,可以修改 cfg-file 增大 subdivisions=16, 32 or 64

## Yolov3 tiny

训练步骤和 Yolov3 类似，下面几个地方有区别：

- 下载 yolov3-tiny 的 weights 文件: https://pjreddie.com/media/files/yolov3-tiny.weights
- 获取预训练权值文件 yolov3-tiny.conv.15: `darknet.exe partial cfg/yolov3-tiny.cfg yolov3-tiny.weights yolov3-tiny.conv.15 15`

更多 yolo 系列的模型可以使用 ([DenseNet201-Yolo](https://github.com/AlexeyAB/darknet/blob/master/build/darknet/x64/densenet201_yolo.cfg)or [ResNet50-Yolo](https://github.com/AlexeyAB/darknet/blob/master/build/darknet/x64/resnet50_yolo.cfg)), 预训练模型可以从这里获取: https://github.com/AlexeyAB/darknet/blob/master/build/darknet/x64/partial.cmdIf 。

# 多 GPU 训练

多 GPU 训练，在迭代1000以上之后：

```shell
darknet.exe detector train cfg/voc.data cfg/yolov3-voc.cfg /backup/yolov3-voc_1000.weights -gpus 0,1,2,3
```

# 什么时候停止训练

一般来说每个类迭代 2000 就足够了，但是总的迭代次数不应少于 4000 次。但是如果想要更深入了解什么时候停止训练，请看下面这个手册：

1. 在训练期间，你会看到很多的错误提示，当看到 0.XXXXXXX avg 不再增长的时候就应该停止训练

```shell
Region Avg IOU: 0.798363, Class: 0.893232, Obj: 0.700808, No Obj: 0.004567, Avg Recall: 1.000000, count: 8 Region Avg IOU: 0.800677, Class: 0.892181, Obj: 0.701590, No Obj: 0.004574, Avg Recall: 1.000000, count: 8
```

当你看到 average loss 0.xxxxxx 在多个 iterations 中都不再减小时，你就应该停止训练。avgerage loss 最终一般在 0.05(小模型，简单数据集) 到 3.0(大模型，复杂数据集) 之间

2. 训练停止之后，你应该从 darknet\build\darknet\x64\backup 中选择出 last .weights 中表现最好的那个

比如说，你在迭代 9000 次时停止训练，但是最好的训练结果可能是 9000 之前的（7000，8000，9000）某一个 weight 文件。这是由过拟合(overfitting)导致的，过拟合指的是你可以在训练集的数据上表现良好，但是在测试集上表现很糟。所以，你应该使用过拟合之前的 weight 文件，也就是 Early stopping point 之前。

如何获取在 Early Stopping Point 之前的 weights？

1. 首先需要在 obj.data 中设置好 valid = valid.txt (验证集图片的路径列表文件，如果没有验证集就使用训练集的文件改名为 vaild.txt)
2. 如果在 9000 次迭代之后停止训练，那么就是使用 9000 次之前的几个 weights 进行验证

1. darknet.exe detector map data/obj.data yolo-obj.cfg backup\yolo-obj_7000.weights
2. darknet.exe detector map data/obj.data yolo-obj.cfg backup\yolo-obj_8000.weights
3. darknet.exe detector map data/obj.data yolo-obj.cfg backup\yolo-obj_9000.weights

![IOU](https://image.shuiyujie.com/2019-09-21-00-45-23.png)

- 接着对比每一个 weight(7000,8000,9000)最后一行输出

1. 选择 mAP (mean average precision) 或者 IoU (intersect over union) 高的
2. 比如说 yolo-obj_8000.weights 的 mAP 最高，就使用这个 weight 进行检测

- 或者使用 -mAP 参数进行训练 darknet.exe detector train data/obj.data yolo-obj.cfg darknet53.conv.74 -map，那么就会在 Loss-chart 中看到 mAP-chart

1. mAP 每 4 个 Epochs 会使用 valid=valid.txt 文件（1 Epoch = images_in_train_txt / batchiterations）
2. 可以用通过修改  [max_batches=](https://github.com/AlexeyAB/darknet/blob/0039fd26786ab5f71d5af725fc18b3f521e7acfd/cfg/yolov3.cfg#L20) 来改变 x 轴坐标的最大值，一般来说 max_batches=2000*classes

# 测试

```shell
# 单张图测试
./darknet detector test data/obj.data yolo-obj.cfg backup/yolo-obj_8000.weights
# 计算 mAP
./darknet detector map data/obj.data yolo-obj.cfg backup/yolo-obj_7000.weights
```

