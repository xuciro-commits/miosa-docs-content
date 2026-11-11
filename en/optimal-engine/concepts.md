---
slug: "optimal-engine/concepts"
title: "Optimal Engine concepts"
description: "The objects Optimal Engine is built from - tenants, workspaces, nodes, signals, source packages, claims, facts, memories, context packages, and projections - and how a source becomes reviewed memory."
section: "Optimal Engine"
order: 8001
---

Optimal Engine keeps three things apart: the evidence you gave it, the truth that was accepted from that evidence, and the views it builds for a reader.
Every object below belongs to one of those three.

## Topology

```text
Tenant / Organization
  -> Workspaces
      -> Nodes
          -> sources, signals, memories, wiki pages
```

| Object | What it is |
|---|---|
| Tenant / Organization | The top scope. It owns workspaces and holds the organization policy every workspace reads. |
| Workspace | One walled section with its own nodes, sources, memories, loops, and rhythm. A key can be limited to some workspaces. |
| Node | A typed thing inside a workspace. |
| Node relationship | A typed link between two nodes. |

A workspace id is `<tenant>:<slug>`, for example `default:sales` for the workspace `sales` in the `default` tenant.
The workspace `default` always exists.

Node kinds, as `POST /api/nodes` accepts them: `unit`, `team`, `project`, `entity`, `domain`, `department`, `operation`, `learning`, `person`, `product`, `partnership`, `context`.

Each new workspace also gets the standard node types the setup and topology tools use: `entity`, `division`, `department`, `team`, `role`, `project`, `process`, `task`, `operation`, `learning`, `person`, `customer`, `product`, `partnership`, `context`.

Relationship types: `parent_child`, `depends_on`, `blocks`, `references`, `collaborates_with`, `sequence_before`, `instance_of`, `owns`, `participates_in`, `supports`.

A project is only one kind of node.
Model ongoing work, such as hiring or support, as an `operation` node instead.

## From a source to memory

```text
source -> signal -> claim -> fact -> context package
```

| Object | What it is |
|---|---|
| Source package | The raw input, kept with its metadata, trust label, and retention state. It is never rewritten. |
| Signal | The classified unit the Engine works on. |
| Context | The indexed unit search returns: a signal, a resource, a memory, or a skill file. |
| Claim | An assertion the Engine found in a source, with a link back to it. A claim is pending until it is reviewed. |
| Fact | An accepted claim, with its evidence, confidence, precision, and the time it is valid for. |
| Memory object | The meaning around facts, sources, and time, such as an episode from a chat session. |
| Derivation ledger | The record of which source each claim, fact, and memory came from. |

### Signals

Every input is classified on five separate dimensions:

```text
Signal = Mode + Genre + Type + Format + Structure
```

Keeping them separate lets the Engine route, parse, and retrieve a meeting transcript differently from a spec or a code file.
Classification is rule-based and runs without a language model.

### Claims and facts

The Engine never treats an extracted sentence as true on its own.
A claim becomes a fact in one of two ways:

- A person or key with the `claims:review` grant promotes it.
- The workspace's promotion rule promotes it automatically.

The rule is set with `OPTIMAL_CLAIM_PROMOTION_MODE`:

| Mode | Behavior |
|---|---|
| `auto_threshold` (default) | Promotes a claim whose score reaches the workspace threshold, 0.85 by default |
| `manual` | Every claim waits for review |
| `agent_review` | Currently behaves like `manual` |

A promoted claim is kept and marked `promoted`, so the evidence trail stays complete.
A newer fact can supersede an older one, and the older one stays readable as history.

### Versioned memories

Alongside the claim lifecycle, the Engine keeps a simpler memory store you write to directly with `POST /api/memory`.
Each memory has versions, and memories link to each other with `updates`, `extends`, `derives`, `contradicts`, or `cites`.
Forgetting a memory hides it without deleting it; deleting is a separate, permanent call.

`POST /api/memory/remember` is the gated way in: it scores a new memory for novelty and salience, then adds it, updates a similar one, skips it, or proposes it as a claim.

## Reading the memory

| Object | What it is |
|---|---|
| Context package | What one reader may see for one task: the facts and memories used, their sources, confidence, and validity. It is a view, not truth. |
| Active memory pool | A working memory for one task, with the observations made during it. |
| Retrieval strategy | `tiered`, `reconstructive`, or `hybrid`. See [Memory and retrieval](/docs/optimal-engine/memory-and-retrieval). |

## Projections

The Engine's database is the source of truth.
Everything else is a projection of it:

- **Workspace files**: each workspace has a folder with `nodes/`, `.wiki/`, `assets/`, and `architectures/`. Editing a file there comes back in as a new source, not as a direct change to the truth.
- **Wiki pages**: curated pages with citations, kept in `.wiki/`.
- **Workspace bundles**: a whole workspace exported as one file and imported into another Engine.
- **The API, CLI, and MCP server**: the same state, read by people and agents.

## Governance

| Object | What it is |
|---|---|
| API key | Identifies the caller. It carries scopes and a list of workspaces it may use. |
| Authority | Allow, ask, or deny rules for reading, searching, writing, exporting, erasing, and syncing. Deny wins. It runs in `off`, `observe`, or `enforce` mode. |
| Organization policy | Rules set once for the organization that every workspace reads and cannot change. |
| Security labels and partitions | Labels such as `confidential` on rows, and the partitions a key may read. |

> **Note: Words in the API**
>
> The API uses the same names as this page: `workspace`, `node`, `claim`, `memory`, `context_package`.
> URIs in results use the `optimal://` scheme.
