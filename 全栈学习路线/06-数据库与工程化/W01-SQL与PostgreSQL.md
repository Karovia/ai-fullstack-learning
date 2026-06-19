---
chapter: 06
week: 01
title: SQL 与 PostgreSQL
type: weekly-tutorial
prerequisites: [05-W06]
estimated_hours: 22
difficulty: ★★★★☆
tags: [database, sql, postgresql, psql, joins, jsonb, window-function]
created: 2026-06-19
---

# 第 06 章 · 第 01 周 · SQL 与 PostgreSQL

## 一、为什么这周很重要

前 5 章你写的接口，所有数据都存在内存里或者 JSON 文件里，重启就没。**真实项目离不开数据库**：用户、订单、文章、评论，都得有个稳的地方放。这一周把 SQL 这门"40 多年没过时的语言"学透，后面 5 周才有根基。

---

## 二、数据库到底是什么

**类比**：把数据库想成"加了安全员的超大版 Excel"。

| 普通 Excel | 数据库 |
|---|---|
| 双击打开，谁都能改 | 必须用账号密码连 |
| 改坏了 Ctrl+Z | 事务 + 备份 + 日志 |
| 几万行就卡 | 几亿行还能毫秒查 |
| 不能多人同时改 | 锁机制保证并发安全 |
| 没有"关系" | 表与表之间用外键关联 |

**关系型 vs NoSQL**：

| 维度 | 关系型 (PostgreSQL/MySQL) | NoSQL (MongoDB/Redis) |
|---|---|---|
| 数据模型 | 表 + 行 + 列，有 schema | 文档 / 键值 / 列族 / 图 |
| 一致性 | 强一致，事务 ACID | 大多最终一致 |
| 扩展 | 垂直为主，分库分表麻烦 | 天然横向扩展 |
| 查询 | SQL，表达力极强 | 各家自定，复杂查询难 |
| 适合 | 业务系统、报表、金融 | 日志、缓存、海量 KV |

**结论**：90% 的项目首选 PostgreSQL，必要时再加 Redis 缓存、Elasticsearch 搜索、MongoDB 存 schemaless 数据。

---

## 三、安装 PostgreSQL（Docker 方式）

```bash
docker run -d --name pg \
  -e POSTGRES_PASSWORD=devpass \
  -e POSTGRES_USER=dev \
  -e POSTGRES_DB=playground \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16
```

进容器执行 psql：

```bash
docker exec -it pg psql -U dev -d playground
```

**psql 必会命令**：

| 命令 | 作用 |
|---|---|
| `\l` | 列出所有库 |
| `\c playground` | 切到某库 |
| `\dt` | 列出表 |
| `\d users` | 看表结构 |
| `\df` | 列函数 |
| `\du` | 列用户 |
| `\timing on` | 显示耗时 |
| `\x` | 切换横竖显示 |
| `\q` | 退出 |

**图形客户端**：DBeaver（免费，全能）、TablePlus（颜值高，付费）、pgAdmin（官方）。

---

## 四、数据类型速查

| 类型 | 说明 | 示例 |
|---|---|---|
| `integer` / `int` | 32 位整数 | `42` |
| `bigint` | 64 位 | 大流水号 |
| `numeric(10,2)` | 精确小数（钱必须用） | `199.99` |
| `real` / `double precision` | 浮点（有精度问题） | 科学计算 |
| `text` | 不限长度字符串 | 文章正文 |
| `varchar(n)` | 限长字符串 | 用户名 |
| `boolean` | true / false | 是否激活 |
| `date` | 年月日 | `2026-06-19` |
| `timestamp` / `timestamptz` | 时间戳，**带时区版本**优先 | `now()` |
| `uuid` | 全局唯一 ID | `gen_random_uuid()` |
| `jsonb` | 二进制 JSON，可索引 | `{"a":1}` |
| `text[]` | 数组 | `ARRAY['a','b']` |

**踩坑**：
- 钱**永远不用 float**，用 `numeric`
- 时间统一存 `timestamptz`，应用层显示再转时区
- 字符串别写死 `varchar(20)`，业务一改就崩，用 `text` 配 CHECK 约束

---

## 五、DDL：建表改表

