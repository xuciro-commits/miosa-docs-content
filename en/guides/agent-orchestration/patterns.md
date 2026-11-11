---
slug: "guides/agent-orchestration/patterns"
title: "Orchestration patterns"
description: "Coordination patterns for MIOSA agent systems: prompt dispatch, durable run groups, subagents, sandbox-hosted orchestrators, self-configuring agents, artifact export, credentials, and permissions."
section: "Agents"
order: 6023
---

These are the coordination patterns that make an agent system work as it grows: how a prompt reaches the right device, how a task fans out into durable run groups, in-sandbox and sandbox-hosted topology, how generated work leaves the runtime, and how credentials and permissions stay scoped.

---

## Prompt dispatch

At runtime, orchestration mostly means dispatching prompts into the right
device and streaming the result back.

| Prompt target | Use it for | Example command |
|---|---|---|
| Sandbox | code, files, tests, builds, artifacts, preview servers | `miosa prompt --sandbox <id> "build the page"` |
| Computer | browser, screenshots, clicks, form fill, visual QA | `miosa prompt --computer <id> "test signup"` |
| BYOC/OpenComputer | private local files, apps, network, customer-controlled hardware | See [OpenComputers](/docs/opencomputers) for host-scoped work |

Create reusable runtime profiles for the agents your product supports. A
profile can be tenant-wide or workspace-specific, and it can apply to sandbox
workers, computer agents, or both.

Prompt-to-device flow should be explicit in your product backend:

```text
user prompt
  -> product run record
  -> runtime profile resolution
  -> sandbox, computer, or BYOC/OpenComputer target
  -> scoped env and secret injection
  -> event stream
  -> files, artifacts, screenshots, previews, deployments
```

This lets Polsia-style operators, cofounder.co-style company operating systems,
Nebula-style virtual devices, and Lovable/Replit/Genspark-style app builders
share one UI contract even when they use different runtimes underneath.

### Durable run groups

Use an Agent Run Group when one product task fans out into multiple child runs.
The group gives your UI one durable record for counts, status, cancellation,
child run lookup, and a group event stream.

```text
user task
  -> agent run group
    -> child run: sandbox coder
    -> child run: sandbox tester
    -> child run: computer browser QA
    -> child run: deployment verifier
  -> artifacts, events, previews, screenshots, deployment URL
```

Current dispatch batches are capped at 100 child runs per API call. For hundreds
or thousands of logical agents, queue the work in your backend and dispatch in
batches against one or more groups. Keep a separate concurrency policy per
tenant/workspace so one customer cannot consume the whole runtime pool.

The user-facing loop should be:

```text
create group
  -> dispatch child runs
  -> stream /agent-run-groups/{id}/events
  -> show child run status and output as it happens
  -> list artifacts from completed child runs
  -> expose download/preview/deploy actions in your UI
```

This avoids opening one browser stream per child agent. Your product backend can
subscribe to one group stream, normalize events into your own task timeline, and
keep the UI live while sandbox and computer agents work in parallel.

When a child run uses a sandbox process-backed runtime, cancellation stops the
recorded sandbox process instead of only changing run status. Use this for
longer build, test, scrape, or artifact-generation tasks where the user may
cancel from your UI.

**CLI**

```bash
miosa api POST /agent-runtime-profiles -d '{
  "name": "lumen-claude",
  "runtime": "claude-code",
  "workspace_id": "lumen",
  "tools": ["filesystem", "shell", "browser"],
  "connectors": ["anthropic/workspace"],
  "env": {"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
  "is_default": true
}'

miosa agent run builder \
  --sandbox sbx_123 \
  --no-follow

miosa api POST /run-groups -d '{
  "name": "lumen-build-and-qa",
  "workspace_id": "ws_123",
  "concurrency_limit": 10,
  "expected_runs": 3
}'

miosa api POST /run-groups/grp_123/dispatch -d '{
  "runs": [
    {"sandbox_id": "sbx_code", "provider": "claude-code", "prompt": "Build the app", "orchestration_role": "coder"},
    {"sandbox_id": "sbx_test", "provider": "codex", "prompt": "Run tests", "orchestration_role": "tester"},
    {"computer_id": "cmp_browser", "provider": "osa", "prompt": "Verify signup in the browser", "orchestration_role": "browser-qa"}
  ],
  "async": true
}'

miosa api GET "/run-groups/grp_123/activity?stream=true&limit=100"
```

**TypeScript**

