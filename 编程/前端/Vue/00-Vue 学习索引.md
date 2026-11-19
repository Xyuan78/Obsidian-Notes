---
title: Vue 学习索引
tags:
  - 前端
  - Vue
aliases:
  - Vue笔记
---

# Vue 学习索引

> [!source] 课程来源
> 已结合 `vue2026.pdf`（172 页）补充。页码对应关系见 [[Vue 课程 PDF 知识点映射]]。课程中的 Vuex、Vue CLI、jQuery/Bootstrap 属于旧式或兼容写法，笔记中已标注现代替代方案。

> [!abstract] 学习路线
> 本套笔记以 Vue 3 + Composition API + `<script setup>` 为主。先掌握响应式数据、模板语法和组件，再学习路由、请求、UI、图表与状态管理。

## 01 基础

- [[01-Vue 基础总览]]：Vue 的定位、声明式渲染、响应式系统和单文件组件。
- [[02-创建 Vue 项目与目录结构]]：Node.js、Vue CLI、Vite、项目配置、目录职责和 Less/Sass。
- [[03-模板语法与响应式数据]]：插值、`v-text`、`v-html`、动态属性、`ref`、`reactive` 和只读状态。
- [[04-条件渲染、列表渲染与事件]]：`v-if`、`v-show`、`v-for`、对象/嵌套循环、事件与按键修饰符。
- [[05-表单绑定与计算监听]]：`v-model`、`computed`、`watch`、`watchEffect` 和表单控件行为。

## 02 组件

- [[01-组件基础]]：组件拆分、单文件组件、注册、Props 和插槽。
- [[02-组件通信]]：父传子、子传父、`v-model`、`provide/inject` 和状态提升。
- [[03-生命周期与组合式 API]]：生命周期钩子、模板引用、`nextTick()` 和组合式函数。

## 03 工程化

- [[01-Vue Router 与 Pinia 入门]]：路由、query/params、嵌套路由、守卫、Store 和持久化。
- [[02-Axios 请求封装]]：实例配置、请求/响应拦截器、API 分层、Token 和错误处理。
- [[03-Element Plus 组件入门]]：布局、菜单、表单验证、表格、分页、对话框、上传、树和国际化。
- [[04-ECharts 数据可视化]]：初始化、接口数据、响应式尺寸、更新和销毁。
- [[05-Vuex 旧项目入门]]：State、Getters、Mutations、Actions、Modules 和 Pinia 对比。
- [[06-TypeScript 与 Vue]]：类型化 Ref、界面接口、Props、Emits、类型检查与依赖兼容。
- [[07-开发调试与工具函数]]：Console、Network、跨域、日期格式化与常用调试方法。

## 04 实战

- [[01-后台管理系统常用模式]]：页面框架、动态菜单、登录认证、路由权限、列表、分页、对话框和上传。

## 学习原则

- 优先使用组合式 API 和 Vue 3。
- 组件保持职责单一，数据流向清晰。
- 模板只负责描述界面，复杂计算放入 `computed` 或组合式函数。
- 不直接修改 Props，也不直接修改 Vuex State。
- 列表渲染必须使用稳定且唯一的 `key`。
- 前端权限控制不能替代后端授权。
- 新项目优先 Pinia，旧 Vuex 项目按现有体系维护。