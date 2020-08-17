---
layout: post
title: PyTorch执行上采样操作的四种方法
subtitle: false
tags: [PyTorch]
---

<!-- ## PyTorch上采样操作的几种简单方法 -->

### 一、反池化操作

反池化操作有两种，一种是反最大池化，另一种是反平均池化。反池化是池化的逆操作，是无法通过池化的结果还原出全部的原始数据。因为池化的过程就只保留了主要信息，舍去部分信息。

#### 反最大池化

![图源网络](https://img-blog.csdnimg.cn/20200610091709602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY0MzU0Nw==,size_16,color_FFFFFF,t_70)

主要原理就是在Maxpooling的时候保留最大值的位置信息，之后在unPooling阶段使用该信息扩充Feature Map，除最大值位置以外，其余补0。如上图，（图源网络）以下是用在自编码结构中的一个上采样类，仅做参考。

```python
class UpSamplingBlock(nn.Module):
    def __init__(self, in_channels, out_channels):
        super(UpSamplingBlock, self).__init__()
        self.unpool = nn.MaxUnpool2d(2, stride=2)  # kernel =2
        self.pool = nn.MaxPool2d(2, stride=2, return_indices=True)
        # return_indices - 如果等于True，会返回输出最大值的序号，对于上采样操作会有帮助,经过kernel=2，stride=2后，尺度缩减为原来的1/4
        self.conv1 = conv5x5(in_channels, out_channels)
        self.bn1 = nn.BatchNorm2d(out_channels)  # out_channels为输入通道数
        self.relu = nn.ReLU(inplace=True)
        self.conv2 = conv3x3(out_channels, out_channels)
        self.bn2 = nn.BatchNorm2d(out_channels)

    def forward(self, x):
        # create indices for unpool
        size = x.size()
        _, indices = self.pool(torch.empty(size[0], size[1], size[2] * 2, size[3] * 2))
        # torch.empty创建一个未被初始化数值的tensor,tensor的大小是由size确定，随机找出最大值安插的位置
        '''m=nn.MaxPool2d((3,3),stride=(1,1),return_indices=True)
           upm=nn.MaxUnpool2d((3,3),stride=(1,1))
           data4=torch.randn(1,1,3,3)
           output5,indices=m(data4)，  indices记录下下采样操作时，其中最大值的位置，即索引
           output6=upm(output5,indices)'''
        # unpool and assign residual
        out = self.unpool(x, indices.to(device))
        residual = self.conv1(out)  # 5X5卷积的残差网络的分支道路
        residual = self.bn1(residual)
        # forward and projection，上采样网络的主道路
        out = self.conv1(out)  # 5X5卷积,经过paddig
        out = self.bn1(out)  # 应该对输出结果out归一化
        out = self.relu(out)
        out = self.conv2(out)  # 3X3卷积，经过padding
        out = self.bn2(out)
        out += residual
        return out
```

#### 反平均池化

反平均池化的操作其实与反最大值池化操作类似，如反最大值池化上图只不过是除了索引位置，其他对应位置也用均值来代替。如下图（图源网络 ）![图源网络](https://img-blog.csdnimg.cn/202006100928151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY0MzU0Nw==,size_16,color_FFFFFF,t_70)

### 二、ConvTranspose2d

PyTorch中的逆置卷积函数：

`nn.ConvTranspose2d(in_channels=, out_channels=, kernel_size=, stride=1, padding=0, output_padding=0, groups=1, bias=True, dilation=1)`
函数的使用按照自己需要的得到的通道数和尺寸来设计参数，具体公式如下：
$$output = (input - 1) * stride + outputpadding - 2 * padding + kernelsize$$

```python
x = torch.ones((1, 3, 3, 3))
ct = nn.ConvTranspose2d(in_channels=3, out_channels=1, kernel_size=3, stride=2, padding=1, output_padding=0)
print(ct(x).shape)		# 最后的输出为torch.Size([1, 1, 5, 5])
```

具体的原理比较复杂，可以参考这篇博客，讲的比较详细：[抽丝剥茧，带你理解转置卷积（反卷积）](https://blog.csdn.net/tsyccnh/article/details/87357447)

### 三、PixelShuffle（直译：像素重组）

pixelshuffle的原理：
pixelshuffle的主要原理就是将r\*r个通道的特征图转换成新的w∗r,h∗r 的上采样结果（比如原来特征图大小为4\*128\*128，现在就调整成大小1\*256\*256）。具体来说，就是按照一定规则将每个像素点的r\*r个通道依次转换为对应的r\*r的图像块。最开始是用于超分辨率领域，原理如下图（图源网络）
在pytorch中有此函数，可以直接拿来使用，在TensorFlow1.x中好像没有这个函数，得自己写代码

```python
# 上采样操作之亚像素卷积，
ps = nn.PixelShuffle(3)	#3为放大倍数，即h与w方向上均扩大3倍
input = autograd.Variable(torch.Tensor(1, 9, 4, 4))
output = ps(input)
print(output.size())
torch.Size([1, 1, 12, 12])
```

### 四、Upsample方法

#### 1.最近邻插值UpsamplingNearest2d函数(Nearest-neighbor)

`torch.nn.UpsamplingNearest2d(size=None, scale_factor=None)`对输入信号做2D最近邻上采样。原理如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200610173947441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY0MzU0Nw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200610174305548.png)

```python
input = torch.arange(1, 5).view(1, 1, 2, 2)
up_near = nn.UpsamplingNearest2d(scale_factor=2)
out = up_near(input)
```

#### 2.双线性插值UpsamplingBilinear2d函数(Bilinear)

`torch.nn.UpsamplingBilinear2d(size=None, scale_factor=None)`对输入信号做2D双线性上采样。size ：一个包含两个整数的元组 (H_out, W_out)指定了输出的长宽，scale_factor：长和宽的一个乘子。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190720150732623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1X2Z1X3lvbmc=,size_16,color_FFFFFF,t_70)

已知$Q_{11} = (x_1, y_1)$，$Q_{12} = (x_1, y_2)$，$Q_{21} = (x_2, y_1)$，$Q_{22} = (x_2, y_2)$四个点的值。

首先在x方向进行线性插值，得到

$$f(R_1) \approx \frac{x_2 - x}{x_2 - x_1}f(Q_{11}) + \frac{x - x_1}{x_2 - x_1}f(Q_{21})  \quad where R_1 =(x, y_1)$$

$$f(R_2) \approx \frac{x_2 - x}{x_2 - x_1}f(Q_{12}) + \frac{x - x_1}{x_2 - x_1}f(Q_{22}) \quad where R_1 =(x, y_1)$$

然后在y方向插值，得到

$$f(P) \approx \frac{y_2 - y}{y_2 - y_1} f(R_1) + \frac{y - y_1} {y_2 - y_1} f(R_2)$$

代码使用与最近邻插值基本一样，他的原理相对烦一点，想详细了解的可以参考这篇博客：[CV03-双线性差值pytorch实现](https://blog.csdn.net/qpeity/article/details/104257203)，有关插值算法可以参考这篇博客[图像常见插值算法——超分辨率(四)](<https://blog.csdn.net/xu_fu_yong/article/details/96593377>)

#### 3.PyTorch有一个函数可以直接代替上面两种插值方法使用

interpolate函数：`torch.nn.functional.interpolate(input,size=None,scale_factor=None,mode='nearest',align_corners=None)`

其中scale_factor 在高度、宽度和深度上面的放大倍数。mode 上采样的方法，包括最近邻（nearest），线性插值（linear），双线性插值（bilinear），三次线性插值（trilinear），默认是最近邻（nearest）。使用起来很方便。
