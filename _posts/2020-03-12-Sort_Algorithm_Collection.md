---
layout: post
title: 十大排序算法总结
subtitle: false
tags: [Algorithm & Data Structure]
---
* This will become a table of contents (this text will be scraped).
{:toc}
<!-- ## 十大排序算法总结 -->

## 排序算法总结

| **排序算法** | 英文名称  | **平均时间复杂度** | **最好情况** | **最坏情况** |   **空间复杂度**    | 稳定性 |
| :----------: | --------- | :----------------: | :----------: | :----------: | :-----------------: | :----: |
|   冒泡排序   | Selection |       O(n²)        |     O(n)     |    O(n²)     |        O(1)         |  稳定  |
|   选择排序   | Bubble    |       O(n²)        |    O(n²)     |    O(n²)     |        O(1)         | 不稳定 |
|   插入排序   | Insertion |       O(n²)        |     O(n)     |    O(n²)     |        O(1)         |  稳定  |
|              |           |                    |              |              |                     |        |
|   希尔排序   | Shell     |      O(nlogn)      |   O(n^1.3)   |    O(n²)     |        O(1)         | 不稳定 |
|   归并排序   | Merge     |      O(nlogn)      |   O(nlogn)   |   O(nlogn)   |     O(n + logn)     |  稳定  |
|   快速排序   | Quick     |      O(nlogn)      |   O(nlogn)   |    O(n²)     | O(1)O(logn)O(nlogn) | 不稳定 |
|              |           |                    |              |              |                     |        |
|    堆排序    | Heap      |      O(nlogn)      |   O(nlogn)   |   O(nlogn)   |        O(1)         | 不稳定 |
|    桶排序    | Bucket    |      O(n + k)      |   O(n + k)   |    O(n²)     |      O(n + k)       |  稳定  |
|   计数排序   | Counting  |      O(n + k)      |   O(n + k)   |   O(n + k)   |      O(n + k)       |  稳定  |
|   基数排序   | Radix     |      O(n × m)      |   O(n × m)   |   O(n × m)   |      O(n + m)       |  稳定  |

**不稳定**：指的是原本有相对顺序的两个相等的元素，经过排序后，相等元素的相对位置发生了改变，例如：原本是[...A_1, A_2...] ，经过排序后变成了[...A_2, A_1...] 

|  **排序算法**  | 平均时间复杂度 | 原地排序 | 额外空间  | 稳定性 |
| :------------: | :------------: | :------: | :-------: | :----: |
|    插入排序    |     O(n^2)     |    √     |   O(1)    |   √    |
|    归并排序    |    O(nlogn)    |    ×     | O(n+logn) |   √    |
|    快速排序    |    O(nlogn)    |    √     |  O(logn)  |   ×    |
|     堆排序     |    O(nlogn)    |    √     |   o(1)    |   ×    |
|                |                |          |           |        |
| 理想的排序算法 |    O(nlogn)    |    √     |   o(1)    |   √    |



## 排序中心思想

根据具体问题的情况讨论具体的方法，分析数据有何特征

**1.是否包含大量重复元素？===> 三路快排**

例如：若此数据中都是独特的，则普通快排就足够了

**2.是否大量数据都距离正确位置很近？===> 插入排序**

例如：银行业务，按照发生时间排序，大量业务完成时存入库中，近乎有序

 **3.是否数据的取值范围非常有限？===> 计数排序**

例如：对学生高考成绩排序，满分750分

**4.对排序是否有额外要求，例如：稳定性？**

例如：快排不稳定，归并排序可能更好

**5.数据具体的存储状况，何种数据结构？**

例如：快排依赖于数据的随机存取，若数据为链表结构，快排可能就不适用了，归并排序更为合适

**6.数据大小是否可以装载在内存中？ ===> 多路归并 TimSort**

例如：数据量很大，内存很小，不足以装载在内存里，需要使用 外部排序算法（例如：多路归并 TimSort()）

