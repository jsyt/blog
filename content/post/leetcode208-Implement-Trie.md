---
title: "LeetCode 208. 实现 Trie (前缀树)"
date: 2020-02-15T12:51:39+08:00
categories: ["Algorithm"]
tags : ["Algorithm", "LeetCode"]
image: "https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/joseph-barrientos-Ji_G7Bu1MoM-unsplash.jpg"
---

### [208\. 实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

Difficulty: **中等**


实现一个 Trie (前缀树)，包含 `insert`, `search`, 和 `startsWith` 这三个操作。

**示例:**

```
Trie trie = new Trie();

trie.insert("apple");
trie.search("apple");   // 返回 true
trie.search("app");     // 返回 false
trie.startsWith("app"); // 返回 true
trie.insert("app");
trie.search("app");     // 返回 true
```

**说明:**

*   你可以假设所有的输入都是由小写字母 `a-z` 构成的。
*   保证所有输入均为非空字符串。

#### Trie 基本结构
- 字典树，即 Trie 树，又称单词查找树或键树，是一种树形结构。典型应用是用于统计和排序大量的字符串(但不仅限于 字符串)，所以经常被搜索引擎系统用于文本词频统计。
- 它的优点是:最大限度地减少 无谓的字符串比较，查询效率 比哈希表高。

#### Trie 基本性质
- 结点本身不存完整单词;
- 从根结点到某一结点，路径上经过的字符连接起来，为该结点对应的 字符串;
- 每个结点的所有子结点路径代表的字符都不相同。

#### Trie 核心思想

- Trie 树的核心思想是空间换时间。
- 利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。

#### Solution

Language: **JavaScript**

```javascript
​/**
 * Initialize your data structure here.
 */
var Trie = function() {
    this.root = new TrieNode();
};

function TrieNode () {
    this.next = {};
    this.isEnd = false;
}

/**
 * Inserts a word into the trie.
 * @param {string} word
 * @return {void}
 */
Trie.prototype.insert = function(word) {
    if (!word) return false;
    let node = this.root;
    for (let c of word) {
        if (!node.next[c]) {
            node.next[c] = new TrieNode();
        }
        node = node.next[c];
    }
    node.isEnd = true;
};

/**
 * Returns if the word is in the trie.
 * @param {string} word
 * @return {boolean}
 */
Trie.prototype.search = function(word) {
    if (!word) return false;
    let node = this.root;
    for (let c of word) {
        if (!node.next[c]) return false;
        node = node.next[c];
    }
    return node.isEnd;
};

/**
 * Returns if there is any word in the trie that starts with the given prefix.
 * @param {string} prefix
 * @return {boolean}
 */
Trie.prototype.startsWith = function(prefix) {
    if (!prefix) return false;
    let node = this.root;
    for (let c of prefix) {
        if (!node.next[c]) return false;
        node = node.next[c]
    }
    return true;
};

/**
 * Your Trie object will be instantiated and called as such:
 * var obj = new Trie()
 * obj.insert(word)
 * var param_2 = obj.search(word)
 * var param_3 = obj.startsWith(prefix)
 */
```
