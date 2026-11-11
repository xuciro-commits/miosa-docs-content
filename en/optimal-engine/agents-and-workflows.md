---
slug: "optimal-engine/agents-and-workflows"
title: "Agents and workflows"
description: "What Optimal Engine gives an agent - a boot briefing, open loops, a daily rhythm, an operating protocol, allow, ask, or deny rules, organization policy, scheduled analyses, webhooks, and action traces."
section: "Optimal Engine"
order: 8007
---

Optimal Engine does not run agents or call tools itself.
It is the memory and the rules an agent works from: what to read first, what is open, what it may do, and a record of what it did.
Any agent reaches it over HTTP, the `optimal` CLI, or the MCP server; see [CLI, SDKs and MCP](/docs/optimal-engine/cli-sdks-mcp).

## Start of a session: boot

```bash
optimal boot -w default:notes
```

`GET /api/boot?workspace=` returns, in one call:

- today's date and the current rhythm note;
- loops that are overdue or due today;
- recent decisions and stale items;
- the brief and the operating protocol;
- the organization policy the workspace follows.

An agent should call it before anything else in a session.

## Daily operation

| Route | CLI | What it does |
|---|---|---|
| `GET /api/brief` | `optimal brief` | Ranked items that need a decision today |
| `GET /api/loops`, `POST /api/loops` | `optimal loops`, `optimal loops add` | Open loops: commitments, replies owed, follow-ups, payments, decisions needed |
| `POST /api/loops/extract` | `optimal loops extract` | Find new loops in recent sources |
| `POST /api/loops/:id/close`, `drop`, `reopen` | `optimal loops close` | Change a loop's state |
| `GET /api/rhythm/current` | `optimal today` | The daily, weekly, or monthly note |
| `GET /api/protocol`, `PUT /api/protocol` | `optimal protocol` | The protocol an agent follows in this workspace |

## Rules for what an agent may do

### Authority

Authority answers allow, ask, or deny for an action on a resource, such as reading, writing, exporting, erasing, or syncing.
Deny wins, and anything without a rule is denied.

```bash
optimal authority decide --action export --resource "optimal://default:notes/..." -w default:notes
```

| Route | What it does |
|---|---|
| `POST /api/authority/decide` | Ask for a decision |
| `GET /api/authority/effective` | The rules that apply to a subject |
| `POST /api/authority/requests` | Ask a person to approve an action |
| `/api/authority/policies`, `/api/authority/grants` | Manage rules and time-limited grants |

Turn it on with `OPTIMAL_AUTHORITY_MODE`: `off` by default, `observe` to record decisions without blocking, or `enforce`.

### Organization policy

`PUT /api/org-section` sets rules once for the organization.
Every workspace reads them through `GET /api/org-policy`, cannot change them, and receives them in `boot` and `assemble`.

### Pre-send check

`POST /api/signals/check` (`optimal check`) checks a draft message against its receiver and channel, and suggests a shorter version.

## Records of work

| Route | What it does |
|---|---|
| `POST /api/agent-actions`, `GET /api/agent-actions/:id` | Record an agent's action, and read it with its linked evidence |
| `POST /api/sessions/commit` | Store a finished chat as an episode; what it learned goes to claim review |
| `GET /api/viability` | A health report on the workspace as an operating system, with alerts |

## Scheduled analyses and jobs

The Engine runs six analyses on a schedule: `promises`, `workspace_population`, `meeting_archive`, `file_audit`, `storage_usage`, and `retention_sweep`.

| Route | What it does |
|---|---|
| `GET /api/analyses` | Their reports |
| `POST /api/analyses/run` | Run one now (`admin`) |
| `GET /api/jobs` | The job queue |
| `GET /api/jobs/dead`, `POST /api/jobs/dead/:id/replay` | Failed jobs, and retrying one |

## Webhooks and live updates

A subscription says what an agent or person wants pushed to them: anything in a workspace, one node, a topic, or one audience.
Create one with `POST /api/subscriptions`; it can be paused, resumed, and deleted.
The Engine pushes matching items over `GET /api/surface/stream`, a server-sent event stream, and to a webhook when the subscription's `metadata` has a `webhook_url`.
`POST /api/subscriptions/:id/test-webhook` sends a test delivery.

> **Note: Workflows and skill packages**
>
> The Engine records workflow traces and can promote repeated ones into skill packages (`mix optimal.skills`).
> Those are records for review and retrieval.
> The Engine does not run a skill package as code, and it has no agent registry or delegation between agents.
