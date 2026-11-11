---
slug: "security/connect-accounts"
title: "连接账户（OAuth）"
description: "一次性授权账户（GitHub、Google、Slack 等）。MIOSA 管理令牌刷新；你的沙盒只会看到一个环境变量。"
section: "Sandboxes"
order: 2019
---

不必粘贴个人访问令牌或 API 密钥，而是直接向上游提供商**授权账户**。MIOSA 存储访问令牌和刷新令牌，在后台刷新它们，并将当前有效的访问令牌以普通环境变量的形式提供给沙盒。

## 何时使用此功能而非密钥

| 你的情况…… | 使用 |
|---|---|
| 拥有 API 密钥字符串（例如 OpenAI `sk-…`、Stripe `sk_live_…`） | [密钥](/docs/security/secrets) |
| 提供商支持 OAuth 的账户（GitHub、Google、Slack 等） | **连接账户** |
| 机器对机器的 OAuth2 客户端（client_id + client_secret） | **连接账户**（机器 OAuth） |

## 连接 GitHub 账户（典型流程）

### 从控制台

```text
1. Sandbox detail → Secrets tab → "+ Connect Account"
2. Pick "GitHub" from the provider grid
3. Browser opens to github.com/login/oauth/authorize
4. Approve the scopes you want to grant
5. You bounce back to MIOSA. Done.
```

新的条目会出现在你的密钥列表中：

```yaml
Name:     github_token
Type:     OAuth Connect
Provider: GitHub
Scopes:   repo, read:user
Expires:  in 8 hours (auto-refreshed)
```

在沙盒内：

```bash
$ echo $GITHUB_TOKEN
miosa-tok-9a3c5e7b1f8d2a4c...
```

像使用真实令牌一样使用它——替换会在出口处完成：

```python
requests.get(
    "https://api.github.com/user/repos",
    headers={"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"},
)
```

访问令牌过期时，MIOSA 会在后台使用刷新令牌生成新令牌。你的代码无需感知。

### 通过 SDK

**Python**

```python
flow = sandbox.secrets.connect(
    provider="github",
    expose_as_env="GITHUB_TOKEN",
)

# `flow.authorize_url` is a URL the end user opens in their browser.
print("Authorize at:", flow.authorize_url)

# Block until the user completes the OAuth dance:
flow.wait_for_completion(timeout=300)
```

**TypeScript**

```ts
const flow = await sandbox.secrets.connect({
  provider: 'github',
  exposeAsEnv: 'GITHUB_TOKEN',
});

console.log('Authorize at:', flow.authorizeUrl);
await flow.waitForCompletion({ timeoutSec: 300 });
```

**Go**

```go
flow, err := sandbox.Secrets.Connect(ctx, miosa.OAuthConnect{
    Provider:    "github",
    ExposeAsEnv: "GITHUB_TOKEN",
    Scopes:      []string{"repo", "read:user"},
})
if err != nil { panic(err) }

fmt.Println("Authorize at:", flow.AuthorizeURL)
if err := flow.WaitForCompletion(ctx, 300); err != nil { panic(err) }
```

**Java**

```java
OAuthFlow flow = sandbox.secrets().connect(OAuthConnect.builder()
    .provider("github")
    .exposeAsEnv("GITHUB_TOKEN")
    .scopes(List.of("repo", "read:user"))
    .build());

System.out.println("Authorize at: " + flow.getAuthorizeUrl());
flow.waitForCompletion(Duration.ofSeconds(300));
```

**Elixir**

```elixir
{:ok, flow} = Miosa.Secrets.connect(sandbox.id, %{
  provider: "github",
  expose_as_env: "GITHUB_TOKEN",
  scopes: ["repo", "read:user"]
})

IO.puts("Authorize at: #{flow.authorize_url}")
{:ok, _} = Miosa.Secrets.wait_for_oauth(flow.id, timeout: 300)
```

## 支持的提供商

