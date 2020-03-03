---
title: N-Sum 问题
date: 2018-05-25T21:53:54+08:00
categories: ["Algorithm"]
tags : ["Algorithm"]
toc: true
# featured_image : ""
keywords: ["N-Sum", "Algorithm"]
description : "N-Sum 问题"
---


## 问题描述
给定一个包含多个整数且排好序的数组 nums 和一个目标值 target，判断 nums 中是否存在 N(N>1) 个元素，使得 N 个元素之和与 target 相等？找出所有满足条件且不重复的N元组。

### 解题思路

通过递归降幂将 N-Sum问题 降幂到 2-Sum 问题，然后采用两边加逼的办法求解


### JavaScript 版本

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @param {number} n
 * @param {number[]} result
 * @param {number[]} results 结果集
 */
function findNsum(nums, target, n, result, results) {
    if(n<2 || nums.length < n || target < nums[0] * n || target > nums[nums.length-1] * n) return ;
    if(n === 2){
        let l = 0,
            r = nums.length - 1;
        while(l < r){
            let s = nums[l] + nums[r];
            if(s == target){
                results.push(result.concat(nums[l], nums[r]));
                l++;
                r--;
                while(l < r && nums[l] == nums[l-1]){
                    l++;
                }
                while(l<r && nums[r] == nums[r+1]){
                    r--;
                }
            }else if(s < target){
                l++;
                while(l < r && nums[l] == nums[l-1]){
                    l++;
                }
            }else{
                r--;
                while(l<r && nums[r] == nums[r+1]){
                    r--;
                }
            }
        }
    }else{
        let len = nums.length - n + 1
        for(let i = 0 ; i < len; i++){
            if(i == 0 || ( i>0 && nums[i] != nums[i - 1])){
                findNsum(nums.slice(i+1), target - nums[i], n - 1, result.concat(nums[i]), results);
            }
        }

    }
}
```


### Python 版本

```python

def findNsum(nums, target, N, result, results):
    if len(nums) < N or N < 2 or target < nums[0]*N or target > nums[-1]*N:  # early termination
        return
    if N == 2: # two pointers solve sorted 2-sum problem
        l,r = 0,len(nums)-1
        while l < r:
            s = nums[l] + nums[r]
            if s == target:
                results.append(result + [nums[l], nums[r]])
                l += 1
                r -= 1
                while l < r and nums[l] == nums[l-1]:
                    l += 1
                while l < r and nums[r] == nums[r+1]:
                    r -= 1
            elif s < target:
                l += 1
                while l < r and nums[l] == nums[l-1]:
                    l += 1
            else:
                r -= 1
                while l < r and nums[r] == nums[r+1]:
                    r -= 1
    else: # recursively reduce N
        for i in range(len(nums)-N+1):
            if i == 0 or (i > 0 and nums[i-1] != nums[i]):
                findNsum(nums[i+1:], target-nums[i], N-1, result+[nums[i]], results)
```
