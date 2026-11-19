---
title: HTML 常用标签与属性速查
tags:
  - 前端
  - HTML
  - 速查
aliases:
  - HTML 标签速查
  - HTML 属性速查
---

# HTML 常用标签与属性速查

> [!abstract] 如何使用本页
> 本页用于快速查标签、属性和适用场景。写页面时优先选择能表达内容含义的标签；属性只负责补充语义、状态或行为，视觉效果优先交给 CSS。

## 先记住 4 条规则

1. 标签表示“内容是什么”，属性补充“这个元素是什么状态、怎样工作”。
2. 属性分为全局属性和元素专用属性；全局属性大多数标签都能使用。
3. 布尔属性出现即生效，推荐直接写属性名，例如 `required`，不必写 `required="required"`。
4. 优先使用语义化标签和现代属性，避免使用只为控制外观的旧式属性。

```html
<a href="https://developer.mozilla.org/" target="_blank" rel="noopener noreferrer">
  MDN Web Docs
</a>
```

- `a`：标签名。
- `href`、`target`、`rel`：元素专用属性。
- `href="..."`：属性名和属性值。

## 常用标签速查

### 文档与元数据

| 标签 | 作用 | 常用属性 |
| --- | --- | --- |
| `<html>` | 文档根元素 | `lang`、`dir` |
| `<head>` | 保存元数据、标题和资源引用 | 全局属性 |
| `<title>` | 浏览器标签页标题，也是搜索结果标题 | 全局属性 |
| `<meta>` | 声明编码、视口、描述等元数据 | `charset`、`name`、`content`、`http-equiv`、`property` |
| `<link>` | 关联 CSS、图标、预加载等外部资源 | `rel`、`href`、`type`、`media`、`as`、`crossorigin` |
| `<style>` | 写入页面内部 CSS | `media`、`nonce` |
| `<script>` | 引入或编写 JavaScript | `src`、`type`、`async`、`defer`、`crossorigin`、`integrity` |
| `<base>` | 设置相对 URL 和链接打开方式的基准 | `href`、`target` |
| `<body>` | 页面可见内容的主体 | 全局属性、`onload` 等事件属性 |

最小文档结构：

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>页面标题</title>
  </head>
  <body>
    <h1>页面主标题</h1>
  </body>
