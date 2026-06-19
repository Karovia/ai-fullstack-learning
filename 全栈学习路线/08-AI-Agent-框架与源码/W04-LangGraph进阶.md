---
chapter: 08
week: W04
title: LangGraph 进阶
date: 2026-06-19
tags: [langgraph, subgraph, checkpointer, human-in-the-loop, streaming, studio]
prereq: [W03-LangGraph入门]
estimated_hours: 18
difficulty: ★★★★★
---

# W04 · LangGraph 进阶

> 入门会画图，进阶才能上线。本周覆盖**子图、并行、Send、Checkpointer、人工审批、Streaming、Studio**，并落地一个完整的客服 Agent。

---

## 一、子图（Subgraph）：把图当节点

复杂 Agent 经常嵌套：主图里某一步本身就是另一张图。

```python
# 子图：研究小工
class ResearchState(TypedDict):
    question: str
    answer: str

rg = StateGraph(ResearchState)
rg.add_node("search", search_node)
rg.add_node("summarize", summarize_node)
rg.add_edge(START, "search")
rg.add_edge("search", "summarize")
rg.add_edge("summarize", END)
research_subgraph = rg.compile()

# 主图直接把子图当节点用
class MainState(TypedDict):
    question: str
    answer: str
    final: str

mg = StateGraph(MainState)
mg.add_node("research", research_subgraph)   # 子图作节点
mg.add_node("polish", polish_node)
mg.add_edge(START, "research")
mg.add_edge("research", "polish")
mg.add_edge("polish", END)
main_app = mg.compile()
```

**状态自动按字段名映射**——主图和子图共有的字段会传递。如果字段名不同，用 `add_node("research", research_subgraph, input_keys=..., output_keys=...)` 显式映射。

---

## 二、并行节点（fan-out / fan-in）

只要从一个节点连出**多条边到不同节点**，它们就并行执行。

```python
g.add_edge("split", "translate_en")
g.add_edge("split", "translate_jp")
g.add_edge("split", "translate_es")
# 三条边后再汇合
g.add_edge("translate_en", "merge")
g.add_edge("translate_jp", "merge")
g.add_edge("translate_es", "merge")
```

`merge` 节点只会**等三个都跑完**才执行（同步屏障）。

---

## 三、Send API：动态 fan-out

并行边数**写死**。如果运行时才知道要派多少个 worker，用 `Send`：

```python
from langgraph.types import Send

def orchestrator(state):
    # LLM 拆任务返回 N 个子任务，N 不定
    subtasks = decompose(state["goal"])
    # 关键：返回一个 list[Send]
    return [Send("worker", {"task": t}) for t in subtasks]

g.add_node("orchestrator", orchestrator)
g.add_node("worker", worker_node)
g.add_node("merge", merge_node)
g.add_edge(START, "orchestrator")
g.add_conditional_edges("orchestrator", lambda s: "merge")  # 也可省
g.add_edge("worker", "merge")  # 所有 worker 都汇到 merge
g.add_edge("merge", END)
```

`Send(node_name, partial_state)` 表示"派一个 worker 实例，初始 state 是这个"。**这是 Orchestrator-Workers 模式的官方实现**。

---

## 四、Checkpointer：持久化与时间旅行

默认 Graph 跑完 state 就丢。加 Checkpointer 后**每一步都存盘**，能恢复、能回放。

### InMemory（开发用）

```python
from langgraph.checkpoint.memory import MemorySaver
app = graph.compile(checkpointer=MemorySaver())

config = {"configurable": {"thread_id": "user-42"}}
app.invoke({"messages": [...]}, config=config)
# 同一 thread_id 再 invoke，会接着上次跑
```

### SQLite（本地持久化）

```python
from langgraph.checkpoint.sqlite import SqliteSaver
import sqlite3

saver = SqliteSaver(sqlite3.connect("agent.db", check_same_thread=False))
app = graph.compile(checkpointer=saver)
```

### Postgres（生产）

```python
from langgraph.checkpoint.postgres import PostgresSaver
saver = PostgresSaver.from_conn_string("postgresql://user:pwd@host/db")
saver.setup()  # 首次建表
app = graph.compile(checkpointer=saver)
```

### 时间旅行

```python
# 列出所有 checkpoint
for snap in app.get_state_history(config):
    print(snap.config["configurable"]["checkpoint_id"], snap.values)

# 回到任意一个
target = list(app.get_state_history(config))[3]
app.invoke(None, config=target.config)  # 从那一步重跑
```

**这是 Agent 调试神器**——bug 不用复现，直接回到出问题前一步重跑。

---

## 五、Human-in-the-Loop：interrupt

危险操作前暂停等人：

```python
from langgraph.types import interrupt, Command

def refund_node(state):
    # 暂停，把待确认信息抛给上层
    decision = interrupt({"action": "refund",
                           "amount": state["amount"]})
    if decision == "approve":
        return {"result": "已退款"}
    return {"result": "已拒绝"}

g.add_node("refund", refund_node)
# ...
app = g.compile(checkpointer=MemorySaver())

# 跑到 interrupt 处暂停
config = {"configurable": {"thread_id": "t1"}}
result = app.invoke({"amount": 100}, config=config)
print(result)  # 看到 __interrupt__ 字段

# 拿到用户决定后恢复
app.invoke(Command(resume="approve"), config=config)
```

