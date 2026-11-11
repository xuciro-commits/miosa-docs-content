---
slug: "forge/agents"
title: "Agents and the Forge API"
description: "Use Forge from the TypeScript and Python SDKs and the REST API: repositories, files, branches, pull requests, checks and releases."
section: "Services & projects"
order: 1037
---

Agents reach Forge the same way you do: with a MIOSA API key, under the same organization roles and branch policy.
Nothing an agent does can skip a policy that would stop you.

## From the TypeScript SDK

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

`miosa.forge.repositories` also has `create`, `get`, `refs`, `tree`, `readme`, `commits`, `putFile`, `deleteFile`, `update` and `delete`.
The Python SDK has the same surface as `client.forge`.

## The REST API

All routes are under `https://api.miosa.ai/api/v1/forge/repositories` and take your key as `Authorization: Bearer <key>`.
`:id` is the repository id.

| Method | Path | What it does |
|---|---|---|
| `GET` | `/` | List repositories |
| `POST` | `/` | Create a repository |
| `GET` | `/:id` | Show a repository |
| `GET` | `/:id/refs` | Branches and tags |
| `GET` | `/:id/tree?ref=main&path=` | A directory listing |
| `GET` | `/:id/blob?ref=main&path=README.md` | A file's content |
| `GET` | `/:id/commits?ref=main` | Commit history |
| `POST` | `/:id/branches` | Create a branch: `{"name": "...", "from": "main"}` |
| `PUT` | `/:id/files/:path` | Write a file as a commit on a branch |
| `GET` | `/:id/pull-requests` | List pull requests |
| `POST` | `/:id/pull-requests` | Open a pull request: `{"title", "head", "base"}` |
| `POST` | `/:id/pull-requests/:number/reviews` | Review a pull request |
| `POST` | `/:id/pull-requests/:number/merge` | Merge a pull request |
| `POST` | `/:id/pull-requests/:number/close` | Close a pull request |
| `GET` | `/:id/check-suites` | Checks reported against commits |
| `GET` | `/:id/releases` | List releases |
| `POST` | `/:id/releases` | Publish a release |

> **Note: Every write needs an Idempotency-Key**
>
> Each `POST` and `PUT` needs an `Idempotency-Key` header.
> Repeating a request with the same key returns the first result instead of acting twice, so an agent can retry safely.
> Without the header the API answers `IDEMPOTENCY_KEY_REQUIRED`.

### Write a file to a branch

A file write is a commit.
It needs the branch's current head commit as `expected_head`, so two writers never overwrite each other silently:

```bash
curl -X PUT "https://api.miosa.ai/api/v1/forge/repositories/$REPO_ID/files/public%2Fapi.html" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d "{\"path\": \"public/api.html\", \"content\": \"<p>From the API.</p>\\n\", \"message\": \"Add a page from the API\", \"branch\": \"api-branch\", \"expected_head\": \"$HEAD_SHA\"}"
```

The answer has the new commit and the policy decision that allowed it.
Without `expected_head` the API answers `EXPECTED_HEAD_REQUIRED`; get the head from `GET /:id/refs`.
