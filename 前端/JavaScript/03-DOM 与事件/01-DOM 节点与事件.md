---
title: DOM 节点与事件
tags:
  - 前端
  - JavaScript
  - DOM
  - 事件
---

# DOM 节点与事件

> [!source] 课程对照
> `html.pdf` 第 33–34、44–47 页。

## 什么是 DOM

浏览器把 HTML 文档解析为一棵 DOM 树。JavaScript 可以通过 DOM API 查找、读取、修改、新增和删除页面元素。

## 获取节点

课程中的常用方法：

```js
document.getElementById("username");
document.getElementsByName("gender");
document.getElementsByTagName("input");
document.getElementsByClassName("item");
```

- `getElementById()`：通过 `id` 返回一个元素；方法名中的 `Element` 是单数。
- `getElementsByName()`：通过 `name` 返回节点集合。
- `getElementsByTagName()`：通过标签名返回集合。
- `getElementsByClassName()`：通过类名返回集合。
- 集合通常是动态集合，遍历和修改时要注意数量变化。

现代写法：

```js
document.querySelector("#username");
document.querySelectorAll(".item");
```

- `querySelector()` 返回第一个匹配元素。
- `querySelectorAll()` 返回静态 NodeList，可以使用 `forEach()`。
- 选择器语法与 CSS 相同。

## 读取和修改内容

```js
const title = document.getElementById("title");

console.log(title.textContent);
title.textContent = "新标题";
title.innerHTML = "<strong>加粗标题</strong>";
```

- `textContent`：读取或设置纯文本，避免 HTML 注入风险。
- `innerHTML`：读取或设置 HTML 字符串，插入不可信内容时存在 XSS 风险。
- `innerText`：受 CSS 显示状态影响，性能和行为与 `textContent` 不完全相同。

## 修改样式和类名

```js
const box = document.getElementById("box");

box.style.display = "none";
box.style.color = "red";
box.classList.add("is-active");
box.classList.remove("is-hidden");
box.classList.toggle("is-active");
```

优先通过 `classList` 切换类名，再在 CSS 中定义样式，避免在 JavaScript 中堆叠大量行内样式。

## 表单状态

```js
const checkbox = document.querySelector("#agree");
checkbox.checked = true;

const input = document.querySelector("#username");
console.log(input.value);
```

- `value`：输入框、下拉框的值。
- `checked`：单选框和复选框是否选中。
- `disabled`：是否禁用。
- `dataset`：读取 `data-*` 自定义数据。

## 常见事件

| 事件 | 触发时机 |
| --- | --- |
| `onchange` | 元素值改变并确认后 |
| `onclick` | 点击元素 |
| `onmouseover` | 鼠标移入 |
| `onmouseout` | 鼠标移出 |
| `onkeydown` | 按下键盘按键 |
| `onload` | 页面或资源加载完成 |
| `focus` | 元素获得焦点 |
| `blur` | 元素失去焦点 |
| `submit` | 表单提交 |

## 绑定事件

### HTML 属性

```html
<button onclick="sayHello()">点击</button>
```

简单直接，但 HTML 与 JavaScript 耦合，项目代码不推荐大量使用。

### DOM 属性

```js
const button = document.querySelector("button");

button.onclick = function () {
  console.log("点击");
};
```

同一个事件属性后绑定的处理函数会覆盖之前的处理函数。

### addEventListener

```js
button.addEventListener("click", function () {
  console.log("点击");
});
```

推荐使用 `addEventListener()`，它可以绑定多个处理函数，并支持捕获、移除监听和更清晰的事件对象。

## 页面加载时机

```js
window.addEventListener("DOMContentLoaded", () => {
  console.log("DOM 已解析完成");
});
```

```js
window.addEventListener("load", () => {
  console.log("页面及资源已加载完成");
});
```

- `DOMContentLoaded`：HTML DOM 已就绪。
- `load`：图片、样式等资源也加载完成。
- 外部脚本放在 `head` 时使用 `defer`，或把脚本放在 `body` 末尾。