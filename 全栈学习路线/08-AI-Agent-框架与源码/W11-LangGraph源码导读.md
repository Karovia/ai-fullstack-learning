---
title: W11 LangGraph 源码导读
chapter: 08-AI-Agent-框架与源码
week: 11
date: 2026-06-19
tags: [agent, langgraph, source-code, pregel, checkpointer]
prev: "[[W10-可观测性]]"
next: "[[W12-写mini-agent-sdk]]"
---

# W11 LangGraph 源码导读

## 0. 一句话开场

会用框架的人多，懂框架的人少。这周我们一起把 LangGraph 拆开看，搞懂它的"图执行引擎"是怎么写的——既能让你写 bug 时一眼看出根因，也是面试 Agent 工程师的硬通货。

## 1. 为什么读源码

类比：开车的人多，懂发动机原理的人少。但能修车、能改装、能造车的人，永远是更稀缺的。

具体收益：

- 出 bug 时不用瞎猜（"为什么 state 没更新？"——你看一眼 channel 实现就知道了）
- 性能优化心里有数（哪些操作 O(n)、哪些操作 O(1)）
- 设计自己 Agent SDK 时有「最佳实践参考」（W12 我们就要造一个）
- 面试时谈源码，比谈使用经验高一档

## 2. 准备工作

```bash
git clone https://github.com/langchain-ai/langgraph.git
cd langgraph
git checkout v0.2.x  # 选个稳定 tag，避免主干太动
code .  # VS Code 或 Cursor，方便跳转
```

库结构（核心包 `libs/langgraph/langgraph/`）：

```
graph/
  __init__.py
  state.py      # StateGraph, CompiledStateGraph
  graph.py      # 基类
  message.py    # MessagesState 等便捷类
pregel/
  __init__.py
  loop.py       # 主循环（PregelLoop）
  read.py       # PregelRead（节点读 channel）
  write.py      # PregelWrite（节点写 channel）
  _runner.py    # 任务调度器
  io.py         # 输入输出处理
  algo.py       # superstep 算法
channels/
  base.py
  last_value.py
  topic.py
  binop.py
  context.py
checkpoint/
  base.py
  memory.py
  serde/
prebuilt/
  chat_agent_executor.py  # create_react_agent 的实现
  tool_node.py
```

阅读心法：**先 demo 再源码**。先用框架跑通最小例子，再把那个例子的执行流程"在源码里走一遍"。

## 3. StateGraph 内部

### 3.1 add_node 做了什么

打开 `graph/state.py` 找 `def add_node`：

```python
def add_node(self, node, action=None, *, metadata=None, input=None, retry=None):
    if isinstance(node, str):
        name, action = node, action
    else:
        action = node
        name = getattr(action, "name", None) or action.__name__
    self.nodes[name] = StateNodeSpec(
        runnable=coerce_to_runnable(action),
        metadata=metadata or {},
        input=input or self.schema,
        retry_policy=retry,
    )
```

关键：所有 node 被包成 `Runnable`（LangChain 的统一抽象，提供 invoke / batch / stream / astream）。

### 3.2 add_edge / add_conditional_edges

```python
self.edges.add((source, target))  # 普通边
self.branches[source][name] = Branch(condition, ends, then)  # 条件边
```

存的是「图的拓扑」，没有任何执行逻辑。

### 3.3 compile 的产物

```python
def compile(self, checkpointer=None, interrupt_before=None, interrupt_after=None, ...):
    self._validate()
    return CompiledStateGraph(
        builder=self,
        nodes={k: PregelNode(...) for k, v in self.nodes.items()},
        channels=self._build_channels(),
        ...
    )
```

`CompiledStateGraph` 才是真正可跑的对象，继承自 `Pregel`。它做了：

- 把每个 node 变成 `PregelNode`（含 channels_in / channels_out / triggers / writers）
- 把 state schema 变成 channels（每个字段一个 channel）
- 把边变成 channel 订阅关系

记住：**StateGraph 是搭积木，CompiledStateGraph / Pregel 才是引擎**。

## 4. Pregel 模型简介

Pregel 是 Google 2010 年提出的图计算模型（论文："Pregel: A System for Large-Scale Graph Processing"），核心思想叫 **BSP（Bulk Synchronous Parallel）**：

```
loop:
    1. 收消息（每个顶点收上一轮发来的消息）
    2. 顶点计算（基于自身状态 + 消息）
    3. 发消息（给邻居）
    4. 同步屏障（所有顶点都做完，才进入下一轮）
直到没有顶点活跃
```

每一轮叫 **superstep**。LangGraph 把每个 node 当作一个顶点，channel 当作消息通道。

