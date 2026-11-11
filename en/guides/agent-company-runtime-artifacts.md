---
slug: "guides/agent-company-runtime-artifacts"
title: "Runtime profiles and artifacts"
description: "The end-to-end lane from runtime profile to artifact export, the prompt-to-device flow, how env and secrets are inherited, and how agent outputs become downloads."
section: "Agents"
order: 6022
---

This page covers the runtime mechanics of an agent product: how a run starts from a runtime profile, how a prompt reaches a device, how environment and secrets are inherited, and how generated files become user-facing downloads.

## E2E runtime profile to artifact export

This is the concrete end-to-end lane to implement first for a Polsia, Cofounder, Nebula, Heuresis, BusinessOS, Lovable, Replit, Genspark, Emergent, or Blackbox AI-style product:

```text
create runtime profile
  -> create or resume sandbox/computer
  -> dispatch Agent Run with profile id
  -> agent writes files in /workspace
  -> export approved artifact paths
  -> store download URL on your product run record
```

**CLI**

```bash
PROFILE_ID=$(miosa api POST /agent-runtime-profiles -d '{
  "workspace_id": "ws_123",
  "name": "Default app builder",
  "runtime": "claude-code",
  "tools": ["files", "exec", "previews", "artifacts"],
  "connectors": ["anthropic/workspace-claude"],
  "env": {"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
  "is_default": true
}' | jq -r '.data.id')

SANDBOX_ID=$(miosa create builder-box \
  --template nextjs \
  --wait \
  --json | jq -r '.id // .data.id')

miosa prompt \
  --sandbox "$SANDBOX_ID" \
  --harness claude-code \
  "Build a lead-capture page, run tests, and write the final bundle to /workspace/dist."

miosa files cp -r "$SANDBOX_ID":/workspace/dist ./lead-capture --json
```

**TypeScript**

```typescript
const profile = await miosa.agentRuntimeProfiles.create({
  workspaceId: "ws_123",
  name: "Default app builder",
  runtime: "claude-code",
  tools: ["files", "exec", "previews", "artifacts"],
  connectors: ["anthropic/workspace-claude"],
  env: { ANTHROPIC_MODEL: "claude-sonnet-4.6" },
  isDefault: true,
})

const sandbox = await miosa.sandboxes.create({
  templateId: "nextjs",
  agentRuntimeProfileId: profile.id,
})

const run = await miosa.agentRuns.run({
  sandboxId: sandbox.id,
  agentRuntimeProfileId: profile.id,
  provider: "claude-code",
  cwd: "/workspace",
  prompt:
    "Build a lead-capture page, run tests, and write the final bundle to /workspace/dist.",
  metadata: { productRunId: "run_123" },
})

const exportInfo = await sandbox.createExport({
  paths: ["/workspace/dist"],
  label: "lead-capture-build",
})

await saveProductRunArtifact(run.id, exportInfo.archiveDownloadUrl)
```

**Python**

```python
profile = client.agent_runtime_profiles.create(
    workspace_id="ws_123",
    name="Default app builder",
    runtime="claude-code",
    tools=["files", "exec", "previews", "artifacts"],
    connectors=["anthropic/workspace-claude"],
    env={"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
    is_default=True,
)

sandbox = client.sandboxes.create(
    template_id="nextjs",
    agent_runtime_profile_id=profile["id"],
)

run = client.agent_runs.run(
    sandbox_id=sandbox.id,
    agent_runtime_profile_id=profile["id"],
    provider="claude-code",
    cwd="/workspace",
    prompt=(
        "Build a lead-capture page, run tests, and write the final bundle "
        "to /workspace/dist."
    ),
    metadata={"product_run_id": "run_123"},
)

export_info = sandbox.create_export(
    ["/workspace/dist"],
    label="lead-capture-build",
)

save_product_run_artifact(run["id"], export_info["archive_download_url"])
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/agent-runtime-profiles \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace_id": "ws_123",
    "name": "Default app builder",
    "runtime": "claude-code",
    "tools": ["files", "exec", "previews", "artifacts"],
    "connectors": ["anthropic/workspace-claude"],
    "is_default": true
  }'

curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id":"nextjs","agent_runtime_profile_id":"arp_123"}'

curl -X POST https://api.miosa.ai/api/v1/agent-runs \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sandbox_id": "sbx_123",
    "agent_runtime_profile_id": "arp_123",
    "provider": "claude-code",
    "cwd": "/workspace",
    "prompt": "Build a lead-capture page, run tests, and write the final bundle to /workspace/dist."
  }'

curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_123/exports \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"paths":["/workspace/dist"],"label":"lead-capture-build"}'

curl "https://api.miosa.ai/api/v1/sandboxes/sbx_123/exports/download?paths[]=/workspace/dist" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output lead-capture.tar.gz
```

