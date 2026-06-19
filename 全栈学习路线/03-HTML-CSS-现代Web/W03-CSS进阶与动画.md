---
chapter: 03
week: 03
duration: 7天
tags: [CSS变量, 暗黑模式, 动画, transform, transition, 容器查询]
---

# W03 · CSS 进阶与动画

> 学习目标：让网页"会动"。掌握主题切换、过渡、关键帧动画、滚动驱动动画。

---

## 一、CSS 变量 var(--name)

### 1.1 基础语法

```css
:root {
  --primary: #2563eb;
  --radius: 8px;
  --gap: 16px;
}

.button {
  background: var(--primary);
  border-radius: var(--radius);
  padding: var(--gap);
}
```

`:root` ≈ `<html>`，**全局变量**写在这里。

### 1.2 局部变量

```css
.card {
  --card-padding: 24px;
  padding: var(--card-padding);
}
.card.compact {
  --card-padding: 12px;   /* 覆盖 */
}
```

### 1.3 与 JS 交互

```js
document.documentElement.style.setProperty('--primary', '#ff0000');
```

变量可被 JS 动态改变，**整个网站颜色立即换肤**。

---

## 二、主题切换（暗黑模式）

### 2.1 系统偏好探测

```css
@media (prefers-color-scheme: dark) {
  body { background: #0a0a0a; color: #f5f5f5; }
}
```

### 2.2 现代写法 light-dark()

```css
:root {
  color-scheme: light dark;
}
body {
  background: light-dark(#fff, #0a0a0a);
  color: light-dark(#0a0a0a, #fff);
}
```

一行解决浅色/深色，不用写媒体查询！

### 2.3 完整切换实战

```html
<!DOCTYPE html>
<html lang="zh-CN" data-theme="light">
<head>
<meta charset="UTF-8">
<style>
  :root {
    --bg: #ffffff;
    --text: #0a0a0a;
    --card: #f5f5f5;
  }
  [data-theme="dark"] {
    --bg: #0a0a0a;
    --text: #f5f5f5;
    --card: #1a1a1a;
  }
  body {
    background: var(--bg);
    color: var(--text);
    transition: background .3s, color .3s;  /* 平滑过渡 */
    font-family: sans-serif;
    padding: 32px;
  }
  .card {
    background: var(--card);
    padding: 24px;
    border-radius: 8px;
    margin-top: 16px;
  }
  button {
    padding: 8px 16px;
    border: 0;
    border-radius: 6px;
    cursor: pointer;
  }
</style>
</head>
<body>
  <button onclick="toggleTheme()">切换主题</button>
  <div class="card">这是一张卡片</div>

  <script>
    function toggleTheme() {
      const html = document.documentElement;
      const next = html.dataset.theme === 'light' ? 'dark' : 'light';
      html.dataset.theme = next;
      localStorage.setItem('theme', next);
    }
    // 启动时读取
    document.documentElement.dataset.theme = localStorage.getItem('theme') || 'light';
  </script>
</body>
</html>
```

---

## 三、选择器进阶

### 3.1 :has() —— 父选择器（终于来了）

```css
/* 卡片里有图片时去掉 padding */
.card:has(img) { padding: 0; }

/* 表单中的 input 聚焦时高亮 label */
label:has(input:focus) { color: blue; }

/* 列表项被勾选 */
li:has(input:checked) { background: yellow; }
```

### 3.2 :is() / :where()

```css
/* 旧 */
header h1, header h2, header h3 { color: red; }

/* 新 */
header :is(h1, h2, h3) { color: red; }

/* :where() 与 :is() 一样，但优先级为 0 —— 容易被覆盖 */
:where(article) p { line-height: 1.6; }
```

### 3.3 :not()

```css
li:not(.active) { color: gray; }
button:not([disabled]):hover { background: blue; }
```

---

## 四、容器查询 @container（新一代响应式）

### 4.1 痛点

媒体查询基于**视口**，但卡片本身可能在侧栏 / 主区，宽度不同样式应不同。

### 4.2 写法

```css
.parent {
  container-type: inline-size;   /* 声明为容器 */
  container-name: card;
}

@container card (min-width: 400px) {
  .card { display: flex; gap: 16px; }
}
```

**卡片的样式根据它所在的容器宽度变**，而不是整个屏幕。组件复用之神。

---

## 五、逻辑属性

中文从左到右、阿拉伯文从右到左。**逻辑属性自动适配**。

| 物理 | 逻辑 |
|------|------|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `margin-top` | `margin-block-start` |
| `padding-left/right` | `padding-inline` |
| `padding-top/bottom` | `padding-block` |

```css
/* 不要再写 */
.box { margin-left: 16px; margin-right: 16px; }

/* 写这个 */
.box { margin-inline: 16px; }
```

