---
title: 排序算法之归并排序
date: 2018-02-09T21:53:54+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "Sort"]
toc: true
# featured_image : ""
keywords: ["排序算法", "归并排序", "merge sort"]
description : "排序算法之归并排序"
summary: "归并排序（MERGE-SORT）是建立在归并操作上的一种有效的排序算法,该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为二路归并。

归并排序可以使用递归和迭代两种方式进行实现"
---



排序算法总览：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/sort-sort.png)


## 归并排序

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/mergeSort-mergesort.gif)



归并排序（MERGE-SORT）是建立在归并操作上的一种有效的排序算法,该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为二路归并。

归并排序可以使用递归和迭代两种方式进行实现


### 递归法（Top-down）
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
4. 重复步骤3直到某一指针到达序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾

### 迭代法（Bottom-up）
原理如下（假设序列共有n个元素）：

1. 将序列每相邻两个数字进行归并操作，形成 ceil(n/2)个序列，排序后每个序列包含两/一个元素
2. 若此时序列数不是1个则将上述序列再次归并，形成 ceil(n/4)个序列，每个序列包含四/三个元素
3. 重复步骤2，直到所有元素排序完毕，即序列数为1

### JavaScrpt 递归版

```js
function mergeSort(arr){
  var len = arr.length;
  if(len <= 1){
    return arr;
  }
  var mid = Math.floor(len/2)
  var left = arr.slice(0, mid);
  var right = arr.slice(mid);

  return merge(mergeSort(left), mergeSort(right))

}

function merge(left, right){
  var result = [];
  while(left.length > 0 && right.length > 0){
    if(left[0] < right[0]){
      result.push(left.shift())
    }else{
      result.push(right.shift())
    }
  }
  return result.concat(left).concat(right);
}
```
### JavaScrpt 迭代版

```js
function mergeSort(arr){
  var len = arr.length;
  var result = [];
  for(var block=1; block < len; block = 2 * block){
    for(var start = 0; start < len; start = start + 2 * block){
      var low = start;
      var mid = (start + block) > len ? len : (start + block)
      var high = (start + 2 * block) > len ? len : (start + 2 * block)
      var start1 = low, end1 = mid;
      var start2 = mid, end2 = high;
      while(start1 < end1 && start2 < end2){
        if(arr[start1] < arr[start2]){
          result[low++] = arr[start1++]
        }else{
          result[low++] = arr[start2++]
        }
      }
      while(start1 < end1){
        result[low++] = arr[start1++]
      }
      while(start2 < end2){
        result[low++] = arr[start2++]
      }
    }
    arr = result;
    result = [];    // 这里一定要将 result 设置为一个新的空数组，否则下一次循环时，修改result的同时也会修改arr
  }
  return result;
}

```