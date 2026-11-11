---
slug: "guides/configure-agent"
title: "Configure an agent"
description: "Configure an agent's identity, instructions, model, devices, tools, memory, connected accounts, triggers, and approval policy."
section: "Agents"
order: 6005
---

An agent is a named worker with instructions, tools, devices, memory, connected
accounts, and approval rules. The goal is to make the agent predictable enough
to trust, but capable enough to do real work inside MIOSA sandboxes and
computers.

---

## Configuration checklist

| Setting | Why it matters |
|---|---|
| Name and description | Helps users and orchestrators route work to the right agent |
| Visibility | Private, workspace, or platform-managed |
| Instructions | Role, rules, output style, safety limits, and domain context |
| Model or runtime | Which model, harness, or custom runtime powers the loop |
| Default device | Where work happens by default: sandbox, computer, or local device |
| Tools | Files, exec, browser, artifacts, deployments, app actions |
| Connected accounts | OAuth grants, managed connectors, API keys, scoped variables |
| Memory | Durable facts the agent can reuse across threads or jobs |
| Triggers | Schedules, webhooks, app events, or user actions |
| Approval policy | Actions that require human confirmation |
| Quotas | Limits for spend, child devices, runtime duration, and concurrency |

---

## Identity

Use a focused name and description.

Good:

```text
Name: Landing Page Builder
Description: Builds, previews, and publishes branded Next.js landing pages.
```

Weak:

```text
Name: Helper
Description: Does stuff.
```

Agents are easier to delegate to when their job is specific.

---

## Visibility

| Mode | Who can use it | Best for |
|---|---|---|
| Private | The creator only | Personal experiments, sensitive research |
| Workspace | Members with access | Shared operators, support, marketing, development |
| Platform-managed | Controlled by your product | Built-in agents inside a white-label SaaS |

Workspace agents should never silently inherit every personal connection in the
workspace. Attach only the accounts and tools they need.

---

## Instructions

Instructions should include:

- the agent's role
- the expected output
- where work must happen
- what tools it may use
- what requires approval
- what counts as done

Example:

```text
You are a Next.js build agent.
Work only inside the assigned MIOSA sandbox at /workspace.
Write files directly into the sandbox.
Run npm install, npm run lint, and npm run build before returning.
Expose port 3000 for preview.
Create artifacts for screenshots and generated reports.
Ask for approval before publishing or creating extra child sandboxes.
```

> **Tip: Make the runtime explicit**
>
> If the agent should code inside a sandbox, say that directly. This prevents the
> bad pattern where an agent builds locally and then tries to upload a finished
> repo after the fact.

---

## Model and runtime

MIOSA can support several runtime shapes:

| Runtime | Use it for |
|---|---|
| OSA | MIOSA-native agent/runtime operations |
| Codex | Code generation and repository maintenance workflows |
| Claude Code | Agentic coding loops with tools and MCP |
| Pi | Compatible harness-based agent execution |
| Hermes | Custom or vendor-provided agent runtime |
| Custom | Bring your own loop, queue, prompt format, or model provider |

The runtime should be a configuration value, not hard-coded into product logic.
Your product can choose the default runtime per agent type and still allow
advanced users to supply custom runtimes later.

### Runtime profiles

For white-label products, configure default runtimes at the tenant or workspace
level instead of repeating the same connector and env setup for every sandbox.

```bash
miosa api POST /agent-runtime-profiles -d '{
  "name": "Default app builder",
  "runtime": "claude-code",
  "tools": ["files", "exec", "previews", "artifacts"],
  "connectors": ["refero/design-research", "anthropic/workspace-claude"],
  "policy": {"approval": "confirm-risky", "max_child_sandboxes": 5},
  "is_default": true
}'
```

```typescript
await miosa.agentRuntimeProfiles.create({
  name: "Default app builder",
  runtime: "claude-code",
  tools: ["files", "exec", "previews", "artifacts"],
  connectors: ["refero/design-research", "anthropic/workspace-claude"],
  policy: { approval: "confirm-risky", max_child_sandboxes: 5 },
  isDefault: true,
})
```

```python
client.agent_runtime_profiles.create(
    name="Default app builder",
    runtime="claude-code",
    tools=["files", "exec", "previews", "artifacts"],
    connectors=["refero/design-research", "anthropic/workspace-claude"],
    policy={"approval": "confirm-risky", "max_child_sandboxes": 5},
    isDefault=True,
)
```

Use profiles for defaults. Use `miosa prompt` or the Agent Runs API for
individual task dispatch.

When a default profile exists, every new sandbox or cloud computer in that
tenant/workspace gets the profile automatically unless the caller opts out.
MIOSA applies the profile before the runtime is inserted:

