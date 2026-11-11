---
slug: "security"
title: "Security"
description: "Built-in credential vault, outbound network controls, and audit log on every MIOSA sandbox and computer. No setup. Free on all plans."
section: "Security & compliance"
order: 14000
---

Every MIOSA Sandbox and Computer ships with a built-in security boundary. **No setup. No add-on SKU. Free on every plan.**

Three things you can use whenever you need them:

| Feature | What it does | Page |
|---|---|---|
| **Secrets** | Store your API keys in MIOSA  -  sandboxes see opaque placeholders, the real values stay encrypted at rest | [Secrets](/docs/security/secrets) |
| **Connect Accounts** | OAuth-based credential connection (GitHub, Google, Slack, …) instead of pasting raw tokens | [Connect Accounts](/docs/security/connect-accounts) |
| **Network Allowlist** | Watch what your sandbox calls; lock it down to an approved list when you're ready | [Network](/docs/security/network) |
| **Audit Log** | Outbound calls recorded to the egress audit log  -  searchable, exportable, live-tail (capture is not yet enabled in the current release) | [Audit Log](/docs/security/audit-log) |

## Getting started in 60 seconds

Nothing to install. Nothing to configure globally. Each Computer / Sandbox already has the three tabs.

### 1. Add a credential

Open any Sandbox or Computer → **Secrets** tab → **+ Add Secret**

```text
Name:         openai_key
Value:        sk-proj-...   ← your real key
Expose as:    OPENAI_API_KEY
```

Or via SDK (Python  -  same shape in TypeScript / Go / Java / Elixir):

```python
sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

Inside the sandbox, `process.env.OPENAI_API_KEY` is now an opaque `miosa-tok-…` placeholder. Your existing code uses it as normal. The real key is swapped in at the network boundary on the way out.

→ Full walkthrough: [Secrets](/docs/security/secrets)

### 2. Use your code unchanged

```python
import os, requests

requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
    json={...},
)
```

That's it. OpenAI sees your real `sk-…`. Your sandbox never does. No code change.

### 3. (Optional) Check what you called + lock it down

After your sandbox has been running for a while:

- **Audit Log tab**  -  live-tail outbound calls. Filter by host, status, time. Egress audit capture is not yet enabled in the current release.
- **Network tab**  -  see the auto-built list of hosts you've contacted. One click to enforce-mode with that list as the allowlist.

Both are opt-in. Audit-only is the default until you decide to lock down.

→ See [Network Allowlist](/docs/security/network) for the lockdown workflow.

### Want to wire up GitHub / Google / Slack instead of pasting tokens?

→ [Connect Accounts](/docs/security/connect-accounts)  -  OAuth flow that handles refresh automatically.

### Building a white-label product on top of MIOSA?

Your end-users don't need MIOSA accounts  -  credentials can be scoped to your own `external_user_id`.

→ [White-label Credentials](/docs/security/white-label)

---

## Your code doesn't change

The whole design rule for MIOSA Security: **your existing code keeps working.**

```python
import os, requests