**7.大量数据都是否近乎有序？===>  快排随机选取标定点**  



## 进阶思想

**1.如果给定的数组已经排好序呢？你将如何优化你的算法？**

排序 + 双指针

**2.如果 nums1 的大小比 nums2 小很多，哪种方法更优？**

哈希表（较小的数组进行哈希计数，然后在另一数组中根据哈希来寻找）

**3.如果 nums2 的元素存储在磁盘上，磁盘内存是有限的，并且你不能一次加载所有的元素到内存中？**

通过归并外排将两个数组排序后再使用排序双指针查找

**4.对应进阶问题三，如果内存十分小，不足以将数组全部载入内存？**

那么必然也不能使用哈希这类费空间的算法，只能选用空间复杂度最小的算法，即解法一，但是解法一中需要改造，一般说排序算法都是针对于内部排序，一旦涉及到跟磁盘打交道（外部排序），则需要特殊的考虑。归并排序是天然适合外部排序的算法，可以将分割后的子数组写到单个文件中，归并时将小文件合并为更大的文件。当两个数组均排序完成生成两个大文件后，即可使用双指针遍历两个文件，如此可以使空间复杂度最低。



## 概述

1.简单排序算法

- 冒泡排序（Bubble Sort）：基本不用，太慢

- 选择排序：基本不用，不稳定
- 插入排序（Insertion Sort）：适用于样本小且基本有序时效率较高

2.常考的排序算法

- 希尔排序

- 归并排序（Merge Sort）

- 快速排序（Quick Sort）

- 拓扑排序（Topological Sort）

3.其他排序算法

- 堆排序（Heap Sort）

- 桶排序（Bucket Sort）

- 计数排序

- 基数排序

## 详细介绍

### 0.排序算法自定义比较函数

```c++
struct Student
{
    string name;
    int score;

    bool operator<(const Student &other_student)
    {
        // return score < other_student.score;  // 不全面
        return score != other_student.score ? 
               score < other_student.score : name < other_student.name;
    }

    friend ostream &operator<<(ostream &os, const Student &student)
    {
        os << "Student: " << student.name << " " << student.score << endl;
        return os;
    }
};

```



### 1.冒泡排序

思想：时刻保持右端的数组是有序的

```c++
// 直观版
void BubbleSort(vector<int> &nums)
{
    for (int i = 0; i < nums.size() - 1; ++i)
    {
        for (int j = 0; j < nums.size(); ++j)
        {
            if (nums[j] > nums[j + 1])
                swap(nums[j], nums[j + 1]);
        }
    }
}

// 优化版——最好记忆版
void BubbleSort(vector<int> &nums)
{
    for (int i = 0; i < nums.size() - 1; ++i)
    {
        bool is_change = false;
        for (int j = 0; j < nums.size() - 1; ++j)
        {
            if (nums[j] > nums[j + 1])
            {
                swap(nums[j], nums[j + 1]);
                is_change = true;
            }
        }
        if (!is_change)
            break;
    }
}

//  优化版-代码简洁
void BubbleSort(vector<int> &nums)
{
    bool is_change = true; //定义is_change，用于标记每轮遍历中是否发生了交换
    for (int i = 0; i < nums.size() - 1 && is_change; ++i)
    {
        is_change = false; //每轮遍历开始，将is_change设置为 false
        for (int j = 0; j < nums.size() - 1; ++j)
        {
            if (nums[j] > nums[j + 1])
            {
                swap(nums[j], nums[j + 1]);
                is_change = true;
            }
        }
    }
}
```

### 2.选择排序

思想：每次从待排序的数组中找出最值，将最值放在待排序数组的前面，n个数字只需要重复n - 1趟；简单实用，但是**不稳定**

