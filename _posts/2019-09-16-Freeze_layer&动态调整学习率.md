---
layout: post
title: 神经网络性能增强指南（实践篇）
subtitle: Freeze layer & 动态调整学习率
tags: [Deep Learning]
---
<!-- ## 神经网络性能增强指南——Freeze layer&动态调整学习率 -->

### Freeze layer & 不同layer，不同学习率

> 例如：对于分类任务的网络模型，pre-trained model已经具有良好的权重参数，为了迁移后使其适应新的数据分布，可以尝试使model.fc层设置较大初始学习率，例如0.001，而model.features层由于主要作用是提取图像特征，可以尝试使该部分conv，bn层等设置较小初始学习率更新参数
>
> 相比于（从epoch=0开始，只训练model.fc层），或是（当epoch=0时，训练model.fc层，当epoch>=1时，训练model.features以及model.fc层，即同时训练整个网络），在后者中，即使是在epoch>=1时训练整个网络，为网络不同层设置不同学习率，看起来也更符合道理一些。

### 不同优化器处理训练的不同时间阶段

> 在网络初始学习阶段使用Adam迅速搜索最快的下降方向，等到网络收敛速度放缓或接近于局部最优点时，更换optimizer为带动量的SGD，如Momentum，往往带动量的优化器在超参数精挑的情况下的结果比Adam效果更好

所以有时需要新建实例化多个optimizer，并且还需要修改optimizer.param_groups对应的学习率，一般的，**新建optimizer更简单也更推荐，optimizer十分轻量级，所以开销很小，但是，新的优化器会初始化动量等状态信息，这对于使用动量的优化器（有momentum参数的SGD）可能会造成收敛中的震荡**

后文的代码基于下述情况为例进行说明：

**epoch=0时，仅训练model.output.fc层，epoch>=1时，训练整个网络模型的参数，且训练过程中model.features和model.output.fc在optimizer中设置不同的学习率**

```python
import argparse
from pytorchcv.model_provider import get_model as ptcv_get_model

parser = argparse.ArgumentParser(description='PyTorch ImageNet Training')
parser.add_argument('--freeze', default=True, type=bool, help='freeze the model.features or not')
parser.add_argument('--totally_unfreeze', default=False, type=bool, help='unfreeze the whole model from epoch_0 to end or not')

"""
状态表
1.freeze model from epoch_0 to end, ONLY train model.fc:
--freeze True --totally_unfreeze False
2.freeze model in epoch_0, unfreeze from epoch_1 to end:   
--freeze False --totally_unfreeze False
3.unfreeze model from epoch_0 to end:   
--freeze False --totally_unfreeze True
--freeze True --totally_unfreeze True
"""

model = ptcv_get_model(name='efficientnet_b5b', pretrained=True)
num_ftrs = model.output.fc.in_features
model.output.fc = nn.Linear(num_ftrs, args.num_classes)	# 例如 args.num_classes==54

if not args.totally_unfreeze:
    for param in model.parameters():
        param.requires_grad = False
        print("totally_unfreeze = False, param.requires_grad = False")
	else:
        print("totally_unfreeze = True, param.requires_grad = True")
        print("0: model.features.final_block.conv.weight.requires_grad = 								{}".format(model.features.final_block.conv.weight.requires_grad))  
        print("0: model.output.fc.weight.requires_grad = {}".format(model.output.fc.weight.requires_grad))  
```

记得经常打印查看模型的layer是否`requires_grad`，用来验证想法是否实现

若使用到分布式并行训练，可参考如下：

```python
args.distributed = args.world_size > 1 or args.multiprocessing_distributed
'''
其中：
parser.add_argument('--world-size', default=-1, type=int,
                    help='number of nodes for distributed training')
parser.add_argument('--multiprocessing-distributed', action='store_true',
                    help='Use multi-processing distributed training to launch '
                         'N processes per node, which has N GPUs. This is the '
                         'fastest way to use PyTorch for either single node or '
                         'multi node data parallel training')
本次华为云比赛只允许使用一块GPU，因此 --multiprocessing-distributed == False，因此只执行了最后一句，torch.nn.DataParallel
'''
if args.distributed:
    # For multiprocessing distributed, DistributedDataParallel constructor
    # should always set the single device scope, otherwise,
    # DistributedDataParallel will use all available devices.
	if args.gpu is not None:
        torch.cuda.set_device(args.gpu)
        model.cuda(args.gpu)
        # When using a single GPU per process and per
        # DistributedDataParallel, we need to divide the batch size
        # ourselves based on the total number of GPUs we have
        args.batch_size = int(args.batch_size / ngpus_per_node)
        args.workers = int((args.workers + ngpus_per_node - 1) / ngpus_per_node)
        model = torch.nn.parallel.DistributedDataParallel(model, device_ids=[args.gpu])
	else:
        model.cuda()
        # DistributedDataParallel will divide and allocate batch_size to all
        # available GPUs if device_ids are not set
        model = torch.nn.parallel.DistributedDataParallel(model)
elif args.gpu is not None:
    torch.cuda.set_device(args.gpu)
    model = model.cuda(args.gpu)
else:
    # DataParallel will divide and allocate batch_size to all available GPUs
    # if args.arch.startswith('alexnet') or args.arch.startswith('vgg'):
    #     model.features = torch.nn.DataParallel(model.features)
    #     model.cuda()
    # else:
    model = torch.nn.DataParallel(model).cuda()
```

