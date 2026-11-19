---
title: JavaScript 课程笔记（逐字稿重写）
tags:
  - 前端
  - JavaScript
  - 课程笔记
aliases:
  - JS 课程笔记
  - JavaScript 逐字稿重写
---

# JavaScript 课程笔记（逐字稿重写）

> [!source] 整理范围
> 本页只根据本次提供的 4 份逐字稿整理，共 17 页。原文是课堂口语记录，本页按知识点重新组织，删除重复表达和课堂闲聊，并补充必要的代码示例。
>
> 为保持笔记可运行，个别口述中不准确或不利于现代 JavaScript 的说法已在对应位置标明校正。

## 1. JavaScript 的作用

网页开发中的基础分工：

- HTML：负责页面的内容和结构。
- CSS：负责页面的样式和布局。
- JavaScript：负责页面的行为和交互。

课程中的典型例子是表格鼠标悬停变色。HTML 负责表格结构，CSS 负责颜色，JavaScript 负责监听鼠标事件并改变样式。

JavaScript 还能完成：

- 点击按钮后执行函数。
- 显示当前日期和时间。
- 读取和修改页面元素。
- 处理字符串、数组和对象。
- 校验表单。
- 在浏览器和服务器之间传输数据。

### JavaScript 与 Java

课程明确强调：JavaScript 和 Java 是两种不同的语言，它们不是包含关系，也不是同一套语法体系。

| 对比项 | JavaScript | Java |
| --- | --- | --- |
| 类型 | 动态类型、弱类型 | 静态类型、强类型 |
| 执行 | 浏览器或 JS 运行时解释执行 | 先编译，再由虚拟机执行 |
| 常见场景 | 网页行为、交互、前后端脚本 | 后端业务、Android、大型应用 |

“弱类型”是课程中的说法。更准确地说，JavaScript 是动态类型且允许隐式类型转换的语言。变量本身没有固定类型，运行时保存的值有类型。

```js
let value = "张三";
console.log(typeof value); // "string"

value = 100;
console.log(typeof value); // "number"
```

同一个变量可以先保存字符串，再保存数字，这是动态类型语言的特点。

## 2. 把 JavaScript 写进网页

### 2.1 写在 script 标签中

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>JavaScript 示例</title>
  </head>
  <body>
    <h1>JavaScript 示例</h1>

    <script>
      alert("页面已加载");
      console.log("JavaScript 正在运行");
    </script>
  </body>
</html>
```

课程说明：`<script>` 可以放在 `head`、`body` 或页面中的其他位置，但实际编写时要考虑代码什么时候使用页面元素。

### 2.2 写在外部 .js 文件中

推荐目录结构：

```text
project/
├── index.html
├── css/
│   └── style.css
└── js/
    └── js-demo.js
```

HTML 中引入：

```html
<script src="./js/js-demo.js"></script>
```

`js/js-demo.js`：

```js
alert("来自外部 JavaScript 文件");
console.log("外部 JS 文件加载成功");
```

课程建议把 JavaScript 写在独立的 `.js` 文件中，HTML 只负责结构，CSS 负责样式，JavaScript 负责行为。

### 2.3 script 放置位置

课程演示了三种情况：

1. 脚本写在 `head` 中。
2. 脚本写在页面内容之前。
3. 脚本写在 `body` 的最后。

如果脚本需要操作页面元素，而元素还没有被浏览器解析出来，脚本可能找不到元素。因此课堂中推荐：

```html
<body>
  <h1 id="title">页面标题</h1>

  <script src="./js/main.js"></script>
</body>
```

现代写法也可以使用 `defer`：

```html
<head>
  <script src="./js/main.js" defer></script>
</head>
```

`defer` 表示先继续解析 HTML，等文档解析完成后再按顺序执行脚本，适合需要操作 DOM 的普通外部脚本。

### 2.4 页面加载时立即执行与按钮触发时执行

页面加载就执行的代码：

```html
<script>
  console.log("页面加载时执行");
</script>
```

用户点击后才执行的代码：

```html
<button onclick="showMessage()">显示提示</button>

<script>
  function showMessage() {
    alert("按钮被点击");
  }
