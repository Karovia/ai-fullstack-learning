---
title: GitHub 项目精选 - AI Agent 全景
chapter: 08
type: github-curation
tags: [github, ai-agent, mcp, claude, langchain, langgraph, autogen, browser-use, computer-use, coding-agent]
created: 2026-06-19
updated: 2026-06-19
---

# 08 章配套 · GitHub 项目精选：AI Agent 全景

> 配套第 08 章「AI Agent - 框架与源码」的 GitHub 仓库精选清单。这是整个学习路线的核心一章，所以这份清单也写得最厚。
> 难度图例：🟢 入门 · 🟡 进阶 · 🔴 高阶 · ⚫ 源码级
> 中英标识：🇨🇳 中文友好 · 🌍 英文为主
> ⭐ = 强烈推荐 / 必读

---

## 🌟 AI Agent 必读核心仓库

> 这是开局阶段的"七件套"——Anthropic + OpenAI 双家官方资料 + Claude Code 本身。把这部分嚼透，比看任何 YouTube 视频都顶用。

### anthropics/courses ⭐ 必读
- URL：https://github.com/anthropics/courses
- Star：约 14k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：5 门官方课，重点看 `tool_use/` 和 `prompt_evaluations/` 两门。
- 推荐理由：Anthropic 官方写的 Agent 启蒙；每节都是 Notebook，能跑能改。
- 中文：🌍 英文，但代码即语言。

### anthropics/anthropic-cookbook ⭐ 必读核心
- URL：https://github.com/anthropics/anthropic-cookbook
- Star：约 14k
- 难度：🟡 进阶
- 学习要点：**`patterns/agents/` 子目录是这本书的"圣经"**——5 个 Agent 模式（Prompt Chaining / Routing / Parallelization / Orchestrator-Workers / Evaluator-Optimizer），每个 200 行 Python，没有任何框架依赖，纯 Anthropic SDK。
- 推荐理由：市面上唯一一份"用 200 行讲清楚 Agent 是什么"的官方代码。读完它，你看 LangGraph / AutoGen / CrewAI 都会觉得"哦不过如此"。
- 中文：🌍 英文 README，代码极简。

### anthropics/anthropic-quickstarts ⭐
- URL：https://github.com/anthropics/anthropic-quickstarts
- Star：约 9k
- 难度：🟡 进阶
- 学习要点：含 `computer-use-demo/`（Claude 操作真实电脑）、`agents/` 模板、`customer-support-agent/`。
- 推荐理由：computer-use-demo 是了解"Claude 怎么看屏幕、怎么点鼠标"的官方样本。

### anthropics/claude-agent-sdk-python ⭐
- URL：https://github.com/anthropics/claude-agent-sdk-python
- Star：约 4k（新出）
- 难度：🟡 进阶
- 学习要点：官方 Agent SDK，封装了 tool use / system prompt / hooks / MCP 客户端。
- 推荐理由：写 Claude Agent 的"标准 SDK"，08 章后期实战必用。

### anthropics/claude-code ⭐
- URL：https://github.com/anthropics/claude-code
- Star：约 25k
- 难度：🟡 进阶 → ⚫ 源码级
- 学习要点：Claude Code CLI 本身。看 hooks 配置、settings.json schema、subagent 启动逻辑。
- 推荐理由：你正在用的工具就是最好的教材。

### openai/openai-agents-python ⭐
- URL：https://github.com/openai/openai-agents-python
- Star：约 8k
- 难度：🟡 进阶
- 学习要点：OpenAI 官方 Agent SDK；handoff、guardrail、tracing。
- 推荐理由：和 Anthropic 的 SDK 对照看，能洞察"两家对 Agent 的理解差异"。

### anthropics/build-with-claude-cookbook
- URL：https://github.com/anthropics/build-with-claude-cookbook
- Star：约 2k
- 难度：🟡 进阶
- 学习要点：偏"产品"的 Anthropic 实战集合，含 chat UI、文档处理 demo。
- 推荐理由：补充 anthropic-cookbook 之外的工程化场景。

---

## 🛠️ Agent 框架（百花齐放，挑 1-2 家用熟即可）

