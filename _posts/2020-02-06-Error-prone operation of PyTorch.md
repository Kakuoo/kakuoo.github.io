---
layout: post
title: PyTorch的隐晦操作
subtitle: false
tags: [PyTorch]
---

pytorch中有很多操作比较隐晦，需要仔细研究结合一些例子才能知道如何操作，在此对这些进行总结！

## torch.gather(input, dim, index, out=None) → Tensor

先看官方的介绍：
如果input是一个n维的tensor，size为 (x0,x1…,xi−1,xi,xi+1,…,xn−1)，dim为i，然后index必须也为n维tensor，size为 (x0,x1,…,xi−1,y,xi+1,…,xn−1)，其中y >= 1，最后输出的out与index的size是一样的。
意思就是按照一个指定的轴（维数）收集值
对于一个三维向量来说：

```python
out[i][j][k] = input[index[i][j][k]][j][k]  # if dim == 0
out[i][j][k] = input[i][index[i][j][k]][k]  # if dim == 1
out[i][j][k] = input[i][j][index[i][j][k]]  # if dim == 2
```

参数:
input (Tensor) – 源tensor
dim (int) – 指定的轴数（维数）
index (LongTensor) – 需要聚集起来的数据的索引
out (Tensor, optional) – 目标tensor

## 看完介绍后，稍微思考一下，然后再看一个例子：

scores是一个计算出来的分数，类型为[torch.FloatTensor of size 5x1000]
而y_var是正确分数的索引，类型为[torch.LongTensor of size 5]
容易知道，这里有1000个类别，有5个输入图像，每个图像得出的分数中只有一个是正确的，正确的索引就在y_var中，这里要做的是将正确分数根据索引标号提取出来。

```python
scores = model(X_var)  # 分数
scores = scores.gather(1, y_var.view(-1, 1)).squeeze()  #进行提取
```
1
2
提取后的scores格式也为[torch.FloatTensor of size 5]
这里讲一下变化过程：
1、首先要知道之前的scores的size为[5,1000]，而y_var的size为[5]，scores为2维，y_var为1维不匹配，所以先用view将其展开为[5,1]的size，这样维数n就与scroes匹配了。
2、接下来进行gather，gather函数中第一个参数为1，意思是在第二维进行汇聚，也就是说通过y_var中的五个值来在scroes中第二维的5个1000中进行一一挑选，挑选出来后的size也为[5,1]，然后再通过squeeze将那个一维去掉，最后结果为[5]

再看一个使用相同思想的例子

```python
def gather_example():
    N, C = 4, 5
    s = torch.randn(N, C)
    y = torch.LongTensor([1, 2, 1, 3])
    print(s)
    print(y)
    print(s.gather(1, y.view(-1, 1)).squeeze())
gather_example()
```


结果为：

```
-0.9526  1.7607 -1.0142 -0.6761  0.3022
-0.8421  0.5325  0.4834  0.8441 -0.1592
 0.8786  2.6909  1.3635  0.1197  0.4031
-0.8397  1.4782  0.4514 -0.8381 -2.0638
[torch.FloatTensor of size 4x5]
 1
 2
 1
 3
[torch.LongTensor of size 4]

 1.7607
 0.4834
 2.6909
-0.8381
[torch.FloatTensor of size 4]
```



## 使用普通python函数实现的例子

假设一个numpy数组s的shape为 (N, C)，y是一个shape为(N,)的numpy数组，内容为 0 <= y[i] < C 整数，然后我们使用s[np.arange(N), y] 来进行在s中挑选每一个和y索引对应的数字，其shape同样为(N,)

## torch.max(input, dim, keepdim=False, out=None) -> (Tensor, LongTensor)

max函数需要注意的是，它是一个过载函数，函数参数不同函数的功能和返回值也不同。
当max函数中有维数参数的时候，它的返回值为两个，第一个为最大的值，第二个为最大值的索引或者标识

```python
>>a = torch.randn(4, 4)
>>a

0.0692  0.3142  1.2513 -0.5428
0.9288  0.8552 -0.2073  0.6409
1.0695 -0.0101 -2.4507 -1.2230
0.7426 -0.7666  0.4862 -0.6628
torch.FloatTensor of size 4x4]

>>>torch.max(a, 1)
(
1.2513
0.9288
1.0695
0.7426
[torch.FloatTensor of size 4]
,
2
0
0
0
[torch.LongTensor of size 4]
)
```



## Tensor隐晦操作

使用Tensor型数据进行比较的时候需要注意，如果比较的是其中的值，那么必须将其化为普通值再进行比较，即使是一维的单个数据，也要用[0]操作符来进行读取。
如果想要整个进行比较，建议使用`torch.equal`来进行比较

```python
>>> apple = torch.Tensor([1,2,3])
>>> apple
Out[20]: 
1
2
3
[torch.FloatTensor of size 3]
apple[0]
Out[21]: 1.0
>>> banana = torch.Tensor([1])
>>> banana
Out[23]: 
1
[torch.FloatTensor of size 1]
>>> banana[0]
Out[24]: 1.0
```

