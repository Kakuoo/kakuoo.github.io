---
layout: post
title: C++，python的常用计时方法及辨析
subtitle: false
tags: [C++, Python]
---

<!-- ## C++，python的常用计时方法及辨析 -->

首先介绍最常用的，但两种精度不是很高（>=10ms）的方法：`clock()`和`GetTickCount()`。接下来是两种高精度的计时方法： `QueryPerformanceCounter()`和`gettimeofday()` 

## C++的计时方法：

### 1. clock()

C系统调用方法，所需头文件`ctime/time.h`，即windows和linux都可以使用。`clock()` 函数返回从开启这个程序进程到程序中调用`clock()`函数之间的CPU时钟计时单元（clock tick）数（墙上时间），返回单位是毫秒，可以用常量CLOCKS_PER_SEC将时间显示单位换算为秒， 这个常量表示每一秒（per second）有多少个时钟计时单元

```c++
#include <ctime>
#include <iostream>
using namespace std;
int main()
{
    clock_t start_time = clock();

    long i = 100000000;
    while (i--) {}

    clock_t end_time = clock();

    double time = double(end_time - start_time) / CLOCKS_PER_SEC;
	cout << "time = " << time << " s" << endl;
    return 0;
}
```

乍一看很合理呀， 程序开始获取，程序结束了获取时间，但是运行多线程程序的时候，时间并没有减少，甚至还有所增加。后来查了资料：

> 发现clock()是程序从启动到函数调用占用CPU的时间。这个函数返回从“开启这个程序进程”到“程序中调clock()函数”时之间的CPU时钟计时单元（clock tick）数。

多线程程序是把任务分到多个线程上，并行计算。而clock()函数计算的是所有的cpu的时间，这样跟单线程的就没有区别了，因此用这个函数不太理想。

### 2. GetTickCount()

`GetTickCount()`是一个Windows API，所需头文件为`<windows.h>`

返回从操作系统启动到现在所经过的毫秒数（ms），精确度有限，跟CPU有关，一般精确度在16ms左右，最精确也不会精确过10ms。它的返回值是`DWORD (unsigned long)`，当统计的毫妙数过大时，将会使结果归0，影响统计结果

```c++
#include <windows.h>
#include <iostream>
using namespace std;
int main()
{
    DWORD start_time = GetTickCount();

    long i = 100000000;
    while (i--) {}

    DWORD end_time = GetTickCount();

    double time = double(end_time - start_time) / CLOCKS_PER_SEC;
	cout << "time = " << time << " s" << endl;
    return 0;
}
```

### 3. QueryPerformanceCounter()

`QueryPerformanceCounter()`是一个Windows API，所需头文件为<windows.h>

这个函数返回高精确度性能计数器的值,它可以以微秒为单位计时，但是`QueryPerformanceCounter()` 确切的精确计时的最小单位是与系统有关的，所以，必须要查询系统以得到`QueryPerformanceCounter()`返回的嘀哒声的频率.。`QueryPerformanceFrequency()` 提供了这个频率值,返回每秒嘀哒声的个数

```c++
#include <windows.h>
#include <iostream>
using namespace std;

int main()
{
    LARGE_INTEGER start_time, end_time, query_frequency;
    QueryPerformanceFrequency(&query_frequency);
    QueryPerformanceCounter(&start_time);

    long i = 100000000;
    while (i--) {}

    QueryPerformanceCounter(&end_time);

    double time = double(end_time.QuadPart - start_time.QuadPart / query_frequency.QuadPart;
    cout << "time = " << time << " s" << endl;
    return 0;
}
```

### 4. gettimeofday()

`gettimeofday()`是 linux环境下的计时函数，`gettimeofday()`会把下式中目前的时间由`tv`所指的结构返回，当地时区的信息则放到`tz`所指的结构中

函数原型

```c++
#include <sys/time.h>
int gettimeofday(struct timeval *tv, struct timezone *tz )
```

这个函数会把时间包装为一个结构体返回。包括秒，微秒，时区等信息

```c++
struct timeval
{
    long tv_sec;  /*秒*/
    long tv_usec; /*微妙*/
};
struct timezone
{
    int tz_minuteswest; /*和greenwich时间差了多少分钟*/
    int tz_dsttime; /*日光节约时间的状态*/
};
```

`gettimeofday()`函数获取从1970年1月1日到现在经过的时间和时区（UTC时间），（按照linux的官方文档，时区已经不再使用，正常应该传`nullptr`）。

