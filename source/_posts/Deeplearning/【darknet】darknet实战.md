---
title: 【darknet】darknet实战
toc: true
categories:
  - DeepLearning
tags:
  - darknet
  - 开源框架
  - deeplearning
abbrlink: 5ba44047
date: 2019-06-13 21:09:45
---

![darknet](http://image.shuiyujie.com/2019-06-13-21-01-25.png)

> Darknet is an open source neural network framework written in C and CUDA. It is fast, easy to install, and supports CPU and GPU computation.
>
> —— https://pjreddie.com/darknet/

本文是对使用 darknet 进行目标检测的小结，包括：

1. 数据集准备：如何使用 labelimage 对数据进行标注，注意事项，文件格式转换
2. darknet 使用：如何编译和修改配置文件
3. 模型评估：如何查看 loss、计算 IoU、recall、mAP

<!-- more -->

# 数据集准备

![如图](http://image.shuiyujie.com/2019-06-04-22-23-29.png)

大多数情况下，数据集决定了任务的成败。一开始我认为标注数据是一件非常枯燥和乏味的事情，但是当模型指标一直上不去，检测和识别效果也一直不好时，回过头才会发现是因为数据标注的有问题。

这只有自己经历过之后才会有体会，得出这样几条经验：

1. 保持类间差距大，类内差距小
2. 一开始先标注少量的图片并训练模型查看效果，根据结果进行调整，否则等到标注了大量图片之后再回过头修改，得不偿失。

## labelimage

图片标注使用的是开源的工具 [LabelImg](https://github.com/tzutalin/labelImg)，可以查看文档自行编译，下载之后的文件结构如下：

```
.
├── build-tools
├── CONTRIBUTING.rst
├── data
    |—— predefined_classes.txt
├── labelImg.py
...
```

- 在`data/predefined_classes.txt`文件配置进行图片的类别

## 界面展示及注意事项

启动 labelimg 之后的界面如下图所示

![labelimg 界面](http://image.shuiyujie.com/labelimg.png)

注意：

1. 存放图片文件夹和存放标记文件的文件夹需要保持一致
2. 正确选择标记文件的格式，是需要 yolo 格式还是 xml 格式，默认为 xml 格式
3. 右侧可以选择默认标签，当密集标注一个类的时候很实用
4. 打完几张标签之后请确认标签是否正确，再去文件目录下确认以下标记文件是否生成且格式正确

## 快捷键

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

## yolo格式和 xml 格式转换

### xml2yolo.py

```python
"""
1. 修改 classes 列表中的元素为当前标签列表
2. 修改 list_xml 指向的 xml 文件保存的位置
"""

#import xml.etree.ElementTree as ET
from xml.etree import ElementTree as ET
import pickle
import os
from os import listdir, getcwd
from os.path import join
import glob

classes = ['rabbit']

def convert(size, box):
      dw = 1. / (size[0])
      dh = 1. / (size[1])
      x = (box[0] + box[1]) / 2.0 - 1
      y = (box[2] + box[3]) / 2.0 - 1
      w = box[1] - box[0]
      h = box[3] - box[2]
      x = x * dw
      w = w * dw
      y = y * dh
      h = h * dh
      return (x, y, w, h)


def convert_annotation(xml_file, txt_file):
      in_file = open(xml_file)

      tree = ET.parse(in_file)
      root = tree.getroot()
      size = root.find('size')
      w = int(size.find('width').text)
      h = int(size.find('height').text)

      for obj in root.iter('object'):

            cls = obj.find('name').text
            if cls not in classes:
                  continue
            cls_id = classes.index(cls)
            xmlbox = obj.find('bndbox')
            b = (float(xmlbox.find('xmin').text), float(xmlbox.find('xmax').text), float(xmlbox.find('ymin').text),
                 float(xmlbox.find('ymax').text))
            bb = convert((w, h), b)
            txt_file.write(str(cls_id) + " " + " ".join([str(a) for a in bb]) + '\n')

list_xml = []
list_xml = glob.glob(r"/file_path/*.xml")

for xml_file in list_xml:
      xml_name = xml_file.split('.')[0]
      txt_file = open('%s.txt' % (xml_name), 'w')
      convert_annotation(xml_file, txt_file)
      txt_file.close()

```

### yolo2xml.py

```python
"""
1. 修改 class_name
2. 修改 src_img_dir/src_txt_dir/src_xml_dir 路径
"""
import os, sys
import glob
from PIL import Image

class_name = ['switch_closed','switch_open','whirled_switch_closed','whirled_switch_open',
        'lifting_switch_closed','lifting_switch_open','closure_switch_closed','closure_switch_open',
        'color_switch01_red','color_switch01_green','color_switch02_red','color_switch02_green',
        'isolation_switch_closed','isolation_switch_open','grounding_knife_switch01_closed','grounding_knife_switch01_open',
        'grounding_knife_switch02_closed','grounding_knife_switch02_open']

def convert_yolo_coordinates_to_voc(x_c_n, y_c_n, width_n, height_n, img_width, img_height):
  ## remove normalization given the size of the image
  x_c = float(x_c_n) * img_width
  y_c = float(y_c_n) * img_height
  width = float(width_n) * img_width
  height = float(height_n) * img_height
  ## compute half width and half height
  half_width = width / 2
  half_height = height / 2
  ## compute left, top, right, bottom
  ## in the official VOC challenge the top-left pixel in the image has coordinates (1;1)
  left = int(x_c - half_width) + 1
  top = int(y_c - half_height) + 1
  right = int(x_c + half_width) + 1
  bottom = int(y_c + half_height) + 1
  return left, top, right, bottom


#  将标注的txt转换为 voc xml

# VEDAI 图像存储位置
src_img_dir = "./switch_mAP"
# VEDAI 图像的 ground truth 的 txt 文件存放位置
src_txt_dir = "./switch_mAP"
src_xml_dir = "./switch_mAP"

img_Lists = glob.glob(src_img_dir + '/*.txt')

# 文件名(含扩展名)
img_basenames = []  # e.g. 100
for item in img_Lists:
      img_basenames.append(os.path.basename(item))

img_names = []  # e.g. 100
for item in img_basenames:
      # 文件名与扩展名
      temp1, temp2 = os.path.splitext(item)
      img_names.append(temp1)

for img in img_names:
      
      im = ""
      suffix = ""
      # 同一个文件夹下存在多种图片格式，在这里加上格式判断
      if os.path.exists(src_img_dir + '/' + img + '.jpg'):
            im = Image.open((src_img_dir + '/' + img + '.jpg'))
            suffix = '.jpg'
      elif os.path.exists(src_img_dir + '/' + img + '.JPG'):
            im = Image.open((src_img_dir + '/' + img + '.JPG'))
            suffix = '.JPG'
      elif os.path.exists(src_img_dir + '/' + img + '.png'):
            im = Image.open((src_img_dir + '/' + img + '.png'))
            suffix = '.png'
      elif os.path.exists(src_img_dir + '/' + img + '.PNG'):
            im = Image.open((src_img_dir + '/' + img + '.PNG'))
            suffix = '.PNG'
      elif os.path.exists(src_img_dir + '/' + img + '.JPEG'):
            im = Image.open((src_img_dir + '/' + img + '.JPEG'))
            suffix = '.JPEG'
      elif os.path.exists(src_img_dir + '/' + img + '.jpeg'):
            im = Image.open((src_img_dir + '/' + img + '.jpeg'))
            suffix = '.jpeg'
                  
      width, height = im.size

      """
      以下部分为 xml 解析
      """

      # open the crospronding txt file
      #  提取每一行并分割
      gt = open(src_txt_dir + '/' + img + '.txt').read().splitlines()

      print(img + '\n')
      # write in xml file
      # os.mknod(src_xml_dir + '/' + img + '.xml')
      xml_file = open((src_xml_dir + '/' + img + '.xml'), 'a')
      xml_file.write('<annotation>\n')
      xml_file.write('    <folder>VOC2007</folder>\n')
      xml_file.write('    <filename>' + str(img) + suffix + '</filename>\n')
      xml_file.write('    <size>\n')
      xml_file.write('        <width>' + str(width) + '</width>\n')
      xml_file.write('        <height>' + str(height) + '</height>\n')
      xml_file.write('        <depth>3</depth>\n')
      xml_file.write('    </size>\n')

      # write the region of image on xml file
      for img_each_label in gt:
            spt = img_each_label.split(' ')  # 这里如果txt里面是以逗号‘，’隔开的，那么就改为spt = img_each_label.split(',')。

            name = class_name[int(spt[0])]
            x_c,y_c,width_n,height_n = spt[1:]
            xmin,ymin,xmax,ymax = convert_yolo_coordinates_to_voc(x_c,y_c,width_n,height_n,width,height)


            xml_file.write('    <object>\n')
            xml_file.write('        <name>' + name + '</name>\n')
            xml_file.write('        <pose>Unspecified</pose>\n')
            xml_file.write('        <truncated>0</truncated>\n')
            xml_file.write('        <difficult>0</difficult>\n')
            xml_file.write('        <bndbox>\n')
            xml_file.write('            <xmin>' + str(xmin) + '</xmin>\n')
            xml_file.write('            <ymin>' + str(ymin) + '</ymin>\n')
            xml_file.write('            <xmax>' + str(xmax) + '</xmax>\n')
            xml_file.write('            <ymax>' + str(ymax) + '</ymax>\n')
            xml_file.write('        </bndbox>\n')
            xml_file.write('    </object>\n')

      xml_file.write('</annotation>')
```



# 使用 darknet 训练模型

## 安装和编译

```
# 从 Github 下载
git clone https://github.com/pjreddie/darknet
```

进入`darknet`目录中，对编译文件`Makefile`进行如下修改

```
GPU=1
CUDNN=1
OPENCV=0
OPENMP=0
DEBUG=0
```

注:

1. 画图或显示图片等操作需要配置OPENCV并设置为1
2. 需要多线程相关操作需要将OPENMP设置为1

修改完成保存并执行`make`命令。

## 目录结构介绍

```
.
├── backup
├── cfg
    |—— voc.data
    |—— yolov3-voc.cfg
├── darknet
├── data
    |—— voc.names
		|—— train.txt
    |—— val.txt
    |—— test.txt
├── scripts
...
```

编译完成之后，我们关注目录中的这几个文件和文件夹。

1. backup 存放训练出来的权值文件
2. cfg 保存配置文件，其中两个文件，在后面介绍
3. darknet 是可执行文件
4. scripts 下是一些脚本
5. data/voc.names 存放类别标签 
6. train.txt 保存用于训练的图片全路径(自建，位置无特殊要求)
7. val.txt 保存用于校正的图片全路径(自建，位置无特殊要求)
8. test.txt 保存用于校正的图片全路径(自建，位置无特殊要求)

*注: train.txt, val.txt, test.txt 中图片数量比例建议为 8:1:1*

## 修改配置文件

### 类别文件 voc.names

修改`data/voc.names`，将我们的类别标签写这个文件，比如说有以下 5 类

```
dog
cat
magpie
pigeon
nest
```

注意：

1. 类别标签要与训练集包含的图片类别一一对应，训练集中有以上 5 类则`voc.names`包含以上 5 类
2. 类别标签需要连续，如果不连续就必须要修改类别标签改成连续的

### 配置 voc.data

修改`cfg/voc.data`文件

```
classes= class_number
train  = /path/train.txt
valid  = /path/val.txt
names = data/voc.names
backup = backup
```

1. classes 配置类别数量，与上一步`voc.names`中类别数量一致
2. train 配置为目录结构一章介绍的`train.txt`文件的位置
3. vaild 配置为目录结构一章介绍的`val.txt`文件的位置
4. names 配置为`voc.names`位置(默认不变即可)
5. backup 配置为权值文件的位置(默认不变即可)

### 配置 yolov3-voc.cfg

在文件开头位置

```
[net]
# Testing
# batch=1 # 测试时开启，训练时关闭
# subdivisions=1 # 测试时开启，训练时关闭
# Training
batch=6 # 训练时开启，测试时关闭
subdivisions=2 # 训练时开启，测试时关闭 
...

learning_rate=0.0001 # 学习率，可以调整得小一点
burn_in=1000
max_batches = 50200
policy=steps
steps=40000,45000
scales=.1,.1
```

1. batch 和 subdivisions 在测试和训练的时候请按照上面注释开启或者关闭
2. batch 一批处理几张图片，没有超过显存的情况下越大越好
3. subdivisions 表示在 batch 中再划分的数量

当前配置的意思是每轮迭代从所有训练集中抽取 6 张图片，这 6 张样本图片又被分成 2 次，每次 3 张送入到网络参与训练。

接着在文件中搜索`yolo`，会有三条结果。每个`yolo`上下都要修改`filters`和`classes`，总共需要修改**3 组共 6 个字段**。

```
[convolutional]
size=1
stride=1
pad=1
filters=30 # 3*(classes+5)
activation=linear

[yolo]
mask = 6,7,8
anchors = 10,13,  16,30,  33,23,  30,61,  62,45,  59,119,  116,90,  156,198,  373,326
classes=5 # 类别(classes)数量
num=9
jitter=.3
ignore_thresh = .5
truth_thresh = 1
random=1
```

## 训练

### 方法一

```
./darknet detector train cfg/voc.data cfg/yolov3-voc.cfg scripts/darknet53.conv.74 -gpus 0
```

### 方法二

```
nohup ./darknet detector train cfg/voc.data cfg/yolov3-voc.cfg scripts/darknet53.conv.74 -gpus 0 >train.log 2>&1 &
```

我们需要保留训练的日志，所以用方法二更好。

## 查看 GPU 使用情况

我们可以使用`nvidia-smi`来查看 gpu 的使用情况，注意先查看 gpu 的使用情况。

```
Fri Mar 29 16:33:31 2019       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 410.78       Driver Version: 410.78       CUDA Version: 10.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  Off  | 00000000:01:00.0  On |                  N/A |
| 59%   87C    P2   221W / 250W |   4874MiB / 11175MiB |     98%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 108...  Off  | 00000000:02:00.0 Off |                  N/A |
| 24%   42C    P8    16W / 250W |   5107MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1218      G   /usr/lib/xorg/Xorg                            69MiB |
|    0     24338      C   ./darknet                                   4793MiB |
|    1     12712      C   ./bin/psd_be                                5095MiB |
+-----------------------------------------------------------------------------+
```

用`nvidia-smi`输出的是静态的信息，如果想要动态查看 gpu 的使用情况可以使用`watch -n 1 nvidia-smi`表示每一秒刷新一个 gpu 的使用情况。

## 查看日志输出信息

```
50181: 0.084739, 0.367084 avg, 0.000001 rate, 0.244283 seconds, 301086 images
Loaded: 0.000030 seconds
Region 82 Avg IOU: 0.775331, Class: 0.611054, Obj: 0.963920, No Obj: 0.005009, .5R: 1.000000, .75R: 0.833333,  count: 6
Region 94 Avg IOU: 0.753700, Class: 0.962260, Obj: 0.777823, No Obj: 0.000218, .5R: 1.000000, .75R: 0.500000,  count: 2
```

1. 50181 表示迭代次数
2. 0.084739 表示整体 Loss
3. 0.367084 avg 表示平均 Loss
4. 0.000001 rate 表示学习率，对应`.cfg`文件中的`learning_rate`
5. 0.244283 seconds 表示当前批次训练花费了多少时间
6. 301086 images 表示目前已经训练了多少照片

判断训练没有异常的标准

1. IOU 表示预测目标与真实目标的交集与并集之比，越接近于 1 越好，出现大量 -nan 表示训练异常
2. Class: 0.611054 表示标记物体的正确率，越接近于 1 越好
3. Obj: 0.963920 越接近 1 越好
4. No Obj 0.005009 越接近 0 越好

当训练整体趋势按照上面描述的一样向好的方向发展就表示训练正常，反之表示训练异常。

## 测试

```
./darknet detector test cfg/voc.data cfg/yolov3-voc.cfg backup/yolov3-voc_50000.weights 1
```

注意：测试时，修改`cfg/yolov3-voc.cfg`配置文件，请参照之前`配置 yolov3-voc.cfg`这一节。

## 其他

1. 训练集中各个类别的数量要尽可能**保持均衡**，不要有的类特别多或者有的特别少
2. `train.txt` 文件中配置了训练用的图片，这些图片必须要有对应的`txt`文件，且`txt`文件不可以为空
3. `train.txt` 文件不可以有空行，存在空行会导致训练失败
4. 开始新的训练注意删除`backup`中原先的`.weight`文件

# 如何评估模型的效果

## darknet 编译格式

```shell
./darknet detector test <data_cfg> <models_cfg> <weights> <test_file> [-thresh] [-out]
./darknet detector train <data_cfg> <models_cfg> <weights> [-thresh] [-gpu] [-gpus] [-clear]
./darknet detector valid <data_cfg> <models_cfg> <weights> [-out] [-thresh]
./darknet detector recall <data_cfg> <models_cfg> <weights> [-thresh]

'<>'必选项，’[ ]‘可选项
```

- data_cfg：数据配置文件，eg：cfg/voc.data
- models_cfg：模型配置文件，eg：cfg/yolov3-voc.cfg
- weights：权重配置文件，eg：weights/yolov3.weights
- test_file：测试文件，eg：*/*/*/test.txt
- -thresh：显示被检测物体中confidence大于等于 [-thresh] 的bounding-box，默认0.005
- -out：输出文件名称，默认路径为results文件夹下，eg：-out "" //输出class_num个文件，文件名为class_name.txt；若不选择此选项，则默认输出文件名为comp4_det_test_"class_name".txt
- -i/-gpu：指定单个gpu，默认为0，eg：-gpu 2
- -gpus：指定多个gpu，默认为0，eg：-gpus 0,1,2



## 根据训练日志生成 loss-iter 曲线

使用 drawcurve.py 解析训练日志，据此生成 loss-iter 曲线。该脚本通过训练日志计算 loss,不过训练日志格式可能会有区别,可能需要自己修改脚本来适应.

```python
# coding: utf-8

import argparse
import sys
import matplotlib.pyplot as plt
def main(argv):
    parser = argparse.ArgumentParser()
    parser.add_argument("log_file",  help = "path to log file"  )
    parser.add_argument( "option", help = "0 -> loss vs iter"  )
    args = parser.parse_args()
    f = open(args.log_file)
    lines  = [line.rstrip("\n") for line in f.readlines()]
    # skip the first 3 lines
    lines = lines[3:]
    numbers = {'1','2','3','4','5','6','7','8','9', '0'}
    iters = []
    loss = []
    for line in lines:
        print(line)
        
        #跳过空行
        if not len(line):
            continue

        if line[0] in numbers:
            args = line.split(" ")
            # print(args)
            if len(args) > 4 and is_number(args[2]):
                iters.append(int(args[0][:-1]))
                loss.append(float(args[2]))
    plt.plot(iters,loss)
    plt.xlabel('iters')
    plt.ylabel('loss')
    plt.grid()
    plt.show()
# 0.692735 seconds, 595200 images

def is_number(s):
    try:
        float(s)
        return True
    except ValueError:
        pass
 
    try:
        import unicodedata
        unicodedata.numeric(s)
        return True
    except (TypeError, ValueError):
        pass
 
    return False

if __name__ == "__main__":
    main(sys.argv)
```

![loss 曲线](/Users/shui/Desktop/yolov3_darknet/images/loss.png)

## 生成预测结果

通过`./darknet detector valid <data_cfg> <models_cfg> <weights>` 可以批量生成模型的测试结果,测试结果保存在`results`目录下面,按照类别分成一个个文件.

```shell
#!/bin/bash
./darknet detector valid switch_18/switch.data switch_18/switch.cfg switch_18/switch.weights -i 1 -out ""
```

- 结果生成在 <data_cfg> 的指定的目录下以 <out_file> 开头的若干文件中，若<data_cfg>没有指定results，那么默认为<darknet_root>/results；
- <models_cfg> 文件中 batch 和 subdivisions 两项必须为1；
- 若 -out 未指定字符串，则在 results 文件夹下生成 comp4_det_test_[类名].txt 文件并保存测试结果；
- 本次实验在 results 文件夹下生成  [类名].txt 文件；

## 统计召回率 recall

```shell
#!/bin/bash
./darknet detector recall switch_18/switch.data switch_18/switch.cfg switch_18/switch.weights
```

输出结果为

```
  835    35    45	RPs/Img: 1.81	IOU: 57.69%	Recall:77.78%
  836    35    45	RPs/Img: 1.81	IOU: 57.69%	Recall:77.78%
  837    35    45	RPs/Img: 1.81	IOU: 57.69%	Recall:77.78%
  838    35    45	RPs/Img: 1.81	IOU: 57.69%	Recall:77.78%
  839    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  840    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  841    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  842    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  843    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  844    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  845    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  846    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  847    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  848    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  849    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  850    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
  851    35    45	RPs/Img: 1.82	IOU: 57.69%	Recall:77.78%
```

数据格式为`Number Correct Total Rps/Img IOU Recall `

- Number表示处理到第几张图片。
- Correct 表示正确的识别除了多少bbox。这个值算出来的步骤是这样的，丢进网络一张图片，网络会预测出很多 bbox，每个bbox都有其置信概率，概率大于threshold的bbox与实际的bbox，也就是labels中txt的内容计算IOU，找出IOU最大的bbox，如果这个最大值大于预设的IOU的threshold，那么correct加一。
- Total表示实际有多少个bbox。
- Rps/img表示平均每个图片会预测出来多少个bbox。
- IOU： 这个是预测出的bbox和实际标注的bbox的交集 除以 他们的并集。显然，这个数值越大，说明预测的结果越好。
- Recall召回率， 意思是检测出物体的个数 除以 标注的所有物体个数。通过代码我们也能看出来就是Correct除以Total的值。

计算的是全部图片的 recall 和 IOU,而不是单类的,不是很方便.

## 计算 mAP

### reval_voc.py

```python
#!/usr/bin/env python

# Adapt from ->
# --------------------------------------------------------
# Fast R-CNN
# Copyright (c) 2015 Microsoft
# Licensed under The MIT License [see LICENSE for details]
# Written by Ross Girshick
# --------------------------------------------------------
# <- Written by Yaping Sun

"""Reval = re-eval. Re-evaluate saved detections."""

import os, sys, argparse
import numpy as np
import cPickle

from voc_eval import voc_eval

def parse_args():
    """
    Parse input arguments
    """
    parser = argparse.ArgumentParser(description='Re-evaluate results')
    parser.add_argument('output_dir', nargs=1, help='results directory',
                        type=str)
    parser.add_argument('--voc_dir', dest='voc_dir', default='data/VOCdevkit', type=str)
    parser.add_argument('--year', dest='year', default='2017', type=str)
    parser.add_argument('--image_set', dest='image_set', default='test', type=str)

    parser.add_argument('--classes', dest='class_file', default='data/voc.names', type=str)

    if len(sys.argv) == 1:
        parser.print_help()
        sys.exit(1)

    args = parser.parse_args()
    return args

def get_voc_results_file_template(image_set, out_dir = 'results'):
    filename = 'comp4_det_' + image_set + '_{:s}.txt'
    path = os.path.join(out_dir, filename)
    return path

def do_python_eval(devkit_path, year, image_set, classes, output_dir = 'results'):
    annopath = os.path.join(
        devkit_path,
        'VOC' + year,
        'Annotations',
        '{:s}.xml')
    imagesetfile = os.path.join(
        devkit_path,
        'VOC' + year,
        'ImageSets',
        'Main',
        image_set + '.txt')
    cachedir = os.path.join(devkit_path, 'annotations_cache')
    aps = []
    # The PASCAL VOC metric changed in 2010
    use_07_metric = True if int(year) < 2010 else False
    print('VOC07 metric? ' + ('Yes' if use_07_metric else 'No'))
    if not os.path.isdir(output_dir):
        os.mkdir(output_dir)
    for i, cls in enumerate(classes):
        if cls == '__background__':
            continue
        filename = get_voc_results_file_template(image_set).format(cls)
        rec, prec, ap = voc_eval(
            filename, annopath, imagesetfile, cls, cachedir, ovthresh=0.5,
            use_07_metric=use_07_metric)
        aps += [ap]
        print('AP for {} = {:.4f}'.format(cls, ap))
        with open(os.path.join(output_dir, cls + '_pr.pkl'), 'w') as f:
            cPickle.dump({'rec': rec, 'prec': prec, 'ap': ap}, f)
    print('Mean AP = {:.4f}'.format(np.mean(aps)))
    print('~~~~~~~~')
    print('Results:')
    for ap in aps:
        print('{:.3f}'.format(ap))
    print('{:.3f}'.format(np.mean(aps)))
    print('~~~~~~~~')
    print('')
    print('--------------------------------------------------------------')
    print('Results computed with the **unofficial** Python eval code.')
    print('Results should be very close to the official MATLAB eval code.')
    print('-- Thanks, The Management')
    print('--------------------------------------------------------------')



if __name__ == '__main__':
    args = parse_args()

    output_dir = os.path.abspath(args.output_dir[0])
    with open(args.class_file, 'r') as f:
        lines = f.readlines()

    classes = [t.strip('\n') for t in lines]

    print('Evaluating detections')
    do_python_eval(args.voc_dir, args.year, args.image_set, classes, output_dir)
```

### voc_eval.py

```python
# --------------------------------------------------------
# Fast/er R-CNN
# Licensed under The MIT License [see LICENSE for details]
# Written by Bharath Hariharan
# --------------------------------------------------------

import xml.etree.ElementTree as ET
import os
import pickle
import numpy as np

def parse_rec(filename):
    """ Parse a PASCAL VOC xml file """
    tree = ET.parse(filename)
    objects = []
    for obj in tree.findall('object'):
        obj_struct = {}
        obj_struct['name'] = obj.find('name').text
        obj_struct['pose'] = obj.find('pose').text
        obj_struct['truncated'] = int(obj.find('truncated').text)
        obj_struct['difficult'] = int(obj.find('difficult').text)
        bbox = obj.find('bndbox')
        obj_struct['bbox'] = [int(bbox.find('xmin').text),
                              int(bbox.find('ymin').text),
                              int(bbox.find('xmax').text),
                              int(bbox.find('ymax').text)]
        objects.append(obj_struct)

    return objects

def voc_ap(rec, prec, use_07_metric=False):
    """ ap = voc_ap(rec, prec, [use_07_metric])
    Compute VOC AP given precision and recall.
    If use_07_metric is true, uses the
    VOC 07 11 point method (default:False).
    """
    if use_07_metric:
        # 11 point metric
        ap = 0.
        for t in np.arange(0., 1.1, 0.1):
            if np.sum(rec >= t) == 0:
                p = 0
            else:
                p = np.max(prec[rec >= t])
            ap = ap + p / 11.
    else:
        # correct AP calculation
        # first append sentinel values at the end
        mrec = np.concatenate(([0.], rec, [1.]))
        mpre = np.concatenate(([0.], prec, [0.]))

        # compute the precision envelope
        for i in range(mpre.size - 1, 0, -1):
            mpre[i - 1] = np.maximum(mpre[i - 1], mpre[i])

        # to calculate area under PR curve, look for points
        # where X axis (recall) changes value
        i = np.where(mrec[1:] != mrec[:-1])[0]

        # and sum (\Delta recall) * prec
        ap = np.sum((mrec[i + 1] - mrec[i]) * mpre[i + 1])
    return ap

def voc_eval(detpath,
             annopath,
             imagesetfile,
             classname,
             cachedir,
             ovthresh=0.5,
             use_07_metric=False):
    """rec, prec, ap = voc_eval(detpath,
                                annopath,
                                imagesetfile,
                                classname,
                                [ovthresh],
                                [use_07_metric])
    Top level function that does the PASCAL VOC evaluation.
    detpath: Path to detections
        detpath.format(classname) should produce the detection results file.
    annopath: Path to annotations
        annopath.format(imagename) should be the xml annotations file.
    imagesetfile: Text file containing the list of images, one image per line.
    classname: Category name (duh)
    cachedir: Directory for caching the annotations
    [ovthresh]: Overlap threshold (default = 0.5)
    [use_07_metric]: Whether to use VOC07's 11 point AP computation
        (default False)
    """
    # assumes detections are in detpath.format(classname)
    # assumes annotations are in annopath.format(imagename)
    # assumes imagesetfile is a text file with each line an image name
    # cachedir caches the annotations in a pickle file

    # first load gt
    if not os.path.isdir(cachedir):
        os.mkdir(cachedir)
    cachefile = os.path.join(cachedir, 'annots.pkl')
    # read list of images
    with open(imagesetfile, 'r') as f:
        lines = f.readlines()
    imagenames = [x.strip() for x in lines]

    if not os.path.isfile(cachefile):
        # load annots
        recs = {}
        for i, imagename in enumerate(imagenames):
            recs[imagename] = parse_rec(annopath.format(imagename))
            if i % 100 == 0:
                print ('Reading annotation for {:d}/{:d}'.format(
                    i + 1, len(imagenames)))
        # save
        print ('Saving cached annotations to {:s}'.format(cachefile))
        with open(cachefile, 'wb') as f:
            pickle.dump(recs, f)
    else:
        # load
        with open(cachefile, 'rb') as f:
            recs = pickle.load(f)

    # extract gt objects for this class
    class_recs = {}
    npos = 0
    for imagename in imagenames:
        R = [obj for obj in recs[imagename] if obj['name'] == classname]
        difficult = np.array([x['difficult'] for x in R]).astype(np.bool)
        bbox = np.array([x['bbox'] for x in R])
        #difficult = np.array([x['difficult'] for x in R]).astype(np.bool)
        det = [False] * len(R)
        npos = npos + sum(~difficult)
        class_recs[imagename] = {'bbox': bbox,
                                 #'difficult': difficult,
                                 'det': det}

    # read dets
    detfile = detpath.format(classname)
    with open(detfile, 'r') as f:
        lines = f.readlines()

    splitlines = [x.strip().split(' ') for x in lines]
    image_ids = [x[0] for x in splitlines]
    confidence = np.array([float(x[1]) for x in splitlines])
    BB = np.array([[float(z) for z in x[2:]] for x in splitlines])

    # sort by confidence
    sorted_ind = np.argsort(-confidence)
    sorted_scores = np.sort(-confidence)
    BB = BB[sorted_ind, :]
    image_ids = [image_ids[x] for x in sorted_ind]

    # go down dets and mark TPs and FPs
    nd = len(image_ids)
    tp = np.zeros(nd)
    fp = np.zeros(nd)
    for d in range(nd):
        R = class_recs[image_ids[d]]
        bb = BB[d, :].astype(float)
        ovmax = -np.inf
        BBGT = R['bbox'].astype(float)

        if BBGT.size > 0:
            # compute overlaps
            # intersection
            ixmin = np.maximum(BBGT[:, 0], bb[0])
            iymin = np.maximum(BBGT[:, 1], bb[1])
            ixmax = np.minimum(BBGT[:, 2], bb[2])
            iymax = np.minimum(BBGT[:, 3], bb[3])
            iw = np.maximum(ixmax - ixmin + 1., 0.)
            ih = np.maximum(iymax - iymin + 1., 0.)
            inters = iw * ih

            # union
            uni = ((bb[2] - bb[0] + 1.) * (bb[3] - bb[1] + 1.) +
                   (BBGT[:, 2] - BBGT[:, 0] + 1.) *
                   (BBGT[:, 3] - BBGT[:, 1] + 1.) - inters)

            overlaps = inters / uni
            ovmax = np.max(overlaps)
            jmax = np.argmax(overlaps)
        """
        if ovmax > ovthresh:

            if not False:
                if not R['det'][jmax]:
                    tp[d] = 1.
                    R['det'][jmax] = 1
                else:
                    fp[d] = 1.
        else:
            fp[d] = 1.
        """
        if ovmax > ovthresh:

            if not False:
                if not R['det'][jmax]:
                    tp[d] = 1.
                    R['det'][jmax] = 1
                else:
                    fp[d] = 1.
        else:
            fp[d] = 1.

    # compute precision recall
    fp = np.cumsum(fp)
    tp = np.cumsum(tp)
    rec = tp / float(npos)
    # avoid divide by zero in case the first detection matches a difficult
    # ground truth
    prec = tp / np.maximum(tp + fp, np.finfo(np.float64).eps)
    ap = voc_ap(rec, prec, use_07_metric)

    return rec, prec, ap
```

### computer_Single_ALL_mAP.py

- results_path = "填写前面生成的 results 文件的路径"
- /xxx/results/{}.txt - results_path 的路径
- /xxx/{}.xml - xml 格式的标注文件
- /xxx/list.txt - 只包含图片名字的列表文件

执行 computer_Single_ALL_mAP.py 将会生成各个分类的 mAP.

```python
from voc_eval import voc_eval 
import os 

current_path = os.getcwd()

results_path = "填写前面生成的 results 文件的路径"
sub_files = os.listdir(results_path) 

mAP = [] 

for i in range(len(sub_files)): 
    class_name = sub_files[i].split(".txt")[0]
    rec, prec, ap = voc_eval('/xxx/results/{}.txt', '/xxx/{}.xml', '/xxx/list.txt', class_name, '.')
    # print("{} :\t {} ".format(class_name, ap)) 
    print("{} :\t {} ".format(class_name, ap)) 
    mAP.append(ap) 
 
# class_name = 'switch_open'
# rec, prec, ap = voc_eval('/aseit-data/program/darknet/results/{}.txt', '/aseit-data/data_set/swich/dataset_test/switch_mAP/{}.xml', '/aseit-data/data_set/swich/dataset_test/list.txt', class_name, '.')
# print("{} :\t {} ".format(class_name, ap)) 
# mAP.append(ap) 

mAP = tuple(mAP) 
print("***************************") 
print("mAP :\t {}".format( float( sum(mAP)/len(mAP)) ))
```



## detector.c

在 test 和计算 recall 的时候需要修改 examples/detector.c 如下:

```c
#include <unistd.h>
#include <fcntl.h>

#include "darknet.h"
#include <sys/stat.h>
#include <stdio.h>
#include <time.h>
#include <sys/types.h>

static int coco_ids[] = {1,2,3,4,5,6,7,8,9,10,11,13,14,15,16,17,18,19,20,21,22,23,24,25,27,28,31,32,33,34,35,36,37,38,39,40,
41,42,43,44,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,67,70,72,73,74,75,76,77,78,79,80,81,82,84,85,86,87,88,89,90};

char *GetFilename(char *fullname)
{
    int from,to,i;
    char *newstr,*temp;
    if(fullname!=NULL){
        //if not find dot
        if((temp=strchr(fullname,'.'))==NULL){
        newstr = fullname;
        }
        else
        {
            from = strlen(fullname) - strlen(temp);
            to = (temp-fullname);
            //the first dot's index
            for (i=from; i<=to; i--){
                if (fullname[i]=='.') break;//find the last dot
            }
            newstr = (char*)malloc(i+1);
            strncpy(newstr,fullname,i);
            *(newstr+i)=0;
        }
    }
    static char name[50] = {""};
    char *q = strrchr(newstr,'/') + 1;
    strncpy(name,q,40);
    return name;
}

void train_detector(char *datacfg, char *cfgfile, char *weightfile, int *gpus, int ngpus, int clear)
{
    list *options = read_data_cfg(datacfg); //解析data文件，用自定义链表options存储训练集基本信息，函数位于option_list.c
    char *train_images = option_find_str(options, "train", "data/train.list");  //从options中找训练集
    char *backup_directory = option_find_str(options, "backup", "/backup/");    //从options中找backup路径

    srand(time(0)); //初始化随机种子数
    char *base = basecfg(cfgfile);  //此函数位于utils.c,返回cfg文件不带后缀的名字
    printf("%s\n", base);
    float avg_loss = -1;
    network **nets = calloc(ngpus, sizeof(network));

    srand(time(0));
    int seed = rand();
    int i;
    for(i = 0; i < ngpus; ++i){
        srand(seed);
#ifdef GPU
        cuda_set_device(gpus[i]);
#endif
        nets[i] = load_network(cfgfile, weightfile, clear);
        nets[i]->learning_rate *= ngpus;
    }
    srand(time(0));
    network *net = nets[0];

    int imgs = net->batch * net->subdivisions * ngpus;
    printf("Learning Rate: %g, Momentum: %g, Decay: %g\n", net->learning_rate, net->momentum, net->decay);
    data train, buffer;

    layer l = net->layers[net->n - 1];

    int classes = l.classes;
    float jitter = l.jitter;

    list *plist = get_paths(train_images);
    //int N = plist->size;
    char **paths = (char **)list_to_array(plist);

    load_args args = get_base_args(net);
    args.coords = l.coords;
    args.paths = paths;
    args.n = imgs;
    args.m = plist->size;
    args.classes = classes;
    args.jitter = jitter;
    args.num_boxes = l.max_boxes;
    args.d = &buffer;
    args.type = DETECTION_DATA;
    //args.type = INSTANCE_DATA;
    args.threads = 64;

    pthread_t load_thread = load_data(args);
    double time;
    int count = 0;
    //while(i*imgs < N*120){
    while(get_current_batch(net) < net->max_batches){
        if(l.random && count++%10 == 0){
            printf("Resizing\n");
            int dim = (rand() % 10 + 10) * 32;
            if (get_current_batch(net)+200 > net->max_batches) dim = 608;
            //int dim = (rand() % 4 + 16) * 32;
            printf("%d\n", dim);
            args.w = dim;
            args.h = dim;

            pthread_join(load_thread, 0);
            train = buffer;
            free_data(train);
            load_thread = load_data(args);

            #pragma omp parallel for
            for(i = 0; i < ngpus; ++i){
                resize_network(nets[i], dim, dim);
            }
            net = nets[0];
        }
        time=what_time_is_it_now();
        pthread_join(load_thread, 0);
        train = buffer;
        load_thread = load_data(args);

        /*
           int k;
           for(k = 0; k < l.max_boxes; ++k){
           box b = float_to_box(train.y.vals[10] + 1 + k*5);
           if(!b.x) break;
           printf("loaded: %f %f %f %f\n", b.x, b.y, b.w, b.h);
           }
         */
        /*
           int zz;
           for(zz = 0; zz < train.X.cols; ++zz){
           image im = float_to_image(net->w, net->h, 3, train.X.vals[zz]);
           int k;
           for(k = 0; k < l.max_boxes; ++k){
           box b = float_to_box(train.y.vals[zz] + k*5, 1);
           printf("%f %f %f %f\n", b.x, b.y, b.w, b.h);
           draw_bbox(im, b, 1, 1,0,0);
           }
           show_image(im, "truth11");
           cvWaitKey(0);
           save_image(im, "truth11");
           }
         */

        printf("Loaded: %lf seconds\n", what_time_is_it_now()-time);

        time=what_time_is_it_now();
        float loss = 0;
#ifdef GPU
        if(ngpus == 1){
            loss = train_network(net, train);
        } else {
            loss = train_networks(nets, ngpus, train, 4);
        }
#else
        loss = train_network(net, train);
#endif
        if (avg_loss < 0) avg_loss = loss;
        avg_loss = avg_loss*.9 + loss*.1;

        i = get_current_batch(net);
        printf("%ld: %f, %f avg, %f rate, %lf seconds, %d images\n", get_current_batch(net), loss, avg_loss, get_current_rate(net), what_time_is_it_now()-time, i*imgs);
        if(i%100==0){
#ifdef GPU
            if(ngpus != 1) sync_nets(nets, ngpus, 0);
#endif
            char buff[256];
            sprintf(buff, "%s/%s.backup", backup_directory, base);
            save_weights(net, buff);
        }
        if(i%10000==0 || (i < 1000 && i%100 == 0)){
#ifdef GPU
            if(ngpus != 1) sync_nets(nets, ngpus, 0);
#endif
            char buff[256];
            sprintf(buff, "%s/%s_%d.weights", backup_directory, base, i);
            save_weights(net, buff);
        }
        free_data(train);
    }
#ifdef GPU
    if(ngpus != 1) sync_nets(nets, ngpus, 0);
#endif
    char buff[256];
    sprintf(buff, "%s/%s_final.weights", backup_directory, base);
    save_weights(net, buff);
}


static int get_coco_image_id(char *filename)
{
    char *p = strrchr(filename, '/');
    char *c = strrchr(filename, '_');
    if(c) p = c;
    return atoi(p+1);
}

static void print_cocos(FILE *fp, char *image_path, detection *dets, int num_boxes, int classes, int w, int h)
{
    int i, j;
    int image_id = get_coco_image_id(image_path);
    for(i = 0; i < num_boxes; ++i){
        float xmin = dets[i].bbox.x - dets[i].bbox.w/2.;
        float xmax = dets[i].bbox.x + dets[i].bbox.w/2.;
        float ymin = dets[i].bbox.y - dets[i].bbox.h/2.;
        float ymax = dets[i].bbox.y + dets[i].bbox.h/2.;

        if (xmin < 0) xmin = 0;
        if (ymin < 0) ymin = 0;
        if (xmax > w) xmax = w;
        if (ymax > h) ymax = h;

        float bx = xmin;
        float by = ymin;
        float bw = xmax - xmin;
        float bh = ymax - ymin;

        for(j = 0; j < classes; ++j){
            if (dets[i].prob[j]) fprintf(fp, "{\"image_id\":%d, \"category_id\":%d, \"bbox\":[%f, %f, %f, %f], \"score\":%f},\n", image_id, coco_ids[j], bx, by, bw, bh, dets[i].prob[j]);
        }
    }
}

void print_detector_detections(FILE **fps, char *id, detection *dets, int total, int classes, int w, int h)
{
    int i, j;
    for(i = 0; i < total; ++i){
        float xmin = dets[i].bbox.x - dets[i].bbox.w/2. + 1;
        float xmax = dets[i].bbox.x + dets[i].bbox.w/2. + 1;
        float ymin = dets[i].bbox.y - dets[i].bbox.h/2. + 1;
        float ymax = dets[i].bbox.y + dets[i].bbox.h/2. + 1;

        if (xmin < 1) xmin = 1;
        if (ymin < 1) ymin = 1;
        if (xmax > w) xmax = w;
        if (ymax > h) ymax = h;

        for(j = 0; j < classes; ++j){
            if (dets[i].prob[j]) fprintf(fps[j], "%s %f %f %f %f %f\n", id, dets[i].prob[j],
                    xmin, ymin, xmax, ymax);
        }
    }
}

void print_imagenet_detections(FILE *fp, int id, detection *dets, int total, int classes, int w, int h)
{
    int i, j;
    for(i = 0; i < total; ++i){
        float xmin = dets[i].bbox.x - dets[i].bbox.w/2.;
        float xmax = dets[i].bbox.x + dets[i].bbox.w/2.;
        float ymin = dets[i].bbox.y - dets[i].bbox.h/2.;
        float ymax = dets[i].bbox.y + dets[i].bbox.h/2.;

        if (xmin < 0) xmin = 0;
        if (ymin < 0) ymin = 0;
        if (xmax > w) xmax = w;
        if (ymax > h) ymax = h;

        for(j = 0; j < classes; ++j){
            int class = j;
            if (dets[i].prob[class]) fprintf(fp, "%d %d %f %f %f %f %f\n", id, j+1, dets[i].prob[class],
                    xmin, ymin, xmax, ymax);
        }
    }
}

void validate_detector_flip(char *datacfg, char *cfgfile, char *weightfile, char *outfile)
{
    int j;
    list *options = read_data_cfg(datacfg);
    char *valid_images = option_find_str(options, "valid", "data/valid.list");
    char *name_list = option_find_str(options, "names", "data/names.list");
    char *prefix = option_find_str(options, "results", "results");
    char **names = get_labels(name_list);
    char *mapf = option_find_str(options, "map", 0);
    int *map = 0;
    if (mapf) map = read_map(mapf);

    network *net = load_network(cfgfile, weightfile, 0);
    set_batch_network(net, 2);
    fprintf(stderr, "Learning Rate: %g, Momentum: %g, Decay: %g\n", net->learning_rate, net->momentum, net->decay);
    srand(time(0));

    list *plist = get_paths(valid_images);
    char **paths = (char **)list_to_array(plist);

    layer l = net->layers[net->n-1];
    int classes = l.classes;

    char buff[1024];
    char *type = option_find_str(options, "eval", "voc");
    FILE *fp = 0;
    FILE **fps = 0;
    int coco = 0;
    int imagenet = 0;
    if(0==strcmp(type, "coco")){
        if(!outfile) outfile = "coco_results";
        snprintf(buff, 1024, "%s/%s.json", prefix, outfile);
        fp = fopen(buff, "w");
        fprintf(fp, "[\n");
        coco = 1;
    } else if(0==strcmp(type, "imagenet")){
        if(!outfile) outfile = "imagenet-detection";
        snprintf(buff, 1024, "%s/%s.txt", prefix, outfile);
        fp = fopen(buff, "w");
        imagenet = 1;
        classes = 200;
    } else {
        if(!outfile) outfile = "comp4_det_test_";
        fps = calloc(classes, sizeof(FILE *));
        for(j = 0; j < classes; ++j){
            snprintf(buff, 1024, "%s/%s%s.txt", prefix, outfile, names[j]);
            fps[j] = fopen(buff, "w");
        }
    }

    int m = plist->size;
    int i=0;
    int t;

    float thresh = .95;
    float nms = .45;

    int nthreads = 4;
    image *val = calloc(nthreads, sizeof(image));
    image *val_resized = calloc(nthreads, sizeof(image));
    image *buf = calloc(nthreads, sizeof(image));
    image *buf_resized = calloc(nthreads, sizeof(image));
    pthread_t *thr = calloc(nthreads, sizeof(pthread_t));

    image input = make_image(net->w, net->h, net->c*2);

    load_args args = {0};
    args.w = net->w;
    args.h = net->h;
    //args.type = IMAGE_DATA;
    args.type = LETTERBOX_DATA;

    for(t = 0; t < nthreads; ++t){
        args.path = paths[i+t];
        args.im = &buf[t];
        args.resized = &buf_resized[t];
        thr[t] = load_data_in_thread(args);
    }
    double start = what_time_is_it_now();
    for(i = nthreads; i < m+nthreads; i += nthreads){
        fprintf(stderr, "%d\n", i);
        for(t = 0; t < nthreads && i+t-nthreads < m; ++t){
            pthread_join(thr[t], 0);
            val[t] = buf[t];
            val_resized[t] = buf_resized[t];
        }
        for(t = 0; t < nthreads && i+t < m; ++t){
            args.path = paths[i+t];
            args.im = &buf[t];
            args.resized = &buf_resized[t];
            thr[t] = load_data_in_thread(args);
        }
        for(t = 0; t < nthreads && i+t-nthreads < m; ++t){
            char *path = paths[i+t-nthreads];
            char *id = basecfg(path);
            copy_cpu(net->w*net->h*net->c, val_resized[t].data, 1, input.data, 1);
            flip_image(val_resized[t]);
            copy_cpu(net->w*net->h*net->c, val_resized[t].data, 1, input.data + net->w*net->h*net->c, 1);

            network_predict(net, input.data);
            int w = val[t].w;
            int h = val[t].h;
            int num = 0;
            detection *dets = get_network_boxes(net, w, h, thresh, .5, map, 0, &num);
            if (nms) do_nms_sort(dets, num, classes, nms);
            if (coco){
                print_cocos(fp, path, dets, num, classes, w, h);
            } else if (imagenet){
                print_imagenet_detections(fp, i+t-nthreads+1, dets, num, classes, w, h);
            } else {
                print_detector_detections(fps, id, dets, num, classes, w, h);
            }
            free_detections(dets, num);
            free(id);
            free_image(val[t]);
            free_image(val_resized[t]);
        }
    }
    for(j = 0; j < classes; ++j){
        if(fps) fclose(fps[j]);
    }
    if(coco){
        fseek(fp, -2, SEEK_CUR); 
        fprintf(fp, "\n]\n");
        fclose(fp);
    }
    fprintf(stderr, "Total Detection Time: %f Seconds\n", what_time_is_it_now() - start);
}


void validate_detector(char *datacfg, char *cfgfile, char *weightfile, char *outfile)
{
    int j;
    list *options = read_data_cfg(datacfg);
    char *valid_images = option_find_str(options, "valid", "data/valid.list");
    char *name_list = option_find_str(options, "names", "data/voc.names");
    char *prefix = option_find_str(options, "results", "results");
    char **names = get_labels(name_list);
    char *mapf = option_find_str(options, "map", 0);
    int *map = 0;
    if (mapf) map = read_map(mapf);

    network *net = load_network(cfgfile, weightfile, 0);
    set_batch_network(net, 1);
    fprintf(stderr, "Learning Rate: %g, Momentum: %g, Decay: %g\n", net->learning_rate, net->momentum, net->decay);
    srand(time(0));

    list *plist = get_paths(valid_images);
    char **paths = (char **)list_to_array(plist);

    layer l = net->layers[net->n-1];
    int classes = l.classes;

    char buff[1024];
    char *type = option_find_str(options, "eval", "voc");
    FILE *fp = 0;
    FILE **fps = 0;
    int coco = 0;
    int imagenet = 0;
    if(0==strcmp(type, "coco")){
        if(!outfile) outfile = "coco_results";
        snprintf(buff, 1024, "%s/%s.json", prefix, outfile);
        fp = fopen(buff, "w");
        fprintf(fp, "[\n");
        coco = 1;
    } else if(0==strcmp(type, "imagenet")){
        if(!outfile) outfile = "imagenet-detection";
        snprintf(buff, 1024, "%s/%s.txt", prefix, outfile);
        fp = fopen(buff, "w");
        imagenet = 1;
        classes = 200;
    } else {
        if(!outfile) outfile = "comp4_det_test_";
        fps = calloc(classes, sizeof(FILE *));
        for(j = 0; j < classes; ++j){
            snprintf(buff, 1024, "%s/%s%s.txt", prefix, outfile, names[j]);
            fps[j] = fopen(buff, "w");
        }
    }


    int m = plist->size;
    int i=0;
    int t;

    float thresh = .95;
    float nms = .45;

    int nthreads = 4;
    image *val = calloc(nthreads, sizeof(image));
    image *val_resized = calloc(nthreads, sizeof(image));
    image *buf = calloc(nthreads, sizeof(image));
    image *buf_resized = calloc(nthreads, sizeof(image));
    pthread_t *thr = calloc(nthreads, sizeof(pthread_t));

    load_args args = {0};
    args.w = net->w;
    args.h = net->h;
    //args.type = IMAGE_DATA;
    args.type = LETTERBOX_DATA;

    for(t = 0; t < nthreads; ++t){
        args.path = paths[i+t];
        args.im = &buf[t];
        args.resized = &buf_resized[t];
        thr[t] = load_data_in_thread(args);
    }
    double start = what_time_is_it_now();
    for(i = nthreads; i < m+nthreads; i += nthreads){
        fprintf(stderr, "%d\n", i);
        for(t = 0; t < nthreads && i+t-nthreads < m; ++t){
            pthread_join(thr[t], 0);
            val[t] = buf[t];
            val_resized[t] = buf_resized[t];
        }
        for(t = 0; t < nthreads && i+t < m; ++t){
            args.path = paths[i+t];
            args.im = &buf[t];
            args.resized = &buf_resized[t];
            thr[t] = load_data_in_thread(args);
        }
        for(t = 0; t < nthreads && i+t-nthreads < m; ++t){
            char *path = paths[i+t-nthreads];
            char *id = basecfg(path);
            float *X = val_resized[t].data;
            network_predict(net, X);
            int w = val[t].w;
            int h = val[t].h;
            int nboxes = 0;
            detection *dets = get_network_boxes(net, w, h, thresh, .5, map, 0, &nboxes);
            if (nms) do_nms_sort(dets, nboxes, classes, nms);
            if (coco){
                print_cocos(fp, path, dets, nboxes, classes, w, h);
            } else if (imagenet){
                print_imagenet_detections(fp, i+t-nthreads+1, dets, nboxes, classes, w, h);
            } else {
                print_detector_detections(fps, id, dets, nboxes, classes, w, h);
            }
            free_detections(dets, nboxes);
            free(id);
            free_image(val[t]);

            free_image(val_resized[t]);
        }
    }
    for(j = 0; j < classes; ++j){
        if(fps) fclose(fps[j]);
    }
    if(coco){
        fseek(fp, -2, SEEK_CUR); 
        fprintf(fp, "\n]\n");
        fclose(fp);
    }
    fprintf(stderr, "Total Detection Time: %f Seconds\n", what_time_is_it_now() - start);
}

void validate_detector_recall(char *datacfg, char *cfgfile, char *weightfile)
{
    network *net = load_network(cfgfile, weightfile, 0);
    set_batch_network(net, 1);
    fprintf(stderr, "Learning Rate: %g, Momentum: %g, Decay: %g\n", net->learning_rate, net->momentum, net->decay);
    srand(time(0));

    list *options = read_data_cfg(datacfg);
    char *valid_images = option_find_str(options, "valid", "data/valid.list");
    list *plist = get_paths(valid_images);
    char **paths = (char **)list_to_array(plist);

    //layer l = net->layers[net->n-1];

    int j, k;

    int m = plist->size;    //测试的图片总数
    int i=0;

    float thresh = .95;
    float iou_thresh = .5;
    float nms = .4;

    int total = 0;      //实际有多少个bbox
    int correct = 0;    //正确识别出了多少个bbox
    int proposals = 0;  //测试集预测的bbox总数
    float avg_iou = 0;
    
    //printf("l.w*l.h*l.n = %d\n",l.w*l.h*l.n);
    for(i = 0; i < m; ++i){
        char *path = paths[i];
        image orig = load_image_color(path, 0, 0);
        image sized = resize_image(orig, net->w, net->h);
        char *id = basecfg(path);
        network_predict(net, sized.data);
        int nboxes = 0;
        detection *dets = get_network_boxes(net, sized.w, sized.h, thresh, .5, 0, 1, &nboxes);
        if (nms) do_nms_obj(dets, nboxes, 1, nms);

        char labelpath[4096];
        find_replace(path, "images", "labels", labelpath);
        find_replace(labelpath, "JPEGImages", "labels", labelpath);
        find_replace(labelpath, ".jpg", ".txt", labelpath);
        find_replace(labelpath, ".JPEG", ".txt", labelpath);

        int num_labels = 0;     //测试集实际的标注框数量
        box_label *truth = read_boxes(labelpath, &num_labels);
        for(k = 0; k < nboxes; ++k){
            if(dets[k].objectness > thresh){
                ++proposals;
            }
        }
        for (j = 0; j < num_labels; ++j) {
            ++total;
            box t = {truth[j].x, truth[j].y, truth[j].w, truth[j].h};
            //printf("truth x=%f, y=%f, w=%f,h=%f\n",truth[j].x,truth[j].y,truth[j].w,truth[j].h);
            float best_iou = 0;
            //对每一个标注的框
            for(k = 0; k < nboxes; ++k){
            //for(k = 0; k < l.w*l.h*l.n; ++k){
                float iou = box_iou(dets[k].bbox, t);
                //printf("predict=%f iou=%f x=%f, y=%f, w=%f,h=%f\n",dets[k].objectness,iou,dets[k].bbox.x,dets[k].bbox.y,dets[k].bbox.w,dets[k].bbox.h);
                if(dets[k].objectness > thresh && iou > best_iou){
                    best_iou = iou;
                }
            }
            //printf("best_iou=%f\n",best_iou);
            avg_iou += best_iou;
            if(best_iou > iou_thresh){
                ++correct;
            }
        }
        fprintf(stderr, "%5d %5d %5d\tRPs/Img: %.2f\tIOU: %.2f%%\tRecall:%.2f%%\n", i, correct, total, (float)proposals/(i+1), avg_iou*100/total, 100.*correct/total);
        free(id);
        free_image(orig);
        free_image(sized);
    }
}

void test_detector(char *datacfg, char *cfgfile, char *weightfile, char *filename, float thresh, float hier_thresh, char *outfile, int fullscreen)
{
    list *options = read_data_cfg(datacfg); //options存储分类的标签等基本训练信息
    char *name_list = option_find_str(options, "names", "data/names.list"); //抽取标签名称
    char **names = get_labels(name_list);

    image **alphabet = load_alphabet(); //加载位于data/labels下的字符图片，用于显示矩形框名称
    network *net = load_network(cfgfile, weightfile, 0);    //用netweork.h中自定义的network结构体存储模型文件,函数位于parser.c
    set_batch_network(net, 1);
    srand(2222222);
    double start_time;
    double end_time;
    double img_time;
    double sum_time=0.0;
    char buff[256];
    char *input = buff;
    float nms=.45;
    int i=0;
    while(1){
        //读取结构对应的权重文件
        if(filename){
            strncpy(input, filename, 256);
            image im = load_image_color(input,0,0);
            image sized = letterbox_image(im, net->w, net->h);  //输入图片大小经过resize至输入大小
            //image sized = resize_image(im, net->w, net->h);
            //image sized2 = resize_max(im, net->w);
            //image sized = crop_image(sized2, -((net->w - sized2.w)/2), -((net->h - sized2.h)/2), net->w, net->h);
            //resize_network(net, sized.w, sized.h);
            layer l = net->layers[net->n-1];

            float *X = sized.data;  //X指向图片的data元素，即图片像素
            start_time=what_time_is_it_now();
            network_predict(net, X);    //network_predict函数负责预测当前图片的数据X
            end_time=what_time_is_it_now();
            img_time= end_time - start_time;

            printf("%s: Predicted in %f seconds.\n", input, img_time);
            int nboxes = 0;
            detection *dets = get_network_boxes(net, im.w, im.h, thresh, hier_thresh, 0, 1, &nboxes);
            //printf("%d\n", nboxes);
            //if (nms) do_nms_obj(boxes, probs, l.w*l.h*l.n, l.classes, nms);
            if (nms) do_nms_sort(dets, nboxes, l.classes, nms);
            draw_detections(im, dets, nboxes, thresh, names, alphabet, l.classes);
            free_detections(dets, nboxes);
            if(outfile)
             {
                save_image(im, outfile);
             }
            else{
                //save_image(im, "predictions");
                char image[2048];
                sprintf(image,"./data/predict/%s",GetFilename(filename));
                save_image(im,image);
                printf("predict %s successfully!\n",GetFilename(filename));
#ifdef OPENCV
                cvNamedWindow("predictions", CV_WINDOW_NORMAL);
                if(fullscreen){
                cvSetWindowProperty("predictions", CV_WND_PROP_FULLSCREEN, CV_WINDOW_FULLSCREEN);
                }
                show_image(im, "predictions");
                cvWaitKey(0);
                cvDestroyAllWindows();
#endif
            }
            free_image(im);
            free_image(sized);
            if (filename) break;
         }
        else {
            printf("Enter Image Path: ");
            fflush(stdout);
            input = fgets(input, 256, stdin);
            if(!input) return;
            strtok(input, "\n");
    
            list *plist = get_paths(input);
            char **paths = (char **)list_to_array(plist);
            printf("Start Testing!\n");
            int m = plist->size;
            if(access("/aseit-data/XCM_WorkSpace/darknet_test/darknet/test_out",0)==-1)//修改成自己的路径
            {
              if (mkdir("/aseit-data/XCM_WorkSpace/darknet_test/darknet/test_out",0777))//修改成自己的路径
               {
                 printf("creat file bag failed!!!");
               }
            }
            for(i = 0; i < m; ++i){
                char *path = paths[i];
                image im = load_image_color(path,0,0);
                image sized = letterbox_image(im, net->w, net->h);  //输入图片大小经过resize至输入大小
                //image sized = resize_image(im, net->w, net->h);
                //image sized2 = resize_max(im, net->w);
                //image sized = crop_image(sized2, -((net->w - sized2.w)/2), -((net->h - sized2.h)/2), net->w, net->h);
                //resize_network(net, sized.w, sized.h);
                layer l = net->layers[net->n-1];

                float *X = sized.data;  //X指向图片的data元素，即图片像素
                start_time = what_time_is_it_now();
                network_predict(net, X);    //network_predict函数负责预测当前图片的数据X
                end_time = what_time_is_it_now();
                img_time = end_time - start_time;
                sum_time = sum_time+img_time;
                printf("Try Very Hard:");
                printf("%s: Predicted in %f seconds.\n", path, img_time);
                int nboxes = 0;
                detection *dets = get_network_boxes(net, im.w, im.h, thresh, hier_thresh, 0, 1, &nboxes);
                //printf("%d\n", nboxes);
                //if (nms) do_nms_obj(boxes, probs, l.w*l.h*l.n, l.classes, nms);
                if (nms) do_nms_sort(dets, nboxes, l.classes, nms);
                draw_detections(im, dets, nboxes, thresh, names, alphabet, l.classes);
                free_detections(dets, nboxes);
                if(outfile){
                    save_image(im, outfile);
                }
                else{
                    char b[2048];
                    sprintf(b,"/aseit-data/XCM_WorkSpace/darknet_test/darknet/test_out/%s",GetFilename(path));//修改成自己的路径
                     
                    save_image(im, b);
                    printf("save %s successfully!\n",GetFilename(path));
#ifdef OPENCV
                    cvNamedWindow("predictions", CV_WINDOW_NORMAL);
                    if(fullscreen){
                        cvSetWindowProperty("predictions", CV_WND_PROP_FULLSCREEN, CV_WINDOW_FULLSCREEN);
                    }
                    show_image(im, "predictions");
                    cvWaitKey(0);
                    cvDestroyAllWindows();
#endif
        }
        free_image(im);
        free_image(sized);
        if (filename) break;
        }
        printf("fps: %.2f   totall image %d\n",(float)m/sum_time,m);
      }
    }
}

/*
void test_detector(char *datacfg, char *cfgfile, char *weightfile, char *filename, float thresh, float hier_thresh, char *outfile, int fullscreen)
{
    list *options = read_data_cfg(datacfg);
    char *name_list = option_find_str(options, "names", "data/names.list");
    char **names = get_labels(name_list);

    image **alphabet = load_alphabet();
    network *net = load_network(cfgfile, weightfile, 0);
    set_batch_network(net, 1);
    srand(2222222);
    double time;
    char buff[256];
    char *input = buff;
    float nms=.45;
    while(1){
        if(filename){
            strncpy(input, filename, 256);
        } else {
            printf("Enter Image Path: ");
            fflush(stdout);
            input = fgets(input, 256, stdin);
            if(!input) return;
            strtok(input, "\n");
        }
        image im = load_image_color(input,0,0);
        image sized = letterbox_image(im, net->w, net->h);
        //image sized = resize_image(im, net->w, net->h);
        //image sized2 = resize_max(im, net->w);
        //image sized = crop_image(sized2, -((net->w - sized2.w)/2), -((net->h - sized2.h)/2), net->w, net->h);
        //resize_network(net, sized.w, sized.h);
        layer l = net->layers[net->n-1];


        float *X = sized.data;
        time=what_time_is_it_now();
        network_predict(net, X);
        printf("%s: Predicted in %f seconds.\n", input, what_time_is_it_now()-time);
        int nboxes = 0;
        detection *dets = get_network_boxes(net, im.w, im.h, thresh, hier_thresh, 0, 1, &nboxes);
        //printf("%d\n", nboxes);
        //if (nms) do_nms_obj(boxes, probs, l.w*l.h*l.n, l.classes, nms);
        if (nms) do_nms_sort(dets, nboxes, l.classes, nms);
        draw_detections(im, dets, nboxes, thresh, names, alphabet, l.classes);
        free_detections(dets, nboxes);
        if(outfile){
            save_image(im, outfile);
        }
        else{
            save_image(im, "predictions");
#ifdef OPENCV
            make_window("predictions", 512, 512, 0);
            show_image(im, "predictions", 0);
#endif
        }

        free_image(im);
        free_image(sized);
        if (filename) break;
    }
}


void censor_detector(char *datacfg, char *cfgfile, char *weightfile, int cam_index, const char *filename, int class, float thresh, int skip)
{
#ifdef OPENCV
    char *base = basecfg(cfgfile);
    network *net = load_network(cfgfile, weightfile, 0);
    set_batch_network(net, 1);

    srand(2222222);
    CvCapture * cap;

    int w = 1280;
    int h = 720;

    if(filename){
        cap = cvCaptureFromFile(filename);
    }else{
        cap = cvCaptureFromCAM(cam_index);
    }

    if(w){
        cvSetCaptureProperty(cap, CV_CAP_PROP_FRAME_WIDTH, w);
    }
    if(h){
        cvSetCaptureProperty(cap, CV_CAP_PROP_FRAME_HEIGHT, h);
    }

    if(!cap) error("Couldn't connect to webcam.\n");
    cvNamedWindow(base, CV_WINDOW_NORMAL); 
    cvResizeWindow(base, 512, 512);
    float fps = 0;
    int i;
    float nms = .45;

    while(1){
        image in = get_image_from_stream(cap);
        //image in_s = resize_image(in, net->w, net->h);
        image in_s = letterbox_image(in, net->w, net->h);
        layer l = net->layers[net->n-1];

        float *X = in_s.data;
        network_predict(net, X);
        int nboxes = 0;
        detection *dets = get_network_boxes(net, in.w, in.h, thresh, 0, 0, 0, &nboxes);
        //if (nms) do_nms_obj(boxes, probs, l.w*l.h*l.n, l.classes, nms);
        if (nms) do_nms_sort(dets, nboxes, l.classes, nms);

        for(i = 0; i < nboxes; ++i){
            if(dets[i].prob[class] > thresh){
                box b = dets[i].bbox;
                int left  = b.x-b.w/2.;
                int top   = b.y-b.h/2.;
                censor_image(in, left, top, b.w, b.h);
            }
        }
        show_image(in, base);
        cvWaitKey(10);
        free_detections(dets, nboxes);


        free_image(in_s);
        free_image(in);


        float curr = 0;
        fps = .9*fps + .1*curr;
        for(i = 0; i < skip; ++i){
            image in = get_image_from_stream(cap);
            free_image(in);
        }
    }
    #endif
}

void extract_detector(char *datacfg, char *cfgfile, char *weightfile, int cam_index, const char *filename, int class, float thresh, int skip)
{
#ifdef OPENCV
    char *base = basecfg(cfgfile);
    network *net = load_network(cfgfile, weightfile, 0);
    set_batch_network(net, 1);

    srand(2222222);
    CvCapture * cap;

    int w = 1280;
    int h = 720;

    if(filename){
        cap = cvCaptureFromFile(filename);
    }else{
        cap = cvCaptureFromCAM(cam_index);
    }

    if(w){
        cvSetCaptureProperty(cap, CV_CAP_PROP_FRAME_WIDTH, w);
    }
    if(h){
        cvSetCaptureProperty(cap, CV_CAP_PROP_FRAME_HEIGHT, h);
    }

    if(!cap) error("Couldn't connect to webcam.\n");
    cvNamedWindow(base, CV_WINDOW_NORMAL); 
    cvResizeWindow(base, 512, 512);
    float fps = 0;
    int i;
    int count = 0;
    float nms = .45;

    while(1){
        image in = get_image_from_stream(cap);
        //image in_s = resize_image(in, net->w, net->h);
        image in_s = letterbox_image(in, net->w, net->h);
        layer l = net->layers[net->n-1];

        show_image(in, base);

        int nboxes = 0;
        float *X = in_s.data;
        network_predict(net, X);
        detection *dets = get_network_boxes(net, in.w, in.h, thresh, 0, 0, 1, &nboxes);
        //if (nms) do_nms_obj(boxes, probs, l.w*l.h*l.n, l.classes, nms);
        if (nms) do_nms_sort(dets, nboxes, l.classes, nms);

        for(i = 0; i < nboxes; ++i){
            if(dets[i].prob[class] > thresh){
                box b = dets[i].bbox;
                int size = b.w*in.w > b.h*in.h ? b.w*in.w : b.h*in.h;
                int dx  = b.x*in.w-size/2.;
                int dy  = b.y*in.h-size/2.;
                image bim = crop_image(in, dx, dy, size, size);
                char buff[2048];
                sprintf(buff, "results/extract/%07d", count);
                ++count;
                save_image(bim, buff);
                free_image(bim);
            }
        }
        free_detections(dets, nboxes);


        free_image(in_s);
        free_image(in);


        float curr = 0;
        fps = .9*fps + .1*curr;
        for(i = 0; i < skip; ++i){
            image in = get_image_from_stream(cap);
            free_image(in);
        }
    }
    #endif
}
*/

/*
void network_detect(network *net, image im, float thresh, float hier_thresh, float nms, detection *dets)
{
    network_predict_image(net, im);
    layer l = net->layers[net->n-1];
    int nboxes = num_boxes(net);
    fill_network_boxes(net, im.w, im.h, thresh, hier_thresh, 0, 0, dets);
    if (nms) do_nms_sort(dets, nboxes, l.classes, nms);
}
*/

//  ./darknet [xxx]中如果命令如果第二个xxx参数是detector，则调用这个
void run_detector(int argc, char **argv)
{
    char *prefix = find_char_arg(argc, argv, "-prefix", 0); //寻找是否有参数prefix, 默认参数0，argv为二维数组，存储了参数字符串
    float thresh = find_float_arg(argc, argv, "-thresh", .5);   //寻找是否有参数thresh，thresh为输出的阈值,默认参数0.24
    float hier_thresh = find_float_arg(argc, argv, "-hier", .5);    //寻找是否有参数hier_thresh,默认0.5
    int cam_index = find_int_arg(argc, argv, "-c", 0);  //寻找是否有参数c，默认0
    int frame_skip = find_int_arg(argc, argv, "-s", 0); //寻找是否有参数s，默认0
    int avg = find_int_arg(argc, argv, "-avg", 3);
    
    //如果输入参数小于4个，输出正确语法如何使用
    //printf 等价于 fprintf(stdout, ...)，这里stderr和stdout默认输出设备都是屏幕，但是stderr一般指标准出错输入设备 
    if(argc < 4){
        fprintf(stderr, "usage: %s %s [train/test/valid] [cfg] [weights (optional)]\n", argv[0], argv[1]);
        return;
    }
    char *gpu_list = find_char_arg(argc, argv, "-gpus", 0); //寻找是否有参数gpus，默认0
    char *outfile = find_char_arg(argc, argv, "-out", 0);   //检查是否指定GPU运算,默认0
    int *gpus = 0;
    int gpu = 0;
    int ngpus = 0;
    if(gpu_list){
        printf("%s\n", gpu_list);
        int len = strlen(gpu_list);
        ngpus = 1;
        int i;
        for(i = 0; i < len; ++i){
            if (gpu_list[i] == ',') ++ngpus;
        }
        gpus = calloc(ngpus, sizeof(int));
        for(i = 0; i < ngpus; ++i){
            gpus[i] = atoi(gpu_list);
            gpu_list = strchr(gpu_list, ',')+1;
        }
    } else {
        gpu = gpu_index;
        gpus = &gpu;
        ngpus = 1;
    }

    int clear = find_arg(argc, argv, "-clear"); //检查clear参数
    int fullscreen = find_arg(argc, argv, "-fullscreen");
    int width = find_int_arg(argc, argv, "-w", 0);
    int height = find_int_arg(argc, argv, "-h", 0);
    int fps = find_int_arg(argc, argv, "-fps", 0);
    //int class = find_int_arg(argc, argv, "-class", 0);

    char *datacfg = argv[3];   //存data文件路径
    char *cfg = argv[4];    //存cfg文件路径
    char *weights = (argc > 5) ? argv[5] : 0;   //存weight文件路径
    char *filename = (argc > 6) ? argv[6]: 0;   //存待检测文件路径
    
    //根据第三个参数的内容，调用不同的函数，并传入之前解析的参数
    if(0==strcmp(argv[2], "test")) test_detector(datacfg, cfg, weights, filename, thresh, hier_thresh, outfile, fullscreen);
    else if(0==strcmp(argv[2], "train")) train_detector(datacfg, cfg, weights, gpus, ngpus, clear);
    else if(0==strcmp(argv[2], "valid")) validate_detector(datacfg, cfg, weights, outfile);
    else if(0==strcmp(argv[2], "valid2")) validate_detector_flip(datacfg, cfg, weights, outfile);
    else if(0==strcmp(argv[2], "recall")) validate_detector_recall(datacfg, cfg, weights);
    else if(0==strcmp(argv[2], "demo")) {
        list *options = read_data_cfg(datacfg);
        int classes = option_find_int(options, "classes", 20);
        char *name_list = option_find_str(options, "names", "data/names.list");
        char **names = get_labels(name_list);
        demo(cfg, weights, thresh, cam_index, filename, names, classes, frame_skip, prefix, avg, hier_thresh, width, height, fps, fullscreen);
    }
    //else if(0==strcmp(argv[2], "extract")) extract_detector(datacfg, cfg, weights, cam_index, filename, class, thresh, frame_skip);
    //else if(0==strcmp(argv[2], "censor")) censor_detector(datacfg, cfg, weights, cam_index, filename, class, thresh, frame_skip);
}
```



# 参考

[YOLO-V3实战（darknet）](https://www.cnblogs.com/xieqi/p/9818056.html)

[Darknet 评估训练好的网络的性能](https://www.jianshu.com/p/7ae10c8f7d77)

[yolov3实战(darknet)](https://www.cnblogs.com/xieqi/p/9818056.html)

[yolo_person_detect](https://github.com/pascal1129/yolo_person_detect)

[Yolo-v3 and Yolo-v2 for Windows and Linux](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects)

[pjreddie/darknet](