---
slug: "security"
title: "安全"
description: "每个 MIOSA 沙箱和计算机都内置凭证保险库、出站网络控制和审计日志。无需设置。所有计划免费。"
section: "Security & compliance"
order: 14000
---

每个 MIOSA 沙箱和计算机都自带内置安全边界。**无需设置。无需额外购买 SKU。所有计划免费。**

三件可随时使用的功能：

| 功能 | 作用 | 页面 |
|---|---|---|
| **机密** | 将 API 密钥存储在 MIOSA 中——沙箱只能看到不透明占位符，真实值在静态存储时保持加密 | [机密](/docs/security/secrets) |
| **连接账户** | 基于 OAuth 的凭证连接（GitHub、Google、Slack……），无需粘贴原始令牌 | [连接账户](/docs/security/connect-accounts) |
| **网络白名单** | 观察沙箱发起的调用；准备好后锁定为仅允许已批准的列表 | [网络](/docs/security/network) |
| **审计日志** | 出站调用记录到出口审计日志——可搜索、可导出、可实时跟踪（当前版本尚未启用捕获） | [审计日志](/docs/security/audit-log) |

## 60 秒快速入门

无需安装。无需全局配置。每个计算机/沙箱都已具备这三个标签页。

### 1. 添加凭证

打开任意沙箱或计算机 → **机密**标签页 → **+ 添加机密**

```text
Name:         openai_key
Value:        sk-proj-...   ← your real key
Expose as:    OPENAI_API_KEY
```

或通过 SDK（Python——TypeScript / Go / Java / Elixir 中的形式相同）：

```python
sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

在沙箱内，`process.env.OPENAI_API_KEY` 现在是一个不透明的 `miosa-tok-…` 占位符。你现有的代码照常使用它。真实密钥会在出站请求经过网络边界时被替换进去。

→ 完整演练：[机密](/docs/security/secrets)

### 2. 无需修改代码

```python
import os, requests

requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
    json={...},
)
```

就是这样。OpenAI 看到的是你的真实 `sk-…`。你的沙箱始终看不到。无需修改代码。

### 3.（可选）检查调用情况并锁定

沙箱运行一段时间后：

- **审计日志标签页**——实时跟踪出站调用。可按主机、状态、时间筛选。当前版本尚未启用出口审计捕获。
- **网络标签页**——查看自动构建的已联系主机列表。一键进入强制执行模式，将该列表作为白名单。

两者均为可选启用。在你决定锁定之前，默认仅审计。

→ 锁定流程请参阅[网络白名单](/docs/security/network)。

### 想接入 GitHub / Google / Slack，而不是粘贴令牌？

→ [连接账户](/docs/security/connect-accounts)——自动处理刷新的 OAuth 流程。

### 在 MIOSA 之上构建白标产品？

你的最终用户无需拥有 MIOSA 账户——凭证可以限定到你自己的 `external_user_id` 范围内。

→ [白标凭证](/docs/security/white-label)

---

## 你的代码无需更改

MIOSA 安全的整体设计原则：**你现有的代码继续正常工作。**

```python
import os, requests

