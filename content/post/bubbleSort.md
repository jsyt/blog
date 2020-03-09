---
title: 排序算法之冒泡排序
date: "2018-02-08T10:53:54+08:00"
categories: ["Algorithm"]
tags : ["Algorithm", "Sort"]
toc: true
# featured_image : ""
keywords: ["排序算法", "冒泡排序", "bubble sort"]
description : "排序算法之冒泡排序"
summary: "冒泡排序（Bubble Sort），是一种计算机科学领域的较简单的排序算法。
它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。
这个算法的名字由来是因为越大的元素会经由交换慢慢“浮”到数列的顶端，故名“冒泡排序”。"
---


排序算法总览：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/sort-sort.png)


## 冒泡排序

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/bubbleSort-bubbleSort.png)


冒泡排序（Bubble Sort），是一种计算机科学领域的较简单的排序算法。
它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。
这个算法的名字由来是因为越大的元素会经由交换慢慢“浮”到数列的顶端，故名“冒泡排序”。



## 算法原理编辑
冒泡排序算法的运作如下：（从前往后）
比较相邻的元素。如果第一个比第二个大，就交换他们两个。
对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
针对所有的元素重复以上的步骤，除了最后一个。
持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

## 算法实现

### JavaScript

```javascript
function bubbleSort(arr){
  for(let i = arr.length-1; i>0; i--){
    for(let j=0; j<i; j++){
      if(arr[j] > arr[j+1]){
        [arr[j], arr[j+1]] = [arr[j+1], arr[j]]    // 利用es6解构语法进行swap
      }
    }
  }
  return arr;
}

// 测试
var arr = [3, 2, 4, 9, 1, 5, 7, 6, 8];
var arrSorted = bubbleSort(arr);
console.log(arrSorted);
// 控制台将输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## 冒泡算法优化

```javascript
function bubbleSort(arr){
  let flag = true;
  for(let i=arr.length-1; flag && i>0; i--){
    flag = false;    //只要flag在下一次外循环条件检测的时候值为false，就说明已经排好序，不用继续循环
    for(let j=0; j<i; j++){
      if(arr[j] > arr[j+1]){
        flag = true;    //如果有交换，就将标记变量赋true
        [arr[j], arr[j+1]] = [arr[j+1], arr[j]]    // 利用es6解构语法进行swap
      }
    }
  }
  return arr;
}

// 测试
var arr = [3, 2, 4, 9, 1, 5, 7, 6, 8];
var arrSorted = bubbleSort(arr);
console.log(arrSorted);
// 控制台将输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
```
