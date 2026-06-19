---
title: GitHub 项目精选 - 数据库 & DevOps
chapter: 06
type: github-curation
tags: [github, database, postgresql, redis, vector-db, docker, kubernetes, devops, observability]
created: 2026-06-19
updated: 2026-06-19
---

# 06 章配套 · GitHub 项目精选：数据库 & DevOps

> 配套第 06 章「数据库与工程化」的 GitHub 仓库精选清单。
> 难度图例：🟢 入门 · 🟡 进阶 · 🔴 高阶 · ⚫ 源码级
> 中英标识：🇨🇳 中文友好 · 🌍 英文为主

---

## 🌱 SQL 学习

### gothinkster/realworld 🟢 🌍
- URL：https://github.com/gothinkster/realworld
- Star：约 80k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：每种语言/框架都有 RealWorld 实现（Medium 克隆），对照看后端 SQL schema、ORM、迁移脚本即可。
- 推荐理由：把"读教程"变成"对照真实项目"，直接看 `node-express-realworld-example-app` 或 `django-rest-framework-realworld-example-app` 的 SQL 部分。
- 中文：部分实现含中文 README。

### lerocha/chinook-database 🟢 🌍
- URL：https://github.com/lerocha/chinook-database
- Star：约 2k
- 难度：🟢 入门
- 学习要点：唱片店示例数据库（Artist/Album/Track/Customer/Invoice），8 张主表、约 50 个字段。SQLite/PostgreSQL/MySQL/SQL Server 全脚本都有。
- 推荐理由：练 JOIN、GROUP BY、窗口函数最佳数据集，比那些 employees 数据库更贴近"业务"。

### lukas/learn-sql 🟢 🌍
- URL：https://github.com/lukas/learn-sql
- Star：约 1k
- 难度：🟢 入门
- 学习要点：从 SELECT 到子查询的渐进练习题。
- 推荐理由：刷题型仓库，配合 SQLite 跑即可。

### walmartlabs/lacinia 🟡 🌍
- URL：https://github.com/walmartlabs/lacinia
- Star：约 2k
- 难度：🟡 进阶
- 学习要点：Clojure 的 GraphQL 实现，看它如何把 GraphQL query 翻译成 SQL，是理解"GraphQL ↔ SQL 边界"的好参考。
- 推荐理由：当 ORM 看腻了，看一眼"声明式查询"如何在另一种范式下表达。

---

## 🌿 PostgreSQL 生态

### postgres/postgres ⚫ 🌍
- URL：https://github.com/postgres/postgres
- Star：约 17k（mirror 仓库 star 偏低，实际全球用户量级最高）
- 难度：⚫ 源码级
- 学习要点：`src/backend/optimizer/`（优化器）、`src/backend/executor/`（执行器）、`src/backend/access/heap/`（行存）。
- 推荐理由：所有讲数据库内部的书最终都指向这份代码。第一次读不要全读，挑「一条 SELECT 1 怎么走完」追一遍。

### supabase/supabase 🟡 🇨🇳（中文社区活跃）
- URL：https://github.com/supabase/supabase
- Star：约 78k
- 难度：🟡 进阶
- 学习要点：PostgREST、GoTrue（Auth）、Realtime、Storage 如何拼起来；RLS（Row Level Security）的工程化用法。
- 推荐理由：看完你就懂"BaaS（Backend as a Service）= PG + 几个独立服务"，而不是黑魔法。

### citusdata/citus 🔴 🌍
- URL：https://github.com/citusdata/citus
- Star：约 11k
- 难度：🔴 高阶
- 学习要点：分布式分片扩展，看 `src/backend/distributed/planner/` 学分布式查询规划。
- 推荐理由：理解"PG 怎么从单机变成分布式"的最简路径。

### timescale/timescaledb 🟡 🌍
- URL：https://github.com/timescale/timescaledb
- Star：约 18k
- 难度：🟡 进阶
- 学习要点：Hypertable 抽象、连续聚合（continuous aggregates）。
- 推荐理由：时序场景几乎首选；学会用就免去引入 InfluxDB/Druid 的成本。

