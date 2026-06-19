---
chapter: 07-AI基础-Prompt与LLM
week: W04
title: RAG 完整实现
duration: 7-10 天
prereq: W01-W03
goal: 从零搭一套生产可用的 RAG：上传 PDF → 提问 → 流式答案 + 引用跳转
tags: [rag, embedding, vector-db, hybrid-search, rerank, citations]
updated: 2026-06-19
---

# W04 · RAG 完整实现

> RAG = **R**etrieval-**A**ugmented **G**eneration（检索增强生成）。一句话：**先查再答**。

---

## 0. 为什么要 RAG

LLM 三大痛：
1. 知识截止，不知公司私域
2. 容易幻觉，胡编人名/数字
3. 上下文虽长，但**全塞进去贵且糊涂**

RAG 让模型在回答前**先去你的资料库查相关片段**，再"开卷答题"。事实正确率立刻起飞。

---

## 1. RAG vs Fine-tune

| 维度 | RAG | Fine-tune |
|---|---|---|
| 改了什么 | 上下文 | 模型权重 |
| 上线速度 | 分钟 | 几小时-几天 |
| 知识更新 | 改文档即可 | 重训 |
| 适合 | 事实/私域知识 | 风格、特定指令格式 |
| 成本 | 低 | 中-高 |
| 可解释 | 高（有引用） | 低 |

经验：**先做 RAG，撞墙再 fine-tune，几乎所有问题都不用 fine-tune**。

---

## 2. 类比：开卷考试

LLM = 学霸（脑子好但不能现查）
RAG = 给学霸一摞书 + 一个图书管理员
管理员根据问题翻到最相关的几页递给学霸，学霸照着答。

---

## 3. RAG 完整流水线（10 步）

```
PDF/Word/网页
   │ 1. 接入
   ▼
Raw Text + 元数据
   │ 2. 解析
   ▼
长文本块
   │ 3. 切片
   ▼
Chunks（含元数据）
   │ 4. 元数据
   ▼
Embedding 向量
   │ 5. 向量化
   ▼
向量库
   │ 6. 存储
   ▼
─────── 查询时 ───────
用户问题 → 7. 向量检索 → 8. 混合 BM25 → 9. 重排 → 10. 拼 prompt 生成
```

---

## 4. 第 1 步：数据接入

来源清单：
- **PDF**：合同、论文、说明书
- **Word/PPT**：内部文档
- **Markdown**：技术 wiki
- **HTML**：网页
- **Confluence / Notion**：用官方 API 拉
- **Google Drive / SharePoint**：connector

经验：先**统一转 Markdown**，所有下游环节只处理 markdown。

---

## 5. 第 2 步：解析

| 工具 | 适合 | 优点 | 坑 |
|---|---|---|---|
| **Docling** (IBM 2024) | 通用 | 表格/公式好 | 慢 |
| **Unstructured.io** | 通用 | 输出结构化元素 | 闭源高级版 |
| **PyMuPDF (fitz)** | PDF 文字 | 极快 | 不识表 |
| **pdfplumber** | 表格 PDF | 表抽得稳 | 慢 |
| **MarkItDown** (微软) | 多格式→md | 简单 | 表格糙 |

```python
# 简单版：PyMuPDF
import fitz
def parse_pdf(path):
    doc = fitz.open(path)
    pages = []
    for i, page in enumerate(doc):
        pages.append({"page": i+1, "text": page.get_text()})
    return pages
```

---

## 6. 第 3 步：切片（chunking）

四种策略：

### 6.1 固定窗口

```python
def fixed_chunks(text, size=500, overlap=50):
    out = []
    for i in range(0, len(text), size - overlap):
        out.append(text[i:i+size])
    return out
```

简单但会切断句子。

