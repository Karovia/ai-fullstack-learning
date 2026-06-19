---
week: 5
chapter: 05-后端开发
title: Node 与 Hono
duration: 1 周
prerequisites: [JS / TS 基础, W02]
goals:
  - 理解 Node 事件循环与 libuv
  - 熟悉 Express / Fastify / Hono / Elysia 现代生态
  - 用 Hono + Drizzle 实现一份与 FastAPI 等价的 API
created: 2026-06-19
---

# W05 · Node 与 Hono：另一种"母语"，同一种工艺

> 类比：FastAPI 是"普通话"，Node/TS 是"广东话"——都讲后端这件事，词不同，文法相通。会两种就能听懂大半个行业。

## 1. Node.js 是什么

- V8（Chrome 的 JS 引擎）+ libuv（异步 IO 库）+ 标准库
- **单线程执行 JS**，但 IO 由 libuv 线程池处理
- 事件循环六大阶段：timers → pending callbacks → idle/prepare → **poll** → check（setImmediate）→ close callbacks
- `process.nextTick` / Promise 微任务在每个阶段之间清空

与浏览器事件循环差异：浏览器只有宏/微任务两层；Node 多了几个内部阶段，所以 `setTimeout(0)` 与 `setImmediate` 顺序在不同阶段会变。

## 2. 三大基础模块

```js
// Buffer：处理二进制
const buf = Buffer.from("hello", "utf8");  // <Buffer 68 65 6c 6c 6f>

// Stream：流式处理大数据
import { createReadStream, createWriteStream } from "node:fs";
import { pipeline } from "node:stream/promises";
import { createGzip } from "node:zlib";

await pipeline(createReadStream("big.log"), createGzip(), createWriteStream("big.log.gz"));

// EventEmitter：观察者模式
import { EventEmitter } from "node:events";
const bus = new EventEmitter();
bus.on("login", user => console.log("welcome", user));
bus.emit("login", { id: 1 });
```

Stream 三种：Readable / Writable / Duplex（双工，如 Socket）/ Transform（边读边变）。

## 3. 现代 Node 项目脚手架

2026 主流：**pnpm + tsx + TypeScript**。

```bash
mkdir conduit && cd conduit
pnpm init
pnpm add -D typescript tsx @types/node
pnpm tsc --init  # 调到 "module": "ESNext", "moduleResolution": "Bundler"
```

`package.json`：

```json
{
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "start": "node --import tsx src/index.ts"
  }
}
```

`tsx` 替代 `ts-node`：更快、零配置、原生 ESM。

## 4. 框架对比

| 框架 | 风格 | 性能 | 生态 | 推荐度 |
|---|---|---|---|---|
| Express 5 | 老牌、回调 | 中 | 海量插件 | 维护老项目用 |
| Fastify | schema 驱动 + 插件 | 高 | 完整 | 企业级首选 |
| **Hono** | 类 Web Standards、轻 | 极高 | 增长快 | **2026 新项目首选** |
| Elysia | Bun 原生、类型炫技 | 极高 | 还在长 | Bun 项目用 |
| NestJS | 企业级 IoC | 中 | 完整 | 大团队、Java 风格 |

Hono 优势：
- API 像 Express 一样直观
- TS 类型推导一流（端点类型导出后前端可直接复用）
- 一份代码可跑 Node / Bun / Deno / Cloudflare Workers / AWS Lambda
- Bundle 极小（核心 ~12KB）

## 5. Hono 入门

```bash
pnpm add hono @hono/node-server
pnpm add -D @types/node
```

`src/index.ts`：

```ts
import { Hono } from "hono";
import { serve } from "@hono/node-server";
import { logger } from "hono/logger";
import { cors } from "hono/cors";

const app = new Hono();
app.use(logger(), cors());

app.get("/", c => c.json({ hello: "world" }));

app.get("/users/:id", c => {
  const id = c.req.param("id");          // 类型自动是 string
  return c.json({ id, name: "Alice" });
});

app.post("/articles", async c => {
  const body = await c.req.json<{ title: string; body: string }>();
  return c.json({ id: 1, ...body }, 201);
});

serve({ fetch: app.fetch, port: 8000 }, info => {
  console.log(`http://localhost:${info.port}`);
});
```

跑：`pnpm dev`。

## 6. 类型化路由 + 校验

用 Zod + `@hono/zod-validator` 拿到端到端类型：

```bash
pnpm add zod @hono/zod-validator
```

```ts
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";

const ArticleIn = z.object({
  title: z.string().min(1).max(120),
  body: z.string(),
  tags: z.array(z.string()).default([]),
});

const route = app.post(
  "/articles",
  zValidator("json", ArticleIn),
  c => {
    const data = c.req.valid("json");    // 类型 = z.infer<typeof ArticleIn>
    return c.json({ id: 1, ...data }, 201);
  }
);

// 导出端点类型，前端 npm 包共享
export type AppType = typeof route;
```

前端：

```ts
import { hc } from "hono/client";
import type { AppType } from "../server/src/index";