</html>
```

### 页面结构与语义

| 标签 | 作用 | 常用属性 |
| --- | --- | --- |
| `<header>` | 页面或某个区块的页头 | 全局属性 |
| `<nav>` | 主要导航链接区域 | `aria-label` |
| `<main>` | 页面唯一的主要内容区域 | 全局属性 |
| `<section>` | 有明确主题的内容分组 | `aria-labelledby` |
| `<article>` | 可独立分发或复用的完整内容 | 全局属性 |
| `<aside>` | 与主内容相关但可独立的补充内容 | 全局属性 |
| `<footer>` | 页面或区块的页脚 | 全局属性 |
| `<div>` | 无额外语义的块级容器 | 全局属性 |
| `<span>` | 无额外语义的行内容器 | 全局属性 |

选择建议：

- 能明确表达含义时，优先使用 `header`、`nav`、`main`、`section`、`article`、`aside`、`footer`。
- `div` 和 `span` 只适合没有更合适语义标签的容器场景。
- `main` 通常每个页面只出现一次。

### 标题、段落与文本

| 标签 | 作用 | 常用属性 |
| --- | --- | --- |
| `<h1>`–`<h6>` | 六级标题，数字越小层级越高 | 全局属性 |
| `<p>` | 段落 | 全局属性 |
| `<br>` | 行内换行，不用于制造段落间距 | 全局属性 |
| `<hr>` | 主题分隔 | 全局属性 |
| `<strong>` | 表示重要性 | 全局属性 |
| `<em>` | 表示语气强调 | 全局属性 |
| `<b>` | 引起注意，不强调重要性 | 全局属性 |
| `<i>` | 表示另一种语气或术语，常见于图标字体 | 全局属性 |
| `<mark>` | 标记与当前上下文相关的内容 | 全局属性 |
| `<small>` | 旁注、法律声明等小号内容 | 全局属性 |
| `<del>` | 已删除内容 | `cite`、`datetime` |
| `<ins>` | 已插入内容 | `cite`、`datetime` |
| `<sub>` | 下标 | 全局属性 |
| `<sup>` | 上标 | 全局属性 |
| `<code>` | 代码片段 | 全局属性 |
| `<pre>` | 保留空格和换行的预格式化文本 | 全局属性 |
| `<kbd>` | 键盘输入 | 全局属性 |
| `<samp>` | 程序输出示例 | 全局属性 |
| `<var>` | 变量名 | 全局属性 |
| `<blockquote>` | 块级引用 | `cite` |
| `<q>` | 行内引用 | `cite` |
| `<cite>` | 作品标题或引用来源 | 全局属性 |
| `<abbr>` | 缩写或首字母缩略词 | `title` |
| `<time>` | 日期或时间 | `datetime` |
| `<dfn>` | 术语定义 | `title` |
| `<address>` | 联系信息 | 全局属性 |

语义选择：

- `strong` / `em` 表示语义，`b` / `i` 主要表示视觉区分。
- `br` 只用于确实需要换行的文本，不应用来布局。
- `pre` 中写代码时通常和 `code` 搭配。

### 链接、图片与多媒体

| 标签 | 作用 | 常用属性 |
| --- | --- | --- |
| `<a>` | 超链接、锚点、邮件或电话链接 | `href`、`target`、`rel`、`download`、`hreflang`、`type`、`referrerpolicy` |
| `<img>` | 嵌入图片 | `src`、`alt`、`width`、`height`、`srcset`、`sizes`、`loading`、`decoding`、`fetchpriority` |
| `<picture>` | 根据条件选择不同图片 | 全局属性 |
| `<source>` | 为 `picture`、`audio`、`video` 提供候选资源 | `src`、`srcset`、`sizes`、`type`、`media` |
| `<figure>` | 独立插图、代码、表格等 | 全局属性 |
| `<figcaption>` | `figure` 的标题或说明 | 全局属性 |
| `<audio>` | 音频播放器 | `src`、`controls`、`autoplay`、`loop`、`muted`、`preload` |
| `<video>` | 视频播放器 | `src`、`controls`、`poster`、`width`、`height`、`playsinline`、`preload` |
| `<track>` | 为音视频添加字幕或章节 | `src`、`kind`、`srclang`、`label`、`default` |
| `<iframe>` | 嵌入另一个页面 | `src`、`srcdoc`、`title`、`loading`、`sandbox`、`allow`、`referrerpolicy` |
| `<canvas>` | JavaScript 绘制图形的画布 | `width`、`height` |
| `<svg>` | 嵌入 SVG 矢量图形 | `viewBox`、`width`、`height`、`fill`、`stroke` |
| `<map>` | 图片映射区域集合 | `name` |
| `<area>` | 图片映射中的可点击区域 | `shape`、`coords`、`href`、`alt`、`target` |

链接示例：

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">外部链接</a>
<a href="#contact">跳到联系方式</a>
<a href="mailto:hello@example.com">发送邮件</a>
<a href="./files/report.pdf" download>下载报告</a>
```

图片示例：

```html
<img
  src="./images/avatar.webp"
  alt="张三的头像"
  width="240"
  height="240"
  loading="lazy"
  decoding="async"
/>
```

关键属性：

- `href`：链接目标；`target="_blank"` 在新标签页打开。
- `rel="noopener noreferrer"`：外链新窗口打开时的安全建议。
- `src`：资源地址；`alt`：图片替代文本和无障碍信息。
- `loading="lazy"`：延后加载非首屏图片；不要滥用于首屏主图。
- `controls`：显示音视频播放控件；`poster`：视频封面。
- `title`：`iframe` 的无障碍名称，不应省略。

### 列表

| 标签 | 作用 | 常用属性 |
| --- | --- | --- |
| `<ul>` | 无序列表 | 全局属性 |
| `<ol>` | 有序列表 | `start`、`reversed`、`type` |
| `<li>` | 列表项 | `value` |
| `<dl>` | 描述列表 | 全局属性 |
| `<dt>` | 描述列表中的名称或术语 | 全局属性 |
| `<dd>` | 对 `dt` 的描述 | 全局属性 |

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
</ul>

