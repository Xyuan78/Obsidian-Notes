---
title: 创建 Vue 项目与目录结构
tags:
  - 前端
  - Vue
  - 工程化
  - 基础
---

# 创建 Vue 项目与目录结构

## 环境准备

先安装：

- Node.js，建议使用当前 LTS 版本。
- npm，通常随 Node.js 一起安装。
- VS Code 与 Vue 官方扩展。

检查环境：

```bash
node --version
npm --version
```

## 使用官方脚手架创建项目

```bash
npm create vue@latest
```

按提示输入项目名并选择功能。例如：

```text
Project name: vue-notes-app
Add TypeScript? No
Add JSX Support? No
Add Vue Router for Single Page Application development? Yes
Add Pinia for state management? Yes
Add Vitest for Unit Testing? No
Add ESLint for code quality? Yes
Add Prettier for code formatting? Yes
```

进入项目并安装依赖：

```bash
cd vue-notes-app
npm install
```

启动开发服务器：

```bash
npm run dev
```

生产构建：

```bash
npm run build
```

本地预览构建结果：

```bash
npm run preview
```

## 常见目录结构

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
- `src/views`：路由页面组件。
- `src/router`：路由配置。
- `src/stores`：Pinia 状态仓库。
- `src/App.vue`：根组件。
- `src/main.js`：应用入口，创建并挂载 Vue 应用。
- `index.html`：Vite 的 HTML 入口。
- `vite.config.js`：Vite 与插件配置。

## 应用入口

```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

createApp(App).use(router).mount("#app");
```

- `createApp(App)`：创建应用实例。
- `.use(router)`：安装路由等插件。
- `.mount("#app")`：挂载到 `index.html` 中的 `#app` 节点。

## Vite 的作用

- 提供快速开发服务器。
- 支持 `.vue` 单文件组件。
- 支持热更新，修改代码后浏览器快速更新。
- 生产构建时打包和优化资源。

## 常见问题

- `npm` 命令不可用：检查 Node.js 是否正确安装。
- 端口被占用：按终端提示换端口，或关闭占用端口的程序。
- 页面空白：查看浏览器控制台和终端错误。
- 资源路径错误：区分 `public` 与 `src/assets` 的使用方式。
- 不要提交 `node_modules`，它应由 `package-lock.json` 和 `npm install` 恢复。
## Vue 2026 课程补充

> [!source] 对应内容
> `vue2026.pdf` 第 1—17 页。

### Vue CLI 与 Vite

课程使用 Vue CLI：

```bash
npm install -g @vue/cli
vue create vue-demo
```

现代新项目更推荐 Vite：

```bash
npm create vue@latest
```

- Vue CLI 基于 Webpack，常见于旧项目和课程示例。
- Vite 启动和热更新更快，是 Vue 3 新项目的常见脚手架。
- 读取旧项目时要理解 `vue.config.js`、`chainWebpack` 等 Vue CLI 配置。
- 新项目优先阅读 `vite.config.js` 和 `package.json`。

### npm 镜像

下载速度不稳定时可以切换镜像：

```bash
npm config set registry https://registry.npmmirror.com/
```

查看当前源：

```bash
npm config get registry
```

不要同时混用多个来源不明的镜像，遇到依赖异常时先检查镜像和依赖锁文件。

### 开发服务器配置

Vue CLI 可以在 `package.json` 脚本中传入参数：

```json
{
  "scripts": {
    "serve": "vue-cli-service serve --open --host localhost --port 3000"
  }
}
```

Vite 配置可以放在 `vite.config.js`：

```js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import { fileURLToPath, URL } from "node:url";

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./src", import.meta.url)),
    },
  },
  server: {
    open: true,
    host: "localhost",
    port: 3000,
  },
});
```

`@` 通常代表 `src` 目录，路径别名可以简化和稳定模块导入。

### 样式预处理器

Vue 单文件组件可以使用 Less 或 Sass：

```vue
<style lang="less" scoped>
@baseSize: 10px;
@backgroundColor: antiquewhite;

.card {
  padding: @baseSize * 2;
  background: @backgroundColor;

  .title {
    font-weight: 700;
  }
}
</style>
```

使用 Less/Sass 前需要安装对应依赖，并在团队中统一预处理器和代码风格。

### 开发调试

- 浏览器 `F12` 打开开发者工具。
- Console 查看日志和错误。
- Network 查看接口请求、状态码、请求参数和响应。
- Elements 查看组件生成的 DOM 和最终样式。
- Vue DevTools 可以查看组件树、Props、事件和状态变化。

<!-- vue2026:project -->