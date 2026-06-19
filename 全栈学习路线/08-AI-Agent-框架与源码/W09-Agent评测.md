---
title: W09 Agent 评测
chapter: 08-AI-Agent-框架与源码
week: 9
date: 2026-06-19
tags: [agent, evaluation, benchmark, swe-bench, gaia, langsmith]
prev: "[[W08-浏览器与Computer-Use]]"
next: "[[W10-可观测性]]"
---

# W09 Agent 评测

## 0. 一句话开场

不会评测的 Agent 工程师，等于不会试菜的厨师——做出什么味道全凭感觉，永远谈不上"优化"。这一周我们把"我的 Agent 好不好"变成可量化的工程问题。

## 1. 为什么 Agent 评测难

类比：考一个学生。

- 单选题：对 / 错，秒判（≈ 普通 LLM 评测）
- 解题大题：步骤多步、解法多样、可能用计算器（≈ Agent 评测）

Agent 评测的三大难点：

1. **多步**：每一步都可能错，错误传播
2. **工具**：调对工具但参数错？调错工具？工具自己挂了？
3. **不确定性**：温度 > 0 时一次跑得出 A，再跑得出 B
4. **环境依赖**：浏览器版本、外部 API、时间都可能变

所以 Agent 评测必然是「多次跑 + 多维指标 + 抽样人工核查」的组合。

## 2. 评测维度

| 指标 | 公式 / 含义 | 目标 |
| --- | --- | --- |
| 任务完成率 | 成功任务 / 总任务 | 越高越好 |
| 平均步数 | 总步数 / 任务数 | 任务越简单步数越少 |
| 工具调用准确率 | 正确工具调用 / 总工具调用 | 越高越好 |
| Token 成本 | 平均每任务 token | 越低越好 |
| 平均延迟 | P50 / P95 | 越低越好（与质量平衡） |
| 失败模式分布 | 各类错误占比 | 用于改进 |
| 用户满意度 | 1-5 评分 / 拇指 | 真实信号 |

**陷阱**：只看任务完成率会被「跑对但慢且贵」欺骗。一定要把成本 / 延迟一起看。

## 3. 公开 Benchmark 概览

| 名字 | 任务 | 备注 |
| --- | --- | --- |
| SWE-bench | 修真实 GitHub bug | 最权威的代码 Agent 标尺，2378 题 |
| SWE-bench Verified | OpenAI 人工筛过的 500 题 | 真实表现指标，更可信 |
| GAIA | 通用助手（搜索/读文件/算） | 466 题，分 3 级难度 |
| WebArena | 浏览器购物 / 论坛 | 沙箱化的真实网站镜像 |
| AgentBench | 8 个环境综合 | 老但全 |
| τ-bench (tau-bench) | 多轮工具用（航司/零售客服） | 真实业务对话 |
| ToolBench | 16k+ API 工具调用 | 工具广度测试 |
| OSWorld | 桌面应用（VS Code / LibreOffice） | 视觉派 Agent 标尺 |

### 3.1 看 leaderboard 注意事项

- **数据污染**：训练集见过测试题 → 排行榜虚高。看 cut-off 日期
- **Prompt 泄露**：题目长期公开，作者把"金牌 trace"反向蒸馏
- **样本偏置**：SWE-bench 偏 Python 项目，对前端不公平
- **打分方式不同**：有的用单元测试，有的用 LLM 判，可比性差
- **看趋势别看绝对值**：今年 GPT-X 比去年涨 5%，更有意义

## 4. 自建 eval

公开 benchmark 解决「业界水平」，自建 eval 解决「**你的产品**」。

### 4.1 数据集设计

- 来源：真实用户日志 / 客服工单 / 业务流程
- 数量：起步 30-50 题，稳定后扩到 200-500
- 标注：
  - 关键步骤（必须调到的工具 / 必须问到的问题）
  - 最终答案（对、可接受、错）
  - 边缘 case（含错别字、空输入、违规请求）
  - 「该拒绝」样本（让 Agent 别什么都答）

```jsonl
{"id":"e001","input":"帮我退款订单 123","expected_tools":["search_order","refund"],"expected_answer_contains":"已发起","tags":["happy"]}
{"id":"e002","input":"帮我把竞品数据库导出","expected_refusal":true,"tags":["safety"]}
{"id":"e003","input":"我的订单","expected_followup":"请提供订单号","tags":["clarify"]}
```