<ol start="3" reversed>
  <li>第三步</li>
  <li>第二步</li>
  <li>第一步</li>
</ol>

<dl>
  <dt>HTML</dt>
  <dd>负责内容结构。</dd>
</dl>
```

`ul` 的 `type` 和列表符号视觉效果建议使用 CSS 控制。

### 表格

| 标签 | 作用 | 常用属性 |
| --- | --- | --- |
| `<table>` | 二维数据表格 | 全局属性 |
| `<caption>` | 表格标题 | 全局属性 |
| `<thead>` | 表头区域 | 全局属性 |
| `<tbody>` | 表格主体区域 | 全局属性 |
| `<tfoot>` | 汇总或脚注区域 | 全局属性 |
| `<tr>` | 表格行 | 全局属性 |
| `<th>` | 表头单元格 | `scope`、`colspan`、`rowspan`、`headers`、`abbr` |
| `<td>` | 数据单元格 | `colspan`、`rowspan`、`headers` |
| `<colgroup>` | 表格列分组 | `span` |
| `<col>` | 定义表格列的属性 | `span` |

```html
<table>
  <caption>课程安排</caption>
  <thead>
    <tr>
      <th scope="col">课程</th>
      <th scope="col">时间</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">HTML</th>
      <td>周一</td>
    </tr>
  </tbody>
</table>
```

- `scope="col"`：表头作用于整列。
- `scope="row"`：表头作用于整行。
- `colspan="2"`：横向合并两列。
- `rowspan="2"`：纵向合并两行。
- `colspan` 和 `rowspan` 要同步删减占位单元格，避免表格错位。

### 表单与交互

| 标签           | 作用                   | 常用属性                                                                    |
| ------------ | -------------------- | ----------------------------------------------------------------------- |
| `<form>`     | 表单容器                 | `action`、`method`、`enctype`、`target`、`name`、`autocomplete`、`novalidate` |
| `<label>`    | 表单控件的可点击标签           | `for`、`form`                                                            |
| `<input>`    | 单行输入控件，类型由 `type` 决定 | `type`、`name`、`value`、`required`、`disabled`、`readonly`、`placeholder` 等  |
| `<textarea>` | 多行文本输入               | `name`、`rows`、`cols`、`maxlength`、`placeholder`、`wrap`                   |
| `<button>`   | 按钮                   | `type`、`name`、`value`、`form`、`formaction`、`disabled`                    |
| `<select>`   | 下拉选择框                | `name`、`multiple`、`size`、`required`、`disabled`                          |
| `<option>`   | 下拉选项                 | `value`、`selected`、`disabled`、`label`                                   |
| `<optgroup>` | 对下拉选项分组              | `label`、`disabled`                                                      |
| `<datalist>` | 提供可输入的建议列表           | 全局属性                                                                    |
| `<output>`   | 计算结果输出区域             | `for`、`form`、`name`                                                     |
| `<progress>` | 任务进度条                | `value`、`max`                                                           |
| `<meter>`    | 已知范围内的度量值            | `value`、`min`、`max`、`low`、`high`、`optimum`                              |
| `<fieldset>` | 将相关表单控件分组            | `name`、`disabled`、`form`                                                |
| `<legend>`   | `fieldset` 的分组标题     | 全局属性                                                                    |
| `<details>`  | 可展开和折叠的内容            | `open`、`name`                                                           |
| `<summary>`  | `details` 的摘要或标题     | 全局属性                                                                    |
| `<dialog>`   | 对话框或弹窗               | `open`、`closedby`                                                       |

表单示例：

```html
<form action="/register" method="post" autocomplete="on">
  <fieldset>
    <legend>账号信息</legend>

    <label for="username">用户名</label>
    <input
      id="username"
      name="username"
      type="text"
      minlength="3"
      maxlength="20"
      autocomplete="username"
      required
    />

    <label for="email">邮箱</label>
    <input id="email" name="email" type="email" required />

    <button type="submit">注册</button>
  </fieldset>
