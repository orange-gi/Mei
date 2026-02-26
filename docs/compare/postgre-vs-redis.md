# PostgreSQL 替代 Redis 实战指南

## 文章概述

本文记录了一位开发者在实际项目中将 Redis 完全替换为 PostgreSQL 的完整经历。作者原本采用典型的 Web 应用技术栈：使用 PostgreSQL 存储持久化数据，使用 Redis 处理缓存、发布订阅和后台任务。在发现 PostgreSQL 能够实现 Redis 的所有功能后，作者进行了全面迁移，并详细记录了迁移过程中的技术实现、性能对比、成本分析和最终结论。

文章提供了丰富的代码示例、性能基准测试数据、分阶段迁移策略以及决策矩阵，旨在帮助其他开发者判断是否应该在自己的项目中采用类似的方案。

---

## 一、为什么 PostgreSQL 可以替代 Redis

### 1.1 成本效益分析

从经济角度来看，单独维护 Redis 基础设施的成本相当可观。作者的 Redis 部署在 AWS ElastiCache 上，2GB 内存的配置每月花费约 45 美元，而如果需要扩展到 5GB，成本将飙升至每月 110 美元。相比之下，作者已经为 PostgreSQL 支付了每月 50 美元的 RDS 费用（包含 20GB 存储），额外的 5GB 数据存储仅需额外支付 0.50 美元。通过消除独立的 Redis 基础设施，作者每月可节省约 100 美元的费用。

这一成本差异在规模扩大时会更加显著。Redis 作为内存数据库，其存储成本与数据量成正比；而 PostgreSQL 作为磁盘存储的数据库，增量存储成本极低。对于数据量持续增长的应用来说，这种成本优势会越来越明显。

### 1.2 运维复杂度降低

管理两个独立的数据库系统会显著增加运维负担。使用 Redis 时，作者面临诸多不确定性：备份策略应该如何选择（RDB 快照、AOF 日志，还是两者结合）？监控配置应该如何设置？故障转移应该采用 Redis Sentinel 还是 Cluster 方案？这些问题都需要专业知识来解决。

相比之下，PostgreSQL 的备份、监控和故障转移流程已经非常成熟，有完善的文档和工具支持。通过将所有功能统一到 PostgreSQL，作者将运维复杂度降低了约 50%，只需要关注一个数据库系统的健康状况和性能优化。

### 1.3 数据一致性保证

在使用分离的缓存和数据库层时，保持数据一致性是一个持续的挑战。典型的操作流程包括：先更新数据库，然后使缓存失效。但这个过程中存在多个可能的失败点：Redis 可能不可用，使缓存失效的调用可能失败，网络问题可能中断操作。一旦发生任何问题，缓存和数据库就会陷入不一致状态。

PostgreSQL 通过原子事务彻底解决了这个问题。数据库操作、缓存更新和通知发布可以在同一个事务中执行，要么全部成功，要么全部回滚。这种事务性保证消除了分布式系统中数据不一致的根本原因，大大简化了应用逻辑并提高了系统可靠性。

---

## 二、PostgreSQL 替代 Redis 的核心技术方案

### 2.1 缓存功能：UNLOGGED 表

PostgreSQL 的 UNLOGGED 表是实现缓存功能的理想选择。UNLOGGED 表在写入时会跳过预写日志（WAL），这使得写入操作更快，同时在正常操作中仍然保持数据耐久性。唯一的特点是它们不会在数据库崩溃后恢复，但对于缓存数据来说，这完全是可以接受的，因为缓存数据可以随时重新生成。

作者创建了一个专门的缓存表结构，包含 key（文本主键）、value（JSONB 类型存储实际数据）和 expires_at（过期时间戳）三个字段。通过在查询中添加 `expires_at > NOW()` 条件来实现 TTL 功能。性能测试显示，Redis 的 SET 操作延迟为 0.05 毫秒，而 PostgreSQL 的 UNLOGGED INSERT 延迟为 0.08 毫秒，对于缓存场景来说这个差异完全可以接受。

```sql
CREATE UNLOGGED TABLE cache (
  key TEXT PRIMARY KEY,
  value JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_cache_expires ON cache(expires_at);
```

### 2.2 发布订阅：LISTEN/NOTIFY

