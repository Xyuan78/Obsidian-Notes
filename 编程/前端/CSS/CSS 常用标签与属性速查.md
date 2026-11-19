---
title: CSS 常用标签与属性速查
tags:
  - 前端
  - CSS
  - 速查
aliases:
  - CSS 常用属性速查
  - CSS 选择器与属性速查
---

# CSS 常用标签与属性速查

> [!info] 先说明一个概念
> CSS 本身没有 HTML 那样的标签。页面中的标签来自 HTML，CSS 通过“选择器”选中这些标签，再用“属性”控制样式。本页分为三部分：常用选择器、常见 HTML 标签的样式方式、常用 CSS 属性速查。

## CSS 的基本写法

```css
选择器 {
  属性: 属性值;
}
```

示例：

```css
.card {
  color: #1f2937;
  background-color: #ffffff;
  padding: 16px;
  border: 1px solid #d1d5db;
  border-radius: 12px;
}
```

- `.card`：选择器。
- `color`、`padding`：CSS 属性。
- `#1f2937`、`16px`：属性值。
- 每条声明以分号结尾。

## 常用选择器

### 基础选择器

| 选择器 | 匹配对象 | 示例 |
| --- | --- | --- |
| `*` | 所有元素 | `* { box-sizing: border-box; }` |
| `标签名` | 指定类型的 HTML 标签 | `p { line-height: 1.7; }` |
| `.类名` | 带有指定 `class` 的元素 | `.card { padding: 16px; }` |
| `#id` | 指定 `id` 的唯一元素 | `#site-title { font-size: 2rem; }` |
| `[attr]` | 拥有指定属性的元素 | `[disabled] { opacity: 0.6; }` |
| `[attr="value"]` | 属性值完全相等的元素 | `input[type="email"]` |

> [!tip] 使用建议
> 组件和模块优先使用类选择器；ID 选择器优先级高，通常用于锚点和 JavaScript 定位，不建议大量用于样式。

### 组合选择器

| 选择器 | 名称 | 示例 | 含义 |
| --- | --- | --- | --- |
| `A B` | 后代选择器 | `.menu a` | `.menu` 内任意层级的 `a` |
| `A > B` | 子选择器 | `.menu > li` | `.menu` 的直接子元素 `li` |
| `A + B` | 相邻兄弟选择器 | `h2 + p` | 紧跟在 `h2` 后的第一个 `p` |
| `A ~ B` | 后续兄弟选择器 | `h2 ~ p` | `h2` 后面的所有同级 `p` |
| `A, B` | 选择器列表 | `h1, h2, h3` | 同时匹配多组元素 |

### 伪类与伪元素

伪类表示元素的状态或位置：

```css
a:hover {
  color: #1d4ed8;
}

button:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
}

li:nth-child(even) {
  background-color: #f8fafc;
}

input:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}
```

常见伪类：

| 伪类 | 作用 |
| --- | --- |
| `:hover` | 鼠标悬停 |
| `:active` | 正在被激活 |
| `:focus` | 获得焦点 |
| `:focus-visible` | 浏览器认为需要显示焦点时 |
| `:visited` | 已访问链接 |
| `:disabled` | 被禁用控件 |
| `:checked` | 被选中的单选框或复选框 |
| `:required` | 必填控件 |
| `:valid` / `:invalid` | 原生校验有效或无效 |
| `:placeholder-shown` | 正在显示占位文本 |
| `:first-child` / `:last-child` | 第一个或最后一个子元素 |
| `:nth-child(n)` | 第 `n` 个子元素 |
| `:not(...)` | 排除指定条件 |
| `:is(...)` | 匹配括号内任意选择器 |
| `:where(...)` | 与 `:is()` 类似，但优先级为 0 |
| `:has(...)` | 根据后代或后续状态选择父元素 |

伪元素表示元素的一部分或生成内容：

```css
.card::before {
  content: "";
  display: block;
  width: 48px;
  height: 4px;
  background-color: #2563eb;
}

::selection {
  color: white;
  background-color: #2563eb;
}
```