</script>
```

课堂中的原则是：

- 页面显示前就要使用的脚本，需要提前引入。
- 只有用户交互时才使用的代码，可以等需要时再执行。
- 外部脚本放在 `body` 末尾有助于先显示页面内容，再执行脚本。

## 3. 输出和调试

### 3.1 alert

```js
alert("Hello World");
```

`alert()` 会弹出提示框，并且会暂停页面继续执行，直到用户点击“确定”。

课堂用它演示了脚本执行顺序：如果 `alert()` 写在页面内容之前，后面的内容可能先被阻塞，用户确认弹框后才继续显示。

适合简单演示，不适合实际项目中的频繁提示。

### 3.2 console.log

```js
console.log("Hello World");
console.log(5 + 6);
```

浏览器中按 `F12` 打开开发者工具，切换到 **Console（控制台）** 查看结果。

课程还提到：

```js
console.log("普通信息");
console.info("提示信息");
console.debug("调试信息");
console.warn("警告信息");
console.error("错误信息");
```

控制台会使用不同样式或颜色区分消息。开发阶段应优先使用 Console，而不是频繁使用 `alert()`。

### 3.3 document.write

```js
document.write("<h1>直接写入页面</h1>");
document.write("<p>这是一段文本</p>");
```

课程演示发现：如果页面已经加载完成后再执行 `document.write()`，可能重写整个页面，把原来显示的内容覆盖掉。

因此不要把它当作现代页面修改元素的常规方式。需要修改页面时，应查找元素并更新它的内容。

## 4. 基本语法

### 4.1 字面量

字面量是直接写在代码中的固定值：

```js
100;
3.14;
"张三";
true;
false;
null;
[1, 2, 3];
{ name: "张三", age: 18 };
```

课程把数字、字符串、表达式、数组和对象都归纳到基本数据表示之中。

### 4.2 注释

单行注释：

```js
// 这是单行注释
```

多行注释：

```js
/*
  这是多行注释
  可以写多行说明
*/
```

课程建议给重要代码写清楚注释，因为过一段时间后，即使自己再看代码也可能忘记当时的思路。

### 4.3 大小写

JavaScript 区分大小写：

```js
let name = "张三";
let Name = "李四";
```

`name` 和 `Name` 是两个不同的变量。方法名也必须保持大小写一致。

### 4.4 分号

```js
let name = "张三";
console.log(name);
```

课程说明分号在部分情况下可以省略，但推荐一行只写一条语句，并保持统一风格。初学阶段显式写分号更清晰。

### 4.5 字符串换行

课程提到可以使用反斜杠连接很长的字符串：

```js
const message = "这是一段很长的文本，\
继续写在下一行";
```

这种写法只适合较长的字符串。普通代码应按可读性进行正常换行。

## 5. 变量

### 5.1 var

```js
var name = "张三";
var age = 18;
```

课程说明 `var` 是较早的 JavaScript 写法（ES5）。它可以声明变量，也可以在同一行中声明多个变量。

### 5.2 let

```js
let count = 0;
count = count + 1;
```

`let` 是 ES6 引入的变量声明方式，用于值需要改变的变量。

课程口述中出现过“JavaScript 不识别 let”的说法。这个说法不准确：现代浏览器和 Node.js 都支持 `let`、`const` 和箭头函数。课堂报错通常来自把 TypeScript 类型注解错误地写进了 JavaScript 文件，而不是来自 `let` 本身。

### 5.3 const

```js
const siteName = "前端学习笔记";
```

`const` 表示不能重新绑定的变量。

```js
const user = { name: "张三" };
user.name = "李四"; // 可以修改对象内部属性

// user = {}; // 错误：不能重新给 user 赋值
```

`const` 限制的是“变量不能重新绑定”，不是对象内部所有内容都完全不可修改。

### 5.4 声明建议

现代 JavaScript 中建议：

- 默认先使用 `const`。
- 确实需要重新赋值时使用 `let`。
- 新代码一般不再优先使用 `var`。
- 变量名要表达清楚含义，避免使用无意义的单字母名称。
- 变量名推荐使用有意义的小驼峰形式，例如 `userName`、`totalCount`。

## 6. 数据类型

课程主要讲到以下类型：

| 类型 | 示例 | `typeof` 结果 |
| --- | --- | --- |
| 数字 | `100`、`3.14` | `"number"` |
| 字符串 | `"张三"`、`'hello'` | `"string"` |
| 布尔值 | `true`、`false` | `"boolean"` |
| 未定义 | `undefined` | `"undefined"` |
| 空值 | `null` | `"object"` |
| 数组 | `[100, 120, 50]` | `"object"` |
| 对象 | `{ name: "张三" }` | `"object"` |
| 日期对象 | `new Date()` | `"object"` |
| 正则表达式 | `/abc/` | `"object"` |
| 函数 | `function () {}` | `"function"` |

课堂口述中曾说“所有内容都是 object”。这句话需要分情况理解：

- `number`、`string`、`boolean`、`undefined`、`null` 是原始类型。
- 数组、普通对象、日期对象、正则表达式是对象类型。
- 函数是可调用的对象，但 `typeof` 函数会返回 `"function"`。
- `typeof null` 返回 `"object"`，是历史遗留行为。
- 数组应该使用 `Array.isArray()` 判断。

```js
console.log(typeof 100);              // "number"
console.log(typeof "张三");            // "string"
console.log(typeof true);             // "boolean"
console.log(typeof undefined);        // "undefined"
console.log(typeof null);             // "object"
console.log(typeof []);               // "object"
console.log(Array.isArray([]));       // true
console.log(typeof function () {});   // "function"
```

### 6.1 typeof

```js
const name = "张三";
let score = 100;

