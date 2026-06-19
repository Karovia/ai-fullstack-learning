---
chapter: 08
week: W01
title: Agent 全景与 5 大模式
date: 2026-06-19
tags: [agent, anthropic, building-effective-agents, 全景]
prereq: [07-LLM基础完成]
estimated_hours: 14
difficulty: ★★★☆☆
---

# W01 · Agent 全景与 5 大模式

> 这一周不写多少代码，但**这是整本路线最关键的一周**。看不懂 Agent 的本质，后面框架学得越多越乱。

---

## 一、一句话搞懂 Agent

**Agent ＝ 一个会自己思考、会调工具、能循环到任务完成的 LLM。**

把它拆开：
- **会思考**：每一步都让 LLM 决定"下一步干什么"
- **会调工具**：可以执行代码、读文件、搜网页、调 API
- **会循环**：一次调不完就再调一次，直到自己说"完成了"

类比：
- **Workflow** 是**自动化流水线**——零件按固定顺序走，机器是哑巴。
- **Agent** 是**派出去的实习生**——你只说目标，他自己想办法，遇到问题自己解决。

---

## 二、Agent vs Workflow：核心区别

|  | Workflow | Agent |
|---|---|---|
| 流程控制 | **写在代码里** | **LLM 自主决定** |
| 步骤数 | **固定** | **不定** |
| 可预测性 | 高 | 低 |
| 灵活性 | 低 | 高 |
| 成本 | 低 | 高（多次 LLM 调用） |
| 调试难度 | 容易 | 难 |
| 适用场景 | 已知流程的任务 | 开放式任务 |

### 决策树：到底选哪个？

```
任务是什么？
├── 步骤完全已知，每次一样 → Workflow（甚至不用 LLM）
├── 步骤已知但内容靠 LLM 生成（写文章 → 翻译 → 校对）→ Workflow（Prompt Chaining）
├── 步骤分几条已知路径，靠 LLM 分流 → Workflow（Routing）
├── 多个独立子任务并行 → Workflow（Parallelization）
├── 有"评审-改进"环节 → Workflow（Evaluator-Optimizer）
├── 任务复杂、步骤数和顺序未知 → Agent
└── 任务可能失败需要自我纠正 → Agent
```

> Anthropic 的核心建议：**"先用 Workflow，扛不住再上 Agent"**。Agent 慢、贵、不稳定，Workflow 是更工程化的选择。

---

## 三、Agent 的兴起时间线

| 时间 | 节点 | 意义 |
|---|---|---|
| 2022.11 | ChatGPT 发布 | LLM 进入大众视野 |
| 2023.06 | OpenAI Function Calling | LLM 第一次能调工具 |
| 2023.10 | AutoGPT / BabyAGI 火爆 | 第一波"自主 Agent"，但很玩具 |
| 2024.04 | Claude Tool Use 正式版 | 工具调用成为基础能力 |
| 2024.10 | Anthropic 发布 Computer Use | Agent 能直接操作电脑 |
| 2024.11 | **MCP 协议发布** | 工具集成的统一标准 |
| 2025.03 | Claude Code、Cursor Agent、Devin | 编码 Agent 进入生产 |
| 2025.06 | OpenAI Agents SDK / Claude Agent SDK | 官方 Agent 抽象 |
| 2026.01 | 长任务自主 Agent（数小时连续工作）| 当前前沿 |

---

## 四、Anthropic 5 大模式（Building Effective Agents）

> 出自 Anthropic 官方博客《Building Effective Agents》，是工业界事实标准。

### 0. 基础原子：Augmented LLM

LLM ＋ **检索 ＋ 工具 ＋ 记忆**。这是后面所有模式的"原子"。

```python
from anthropic import Anthropic

client = Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    tools=[{
        "name": "search_docs",
        "description": "在知识库里搜索",
        "input_schema": {
            "type": "object",
            "properties": {"query": {"type": "string"}},
            "required": ["query"]
        }
    }],
    messages=[{"role": "user", "content": "公司退款政策是什么？"}]
)
print(response)
```

### 1. Prompt Chaining（顺序链）

把任务拆成有先后的几个 LLM 调用。