| 提供商 | 状态 |
|---|---|
| GitHub | ✅ 可用 |
| Google（Gmail、Drive、日历） | 逐步推出 |
| Slack | 逐步推出 |
| Linear | 逐步推出 |
| Notion | 逐步推出 |
| Stripe | 计划中 |
| HubSpot | 计划中 |
| 自定义 OAuth2 提供商 | 计划中——提交请求 |

在我方添加新的知名提供商只需修改配置——无需发布 SDK。

## 机器 OAuth（客户端凭证）

适用于使用 OAuth2 客户端凭证流程（没有最终用户，机器对机器）的提供商 API：

```python
sandbox.secrets.add_machine_oauth(
    name="my_internal_api",
    client_id="abc...",
    client_secret="def...",
    token_url="https://my-api.example.com/oauth/token",
    scopes=["read", "write"],
    expose_as_env="INTERNAL_API_TOKEN",
)
```

MIOSA 会针对 `token_url` 签发并缓存访问令牌，在过期前刷新，并作为 `INTERNAL_API_TOKEN` 注入。

## 智能体会注入什么

对于 OAuth Connect 密钥，智能体会注入：

- 默认注入**访问令牌**（最常见的情况——`Authorization: Bearer …`）
- **遇到 401 时刷新**：如果上游返回 401，智能体会先尝试刷新，再将错误抛给你的代码

刷新令牌本身**绝不会**暴露给沙盒。它加密保存在 MIOSA 的保险库中。

## 轮换、撤销与范围变更

- **轮换**：自动进行。你无需做任何事。
- **撤销**：打开 Secrets 标签页 → 点击“断开连接” → MIOSA 会立即销毁刷新令牌。下一次出站请求会以 401 失败。
- **范围变更**：需要重新授权。点击密钥上的“重新授权”；你会带着新的作用域再次完成 OAuth 授权流程。

## 白标 OAuth（使用你的品牌，而非 MIOSA 品牌）

默认情况下，最终用户在 OAuth 同意屏幕上会看到“授权 MIOSA 访问你的 GitHub”。对于白标部署，请注册你自己的 OAuth 应用，并让授权流程使用你的品牌——请参阅[白标凭证](/docs/security/white-label)。

## 实际示例——智能体读取 GitHub issue 并向 Slack 发布更新

通过 OAuth 连接 GitHub 和 Slack。智能体读取未关闭的 issue，并向 Slack 频道发布每日摘要。凭据绝不会进入沙盒。

**Python**

```python
import os, threading
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
sandbox = client.sandboxes.create(template_id="python-ds")

# Connect GitHub
github_flow = sandbox.secrets.connect(
    provider="github",
    expose_as_env="GITHUB_TOKEN",
    scopes=["repo", "read:user"],
)
print("1. Authorize GitHub at:", github_flow.authorize_url)
github_flow.wait_for_completion(timeout=300)
print("   GitHub connected.")

# Connect Slack
slack_flow = sandbox.secrets.connect(
    provider="slack",
    expose_as_env="SLACK_BOT_TOKEN",
    scopes=["chat:write", "channels:read"],
)
print("2. Authorize Slack at:", slack_flow.authorize_url)
slack_flow.wait_for_completion(timeout=300)
print("   Slack connected.")

# Write and run the agent
agent_code = '''
import os, requests, json

GITHUB_TOKEN = os.environ["GITHUB_TOKEN"]
SLACK_TOKEN  = os.environ["SLACK_BOT_TOKEN"]
REPO         = "your-org/your-repo"
CHANNEL      = "#engineering-digest"

# Fetch open issues
issues = requests.get(
    f"https://api.github.com/repos/{REPO}/issues?state=open&per_page=10",
    headers={"Authorization": f"Bearer {GITHUB_TOKEN}"},
).json()

# Build digest message
lines = [f"*Open issues digest for {REPO}*"]
for i in issues:
    lines.append(f"• <{i['html_url']}|#{i['number']} {i['title']}>")
message = "\\n".join(lines)

# Post to Slack
resp = requests.post(
    "https://slack.com/api/chat.postMessage",
    headers={"Authorization": f"Bearer {SLACK_TOKEN}"},
    json={"channel": CHANNEL, "text": message},
)
print("Posted:", resp.json().get("ok"))
'''

sandbox.files.write("/workspace/digest.py", agent_code)
result = sandbox.exec("python /workspace/digest.py")
print(result.stdout)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.create({ template: 'node-web' });

// Connect GitHub
const githubFlow = await sandbox.secrets.connect({
  provider: 'github',
  exposeAsEnv: 'GITHUB_TOKEN',
  scopes: ['repo', 'read:user'],
});
console.log('1. Authorize GitHub at:', githubFlow.authorizeUrl);
await githubFlow.waitForCompletion({ timeoutSec: 300 });
console.log('   GitHub connected.');

// Connect Slack
const slackFlow = await sandbox.secrets.connect({
  provider: 'slack',
  exposeAsEnv: 'SLACK_BOT_TOKEN',
  scopes: ['chat:write', 'channels:read'],
});
console.log('2. Authorize Slack at:', slackFlow.authorizeUrl);
await slackFlow.waitForCompletion({ timeoutSec: 300 });
console.log('   Slack connected.');

// Run the digest agent
const result = await sandbox.exec('node /workspace/digest.js');
console.log(result.stdout);
```