### langchain-ai/langgraph ⭐ 事实标准
- URL：https://github.com/langchain-ai/langgraph
- Star：约 10k
- 难度：🟡 进阶
- 学习要点：用图（StateGraph）描述 Agent；checkpointer 持久化；human-in-the-loop。
- 推荐理由：复杂多 Agent 编排目前最稳的选择，被 LangSmith / LangGraph Cloud 系列绑定。
- 中文：🌍 英文。

### langchain-ai/langchain
- URL：https://github.com/langchain-ai/langchain
- Star：约 95k
- 难度：🟡 进阶
- 学习要点：document loader、retriever、agent_executor、LCEL（LangChain Expression Language）。
- 推荐理由：争议大但仍是 RAG / Agent 工程化的事实起点。

### microsoft/autogen
- URL：https://github.com/microsoft/autogen
- Star：约 36k
- 难度：🟡 进阶
- 学习要点：v0.4 重写后基于 actor 模型；GroupChat、Studio GUI。
- 推荐理由：多 Agent 对话研究领域引用最多的框架。

### crewAIInc/crewAI
- URL：https://github.com/crewAIInc/crewAI
- Star：约 28k
- 难度：🟢 入门
- 学习要点：role / goal / task 三抽象；YAML 即编排。
- 推荐理由：最容易"5 分钟跑起来一个 Agent 团队"的框架。

### TransformerOptimus/SuperAGI
- URL：https://github.com/TransformerOptimus/SuperAGI
- Star：约 16k
- 难度：🟡 进阶
- 学习要点：含 Web GUI；Marketplace 风格的 toolkit。
- 推荐理由：早期"AutoGPT 工程化"代表，看历史价值。

### embedchain/mem0（原 embedchain）
- URL：https://github.com/mem0ai/mem0
- Star：约 23k
- 难度：🟢 入门
- 学习要点：长期记忆抽象，自动 fact extraction + vector + graph 三层。
- 推荐理由：让 Agent 有"持久记忆"的最易用方案。

### openai/swarm
- URL：https://github.com/openai/swarm
- Star：约 18k
- 难度：🟢 入门
- 学习要点：~600 行 Python；纯函数 + handoff，没有任何状态机。
- 推荐理由：OpenAI 出的"教学用"轻量框架；后被 openai-agents-python 替代但仍值得一读。

### livekit/agents
- URL：https://github.com/livekit/agents
- Star：约 5k
- 难度：🟡 进阶
- 学习要点：实时语音 Agent；STT-LLM-TTS pipeline；VAD、打断处理。
- 推荐理由：做语音助手的几乎只能选它。

### pydantic/pydantic-ai
- URL：https://github.com/pydantic/pydantic-ai
- Star：约 9k
- 难度：🟡 进阶
- 学习要点：强类型 Agent；Pydantic schema 即 tool；与 FastAPI 风格统一。
- 推荐理由：Python 后端工程师写 Agent 的最佳路径。

### simular-ai/agent-s
- URL：https://github.com/simular-ai/agent-s
- Star：约 4k
- 难度：🔴 高阶
- 学习要点：截图 + 视觉模型操作 GUI；hierarchical planning。
- 推荐理由：开源 computer-use 复现，对照 Anthropic 官方 demo 更深入。

### letta-ai/letta（原 MemGPT）
- URL：https://github.com/letta-ai/letta
- Star：约 16k
- 难度：🟡 进阶
- 学习要点：分层记忆（核心记忆 + 召回记忆 + 文档记忆）；论文同名实现。
- 推荐理由：长期记忆 Agent 必学，论文 MemGPT 同名实现。

### stanfordnlp/dspy
- URL：https://github.com/stanfordnlp/dspy
- Star：约 22k
- 难度：🔴 高阶
- 学习要点：用"编译"思路自动优化 prompt；signature、predictor、teleprompter 三抽象。
- 推荐理由：颠覆"手写 prompt"范式；Stanford 出品论文同步代码。

---

## 🔌 MCP（Model Context Protocol）

> MCP 是 2024 末 Anthropic 推出的"Agent 的 USB-C"——统一了 Agent 调用外部工具的协议。08 章重头戏之一。

### modelcontextprotocol/servers ⭐
- URL：https://github.com/modelcontextprotocol/servers
- Star：约 22k
- 难度：🟡 进阶
- 学习要点：官方参考实现：filesystem、git、postgres、puppeteer、slack 等十几个 server。
- 推荐理由：写自己 MCP server 之前，先把这里挑 3 个看完。

