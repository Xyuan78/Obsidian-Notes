---
title: TypeScript 与 Vue
tags:
  - 前端
  - Vue
  - TypeScript
---

# TypeScript 与 Vue

> [!source] 课程对照
> `vue2026.pdf` 第 20—26、105—116、169—171 页。

## 使用 TypeScript 的组件

```vue
<script lang="ts" setup>
import { ref } from "vue";

const count = ref<number>(0);
const message = ref<string>("Hello Vue");
</script>
```

- `<script setup lang="ts">` 表示使用 TypeScript。
- `ref<number>(0)` 明确声明 `count` 是数字。
- TypeScript 会在编译阶段帮助检查类型错误。

## 接口与对象

```ts
export interface Teacher {
  name: string;
  address: string;
  age: number;
}

const teacher: Teacher = {
  name: "张三",
  age: 38,
  address: "南京",
};

const teachers: Teacher[] = [teacher];
```

组件中使用：

```vue
<script lang="ts" setup>
import type { Teacher } from "@/types/teacher";

const teachers = ref<Teacher[]>([]);
</script>

<template>
  <p v-for="teacher in teachers" :key="teacher.name">
    {{ teacher.name }}
  </p>
</template>
```

能够推断出来的简单类型可以省略，复杂对象、接口数据和组件 Props 建议声明明确类型。

## Typed Props

```vue
<script lang="ts" setup>
interface Props {
  id: string;
  title?: string;
  count: number;
}

const props = withDefaults(defineProps<Props>(), {
  title: "默认标题",
  count: 0,
});
</script>
```

- `?` 表示可选属性。
- `withDefaults()` 为可选 Props 提供默认值。
- 子组件仍然不能修改 Props。

## Typed Emits

```vue
<script lang="ts" setup>
const emit = defineEmits<{
  submit: [value: string];
  close: [];
}>();

function submit() {
  emit("submit", "保存成功");
}
</script>
```

## reactive 与类型

```ts
import { reactive } from "vue";

interface LoginForm {
  username: string;
  password: string;
}

const form = reactive<LoginForm>({
  username: "",
  password: "",
});
```

## TypeScript 常见问题

### 类型定义与依赖版本不兼容

当 `node_modules` 中的类型定义报语法错误时：

```bash
tsc --version
npm ls typescript
npm ls @types/lodash
```

处理顺序：

1. 确认 TypeScript 与类型包版本兼容。
2. 使用 `npm ls` 检查是否存在多个版本。
3. 清理并重新安装依赖。
4. 检查 `tsconfig.json`、Vite 或 Webpack 配置。
5. 最后才考虑使用 `skipLibCheck` 临时绕过第三方声明文件检查。

```json
{
  "compilerOptions": {
    "skipLibCheck": true
  }
}
```

`skipLibCheck` 会降低类型检查覆盖面，不应把它当成首选修复方案。

### 避免滥用 any

`any` 会关闭类型检查，适合迁移旧代码的临时阶段，但长期使用会削弱 TypeScript 的价值。

优先使用：

- 明确接口。
- 泛型。
- `unknown` 加类型收窄。
- 联合类型。
- 可选属性和 `null` 检查。

## 类型检查

```bash
npm run type-check
npm run build
```

具体命令取决于项目的 `package.json`。提交前至少执行类型检查和构建。