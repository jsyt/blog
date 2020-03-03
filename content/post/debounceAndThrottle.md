---
title: JavaScript 之函数防抖与节流
date: 2018-05-10T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript", "Debounce", "Throttle"]
toc: true
# featured_image : ""
keywords: ["JavaScript", "防抖", "节流"]
description : "JavaScript 之函数防抖与节流"
---


## 函数防抖（debounce)
函数防抖是指在函数调用动作触发n秒后才开始执行，n秒内若再次触发，则重新开始计时，再次等待n秒后才开始执行。如果n秒内不断触发，那就不断重新开始计时，一直等到有一个n秒内没有触发，才开始执行此函数。

根据描述，我们可以用`setTimeout`来实现一个简单版的防抖函数


### 第一版

```javascript
／**
* @ fn 回调函数
* @ delay 延迟时间
*／
function debounce(fn, delay){
    let timer = null;
    return function(){
        clearTimeout(timer);
        timer = setTimeout(fn, delay);
    }
}

```
<!-- mo-->

由于`setTimeout`的回调函数内的`this`是指向`window`，如果不传参数则`argument`对象为空，所以我们得修复`this`的指向，并将`argument`对象也传给回调函数

### 第二版

```javascript
／**
* @ fn 回调函数
* @ delay 延迟时间
*／
function debounce(fn, delay){
    let timer = null;
    return function(){
        let context = this,
            arg = arguments;
        clearTimeout(timer);
        timer = setTimeout(function(){
            fn.apply(context, arg);
        }, delay);
    }
}

```
现在我们新增一个立即执行的需求，就是第一次触发后就立即执行，然后再等待n秒后再执行，n秒内如果有触发则重新计时。我们新增一个参数immediate，true表示立即执行，false表示非立即执行

### 第三版

```javascript
／**
* @ fn 回调函数
* @ delay 延迟时间
* @ immediate 是否立即执行
*／
function debounce(fn, delay, immediate){
    let timer = null;
    return function(){
        let context = this,
            arg = arguments;
        if(timer){
            clearTimeout(timer);
        }
        if(immediate){
            if(!timer){
                fn.apply(context, arg);
            }
            timer = setTimeout(function(){
                timer = null;
            }, delay);
        }else{
            timer = setTimeout(function(){
                fn.apply(context, arg);
            }, delay);
        }
    }
}

```

### underscore 实现版本：

```javascript
_.debounce = function(func, wait, immediate) {
    var timeout, args, context, timestamp, result;
    var later = function() {
        var last = _.now() - timestamp;
        if (last < wait && last >= 0) {
            timeout = setTimeout(later, wait - last);
        } else {
            timeout = null;
            if (!immediate) {
                result = func.apply(context, args);
                if (!timeout) context = args = null;
            }
        }
    };
    return function() {
        context = this;
        args = arguments;
        timestamp = _.now();
        var callNow = immediate && !timeout;
        if (!timeout) timeout = setTimeout(later, wait);
        if (callNow) {
            result = func.apply(context, args);
            context = args = null;
        }
        return result;
    };
};
```

## 函数节流（throttle）
函数节流是指每隔n秒钟就执行一次事件，不管你在n秒内触发了多少次事件，都是每隔n秒才执行一次。

可以用定时器和时间戳两种方式实现

### 时间戳版本

```javascript
／**
* @ fn 回调函数
* @ wait 间隔时间
*／
function throttle(fn, wait){
    let pre = 0;
    return function(){
        let now = +new Data();
        let remain = now - pre;
        if(remain >= wait || remain <= 0 ){
            fn.apply(this, arguments);
            pre = now;
        }
    }
}

```
时间戳版本，第一次会立即触发并执行回调函数，但是最后一次触发如果是在最后一个n秒内发生的，则最后一次触发并不会执行回调函数

### 定时器版本

```javascript
／**
* @ fn 回调函数
* @ wait 间隔时间
*／
function throttle(fn, wait){
    let timer = null;
    return function(){
        let context = this,
            arg = arguments;
        if(!timer){
            timer = setTimeout(function(){
                fn.apply(context, arg);
                timer = null;
            }, wait);
        }
    }
}

```
定时器版本第一次触发后会在n秒后再执行回调函数，最后一次触发如果是在最后一个n秒内发生，则最后一次触发也会执行回调函数

我们可以结合两个版本的优点实现一个首次会立即执行，最后一次也会执行的版本

### 时间戳定时器混合版本

```javascript
／**
* @ fn 回调函数
* @ wait 间隔时间
*／
function throttle(fn, wait){
    let pre = 0，
        timer = null;
    return function(){
        let context = this,
            arg = arguments,
            now = +new Data(),
            remaining = wait - (now - pre);
        if((remaining < 0 || remaining >= wait)){
            if(!timer){
                fn.apply(this, arguments);
                pre = now;
            }
            timer = setTimeout(function(){
                pre = now;
                fn.apply(context, arg);
            }, wait);
        }else{
            clearTimeout(timer);
            timer = setTimeout(function(){
                pre = now;
                fn.apply(context, arg);
            }, remaining);
        }
    }
}

```

### underscore实现版本

```javascript
／**
* @ func 回调函数
* @ wait 间隔时间
* @ options options.leading = true 表示首次立即执行 options.leading = false 表示首次不立即执行 ；
* @     options.trailing = true 表示最后一次执行 options.trailing = false 表示最后一次不执行
*／
_.throttle = function(func, wait, options) {
    var context, args, result;
    var timeout = null;
    var previous = 0;
    if (!options) options = {};
    var later = function() {
        previous = options.leading === false ? 0 : _.now();
        timeout = null;
        result = func.apply(context, args);
        if (!timeout) context = args = null;
    };
    return function() {
        var now = _.now();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            result = func.apply(context, args);
            if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(later, remaining);
        }
        return result;
    };
};
```
underscore 的版本有一个很好的地方就是当事件频繁触发时不用一直设置定时器和清除定时器。但是这个版本有两个问题，第一个就是当设置`options.leading = false` 和 `options.trailing = false` 首次调用时 `remaining = wait` if 和 else if 分支都不会进去，这是一个bug；第二个问题就是当设置`options.leading = true` 和 `options.trailing = true` 首次调用时 `previous = 0` now 等于一个很大的正数，`remaining = wait - (now - 0) < 0` 数一个很大的负数，`timeout = null; !timeout = true` 进入 else if 分支的时候，执行`timeout = setTimeout(later, remaining);`的时候，给定时器设延迟执行，这应该