| Profile field | Sandbox behavior | Computer behavior |
|---|---|---|
| `runtime` | Stored in sandbox metadata so orchestrators know which agent harness to start | Stored in computer settings for orchestration and dispatch |
| `tools` | Stored in metadata as the expected tool bundle | Stored in settings as the expected tool bundle |
| `env` | Merged into sandbox env at boot; request-level env wins | Encrypted in the computer env store and synced after OSA is healthy |
| `connectors` | Bound through MIOSA egress as opaque placeholder tokens | Bound through MIOSA egress as opaque placeholder tokens |
| `policy` | Stored in metadata for approval, child-device, and quota decisions | Stored in settings for approval, child-device, and quota decisions |

```bash
# Uses the workspace or tenant default profile automatically.
miosa create builder --template nextjs --ttl 1h --wait

# Force a specific profile for this sandbox (API):
miosa api POST /sandboxes -d '{
  "name": "builder",
  "template": "nextjs",
  "agent_runtime_profile_id": "9f4b3d8e-0000-0000-0000-000000000000"
}'

# Create a browser/desktop computer with the workspace default profile.
miosa computer create browser-worker
```

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "nextjs",
  agentRuntimeProfileId: "9f4b3d8e-0000-0000-0000-000000000000",
})

const computer = await miosa.computers.create({
  name: "browser-worker",
  agentRuntimeProfileId: "9f4b3d8e-0000-0000-0000-000000000000",
})
```

```python
sandbox = client.sandboxes.create(
    template_id="nextjs",
    agent_runtime_profile_id="9f4b3d8e-0000-0000-0000-000000000000",
)

computer = client.computers.create(
    name="browser-worker",
    agent_runtime_profile_id="9f4b3d8e-0000-0000-0000-000000000000",
)
```

Connector secrets are not copied into metadata or returned to the agent. The
runtime sees env vars such as `REFERO_MCP_TOKEN` or `ANTHROPIC_API_KEY`, but
those values are MIOSA egress placeholders (`miosa-tok-*`). The proxy swaps the
placeholder for the provider credential only when the request is allowed.

---

## Default device

Every agent should have a default device policy.

| Policy | Behavior |
|---|---|
| Sandbox-first | Create or resume a sandbox for code, files, commands, previews, and artifacts |
| Computer-first | Use a persistent desktop/browser computer for GUI and web tasks |
| Local-first | Use the paired user machine when local files or private network access is needed |
| Orchestrator | Start in one root device and launch child devices for specialized work |

For app builders, sandbox-first should be the default. For browser and RPA
agents, computer-first is usually correct.

---

## Tools

Attach only the toolkits the agent needs.

Common bundles:

| Bundle | Includes |
|---|---|
| Build app | Files, exec, live preview, snapshots, artifacts, publish |
| Browser QA | Browser, screenshots, clicks, form fill, preview access |
| Research | Web search, page extraction, Refero design research, note files |
| Artifact factory | Files, exec, media conversion, artifacts, version history |
| Integrations | Connected accounts, webhooks, app actions |
| Operator | Computer, browser, terminal, long-running jobs |

---

## Accounts and variables

Use account connections and variables instead of chat-pasted secrets.

| Type | Example | How it is exposed |
|---|---|---|
| OAuth account | GitHub, Google, Slack | Short-lived token or egress-brokered placeholder |
| API key | Anthropic, OpenAI, Stripe | Scoped env var |
| Managed connector | Refero design research | MIOSA-owned token, brokered to provider only |
| Tenant variable | `NEXT_PUBLIC_BRAND_NAME` | Plain env var |
| Runtime token | `MIOSA_API_KEY` | Scoped token for one workspace, user, project, or run |

Never paste a secret into an agent conversation. Store it through the dashboard,
API, CLI, or connector flow.

---

## Memory

Memory is durable context the agent can reuse. Keep it scoped.

Good memories:

- brand colors
- project routes
- client preferences
- deployment URLs
- app architecture notes
- recurring task rules

Avoid saving:

- raw secrets
- one-time OTPs
- irrelevant chat history
- stale debugging guesses

Memory should be editable and revocable by workspace owners.

---

## Triggers and jobs

Agents can run manually or automatically.

| Trigger | Example |
|---|---|
| Manual | User asks the agent to build a landing page |
| Schedule | Run a daily competitor scrape |
| Webhook | New GitHub issue creates a triage task |
| App event | New CRM lead generates a proposal |
| Runtime event | Failed deploy creates a fix task |

Triggered agents should write run records, stream events, and respect quotas.

---

## Approval policy

Require approval for risky actions:

- publishing to production
- creating extra child devices above a threshold
- sending messages externally
- writing to connected apps
- spending credits above a configured budget
- deleting files, snapshots, deployments, or data
- using high-privilege credentials

Routine reads, file writes inside the assigned workspace, tests, and preview
starts can usually be automatic.

---

## Related

- [What agents can do](/docs/guides/agent-capabilities): Tool catalog, permissions model, and product surfaces.
- [Agent devices](/docs/guides/agent-devices): Choose the right runtime for code, browser, desktop, local network, or deploy.
- [Agent orchestration](/docs/guides/agent-orchestration): Coordinate multiple agents, queues, device pools, and sub-agents.
- [Managed connectors](/docs/guides/managed-connectors): Give agents provider tools without exposing provider credentials.