### modelcontextprotocol/python-sdk
- URL：https://github.com/modelcontextprotocol/python-sdk
- Star：约 9k
- 难度：🟢 入门
- 学习要点：`@mcp.tool` 装饰器；FastMCP 高阶 API；stdio / SSE 两种 transport。
- 推荐理由：Python 写 MCP server 一行起步。

### modelcontextprotocol/typescript-sdk
- URL：https://github.com/modelcontextprotocol/typescript-sdk
- Star：约 6k
- 难度：🟢 入门
- 学习要点：Node 写 MCP；和 Claude Desktop 集成最直接。
- 推荐理由：JS 生态首选。

### modelcontextprotocol/inspector ⭐
- URL：https://github.com/modelcontextprotocol/inspector
- Star：约 4k
- 难度：🟢 入门
- 学习要点：MCP server 调试工具，带 GUI；列出 tool / resource / prompt 并直接调用。
- 推荐理由：开发 MCP 必备，Postman 之于 REST。

### punkpeye/awesome-mcp-servers ⭐
- URL：https://github.com/punkpeye/awesome-mcp-servers
- Star：约 48k
- 难度：🟢 入门
- 学习要点：社区收集的 1000+ MCP server 索引。
- 推荐理由：你想要的 MCP server 大概率在这里。

### punkpeye/awesome-mcp-clients
- URL：https://github.com/punkpeye/awesome-mcp-clients
- Star：约 4k
- 难度：🟢 入门
- 学习要点：可用 MCP 的客户端列表（Claude Desktop / Cline / Continue / 5ire / ChatMCP 等）。
- 推荐理由：找"哪个工具能跑我的 server"的最快目录。

### wong2/awesome-mcp-servers 🇨🇳
- URL：https://github.com/wong2/awesome-mcp-servers
- Star：约 4k
- 难度：🟢 入门
- 学习要点：punkpeye 的中文友好版，国内开发者维护。
- 推荐理由：中文社区视角。

### executeautomation/mcp-playwright
- URL：https://github.com/executeautomation/mcp-playwright
- Star：约 3k
- 难度：🟡 进阶
- 学习要点：把 Playwright 浏览器自动化包成 MCP；可让 Claude 直接驱动浏览器。
- 推荐理由：和后面 browser-use 对照看。

### modelcontextprotocol/quickstart-resources
- URL：https://github.com/modelcontextprotocol/quickstart-resources
- Star：约 1k
- 难度：🟢 入门
- 学习要点：官方 Quickstart 配套代码，覆盖 server / client / Claude Desktop 集成。
- 推荐理由：跟着官方文档敲一遍。

---

## 🌐 浏览器 / Computer Use

### browser-use/browser-use ⭐
- URL：https://github.com/browser-use/browser-use
- Star：约 52k
- 难度：🟡 进阶
- 学习要点：以"DOM tree → 编号元素 → LLM 选编号"为核心；不依赖视觉模型也能跑。
- 推荐理由：开源 web agent 现在的事实标杆，源码 ~5000 行可读。

### anthropics/anthropic-quickstarts/computer-use-demo ⭐
- URL：https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo
- Star：见 quickstarts 主仓库
- 难度：🟡 进阶
- 学习要点：Docker 容器内跑 Ubuntu desktop；Claude 通过截图 + xdotool 操作。
- 推荐理由：computer-use 范畴的"Hello World"。

### microsoft/playwright-mcp ⭐
- URL：https://github.com/microsoft/playwright-mcp
- Star：约 9k
- 难度：🟡 进阶
- 学习要点：微软官方 MCP server，把 Playwright 暴露给任意 MCP 客户端。
- 推荐理由：比上面 executeautomation 版更稳。

### steel-dev/steel-browser
- URL：https://github.com/steel-dev/steel-browser
- Star：约 4k
- 难度：🟡 进阶
- 学习要点：开源版"Browserbase"，带 session 录制、CAPTCHA 解、residential proxy 接入。
- 推荐理由：自己跑 browser farm 的开源底座。

