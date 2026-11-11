---
slug: "security/secrets"
title: "机密 - 快速入门"
description: "在 MIOSA 中存储 API 密钥。你的沙箱使用不透明占位符，真实密钥永远不会存在虚拟机内。"
section: "Sandboxes"
order: 2018
---

只需在 MIOSA 中存储一次 API 密钥。所有使用该密钥的沙箱都会看到一个不透明的占位符环境变量。在一个地方轮换，所有沙箱就会自动获取新值。

## 添加 API 密钥（30 秒）

### 从控制台添加

```text
1. Open your Sandbox or Computer in the MIOSA dashboard
2. Click the "Secrets" tab
3. Click "+ Add Secret"
4. Fill in:
     Name:        openai_key
     Value:       sk-proj-...  (paste your real key)
     Expose as:   OPENAI_API_KEY
5. Save
```

在沙箱内，`process.env.OPENAI_API_KEY` 现在是一个不透明令牌。在代码中像使用真实密钥一样使用它即可——替换会在出口处完成。

### 通过 SDK

**Python**

```python
from miosa import Miosa
client = Miosa(api_key="msk_u_...")
sandbox = client.sandboxes.create(template_id="python-ds")

sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

**TypeScript**

```ts
import { Miosa } from '@miosa/sdk';
const miosa = new Miosa({ apiKey: 'msk_u_...' });
const sandbox = await miosa.sandboxes.create({ template: 'node-web' });

await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
```

**Go**

```go
client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
sandbox, _ := client.Sandboxes.Create(ctx, miosa.SandboxCreate{Template: "go-web"})

sandbox.Secrets.Set(ctx, miosa.SecretSet{
    Name:         "openai_key",
    Value:        realKey,
    ExposeAsEnv:  "OPENAI_API_KEY",
})
```

## 占位符的工作原理

绑定完成后，在沙箱内：

```bash
$ echo $OPENAI_API_KEY
miosa-tok-7f2a8c1d4b2e6f0a...
```

当代码发起出站调用时：

```python
requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}"},
    json={...},
)
```

……虚拟机宿主机上的 MIOSA 智能体会看到该请求，在 `Authorization` 请求头中找到 `miosa-tok-7f2a8c1d4b2e6f0a`，并在转发给 OpenAI 之前**将其替换为真实的 `sk-proj-...`**。OpenAI 看到的是真实密钥，而你的沙箱始终看不到。

每个绑定都有唯一的占位符——不存在可供攻击者暴力枚举的全局占位符表。

> **注意：替换发生的位置**
>
> 请求头值、查询参数、URL 路径以及（在缓冲后）请求正文都会被扫描以查找占位符令牌。Basic 认证请求头会先解码，再替换，然后重新编码。

## 轮换密钥

```python
sandbox.secrets.rotate("openai_key", new_value="sk-proj-newvalue...")
```

所有使用此机密的沙箱都会在下一次出站请求时获取新值。无需重启，无需重新部署。占位符令牌不会改变。

## 列出和删除

```python
secrets = sandbox.secrets.list()
# Returns metadata only  -  name, type, preview (last 4 chars), last_used_at.
# Never returns the real value.

sandbox.secrets.delete("openai_key")
```

## 作用域层级

机密可以在五个层级设置作用域——智能体会为发出请求的资源选择最具体的匹配项：

| 作用域 | 使用场景 |
|---|---|
| `tenant` | 你所在组织的全公司凭据 |
| `workspace` | 一个工作区内所有人共享的凭据 |
| `user` | 单个 MIOSA 用户的个人凭据 |
| `external_user` | 白标最终用户的凭据（参见[白标](/docs/security/white-label)） |
| `external_workspace` | 白标工作区的凭据 |

使用 `sandbox.secrets.set` 时的默认作用域取决于发起调用的 API 密钥。租户管理员可以创建租户级机密；个人用户可为自己拥有的资源创建用户级机密。

## 运行时配置文件与机密的区别

智能体运行时配置文件与机密解决的是不同问题。

| 配置项 | 存放位置 | 示例 |
|---|---|---|
| 非机密运行时默认值 | 智能体运行时配置文件 `env` | `ANTHROPIC_MODEL=claude-sonnet-4.6` |
| 提供商 API 密钥 | 托管机密或连接器 | `ANTHROPIC_API_KEY`、`OPENAI_API_KEY`、`HIGGSFIELD_API_KEY` |
| 工具目录 | 智能体运行时配置文件 `tools` | `filesystem`、`shell`、`browser`、`refero` |
| 账户连接 | 连接器绑定 | GitHub 组织、Slack 工作区、CRM 租户 |
| 单次运行覆盖 | 智能体运行 `env` | `FEATURE_FLAG=on` |

对于白标产品，请使用以下层级：

```text
Tenant default
  -> workspace default
    -> project or resource binding
      -> sandbox/computer runtime
        -> run-level override