```sql
-- 建表
CREATE TABLE users (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  email       text NOT NULL UNIQUE,
  username    text NOT NULL UNIQUE,
  password    text NOT NULL,
  age         integer CHECK (age >= 0 AND age < 150),
  is_active   boolean NOT NULL DEFAULT true,
  metadata    jsonb NOT NULL DEFAULT '{}'::jsonb,
  tags        text[] NOT NULL DEFAULT '{}',
  created_at  timestamptz NOT NULL DEFAULT now()
);

-- 加列
ALTER TABLE users ADD COLUMN bio text;

-- 改列类型
ALTER TABLE users ALTER COLUMN bio TYPE varchar(500);

-- 加约束
ALTER TABLE users ADD CONSTRAINT email_lower CHECK (email = lower(email));

-- 删表
DROP TABLE IF EXISTS users CASCADE;
```

**主外键示例**：

```sql
CREATE TABLE posts (
  id         bigserial PRIMARY KEY,
  author_id  uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title      text NOT NULL,
  body       text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);
```

`ON DELETE CASCADE` 表示父记录删除时，子记录跟着删；可改为 `RESTRICT`（禁止删）或 `SET NULL`。

---

## 六、DML：增删改

```sql
-- 插入
INSERT INTO users (email, username, password)
VALUES ('a@x.com', 'alice', 'hashed');

-- 一次插多行
INSERT INTO users (email, username, password) VALUES
  ('b@x.com', 'bob',   'h'),
  ('c@x.com', 'carol', 'h')
RETURNING id, username;   -- 返回插入后的字段

-- 更新
UPDATE users SET is_active = false WHERE email = 'a@x.com';

-- 删除
DELETE FROM users WHERE created_at < now() - interval '1 year';

-- UPSERT（关键技能）
INSERT INTO users (email, username, password)
VALUES ('a@x.com', 'alice', 'h')
ON CONFLICT (email) DO UPDATE
  SET username = EXCLUDED.username,
      password = EXCLUDED.password;
```

`ON CONFLICT DO NOTHING` 是另一个常用变体，适合"有就跳过"。

---

## 七、DQL：查询

```sql
SELECT id, username, created_at
FROM users
WHERE is_active = true
  AND created_at > now() - interval '7 days'
  AND username ILIKE 'a%'
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;
```

| 关键字 | 作用 |
|---|---|
| `WHERE` | 过滤 |
| `ILIKE` | 不区分大小写模糊匹配 |
| `IN (...)` | 列表 |
| `BETWEEN a AND b` | 范围（含两端） |
| `IS NULL` | 空判断（`= NULL` 是错的） |
| `ORDER BY x DESC NULLS LAST` | 排序 |
| `LIMIT / OFFSET` | 分页 |

---

## 八、聚合与分组

```sql
SELECT
  date_trunc('day', created_at) AS day,
  count(*)            AS total,
  count(DISTINCT author_id) AS authors,
  avg(length(body))   AS avg_len
FROM posts
WHERE created_at > now() - interval '30 days'
GROUP BY day
HAVING count(*) > 10
ORDER BY day;
```

**WHERE vs HAVING**：WHERE 在分组前过滤行，HAVING 在分组后过滤组。

---

## 九、JOIN：多表的灵魂

```
A = {1,2,3}     B = {2,3,4}

INNER JOIN  ：交集 → {2,3}
LEFT  JOIN  ：A 全留，B 没的填 NULL → {1,2,3} + B 填空
RIGHT JOIN  ：B 全留 → {2,3,4} + A 填空
FULL  JOIN  ：并集 → {1,2,3,4}
CROSS JOIN  ：笛卡儿积 → 9 行
SELF  JOIN  ：自己 join 自己（树形结构、上下级）
```

```sql
-- INNER：只要双方都有
SELECT u.username, p.title
FROM users u
JOIN posts p ON p.author_id = u.id;

-- LEFT：用户即便没文章也保留
SELECT u.username, count(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON p.author_id = u.id
GROUP BY u.id;

-- SELF JOIN：员工 + 上级
SELECT e.name, m.name AS manager
FROM employees e
LEFT JOIN employees m ON m.id = e.manager_id;
```

