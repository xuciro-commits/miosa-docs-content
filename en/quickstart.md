---
slug: "quickstart"
title: "Quickstart"
description: "Install the CLI, start a sandbox, let an agent build something inside it, and open it at a live URL."
section: "Get started"
order: 1
---

MIOSA runs isolated Linux sandboxes and full cloud desktops for your code and your AI agents, then publishes the result to a live URL.
This page takes you from install to an app that an agent built, in four steps.

Video: [Quickstart, explained](https://miosa.ai/videos/quickstart-explained.mp4) (1:35)

## Let your agent drive MIOSA

Most people run MIOSA through a coding agent, not by hand.
Your agent can run on your laptop and control the whole platform.
It has four ways in:

| Way in | What to do |
|---|---|
| CLI | Your agent runs `miosa` commands. Start it with the [Agent operating guide](/docs/guides/agent-operating-guide). |
| MCP server | `miosa mcp install --client claude` (also `cursor` and `gemini`). OSA and Hermes use a config file instead: see [MCP](/docs/mcp). |
| Skills | Claude Code skills for sandboxes, computers, and deploys. See [Claude Code skills](/docs/guides/skills). |
| `llms.txt` | Point any agent at [miosa.ai/llms.txt](https://miosa.ai/llms.txt). It lists every docs page in plain text. |

This works with OSA, Claude Code, Codex, OpenCode, Hermes, Kimi Code, and more.
The rest of this page puts an agent inside a sandbox.

Already know what you need?

- [Sandbox to deployment](/docs/quickstart/sandbox-to-deployment): the recommended first walkthrough.
- [Agent-driven build](/docs/quickstart/agent-driven-build): dispatch work, stream events, and collect artifacts from your backend.
- [Docker Host](/docs/quickstart/app-engine): run containerized applications on a workspace Docker Host host; check [availability](/docs/availability) first.
- [Connect your own server](/docs/opencomputers/connect): run your first command on a Linux machine you operate.

## 1. Install

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/miosa-cli-go/main/install.sh | sh
```

**Windows**

```powershell
npm i -g @miosa/cli
```

The installer does these steps:

1. Detects your OS and CPU.
2. Downloads the matching `miosa` binary and checks its SHA-256.
3. Installs it to `~/.local/bin/miosa` (or `/usr/local/bin` when run as root).

Then run `miosa login` to sign in: it opens your browser, and the key for this machine is stored in `~/.miosa/config.toml`.
Prefer npm, Homebrew, or a manual download? See [Install the CLI or SDK](/docs/cli/install).

## 2. Start a sandbox

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
miosa list                     # see all your sandboxes
```

A sandbox is a fast, isolated Linux VM.
Your agent writes files into it, runs commands, and opens ports as live preview URLs.
Its filesystem persists when the sandbox pauses, so you can resume it later.
Read [Sandboxes](/docs/develop/sandboxes) for sizes, limits, and lifecycle.

## 3. Run an agent inside it

Pick your agent.
MIOSA installs it in the sandbox and runs it there.
The agent uses your own model account, so you pay your own provider.
MIOSA credits never pay for the model.

Each agent has three parts: connect your account, start the agent, and give it a task.

**OSA**

OSA is MIOSA's own agent. It works with the model you connect.

**Connect.**
Open the MIOSA console, go to **Agents**, and connect your model key for OSA.

**Start it and give it a task.**

```bash
miosa prompt --harness osa --sandbox my-box \
  "Build a landing page and serve it on port 3000"
```

**Claude Code**

**Connect.**
Sign in with your Claude plan.
The command prints a link and a code. Approve it in your browser, then paste the code back.

```bash
miosa connections add models claude --sandbox my-box
```

Or use your own Anthropic key:

```bash
miosa connections add models anthropic --key-stdin <<< "$ANTHROPIC_API_KEY"
```

**Start it and give it a task.**

```bash
miosa prompt --harness claude-code --sandbox my-box \
  "Build a landing page and serve it on port 3000"
```

**Codex**

**Connect.**
Sign in with your ChatGPT plan.
The command prints a link and a code. Approve it in your browser.

```bash
miosa connections add models chatgpt --sandbox my-box
```

Or use your own OpenAI key:

```bash
miosa connections add models openai --key-stdin <<< "$OPENAI_API_KEY"
```

**Start it and give it a task.**

```bash
miosa prompt --harness codex --sandbox my-box \
  "Build a landing page and serve it on port 3000"
```

**OpenCode**

**Connect.**
In the MIOSA console, go to **Agents** and connect one of these for OpenCode: an Anthropic, OpenAI, OpenRouter, or Mistral key, or your ChatGPT plan.
`miosa connections` does not cover this agent yet.

**Start it and give it a task.**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"opencode\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Hermes**

Hermes Agent is a first-class runner in the run API.
It is not in the CLI `--harness` list yet, so start it with the run API.

**Connect.**
In the MIOSA console, go to **Agents**, and connect a ChatGPT plan, a Claude plan (Max with extra usage), or an Anthropic, OpenAI, or OpenRouter key.
`miosa connections` does not cover this agent yet.

**Start it and give it a task.**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"hermes\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Kimi Code**

**Connect.**
In the MIOSA console, go to **Agents** and connect your Kimi Code plan or your Moonshot key.
`miosa connections` does not cover this agent yet.

**Start it and give it a task.**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"kimi\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Pi**

**Connect.**
In the MIOSA console, go to **Agents** and connect one of these for Pi: an Anthropic, OpenAI, OpenRouter, or Mistral key, or your Claude or ChatGPT plan.
`miosa connections` does not cover this agent yet.

**Start it and give it a task.**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"pi\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Mistral**

**Connect.**
In the MIOSA console, go to **Agents** and connect your Mistral Vibe plan or your Mistral key.
`miosa connections` does not cover this agent yet.

**Start it and give it a task.**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"mistral\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

If you have not connected an account, the run is refused or fails.
MIOSA never falls back to its own keys.

## 4. See what it built

```bash
miosa preview create my-box 3000 --visibility public
```

Open the URL it prints.
That is the page your agent built, served from inside the sandbox.

If nothing loads, the agent did not leave a server running.
Start one yourself:

```bash
miosa process start my-box --name web --cwd /workspace -- python3 -m http.server 3000 --bind 0.0.0.0
```

## Or do it by hand

You do not need an agent for this.
Write a file, start a server, and open a preview yourself.

**CLI**

```bash
miosa exec my-box -- echo "hello from MIOSA"
echo "<h1>Hello</h1>" | miosa files write my-box:/workspace/index.html
miosa process start my-box --name web --cwd /workspace -- python3 -m http.server 3000 --bind 0.0.0.0
miosa preview create my-box 3000 --visibility public
```

`miosa process start` returns at once and keeps the server running.
Do not use `miosa exec` for a server, because `exec` waits for the command to end.

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceSlug: "default", // the workspace it belongs to; every create names one
  wait: true, // hold the call until the sandbox is ready
})
console.log(sandbox.id, sandbox.state)
// sbx_a1b2c3d4e5  running

