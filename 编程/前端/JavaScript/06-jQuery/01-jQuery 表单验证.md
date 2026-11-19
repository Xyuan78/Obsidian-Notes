---
title: jQuery 表单验证
tags:
  - 前端
  - JavaScript
  - jQuery
  - 表单验证
---

# jQuery 表单验证

> [!source] 课程对照
> `html.pdf` 第 47–53 页。

## jQuery 是什么

jQuery 是早期非常流行的 JavaScript 工具库，封装了 DOM 选择、事件绑定、样式操作和 Ajax 等常用能力。

现代项目可以优先使用原生 DOM API，但阅读旧项目和课程代码时仍需要理解 jQuery。

## 引入 jQuery

课程使用 `jquery-1.8.3.js`：

```html
<script src="./js/jquery-1.8.3.js"></script>
<script src="./js/register.js"></script>
```

旧版本存在兼容性和安全问题。新项目如需使用 jQuery，应选择最新的安全版本，并通过可靠 CDN 或依赖管理器引入。

## jQuery 基础操作

```js
$(function () {
  const value = $("#userName").val();
  const id = $("#userName").attr("id");

  $("#userNameId")
    .removeClass()
    .addClass("error_prompt")
    .html("用户名不能为空");

  $("#userName").focus(function () {
    $("#userNameId").addClass("import_prompt");
  });

  $("#userName").blur(function () {
    // 校验用户名
  });
});
```

- `$()`：选择元素或包装 DOM。
- `.val()`：读取或设置表单值。
- `.attr()`：读取或设置属性。
- `.addClass()` / `.removeClass()`：增删类名。
- `.html()`：读取或设置 HTML 内容。
- `.focus()` / `.blur()`：绑定焦点和失焦事件。
- `.submit()`：绑定表单提交事件。
- `.find().each()`：在范围内查找并遍历元素。
- `.bind()`：绑定一个或多个事件，现代 jQuery 也支持 `.on()`。

## 课程验证项目

课程注册页验证了：

- 用户名：字母、数字、下划线、点和减号组成，长度 4–18，首尾必须为字母或数字。
- 密码：不能为空，长度 6–16。
- 重复密码：不能为空，必须和密码一致。
- 昵称：汉字、字母、数字、下划线和部分特殊字符组成，长度为 4–20 个字符，一个汉字按两个字符计算。
- 关联手机号：课程正则只允许 `13`、`15`、`18` 开头。
- 邮箱：使用简化正则以检查基本格式。

## 正则表达式

```js
const usernameReg = /^[0-9a-zA-Z][0-9a-zA-Z_.-]{2,16}[0-9a-zA-Z]$/;
const passwordReg = /^.{6,16}$/;
const nicknameReg = /^([\u4e00-\u9fa5]|\w|[@!#$%&*])+$/;
const mobileReg = /^(13|15|18)\d{9}$/;
const emailReg = /^\w+@\w+(\.[a-zA-Z]{2,3}){1,2}$/;
```

常见方法：

- `.test(value)`：测试字符串是否符合正则，返回布尔值。
- `^`：开头。
- `$`：结尾。
- `\d`：数字。
- `\w`：字母、数字或下划线。
- `{n,m}`：重复 `n` 到 `m` 次。
- `[\u4e00-\u9fa5]`：常见中文字符范围。

## 课程正则的局限

- 手机号只匹配 `13`、`15`、`18` 开头，已不符合现在更广泛的号段，应使用更完整的规则。
- 邮箱正则非常简化，真实邮箱格式更复杂。简单校验可以，但要允许合理格式。
- 正则验证不能代替服务器端验证。
- 不应把复杂正则写得无法维护，应写清测试案例。

## 表单提交处理

课程写法：

```js
$("#registerForm").submit(function () {
  let isValid = true;

  $(this).find("input[id]").each(function () {
    if (!validate($(this))) {
      isValid = false;
    }
  });

  return false;
});
```

`return false` 会阻止默认提交。现代原生写法优先使用：

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();

  // 执行校验
});
```

## 可维护的验证思路

1. 每个字段只负责返回自己的验证结果。
2. 错误提示放在对应字段附近。
3. `focus` 时显示填写要求，`blur` 时执行格式验证。
4. 提交时统一验证全部字段。
5. 输入值变化后及时清除旧错误。
6. 浏览器原生校验、JavaScript 校验和服务器端校验结合使用。