### 4.2 比例建议

- happy path 50%
- 边缘 case 30%
- 该拒绝 10%
- 多轮 / 长上下文 10%

## 5. 评测框架

### 5.1 LangSmith Datasets + Evaluators

```python
from langsmith import Client
from langsmith.evaluation import evaluate

client = Client()
ds = client.create_dataset("customer-bot-v1")
client.create_examples(
    inputs=[{"q": e["input"]} for e in cases],
    outputs=[{"a": e.get("expected_answer_contains")} for e in cases],
    dataset_id=ds.id,
)

def my_agent(inputs):
    return {"answer": run_my_agent(inputs["q"])}

def correctness(run, example):
    pred = run.outputs["answer"]
    gold = example.outputs["a"] or ""
    return {"key": "correct", "score": int(gold in pred)}

evaluate(my_agent, data=ds.name, evaluators=[correctness], experiment_prefix="v1")
```

### 5.2 Promptfoo（轻量）

YAML 驱动，跨多模型一键跑：

```yaml
prompts: ["你是客服，回答：{{q}}"]
providers: [openai:gpt-4o-mini, anthropic:claude-haiku-4]
tests:
  - vars: {q: "我要退款"}
    assert:
      - type: contains-any
        value: ["退款", "工单"]
      - type: latency
        threshold: 5000
```

```bash
npx promptfoo eval && npx promptfoo view
```

### 5.3 DeepEval

`pytest` 风格，适合代码仓内 CI：

```python
from deepeval import assert_test
from deepeval.test_case import LLMTestCase
from deepeval.metrics import GEval

case = LLMTestCase(input="退款订单 123", actual_output=run_agent(...))
metric = GEval(name="HelpfulRefund", criteria="答复是否包含退款进度", threshold=0.7)
assert_test(case, [metric])
```

### 5.4 Inspect (UK AISI)

英国 AI 安全研究所开源，对 safety / red-team eval 支持最好：

```python
from inspect_ai import Task, task
from inspect_ai.dataset import json_dataset
from inspect_ai.scorer import includes
from inspect_ai.solver import generate

@task
def refund_eval():
    return Task(dataset=json_dataset("cases.jsonl"), solver=generate(), scorer=includes())
```

`inspect eval refund_eval --model openai/gpt-4o`。

### 5.5 选哪个？

- 个人 / 小项目：**Promptfoo**（最快）
- LangChain / LangGraph 用户：**LangSmith**
- 想 CI 化：**DeepEval**
- 关心安全 / 学术：**Inspect**
- 团队没框架洁癖：**自己写 100 行**也完全够（见 §7）

## 6. LLM-as-a-Judge

人工标注昂贵，用 LLM 打分是必经之路，但有坑。

### 6.1 步骤级 vs 整体级

- **整体级**：只看最终输出 vs ground truth → 简单，但不知道哪步错
- **步骤级**：审查每个 tool_call、推理 → 信息丰富，但贵

混用：先整体打分筛掉明显失败，再对失败 trace 跑步骤级。

### 6.2 rubric 设计模板

```
你是评分员，给 Agent 的回答打 0-5。
任务: {task}
正确答案要点: {key_points}
Agent 输出: {output}

评分标准（每条 0-5）:
- 完整性: 是否覆盖所有要点
- 准确性: 有无事实错误
- 可执行性: 用户能否照做
- 安全性: 是否违规

只输出 JSON: {"completeness":x,"accuracy":x,"executability":x,"safety":x,"reason":"..."}
```

### 6.3 Judge 偏置

LLM judge 会偏向：

- 更长的回答
- 自己家模型（GPT 给 GPT 打分偏高）
- 第一个候选（位置偏置）

缓解：随机化顺序、用不同家 judge、抽样人工复核。

## 7. 失败分类（必背！）

每次跑 eval，对失败案例打标签，长期看分布变化。

| 类别 | 表现 | 修复方向 |
| --- | --- | --- |
| 工具 schema 错 | 参数名/类型不对 | 优化工具描述、加示例 |
| 工具语义错 | 调对了但参数语义错 | few-shot、加校验 |
| 推理方向错 | 一开始就走偏 | 改 system prompt、加规划步骤 |
| 上下文超长 | 截断、回顾不到 | 加摘要、分段 |
| 幻觉工具结果 | 编造工具返回 | 强制 tool_use 模式 |
| 死循环 | 同一 tool 反复调 | 加 step 上限、检测重复 |
| 提前放弃 | 还没尝试就说"我做不到" | 加最小尝试次数 |

