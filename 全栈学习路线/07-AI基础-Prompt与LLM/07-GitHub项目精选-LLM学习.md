---
title: GitHub 项目精选 - LLM 学习
chapter: 07
type: github-curation
tags: [github, llm, prompt-engineering, rag, fine-tuning, multimodal, evaluation]
created: 2026-06-19
updated: 2026-06-19
---

# 07 章配套 · GitHub 项目精选：LLM 学习

> 配套第 07 章「AI 基础 - Prompt 与 LLM」的 GitHub 仓库精选清单。
> 难度图例：🟢 入门 · 🟡 进阶 · 🔴 高阶 · ⚫ 源码级
> 中英标识：🇨🇳 中文友好 · 🌍 英文为主

---

## 🌱 LLM 入门（先读这一组，少踩坑）

### anthropics/courses ⭐ 必做
- URL：https://github.com/anthropics/courses
- Star：约 14k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：5 门官方课程：API fundamentals、prompt engineering interactive、real-world prompting、prompt evaluations、tool use。
- 推荐理由：Anthropic 官方写的 Claude 课程，目前市面上最系统的"会用 LLM"教材。每节课都是 Jupyter，一边读一边跑。
- 中文：🌍 英文为主，配合 datawhalechina/llm-cookbook 中文版可对照。

### anthropics/anthropic-cookbook ⭐ 必读
- URL：https://github.com/anthropics/anthropic-cookbook
- Star：约 14k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：`patterns/agents/` 子目录是写 Agent 的圣经；`misc/` 含 PDF/RAG/citation 等实战。
- 推荐理由：每个 notebook 都是工程师写给工程师，没废话。

### openai/openai-cookbook
- URL：https://github.com/openai/openai-cookbook
- Star：约 64k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：function calling、batch API、structured output、Assistants 实战。
- 推荐理由：和 anthropic-cookbook 是一对，看哪家就追哪家。

### datawhalechina/llm-cookbook 🇨🇳 ⭐
- URL：https://github.com/datawhalechina/llm-cookbook
- Star：约 21k
- 难度：🟢 入门
- 学习要点：吴恩达 ChatGPT Prompt Engineering / Building Systems / LangChain 三门课的中文翻译 + Notebook。
- 推荐理由：中文学习者的 LLM 第一站。

### datawhalechina/llm-universe 🇨🇳
- URL：https://github.com/datawhalechina/llm-universe
- Star：约 9k
- 难度：🟢 入门
- 学习要点：从 0 搭建一个本地 RAG 应用，覆盖文档加载、chunk、embed、检索、评测。
- 推荐理由：手把手的中文 RAG 入门项目。

### datawhalechina/prompt-engineering-for-developers 🇨🇳
- URL：https://github.com/datawhalechina/prompt-engineering-for-developers
- Star：约 13k
- 难度：🟢 入门
- 学习要点：吴恩达 Prompt 课程中文化，比 llm-cookbook 更聚焦 prompt。
- 推荐理由：3 小时刷完 = 入门毕业。

### microsoft/generative-ai-for-beginners
- URL：https://github.com/microsoft/generative-ai-for-beginners
- Star：约 85k
- 难度：🟢 入门
- 学习要点：21 节课，每节有视频 + Notebook + 任务，覆盖文本/图像/RAG/Agent。
- 推荐理由：微软出品，结构最完整的"什么是 GenAI"通识课。

### microsoft/AI-Agents-for-Beginners ⭐ 重点
- URL：https://github.com/microsoft/AI-Agents-for-Beginners
- Star：约 15k（新出，增长极快）
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：10 节课覆盖 ReAct、Planning、Tool use、Multi-agent、AutoGen 实战。
- 推荐理由：07 章学完就接 08 章 Agent 的最佳过渡仓库。

---

## 🌿 Prompt 工程

### f/awesome-chatgpt-prompts 🟢
- URL：https://github.com/f/awesome-chatgpt-prompts
- Star：约 117k
- 难度：🟢 入门
- 学习要点：上千个角色 prompt 模板。
- 推荐理由：写 system prompt 没灵感时来抄。

