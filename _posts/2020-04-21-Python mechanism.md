---
layout: post
title: 使用Python时需注意的相关机制
subtitle: false
tags: [Python]
---

* This will become a table of contents (this text will be scraped).
{:toc}

<!-- ## Python的相关机制 -->

### pyc文件的相关说明

> `C`是`compiled` **编译过的**的意思

**操作步骤**

1. 浏览程序目录发现多出了一个`__pycache__`的目录
2. 目录下还有一个`test_print.pyc`文件，cpython-36表示Python解释器的版本
3. 这个`pyc`文件是由Python解释器将**模块的源码**转换为**字节码**

Python这样保存字节码是作为一种**启动速度的优化**

**字节码**

- Python在解释源程序时分为两个步骤
  1. 首先处理源代码，**编译** 生成一个二进制**字节码**
  2. 再对**字节码**进行处理，才回生成CPU能够识别的**机器码**
- 有了模块的字节码之后，下一次运行程序时，如果在**上次保存字节码之后**没有修改过源代码，Python将会加载**.pyc**文件并跳过编译这个步骤
- 当Python重编译时，它会自动检查源文件和字节码文件的时间戳
- 如果你又修改了源代码，下次程序运行时，字节码将自动重新创建



### Linux上的Shebang符合（#！）

- `#!`这个符号叫做`Shebang`或者`Sha-bang`
- Shebang通常在Unix系统脚本中的**第一行开头**使用
- 指明**执行这个脚本文件**的**解释程序**，指明应当使用哪个解释器解释该程序

**使用Shebang的步骤**

1.使用`which`查询`python3`解释器所在路径

```shell
$ which python3
```

2.修改要运行的**主python文件**，在第一行增加以下内容

```shell
# windows下为（例如）
#! /d/ProgramData/Anaconda3/python

# unix下为
#! /usr/bin/python3
```

3.修改**主python文件**的文件权限，增加执行权限

```shell
$ chmod +x cards_main.py
```

4.在需要时执行程序即可

```shell
./cards_main.py
```



### 哈希（Hash）

python中内置有一个名字叫做`hash(o)`的函数

- 接受一个**不可变类型**的数据作为**参数**
- **返回**结果是一个**整数**

哈希是一种**算法**，其作用就是提取数据的特征码（指纹）

- 相同的内容 得到 相同的结果
- 不同的内容 得到 不同的结果

在Python中，设置字典的**键值对**时，会首先对`key`进行`hash`已决定如何在内存中保存字典的数据，以方便后续对字典的操作：**增、删、改、查**

- 键值对的`key`必须是**不可变类型数据**（除list，dict外的所有，如：int，bool，float，complex，long(2.x)）
- 键值对的`value`可以是**任意类型的数据**



### 函数的参数

#### 不可变和可变参数

> 在函数内部，针对参数使用赋值语句，不会影响调用函数时传递的实参变量

无论传递的参数是可变还是不可变，只要**针对参数**使用**赋值语句**，会在**函数内部**修改**局部变量的引用**，**不会影响到外部变量的引用**

> 如果传递的参数是**可变类型**，在函数内部，使用**方法**修改了数据的内容，**会影响到外部的数据**

**特别的对于** `+=`

在`python`中，列表变量使用`+=`本质上是在执行**列表变量**的`extend`方法，不会修改变量的引用，会影响外部的数据

```python
# 相加再赋值
num_list = num_list = num_list

# 本质上是调用列表的extend方法
num_list += num_list
num_list.extend()
```

#### 多值参数

定义支持多值参数的函数

- 有时可能需要 **一个函数** 能够处理的参数 **个数** 是不确定的，这个时候，就可以使用**多值参数**
- python中有两种多值参数
  - 参数名前增加**一个**`*`可以接收**元组**
  - 参数名前增加**两个**`*`可以接收**字典**
- 一般在给多值参数命名时，**习惯**使用以下两个名字
  - `*args` —— 存放元组参数，前面有一个`*`
  - `**kwargs` —— 存放字典参数，前面有两个`*`
- `args` 是 `arguments` 的缩写，有变量的含义
- `kw` 是 `keyword` 的缩写，`kwargs`可以记忆 键值对参数

```python
def sum_numbers(*args):
	num = 0
	print(args)
	
	for n in args:
		num += n
	return num

result = sum_number(1, 2, 3, 4, 5)

# ==========两种方式均可============

def sum_numbers(args):
	num = 0
	print(args)
	
	for n in args:
		num += n
	return num

result = sum_number((1, 2, 3, 4, 5))
```

**元组和字典的拆包**

在调用带有多值参数的函数时，如果希望如下操作：

- 将一个**元组**变量，直接传递给`args`
- 将一个**字典**变量，直接传递给`kwargs`

则可以使用**拆包**，简化参数的传递，**拆包**的方式为：

- 在**元组**变量前，增加一个`*`
- 在**字典**变量前，增加两个`*`