### MultiOn/multion
- URL：https://github.com/MultiOn/multion
- Star：约 2k
- 难度：🟡 进阶
- 学习要点：闭源服务的开源 SDK；保留它是为了对比生态。
- 推荐理由：作为参考存档。

### OpenInterpreter/open-interpreter ⭐
- URL：https://github.com/OpenInterpreter/open-interpreter
- Star：约 57k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：自然语言驱动本地终端；可看作"computer-use 的命令行版"。
- 推荐理由："让 LLM 跑代码"领域开山项目，源码值得通读。

### openai/CUA-OpenAI-Operator（参考）
- URL：https://github.com/openai/cua（社区相关项目）
- Star：约 1k
- 难度：🔴 高阶
- 学习要点：OpenAI Operator 风格的 computer-use 复现。
- 推荐理由：作为 Anthropic computer-use 的对照。

---

## 🤖 编码 Agent / Cursor 类

### All-Hands-AI/OpenHands ⭐
- URL：https://github.com/All-Hands-AI/OpenHands
- Star：约 38k
- 难度：🔴 高阶
- 学习要点：开源版 Devin；docker sandbox 执行；浏览器 + 文件 + bash 三栖。
- 推荐理由：完整的"软件工程 Agent"工程化样板。

### smol-ai/developer
- URL：https://github.com/smol-ai/developer
- Star：约 12k
- 难度：🟢 入门
- 学习要点：~200 行 Python 跑一个最小编码 Agent。
- 推荐理由："你也可以写一个 GitHub Copilot"的鼓舞型项目。

### paul-gauthier/aider ⭐
- URL：https://github.com/Aider-AI/aider
- Star：约 22k
- 难度：🟡 进阶
- 学习要点：repo-map（用 tree-sitter 抽取代码结构）；diff 应用模式。
- 推荐理由：命令行编码 Agent 的代表，benchmark（aider polyglot）也是行业参考。

### Aider-AI/aider
- URL：https://github.com/Aider-AI/aider
- Star：见上
- 难度：🟡 进阶
- 学习要点：同上；aider 当前的官方 org。
- 推荐理由：保留两条 URL 是因为社区两个名字都有引用。

### gpt-engineer-org/gpt-engineer
- URL：https://github.com/gpt-engineer-org/gpt-engineer
- Star：约 53k
- 难度：🟢 入门
- 学习要点：从一段需求生成整个项目；clarify-then-generate 模式。
- 推荐理由：早期"agent 写代码"的代表，源码简单清晰。

### assafelovic/gpt-researcher
- URL：https://github.com/assafelovic/gpt-researcher
- Star：约 16k
- 难度：🟡 进阶
- 学习要点：plan-search-aggregate-write 多 agent 流；output 是研究报告。
- 推荐理由："研究 Agent"领域的范本。

### stitionai/devika
- URL：https://github.com/stitionai/devika
- Star：约 19k
- 难度：🟡 进阶
- 学习要点：开源 Devin 早期克隆；含 plan / browse / code agent。
- 推荐理由：看历史。

### princeton-nlp/SWE-agent ⭐
- URL：https://github.com/princeton-nlp/SWE-agent
- Star：约 14k
- 难度：🔴 高阶
- 学习要点：在 SWE-bench 上的顶尖方案；"Agent-Computer Interface"理论值得读。
- 推荐理由：学术界"软件工程 Agent"代表，论文配代码。

---

## 📊 Agent 评测 / 可观测

### langfuse/langfuse ⭐
- URL：https://github.com/langfuse/langfuse
- Star：约 7k
- 难度：🟡 进阶
- 学习要点：trace / span / generation 数据模型；自托管 Web UI；评测、prompt 版本管理。
- 推荐理由：自托管的 LLM 应用观测事实标准。

### Arize-ai/phoenix
- URL：https://github.com/Arize-ai/phoenix
- Star：约 4k
- 难度：🟡 进阶
- 学习要点：Notebook 内启动；OTel-based；带 RAG eval。
- 推荐理由：本地起来最快的 LLM trace 工具。

### traceloop/openllmetry
- URL：https://github.com/traceloop/openllmetry
- Star：约 5k
- 难度：🟡 进阶
- 学习要点：把所有主流 LLM SDK 自动包成 OpenTelemetry。
- 推荐理由：你已经用 OTel 的话直接接它。

