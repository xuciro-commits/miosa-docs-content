---
slug: "build/comet"
title: "Build a Comet-style agentic browser"
description: "Comet: A browser that navigates and answers in place. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10035
---

`~25 min` `TypeScript` `Python`

> **What you're building**: A browser that navigates and answers in place.
>
> **Primitives you'll use**: Computer, Sandbox, Agent and harness, Qdrant vectors, Deployment (Docker Host)

## What you're building

an agentic browser that navigates, extracts and answers in place Modelled on Comet, by Perplexity.

A browser with an assistant built in: it reads the current page, navigates for the user and answers with sources.

Primitives you'll use: Computer · Sandbox · Agent and harness · Qdrant vectors · Deployment (App Engine)

- An assistant inside the browser
- Navigates and extracts on its own
- Answers about the current page
- Cited answers

Source: [www.perplexity.ai/comet](https://www.perplexity.ai/comet). Comet is a trademark of Perplexity; this guide is not affiliated with or endorsed by them.

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Computer - A persistent Linux desktop for browser, mouse, and screenshot work. - `miosa computer create app-desktop` [Docs](/docs/computers/overview)
3. Sandbox - The isolated Linux workspace the agent writes code and runs commands in. - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
4. Qdrant vectors - Managed vector store for search and embeddings, injected as QDRANT_URL. - `miosa api POST /databases -d '{"name":"app-index","engine":"qdrant"}'` [Docs](/docs/data/qdrant)
5. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
6. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
7. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)
8. Agent and harness - Turns a prompt into work: pick the harness and model a run uses. - `miosa agent harnesses` [Docs](/docs/api-reference/agent-runs)
9. Deployment (App Engine) - Publishes the app to an immutable, versioned URL with rollback. - `miosa deploy create --from-sandbox app-box --name app --wait` [Docs](/docs/docker-host/overview)

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

A real browser the agent drives -> a MIOSA computer (persistent Linux desktop with a browser)

## Data and storage

Qdrant holds chunk embeddings; the payload carries url, title, chunk id and fetched_at so every answer can cite a saved page.

Keep threads and answers in your own datastore, with their citations.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create comet-customer-1
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
   miosa api POST /databases -d '{"name":"comet-index","engine":"qdrant"}'
   ```
   Check: QDRANT_URL and QDRANT_API_KEY are available to the pipeline.
2. Create the pipeline sandbox that fetches, extracts and indexes pages.
   ```bash
   miosa create comet-box --template agent-python --wait
   ```
   Check: The sandbox is running and can fetch a public URL.
3. Create the computer the assistant browses with: pages that need a real browser (logins, scripts) are read there.
   ```bash
   miosa computer create comet-desktop
   ```
   Check: computer.screenshot() shows a page the assistant opened.
4. Retrieve: for a question, call a search source you bring (a search API) or crawl a defined corpus, fetch the pages, and extract clean text with url, title and fetched_at.
   Check: Every stored page has a URL and a fetch time.
5. Chunk, embed and upsert into Qdrant with the url, title and chunk id in the payload.
   Check: The chunk count in Qdrant matches what you extracted.
6. Answer: give the model ONLY the retrieved chunks and instruct it to answer from them, citing chunk numbers; if the chunks do not contain the answer it must say so.
   ```bash
   miosa prompt --computer comet-desktop --harness osa --model <anthropic-model-id> --chat <chat-id> "Answer the question using only the provided chunks; cite them as [n]"
   ```
   Check: Every claim carries a citation that resolves to a saved page; an unanswerable question gets "I could not find this".
7. Follow-ups: keep a thread id and the last turns, and rewrite the follow-up into a standalone query before retrieval.
   Check: "And what about its price?" retrieves about the right product.
8. Publish the answer UI: stream the answer text first, then the citations.
   ```bash
   miosa deploy create --from-sandbox comet-box --name comet --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   Check: The public_url answers a question end to end.

## Limits and costs

Respect robots.txt and rate limits when crawling; cache fetched pages so repeated questions do not refetch.

Bound retrieval: a fixed number of chunks per answer keeps latency and cost predictable.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

Answers cite sources and the citations resolve to saved pages.

A question the index cannot answer is declined, not invented.

Each customer is isolated in its own workspace and usage is metered against them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