### pgvector/pgvector 🟢 🌍
- URL：https://github.com/pgvector/pgvector
- Star：约 14k
- 难度：🟢 入门
- 学习要点：CREATE INDEX ... USING ivfflat / hnsw 两种向量索引；`<->`、`<#>`、`<=>` 三种距离。
- 推荐理由：RAG 时代必备 PG 扩展，源码不到 1 万行 C，可读完。

### postgresml/postgresml 🟡 🌍
- URL：https://github.com/postgresml/postgresml
- Star：约 6k
- 难度：🟡 进阶
- 学习要点：在 PG 内部以扩展形式调用 transformers / xgboost；SQL 即 ML pipeline。
- 推荐理由：开眼界仓库——"数据放在哪儿，计算就放在哪儿"。

### prisma/migrate 🟢 🌍
- URL：https://github.com/prisma/prisma（migrate 在主仓库子目录）
- Star：约 40k
- 难度：🟢 入门
- 学习要点：声明式 schema → 自动生成 SQL 迁移；shadow database 做 diff。
- 推荐理由：现代 Node 后端事实标准。

---

## 🌲 Redis

### redis/redis ⚫ 🌍
- URL：https://github.com/redis/redis
- Star：约 67k
- 难度：⚫ 源码级
- 学习要点：`src/dict.c`（哈希表）、`src/t_string.c`、`src/networking.c`、`src/aof.c`/`src/rdb.c`。
- 推荐理由：教科书级 C 项目，单看 dict.c 就能学到 rehash 双桶策略。

### DragonflyDB/dragonfly 🟡 🌍
- URL：https://github.com/dragonflydb/dragonfly
- Star：约 27k
- 难度：🟡 进阶
- 学习要点：单机多线程 + 共享无锁数据结构；与 Redis 协议兼容但 25× 吞吐。
- 推荐理由：学"现代 C++ + io_uring"如何重写一个老软件。

### redis-stack/redis-stack 🟢 🌍
- URL：https://github.com/redis-stack/redis-stack
- Star：约 1k
- 难度：🟢 入门
- 学习要点：把 Redis + RediSearch + RedisJSON + RedisTimeSeries 一键打包。
- 推荐理由：一行 Docker 试遍 Redis 全家桶。

### luin/ioredis 🟢 🌍
- URL：https://github.com/redis/ioredis
- Star：约 14k
- 难度：🟢 入门
- 学习要点：Cluster/Sentinel/Pipeline/Lua 客户端封装。
- 推荐理由：Node 生态最好的 Redis 客户端，源码 ~1 万行可通读。

---

## 🎯 ORM / 数据迁移

### sqlalchemy/sqlalchemy 🟡 🌍
- URL：https://github.com/sqlalchemy/sqlalchemy
- Star：约 10k
- 难度：🟡 进阶
- 学习要点：Core（表达式语言）vs ORM 双层；`session.flush()` 与 `unit of work` 模式。
- 推荐理由：Python 后端绕不开，学会就能从"会用"到"懂为什么"。

### MagicStack/asyncpg 🟡 🌍
- URL：https://github.com/MagicStack/asyncpg
- Star：约 7k
- 难度：🟡 进阶
- 学习要点：自己写 PG wire protocol，绕过 libpq；这就是它比 psycopg2 快的原因。
- 推荐理由：想看 Python 异步 IO 实战的标准答案。

### alembic/alembic 🟢 🌍
- URL：https://github.com/sqlalchemy/alembic
- Star：约 3k
- 难度：🟢 入门
- 学习要点：autogenerate vs 手写迁移；分支合并。
- 推荐理由：SQLAlchemy 官配迁移工具。

### prisma/prisma 🟢 🌍
- URL：https://github.com/prisma/prisma
- Star：约 40k
- 难度：🟢 入门
- 学习要点：schema.prisma DSL；Rust 写的查询引擎；类型安全 client。
- 推荐理由：让前端工程师也能舒服写后端。

### drizzle-team/drizzle-orm 🟢 🌍
- URL：https://github.com/drizzle-team/drizzle-orm
- Star：约 25k
- 难度：🟢 入门
- 学习要点：TypeScript-first、零运行时开销、SQL-like API。
- 推荐理由：Edge 场景（Cloudflare Workers）首选。

