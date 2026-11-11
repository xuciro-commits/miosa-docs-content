---
slug: "sdks/egress"
title: "Egress SDK reference"
description: "Every Secrets, Connect Accounts, Network Allowlist, and Audit Log SDK method across Python, TypeScript, Go, Java, and Elixir - side by side."
section: "CLI, SDKs & MCP"
order: 11021
---

Side-by-side reference for every egress security method across all five MIOSA SDKs. Use this as a lookup table  -  for conceptual background, see the individual feature pages.

---

## Secrets

### `secrets.set`  -  Store a secret

**Python**

```python
sandbox.secrets.set(
    name: str,
    value: str,
    expose_as_env: str,
    scope: str = "sandbox",            # "sandbox" | "user" | "external_user" | "external_workspace" | "tenant"
    external_user_id: str | None = None,
    external_workspace_id: str | None = None,
)
```

Returns `None`. Raises `MiosaAPIError` on validation failure.

```python
sandbox.secrets.set(
    name="openai_key",
    value=os.environ["MY_OPENAI_KEY"],
    expose_as_env="OPENAI_API_KEY",
)
```

**TypeScript**

```typescript
await sandbox.secrets.set({
  name: string,
  value: string,
  exposeAsEnv: string,
  scope?: 'sandbox' | 'user' | 'external_user' | 'external_workspace' | 'tenant',
  externalUserId?: string,
  externalWorkspaceId?: string,
}): Promise<void>
```

```typescript
await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
```

**Go**

```go
err := sandbox.Secrets.Set(ctx, miosa.SecretSet{
    Name:                string,
    Value:               string,
    ExposeAsEnv:         string,
    Scope:               string,   // "sandbox" | "user" | "external_user" | "external_workspace" | "tenant"
    ExternalUserID:      string,   // optional
    ExternalWorkspaceID: string,   // optional
})
```

```go
err := sandbox.Secrets.Set(ctx, miosa.SecretSet{
    Name:        "openai_key",
    Value:       os.Getenv("MY_OPENAI_KEY"),
    ExposeAsEnv: "OPENAI_API_KEY",
})
```

**Java**

```java
sandbox.secrets().set(SecretSet.builder()
    .name(String name)
    .value(String value)
    .exposeAsEnv(String exposeAsEnv)
    .scope(String scope)              // optional
    .externalUserId(String id)        // optional
    .externalWorkspaceId(String id)   // optional
    .build());
```

```java
sandbox.secrets().set(SecretSet.builder()
    .name("openai_key")
    .value(System.getenv("MY_OPENAI_KEY"))
    .exposeAsEnv("OPENAI_API_KEY")
    .build());
```

**Elixir**

```elixir
Miosa.Secrets.set(sandbox_id, %{
  name: String.t(),
  value: String.t(),
  expose_as_env: String.t(),
  scope: String.t() | nil,               # optional
  external_user_id: String.t() | nil,    # optional
  external_workspace_id: String.t() | nil
})
# Returns {:ok, nil} | {:error, reason}
```

```elixir
{:ok, _} = Miosa.Secrets.set(sandbox.id, %{
  name: "openai_key",
  value: System.get_env("MY_OPENAI_KEY"),
  expose_as_env: "OPENAI_API_KEY"
})
```

---

### `secrets.list`  -  List secrets on a sandbox

Returns metadata only. The `value` field is never returned.

**Python**

```python
secrets = sandbox.secrets.list()
# Returns List[SecretMeta]
# SecretMeta: name, type, expose_as_env, scope, preview (last 4 chars), last_used_at

for s in secrets:
    print(s.name, s.expose_as_env, s.last_used_at)
```

**TypeScript**

```typescript
const secrets = await sandbox.secrets.list();
// Returns SecretMeta[]
// SecretMeta: { name, type, exposeAsEnv, scope, preview, lastUsedAt }

for (const s of secrets) {
  console.log(s.name, s.exposeAsEnv, s.lastUsedAt);
}
```

**Go**

```go
secrets, err := sandbox.Secrets.List(ctx)
// Returns []miosa.SecretMeta

for _, s := range secrets {
    fmt.Println(s.Name, s.ExposeAsEnv, s.LastUsedAt)
}
```

**Java**

```java
List<SecretMeta> secrets = sandbox.secrets().list();
for (SecretMeta s : secrets) {
    System.out.println(s.getName() + " " + s.getExposeAsEnv());
}
```