```c++
// 直观版
void SelectSort(vector<int> &nums)
{
    for (int i = 0; i < nums.size() - 1; ++i)
    {
        for (int j = i + 1; j < nums.size(); ++j)
        {
            if (nums[i] > nums[j])
                swap(nums[i], nums[j]);
        }
    }
}

// 优化版 避免了多次swap
void SelectSort(vector<int> &nums)
{
    for (int i = 0; i < nums.size() - 1; ++i)
    {
        int min_idx = i;
        for (int j = i + 1; j < nums.size(); ++j)
        {
            if (nums[min_idx] > nums[j])
            {
                min_idx = j;
            }
        }
        swap(nums[i], nums[min_idx]);
    }
}

// 更优化版
// 在优化版的基础上，同时找到数组中的最小值和最大值，同时进行放置操作
```

### 3.插入排序

思想：每次将待排序的数组元素插入到已经排好序的部分中，可以提前终止内层循环，需要时刻保持前端的数组是有序的，**更适用于基本有序的数组**

```c++
// 直观版
void InsertionSort(vector<int> &nums)
{
    for (int i = 1; i < nums.size(); ++i)
    {
        for (int j = i - 1; j >= 0; --j)
        {
            if (nums[j] > nums[i])
                swap(nums[j], nums[i]);
            else
                break;
        }
    }
}

// 优化版——更好记忆版
void InsertionSort(vector<int> &nums)
{
    for (int i = 1, j, value; i < nums.size(); ++i)
    {
        value = nums[i];
        for (j = i - 1; j >= 0; j--)
        {
            if (nums[j] > value)
                nums[j + 1] = nums[j]; // 平移左边的有序数据
            else
                break;
        }
        nums[j + 1] = value; //插入数据
    }
}

// 优化版-代码简洁
void InsertionSort(vector<int> &nums)
{
    for (int i = 1, j, value; i < nums.size(); ++i)
    {
        value = nums[i];
        for (j = i - 1; j >= 0 && nums[j] > value; --j)
        {
            nums[j + 1] = nums[j];
        }
        nums[j + 1] = value;
    }
}
```



**对于10w个随机数的数组进行排序**

- 直观版的InsertionSort算法用时135s

- 优化版的SelectionSort算法用时为7.725s

- 优化版的InsertionSort算法用时为4.594s

**对于近乎有序的10w个数的数组进行排序**

- 直观版的InsertionSort算法用时0.118s

- 优化版的SelectionSort算法用时为7.731s

- 优化版的InsertionSort算法用时为0.00215s

**对于近乎有序的数组进行排序，插入排序的性能甚至超过了O(nlogn)复杂度的排序算法**



### 4.希尔排序

思想：由插入排序改进得到；例如：使用gap = 4的插入排序排完一遍，再以gap = 2的插入排序排完一遍，最后必须再使用gap = 1的插入排序完成一遍

**优势**：

- 在间隔较大时，元素移动的次数较少，在间隔较少时，例如为1时所需移动的距离就比较短

- 比普通的插入排序效率高

**劣势**： 跳着排序，不稳定

**工程上有的人说如果遇到问题先用希尔排序排一遍，如果效果较好，就不要再使用别的方法了**

> 举例：gap = 4时，下列序列可划分如下，分别对于每组相同索引的元素应用插入排序
>
> [9, 6, 11, 3,  5, 12, 8, 7,  10, 15, 14, 4,  1, 13, 2]
>
> [9, 6, 11, 3], [5, 12, 8, 7], [10, 15, 14, 4], [1, 13, 2]

