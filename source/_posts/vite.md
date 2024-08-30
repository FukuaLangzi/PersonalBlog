---
title: vite
date: 2024-08-08 15:45:38
tags:
  - 工程化
  - vite
description: vite的一些特点和原理
---

# Vite 的工作原理

### vite 解决了传统的 webpack 等 bundler 的性能瓶颈问题，第一次运行的太慢，而 vite 利用浏览器的新特性解决了这个问题

### 开发环境

在开发环境中，vite 实际上用最新的 ESmodule 的方式来提供源码，实际上是让浏览器接管了打包程序的部分工作

- 传统的 Bundle 方式

先打包，再启动开发环境，时间主要花在打包上

- vite 的 Bundlle 方式

一开始服务就启动，动态加载

### 但是 vite 并不是什么都没做，在预处理阶段，将代码分成了两部分

- 依赖

  - 使用 esbuild 进行预构建，esbuild 由 go 编写，速度较快。
    - 处理 Commonjs 以及 UMD 类型文件的兼容性，转换为最新的 ESM 以及 ESM 的导入形式。
    - 将本来需要多个模块的合并成一个模块统一请求，减少网络请求
    - 缓存功能，将预构建的依赖项存储在 node_modules/.vite 中

- 源码

  - 包含一些非标准文件，比如 JSX/CSS/VUE

### 生产环境

使用 rollup 代替 esbuild

为什么？

esbuild 和 Rollup 都是现代 JavaScript 模块打包工具，但它们在设计目标和实现方式上存在一些区别：

1. **性能**：esbuild 用 go 编写，编译速度快，能够在编译阶段就将源码转译为机器码，相比其他打包工具，速度可以快 100 倍 +。相比之下，Rollup 虽然不是用 Go 语言编写，但在打包 ES6 模块和提供 tree shaking 方面表现出色，生成的打包文件体积小且干净。
2. **用途**：Rollup 更适合用于库的开发，特别是那些专注于 ES6 模块的库，它原生支持 tree shaking，并且插件 API 友好。而 esbuild 由于其快速的构建速度，适用于任何需要快速构建的场景，包括日常开发和 CI/CD 环境。
3. **模块支持**：Rollup 原生支持 ES6 模块，对 CommonJS 的支持需要依赖插件，而 esbuild 支持多种模块格式，包括 CommonJS、ES6 模块、AMD 等。
4. **配置和生态**：Rollup 提供了丰富的插件 hooks 和灵活的定制能力，适合需要复杂配置的项目，而 esbuild 的配置相对简单，插件系统不如 Rollup 成熟。
5. **开发体验**：Rollup 在开发库时可能更受青睐，因为它可以提供更多的定制和优化，而 esbuild 则在需要快速迭代和构建的项目中表现出色。
6. **社区和生态**：Rollup 作为较早出现的打包工具，拥有成熟的社区和丰富的插件生态，而 esbuild 虽然出现较晚，但因其高性能特点，正在迅速获得社区的关注和应用。

Vite 的选用原因：

vite 目前的插件 API 与使用 esbuild 作为打包器并不兼容，尽管 esbuild 速度更快，但是 Vite 更青睐于 Rollup 灵活的插件 API 和基础建设。
