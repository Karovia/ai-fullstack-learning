---
title: MVP 开发流程
chapter: 09-综合实战-毕业项目
week: 3-8
stage: 毕业项目
duration: 4-6 周
prerequisites:
  - 完成 [[02-架构设计与技术选型]]，已有 RFC
outputs:
  - 可上线的 MVP（功能完整、有部署、有监控）
  - 6 周日报 / 周报
  - GitHub 仓库（commits + PR + Issues）
tags:
  - MVP
  - 项目管理
  - 工程实践
updated: 2026-06-19
---

# MVP 开发流程

> "MVP 不是一个不完整的产品，而是一个能跑通价值循环的最小完整产品。"

本章是动手最多的一章。如果前两章是设计，这一章就是 6 周里每天的"作战手册"。读完后你会有一份周计划 + 每日打卡模板，照着做就能避开 80% 的翻车点。

## 一、开发节奏：4-6 周 MVP

### 核心原则

- **每周必须有可见进展**：周末把当周成果发给 3 个朋友看，不能解释超过 1 句话。
- **每周一个 demo-able milestone**：能演示，不一定能上线。
- **超过 6 周还没 MVP，必须砍范围**，不是延期。

### 节奏配比（每周 5 个工作日）

- 周一：规划 + 高难度 deep work
- 周二/三/四：执行
- 周五：联调 + 部署 + 写日志
- 周末：1 天放松、1 天看反馈

## 二、项目脚手架

### Monorepo vs 多仓

| | Monorepo | 多仓 |
|---|---------|------|
| 共享类型 | 容易 | 要发 npm 包 |
| CI 时间 | 长 | 短 |
| 心智负担 | 低 | 高 |
| 适合 MVP | ✅ | ❌ |

**推荐 MVP**：Monorepo（pnpm workspace 或 Turborepo）。

### 推荐目录结构

```
my-project/
├── apps/
│   ├── web/              # Next.js 前端
│   └── api/              # FastAPI 后端
├── packages/
│   ├── shared/           # 共享类型 (TS)
│   └── ui/               # 共享组件 (可选)
├── infra/
│   ├── docker/
│   └── fly.toml
├── docs/
│   ├── PRD.md
│   ├── RFC.md
│   └── runbook.md
├── .github/
│   └── workflows/        # CI
├── docker-compose.yml    # 本地开发
├── package.json
├── pnpm-workspace.yaml
└── README.md
```

Python 后端独立用 `uv` 管理，不进 pnpm workspace。

## 三、开发优先级排序

### 三层金字塔

```
        ╱╲           v1.1+ 增强
       ╱  ╲          
      ╱────╲         核心增值（差异化）
     ╱      ╲        
    ╱────────╲       基础（鉴权、支付、监控）
```

**先做基础（撑得住），再做核心（留得下用户），最后增强（让用户爱上）。**

很多人反过来——先做酷炫 AI，最后才发现没鉴权，这时候已经晚了。

## 四、Week-by-Week 详细计划

以 AskMyDocs 类项目为例。

### Week 1: 脚手架 + CI/CD + 鉴权

**目标**：能注册、能登录、有部署，看起来"像一个产品"。

- [ ] 初始化 Monorepo（pnpm + Turbo）
- [ ] 前端 Next.js + Tailwind + shadcn 初始化
- [ ] 后端 FastAPI + uv 初始化
- [ ] docker-compose 起 Postgres + Redis 本地
- [ ] Alembic 第一个迁移：users 表
- [ ] next-auth 邮箱魔法链接
- [ ] 部署：Vercel 前端 + Fly.io 后端 + Supabase DB
- [ ] GitHub Actions：lint + type-check + test
- [ ] Sentry 接入
- [ ] 域名 + HTTPS

**周末 demo**：朋友能注册登录，看到一个空页面。

### Week 2: 后端核心 API + DB

**目标**：核心数据模型 + CRUD 跑通。

- [ ] documents / chunks / messages / conversations 表
- [ ] 文件上传 → R2/S3
- [ ] PDF 解析（pypdf / unstructured）
- [ ] chunk 切分策略（500 字 / 50 重叠）
- [ ] Embedding 接入（Voyage）+ pgvector 入库
- [ ] OpenAPI doc 生成
- [ ] 单元测试覆盖核心 service

**周末 demo**：用 curl 上传 PDF，看到 chunks 入库。

### Week 3: 前端核心页面

**目标**：UI 跑通主路径。

- [ ] 上传页面（拖拽 + 进度条）
- [ ] 文档列表
- [ ] 对话页面骨架
- [ ] 用 mock 数据先跑通流程
- [ ] TanStack Query 接口对接
- [ ] 错误状态 / loading / 空状态

**周末 demo**：能上传、能看到列表，对话页是假的。

### Week 4: AI 接入 + RAG

**目标**：核心价值闭环。

- [ ] Claude Sonnet SSE 流式
- [ ] RAG pipeline：embed → 检索 → rerank → 组装 prompt
- [ ] 引用原文渲染（点击跳到 PDF 那段）
- [ ] Langfuse 追踪
- [ ] 限流（按用户 / 按全局）
- [ ] 成本日志写入

**周末 demo**：真的能问答了，有引用、有流式。

### Week 5: 联调 + Bug

**目标**：所有 P0/P1 bug 清零。

- [ ] 端到端流程跑 50 遍
- [ ] 找 5 个真实用户内测，收集 bug
- [ ] 边界 case：超长 PDF / 中英混排 / 扫描件
- [ ] 性能：首字延迟 / 并发
- [ ] 移动端适配
- [ ] 文案打磨（不要"系统繁忙"，要具体）

