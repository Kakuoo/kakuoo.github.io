---
layout: post
title: SSH远程连接Linux服务器指南
subtitle: false
tags: [Linux]
---
* This will become a table of contents (this text will be scraped).
{:toc}

<!-- ## SSH远程连接Linux服务器指南 -->

### 背景

作为IT从业人员，我们要防范一切黑客可以攻击服务器的潜在危险点，这不仅是为自己也是为别人负责。

使用密码连接远程Linux服务器，使黑客有一定几率撞对密码而达到登陆服务器的目的，更糟糕的情况是被黑的user可能还有sudo权限。

**解决这个问题的办法就是使用SSH通过密钥连接远程Linux服务器，并且禁止使用密码连接服务器。**

### SSH介绍

SSH全称Secure Shell，也称Socket Socket Shell，是一种网络协议，他给管理员提供一种安全的方式访问远程计算机，是一种公钥加密方式。在ssh protocol version 2中提供5种类型密钥，分别是：

```
RSA
RSA1
DSA
ECDSA
ED25519
```

### 步骤1：生成密钥对

首先在客户端上使用以下命令生成RSA密钥对

```
ssh-keygen -t rsa
```

可以不设置密码， 一路回车，就可以在`C:\Users\Kaku\.ssh`文件夹下生成==公钥（id_rsa.pub）==和==私钥（id_rsa）==，我们可以拷贝公钥给别的主机，然后私钥保留给自己。

读者可能有两个疑惑：

第一，生成的密钥对应当放在何处？

因为密钥一定要牢牢把握在自己手中，不能让别人知道。所以我们一定要在自己的物理机上生成密钥对，千万不能在远程计算机上生成，否则就不符合密钥的定义了。

第二，如何生成密钥对？

1. 在本机终端运行`ssh-keygen`命令, 即可生成一对默认的RSA密钥。管理员可以通过`ssh-keygen -t`命令自定义上述的5种密钥类型，具体可以通过`man ssh-keygen`查看。这里我们用默认的RSA密钥即可。
2. 然后自己输入保存的路径，推荐保存到home目录的.ssh文件夹下。
3. 输入管理密码(不建议为空，并且需记住，可以与系统账户密码不同)。注意，这里的密码是防止别人用你的电脑SSH无密码登陆远程服务器，可以理解成开机密码，可防止别人乱动你的电脑。

### 步骤2：把公钥内容复制到服务器的认证列表中

这里读者可能又有三个问题。

第一，什么是服务器认证列表？

服务器认证列表是一个文件（authorized_keys），可以理解为<存储用户SSH公钥的地方>，因为SSH是一个验证过程，所以服务器需要事先保存对方的公钥，这样管理员就可以指定哪些用户(准确说是密钥对)可以登录了。

第二，认证列表的路径是什么？

在服务器的配置文件`/etc/ssh/sshd_config`中记录的着认证列表的目录。

首先, 我们可以先进入服务器(若没有设置SSH登陆只能用密码登陆)，然后进入此路径查看，如下：

```shell
cat /etc/ssh/sshd_config
```

查看`sshd_config`文件，其中一行可以看到服务器认证列表位于`/.ssh/authorized_keys`（注意，此文件不一定存在，若不存在，后续需自行创建)，如下所示：

```shell
#AuthorizedKeysFile     %h/.ssh/authorized_keys
```

第三，如何把公钥复制进认证列表？

首先，认证列表文件不一定存在，所以我们要先在远程服务器上执行创建命令, 并设置权限：

```shell
mkdir .ssh  //创建文件夹
touch .ssh/authorized_keys  // 创建文件
#chmod 700 .ssh      //设置权限
#chmod 600 .ssh/auauthorized_keys  //设置权限
chmod 644 .ssh/auauthorized_keys  //设置权限
```

然后用`vim`编辑器打开:

```jsx
vim ~/.ssh/authorized_keys
```

如果没设置过SSH的公钥，里面内容是为空的。如果设置过SSH公钥，则空行添加公钥。

第三步也可进行更简便的操作：利用ssh自带的工具`ssh-copy-id`直接在客户端进行部署密钥，将密钥拷贝到服务端对应位置，这里是在root用户下，也可以使用别的用户，注意把`yourip`替换为服务器的`ip`或者域名

```shell
ssh-copy-id -i /root/.ssh/id_rsa.pub root@your_ip
```

然后修改文件名`id_ras.pub`为`authorized_keys` 

```shell
mv id_rsa.pub authorized_keys #重命名
```

### 步骤3：SSH远程连接Linux服务器

一切设置都已完成，我们如何连接到远程服务器呢？命令格式如下，中括号`[]`内为可选参数:

```shell
ssh user@remote [-p port]
#或者如下所示
ssh student@127.0.0.1 -p 2222 [-i ~/.ssh/MyLinux]
```

> 各部分参数含义如下：
>
> - ssh：表示ssh连接
> - user：表示连接服务器的用户名
> - remote：表示远程主机的host IP(这里是本机)
> - -p port：表示远程主机端口(默认22，可在`sshd_config`文件中查看)
> - -i ~/.ssh/id_rsa.pub：表示用户公钥

### 实践步骤4：设置只许SSH登录，禁止密码登陆

最终我们的目的是消除密码登陆这一留给黑客的安全隐患，而只采用用SSH登陆，故我们在服务器配置文件`/etc/ssh/sshd_config`里小小的设置一下即可。

```shell
vim /etc/ssh/sshd_config
```

```shell
#禁用密码验证
PasswordAuthentication no
#启用密钥验证
RSAAuthentication yes
PubkeyAuthentication yes
```

把其中的`PasswordAuthentication`中的yes改成no，然后重启SSH服务即可

```shell
service sshd restart #centos系统
service ssh restart #ubuntu系统
/etc/init.d/ssh restart #debian系统
```

### 可能遇到的问题：

当我们要拷贝了一份公钥给别的主机后，如果使用`FinalShell`，`MobaXterm`等这类SSH软件进行公钥连接时会发现，不支持`ssh-keygen`生成的`id_rsa`私钥文件，原因是`ssh-keygen`生成的密钥文件有新旧版本不同的区别，使用`ssh-keygen`生成的密钥发现变成了如下格式：

```shell
-----BEGIN OPENSSH PRIVATE KEY-----
#(此处代表文件内容)
-----END OPENSSH PRIVATE KEY-----
```

这是一种新的密钥格式， 而且很多软件对这种格式的密钥都是不支持的。如此不得不把私钥转换成`RSA – PEM`格式。可以使用ssh-keygen的命令进行转换

```shell
ssh-keygen -p -m PEM -f "./id_rsa"
```

得到的`id_rsa`为如下形式：

```shell
-----BEGIN RSA PRIVATE KEY-----
#(此处代表文件内容)
-----END RSA PRIVATE KEY-----
```

