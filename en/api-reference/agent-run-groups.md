---
slug: "api-reference/agent-run-groups"
title: "Agent Run Groups"
description: "Create durable multi-agent orchestration groups, dispatch run manifests, inspect counts, and cancel grouped work."
section: "API reference"
order: 12035
---

Agent Run Groups are the durable control-plane object for multi-agent fanout.
Use a group when one user request creates many related Agent Runs across
sandboxes, computers, or custom runtimes.

Base path: `/api/v1/agent-run-groups`

> **Note: Queued dispatch is available**
>
> Use synchronous dispatch for small fanout where you want child run creation in
> the response. Use `async: true` for queued fanout: MIOSA stores each manifest
> entry, enforces the group's concurrency limit while workers claim entries, and
> returns immediately with queued entry records.

## Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/v1/agent-run-groups` | List groups for the tenant |
| `POST` | `/api/v1/agent-run-groups` | Create a group |
| `GET` | `/api/v1/agent-run-groups/{id}` | Fetch one group |
| `POST` | `/api/v1/agent-run-groups/{id}/dispatch` | Dispatch a bounded manifest of child Agent Runs |
| `GET` | `/api/v1/agent-run-groups/{id}/events` | Replay or stream durable events from every child run in the group |
| `POST` | `/api/v1/agent-run-groups/{id}/cancel` | Cancel the group and running children |

## Create

```http
POST /api/v1/agent-run-groups
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "name": "lumen-browser-checks",
  "workspace_id": "ws_123",
  "project_id": "proj_123",
  "concurrency_limit": 25,
  "expected_runs": 100,
  "metadata": {
    "user_task_id": "task_123"
  }
}
```

## Dispatch

Dispatch accepts a `runs` array. Each entry uses the same fields as
[Agent Runs](/docs/api-reference/agent-runs), plus optional
`orchestration_role` and `parent_agent_run_id`.

```http
POST /api/v1/agent-run-groups/{id}/dispatch
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "runs": [
    {
      "sandbox_id": "sbx_code_1",
      "provider": "claude-code",
      "cwd": "/workspace",
      "prompt": "Implement the pricing page and export /workspace/report.html",
      "orchestration_role": "coder",
      "metadata": {
        "artifact_paths": ["/workspace/report.html"]
      }
    },
    {
      "computer_id": "cmp_browser_1",
      "provider": "osa",
      "prompt": "Open the preview, test signup, and capture screenshots.",
      "orchestration_role": "browser-qa"
    }
  ],
  "async": true
}
```

Set `async` to `true` when the group should queue entries and return before the
child Agent Runs are created. MIOSA creates one durable entry per manifest item,
then dispatcher workers claim entries under the group's `concurrency_limit`.

Current dispatch batches are capped at `100` entries per request. For larger
fleets, shard logical work into multiple groups or dispatch batches from your
backend.

## Response

```json
{
  "data": {
    "group": {
      "id": "grp_123",
      "name": "lumen-browser-checks",
      "status": "running",
      "concurrency_limit": 25,
      "expected_runs": 100,
      "counts": {
        "total": 2,
        "running": 0,
        "succeeded": 2,
        "failed": 0,
        "canceled": 0
      },
      "entry_counts": {
        "total": 2,
        "queued": 0,
        "running": 0,
        "succeeded": 2,
        "failed": 0,
        "canceled": 0
      }
    },
    "results": [
      {
        "index": 0,
        "ok": true,
        "run": {
          "id": "run_123",
          "agent_run_group_id": "grp_123",
          "orchestration_role": "coder",
          "status": "succeeded"
        }
      }
    ],
    "entries": []
  }
}
```

Async dispatch returns `entries` immediately and usually returns an empty
`results` array:

```json
{
  "data": {
    "group": {
      "id": "grp_123",
      "status": "running",
      "concurrency_limit": 25,
      "entry_counts": {
        "total": 2,
        "queued": 2,
        "running": 0,
        "succeeded": 0,
        "failed": 0,
        "canceled": 0
      }
    },
    "results": [],
    "entries": [
      {
        "id": "agre_123",
        "agent_run_group_id": "grp_123",
        "index": 0,
        "status": "queued",
        "attempts": 0
      }
    ]
  }
}
```

