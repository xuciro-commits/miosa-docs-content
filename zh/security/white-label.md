---
slug: "security/white-label"
title: "最终用户凭据和 OAuth"
description: "如何在 MIOSA 之上将机密和 OAuth 连接限定到您自己的最终用户——使用您的品牌，而不是我们的。"
section: "Platform builders"
order: 9010
---

如果您正在 **MIOSA 之上**构建产品（例如 AI 编程智能体、垂直 SaaS 或智能体机构工具），并且最终用户通过*您的* UI 创建沙箱，MIOSA Security 可以将他们作为一等身份提供支持——即使他们没有 MIOSA 账户。

本页涵盖：

1. 使用 `external_user_id` 将沙箱归属到您的最终用户
2. 存储限定到这些最终用户的凭据
3. 将 OAuth 授权同意流程品牌化为您自己的产品
4. 按最终用户、按工作区进行审计

## 背景——外部归属

每台计算机和每个沙箱都已支持三个由调用方提供的归属字段：

| 字段 | 用途 |
|---|---|
| `external_user_id` | 您的最终用户在您系统中的身份 |
| `external_workspace_id` | 您的工作区/团队分组 |
| `external_project_id` | 您的项目身份 |

这些是标签，不是身份验证边界。但 MIOSA Security 将它们用作**凭据范围限定键**，这意味着您可以存储只对*该*最终用户的资源生效的机密。

更全面的说明参见[外部归属](/docs/platform/attribution)。

## 在创建时写入归属信息

当您的后端为某个最终用户创建沙箱时，请传入归属信息：

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="python-ds",
    external_user_id="dana@example.net",
    external_workspace_id="acme",
    external_project_id="intake-flow",
)
```

**TypeScript**

```typescript
const sandbox = await client.sandboxes.create({
  template: 'node-web',
  externalUserId: 'dana@example.net',
  externalWorkspaceId: 'acme',
  externalProjectId: 'intake-flow',
});
```

**Go**

```go
sandbox, err := client.Sandboxes.Create(ctx, miosa.SandboxCreate{
    Template:            "python-ds",
    ExternalUserID:      "dana@example.net",
    ExternalWorkspaceID: "acme",
    ExternalProjectID:   "intake-flow",
})
```

**Java**

```java
Sandbox sandbox = client.sandboxes().create(SandboxCreate.builder()
    .template("python-ds")
    .externalUserId("dana@example.net")
    .externalWorkspaceId("acme")
    .externalProjectId("intake-flow")
    .build());
```

**Elixir**

```elixir
{:ok, sandbox} = Miosa.Sandboxes.create(%{
  template_id: "python-ds",
  external_user_id: "dana@example.net",
  external_workspace_id: "acme",
  external_project_id: "intake-flow"
})
```

您选择的值对 MIOSA 来说是不透明的——它们可以是您系统中任何有意义的 ID。

## 存储限定到最终用户的凭据

假设 Dana 在您的 UI 中粘贴了她自己的 OpenAI 密钥。请将其保存为限定到她：

```python
client.secrets.set(
    name="openai_key",
    value="sk-proj-her-personal-key...",
    scope="external_user",
    external_user_id="dana@example.net",
    expose_as_env="OPENAI_API_KEY",
)
```

现在，您用 `external_user_id="dana@example.net"` 创建的任何沙箱都会将 `OPENAI_API_KEY` 解析为她的密钥。同一账户中的其他成员**不会**看到它。

对于账户级凭据（例如共享的公司密钥）：

```python
client.secrets.set(
    name="ehr_api_key",
    value="...",
    scope="external_workspace",
    external_workspace_id="acme",
    expose_as_env="EHR_API_KEY",
)
```

## 智能体如何决定使用哪个凭据

当沙箱发起出站调用且智能体需要替换占位符时，会按以下优先级顺序查找：

```text
1. Direct binding on this specific sandbox
2. Secret scoped to user (owner_user_id  -  for MIOSA-native users)
3. Secret scoped to external_user (external_user_id match)
4. Secret scoped to external_workspace (external_workspace_id match)
5. Secret scoped to MIOSA workspace
6. Tenant-wide secret
```

先匹配者胜出。如果没有任何匹配，占位符会留在请求中，上游调用随之失败（这才是正确的行为——总比悄悄泄露要好）。

## 白标 OAuth（使用您的品牌，而不是 MIOSA 的）

默认情况下，当您的某个最终用户在 UI 中点击“Connect GitHub”时，他们会被发送到 GitHub，授权同意屏幕显示 **“Authorize MIOSA”**。要获得真正的白标体验，请覆盖该设置：

### 1. 在提供商处注册您自己的 OAuth 应用

以 GitHub 为例：

```text
https://github.com/settings/applications/new