### typeorm/typeorm 🟢 🌍
- URL：https://github.com/typeorm/typeorm
- Star：约 35k
- 难度：🟢 入门
- 学习要点：装饰器风格 entity；ActiveRecord vs DataMapper 双模式。
- 推荐理由：NestJS 生态的事实标配 ORM。

---

## 🔍 向量数据库 / 搜索

### qdrant/qdrant 🟡 🌍
- URL：https://github.com/qdrant/qdrant
- Star：约 23k
- 难度：🟡 进阶
- 学习要点：Rust 写 HNSW + 过滤；payload-aware 索引。
- 推荐理由：性能/易用性平衡最佳的开源向量库。

### lancedb/lancedb 🟡 🌍
- URL：https://github.com/lancedb/lancedb
- Star：约 6k
- 难度：🟡 进阶
- 学习要点：列存格式 Lance（Arrow 兼容）、零拷贝读、版本化。
- 推荐理由：Embed in Python/JS，不需要服务进程。

### chroma-core/chroma 🟢 🌍
- URL：https://github.com/chroma-core/chroma
- Star：约 18k
- 难度：🟢 入门
- 学习要点：极简 API，3 行代码起步。
- 推荐理由：原型阶段最快上手的向量库。

### weaviate/weaviate 🟡 🌍
- URL：https://github.com/weaviate/weaviate
- Star：约 12k
- 难度：🟡 进阶
- 学习要点：内建 modules（OpenAI、HF）做 embed；GraphQL 查询。
- 推荐理由：开箱即用的"语义 + 关键词"混合搜索。

### milvus-io/milvus 🔴 🇨🇳
- URL：https://github.com/milvus-io/milvus
- Star：约 32k
- 难度：🔴 高阶
- 学习要点：分布式架构，coord/proxy/data/index/query node 解耦。
- 推荐理由：中文社区最活跃的向量库；Zilliz 团队源自 Oracle/SAP 数据库背景。

### typesense/typesense 🟡 🌍
- URL：https://github.com/typesense/typesense
- Star：约 23k
- 难度：🟡 进阶
- 学习要点：C++ 单二进制；自动同义词；typo tolerance 默认开。
- 推荐理由：Algolia 的开源平替。

### meilisearch/meilisearch 🟢 🌍
- URL：https://github.com/meilisearch/meilisearch
- Star：约 47k
- 难度：🟢 入门
- 学习要点：Rust 写、prefix-search 极快、文档导向。
- 推荐理由：5 分钟跑起来的全文搜索。

### elastic/elasticsearch 🔴 🌍
- URL：https://github.com/elastic/elasticsearch
- Star：约 70k
- 难度：🔴 高阶
- 学习要点：Lucene 之上的分布式搜索；mapping、analyzer、shard。
- 推荐理由：仍然是行业事实标准；学了好换饭。

---

## 🐳 Docker / K8s

### docker/docker（moby/moby）⚫ 🌍
- URL：https://github.com/moby/moby
- Star：约 70k
- 难度：⚫ 源码级
- 学习要点：containerd 接口、libnetwork、buildkit 解耦后保留的核心 daemon。
- 推荐理由：所有容器知识的源头。

### kubernetes/kubernetes ⚫ 🌍
- URL：https://github.com/kubernetes/kubernetes
- Star：约 113k
- 难度：⚫ 源码级
- 学习要点：先读 `pkg/scheduler/` 与 `pkg/controller/`；从一个 Pod 创建追到调度落地。
- 推荐理由：云原生时代最大教科书。

### helm/helm 🟡 🌍
- URL：https://github.com/helm/helm
- Star：约 28k
- 难度：🟡 进阶
- 学习要点：Chart 模板渲染、release lifecycle、依赖解析。
- 推荐理由：K8s 包管理事实标准。

### kubernetes-sigs/kind 🟢 🌍
- URL：https://github.com/kubernetes-sigs/kind
- Star：约 14k
- 难度：🟢 入门
- 学习要点：在 Docker 里跑 K8s 节点，CI 友好。
- 推荐理由：本地学 K8s 最快的工具。

