---
title: Element Plus 组件入门
tags:
  - 前端
  - Vue
  - Element Plus
  - UI
---

# Element Plus 组件入门

> [!source] 课程对照
> `vue2026.pdf` 第 65–66、95–146 页。

## Element Plus 简介

Element Plus 是面向 Vue 3 的组件库。Vue 2 项目通常使用 Element UI，Vue 3 项目使用 Element Plus。

安装：

```bash
npm install element-plus
```

完整引入：

```js
import { createApp } from "vue";
import ElementPlus from "element-plus";
import "element-plus/dist/index.css";
import App from "./App.vue";

createApp(App).use(ElementPlus).mount("#app");
```

实际项目也可以使用自动按需引入，减少最终包体积。

## 图标

安装：

```bash
npm install @element-plus/icons-vue
```

注册全部图标：

```js
import * as ElementPlusIconsVue from "@element-plus/icons-vue";

const app = createApp(App);

for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component);
}

app.mount("#app");
```

组件中使用：

```vue
<el-icon><Setting /></el-icon>
```

## 页面布局

```vue
<template>
  <el-container class="layout">
    <el-header>页头</el-header>

    <el-container>
      <el-aside width="220px">侧边栏</el-aside>
      <el-main>
        <RouterView />
      </el-main>
    </el-container>
  </el-container>
</template>

<style scoped>
.layout {
  min-height: 100vh;
}
</style>
```

- `el-container`：外层容器。
- `el-header`：页头。
- `el-aside`：侧边栏。
- `el-main`：主要内容。
- `el-footer`：页脚。
- 后台系统常把 `<RouterView />` 放在 `el-main` 中。

## 菜单与路由

```vue
<el-menu router :default-active="$route.path">
  <el-menu-item index="/dashboard">首页</el-menu-item>

  <el-sub-menu index="system">
    <template #title>系统管理</template>
    <el-menu-item index="/users">用户管理</el-menu-item>
    <el-menu-item index="/roles">角色管理</el-menu-item>
  </el-sub-menu>
</el-menu>
```

`:router="true"` 或直接写 `router` 后，菜单项的 `index` 会作为路由地址。动态菜单通常由后端或静态配置生成，再通过 `v-for` 渲染。

菜单是否显示不能代替后端权限控制。

## 消息与确认框

```js
import { ElMessage, ElMessageBox } from "element-plus";

ElMessage.success("保存成功");
ElMessage.error("保存失败");

await ElMessageBox.confirm("确定删除吗？", "警告", {
  type: "warning",
  confirmButtonText: "确定",
  cancelButtonText: "取消",
});
```

`ElMessageBox.confirm()` 返回 Promise，需要处理确认和取消分支。

## 表单与验证

```vue
<script setup>
import { reactive, ref } from "vue";
import type { FormInstance, FormRules } from "element-plus";

const formRef = ref<FormInstance>();

const form = reactive({
  username: "",
  password: "",
});

const rules: FormRules = {
  username: [
    { required: true, message: "请输入用户名", trigger: "blur" },
    { min: 4, max: 10, message: "长度应为 4–10", trigger: "blur" },
  ],
  password: [
    { required: true, message: "请输入密码", trigger: "blur" },
    { min: 6, max: 16, message: "长度应为 6–16", trigger: "blur" },
  ],
};

async function submit() {
  if (!formRef.value) return;

  const valid = await formRef.value.validate().catch(() => false);
  if (!valid) return;

  // 提交表单
}
</script>

<template>
  <el-form ref="formRef" :model="form" :rules="rules" label-width="90px">
    <el-form-item label="用户名" prop="username">
      <el-input v-model="form.username" />
    </el-form-item>

    <el-form-item label="密码" prop="password">
      <el-input v-model="form.password" type="password" />
    </el-form-item>

    <el-button type="primary" @click="submit">提交</el-button>
  </el-form>
</template>
```

- `model`：表单数据。
- `rules`：验证规则。
- `prop`：字段名，必须和规则键对应。
- `required`：必填。
- `min` / `max`：长度范围。
- `pattern`：正则验证。
- `trigger`：触发时机，常见为 `blur`、`change`。
- `validate()`：执行验证。
- `clearValidate()`：清除验证状态和提示。
- `resetFields()`：重置字段值和验证状态，适合明确需要恢复初始值的表单。

## 表格

```vue
<el-table :data="users" @selection-change="handleSelectionChange">
  <el-table-column type="selection" width="55" />
  <el-table-column prop="id" label="ID" width="80" />
  <el-table-column prop="name" label="姓名" />
  <el-table-column label="操作">
    <template #default="{ row }">
      <el-button @click="view(row)">查看</el-button>
      <el-button type="danger" @click="remove(row)">删除</el-button>
    </template>
  </el-table-column>
</el-table>
```

- `:data`：表格数据。
- `prop`：字段名。
- `label`：列标题。
- `type="selection"`：多选列。
- `scope.row` 或插槽中的 `{ row }` 表示当前行。
- `@selection-change` 可以获得选中行数组。

删除、编辑等操作必须再次经过权限校验和服务器验证。

## 分页

```vue
<el-pagination
  v-model:current-page="pagination.page"
  v-model:page-size="pagination.size"
  :page-sizes="[10, 20, 50]"
  layout="total, sizes, prev, pager, next, jumper"
  :total="pagination.total"
  @size-change="loadData"
  @current-change="loadData"
/>
```

后端分页时，前端通常只保存当前页数据，并向后端发送页码、每页数量和查询条件。

## 对话框表单

```vue
<el-dialog v-model="dialogVisible" title="新增用户" width="600px">
  <UserForm />
  <template #footer>
    <el-button @click="dialogVisible = false">取消</el-button>
    <el-button type="primary">保存</el-button>
  </template>
</el-dialog>
```

对话框关闭时，应根据需求决定使用 `resetFields()` 或 `clearValidate()`：

- 重新打开时需要恢复空表单：使用 `resetFields()`。
- 只清除上一次错误提示，保留当前值：使用 `clearValidate()`。

## 上传与树形控件

- `el-upload`：上传文件，监听成功、失败和上传前校验。
- `el-tree`：展示层级数据，支持复选框、默认展开和节点操作。
- 树形数据通常需要后端或前端递归转换成 `children` 结构。
- 上传文件类型、大小和权限必须由服务器再次校验。

## 国际化

Element Plus 可以配置语言包。项目自身文案可以使用 `vue-i18n` 管理：

```js
import { createI18n } from "vue-i18n";

const i18n = createI18n({
  legacy: false,
  locale: "zh-CN",
  fallbackLocale: "en",
  messages: {
    "zh-CN": {
      login: "登录",
    },
    en: {
      login: "Login",
    },
  },
});
```

国际化不仅是替换文字，还要考虑日期、数字、表单校验和布局长度差异。

## 常见错误

- 忘记引入 Element Plus CSS，导致组件样式异常。
- 表单规则键与 `prop` 不一致。
- 直接修改表格行对象却未同步服务器。
- 前端隐藏按钮就认为权限安全。
- 所有组件全局注册，导致包体积过大。
- 对话框关闭后没有清理表单验证状态。