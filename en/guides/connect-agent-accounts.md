---
slug: "guides/connect-agent-accounts"
title: "Connect agent accounts (keys and subscriptions)"
description: "Connect a model key, a Claude or ChatGPT plan, or a tool-server key to your workspace, or to one of your end users, from your own code. No console visit needed."
section: "Agents"
order: 6019
---

An agent run thinks with an account that belongs to its owner.
MIOSA never powers your agents with its own model keys.
So before a run can start, a model provider has to be connected: an API key, or a subscription plan such as Claude or ChatGPT.

You can do that in the console.
You can also do it from your own code, for your own workspace or for each of your end users.
That second case is what a platform needs: your customer connects their own Claude plan inside your product, and every run on their machines uses it.

This guide covers both.

## What you can connect

| Connection | What it is | How you connect it | Name in the SDK |
|---|---|---|---|
| Claude plan | A Claude Pro or Max subscription. | Sign-in, the user pastes a code back. | `claude` |
| ChatGPT plan | A ChatGPT Plus, Pro or Team subscription (Codex). | Sign-in, the user enters a code. | `chatgpt` |
| Kimi plan | A Kimi Code subscription. | Sign-in, the user enters a code. | `kimi` |
| Mistral Vibe plan | A Mistral Vibe subscription. | Sign-in in the browser. | `mistral` |
| Anthropic key | An Anthropic API key. | Set it. | `anthropic` |
| OpenAI key | An OpenAI API key. | Set it. | `openai` |
| OpenRouter key | An OpenRouter API key. | Set it. | `openrouter` |
| Other keys | DeepSeek, LLM Gateway, Moonshot, Mistral, Amazon Bedrock. | Set it. | `deepseek`, `llmgateway`, `moonshot`, `mistral`, `bedrock` |
| Tool-server keys | The key a tool server needs: Apify, Refero, any MCP server. | Set it. | `toolKeys` |

A key never leaves the platform.
Every read returns a status, a redacted preview such as `sk-ant...1234`, and for a plan the account email and plan name.
Never the secret.

