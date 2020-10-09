---
layout: post
title: PyTorch数据处理Pipeline及主机与设备之间的数据传输分析
subtitle: false
tags: [PyTorch]
---

<!-- ## PyTorch数据处理Pipeline及主机与设备之间的数据传输分析 -->

这篇文章的目的是展示如何使用**多线程来并行化数据处理**和从**可分页内存**到**页面锁定内存**的数据传输。在查看[PyTorch的dataloader](https://pytorch.org/docs/stable/data.html)中的[*pin_memory*标志](https://pytorch.org/docs/stable/data.html)

### 主机与GPU互连

GPU通常通过PCIe连接连接到主板，并且必须通过此PCIe链接将来自主（主机）内存的数据传输到GPU内存。该链接的预期带宽是多少？为了回答这个问题，让我们看一些关于PCIe链接的基本信息。![img](https://www.telesens.co/wp-content/uploads/2019/02/img_5c6869028af3c.png)

PCIe链路性能的特征在于传输速率和编码方案。传输速率是指传输的总位数，包括数据位和开销位。编码方案是数据位数与总位数之比。PCIe的许多版本在传输速率和编码方案方面有所不同。有关更多信息，请参阅[PCIe Wikipedia文章](https://en.wikipedia.org/wiki/PCI_Express)。

### 主机内存类型

关于主机内存，有两个主要类别–**可分页（或“未固定”）**和**页面锁定（或“固定”）内存**。**在C程序中使用malloc分配内存时，分配是在可分页内存中完成的。GPU无法直接从可分页的主机内存访问数据，因此，当调用从可分页的主机内存到设备内存的数据传输时，CUDA驱动程序首先分配一个临时固定的主机阵列，将主机数据复制到固定的阵列，然后传输数据从固定阵列到设备内存**，如下所示（有关更多信息，请参见[本页](https://devblogs.nvidia.com/how-optimize-data-transfers-cuda-cc/)）

![img](https://www.telesens.co/wp-content/uploads/2019/02/img_5c686a404c57a.png)

### 从固定和非固定内存传输主机到设备的性能

下图显示了涉及的两种转移类型的图表，其中**h2d（host to device 主机到设备），h2h（host to host 主机到主机）**，下图中主要比较 **h2d transfer_time(no pinning)**, **h2d transfer_time(with pinning, total)** ，即如下：

- **h2d传输（无固定）绿色线：**在这种情况下，数据从可分页内存复制到GPU主机内存。CUDA驱动程序负责将数据复制到页面锁定的内存中
- **h2d传输（带有固定）黑色线：**在这种情况下，数据在pycuda中使用*register_host_memory*函数分配在可分页的内存中，并显式复制到页面锁定的内存中，这部分传输时间（h2h传输时间）为蓝色线。然后，页面锁定内存中的数据被复制到GPU，该部分传输时间为红色线。总时间是两者的总和，如黑色曲线所示

![img](https://www.telesens.co/wp-content/uploads/2019/02/img_5c699d92ad178.png)

不出所料，**两个传输时间（绿色曲线和黑色曲线）几乎相同。这是因为对于从可分页内存进行h2d传输的过程，CUDA驱动程序在内部默认隐式执行从可分页内存到页面锁定内存的传输过程**。

从固定内存到设备的数据传输将比从非固定内存传输的数据具有更高的带宽。将数据从非固定内存传输到固定内存是一项昂贵的操作。因此，很明显，只有在**可以直接将数据分配到页面锁定的内存中**或者**将GPU上的数据处理与数据传输并行化的情况下，因此数据传输延迟会部分或完全隐藏，从页面锁定的内存进行的传输才更有效率**。CUDA提供了诸如cudaMallocHost（）之类的API来实现**第一种方法，即将数据分配到锁页内存中**。

### PyTorch 1.0中的数据加载器

PyTorch的数据加载器采用上述的**第二种方法，即改进数据传输并行化的方式**。**使用多个进程将磁盘中的批处理数据加载到可分页内存中，然后启动一个单独的线程将加载的数据传输到固定的内存中（如果*pin_memory*标志= True）**。如果不是，当用户调用*枚举*或*next（iter）*时，将直接返回由多个进程加载的数据（请参阅PyTorch文档中[imagenet示例的](https://github.com/pytorch/examples/tree/master/imagenet)代码）。下图显示了这两个选项的示意图。

![img](https://www.telesens.co/wp-content/uploads/2019/02/img_5c68799e2175b.png)







### 参考文献

[Pipelining data processing and host-to-device data transfer](https://www.telesens.co/2019/02/16/efficient-data-transfer-from-paged-memory-to-gpu-using-multi-threading/)

