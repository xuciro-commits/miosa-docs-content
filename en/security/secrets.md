---
slug: "security/secrets"
title: "Secrets  -  Quickstart"
description: "Store API keys in MIOSA. Your sandbox uses an opaque placeholder. The real key never lives inside the VM."
section: "Sandboxes"
order: 2018
---

Store an API key once in MIOSA. Every sandbox using it sees an opaque placeholder env var. Rotate in one place, every sandbox picks up the new value automatically.

## Add an API key (30 seconds)

### From the dashboard

```text
1. Open your Sandbox or Computer in the MIOSA dashboard
2. Click the "Secrets" tab
3. Click "+ Add Secret"
4. Fill in:
     Name:        openai_key
     Value:       sk-proj-...  (paste your real key)
     Expose as:   OPENAI_API_KEY
5. Save
```

Inside the sandbox, `process.env.OPENAI_API_KEY` is now an opaque token. Use it in your code exactly as you would the real key  -  the swap happens on egress.

### From the SDK

**Python**

```python
from miosa import Miosa
client = Miosa(api_key="msk_u_...")
sandbox = client.sandboxes.create(template_id="python-ds")

sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

**TypeScript**

```ts
import { Miosa } from '@miosa/sdk';
const miosa = new Miosa({ apiKey: 'msk_u_...' });
const sandbox = await miosa.sandboxes.create({ template: 'node-web' });

await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
```

**Go**

```go
client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
sandbox, _ := client.Sandboxes.Create(ctx, miosa.SandboxCreate{Template: "go-web"})

sandbox.Secrets.Set(ctx, miosa.SecretSet{
    Name:         "openai_key",
    Value:        realKey,
    ExposeAsEnv:  "OPENAI_API_KEY",
})
```

## How the placeholder works

Inside the sandbox after binding:

```bash
$ echo $OPENAI_API_KEY
miosa-tok-7f2a8c1d4b2e6f0a...
```

When your code makes an outbound call:

```python
requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
    json={...},
)
```

…the MIOSA proxy on your VM's host sees the request, finds `miosa-tok-7f2a8c1d4b2e6f0a` in the `Authorization` header, and **swaps it with your real `sk-proj-...`** before forwarding to OpenAI. OpenAI sees the real key. Your sandbox never did.

The placeholder is unique per binding  -  there is no global table of placeholders an attacker could brute-force.

> **Note: Where the swap happens**
>
> Header values, query parameters, URL paths, and (when buffered) request bodies are all scanned for placeholder tokens. Basic-auth headers are decoded → swapped → re-encoded.

## Rotate a key

```python
sandbox.secrets.rotate("openai_key", new_value="sk-proj-newvalue...")
```

Every sandbox using this secret picks up the new value on its next outbound request. No restart. No redeploy. The placeholder token doesn't change.

## List + delete

```python
secrets = sandbox.secrets.list()
# Returns metadata only  -  name, type, preview (last 4 chars), last_used_at.
# Never returns the real value.

sandbox.secrets.delete("openai_key")
```

## Scope levels

A secret can be scoped at five levels  -  the proxy picks the most specific match for the resource making the request:

| Scope | Use case |
|---|---|
| `tenant` | Your organization's company-wide credentials |
| `workspace` | Credentials shared by everyone in one workspace |
| `user` | An individual MIOSA user's personal credentials |
| `external_user` | A white-label end-user's credentials (see [White-label](/docs/security/white-label)) |
| `external_workspace` | A white-label workspace's credentials |

Default scope when using `sandbox.secrets.set` is determined by the API key making the call. Tenant admins can create tenant-wide secrets; individual users create user-scoped ones for their own resources.

## Runtime profiles vs secrets

Agent runtime profiles and secrets solve different problems.

| Configuration | Put it in | Example |
|---|---|---|
| Non-secret runtime defaults | Agent runtime profile `env` | `ANTHROPIC_MODEL=claude-sonnet-4.6` |
| Provider API keys | Managed secret or connector | `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `HIGGSFIELD_API_KEY` |
| Tool catalog | Agent runtime profile `tools` | `filesystem`, `shell`, `browser`, `refero` |
| Account connection | Connector binding | GitHub org, Slack workspace, CRM tenant |
| Per-run override | Agent run `env` | `FEATURE_FLAG=on` |