Application name:  Lumen Cloud
Homepage URL:      https://lumen.example
Authorization callback URL:  https://app.lumen.example/oauth/callback
```

### 2. 将凭据填入 MIOSA

在租户管理控制台中依次进入 Settings → OAuth Providers → “Add provider override”：

```text
Provider:       github
Display name:   GitHub
Client ID:      <from GitHub>
Client Secret:  <from GitHub>
Callback URL:   https://app.lumen.example/oauth/callback
Branding:       { "button_label": "Connect GitHub to Lumen" }
```

从现在起，当您的最终用户连接 GitHub 时，他们会看到“Authorize Lumen Cloud”——您的名称、您的品牌、您的回调域名。

### 3. 确保回调智能体到 MIOSA

您的 `https://app.lumen.example/oauth/callback` 路由会接收 OAuth `code` 参数，并将其转发到 MIOSA 的回调端点：

```python
@app.get("/oauth/callback")
def oauth_callback(request):
    code = request.query_params["code"]
    state = request.query_params["state"]
    # Forward to MIOSA
    return miosa.oauth.complete(provider="github", code=code, state=state)
```

MIOSA 会代表您将 code 兑换为令牌，并将其存储在保险库中。

## 按用户审计

审计日志的每一行都携带完整身份链，因此您可以按自己的用户 ID 查询：

```python
# All activity by one end-user
events = client.audit.list(
    since="7d",
    external_user_id="dana@example.net",
)

# All activity in one workspace
events = client.audit.list(
    since="30d",
    external_workspace_id="acme",
)
```

UI 中也可使用相同的筛选条件。

## 隐私边界

租户管理员（也就是作为白标运营者的您）可以：

- ✅ 查看某个 `external_user_id` 是否绑定了机密
- ✅ 读取审计行，了解每次调用解析了哪个凭据
- ✅ 禁用机密以立即撤销访问
- ❌ 重新读取任何已存机密的加密值

“您不能读取已存值”这一规则适用于 MIOSA 中的每一位参与者——包括平台工程师。如果您的最终用户问“Lumen 在我保存后能看到我的 OpenAI 密钥吗”，答案是：“不能，值是加密的；只有离开您沙箱的请求体会在传输过程中短暂包含它。”

## 完全禁用最终用户范围的凭据

有些白标客户希望所有凭据集中管理——最终用户不能自带密钥。可在租户设置中切换：

```text
Settings → Security → "Allow end-user-scoped credentials" → Off
```

关闭后，只有 `tenant` 和 `external_workspace` 范围的机密会生效。创建最终用户范围的机密会返回 403。

## 真实示例——Lumen：每个最终用户一个沙箱

Lumen 是一个构建在 MIOSA 上的 SaaS。Lumen 的每位最终用户都有自己的沙箱。他们通过 Lumen 的设置页提供自己的 OpenAI 密钥。审计日志展示在 Lumen 自己的控制台中。GitHub OAuth 显示“Authorize Lumen”，而不是“Authorize MIOSA”。

**Python**

```python
import os
from miosa import Miosa
from flask import Flask, request, jsonify, redirect

app_server = Flask(__name__)
client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def get_or_create_sandbox(member_id: str, account_id: str):
    """Return existing sandbox or create a new one for this member."""
    sandboxes = client.sandboxes.list(external_user_id=member_id)
    if sandboxes:
        return sandboxes[0]
    return client.sandboxes.create(
        template_id="python-ds",
        external_user_id=member_id,
        external_workspace_id=account_id,
        external_project_id="agent-workspace",
    )

@app_server.post("/api/members/<member_id>/openai-key")
def save_openai_key(member_id: str):
    """Member saves their own OpenAI key through the Lumen settings page."""
    data = request.json
    client.secrets.set(
        name="openai_key",
        value=data["api_key"],
        scope="external_user",
        external_user_id=member_id,
        expose_as_env="OPENAI_API_KEY",
    )
    return jsonify({"status": "saved"})

@app_server.post("/api/members/<member_id>/connect-github")
def connect_github(member_id: str):
    """Start GitHub OAuth for a member."""
    account_id = request.json["account_id"]
    sbx = get_or_create_sandbox(member_id, account_id)
    flow = sbx.secrets.connect(
        provider="github",
        expose_as_env="GITHUB_TOKEN",
        scopes=["repo", "read:user"],
    )
    return jsonify({"authorize_url": flow.authorize_url})

@app_server.get("/oauth/callback")
def oauth_callback():
    """OAuth callback forwarded from Lumen's branded domain."""
    client.oauth.complete(
        provider="github",
        code=request.args["code"],
        state=request.args["state"],
    )
    return redirect("/dashboard?connected=github")

@app_server.get("/api/members/<member_id>/activity")
def member_activity(member_id: str):
    """Per-member audit feed for the Lumen compliance dashboard."""
    events = client.audit.list(
        since="7d",
        external_user_id=member_id,
        limit=200,
    )
    return jsonify([{
        "time": e.timestamp,
        "host": e.host,
        "action": e.action,
        "status": e.status_code,
    } for e in events])
```

