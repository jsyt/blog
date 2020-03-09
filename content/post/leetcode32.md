---
title: "Leetcode32"
date: 2020-03-09T22:51:54+08:00
tags: [""]
categories: [""]
description : ""
image: ""
---

### [32\. 最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)

Difficulty: **困难**


给定一个只包含 `'('` 和 `')'` 的字符串，找出最长的包含有效括号的子串的长度。

**示例 1:**

```
输入: "(()"
输出: 2
解释: 最长有效括号子串为 "()"
```

**示例 2:**

```
输入: ")()())"
输出: 4
解释: 最长有效括号子串为 "()()"
```


#### Solution

Language: **JavaScript**

#### 解法一——暴力求解（超时）

```javascript
​/**
 * @param {string} s
 * @return {number}
 */
var longestValidParentheses = function(s) {
    let maxLen = 0;
    function valid (i, j) {
        let stack = [];
        while (i < j) {
            if (s[i++] === '(') {
                stack.push(')')
            } else {
                if (stack.length === 0) return false;
                stack.pop()
            }
        }
        return stack.length === 0;
    }
    for (let i = 0; i < s.length; i++) {
        for (let j = i + 2; j <= s.length; j += 2) {
            if (valid(i, j)) {
                if (j - i > maxLen) {
                    maxLen = j - i
                }
            }
        }
    }
    return maxLen
};
```

#### 解法二——