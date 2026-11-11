---
slug: "api-reference/sandboxes"
title: "Sandboxes API"
description: "API reference for creating, managing, and executing code in MIOSA sandboxes - persistent-by-default isolated microVM workspaces."
section: "API reference"
order: 12014
---

Sandboxes are persistent-by-default isolated microVM workspaces that restore from pre-seeded snapshots, accept exec and file operations, expose previews, and are billed by compute/runtime usage. They are designed for AI-agent code execution workloads, artifact generation, and app previews.

Base path: `/api/v1/sandboxes`

> **Note**
>
> All endpoints require `Authorization: Bearer &lt;api_key&gt;`. Get a key from the [MIOSA dashboard](https://miosa.ai/settings/api-keys).

For higher-level access, use the official SDKs: [Python](/docs/sdks/python), [TypeScript](/docs/sdks/typescript), [Go](/docs/sdks/go), [Java](/docs/sdks/java), [Elixir](/docs/sdks/elixir).

- [Create and run](/docs/api-reference/sandboxes/create): Create a sandbox, the fused run endpoint, template apps, artifacts, and custom templates.
- [Manage](/docs/api-reference/sandboxes/manage): List, get, destroy, pause, resume, fork, extend the timeout, and read usage.

---

## List sandbox templates and sizes

**`GET /api/v1/templates`**

This is the canonical catalog for sandbox template IDs, template defaults, per-template size support, readiness, and exact versioned resource contracts.
Clients need no secondary catalog endpoint to discover or admit a sandbox configuration.
```bash
curl https://api.miosa.ai/api/v1/templates
```

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "miosa-sandbox",
      "name": "MIOSA Sandbox",
      "product": "sandbox",
      "default_size": "small",
      "primitive": "sandbox",
      "sdk_name": "miosa-sandbox",
      "cli_name": "miosa-sandbox",
      "readiness": "fast_ready",
      "sizes": [
        {
          "size": "small",
          "state": "fast_ready",
          "resource_contract": {
            "contract_id": "sandbox/small@v1",
            "contract_version": "v1",
            "product": "sandbox",
            "size": "small",
            "vcpus": 2,
            "memory_mb": 4096,
            "disk_size_mb": 10240
          }
        }
      ]
    }
  ],
  "templates": [
    {
      "id": "miosa-sandbox",
      "name": "MIOSA Sandbox",
      "product": "sandbox",
      "default_size": "small",
      "primitive": "sandbox",
      "sdk_name": "miosa-sandbox",
      "cli_name": "miosa-sandbox",
      "readiness": "fast_ready",
      "sizes": [
        {
          "size": "small",
          "state": "fast_ready"
        }
      ]
    }
  ],
  "shape_contracts": {
    "sandbox": [],
    "computer": []
  },
  "readiness_states": ["fast_ready", "cold_boot_only", "missing"]
}
```

`fast_ready` sizes are creatable through the fast start path.
`cold_boot_only` sizes are also creatable but start more slowly.
`missing` means unavailable and must not be submitted.
Use that template's `default_size` when the caller does not have a stronger requirement.
Do not combine a size advertised by one template with another template.

---

## Configuration and request parameters

`GET /api/v1/templates` returns platform configuration: available templates, the default size, readiness, and versioned resource contracts.
Applications select that configuration with sandbox parameters; they do not send internal host, provider, network, or placement fields.

Prefer `size` for create requests.
The default `small` contract is 2 vCPU, 4096 MiB RAM, and 10240 MiB disk.
`xs` is selectable when the catalog reports it as available.
Compatibility resource fields must include `cpu_count`, `memory_mb`, and `disk_size_mb` together and exactly match one published contract.

Responses include the resolved contract under `resource_contract`.
Contract IDs are versioned, for example `sandbox/small@v1`; persist that identity when usage or audit records must remain reproducible.

---

## Sandbox Object

The full sandbox object returned by create, get, and list endpoints:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique sandbox identifier. |
| `tenant_id` | string | Owning tenant ID. |
| `owner_id` | string | Creating user ID. |
| `template_id` | string | Boot template name. |
| `state` | string | Current lifecycle state. |
| `size` | string | Canonical named size: `xs`, `small`, `medium`, `large`, or `xl`. |
| `resource_contract` | object | Versioned resolved contract with ID, product, size, vCPU, memory, and disk. |
| `cpu_count` | integer | Allocated vCPUs. |
| `memory_mb` | integer | Allocated RAM in MB. |
| `disk_size_mb` | integer | Allocated root disk in MB. |
| `ready` | boolean | Whether command and readiness checks are accepting work. |
| `ready_at` | string/null | ISO timestamp when `ready` became true. |
| `preview_url` | string/null | Tenant-aware base preview URL for the sandbox slug. Use `/expose` for a specific port. |
| `timeout_sec` | integer | Max seconds for the active session. Persistent timeout pauses; non-persistent timeout destroys. |
| `timeout_remaining_ms` | integer \| null | Remaining active-session time; null before start or when timeout enforcement is disabled. |
| `always_on` | boolean | Whether timeout and idle-timeout enforcement are disabled. |
| `persistent` | boolean | Whether the sandbox preserves filesystem state across stop/timeout. |
| `idle_timeout_sec` | integer | Seconds without activity before the sandbox pauses. `0` disables it; accounts created on or after October 11, 2026 default to `1800` (30 minutes) when unset. |
| `total_runtime_sec` | integer \| null | Billed seconds. Set only after destruction. |
| `metadata` | object | Caller-supplied metadata. |
| `created_at` | string | ISO-8601 creation timestamp. |
| `started_at` | string \| null | ISO-8601 timestamp when VM entered `running`. |
| `destroyed_at` | string \| null | ISO-8601 timestamp when VM was destroyed. |

---

## Lifecycle States

creating goes to running on ready; running goes to stopped on stop, and back to running on resume.

running goes to paused on pause, and back to running on resume.

running goes to destroyed on destroy.

any state goes to error on failure (terminal).

| State | Description |
|-------|-------------|
| `provisioning` | VM is booting. Exec and file operations are not yet available. |
| `running` | VM is reachable. All operations allowed. |
| `paused` | Compute is stopped and filesystem state is preserved. Can be resumed. |
| `destroyed` | Terminal. VM is gone, billing is settled. |
| `error` | Terminal failure. Destroy and create a new sandbox. |

---

## Error Response Format

All error responses use this shape:
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "sandbox not found",
    "details": null
  }
}
```