PostgreSQL 内置的发布订阅功能是许多开发者不了解的隐藏特性。通过 LISTEN 和 NOTIFY 命令，开发者可以轻松实现进程间通信。发布者使用 NOTIFY 命令发送消息，订阅者通过 LISTEN 命令监听特定通道并接收异步通知。

性能对比显示，Redis 的发布订阅延迟为 1-2 毫秒，而 PostgreSQL 为 2-5 毫秒。虽然 PostgreSQL 稍慢，但其独特优势在于可以在事务中发送通知，并且可以与查询结合实现更复杂的模式。例如，通过数据库触发器，可以在插入新数据时自动触发通知，实现真正的数据驱动通知。

```sql
-- 发布通知
NOTIFY notifications, '{"userId": 123, "msg": "Hello"}';

-- 监听通知
LISTEN notifications;
```

### 2.3 任务队列：SKIP LOCKED

PostgreSQL 的 SKIP LOCKED 特性使其成为一个强大的无锁任务队列。当多个工作进程同时尝试获取任务时，SKIP LOCKED 确保每个任务只分配给一个工作进程，而其他工作进程会跳过已被锁定的行继续处理其他任务。这种模式提供了并发处理能力、自动防止重复执行、以及在 worker 崩溃时任务自动重新可用的特性。

性能测试表明，Redis 的 BRPOP 操作延迟为 0.1 毫秒，PostgreSQL 的 SKIP LOCKED 查询延迟为 0.3 毫秒，对于大多数生产工作负载来说，这个差异可以忽略不计。

```sql
WITH next_job AS (
  SELECT id FROM jobs
  WHERE queue = $1
    AND attempts < max_attempts
    AND scheduled_at <= NOW()
  ORDER BY scheduled_at
  LIMIT 1
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs
SET attempts = attempts + 1
FROM next_job
WHERE jobs.id = next_job.id
RETURNING *;
```

### 2.4 速率限制

PostgreSQL 提供了多种实现速率限制的方法。简单的速率限制使用计数器表加时间戳，而更复杂的实现则记录每个请求以支持时间窗口查询。PostgreSQL 方法的优势在于当速率限制逻辑复杂或需要与其他业务逻辑在同一个事务中执行时特别有用。

### 2.5 会话存储：JSONB

使用 PostgreSQL 的 JSONB 列存储会话数据不仅提供了 Redis 的所有功能，还能进行内部查询。例如，可以查找特定用户的所有会话，或根据存储在 JSON 中的角色筛选会话。这种查询能力在使用 Redis 的字符串存储时是不可能实现的。

```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  data JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

-- 查找特定用户的所有会话
SELECT * FROM sessions
WHERE data->>'userId' = '123';
```

---

## 三、性能基准测试对比

### 3.1 独立操作性能

作者在 AWS RDS db.t3.medium 实例（2 vCPU，4GB RAM）上使用包含 100 万条缓存记录和 1 万个会话的生产数据集进行了性能基准测试。测试结果显示，PostgreSQL 在所有操作上都比 Redis 慢，但所有操作仍然保持在 1 毫秒以下。

| 操作类型 | Redis | PostgreSQL | 差异 |
|---------|-------|------------|------|
| 缓存写入 | 0.05ms | 0.08ms | 慢 60% |
| 缓存读取 | 0.04ms | 0.06ms | 慢 50% |
| 发布订阅 | 1.2ms | 3.1ms | 慢 158% |
| 队列推送 | 0.08ms | 0.15ms | 慢 87% |
| 队列弹出 | 0.12ms | 0.31ms | 慢 158% |

### 3.2 组合操作性能

然而，作者发现当多个操作需要组合执行时，PostgreSQL 由于消除了网络往返，实际性能可能更优。在一个典型的场景中——插入数据、使缓存失效、通知订阅者——使用 Redis 需要约 4 毫秒（2 毫秒数据库操作 + 1 毫秒缓存删除网络往返 + 1 毫秒发布网络往返）。而 PostgreSQL 在单一数据库连接上执行所有操作只需约 2.2 毫秒，消除了网络开销。

---

## 四、何时保留 Redis

虽然 PostgreSQL 可以替代 Redis，但作者强调这种替代并非普遍适用。以下情况建议保留 Redis：

**极端性能需求**：Redis 在单实例上可以处理 10 万次以上的操作每秒，而 PostgreSQL 通常处理能力在 1 万到 5 万次每秒。对于每秒处理数百万次缓存读取的应用，Redis 仍是更好的选择。

