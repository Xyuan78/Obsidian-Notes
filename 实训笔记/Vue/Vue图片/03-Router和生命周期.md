---
title: Vue Router 与生命周期
tags:
  - 前端
  - Vue
  - 实训
  - Vue Router
  - 生命周期
aliases:
  - 路由守卫
---

## 1. router
### 1.1 router配置
设置router
在router下的index.tx文件夹下
```js
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'

import HomeView from '../views/HomeView.vue'

const routes: Array<RouteRecordRaw> = [

  // 关于页，异步加载

  {

    path: '/about',

    name: 'about',

    component: () => import('../views/AboutView.vue')

  }
  
const router = createRouter({

  history: createWebHistory(process.env.BASE_URL),

  routes

})

  

//路由守卫

router.beforeEach((to, from, next) => {

  //不要考虑，直接放行

  if(to.name === 'login' || to.name === 'register' || to.name === 'index' || to.name === 'personal'){

    next()

    return

  }

  //获得数据

  let isLogin = sessionStorage.getItem('isLogin')

    if(isLogin == null || isLogin === 'null' || isLogin === 'false' || isLogin === 'undefined'){

      //没有登录，跳转登录页

      alert('请先登录')

      next('/login')

      return

    }

    next()

  }

)

export default router
```

应用入口中安装路由：
在main.ts中
```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

createApp(App).use(router).mount("#app");
```

### 1.2 router使用
在vue文件中，在temple标签中通过‘@click = 函数名’，然后在script标签中创建函数名来进行跳转
```js
const foHome = () => {
	router.push('/home')
	rouoter.replace('/home')
}
```
## 2.生命周期

## 相关笔记

- [[实训笔记/Vue/02-基础语法|Vue 基础语法]] — 上一篇
- [[01-Vue Router 与 Pinia 入门|Vue Router 与 Pinia 入门（主库）]] — createRouter 与路由守卫
- [[03-生命周期与组合式 API|生命周期与组合式 API（主库）]] — 本文第 2 节的完整版本
