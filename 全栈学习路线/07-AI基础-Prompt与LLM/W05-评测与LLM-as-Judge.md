---
chapter: 07-AI基础-Prompt与LLM
week: W05
title: 评测与 LLM-as-Judge
duration: 7 天
prereq: W01-W04
goal: 给自己的 LLM 应用建一套 50 题评测集 + 自动跑 + 模型对比报告
tags: [eval, llm-judge, promptfoo, ragas, ab-test]
updated: 2026-06-19
---

# W05 · 评测与 LLM-as-Judge

> 没有评测，你**改了 prompt 不知道是变好还是变坏**，调了 RAG 不知道命中率涨了还是跌了。这是产品化与玩具的分水岭。

---

## 0. 一句话警告

> "**It's not done until it's measured.**" — 你看 demo 觉得好，到生产 100 个用户里 30 个翻车，你都没法定位是哪轮改坏的。

---

## 1. 评测三难

任何评测都要在三角里取舍：

```
        覆盖（cover all cases）
           /\
          /  \
         /    \
        /      \
   客观 ────── 成本
```

- **想全覆盖** → 数据集巨大 → 贵
- **想客观** → 自动指标 → 但语义指标差
- **想便宜** → 样本少 → 不全

实践方案：**人工小样本 + 自动指标 + LLM-judge** 三层。

---

## 2. 离线 vs 在线

| | 离线 | 在线 |
|---|---|---|
| 时机 | 上线前 | 上线后 |
| 数据 | 固定 eval set | 真实流量 |
| 指标 | 准确率/Pass@k | 用户 👍👎、停留 |
| 工具 | Promptfoo / Ragas | LangSmith / Helicone / 自建 |

二者都要做，**先离线把 prompt 钉住，再上线收真实信号反哺**。

---

## 3. 评测数据集构建

### 3.1 黄金法则：手工编 30-50 个真实任务

**别用模型生成 eval**（你在用模型考自己）。

每条至少包含：

```yaml
- id: q01
  input: "公司年假怎么算？"
  category: 政策类
  expected_keywords: ["工龄", "10天", "15天"]
  expected_doc_id: "员工手册-3.2"
  difficulty: easy
  notes: "新员工最常问"
```

### 3.2 必须包含的边缘 case

- **超长输入**（>10K token）
- **中英混排**
- **歧义问题**（"老王是谁"，多个老王）
- **应该拒答**的问题（敏感、超域）
- **多步推理**
- **故意错误前提**（"如何治愈感冒？" — 应说"无法治愈，只能缓解"）

### 3.3 ground truth 怎么来

- 文档类：人工标"答案应来自 doc X 第 Y 页"
- 数学类：直接写答案
- 主观类：写一份**参考答案 + 评分 rubric**

---

## 4. 自动指标（便宜但局限）

### 4.1 字符串匹配

```python
def contains(answer, keywords):
    return all(k in answer for k in keywords)
```

够用在"必须包含某关键词"场景。

### 4.2 正则

```python
import re
def has_phone(ans):
    return bool(re.search(r"1[3-9]\d{9}", ans))
```

### 4.3 数值比较

```python
def near(actual, expected, tol=0.05):
    return abs(actual - expected) / abs(expected) < tol
```

### 4.4 JSON Schema 校验

```python
from jsonschema import validate
validate(instance=json.loads(out), schema=expected_schema)
```

### 4.5 BLEU / ROUGE 局限

适合机器翻译/摘要的**词重叠**度，**完全不能反映语义**。"今天天气好"和"今日气候不错"BLEU 极低但语义同。**别拿 BLEU 当主指标**。

---

## 5. LLM-as-a-Judge（核心武器）

### 5.1 直接打分（reference-free）

```
你是公正的评估员。
请按以下维度给回答打 1-5 分：
- 准确性
- 完整性
- 简洁性

问题：{q}
回答：{a}

输出 JSON：{"准确":n,"完整":n,"简洁":n,"理由":"..."}
```

### 5.2 对照参考答案（reference-based）

```
请比较"模型答案"和"参考答案"的语义一致性，
打 1-5 分（5 = 完全等价，1 = 完全不同）。
```

### 5.3 Pairwise 比较（最稳）

```
给定问题与两个回答 A、B，选出更好的一个。
回答必须为 A / B / Tie。
```

Pairwise 比绝对打分**信噪比高**得多，是 Chatbot Arena 的做法。

### 5.4 Rubric 设计要点

- 维度 ≤ 5 个
- 每分必须有**举例**
- 让 judge **先解释再打分**（CoT）
- 用强模型当 judge（Sonnet 4.5+）

### 5.5 Judge 偏见与防御

| 偏见 | 表现 | 防御 |
|---|---|---|
| 位置偏差 | 偏爱 A 或 B | 同对随机翻转两次取平均 |
| 长度偏差 | 偏爱长答 | rubric 明示"无关长度" |
| 自我偏好 | judge 偏爱自家模型 | 用别家模型当 judge |
| 风格偏差 | 偏爱 markdown | rubric 明示 |

---

## 6. 人工评测平台 Argilla

```bash
pip install argilla
```

把 (问题, 答案 A, 答案 B) 推到 Argilla，团队成员在网页上点选偏好。结果导出后即"金标"。适合周期性请专家标注。

