---
slug: "develop/sandboxes/internet-access"
title: "Internet access and egress"
description: "Whether a sandbox can reach the internet by default, the two layers of egress control, how to lock a sandbox to an allowlist, and how to debug a blocked call."
section: "Networking & domains"
order: 5002
---

## Default network behavior

- **Inbound.** A sandbox is not reachable from the internet. Only the ports you expose as a [preview](/docs/develop/previews) accept traffic, and only through the MIOSA proxy.
- **Outbound.** A new sandbox can reach the internet. It starts in **audit-only** mode: no outbound call is blocked. Egress audit capture is not yet enabled in the current release, so calls are not recorded yet. A Computer's network policy defaults to `unrestricted`.

So by default your code can install packages, call APIs, and clone repos. Nothing is blocked until you choose to enforce an allowlist.

### What a sandbox cannot reach

A sandbox cannot reach the platform's own internal network.
The only exception is the managed-database proxy range, which is what a managed Postgres, MySQL, or Redis connection uses inside the sandbox.
This is a mesh-level deny, checked before the egress allowlist, so adding an internal host to an allowlist does not open it.
It is enforced on both runtimes.

`169.254.169.254` is not an egress destination on MIOSA.
It is Firecracker's metadata service (MMDS), the guest-side channel the host uses to hand a sandbox its identity and configuration, and it is answered inside the machine rather than on the host network.
Leave it denied, as the examples below do.

## Two layers of control

MIOSA gives you two ways to constrain outbound traffic. They compose.

| Layer | Scope | What it decides |
|---|---|---|
| Egress policy + allowlist | Computers and Sandboxes | The hosts a resource may reach, in `audit_only` or `enforce` mode |
| Network policy | Computers | A microVM-level mode: `unrestricted`, `allowlist`, `denylist`, or `isolated` |

A Computer or Sandbox resolves its egress policy as: resource override, then tenant default. The tenant default starts in `audit_only` mode.

## Watch, then enforce

The intended path for a sandbox is to observe real traffic, then lock it down to what it actually uses. This is the same flow documented in full on the [network allowlist reference](/docs/security/network):

1. **Observe.** Let the sandbox run normally. Every outbound host is recorded.
2. **Review.** Pull the hosts it contacted, ranked by frequency.
3. **Enforce.** Apply those hosts as the allowlist and switch the policy to `enforce`. Unlisted hosts are then blocked.
4. **Operate.** Blocked hosts land in a pending queue with one-click allow or deny. Revert to audit-only at any time without losing the allowlist.

**Python**

```python
# Suggested allowlist from observed traffic
for h in sandbox.network.suggestions(since="7d"):
    print(h.host, h.request_count)

# Allow, deny, and enforce
sandbox.network.allow("api.openai.com")
sandbox.network.allow("*.anthropic.com", methods=["POST"])
sandbox.network.deny("169.254.169.254")   # Firecracker metadata service (MMDS)
sandbox.network.lockdown()

# Revert, or inspect
sandbox.network.observe()
print(sandbox.network.get_policy().mode)
```

**TypeScript**

```typescript
const hosts = await sandbox.network.suggestions({ since: '7d' });
await sandbox.network.allow('api.openai.com');
await sandbox.network.allow('*.anthropic.com', { methods: ['POST'] });
await sandbox.network.deny('169.254.169.254');  // Firecracker metadata service (MMDS)
await sandbox.network.lockdown();
await sandbox.network.observe();
const policy = await sandbox.network.getPolicy();
```

### Allowlist rules

| Kind | Example | Matches |
|---|---|---|
| Exact host | `api.openai.com` | Only this hostname |
| Wildcard | `*.openai.com` | Any subdomain of `openai.com` |
| CIDR | `10.0.0.0/8` | Any IP in this range, when the target is an IP literal |

Rules are matched in priority order: exact beats wildcard, wildcard beats CIDR, and an explicit `deny` always beats an `allow`. A rule can be scoped to a method and a path, and marked `warn_only` to let the request through while flagging it in the audit log.

Rules attach at two levels: per-resource (one sandbox or computer) and tenant-wide (every resource unless overridden).

## Restrict a Computer at the microVM level

Computers also accept a network policy that takes effect within seconds, without a restart.

**TypeScript**

```typescript
await client.networkPolicy.update(computerId, {
  mode: 'allowlist',
  rules: [
    { host: 'registry.npmjs.org', port: 443, protocol: 'tcp' },
    { host: 'pypi.org',          port: 443, protocol: 'tcp' },
  ],
});
```

**curl**

```bash
curl -X PUT https://api.miosa.ai/api/v1/computers/$COMPUTER_ID/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"mode": "allowlist", "rules": [{"host": "github.com", "port": 443, "protocol": "tcp"}]}'
```

| Mode | Description |
|---|---|
| `unrestricted` | Default. All outbound traffic allowed |
| `allowlist` | Only listed hosts and ports permitted |
| `denylist` | All traffic allowed except listed rules |
| `isolated` | All outbound blocked (DNS and MIOSA internal excluded) |

A `PUT` replaces the whole policy; send every rule on every call. `DELETE` resets it to `unrestricted`.

## Credentials on the way out

Egress is also where MIOSA swaps placeholders for real credentials. Your workload sees an opaque `miosa-tok-...` value in its environment; the proxy substitutes the real secret on the outbound request, so the value never lives inside the sandbox. See [Secrets](/docs/security/secrets) and the [Egress API](/docs/api-reference/egress).

## Debug a blocked call

**Python**

```python
# What is being blocked right now
for p in sandbox.network.pending():
    print(p.host, p.request_count, "blocked")

# Or filter the audit log for rejects
for e in sandbox.audit.list(since="1h", action="reject", limit=50):
    print(e.host, e.rejected_by)
```

Common allowlist gaps:

- **`pip install`** needs `pypi.org`, `files.pythonhosted.org`, and `pythonhosted.org`.
- **`npm install`** needs `registry.npmjs.org`; some postinstall scripts also reach `*.github.com`.
- **CDNs and dynamic subdomains** are usually not covered by an exact rule. Use a wildcard like `*.openai.com`.

Every blocked row carries a `rejected_by` field naming the rule that fired. For the full list of fixes, see [Security troubleshooting](/docs/security/troubleshooting).

> **Note: Lockdown does not affect inbound traffic**
>
> Enforcing an allowlist restricts outbound calls only. Preview URLs into your sandbox keep working exactly as before.

## See also

- [Network allowlist reference](/docs/security/network): The watch-then-enforce flow, the SDK methods, and the pending queue.
- [Egress API](/docs/api-reference/egress): Policies, allowlist rules, the audit log, and the vault.
- [Network policy API](/docs/api-reference/network-policy): The microVM-level policy for Computers.
