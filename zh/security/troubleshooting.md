---
slug: "security/troubleshooting"
title: "安全——故障排查"
description: "MIOSA 机密、连接账户、网络允许列表、审计日志和白标凭据中最常见的 15 个问题，以及根因和修复方法。"
section: "Security & compliance"
order: 14005
---

所有 MIOSA Security 功能中 15 个最常见问题的快速参考。每条均按“症状 → 根因 → 修复”组织。

---

## 机密

### 1. 机密值没有被替换：出站请求使用了原始占位符

**症状**：您的沙箱在 `$OPENAI_API_KEY` 中看到 `miosa-tok-...`，而该占位符到达上游 API，返回 401。

**根因**：智能体通过扫描出站请求中的已知令牌前缀来替换占位符。如果请求使用非标准响应头名称，或者根本没有包含 `Authorization` 响应头，替换就会被跳过。

**修复**：
- 确认您使用的 SDK 或库将令牌放在 `Authorization: Bearer ...` 响应头中。
- 对于查询字符串认证（例如 `?api_key=...`），确保令牌原样出现在查询字符串中——智能体也会扫描查询参数。
- 检查审计行的 `request_transforms.secrets.swapped` 字段。如果为空，说明智能体没有匹配到占位符。
- 验证环境变量名称的大小写完全一致。`OPENAI_API_KEY` ≠ `openai_api_key`。智能体在查找 `expose_as_env` 绑定时区分大小写。

---

### 2. `secrets.list()` 显示有机密，但沙箱内的 `$ENV_VAR` 为空

**症状**：机密在控制台和 SDK 中可见，但在沙箱内执行 `echo $OPENAI_API_KEY` 没有任何输出。

**根因**：机密创建时使用了 `scope="tenant"` 或 `scope="workspace"`，但沙箱不是在同一个工作区下创建的。或者 `expose_as_env` 的值有拼写错误。

**修复**：
```python
# Check the binding on the sandbox directly
secrets = sandbox.secrets.list()
for s in secrets:
    print(s.name, s.expose_as_env, s.scope)

# If missing, bind it explicitly to this sandbox
sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

---

### 3. 轮换密钥后没有生效——沙箱仍在使用旧密钥

**症状**：您调用了 `sandbox.secrets.rotate(name, new_value=...)`，但发往 API 的出站调用仍然成功，看起来像是旧密钥还在工作。

**根因**：这几乎总是正确行为。轮换是即时的——智能体会在下一次携带占位符的请求中查找新值。如果轮换后上游调用成功，说明新密钥已经在使用。旧密钥可能仍在提供商侧有效（MIOSA 不会让密钥失效——只有提供商侧才会）。

**修复**：通过检查审计日志验证轮换是否生效：
```python
events = sandbox.audit.list(since="5m", host="api.openai.com", limit=5)
for e in events:
    print(e.timestamp, e.status_code, e.request_transforms)
