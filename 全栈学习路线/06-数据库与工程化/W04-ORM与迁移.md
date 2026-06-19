---
chapter: 06
week: 04
title: ORM 与迁移
type: weekly-tutorial
prerequisites: [06-W03]
estimated_hours: 20
difficulty: ★★★★☆
tags: [orm, sqlalchemy, alembic, migration, n+1, drizzle, prisma]
created: 2026-06-19
---

# 第 06 章 · 第 04 周 · ORM 与迁移

## 一、ORM 是什么

**类比**：你在中国，对方在法国，ORM 是同声传译——你说 Python（对象），数据库听 SQL；它中间翻译。

```python
# 你写：
user = await session.get(User, 1)
user.name = "alice"
await session.commit()

# ORM 翻译成：
SELECT * FROM users WHERE id = 1;
UPDATE users SET name = 'alice' WHERE id = 1;
```

**好处**：
- 代码层面对象化，IDE 补全、类型检查
- schema 改了，迁移工具自动生成 ALTER
- 切数据库（PG ↔ MySQL）改一行配置

**坏处**：
- 一不小心 N+1 查询，性能崩
- 复杂 SQL 用 ORM 写得比裸 SQL 还啰嗦
- 多一层抽象，调试时多一层堆栈

**结论**：日常 CRUD 用 ORM，复杂报表/性能瓶颈处用裸 SQL（ORM 都支持 `text()` 转写）。

---

## 二、SQLAlchemy 2.0 新风格

2.0 抛弃了老 `Query` API，全力推**Core + ORM 统一的 select() 风格**，也引入了 `Mapped[T]` 类型注解。

### 安装

```bash
pip install "sqlalchemy[asyncio]>=2.0" asyncpg alembic
```

### Engine + Session

```python
# db.py
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

DATABASE_URL = "postgresql+asyncpg://dev:devpass@localhost:5432/playground"

engine = create_async_engine(
    DATABASE_URL,
    echo=False,            # 调试改 True 看 SQL
    pool_size=20,
    max_overflow=10,
    pool_pre_ping=True,    # 防"已断开"连接
    pool_recycle=1800,
)

SessionLocal = async_sessionmaker(engine, expire_on_commit=False, class_=AsyncSession)

async def get_session():
    async with SessionLocal() as s:
        yield s
```

### 模型定义

```python
# models.py
from datetime import datetime
import uuid
from sqlalchemy import String, Integer, ForeignKey, Boolean, Text, UniqueConstraint, Index
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from sqlalchemy.dialects.postgresql import UUID, JSONB

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id:         Mapped[uuid.UUID] = mapped_column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    email:      Mapped[str]       = mapped_column(String(255), unique=True, index=True, nullable=False)
    username:   Mapped[str]       = mapped_column(String(50),  unique=True, nullable=False)
    password:   Mapped[str]       = mapped_column(String(200), nullable=False)
    is_active:  Mapped[bool]      = mapped_column(Boolean, default=True, nullable=False)
    created_at: Mapped[datetime]  = mapped_column(default=datetime.utcnow, nullable=False)

    posts: Mapped[list["Post"]] = relationship(back_populates="author", cascade="all, delete-orphan")

class Post(Base):
    __tablename__ = "posts"

    id:         Mapped[int]  = mapped_column(primary_key=True)
    author_id:  Mapped[uuid.UUID] = mapped_column(ForeignKey("users.id", ondelete="CASCADE"), index=True)
    title:      Mapped[str]  = mapped_column(String(200), nullable=False)
    body:       Mapped[str]  = mapped_column(Text, nullable=False)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow, index=True)

    author: Mapped["User"] = relationship(back_populates="posts")
    tags:   Mapped[list["Tag"]] = relationship(secondary="post_tags", back_populates="posts")

    __table_args__ = (
        Index("ix_post_author_created", "author_id", "created_at"),
        UniqueConstraint("author_id", "title", name="uq_author_title"),
    )

class Tag(Base):
    __tablename__ = "tags"
    id:   Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50), unique=True)
    posts: Mapped[list["Post"]] = relationship(secondary="post_tags", back_populates="tags")

class PostTag(Base):
    __tablename__ = "post_tags"
    post_id: Mapped[int] = mapped_column(ForeignKey("posts.id", ondelete="CASCADE"), primary_key=True)
    tag_id:  Mapped[int] = mapped_column(ForeignKey("tags.id",  ondelete="CASCADE"), primary_key=True)
```

### 关系类型

| 关系 | 模型写法 |
|---|---|
| **one-to-many** | parent: `relationship(...)`；child: `ForeignKey + back_populates` |
| **many-to-one** | 上面的反面 |
| **many-to-many** | 中间表 + `secondary=` |
| **one-to-one** | `relationship(uselist=False)` 或外键加 unique |

---

## 三、查询

```python
from sqlalchemy import select, func
from sqlalchemy.orm import selectinload, joinedload

# 1. 主键拿对象
user = await session.get(User, user_id)

# 2. select where order limit
stmt = (select(Post)
        .where(Post.author_id == author_id)
        .order_by(Post.created_at.desc())
        .limit(20))
posts = (await session.execute(stmt)).scalars().all()

# 3. JOIN
stmt = (select(Post, User)
        .join(User, Post.author_id == User.id)
        .where(User.is_active == True))
rows = (await session.execute(stmt)).all()
for post, user in rows: ...

# 4. 聚合
stmt = select(func.count()).select_from(Post).where(Post.author_id == author_id)
total = (await session.execute(stmt)).scalar_one()

# 5. group by
stmt = (select(User.id, func.count(Post.id))
        .join(Post, isouter=True)
        .group_by(User.id))

# 6. 原生 SQL
from sqlalchemy import text
rows = await session.execute(text("SELECT * FROM users WHERE email = :e"), {"e": "a@x.com"})
```