**周末 demo**：朋友用着不卡、不出错。

### Week 6: 内测 + 优化

**目标**：准备发布。

- [ ] 邀请 30 个种子用户
- [ ] 关键页面埋点（PostHog）
- [ ] 漏斗分析：注册→上传→提问→留存
- [ ] SEO 基础（landing page、meta）
- [ ] 隐私政策 + ToS
- [ ] 备份策略验证（真删一次再恢复）
- [ ] 写 launch checklist
- [ ] 准备 ProductHunt 素材

**周末 demo**：可以发布了。

## 五、每日开发流程

### 标准一天（可独立可团队）

```
09:00  晨会（自言自语版）
       - 昨天做了什么？
       - 今天目标 1-3 件
       - 有什么阻塞？
       
09:30  Deep Work 1（90 min）
11:00  休息 + 邮件
11:15  Deep Work 2（45 min）
12:00  午饭

14:00  Deep Work 3（90 min）
15:30  Code Review 自己 PR
16:00  联调 / 测试
17:00  写日报 + commit + push
       - 今日进度截图
       - 遇到什么问题
       - 明日 top 3
17:30  下班
```

### 日报模板

```markdown
## 2026-06-19 (W3 D2)
### 完成
- [x] 实现 PDF 解析 service
- [x] 写了 unit test，覆盖率 78%
### 未完成
- [ ] chunk 切分（明天继续）
### 阻塞
- 中文 PDF 编码问题，明天问 GPT
### 学到
- pypdf 对扫描件无效，要 fallback 到 OCR
```

## 六、GitHub Issues + Projects 管理

### Issue 模板

```markdown
**作为** [角色]
**我想** [目标]
**这样** [价值]

### 验收
- [ ] 条件 1
- [ ] 条件 2

### 技术备注
- 涉及表：X、Y
```

### Projects 看板列

`Backlog → This Week → In Progress → Review → Done`

每周一从 Backlog 拉 5-7 个进 This Week，超出就要砍。

## 七、Git 分支策略

**MVP 阶段就一条主线**：

```
main          ─●─●─●─●─ (永远可部署)
feature/xxx       ●─●─┘
```

- main 直接锁保护：必须 PR + CI 通过
- feature 分支生命周期 ≤ 3 天，长了说明拆得不够细
- 每个 PR ≤ 400 行（含测试），更长就再拆

## 八、代码 Review 自我清单

提交前自查：

- [ ] 删掉了所有 console.log / print
- [ ] 没有写死的 API key
- [ ] 边界：null / 空数组 / 超长输入
- [ ] 错误处理：每个 await 都包了 try-catch 或 .catch
- [ ] 性能：N+1 查询？
- [ ] 测试：新增逻辑有对应测试
- [ ] 文档：API 改了 → OpenAPI 更新；表改了 → ERD 更新

## 九、文档同步

每个 PR 要同步：
- API 改 → OpenAPI 自动生成 + 提交
- README：环境变量列表
- CHANGELOG：用户能感知的变化
- runbook.md：上线/回滚步骤

## 十、联调技巧

### 前后端契约先行

1. 后端先发 OpenAPI（哪怕 mock）
2. 前端用 `openapi-typescript` 生成 TS 类型
3. 前端基于 mock 写 UI
4. 后端实现，前端无缝切换

### Mock 工具
- MSW（前端拦截 fetch）
- json-server（5 分钟起一个 REST mock）

### 联调日志清单
- 前端：Network tab 截图 + payload
- 后端：request_id 全链路
- 出错时拼 `request_id` 定位

## 十一、何时重构？

> **MVP 期间禁止边写边重构。** 写丑没关系，跑通才是命。

- 每周五留 2 小时"小重构"：删死代码、统一命名。
- MVP 完成后留 1 周"大重构"：抽象、性能。
- 重构必须有 PR + 必须有测试。

## 十二、6-8 周 MVP 失败常见原因

| 失败原因 | 占比 | 防御 |
|----------|------|------|
| 功能蔓延 | 40% | 砍范围红线 |
| 技术债爆雷 | 20% | 每周小重构 |
| 卡 bug 出不来 | 15% | 卡超过 2 小时就求助 |
| 状态崩溃 | 10% | 每周休 1.5 天 |
| 选型错误 | 10% | RFC 阶段 review |
| 联调地狱 | 5% | 契约先行 |

## 十三、本章实战清单

- [ ] 按目录结构初始化项目（commit 1）
- [ ] CI 跑通绿
- [ ] 写完 W1-W6 周计划，钉到墙上
- [ ] 每日填日报模板（哪怕 5 行）
- [ ] 每周五给 3 位朋友发 demo（不超过 1 句话解释）
- [ ] 每个 PR ≤ 400 行
- [ ] 第 6 周末完成可发布版本

## 十四、案例：学员小 L 的 6 周

W1-2 节奏正常。W3 卡在"AI 引用渲染"3 天 → 改方案：先返回纯文本，引用做成 v1.1。W4 准时进 RAG。W5 测试发现中文 embedding 召回差 → 换 Voyage 中文模型。W6 内测拿到 12 个种子用户、留存 50%。

关键决策：W3 卡住时果断砍功能，没有硬刚。

---
**上一篇 ←** [[02-架构设计与技术选型]]
**下一篇 →** [[04-发布与冷启动]]