## 故障排查

**`flow.wait_for_completion` 超时——用户已授权，但流程始终不结束。**
通常意味着 OAuth 回调 URL 错误。请验证提供商侧 OAuth 应用设置中的回调 URL 与 MIOSA 期望的一致。对于白标设置，确保你的智能体端点将 `code` 和 `state` 参数转发给 `client.oauth.complete()`。

**GitHub OAuth 返回 "bad_verification_code"。**
授权码是一次性的，TTL 很短（GitHub 为 10 分钟）。如果 `wait_for_completion` 耗时过长，或回调被调用两次，授权码就会失效。重新发起 `sandbox.secrets.connect()` 以获取新流程。

**令牌已过期——沙盒从 GitHub 收到 401。**
MIOSA 会自动刷新令牌。如果刷新失败（用户撤销了令牌），控制台会显示“需要重新连接”。调用 `sandbox.secrets.reauthorize(provider="github")`，让用户重新授权。

**令牌刷新期间出现 "invalid_grant"。**
GitHub（以及大多数提供商）会在用户撤销 OAuth 应用权限时使刷新令牌失效。通过 `sandbox.secrets.reauthorize()` 重新连接。

**Slack 消息发送成功，但令牌出现在消息正文中。**
当 Slack SDK 将令牌放入请求正文而不是 `Authorization` 头时，就会发生这种情况。MIOSA 智能体会扫描缓冲的正文，但 Slack 标准的 `Bearer` 头才是可靠路径。确保使用 `Authorization: Bearer $SLACK_BOT_TOKEN`。

更多问题请参阅[安全故障排查中心](/docs/security/troubleshooting)。

## 常见问题

**我的 OAuth 提供商不在列表中。可以手动添加吗？**
可以——提交请求，或在 Custom OAuth2 路径上线后使用该路径（第 3 阶段）。目前，任何签发静态访问令牌的提供商都可以作为普通 API 密钥通过[密钥](/docs/security/secrets)使用。

**沙盒能看到刷新令牌吗？**
不会。只会注入当前访问令牌。刷新令牌加密保存在 MIOSA 中。

**可以把一个 OAuth 账户绑定到多个沙盒吗？**
可以。将密钥保存在 `tenant`、`workspace` 或 `user` 作用域；该作用域内的每个沙盒都会继承此绑定。

**如果提供商在其侧撤销了我的令牌，会发生什么？**
智能体下一次刷新尝试会失败。我们会在控制台中以“需要重新连接”徽标在密钥上提示，你的下一次出站请求会返回 401。
