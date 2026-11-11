---
slug: "optimal-engine/cli-sdks-mcp"
title: "Optimal Engine CLI, SDKs and MCP"
description: "The optimal client CLI, the Engine's MCP server for Claude Code and Codex, the mix tasks for operators, and the TypeScript and Python clients - how to install each from the repository and point it at an Engine."
section: "Optimal Engine"
order: 8009
---

Every interface below talks to the same HTTP API, so a workspace looks the same from each.
All of them install from a checkout of [Miosa-osa/OptimalEngine](https://github.com/Miosa-osa/OptimalEngine).

| Interface | For | Where |
|---|---|---|
| `optimal` CLI (also `oe`) | People and agents in a terminal | `apps/cli` |
| MCP server | Agent clients such as Claude Code and Codex | `apps/mcp` |
| Mix tasks | Operators on the Engine's own machine | `lib/mix/tasks` |
| TypeScript client | Apps | `sdks/typescript` |
| Python client | Apps and scripts | `sdks/python` |

> **Note: Not on npm or PyPI**
>
> The clients and the MCP server are not published to npm or PyPI.
> Build them from the repository as shown below.

## The optimal CLI

A single shell script that needs `sh`, `curl` 7.55 or newer, and `awk`.
It sends the key to curl on standard input, so the key never shows in a process list, and it never prints the key.

```bash
apps/cli/install.sh            # installs optimal and oe into /usr/local/bin or ~/.local/bin
optimal --version
```

### Point it at an Engine

Settings are read in this order, first match wins:

1. Flags: `--api <url>`, `--key-file <path>`, `-w <workspace>`, `--profile <name>`.
2. A named profile, from `--profile` or `OPTIMAL_PROFILE`.
3. Environment: `OPTIMAL_ENGINE_URL`, `OPTIMAL_ENGINE_API_KEY` or `OPTIMAL_ENGINE_API_KEY_FILE`, `OPTIMAL_WORKSPACE`.
4. The default profile, `~/.config/optimal/profile`.
5. `http://127.0.0.1:4200`.

A profile file holds the same settings:

```text
url=http://127.0.0.1:4200
key_file=/home/me/.config/optimal/key
workspace=default:notes
```

`optimal doctor` reports where each setting came from and whether the Engine answers.
On MIOSA, `miosa engine connect` writes this profile for you; see [Optimal Engine on MIOSA](/docs/optimal-engine/miosa#from-a-terminal).

### Commands

| Group | Commands |
|---|---|
| Session | `boot`, `today`, `fires`, `brief`, `protocol`, `commitments` |
| Loops | `loops list`, `loops add`, `loops close`, `loops drop`, `loops reopen`, `loops extract` |
| Retrieve | `search` (`find`), `grep`, `assemble`, `read`, `ask` (`rag`), `reconstruct`, `memory`, `claims`, `review` |
| Record | `capture`, `aware`, `note`, `lesson`, `decision`, `task`, `close`, `remember` |
| Chats and policy | `session commit`, `session list`, `session show`, `policy`, `org-section` |
| Rules | `routing`, `authority effective`, `authority decide`, `authority request`, `authority export` |
| Setup and health | `init`, `size`, `viability`, `chain`, `check`, `analyses`, `jobs` |
| Engine | `status`, `health`, `version`, `doctor`, `url`, `workspaces`, `organizations`, `whoami` |
| Any route | `api <METHOD> /api/<path> [json]` |

```bash
optimal capture "Decided: annual plans get two months free." -w default:notes --title "Pricing"
optimal search "annual plans" -w default:notes -n 5
optimal ask "What did we decide about pricing?" -w default:notes
optimal api GET /api/version
```

Common options: `-n` for a limit, `--as-of` for a point in time, `--mode lexical|hybrid|auto`, `--field a.b` to print one field, and `--raw` for unformatted JSON.

| Exit code | Meaning |
|---|---|
| 0 | Done |
| 1 | Failed |
| 64 | Wrong usage |
| 65 | The Engine's answer could not be read |
| 69 | The Engine did not answer |
| 77 | The key was refused (401 or 403) |
| 78 | A setting is wrong, the command was refused, or the client is too old (426) |

## MCP server

A stdio MCP server over the same API, for agent clients.
It needs Node 20 or newer.

```bash
cd apps/mcp
npm ci && npm run build
```

It reads the same settings as the CLI, from the environment or a profile, once when it starts.
Restart it after the key changes.

Register it with Claude Code:

```bash
claude mcp add --scope user \
  --env OPTIMAL_ENGINE_URL=http://127.0.0.1:4200 \
  --env OPTIMAL_ENGINE_API_KEY_FILE=/home/me/.config/optimal/key \
  --env OPTIMAL_WORKSPACE=default:notes \
  optimal-engine -- node /path/to/OptimalEngine/apps/mcp/dist/index.js
```

Give it a key with only what its agent needs, for example `read` and `write` on one workspace.

| Group | Tools |
|---|---|
| Operating loop | `boot`, `brief`, `protocol`, `loops_list`, `loops_add`, `loops_close` |
| Retrieve | `search`, `grep`, `ask`, `assemble`, `read`, `recall`, `profile`, `wiki_get`, `workspaces` |
| Record | `capture`, `record`, `close`, `add_memory`, `forget_memory` |
| Chats and policy | `session_commit`, `sessions_list`, `session_get`, `org_policy` |
| Rules and checks | `authority_decide`, `authority_effective`, `check_draft`, `viability`, `action_step`, `action_chain` |
| Rendering | `render_context`, `render_specs` |

## Mix tasks

On the Engine's own machine, a checkout has `mix optimal.*` tasks that work on the store directly.
Useful ones:

| Task | What it does |
|---|---|
| `mix optimal.auth` | Mint, list, revoke, and delete API keys |
| `mix optimal.bootstrap` | Compile, migrate, and load the sample workspace |
| `mix optimal.status`, `mix optimal.health` | Liveness, and ten store diagnostics |
| `mix optimal.ingest`, `mix optimal.ingest_workspace` | Ingest text or a folder |
| `mix optimal.import_notion` | Import a Notion export |
| `mix optimal.connector`, `mix optimal.pull` | Set up and run app connectors |
| `mix optimal.search`, `mix optimal.rag`, `mix optimal.assemble` | Read from the command line |
| `mix optimal.claims`, `mix optimal.facts` | Review claims and list facts |
| `mix optimal.backup`, `mix optimal.restore`, `mix optimal.migrate` | Operations; see [Operations](/docs/optimal-engine/operations) |
| `mix optimal.embed_backfill`, `mix optimal.index` | Embed missing chunks, or reindex |
| `mix optimal.compliance` | Data export, erasure, legal holds, and retention |
| `mix optimal.conformance` | Run the conformance suite against a live Engine |

`docs/guides/mix-tasks.md` in the repository lists all of them.

## TypeScript client

```bash
cd sdks/typescript
npm install && npm run build
```

```ts
import { OptimalEngine } from "@optimal-engine/client";

const engine = new OptimalEngine({
  baseUrl: "http://127.0.0.1:4200",
  apiKey: process.env.OPTIMAL_ENGINE_API_KEY,
  workspace: "default:notes",
});

await engine.ingest("Decided: annual plans get two months free.", { title: "Pricing" });
const hits = await engine.search("annual plans");
const context = await engine.ask("What did we decide about pricing?");
console.log(context.envelope.body, context.envelope.sources);
```

It also has `grep`, `assemble`, `profile`, `createNode`, and sub-clients for `memory`, `workspaces`, `wiki`, `recall`, `subscriptions`, `surface`, and `render`.
Adapters for the Vercel AI SDK and the OpenAI Agents SDK are at `@optimal-engine/client/adapters/ai-sdk` and `/adapters/openai-agents`.

## Python client

```bash
pip install ./sdks/python
```

```python
from optimal_engine import OptimalEngine

engine = OptimalEngine(base_url="http://127.0.0.1:4200", workspace="default:notes", api_key="oe_...")
results = engine.search("annual plans")
memory = engine.memory.create(content="Customer Example Co renews in March.")
```

It needs Python 3.10 or newer, and has a sync `OptimalEngine` and an async `AsyncOptimalEngine`.
It covers search, grep, recall, memory, and workspaces.

> **Caution: Python ask**
>
> In the current Python client, `ask()` reads the answer from the wrong field and returns an empty `body`.
> Call `POST /api/rag` directly, or use the TypeScript client, until it is fixed.