# This code is identical with or without MIOSA Secrets:
response = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
    json={...},
)
```

变化的是 `OPENAI_API_KEY` *的值*。不使用机密功能时，它是你真实的 `sk-...` 密钥。使用机密功能时，它是一个不透明占位符，例如 `miosa-tok-7f2a8c...`。MIOSA 智能体会在出站时替换为真实密钥。OpenAI 看到真实密钥。你的沙箱始终看不到。

## 三项保证

> **注意：默认仅审计**
>
> 新沙箱不会阻止任何请求。智能体只观察并记录，绝不会干扰。只有显式点击“锁定”后才会进入“强制执行”模式——而且一键即可撤销。

> **注意：机密保持加密**
>
> 保存后，机密值使用 ChaCha20-Poly1305 加密。只有在智能体需要将其替换进出站请求的那一刻才会解封。任何人——包括 MIOSA 工程师——都无法通过 API 或 UI 重新读取已存储的值。

> **注意：现有模式仍然有效**
>
> `.env` 文件仍然有效。`tenant_provider_keys` 和 `workspace_env_vars` 仍然有效。MIOSA 机密功能只是增加了一个新层；它不会替换任何现有内容。

## 下一步

- **第一次使用？** 从[机密——快速入门](/docs/security/secrets)开始。
- **想接入 OAuth（GitHub 等）？** 请阅读[连接账户](/docs/security/connect-accounts)。
- **想锁定出站流量？** 请阅读[网络白名单](/docs/security/network)。
- **在 MIOSA 上构建白标产品？** 请阅读[白标凭证](/docs/security/white-label)。

## 术语表

**占位符令牌**
MIOSA 注入到沙箱环境中、用于代替真实凭证的不透明字符串（格式：`miosa-tok-<hex>`）。智能体会在每次出站请求时用真实值替换该占位符。每个绑定对应的占位符都是唯一的——无法通过暴力破解发现真实密钥。

**OAuth 连接**
一项 MIOSA 功能，允许你通过标准 OAuth 2.0 流程授权第三方账户（GitHub、Google、Slack……），而无需粘贴静态 API 密钥。MIOSA 会存储生成的访问令牌和刷新令牌，自动刷新，并将当前有效的访问令牌作为占位符环境变量暴露给沙箱。

**解析器作用域遍历**
当沙箱发起出站请求、智能体需要将占位符匹配到真实凭证时，MIOSA 执行的有序查找。智能体按以下优先级检查作用域：(1) 直接沙箱绑定，(2) 用户作用域机密，(3) external_user 作用域机密，(4) external_workspace 作用域机密，(5) MIOSA 工作区机密，(6) 租户范围机密。首个匹配项胜出。如果没有任何匹配，占位符将原样传递，上游 API 调用失败（这是预期的安全行为）。

## 真实示例——5 分钟构建 OpenAI 客服智能体

这是一个安全使用 OpenAI 的 AI 智能体的完整设置。运行以下五条命令即可完成。

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# 1. Create sandbox
sandbox = client.sandboxes.create(template_id="python-ds")

# 2. Store the OpenAI key  -  sandbox never sees the real value
sandbox.secrets.set(
    name="openai_key",
    value=os.environ["MY_OPENAI_KEY"],
    expose_as_env="OPENAI_API_KEY",
)

# 3. Lock down to only api.openai.com
sandbox.network.allow("api.openai.com")
sandbox.network.lockdown()

# 4. Run the agent  -  uses $OPENAI_API_KEY exactly like normal
result = sandbox.exec("""
python3 -c "
import os, requests
resp = requests.post(
    'https://api.openai.com/v1/chat/completions',
    headers={'Authorization': f'Bearer {os.environ[\"OPENAI_API_KEY\"]}'},
    json={'model': 'gpt-4o-mini', 'messages': [{'role': 'user', 'content': 'Hello!'}]}
)
print(resp.json()['choices'][0]['message']['content'])
"
""")
print(result.stdout)

# 5. Check the audit trail
events = sandbox.audit.list(since="5m")
for e in events:
    print(e.host, e.action, e.status_code)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

// 1. Create sandbox
const sandbox = await client.sandboxes.create({ template: 'python-ds' });

// 2. Store the OpenAI key
await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});

// 3. Lock down
await sandbox.network.allow('api.openai.com');
await sandbox.network.lockdown();

// 4. Run the agent
const result = await sandbox.exec(`
python3 -c "
import os, requests
resp = requests.post(
    'https://api.openai.com/v1/chat/completions',
    headers={'Authorization': f'Bearer {os.environ[\"OPENAI_API_KEY\"]}'},
    json={'model': 'gpt-4o-mini', 'messages': [{'role': 'user', 'content': 'Hello!'}]}
)
print(resp.json()['choices'][0]['message']['content'])
"
`);
console.log(result.stdout);

// 5. Audit trail
const events = await sandbox.audit.list({ since: '5m' });
for (const e of events) {
  console.log(e.host, e.action, e.statusCode);
}
```

## 故障排查

**我的沙箱环境变量包含占位符，但 API 返回 401。**
智能体无法匹配出站请求标头中的占位符。确认 SDK 将令牌放在 `Authorization: Bearer ...` 中。请参阅[故障排查 #1](/docs/security/troubleshooting#1-secret-value-is-not-being-swapped-outbound-request-uses-the-raw-placeholder)。

**我启用了锁定，但什么都不工作。**
调用 `sandbox.network.pending()` 查看被阻止的主机，然后将其加入允许列表。或者在诊断期间用 `sandbox.network.observe()` 恢复。

**我看不到任何审计记录。**
审计日志仅记录出站调用。如果沙箱尚未发起任何出站请求，日志就是空的。触发一次调用后重新查询。

**GitHub 机密上显示“需要重新连接”。**
OAuth 令牌已被用户撤销或已过期。调用 `sandbox.secrets.reauthorize(provider="github")` 重新执行 OAuth 授权流程。请参阅[连接账户故障排查](/docs/security/troubleshooting#4-github-oauth-refresh-failing--sandbox-gets-401-on-apigithubcom)。

完整的问题与修复列表，请参阅[安全故障排查中心](/docs/security/troubleshooting)。
