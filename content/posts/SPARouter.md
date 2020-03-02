---
title: 单页面（SPA）路由实现原理
date: 2019-05-25T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript", "Router"]
toc: true
featured_image : ""
keywords: ["单页面路由", "SPA", "路由"]
description : "单页面（SPA）路由实现原理"
summary: "什么是单页面应用 ?
单页面应用（SPA）即single page application，目前在前后端分离的项目中，一般都是采用 SPA 的模式，整个应用只有一个 html 页面。后端接口只负责提供数据，而页面路由则需要前端自己完成。单页面应用的优势：1. 减少 http 请求数，降低服务器压力；2. 有利于前后端分离；3. 页面流畅度更高，用户体验更加友好。"
---
 
## 什么是单页面应用 ?

单页面应用（SPA）即single page application，目前在前后端分离的项目中，一般都是采用 SPA 的模式，整个应用只有一个 html 页面。后端接口只负责提供数据，而页面路由则需要前端自己完成。

## 单页面应用的优势

### 1. 减少 http 请求数，降低服务器压力
单页面应用在切换页面时，路由由前端路由系统完成，不需要向服务器发送请求，服务器也不用再返回一个 html 页面。
### 2. 有利于前后端分离
使用单页面应用后，后端不需要再配置路由接口，只用提供访问数据的接口，这样使得前后端工作解耦，开发效率更高。
### 3. 页面流畅度更高，用户体验更加友好
单页面路由切换时，页面通过本地 js 计算生成，页面切换更快更流畅。



## 单页面应用的缺点

由于单页面应用的内容都是由 js 计算生成，所以不利于 SEO，不过这可以通过 prerender、服务端渲染等技术解决。

## 如何实现单页面路由

单页面路由在切换页面或者说 URL 改变时，并没有刷新页面，而是通过 js 计算，将对应 URL路径需要展示的组件显示在页面上。所以要实现单页面路由，需要做到以下两点：
1. 如何在改变 URL 时，不引起页面的刷新
2. URL 改变时，如何监听 URL 的变化，并对页面进行改变

目前实现单页面路由有两种方法，分别是通过 url hash 和 h5 history api，下面介绍下如何通过这两种方法实现单页面路由。

### HashRouter
当 URL 中的 hash 发生改变时，页面不会刷新，我们可以用 hashchange 来监听 hash 的改变。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
   <a href="#/a">跳转到/a</a>
   <a href="#/b">跳转到/b</a>
   <div id="root" style="border:3px solid red;height:200px"></div>
<script>
let container = document.getElementById('root');
window.addEventListener('hashchange',(event)=>{
  //console.log(event);
  //container.innerHTML = event.newURL;
  container.innerHTML = `当前的路径是 ${window.location.hash.slice(1)}`;
});

</script>
</body>
</html>

```


### HistoryRouter
HTML5 中引入了 history.pushState() 和 history.replaceState()方法，这个两个方法分别可以添加和修改 URL 栈中当前的记录，可以修改当前 URL 中的 path 部分，并且不会引起页面的刷新。
通过 window.onpopstate 函数可以监听 URL 栈中的回退事件，当通过点击事件进行跳转时，可以统一使用 history.pushstate 来进行跳转，这样通过重写 history.pushstate 就可以监听到

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<div id="root" style="border:3px solid red;height:200px">  </div>
<button onclick="push('/a')">/a</button>
<button onclick="push('/b')">/b</button>
<button onclick="push('/c')">/c</button>
<script>
let container = document.getElementById('root');
//监听弹出状态的事件
window.onpopstate = function(event){
  //console.log(event);
  container.innerHTML= event.state.to;
}
window.onpushstate = function(state,title,url){
  //console.log(event);
  container.innerHTML= state.to||url;
}
let oldPush = window.history.pushState;
window.history.pushState = function(state,title,url){
    oldPush.call(window.history,state,title,url);
    window.onpushstate(state,title,url);
}

function push(to){
  console.log('pushstate!')
   //pushState 三个参数 新的状态对象 标题(已经废弃) to 跳转到的路径
   window.history.pushState({to},null,to);
}

</script>
</body>
</html>

```
