---
title: 第 03 章 - HTML / CSS 现代 Web
chapter: 03
duration: 4 周
prerequisite: [[02-JavaScript-从零到源码/02-JavaScript总纲]]
tags: [HTML, CSS, 前端]
---

# 第 03 章 · HTML / CSS 现代 Web（4 周）

> HTML/CSS 不是"会写"就够，要做到**像素级还原 + 性能 + 可访问性**。

## 🎯 章节目标

- 写出语义化、可访问（a11y）的 HTML5
- 掌握 Flex / Grid / Container Query 等现代布局
- 会写响应式、暗黑模式、动画
- 理解浏览器渲染管线（Parse → Style → Layout → Paint → Composite）
- 能像素级还原任意 UI 设计稿
- 掌握 Tailwind CSS（2026 年事实标准）

## 📅 周计划

| 周 | 主题 | 关键产出 |
|---|------|----------|
| W1 | HTML5 语义化 + 可访问性 + SEO + Web APIs | 个人简历页 |
| W2 | CSS 基础 + Flex + Grid + 响应式 | 仿 GitHub 主页 |
| W3 | CSS 进阶：变量 / 动画 / 滤镜 / 暗黑模式 / 容器查询 | 仿 Apple 产品页 |
| W4 | Tailwind CSS + 渲染原理 + 性能 | 仿 Linear / Stripe 落地页 |

## 📖 详细内容

### W1 - HTML5
- 语义化标签（header/nav/main/article/section/aside/footer）
- 表单 + 校验（required/pattern/type）
- 可访问性 ARIA 属性 + 键盘导航
- Open Graph / Twitter Card / JSON-LD（SEO）
- 现代 Web APIs：Intersection Observer / ResizeObserver / Web Storage / Clipboard / Web Share

### W2 - CSS 基础与布局
- 选择器优先级 + 继承
- 盒模型 + box-sizing
- 定位（static/relative/absolute/fixed/sticky）
- **Flexbox 完全指南**（[CSS-Tricks Flex](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)）
- **Grid 完全指南**（[CSS-Tricks Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)）
- 媒体查询 + 移动优先

### W3 - CSS 进阶
- CSS 变量 + 主题切换（暗黑模式）
- 动画：transition / @keyframes / animation
- transform 3D + will-change（性能）
- 滤镜：filter / backdrop-filter
- 现代特性：`:has()` / `@container` / `subgrid`
- Logical Properties（margin-inline 等）
- 必读：张鑫旭《CSS 世界》《CSS 选择器世界》《CSS 新世界》

### W4 - Tailwind + 性能
- Tailwind CSS 思路 + 配置 + 最佳实践
- shadcn/ui（基于 Tailwind 的组件方案）
- 渲染管线：Parse → Style → Layout → Paint → Composite
- 重排（Reflow）vs 重绘（Repaint）vs 合成（Composite）
- Critical CSS / 字体加载策略 / 图片懒加载
- Lighthouse + Web Vitals（LCP / FID / CLS / INP）

## 📝 考核任务

- [ ] **T1**：纯 HTML 写一份语义化简历，通过 [WAVE](https://wave.webaim.org/) 0 错误
- [ ] **T2**：[Frontend Mentor](https://www.frontendmentor.io/) 完成 5 个 newbie 项目
- [ ] **T3**：实现仅 CSS 的：手风琴 / 模态框 / 步骤条 / 切换开关
- [ ] **T4**：实现暗黑模式切换（含系统偏好检测 + 本地保存 + 平滑过渡）
- [ ] **T5**：用 CSS Grid 复刻一份杂志排版
- [ ] **T6**：用 Tailwind 复刻 [shadcn/ui](https://ui.shadcn.com/) 中 10 个组件
- [ ] **T7**：把任意一个之前做的页面性能优化到 Lighthouse 95+

## 🚀 章节大项目：像素级还原 3 个真实站点

要求**不看 F12**，仅看视觉效果还原（容器宽度允许 2px 内偏差）。

### 项目 A：[Linear](https://linear.app/) 落地页（中等难度）
- 复杂的渐变 + 滚动动画
- 视频背景 + Hero
- 响应式（移动 + 桌面）
- **技术栈**：HTML + Tailwind CSS + 极少 JS

### 项目 B：[Stripe](https://stripe.com/) 首页（高难度）
- 双层视差滚动
- 复杂卡片悬浮动画
- 全局色彩主题切换
- **技术栈**：HTML + Tailwind + GSAP（动画库）

### 项目 C：[Apple AirPods Pro](https://www.apple.com/airpods-pro/) 产品页（地狱难度）
- 滚动驱动的 3D 旋转/缩放
- 3-4 段视频拼接成连续滚动
- 文字与媒体的精确同步
- **技术栈**：HTML + Tailwind + Lenis（平滑滚动）+ ScrollTrigger

### 验收标准
- [ ] 三个项目全部部署到 Vercel，公网可访问
- [ ] 移动端 + 平板 + 桌面三档响应式
- [ ] Lighthouse 性能 ≥ 90，可访问性 = 100
- [ ] 总 JS 体积 < 100KB（gzip）
- [ ] 截图对比 90% 相似度（用 [BackstopJS](https://github.com/garris/BackstopJS) 测试）

### 进阶挑战
- 用 Astro 做 SSG 输出
- 加 i18n 中英双语切换
- 用 [Motion One](https://motion.dev/) 替代 GSAP（更小）

## 📚 推荐资源

### 文档
- 📖 [MDN HTML/CSS（中文）](https://developer.mozilla.org/zh-CN/docs/Web)
- 📖 [web.dev/learn](https://web.dev/learn/) - Google 官方现代 Web 课
- 📖 [Tailwind 中文文档](https://www.tailwindcss.cn/)

### 必读博客
- 📖 [张鑫旭博客](https://www.zhangxinxu.com/wordpress/) - 中文 CSS 第一博
- 📖 [CSS-Tricks](https://css-tricks.com/)
- 📖 [Josh Comeau Blog](https://www.joshwcomeau.com/) - 高质量交互式 CSS 教程

### 书
- 📘 《CSS 揭秘》Lea Verou
- 📘 张鑫旭《CSS 世界》三部曲

### 练习
- 🛠️ [Frontend Mentor](https://www.frontendmentor.io/) - 真实设计稿挑战
- 🛠️ [CSS Battle](https://cssbattle.dev/) - 最少代码复现 UI
- 🛠️ [100 Days CSS](https://100dayscss.com/)

### 灵感
- 🎨 [Awwwards](https://www.awwwards.com/)
- 🎨 [CodePen](https://codepen.io/)

---

✅ 三个站点像素级还原 → 进入 [[04-前端框架-React/04-React总纲]]
