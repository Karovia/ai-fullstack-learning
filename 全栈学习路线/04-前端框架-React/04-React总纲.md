---
title: 第 04 章 - React 从使用到源码
chapter: 04
duration: 10 周
prerequisite: [[03-HTML-CSS-现代Web/03-HTML-CSS总纲]]
tags: [React, 前端, 源码级]
---

# 第 04 章 · React 从使用到源码（10 周）

> 选 React 是因为：1) 工业界占比最大；2) Anthropic/Vercel 等头部公司主推；3) 学完 React 思想，Vue/Solid 等都好懂。

## 🎯 章节目标

- 用 React 18+/19 写出生产级应用
- 完全掌握 Hooks 心智模型 + 自定义 Hook
- 用 Next.js App Router 做 SSR/RSC
- 状态管理：Zustand / Jotai / TanStack Query
- 表单：React Hook Form + Zod
- **手写 mini-react（含 Fiber + 调度器 + Hooks）**
- 读懂 React 真实源码：reconciler / scheduler / fiber

## 📅 周计划

| 周 | 主题 | 难度 | 产出 |
|---|------|------|------|
| W1 | JSX + 组件 + Props + 列表/条件渲染 | 入门 | 计算器 + Todo |
| W2 | Hooks 全集：useState/useEffect/useRef/useMemo/useCallback | 入门→进阶 | 数据看板 |
| W3 | 自定义 Hook + Context + useReducer | 进阶 | 主题/i18n/Auth Hook |
| W4 | 状态管理：Zustand + TanStack Query | 进阶 | 真实 API 应用 |
| W5 | 路由 + 表单：React Router / RHF + Zod | 进阶 | 后台管理系统 |
| W6 | Next.js App Router + RSC + Server Action | 进阶 | 全栈博客 |
| W7 | 测试：Vitest + Testing Library + Playwright | 进阶 | 测试套件 |
| W8 | 性能优化：React DevTools / Profiler / 虚拟列表 | 进阶 | 性能报告 |
| W9 | 手写 mini-react v1：JSX → Fiber → Render | 源码级 | mini-react |
| W10 | 手写 mini-react v2：Hooks + Diff + 调度器 | 源码级 | 完整 mini-react |

## 📖 详细内容

### W1 - JSX 与组件基础
- React 18/19 心智模型
- JSX 编译产物（React.createElement）
- 函数组件 vs 类组件（类组件只读不写）
- Props + Children + key
- 条件 / 列表渲染最佳实践

### W2 - Hooks 全集
- useState（函数式更新、惰性初始化）
- useEffect（依赖数组、清理函数、闭包陷阱）
- useRef（DOM 引用 + 可变值）
- useMemo / useCallback（**只在性能敏感处用**）
- useTransition / useDeferredValue（React 18 并发）
- use Hook（React 19 新增）

### W3 - 自定义 Hook + Context
- Context + useContext
- useReducer（复杂状态用）
- 自定义 Hook 设计原则（命名 use*、不破坏 hook 规则）
- 经典 Hook：useLocalStorage / useDebounce / useFetch / useEventListener

### W4 - 状态管理 + 数据获取
- Zustand（最轻量，2026 主流）
- Jotai（原子化）
- TanStack Query（服务端状态）
- SWR
- 何时用 Context、何时用 Zustand、何时用 Query

### W5 - 路由 + 表单
- React Router v6/v7（loader / action）
- React Hook Form（性能最佳）
- Zod（schema 校验）
- 文件上传 / 富文本编辑器（Tiptap）

### W6 - Next.js
- App Router 架构
- Server Components vs Client Components
- Server Actions
- Streaming + Suspense
- 数据获取（fetch + cache）
- 部署到 Vercel

### W7 - 测试
- Vitest 单元测试
- React Testing Library（行为驱动）
- Mock：MSW（API 拦截）
- E2E：Playwright

### W8 - 性能优化
- React DevTools Profiler 用法
- React.memo / useMemo / useCallback 三剑客（不要滥用）
- 虚拟列表：TanStack Virtual
- Code splitting + lazy + Suspense
- 编译器优化：React Compiler（自动 memo）

### W9-10 - 手写 mini-react（核心！）

