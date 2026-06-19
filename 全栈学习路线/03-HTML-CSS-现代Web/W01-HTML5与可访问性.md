---
chapter: 03
week: 01
duration: 7天
tags: [HTML5, 语义化, 表单, 可访问性, SEO]
---

# W01 · HTML5 与可访问性

> 学习目标：能从零写出**结构清晰、对搜索引擎和盲人用户都友好**的网页。

---

## 一、HTML 是什么？

### 1.1 一个比喻

打开一份 Word 简历，里面有：标题、姓名、工作经历、联系方式。这些"内容块"如果脱掉样式（字体、颜色），剩下的就是"结构"。

**HTML 就是网页的结构层 / 骨架 / 简历的 Word 模板**。

```
┌─────────────────────────┐
│  [标题] 张三的简历         │  <- h1
│  [段落] 我是后端工程师...   │  <- p
│  [列表] · Python          │  <- ul > li
│         · JavaScript      │
│  [按钮] 联系我             │  <- button
└─────────────────────────┘
       结构（HTML）
+ 化妆（CSS）+ 行为（JS）= 网页
```

| 层 | 角色 | 类比 |
|----|------|------|
| HTML | 结构 / 内容 | 简历的文字内容 |
| CSS | 样式 / 外观 | 字体颜色排版 |
| JS | 行为 / 交互 | 点击按钮发邮件 |

### 1.2 第一个 HTML 文件

打开 VS Code，新建 `hello.html`，输入下面内容（敲 `!` 然后按 Tab，VS Code 会自动生成骨架）：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>我的第一个网页</title>
</head>
<body>
  <h1>你好，世界！</h1>
  <p>这是我用 HTML 写的第一行字。</p>
</body>
</html>
```

保存后，**双击文件**或在文件管理器中"用浏览器打开"。你应该看到一个标题加一段文字。

> 💡 提示：`.html` 文件本质就是文本文件，浏览器会按 HTML 规则把它"解码"为可视化页面。

---

## 二、文档结构详解

### 2.1 五大必备元素

```html
<!DOCTYPE html>           <- 告诉浏览器：我是 HTML5
<html lang="zh-CN">       <- 根标签，lang 用于翻译/朗读
  <head>...</head>        <- 头部：元信息（不显示）
  <body>...</body>        <- 身体：可见内容
</html>
```

### 2.2 head 里必装的"五件套"

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>张三的个人主页 | 全栈工程师</title>
  <meta name="description" content="张三的简历与作品集，专注 Python/Web 开发。">
  <meta property="og:image" content="https://example.com/cover.png">
</head>
```

| 标签 | 作用 | 不写的后果 |
|------|------|-----------|
| `charset=UTF-8` | 中文编码 | 中文乱码🀄→???? |
| `viewport` | 移动端缩放 | 手机上字超小 |
| `title` | 浏览器标签文字 | 显示文件名 |
| `description` | 搜索引擎摘要 | 谷歌随机截一段 |
| `og:image` | 微信/Twitter 分享卡片 | 分享只有链接没图 |

---

## 三、语义化标签全集

### 3.1 什么叫"语义化"？

非语义化（旧写法 ❌）：

```html
<div class="header">...</div>
<div class="nav">...</div>
<div class="content">...</div>
```

语义化（HTML5 ✅）：

```html
<header>...</header>
<nav>...</nav>
<main>...</main>
```

**好处**：搜索引擎能识别区块、屏幕阅读器能正确朗读、代码自己说话。

### 3.2 页面骨架图

```
┌──────────────────────────────────┐
│  <header>  网站标题 / Logo        │
├──────┬───────────────────────────┤
│ <nav>│  <main>                   │
│      │   <article>               │
│ 侧栏  │     <section>子区块       │
│      │     <section>子区块       │
│      │   </article>              │
│      │   <aside>相关链接</aside>   │
├──────┴───────────────────────────┤
│  <footer>  版权 / 联系方式        │
└──────────────────────────────────┘
```

### 3.3 完整对照表

