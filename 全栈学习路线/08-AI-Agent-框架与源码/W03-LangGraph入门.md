---
chapter: 08
week: W03
title: LangGraph 入门
date: 2026-06-19
tags: [langgraph, langchain, state-machine, agent-framework]
prereq: [W02-手写Agent不用框架]
estimated_hours: 16
difficulty: ★★★★☆
---

# W03 · LangGraph 入门

> 上一周你手写了 mini-agent，知道**循环**长什么样。这一周用 LangGraph 把"循环"变成"图"。

---

## 一、LangChain vs LangGraph：什么关系？

| | LangChain | LangGraph |
|---|---|---|
| 定位 | LLM 调用链工具箱 | 状态机式 Agent 框架 |
| 抽象 | Chain / Runnable | Graph / Node / Edge |
| 适合 | 简单顺序流 | 循环、分支、多 Agent |
| 现状 | 老牌、API 偏冗杂 | LangChain 团队 2024 主推、事实标准 |

**结论：写 Agent 直接用 LangGraph，不用纠结 LangChain。**

---

## 二、为什么大家都选 LangGraph？

1. **状态机心智模型**：和真实 Agent 工作方式一致。
2. **持久化（Checkpointer）**：内建 SQLite/Postgres 存档，断点恢复。
3. **Human-in-the-loop**：内建 `interrupt()`，比手写优雅。
4. **可视化（LangGraph Studio）**：图能在 IDE 里实时看见。
5. **生态大**：Anthropic / OpenAI / 各厂自家 Agent 也用它的运行时。
6. **部署（LangGraph Cloud / Server）**：写完直接上线。

类比：**LangGraph 是 Agent 界的 React**——不是必需品，但是事实标准。

---

## 三、安装

```bash
pip install langgraph langchain-anthropic langchain-core
export ANTHROPIC_API_KEY=...
```

---

## 四、Hello Graph（最小例子）

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    text: str

def upper(state: State) -> State:
    return {"text": state["text"].upper()}

graph = StateGraph(State)
graph.add_node("upper", upper)
graph.add_edge(START, "upper")
graph.add_edge("upper", END)

app = graph.compile()
print(app.invoke({"text": "hello"}))   # {'text': 'HELLO'}
```

恭喜，**你写了第一个 LangGraph**。

---

## 五、核心三概念

### 1. State（状态）

**整张图共享的字典**，每个节点读它、写它。

```python
class State(TypedDict):
    messages: list
    user_name: str
    step_count: int
```

### 2. Node（节点）

**普通 Python 函数**：吃 state，吐 state 的 *部分更新*。

```python
def my_node(state: State) -> dict:
    return {"step_count": state["step_count"] + 1}
```

返回的字典会**合并**进 state。

### 3. Edge（边）

- `add_edge(A, B)`：A 完成后必走 B
- `add_conditional_edges(A, fn)`：fn 看 state 决定下一步
- `START` / `END`：内置起止节点

---

## 六、6 步构建 StateGraph

```python
# 1. 定义 State
class State(TypedDict): ...

# 2. 创建 graph
graph = StateGraph(State)

# 3. 加节点
graph.add_node("plan", plan_fn)
graph.add_node("act", act_fn)

# 4. 加边
graph.add_edge(START, "plan")
graph.add_conditional_edges("plan", should_act,
                             {"yes": "act", "no": END})
graph.add_edge("act", "plan")  # 循环

# 5. 编译
app = graph.compile()

# 6. 调用
app.invoke({...})
# 或流式
for chunk in app.stream({...}):
    print(chunk)
```

---

## 七、实例 1：多节点顺序

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    topic: str
    outline: str
    article: str

def make_outline(s):
    return {"outline": f"## {s['topic']}\n1. 引言\n2. 主体\n3. 结论"}

def write_article(s):
    return {"article": f"基于大纲写文章：\n{s['outline']}\n（正文略）"}

g = StateGraph(State)
g.add_node("outline", make_outline)
g.add_node("write", write_article)
g.add_edge(START, "outline")
g.add_edge("outline", "write")
g.add_edge("write", END)
app = g.compile()

print(app.invoke({"topic": "LangGraph 入门"}))
```

---

## 八、实例 2：条件分支

```python
class State(TypedDict):
    msg: str
    label: str

def classify(s):
    if "退款" in s["msg"]: return {"label": "refund"}
    if "技术" in s["msg"]: return {"label": "tech"}
    return {"label": "faq"}

def refund(s): return {"msg": "已转给退款专员"}
def tech(s): return {"msg": "已转给技术专员"}
def faq(s): return {"msg": "FAQ: 请查官网"}

def route(s): return s["label"]   # 返回字符串决定走哪条

g = StateGraph(State)
g.add_node("classify", classify)
g.add_node("refund", refund); g.add_node("tech", tech); g.add_node("faq", faq)

g.add_edge(START, "classify")
g.add_conditional_edges("classify", route,
                         {"refund": "refund", "tech": "tech", "faq": "faq"})
for n in ["refund", "tech", "faq"]:
    g.add_edge(n, END)
app = g.compile()
print(app.invoke({"msg": "我要退款", "label": ""}))
```