const client = hc<AppType>("http://localhost:8000");
const res = await client.articles.$post({ json: { title: "Hi", body: "..." } });
```

完全类型化的 RPC，无需 OpenAPI 生成。FastAPI 对应能力要靠 `openapi-typescript-codegen` 后处理，Hono 直接共享 TS 类型，更轻。

## 7. JWT / 中间件 / 错误处理

```ts
import { jwt, sign } from "hono/jwt";

const SECRET = process.env.JWT_SECRET!;

app.post("/auth/login", async c => {
  const { email, password } = await c.req.json();
  const user = await authenticate(email, password);
  if (!user) return c.json({ error: "bad creds" }, 401);
  const token = await sign({ sub: user.id, exp: Math.floor(Date.now()/1000) + 1800 }, SECRET);
  return c.json({ access_token: token });
});

const protect = jwt({ secret: SECRET });

app.use("/me", protect);
app.get("/me", c => {
  const payload = c.get("jwtPayload");
  return c.json({ id: payload.sub });
});

app.onError((err, c) => {
  console.error(err);
  return c.json({ error: { message: err.message } }, 500);
});
```

## 8. Drizzle ORM

类型友好的 SQL builder + migration，比 Prisma 轻、比裸 SQL 安全。

```bash
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit
```

`db/schema.ts`：

```ts
import { pgTable, serial, text, timestamp, integer } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  email: text("email").notNull().unique(),
  passwordHash: text("password_hash").notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

export const articles = pgTable("articles", {
  id: serial("id").primaryKey(),
  title: text("title").notNull(),
  body: text("body").notNull(),
  authorId: integer("author_id").references(() => users.id).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});
```

`db/client.ts`：

```ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";

const sql = postgres(process.env.DATABASE_URL!);
export const db = drizzle(sql, { schema });
```

查询：

```ts
import { eq, desc } from "drizzle-orm";

const list = await db.select().from(articles).orderBy(desc(articles.createdAt)).limit(20);
const one  = await db.select().from(articles).where(eq(articles.id, 42));
const inserted = await db.insert(articles).values({ title, body, authorId }).returning();
```

迁移：`pnpm drizzle-kit generate` → `pnpm drizzle-kit migrate`。

## 9. WebSocket / SSE in Hono

Node adapter 下 WebSocket：

```ts
import { createNodeWebSocket } from "@hono/node-ws";

const { injectWebSocket, upgradeWebSocket } = createNodeWebSocket({ app });

app.get(
  "/ws/echo",
  upgradeWebSocket(c => ({
    onMessage(evt, ws) { ws.send(`echo: ${evt.data}`); },
    onClose() { console.log("bye"); },
  }))
);

const server = serve({ fetch: app.fetch, port: 8000 });
injectWebSocket(server);
```

SSE 用 `streamSSE`：

```ts
import { streamSSE } from "hono/streaming";

app.get("/sse/clock", c =>
  streamSSE(c, async stream => {
    let i = 0;
    while (true) {
      await stream.writeSSE({ event: "tick", data: String(i++) });
      await stream.sleep(1000);
    }
  })
);
```

## 10. Edge runtime（Cloudflare Workers）

Hono 同一份代码可部署到 CF Workers——免费、全球节点、冷启动 0ms。注意：CF Workers 没 Node API（没 `fs`），数据库走 D1 / Hyperdrive / Neon serverless driver。

```bash
pnpm add -D wrangler
pnpm wrangler deploy src/index.ts
```

适合 API gateway / 鉴权 / SSR 边缘渲染；重数据/长连接用回 Node。

## 11. Bun runtime（可选）

Bun 是 JS 运行时 + 包管理 + bundler。性能比 Node 快 2-4 倍，原生 TS、原生 fetch、内置 SQLite。但生态成熟度仍落后 Node 一截，2026 更适合工具型脚本与 Edge。

```bash
bun init
bun add hono
bun --watch src/index.ts
```

## 实战：用 Hono 重做 W02 的 Conduit API

要求：
- Drizzle + PostgreSQL（用 Docker 起）
- 三资源 CRUD（users / articles / comments）
- JWT 登录 + bcrypt（用 `bcryptjs`）
- Zod 校验请求
- 端到端类型导出，前端用 `hc` 调用
- 跑通 /docs 等价物（`@hono/zod-openapi` 自动生成 OpenAPI）

完成后对比：
1. 同样的接口，Python vs TS 写起来分别需要哪些样板？
2. 类型推导在两边的体验差异
3. 部署体积（FastAPI 镜像 ~150MB；Hono Node 镜像 ~80MB；CF Workers 0）

## ✅ 本周 Checklist

- [ ] 能口述 Node 事件循环六大阶段
- [ ] 用 Stream + pipeline 处理过一个大文件
- [ ] 用 Hono + Zod 写过 5+ 端点
- [ ] 端到端类型 `hc<AppType>` 跑通
- [ ] Drizzle 完成 schema + migrate + 5 种查询
- [ ] WebSocket / SSE 在 Hono 中各跑过一个
- [ ] 把 Conduit API 用 Hono 重写完成
- [ ] 部署一次到 Cloudflare Workers（hello world 也行）

下周：把任何后端都需要的"基础设施"——限流 / 缓存 / 日志 / 队列 — 装齐。
