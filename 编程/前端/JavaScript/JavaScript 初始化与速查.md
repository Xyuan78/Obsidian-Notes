---
title: JavaScript 初始化与速查
tags:
  - 前端
  - JavaScript
  - 初始化
  - 速查
aliases:
  - JS 初始化
  - JS 速查
  - JavaScript 启动页
---

# JavaScript 初始化与速查

> [!abstract] 页面用途
> 本页是 JavaScript 的启动入口，用于快速建立项目结构、回顾核心语法、连接 DOM、处理事件并查阅常用 API。系统学习仍以 [[00-JavaScript 学习索引]] 和各阶段笔记为主线。

## JavaScript 的职责

- HTML：页面的内容和结构。
- CSS：页面的布局和视觉样式。
- JavaScript：页面的行为、交互和数据处理。

```text
HTML → 页面有什么
CSS  → 页面看起来怎样
JS   → 页面做什么
```

## 最小可运行示例

`index.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JavaScript 初始化示例</title>
  </head>
  <body>
    <h1 id="title">JavaScript 初始化示例</h1>
    <button id="action">点击测试</button>

    <script src="./js/main.js" defer></script>
  </body>
</html>
```

`js/main.js`：

```js
const title = document.querySelector("#title");
const button = document.querySelector("#action");

button.addEventListener("click", () => {
  title.textContent = "按钮已经点击";
  console.log("JavaScript 已正常运行");
});
```

打开 `index.html` 后，点击按钮即可验证 JavaScript 是否加载成功。

## 推荐项目结构

```text
project/
├── index.html
├── css/
│   └── style.css
└── js/
    ├── main.js
    └── modules/
        └── utils.js
```

- `index.html`：页面结构与内容。
- `css/`：页面样式。
- `js/main.js`：应用入口和主要交互。
- `js/modules/`：可复用的 JavaScript 模块。
- 文件名使用清晰的小写英文，避免空格和含义不明的缩写。

## JavaScript 写在哪里

### 外部脚本

```html
<script src="./js/main.js" defer></script>
```

这是实际项目中最推荐的方式，便于缓存、复用和维护。

### 模块脚本

```html
<script type="module" src="./js/main.js"></script>
```

模块脚本默认延迟执行，可以使用 `import` 和 `export`。模块通常还会受到 CORS 限制，推荐通过本地开发服务器打开页面。

### 页面内脚本

```html
<script>
  console.log("页面已加载");
</script>
```

适合小型演示，不适合大型项目长期维护。

### 行内事件

```html
<button onclick="alert('你好')">点击</button>
```

HTML 和 JavaScript 强耦合，现代项目不推荐大量使用。

## 脚本加载时机

```html
<!-- 解析完 HTML 后、DOMContentLoaded 之前执行，推荐 -->
<script src="./js/main.js" defer></script>
```

```html
<!-- 放在 body 末尾，此时前面的 DOM 已可访问 -->
<script src="./js/main.js"></script>
</body>
```

```js
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM 已解析完成");
});

window.addEventListener("load", () => {
  console.log("页面和资源都已加载完成");
});
```

- `DOMContentLoaded`：HTML 文档结构已就绪。
- `load`：图片、样式等资源也加载完成。
- 使用 `defer` 时通常不需要再监听 `DOMContentLoaded` 才操作 DOM。

## 变量声明

```js
const siteName = "前端学习笔记";
let count = 0;

count += 1;
```

| 关键字 | 作用域 | 是否可重新赋值 | 建议 |
| --- | --- | --- | --- |
| `const` | 块级 | 否，但对象内容可修改 | 默认首选 |
| `let` | 块级 | 是 | 需要重新赋值时使用 |
| `var` | 函数级 | 是 | 旧式写法，新代码一般不优先使用 |

```js
const user = { name: "张三" };
user.name = "李四"; // 可以修改对象属性

// user = {}; // 错误：不能重新绑定 const
```

命名建议：

