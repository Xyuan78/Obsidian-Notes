---
title: CSS 基础总览
tags:
  - 前端
  - CSS
  - 基础
---

# CSS 基础总览

> [!abstract] 一句话理解
> CSS（Cascading Style Sheets，层叠样式表）负责页面元素的外观、尺寸、位置和动效。

## CSS 的基本语法

```css
选择器 {
  属性名: 属性值;
  属性名: 属性值;
}
```

示例：

```css
.card {
  color: #333333;
  background-color: #ffffff;
  padding: 16px;
  border-radius: 8px;
}
```

- `card` 是类选择器。
- `color` 和 `background-color` 是属性。
- `#333333` 和 `#ffffff` 是属性值。
- 每条声明以分号结尾。
- 多行声明中保持统一缩进，便于阅读。

## CSS 的三种引入方式

### 外部样式表

```html
<link rel="stylesheet" href="./style.css" />
```

优点：HTML 与样式分离、可复用、易缓存。实际项目优先使用这种方式。

### 内部样式表

```html
<style>
  h1 {
    color: navy;
  }
</style>
```

适合单页面演示，不适合大型项目长期维护。

### 行内样式

```html
<p style="color: red;">红色文字</p>
```

优先级较高且难以复用和统一维护，除特殊情况外避免使用。

## 注释

```css
/* 单行注释 */
/*
  多行注释
  也可以这样写
*/
```

项目代码应删除失效注释，但可以保留解释“为什么这样做”的说明。

## 颜色

```css
.example {
  color: red;
  background-color: #f5f5f5;
  border-color: rgb(0 100 200);
  box-shadow: 0 0 0 4px rgb(0 100 200 / 20%);
  accent-color: hsl(210 80% 50%);
}
```

常见写法：

- 颜色关键字：`red`、`rebeccapurple`
- HEX：`#ffffff`、`#333`
- RGB / RGBA：`rgb(0 0 0)`、`rgb(0 0 0 / 50%)`
- HSL：`hsl(210 80% 50%)`

带透明度的颜色常用 `/` 语法。设计系统中的颜色建议用 CSS 自定义属性统一管理：

```css
:root {
  --color-primary: #2563eb;
  --color-text: #1f2937;
}

.button {
  color: var(--color-primary);
}
```

## 长度单位

绝对单位：

- `px`：像素，界面中仍很常见，但不等同于物理像素。

相对单位：

- `em`：相对于当前元素的字体大小。
- `rem`：相对于根元素 `<html>` 的字体大小。
- `%`：相对于父元素或上下文。
- `vw` / `vh`：视口宽度和高度的百分比。
- `ch`：相对于当前字体中 `0` 的宽度。

基础排版中，字号常用 `rem`，行高和间距可使用无单位数值或 `rem`。

```css
html {
  font-size: 16px;
}

h1 {
  font-size: 2rem;
}

p {
  line-height: 1.7;
}
```

## 字体与文本

```css
body {
  font-family: "Noto Sans SC", "Microsoft YaHei", sans-serif;
  font-size: 16px;
  line-height: 1.7;
  color: #1f2937;
}

.title {
  font-size: 2rem;
  font-weight: 700;
  text-align: center;
}

.link {
  color: #2563eb;
  text-decoration: none;
}

.link:hover {
  text-decoration: underline;
}
```

字体族应提供后备字体，最后通常以 `sans-serif` 或 `serif` 收尾。

## 层叠与继承

当多个规则作用于同一元素时，浏览器会结合：

1. 样式来源和重要性。
2. 选择器优先级。
3. 源码出现顺序。

有些属性会继承，例如 `color`、`font-family`、`line-height`；有些不会，例如 `margin`、`padding`、`border`。

## 常见错误

- 声明末尾漏写分号。
- 类名选择器忘记写 `.`。
- 用行内样式处理大量页面样式。
- 所有尺寸都写死为 `px`。
- 只考虑桌面宽度，没有留出响应式空间。
- 使用 `!important` 掩盖选择器结构问题。
- 颜色和间距散落在多处，没有统一变量。

## 练习

创建一个 `.article` 卡片样式，设置字体、行高、文字颜色、背景色、内边距、圆角和边框，并使用 `:root` 定义至少两个颜色变量。
## 课程 PDF 对照补充

> [!source] 对应内容
> `html.pdf` 第 28–29 页。

### HTML、CSS、JavaScript 的分工

- HTML：页面的骨架，负责内容和结构。
- CSS：页面的“美妆”，负责字体、颜色、间距、布局等视觉表现。
- JavaScript：页面的动作，负责交互和行为。

### CSS 的定义

CSS 全称 Cascading Style Sheets，即层叠样式表，文件扩展名为 `.css`。

- CSS 为 HTML、XML 等结构化文档添加样式。
- 样式通常存储在样式表中，而不是散落在每个元素上。
- 外部样式表可以复用，减少重复代码，也便于多人和多页面协作。
- 多个样式规则可以通过层叠机制共同作用于一个元素。
- 将样式加入 HTML，是为了解决内容与表现混在一起的问题。

```html
<link rel="stylesheet" href="./css/style.css" />
```

### div + CSS 布局

课程中的 `div + CSS` 指使用 HTML 容器组织页面结构，再使用 CSS 控制布局。现代项目仍然使用这种思想，但布局工具优先选择：

- Flexbox：一维布局。
- Grid：二维布局。
- 正常文档流和 `gap`：优先使用的基础方案。

### CSS3

课程中提到的 CSS3 方向包括：

- 新元素和新属性。
- 更丰富的视觉效果。
- 过渡、动画和变换。
- 2D / 3D 图形能力。
- 配合 HTML5 完成音视频、本地存储和 Web 应用界面。

CSS 规范现在采用模块化演进，不同模块可以处于不同成熟阶段，因此不再适合简单理解成“所有功能统一进入某个 CSS4 版本”。

<!-- pdf-course:css-basic -->