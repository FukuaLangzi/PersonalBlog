---
title: bundle
date: 2024-08-08 15:24:36
tags: 工程化 bundle
description: bundle的一些小知识
---

# 诞生的原因

官方支持的 import 和 export 这种同步加载方式目前在大多数浏览器中无法使用
详情见 mdn 文档的浏览器兼容性部分
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules#%E6%A8%A1%E5%9D%97%E5%8C%96%E7%9A%84%E8%83%8C%E6%99%AF

# Bundler 是什么

打包工具，将浏览器不支持的模块进行编译，转换，合并最后生成的代码可以在浏览器端良好的运行的工具

# 有哪些打包工具

### Webpack

Webpack 是一个功能强大且广泛使用的打包工具，支持多种资源类型的打包，如 JavaScript、CSS、图片等。它具有代码拆分、代码压缩、热模块替换等开箱即用的功能，适合构建高性能的 Web 应用。Webpack 的生态系统庞大，插件丰富，但配置相对复杂，构建速度可能较慢。

### Rollup

Rollup 是一个专注于 ES6 模块的打包工具，特别适用于构建 JavaScript 库和组件。它支持 Tree Shaking，可以自动剔除未使用的代码，减少打包后的文件体积。Rollup 原生支持 ES 模块，但处理 CommonJS 和 UMD 模块时可能需要复杂配置。

### Vite

Vite 是一个新兴的前端构建工具，它在开发环境下使用原生 ES 模块导入（ESM）和高效的开发服务器，实现快速的冷启动和热更新。Vite 在生产环境下则使用 Rollup 进行打包，支持 Tree Shaking 和 ESM。Vite 的配置简单，启动速度快，但社区活跃度和生态丰富度相比 Webpack 还有一定的差距。

# 上面提到了一些模块定义规范，顺便介绍一些模块化的历史方便理解

## 1. 模块是什么

```javascript
   import ... from ...
   export default ...
```

## 2. 模块化有哪些优点，为什么要模块化

- 可维护性
- 可复用性

## 3. 在 ES6 之前没有模块的年代是怎么做的

```javascript
// 使用古老的 backbone.js
<script src="todos.js"></script>
// 通过标签引入的方式
```

## 4. 为了解决没有模块的问题，开发者提出了一系列规范和约定来实现模块化的特征，大致分为了三个阶段

### 一阶段，称为全局变量 + 命名空间（namespace）

一个项目中基于同一个全局变量，各个模块按照自己的命名空间进行挂载，jquery 等较早时期的项目就使用了这种做法。

```javascript
// IIFE 自执行函数，创建一个封闭的作用域，赋值给一个全局变量
var namesCollection = (function () {
  // private members
  var objects = [];
  // public method
  function addObject(object) {
    object.push(object);
  }
  return {
    addName: addObject,
  };
})();
namesCollection.addName("viking");
```

缺点：

- 依赖全局变量，污染全局作用域，不安全
- 依赖约定命名空间来避免冲突，可靠性不高
- 需要手动管理依赖并控制执行顺序，很容易出错
- 需要最终上线前手动合并所有用到的模块

### 二阶段，Common.js

nodejs 推出的时候，开发出了自己的模块化系统，称为 commonJS

```javascript
const bar = require("./bar");
module.exports = function () {};
```

- 只是为服务器端使用的，不符合浏览器的标准，没法在浏览器里直接运行

### 三阶段，AMD（Asynchronous module definition）

在 commonJS 推出之后，前端工程师们想有一种在浏览器中可以用的模块化方法，于是产生了一种类似 commonjs 的格式，称之为 AMD

- 采用异步方式加载模块
- 只需要在全局环境中定义 require 和 define，不需要其他的全局变量
- 通过文件路径或模块自己声明的模块名来定位模块
- 提供了打包工具自动分析依赖并合并
- 配合特定的 AMD 加载器使用，RequireJS
- 同时还诞生了很多类似的模块标准 CMD(阿里发明的，加载器是 seajs)
  AMD 和 CMD 当时是竞争关系
  define(function(require) {
  // 通过相对路径获取依赖模块
  const bar = require('./bar')
  // 模块产出
  return function() {
  }
  })

## 5. 最终，JS 官方终于出手，提供了当前公认的模块化标准 ES6 modules

```javascript
import bar from "./bar";
export default function () {}
```

- 引入和暴露的方式更加多样
- 同时支持复杂的静态分析（treeShaking）
- 和 commonjs 一样，部分浏览器不支持这种语法
- 许多最新版的浏览器目前支持了这种语法，所以 vite 采用了两种方式，将生产环境和开发环境分开，开发环境使用 ESmodule，生产环境会使用 Bundler 进行打包

# 于是，为了代码能兼容各种浏览器，Bundler 应运而生，话题回到我们刚才提的打包工具

他们的本质功能就是处理这些 ES6 的模块代码，生成浏览器可以识别的代码

# 注：UMD、AMD、CMD 的关系

## 1. AMD(Asynchronous Module Definition)

- AMD 是 RequireJS 提出的模块定义方式，主要用于浏览器环境。
- 它支持异步加载模块，即在需要时才加载模块，有助于提高页面加载性能。
- AMD 模块使用 define 函数来定义模块，并通过 require 函数来异步加载依赖。

## 2. CMD(Common Module Definition)

- CMD 是 Sea.js 提出的模块定义方式，同样主要用于浏览器环境。
- 它与 AMD 类似，支持异步加载，但定义模块的语法不同。
- CMD 模块使用 define 函数定义模块，但依赖列表和模块实现是分开的，且依赖是同步加载的。

## 3. UMD(Universal Module Definition)

- UMD 是为解决多环境兼容性问题而设计的，它兼容 AMD、CommonJS（Node.js 的模块系统）以及作为全局变量使用。
- UMD 通过条件判断来支持 AMD 和 CommonJS 模块定义方式，同时也可以作为普通的全局变量在脚本标签中使用。
- 因此，UMD 可以看作是 AMD 和 CommonJS 的一种“桥接”，使得模块可以在不同的 JavaScript 环境中无缝工作。

## 简单来说，UMD 包括 AMD 和 Commonjs，是他们两个的超集。
