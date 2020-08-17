---
layout: post
title: TensorFlow的两种数据格式NHWC和NCHW
subtitle: false
tags: [TensorFlow]
---

<!-- ## TensorFlow的两种数据格式NHWC和NCHW -->

图像数据格式定义了一批图片数据的存储顺序。在调用 TensorFlow API 时会经常看到 `data_format` 参数：

`data_format` 默认值为 "`NHWC`"，也可以手动设置为 "`NCHW`"。这个参数规定了 `input Tensor` 和 `output Tensor` 的排列方式。

data_format 设置为 "`NHWC`" 时，排列顺序为 `[batch, height, width, channels]`；

设置为 "`NCHW`" 时，排列顺序为 `[batch, channels, height, width]`。

其中 N 表示这批图像有几张，H 表示图像在竖直方向有多少像素，W 表示水平方向像素数，C 表示通道数（例如黑白图像的通道数 C = 1，而 RGB 彩色图像的通道数 C = 3）。为了便于演示，我们后面作图均使用 RGB 三通道图像。

两种格式的区别如下图所示：
![NHCW_NHWC_cmp]({{site.url}}/assets/blog_images/TensorFlow_NHWC_NCHW/NHCW_NHWC_cmp.png)

`NCHW` 中，C 排列在外层，每个通道内像素紧挨在一起，即 '`RRRRRRGGGGGGBBBBBB`' 这种形式。

`NHWC` 格式，C 排列在最内层，多个通道对应空间位置的像素紧挨在一起，即 '`RGBRGBRGBRGBRGBRGB`' 这种形式。

如果我们需要对图像做彩色转灰度计算，NCHW 计算过程如下：

![NCHW]({{site.url}}/assets/blog_images/TensorFlow_NHWC_NCHW/NCHW.png)

即 R 通道所有像素值乘以 0.299，G 通道所有像素值乘以 0.587，B 通道所有像素值乘以 0.114，最后将三个通道结果相加得到灰度值。

相应地，NHWC 数据格式的彩色转灰度计算过程如下：

![NHWC]({{site.url}}/assets/blog_images/TensorFlow_NHWC_NCHW/NHWC.png)


输入数据分成多个(R, G, B) 像素组，每个像素组中 R 通道像素值乘以 0.299，G 通道像素值乘以 0.587，B 通道像素值乘以 0.114 后相加得到一个灰度输出像素。将多组结果拼接起来得到所有灰度输出像素。

以上使用两种数据格式进行 RGB -> 灰度计算的复杂度是相同的，区别在于访存特性。通过两张图对比可以发现，**NHWC 的访存局部性更好**（每三个输入像素即可得到一个输出像素），**NCHW** 则必须等所有通道输入准备好才能得到最终输出结果，**需要占用较大的临时空间**。

在 CNN 中常常见到 1x1 卷积（例如：[用于移动和嵌入式视觉应用的 MobileNets](http://mp.weixin.qq.com/s?__biz=MzI2MzYwNzUyNg==&mid=2247483973&idx=1&sn=b0b9aa4190f5ac9a34421beaa92eb932&chksm=eab807ccddcf8edaa798098c73b82ee35f4b22e159ddcd4ffb0d0cd6cae77a170a59a5c441e4&scene=21#wechat_redirect)），也是每个输入 channel 乘一个权值，然后将所有 channel 结果累加得到一个输出 channel。如果使用 NHWC 数据格式，可以将卷积计算简化为矩阵乘计算，即 **1x1 卷积核实现了每个输入像素组到每个输出像素组的线性变换**。

TensorFlow 为什么选择 NHWC 格式作为默认格式？因为早期开发都是基于 CPU，使用 NHWC 比 NCHW 稍快一些（不难理解，NHWC 局部性更好，cache 利用率高）。

NCHW 则是 Nvidia cuDNN 默认格式，使用 GPU 加速时用 NCHW 格式速度会更快（也有个别情况例外）。

**最佳实践**：设计网络时充分考虑两种格式，最好能灵活切换，在 GPU 上训练时使用 NCHW 格式，在 CPU 上做预测时使用 NHWC 格式。

### 两种数据格式的相互转换

```python
# NHWC –> NCHW：
import tensorflow as tf
x = tf.reshape(tf.range(24), [1, 3, 4, 2])
out = tf.transpose(x, [0, 3, 1, 2])

# NCHW –> NHWC：
import tensorflow as tf
x = tf.reshape(tf.range(24), [1, 2, 3, 4])
out = tf.transpose(x, [0, 2, 3, 1])
```

参考文章：[慢慢学TensorFlow](<https://mp.weixin.qq.com/s/I4Q1Bv7yecqYXUra49o7tw>)





