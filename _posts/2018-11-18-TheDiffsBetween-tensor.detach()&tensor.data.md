---
layout: post
title: PyTorch中tensor.detach()和tensor.data的区别
subtitle: 其是否加入计算历史，requires_grad =False,In-place操作的影响
gh-repo: Kakuoo/kakuoo.github.io
gh-badge: [star, fork, follow]
tags: [PyTorch]
comments: true
---
* This will become a table of contents (this text will be scraped).
{:toc}

PyTorch0.4中，`.data `仍保留，但建议使用 `.detach()`，区别在于 `.data `返回和 x 的相同数据 tensor，但不会加入到x的计算历史里，且`requires_grad = False`，这样有些时候是不安全的，因为 x.data 不能被 autograd 追踪求微分 。 `.detach() `返回相同数据的 tensor ，且 `requires_grad=False `，但能通过` in-place `操作报告给 autograd 在进行反向传播的时候。
举例：

## tensor.detach()

当我们再训练网络的时候可能希望保持一部分的网络参数不变，只对其中一部分的参数进行调整；或者只训练部分分支网络，并不让其梯度对主网络的梯度造成影响，这时候我们就需要使用detach()函数来切断一些分支的反向传播

返回一个新的`Variable`，从当前计算图中分离下来的，但是仍指向原变量的存放位置,不同之处只是requires_grad为false，得到的这个`Variable`永远不需要计算其梯度，不具有grad。**即使之后重新将它的requires_grad置为true,它也不会具有梯度grad**

这样我们就会继续使用这个新的`Variable进行计算，后面当我们进行`反向传播时，到该调用detach()的`Variable`就会停止，不能再继续向前进行传播

```python
# 源码
def detach(self):
        """Returns a new Variable, detached from the current graph.
        Result will never require gradient. If the input is volatile, the output
        will be volatile too.
        .. note::
          Returned Variable uses the same data tensor, as the original one, and
          in-place modifications on either of them will be seen, and may trigger
          errors in correctness checks.
        """
        result = NoGrad()(self)  # this is needed, because it merges version counters
        result._grad_fn = None
　　　　 return result
```

如果输入 `volatile=True(即不需要保存记录，当只需要结果而不需要更新参数时这么设置来加快运算速度)`，那么返回的`Variable` `volatile=True`。（`volatile`已经弃用）

**注意：**

返回的`Variable`和原始的`Variable`公用同一个`data tensor`。`in-place函数`修改会在两个`Variable`上同时体现(因为它们共享`data tensor`)，当要对其调用backward()时可能会导致错误。

```python
>>> a = torch.tensor([1,2,3.], requires_grad =True)
>>> out = a.sigmoid()
>>> c = out.detach()
c>>> .zero_()
tensor([ 0., 0., 0.])

>>> out                   #  out的值被c.zero_()修改 !!
tensor([ 0., 0., 0.])

>>> out.sum().backward()  #  需要原来out得值，但是已经被c.zero_()覆盖了，结果报错
RuntimeError: one of the variables needed for gradient
computation has been modified by an
```

正常的例子是：

```python
a = torch.tensor([1, 2, 3.], requires_grad=True)
print(a.grad)
out = a.sigmoid()

out.sum().backward()
print(a.grad)
```

返回：

```python
None
tensor([0.1966, 0.1050, 0.0452])
```

当使用detach()但是没有进行更改时，并不会影响backward():

```python
a = torch.tensor([1, 2, 3.], requires_grad=True)
print(a.grad)
out = a.sigmoid()
print(out)

#添加detach(),c的requires_grad为False
c = out.detach()
print(c)

#这时候没有对c进行更改，所以并不会影响backward()
out.sum().backward()
print(a.grad)
```

返回：

```python
None
tensor([0.7311, 0.8808, 0.9526], grad_fn=<SigmoidBackward>)
tensor([0.7311, 0.8808, 0.9526])
tensor([0.1966, 0.1050, 0.0452])
# 可见c,out之间的区别是c是没有梯度的，out是有梯度的
```

如果这里使用的是c进行sum()操作并进行backward()，则会报错：

```python
a = torch.tensor([1, 2, 3.], requires_grad=True)
print(a.grad)
out = a.sigmoid()
print(out)

#添加detach(),c的requires_grad为False
c = out.detach()
print(c)

#使用新生成的Variable进行反向传播
c.sum().backward()
print(a.grad)
```

