---
title: GitHub 项目精选 · CSS 与现代设计
chapter: 03
type: resource-pack
tags: [github, html, css, tailwind, shadcn, design-system, 项目精选]
difficulty: 🌱→🌲
updated: 2026-06-19
---

# 📦 GitHub 项目精选：CSS 与现代设计

> 这一章我们已经把 HTML 语义、CSS 现代特性、布局、动画、Tailwind 工程化都学过一轮。
> 接下来，**真正让你水平起飞的方式只有一个：读真·项目源码、抄真·设计稿**。
>
> 这份清单是我个人 review 过的「学 CSS 必看 GitHub 仓库」，按从易到难分组。
> 难度标记：🌱 入门 / 🌿 进阶 / 🌲 高阶。
> Star 数为 **2026 年 6 月估值**，仅供你判断热度，不必迷信。

---

## 🌱 入门 - 跟着抄就能学会

> 这层的目标只有一个：**把手感练出来**。
> 不要思考架构，就盯着设计稿一个一个抄完。

### `frontendmentor-io/community`（Frontend Mentor 社区） · ~3k ⭐
- **链接**：<https://www.frontendmentor.io>
- **是什么**：每周提供真实 Figma 设计稿，让你独立还原成代码。
- **为什么必学**：CSS 学不进去 90% 是因为没有「目标参照物」。这里把目标给你了。
- **建议刷法**：从 Newbie 级别开始，每天 1 个，做满 20 个再看 Junior 难度。

### `bradtraversy/50projects50days` · ~37k ⭐
- **链接**：<https://github.com/bradtraversy/50projects50days>
- **是什么**：50 个纯 HTML/CSS/JS 小项目，每个 100~200 行。
- **为什么必学**：CSS 动画、过渡、伪类、selector 用法的「微型练习册」。
- **建议刷法**：自己先写 10 分钟，写不出来再看答案。

### `AhmadShaheer/30-Days-Of-HTML` · ~1k ⭐
- **链接**：<https://github.com/AhmadShaheer/30-Days-Of-HTML>
- **是什么**：30 天 HTML 闯关，每天一个语义化练习。
- **为什么必学**：弥补很多前端「只会 div」的硬伤。

### 30-Day-CSS-Challenge 类项目（如 `harshwardhanmalpani/30-day-css-challenge`）
- **是什么**：30 天 CSS 挑战，每天画一个图形/UI 元素。
- **为什么必学**：训练你「用 CSS 画画」的肌肉记忆。