**Elixir**

```elixir
{:ok, secrets} = Miosa.Secrets.list(sandbox.id)
Enum.each(secrets, fn s ->
  IO.puts("#{s.name} → #{s.expose_as_env}")
end)
```

---

### `secrets.rotate`  -  Update a secret's value

The placeholder token is unchanged. All sandboxes using this secret pick up the new value on their next request.

**Python**

```python
sandbox.secrets.rotate("openai_key", new_value="sk-proj-newvalue...")
```

**TypeScript**

```typescript
await sandbox.secrets.rotate('openai_key', { new_value: 'sk-proj-newvalue...' });
```

**Go**

```go
err := sandbox.Secrets.Rotate(ctx, "openai_key", "sk-proj-newvalue...")
```

**Java**

```java
sandbox.secrets().rotate("openai_key", "sk-proj-newvalue...");
```

**Elixir**

```elixir
{:ok, _} = Miosa.Secrets.rotate(sandbox.id, "openai_key", "sk-proj-newvalue...")
```

---

### `secrets.delete`  -  Remove a secret

Removes the binding. The sandbox's env var is unset on the next exec.

**Python**

```python
sandbox.secrets.delete("openai_key")
```

**TypeScript**

```typescript
await sandbox.secrets.delete('openai_key');
```

**Go**

```go
err := sandbox.Secrets.Delete(ctx, "openai_key")
```

**Java**

```java
sandbox.secrets().delete("openai_key");
```

**Elixir**

```elixir
{:ok, _} = Miosa.Secrets.delete(sandbox.id, "openai_key")
```

---

## Connect Accounts (OAuth)

### `secrets.connect`  -  Initiate an OAuth flow

**Python**

```python
flow = sandbox.secrets.connect(
    provider: str,           # "github" | "google" | "slack" | …
    expose_as_env: str,
    scopes: list[str] | None = None,
)
# Returns OAuthFlow
# OAuthFlow: authorize_url (str), wait_for_completion(timeout: int)

print(flow.authorize_url)
flow.wait_for_completion(timeout=300)
```

**TypeScript**

```typescript
const flow = await sandbox.secrets.connect({
  provider: string,        // 'github' | 'google' | 'slack' | …
  exposeAsEnv: string,
  scopes?: string[],
});
// Returns OAuthFlow
// OAuthFlow: { authorizeUrl: string, waitForCompletion(opts): Promise<void> }

console.log(flow.authorizeUrl);
await flow.waitForCompletion({ timeoutSec: 300 });
```

**Go**

```go
flow, err := sandbox.Secrets.Connect(ctx, miosa.OAuthConnect{
    Provider:    string,   // "github" | "google" | "slack"
    ExposeAsEnv: string,
    Scopes:      []string,
})
// flow.AuthorizeURL string
// flow.WaitForCompletion(ctx, timeoutSec int) error

fmt.Println(flow.AuthorizeURL)
err = flow.WaitForCompletion(ctx, 300)
```

**Java**

```java
OAuthFlow flow = sandbox.secrets().connect(OAuthConnect.builder()
    .provider("github")
    .exposeAsEnv("GITHUB_TOKEN")
    .scopes(List.of("repo", "read:user"))
    .build());

System.out.println(flow.getAuthorizeUrl());
flow.waitForCompletion(Duration.ofSeconds(300));
```

**Elixir**

```elixir
{:ok, flow} = Miosa.Secrets.connect(sandbox.id, %{
  provider: "github",
  expose_as_env: "GITHUB_TOKEN",
  scopes: ["repo", "read:user"]
})

IO.puts(flow.authorize_url)
{:ok, _} = Miosa.Secrets.wait_for_oauth(flow.id, timeout: 300)
```

---

### `secrets.connect`  -  Start or re-run an OAuth flow

Use to connect a provider, or to re-run an OAuth flow when a token was revoked or scopes changed.

**Python**

```python
flow = sandbox.secrets.connect(provider="github")
print(flow.authorize_url)
flow.wait_for_completion(timeout=300)
```

**TypeScript**

```typescript
const flow = await sandbox.secrets.connect({ provider: 'github' });
await flow.waitForCompletion({ timeoutSec: 300 });
```

**Go**

