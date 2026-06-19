---
chapter: 03
week: 02
duration: 7天
tags: [CSS, 选择器, 盒模型, Flexbox, Grid, 响应式]
---

# W02 · CSS 基础与 Flex/Grid

> 学习目标：能不查文档写出居中、两栏、三栏、卡片网格等任意布局。

---

## 一、CSS 是什么？

### 1.1 一个比喻

HTML 给你"素颜的人"，CSS 是化妆师。
HTML 给你"毛坯房"，CSS 是装修队。

```
素颜：<h1>张三</h1>
化妆：h1 { color: red; font-size: 32px; }
       ↓
   ╔═══════╗
   ║ 张三  ║   <- 大、红、加粗
   ╚═══════╝
```

### 1.2 三种引入方式

```html
<!-- 1. 行内（不推荐，难维护） -->
<h1 style="color: red;">标题</h1>

<!-- 2. 内部 -->
<style>
  h1 { color: red; }
</style>

<!-- 3. 外部 ✅ 推荐 -->
<link rel="stylesheet" href="style.css">
```

> 永远用**外部 css 文件**：缓存可复用、关注点分离、团队协作友好。

---

## 二、选择器全集

### 2.1 基础选择器

```css
/* 标签 */         h1     { color: red; }
/* 类 */           .box   { color: red; }
/* ID */           #app   { color: red; }   /* 一页只用一次 */
/* 属性 */         input[type="email"] { border: 1px solid blue; }
/* 通用 */         *      { box-sizing: border-box; }
```

### 2.2 组合选择器

```css
.parent .child     /* 后代（任意层级） */
.parent > .child   /* 直接子元素 */
h1 + p             /* h1 后紧跟的 p（兄弟） */
h1 ~ p             /* h1 之后的所有 p */
```

### 2.3 伪类（状态）

```css
a:hover            /* 鼠标悬停 */
input:focus        /* 输入框聚焦 */
button:active      /* 按下瞬间 */
li:nth-child(odd)  /* 奇数行 */
li:first-child
li:last-child
input:checked      /* 勾选状态 */
input:disabled
```

### 2.4 伪元素（虚构内容）

```css
p::first-letter { font-size: 2em; }     /* 段首字母 */
p::first-line   { font-weight: bold; }  /* 段首行 */
.btn::before    { content: "▶ "; }      /* 在元素前插入内容 */
.btn::after     { content: " ←"; }
::selection     { background: yellow; } /* 选中时高亮 */
```

### 2.5 现代选择器（必学）

```css
/* :has() — 如果包含什么 */
.card:has(img) { padding: 0; }

/* :is() — 简写多个选择器 */
:is(h1, h2, h3) { font-family: serif; }

/* :where() — 像 :is() 但优先级为 0 */
:where(h1, h2) { margin: 0; }

/* :not() — 排除 */
li:not(.active) { color: gray; }
```

### 2.6 优先级（specificity）计算

公式：`(行内, ID, 类/属性/伪类, 标签/伪元素)` 比大小，越左权重越高。

| 选择器 | 优先级 |
|--------|--------|
| `h1` | 0,0,0,1 |
| `.btn` | 0,0,1,0 |
| `#app` | 0,1,0,0 |
| `style="..."` | 1,0,0,0 |
| `!important` | 直接核武器（避免使用） |

```css
.btn { color: red; }       /* 0,0,1,0 */
button.btn { color: blue; } /* 0,0,1,1 ← 赢 */
```

### 2.7 继承与级联

**继承**：`color`/`font-*`/`line-height` 等会从父元素传给子元素，但 `border`/`margin` 不会。

**级联**：同优先级时，**后写的覆盖先写的**。

---

## 三、颜色与单位

### 3.1 颜色表示法

```css
color: red;                  /* 关键字 */
color: #ff0000;              /* 十六进制 */
color: #f00;                 /* 缩写 */
color: rgb(255, 0, 0);       /* RGB */
color: rgba(255, 0, 0, 0.5); /* 带透明度 */
color: hsl(0, 100%, 50%);    /* 色相-饱和-亮度 */
color: oklch(0.7 0.2 30);    /* 现代颜色（感知均匀） */
```