await sandbox.files.write(
  "/workspace/index.html",
  `<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>`
)

// Start the server in the background. This call returns right away.
await sandbox.exec(
  "nohup python3 -m http.server 3000 --bind 0.0.0.0 > /tmp/web.log 2>&1 &",
  { cwd: "/workspace" }
)

const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

You can also hand the whole job to an agent: `await sandbox.run("Build a landing page and serve it on port 3000", { runner: "codex" })`.

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_slug="default",  # the workspace it belongs to; every create names one
    wait=True,
)
print(sandbox.id, sandbox.state)
# sbx_a1b2c3d4e5  running

sandbox.files.write(
    "/workspace/index.html",
    """<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>""",
)

# Start the server in the background. This call returns right away.
sandbox.processes.start("python3 -m http.server 3000 --bind 0.0.0.0", name="web")

preview = sandbox.previews.create(port=3000)
print(preview["url"])
# https://3000-abc12345.sandbox.miosa.ai
```

Open the preview URL MIOSA returns. Your `index.html` is served live from inside the sandbox.

## Pick what you need

| You want to | Product | Start here |
|---|---|---|
| Run code, builds, tests, and dev servers | [Sandbox](/docs/develop/sandboxes) | [Create and connect](/docs/develop/sandboxes/create) |
| Drive a browser, a desktop app, screenshots | [Computer](/docs/computers/overview) | [Computers overview](/docs/computers/overview) |
| Ship an always-on app at a public URL | [Docker Host](/docs/docker-host/docker-deploy) | [Deploy overview](/docs/docker-host/overview) |
| Run an AI agent against a sandbox or computer | [Agent Runs](/docs/api-reference/agent-runs) | [Agent operating guide](/docs/guides/agent-operating-guide) |
| Build a white-label product on MIOSA | [Platform builders](/docs/platform/overview) | [Platform overview](/docs/platform/overview) |

## Next

- [Sandbox to deployment](/docs/quickstart/sandbox-to-deployment): The full first walkthrough: create a sandbox, write files, preview, publish to production, and prove the deployment is live.
- [Agent-driven build](/docs/quickstart/agent-driven-build): Dispatch work to your agent, stream events, and collect artifacts from your backend, with external attribution.
- [Docker Host](/docs/quickstart/app-engine): Run containerized applications on a workspace Docker Host host.
- [Connect your own server](/docs/opencomputers/connect): Run your first command on a Linux machine you operate.

- [Concepts](/docs/concepts): Every MIOSA primitive in one architecture reference.
- [API reference](/docs/api-reference): Every endpoint, error code, and webhook.
