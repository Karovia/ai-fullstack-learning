---
chapter: 08
week: W05
title: MCP 协议详解
date: 2026-06-19
tags: [mcp, model-context-protocol, anthropic, tool-protocol, claude-desktop]
prereq: [W02-手写Agent不用框架]
estimated_hours: 16
difficulty: ★★★★☆
---

# W05 · MCP 协议详解

> MCP（Model Context Protocol）是 Anthropic 2024.11 开源、2025 年成为跨厂商标准的"AI 工具/数据连接协议"。学会它，你写的 server 能被 Claude Desktop、Cursor、Windsurf、自家 Agent 等任意客户端复用。

---

## 一、MCP 是什么？

**用一句话：USB-C 之于硬件，MCP 之于 LLM 工具。**

之前每个 AI 客户端都要自己定义一套"怎么接工具"的方式：
- Claude Desktop 一种
- Cursor 一种
- 自家 Agent 又一种

写一个"读 Notion"的工具要重复三遍。MCP 给出统一标准：**一个 server，所有 client 通用**。

---

## 二、为什么必须有 MCP？解决 N×M 集成爆炸

```
没有 MCP：N 个客户端 × M 个工具 = N×M 个适配
有 MCP：  N + M 个适配（每边只对接 MCP）
```

到 2026 年，主流 LLM 客户端（Claude Desktop、Cursor、Windsurf、ChatGPT、Cline、Continue 等）都已经原生支持 MCP。

---

## 三、三大原语（Primitives）

MCP server 可以暴露三类能力：

| 原语 | 类比 | 谁触发 | 例子 |
|---|---|---|---|
| **Tools** | 函数（写） | LLM 主动调 | `create_issue`、`run_sql` |
| **Resources** | 文件（读） | 客户端选择 / LLM 引用 | `file://README.md`、`db://users` |
| **Prompts** | 模板 | 用户从 UI 选 | `/summarize-pr`、`/code-review` |

简单记忆：
- **Tools = 动词**（做事）
- **Resources = 名词**（数据）
- **Prompts = 快捷方式**（给用户的）

---

## 四、协议层：JSON-RPC 2.0

底层就是 **JSON-RPC 2.0**——和 LSP（语言服务器协议）同款。