### 6.2 递归字符（最常用）

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500, chunk_overlap=50,
    separators=["\n\n", "\n", "。", "！", "？", " ", ""],
)
chunks = splitter.split_text(text)
```

按段→句→字逐级切，**首选**。

### 6.3 语义切片

用 embedding 比较相邻句子相似度，相似度骤降处切。

```python
# 伪代码
sims = [cos(emb(s[i]), emb(s[i+1])) for i in range(len(s)-1)]
breakpoints = [i for i, x in enumerate(sims) if x < threshold]
```

慢但块更"内聚"。

### 6.4 标题切（Markdown）

```python
from langchain_text_splitters import MarkdownHeaderTextSplitter
splitter = MarkdownHeaderTextSplitter(headers_to_split_on=[
    ("#", "h1"), ("##", "h2"), ("###", "h3"),
])
```

文档天然有结构时最准。

**经验值**：chunk 500-800 字符，overlap 50-100。

---

## 7. 第 4 步：元数据

每个 chunk 必须带：

```python
{
    "id": "doc_123_p5_c2",
    "text": "……",
    "source": "员工手册.pdf",
    "page": 5,
    "section": "请假流程",
    "created_at": "2026-03-01",
    "doc_type": "policy",
}
```

后续过滤、引用、时效性都靠它。

---

## 8. 第 5 步：向量化

主流 embedding 模型（2026）：

| 模型 | 维度 | 适合 |
|---|---|---|
| **voyage-3** (Anthropic 推荐) | 1024 | 通用强 |
| **text-embedding-3-large** (OpenAI) | 3072 | 通用 |
| **bge-m3** (BAAI) | 1024 | 多语言 + 开源 |
| **Qwen3-Embedding** | 1024 | 中文最强 |

```python
import voyageai
vo = voyageai.Client()
vecs = vo.embed(texts, model="voyage-3", input_type="document").embeddings
# 查询时 input_type="query"
```

注意：document 与 query 用**不同 input_type**，准确率明显提升。

---

## 9. 第 6 步：向量库

| 库 | 适合 |
|---|---|
| **pgvector** | 已用 Postgres，最省心 |
| **Qdrant** | 独立部署，过滤强 |
| **LanceDB** | 嵌入式，零运维 |
| **Milvus** | 超大规模 |

pgvector 示例：

```sql
CREATE EXTENSION vector;
CREATE TABLE chunks (
    id text primary key,
    text text,
    embedding vector(1024),
    source text, page int, meta jsonb
);
CREATE INDEX ON chunks USING hnsw (embedding vector_cosine_ops);
```

```python
import psycopg2
cur.executemany("INSERT INTO chunks VALUES (%s,%s,%s,%s,%s,%s)",
    [(c["id"], c["text"], c["vec"], c["source"], c["page"], json.dumps(c["meta"]))
     for c in chunks])
```

---

## 10. 第 7 步：向量检索

```sql
SELECT id, text, source, page,
       1 - (embedding <=> %s::vector) AS score
FROM chunks
WHERE doc_type = 'policy'
ORDER BY embedding <=> %s::vector
LIMIT 20;
```

`<=>` 是 cosine 距离。先取 top 20，留给后面重排。

---

## 11. 第 8 步：混合检索 (BM25 + 向量 + RRF)

向量擅长**语义**，BM25 擅长**关键词**（产品 ID、错误码）。混合最稳。

```python
def rrf(lists, k=60):
    """Reciprocal Rank Fusion"""
    scores = {}
    for lst in lists:
        for rank, doc_id in enumerate(lst):
            scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank)
    return sorted(scores, key=scores.get, reverse=True)

vec_top = vector_search(q)
bm25_top = bm25_search(q)
fused = rrf([vec_top, bm25_top])[:20]
```

Postgres 也支持 `ts_rank` 做 BM25 的近似。

---

## 12. 第 9 步：重排 (Rerank)

混合检索后剩 20 条，用一个**交叉编码器**对每条 (query, chunk) 打分：

```python
import cohere
co = cohere.Client()
out = co.rerank(model="rerank-3.5", query=q,
                documents=[c["text"] for c in fused], top_n=5)
top = [fused[r.index] for r in out.results]
```

或开源 `bge-reranker-v2-m3`。**重排是 RAG 提升幅度最大的一步**，能 +10-20% 命中率。

---

## 13. 第 10 步：生成 + 引用

```python
SYSTEM = """你是文档助手。只能根据 <context> 内的资料回答。
每个事实后面用 [n] 标注引用编号；找不到就回答"资料中未提及"。"""

ctx = "\n\n".join(f"[{i+1}] (来源:{c['source']} p.{c['page']})\n{c['text']}"
                  for i, c in enumerate(top))

resp = client.messages.create(
    model="claude-sonnet-4-5", max_tokens=1024,
    system=SYSTEM,
    messages=[{"role":"user","content":
        f"<context>\n{ctx}\n</context>\n\n问题：{q}"}],
)
```

引用 `[n]` 与上面的列表一一对应，前端可点击跳转到 PDF 第 n 页。

---

## 14. 进阶技巧

### 14.1 HyDE（Hypothetical Document Embeddings）

让模型先**假装回答**，用假答案去检索：

```python
fake_answer = llm("假设你知道答案，回答这个问题：" + q)
top = vector_search(emb(fake_answer))
```

短问题 → 长假答案，向量更稳。

### 14.2 父子文档

切两套：小切片（精准查）+ 大切片（完整给）。查命中小，回填用大。

### 14.3 Contextual Retrieval（Anthropic 2024）

每个 chunk 入库前，让 LLM 加一段**它在原文中的上下文摘要**再 embed：

```
chunk_with_context = "本节属于《员工手册》第 3 章请假流程，描述年假天数计算规则。\n\n" + chunk_text
```

Anthropic 实测：召回错误率降 49%。配合 prompt cache 成本可控。

### 14.4 GraphRAG (Microsoft)

把文档抽成知识图谱，问"X 与 Y 的关系"这类多跳问题特别准。重型方案。

### 14.5 多查询扩展

让 LLM 把用户问题改写成 3-5 个不同表述并行检索：

```
请把以下问题改写成 4 个不同表述，输出 JSON 数组：
{q}
```

---

## 15. 多轮对话 + RAG

每轮先做**问题改写**：把当前问题 + 前几轮历史融合成自包含问题，再去检索。

```python
def rewrite(history, q):
    sys = "把当前问题改写为不依赖上下文的独立问题。"
    return llm(sys, history + [{"role":"user","content":q}])