```python
def demo(*args, **kwargs):
	print(args)
	print(kwargs)
	
gl_nums = (1, 2, 3)
gl_dict = {"name": "xiaoming", "age": 18}

# 拆包 错误用法
demo(gl_nums, gl_dict)
# 拆包 正确用法
demo(*gl_nums, **gl_dict)
# 拆包语法可以简化元组变量/字典变量的传递

# 若不使用拆包
demo(1, 2, 3, name="xiaoming", age=18)
```



### from .. import XX 是个啥

例如：官方文档

```text
sound/                          Top-level package
      __init__.py               Initialize the sound package
      formats/                  Subpackage for file format conversions
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
	effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
	filters/                  Subpackage for filters
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
```

1.在当前文件夹`effects`里的`__init__.py`程序中导入`echo`子模块。

```python
from . import echo
```

2.在上一级文件夹`sound`里的`__init__.py`程序中导入`formats`。

```python
from .. import formats
```

3.在上一级文件夹`sound`下的**filters**文件夹里的`__init__.py`程序中导入`equalizer`子模块。

```python
from ..filters import equalizer
```



### from \_\_future\_\_ import *的作用

我们在读代码的时候，总是会看到代码开头会加上from \_\_future\_\_ import *这样的语句。这样的做法的作用就是将新版本的特性引进当前版本中，也就是说我们可以在当前版本使用新版本的一些特性。

例如，在python2.x和python3.x中print的标准写法分别是

```python
# python 2.x
print "Hello World"
 
# python 3.x
print("Hello World")
```

如果你想用python2.x体验python3.x的写法，就可以使用from \_\_future\_\_ import print_function来实现，

```python
# python 2.x
from __future__ import print_function
print("Hello World")
```

而这时候如果再使用原来python2.x的标准写法就会报错

```python
# python 2.x
from __future__ import print_function
print "Hello World"
 
>>> print "Hello World"
  File "<stdin>", line 1
    print "Hello World"
                      ^
SyntaxError: invalid syntax
```

除了print函数，\_\_future\_\_ 模块还有很多其他功能

1.整数除法

```python
# python 2.x
5/2
>>> 2
 
from __future__ import division
5/2
>>> 2.5
```

2.with 用法

```python
# python 2.x
try:
    with open('test.txt', 'w') as f:
    f.write('Hello World')
finally:
    f.close()
 
# 用with替代上述异常检测代码：
from __future__ import with_statement
with open('test.txt', 'w') as f:
    f.write('Hi there!')
```

3.绝对引入（absolute_import）

绝对引入主要是针对python2.4及之前的版本的，这些版本在引入某一个.py文件时，会首先从当前目录下查找是否有该文件。如果有，则优先引用当前包内的文件。而如果我们想引用python自带的.py文件时，则需要使用

```python
from __future__ import absolute_import
```

1. unicode_literals

在Python中有些库的接口要求参数必须是str类型字符串，有些接口要求参数必须是unicode类型字符串。对于str类型的字符串，调用len()和遍历时，其实都是以字节为单位的，这个太就比较坑了，同一个字符使用不同的编码格式，长度往往是不同的。对unicode类型的字符串调用len()和遍历才是以字符为单位，这是我们所想要的。另外，Django，Django REST framework的接口都是返回unicode类型的字符串。为了统一，我个人建议使用from \_\_future\_\_ import unicode_literals，将模块中显式出现的所有字符串转为unicode类型（想要了解更多unicode的知识，可以看[这里](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/Unicode)）。





### 关键字、函数和方法

- **关键字** 是Python内置的、具有特殊意义的标识符

```python
import keyword
print(keyword.kwlist)
print(len(keyword.wklist))

['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 'continue', '
def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if',
 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'retu
rn', 'try', 'while', 'with', 'yield']
```

> 关键字后面不需要使用括号

- **函数** 封装了独立功能，可以直接调用

```
函数名(参数)
```

> 函数需要死记硬背

- **方法** 和函数类似，同样是封装了独立的功能，方法
- 方法需要通过 **对象** 来调用，表示针对这个 对象 要做的操作

```
对象.对象名(参数)
```

> 在对象后面输入`.`，然后选择针对这个变量要执行的操作，记忆起来比函数简单很多



### eval函数

`eval()`函数会将**字符串**当成**有效的表达式**来求值并**返回计算结果**

```python
# 基本的数学计算
>>> eval("1 + 1")
2

# 字符串重复
>>> eval("'*' * 10")
**********

# 将字符串转换成列表
>>> type(eval("[1, 2, 3, 4, 5]"))
list

# 将字符串转换成字典
>>> type(eval("{'name': 'xiangming', 'age': 18}"))
dict
```

不要滥用`eval`

> 在开发时，千万不要使用`eval`直接转换`input`的结果

```python
# 用户可以通过在input()中输入此命令导入os模块，执行任何操作，例如：touch，rm，高危操作
__import__('os').system('ls')
```

等价代码

```python
import os
os.system("终端命令")
```

执行成功返回0，执行失败返回错误信息