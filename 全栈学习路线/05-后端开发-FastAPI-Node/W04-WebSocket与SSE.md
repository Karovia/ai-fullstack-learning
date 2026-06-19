---
week: 4
chapter: 05-后端开发
title: WebSocket 与 SSE
duration: 1 周
prerequisites: [W02, W03]
goals:
  - 理解长连接选型与实现
  - 用 FastAPI 写 WebSocket 聊天 + Redis 跨进程
  - 写 SSE 流式 AI 回复接口
created: 2026-06-19
---

# W04 · WebSocket 与 SSE：让服务端"主动说话"

> 类比：HTTP 请求像"敲门递条子"——你问一句我答一句；WebSocket 像"装好对讲机"——挂着随时双向喊话；SSE 像"广播电台"——只有我说你听，但持续地说。

## 1. 为什么要长连接

短轮询：每秒发一个 HTTP，浪费 + 实时性差。
长轮询：服务端 hang 住直到有事。改善但仍每事一连。
**WebSocket**：一次握手永久全双工。
**SSE**：一次握手永久服务端→客户端单向流。

选型口诀：
- 双向、低延迟、二进制 → **WebSocket**（聊天、协作、游戏）
- 服务端单向推、走 HTTP/HTTPS、自动重连 → **SSE**（行情、AI 流式回复）
- 偶尔一次推送 → 短轮询足够

## 2. WebSocket 协议

握手是一次特殊的 HTTP 请求：

```
GET /ws HTTP/1.1
Host: api.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

服务端回 101：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

之后 TCP 连接不再说 HTTP，改说 WebSocket 帧（opcode + mask + payload）。

## 3. FastAPI WebSocket 基础

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws/echo")
async def echo(ws: WebSocket):
    await ws.accept()
    try:
        while True:
            text = await ws.receive_text()
            await ws.send_text(f"echo: {text}")
    except WebSocketDisconnect:
        print("bye")
```

浏览器测：

```js
const ws = new WebSocket("ws://localhost:8000/ws/echo");
ws.onmessage = e => console.log(e.data);
ws.onopen = () => ws.send("hi");
```

也能 `receive_json()` / `send_json()`。

## 4. ConnectionManager：管理多客户端

```python
from collections import defaultdict
from fastapi import WebSocket

class ConnectionManager:
    def __init__(self):
        self.rooms: dict[str, set[WebSocket]] = defaultdict(set)

    async def join(self, room: str, ws: WebSocket):
        await ws.accept()
        self.rooms[room].add(ws)

    def leave(self, room: str, ws: WebSocket):
        self.rooms[room].discard(ws)
        if not self.rooms[room]:
            del self.rooms[room]

    async def broadcast(self, room: str, message: dict, exclude: WebSocket | None = None):
        dead = []
        for ws in self.rooms.get(room, set()):
            if ws is exclude: continue
            try:
                await ws.send_json(message)
            except Exception:
                dead.append(ws)
        for ws in dead:
            self.leave(room, ws)

manager = ConnectionManager()
```

## 5. 鉴权 + 房间聊天

WebSocket 没法用普通 `Authorization` 头（浏览器原生 API 不让设），常见做法：
- 拼到查询参数 `?token=`（注意日志泄漏）
- 用 Cookie（同源情况下浏览器会带）
- 连上后第一条消息发 token 鉴权

```python
from fastapi import Query, status

async def ws_user(ws: WebSocket, token: str = Query(...)):
    try:
        uid = parse_token(token)
    except Exception:
        await ws.close(code=status.WS_1008_POLICY_VIOLATION)
        return None
    return uid

@app.websocket("/ws/rooms/{room}")
async def chat(ws: WebSocket, room: str, token: str = Query(...)):
    uid = await ws_user(ws, token)
    if uid is None: return
    await manager.join(room, ws)
    await manager.broadcast(room, {"type": "join", "user": uid})
    try:
        while True:
            msg = await ws.receive_json()
            await manager.broadcast(room, {"type": "msg", "from": uid, "text": msg["text"]})
    except WebSocketDisconnect:
        manager.leave(room, ws)
        await manager.broadcast(room, {"type": "leave", "user": uid})
```

## 6. 心跳保活

中间网关（Nginx / Cloudflare）默认 60s 没流量就断。两端都做 ping/pong：

```python
# 服务端：定期发 ping
import asyncio
async def ping_loop(ws: WebSocket):
    while True:
        await asyncio.sleep(20)
        try: await ws.send_json({"type": "ping"})
        except: break
```

客户端收到 `ping` 回 `pong`。WebSocket 协议层也有 ping/pong 帧，但 FastAPI 暴露起来略麻烦，应用层自己发更直观。

## 7. Redis Pub/Sub：多机分发

部署多副本时，房间里的连接散落在不同进程。用 Redis 做"广播总线"：

```bash
uv add "redis[hiredis]"
```