### Kevin Powell 的 CodePen 集合
- **链接**：<https://codepen.io/kevinpowell>
- **是什么**：YouTube CSS 教父，所有教程的源码都开源在 CodePen。
- **配套频道**：[Kevin Powell @ YouTube](https://youtube.com/@KevinPowell)
- **为什么必学**：他每条视频都会回答一个具体问题（如「为什么 height: 100% 不生效？」），看完你的 CSS 直觉会强很多。

---

## 🌿 CSS 深度学习

> 这一层进入「我不仅要会用，还要懂原理」阶段。

### `necolas/normalize.css` · ~49k ⭐
- **链接**：<https://github.com/necolas/normalize.css>
- **是什么**：跨浏览器样式归一化的事实标准。
- **学什么**：每行 CSS 旁边都有注释，告诉你**为什么浏览器默认值会有坑**。读完一遍，等于读了半本《CSS 浏览器兼容性手册》。

### `csstools/sanitize.css` · ~5k ⭐
- **是什么**：normalize.css 的现代继任者，预设更激进。
- **对比学**：normalize 保守、sanitize 强势。读 diff 你就懂「现代 reset」是什么取舍。

### `animate-css/animate.css` · ~81k ⭐
- **链接**：<https://github.com/animate-css/animate.css>
- **是什么**：最经典的预制 CSS 动画库。
- **学什么**：所有动画都是纯 `@keyframes`，是学 CSS 动画语法的最佳教科书。

### `michalsnik/aos`（Animate On Scroll） · ~26k ⭐
- **是什么**：滚动到视窗就触发动画的经典库。
- **学什么**：IntersectionObserver + CSS 变量的精妙结合。

### `AllThingsSmitty/css-protips` · ~27k ⭐
- **链接**：<https://github.com/AllThingsSmitty/css-protips>
- **是什么**：100+ 条 CSS 实用小技巧。
- **学什么**：每天通勤路上读 5 条，3 周读完，脑子里会有大量「啊原来还能这样」的瞬间。

---

## 🌲 现代 CSS 工程化

> 这一层进入企业生产环境必备。

### `tailwindlabs/tailwindcss` · ~85k ⭐
- **链接**：<https://github.com/tailwindlabs/tailwindcss>
- **是什么**：原子化 CSS 的事实标准。
- **学什么**：除了 API 之外，**读 `oxide` 引擎源码**（Rust 编写，2024 重写），理解一个 CSS 框架如何做到 5ms 编译。

### `shadcn-ui/ui` · ~80k ⭐ ⭐ ⭐ 现象级
- **链接**：<https://github.com/shadcn-ui/ui>
- **是什么**：2025 年最火的「不是组件库的组件库」——你 copy 源码到自己项目。
- **必读**：`apps/www/registry/default/ui/*.tsx`，每个组件都是 Radix + Tailwind 的范本。
- **学什么**：什么叫「分发即拥有」的新一代组件分发哲学。

### `radix-ui/themes` · ~16k ⭐
- **是什么**：Radix 官方完整主题方案，shadcn 的灵感来源。
- **学什么**：可访问性（a11y）的标杆做法。

### `chakra-ui/chakra-ui` · ~38k ⭐
- **是什么**：第一个把「Style Props」推到主流的库。
- **学什么**：组件库 API 设计中的「composability」。

### `mui/material-ui` · ~95k ⭐
- **是什么**：Material Design 的 React 实现，企业级首选之一。
- **学什么**：复杂主题系统、CSS-in-JS 的工程化典范（v5 后切换到 emotion）。

### `mantinedev/mantine` · ~28k ⭐
- **是什么**：100+ 组件 + 50+ Hooks 的全家桶。
- **学什么**：API 设计的「合理克制」——既不像 MUI 那么重，也不像 shadcn 那么轻。

### `emotion-js/emotion` · ~17k ⭐
- **是什么**：CSS-in-JS 的现代代表。
- **学什么**：运行时样式注入的性能优化思路。

### `styled-components/styled-components` · ~41k ⭐
- **是什么**：CSS-in-JS 元老。
- **学什么**：标签模板字符串 + 主题 Context 的开山玩法。

---

## 🎨 UI 资源 / 设计灵感

### `troxler/awesome-css-frameworks` · ~9k ⭐
- 一份你能找到的最全 CSS 框架索引，按类型分类。

### CodePen 精选合集
- 关注 [@CodePen](https://codepen.io/codepen) Picks。
- 推荐订阅：CodePen Spark 周刊。

### `tailwindlabs/heroicons` · ~22k ⭐
- Tailwind 官方图标库，SVG 源码本身就是 SVG 学习材料。

### `lucide-icons/lucide` · ~13k ⭐
- Feather 的现代 fork，shadcn 默认图标库。
- **为什么推荐**：每个 icon 都有清晰的 24×24 grid 规范，对学 SVG 极友好。

### `iconify/iconify` · ~5k ⭐
- 200k+ 图标聚合方案，按需加载。
- **学什么**：图标基础设施怎么设计。

---

## 🏆 Easy-vibe 风格（中文友好 / 美感优先）

### `chokcoco/CSS-Inspiration` · ~5k ⭐
- **链接**：<https://github.com/chokcoco/CSS-Inspiration>
- **作者**：腾讯 ChokCoco（小肆）。
- **是什么**：中文世界 CSS 灵感库的天花板。
- **每篇都附 CodePen Demo**，从 hover 特效到滤镜、混合模式、SVG 滤镜、CSS 着色器一应俱全。

### `MaximeHeckel/blog.maximeheckel.com` · ~1.2k ⭐
- **链接**：<https://github.com/MaximeHeckel/blog.maximeheckel.com>
- **是什么**：高互动博客的 OSS 源码（Next.js + Framer Motion + 自研 design system）。
- **学什么**：博客即作品集的极致形态。

### `josephnlu/30-days-of-css-experiments` 类项目
- 自我挑战的「每日一图」库，看作者 30 天能从青铜变王者。

---

## 🎓 大师博客（订阅级别推荐）

### Josh W. Comeau
- **GitHub**：<https://github.com/joshwcomeau>
- **博客**：<https://www.joshwcomeau.com>
- **代表作**：CSS for JS Devs 课程；个人博客每篇都是「交互式 CSS 教学」。

### Una Kravets
- **GitHub**：<https://github.com/una>
- **代表作**：1-line CSS layouts、CSS Houdini 推广。
- **关键词**：Chrome DevRel + 容器查询布道者。

### Adam Argyle
- **GitHub**：<https://github.com/argyleink>
- **代表作**：`open-props`、Chrome `gui-challenges`。
- **学什么**：现代 CSS（`@layer`、`color-mix()`、`@scope`）的最前沿。

---

## 🛠️ 实战：像素级还原 Landing Page

> 找到一个你审美喜欢的官网，**fork 一份源码或自己 1:1 还原**，是进阶最快的路径。

推荐目标（公开源码或可右键查看的）：

1. **Linear-style 模板**：`weijunext/landing-page-boilerplate`、`Skolaczk/next-starter`
2. **Stripe-style 模板**：搜索 `stripe-clone`，`thedevdojo/genesis` 是开源精品
3. **Vercel 风格**：`vercel/examples`、`steven-tey/precedent`
4. **Apple 风格**：`shadcn` 的个人作品集 `shadcn/shadcn.com`

**还原方法论**：
- 第 1 天：截图 + 用 Figma 量出所有间距、字号
- 第 2 天：搭骨架（HTML 语义 + Tailwind 容器）
- 第 3 天：填内容 + 字体（注意字间距 letter-spacing）
- 第 4 天：动画与过渡
- 第 5 天：深浅色模式 + 响应式断点

做满 3 个 landing page，你的 CSS 在国内就能进 P5 序列了。

---

## 📑 一周学习路径建议

| 天 | 任务 |
|---|---|
| Day 1 | 注册 frontendmentor，挑 1 个 newbie 题做完 |
| Day 2 | 读 normalize.css 全文（带注释） |
| Day 3 | 读 chokcoco/CSS-Inspiration 前 10 篇 |
| Day 4 | clone shadcn-ui/ui，读 5 个组件源码 |
| Day 5 | 选一个 landing page 1:1 还原 |
| Day 6 | 学 framer-motion 基础 + 给 Day 5 加动画 |
| Day 7 | 复盘，把不会的 CSS 属性整理成 Anki 卡 |

---

## 🧭 总结：什么时候看什么

- **写不出动画** → animate.css + Kevin Powell
- **响应式画不对** → Frontend Mentor 强练
- **想做高级感** → MaximeHeckel + Josh Comeau
- **想做生产级组件库** → shadcn/ui + radix + mantine
- **想研究底层引擎** → tailwindcss/oxide

> 记住：**CSS 是肌肉记忆驱动的，看 100 遍不如自己做 3 遍。**
