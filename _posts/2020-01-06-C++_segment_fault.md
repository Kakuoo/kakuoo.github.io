---
layout: post
title: C++ 段错误解析
subtitle: false
tags: [C++]
---
* This will become a table of contents (this text will be scraped).
{:toc}

<!-- ## C++段错误解析 -->

### 一、什么是段错误？

一旦一个程序发生了越界访问，cpu 就会产生相应的保护，于是 segmentation fault 就出现了，通过上面的解释，**段错误应该就是访问了不可访问的内存**，这个内存区要么是不存在的，要么是受到系统保护的，还有可能是缺少文件或者文件损坏。

### 二、段错误产生的原因

下面是一些典型的段错误的原因:
非关联化空指针——这是特殊情况由内存管理硬件
试图访问一个不存在的内存地址(在进程的地址空间)
试图访问内存的程序没有权利(如内核结构流程上下文)
试图写入只读存储器(如代码段)

#### 1、访问不存在的内存地址

在C代码,分割错误通常发生由于指针的错误使用,特别是在C动态内存分配。**非关联化一个空指针总是导致段错误**,但野指针和悬空指针指向的内存,可能会或可能不会存在,而且可能或不可能是可读的还是可写的,因此会导致瞬态错误。

```cpp
#include <stdio.h>

int main (void)
{
	int *ptr = NULL;
	*ptr = 0;
	return 0;
}
输出结果：
段错误（核心已转储）
```

现在,**非关联化这些变量可能导致段错误:非关联化空指针通常会导致段错误**,阅读时从野指针可能导致随机数据但没有段错误,和阅读从悬空指针可能导致有效数据,然后随机数据覆盖。

#### 2、访问系统保护的内存地址

```cpp
#include <stdio.h>

int main (void)
{
	int *ptr = (int *)0;
	*ptr = 100;
	return 0;
}
输出结果:
段错误（核心已转储）
```

#### 3、访问只读的内存地址

写入只读存储器提出了一个 segmentation fault,这个发生在程序写入自己的一部分代码段或者是只读的数据段,这些都是由操作系统加载到只读存储器。

```cpp
#include <stdio.h>
#include <string.h>

int main (void)
{
	char *ptr = "test";
	strcpy (ptr, "TEST");
	return 0;
}
输出结果：
段错误（核心已转储）
#include <stdio.h>

int main (void)
{
	char *ptr = "hello";
	*ptr = 'H';
	return 0;
}
输出结果：
段错误（核心已转储）
```

上述例子ANSI C代码通常会导致段错误和内存保护平台。它试图修改一个字符串文字,这是根据ANSI C标准未定义的行为。大多数编译器在编译时不会抓,而是编译这个可执行代码,将崩溃。

包含这个代码被编译程序时,字符串“hello”位于rodata部分程序的可执行文件的只读部分数据段。**当加载时,操作系统与其他字符串和地方常数只读段的内存中的数据**。当执行时,一个变量 ptr 设置为指向字符串的位置,并试图编写一个H字符通过变量进入内存,导致段错误。编译程序的编译器不检查作业的只读的位置在编译时,和运行类unix操作系统产生以下运行时发生 segmentation fault。

可以纠正这个代码使用一个数组而不是一个字符指针,这个栈上分配内存并初始化字符串的值:

```cpp
#include <stdio.h>

int main (void)
{
	char ptr[] = "hello";
	ptr[0] = 'H';
	return 0;
}
```

即使不能修改字符串(相反,这在C标准未定义行为),在C char *类型,所以没有隐式转换原始代码,在c++的 const char *类型,因此有一个隐式转换,所以编译器通常会抓住这个特定的错误。

#### 4、空指针废弃

因为是一个很常见的程序错误空指针废弃（读或写在一个空指针，用于C的意思是“没有对象指针”作为一个错误指示器），大多数操作系统内存访问空指针的地址，这样它会导致段错误。

```cpp
#include <stdio.h>

int main (void)
{
	int *ptr = NULL;
	printf ("%d\n", *ptr);
	return 0;
}
输出结果：
段错误（核心已转储）
```

这个示例代码创建了一个空指针，然后试图访问它的值（读值）。在运行时在许多操作系统中，这样做会导致段错误。

非关联化一个空指针，然后分配（写一个值到一个不存在的目标）也通常会导致段错误。

```cpp
#include <stdio.h>

int main (void)
{
	int *ptr = NULL;
	*ptr = 1;
	return 0;
}
// 输出结果：
// 段错误（核心已转储）
```

下面的代码包含一个空指针，但当编译通常不会导致段错误，值是未使用的。因此，废弃通常会被优化掉，死代码消除。

```cpp
#include <stdio.h>

int main (void)
{
	int *ptr = NULL;
	*ptr;
	return 0;
}
```

***\*还有，比如malloc 动态分配内存，释放、置空完成后，不可再使用该指针。\****

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
	char* str=(char* )malloc(100);
	if(*str)
	{
		return;	
	}
	strcpy(str,"hello");
	printf("%s\n",str);
	free(str);
	str=NULL;
	strcpy(str,"abcdef");
	return 0;
}
输出结果：
hello
段错误 (核心已转储)
```

#### 5、堆栈溢出

```cpp
#include <stdio.h>
#include <string.h>

int main (void)
{
	main ();
	return 0;
}
输出结果：
段错误（核心已转储）
```

上述例子的无限递归，导致的堆栈溢出会导致段错误，但无线递归未必导致堆栈溢出，优化执行的编译器和代码的确切结构。在这种情况下，遥不可及的代码（返回语句）行为是未定义的。因此，编译器可以消除它，使用尾部调用优化，可能导致没有堆栈使用。其他优化可能包括将递归转换成迭代，给出例子的结构功能永远会导致程序运行，虽然可能不是其他堆栈溢出。

#### 6、内存越界（数组越界，变量类型不一致等）

```cpp
#include <stdio.h>

int main (void)
{
	char test[10];
	printf ("%c\n", test[100000]);
	return 0;
}
输出结果：
段错误（核心已转储）
```

### 三、段错误信息的获取

程序发生段错误时，提示信息很少，下面有几种查看段错误的发生信息的途径。

#### 1、dmesg

dmesg 可以在应用程序崩溃时，显示内存中保存的相关信息。如下所示，通过 dmesg 命令可以查看发生段错误的程序名称、引起段错误发生的内存地址、指令指针地址、堆栈指针地址、错误代码、错误原因等。

```cpp
root@#dmesg
[ 6357.422282] a.out[3044]: segfault at 806851c ip b75cd668 sp bf8b2100 error 4 in libc-2.15.so[b7559000+19f000]
```

#### 2、-g

使用gcc编译程序的源码时，加上 -g 参数，这样可以使得生成的二进制文件中加入可以用于 gdb 调试的有用信息。

**参考：[C语言再学习 -- GCC编译过程](http://blog.csdn.net/qq_29350001/article/details/53339861)**

gdb的简单使用：

(gdb)l  列表（list）
(gdb)r  执行（run）
(gdb)n  下一个（next）
(gdb)q  退出（quit）
(gdb)p  输出（print）
(gdb)c  继续（continue）
(gdb)b 4 设置断点（break）
(gdb)d   删除断点（delete）
