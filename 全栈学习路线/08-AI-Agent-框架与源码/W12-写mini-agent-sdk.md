---
title: W12 写 mini-agent-sdk + 章节收尾
chapter: 08-AI-Agent-框架与源码
week: 12
date: 2026-06-19
tags: [agent, sdk, project, capstone, opensource]
prev: "[[W11-LangGraph源码导读]]"
next: "[[../09-毕业项目/W01-毕业项目启动]]"
---

# W12 写 mini-agent-sdk + 章节收尾

## 0. 一句话开场

学了八周，到了"自己做一个"的时候。这一周我们用不到 500 行代码，写一个能用、能扩、能开源的 mini-agent-sdk，把前面所学全部串起来——这是你简历上最硬的一笔。

## 1. 设计目标

打造一个 SDK，至少满足：

1. `@tool` 装饰器：自动从函数签名生成 JSON Schema
2. `Agent(name, system, tools, model)`：声明式定义 Agent
3. 同步 / 流式执行循环
4. 多 Agent **handoff**（Agent A 转交给 Agent B）
5. **持久化**：基于 SQLite 的 checkpoint，可恢复
6. **trace**：每步记录到 JSON / SQLite / Langfuse
7. **人工审批 hook**：危险工具调用前 callback

非目标（避免功能蔓延）：

- 不重写 LangChain
- 不做模型路由（直接调 OpenAI 兼容 API）
- 不做 RAG
- 不做训练

## 2. 项目结构

```
mini_agent/
├── pyproject.toml
├── README.md
├── mini_agent/
│   ├── __init__.py
│   ├── agent.py        # 核心 Agent 类 + 执行循环
│   ├── tool.py         # @tool 装饰器
│   ├── handoff.py      # Agent 间转交
│   ├── checkpoint.py   # SQLite 持久化
│   ├── trace.py        # 简易 trace
│   ├── approval.py     # 人工审批 hook
│   └── llm.py          # OpenAI 兼容 client
├── tests/
│   └── test_agent.py
└── examples/
    └── customer_service.py
```

## 3. 完整代码（约 480 行）

下面给出每个模块，逐段讲解。

### 3.1 tool.py（约 60 行）

```python
import inspect, json
from typing import Callable, get_type_hints

def tool(fn: Callable):
    """装饰器：从函数签名自动生成 JSON Schema。"""
    sig = inspect.signature(fn)
    hints = get_type_hints(fn)
    props, required = {}, []
    type_map = {str: "string", int: "integer", float: "number",
                bool: "boolean", list: "array", dict: "object"}
    for name, p in sig.parameters.items():
        t = hints.get(name, str)
        props[name] = {"type": type_map.get(t, "string")}
        if p.default is inspect.Parameter.empty:
            required.append(name)
    fn._tool_schema = {
        "type": "function",
        "function": {
            "name": fn.__name__,
            "description": (fn.__doc__ or "").strip(),
            "parameters": {"type": "object", "properties": props, "required": required},
        },
    }
    fn._is_tool = True
    fn._dangerous = getattr(fn, "_dangerous", False)
    return fn

def dangerous(fn):
    """标记为危险工具，调用前会触发审批 hook。"""
    fn._dangerous = True
    return fn

class ToolRegistry:
    def __init__(self, tools=None):
        self.tools = {t.__name__: t for t in (tools or [])}
    def schemas(self):
        return [t._tool_schema for t in self.tools.values()]
    def call(self, name, args):
        if name not in self.tools:
            return {"error": f"unknown tool {name}"}
        try:
            return {"result": self.tools[name](**args)}
        except Exception as e:
            return {"error": str(e)}
```

讲解：

- 把工具元数据挂在函数对象上，避免引入额外的 class
- `dangerous` 是一个标记，配合 `approval.py` 的 hook
- ToolRegistry 把工具集中管理，便于多 Agent 共享

### 3.2 llm.py（约 30 行）

```python
import os
from openai import OpenAI

def make_client():
    return OpenAI(
        api_key=os.environ.get("OPENAI_API_KEY"),
        base_url=os.environ.get("OPENAI_BASE_URL"),
    )

def chat(client, model, messages, tools=None, stream=False):
    return client.chat.completions.create(
        model=model, messages=messages,
        tools=tools or None, tool_choice="auto" if tools else None,
        stream=stream,
    )
```

讲解：故意做薄，所有"模型差异"都靠 `OPENAI_BASE_URL` 切（兼容 DeepSeek / Moonshot / 自部署 vLLM）。

### 3.3 trace.py（约 60 行）