### promptslab/Awesome-Prompt-Engineering 🟡
- URL：https://github.com/promptslab/Awesome-Prompt-Engineering
- Star：约 4k
- 难度：🟡 进阶
- 学习要点：论文 + 工具 + tutorial 综合索引。
- 推荐理由：研究向，深入 CoT/ToT/ReAct 概念时来翻。

### dair-ai/Prompt-Engineering-Guide ⭐
- URL：https://github.com/dair-ai/Prompt-Engineering-Guide
- Star：约 52k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：从 Zero-shot → Few-shot → CoT → Self-Consistency → ReAct 体系化梳理。
- 推荐理由：网页版（promptingguide.ai）就是它的渲染结果，业界最被引用的 prompt 教程。

### brexhq/prompt-engineering 🟡
- URL：https://github.com/brexhq/prompt-engineering
- Star：约 9k
- 难度：🟡 进阶
- 学习要点：Brex 实战经验，工程师视角。
- 推荐理由：500 行 markdown，浓缩生产经验。

### anthropics/prompt-eng-interactive-tutorial ⭐
- URL：https://github.com/anthropics/prompt-eng-interactive-tutorial
- Star：约 16k
- 难度：🟢 入门
- 学习要点：9 章交互式 Notebook，从 basic → advanced（含 example workbook）。
- 推荐理由：Anthropic 自家最体系的 prompt 教材，比博客深。

---

## 🌲 LLM 原理 / 从零实现

### rasbt/LLMs-from-scratch ⭐ 必做
- URL：https://github.com/rasbt/LLMs-from-scratch
- Star：约 45k
- 难度：🟡 进阶
- 学习要点：从 tokenizer → attention → transformer → pretrain → fine-tune → instruction-tuning，全流程 PyTorch 写。
- 推荐理由：Sebastian Raschka 著《Build a Large Language Model (From Scratch)》配套；07 章想"懂内部"必读。

### karpathy/nanoGPT ⭐
- URL：https://github.com/karpathy/nanoGPT
- Star：约 39k
- 难度：🟡 进阶
- 学习要点：~300 行 PyTorch 训出 GPT-2；`train.py` + `model.py` 通读 1 小时。
- 推荐理由：Karpathy 的代码就是教材本身。

### karpathy/llm.c ⭐
- URL：https://github.com/karpathy/llm.c
- Star：约 26k
- 难度：🔴 高阶
- 学习要点：纯 C/CUDA 实现 GPT-2 训练，不依赖 PyTorch。
- 推荐理由：想搞懂"框架到底为我做了什么"，看这个。

### karpathy/build-nanogpt
- URL：https://github.com/karpathy/build-nanogpt
- Star：约 7k
- 难度：🟡 进阶
- 学习要点：4 小时 YouTube 视频 "Let's reproduce GPT-2 (124M)" 的代码。
- 推荐理由：边看边敲，效果拔群。

### karpathy/micrograd
- URL：https://github.com/karpathy/micrograd
- Star：约 12k
- 难度：🟢 入门
- 学习要点：~150 行 Python 写出自动微分 + 神经网络。
- 推荐理由：5 小时通读，永久解决"反向传播是什么"困惑。

### ml-explore/mlx
- URL：https://github.com/ml-explore/mlx
- Star：约 18k
- 难度：🟡 进阶
- 学习要点：Apple Silicon 原生张量框架，统一内存模型；`mlx-examples/` 含 LLM 推理 demo。
- 推荐理由：MacBook 用户跑本地模型最爽路径。

### ggerganov/llama.cpp ⭐
- URL：https://github.com/ggerganov/llama.cpp
- Star：约 75k
- 难度：🟡 进阶 → ⚫ 源码级
- 学习要点：GGUF 格式、量化（Q4_K_M 等）、CPU/GPU 后端切换。
- 推荐理由：一台 MacBook 就能跑 70B 模型的"魔法"，源码可读。

### vllm-project/vllm
- URL：https://github.com/vllm-project/vllm
- Star：约 38k
- 难度：🔴 高阶
- 学习要点：PagedAttention（受 OS 虚拟内存启发）、continuous batching。
- 推荐理由：生产级推理服务事实标准。

---

## 🔍 RAG 完整方案

