---
title: 真实开源 SaaS 拆解
type: 资源库 / 源码精读
tags: [saas, 开源, 商业级架构, 源码精读, 毕业项目]
难度: 高级
更新时间: 2026-06-19
---

# 真实开源 SaaS 拆解

## 为什么读真实 SaaS 源码

教程项目和 toy project 教你"怎么写"，但教不会你**商业级软件长什么样**。真实 SaaS 的代码包含了教程从不展示的内容：

- 多租户隔离、计费、配额
- 鉴权、SSO、SCIM、审计日志
- 灰度发布、AB 测试、特性开关
- 后台任务、队列、定时调度
- 监控、告警、错误聚合
- 国际化、可访问性、SEO

读完一个真实 SaaS，你会从"会写功能"升级到"会做产品"。这是**毕业项目最关键的一跃**——也是大厂面试官真正想看到的能力。

下面是 10 个 2026 年最值得拆解的开源 SaaS。每个我都给出**完整的拆解卡片**：定位、技术栈、目录结构、阅读路径、关键技术点、对你毕业项目的启发。

---

## 1. supabase/supabase（~75k star）

### 项目定位 + 商业模式
开源 Firebase 替代品。提供 Postgres + Auth + Storage + Realtime + Edge Functions 一体化后端。商业模式：自托管免费 + 云托管按用量计费。

### 技术栈
- **后端**：Postgres + PostgREST + GoTrue（Auth）+ Realtime（Elixir）+ Storage（Node）
- **前端**：Next.js + Tailwind + Radix UI
- **基础设施**：Docker Compose / Kubernetes，Kong 网关

### 目录结构关键模块
- `apps/studio`：管理控制台
- `apps/docs`：文档站
- `packages/ui`：组件库
- `supabase/migrations`：数据库迁移

### 阅读路径
1. 先跑通 `docker-compose up`
2. 阅读 `apps/studio` 的 Next.js 路由
3. 看 PostgREST 如何把 SQL 变成 REST API
4. 看 Realtime 模块如何用 Postgres 逻辑复制做实时订阅

### 学到的 3 个关键技术点
1. **以 Postgres 为中心的架构**：把数据库当作平台，而非仅仅存储
2. **Row Level Security**：行级权限是多租户最优雅的方案
3. **Edge Functions 模型**：基于 Deno 的边缘运行时

### 对毕业项目的启发
你不需要自己造 Auth/Storage 轮子，直接基于 Supabase 起手能省 80% 工作量。

---

## 2. n8n-io/n8n（~60k star）

### 项目定位 + 商业模式
开源自动化平台（Zapier 替代品）。可视化工作流编辑器 + 400+ 集成节点。商业模式：社区版免费 + 企业版（SSO/审计/多环境）收费。

### 技术栈
- TypeScript 全栈
- Vue 3 + Pinia（前端）
- Express + TypeORM（后端）
- Redis（队列模式）

### 目录结构关键模块
- `packages/cli`：CLI 入口
- `packages/core`：执行引擎
- `packages/editor-ui`：可视化编辑器
- `packages/nodes-base`：所有内置节点

### 阅读路径
1. 跑通 `npm run start`
2. 看一个简单 node（如 HTTP Request）的实现
3. 看 `packages/core` 的工作流执行引擎
4. 看队列模式（Redis）如何分布式执行

### 学到的 3 个关键技术点
1. **DAG 工作流引擎**：节点 + 边 + 触发条件
2. **插件化架构**：怎么让第三方写 node
3. **可视化编辑器**：拖拽 + 连线 + 实时预览

### 对毕业项目的启发
任何「可视化配置 + 后端执行」的产品（低代码、ETL、CI 流水线）都能借鉴 n8n 架构。

---

## 3. outline/outline

### 项目定位 + 商业模式
团队 wiki / 知识库（Notion 替代品）。商业模式：自托管免费 + 云托管订阅。

### 技术栈
- React + MobX（前端）
- Node + Koa + Sequelize（后端）
- Postgres + Redis + S3
- Prosemirror（编辑器内核）

### 目录结构关键模块
- `app/scenes`：页面级组件
- `app/editor`：基于 Prosemirror 的富文本编辑器
- `server/models`：领域模型
- `server/api`：REST API

### 阅读路径
1. 先跑 docker-compose
2. 看 `app/editor` 富文本编辑器（极佳的 Prosemirror 实战）
3. 看 `server/api/documents.ts` 文档 CRUD + 权限
4. 看实时协同模块