console.log(typeof name);  // "string"
console.log(typeof score); // "number"
```

`typeof` 用于查看值的类型，可以写括号，也可以不写括号：

```js
typeof name;
typeof(name);
```

课程更推荐使用括号形式，阅读时更清楚。

### 6.2 undefined

```js
let value;
console.log(value); // undefined
```

常见情况：

- 变量已经声明，但没有赋值。
- 访问对象中不存在的属性。
- 函数没有返回值时，默认得到 `undefined`。

```js
const student = { name: "张三" };
console.log(student.age); // undefined
```

### 6.3 null

```js
let selectedUser = null;
```

`null` 通常表示程序员主动设置的“空值”。

课程尝试用 Java 的堆内存解释 `undefined` 和 `null`，但并不适合直接套用到 JavaScript。实际使用时可以这样记：

- `undefined`：还没有值，或者属性不存在。
- `null`：明确表示空，是开发者主动赋予的值。

```js
console.log(undefined == null);  // true
console.log(undefined === null); // false
```

### 6.4 数组也是对象

```js
const scores = [100, 120, 50];
const mixed = ["张三", 18, true, null];

console.log(scores[0]);        // 100
console.log(scores.length);    // 3
console.log(typeof scores);    // "object"
console.log(Array.isArray(scores)); // true
```

课程强调：JavaScript 数组不像 C 语言数组那样要求所有元素类型完全一致，同一个数组中可以混合字符串、数字、布尔值等。

## 7. TS 类型注解与 JavaScript 的区别

课程中演示了在变量和函数后写类型：

```ts
let age: number = 18;
const name: string = "张三";

function add(a: number, b: number): number {
  return a + b;
}
```

这些写法属于 TypeScript，不是普通 JavaScript。

```js
let age = 18;
const name = "张三";

function add(a, b) {
  return a + b;
}
```

课程使用接口描述对象结构：

```ts
interface Teacher {
  id: number;
  name: string;
  year: number;
}

const teacher1: Teacher = {
  id: 1,
  name: "张老师",
  year: 10,
};
```

课程结论：

- JavaScript 变量不需要写类型。
- TypeScript 可以为变量、函数参数、函数返回值和对象增加类型。
- TypeScript 是建立在 JavaScript 基础上的语言，需要编译或转换后运行。
- TypeScript 中数字类型使用 `number`，不是 Java 中的 `int`。
- 函数没有返回值时，TypeScript 可以把返回类型写成 `void`。
- 当前课程先学习 JavaScript，之后再把类型系统自然迁移到 TypeScript。

## 8. 运算符

### 8.1 算术运算符

```js
const a = 10;
const b = 3;

console.log(a + b); // 13
console.log(a - b); // 7
console.log(a * b); // 30
console.log(a / b); // 3.333...
console.log(a % b); // 1
```

课程还提到自增和自减：

```js
let count = 0;

count++;
count--;
```

可以使用小括号改变运算顺序：

```js
const result = (10 + 20) * 2;
```

### 8.2 赋值运算符

```js
let value = 10;

value += 5; // 15
value -= 3; // 12
value *= 2; // 24
value /= 4; // 6
```

### 8.3 字符串拼接

```js
const firstName = "张";
const lastName = "三";

console.log(firstName + lastName); // "张三"
```

“+”两侧只要有一侧是字符串，通常就会把另一侧转换成字符串后拼接：

```js
console.log(5 + 5);      // 10
console.log("5" + 5);    // "55"
console.log("5" + "5");  // "55"
```

### 8.4 比较运算符

```js
console.log(10 > 5);   // true
console.log(10 < 5);   // false
console.log(10 >= 10); // true
console.log(10 <= 5);  // false
console.log(10 == "10");  // true
console.log(10 === "10"); // false
```

课程重点强调了 `==` 和 `===`：

| 运算符 | 是否比较类型 | 建议 |
| --- | --- | --- |
| `==` | 否，会进行类型转换 | 初学代码不推荐 |
| `!=` | 否，会进行类型转换 | 初学代码不推荐 |
| `===` | 是，值和类型都要相同 | 推荐 |
| `!==` | 是，值和类型都要不同 | 推荐 |

```js
const name = "100";

