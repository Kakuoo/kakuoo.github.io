---
layout: post
title: 计算机视觉 注意力机制（Attention Mechanism）综述
# subtitle: 
tags: [Computer Vision]
---

<!-- ## 计算机视觉 注意力机制（Attention Mechanism）综述 -->

## 综述

注意力机制（Attention Mechanism）源于对人类视觉的研究。在认知科学中，由于信息处理的瓶颈，人类会选择性地关注所有信息的一部分，同时忽略其他可见的信息。上述机制通常被称为注意力机制。人类视网膜不同的部位具有不同程度的信息处理能力，即敏锐度（Acuity），只有视网膜中央凹部位具有最强的敏锐度。为了合理利用有限的视觉信息处理资源，人类需要选择视觉区域中的特定部分，然后集中关注它。例如，人们在阅读时，通常只有少量要被读取的词会被关注和处理。综上，注意力机制主要有两个方面

- 决定需要关注输入的哪部分。
- 分配有限的信息处理资源给重要的部分。

计算机视觉（computer vision）中的注意力机制（attention）的基本思想就是想让系统学会注意力——能够忽略无关信息而关注重点信息。本文关注的领域是计算机视觉中的注意力机制，同时在自然语言处理（NLP）或者视觉问答系统（VQA）中也有对应的注意力机制，可以相关文章可以看[Attention模型方法综述](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247489853&idx=1&sn=5e2949f97de263de5208fa990c0287e2&chksm=96e9c6bda19e4fabf790489d5c1e09f5af864c80d1f4b40257f2a071b99a0832317e2aa7620d&mpshare=1&scene=21&srcid=0611VZs4bFBSUSNPuiuAUbOL#wechat_redirect)。

在计算机视觉领域，注意力机制被引入来进行视觉信息处理。注意力是一种机制，或者方法论，并没有严格的数学定义。比如，传统的局部图像特征提取、显著性检测、滑动窗口方法等都可以看作一种注意力机制。在神经网络中，注意力模块通常是一个额外的神经网络，能够硬性选择输入的某些部分，或者给输入的不同部分分配不同的权重。本文的注意力机制主要指代神经网络中的注意力机制。

近几年来，深度学习与视觉注意力机制结合的研究工作，大多数是集中于使用**掩码(mask)**来形成注意力机制。掩码的原理在于通过另一层新的权重，将图片数据中关键的特征标识出来，通过学习训练，让深度神经网络学到每一张新图片中需要关注的区域，也就形成了注意力。

注意力机制可以分为四类：基于输入项的柔性注意力（Item-wise Soft Attention）、基于输入项的硬性注意力（Item-wise Hard Attention）、基于位置的柔性注意力（Location-wise Soft Attention）、基于位置的硬性注意力（Location-wise Hard Attention）。

对于基于项的注意力和基于位置的注意力，它们的输入形式是不同的。基于项的注意力的输入需要是包含明确的项的序列，或者需要额外的预处理步骤来生成包含明确的项的序列（这里的项可以是一个向量、矩阵，甚至一个特征图）。而基于位置的注意力则是针对输入为一个单独的特征图设计的，所有的目标可以通过位置指定。

这种思想，进而演化成两种不同类型的注意力，一种是**软注意力(soft attention)**，另一种则是**硬注意力(hard attention)**。

软注意力的关键点在于，这种注意力更关注区域或者通道，而且**软注意力是确定性的注意力**，学习完成后直接可以通过网络生成，最关键的地方是<font color=#00ffff>软注意力是可微的 </font>，这是一个非常重要的地方。可以微分的注意力就可以通过神经网络算出梯度并且前向传播和后向传播来学习得到注意力的权重。

强注意力与软注意力不同点在于，首先强注意力是更加关注点，也就是图像中的每个点都有可能延伸出注意力，同时强注意力是一个随机的预测过程，更强调动态变化。当然，最关键是硬注意力是一个不可微的注意力，训练过程往往是通过**增强学习(reinforcement learning)**来完成的

为了更清楚地介绍计算机视觉中的注意力机制，这篇文章将从注意力域（attention domain）的角度来分析几种注意力的实现方法。其中主要是三种注意力域，**空间域(spatial domain)**，**通道域(channel domain)**，**混合域(mixed domain)。**

还有另一种比较特殊的硬注意力实现的注意力域，**时间域(time domain)**，但是因为硬注意力是使用reinforcement learning来实现的，训练起来有所不同，所以之后再详细分析。

## 软注意力的注意力域（Soft Attention）

为了将问题能够更快的展现，我会介绍三篇文章，通过三篇文章中的注意力域的不同来介绍如何实现具有注意力机制的深度学习模型。每篇文章的介绍分为两个部分，首先从想法上来介绍模型的设计思路，然后深入了解模型结构(architecture)部分。

### 空间域（Spatial Domain）

**设计思路：**

**[Spatial Transformer Networks（STN）](<https://arxiv.org/pdf/1506.02025.pdf>)**[^1]模型是15年NIPS上的文章，这篇文章通过注意力机制，将原始图片中的空间信息变换到另一个空间中并保留了关键信息。

这篇文章的思想非常巧妙，因为卷积神经网络中的池化层（pooling layer）直接使用max pooling 或者average pooling 的方法，将图片信息压缩，减少运算量提升准确率。但是这篇文章认为之前pooling的方法太过于暴力，直接将信息合并会导致关键信息无法识别出来，所以提出了一个叫空间转换器（spatial transformer）的模块，将图片中的的空间域信息做对应的空间变换，从而能将关键的信息提取出来。

> Unlike pooling layers, where the receptive fields are fixed and local, the spatial transformer module is a dynamic mechanism that can actively spatially transform an image (or a feature map) by producing an appropriate transformation for each input sample.

**模型结构：**

<!-- ![STN](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\STN.png) -->
![STN]({{site.url}}/assets/blog_images/attention_mechanism/STN.png)


- (a)列是原始的图片信息，其中第一个手写数字7没有做任何变换，第二个手写数字5，做了一定的旋转变化，而第三个手写数字6，加上了一些噪声信号；
- (b)列中的彩色边框是学习到的spatial transformer的框盒（bounding box），每一个框盒其实就是对应图片学习出来的一个spatial transformer；
- (c)列中是通过spatial transformer转换之后的特征图，可以看出7的关键区域被选择出来，5被旋转成为了正向的图片，6的噪声信息没有被识别进入；
- 最终可以通过这些转换后的特征图来预测出（d）列中手写数字的数值。

spatial transformer其实就是注意力机制的实现，因为训练出的spatial transformer能够找出图片信息中需要被关注的区域，同时这个transformer又能够具有旋转、缩放变换的功能，这样图片局部的重要信息能够通过变换而被框盒提取出来。

![Spatial Transformer](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\Spatial Transformer.png)

这是空间变换网络（spatialtransformer network）中最重要的**空间变换模块**，这个模块可以作为新的层直接加入到原有的网络结构，比如ResNet中。

模块输入$$U \in \R^{H \times W\times C} $$，神经网络训练中使用的数据类型都是张量(tensor)，H是上一层tensor的高度(height)，W是上一层tensor的宽度(width)，而C代表tensor的通道(channel)，比如图片基本的三通道（RGB），或者是经过卷积层(convolutional layer)之后，不同卷积核(kernel)都会产生不同的通道信息。

之后这个输入进入两条路线，一条路线是信息进入定位网络（localisation net），另一条路线是原始信号直接进入采样层（sampler）。

其中定位网络会学习到一组参数θ，而这组参数就能够作为网格生成器（grid generator）的参数，生成一个采样信号，这个采样信号其实是一个变换矩阵（绿色部分），与原始图片相乘之后，可以得到变换之后的矩阵$$V \in \R^{H' \times W' \times C'} $$，这个V也就是变换之后的图片特征了，变换之后的矩阵大小是可以通过调节变换矩阵来形成缩放的。

<!-- ![TransformMatrix](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\TransformMatrix.png) -->
![TransformMatrix]({{site.url}}/assets/blog_images/attention_mechanism/TransformMatrix.png)

通过这张转换图片，可以看出空间转换器中产生的采样矩阵是能够将原图中关键的信号提取出来，（a）中的采样矩阵是单位矩阵，不做任何变换，（b）中的矩阵是可以产生缩放旋转变换的采样矩阵。

这个模块加进去最大的好处就是能够对上一层信号的关键信息进行识别(attention)，并且该信息矩阵是一个**可以微分**的矩阵，因为每一个目标（target）点的信息其实是所有源（source）点信息的一个组合，这个组合可以是一个线性组合，复杂的变换信息也可以用**核函数(kernel)**来表示：V是转换后的信息，U是转换前的信息，k是一个核函数。

理论上来说，这样的模块是可以加在**任意层**的，因为模块可以同时对通道信息和矩阵信息同时处理。

但是由于文章提出对所有的通道信息进行统一处理变换，我认为这种模块其实更适用于**原始图片输入层**之后的变化，因为卷积层之后，每一个卷积核(filter)产生的通道信息，所含有的信息量以及重要程度其实是不一样的，都用同样的transformer其实可解释性并不强。也由此，我们可以引出第二种注意域的机制——通道域(channel domain)注意力机制。

### 通道域（Channel Domain）

**设计思路：**

通道域的注意力机制原理很简单，我们可以从基本的信号变换的角度去理解。信号系统分析里面，任何一个信号其实都可以写成正弦波的线性组合，经过时频变换之后，时域上连续的正弦波信号就可以用一个频率信号数值代替了。

<!-- ![Time&Frequency](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\Time&Frequency.jpg) -->
![Time&Frequency]({{site.url}}/assets/blog_images/attention_mechanism/Time&Frequency.jpg)

在卷积神经网络中，每一张图片初始会由（R，G，B）三通道表示出来，之后经过不同的卷积核之后，每一个通道又会生成新的信号，比如图片特征的每个通道使用64核卷积，就会产生64个新通道的矩阵（H,W,64），H,W分别表示图片特征的高度和宽度。

每个通道的特征其实就表示该图片在不同卷积核上的分量，类似于时频变换，而这里面用卷积核的卷积类似于信号做了傅里叶变换，从而能够将这个特征一个通道的信息给分解成64个卷积核上的信号分量。

<!-- ![filters](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\filters.jpg) -->
![filters]({{site.url}}/assets/blog_images/attention_mechanism/filters.jpg)

既然每个信号都可以被分解成核函数上的分量，产生的新的64个通道对于关键信息的贡献肯定有多有少，如果我们给每个通道上的信号都增加一个权重，来代表该**通道与关键信息的相关度**的话，这个权重越大，则表示相关度越高，也就是我们越需要去注意的通道了。

**模型结构：**

[Squeeze-and-Excitation Networks](<https://arxiv.org/pdf/1709.01507.pdf>)[^2]中提出了一个非常重要的SENet的模型结构，靠着这个模型获得了ImageNet的冠军，这个模型是非常有创造力的设计。

<!-- ![SENet](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\SENet.png) -->
![SENet]({{site.url}}/assets/blog_images/attention_mechanism/SENet.png)

首先最左边是原始输入图片特征X，然后经过变换，比如卷积变换$F_{tr}$，产生了新的特征信号U。U有C个通道，我们希望通过注意力模块来学习出每个通道的权重，从而产生**通道域的注意力**.

中间的模块就是SENet的创新部分，也就是注意力机制模块。这个注意力机制分成三个部分：**挤压(squeeze)**，**激励(excitation)**，以及**注意(attention)**。

挤压函数$F_{sq}$做了一个全局平均值，把每个通道内所有的特征值相加再平均，也是全局平均池化（global average pooling）的数学表达式。

激励函数$F_{ex}$，其中δ函数是ReLU，而σ是一个sigmoid激活函数。W1和W2的维度分别是$w_1 \in  \R^{\frac{C}{r} \times C}$和$w_2 \in  \R^{C \times  \frac{C}{r}}$。通过训练学习这两个权重，得到一个一维的激励权重来激活每一层通道。

尺度函数$F_{scale}$，其实就是一个放缩的过程，不同通道的值乘上不同的权重，从而可以**增强对关键通道域**的注意力。

### 混合域

了解前两种注意力域的设计思路后，简单对比一下。首先，空间域的注意力是忽略了通道域中的信息，将每个通道中的图片特征同等处理，这种做法会将空间域变换方法局限在原始图片特征提取阶段，应用在神经网络层其他层的可解释性不强。

而通道域的注意力是对一个通道内的信息直接全局平均池化，而忽略每一个通道内的局部信息，这种做法其实也是比较暴力的行为。所以结合两种思路，就可以设计出混合域的注意力机制模型[8]。

**设计思路：**

这篇文章中提出的注意力机制是与深度残差网络(Deep Residual Network)相关的方法，基本思路是能够将注意力机制应用到ResNet中，并且使网络能够训练的比较深。

[Residual Attention Network for Image Classification](<https://arxiv.org/pdf/1704.06904.pdf>)[^3]注意力的机制是软注意力基本的**加掩码(mask)**机制，但是不同的是，这种注意力机制的mask借鉴了残差网络的想法，不只根据当前网络层的信息加上mask，还把上一层的信息传递下来，这样就防止mask之后的信息量过少引起的网络层数不能堆叠很深的问题。

正如之前说的，文章中提出的注意力mask，不仅仅只是对空间域或者通道域注意，这种mask可以看作是**每一个特征元素（element）的权重**。通过给每个特征元素都找到其对应的注意力权重，就可以同时形成了空间域和通道域的注意力机制。

很多人看到这里就会有疑问，这种做法应该是从空间域或者通道域非常自然的一个过渡，怎么做单一域注意力的人都没有想到呢？原因有：

- 如果你给每一个特征元素都赋予一个mask权重的话，mask之后的信息就会非常少，可能直接就破坏了**网络深层的特征信息**；
- 另外，如果你可以加上注意力机制之后，残差单元（Residual Unit）的恒等映射（identical mapping）特性会被破坏，从而很难训练。

所以该文章的注意力机制的创新点在于提出了**残差注意力学习(residual attention learning)**，不仅只把mask之后的特征张量作为下一层的输入，同时也将mask之前的特征张量作为下一层的输入，这时候可以得到的特征更为丰富，从而能够更好的注意关键特征。

**模型结构：**

文章中模型结构是非常清晰的，整体结构上，是三阶注意力模块(3-stage attention module)。每一个注意力模块可以分成两个分支(看stage2)，上面的分支叫**主分支(trunk branch)**，是基本的残差网络(ResNet)的结构。而下面的分支是**软掩码分支(soft mask branch)**，而软掩码分支中包含的主要部分就是残差注意力学习机制。通过下采样(down sampling)和上采样(up sampling)，以及残差模块(residual unit)，组成了注意力的机制，在每个module中均使用bottom-up top-down结构，**The bottom-up top-down structure mimics the fast feedforward and feedback attention process**。

<!-- ![Residual Attention Network for Image Classification](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\Residual Attention Network for Image Classification.png) -->
![F_function]({{site.url}}/assets/blog_images/attention_mechanism/AttentionforClassification.png)

模型结构中比较创新的残差注意力机制是：

$H_{i,c}(x) = (1+M_{i,c(x)}) * F_{i,c}(x)$

H是注意力模块的输出，F是上一层的图片张量特征，M是软掩码的注意力参数。这就构成了残差注意力模块，能将图片特征和加强注意力之后的特征一同输入到下一模块中。F函数可以选择不同的函数，就可以得到不同注意力域的结果：

<!-- ![F_function](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\F_function.png) -->
![F_function]({{site.url}}/assets/blog_images/attention_mechanism/F_function.png)

- $f_{1}$是对图片特征张量直接sigmoid激活函数，就是**混合域的注意力**；
- $f_{2}$是对图片特征张量直接做全局平均池化（global average pooling），所以得到的是**通道域的注意力**（类比SENet）；
- $f_{3}$是求图片特征张量在通道域上的平均值的激活函数，类似于忽略了通道域的信息，从而得到**空间域的注意力**。

### 时间域注意力

这个概念其实比较大，因为计算机视觉只是单一识别图片的话，并没有时间域这个概念，但是[Recurrent Attention Model](<https://papers.nips.cc/paper/5542-recurrent-models-of-visual-attention.pdf>)[^4]这篇文章中，提出了一种基于递归神经网络（Recurrent Neural Network，RNN）的注意力机制识别模型。

RNN模型比较适合的场景是数据具有时序特征，比如使用RNN产生注意力机制做的比较好的是在自然语言处理的问题上。因为自然语言处理的是文本分析，而文本产生的背后其实是有一个时序上的关联性，比如一个词之后还会跟着另外一个词，这就是一个时序上的依赖关联性。

而图片数据本身，并不具有天然的时序特征，一张图片往往是一个时间点下的采样。但是在**视频数据**中，RNN就是一个比较好的数据模型，从而能够使用RNN来产生识别注意力。

特意将RNN的模型称之为时间域的注意力，是因为这种模型在前面介绍的空间域，通道域，以及混合域之上，又新增加了一个时间的维度。这个维度的产生，其实是基于采样点的时序特征。

Recurrent Attention Model 中将注意力机制看成对一张图片上的一个区域点的采样，这个采样点就是需要注意的点。而这个模型中的注意力因为不再是一个可以微分的注意力信息，因此这也是一个硬注意力（hard attention）模型。这个模型的训练是需要使用**增强学习（reinforcementlearning）**来训练的，训练的时间更长。

这个模型更需要了解的并不是RNN注意力模型，因为这个模型其实在自然语言处理中介绍的更详细，更需要了解的是这个模型的如何将图片信息转换成时序上的采样信号的

<!-- ![GlimpseSensor](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\GlimpseSensor.png) -->
![GlimpseSensor]({{site.url}}/assets/blog_images/attention_mechanism/GlimpseSensor.png)

这个是模型中的关键点，叫**Glimpse Sensor**，我翻译为**扫****视器**，这个sensor的关键点在于先确定好图片中需要关注的点（像素），这时候这个sensor开始采集三种信息，信息量是相同的，一个是非常细节（最内层框）的信息，一个是中等的局部信息，一个是粗略的略缩图信息。

这三个采样的信息是在$l_{t-1}$位置中产生的图片信息，而下一个时刻，随着t的增加，采样的位置又开始变化，至于l随着t该怎么变化，这就是需要使用增强学习来训练的东西了。

有关RNN做attention的，还是应该去了解自然语言处理，如机器翻译中的做法，这里就不再继续深入介绍，想深入了解的，推荐阅读[Attention模型方法综述](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247489853&idx=1&sn=5e2949f97de263de5208fa990c0287e2&chksm=96e9c6bda19e4fabf790489d5c1e09f5af864c80d1f4b40257f2a071b99a0832317e2aa7620d&mpshare=1&scene=21&srcid=0611VZs4bFBSUSNPuiuAUbOL#wechat_redirect)。



[^1]: Jaderberg, Max, Karen Simonyan, and AndrewZisserman. "Spatial transformer networks." Advances in neural information processing systems. 2015.
[^2]: Hu, Jie, Li Shen, and Gang Sun."Squeeze-and-excitation networks." arXiv preprintarXiv:1709.01507 (2017).
[^3]: Wang, Fei, et al. "Residual attentionnetwork for image classification." arXiv preprint arXiv:1704.06904 (2017).
[^4]: Mnih, Volodymyr, Nicolas Heess, and AlexGraves. "Recurrent models of visual attention." Advances inneural information processing systems. 2014.