- 使用清晰、可读的英文名称，例如 `userName`、`isLoading`。
- 变量和函数通常使用小驼峰：`getUser()`、`totalPrice`。
- 类名通常使用大驼峰：`UserCard`、`ApiClient`。
- 常量、配置项可以使用大写下划线：`MAX_RETRY_COUNT`。
- 不要用 `a`、`data1`、`temp` 等含义模糊的名称表示重要数据。

## 数据类型

### 原始类型

| 类型 | 示例 | 说明 |
| --- | --- | --- |
| `string` | `"hello"`、`'你好'`、`` `JS` `` | 文本 |
| `number` | `10`、`3.14`、`NaN` | 数字 |
| `boolean` | `true`、`false` | 布尔值 |
| `undefined` | `undefined` | 已声明但未赋值 |
| `null` | `null` | 主动表示空值 |
| `symbol` | `Symbol("id")` | 唯一符号 |
| `bigint` | `9007199254740991n` | 大整数 |

### 对象类型

```js
const array = ["HTML", "CSS", "JavaScript"];
const object = { name: "张三", age: 18 };
const date = new Date();
const fn = () => console.log("函数");
```

数组、普通对象、日期和函数都属于对象体系的类型。

### 判断类型

```js
typeof "hello";       // "string"
typeof 10;            // "number"
typeof true;          // "boolean"
typeof undefined;     // "undefined"
typeof null;          // "object"，历史遗留行为
typeof [];            // "object"
Array.isArray([]);    // true
Number.isNaN(NaN);    // true
```

## 类型转换

```js
String(123);       // "123"
Number("123");     // 123
Number("abc");     // NaN
Boolean(0);        // false
Boolean("hello");  // true
parseInt("15.8");  // 15
parseFloat("15.8");// 15.8
```

隐式转换容易产生意外：

```js
"5" + 1;   // "51"
"5" - 1;   // 4
1 + true;  // 2
```

- `+` 遇到字符串时通常会拼接。
- `-`、`*`、`/` 通常会把值尝试转换成数字。
- 需要转换时，优先显式使用 `String()`、`Number()`、`Boolean()`。
- 用户输入通常都是字符串，计算前应转换为数字。
- 浮点数计算可能出现精度问题，例如 `0.1 + 0.2` 不等于精确的 `0.3`。金额等场景可使用整数分单位或专用十进制库。

## 运算符

```js
const a = 10;
const b = 3;

console.log(a + b); // 13
console.log(a - b); // 7
console.log(a * b); // 30
console.log(a / b); // 3.333...
console.log(a % b); // 1
console.log(a ** b); // 1000
```

比较和逻辑：

```js
console.log(1 === "1"); // false
console.log(1 !== "1"); // true
console.log(true && false); // false
console.log(true || false); // true
console.log(!true); // false
```

现代写法：

```js
const nickname = user.nickname ?? "匿名用户";
const city = user.address?.city;
```

- `===` / `!==`：严格比较，推荐使用。
- `==` / `!=`：会触发类型转换，容易产生意外。
- `??`：左侧为 `null` 或 `undefined` 时使用右侧值。
- `?.`：属性或方法不存在时安全返回 `undefined`。

## 条件与循环

### if

```js
const score = 85;

if (score >= 90) {
  console.log("优秀");
} else if (score >= 60) {
  console.log("及格");
} else {
  console.log("不及格");
}
```

### switch

```js
switch (status) {
  case "loading":
    console.log("加载中");
    break;

  case "success":
    console.log("成功");
    break;

  default:
    console.log("未知状态");
}
```

### for

```js
for (let i = 0; i < 5; i += 1) {
  console.log(i);
}
```

### for...of

```js
const colors = ["red", "green", "blue"];

for (const color of colors) {
  console.log(color);
}
```

### 遍历对象

```js
const user = { name: "张三", age: 18 };

for (const key in user) {
  if (Object.hasOwn(user, key)) {
    console.log(key, user[key]);
  }
}
```

