---
title: 创建 Vue 项目与目录结构
tags:
  - 前端
  - Vue
  - 实训
  - 工程化
  - 环境配置
---

## 1.检查环境
```bash
node --version
npm --version
```
## 2. Vite创建项目
###### 1.使用管理员终端打开需要创建项目的文件夹，使用命令

```bash
npm create vue@latest
```
##### 2.按提示词选择功能

![[Pasted image 20260917090524.png]]
![[Pasted image 20260917090825.png]]
![[Pasted image 20260917090835.png]]
![[Pasted image 20260917090846.png]]
![[Pasted image 20260917090915.png]]
至此，已完成项目的创建，下一步需要安装依赖

##### 3.进入项目安装依赖
1.进入项目
```bash
cd vue-notes-app
```
![[Pasted image 20260917091104.png]]
2.安装依赖
```bash
npm install
```
出现下面这个之后代表完成
![[Pasted image 20260917091414.png]]

3.启动开发服务器
```bash
npm run dev
```
## 3.Vue CLI创建项目
##### 1.在文件夹下管理员终端
命令：
```
vue create 名称 回车
```
2.
![[Pasted image 20260917101743.png]]![[Pasted image 20260917101829.png]]
![[Pasted image 20260917101846.png]]

一路yes，然后选择动态CSS

![[Pasted image 20260917101930.png]]

安装less
```npm
npm install -D less less-loader
```

安装Element Plus
```npm
npm install element-plus
```
##### 2.最后测试
```npm
npm run serve
```
如果出现报错，在tsconfig.json中加入
```json
{ 
"compilerOptions": { 
// 关闭 "找不到类型声明" 的错误提示
 "noImplicitAny": false,
	  } 
  } 

```
## 4.常见项目结构
```text
vue-notes-app/
├── public/
├── src/
│   ├── assets/
│   ├── components/
│   ├── router/
│   ├── stores/
│   ├── views/
│   ├── App.vue
│   └── main.js
├── index.html
├── package.json
├── vite.config.js
└── README.md
```
- `public`：不会经过构建处理的静态资源。
- `src/assets`：会经过 Vite 构建处理的资源。
- `src/components`：可复用组件。
- `src/views`：自己写的页面。
- `src/router`：路由配置。
- `src/stores`：Pinia 状态仓库。
- `src/App.vue`：根组件。
- `src/main.js`：应用入口，创建并挂载 Vue 应用。
- `index.html`：Vite 的 HTML 入口。
- package.json：安装的插件（或依赖）
- `vite.config.js`：Vite 与插件配置。
- node_models：插件源代码
## 5.常见问题
- `npm` 命令不可用：检查 Node.js 是否正确安装。
- 端口被占用：按终端提示换端口，或关闭占用端口的程序。
- 页面空白：查看浏览器控制台和终端错误。
- 资源路径错误：区分 `public` 与 `src/assets` 的使用方式。
- 不要提交 `node_modules`，它应由 `package-lock.json` 和 `npm install` 恢复。
##### 1.修改端口号
在package.json中修改
```json
"scripts": {

    "serve": "vue-cli-service serve -- open --host localhost --port 3000",

    "build": "vue-cli-service build"

  },//--poet + 端口号
  //--open 自动打开
  //--host localhost  服务器是localhost
```
##### 2.处理变量声明后不使用，程序报错
打开vue.config.js
```js
const { defineConfig } = require('@vue/cli-service')

module.exports = defineConfig({
  transpileDependencies: true,
  lintOnSave : false,
})
```


## 相关笔记

- [[实训笔记/Vue/02-基础语法|Vue 基础语法]] — 建好项目后的下一步
- [[实训笔记/JS/03-Bootstrap集成与组件化开发|Bootstrap 集成与组件化开发]] — 本笔记 `main.js` 的依赖导入
- [[00-Vue 学习索引|Vue 学习索引（主库）]] — 整套 Vue 笔记的入口
- [[02-创建 Vue 项目与目录结构|创建 Vue 项目与目录结构（主库）]] — 同一主题，Vite 与 Vue CLI 对照