```c++
// gap为 nums.size() >> 2的希尔排序
void ShellSort(vector<int> &nums)
{
    for (int gap = nums.size() >> 2; gap > 0; gap /= 2)
    {
        for (int i = gap; i < nums.size(); i++)
        {
            for (int j = i; j > gap - 1; j -= gap)
            {
                if (nums[j - gap] > nums[j])
                    swap(nums[j - gap], nums[j]);
                else
                    break;
            }
        }
    }
}

// 修改了j的表达方式
void ShellSort(vector<int> &nums)
{
    for (int gap = nums.size() >> 2; gap > 0; gap >> 1)
    {
        for (int i = gap; i < nums.size(); i++)
        {
            for (int j = i - gap; j >= 0; j -= gap)
            {
                if (nums[j] > nums[i])
                    swap(nums[j], nums[i]);
                else
                    break;
            }
        }
    }
}

// 通用的希尔排序
void ShellSort(vector<int> &nums)
{
    int h = 1;
    while (h <= nums.size() / 3)
    {
        h = h * 3 + 1;
    }

    for (int gap = h; gap > 0; gap = (gap - 1) / 3)
    {
        for (int i = gap; i < nums.size(); i++)
        {
            for (int j = i; j > gap - 1; j -= gap)
            {
                if (nums[j - gap] > nums[j])
                    swap(nums[j - gap], nums[j]);
                else
                    break;
            }
        }
    }
}

// 通用的希尔排序
// knuth序列 h = 1; h = 3*h + 1
// 修改了h和j的表达方式
void ShellSort(vector<int> &nums)
{
    int h = 1;
    int h_res = 1;
    while (h <= nums.size() / 3)
    {
        h_res = h;
        h = h * 3 + 1;
    }

    for (int gap = h_res; gap > 0; gap = (gap - 1) / 3)
    {
        for (int i = gap; i < nums.size(); i++)
        {
            for (int j = i - gap; j >= 0; j -= gap)
            {
                if (nums[j] > nums[i])
                    swap(nums[j], nums[i]);
                else
                    break;
            }
        }
    }
}
```

### 5.归并排序

思想：使用分治的思想，借助递归的形式进行实现

> Java中：
>
> 纯数字型（numerical）数组排序可使用快排，调用 array.sort()   ---> DualPivotQuickSort() 双轴快排
>
> 对于对象（object）的排序一般要求稳定：若未提供Comparator（比较器），则默认调用 array.sort() ，否则使用 LegacyMergeSort()，或者根据 userRequested，使用 TimSort()
>
>  
>
> 其中，TimSort()是改进的归并排序算法
>
> 算法思想：多路归并，将待排序数组分成多块，然后直接两两归并
>
> if  给定值 < MIN_RANGE，即数组较小的情况:
>
> ​	使用 binarySort()，二分插入排序
>
> else：
>
> ​	使用 TimSort()，将其中每一小块按照二分插入排好顺序，再将所有块两两归并



方法一：推荐，效率较高

```c++
void MergeSort(vector<int> &nums, int left, int right)
{
    if (left >= right)
        return;

    // int mid = (left + right) / 2
    int mid = left + (right - left) / 2; // 相比于上式要好，可以避免数值溢出
    MergeSort(nums, left, mid);
    MergeSort(nums, mid + 1, right);
    // 如果nums[mid] <= nums[mid + 1]，且已知数组前半部分，后半部分分别有序，则可以不需要归并
    if (nums[mid] > nums[mid + 1])
        merge_two_sub_vec(nums, left, mid, right);
}

void merge_two_sub_vec(vector<int> &nums, int left, int mid, int right)
{
    vector<int> copy = nums;
    int i = left, j = mid + 1, k = left;
    while (k <= right)
    {
        if (i > mid) // 前两步与后两步的顺序不可颠倒
        {
            nums[k++] = copy[j++];
        }
        else if (j > right)
        {
            nums[k++] = copy[i++];
        }
        else if (copy[i] <= copy[j]) // 归并排序，稳定算法 <=
        {
            nums[k++] = copy[i++];
        }
        else if (copy[i] > copy[j])
        {
            nums[k++] = copy[j++];
        }
    }
}
```



方法二：此方法较为低效，首先没有优化比较的方法，其次push_back()方法受vector扩容机制影响，效率过低