返回：

```python
None
tensor([0.7311, 0.8808, 0.9526], grad_fn=<SigmoidBackward>)
tensor([0.7311, 0.8808, 0.9526])
Traceback (most recent call last):
  File "test.py", line 13, in <module>
    c.sum().backward()
  File "/anaconda3/envs/deeplearning/lib/python3.6/site-packages/torch/tensor.py", line 102, in backward
    torch.autograd.backward(self, gradient, retain_graph, create_graph)
  File "/anaconda3/envs/deeplearning/lib/python3.6/site-packages/torch/autograd/__init__.py", line 90, in backward
    allow_unreachable=True)  # allow_unreachable flag
RuntimeError: element 0 of tensors does not require grad and does not have a grad_fn
```

如果此时对c进行了更改，这个更改会被`autograd`追踪，在对out.sum()进行backward()时也会报错，因为此时的值进行backward()得到的梯度是错误的：

```python
a = torch.tensor([1, 2, 3.], requires_grad=True)
print(a.grad)
out = a.sigmoid()
print(out)

#添加detach(),c的requires_grad为False
c = out.detach()
print(c)
c.zero_() #使用in place函数对其进行修改

#会发现c的修改同时会影响out的值
print(c)
print(out)

#这时候对c进行更改，所以会影响backward()，这时候就不能进行backward()，会报错
out.sum().backward()
print(a.grad)
```

返回：

```python
None
tensor([0.7311, 0.8808, 0.9526], grad_fn=<SigmoidBackward>)
tensor([0.7311, 0.8808, 0.9526])
tensor([0., 0., 0.])
tensor([0., 0., 0.], grad_fn=<SigmoidBackward>)
Traceback (most recent call last):
  File "test.py", line 16, in <module>
    out.sum().backward()
  File "/anaconda3/envs/deeplearning/lib/python3.6/site-packages/torch/tensor.py", line 102, in backward
    torch.autograd.backward(self, gradient, retain_graph, create_graph)
  File "/anaconda3/envs/deeplearning/lib/python3.6/site-packages/torch/autograd/__init__.py", line 90, in backward
    allow_unreachable=True)  # allow_unreachable flag
RuntimeError: one of the variables needed for gradient computation has been modified by an inplace operation
```



## tensor.data

如果上面的操作使用的是.data，效果会不同：

这里的不同在于.data的修改不会被autograd追踪，这样当进行backward()时它不会报错，回得到一个错误的backward值

```python
a = torch.tensor([1, 2, 3.], requires_grad=True)
print(a.grad)
out = a.sigmoid()
print(out)

c = out.data
print(c)
c.zero_() #使用in place函数对其进行修改

#会发现c的修改同时也会影响out的值
print(c)
print(out)

#这里的不同在于.data的修改不会被autograd追踪，这样当进行backward()时它不会报错，会得到一个错误的backward值
out.sum().backward()
print(a.grad)
```

返回：

```python
None
tensor([0.7311, 0.8808, 0.9526], grad_fn=<SigmoidBackward>)
tensor([0.7311, 0.8808, 0.9526])
tensor([0., 0., 0.])
tensor([0., 0., 0.], grad_fn=<SigmoidBackward>)
tensor([0., 0., 0.])
```

上面的内容实现的原理是：**In-place 正确性检查**

所有的`Variable`都会记录用在他们身上的 `in-place operations`。如果`pytorch`检测到`variable`在一个`Function`中已经被保存用来`backward`，但是之后它又被`in-place operations`修改。当这种情况发生时，在`backward`的时候，`pytorch`就会报错。**这种机制保证了，如果你用了`in-place operations`，但是在`backward`过程中没有报错，那么梯度的计算就是正确的。**

**下面结果正确是因为改变的是sum()的结果，中间值a.sigmoid()并没有被影响，所以其对求梯度并没有影响**：

```python
a = torch.tensor([1, 2, 3.], requires_grad=True)
print(a.grad)
out = a.sigmoid().sum() #但是如果sum写在这里，而不是写在backward()前，得到的结果是正确的
print(out)


c = out.data
print(c)
c.zero_() #使用in place函数对其进行修改

#会发现c的修改同时也会影响out的值
print(c)
print(out)

#sum没有写在这里
out.backward()
print(a.grad)
```

