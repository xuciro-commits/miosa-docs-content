---
slug: "security/audit-log"
title: "Audit Log"
description: "Egress audit log for outbound calls: rows, live tail, filters and export. Capture is not yet enabled in the current release."
section: "Security & compliance"
order: 14002
---

When egress audit capture is enabled, every outbound call from your sandbox is recorded automatically with no setup, and the Audit Log tab on each sandbox/computer is where you read it. Egress audit capture is not yet enabled in the current release, so the tab can be empty until the egress audit pipeline ships.

> **Caution: Current status**
>
> The egress audit log surface exists, but capture is not yet wired in the current release. Do not rely on it as the only record of network activity yet.

## What's in each row

| Column | Description |
|---|---|
| `timestamp` | When the request happened (microsecond precision) |
| `host` | Upstream hostname |
| `method` | GET, POST, PUT, … |
| `path` | URL path (query string redacted) |
| `action` | `allow`, `reject`, `stub`, `error`, `client_cancel` |
| `status_code` | Upstream response code (or 0 if no response) |
| `duration_ms` | Wall-clock time |
| `bytes_in` / `bytes_out` | Volume |
| `request_transforms` | Which transforms fired (secrets, allowlist, header allowlist, …) |
| `mcp` | MCP message trace, when the call was an MCP JSON-RPC request |

## Live tail

The Audit Log tab streams new events in real time over WebSocket. New rows appear at the top as your sandbox makes calls.

## Filtering

In the UI, use the filter bar to scope by:

- **Time range**  -  last hour, 24h, 7d, 30d, custom
- **Host**  -  exact or wildcard
- **HTTP method**
- **Action**  -  show only blocked, only allowed, only errors
- **Transform**  -  show only requests that swapped a particular secret, or only MCP tool calls
- **Status code**  -  show only 4xx, only 5xx, etc.

All filters are URL-shareable.

## From the SDK

**Python**

```python
# List recent events
events = sandbox.audit.list(
    since="1h",
    host="api.openai.com",
    action="allow",
    limit=100,
)

for e in events:
    print(e.timestamp, e.host, e.method, e.status_code, e.duration_ms)

# Live tail (blocks; yields events as they arrive)
for event in sandbox.audit.tail():
    print(event.host, event.action)
    if event.action == "reject":
        break

# Tenant-wide audit (admin-scoped key required)
events = client.audit.list(
    since="24h",
    external_user_id="dana@example.net",
)
```

**TypeScript**

```typescript
const events = await sandbox.audit.list({
  since: '1h',
  host: 'api.openai.com',
  action: 'allow',
  limit: 100,
});

for (const e of events) {
  console.log(e.timestamp, e.host, e.method, e.statusCode, e.durationMs);
}

for await (const event of sandbox.audit.tail()) {
  console.log(event.host, event.action);
  if (event.action === 'reject') break;
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
    fmt.Println(e.Timestamp, e.Host, e.Method, e.StatusCode, e.DurationMs)
}

// Live tail
stream, err := sandbox.Audit.Tail(ctx)
for {
    event, err := stream.Next()
    if err != nil { break }
    fmt.Println(event.Host, event.Action)
    if event.Action == "reject" { break }
}

// Tenant-wide
events, err = client.Audit.List(ctx, miosa.AuditListOpts{
    Since:          "24h",
    ExternalUserID: "dana@example.net",
})
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

// Live tail
try (AuditStream stream = sandbox.audit().tail()) {
    for (AuditEvent event : stream) {
        System.out.println(event.getHost() + " " + event.getAction());
        if ("reject".equals(event.getAction())) break;
    }
}
```

**Elixir**

```elixir
{:ok, events} = Miosa.Audit.list(sandbox.id,
  since: "1h", host: "api.openai.com", action: "allow", limit: 100)
Enum.each(events, fn e ->
  IO.puts("#{e.timestamp} #{e.host} #{e.method} #{e.status_code} #{e.duration_ms}ms")
end)

# Live tail
Miosa.Audit.tail(sandbox.id, fn event ->
  IO.puts("#{event.host} #{event.action}")
  if event.action == "reject", do: :stop, else: :continue
end)

# Tenant-wide
{:ok, events} = Miosa.Audit.list_tenant(%{since: "24h", external_user_id: "dana@example.net"})
```

## Common queries

### "Why did this fail?"
Filter by `action=reject` for the last hour. Each row tells you which rule blocked it (`rejected_by` column).

### "Which secret did this request use?"
Each row's `request_transforms` JSON includes a `secrets` entry listing the secret names swapped. UI surfaces this in the detail drawer; SDK exposes it as `event.request_transforms.secrets.swapped`.

### "What did our AI agent do last quarter?"
Tenant-admin filter by date range and resource type. Export CSV.

### "Show me all calls by Dana"
For white-label deployments, filter by `external_user_id`. See [White-label Credentials](/docs/security/white-label).

## Export

Every filtered view can be exported as CSV or JSON via the "Export" button (UI) or the SDK:

```python
sandbox.audit.export(since="30d", format="csv", path="audit-30d.csv")
```

## Retention