### agenta-ai/agenta
- URL：https://github.com/agenta-ai/agenta
- Star：约 2k
- 难度：🟡 进阶
- 学习要点：Prompt playground + 评测 + 部署一站式。
- 推荐理由：替代 PromptLayer / Helicone 的开源选择。

### langwatch/langwatch
- URL：https://github.com/langwatch/langwatch
- Star：约 1k
- 难度：🟡 进阶
- 学习要点：Web UI 强、Studio 拖拉拽。
- 推荐理由：产品视角的 LLM 观测。

### uptrain-ai/uptrain
- URL：https://github.com/uptrain-ai/uptrain
- Star：约 2k
- 难度：🟡 进阶
- 学习要点：60+ pre-built check；retraining trigger。
- 推荐理由：质量监控向。

### princeton-nlp/SWE-bench
- URL：https://github.com/princeton-nlp/SWE-bench
- Star：约 3k
- 难度：🔴 高阶
- 学习要点：300 个真实 GitHub issue；现已成为编码 Agent 的"高考"。
- 推荐理由：评测自己 Agent 的标准 benchmark。

### OSU-NLP-Group/Mind2Web
- URL：https://github.com/OSU-NLP-Group/Mind2Web
- Star：约 1k
- 难度：🔴 高阶
- 学习要点：2000+ 真实网站任务；Web Agent benchmark。
- 推荐理由：browser-use 论文里在用它。

### web-arena-x/webarena
- URL：https://github.com/web-arena-x/webarena
- Star：约 1k
- 难度：🔴 高阶
- 学习要点：在 Docker 里搭出 GitLab/Reddit/电商等可重现环境。
- 推荐理由：Web Agent 评测最严肃方案。

---

## 🎨 多 Agent / 模拟

### microsoft/autogen
- URL：https://github.com/microsoft/autogen
- Star：约 36k
- 难度：🟡 进阶
- 学习要点：见框架部分；GroupChat 实现是多 Agent 对话经典。
- 推荐理由：多 Agent 起手第一站。

### camel-ai/camel
- URL：https://github.com/camel-ai/camel
- Star：约 15k
- 难度：🟡 进阶
- 学习要点：role-playing 双 Agent 论文同名；Society / Workforce 多 agent。
- 推荐理由：学术与工程结合得不错。

### a16z-infra/ai-town
- URL：https://github.com/a16z-infra/ai-town
- Star：约 8k
- 难度：🟡 进阶
- 学习要点：Convex 后端 + React 前端的"虚拟小镇"。
- 推荐理由：Stanford generative agents 论文的工程版，玩起来最有意思的多 agent demo。

### Significant-Gravitas/AutoGPT
- URL：https://github.com/Significant-Gravitas/AutoGPT
- Star：约 170k
- 难度：🟡 进阶
- 学习要点：2023 神话级开源项目，现已转型为 platform。
- 推荐理由：看 Agent 这一波是怎么起来的。

### yoheinakajima/babyagi
- URL：https://github.com/yoheinakajima/babyagi
- Star：约 20k
- 难度：🟢 入门
- 学习要点：100 行 Python 演示 task-driven 自主 agent。
- 推荐理由：和 AutoGPT 是兄弟，更小更易读。

---

## 💎 Easy-vibe Agent 项目（30 分钟跑 + 可改造）

### Shubhamsaboo/awesome-llm-apps ⭐ 必逛
- URL：https://github.com/Shubhamsaboo/awesome-llm-apps
- Star：约 45k
- 难度：🟢 入门
- 学习要点：100+ 实例代码：RAG、Agent、Tool use、Memory，每个 50-200 行。
- 推荐理由：Agent 时代的"代码鸡汤"，挑一个改一个。
- 中文：🌍 但代码极简。

### e2b-dev/awesome-ai-agents ⭐
- URL：https://github.com/e2b-dev/awesome-ai-agents
- Star：约 18k
- 难度：🟢 入门
- 学习要点：按"开源 / 闭源 / 框架 / 应用 / benchmark"分类的 Agent 全景索引。
- 推荐理由：每周翻一次掌握行业脉搏。

### AnswerDotAI/fasthtml
- URL：https://github.com/AnswerDotAI/fasthtml
- Star：约 6k
- 难度：🟢 入门
- 学习要点：Jeremy Howard 出品的"全栈 Python"——HTML 即 Python 函数。
- 推荐理由：写 Agent demo 前端最快路径。