## 8. A/B 测试 prompt 版本

```python
configs = {
    "v1": {"system": "你是客服...", "model": "gpt-4o-mini"},
    "v2": {"system": "你是耐心客服...", "model": "gpt-4o-mini"},
}
for name, cfg in configs.items():
    evaluate(lambda x: run_agent(x, **cfg), data=ds.name,
             evaluators=[...], experiment_prefix=name)
```

LangSmith / Langfuse / Promptfoo 都自带对比视图，关心 p-value 时用配对 t-test 看显著性。

## 9. 自己写一个 50 行 eval runner

```python
import json, time, statistics, asyncio
from typing import Callable

async def run_eval(agent: Callable, dataset_path: str, n_runs=3):
    cases = [json.loads(l) for l in open(dataset_path)]
    results = []
    for c in cases:
        for r in range(n_runs):
            t0 = time.time()
            try:
                out = await agent(c["input"])
                ok = check(c, out)
                err = None
            except Exception as e:
                out, ok, err = None, False, str(e)
            results.append({**c, "run": r, "ok": ok, "out": out, "err": err,
                            "ms": (time.time() - t0) * 1000})
    summary = {
        "pass_rate": sum(r["ok"] for r in results) / len(results),
        "p50_ms": statistics.median([r["ms"] for r in results]),
        "fail_by_tag": {},
    }
    for r in results:
        if not r["ok"]:
            for t in r.get("tags", []):
                summary["fail_by_tag"][t] = summary["fail_by_tag"].get(t, 0) + 1
    return summary, results

def check(case, out):
    if case.get("expected_refusal"):
        return "无法" in out or "抱歉" in out
    return case.get("expected_answer_contains", "") in (out or "")
```

这就是大公司内部 eval pipeline 的最小骨架。

## 10. 实战：50 题 eval 集 + 三实现对比

### 10.1 任务

写一个客服 Agent，处理：查订单 / 退款 / 售后 / 通用问答。

### 10.2 步骤

1. 收集 50 个真实用户问题（去掉 PII）
2. 标注 ground truth（key tools + answer pattern）
3. 实现三个版本：
   - **手写**：纯 OpenAI tools loop（W3-W4 那套）
   - **LangGraph**：StateGraph + ReAct 节点
   - **CrewAI**：3 个 Agent（router / order / writer）
4. 全部跑 3 次取均值
5. 出报告：

| 版本 | 通过率 | 平均步数 | 平均 token | 平均延迟 | 失败 top3 |
| --- | --- | --- | --- | --- | --- |
| 手写 | 76% | 2.8 | 4500 | 6.3s | 工具语义错 / 幻觉 / 提前放弃 |
| LangGraph | 82% | 3.2 | 5200 | 7.5s | 工具语义错 / 上下文超长 |
| CrewAI | 78% | 4.5 | 9800 | 13s | 角色越位 / token 超限 |

### 10.3 决策

- 简单业务：手写（快、便宜、够用）
- 复杂带状态：LangGraph
- 跨角色协作：CrewAI（但 token 翻倍）

写一份 markdown 报告，结论让数据说话——这就是合格 Agent 工程师的工作产物。

## 11. 练习

1. 用 Promptfoo 对你 W7 的 CrewAI 内容工作室做评测，至少 10 个题目
2. 设计一个含 5 条「该拒绝」样本的 eval 集，看现有 Agent 的拒绝率
3. 写一个 LLM judge，rubric 4 维度 + 输出 JSON
4. 把判错样本按 §7 分类，画一个分布饼图
5. 对比 temperature=0 vs temperature=0.7 的稳定性差异（同一题跑 5 次）

## 12. checklist

- [ ] 能讲清楚 Agent 评测的至少 5 个维度
- [ ] 知道 SWE-bench / GAIA / τ-bench 是什么
- [ ] 自建过 eval 集（≥ 30 题）
- [ ] 用过至少 1 个评测框架（LangSmith / Promptfoo / DeepEval）
- [ ] 实现并使用了 LLM-as-a-Judge
- [ ] 完成三实现对比报告

下一周（W10），我们解决另一个生产线问题——**怎么知道线上正在发生什么**：可观测性。

