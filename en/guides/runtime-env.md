---
slug: "guides/runtime-env"
title: "Runtime env for agent devices"
description: "Define inherited environment variables for sandboxes, computers, and agent runtimes at tenant, workspace, or project scope."
section: "Agents"
order: 6013
---

Runtime env lets a platform owner define environment variables once and have
them inherited by sandboxes, computers, and agent runs. Use it for provider keys,
model configuration, MCP tokens, workspace defaults, and white-label customer
settings that should be available automatically when a device starts or runs a
command.

This is the foundation for products where users create many agent workspaces:
app builders, browser operators, 24/7 company agents, research agents, and
customer-specific automation fleets.

> **Note: Responses are redacted**
>
> Runtime env values are encrypted at rest and are never returned as plaintext by
> the API, CLI, or SDKs. Responses include metadata and a short preview only.

## Scope model

MIOSA resolves runtime env in this order:

| Precedence | Scope | Applies to |
|---:|---|---|
| 1 | Tenant | Every workspace, project, sandbox, computer, and agent under the tenant |
| 2 | Workspace | Every project, sandbox, computer, and agent in that workspace |
| 3 | Project | Sandboxes, computers, deployments, and agent runs attributed to that project |
| 4 | Resource | Per-sandbox or per-computer env set directly on the resource |
| 5 | Run | Env passed directly to a single exec or agent run |

More specific values override broader values with the same name. A project value
overrides a workspace value; a workspace value overrides a tenant value.

## Target model

Each runtime env var also has a target:

| Target | Use it when |
|---|---|
| `all` | The value is safe and useful for every runtime surface |
| `sandbox` | The value should only appear in sandbox exec/build/agent sessions |
| `computer` | The value should only appear in desktop computer commands and agents |
| `agent` | The value should be injected into prompt-driven agent runs on sandboxes or computers |

At the same scope, target-specific values override `all`.

## Common patterns

- **White-label default**: Set `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, or product feature flags at
  workspace scope for one client workspace.
- **Project provider key**: Set `HIGGSFIELD_API_KEY` or image model keys at project scope for one app or customer workflow.
- **Agent runtime config**: Set `ANTHROPIC_MODEL`, `MIOSA_AGENT_CWD`, or MCP endpoint configuration for a class of agents.
- **Sandbox build secret**: Set deployment, registry, package manager, or private API credentials for generated builds.

## CLI

Runtime Env has no dedicated `miosa` subcommand; drive it with `miosa api`
against the `/runtime-env` endpoint. Set a tenant-wide sandbox value:

```bash
miosa api POST /runtime-env -d '{
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "scope": "tenant",
  "target": "sandbox"
}'
```

Set a workspace-wide computer value:

```bash
miosa api POST /runtime-env -d '{
  "name": "OPENAI_API_KEY",
  "value": "sk-...",
  "scope": "workspace",
  "workspace_id": "ws_123",
  "target": "computer"
}'
```

Set a project value for all runtime surfaces:

```bash
miosa api POST /runtime-env -d '{
  "name": "HIGGSFIELD_API_KEY",
  "value": "higgs_...",
  "scope": "project",
  "project_id": "prj_123",
  "target": "all"
}'
```

List, show and delete:

```bash
miosa api GET /runtime-env?scope=workspace&workspace_id=ws_123
miosa api GET /runtime-env/<env-id>
miosa api DELETE /runtime-env/<env-id>
```

Filter by target to see one surface's variables:

```bash
miosa api GET /runtime-env?target=sandbox
```

Run an agent with inherited `target=agent` env and download the file it creates:

```bash
miosa api POST /runtime-env -d '{
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "scope": "project",
  "project_id": "prj_123",
  "target": "agent"
}'

miosa agent run my-agent \
  --sandbox sbx_123 \
  --no-follow

miosa run files <run-id> --json
miosa run download <run-id> <file-id> --output ./report.html
```

## SDKs

**TypeScript**

```ts
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

