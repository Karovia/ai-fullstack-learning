---
title: 第 08 章 - AI Agent 框架与源码
chapter: 08
duration: 12 周
prerequisite: [[07-AI基础-Prompt与LLM/07-AI基础总纲]]
tags: [AI-Agent, MCP, LangGraph, 源码级]
---

# 第 08 章 · AI Agent 框架与源码（12 周）

> 这是整条路线的**核心章节**。学完这一章，你能独立设计、实现、调优生产级 Agent 系统。

## 🎯 章节目标

- 理解 Agent 设计模式（Anthropic 5 模式 + 多 Agent 协作）
- 精通 LangGraph（事实标准）+ LangChain
- 掌握 MCP（Model Context Protocol）—— 实现 Server + Client
- 用 Claude Agent SDK / OpenAI Agents SDK
- 多 Agent 框架：CrewAI / AutoGen
- 浏览器自动化 Agent：browser-use / Computer Use
- **源码级**阅读 LangGraph、Claude Code、OpenAI Agents SDK
- 完整 Agent 工程化：可观测性 / 评测 / 安全 / 限流

## 📅 周计划

| 周 | 主题 | 产出 |
|---|------|------|
| W1 | Agent 总览 + Anthropic 5 模式 | 5 个 demo |
| W2 | 手写 Agent（不用框架，纯 while loop） | mini-agent |
| W3 | LangChain + LangGraph 入门 | LangGraph 基础 |
| W4 | LangGraph 进阶：StateGraph / 子图 / 并行 / 持久化 | 客服 Agent |
| W5 | MCP 协议详解 + 写第一个 MCP Server | 自定义 MCP server |
| W6 | Claude Agent SDK + OpenAI Agents SDK | 双框架 demo |
| W7 | 多 Agent 协作：CrewAI / AutoGen / LangGraph 多 Agent | 模拟公司 |
| W8 | 浏览器/电脑控制 Agent：Computer Use / browser-use | 自动化助手 |
| W9 | Agent 评测：AgentBench / SWE-bench / 自建 | 评测系统 |
| W10 | 可观测性：LangSmith / Langfuse / OpenTelemetry | 全链路追踪 |
| W11 | 源码级 1：LangGraph 源码导读 | 源码注释 |
| W12 | 源码级 2：Claude Agent SDK / Code 源码 + 写 mini 版 | mini-agent-sdk |

## 📖 详细内容

### W1 - Agent 全景

**Anthropic 5 大模式（Building Effective Agents）：**
1. **Augmented LLM** - LLM + 检索 + 工具
2. **Prompt Chaining** - 顺序链
3. **Routing** - 分类后路由到专门 prompt
4. **Parallelization** - 并行（投票 / 分片）
5. **Orchestrator-Workers** - 编排者 + 工作者
6. **Evaluator-Optimizer** - 生成 + 评判循环

**Agent vs Workflow** 区别：
- **Workflow**: 流程编排，控制权在代码
- **Agent**: LLM 自主决策循环（while not done）

**必读**：[Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)

### W2 - 手写 Agent（关键！）
不依赖任何框架，从零实现一个 Agent：

```python
while True:
    response = call_llm(messages, tools)
    if response.stop_reason == "end_turn":
        break
    for tool_call in response.tool_uses:
        result = execute_tool(tool_call)
        messages.append(result)
```

亲手写一遍，再去看框架就秒懂。

参考 Anthropic Cookbook 的 [agents](https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents) 目录。

### W3 - LangChain + LangGraph 入门
- LangChain 现状（2026：核心已稳定，重心转向 LangGraph）
- LCEL（LangChain Expression Language）
- LangGraph 核心概念：State / Node / Edge / Conditional Edge
- 第一个 Graph：单节点 → 多节点 → 条件分支

### W4 - LangGraph 进阶
- StateGraph（带状态的图）
- 子图（subgraph）
- 并行节点（fan-out / fan-in）
- 持久化（Checkpointer）+ Human-in-the-loop
- Streaming + 中断恢复
- LangGraph Studio 可视化调试

### W5 - MCP 协议（重点！）
**MCP = Model Context Protocol**，Anthropic 2024 推出，2025 已成跨厂商事实标准（OpenAI / Google / Cursor / Claude Code 全部支持）。

