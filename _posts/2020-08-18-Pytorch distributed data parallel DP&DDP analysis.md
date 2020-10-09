---
layout: post
title: Pytorch分布式数据并行训练
subtitle: false
tags: [PyTorch]
---

<!-- ## Pytorch分布式数据并行训练 -->

本文将解释在Pytorch 1.0中实现的数据并行训练（DP）与分布式数据并行训练（DDP）之间的区别，并使用NVIDIA的Visual Profiler（nvvp）可视化所涉及的计算和数据传输操作。

回想一下，Minibatch随机梯度下降（MB-SGD，或简称为SGD），在DNN训练期间使用的算法会在计算参数更新之前对整个批次的梯度进行平均。当增加批量大小时，将对大量元素进行平均。在有限的情况下，如果批次大小=训练数据集的大小，则SGD变为简单的梯度下降，因为每个批次都由相同的元素组成。随着批次大小的增加，我们必须确保保持训练的准确性和泛化性能。[这个](https://arxiv.org/pdf/1706.02677.pdf) 来自Facebook AI的精彩论文展示了如何调整学习率和其他超参数，以保持准确性和泛化性能。

### 并行数据加载

流行的深度学习框架（例如Pytorch和Tensorflow）为分布式培训提供内置支持。但是，要有效地使用这些功能，需要从磁盘读取输入数据开始，仔细研究并全面了解培训中涉及的每个步骤。从广义上讲，涉及四个步骤：

1. 将数据从磁盘加载到主机
2. 将数据从可分页内存传输到主机上的固定内存。请参阅[此](https://www.telesens.co/2019/02/16/efficient-data-transfer-from-paged-memory-to-gpu-using-multi-threading/)有关分页和固定的内存更多信息。
3. 将数据从固定内存传输到GPU
4. 在GPU上向前和向后传递

这些步骤中的每一个都必须尽可能并行化，并且每当不存在数据依赖性时，下一个批次的步骤都应与当前批次一起进行Pipeline流水线处理。

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca4eff975d80.png)

幸运的是，深度学习库为所有这些步骤提供了支持。`PyTorch`中的`Dataloader`（我将在本文中重点介绍的框架）提供使用多个进程（通过将`num_workers`> 0设置）从磁盘加载数据以及将多页数据从可分页内存到固定内存的能力（通过设置） `pin_memory = True`）。

一般的，对于大批量的数据，若仅有一个线程用于加载数据，则数据加载时间占主导地位，这意味着无论我们如何加快数据处理速度，性能都会受到数据加载时间的限制。现在，设置`num_workers = 4`以及`pin_memory = True`。这样，可以使用多个进程从磁盘读取不重叠的数据，并启动生产者-消费者线程以将这些进程读取的数据从可分页的内存转移到固定的内存。



![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca5012ba2dca.png)

现在，多个进程能够更快地加载数据，并且当数据处理时间足够长时，流水线数据加载几乎可以完全隐藏数据加载延迟。这是因为在处理当前批次的同时，将从磁盘读取下一个批次的数据，并将其传输到固定内存。如果处理当前批次的时间足够长，则下一个批次的数据将立即可用。**这个想法还建议如何为num_workers参数设置适当的值。应该设置此参数，以使从磁盘读取批处理数据的速度比GPU处理当前批处理的速度更快（但不能更高，因为这只会浪费多个进程使用的系统资源）。**

**请注意，到目前为止，我们仅解决了从磁盘加载数据以及从可分页到固定内存的数据传输问题。**从固定内存到GPU的数据传输（`tensor.cuda()`）也可以使用CUDA流进行流水线处理。PyTorch的默认Imagenet示例不执行此操作，但是NVIDIA的顶点库显示了如何执行此操作的示例。我将在稍后详细讨论。

### 数据并行（DP）和分布式数据并行（DDP）

现在将使用GPU网络检查数据并行处理。基本思想是，网络中的每个GPU使用模型的本地副本对一批数据进行正向和反向传播。反向传播期间计算出的梯度将发送到服务器，该服务器运行reduce归约操作以计算平均梯度。然后将平均梯度结果发送回GPU，GPU使用SGD更新模型参数。使用数据并行性和有效的网络通信软件库（例如NCCL），可以实现使训练时间几乎线性减少。网络上有很多不错的教程，它们详细解释了数据并行性如何工作。例如去年[大佬写的一个帖子](https://www.telesens.co/2017/12/25/understanding-data-parallelism-in-machine-learning/) ，它完全使用Python在模拟数据上实现了数据并行性，而没有使用任何深度学习框架。

Pytorch 1.0具有两个支持数据并行性的系统。第一个在[nn.parallel.data_parallel](https://pytorch.org/docs/stable/_modules/torch/nn/parallel/data_parallel.html)中实现简称为数据并行。如下图所示，该系统通过将整个小型批处理加载到主线程上，然后将子小型批处理分散到整个GPU网络中来工作。每个GPU在单独的线程上的子微型批处理上运行正向传递。然后在主GPU上收集网络输出，并通过将网络输出与批次中每个元素的真实数据标签进行比较来计算损失函数值。接下来，损失值分散在各个GPU上，每个GPU进行反向传播以计算梯度。最后，在主GPU上进行梯度下降，并更新主GPU上的模型参数。这样就完成了一次迭代。请注意，由于模型参数仅在主GPU上更新，而其他从属GPU此时并不是同步更新的，所以需要在下一次迭代前将主GPU上的模型参数broadcast扩散到其他从属GPU上。

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca50bcd68a1e.png)

可以在nn.parallel.data_parallel.py中的转发功能中遵循`步骤2-5`。在Python代码中设置断点并检查数据如何前向传播，如何流经每个步骤是很有启发性的。

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca50d9e2812e.png)

不幸的是，反向传播的实现隐藏在C代码中，因此无法单步执行Python中的代码。下表列出了这种设计的许多低效率之处：

- 冗余数据副本
  - 数据从主机复制到主GPU，然后将子微型批分散在其他GPU上
- 在前向传播之前跨GPU进行模型复制
  - 由于模型参数是在主GPU上更新的，因此模型必须在每次正向传递的开始时重新同步
- 每批的线程创建/销毁开销
  - 并行转发是在多个线程中实现的（这可能只是PyTorch问题）
- 梯度减少流水线机会未开发
  - 在Pytorch 1.0数据并行实现中，梯度下降发生在反向传播的末尾。我将在分布式数据并行部分中对此进行详细讨论。
- 在主GPU上不必要地收集模型输出output
- GPU利用率不均
  - 在主GPU上执行损失loss计算
  - 梯度下降，在主GPU上更新参数

#### 使用CUDA流管道传输主机到GPU的数据

上面我提到过，我们还没有利用机会将固定的内存到GPU的数据进行管道传输。这可以使用CUDA流来完成。如果你不熟悉CUDA流，  [这里是](https://devblogs.nvidia.com/how-overlap-data-transfers-cuda-cc/)来自NVIDIA的优秀教程。

[NVIDIA APEX](https://github.com/NVIDIA/apex)的DataLoader引入data_prefetcher类从Pytorch的DataLoader取数据，并使用CUDA流以管线数据传送到GPU。现在，在GPU上执行到浮点和图像标准化的转换，这比在CPU上快得多，并且节省了大量的数据加载带宽。

NVIDIA的`apex`库还引入了许多其他优化，例如**混合精度训练**和**动态损失缩放**，我在这些实验中没有进行研究。我采用了与主机到设备流水线相关的代码并输入规范化，并将其添加到Pytorch Imagenet示例中。可以在[此处](https://drive.google.com/open?id=1zHBoGMwAII0COXEOMm9TlqlbhDgJWram)下载代码。

### 分布式数据并行

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca570946ee1c.png)

分布式并行数据消除了上面提到的并行数据的所有低效率。不再有主GPU，每个GPU执行相同的任务。与我们之前看到的数据并行的多线程体系结构相比，对每个GPU的培训都是在自己的过程中进行的。每个进程都从磁盘加载其自己的数据。分布式数据采样器可确保加载的数据在各个进程之间不重叠。稍后我们将研究其工作原理。损失函数的前向传播和计算在每个GPU上独立执行。因此，不需要收集网络输出。在反向传播期间，梯度下降在所有GPU上均被执行，从而确保每个GPU在反向传播结束时最终得到平均梯度的相同副本。

除了其更简单的数据流外，分布式数据并行还利用了另一个流水线机会，即通过梯度计算进行梯度All-Reduce。如下所示，某一层参数梯度的计算不依赖于上一层的梯度。因此，可以通过梯度All-Reduce对流水线中的梯度计算进行流水线处理。

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca573df4957d.png)

该实验在进行时没有在顶点库中进行优化，因此图像数据仍作为浮点值加载。要注意的重要一点是，梯度全约操作与反向传递整齐地流水线化。由于必须先计算梯度，然后才能减小梯度，因此减小必须比向后传递落后一步。此外，为了更有效地利用网络资源，在执行All-Reduce之前，将梯度分组到存储桶中。Pytorch 1.0中的默认存储桶大小为25 MB。如下所示，最终权重更新必须等待最后的梯度All-Reduce操作完成。

分布式数据并行通常用于多主机设置中，其中每个主机具有多个GPU，并且主机通过网络连接。默认情况下，每个GPU上都会运行一个进程。根据Pytorch的文档，此配置是使用分布式数据并行的最有效方法。另一种可能的配置是在每个主机上运行一个进程，该进程控制该系统上的所有GPU。在这种配置下，每个进程都在其控制的GPU上并行运行数据并行（我们考虑的第一个系统）。

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca61ddf35f63.png)

在多主机设置中，并行在每个主机上启动分布式数据并行应用程序。因此，需要某种机制，以使在不同主机上运行的多个进程同步。这是[init_process_group](https://pytorch.org/docs/stable/distributed.html#)函数的工作，该函数使用共享文件系统或TCP IP地址/端口来同步进程。

另一个重要要求是每个进程都应加载数据的非重叠副本。Pytorch提供了一个[DistributedSampler](https://pytorch.org/docs/stable/data.html?highlight=distributed%20sampler#torch.utils.data.distributed.DistributedSampler)来确保这一点。让我们考虑一个多主机设置，其中一个进程控制一个GPU。可以说有2个主机，每个主机具有3个GPU。每个过程都可以通过共享文件系统访问培训数据，也可以维护自己的数据本地副本。要读取数据的非重叠副本，每个进程必须知道进程组中的进程数及其在组中的自己的等级。有了此信息，每个进程可以将其等级用作偏移量，并将进程数用作跨步以读取不重叠的块。该信息通过参数world_size和rank提供。

这里world_size是指分布式系统中的主机数（在我们的示例中为2），等级是每个主机的等级。根据此信息，进程总数将计算为![\ times](https://www.telesens.co/wp-content/ql-cache/quicklatex.com-3e2a3b7b9d8913e71519bf7df9eb51b3_l3.svg)每主机GPU的world_size个数。该数字（在我们的示例中为6）然后变为新的world_size。然后，将每个进程的全局等级（在进程组中唯一）计算为本地等级（GPU id）+每个主机![\ times](https://www.telesens.co/wp-content/ql-cache/quicklatex.com-3e2a3b7b9d8913e71519bf7df9eb51b3_l3.svg)host_rank的GPU数量。全局等级和世界大小可以用作每个过程的偏移量和跨度，以加载不重叠的批处理数据。我尚未对此进行验证，但是我相信此设置要求每台主机上的GPU数量必须相同。

![img](https://www.telesens.co/wp-content/uploads/2019/04/img_5ca62edc76524.png)





### 参考文献

[Distributed data parallel training using Pytorch on AWS](https://www.telesens.co/2019/04/04/distributed-data-parallel-training-using-pytorch-on-aws/)

