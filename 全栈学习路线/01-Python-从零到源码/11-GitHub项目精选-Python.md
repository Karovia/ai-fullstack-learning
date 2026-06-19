---
title: GitHub 项目精选 - Python
chapter: 01-Python-从零到源码
section: 11
tags: [GitHub项目, 资源]
---

# GitHub 项目精选 · Python

> 学 Python 的最大坑是：**只看官方教程**。Python 真正的精华在那些被全世界打磨过 10 年的开源库里。
> 
> 这一篇按"入门 → 进阶 → 源码级 → 工具/数据/AI"分层。每个项目都给"怎么用它学习"，让你的 fork 不只是收藏。

---

## 🌱 入门级 Python 项目（跟着改）

适合刚学完语法、想找"真实代码"练手的人。

### **TheAlgorithms/Python** ⭐ ~190k 🌱入门 [英文]
一句话：用 Python 实现的"算法百科全书"，从冒泡到神经网络。

- GitHub: https://github.com/TheAlgorithms/Python
- **怎么用它学习**：
  1. 不要从头读，挑你这周在学的算法（比如二分查找）
  2. **先盖住代码自己写一遍**
  3. 对比作者实现，注意类型注解、docstring、测试三件套
  4. 你会发现"原来工程级 Python 长这样"
- 推荐理由：这是新手最早能感受到"代码风格美感"的项目。每个文件都像一篇短文。

### **jackfrued/Python-100-Days** ⭐ ~155k 🌱入门 [中文]
一句话：100 天从 Python 新手到大师（中文）。

- GitHub: https://github.com/jackfrued/Python-100-Days
- **怎么用它学习**：
  1. 不要真的跑 100 天，挑你需要的章节读
  2. Day 01-15（基础）跳着读，Day 16-30（函数/OOP）必读
  3. Day 41 开始的爬虫、Web 开发部分对找工作最有用
- 推荐理由：中文 Python 教程的天花板，覆盖面广 + 实战导向。

### **geekcomputers/Python** ⭐ ~32k 🌱入门 [英文]
一句话：100+ 个超小 Python 脚本合集。

- GitHub: https://github.com/geekcomputers/Python
- **怎么用它学习**：每天挑一个脚本（比如"自动整理下载文件夹""批量改图片名"）跑一遍 + 改造。
- 推荐理由：让你立刻体会"Python 是工具不是学科"，把刚学的东西用起来。

### **realpython/python-basics-exercises** 🌱入门 [英文]
一句话：Real Python 出版社配套练习。

- GitHub: https://github.com/realpython/python-basics-exercises
- **怎么用它学习**：每章先自己做，再看官方答案。Real Python 的解法常常给你"还能这样写"的惊喜。
- 推荐理由：练习题质量高，出题人是 Python 老炮。

### **norvig/pytudes** ⭐ ~23k 🌿进阶 [英文]
一句话：Google 研究总监 Peter Norvig 写的 Python 谜题 + 解题笔记。

- GitHub: https://github.com/norvig/pytudes
- **怎么用它学习**：每周挑一个 Notebook 慢慢读。**重点不是答案，而是大佬的思路展开方式**。
- 推荐理由：写过《人工智能：一种现代方法》的 Norvig 教你"Python 思维"，这个经验无价。

### **donnemartin/data-science-ipython-notebooks** ⭐ ~28k 🌱入门 [英文]
一句话：数据科学相关 Notebook 大合集（pandas/numpy/scikit-learn 等）。

- GitHub: https://github.com/donnemartin/data-science-ipython-notebooks
- **怎么用它学习**：直接把 Notebook 在 Colab 打开 + 改代码 + 看效果，循环 100 次。
- 推荐理由：从语法到数据科学的最自然过渡桥梁。

---

## 🌿 进阶 Python 项目（读源码）

学语法到这里就够了，下面要培养的是"读别人代码的能力"。

### **psf/requests** ⭐ ~52k 🌿进阶 [英文]
一句话：Python HTTP 客户端事实标准，"for humans" 的代名词。

- GitHub: https://github.com/psf/requests
- **怎么用它学习**：
  1. 入口：`requests/api.py` 看 `get/post` 的几行实现
  2. 顺藤摸瓜到 `sessions.py`（核心）
  3. 看 `models.py` 学 Request/Response 抽象
  4. 用一周时间画出整个调用链路图
- 推荐理由：API 设计教科书。同一件事 urllib 写要 30 行，requests 一行。**这就是抽象的力量**。

### **pallets/flask** ⭐ ~68k 🌿进阶 [英文]
一句话：Python Web 微框架的优雅典范。

