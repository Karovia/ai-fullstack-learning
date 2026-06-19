---
chapter: 07-AI基础-Prompt与LLM
week: W01
title: LLM 全景与 API
duration: 7-10 天
prereq: 第 06 章 后端进阶（HTTP / 流式 / 异步）
goal: 理解 LLM 是什么、知道 2026 主流模型怎么选、能用 Anthropic API 写出带流式与缓存的 chat CLI
tags: [llm, anthropic, api, ollama, tokenizer]
updated: 2026-06-19
---

# W01 · LLM 全景与 API

> 一句话：**LLM 是一台"读完前文就猜下一个字"的超级机器**。把这个直觉刻进脑子，后面所有玄学（Prompt、RAG、Agent）都能落回到地面。

---

## 0. 写在前面：本周学完你能做什么

- 能向同事用大白话解释"LLM 是什么 / 怎么训出来的"
- 知道 2026 年主流模型分别擅长什么、贵不贵、上下文多大
- 能用 Anthropic API 写出**流式 + 多轮 + 切模型 + prompt cache** 的命令行聊天工具
- 能在自己的 Mac 上用 Ollama 跑本地 Llama 4
- 看见账单不慌，能用 tokenizer 反推一段对话花了多少钱

---

## 1. LLM 是什么：5 分钟建立心智模型

### 1.1 一句话定义

> **LLM (Large Language Model) = 给定一串 token，预测下一个 token 概率分布的神经网络。**

### 1.2 类比：写作接龙的天才学生

想象一个从小读完了整个互联网的学生，你给他半句话"今天天气真"，他立刻知道接"好"的概率 70%、"差"的概率 20%、"热"的概率 10%……然后他从这个分布里**采样**一个词，写出来，再把这个词加回去，继续猜下一个。

ChatGPT、Claude、Gemini 本质都在做这件事。

### 1.3 为什么"只是猜下一个词"能涌现智能？

因为要把"猜下一个词"做到极致，模型必须**学会**：
- 语法（不然句子不通）
- 事实（不然下一个词错）
- 推理（不然解不了"小明有 3 个苹果……"）
- 风格（不然不像人写的）

规模一上去，这些能力**涌现 (emergence)**。这就是 Scaling Law 的魔法。

---

## 2. 训练流程速览：从乱码到 ChatGPT

```
原始互联网文本 (15T tokens)
     │ Pretrain（无监督，下一个词预测）
     ▼
基础模型 (Base Model) — 会接龙但不会聊天
     │ SFT（监督微调，给"指令-回答"对）
     ▼
指令模型 (Instruct Model) — 会聊天但有时不听话
     │ RLHF / DPO（用人类偏好对齐）
     ▼
对齐后模型 (Chat Model) — 你在用的 Claude/GPT
```

| 阶段 | 数据 | 算力 | 时间 |
|---|---|---|---|
| Pretrain | 万亿 tokens | 数千 H100，数月 | 占 99% 成本 |
| SFT | 数万-数百万对话 | 几十张卡，几天 | 教礼貌 |
| RLHF/DPO | 数万人类偏好 | 同上 | 教价值观 |

记一句：**Pretrain 决定了"知道多少"，后训练决定了"听不听话"**。

---

## 3. 2026 主流模型现状（重要！背下来）

> 截至 2026 年 6 月，下面是真实在用的旗舰阵容。

### 3.1 Claude（Anthropic）— 代码与 Agent 之王

| 模型 | 定位 | 上下文 | 特点 |
|---|---|---|---|
| **Claude Opus 4.5** | 最强旗舰 | 1M | 长任务规划、复杂代码 |
| **Claude Sonnet 4.5** | 主力 | 1M | 性价比之王，extended thinking |
| **Claude Haiku 4.5** | 小快灵 | 200K | 高频调用、便宜快 |

亮点：1M 上下文、Tool Use 稳定、prompt caching 省钱、computer use。

### 3.2 OpenAI

| 模型 | 定位 | 备注 |
|---|---|---|
| **GPT-5** | 旗舰 | 通用强 |
| **GPT-4.5** | 上一代主力 | 仍在大量产品中 |
| **o3 / o4** | 推理系列 | 数学/代码题霸 |

### 3.3 Google Gemini 2.5

- **Pro**：多模态最强（视频原生）
- **Flash**：超长上下文 + 极便宜

### 3.4 开源阵营

| 模型 | 厂家 | 特点 |
|---|---|---|
| **Llama 4** | Meta | 开源旗舰，本地可跑 |
| **DeepSeek V3 / R1** | DeepSeek | 推理强 + 价格屠夫 |
| **Qwen 3** | 阿里 | 中文最强开源 |

### 3.5 选型矩阵