```python
import asyncio, json
from redis.asyncio import Redis

redis = Redis.from_url("redis://localhost:6379")

async def fan_out(room: str, message: dict):
    await redis.publish(f"room:{room}", json.dumps(message))

async def subscribe_loop():
    pubsub = redis.pubsub()
    await pubsub.psubscribe("room:*")
    async for msg in pubsub.listen():
        if msg["type"] != "pmessage": continue
        room = msg["channel"].decode().split(":", 1)[1]
        data = json.loads(msg["data"])
        # 给本进程内该房间的连接全推一份
        await manager.broadcast(room, data)
```

发消息走 `fan_out`，启动时 `asyncio.create_task(subscribe_loop())`。每个进程自己只管本地连接，跨进程靠 Redis。

## 8. SSE：服务端单向流

SSE = HTTP 响应永远不结束，每条事件一行 `data: ...\n\n`。浏览器原生 `EventSource` 自动重连。

```python
from fastapi import APIRouter
from fastapi.responses import StreamingResponse
import asyncio, json

@router.get("/sse/clock")
async def clock():
    async def gen():
        while True:
            yield f"data: {json.dumps({'now': time.time()})}\n\n"
            await asyncio.sleep(1)
    return StreamingResponse(gen(), media_type="text/event-stream")
```

事件可命名 + id：

```
event: token
id: 42
data: {"text": "hello"}

```

客户端：

```js
const es = new EventSource("/sse/clock");
es.onmessage = e => console.log(JSON.parse(e.data));
es.addEventListener("token", e => render(JSON.parse(e.data)));
```

## 9. 实战：AI 流式回复（SSE）

调 Anthropic / OpenAI 的流式接口，把 token 转推到前端。下面用 Anthropic SDK 演示：

```bash
uv add anthropic
```

```python
from anthropic import AsyncAnthropic
client = AsyncAnthropic()  # 读 ANTHROPIC_API_KEY

@router.post("/chat/stream")
async def chat_stream(body: ChatIn):
    async def gen():
        async with client.messages.stream(
            model="claude-sonnet-4-5",
            max_tokens=1024,
            messages=[{"role": "user", "content": body.prompt}],
        ) as stream:
            async for text in stream.text_stream:
                yield f"event: token\ndata: {json.dumps({'text': text})}\n\n"
            final = await stream.get_final_message()
            yield f"event: done\ndata: {json.dumps({'usage': final.usage.model_dump()})}\n\n"
    return StreamingResponse(gen(), media_type="text/event-stream")
```

前端用 `EventSource` 或 fetch + ReadableStream（POST 用后者）。

## 10. WebSocket vs SSE vs Long Polling

| 场景 | 选 | 理由 |
|---|---|---|
| 双向高频（聊天 / 游戏） | WebSocket | 全双工 |
| 服务端推为主（行情 / AI 流式） | SSE | 简单 + 自动重连 + HTTP 友好 |
| 偶尔有事（评论数刷新） | 短轮询 | 实现成本最低 |
| 浏览器不支持 WS（极少） | Long Polling | fallback |

SSE 优势常被低估：能复用 HTTP 中间件、CDN、鉴权；HTTP/2 下还能多路复用；Cloudflare / Nginx 默认放行。

## 11. 文件上传 + 静态文件

```python
from fastapi import UploadFile, File

@router.post("/upload")
async def upload(file: UploadFile = File(...)):
    if file.size and file.size > 10 * 1024 * 1024:
        raise HTTPException(413, "too large")
    path = f"./uploads/{file.filename}"
    with open(path, "wb") as f:
        while chunk := await file.read(1024 * 1024):
            f.write(chunk)
    return {"path": path, "size": file.size}
```

静态目录：

```python
from fastapi.staticfiles import StaticFiles
app.mount("/static", StaticFiles(directory="static"), name="static")
```

大文件分片：前端切块（5MB / 块）+ 各自带 `chunk_index / total / upload_id`，全收到后合并。或直接走 S3 `multipart_upload` 让客户端直传，后端只签 URL。

## 实战：实时聊天 + AI 流式问答

最少两端点：
1. `WS /ws/rooms/{room}?token=` — 房间聊天，跨进程靠 Redis Pub/Sub
2. `POST /chat/stream` — SSE 流式 AI 回复

bonus：
- WS 收到的消息持久化到 PostgreSQL（W02 已学）
- 上线时拉最近 50 条历史
- "正在输入..." 提示（broadcast typing 事件）

跑两个 uvicorn 进程（端口 8000 / 8001），前端连 8000，后端用 Redis 转发，两边都收到——这才叫"过了横向扩展这一关"。

## ✅ 本周 Checklist

- [ ] 看懂 WebSocket 握手报文
- [ ] FastAPI 写 echo / 房间聊天
- [ ] ConnectionManager 处理掉队连接
- [ ] WebSocket 鉴权（query / first message 任选）
- [ ] 心跳保活逻辑两端都有
- [ ] 跨两个进程跑 Redis Pub/Sub 广播
- [ ] SSE clock 端点正常自动重连
- [ ] AI 流式接口跑通，前端 token 一字一字出
- [ ] 三种长连接选型能口述

下周：去 Node 那边走一遭，看看同样的事 Hono 怎么做。
