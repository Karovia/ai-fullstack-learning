---
title: W07 多 Agent 协作
chapter: 08-AI-Agent-框架与源码
week: 7
date: 2026-06-19
tags: [agent, multi-agent, crewai, autogen, langgraph]
prev: "[[06-LangChain与LangGraph/W06-LangGraph进阶]]"
next: "[[W08-浏览器与Computer-Use]]"
---

# W07 多 Agent 协作

## 0. 一句话开场

单 Agent 像一个全能的人，什么都做但什么都不精。多 Agent 像一家公司，老板分活，每个员工只干自己擅长的，效率反而更高——这一周我们就来组建你的"AI 公司"。

## 1. 为什么要多 Agent

### 1.1 类比：饭店厨房

你一个人在家做饭，要切菜、炒菜、装盘、洗碗，做四个菜可能两小时。但米其林餐厅里：

- 主厨（Supervisor）：决定上什么菜、什么顺序
- 切配（Researcher）：洗、切、备料
- 炒锅（Writer）：负责火候
- 摆盘（Critic）：检查色香味再上桌

各司其职，二十分钟出十道菜。

### 1.2 多 Agent 的三大好处

1. **专业分工**：每个 Agent 用最适合的 prompt + 工具集
2. **上下文隔离**：调研 Agent 的 50 万字资料不会污染写作 Agent 的 context window
3. **角色明确**：让 LLM 扮演单一角色比"既是研究员又是作家"幻觉更少

### 1.3 多 Agent 的代价

- Token 成本翻倍（多 Agent 间要互相传消息）
- 延迟变高（串行执行更慢）
- 调试难（错误传播链长）
- 需要协调机制（不然吵架）

**判断准则**：单 Agent 能解决就别上多 Agent，但当任务跨领域、需要"审视自己产出"时，多 Agent 收益明显。

## 2. 多 Agent 三种拓扑

### 2.1 Supervisor（中心调度）

```
        ┌──────────┐
        │Supervisor│
        └────┬─────┘
       ┌─────┼─────┐
       ▼     ▼     ▼
    Agent A  B     C
```

- 一个老板，N 个工人
- 老板决定下一步派谁干
- 适合：任务清晰可拆解、需要全局视角
- 例子：客服系统（路由到「订单」「售后」「技术支持」）

### 2.2 Hierarchical（树形）

```
       CEO
      /   \
   Mgr1   Mgr2
   / \    / \
  A   B  C   D
```

- 多层 Supervisor
- 适合：复杂大任务（比如"做一份完整的商业计划书"）
- 例子：CrewAI 的 hierarchical process、AutoGen 的嵌套群聊

### 2.3 Network / Swarm（点对点）

```
   A ──── B
   │  ╲ ╱ │
   │  ╳   │
   │  ╱ ╲ │
   D ──── C
```

- 任意 Agent 可以叫任意 Agent
- 没有固定流程，谁的能力对就谁上
- 适合：开放性、探索性任务
- 风险：容易死循环或 token 爆炸，必须有 max_steps 兜底

### 2.4 选哪个？

| 拓扑 | 适用 | 复杂度 |
| --- | --- | --- |
| Supervisor | 90% 业务场景 | 低 |
| Hierarchical | 大型任务、组织模拟 | 中 |
| Swarm | 创意、不确定路径 | 高 |

**新手建议**：90% 情况用 Supervisor，剩下 10% 再考虑别的。

## 3. CrewAI 入门

### 3.1 安装

```bash
pip install 'crewai[tools]'==0.80.0
export OPENAI_API_KEY=sk-xxx
```

### 3.2 四大核心概念

- **Agent**：扮演角色的 LLM（role / goal / backstory）
- **Task**：要完成的具体任务（description / expected_output / agent）
- **Crew**：把 Agent 和 Task 组合起来的"团队"
- **Process**：sequential（串行）/ hierarchical（带 manager）

### 3.3 完整代码：模拟内容工作室

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, WebsiteSearchTool

search = SerperDevTool()
web = WebsiteSearchTool()

planner = Agent(
    role="选题策划",
    goal="为科技博客挑选有传播力的选题",
    backstory="你做过 5 年新媒体主编，对热点敏锐",
    tools=[search],
    allow_delegation=False,
    verbose=True,
)

researcher = Agent(
    role="资料研究员",
    goal="为选题搜集 5 条以上权威资料",
    backstory="你是图书馆学博士，擅长挖一手信息",
    tools=[search, web],
    allow_delegation=False,
    verbose=True,
)

writer = Agent(
    role="科技作者",
    goal="写出 1500 字、有观点的中文文章",
    backstory="你是公众号 10w+ 作者，文风干练有梗",
    allow_delegation=False,
    verbose=True,
)

critic = Agent(
    role="主编",
    goal="对稿件提出修改意见，确保事实准确",
    backstory="你是前财经周刊主编，眼里揉不得沙子",
    allow_delegation=False,
    verbose=True,
)