```go
flow, err := sandbox.Secrets.Connect(ctx, "github")
err = flow.WaitForCompletion(ctx, 300)
```

**Java**

```java
OAuthFlow flow = sandbox.secrets().connect("github");
flow.waitForCompletion(Duration.ofSeconds(300));
```

**Elixir**

```elixir
{:ok, flow} = Miosa.Secrets.connect(client, "github")
```

---

## Network Allowlist

### `network.allow`  -  Add a host to the allowlist

**Python**

```python
sandbox.network.allow(
    host: str,                  # e.g. "api.openai.com" or "*.openai.com"
    methods: list[str] | None,  # ["GET", "POST"]  -  None means all
    warn_only: bool = False,    # allow but flag in audit log
)
```

```python
sandbox.network.allow("api.openai.com")
sandbox.network.allow("*.anthropic.com", methods=["POST"])
sandbox.network.allow("api.experimental.io", warn_only=True)
```

**TypeScript**

```typescript
await sandbox.network.allow(
  host: string,
  opts?: { methods?: string[]; warnOnly?: boolean }
)
```

```typescript
await sandbox.network.allow('api.openai.com');
await sandbox.network.allow('*.anthropic.com', { methods: ['POST'] });
await sandbox.network.allow('api.experimental.io', { warnOnly: true });
```

**Go**

```go
err := sandbox.Network.Allow(ctx, "api.openai.com", miosa.AllowOpts{})
err = sandbox.Network.Allow(ctx, "*.anthropic.com", miosa.AllowOpts{
    Methods:  []string{"POST"},
})
err = sandbox.Network.Allow(ctx, "api.experimental.io", miosa.AllowOpts{
    WarnOnly: true,
})
```

**Java**

```java
sandbox.network().allow("api.openai.com");
sandbox.network().allow("*.anthropic.com", AllowOpts.builder().methods(List.of("POST")).build());
```

**Elixir**

```elixir
{:ok, _} = Miosa.Network.allow(sandbox.id, "api.openai.com")
{:ok, _} = Miosa.Network.allow(sandbox.id, "*.anthropic.com", methods: ["POST"])
```

---

### `network.deny`  -  Explicitly block a host

Deny rules win over allow rules.

**Python**

```python
sandbox.network.deny("169.254.169.254")
```

**TypeScript**

```typescript
await sandbox.network.deny('169.254.169.254');
```

**Go**

```go
err := sandbox.Network.Deny(ctx, "169.254.169.254")
```

**Java**

```java
sandbox.network().deny("169.254.169.254");
```

**Elixir**

```elixir
{:ok, _} = Miosa.Network.deny(sandbox.id, "169.254.169.254")
```

---

### `network.lockdown` and `network.observe`

**Python**

```python
sandbox.network.lockdown()   # enforce mode  -  apply allowlist
sandbox.network.observe()    # audit-only  -  log everything, block nothing
```

**TypeScript**

```typescript
await sandbox.network.lockdown();
await sandbox.network.observe();
```

**Go**

```go
err := sandbox.Network.Lockdown(ctx)
err = sandbox.Network.Observe(ctx)
```

**Java**

```java
sandbox.network().lockdown();
sandbox.network().observe();
```

**Elixir**

```elixir
{:ok, _} = Miosa.Network.lockdown(sandbox.id)
{:ok, _} = Miosa.Network.observe(sandbox.id)
```

---

### `network.suggestions`  -  Hosts observed in a time window

**Python**

```python
hosts = sandbox.network.suggestions(since: str = "7d")
# Returns List[HostSuggestion]
# HostSuggestion: host, request_count, last_seen, methods

for h in hosts:
    print(h.host, h.request_count)
```

**TypeScript**

```typescript
const hosts = await sandbox.network.suggestions({ since?: string });
// Returns HostSuggestion[]
// HostSuggestion: { host, requestCount, lastSeen, methods }
```

**Go**

```go
hosts, err := sandbox.Network.Suggestions(ctx, miosa.SuggestionsOpts{Since: "7d"})
// returns []miosa.HostSuggestion
```

**Java**

```java
List<HostSuggestion> hosts = sandbox.network().suggestions(SuggestionsOpts.builder()
    .since("7d").build());
```

**Elixir**

```elixir
{:ok, hosts} = Miosa.Network.suggestions(sandbox.id, since: "7d")
```

---

