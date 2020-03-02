---
title: Express路由与中间件原理（中间件篇）
date: 2019-07-20T21:53:54+08:00
categories: ["Nodejs"]
tags : ["Nodejs", "Express"]
toc: true
featured_image : ""
keywords: ["Nodejs", "Express"]
description : "Express路由与中间件原理（中间件篇）"
summary: "`Express` 路由原理见 [Express路由与中间件原理（路由篇）](http://www.goyth.com/2019/07/20/expressRouter/)
`Express` 中间件通常用来一些公用的逻辑，并可以将处理的结果挂载在 `req`、`res` 上，以供后面的中间件函数，或路由函数使用。因此通常情况下中间件函数会放在路由的前面。在 `Express` 中，注册一个中间件与注册一个路由一样，也是放在 `app.routes` 中，只是中间件的 `method` 为 middle。"
---


## Express 中间件

`Express` 路由原理见 [Express路由与中间件原理（路由篇）](http://www.goyth.com/2019/07/20/expressRouter/)

`Express` 中间件通常用来一些公用的逻辑，并可以将处理的结果挂载在 `req`、`res` 上，以供后面的中间件函数，或路由函数使用。因此通常情况下中间件函数会放在路由的前面


```js
let express = require('express')
// let express = require('./express')

let app = express();

app.use('/', (req, res, next) => {
  console.log(1)
  next()
  console.log(2)
})

app.use('/', (req, res, next) => {
  console.log(3)
  next()
  console.log(4)
})

app.use('/user', (req, res, next) => {
  console.log(5)
  next()
})

app.use('/user', (req, res, next) => {
  console.log(6)
  next()
})

```

在 `Express` 中，注册一个中间件与注册一个路由一样，也是放在 `app.routes` 中，只是中间件的 `method` 为 middle。

但是在中间件函数中，有可能要执行异步逻辑，中间件函数中是通过调用 `next` 回调函数，来执行下一个中间件函数或路由，因此我们不能再通过循环来遍历 `app.routes` ，而是应该通过回调函数来进行调用

### 第三版

```js
// express.js
let http = require('http')
let url = require('url')

function application() {

  let app = (req, res) => {
    // 获取请求方法
    let method = req.method.toLowerCase();
    // 获取请求路径
    let {
      pathname
    } = url.parse(req.url)
    // 使用 next 回调函数来遍历 app.routes
    function next(index) {
      if (index >= app.routes.length) {
        return res.end(`cannot ${method}  ${pathname}`);
      }
      let layer = app.routes[index];
      // 中间件
      if (layer.method === 'middle') {
        let {
          path,
          callback
        } = layer;
        if (path === '/' || pathname.startsWith(path + '/') || pathname === path) {
          callback(req, res, () => {
            next(index + 1)
          });
        } else { // 如果不匹配则取下一个中间件或路由
          next(index + 1)
        }
      } else {  //  路由
        let layerPath = layer.path
        let layerMethod = layer.method
        if (layerPath.params && (layerMethod === method || "all" === layerMethod) && layerPath.test(pathname)) {
          // 过滤掉第一项
          let [, ...args] = pathname.match(layer.path)
          req.params = layerPath.params.reduce((memo, current, index) => (memo[current] = args[index], memo), {});
          return layer.callback(req, res);
        } else if ((layerMethod === method || "all" === layerMethod) && (layerPath === pathname || ("*" === layerPath))) {
          return layer.callback(req, res)
        } else { // 如果不匹配则取下一个中间件或路由
          next(index + 1)
        }
      }

    }

    next(0)

    res.end(`cannot ${method}  ${pathname}`)
  }
  // 存放中间件或路由的 layer 数组
  app.routes = [];
  // 中间件注册方法
  app.use = function (path, callback) {
    if (!callback) {
      callback = path;
      path = '/';
    }
    let layer = {
      method: 'middle',
      path,
      callback
    }
    app.routes.push(layer)
  }

  ;['get', 'post', 'all'].forEach(method => {
    app[method] = (path, callback) => {
      // /user/:id/:name -> /user/:([^\/]+)/:([^\/]+)
      if (path.includes(':')) {
        let params = []
        path = path.replace(/:([^\/]+)/g, (...args) => {
          params.push(args[1])
          return '([^\/]+)'
        })
        path = new RegExp(path);
        path.params = params
      }

      let layer = {
        method,
        path,
        callback
      }
      app.routes.push(layer)
    }
  });

  app.listen = (...args) => {
    let server = http.createServer(app)
    server.listen(...args)
  }

  return app;
}

module.exports = application;

```

## 总结
第三版实现了与`Express`相同的路由与中间件功能，但是与源码相比还是有一些区别的，这里只是提供了一个简化版的思路。`Express`源码考虑的东西很多，但是大体思想是类似的。总结一下这一版路由与中间件的实现思想就是维护一个队列，通过 `app.get` 、`app.post`、 `app.all`、`app.use` 来构建一个 `layer` 对象，这个 `layer` 对象中包含了，`method`、`path`、`callback` 函数，然后将 `layer` 对象存放在队列中。
当请求到来时，通过`next` 回调函数来遍历队列中的每一项，如果 `method` 与 `path` 都匹配的话，则执行该 `callback`。
