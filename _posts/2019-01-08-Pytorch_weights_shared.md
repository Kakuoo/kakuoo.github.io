---
layout: post
title: PyTorch参数共享
subtitle: false
tags: [PyTorch]
---

<!-- ## Pytorch参数共享 -->
在很多神经网络中，往往会出现多个层共享一个权重的情况，pytorch可以快速地处理权重共享问题。

### 例子1：

```python
class ConvNet(nn.Module):
    def __init__(self):
        super(ConvNet, self).__init__()
        self.conv_weight = nn.Parameter(torch.randn(3, 3, 5, 5))

    def forward(self, x):
        x = nn.functional.conv2d(x, self.conv_weight, bias=None, stride=1, padding=2, dilation=1, groups=1)
        x = nn.functional.conv2d(x, self.conv_weight.transpose(2, 3).contiguous(), bias=None, stride=1, padding=0, dilation=1, groups=1)
    return x

```

上边这段程序定义了两个卷积层，这两个卷积层共享一个权重conv_weight，第一个卷积层的权重是conv_weight本身，第二个卷积层是conv_weight的**(2, 3)位置上的转置**（即[BS, C, H, W]中的[H, W]的转置）。**注意在gpu上运行时，transpose()后边必须加上.contiguous()，可使被用于转置操作的内存块连续化，否则会报错。**

```python
# 转置说明
a = torch.tensor([[0, 1],[2, 3]])
tensor([[0, 1],
        [2, 3]])

b = a.transpose(0,1)
tensor([[0, 2],
        [1, 3]])
```

### 注意：

$$Var(w_i) = \frac{1}{n}$$

$$W^{[l]} = np.random.randn(shape) * np.sqrt(\frac{1}{n^{[l-1]}})$$

> 根据Ng 吴恩达的课程，为了降低梯度消失和梯度爆炸问题，为了预防权重过大或过小，权重初始化时应遵循上述公式，其中$n^{[l-1]}$为上一次神经元的个数，如果使用的是ReLU激活函数，可以将$\frac{1}{n^{[l-1]}}$更改成$\frac{2}{n^{[l-1]}}$，效果会更好，据此，建议将参数初始化的代码更改为：

```python
self.conv_weight = nn.Parameter(torch.randn(3, 3, 5, 5) * 																			torch.tensor(2/num_features).sqrt_())
```

### 例子2：

```python
class LinearNet(nn.Module):
    def __init__(self):
        super(LinearNet, self).__init__()
        self.linear_weight = nn.Parameter(torch.randn(3, 3))

    def forward(self, x):
        x = nn.functional.linear(x, self.linear_weight)
        x = nn.functional.linear(x, self.linear_weight.t())

return x
```

这个网络实现了一个双层感知器，权重同样是一个parameter的本身及其转置。

### 例子3：

```python
class LinearNet2(nn.Module):
    def __init__(self):
        super(LinearNet2, self).__init__()
        self.w = nn.Parameter(torch.FloatTensor([[1.1,0,0], [0,1,0], [0,0,1]]))

    def forward(self, x):
        x = x.mm(self.w)
        x = x.mm(self.w.t())
    return x
```

这个方法直接用mm函数将x与w相乘，与上边的网络效果相同。

### 例子4：

与`embed_tokens.weight`使用同样的参数

```python
if self.share_input_output_embed:
    x = F.linear(x, self.embed_tokens.weight)
elif not self.share_input_output_embed:
    self.embed_out = nn.Parameter(torch.Tensor(len(dictionary), output_embed_dim))
    nn.init.normal_(self.embed_out, mean=0, std=output_embed_dim ** -0.5)

if self.share_input_output_embed:
    x = F.linear(x, self.embed_tokens.weight)
else:
    x = F.linear(x, self.embed_out)
```

初始化的时候还可以让`padding`的部分为`0`

```python
def Embedding(num_embeddings, embedding_dim, padding_idx):
    m = nn.Embedding(num_embeddings, embedding_dim, padding_idx=padding_idx)
    nn.init.normal_(m.weight, mean=0, std=embedding_dim ** -0.5)
    nn.init.constant_(m.weight[padding_idx], 0)
    return m
```