### simonw/llm
- URL：https://github.com/simonw/llm
- Star：约 5k
- 难度：🟢 入门
- 学习要点：CLI 工具；插件可挂 tool；与 sqlite 对话历史天然集成。
- 推荐理由：Simon Willison 工程美学的代表作。

### mckayfairbanks/awesome-agents
- URL：https://github.com/mckayfairbanks/awesome-agents（社区索引）
- Star：约 1k
- 难度：🟢 入门
- 学习要点：按用途分类的 Agent 项目集合。
- 推荐理由：作为 e2b-dev 列表的补充。

### kyrolabs/awesome-langchain
- URL：https://github.com/kyrolabs/awesome-langchain
- Star：约 9k
- 难度：🟢 入门
- 学习要点：LangChain 生态项目集合。
- 推荐理由：还在用 langchain 的话，从这里淘金。

### EasyVibe / Vibe Agent 风格项目（关键词）
- 推荐方向：搜索 GitHub trending 关键词 `vibe-coding` / `vibe-agent` / `easy-agent`，社区里有大量"30 分钟开箱"风格的 demo 仓库；本节关注的是"可立刻跑、可立刻改"的小项目方法论而非某个固定仓库。
- 推荐理由：保持对"开箱即用"风格的敏感度，是 Agent 落地的关键直觉。

---

## 🏆 学习路径推荐 - 真实的 Agent 产品

> 这一组是"已经在用户手里赚钱"的开源 Agent 产品，看它们怎么做产品化是 08 章的"应用篇"。

### Cline（VSCode 插件，开源 Cursor 替代）⭐
- URL：https://github.com/cline/cline
- Star：约 36k
- 难度：🟡 进阶
- 学习要点：VSCode 插件架构；MCP 客户端实现；plan/act 二元模式。
- 推荐理由：在 VSCode 里学 MCP 最直接的入口。

### continuedev/continue
- URL：https://github.com/continuedev/continue
- Star：约 22k
- 难度：🟡 进阶
- 学习要点：multi-IDE（VSCode + JetBrains）；自定义 model provider；context provider 系统。
- 推荐理由：架构最干净的开源 IDE Agent。

### danny-avila/LibreChat
- URL：https://github.com/danny-avila/LibreChat
- Star：约 25k
- 难度：🟡 进阶
- 学习要点：多 Provider 统一前端；自带 Agents、文件、Auth、code interpreter。
- 推荐理由：自托管 ChatGPT 平替的工业级方案。

### lobehub/lobe-chat ⭐ 🇨🇳
- URL：https://github.com/lobehub/lobe-chat
- Star：约 50k
- 难度：🟡 进阶
- 学习要点：Next.js 14 工程标杆；插件、Agent Market、知识库、TTS/STT。
- 推荐理由：中文社区最活跃的开源 ChatGPT 替代品。

---

## 🎓 真正能学到东西的小项目（Easy-vibe 严选 10 个）

> 严格的标准：**能在一个晚上读完核心代码 + 改出一个能 demo 的版本**。

### 1. anthropic-cookbook patterns/agents ⭐
- URL：https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents
- 量级：5 个 Notebook，每个约 200 行
- 学到什么：5 个 Agent 模式的最小实现，**没有任何框架**。读完它你会发现 90% 的"Agent 框架"只是给这 200 行包了一层。
- 怎么改：把 Orchestrator-Workers 模式改成"研究助手"——orchestrator 拆任务，workers 并行跑搜索。

### 2. lavague-ai/LaVague
- URL：https://github.com/lavague-ai/LaVague
- 量级：~3000 行 Python
- 学到什么：自然语言 → Selenium 操作的最简实现；retrieval-augmented action prediction。
- 怎么改：把它的 LLM 切成 Claude，让它操作你常用的网站（如订餐、查快递）。

### 3. ItzCrazyKns/Perplexica ⭐
- URL：https://github.com/ItzCrazyKns/Perplexica
- Star：约 17k
- 量级：Next.js + Express，~5000 行
- 学到什么：开源 Perplexity；SearxNG 做搜索；reranker 选 top-k。
- 怎么改：替换搜索源为你内部知识库，瞬间变成"企业版 Perplexity"。

