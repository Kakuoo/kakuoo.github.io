---
layout: post
title: tmux使用指南
subtitle: false
tags: [Tools]
---

* This will become a table of contents (this text will be scraped).
{:toc}
<!-- ## tmux使用指南 -->

最近由于疫情原因不能返校，只能通过 SSH 远程链接学校的电脑和服务器跑程序。但是使用 SSH 进行远程终端操作时，会将每次 SSH 连接的会话 (session) 与进程进行绑定。当关闭或者断开 SSH 链接时，正在运行的进程也会随之终止。这种问题可通过使用 screen 或 tmux，将 SSH 中的进程调到后台运行进行解决。

tmux是一个 (`Terminal Multiplexer`终端复用器的简称），它可以启动一系列终端会话。

我们使用命令行时，打开一个终端窗口，会话开始，执行某些命令如npm run dev，关闭此终端窗口，会话结束，npm run dev服务会话随之被关闭。有时我们希望我们运行的服务如npm run dev 或者一些cd命令等，被保留，而不是关闭窗口再打开后，重新手动执行。tmux的主要用途就在于此。它解绑了会话和终端窗口。关闭终端窗口再打开，会话并不终止，而是继续运行在执行。将会话与终端窗后彻底分离。

使用场景：

> 1. 关闭终端,再次打开时原终端里面的任务进程依然不会中断 ;
>
> 2. 处于异地的两人可以对同一会话进行操作，一方的操作另一方可以实时看到 ;
>
> 3. 可以在单个屏幕的灵活布局下开出很多终端，然后就能协作地使用它们 ;

### 安装

tmux使用c语言实现的（[tmux的github](https://github.com/tmux/tmux.git)），可运行在OpenBSD，FreeBSD,NetBSD,Linux,OS X,Solaris上。

#### 安装方法一

```powershell
git clone https://github.com/tmux/tmux.git
cd tmux
sh autogen.sh
./configure && make
```

#### 安装方法二

```shell
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux

# Mac
$ brew install tmux
```

## 使用方法

### 启动与退出

讲解使用之前，我们需要先搞清楚窗口与会话的概念。

所谓窗口，其实就是我们终端打开的一个tab，如终端里面所操作的命令、启动的服务，为会话，如下图所示：

在理解了窗口和会话的观念上，我们介绍下tmux使用。

```shell
# 启动tmux
$ tmux

# 退出
$ exit 或 Ctrl+D
```

在终端窗口上，运行tmux，其实就打开了一个终端与tmux服务的会话。只不过我们可以在tmux会话上层，再次输入’会话‘命令，使tmux上层运行的'会话'与终端窗口进行分离。这里面tmux其实可以称之为伪窗口（它其实是会话）。

启动tmux后，底部[0] 表示第0个tmux伪窗口，再启动一个tmux伪窗口，则为[1],依次递增。若使用如下命令，则底部不再是数字，而是命名的名字。

```shell
# 启动命名tmux
$ tmux new -s <name>
```

### 分离会话

在会话窗口上，执行`cd demo`操作后，再执行`tmux detach`，可见退出了tmux伪窗口

```shell
# 分离会话
$ tmux detach
```

执行`tmux ls`可看到当前所有的tmux伪窗口。

### 重连会话

我们通过`tmux detach`关闭tmux伪窗口后，希望能再次进入某一个会话窗口，怎么操做？

```shell
# 重接会话 使用伪窗口编号
$ tmux attach -t 0

# 重接会话 使用伪窗口名称
$ tmux attach -t xiaoqi
```

### 杀死会话

有时候我们想彻底关闭某个会话，不想让其再执行，怎么操作?

```shell
# 使用会话编号
$ tmux kill-session -t 0

# 使用会话名称
$ tmux kill-session -t <name>
```

### 切换会话

```shell
# 使用会话编号
$ tmux switch -t 0

# 使用会话名称
$ tmux switch -t <session-name>
```

### 重命名会话

```bash
$ tmux rename-session -t 0 <new-name>
```

### 其他命令

```shell
# 列出所有快捷键，及其对应的 Tmux 命令
$ tmux list-keys

# 列出所有 Tmux 命令及其参数
$ tmux list-commands

# 列出当前所有 Tmux 会话的信息
$ tmux info

# 重新加载当前的 Tmux 配置
$ tmux source-file ~/.tmux.conf
```