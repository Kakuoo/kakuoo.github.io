---
layout: post
title: C++ 加速输入输出
subtitle: 使用sync_with_stdio()与cin.tie()
tags: [C++]
---
* This will become a table of contents (this text will be scraped).
{:toc}

<!-- ## sync_with_stdio与cin.tie的加速输入输出 -->

我是怎么在不知道这一对函数的情况下活到今天的，以前碰到cin TLE的时候总是傻乎乎地改成scanf，甚至还相信过C++在IO方面效率低下的鬼话，殊不知这只是C++为了兼容C而采取的保守措施。

```c++
#include <iostream>
int main() 
{
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);	//std::cin.tie(0);
    // IO
}
```

### sync_with_stdio()

这个函数是一个“是否兼容stdio”的开关，C++为了兼容C，保证程序在使用了std::printf和std::cout的时候不发生混乱，将输出流绑到了一起。

在c++中之所以cin，cout效率低，是因为先把要输出的东西存入缓冲区，再输出，导致效率降低，而这段语句可以来打消iostream的输入和输出缓存，可节省时间，使效率与scanf与printf相差无几，还有应注意的是scanf与printf使用的头文件应是stdio.h而不是 iostream。

用大白话讲，cin慢是有原因的，其实默认的时候，cin与stdin总是保持同步的，也就是说这两种方法可以混用，而不必担心文件指针混乱，同时cout和stdout也一样，两者混用不会输出顺序错乱。正因为这个兼容性的特性，导致cin有许多额外的开销，如何禁用这个特性呢？只需一个语句std::ios::sync_with_stdio(false)，这样就可以取消cin与stdin的同步了。

### tie()

tie是将两个stream绑定的函数，空参数的话返回当前的输出流指针。

std :: cin默认是与std :: cout绑定的，所以每次操作的时候都要调用flush，这样增加了IO的负担，通过tie(nullptr)或tie(0)来解除std :: cin和std :: cout之间的绑定，进一步加快执行效率。

```c++
#include <iostream>
#include <fstream>
 
///////////////////////////SubMain//////////////////////////////////
int main(int argc, char *argv[])
{
	std::ostream *prevstr;
	std::ofstream ofs;
	ofs.open("test.txt");
 
	std::cout << "tie example:\n";	// 直接输出到屏幕
 
	*std::cin.tie() << "This is inserted into cout\n";	// 空参数调用返回默认的output stream，也就是cout
	prevstr = std::cin.tie(&ofs);						// cin绑定ofs，返回原来的output stream
	*std::cin.tie() << "This is inserted into the file\n";	// ofs，输出到文件
	std::cin.tie(prevstr);									// 恢复
 
	ofs.close();
	system("pause");
	return 0;
}
///////////////////////////End Sub//////////////////////////////////
```

输出：

```shell
tie example:
This is inserted into cout
请按任意键继续. . .
```

同时当前目录下的test.txt输出：

```text
This is inserted into the file
```

### 应用

在ACM里，经常出现数据集超大造成 cin TLE的情况。这时候大部分人（包括原来我也是）认为这是cin的效率不及scanf的错，甚至还上升到C语言和C++语言的执行效率层面的无聊争论。其实像上文所说，这只是C++为了兼容而采取的保守措施。我们可以在IO之前将stdio解除绑定，这样做了之后要注意不要同时混用cout和printf之类。

在默认的情况下cin绑定的是cout，每次执行 << 操作符的时候都要调用flush，这样会增加IO负担。可以通过tie(0)（0表示NULL）来解除cin与cout的绑定，进一步加快执行效率。