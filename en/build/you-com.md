---
slug: "build/you-com"
title: "Build a You.com-style search and answer API"
description: "You.com: An API your agents call to search and answer, with sources. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10036
---

`~25 min` `TypeScript` `Python`

> **What you're building**: An API your agents call to search and answer, with sources.
>
> **Primitives you'll use**: Sandbox, Agent and harness, Qdrant vectors, Deployment (Docker Host)

## What you're building

a search-and-answer API: your agents call it, and it returns an answer with its sources Modelled on You.com, by You.com.

A query goes to a search source you bring; the top pages are fetched and read, and a model answers from them with the source URLs attached, returned as JSON for your agents to consume.

Primitives you'll use: Sandbox · Agent and harness · Qdrant vectors · Deployment (App Engine)

- A search endpoint your agents call
- Answers grounded in the pages it read
- Source URLs on every answer
- A model-free search mode that stays cheap

Source: [you.com](https://you.com/). You.com is a trademark of its owner; this guide is not affiliated with or endorsed by them.

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Sandbox - The isolated Linux workspace the agent writes code and runs commands in. - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. Qdrant vectors - Managed vector store for search and embeddings, injected as QDRANT_URL. - `miosa api POST /databases -d '{"name":"app-index","engine":"qdrant"}'` [Docs](/docs/data/qdrant)
4. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
5. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
6. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)
7. Agent and harness - Turns a prompt into work: pick the harness and model a run uses. - `miosa agent harnesses` [Docs](/docs/api-reference/agent-runs)
8. Deployment (App Engine) - Publishes the app to an immutable, versioned URL with rollback. - `miosa deploy create --from-sandbox app-box --name app --wait` [Docs](/docs/docker-host/overview)

## Architecture

```mermaid
sequenceDiagram
  participant U as User
  participant P as Your pipeline
  participant S as Sandbox
  participant V as Vector index
  participant M as Model

  U->>P: ask a question
  P->>S: fetch and extract the top pages
  S->>V: embed and store chunks
  P->>V: retrieve the best chunks
  P->>M: answer from those chunks only
  M-->>P: answer with citations
  P-->>U: answer and its sources
```

## How it maps onto MIOSA

Searches and reads sources -> a MIOSA sandbox running fetch/extract jobs (a computer when pages need a real browser)

Retrieval over what it read -> managed Qdrant (`QDRANT_URL`) for chunk embeddings

Answers with citations -> agent runs (`miosa prompt`, client.runs) streaming events that answer only from retrieved chunks and return source ids

Serves the answer UI or API -> a MIOSA deployment (`miosa deploy create`), immutable and versioned

## Data and storage

Qdrant holds chunk embeddings; the payload carries url, title, chunk id and fetched_at so every answer can cite a saved page.

Keep threads and answers in your own datastore, with their citations.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create you-com-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Agent loop

Harness: OSA is MIOSA's own harness and works with any model provider you connect, including your own model.

Model: Anthropic (Claude). Calls use my own provider key (`miosa connections add models`); MIOSA platform keys are never used.

Sessions: one chat per project or conversation, so the agent keeps its context. The first `miosa prompt` on a sandbox uses `--new-chat` (a chat id is printed); every later turn passes `--chat <chat-id>`. `--reuse chat` keeps one new machine per chat so files persist.

Streaming: follow a run with `miosa run follow <run-id>` or client.runs.streamEvents(run.id), and steer or stop it with `miosa run steer` and `miosa run interrupt`.

Retrieval comes first, generation second: the model sees only retrieved chunks and must cite them. The harness runs in the pipeline sandbox so fetching and extraction code is isolated.

## Steps

1. Create the vector index.
   ```bash
   miosa api POST /databases -d '{"name":"you-com-index","engine":"qdrant"}'
   ```
   Check: QDRANT_URL and QDRANT_API_KEY are available to the pipeline.
2. Create the pipeline sandbox that fetches, extracts and indexes pages.
   ```bash
   miosa create you-com-box --template agent-python --wait
   ```
   Check: The sandbox is running and can fetch a public URL.
3. Ingest: crawl the corpus you serve, extract clean text, and store url, title and fetched_at with every page. (Search sources you do not crawl are an external API you bring.)
   Check: Every stored page has a URL and a fetch time.
4. Chunk, embed and upsert into Qdrant with the url, title and chunk id in the payload.
   Check: The chunk count in Qdrant matches what you extracted.
5. Serve search as an API: POST /search returns ranked results with their text. Issue API keys to your callers, rate-limit per key, and meter calls.
   Check: A call without a key is rejected and one with a key returns results with source URLs.
6. Optional answer endpoint: POST /answer runs the harness over the top results and returns a short answer with the same citations. Plain /search stays model-free and cheap.
   ```bash
   miosa prompt --sandbox you-com-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Answer the query using only these search results; cite them as [n]"
   ```
   Check: Every sentence of the answer cites a returned result.
7. Publish the API.
   ```bash
   miosa deploy create --from-sandbox you-com-box --name you-com --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   Check: The public_url answers a question end to end.

## Limits and costs

Respect robots.txt and rate limits when crawling; cache fetched pages so repeated questions do not refetch.

Bound retrieval: a fixed number of chunks per answer keeps latency and cost predictable.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

Answers cite sources and the citations resolve to saved pages.

A question the index cannot answer is declined, not invented.

Per-key rate limits and metering work.

Each customer is isolated in its own workspace and usage is metered against them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