---

## 7. 评测框架对比

| 工具 | 定位 | 强项 |
|---|---|---|
| **Promptfoo** | 轻量 yaml-driven | 多模型对比、CI 集成 |
| **Inspect** (UK AISI) | 严肃科研 | 多步 agent 评测 |
| **Ragas** | RAG 专用 | 忠实度/相关度内置 |
| **DeepEval** | pytest 风格 | 写测试用例方式 |
| **LangSmith** | 在线一体 | trace + eval |

---

## 8. Promptfoo 上手

### 8.1 安装

```bash
npm i -g promptfoo
```

### 8.2 配置 promptfooconfig.yaml

```yaml
prompts:
  - "你是简洁的助手。问题：{{q}}"
  - "你是严谨的助手，先列要点再回答。问题：{{q}}"

providers:
  - anthropic:messages:claude-haiku-4-5
  - anthropic:messages:claude-sonnet-4-5
  - openai:chat:gpt-5

tests:
  - vars: { q: "公司年假怎么算？" }
    assert:
      - type: contains
        value: "工龄"
      - type: llm-rubric
        value: "回答应清晰说明天数与工龄关系"

  - vars: { q: "12345 × 6789 = ?" }
    assert:
      - type: equals
        value: "83810205"
```

### 8.3 跑

```bash
promptfoo eval
promptfoo view   # 浏览器看可视化报表
```

立刻看到**3 模型 × 5 题 × 2 prompt 版本**的胜负矩阵。

### 8.4 CI 集成

```yaml
# .github/workflows/eval.yml
- run: promptfoo eval --max-concurrency 4
- run: promptfoo eval --share   # 生成可分享报告链接
```

每次 PR 改 prompt 自动跑，跌点即报警。

---

## 9. Ragas（RAG 专用）

```python
from ragas import evaluate
from ragas.metrics import (faithfulness, answer_relevancy,
                           context_precision, context_recall)

result = evaluate(
    dataset,           # 含 question, answer, contexts, ground_truth
    metrics=[faithfulness, answer_relevancy,
             context_precision, context_recall],
)
print(result)
```

四大指标：
- **faithfulness**：答案中事实是否都来自 context
- **answer_relevancy**：答案是否切题
- **context_precision**：检索的 context 中有多少是有用的
- **context_recall**：金标信息是否都被检索到

---

## 10. 在线评测

### 10.1 显式信号

每条回答下加 👍👎 + 可选填理由。汇到表里：

```sql
CREATE TABLE feedback (
    id text, prompt text, response text,
    rating int, reason text, model text, ts timestamp);
```

### 10.2 隐式信号

- 用户**有没有继续追问**（满意度高 → 不追问 / 满意度低 → 反复改问）
- **复制率**（回答被复制说明有用）
- **停留时间**

### 10.3 A/B 测试

```python
def route(user_id):
    return "A" if hash(user_id) % 2 == 0 else "B"
```

两组 prompt 各跑一周，比较 👍 率、复制率、追问率。**一次只改一个变量**，否则归因失败。

---

## 11. 实战：你的项目 50 题 eval set

> 拿 W04 做的 RAG 系统当试验品。

### 11.1 数据集

```yaml
# eval.yaml
tests:
  - vars: { q: "新员工年假几天？" }
    assert:
      - type: contains-any
        value: ["5天", "5 天"]
      - type: llm-rubric
        value: "答案必须基于员工手册第 3 章；找不到应说'未提及'"
  # … 49 more
```

至少覆盖：

| 类型 | 数量 |
|---|---|
| 直接事实 | 15 |
| 多文档综合 | 10 |
| 数值计算 | 5 |
| 应该拒答 | 5 |
| 多轮上下文 | 5 |
| 中英混合 | 5 |
| 长输入 | 5 |

### 11.2 跑出对比报告

```bash
# 试 3 种 RAG 配置
RAG_CONFIG=plain promptfoo eval --output plain.json
RAG_CONFIG=hybrid promptfoo eval --output hybrid.json
RAG_CONFIG=hybrid+rerank promptfoo eval --output rerank.json
```

把胜率画成柱状图，写进 README，团队评审看一眼即懂。

---

## 12. 练习

1. 把你 W04 的 RAG 数据集做满 50 题
2. 用 Promptfoo 对比 Haiku / Sonnet / Opus 三模型 × 2 prompt
3. 写一个 LLM-judge，pairwise 评 100 对样本，统计位置偏差
4. 给 chat CLI 加 👍👎 + 写到 SQLite
5. 用 Ragas 出一份 RAG 报告并改善其中最差的指标

---

## 13. Checklist

- [ ] 离线 eval set ≥ 50 题，含边缘 case
- [ ] 5 种自动指标（contains/regex/数值/schema/exact）至少各用一种
- [ ] LLM-judge 至少做 pairwise + rubric
- [ ] 知道并防御 4 种 judge 偏见
- [ ] Promptfoo 跑出 3 模型对比报表
- [ ] Ragas 跑出 RAG 4 指标
- [ ] 在线 👍👎 收集已部署
- [ ] 至少做过一次 A/B 测试
- [ ] eval 已经接进 CI

下一周是终极硬仗：从零实现一个 GPT，建立你的 LLM 终极心智模型。