t1 = Task(
    description="围绕 '2026 AI Agent 行业 3 个趋势' 给出 1 个标题 + 大纲",
    expected_output="一个抓眼球的标题 + 5 段大纲",
    agent=planner,
)
t2 = Task(
    description="基于大纲，搜集每个论点的 2-3 个数据 / 案例",
    expected_output="带 URL 引用的资料合集 markdown",
    agent=researcher,
    context=[t1],
)
t3 = Task(
    description="融合资料写一篇 1500 字中文文章，自然带入数据",
    expected_output="完整 markdown 文章",
    agent=writer,
    context=[t1, t2],
)
t4 = Task(
    description="校对文章：标题党？事实错误？逻辑断点？给修改清单",
    expected_output="修改清单 + 最终终稿",
    agent=critic,
    context=[t3],
)

crew = Crew(
    agents=[planner, researcher, writer, critic],
    tasks=[t1, t2, t3, t4],
    process=Process.sequential,  # 也可以 Process.hierarchical
    verbose=True,
)

result = crew.kickoff()
print(result)
```

### 3.4 sequential vs hierarchical

- `sequential`：按 task 顺序，谁的 context 就拿前面 task 的输出
- `hierarchical`：必须传 `manager_llm`，由一个 manager Agent 决定派谁干

```python
crew = Crew(
    agents=[researcher, writer, critic],
    tasks=[t_overall],
    process=Process.hierarchical,
    manager_llm="gpt-4o",
)
```

## 4. AutoGen v0.4 入门

### 4.1 背景

AutoGen 在 2025 年初做了重写，从同步 API 改为异步 + actor 模型，老代码（v0.2）大幅不兼容。本章基于 v0.4。

```bash
pip install "autogen-agentchat>=0.4" "autogen-ext[openai]>=0.4"
```

### 4.2 核心概念

- **AssistantAgent**：能调工具的 LLM Agent
- **UserProxyAgent**：代表"人"，可执行代码
- **GroupChat / RoundRobinGroupChat / SelectorGroupChat**：群聊容器
- **Termination**：停止条件（如关键词出现 / 轮数到）

### 4.3 完整代码：模拟产品需求评审会

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination
from autogen_ext.models.openai import OpenAIChatCompletionClient

model = OpenAIChatCompletionClient(model="gpt-4o-mini")

pm = AssistantAgent(
    name="PM",
    model_client=model,
    system_message="你是产品经理，只关注用户价值和优先级。",
)
designer = AssistantAgent(
    name="Designer",
    model_client=model,
    system_message="你是 UX 设计师，关心交互流程与可用性。",
)
engineer = AssistantAgent(
    name="Engineer",
    model_client=model,
    system_message="你是后端工程师，提技术可行性、成本和风险。",
)
qa = AssistantAgent(
    name="QA",
    model_client=model,
    system_message="你是测试工程师，找漏洞、边缘 case。最后说 APPROVED 表示通过。",
)

term = TextMentionTermination("APPROVED") | MaxMessageTermination(20)

team = SelectorGroupChat(
    participants=[pm, designer, engineer, qa],
    model_client=model,
    termination_condition=term,
)

async def main():
    await team.run_stream(task="评审需求：给我们 SaaS 产品加一个 AI 周报功能")

asyncio.run(main())
```

### 4.4 Code Execution Agent

AutoGen 自带本地 / Docker 沙箱执行：

```python
from autogen_ext.code_executors.docker import DockerCommandLineCodeExecutor
executor = DockerCommandLineCodeExecutor(image="python:3.12-slim")
```

让 Engineer Agent 真的把代码跑起来再回答可行性。

## 5. LangGraph 多 Agent 模式

### 5.1 Supervisor 模式

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Literal

class S(TypedDict):
    messages: list
    next: str

def supervisor(s: S):
    # 让 LLM 决定下一个 worker
    decision = call_llm_route(s["messages"])  # 返回 "researcher" / "writer" / "FINISH"
    return {"next": decision}

def researcher(s): ...
def writer(s): ...

