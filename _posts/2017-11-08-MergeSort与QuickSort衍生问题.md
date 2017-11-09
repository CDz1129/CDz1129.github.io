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

解决思路过程：

当然，最简单的办法是，将数组中的每个元素两两比较找出所有的逆序对，此时算法的复杂度为`O(n^2)`。

联系我们上一篇所学的归并排序核心思想，转变一下就可以运用到寻找逆序对的过程中，因为`mergesort`的核心思想是将两个有序的数组合并，那么我们可以思考 这个合并的过程中，就可以寻找到 逆序对，且是一次性寻找到多个，那么就将此问题变成了`O(nlogn)`算法复杂度问题。

![mergesort-inverse](http://wx1.sinaimg.cn/large/63d77fe7gy1flatg0m7zrj20ry08ygmf.jpg)

#### 算法实现
```java
    private static long inverse(int[] arr, int left, int right) {

        if (left - right >= 0) {
            return 0;
        }

        int mid = (left + right) / 2;


        long leftNum = inverse(arr, left, mid);
        long rightNum = inverse(arr, mid + 1, right);
        long num = merge(arr, left, mid, right);
        return leftNum + rightNum + num;
    }


    /**
     * 对于arr[left...mid] 与 arr[mid + 1 ... right],进行归并过程 并找出 逆序对
     *
     * @param arr
     * @param left
     * @param mid
     * @param right
     * @return
     */
    private static long merge(int[] arr, int left, int mid, int right) {

        int[] aux = Arrays.copyOfRange(arr, left, right + 1);

        long inverseNum = 0;
        int k;
        int i = left;
        int j = mid + 1;
        for (k = left; k <= right; k++) {
            if (i > mid) {
                // 如果左半部分元素已经全部处理完毕
                arr[k] = aux[j - left];
                j++;
            } else if (j > right) {
                // 如果右半部分元素已经全部处理完毕
                arr[k] = aux[i - left];
                i++;
            } else if (aux[i - left] <= aux[j - left]){
                // 左半部分所指元素 <= 右半部分所指元素
                arr[k] = aux[i - left];
                i++;
            } else {   // 右半部分所指元素 < 左半部分所指元素
                arr[k] = aux[j - left];
                j++;
                // 此时, 因为右半部分k所指的元素小
                // 这个元素和左半部分的所有未处理的元素都构成了逆序数对
                // 左半部分此时未处理的元素个数为 mid - i + 1
                inverseNum += (long) (mid - i + 1);
            }
        }
        return inverseNum;
    }
```

>   逆序对作用：
>   
>   逆序对的多少，可以最直观的表现出一个数组的有序性。
## QuickSort衍生问题
### 取出数组中第n大的元素

解决思路过程：

>   极端化问题，找出数组中的最大值与最小值，这个问题我们都知道直接遍历整个数组一次就可以找到，此时的算法复杂度为`O(n)`
>   
>   如果要找出第n大元素，此时可以将其数组排序现阶段的最优为 `O(nlogn)` 级别，那么解决此问题的算法复杂度即为`O(nlogn)` 
>   
>   而结合所学的`quicksort`思想，其在一次排序的过程中，已经找出了 待排序元素在此数组中的位子，且 左边小于其，右边大于其，利用此思路，便可以将此问题优化为`O(n)`级别，具体是在找出此元素位子后与`n`做比较，如果`n>j`说明在`n`的位子在`j`的右边，且`j`右边的所有元素都是大于`arr[j]`的,故下一次直接从`j`的右边查询即可。

![quicksort-select-n](http://wx4.sinaimg.cn/large/63d77fe7gy1flauds39o0j20lx0c40t9.jpg)
#### 算法实现
```java
    /**
     * @param arr
     * @param l
     * @param r
     * @return
     */
    private static int partition3Ways(int[] arr, int l, int r, int index) {
        Random random = new Random();
        int rand = l + random.nextInt(r - l + 1);
        CommonUtils.swap(arr, l, rand);
        //标记元素
        int p = arr[l];

        //设置三数组边界，使得其初始化时为空数组。

        //int[l+1...lt] < p
        int lt = l;
        //int[gt...r] < p
        int gt = r + 1;
        //int[lt+1...i) = p
        int i = l + 1;

        while (gt > i) {
            if (arr[i] > p) {
                gt--;
                CommonUtils.swap(arr, i, gt);
            } else if (arr[i] < p) {
                lt++;
                CommonUtils.swap(arr, i, lt);
                i++;
            } else {
                i++;
            }
        }
        CommonUtils.swap(arr, l, lt);
        //此时 lt-1 是因为第一个元素交换后，将arr[l+1...lt]变成了--> arr[l,lt-1]
        //arr[l...lt-1] < p ; arr[lt...gt) = p ; arr[gt...r] > p
        if (lt > index) {

            return quickSort3Ways(arr, l, lt - 1, index);

        } else if (lt <= index && index < gt) {

            return arr[index];

        } else if (gt <= index) {

            return quickSort3Ways(arr, gt, r, index);

        } else {

            return 0;
        }
    }
```
