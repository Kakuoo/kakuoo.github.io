---
layout: post
title: Python tqdm 库使用
subtitle: false
tags: [Python]
---

## Python tqdm 库使用

使用python tqdm进度条库让你的python进度可视化

`Tqdm`在阿拉伯语表示“progress”进步，在西班牙语中则是“I love you so much”的缩写，表示`我非常爱你`。是一个快速，可扩展的`Python`进度条，可以在`Python`长循环中添加一个进度提示信息，用户只需要封装任意的迭代器`tqdm(iterator)`即可完成进度条。相比`ProgressBar`来说`Tqdm`的开销非常低，同时`Tqdm`可以在任何环境中不需要任何依赖运行。

### Tqdm安装

tqdm并不是python的系统包，因此可以使用`pip`安装稳定版本

```shell
pip install tqdm
```

也可以使用`conda`来安装最新的版本

```shell
conda install -c conda-forge tqdm
```

你可以使用如下代码测试是否安装成功：

```python
from tqdm import tqdm
for i in tqdm(range(10000)):
    pass
```

同时你也可以通过`shell`命令操作

```shell
$ seq 9999999 | tqdm --unit_scale | wc -l
10.0Mit [00:02, 3.58Mit/s]
9999999

$ 7z a -bd -r backup.7z docs/ | grep Compressing | tqdm --total $(find docs/ -type f | wc -l) --unit files >> backup.log
100%|███████████████████████████████▉| 8014/8014 [01:37<00:00, 82.29files/s]
```

### Tqdm的用法

`tqdm`是非常通用的，并且可以以多种方式使用。下面给出三个主要部分。

tqdm的用法主要有3种：

- 自动控制
- 手动控制
- 脚本或命令行

#### 1、基于可迭代

总结tqdm()各地可迭代：

```python
import tqdm

text = ""
for char in tqdm(["a", "b", "c", "d"]):
    text = text + char
```

trange(i)是tqdm(range(i))的一个特殊的优化实例，是另一种写法：

```python
for i in trange(100):
    pass
```

在循环外的实例化允许手动控制tqdm()：

```python
pbar = tqdm(["a", "b", "c", "d"])
for char in pbar:
    pbar.set_description("Processing %s" % char)
```

#### 2、手册

tqdm()通过使用with语句对更新进行手动控制：

```python
with tqdm(total=100) as pbar:
    for i in range(10):
        pbar.update(10)
```

如果提供了可选变量total（或可迭代len()），则显示预测统计信息。

with也是可选的（你可以分配tqdm()给一个变量，但在这种情况下，不要忘记del或close()在最后：

```python
pbar = tqdm(total=100)
for i in range(10):
    pbar.update(10)
pbar.close()
```

#### 3、Module

也许最好的用法`tqdm`是在脚本或命令行中。简单地插入`tqdm`（或`python -m tqdm`）之间的管道将穿过所有`stdin`到`stdout`在打印进度`stderr`。

下面的例子演示了当前目录中所有`Python`文件中的行数，包括时序信息。

```shell
$ time find . -name '*.py' -exec cat \{} \; | wc -l
857365

real    0m3.458s
user    0m0.274s
sys     0m3.325s

$ time find . -name '*.py' -exec cat \{} \; | tqdm | wc -l
857366it [00:03, 246471.31it/s]
857365

real    0m3.585s
user    0m0.862s
sys     0m3.358s
```

请注意，tqdm也可以指定通常的参数。

```shell
$ find . -name '*.py' -exec cat \{} \; |
    tqdm --unit loc --unit_scale --total 857366 >> /dev/null
100%|███████████████████████████████████| 857K/857K [00:04<00:00, 246Kloc/s]
```

备份一个大目录？

```shell
$ 7z a -bd -r backup.7z docs/ | grep Compressing |
    tqdm --total $(find docs/ -type f | wc -l) --unit files >> backup.log
100%|███████████████████████████████▉| 8014/8014 [01:37<00:00, 82.29files/s]
```

tqdm就是用来显示进度条的，很漂亮，使用很直观，使用起来非常简单，而且基本不影响原程序效率。如果所有的程序都添加了这样的进度条，是该多么舒服啊！详细资料见[tqdm库的gitub](https://github.com/tqdm/tqdm)