| 何时用 |  |
|--------|--|
| `#hex` | 设计稿给的颜色 |
| `rgba` | 需要透明度 |
| `hsl` | 调"同色不同亮"很方便 |
| `oklch` | 2025+ 推荐，颜色过渡更自然 |

### 3.2 单位

| 单位 | 含义 | 何时用 |
|------|------|--------|
| `px` | 像素（绝对） | 边框、阴影 |
| `em` | 相对父元素 font-size | 内边距 |
| `rem` | 相对根（html）font-size | **首选**字号、间距 |
| `%` | 相对父元素 | 宽度 |
| `vw` / `vh` | 视口宽/高 1% | 全屏布局 |
| `dvh` | 动态视口高（解决手机地址栏） | 全屏 hero |
| `ch` | 一个 "0" 字符宽 | 文章宽度（约 65ch 最佳阅读） |

```css
html { font-size: 16px; }
.title { font-size: 2rem; }   /* = 32px */
.body  { max-width: 65ch; }    /* ~65 个字符宽 */
.hero  { height: 100dvh; }     /* 占满屏幕 */
```

---

## 四、盒模型

每个元素都是一个**盒子**，由内到外四层：

```
┌──────────── margin ────────────┐
│ ┌───────── border ──────────┐  │
│ │ ┌────── padding ──────┐   │  │
│ │ │     content         │   │  │
│ │ │  (width × height)   │   │  │
│ │ └────────────────────┘    │  │
│ └─────────────────────────┘   │
└────────────────────────────────┘
```

```css
.box {
  width: 200px;
  padding: 20px;     /* 内边距 */
  border: 2px solid; /* 边框 */
  margin: 10px;      /* 外边距 */
}
```

### 4.1 致命陷阱：默认 box-sizing

默认 `content-box`：`width=200px` 是**内容**宽度，加上 padding+border 实际占 244px。

**全局修复（每个项目第一行）**：

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
```

`border-box` 让 `width` 包含 padding 和 border，所见即所得。

---

## 五、显示模式

| display | 特性 | 示例 |
|---------|------|------|
| `block` | 独占一行，可设宽高 | div / p / h1 |
| `inline` | 跟文字同行，**不可设宽高** | span / a |
| `inline-block` | 同行 + 可设宽高 | 按钮、图标 |
| `none` | 完全消失（不占空间） | 隐藏 |
| `flex` / `grid` | 现代布局 | 见下文 |

```css
span { display: block; }   /* 让 span 像 div 一样独占行 */
img  { display: block; }   /* 去除图片底部莫名空隙 */
```

---

## 六、定位 position

| 值 | 行为 |
|----|------|
| `static` | 默认，不偏移 |
| `relative` | 相对**自己原位置**偏移 |
| `absolute` | 相对**最近的 relative 父元素** |
| `fixed` | 相对**视口**（固定在屏幕） |
| `sticky` | 滚动到边界变 fixed |

```
relative: 占原位置 + 偏移
┌────────┐
│ A原位  │ ← 仍占空间
└────────┘
   └──→ A 偏移到这里（透明占位仍在）

absolute: 脱离文档流
┌────────┐
│ B 飘走  │ ← 不再占空间，其他元素挤上来
└────────┘
```

```css
.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);  /* 真正居中 */
}

