---
week: 2
chapter: 05-后端开发
title: FastAPI 入门
duration: 1 周
prerequisites: [W01, Python 类型注解]
goals:
  - 能独立搭起一个生产可用的 FastAPI 项目
  - 掌握路由、Pydantic、依赖注入、中间件
  - 写完整 CRUD 并通过自动文档调试
created: 2026-06-19
---

# W02 · FastAPI 入门：用类型把 API 写"自动"

> 类比：FastAPI 像"会自动给文件夹生成索引"的图书管理员。你只要把书（路由）放上架并贴上标签（Pydantic 类型），它就替你写好"目录"（OpenAPI 文档）、"借阅卡"（请求校验）、"归还单"（响应序列化）。

## 1. 为什么是 FastAPI

- **异步原生**：基于 ASGI（Starlette），单进程并发上万连接
- **类型即合约**：Pydantic 模型一处定义，三处复用（校验 / 文档 / 序列化）
- **自动 OpenAPI**：访问 `/docs` 直接交互式调试
- **依赖注入**：清晰的可测试架构
- **生态**：SQLModel / SQLAlchemy / Tortoise / Beanie 任选

直接对比 Flask：Flask 你要自己写 Marshmallow / apispec / pytest fixtures，FastAPI 全部内置。

## 2. 安装与第一个 API

用 W04 章学过的 `uv`：

```bash
uv init blog-api && cd blog-api
uv add fastapi "uvicorn[standard]" pydantic-settings
```

`main.py`：

```python
from fastapi import FastAPI

app = FastAPI(title="Blog API", version="0.1.0")

@app.get("/")
def root():
    return {"hello": "world"}
```

跑起来：

```bash
uv run uvicorn main:app --reload
```

打开 http://127.0.0.1:8000 看到 JSON，打开 /docs 看到交互文档。一行注解都没写，OpenAPI 就有了——这就是"类型驱动"的力量。

## 3. 路径参数 / 查询参数 / 请求体

```python
from fastapi import FastAPI, Query, Path
from pydantic import BaseModel, Field

app = FastAPI()

class ArticleIn(BaseModel):
    title: str = Field(min_length=1, max_length=120)
    body: str
    tags: list[str] = []

class ArticleOut(ArticleIn):
    id: int

@app.get("/articles/{article_id}")
def get_article(
    article_id: int = Path(ge=1),                   # 路径参数 + 校验
    include_comments: bool = Query(False),          # 查询参数
):
    return {"id": article_id, "title": "Hello", "body": "...", "tags": []}

@app.post("/articles", response_model=ArticleOut, status_code=201)
def create_article(payload: ArticleIn):             # 请求体（自动 422）
    return {"id": 1, **payload.model_dump()}
```

记住三条铁律：
1. 类型是 `int / float / bool / str / Enum / UUID / datetime` 等基础类型 → 路径或查询参数
2. 类型是 Pydantic `BaseModel` → 请求体
3. `response_model` 永远写上，它才是真正"出门检查"

## 4. Pydantic v2 关键技巧

```python
from pydantic import BaseModel, EmailStr, field_validator, ConfigDict
from datetime import datetime

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)

    @field_validator("password")
    @classmethod
    def strong(cls, v: str) -> str:
        if not any(c.isdigit() for c in v):
            raise ValueError("password must contain a digit")
        return v

class UserOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)  # 可从 ORM 对象转

    id: int
    email: EmailStr
    created_at: datetime
```

`from_attributes=True`（旧名 `orm_mode`）让你能 `UserOut.model_validate(user_orm_obj)` 直接转换 SQLAlchemy 对象。

## 5. 模块化：APIRouter + 项目结构

