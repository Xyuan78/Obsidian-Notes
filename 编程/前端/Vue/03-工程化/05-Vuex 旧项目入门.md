---
title: Vuex 旧项目入门
tags:
  - 前端
  - Vue
  - Vuex
  - 状态管理
---

# Vuex 旧项目入门

> [!warning] 适用范围
> Vuex 常见于 Vue 2 和早期 Vue 3 项目。新 Vue 3 项目通常优先使用 Pinia。本笔记用于读懂和维护旧项目。

> [!source] 课程对照
> `vue2026.pdf` 第 36–46、54—56 页。

## Vuex 的核心概念

- `state`：集中存储状态。
- `getters`：类似计算属性，根据 State 派生出结果。
- `mutations`：同步修改 State 的方法。
- `actions`：处理异步操作，最后通过 `commit` 调用 Mutation。
- `modules`：把状态拆分成多个模块。

## 创建 Store

```js
import { createStore } from "vuex";

const store = createStore({
  state: {
    count: 0,
  },
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  mutations: {
    setCount(state, value) {
      state.count = value;
    },
  },
  actions: {
    async loadCount({ commit }) {
      const value = await fetchCount();
      commit("setCount", value);
    },
  },
});

export default store;
```

应用入口安装：

```js
createApp(App).use(store).mount("#app");
```

## 读取和修改状态

```js
import { useStore } from "vuex";

const store = useStore();

console.log(store.state.count);
console.log(store.getters.doubleCount);

store.commit("setCount", 10);
await store.dispatch("loadCount");
```

- `state` 不要直接修改，应通过 `commit` 调用 Mutation。
- `getters` 适合派生计算，不负责修改状态。
- `actions` 负责异步流程，再调用 Mutation。

## 异步 Action

```js
actions: {
  async login({ commit }, credentials) {
    const user = await loginApi(credentials);
    commit("setUser", user);
    return user;
  },
}
```

Action 可以返回 Promise，调用方可以继续使用 `await`、`then()` 和 `catch()`。

## 模块化与命名空间

```js
const userModule = {
  namespaced: true,
  state: () => ({
    profile: null,
  }),
  mutations: {
    setProfile(state, profile) {
      state.profile = profile;
    },
  },
  actions: {
    async loadProfile({ commit }) {
      const profile = await getProfile();
      commit("setProfile", profile);
    },
  },
  getters: {
    displayName: (state) => state.profile?.name ?? "未登录",
  },
};

const store = createStore({
  modules: {
    user: userModule,
  },
});
```

调用：

```js
store.commit("user/setProfile", profile);
await store.dispatch("user/loadProfile");
console.log(store.getters["user/displayName"]);
```

模块化可以减少大型项目中的状态命名冲突。模块状态应使用函数返回，避免模块之间共享同一个对象。

## Vuex 与 Pinia 对比

| 对比项 | Vuex | Pinia |
| --- | --- | --- |
| 适用 | Vue 2、旧 Vue 3 项目 | Vue 2.7、Vue 3 |
| Mutation | 必须使用 | 没有 Mutation |
| 语法 | `commit` / `dispatch` | 直接调用 Store 方法 |
| TypeScript | 配置更繁琐 | 类型推导更自然 |
| 模块 | Modules / namespaced | 多个独立 Store |
| 建议 | 维护旧项目 | 新项目优先 |

## 旧项目维护建议

- 先确认当前项目使用的是 Vuex 还是 Pinia，不要混入两套完全重复的状态管理。
- 不要在组件中直接修改 Vuex State。
- 明确区分同步 Mutation 和异步 Action。
- 模块使用 `namespaced: true` 时，调用路径必须包含命名空间。
- 状态过多、模块依赖混乱时，应拆分领域 Store。
- 需要注意 Store 与 `localStorage` 的区别：Store 提供响应式共享状态，存储对象只负责持久化数据。