---

## 四、N+1 问题诊断与解决

**问题代码**：

```python
users = (await session.execute(select(User).limit(20))).scalars().all()
for u in users:
    print(u.username, len(u.posts))   # 每个 u.posts 触发一次 SQL！
```

→ 1 + 20 次 SQL，慢。

**加载策略**：

| 策略 | 行为 | 适合 |
|---|---|---|
| **lazy**（默认） | 用到才查 | 不一定用到 |
| **select** | 同上 | 不推荐 |
| **joined** | LEFT JOIN 一次拉到 | one-to-one 或子集小 |
| **subquery** | 子查询 | 较老方案 |
| **selectin** | `IN (id, id, ...)` 二次查询 | **one-to-many 推荐** |

```python
# 解决：selectinload
stmt = select(User).options(selectinload(User.posts)).limit(20)

# 或单查询 joinedload（注意会乘行数）
stmt = select(User).options(joinedload(User.posts)).limit(20)
```

`echo=True` 启动看实际 SQL 是定位 N+1 的最快方法。

---

## 五、Alembic 迁移

```bash
alembic init alembic
```

改 `alembic.ini` 的 `sqlalchemy.url`，改 `env.py` 把 `target_metadata = Base.metadata`。

### 自动生成

```bash
alembic revision --autogenerate -m "create users and posts"
alembic upgrade head
alembic downgrade -1
alembic current
alembic history
```

**重要**：autogenerate **不是万能**：
- 改列类型：可能漏，要手工补
- 列改名：会被识别成 drop + add，**数据会丢**！手动改成 `op.alter_column(...)`
- 自定义类型 / 枚举：要手工处理

### 数据迁移

```python
def upgrade():
    op.add_column("users", sa.Column("display_name", sa.String(50)))
    # 数据迁移
    op.execute("UPDATE users SET display_name = username")
    op.alter_column("users", "display_name", nullable=False)
```

---

## 六、连接池与读写分离

```python
# 主库
write_engine = create_async_engine(DATABASE_URL_WRITE, pool_size=20)
# 只读库（多个）
read_engine  = create_async_engine(DATABASE_URL_READ,  pool_size=40)

class RoutedSession(AsyncSession):
    async def get_bind(self, mapper=None, clause=None, **kw):
        if self._flushing or self.in_transaction():
            return write_engine.sync_engine
        # SELECT 默认走只读
        if clause is not None and clause.is_select:
            return read_engine.sync_engine
        return write_engine.sync_engine
```

**连接池配置原则**：
- `pool_size`：常驻连接数，CPU 核数 × 2 起步
- `max_overflow`：突发额外连接，pool_size 的 50%
- `pool_pre_ping=True`：防"中间网络断了"
- `pool_recycle=1800`：30 分钟回收一次

---

## 七、Drizzle ORM（Node 现代选择）

```ts
import { drizzle } from "drizzle-orm/node-postgres";
import { pgTable, text, integer, timestamp } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: text("id").primaryKey(),
  email: text("email").notNull().unique(),
  createdAt: timestamp("created_at").defaultNow(),
});

const db = drizzle(pool);
const list = await db.select().from(users).where(eq(users.email, "a@x.com")).limit(10);
```

特点：TypeScript 一等公民、SQL-like API、零运行时反射、支持 edge runtime。

---

## 八、ORM 对比表

| 工具 | 语言 | 特点 |
|---|---|---|
| **SQLAlchemy 2.0** | Python | 业界天花板，复杂查询表达力强 |
| **Django ORM** | Python | 跟 Django 强绑定，写 web 极快 |
| **Tortoise / SQLModel** | Python | 轻量，async 友好，复杂查询弱 |
| **Prisma** | TS | schema → 自动生成类型 + client，DX 高 |
| **Drizzle** | TS | SQL-like，性能好，无运行时反射 |
| **TypeORM** | TS | 老牌，问题不少 |
| **GORM** | Go | 生态最全 |
| **sqlx / sqlc** | Go/Rust | 写 SQL + 生成类型，反 ORM 派 |

---

## 九、本周实战：用 SQLAlchemy 2.0 + Alembic 重写 Conduit 模型层

W3 的 Conduit（你已经用裸 SQL 或 Tortoise 写过）改造成 SQLAlchemy 2.0：

1. 模型：`User / Article / Comment / Tag / Follow / Favorite`
2. Alembic 初始迁移 + 一次"加列"迁移演示
3. 接口实现注意 N+1：列表接口加 `selectinload(Article.author).selectinload(...)`
4. 用 pytest + httpx 跑通现有测试套件
5. 用 `echo=True` 抓出每个接口实际 SQL，写一份"SQL 审计报告"附在 README

---

## 十、Checklist

- [ ] 用 `Mapped[]` 写得出 4 种关系
- [ ] 知道 selectinload vs joinedload 何时选哪个
- [ ] N+1 问题能现场演示并修复
- [ ] Alembic 自动迁移用过、改名场景手动改过
- [ ] 连接池 4 个参数知道怎么调
- [ ] Conduit 模型层用 ORM 重写跑通

下一周走出关系型，进入**向量与全文搜索**——AI 时代的检索基础。