请求示例：
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {"name": "search", "arguments": {"q": "MCP"}}
}
```

主要 method：
- `initialize` / `initialized`
- `tools/list` / `tools/call`
- `resources/list` / `resources/read`
- `prompts/list` / `prompts/get`
- `notifications/*`（双向通知）

---

## 五、三种通信方式

| 方式 | 场景 | 特点 |
|---|---|---|
| **stdio** | 本机进程（Claude Desktop ↔ 本地 server） | 最简单，最常用 |
| **HTTP + SSE** (旧) | 远程 server | 已被 Streamable 替代 |
| **Streamable HTTP** | 远程 server | 单 endpoint，双向流，2025 标准 |

---

## 六、安全模型

- 每个 server 启动需用户**显式授权**（在 client 配置里加）
- Tool 调用建议加**确认弹窗**（Claude Desktop 默认会问）
- Server 跑在用户机器/账号下，**不应**用 root
- Resources 只暴露必要数据，避免越权
- 远程 server 必须 HTTPS + 鉴权

---

## 七、Python SDK：写 MCP Server

安装：

```bash
pip install "mcp[cli]"
```

最小 server（FastMCP）：

```python
# my_mcp.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("demo-server")

@mcp.tool()
def add(a: int, b: int) -> int:
    """加法"""
    return a + b

@mcp.resource("note://{name}")
def read_note(name: str) -> str:
    """读笔记"""
    with open(f"notes/{name}.md") as f:
        return f.read()

@mcp.prompt()
def daily_summary(date: str) -> str:
    """生成日报模板"""
    return f"请总结 {date} 的工作要点"

if __name__ == "__main__":
    mcp.run()  # 默认 stdio
```

启动并调试：

```bash
mcp dev my_mcp.py   # 启动 + 打开 inspector
```

---

## 八、TypeScript SDK：写 MCP Server

```bash
npm i @modelcontextprotocol/sdk
```

```ts
// server.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "demo", version: "0.1.0" });

server.tool(
  "add",
  "加法",
  { a: z.number(), b: z.number() },
  async ({ a, b }) => ({ content: [{ type: "text", text: String(a + b) }] })
);

server.resource(
  "note",
  "note://{name}",
  async (uri, { name }) => ({
    contents: [{ uri: uri.href, text: `内容 of ${name}` }]
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

跑：`npx tsx server.ts`

---

## 九、Client 集成

### 1. Claude Desktop

编辑 `~/Library/Application Support/Claude/claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "demo": {
      "command": "python",
      "args": ["/abs/path/to/my_mcp.py"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem",
               "/Users/me/Documents"]
    }
  }
}
```

重启 Claude Desktop，左下角出现 MCP 图标，能看到所有 tools / resources / prompts。

### 2. Cursor / Windsurf

类似 JSON 配置，路径分别是：
- Cursor: `~/.cursor/mcp.json`（或项目里的 `.cursor/mcp.json`）
- Windsurf: `~/.codeium/windsurf/mcp_config.json`

### 3. 在自家 Agent 里集成 MCP

Python：

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def use_mcp():
    params = StdioServerParameters(
        command="python", args=["my_mcp.py"]
    )
    async with stdio_client(params) as (read, write):
        async with ClientSession(read, write) as sess:
            await sess.initialize()
            tools = await sess.list_tools()
            print(tools)
            r = await sess.call_tool("add", {"a": 1, "b": 2})
            print(r)
```

把 `tools` 适配成 Anthropic API 的 tool 格式，就把 MCP 接进了你 W02 的 mini-agent。

---

## 十、生态：现成的 MCP Server

awesome-mcp-servers 列表里已经有数百个，常用：

| Server | 用途 |
|---|---|
| `@modelcontextprotocol/server-filesystem` | 本地文件系统 |
| `@modelcontextprotocol/server-github` | GitHub API |
| `@modelcontextprotocol/server-postgres` | Postgres 查询 |
| `@modelcontextprotocol/server-slack` | Slack 消息 |
| `@modelcontextprotocol/server-brave-search` | Brave 搜索 |
| `@modelcontextprotocol/server-puppeteer` | 浏览器自动化 |
| `mcp-obsidian` | Obsidian Vault |
| `@notionhq/notion-mcp-server` | Notion |

---

## 十一、写 MCP Server 最佳实践

1. **命名清晰**：`get_user` / `create_issue`，别用缩写。
2. **描述要好**：LLM 完全靠 description 决定是否调用。
3. **输入 schema 严格**：用 Pydantic / zod，别用 dict。
4. **错误用 isError**：返回 `{isError: True, content: [...]}` 让 LLM 知道。
5. **别返回上 G 的数据**：分页 / 截断。
6. **危险操作分两步**：先 `prepare`、再 `confirm`。
7. **资源用 URI**：`gh://issues/123` 比 string id 强。
8. **文档放仓库 README**：客户端的人要看怎么配。

---

## 十二、调试：MCP Inspector

```bash
mcp dev my_mcp.py
# 或者纯 inspector
npx @modelcontextprotocol/inspector python my_mcp.py
```

浏览器里能：
- 看到所有 tools / resources / prompts
- 手动调一次看输入输出
- 查看 JSON-RPC 报文

**写 MCP 必备调试器**。

---

## 十三、本周实战：Obsidian Vault MCP Server

目标：让 Claude Desktop 能读写你的笔记库。

### 设计

| 名称 | 类型 | 输入/URI | 描述 |
|---|---|---|---|
| `list_notes` | tool | folder?: str | 列所有 .md 路径 |
| `read_note` | tool | path | 读笔记内容 |
| `create_note` | tool | path, content | 新建笔记 |
| `append_note` | tool | path, content | 追加内容 |
| `search` | tool | query | 全文搜索（简易 grep） |
| `vault_config` | resource | `vault://config` | Vault 配置 |
| `daily_summary` | prompt | date | 日报模板 |

### 实现（核心片段）

```python
# obsidian_mcp.py
from pathlib import Path
from mcp.server.fastmcp import FastMCP

VAULT = Path("/Users/me/Vault")
mcp = FastMCP("obsidian-vault")

@mcp.tool()
def list_notes(folder: str = "") -> list[str]:
    """列出所有笔记路径"""
    base = VAULT / folder
    return [str(p.relative_to(VAULT)) for p in base.rglob("*.md")]

@mcp.tool()
def read_note(path: str) -> str:
    """读笔记"""
    return (VAULT / path).read_text()

@mcp.tool()
def create_note(path: str, content: str) -> str:
    """新建笔记（已存在会拒绝）"""
    p = VAULT / path
    if p.exists():
        return f"ERROR: {path} already exists"
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(content)
    return f"OK: created {path}"

@mcp.tool()
def append_note(path: str, content: str) -> str:
    """追加到笔记尾部"""
    p = VAULT / path
    if not p.exists():
        return f"ERROR: {path} not found"
    with p.open("a") as f:
        f.write("\n" + content)
    return f"OK: appended to {path}"

@mcp.tool()
def search(query: str) -> list[dict]:
    """简易全文搜索"""
    hits = []
    for p in VAULT.rglob("*.md"):
        text = p.read_text(errors="ignore")
        if query.lower() in text.lower():
            idx = text.lower().find(query.lower())
            hits.append({
                "path": str(p.relative_to(VAULT)),
                "snippet": text[max(0, idx - 40): idx + 80]
            })
    return hits[:20]

@mcp.resource("vault://config")
def vault_config() -> str:
    return f"vault path: {VAULT}\nnotes count: {len(list(VAULT.rglob('*.md')))}"

@mcp.prompt()
def daily_summary(date: str) -> str:
    return f"请总结 {date} 这一天我新建/修改的笔记，并提炼 3 条 takeaway。"

if __name__ == "__main__":
    mcp.run()
```

### 接 Claude Desktop

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "python",
      "args": ["/abs/path/to/obsidian_mcp.py"]
    }
  }
}
```

重启 Claude Desktop，问："帮我列一下笔记库里 W 开头的文件"，Claude 会自动调 `list_notes` 然后筛选。

---

## 十四、checklist

- [ ] 能讲清 Tools / Resources / Prompts 区别
- [ ] FastMCP 写过最小 server
- [ ] mcp inspector 用过
- [ ] Claude Desktop 接过自定义 server
- [ ] Obsidian Vault MCP server 实战完成
- [ ] 自己 mini-agent 也能接 MCP

---

## 十五、参考

- spec: modelcontextprotocol.io
- python-sdk: github.com/modelcontextprotocol/python-sdk
- typescript-sdk: github.com/modelcontextprotocol/typescript-sdk
- awesome-mcp-servers
- Anthropic 博客《Introducing the Model Context Protocol》(2024.11)

下一周：**Claude Agent SDK + OpenAI Agents SDK**——官方抽象。