或使用更明确的方法：

```js
Object.keys(user).forEach((key) => {
  console.log(key, user[key]);
});
```

`for...in` 会遍历可枚举属性名，不应直接用来遍历数组。

## 函数

### 函数声明

```js
function add(a, b) {
  return a + b;
}
```

### 函数表达式

```js
const subtract = function (a, b) {
  return a - b;
};
```

### 箭头函数

```js
const multiply = (a, b) => a * b;
const square = (number) => number * number;
```

箭头函数没有自己的 `this`、`arguments` 和 `super`。回调中需要保留外层 `this` 时，箭头函数通常更方便。

### 默认参数与剩余参数

```js
function createUser(name, role = "visitor", ...tags) {
  return { name, role, tags };
}

createUser("张三", "editor", "frontend", "active");
```

### 解构参数

```js
function renderCard({ title, description = "" }) {
  return `${title}: ${description}`;
}
```

## 作用域与闭包

```js
const globalValue = "全局";

function outer() {
  const outerValue = "外层";

  function inner() {
    console.log(globalValue);
    console.log(outerValue);
  }

  return inner;
}

const fn = outer();
fn();
```

- 内层作用域可以访问外层作用域。
- 外层作用域不能直接访问内层局部变量。
- 闭包是函数和其词法作用域的组合，常用于保存状态或创建工厂函数。
- 不要为了使用闭包而制造不必要的作用域嵌套。

## 数组

```js
const skills = ["HTML", "CSS", "JavaScript"];

console.log(skills[0]);
console.log(skills.length);

skills.push("TypeScript");
skills.pop();
```

### 常用数组方法

| 方法 | 作用 | 是否改变原数组 |
| --- | --- | --- |
| `push()` | 末尾添加 | 是 |
| `pop()` | 删除末尾 | 是 |
| `shift()` | 删除开头 | 是 |
| `unshift()` | 开头添加 | 是 |
| `slice()` | 截取一部分 | 否 |
| `splice()` | 删除、替换或插入 | 是 |
| `map()` | 映射为新数组 | 否 |
| `filter()` | 过滤为新数组 | 否 |
| `find()` | 找第一个匹配项 | 否 |
| `some()` | 是否至少一个满足 | 否 |
| `every()` | 是否全部满足 | 否 |
| `reduce()` | 汇总为单个值 | 否 |
| `sort()` | 排序 | 是 |

```js
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map((number) => number * 2);
const evenNumbers = numbers.filter((number) => number % 2 === 0);
const total = numbers.reduce((sum, number) => sum + number, 0);

console.log(doubled, evenNumbers, total);
```

数组解构：

```js
const [first, second, ...rest] = [10, 20, 30, 40];
console.log(first, second, rest);
```

## 对象

```js
const user = {
  name: "张三",
  age: 18,
  skills: ["HTML", "CSS"],
};

console.log(user.name);
console.log(user["age"]);
```

读取、添加和删除：

```js
user.city = "北京";
delete user.age;
console.log("name" in user);
```

对象简写与计算属性：

```js
const name = "张三";
const key = "role";

const person = {
  name,
  [key]: "admin",
};
```

对象解构：

```js
const { name, age = 0 } = user;
```

常用对象方法：

```js
Object.keys(user);       // 键名数组
Object.values(user);     // 值数组
Object.entries(user);    // 键值对数组
Object.hasOwn(user, "name");
Object.assign({}, user);
```

复制对象：

```js
const copy = { ...user };
const merged = { ...user, role: "editor" };
```

展开语法只进行浅复制，嵌套对象仍会共享引用。

## 字符串

```js
const text = "JavaScript";

console.log(text.length);
console.log(text.toUpperCase());
console.log(text.toLowerCase());
console.log(text.includes("Script"));
console.log(text.startsWith("Java"));
console.log(text.endsWith("Script"));
console.log(text.slice(0, 4));
console.log(text.replace("Java", "Type"));
console.log(text.split(""));
console.log(text.trim());
```