console.log(name == 100);  // true，字符串被转换后比较
console.log(name === 100); // false，类型不同
```

### 8.5 逻辑运算符

```js
const isAdult = true;
const hasTicket = false;

console.log(isAdult && hasTicket); // false
console.log(isAdult || hasTicket); // true
console.log(!isAdult);             // false
```

- `&&`：与，两边都为真时结果才为真。
- `||`：或，任意一边为真时结果为真。
- `!`：非，反转布尔值。

## 9. 类型转换

### 9.1 隐式转换

```js
console.log("5" + 1); // "51"
console.log("5" - 1); // 4
console.log(1 + true); // 2
```

`+` 遇到字符串时通常会拼接，其他算术运算通常会尝试把值转换成数字。隐式转换容易造成错误，关键代码应显式转换。

### 9.2 数字转字符串

```js
const number = 100;

const text1 = String(number);
const text2 = number.toString();
const text3 = number + "";

console.log(text1, text2, text3);
```

### 9.3 字符串转数字

```js
const number1 = Number("100");       // 100
const number2 = parseInt("100");    // 100
const number3 = parseFloat("15.8"); // 15.8
```

课程重点讲解了 `parseInt()` 的进制参数：

```js
console.log(parseInt("110", 2));  // 6，二进制
console.log(parseInt("FF", 16));  // 255，十六进制
console.log(parseInt("123abc"));  // 123，遇到非法字符停止
console.log(parseInt("abc"));     // NaN
```

说明：

- 第二个参数表示按什么进制解析。
- 没有传入进制时，实际项目中建议养成显式写十进制的习惯，例如 `parseInt(value, 10)`。
- 遇到无法解析的内容时会得到 `NaN`。

### 9.4 NaN

```js
const result = Number("abc");
console.log(result);       // NaN
console.log(typeof result); // "number"
```

`NaN` 表示 **Not a Number（不是一个有效数字）**，但它的数据类型仍然是 `number`。

判断 `NaN` 使用：

```js
console.log(Number.isNaN(NaN)); // true
```

### 9.5 布尔值转换

以下值通常转换为 `false`：

```js
Boolean(false);
Boolean(0);
Boolean("");
Boolean(null);
Boolean(undefined);
Boolean(NaN);
```

其他常见的非空字符串、非零数字和对象通常转换为 `true`。

## 10. 流程控制

### 10.1 if / else

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

执行过程：

1. 判断第一个条件。
2. 条件成立就执行对应代码块。
3. 条件不成立时继续判断下一个 `else if`。
4. 所有条件都不成立时执行 `else`。

### 10.2 switch

```js
const today = new Date().getDay();

switch (today) {
  case 0:
    console.log("星期日");
    break;

  case 1:
    console.log("星期一");
    break;

  case 6:
    console.log("星期六");
    break;

  default:
    console.log("工作日");
}
```

课程强调：

- `getDay()` 返回 `0` 到 `6`。
- `0` 表示星期日。
- `switch` 中不要忘记 `break`，否则可能继续执行后续分支。
- 所有分支都不匹配时执行 `default`。
- JavaScript 的 `switch` 可以使用数字和字符串。

### 10.3 for 循环

```js
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
```

执行顺序：

1. 初始化 `let i = 1`。
2. 判断 `i <= 5`。
3. 条件为真时执行循环体。
4. 执行 `i++`。
5. 重复判断和循环。

### 10.4 for...in

```js
const student = {
  name: "张三",
  age: 18,
};

for (const key in student) {
  console.log(key, student[key]);
}
```

数组中使用 `for...in` 时，得到的是下标：

```js
const colors = ["red", "green", "blue"];

for (const index in colors) {
  console.log(index, colors[index]);
}
```

现代代码遍历数组值时，更推荐 `for...of`：

```js
for (const color of colors) {
  console.log(color);
}
```

### 10.5 while

```js
let count = 1;

while (count <= 5) {
  console.log(count);
  count++;
}
```

条件为真时执行循环体，条件为假时停止。

### 10.6 do...while

```js
let count = 1;

do {
  console.log(count);
  count++;
} while (count <= 5);
```

`do...while` 先执行一次循环体，再判断条件，所以至少执行一次。

### 10.7 break 和 continue

```js
for (let i = 1; i <= 10; i++) {
  if (i === 5) {
    break;
  }

  console.log(i);
}
```

`break` 用于跳出整个循环。

```js
for (let i = 1; i <= 5; i++) {
  if (i === 3) {
    continue;
  }

  console.log(i);
}
```

`continue` 用于跳过本轮剩余代码，进入下一轮循环。

## 11. 函数

### 11.1 函数声明

```js
function add(a, b) {
  return a + b;
}

