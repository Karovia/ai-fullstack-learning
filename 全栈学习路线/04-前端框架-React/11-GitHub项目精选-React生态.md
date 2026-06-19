---
title: GitHub 项目精选 · React 生态
chapter: 04
type: resource-pack
tags: [github, react, nextjs, zustand, tanstack, react-hook-form, 项目精选]
difficulty: 🌱→🌲
updated: 2026-06-19
---

# 📦 GitHub 项目精选：React 生态

> React 学到中段，最容易卡在「我会写组件了，但我不知道一个**真·项目**该怎么组织」。
> 这份清单按 **入门 → 进阶 → 源码级** 的顺序，带你看 React 生态各个层次的标杆项目。
>
> 难度：🌱 入门 / 🌿 进阶 / 🌲 高阶。
> Star 数为 **2026 年 6 月估值**。

---

## 🌱 入门 - React 起步

### `facebook/create-react-app` · ~103k ⭐ （已弃用，但值得当历史读）
- **链接**：<https://github.com/facebook/create-react-app>
- **是什么**：React 官方脚手架，2023 起官方不再维护。
- **为什么仍要看**：理解 webpack + babel 的"老 React 工具链"长什么样，对你看历史代码极有帮助。

### `vitejs/vite` · ~75k ⭐
- **链接**：<https://github.com/vitejs/vite>
- **是什么**：现代前端的事实标准构建工具。
- **学什么**：esbuild + Rollup 双引擎、HMR 工作原理。Vite 文档本身就是绝佳的「现代构建科普」。

### `PacktPublishing/React-Projects` · ~1k ⭐
- 配套 Packt React Projects 一书的源码。
- 每章一个完整小项目，结构规范度比"30 days"系列高。

### `enaqx/awesome-react` · ~67k ⭐
- 进入 React 世界的索引页之一，按主题分类的资源大全。

### `brillout/awesome-react-components` · ~43k ⭐
- 只列**值得用**的 React 组件库（人工审过）。
- 写真实项目缺组件时第一个翻的清单。

---

## 🌿 React 进阶项目模板

### `vercel/next.js` · ~130k ⭐
- **链接**：<https://github.com/vercel/next.js>
- **是什么**：React 服务端框架的事实标准（SSR / RSC / Middleware 全都它定义的）。
- **必读子目录**：`packages/next/src/server`（理解 RSC 是怎么序列化的）。

### `shadcn-ui/taxonomy` · ~17k ⭐
- **链接**：<https://github.com/shadcn-ui/taxonomy>
- **是什么**：shadcn 作者本人写的 Next.js 13+ 完整产品级模板。
- **必看**：app router + auth + Stripe 订阅 + MDX 文档站，**当年定义了 Next.js App Router 的最佳实践**。

### `shadcn-ui/next-template` · ~3k ⭐
- shadcn/ui 的最小 starter，做新项目最先 clone 它。

### `alan2207/bulletproof-react` · ~26k ⭐ ⭐ ⭐ 必读
- **链接**：<https://github.com/alan2207/bulletproof-react>
- **是什么**：被几乎所有 senior 推荐的 React 项目结构指南。
- **学什么**：feature-based 目录、API 层、表单层、错误边界、测试层的「教科书级」组织方式。
- **建议**：把它的 README 当书读 3 遍。

### `steven-tey/precedent` · ~7k ⭐
- Vercel 官方推荐 starter，作者是 dub.co 创始人。
- **学什么**：Auth.js + Tailwind + Vercel Postgres 的"最小生产配置"。

### `ixartz/Next-js-Boilerplate` · ~10k ⭐
- 国内独立开发者最爱用的脚手架之一，i18n / Sentry / Husky / Lint-Staged 全到位。

### `t3-oss/create-t3-app` · ~27k ⭐
- **链接**：<https://github.com/t3-oss/create-t3-app>
- **是什么**：「类型安全全栈」哲学的代表，Next.js + tRPC + Prisma + Tailwind + NextAuth。
- **学什么**：什么叫端到端类型安全（E2E type safety）。

---

