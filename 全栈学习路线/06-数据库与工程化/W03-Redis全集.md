---
chapter: 06
week: 03
title: Redis 全集
type: weekly-tutorial
prerequisites: [06-W02]
estimated_hours: 20
difficulty: ★★★★☆
tags: [redis, cache, distributed-lock, rate-limit, leaderboard, queue]
created: 2026-06-19
---

# 第 06 章 · 第 03 周 · Redis 全集

## 一、Redis 的位置

**类比**：PostgreSQL 是仓库，又大又结构化但拿东西要走过去；Redis 是你桌上的便利贴，**速度极快但容量小**。99% 的高并发场景缺的不是数据库，是**缓存层**。

Redis 还能干别的：分布式锁、限流、排行榜、消息队列、地理位置、签到、计数器…… 是后端工程师的瑞士军刀。

---

## 二、安装

```bash
docker run -d --name redis -p 6379:6379 \
  -v redisdata:/data \
  redis:7-alpine redis-server --appendonly yes
```

```bash
docker exec -it redis redis-cli
127.0.0.1:6379> PING
PONG
```

带密码起：`redis-server --requirepass devpass`，连：`redis-cli -a devpass`。

---

## 三、5 种基本数据类型

### 1. String（万能选手）

```bash
SET name "alice"
GET name
SET counter 0
INCR counter           # 原子 +1
INCRBY counter 5
SETEX session 60 "tk"  # 60 秒过期
SET key val NX EX 30   # 不存在才设置 + 30s 过期（分布式锁基础）
```

**用途**：缓存对象（JSON 序列化存）、计数器、分布式锁、限流。

### 2. List（双端链表）

```bash
LPUSH q "task1" "task2"   # 左推
RPUSH q "task3"            # 右推
LRANGE q 0 -1             # 查看全部
RPOP q                     # 右弹
BLPOP q 5                  # 阻塞弹（5 秒）
LLEN q
```

**用途**：消息队列（粗糙版）、最近 N 条记录（LPUSH + LTRIM）、时间线。

### 3. Hash（对象）

```bash
HSET user:1 name alice age 30 city beijing
HGET user:1 name
HGETALL user:1
HINCRBY user:1 age 1
```

**用途**：用对象式缓存代替"整个 JSON 反复序列化"。

### 4. Set（无序集合）

```bash
SADD tags "go" "rust" "python"
SISMEMBER tags "go"
SMEMBERS tags
SINTER tags1 tags2     # 交集（共同好友）
SUNION  tags1 tags2     # 并集
SDIFF   tags1 tags2     # 差集
```

**用途**：标签、共同好友、去重。

### 5. Sorted Set / ZSET（排行榜杀器）

```bash
ZADD lb 100 alice 90 bob 95 carol
ZREVRANGE lb 0 9 WITHSCORES   # Top 10
ZRANK   lb alice               # 名次（升序）
ZREVRANK lb alice              # 名次（降序）
ZINCRBY lb 5 alice
ZRANGEBYSCORE lb 80 100
```

**用途**：排行榜、延迟队列（用时间戳作 score）、按时间窗的最近 N 条。

---

## 四、高级类型

| 类型 | 命令前缀 | 典型场景 |
|---|---|---|
| **Stream** | XADD/XREAD/XGROUP | Kafka 风格消息流，支持消费组 |
| **HyperLogLog** | PFADD/PFCOUNT | 海量 UV 估算（误差 0.81%，内存 12KB） |
| **Bitmap** | SETBIT/GETBIT | 签到、活跃用户位图（1 亿用户只 12MB） |
| **Geo** | GEOADD/GEORADIUS | "附近的人"、外卖派单 |
| **Bitfield** | BITFIELD | 多个紧凑整数共用 string |

```bash
# 签到 bitmap
SETBIT signin:202606:alice 18 1     # 6/19 签到
GETBIT signin:202606:alice 18
BITCOUNT signin:202606:alice         # 当月签到天数

# Geo
GEOADD bj 116.4 39.9 "tiananmen"
GEORADIUS bj 116.4 39.9 5 km WITHCOORD WITHDIST
```

---

## 五、过期与淘汰

```bash
EXPIRE key 60
TTL key
PERSIST key
```

**淘汰策略**（内存满时怎么挑出去）：

| 策略 | 说明 |
|---|---|
| `noeviction` | 内存满直接报错（默认） |
| `allkeys-lru` | 所有 key 中 LRU |
| `allkeys-lfu` | 访问频率低的优先逐出（推荐） |
| `volatile-lru/lfu` | 仅设了过期的 key |
| `volatile-ttl` | 快过期的 key 优先 |
| `allkeys-random` / `volatile-random` | 随机 |

`maxmemory 2gb` + `maxmemory-policy allkeys-lfu` 是常用缓存配置。

---

## 六、持久化

| 方式 | 原理 | 优点 | 缺点 |
|---|---|---|---|
| **RDB** | 周期性快照 dump.rdb | 文件小、恢复快 | 丢数据可能多 |
| **AOF** | 追加每条写命令到日志 | 几乎不丢 | 文件大、恢复慢 |
| **混合**（推荐） | RDB + 增量 AOF | 兼顾两者 | 7.0+ 默认 |

`appendonly yes` + `appendfsync everysec`：每秒 fsync，最多丢 1 秒数据。

---

## 七、高可用

