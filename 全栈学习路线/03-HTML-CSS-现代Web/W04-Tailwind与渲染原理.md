---
chapter: 03
week: 04
duration: 7天
tags: [Tailwind, shadcn, 浏览器渲染, 性能优化, Lighthouse]
---

# W04 · Tailwind 与渲染原理

> 学习目标：用 Tailwind 一天写完一周的页面 + 看懂浏览器是怎么把 HTML 变成像素的。

---

## 一、为什么用 Tailwind？

### 1.1 痛点

写普通 CSS 的真实日常：

```html
<button class="primary-btn">提交</button>
```

```css
.primary-btn { ... 20 行 ... }
.primary-btn:hover { ... }
@media (min-width: 768px) { .primary-btn { ... } }
```

**问题**：
- 起类名累（`.card-header-title-wrapper-inner`）
- CSS 文件越写越大
- 改一处怕影响别处
- 不同人写出"五种按钮"

### 1.2 Tailwind 的解决方案

```html
<button class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-md md:px-6">
  提交
</button>
```

**0 行 CSS**，直接在 HTML 里组合**工具类**（utility-first）。

| 对比 | 传统 CSS | Tailwind |
|------|---------|----------|
| 起名 | 想很久 | 不需要 |
| 改样式 | 跳两个文件 | 改一行 |
| 删除组件 | CSS 残留 | HTML 删掉就完事 |
| 包大小 | 越写越大 | PurgeCSS 只打包用到的 |

> 你担心的"HTML 太长"在 VS Code 里有自动折行 + Tailwind 智能提示，写起来比传统 CSS 快 3 倍。

---

## 二、安装与配置

### 2.1 Vite 项目

```bash
npm create vite@latest my-app
cd my-app
npm install
npm install -D tailwindcss@latest @tailwindcss/vite
```

`vite.config.js`：

```js
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
});
```

`src/index.css`：

```css
@import "tailwindcss";
```

`src/main.js`：

```js
import './index.css';
```

完成！`npm run dev` 启动。

### 2.2 验证

```html
<h1 class="text-3xl font-bold text-blue-600">Hello Tailwind</h1>
```

如果字大、加粗、蓝色，就成了。

---

## 三、工具类思维

### 3.1 间距类（spacing）

每个数字 = `0.25rem` = `4px`。

```html
<div class="p-4">    <!-- padding: 16px -->
<div class="px-6">   <!-- padding-left/right: 24px -->
<div class="py-2">   <!-- padding-top/bottom: 8px -->
<div class="m-8">    <!-- margin: 32px -->
<div class="mt-auto"><!-- margin-top: auto -->
```

| 数字 | 像素 |
|------|------|
| 1 | 4 |
| 2 | 8 |
| 4 | 16 |
| 6 | 24 |
| 8 | 32 |
| 12 | 48 |
| 16 | 64 |

### 3.2 颜色

```html
<div class="bg-blue-500 text-white">
<div class="text-red-600">
<div class="border border-gray-300">
```

每种颜色有 50/100/200/.../900 共 11 档（数字越大越深）。

### 3.3 排版

```html
<p class="text-sm">小字</p>
<p class="text-base">正常</p>
<p class="text-lg">大字</p>
<p class="text-2xl font-bold">大标题</p>
<p class="leading-relaxed tracking-wide">行高与字距</p>
```

### 3.4 Flex / Grid 速写

```html
<!-- flex 居中 -->
<div class="flex items-center justify-center h-screen">

<!-- 网格 -->
<div class="grid grid-cols-3 gap-4">

<!-- 自适应 -->
<div class="grid grid-cols-[repeat(auto-fit,minmax(240px,1fr))] gap-4">
```

---

## 四、响应式

加前缀切换断点：

```html
<div class="text-sm md:text-base lg:text-lg">
  小屏小字 → 中屏正常 → 大屏更大
</div>

<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
  <!-- 手机 1 列、平板 2 列、桌面 4 列 -->
</div>
```

| 前缀 | 起始宽度 |
|------|---------|
| 默认 | 0+ (移动) |
| `sm:` | 640px |
| `md:` | 768px |
| `lg:` | 1024px |
| `xl:` | 1280px |
| `2xl:` | 1536px |

**移动优先**：默认是手机样式，加前缀往大屏覆盖。

---

## 五、状态前缀

```html
<button class="bg-blue-500 hover:bg-blue-600 active:bg-blue-700 focus:ring-2">
  按钮
</button>

<input class="border border-gray-300 focus:border-blue-500 invalid:border-red-500">

<!-- 暗黑模式 -->
<div class="bg-white dark:bg-gray-900 text-black dark:text-white">
```

| 前缀 | 含义 |
|------|------|
| `hover:` | 鼠标悬停 |
| `focus:` | 聚焦 |
| `active:` | 按下 |
| `disabled:` | 禁用 |
| `dark:` | 暗黑模式 |
| `group-hover:` | 父元素 hover 时子元素生效 |
| `peer-checked:` | 兄弟勾选时生效 |

---