**Redis 特有数据结构**：Redis 提供了许多 PostgreSQL 难以高效实现的数据结构，包括用于排行榜的有序集合、用于唯一计数估计的 HyperLogLog、用于地理位置查询的地理空间索引，以及用于高级发布订阅的 Streams。

**独立缓存层架构**：在微服务架构中，如果需要独立缓存层或多个服务共享缓存，Redis 的专门设计具有优势。

**专业运维团队**：拥有专业基础设施团队的组织可以管理 Redis 的运维复杂性并从中获取最大性能。

---

## 五、分阶段迁移策略

作者建议采用渐进式迁移策略，整个过程约需四周时间：

**第一阶段（第 1 周）——双写阶段**：同时向 Redis 和 PostgreSQL 写入，继续从 Redis 读取。监控命中率、延迟，建立性能基线。

**第二阶段（第 2 周）——读切换阶段**：优先从 PostgreSQL 读取，如果数据不存在则回退到 Redis。监控错误率和性能影响。

**第三阶段（第 3 周）——写切换阶段**：只向 PostgreSQL 写入，移除写入路径中的 Redis。监控系统是否正常工作。

**第四阶段（第 4 周）——下线阶段**：关闭 Redis，监控系统是否出现问题。如果一切正常，则迁移成功。

---

## 六、三个月后的实际效果

### 6.1 获得收益

- 每月节省 100 美元（无需 ElastiCache）
- 备份复杂度降低 50%
- 减少一个需要监控的服务
- 部署依赖减少，简化部署流程

### 6.2 付出代价

- 缓存操作增加约 0.5 毫秒延迟
- 无法使用 Redis 的特殊数据结构（但项目不需要）

### 6.3 最终结论

作者表示，如果重新选择，仍然会进行这次迁移，因为对于他们的使用场景来说收益远大于代价。但他们不会无差别地推荐这种替代方案。

---

## 七、决策矩阵

### 应该用 PostgreSQL 替代 Redis 的情况

- 主要使用 Redis 进行简单缓存或会话存储
- 缓存命中率低于 95%（写入操作较多）
- 需要事务性数据一致性保证
- 可以接受 0.1-1 毫秒的操作延迟增加
- 是缺乏运维资源的小型团队

### 应该保留 Redis 的情况

- 需要每秒 10 万次以上的操作吞吐量
- 使用 Redis 特有数据结构（有序集合等）
- 拥有专业运维团队
- 需要亚毫秒级延迟
- 需要地理复制能力

---

## 八、总结

PostgreSQL 确实可以在许多场景下替代 Redis，为开发者提供更低的运维成本、简化的基础设施和更强的数据一致性保证。关键在于理解两者的特点，根据具体业务需求做出合理选择。对于中小型应用、简单的缓存需求、追求简化运维的团队，PostgreSQL 是一个值得考虑的方案。对于追求极致性能、需要 Redis 特有数据结构、或有专业团队支撑的场景，保留 Redis 仍然是明智的选择。

技术选型没有放之四海而皆准的最佳方案，只有最适合特定场景的方案。希望本文的分析能帮助读者在 PostgreSQL 和 Redis 之间做出更明智的决策。


