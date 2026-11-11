---
slug: "forge/agents"
title: "智能体和 Forge API"
description: "使用 TypeScript 和 Python SDK 以及 REST API 中的 Forge：存储库、文件、分支、拉取请求、检查和发布。"
section: "Services & projects"
order: 1037
---

智能体以与您相同的方式到达 Forge：使用 MIOSA API 密钥，在相同的组织角色和分支机构策略下。
智能体人所做的任何事情都无法跳过阻止您的政策。

## 来自 TypeScript SDK

```bash
npm install @miosa/sdk
```

```ts
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY });

// Every repository in your organization.
const repos = await miosa.forge.repositories.list();
console.log(repos.map((repo) => repo.name));

// Read a file from the default branch.
const repo = repos.find((r) => r.slug === "hello-forge");
const file = await miosa.forge.repositories.blob(repo.id, { path: "public/index.html" });
console.log(file.content);
```

`miosa.forge.repositories` 还具有 `create`、`get`、`refs`、`tree`、`readme`、`commits`、`putFile`、`deleteFile`、`update` 和 `delete`。
Python SDK 具有与 `client.forge` 相同的表面。

## REST API

所有路线都在`https://api.miosa.ai/api/v1/forge/repositories`下，并将您的密钥作为`Authorization: Bearer <key>`。
`:id` 是存储库 ID。

|方法|路径|它有什么作用 |
|---|---|---|
| `GET` | `/` |列出存储库 |
| `POST` | `/` |创建存储库 |
| `GET` | `/:id` |显示存储库 |
| `GET` | `/:id/refs` |分支和标签 |
| `GET` | `/:id/tree?ref=main&path=` |目录列表 |
| `GET` | `/:id/blob?ref=main&path=README.md` |文件的内容 |
| `GET` | `/:id/commits?ref=main` |提交历史 |
| `POST` | `/:id/branches` |创建分支： `{"name": "...", "from": "main"}` |
| `PUT` | `/:id/files/:path` |将文件写为分支上的提交 |
| `GET` | `/:id/pull-requests` |列出拉取请求 |
| `POST` | `/:id/pull-requests` |打开拉取请求：`{"title", "head", "base"}` |
| `POST` | `/:id/pull-requests/:number/reviews` |审查拉取请求 |
| `POST` | `/:id/pull-requests/:number/merge` |合并拉取请求 |
| `POST` | `/:id/pull-requests/:number/close` |关闭拉取请求 |
| `GET` | `/:id/check-suites` |针对提交报告的检查 |
| `GET` | `/:id/releases` |发布列表 |
| `POST` | `/:id/releases` |发布版本 |

> **注意：每次写入都需要一个幂等密钥**
>
> 每个 `POST` 和 `PUT` 都需要一个 `Idempotency-Key` 标头。
> 使用相同密钥重复请求会返回第一个结果，而不是执行两次，因此智能体可以安全地重试。
> 如果没有标头，API 会回答 `IDEMPOTENCY_KEY_REQUIRED`。

### 将文件写入分支

文件写入就是提交。
它需要分支的当前头提交为 `expected_head`，因此两个编写器永远不会默默地相互覆盖：

```bash
curl -X PUT "https://api.miosa.ai/api/v1/forge/repositories/$REPO_ID/files/public%2Fapi.html" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d "{\"path\": \"public/api.html\", \"content\": \"<p>From the API.</p>\\n\", \"message\": \"Add a page from the API\", \"branch\": \"api-branch\", \"expected_head\": \"$HEAD_SHA\"}"
```

答案是新的提交和允许它的政策决定。
如果没有 `expected_head`，API 会回答 `EXPECTED_HEAD_REQUIRED`；从`GET /:id/refs`获取头部。
