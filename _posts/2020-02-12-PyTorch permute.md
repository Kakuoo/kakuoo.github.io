---
layout: post
title: PyTorch之permute函数
subtitle: false
tags: [PyTorch]
---

<!-- ## PyTorch之permute函数 -->

### 官方中英文doc：

```python3
  torch.Tensor.permute (Python method, in torch.Tensor)
```

#### 1.1 permute(dims)

将tensor的维度换位。

参数： - __dims__ (int ..*) - 换位顺序

例：

```python3
>>> x = torch.randn(2, 3, 5) 
>>> x.size() 
torch.Size([2, 3, 5]) 
>>> x.permute(2, 0, 1).size() 
torch.Size([5, 2, 3])
```

#### 1.2 permute(\*dims) → Tensor

**Permute** the dimensions of this tensor.

**Parameters：** *dims (int...) – The **desired ordering** of dimensions

Example：

```python3
>>> x = torch.randn(2, 3, 5) 
>>> x.size() 
torch.Size([2, 3, 5]) 
>>> x.permute(2, 0, 1).size() 
torch.Size([5, 2, 3])
```

### permute的使用

permute函数功能还是比较简单的，下面主要介绍几个细节点：

#### transpose与permute的异同

**Tensor.permute(a,b,c,d, ...)**：permute函数可以对任意高维矩阵进行转置，但没有 torch.permute() 这个调用方式， 只能 Tensor.permute()：

```python3
>>> torch.randn(2,3,4,5).permute(3,2,0,1).shape
torch.Size([5, 4, 2, 3])
```

**torch.transpose(Tensor, a,b)**：**transpose只能操作2D矩阵的转置**，有两种调用方式；

另：连续使用transpose也可实现permute的效果：

```python3
>>> torch.randn(2,3,4,5).transpose(3,0).transpose(2,1).transpose(3,2).shape
torch.Size([5, 4, 2, 3])
>>> torch.randn(2,3,4,5).transpose(1,0).transpose(2,1).transpose(3,1).shape
torch.Size([3, 5, 2, 4])
```

从以上操作中可知，**permute相当于可以同时操作于tensor的若干维度，transpose只能同时作用于tensor的两个维度；**

#### permute函数与contiguous、view函数的关联

**contiguous**：view只能作用在contiguous的variable上，如果在view之前调用了transpose、permute等，就需要调用contiguous()来返回一个contiguous copy；

一种可能的解释是：有些tensor并不是占用一整块内存，而是由不同的数据块组成，而tensor的view()操作依赖于内存是整块的，这时只需要执行contiguous()这个函数，把tensor变成在内存中连续分布的形式；

判断tensor是否为contiguous，可以调用torch.Tensor.is_contiguous()函数:

```python3
import torch 
x = torch.ones(10, 10) 
x.is_contiguous()                                 # True 
x.transpose(0, 1).is_contiguous()                 # False
x.transpose(0, 1).contiguous().is_contiguous()    # True
```

另：在PyTorch的0.4版本中，增加了torch.reshape()，与 numpy.reshape() 的功能类似，大致相当于 tensor.contiguous().view()，这样就省去了对tensor做view()变换前，调用contiguous()的麻烦；

### permute与view函数功能demo

```python3
import torch
import numpy as np

a=np.array([[[1,2,3],[4,5,6]]])
unpermuted=torch.tensor(a)
print(unpermuted.size())              #  ——>  torch.Size([1, 2, 3])

permuted=unpermuted.permute(2,0,1)
print(permuted.size())                #  ——>  torch.Size([3, 1, 2])

view_test = unpermuted.view(1,3,2)
print(view_test.size())               #  ——>  torch.Size([1, 3, 2])
```

利用函数 permute(2,0,1) 可以把 Tensor([[[1,2,3],[4,5,6]]]) 转换成：

```python3
tensor([[[ 1,  4]],
        [[ 2,  5]],
        [[ 3,  6]]])     # print(permuted)    
```

如果使用view(1,3,2) 可以得到：

```python3
tensor([[[ 1,  2],
         [ 3,  4],
         [ 5,  6]]])   # print(view_test)
```

参考文章

<https://zhuanlan.zhihu.com/p/64376950>

[https://pytorch.org/docs/stable/tensors.html?highlight=permute#torch.Tensor.permute](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/tensors.html%3Fhighlight%3Dpermute%23torch.Tensor.permute)

[https://pytorch-cn.readthedocs.io](https://link.zhihu.com/?target=https%3A//pytorch-cn.readthedocs.io/zh/latest/package_references/Tensor/%23permutedims)