## 六、任意值

类没覆盖到的尺寸用方括号写：

```html
<div class="w-[317px] h-[42vh] top-[12.5%]">
<div class="bg-[#1a73e8]">
<div class="grid-cols-[200px_1fr_120px]">
```

任何 CSS 值都能塞进 `[]`。

---

## 七、自定义主题

`tailwind.config.js`：

```js
export default {
  theme: {
    extend: {
      colors: {
        brand: {
          50:  '#eff6ff',
          500: '#2563eb',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui'],
      },
      borderRadius: {
        '4xl': '2rem',
      },
    },
  },
};
```

之后就能用 `bg-brand-500`、`rounded-4xl`。

---

## 八、组件抽离 @apply

如果一组类反复用，抽出来：

```css
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-md;
  }
}
```

```html
<button class="btn-primary">提交</button>
```

> 但建议**先组件化（React 组件）**而不是抽 CSS 类，更现代化。

---

## 九、shadcn/ui 简介

> 不是传统组件库，而是"复制粘贴的组件源码"。基于 Tailwind + Radix UI。

```bash
npx shadcn@latest init
npx shadcn@latest add button card dialog input
```

它把组件源码**复制到你项目里**（`components/ui/button.tsx`），你想改就改，不依赖远程包。

特点：
- 完全可定制
- 可访问性内置
- 设计精致
- 配合 Next.js 极佳

业内 2025 年最流行的组件方案。

---

## 十、浏览器渲染管线

### 10.1 全图

```
HTML 文本
   │
   ▼
[解析] ──→ DOM 树（Document Object Model）
   │
CSS 文本
   │
   ▼
[解析] ──→ CSSOM 树（CSS Object Model）
   │
   ▼
[合成] ──→ Render Tree（渲染树）
   │
   ▼
[Layout 布局] —— 计算每个元素的大小、位置
   │
   ▼
[Paint 绘制] —— 把元素画成像素
   │
   ▼
[Composite 合成] —— 把多层叠加输出到屏幕
```

### 10.2 关键渲染路径优化

1. **CSS 是渲染阻塞资源**：`<link rel="stylesheet">` 不下载完不渲染
   - 把首屏 CSS 内联在 `<head>`（Critical CSS）
   - 非首屏 CSS 用 `media="print"` + JS 动态切换

2. **JS 默认阻塞解析**：用 `defer` 或 `async`
   ```html
   <script src="app.js" defer></script>
   ```

3. **预加载关键资源**：
   ```html
   <link rel="preload" href="font.woff2" as="font" crossorigin>
   ```

---

## 十一、重排 / 重绘 / 合成

| 阶段 | 触发什么 | 性能 |
|------|----------|------|
| **Layout（重排 reflow）** | 元素大小、位置、文档结构变化 | 💸 最贵 |
| **Paint（重绘 repaint）** | 颜色、阴影、可见性变化 | 中等 |
| **Composite（合成）** | transform / opacity 变化 | ⚡ 最便宜 |

### 11.1 触发清单

**触发 Layout（避免高频）**：
- `width / height / margin / padding`
- `top / left / right / bottom`
- `display / position`
- `font-size`

**只触发 Paint**：
- `color / background-color`
- `box-shadow / border-radius`
- `visibility`

**只触发 Composite**（GPU 加速）：
- `transform`
- `opacity`
- `filter`

### 11.2 性能口诀

> 动画用 `transform` 和 `opacity`，永远别用 `top/left`。

❌ 卡顿写法：
```css
@keyframes move {
  to { left: 200px; }   /* 每帧重排 */
}
```

✅ 流畅写法：
```css
@keyframes move {
  to { transform: translateX(200px); }   /* 只合成 */
}
```

### 11.3 will-change 提示

```css
.animated {
  will-change: transform;  /* 预告浏览器：我要动了，提前准备 GPU 层 */
}
```

⚠️ 只在**真要动的元素**用，滥用反而吃内存。

---

## 十二、字体加载策略

```css
@font-face {
  font-family: 'Inter';
  src: url('inter.woff2') format('woff2');
  font-display: swap;   /* 关键 */
}
```

| font-display | 行为 |
|--------------|------|
| `auto` | 浏览器决定（可能空白 3 秒） |
| `block` | 等字体，不显示文字（最差） |
| `swap` | 先显示后备字体，加载完再换（推荐） |
| `fallback` | 稍等一下再用后备 |
| `optional` | 网慢就放弃 |

---

## 十三、图片优化

```html
<picture>
  <source srcset="img.avif" type="image/avif">
  <source srcset="img.webp" type="image/webp">
  <img
    src="img.jpg"
    alt="..."
    width="800"
    height="600"
    loading="lazy"
    decoding="async"
    srcset="img-400.jpg 400w, img-800.jpg 800w"
    sizes="(max-width: 600px) 100vw, 50vw"
  >
</picture>
```

**省 60%+ 流量**，**LCP 提升 30%+**。