| 标签 | 用途 | 例子 |
|------|------|------|
| `<header>` | 页眉/标题区 | 网站 logo + 导航 |
| `<nav>` | 导航条 | 主菜单 |
| `<main>` | 主内容（每页只能 1 个） | 正文区 |
| `<article>` | 独立成篇的内容 | 一篇博客 |
| `<section>` | 主题分组 | "关于我"小节 |
| `<aside>` | 边栏/相关 | 推荐文章 |
| `<footer>` | 页脚 | 版权信息 |
| `<figure>` | 图+说明 | 配图 |
| `<figcaption>` | 图片说明文字 | "图1: 原理图" |
| `<time>` | 时间 | `<time datetime="2026-06-19">今天</time>` |
| `<mark>` | 高亮 | 像荧光笔涂的黄色 |

### 3.4 通用标签速查

```html
<h1>一级标题</h1> ... <h6>六级</h6>   <!-- 一页只用 1 个 h1 -->
<p>段落文字。</p>
<a href="https://google.com">链接</a>
<a href="mailto:me@x.com">发邮件</a>
<img src="cat.jpg" alt="一只橘猫" />
<br>  <!-- 换行 -->
<hr>  <!-- 水平分割线 -->

<ul>无序列表
  <li>项 1</li>
  <li>项 2</li>
</ul>

<ol>有序列表
  <li>第 1 步</li>
  <li>第 2 步</li>
</ol>

<div>块级容器（无语义）</div>
<span>行内容器（无语义）</span>
```

> 🎯 **div vs span**：`div` 独占一行（像 `<p>`），`span` 跟文字混排。

---

## 四、表单 form

表单是"用户输入" → "数据提交到后端"的关键。

### 4.1 基础结构

```html
<form action="/submit" method="POST">
  <label for="name">姓名：</label>
  <input id="name" name="name" type="text" required>

  <label for="email">邮箱：</label>
  <input id="email" name="email" type="email" required>

  <button type="submit">提交</button>
</form>
```

> `label` 的 `for` 必须等于 `input` 的 `id`，点击文字时聚焦输入框（**对盲人极其重要**）。

### 4.2 input 的 10 种 type

| type | 用途 | 移动端键盘 |
|------|------|------------|
| `text` | 普通文本 | 字母键盘 |
| `password` | 密码（屏蔽显示） | 字母键盘 |
| `email` | 邮箱（自动校验 @） | 带 @ 键 |
| `tel` | 电话 | 数字键盘 |
| `url` | 网址 | 带 .com 键 |
| `number` | 数字（带 min/max） | 数字键盘 |
| `date` | 日期选择器 | 日历 |
| `color` | 颜色选择器 | 调色板 |
| `file` | 文件上传 | 系统选择器 |
| `range` | 滑块 | 拖动条 |

### 4.3 校验属性

```html
<input type="text" required>                      <!-- 必填 -->
<input type="number" min="0" max="100">           <!-- 数字范围 -->
<input type="text" pattern="[0-9]{6}">            <!-- 6 位数字 -->
<input type="email">                              <!-- 自动校验邮箱格式 -->
<textarea minlength="10" maxlength="500"></textarea>
```

校验失败时浏览器会自动弹出红框提示，**0 行 JS**。

### 4.4 select / textarea / fieldset

```html
<fieldset>
  <legend>个人信息</legend>

  <label>城市：
    <select name="city">
      <option value="bj">北京</option>
      <option value="sh">上海</option>
    </select>
  </label>

  <label>留言：
    <textarea name="msg" rows="4"></textarea>
  </label>
</fieldset>
```

`fieldset + legend` 把相关字段视觉上"圈一起"，对屏幕阅读器极友好。

### 4.5 动手任务 1

写一个登录表单：邮箱 + 密码 + "记住我" 勾选 + 登录按钮。要求邮箱必填且必须是邮箱格式。

<details>
<summary>点击查看答案</summary>

