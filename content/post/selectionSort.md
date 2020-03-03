---
title: 排序算法之选择排序
date: 2018-02-08T21:53:54+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "Sort"]
toc: true
# featured_image : ""
keywords: ["排序算法", "选择排序", "select sort"]
description : "排序算法之选择排序"
summary: "选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理是每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完。 选择排序是不稳定的排序方法（比如序列[5， 5， 3]第一次就将第一个[5]与[3]交换，导致第一个5挪动到第二个5后面）。"
---




排序算法总览：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/sort-sort.png)


## 选择排序

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/selectSort.png)


选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理是每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完。 选择排序是不稳定的排序方法（比如序列[5， 5， 3]第一次就将第一个[5]与[3]交换，导致第一个5挪动到第二个5后面）。


## 算法实现

```javascript
function selectionSort(arr){
  var len = arr.length,
      minIndex = 0;
  for(var i = 0; i < len-1; i++){
    minIndex = i;
    for(var j = i + 1; j < len; j++){
      if(arr[j] < arr[minIndex]){
        minIndex = j;
      }
    }
    if(minIndex != i){
      [arr[minIndex], arr[i]] = [arr[i], arr[minIndex]]
    }
  }
  return arr;
}

// 测试
var arr = [3, 2, 4, 9, 1, 5, 7, 6, 8];
var arrSorted = selectionSort(arr);
console.log(arrSorted);
// 控制台将输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
```