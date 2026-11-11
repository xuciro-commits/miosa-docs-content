---
slug: "security/audit-log"
title: "审计日志"
description: "出站调用的出口审计日志：每行内容、实时跟踪、筛选和导出。当前版本尚未启用捕获功能。"
section: "Security & compliance"
order: 14002
---

启用出口审计捕获后，沙箱发出的每一次出站调用都会被自动记录，无需任何设置；你可以在每个沙箱/计算机的“审计日志”标签页中查看。当前版本尚未启用出口审计捕获，因此在出口审计流水线发布之前，该标签页可能为空。

> **警告：当前状态**
>
> 出口审计日志界面已经存在，但当前版本尚未接入捕获功能。暂时不要将其作为网络活动的唯一记录。

## 每一行的内容

| 列 | 说明 |
|---|---|
| `timestamp` | 请求发生的时间（微秒精度） |
| `host` | 上游主机名 |
| `method` | GET、POST、PUT 等 |
| `path` | URL 路径（查询字符串已隐去） |
| `action` | `allow`、`reject`、`stub`、`error`、`client_cancel` |
| `status_code` | 上游响应状态码（无响应时为 0） |
| `duration_ms` | 实际耗时 |
| `bytes_in` / `bytes_out` | 流量 |
| `request_transforms` | 触发了哪些转换（机密、允许列表、标头允许列表等） |
| `mcp` | 当调用是 MCP JSON-RPC 请求时，记录 MCP 消息追踪 |

## 实时跟踪

“审计日志”标签页通过 WebSocket 实时流式传输新事件。沙箱发起调用时，新记录会出现在顶部。

## 筛选

在界面中，使用筛选栏按以下条件限定范围：

- **时间范围**——最近 1 小时、24 小时、7 天、30 天、自定义
- **主机**——精确匹配或通配符
- **HTTP 方法**
- **操作**——仅显示被阻止、仅显示已允许、仅显示错误
- **转换**——仅显示替换了特定机密的请求，或仅显示 MCP 工具调用
- **状态码**——仅显示 4xx、仅显示 5xx 等

所有筛选条件都可以通过 URL 分享。

## 通过 SDK

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

## 常见查询

### “为什么失败了？”
筛选最近一小时的 `action=reject`。每一行都会告诉你哪条规则阻止了它（`rejected_by` 列）。

### “这个请求使用了哪个机密？”
每一行的 `request_transforms` JSON 都包含一个 `secrets` 条目，列出被替换的机密名称。界面会在详情抽屉中展示；SDK 通过 `event.request_transforms.secrets.swapped` 暴露。

### “我们的 AI 智能体上一季度做了什么？”

租户管理员可以按日期范围和资源类型筛选，并导出 CSV。

### “显示 Dana 的所有调用”
对于白标部署，请按 `external_user_id` 筛选。参见[白标凭据](/docs/security/white-label)。

## 导出

每个筛选后的视图都可以通过“导出”按钮（界面）或 SDK 导出为 CSV 或 JSON：

```python
sandbox.audit.export(since="30d", format="csv", path="audit-30d.csv")
```

## 保留期限

| 套餐 | 保留期限 |
|---|---|
| 所有套餐（默认） | 90 天 |
| 自定义保留期限（更长） | 企业版套餐可用 |

超出保留期限后，数据行会从 `egress_audit` 中删除。如果需要更长的记录，请导出数据。

> **注意：审计记录在资源删除后仍会保留**
>
> 如果删除沙箱，其审计记录仍会在保留期限内留在租户的审计表中。身份列（tenant、workspace、user、external_*）在写入时进行反规范化处理，因此查询无需连接。

## 编程式告警（阶段 5）

你将能够把保存的查询接入告警：

> “如果有任何沙箱在 5 分钟内对从未见过的主机发出超过 100 次请求，请通知我。”

此功能将在安全功能的第 5 阶段推出。

## 实际示例：多租户 SaaS 的合规仪表板

你正在构建一个 SaaS，其中每个最终用户都在自己的沙箱中运行。合规要求你在自己的仪表板中按用户展示所有出站调用，并支持导出 30 天的数据。

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

## 故障排查

**沙箱正在运行，但审计日志为空。**
审计日志只记录沙箱发出的*出站*调用。如果沙箱只是在等待输入，或进行无网络调用的进程内计算，则不会出现任何记录。请触发一次出站调用，然后重新查询。

**`audit.tail()` 在几秒后断开连接。**
跟踪连接通过 WebSocket ping 保持存活。如果你的网络空闲连接超时时间较短，连接可能会断开。请在重试循环中重新连接：

```python
while True:
    try:
        for event in sandbox.audit.tail():
            process(event)
    except ConnectionError:
        time.sleep(1)  # reconnect
```

**`request_transforms.secrets.swapped` 为空——但我已绑定了机密。**
如果沙箱的环境变量包含真实密钥（例如来自覆盖了占位符的 `.env` 文件），智能体就没有可替换的内容。运行 `sandbox.exec("echo $OPENAI_API_KEY")`——如果它打印的是真实的 `sk-proj-...` 而不是 `miosa-tok-...`，说明 `.env` 文件生效了。

**租户范围的 `client.audit.list()` 返回了过多记录。**
使用 `external_user_id`、`external_workspace_id`、`host` 或 `action` 筛选器缩小范围。每次调用的 `limit` 最大为 1000。如需更大的导出量，请使用 `audit.export()`。

**我需要早于 90 天的审计记录。**
默认保留期限为 90 天。如需更长的保留期限，请联系支持人员为你的租户启用自定义保留。如果需要永久归档，请定期导出。

更多问题请参阅[安全故障排查中心](/docs/security/troubleshooting)。

## 常见问题

**审计日志会捕获请求正文吗？**
默认不会——只捕获标头（已隐去敏感信息）和元数据。如果出于合规需要，你可以按资源配置选择捕获正文，并会显示“可能包含敏感数据”的警告。

**响应正文会被捕获吗？**
不会。审计仅记录请求侧。

**可以完全禁用审计日志吗？**
不可以——审计是平台基线功能。你可以缩短保留期限，但审计始终开启。

**实时跟踪会消耗套餐的 API 配额吗？**
不会。审计流式传输通过 WebSocket 进行，不计入 REST API 速率限制。