常见伪元素：

| 伪元素 | 作用 |
| --- | --- |
| `::before` | 在元素内容前生成内容 |
| `::after` | 在元素内容后生成内容 |
| `::placeholder` | 输入框占位文本 |
| `::selection` | 用户选中的文本 |
| `::marker` | 列表项标记 |
| `::first-line` | 第一行文本 |
| `::first-letter` | 第一个字母或汉字 |

`::before` 和 `::after` 必须设置 `content` 才会生成。

## 常见 HTML 标签的 CSS 写法

| HTML 标签 | 常见样式目标 | 常用 CSS 属性 |
| --- | --- | --- |
| `<html>` | 根字号、滚动行为、主题变量 | `font-size`、`scroll-behavior`、自定义属性 |
| `<body>` | 页面默认字体、颜色、背景、外边距 | `margin`、`font-family`、`line-height`、`color`、`background-color` |
| `<header>` | 页头布局、粘性定位 | `display`、`padding`、`position: sticky`、`z-index` |
| `<nav>` | 导航布局与链接间隔 | `display: flex`、`gap`、`justify-content` |
| `<main>` | 主内容最大宽度和居中 | `width`、`max-width`、`margin-inline: auto`、`padding` |
| `<section>` | 内容分区间距 | `padding`、`margin`、`display: grid` |
| `<article>` | 独立卡片或文章 | `padding`、`border`、`border-radius`、`box-shadow` |
| `<aside>` | 侧栏内容 | `width`、`flex`、`background-color` |
| `<footer>` | 页脚布局与辅助文字 | `padding`、`color`、`text-align` |
| `<h1>`–`<h6>` | 标题字号、行高、间距 | `font-size`、`font-weight`、`line-height`、`margin-block` |
| `<p>` | 正文字体与段落间距 | `font-size`、`line-height`、`margin-block` |
| `<a>` | 链接颜色、下划线、状态变化 | `color`、`text-decoration`、`:hover`、`:focus-visible` |
| `<strong>` | 重要性强调 | `font-weight`、`color` |
| `<em>` | 语气强调 | `font-style`、`color` |
| `<ul>` / `<ol>` | 列表间距和项目符号 | `padding-inline-start`、`list-style`、`display: grid`、`gap` |
| `<li>` | 列表项间距 | `margin`、`padding`、`line-height` |
| `<img>` | 响应式尺寸、裁剪、圆角 | `display`、`width`、`height`、`max-width`、`object-fit`、`border-radius` |
| `<figure>` | 图文组合 | `margin`、`display: grid`、`gap` |
| `<figcaption>` | 图片说明 | `font-size`、`color`、`text-align` |
| `<table>` | 表格宽度和边框合并 | `width`、`border-collapse`、`table-layout` |
| `<th>` / `<td>` | 单元格边距、对齐、边框 | `padding`、`text-align`、`border` |
| `<form>` | 表单布局 | `display: grid`、`gap`、`max-width` |
| `<label>` | 标签排版 | `display`、`font-weight`、`margin-block-end` |
| `<input>` / `<textarea>` / `<select>` | 输入控件外观与焦点状态 | `width`、`padding`、`border`、`border-radius`、`:focus-visible` |
| `<button>` | 按钮尺寸、颜色和状态 | `padding`、`border`、`background-color`、`cursor`、`:hover` |
| `<details>` / `<summary>` | 折叠内容和摘要 | `padding`、`border`、`cursor`、`::marker` |
| `<dialog>` | 原生对话框 | `padding`、`border`、`border-radius`、`box-shadow`、`::backdrop` |

### 全局基础样式示例

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  font-size: 16px;
}

body {
  margin: 0;
  color: #1f2937;
  background-color: #f8fafc;
  font-family: "Noto Sans SC", "Microsoft YaHei", sans-serif;
  line-height: 1.7;
}

img {
  display: block;
  max-width: 100%;
  height: auto;
}