---

## 九、实例 3：循环（条件边回到自己）

```python
class State(TypedDict):
    n: int
    log: list

def step(s):
    return {"n": s["n"] + 1, "log": s["log"] + [s["n"]]}

def cont(s):
    return "loop" if s["n"] < 5 else "done"

g = StateGraph(State)
g.add_node("step", step)
g.add_edge(START, "step")
g.add_conditional_edges("step", cont, {"loop": "step", "done": END})
app = g.compile()
print(app.invoke({"n": 0, "log": []}))
# {'n': 5, 'log': [0,1,2,3,4]}
```

---

## 十、Reducer：状态合并策略

默认 state 字段是**覆盖**。但有些字段（如 `messages`）你希望 **追加**。

```python
from typing import Annotated
from operator import add
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]   # 追加而不是覆盖
    counter: Annotated[int, add]              # 累加
    log: Annotated[list, add]                 # 列表拼接
```

`Annotated[T, reducer]` 告诉 LangGraph："这个字段更新时用 reducer 合并"。

`add_messages` 还会**自动按 id 去重**，是消息列表标配。

---

## 十一、ReAct Agent 模板

LangGraph 提供开箱即用的 ReAct Agent：

```python
from langgraph.prebuilt import create_react_agent
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """查天气"""
    return f"{city} 24度晴"

@tool
def calc(expr: str) -> str:
    """计算"""
    return str(eval(expr))

llm = ChatAnthropic(model="claude-sonnet-4-5")
agent = create_react_agent(llm, tools=[get_weather, calc])

result = agent.invoke({"messages": [{"role": "user",
                                       "content": "北京天气如何？1+2 等于几？"}]})
for m in result["messages"]:
    print(m.type, ":", m.content)
```

**3 行配工具，Agent 就有了**。这是日常最常用的捷径。

---

## 十二、ToolNode：把工具列表变节点

如果你想**自己画图**而不是用 prebuilt：

```python
from langgraph.prebuilt import ToolNode
from langgraph.graph import StateGraph, MessagesState, START, END

tools = [get_weather, calc]
tool_node = ToolNode(tools)

llm_with_tools = ChatAnthropic(model="claude-sonnet-4-5").bind_tools(tools)

def call_llm(state):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

def should_continue(state):
    last = state["messages"][-1]
    return "tools" if last.tool_calls else END

g = StateGraph(MessagesState)
g.add_node("llm", call_llm)
g.add_node("tools", tool_node)
g.add_edge(START, "llm")
g.add_conditional_edges("llm", should_continue, {"tools": "tools", END: END})
g.add_edge("tools", "llm")
agent = g.compile()
```

**这就是 ReAct 的内部实现**。看懂这张图，你就懂了 90% 的 LangGraph。

---

## 十三、本周实战：用 LangGraph 重写 W01 的 5 模式

写 5 个 .py 文件，每个一张图：

| 模式 | 图结构提示 |
|---|---|
| Prompt Chaining | `START → outline → write → polish → END` |
| Routing | `START → classify → 条件边 → faq/tech/refund → END` |
| Parallelization | START 后三条边并行到三个翻译节点，再汇合到 merge |
| Orchestrator-Workers | orchestrator 用 `Send` API 派 N 个 worker，再 reducer 汇合（W04 详细讲）|
| Evaluator-Optimizer | `gen → eval`，eval 条件边：PASS→END，FAIL→gen |

跑通后对比 W01 纯 SDK 版本的代码量和清晰度。

---

## 十四、checklist

- [ ] Hello Graph 能跑
- [ ] 能讲清 State / Node / Edge
- [ ] 条件边、循环都写过
- [ ] 知道 reducer 怎么用，`add_messages` 用过
- [ ] `create_react_agent` 跑通
- [ ] 自己画的 ToolNode + LLM 图跑通
- [ ] 5 模式 LangGraph 版全部跑通

---

## 十五、参考

- LangGraph 官方文档 (langchain-ai.github.io/langgraph)
- LangGraph 教程 · *Build a chatbot*
- LangGraph 源码 · `prebuilt/chat_agent_executor.py`
- 本路线 W01 / W02

下一周：**子图、Send、Checkpointer、人工审批、Streaming、Studio**——LangGraph 进阶全家桶。
