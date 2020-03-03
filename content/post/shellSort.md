---
title: 排序算法之希尔排序
date: 2018-02-08T21:53:54+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "Sort"]
toc: true
# featured_image : ""
keywords: ["排序算法", "希尔排序", "shell sort"]
description : "排序算法之希尔排序"
summary: "希尔排序(Shell's Sort)也称递减增量排序算法，是插入排序的一种更高效的改进版本。但希尔排序是非稳定排序算法。
希尔排序是基于插入排序的以下两点性质而提出改进方法的：
* 插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率；
* 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位；

希尔排序的基本思想是：先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录基本有序时，再对全体记录进行依次直接插入排序。"
---



排序算法总览：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/sort-sort.png)


## 希尔排序

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/shellSort.jpeg)


希尔排序(Shell's Sort)也称递减增量排序算法，是插入排序的一种更高效的改进版本。但希尔排序是非稳定排序算法。
希尔排序是基于插入排序的以下两点性质而提出改进方法的：
* 插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率；
* 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位；

希尔排序的基本思想是：先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录基本有序时，再对全体记录进行依次直接插入排序。



## 插入排序算法回顾

```js
function insertSort(arr){
  var len = arr.length;
  var temp;
  for(var i=1; i<len-1; i++){
    if(arr[i-1] > arr[i]){
      temp = arr[i];
      for(var j = i-1; j >= 0 && arr[j] > temp; j--){
        arr[j+1] = arr[j];
      }
      arr[j+1] = temp;
    }
  }
}

```

## 希尔排序算法实现

```js
function shellSort(arr){
  var len = arr.length;
  var temp;
  for(var gap = Math.floor(len/2); gap > 0; gap = Math.floor(gap/2)){
    for(var i=gap; i<len; i++){
      if(arr[i-gap] > arr[i]){
        temp = arr[i];
        for(var j = i-gap; j >= 0 && arr[j] > temp; j -= gap){
          arr[j+gap] = arr[j];
        }
        arr[j+gap] = temp;
      }
    }
  }
  return arr;
}

// 测试
var arr = [3, 2, 4, 9, 1, 5, 7, 6, 8];
var arrSorted = shellSort(arr);
console.log(arrSorted);
// 控制台将输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
```