为什么这么设计？

- 同步屏障让并行执行可预测（不会出现"A 还没跑完，B 就读到一半数据"）
- 容易做 checkpoint（每个 superstep 之间状态一致）
- 容易做 streaming（一个 superstep 一个事件）

## 5. Pregel 在 LangGraph 中的实现

### 5.1 主循环 PregelLoop

`pregel/loop.py`，简化伪代码：

```python
class PregelLoop:
    async def tick(self) -> bool:
        # 1. 选出本轮要跑的 task
        self.tasks = prepare_next_tasks(
            self.checkpoint, self.channels, self.nodes, self.step,
        )
        if not self.tasks:
            return False  # 没活了，退出
        # 2. 执行（可并发）
        async for fut in run_with_retry(self.tasks):
            await self.put_writes(fut.id, fut.writes)
        # 3. 应用写入到 channels（同步屏障）
        apply_writes(self.checkpoint, self.channels, self.tasks)
        # 4. 持久化 checkpoint
        if self.checkpointer:
            await self.checkpointer.aput(self.config, self.checkpoint, ...)
        self.step += 1
        return True
```

外层：

```python
while await self.tick():
    pass
```

### 5.2 PregelExecutableTask

```python
@dataclass
class PregelExecutableTask:
    name: str
    input: Any
    proc: Runnable
    writes: list  # 节点产生的 channel 写入
    triggers: Sequence[str]
    retry_policy: RetryPolicy | None
    id: str
    path: tuple
```

每个 task = 「这一轮要跑的某个节点的某次实例」。同一节点可能被 fan-out 多次（map-reduce 场景）。

### 5.3 prepare_next_tasks

`pregel/algo.py`：

- 看每个节点 `triggers`（订阅的 channel）有没有"被更新过"
- 有更新 → 该节点入选本轮 task
- 通过 `Send` 包发起的 fan-out 也在这里展开

## 6. Channel：节点间通信

### 6.1 LastValue

`channels/last_value.py`：每次 update 覆盖前值。这是最常见的字段类型。

```python
class LastValue(BaseChannel):
    def update(self, values):
        if not values: return False
        if len(values) > 1:
            raise InvalidUpdateError("LastValue 只能收一个写入")
        self.value = values[-1]
        return True
    def get(self): return self.value
```

如果两个并行 node 都写同一个 LastValue 字段 → 报错。这就是为什么「并行写 messages 必须用 reducer」。

### 6.2 Topic

`channels/topic.py`：append 模式，所有写入都保留。`messages` 字段就是 Topic（叫 `add_messages`）。

```python
class Topic(BaseChannel):
    def update(self, values):
        self.values.extend(values)
        return True
```

### 6.3 BinaryOperatorAggregate

`channels/binop.py`：用任意二元算子聚合（求和 / 取最大 / 拼接）。

```python
from operator import add
counter: Annotated[int, add]   # 多个写入会被加起来
```

### 6.4 Context

把"运行时依赖"（如 db connection）注入到节点，不参与 checkpoint。

## 7. 中断（interrupt）实现

人工审批 / 长任务暂停 必备。

### 7.1 静态 interrupt

`compile(interrupt_before=["payment"])`：在执行 `payment` 节点前 PregelLoop 直接 break，把状态保存。下次 `invoke(None, config)` 时从 checkpoint 恢复继续跑。

### 7.2 动态 interrupt

```python
from langgraph.types import interrupt
def node(state):
    answer = interrupt({"question": "你确认转账吗？"})
    return {"confirmed": answer}
```

源码里：`interrupt()` 抛出 `GraphInterrupt` 异常 → PregelLoop catch → 标记任务挂起 → 持久化 → 等待 `Command(resume=...)` 注入。

阅读时找：`pregel/_runner.py` 中的 `GraphInterrupt` 处理分支。

## 8. Checkpointer 设计

### 8.1 数据模型

```python
class Checkpoint(TypedDict):
    v: int
    id: str           # 当前 checkpoint id
    ts: str           # ISO 时间
    channel_values: dict
    channel_versions: dict
    versions_seen: dict[node, dict[channel, version]]
    pending_sends: list
```

### 8.2 父子关系

每个 checkpoint 通过 `parent_checkpoint_id` 指向上一个，形成链。同一个 thread 是一条链，分叉（fork）会产生子链——这就是 time-travel debugging 的基础。

### 8.3 存储后端

- `MemorySaver`：内存（开发用）
- `SqliteSaver` / `AsyncSqliteSaver`：单机持久化
- `PostgresSaver`：生产
- 字段：`checkpoints` 表 + `writes` 表（增量写入，避免每次复制全量 state）