For white-label products, use this hierarchy:

```text
Tenant default
  -> workspace default
    -> project or resource binding
      -> sandbox/computer runtime
        -> run-level override
```

A Lumen-style platform can set an Anthropic key once for a customer workspace. Every
sandbox or computer launched in that workspace can receive the right placeholder
or connector token automatically, while the runtime profile chooses which agent
harness and tools are enabled.

Do not store long-lived provider secrets in runtime profile `env`. Profiles are
for defaults and policy. Secrets and connectors are for credentials.

## What the proxy can do beyond a string swap

For OAuth-managed credentials (see [Connect Accounts](/docs/security/connect-accounts)), the proxy also:

- **Refreshes expired tokens** automatically using the stored refresh token
- **Surfaces token revocation** (when the upstream side invalidates the access) as a 401 to your code, exactly as if you'd held the token yourself

You don't write any of that logic.

## All 5 SDKs

**Python**

```python
from miosa import Miosa
client = Miosa(api_key="msk_u_...")
sandbox = client.sandboxes.create(template_id="python-ds")

sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
sandbox.secrets.rotate("openai_key", new_value="sk-proj-newvalue...")
secrets = sandbox.secrets.list()
sandbox.secrets.delete("openai_key")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';
const miosa = new Miosa({ apiKey: 'msk_u_...' });
const sandbox = await miosa.sandboxes.create({ template: 'node-web' });

await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
await sandbox.secrets.rotate({ name: 'openai_key', newValue: 'sk-proj-newvalue...' });
const secrets = await sandbox.secrets.list();
await sandbox.secrets.delete('openai_key');
```

**Go**

```go
client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
sandbox, _ := client.Sandboxes.Create(ctx, miosa.SandboxCreate{Template: "go-web"})

sandbox.Secrets.Set(ctx, miosa.SecretSet{
    Name:        "openai_key",
    Value:       realKey,
    ExposeAsEnv: "OPENAI_API_KEY",
})
sandbox.Secrets.Rotate(ctx, "openai_key", "sk-proj-newvalue...")
secrets, _ := sandbox.Secrets.List(ctx)
sandbox.Secrets.Delete(ctx, "openai_key")
```

**Java**

```java
MiosaClient client = new MiosaClient(System.getenv("MIOSA_API_KEY"));
Sandbox sandbox = client.sandboxes().create(SandboxCreate.builder()
    .template("python-ds").build());

sandbox.secrets().set(SecretSet.builder()
    .name("openai_key")
    .value(System.getenv("MY_OPENAI_KEY"))
    .exposeAsEnv("OPENAI_API_KEY")
    .build());
sandbox.secrets().rotate("openai_key", "sk-proj-newvalue...");
List<SecretMeta> secrets = sandbox.secrets().list();
sandbox.secrets().delete("openai_key");
```

**Elixir**

```elixir
{:ok, sandbox} = Miosa.Sandboxes.create(%{template_id: "python-ds"})

{:ok, _} = Miosa.Secrets.set(sandbox.id, %{
  name: "openai_key",
  value: System.get_env("MY_OPENAI_KEY"),
  expose_as_env: "OPENAI_API_KEY"
})
{:ok, _} = Miosa.Secrets.rotate(sandbox.id, "openai_key", "sk-proj-newvalue...")
{:ok, secrets} = Miosa.Secrets.list(sandbox.id)
{:ok, _} = Miosa.Secrets.delete(sandbox.id, "openai_key")
```

## Real-world example  -  multi-service agent with OpenAI and Stripe

