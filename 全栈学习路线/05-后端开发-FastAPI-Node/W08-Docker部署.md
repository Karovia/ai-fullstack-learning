---
week: 8
chapter: 05-后端开发
title: Docker 与部署
duration: 1 周
prerequisites: [W02-W07]
goals:
  - 写出生产级 Dockerfile（多阶段 + 缓存友好）
  - docker-compose 一键起整套服务
  - 把项目部署到 Fly.io，配域名 + HTTPS
created: 2026-06-19
---

# W08 · Docker 与部署：让世界访问到你写的服务

> 类比：**集装箱**。古代船运每家货物形状不同，装卸要重新摆放，又慢又乱。集装箱标准化之后，码头机器一抓一放——管它是袜子还是冰箱。Docker 就是软件的集装箱：你的应用 + 依赖 + 系统库都打成一个标准包，谁的机器都能跑。

## 1. 镜像 vs 容器

- **镜像（Image）**：只读模板。"菜谱 + 食材"。
- **容器（Container）**：镜像运行起来的实例。"做出来的那盘菜"。
- 一镜多容器：从同一镜像可启 N 个容器，互不影响。

镜像是分层的：每条 Dockerfile 指令产生一层。多个镜像可共享底层（Linux base），所以拉镜像比看起来快。

## 2. Dockerfile 全语法

```dockerfile
FROM python:3.13-slim AS base

# 元数据
LABEL maintainer="you@example.com"

# 环境变量
ENV PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PYTHONDONTWRITEBYTECODE=1

# 工作目录
WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential libpq-dev curl \
    && rm -rf /var/lib/apt/lists/*

# 拷贝依赖清单（先拷依赖，提高缓存命中）
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev

# 拷代码
COPY . .

# 暴露端口（声明用，实际靠 -p 映射）
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# 启动命令
CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

要点：
- `RUN`：构建时执行，产生一层。能合并就合并（`&&`），层少镜像小。
- `COPY` vs `ADD`：默认用 `COPY`，`ADD` 多了解压 + URL 拉取，反而易出问题。
- `CMD` vs `ENTRYPOINT`：`ENTRYPOINT ["python"]` + `CMD ["main.py"]` = `python main.py`，且参数可被 `docker run` 覆盖。简单场景只写 `CMD` 即可。
- `HEALTHCHECK`：Docker 自动探活，结合 compose 决定服务是否 ready。

## 3. 多阶段构建（必学）

构建期 vs 运行期分离，把编译产物拷出来，丢掉编译器：

```dockerfile
# ----- Stage 1: 构建 -----
FROM python:3.13-slim AS builder

