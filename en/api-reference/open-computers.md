---
slug: "api-reference/open-computers"
title: "OpenComputers API"
description: "Register and control customer-owned machines, including on-premises servers, via the MIOSA API - exec, files, tunnels, AI agents, inference clusters, and secrets."
section: "API reference"
order: 12031
---

OpenComputers lets you register physical or virtual machines you already own (Mac, Linux, Windows) and control them through MIOSA's API  -  run commands, manage files, expose HTTP tunnels, dispatch AI agents, build inference clusters, and manage secrets.

Hosts may be on-premises servers, workstations, or existing machines at a hosting provider.
Available operations depend on the installed agent and the host's capabilities.
See [OpenComputers](/docs/opencomputers) for hardware, connectivity, and data responsibilities.

Base path: `/api/v1/opencomputers`

Verbs supported: **GET** (list/show), **POST** (register/create/exec/dispatch), **PATCH** (update tags/tunnels), **DELETE** (revoke/cancel).

> **Caution**
>
> Rate limits vary by key, tenant policy, and endpoint family. Honor `X-RateLimit-*` and `Retry-After` response headers. Exec jobs and agent sessions run asynchronously  -  poll or stream for results rather than holding the connection.

> **Note**
>
> Register the host and follow the generated instructions to connect the supported OpenComputers agent on the target machine. The **host key** returned on registration is shown exactly
> once  -  store it securely.

---

## Hosts

### List hosts

**`GET /api/v1/opencomputers/hosts`**

Returns an object with a `hosts` array.
Inspect each host's `id`, `state`, `capabilities`, and `last_heartbeat` before selecting it.
Use the returned ID exactly; examples below use placeholders.

```json
{
  "hosts": [
    {
      "id": "<host-id>",
      "name": "lab-server",
      "state": "active",
      "capabilities": ["exec"],
      "last_heartbeat": "2026-09-08T12:00:00Z"
    }
  ]
}
```

This is an abbreviated response.
A recent heartbeat and a completed operation are stronger evidence of readiness than a host record alone.

---

### Register a host

**`POST /api/v1/opencomputers/hosts`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Display name for the host |
| `region` | string | No | Region label |
| `platform` | string | No | Host operating-system selection |

Registration returns `{ host, host_key, control_url, warning }`.
The `host` object contains the new host ID; `host_key` authenticates the installed agent.
Use your platform API key for REST requests, not the host key.

> **Caution**
>
> `host_key` is returned **only once** at registration time. The host agent uses
> it to authenticate. Store it immediately  -  it cannot be retrieved later.

---

### Get a host

**`GET /api/v1/opencomputers/hosts/{id}`**

---

### Revoke a host

**`DELETE /api/v1/opencomputers/hosts/{id}`**

Returns `204 No Content`. The host agent will no longer be able to authenticate.

---

### Host event stream (SSE)

**`GET /api/v1/opencomputers/hosts/{id}/events`**

Streams real-time lifecycle events from the host as Server-Sent Events.
```text

event: status_change
data: {"type":"status_change","host_id":"host_abc","data":{"status":"online"},"timestamp":"..."}
```

---

## Jobs

Run shell commands on a registered host and retrieve output.

### Run a job

**`POST /api/v1/opencomputers/hosts/{id}/exec`**

| Field | Type | Required | Description |
|---|---|---|---|
| `cmd` | string | Yes | Command to execute |
| `args` | string[] | No | Arguments passed separately |
| `env` | string[] | No | Environment entries in `KEY=VALUE` format |
| `cwd` | string | No | Working directory |
| `timeout_ms` | integer | No | Timeout in milliseconds; default 30000 |
| `stream` | boolean | No | Default `true`: stream SSE; `false`: collect output as JSON |

With `stream: true`, the POST response carries `exec_chunk`, `exec_result`, or `exec_error` events.
Keep the connection open until the terminal result.
With `stream: false`, a completed command returns:

```json
{
  "job_id": "<job-id>",
  "exit_code": 0,
  "stdout": "lab-server\n",
  "stderr": ""
}
```