### kubernetes/minikube 🟢 🌍
- URL：https://github.com/kubernetes/minikube
- Star：约 30k
- 难度：🟢 入门
- 学习要点：单节点本地 K8s，自带 dashboard、addon。
- 推荐理由：和 kind 二选一即可。

### k3s-io/k3s 🟡 🌍
- URL：https://github.com/k3s-io/k3s
- Star：约 30k
- 难度：🟡 进阶
- 学习要点：单二进制 K8s（<100MB），SQLite 替代 etcd 模式。
- 推荐理由：边缘/IoT/家庭实验室必备。

### rancher/rancher 🟡 🌍
- URL：https://github.com/rancher/rancher
- Star：约 24k
- 难度：🟡 进阶
- 学习要点：多集群管理 UI；项目/命名空间隔离模型。
- 推荐理由："K8s 上的 cPanel"，给非平台工程师友好。

---

## 📊 监控 / 可观测

### prometheus/prometheus 🟡 🌍
- URL：https://github.com/prometheus/prometheus
- Star：约 57k
- 难度：🟡 进阶
- 学习要点：TSDB 存储引擎（block + WAL）、PromQL、scrape 模型。
- 推荐理由：可观测三件套之首。

### grafana/grafana 🟡 🌍
- URL：https://github.com/grafana/grafana
- Star：约 63k
- 难度：🟡 进阶
- 学习要点：dashboard JSON 协议、datasource plugin 系统。
- 推荐理由：和 Prometheus 黄金搭档。

### grafana/loki 🟡 🌍
- URL：https://github.com/grafana/loki
- Star：约 24k
- 难度：🟡 进阶
- 学习要点："只索引 label 不索引内容"思路；LogQL 与 PromQL 同形。
- 推荐理由：低成本日志方案首选。

### grafana/tempo 🟡 🌍
- URL：https://github.com/grafana/tempo
- Star：约 4k
- 难度：🟡 进阶
- 学习要点：trace 存储，对象存储优化。
- 推荐理由：补齐"指标-日志-追踪"三角形。

### getsentry/sentry 🟡 🇨🇳（中文社区活跃）
- URL：https://github.com/getsentry/sentry
- Star：约 39k
- 难度：🟡 进阶
- 学习要点：错误聚合（grouping algorithm）、release tracking、performance。
- 推荐理由：面向"开发者错误"，与 Prometheus 互补。

### open-telemetry/opentelemetry-python 🟡 🌍
- URL：https://github.com/open-telemetry/opentelemetry-python
- Star：约 2k
- 难度：🟡 进阶
- 学习要点：Tracer/Meter/Logger SDK；自动 instrumentation。
- 推荐理由：未来三五年的可观测协议标准。

### louislam/uptime-kuma 🟢 🇨🇳
- URL：https://github.com/louislam/uptime-kuma
- Star：约 58k
- 难度：🟢 入门
- 学习要点：Vue + Node + SQLite 的简单架构；多协议探测。
- 推荐理由：自托管的"UptimeRobot"，5 分钟跑起来。

### netdata/netdata 🟢 🌍
- URL：https://github.com/netdata/netdata
- Star：约 72k
- 难度：🟢 入门
- 学习要点：1 秒粒度的实时仪表盘；C 写的 agent。
- 推荐理由：装一台机器，瞬间出几百张图。

---

## ⚙️ CI/CD / Infra

### sdras/awesome-actions 🟢 🌍
- URL：https://github.com/sdras/awesome-actions
- Star：约 25k
- 难度：🟢 入门
- 学习要点：GitHub Actions 主题市场目录。
- 推荐理由：写 workflow 前先在这里搜一遍。

### kubernetes/dashboard 🟢 🌍
- URL：https://github.com/kubernetes/dashboard
- Star：约 14k
- 难度：🟢 入门
- 学习要点：官方 Web UI；RBAC token 登录。
- 推荐理由：肉眼看 K8s 状态最直接的工具。

### argoproj/argo-cd 🔴 🌍
- URL：https://github.com/argoproj/argo-cd
- Star：约 18k
- 难度：🔴 高阶
- 学习要点：GitOps 模型；Application CRD；自动 sync。
- 推荐理由：现代 CD 标杆。

