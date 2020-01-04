---
layout: post
title: PyCharm使用教程
subtitle: PyCharm的相关配置
tags: [Python]
---

<!-- ## PyCharm使用教程 -->

### 1.安装和启动步骤

1.1 执行以下终端命令，解压缩下载后的安装包

```shell
tar -zxvf pycharm-professional-2018.1.3.tar.gz
```

1.2 将解压后的目录移动到`/opt`目录下，可以方便其他用户使用

> /opt 目录用户存放给主机额外安装的软件

```shell
sudo mv pycharm-2018.1.3/ /opt
```

1.3 切换工作目录

```shell
cd /opt/pycharm-2018.1.3/bin
```

1.4 启动PyCharm

```shell
./pycharm.sh
```

### 2. 设置专业版启动图标

在专业版中，希望点击左侧任务栏中的图标，启动最新版本PyCharm，可以选择菜单 `Tools/Create Desktop Entry...`可以设置任务栏启动图标

> 注意：设置图标时，需要勾选 `Create the entry for all users`

### 3.卸载之前版本的PyCharm

1）程序安装过程

- 程序文件目录
  - 将安装包解压缩，并且移动到/opt 目录下
  - 所有的相关文件都保存在解压缩的目录下
- 配置文件目录
  - 启动PyCharm后，会在用户家目录下建立一个`.PyCharmxxx`的隐藏目录
  - 保存PyCharm相关的配置信息
- 快捷方式文件
  - `/usr/share/applications/jetbrains-pycharm.desktop`

> 在ubuntu中，应用程序启动的快捷方式通常保存在`/usr/share/applications/`目录

 2）程序卸载

要卸载PyCharm只需要执行以下两步工作：

1.删除解压缩目录

```shell
sudo rm -r /opt/pycharm-2017.1.3/
```

2.删除家目录下用于保存配置信息的隐藏目录

```shell
rm -r ~/.PyCharm2017.3/
```

> 如果不再使用PyCharm还需要将`/usr/share/applications/`下的`jetbrains-pycharm.desktop`删掉