```

---

## 16. 缓存

- **embedding 缓存**：text → vec 用 sqlite/redis 缓存，重复文档不重算
- **检索缓存**：query → top-k 缓存 5 分钟，热门问题秒答
- **LLM 缓存**：完全相同的 prompt 直接返回上次结果

---

## 17. 评测（详见 W05）

- **召回率 Recall@k**：金标 chunk 是否在 top-k 里
- **MRR**：金标排第几
- **答案相关度**：用 LLM-as-Judge 打分
- **忠实度 (Faithfulness)**：答案里的事实是否都能在 context 找到

工具：**Ragas** 是 RAG 专用评测库。

---

## 18. 实战：完整 RAG 系统

### 18.1 后端 FastAPI

```python
# app.py
from fastapi import FastAPI, UploadFile
from fastapi.responses import StreamingResponse
import voyageai, anthropic
from pipeline import parse, chunk, embed, store, retrieve, rerank

app = FastAPI()
ant = anthropic.Anthropic()

@app.post("/upload")
async def upload(file: UploadFile):
    text = parse(await file.read(), file.filename)
    chunks = chunk(text, file.filename)
    embed(chunks)
    store(chunks)
    return {"ok": True, "chunks": len(chunks)}

@app.post("/ask")
async def ask(q: str):
    top = rerank(q, retrieve(q, k=20), k=5)
    ctx = "\n\n".join(f"[{i+1}] (来源:{c['source']} p.{c['page']})\n{c['text']}"
                      for i, c in enumerate(top))
    def gen():
        with ant.messages.stream(
            model="claude-sonnet-4-5", max_tokens=1024,
            system="只根据 context 回答，事实标 [n]，找不到说不知道。",
            messages=[{"role":"user","content":
                f"<context>\n{ctx}\n</context>\n问题：{q}"}],
        ) as s:
            for t in s.text_stream:
                yield t
        yield "\n\n__CITATIONS__:" + json.dumps(top)
    return StreamingResponse(gen(), media_type="text/plain")
```

### 18.2 前端 Next.js（关键片段）

```tsx
// app/page.tsx
"use client";
import { useState } from "react";
export default function Home() {
  const [q, setQ] = useState("");
  const [ans, setAns] = useState("");
  const [cites, setCites] = useState<any[]>([]);

  async function ask() {
    setAns(""); setCites([]);
    const r = await fetch("/api/ask", {method:"POST", body: JSON.stringify({q})});
    const reader = r.body!.getReader(); const dec = new TextDecoder();
    let buf = "";
    while (true) {
      const { done, value } = await reader.read(); if (done) break;
      buf += dec.decode(value);
      if (buf.includes("__CITATIONS__:")) {
        const [text, json] = buf.split("__CITATIONS__:");
        setAns(text); setCites(JSON.parse(json));
      } else setAns(buf);
    }
  }
  return (
    <main className="p-8 max-w-3xl mx-auto">
      <input className="border p-2 w-full" value={q} onChange={e=>setQ(e.target.value)}/>
      <button onClick={ask} className="bg-black text-white px-4 py-2 mt-2">问</button>
      <article className="prose mt-6 whitespace-pre-wrap">{ans}</article>
      <ol className="mt-6 text-sm">
        {cites.map((c,i)=><li key={i}>[{i+1}] {c.source} p.{c.page}</li>)}
      </ol>
    </main>
  );
}
```

把 `[n]` 渲染为可点击 anchor，跳到底部引用并高亮 PDF 页（用 react-pdf）即可。

---

## 19. 练习

1. 上传 3 份不同 PDF（合同、论文、说明书），各问 5 个问题，记录命中率
2. 关掉重排，看准确率掉多少
3. 用 Contextual Retrieval 改造 chunk，对比效果
4. 给 RAG 加多轮对话，验证问题改写的必要性
5. 用 Ragas 跑出忠实度 / 答案相关度指标

---

## 20. Checklist

- [ ] RAG 10 步流水线能口述
- [ ] 至少 3 种切片策略实现并比较过
- [ ] 元数据 schema 设计完整
- [ ] embedding 模型选过、文档/查询 input_type 区分
- [ ] pgvector 或 Qdrant 跑通
- [ ] 实现混合检索（BM25 + 向量 + RRF）
- [ ] 重排接通
- [ ] 答案带可跳转引用
- [ ] HyDE / Contextual Retrieval 至少试一种
- [ ] FastAPI + Next.js 端到端 demo 跑通
- [ ] Ragas 出过一次评测报告

下一周：解决"我怎么知道改好了"的问题 — 评测与 LLM-as-Judge。