await miosa.runtimeEnv.set({
  scope: "workspace",
  workspaceId: "ws_123",
  target: "sandbox",
  name: "ANTHROPIC_API_KEY",
  value: process.env.ANTHROPIC_API_KEY!,
  metadata: { provider: "anthropic" },
});

const vars = await miosa.runtimeEnv.list({
  scope: "workspace",
  workspaceId: "ws_123",
  target: "sandbox",
});
```

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

client.runtime_env.set(
    scope="workspace",
    workspace_id="ws_123",
    target="sandbox",
    name="ANTHROPIC_API_KEY",
    value=os.environ["ANTHROPIC_API_KEY"],
    metadata={"provider": "anthropic"},
)

vars = client.runtime_env.list(
    scope="workspace",
    workspace_id="ws_123",
    target="sandbox",
)
```

## Verify inside a sandbox

Create a sandbox, wait for the VM session, then read the inherited value:

```bash
miosa create env-check \
  --template miosa-sandbox \
  --ttl 1h \
  --wait \
  --json

miosa exec env-check \
  --json \
  -- printf "$ANTHROPIC_API_KEY"
```

Use `--port` only when you also want app preview readiness:

```bash
miosa wait <sandbox-id> --timeout 180s --json
```

## Verify inside a computer

Use computers when the agent needs a desktop, browser session, GUI automation,
or long-running state:

```bash
miosa computer create browser-agent --json
miosa device exec <computer-id> --command 'printf "$OPENAI_API_KEY"' --json
```

## Verify inside an agent run

Agent runs are prompt-driven tasks dispatched to a sandbox or computer. They
inherit `target=agent` env, then apply any per-run `env` overrides. The run
record stores command metadata, status, output, events, and declared artifact
paths.

**TypeScript**

```ts
const run = await miosa.agentRuns.run({
  sandboxId: "sbx_123",
  provider: "claude-code",
  cwd: "/workspace",
  prompt: "Create /workspace/report.html with the final proposal.",
  metadata: {
    artifact_paths: ["/workspace/report.html"],
  },
});

const artifacts = await miosa.agentRuns.artifacts(run.id);
const html = await miosa.agentRuns.downloadArtifact(run.id, artifacts[0].id);
```

**Python**

```python
run = client.agent_runs.run(
    sandbox_id="sbx_123",
    provider="claude-code",
    cwd="/workspace",
    prompt="Create /workspace/report.html with the final proposal.",
    metadata={"artifact_paths": ["/workspace/report.html"]},
)

artifacts = client.agent_runs.artifacts(run["id"])
html = client.agent_runs.download_artifact(run["id"], artifacts[0]["id"])
```

## Agent runtime profiles and connectors

Runtime env is one layer of the agent configuration stack:

| Layer | Purpose |
|---|---|
| Runtime env | Inherited env vars and provider keys by scope and target |
| Agent runtime profile | Runtime choice, non-secret defaults, tools, policy, connectors |
| Connector | Brokered provider identity or managed account |
| Egress | Network policy and brokered token exchange |
| Agent run | One prompt/task dispatched to a sandbox or computer |

Store provider keys in runtime env when the runtime expects a normal env var.
Use connectors when MIOSA should broker a provider grant or managed tool.

## Export generated work

When the agent creates files, download declared agent-run artifacts or export
raw sandbox paths instead of scraping terminal output:

```bash
miosa run files <run-id> --json
miosa run download <run-id> <file-id> --output ./report.pdf
```

```bash
miosa files cp <sandbox-id>:/workspace/out/report.pdf ./report.pdf --json

miosa files cp -r <sandbox-id>:/workspace/out ./customer-deliverables --json
```

For a new live Docker Host app, expose the development port and publish with an explicit production identity:

```bash
miosa wait <sandbox-id> --json
miosa deploy create --from-sandbox <sandbox-id> \
  --name my-app \
  --dir /workspace \
  --port 3000 \
  --type docker_deploy \
  --wait \
  --json
```

For later updates, publish to the existing deployment ID so the stable URL and release history remain attached to the same app:

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --dir /workspace \
  --port 3000 \
  --type docker_deploy \
  --wait \
  --json
```