```typescript
const profile = await miosa.agentRuntimeProfiles.create({
  name: "lumen-claude",
  runtime: "claude-code",
  workspaceId: "ws_123",
  isDefault: true,
  appliesTo: { resources: ["sandbox", "computer"] },
  tools: ["filesystem", "shell", "browser"],
  connectors: ["anthropic/workspace"],
  env: { ANTHROPIC_MODEL: "claude-sonnet-4.6" },
})

const run = await miosa.agentRuns.run({
  sandboxId: "sbx_123",
  prompt: "Build the intake form and export the generated HTML.",
  agentRuntimeProfileId: profile.id,
  cwd: "/workspace",
})

const group = await miosa.agentRunGroups.create({
  name: "lumen-build-and-qa",
  workspaceId: "ws_123",
  concurrencyLimit: 10,
  expectedRuns: 3,
})

await miosa.agentRunGroups.dispatch(
  group.id,
  [
    {
      sandboxId: "sbx_code",
      provider: "claude-code",
      prompt: "Build the app",
      orchestrationRole: "coder",
    },
    {
      computerId: "cmp_browser",
      provider: "osa",
      prompt: "Verify signup in the browser",
      orchestrationRole: "browser-qa",
    },
  ],
  { async: true },
)

for await (const event of miosa.agentRunGroups.streamEvents(group.id)) {
  timeline.append({
    childRunId: event.agent_run_id,
    type: event.type,
    message: event.message,
  })
}
```

**Python**

```python
profile = client.agent_runtime_profiles.create(
    name="lumen-claude",
    runtime="claude-code",
    workspace_id="ws_123",
    is_default=True,
    applies_to={"resources": ["sandbox", "computer"]},
    tools=["filesystem", "shell", "browser"],
    connectors=["anthropic/workspace"],
    env={"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
)

run = client.agent_runs.run(
    sandbox_id="sbx_123",
    agent_runtime_profile_id=profile["id"],
    cwd="/workspace",
    prompt="Build the intake form and export the generated HTML.",
)

group = client.agent_run_groups.create(
    name="lumen-build-and-qa",
    workspace_id="ws_123",
    concurrency_limit=10,
    expected_runs=3,
)

client.agent_run_groups.dispatch(
    group["id"],
    [
        {
            "sandbox_id": "sbx_code",
            "provider": "claude-code",
            "prompt": "Build the app",
            "orchestration_role": "coder",
        },
        {
            "computer_id": "cmp_browser",
            "provider": "osa",
            "prompt": "Verify signup in the browser",
            "orchestration_role": "browser-qa",
        },
    ],
    async_=True,
)

for event in client.agent_run_groups.stream_events(group["id"]):
    timeline.append(
        {
            "child_run_id": event["agent_run_id"],
            "type": event["type"],
            "message": event.get("message"),
        }
    )
```

If no explicit profile is passed, MIOSA resolves the workspace default profile
first, then the tenant default profile. Request-level `env` values override
non-secret profile defaults. Secret provider keys should come from managed
secrets or connectors, not from profile `env`.

Recommended inheritance order:

| Layer | Inherits into the run | Should not contain |
|---|---|---|
| Tenant default profile | Organization-wide runtime, tool, and non-secret env defaults | Customer-specific provider keys |
| Workspace default profile | Company/customer runtime choice, cwd defaults, allowed tools | Tenant admin keys |
| Agent profile | Role instructions, connectors, approval policy, model hints | Browser-visible secrets |
| Run request | Prompt, cwd, timeout, non-secret env overrides | Long-lived provider credentials |

Use managed secrets or connectors for anything that can spend money, access
customer data, publish externally, or call a provider API. The run should
receive a scoped MIOSA runtime token with tenant/workspace/user/run attribution,
plus brokered provider access only for the endpoints the profile allows.

Every dispatch should record:

- prompt
- runtime or harness
- device id
- scoped credentials and connectors
- approval policy
- streamed events
- files, artifacts, screenshots, previews, or deployment outputs

The user experience should be the same whether the prompt is handled by OSA,
Codex, Claude Code, Hermes, OpenCode, OpenClaw, or a custom runtime.

---

## Subagents inside sandboxes

You can also run subagents inside a sandbox. This is useful when the product
wants a primary orchestrator to delegate work into an isolated workspace.

```text
Account / tenant
  -> workspace
    -> orchestrator service
      -> sandbox
        -> subagent A: code editor
        -> subagent B: test runner
        -> subagent C: artifact generator
        -> subagent D: deploy verifier
```

The sandbox receives the environment and scoped credentials it needs. The
subagents run inside `/workspace`, communicate through files, local processes,
stdout, and MIOSA events, and report results back to the orchestrator.

Good uses for sandbox-hosted subagents:

- parallel code review agents inside one repo
- test generation plus test execution
- report generation workers
- artifact conversion workers
- dependency install/build/lint/test pipelines

Avoid using one sandbox as an uncontrolled multi-tenant process host. A sandbox
should belong to one tenant/workspace/security boundary.

---

---

## Sandbox-hosted orchestrators

The orchestrator does not always have to live in your backend. A MIOSA sandbox
can be the root orchestrator workspace.

```text
Tenant account
  -> workspace
    -> root orchestrator sandbox
      -> orchestration config
      -> local subagents
      -> child MIOSA sandboxes
      -> child MIOSA computers
      -> artifacts
      -> previews
      -> deployments
```

This pattern is useful for autonomous coding systems, research systems,
artifact factories, and customer-owned agent runtimes. The root sandbox keeps
the project files, agent configuration, run history, and local tooling. The
orchestrator process inside that sandbox can call MIOSA to launch additional
sandboxes or computers when it needs isolated workers.

Good examples of this pattern:

