---
layout: post
title: PyTorch理解并调整Tensor的维度
subtitle: false
gh-repo: Kakuoo/kakuoo.github.io
gh-badge: [star, fork, follow]
tags: [PyTorch]
comments: true
---
* This will become a table of contents (this text will be scraped).
{:toc}

## 一、理解tensor的维度

TensorFlow对张量的阶、维度、形状有着明确的定义，而在pytorh中对其的定义却模糊不清，仅仅有一个torch.size()的函数来查看张量的大小（我理解的这个大小指的就是TensorFlow对张量的形状描述，也和numpy的.shape类似）。所以，首先要搞清楚如何看一个张量的形状。

```python
import torch

z = torch.ones(2,3,4)
print(z)
print(z.size())
print(z.size(0))
print(z.size(1))
print(z.size(2))
```


以上代码的控制台输出为：

```python
tensor([[[1., 1., 1., 1.],
         [1., 1., 1., 1.],
         [1., 1., 1., 1.]],

        [[1., 1., 1., 1.],
     	 [1., 1., 1., 1.],
    	 [1., 1., 1., 1.]]])

torch.Size([2, 3, 4])
2
3
4
```

可见，我们成功创建了一个（2，3，4）大小的张量，那么我们人工应该怎么辨别一个张量的大小呢？为了直观，我把这个张量的中括号调整一个位置：

```python
[
    [
         [1., 1., 1., 1.],
         [1., 1., 1., 1.],
         [1., 1., 1., 1.]
    ],
    [
         [1., 1., 1., 1.],
         [1., 1., 1., 1.],
         [1., 1., 1., 1.]
    ]
]
```

 我们可以看到：

**第一层（最外层）中括号里面包含了两个中括号（以逗号进行分割），这就是（2，3，4）中的2**

**第二层中括号里面包含了三个中括号（以逗号进行分割），这就是（2，3，4）中的3**

**第三层中括号里面包含了四个数（以逗号进行分割），这就是（2，3，4）中的4**

结论：**pytorch中的tensor维度可以通过第一个数前面的中括号数量来判断，有几个中括号维度就是多少。拿到一个维度很高的向量，将最外层的中括号去掉，数最外层逗号的个数，逗号个数加一就是最高维度的维数，如此循环，直到全部解析完毕。**

我们还看到：

z.size(0) = 2，z.size(1) = 3，z.size(2) = 4

第0维度为2，第1维度为3，第2维度为4，即维度的标号是以0开始的，这点在squeeze()和unsqueeze()的形参说明中可以得到印证。

## 二、squeeze()和unsqueeze()

### 1. torch.squeeze(input, dim=None, out=None)

  当不给定dim时，将输入张量形状中的1 去除并返回。 如果输入是形如(A×1×B×1×C×1×D)(A×1×B×1×C×1×D)，那么输出形状就为： (A×B×C×D)(A×B×C×D)

当给定dim时，那么挤压操作只在给定维度上。即若tensor.size(dim) = 1，则去掉该维度

例如，输入形状为: (A×1×B)(A×1×B), squeeze(input, 0) 将会保持张量不变，只有用 squeeze(input, 1)，形状会变成 (A×B)(A×B)。

注意： 返回张量与输入张量共享内存，所以改变其中一个的内容会改变另一个。

参数:

- input (Tensor) – 输入张量
- dim (int, optional) – 如果给定，则input只会在给定维度挤压，维度的索引（从0开始）
- out (Tensor, optional) – 输出张量

### 2.torch.unsqueeze(input, dim, out=None)

返回一个新的张量，对输入的指定位置插入维度 1

注意： 返回张量与输入张量共享内存，所以改变其中一个的内容会改变另一个。

如果dim为负，则将会被转化dim+input.dim()+1dim+input.dim()+1

参数:

- tensor (Tensor) – 输入张量
- dim (int) – 插入维度的索引（从0开始）
- out (Tensor, optional) – 结果张量

```python
import torch

x = torch.ones(4)
print(x)
print(x.size())

y = torch.unsqueeze(x, 0)
print(y)
print(y.size())

z = torch.unsqueeze(x, 1)
print(z)
print(z.size())
```


 插入维度之前：

[ 1, 1, 1, 1 ]

在第0维插入一个维度，使其变成（1，4），即在最外层插入一个中括号即可：

[ [ 1, 1, 1, 1 ] ]

在第1维插入一个维度，使其变成（4，1）

[ [1], [1], [1], [1] ]

程序输出如下：

```python
tensor([1., 1., 1., 1.])
torch.Size([4])
tensor([[1., 1., 1., 1.]])
torch.Size([1, 4])
tensor([[1.],
        [1.],
        [1.],
        [1.]])
torch.Size([4, 1])
```