参考社区 [`fastapi-best-practices`](https://github.com/zhanymkanov/fastapi-best-practices)：

```
blog/
├── main.py
├── core/
│   ├── config.py          # pydantic-settings
│   ├── db.py              # 数据库连接
│   └── security.py
├── modules/
│   ├── users/
│   │   ├── router.py      # 路由
│   │   ├── schemas.py     # Pydantic
│   │   ├── service.py     # 业务
│   │   └── models.py      # ORM
│   └── articles/
│       └── ...
└── tests/
```

`modules/users/router.py`：

```python
from fastapi import APIRouter, Depends
from .schemas import UserOut, UserCreate
from .service import UserService, get_user_service

router = APIRouter(prefix="/users", tags=["users"])

@router.post("", response_model=UserOut, status_code=201)
async def create(payload: UserCreate, svc: UserService = Depends(get_user_service)):
    return await svc.create(payload)
```

`main.py`：

```python
from fastapi import FastAPI
from modules.users.router import router as users_router
from modules.articles.router import router as articles_router

app = FastAPI()
app.include_router(users_router, prefix="/api")
app.include_router(articles_router, prefix="/api")
```

## 6. 依赖注入（核心！）

**Depends 是 FastAPI 的灵魂**。它做四件事：

- 把"如何获得一个东西"和"如何用它"分开
- 自动在文档里展示出来（如 OAuth2、Header 校验）
- 自动 cache 同一请求中重复依赖
- 测试时可一行替换（`app.dependency_overrides`）

```python
from fastapi import Depends, Header, HTTPException

def get_db():
    db = SessionLocal()
    try:
        yield db          # yield 之前是"建立"，之后是"清理"
    finally:
        db.close()

def get_current_user(token: str = Header(alias="Authorization"), db = Depends(get_db)):
    user = decode_and_lookup(token, db)
    if not user:
        raise HTTPException(401)
    return user

def require_admin(user = Depends(get_current_user)):
    if not user.is_admin:
        raise HTTPException(403)
    return user

@router.delete("/articles/{id}")
def delete_article(id: int, _: None = Depends(require_admin)):
    ...
```

观察：`require_admin` 嵌套依赖了 `get_current_user`，后者又嵌套了 `get_db`。FastAPI 会按拓扑顺序执行，并把同一请求的 `get_db` 共享。

## 7. 中间件

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

自定义中间件（请求 ID + 耗时）：

```python
import time, uuid
from starlette.middleware.base import BaseHTTPMiddleware

class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        rid = request.headers.get("X-Request-ID", uuid.uuid4().hex)
        start = time.perf_counter()
        response = await call_next(request)
        response.headers["X-Request-ID"] = rid
        response.headers["X-Process-Time"] = f"{time.perf_counter() - start:.3f}"
        return response

app.add_middleware(RequestIDMiddleware)
```

## 8. 全局异常处理

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class BizError(Exception):
    def __init__(self, code: str, message: str, status: int = 400):
        self.code, self.message, self.status = code, message, status

@app.exception_handler(BizError)
async def biz_error_handler(req: Request, exc: BizError):
    return JSONResponse(
        status_code=exc.status,
        content={"error": {"code": exc.code, "message": exc.message}},
    )
```

业务里直接 `raise BizError("EMAIL_TAKEN", "邮箱已注册", 409)`，前端拿到统一结构。

## 9. 后台任务

```python
from fastapi import BackgroundTasks

def send_welcome_email(email: str):
    ...  # 真正发邮件，可能耗时

@router.post("/users")
def register(payload: UserCreate, bg: BackgroundTasks, svc=Depends(get_user_service)):
    user = svc.create(payload)
    bg.add_task(send_welcome_email, user.email)   # 响应返回后才执行
    return user
```

注意：BackgroundTasks 是**同进程**任务，崩了就没。要可靠投递走 W06 的任务队列。

## 10. lifespan：启动 / 关闭

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动：连数据库、连 Redis、预热模型
    app.state.redis = await create_redis_pool()
    yield
    # 关闭：优雅释放
    await app.state.redis.close()

app = FastAPI(lifespan=lifespan)
```

## 实战：CRUD（用户 / 文章 / 评论）

最小可用 SQLite + SQLModel：

```bash
uv add sqlmodel
```

```python
# core/db.py
from sqlmodel import SQLModel, create_engine, Session

engine = create_engine("sqlite:///app.db")

def init_db(): SQLModel.metadata.create_all(engine)
def get_session():
    with Session(engine) as s: yield s
```

```python
# modules/articles/models.py
from sqlmodel import SQLModel, Field
from datetime import datetime

class Article(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    title: str
    body: str
    author_id: int = Field(foreign_key="user.id")
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

```python
# modules/articles/router.py
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session, select
from core.db import get_session
from .models import Article
from .schemas import ArticleIn, ArticleOut

router = APIRouter(prefix="/articles", tags=["articles"])

@router.get("", response_model=list[ArticleOut])
def list_articles(skip: int = 0, limit: int = 20, db: Session = Depends(get_session)):
    return db.exec(select(Article).offset(skip).limit(limit)).all()

@router.post("", response_model=ArticleOut, status_code=201)
def create(payload: ArticleIn, db: Session = Depends(get_session), me=Depends(get_current_user)):
    article = Article(**payload.model_dump(), author_id=me.id)
    db.add(article); db.commit(); db.refresh(article)
    return article

@router.get("/{id}", response_model=ArticleOut)
def get(id: int, db: Session = Depends(get_session)):
    a = db.get(Article, id)
    if not a: raise HTTPException(404)
    return a

@router.put("/{id}", response_model=ArticleOut)
def update(id: int, payload: ArticleIn, db: Session = Depends(get_session), me=Depends(get_current_user)):
    a = db.get(Article, id)
    if not a: raise HTTPException(404)
    if a.author_id != me.id: raise HTTPException(403)
    for k, v in payload.model_dump().items(): setattr(a, k, v)
    db.add(a); db.commit(); db.refresh(a)
    return a

@router.delete("/{id}", status_code=204)
def delete(id: int, db: Session = Depends(get_session), me=Depends(get_current_user)):
    a = db.get(Article, id)
    if not a: raise HTTPException(404)
    if a.author_id != me.id: raise HTTPException(403)
    db.delete(a); db.commit()
```

文章 + 评论 + 用户三表自己照着扩。

打开 /docs，点 Authorize 输入 token，逐个端点点 Try it out 跑通。

## ✅ 本周 Checklist

- [ ] 能默写最小 FastAPI 应用
- [ ] 路径参数 / 查询参数 / 请求体三种参数熟练
- [ ] Pydantic v2 校验、`response_model`、`from_attributes` 都用过
- [ ] 项目按 modules 拆分，至少 3 个模块
- [ ] Depends 嵌套至少 3 层（db → user → admin）
- [ ] CORS、GZip、自定义中间件各一个
- [ ] 全局异常处理输出统一结构
- [ ] BackgroundTasks 发了一封"邮件"（先 print 即可）
- [ ] CRUD 三资源全部跑通，/docs 可交互测试

下周：把"谁能进来"和"谁能干嘛"这两件事讲透。
