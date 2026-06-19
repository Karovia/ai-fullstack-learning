---
week: 1
chapter: 05-后端开发
title: HTTP 与 REST
duration: 1 周
prerequisites: [Python, JS 基础]
goals:
  - 彻底理解 HTTP 请求/响应模型
  - 掌握 9 大方法、常见状态码、关键 Headers
  - 看懂并能写 OpenAPI 3.1 规范
  - 用 curl/HTTPie 调试任意 REST API
created: 2026-06-19
---

# W01 · HTTP 与 REST：所有后端的"母语"

> 类比：HTTP 是邮局的"信件协议"。你写的不是信，是按格式填的明信片：
> 第一行写"做什么 + 寄给谁"（method + URL），中间写收件须知（headers），最后才是正文（body）。对方收到后，回一张同样格式的明信片，第一行写"成功了 / 失败了"（status code）。

学会任何后端框架之前，先把"信件格式"看懂。这一周不写一行业务代码，但会决定你后面 7 周写得有多稳。

## 1. HTTP 是什么

HTTP（HyperText Transfer Protocol）是**应用层协议**，跑在 TCP 之上。它的核心模型只有一句话：**客户端发请求，服务端发响应，连接结束**。这是"无状态"的本质——每个请求都是独立的，服务器默认不记得你是谁。

为什么要无状态？因为简单。一台机器挂了，请求扔到另一台也能处理；要扩容只需多加机器。所有"记住用户"的能力（Cookie / Session / Token）都是后来"贴"在无状态之上的补丁。

### 一次请求长这样

```
POST /api/articles HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
Content-Length: 58

{"title": "Hello", "body": "First post"}
```

四部分：
1. **请求行**：`method  路径  协议版本`
2. **Headers**：每行 `Key: Value`，一直到空行
3. **空行**（必须）
4. **Body**：可选，根据 `Content-Length` 或 `Transfer-Encoding` 判断长度

响应同理：

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/articles/42

{"id": 42, "title": "Hello"}
```

第一行是 `协议  状态码  原因短语`，后面同样是 headers + 空行 + body。

## 2. 9 大方法与幂等性

| 方法 | 用途 | 安全 | 幂等 | 有 Body |
|---|---|---|---|---|
| GET | 读取资源 | 是 | 是 | 否 |
| HEAD | 只取响应头 | 是 | 是 | 否 |
| OPTIONS | 询问支持的方法（CORS 预检） | 是 | 是 | 否 |
| POST | 创建资源 / 触发动作 | 否 | **否** | 是 |
| PUT | 整体替换资源 | 否 | 是 | 是 |
| PATCH | 局部更新 | 否 | 否（约定俗成不保证） | 是 |
| DELETE | 删除资源 | 否 | 是 | 否 |
| TRACE | 调试回显 | 是 | 是 | 否 |
| CONNECT | 建立隧道（HTTPS 代理） | 否 | 否 | 否 |

**幂等（idempotent）**：同样请求发 1 次或 N 次，服务器最终状态相同。GET 当然幂等；DELETE 也是幂等的（删第二次只是 404）；PUT 把资源整体替换为某个值，自然幂等；POST 每次创建一个新资源，所以**不幂等**。

为什么要懂幂等？**网络重试只能重试幂等方法**。前端、网关、运维做超时重试时，POST 必须特别小心，否则一笔订单付两次。

## 3. 状态码地图

记住"百位数语义"：
- **1xx** 信息（很少见）
- **2xx** 成功
- **3xx** 重定向
- **4xx** 客户端错误（你写错了）
- **5xx** 服务端错误（我崩了）

必背 16 个：

| 码 | 含义 | 何时用 |
|---|---|---|
| 200 OK | 通用成功 | GET 成功 |
| 201 Created | 已创建 | POST 创建资源后，配合 `Location` |
| 204 No Content | 成功但无返回 | DELETE 成功 |
| 301 / 302 | 永久 / 临时重定向 | 旧 URL 搬家 |
| 304 Not Modified | 未变更（缓存有效） | 带 If-None-Match 时 |
| 400 Bad Request | 请求格式错 | JSON 解析失败 |
| 401 Unauthorized | 未认证 | 没带或带错 Token |
| 403 Forbidden | 已认证但无权限 | 普通用户访问管理员接口 |
| 404 Not Found | 资源不存在 | id 找不到 |
| 409 Conflict | 冲突 | 注册重名邮箱 |
| 422 Unprocessable | 校验失败 | Pydantic / Zod 校验未过 |
| 429 Too Many Requests | 限流 | 刷接口被挡 |
| 500 Internal Server Error | 服务器自身 bug | 未捕获异常 |
| 502 Bad Gateway | 网关上游错 | Nginx 后端挂了 |
| 503 Service Unavailable | 服务不可用 | 维护中、过载 |

**401 vs 403 永远搞混**：401 = "你是谁？"，403 = "我知道你是谁，但你不行。"

## 4. 关键 Headers

- `Content-Type: application/json` — 告诉对方 body 的格式
- `Accept: application/json` — 我希望你回什么格式
- `Authorization: Bearer <token>` — 凭证
- `Cookie: sid=xxx` — 由浏览器自动带上
- `Set-Cookie: sid=xxx; HttpOnly; Secure; SameSite=Lax` — 服务端下发
- `Cache-Control: max-age=3600, public` — 缓存策略
- `ETag: "abc123"` + `If-None-Match` — 协商缓存
- `Access-Control-Allow-Origin: https://app.com` — CORS 核心
- `X-Request-ID: <uuid>` — 链路追踪自定义头
- `X-Forwarded-For` — 客户端真实 IP（经反代后）

### Cookie / Session / Token

