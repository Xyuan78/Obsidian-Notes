---
title: Axios 请求封装
tags:
  - 前端
  - Vue
  - Axios
  - 请求
---

# Axios 请求封装

> [!source] 课程对照
> `vue2026.pdf` 第 57–68 页。

## Axios 的作用

Axios 是基于 Promise 的 HTTP 客户端，可以在浏览器和 Node.js 中使用。Vue 页面常通过 Axios 与后端接口交互。

安装：

```bash
npm install axios
```

## 最基础的请求

```js
import axios from "axios";

const response = await axios.get("https://api.example.com/users");
console.log(response.data);
```

常见请求方法：

```js
axios.get(url, { params });
axios.post(url, data);
axios.put(url, data);
axios.delete(url);
```

- `get` 的查询参数通常放在 `params` 中。
- `post`、`put`、`patch` 的数据通常放在请求体中。
- 删除操作可以使用 `delete`。

## 创建统一实例

```js
import axios from "axios";

const service = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 8000,
  headers: {
    "Content-Type": "application/json",
  },
  withCredentials: true,
});

export default service;
```

- `baseURL`：统一接口前缀。
- `timeout`：超时时间。
- `withCredentials`：请求是否携带跨站 Cookie，是否开启取决于后端 CORS 配置。
- 不要把生产地址硬编码在多个文件中，可以使用环境变量。

## 请求拦截器

```js
service.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  },
  (error) => Promise.reject(error),
);
```

请求拦截器适合统一添加 Token、租户标识、语言等信息。

不要在 `FormData` 请求中手动固定 JSON `Content-Type`，浏览器需要自动生成带 boundary 的请求头。

## 响应拦截器

```js
service.interceptors.response.use(
  (response) => {
    return response.data;
  },
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("token");
      // 跳转登录页或刷新登录状态
    }

    return Promise.reject(error);
  },
);
```

响应拦截器适合：

- 统一返回 `response.data`。
- 统一处理 401、403、500 等状态。
- 统一显示错误消息。
- 统一关闭 Loading。

不要在每次请求失败时都重复弹出相同提示，可以增加错误去重或节流。

## Loading 的状态管理

错误做法是用一个布尔值处理并发请求，因为较早请求完成后可能提前关闭 Loading。

```js
let loadingCount = 0;

function showLoading() {
  if (loadingCount === 0) {
    // 打开全局 Loading
  }
  loadingCount++;
}

function hideLoading() {
  loadingCount--;
  if (loadingCount <= 0) {
    loadingCount = 0;
    // 关闭全局 Loading
  }
}
```

## 封装请求方法

```js
export function get(config) {
  return service({
    ...config,
    method: "get",
    params: config.data,
  });
}

export function post(config) {
  return service({
    ...config,
    method: "post",
    data: config.data,
  });
}

export function put(config) {
  return service({
    ...config,
    method: "put",
    data: config.data,
  });
}

export function del(config) {
  return service({
    ...config,
    method: "delete",
    data: config.data,
  });
}
```

## API 模块化

```js
import { get, post } from "./service";

export function getUserList(data) {
  return get({
    url: "/users",
    data,
  });
}

export function createUser(data) {
  return post({
    url: "/users",
    data,
  });
}
```

组件中只调用业务 API：

```js
import { getUserList } from "@/api/user";

const users = await getUserList({ page: 1, size: 10 });
```

这样可以把接口地址、请求方式和数据处理集中在 `api` 或 `service` 层，组件不必到处拼 URL。

## Token 与安全

课程把 Token 和验证码保存到 `localStorage`。这种方式使用简单，但存在 XSS 风险。

- 不要保存明文密码。
- 高敏感认证优先考虑 `HttpOnly`、`Secure`、`SameSite` Cookie。
- 如果使用 `localStorage`，Token 必须有合理过期时间，并做好 XSS 防护。
- 前端 MD5 不能代替 HTTPS，也不能替代服务器端密码哈希。
- 服务器必须重新校验权限，不能只相信前端路由守卫。

## 常见错误

- 在组件中到处写完整接口地址。
- 请求失败后只 `console.log`，用户看不到明确反馈。
- 响应拦截器已经返回 `response.data`，业务代码又访问一次 `.data`。
- 并发请求共用单个 Loading 状态。
- 永远不清除失效 Token。
- 把密码或密钥写进前端环境变量，前端环境变量并不是秘密存储。