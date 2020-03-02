---
title: Babel 核心模块介绍
date: 2019-01-05T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript", "Babel"]
toc: true
featured_image : ""
keywords: ["Babel"]
description : "Babel 核心模块介绍"
---
## Babel 介绍
Babel 是一个通用的多用途 JavaScript 编译器。通过 Babel 你可以使用（并创建）下一代的 JavaScript，以及下一代的 JavaScript 工具。
Babel 把用最新标准编写的 JavaScript 代码向下编译成可以在今天随处可用的版本。 这一过程叫做“源码到源码”编译， 也被称为转换编译（transpiling，是一个自造合成词，即转换＋编译。以下也简称为转译）。



例如，Babel 能够将新的 ES2015 箭头函数语法：
```js
const square = n => n * n;
```
转译为：
```js
const square = function square(n) {
  return n * n;
};
```

## Babel核心模块介绍

### Babel-core
可以看做 babel 的编译器。babel 的核心 api 都在这里面。它可以把 js 代码，抽象成 AST，然后修改 AST ，再根据AST生成新的js代码，这就是Babel 转译的过程。但是如何转译、转译哪些内容，着需要我们事先设置好的 plugin，plugin 就是如何转译的规则。

### babel-register
它在底层改写了node的require方法，引入babel-register之后所有require并以.es6, .es, .jsx 和 .js为后缀的模块都会经过babel的转译。

### babel-polyfill

它同样是引用了 `core-js` 和 `regenerator`，垫片支持是一样的。`babel-polyfill` 是为了模拟一个完整的ES2015 +环境，旨在用于应用程序而不是库/工具。
它会让我们程序的执行环境，模拟成完美支持 es6+ 的环境，毕竟无论是浏览器环境还是 node 环境对 `es6+` 的支持都不一样。它是以重载全局变量 （E.g: `Promise`）,还有原型和类上的静态方法（E.g：`Array.prototype.reduce/Array.form`），从而达到对 `es6+` 的支持。不同于 `babel-runtime` 的是，`babel-polyfill` 是一次性引入你的项目中的，就像是 React 包一样，同项目代码一起编译到生产环境。

## plugins
要说 plugins 就不得不提 babel 编译的过程。babel 编译分为三步：

1. parser：通过 babylon 解析成 AST。
2. transform[s]：All the plugins/presets ，进一步的做语法等自定义的转译，仍然是 AST。
3. generator： 最后通过 babel-generator 生成  output string。

所以 plugins 是在第二步加强转译的，所以假如我们自己写个 plugin，应该就是对 ast 结构做一个遍历，操作。

### babel-runtime

这个包引用了 `core-js` 和 `regenerator`，那么什么是 core-js 和 regenerator 呢。
首先我们要知道上面提到的 babel-core 是对语法进行 transform 的，但是它不支持 build-ints（Eg: `promise`，`Set`，`Map`），prototype function（Eg: `array.reduce`,`string.trim`），`class` `static function` （Eg：`Array.form`，`Object.assgin`），regenerator （Eg：`generator`，`async`）等等拓展的编译。所以才要用到 core-js 和 regenerator。
#### core-js
core-js 是用于 JavaScript 的组合式标准化库，它包含 es5 （e.g: object.freeze）, es6的 promise，symbols, collections, iterators, typed arrays， es7+提案等等的 polyfills 实现。也就是说，它几乎包含了所有 JavaScript 最新标准的垫片。但是不包含 generator。

#### regenerator
regenerator 实现了 generator/yeild， async/await。

### babel-plugin-transform-runtime
transform-runtime 是为了方便使用 babel-runtime 的，它会分析我们的 AST 中，是否有引用 babel-rumtime 中的垫片（通过映射关系），如果有，就会在当前模块顶部插入我们需要的垫片。

## transform-runtime 对比 babel-polyfill
其实通过上面的介绍我们已经了解他们是干什么的了，这里再稍微总结区分一下吧。我在这里把 babel-runtime 和 babel-plugin-transform-runtime 统称为 transform-runtime，因为一起用才比较好。

babel-polyfill 是当前环境注入这些 es6+ 标准的垫片，好处是引用一次，不再担心兼容，而且它就是全局下的包，代码的任何地方都可以使用。缺点也很明显，它可能会污染原生的一些方法而把原生的方法重写。如果当前项目已经有一个 polyfill 的包了，那你只能保留其一。而且一次性引入这么一个包，会大大增加体积。如果你只是用几个特性，就没必要了，如果你是开发较大的应用，而且会频繁使用新特性并考虑兼容，那就直接引入吧。
transform-runtime 是利用 plugin 自动识别并替换代码中的新特性，你不需要再引入，只需要装好 babel-runtime 和 配好 plugin 就可以了。好处是按需替换，检测到你需要哪个，就引入哪个 polyfill，如果只用了一部分，打包完的文件体积对比 babel-polyfill 会小很多。而且 transform-runtime 不会污染原生的对象，方法，也不会对其他 polyfill 产生影响。所以 transform-runtime 的方式更适合开发工具包，库，一方面是体积够小，另一方面是用户（开发者）不会因为引用了我们的工具，包而污染了全局的原生方法，产生副作用，还是应该留给用户自己去选择。缺点是随着应用的增大，相同的 polyfill 每个模块都要做重复的工作（检测，替换），虽然 polyfill 只是引用，编译效率不够高效。

## presets
显然这样一个一个配置插件会非常的麻烦，为了方便，babel为我们提供了一个配置项叫做persets（预设）。

预设就是一系列插件的集合，

### babel-preset-env
它能根据当前的运行环境，自动确定你需要的 plugins 和 polyfills。通过各个 es标准 feature 在不同浏览器以及 node 版本的支持情况，再去维护一个 feature 跟 plugins 之间的映射关系，最终确定需要的 plugins。

#### preset-env 配置
```js
{
  "presets": [
    [
      "env",
      {
        "targets": { // 配支持的环境
          "browsers": [ // 浏览器
            "last 2 versions",
            "safari >= 7"
          ],
          "node": "current"
        },
        "modules": true,  //设置ES6 模块转译的模块格式 默认是 commonjs
        "debug": true, // debug，编译的时候 console
        "useBuiltIns": false, // 是否开启自动支持 polyfill
        "include": [], // 总是启用哪些 plugins
        "exclude": []  // 强制不启用哪些 plugins，用来防止某些插件被启用
      }
    ]
  ],
  plugins: [
    "transform-react-jsx" //如果是需要支持 jsx 这个东西要单独装一下。
  ]
}
```

