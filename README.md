# AI 全栈学习路线 🚀

> 一份从**零基础**到**独立交付 AI 全栈产品**的系统化学习路线，**Obsidian 友好**，配套笔记 / 项目 / 进度追踪。
>
> 目标：**Python · JavaScript · React · FastAPI · LLM · Agent** —— 都做到能写、能讲、能读源码。

![status](https://img.shields.io/badge/status-in--progress-yellow)
![duration](https://img.shields.io/badge/周期-9~18%20个月-blue)
![level](https://img.shields.io/badge/起点-零基础-green)
![target](https://img.shields.io/badge/终点-AI%20全栈%20%2B%20Agent-purple)

---

## 📌 这是什么

这是我自己用来学习「AI 全栈 + Agent」的 **Obsidian 知识库**，按周组织、按章节推进，覆盖 **80+ 篇详细笔记**。

整份路线遵循 4 个原则：

1. **不速成** —— 源码级精通必须经过「理解 → 模仿 → 重写 → 教会别人」四个阶段。
2. **项目驱动** —— 每章配一个完整验收项目，没做完不算学完。
3. **费曼学习法** —— 每节学完写一篇笔记教会别人。
4. **造轮子** —— 到源码阶段必须亲手写 mini-react / mini-langchain。

---

## 🗺️ 路线全景

```
00 学习方法论
   ↓
01 Python 从零到源码 ─┐
                      ├─→ 05 后端 FastAPI ─┐
02 JavaScript 从零到源码                    │
   ↓                                        ├─→ 06 数据库 + 工程化
03 HTML / CSS 现代 Web                      │      ↓
   ↓                                        │   07 AI 基础 (Prompt + LLM)
04 React 从使用到源码 ──────────────────────┘      ↓
                                              08 AI Agent 框架与源码
                                                   ↓
                                              09 毕业项目（能上线、能赚钱）
```

> 完整的可交互思维导图见 [`全栈学习路线/🗺️学习路线思维导图.canvas`](全栈学习路线/🗺️学习路线思维导图.canvas)（用 Obsidian 打开效果最好）。

---

## 📚 章节目录

| #  | 模块                          | 周期      | 章节验收项目                              |
|----|-------------------------------|-----------|-------------------------------------------|
| 00 | 学习方法论                    | 1 周      | 写一份你自己的「学习契约」                |
| 01 | Python 从零到源码             | 10 周     | 命令行工具 + 爬虫 + 阅读 CPython 源码     |
| 02 | JavaScript 从零到源码         | 10 周     | 手写 Promise / 实现 mini-lodash           |
| 03 | HTML / CSS 现代 Web           | 4 周      | 像素级还原 3 个真实网站                   |
| 04 | React 从使用到源码            | 10 周     | 在线协作白板 + 手写 mini-react            |
| 05 | FastAPI + Node 后端           | 8 周      | 完整 RESTful API + JWT + 测试             |
| 06 | 数据库 + 工程化               | 6 周      | PostgreSQL + Redis + Docker 部署          |
| 07 | AI 基础：Prompt + LLM         | 6 周      | 从零实现 GPT（rasbt 课）                  |
| 08 | AI Agent 框架与源码           | 12 周     | 多 Agent 协作系统 + MCP server            |
| 09 | 综合实战 · 毕业项目           | 8-12 周   | 一个能上线、能赚钱、能写进简历的产品      |

详见 [`全栈学习路线/学习路线全景图.md`](全栈学习路线/学习路线全景图.md) 与 [`全栈学习路线/INDEX.md`](全栈学习路线/INDEX.md)。

---

## 📂 仓库结构

```
AI全栈学习路线/
├── 全栈学习路线/
│   ├── README.md                ← 主路线总览
│   ├── INDEX.md                 ← 80+ 文件全索引
│   ├── 学习路线全景图.md
│   ├── 🗺️学习路线思维导图.canvas
│   ├── 00-学习方法论/
│   ├── 01-Python-从零到源码/
│   ├── 02-JavaScript-从零到源码/
│   ├── 03-HTML-CSS-现代Web/
│   ├── 04-前端框架-React/
│   ├── 05-后端开发-FastAPI-Node/
│   ├── 06-数据库与工程化/
│   ├── 07-AI基础-Prompt与LLM/
│   ├── 08-AI-Agent-框架与源码/
│   ├── 09-综合实战-毕业项目/
│   ├── _我的进度追踪/            ← 周报 / 打卡 / 学习契约
│   ├── _资源库/                  ← 书单 / 课程 / awesome 索引
│   └── _附录-面试与简历/
├── .obsidian/                    ← Obsidian 配置（保留，开箱即用）
└── README.md                     ← 你正在看的这个
```

---

## 🚀 快速开始

### 方式一：当成学习路线读（推荐）

```bash
git clone https://github.com/Karovia/ai-fullstack-learning.git
cd ai-fullstack-learning
```

然后用 [Obsidian](https://obsidian.md/) **打开整个文件夹作为 vault**，所有 `[[wiki-link]]` 内链、思维导图 Canvas、Dataview 都能正常工作。

### 方式二：直接当文档浏览

GitHub 上直接点 [`全栈学习路线/INDEX.md`](全栈学习路线/INDEX.md) 即可逐篇浏览。

### 方式三：Fork 它，写自己的版本

最推荐的方式。**学习路线是高度个人化的**，把它 fork 下来，按自己节奏改，往里面塞自己的笔记、项目、踩坑记录。

---

## 🛠️ 工具栈建议

| 类型      | 工具                          | 备注                              |
|-----------|-------------------------------|-----------------------------------|
| 编辑器    | VS Code / Cursor              | 必装：Python、ESLint、GitLens     |
| 终端      | iTerm2 + Oh My Zsh            | Windows 用 Windows Terminal + WSL2 |
| Python    | uv 或 pyenv + poetry          | uv 是 2025 后新标准               |
| Node.js   | nvm + pnpm                    | pnpm 比 npm 快很多                |
| 容器      | Docker Desktop                | 第 06 章会用到                    |
| AI 助手   | Claude / Cursor               | 学习而非依赖                      |
| 笔记本体  | **Obsidian**                  | 就是这个仓库本身                  |

---

## ✅ 进度追踪

- 周日晚写一份周报 → [`_我的进度追踪/周报模板`](全栈学习路线/_我的进度追踪)
- 每完成一个章节项目 → 更新 [`_我的进度追踪/项目里程碑`](全栈学习路线/_我的进度追踪)
- 365 天打卡 → 同上目录

---

## 🤝 贡献 & 反馈

这是一份**个人学习仓库**，但欢迎：

- 🐛 发现错别字 / 死链 / 错误内容 → 提 Issue 或 PR
- 💡 有更好的资源推荐 → 提 Issue
- 🌟 自己学下来有补充 → 欢迎 fork 之后的改良版本互相借鉴

---

## 📜 License

笔记内容遵循 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)（**署名 — 非商业 — 相同方式共享**）。
代码片段（如有）默认 MIT。

---

## 🔥 Now what

→ 打开 [`全栈学习路线/学习路线全景图.md`](全栈学习路线/学习路线全景图.md)，从第 0 周开始。

> **「不会的代码不要复制粘贴。先理解，再敲一遍，再合上教材自己默写一遍。」**
>
> —— 这份路线的唯一原则