返回：

```python
None
tensor(2.5644, grad_fn=<SumBackward0>)
tensor(2.5644)
tensor(0.)
tensor(0., grad_fn=<SumBackward0>)
tensor([0.1966, 0.1050, 0.0452])
```







```python
>>> a = torch.tensor([1,2,3.], requires_grad =True)
>>> out = a.sigmoid()
>>> c = out.data
>>> c.zero_()
tensor([ 0., 0., 0.])

>>> out                   #  out的数值被c.zero_()修改
tensor([ 0., 0., 0.])

>>> out.sum().backward()  #  反向传播
>>> a.grad                #  这个结果很严重的错误，因为out已经改变了
tensor([ 0., 0., 0.])
```



## tensor.detach_()

将一个`Variable`从创建它的图中分离，并把它设置成叶子`variable`

其实就相当于变量之间的关系本来是x ---> m ---> y,这里的叶子variable是x，但是这个时候对m进行了.detach_()操作,其实就是进行了两个操作：

- 将m的grad_fn的值设置为None,这样m就不会再与前一个节点x关联，这里的关系就会变成x, m -> y,此时的m就变成了叶子结点
- 然后会将m的requires_grad设置为False，这样对y进行backward()时就不会求m的梯度

这么一看其实detach()和detach\_()很像，两个的区别就是detach_()是对本身的更改，detach()则是生成了一个新的variable

比如x -> m -> y中如果对m进行detach()，后面如果反悔想还是对原来的计算图进行操作还是可以的

但是如果是进行了detach_()，那么原来的计算图也发生了变化，就不能反悔了


## Pytorch in-place 操作

in-place operation在pytorch中是指改变一个tensor的值的时候，不经过复制操作，而是直接在原来的内存上改变它的值。可以把它成为原地操作符。

在pytorch中经常加后缀“\_”来代表原地in-place operation，比如说.add\_() 或者.scatter()。python里面的+=，\*=也是in-place operation。

下面是正常的加操作,执行结束加操作之后x的值没有发生变化：
```python
import torch
x=torch.rand(2) #tensor([0.8284, 0.5539])
print(x)
y=torch.rand(2)
print(x+y)      #tensor([1.0250, 0.7891])
print(x)        #tensor([0.8284, 0.5539])
1
2
3
4
5
```

下面是原地操作，执行之后改变了原来变量的值：

```python
import torch
x=torch.rand(2) #tensor([0.8284, 0.5539])
print(x)
y=torch.rand(2)
x.add_(y)
print(x)        #tensor([1.1610, 1.3789])
1
2
3
4
5
```
在官方文档中有这一段话：

> 如果你使用了`in-place operation`而没有报错的话，那么你可以确定你的梯度计算是正确的。

结论：只有那些在反向传播过程中不需要使用到的tensor才能做in-place操作，否则不能。

### Variable上的In-place操作
在自动求导中支持in-place操作是一件很困难的事情，我们在大多数情况下都不鼓励使用它们。Autograd的缓冲区释放和重用非常高效，并且很少场合下in-place操作能实际上明显降低内存的使用量。除非您在内存压力很大的情况下，否则您可能永远不需要使用它们。

限制in-place操作适用性主要有两个原因：

１．覆盖梯度计算所需的值。这就是为什么变量不支持log_。它的梯度公式需要原始输入，而虽然通过计算反向操作可以重新创建它，但在数值上是不稳定的，并且需要额外的工作，这往往会与使用这些功能的目的相悖。

２．每个in-place操作实际上需要实现重写计算图。不合适的版本只需分配新对象并保留对旧图的引用，而in-place操作则需要将所有输入的creator更改为表示此操作的Function。这就比较棘手，特别是如果有许多变量引用相同的存储（例如通过索引或转置创建的），并且如果被修改输入的存储被任何其他Variable引用，则in-place函数实际上会抛出错误。

### In-place正确性检查
每个变量保留有version counter，它每次都会递增，当在任何操作中被使用时。当Function保存任何用于后向的tensor时，还会保存其包含变量的version counter。一旦访问self.saved_tensors，它将被检查，如果它大于保存的值，则会引起错误。
