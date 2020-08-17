---
layout: post
title: Dlib与OpenCV图像互转
subtitle: false
tags: [Project]
---

<!-- ## Dlib与OpenCV图片互转 -->

由于疲劳检测项目需要用使用人脸68个关键点检测得到的数据，而OpenCV的检测方法准确度不高，故果断采用Dlib进行开发，但在进行的过程中，也碰到了不少问题，最主要的就是人脸特征点检测时异常卡顿。而刚开始使用Dlib时，在Debug模式下，由于相机（30FPS）实时采集到的数据几十秒才检测出一帧，比蜗牛还慢，最初的分析还以为是图像转换时遇到的问题，后来发现这不是主要问题，但仍要记录一下，故引出下文。

在做人脸关键点检测时候碰到Dlib的图片格式转换以及Dlib与OpenCV图片格式的互转，找了不少资料，现总结如下：

1：首先，dlib图片格式与OpenCV还是有一定区别的，dlib是以`dlib::array2d`的形式呈现，而OpenCV是以`cv::Mat`的形式呈现，关于OpenCV图像之间的转换，网上有很多资料，这里不再赘述，仅介绍一下dlib的图片格式转换以及dlib与OpenCV之间图片格式的互转。

2：dlib中读取图片：

```c++
dlib::array2d<dlib::rgb_pixel> img_rgb;
dlib::load_image(img_rgb, "test_image.jpg");
```

3：dlib RGB图片转换成Gray图片：

```c++
dlib::array2d<unsigned char> img_gray;
dlib::assign_image(img_gray, img_rgb);
```

4：dlib转换成OpenCV图片：

```c++
#include <dlib/opencv.h>
#include <opencv2/opencv.hpp>
cv::Mat img = dlib::toMat(img_gray);
```

5：OpenCV转dlib：

```c++
#include<dlib/opencv.h>
#include<opencv2/opencv.hpp>
cv::Mat img =cv::imread("test_image.jpg");
dlib::cv_image<rgb_pixel>=dlib_img(img);
```

6:OpenCV灰度图片转dlib灰度图片：

```c++
#include<dlib/opencv.h>
#include<opencv2/opencv.hpp>
cv::gray_img
cv::Mat rgb_img = cv::imread("test_image.jpg");
cv::cvtcolor(rgb_img,gray_img,cv::COLOR_BGR2GRAY);
dlib::cv_image(uchar)=dlib_gray_img(gray_img);
```

参考链接：

[Convert RGB Image to Grayscale Image in DLIB](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/38180410/convert-rgb-image-to-grayscale-image-in-dlib)

[http://dlib.net/imaging.html](https://link.zhihu.com/?target=http%3A//dlib.net/imaging.html)