| Plan | Retention |
|---|---|
| All plans (default) | 90 days |
| Custom retention (longer) | available on enterprise plans |

Beyond the retention window, rows are deleted from `egress_audit`. Export your data if you need a longer record.

> **Note: Audit rows survive resource deletion**
>
> If you delete a sandbox, its audit rows remain in the tenant's audit table for the retention window. Identity columns (tenant, workspace, user, external_*) are denormalized at write time so queries don't need joins.

## Programmatic alerting (Phase 5)

You'll be able to wire saved queries into alerting:

> "Notify me if any sandbox makes more than 100 requests to a never-before-seen host in 5 minutes."

This ships in Phase 5 of the Security rollout.

## Real-world example  -  compliance dashboard for a multi-tenant SaaS

You're building a SaaS where each end user runs in their own sandbox. Compliance requires you to surface all outbound calls per user in your own dashboard, with a 30-day export.

**Python**

```python
import os, json
from miosa import Miosa
from datetime import datetime

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def get_member_report(member_id: str, days: int = 30) -> dict:
    """Return an audit summary for one member over the last N days."""
    events = client.audit.list(
        since=f"{days}d",
        external_user_id=member_id,
        limit=1000,
    )

    hosts = {}
    blocked = []
    for e in events:
        hosts[e.host] = hosts.get(e.host, 0) + 1
        if e.action == "reject":
            blocked.append({
                "timestamp": e.timestamp,
                "host": e.host,
                "method": e.method,
                "rule": e.rejected_by,
            })

    return {
        "member_id": member_id,
        "period_days": days,
        "total_requests": len(events),
        "unique_hosts": len(hosts),
        "top_hosts": sorted(hosts.items(), key=lambda x: -x[1])[:10],
        "blocked_count": len(blocked),
        "blocked_events": blocked[:20],
    }

report = get_member_report("dana@example.net", days=30)
print(json.dumps(report, indent=2))

# Export to CSV
client.audit.export(
    since="30d",
    format="csv",
    path=f"audit-dana-30d.csv",
    external_user_id="dana@example.net",
)
print("Exported audit-dana-30d.csv")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function getMemberReport(memberId: string, days = 30) {
  const events = await client.audit.list({
    since: `${days}d`,
    externalUserId: memberId,
    limit: 1000,
  });

  const hosts: Record<string, number> = {};
  const blocked = [];

  for (const e of events) {
    hosts[e.host] = (hosts[e.host] ?? 0) + 1;
    if (e.action === 'reject') {
      blocked.push({ timestamp: e.timestamp, host: e.host, rule: e.rejectedBy });
    }
  }

  return {
    memberId,
    periodDays: days,
    totalRequests: events.length,
    uniqueHosts: Object.keys(hosts).length,
    topHosts: Object.entries(hosts).sort(([, a], [, b]) => b - a).slice(0, 10),
    blockedCount: blocked.length,
    blockedEvents: blocked.slice(0, 20),
  };
}

const report = await getMemberReport('dana@example.net', 30);
console.log(JSON.stringify(report, null, 2));

await client.audit.export({
  since: '30d',
  format: 'csv',
  path: 'audit-dana-30d.csv',
  externalUserId: 'dana@example.net',
});
```

## Troubleshooting

**Audit log is empty even though the sandbox is running.**
The audit log only records *outbound* calls from the sandbox. If the sandbox is just waiting for input or doing in-process computation with no network calls, no rows appear. Trigger an outbound call and re-query.

**`audit.tail()` disconnects after a few seconds.**
Tail connections are kept alive with WebSocket pings. If your network has a short idle-connection timeout, the connection can drop. Reconnect in your retry loop:

```python
while True:
    try:
        for event in sandbox.audit.tail():
            process(event)
    except ConnectionError:
        time.sleep(1)  # reconnect
```

**`request_transforms.secrets.swapped` is empty  -  but I have secrets bound.**
If the sandbox's env var contains the real key (e.g., from a `.env` file that overrode the placeholder), the proxy has nothing to swap. Run `sandbox.exec("echo $OPENAI_API_KEY")`  -  if it prints the real `sk-proj-...` instead of `miosa-tok-...`, the `.env` file is winning.

**Tenant-wide `client.audit.list()` returns too many rows.**
Narrow with `external_user_id`, `external_workspace_id`, `host`, or `action` filters. Maximum `limit` per call is 1000. For larger exports, use `audit.export()`.

**I need audit rows older than 90 days.**
Default retention is 90 days. For longer retention, contact support to enable custom retention on your tenant. Export regularly if you need a permanent archive.

For more issues, see the [Security troubleshooting hub](/docs/security/troubleshooting).

## Frequently asked

**Does the audit log capture request bodies?**
Not by default  -  only headers (redacted) and metadata. You can opt in to body capture per-resource if you need it for compliance, with a "may contain sensitive data" warning.

**Are response bodies captured?**
No. Audit is request-side only.

**Can I disable the audit log entirely?**
No  -  audit is the platform baseline. You can shorten retention but not disable it.

**Will live-tail consume my plan's API quota?**
No. Audit streaming is over a WebSocket and doesn't count against REST API rate limits.
