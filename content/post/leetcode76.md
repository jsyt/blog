---
title: "LeetCode 76. 最小覆盖子串"
date: 2020-03-09T09:53:57+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "LeetCode"]
description : ""
image: "https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/md-al-amin-nowshad-akjK1AarY8U-unsplash.jpg"
---

### [76\. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

Difficulty: **困难**


给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字母的最小子串。

**示例：**

```
输入: S = "ADOBECODEBANC", T = "ABC"
输出: "BANC"
```

**说明：**

*   如果 S 中不存这样的子串，则返回空字符串 `""`。
*   如果 S 中存在这样的子串，我们保证它是唯一的答案。

#### 解题思路

1. 要送`S`字符串中找出包含 `T` 所有字母的最小子串，那么首先就得记录`T`中有哪些字符，然后再去遍历 `S`，从`S`中寻找包含 `T` 所有字母的子串
2. 这里我们可以先用一个`map`，`needs`来记录`T`中的字符，以及字符的数量
3. 然后维护一个窗口，用索引`l`和`r`来表示这个窗口的左右边界，刚开始窗口的大小为0，即`l = 0`、`r = 0`
4. 然后开始遍历`S`，从窗口的右侧依次放入元素，也用一个`map`，` windows`来记录`S`中的字符及其字符的数量
5. 如果`windows[c1] === needs[c1]`，则说明窗口中有一个字符的数量与`T`中相等，则将计数器`count++`
6. 如果`count`等于`needs`中的`key`的数量和，则说明窗口中有`T`中所有的字符串，此时窗口所包含的子串就是一个包含 `T` 所有字母的子串
7. 由于答案是要寻找最小的字串，所以可以记录下符合要求的子串的起始位置以及其长度，起始位置就是`l`，长度为`r - l`
8. 找到符合要求的子串后，就开始从窗口的左侧移除字符，直到该子串不符合要求，根据将要移除的字符`c`，判断`windows[c] === needs[c]`，如果相等则要将则将计数器`count--`，然后移除该字符`windows[c]--`，最后将左边界索引`l++`
9. 重复上面的逻辑找出所有可能的子串，比较每一个子串的长度，最后返回最小的子串

#### Solution

Language: **JavaScript**

```javascript
​/**
 * @param {string} s
 * @param {string} t
 * @return {string}
 */
var minWindow = function(s, t) {
    let windows = {}, needs = {}, l = 0, r = 0, count = 0, start = -1, minLen = Infinity;
    [...t].forEach(c => needs[c] ? needs[c]++ : needs[c] = 1)
    let keyLen = Object.keys(needs).length;
    while (r < s.length) {
        let c1 = s[r++];
        windows[c1] ? windows[c1]++ : windows[c1] = 1;
        if (windows[c1] === needs[c1]) count++;
        while(count === keyLen) {
            if (r - l < minLen) {
                start = l;
                minLen = r - l;
            }
            let c2 = s[l++];
            if (windows[c2]-- === needs[c2]) count--;
        }
    }
    return start === -1 ? "" : s.substr(start, minLen)
};
```
