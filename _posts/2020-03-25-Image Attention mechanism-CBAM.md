---
layout: post
title: 图像Attention机制——CBAM
# subtitle: 
tags: [Computer Vision]
---

<!-- ## 图像Attention机制——CBAM -->

**paper:** [[CBAM: Convolutional Block Attention Module](<https://arxiv.org/abs/1807.06521v2>)]

## 1. 摘要：

卷积块的注意力模块（Convolutional Block Attention Module），简称CBAM，该模块是一个简单高效的前向卷积神经网络注意力模块。给定一张特征图，CBAM沿着通道（channel）和空间（spatial）两个单独的维度依次推断注意力图，然后将注意力图和输入特征图相乘，进行自适应特征细化。因为CBAM是一个轻量级的通用模块，可以无缝的集成到任何CNN架构中，几乎对效率，算力没有影响，能够实现端到端的训练。

## 2. 介绍：

卷积神经网络凭借其强大的特征提取和表达能力，在计算机视觉任务中取得了很好的应用效果，为了进一步提升CNNs的性能，近来的方法会从三个方面考虑：**深度，宽度，基数。**

在深度方面，VGGNet已经证明，堆积相同形状的卷积块能取得不错的效果，基于同样的思想，ResNet在堆积的基础上，加入skip connection，通过残差学习，在保证性能不退化的基础上，大大加深了网络层数。

<!-- ![resnet](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\Resnet.png) -->
![resnet]({{site.url}}/assets/blog_images/attention_mechanism/Resnet.png)

在网络宽度方面，GoogleNet是一个重要尝试，该结构将CNN中常用的卷积（1x1，3x3，5x5）、池化操作（3x3）堆叠在一起（卷积、池化后的尺寸相同，将通道相加），增加了网络的宽度。

<!-- ![Inception](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\Inception.jpg) -->
![Inception]({{site.url}}/assets/blog_images/attention_mechanism/Inception.png)

在网络基数方面，Facebook的ResNeXt是一个很好的工作，何为基数（cardinality）呢？Cardinatity指的是一个block中所具有的相同分支的数目。ResNeXt通过一系列实验证明了，加大基数能够达到甚至超越加大宽度和深度的效果，具有更强的特征表现能力。

<!-- ![ResnetX](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\ResnetX.png) -->
![ResnetX]({{site.url}}/assets/blog_images/attention_mechanism/ResnetX.png)

除了这些因素，作者则研究了网络架构设计的另一个不同方向：注意力机制。最近几年，在计算机视觉领域，颇有点"万物皆可attention"的意思，涌现了很多基于attention的工作，attention不仅要告诉我们重点关注哪里，还要提高关注点的表示。 我们的目标是通过使用注意机制来增加表现力，关注重要特征并抑制不必要的特征。

从总体上说，注意力机制分为三个方面：空间域，通道域和混合域。本文是通道域和空间域，为了强调通道和空间这两个维度上的有意义特征，作者将注意力过程分为两个独立的部分，通道注意力模块和空间注意力模块。来分别在通道和空间维度上学习关注什么、在哪里关注。这样不仅可以节约参数和计算力，而且保证了其可以作为即插即用的模块集成到现有的网络架构中去。此外，通过了解要强调或抑制的信息也有助于网络内的信息流动。

## 3. 网络结构

本文提出的CBAM网络（Convolutional Block Attention Module）。如下图所示，依次用到注意力机制中的通道域模块和空间域模块，通过这两个模块，得到细化后的feature。网络具有了学习“What”和“Where”的能力，让网络更好的知道哪些信息需要强调，哪些信息需要抑制。

<!-- ![CBAM](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\CBAM.png) -->
![CBAM]({{site.url}}/assets/blog_images/attention_mechanism/CBAM.png)

通道注意力和空间注意力这两个模块可以以并行或者顺序的方式组合在一起，但是作者发现顺序组合并且将通道注意力放在前面可以取得更好的效果。

### 通道注意力模块（Channel attention module）

通道域的核心思想是，对经过卷积得到的特征图的每一层，乘以不同的权重，表示该层表示的特征对于关键信息的关联程度和重要程度，相应的，权重越大，表示该层表示的信息对于关键信息越重要，关联程度越高；权重越小，表示该层表示的信息对于关键信息越不重要。

<!-- ![channel_attention](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\channel_attention.png) -->
![channel_attention]({{site.url}}/assets/blog_images/attention_mechanism/channel_attention.png)

如上图所示，输入是一个 H×W×C 的特征 F，我们先分别进行一个空间的全局平均池化和全局最大池化得到两个 1×1×C 的通道描述。接着，再将它们分别送入一个两层的神经网络，第一层神经元个数为 C/r，激活函数为 Relu，第二层神经元个数为 C。注意，这个两层的神经网络是共享的。然后，再将得到的两个特征相加后经过一个 Sigmoid 激活函数得到权重系数 Mc。最后，拿权重系数和原来的特征 F 相乘即可得到缩放后的新特征。

输入： H×W×C  feature map

输出：1x1xC channel attention featrue map

步骤：

1. 对于输入feature map，分别进行平均池化和最大池化聚合空间信息，得到两个C维池化特征图，F_avg和F_max
2. 将F_avg和F_max送入包含一个隐层的多层感知器MLP里，得到两个1x1xC的通道注意力图。其中，为了减少参数量，隐层神经元的个数为C/r，r也被称作压缩比
3. 将经过MLP得到的两个通道注意力图进行对应元素相加，激活，得到最终的通道注意力图Mc



### 空间注意力模块（Spatial attention module）

空间域的设计思路是通过注意力机制，更关注的是位置特性。将原始图片中的空间信息通过空间转换模块，变换到另一个空间中并保留关键信息。

<!-- ![spatial_attention](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\spatial_attention.png) -->
![spatial_attention]({{site.url}}/assets/blog_images/attention_mechanism/spatial_attention.png)


与通道注意力相似，给定一个 H×W×C 的特征 F‘，先分别进行通道维度上的平均池化和最大池化得到两个 H×W×1 的通道描述，并将这两个描述按照通道拼接在一起。然后，经过一个 7×7 的卷积层，激活函数为 Sigmoid，得到权重系数 Ms。最后，拿权重系数和特征 F’ 相乘即可得到缩放后的新特征。

输入： H×W×C  Channel-refined feature（经过通道注意力模块精调后的feature map）

输出：1HxW spatial featrue  map

步骤：

1. 对于F’首先沿着通道方向进行最大池化和平均池化，得到两个二维的feature map F_avg和F_max，属性都是1xHxW，将得到的两个feature map进行维度拼接（concatenate），得到拼接后的feature map
2. 对于拼接后的feature map，利用size为7x7（或其他尺寸）的卷积层生成空间注意力图Ms。

## 4.问题答疑

（1）为什么要对输入特征进行全局平均池化和全局最大池化两种处理方式？

为了更好的聚合feature map的信息并减少参数量，池化是必不可少的步骤，对于空间信息的聚合，目前普遍采用的是平均池化的方法，但作者认为，最大池化会收集到不同于平均池化的，关于不同目标特征的表示，这对于后续得到更精细的注意力通道图是有帮助的。实验也表明，对于平均池化和最大池化的综合运用，是有助于性能提升的。

（2）MLP的结构是如何设计的？

本文用到的是只有一层hidden layer的MLP，非常简单，以W0和W1分别表示隐层权重和输出层权重，则W0和W1的参数是共享的。

（3）为什么要沿着通道维度进行平均池化和最大池化？

作者借鉴ICLR2017的论文《Paying more attention to attention: Improving the performance of convolutional neural networks via attention transfer》，认为沿着通道轴应用池化操作可以有效地突出显示含有关键信息的区域。作者的实验也支持这一观点。

（4）维度拼接（Concatenate）具体操作过程？

Concatenate的具体操作是将两个size相同的feature map，在维度方向进行拼接，拼接完成后，新的特征图的通道会是原来两个特征图通道之和。这种操作是进行**特征融合的普遍做法**。

## 5. 代码实现

### TensorFlow代码实现

```python
def CBAM(input, reduction):
    """
    @Convolutional Block Attention Module
    """

    _, width, height, channel = input.get_shape()  # (B, W, H, C)

    # channel attention
    x_mean = tf.reduce_mean(input, axis=(1, 2), keepdims=True)   # (B, 1, 1, C)
    x_mean = tf.layers.conv2d(x_mean, channel // reduction, 1, activation=tf.nn.relu, name='CA1')  # (B, 1, 1, C // r)
    x_mean = tf.layers.conv2d(x_mean, channel, 1, name='CA2')   # (B, 1, 1, C)

    x_max = tf.reduce_max(input, axis=(1, 2), keepdims=True)  # (B, 1, 1, C)
    x_max = tf.layers.conv2d(x_max, channel // reduction, 1, activation=tf.nn.relu, name='CA1', reuse=True)
    # (B, 1, 1, C // r)
    x_max = tf.layers.conv2d(x_max, channel, 1, name='CA2', reuse=True)  # (B, 1, 1, C)

    x = tf.add(x_mean, x_max)   # (B, 1, 1, C)
    x = tf.nn.sigmoid(x)        # (B, 1, 1, C)
    x = tf.multiply(input, x)   # (B, W, H, C)

    # spatial attention
    y_mean = tf.reduce_mean(x, axis=3, keepdims=True)  # (B, W, H, 1)
    y_max = tf.reduce_max(x, axis=3, keepdims=True)  # (B, W, H, 1)
    y = tf.concat([y_mean, y_max], axis=-1)     # (B, W, H, 2)
    y = tf.layers.conv2d(y, 1, 7, padding='same', activation=tf.nn.sigmoid)    # (B, W, H, 1)
    y = tf.multiply(x, y)  # (B, W, H, C)

    return y
```

### Pytorch代码实现

#### 通道注意力和空间注意力模块

```python
# 通道注意力模块
class ChannelAttention(nn.Module):
    def __init__(self, in_planes, ratio=16):
        super(ChannelAttention, self).__init__()
        self.avg_pool = nn.AdaptiveAvgPool2d(1)
        self.max_pool = nn.AdaptiveMaxPool2d(1)

        self.conv1   = nn.Conv2d(in_planes, in_planes // 16, 1, bias=False)
        self.relu1 = nn.ReLU()
        self.conv2   = nn.Conv2d(in_planes // 16, in_planes, 1, bias=False)

        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        avg_out = self.conv2(self.relu1(self.conv1(self.avg_pool(x))))
        max_out = self.conv2(self.relu1(self.conv1(self.max_pool(x))))
        out = avg_out + max_out
        return self.sigmoid(out)
        
# 空间注意力模块
class SpatialAttention(nn.Module):
    def __init__(self, kernel_size=7):
        super(SpatialAttention, self).__init__()

        assert kernel_size in (3, 7), 'kernel size must be 3 or 7'
        padding = 3 if kernel_size == 7 else 1

        self.conv1 = nn.Conv2d(2, 1, kernel_size, padding=padding, bias=False)
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        avg_out = torch.mean(x, dim=1, keepdim=True)
        max_out, _ = torch.max(x, dim=1, keepdim=True)
        x = torch.cat([avg_out, max_out], dim=1)
        x = self.conv1(x)
        return self.sigmoid(x)
```

#### 在Resnet152网络中添加CBAM模块

> **注意：**
>
> 因为不能改变ResNet的网络结构，所以CBAM不能加在block里面，因为加进去网络结构发生了变化，所以不能用预训练参数。加在第一层卷积和最后一层卷积不改变网络，可以用预训练参数
>
> **若依据原文中如下图所示，应该是描述Resnet34及以下的网络的block块中，应在每个`class BasicBlock(nn.Module)`的`def forward()`中添加CBAM模块**
>
> **若在Resnet50及以上的网络中，例如：Resnet152则应在每个`class Bottleneck(nn.Module)`中添加CBAM模块，而不是像下面代码里面一样在`Class Resnet(nn.Module)`的`def forward()`中添加，下文代码所示，仅供参考。**

<!-- ![CBAM_Resnet](D:\ProgramData\Git_Repository\Kakuoo.github.io\assets\blog_images\attention_mechanism\CBAM_Resnet.png) -->
![CBAM_Resnet]({{site.url}}/assets/blog_images/attention_mechanism/CBAM_Resnet.png)

```python
class ResNet(nn.Module):

    def __init__(self, block, layers, num_classes=1000, zero_init_residual=False,
                 groups=1, width_per_group=64, replace_stride_with_dilation=None,
                 norm_layer=None):
        super(ResNet, self).__init__()
        if norm_layer is None:
            norm_layer = nn.BatchNorm2d
        self._norm_layer = norm_layer

        self.inplanes = 64
        self.dilation = 1
        if replace_stride_with_dilation is None:
            # each element in the tuple indicates if we should replace
            # the 2x2 stride with a dilated convolution instead
            replace_stride_with_dilation = [False, False, False]
        if len(replace_stride_with_dilation) != 3:
            raise ValueError("replace_stride_with_dilation should be None "
                             "or a 3-element tuple, got {}".format(replace_stride_with_dilation))
        self.groups = groups
        self.base_width = width_per_group
        self.conv1 = nn.Conv2d(3, self.inplanes, kernel_size=7, stride=2, padding=3,
                               bias=False)
        self.bn1 = norm_layer(self.inplanes)
        self.relu = nn.ReLU(inplace=True)

        ##################################
        # 网络的第一层加入注意力机制
        self.ca0 = ChannelAttention(self.inplanes)
        self.sa0 = SpatialAttention()
        ##################################

        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
        self.layer1 = self._make_layer(block, 64, layers[0])
        self.layer2 = self._make_layer(block, 128, layers[1], stride=2,
                                       dilate=replace_stride_with_dilation[0])
        self.layer3 = self._make_layer(block, 256, layers[2], stride=2,
                                       dilate=replace_stride_with_dilation[1])
        self.layer4 = self._make_layer(block, 512, layers[3], stride=2,
                                       dilate=replace_stride_with_dilation[2])
        ##################################
        # 网络的最后一层加入注意力机制
        self.ca1 = ChannelAttention(self.inplanes)
        self.sa1 = SpatialAttention()
        ##################################

        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * block.expansion, num_classes)

        for m in self.modules():
            if isinstance(m, nn.Conv2d):
                nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')
            elif isinstance(m, (nn.BatchNorm2d, nn.GroupNorm)):
                nn.init.constant_(m.weight, 1)
                nn.init.constant_(m.bias, 0)

        # Zero-initialize the last BN in each residual branch,
        # so that the residual branch starts with zeros, and each residual block behaves like an identity.
        # This improves the model by 0.2~0.3% according to https://arxiv.org/abs/1706.02677
        if zero_init_residual:
            for m in self.modules():
                if isinstance(m, Bottleneck):
                    nn.init.constant_(m.bn3.weight, 0)
                elif isinstance(m, BasicBlock):
                    nn.init.constant_(m.bn2.weight, 0)

    def _make_layer(self, block, planes, blocks, stride=1, dilate=False):
        norm_layer = self._norm_layer
        downsample = None
        previous_dilation = self.dilation
        if dilate:
            self.dilation *= stride
            stride = 1
        if stride != 1 or self.inplanes != planes * block.expansion:
            downsample = nn.Sequential(
                conv1x1(self.inplanes, planes * block.expansion, stride),
                norm_layer(planes * block.expansion),
            )

        layers = []
        layers.append(block(self.inplanes, planes, stride, downsample, self.groups,
                            self.base_width, previous_dilation, norm_layer))
        self.inplanes = planes * block.expansion
        for _ in range(1, blocks):
            layers.append(block(self.inplanes, planes, groups=self.groups,
                                base_width=self.base_width, dilation=self.dilation,
                                norm_layer=norm_layer))

        return nn.Sequential(*layers)

    def forward(self, x):
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)

        ##################################
        x = self.ca0(x) * x
        x = self.sa0(x) * x
        ##################################

        x = self.maxpool(x)

        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)

        ##################################
        x = self.ca1(x) * x
        x = self.sa1(x) * x
        ##################################

        x = self.avgpool(x)
        x = x.reshape(x.size(0), -1)
        x = self.fc(x)

        return x
```



### Keras, TensorFlow代码实现

#### 通道注意力机制

```python
def channel_attention(input_feature, ratio=8):
	
	channel_axis = 1 if K.image_data_format() == "channels_first" else -1
	channel = input_feature._keras_shape[channel_axis]
	
	shared_layer_one = Dense(channel//ratio,
							 kernel_initializer='he_normal',
							 activation = 'relu',
							 use_bias=True,
							 bias_initializer='zeros')

	shared_layer_two = Dense(channel,
							 kernel_initializer='he_normal',
							 use_bias=True,
							 bias_initializer='zeros')
	
	avg_pool = GlobalAveragePooling2D()(input_feature)    
	avg_pool = Reshape((1,1,channel))(avg_pool)
	assert avg_pool._keras_shape[1:] == (1,1,channel)
	avg_pool = shared_layer_one(avg_pool)
	assert avg_pool._keras_shape[1:] == (1,1,channel//ratio)
	avg_pool = shared_layer_two(avg_pool)
	assert avg_pool._keras_shape[1:] == (1,1,channel)
	
	max_pool = GlobalMaxPooling2D()(input_feature)
	max_pool = Reshape((1,1,channel))(max_pool)
	assert max_pool._keras_shape[1:] == (1,1,channel)
	max_pool = shared_layer_one(max_pool)
	assert max_pool._keras_shape[1:] == (1,1,channel//ratio)
	max_pool = shared_layer_two(max_pool)
	assert max_pool._keras_shape[1:] == (1,1,channel)
	
	cbam_feature = Add()([avg_pool,max_pool])
	cbam_feature = Activation('hard_sigmoid')(cbam_feature)
	
	if K.image_data_format() == "channels_first":
		cbam_feature = Permute((3, 1, 2))(cbam_feature)
	
	return multiply([input_feature, cbam_feature])
```

#### 空间注意力机制

```python
def spatial_attention(input_feature):
	kernel_size = 7
	if K.image_data_format() == "channels_first":
		channel = input_feature._keras_shape[1]
		cbam_feature = Permute((2,3,1))(input_feature)
	else:
		channel = input_feature._keras_shape[-1]
		cbam_feature = input_feature
	
	avg_pool = Lambda(lambda x: K.mean(x, axis=3, keepdims=True))(cbam_feature)
	assert avg_pool._keras_shape[-1] == 1
	max_pool = Lambda(lambda x: K.max(x, axis=3, keepdims=True))(cbam_feature)
	assert max_pool._keras_shape[-1] == 1
	concat = Concatenate(axis=3)([avg_pool, max_pool])
	assert concat._keras_shape[-1] == 2
	cbam_feature = Conv2D(filters = 1,
					kernel_size=kernel_size,
					activation = 'hard_sigmoid',
					strides=1,
					padding='same',
					kernel_initializer='he_normal',
					use_bias=False)(concat)
	assert cbam_feature._keras_shape[-1] == 1
	
	if K.image_data_format() == "channels_first":
		cbam_feature = Permute((3, 1, 2))(cbam_feature)
		
	return multiply([input_feature, cbam_feature])
```

#### 构建CBAM

```python
def cbam_block(cbam_feature,ratio=8):
	"""Contains the implementation of Convolutional Block Attention Module(CBAM) block.
	As described in CBAM: Convolutional Block Attention Module.
	"""
	
	cbam_feature = channel_attention(cbam_feature, ratio)
	cbam_feature = spatial_attention(cbam_feature, )
	return cbam_feature
```

#### 在相应的位置添加CBAM

```python
inputs = x
residual = layers.Conv2D(filter, kernel_size = (1, 1), strides = strides, padding = 					'same')(inputs)
residual = layers.BatchNormalization(axis = bn_axis)(residual)
cbam = cbam_block(residual)
x = layers.add([x, residual, cbam])
```