```
如果轮换后的 `status_code` 为 200，说明新密钥正在使用。

要强制旧密钥停止工作：请在提供商侧撤销它（例如在 OpenAI 控制台中删除），然后在 MIOSA 上轮换。

---

## 连接账户（OAuth）

### 4. GitHub OAuth 刷新失败——沙箱访问 `api.github.com` 收到 401

**症状**：沙箱原本运行正常，随后突然开始收到 GitHub 的 401。审计日志显示 `action=allow`，但 `status_code=401`。

**根因**：GitHub 访问令牌已过期，刷新尝试失败。常见原因包括：
- 用户在 GitHub 设置中撤销了该 GitHub OAuth 应用的访问权限
- 刷新令牌本身已过期（GitHub 个人 OAuth 令牌不会过期，但 GitHub Apps 令牌在 8 小时未刷新后会过期）
- 提供商侧的权限范围已更改

**修复**：
1. 在 MIOSA 控制台 → Secrets 标签页中，GitHub 条目会显示“需要重新连接”。
2. 调用 `flow.reauthorize()` 重新执行 OAuth 流程：
```python
flow = sandbox.secrets.reauthorize(provider="github")
print("Authorize at:", flow.authorize_url)
flow.wait_for_completion(timeout=300)
```

---

### 5. 自定义 OAuth 提供商返回“invalid_state”

**症状**：来自自定义 OAuth2 提供商的 OAuth 回调包含 `error=invalid_state`。

**根因**：OAuth 流程中的 `state` 参数有效期为 10 分钟。如果用户授权耗时超过 10 分钟，或者回调被调用两次（例如用户点击返回按钮后重试），state 就会过期或被消耗。

**修复**：重新发起流程。State 是一次性使用的。
```python
# Don't reuse the old flow object  -  create a new one
flow = sandbox.secrets.connect(
    provider="my_custom_provider",
    expose_as_env="MY_API_TOKEN",
)
print("New authorize URL:", flow.authorize_url)
```
如果您正在构建 UI，请向用户显示“重试”按钮，由它调用您的后端来发起新的 `connect()`。

---

### 6. 提供商 OAuth 应用显示“Authorize MIOSA”，而不是我的品牌

**症状**：您的最终用户在 GitHub（或其他提供商）授权同意屏幕上看到 MIOSA 品牌。

**根因**：您尚未为租户注册自定义 OAuth 应用覆盖。

**修复**：参见[白标 OAuth 设置](/docs/security/white-label#white-label-oauth-your-branding-not-miosas)。在提供商处注册您自己的 OAuth 应用，然后在 Settings → OAuth Providers 中添加覆盖。

---

## 网络允许列表

### 7. 锁定网络后沙箱无法访问 `pypi.org`

**症状**：锁定网络后，`pip install <package>` 因连接错误失败。

**根因**：`pip` 会访问三个域名，三者都必须加入允许列表：

| 域名 | 作用 |
|---|---|
| `pypi.org` | 软件包索引查询 |
| `files.pythonhosted.org` | 软件包下载 CDN |
| `pythonhosted.org` | 旧版软件包下载（部分软件包仍在使用） |

**修复**：
```python
sandbox.network.allow("pypi.org")
sandbox.network.allow("files.pythonhosted.org")
sandbox.network.allow("pythonhosted.org")
# If you also need conda:
sandbox.network.allow("conda.anaconda.org")
sandbox.network.allow("repo.anaconda.com")
```

---

### 8. 锁定网络后 `npm install` 失败

**症状**：切换到强制模式后，`npm install` 或 `pnpm install` 失败。

**根因**：npm registry 及其 CDN 不在允许列表中。

**修复**：
```python
sandbox.network.allow("registry.npmjs.org")
sandbox.network.allow("*.npmjs.com")
```
如果您使用 pnpm 自己的镜像：
```python
sandbox.network.allow("registry.npmjs.org")
```

---

### 9. 智能体不再调用已在允许列表中的主机

**症状**：您可以在允许列表中看到该主机，但智能体仍然被阻止。

**根因**：智能体正在解析规则未覆盖的子域名或别名。例如，列表中包含 `api.openai.com`，但智能体同时还在访问 `oai.endpoint.amazonaws.com`（OpenAI 的后端 CDN）。

**修复**：
1. 检查待处理请求队列：
```python
pending = sandbox.network.pending()
for p in pending:
    print(p.host, p.request_count)
