---
layout: post
title: PyTorch之contiguous函数
subtitle: false
tags: [PyTorch]
---

<!-- ## Pytorch之contiguous函数 -->

### 官方中英文doc

```python
# contiguous() → Tensor
torch.Tensor.contiguous (Python method, in torch.Tensor)
torch.Tensor.is_contiguous (Python method, in torch.Tensor)

# 返回一个内存连续的有相同数据的tensor，如果原tensor内存连续，则返回原tensor
# Returns a contiguous tensor containing the same data as self tensor. If self tensor is contiguous, this function returns the self tensor.
```

### pytorch contiguous()的使用

contiguous一般与transpose，permute，view搭配使用：使用transpose或permute进行维度变换后，调用contiguous，然后方可使用view对维度进行变形（如：tensor_var.contiguous().view() ），示例如下：

```python
x = torch.Tensor(2,3)
y = x.permute(1,0)         # permute：二维tensor的维度变换，此处功能相当于转置transpose
y.view(-1)                 # 报错，view使用前需调用contiguous()函数
y = x.permute(1,0).contiguous()
y.view(-1)                 # OK
```

具体原因有两种说法：

1. transpose、permute等维度变换操作后，tensor在内存中不再是连续存储的，而view操作要求tensor的内存连续存储，所以需要contiguous来返回一个contiguous copy；
2. 维度变换后的变量是之前变量的浅拷贝，指向同一区域，即view操作会连带原来的变量一同变形，这是不合法的，所以也会报错；---- 这个解释有部分道理，也即contiguous返回了tensor的深拷贝contiguous copy数据；

### contiguous函数分析，参考CSDN博客

在pytorch中，只有很少几个操作是不改变tensor的内容本身，而**只是重新定义下标与元素的对应关系**。换句话说，这种操作**不进行数据拷贝和数据的改变，变的是元数据**，这些操作是：

```python
narrow(), view(), expand(), transpose()
```

举个栗子，在使用transpose()进行转置操作时，**pytorch并不会创建新的、转置后的tensor**，而是**修改了tensor中的一些属性（也就是元数据），使得此时的offset和stride是与转置tensor相对应的**，而**转置的tensor和原tensor的内存是共享的**！

为了证明这一点，我们来看下面的代码：

```python
x = torch.randn(3, 2)
y = x.transpose(x, 0, 1)
x[0, 0] = 233
print(y[0, 0])       # print 233
```

可以看到，**改变了x的元素的值的同时，y的元素的值也发生了变化；**也即，经过上述操作后得到的tensor，它**内部数据的布局方式和从头开始创建一个常规的tensor的布局方式是不一样的**！于是就有contiguous()的用武之地了。

在上面的例子中，**x是contiguous的，但y不是（因为内部数据不是通常的布局方式）**。注意：不要被contiguous的字面意思“连续的”误解，**tensor中数据还是在内存中一块区域里，只是布局的问题**！

**当调用contiguous()时，会强制拷贝一份tensor，让它的布局和从头创建的一模一样；**

一般来说这一点不用太担心，如果你没在需要调用contiguous()的地方调用contiguous()，运行时会提示你：

```python
RuntimeError: input is not contiguous
```

只要看到这个错误提示，加上contiguous()就好啦～

### is_contiguous()函数

**is_contiguous() → bool**

```text
Returns True if self tensor is contiguous in memory in C order.
如果该tensor在内存中是连续的则返回True；
```

pytorch里面的 **contiguous() 是以 C 为顺序保存在内存里面**，如果不是，则返回一个以 C 为顺序保存的tensor：

```python
tensor_var.is_contiguous()           # 可以用来判断tensor是否以 C 为顺序保存的
```

一些可能导致不是以 C 为顺序保存的可能为：

```python
import torch
x = torch.ones(10, 10)
x.is_contiguous()						      # True
x.transpose(0, 1).is_contiguous()               # False，transpose会改变tensor变量内存的布局方式
x.transpose(0, 1).contiguous().is_contiguous()	# True
```

**4.2 view()、reshape()函数的差异**

在pytorch 0.4中，增加了`torch.reshape()`，与 `numpy.reshape()` 的功能类似，大致相当于 `tensor.contiguous().view()`，这样就省去了对tensor做`view()`变换前，调用`contiguous()`的麻烦；
