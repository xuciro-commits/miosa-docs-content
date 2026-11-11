---
slug: "optimal-engine/memory-and-retrieval"
title: "Memory and retrieval"
description: "How Optimal Engine searches (full-text, vector, hybrid), what ask and assemble return, how claims are reviewed into facts, how versioned memories work, and what the benchmark numbers mean."
section: "Optimal Engine"
order: 8006
---

Optimal Engine has four ways to read memory, from a plain search to a governed context package for an agent.

| Call | Returns | Use it for |
|---|---|---|
| `GET /api/search` | Ranked results with their sources | Finding things |
| `GET /api/grep` | Matching chunks, with intent, scale, and modality filters | Finding exact passages |
| `POST /api/rag` (`optimal ask`) | A context envelope sized for a reader, with sources | Giving a model what it needs to answer one question |
| `POST /api/assemble` (`optimal assemble`) | A tiered context package with reviewed facts and memories | Briefing an agent before it acts |

None of them writes an answer with a language model.
They return context and the sources it came from; your model writes the answer.

## Search

```bash
curl -s "http://127.0.0.1:4200/api/search?q=renewal&workspace=default:notes&limit=10&mode=auto" \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

| Parameter | Default | Meaning |
|---|---|---|
| `q` | required | Query |
| `workspace` | `default` | Workspace id |
| `limit`, `offset` | `10`, `0` | Paging |
| `mode` | `auto` | `auto`, `lexical`, or `hybrid` |
| `type` | all | `memory` searches only durable memories |
| `as_of` | now | ISO 8601 instant to measure recency against |

Each result has an `id`, `title`, `uri`, `content`, a short `l0_abstract`, a `score`, and the `workspace_id` it came from.
`inherited: true` marks a result that came from the organization policy rather than the workspace itself.

### How ranking works

- **Full-text**: SQLite FTS5 BM25, multiplied by a recency decay that depends on the kind of source and by a signal-quality score.
- **Vector**: cosine similarity over embeddings, when an embedding model is set up.
- **Hybrid**: a weighted mix of the two, then a boost for results connected in the graph.

| `mode` | Behavior |
|---|---|
| `auto` | Uses vectors when the embedding model answers, and full-text otherwise |
| `lexical` | Full-text only |
| `hybrid` | Requires vectors; answers `503 hybrid_unavailable` instead of quietly falling back |

Ties are broken by `uri` and then `id`, so the same query on the same data returns the same order.

> **Note: Vector scale**
>
> Vectors are stored in SQLite and compared in the Engine process, without an approximate nearest-neighbor index.
> Vector search time grows with the number of chunks, so watch `search_latency_ms` in [metrics](/docs/optimal-engine/operations#observability) as a workspace grows.

## Ask

```bash
curl -s -X POST http://127.0.0.1:4200/api/rag \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What did we decide about pricing?", "workspace": "default:notes", "format": "claude", "bandwidth": "small"}'
```

The Engine reads the intent of the question, looks for a curated wiki page first, searches when there is none, and fits the result to the reader.

| Field | Values |
|---|---|
| `format` | `plain`, `markdown`, `claude`, `openai` |
| `bandwidth` | `small`, `medium`, `large` |
| `strategy` | `tiered`, `reconstructive`, `hybrid` |
| `audience` | Who will read it, used to filter and shape the result |
| `context_package` | `true` returns a governed context package as well |

```json
{
  "source": "chunks",
  "envelope": {
    "body": "...",
    "format": "claude",
    "sources": ["optimal://..."],
    "warnings": []
  },
  "trace": {"intent": "...", "n_candidates": 12, "n_delivered": 4, "elapsed_ms": 41}
}
```

`source` says where the context came from: `wiki`, `chunks`, `memory`, `context_package`, or `empty`.
`GET /api/rag/stream` returns the same steps as server-sent events: `intent`, `wiki_hit` or `chunks`, `composing`, `envelope`, and `done`.

## Assemble and reconstruct

`POST /api/assemble` returns what an agent should read before acting: tiers `l0` to `l3`, the `total_tokens`, the `sources`, and a `context_package`.
The context package holds reviewed facts and memory objects only, so a new source shows up there after its claims are promoted.

`POST /api/reconstruct` uses the reconstructive strategy: it expands across associations between memories the reader is allowed to see.
Associations can improve recall, but they never decide what is true.
Record how useful a reconstruction was with `POST /api/reconstruct/:id/outcome`, which the Engine credits per workspace and intent.

## Reviewing claims

```bash
optimal claims -w default:notes
optimal review -w default:notes
```

| Route | What it does |
|---|---|
| `GET /api/memory-core/claim-review` | The review queue |
| `GET /api/memory-core/claims` | Claims, with their status and sources |
| `POST /api/memory-core/claims/:id/promote` | Accept a claim as a fact (`claims:review`) |
| `POST /api/memory-core/claims/:id/reject` | Reject a claim (`claims:review`) |

The default rule promotes a claim on its own when its score reaches the workspace threshold of 0.85.
Set `OPTIMAL_CLAIM_PROMOTION_MODE=manual` to review every claim yourself.

## Versioned memories

```bash
curl -s -X POST http://127.0.0.1:4200/api/memory/remember \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace": "default:notes", "content": "Customer Example Co renews in March."}'
```

`remember` scores the memory first and answers with an `action`:

| `action` | Meaning |
|---|---|
| `add` | Stored as a new memory |
| `update` | Stored as a new version of a similar memory |
| `skip` | Already known; nothing changed |
| `pending_claim` | Proposed as a claim for review |
| `rejected` | Below the gate |

The `gate` object explains the score: `novelty`, `salience`, `prediction_error`, and `reason`.
`POST /api/memory` stores a memory without the gate.

| Route | What it does |
|---|---|
| `GET /api/memory` | List, with `q`, `include_forgotten`, and paging |
| `GET /api/memory/:id/versions` | The version chain |
| `GET /api/memory/:id/relations` | Links: `updates`, `extends`, `derives`, `contradicts`, `cites` |
| `POST /api/memory/:id/update`, `extend`, `derive` | A new linked version |
| `POST /api/memory/:id/forget` | Hide it; the row stays |
| `DELETE /api/memory/:id` | Delete it permanently |

## Time travel

`search`, `grep`, `ask`, and `assemble` take `as_of`, an ISO 8601 instant.
Facts carry the time they are valid for, and a superseded fact stays readable as history.

## Benchmarks

On LoCoMo, all 1,540 questions, the Engine's candidate portfolio retrieves at least one gold evidence item for 96.1% of questions and 87.1% of all gold evidence at top 100.
That measures retrieval, not answer accuracy.
The candidate portfolio is opt-in (`memory_search_mode=portfolio` on a memory search); the default memory search is `hybrid`.
The repository's README has the full results, their limits, and the commands that reproduce them under `benchmarks/`.