| 场景 | 推荐 |
|---|---|
| 写代码 / Agent | Claude Sonnet 4.5 |
| 极致便宜批处理 | Haiku 4.5 / DeepSeek V3 |
| 多模态视频 | Gemini 2.5 Pro |
| 离线 / 数据敏感 | Llama 4 本地 |
| 中文创作 | Qwen 3 / Claude |
| 数学题 | o4 / DeepSeek R1 |

**比较维度**：上下文、价格（input/output）、速度（tok/s）、推理能力、Tool Use 稳定性、多模态、是否开源。

---

## 4. 闭源 vs 开源 vs 本地

| | 闭源 API | 开源 API | 本地 (Ollama) |
|---|---|---|---|
| 质量 | 最强 | 中-强 | 弱-中 |
| 成本 | 按 token | 按 token / 自建 | 一次电费 |
| 隐私 | 数据出境 | 看供应商 | 100% 本地 |
| 延迟 | 网络 | 网络 | 取决于硬件 |
| 适合 | 产品上线 | 国内合规 | 学习/原型 |

---

## 5. Anthropic API 完整教程

### 5.1 申请 API key

1. 打开 console.anthropic.com → 注册
2. Billing 充值（最低 5 美元）
3. API Keys → Create Key → 复制 `sk-ant-xxx`
4. 写进 `~/.zshrc`：

```bash
export ANTHROPIC_API_KEY="sk-ant-xxx"
```

### 5.2 安装 SDK

```bash
pip install anthropic
```

### 5.3 第一个请求

```python
from anthropic import Anthropic

client = Anthropic()  # 自动读 ANTHROPIC_API_KEY

resp = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "用一句话解释 LLM"}
    ],
)
print(resp.content[0].text)
```

### 5.4 messages 接口三种角色

```python
messages = [
    {"role": "user", "content": "你好"},          # 用户
    {"role": "assistant", "content": "您好！"},    # 模型上一轮
    {"role": "user", "content": "讲个笑话"},
]

resp = client.messages.create(
    model="claude-sonnet-4-5",
    system="你是一位幽默的相声演员，回答都用包袱。",  # ← system 单独传
    max_tokens=512,
    messages=messages,
)
```

注意：**system 不放在 messages 里**，是顶级参数。

### 5.5 关键参数

| 参数 | 作用 | 建议 |
|---|---|---|
| `temperature` | 0=确定，1=随机 | 代码/事实 0，创作 0.7 |
| `top_p` | 核采样 | 一般保持默认 |
| `max_tokens` | 最长输出 | 设上限，防失控 |
| `stop_sequences` | 遇到就停 | 自定义边界 |

### 5.6 流式输出（SSE）

```python
with client.messages.stream(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "写一首关于秋天的诗"}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

底层是 Server-Sent Events，每个 `event: content_block_delta` 携带一片 token。

### 5.7 Extended Thinking（推理模式）

Sonnet 4.5 支持显式"先想再答"：

```python
resp = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=8192,
    thinking={"type": "enabled", "budget_tokens": 4000},
    messages=[{"role": "user", "content": "证明根号 2 是无理数"}],
)
for block in resp.content:
    if block.type == "thinking":
        print("【思考】", block.thinking)
    elif block.type == "text":
        print("【回答】", block.text)
```

`budget_tokens` 是模型可以"在脑子里写草稿"的预算。

### 5.8 错误处理

| 状态码 | 含义 | 处理 |
|---|---|---|
| 400 | 请求格式错 | 修代码 |
| 401 | key 错 | 检查 env |
| 429 | 速率限制 | 退避重试 |
| 529 | 服务器忙 | 退避重试 |

```python
import time, anthropic
def call_with_retry(fn, max_try=5):
    for i in range(max_try):
        try:
            return fn()
        except (anthropic.RateLimitError, anthropic.APIStatusError) as e:
            if e.status_code in (429, 529) and i < max_try - 1:
                time.sleep(2 ** i)
                continue
            raise
```

---

## 6. 多模态：图片与 PDF

### 6.1 图片输入

```python
import base64, httpx
img = base64.standard_b64encode(httpx.get("https://...").content).decode()
resp = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64",
             "media_type": "image/jpeg", "data": img}},
            {"type": "text", "text": "图里有几只猫？"},
        ],
    }],
)
```

### 6.2 PDF 输入

```python
{"type": "document", "source": {"type": "base64",
 "media_type": "application/pdf", "data": pdf_b64}}
