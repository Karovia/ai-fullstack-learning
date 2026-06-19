---
title: build-your-own-x 中文跟做指南
type: 资源库 / 跟做手册
tags: [build-your-own-x, 造轮子, 源码级学习, codecrafters]
难度: 进阶
更新时间: 2026-06-19
---

# build-your-own-x 中文跟做指南

## 它是什么

[**codecrafters-io/build-your-own-x**](https://github.com/codecrafters-io/build-your-own-x) 是 GitHub 上最有名的"造轮子合集"之一，目前 star 数约 **370k**。它不是一个代码仓库，而是一份**精心整理的索引**，把"如何从零写一个 X"的教程汇集起来：从 Web 服务器到操作系统，从数据库到神经网络，几乎你能想到的所有计算机系统都有人手把手教你写一个最小可运行版本。

## 为什么是源码级学习的捷径

工业级软件（Postgres、Linux、React）有数百万行代码，普通学习者读三个月也找不到入口。而 build-your-own-x 收录的项目通常有三个特征：

1. **聚焦本质**：把生产系统里的工程包袱（兼容性、性能优化、错误恢复）剥掉，只留核心算法。
2. **配套博客 / 视频**：作者带着你一步步写，每一步都有讲解。
3. **可控范围**：通常 1000~5000 行，一周内可以走完。

读完一个 build-your-own-X，你对 X 的理解会从"使用者"跃迁到"实现者"。这是工程师成长最划算的一种投资。

## 为什么 2026 年比以往任何时候都更应该造轮子

LLM 让"会用"变得太便宜了。你能让 Claude 给你写一个数据库 demo——但你看不懂它在做什么。**护城河从"会用"转移到"懂底层"**。能读懂源码、能造轮子的人，才能驾驭 AI，而不是被 AI 替代。

---

## 按方向分组：可跟做的子项目

下面每个方向都给出推荐路径、难度评估，以及对中文读者最友好的资源。

### 1. 写一个 Web 服务器（推荐入门首选）

- **难度**：⭐⭐ 入门
- **路径**：先写 TCP echo server → 解析 HTTP 报文 → 路由表 → 中间件
- **推荐仓库**：
  - Python 版：`mtrebi/http-server`（含中文注释 fork）
  - Node 版：跟做 [Build your own simple web server with Node.js](https://www.codementor.io/) 系列
  - Go 版：`gorilla/mux` 阅读源码即可
- **中文友好**：「[一个支持 HTTP 1.1 的 Web 服务器](https://github.com/qicosmos/cinatra)」

跟做产出：能解析 GET/POST、支持 keep-alive、能托管静态文件。

### 2. 写一个 Git

- **难度**：⭐⭐⭐ 进阶
- **路径**：blob → tree → commit → ref → packfile
- **推荐**：
  - Wyatt 的 [Build Your Own Git](https://wyag.thb.lt/)（Python，最佳跟做教程）
  - James Coglan《Building Git》（一整本书）
- **中文友好**：[gitlet 中文导读](https://github.com/maryrosecook/gitlet)

跟做产出：自己实现 `init / add / commit / log / branch / merge`。

### 3. 写一个数据库

- **难度**：⭐⭐⭐⭐ 高
- **路径**：B+ Tree → 页式存储 → WAL → MVCC → SQL parser
- **推荐**：
  - **cstack/db_tutorial**（C 语言，跟着写一个小型 SQLite，是最好的入门）
  - **erikgrinaker/toydb**（Rust，分布式 mini DB）
  - 6.824 / 6.5840 MIT 课程项目
- **中文友好**：[《自己动手写数据库》系列博客](https://draveness.me/)（左其盛 / draven 系列文章）

跟做产出：能跑 `INSERT` / `SELECT`，磁盘持久化，支持简单事务。

### 4. 写一个 Shell

- **难度**：⭐⭐ 入门 + 系统调用
- **路径**：fork/exec → 内建命令 → 管道 → job control
- **推荐**：
  - **brenns10/lsh**（C，~250 行，简单优雅）
  - Stephen Brennan 博客文章配套
- **中文友好**：[《自己动手写一个 Shell》](https://github.com/brenns10/lsh) 中文翻译版

### 5. 写一个编程语言

- **难度**：⭐⭐⭐⭐ 高（但收益最大）
- **路径**：lexer → parser → AST → 解释器 → bytecode VM
- **推荐**：
  - **Crafting Interpreters**（Robert Nystrom，免费在线书，神级教程）
  - **Thorsten Ball 的《Writing An Interpreter In Go》和《Writing A Compiler In Go》**
  - **rust-lang/rustc-dev-guide**（如想啃硬骨头）
- **中文友好**：[《手把手教你写编译器》](https://github.com/RUCAIBox/) 系列、《自制编程语言》（前桥和弥）

跟做产出：一个能跑 `fib(20)` 的图灵完备小语言。

### 6. 写一个区块链

- **难度**：⭐⭐⭐ 进阶
- **路径**：Merkle Tree → PoW → P2P → 钱包 → 智能合约
- **推荐**：
  - **Jeiwan/blockchain_go**（Go 版本，含中文社区翻译）
  - **lhartikk/naivechain**（200 行 JS 区块链）
- **中文友好**：[精通区块链开发（中文版）](https://github.com/inoutcode/ethereum_book)

### 7. 写一个 Docker

- **难度**：⭐⭐⭐⭐ 高（涉及 Linux namespace）
- **路径**：namespace → cgroup → rootfs → 镜像层 → 网络
- **推荐**：
  - **p8952/bocker**（100 行 bash 写 Docker，必看）
  - **xianlubird/mydocker**（Go 版，配套书《自己动手写 Docker》中文）
- **中文友好**：陈显鹭《自己动手写 Docker》——中文 build-your-own-X 的典范

### 8. 写一个神经网络

- **难度**：⭐⭐ 入门（数学基础够即可）
- **路径**：感知机 → MLP → 反向传播 → CNN → Transformer
- **推荐**：
  - **karpathy/micrograd** + **nanoGPT**（前文已述）
  - **rasbt/LLMs-from-scratch**
- **中文友好**：李沐《动手学深度学习》（d2l.ai）—— 中文 ML 教学神书

### 9. 写一个 Vue / React

- **难度**：⭐⭐⭐ 进阶
- **路径**：响应式 → VDOM → diff → 调度
- **推荐**：
  - **pomber/didact**（300 行 React）
  - **cuixiaorui/mini-vue**（中文，TDD 风格）
  - **BetaSu/big-react**（卡颂手写 React，逐 commit）

### 10. 写一个搜索引擎

- **难度**：⭐⭐⭐ 进阶
- **路径**：爬虫 → 倒排索引 → TF-IDF → BM25 → 向量检索
- **推荐**：
  - **akrylysov/pogreb**（嵌入式 KV）+ 自己加倒排索引
  - **mayooear/gpt4-pdf-chatbot**（向量检索 RAG 入门）
- **中文友好**：《这就是搜索引擎》（张俊林）配套实现

---

## codecrafters.io 付费课程评价

如果上面跟做过程中发现自己「看懂但写不出来」，可以考虑 [codecrafters.io](https://codecrafters.io/) 的官方付费课程（Build Your Own Redis / Git / SQLite / HTTP Server / Docker / Shell / Kafka 等）。它的优势：

- **测试驱动**：每一步都有自动化测试，做对才能进入下一关。
- **多语言支持**：Python / Go / Rust / TypeScript / Java / C++ 等。
- **GitHub 集成**：直接 push 到 codecrafters 远端跑测试。

价格约 30 美元/月，建议**先免费跟做开源教程，确认自己有兴趣再付费**。

---

## 6 个月跟做计划（每月一个轮子）

| 月份 | 主题 | 目标产出 | 学习重点 |
|---|---|---|---|
| 第 1 月 | Web 服务器（Python） | 支持 HTTP/1.1、静态文件、路由 | 网络编程、协议解析 |
| 第 2 月 | Shell（C） | 内建命令、管道、重定向 | 系统调用、进程模型 |
| 第 3 月 | Git（Python） | init/add/commit/log/branch/merge | 内容寻址、有向图 |
| 第 4 月 | 数据库（C 跟 db_tutorial） | B+ Tree 单表 KV | 存储引擎、索引 |
| 第 5 月 | 解释器（Go 跟 Monkey） | 完整图灵语言 | 编译原理 |
| 第 6 月 | mini React 或 nanoGPT | 看你前后端兴趣 | 框架/AI 底层 |

每个月节奏：第 1 周读资料和原仓库，第 2~3 周边写边读，第 4 周写复盘博客 + 加一个改造特性。

---

## 跟做要避开的坑

1. **不要一开始就追求"完美实现"**：先让 demo 跑起来，再回头优化。
2. **不要照抄**：照抄 = 没学。每写 50 行就关掉原作者代码自己默写。
3. **不要孤军奋战**：加入相关方向的 Discord / 知乎话题，遇到卡点求助。
4. **不要省略写博客**：写出来才是真懂。读者越多，逼迫你想清楚的压力越大。

---

## 心法

> "What I cannot create, I do not understand." — Richard Feynman

build-your-own-x 的精髓不在于你最终造出的轮子有多好——你造的版本一定不如 Postgres 强、不如 React 快。它的精髓在于：当你以后再用 Postgres 和 React 时，你**知道里面在发生什么**。

这种"内部视角"，是 LLM 时代最稀缺的能力。