> **Note**
>
> One connected plan can power several harnesses where the vendor supports it.
> A ChatGPT plan powers Codex and OpenCode, for example.
> Connect it once, and any run on a compatible harness picks it up.
> See [which harness uses which connection](/docs/guides/connect-agent-accounts#which-harness-uses-which-connection).

## Who the connection belongs to

Every call takes the same two optional identifiers.

| You pass | The connection belongs to | A run uses it |
|---|---|---|
| Nothing | Your organization. | In every workspace, unless a narrower one exists. |
| `workspaceId` | One workspace. | For runs in that workspace. |
| `externalUserId` | One of your end users: your own id for them. | For runs on that end user's machines, before the workspace's. |

A run looks for the account in this order, and stops at the first one it finds:

1. The end user's own connection.
2. The workspace's connection.
3. The organization's connection.

The end user is the one on the machine the run uses.
You tag a machine with `externalUserId` when you create it, the same id you use everywhere else in your platform ([Ownership and attribution](/docs/platform/attribution)).

Writes and deletes always touch exactly the scope you name.
Setting a key for one end user never changes the workspace's key, and removing it falls back to the workspace's.
An end user's connection is never visible to another end user, to the workspace, or to another organization.

## Your own workspace

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

// An API key
await miosa.connections.setKey({
  provider: "anthropic",
  key: process.env.ANTHROPIC_API_KEY!,
  workspaceId,
})

// Check that it works
const check = await miosa.connections.test({ provider: "anthropic", workspaceId })
console.log(check.ok, check.httpStatus) // true 200
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# An API key
miosa.connections.set_key(
    "anthropic",
    key=os.environ["ANTHROPIC_API_KEY"],
    workspace_id=workspace_id,
)

# Check that it works
check = miosa.connections.test("anthropic", workspace_id=workspace_id)
print(check["ok"], check["http_status"])  # True 200
```

**CLI**

```bash
# The key comes from standard input, never from an argument
miosa connections add models anthropic --workspace-id "$WORKSPACE_ID" --key-stdin <<< "$ANTHROPIC_API_KEY"

miosa connections test models anthropic --workspace-id "$WORKSPACE_ID"
```

Setting a key again replaces it.
That is how you rotate one.

## Your end users

Pass `externalUserId` and the connection is theirs.

### An API key

**TypeScript**

```typescript
await miosa.connections.setKey({
  provider: "openrouter",
  key: userSuppliedKey,
  externalUserId: "cus_123",
})
```

**Python**

```python
miosa.connections.set_key(
    "openrouter",
    key=user_supplied_key,
    external_user_id="cus_123",
)
```

**CLI**

```bash
miosa connections add models openrouter --external-user cus_123 --key-stdin <<< "$USER_KEY"
```

An end user's key is for agents only.
It is never offered to the AI Gateway.

### A Claude plan or a ChatGPT plan

Connecting a plan is a sign-in the user completes in their own browser.
Your code starts it, shows the user a link, and waits.
The call returns at once with what to show.
Nothing is connected yet.

**TypeScript**

```typescript
const signin = await miosa.connections.connectSubscription({
  provider: "claude", // or "chatgpt", "kimi", "mistral"
  externalUserId: "cus_123",
})

// Show these in YOUR UI.
signin.verificationUrl // open this in the user's browser
signin.userCode        // ChatGPT and Kimi: the code they type there
signin.mode            // "paste_code" (Claude) or "device_code"
signin.expiresAt       // about 15 minutes

// Claude: the page shows a code after they approve. Your UI passes it back.
const connected = await signin.submitCode(codeTheyPasted)
console.log(connected.accountEmail, connected.accountPlan) // alice@example.com max

// ChatGPT, Kimi: they enter the code at the link. Wait until it lands.
const connected2 = await signin.wait({ onUpdate: (s) => console.log(s.status) })
```

**Python**

```python
signin = miosa.connections.connect_subscription(
    "claude",  # or "chatgpt", "kimi", "mistral"
    external_user_id="cus_123",
)

# Show these in YOUR UI.
signin.verification_url  # open this in the user's browser
signin.user_code         # ChatGPT and Kimi: the code they type there
signin.mode              # "paste_code" (Claude) or "device_code"
signin.expires_at        # about 15 minutes

# Claude: the page shows a code after they approve. Your UI passes it back.
connected = signin.submit_code(code_they_pasted)
print(connected["account_email"], connected["account_plan"])  # alice@example.com max

# ChatGPT, Kimi: they enter the code at the link. Wait until it lands.
connected = signin.wait(on_update=lambda s: print(s["status"]))
```

**CLI**

```bash
# Prints the link and the code, then returns. Your UI shows them.
miosa connections add models claude --external-user cus_123 --no-wait --json

# Claude: hand over the pasted code.
miosa connections signin code SIGNIN_ID --external-user cus_123 --code-stdin <<< "$PASTED_CODE"

# ChatGPT, Kimi: wait for the user to enter the code.
miosa connections signin wait SIGNIN_ID --external-user cus_123
```

There are two shapes of sign-in:

| Mode | Plans | What the user does | What you do |
|---|---|---|---|
| `paste_code` | Claude | Approves at the link, then copies the code the page shows. | Pass that code to `submitCode`. |
| `device_code` | ChatGPT, Kimi | Opens the link and types `userCode`. | Call `wait()`, or `poll()` on a timer. |

`wait()` resolves when the plan is connected.
It rejects with `SubscriptionSignInError` if the sign-in failed, expired or was cancelled.
`poll()` never rejects for a failed sign-in: read its `status`.
`cancel()` ends the sign-in.

A wrong or expired pasted code is rejected with `SIGNIN_CODE_REJECTED` (422).
The sign-in stays open, so the user can paste again.

The sign-in link and the code carry no credentials.
The plan's tokens are stored encrypted on the platform, refreshed in the background, and never returned by any call.

> **Caution**
>
> Show the link to the person who owns the plan, in your own UI.
> Do not open it for them from your server.
> The sign-in is the user's consent to connect their plan.

### Check, list and remove

**TypeScript**

```typescript
// Is it connected, and where does it come from?
const claude = await miosa.connections.get({ provider: "claude", externalUserId: "cus_123" })
claude.connected    // true
claude.scope        // "external_user", or "workspace" when it is inherited
claude.inherited    // true when it comes from a wider scope
claude.preview      // "sk-ant...1234" for a key. Never the secret.

// Everything this end user has, with the workspace's inherited connections marked
const all = await miosa.connections.list({ externalUserId: "cus_123", workspaceId })

// Does it work?
await miosa.connections.test({ provider: "claude", externalUserId: "cus_123" })

// Disconnect only the end user's own. The workspace's, if any, is untouched.
await miosa.connections.delete({ provider: "claude", externalUserId: "cus_123" })
```

**Python**

```python
claude = miosa.connections.get("claude", external_user_id="cus_123")
claude["connected"]   # True
claude["scope"]       # "external_user", or "workspace" when it is inherited
claude["inherited"]   # True when it comes from a wider scope

everything = miosa.connections.list(external_user_id="cus_123", workspace_id=workspace_id)

miosa.connections.test("claude", external_user_id="cus_123")
miosa.connections.delete("claude", external_user_id="cus_123")
```

**CLI**

```bash
miosa connections list --external-user cus_123 --workspace-id "$WORKSPACE_ID"
miosa connections test models claude --external-user cus_123
miosa connections rm models claude --external-user cus_123 --yes
```

`test` makes one cheap authenticated request to the provider, such as listing models.
For a plan it checks the stored state instead.
It answers `ok: true`, `ok: false` (the provider rejected the key) or `ok: null` (the provider did not answer, which is not the key's fault).
It never returns the key or the provider's response.

## Tool-server keys

A tool server is an MCP server your agents use: Apify, Refero, your own.
You define the server once, with its credential as a placeholder such as `{{secret:APIFY_TOKEN}}`.
Each end user then supplies their own value.
An agent run for that end user reaches the tool server with their key, and falls back to the workspace's.

**TypeScript**

```typescript
// Which keys do my tool servers need, and does this end user have them?
const keys = await miosa.connections.toolKeys.list({ workspaceId, externalUserId: "cus_123" })
// [{ name: "APIFY_TOKEN", connected: false, servers: [{ name: "Apify" }], ... }]

await miosa.connections.toolKeys.set({
  name: "APIFY_TOKEN",
  value: userApifyToken,
  workspaceId,
  externalUserId: "cus_123",
})

await miosa.connections.toolKeys.delete({ name: "APIFY_TOKEN", externalUserId: "cus_123" })
```

**Python**

```python
keys = miosa.connections.tool_keys.list(workspace_id=workspace_id, external_user_id="cus_123")

miosa.connections.tool_keys.set(
    "APIFY_TOKEN",
    user_apify_token,
    workspace_id=workspace_id,
    external_user_id="cus_123",
)
miosa.connections.tool_keys.delete("APIFY_TOKEN", external_user_id="cus_123")
```

**CLI**

```bash
miosa connections tool-keys list --external-user cus_123
miosa connections tool-keys set APIFY_TOKEN --external-user cus_123 --key-stdin <<< "$USER_TOKEN"
miosa connections tool-keys rm APIFY_TOKEN --external-user cus_123 --yes
```

Only a name that a tool server references can be set.
Anything else is `UNKNOWN_TOOL_KEY` (404), so this is not a general-purpose secret store.

## Which harness uses which connection

A harness is the agent that runs on the machine: Claude Code, Codex, OpenCode and the others.
Each accepts a set of connections.

| Harness (`runner`) | Accepts |
|---|---|
| Claude Code (`claude-code`) | Claude plan, Anthropic key, OpenRouter key, DeepSeek key, Amazon Bedrock |
| Codex (`codex`) | ChatGPT plan, OpenAI key, DeepSeek key, Amazon Bedrock |
| OpenCode (`opencode`) | ChatGPT plan, Anthropic key, OpenAI key, OpenRouter key, LLM Gateway key, Mistral key, Mistral Vibe plan |
| Pi (`pi`) and Prime Agent (`prime`) | Claude plan, ChatGPT plan, Anthropic key, OpenAI key, OpenRouter key, LLM Gateway key, Mistral key, Mistral Vibe plan |
| Hermes Agent (`hermes`) | ChatGPT plan, Claude plan, Anthropic key, OpenAI key, OpenRouter key |
| Kimi Code (`kimi`) | Kimi plan, Moonshot key |
| Mistral Vibe (`mistral`) | Mistral Vibe plan, Mistral key |
| OSA (`osa`) | An Anthropic, OpenAI or OpenRouter key, or the model key you set in Agent Studio |

Two limits come from the vendors, not from MIOSA:

- OpenCode does not take a Claude plan. Anthropic prohibits it, and OpenCode removed the support.
- Hermes takes a Claude plan only on a Claude Max plan with extra usage purchased. A Pro plan is refused by Anthropic.

You connect once, and any run on a compatible harness picks the connection up.
You do not pick a connection per run.

To see what a given user can run right now:

**TypeScript**

```typescript
for (const harness of await miosa.harnesses.list({ externalUserId: "cus_123" })) {
  const usable = harness.connections.filter((c) => c.usable).map((c) => c.credential)
  console.log(harness.id, harness.ready ? "ready" : "no credential", usable)
}
// claude-code ready ["claude_subscription"]
// opencode    ready ["chatgpt_subscription"]
// pi          no credential []
```

**Python**

```python
for harness in miosa.harnesses.list(external_user_id="cus_123"):
    usable = [c["credential"] for c in harness["connections"] if c["usable"]]
    print(harness["id"], "ready" if harness["ready"] else "no credential", usable)
```

**CLI**

```bash
miosa connections harnesses --external-user cus_123
```

The harness settings (which method is active, the default model) belong to you, the builder.
They are set per workspace and are the same for all of your end users.
The connections are theirs.
A key minted for an end user cannot change the harness settings.

## Choose which connection a harness uses

A harness that takes one method at a time (Claude Code, Codex) uses one connection.
A harness that takes several (OpenCode, pi, Prime Agent, Kimi Code, Mistral Vibe, Hermes Agent) uses the ones you enable.
These are the builder's settings, set per organization or per workspace.

**TypeScript**

```typescript
await miosa.agentSettings.updateHarness("claude-code", { authMethod: "claude_subscription", workspaceId })
await miosa.agentSettings.updateHarness("opencode", { enabledProviders: ["chatgpt_subscription", "openrouter"] })
```

**Python**

```python
miosa.agent_settings.update_harness("claude-code", auth_method="claude_subscription", workspace_id=workspace_id)
miosa.agent_settings.update_harness("opencode", enabled_providers=["chatgpt_subscription", "openrouter"])
```

**CLI**

```bash
miosa connections use claude-code claude                    # the Claude plan
miosa connections use opencode chatgpt openrouter           # several
miosa connections use opencode --all                        # every connected one
miosa connections use codex --model gpt-6.1-sol --effort high
```

Then run on any harness, with the model you want:

```bash
miosa prompt --harness opencode --model gpt-6.1-sol --sandbox my-box "add a health check"
```

`miosa prompt --harness` takes every harness the platform lists, including ones added after your CLI was released.
`miosa connections harnesses opencode` lists that harness's models and which are available with what is connected.

## Build it into your product

A "Connect your Claude plan" button in your settings page is two requests to your backend.
The sign-in id travels through the browser, and your backend names the end user from its own session.

```typescript
// 1. Start. Send the browser only what it needs.
app.post("/api/connect/claude", async (req, res) => {
  const signin = await miosa.connections.connectSubscription({
    provider: "claude",
    externalUserId: req.user.id, // from YOUR session, never from the request body
  })
  res.json({ signinId: signin.id, url: signin.verificationUrl })
})

// 2. The user approves in a new tab and pastes the code into your page.
app.post("/api/connect/claude/code", async (req, res) => {
  const signin = miosa.connections.resumeSubscription({
    id: req.body.signinId,
    provider: "claude",
    externalUserId: req.user.id,
  })
  const status = await signin.submitCode(req.body.code)
  res.json({ connected: status.connected, email: status.accountEmail, plan: status.accountPlan })
})
```

`resumeSubscription` picks a sign-in up by its id.
Because it carries the end user, a sign-in id belonging to someone else is `NOT_FOUND`, even with your organization-wide key.

The worked example, with a runnable server, is in the SDK repository under `examples/connect-agent-accounts`.
It connects a Claude plan, a ChatGPT plan, a key and a tool-server key for two customers, and shows that one customer's connection never reaches the other's runs.

## Security

- **Server only.** Call these methods from your backend with your `msk_*` key.
  Never put that key in a browser.
- **Mint an end-user key** for anything that runs near your customer's code. A key minted with an `externalUserId` always acts as that end user.
  It cannot read or change the workspace's or the organization's connections (`403 SCOPE_FORBIDDEN`), cannot change harness settings, and cannot see another end user's sign-ins.
- **No secret is returned**, by any call. Keys and plan tokens are encrypted at rest, bound to the end user they belong to, and decrypted only to start a run.
- **Audit.** Every connect, replace, remove and use is written to the [audit log](/docs/security/audit-log) with the end user's id and never the value.
- **Isolation.** A connection is checked against your organization on every read.
  An end user's id means nothing to another organization.
- **Scopes.** Reading needs `integrations:read` (plus `connections:read` for the connections list).
  Changing needs `integrations:write` (and `connections:write` for sign-ins).
- **Plans are the user's.** A plan connected for an end user is used only for runs on that end user's machines.
  It is never used to serve your other customers.
- **Rate limits** apply as everywhere ([Rate limits](/docs/rate-limits)).

## Errors

| Code | Status | Meaning |
|---|---|---|
| `SCOPE_FORBIDDEN` | 403 | The key acts for one end user and asked for something wider. |
| `MISSING_EXTERNAL_USER_ID` | 422 | `scope: "external_user"` without an `externalUserId`. |
| `CONFLICTING_SCOPE` | 422 | `scope` does not agree with the identifiers. |
| `INVALID_EXTERNAL_USER_ID` | 422 | The id is longer than 255 characters or has control characters. |
| `USE_SIGNIN` | 422 | A plan cannot be set; connect it with `connectSubscription`. |
| `VALIDATION_FAILED` | 422 | A field is missing or wrong, for example an empty key. |
| `SIGNIN_CODE_REJECTED` | 422 | The pasted code was wrong or expired. Paste again. |
| `SIGNIN_PROVIDER_ERROR` | 502 | The provider's sign-in service did not answer. Start again. |
| `NOT_FOUND` | 404 | Nothing is connected at exactly that scope. |
| `UNKNOWN_CREDENTIAL` | 404 | Not a credential MIOSA knows. |
| `UNKNOWN_TOOL_KEY` | 404 | No tool server references that secret name. |
| `OWN_CREDENTIALS_REQUIRED` | 422 | A run was started with nothing connected for its harness. |

## The API underneath

The SDK and CLI are thin wrappers.
Every route accepts `workspace_id`, `external_user_id` and an optional `scope`.

| Route | Does |
|---|---|
| `GET /agents/credentials` | Every credential and its state at the scope. |
| `PUT /agents/credentials/{credential}` | Set or replace a key. |
| `DELETE /agents/credentials/{credential}` | Remove it at exactly the scope. |
| `POST /agents/credentials/{credential}/test` | Does it work? |
| `POST /agent-signins` | Start a plan sign-in. Returns the link and the code. |
| `GET /agent-signins/{id}` | Poll it. |
| `POST /agent-signins/{id}/code` | Hand over a pasted code. |
| `DELETE /agent-signins/{id}` | Cancel it. |
| `GET /agents/tool-keys` | The keys your tool servers need. |
| `PUT /agents/tool-keys/{name}` | Set one. |
| `DELETE /agents/tool-keys/{name}` | Remove one. |
| `GET /connections` | Models, apps and tool servers in one list. |
| `GET /agents/settings` | Harness readiness at the scope. |

## Next

- [Agent-driven build](/docs/quickstart/agent-driven-build): Run an agent against a sandbox once an account is connected.
- [Ownership and attribution](/docs/platform/attribution): Tag every machine with your own customer ids.
- [Agents in your product](/docs/platform/agents-in-your-product): One machine per user, with an agent working in it.
- [Agent operating guide](/docs/guides/agent-operating-guide): Dispatch work and collect results.