### `network.suggestions`  -  Hosts the sandbox contacted

**Python**

```python
suggestions = sandbox.network.suggestions()
# Returns the hosts the sandbox has contacted.
for host in suggestions:
    print(host["host"], host.get("request_count"))
```

**TypeScript**

```typescript
const suggestions = await sandbox.network.suggestions();
// Returns the hosts the sandbox has contacted.
for (const host of suggestions) console.log(host.host, host.request_count);
```

**Go**

```go
suggestions, err := sandbox.Network.Suggestions(ctx)
```

**Java**

```java
var suggestions = sandbox.network().suggestions();
```

**Elixir**

```elixir
{:ok, suggestions} = Miosa.Network.suggestions(client)
```

---

### `network.policies`  -  Inspect current policies

**Python**

```python
policies = sandbox.network.policies()
for policy in policies:
    print(policy["id"], policy["mode"])  # "observe" | "enforce"
```

**TypeScript**

```typescript
const policies = await sandbox.network.policies();
for (const policy of policies) console.log(policy.id, policy.mode);
```

**Go**

```go
policies, err := sandbox.Network.Policies(ctx)
```

**Java**

```java
var policies = sandbox.network().policies();
```

**Elixir**

```elixir
{:ok, policies} = Miosa.Network.policies(client)
```

---

## Audit Log

### `audit.list`  -  Query audit events

> **Note: Return shape**
>
> Every row has: `timestamp`, `host`, `method`, `path`, `action`, `status_code`, `duration_ms`, `bytes_in`, `bytes_out`, `request_transforms`, `mcp`.

**Python**

```python
events = sandbox.audit.list(
    since: str = "1h",                # "15m" | "1h" | "24h" | "7d" | "30d" | ISO8601
    host: str | None = None,          # exact or glob, e.g. "*.openai.com"
    method: str | None = None,        # "GET" | "POST" | …
    action: str | None = None,        # "allow" | "reject" | "stub" | "error"
    limit: int = 100,                 # max 1000
    external_user_id: str | None = None,
    external_workspace_id: str | None = None,
)
# Returns List[AuditEvent]
```

```python
events = sandbox.audit.list(since="1h", host="api.openai.com", action="allow")
for e in events:
    print(e.timestamp, e.host, e.status_code, e.duration_ms, "ms")
```

**TypeScript**

```typescript
const events = await sandbox.audit.list({
  since?: string,
  host?: string,
  method?: string,
  action?: 'allow' | 'reject' | 'stub' | 'error',
  limit?: number,
  externalUserId?: string,
  externalWorkspaceId?: string,
}): Promise<AuditEvent[]>
```

```typescript
const events = await sandbox.audit.list({ since: '1h', action: 'reject' });
for (const e of events) {
  console.log(e.timestamp, e.host, e.statusCode, e.durationMs, 'ms');
}
```

**Go**

```go
events, err := sandbox.Audit.List(ctx, miosa.AuditListOpts{
    Since:  "1h",
    Host:   "api.openai.com",
    Action: "allow",
    Limit:  100,
})
for _, e := range events {
    fmt.Println(e.Timestamp, e.Host, e.StatusCode, e.DurationMs, "ms")
}
```

**Java**

```java
List<AuditEvent> events = sandbox.audit().list(AuditListOpts.builder()
    .since("1h")
    .host("api.openai.com")
    .action("allow")
    .limit(100)
    .build());

for (AuditEvent e : events) {
    System.out.println(e.getTimestamp() + " " + e.getHost() + " " + e.getStatusCode());
}
```

**Elixir**

```elixir
{:ok, events} = Miosa.Audit.list(sandbox.id, since: "1h", host: "api.openai.com", action: "allow")
Enum.each(events, fn e ->
  IO.puts("#{e.timestamp} #{e.host} #{e.status_code} #{e.duration_ms}ms")
end)
```

---

### `audit.tail`  -  Live stream of audit events

Blocks and yields events as they arrive over a WebSocket.

**Python**

```python
# Blocks forever; break out when needed
for event in sandbox.audit.tail():
    print(event.host, event.action, event.status_code)
    if event.action == "reject":
        break
```

**TypeScript**

```typescript
// The TypeScript SDK has no live tail; poll audit.list().
const events = await sandbox.audit.list({ since: '1h' });
for (const event of events) {
  console.log(event.host, event.action, event.status_code);
  if (event.action === 'reject') break;
}
```

