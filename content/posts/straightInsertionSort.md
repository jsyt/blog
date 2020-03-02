---
title: 排序算法之直接插入排序
date: 2018-02-08T21:53:54+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "Sort"]
toc: true
featured_image : ""
keywords: ["排序算法", "插入排序", "insert sort"]
description : "排序算法之直接插入排序"
summary: "常见的内部排序算法有：冒泡排序、选择排序、插入排序、希尔排序、归并排序、快速排序、堆排序、基数排序等。这里主要介绍直接插入排序.
直接插入排序(Straight Insertion Sort)的基本思想是：把n个待排序的元素看成为一个有序表和一个无序表。开始时有序表中只包含1个元素，无序表中包含有n-1个元素，排序过程中每次从无序表中取出第一个元素，将它插入到有序表中的适当位置，使之成为新的有序表，重复n-1次可完成排序过程。"
---


常见的内部排序算法有：冒泡排序、选择排序、插入排序、希尔排序、归并排序、快速排序、堆排序、基数排序等。这里主要介绍直接插入排序

排序算法总览：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/sort-sort.png)


## 直接插入排序

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/insertSort.png)


直接插入排序(Straight Insertion Sort)的基本思想是：把n个待排序的元素看成为一个有序表和一个无序表。开始时有序表中只包含1个元素，无序表中包含有n-1个元素，排序过程中每次从无序表中取出第一个元素，将它插入到有序表中的适当位置，使之成为新的有序表，重复n-1次可完成排序过程。


## 算法实现

```javascript
function straightInsertSort(arr){
  var len = arr.length;
  var temp;
  for(var i=1; i<len; i++){
    if(arr[i] < arr[i-1]){
      temp = arr[i];
      for(var j = i-1; j >= 0 && arr[j] > temp; j--){
        arr[j+1] = arr[j];
      }
      arr[j+1] = temp;
    }
  }
  return arr;
}

// 测试
var arr = [3, 2, 4, 9, 1, 5, 7, 6, 8];
var arrSorted = straightInsertSort(arr);
console.log(arrSorted);
// 控制台将输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
```