</form>
```

表单要点：

- `action` 是提交地址，`method` 常用 `get` 或 `post`。
- 上传文件时通常使用 `enctype="multipart/form-data"`。
- 每个输入控件都应有可理解的 `label`。
- `name` 决定提交字段名；没有 `name` 的控件通常不会提交。
- 单选框使用相同的 `name` 组成互斥组。
- 控件在表单中时，`button` 可能默认是 `submit`，建议始终明确写 `type`。

## 常用 input 类型

| `type` | 用途 | 常用专属属性 |
| --- | --- | --- |
| `text` | 普通单行文本 | `minlength`、`maxlength`、`pattern`、`placeholder` |
| `password` | 密码输入 | `minlength`、`maxlength`、`autocomplete` |
| `email` | 邮箱输入 | `multiple`、`autocomplete` |
| `tel` | 电话号码 | `pattern`、`autocomplete` |
| `url` | URL 输入 | `pattern`、`placeholder` |
| `search` | 搜索框 | `list`、`autocomplete` |
| `number` | 数值输入 | `min`、`max`、`step` |
| `range` | 范围滑块 | `min`、`max`、`step`、`list` |
| `date` | 日期 | `min`、`max`、`step` |
| `time` | 时间 | `min`、`max`、`step` |
| `datetime-local` | 本地日期和时间 | `min`、`max`、`step` |
| `month` | 年月 | `min`、`max`、`step` |
| `week` | 年周 | `min`、`max`、`step` |
| `file` | 文件上传 | `accept`、`multiple`、`capture` |
| `checkbox` | 复选框 | `checked`、`value` |
| `radio` | 单选框 | `checked`、`value`、相同 `name` |
| `color` | 颜色选择器 | `value` |
| `hidden` | 不显示的数据字段 | `name`、`value` |
| `submit` | 提交按钮 | `formaction`、`formmethod`、`formnovalidate` |
| `reset` | 重置按钮 | `value` |
| `button` | 普通按钮 | `value` |

## 常用全局属性

全局属性可以用于绝大多数 HTML 元素。

| 属性 | 作用 | 示例 |
| --- | --- | --- |
| `id` | 文档内唯一标识 | `id="site-header"` |
| `class` | 一个或多个可复用类名 | `class="card card--featured"` |
| `style` | 行内 CSS，常用于临时测试 | `style="color: red"` |
| `title` | 鼠标悬停时的补充说明 | `title="更多信息"` |
| `lang` | 元素内容的语言 | `lang="zh-CN"` |
| `dir` | 文本方向 | `dir="rtl"` |
| `hidden` | 隐藏元素，布尔属性 | `hidden` |
| `tabindex` | 调整键盘焦点顺序 | `tabindex="0"` |
| `contenteditable` | 允许用户直接编辑内容 | `contenteditable="true"` |
| `draggable` | 是否允许拖拽 | `draggable="true"` |
| `spellcheck` | 是否进行拼写检查 | `spellcheck="false"` |
| `translate` | 是否允许翻译 | `translate="no"` |
| `accesskey` | 键盘快捷键 | `accesskey="s"` |
| `data-*` | 存放自定义数据 | `data-id="42"` |
| `role` | 补充 ARIA 角色 | `role="navigation"` |
| `aria-*` | 补充无障碍状态和关系 | `aria-label="主导航"` |
| `on*` | 事件处理属性 | `onclick="..."`，现代项目优先用 `addEventListener` |

`data-*` 读取示例：

```html
<button class="card" data-user-id="42">查看用户</button>

<script>
  const button = document.querySelector(".card");
  console.log(button.dataset.userId); // "42"