```c++
void MergeSort_2(vector<int> &nums)
{
    if (nums.size() < 2)
        return;

    int mid = nums.size() / 2;
    vector<int> sub_vec1;
    vector<int> sub_vec2;
    for (int i = 0; i < mid; ++i)
        sub_vec1.push_back(nums[i]);
    for (int i = mid; i < nums.size(); ++i)
        sub_vec2.push_back(nums[i]);
    
    MergeSort_2(sub_vec1);
    MergeSort_2(sub_vec2);
    nums.clear();
    merge_sort_two_vec_2(sub_vec1, sub_vec2, nums);
}

void merge_sort_two_vec_2(vector<int> &sub_vec1, 
                          vector<int> &sub_vec2, vector<int> &nums)
{
    int i = 0;
    int j = 0;
    while (i < sub_vec1.size() && j < sub_vec2.size())
    {
        if (sub_vec1[i] <= sub_vec2[j])
        {
            nums.push_back(sub_vec1[i]);
            i++;
        }
        else
        {
            nums.push_back(sub_vec2[j]);
            j++;
        }
    }
    // 将sub_vec1或sub_vec2中剩余元素push进入vec中
    for (; i < sub_vec1.size(); ++i) // 或者 while (i++ != sub_vec1.size())
    {
        nums.push_back(sub_vec1[i]);
    }
    for (; j < sub_vec2.size(); ++j) // 或者 while (j++ != sub_vec2.size())
    {
        nums.push_back(sub_vec2[j]);
    }
}
```



方法三：使用模板

```c++
template <typename T>
void InsertionSort(T arr[], int left, int right)
{
    for (int i = left + 1, j; i <= right; i++)
    {
        T value = arr[i];
        for (j = i - 1; j >= left && arr[j] > value; j--)
        {
            arr[j + 1] = arr[j];
        }
        arr[j + 1] = value;
    }
}

template <typename T>
void __MergeTwoSubVec(T arr[], int left, int mid, int right)
{
    // aux == auxiliary 辅助的空间
    T aux[right - left + 1];
    for (int i = left; i <= right; i++)
    {
        aux[i - left] = arr[i];
    }

    int i = left, j = mid + 1;
    for (int k = left; k <= right; k++)
    {
        if (i > mid)
        {
            arr[k] = aux[j - left];
            j++;
        }
        else if (j > right)
        {
            arr[k] = aux[i - left];
            i++;
        }
        else if (aux[i - left] <= aux[j - left])
        {
            arr[k] = aux[i - left];
            i++;
        }
        else if (aux[i - left] > aux[j - left])
        {
            arr[k] = aux[j - left];
            j++;
        }
    }
}

template <typename T>
void __MergeSort(T arr[], int left, int right)
{
    // if (left >= right)
    //     return;

    // 当递归至数组内元素数量较少时，可以转而使用插入排序
    int INSERTION_SORT_THRESHOLD = 15;
    if (right - left <= INSERTION_SORT_THRESHOLD)
    {
        InsertionSort(arr, left, right);
        return;
    }

    int mid = left + (right - left) / 2;
    __MergeSort(arr, left, mid);
    __MergeSort(arr, mid + 1, right);
    // 如果nums[mid] <= nums[mid + 1]，且已知数组前半部分，后半部分分别有序，则可以不需要归并
    if (arr[mid] > arr[mid + 1])
        __MergeTwoSubVec(arr, left, mid, right);
}

template <typename T>
void MergeSort(T arr[], int n)
{
    __MergeSort(arr, 0, n - 1);
}
```



方法四：自底向上的归并排序 Bottom-Up

```c++
template <typename T>
void MergeSortBU(T arr[], int n)
{
    for (int sz = 1; sz <= n; sz += sz)
    {
        for (int i = 0; i + sz < n; i += sz + sz)
        {
            // 对 arr[i, ..., i+sz-1] 和 arr[i+sz, ..., i+2*sz-1] 进行归并
            __MergeTwoSubVec(arr, i, i + sz - 1, min(i + sz + sz - 1, n - 1));
        }
    }
}
```



### 6.快速排序

思想：冒泡排序的改进版，工程上遇到任何问题都可以先试用快速排序排一遍

