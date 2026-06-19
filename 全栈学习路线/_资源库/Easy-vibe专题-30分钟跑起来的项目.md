---
title: Easy-vibe 专题 - 30 分钟跑起来的项目
type: 资源库 / 专题清单
tags: [easy-vibe, 小项目, 顿悟, 源码学习, 练手]
难度: 入门到进阶
更新时间: 2026-06-19
---

# Easy-vibe 专题：30 分钟跑起来的项目

## 什么叫 Easy-vibe 风格

Easy-vibe 是一种项目气质，不是分类。它的特征只有四条：

1. **30 分钟跑起来**：clone、装依赖、跑通 demo，半小时内一定能看到效果。
2. **代码量小**：核心逻辑通常 100~2000 行，一个下午能读完。
3. **改造空间大**：不是黑盒，每一行都能看懂、能改、能延伸。
4. **看完有顿悟**：不是「学了一个 API」，而是「我理解了某件事的本质」。

巨型项目（Linux 内核、Chromium、TensorFlow）当然伟大，但对于学习者，它们更像博物馆——你能看，但摸不着。Easy-vibe 项目相反，它们是工坊，每件作品都还冒着热气。

## 为什么对学习者价值巨大

- **打破"我也能写"的心理门槛**：当你看完 150 行的 micrograd，你会意识到反向传播不是黑魔法。
- **建立"小切片"思维**：复杂系统都是由可独立理解的小切片组成。
- **快速反馈**：跑通的成就感是学习的最大燃料。
- **可改造可炫耀**：在简历和博客里，"我手写了一个 X" 永远比 "我用了 X" 更打动人。

下面按学习阶段分组。每个项目我都标注了**核心代码行数**和**预估跑通时间**。

---

## 🐣 完全零基础阶段（看完即懂）

### karpathy/micrograd（必看）

- **行数**：~150 行 Python
- **跑通时间**：20 分钟
- **学到的东西**：自动微分的本质，反向传播是怎么"自动"的
- **30 分钟跑通清单**：
  1. `git clone https://github.com/karpathy/micrograd`
  2. `pip install numpy` （甚至 numpy 都不强制）
  3. 打开 `demo.ipynb`，逐 cell 运行
  4. 阅读 `engine.py`（约 100 行）
  5. 阅读 `nn.py`（约 50 行）
  6. 在白纸上手写一遍 `Value` 类的 `__mul__` 和 `backward`

### karpathy/makemore

- **行数**：~500 行
- **跑通时间**：30 分钟
- **学到的东西**：从 bigram 到 Transformer 的渐进式建模思路
- **30 分钟跑通清单**：
  1. clone 仓库
  2. `python makemore.py -i names.txt -o names`
  3. 看到 loss 下降、生成新名字
  4. 切换 `--type` 体验 mlp / rnn / gru / transformer
  5. 把 `names.txt` 换成中文姓名重训练一次

### norvig/pytudes/spell.py

- **行数**：21 行（不含注释）
- **跑通时间**：5 分钟
- **学到的东西**：概率拼写纠正的优雅实现，Peter Norvig 的写作哲学
- **30 分钟跑通清单**：
  1. 下载 `spell.py` 和 `big.txt`
  2. `python -i spell.py`
  3. `correction('speling')` → `'spelling'`
  4. 阅读全部 21 行，理解 `edits1` / `known` / `candidates`
  5. 把英文语料换成中文，体会失败原因

---

## 🌱 入门阶段（练手 + 改造）

| 项目 | 内容 | 适合谁 |
|---|---|---|
| **bradtraversy/50projects50days** | HTML/CSS/JS 50 个小项目 | 前端入门，每个 30 分钟 |
| **florinpop17/app-ideas** | 700+ 项目灵感按难度分级 | 所有阶段，灵感库 |
| **john-smilga** 系列 | YouTube 大量视频配套代码 | 跟做学习者 |
| **30-Day-Vanilla-JS-Coding-Challenge** | 30 天纯 JS 挑战 | 巩固原生 JS |
| **TwoSetAI/lecture-summarizer** | 视频字幕摘要小工具 | 接触 LLM 应用 |

**入门阶段建议节奏**：每天一个，连做 30 天。不要"看懂"就过，每个都要 fork、改一处、push。

---

## 🌿 进阶阶段（小框架 / 小工具）

读小框架的源码，是从"会用"跨到"会写"的关键一步。

### 推荐清单

