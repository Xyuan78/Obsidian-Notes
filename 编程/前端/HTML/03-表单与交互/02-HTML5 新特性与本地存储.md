---
title: HTML5 新特性与本地存储
tags:
  - 前端
  - HTML
  - HTML5
  - localStorage
---

# HTML5 新特性与本地存储

> [!source] 课程对照
> `html.pdf` 第 27–28 页。

## HTML5 的主要方向

课程中列出的 HTML5 / CSS3 相关内容：

- 新的语义元素和页面结构标签。
- 新的表单控件和表单属性。
- 音频与视频。
- 二维和三维图形能力。
- 客户端本地存储。
- 更完整的 Web 应用能力。

其中 CSS 动画、2D/3D 绘图、Web 应用体验等还需要结合 CSS 和 JavaScript 使用。

## 新增表单能力

```html
<label for="birth">出生年月</label>
<input id="birth" type="date" name="birth" />

<label for="email">邮箱</label>
<input id="email" type="email" name="email" />

<label for="avatar">头像</label>
<input id="avatar" type="file" name="avatar" accept="image/*" />

<label for="level">熟练程度</label>
<input id="level" type="range" name="level" min="0" max="10" />
```

常见 HTML5 控件和属性：

- 控件类型：`date`、`email`、`url`、`number`、`range`、`color`、`search` 等。
- 校验属性：`required`、`min`、`max`、`step`、`pattern`、`minlength`、`maxlength`。
- 输入辅助：`placeholder`、`autocomplete`、`datalist`。
- 文件上传：`type="file"`，必要时使用 `accept` 限制文件类型。

HTML5 原生校验只是第一层校验，服务器端必须再次验证。

## audio 与 video

```html
<video controls width="640">
  <source src="./media/demo.mp4" type="video/mp4" />
  您的浏览器不支持 video 元素。
</video>

<audio controls>
  <source src="./media/demo.mp3" type="audio/mpeg" />
  您的浏览器不支持 audio 元素。
</audio>
```

- `controls`：显示播放控件。
- `source`：提供媒体源和类型。
- 标签内部文字是浏览器不支持时的后备内容。
- 媒体文件受版权、格式、体积和网络加载影响。

## 客户端存储

浏览器提供 `localStorage` 和 `sessionStorage` 两个常用存储对象。

### localStorage

- 用于长期保存同一个网站的数据。
- 数据没有默认过期时间。
- 除非用户或程序主动删除，否则通常在关闭浏览器后仍保留。

### sessionStorage

- 用于临时保存同一个窗口或标签页的数据。
- 关闭窗口或标签页后，数据会清除。

两者 API 基本相同：

```js
// 保存数据；值通常转换为字符串
localStorage.setItem("theme", "dark");

// 读取数据
const theme = localStorage.getItem("theme");

// 删除单个数据
localStorage.removeItem("theme");

// 清空当前来源的全部数据
localStorage.clear();

// 获取指定索引处的键名
const key = localStorage.key(0);
```

## 存储对象的注意事项

- Web Storage 存储的是字符串，对象需要 `JSON.stringify()` 后再保存，读取时用 `JSON.parse()` 还原。
- `localStorage` 不应保存密码、令牌等敏感信息。
- 不要在循环中无节制地写入大量数据。
- 存储失败可能发生在隐私模式、空间不足或安全限制下，应处理异常。
- 同一来源的页面共享 `localStorage`，不同网站之间不能直接访问彼此的数据。

```js
const settings = { theme: "dark", fontSize: 16 };

localStorage.setItem("settings", JSON.stringify(settings));

const saved = JSON.parse(localStorage.getItem("settings") ?? "{}");
console.log(saved.theme);
```

## 课程提到的其他方向

- 2D / 3D 图形：Canvas、SVG、WebGL 等。
- 本地数据库：IndexedDB 等现代浏览器数据库；旧的 Web SQL 并不是通用标准。
- Web 应用：离线能力、后台任务、推送、文件系统等，需要通过现代 Web API 组合实现。