```python
import json, time, uuid, sqlite3, contextvars
from pathlib import Path

_current = contextvars.ContextVar("trace", default=None)

class Trace:
    def __init__(self, name, parent=None, attrs=None):
        self.id = str(uuid.uuid4())
        self.parent_id = parent.id if parent else None
        self.name, self.attrs = name, attrs or {}
        self.start = time.time(); self.end = None
        self.children, self.events = [], []
    def event(self, name, **kw):
        self.events.append({"t": time.time(), "name": name, **kw})
    def __enter__(self):
        parent = _current.get()
        if parent: parent.children.append(self)
        self.tok = _current.set(self); return self
    def __exit__(self, *exc):
        self.end = time.time(); _current.reset(self.tok)
        if self.parent_id is None:
            _writers.append(self.to_dict())
    def to_dict(self):
        return {"id": self.id, "parent": self.parent_id, "name": self.name,
                "ms": (self.end - self.start) * 1000, "attrs": self.attrs,
                "events": self.events, "children": [c.to_dict() for c in self.children]}

_writers = []  # 简单内存缓冲

def dump_traces(path="traces.jsonl"):
    p = Path(path)
    with p.open("a") as f:
        for t in _writers:
            f.write(json.dumps(t, ensure_ascii=False) + "\n")
    _writers.clear()

def current_trace():
    return _current.get()
```

讲解：复刻 W10 的迷你 OTEL，足以喂给 Langfuse / 自家面板。

### 3.4 checkpoint.py（约 60 行）

```python
import json, sqlite3, time
from pathlib import Path

DDL = """
CREATE TABLE IF NOT EXISTS checkpoint (
  thread_id TEXT, ts TEXT, parent_ts TEXT,
  state TEXT, PRIMARY KEY (thread_id, ts));
"""

class SqliteCheckpoint:
    def __init__(self, path="agent.db"):
        self.con = sqlite3.connect(path); self.con.execute(DDL); self.con.commit()
    def put(self, thread_id, state, parent_ts=None):
        ts = f"{time.time():.6f}"
        self.con.execute("INSERT INTO checkpoint VALUES (?,?,?,?)",
                         (thread_id, ts, parent_ts, json.dumps(state, ensure_ascii=False)))
        self.con.commit(); return ts
    def latest(self, thread_id):
        cur = self.con.execute(
            "SELECT ts, state FROM checkpoint WHERE thread_id=? ORDER BY ts DESC LIMIT 1",
            (thread_id,))
        row = cur.fetchone()
        return (row[0], json.loads(row[1])) if row else (None, None)
    def history(self, thread_id):
        return [(ts, json.loads(s)) for ts, s in self.con.execute(
            "SELECT ts, state FROM checkpoint WHERE thread_id=? ORDER BY ts", (thread_id,))]
```

讲解：极简版 LangGraph checkpointer。一条记录 = 一次 superstep 后的全量状态。生产环境建议增量写入。

### 3.5 approval.py（约 30 行）

```python
from typing import Callable

class ApprovalHook:
    """危险工具调用前调用，返回 True 通过，False 拒绝。"""
    def __init__(self, fn: Callable[[str, dict], bool] = None):
        self.fn = fn or self.default
    def default(self, tool_name, args):
        ans = input(f"[approval] 调用 {tool_name}({args})? (y/N): ")
        return ans.strip().lower() == "y"
    def __call__(self, name, args):
        return self.fn(name, args)
```

讲解：默认走命令行 yN，可注入自定义函数（飞书机器人 / 钉钉 webhook）。

### 3.6 handoff.py（约 30 行）

```python
from .tool import tool

def make_handoff_tool(target_agent_name: str):
    @tool
    def handoff(reason: str = ""):
        """把对话转交给指定 Agent。"""
        return {"__handoff__": target_agent_name, "reason": reason}
    handoff.__name__ = f"handoff_to_{target_agent_name}"
    handoff._tool_schema["function"]["name"] = handoff.__name__
    return handoff
```

讲解：handoff 不是真正"调函数"，而是返回一个特殊 dict，执行循环识别后切换 active agent。

### 3.7 agent.py（约 180 行，核心）

