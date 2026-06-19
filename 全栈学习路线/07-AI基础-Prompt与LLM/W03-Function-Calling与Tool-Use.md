---
chapter: 07-AI基础-Prompt与LLM
week: W03
title: Function Calling 与 Tool Use
duration: 7 天
prereq: W01 API、W02 Prompt 工程
goal: 让 LLM 不仅会说话，还会调用真实工具——查天气、算账、发邮件，并稳健处理失败、并行与依赖
tags: [tool-use, function-calling, anthropic, react]
updated: 2026-06-19
---

# W03 · Function Calling 与 Tool Use

> 这一周是**整章最重要的基础**：Agent 章节里几乎所有"自主行动"，都是 Tool Use 这个原语堆出来的。

---

## 0. 为什么 LLM 必须会用工具

LLM 是"读完就猜下一个词"的机器，它先天有几个弱点：

1. **不会算数**：`12345 × 6789` 它经常算错
2. **不知道今天日期**：知识截止某月
3. **拿不到实时数据**：天气、股价、订单状态
4. **不能写文件 / 发邮件 / 调 API**

工具 (tool) 就是一座桥：模型决定**调什么、传什么参数**，你的代码**真正去执行**，把结果告诉它，它再继续推理。

---

## 1. 类比：让一位顾问在客户面前操作 Excel

LLM = 顾问（嘴会说，手不能动）
工具 = 顾问能呼叫的助理（会查表、会算账、会发邮件）

顾问对你说"我需要查 6 月营收"，助理跑去查，回来递一张纸条，顾问继续讲下去。**关键：模型只输出"我要调谁、传什么"，真正执行的是你**。

---

## 2. Tool Use 原理（核心循环）

```
┌─────────────────────────────────────────────────┐
│ 1. 你给模型：messages + tools 定义              │
│ 2. 模型回：stop_reason="tool_use"，给出 tool_use 块 │
│ 3. 你的代码执行该工具，拿到结果                 │
│ 4. 你把 tool_result 块当作 user 消息追加进去    │
│ 5. 再调一次，模型可能再调工具或给最终答案       │
│ 6. 循环到 stop_reason="end_turn"                │
└─────────────────────────────────────────────────┘
```

**记住一句**：tool_use 是**预测下一个词的副作用**，模型并没"真的"调用，是它输出一段 JSON，由你执行。

---

## 3. Anthropic Tool Use 完整 API

### 3.1 工具 schema

```python
tools = [{
    "name": "get_weather",
    "description": "获取指定城市当前天气。仅在用户明确询问天气时使用。",
    "input_schema": {
        "type": "object",
        "properties": {
            "city": {"type": "string", "description": "中文城市名，如 北京"},
            "unit": {"type": "string", "enum": ["c", "f"], "default": "c"},
        },
        "required": ["city"],
    },
}]
```

### 3.2 第一次调用

```python
resp = client.messages.create(
    model="claude-sonnet-4-5", max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "北京今天多少度？"}],
)
print(resp.stop_reason)   # "tool_use"
for block in resp.content:
    print(block.type, getattr(block, "name", ""), getattr(block, "input", ""))
```

可能拿到：

```
tool_use  get_weather  {"city": "北京"}
```

### 3.3 回填 tool_result

```python
tool_use_block = next(b for b in resp.content if b.type == "tool_use")
result = real_get_weather(**tool_use_block.input)   # 你的真实函数

messages.append({"role": "assistant", "content": resp.content})
messages.append({"role": "user", "content": [{
    "type": "tool_result",
    "tool_use_id": tool_use_block.id,
    "content": str(result),
}]})

resp2 = client.messages.create(
    model="claude-sonnet-4-5", max_tokens=1024,
    tools=tools, messages=messages,
)
print(resp2.content[0].text)
```

### 3.4 通用循环

```python
def run(user_msg, tools, executors, max_steps=8):
    messages = [{"role": "user", "content": user_msg}]
    for _ in range(max_steps):
        resp = client.messages.create(
            model="claude-sonnet-4-5", max_tokens=1024,
            tools=tools, messages=messages)
        messages.append({"role": "assistant", "content": resp.content})
        if resp.stop_reason != "tool_use":
            return "".join(b.text for b in resp.content if b.type == "text")
        tool_results = []
        for block in resp.content:
            if block.type != "tool_use":
                continue
            try:
                out = executors[block.name](**block.input)
                tool_results.append({"type": "tool_result",
                    "tool_use_id": block.id, "content": str(out)})
            except Exception as e:
                tool_results.append({"type": "tool_result",
                    "tool_use_id": block.id, "content": f"ERROR: {e}",
                    "is_error": True})
        messages.append({"role": "user", "content": tool_results})
    return "(超过最大步数)"
```

