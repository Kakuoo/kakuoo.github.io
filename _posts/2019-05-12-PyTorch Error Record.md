---
layout: post
title: PyTorch错误记录
subtitle: PyTorch Error Record
tags: [PyTorch]
comments: true
---

<!-- # PyTorch Error Record -->

### 没有if子句保护的多处理（Multiprocessing）错误

```shell
RuntimeError:
        An attempt has been made to start a new process before the
        current process has finished its bootstrapping phase.

        This probably means that you are not using fork to start your
        child processes and you have forgotten to use the proper idiom
        in the main module:

            if __name__ == '__main__':
                freeze_support()
                ...

        The "freeze_support()" line can be omitted if the program
        is not going to be frozen to produce an executable.
```

此问题有关于windows上多进程实现的方式。在windows上，子进程会自动import启动它的这个文件，而在import的时候是会自动执行这些语句的。如果不加`__main__`限制的话，就会无限递归创建子进程，进而报错。于是import的时候使用 `name == "__main__"` 保护起来就可以了；在Linux环境中，相同的代码不添加 `name == "__main__"`，也可以正常运行。

`multiprocessing`在Windows上，使用`spawn`代替的实现有所不同`fork`。因此，我们必须用if子句包装代码，以防止代码多次执行。将代码重构为以下结构。

```python
import torch

def main()
    for i, data in enumerate(dataloader):
        # do something here

if __name__ == '__main__':
    main()
```

### CUDA IPC 操作错误

```
THCudaCheck FAIL file=torch\csrc\generic\StorageSharing.cpp line=252 error=63 : OS call failed or operation not supported on this OS
```

Windows不支持它们。像对CUDA张量执行多处理之类的操作无法成功，有两种选择。

1.不要使用`multiprocessing`。设置`num_worker`的 [`DataLoader`](https://pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader)为零。

2.改为共享CPU张量。确保您的自定义 `DataSet`返回CPU张量。



参考网址：[PyTorch官方社区](https://pytorch.org/docs/stable/notes/windows.html#cuda-ipc-operations)