### langchain-ai/langchain
- URL：https://github.com/langchain-ai/langchain
- Star：约 95k
- 难度：🟡 进阶
- 学习要点：document loader、splitter、retriever、chain、agent。
- 推荐理由：好坏争议大，但仍是 RAG 工程化的事实起点。

### run-llama/llama_index
- URL：https://github.com/run-llama/llama_index
- Star：约 38k
- 难度：🟡 进阶
- 学习要点：以"数据 → 索引 → 查询"为核心抽象；多种高级检索（hybrid、auto-merge、recursive）。
- 推荐理由：纯 RAG 场景比 langchain 更聚焦。

### infiniflow/ragflow ⭐ 🇨🇳
- URL：https://github.com/infiniflow/ragflow
- Star：约 30k
- 难度：🟡 进阶
- 学习要点：深度文档解析（含表格、版面）；可视化 RAG pipeline。
- 推荐理由：中文场景下文档解析最强的开源 RAG。

### QuivrHQ/quivr
- URL：https://github.com/QuivrHQ/quivr
- Star：约 37k
- 难度：🟡 进阶
- 学习要点：个人"Second Brain"，多 brain 并存。
- 推荐理由：把自己 Obsidian 喂进去就能问。

### danny-avila/LibreChat
- URL：https://github.com/danny-avila/LibreChat
- Star：约 25k
- 难度：🟡 进阶
- 学习要点：多 Provider（OpenAI/Claude/Gemini/本地）统一前端；自带 Auth、文件、Agents。
- 推荐理由：自托管 ChatGPT 平替的工业级方案。

### lobehub/lobe-chat ⭐ 🇨🇳
- URL：https://github.com/lobehub/lobe-chat
- Star：约 50k
- 难度：🟡 进阶
- 学习要点：Next.js 14 工程标杆；插件系统、知识库、TTS/STT 全。
- 推荐理由：中文社区最活跃的开源 ChatGPT 替代品。

### mckaywrigley/chatbot-ui
- URL：https://github.com/mckaywrigley/chatbot-ui
- Star：约 29k
- 难度：🟢 入门
- 学习要点：极简 Next.js + Supabase 全栈聊天 UI。
- 推荐理由：2 小时部署一个属于自己的 chat。

### vercel/ai
- URL：https://github.com/vercel/ai
- Star：约 11k
- 难度：🟢 入门
- 学习要点：`useChat` / `useCompletion` Hook、流式 UI、tool call 协议。
- 推荐理由：在 Next.js 里加 AI 的标准 SDK。

---

## 📊 评测

### promptfoo/promptfoo
- URL：https://github.com/promptfoo/promptfoo
- Star：约 5k
- 难度：🟢 入门
- 学习要点：YAML 配置 + CLI 跑批评测；红队（red-team）模式。
- 推荐理由：从"凭感觉"升级到"数据驱动"调 prompt。

### explodinggradients/ragas
- URL：https://github.com/explodinggradients/ragas
- Star：约 7k
- 难度：🟡 进阶
- 学习要点：Faithfulness、Answer Relevancy、Context Precision/Recall 四件套。
- 推荐理由：RAG 评测工业标准。

### UKGovernmentBEIS/inspect_ai
- URL：https://github.com/UKGovernmentBEIS/inspect_ai
- Star：约 1k
- 难度：🟡 进阶
- 学习要点：UK AISI 出品，Solver/Scorer/Task 三件套。
- 推荐理由：Anthropic、OpenAI 都用的 safety eval 框架。

### confident-ai/deepeval
- URL：https://github.com/confident-ai/deepeval
- Star：约 4k
- 难度：🟢 入门
- 学习要点：pytest 风格，14 种内置 metric。
- 推荐理由：写过 pytest 的人 5 分钟上手。

### EleutherAI/lm-evaluation-harness
- URL：https://github.com/EleutherAI/lm-evaluation-harness
- Star：约 8k
- 难度：🟡 进阶
- 学习要点：HuggingFace Open LLM Leaderboard 的底层；MMLU/GSM8K/HellaSwag 跑分。
- 推荐理由：模型层（不是应用层）评测的事实标准。

---

## 🛠️ 微调 / 训练

