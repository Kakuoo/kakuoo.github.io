---
layout: post
title: Dlib人脸landmark检测速度优化
subtitle: false
tags: [Project]
---

<!-- ## Dlib人脸landmark检测速度优化 -->

### 背景

Dlib是建立在OpenCV基础上的一个计算机视觉库，很多方面在一定程度上优于OpenCV的效果，比如人脸检测，人脸关键点提取，其检测准确率比OpenCV更高，鲁棒性也更好，但是随之牺牲的是计算时间以及硬件资源。最近在做疲劳检测的过程中需要用到人脸68个关键点检测，首先想到的是OpenCV和Dlib,由于在以前的项目上见识到两者效果的差距，果断采用Dlib进行开发，但在进行的过程中也是碰到了各种问题，最主要的是特征点检测时视频会卡顿的非常严重，经分析，最终确定了是dlib检测的时候占用太多计算量，代码体现在官方文档中的如下一段

```c++
dlib::frontal_face_detector face_bbox_detector = dlib::get_frontal_face_detector();
dlib::cv_image<dlib::rgb_pixel> dlib_img(frame);
std::vector<rectangle> face_bboxes = face_bbox_detector(dlib_img);
```

`dlib::frontal_face_detector`会造成dlib检测速度过慢，导致调用摄像头进行关键点检测时会卡顿的非常严重，因此，提高检测速度刻不容缓。

**软硬件环境：**

> 系统环境：windows10
>
> CPU：Intel(R) Core(TM) i7-8700K @3.70GHz, 6核
>
> 内存：32GB
>
> 显卡：Titan Xp
>
> IDE：Visual Studio 2017
>
> Dlib：19.20， OpenCV：3.4.1

### 速度优化解决方案：

#### 1.系统架构——指令集

由于第一点并没有解决问题，返回Cmake查看编译dlib库的相关日志并查阅官方文档，请注意下面几句话。

```text
Finally, note that the face detector is fastest when compiled with at least
    SSE2 instructions enabled.  So if you are using a PC with an Intel or AMD
    chip then you should enable at least SSE2 instructions.  If you are using
    cmake to compile this program you can enable them by using one of the
    following commands when you create the build project:
        cmake path_to_dlib_root/examples -DUSE_SSE2_INSTRUCTIONS=ON
        cmake path_to_dlib_root/examples -DUSE_SSE4_INSTRUCTIONS=ON
        cmake path_to_dlib_root/examples -DUSE_AVX_INSTRUCTIONS=ON
    This will set the appropriate compiler options for GCC, clang, Visual
    Studio, or the Intel compiler.  If you are using another compiler then you
    need to consult your compiler's manual to determine how to enable these
    instructions.  Note that AVX is the fastest but requires a CPU from at least
    2011.  SSE4 is the next fastest and is supported by most current machines.
```

看到这的时候，心里凉了一大截，都怪自己初次使用Cmake，对Cmake了解不多，简单的点几下按钮，选择下路径，就把dlib给装了，现在想想，做任何事都不能急啊，扎心。我第一次采用的源码编译安装Dlib，使用Dlib需要先安装OpenCV，关于这两个的安装可以自行谷歌，或者查阅我标签为`Dlib`下的相关博客。

我这里安装会使用Cmake安装，**建议源码安装前请自己阅读源码中的README，以及dlib源码文件夹`dlib-19.20\CMakeLists.txt`，以及dlib-19.20\examples\CMakeLists.txt**，可以了解dlib使用Cmake编译的注意事项。

Cmake默认的安装方式虽然能安装好Dlib，但是这样安装的后果就是我之所以写这篇博文的导火线。这样安装会导致Dlib进行关键点检测的时候速度异常的慢。

参考官方文档及如下网站（该网站可能需要 fan *qiang),