> 为了避免**快排在处理单调有序数列中复杂度过高**的问题：
>
> 1.Java中的DualPivotQuickSort（双轴快速排序），首先会判断数组是否单调递增或递减，如果是单调序列，则不用快速排序，直接使用MergeSort
>
> 2.pivot选择从数组中随机选取，然后放在left或者right的位置上，再开始快速排序

快排通用程序部分

```c++
void QuickSort(vector<int> &nums, int left, int right)
{
    if (left >= right)
        return;

    int pivot_idx = partition2_1(nums, left, right);
    QuickSort(nums, left, pivot_idx - 1);
    QuickSort(nums, pivot_idx + 1, right);
}
```

#### 单轴快速排序

方法一：快排——单边版本

```c++
// 1.1  较好理解的版本
int partition1_1(vector<int> &nums, int left, int right)
{
    int pivot = nums[right]; // 取右边为比较过程的基准值
    int mark = left;        // mark用于标记返回时，轴所在的正确的下标位置
    for (int i = left; i < right; ++i)
    {
        if (nums[i] < pivot)
        {
            swap(nums[mark], nums[i]);
            mark++;
        }
    }

    swap(nums[mark], nums[right]);
    return mark;
}

// 1.2
int partition1_2(vector<int> &nums, int left, int right)
{
    int pivot = nums[left]; // 取左边为比较过程的基准值，pivot为轴的值
    int mark = left;        // mark用于标记返回时，轴所在的正确的下标位置
    for (int i = left + 1; i <= right; ++i)
    {
        if (nums[i] < pivot)
        {
            // 小于基准值则mark+1， 并交换位置
            mark++;  // 由于i从left + 1开始，所以此行在下一行的前面
            swap(nums[mark], nums[i]);
        }
    }

    // 基准值与mark对应元素调换位置
    swap(nums[left], nums[mark]);
    return mark;
}

// 1.3
int partition1_3(vector<int> &nums, int left, int right)
{
    int pivot = nums[right]; // 取右边为基准值
    int mark = left - 1;    // mark用于标记返回时的轴的下标位置
    for (int i = left; i < right; ++i)
    {
        if (nums[i] < pivot)
        {
            mark++;
            swap(nums[mark], nums[i]);
        }
    }

    swap(nums[mark + 1], nums[right]);
    return (mark + 1);
}
```



方法二：快排——双边循环版本

> 双指针法：
>
> 此方法针对于类似nums有10w个数字，但是都位于[0, 10]之间，如果采用上述方法，则每次需要swap大量元素实现小于区和大于区的平移，过于复杂，故进行改进，平移过程类似于下图：
>
> [[小于区]...[大于区]...[待处理区]]
>
>  
>
> 下面介绍的实际方法中采用的区域划分如下列所示：
>
> [[小于等于区]...[待处理区]...[大于等于区]]，合理降低了时间复杂度，在面对此种情况时，更为高效

```c++
// 代码实现待补全
```

快排——模板，泛型编程

```c++
#include <stdlib.h>  // for rand() and srand()

template <typename T>
void QuickSort(T arr[], int n)
{
    srand((unsigned int)(time(nullptr)));
    __QuickSort(arr, 0, n - 1);
}

template <typename T>
void __QuickSort(T arr[], int left, int right)
{
    if (left >= right)
        return;

    int pos = partition(arr, left, right);
    __QuickSort(arr, left, pos - 1);
    __QuickSort(arr, pos + 1, right);
}

// 单向循环
template <typename T>
int partition3_1(T arr[], int left, int right)
{
    swap(arr[left + rand() % (right - left + 1)], arr[left]);

    T pivot = arr[left];
    int mark = left;
    for (int i = left + 1; i <= right; i++)
    {
        if (arr[i] < pivot)
        {
            swap(arr[mark + 1], arr[i]);
            mark++;
        }
    }
    swap(arr[mark], arr[left]);
    return mark;
}

// 双向循环
template <typename T>
int partition3_2(T arr[], int left, int right)
{
    swap(arr[left], arr[rand() % (right - left + 1) + left]);
    
    T pivot = arr[left];
    int i = left + 1;
    int j = right;
    while (true)
    {
        while (i <= right && arr[i] < pivot)
            i++;
        while (j >= left + 1 && arr[j] > pivot)
            j--;

        if (i > j)
            break;

        swap(arr[i], arr[j]);
        i++;
        j--;
    }

    swap(arr[j], arr[left]);  // j 或者 (i - 1)
    return j;  // j 或者 (i - 1)
}
```