- GitHub: https://github.com/pallets/flask
- **怎么用它学习**：
  1. 先写一个 hello world，再去读 `flask/app.py`
  2. 重点看 `Flask.__call__` 和 `wsgi_app` 方法
  3. 跟一个请求从进来到响应的全流程
- 推荐理由：源代码注释 = 教学，被誉为"Python 项目的标杆"。

### **encode/httpx** ⭐ ~14k 🌿进阶 [英文]
一句话：requests 的现代继承者，支持 async + HTTP/2。

- GitHub: https://github.com/encode/httpx
- **怎么用它学习**：和 requests 对比读，**找出 async 在哪里、为什么需要它**。
- 推荐理由：学异步 Python 的最佳真实案例。

### **pydantic/pydantic** ⭐ ~22k 🌿进阶 [英文]
一句话：基于类型注解的数据校验库，FastAPI 的灵魂。

- GitHub: https://github.com/pydantic/pydantic
- **怎么用它学习**：
  1. 先用它（写 5 个 BaseModel）
  2. 再读 `_internal/_model_construction.py` 看元类怎么玩的
  3. 这是学 metaclass 最好的真实案例
- 推荐理由：让你彻底理解"Python 类型注解的运行时威力"。

### **tiangolo/fastapi** ⭐ ~80k 🌿进阶 [英文]
一句话：现代 Python Web 框架，性能/类型/文档三优。

- GitHub: https://github.com/tiangolo/fastapi
- **怎么用它学习**：
  1. 先用 FastAPI 写一个 CRUD（半天）
  2. 再读官方教程级别的源码（重点：`dependencies/utils.py` 依赖注入）
  3. 终极挑战：理解它怎么把 type hint 变成 OpenAPI 文档的
- 推荐理由：2020 后 Python Web 教科书，比 Flask 更现代但又不像 Django 那么重。

### **rich-click/rich** ⭐ ~50k 🌿进阶 [英文]
一句话：终端漂亮输出库，让你的 Python 脚本带颜色、表格、进度条。

- GitHub: https://github.com/Textualize/rich
- **怎么用它学习**：直接看 `examples/` 文件夹，每个例子 30 行教你一个特性。
- 推荐理由：让你以后写 CLI 脚本再也不想用 print。

### **astral-sh/uv** ⭐ ~32k 🌿进阶 [英文]
一句话：Rust 写的 Python 包管理器，比 pip 快 100 倍。

- GitHub: https://github.com/astral-sh/uv
- **怎么用它学习**：**第一时间用上**——`uv venv` `uv pip install` 替代 pip，体感 5 分钟。源码不用读（是 Rust），重点是它**改变了 Python 工具链**。
- 推荐理由：2024 年 Python 工具链事实新标准。

### **astral-sh/ruff** ⭐ ~36k 🌿进阶 [英文]
一句话：Rust 写的 Python linter & formatter，替代 flake8 + black + isort。

- GitHub: https://github.com/astral-sh/ruff
- **怎么用它学习**：装上、设个 pre-commit hook、忘记它存在。读 `ruff.toml` 看规则配置。
- 推荐理由：和 uv 一起，让 Python 工具链快到飞起。

### **python-poetry/poetry** ⭐ ~32k 🌿进阶 [英文]
一句话：现代 Python 依赖管理 + 打包一体化工具。

- GitHub: https://github.com/python-poetry/poetry
- **怎么用它学习**：用一周项目体会 `pyproject.toml` 的好。**对比 requirements.txt 你会回不去**。
- 推荐理由：理解"依赖锁文件"概念后，部署事故会少 80%。

---

## 🌲 源码级（CPython 阅读）

读到这里你就是"看 Python 源码的人"了。

### **python/cpython** ⭐ ~63k 🌲源码级 [英文]
一句话：Python 解释器本身的源码（C + Python）。

- GitHub: https://github.com/python/cpython
- **怎么用它学习**：
  1. **不要全读**，会疯
  2. 进 `Lib/` 文件夹，挑一个你常用的标准库（如 `pathlib.py`、`functools.py`）
  3. 读完一个文件写一篇博客
  4. 进阶：进 `Objects/listobject.c` 看 list 的 C 实现
- 推荐理由：能让你说出"Python 的 list 其实是 PyObject 数组"这种话。

### **rspeer/python-ftfy** ⭐ ~3.8k 🌿进阶 [英文]
一句话：自动修复"乱码"文本（mojibake）的小而美库。

- GitHub: https://github.com/rspeer/python-ftfy
- **怎么用它学习**：通读全部源码（不大），学"如何把一个小问题做到极致"。
- 推荐理由：小项目精读的经典样本。

