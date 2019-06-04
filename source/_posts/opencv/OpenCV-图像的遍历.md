---
title: OpenCV-图像的遍历
toc: true
categories:
  - OpenCV
tags:
  - opencv
abbrlink: dfe68fea
date: 2019-05-25 09:19:43
---

计算机使用 0/1 编码存储图像，数字图像在计算机中同样也使用 0/1 编码来存储。在计算机看来图像是一堆亮度不同的点组成的矩阵。一般灰度图用 2 维矩阵来表示，彩色图片是多通道的，则用 3 维矩阵来表示。

![灰度图像](http://image.shuiyujie.com/2019-05-25-09-31-19.png)

我们一般接触的图像都是 8 位整数（CV_8U），所以灰度图像包含 0～255 灰度，其中 0 代表最⿊，1表⽰最⽩。

![彩色图像](http://image.shuiyujie.com/2019-05-25-09-32-13.png)

彩色图像比如 RGB 图像，每个像素用三个字节来表示，而 OpenCV 中存储 RGB 图像以 BGR 的顺序存储图像，所以存储方式如上所示。

本文将介绍遍历灰度图像和彩色图像的方法，其本质即为遍历图像矩阵，可以对比二维数组的遍历来学习。

<!-- more -->

# 使用 at() 遍历图像

```c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

int main()
{
    // 使用构造器创建 Mat，注意类型
    cv::Mat grayImage(400, 500, CV_8UC1);
    cv::Mat colorImage(400, 500, CV_8UC3);

    for(int i = 0; i < grayImage.rows; i++)
    {
        for(int j = 0; j < grayImage.cols; j++)
        {
            // 灰度图像是单通道的
            grayImage.at<uchar>(i, j) = (i + j) % 255;
        }
    }

    for(int i = 0; i < colorImage.rows; i++)
    {
        for(int j = 0; j < colorImage.cols; j++)
        {
            // 三通道用向量来表示
            cv::Vec3b pixel;
            pixel[0] = i % 255;
            pixel[1] = j % 255;
            pixel[2] = 0;
            colorImage.at<cv::Vec3b>(i, j) = pixel;
        }
    }

    cv::imshow("1. GrayImage", grayImage);
    cv::imshow("2. ColorImage", colorImage);

    cv::waitKey(0);

    return 0;
}
```

![效果图](http://image.shuiyujie.com/2019-05-25-10-47-43.png)

at() 的优点在于可读性强，缺点在于效率不高。图像遍历操作时常用且开销很大的操作，所以不推荐使用 at() 做图像的遍历操作。

# 使用指针遍历

```C++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

int main()
{
    cv::Mat grayImage(400, 500, CV_8UC1);
    cv::Mat colorImage(400, 500, CV_8UC3);

    for(int i = 0; i < grayImage.rows; i++)
    {
        uchar *p = grayImage.ptr<uchar>(i);
        
        for(int j =0; j < grayImage.cols; j++)
        {
            p[j] = (i + j) % 255;
        }
    }

    for(int i = 0; i < colorImage.rows; i++)
    {
        
        cv::Vec3b *p = colorImage.ptr<cv::Vec3b>(i);
    
        for(int j =0; j < colorImage.cols; j++)
        {
            p[j][0] = i % 255;
            p[j][1] = j % 255;
            p[j][2] = 0;
        }
    }

    cv::imshow("1. GrayImage", grayImage);
    cv::imshow("2. ColorImage", colorImage);

    cv::waitKey(0);

    return 0;
}
```

![效果图](http://image.shuiyujie.com/2019-05-25-10-47-43.png)

程序是用 `image.ptr()` 返回矩阵每一行的头指针，紧接着遍历这一行的元素。

指针的优点在于访问速度快，缺点在于指针较为复杂，不熟悉指针的朋友容易犯错且不易查找错误。

# 迭代器遍历

```c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

int main()
{
    cv::Mat grayImage(400, 500, CV_8UC1);
    cv::Mat colorImage(400, 500, CV_8UC3);

    cv::MatIterator_<uchar> grayit, grayend;

    for (grayit = grayImage.begin<uchar>(), grayend = grayImage.end<uchar>();
         grayit != grayend; grayit++)
    {
        *grayit = rand() % 255;
    }

    cv::MatIterator_<cv::Vec3b> colorit, colorend;
    for (colorit = colorImage.begin<cv::Vec3b>(), colorend = colorImage.end<cv::Vec3b>();
         colorit != colorend; colorit++)
    {
        (*colorit)[0] = rand() % 255;
        (*colorit)[1] = rand() % 255;
        (*colorit)[2] = rand() % 255;
    }

    cv::imshow("1. GrayImage", grayImage);
    cv::imshow("2. ColorImage", colorImage);

    cv::waitKey(0);

    return 0;
}
```

![迭代器遍历](http://image.shuiyujie.com/2019-05-25-11-13-05.png)

在 C++ 的 STL 库，或者 Java, Python  等语言中都提供了对迭代器的支持，OpenCV 也支持使用迭代器的方式进行遍历，代码如上所示。

------

本文介绍了遍历 Mat 的三种方式: 使用 at() 遍历，使用指针遍历，以及使用迭代器遍历。推荐使用指针的方式遍历图像，其效率最高，在编写图像处理的算法时，效率是我们不可忽视的要素。