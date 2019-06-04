---
title: OpenCV-选取图像的感兴趣区域
toc: true
categories:
  - OpenCV
tags:
  - opencv
abbrlink: d4c8e253
date: 2019-06-02 17:41:57
---

Mat 类提供了多种方便的方法来选择图像的局部区域。使用这些方法时需要注意,这些方法并不进行内存的复制操作。如果将局部区域赋值给新的 Mat 对象,新对象与原始对象共用相同的数据区域,不新申请内存,因此这些方法的执行速度都比较快。

<!-- more -->

## 单行或者单列选择

提取矩阵的一行或者一列可以使用函数 row()或 col()。函数的声明如下:

```c++
Mat Mat::row(int i) const
Mat Mat::col(int j) const
```

参数 i 和 j 分别是行标和列标。例如取出 A 矩阵的第 i 行可以使用如下代码:

```c++
Mat line = A.row(i);
```

例如取出 A 矩阵的第 i 行,将这一行的所有元素都乘以 2,然后赋值给第 j 行,可以这样写:

```c++
A.row(j) = A.row(i)*2;
```

## 用 Range 选择多行或多列

Range 是 OpenCV 中新增的类,该类有两个关键变量 star 和 end。Range 对象可以用来表示矩阵的多个连续的行或者多个连续的列。其表示的范围为从 start到 end,包含 start,但不包含 end。Range 类的定义如下:

```c++
class Range
{
    public:
    ...
    int start, end;
};
```

Range 类还提供了一个静态方法 all(),这个方法的作用如同 Matlab 中的“:”,表示所有的行或者所有的列。

```c++
//创建一个单位阵
Mat A = Mat::eye(10, 10, CV_32S);
//提取第 1 到 3 列(不包括 3)
Mat B = A(Range::all(), Range(1, 3));
//提取 B 的第 5 至 9 行(不包括 9)
//其实等价于 C = A(Range(5, 9), Range(1, 3))
Mat C = B(Range(5, 9), Range::all());
```

## 感兴趣区域

从图像中提取感兴趣区域(Region of interest)有两种方法,一种是使用构造函数,如下例所示:

```c++
//创建宽度为 320,高度为 240 的 3 通道图像
Mat img(Size(320,240),CV_8UC3);
//roi 是表示 img 中 Rect(10,10,100,100)区域的对象
Mat roi(img, Rect(10,10,100,100));
```

除了使用构造函数,还可以使用括号运算符,如下:

```c++
Mat roi2 = img(Rect(10,10,100,100));
```

当然也可以使用 Range 对象来定义感兴趣区域,如下:

```c++
//使用括号运算符
Mat roi3 = img(Range(10,100),Range(10,100));
//使用构造函数
Mat roi4(img, Range(10,100),Range(10,100));
```

## 取对角线元素

矩阵的对角线元素可以使用 Mat 类的 diag()函数获取,该函数的定义如下:

```c++
Mat Mat::diag(int d) const
```

参数 d=0 时,表示取主对角线;当参数 d>0 是,表示取主对角线下方的次对角线,如 d=1 时,表示取主对角线下方,且紧贴主多角线的元素;当参数 d<0 时,表示取主对角线上方的次对角线。

如同 row()和 col()函数, diag()函数也不进行内存复制操作,其复杂度也是 O(1)。