The `x-request-id` response header is set on every request. Include it in support requests.

---

## Code interpreter

**`POST /api/v1/sandboxes/{id}/code/run`** (alias: `POST /api/v1/sandboxes/{id}/code`)

Runs one cell in a persistent interpreter and returns its output.
The kernel starts lazily on first use, and variables, imports and definitions persist across calls in the same context.
`language` is `python` (the default) or `javascript`; an unsupported value returns `400 UNKNOWN_LANGUAGE`.
`context` names an independent namespace, `default` unless you pass one; `timeout` is capped at 300 seconds.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/code/run \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "code": "total = sum(range(10))\ntotal", "language": "python" }'
```

```json
{
  "data": {
    "ok": true,
    "stdout": "",
    "stderr": "",
    "result": "45",
    "language": "python"
  }
}
```

| Endpoint | What it does |
|---|---|
| `GET /api/v1/sandboxes/{id}/code` | Report interpreter state. Never starts a kernel. Returns `sandbox_not_running` when the sandbox is not running. |
| `GET /api/v1/sandboxes/{id}/code/contexts?language=` | List the contexts that exist for a language. |
| `POST /api/v1/sandboxes/{id}/code/contexts` | Create or reuse a context. |
| `DELETE /api/v1/sandboxes/{id}/code/contexts/{context}` | Reset one context. |
| `DELETE /api/v1/sandboxes/{id}/code?language=` | Reset every context for a language. |

Reading (`GET`) needs `sandboxes:read`; running and resetting need `sandboxes:exec`.
The sandbox must be running: the interpreter does not wake a paused sandbox.

---

## Rate Limits

| Limit | Value |
|-------|-------|
| Request rate | Enforced per workspace and per endpoint group; an over-limit response carries `Retry-After` |
| Concurrent sandboxes per tenant | Plan cap: 10 (Developer), 250 (Business), 500 (Enterprise) |
| `timeout_sec` | 1-86400 seconds; default 3600 |
| Default resource shape | `small`: 2 vCPU, 4 GiB RAM, 10 GiB disk |
| Named resource schema | `xs` through `xl`; actual availability comes from `GET /templates` and tenant policy |
| Max exec timeout | 1-86400 seconds; default 3600 |
| Max file upload size | 500 MB |

Rate-limited responses return HTTP `429` with a `Retry-After` header indicating seconds to wait.

---

## See also

- [Python SDK](/docs/sdks/python)  -  `miosa`
- [TypeScript SDK](/docs/sdks/typescript)  -  `@miosa/sdk`
- [Go SDK](/docs/sdks/go)  -  `github.com/miosa-ai/miosa-go`
- [Java SDK](/docs/sdks/java)  -  `ai.miosa:miosa-sdk`
- [Elixir SDK](/docs/sdks/elixir)  -  `:miosa`
- [Events (SSE)](/docs/api-reference/events)  -  SSE reference for other MIOSA resources
- [Error Codes](/docs/api-reference/errors)  -  full error code catalog including sandbox-specific codes
