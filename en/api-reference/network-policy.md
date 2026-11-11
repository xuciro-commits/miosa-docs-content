---
slug: "api-reference/network-policy"
title: "Network Policy API"
description: "API reference for configuring per-computer network access controls on MIOSA computers."
section: "API reference"
order: 12054
---

Network policies control what external hosts and ports a computer can reach. Use them to sandbox untrusted workloads, enforce egress restrictions, or prevent AI agents from making unexpected outbound connections.

Base path: `/api/v1/computers/{id}/network-policy`

> **Note**
>
> Network policies apply at the microVM level. Changing a policy takes effect within seconds without restarting the computer.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Restrict to only allow HTTPS to GitHub and PyPI
await client.networkPolicy.update(computerId, {
  mode: 'allowlist',
  rules: [
    { host: 'github.com',  port: 443, protocol: 'tcp' },
    { host: 'pypi.org',    port: 443, protocol: 'tcp' },
    { host: 'files.pythonhosted.org', port: 443, protocol: 'tcp' },
  ],
});
```
```bash
curl -X PUT https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "allowlist",
    "rules": [
      { "host": "github.com", "port": 443, "protocol": "tcp" }
    ]
  }'
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/computers/{id}/network-policy` | Get current network policy |
| `PUT` | `/computers/{id}/network-policy` | Set network policy (replaces existing) |
| `DELETE` | `/computers/{id}/network-policy` | Reset to default (unrestricted) |

---

## Get Network Policy

**`GET /api/v1/computers/{id}/network-policy`**

### Response  -  `200 OK`
```json
{
  "data": {
    "computer_id": "...",
    "mode": "unrestricted",
    "rules": [],
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### Policy Modes

| Mode | Description |
|------|-------------|
| `unrestricted` | Default. All outbound traffic allowed |
| `allowlist` | Only listed hosts/ports permitted |
| `denylist` | All traffic permitted except listed rules |
| `isolated` | All outbound blocked (DNS and MIOSA internal excluded) |
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Set Network Policy

**`PUT /api/v1/computers/{id}/network-policy`**

Replaces the entire policy. Partial updates are not supported  -  send all rules on every PUT.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mode` | string | Yes | `"unrestricted"`, `"allowlist"`, `"denylist"`, or `"isolated"` |
| `rules` | array | No | Required when mode is `"allowlist"` or `"denylist"` |

#### Rule Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `host` | string | Yes | Hostname or CIDR block (e.g. `10.0.0.0/8`) |
| `port` | integer | No | Port number. Omit to match all ports |
| `protocol` | string | No | `"tcp"`, `"udp"`, or `"any"` (default) |

### Response  -  `200 OK`

Updated policy object.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `rules required for allowlist/denylist mode` | Empty rules for a filtering mode |
| 404 | `computer not found` | Computer does not exist |
```bash
# Full isolation except for GitHub
curl -X PUT https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "allowlist",
    "rules": [
      { "host": "github.com", "port": 443, "protocol": "tcp" },
      { "host": "objects.githubusercontent.com", "port": 443, "protocol": "tcp" }
    ]
  }'
```

---

## Reset to Default

**`DELETE /api/v1/computers/{id}/network-policy`**

Removes any custom policy. The computer reverts to unrestricted outbound access.

### Response  -  `200 OK`
```json
{
  "data": { "computer_id": "...", "mode": "unrestricted" }
}
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/network-policy \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Common Recipes

### Sandbox an untrusted AI task
```python
# Lock down before running untrusted code
client.network_policy.update(computer_id, mode="isolated")

# Run the task
result = client.exec(computer_id, command="python3 /workspace/untrusted.py")

# Restore after
client.network_policy.delete(computer_id)
```

### Allow only package registries for a build environment
```typescript
const packageHosts = [
  { host: 'registry.npmjs.org', port: 443, protocol: 'tcp' as const },
  { host: 'pypi.org',           port: 443, protocol: 'tcp' as const },
  { host: 'files.pythonhosted.org', port: 443, protocol: 'tcp' as const },
  { host: 'pkg.go.dev',         port: 443, protocol: 'tcp' as const },
  { host: 'proxy.golang.org',   port: 443, protocol: 'tcp' as const },
];

await client.networkPolicy.update(computerId, {
  mode: 'allowlist',
  rules: packageHosts,
});
```

### Block a known malicious IP range
```typescript
await client.networkPolicy.update(computerId, {
  mode: 'denylist',
  rules: [
    { host: '185.220.0.0/16' },
  ],
});
```