# This code is identical with or without MIOSA Secrets:
response = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
    json={...},
)
```

What changes is *the value* of `OPENAI_API_KEY`. Without Secrets, it's your real `sk-...` key. With Secrets, it's an opaque placeholder like `miosa-tok-7f2a8c...`. The MIOSA proxy substitutes the real key on the way out. OpenAI sees the real key. Your sandbox never does.

## Three guarantees

> **Note: Audit-only by default**
>
> New sandboxes do not block anything. The proxy watches and logs but never gets in the way. You only enter "enforce" mode by explicitly clicking Lock Down  -  and it's reversible in one click.

> **Note: Secrets stay encrypted**
>
> Once saved, secret values are encrypted with ChaCha20-Poly1305. They are unsealed only at the moment the proxy needs to swap them into an outbound request. Nobody  -  including MIOSA engineers  -  can re-read a stored value via the API or UI.

> **Note: Existing patterns still work**
>
> `.env` files still work. `tenant_provider_keys` and `workspace_env_vars` still work. MIOSA Secrets adds a new layer; it doesn't replace anything that exists today.

## Where to go next

- **First time using this?** Start with [Secrets  -  Quickstart](/docs/security/secrets).
- **Want to wire up OAuth (GitHub etc.)?** Read [Connect Accounts](/docs/security/connect-accounts).
- **Want to lock down outbound traffic?** Read [Network Allowlist](/docs/security/network).
- **Building a white-label product on MIOSA?** Read [White-label Credentials](/docs/security/white-label).

## Glossary

**Placeholder token**
An opaque string (format: `miosa-tok-<hex>`) that MIOSA injects into a sandbox's environment in place of a real credential. The proxy swaps the placeholder with the real value on every outbound request. The placeholder is unique per binding  -  it cannot be brute-forced to discover the real key.

**OAuth Connect**
A MIOSA feature that lets you authorize a third-party account (GitHub, Google, Slack, …) via the standard OAuth 2.0 flow instead of pasting a static API key. MIOSA stores the resulting access and refresh tokens, refreshes them automatically, and exposes the current valid access token to the sandbox as a placeholder env var.

**Resolver scope walk**
The ordered lookup MIOSA performs when a sandbox makes an outbound request and the proxy needs to match a placeholder to a real credential. The proxy checks scopes in this priority order: (1) direct sandbox binding, (2) user-scoped secret, (3) external_user-scoped secret, (4) external_workspace-scoped secret, (5) MIOSA workspace secret, (6) tenant-wide secret. First match wins. If nothing matches, the placeholder passes through unchanged and the upstream API call fails (the intended safe behavior).

## Real-world example  -  OpenAI customer-support agent in 5 minutes

This is the complete setup for an AI agent that uses OpenAI safely. Run these five commands and you're done.

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# 1. Create sandbox
sandbox = client.sandboxes.create(template_id="python-ds")

# 2. Store the OpenAI key  -  sandbox never sees the real value
sandbox.secrets.set(
    name="openai_key",
    value=os.environ["MY_OPENAI_KEY"],
    expose_as_env="OPENAI_API_KEY",
)

# 3. Lock down to only api.openai.com
sandbox.network.allow("api.openai.com")
sandbox.network.lockdown()

# 4. Run the agent  -  uses $OPENAI_API_KEY exactly like normal
result = sandbox.exec("""
python3 -c "
import os, requests
resp = requests.post(
    'https://api.openai.com/v1/chat/completions',
    headers={'Authorization': f'Bearer {os.environ[\"OPENAI_API_KEY\"]}'},
    json={'model': 'gpt-4o-mini', 'messages': [{'role': 'user', 'content': 'Hello!'}]}
)
print(resp.json()['choices'][0]['message']['content'])
"
""")
print(result.stdout)

# 5. Check the audit trail
events = sandbox.audit.list(since="5m")
for e in events:
    print(e.host, e.action, e.status_code)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

// 1. Create sandbox
const sandbox = await client.sandboxes.create({ template: 'python-ds' });

// 2. Store the OpenAI key
await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});

// 3. Lock down
await sandbox.network.allow('api.openai.com');
await sandbox.network.lockdown();

// 4. Run the agent
const result = await sandbox.exec(`
python3 -c "
import os, requests
resp = requests.post(
    'https://api.openai.com/v1/chat/completions',
    headers={'Authorization': f'Bearer {os.environ[\"OPENAI_API_KEY\"]}'},
    json={'model': 'gpt-4o-mini', 'messages': [{'role': 'user', 'content': 'Hello!'}]}
)
print(resp.json()['choices'][0]['message']['content'])
"
`);
console.log(result.stdout);

// 5. Audit trail
const events = await sandbox.audit.list({ since: '5m' });
for (const e of events) {
  console.log(e.host, e.action, e.statusCode);
}
```

## Troubleshooting

**My sandbox's env var contains a placeholder but the API returns 401.**
The proxy couldn't match the placeholder in the outbound request header. Confirm the SDK puts the token in `Authorization: Bearer ...`. See [troubleshooting #1](/docs/security/troubleshooting#1-secret-value-is-not-being-swapped-outbound-request-uses-the-raw-placeholder).

**I enabled lockdown and nothing works.**
Call `sandbox.network.pending()` to see the blocked hosts, then allow them. Or revert with `sandbox.network.observe()` while you diagnose.

**I can't see any audit rows.**
The audit log records outbound calls only. If the sandbox hasn't made any outbound requests, the log is empty. Trigger a call and re-query.

**I see "Reconnect needed" on a GitHub secret.**
The OAuth token was revoked by the user or expired. Call `sandbox.secrets.reauthorize(provider="github")` to re-run the OAuth dance. See [Connect Accounts troubleshooting](/docs/security/troubleshooting#4-github-oauth-refresh-failing--sandbox-gets-401-on-apigithubcom).

For the full list of issues and fixes, see the [Security troubleshooting hub](/docs/security/troubleshooting).