- **主从复制**：`REPLICAOF master_ip 6379`，从可读不可写
- **Sentinel**（哨兵）：监控 + 自动故障切换，3 个哨兵起步
- **Cluster**：原生分片，16384 个 slot 分到 N 个节点，支持横向扩展

小项目主从够，中型加哨兵，大型上 Cluster。

---

## 八、事务、乐观锁、Lua

### MULTI / EXEC

```bash
MULTI
INCR counter
INCR counter
EXEC
```

注意：Redis 事务**不是 ACID**！EXEC 期间命令打包发送但**不能回滚**，不能中途看结果再决定。

### WATCH 乐观锁

```bash
WATCH balance
GET balance        # 假设是 100
MULTI
DECRBY balance 50
EXEC               # 如果中间 balance 被别人改了，EXEC 返回 nil
```

### Lua（原子执行，工程首选）

```lua
-- decr_if_enough.lua
local cur = tonumber(redis.call('GET', KEYS[1]))
if cur and cur >= tonumber(ARGV[1]) then
  redis.call('DECRBY', KEYS[1], ARGV[1])
  return 1
end
return 0
```

```bash
redis-cli --eval decr_if_enough.lua balance , 50
```

**原子保证**：脚本期间 Redis 单线程不会插其他命令。

---

## 九、Pub/Sub vs Stream

- **Pub/Sub**：广播，订阅者掉线消息丢，无持久化
- **Stream**：消息持久化、消费组、ACK，类似 Kafka 简化版

新代码优先用 Stream。

---

## 十、客户端

**Python (redis-py)**：

```python
import redis.asyncio as redis

r = redis.Redis(host="localhost", port=6379, decode_responses=True)
await r.set("k", "v", ex=60)
v = await r.get("k")
```

**Node (ioredis)**：

```ts
import Redis from "ioredis";
const r = new Redis();
await r.set("k", "v", "EX", 60);
const v = await r.get("k");
```

---

## 十一、应用模式（重点）

### 1. 分布式锁（SETNX + 过期 + Lua 释放）

```python
import uuid

LOCK_KEY = "lock:order:1001"

async def acquire(timeout=10):
    token = str(uuid.uuid4())
    ok = await r.set(LOCK_KEY, token, nx=True, ex=timeout)
    return token if ok else None

RELEASE = """
if redis.call('GET', KEYS[1]) == ARGV[1] then
  return redis.call('DEL', KEYS[1])
else
  return 0
end
"""

async def release(token):
    await r.eval(RELEASE, 1, LOCK_KEY, token)
```

**为什么 Lua**：避免"我以为还是我的锁，但 GET 后 DEL 之前别人拿了"。

**生产建议**：直接用 **Redlock** 库或 [redis-py-lock]，别从零造。

### 2. 限流（令牌桶 / 滑动窗口 Lua）

```lua
-- KEYS[1] = key, ARGV[1] = capacity, ARGV[2] = rate(per sec), ARGV[3] = now
local data = redis.call('HMGET', KEYS[1], 'tokens', 'ts')
local tokens = tonumber(data[1]) or tonumber(ARGV[1])
local last = tonumber(data[2]) or tonumber(ARGV[3])
local delta = (tonumber(ARGV[3]) - last) * tonumber(ARGV[2])
tokens = math.min(tonumber(ARGV[1]), tokens + delta)
if tokens >= 1 then
  tokens = tokens - 1
  redis.call('HMSET', KEYS[1], 'tokens', tokens, 'ts', ARGV[3])
  redis.call('EXPIRE', KEYS[1], 60)
  return 1
else
  return 0
end
```

### 3. 排行榜

```python
await r.zincrby("lb:202606", 1, "alice")     # 加分
top10 = await r.zrevrange("lb:202606", 0, 9, withscores=True)
my_rank = await r.zrevrank("lb:202606", "alice")
```

### 4. 延迟队列

```python
# 投递：score = 执行时间戳
await r.zadd("delayq", {payload: time.time() + 60})

# 消费 worker（轮询）
while True:
    now = time.time()
    items = await r.zrangebyscore("delayq", 0, now, start=0, num=10)
    for it in items:
        # 用 ZREM 抢占（多 worker 防重）
        if await r.zrem("delayq", it):
            await handle(it)
    await asyncio.sleep(1)
```

### 5. 短链

```python
import hashlib, base64

def shorten(url):
    h = hashlib.md5(url.encode()).digest()[:6]
    code = base64.urlsafe_b64encode(h).decode().rstrip("=")
    await r.set(f"short:{code}", url)
    return code
```

---

## 十二、本周实战

写一个 `redis-toolbox` 项目，包含 4 个子模块：

1. **限流器**：FastAPI 中间件，令牌桶 Lua，QPS 限 10
2. **分布式锁**：上面那段，写单测验证 100 并发只有 1 个拿到
3. **排行榜**：CLI 程序，加分 + 查询 Top10 + 查我名次
4. **延迟队列**：`enqueue(payload, delay_sec)` + worker，验证 60 秒后消费到

加 README，附性能测试结果（locust 跑限流接口）。

---

## 十三、Checklist

- [ ] 5 种基本类型每种说出 2 个真实业务用法
- [ ] 写得出"加锁、释放锁"完整 Lua
- [ ] 知道 Pub/Sub 和 Stream 何时选哪个
- [ ] RDB / AOF / 混合 区别清晰
- [ ] 哨兵和 Cluster 区别讲得清
- [ ] 限流、排行榜、延迟队列三个 demo 跑通

下一周回到关系型数据库，但用 **ORM** 让代码层不再写裸 SQL。
