---
title: Webpack 工作原理（三）
date: 2019-01-12T21:53:54+08:00
categories: ["Nodejs"]
tags : ["Nodejs", "webpack"]
toc: true
# featured_image : ""
keywords: ["Webpack", "Nodejs"]
description : "Webpack 工作原理（三）—— 实现原理及实现一个简单版"
summary: "webpack Loader 是一个符合 `commonjs` 规范的模块，这个模块导出一个函数，它的主要作用是对源码进行转换，webpack 在调用 Loader时，会将源代码作为参数传递给这个Loader，然后该loader会对源码进行转换，并且返回转换后的内容。"
---
## 什么是 webpack Loader
webpack Loader 是一个符合 `commonjs` 规范的模块，这个模块导出一个函数，它的主要作用是对源码进行转换，webpack 在调用 Loader时，会将源代码作为参数传递给这个Loader，然后该loader会对源码进行转换，并且返回转换后的内容。


一个最简单的 Loader 的源码如下：
```js
module.exports = function(source) {
  // source 为 compiler 传递给 Loader 的一个文件的原内容
  // 该函数需要返回处理后的内容，这里简单起见，直接把原内容返回了，相当于该 Loader 没有做任何转换
  return source;
};
```

### loader 执行顺序

**webpack.config.js**
```js
module.exports = {
  //...
  module: {
    rules: [
      {
        //...
        use: [
          'a-loader',
          'b-loader',
          'c-loader'
        ]
      }
    ]
  }
};
```

loader 的执行顺序是从后往前的，例如顺序配置`'a-loader'`,`'b-loader'`,`'c-loader'`，则它们的执行顺序是`'c-loader'` --> `'b-loader'` --> `'a-loader'`，`'c-loader'`接受 webpack 传递进来的原始数据，进行相应的转换，`'b-loader'` 接受 'c-loader' 转换都的结果进行进一步转换，`'a-loader'` 接受 `'b-loader'` 转换的结果，进行进一步的转换，产生最终转换都的源码

### 处理二进制数据
在默认的情况下，Webpack 传给 Loader 的原内容都是 `UTF-8` 格式编码的字符串。
但有些场景下 Loader 不是处理文本文件，而是处理二进制文件，例如 `file-loader`，就需要 Webpack 给 Loader 传入二进制格式的数据。
为此，你需要这样编写 Loader：
```js
module.exports = function(source) {
    // 在 exports.raw === true 时，Webpack 传给 Loader 的 source 是 Buffer 类型的
    source instanceof Buffer === true;
    // Loader 返回的类型也可以是 Buffer 类型的
    // 在 exports.raw !== true 时，Loader 也可以返回 Buffer 类型的结果
    return source;
};
// 通过 exports.raw 属性告诉 Webpack 该 Loader 是否需要二进制数据
module.exports.raw = true;
```
以上代码中最关键的代码是最后一行 `module.exports.raw = true;`，没有该行 Loader 只能拿到字符串。

## webpack loader API 介绍
webpack在 loader 的 `this` 上挂在了很多有用的属性或方法，后面会有详细介绍。

### this.callback()
在 loader 中如果是采用 `return` 的方式，那么默认只会返回一个值，如果你需要返回多个值可以采用`this.callback`

例如以用 babel-loader 转换 ES6 代码为例，它还需要输出转换后的 ES5 代码对应的 Source Map，以方便调试源码。
为了把 Source Map 也一起随着 ES5 代码返回给 Webpack，可以这样写：

```js
module.exports = function(source) {
  // 通过 this.callback 告诉 Webpack 返回的结果
  this.callback(null, source, sourceMaps);
  // 当你使用 this.callback 返回内容时，该 Loader 必须返回 undefined，
  // 以让 Webpack 知道该 Loader 返回的结果在 this.callback 中，而不是 return 中
  return;
};
```
其中的 `this.callback` 是 Webpack 给 Loader 注入的 API，以方便 Loader 和 Webpack 之间通信。
`this.callback` 的详细使用方法如下：
```js
this.callback(
    // 当无法转换原内容时，给 Webpack 返回一个 Error
    err: Error | null,
    // 原内容转换后的内容
    content: string | Buffer,
    // 用于把转换后的内容得出原内容的 Source Map，方便调试
    sourceMap?: SourceMap,
    // 如果本次转换为原内容生成了 AST 语法树，可以把这个 AST 返回，
    // 以方便之后需要 AST 的 Loader 复用该 AST，以避免重复生成 AST，提升性能
    abstractSyntaxTree?: AST
);
```
Source Map 的生成很耗时，通常在开发环境下才会生成 Source Map，其它环境下不用生成，以加速构建。
为此 Webpack 为 Loader 提供了 this.sourceMap API 去告诉 Loader 当前构建环境下用户是否需要 Source Map。
如果你编写的 Loader 会生成 Source Map，请考虑到这点。


### this.async()
Loader 有同步和异步之分，上面介绍的 Loader 都是同步的 Loader，因为它们的转换流程都是同步的，转换完成后再返回结果。
但在有些场景下转换的步骤只能是异步完成的，例如你需要通过网络请求才能得出结果，如果采用同步的方式网络请求就会阻塞整个构建，导致构建非常缓慢。

