---
title: vue-plugins
date: 2024-08-08 16:52:04
tags: vue, 工程化
description: vue插件初探
---

# vue 插件

### 介绍

插件 (Plugins) 是一种能为 Vue 添加全局功能的工具代码

```javascript
import { createApp } from "vue";
import App from "./App.vue";
const app = createApp(App);

app.use(myPlugin, {
  // 配置项
});
app.mount("#app");
```

其中，myPlugin 是我们自定义的插件对象

### 插件对象的规范

一个插件可以是一个拥有 install() 方法的对象，也可以直接是一个安装函数本身。安装函数会接收到安装他的应用实例和传递给 app.use()的额外选项作为参数：

```javascript
const myPlugin = {
  install(app, options) {
    // 配置此应用
  },
};
```

插件没有严格定义的使用范围，但是插件发挥作用的常见场景主要包括以下几种

- 通过 app.component() 和 app.directive() 注册一到多个全局组件或自定义指令。

- 通过 app.provide() 使一个资源可被注入进整个应用。

- 向 app.config.globalProperties 中添加一些全局实例属性或方法

- 一个可能上述三种都包含了的功能库 (例如 vue-router)。

### 插件代码示例

代码文件

```typescript
import type { App } from "vue";
import Button from "你的自定义组件地址";
const plugins = {
  install: (app: App) => {
    app.config.globalProperties.$echo = (name: string) => {
      return `hello ${name}`;
    };
    app.component("wrButton", Button);
    // test为injectKey
    app.provide("test", { message: "from plugin" });
  },
};
export default plugins;
```

使用示例

```javascript
// main.js
app.use(plugins);
```

```
// index.vue
{{$echo('echo in index')}}
<wrButton>test</wrButton>
console.log('inject('test')')
```

但是在使用自定义的全局属性时，$echo 会报错，这是因为我们没有声明他的类型

### 扩展全局属性

某些插件会通过 app.config.globalProperties 为所有组件都安装全局可用的属性。举例来说，我们可能为了请求数据而安装了 this.$http，或者为了国际化而安装了 this.$translate。为了使 TypeScript 更好地支持这个行为，Vue 暴露了一个被设计为可以通过 TypeScript 模块扩展来扩展的 ComponentCustomProperties 接口：

```typescript
import axios from "axios";

declare module "vue" {
  interface ComponentCustomProperties {
    $http: typeof axios;
    $translate: (key: string) => string;
  }
}
```

### 类型扩展的位置

我们可以将这些类型扩展放在一个 .ts 文件，或是一个影响整个项目的 \*.d.ts 文件中。无论哪一种，都应确保在 tsconfig.json 中包括了此文件。对于库或插件作者，这个文件应该在 package.json 的 types 属性中被列出。

为了利用模块扩展的优势，你需要确保将扩展的模块放在 TypeScript 模块 中。 也就是说，该文件需要包含至少一个顶级的 import 或 export，即使它只是 export {}。如果扩展被放在模块之外，它将覆盖原始类型，而不是扩展!

```typescript
// 不工作，将覆盖原始类型。
declare module "vue" {
  interface ComponentCustomProperties {
    $translate: (key: string) => string;
  }
}
```

```typescript
// 正常工作。
export {};

declare module "vue" {
  interface ComponentCustomProperties {
    $translate: (key: string) => string;
  }
}
```

上面是文档写的，自己操作的话代码长这样（图方便写一起了）

```typescript
// ts文件，必须有导出或者导入
import type { App } from "vue";
const plugins = {
  install: (app: App) => {
    app.config.globalProperties.$echo = (name: string) => {
      return `hello ${name}`;
    };
    // test为injectKey
    app.provide("test", { message: "from plugin" });
  },
};
declare module "vue" {
  interface ComponentCustomProperties {
    $echo: (name: string) => string;
  }
}
export default plugins;
```
