---
title: JavaScript 基础与输出
tags:
  - 前端
  - JavaScript
  - 基础
---

# JavaScript 基础与输出

> [!source] 课程对照
> `html.pdf` 第 29–32 页。

## JavaScript 的定位

- HTML 定义网页内容和结构。
- CSS 控制页面布局和视觉样式。
- JavaScript 控制网页行为和交互。

JavaScript 是运行在浏览器中的脚本语言，可以读取和修改页面、响应用户事件、发送网络请求并进行数据处理。

## JavaScript 写在哪里

### 写在 HTML 属性中

```html
<button onclick="alert('你好')">点击</button>
```

适合非常小的演示，不利于维护和复用。

### 写在 script 标签中

```html
<script>
  console.log("页面已加载");
</script>
```

适合单页面演示或少量页面脚本。

### 写在独立的 .js 文件中

```html
<script src="./js/app.js" defer></script>
```

实际项目优先使用外部文件，实现 HTML、CSS、JavaScript 分离。

## 输出与调试

```js
window.alert("弹出提示");

document.write("直接写入文档");

document.getElementById("message").innerHTML = "写入元素";

console.log("输出到控制台");
```

- `alert()`：弹出警告框，会阻塞页面，不适合频繁使用。
- `document.write()`：在页面解析阶段可用，页面加载完成后使用可能覆盖整个文档，现代项目不推荐。
- `innerHTML`：写入元素的 HTML 内容，使用时注意 XSS 风险，不要直接插入不可信字符串。
- `console.log()`：输出到开发者工具 Console。
- 调试时还可以使用 `console.table()`、`console.error()`、断点等。

## JavaScript 语法基础

```js
// 单行注释

/*
  多行注释
*/

let message = "Hello";
console.log(message);
```

- JavaScript 区分大小写。
- `getElementById` 与 `getElementbyID` 不是同一个方法。
- `myVariable` 与 `MyVariable` 是两个不同的变量。
- JavaScript 使用 Unicode 字符集。
- 语句末尾通常使用分号；现代 JavaScript 会进行自动分号插入，但显式分号更清晰。
- 关键字不能作为普通变量名，例如 `if`、`for`、`function`、`return`、`class`、`const` 等。

## 常见关键字与控制语句

| 关键字 | 作用 |
| --- | --- |
| `if` / `else` | 根据条件执行不同代码 |
| `switch` | 在多个分支中选择 |
| `for` | 循环指定次数 |
| `while` | 条件为真时循环 |
| `do...while` | 至少执行一次，再判断条件 |
| `break` | 跳出循环或 `switch` |
| `continue` | 跳过当前迭代，继续下一次 |
| `function` | 定义函数 |
| `return` | 返回函数结果并结束函数 |
| `var` / `let` / `const` | 声明变量或常量 |
| `try` / `catch` / `throw` | 错误处理 |