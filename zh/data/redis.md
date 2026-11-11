---
slug: "data/redis"
title: "Redis/缓存"
description: "每个项目托管 Redis，用于缓存、会话和发布/订阅。"
section: "Databases & storage"
order: 4003
---

> **提示：可用**
>
> 托管 Redis 已上线。
> 每个项目实例和 `REDIS_URL` 注入可用。
> Redis 是显式配置或附加的，不应假定它存在于每个沙箱模板中。

## 你得到什么

每个项目：一个托管 Redis 实例，凭据注入为 `REDIS_URL`：
```bash

REDIS_URL=redis://default:pass@host:port/0
```

标准 Redis 客户端无需修改即可工作：
```typescript
import { createClient } from "redis"
const redis = createClient({ url: process.env.REDIS_URL })
await redis.connect()

await redis.set("user:123:last-seen", new Date().toISOString())
const value = await redis.get("user:123:last-seen")
```

## 用例

- **缓存** - 短暂的计算结果。
- **会话** - 快速会话/凤凰会话/类似。
- **Pub/sub** - 智能体协调，实时扇出。
- **速率限制** - 原子 INCR + TTL。
- **作业队列** - Bull、BullMQ、Sidekiq、Oban 轻型替代品。

对于大规模的完整作业队列工作负载，MIOSA 有一个 [后台作业](/docs/docker-host/runtime-instances) 原语，它比在 Redis 上滚动您自己的原语更适合。

## 规定
```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "redis",
  size: "small",  // small | medium | large
})
```

## 限制和耐久性

内存、连接和持久性设置取决于返回的托管数据库计划。
检查所选实例的数据库响应和指标。
将 Redis 视为缓存类，除非所选计划明确保证应用程序所需的持久性。

## 发布/订阅注意事项

如果您使用 Redis 发布/订阅进行跨运行时协调，则同一服务的多个运行时实例都可以订阅 - MIOSA 不会删除重复数据或智能体发布/订阅。对于更高容量的实时扇出，请考虑通过 MIOSA 的 [Events](/docs/api-reference/events) 或专用消息智能体使用 SSE。

## 带上自己的Redis？

是的。将 `REDIS_URL` 设置为您喜欢的任何外部 Redis（Upstash、AWS ElastiCache、您自己的）的环境变量。

## 另请参阅

- [概述](/docs/data/overview) - 更广泛的数据平面
- [运行时实例](/docs/docker-host/runtime-instances) - 用于后台作业工作负载