A timeout or disconnected client is not proof that the process stopped.
Inspect the job and host before retrying an operation with side effects.

---

### List jobs

**`GET /api/v1/opencomputers/hosts/{id}/jobs`**

---

### Get a job

**`GET /api/v1/opencomputers/hosts/{id}/exec/{job_id}`**

---

### Stream job output (SSE)

**`GET /api/v1/opencomputers/hosts/{id}/exec/{job_id}/stream`**

Streams stdout/stderr in real time while the job is running.

---

### Cancel a job

**`DELETE /api/v1/opencomputers/hosts/{id}/exec/{job_id}`**

Returns `204 No Content`.

---

## File System

Manage files and directories on a registered host.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/opencomputers/hosts/{id}/fs/list` | GET | List directory (`?path=`) |
| `/opencomputers/hosts/{id}/fs/stat` | GET | Stat a path (`?path=`) |
| `/opencomputers/hosts/{id}/fs/download` | GET | Download a file (`?path=`) |
| `/opencomputers/hosts/{id}/fs/upload` | POST | Upload a file (multipart, `?path=`) |
| `/opencomputers/hosts/{id}/fs/delete` | DELETE | Delete file/dir (`?path=`) |
| `/opencomputers/hosts/{id}/fs/mkdir` | POST | Create directory (`{"path":"..."}`) |

**List response:**
```json
{
  "path": "/workspace/projects",
  "entries": [
    {
      "name": "my-app",
      "path": "/workspace/projects/my-app",
      "size": 0,
      "is_dir": true,
      "modified_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

## Terminal

### Issue a terminal ticket

**`POST /api/v1/opencomputers/hosts/{id}/terminal/ticket`**

Returns a short-lived WebSocket authentication ticket. Connect to `ws_url`
immediately using the ticket as a query parameter.
```json
{
  "ticket": "tk_abc123",
  "ws_url": "wss://api.miosa.ai/opencomputers/ws/terminal?ticket=tk_abc123",
  "expires_at": "2026-01-01T00:00:30Z"
}
```

---

## Desktop (VNC)

### Issue a desktop ticket

**`POST /api/v1/opencomputers/hosts/{id}/desktop/ticket`**

Same shape as the terminal ticket. Connect to `ws_url` with the ticket to
start a VNC session.

---

## Tunnels

Expose local ports on the host over MIOSA-managed public URLs.

### List tunnels

**`GET /api/v1/opencomputers/hosts/{id}/tunnels`**

### Create a tunnel

**`POST /api/v1/opencomputers/hosts/{id}/tunnels`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target_port` | integer | Yes | Local port to expose |
| `auth_mode` | string | No | `public` \| `tenant_only` \| `password` (default: `tenant_only`) |
| `slug` | string | No | Custom slug for the public URL |
```json
{
  "id": "tun_abc",
  "host_id": "host_abc123",
  "slug": "my-app-dev",
  "target_port": 3000,
  "auth_mode": "tenant_only",
  "public_url": "https://api.miosa.ai/t/my-app-dev",
  "enabled": true,
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

### Get / Update / Delete a tunnel

**`GET /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}`**

**`PATCH /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}`**

| Field | Type | Description |
|-------|------|-------------|
| `target_port` | integer | Change target port |
| `auth_mode` | string | Change auth mode |
| `enabled` | boolean | Enable or disable the tunnel |

**`DELETE /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}`**  -  `204 No Content`

---

## Agents

Dispatch an AI agent to complete a task autonomously on the host.

Dispatch over the API with `miosa api`:

```bash
miosa api POST /opencomputers/hosts/host_abc123/agent/dispatch \
  -d '{"task": "Audit the local browser workflow"}'
```

### Dispatch an agent

**`POST /api/v1/opencomputers/hosts/{id}/agent/dispatch`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `task` | string | Yes | Natural-language task description |
| `model` / `model_id` | string | No | Override the default runtime/model |
| `tools` | string[] | No | Tool names to expose to the agent session |
| `budget.max_steps` | integer | No | Maximum agent steps (default: 30) |
| `budget.max_tokens` | integer | No | Maximum token budget (default: 100000) |
| `budget.timeout_ms` | integer | No | Session timeout in milliseconds |
| `agent_runtime_profile_id` | UUID/string | No | Force a specific runtime profile |
| `skip_agent_runtime_profile` | boolean | No | Do not apply the tenant/workspace default profile |
| `context` | object | No | Additional context key-value pairs |

```json
{
  "id": "sess_abc",
  "session_id": "sess_abc",
  "host_id": "host_abc123",
  "task": "Run the test suite and fix any failing tests",
  "status": "running",
  "sse_url": "/api/v1/opencomputers/hosts/host_abc123/agent/sessions/sess_abc/events",
  "optimal_session_id": "opt_sess_123",
  "agent_runtime_profile_id": "arp_123",
  "runtime_context": {
    "agent_runtime_profile": {
      "id": "arp_123",
      "runtime": "claude-code",
      "tools": ["browser", "filesystem"],
      "env_keys": ["ANTHROPIC_API_KEY"]
    }
  }
}
```

**Session statuses:** `running` | `succeeded` | `failed` | `canceled`

### List / Get sessions

**`GET /api/v1/opencomputers/hosts/{id}/agent/sessions`**

**`GET /api/v1/opencomputers/hosts/{id}/agent/sessions/{session_id}`**

### Stream agent events (SSE)

**`GET /api/v1/opencomputers/hosts/{id}/agent/sessions/{session_id}/events`**

### Cancel a session

**`DELETE /api/v1/opencomputers/hosts/{id}/agent/sessions/{session_id}`**  -  `204 No Content`

---

## Inference Clusters

Group multiple hosts to serve an LLM over an OpenAI-compatible endpoint.

### List clusters

**`GET /api/v1/opencomputers/clusters`**

### Create a cluster

**`POST /api/v1/opencomputers/clusters`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Cluster name |
| `model` | string | Yes | Model to serve (e.g. `llama3:70b`) |
| `host_ids` | string[] | Yes | IDs of hosts in the cluster |
```json
{
  "id": "cl_abc",
  "name": "my-cluster",
  "model": "llama3:70b",
  "slug": "my-cluster",
  "status": "active",
  "host_ids": ["host_abc123"],
  "inference_url": "https://api.miosa.ai/inference/my-cluster/v1",
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

The `inference_url` is OpenAI-compatible:
```http

POST {inference_url}/chat/completions
```

### Get / Start / Stop / Delete

**`GET /api/v1/opencomputers/clusters/{id}`**

**`POST /api/v1/opencomputers/clusters/{id}/start`**

**`POST /api/v1/opencomputers/clusters/{id}/stop`**

**`DELETE /api/v1/opencomputers/clusters/{id}`**  -  `204 No Content`

---

## Secrets

Store encrypted key-value secrets accessible to the host agent at runtime.

### Tenant-scoped secrets

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/opencomputers/secrets` | GET | List tenant secrets |
| `/opencomputers/secrets` | POST | Create a secret |
| `/opencomputers/secrets/{id}` | PATCH | Update value or description |
| `/opencomputers/secrets/{id}` | DELETE | Delete a secret |

### Host-scoped secrets

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/opencomputers/hosts/{id}/secrets` | GET | List host secrets |
| `/opencomputers/hosts/{id}/secrets` | POST | Create a host secret |
| `/opencomputers/hosts/{id}/secrets/{secret_id}` | DELETE | Delete a host secret |

**Create request:**
```json
{ "name": "GITHUB_TOKEN", "value": "ghp_xxx", "description": "CI token" }
```

**Response (value is never returned):**
```json
{
  "id": "sec_abc",
  "name": "GITHUB_TOKEN",
  "description": "CI token",
  "host_id": null,
  "tenant_id": "t_abc",
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

---

## SDK Examples

**TypeScript**

```typescript
import Miosa from 'miosa';

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY });

// Register a host  -  save host_key immediately
const host = await miosa.openComputers.hosts.create({ name: 'my-mac' });
console.log('Host key (save this!):', host.host_key);

// Run a command
const job = await miosa.openComputers.jobs.run(host.id, {
  command: 'npm test',
});
console.log('Exit code:', job.exit_code);
console.log('Output:', job.stdout);

// Expose a local dev server
const tunnel = await miosa.openComputers.tunnels.create(host.id, {
  target_port: 3000,
});
console.log('Public URL:', tunnel.public_url);

// Dispatch an AI agent
const session = await miosa.openComputers.agents.dispatch(host.id, {
  task: 'Run the test suite and fix any failing tests',
  max_turns: 30,
});
console.log('Session:', session.id, session.status);
```

**Python**

```python
import miosa

client = miosa.Miosa(api_key="msk_u_...")

# Register a host
host = client.open_computers.hosts.create(
    miosa.resources.open_computers.types.HostCreateParams(name="my-mac")
)
print("Host key:", host.host_key)

# Run a command
from miosa.resources.open_computers.types import JobRunParams
job = client.open_computers.jobs.run(host.id, JobRunParams(command="npm test"))
print(f"Exit code: {job.exit_code}, stdout: {job.stdout}")

# Create a tunnel
from miosa.resources.open_computers.types import TunnelCreateParams
tunnel = client.open_computers.tunnels.create(host.id, TunnelCreateParams(target_port=3000))
print("Public URL:", tunnel.public_url)
```

**Go**

```go
client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))

// Register a host
host, err := client.OpenComputers.Hosts.Create(ctx, miosa.CreateHostInput{
    Name: "my-mac",
})
if err != nil { log.Fatal(err) }
fmt.Println("Host key:", *host.HostKey)

// Run a command
job, err := client.OpenComputers.Jobs.Run(ctx, host.ID, miosa.RunJobInput{
    Command: "npm test",
})
if err != nil { log.Fatal(err) }
fmt.Printf("Exit code: %d\nStdout: %s\n", *job.ExitCode, *job.Stdout)

// Create a tunnel
tunnel, err := client.OpenComputers.Tunnels.Create(ctx, host.ID, miosa.CreateTunnelInput{
    TargetPort: 3000,
})
fmt.Println("Public URL:", tunnel.PublicURL)
```

**Elixir**

```elixir
client = Miosa.client("msk_u_...")

# Register a host
{:ok, host} = Miosa.OpenComputers.Hosts.create(client, %{name: "my-mac"})
IO.puts("Host key: #{host["host_key"]}")

# Run a command
{:ok, job} = Miosa.OpenComputers.Jobs.run(client, host["id"], %{command: "npm test"})
IO.puts("Exit code: #{job["exit_code"]}")

# Create a tunnel
{:ok, tunnel} = Miosa.OpenComputers.Tunnels.create(client, host["id"], %{target_port: 3000})
IO.puts("Public URL: #{tunnel["public_url"]}")
```

**Java**

```java
MiosaClient miosa = new MiosaClient(System.getenv("MIOSA_API_KEY"));

// Register a host
HostData host = miosa.openComputers().hosts()
    .create(new CreateHostParams("my-mac"));
System.out.println("Host key: " + host.hostKey);

// Run a command
JobData job = miosa.openComputers().jobs()
    .run(host.id, new RunJobParams("npm test"));
System.out.printf("Exit code: %d%nStdout: %s%n", job.exitCode, job.stdout);

// Create a tunnel
TunnelData tunnel = miosa.openComputers().tunnels()
    .create(host.id, new CreateTunnelParams(3000));
System.out.println("Public URL: " + tunnel.publicUrl);
```

---

## Common Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `NOT_FOUND` | Host or resource does not exist in this tenant |
| 403 | `FORBIDDEN` | Authenticated but not authorized |
| 400 | `INVALID_ID` | Path parameter is not a valid ID |
| 409 |  -  | Host is offline; command cannot be dispatched |
| 502 |  -  | Host agent is unreachable |

---

## See also

- [Computers API](/docs/api-reference/computers)  -  managed MIOSA VMs vs. BYOC hosts
- [Exec API](/docs/api-reference/exec)  -  exec on managed computers
- [Regions](/docs/api-reference/regions)  -  region slugs accepted on host creation
- [Error Codes](/docs/api-reference/errors)  -  complete error code reference