注意：如果泛型编程中使用容器，需注意传递引用`vector<T> &arr`，上述代码中不需要传递引用，因为传递数组名就是传递指针

#### 双轴快速排序

> 思想：
>
> 分为四个区域，less区，mid区，more区，以及中台区
>
> 两个轴（保证轴1小于轴2），将小于轴1的元素放在左边，大于轴1小于轴2的元素放中间，大于轴2的放右边
>
>
>
> 算法思想：
>
> 如果 数组长度小于QUICKSORT_THRESHOLD：
>
> ​	使用原始的sort排序算法（单轴排序）
>
> 否则：
>
> ​	调用归并排序（TimSort），将数组分成多块
>
> ​	如果 少于67块：
>
> ​		使用TimSort
>
> ​	否则：
>
> ​		说明该数组不是高度结构化的（高度结构化适合做归并排序）
>
> ​		使用QuickSort代替MergeSort
>
> ​		--------------（至此进入QuickSort排序）--------------
>
> ​		如果 数组长度小于INSERTIONSORT_THRESHOLD：
>
> ​			使用插入排序（优化的插入排序，PairInsertionSort（双插入排序），即一次性插入两个数）
>
> ​		否则：
>
> ​			使用双轴快排
>
> ​			--------------（至此进入双轴快速排序）--------------
>
> ​			通过中点向左向右分别移两格，找出五个点，并对五个点的值进行排序
>
> ​			如果 五个点均不相同时：
>
> ​				第二个和第四个点作为轴
>
> ​			否则：
>
> ​				说明五个点中有几个点相等
>
> ​				则使用第三个点进行排序，属于Dutch National Flag（荷兰国旗问题）

```c++
// 代码实现待补全
```

#### 三轴快速排序

> 算法思想：
>
> 分为四个区
>
> [ [小于区]]...[等于区]...[待处理区]...[大于区] ]
>
> 之后递归小于区，大于区两部分，继续进行三路快速排序

```c++
template <typename T>
void QuickSort3Ways(T arr[], int n)
{
    srand((unsigned int)time(nullptr));
    __QuickSort3Ways(arr, 0, n - 1);
}

template <typename T>
void __QuickSort3Ways(T arr[], int left, int right)
{
    int INSERTION_SORT_THRESHOLD = 15;
    if (right - left <= INSERTION_SORT_THRESHOLD)
    {
        InsertionSort(arr, 0, n - 1);
        return;
    }

    // partition部分代码
    swap(arr[left], arr[rand() % (right - left + 1) + left]);
    T pivot = arr[left];

    int left_pos = left;   // arr[left + 1, ..., left_pos] < pivot
    int right_pos = right; // arr[right_pos, ..., right] > pivot
    int i = left + 1;      // arr[left + 1, ..., i) == pivot  前闭后开区间
    while (i < right_pos)
    {
        if (arr[i] < pivot)
        {
            swap(arr[i], arr[left_pos + 1]);
            left_pos++;
            i++;
        }
        else if (arr[i] > pivot)
        {
            swap(arr[i], arr[right_pos - 1]);
            right_pos--;
        }
        else if (arr[i] == pivot)
        {
            i++;
        }
    }
    swap(arr[left_pos], arr[left]);

    __QuickSort3Ways(arr, left, left_pos - 1);
    __QuickSort3Ways(arr, right_pos, right);
}
```

