---
slug: "security/network"
title: "Network Allowlist"
description: "Watch what your sandbox calls. When you're ready, lock it down with a one-click allowlist. Reversible."
section: "Networking & domains"
order: 5003
---

By default, every new MIOSA sandbox is configured in **audit-only mode**: outbound calls are not blocked. Audit-only mode logs every outbound request, so you can see which hosts your sandbox calls before you lock down.

> **Note: Opt-in, always reversible**
>
> Lockdown is never applied automatically. You explicitly click "Lock Down"  -  and you can revert to audit-only at any time with one click. No data is lost in either direction.

## The two modes

| Mode | What it does | Default? |
|---|---|---|
| **Audit-only** | Logs every outbound request; nothing is blocked. | ✅ yes  -  every new sandbox starts here |
| **Enforce** | Applies the allowlist. Requests to unlisted hosts are blocked. | opt-in |

## The "watch then enforce" flow

### Day 0  -  Sandbox is created

No setup. Audit-only is on by default. Your sandbox makes outbound calls exactly like it did before MIOSA Egress existed. Outbound calls are logged, and the hosts they reach build your allowlist.

### Day 0-7  -  Use your sandbox normally

Build, deploy, run your agent  -  whatever. MIOSA collects audit data.

### Day 7+  -  Visit the Network tab

The Network tab shows you:

```text
HOSTS CONTACTED IN THE LAST 7 DAYS                  REQUESTS

api.openai.com                                        1,247
*.anthropic.com                                          83
github.com                                               412
pypi.org                                                  29
files.pythonhosted.org                                   161
api.github.com                                          1,053
```

Click "Lock Down". A confirmation modal appears showing the checkboxes for each host. You uncheck anything you don't want. Click Confirm.

Mode flips to **enforce**. The proxy now blocks anything not on the list.

### After lockdown  -  New hosts go to Pending

If your sandbox tries to reach a host that isn't on the allowlist, the request is blocked AND the host appears in the **Pending Requests** section of the Network tab. One click → Allow or Block.

You're never stuck  -  denied requests are visible, attributable, and one click away from resolution.

## SDK  -  Network controls

**Python**

```python
# Allow a host (audit-only or enforce  -  doesn't matter; allow is allow)
sandbox.network.allow("api.openai.com")
sandbox.network.allow("*.anthropic.com", methods=["POST"])

# Explicit deny  -  overrides any allow
sandbox.network.deny("169.254.169.254")

# Switch to enforce mode using the current allowlist
sandbox.network.lockdown()

# Switch back to audit-only
sandbox.network.observe()

# Auto-suggested allowlist from observed traffic
hosts = sandbox.network.suggestions(since="7d")
for h in hosts:
    print(h.host, h.request_count, h.last_seen)

# Inspect the current policy
policy = sandbox.network.get_policy()
print(policy.mode, policy.allow_rules, policy.deny_rules)

# Pending blocked requests
pending = sandbox.network.pending()
for p in pending:
    print(p.host, p.request_count, "blocked")
```

**TypeScript**

```typescript
await sandbox.network.allow('api.openai.com');
await sandbox.network.allow('*.anthropic.com', { methods: ['POST'] });
await sandbox.network.deny('169.254.169.254');
await sandbox.network.lockdown();
await sandbox.network.observe();

const hosts = await sandbox.network.suggestions({ since: '7d' });
const policy = await sandbox.network.getPolicy();
const pending = await sandbox.network.pending();
```

**Go**

```go
sandbox.Network.Allow(ctx, "api.openai.com", miosa.AllowOpts{})
sandbox.Network.Allow(ctx, "*.anthropic.com", miosa.AllowOpts{Methods: []string{"POST"}})
sandbox.Network.Deny(ctx, "169.254.169.254")
sandbox.Network.Lockdown(ctx)
sandbox.Network.Observe(ctx)

hosts, _ := sandbox.Network.Suggestions(ctx, miosa.SuggestionsOpts{Since: "7d"})
policy, _ := sandbox.Network.GetPolicy(ctx)
pending, _ := sandbox.Network.Pending(ctx)
```

**Java**

```java
sandbox.network().allow("api.openai.com");
sandbox.network().allow("*.anthropic.com", AllowOpts.builder().methods(List.of("POST")).build());
sandbox.network().deny("169.254.169.254");
sandbox.network().lockdown();
sandbox.network().observe();

List<HostSuggestion> hosts = sandbox.network().suggestions(
    SuggestionsOpts.builder().since("7d").build());
NetworkPolicy policy = sandbox.network().getPolicy();
List<PendingHost> pending = sandbox.network().pending();
```