ENV UV_LINK_MODE=copy
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends build-essential libpq-dev \
    && rm -rf /var/lib/apt/lists/*
RUN pip install uv

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

# ----- Stage 2: 运行 -----
FROM python:3.13-slim AS runtime

ENV PATH="/app/.venv/bin:$PATH" PYTHONUNBUFFERED=1
WORKDIR /app

# 只装运行时需要的系统库（不要 build-essential）
RUN apt-get update && apt-get install -y --no-install-recommends libpq5 curl \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/.venv /app/.venv
COPY . .

USER 1000:1000   # 不用 root 跑业务进程
EXPOSE 8000
HEALTHCHECK CMD curl -f http://localhost:8000/health || exit 1
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

效果：单阶段镜像 ~600MB → 多阶段 ~150MB。

Node/Hono 同理：

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

FROM node:22-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json .
USER node
CMD ["node", "dist/index.js"]
```

## 4. .dockerignore 必写

```
.git
.venv
__pycache__
*.pyc
.pytest_cache
htmlcov
.env
.env.*
node_modules
dist
.DS_Store
```

不写的话 `COPY . .` 会把上百兆文件塞进镜像 + context，构建巨慢。

## 5. 缓存层优化

Docker 按指令缓存：上面没变的层复用，下面失效。**把变得最少的放上面**：

1. base image
2. 系统 apt 安装
3. 依赖清单拷贝 + 安装
4. 代码拷贝
5. 启动命令

99% 改代码不会让前 3 步重跑，构建几秒搞定。

## 6. docker compose（多服务编排）

`compose.yml`：

```yaml
services:
  api:
    build: ./api
    image: conduit/api:dev
    env_file: ./api/.env
    ports: ["8000:8000"]
    depends_on:
      db: { condition: service_healthy }
      redis: { condition: service_started }
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 10s
      retries: 5

  worker:
    build: ./api
    command: arq app.tasks.WorkerSettings
    env_file: ./api/.env
    depends_on: [redis]

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: conduit
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "app"]
      interval: 5s

  redis:
    image: redis:7-alpine
    volumes: [redisdata:/data]

  caddy:
    image: caddy:2-alpine
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddydata:/data
    depends_on: [api]

volumes:
  pgdata:
  redisdata:
  caddydata:
```

操作：

```bash
docker compose up -d --build
docker compose logs -f api
docker compose exec db psql -U app conduit
docker compose down       # 保留卷
docker compose down -v    # 删卷（清数据）
```

## 7. Volumes / Networks / Env / Secrets

- **Volumes**：数据持久化（PG 数据、上传文件）。命名卷比 bind mount 更可控。
- **Networks**：默认 compose 建一个，服务之间用服务名互通（`http://api:8000`）。
- **Env**：`env_file:` 指 `.env`，**永远不要**把 `.env` 提交到 git。`.env.example` 留模板。
- **Secrets**：生产用 `docker secret` 或平台级（Fly secrets / GH Actions secrets），不要写在 compose 里。

## 8. 反向代理：Nginx vs Caddy

**Caddy** 更省事，自动 Let's Encrypt：

```caddy
api.example.com {
    reverse_proxy api:8000
    encode gzip zstd
    log {
        output file /data/access.log
    }
}

example.com, www.example.com {
    redir https://app.example.com{uri}
}

app.example.com {
    root * /srv/app
    try_files {path} /index.html
    file_server
    encode gzip zstd
}
```

启动：`docker compose up -d caddy`，DNS A 记录指过来——HTTPS 自动签好。

Nginx 同样能做但要手动管证书（certbot）+ 写更多配置。新项目首选 Caddy。

## 9. CI/CD：GitHub Actions

`.github/workflows/deploy.yml`：

```yaml
name: build-and-deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.13" }
      - run: pip install uv && uv sync
      - run: uv run pytest --cov=app

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: ./api
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/api:latest
            ghcr.io/${{ github.repository }}/api:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

`cache-from/to: gha` 让构建跨 PR 共享缓存，CI 从 5 分钟降到 1 分钟。

## 10. 部署平台对比

| 平台 | 类型 | 上手 | 适合 |
|---|---|---|---|
| Vercel | Serverless | 极易 | 前端 + Edge API |
| Cloudflare Workers / Pages | Edge | 易 | Hono、JAMstack、全球分发 |
| Fly.io | 容器 + 全球 | 易 | 全栈 + 实时（WebSocket 友好） |
| Railway | 容器 PaaS | 极易 | MVP / 个人项目 |
| Render | 容器 PaaS | 极易 | 类 Heroku 替代 |
| AWS ECS / Fargate | 容器 + 大全套 | 中 | 企业级 |
| GCP Cloud Run | 容器 Serverless | 中 | Google 生态 |
| 自建 VPS（Hetzner / DigitalOcean） | 裸 | 中 | 学习 / 极致省钱 |

教学路径：MVP 用 Fly/Railway，规模上来再考虑 AWS/GCP。

## 11. 部署到 Fly.io（实战）

```bash
brew install flyctl
flyctl auth signup
cd api
flyctl launch --no-deploy   # 生成 fly.toml
```

`fly.toml` 关键：

```toml
app = "conduit-api"
primary_region = "nrt"

[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "8000"

[http_service]
  internal_port = 8000
  force_https = true
  auto_stop_machines = "stop"
  auto_start_machines = true
  min_machines_running = 1

[[services.tcp_checks]]
  interval = "15s"
  timeout = "2s"
  grace_period = "10s"
```

```bash
flyctl secrets set DATABASE_URL=... JWT_SECRET=... ANTHROPIC_API_KEY=...
flyctl postgres create --name conduit-db --region nrt
flyctl postgres attach conduit-db
flyctl deploy
flyctl logs
flyctl certs add api.example.com    # 然后 DNS 加 CNAME
```

WebSocket、SSE 在 Fly 都原生支持。`min_machines_running=1` 防止冷启动。

## 12. 蓝绿 / 金丝雀（概念）

- **蓝绿**：两套环境（蓝旧 / 绿新），切流量瞬时；要双倍资源
- **金丝雀**：先把 5% 流量打给新版本，盯指标 → 50% → 100%

云平台多内置（Fly machines、AWS CodeDeploy、Argo Rollouts）。新手上线知道有这玩意，先用平台默认滚动更新即可。

## 13. 监控与告警

- **可用性**：UptimeRobot / Better Stack 每分钟探活
- **错误率**：Sentry（W06）
- **指标**：Prometheus + Grafana（Fly 内置 metrics）
- **日志**：Loki / Better Stack
- **告警**：Slack / 邮件 / 飞书 webhook

入门最少装两个：UptimeRobot（免费）+ Sentry（已学）。

## 实战：把 Conduit + 聊天后端部署上线

里程碑：
1. `docker compose up -d` 本地一键起 api + worker + db + redis + caddy
2. 写好 `.dockerignore`，镜像 ≤ 200MB
3. GitHub Actions：push main → 跑测试 → 构建推 ghcr → 部署 Fly
4. Fly 上配置 secrets，挂 PG，加自定义域名 + HTTPS
5. 浏览器打开 https://api.<你域名>/docs 能用
6. UptimeRobot 加监控，故意停一下 Fly machine 看告警

## ✅ 本周 Checklist

- [ ] 写过多阶段 Dockerfile（API 镜像 ≤ 200MB）
- [ ] `.dockerignore` 完整
- [ ] compose 起 5 个服务一键跑通
- [ ] Caddy 自动 HTTPS
- [ ] GitHub Actions 跑通 test → build → deploy
- [ ] Fly.io 部署成功，自定义域名 HTTPS 可访问
- [ ] WebSocket 在线上能正常连
- [ ] Sentry / UptimeRobot 告警链路打通
- [ ] 写一份 README，新人 5 分钟能跑起本地环境

恭喜！你已具备独立交付一个生产级后端服务的能力。下一章建议：**数据库进阶**（索引 / 事务 / 复制 / 分库分表）和**系统设计**面试材料。