### unslothai/unsloth ⭐
- URL：https://github.com/unslothai/unsloth
- Star：约 17k
- 难度：🟡 进阶
- 学习要点：Triton kernel 重写注意力，2× 速度、50% 显存；Colab 友好。
- 推荐理由：单卡微调 Llama/Qwen/Phi 最快路径。

### huggingface/transformers
- URL：https://github.com/huggingface/transformers
- Star：约 135k
- 难度：🟡 进阶 → ⚫ 源码级
- 学习要点：PreTrainedModel / Tokenizer / Trainer 三件套。
- 推荐理由：NLP 时代的 PyTorch，现在是 LLM 时代的"操作系统"。

### huggingface/peft
- URL：https://github.com/huggingface/peft
- Star：约 16k
- 难度：🟡 进阶
- 学习要点：LoRA、QLoRA、Prefix Tuning、IA3 实现。
- 推荐理由：搞清楚"为什么我能在 RTX 3090 上微调 7B"。

### hiyouga/LLaMA-Factory ⭐ 🇨🇳
- URL：https://github.com/hiyouga/LLaMA-Factory
- Star：约 35k
- 难度：🟢 入门 → 🟡 进阶
- 学习要点：图形界面 + 100+ 模型支持；SFT、DPO、ORPO、KTO 一键。
- 推荐理由：中文社区最易用的微调工具，无脑 GUI。

### axolotl-ai-cloud/axolotl
- URL：https://github.com/axolotl-ai-cloud/axolotl
- Star：约 8k
- 难度：🟡 进阶
- 学习要点：YAML 驱动的微调，多 GPU/DeepSpeed/FSDP 都支持。
- 推荐理由：生产环境多机多卡微调首选。

---

## 🎤 多模态

### openai/whisper
- URL：https://github.com/openai/whisper
- Star：约 73k
- 难度：🟢 入门
- 学习要点：encoder-decoder 语音识别；多语言支持。
- 推荐理由：本地 ASR 事实标准。

### suno-ai/bark
- URL：https://github.com/suno-ai/bark
- Star：约 36k
- 难度：🟡 进阶
- 学习要点：Transformer-based TTS，能做笑声、叹气等非语言音。
- 推荐理由：玩起来最有意思的 TTS。

### comfyanonymous/ComfyUI ⭐
- URL：https://github.com/comfyanonymous/ComfyUI
- Star：约 65k
- 难度：🟡 进阶
- 学习要点：节点式图像生成；LoRA、ControlNet、IPAdapter 全。
- 推荐理由：图像生成的 Photoshop。

### AUTOMATIC1111/stable-diffusion-webui
- URL：https://github.com/AUTOMATIC1111/stable-diffusion-webui
- Star：约 145k
- 难度：🟢 入门
- 学习要点：开箱即用 Web UI；插件生态最大。
- 推荐理由：第一次玩 SD 就用它。

### Stability-AI/StableDiffusion
- URL：https://github.com/Stability-AI/StableDiffusion
- Star：约 68k
- 难度：🟡 进阶
- 学习要点：官方 reference 实现。
- 推荐理由：理解 latent diffusion 直接读官方代码。

---

## 🏆 Easy-vibe LLM 项目（30 分钟跑起来 + 可改造）

### Shubhamsaboo/awesome-llm-apps ⭐ 必逛
- URL：https://github.com/Shubhamsaboo/awesome-llm-apps
- Star：约 45k
- 难度：🟢 入门
- 学习要点：100+ 实例代码：RAG、Agent、Tool use、Multimodal、Memory，每个 50-200 行。
- 推荐理由：LLM 时代的"鸿渐 cookbook"，挑一个改一个。

### mlabonne/llm-course ⭐
- URL：https://github.com/mlabonne/llm-course
- Star：约 38k
- 难度：🟡 进阶
- 学习要点：3 条路径（Fundamentals/Scientist/Engineer）；含交互式 LLM roadmap。
- 推荐理由：博客质量高，roadmap 截图常被引用。

### lavague-ai/LaVague
- URL：https://github.com/lavague-ai/LaVague
- Star：约 6k
- 难度：🟢 入门
- 学习要点：用自然语言驱动 Selenium 浏览器。
- 推荐理由：迷你 Web Agent，源码可一晚读完。