**Go**

```go
stream, err := sandbox.Audit.Tail(ctx)
for {
    event, err := stream.Next()
    if err != nil { break }
    fmt.Println(event.Host, event.Action, event.StatusCode)
    if event.Action == "reject" { break }
}
```

**Java**

```java
try (AuditStream stream = sandbox.audit().tail()) {
    for (AuditEvent event : stream) {
        System.out.println(event.getHost() + " " + event.getAction());
        if ("reject".equals(event.getAction())) break;
    }
}
```

**Elixir**

```elixir
Miosa.Audit.tail(sandbox.id, fn event ->
  IO.puts("#{event.host} #{event.action} #{event.status_code}")
  if event.action == "reject", do: :stop, else: :continue
end)
```

---

### `audit.list`  -  Query events and export client-side

There is no server-side export endpoint; query with `audit.list(...)` and write the file yourself.

**Python**

```python
import csv

events = sandbox.audit.list(since="30d")
with open("audit-30d.csv", "w", newline="") as f:
    w = csv.writer(f)
    w.writerow(["host", "method", "path", "action", "status_code"])
    for e in events:
        w.writerow([e["host"], e["method"], e["path"], e["action"], e["status_code"]])
```

**TypeScript**

```typescript
const events = await sandbox.audit.list({ since: '30d' });
// Write `events` to CSV or JSON in your own process.
```

---

## Tenant-level methods

Some operations apply across your entire tenant rather than a single sandbox. These use `client` rather than `sandbox`.

### `client.secrets.set`  -  Scope a secret at tenant or workspace level

**Python**

```python
# Tenant-wide credential  -  all sandboxes in the tenant resolve it
client.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    scope="tenant",
    expose_as_env="OPENAI_API_KEY",
)

# External-user credential
client.secrets.set(
    name="user_openai_key",
    value="sk-proj-...",
    scope="external_user",
    external_user_id="dana@example.net",
    expose_as_env="OPENAI_API_KEY",
)
```

**TypeScript**

```typescript
await client.secrets.set({
  name: 'openai_key',
  value: 'sk-proj-...',
  scope: 'tenant',
  exposeAsEnv: 'OPENAI_API_KEY',
});
```

**Go**

```go
err := client.Secrets.Set(ctx, miosa.SecretSet{
    Name:        "openai_key",
    Value:       "sk-proj-...",
    Scope:       "tenant",
    ExposeAsEnv: "OPENAI_API_KEY",
})
```

**Java**

```java
client.secrets().set(SecretSet.builder()
    .name("openai_key")
    .value("sk-proj-...")
    .scope("tenant")
    .exposeAsEnv("OPENAI_API_KEY")
    .build());
```

**Elixir**

```elixir
{:ok, _} = Miosa.Secrets.set(client, %{
  name: "openai_key", value: "sk-proj-...",
  scope: "tenant",
  expose_as_env: "OPENAI_API_KEY"
})
```

### `client.audit.list`  -  Tenant-wide audit query

Same parameters as `sandbox.audit.list` plus optional `external_user_id` / `external_workspace_id` filter.

**Python**

```python
# All activity by one end-user across all sandboxes
events = client.audit.list(
    since="7d",
    external_user_id="dana@example.net",
)

# All activity for one workspace
events = client.audit.list(
    since="30d",
    external_workspace_id="acme",
)
```

**TypeScript**

```typescript
const events = await client.audit.list({
  since: '7d',
  externalUserId: 'dana@example.net',
});
```

**Go**

```go
events, err := client.Audit.List(ctx, miosa.AuditListOpts{
    Since:          "7d",
    ExternalUserID: "dana@example.net",
})
```

**Java**

```java
List<AuditEvent> events = client.audit().list(AuditListOpts.builder()
    .since("7d")
    .externalUserId("dana@example.net")
    .build());
```

**Elixir**

```elixir
{:ok, events} = Miosa.Audit.list(client, %{since: "7d", external_user_id: "dana@example.net"})
```

---

## See also

- [Secrets  -  Quickstart](/docs/security/secrets)
- [Connect Accounts](/docs/security/connect-accounts)
- [Network Allowlist](/docs/security/network)
- [Audit Log](/docs/security/audit-log)
- [Security troubleshooting](/docs/security/troubleshooting)