```html
<form>
  <label>邮箱：<input type="email" name="email" required></label><br>
  <label>密码：<input type="password" name="pwd" required minlength="6"></label><br>
  <label><input type="checkbox" name="remember"> 记住我</label><br>
  <button type="submit">登录</button>
</form>
```
</details>

---

## 五、表格

```html
<table>
  <thead>
    <tr><th>姓名</th><th>年龄</th></tr>
  </thead>
  <tbody>
    <tr><td>张三</td><td>25</td></tr>
    <tr><td>李四</td><td>30</td></tr>
  </tbody>
</table>
```

```
┌──────┬──────┐
│ 姓名 │ 年龄 │   <- thead
├──────┼──────┤
│ 张三 │ 25   │   <- tbody > tr
│ 李四 │ 30   │
└──────┴──────┘
```

**注意**：表格用于**展示数据**，不要用于排版（那是 1999 年的事）。

---

## 六、多媒体

### 6.1 图片优化

```html
<!-- 基础 -->
<img src="cat.jpg" alt="一只橘猫" width="400" height="300" loading="lazy">

<!-- 响应式（不同屏幕加载不同尺寸） -->
<img src="cat-800.jpg"
     srcset="cat-400.jpg 400w, cat-800.jpg 800w, cat-1600.jpg 1600w"
     sizes="(max-width: 600px) 100vw, 50vw"
     alt="橘猫">

<!-- 不同格式（浏览器选最优） -->
<picture>
  <source srcset="cat.avif" type="image/avif">
  <source srcset="cat.webp" type="image/webp">
  <img src="cat.jpg" alt="橘猫">
</picture>
```

| 属性 | 作用 |
|------|------|
| `alt` | 图加载失败时显示的文字 + 屏幕阅读器朗读 |
| `loading="lazy"` | 滚到附近才加载，**首屏快 50%** |
| `srcset` | 自动选合适尺寸 |
| `width/height` | 防止"加载时跳动"（CLS） |

### 6.2 视频和音频

```html
<video src="movie.mp4" controls width="600" poster="cover.jpg"></video>
<audio src="music.mp3" controls></audio>
```

`controls` 显示播放器、`poster` 是封面图。

---

## 七、可访问性 a11y

> 全球约 15% 人口（约 12 亿）有某种残障。a11y = accessibility（11 个字母在中间）。

### 7.1 五条铁律

1. **每张 `<img>` 都要有 `alt`**（装饰图填空字符串 `alt=""`）
2. **`<label>` 必须配 `<input>`**
3. **按钮就用 `<button>`，链接就用 `<a>`**（不要用 `<div onclick>`）
4. **键盘必须能用**（按 Tab 应能逐个聚焦）
5. **颜色不是唯一信息源**（红色错误提示也要带 ✗ 图标）

### 7.2 ARIA 属性

```html
<!-- 没有可见文字的图标按钮 -->
<button aria-label="关闭对话框">×</button>

<!-- 用其他元素标注 -->
<input id="search" aria-labelledby="search-label">
<span id="search-label">搜索商品</span>

<!-- 自定义 tab -->
<div role="tablist">
  <div role="tab" tabindex="0">概览</div>
  <div role="tab" tabindex="-1">详情</div>
</div>
```

| 属性 | 用途 |
|------|------|
| `aria-label` | 给元素一个朗读用的名字 |
| `aria-labelledby` | 用别的元素当名字 |
| `role` | 告诉屏幕阅读器"我是什么" |
| `tabindex="0"` | 加入 Tab 顺序 |
| `tabindex="-1"` | 不被 Tab 聚焦但可被代码聚焦 |

### 7.3 自我测试

- 拔掉鼠标，只用 **Tab / Shift+Tab / Enter** 操作整个页面
- macOS：`Cmd + F5` 开 VoiceOver
- Windows：装 NVDA（免费）

---

## 八、SEO 基础

### 8.1 robots.txt

放在网站根目录 `/robots.txt`：

```
User-agent: *
Allow: /
Disallow: /admin/
Sitemap: https://example.com/sitemap.xml
```

### 8.2 sitemap.xml