- **expressjs/express**：Node.js 老牌轻量 Web 框架。读 `lib/router/` 即可理解中间件本质。
- **honojs/hono**：Edge 优先现代 Web 框架，源码 < 5000 行，TypeScript 写得极干净。
- **pmndrs/zustand**：状态管理库，~1000 行 TS。看完你会笑 Redux 的复杂。
- **pomber/didact**：300 行手写 React，配 [博客](https://pomb.us/build-your-own-react/) 食用。
- **BetaSu/big-react**：卡颂老师手写 React，按 commit 学习。
- **cuixiaorui/mini-vue**：手写 Vue 3，跟随测试驱动开发。
- **hiroppy/the-sample-of-module-federation**：Webpack Module Federation 最小可运行例。
- **simonw/sqlite-utils**：单人开发的高质量 Python 工具，结构教科书级。

### 阅读这一阶段项目的方法

1. 先跑 demo，看现象。
2. 找一个核心 API（比如 `app.use`、`useState`），从 README 反推到入口。
3. 用 IDE 的 Go to Definition 顺着调用栈走一遍。
4. 在自己的笔记本上画一张调用图。
5. 删掉一个文件，看它会哪里报错——这是学结构的最快方法。

---

## 🔥 LLM / Agent 阶段（吃透 AI 时代）

这是 2026 年最值得吃透的方向。

| 项目 | 行数 | 学到什么 |
|---|---|---|
| **karpathy/nanoGPT** | ~300 行 | 完整 GPT 训练循环 |
| **karpathy/llm.c** | C + CUDA | 不依赖 PyTorch 的 LLM |
| **karpathy/llama2.c** | ~700 行 | 推理引擎本质 |
| **rasbt/LLMs-from-scratch** | 章节代码 | 从零搭 LLM 的教科书 |
| **anthropic-cookbook/patterns/agents** | ~200 行 / 模式 | 5 个核心 Agent 模式 |
| **assafelovic/gpt-researcher** | 中等 | 研究型 Agent 结构 |
| **paul-gauthier/aider** Tier 1 file | 单文件入口 | 命令行编码 Agent |
| **Doriandarko/o1-engineer** | 小 | 推理模型工程化 |
| **simonw/llm** | 中等 | 命令行 LLM 工具链 |

**强烈建议路径**：先 nanoGPT 跑通 → 读 anthropic-cookbook 5 模式 → 自己实现一个最小 Agent。

---

## 🎯 专题：30 分钟做完一个 Web 应用

这一类适合在面试前一周临时抱佛脚，或者周末做个能 demo 的副业。

- **vercel/ai-chatbot**：Next.js + AI SDK 官方模板，部署到 Vercel 5 分钟出活。
- **vercel/v0** 灵感站：用 v0 生成 UI 草稿再改造。
- **t3-oss/create-t3-app**：Next.js + tRPC + Prisma + Tailwind 全家桶。
- **openai/openai-quickstart-node**：最小可运行 OpenAI 调用模板。

---

## 📋 跟做模板（通用六步）

每跟做一个 Easy-vibe 项目，都按这个模板走：

1. **fork 仓库**到自己账号
2. **30 分钟内跑通 demo**（如果跑不通，记录卡点写到 issue 里）
3. **默写一遍核心 100 行**（关电脑、白纸或 Notion 凭记忆写）
4. **改造**：增加 1 个新功能（比如给 micrograd 加 `tanh` 算子）
5. **写复盘博客**（哪怕只发到自己的 Obsidian）
6. **把改造版 push 自己仓库**，README 里链接原作者

走完这六步，你就真的"拥有"了这个项目，而不是"看过"它。

---

## ⭐ 我的"小项目大顿悟"清单 Top 10

| # | 项目 | 顿悟点 |
|---|---|---|
| 1 | micrograd | 反向传播只是链式法则 + 拓扑排序 |
| 2 | nanoGPT | Transformer = Attention + 残差 + LayerNorm，没别的 |
| 3 | pomber/didact | React 的核心是 fiber 链表 + 协调算法 |
| 4 | norvig/spell.py | 概率模型可以 21 行写完 |
| 5 | hono | Web 框架本质是路由表 + 中间件管道 |
| 6 | zustand | 状态管理只需一个发布订阅 + 浅比较 |
| 7 | sqlite-utils | 命令行工具的可读性可以美如散文 |
| 8 | cuixiaorui/mini-vue | 响应式 = Proxy + 依赖收集 |
| 9 | llama2.c | 推理就是矩阵乘 + softmax 循环 |
| 10 | aider 单文件版 | Agent 就是 LLM + 工具调用 + 循环 |

---

## 心法

不要只收藏，要**真的跑**。一个跑过、改过、写过博客的小项目，胜过 100 个只 star 的大项目。

Easy-vibe 项目的最大价值，是让你重新相信：**软件是人写出来的，你也是人**。