### 学到的 3 个关键技术点
1. **基于 Prosemirror 的所见即所得编辑器**
2. **基于 CRDT / OT 的实时协同**
3. **细粒度权限模型**（团队 / 集合 / 文档三级）

### 对毕业项目的启发
任何「内容创作 + 协作」类产品的范本。

---

## 4. calcom/cal.com（~33k star）

### 项目定位 + 商业模式
开源 Calendly 替代品（在线预约调度）。商业模式：自托管 + Cloud 订阅 + 企业 SSO。

### 技术栈
- Next.js 全栈
- tRPC（端到端类型安全）
- Prisma + Postgres
- Tailwind + shadcn/ui

### 目录结构关键模块
- `apps/web`：主应用
- `packages/features`：业务功能模块化
- `packages/app-store`：第三方集成（Zoom / Google Meet 等）

### 阅读路径
1. 看 Next.js App Router 结构
2. 看 tRPC 路由定义（极其优雅）
3. 看 Prisma schema（模型设计教科书）
4. 看 app-store 怎么做插件机制

### 学到的 3 个关键技术点
1. **Next.js + tRPC 全栈最佳实践**
2. **时区 / 日历的工程难题**（你以为简单，其实超难）
3. **Monorepo + Turborepo 组织大型项目**

### 对毕业项目的启发
做毕业项目用 Next.js + tRPC + Prisma，文件夹结构直接抄 cal.com。

---

## 5. PostHog/posthog

### 项目定位 + 商业模式
开源产品分析（Mixpanel / Amplitude 替代品）。商业模式：Cloud 按事件量计费 + 企业 SSO。

### 技术栈
- Django + Celery（后端）
- React + TypeScript（前端）
- ClickHouse（事件存储）
- Kafka + Redis

### 目录结构关键模块
- `posthog/api`：DRF API
- `frontend/src`：React 前端
- `ee`：企业版功能（独立目录）

### 阅读路径
1. 看事件采集 SDK
2. 看 ClickHouse 表结构（事件存储分析）
3. 看 Funnel / Retention 查询实现
4. 看 Feature Flag / AB Test 模块

### 学到的 3 个关键技术点
1. **事件型数据建模**（OLAP vs OLTP）
2. **ClickHouse 在分析场景的应用**
3. **Feature Flag 系统设计**

### 对毕业项目的启发
任何"用户行为分析"产品都可以从 PostHog 借鉴模型。

---

## 6. lobehub/lobe-chat（~50k star，中文社区出品）

### 项目定位 + 商业模式
高颜值多模型 ChatGPT 替代品。商业模式：自托管 + Lobe Cloud 订阅。

### 技术栈
- Next.js 14 App Router
- Zustand + tRPC
- Drizzle ORM
- Vercel AI SDK

### 目录结构关键模块
- `src/app`：Next.js App Router
- `src/features`：业务模块
- `src/services`：与多个 LLM 厂商通信

### 阅读路径
1. 看多模型抽象层（如何统一 OpenAI / Anthropic / 国产模型）
2. 看插件市场架构
3. 看 Agent / Assistant 模式实现
4. 看 PWA + 本地优先设计

### 学到的 3 个关键技术点
1. **多 LLM 厂商抽象**
2. **本地优先（local-first）数据架构**
3. **现代化 Next.js App Router 全栈实践**

### 对毕业项目的启发
做 AI 应用，前端架构直接学 lobe-chat，省 70% 思考。

---

## 7. danny-avila/LibreChat

### 项目定位 + 商业模式
全功能多模型聊天平台（开源 ChatGPT Plus）。完全免费 / 自托管。

### 技术栈
- React + Recoil
- Node + Express + MongoDB
- 支持 OpenAI / Anthropic / Google / 本地模型

### 目录结构关键模块
- `api/server/routes`：聊天路由
- `client/src/components`：UI 组件
- `packages/data-provider`：数据访问层

### 阅读路径
1. 看 SSE 流式响应实现
2. 看上下文窗口管理
3. 看插件 / 工具调用集成
4. 看 RAG 文件上传问答

### 学到的 3 个关键技术点
1. **流式输出（SSE）的工程实现**
2. **多模态消息（文本 / 图片 / 文件）模型**
3. **MongoDB 在聊天场景的使用**

### 对毕业项目的启发
做聊天类 AI 应用，LibreChat 比 lobe-chat 更"工业重"，适合学复杂业务。

---

## 8. All-Hands-AI/OpenHands（~38k star）

### 项目定位 + 商业模式
开源 Devin（AI 自主软件工程师）。完全开源，提供 Cloud 服务。

