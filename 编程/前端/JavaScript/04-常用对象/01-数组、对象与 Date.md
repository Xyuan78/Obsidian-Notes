---
title: 数组、对象与 Date
tags:
  - 前端
  - JavaScript
  - Date
  - 对象
---

# 数组、对象与 Date

> [!source] 课程对照
> `html.pdf` 第 36、40–44 页。

## 数组

```js
const cars = ["BMW", "Volvo", "Saab", "Ford"];

console.log(cars[0]);
cars.push("Tesla");
cars.pop();
```

- 数组索引从 `0` 开始。
- `length` 返回元素数量。
- `push()` 在末尾添加，`pop()` 删除末尾元素。
- `for...of` 适合遍历数组值。
- `map()`、`filter()`、`find()` 等现代数组方法适合处理数据。

## 对象

```js
const person = {
  fname: "Bill",
  lname: "Gates",
  age: 56,
};

console.log(person.fname);
console.log(person["age"]);
```

- 对象由键值对组成。
- 可以通过点语法或方括号读取属性。
- 方括号适合属性名包含特殊字符或从变量动态获取时使用。
- `for...in` 可以遍历对象键名。

## Date 日期时间对象

```js
const now = new Date();

console.log(now.getFullYear());
console.log(now.getMonth());
console.log(now.getDate());
console.log(now.getDay());
console.log(now.getHours());
console.log(now.getMinutes());
console.log(now.getSeconds());
console.log(now.getMilliseconds());
console.log(now.getTime());
```

### 读取方法

| 方法 | 说明 |
| --- | --- |
| `getFullYear()` | 四位年份 |
| `getMonth()` | 月份，范围 0–11 |
| `getDate()` | 一月中的第几天，范围 1–31 |
| `getDay()` | 一周中的第几天，0 表示星期天 |
| `getHours()` | 小时，范围 0–23 |
| `getMinutes()` | 分钟，范围 0–59 |
| `getSeconds()` | 秒数，范围 0–59 |
| `getMilliseconds()` | 毫秒，范围 0–999 |
| `getTime()` | 从 1970-01-01 起的毫秒时间戳 |
| `getTimezoneOffset()` | 本地时间与 UTC 相差的分钟数 |

### 设置方法

```js
const date = new Date();

date.setFullYear(2026);
date.setMonth(8); // 8 表示九月
date.setDate(14);
date.setHours(12);
date.setMinutes(30);
date.setSeconds(0);
```

### 格式化输出

```js
const now = new Date();

console.log(now.toLocaleString());
console.log(now.toLocaleDateString());
console.log(now.toLocaleTimeString());
console.log(now.toISOString());
```

- `toLocaleString()` 按本地格式输出日期和时间。
- `toISOString()` 使用 UTC 标准格式，适合数据交换。
- `getYear()`、`setYear()`、`toGMTString()` 已废弃，不要在新代码中使用。
- 月份从 `0` 开始，这是常见错误来源。