**没有 Checkpointer 就没有 interrupt**——状态必须存得下才能暂停。

---

## 六、Streaming：4 种模式

```python
for chunk in app.stream(input, config=cfg, stream_mode="values"):
    print(chunk)
```

| stream_mode | 输出 | 用途 |
|---|---|---|
| `values` | 每步后**完整 state** | 看整体演变 |
| `updates` | 每步**只输出新增字段** | 节省带宽 |
| `messages` | LLM **token 级**流式 + 工具调用 | 给前端打字效果 |
| `debug` | 详细执行事件（节点开始/结束/错误） | 调试 |

**多模式同时**：

```python
for kind, chunk in app.stream(input, config=cfg,
                                stream_mode=["updates", "messages"]):
    print(kind, chunk)
```

---

## 七、LangGraph Studio：可视化调试

**Studio 是 LangGraph 团队做的桌面 / Web 应用**，把你的图渲染成可点击的流程图：

- 看每一步的 state 变化
- 改 state 后重跑
- 时间旅行（点任意 checkpoint）
- 直接编辑节点 prompt 试错

启动方式（写一个 `langgraph.json` 后）：

```bash
pip install langgraph-cli
langgraph dev   # 本地起 Studio
```

`langgraph.json` 例子：

```json
{
  "dependencies": ["."],
  "graphs": { "agent": "./my_agent.py:app" },
  "env": ".env"
}
```

---

## 八、LangGraph Cloud / Server（部署，简介）

写完的 Graph 可以一键部署：

- **LangGraph Server**：自托管的 FastAPI，含线程/checkpoint/调度
- **LangGraph Cloud**：LangChain 官方托管版

部署后自动暴露 REST：

```
POST /threads/{tid}/runs       # 跑一次
POST /threads/{tid}/runs/stream # 流式跑
GET  /threads/{tid}/state       # 看状态
```

**这一段了解即可**——本周不需要部署。

---

## 九、本周实战：客服 Agent

把上面所有特性串起来：

### 需求

- 用户来消息 → Agent 识别意图 → 走不同分支
- 分支：FAQ / 工具调用 / 人工接管
- 接 RAG 知识库（W06 学的，先 mock）
- 工具：查订单 / 退款 / 升级 VIP
- 退款 > 500 元需要**人工审批**
- 对话历史**持久化**到 SQLite，下次接着聊

### 状态设计

```python
from typing import Annotated, TypedDict
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]
    intent: str          # faq/tool/human
    user_id: str
    pending_refund: dict # {"order_id": ..., "amount": ...}
```

### 图结构

```
START → classify → conditional:
                   ├─ "faq"   → rag_answer → END
                   ├─ "tool"  → tool_loop  → END
                   └─ "human" → handoff_human → END
```

`tool_loop` 是子图（ReAct 模式），里面挂三个工具：

```python
@tool
def get_order(order_id: str) -> str:
    return f"订单 {order_id}: 已发货"

@tool
def refund(order_id: str, amount: float) -> str:
    if amount > 500:
        decision = interrupt({"action": "refund_high",
                                "order_id": order_id,
                                "amount": amount})
        if decision != "approve":
            return "用户拒绝退款"
    return f"已退款 {amount} 元"

@tool
def upgrade_vip(user_id: str) -> str:
    return f"{user_id} 已升级 VIP"
```

### 编译

```python
saver = SqliteSaver(sqlite3.connect("cs.db", check_same_thread=False))
app = graph.compile(checkpointer=saver)
```

### 调用

```python
config = {"configurable": {"thread_id": "user-42"}}

# 第一轮
for c in app.stream(
    {"messages": [{"role": "user", "content": "我想退款 800 元"}],
     "user_id": "u42"},
    config=config, stream_mode="updates"
):
    print(c)

# 触发 interrupt 后
app.invoke(Command(resume="approve"), config=config)
```

### 用 Studio 调试

`langgraph dev` 启动后，浏览器里点开图，能看到：
- classify 节点为啥判成 "tool"
- refund 节点 interrupt 时的 state
- 时间旅行回到 classify 改 prompt 重跑

---

## 十、checklist

- [ ] 子图嵌套跑通
- [ ] Send API 派动态 worker 数
- [ ] Checkpointer + 时间旅行实操过一次
- [ ] interrupt + Command(resume=...) 走通
- [ ] 4 种 stream_mode 都见过输出
- [ ] LangGraph Studio 在本地起来过
- [ ] 客服 Agent 可以连续对话、能持久化、能审批

---

## 十一、参考

- LangGraph 文档 · *Subgraphs / Send / Persistence / Human-in-the-loop*
- LangGraph 文档 · *Streaming*
- LangGraph Studio 官方教程
- LangGraph 开源仓库 · `examples/customer_support`

下一周：**MCP 协议**——比 LangGraph 更底层、跨厂商的工具/资源标准。