### simonw/llm
- URL：https://github.com/simonw/llm
- Star：约 5k
- 难度：🟢 入门
- 学习要点：`llm "explain X"`、`llm chat`、模板、log 到 SQLite。
- 推荐理由：命令行 LLM 工具的优雅范本，作者 Simon Willison 是行业 KOL。

### simonw/datasette-llm
- URL：https://github.com/simonw/datasette-llm
- Star：约 0.3k
- 难度：🟢 入门
- 学习要点：Datasette 网页里直接对 SQL 数据 LLM 问答。
- 推荐理由：理解"LLM + 结构化数据"工程化的简洁示例。

---

## 📚 必读论文 / 综述仓库

### Hannibal046/Awesome-LLM
- URL：https://github.com/Hannibal046/Awesome-LLM
- Star：约 23k
- 难度：🟡 进阶
- 学习要点：模型、论文、数据集、tutorial 大全索引。
- 推荐理由：每周末刷一次保持信息流新鲜。

### mlabonne/llm-datasets
- URL：https://github.com/mlabonne/llm-datasets
- Star：约 3k
- 难度：🟡 进阶
- 学习要点：精选的 SFT/DPO/RLHF 数据集索引。
- 推荐理由：自己做微调时数据从哪来？这里。

### NielsRogge/Transformers-Tutorials
- URL：https://github.com/NielsRogge/Transformers-Tutorials
- Star：约 11k
- 难度：🟡 进阶
- 学习要点：HuggingFace 工程师本人写的 200+ Notebook，几乎覆盖所有热门视觉/语言模型。
- 推荐理由：HuggingFace 模型怎么用，看这个比文档清晰。

---

## 🗓️ 60 天 LLM 学习每周读哪个仓库

> 建议每周聚焦 1-2 个仓库；不必通读，刷主线 README + 跑 1 个示例。

| 周 | 主题 | 主仓库 | 副仓库 |
|----|------|--------|--------|
| W1 | API 基础 | anthropics/courses | openai/openai-cookbook |
| W2 | Prompt 入门 | anthropics/prompt-eng-interactive-tutorial | dair-ai/Prompt-Engineering-Guide |
| W3 | Prompt 进阶 | brexhq/prompt-engineering | anthropics/anthropic-cookbook |
| W4 | 中文化补强 | datawhalechina/llm-cookbook | datawhalechina/prompt-engineering-for-developers |
| W5 | 通识扫盲 | microsoft/generative-ai-for-beginners | — |
| W6 | 神经网络基础 | karpathy/micrograd | karpathy/build-nanogpt |
| W7 | 从零写 GPT | karpathy/nanoGPT | rasbt/LLMs-from-scratch（Ch1-3）|
| W8 | 继续写 GPT | rasbt/LLMs-from-scratch（Ch4-7）| — |
| W9 | 推理引擎 | ggerganov/llama.cpp | ml-explore/mlx |
| W10 | 生产推理 | vllm-project/vllm | — |
| W11 | RAG 入门 | datawhalechina/llm-universe | langchain-ai/langchain（quickstart）|
| W12 | RAG 工程化 | run-llama/llama_index | infiniflow/ragflow |
| W13 | 评测 | promptfoo/promptfoo | explodinggradients/ragas |
| W14 | 微调 | unslothai/unsloth | hiyouga/LLaMA-Factory |
| W15 | 多模态 | openai/whisper + ComfyUI | — |
| W16 | 串起来 | Shubhamsaboo/awesome-llm-apps | mlabonne/llm-course |
| W17 | 过渡到 Agent | microsoft/AI-Agents-for-Beginners | anthropics/anthropic-cookbook（patterns/agents）|

学完即顺势进入第 08 章 Agent。

---

## 📌 学习心法

1. **先用再读**：所有仓库都先 clone + 跑 example，再看源码。
2. **一周一仓库**：贪多嚼不烂。
3. **写复盘**：每读一个，在自己 vault 里写 100 字"我学到了什么"。
4. **跟着 Karpathy + Anthropic + Sebastian Raschka 三个老师**：他们的代码可以当教材。

---

> 本文配套：第 07 章「AI 基础 - Prompt 与 LLM」 · 下一章：08 Agent。