```python
import json, uuid
from .tool import ToolRegistry
from .llm import make_client, chat
from .trace import Trace, current_trace
from .approval import ApprovalHook
from .checkpoint import SqliteCheckpoint

class Agent:
    def __init__(self, name, system, tools=None, model="gpt-4o-mini"):
        self.name, self.system, self.model = name, system, model
        self.registry = ToolRegistry(tools)
    def schema(self):
        return self.registry.schemas()

class Runtime:
    def __init__(self, agents, checkpointer: SqliteCheckpoint = None,
                 approval: ApprovalHook = None, max_steps=20):
        self.agents = {a.name: a for a in agents}
        self.client = make_client()
        self.checkpointer = checkpointer
        self.approval = approval
        self.max_steps = max_steps

    def _maybe_approve(self, agent, name, args):
        fn = agent.registry.tools.get(name)
        if fn and getattr(fn, "_dangerous", False) and self.approval:
            if not self.approval(name, args):
                return {"error": "user rejected"}
        return None

    def run(self, thread_id: str, user_input: str, start_agent: str):
        ts, state = (None, None)
        if self.checkpointer:
            ts, state = self.checkpointer.latest(thread_id)
        if state is None:
            state = {"active": start_agent, "messages": []}
        agent = self.agents[state["active"]]
        state["messages"].append({"role": "user", "content": user_input})

        with Trace("agent.run", attrs={"thread": thread_id, "agent": agent.name}) as root:
            for step in range(self.max_steps):
                with Trace(f"step.{step}", attrs={"agent": agent.name}) as st:
                    msgs = [{"role": "system", "content": agent.system}] + state["messages"]
                    resp = chat(self.client, agent.model, msgs, tools=agent.schema())
                    msg = resp.choices[0].message
                    st.event("llm", model=agent.model,
                             tokens=getattr(resp.usage, "total_tokens", 0))
                    if not msg.tool_calls:
                        state["messages"].append({"role": "assistant", "content": msg.content})
                        if self.checkpointer:
                            self.checkpointer.put(thread_id, state, parent_ts=ts)
                        return msg.content
                    # 有 tool 调用
                    state["messages"].append(msg.model_dump(exclude_none=True))
                    for tc in msg.tool_calls:
                        name = tc.function.name
                        args = json.loads(tc.function.arguments or "{}")
                        # handoff 拦截
                        if name.startswith("handoff_to_"):
                            target = name.replace("handoff_to_", "")
                            state["active"] = target
                            state["messages"].append({
                                "role": "tool", "tool_call_id": tc.id,
                                "content": json.dumps({"handoff": target})})
                            agent = self.agents[target]
                            st.event("handoff", to=target)
                            break
                        # 审批
                        rej = self._maybe_approve(agent, name, args)
                        if rej:
                            state["messages"].append({"role": "tool",
                                "tool_call_id": tc.id, "content": json.dumps(rej)})
                            continue
                        out = agent.registry.call(name, args)
                        st.event("tool", name=name, ok="error" not in out)
                        state["messages"].append({"role": "tool",
                            "tool_call_id": tc.id, "content": json.dumps(out, ensure_ascii=False)})
                    if self.checkpointer:
                        ts = self.checkpointer.put(thread_id, state, parent_ts=ts)
            return "[max steps reached]"
```

讲解：

- 单循环既处理工具又处理 handoff，handoff 通过特殊命名识别
- 每步存一次 checkpoint，进程崩溃后 `latest` 即可恢复
- trace 用 `with` 嵌套，自然形成树
- max_steps 兜底，防死循环

### 3.8 \_\_init\_\_.py（约 10 行）

```python
from .agent import Agent, Runtime
from .tool import tool, dangerous, ToolRegistry
from .handoff import make_handoff_tool
from .checkpoint import SqliteCheckpoint
from .approval import ApprovalHook
from .trace import Trace, dump_traces, current_trace
```

### 3.9 examples/customer_service.py（约 50 行）

```python
from mini_agent import (Agent, Runtime, tool, dangerous,
                        make_handoff_tool, SqliteCheckpoint, ApprovalHook, dump_traces)

@tool
def search_order(order_id: str):
    "查订单"
    return {"order_id": order_id, "status": "shipped", "amount": 199}

@tool
@dangerous
def refund(order_id: str, amount: float):
    "退款（危险操作）"
    return {"order_id": order_id, "refunded": amount}

to_refund = make_handoff_tool("refund_agent")
to_general = make_handoff_tool("general_agent")

general = Agent(
    name="general_agent",
    system="你是客服总台，遇到退款相关请调用 handoff_to_refund_agent。",
    tools=[search_order, to_refund],
)
refund_agent = Agent(
    name="refund_agent",
    system="你是退款专员，确认订单后用 refund 工具操作，结束后用 handoff_to_general_agent 交回。",
    tools=[search_order, refund, to_general],
)

rt = Runtime([general, refund_agent],
             checkpointer=SqliteCheckpoint("demo.db"),
             approval=ApprovalHook())

if __name__ == "__main__":
    print(rt.run("t1", "我要退订单 A123 的款", start_agent="general_agent"))
    dump_traces()
```

跑一次，控制台会问你是否批准 refund，输入 y 后看到完整流程结束。

## 4. 测试（pytest）

`tests/test_agent.py`：

