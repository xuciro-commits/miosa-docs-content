---
slug: "quickstart/agent-driven-build"
title: "Agent-driven build"
description: "Dispatch work to your agent, stream events, and collect artifacts from your backend, tagging every resource with your own customer IDs."
section: "Get started"
order: 3
---

This track treats the sandbox as an agent's tool environment.
You can supply the intelligence from your own backend, or use the Agent Runs API when you want MIOSA to dispatch OSA, Claude Code, Codex, or another harness against a sandbox or computer target.

For the managed runner path, see [Agent Runs](/docs/api-reference/agent-runs) and [Agent Run Groups](/docs/api-reference/agent-run-groups).
To try a harness from the terminal first, follow [Quickstart, step 3](/docs/quickstart#3-run-an-agent-inside-it).

## Pick a harness

A harness is the coding agent that MIOSA runs inside the sandbox.
You pick it with the `runner` field.
The agent uses your own model account, never MIOSA credits.

| Agent | `runner` | Connect your account |
|---|---|---|
| OSA | `osa` | Connect your model key in the console, under **Agents**. |
| Claude Code | `claude-code` | `miosa connections add models claude` (your plan) or `anthropic` (your key). |
| Codex | `codex` | `miosa connections add models chatgpt` (your plan) or `openai` (your key). |
| OpenCode | `opencode` | `miosa connections add models chatgpt`, `openrouter` or `anthropic`, then `miosa connections use opencode --all`. |
| Kimi Code | `kimi` | `miosa connections add models kimi` (your plan). |
| Pi | `pi` | `miosa connections add models anthropic`, `chatgpt` or `openrouter`, then `miosa connections use pi --all`. |
| Mistral | `mistral` | `miosa connections add models vibe` (your plan) or `mistral` (your key). |
| Hermes | `hermes` | `miosa connections add models chatgpt` (your plan) or `anthropic` / `openai` / `openrouter` (your key). |

Run one from your backend:

**TypeScript**

```typescript
const run = await sandbox.run("Build a landing page and serve it on port 3000", {
  runner: "osa", // or "claude-code", "codex", "opencode", "kimi", "pi", "mistral"
})
console.log(run.status)
```

**Python**

```python
run = sandbox.run_agent(
    "Build a landing page and serve it on port 3000",
    runner="osa",  # or "claude-code", "codex", "opencode", "kimi", "pi", "mistral"
)
print(run["status"])
```

If the account for that harness is not connected, MIOSA refuses the run and says what to connect.

You do not need the console to connect one.
Set a key or connect a Claude or ChatGPT plan from your own code, for your workspace or for each of your end users: [Connect agent accounts](/docs/guides/connect-agent-accounts).

**TypeScript**

```typescript
await miosa.connections.setKey({ provider: "anthropic", key: process.env.ANTHROPIC_API_KEY! })

// A customer's own Claude plan, used only for runs on their machines
const signin = await miosa.connections.connectSubscription({ provider: "claude", externalUserId: "user-456" })
console.log(signin.verificationUrl) // show it in your UI, then: await signin.submitCode(pastedCode)
```

**Python**

```python
miosa.connections.set_key("anthropic", key=os.environ["ANTHROPIC_API_KEY"])

# A customer's own Claude plan, used only for runs on their machines
signin = miosa.connections.connect_subscription("claude", external_user_id="user-456")
print(signin.verification_url)  # show it in your UI, then: signin.submit_code(pasted_code)
```

### Step 1  -  Create a sandbox with external attribution.
   If you're embedding MIOSA in your own product, pass your user/project IDs so you can filter resources later.

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceSlug: "default", // the MIOSA workspace it belongs to
  wait: true, // hold the call until the sandbox is ready
  externalWorkspaceId: "acct-123", // your own grouping id, not a MIOSA workspace
  externalUserId: "user-456",
  externalProjectId: "intake-portal-789",
})
console.log(sandbox.id)
// sbx_f6g7h8i9j0
```

**Python**

```python
sandbox = miosa.sandboxes.create(
    template_id="miosa-sandbox",
    wait=True,  # hold the call until the sandbox is ready
    external_workspace_id="acct-123",
    external_user_id="user-456",
    external_project_id="intake-portal-789",
)
print(sandbox.id)
# sbx_f6g7h8i9j0
```

### Step 2  -  Give your agent an exec tool.
   The pattern below shows the structure. Substitute `callYourLLM` with your actual model call.

**TypeScript**

```typescript
// Tool the agent can call
async function execInSandbox(command: string): Promise<string> {
  const result = await sandbox.exec(command)
  return result.stdout + result.stderr
}

// Example: agent scaffolds a project
const output = await execInSandbox("npx create-react-app /workspace/my-app --template minimal 2>&1 | tail -5")
console.log(output)
// Success! Created /workspace/my-app
// Happy hacking!
```

**Python**

```python
def exec_in_sandbox(command: str) -> str:
    result = sandbox.exec(command)
    return result.stdout + result.stderr

