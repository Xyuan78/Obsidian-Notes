---
title: Vue 基础总览
tags:
  - 前端
  - Vue
  - 基础
---

# Vue 基础总览

> [!abstract] 一句话理解
> Vue 是一个用于构建用户界面的渐进式 JavaScript 框架，重点解决“数据变化后，界面如何自动更新”的问题。

## Vue 的核心特点

- 声明式渲染：描述数据与界面之间的关系，不手动操作每个 DOM。
- 响应式系统：数据变化后，依赖该数据的视图会自动重新渲染。
- 组件化：把页面拆成可复用、可维护的组件。
- 渐进式：可以从一个小组件开始，也可以扩展成完整单页应用。
- 单文件组件：HTML、JavaScript 和 CSS 可以组织在同一个 `.vue` 文件中。

## 最小示例

```vue
<script setup>
import { ref } from "vue";

const count = ref(0);

function increase() {
  count.value++;
}
</script>

<template>
  <button type="button" @click="increase">
    点击次数：{{ count }}
  </button>
</template>

<style scoped>
button {
  padding: 8px 14px;
  border-radius: 8px;
}
</style>
```

执行过程：

1. `ref(0)` 创建响应式数据。
2. 模板读取 `count`，建立依赖关系。
3. 点击按钮后执行 `increase()`。
4. `count.value` 变化，Vue 自动更新相关界面。

## 单文件组件

一个 `.vue` 文件通常包含：

```vue
<script setup>
// JavaScript 逻辑
</script>

<template>
  <!-- 页面结构 -->
</template>

<style scoped>
/* 当前组件的样式 */
</style>
```

- `<script setup>`：组合式 API 的语法糖，变量和函数可以直接在模板中使用。
- `<template>`：组件模板。
- `<style scoped>`：样式默认只作用于当前组件。
- 一个组件可以有多个 `<style>`，但通常保留一个主样式块。

## Vue 2 与 Vue 3

- Vue 2 主要使用 Options API，通过 `data`、`methods`、`computed` 等选项组织代码。
- Vue 3 推荐 Composition API，通过 `ref`、`reactive`、`computed` 等函数组织逻辑。
- 新项目应优先选择 Vue 3，旧 Vue 2 项目仍可从 Options API 开始阅读。
- 两种 API 都能解决问题，重点是团队内保持风格一致。

## 适合使用 Vue 的场景

- 需要频繁根据数据更新页面的后台系统。
- 有大量可复用 UI 组件的项目。
- 单页应用、管理后台、表单系统、数据看板。
- 需要 Vue Router、Pinia 等完整生态的中大型前端项目。

## 常见误区

- 把 Vue 当成 DOM 操作库，仍然手动查找和修改大量元素。
- 在模板中写复杂逻辑，降低可读性。
- 直接修改 Props，破坏单向数据流。
- 用数组索引作为动态列表的 `key`。
- 把所有状态都放进全局 Store，导致组件之间过度耦合。