- **Cookie**：浏览器存储的小块数据，每次自动随请求发到同源服务器。
- **Session**：服务端存"sid → 用户信息"，下发 sid 到 Cookie。**有状态**，扩容要共享存储（如 Redis）。
- **Token (JWT)**：服务端不存，用户每次自带；**无状态**，但很难主动失效（要靠黑名单或短过期）。

## 5. HTTPS 与 TLS（30 秒版）

明文 HTTP 谁都能偷看。HTTPS = HTTP + TLS。TLS 握手简化：
1. 客户端打招呼：我支持哪些加密套件 + 一个随机数
2. 服务端：选定套件 + 证书（含公钥）+ 一个随机数
3. 客户端验证证书（CA 签的吗？域名对吗？没过期吗？），生成"预主密钥"，用公钥加密发回
4. 双方各自算出对称密钥，之后所有通信用对称加密（快）

证书免费签发：**Let's Encrypt**（W08 用 Caddy 自动签）。

## 6. HTTP/1.1 vs 2 vs 3

- **1.1**：文本协议，一个 TCP 连接一次只能跑一个请求（队头阻塞）；可 keep-alive 复用 TCP
- **2**：二进制分帧，一个连接多路复用，头部压缩（HPACK）；但仍跑在 TCP 上
- **3**：基于 QUIC（UDP），解决 TCP 队头阻塞；移动网络切换 IP 不断流

知道差别就够，写应用代码很少直接感知。

## 7. REST 是什么

Roy Fielding 2000 年博士论文提出的 6 条架构约束：

1. **Client-Server**：前后端分离
2. **Stateless**：每个请求自带所有信息
3. **Cacheable**：响应必须能被缓存（通过 headers 控制）
4. **Uniform Interface**：URL 标识资源 + HTTP 方法表达动作
5. **Layered System**：客户端不知道中间有几层网关
6. **Code on Demand**（可选）：服务端可下发代码到客户端执行

工业界 99% 说的"RESTful"只是**第 4 条**：用 URL 表示资源、用方法表达动作。真严格的 REST 还要 HATEOAS（响应里包含可继续操作的链接），但太繁琐，少有人用。

### 资源命名规范

- 用名词复数：`/users`, `/articles`，不是 `/getUser`
- 嵌套表达从属：`/articles/42/comments`
- 集合用 GET，创建用 POST，单个资源用 PUT/PATCH/DELETE
- 过滤排序分页用查询参数：`/articles?tag=python&page=2&size=20&sort=-created_at`
- 动作型接口（少用）：`/articles/42/publish` 用 POST

## 8. OpenAPI 3.1（API 的"图纸"）

OpenAPI 是描述 REST API 的 YAML/JSON 规范，写完可以：
- 自动生成文档（Swagger UI / Redoc）
- 自动生成客户端 SDK（任意语言）
- Mock 服务、契约测试

最小例子：

```yaml
openapi: 3.1.0
info:
  title: Blog API
  version: 1.0.0
paths:
  /articles:
    get:
      summary: 列出文章
      parameters:
        - name: tag
          in: query
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/Article' }
    post:
      summary: 创建文章
      security: [{ bearerAuth: [] }]
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/ArticleCreate' }
      responses:
        '201':
          description: Created
components:
  schemas:
    Article:
      type: object
      required: [id, title, body]
      properties:
        id: { type: integer }
        title: { type: string }
        body: { type: string }
    ArticleCreate:
      type: object
      required: [title, body]
      properties:
        title: { type: string, minLength: 1 }
        body: { type: string }
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

好消息：W02 起 FastAPI 会**自动**生成这个文档，你只要写 Pydantic 模型即可。

## 9. 调试工具对比

- **curl**：必学，所有服务器都有
- **HTTPie**：`http GET api.example.com/users name==alice`，人类友好
- **Postman**：图形化，团队共享集合，有 mock 和测试
- **Bruno**：开源 + 文件存储 + 走 git，新宠

## 实战练习

### 任务 A：用 curl 调 GitHub API

```bash
# 1. 列出某用户的仓库
curl -i https://api.github.com/users/torvalds/repos

# 2. 用 token 创建一个 issue（先去 GitHub 设置 PAT）
curl -X POST \
  -H "Authorization: Bearer $GH_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -d '{"title":"hello","body":"from curl"}' \
  https://api.github.com/repos/<你>/sandbox/issues

# 3. 看 rate limit
curl -I -H "Authorization: Bearer $GH_TOKEN" https://api.github.com/users/octocat
# 关注响应头里的 X-RateLimit-Limit / X-RateLimit-Remaining
```

### 任务 B：设计博客 REST API

资源：User / Article / Comment / Tag。要求：
- 用 OpenAPI 3.1 描述至少 8 个端点
- 包含分页、排序、过滤
- 给出 4 个错误响应（401/403/404/422）
- 用 [editor.swagger.io](https://editor.swagger.io) 在线校验通过

提示端点：
```
POST   /auth/register
POST   /auth/login
GET    /articles?tag=&page=&author=
POST   /articles
GET    /articles/{slug}
PUT    /articles/{slug}
DELETE /articles/{slug}
POST   /articles/{slug}/comments
GET    /articles/{slug}/comments
DELETE /comments/{id}
GET    /tags
```

## ✅ 本周 Checklist

- [ ] 能口述 HTTP 请求/响应四部分
- [ ] 9 大方法及其幂等性写在脑子里
- [ ] 16 个状态码闭着眼能用
- [ ] 区分 401 vs 403、Cookie vs Token
- [ ] 看懂 TLS 握手大致流程
- [ ] 能写 OpenAPI 描述 10+ 端点
- [ ] curl 三件套（GET/POST/带 header）熟练
- [ ] GitHub API 跑通 3 个调用

下一周：用 FastAPI 把这套 OpenAPI"翻译"成可运行的服务。