[Speeding up Dlib's Facial Landmark Detector](https://www.learnopencv.com/speeding-up-dlib-facial-landmark-detector/)

发现了问题的所在，总结如下：

**Dlib原Paper指出 landmark detector能达到1000FPS的速度，但是你不可能达到这样的速度，因为在检测特征点之前首先需要检测人脸，这会花费至少几十毫秒（请各位大佬指正这句话翻译是否正确：takes a few 10s of milliseconds），但是使用一些优化措施能轻松的达到30FPS。**

可以注意到：

```text
        cmake path_to_dlib_root/examples -DUSE_SSE2_INSTRUCTIONS=ON
        cmake path_to_dlib_root/examples -DUSE_SSE4_INSTRUCTIONS=ON
        cmake path_to_dlib_root/examples -DUSE_AVX_INSTRUCTIONS=ON
```

AVX模式是最快的一种编译方式，但是这可能只适合支持AVX的CPU电脑，查看电脑是否支持AVX，若支持请把该模式打开，查看是否支持，请在终端输入以下语句：

```text
cat /proc/cpuinfo | grep avx
```

找到flags部分，如果其中输出有AVX表明支持AVX模式，同时编译的时候建议开启SSE4。

如果使用的是Intel或AMD芯片，请至少启用SSE2指令。SSE4是第二快的，并且受大多数当前计算机支持。AVX是最快的，但至少需要是2011年后生产的CPU的才能支持。

**AVX是256位的字符指令集，SSE2，SSE4是128位的字符指令集，256位的字符集可以同时执行8个32位的，16个16位，32个8位的，如此计算，而128位只能同时执行4个32位的，8个16位，16个8位的，速度相对慢很多。**

卸载第一次装的Dlib后我勾选以下的语句编译Dlib:

```text
DLIB_USE_BLAS
USE_AVX_INSTRUCTIONS
USE_SSE2_INSTRUCTIONS
USE_SSE4_INSTRUCTIONS
```

#### 2.编译方式——Debug/Release

Debug模式下会附带大量调试信息，十分缓慢，一帧图像的检测耗时约半分钟，在Release模式下，可以达到20FPS，泪的教训，除了调试，一定要运行Release的版本。

**需要注意的几点：**

1：源码安装的时候请尽量使用--prefix，指定安装路径，这样你在卸载软件的时候只需删除路径下的安装包即可，移植的时候把安装包拷贝到另一台电脑即成功（前提是系统相同），这点是我在卸载第一次安装的Dlib的时候踩过的坑，参考网站上各种方法，比如在源码下使用make uninstall卸载也都无济于事，只能自己手动在原先默认安装的位置删除各种文件，还存在删错了的可能，所以强烈建议安装的时候**指明安装路径**，同时在安装前最好看一下readme文件。

2：安装DLib时可能提示你安装 BLAS library，按照提示安装即可。可以直接在Cmake的configure界面中勾选`DLIB_USE_BLAS`

```text
*** No BLAS library found so using dlib's built in BLAS. However, if you ***
*** install an optimized BLAS such as OpenBLAS or the Intel MKL your code ***
*** will run faster. On Ubuntu you can install OpenBLAS by executing: ***
*** sudo apt-get install libopenblas-dev liblapack-dev ***
*** Or you can easily install OpenBLAS from source by downloading the ***
*** source tar file from http://www.openblas.net, extracting it, and ***
*** running: ***
*** make; sudo make install ***
```

3：使用命名空间的时候，using namespace dlib与using namespace cv这两个最好不要同时出现在一起，可能造成某些函数解析错误。

#### 3.图像格式转换

首先自然的想到将RGB图片转换成灰度图片，减少计算量，dlib 官方提供的example采用的是Dlib格式的图片，如果你采用OpenCV格式的图片作为参数传递会提示参数不匹配的错误，代码传送门

[dlib C++ Library - webcam_face_pose_ex.cppdlib.net](http://dlib.net/webcam_face_pose_ex.cpp.html)

关于Dlib的图片格式转换以及Dlib与OpenCV图片格式之间的互相转换，请参考博文[Dlib与OpenCV图片格式转换](https://zhuanlan.zhihu.com/p/36489663)。转换成功后再次运行，发现并没有什么卵用，速度依旧慢的跟蜗牛一般，然后寻思另一条解决道路，这便有了第二点。

#### 4.调整视频帧的分辨率

面部地标检测器算法通常要求用户提供包含面部的边界框。该算法将此框作为输入并返回界标。这些算法报告的时间仅是进行地标检测所需的时间，而不是面部检测所需的时间。具有里程碑意义的检测算法可以在不到5毫秒的时间内运行，但是人脸检测可能需要很长时间（30毫秒）。人脸检测的速度取决于图像的分辨率，因为使用较小分辨率的图像，您会寻找较小范围的人脸。缺点是您可能会错过较小的面孔，但是在疲劳检测中，我们都需要让计算机能清晰的分辨出人眼周围的关键点，所以图像分辨率不能设置的太低。

加快人脸检测的一种简单方法是调整视频帧的大小。将摄像头以720p（即1280×720）分辨率录制视频，可以将图像的大小调整为四分之一以进行人脸检测。应通过将坐标除以用于调整原始帧大小的比例来调整获得的边界框的大小。这使我们能够以全分辨率进行面部地标检测，例如当分辨率从720p更改为360p时，需要显示的实际像素数减少了4倍。

#### 5.跳帧检测

一般的摄像头通常以30FPS录制视频，在典型的应用场景中，人正坐在摄像头前并且移动不大。因此，无需在每一帧中都检测到脸部。我们可以简单地基于之前获得的几帧面部边界框进行面部地标检测。如果每3帧进行一次人脸检测，则可以将地标检测速度提高近三倍。

是否可以比使用框架的先前位置做得更好？是的，我们可以使用卡尔曼滤波来预测未进行检测的帧中人脸的位置，但是在网络摄像头应用程序中，这是一个过大的选择。

#### 6.自定义脸部渲染器

Dlib的面部渲染对我来说效果不佳；框架渲染不流畅。因此，我使用OpenCV的折线编写了自己的代码。代码如下所示

```c++
#ifndef BIGVISION_RENDER_FACE_H_
#define BIGVISION_RENDER_FACE_H_
#include <dlib/image_processing/frontal_face_detector.h>
#include <opencv2/opencv.hpp>

void draw_polyline(cv::Mat &img, const dlib::full_object_detection& d, const int start, const int end, bool isClosed = false)
{
	std::vector <cv::Point> points;
	for (int i = start; i <= end; ++i)
	{
		points.push_back(cv::Point(d.part(i).x(), d.part(i).y()));
	}
	cv::polylines(img, points, isClosed, cv::Scalar(255, 0, 0), 2, 16);
}

void render_face(cv::Mat &img, const dlib::full_object_detection& d)
{
	DLIB_CASSERT
	(
		d.num_parts() == 68,
		"\n\t Invalid inputs were given to this function. "
		<< "\n\t d.num_parts():  " << d.num_parts()
	);

	draw_polyline(img, d, 0, 16);           // Jaw line
	draw_polyline(img, d, 17, 21);          // Left eyebrow
	draw_polyline(img, d, 22, 26);          // Right eyebrow
	draw_polyline(img, d, 27, 30);          // Nose bridge
	draw_polyline(img, d, 30, 35, true);    // Lower nose
	draw_polyline(img, d, 36, 41, true);    // Left eye
	draw_polyline(img, d, 42, 47, true);    // Right Eye
	draw_polyline(img, d, 48, 59, true);    // Outer lip
	draw_polyline(img, d, 60, 67, true);    // Inner lip
}

#endif // BIGVISION_RENDER_FACE_H_
```



### 后记：

此次使用Dlib学习到了很多经验，包括安装的一些注意事项，以及Dlib与OpenCV图片格式转换问题。按照以上思路整理后，在此运行代码，你会发现速度会变得非常和谐优雅，不会出现卡顿的现象，nice！



## 参考链接：

[dlib C++ Library - webcam_face_pose_ex.cpp](https://link.zhihu.com/?target=http%3A//dlib.net/webcam_face_pose_ex.cpp.html)

[Speeding up Dlib's Facial Landmark Detector](https://link.zhihu.com/?target=https%3A//www.learnopencv.com/speeding-up-dlib-facial-landmark-detector/)

[What is cmake equivalent of 'configure --prefix=DIR && make all install '?](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/6003374/what-is-cmake-equivalent-of-configure-prefix-dir-make-all-install)

[Ubuntu 16.04开启dlib对于AVX或者CUDA的支持](https://link.zhihu.com/?target=http%3A//www.mobibrw.com/2017/7153)

[Ubuntu下dlib库编译安装 - actonton - 博客园](https://link.zhihu.com/?target=http%3A//www.cnblogs.com/whenyd/p/7721989.html)

[Compiling dlib on OS X](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/38156075/compiling-dlib-on-os-x)

[CLion 2016.3.2 EAP: CMake configurations, project templates and GCC6](https://link.zhihu.com/?target=https%3A//blog.jetbrains.com/clion/2016/12/clion-2016-3-2-eap/)