---

## 六、transform 变换

### 6.1 四大基础

```css
transform: translate(100px, 50px);  /* 平移 */
transform: rotate(45deg);            /* 旋转 */
transform: scale(1.5);               /* 缩放 */
transform: skew(20deg);              /* 倾斜 */

/* 组合 */
transform: translate(50%, -50%) rotate(45deg) scale(1.2);
```

```
原始：    rotate(15deg):    scale(1.3):     translate(20,0):
┌──┐         ┌──┐             ┌──────┐          ┌──┐
│A │          \A \             │  A   │   ····   │A │
└──┘           └──┘             └──────┘          └──┘
```

### 6.2 3D 变换

```css
.card {
  transform: perspective(800px) rotateY(20deg);
}

.parent {
  perspective: 1000px;   /* 设在父元素上效果更稳 */
}
```

`perspective` 越小越夸张（像把眼睛凑近）。

### 6.3 transform-origin

变换的"轴心"，默认是元素中心。

```css
.box {
  transform-origin: top left;   /* 从左上角旋转 */
  transform: rotate(45deg);
}
```

---

## 七、transition 过渡

> 当一个属性"改变"时，让它"花点时间"过渡，而不是瞬间变。

```css
.btn {
  background: blue;
  transition: background 0.3s ease, transform 0.2s;
}
.btn:hover {
  background: red;
  transform: scale(1.1);
}
```

### 7.1 四个属性

```css
transition-property: background;     /* 哪个属性 */
transition-duration: 0.3s;            /* 多久 */
transition-timing-function: ease;     /* 缓动 */
transition-delay: 0.1s;               /* 延迟 */

/* 缩写 */
transition: background 0.3s ease 0.1s;
```

### 7.2 缓动函数

| 关键字 | 曲线 | 感觉 |
|--------|------|------|
| `linear` | 匀速 | 机械 |
| `ease` | 慢-快-慢 | 默认，自然 |
| `ease-in` | 慢-快 | 启动慢 |
| `ease-out` | 快-慢 | 减速到位 |
| `cubic-bezier(.68,-.55,.27,1.55)` | 自定义 | 反弹效果 |

可视化网站：https://cubic-bezier.com

### 7.3 动手任务 1：弹跳按钮

```css
.btn {
  padding: 12px 24px;
  background: #2563eb;
  color: white;
  border: 0;
  border-radius: 8px;
  transition: transform 0.3s cubic-bezier(.68,-.55,.27,1.55);
  cursor: pointer;
}
.btn:hover { transform: scale(1.1) rotate(-2deg); }
.btn:active { transform: scale(0.95); }
```

---

## 八、@keyframes 关键帧动画

> transition 只能管"两点之间"。复杂动画用 @keyframes。

### 8.1 基础

```css
@keyframes bounce {
  0%   { transform: translateY(0); }
  50%  { transform: translateY(-30px); }
  100% { transform: translateY(0); }
}

.ball {
  animation: bounce 1s ease infinite;
}
```

### 8.2 全部属性

```css
animation-name: bounce;
animation-duration: 1s;
animation-timing-function: ease;
animation-delay: 0s;
animation-iteration-count: infinite;   /* 或数字 */
animation-direction: alternate;        /* 来回 */
animation-fill-mode: forwards;         /* 停在最后一帧 */
animation-play-state: running;         /* 或 paused */

/* 缩写 */
animation: bounce 1s ease 0s infinite alternate forwards;
```

### 8.3 案例：加载旋转图标

```css
@keyframes spin {
  to { transform: rotate(360deg); }
}
.spinner {
  width: 32px;
  height: 32px;
  border: 4px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
```

---

## 九、滤镜 filter

### 9.1 基础滤镜

```css
filter: blur(5px);              /* 模糊 */
filter: brightness(1.2);         /* 亮度 */
filter: contrast(1.5);           /* 对比 */
filter: grayscale(100%);         /* 灰度 */
filter: hue-rotate(90deg);       /* 色相 */
filter: drop-shadow(0 4px 8px #0008);

/* 组合 */
filter: blur(2px) brightness(0.8);
```

### 9.2 backdrop-filter（毛玻璃）

```css
.glass {
  background: rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(20px);    /* 给元素**后面**的内容模糊 */
  border: 1px solid rgba(255,255,255,0.4);
  border-radius: 16px;
}
```

iOS 风格毛玻璃效果一行搞定。

---

## 十、渐变

```css
/* 线性 */
background: linear-gradient(135deg, #ff6b6b, #4ecdc4);

/* 径向 */
background: radial-gradient(circle at center, yellow, red);

/* 锥形 */
background: conic-gradient(red, yellow, green, blue, red);

/* 多色 + 位置 */
background: linear-gradient(to right, red 0%, yellow 50%, green 100%);
```