const result = add(10, 20);
console.log(result); // 30
```

结构：

- `function`：函数关键字。
- `add`：函数名。
- `(a, b)`：参数列表。
- `{ ... }`：函数体。
- `return`：返回结果并结束函数。

JavaScript 函数不需要写返回类型。是否返回值取决于函数体中有没有 `return`。

```js
function printMessage(message) {
  console.log(message);
}

console.log(printMessage("你好")); // 先输出"你好"，再输出 undefined
```

没有显式 `return` 时，函数返回 `undefined`。

### 11.2 函数表达式

```js
const add = function (a, b) {
  return a + b;
};

console.log(add(1, 2));
```

函数可以作为值赋给变量，也可以作为参数传给其他函数。

### 11.3 箭头函数

课程把箭头函数称为 Lambda 表达式：

```js
const add = (a, b) => {
  return a + b;
};

const square = (number) => number * number;
const sayHello = () => "Hello";
```

简写规则：

- 只有一个参数时，可以省略参数括号。
- 函数体只有一条返回表达式时，可以省略 `return` 和花括号。
- 需要多行逻辑时，仍然使用完整花括号。

```js
const greet = (name) => `你好，${name}`;
console.log(greet("张三"));
```

### 11.4 参数

JavaScript 参数不需要声明类型：

```js
function createStudent(name, age) {
  return {
    name,
    age,
  };
}

createStudent("张三", 18);
```

TypeScript 中才会写参数类型：

```ts
function createStudent(name: string, age: number) {
  return { name, age };
}
```

### 11.5 函数也是一种对象

课程强调函数类型也是一种对象类型。函数可以被赋值、传递和调用：

```js
const runTask = function () {
  console.log("执行任务");
};

const anotherTask = runTask;

anotherTask();
```

### 11.6 课程中的函数示例

课程通过按钮点击函数演示：

```html
<button onclick="fun01()">执行函数</button>

<script>
  function fun01() {
    alert("函数执行成功");
  }
</script>
```

后续把函数拆成多个独立函数，每个函数负责一个明确任务。

## 12. 数组

### 12.1 创建和访问

```js
const scores = [100, 120, 50];

console.log(scores[0]); // 100
console.log(scores[1]); // 120
console.log(scores.length); // 3
```

数组下标从 `0` 开始。

### 12.2 类型可以混合

```js
const studentInfo = ["张三", 18, true, null];
```

课程把这种写法看作 JavaScript 的动态类型特点。实际项目为了可维护性，通常仍然让同一个数组尽量保存相似类型的数据。

### 12.3 数组的类型

```js
const scores = [100, 120, 50];

console.log(typeof scores);          // "object"
console.log(Array.isArray(scores)); // true
```

数组是对象类型，不能只通过 `typeof` 判断是否为数组。

### 12.4 遍历

```js
const scores = [100, 120, 50];

for (let i = 0; i < scores.length; i++) {
  console.log(scores[i]);
}
```

```js
for (const score of scores) {
  console.log(score);
}
```

## 13. 对象

### 13.1 键值对

```js
const student = {
  id: "001",
  name: "张三",
  age: 20,
  isMale: false,
  scores: [100, 120, 50],
};
```

对象由键值对组成，通常称为属性和属性值。

### 13.2 点语法

```js
console.log(student.name);
console.log(student.scores[0]);
```

### 13.3 方括号语法

```js
console.log(student["name"]);
```

当属性名放在字符串中，或属性名需要通过变量动态确定时，方括号语法更合适：

```js
const key = "age";
console.log(student[key]);
```

### 13.4 嵌套对象

课程中创建了地址对象，再把它放入学生对象：

```js
const address = {
  province: "河南",
  city: "新乡",
  district: "红旗区",
  detail: "某某路 001 号",
};

const student = {
  id: "001",
  name: "张三",
  address: address,
};
```

访问嵌套属性：

```js
console.log(student.address.city);
```

### 13.5 对象与 JSON

```js
const student = {
  name: "张三",
  age: 18,
};
```

对象的字面量写法与 JSON 很相似。JSON 常用于前后端数据交换，但 JSON 的要求更严格：

- JSON 的属性名必须使用双引号。
- JSON 不能直接写函数。
- JSON 不能保存 `undefined`。
- JavaScript 对象可以包含更丰富的值。

```js
const jsonText = JSON.stringify(student);
const parsedStudent = JSON.parse(jsonText);
```

### 13.6 对象中不存在的属性

```js
const student = {
  name: "张三",
};