**Elixir**

```elixir
{:ok, _} = Miosa.Network.allow(sandbox.id, "api.openai.com")
{:ok, _} = Miosa.Network.allow(sandbox.id, "*.anthropic.com", methods: ["POST"])
{:ok, _} = Miosa.Network.deny(sandbox.id, "169.254.169.254")
{:ok, _} = Miosa.Network.lockdown(sandbox.id)
{:ok, _} = Miosa.Network.observe(sandbox.id)

{:ok, hosts}   = Miosa.Network.suggestions(sandbox.id, since: "7d")
{:ok, policy}  = Miosa.Network.get_policy(sandbox.id)
{:ok, pending} = Miosa.Network.pending(sandbox.id)
```

## Pattern matching

| Kind | Example | Matches |
|---|---|---|
| Exact host | `api.openai.com` | only this hostname |
| Wildcard | `*.openai.com` | any subdomain of openai.com |
| CIDR (IP only) | `10.0.0.0/8` | any IP in this range, only when the request target is an IP literal (not a hostname) |

Precedence at request time:

1. Exact match wins over wildcard
2. Wildcard wins over CIDR
3. Explicit `deny` rules always win over `allow`

## Per-resource vs tenant-wide rules

You can attach allowlist rules at two levels:

- **Per-resource**  -  applies to just one sandbox or computer
- **Tenant-wide**  -  applies to every resource in your tenant unless a resource has its own override

This means you can have a tenant default of "only allow our internal API + npm" and per-sandbox additions on top.

## When to use warn-mode rules

You can mark a rule as `warn_only`:

```python
sandbox.network.allow("api.experimental.example.com", warn_only=True)
```

In enforce mode, a `warn_only` allow rule still lets the request through but flags it in the audit log. Useful for trial-running a new third-party integration before committing to it.

## Rolling back

If lockdown breaks something:

```python
sandbox.network.observe()
```

You're back to audit-only. Allowlist rules are preserved (so you can re-lockdown later) but they're no longer enforcing.

## Real-world example  -  Python data pipeline locked to its upstream services

A Python sandbox that fetches data from a REST API, processes it, and writes results to an S3-compatible endpoint. Locked down to exactly those three hosts.

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
sandbox = client.sandboxes.create(template_id="python-ds")

# Store credentials in the vault
sandbox.secrets.set(name="data_api_key", value=os.environ["DATA_API_KEY"], expose_as_env="DATA_API_KEY")
sandbox.secrets.set(name="s3_key", value=os.environ["S3_SECRET_KEY"], expose_as_env="AWS_SECRET_ACCESS_KEY")

# Let it run for a minute to observe traffic
import time
pipeline = """
import os, requests, boto3

resp = requests.get(
    "https://data.example.com/api/v1/events?limit=100",
    headers={"X-Api-Key": os.environ["DATA_API_KEY"]},
)
events = resp.json()

# Process…
summary = {"count": len(events), "total_value": sum(e["value"] for e in events)}

s3 = boto3.client(
    "s3",
    endpoint_url="https://storage.example.com",
    aws_access_key_id="access",
    aws_secret_access_key=os.environ["AWS_SECRET_ACCESS_KEY"],
)
import json
s3.put_object(Bucket="results", Key="summary.json", Body=json.dumps(summary))
print("Done:", summary)
"""
sandbox.files.write("/workspace/pipeline.py", pipeline)
sandbox.exec("pip install boto3 -q")
sandbox.exec("python /workspace/pipeline.py")   # observe this run

# Now lock down based on what we observed
suggestions = sandbox.network.suggestions(since="1h")
print("Observed hosts:")
for h in suggestions:
    print(f"  {h.host} ({h.request_count} requests)")

# Allow only the three we expect
sandbox.network.allow("data.example.com")
sandbox.network.allow("storage.example.com")
sandbox.network.allow("pypi.org")          # for the pip install above
sandbox.network.allow("files.pythonhosted.org")
sandbox.network.lockdown()
print("Locked. Mode:", sandbox.network.get_policy().mode)

# Run again  -  still works, nothing else reachable
result = sandbox.exec("python /workspace/pipeline.py")
print(result.stdout)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.create({ template: 'python-ds' });

await sandbox.secrets.set({ name: 'data_api_key', value: process.env.DATA_API_KEY!, exposeAsEnv: 'DATA_API_KEY' });

