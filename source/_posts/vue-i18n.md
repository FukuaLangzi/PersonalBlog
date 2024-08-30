---
title: vue-i18n
date: 2024-08-30 11:06:47
description: 在使用vue-i8n的过程中遇到的奇怪bug
tags:
  - vue
  - 国际化
  - bug
---

## 背景

在做一个要求国际化的外包项目，于是使用了 vue-i8n，开发环境时没有任何问题，线上部署时碰到了非常 nt 的 bug

## bug 复现

### 环境

pnpm vue3 vue-i18n@9^

### 创建项目

```powershell
pnpm create vite
```

创建完项目之后引入 vue-i8n

```powershell
pnpm install vue-i18n
```

注意，vue-i18n 的 8 版本对应 vue2，9 版本及以上对应 vue3

### 引入 i18n

```javascript
// main.js
import { createApp } from "vue";
import App from "./App.vue";
import { createI18n } from "vue-i18n";
import en from "./langurage/en";
import zh from "./langurage/zh";

// 组合语言包对象
const messages = {
  en,
  zh,
};

// 创建实例对象
const i18n = createI18n({
  legacy: false, // 设置为 false，启用 composition API 模式
  messages,
  locale: "en",
});

// 创建 Vue 实例
const app = createApp(App);

// 注册对象
app.use(i18n);

// 挂载到 Dom 元素中
app.mount("#app");
```

在组件中使用

```javascript
// App.vue
<script setup>
import { useI18n } from "vue-i18n";
import { onMounted } from "vue";
const { locale } = useI18n();

// 切换语言
const changeLocale = async (lang) => {
  localStorage.removeItem("language");
  await localStorage.setItem("language", lang);
  locale.value = lang;
};
onMounted(() => {
  locale.value = localStorage.getItem("language");
});
</script>

<template>
  <div>{{ $t("test.a") }}</div>
  <button @click.prevent="changeLocale('zh')">切换中文</button>
  <button @click.prevent="changeLocale('en')">切换英文</button>
</template>

<style scoped>
button {
  background-color: #fff;
  color: #000;
  margin: 10px;
}
</style>

```

这样处理之后，开发时可以正常运行，而且将本机语种在切换后持久化存储

### 接下来尝试打包

```powershell
npm run build
```

### 简单启个 express 当服务器测试一下

```powershell
express test
cd test
npm i
code .
```

打开项目之后把 public 下面的东西清空，把 dist 下的文件放进去，打开后成功报错，完成了 bug 复现

bug 图片如下

{% asset_img image.png %}

## bug 修复

### 先从报错看

其实看不出什么但是能注意到有个 Proxy 的报错，猜测可能和 vue 的响应式有关

再次检查代码之后觉得应该不是自己的问题，应该是用的某一个库的问题，因为这套系统里只有 vue-i8n 是第一次用，其他还有的就是动画库，不过应该关系不大，所以先试试把 i18n 的东西全部去掉

去掉之后运行成功，因为 i18n 的代码量不大，所以可以一步步排查

### 最终问题

问题出在这一句上

```javascript
// App.vue
onMounted(() => {
  locale.value = localStorage.getItem("language");
});
```

结合报错和出问题的代码，初步猜测是由于在 onmounted 调用的时候，locale 这个变量还没有初始化，导致值为 undefined，所以出现了报错，将取 localstorage 的部分转移到 main.js 中就好了

```javascript
// main.js
const i18n = createI18n({
  legacy: false, // 设置为false可以启用compositionAPI模式
  messages,
  locale: localStorage.getItem("language"), // 获取浏览器的语言
});
```
