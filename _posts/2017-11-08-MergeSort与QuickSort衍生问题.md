---
layout:     post
title:      排序算法衍生问题
subtitle:   MergeSort与QuickSort衍生问题
date:       2017-11-08
author:     CDz
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - 算法
    - Java
    - MergeSort
    - QuickSort
---

# 排序算法衍生问题
[上一篇](https://cdz1129.github.io/2017/11/03/O(nlogn)%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95%E6%80%9D%E8%80%83/)具体分析了MergeSort与QuickSort的实现与优化。

从上篇我们可以总结出，不论是MergeSort还是QuickSort都使用了分治算法的思想，即将一个问题分解成相同的子问题，将子问题解决了，原问题也就得以解决。

而MergeSort与QuickSort不同在于，MergeSort注重合并的操作而不注重分。QuickSort注重如何来分，因为需要分的位置已经是其排完序之后所在的位置，故不需要注重合的过程。

故由此可以延伸出一些问题得以使用其两算法的特性来解决。
## MergeSort衍生问题
### 逆序对
何为逆序对？
> 一个数组中 可以有 很多的数字对，其中为逆序的便为逆序对
>
>例如：[3,1,2]中一共有 (3,1) (3,2) (1,2) 这些数对，而其中(3,1) (3,2) 便是其逆序对。


## QuickSort衍生问题
### 取出数组中第n大的元素