g = StateGraph(S)
g.add_node("supervisor", supervisor)
g.add_node("researcher", researcher)
g.add_node("writer", writer)
g.set_entry_point("supervisor")
g.add_conditional_edges(
    "supervisor",
    lambda s: s["next"],
    {"researcher": "researcher", "writer": "writer", "FINISH": END},
)
g.add_edge("researcher", "supervisor")
g.add_edge("writer", "supervisor")
app = g.compile()
```

### 5.2 Hierarchical（嵌套子图）

把每个团队做成一个子图，再在外层 graph 中作为节点 `g.add_node("research_team", research_team_app)`。

### 5.3 Swarm（共享 state + handoff）

LangGraph 0.2+ 提供 `langgraph-swarm`，每个 Agent 有 `handoff` 工具，可显式把控制权交给别的 Agent。

```python
from langgraph_swarm import create_swarm, create_handoff_tool
to_engineer = create_handoff_tool(agent_name="engineer")
```

## 6. 三框架对比

| 维度 | CrewAI | AutoGen v0.4 | LangGraph |
| --- | --- | --- | --- |
| 心智模型 | 角色化任务流 | 群聊对话 | 显式状态图 |
| 易用性 | 高 | 中 | 中（需懂图） |
| 灵活度 | 中 | 高 | 极高 |
| 持久化 | 中（自带） | 需自己加 | 强（Checkpointer） |
| 流式 | 一般 | 强 | 强 |
| 适合场景 | 流水线 | 协作讨论 | 生产级复杂工作流 |
| 学习曲线 | 1 天上手 | 3 天 | 5 天 |

## 7. 多 Agent 反模式

不该用多 Agent 的信号：

1. 任务能用一个 prompt + 几个工具搞定
2. 上下文 < 8K，单 Agent 完全装得下
3. 实时延迟敏感（< 2 秒响应）
4. 没有评测机制（多 Agent 出错你都不知道哪步错的）
5. 团队还没把单 Agent 玩熟

**Anthropic 官方说过**："多 Agent 是 LLM 工程的最后手段，不是默认选择。"

## 8. Agent 间通信协议

### 8.1 消息格式（推荐）

```json
{
  "from": "researcher",
  "to": "writer",
  "type": "result",
  "payload": {"summary": "...", "sources": [...]},
  "trace_id": "abc-123"
}
```

### 8.2 共享 state vs 消息传递

- 共享 state（LangGraph 风格）：所有 Agent 读写同一个 dict，方便但容易冲突
- 消息传递（AutoGen 风格）：显式发消息，更解耦但啰嗦

实战建议：核心数据放共享 state，临时讨论用消息。

## 9. 实战：AI 投资研究小组

任务：给一只股票（如 NVDA）出投研报告。

四个 Agent：

1. **行业分析师**：搜行业趋势、竞争格局
2. **财报阅读员**：解析最新季报关键指标
3. **风险分析师**：找潜在风险（监管 / 技术 / 市场）
4. **投资顾问**：综合给出买 / 持有 / 卖

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, ScrapeWebsiteTool
search, scrape = SerperDevTool(), ScrapeWebsiteTool()

industry = Agent(role="行业分析师", goal="给出 {ticker} 所在行业 5 年趋势",
                 backstory="麦肯锡 TMT 出身", tools=[search, scrape], verbose=True)
finance = Agent(role="财报分析师", goal="解析 {ticker} 最新季报",
                backstory="CFA 三级，擅长拆解利润表", tools=[search, scrape], verbose=True)
risk = Agent(role="风险分析师", goal="列 {ticker} 三大风险",
             backstory="前央行金融稳定局", tools=[search, scrape], verbose=True)
advisor = Agent(role="投资顾问", goal="给出明确的 BUY/HOLD/SELL 建议",
                backstory="20 年公募基金经理", verbose=True)

t1 = Task(description="分析 {ticker} 所在行业的趋势与对手", agent=industry,
          expected_output="行业 5 年趋势 + 主要对手 SWOT")
t2 = Task(description="解析 {ticker} 最新一季度的营收/利润/现金流", agent=finance,
          expected_output="关键财务指标表 + 同比环比")
t3 = Task(description="找 {ticker} 三大风险并量化影响", agent=risk,
          expected_output="风险清单 + 概率/影响打分")
t4 = Task(description="综合以上输出投资建议", agent=advisor,
          expected_output="BUY/HOLD/SELL + 一页报告", context=[t1, t2, t3])

crew = Crew(agents=[industry, finance, risk, advisor],
            tasks=[t1, t2, t3, t4], process=Process.sequential, verbose=True)
print(crew.kickoff(inputs={"ticker": "NVDA"}))
```

跑完看产物，重点观察：

- 各 Agent 是否守在自己角色（risk 别去写买卖建议）
- 引用是否真实存在（要主动核查）
- token 总量（这种任务 1 次约 8-15k token）

## 10. 练习

1. 把上面投资小组改成 LangGraph Supervisor 实现，并对比 token 消耗
2. 为投资小组加一个"用户提问"环节（人类可以打断追问）
3. 用 AutoGen 重写"内容工作室"，让 critic 和 writer 真的"吵架"，观察是否质量更高
4. 设计一个故意会失败的任务（比如让 risk 给买卖建议），看你的 system prompt 是否能稳住角色
5. 给整个 Crew 加上日志，输出每个 Agent 的 token / 耗时

## 11. checklist

- [ ] 能画出三种多 Agent 拓扑并各举一例
- [ ] 用 CrewAI 跑通了内容工作室
- [ ] 用 AutoGen 跑通了产品评审会
- [ ] 用 LangGraph 写过 Supervisor 模式
- [ ] 能说出多 Agent 反模式三条
- [ ] 完成投资研究小组实战并核对产出
- [ ] 估算了一次任务的 token 成本

下一周（W08），我们让 Agent 长出"手"——直接操作浏览器和电脑。