</script>
```

## 常用属性详解

### 链接与资源

| 属性 | 适用标签 | 说明 |
| --- | --- | --- |
| `href` | `a`、`link`、`area`、`base` | 链接或资源地址 |
| `target` | `a`、`base`、`form` | 在何处打开；`_blank` 表示新标签页 |
| `rel` | `a`、`link` | 当前文档与目标资源的关系 |
| `download` | `a`、`area` | 提示浏览器下载；有值时可作为文件名 |
| `src` | `img`、`script`、`iframe`、`audio`、`video`、`source` | 外部资源地址 |
| `alt` | `img`、`area` | 替代文本；装饰图片可使用 `alt=""` |
| `width` / `height` | `img`、`video`、`iframe`、`canvas` | 宽高，建议同时提供数值以减少布局跳动 |
| `srcset` / `sizes` | `img`、`source` | 响应式图片候选集合和尺寸条件 |
| `poster` | `video` | 视频封面图 |
| `loading` | `img`、`iframe` | `lazy` 表示延迟加载，`eager` 表示立即加载 |

### 表单与校验

| 属性 | 适用标签 | 说明 |
| --- | --- | --- |
| `name` | 表单控件 | 提交字段名 |
| `value` | 表单控件 | 控件值或按钮值 |
| `type` | `input`、`button` | 指定控件类型或按钮行为 |
| `required` | 多数输入控件 | 必填，布尔属性 |
| `disabled` | 表单控件 | 禁用、不可聚焦，通常不会提交 |
| `readonly` | 可文本输入控件 | 只读、可聚焦，仍会提交 |
| `placeholder` | 输入控件 | 输入提示，不能代替 `label` |
| `autocomplete` | `form`、输入控件 | 浏览器自动填充提示 |
| `pattern` | 文本类输入 | 正则校验规则 |
| `minlength` / `maxlength` | 文本类输入 | 文本长度限制 |
| `min` / `max` / `step` | 数值、日期、范围控件 | 数值范围和步长 |
| `multiple` | `input`、`select` | 允许多个值或文件 |
| `accept` | 文件输入 | 可接受的文件类型 |
| `list` | 输入控件 | 关联 `datalist` 的建议列表 |
| `checked` | 单选、复选框 | 默认选中，布尔属性 |
| `selected` | `option` | 默认选中，布尔属性 |
| `for` | `label`、`output` | 关联控件的 `id` |
| `form` | 表单控件 | 关联到指定表单的 `id` |
| `novalidate` | `form` | 关闭浏览器原生提交校验 |

### 表格与布局

| 属性 | 适用标签 | 说明 |
| --- | --- | --- |
| `colspan` | `th`、`td` | 横向跨列数 |
| `rowspan` | `th`、`td` | 纵向跨行数 |
| `scope` | `th` | 说明表头作用于列、行或分组 |
| `headers` | `td`、`th` | 关联多个表头单元格的 `id` |
| `span` | `col`、`colgroup` | 指定跨越的列数 |
| `open` | `details`、`dialog` | 展开或打开，布尔属性 |
| `aria-label` | 大多数元素 | 为无可见名称的控件或区域提供无障碍名称 |

### 音视频与嵌入

| 属性 | 适用标签 | 说明 |
| --- | --- | --- |
| `controls` | `audio`、`video` | 显示播放控件 |
| `autoplay` | `audio`、`video` | 自动播放，浏览器通常要求同时 `muted` |
| `muted` | `audio`、`video` | 默认静音 |
| `loop` | `audio`、`video` | 循环播放 |
| `preload` | `audio`、`video` | `none`、`metadata`、`auto` |
| `playsinline` | `video` | 移动端尽量内联播放 |
| `kind` / `srclang` | `track` | 字幕类型和语言 |
| `title` | `iframe` | 嵌入式页面的无障碍标题，必须提供 |
| `sandbox` | `iframe` | 限制嵌入页面的能力 |
| `allow` | `iframe` | 授权传感器、全屏等功能 |

## 常用布尔属性

布尔属性出现即表示 `true`，移除则表示 `false`。

| 属性 | 典型用途 |
| --- | --- |
| `hidden` | 隐藏元素 |
| `disabled` | 禁用控件 |
| `required` | 必填校验 |
| `readonly` | 只读 |
| `checked` | 默认勾选 |
| `selected` | 默认选中选项 |
| `multiple` | 允许多选 |
| `autofocus` | 页面加载时自动聚焦 |
| `novalidate` | 表单不执行原生校验 |
| `formnovalidate` | 当前提交按钮跳过校验 |
| `open` | 展开 `details` 或打开 `dialog` |
| `controls` | 显示音视频控件 |
| `autoplay` | 自动播放 |
| `muted` | 默认静音 |
| `loop` | 循环播放 |
| `default` | `track` 默认启用 |

推荐写法：

```html
<input type="checkbox" checked disabled />
```

## 常见选择与易错点

### `strong` 和 `b` 怎么选

- 内容重要、需要强调语义时使用 `<strong>`。
- 只是希望视觉加粗，但不强调重要性时，优先用 CSS；确实需要无语义加粗时可用 `<b>`。

### `em` 和 `i` 怎么选

- 强调语气使用 `<em>`。
- 术语、外语、技术名称或图标字体可使用 `<i>`；图标通常更推荐规范图标组件或 SVG。

### `article` 和 `section` 怎么选

- 内容可独立分发、复用或具有完整主题时使用 `<article>`。
- 只是一个有标题的内容分组时使用 `<section>`。
- 只是为了套样式时使用 `<div>`。

### `button` 和链接怎么选

- 触发当前页面操作、提交表单使用 `<button>`。
- 跳转地址、下载资源使用 `<a href="...">`。
- 不要给 `div` 或 `span` 随意绑定点击事件来冒充按钮。

### 图片的 `alt`

- 有信息价值的图片写清楚替代文本。
- 纯装饰图片使用 `alt=""`，让辅助技术忽略它。
- 不要把所有图片都写 `alt="图片"`，这没有提供有效信息。

### `disabled` 和 `readonly`

- `disabled`：控件禁用、不可聚焦，通常不会提交。
- `readonly`：内容不能编辑，但仍可聚焦，通常仍会提交。

### 尺寸与布局

- 图片同时提供 `width` 和 `height` 可以预留空间，减少布局跳动。
- `style` 只适合临时测试，正式样式放到 CSS 中。
- 不要让 `br` 或表格承担页面布局。

## 不建议继续使用的写法

| 旧写法 | 问题 | 现代替代 |
| --- | --- | --- |
| `font`、`center`、`big`、`strike` | 只控制外观，语义弱 | CSS、`strong`、`em`、`del` |
| `align`、`valign`、`bgcolor` | 旧式表现属性 | CSS `text-align`、`vertical-align`、`background` |
| 表格 `border`、`cellpadding`、`cellspacing` | 旧式表格样式 | CSS `border`、`padding`、`border-spacing` |
| 用 `<table>` 搭建整体页面布局 | 结构复杂、响应式困难 | CSS Grid、Flexbox |
| `<frameset>`、`<frame>` | 已过时，导航与无障碍问题多 | 语义化布局，必要时少量使用 `iframe` |
| 用多个 `<br>` 控制间距 | 间距依赖内容，难维护 | CSS `margin`、`padding`、`gap` |
| 无 `type` 的按钮 | 在表单中可能意外提交 | 明确写 `type="button"` 或 `type="submit"` |

## 一眼选标签

| 需求 | 首选标签 |
| --- | --- |
| 页面主体标题 | `<h1>` |
| 普通文字段落 | `<p>` |
| 重点内容 | `<strong>` |
| 强调语气 | `<em>` |
| 跳转链接 | `<a href="...">` |
| 展示图片 | `<img>` |
| 展示视频 | `<video>` |
| 多项并列内容 | `<ul>` |
| 有先后顺序的步骤 | `<ol>` |
| 名称和解释 | `<dl>` |
| 二维数据 | `<table>` |
| 用户输入 | `<form>` + `<input>` / `<select>` / `<textarea>` |
| 触发操作 | `<button>` |
| 可折叠内容 | `<details>` + `<summary>` |
| 无合适语义的内容容器 | `<div>` / `<span>` |

## 相关笔记

- [[01-HTML 基础总览]]
- [[01-文本、列表、链接与图片]]
- [[02-表格、容器与页面布局]]
- [[03-元数据、地图与内嵌框架]]
- [[01-表单基础]]
- [[02-HTML5 新特性与本地存储]]