模板字符串：

```js
const name = "张三";
const score = 90;

console.log(`${name} 的分数是 ${score}`);
```

## DOM 初始化

DOM（Document Object Model，文档对象模型）把 HTML 解析为可编程的节点树。

```js
const title = document.querySelector("#title");
const items = document.querySelectorAll(".item");
```

| 方法 | 返回值 | 说明 |
| --- | --- | --- |
| `getElementById()` | 单个元素或 `null` | 通过 `id` 查找 |
| `querySelector()` | 第一个匹配元素或 `null` | 使用 CSS 选择器 |
| `querySelectorAll()` | 静态 NodeList | 返回全部匹配元素 |
| `getElementsByClassName()` | 动态集合 | 通过类名查找 |
| `getElementsByTagName()` | 动态集合 | 通过标签名查找 |

现代代码优先使用 `querySelector()` 和 `querySelectorAll()`，选择器规则与 CSS 一致。

### 读取与修改内容

```js
const title = document.querySelector("#title");

console.log(title.textContent);
title.textContent = "新标题";
```

```js
title.innerHTML = "<strong>加粗标题</strong>";
```

- `textContent`：读写纯文本，处理用户内容时优先使用。
- `innerHTML`：读写 HTML 字符串，插入不可信内容时存在 XSS 风险。
- `innerText`：受页面显示状态影响，行为与 `textContent` 不完全相同。

### 修改类与样式

```js
const card = document.querySelector(".card");

card.classList.add("is-active");
card.classList.remove("is-hidden");
card.classList.toggle("is-open");
console.log(card.classList.contains("is-active"));
```

```js
card.style.color = "red";
card.style.setProperty("--card-space", "16px");
```

优先通过类名切换状态，再在 CSS 中定义视觉样式。行内样式适合动态计算值或临时处理。

### 属性与数据

```js
const link = document.querySelector("a");

console.log(link.href);
link.setAttribute("aria-current", "page");
link.removeAttribute("aria-current");

const button = document.querySelector("button");
console.log(button.dataset.userId);
button.dataset.userId = "42";
```

`data-user-id="42"` 对应 `dataset.userId`。

### 创建、插入与删除节点

```js
const list = document.querySelector("#list");

const item = document.createElement("li");
item.textContent = "JavaScript";

list.append(item);
item.remove();
```

常用节点操作：

| 方法 | 作用 |
| --- | --- |
| `createElement()` | 创建元素 |
| `append()` / `prepend()` | 在末尾或开头插入节点 |
| `before()` / `after()` | 在元素前或后插入节点 |
| `replaceWith()` | 替换节点 |
| `remove()` | 删除节点 |
| `cloneNode()` | 复制节点 |

## 事件

### addEventListener

```js
const button = document.querySelector("#action");

button.addEventListener("click", (event) => {
  console.log("点击元素：", event.currentTarget);
});
```

同一个事件可以绑定多个监听器，推荐使用 `addEventListener()`。

### 常用事件

| 事件 | 触发时机 |
| --- | --- |
| `click` | 点击 |
| `dblclick` | 双击 |
| `input` | 输入内容变化 |
| `change` | 值变化并确认 |
| `submit` | 表单提交 |
| `keydown` / `keyup` | 键盘按下 / 松开 |
| `focus` / `blur` | 获得 / 失去焦点 |
| `mouseenter` / `mouseleave` | 鼠标进入 / 离开 |
| `pointerdown` / `pointerup` | 指针按下 / 松开 |
| `DOMContentLoaded` | HTML DOM 解析完成 |

### 阻止默认行为

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();
  console.log("改为由 JavaScript 处理提交");
});
```

### 事件委托

```js
list.addEventListener("click", (event) => {
  const button = event.target.closest("button[data-id]");

  if (!button) {
    return;
  }

  console.log(button.dataset.id);
});
```

事件委托利用事件冒泡，在父元素上处理多个子元素的事件，适合动态列表。

### 表单读取

```js
const form = document.querySelector("#login-form");