======================================================================================================================================
原文：https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942
I Replaced Redis with PostgreSQL (And It's Faster)
#
postgres
#
redis
#
database
#
performance
I had a typical web app stack:

PostgreSQL for persistent data
Redis for caching, pub/sub, and background jobs
Two databases. Two things to manage. Two points of failure.

Then I realized: PostgreSQL can do everything Redis does.

I ripped out Redis entirely. Here's what happened.

The Setup: What I Was Using Redis For
Before the change, Redis handled three things:

1. Caching (70% of usage)
// Cache API responses
await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
2. Pub/Sub (20% of usage)
// Real-time notifications
redis.publish('notifications', JSON.stringify({ userId, message }));
3. Background Job Queue (10% of usage)
// Using Bull/BullMQ
queue.add('send-email', { to, subject, body });
The pain points:

Two databases to backup
Redis uses RAM (expensive at scale)
Redis persistence is... complicated
Network hop between Postgres and Redis
Why I Considered Replacing Redis
Reason #1: Cost
My Redis setup:

AWS ElastiCache: $45/month (2GB)
Growing to 5GB would cost $110/month
PostgreSQL:

Already paying for RDS: $50/month (20GB storage)
Adding 5GB of data: $0.50/month
Potential savings: ~$100/month

Reason #2: Operational Complexity
With Redis:

Postgres backup ✅
Redis backup ❓ (RDB? AOF? Both?)
Postgres monitoring ✅
Redis monitoring ❓
Postgres failover ✅
Redis Sentinel/Cluster ❓
Without Redis:

Postgres backup ✅
Postgres monitoring ✅
Postgres failover ✅
One less moving part.

Reason #3: Data Consistency
The classic problem:

// Update database
await db.query('UPDATE users SET name = $1 WHERE id = $2', [name, id]);

// Invalidate cache
await redis.del(`user:${id}`);

// ⚠️ What if Redis is down?
// ⚠️ What if this fails?
// Now cache and DB are out of sync
With everything in Postgres: transactions solve this.

PostgreSQL Feature #1: Caching with UNLOGGED Tables
Redis:

await redis.set('session:abc123', JSON.stringify(sessionData), 'EX', 3600);
PostgreSQL:

CREATE UNLOGGED TABLE cache (
  key TEXT PRIMARY KEY,
  value JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_cache_expires ON cache(expires_at);
Insert:

INSERT INTO cache (key, value, expires_at)
VALUES ($1, $2, NOW() + INTERVAL '1 hour')
ON CONFLICT (key) DO UPDATE
  SET value = EXCLUDED.value,
      expires_at = EXCLUDED.expires_at;
Read:

SELECT value FROM cache
WHERE key = $1 AND expires_at > NOW();
Cleanup (run periodically):

DELETE FROM cache WHERE expires_at < NOW();
What is UNLOGGED?
UNLOGGED tables:

Skip the Write-Ahead Log (WAL)
Much faster writes
Don't survive crashes (perfect for cache!)
Performance:

Redis SET: 0.05ms
Postgres UNLOGGED INSERT: 0.08ms
Close enough for caching.

PostgreSQL Feature #2: Pub/Sub with LISTEN/NOTIFY
This is where it gets interesting.

PostgreSQL has native pub/sub that most developers don't know about.

Redis Pub/Sub
// Publisher
redis.publish('notifications', JSON.stringify({ userId: 123, msg: 'Hello' }));

// Subscriber
redis.subscribe('notifications');
redis.on('message', (channel, message) => {
  console.log(message);
});
PostgreSQL Pub/Sub
-- Publisher
NOTIFY notifications, '{"userId": 123, "msg": "Hello"}';
// Subscriber (Node.js with pg)
const client = new Client({ connectionString: process.env.DATABASE_URL });
await client.connect();

await client.query('LISTEN notifications');

client.on('notification', (msg) => {
  const payload = JSON.parse(msg.payload);
  console.log(payload);
});
Performance comparison:

Redis pub/sub latency: 1-2ms
Postgres NOTIFY latency: 2-5ms
Slightly slower, but:

No extra infrastructure
Can use in transactions
Can combine with queries
Real-World Example: Live Tail
In my log management app, I needed real-time log streaming.

With Redis:

// When new log arrives
await db.query('INSERT INTO logs ...');
await redis.publish('logs:new', JSON.stringify(log));

// Frontend listens
redis.subscribe('logs:new');
Problem: Two operations. What if publish fails?

With PostgreSQL:

CREATE FUNCTION notify_new_log() RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_notify('logs_new', row_to_json(NEW)::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_inserted
AFTER INSERT ON logs
FOR EACH ROW EXECUTE FUNCTION notify_new_log();
Now it's atomic. Insert and notify happen together or not at all.

// Frontend (via SSE)
app.get('/logs/stream', async (req, res) => {
  const client = await pool.connect();

  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
  });

  await client.query('LISTEN logs_new');

  client.on('notification', (msg) => {
    res.write(`data: ${msg.payload}\n\n`);
  });
});
Result: Real-time log streaming with zero Redis.

PostgreSQL Feature #3: Job Queues with SKIP LOCKED
Redis (using Bull/BullMQ):

queue.add('send-email', { to, subject, body });

queue.process('send-email', async (job) => {
  await sendEmail(job.data);
});
PostgreSQL:

CREATE TABLE jobs (
  id BIGSERIAL PRIMARY KEY,
  queue TEXT NOT NULL,
  payload JSONB NOT NULL,
  attempts INT DEFAULT 0,
  max_attempts INT DEFAULT 3,
  scheduled_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_jobs_queue ON jobs(queue, scheduled_at) 
WHERE attempts < max_attempts;
Enqueue:

INSERT INTO jobs (queue, payload)
VALUES ('send-email', '{"to": "user@example.com", "subject": "Hi"}');
Worker (dequeue):

WITH next_job AS (
  SELECT id FROM jobs
  WHERE queue = $1
    AND attempts < max_attempts
    AND scheduled_at <= NOW()
  ORDER BY scheduled_at
  LIMIT 1
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs
SET attempts = attempts + 1
FROM next_job
WHERE jobs.id = next_job.id
RETURNING *;
The magic: FOR UPDATE SKIP LOCKED

This makes PostgreSQL a lock-free queue:

Multiple workers can pull jobs concurrently
No job is processed twice
If a worker crashes, job becomes available again
Performance:

Redis BRPOP: 0.1ms
Postgres SKIP LOCKED: 0.3ms
Negligible difference for most workloads.

PostgreSQL Feature #4: Rate Limiting
Redis (classic rate limiter):

const key = `ratelimit:${userId}`;
const count = await redis.incr(key);
if (count === 1) {
  await redis.expire(key, 60); // 60 seconds
}

if (count > 100) {
  throw new Error('Rate limit exceeded');
}
PostgreSQL:

CREATE TABLE rate_limits (
  user_id INT PRIMARY KEY,
  request_count INT DEFAULT 0,
  window_start TIMESTAMPTZ DEFAULT NOW()
);

-- Check and increment
WITH current AS (
  SELECT 
    request_count,
    CASE 
      WHEN window_start < NOW() - INTERVAL '1 minute'
      THEN 1  -- Reset counter
      ELSE request_count + 1
    END AS new_count
  FROM rate_limits
  WHERE user_id = $1
  FOR UPDATE
)
UPDATE rate_limits
SET 
  request_count = (SELECT new_count FROM current),
  window_start = CASE
    WHEN window_start < NOW() - INTERVAL '1 minute'
    THEN NOW()
    ELSE window_start
  END
WHERE user_id = $1
RETURNING request_count;
Or simpler with a window function:

CREATE TABLE api_requests (
  user_id INT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Check rate limit
SELECT COUNT(*) FROM api_requests
WHERE user_id = $1
  AND created_at > NOW() - INTERVAL '1 minute';

-- If under limit, insert
INSERT INTO api_requests (user_id) VALUES ($1);

-- Cleanup old requests periodically
DELETE FROM api_requests WHERE created_at < NOW() - INTERVAL '5 minutes';
When Postgres is better:

Need to rate limit based on complex logic (not just counts)
Want rate limit data in same transaction as business logic
When Redis is better:

Need sub-millisecond rate limiting
Extremely high throughput (millions of requests/sec)
PostgreSQL Feature #5: Sessions with JSONB
Redis:

await redis.set(`session:${sessionId}`, JSON.stringify(sessionData), 'EX', 86400);
PostgreSQL:

CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  data JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_sessions_expires ON sessions(expires_at);

-- Insert/Update
INSERT INTO sessions (id, data, expires_at)
VALUES ($1, $2, NOW() + INTERVAL '24 hours')
ON CONFLICT (id) DO UPDATE
  SET data = EXCLUDED.data,
      expires_at = EXCLUDED.expires_at;

-- Read
SELECT data FROM sessions
WHERE id = $1 AND expires_at > NOW();
Bonus: JSONB Operators

You can query inside the session:

-- Find all sessions for a specific user
SELECT * FROM sessions
WHERE data->>'userId' = '123';

-- Find sessions with specific role
SELECT * FROM sessions
WHERE data->'user'->>'role' = 'admin';
You can't do this with Redis!

Real-World Benchmarks
I ran benchmarks on my production dataset:

Test Setup
Hardware: AWS RDS db.t3.medium (2 vCPU, 4GB RAM)
Dataset: 1 million cache entries, 10k sessions
Tool: pgbench (custom scripts)
Results
Operation	Redis	PostgreSQL	Difference
Cache SET	0.05ms	0.08ms	+60% slower
Cache GET	0.04ms	0.06ms	+50% slower
Pub/Sub	1.2ms	3.1ms	+158% slower
Queue push	0.08ms	0.15ms	+87% slower
Queue pop	0.12ms	0.31ms	+158% slower
PostgreSQL is slower... but:

All operations still under 1ms
Eliminates network hop to Redis
Reduces infrastructure complexity
Combined Operations (The Real Win)
Scenario: Insert data + invalidate cache + notify subscribers

With Redis:

await db.query('INSERT INTO posts ...');       // 2ms
await redis.del('posts:latest');                // 1ms (network hop)
await redis.publish('posts:new', data);         // 1ms (network hop)
// Total: ~4ms
With PostgreSQL:

BEGIN;
INSERT INTO posts ...;                          -- 2ms
DELETE FROM cache WHERE key = 'posts:latest';  -- 0.1ms (same connection)
NOTIFY posts_new, '...';                        -- 0.1ms (same connection)
COMMIT;
-- Total: ~2.2ms
PostgreSQL is faster when operations are combined.

When to Keep Redis
Don't replace Redis if:

1. You Need Extreme Performance
Redis: 100,000+ ops/sec (single instance)
Postgres: 10,000-50,000 ops/sec
If you're doing millions of cache reads/sec, keep Redis.

2. You're Using Redis-Specific Data Structures
Redis has:

Sorted sets (leaderboards)
HyperLogLog (unique count estimates)
Geospatial indexes
Streams (advanced pub/sub)
Postgres equivalents exist but are clunkier:

-- Leaderboard in Postgres (slower)
SELECT user_id, score
FROM leaderboard
ORDER BY score DESC
LIMIT 10;

-- vs Redis
ZREVRANGE leaderboard 0 9 WITHSCORES
3. You Have a Separate Caching Layer Requirement
If your architecture mandates a separate cache tier (e.g., microservices), keep Redis.

Migration Strategy
Don't rip out Redis overnight. Here's how I did it:

Phase 1: Side-by-Side (Week 1)
// Write to both
await redis.set(key, value);
await pg.query('INSERT INTO cache ...');

// Read from Redis (still primary)
let data = await redis.get(key);
Monitor: Compare hit rates, latency.

Phase 2: Read from Postgres (Week 2)
// Try Postgres first
let data = await pg.query('SELECT value FROM cache WHERE key = $1', [key]);

// Fallback to Redis
if (!data) {
  data = await redis.get(key);
}
Monitor: Error rates, performance.

Phase 3: Write to Postgres Only (Week 3)
// Only write to Postgres
await pg.query('INSERT INTO cache ...');
Monitor: Everything still works?

Phase 4: Remove Redis (Week 4)
# Turn off Redis
# Watch for errors
# Nothing breaks? Success!
Code Examples: Complete Implementation
Cache Module (PostgreSQL)
// cache.js
class PostgresCache {
  constructor(pool) {
    this.pool = pool;
  }

  async get(key) {
    const result = await this.pool.query(
      'SELECT value FROM cache WHERE key = $1 AND expires_at > NOW()',
      [key]
    );
    return result.rows[0]?.value;
  }

  async set(key, value, ttlSeconds = 3600) {
    await this.pool.query(
      `INSERT INTO cache (key, value, expires_at)
       VALUES ($1, $2, NOW() + INTERVAL '${ttlSeconds} seconds')
       ON CONFLICT (key) DO UPDATE
         SET value = EXCLUDED.value,
             expires_at = EXCLUDED.expires_at`,
      [key, value]
    );
  }

  async delete(key) {
    await this.pool.query('DELETE FROM cache WHERE key = $1', [key]);
  }

  async cleanup() {
    await this.pool.query('DELETE FROM cache WHERE expires_at < NOW()');
  }
}

module.exports = PostgresCache;
Pub/Sub Module
// pubsub.js
class PostgresPubSub {
  constructor(pool) {
    this.pool = pool;
    this.listeners = new Map();
  }

  async publish(channel, message) {
    const payload = JSON.stringify(message);
    await this.pool.query('SELECT pg_notify($1, $2)', [channel, payload]);
  }

  async subscribe(channel, callback) {
    const client = await this.pool.connect();

    await client.query(`LISTEN ${channel}`);

    client.on('notification', (msg) => {
      if (msg.channel === channel) {
        callback(JSON.parse(msg.payload));
      }
    });

    this.listeners.set(channel, client);
  }

  async unsubscribe(channel) {
    const client = this.listeners.get(channel);
    if (client) {
      await client.query(`UNLISTEN ${channel}`);
      client.release();
      this.listeners.delete(channel);
    }
  }
}

module.exports = PostgresPubSub;
Job Queue Module
// queue.js
class PostgresQueue {
  constructor(pool) {
    this.pool = pool;
  }

  async enqueue(queue, payload, scheduledAt = new Date()) {
    await this.pool.query(
      'INSERT INTO jobs (queue, payload, scheduled_at) VALUES ($1, $2, $3)',
      [queue, payload, scheduledAt]
    );
  }

  async dequeue(queue) {
    const result = await this.pool.query(
      `WITH next_job AS (
        SELECT id FROM jobs
        WHERE queue = $1
          AND attempts < max_attempts
          AND scheduled_at <= NOW()
        ORDER BY scheduled_at
        LIMIT 1
        FOR UPDATE SKIP LOCKED
      )
      UPDATE jobs
      SET attempts = attempts + 1
      FROM next_job
      WHERE jobs.id = next_job.id
      RETURNING jobs.*`,
      [queue]
    );

    return result.rows[0];
  }

  async complete(jobId) {
    await this.pool.query('DELETE FROM jobs WHERE id = $1', [jobId]);
  }

  async fail(jobId, error) {
    await this.pool.query(
      `UPDATE jobs
       SET attempts = max_attempts,
           payload = payload || jsonb_build_object('error', $2)
       WHERE id = $1`,
      [jobId, error.message]
    );
  }
}

module.exports = PostgresQueue;
Performance Tuning Tips
1. Use Connection Pooling
const { Pool } = require('pg');

const pool = new Pool({
  max: 20,  // Max connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
2. Add Proper Indexes
CREATE INDEX CONCURRENTLY idx_cache_key ON cache(key) WHERE expires_at > NOW();
CREATE INDEX CONCURRENTLY idx_jobs_pending ON jobs(queue, scheduled_at) 
  WHERE attempts < max_attempts;
3. Tune PostgreSQL Config
# postgresql.conf
shared_buffers = 2GB           # 25% of RAM
effective_cache_size = 6GB     # 75% of RAM
work_mem = 50MB                # For complex queries
maintenance_work_mem = 512MB   # For VACUUM
4. Regular Maintenance
-- Run daily
VACUUM ANALYZE cache;
VACUUM ANALYZE jobs;

-- Or enable autovacuum (recommended)
ALTER TABLE cache SET (autovacuum_vacuum_scale_factor = 0.1);
The Results: 3 Months Later
What I saved:

✅ $100/month (no more ElastiCache)
✅ 50% reduction in backup complexity
✅ One less service to monitor
✅ Simpler deployment (one less dependency)
What I lost:

❌ ~0.5ms latency on cache operations
❌ Redis's exotic data structures (didn't need them)
Would I do it again? Yes, for this use case.

Would I recommend it universally? No.

Decision Matrix
Replace Redis with Postgres if:

✅ You're using Redis for simple caching/sessions
✅ Cache hit rate is < 95% (lots of writes)
✅ You want transactional consistency
✅ You're okay with 0.1-1ms slower operations
✅ You're a small team with limited ops resources
Keep Redis if:

❌ You need 100k+ ops/second
❌ You use Redis data structures (sorted sets, etc.)
❌ You have dedicated ops team
❌ Sub-millisecond latency is critical
❌ You're doing geo-replication
Resources
PostgreSQL Features:

LISTEN/NOTIFY Docs
SKIP LOCKED
UNLOGGED Tables
Tools:

pgBouncer - Connection pooling
pg_stat_statements - Query performance
Alternative Solutions:

Graphile Worker - Postgres-based job queue
pg-boss - Another Postgres queue
TL;DR
I replaced Redis with PostgreSQL for:

Caching → UNLOGGED tables
Pub/Sub → LISTEN/NOTIFY
Job queues → SKIP LOCKED
Sessions → JSONB tables
Results:

Saved $100/month
Reduced operational complexity
Slightly slower (0.1-1ms) but acceptable
Transactional consistency guaranteed
When to do this:

Small to medium apps
Simple caching needs
Want to reduce moving parts
When NOT to do this:

High-performance requirements (100k+ ops/sec)
Using Redis-specific features
Have dedicated ops team
Have you replaced Redis with Postgres (or vice versa)? What was your experience? Drop your benchmarks in the comments! 👇

P.S. - Want a follow-up on "PostgreSQL Hidden Features" or "When Redis is Actually Better"? Let me know!
