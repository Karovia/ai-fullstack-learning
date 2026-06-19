---
title: awesome 索引导航大全
type: 资源库 / 索引地图
tags: [awesome, 资源导航, 学习索引, github]
难度: 全阶段
更新时间: 2026-06-19
---

# awesome 索引导航大全

## awesome 是什么

awesome 是 GitHub 上一种**社区维护的"精选清单"格式**。最早由 Sindre Sorhus 在 2014 年发起 [`sindresorhus/awesome`](https://github.com/sindresorhus/awesome)，从此每一个领域都长出了自己的 awesome-X 仓库：awesome-python、awesome-react、awesome-llm……

awesome 的本质是**社区评议过的目录**：被收录意味着至少有一群人在用、在维护。它不是搜索引擎能替代的，因为它有"温度"——背后是真实工程师的取舍。

## 怎么用 awesome（先告诉你结论）

awesome 列表很容易让人陷入收集癖。**正确用法只有一句话：带着具体问题去查，查完就关。**

错误用法：周日下午打开一个 awesome-llm，从头滑到尾，star 了 100 个仓库，周一全忘了。

正确用法：今天我要做一个 RAG 项目，需要选一个向量库，去 awesome-rag 里看 3 个对比项，决定用哪个，关掉。

下面把 awesome 列表按主题分类整理。**不需要全 star，看到对应需求时再来查。**

---

## 通用 awesome

- **sindresorhus/awesome**（~370k star）：awesome 之祖，所有方向的入口。
- **awesome-search**：搜索 awesome 列表的元工具。
- **trackawesomelist.com**：跟踪 awesome 列表更新，看新增的项目。

---

## 编程语言

| 列表 | 描述 |
|---|---|
| **vinta/awesome-python** | Python 老牌大全，~220k star |
| **jobbole/awesome-python-cn** | 中文版 awesome-python |
| **sorrycc/awesome-javascript** | 云谦版 JavaScript |
| **avelino/awesome-go** | Go 生态最权威清单 |
| **rust-unofficial/awesome-rust** | Rust 全家桶 |
| **dzharii/awesome-typescript** | TypeScript 资源 |
| **fffaraz/awesome-cpp** | C++ 资源 |
| **akullpp/awesome-java** | Java 库 |

---

## 前端

- **enaqx/awesome-react**：React 周边一站式
- **vuejs/awesome-vue**：Vue 官方维护
- **awesome-css-group/awesome-css**：CSS 进阶
- **brillout/awesome-frontend-libraries**：前端库分类
- **csabapalfi/awesome-web-performance**：Web 性能
- **dypsilon/frontend-dev-bookmarks**：前端开发书签集

前端高频需求："找一个图表库 / 表单库 / 状态管理"——直接去 awesome-react / awesome-vue 对应章节。

---

## 后端

- **mjhea0/awesome-fastapi**：FastAPI 资源
- **sindresorhus/awesome-nodejs**：Node 全家
- **dhamaniasad/awesome-postgres**：Postgres 周边
- **JStumpp/awesome-redis**：Redis 客户端、工具
- **mfornos/awesome-microservices**：微服务架构资源
- **awesome-graphql**：GraphQL 生态
- **awesome-grpc**：gRPC 资源

---

## AI / LLM / Agent（2026 年最热分类）

这是当前更新最快、信号噪声比最高的几个清单：

| 列表 | 重点 |
|---|---|
| **Hannibal046/Awesome-LLM** | LLM 论文 / 框架 / 教程总览 |
| **Shubhamsaboo/awesome-llm-apps** | LLM 应用案例库（带源码） |
| **e2b-dev/awesome-ai-agents** | Agent 框架对比 |
| **punkpeye/awesome-mcp-servers** | MCP 服务器清单（2025 后核心） |
| **steven2358/awesome-generative-ai** | 生成式 AI 总览 |
| **promptslab/Awesome-Prompt-Engineering** | 提示工程 |
| **awesome-rag** | RAG 技术栈 |
| **mlabonne/llm-course** | LLM 工程师课程地图 |
| **kyrolabs/awesome-langchain** | LangChain 生态 |
| **eugeneyan/applied-ml** | 工业级 ML 案例 |

**强烈建议每周扫一眼 mlabonne/llm-course 和 awesome-mcp-servers**，是当前 AI 工程方向迭代最快的两个清单。

---

## DevOps

- **veggiemonk/awesome-docker**：Docker 工具与最佳实践
- **ramitsurana/awesome-kubernetes**：K8s 全家桶
- **awesome-selfhosted/awesome-selfhosted**：可自部署应用清单（喜欢搞 homelab 必看）
- **bregman-arie/devops-exercises**：DevOps 面试题
- **awesome-sysadmin**：系统管理员资源

---

## 学习路径（最值得收藏的几个）

这一组是 GitHub 上最高频被推荐的"学习总入口"：

| 仓库 | 描述 |
|---|---|
| **EbookFoundation/free-programming-books** | 免费编程书全集（多语言） |
| **codecrafters-io/build-your-own-x** | 造轮子大全（前文专题已介绍） |
| **practical-tutorials/project-based-learning** | 项目驱动学习清单 |
| **trekhleb/learn-anything** | 学习任何话题的可视化路径图 |
| **kamranahmedse/developer-roadmap** | 各方向 Roadmap 图 |
| **donnemartin/system-design-primer** | 系统设计入门 |
| **microsoft/ML-For-Beginners** | 微软出品的 ML 入门课 |

---

## 中文资源（强烈推荐）

中文社区的 awesome 列表质量并不输英文，对中文学习者更友好：

- **justjavac/free-programming-books-zh_CN**：免费中文编程书
- **CyC2018/CS-Notes**：科班知识点笔记，国内面试人手一份
- **doocs/advanced-java**：Java 进阶面试合集
- **PKUFlyingPig/cs-self-learning**：CS 自学指南（北大版）
- **datawhalechina** 全套：开源学习社区，AI/ML 教程一流
- **labuladong/fucking-algorithm**：算法刷题
- **ruanyf/weekly**：阮一峰科技周刊（信息源）
- **GitHubDaily/GitHubDaily**：每日 GitHub 推荐

---

## 工具

- **agarrharr/awesome-cli-apps**：命令行神器集合
- **iCHAIT/awesome-macOS**：Mac 软件清单
- **jaywcjlove/awesome-mac**：中文版 awesome-mac
- **stephenh/awesome-developer-tools**：开发工具
- **mtdvio/every-programmer-should-know**：每个程序员都该知道的事

---

## 一些被低估但极有价值的清单

- **danistefanovic/build-your-own-x**（同 codecrafters）
- **ossu/computer-science**：开源 CS 学位（系统化路径）
- **jwasham/coding-interview-university**：编程面试系统课
- **public-apis/public-apis**：免费公共 API 清单（找练手数据源神器）
- **GorvGoyl/Clone-Wars**：知名网站克隆项目集合
- **simonw/datasette**：数据探索神器，但作者维护的 [TIL 站](https://til.simonwillison.net/) 是被低估的资源
- **weibocom/motan**：微博开源项目（中国大厂源码学习）

---

## 如何高效用 awesome 列表（7 条建议）

1. **不要全 star**。star 100 个不会用的项目 = 在记忆里建了 100 个垃圾标签。
2. **带具体问题来查**。"我需要一个 X" 比 "我想看看 X 领域有什么" 高效十倍。
3. **设置过滤维度**：先按 star 数过滤掉 < 1000 的，再按"最近 6 个月有更新"过滤。
4. **看 Contributors 数量**：单人维护的项目慎用，社区项目优先。
5. **关注更新时间**：awesome 列表本身也会过时。看 README 顶部 last update。
6. **用 trackawesomelist.com 订阅**：只看新增条目，避免重复浏览。
7. **维护自己的 awesome-mine.md**：把真正用过、踩过坑的资源整理在自己的笔记里。这才是真正属于你的 awesome。

---

## 心法

awesome 列表是**地图**，不是**目的地**。地图再多，不出发也到不了任何地方。

宁愿读完一本《精通 Postgres》，也不要 star 50 个 awesome-postgres 里的项目。

把 awesome 列表当作"问题驱动的字典"，不要当作"百科全书的休闲读物"。