### **pallets/click** ⭐ ~16k 🌿进阶 [英文]
一句话：Python CLI 框架，装饰器优雅写命令行。

- GitHub: https://github.com/pallets/click
- **怎么用它学习**：读 `core.py` 看装饰器怎么和命令树结合。**这是装饰器进阶的最佳教材**。
- 推荐理由：Flask 同作者，代码风格一脉相承。

### **encode/starlette** ⭐ ~10k 🌲源码级 [英文]
一句话：FastAPI 底层的 ASGI 框架。

- GitHub: https://github.com/encode/starlette
- **怎么用它学习**：FastAPI 写熟后再读它，你会发现"原来 FastAPI 90% 的魔法在这一层"。
- 推荐理由：学 ASGI / async Web 的最佳样本。

### **pyenv/pyenv** ⭐ ~38k 🌿进阶 [英文]
一句话：Python 多版本管理工具。

- GitHub: https://github.com/pyenv/pyenv
- **怎么用它学习**：**先装上用**（管理 3.10 / 3.11 / 3.12）。源码是 Bash，读 `bin/pyenv` 学 shell shim 模式。
- 推荐理由：管理 Python 多版本的事实标准。

---

## 🛠️ 命令行 / 爬虫工具

### **scrapy/scrapy** ⭐ ~52k 🌿进阶 [英文]
一句话：工业级爬虫框架。

- GitHub: https://github.com/scrapy/scrapy
- **怎么用它学习**：跟 tutorial 抓 quotes.toscrape.com，再换成你想抓的真实站点（注意 robots.txt）。读源码看 Twisted 异步模型。
- 推荐理由：从"会用 requests"到"懂爬虫架构"的桥梁。

### **psf/black** ⭐ ~39k 🌿进阶 [英文]
一句话："不可妥协"的 Python 代码格式化工具。

- GitHub: https://github.com/psf/black
- **怎么用它学习**：装上、配置、再也不想其他格式化工具。读源码学 AST 操作。
- 推荐理由：让团队代码风格之争终结的神器。

### **mwouts/jupytext** ⭐ ~7k 🌿进阶 [英文]
一句话：让 Notebook 文件能 git diff（双向同步 .ipynb 和 .py）。

- GitHub: https://github.com/mwouts/jupytext
- **怎么用它学习**：装上 + 配置，然后在 git 里看 Notebook diff——你会哭着感谢作者。
- 推荐理由：解决数据科学家"Notebook 没法 code review"的世纪难题。

---

## 📊 数据科学（顺带学）

### **pandas-dev/pandas** ⭐ ~43k 🌲源码级 [英文]
一句话：Python 数据处理事实标准库。

- GitHub: https://github.com/pandas-dev/pandas
- **怎么用它学习**：先用 100 小时（处理真实 CSV），再去看 `core/frame.py`。**没用过别读源码，会迷失**。
- 推荐理由：所有数据相关工作的基础。

### **numpy/numpy** ⭐ ~28k 🌲源码级 [英文]
一句话：Python 数值计算基石。

- GitHub: https://github.com/numpy/numpy
- **怎么用它学习**：用——读官方 quickstart 即可。源码涉及大量 C，新手不必啃。
- 推荐理由：理解"向量化思维"是迈入数据/AI 的门票。

### **streamlit/streamlit** ⭐ ~36k 🌿进阶 [英文]
一句话：用纯 Python 写 Web UI 的神奇库。

- GitHub: https://github.com/streamlit/streamlit
- **怎么用它学习**：用 50 行代码写一个数据 Dashboard，分享给同事。源码看它怎么把 Python 调用变成前端组件。
- 推荐理由：让数据/AI 工程师"前端零基础"也能交付产品。

---

## 💼 真实生产项目模板

### **fastapi/full-stack-fastapi-template** ⭐ ~32k 🌿进阶 [英文]
一句话：FastAPI 官方全栈模板（FastAPI + React + PostgreSQL + Docker）。

- GitHub: https://github.com/fastapi/full-stack-fastapi-template
- **怎么用它学习**：clone 跑起来，**研究目录结构**——这是工业级项目应有的样子。
- 推荐理由：你以后写真实后端的目录会和它越来越像。

### **zhanymkanov/fastapi-best-practices** ⭐ ~10k 🌿进阶 [英文]
一句话：FastAPI 项目最佳实践合集（一份 README）。

- GitHub: https://github.com/zhanymkanov/fastapi-best-practices
- **怎么用它学习**：每条建议读完后，对照自己代码改造一处。
- 推荐理由：FastAPI 真实生产经验浓缩。