```

类似 Lumen 的平台可以为某个客户工作区一次性配置 Anthropic 密钥。之后在该工作区中启动的每个沙箱或计算机都能自动获得正确的占位符或连接器令牌，而运行时配置文件则决定启用哪种智能体运行框架和工具。

不要将长期有效的提供商机密存放在运行时配置文件的 `env` 中。配置文件用于默认值和策略；机密和连接器用于凭据。

## 智能体除了字符串替换还能做什么

对于由 OAuth 管理的凭据（参见[连接账户](/docs/security/connect-accounts)），智能体还会：

- 使用已存储的刷新令牌**自动刷新过期令牌**
- **将令牌撤销暴露出来**（当上游使访问权限失效时），以 401 形式返回给代码，就像你自己持有该令牌一样

你无需编写任何相关逻辑。

## 全部 5 种 SDK

**Python**

```python
from miosa import Miosa
client = Miosa(api_key="msk_u_...")
sandbox = client.sandboxes.create(template_id="python-ds")

sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
sandbox.secrets.rotate("openai_key", new_value="sk-proj-newvalue...")
secrets = sandbox.secrets.list()
sandbox.secrets.delete("openai_key")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';
const miosa = new Miosa({ apiKey: 'msk_u_...' });
const sandbox = await miosa.sandboxes.create({ template: 'node-web' });

await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
await sandbox.secrets.rotate({ name: 'openai_key', newValue: 'sk-proj-newvalue...' });
const secrets = await sandbox.secrets.list();
await sandbox.secrets.delete('openai_key');
```

**Go**

```go
client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
sandbox, _ := client.Sandboxes.Create(ctx, miosa.SandboxCreate{Template: "go-web"})

sandbox.Secrets.Set(ctx, miosa.SecretSet{
    Name:        "openai_key",
    Value:       realKey,
    ExposeAsEnv: "OPENAI_API_KEY",
})
sandbox.Secrets.Rotate(ctx, "openai_key", "sk-proj-newvalue...")
secrets, _ := sandbox.Secrets.List(ctx)
sandbox.Secrets.Delete(ctx, "openai_key")
```

**Java**

```java
MiosaClient client = new MiosaClient(System.getenv("MIOSA_API_KEY"));
Sandbox sandbox = client.sandboxes().create(SandboxCreate.builder()
    .template("python-ds").build());

sandbox.secrets().set(SecretSet.builder()
    .name("openai_key")
    .value(System.getenv("MY_OPENAI_KEY"))
    .exposeAsEnv("OPENAI_API_KEY")
    .build());
sandbox.secrets().rotate("openai_key", "sk-proj-newvalue...");
List<SecretMeta> secrets = sandbox.secrets().list();
sandbox.secrets().delete("openai_key");
```

**Elixir**

```elixir
{:ok, sandbox} = Miosa.Sandboxes.create(%{template_id: "python-ds"})