### 4. mckaywrigley/chatbot-ui
- URL：https://github.com/mckaywrigley/chatbot-ui
- Star：约 29k
- 量级：Next.js + Supabase
- 学到什么：最干净的 ChatGPT 风 UI；流式渲染、Markdown 处理。
- 怎么改：套自己的 system prompt + tool，2 小时上线一个垂直 chat。

### 5. shroominic/codeinterpreter-api
- URL：https://github.com/shroominic/codeinterpreter-api
- Star：约 4k
- 量级：~2000 行 Python
- 学到什么：本地 sandboxed Python 执行；ChatGPT Code Interpreter 的开源复刻。
- 怎么改：换 sandbox 为 e2b 或 modal，做生产级。

### 6. SciPhi-AI/R2R
- URL：https://github.com/SciPhi-AI/R2R
- Star：约 5k
- 量级：FastAPI + 多 vector store
- 学到什么：生产 RAG：ingestion / hybrid search / KG / agentic retrieval 全链路。
- 怎么改：作为你公司知识库的底座。

### 7. microsoft/AI-Agents-for-Beginners ⭐
- URL：https://github.com/microsoft/AI-Agents-for-Beginners
- Star：约 15k
- 量级：10 节课
- 学到什么：从 ReAct → Planning → Tool use → Multi-agent → AutoGen 实战。
- 怎么改：跟着课走完，每节都把示例改一个 Agent 任务。

### 8. Doriandarko/o1-engineer
- URL：https://github.com/Doriandarko/o1-engineer
- Star：约 3k
- 量级：~1000 行 Python
- 学到什么：用 o1 / Claude 4 这种"会思考的模型"做编码 agent 的极简范式。
- 怎么改：拿来直接当个人编码助手。

### 9. OpenDevin/OpenDevin（即 OpenHands）
- URL：https://github.com/All-Hands-AI/OpenHands
- Star：约 38k
- 量级：~50k 行 Python+TS
- 学到什么：完整软件工程 Agent：sandbox、browser、LLM、UI；event stream architecture。
- 怎么改：太大不必全改，挑一个 agent skill 替换或加新 skill。

### 10. Aider-AI/aider ⭐
- URL：https://github.com/Aider-AI/aider
- Star：约 22k
- 量级：~30k 行 Python
- 学到什么：repo-map、tree-sitter 抽 ctags、edit format（whole / diff / udiff）切换。
- 怎么改：把 aider 当自己日常 commit 工具，这本身就是学习。

---

## 📰 必关注组织 / 个人

### GitHub 组织
- **anthropics**：Claude / cookbook / SDK / courses
- **langchain-ai**：LangChain / LangGraph / LangSmith
- **microsoft**：autogen / generative-ai-for-beginners / AI-Agents-for-Beginners
- **openai**：openai-cookbook / openai-agents-python / swarm
- **modelcontextprotocol**：MCP 全家
- **e2b-dev**：sandbox / awesome-ai-agents
- **huggingface**：transformers / smolagents / lerobot
- **stanfordnlp**：dspy / 多 agent 论文同步代码

### Twitter / X
- **@AnthropicAI**：官方更新一手源
- **@LangChainAI**：每周新 feature
- **@simonw**：Simon Willison，工程美学之神
- **@karpathy**：Andrej Karpathy，源码即教材
- **@swyx**：Latent Space podcast，AI 趋势第一线

### 中文
- **Datawhale**（GitHub: datawhalechina）：中文 LLM 学习资料库
- **稀土掘金 AI**（juejin.cn）：每天 AI 工程实践文章
- **机器之心 GitHub**：论文 + 代码索引

---

## 🗓️ 60 天 Agent 学习仓库阅读路线

> 核心原则：**先读官方再读社区，先 200 行再 5 万行**。

