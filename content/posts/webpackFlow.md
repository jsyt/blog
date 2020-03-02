---
title: Webpack 工作原理（二）——打包构建流程分析
date: 2018-12-10T21:53:54+08:00
categories: ["Nodejs"]
tags : ["Nodejs", "webpack"]
toc: true
featured_image : ""
keywords: ["Webpack", "Nodejs"]
description : "Webpack 工作原理（二）——打包构建流程分析"
summary: "webpack 主要工作流程
Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：
**初始化参数**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- **开始编译**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的run方法开始执行编译； 确定入口：根据配置中的 entry 找出所有的入口文件
- **编译模块**：从入口文件出发，调用所有配置的 Loader 对模块进行编译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；"
---

## 如何调试 webpack

require(加载) node_modules/webpack-cli/bin 目录下的cli.js，预先设置debugger（断点），然后开启调试模式。

### debugger.js
```js
var webpackPath=require('path').resolve(__dirname,'node_modules', 'webpack-cli', 'bin', 'cli.js');
require(webpackPath);
```

### webpack.config.js
```js
const path=require('path');
module.exports={
    mode:"development",
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname,'dist'),
        filename:'bundle.js'
    }
}
```

## webpack 主要工作流程
Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- **初始化参数**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- **开始编译**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的run方法开始执行编译； 确定入口：根据配置中的 entry 找出所有的入口文件
- **编译模块**：从入口文件出发，调用所有配置的 Loader 对模块进行编译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
- **完成模块编译**：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
- **输出资源**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
- **输出完成**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。



## webpack打包流程图
![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/webpackFlow-webpack2.svg)


## 流程详解

### 初始化阶段
|事件名	|解释|	代码位置|
| :------| :------ | :------ |
|读取命令行参数&ensp;|	从命令行中读取用户输入的参数|	require("./convert-argv")(argv)|
|实例化 Compiler	|1.用上一步得到的参数初始Compiler 实例 <br>2.Compiler 负责文件监听和启动编译 <br>3.Compiler 实例中包含了完整的<br> Webpack 配置，全局只有一个 Compiler 实例。|compiler = webpack(options);
|加载插件|	1.依次调用插件的 apply 方法，<br>让插件可以监听后续的所有事件节点。<br>同时给插件传入 compiler 实例的引用，<br>以方便插件通过 compiler 调用 <br>Webpack 提供的 API。|	plugin.apply(compiler)|
|处理入口	|读取配置的 Entrys，为每个 Entry 实例化<br>一个对应的 EntryPlugin，为后面该 <br>Entry 的递归解析工作做准备|	new EntryOptionPlugin().apply(compiler) <br>new SingleEntryPlugin(context, item, name) <br>compiler.hooks.make.tapAsync|
### 编译阶段
|事件名	|解释	|代码位置|
| :------| :------ | :------ |
|run|	启动一次新的编译|	this.hooks.run.callAsync|
|compile|	该事件是为了告诉插件一次新的编译将要启动，同时会给插件传入compiler 对象。|	compile(callback)|
|compilation	|当 Webpack 以开发模式运行时，每当检测到文件变化，一次新的 Compilation 将被创建。一个 Compilation 对象包含了当前的模块资源、编译生成资源、变化的文件等。Compilation 对象也提供了很多事件回调供插件做扩展。|	newCompilation(params)|
|make	|一个新的 Compilation 创建完毕主开始编译	|this.hooks.make.callAsync|
|addEntry	|即将从 Entry 开始读取文件	|compilation.addEntry <br> this._addModuleChain|
|moduleFactory	|创建模块工厂	|const moduleFactory = this.dependencyFactories.get(Dep)|
|create	|创建模块|	moduleFactory.create|
|factory	|开始创建模块|	factory(result, (err, module) <br> resolver(result) this.hooks.resolver.tap("NormalModuleFactory")|
|resolveRequestArray|	解析loader路径	|resolveRequestArray|
|resolve	|解析资源文件路径	|resolve|
|userRequest	|得到包括loader在内的资源文件的绝对路径用!拼起来的字符串	|userRequest|
|ruleSet.exec	|它可以根据模块路径名，匹配出模块所需的loader|	this.ruleSet.exec|
|_run	|它可以根据模块路径名，匹配出模块所需的loader	|_run|
|loaders|	得到所有的loader数组	|results[0].concat(loaders, results[1], results[2])|
|getParser	|获取AST解析器	|this.getParser(type, settings.parser)|
|buildModule|	开始编译模块|	this.buildModule(module) <br>buildModule(module, optional, origin,dependencies, thisCallback)|
|build|	开始真正编译入口模块|	build(options)|
|doBuild	|开始真正编译入口模块	|doBuild|
|执行loader	|使用loader进行转换|	runLoaders <br> runLoaders|
|iteratePitchingLoaders	|开始递归执行pitch loader|	iteratePitchingLoaders|
|loadLoader	|加载loader	|loadLoader|
|runSyncOrAsync	|执行pitchLoader|	runSyncOrAsync|
|processResource	|开始处理资源|	processResource<br>options.readResource<br>iterateNormalLoaders<br>iterateNormalLoaders|
|createSource	|创建源代码对象	|this.createSource|
|parse	|使用parser转换抽象语法树	|this.parser.parse|
|parse|	解析抽象语法树	|parse(source, initialState)|
|acorn.parse|	解析语法树|	acorn.parse(code, parserOptions)|
|ImportDependency	|遍历并添加添加依赖	|parser.state.module.addDependency(clearDep)|
|succeedModule|	生成语法树后就表示一个模块编译完成|	this.hooks.succeedModule.call(module)|
|processModuleDependencies|	递归编译依赖的模块|	this.processModuleDependencies(module)<br>processModuleDependencies(module, callback)<br>this.addModuleDependencies<br>buildModule|
|make后	|结束make	|this.hooks.make.callAsync(compilation, err => {}|
|finish	|编译完成	|compilation.finish();|
### 结束阶段
|事件名	|解释	|代码|
| :------| :------ | :------ |
|seal	|封装	|compilation.seal <br> seal(callback)|
|addChunk	|生成资源|	addChunk(name)|
|createChunkAssets	|创建资源|	this.createChunkAssets()|
|getRenderManifest	|获得要渲染的描述文件|	getRenderManifest(options)|
|render	|渲染源码|	source = fileManifest.render();|
|afterCompile	|编译结束|	this.hooks.afterCompile|
|shouldEmit	|所有需要输出的文件已经生成好，询问插件哪些文件需要输出，哪些不需要。|this.hooks.shouldEmit|
|emit	|确定好要输出哪些文件后，执行文件输出，可以在这里获取和修改输出内容。|	this.emitAssets(compilation) <br>this.hooks.emit.callAsync <br>const emitFiles = err<br>this.outputFileSystem.writeFile|
|this.emitRecords	|写入记录|	this.emitRecords|
|done	|全部完成|	this.hooks.done.callAsync|
