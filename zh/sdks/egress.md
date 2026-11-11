---
slug: "sdks/egress"
title: "出口（安全）SDK 参考"
description: "Python、TypeScript、Go、Java 和 Elixir 中所有机密、连接账户、网络允许列表和审计日志 SDK 方法——并排对照。"
section: "CLI, SDKs & MCP"
order: 11021
---

本页并排对照全部五个 MIOSA SDK 中的每一个出口安全方法。可将其作为查询表  -  概念背景请参阅各个功能页面。

---

## 机密

### `secrets.set`  -  存储机密

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

返回 `None`。验证失败时抛出 `MiosaAPIError`。

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

### `secrets.list`  -  列出沙箱上的机密

仅返回元数据。`value` 字段永远不会返回。

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

### `secrets.rotate`  -  更新机密的值

占位符令牌保持不变。所有使用此机密的沙箱会在下一次请求时获取新值。

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

### `secrets.delete`  -  移除机密

移除绑定。沙箱的环境变量会在下一次 exec 时取消设置。

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

## 连接账户（OAuth）

### `secrets.connect`  -  发起 OAuth 流程

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

### `secrets.connect`  -  启动或重新运行 OAuth 流程

用于连接提供商，或在令牌被撤销或作用域发生变化时重新运行 OAuth 流程。

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

## 网络允许列表

### `network.allow`  -  将主机添加到允许列表

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

### `network.deny`  -  显式阻止主机

拒绝规则优先于允许规则。

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

### `network.lockdown` 和 `network.observe`

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

### `network.suggestions`  -  时间窗口内观察到的主机

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

### `network.suggestions`  -  沙箱联系过的主机

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

### `network.policies`  -  检查当前策略

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

## 审计日志

### `audit.list`  -  查询审计事件

> **注意：返回结构**
>
> 每一行都包含：`timestamp`、`host`、`method`、`path`、`action`、`status_code`、`duration_ms`、`bytes_in`、`bytes_out`、`request_transforms`、`mcp`。

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

### `audit.tail`  -  审计事件的实时流

通过 WebSocket 阻塞并逐个产出到达的事件。

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

### `audit.list`  -  查询事件并在客户端导出

没有服务器端导出端点；请用 `audit.list(...)` 查询并自行写入文件。

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

## 租户级方法

有些操作适用于整个租户，而不是单个沙箱。这些操作使用 `client` 而不是 `sandbox`。

### `client.secrets.set`  -  在租户或工作区级别设置机密作用域

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

### `client.audit.list`  -  全租户审计查询

参数与 `sandbox.audit.list` 相同，另可选用 `external_user_id` / `external_workspace_id` 过滤。

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

## 另请参阅

- [机密  -  快速入门](/docs/security/secrets)
- [连接账户](/docs/security/connect-accounts)
- [网络允许列表](/docs/security/network)
- [审计日志](/docs/security/audit-log)
- [安全故障排查](/docs/security/troubleshooting)
