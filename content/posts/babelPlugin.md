---
title: 编写一个 Babel 插件
date: 2019-01-06T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript", "Babel"]
toc: true
featured_image : ""
keywords: ["Babel"]
description : "编写一个 Babel 插件"
---
## Babel转译流程

Babel 对源码进行转译时，主要有三个步骤
![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/babel.png)

1. 首先通过`Babylon` 将源码转化成 AST
2. 然后再通过`babel-traverse`遍历 AST，找到需要更改的 AST 节点，对其进行修改
3. 根据修改后的 AST，通过`babel-generator`将修改后的 AST重新生成源码
Babel插件主要是处理第二步。

### Babylon
Babylon 是 Babel 的解析器，主要负责将源码转化成 AST。
```js
import * as babylon from "babylon";

const code = `function square(n) {
  return n * n;
}`;

babylon.parse(code);
// Node {
//   type: "File",
//   start: 0,
//   end: 38,
//   loc: SourceLocation {...},
//   program: Node {...},
//   comments: [],
//   tokens: [...]
// }
```
我们还能像下面这样传递选项给 parse()方法：
```js
babylon.parse(code, {
  sourceType: "module", // default: "script"
  plugins: ["jsx"] // default: []
});
```
sourceType 可以是 "module" 或者 "script"，它表示 Babylon 应该用哪种模式来解析。 "module" 将会在严格模式下解析并且允许模块定义，"script" 则不会。

*注意： sourceType 的默认值是 "script" 并且在发现 import 或 export 时产生错误。 使用 scourceType: "module" 来避免这些错误。*

### babel-traverse
Babel Traverse（遍历）模块维护了整棵AST树的状态，并且负责替换、移除和添加AST节点。

我们可以和 Babylon 一起使用来遍历和更新节点：
```js
import * as babylon from "babylon";
import traverse from "babel-traverse";

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

traverse(ast, {
  enter(path) {
    if (
      path.node.type === "Identifier" &&
      path.node.name === "n"
    ) {
      path.node.name = "x";
    }
  }
});
```

### babel-types
Babel Types模块是一个针对于 AST 节点的工具库，它包含了构造、验证以及变换 AST 节点的方法。 该工具库包含考虑周到的工具方法，对编写处理AST逻辑非常有用。

可以运行以下命令来安装它：
```js
import traverse from "babel-traverse";
import * as t from "babel-types";

traverse(ast, {
  enter(path) {
    if (t.isIdentifier(path.node, { name: "n" })) {
      path.node.name = "x";
    }
  }
});
```

### babel-generator
Babel Generator模块是 Babel 的代码生成器，它读取AST并将其转换为代码和源码映射（sourcemaps）。
```js
import * as babylon from "babylon";
import generate from "babel-generator";

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

generate(ast, {}, code);
// {
//   code: "...",
//   map: "..."
// }
```
你也可以给 generate() 方法传递选项。.
```js
generate(ast, {
  retainLines: false,
  compact: "auto",
  concise: false,
  quotes: "double",
  // ...
}, code);
```

## Babel 插件编写

Babel 插件是一个接收了当前babel对象作为参数的函数
```js
export default function(babel) {
  // plugin contents
}
```
我们可以直接取解构出 babel.types
```js
export default function({ types: t }) {
  // plugin contents
}
```
接着返回一个对象，其 visitor 属性是这个插件的主要访问者。
```js
export default function({ types: t }) {
  return {
    visitor: {
      // visitor contents
    }
  };
};
```
Visitor 中的每个函数接收2个参数：path 和 state
```js
export default function({ types: t }) {
  return {
    visitor: {
      Identifier(path, state) {},
      ASTNodeTypeHere(path, state) {}
    }
  };
};
```
### Babel 插件编写思路
1. Babel 插件主要是对 AST 进行转换
2. 通过[astexplorer.net](https://astexplorer.net/)来比较转换前和转换后的 AST 的差异。
3. 通过插件visitor模式遍历需要修改的 AST 节点
4. 通过[babel-types](https://babeljs.io/docs/en/next/babel-types.html)来判断AST节点类型或者修改、构建新的AST节点
5. 通过path.replaceWith()或者path.replaceWithMultiple()替换掉原来的节点


### 箭头函数插件
```js
const square = n => n * n;
```
我们写一个插件将上面的箭头函数转化成下面的普通函数
```js
const square = function (n) {
  return n * n;
}
```
Babel 插件主要是对 AST 进行转换
首先我们可以通过[astexplorer.net](https://astexplorer.net/)来观察一下，转换前和转换后的 AST 的差异。
转换前：
![AST](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/ast01.png)


转换后：
![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/ast02.png)

比较两个AST 后，我们知道要将ArrowFunctionExpression替换成FunctionExpression，BinaryExpression替换成BlockStatement，那我们就要构建一个FunctionExpression节点，如何构建FunctionExpression节点呢，我们可以在[babel-types](https://babeljs.io/docs/en/next/babel-types.html)查找构建方法

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/fe.png)


```js
let babel = require('@babel/core')
let t = require('babel-types')

let transformArrayFunctions = {
  visitor: {
    ArrowFunctionExpression: (path, state)=>{
      let node = path.node;
      let id = path.parent.id;
      let params = node.params;
      let body = t.blockStatement([t.returnStatement(node.body)])
      let functionExpression = t.functionExpression(id,params,body,false,false);
      path.replaceWith(functionExpression);
    }
  }
}
const result = babel.transform(code, {
    plugins: [transformArrowFunctions]
});
console.log(result.code);


```