跟着卡颂 [big-react](https://github.com/BetaSu/big-react) 一步步实现：
- W9 v1：JSX 转换 → Element Tree → Fiber Tree → Commit DOM
- W10 v2：调度器（lane 模型）+ Diff 算法 + Hooks 实现 + 并发渲染

**必看**：[React 技术揭秘 - 卡颂](https://react.iamkasong.com/)

## 📝 考核任务

- [ ] **T1**：实现一个 Todo 应用，含拖拽排序、过滤、本地持久化
- [ ] **T2**：实现 5 个自定义 Hook（useDebounce / useLocalStorage / useFetch / useOnClickOutside / useMediaQuery）
- [ ] **T3**：用 Zustand + TanStack Query 重写 T1
- [ ] **T4**：用 RHF + Zod 做一个含 10 个字段的复杂表单
- [ ] **T5**：用 Next.js App Router 做一个 SSR 博客（数据来自 markdown 文件）
- [ ] **T6**：给任意项目加测试，覆盖率 ≥ 70%
- [ ] **T7**：性能优化案例：将一个慢渲染场景从 60ms 优化到 < 16ms，写报告
- [ ] **T8**：跟着 big-react 仓库手写 mini-react，能跑通官方 7 个 demo
- [ ] **T9**：写一篇博客《React Fiber 是什么，为什么需要它》(≥ 5000 字)
- [ ] **T10**：阅读 React 真实源码 `packages/react-reconciler/src/ReactFiberBeginWork.new.js`，做行级注释

## 🚀 章节大项目：在线协作白板（Excalidraw 简化版）

### 需求
做一个可以多人实时协作的白板：
- 画矩形 / 圆 / 箭头 / 文字
- 拖拽 / 缩放 / 旋转
- 撤销 / 重做（Command 模式）
- 多人光标 + 实时同步（用 [Yjs](https://github.com/yjs/yjs) CRDT）
- 房间 + 分享链接
- 导出 PNG / SVG / JSON

### 技术栈
- React 19 + TypeScript
- Vite + pnpm
- Zustand（本地状态）
- Yjs + y-websocket（CRDT 协作）
- TanStack Query（服务端状态）
- Tailwind + shadcn/ui
- Konva.js / fabric.js（Canvas 库，二选一）
- 后端用 Node.js + Hono（W5 章节会学）
- 部署：前端 Vercel，后端 Railway/Fly.io

### 验收标准
- [ ] 单人画板功能完整
- [ ] 双人 / 三人实时协作不冲突
- [ ] 网络抖动恢复后能自动合并
- [ ] 撤销/重做无 bug
- [ ] 移动端可用（触摸事件）
- [ ] Lighthouse 性能 ≥ 85
- [ ] E2E 测试覆盖 5 个核心场景
- [ ] 公网可访问，README 含演示 GIF

### 进阶挑战
- 加 AI 智能图形识别（手绘转规范图形）
- 接入 Liveblocks 替换自建后端
- 导出为可编辑的 Excalidraw 格式

## 📚 推荐资源

### 入门
- 📖 [React 中文新文档](https://zh-hans.react.dev/) - **首选，2024 重写**
- 📖 [Next.js 中文文档](https://www.nextjs.cn/)
- 📘 《React 设计原理》卡颂

### 进阶
- 📖 [Patterns.dev](https://www.patterns.dev/) - 现代 Web 模式
- 🛠️ [Bulletproof React](https://github.com/alan2207/bulletproof-react) - 工业级最佳实践
- 🛠️ [taxonomy](https://github.com/shadcn-ui/taxonomy) - shadcn 作者出品的 Next.js 模板

### 源码级
- 📖 [React 技术揭秘 - 卡颂](https://react.iamkasong.com/) - **中文最系统**
- 🛠️ [big-react](https://github.com/BetaSu/big-react) - 跟做 mini-react
- 🛠️ [react-source](https://github.com/facebook/react)
- 📖 [React 源码解读 - 贺师俊](https://github.com/AttackXiaoJinJin/reactExplain)

### 视频
- 🎥 [Theo - t3.gg](https://www.youtube.com/@t3dotgg) - 现代 React 生态
- 🎥 [Lee Robinson](https://leerob.io/) - Next.js 官方 DevRel

---

✅ 完成项目 → 进入 [[05-后端开发-FastAPI-Node/05-后端总纲]]