```python
def write_blog(topic: str) -> str:
    # Step 1: 大纲
    outline = client.messages.create(
        model="claude-sonnet-4-5", max_tokens=512,
        messages=[{"role": "user", "content": f"为「{topic}」写大纲"}]
    ).content[0].text

    # Step 2: 正文
    draft = client.messages.create(
        model="claude-sonnet-4-5", max_tokens=2048,
        messages=[{"role": "user", "content": f"按大纲写正文：\n{outline}"}]
    ).content[0].text

    # Step 3: 校对
    final = client.messages.create(
        model="claude-sonnet-4-5", max_tokens=2048,
        messages=[{"role": "user", "content": f"校对并润色：\n{draft}"}]
    ).content[0].text

    return final
```

**适合**：内容生成、翻译流水线、ETL 提示。
**注意**：每一步可以加"门控检查"（例如大纲不合格就打回）。

### 2. Routing（路由分诊）

先分类，再路由到不同的专门 prompt 或模型。

```python
def route(user_msg: str):
    cls = client.messages.create(
        model="claude-haiku-4-5",  # 用便宜的小模型分类
        max_tokens=20,
        messages=[{
            "role": "user",
            "content": f"分类：{user_msg}\n选项：[退款/技术/咨询]\n只输出标签。"
        }]
    ).content[0].text.strip()

    if "退款" in cls:
        return refund_agent(user_msg)
    elif "技术" in cls:
        return tech_agent(user_msg)
    else:
        return faq_agent(user_msg)
```

**适合**：客服分诊、多模型成本优化（简单问题走小模型）。

### 3. Parallelization（并行）

两种玩法：
- **Sectioning**：任务切片，并行处理（翻译大文档 → 切段 → 并行）
- **Voting**：同一问题多次问，投票/取最优

```python
import asyncio
from anthropic import AsyncAnthropic
aclient = AsyncAnthropic()

async def review_one(code: str, aspect: str):
    r = await aclient.messages.create(
        model="claude-sonnet-4-5", max_tokens=512,
        messages=[{"role": "user", "content": f"从{aspect}角度审查代码：\n{code}"}]
    )
    return aspect, r.content[0].text

async def parallel_review(code):
    aspects = ["安全性", "性能", "可读性"]
    results = await asyncio.gather(*[review_one(code, a) for a in aspects])
    return dict(results)
```

**适合**：大文档分片、多视角评审、自洽性投票。

### 4. Orchestrator-Workers（编排者-工人）

一个**大脑** LLM 拆任务 → 派给 N 个 worker LLM 并行执行 → 大脑汇总。

```python
def orchestrator(goal: str):
    # 大脑拆解
    plan = client.messages.create(
        model="claude-sonnet-4-5", max_tokens=512,
        messages=[{"role": "user", "content": f"把任务「{goal}」拆成 3-5 个独立子任务，输出 JSON 数组"}]
    ).content[0].text
    subtasks = json.loads(plan)

    # 工人并行执行
    results = [worker(t) for t in subtasks]

    # 大脑汇总
    final = client.messages.create(
        model="claude-sonnet-4-5", max_tokens=2048,
        messages=[{"role": "user", "content": f"根据子结果汇总最终答案：\n{results}"}]
    ).content[0].text
    return final
```

**适合**：研究报告、多文件代码改动、复杂调度。**Claude Code、Cursor 内部都用这个模式**。

### 5. Evaluator-Optimizer（生成-评判循环）

生成 → 让另一个 LLM 评分 → 不达标就改 → 循环。

```python
def write_with_review(prompt: str, max_rounds=3):
    draft = ""
    for i in range(max_rounds):
        # 生成/改进
        draft = client.messages.create(
            model="claude-sonnet-4-5", max_tokens=1024,
            messages=[{"role": "user",
                       "content": f"任务：{prompt}\n上一版：{draft}\n请生成或改进"}]
        ).content[0].text

        # 评判
        verdict = client.messages.create(
            model="claude-sonnet-4-5", max_tokens=256,
            messages=[{"role": "user",
                       "content": f"评分（PASS/FAIL + 理由）：\n{draft}"}]
        ).content[0].text

        if "PASS" in verdict:
            return draft
    return draft
```

