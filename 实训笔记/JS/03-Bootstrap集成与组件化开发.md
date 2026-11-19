---
title: Bootstrap 集成与组件化开发
tags:
  - 前端
  - Vue
  - 实训
  - 工程化
  - 组件
---

## 1.Bootstrap集成
### 1.1核心依赖安装
安装Bootstrap、jQuery和Popper.js
```npm
npm install bootstrap jquery popper.js
```
### 1.2 修改main.js
```js
在main.js中导入，bootstrap与对应的css 
import 'bootstrap/dist/css/bootstrap.css'

import 'bootstrap/dist/js/bootstrap.bundle.min.js'
```
### 1.3修改vue.config.js
加入以下代码
```js
const { defineConfig } = require('@vue/cli-service')

module.exports = defineConfig({

  transpileDependencies: true,

  lintOnSave: false

})
```

## 2. 组件化(类比与Java的类)
### 2.1创建组件
组件可以把页面拆成独立、可复用的部分。拆分后，每部分职责更明确，样式和逻辑更容易维护。
```vue
<script setup>
const props = defineProps({
  name: {
    type: String,
    required: true,
  },
  age: {
    type: Number,
    default: 0,
  },
});
</script>

<template>
  <article class="user-card">
    <h2>{{ props.name }}</h2>
    <p>年龄：{{ props.age }}</p>
  </article>
</template>

<style scoped>
.user-card {
  padding: 16px;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
}
</style
```

### 2.2使用组件

父组件中导入后即可在模板中使用：

```vue
<script setup>
import UserCard from "./components/UserCard.vue";
</script>

<template>
  <UserCard name="张三" :age="18" />
</template>
```

使用 `<script setup>` 时，导入的组件会自动暴露给模板，不需要再写 `components` 注册选项。
### 2.3 Props

- Props 是父组件传给子组件的数据。
- Props 应保持单向流动。
- 子组件不能直接修改 Props。
- 需要修改时，应通过事件通知父组件修改源数据。

```vue
<script setup>
const props = defineProps({
  title: String,
  count: {
    type: Number,
    default: 0,
  },
});
</script>
```
常用类型：

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Function`

## 相关笔记

- [[实训笔记/Vue/01-创建Vue项目与目录结构|创建 Vue 项目与目录结构]] — 装依赖前的项目准备
- [[实训笔记/Vue/02-基础语法|Vue 基础语法]] — 组件里的模板语法
- [[01-组件基础|组件基础（主库）]] — props、组件注册与复用
- [[01-Vue Router 与 Pinia 入门|Vue Router 与 Pinia 入门（主库）]] — 工程化后续
- [[01-jQuery 表单验证|jQuery 表单验证（主库）]] — 本笔记装的 jQuery 的实际用法
