---
layout: post
title: Python Pillow 和 cv2 图片 resize 速度的比较
subtitle: cv2 比 PIL 快不少，至少在图像 resize 上
tags: [Python]
---

<!-- # Python Pillow 和 cv2 图片 resize 速度的比较 -->

今天要说的事情很简单，就是比较了一下 PIL 和 cv2 resize 图片的速度。我们都知道，Python 中有关图像处理的库有很多，常见的有 cv2，scikit-image，PIL （严谨点应该叫 Pillow，下文就用 PIL 来代替了） 等等。在用 Python 进行深度学习图像任务的时候，我们常常会使用 PIL 这个库来读取图片（尤其是在用 PyTorch 的时候）。至于为什么 PIL 比较常用，我也不知道… 难道是 TorchVision 带来的风气（https://github.com/pytorch/vision#image-backend）？ 但在进行视频流处理的时候，我们往往会用到 cv2，因为都会用到 cv2.VideoCapture() 来读视频（应该没有人第一反应是其他的库吧）。

为什么会想到对比这二者 resize 图片的速度？原因是最近处理视频流的时候用的是 cv2 读取，每一帧读出来的结果是一个3维的 Numpy Array。然后要 resize 一下送到模型嘛，因为惯性我就用了 PIL 来做图片 resize （而没有用 cv2.resize）。PIL 的 resize 只能对 PIL Image 类做处理，所以我先把 Numpy Array 转成 PIL Image， 然后 resize， 然后再转回 Numpy Array。 我后来再看代码的时候心想这 tm 是什么操作？那索性来比一比这二者的速度吧。

因为这不是什么严肃的对比，所以我就不列啥硬件软件配置了。但大体上就是，一台普通的电脑，用着 pip3 安装来的普通的 cv2 和 PIL，做的一次简单的对比。

resize对比中我们使用的是 CV 界中的经典图像，512x512 的豪华彩色三通道 Lena 的 png 图片

![Lena](https://img-blog.csdnimg.cn/20191110124017979.png?type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J5cm9uMTIzNDU2c2ZzZnNmYQ==,size_16,color_FFFFFF,t_70#pic_center)

首先我们先测试一下 cv2 的速度，我们采用双线性插值，将 512x512 的图片 resize 到 1024x1024：

```python
repeat = 2000
im = cv2.imread('lena512_colour.png')
print(type(im), im.shape)
# <class 'numpy.ndarray'> (512, 512, 3)

start = time.time()
for i in range(repeat):
    im_resized = cv2.resize(im, (1024, 1024), interpolation=cv2.INTER_LINEAR)

print('cv2 resize - total time of %d is %.3f s' % (repeat, time.time()-start))
# cv2 resize - total time of 2000 is 3.789 s
```

然后试试 PIL 吧，所以条件一致的情况下，我们假定需要的输入和输出都是 `Numpy Array`：

```python
repeat = 2000
im = cv2.imread('lena512_colour.png')

start = time.time()
for i in range(repeat):
    tmp = Image.fromarray(im)
    tmp = tmp.resize((1024, 1024), resample=Image.BILINEAR)
    im_resized = np.array(tmp)

print('PIL resize - total time of %d is %.3f s' % (repeat, time.time()-start))
# PIL resize - total time of 2000 is 40.714 s
```

这… 被吊打好嘛。当然这对 PIL 有些不公平，毕竟 `Numpy Array` 和 `PIL Image` 互相转也要花费时间，所以我们来测一下输入输出都是 `PIL Image` 时候的速度：

```python
repeat = 2000

im = Image.open('lena512_colour.png')
print(type(im))
# <class 'PIL.PngImagePlugin.PngImageFile'>

start = time.time()
for i in range(repeat):
    im_resized = im.resize((1024, 1024), resample=Image.BILINEAR)

print('PIL resize - total time of %d is %.3f s' % (repeat, time.time()-start))
# PIL resize - total time of 2000 is 27.219 s
```

好嘛，还是被吊打… 我查了查资料，Kaggle 上有位老哥做了比较全的对比，比我严谨多了，结果也是 PIL 被吊打（https://www.kaggle.com/vfdev5/pil-vs-opencv）。还有老哥建议用优化过的 Pillow-SIMD，但是貌似官方的测试结果（https://python-pillow.org/pillow-perf/）还是差 OpenCV 好多啊… well…


## 买一送一：cv2 的 BGR

我们都知道，用 cv2 打开彩色三通道图像的时候，通道的顺序是 BGR，所以比如我们用 pyplot 来显示图片的时候，图片是不正常的，效果如下：（青紫色图像）

![冷色调的 Lena](https://img-blog.csdnimg.cn/20191110124748532.png?type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J5cm9uMTIzNDU2c2ZzZnNmYQ==,size_16,color_FFFFFF,t_70#pic_center)

所以我们在预处理的时候，还要把通道给变成 RGB。怎么变呢，我知道有三种方法：

```python
# 方法一：
repeat = 50000
im = cv2.imread('lena512_colour.png')

start = time.time()
for i in range(repeat):
    b, g, r = cv2.split(im)
    im_rgb1 = cv2.merge([r, g, b])

print('method 1 - total time of %d is %.3f s' % (repeat, time.time()-start))
# method 1 - total time of 50000 is 9.279 s
```

```python
# 方法二：
repeat = 50000
im = cv2.imread('lena512_colour.png')

start = time.time()
for i in range(repeat):
    im_rgb2 = cv2.cvtColor(im, cv2.COLOR_BGR2RGB)

print('method 2 - total time of %d is %.3f s' % (repeat, time.time()-start))
# method 2 - total time of 50000 is 1.602 s
```

```python
# 方法三：
repeat = 50000
im = cv2.imread('lena512_colour.png')

start = time.time()
for i in range(repeat):
    im_rgb3 = im[: , : , ::-1]

print('method 3 - total time of %d is %.3f s' % (repeat, time.time()-start))
# method 3 - total time of 50000 is 0.027 s
```

三种方法的结果都是一样的。但是大家需要主要的是，虽然第三种速度最快，但 im_rgb3 和 im 是共享内存的哦，也就是如果后边 im 的值被（inplace）修改了，im_rgb3 的值也会跟着相应变化，而前两种方法是不会有这种情况的。

总结
经过不严谨的对比显示，cv2 比 PIL 快不少，至少在图像 resize 上。好了，不多说了，周一的时候把改正的代码测试一下，估计模型的实时处理速度会提升一点，没准老板会夸我优化做得不错呢（玩笑，测试用的 demo 而已），CV 从业者的一天，往往就是这么朴实无华且枯燥。

---

参考博客：[AlanBupt](https://blog.csdn.net/byron123456sfsfsfa/article/details/102996399)