{:ok, _} = Miosa.Secrets.set(sandbox.id, %{
  name: "openai_key",
  value: System.get_env("MY_OPENAI_KEY"),
  expose_as_env: "OPENAI_API_KEY"
})
{:ok, _} = Miosa.Secrets.rotate(sandbox.id, "openai_key", "sk-proj-newvalue...")
{:ok, secrets} = Miosa.Secrets.list(sandbox.id)
{:ok, _} = Miosa.Secrets.delete(sandbox.id, "openai_key")
```

## 实际示例 - 使用 OpenAI 和 Stripe 的多服务智能体

一个通过同时调用 OpenAI 和 Stripe API 来起草付款摘要的智能体。两个机密，一个沙箱，虚拟机内没有任何凭据。

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
sandbox = client.sandboxes.create(template_id="python-ds")

# Store both secrets
sandbox.secrets.set(
    name="openai_key",
    value=os.environ["MY_OPENAI_KEY"],
    expose_as_env="OPENAI_API_KEY",
)
sandbox.secrets.set(
    name="stripe_key",
    value=os.environ["MY_STRIPE_KEY"],
    expose_as_env="STRIPE_SECRET_KEY",
)

agent_code = '''
import os, requests, json

# Both env vars are placeholders; the proxy swaps them on egress
OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
STRIPE_SECRET_KEY = os.environ["STRIPE_SECRET_KEY"]

# Fetch recent Stripe charges
charges = requests.get(
    "https://api.stripe.com/v1/charges?limit=5",
    auth=(STRIPE_SECRET_KEY, ""),
).json()["data"]

# Ask OpenAI to summarize them
summary_prompt = "Summarize these Stripe charges in one sentence: " + json.dumps(charges)
resp = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
    json={"model": "gpt-4o-mini", "messages": [{"role": "user", "content": summary_prompt}]},
)
print(resp.json()["choices"][0]["message"]["content"])
'''

sandbox.files.write("/workspace/agent.py", agent_code)
result = sandbox.exec("python /workspace/agent.py")
print(result.stdout)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.create({ template: 'node-web' });

await sandbox.secrets.set({
  name: 'openai_key',
  value: process.env.MY_OPENAI_KEY!,
  exposeAsEnv: 'OPENAI_API_KEY',
});
await sandbox.secrets.set({
  name: 'stripe_key',
  value: process.env.MY_STRIPE_KEY!,
  exposeAsEnv: 'STRIPE_SECRET_KEY',
});

const agentCode = `
const resp = await fetch('https://api.stripe.com/v1/charges?limit=5', {
  headers: { Authorization: 'Bearer ' + process.env.STRIPE_SECRET_KEY },
});
const { data: charges } = await resp.json();

const ai = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer ' + process.env.OPENAI_API_KEY,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'gpt-4o-mini',
    messages: [{ role: 'user', content: 'Summarize: ' + JSON.stringify(charges) }],
  }),
});
const { choices } = await ai.json();
console.log(choices[0].message.content);
`;

await sandbox.files.write('/workspace/agent.ts', agentCode);
const result = await sandbox.exec('npx tsx /workspace/agent.ts');
console.log(result.stdout);
```

## 故障排查

**沙箱环境变量为空——`echo $OPENAI_API_KEY` 没有任何输出。**
该机密保存时使用了 `scope="tenant"` 或 `scope="workspace"`，但沙箱不属于该作用域。请改用 `scope="sandbox"` 重新保存（这是使用 `sandbox.secrets.set` 时的默认值）。

**`secrets.list()` 能显示该机密，但上游 API 返回 401。**
智能体没有在请求中看到占位符。确认所用库将令牌放在 `Authorization: Bearer ...` 中。检查审计记录中的 `request_transforms.secrets.swapped`——如果为空，说明智能体从未匹配到。

**我轮换了密钥，但没有任何变化。**
轮换是即时生效的。轮换后上游 API 调用成功，说明新密钥已经生效。要确认旧密钥确实已失效，请在提供商侧撤销它。

**我删除了机密，但沙箱内的环境变量仍有值。**
沙箱进程会在启动时缓存环境变量。请重启进程或打开新的 exec 会话——智能体会在每次请求时检查保管库，但进程内存中的环境变量仍反映其最后读取的值。

更多问题请参阅[安全故障排查中心](/docs/security/troubleshooting)。

## 常见问题

**保存后能通过 SDK 读取机密的真实值吗？**
不能。`secrets.list` 只返回元数据（名称、类型、预览、最后使用时间）。值处于密封状态。

**如果我想继续使用 `.env` 文件怎么办？**
继续使用即可。MIOSA Secrets 完全是增量功能，`.env` 文件仍然可用。对于希望集中管理的凭据，请使用 Secrets。

**这会让我的沙箱变慢吗？**
每次出站请求约增加 1-3 毫秒。与典型的 API 延迟相比可以忽略不计。

**它适用于 `curl`、`requests`、`fetch`、`axios`、`gRPC` 等吗？**
适用。智能体在网络层工作；语言运行时不会感知到替换。