| 格式 | 体积 | 兼容 |
|------|------|------|
| JPG | 100% | 全部 |
| WebP | 60% | 现代浏览器 |
| AVIF | 40% | 2024+ 浏览器 |

---

## 十四、Web Vitals

谷歌官方核心三指标：

| 指标 | 含义 | 优秀 |
|------|------|------|
| **LCP** | 最大内容渲染时间 | < 2.5s |
| **INP** | 交互到反馈延迟 | < 200ms |
| **CLS** | 视觉稳定度（不抖） | < 0.1 |

### 14.1 Lighthouse

Chrome DevTools → Lighthouse → 跑分。

得分组成：
- Performance 性能
- Accessibility 可访问性
- Best Practices 最佳实践
- SEO

目标：**95+ 全绿**。

### 14.2 DevTools

- **Performance** 面板：录制操作，查看哪一帧卡顿
- **Coverage** 面板：查看 CSS/JS 用了多少（删除未用代码）
- **Network** 面板：看资源加载瀑布图

---

## 十五、本周大实战

### 15.1 用 Tailwind 复刻 shadcn/ui 卡片

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>Tailwind Card</title>
<script src="https://cdn.tailwindcss.com"></script>   <!-- 临时玩用 CDN -->
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-4">

<div class="max-w-sm bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-2xl transition-shadow">
  <img src="https://picsum.photos/400/200" alt="" class="w-full h-48 object-cover">
  <div class="p-6">
    <span class="inline-block px-2 py-1 text-xs font-semibold text-blue-700 bg-blue-100 rounded-full">
      新品
    </span>
    <h2 class="mt-3 text-xl font-bold text-gray-900">商品标题</h2>
    <p class="mt-2 text-gray-600 text-sm leading-relaxed">
      这是一段商品描述，用来展示卡片的内容布局。
    </p>
    <div class="mt-4 flex items-center justify-between">
      <span class="text-2xl font-bold text-gray-900">¥299</span>
      <button class="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors">
        立即购买
      </button>
    </div>
  </div>
</div>

</body>
</html>
```

### 15.2 性能挑战：让 Lighthouse 95+

新建一个商品列表页（10 张图 + 一段文字 + 几个按钮），跑 Lighthouse，按下面清单优化：

**Checklist**：
- [ ] 所有 `<img>` 加 `loading="lazy"` + `width`/`height`
- [ ] 用 `<picture>` 提供 WebP/AVIF
- [ ] CSS 内联首屏 critical 部分
- [ ] JS 加 `defer`
- [ ] 字体 `font-display: swap`
- [ ] 启用 GZIP/Brotli（部署后）
- [ ] 删除未使用的 CSS（Coverage 面板）
- [ ] 大图压缩到 < 200KB
- [ ] 用 `transform` 做动画
- [ ] 元素都有显式宽高（防 CLS）

跑完目标：
```
Performance:    95+
Accessibility:  100
Best Practices: 100
SEO:            100
```

---

## ✅ 周末 Checklist

- [ ] 能用 Tailwind 不查文档写完整页面
- [ ] 知道响应式前缀 sm/md/lg
- [ ] 自定义过 tailwind.config 主题
- [ ] 用过 shadcn/ui 安装组件
- [ ] 能画浏览器渲染管线流程图
- [ ] 知道哪些属性触发 layout/paint/composite
- [ ] 会读 Lighthouse 报告
- [ ] Web Vitals 三大指标背得出

## 🐛 本周常见错误

| 现象 | 原因 | 解决 |
|------|------|------|
| Tailwind 类名不生效 | content 没配 / 文件路径错 | 检查 `tailwind.config.js` 的 content |
| 生产打包巨大 | 没用 PurgeCSS | Tailwind v3+ 默认开启 |
| 暗黑模式不切换 | 没在 `<html>` 加 `class="dark"` | 用 JS 切换 |
| CLS 太高 | 图片没设 width/height | 都加上 |
| LCP > 4s | 大图没优化 / 字体阻塞 | WebP + font-display:swap |
| 动画掉帧 | 用 left/top 动画 | 改成 transform |

## 📚 资源

- Tailwind 官方：https://tailwindcss.com
- 速查：https://nerdcave.com/tailwind-cheat-sheet
- shadcn/ui：https://ui.shadcn.com
- Web Vitals：https://web.dev/vitals
- 渲染原理：https://developer.mozilla.org/zh-CN/docs/Web/Performance/How_browsers_work
- 图片压缩：https://squoosh.app
- 字体优化：https://fonts.google.com（带 `&display=swap` 参数）

---

## 🎓 第 03 章总结

恭喜！你完成了 **HTML/CSS/响应式/动画/Tailwind/性能** 的完整学习。

**你现在能做的事**：
- 写出语义化、可访问、SEO 友好的 HTML 页面
- 不查文档用 Flex/Grid 做任意布局
- 实现暗黑模式、动画、毛玻璃等现代视觉效果
- 用 Tailwind 1 小时搭出精致界面
- 把页面 Lighthouse 跑到 95+

**下一章预告**：第 04 章 React/Next.js —— 让网页变成真正的"应用"。
