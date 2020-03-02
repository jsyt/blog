---
title: JavaScript之apply、call和bind的模拟实现
date: 2018-05-27T22:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript"]
toc: true
keywords: ["JavaScript", "apply", "call"]
description : "JavaScript apply、call和bind的模拟实现"
featured_image : ""
---


## apply()

apply 方法传入两个参数：一个是作为函数上下文的对象，另外一个是作为函数参数所组成的数组。当第一个参数为 `null` 时，函数上下文为 `window`。

```javascript
var obj = {
    name : 'luke'
}

function func(age, gender){
    console.log(this.name + ' ' + age + ' ' + gender);
}

func.apply(obj, [18, 'male']);    // luke 18 male
```



## apply模拟实现

```javascript
Function.prototype.apply2 = function(context, arrArgs){
  context = context || window;
  context.fn = this;
  let args = [];
  for(let i=0, len=arrArgs.length; i<len; i++){
    args.push('arrArgs['+i+']');
  }
  let result = eval('context.fn('+args+')');
  delete context.fn;
  return result;
}

// 测试一下
var obj = {
    name : 'luke'
}

function func(age, gender){
    console.log(this.name + ' ' + age + ' ' + gender);
}

func.apply2(obj, [18, 'male']);    // luke 18 male

```

## call()

call 方法第一个参数也是作为函数上下文的对象，但是后面传入的是一个参数列表，而不是单个数组。当第一个参数为 `null` 时，函数上下文也是 `window`。

```javascript
var obj = {
    name : 'luke'
}

function func(age, gender){
    console.log(this.name + ' ' + age + ' ' + gender);
}

func.call(obj, 18, 'male');    // luke 18 male
```


## call模拟实现

```javascript
Function.prototype.call2 = function(context){
  context = context || window;
  context.fn = this;
  let args = [];
  for(let i=1, len=arguments.length; i<len; i++){
    args.push('arguments['+i+']')
  }
  let result = eval('context.fn('+args+')');
  delete context.fn;
  return result;
}

// 测试一下
var obj = {
    name : 'luke'
}

function func(age, gender){
    console.log(this.name + ' ' + age + ' ' + gender);
}

func.call2(obj, 18, 'male');    // luke 18 male
```

## bind()
bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。

```javascript
var obj = {
    name : 'luke'
}

function func(age, gender){
    console.log(this.name + ' ' + age + ' ' + gender);
}

let bindfn = func.bind(obj, 18);
bindfn('male')    // luke 18 male

```

## bind模拟实现

```javascript
Function.prototype.bind2 = function(){
  let context = [].shift.call(arguments) || window;
  let args = [].slice.call(arguments);
  let self = this;
  function Bd(){
    return self.apply(this instanceof Bd ? this : context, args.concat([].slice.call(arguments)));
  }
  Bd.prototype = Object.create(self.prototype);
  return Bd;
}

// 测试一下
var obj = {
    name : 'luke'
}

function func(age, gender){
    console.log(this.name + ' ' + age + ' ' + gender);
}

let bindfn = func.bind2(obj, 18);
bindfn('male')    // luke 18 male

```