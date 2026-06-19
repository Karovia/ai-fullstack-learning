---
title: 第 01 章 - Python 从零到源码
chapter: 01
duration: 10 周
prerequisite: [[00-学习方法论/00-如何高效学习编程]]
tags: [Python, 后端, 源码级]
---

# 第 01 章 · Python 从零到源码（10 周）

## 🎯 章节目标

学完本章你将：
- 用 Python 写出**任何**算法题、命令行工具、爬虫
- 理解装饰器、生成器、协程、元类、描述符等高级特性
- 能阅读 CPython 源码（Objects/、Modules/）
- 写出符合 PEP 8 的工程化代码（类型注解、单元测试、打包发布）

## 📅 周计划

| 周 | 主题 | 难度 | 关键产出 |
|---|------|------|----------|
| W1 | 语法基础：变量/类型/运算/控制流 | 入门 | 100 道基础题 |
| W2 | 数据结构：list/tuple/dict/set + 字符串 | 入门 | LeetCode 简单 20 题 |
| W3 | 函数 + 模块 + 文件 IO | 入门 | 命令行工具 v1 |
| W4 | OOP：类/继承/魔术方法/多态 | 进阶 | 自定义类库 |
| W5 | 异常 + 装饰器 + 上下文管理器 | 进阶 | 装饰器 5 件套 |
| W6 | 迭代器/生成器/yield from | 进阶 | 实现 itertools |
| W7 | 并发：threading/multiprocessing/asyncio | 进阶 | 异步爬虫 |
| W8 | 类型系统：typing + Pydantic + Protocol | 进阶 | 类型化重构 |
| W9 | 工程化：pytest/uv/打包/发布 PyPI | 进阶 | 发一个真实包 |
| W10 | CPython 源码导读 | 源码级 | 源码笔记 |

## 📖 详细内容大纲

