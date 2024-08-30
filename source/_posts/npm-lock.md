---
title: npm锁版本
date: 2024-08-13 14:10:35
tags:
  - 工程化
description: 关于锁定依赖版本的那些事
---

# 版本号解析

分为三部分

例如：antd-mobile: ^5.8.1-alpha.3

其中， 三个数字依次为主版本号（major）、次版本号（minor）、小版本号（patch）、版本号标签。

以上的版本号书写，都遵循 Semver 规范

# Semver 规范

Major：新增不可向后兼容的新功能

Minor：新增可向后兼容的新功能

Patch：修复 bug
简单来说就是：主版本一般是大更新，比如从 vue2 -> vue3 这种，次版本就是加个功能之类的，大体上变化不大。

# 常用的版本号标签

latest（默认）、alpha（内测）、beta（公测）、next（下一个）、rc（候选）、experimental（实验）

# 版本前的符号

符号^: 主版本号固定，其他版本号更新到最新版（也就是说次版本号和小版本号都是会更新到最新的）

符号~: 主次版本号固定，小版本号更新到最新版

无符号：固定版本号

# 如何锁定版本

答：npm lock，从 npm 5 开始，当执行 npm install 时，会自动生成 package-lock.json 文件

# 作用

npm lock 文件（如 package-lock.json）的作用是确保在不同机器上或在不同时间安装相同的依赖包时，获得相同的版本，以避免由于版本不一致而产生的问题。在安装依赖包时，npm lock 文件会锁定当前的依赖树，并记录每个依赖包的确切版本号和依赖关系。在之后重新安装时会根据 lock 文件进行以来解析和安装。

# lock 文件是如何生成的

生成 npm-lock 文件的原理如下：

- 在第一次 npm i 时，npm 会先检查 package.json 文件，并根据依赖包信息生成 node_modules 文件夹存储依赖

- 在生成依赖文件夹时，npm 会生成一个 npm-shrinkwrap.json 或者 package-lock 文件，用来记录精确版本信息和依赖关系，且 npm-shrinkwrap 的优先级更高

- 下次安装时会优先检查这两个文件，如果有就按照这两个来，没有才会从新从 package.json 来解析依赖树

# npm-shrinkwrap.json 文件

在作用上他与 package-lock 文件的功能相同，但是区别在于，npm-shrinkwrap.json 可以锁定所有的依赖包版本，包括间接依赖的版本，而 package-lock.json 文件只会锁定直接依赖包的版本

同时，在使用 npm-shrinkwrap 文件时，如果要升级依赖包的版本，需要手动更新 npm-shrinkwrap 文件中的版本号。

# 使用 npm-shrinkwrap.json

在根目录下：

```powershell
npm shrinkwrap
```

如果需要在安装新的包的同时更新 npm-shrinkwrap 文件：

```powershell
npm shrinkwrap --dev
```
