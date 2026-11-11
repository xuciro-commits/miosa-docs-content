---
slug: "security/network"
title: "网络白名单"
description: "观察沙箱会访问哪些主机；准备好后，一键启用白名单锁定。随时可撤销。"
section: "Networking & domains"
order: 5003
---

默认情况下，每个新建的 MIOSA 沙箱都配置为**仅审计模式**：不会阻止任何出站请求。仅审计模式会记录每个出站请求，因此您可以在锁定前看清沙箱访问了哪些主机。

> **注意：需手动启用，始终可撤销**
>
> 锁定绝不会自动生效。需要您明确点击“锁定”（Lock Down），并且随时可以一键恢复为仅审计模式。无论切换哪个方向，都不会丢失数据。

## 两种模式

| 模式 | 作用 | 默认？ |
|---|---|---|
| **仅审计** | 记录每个出站请求；不阻止任何内容。 | ✅ 是——每个新沙箱都从这里开始 |
| **强制执行** | 应用白名单。发往未列出主机的请求会被阻止。 | 需手动启用 |

## “先观察，后强制执行”流程

### 第 0 天——创建沙箱

无需设置。仅审计模式默认开启。沙箱的出站请求方式与 MIOSA Egress 出现之前完全相同。出站请求会被记录，其访问的主机会构成您的白名单。

### 第 0–7 天——正常使用沙箱

构建、部署、运行您的智能体——都可以。MIOSA 会收集审计数据。

### 第 7 天以后——访问“网络”（Network）标签页

“网络”（Network）标签页会显示：

```text
HOSTS CONTACTED IN THE LAST 7 DAYS                  REQUESTS

api.openai.com                                        1,247
*.anthropic.com                                          83
github.com                                               412
pypi.org                                                  29
files.pythonhosted.org                                   161
api.github.com                                          1,053
```

点击“锁定”（Lock Down）。确认弹窗会显示每个主机对应的复选框。取消勾选任何不想保留的主机，然后点击“确认”。

模式会切换为**强制执行**。智能体此后会阻止所有不在列表中的请求。

### 锁定之后——新主机进入待处理

如果沙箱尝试访问不在白名单中的主机，该请求会被阻止，同时该主机会出现在“网络”（Network）标签页的**待处理请求**（Pending Requests）区域。一键允许或阻止。

您不会陷入僵局：被拦截的请求清晰可见、可归因，只需点击一次即可处理。

## SDK——网络控制

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

## 模式匹配

| 类型 | 示例 | 匹配对象 |
|---|---|---|
| 精确主机 | `api.openai.com` | 仅匹配此主机名 |
| 通配符 | `*.openai.com` | openai.com 的任意子域名 |
| CIDR（仅 IP） | `10.0.0.0/8` | 此范围内的任意 IP；仅当请求目标是 IP 字面量（而非主机名）时适用 |

请求时的优先级：

1. 精确匹配优先于通配符
2. 通配符优先于 CIDR
3. 显式 `deny` 规则始终优先于 `allow`

## 资源级规则与租户级规则

您可以在两个层级附加白名单规则：

- **资源级**——仅适用于一个沙箱或计算机
- **租户级**——适用于租户中的每个资源，除非该资源有自己的覆盖规则

这意味着，您可以将租户默认规则设为“只允许我们的内部 API 和 npm”，再在单个沙箱上追加规则。

## 何时使用警告模式规则

您可以将规则标记为 `warn_only`：

```python
sandbox.network.allow("api.experimental.example.com", warn_only=True)
```

在强制执行模式下，`warn_only` 允许规则仍会放行请求，但会在审计日志中标记。适合在正式采用新的第三方集成前先试运行。

## 回滚

如果锁定导致功能异常：

```python
sandbox.network.observe()
```

即可恢复为仅审计模式。白名单规则会保留（方便之后重新锁定），但不再强制执行。

## 实际示例——锁定到上游服务的 Python 数据管道

一个 Python 沙箱从 REST API 获取数据，处理后将结果写入兼容 S3 的端点，并锁定为仅允许这三个主机。

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

## 在多个沙箱之间复用策略

导出一个已锁定的策略，并将其应用到新沙箱：

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

如果希望每个新沙箱都使用同一策略，请使用 `client.network.allow(...)` 和 `client.network.lockdown()` 在租户级应用规则；这些规则会成为所有资源的默认值，除非某个沙箱将其覆盖。

> **注意：模板会携带策略**
>
> 从模板创建的沙箱会继承模板的网络策略。如果模板是在强制执行模式下创建快照的，新沙箱会以锁定状态启动。创建后立即调用 `new_sandbox.network.observe()`，即可将其恢复为仅审计模式。

## 完整脚本

端到端流程：观察、检查、锁定，并查看待处理队列。

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

## 故障排除

**锁定后 `pip install` 失败——提示“Could not find a version that satisfies the requirement”。**
需要三个域名：`pypi.org`、`files.pythonhosted.org` 和 `pythonhosted.org`。请在锁定前允许全部三个域名。

**锁定后智能体停止工作，不清楚哪里被阻止。**
打开“网络”（Network）标签页 →“待处理请求”（Pending Requests），或调用 `sandbox.network.pending()`。每个被阻止的行都有一个 `rejected_by` 字段，指明触发的规则。

**锁定后 `npm install` 失败。**
请允许 `registry.npmjs.org`。部分包还会在 postinstall 脚本执行期间从 `*.github.com` 拉取内容——请查看待处理队列中的具体主机。

**我的沙箱处于强制执行模式，但审计日志中仍会出现意外的主机。**
即使在强制执行模式下，仅审计的日志条目也始终会出现（被阻止的调用仍会以 `action=reject` 记录）。按 `action=reject` 过滤即可查看被阻止的内容。

**我希望每个新沙箱都使用同一策略，而不必逐个配置。**
请使用 `client.network.allow(...)` 和 `client.network.lockdown()` 在租户级应用规则；这些规则会成为租户内所有资源的默认值，除非在沙箱级覆盖。

**调用 `observe()` 后，再次调用 `lockdown()` 时我的规则不见了。**
`observe()` 从不会删除规则。如果看起来缺失，请检查 `sandbox.network.get_policy().allow_rules`，它们会列在那里。规则消失的唯一途径是您显式调用了 `sandbox.network.clear_rules()`。

更多问题请参阅[安全故障排除指南](/docs/security/troubleshooting)。

## 常见问题

**锁定会阻止 npm install 或 pip install 吗？**
仅当您未允许 `pypi.org`、`files.pythonhosted.org`、`registry.npmjs.org` 等域名时才会。7 天的审计数据已经显示出这些主机，它们就在建议列表中。

**我锁定后，智能体突然停止工作。**
打开“网络”（Network）标签页 →“待处理请求”（Pending Requests），即可准确看到被阻止的内容。一键允许，或通过 `observe()` 恢复。

**可以在多个沙箱之间共享白名单吗？**
可以——在租户或工作区范围应用，或者从其他资源复制“策略模板”（阶段 5）。

**锁定会影响入站（预览）流量吗？**
不会。锁定仅针对出站流量。进入沙箱的预览 URL 保持与之前完全一样。