# Example: agent scaffolds a project
output = exec_in_sandbox(
    "npx create-react-app /workspace/my-app --template minimal 2>&1 | tail -5"
)
print(output)
# Success! Created /workspace/my-app
# Happy hacking!
```

### Step 3  -  Let the agent write files directly.

**TypeScript**

```typescript
// Agent writes a component
await sandbox.files.write(
  "/workspace/my-app/src/App.jsx",
  `export default function App() {
  return <main><h1>Generated by my agent</h1></main>;
}`
)
```

**Python**

```python
# Agent writes a component
sandbox.files.write(
    "/workspace/my-app/src/App.jsx",
    """export default function App() {
  return <main><h1>Generated by my agent</h1></main>;
}""",
)
```

### Step 4  -  Start the dev server, create a preview, stream it to your user.

**TypeScript**

```typescript
// Start the dev server in the background. This call returns right away.
await sandbox.exec("nohup npm start > /tmp/app.log 2>&1 &", { cwd: "/workspace/my-app" })

const preview = await sandbox.previews.create(3000)
const share = await sandbox.previews.share(preview.id as string)

console.log("embed this URL in your product's UI:", share.share_url)
// https://3000-abc12345.sandbox.miosa.ai?preview_token=miosa_pv_...
```

**Python**

```python
# Start the dev server in the background. This call returns right away.
sandbox.processes.start("cd /workspace/my-app && npm start", name="app")

preview = sandbox.previews.create(port=3000)
share = sandbox.previews.share(preview["id"])

print("embed this URL in your product's UI:", share["share_url"])
# https://3000-abc12345.sandbox.miosa.ai?preview_token=miosa_pv_...
```

   The `share_url` is safe to put in an `&lt;iframe&gt;` in your end user's browser. It expires after the configured TTL. Your `msk_*` key never leaves your server.

### Step 5  -  Take a screenshot of a Computer (optional).
   If your agent needs to verify GUI output or run browser-based tasks, use a Computer instead of (or alongside) a Sandbox.

**TypeScript**

```typescript
const computer = await miosa.computers.create({
  name: "agent-desktop",
  template_type: "miosa-desktop",
  size: "small",
})

// Let the agent observe the screen
const png = await computer.desktop.screenshot()
// → Buffer containing PNG bytes; send to your vision model

// Let the agent interact
await computer.desktop.click({ x: 800, y: 400 })
await computer.desktop.type("hello world")

await computer.destroy()
```

**Python**

```python
computer = miosa.computers.create(
    name="agent-desktop",
    template_type="miosa-desktop",
    size="small",
)

# Let the agent observe the screen
png = computer.desktop.screenshot()
# → bytes containing PNG; send to your vision model

# Let the agent interact
computer.desktop.click(x=800, y=400)
computer.desktop.type("hello world")

computer.destroy()
```

### Step 6  -  Publish what the agent built.

**TypeScript**

```typescript
const result = await sandbox.deploy({
  name: "intake-portal",
  outputPath: "/workspace/my-app/build",
})
// Read the production URL from the response; do not construct it.
console.log(result)
```

**Python**

```python
result = sandbox.deploy(
    name="intake-portal",
    output_path="/workspace/my-app/build",
)
# Read the production URL from the response; do not construct it.
print(result)
```

### Step 7  -  Filter all sandboxes for this customer.

**TypeScript**

```typescript
const allForCustomer = await miosa.sandboxes.list({
  externalWorkspaceId: "acct-123",
})
console.log(`${allForCustomer.length} sandboxes for acct-123`)
// 3 sandboxes for acct-123
```

**Python**

```python
all_for_customer = miosa.sandboxes.list(external_workspace_id="acct-123")
print(f"{len(all_for_customer)} sandboxes for acct-123")
# 3 sandboxes for acct-123
```

### Step 8  -  Clean up.

**TypeScript**

```typescript
await sandbox.destroy()
```

**Python**

```python
sandbox.destroy()
```

### What you just did

| Step | What happened |
|------|--------------|
| External attribution | Tagged every resource with your customer's IDs  -  no MIOSA accounts needed for your end users |
| Exec as agent tool | Ran arbitrary shell commands inside an isolated VM from your agent loop |
| `files.write` | Agent wrote generated code directly to the sandbox file system |
| Share token | Minted a browser-safe URL for preview embedding  -  workspace key stays server-side |
| Computer screenshot | Optionally gave the agent a real desktop with a vision-accessible screen |
| `sandbox.deploy` | Shipped the agent's output to a permanent immutable URL |
| `sandboxes.list` with attribution | Queried only resources belonging to one customer  -  cross-tenant leakage impossible |