console.log(student.age); // undefined
```

访问不存在的属性通常不会直接报错，而是返回 `undefined`。这也是课程中函数传参写错属性名后得到 `undefined` 的原因。

## 14. 字符串

### 14.1 创建字符串

```js
const text1 = "双引号字符串";
const text2 = '单引号字符串';
const text3 = `模板字符串`;
```

课程强调字符串可以用单引号或双引号包起来，两者都可以。

### 14.2 下标访问

```js
const text = "JavaScript";

console.log(text[0]); // "J"
console.log(text[4]); // "S"
console.log(text.length); // 10
```

字符串可以按下标读取单个字符，但它不是可以随意修改的数组。

### 14.3 转义字符

```js
const message = "第一行\n第二行";
const tab = "姓名\t张三";
const quote = "他说：\"你好\"";
```

常见转义：

| 转义符 | 含义 |
| --- | --- |
| `\n` | 换行 |
| `\r` | 回车 |
| `\t` | 制表符 |
| `\b` | 退格 |
| `\\` | 反斜杠 |
| `\"` | 双引号 |
| `\'` | 单引号 |

### 14.4 常用字符串属性与方法

```js
const text = "  HTML,CSS,JavaScript  ";

console.log(text.length);
console.log(text.indexOf("CSS"));
console.log(text.lastIndexOf("JavaScript"));
console.log(text.toUpperCase());
console.log(text.trim());
console.log(text.replace("HTML", "TypeScript"));
console.log(text.split(","));
```

课程重点演示了 `split()`：

```js
const input = "张三,李四,王五,洗衣服";
const names = input.split(",");

console.log(names);
```

结果是把一个字符串按逗号拆成数组：

```text
["张三", "李四", "王五", "洗衣服"]
```

## 15. Date 日期对象

### 15.1 创建当前日期

```js
const now = new Date();
console.log(now);
```

### 15.2 常用读取方法

```js
const now = new Date();

const year = now.getFullYear();
const month = now.getMonth() + 1;
const day = now.getDate();
const week = now.getDay();
const hours = now.getHours();
const minutes = now.getMinutes();
const seconds = now.getSeconds();
const milliseconds = now.getMilliseconds();
```

| 方法 | 返回值 |
| --- | --- |
| `getFullYear()` | 四位年份 |
| `getMonth()` | 月份，范围 `0`–`11` |
| `getDate()` | 一月中的第几天，范围 `1`–`31` |
| `getDay()` | 星期，范围 `0`–`6` |
| `getHours()` | 小时，范围 `0`–`23` |
| `getMinutes()` | 分钟，范围 `0`–`59` |
| `getSeconds()` | 秒，范围 `0`–`59` |
| `getMilliseconds()` | 毫秒 |
| `getTime()` | 从 1970 年 1 月 1 日开始的毫秒数 |

月份和星期是常见错误来源：

```js
const now = new Date();

console.log(now.getMonth()); // 例如九月得到 8
console.log(now.getMonth() + 1); // 九月得到 9

console.log(now.getDay()); // 0 表示星期日
```

### 15.3 日期补零

课程的数字时钟通过判断数字是否小于 `10` 来补零：

```js
function pad(number) {
  if (number < 10) {
    return "0" + number;
  }

  return String(number);
}
```

使用：

```js
const now = new Date();

const month = pad(now.getMonth() + 1);
const day = pad(now.getDate());
const hours = pad(now.getHours());
const minutes = pad(now.getMinutes());
const seconds = pad(now.getSeconds());
```

### 15.4 拼出日期时间

```js
function getDateTimeText() {
  const now = new Date();

  const year = now.getFullYear();
  const month = pad(now.getMonth() + 1);
  const day = pad(now.getDate());
  const hours = pad(now.getHours());
  const minutes = pad(now.getMinutes());
  const seconds = pad(now.getSeconds());

  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
}
```

### 15.5 数字时钟页面

HTML：

```html
<h1>数字时钟</h1>
<div id="clock">等待显示时间</div>
<button onclick="showDate()">显示当前时间</button>
```

JavaScript：