- 协议三要素：**Tools**（执行）/ **Resources**（数据）/ **Prompts**（模板）
- 通信：stdio / HTTP+SSE / Streamable HTTP
- Server 实现（Python SDK / TypeScript SDK）
- Client 集成（Claude Desktop / Cursor / 你自己的 Agent）

**实战**：写一个 "Obsidian Vault MCP Server"，暴露 Tools：list_notes / read_note / create_note / search。

### W6 - 官方 Agent SDK
- **Claude Agent SDK** - Anthropic 官方
- **OpenAI Agents SDK** - OpenAI 官方（2025）
- 对比：handoff / guardrail / tracing 设计

### W7 - 多 Agent 协作
- **CrewAI** - 角色化（Researcher / Writer / Critic）
- **AutoGen** - Microsoft，对话驱动
- **LangGraph 多 Agent** - 最灵活
- Supervisor 模式 vs Hierarchical 模式 vs Network 模式

### W8 - 浏览器 / 电脑控制 Agent
- **Computer Use** (Anthropic) - 真实操作鼠标键盘
- **browser-use** - 浏览器专用，更稳
- **Playwright** + LLM 自定义实现
- 安全沙箱：Docker + VNC

### W9 - Agent 评测
- 任务级：SWE-bench / AgentBench / GAIA / WebArena
- 自建 eval：用真实业务任务集
- 多轮对话评测：[Agent-as-a-Judge](https://arxiv.org/abs/2410.10934)
- 失败分类：工具调用错 / 推理错 / 上下文超长 / 幻觉

### W10 - 可观测性
- LangSmith（LangChain 出品，最完整）
- Langfuse（开源自托管）
- OpenTelemetry + Phoenix
- 必看 6 项指标：成功率 / 平均轮数 / token 成本 / 延迟 / 工具失败率 / 用户满意度

### W11 - LangGraph 源码导读
- `langgraph/graph/state.py` - StateGraph
- `langgraph/pregel/` - Pregel 引擎（图执行内核）
- `langgraph/checkpoint/` - 持久化
- 写 100+ 行源码注释

### W12 - Claude Agent SDK 源码 + 写 mini 版
- 阅读 [`anthropic-quickstarts/agents`](https://github.com/anthropics/anthropic-quickstarts) 和 Claude Agent SDK
- 用 < 500 行 Python 写一个 mini-agent-sdk，含：
  - Agent 定义
  - 工具注册
  - 执行循环
  - Handoff
  - Streaming
  - 简易 trace

## 📝 考核任务

- [ ] **T1**：用裸 API 实现 Anthropic 5 模式各一个 demo
- [ ] **T2**：手写一个 Agent（< 200 行），不用任何框架，能完成 5 个工具任务
- [ ] **T3**：用 LangGraph 实现一个客服 Agent（含 RAG + 工具 + 人工接管）
- [ ] **T4**：写一个 MCP Server 暴露 Obsidian 操作，接到 Claude Desktop 用起来
- [ ] **T5**：用 CrewAI 模拟一个内容工作室（选题 → 调研 → 写作 → 审校 → 发布）
- [ ] **T6**：用 browser-use 做一个自动找工作 Agent（投简历 → 跟踪进度）
- [ ] **T7**：搭建 Langfuse 自托管 + 接入你的 Agent，看 7 天 trace 报告
- [ ] **T8**：构建 50 题的 eval 集，对比 3 种 Agent 实现的成功率
- [ ] **T9**：阅读 LangGraph `pregel` 模块全部源码，做行级注释
- [ ] **T10**：写一个 mini-agent-sdk（< 500 行），开源到 GitHub

## 🚀 章节大项目（多选一 + 必做一个）

> 三选一作为本章主项目，建议至少做完 1 个 + 简版另一个。

### 项目 A：`AutoCoder` —— Claude Code 简版

做一个能自主写代码的命令行 Agent：
- 接受需求 → 探索代码库 → 规划 → 编辑 → 测试 → 修 bug
- MCP 工具：read_file / edit_file / run_bash / search
- 多步骤 plan / human approval / undo
- 流式输出 + 实时 diff 展示
- 评测：能自动修 5 个真实 GitHub issue

**技术栈**：Python + Claude Sonnet + MCP + Rich CLI

### 项目 B：`AgentOps` —— Agent 运营平台

做一个让团队管理多个 Agent 的 SaaS：
- Agent 编辑器（拖拽节点 = LangGraph Studio 简版）
- 在线运行 + 流式查看
- Trace + 评测仪表盘
- A/B 测试两个 prompt 版本
- 团队协作 + 权限

**技术栈**：Next.js 15 + FastAPI + LangGraph + Langfuse + PostgreSQL

### 项目 C：`Family CFO` —— 家庭财务多 Agent

模拟家庭财务团队：
- **Bookkeeper**: 自动从邮件/银行 API 抓账单分类
- **Analyst**: 月度报告 + 异常检测
- **Advisor**: 给出预算建议 + 投资建议（**仅供参考**）
- **Tax Helper**: 报税预演

**技术栈**：CrewAI + LangGraph + 真实银行 API（开放银行 / 信用卡 OFX）+ 邮件 IMAP

### 验收标准（任选项目通用）
- [ ] 全部 Agent 行为有 trace 可看
- [ ] eval 集 ≥ 30 题，成功率 ≥ 80%
- [ ] 至少集成 1 个自定义 MCP Server
- [ ] 流式响应（不能等全部完成才显示）
- [ ] 安全：工具有 confirm 模式 / 危险操作隔离
- [ ] 成本控制：单次任务 < $0.5，超预算告警
- [ ] 完整 README + 视频演示 + 架构图

### 进阶挑战
- 把 Agent 部署成 MCP Server，让别人也能调用
- 加入长期记忆（Letta / mem0）
- 用 [DSPy](https://github.com/stanfordnlp/dspy) 自动优化 prompt
- 训练一个小 LoRA 让 Agent 在你的数据上更聪明（用 unsloth）

## 📚 推荐资源

### 必读核心
- 📖 [Building Effective Agents (Anthropic)](https://www.anthropic.com/research/building-effective-agents) - **第一原理**
- 🛠️ [Anthropic Cookbook - Agents](https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents)
- 📖 [Anthropic - Agentic Coding Best Practices](https://www.anthropic.com/engineering)
- 📖 [LangGraph 官方文档](https://langchain-ai.github.io/langgraph/)
- 📖 [MCP 官方文档](https://modelcontextprotocol.io)

### 框架文档
- 📖 [LangChain Python](https://python.langchain.com/)
- 📖 [CrewAI Docs](https://docs.crewai.com/)
- 📖 [AutoGen](https://microsoft.github.io/autogen/)
- 📖 [Claude Agent SDK](https://docs.anthropic.com/en/api/agent-sdk/overview)
- 📖 [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)

### MCP
- 🛠️ [MCP Servers Repo](https://github.com/modelcontextprotocol/servers) - 官方参考实现
- 🛠️ [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)

### 多 Agent
- 🛠️ [awesome-ai-agents](https://github.com/e2b-dev/awesome-ai-agents)
- 🛠️ [awesome-llm-apps (Shubhamsaboo)](https://github.com/Shubhamsaboo/awesome-llm-apps) - 100+ 真实 Agent 应用

### 浏览器 / Computer Use
- 🛠️ [browser-use](https://github.com/browser-use/browser-use)
- 🛠️ [Anthropic Computer Use Quickstart](https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo)

### 评测 / 可观测
- 🛠️ [Langfuse](https://github.com/langfuse/langfuse) - 开源
- 🛠️ [Phoenix (Arize)](https://github.com/Arize-ai/phoenix)
- 🛠️ [LangSmith](https://smith.langchain.com/)

### 进阶研究方向
- 📖 [Stanford CS 25 - Transformers](https://web.stanford.edu/class/cs25/)
- 📖 [DSPy](https://github.com/stanfordnlp/dspy) - 自动优化 prompt
- 📖 [Letta](https://github.com/letta-ai/letta) - 长期记忆

---

✅ 12 周后 → 进入 [[09-综合实战-毕业项目/09-毕业项目总纲]]