### 技术栈
- Python（核心 Agent）
- React（前端）
- Docker（沙盒）
- LiteLLM（多模型）

### 目录结构关键模块
- `openhands/agenthub`：Agent 实现集合
- `openhands/runtime`：执行沙盒
- `openhands/llm`：LLM 抽象层
- `frontend`：聊天交互

### 阅读路径
1. 看 Agent 主循环（observation → action）
2. 看 Docker 沙盒安全设计
3. 看工具调用（bash、edit、browser）
4. 看多 Agent 协作模式

### 学到的 3 个关键技术点
1. **完整 Agent 循环架构**
2. **沙盒化执行（容器 + 资源限制）**
3. **多 Agent 协作协议**

### 对毕业项目的启发
做"AI Coding 类"产品的最佳学习对象，2026 年最热门方向之一。

---

## 9. infiniflow/ragflow（~30k star，中文出品）

### 项目定位 + 商业模式
中文 RAG 平台。开源 + 企业版。

### 技术栈
- Python + Flask（后端）
- React（前端）
- Elasticsearch + MinIO
- 自研 DeepDoc 文档解析

### 目录结构关键模块
- `api`：API 层
- `rag`：RAG 核心引擎
- `deepdoc`：文档解析（OCR + 版式分析）
- `agent`：Agent 编排

### 阅读路径
1. 看 DeepDoc 如何处理 PDF / Word（含表格）
2. 看检索流水线（chunking → embedding → rerank）
3. 看 Agent 编排
4. 看 chat API

### 学到的 3 个关键技术点
1. **复杂文档解析**（不只是 PDF 文本提取）
2. **混合检索 + Rerank**
3. **可视化 RAG 流水线编辑**

### 对毕业项目的启发
做企业知识库 / 检索类产品，ragflow 是中文社区最好的范本。

---

## 10. excalidraw/excalidraw

### 项目定位 + 商业模式
开源白板 / 手绘风画图工具。完全免费 + Excalidraw+ 协作版订阅。

### 技术栈
- React + TypeScript
- Canvas（自渲染，不依赖 SVG）
- 端到端加密（E2E）

### 目录结构关键模块
- `packages/excalidraw`：核心库
- `excalidraw-app`：网页应用
- `src/element`：图形元素模型
- `src/renderer`：渲染层

### 阅读路径
1. 看 Element 数据模型
2. 看 Canvas 渲染主循环
3. 看 RoughJS 如何画手绘风格
4. 看协同编辑实现

### 学到的 3 个关键技术点
1. **Canvas 性能优化**（大量元素时不卡）
2. **图形选区 / 变换 / 撤销栈设计**
3. **端到端加密协同**

### 对毕业项目的启发
做任何"可视化编辑器"（流程图、思维导图、电路图）的最佳前端范本。

---

## 5 步法读 SaaS 源码

读真实 SaaS 是工程师成长投资回报最高的活动之一。给你一个可复用的方法论：

### 第 1 步：让它跑起来（半天）
- 严格按 README 走 docker-compose
- 写下踩坑笔记（这些就是你日后的博客素材）
- 截图首页和核心功能，确认你看到的就是它"上线时的样子"

### 第 2 步：从用户路径反推代码（1 天）
- 选一个核心用户流程（如：注册 → 创建第一个项目 → 邀请成员）
- 用浏览器 DevTools 看 Network 请求
- 跟着请求路径找到后端 handler
- 画一张序列图

### 第 3 步：摸清架构骨架（1~2 天）
- 列出顶层目录结构，每个目录写一句话功能
- 找到「领域模型」（通常在 `models` / `entities` / `schema`）
- 画 ER 图，理解核心表关系

### 第 4 步：精读 3 个核心模块（3~5 天）
- 选 3 个你最感兴趣的模块（例如：鉴权、计费、Realtime）
- 每个模块写一篇 2000 字博客
- 把关键代码片段抄到自己笔记本

### 第 5 步：偷一个特性回家（1 周）
- 在自己的项目里复刻一个该 SaaS 的某个特性
- 比如把 cal.com 的可用时段算法搬到自己项目
- 写一篇「我是如何借鉴 X 实现 Y」的博客

走完这 5 步，你对这个 SaaS 的理解就超过 99% 只看过 README 的人。

---

## 心法

> 学软件工程，没有比读优秀的真实代码更高效的事。

教程是别人嚼碎喂给你的。读真实 SaaS 是你自己去"狩猎"知识——过程更慢，但每一口都是真肉。

**毕业项目的天花板，等于你读过的最大 SaaS 的下限。** 现在就挑一个开始读。