---

## 4. 错误处理

模型可能：
- 传错参数（缺字段、类型不对）→ 返回 `is_error=True` 的 tool_result，模型会自行重试
- 死循环调同一个工具 → 设 `max_steps`
- 幻觉一个不存在的工具名 → 你 try 时 KeyError，回 `ERROR: unknown tool`

**经验**：tool_result 里**老老实实把错误写清楚**，模型很会自我修正。

---

## 5. 并行工具调用

Claude 4.5 系列**默认支持并行**：一次回复里可以输出多个 tool_use 块，你**应该并发执行**：

```python
import asyncio
async def run_tools(blocks):
    tasks = [executors[b.name](**b.input) for b in blocks if b.type=="tool_use"]
    return await asyncio.gather(*tasks, return_exceptions=True)
```

把 `tool_results` 列表一次性回填，省一轮 RTT。

---

## 6. 工具定义最佳实践

1. **description 详细写**：包含"什么时候用 / 什么时候不要用 / 返回什么"。模型主要靠它选工具。
2. **参数类型严格**：用 enum 限定可选值；用 pattern 限定格式
3. **少而精**：超过 20 个工具准确率明显下降；多了用"工具路由 agent"
4. **命名一致**：动词_名词（`get_weather`、`send_email`、`search_docs`）
5. **幂等优先**：同样输入应得到同样结果；非幂等动作（转账）必须人工确认
6. **隐藏内部细节**：description 写给模型看，别把 SQL 表名暴露出来

---

## 7. 工具间依赖

例：先 `search_user(name)` 拿到 user_id，再 `send_email(user_id, ...)`。

不需要你在 schema 里声明依赖，**让模型自己规划**。前提是 description 足够清楚（"send_email 需要先用 search_user 拿到 user_id"）。

---

## 8. 强制使用工具：tool_choice

```python
tool_choice = {"type": "auto"}        # 默认
tool_choice = {"type": "any"}         # 必须调某个工具
tool_choice = {"type": "tool", "name": "get_weather"}  # 必须调这个
```

第三种用法让 Tool Use 变成**结构化输出工具**：定义一个假工具 `record_answer`，schema 即输出格式，模型必然按 schema 填。

---

## 9. 隐藏工具结果（防污染）

有时工具返回大段日志，会污染上下文。技巧：

- 在你的代码里**对结果做摘要**再回填
- 或截断到前 2000 字符 + "[truncated]"
- 永远不要把 API key、用户隐私原样回灌

---

## 10. 完整代码：4 工具助手

