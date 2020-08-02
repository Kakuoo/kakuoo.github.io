---
layout: post
title: PyTorch 关于代码结构的理解
subtitle: false
tags: [PyTorch]
---

<!-- ## Pytorch 关于代码结构的理解 -->

- 一直对于model.eval()和torch.no_grad()有些疑惑
- 之前看博客说，只用torch.no_grad()即可
- 但是今天查资料，发现不是这样，而是两者都用，因为两者有着不同的作用
- 引用[stackoverflow](https://stackoverflow.com/questions/55627780/evaluating-pytorch-models-with-torch-no-grad-vs-model-eval):

Use both. They do different things, and have different scopes.
**with torch.no_grad**： disables tracking of gradients in autograd.
**model.eval()**： changes the forward() behaviour of the module it is called upon. eg, it disables dropout and has batch norm use the entire population statistics

model.eval()和with torch.no_grad():



主要是以下这几个问题一直没搞清楚：

> 1.分不清楚model.train(),model.eval()在哪一步用？
>
> 2.optimizer.zero_grad()和model.zero_grad()区别是啥？
>
> 3.用了with torch.no_grad()还用model.eval()吗？

第一个问题：有些人的做法是每个epoch只验证一次，而此处按照迭代次数进行验证：

```python
for epoch in range(1, args.epochs+1):

    n_correct, n_total = 0, 0
    losses = []
    model.train()
    for batch_idx, batch in enumerate(train_loader.next_batch()):
        # 看这里，有人喜欢把model.train放这里，也没问题
        # model.train()
        ....
        # 看这里，就是这个model.eval()导致测试失效，为什么呢？看第三个问题
        model.eval()
        if iterations % args.dev_every == 0:
            with torch.no_grad():
```

两条语句有固定的使用场景。

**在训练模型时会在前面加上：**

```python
model.train()
```

**在测试模型时在前面使用:**

```python
model.eval()
```

同时发现，如果不使用这两条语句，程序也可以运行。这两个方法是针对在网络train和eval时采用不同方式的情况，比如Batch Normalization和Dropout。

下面对Batch Normalization和Dropout做一下详细的解析：

**Batch Normalization**

BN的作用主要是对网络中间的每层进行归一化处理，并且使用变换重构（Batch Normalization Transform）保证每层所提取的特征分布不会被破坏。
训练时是针对每个mini-batch的，但是在测试中往往是针对单张图片，即不存在mini-batch的概念。由于网络训练完毕后参数都是固定的，因此每个batch的均值和方差都是不变的，因此直接结算所有batch的均值和方差。**所有Batch Normalization的训练和测试时的操作不同。**

**Dropout**

Dropout能够克服Overfitting，在每个训练Batch中，通过忽略一半的特征检测器，可以明显的减少过拟合现象。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190610231238411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjAxODExMg==,size_16,color_FFFFFF,t_70)
在训练中，每个隐层的神经元先乘以概率P，然后再进行激活。
在测试中，所有的神经元先进行激活，然后每个隐层神经元的输出乘P

### 第二个问题：搬运网上的解释

if`optimizer = optim.Optimizer(net.parameters())`,they are the same.

there might be use cases where you would like to use different optimizers for different parts of your model. In such a case,`model.zero_grad()`would clear all parameters of the model, while the`optimizer.zero_grad()`call will just clean the gradients of the parameters that were passed to it

### 第三个问题：[stackoverflow的解释]([https://stackoverflow.com/questions/55627780/evaluating-pytorch-models-with-torch-no-grad-vs-model-eval](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/55627780/evaluating-pytorch-models-with-torch-no-grad-vs-model-eval))

They do different things, and have different scopes.

- `with torch.no_grad` - disables tracking of gradients in `autograd`.

- `model.eval()` changes the `forward()` behaviour of the module it is called upon

- - eg, it disables dropout and has batch norm use the entire population statistics

### `with torch.no_grad`

The `torch.autograd.no_grad`[documentation](https://pytorch.org/docs/stable/autograd.html#torch.autograd.no_grad) says:

> Context-manager that disabled [sic] gradient calculation.
> Disabling gradient calculation is useful for inference, when you are sure that you will not call `Tensor.backward()`. It will reduce memory consumption for computations that would otherwise have `requires_grad=True`. In this mode, the result of every computation will have `requires_grad=False`, even when the inputs have `requires_grad=True`.

### `model.eval()`

The `nn.Module.eval` [documentation](https://pytorch.org/docs/stable/nn.html#torch.nn.Module.eval) says:

> Sets the module in evaluation mode.
> This has any effect only on certain modules. See documentations of particular modules for details of their behaviors in training/evaluation mode, if they are affected, e.g. `Dropout`, `BatchNorm`, etc



[参考文章](https://zhuanlan.zhihu.com/p/64411611)

如果按照第一段代码所示，那么我有一些疑惑：

好像因为code只在最外面train()了一下，所以一旦eval了一次后后面迭代的模型都不在train mode下了。