### **asacristani/fastapi-rocket-boilerplate** 🌿进阶 [英文]
一句话：开箱即用的 FastAPI 脚手架（含认证、Celery、测试）。

- GitHub: https://github.com/asacristani/fastapi-rocket-boilerplate
- **怎么用它学习**：作为下一个个人项目的起点。
- 推荐理由：避免重复造"项目骨架"轮子。

---

## 🎓 跟着名师做（视频+代码）

### **karpathy/micrograd** ⭐ ~10k 🌿进阶 [英文]
一句话：Andrej Karpathy 用 150 行 Python 实现自动微分。

- GitHub: https://github.com/karpathy/micrograd
- **怎么用它学习**：**配 YouTube "The spelled-out intro to neural networks"** 一边看一边敲。看完你彻底懂"反向传播"。
- 推荐理由：AI 入门最高质量 100 行。

### **karpathy/nanoGPT** ⭐ ~38k 🌲源码级 [英文]
一句话：300 行 Python 实现 GPT。

- GitHub: https://github.com/karpathy/nanoGPT
- **怎么用它学习**：跟 Karpathy 的"Let's build GPT"YouTube 视频敲代码。**之后所有 LLM 论文你都看得懂**。
- 推荐理由：把 GPT 从神坛拉到 300 行。

### **karpathy/llm.c** ⭐ ~25k 🌲源码级 [英文]
一句话：用 C 重写 LLM 训练（不依赖 PyTorch）。

- GitHub: https://github.com/karpathy/llm.c
- **怎么用它学习**：先把 nanoGPT 吃透再来。这是给"想理解 GPU 内存与算力"的人。
- 推荐理由：进阶到"LLM 系统级理解"的最佳路径。

---

## 🏆 Easy-vibe 风格代表项目（短小精悍）

特点：**< 1000 行 + 立刻能跑 + 改一行有反馈 + 看完顿悟**。

- **karpathy/minGPT** ⭐ ~22k：nanoGPT 的前身，更像教学稿
- **tinygrad/tinygrad** ⭐ ~28k：把 PyTorch 压缩到 1000 行的 DL 框架
- **karpathy/minRF**：极简 rectified flow（图像生成）
- **build-your-own-x 中的 Python 子项目**：手写 git / shell / regex 引擎

**怎么用它们学习**：每周末选一个，**用 4 小时读完 + 写一篇博客**。这种"短闭环"最练肌肉。

---

## 📖 项目跟读 5 步法

不是"打开仓库随便翻翻"。是有方法的。

### Step 1：Run（先让它跑起来）
- clone → 装依赖 → 跑示例
- **跑不起来就去搜 issues**，前人 99% 踩过同样坑
- 跑通再读代码，否则纯纸上谈兵

### Step 2：Use（用一周）
- 至少用它做 3 个小 demo
- 体感"它解决了什么问题、不解决什么问题"
- 没用过的库不要直接读源码

### Step 3：Skim（扫一遍目录结构）
- 看 README + `tree -L 2` 项目结构
- 找入口文件（一般是 `__init__.py` / `main.py` / `cli.py`）
- 画一张"模块依赖图"

### Step 4：Read（精读一个模块）
- 选最核心的一个文件（比如 requests 的 sessions.py）
- 一行行读，不懂的查文档/Issues
- 在 fork 里加注释，commit 上去

### Step 5：Contribute（贡献一点点）
- 找 `good-first-issue` 标签
- 修一个文档错别字 / 补一个测试 / 修一个小 bug
- 第一个 PR 被合并的瞬间，你的 Python 段位+1

---

## ⚠️ Python 学习者陷阱

1. **囤教程**：100 个收藏夹链接 + 50G 视频 = 0 行代码。
2. **跳过文档读源码**：requests 文档你都没读完，去读源码就是装 X。
3. **追新潮库不学语言核心**：会用 fastapi 不会写装饰器，是"积木工程师"。
4. **只看不写**：Python 是手感语言，停笔超过一周，能力衰减 30%。
5. **不写 type hint**：2024 年还在裸 def 的 Python 工程师，正在被淘汰。
6. **包管理凑合**：项目交付时才发现依赖不全。**从 day 1 用 uv / poetry**。
7. **忽视测试**：写 3 行就 pytest，是工业级 Python 工程师的标配。

---

## 📌 一句话收尾

**Python 的精华不在解释器，在你能 fork 的那一万个仓库里。**
下一篇我们进入 JavaScript 项目精选，开始另一座金矿。