.sticky-header {
  position: sticky;
  top: 0;     /* 滚到顶端就吸附 */
}
```

### 6.1 z-index

数字越大越靠上，但**只对 position 不为 static 的元素生效**。

```
z-index: 10  ← 顶层（modal）
z-index: 5   ← 中层（dropdown）
z-index: 1   ← 底层（header）
```

> 🐛 坑：父元素 z-index=1 时，子元素 z-index=999 也飞不出父元素的"层"。叫**层叠上下文**。

---

## 七、Flexbox 完全指南

> 一维布局神器（横或纵）。**写法**：父元素加 `display: flex`。

### 7.1 容器属性

```css
.container {
  display: flex;
  flex-direction: row;       /* row(默认) | column | row-reverse */
  flex-wrap: wrap;           /* nowrap(默认) | wrap */
  justify-content: center;   /* 主轴对齐 */
  align-items: center;       /* 交叉轴对齐 */
  gap: 16px;                 /* 项目间距 */
}
```

### 7.2 justify-content（主轴）视图

```
flex-start  : [A][B][C]·····
center      : ····[A][B][C]····
flex-end    : ·····[A][B][C]
space-between: [A]···[B]···[C]
space-around : ·[A]··[B]··[C]·
space-evenly : ··[A]··[B]··[C]··
```

### 7.3 align-items（交叉轴）

```
stretch (默认):       flex-start:        center:         flex-end:
┌──┐┌──┐┌──┐         ┌──┐┌──┐┌──┐       ····           ····
│  ││  ││  │ 拉满高    │AA││BB││CC│      [A][B][C]      ····
│  ││  ││  │         └──┘└──┘└──┘       ····          [A][B][C]
└──┘└──┘└──┘
```

### 7.4 项目属性

```css
.item {
  flex-grow: 1;     /* 剩余空间分配比例（0=不分） */
  flex-shrink: 1;   /* 空间不够时收缩比例 */
  flex-basis: auto; /* 初始大小 */
  flex: 1;          /* 缩写：1 1 0 = 自动平分剩余空间 */
  align-self: end;  /* 单独覆盖 align-items */
  order: 2;         /* 重新排序，数字越小越靠前 */
}
```

### 7.5 万能居中（一行解决）

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

### 7.6 动手任务 1：导航栏

让 logo 在左，菜单在右，所有项垂直居中。

```html
<nav class="nav">
  <div class="logo">MySite</div>
  <ul class="menu">
    <li>首页</li><li>博客</li><li>关于</li>
  </ul>
</nav>
```

<details>
<summary>答案</summary>

```css
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
}
.menu {
  display: flex;
  gap: 24px;
  list-style: none;
}
```
</details>

---

## 八、Grid 完全指南

> 二维布局神器（横+纵同时）。

### 8.1 基础

```css
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 100px;   /* 三列：固定/弹性/固定 */
  grid-template-rows: 80px auto 60px;       /* 三行 */
  gap: 16px;
}
```

```
┌──────┬───────────────┬──────┐
│ 200px│      1fr      │ 100px│  80px
├──────┼───────────────┼──────┤
│      │               │      │  auto
├──────┼───────────────┼──────┤
│      │               │      │  60px
└──────┴───────────────┴──────┘
```

`fr` = fraction，**剩余空间的份数**。

### 8.2 区域命名（最直观）

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: 60px 1fr 40px;
  grid-template-areas:
    "header header"
    "side   main"
    "footer footer";
}
.header { grid-area: header; }
.side   { grid-area: side; }
.main   { grid-area: main; }
.footer { grid-area: footer; }
```

直接画图！⭐⭐⭐⭐⭐

### 8.3 自适应卡片网格（神技）

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
}
```

效果：每张卡至少 240px，能放几张就放几张，剩余平分。**不写媒体查询自动响应式！**

| 函数 | 作用 |
|------|------|
| `repeat(3, 1fr)` | 重复 3 次 |
| `repeat(auto-fit, ...)` | 能塞几个塞几个，剩余空间填满 |
| `repeat(auto-fill, ...)` | 同上但留空槽位 |
| `minmax(200px, 1fr)` | 最小 200，最大 1fr |

### 8.4 项目跨多格

```css
.featured {
  grid-column: 1 / 3;       /* 占 1-3 列 */
  grid-row: span 2;          /* 跨 2 行 */
}
```

### 8.5 place-items 速写

```css
.center {
  display: grid;
  place-items: center;     /* 一句话居中 */
  height: 100vh;
}
```

---

## 九、媒体查询 @media

### 9.1 移动优先（推荐）

**先写小屏，再用 min-width 加桌面样式**：

```css
/* 默认：移动端 */
.container { padding: 16px; }

/* 平板及以上 */
@media (min-width: 768px) {
  .container { padding: 32px; }
}

