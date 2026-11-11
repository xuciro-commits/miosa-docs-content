---
slug: "migrating-from-miosa-sandbox"
title: "Migrating from miosa-sandbox"
description: "Move off the deprecated @miosa/sandbox, miosa-sandbox, and sandboxes-go packages onto the canonical MIOSA SDKs."
section: "Optional"
order: 17030
---

The standalone sandbox packages are deprecated compatibility wrappers. Each one now delegates to the canonical MIOSA SDK, so the fastest migration is often to install the canonical package and change one import.

| Deprecated package | Canonical replacement |
|---|---|
| `@miosa/sandbox` (JavaScript) | `@miosa/sdk` |
| `miosa-sandbox` (Python) | `miosa` |
| `sandboxes-go` (Go) | `github.com/Miosa-osa/miosa-go/v2` |

## JavaScript

```bash
npm install @miosa/sdk
```

```typescript
// Before
import { MIOSA } from '@miosa/sandbox';
const client = new MIOSA({ apiKey: process.env.MIOSA_API_KEY! });

// After
import { Miosa } from '@miosa/sdk';
const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
```

`@miosa/sandbox` re-exports the canonical client, so it keeps installing while you migrate, but new code should import `@miosa/sdk` directly. The renamed classes map as:

| Old import | Canonical replacement |
|---|---|
| `MIOSA` | `Miosa` |
| `SandboxClient` | `Sandboxes` |
| `ComputerClient` | `Computers` |
| `CreateSandboxOptions` | `SandboxCreateParams` |
| `CreateComputerOptions` | `ComputerCreateParams` |

## Python

```bash
pip install miosa
```

```python
# Before
from miosa_sandbox import MIOSA
with MIOSA(api_key="msk_...") as client:
    with client.sandboxes.create() as sandbox:
        print(sandbox.exec("python3 -c 'print(1 + 1)'").stdout)

# After
from miosa import Miosa
with Miosa(api_key="msk_...") as client:
    with client.sandboxes.create() as sandbox:
        print(sandbox.exec("python3 -c 'print(1 + 1)'").stdout)
```

`MIOSA` and `AsyncMIOSA` are preserved in the compatibility package and return the canonical `miosa` resource classes; the async class becomes `AsyncMiosa`.

## Go

```bash
go get github.com/Miosa-osa/miosa-go/v2
```

```go
// Before
import sandboxes "github.com/miosa-ai/sandboxes-go"
client, _ := sandboxes.New(sandboxes.Config{APIKey: os.Getenv("MIOSA_API_KEY")})

// After
import miosa "github.com/Miosa-osa/miosa-go/v2"
client := miosa.NewClient("msk_...")
```

> **Note: Sunset window**
>
> The `sandboxes-go` package receives bug fixes but no new features, with a sunset date of 2026-11-08. The JavaScript and Python compatibility packages exist only so older integrations keep installing while they migrate.

## Behavior differences to expect

- **One client, one surface.** In the canonical SDKs, sandboxes live on the client (`client.sandboxes`) alongside computers, files, snapshots, and agent runs. Code that reached for a separate sandbox client now goes through the single client.
- **Ownership selectors are first-class.** `workspace_id`, `project_id`, and `external_user_id` can be passed directly on create, and are exposed on the returned handle.
- **`size` and `resource_contract`.** When no size is supplied, the compatibility packages request the canonical `small` contract: 2 vCPU, 4096 MiB memory, and 10240 MiB disk. The response exposes `size`, `resource_contract`, and `usage()` on both sync and async handles.
- **Canonical names.** Use `client.sandboxes` on the sandbox image, `client.computers` with `miosa-desktop`, and `client.dockerDeploy` for the Docker Host appliance. `computer` is the user-facing product name; `miosa-desktop` is the template it boots from.

## Verify the migration

The compatibility packages and the canonical SDKs hit the same API, so the check is behavioral:

```python
from miosa import Miosa

with Miosa(api_key="msk_...") as client:
    sbx = client.sandboxes.create(template_id="miosa-sandbox", size="small")
    print(sbx.state, sbx.resource_contract)
    print(sbx.exec("python3 -c 'print(1 + 1)'").stdout)   # 2
    sbx.destroy()
```

If that prints `2` and a `small` contract, the migration is complete.

## See also

- [TypeScript SDK](/docs/sdks/typescript): The canonical JavaScript and TypeScript client.
- [Python SDK](/docs/sdks/python): The canonical Python client, sync and async.
- [Go SDK](/docs/sdks/go): The canonical Go client, version 2.