```
2. 将缺失的主机加入允许列表。
3. 对于 CDN 和动态子域名，使用通配符：`*.openai.com`。

---

### 10. 我锁定了一个沙箱，但同一模板创建的所有新沙箱也被锁定

**症状**：从模板创建的沙箱意外继承了强制模式。

**根因**：您制作快照的模板已经处于强制模式。策略状态是模板快照的一部分。

**修复**：从模板创建沙箱后，立即调用：
```python
new_sandbox.network.observe()
```
要避免影响所有未来沙箱，请在模板处于仅审计模式时制作快照。

---

## 审计日志

### 11. 审计日志为空——运行中的沙箱没有任何记录

**症状**：`sandbox.audit.list(since="24h")` 返回空列表，尽管沙箱正在运行并发出调用。

**根因**：可能是以下原因：
- 沙箱没有发出任何*出站*调用（审计日志只记录出口流量，不记录入站流量）
- 沙箱创建于租户启用 MIOSA 出口功能之前

**修复**：确认沙箱内部存在出站流量：
```python
r = sandbox.exec("curl -s -o /dev/null -w '%{http_code}' https://api.openai.com/")
print(r.stdout)  # 401 is fine  -  it proves egress happened
```
然后重新查询审计日志。如果确认有出口流量但日志仍为空，请提交支持工单。

---

### 12. 即使绑定了机密，审计行仍显示 `request_transforms.secrets.swapped = []`

**症状**：发往 `api.openai.com` 的请求成功，但审计行显示没有替换任何机密。

**根因**：机密绑定是在*该次请求之后*添加的，或者沙箱从 `.env` 文件解析到了真实密钥而不是占位符（因此无需替换）。

**修复**：检查环境变量中包含的是占位符还是真实密钥：
```python
r = sandbox.exec("echo $OPENAI_API_KEY")
print(r.stdout)
# "miosa-tok-..." means placeholder is present → proxy will swap
# "sk-proj-..."  means real key is in the env → proxy has nothing to swap
```
如果环境变量中是真实密钥（例如来自 `.env` 文件），智能体永远不会看到占位符，也就不会发生替换。

---

## 白标

### 13. 按 `external_user_id` 查询审计日志却返回其他用户的记录

**症状**：`client.audit.list(external_user_id="alice@example.com")` 返回明显属于其他用户的事件。

**根因**：这不应该发生——`external_user_id` 在您的租户内是严格相等筛选。最常见的原因是您自己的代码中应用了通配符匹配。

**修复**：确认您传入的是精确 ID，而不是模式：
```python
# Correct
events = client.audit.list(external_user_id="alice@example.com")

# Wrong  -  this is not a prefix match, but double-check your exact string
events = client.audit.list(external_user_id="alice@")  # won't match anything
```
如果问题仍然存在，请检查是否错误地用同一个 `external_user_id` 创建了多个沙箱。

---

### 14. `scope="external_user"` 机密返回 403

**症状**：调用 `client.secrets.set(..., scope="external_user", external_user_id="...")` 返回 HTTP 403。

**根因**：您的租户已禁用“Allow end-user-scoped credentials”。

**修复**：在 MIOSA 控制台 → Settings → Security 中，将“Allow end-user-scoped credentials”切换为 On。如果您是有意禁用，且此调用来自您自己的后端（而非最终用户），请改用 `scope="tenant"` 或 `scope="external_workspace"`。

---

### 15. 解析器范围查找跳过了预期机密

**症状**：带有 `external_user_id="alice"` 的沙箱没有获取到为 `external_workspace_id="team-a"` 设置的 `scope="external_workspace"` 机密，尽管 Alice 属于 team-a。

**根因**：解析器的范围查找使用沙箱创建时写入的 `external_workspace_id`。如果沙箱创建时没有 `external_workspace_id="team-a"`，工作区级机密就不会为它解析。

**修复**：验证沙箱带有正确的工作区标记：
```python
sbx = client.sandboxes.get(sandbox_id)
print(sbx.external_workspace_id)   # should be "team-a"
```
如果缺失或错误，请使用正确的归属信息创建新沙箱。归属字段在创建后不可更改。

---

## 仍然无法解决？

- 使用审计日志 → 详情抽屉：每一行都会准确显示执行了哪些转换、触发了哪些规则。
- 完整参考文档请查看[网络允许列表](/docs/security/network)、[机密](/docs/security/secrets)、[连接账户](/docs/security/connect-accounts)。
- 如遇缺陷，请从 MIOSA 控制台提交支持工单——附上沙箱 ID 和相关审计行 ID。