```python
# tools_assistant.py
import math, smtplib, datetime as dt
import requests
from anthropic import Anthropic

client = Anthropic()

# ---------- 真实函数 ----------
def get_weather(city: str, unit: str = "c"):
    r = requests.get("https://wttr.in/" + city, params={"format": "j1"}, timeout=5)
    cur = r.json()["current_condition"][0]
    return {"temp_c": cur["temp_C"], "desc": cur["lang_zh"][0]["value"]}

def calculator(expr: str):
    return eval(expr, {"__builtins__": {}}, math.__dict__)

def web_search(q: str, k: int = 3):
    # 假装搜索：真实里接 Tavily / Bing
    return [{"title": f"结果{i}", "snippet": f"关于 {q} 的内容{i}"} for i in range(k)]

def send_email(to: str, subject: str, body: str):
    # demo：不真发，写到文件
    with open("outbox.log", "a") as f:
        f.write(f"{dt.datetime.now()} -> {to}: {subject}\n{body}\n---\n")
    return {"status": "queued"}

executors = {"get_weather": get_weather, "calculator": calculator,
             "web_search": web_search, "send_email": send_email}

# ---------- schema ----------
tools = [
    {"name": "get_weather", "description": "查询城市当前天气。",
     "input_schema": {"type": "object", "required": ["city"],
        "properties": {"city": {"type": "string"},
                       "unit": {"type": "string", "enum": ["c","f"]}}}},
    {"name": "calculator", "description": "计算 Python 表达式（仅数学，无副作用）。",
     "input_schema": {"type": "object", "required": ["expr"],
        "properties": {"expr": {"type": "string"}}}},
    {"name": "web_search", "description": "联网搜索关键词，返回 top-k 摘要。",
     "input_schema": {"type": "object", "required": ["q"],
        "properties": {"q": {"type": "string"}, "k": {"type": "integer"}}}},
    {"name": "send_email", "description": "向指定邮箱发送邮件。注意：必须先经用户确认。",
     "input_schema": {"type": "object", "required": ["to","subject","body"],
        "properties": {"to": {"type": "string"}, "subject": {"type": "string"},
                       "body": {"type": "string"}}}},
]

# ---------- 主循环（同 §3.4） ----------
def run(user_msg):
    messages = [{"role": "user", "content": user_msg}]
    for _ in range(8):
        resp = client.messages.create(model="claude-sonnet-4-5", max_tokens=1024,
            tools=tools, messages=messages)
        messages.append({"role":"assistant","content": resp.content})
        if resp.stop_reason != "tool_use":
            return "".join(b.text for b in resp.content if b.type=="text")
        tr = []
        for b in resp.content:
            if b.type != "tool_use": continue
            try:
                out = executors[b.name](**b.input)
                tr.append({"type":"tool_result","tool_use_id":b.id,"content":str(out)})
            except Exception as e:
                tr.append({"type":"tool_result","tool_use_id":b.id,
                           "content":f"ERROR: {e}","is_error":True})
        messages.append({"role":"user","content":tr})

if __name__ == "__main__":
    print(run("帮我看看北京和上海的气温差几度，并把答案邮件发到 me@x.com"))
```

跑一下：模型会先并行调 `get_weather` 两次，再调 `calculator`，最后调 `send_email`。

---

## 11. 与 OpenAI Function Calling 对比

| | Anthropic | OpenAI |
|---|---|---|
| 字段名 | `tools` / `tool_use` / `tool_result` | `tools` / `tool_calls` / `tool` role |
| schema | input_schema (JSON Schema) | parameters (JSON Schema) |
| 结果回填 | role=user + tool_result 块 | role=tool 单独消息 |
| 并行 | 默认支持 | 默认支持 |

迁移成本不大，主要是字段映射。

---

## 12. Pydantic 自动生成 schema

```python
from pydantic import BaseModel, Field
class WeatherArgs(BaseModel):
    city: str = Field(description="中文城市名")
    unit: str = Field("c", pattern="^[cf]$")

schema = WeatherArgs.model_json_schema()
tool = {"name":"get_weather","description":"...","input_schema": schema}
```

更复杂的可用 `instructor` / `marvin` 这类库自动包装。

---

## 13. 跑一遍 Anthropic Cookbook

强烈建议把 `anthropic-cookbook/tool_use/` 下所有 demo 跑一遍：
- customer_service_agent.ipynb
- calculator_tool.ipynb
- parallel_tools.ipynb
- vision_with_tools.ipynb

每个 notebook 不长，能把官方推荐写法刻进肌肉。

---

## 14. 实战练习

1. 把第 10 节的助手扩充：加 `read_file`、`write_file`、`list_dir` 三个工具，做成"本地文件助手"
2. 测试模型在 30 个真实任务上**调对工具**的成功率，记录到 csv
3. 故意 description 写得很烂，看模型会不会乱调，反推清晰描述的重要性
4. 用 tool_choice=tool 把 Tool Use 当结构化输出，从一段简历里抽 7 个字段
5. 给一个工具引入随机失败（30% 抛错），观察模型重试行为

---

## 15. Checklist

- [ ] 能口述 Tool Use 6 步循环
- [ ] 写过 schema，至少包含 enum / required
- [ ] 通用循环代码能默写
- [ ] 处理过 is_error 重试
- [ ] 并行工具用过
- [ ] tool_choice 三种值都试过
- [ ] 4 工具助手跑通
- [ ] Pydantic 自动 schema 用过
- [ ] Cookbook 至少跑完 4 个 demo
- [ ] 知道幂等 / 危险动作走人工确认

下一周：把 Tool Use 与外部知识结合 — RAG。