### 8.4 序列化

`checkpoint/serde/jsonplus.py`：基于 msgpack，支持 datetime / set / Pydantic 等扩展。每个对象前缀类型 tag，解码时按 tag 还原。

如果你存了自定义类，没注册 serde → 反序列化失败 → 一定要看这块代码。

## 9. 流式实现

`Pregel.astream`：

```python
async def astream(self, input, config=None, *, stream_mode="values"):
    async with PregelLoop(...) as loop:
        while await loop.tick():
            for chunk in loop.output(stream_mode):
                yield chunk
```

stream_mode 几种：

- `values`：每个 superstep 后输出完整 state
- `updates`：只输出本 step 的增量
- `messages`：输出 LLM token 级流式（基于 LangChain callback）
- `debug`：极详细，包含每个 task 的输入输出

底层用 Python 的 async generator + 协程，把 LLM 流式 token 也透传出来。

## 10. 阅读路线推荐

按这个顺序读最好懂：

1. `graph/state.py` 的 `StateGraph` 类（30 分钟）
2. `pregel/__init__.py` 的 `Pregel` 类签名（10 分钟，先认识接口）
3. `pregel/loop.py` 的 `tick`（重点，1 小时）
4. `pregel/algo.py` 的 `prepare_next_tasks` 和 `apply_writes`（1 小时）
5. `channels/last_value.py` + `topic.py`（30 分钟）
6. `checkpoint/base.py` 和 `memory.py`（30 分钟）
7. `prebuilt/chat_agent_executor.py`（看官方 ReAct 怎么写，0.5 小时）
8. `pregel/_runner.py`（最难，留到最后）

### 关键 commit 推荐

- 加入 Pregel 模型的初版 commit（搜 commit message "pregel"）
- 引入 interrupt 的 commit
- v0.2 里 Send / Command 的引入

不必每行都懂，**先看主线，跳过细枝末节**。

## 11. 边读边记笔记

强烈建议用 Obsidian / 你的 vault：

- 每个核心类一篇笔记（StateGraph / Pregel / PregelLoop / Channel / Checkpoint）
- 每篇含：作用 / 关键方法 / 关键字段 / 我画的时序图 / 我的疑问
- 搭一个 graph view，看看依赖关系

笔记越啰嗦越好，三个月后回头你会感谢自己。

## 12. 实战：给 _runner.py 加中文注释 + 写博客

### 12.1 加注释

fork 一份 langgraph，在 `pregel/_runner.py` 的每个函数 / 关键分支加中文注释。例：

```python
async def arun_with_retry(task, retry_policy, ...):
    """
    带重试地跑一个 task。
    设计点：
    - 每次重试前 sleep 指数退避
    - 只对 retry_policy.retry_on 中的异常重试
    - GraphInterrupt 永远不重试（语义错误）
    """
    ...
```

PR 不一定能合并（项目主语为英文），但你的注释版可以放自己 GitHub 仓库。

### 12.2 写博客

主题建议（任选一）：

1. 《一图读懂 LangGraph 的 Pregel 模型》
2. 《我把 LangGraph checkpoint 表结构搞懂了》
3. 《LangGraph interrupt 是怎么实现的》
4. 《为什么 LangGraph 把 channel 抽象出来——和 Pregel 的关系》

每篇配 3-5 张时序图 / 流程图。这种博客在中文 AI 社群极少，很容易出圈。

## 13. 练习

1. 画 Pregel 一次 superstep 的时序图（含 prepare_next_tasks → run → apply_writes → checkpoint）
2. 给一个最小 demo（2 个节点，1 条边）打断点，单步看 PregelLoop 是怎么跑的
3. 自己实现一个最简版 LastValue Channel（10 行），跑通一个 toy 状态机
4. 用 SqliteSaver 跑同一个 graph，停下来，重启进程恢复 → 观察 thread / checkpoint 表的变化
5. 把你的注释版 fork push 到 GitHub，README 写阅读顺序

## 14. checklist

- [ ] 能口述 Pregel 模型的三步循环
- [ ] 看过 `pregel/loop.py` 的 tick 方法源码
- [ ] 理解 LastValue / Topic / BinaryOp 三种 channel 区别
- [ ] 看过 checkpointer 的表结构 + 序列化
- [ ] 在自己的 LangGraph 里读懂过一次 trace
- [ ] 完成 _runner.py 中文注释
- [ ] 写完一篇源码博客并发出去

下一周（W12）是收尾——我们把这八周所学融合，**亲手写一个 mini-agent-sdk**。