```python
from mini_agent import Agent, Runtime, tool

@tool
def add(a: int, b: int):
    "相加"
    return a + b

def test_basic_tool(monkeypatch):
    calls = []
    class FakeChat:
        def __init__(self): self.i = 0
        def __call__(self, *_, **__):
            self.i += 1
            if self.i == 1:
                return type("R", (), {"choices": [type("C", (), {
                    "message": type("M", (), {"content": None, "tool_calls": [
                        type("T", (), {"id": "1", "function": type("F", (), {
                            "name": "add", "arguments": '{"a":1,"b":2}'})})
                    ]})()})()], "usage": type("U", (), {"total_tokens": 10})})
            return type("R", (), {"choices": [type("C", (), {
                "message": type("M", (), {"content": "3", "tool_calls": []})()})()],
                "usage": type("U", (), {"total_tokens": 5})})

    from mini_agent import agent as agent_mod
    monkeypatch.setattr(agent_mod, "chat", FakeChat())
    rt = Runtime([Agent("calc", "你会算数", tools=[add])])
    out = rt.run("t", "1+2", start_agent="calc")
    assert "3" in out
```

跑 `pytest -q`，绿了你就成了。

## 5. 开源到 GitHub

```bash
cd mini_agent
git init && git add . && git commit -m "init"
gh repo create mini-agent --public --source=. --push
```

`README.md` 至少包含：

- 一句话定位（"< 500 行的可读 Agent SDK，够用即可"）
- 安装：`pip install -e .`
- 30 秒 demo（最小代码）
- 核心概念图（Agent / Tool / Handoff / Runtime）
- FAQ：和 LangGraph、AutoGen 的区别
- License: MIT

文档站建议：

- **Mintlify**（最美，免费）：写 mdx，五分钟一个站
- **Docusaurus**（Meta）：开源、可控
- **VitePress**：极简、纯 markdown

## 6. 第 08 章总结

### 6.1 你现在掌握了什么

- 写过单 Agent + 多 Agent + 浏览器 Agent
- 用过 CrewAI / AutoGen / LangGraph 三大主流
- 自建 eval、接 observability、读过 LangGraph 源码
- 自己造了一个 SDK 并开源

### 6.2 你和市场上 90% Agent 工程师的差距在哪

会跑 demo 的人不少，但你会发现工作里真正卡人的从来不是技术，而是：

- **产品视角**：用户真实路径是什么？哪一步会怎么放弃？什么时候不该用 AI？
- **业务理解**：能不能听完业务方 30 分钟描述，就拆出该用单 / 多 Agent，哪些可以纯规则
- **优化经验**：看到 trace 能秒定位"上下文太长 / 工具描述模糊 / 模型选型不当"
- **工程素养**：日志 / 监控 / 回滚 / 灰度，不靠 AI 也得会

补这块的方法：去做真实业务（哪怕是公司内部小工具），跑通"上线 → 收反馈 → 迭代"完整循环。技术之外的东西，只能在战场上学。

### 6.3 怎么继续提升

- 持续读论文：每周 1 篇 Agent 顶会（NeurIPS / ICML / ICLR / ACL workshop）
- 持续看源码：LangGraph / AutoGen / OpenAI Agents SDK / smolagents 轮流读
- 持续做评测：建立你自己的"Agent 健康度"看板
- 写作输出：博客 / 视频 / 公众号，逼自己讲清楚等于真的懂

## 7. 进入毕业项目（第 09 章）的准备清单

- [ ] 选题：选一个你**真的会自己用**的方向（不要想象别人会用什么）
- [ ] 用户：至少能找到 5 个真实用户（朋友、同事、自己）
- [ ] 数据：先收 50 条真实使用样本作为 eval 集
- [ ] 技术栈：固化你最熟的（建议 LangGraph + Langfuse + 自家 mini SDK 的混合）
- [ ] 部署：选好云（Railway / Fly.io / 自有服务器）
- [ ] 时间：留出至少 6 周专注开发
- [ ] 文档：从第一天起写 changelog
- [ ] 推广：上线后准备好 Twitter / 即刻 / 小红书的发文

## 8. 练习（章节级）

1. 给 mini-agent-sdk 加一个 `astream` 异步流式接口
2. 把审批 hook 改成飞书机器人审批
3. 把 SqliteCheckpoint 替换成 PostgresCheckpoint，验证多机部署
4. 加一个 `LangfuseTraceWriter`，把 trace 推到 Langfuse
5. 写一篇博客《我用 500 行写了一个 Agent SDK》并发到掘金 / 知乎

## 9. checklist

- [ ] mini-agent-sdk 完整跑通 + 测试通过
- [ ] 已 push 到 GitHub 并写 README
- [ ] 写了一篇配套博客
- [ ] 第 08 章笔记整理进 vault
- [ ] 第 09 章选题已定
- [ ] 至少 5 个真实用户排队等你交付

第 08 章到此结束。下一章是毕业项目——把所学的一切，凝结成你简历上最响亮的一行。