An agent that drafts payment summaries by calling both OpenAI and the Stripe API. Two secrets, one sandbox, no credentials in the VM.

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
sandbox = client.sandboxes.create(template_id="python-ds")

# Store both secrets
sandbox.secrets.set(
    name="openai_key",
    value=os.environ["MY_OPENAI_KEY"],
    expose_as_env="OPENAI_API_KEY",
)
sandbox.secrets.set(
    name="stripe_key",
    value=os.environ["MY_STRIPE_KEY"],
    expose_as_env="STRIPE_SECRET_KEY",
)

agent_code = '''
import os, requests, json

# Both env vars are placeholders; the proxy swaps them on egress
OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
STRIPE_SECRET_KEY = os.environ["STRIPE_SECRET_KEY"]

# Fetch recent Stripe charges
charges = requests.get(
    "https://api.stripe.com/v1/charges?limit=5",
    auth=(STRIPE_SECRET_KEY, ""),
).json()["data"]

# Ask OpenAI to summarize them
summary_prompt = "Summarize these Stripe charges in one sentence: " + json.dumps(charges)
resp = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
    json={"model": "gpt-4o-mini", "messages": [{"role": "user", "content": summary_prompt}]},
)
print(resp.json()["choices"][0]["message"]["content"])
'''

sandbox.files.write("/workspace/agent.py", agent_code)
result = sandbox.exec("python /workspace/agent.py")
print(result.stdout)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.create({ template: 'node-web' });

await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
await sandbox.secrets.set({
  name: 'stripe_key',
  value: process.env.MY_STRIPE_KEY!,
  exposeAsEnv: 'STRIPE_SECRET_KEY',
});

const agentCode = `
const resp = await fetch('https://api.stripe.com/v1/charges?limit=5', {
  headers: { Authorization: 'Bearer ' + process.env.STRIPE_SECRET_KEY },
});
const { data: charges } = await resp.json();

const ai = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer ' + process.env.OPENAI_API_KEY,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'gpt-4o-mini',
    messages: [{ role: 'user', content: 'Summarize: ' + JSON.stringify(charges) }],
  }),
});
const { choices } = await ai.json();
console.log(choices[0].message.content);
`;

await sandbox.files.write('/workspace/agent.ts', agentCode);
const result = await sandbox.exec('npx tsx /workspace/agent.ts');
console.log(result.stdout);
```

## Troubleshooting

**The sandbox env var is empty  -  `echo $OPENAI_API_KEY` prints nothing.**
The secret was saved at `scope="tenant"` or `scope="workspace"` but the sandbox doesn't belong to that scope. Re-save with `scope="sandbox"` (the default when using `sandbox.secrets.set`).

**`secrets.list()` shows the secret but the upstream API returns 401.**
The proxy is not seeing the placeholder in the request. Confirm the library you're using puts the token in `Authorization: Bearer ...`. Check the audit row's `request_transforms.secrets.swapped`  -  if empty, the proxy never matched.

**I rotated the key but nothing changed.**
Rotation is instant. The upstream API call succeeding after rotation means the new key is already working. Verify the old key is actually invalid by revoking it on the provider side.

**I deleted a secret but the env var still has a value inside the sandbox.**
The sandbox process caches env vars at startup. Restart the process or open a new exec session  -  the proxy checks the vault on each request, but the env var in the process's memory reflects the last value it read.

For more issues, see the [Security troubleshooting hub](/docs/security/troubleshooting).

## Frequently asked

**Can I read a secret's real value via the SDK after I save it?**
No. `secrets.list` returns metadata only (name, type, preview, last-used time). The value is sealed.

**What if I want to keep using `.env` files?**
Use them. MIOSA Secrets is purely additive; `.env` files continue to work. Use Secrets for credentials you want centrally managed.

**Will this slow my sandbox down?**
About 1-3 ms per outbound request. Trivial vs typical API latency.

**Does it work with `curl`, `requests`, `fetch`, `axios`, `gRPC`, …?**
Yes. The proxy operates at the network layer; the swap is invisible to the language runtime.