/* 桌面 */
@media (min-width: 1024px) {
  .container { padding: 48px; max-width: 1200px; margin: 0 auto; }
}
```

### 9.2 常用断点

| 断点 | 设备 |
|------|------|
| 640px | sm 大手机 |
| 768px | md 平板 |
| 1024px | lg 笔记本 |
| 1280px | xl 桌面 |

---

## 十、本周大实战

### 10.1 复刻 GitHub 主页（简化版）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>GitHub Clone</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, sans-serif; color: #1f2328; }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 32px;
    background: #24292f;
    color: white;
  }
  .header nav ul {
    display: flex;
    gap: 16px;
    list-style: none;
  }
  .header a { color: white; text-decoration: none; }

  .hero {
    text-align: center;
    padding: 80px 16px;
    background: linear-gradient(135deg, #0d1117, #1f6feb);
    color: white;
  }
  .hero h1 { font-size: clamp(32px, 6vw, 64px); }
  .hero p  { margin-top: 16px; font-size: 20px; opacity: 0.9; }
  .hero button {
    margin-top: 24px;
    padding: 12px 24px;
    background: #2da44e;
    color: white;
    border: 0;
    border-radius: 6px;
    font-size: 16px;
    cursor: pointer;
  }

  .features {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
    gap: 24px;
    padding: 64px 32px;
    max-width: 1200px;
    margin: 0 auto;
  }
  .card {
    padding: 24px;
    border: 1px solid #d0d7de;
    border-radius: 8px;
  }
  .card h3 { margin-bottom: 12px; }
</style>
</head>
<body>

<header class="header">
  <strong>GitHub</strong>
  <nav>
    <ul>
      <li><a href="#">Features</a></li>
      <li><a href="#">Pricing</a></li>
      <li><a href="#">Sign in</a></li>
    </ul>
  </nav>
</header>

<section class="hero">
  <h1>Built for developers</h1>
  <p>Where the world builds software</p>
  <button>Sign up for free</button>
</section>

<section class="features">
  <article class="card">
    <h3>Repositories</h3>
    <p>Host your code with version control.</p>
  </article>
  <article class="card">
    <h3>Issues</h3>
    <p>Track bugs and feature requests.</p>
  </article>
  <article class="card">
    <h3>Actions</h3>
    <p>Automate your workflow with CI/CD.</p>
  </article>
</section>

</body>
</html>
```

保存为 `github.html` 双击打开看效果。

### 10.2 Frontend Mentor

去 [frontendmentor.io](https://www.frontendmentor.io) 注册，做 2 个 newbie 项目（免费）：
- QR code component
- Order summary card

---

## ✅ 周末 Checklist

- [ ] 能背出选择器优先级公式
- [ ] 项目第一行写 `* { box-sizing: border-box; }`
- [ ] 知道 px / em / rem / vw 区别
- [ ] 不再用 `float` 布局
- [ ] 一行代码用 flex 居中
- [ ] 一行代码用 grid 写自适应卡片
- [ ] 写了至少 1 个完整页面

## 🐛 本周常见错误

| 现象 | 原因 | 解决 |
|------|------|------|
| 整个页面白屏 | CSS 文件路径错 | 检查 `<link href>` |
| 居中失败 | 父元素没设高度 | 父元素 `height: 100vh` |
| 宽度溢出 | 没用 border-box | 全局 `* { box-sizing: border-box; }` |
| margin 重叠 | 相邻元素 margin 取最大值 | 改用 padding 或 gap |
| z-index 不生效 | 没设 position | 加 `position: relative` |
| 图片下面有空白 | img 默认 inline | `img { display: block; }` |
| flex 子项不收缩 | flex-shrink 默认 1 但内容太长 | `min-width: 0` |

## 📚 资源

- MDN CSS：https://developer.mozilla.org/zh-CN/docs/Web/CSS
- Flexbox 游戏：https://flexboxfroggy.com
- Grid 游戏：https://cssgridgarden.com
- 小抄：https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- 配色：https://coolors.co
- Frontend Mentor：https://www.frontendmentor.io

下周让网页"动起来"！