```c++
#include <sys/time.h>
#include <iostream>
using namespace std;
int main()
{
    timeval start_time;  // C中需包含struct关键字，C++可以省略，struct timeval start_time;
    timeval end_time;    // C中需包含struct关键字，C++可以省略，struct timeval end_time;
    // struct timezone tz;

    gettimeofday(&start_time, nullptr);  //gettimeofday(&start_time, &tz);结果一样
    cout << start_time.tv_sec << endl;
    cout << start_time.tv_usec << endl;

    long i = 100000000;
    while (i--) {}

    gettimeofday(&end_time, nullptr);
    cout << end_time.tv_sec << endl;
    cout << end_time.tv_usec << endl;

    double time = double(end_time.tv_sec - start_time.tv_sec) 
        		+ double(end_time.tv_usec - start_time.tv_usec)/1000000.0; //换算为秒
    
   // double time = double(end_time.tv_sec - start_time.tv_sec) * 1000000
   //			+ (end_time.tv_usec - start_time.tv_usec);	// 换算为微秒
    
    cout << "time = " << time << " s" << endl;
    return 0;
}
```

### 5. 额外的 time()方法

还有一种C系统调用方法--`time()`，但是精度很低（秒级），不建议使用，这里就稍微带下用法

```c++
#include <ctime>
#include <iostream>
using namespace std;
int main()
{
    time_t start_time = time(nullptr);

    long i = 100000000;
    while (i--) {}

    time_t end_time = time(nullptr);

    cout << "time = " << double(end_time - start_time) / CLOCKS_PER_SEC << " s" << endl;
    return 0;
}
```



### 辨析：

**`clock_t`与`time_t`的区别及联系**

`clock()`返回从“开启这个程序进程”到“程序中调用`clock()`函数”时之间的**CPU时钟计时单元数（clock tick）**，而`sleep(5)`并不占用cpu资源，导致`start_time`和`end_time`返回的数值相同。`time(&temp)`返回从CUT（Coordinated Universal Time）时间1970年1月1日00:00:00（称为UNIX系统的Epoch时间）到当前时刻的秒数，然后调用`localtime`函数将 `time_t`所表示的UTC时间转换为本地时间（我们是+8区，比UTC多8个小时）并转成`struct tm`类型，该类型的各数据成员分别表示年月日时分秒。

总之，用`time_t`计时才是人们正常意识上的秒数（**墙上时钟，系统时间戳**），而clock_t计时所表示的是占用CPU的时间（**CPU时钟计时单元数**）。

 `time（NULL）`是指返回从1970年1.1日（元旦）午夜0点到现在的秒数。是实际时间，以秒为单位，误差较大
 `clock()`进程使用的cpu时间，作用不大。处理器时间或频率，以毫秒为单位，通常配合CLOCKS_PER_SEC使用



## python的计时方法：

### 1. time.time()

```python
import time

start_time = time.time()
func()
end_time = time.time()

print str(end_time)
```

### 2. time.clock()

```python
import time

start_time = time.clock()
func()
end_time = time.clock()

print str(end_time - start_time)
```

### 3. datetime()

```python
import datetime

start_time = datetime.datetime.now()
func()
end_time = datetime.datetime.now()

print (end_time - start_time)
```

其中，`time.time()`的精度比较高。`datetime(`)基本上是性能最差的。这个其实是和系统有关系。一般我们推荐使用`time.time()`和`time.clock()`。在Linux系统，`time.time()`返回的是UTC时间。 在很多系统中`time.time()`的精度都是非常低的，包括windows。

`datetime()`和`time.time()`都包含了其他程序使用CPU的时间，是程序开始到程序结束的运行时间，`time.clock()`只计算了程序运行的CPU时间

总的来讲，在 Unix 系统中，建议使用 `time.time()`，在 Windows 系统中，建议使用 `time.clock()`。

### 辨析：

**python `time`包中的`time.time()`和`time.clock()`区别和使用**

- cpu 的运行机制：cpu是多任务的，例如在多进程的执行过程中，一段时间内会有对各进程被处理。一个进程从开始到结束其实是在这期间的一些列时间片断上断断续续执行的。所以这就引出了程序执行的**cpu时钟计时单元数**（该程序单纯在cpu上运行所需时间）和**墙上时钟wall time**。
- `time.time()`是统计的wall time(即墙上时钟)，也就是系统时钟的时间戳（1970纪元后经过的浮点秒数）。所以两次调用的时间差即为系统经过的总时间。
- `time.clock()`是统计cpu时间的工具，这在统计某一程序或函数的执行速度最为合适。两次调用time.clock()函数的差值即为程序运行的cpu时间
- `time.sleep()`只影响`time.time()`，而不影响`time.clock()`

### 相关名词解析

#### 时间戳

时间戳是自 1970 年 1 月 1 日（08:00:00 GMT）至当前时间的总秒数。它也被称为 Unix 时间戳（Unix Timestamp），它在unix、c的世界里随处可见；常见形态是浮点数，小数点后面是毫秒。两个时间戳相减就是时间间隔（单位：秒）

#### 当前时间

```bash
>>> import datetime
>>> import time

>>> now = time.strftime("%Y-%m-%d %H:%M:%S") // 大小写要注意，不同%字母代表不同值
>>> print (now)
2020-05-26 12:11:27

>>> now = datetime.datetime.now()
>>> print(now)
2020-05-26 12:11:57.467664
```