// Observe one run, then lock down
const suggestions = await sandbox.network.suggestions({ since: '1h' });
for (const h of suggestions) {
  await sandbox.network.allow(h.host);
}
await sandbox.network.lockdown();
console.log('Locked to:', suggestions.map(h => h.host).join(', '));
```

## Reuse a policy across sandboxes

Export a locked policy and apply it to a new sandbox:

```python
# Export the policy from a locked sandbox
policy = sandbox.network.get_policy()

# Apply it to a new sandbox
new_sandbox = client.sandboxes.create(template_id="python-ds")
for rule in policy.allow_rules:
    new_sandbox.network.allow(rule.host, methods=rule.methods)
for rule in policy.deny_rules:
    new_sandbox.network.deny(rule.host)
new_sandbox.network.lockdown()
print(f"Policy cloned to {new_sandbox.id}.")
```

For the same policy on every new sandbox, apply rules at the tenant level with `client.network.allow(...)` and `client.network.lockdown()`; they become the default for all resources unless a sandbox overrides them.

> **Caution: Templates carry the policy**
>
> A sandbox created from a template inherits the template's network policy. If the template was snapshotted in enforce mode, new sandboxes start locked down. Call `new_sandbox.network.observe()` right after create to return it to audit-only.

## Full script

End-to-end: observe, review, lock down, and check the pending queue.

```python
#!/usr/bin/env python3
"""lockdown_workflow.py  -  audit-only to enforce in one pass"""

import os, time
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
sandbox = client.sandboxes.create(template_id="python-ds")
print(f"Sandbox {sandbox.id}  -  mode: {sandbox.network.get_policy().mode}")

# 1. Observe (use 7d in production)
time.sleep(10)

# 2. Review observed hosts
suggestions = sandbox.network.suggestions(since="1h")
for h in suggestions:
    print(f"  {h.host}  ({h.request_count} requests)")

# 3. Lock down with the observed hosts as the allowlist
for h in suggestions:
    sandbox.network.allow(h.host)
sandbox.network.lockdown()
print(f"Locked. Mode: {sandbox.network.get_policy().mode}")

# 4. Check what is still being blocked
pending = sandbox.network.pending()
for p in pending:
    print(f"  {p.host}  ({p.request_count} blocked)")
```

## Troubleshooting

**`pip install` fails after lockdown  -  "Could not find a version that satisfies the requirement".**
Three domains are needed: `pypi.org`, `files.pythonhosted.org`, and `pythonhosted.org`. Allow all three before locking down.

**An agent stopped working after lockdown and I cannot tell what is blocked.**
Open the Network tab -> Pending Requests, or call `sandbox.network.pending()`. Each blocked row has a `rejected_by` field naming the rule that fired.

**`npm install` fails after lockdown.**
Allow `registry.npmjs.org`. Some packages also fetch from `*.github.com` during postinstall scripts  -  check the Pending queue for exact hosts.

**My sandbox is in enforce mode but I see unexpected hosts in the audit log.**
Audit-only log entries always appear even in enforce mode (blocked calls still get logged with `action=reject`). Filter by `action=reject` to see what's being blocked.

**I want the same policy on every new sandbox without configuring each one.**
Apply rules at the tenant level using `client.network.allow(...)` and `client.network.lockdown()`  -  these become the default for all resources in the tenant unless overridden per sandbox.

**After `observe()`, my rules are gone when I call `lockdown()` again.**
Rules are never deleted by `observe()`. If they appear missing, check `sandbox.network.get_policy().allow_rules`  -  they'll be listed there. The only way rules disappear is if you called `sandbox.network.clear_rules()` explicitly.

For more issues, see the [Security troubleshooting hub](/docs/security/troubleshooting).

## Frequently asked

**Will lockdown block npm install or pip install?**
Only if you don't allow `pypi.org`, `files.pythonhosted.org`, `registry.npmjs.org`, etc. The 7-day audit data will have already shown you these  -  they're on the suggested list.

**My agent suddenly stopped working after I locked down.**
Open the Network tab → Pending Requests. You'll see exactly what got blocked. One-click allow, or revert with `observe()`.

**Can I share allowlists across sandboxes?**
Yes  -  apply them at the tenant or workspace scope, or copy a "policy template" from another resource (Phase 5).

**Does lockdown affect inbound (preview) traffic?**
No. Lockdown is for outbound only. Preview URLs into your sandbox keep working exactly as before.
