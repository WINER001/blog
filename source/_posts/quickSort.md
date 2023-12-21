---
title: QuickSort
date: 2023/12/14 21:12
tags: 
  - 知识点
categories: 知识点
top_img: ./img/41.jpg
cover: ./img/66.jpg
---



# QuickSort（快排）

### 1.代码

```java
public static void quickSort(int[] a,int left,int right){
	int l = left;
    int r = right;
    while(l<r){
        int tar = a[l]; //选一个基准
        while(l<r && a[r]> tar){//从右边开始，找一个小于tar的
            r --;
        }
        if(l<r){//找到了，放到当前
            a[l++] = a[r];
        }
        while(l<r && a[l] < tar){//从左边开始，找一个大于tar的
            l ++;
        }
        if(l< r){//找到了，放到右面
            a[r--] = a[l];
        }
    }
    a[r] = tar;
    quickSort(a,left,l-1);//递归左边
    quickSort(a,l+1,right);//递归右边
}
```



### 2.时间复杂度nlogn

这是一个递归问题，O(n) = n +2O(n/2)
递归问题的时间复杂度计算有个公式

```
对于T(n) = aT(n/b) + f(n):
f(n) < n^(logba) ,则 T(n) = O(n^(logba))
f(n) = n^(logba) ,则 T(n) = O(n^(logba) * logn)
f(n) > n^(logba) ,则 T(n) = O(f(n))
```

故快排的时间复杂度为nlogn