```js
function pad(number) {
  return number < 10 ? "0" + number : String(number);
}

function showDate() {
  const now = new Date();

  const year = now.getFullYear();
  const month = pad(now.getMonth() + 1);
  const day = pad(now.getDate());
  const hours = pad(now.getHours());
  const minutes = pad(now.getMinutes());
  const seconds = pad(now.getSeconds());

  const text = `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
  document.getElementById("clock").textContent = text;
}
```

课堂版本使用 `innerHTML` 把时间写入页面。如果只是写入普通文本，现代写法更推荐 `textContent`，它可以避免把字符串误当成 HTML。

### 15.6 让时钟自动运行

课程最后说明：当前版本只有点击按钮才会刷新，后续学习定时器和延时器之后，可以让时钟自动变化。

```js
setInterval(showDate, 1000);
```

这句代码表示每秒执行一次 `showDate()`。是否启用要根据页面需求决定。

## 16. DOM 基础

DOM（Document Object Model，文档对象模型）把 HTML 解析成可以操作的节点树。

### 16.1 通过 id 获取元素

```html
<div id="clock">等待显示时间</div>
```

```js
const clock = document.getElementById("clock");
```

课程中的 `getElementById()` 名称必须完全一致，大小写不能写错。

### 16.2 修改元素内容

```js
const box = document.getElementById("clock");

box.innerHTML = "<strong>新的内容</strong>";
box.textContent = "纯文本内容";
```

区别：

- `innerHTML`：把字符串当作 HTML 片段解析。
- `textContent`：只写入纯文本。
- 如果内容来自用户输入，优先使用 `textContent`，避免 XSS 风险。

### 16.3 document.write 的问题

课程演示了：

```js
document.write("<h1>JavaScript</h1>");
```

在页面已经加载完成后再执行，可能覆盖原有页面内容。因此实际修改页面时，优先获取元素并更新其内容。

## 17. 事件

### 17.1 单击事件

```html
<button onclick="showDate()">显示时间</button>
```

```js
function showDate() {
  console.log("按钮被点击");
}
```

`onclick` 是单击事件。点击按钮时，浏览器会调用指定的函数。

### 17.2 直接写一行事件代码

```html
<button onclick="alert('Hello World')">提示</button>
```

课程说明，只有非常简单的代码才适合直接写在 HTML 属性中；复杂逻辑应放到函数中。

### 17.3 使用 this

```html
<button onclick="this.innerHTML = '已点击'">点击我</button>
```

`this` 在这里表示当前触发事件的按钮元素。课堂中还提到可以用 `this.innerHTML` 修改当前元素内容。

### 17.4 返回上一页

一行事件代码也可以处理简单导航：

```html
<button onclick="window.history.back()">返回</button>
```

课堂口述中的目标是返回上一页，但正确的浏览器历史 API 是 `window.history.back()` 或 `window.history.go(-1)`。

### 17.5 常见事件

课程提到的常见事件包括：

| 事件 | 触发时机 |
| --- | --- |
| `onclick` | 鼠标单击 |
| `onchange` | 值改变并确认后 |
| `onmouseover` | 鼠标移入 |
| `onmouseout` | 鼠标移出 |
| `onblur` | 元素失去焦点 |
| `onkeydown` / `onkeyup` | 键盘按下 / 松开 |

事件通常分为：

- 鼠标事件。
- 键盘事件。
- 表单事件。
- 窗口事件。

### 17.6 onblur 表单校验

课程以用户名唯一性为例：

```html
<label for="username">用户名</label>
<input id="username" name="username" onblur="checkUsername()" />

<p id="message"></p>
```

```js
function checkUsername() {
  const input = document.getElementById("username");
  const message = document.getElementById("message");

  if (input.value.trim() === "") {
    message.textContent = "用户名不能为空";
    return;
  }

  message.textContent = "正在检查用户名...";
}
```

`onblur` 适合在用户离开输入框后触发表单校验。真实项目中还需要由服务器检查用户名是否已经存在。

### 17.7 现代事件绑定

课程演示的是 HTML 事件属性，例如 `onclick` 和 `onblur`。现代项目通常更推荐：

```js
const button = document.querySelector("button");