---

## 十、子查询、CTE、窗口函数

**子查询 + EXISTS**：

```sql
SELECT username FROM users u
WHERE EXISTS (
  SELECT 1 FROM posts p
  WHERE p.author_id = u.id AND p.created_at > now() - interval '1 day'
);
```

**CTE（推荐替代多层子查询，可读性飞起）**：

```sql
WITH active_users AS (
  SELECT id FROM users WHERE is_active
),
recent_posts AS (
  SELECT * FROM posts WHERE created_at > now() - interval '7 days'
)
SELECT u.id, count(p.id)
FROM active_users u
LEFT JOIN recent_posts p ON p.author_id = u.id
GROUP BY u.id;
```

**递归 CTE**（树/图结构）：

```sql
WITH RECURSIVE tree AS (
  SELECT id, parent_id, name, 1 AS depth FROM categories WHERE parent_id IS NULL
  UNION ALL
  SELECT c.id, c.parent_id, c.name, t.depth + 1
  FROM categories c JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree ORDER BY depth, name;
```

**窗口函数**（"分组但不合并行"）：

```sql
SELECT
  username,
  score,
  ROW_NUMBER()  OVER (PARTITION BY class ORDER BY score DESC) AS rn,
  RANK()        OVER (PARTITION BY class ORDER BY score DESC) AS rk,
  DENSE_RANK()  OVER (PARTITION BY class ORDER BY score DESC) AS drk,
  LAG(score, 1) OVER (PARTITION BY class ORDER BY date)       AS prev,
  LEAD(score,1) OVER (PARTITION BY class ORDER BY date)       AS next,
  SUM(score)    OVER (PARTITION BY class)                     AS class_total
FROM exam;
```

| 函数 | 区别 |
|---|---|
| ROW_NUMBER | 1,2,3,4 永不重 |
| RANK | 1,2,2,4 同分跳号 |
| DENSE_RANK | 1,2,2,3 同分不跳号 |
| LAG/LEAD | 取前/后 N 行 |
| SUM/AVG OVER | 累计/分组聚合保留行 |

---

## 十一、jsonb 与数组

```sql
-- 包含
SELECT * FROM users WHERE metadata @> '{"vip": true}';

-- 取值
SELECT metadata->'addr'->>'city' FROM users;

-- 路径查询
SELECT jsonb_path_query(metadata, '$.tags[*]') FROM users;

-- 数组
SELECT * FROM users WHERE 'admin' = ANY(tags);
SELECT * FROM users WHERE tags && ARRAY['admin','vip'];   -- 交集非空
```

---

## 十二、视图与物化视图

```sql
-- 视图：每次查实时计算
CREATE VIEW v_active_user_post AS
SELECT u.id, u.username, count(p.id) AS posts
FROM users u LEFT JOIN posts p ON p.author_id = u.id
WHERE u.is_active GROUP BY u.id;

-- 物化视图：结果存盘，要手动 REFRESH，适合慢报表
CREATE MATERIALIZED VIEW mv_daily_stats AS
SELECT date_trunc('day', created_at) AS d, count(*) FROM posts GROUP BY 1;

REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_stats;
```

---

## 十三、本周练习

1. **LeetCode SQL 50 题**全做（Easy → Medium）
2. **SQLZoo** 全部 9 关
3. 设计博客系统 schema：`users / posts / comments / tags / post_tags`，写出建表 SQL，造 100 条假数据，写 10 条业务 SQL（最热文章、近 7 天活跃作者…）
4. 用 jsonb 设计一个"动态表单提交"表，写 5 条查询

## 十四、Checklist

- [ ] 能在 5 分钟内用 Docker 起一个 PG，并用 psql 连上
- [ ] 默写 8 种数据类型适用场景
- [ ] 写得出 4 种 JOIN 并讲清差异
- [ ] CTE 替换嵌套子查询无压力
- [ ] 窗口函数能解决"组内 Top N"
- [ ] jsonb 至少会 3 种查询写法
- [ ] LeetCode SQL 通过 ≥ 30 道

下一周进入索引、事务、锁——把"会写 SQL"升级成"会写**快**的 SQL"。
