---
title: Vue 课程 PDF 知识点映射
tags:
  - 前端
  - Vue
  - 课程
  - 索引
---

# Vue 课程 PDF 知识点映射

> [!info] 文档信息
> 原文件：`C:\Users\Lenovo\Desktop\实训课程\9.14\vue2026.pdf`  
> 页数：172 页  
> 处理原则：只提取可复用的前端知识；作业要求、后端 Java、数据库脚本、广告式延伸阅读和无关新闻分类不作为 Vue 教学正文。

## 页面与笔记对应

| PDF 页码 | 主要内容 | 对应笔记 |
| --- | --- | --- |
| 1–17 | Node.js、npm、VSCode、Vue CLI、项目创建、目录结构、端口和别名 | [[02-创建 Vue 项目与目录结构]] |
| 18–19 | 插值、`v-text`、`v-html`、`v-bind`、`v-on`、条件判断、列表 | [[03-模板语法与响应式数据]]、[[04-条件渲染、列表渲染与事件]] |
| 20–23 | 组件、Typed Props、Emits、数组方法和 Less/Sass | [[01-组件基础]]、[[06-TypeScript 与 Vue]] |
| 24–27 | computed、watch、watchEffect、接口和类 | [[05-表单绑定与计算监听]]、[[06-TypeScript 与 Vue]] |
| 28–31 | 事件处理、事件修饰符、按键修饰符、readonly 和自定义属性 | [[04-条件渲染、列表渲染与事件]]、[[03-模板语法与响应式数据]] |
| 32–34 | 表单控件、v-model 修饰符和生命周期 | [[05-表单绑定与计算监听]]、[[03-生命周期与组合式 API]] |
| 36–46 | Vuex State、Getters、Mutations、Actions、Modules 和命名空间 | [[05-Vuex 旧项目入门]] |
| 47 | localStorage 与 sessionStorage | [[01-后台管理系统常用模式]] |
| 48–56 | Vue Router、query/params、嵌套路由、命名视图和路由守卫 | [[01-Vue Router 与 Pinia 入门]] |
| 57–68 | Axios、请求封装、拦截器、Loading、Token 和 API 模块化 | [[02-Axios 请求封装]] |
| 65–69 | Element Plus 引入、图标、Bootstrap/jQuery 兼容 | [[03-Element Plus 组件入门]] |
| 70–95 | Spring Boot、MyBatis、拦截器、跨域和后端响应封装 | 不作为 Vue 核心收录；安全边界归纳到 Axios 与实战笔记 |
| 96–146 | Element Plus 布局、菜单、表单验证、表格、分页、对话框、上传、树和国际化 | [[03-Element Plus 组件入门]] |
| 150–155 | ECharts 安装、初始化、数据更新和图表类型 | [[04-ECharts 数据可视化]] |
| 156–162 | OA 项目、菜单、登录、验证码和 setup 语法糖 | [[01-后台管理系统常用模式]] |
| 163–168 | Console 调试、跨域和日期格式化 | [[07-开发调试与工具函数]] |
| 169–171 | TypeScript 与类型包版本兼容、依赖检查和构建配置 | [[06-TypeScript 与 Vue]] |
| 172 | 新闻分类示例 | 与 Vue 无关，未收录 |

## 旧式写法提示

- 课程基于较早的 Vue CLI、Vuex 和 jQuery/Bootstrap 生态。
- 新 Vue 3 项目优先 Vite、Pinia 和现代 Element Plus 用法。
- `v-html` 不能直接渲染不可信用户输入。
- 前端 MD5 不是密码安全方案，密码必须通过 HTTPS 传输并由服务器安全哈希。
- 验证码必须由服务器校验，不能只在前端比较答案。
- 前端菜单和路由权限必须由服务器端权限校验兜底。