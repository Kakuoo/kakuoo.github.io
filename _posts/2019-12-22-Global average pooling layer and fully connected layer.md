---
layout: post
title: 全局平均池化层和全连接层的作用辨析
subtitle: finetune是一门学问
tags: [Deep Learning]
---

<!-- ## 全局平均池化层和全连接层的作用辨析 -->

在卷积神经网络的初期，卷积层通过池化层（一般是 最大池化）后总是要一个或n个全连接层，最后在softmax分类。其特征就是全连接层的参数超多，使模型本身变得非常臃肿。

之后，有大牛在NIN（Network in Network）论文中提到了使用全局平局池化层代替全连接层的思路，以下是摘录的一部分资料：

> Global average pooling。既然全连接网络可以使feature map的维度减少，进而输入到softmax，但是又会造成过拟合，是不是可以用pooling来代替全连接。
> 答案是肯定的，Network in Network工作使用GAP来取代了最后的全连接层，直接实现了降维，更重要的是极大地减少了网络的参数(CNN网络中占比最大的参数其实后面的全连接层)。Global average pooling的结构如下图所示:

![img](https://pic3.zhimg.com/80/v2-1b18801a5cc4ebd994aeb9d5d84377fa_720w.jpg)

由此就可以比较直观地说明了。这两者合二为一的过程我们可以探索到GAP的真正意义是:**对整个网路在结构上做正则化防止过拟合**。其直接剔除了全连接层中黑箱的特征，**直接赋予了每个channel实际的类别意义**。
实践证明其效果还是比较可观的，同时GAP可以实现任意图像大小的输入。但是值得我们注意的是，使用gap可能会造成收敛速度减慢。



但是，全局平均池化层比较全连接层，为什么会收敛速度变慢，它们对模型的训练有什么差异呢？我没有找到相关的文章的介绍。**以下是发挥我自己的想象（很有可能是错误的）**来理解的几个点：

1.全连接层结构的模型，对于训练学习的过程，**全连接网络部分会大幅提升模型的复杂度，增加模型参数量，容易造成模型过拟合**，直观上相比于卷积网络部分，**学习任务更多的由全连接部分承担**。这意味着，**卷积的特征学习的低级一些**，没有关系，全连接不断学习调整参数，一样能很好的分类。此处是完全猜测，没有道理。

2.**全局平均池化层代替全连接层的模型，学习训练的压力全部前导到卷积层**。卷积的特征学习相较来说要更为"高级"一些。（因此收敛速度变慢?）为什么这么想呢？我的理解是，**全局平均池化较全连接层，应该会淡化不同特征间的相对位置的组合关系（“全局”的概念即如此）**。因此，卷积训练出来的特征应该更加“高级”。

3.以上的两个观点联合起来，可以推导出，**全局平均池化层代替全连接层虽然有好处，但是不利于迁移学习。**因为参数较为“固化”在卷积的诸层网络中。增加新的分类，那就意味着相当数量的卷积特征要做调整。而全连接层模型则可以更好的迁移学习，因为它的参数很大一部分调整在全连接层，迁移的时候卷积层可能也会调整，但是相对来讲要小的多了。



参考文献：[Crazymxm的知乎博客](https://zhuanlan.zhihu.com/p/46235425)