## Events

Use group events when your UI needs one stream for an entire multi-agent task.
MIOSA records child Agent Run events and replays them through the group in
sequence order. This lets your product render progress from the coder, tester,
browser QA agent, and deployment verifier without opening one stream per child
run.

Replay existing events as JSON:

```http
GET /api/v1/agent-run-groups/{id}/events
Authorization: Bearer msk_u_...
```

Stream existing events plus live events with SSE:

```http
GET /api/v1/agent-run-groups/{id}/events?stream=true
Accept: text/event-stream
Authorization: Bearer msk_u_...
```

Event payloads include the group id, child run id, event sequence, event type,
message, payload, and timestamp:

```json
{
  "data": [
    {
      "id": "ev_123",
      "agent_run_group_id": "grp_123",
      "agent_run_id": "run_123",
      "sequence": 4,
      "type": "command_finished",
      "message": "Command completed",
      "payload": {
        "exit_code": 0
      },
      "created_at": "2026-06-15T21:20:00Z"
    }
  ]
}
```

Terminal group states close the SSE response after replay. Running groups keep
the stream open and emit heartbeat comments while waiting for new child events.

## SDK

**TypeScript**

```typescript
const group = await miosa.agentRunGroups.create({
  name: "lumen-browser-checks",
  workspaceId: "ws_123",
  concurrencyLimit: 25,
  expectedRuns: 100,
});

const result = await miosa.agentRunGroups.dispatch(
  group.id,
  [
    {
      sandboxId: "sbx_code_1",
      provider: "claude-code",
      cwd: "/workspace",
      prompt: "Implement the pricing page and export /workspace/report.html",
      orchestrationRole: "coder",
      metadata: { artifact_paths: ["/workspace/report.html"] },
    },
  ],
  { async: true },
);

const withRuns = await miosa.agentRunGroups.get(group.id, { includeRuns: true });

const events = await miosa.agentRunGroups.events(group.id);

for await (const event of miosa.agentRunGroups.streamEvents(group.id)) {
  console.log(event.agent_run_id, event.type, event.message);
}
```

**Python**

```python
group = client.agent_run_groups.create(
    name="lumen-browser-checks",
    workspace_id="ws_123",
    concurrency_limit=25,
    expected_runs=100,
)

result = client.agent_run_groups.dispatch(
    group["id"],
    [
        {
            "sandbox_id": "sbx_code_1",
            "provider": "claude-code",
            "cwd": "/workspace",
            "prompt": "Implement the pricing page and export /workspace/report.html",
            "orchestration_role": "coder",
            "metadata": {"artifact_paths": ["/workspace/report.html"]},
        }
    ],
    async_=True,
)

with_runs = client.agent_run_groups.get(group["id"], include_runs=True)

events = client.agent_run_groups.events(group["id"])

for event in client.agent_run_groups.stream_events(group["id"]):
    print(event["agent_run_id"], event["type"], event.get("message"))
```

## CLI

```bash
miosa api POST /run-groups -d '{
  "name": "lumen-browser-checks",
  "workspace_id": "ws_123",
  "concurrency_limit": 25,
  "expected_runs": 100
}'

miosa api POST /run-groups/grp_123/dispatch -d '{
  "runs": [
    {
      "sandbox_id": "sbx_code_1",
      "provider": "claude-code",
      "prompt": "Build and export report",
      "metadata": {"artifact_paths": ["/workspace/report.html"]}
    }
  ],
  "async": true
}'

miosa api GET /run-groups/grp_123?include=runs
miosa api GET /run-groups/grp_123/activity
miosa api GET "/run-groups/grp_123/activity?stream=true&limit=100"
miosa api POST /run-groups/grp_123/cancel
```

## Next

- [Agent Runs](/docs/api-reference/agent-runs): Dispatch one prompt into a sandbox or computer and collect events and artifacts.
- [Agent orchestration](/docs/guides/agent-orchestration): Design the product backend that assigns devices, streams progress, and stores outputs.
- [Runtime env](/docs/api-reference/runtime-env): Configure inherited environment variables for agents and deployments.
