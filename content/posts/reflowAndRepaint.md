---
title: 浏览器渲染之回流（Reflow）与重绘（Repaint）
date: 2018-05-25T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript", "Browser"]
toc: true
featured_image : ""
keywords: ["浏览器", "回流", "重绘", "Reflow", "Repaint"]
description : "浏览器渲染之回流（Reflow）与重绘（Repaint）"
summary: "回流（Reflow）和重绘（Repaint）的定义
回流（Reflow）
对于DOM结构中的各个元素都有自己的盒子（模型），这些都需要浏览器根据各种样式（浏览器的、开发人员定义的等）来计算，并根据计算结果将元素放到它该出现的位置，这个过程称之为reflow。
重绘（Repaint）
当各种盒子的位置、大小以及其他属性，例如颜色、字体大小等都确定下来后，浏览器于是便把这些元素都按照各自的特性绘制了一遍，于是页面的内容出现了，这个过程称之为 repaint。"
---


## 浏览器渲染流程
浏览器渲染流程如下图所示：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/reflowAndRepaint-webkitflow.png)

大概可以划分成以下几个步骤：

1. 通过HTML解析器解析HTML文本并构建DOM tree
2. 通过CSS解析器解析CSS样式表并构建CSSOM tree
3. 根据DOM tree 和 CSSOM tree 构建 Render tree
4. Render tree 刚构建完后是没有元素节点坐标、尺寸大小等信息的，此时需要通过Reflow(Layout)进行布局处理，计算出元素在屏幕上显示的位置，尺寸大小等信息。
5. 遍历渲染树，对每一个元素节点进行绘制（Painting）

回流（Reflow）与重绘（Repaint）就分别发生在第四步和第五步



## 回流（Reflow）和重绘（Repaint）的定义
### 回流（Reflow）
对于DOM结构中的各个元素都有自己的盒子（模型），这些都需要浏览器根据各种样式（浏览器的、开发人员定义的等）来计算，并根据计算结果将元素放到它该出现的位置，这个过程称之为reflow。
### 重绘（Repaint）
当各种盒子的位置、大小以及其他属性，例如颜色、字体大小等都确定下来后，浏览器于是便把这些元素都按照各自的特性绘制了一遍，于是页面的内容出现了，这个过程称之为 repaint。

> 回流（Reflow）和重绘（Repaint）会对性能产生一定的影响，尤其是当引发全局的回流和重绘时。

## 导致回流（Reflow）和重绘（Repaint）的操作
1. 调整窗口大小
2. 改变字体
3. 增加或者移除样式表
4. 内容变化，比如用户在input框中输入文字
5. 激活 CSS 伪类，比如 :hover (IE 中为兄弟结点伪类的激活)
6. 操作 class 属性
7. 脚本操作 DOM
8. 计算 offsetWidth 和 offsetHeight 属性
9. 设置 style 属性的值

## 如何尽量避免回流（Reflow）和重绘（Repaint）

1. 不要一条一条地修改 DOM 的样式。与其这样，还不如预先定义好 css 的 class，然后修改 DOM 的 className，即将多次改变样式属性的操作合并成一次操作：
```javascript
    // 不好的写法
    var left = 10,
    top = 10;
    el.style.left = left + "px";
    el.style.top  = top  + "px";
    el.style.background = '#eee';
    // 比较好的写法
    el.className += " theclassname";
```
2. 让要操作的元素进行”离线处理”，处理完后一起更新
- 使用documentFragment对象进行缓存操作,引发一次回流和重绘；
- 使用display:none技术，只引发两次回流和重绘。原理：由于display属性为none的元素不在渲染树中，对隐藏的元素操 作不会引发其他元素的重排。如果要对一个元素进行复杂的操作时，可以先隐藏它，操作完成后再显示。这样只在隐藏和显示时触发2次重排。
- 先克隆Dom节点(cloneNode) 修改完后，再用克隆的Dom节点将原来的节点替换掉，只引发一次回流和重绘；
3. 将需要多次重排的元素，position属性设为absolute或fixed，这样此元素就脱离了文档流，它的变化不会影响到其他元素为动画的 HTML 元素，例如动画，那么修改他们的 CSS 是会大大减小 reflow 。因为,它们不影响其他元素的布局，所它他们只会导致重新绘制，而不是一个完整回流。这样消耗会更低
4. 不要用tables布局的一个原因就是tables中某个元素一旦触发reflow就会导致table里所有的其它元素reflow。在适合用table的场合，可以设置table-layout为auto或fixed，这样可以让table一行一行的渲染，这种做法也是为了限制reflow的影响范围。
5. 尽可能的修改层级比较低的 DOM节点。当然，改变层级比较底的 DOM节点有可能会造成大面积的 reflow，但是也可能影响范围很小。
因为改变 DOM 树中的一级会导致所有层级的改变，上至根部，下至被改变节点的子节点。这导致大量时间耗费在执行 reflow 上面
6. 不要把 DOM 节点的属性值放在一个循环里当成循环里的变量。不然这会导致大量地读写这个结点的属性。
7. 避免使用CSS的JavaScript表达式，如果css里有expression，每次都会重新计算一遍。