**适合**：代码生成、文学润色、有"对错"标准的任务。

### 终极模式：真·Agent（自主循环）

不固定流程，让 LLM 自己看 tool_use 决定下一步：

```python
def agent_loop(user_msg, tools):
    messages = [{"role": "user", "content": user_msg}]
    while True:
        r = client.messages.create(
            model="claude-sonnet-4-5", max_tokens=2048,
            tools=tools, messages=messages
        )
        messages.append({"role": "assistant", "content": r.content})
        if r.stop_reason == "end_turn":
            return r
        # 执行工具
        tool_results = [run_tool(b) for b in r.content if b.type == "tool_use"]
        messages.append({"role": "user", "content": tool_results})
```

**这就是 W02 要展开手写的核心。**

---

## 五、多 Agent 协作

不是只有"单 Agent + N 工具"，还有"**多个 Agent 互相协作**"。

| 拓扑 | 谁说话 | 例子 |
|---|---|---|
| Hierarchical | 主管 → 多个下级 | Claude Code 的子 Agent |
| Network | 任意 Agent → 任意 | AutoGen 群聊 |
| Sequential | A → B → C | LangGraph 多 Agent 链 |
| Handoff | A 把控制权交给 B | OpenAI Agents SDK 的 handoff |

> 警告：**多 Agent 复杂度爆炸**。先单 Agent，证明撑不住再加。

---

## 六、真实生产案例

| 产品 | 用什么 |
|---|---|
| **Cursor** | 自研 Orchestrator-Workers + Tool Use，外加多模型路由 |
| **Claude Code** | 单 Agent 主循环 + 子 Agent (Task tool) + 大量 tool |
| **Devin** | 长程规划 + 浏览器/编辑器 tool + 自我反思 |
| **Replit Agent** | 代码生成 + 沙箱执行 + Evaluator |
| **Perplexity** | RAG + Routing + 简单 Agent |
| **GitHub Copilot Workspace** | Workflow 主导（规划 → 实施 → PR）|
| **Zapier Central** | Routing + Tool Use（接 N 个 SaaS） |

---

## 七、Agent 设计黄金原则

1. **简单优先**：能一个 prompt 解决，别上 chain；能 chain，别上 agent。
2. **明确边界**：工具的输入/权限/副作用要清晰（写文件 vs 删文件分开）。
3. **流式可观察**：每步都要能看见 LLM 在想什么、调了什么工具。
4. **失败可恢复**：tool 错误不应让循环崩溃，要 catch + 反馈给 LLM。
5. **成本可控**：设 token / 步数 / 时间 / 美元上限，超了停。
6. **人在环上**：危险操作（删数据、付钱、发邮件）必须问人。
7. **可重放**：保存所有 messages，便于复现 bug。

---

## 八、本周实战：5 模式各做一个 Demo

每个 demo < 100 行，全部跑通：

| Demo | 模式 | 目标 |
|---|---|---|
| 1 | Prompt Chaining | "主题 → 大纲 → 文章 → 校对" 写博客 |
| 2 | Routing | 客服分诊（退款/技术/咨询） |
| 3 | Parallelization | 并行翻译 5 段文字 |
| 4 | Orchestrator-Workers | 写一份"X 公司调研报告"（拆 5 个子题并行查） |
| 5 | Evaluator-Optimizer | 生成代码 → 评判 → 改 → 直到 PASS |

把代码放到 `demos/` 目录下，每个一个文件夹，附 README。

---

## 九、checklist

- [ ] 能用一句话讲清 Agent vs Workflow 区别
- [ ] 5 大模式各能说出适用场景
- [ ] 5 个 demo 都跑通
- [ ] 读完 Anthropic《Building Effective Agents》原文
- [ ] 看过至少 2 个生产 Agent（Claude Code / Cursor）的工作流截图

---

## 十、参考

- Anthropic, *Building Effective Agents* (2024.12)
- Anthropic Cookbook · Agents 章
- LangChain, *Multi-Agent Systems* 文档
- Claude Code 内部架构博客（Anthropic 2025）

下一周：**抛开所有框架，纯手写一个 300 行的 mini Agent**。