| 周 | 主题 | 主仓库 | 副仓库 / 任务 |
|----|------|--------|---------------|
| W1 | Agent 范式 | anthropic-cookbook/patterns/agents | 通读 5 个 notebook |
| W2 | Anthropic 官方 | anthropics/courses | tool_use + prompt_evaluations |
| W3 | OpenAI 对照 | openai/openai-cookbook | function calling + assistants |
| W4 | Claude SDK | anthropics/claude-agent-sdk-python | 写 1 个 mini agent |
| W5 | OpenAI SDK | openai/openai-agents-python | handoff demo |
| W6 | LangGraph | langchain-ai/langgraph | tutorial 全跑一遍 |
| W7 | CrewAI / AutoGen | crewAIInc/crewAI + microsoft/autogen | 各起 1 个多 agent demo |
| W8 | DSPy | stanfordnlp/dspy | 用 DSPy 优化一个 prompt |
| W9 | MCP 入门 | modelcontextprotocol/python-sdk | 写 1 个 MCP server |
| W10 | MCP 进阶 | modelcontextprotocol/servers + inspector | 改 1 个官方 server |
| W11 | Browser Agent | browser-use/browser-use | 让它帮你订餐 |
| W12 | Computer Use | anthropic-quickstarts/computer-use-demo | Docker 里跑通 |
| W13 | Coding Agent 1 | Aider-AI/aider | 用 aider 提一个 PR |
| W14 | Coding Agent 2 | All-Hands-AI/OpenHands | 跑通本地版 |
| W15 | 评测 | langfuse/langfuse + SWE-bench | 接到自己 agent |
| W16 | 多 agent 模拟 | a16z-infra/ai-town | 周末玩一玩 |
| W17 | 长期记忆 | letta-ai/letta + mem0ai/mem0 | 给自己 agent 加记忆 |
| W18 | 选 1 个产品深入 | Cline 或 lobe-chat | 阅读源码并提 PR |

完成 60 天，你就走完了 Agent 工程师的"独立路径"。

---

## 🌳 Agent 框架决策树（如何选适合自己的）

```
你的目标是什么？
│
├─ 学习/研究/演示 < 50 行代码
│  └─ 用 anthropic-cookbook patterns/agents 直接照抄
│
├─ 单机本地脚本/工具
│  ├─ Python 强类型偏好 → pydantic-ai
│  ├─ 命令行偏好 → simonw/llm + 自定义 plugin
│  └─ 极简 → openai/swarm 或 anthropic SDK 裸调用
│
├─ 多 Agent 协作（GroupChat / role-play）
│  ├─ 工程化产品 → microsoft/autogen v0.4
│  └─ 5 分钟出 demo → crewAIInc/crewAI
│
├─ 复杂状态机/审批流/HITL
│  └─ langchain-ai/langgraph（事实标准）
│
├─ Web 浏览器自动化
│  ├─ DOM 派 → browser-use/browser-use
│  ├─ 视觉派 → anthropic computer-use-demo
│  └─ Playwright 已有团队 → microsoft/playwright-mcp
│
├─ 编码 Agent
│  ├─ 命令行/CI 集成 → Aider
│  ├─ 完整产品 → OpenHands
│  └─ IDE 插件 → Cline 或 Continue
│
├─ 实时语音
│  └─ livekit/agents（无替代品）
│
├─ 长期记忆
│  ├─ 简单 → mem0ai/mem0
│  └─ 论文风 → letta-ai/letta
│
├─ 自动 Prompt 优化
│  └─ stanfordnlp/dspy
│
└─ 集成大量外部工具
   └─ MCP 协议 + 现有 awesome-mcp-servers 中找
```

---

## 📌 学习心法

1. **先 200 行再 5 万行**：从 anthropic-cookbook patterns 起步，否则容易陷入"框架陷阱"。
2. **先官方再社区**：Anthropic / OpenAI / MCP 官方是真理之源。
3. **写比读重要**：每读一个仓库，至少改一处跑通。
4. **MCP 是未来**：花 1 周专门做 MCP，回报极高。
5. **跟 3 个人**：Karpathy（思想）、Simon Willison（工程）、Anthropic devrel（前沿）。
6. **写复盘**：每读一个仓库，在 vault 写 200 字"3 句话讲清楚它"。
7. **不要贪多框架**：选 1 个主用（推荐 LangGraph 或裸 SDK）+ 1 个备选（CrewAI 或 AutoGen）即可。

---

> 本文配套：第 08 章「AI Agent - 框架与源码」 · 全栈学习路线收尾。
> 配套兄弟篇：06 章数据库 / DevOps、07 章 LLM 学习。
