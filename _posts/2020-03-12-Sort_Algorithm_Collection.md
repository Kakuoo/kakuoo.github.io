---
layout: post
title: 十大排序算法总结
subtitle: false
tags: [Algorithm & Data Structure]
---
* This will become a table of contents (this text will be scraped).
{:toc}
<!-- ## 十大排序算法总结 -->

### 排序算法

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





### 总结

|  **排序算法**  | 平均时间复杂度 | 原地排序 | 额外空间  | 稳定性 |
| :------------: | :------------: | :------: | :-------: | :----: |
|    插入排序    |     O(n^2)     |    √     |   O(1)    |   √    |
|    归并排序    |    O(nlogn)    |    ×     | O(n+logn) |   √    |
|    快速排序    |    O(nlogn)    |    √     |  O(logn)  |   ×    |
|     堆排序     |    O(nlogn)    |    √     |   o(1)    |   ×    |
|                |                |          |           |        |
| 理想的排序算法 |    O(nlogn)    |    √     |   o(1)    |   √    |