在转换步骤是异步时，你可以这样：
```js
module.exports = function(source) {
    // 告诉 Webpack 本次转换是异步的，Loader 会在 callback 中回调结果
    var callback = this.async();
    someAsyncOperation(source, function(err, result, sourceMaps, ast) {
        // 通过 callback 返回异步执行后的结果
        callback(err, result, sourceMaps, ast);
    });
};
```

### this.cacheable()

在有些情况下，有些转换操作需要大量计算非常耗时，如果每次构建都重新执行重复的转换操作，构建将会变得非常缓慢。
为此，Webpack 会默认缓存所有 Loader 的处理结果，也就是说在需要被处理的文件或者其依赖的文件没有发生变化时，
是不会重新调用对应的 Loader 去执行转换操作的。

如果你想让 Webpack 不缓存该 Loader 的处理结果，可以这样：
```js
module.exports = function(source) {
  // 关闭该 Loader 的缓存功能
  this.cacheable(false);
  return source;
};
```

### this.context
当前处理文件的所在目录，假如当前 Loader 处理的文件是 `/src/main.js`，则 `this.context` 就等于 `/src`。
### this.resource
当前处理文件的完整请求路径，包括 querystring，例如 `/src/main.js?name=1`。
### this.resourcePath
当前处理文件的路径，例如 `/src/main.js`。
### this.resourceQuery
当前处理文件的 querystring。
### this.target
等于 Webpack 配置中的 Target
### this.loadModule
但 Loader 在处理一个文件时，如果依赖其它文件的处理结果才能得出当前文件的结果时，就可以通过 `this.loadModule(request: string, callback: function(err, source, sourceMap, module))` 去获得 request 对应文件的处理结果。

### this.resolve
像 require 语句一样获得指定文件的完整路径，使用方法为 `resolve(context: string, request: string, callback: function(err, result: string))`。
### this.addDependency
给当前处理文件添加其依赖的文件，以便再其依赖的文件发生变化时，会重新调用 Loader 处理该文件。使用方法为 `addDependency(file: string)`。
### this.addContextDependency
和 `addDependency` 类似，但 `addContextDependency` 是把整个目录加入到当前正在处理文件的依赖中。使用方法为 `addContextDependency(directory: string)`。
### this.clearDependencies
清除当前正在处理文件的所有依赖，使用方法为 `clearDependencies()`。
### this.emitFile
输出一个文件，使用方法为 `emitFile(name: string, content: Buffer|string, sourceMap: {...})`。


## 编写loader

### loader 编写准则
1. loader 应该遵循单一职责原则，一个loader只完成一个功能，如果需要多步转换，则应该编写多个loader进行链式调用。
2. 不要在模块代码中插入绝对路径，因为当项目根路径变化时，文件绝对路径也会变化。`loader-utils` 中的 `stringifyRequest` 方法，可以将绝对路径转化为相对路径。
3. 如果一个 loader 使用外部资源（例如，从文件系统读取），必须声明它。这些信息用于使缓存 loaders 无效，以及在观察模式(watch mode)下重编译。


### loader 工具库
#### loader-utils
[loader-utils](https://github.com/webpack/loader-utils) 包。它提供了许多有用的工具，但最常用的一种工具是获取传递给 loader 的选项

#### schema-utils
[schema-utils](https://github.com/webpack-contrib/schema-utils) 包配合 loader-utils，用于保证 loader 选项，进行与 JSON Schema 结构一致的校验

### babel-loader

```js
const babel=require('babel-core');
const path=require('path');

module.exports=function (source) {
    const options = {
        presets: ['env'],
        sourceMap: true,
        filename:this.request.split('/').pop()
    }
    let result=babel.transform(source,options);
    return this.callback(null,result.code,result.map);
}
```

### BannerLoader

给所有的文件添加同一的版权注释，可以接受一个文本，或者一个文件。

**banner-loader.js**

```js
const loaderUtils = require('loader-utils');
const validateOptions = require('schema-utils');
const fs = require('fs');
function loader(source) {
    //把loader改为异步,任务完成后需要手工执行callback
    let cb = this.async();
    //启用loader缓存
    this.cacheable && this.cacheable();
    //用来验证options的合法性
    let schema = {
        type: 'object',
        properties: {
            filename: {
                type: 'string'
            },
            text: {
                type: 'string'
            }
        }
    }
    //通过工具方法获取options
    let options = loaderUtils.getOptions(this);
    //用来验证options的合法性
    validateOptions(schema, options, 'Banner-Loader');
    let { text, filename } = options;
    if (text) {
        cb(null, text + source);
    } else if (filename) {
        fs.readFile(filename, 'utf8', (err, text) => {
            cb(err, text + source);
        });
    }
}

module.exports = loader;
```

**banner.js**
```js
/* © 2019 GOYTH All Rights Reserved.  */
```

**webpack.config.js**
```js
module.exports = {
  //...
  resolveLoader: {
    alias: {
      "babel-loader": resolve('./build/banner-loader.js')
    }
  },
  module: {
    rules: [
      {
        test:/.+\.js$/
        use: [
          {
            loader: "babel-loader",
            options:{filename:"./src/loaders/banner.js"}
//          options: {text: "© 2019 GOYTH All Rights Reserved. "}
          }
        ]
      }
    ]
  }
};
```

*参考链接*
*https://webpack.js.org/api/loaders/*
*https://github.com/webpack/loader-utils*
*https://github.com/webpack-contrib/schema-utils*
*https://segmentfault.com/a/1190000012718374*
