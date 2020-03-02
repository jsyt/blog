---
title: 排序算法之快速排序
date: 2018-02-09T21:53:54+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "Sort"]
toc: true
featured_image : ""
keywords: ["排序算法", "快速排序", "quick sort"]
description : "排序算法之快速排序"
summary: "快速排序是图灵奖得主 C. R. A. Hoare 于 1960 年提出的一种划分交换排序。它采用了一种分治的策略，通常称其为分治法(Divide-and-ConquerMethod)。

分治法的基本思想是：将原问题分解为若干个规模更小但结构与原问题相似的子问题。递归地解这些子问题，然后将这些子问题的解组合为原问题的解。

利用分治法可将快速排序的分为三步：

1. 在数据集之中，选择一个元素作为”基准”（pivot）。
"
---


排序算法总览：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/sort-sort.png)


## 快速排序

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/quickSort-quicksort.gif)

快速排序是图灵奖得主 C. R. A. Hoare 于 1960 年提出的一种划分交换排序。它采用了一种分治的策略，通常称其为分治法(Divide-and-ConquerMethod)。

分治法的基本思想是：将原问题分解为若干个规模更小但结构与原问题相似的子问题。递归地解这些子问题，然后将这些子问题的解组合为原问题的解。

利用分治法可将快速排序的分为三步：

1. 在数据集之中，选择一个元素作为”基准”（pivot）。
2. 所有小于”基准”的元素，都移到”基准”的左边；所有大于”基准”的元素，都移到”基准”的右边。这个操作称为分区 (partition) 操作，分区操作结束后，基准元素所处的位置就是最终排序后它的位置。
3. 对”基准”左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。



### JavaScript 递归版

```js
function quickSort(arr){
  var high = arr.length - 1;
  qSort(arr, 0, high);
}

function qSort(arr, low, high){
  if(low >= high){
    return arr;
  }
  var mid = partition(arr, low, high);
  qSort(arr, low, mid-1);
  qSort(arr, mid+1, high);
}

function partition(arr, low, high){
  var pivot = arr[low];
  while(low < high){
    while(low < high && arr[high] >= pivot){
      high--;
    }
    arr[low] = arr[high];
    while(low < high && arr[low] <= pivot){
      low++;
    }
    arr[high] = arr[low];
  }
  arr[low] = pivot;
  return low;
}

```

### JavaScript 迭代版

```js
function quickSort(arr){
  var high = arr.length - 1;
  qSort(arr, 0, high);
}

function qSort(arr, low, high){
  var stack = [];
  var top = -1;
  stack[++top] = low;
  stack[++top] = high;
  while(top >= 0){
    high = stack[top--];
    low = stack[top--];
    var mid = partition(arr, low, high);
    if(low < mid - 1){
      stack[++top] = low;
      stack[++top] = mid - 1;
    }
    if(mid + 1 < high){
      stack[++top] = mid + 1;
      stack[++top] = high;
    }
  }
}

function partition(arr, low, high){
  var pivot = arr[low];
  while(low < high){
    while(low < high && arr[high] >= pivot){
      high--;
    }
    arr[low] = arr[high];
    while(low < high && arr[low] <= pivot){
      low++;
    }
    arr[high] = arr[low];
  }
  arr[low] = pivot;
  return low;
}
```