告诉搜索引擎你有哪些页面：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url><loc>https://example.com/</loc></url>
  <url><loc>https://example.com/about</loc></url>
</urlset>
```

### 8.3 Open Graph（分享卡片）

```html
<meta property="og:title" content="张三的简历">
<meta property="og:description" content="全栈工程师">
<meta property="og:image" content="https://example.com/cover.png">
<meta property="og:url" content="https://example.com">
```

### 8.4 JSON-LD（结构化数据）

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "张三",
  "jobTitle": "全栈工程师",
  "url": "https://example.com"
}
</script>
```

谷歌看到后能在搜索结果里展示头像、职位等增强信息。

---

## 九、本周大实战：语义化简历页

### 任务

写一份完整简历页，包含个人信息、工作经历、项目、联系表单。

### 完整答案

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>张三 · 全栈工程师</title>
  <meta name="description" content="张三的个人简历与作品集">
</head>
<body>

  <header>
    <h1>张三</h1>
    <p>全栈工程师 · 北京</p>
    <nav>
      <ul>
        <li><a href="#about">关于</a></li>
        <li><a href="#exp">经历</a></li>
        <li><a href="#contact">联系</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <section id="about">
      <h2>关于我</h2>
      <p>5 年经验的全栈工程师，擅长 <mark>Python</mark> 与 <mark>React</mark>。</p>
      <figure>
        <img src="me.jpg" alt="张三的工作照" loading="lazy">
        <figcaption>2024 年于公司年会</figcaption>
      </figure>
    </section>

    <section id="exp">
      <h2>工作经历</h2>
      <article>
        <h3>ABC 公司 · 高级工程师</h3>
        <time datetime="2022-03">2022.03</time> - 至今
        <ul>
          <li>负责后端 API 开发</li>
          <li>带 5 人小组</li>
        </ul>
      </article>
    </section>

    <section id="contact" aria-labelledby="contact-title">
      <h2 id="contact-title">联系我</h2>
      <form action="/submit" method="POST">
        <fieldset>
          <legend>留言</legend>
          <label>姓名：<input type="text" name="name" required></label>
          <label>邮箱：<input type="email" name="email" required></label>
          <label>留言：<textarea name="msg" required minlength="10"></textarea></label>
          <button type="submit">发送</button>
        </fieldset>
      </form>
    </section>
  </main>

  <footer>
    <p>&copy; <time datetime="2026">2026</time> 张三</p>
  </footer>

</body>
</html>
```

把它保存为 `index.html`，浏览器打开。**没有任何样式**也应该结构清晰、可滚动、可点击锚链跳转。

---

## ✅ 周末 Checklist

- [ ] 能解释 HTML / CSS / JS 三层职责
- [ ] head 里能背出五件套
- [ ] 不再用 `<div>` 当 header / nav / footer
- [ ] 每张 `<img>` 都有 `alt`
- [ ] 表单有 `<label for>` 配对
- [ ] 知道 input 的 10 种 type
- [ ] 用 Tab 能走完整个页面
- [ ] 写完了简历页

## 🐛 本周常见错误

| 现象 | 原因 | 解决 |
|------|------|------|
| 中文乱码 | 没写 `<meta charset="UTF-8">` | 头部加上 |
| 手机上字超小 | 没写 viewport | 加 `viewport` meta |
| `<img>` 显示叉 | 路径错或没有 alt | 检查路径，加 alt |
| 表单提交后页面消失 | form 没 action | 加 action 或 `onsubmit="return false"` |
| Tab 不到自定义按钮 | 用了 `<div onclick>` | 改成 `<button>` |
| `<h1>` 用了好几个 | 滥用 | 一页 1 个 h1 |

## 📚 资源

- MDN HTML 参考：https://developer.mozilla.org/zh-CN/docs/Web/HTML
- W3C 校验：https://validator.w3.org/
- 可访问性自查工具：axe DevTools（Chrome 扩展）
- Schema.org：https://schema.org/
- HTML 模板生成：emmet 速记法（VS Code 自带）

下周开始上"妆"——CSS 见！