button.addEventListener("click", () => {
  console.log("按钮被点击");
});
```

这样可以避免把 JavaScript 逻辑大量写进 HTML，还能给同一个事件添加多个处理函数。

## 18. 课程案例：显示日期与数字时钟

课堂案例把以下知识点串在一起：

1. 使用 `new Date()` 获取当前时间。
2. 使用 `getFullYear()`、`getMonth()` 等方法读取年月日时分秒。
3. 使用 `if` 判断是否需要补零。
4. 使用字符串拼接组织显示格式。
5. 通过 `document.getElementById()` 获取页面元素。
6. 使用 `innerHTML` 或 `textContent` 把结果写入页面。
7. 通过 `onclick` 按钮事件触发函数。
8. 后续使用 `setInterval()` 让时钟每秒更新。

## 19. 课程重点回顾

| 主题 | 必须掌握 |
| --- | --- |
| JavaScript 作用 | 控制网页行为、响应用户操作、修改页面 |
| 引入方式 | 内部 `<script>`、外部 `.js` 文件 |
| 放置位置 | 保证脚本使用 DOM 时，目标元素已经存在 |
| 输出 | `alert()`、`console.log()`、`document.write()` 的区别 |
| 变量 | `var`、`let`、`const` 的基本区别 |
| 数据类型 | 数字、字符串、布尔、`undefined`、`null`、数组、对象、日期 |
| 类型判断 | `typeof`、`Array.isArray()` |
| 运算符 | 算术、赋值、比较、逻辑 |
| 相等判断 | 优先使用 `===` 和 `!==` |
| 类型转换 | 数字与字符串转换、`parseInt()`、`NaN` |
| 流程控制 | `if`、`switch`、`for`、`for...in`、`while`、`do...while` |
| 函数 | 函数声明、函数表达式、箭头函数、参数、`return` |
| 对象 | 键值对、点语法、方括号语法、嵌套对象、JSON 形态 |
| 字符串 | 长度、下标、转义、查找、替换、拆分 |
| 日期 | `Date` 获取年月日时分秒、月份和星期索引 |
| DOM | 获取元素、修改内容 |
| 事件 | `onclick`、`onchange`、`onmouseover`、`onmouseout`、`onblur` |
| 项目应用 | 数字时钟、表格交互、用户名失焦校验 |

## 20. 课堂错误与校正

### 20.1 `let` 不是 JavaScript 不支持

现代 JavaScript 支持 `let`、`const` 和箭头函数。课堂上出现报错，常见原因是：

- 把 TypeScript 类型注解写进了 JavaScript 文件。
- 变量名或属性名拼错。
- 函数名或括号不匹配。
- 在元素尚未创建时就获取元素。

### 20.2 不要把所有类型都说成对象

原始类型和对象类型需要区分：

```js
typeof 100;    // "number"
typeof "abc";  // "string"
typeof [];     // "object"
typeof {};     // "object"
typeof null;   // "object"
```

### 20.3 不要用 `==` 代替 `===`

```js
"100" == 100;  // true，容易隐藏类型问题
"100" === 100; // false，推荐这种判断方式
```

### 20.4 `undefined` 和 `null`

- `undefined` 常表示未赋值、属性不存在或函数没有返回值。
- `null` 常表示开发者主动设置的空值。

### 20.5 字符串和数字相加

```js
5 + 5;      // 10
"5" + 5;    // "55"
```

计算前应确认输入已经转换成数字：

```js
const value = Number(input.value);

if (Number.isNaN(value)) {
  console.log("请输入有效数字");
}
```

### 20.6 先确认元素存在，再操作元素

```js
const button = document.getElementById("submit-button");

if (button) {
  button.addEventListener("click", () => {
    console.log("点击");
  });
}
```

如果脚本放在页面元素之前，或者 `id` 写错，查找结果可能是 `null`。

## 21. 最小练习

### 练习一：输出与变量

创建一个外部 JS 文件，输出：

```js
const name = "张三";
let age = 18;

console.log(name, age);
console.log(typeof name);
console.log(typeof age);
```

### 练习二：类型比较

```js
const value = "100";

console.log(value == 100);
console.log(value === 100);
```

说明两次结果为什么不同。

### 练习三：函数与按钮

创建一个按钮，点击后调用函数，并在页面中显示一句话。

### 练习四：字符串拆分

```js
const input = "张三,李四,王五,洗衣服";
const result = input.split(",");

console.log(result);
```

### 练习五：日期显示

创建一个按钮：

```html
<button onclick="showDateTime()">显示日期时间</button>
<div id="result"></div>
```

函数中读取当前日期，并按 `YYYY-MM-DD HH:mm:ss` 的格式显示。

### 练习六：数字时钟

把日期显示函数改造成：

1. 月份小于 `10` 时补 `0`。
2. 日期小于 `10` 时补 `0`。
3. 小时、分钟、秒小于 `10` 时补 `0`。
4. 每秒自动刷新一次。

## 22. 代码可读性建议

- 变量名使用有意义的小驼峰形式。
- 函数名使用动词或动词短语，例如 `showDate`、`checkUsername`。
- 一个函数只处理一个主要任务。
- 重复逻辑提取为函数，例如日期补零。
- 优先使用 `const`，需要重新赋值时使用 `let`。
- 比较时使用 `===` 和 `!==`。
- 操作 DOM 前确认目标元素存在。
- 用户输入先转换、再计算、再校验。
- 修改页面纯文本时优先使用 `textContent`。
- 开发时使用 Console 排查错误，不要只依赖弹框。

## 23. 相关入口

- [[00-JavaScript 学习索引]]