Use a sandbox for generated files, reports, code, previews, and app bundles. Add
a computer step only when the run needs browser interaction, screenshots, or
visual QA. Use [Files, exec, and artifacts](/docs/develop/files-and-exec) to
promote approved paths into downloads, and [Agent Runtime Profiles](/docs/api-reference/agent-runtime-profiles)
for the reusable profile contract.

---

## Prompt-to-device flow

Use one product-level run model whether the user is building an app, generating
a report, or asking an operator to complete a business task.

```text
prompt
  -> resolve workspace and agent
  -> select runtime profile
  -> select sandbox, computer, or BYOC/OpenComputer target
  -> inject scoped env and secrets
  -> stream events while the device works
  -> save files, screenshots, artifacts, preview URLs, and deploy outputs
```

For Lovable, Replit, Genspark, Emergent, or Blackbox AI-style app builders, the default target should be a
persistent sandbox. The agent writes files into `/workspace`, runs install and
build commands there, starts a preview server, fixes errors in place, exports
deliverables, and publishes only after the user or browser QA approves the
result.

For Polsia or cofounder.co-style company operators, the same flow starts from a
business task instead of an app prompt. The orchestrator may use a sandbox for
documents and code, a computer for browser work, and BYOC/OpenComputer when the
task must run on customer-owned infrastructure.

## Runtime profiles and env inheritance

Runtime profiles keep the agent harness separate from the device. A profile
answers: which runtime starts, which tools are allowed, which non-secret env
defaults are present, and which connectors or secret groups can be requested.

Recommended resolution order:

| Source | Purpose |
|---|---|
| Tenant default profile | Platform-wide fallback for new workspaces |
| Workspace default profile | Customer/company-specific runtime choice |
| Agent profile | Role-specific tools, model, instructions, and connectors |
| Run override | One task's prompt, cwd, timeout, and non-secret env overrides |

Environment and secrets should be inherited narrowly:

| Value type | Inheritance rule |
|---|---|
| Non-secret env | Tenant defaults -> workspace defaults -> profile env -> run env |
| MIOSA runtime token | Minted per run or session with workspace/user/run attribution |
| Provider secrets | Resolved through managed secrets or connectors, never copied into chat |
| Browser-visible config | Only short-lived preview/share tokens and public config |
| BYOC/OpenComputer host secrets | Stay on the customer-owned host unless explicitly brokered |

If a value can spend money, access customer data, publish externally, or call a
provider API, treat it as a secret or connector grant instead of plain profile
env.

## Artifacts and downloads

Every agent run should end with named outputs, not just terminal text. Use
exports for files users should download and store the returned descriptor on
your product's run, task, or project record.

| Output | Device that produces it | User-facing surface |
|---|---|---|
| Generated app | Sandbox | Preview URL, deployment URL, source ZIP |
| PDF/DOCX/report | Sandbox | Artifact card with download button |
| Screenshot set | Computer | QA evidence and downloadable images |
| Research bundle | Sandbox | Markdown/CSV/ZIP artifact |
| Customer-local result | BYOC/OpenComputer | Host-side file plus brokered export if approved |

See [Files, exec, and artifacts](/docs/develop/files-and-exec) for the export
API shape.

> **Note: Related**
>
> [Build an agent-company platform](/docs/guides/build-agent-company) covers the orchestrator, runtime, and device modules this lane plugs into, and [Agent orchestration](/docs/guides/agent-orchestration) covers running many of these at once.