不过需要注意的一点是，创建模型时如果使用了`torch.nn.DataParallel`并行化GPU封装模型，模型保存时的dict就都会有`module.`的前缀，因此optimizer的写法要注意是否需要添加`module.`

```python
if args.optimizer == 'Adam':
    optimizer = torch.optim.Adam([
            {'params': model.module.features.parameters(), 'lr':args.lr*0.1},
            {'params': model.module.output.fc.parameters()}],
            lr = args.lr)
elif args.optimizer == 'RMSprop':
	optimizer = torch.optim.RMSprop([
            {'params': model.module.features.parameters(), 'lr':args.lr*0.1},
            {'params': model.module.output.fc.parameters()}],
            lr = args.lr, alpha=0.9)
elif args.optimizer == 'Momentum':
	optimizer = torch.optim.SGD([
            {'params': model.module.features.parameters(), 'lr':args.lr*0.1},
            {'params': model.module.output.fc.parameters()}],
            lr = args.lr, momentum=args.momentum, weight_decay=args.weight_decay)
else:
	raise Exception('Unknown type of optimizer')

criterion = nn.CrossEntropyLoss().cuda(args.gpu)
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer=optimizer, mode='max', 										factor=0.7, patience=5,min_lr=1e-8)
```

至此模型初始化，optimizer和criterion实例化，都已完成，进入训练主体的逻辑部分

```python
for epoch in range(args.start_epoch, args.epochs):
    if args.distributed:
    train_sampler.set_epoch(epoch)
    # adjust_learning_rate(optimizer, epoch, args)

    if not args.totally_unfreeze and not args.freeze and epoch == args.start_epoch + 1:
    	for param in model.module.parameters():
    		param.requires_grad = True
    	print("epoch >= 1, param.requires_grad = True\n"
    		"Unfreeze all model.parameters() sucessfully")
    print("2.args.freeze = {}".format(args.freeze))
    print("2.args.totally_unfreeze = {}".format(args.totally_unfreeze))

    print("2: model.module.features.final_block.conv.weight.requires_grad = 						{}".format(model.module.features.final_block.conv.weight.requires_grad))
    print("2: model.module.output.fc.weight.requires_grad = 										{}".format(model.module.output.fc.weight.requires_grad))

    print("Optimizer'state_dict()['param_groups'][0]['lr']: 										{}".format(optimizer.state_dict()['param_groups'][0]['lr']))
    print("Optimizer'state_dict()['param_groups'][1]['lr']: 										{}".format(optimizer.state_dict()['param_groups'][1]['lr']))

	# train for one epoch
	train(train_loader, model, criterion, optimizer, epoch, args)

     # evaluate on validation set
        if (epoch + 1) % args.print_freq == 0:
            acc1 = validate(val_loader, model, criterion, args)

            scheduler.step(metrics=acc1)

```



### 关于 state_dict() 和 param_groups 的一点分析：

```python
optimizer = Optim.Adam([
    {'params': model.features.parameters(),'eps': 1e-06},
    {'params': model.output.fc.parameters(), 'lr': 1e-3}],
    lr = 1e-2)

print(optimizer.state_dict())

# 输出
{'state': {},
 'param_groups': [
     {'eps': 1e-06, 'lr': 0.01, 'betas': (0.9, 0.999), 'weight_decay': 0, 'amsgrad': False, 		'params':[...]},
     {'lr': 0.001, 'betas': (0.9, 0.999), 'eps': 1e-08, 'weight_decay': 0, 'amsgrad': 			False, 'params':[]}
 ]
}

# ==========================================
a = [i for i in optimizer.param_groups[0].keys()]
print(a)

# 输出
['params', 'eps', 'lr', 'betas', 'weight_decay', 'amsgrad']
```

对于optimizer含有两组不同params而言，经过测试，可以看出，

- `state_dict()` 为包含2个元素的 `dict`，其中每个元素为`dict`形式，分别为`'state'`，`'param_groups'`
- `param_groups` 为包含2个元素的 `list`，其中每个元素为 `dict` 形式，即`optimizer.param_groups[0]`或`optimizer.param_groups[1]`均为`dict`形式，其每个均包含有六组`key-value`键值对元素，分别为`'params', 'eps', 'lr', 'betas', 'weight_decay', 'amsgrad'`

打印查看`model`的`state_dict()` ：

```python
for key in model.state_dict():
	print(key, "\t", model.state_dict()[key].size())

for k, v in model.features.state_dict().items():
    print(k)
```

打印查看、修改`optimizer`的 `state_dict()` 和 `param_groups`：

```python
print(optimizer.param_groups[0]['lr'])
print(optimizer.param_groups[1]['lr'])

# 方法一，修改optimizer第二组params的学习率lr
optimizer.param_groups[1]["lr"]=0.1
# 方法二，修改optimizer所有组params的学习率lr
for param_group in optimizer.param_groups:
    param_group["lr"] = 0.1
```

另外一个值得注意的点是：

```python
print(optimizer.state_dict()['param_groups'][0]['lr'])

# 修改optimizer的学习率
optimizer.state_dict()['param_groups'][0]['lr'] =0.0001   # 此种方式无效，无法改变dict的值
optimizer.param_groups[0]["lr"]=0.0001  # 此种方式有效，可以改变list的值

print(optimizer.state_dict()['param_groups'][0]['lr'])
```
