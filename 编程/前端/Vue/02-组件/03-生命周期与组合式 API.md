---
title: 生命周期与组合式 API
tags:
  - 前端
  - Vue
  - 生命周期
  - Composition API
---

# 生命周期与组合式 API

## 组件生命周期

一个组件通常会经历：

1. 创建和初始化。
2. 挂载到页面。
3. 响应式更新。
4. 卸载和清理。

组合式 API 中常见的生命周期钩子：

- `onBeforeMount`
- `onMounted`
- `onBeforeUpdate`
- `onUpdated`
- `onBeforeUnmount`
- `onUnmounted`

```vue
<script setup>
import { onMounted, onUnmounted, ref } from "vue";

const count = ref(0);
let timerId;

onMounted(() => {
  console.log("组件已挂载");

  timerId = setInterval(() => {
    count.value++;
  }, 1000);
});

onUnmounted(() => {
  console.log("组件即将卸载");
  clearInterval(timerId);
});
</script>
```

## 何时读取 DOM

`onMounted` 执行时，组件已经挂载到 DOM：

```vue
<script setup>
import { onMounted, ref } from "vue";

const inputRef = ref(null);

onMounted(() => {
  inputRef.value?.focus();
});
</script>

<template>
  <input ref="inputRef" />
</template>
```

模版引用名称必须与 `ref` 变量名一致。

## nextTick

修改响应式数据后，DOM 不一定立即更新。需要读取更新后的 DOM 时，使用 `nextTick()`：

```js
import { nextTick, ref } from "vue";

const visible = ref(false);

async function showMessage() {
  visible.value = true;
  await nextTick();
  console.log("消息元素已经渲染");
}
```

## 组合式函数

组合式函数是以 `use` 开头的可复用函数，用于封装响应式逻辑。

`src/composables/useCounter.js`：

```js
import { computed, ref } from "vue";

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);

  const doubled = computed(() => count.value * 2);

  function increase() {
    count.value++;
  }

  function reset() {
    count.value = initialValue;
  }

  return {
    count,
    doubled,
    increase,
    reset,
  };
}
```

组件中使用：

```vue
<script setup>
import { useCounter } from "../composables/useCounter";

const { count, doubled, increase, reset } = useCounter(10);
</script>

<template>
  <p>{{ count }} / {{ doubled }}</p>
  <button @click="increase">增加</button>
  <button @click="reset">重置</button>
</template>
```

## 组合式函数规范

- 名称以 `use` 开头。
- 可以调用其他组合式函数。
- 返回值通常包含响应式状态和操作方法。
- 不要在组合式函数中依赖特定组件实例，除非设计上确实需要。
- 请求、表单、轮询、权限、主题等逻辑都适合抽出复用。
- 生命周期钩子应在组件或组合式函数的同步执行阶段注册。

## 常见错误

- 在 `onUnmounted` 中忘记清理定时器、事件监听和订阅。
- 在组件创建前访问 DOM。
- 修改数据后立刻读取 DOM，却不等待 `nextTick()`。
- 把所有业务逻辑都塞进组件，无法复用和测试。
- 组合式函数命名没有体现用途。