### W1 - 语法基础
- 变量、内置类型（int/float/str/bool/None）
- 算术、比较、逻辑、位运算
- if / for / while / break / continue / else
- 输入输出 `input()` / `print()` / f-string
- **必看**：[廖雪峰 Python 教程 - 基础](https://liaoxuefeng.com/books/python/basic/index.html)

### W2 - 数据结构
- list（切片、列表推导、sort/sorted）
- tuple、namedtuple
- dict（字典推导、defaultdict、Counter、OrderedDict）
- set / frozenset
- str 详解（编码、正则）
- **必看**：[Python 官方教程 - 数据结构](https://docs.python.org/zh-cn/3/tutorial/datastructures.html)

### W3 - 函数与模块
- 函数定义、参数（位置/关键字/默认/*args/**kwargs）
- 作用域 LEGB
- lambda + map/filter/reduce
- 模块、包、`__init__.py`、相对导入
- 文件 IO、pathlib、json/csv

### W4 - OOP
- class、self、`__init__`
- 继承、MRO（C3 算法）、`super()`
- 魔术方法：`__repr__` / `__eq__` / `__hash__` / `__getitem__` / `__call__`
- @property / @classmethod / @staticmethod
- dataclass、enum
- **必读**：《Fluent Python》第 1 章 - "Python 数据模型"

### W5 - 装饰器与上下文管理器
- 闭包 + 高阶函数
- 装饰器（无参/有参/类装饰器）
- functools.wraps / lru_cache / partial
- with 语句、`__enter__/__exit__`、contextlib
- 异常体系、自定义异常、try/except/else/finally

### W6 - 迭代器与生成器
- 可迭代 vs 迭代器（`__iter__` / `__next__`）
- 生成器（yield）
- 协程基础（send/throw/close）
- yield from
- 实现 itertools 的 chain/islice/groupby

### W7 - 并发
- threading + GIL 原理
- multiprocessing + Pool
- concurrent.futures
- asyncio：事件循环、协程、Task、async/await
- aiohttp / httpx 异步 HTTP

### W8 - 类型系统
- typing：List/Dict/Optional/Union/Literal/Generic
- TypeVar、Protocol（结构化类型）
- Pydantic v2（数据验证）
- mypy 静态检查

### W9 - 工程化
- 项目结构（src layout）
- pytest + 覆盖率
- uv（替代 pip+venv+poetry）
- ruff（替代 flake8+black+isort）
- 打包、发布到 PyPI、版本号
- pre-commit + GitHub Actions CI

### W10 - CPython 源码导读
- CPython 整体架构（Parser → AST → Compiler → Bytecode → Eval Loop）
- `Objects/listobject.c` - list 实现
- `Objects/dictobject.c` - dict 实现（hash 冲突解决）
- `Python/ceval.c` - 字节码执行循环
- 必读：《CPython Internals》（Anthony Shaw）

## 📝 考核任务（10 项，每周至少 1 项）

- [ ] **T1（W1）**：完成 [Python 基础 100 题](https://github.com/jackfrued/Python-100-Days) 第 1-30 题
- [ ] **T2（W2）**：LeetCode 简单 20 题用 Python 通过
- [ ] **T3（W3）**：实现一个 `cli-todo` 命令行待办，支持 add/list/done/remove
- [ ] **T4（W4）**：实现一个简易 `Vector` 类（支持 +、-、*、==、`__repr__`），通过《Fluent Python》第一章的所有测试
- [ ] **T5（W5）**：手写 5 个装饰器：`@timer`、`@retry`、`@cache`、`@deprecated`、`@validate_args`
- [ ] **T6（W6）**：手写一个生成器版的 `range / enumerate / zip / chain`
- [ ] **T7（W7）**：写一个异步爬虫抓取豆瓣 Top 250，并发 10，3 秒内完成
- [ ] **T8（W8）**：用 Pydantic + typing 重写 T3 的 cli-todo，引入数据校验
- [ ] **T9（W9）**：将 T8 打包发布到 PyPI（可用 testpypi），别人能 `pip install your-todo` 安装
- [ ] **T10（W10）**：阅读 `Objects/listobject.c`，画一张 PyListObject 内存布局图，写一篇 2000 字博客解析

## 🚀 章节大项目：`pyspider-cli` —— 异步并发爬虫工具

### 需求
做一个**生产级**的命令行爬虫工具，能：
1. 输入 URL 列表（文件或 stdin）
2. 异步并发抓取（可配置并发数）
3. 自动重试 + 限速
4. 解析 HTML（BeautifulSoup 或 lxml）
5. 输出 JSON / CSV
6. 进度条（rich）+ 日志（loguru）
7. 配置用 YAML，校验用 Pydantic
8. 全量类型注解 + 90% 测试覆盖率
9. 打包发布到 PyPI

### 技术栈
- Python 3.12+
- httpx（异步 HTTP）
- asyncio
- BeautifulSoup4
- typer（CLI 框架，比 argparse 现代）
- rich（进度条 / 表格输出）
- loguru（日志）
- pydantic v2（配置 + 数据模型）
- pytest + pytest-asyncio + pytest-cov
- ruff + mypy
- uv（包管理）

### 验收标准
- [ ] `pip install pyspider-cli` 后能直接 `pyspider crawl urls.txt -c 10 -o out.json`
- [ ] 100 个 URL 在 5 秒内完成（依赖网速）
- [ ] 单元测试覆盖率 ≥ 85%
- [ ] mypy --strict 0 错误
- [ ] README 完整（中英双语 + 截图 GIF）
- [ ] GitHub Actions 自动跑测试 + 发版
- [ ] 至少 1 个真实抓取案例（如：豆瓣 / Hacker News / GitHub Trending）

### 进阶挑战
- 加入 Playwright 支持渲染 JS 网站
- 实现一个简易的"调度器"（去重 + 队列）
- 加入 SQLite/Redis 持久化

## 🐍 源码级延伸（W10 之后可做）

- 用 Python 实现一个 [yet-another-Python-interpreter](https://github.com/python/cpython)（解释器子集）
- 阅读 [PEP 8](https://peps.python.org/pep-0008/) / [PEP 20](https://peps.python.org/pep-0020/) / [PEP 484](https://peps.python.org/pep-0484/) / [PEP 492](https://peps.python.org/pep-0492/)
- 给 CPython 提一个文档级 PR

## 📚 推荐资源

### 入门
- 📖 [廖雪峰 Python 教程](https://liaoxuefeng.com/books/python/) - 中文最佳入门
- 📖 [Python 官方中文教程](https://docs.python.org/zh-cn/3/tutorial/)
- 📘 《Python 编程：从入门到实践》(Eric Matthes)

### 进阶
- 📘 《Fluent Python (2nd)》Luciano Ramalho - **必读**
- 📘 《Effective Python (3rd)》Brett Slatkin
- 📖 [Real Python](https://realpython.com/) - 高质量专题文章
- 📖 [Python Cookbook 中文版](https://github.com/yidao620c/python3-cookbook)

### 源码级
- 📘 《CPython Internals》Anthony Shaw - 唯一系统讲解释器实现
- 🔗 [CPython 源码](https://github.com/python/cpython)
- 📖 [PEP 索引](https://peps.python.org/)
- 📖 [Python Language Reference](https://docs.python.org/zh-cn/3/reference/)

### 项目实战
- 🛠️ [Python-100-Days](https://github.com/jackfrued/Python-100-Days) ~155k star
- 🛠️ [build-your-own-x](https://github.com/codecrafters-io/build-your-own-x)

---

✅ 完成全部 10 项考核 + 章节项目 → 进入 [[02-JavaScript-从零到源码/02-JavaScript总纲]]