- root sandbox runs an agent manager
- child sandbox builds a web app
- child sandbox runs tests in parallel
- child computer opens the preview in a browser
- root sandbox collects screenshots, logs, and artifacts
- deploy agent publishes the verified result

The backend still owns tenant enforcement, billing, quotas, audit logging, and
token minting. The sandbox-hosted orchestrator owns the local plan and execution
strategy for that workspace.

---

---

## Self-configuring agents

A sandbox-hosted orchestrator can customize its own setup. It can write config
files, install packages, create worker scripts, generate task definitions, and
adjust its local workflow as the project evolves.

Typical files inside `/workspace`:

```text
/workspace
  agent.config.json
  miosa.runbook.md
  tasks/
    build.json
    test.json
    browser-qa.json
  agents/
    coder.ts
    tester.ts
    artifact-generator.py
  artifacts/
  logs/
```

Example `agent.config.json`:

```json
{
  "workspace": "lumen-demo",
  "defaultTemplate": "nextjs",
  "workers": {
    "coder": { "runtime": "local", "tools": ["files", "exec"] },
    "tester": { "runtime": "child-sandbox", "template": "miosa-sandbox" },
    "browserQa": { "runtime": "computer", "template": "ubuntu-browser" }
  },
  "limits": {
    "maxConcurrentWorkers": 8,
    "maxChildSandboxes": 4,
    "maxChildComputers": 1
  }
}
```

This lets an agent improve its own operating procedure without requiring every
change to be hard-coded in the parent application.

Use these guardrails:

- keep config files visible in `/workspace`
- require explicit approval for quota or permission increases
- validate generated config before applying it
- cap child sandbox/computer creation
- write run events back to the product UI
- snapshot before major self-modification
- keep tenant admin credentials outside the sandbox

The goal is controlled autonomy: the agent can code and configure its own
workspace, but MIOSA still enforces account boundaries and resource limits.

---

---

## Exporting generated work

Agents should leave concrete outputs behind. For app builders and agent-company
products, those outputs become the user's deliverable: HTML, PDFs, DOCX files,
CSV exports, screenshots, ZIPs, source patches, build logs, or deployment URLs.

Use declared artifact paths on Agent Runs when you expect a generated file:

```json
{
  "sandbox_id": "sbx_code",
  "provider": "claude-code",
  "prompt": "Build the landing page and export /workspace/artifacts/site.html",
  "metadata": {
    "artifact_paths": ["/workspace/artifacts/site.html"]
  }
}
```

When the run completes, list and download artifacts:

```bash
miosa run files <run-id> --json
miosa run download <run-id> <file-id> --output ./site.html
```

MIOSA captures artifact bytes into managed storage when possible. If an
artifact has `persisted: true`, your UI can keep showing a download button after
the runtime stops. If it is not persisted, keep the sandbox or computer running
until the file has been downloaded, published, or copied somewhere durable.

Use this split in your product UI:

| Output type | Best surface |
|---|---|
| HTML, PDF, DOCX, CSV, ZIP | Artifact download |
| Web app | Sandbox preview, then deployment |
| Screenshot or visual QA result | Artifact preview plus event timeline |
| Source code changes | Files panel, git diff, artifact ZIP |
| Long-running product | Deployment or Docker Host URL |

---

---

## Credentials and account connection

Large agent systems should use scoped credentials instead of sharing one tenant
admin key everywhere.

```text
Tenant key
  -> server-side orchestrator only
  -> mints scoped workspace/user tokens
  -> tokens injected into sandbox/computer runtime
  -> subagents call MIOSA within their allowed scope
```

Typical environment injected into a runtime:

```bash
MIOSA_API_KEY            # a scoped key from the console
MIOSA_TENANT_ID=...
MIOSA_WORKSPACE_ID=...
MIOSA_PROJECT_ID=...
MIOSA_USER_ID=...
MIOSA_RUN_ID=...
```

This lets every subagent be connected to the correct account while still giving
the platform auditability, quotas, and revocation.

Important credential rules:

- Browser code never receives tenant admin keys.
- Sandboxes receive scoped runtime keys, not root account keys.
- Every token has workspace/project/user/run attribution.
- Every command, file write, preview, snapshot, and deploy is auditable.
- Quotas apply at tenant, workspace, user, and runtime-pool levels.

---

---

## Permissions and safety

Each agent should receive only the permissions it needs.

| Agent | Typical permissions |
|---|---|
| Sandbox code agent | sandbox files, exec, previews, snapshots |
| Artifact agent | sandbox files, exec, artifact export |
| Browser QA agent | computer screenshot, click, type, navigation |
| Deploy agent | deployment publish, domains, rollback |
| Billing/support agent | usage, credits, audit log |

Use scoped workspace keys or server-side tokens. Do not put tenant admin keys in
browser code.

---

---

## See also

- [Agent orchestration](/docs/guides/agent-orchestration): The concepts: scale model, runtime selection, communication primitives, and the queue and pool architecture.
- [Orchestration examples](/docs/guides/agent-orchestration/examples): Worked flows for app builders, sandbox-first code agents, and computer QA agents.