form.addEventListener("submit", (event) => {
  event.preventDefault();

  const formData = new FormData(form);
  const username = formData.get("username");

  console.log({ username });
});
```

## 模块

`js/math.js`：

```js
export function add(a, b) {
  return a + b;
}

export const PI = Math.PI;

export default function multiply(a, b) {
  return a * b;
}
```

`js/main.js`：

```js
import multiply, { add, PI } from "./math.js";

console.log(add(1, 2));
console.log(multiply(2, 3));
console.log(PI);
```

HTML：

```html
<script type="module" src="./js/main.js"></script>
```

- `export`：导出变量、函数或类。
- `import`：导入其他模块。
- `type="module"`：启用模块语法。
- 模块路径通常需要写扩展名，例如 `./math.js`。
- 模块默认使用严格模式，不会把顶层变量自动挂到 `window`。

## 错误处理

```js
try {
  const data = JSON.parse("{ invalid json }");
  console.log(data);
} catch (error) {
  console.error("解析失败：", error);
} finally {
  console.log("无论成功或失败都会执行");
}
```

```js
function divide(a, b) {
  if (b === 0) {
    throw new Error("除数不能为 0");
  }

  return a / b;
}
```

建议：

- 只捕获能够处理的错误。
- 不要使用空的 `catch {}` 隐藏问题。
- 错误信息应说明上下文，但不要泄露敏感数据。
- 异步代码中的错误需要使用 `try...catch` 或 `.catch()` 处理。

## 异步编程

### Promise

```js
fetch("./data.json")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return response.json();
  })
  .then((data) => {
    console.log(data);
  })
  .catch((error) => {
    console.error(error);
  });
```

### async / await

```js
async function loadData() {
  try {
    const response = await fetch("./data.json");

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("加载失败：", error);
    return null;
  }
}
```

- `async` 函数总是返回 Promise。
- `await` 等待 Promise 完成。
- `fetch()` 只在网络层面失败时拒绝，HTTP 404、500 仍然需要检查 `response.ok`。
- 多个无依赖请求可以使用 `Promise.all()` 并发执行。

```js
async function loadAll() {
  const [users, posts] = await Promise.all([
    fetch("/api/users").then((response) => response.json()),
    fetch("/api/posts").then((response) => response.json()),
  ]);

  return { users, posts };
}
```

## JSON

```js
const user = {
  name: "张三",
  skills: ["HTML", "CSS"],
};

const json = JSON.stringify(user);
const parsed = JSON.parse(json);

console.log(json, parsed);
```

- `JSON.stringify()`：对象或数组转 JSON 字符串。
- `JSON.parse()`：JSON 字符串转 JavaScript 值。
- JSON 字符串必须使用双引号。
- 解析外部输入时应使用 `try...catch`。
- JSON 不能直接表示 `undefined`、函数、`Symbol` 等值。

## 浏览器存储

```js
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme");
localStorage.removeItem("theme");
```

```js
const settings = { fontSize: 16, theme: "dark" };

localStorage.setItem("settings", JSON.stringify(settings));

const saved = JSON.parse(localStorage.getItem("settings") ?? "{}");
```

- `localStorage`：长期保存，关闭浏览器后仍存在。
- `sessionStorage`：当前标签页会话期间保存。
- 存储值都是字符串，对象需要通过 JSON 转换。
- 存储大小有限，不适合保存大文件。
- 不要把密码、访问令牌等敏感信息直接保存在 Web Storage 中。

## 常用内置对象

| 对象 | 常见方法或用途 |
| --- | --- |
| `Math` | `Math.round()`、`Math.floor()`、`Math.ceil()`、`Math.random()` |
| `Date` | 日期时间读取、设置和格式化 |
| `JSON` | `JSON.stringify()`、`JSON.parse()` |
| `Object` | `Object.keys()`、`Object.values()`、`Object.entries()` |
| `Array` | `Array.isArray()`、`Array.from()` |
| `String` | 字符串查找、截取和转换 |
| `Number` | `Number.isNaN()`、`Number.isFinite()`、`Number.parseInt()` |
| `Promise` | 异步任务协作 |
| `Map` | 键可以是任意类型的键值集合 |
| `Set` | 自动去重的值集合 |

## 常用代码片段

### 切换类名

```js
const button = document.querySelector("[data-menu-button]");
const menu = document.querySelector("[data-menu]");

