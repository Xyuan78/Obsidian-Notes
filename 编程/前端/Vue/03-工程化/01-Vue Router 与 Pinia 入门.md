---
title: Vue Router 与 Pinia 入门
tags:
  - 前端
  - Vue
  - Vue Router
  - Pinia
---

# Vue Router 与 Pinia 入门

## 安装

创建项目时可以选择 Router 和 Pinia：

```bash
npm create vue@latest
```

也可以单独安装：

```bash
npm install vue-router pinia
```

## Vue Router 基础

路由用于把 URL 映射到不同页面组件。

```js
import { createRouter, createWebHistory } from "vue-router";
import HomeView from "../views/HomeView.vue";
import AboutView from "../views/AboutView.vue";

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: "/",
      name: "home",
      component: HomeView,
    },
    {
      path: "/about",
      name: "about",
      component: AboutView,
    },
  ],
});

export default router;
```

应用入口中安装路由：

```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

createApp(App).use(router).mount("#app");
```

## RouterLink 与 RouterView

```vue
<template>
  <nav>
    <RouterLink to="/">首页</RouterLink>
    <RouterLink to="/about">关于</RouterLink>
  </nav>

  <RouterView />
</template>
```

- `RouterLink`：生成导航链接。
- `RouterView`：渲染当前路由匹配的组件。

## 动态路由参数

```js
const routes = [
  {
    path: "/users/:id",
    name: "user-detail",
    component: () => import("../views/UserDetailView.vue"),
  },
];
```

组件中读取参数：

```vue
<script setup>
import { useRoute } from "vue-router";

const route = useRoute();
const userId = route.params.id;
</script>
```

编程式导航：

```js
import { useRouter } from "vue-router";

const router = useRouter();

router.push("/");
router.push({ name: "user-detail", params: { id: 1 } });
```

## 路由懒加载

```js
const UserDetailView = () => import("../views/UserDetailView.vue");
```

按需加载页面组件，减少首屏包体积。

## Pinia 基础

Pinia 用于管理跨组件、跨页面共享的状态。

```js
import { defineStore } from "pinia";
import { computed, ref } from "vue";

export const useCounterStore = defineStore("counter", () => {
  const count = ref(0);
  const doubled = computed(() => count.value * 2);

  function increase() {
    count.value++;
  }

  return {
    count,
    doubled,
    increase,
  };
});
```

组件中使用：

```vue
<script setup>
import { storeToRefs } from "pinia";
import { useCounterStore } from "../stores/counter";

const counterStore = useCounterStore();
const { count, doubled } = storeToRefs(counterStore);
</script>

<template>
  <p>{{ count }} / {{ doubled }}</p>
  <button @click="counterStore.increase">增加</button>
</template>
```

## 什么时候使用 Pinia

适合放进 Store 的状态：

- 用户登录信息。
- 购物车。
- 主题和语言设置。
- 多个页面共享的业务数据。
- 需要统一操作和持久化的状态。

不必放进 Store 的状态：

- 单个组件内部状态。
- 只在一个页面短暂使用的表单值。
- 可以通过 Props 清晰传递的数据。

## 状态持久化

Pinia 本身不负责自动持久化。可以：

- 手动使用 `localStorage`。
- 使用持久化插件。
- 对需要保存的数据做序列化。
- 不要把密码、令牌等敏感信息随意写入本地存储。

## 常见错误

- 把所有数据都放进 Pinia，导致组件之间隐性耦合。
- 忘记在应用入口安装 `router` 或 `pinia`。
- 直接解构 Store，导致响应性丢失，应使用 `storeToRefs()`。
- 路由页面和普通小组件混在一起。
- 依赖未校验的动态参数。
## Vue 2026 课程补充

> [!source] 对应内容
> `vue2026.pdf` 第 48—57 页。

### query 与 params

```js
router.push("/search?name=pen");
router.push({ path: "/search", query: { name: "pen" } });
router.push({ name: "user-detail", params: { id: 1 } });
```

- `query`：出现在 URL 查询字符串中，例如 `/search?name=pen`，适合筛选、分页和可分享的查询条件。
- `params`：对应路由路径中的动态参数，例如 `/users/:id`。
- 使用 `name` 跳转时更适合携带 `params`。
- 页面中可以通过 `route.query` 和 `route.params` 读取参数。

```vue
<script setup>
import { useRoute } from "vue-router";

const route = useRoute();
const keyword = route.query.keyword;
const userId = route.params.id;
</script>
```

### 编程式导航

```js
router.push("/main"); // 新增历史记录
router.replace("/login"); // 替换当前历史记录
router.back();
router.go(-1);
```

普通导航优先使用 `RouterLink`，需要在逻辑执行后跳转时使用 `useRouter()`。

### 嵌套路由与命名视图

嵌套路由适合“主框架 + 内容区”的后台布局：

```js
const routes = [
  {
    path: "/main",
    component: () => import("../views/MainView.vue"),
    children: [
      {
        path: "users",
        component: () => import("../views/UserListView.vue"),
      },
    ],
  },
];
```

父组件中需要放置 `<RouterView />`，子页面才会显示。

同一页面有多个显示区域时可以使用命名视图：

```vue
<RouterView />
<RouterView name="sidebar" />
```

```js
{
  path: "/main",
  components: {
    default: () => import("../views/MainView.vue"),
    sidebar: () => import("../views/SidebarView.vue"),
  },
}
```

### 路由守卫

登录权限判断可以放在全局前置守卫中：

```js
router.beforeEach((to) => {
  const isLoggedIn = Boolean(localStorage.getItem("token"));

  if (to.meta.requiresAuth && !isLoggedIn) {
    return {
      name: "login",
      query: {
        redirect: to.fullPath,
      },
    };
  }

  if (to.name === "login" && isLoggedIn) {
    return { name: "home" };
  }

  return true;
});
```

- `to`：即将进入的路由。
- `from`：当前离开的路由。
- 返回 `true` / 不返回值：允许导航。
- 返回 `false`：取消导航。
- 返回路由地址对象：重定向到其他页面。
- 动态路由需要添加 `meta`，例如 `meta: { requiresAuth: true }`。

路由守卫只负责前端体验，真正的权限和资源保护必须由服务器执行。

<!-- vue2026:router -->