可视化生成：https://cssgradient.io

---

## 十一、阴影

### 11.1 box-shadow

```css
/* x偏移  y偏移  模糊  扩散  颜色 */
box-shadow: 0 4px 8px 0 rgba(0,0,0,0.1);

/* 多层叠加更立体 */
box-shadow:
  0 1px 2px rgba(0,0,0,.05),
  0 4px 8px rgba(0,0,0,.08),
  0 16px 32px rgba(0,0,0,.12);

/* 内阴影 */
box-shadow: inset 0 2px 4px rgba(0,0,0,.2);
```

### 11.2 text-shadow

```css
text-shadow:
  0 1px 0 #ccc,
  0 2px 0 #c9c9c9,
  0 5px 10px rgba(0,0,0,.3);
```

---

## 十二、其他黑科技

### 12.1 clip-path 剪切

```css
.triangle { clip-path: polygon(50% 0, 100% 100%, 0 100%); }
.circle   { clip-path: circle(50%); }
.diamond  { clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%); }
```

可视化：https://bennettfeely.com/clippy

### 12.2 mix-blend-mode

```css
.text-overlay {
  mix-blend-mode: difference;  /* 像 PS 的"差值"图层 */
  color: white;
}
```

### 12.3 scroll-snap

```css
.container {
  scroll-snap-type: x mandatory;
  overflow-x: auto;
  display: flex;
}
.slide {
  scroll-snap-align: center;
  flex: 0 0 100%;
}
```

像滑屏 App 一样按页吸附。

---

## 十三、本周大实战

### 13.1 纯 CSS 模态框（无 JS）

利用 `:target` 伪类：

```html
<a href="#modal">打开</a>
<div id="modal" class="modal">
  <div class="modal-content">
    <a href="#" class="close">×</a>
    <h2>这是模态框</h2>
  </div>
</div>

<style>
.modal {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  pointer-events: none;
  transition: opacity .3s;
}
.modal:target {
  opacity: 1;
  pointer-events: auto;
}
.modal-content {
  background: white;
  padding: 32px;
  border-radius: 12px;
  position: relative;
  transform: scale(.8);
  transition: transform .3s;
}
.modal:target .modal-content { transform: scale(1); }
.close {
  position: absolute;
  top: 8px; right: 16px;
  font-size: 24px;
  text-decoration: none;
  color: #666;
}
</style>
```

### 13.2 Apple 风格按钮

```html
<button class="apple-btn">购买 →</button>

<style>
.apple-btn {
  padding: 14px 32px;
  background: linear-gradient(180deg, #007aff, #0051d5);
  color: white;
  border: 0;
  border-radius: 980px;       /* 大圆角变胶囊 */
  font-size: 17px;
  font-weight: 500;
  cursor: pointer;
  box-shadow: 0 4px 16px rgba(0, 122, 255, 0.4);
  transition: all .25s;
}
.apple-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 122, 255, 0.5);
}
.apple-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.4);
}
</style>
```

### 13.3 滚动驱动动画（@scroll-timeline）

```css
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(40px); }
  to   { opacity: 1; transform: translateY(0); }
}
.section {
  animation: fadeIn linear;
  animation-timeline: view();      /* 元素进入视口时触发 */
  animation-range: entry 0% cover 30%;
}
```

无需 JS、无需 IntersectionObserver，浏览器原生支持滚动动画。

---

## ✅ 周末 Checklist

- [ ] 用 CSS 变量替换写死的颜色
- [ ] 实现暗黑模式 + 平滑过渡
- [ ] 用 :has() 写过 1 次父选择器
- [ ] 写过 @keyframes 动画
- [ ] 知道 transform 比 left/top 性能好
- [ ] 实现过纯 CSS 模态框/手风琴
- [ ] 知道毛玻璃怎么写

## 🐛 本周常见错误

| 现象 | 原因 | 解决 |
|------|------|------|
| 动画卡顿 | 用了 left/top/width 触发 layout | 改用 transform/opacity |
| transition 不生效 | 起点没值（如 display:none → block） | 用 opacity + visibility 替代 |
| transform 不生效 | 用在 inline 元素 | 改为 inline-block 或 block |
| 暗黑模式闪一下白屏 | JS 后才设置 | 头部内联脚本读 localStorage |
| backdrop-filter 没效果 | 父元素背景不透明 | 设半透明背景或祖先无 overflow |
| 动画结束后回弹 | 默认 animation-fill-mode: none | 加 `forwards` |

## 📚 资源

- 动画灵感：https://animista.net
- 缓动可视化：https://easings.net
- clip-path 工具：https://bennettfeely.com/clippy
- CSS-Tricks：https://css-tricks.com
- 渐变生成：https://cssgradient.io

下周用工具加速：Tailwind！