### hashicorp/terraform 🟡 🌍
- URL：https://github.com/hashicorp/terraform
- Star：约 43k
- 难度：🟡 进阶
- 学习要点：HCL 语法；plan/apply 二阶段；provider 插件。
- 推荐理由：IaC 事实标准，云资源声明式管理。

---

## 🏆 Easy-vibe / 全套生产级

### supabase/supabase 🌟 🇨🇳
- URL：https://github.com/supabase/supabase
- Star：约 78k
- 难度：🟢 入门
- 学习要点：一套 CLI 起本地全栈：PG + Auth + Storage + Realtime + Edge Functions。
- 推荐理由：周末就能搭出一个生产级后端。

### pocketbase/pocketbase 🌟 🌍
- URL：https://github.com/pocketbase/pocketbase
- Star：约 47k
- 难度：🟢 入门
- 学习要点：单 Go 二进制 + SQLite + 内嵌管理 UI。
- 推荐理由："2020s 版的 Firebase 替代"，部署一条命令。

### directus/directus 🌟 🌍
- URL：https://github.com/directus/directus
- Star：约 29k
- 难度：🟡 进阶
- 学习要点：把任何 SQL 数据库瞬间包装成 REST + GraphQL + 后台管理。
- 推荐理由：内容驱动型项目（CMS-like）首选。

### appwrite/appwrite 🌟 🌍
- URL：https://github.com/appwrite/appwrite
- Star：约 47k
- 难度：🟡 进阶
- 学习要点：BaaS 方案，多语言 SDK 全。
- 推荐理由：Supabase 的 PHP/MariaDB 替代。

---

## 🚀 周末挑战：自建一个 mini-Supabase

> 把上面的开源积木拼一个"75 分版 Supabase"——这是这一章最值得花一个周末的练习。

**目标架构：**

```
┌─────────────┐
│  Next.js    │  ← prisma 类型安全访问
└──────┬──────┘
       │ HTTP
┌──────▼──────┐    ┌──────────────┐
│  PostgREST  │ ←  │ Postgres 16  │ ← pgvector 扩展
└─────────────┘    │  + RLS       │
                   └──────────────┘
       ▲                  ▲
       │                  │
┌──────┴──────┐    ┌──────┴──────┐
│  GoTrue     │    │ MinIO (S3)  │
│  (Auth)     │    │  Storage    │
└─────────────┘    └─────────────┘

       ▲
       │  全部用 docker-compose 起
       │
┌──────┴──────┐
│ Prometheus  │ + Grafana + Loki  ← 监控
└─────────────┘
```

**两天计划：**

- **D1 上午**：docker-compose 起 PG 16 + pgvector + PostgREST，跑通 `GET /todo` 自动生成的 REST。
- **D1 下午**：加 GoTrue，在 PG 里开 RLS，验证未登录用户读不到他人 todo。
- **D2 上午**：MinIO 当 S3，写一个上传图片 → 存 pgvector embedding 的小 demo（顺带练 ANN 搜索）。
- **D2 下午**：Grafana + Prometheus 监控容器；Loki 收日志。截图发朋友圈收赞。

完成它，你就把这一章 80% 的概念都摸过一遍——比读 10 篇博客都强。

---

## 📌 学习路径推荐

| 周 | 主题 | 推荐仓库 |
|----|------|----------|
| W1 | SQL 基础 | chinook-database + lukas/learn-sql |
| W2 | PG 实战 | supabase/supabase + pgvector |
| W3 | ORM | prisma 或 sqlalchemy（按语言挑）|
| W4 | Redis | redis/redis 文档 + ioredis 上手 |
| W5 | 向量库 | qdrant 或 chroma 二选一 |
| W6 | Docker | moby + 写 5 个 Dockerfile |
| W7 | K8s | kind + 跟着官方 tutorial |
| W8 | 可观测 | Prometheus + Grafana + Loki 全套 |
| W9 | mini-Supabase 挑战 | 见上方 |
| W10 | 选定一项深入 | 任选一个 ⚫ 源码级仓库通读 |

---

> 本文配套：第 06 章「数据库与工程化」 · 后续章节：07 LLM、08 Agent。
