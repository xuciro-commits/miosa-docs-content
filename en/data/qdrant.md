---
slug: "data/qdrant"
title: "Qdrant (Vector Database)"
description: "Per-project vector database for semantic search and embedding storage. QDRANT_URL injected at boot."
section: "Databases & storage"
order: 4004
---

> **Tip: Available**
>
> Managed Qdrant is live. Per-project dedicated instances, QDRANT_URL injection at boot, and data persistence across VM destroy are all working.

## When to use Qdrant

Use Qdrant when an agent or app needs durable semantic memory:

- Retrieval-augmented generation (RAG)
- Document and knowledge-base search
- Product or lead similarity matching
- Long-running agent memory across sessions
- Embeddings produced inside sandboxes or computers

Use object storage for the raw files and Qdrant for the vectors and payload metadata that make those files searchable.

## What you get

Per project: a dedicated Qdrant instance. Credentials injected at boot:

```bash
QDRANT_URL=https://qdrant.proj_xxx.miosa.internal
QDRANT_API_KEY           # from the Qdrant dashboard
```

Use any Qdrant-compatible client:

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

## Provision

Add Qdrant when creating a project:

```typescript
const project = await miosa.projects.create({
  name: "my-rag-app",
  data: { qdrant: true },
})
```

Or add to an existing project:

```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "qdrant",
  size: "small",  // small | medium | large
})
```

**cURL:**

```bash
curl -X POST https://api.miosa.ai/api/v1/projects/$PROJECT_ID/data-services \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"type":"qdrant","size":"small"}'
```

## Data persists past VM destroy

Collections and vectors survive sandbox destruction and deployment rollbacks. The Qdrant instance keeps running between VM lifecycles. You do not need to re-index on every boot.

## Typical pattern: RAG pipeline

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

## Limits

| Tier | Memory | Disk | Max collections |
|---|---|---|---|
| small | 1 GB | 10 GB | 10 |
| medium | 4 GB | 50 GB | 50 |
| large | 16 GB | 200 GB | unlimited |

Vector count per collection is bounded by memory. At 1536 dimensions (OpenAI `text-embedding-3-small`), `small` fits ~100K vectors; `medium` fits ~400K.

## Bring your own Qdrant?

Yes. Set `QDRANT_URL` and `QDRANT_API_KEY` as env vars to point at an external instance (Qdrant Cloud, self-hosted):

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

## See also

- [Data overview](/docs/data/overview): Understand how MIOSA data services attach to compute.
- [Storage and buckets](/docs/data/storage): Store source files, generated assets, and exports.
- [Postgres](/docs/data/postgres): Relational data alongside your vectors.
- [Agent devices](/docs/guides/agent-devices): Decide when work belongs in a sandbox, computer, or local device.