a {
  color: #2563eb;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}
```

## 常用属性分类速查

### 布局与显示

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `display` | `block`、`inline`、`inline-block`、`flex`、`grid`、`none` | 设置元素的显示与布局方式 |
| `position` | `static`、`relative`、`absolute`、`fixed`、`sticky` | 设置定位方式 |
| `top` / `right` / `bottom` / `left` | 长度、百分比、`auto` | 偏移定位元素 |
| `inset` | `0`、`10px auto` | 同时设置四个方向的定位值 |
| `z-index` | 整数、`auto` | 控制层叠顺序 |
| `float` | `left`、`right`、`none` | 让元素浮动，现代布局应谨慎使用 |
| `clear` | `left`、`right`、`both` | 清除浮动影响 |
| `overflow` | `visible`、`hidden`、`auto`、`scroll` | 控制超出内容如何处理 |
| `visibility` | `visible`、`hidden` | 控制可见性，元素仍占空间 |
| `opacity` | `0`–`1` | 设置整体透明度 |
| `aspect-ratio` | `16 / 9`、`1 / 1` | 设置首选宽高比 |

`display: none` 不显示元素且不占空间；`visibility: hidden` 隐藏元素但仍占空间。

### 盒模型与尺寸

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `width` / `height` | 长度、百分比、`auto` | 设置宽高 |
| `min-width` / `max-width` | 长度、百分比、`none` | 设置宽度约束 |
| `min-height` / `max-height` | 长度、百分比、`none` | 设置高度约束 |
| `box-sizing` | `content-box`、`border-box` | 决定宽高是否包含内边距和边框 |
| `padding` | 长度、百分比 | 设置内边距 |
| `margin` | 长度、百分比、`auto` | 设置外边距 |
| `border` | 宽度、样式、颜色 | 设置边框 |
| `border-radius` | 长度、百分比 | 设置圆角 |
| `outline` | 宽度、样式、颜色 | 设置轮廓，不占布局空间 |
| `box-shadow` | 偏移、模糊、扩散、颜色 | 设置盒子阴影 |

推荐统一设置：

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

盒模型简写按顺时针方向理解：

```css
padding: 上 右 下 左;
margin: 上下 左右;
border-width: 上 右 下 左;
```

逻辑方向属性更利于国际化布局：

```css
padding-block: 12px;
padding-inline: 20px;
margin-inline: auto;
```

### 字体与文本

| 属性                | 常用值                                  | 作用         |
| ----------------- | ------------------------------------ | ---------- |
| `font-family`     | 字体族列表、`sans-serif`                   | 设置字体       |
| `font-size`       | `16px`、`1rem`、`clamp(...)`           | 设置字号       |
| `font-weight`     | `400`、`700`、`normal`、`bold`          | 设置字重       |
| `font-style`      | `normal`、`italic`                    | 设置字体样式     |
| `line-height`     | 无单位数值、长度                             | 设置行高       |
| `letter-spacing`  | 长度                                   | 设置字符间距     |
| `word-spacing`    | 长度                                   | 设置单词间距     |
| `text-align`      | `left`、`center`、`right`、`justify`    | 设置行内内容水平对齐 |
| `text-decoration` | `none`、`underline`、`line-through`    | 设置文本装饰线    |
| `text-transform`  | `uppercase`、`lowercase`、`capitalize` | 转换英文字母大小写  |
| `text-indent`     | 长度                                   | 设置首行缩进     |
| `text-overflow`   | `ellipsis`、`clip`                    | 设置溢出文本显示方式 |
| `white-space`     | `normal`、`nowrap`、`pre-wrap`         | 控制空白和换行    |
| `overflow-wrap`   | `normal`、`break-word`、`anywhere`     | 控制长单词换行    |
| `word-break`      | `normal`、`break-all`                 | 控制单词断行     |
| `color`           | 颜色值                                  | 设置文字颜色     |

文本单行省略：

```css
.ellipsis {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
```

多行省略：

```css
.line-clamp {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
```

多行省略涉及浏览器兼容和可访问性，重要内容不应被不可逆隐藏。

### 颜色、背景与渐变

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `color` | 颜色值 | 设置文字颜色 |
| `background-color` | 颜色值、`transparent` | 设置背景色 |
| `background-image` | URL、渐变 | 设置背景图片或渐变 |
| `background-repeat` | `repeat`、`no-repeat` | 设置背景重复方式 |
| `background-position` | 关键字、长度、百分比 | 设置背景位置 |
| `background-size` | `cover`、`contain`、长度 | 设置背景尺寸 |
| `background` | 多个背景属性组合 | 背景简写 |
| `accent-color` | 颜色值 | 设置复选框、单选框等强调色 |
| `caret-color` | 颜色值 | 设置输入光标颜色 |

颜色写法：

```css
.example {
  color: #1f2937;
  background-color: #f8fafc;
  border-color: rgb(37 99 235);
  box-shadow: 0 0 0 4px rgb(37 99 235 / 18%);
}
```

常见颜色格式：

- 颜色关键字：`red`、`transparent`、`currentColor`
- HEX：`#ffffff`、`#333`
- RGB / RGBA：`rgb(0 0 0)`、`rgb(0 0 0 / 50%)`
- HSL：`hsl(210 80% 50%)`

渐变示例：

```css
.banner {
  background-image: linear-gradient(135deg, #2563eb, #7c3aed);
}
```

### 边框、轮廓与视觉效果

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `border` | `1px solid #d1d5db` | 边框简写 |
| `border-width` | 长度 | 边框宽度 |
| `border-style` | `solid`、`dashed`、`dotted`、`none` | 边框样式 |
| `border-color` | 颜色值 | 边框颜色 |
| `border-radius` | 长度、百分比 | 圆角 |
| `outline` | 宽度、样式、颜色 | 焦点或强调轮廓 |
| `outline-offset` | 长度 | 轮廓与元素之间的距离 |
| `box-shadow` | `x y blur spread color` | 盒子阴影 |
| `text-shadow` | `x y blur color` | 文字阴影 |
| `opacity` | `0`–`1` | 整体透明度 |
| `filter` | `blur()`、`brightness()`、`grayscale()` | 图像滤镜 |
| `backdrop-filter` | `blur()`、`saturate()` | 背景滤镜，需注意兼容性 |
| `cursor` | `pointer`、`not-allowed`、`grab` | 鼠标指针样式 |

推荐始终保留可见焦点：

```css
:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
}
```

## Flexbox 常用属性

Flexbox 适合一维排列：一行或一列。

```css
.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}
```

容器属性：

| 属性                | 常用值                                                                 | 作用         |
| ----------------- | ------------------------------------------------------------------- | ---------- |
| `display`         | `flex`、`inline-flex`                                                | 建立 Flex 容器 |
| `flex-direction`  | `row`、`column`、`row-reverse`、`column-reverse`                       | 设置主轴方向     |
| `justify-content` | `flex-start`、`center`、`space-between`、`space-around`、`space-evenly` | 主轴对齐       |
| `align-items`     | `stretch`、`flex-start`、`center`、`flex-end`、`baseline`               | 单行交叉轴对齐    |
| `align-content`   | 多行交叉轴分布                                                             | 多行整体对齐     |
| `flex-wrap`       | `nowrap`、`wrap`、`wrap-reverse`                                      | 是否换行       |
| `gap`             | 长度                                                                  | 设置项目间距     |
| `place-content`   | `align-content justify-content`                                     | 同时设置内容分布   |

项目属性：

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `flex` | `1`、`0 1 auto`、`1 1 240px` | 简写三个伸缩属性 |
| `flex-grow` | 数字 | 剩余空间分配比例 |
| `flex-shrink` | 数字 | 空间不足时收缩比例 |
| `flex-basis` | `auto`、长度、百分比 | 初始主轴尺寸 |
| `align-self` | `auto`、`center`、`flex-start`、`flex-end` | 单独设置交叉轴对齐 |
| `order` | 整数 | 调整显示顺序 |
| `min-width` | `0` 等 | 防止 Flex 子项无法收缩 |

常用写法：

```css
.item {
  flex: 1 1 240px;
  min-width: 0;
}
```

## Grid 常用属性

Grid 适合二维布局：同时控制行和列。

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 20px;
}
```

容器属性：

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `display` | `grid`、`inline-grid` | 建立 Grid 容器 |
| `grid-template-columns` | `1fr 2fr`、`repeat(...)` | 定义列 |
| `grid-template-rows` | `auto 1fr`、`repeat(...)` | 定义行 |
| `grid-template-areas` | 区域名称网格 | 定义命名布局区域 |
| `grid-auto-columns` | 长度、`auto` | 自动生成列的尺寸 |
| `grid-auto-rows` | 长度、`auto` | 自动生成行的尺寸 |
| `grid-auto-flow` | `row`、`column`、`dense` | 自动放置方向 |
| `gap` | 长度 | 行、列间距 |
| `align-items` | 单元格在行轴上的对齐 | 网格项目对齐 |
| `justify-items` | 单元格在列轴上的对齐 | 网格项目对齐 |
| `place-items` | `align-items justify-items` | 同时设置两个方向 |
| `justify-content` | 整个网格在容器中的水平分布 | 网格整体对齐 |
| `align-content` | 整个网格在容器中的垂直分布 | 网格整体对齐 |

项目属性：

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `grid-column` | `1 / 3`、`span 2` | 设置项目跨列 |
| `grid-row` | `1 / 3`、`span 2` | 设置项目跨行 |
| `grid-area` | 区域名称或四个网格线 | 设置项目位置或命名区域 |
| `align-self` | 单个项目行轴对齐 | 覆盖容器设置 |
| `justify-self` | 单个项目列轴对齐 | 覆盖容器设置 |
| `place-self` | 同时设置两个方向 | 组合简写 |

常用长度函数：

```css
.layout {
  display: grid;
  grid-template-columns: 240px minmax(0, 1fr);
  gap: 24px;
}
```

## 定位与层叠

| `position` | 是否脱离文档流 | 定位参照 | 常见用途 |
| --- | --- | --- | --- |
| `static` | 否 | 正常文档流 | 默认状态 |
| `relative` | 否 | 自身原位置 | 微调或作为定位祖先 |
| `absolute` | 是 | 最近的非 `static` 定位祖先 | 角标、浮层定位 |
| `fixed` | 是 | 视口 | 固定工具栏、悬浮按钮 |
| `sticky` | 否 | 最近的滚动容器 | 粘性页头、目录 |

```css
.card {
  position: relative;
}

.badge {
  position: absolute;
  top: 8px;
  right: 8px;
}
```

定位属性：

- `top`、`right`、`bottom`、`left`
- `inset` 及其方向变体
- `z-index`
- `position`

`z-index` 会受到层叠上下文影响，不是数值越大就一定能覆盖所有元素。

## 动画、过渡与变换

### transition

```css
.button {
  background-color: #2563eb;
  transition: background-color 180ms ease;
}

.button:hover {
  background-color: #1d4ed8;
}
```

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `transition-property` | 属性名、`all` | 参与过渡的属性 |
| `transition-duration` | `180ms`、`0.2s` | 持续时间 |
| `transition-timing-function` | `ease`、`ease-out`、`cubic-bezier(...)` | 缓动函数 |
| `transition-delay` | 时间 | 延迟时间 |
| `transition` | 多组简写 | 组合设置 |

不建议无差别使用 `transition: all`，它可能让意外变化的属性也产生动画。

### transform

```css
.card:hover {
  transform: translateY(-4px) scale(1.02);
}
```

常用变换：

| 函数 | 作用 |
| --- | --- |
| `translate()` / `translateX()` / `translateY()` | 平移 |
| `scale()` / `scaleX()` / `scaleY()` | 缩放 |
| `rotate()` | 旋转 |
| `skew()` | 倾斜 |
| `matrix()` | 使用矩阵进行复杂变换 |

`transform` 通常比修改 `top`、`left`、`width`、`height` 更适合动画。

### animation

```css
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(12px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message {
  animation: fade-in 400ms ease-out both;
}
```

| 属性 | 常用值 | 作用 |
| --- | --- | --- |
| `animation-name` | 关键帧名称 | 使用的动画 |
| `animation-duration` | 时间 | 持续时间 |
| `animation-timing-function` | `ease`、`linear` | 缓动 |
| `animation-delay` | 时间 | 延迟 |
| `animation-iteration-count` | 数字、`infinite` | 播放次数 |
| `animation-direction` | `normal`、`reverse`、`alternate` | 播放方向 |
| `animation-fill-mode` | `none`、`forwards`、`backwards`、`both` | 动画前后的状态 |
| `animation-play-state` | `running`、`paused` | 播放或暂停 |

尊重用户减少动态效果的设置：

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## 响应式与函数

### 媒体查询

```css
.card-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}

@media (min-width: 640px) {
  .card-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (min-width: 1024px) {
  .card-grid {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}
```

移动优先：先写小屏基础样式，再用 `min-width` 逐级增强。

### 常用函数

| 函数 | 作用 | 示例 |
| --- | --- | --- |
| `var()` | 读取 CSS 自定义属性 | `color: var(--color-primary);` |
| `calc()` | 混合计算不同单位 | `width: calc(100% - 32px);` |
| `min()` | 取较小值 | `width: min(100% - 32px, 1100px);` |
| `max()` | 取较大值 | `width: max(320px, 50%);` |
| `clamp()` | 限制在最小值和最大值之间 | `font-size: clamp(1.5rem, 4vw, 2.5rem);` |
| `rgb()` / `hsl()` | 定义颜色 | `rgb(37 99 235 / 50%)` |
| `linear-gradient()` | 线性渐变 | `linear-gradient(135deg, red, blue)` |
| `url()` | 引用外部资源 | `background-image: url("./bg.webp");` |

### CSS 自定义属性

```css
:root {
  --color-primary: #2563eb;
  --color-text: #1f2937;
  --space-4: 16px;
  --radius: 12px;
}

.button {
  color: white;
  background-color: var(--color-primary);
  padding: var(--space-4);
  border-radius: var(--radius);
}
```

自定义属性可以集中管理颜色、间距、圆角和主题，也可以在不同选择器内覆盖。

### 常用 at-rule

| 规则 | 作用 |
| --- | --- |
| `@media` | 根据视口、设备或用户偏好应用样式 |
| `@supports` | 浏览器支持某属性时应用样式 |
| `@container` | 根据父容器尺寸应用样式 |
| `@keyframes` | 定义动画关键帧 |
| `@font-face` | 引入字体文件 |
| `@layer` | 声明层叠层，管理大型项目的级联顺序 |
| `@property` | 注册自定义属性的类型和默认值 |

## 常用属性值速查

### 尺寸单位

| 单位 | 含义 |
| --- | --- |
| `px` | CSS 像素，界面中常见 |
| `%` | 相对于父元素或当前上下文 |
| `em` | 相对于当前元素字号 |
| `rem` | 相对于根元素字号 |
| `vw` / `vh` | 视口宽度 / 高度的 1% |
| `dvw` / `dvh` | 动态视口宽度 / 高度，适合移动端 |
| `ch` | 当前字体中数字 `0` 的大致宽度 |
| `fr` | Grid 容器中的剩余空间份额 |
| `deg` | 角度，适合旋转 |
| `s` / `ms` | 秒 / 毫秒，适合动画和过渡 |

### 常用关键字

| 关键字 | 含义 |
| --- | --- |
| `inherit` | 继承父元素计算值 |
| `initial` | 使用属性的初始值 |
| `unset` | 可继承属性表现为继承，否则表现为初始值 |
| `revert` | 回退到浏览器或用户样式层 |
| `auto` | 由浏览器自动计算 |
| `none` | 不使用对应效果或值 |
| `currentColor` | 使用当前元素的 `color` 值 |
| `transparent` | 完全透明 |
| `important` | 在声明末尾提高重要性，应谨慎使用 |

## 层叠、优先级与继承

浏览器决定最终样式时，会综合：

1. 重要性：带 `!important` 的声明通常更强。
2. 来源与层：浏览器、用户、作者样式，以及 `@layer`。
3. 选择器优先级。
4. 源码顺序：相同优先级时，后出现者生效。

优先级粗略顺序：

```text
行内样式
ID 选择器
类、属性选择器、伪类
元素、伪元素
```

示例：

```css
p { color: black; }       /* 0-0-1 */
.text { color: blue; }    /* 0-1-0 */
#intro { color: red; }    /* 1-0-0 */
```

减少样式冲突的做法：

- 维持较低且稳定的选择器优先级。
- 优先使用类选择器。
- 不滥用 ID 选择器和 `!important`。
- 将可继承属性放在 `body` 等高层元素上。
- 使用 CSS 自定义属性管理主题值。
- 大型项目可考虑 `@layer`，但不要为了简单页面过度设计。

## 常见组件速查

### 按钮

```css
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 44px;
  padding: 10px 18px;
  border: 1px solid transparent;
  border-radius: 10px;
  color: #ffffff;
  background-color: #2563eb;
  font: inherit;
  cursor: pointer;
  transition:
    background-color 180ms ease,
    transform 180ms ease;
}

.button:hover {
  background-color: #1d4ed8;
}

.button:active {
  transform: translateY(1px);
}

.button:focus-visible {
  outline: 3px solid #93c5fd;
  outline-offset: 2px;
}

.button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}
```

### 卡片

```css
.card {
  padding: 20px;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  background-color: #ffffff;
  box-shadow: 0 8px 24px rgb(15 23 42 / 8%);
}

.card h2 {
  margin-block: 0 8px;
  font-size: 1.25rem;
}

.card p {
  margin-block: 0;
  color: #475569;
  line-height: 1.7;
}
```

### 响应式图片

```css
img {
  display: block;
  width: 100%;
  max-width: 100%;
  height: auto;
}

.avatar {
  width: 120px;
  aspect-ratio: 1;
  object-fit: cover;
  border-radius: 50%;
}
```

### 表单控件

```css
input,
textarea,
select {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  background-color: #ffffff;
  font: inherit;
}

input:focus-visible,
textarea:focus-visible,
select:focus-visible {
  border-color: #2563eb;
  outline: 3px solid rgb(37 99 235 / 18%);
  outline-offset: 1px;
}

input:disabled {
  background-color: #f1f5f9;
  cursor: not-allowed;
}
```

## 常见错误与建议

- 只写属性，不理解它作用于哪个标签和布局上下文。
- 类选择器忘记写 `.`。
- 声明末尾漏写分号。
- 用 `!important` 掩盖选择器冲突，而不是调整结构。
- 所有尺寸都写死为 `px`，导致响应式困难。
- 使用固定高度承载可能增多的内容。
- 图片没有设置 `max-width: 100%`，小屏出现横向滚动。
- 使用 `outline: none` 删除焦点后，没有提供可见替代。
- 用 `transition: all` 造成不可控动画。
- 依赖 `:hover` 表示唯一操作状态，触屏设备可能无法触发。
- 用 `float` 和绝对定位承担本应由 Flexbox、Grid 或正常文档流处理的布局。
- 只按桌面宽度测试，未检查窄屏、长文本和放大后的页面。

> [!warning] 最重要的实践原则
> 先保证结构和内容正确，再写布局；先使用正常文档流、Flexbox 和 Grid，再考虑浮动与绝对定位；优先使用可维护、可访问的选择器和属性。

## 相关笔记

- [[01-CSS 基础总览]]
- [[01-选择器基础]]
- [[01-盒模型基础]]
- [[01-布局基础]]
- [[01-响应式基础]]
- [[01-动画与过渡基础]]