**TypeScript**

```typescript
import express from 'express';
import { Miosa } from '@miosa/sdk';

const app = express();
app.use(express.json());
const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function getOrCreateSandbox(memberId: string, accountId: string) {
  const sandboxes = await client.sandboxes.list({ externalUserId: memberId });
  return sandboxes[0] ?? client.sandboxes.create({
    template: 'python-ds',
    externalUserId: memberId,
    externalWorkspaceId: accountId,
    externalProjectId: 'agent-workspace',
  });
}

app.post('/api/members/:memberId/openai-key', async (req, res) => {
  await client.secrets.set({
    name: 'openai_key',
    value: req.body.apiKey,
    scope: 'external_user',
    externalUserId: req.params.memberId,
    exposeAsEnv: 'OPENAI_API_KEY',
  });
  res.json({ status: 'saved' });
});

app.post('/api/members/:memberId/connect-github', async (req, res) => {
  const sbx = await getOrCreateSandbox(req.params.memberId, req.body.accountId);
  const flow = await sbx.secrets.connect({
    provider: 'github',
    exposeAsEnv: 'GITHUB_TOKEN',
    scopes: ['repo', 'read:user'],
  });
  res.json({ authorizeUrl: flow.authorizeUrl });
});

app.get('/oauth/callback', async (req, res) => {
  await client.oauth.complete({
    provider: 'github',
    code: req.query.code as string,
    state: req.query.state as string,
  });
  res.redirect('/dashboard?connected=github');
});

app.get('/api/members/:memberId/activity', async (req, res) => {
  const events = await client.audit.list({
    since: '7d',
    externalUserId: req.params.memberId,
    limit: 200,
  });
  res.json(events.map(e => ({ time: e.timestamp, host: e.host, action: e.action, status: e.statusCode })));
});
```

## 故障排查

**沙箱找不到我为最终用户存储的机密——环境变量为空。**
创建沙箱时使用的 `external_user_id` 必须与存储机密时使用的相同。归属信息在创建时写入，之后无法更改。请用 `client.sandboxes.get(sandbox_id).external_user_id` 验证。

**`scope="external_user"` 返回 HTTP 403。**
您的租户已禁用“Allow end-user-scoped credentials”。请在 Settings → Security 中启用，或改用 `scope="external_workspace"` 存储工作区级凭据。

**解析器跳过了我的工作区范围机密。**
沙箱创建时必须带有匹配的 `external_workspace_id`。如果沙箱创建时没有该字段，工作区级的范围查找项就是空值，不会匹配。

**添加提供商覆盖后，OAuth 授权同意屏幕仍显示“Authorize MIOSA”。**
请仔细检查覆盖配置中的 `client_id` 是否与您在提供商处注册的 OAuth 应用一致，而不是 MIOSA 的默认客户端 ID。如有需要，请删除并重新添加覆盖配置。

**我的最终用户能在 MIOSA 控制台中看到其他用户的沙箱。**
最终用户不应拥有 MIOSA 账户或 API 密钥。如果他们已有，请改为创建限定到该用户的浏览器令牌：`client.browser_tokens.create(external_user_id="...")`。浏览器令牌是只读的，并且限定到该用户的资源。

更多问题参见[安全故障排查中心](/docs/security/troubleshooting)。

## 常见问题

**我的最终用户没有 MIOSA 账户。他们仍然可以使用 OAuth Connect 吗？**
可以。OAuth 流程会将生成的令牌归属到您后端在发起流程时传入的任意 `external_user_id`。

**同一个 `external_user_id` 可以存在于多个租户中吗？**
可以——`external_user_id` 在租户内限定范围。相同字符串在两个不同租户中代表两个独立身份。

**如果某个最终用户同时属于我的两个工作区，会怎样？**
您可以为每个工作区创建相应范围的凭据——智能体会根据发起请求的沙箱解析出正确的那一个。

**我的最终用户能自己查看审计日志吗？**
这由您决定——MIOSA 提供审计数据，如何白标 UI 中展示由您决定。大多数白标客户会在自己的控制台中向最终用户展示“您的活动”。

**如何将现有的应用内机密存储迁移到 MIOSA Secrets？**
通过 SDK 批量导入是最简单的方式：

```python
for user_id, openai_key in your_existing_secret_store.items():
    client.secrets.set(
        name="openai_key",
        value=openai_key,
        scope="external_user",
        external_user_id=user_id,
        expose_as_env="OPENAI_API_KEY",
    )
```

验证完成后，请从旧存储中删除。
