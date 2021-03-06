---
layout: post
title: 并发和并行的区别
subtitle: false
tags: [Operating System]
---

<!-- ## 并发和并行的区别 -->

**并发（concurrency）**和**并行（parallellism）**是：

1. 解释一：并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔发生。
2. 解释二：并行是在不同实体上的多个事件，并发是在同一实体上的多个事件。
3. 解释三：并行是在多台处理器上同时处理不同任务。如 hadoop 分布式集群，并发是在一台处理器上“同时”处理多个任务。

**并行”概念是“并发”概念的一个子集**。也就是说，你可以编写一个拥有多个线程或者进程的并发程序，但如果没有多核处理器来执行这个程序，那么就不能以并行方式来运行代码。所以并发编程的目标是充分的利用处理器的每一个核，以达到最高的处理性能。

> 那么，并发是不是一个线程，并行是多个线程？
>
> 答：并发和并行都可以是开很多个线程，就看这些线程能不能同时被（多个）CPU执行，如果可以就说明是并行，而并发是多个线程被（一个）CPU轮流着切换执行。

与可以一起出发的并发（concurrent）相对的是不可以一起出发的顺序（sequential）：

- 顺发：上一个开始执行的任务完成后，当前任务才能开始执行
- 并发：无论上一个开始执行的任务是否完成，当前任务都可以开始执行

与可以一起执行的并行（parallel）相对的是不可以一起执行的串行（serial）：

- 串行：有一个任务执行单元，从物理上就只能一个任务、一个任务地执行
- 并行：有多个任务执行单元，从物理上就可以多个任务一起执行

综上，并发与并行并不是互斥的概念，只是前者关注的是任务的抽象调度、后者关注的是任务的实际执行。而它们又是相关的，比如并行一定会允许并发。

**并行(parallel)**：指在同一时刻，有多条指令在多个处理器上同时执行。所以无论从微观还是从宏观来看，二者都是一起执行的。

![img](https://upload-images.jianshu.io/upload_images/7557373-72912ea8e89c4007.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/313/format/webp)

**并发(concurrency)**：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，CPU采用分时复用的手段， 将时间分成若干段，使多个进程快速交替的执行。

![img](https://upload-images.jianshu.io/upload_images/7557373-da64ffd6d1effaac.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/295/format/webp)

并行在多处理器系统中存在，而并发可以在单处理器和多处理器系统中都存在，并发能够在单处理器系统中存在是因为并发是并行的假象，并行要求程序能够同时执行多个操作，而并发只是要求程序假装同时执行多个操作（每个小时间片执行一个操作，多个操作快速切换执行）。

当有多个线程在操作时，如果系统只有一个 CPU，则它根本不可能真正同时进行一个以上的线程，它只能把 CPU 运行时间划分成若干个时间段，再将时间段分配给各个线程执行，在一个时间段的线程代码运行时,其它线程处于挂起状态.这种方式我们称之为并发（Concurrent）。

当系统有一个以上 CPU 时，则线程的操作有可能非并发。当一个 CPU 执行一个线程时，另一个 CPU 可以执行另一个线程，两个线程互不抢占 CPU 资源，可以同时进行，这种方式我们称之为并行（Parallel）。

![img](https://upload-images.jianshu.io/upload_images/7557373-3fd1b599534cc187.png?imageMogr2/auto-orient/strip|imageView2/2/w/652/format/webp)



