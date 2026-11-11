---
slug: "develop/sandboxes/internet-access"
title: "互联网接入和出口"
description: "沙箱默认是否可以访问互联网、两层出口控制、如何将沙箱锁定到白名单以及如何调试被阻止的调用。"
section: "Networking & domains"
order: 5002
---

## 默认网络行为

- **入站。** 无法通过互联网访问沙箱。只有您公开为 [预览](/docs/develop/previews) 的端口接受流量，并且仅通过 MIOSA 智能体。
- **出站。** 新的沙箱可以访问互联网。它以**仅审核**模式启动：不会阻止出站呼叫。当前版本中尚未启用出口审核捕获，因此尚未记录呼叫。计算机的网络策略默认为 `unrestricted`。

因此，默认情况下，您的代码可以安装包、调用 API 和克隆存储库。在您选择强制执行白名单之前，不会阻止任何内容。

### 沙箱无法达到的目标

沙箱无法到达平台自己的内部网络。
唯一的例外是托管数据库智能体范围，这是托管 Postgres、MySQL 或 Redis 连接在沙箱内使用的智能体范围。
这是网格级拒绝，在出口允许列表之前进行检查，因此将内部主机添加到允许列表不会打开它。
它在两个运行时上均强制执行。

`169.254.169.254` 不是 MIOSA 上的出口目的地。
它是 Firecracker 的元数据服务 (MMDS)，是主机用来向沙箱传递其身份和配置的来宾端通道，并且在计算机内部而不是在主机网络上进行应答。
让它被拒绝，如下面的示例所示。

## 两层控制

MIOSA 为您提供两种限制出站流量的方法。他们作曲。

|层|范围 |它决定什么 |
|---|---|---|
|出口政策 + 允许名单 |计算机和沙箱|在 `audit_only` 或 `enforce` 模式下，资源可能到达的主机 |
|网络政策|计算机 | microVM 级模式：`unrestricted`、`allowlist`、`denylist` 或 `isolated` |

计算机或沙箱将其出口策略解析为：资源覆盖，然后是租户默认。租户默认以 `audit_only` 模式启动。

## 观察，然后执行

沙箱的预期路径是观察真实流量，然后将其锁定到实际使用的流量。这与 [网络白名单参考](/docs/security/network) 上完整记录的流程相同：

1. **观察。** 让沙箱正常运行。每个出站主机都会被记录。
2. **查看。** 拉取其联系的主机，按频率排名。
3. **强制。** 将这些主机应用为白名单，并将策略切换为 `enforce`。然后，未列出的主机将被阻止。
4. **操作。** 被阻止的主机通过一键允许或拒绝进入待处理队列。随时恢复为仅限审核，而不会丢失许可名单。

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

**打字稿**

```typescript
const hosts = await sandbox.network.suggestions({ since: '7d' });
await sandbox.network.allow('api.openai.com');
await sandbox.network.allow('*.anthropic.com', { methods: ['POST'] });
await sandbox.network.deny('169.254.169.254');  // Firecracker metadata service (MMDS)
await sandbox.network.lockdown();
await sandbox.network.observe();
const policy = await sandbox.network.getPolicy();
```

### 白名单规则

|亲切 |示例|比赛|
|---|---|---|
|确切的主机| `api.openai.com` |仅此主机名 |
|通配符| `*.openai.com` | `openai.com` | 的任何子域
| CIDR | `10.0.0.0/8` |当目标是 IP 文字时，此范围内的任何 IP |

规则按优先级顺序匹配：精确匹配通配符，通配​​符匹配 CIDR，显式 `deny` 始终匹配 `allow`。规则的范围可以限定为方法和路径，并标记为 `warn_only` 以允许请求通过，同时在审核日志中对其进行标记。

规则附加在两个级别：每个资源（一台沙箱或计算机）和租户范围（每个资源，除非被覆盖）。

## 在 microVM 级别限制计算机

计算机还接受在几秒钟内生效的网络策略，无需重新启动。

**打字稿**

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

|模式|描述 |
|---|---|
| `unrestricted` |默认。允许所有出站流量 |
| `allowlist` |仅允许列出的主机和端口 |
| `denylist` |允许除列出的规则之外的所有流量 |
| `isolated` |所有出站被阻止（DNS 和 MIOSA 内部除外）|

`PUT` 取代整个策略；每次调用时发送每条规则。 `DELETE` 将其重置为 `unrestricted`。

## 外出时的凭证

出口也是 MIOSA 将占位符交换为真实凭证的地方。您的工作负载在其环境中看到不透明的 `miosa-tok-...` 值；智能体替换出站请求上的真实秘密，因此该值永远不会存在于沙箱内。请参阅 [Secrets](/docs/security/secrets) 和 [Egress API](/docs/api-reference/egress)。

## 调试阻塞的调用

**Python**

```python
# What is being blocked right now
for p in sandbox.network.pending():
    print(p.host, p.request_count, "blocked")

# Or filter the audit log for rejects
for e in sandbox.audit.list(since="1h", action="reject", limit=50):
    print(e.host, e.rejected_by)
```

常见的许可名单差距：- **`pip install`** 需要 `pypi.org`、`files.pythonhosted.org` 和 `pythonhosted.org`。
- **`npm install`**需要`registry.npmjs.org`；一些安装后脚本也达到`*.github.com`。
- **CDN 和动态子域** 通常不包含在确切的规则中。使用通配符，例如 `*.openai.com`。

每个被阻止的行都带有一个 `rejected_by` 字段，用于命名触发的规则。有关修复的完整列表，请参阅[安全故障排除](/docs/security/troubleshooting)。

> **注意：锁定不会影响入站流量**
>
> 强制执行白名单仅限制出站呼叫。沙盒中的预览 URL 继续像以前一样正常工作。

## 另请参阅

- [网络白名单参考](/docs/security/network)：先观察后执行流程、SDK 方法和待处理队列。
- [Egress API](/docs/api-reference/egress)：策略、白名单规则、审核日志和保管库。
- [网络策略 API](/docs/api-reference/network-policy)：计算机的 microVM 级策略。