button.addEventListener("click", () => {
  menu.classList.toggle("is-open");
  const isOpen = menu.classList.contains("is-open");
  button.setAttribute("aria-expanded", String(isOpen));
});
```

### 渲染列表

```js
const users = [
  { id: 1, name: "张三" },
  { id: 2, name: "李四" },
];

const list = document.querySelector("#user-list");

const fragment = document.createDocumentFragment();

users.forEach((user) => {
  const item = document.createElement("li");
  item.textContent = user.name;
  item.dataset.id = String(user.id);
  fragment.append(item);
});

list.replaceChildren(fragment);
```

优先使用 `textContent` 插入用户数据，避免 XSS。

### 读取表单

```js
const form = document.querySelector("#profile-form");

form.addEventListener("submit", (event) => {
  event.preventDefault();

  const data = Object.fromEntries(new FormData(form));
  console.log(data);
});
```

### 防抖

```js
function debounce(callback, delay = 300) {
  let timerId;

  return (...args) => {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      callback(...args);
    }, delay);
  };
}
```

### 本地主题初始化

```js
const savedTheme = localStorage.getItem("theme");

if (savedTheme === "dark") {
  document.documentElement.dataset.theme = "dark";
}
```

## 初始化检查清单

- [ ] 外部脚本是否使用了 `defer`，或放在 `body` 末尾。
- [ ] 是否能在浏览器 Console 中看到预期输出。
- [ ] 选择器查找的元素是否存在，是否需要处理 `null`。
- [ ] 是否优先使用 `const`，仅在需要重新赋值时使用 `let`。
- [ ] 用户输入是否进行了类型转换和校验。
- [ ] 表单提交是否按需要调用 `event.preventDefault()`。
- [ ] 插入用户内容时是否使用 `textContent` 而不是 `innerHTML`。
- [ ] 异步请求是否检查了 `response.ok`。
- [ ] 是否处理了加载中、成功、失败和空数据状态。
- [ ] 是否避免把密码、令牌等敏感数据写入 localStorage。
- [ ] 页面是否能通过键盘操作，并保留可见焦点。
- [ ] 是否在 Console 和 Network 面板中没有意外错误。

## 推荐学习顺序

1. [[01-JavaScript 基础与输出]]
2. [[02-变量、数据类型与类型转换]]
3. [[01-运算符、分支与循环]]
4. [[02-函数]]
5. [[01-DOM 节点与事件]]
6. [[01-数组、对象与 Date]]
7. [[01-数字时钟]]
8. [[02-复选框全选与显示隐藏]]
9. [[01-jQuery 表单验证]]

> [!tip] 学习方法
> 每个知识点都用浏览器跑一遍，先在 Console 验证语言行为，再把结果接入 DOM。遇到错误时优先看 Console 的报错行号和调用栈，不要靠猜。

## 相关笔记

- [[00-JavaScript 学习索引]]
- [[01-JavaScript 基础与输出]]
- [[02-变量、数据类型与类型转换]]
- [[01-运算符、分支与循环]]
- [[02-函数]]
- [[01-DOM 节点与事件]]
- [[01-数组、对象与 Date]]
- [[01-数字时钟]]
- [[02-复选框全选与显示隐藏]]
- [[01-jQuery 表单验证]]