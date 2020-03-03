---
title: Express路由与中间件原理（路由篇）
date: 2019-07-20T21:53:54+08:00
categories: ["Nodejs"]
tags : ["Nodejs", "Express"]
toc: true
# featured_image : ""
keywords: ["Nodejs", "Express"]
description : "Express路由与中间件原理（路由篇）"
summary: "导入 `express` 后会得到一个 `express` 函数，执行这个函数后返回一个 `app` `函数，app` 上有一个 `listen` 函数，执行这个 `listen` 函数就会启一个 `http` 服务，通过 `app.get`、`app.post`、`app.all`等函数来注册监听函数，如果 `http server` 监听到有请求到来，就会调用在 `app` 上注册的相应的回调函数。"
---


## Express 路由与中间件介绍


### Express 路由


```js
let express = require('express')

let app = express();

app.get('/', (req, res) => {
  res.end('get home')
})

app.post('/', (req, res) => {
  res.end("post hemo")
})

app.all('/all', (req, res) => {
  res.end("all")
})

app.all('*', (req, res) => {
  res.end("all *")
})

app.listen(3000, () => {
  console.log("sever run port 3000")
})
```

导入 `express` 后会得到一个 `express` 函数，执行这个函数后返回一个 `app` `函数，app` 上有一个 `listen` 函数，执行这个 `listen` 函数就会启一个 `http` 服务，通过 `app.get`、`app.post`、`app.all`等函数来注册监听函数，如果 `http server` 监听到有请求到来，就会调用在 `app` 上注册的相应的回调函数。


在 express 内部有一个 routes 数组， `app.get`、`app.post`、`app.all`等函数会产生一个 `layer` 对象，并将 `layer` 对象存放在 `routes` 数组中，这个 `layer` 对象中包含了 `method`、`path`、`callback`，当请求到来时就会去遍历 `routes` 数组，如果该请求的 `method`、`path`与 `routes`数组中 `layer` 对象的`method`、`path`一致，则执行该 `layer` 中的 `callback` 方法

### 第一版
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

    for (let i = 0; i < app.routes.length; i++) {
      const layer = app.routes[i];
      if ((layer.method === method || "all" === layer.method) && (layer.path === pathname || ("*" === layer.path))) {
        return layer.callback(req, res)
      }
    }
    res.end(`cannot ${method}  ${pathname}`)
  }

  app.routes = []

  app.get = (path, callback) => {
    let layer = {
      method: 'get',
      path,
      callback
    }
    app.routes.push(layer)
  }

  app.post = (path, callback) => {
    let layer = {
      method: 'post',
      path,
      callback
    }
    app.routes.push(layer)
  }

  app.all = (path, callback) => {
    let layer = {
      method: 'all',
      path,
      callback
    }
    app.routes.push(layer)
  }

  app.listen = (...args) => {
    let server = http.createServer(app)
    server.listen(...args)
  }

  return app;
}

module.exports = application;

```

### 支持匹配路径参数
```js
let express = require('express')
// let express = require('./express')

let app = express();

app.get('/user/:id/:name', (req, res) => {
  console.log(req.params.id)
  console.log(req.params.name)
  res.end('ok')
})

app.listen(3000, () => {
  console.log("sever run port 3000")
})
```

如果路径中`/`后面以`:`开头，则表示为一个路径参数
以`/user/:id/:name`为例：
如果请求路径为`/user/123/luke`，那么 express 会取出`[id, name]`和`[123, luke]`组成一个`{id: 123, name: luke}`的对象，并将这个对象挂载在 `req.params`上

这里可以使用[path-to-regexp](https://www.npmjs.com/package/path-to-regexp)
具体的做法是先通过正则表达式将`/user/:id/:name`中的 id 和 name 取出放在一个数组中，然后将 `/user/:id/:name` 替换成 `/user/:([^\/]+)/:([^\/]+)`，让 `path = new RegEpx('/user/:([^\/]+)/:([^\/]+)')`，然后在匹配请求时，通过该正则表达式来进行匹配，并将组成的对象挂载在 `req.params` 上。

### 第二版

```js
// express.js
let http = require('http')
let url = require('url')

function application() {

  let app = (req, res) => {
    // 获取请求方法
    let method = req.method.toLowerCase();
    // 获取请求路径
    let { pathname } = url.parse(req.url)

    for (let i = 0; i < app.routes.length; i++) {
      const layer = app.routes[i];
      let layerPath = layer.path
      let layerMethod = layer.method
      if (layerPath.params) {
        console.log(layerPath.params)
        if ((layerMethod === method || "all" === layerMethod) && layerPath.test(pathname)) {
          // 过滤掉第一项
          let [, ...args] = pathname.match(layer.path)
          console.log(args)
          req.params = layerPath.params.reduce((memo, current, index) => (memo[current] = args[index], memo), {});
          console.log(req.params)
          return layer.callback(req, res);
        }
      }

      if ((layerMethod === method || "all" === layerMethod) && (layerPath === pathname || ("*" === layerPath))) {
        return layer.callback(req, res)
      }
    }
    res.end(`cannot ${method}  ${pathname}`)
  }

  app.routes = [];

  ['get', 'post', 'all'].forEach(method => {
    app[method] = (path, callback) => {
      if (path.includes(':')) {
        let params = []
        // /user/:id/:name -> /user/:([^\/]+)/:([^\/]+)
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
这一版实现了 `epress` 路由的功能，但是跟 `express` 的源码区别还是很大的，这里只是提供了一种简化的思路。并且没有涉及`express` 的中间件，下一篇[《Express路由与中间件原理（中间件篇）》](http://www.goyth.com/2019/07/20/expressMiddleware/)会添加 中间件的实现，并对这一版做出改进。