## 🌲 React 源码级

> 想突破前端瓶颈，必须读源码。但 React 本体太复杂，先从迷你实现入手。

### `facebook/react` · ~230k ⭐
- 本体太重，**新人不要直接读**。先把下面三个仓搞懂再回来。

### `BetaSu/big-react` · ~7k ⭐ ⭐ ⭐ 中文最佳
- **链接**：<https://github.com/BetaSu/big-react>
- **作者**：卡颂（《React 设计原理》作者）。
- **是什么**：从 0 实现 React 的逐 commit 教学项目。
- **学什么**：Fiber、Reconciler、Scheduler 怎么从无到有。
- **配套书**：《React 技术揭秘》《React 设计原理》。

### `acdlite/react-fiber-architecture` · ~10k ⭐
- React Fiber 设计文档（核心成员 Andrew Clark 亲笔）。
- 不长，但是 React 16+ 一切的基石。

### `pomber/didact` · ~9k ⭐
- **链接**：<https://github.com/pomber/didact>
- 300 行手写 React，配套博客 [《Build your own React》](https://pomb.us/build-your-own-react/)。
- **新人首读**：1 小时读完，立刻理解 vDOM / Reconciler / Hooks 怎么跑。

### `preactjs/preact` · ~37k ⭐
- 3KB 的 React 兼容实现，源码可读性远高于 React 本体。
- **学什么**：在保留 API 兼容的前提下如何极致瘦身。

---

## 🎨 状态 / 数据 / 表单

### `pmndrs/zustand` · ~50k ⭐ ⭐ ⭐ 必读源码
- **链接**：<https://github.com/pmndrs/zustand>
- **代码量**：核心 ~200 行 TypeScript。
- **学什么**：怎么用 `useSyncExternalStore` 做出比 Redux 简单 10 倍但同样强大的状态库。
- **后面有「读 zustand 源码 1 周计划」专门拆解。**

### `TanStack/query` (React Query) · ~45k ⭐
- 服务端状态管理事实标准。
- **学什么**：cache + stale-while-revalidate + 请求去重的工业级实现。

### `TanStack/router` · ~10k ⭐
- 类型安全路由的最强实现，比 Next.js Router 更适合 SPA。

### `react-hook-form/react-hook-form` · ~42k ⭐
- 表单库的性能王者，几乎不引发重渲染。
- **学什么**：用 `useRef` + 订阅模式做表单的设计哲学。

### `colinhacks/zod` · ~36k ⭐
- TypeScript schema validator 的事实标准。
- 配合 react-hook-form / tRPC / Next.js 形成「类型安全铁三角」。

### `pmndrs/jotai` · ~20k ⭐
- 原子化状态库，思路像 Recoil 但更简洁。

---

## 🛠️ 工具集 / Hooks

### `streamich/react-use` · ~42k ⭐
- 100+ Hooks 的瑞士军刀。
- **学什么**：每个 Hook 的实现都是一个独立小练习。

### `uidotdev/usehooks` · ~10k ⭐
- ui.dev 出品的精品 Hooks 集，质量比 react-use 更精。

### `alibaba/hooks` (ahooks) · ~14k ⭐
- 中文 Hooks 库的事实标准，文档双语。
- **学什么**：企业级 Hook 的边界处理（取消、防抖、SSR）。

### `mantinedev/mantine` 内置 hooks
- `@mantine/hooks` 单独可用，约 50 个高质量 hook。

---

## 🎬 动画 / 视觉

### `framer/motion` · ~26k ⭐
- 现代 React 动画第一选择，API 直觉到怀疑人生。

### `pmndrs/react-spring` · ~28k ⭐
- 物理弹簧动画，与 react-three-fiber 是绝配。

### `studio-freight/lenis` · ~11k ⭐
- 顶级官网常用的 smooth-scroll 库。
- 配合 GSAP ScrollTrigger 能做出 Awwwards 级体验。

### `greensock/GSAP` · ~20k ⭐
- 专业前端动画工业标准，2024 起完全免费。
- **配套库**：Flip / ScrollTrigger / SplitText。

---

## 🏆 Easy-vibe React 项目（中文友好/小而完整）

### John Smilga 系列
- **GitHub**：<https://github.com/john-smilga>
- 70+ 个由浅入深的 React 实战仓库，每个都附 YouTube 课程。

### `bradtraversy/react-projects`
- Brad 出品的 React 教学项目集，适合英文听力练习。

### `florinpop17/app-ideas` · ~85k ⭐
- **链接**：<https://github.com/florinpop17/app-ideas>
- 700+ 个项目灵感，按难度分级。
- **学什么**：当你想自己练习但「没题做」的时候来这里。

### `gothinkster/react-redux-realworld-example-app`
- RealWorld 项目的 React 实现（Medium 克隆），同需求不同框架对比利器。

### `excalidraw/excalidraw` · ~95k ⭐ ⭐ ⭐ 前端宝典
- **链接**：<https://github.com/excalidraw/excalidraw>
- **是什么**：手绘风协作白板。
- **为什么必读**：canvas + 协作 + 大量 React 性能技巧、虚拟列表、Worker 集成。
- **后面有「做 excalidraw 简版的路线」拆解。**

### `outline/outline` · ~30k ⭐
- 开源 Notion 替代，前后端全栈典范，Slate.js + Koa。

---

## 💼 真实顶级 React 应用源码

### `vercel/next.js` 自己
- Next.js 自己的官网就是用 Next.js 写的（在 `examples/` 和 `docs/` 里）。

### `vuejs/vitepress` (虽是 Vue，但值得读)
- 文档站架构思路对 React 也通用。

### `TanStack/table` · ~26k ⭐
- 业界最强 Headless Table，500k+ 行数据流畅。
- **学什么**：headless 模式 + plugin 架构。

### `tldraw/tldraw` · ~37k ⭐
- 协作白板，比 excalidraw 更工程化。
- **学什么**：CRDT、向量图形系统、Shape 插件机制。

---

## 📅 专题 1：读 zustand 源码 1 周计划

zustand 是个**「200 行教科书」**，新人读源码的最佳第一站。

| 天 | 内容 |
|---|---|
| Day 1 | 通读 README，跑 5 个 demo，建立体感 |
| Day 2 | 读 `src/vanilla.ts`（核心 createStore，无 React） |
| Day 3 | 读 `src/react.ts`（怎么对接 useSyncExternalStore） |
| Day 4 | 读 `middleware/persist.ts`（中间件模式） |
| Day 5 | 读 `middleware/immer.ts` + `devtools.ts` |
| Day 6 | 自己用 200 行实现一个 mini-zustand |
| Day 7 | 写一篇「我读 zustand 源码学到了什么」博客 |

**关键收获**：
- 发布订阅 pattern 的 React 18 安全实现
- 为什么 selector 要做浅比较
- 中间件如何洋葱模型组合

---

## 🛣 专题 2：做 excalidraw 简版的路线（4 周）

### Week 1：白板雏形
- canvas 基础：line / rect / freedraw 三种工具
- 鼠标事件 → 数据模型 → 渲染循环
- 状态用 zustand

### Week 2：交互升级
- 选中、移动、缩放（Transform Handles）
- 撤销/重做（command pattern）
- 键盘快捷键

### Week 3：导出 + 持久化
- 导出 PNG / SVG
- IndexedDB 本地保存（用 Dexie）
- 多文档切换

### Week 4：协作雏形
- WebSocket + 增量同步
- 引入 Yjs CRDT
- 部署到 Vercel + Render

做完这个，你的简历可以直接挂出来，面试 P6 不慌。

---

## 🧭 总结：按你目前阶段挑

- **刚学完 Hooks** → bulletproof-react + create-t3-app
- **想突破组件设计** → shadcn/ui + radix
- **想懂状态管理底层** → zustand + jotai 源码
- **想做炫酷官网** → framer-motion + lenis + gsap
- **想冲 P6/P7** → React 源码 → big-react → tldraw

> 记住：React 的水平上限不是看你会多少 API，而是**你读过多少高质量项目**。
