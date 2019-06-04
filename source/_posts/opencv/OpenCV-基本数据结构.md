---
title: OpenCV-基本数据结构
toc: true
categories:
  - OpenCV
tags:
  - opencv
abbrlink: 463fc5c6
date: 2019-05-25 12:22:04
---

![OpenCV 基本数据结构](http://image.shuiyujie.com/2019-05-25-13-01-13.png)

本文介绍 OpenCV 的基本数据结构，做到心中有数就不会在阅读示例代码的时候发憷。

<!-- more -->

# Mat 类

Mat 是 OpenCV 中最重要的一种数据结构，OpenCV 将其定义为一个类，用于存储图像矩阵。

| 属性       | 释义                               |
| ---------- | ---------------------------------- |
| dims       | 矩阵的维度，如 3x4x5 的矩阵为 3 维 |
| data       | uchar 类型指针, 指向矩阵数据内存   |
| rows, cols | 矩阵的行数、列数                   |
| type       | 矩阵元素类型 + 通道数              |
| depth      | 像素位数(bist)                     |
| channels   | 通道数量                           |
| elemSize   | 矩阵中每一个元素的数据大小         |
| elemSize1  | 单通道的矩阵元素占用的数据大小     |

type 表示矩阵元素类型和通道数。矩阵元素类型一般都是 8 位无符号整数，即 CV_8U，彩色图像一般为 3 通道，灰度图像则为单通道。所以灰度图像的 type 可以表示为 CV_8UC1，3 通道的 RGB 图像的 type 可以表示为 CV_8UC3。

以 CV_8UC3 为例，depth = CV_8U，channels = 3。

elemSize = channels * depth / 8 ，type是CV_8UC3，elemSize = 3 * 8 / 8 = 3bytes。

elemSize1 = depth / 8，type是CV_8UC3，elemSize1 = 8 / 8 = 1bytes。

通过遍历图像可以体会 Mat 类的使用，可以参考 [OpenCV-图像的遍历](<https://shuiyujie.com/post/dfe68fea.html>)。

# Ponit 类 - 点的表示

Point 类表示二维坐标系下的点，即 (x, y) 形式的 2D 点。

```c++
// 方法一
Point point1 = Point(10, 8);

// 方法二
Point point2;
point2.x = 8;
point2.y = 10;
```

其中 `Point_<int>`, Point2i, Point 等价，`Point_<float>`, Point2f 等价。

# Scalar 类 - 颜色的表示

Scalar() 表示具有 4 个元素的数组，用于传递像素值，如 RGB 颜色值。RGB 颜色值只有三个参数，Scalar() 用不到第四个参数可以缺省。

```c++
Mat M(3, 2, CV_8UC3, Scalar(0,0,255));
```

其中 `Scalar(0,0,255)` 表示 BGR 的值分别为 (0, 0, 255) 是红色。

# Size 类 - 尺寸的表示

```c++
typedef Size_<int> Size2i;
typedef Size_<int64> Size2l;
typedef Size_<float> Size2f;
typedef Size_<double> Size2d;
typedef Size2i Size;
```

源代码中的定义上述代码所示。Size_ 是模板类，`Size_<int>` 表示其模板类型为 int。之后给 `Size_<int>` 别名为 Size2i，再给 Size2i 别名为 Size。由此可见，**`Size_<int>`, Size2i, Size 等价。**

`Size size = Size(8, 10);` 表示宽和长分别为 8 和 10。

# Rect 类 - 矩形的表示

- Rect 类的成员变量有 x, y, width, heigh
- Size() 返回值为 Size
- area() 返回矩形面积
- contains(Point) 判断点是否在矩形内
- inside(Rect) 判断矩形是否在该矩形内
- tl() 返回左上角点坐标
- br() 返回右下角点坐标

```c++
// 矩形交集
Rect rect = rect1 & rect2;
// 矩形并集
Rect rect = rect1 | rect2;
// 平移操作
Rect rectShift = rect + point;
// 缩放操作
Rect rectScale = rect + size;
```