```

Claude 会把 PDF 当成"图 + 文"理解，长 PDF 自动分页。

---

## 7. Tokenizer 与成本

### 7.1 BPE 速览

LLM 不直接看字符，先把文本切成 **token**（一段子词）。Claude/GPT 用 BPE 算法：高频组合合并成一个 id。

经验值（英文）：1 token ≈ 0.75 词；（中文）：1 字 ≈ 1.5-2 token。

### 7.2 算 token

```python
# Anthropic 提供 count_tokens
n = client.messages.count_tokens(
    model="claude-sonnet-4-5",
    messages=[{"role": "user", "content": "你好"}],
)
print(n.input_tokens)
```

### 7.3 成本心算公式

```
成本 = input_tokens × 单价_in + output_tokens × 单价_out
       - cached_tokens × (1 - 0.1)   # 命中缓存只收 10%
```

每次调用前估一估，月底就不会被账单吓到。

---

## 8. Prompt Caching（重点！能省 90%）

如果你的 system prompt 很长（比如塞了一本说明书），每次都重传太浪费。Anthropic 让你**缓存前缀**：

```python
resp = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    system=[
        {"type": "text", "text": "You are a helpful assistant."},
        {"type": "text", "text": LONG_DOC,
         "cache_control": {"type": "ephemeral"}},   # ← 缓存这一段
    ],
    messages=[{"role": "user", "content": "总结第三章"}],
)
```

第一次写入贵 25%，之后 5 分钟内命中只收 10%。RAG / Agent / 长 system 必开。

---

## 9. 用 Ollama 跑本地 Llama 4

```bash
brew install ollama
ollama serve &
ollama pull llama4
ollama run llama4 "你好"
```

也提供 OpenAI 兼容 API：

```python
from openai import OpenAI
local = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")
print(local.chat.completions.create(
    model="llama4",
    messages=[{"role": "user", "content": "你好"}],
).choices[0].message.content)
```

M2/M3 Mac 16G 跑 8B 模型流畅，70B 需要 64G+。

---

## 10. 实战：chat CLI

需求：命令行聊天，支持历史、流式、`/model` 切换、`/clear` 清空。

```python
# chat_cli.py
import os, sys
from anthropic import Anthropic

MODELS = {
    "1": "claude-haiku-4-5",
    "2": "claude-sonnet-4-5",
    "3": "claude-opus-4-5",
}

def main():
    client = Anthropic()
    model = MODELS["2"]
    history = []
    system = "你是一位耐心的中文助教。"
    print(f"模型 {model}，输入 /help 查看命令")
    while True:
        try:
            user = input("\n你> ").strip()
        except (EOFError, KeyboardInterrupt):
            break
        if not user:
            continue
        if user.startswith("/"):
            cmd, *arg = user.split(maxsplit=1)
            if cmd == "/help":
                print("/model 1|2|3  /clear  /quit"); continue
            if cmd == "/model" and arg and arg[0] in MODELS:
                model = MODELS[arg[0]]; print("已切到", model); continue
            if cmd == "/clear":
                history = []; print("历史已清空"); continue
            if cmd == "/quit":
                break
            print("未知命令"); continue

        history.append({"role": "user", "content": user})
        print("AI> ", end="", flush=True)
        with client.messages.stream(
            model=model, max_tokens=1024, system=system, messages=history,
        ) as stream:
            chunks = []
            for t in stream.text_stream:
                print(t, end="", flush=True); chunks.append(t)
        history.append({"role": "assistant", "content": "".join(chunks)})

if __name__ == "__main__":
    main()
```

跑：`python chat_cli.py`，体验流式吐字。

---

## 11. 练习

1. 把上面 CLI 加上 `/save filename.md`，把对话存成 Markdown。
2. 给 system prompt 加一段 5000 字的"使用手册"，开 prompt cache，比较 1/2/10 次调用账单。
3. 写一个脚本：读取一张图（图里是手写算式），调 Sonnet 4.5 算出结果。
4. 在 Ollama 上跑 Llama 4，让它和 Claude Haiku 回答同一个问题，肉眼对比。
5. 用 `count_tokens` 估算一份 100 页 PDF 塞进 1M 上下文要花多少钱。

---

## 12. Checklist

- [ ] 能说清 Pretrain / SFT / RLHF 三阶段
- [ ] 能背出 2026 年 5 家主流厂商的旗舰名字
- [ ] API Key 已配好，第一个请求跑通
- [ ] system / user / assistant 三角色不混
- [ ] 流式输出能跑
- [ ] Extended thinking 至少试过一次
- [ ] 知道 429 / 529 怎么处理
- [ ] 多模态（图 + PDF）至少各试一次
- [ ] 用 count_tokens 算过一次成本
- [ ] prompt caching 能正确写 cache_control
- [ ] Ollama 本地跑通一个开源模型
- [ ] chat CLI 完成且支持流式 + 切模型

读完这一周，你已经站在 LLM 应用开发的起跑线上。下一周我们让 Prompt 从"能用"变成"好用"。
