---
title: Ubuntu16.04配置OpenCV环境
toc: true
categories:
  - OpenCV
tags:
  - opencv
  - ubuntu
  - linux
abbrlink: '5359e313'
date: 2019-03-19 19:42:00
---


本文介绍在 Ubuntu16.04 中配置 OpenCV 的环境 

Linux 环境: Ubuntu16.04 LST 

软件版本:　opencv-3.4.0 、opencv_contrib-3.4.0 (安装包在文末下载) 

参考文档：[Installation in Linux](https://docs.opencv.org/3.4.0/d7/d9f/tutorial_linux_install.html) 

<!-- more -->

# 依赖安装 

执行以下命令

```
[compiler] sudo apt-get install build-essential

[required] sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev cmake-gui

[optional] sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

下载以下两个文件，注意版本

[opencv-3.4.0](https://github.com/opencv/opencv/releases) 、[opencv_contrib-3.4.0](https://github.com/opencv/opencv_contrib/releases)

解压文件，并新建一个build文件夹作为编译目录，如下 

```
opencv/
├── build
├── opencv-3.4.0
└── opencv_contrib-3.4.0
```



# 使用CMake-gui进行编译

使用cmake-gui生成Makefile,并进行编译

## 生成 Makefile

![生成 Makefile](http://image.shuiyujie.com/Snipaste_2019-03-19_19-26-50.png)

1. 点击 Browse_Source... 引入 opencv-3.4.0 文件夹 
2. 点击 Browse_Build... 引入 build 文件夹 
3. 点击 Configure 
4. 点击 Generate 
5. 弹出如下对话框，选择使用Makefile生成的系统平台 

```
注：
1. 第一次会弹出`CMakeSetup`选择`default`那个就可以了
2. 开始的时候没有上图红色部分
```

之后我们勾选下面这几个选项

1. 编译成静态库，取消选择 BUILD_SHARED_LIBS 
2. 勾选　OPENCV_ENABLE_NOFREE 
3. 选择　OPENCV_EXTRA_MODULES_PATH 引入 opencv_contrib-3.4.0 下的 modules 文件夹 
4. 点击 Configure 
5. 点击 Generate 

## 编译

进入 build 文件夹路径

> make -j7 # 7 表示7个并发

修改　opencv.pv 文件

> cd /build/unix_install
>
> vim opencv.pc

```
# Package Information for pkg-config
prefix=/usr/local
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir_old=${prefix}/include/opencv
includedir_new=${prefix}/include

Name: OpenCV
Description: Open Source Computer Vision Library
Version: 3.4.0
Libs: -L${exec_prefix}/lib -lopencv_stitching -lopencv_superres -lopencv_videostab -lopencv_aruco -lopencv_bgsegm -lopencv_bioinspired -lopencv_ccalib -lopencv_dpm -lopencv_face -lopencv_photo -lopencv_freetype -lopencv_fuzzy -lopencv_img_hash -lopencv_line_descriptor -lopencv_optflow -lopencv_reg -lopencv_rgbd -lopencv_saliency -lopencv_stereo -lopencv_structured_light -lopencv_phase_unwrapping -lopencv_surface_matching -lopencv_tracking -lopencv_datasets -lopencv_text -lopencv_dnn -lopencv_plot -lopencv_xfeatures2d -lopencv_shape -lopencv_video -lopencv_ml -lopencv_ximgproc -lopencv_calib3d -lopencv_features2d -lopencv_highgui -lopencv_videoio -lopencv_flann -lopencv_xobjdetect -lopencv_imgcodecs -lopencv_objdetect -lopencv_xphoto -lopencv_imgproc -lopencv_core
Libs.private: -ldl -lm -lpthread -lrt
Cflags: -I${includedir_old} -I${includedir_new}
```

将文件中的`Libs.private:`删除

> sudo make install
>
> sudo updatedb
>
> sudo ldconfig

# 运行

写一段测试程序，可以使用命令行的方式或者 CMake 进行编译

## 测试程序

```c++
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/core/core.hpp"
#include "opencv2/video/video.hpp"
#include <iostream>

using namespace cv;

int main(int argc, char **argv)
{
    // 从命令行参数读取图片路径
    char *imageName = argv[1];

    Mat image;
    image = imread(imageName, IMREAD_COLOR);

    if (argc != 2 || !image.data)
    {
        std::cout << "No image data" << std::endl;
        return -1;
    }

    Mat gray_image;
    cvtColor(image, gray_image, COLOR_BGR2GRAY);

    // 保存转换之后的灰度图片
    imwrite("Gray_Image.jpg", gray_image);

    namedWindow(imageName, WINDOW_AUTOSIZE);
    namedWindow("Gray image", WINDOW_AUTOSIZE);

    imshow(imageName, image);
    imshow("Gray image", gray_image);
    waitKey(0);

    return 0;
}
```

## 命令行方式编译

> g++ ModifyImage.cpp `pkg-config --cflags --libs opencv` -o ModifyImage

## CMake 方式编译

新建一个`CMakeLists.txt`

```
cmake_minimum_required(VERSION 2.8)
project( ModifyImage )
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_executable( ModifyImage ModifyImage.cpp )
target_link_libraries( ModifyImage ${OpenCV_LIBS} )
```

输入以下命令

> cmake .
>
> make

通过以上两种方式都会生成一个可执行文件，我们可以执行它

> ./ModifyImage

