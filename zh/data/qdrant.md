---
slug: "data/qdrant"
title: "Qdrant（矢量数据库）"
description: "用于语义搜索和嵌入存储的每个项目矢量数据库。 QDRANT_URL 在启动时注入。"
section: "Databases & storage"
order: 4004
---

> **提示：可用**
>
> 托管 Qdrant 已上线。每个项目的专用实例、启动时的 QDRANT_URL 注入以及跨 VM 销毁的数据持久性都在正常工作。

## 何时使用 Qdrant

当智能体或应用程序需要持久语义记忆时，请使用 Qdrant：

- 检索增强生成（RAG）
- 文档和知识库搜索
- 产品或线索相似度匹配
- 跨会话长时间运行的智能体内存
- 在沙箱或计算机内生成的嵌入

使用对象存储存储原始文件，使用 Qdrant 存储向量和有效负载元数据，使这些文件可搜索。

## 你得到什么

每个项目：一个专用的 Qdrant 实例。启动时注入的凭据：

```bash
QDRANT_URL=https://qdrant.proj_xxx.miosa.internal
QDRANT_API_KEY          # 由 MIOSA 注入，请勿填写
```

使用任何与 Qdrant 兼容的客户端：

```typescript
import { QdrantClient } from "@qdrant/js-client-rest"

const client = new QdrantClient({
  url: process.env.QDRANT_URL,
  apiKey: process.env.QDRANT_API_KEY,
})

// Create a collection
await client.createCollection("documents", {
  vectors: { size: 1536, distance: "Cosine" },
})

// Upsert vectors
await client.upsert("documents", {
  points: [
    { id: 1, vector: embedding, payload: { text: "Hello world" } },
  ],
})

// Semantic search
const results = await client.search("documents", {
  vector: queryEmbedding,
  limit: 5,
  with_payload: true,
})
```

```python
from qdrant_client import QdrantClient
import os

client = QdrantClient(
    url=os.environ["QDRANT_URL"],
    api_key=os.environ["QDRANT_API_KEY"],
)

results = client.search(
    collection_name="documents",
    query_vector=query_embedding,
    limit=5,
)
```

## 规定

创建项目时添加Qdrant：

```typescript
const project = await miosa.projects.create({
  name: "my-rag-app",
  data: { qdrant: true },
})
```

或者添加到现有项目中：

```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "qdrant",
  size: "small",  // small | medium | large
})
```

**curl：**

```bash
curl -X POST https://api.miosa.ai/api/v1/projects/$PROJECT_ID/data-services \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"type":"qdrant","size":"small"}'
```

## 虚拟机销毁后数据仍然存在

集合和向量可以在沙箱破坏和部署回滚中幸存下来。 Qdrant 实例在虚拟机生命周期之间保持运行。您不需要在每次启动时重新索引。

## 典型模式：RAG 管道

```typescript
import OpenAI from "openai"
import { QdrantClient } from "@qdrant/js-client-rest"

const openai = new OpenAI()
const qdrant = new QdrantClient({
  url: process.env.QDRANT_URL,
  apiKey: process.env.QDRANT_API_KEY,
})

// Index a document
async function index(id: number, text: string) {
  const { data } = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: text,
  })
  await qdrant.upsert("docs", {
    points: [{ id, vector: data[0].embedding, payload: { text } }],
  })
}

// Query
async function query(question: string, topK = 5) {
  const { data } = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: question,
  })
  return qdrant.search("docs", {
    vector: data[0].embedding,
    limit: topK,
    with_payload: true,
  })
}
```

## 限制

|等级 |内存|磁盘 |最大收藏量 |
|---|---|---|---|
| 小 | 1 GB | 10 GB | 10 |
| 中等 | 4 GB | 50 GB | 50 |
|大| 16GB| 200GB|无限|

每个集合的向量计数受内存限制。在 1536 维（OpenAI `text-embedding-3-small`）时，`small` 适合约 100K 个向量； `medium` 适合 ~400K。

## 带上你自己的 Qdrant？

是的。将 `QDRANT_URL` 和 `QDRANT_API_KEY` 设置为环境变量以指向外部实例（Qdrant Cloud，自托管）：

```typescript
await miosa.deployments.env.set(deploymentId, {
  environment: "production",
  vars: {
    QDRANT_URL: "https://your-cluster.qdrant.io",
    QDRANT_API_KEY: "your-api-key",
  },
})
```

## Python SDK

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Provision a managed Qdrant instance
qdrant = miosa.databases.create(
    name="my-rag-db",
    engine="qdrant",
    size="small",
)
print(qdrant["id"])   # db_...

# Get connection credentials
creds = miosa.databases.credentials(qdrant["id"])
print(creds["url"])       # https://qdrant.proj_xxx.miosa.internal
print(creds["api_key"])

# Teardown
miosa.databases.delete(qdrant["id"])
```

## 另请参阅

- [数据概述](/docs/data/overview)：了解 MIOSA 数据服务如何附加到计算。
- [存储和存储桶](/docs/data/storage)：存储源文件、生成的资产和导出。
- [Postgres](/docs/data/postgres)：关系数据和向量。
- [智能体设备](/docs/guides/agent-devices)：决定工作何时属于沙箱、计算机或本地设备。
