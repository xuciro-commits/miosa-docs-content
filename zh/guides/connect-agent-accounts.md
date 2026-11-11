---
slug: "guides/connect-agent-accounts"
title: "连接智能体帐户（密钥和订阅）"
description: "从您自己的代码将模型密钥、Claude 或 ChatGPT 计划或工具服务器密钥连接到您的工作区或您的最终用户之一。无需访问控制台。"
section: "Agents"
order: 6019
---

智能体运行时使用属于其所有者的帐户进行思考。
MIOSA 绝不会使用自己的模型密钥为您的智能体提供支持。
因此，在开始运行之前，必须连接模型提供者：API 密钥或订阅计划，例如 Claude 或 ChatGPT。

您可以在控制台中执行此操作。
您还可以通过自己的代码、为自己的工作区或每个最终用户执行此操作。
第二种情况是平台所需要的：您的客户将他们自己的克劳德计划连接到您的产品中，并且他们机器上的每次运行都使用它。

本指南涵盖了两者。

## 您可以连接什么

|连接|它是什么 |如何连接它 | SDK 中的名称 |
|---|---|---|---|
|克劳德计划| Claude Pro 或 Max 订阅。 |登录时，用户粘贴代码。 | `claude` |
| ChatGPT 计划 | ChatGPT Plus、Pro 或 Team 订阅 (Codex)。 |登录时，用户输入代码。 | `chatgpt` |
|基米计划| Kimi 代码订阅。 |登录时，用户输入代码。 | `kimi` |
|米斯特拉尔氛围计划|米斯特拉尔氛围订阅。 |在浏览器中登录。 | `mistral` |
|人择钥匙| Anthropic API 密钥。 |设置它。 | `anthropic` |
| OpenAI 密钥 | OpenAI API 密钥。 |设置它。 | `openai` |
| OpenRouter 密钥 | OpenRouter API 密钥。 |设置它。 | `openrouter` |
|其他按键 | DeepSeek、LLM Gateway、Moonshot、Mistral、Amazon Bedrock。 |设置它。 | `deepseek`、`llmgateway`、`moonshot`、`mistral`、`bedrock` |
|工具服务器密钥 |工具服务器需要的密钥：Apify、Refero、任何 MCP 服务器。 |设置它。 | `toolKeys` |

钥匙永远不会离开平台。
每次读取都会返回一个状态、一个经过编辑的预览（例如 `sk-ant...1234`）以及计划的帐户电子邮件和计划名称。
从来都不是秘密。

> **注意**
>
> 在供应商支持的情况下，一个连接计划可以为多个运行框架供电。
> 例如，ChatGPT 计划为 Codex 和 OpenCode 提供支持。
> 连接一次，兼容运行框架上的任何运行都会拾取它。
> 查看[哪个运行框架使用哪个连接](/docs/guides/connect-agent-accounts#which-harness-uses-which-connection)。

## 连接属于谁

每个调用都采用相同的两个可选标识符。

|你通过|该连接属于 |跑步用它|
|---|---|---|
|什么都没有|你的组织。 |在每个工作空间中，除非存在较窄的工作空间。 |
| `workspaceId` |一个工作区。 |对于在该工作区中运行。 |
| `externalUserId` |您的最终用户之一：您自己的 ID。 |用于在工作区之前在最终用户的计算机上运行。 |

运行按以下顺序查找帐户，并在找到的第一个帐户处停止：

1. 最终用户自己的连接。
2. 工作区的连接。
3. 组织的联系。

最终用户是运行所使用的机器上的用户。
创建机器时，您可以使用 `externalUserId` 标记机器，该 ID 与平台中其他任何地方使用的 ID 相同（[所有权和归属 ](/docs/platform/attribution)）。

写入和删除始终精确地触及您指定的范围。
为一个最终用户设置密钥永远不会更改工作区的密钥，并且删除它会回到工作区的密钥。
最终用户的连接对其他最终用户、工作区或其他组织永远不可见。

## 你自己的工作空间

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

// An API key
await miosa.connections.setKey({
  provider: "anthropic",
  key: process.env.ANTHROPIC_API_KEY!,
  workspaceId,
})

// Check that it works
const check = await miosa.connections.test({ provider: "anthropic", workspaceId })
console.log(check.ok, check.httpStatus) // true 200
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# An API key
miosa.connections.set_key(
    "anthropic",
    key=os.environ["ANTHROPIC_API_KEY"],
    workspace_id=workspace_id,
)

# Check that it works
check = miosa.connections.test("anthropic", workspace_id=workspace_id)
print(check["ok"], check["http_status"])  # True 200
```

**命令行界面**

```bash
# The key comes from standard input, never from an argument
miosa connections add models anthropic --workspace-id "$WORKSPACE_ID" --key-stdin <<< "$ANTHROPIC_API_KEY"

miosa connections test models anthropic --workspace-id "$WORKSPACE_ID"
```

再次设置密钥即可替换。
这就是你旋转一个的方式。

## 您的最终用户

通过 `externalUserId`，连接就属于他们了。

### API 密钥

**打字稿**

```typescript
await miosa.connections.setKey({
  provider: "openrouter",
  key: userSuppliedKey,
  externalUserId: "cus_123",
})
```

**Python**

```python
miosa.connections.set_key(
    "openrouter",
    key=user_supplied_key,
    external_user_id="cus_123",
)
```

**命令行界面**

```bash
miosa connections add models openrouter --external-user cus_123 --key-stdin <<< "$USER_KEY"
```

最终用户的密钥仅供智能体使用。
它永远不会提供给 AI 网关。

### Claude 计划或 ChatGPT 计划

连接计划是用户在自己的浏览器中完成的登录。
您的代码启动它，向用户显示一个链接，然后等待。
呼叫立即返回并显示要显示的内容。
还没有任何连接。

**打字稿**

```typescript
const signin = await miosa.connections.connectSubscription({
  provider: "claude", // or "chatgpt", "kimi", "mistral"
  externalUserId: "cus_123",
})

// Show these in YOUR UI.
signin.verificationUrl // open this in the user's browser
signin.userCode        // ChatGPT and Kimi: the code they type there
signin.mode            // "paste_code" (Claude) or "device_code"
signin.expiresAt       // about 15 minutes

// Claude: the page shows a code after they approve. Your UI passes it back.
const connected = await signin.submitCode(codeTheyPasted)
console.log(connected.accountEmail, connected.accountPlan) // alice@example.com max

// ChatGPT, Kimi: they enter the code at the link. Wait until it lands.
const connected2 = await signin.wait({ onUpdate: (s) => console.log(s.status) })
```

**Python**

```python
signin = miosa.connections.connect_subscription(
    "claude",  # or "chatgpt", "kimi", "mistral"
    external_user_id="cus_123",
)

# Show these in YOUR UI.
signin.verification_url  # open this in the user's browser
signin.user_code         # ChatGPT and Kimi: the code they type there
signin.mode              # "paste_code" (Claude) or "device_code"
signin.expires_at        # about 15 minutes

# Claude: the page shows a code after they approve. Your UI passes it back.
connected = signin.submit_code(code_they_pasted)
print(connected["account_email"], connected["account_plan"])  # alice@example.com max

# ChatGPT, Kimi: they enter the code at the link. Wait until it lands.
connected = signin.wait(on_update=lambda s: print(s["status"]))
```

**命令行界面**

```bash
# Prints the link and the code, then returns. Your UI shows them.
miosa connections add models claude --external-user cus_123 --no-wait --json

# Claude: hand over the pasted code.
miosa connections signin code SIGNIN_ID --external-user cus_123 --code-stdin <<< "$PASTED_CODE"

# ChatGPT, Kimi: wait for the user to enter the code.
miosa connections signin wait SIGNIN_ID --external-user cus_123
```

登录有两种形式：

|模式|计划|用户做什么 |你做什么|
|---|---|---|---|
| `paste_code` |克劳德|在链接处批准，然后复制页面显示的代码。 |将该代码传递给 `submitCode`。 |
| `device_code` | ChatGPT，基米 |打开链接并输入 `userCode`。 |在计时器上调用 `wait()` 或 `poll()`。 |

`wait()` 在连接计划时解析。
如果登录失败、过期或被取消，则会拒绝 `SubscriptionSignInError`。
`poll()` 永远不会因为登录失败而拒绝：读取其 `status`。
`cancel()` 结束登录。

错误或过期的粘贴代码将被 `SIGNIN_CODE_REJECTED` (422) 拒绝。
登录保持打开状态，以便用户可以再次粘贴。

登录链接和代码不包含任何凭据。
该计划的令牌在平台上加密存储，在后台刷新，并且永远不会由任何调用返回。

> **注意**
>
> 在您自己的 UI 中显示计划所有者的链接。
> 不要从您的服务器为他们打开它。
> 登录即表示用户同意连接其计划。

### 检查、列出和删除

**打字稿**

```typescript
// Is it connected, and where does it come from?
const claude = await miosa.connections.get({ provider: "claude", externalUserId: "cus_123" })
claude.connected    // true
claude.scope        // "external_user", or "workspace" when it is inherited
claude.inherited    // true when it comes from a wider scope
claude.preview      // "sk-ant...1234" for a key. Never the secret.

// Everything this end user has, with the workspace's inherited connections marked
const all = await miosa.connections.list({ externalUserId: "cus_123", workspaceId })

// Does it work?
await miosa.connections.test({ provider: "claude", externalUserId: "cus_123" })

// Disconnect only the end user's own. The workspace's, if any, is untouched.
await miosa.connections.delete({ provider: "claude", externalUserId: "cus_123" })
```

**Python**

```python
claude = miosa.connections.get("claude", external_user_id="cus_123")
claude["connected"]   # True
claude["scope"]       # "external_user", or "workspace" when it is inherited
claude["inherited"]   # True when it comes from a wider scope

everything = miosa.connections.list(external_user_id="cus_123", workspace_id=workspace_id)

miosa.connections.test("claude", external_user_id="cus_123")
miosa.connections.delete("claude", external_user_id="cus_123")
```

**命令行界面**

```bash
miosa connections list --external-user cus_123 --workspace-id "$WORKSPACE_ID"
miosa connections test models claude --external-user cus_123
miosa connections rm models claude --external-user cus_123 --yes
```

`test` 向提供商发出一项廉价的身份验证请求，例如列出型号。
对于计划，它会检查存储的状态。
它回答`ok: true`、`ok: false`（提供者拒绝了密钥）或`ok: null`（提供者没有回答，这不是密钥的错误）。
它永远不会返回密钥或提供者的响应。

## 工具-服务器密钥

工具服务器是您的智能体使用的 MCP 服务器：Apify、Refero、您自己的。
您可以定义服务器一次，并将其凭据作为占位符，例如 `{{secret:APIFY_TOKEN}}`。
然后每个最终用户提供自己的价值。
为该最终用户运行的智能体使用其密钥到达工具服务器，然后返回到工作区的服务器。

**打字稿**

```typescript
// Which keys do my tool servers need, and does this end user have them?
const keys = await miosa.connections.toolKeys.list({ workspaceId, externalUserId: "cus_123" })
// [{ name: "APIFY_TOKEN", connected: false, servers: [{ name: "Apify" }], ... }]

await miosa.connections.toolKeys.set({
  name: "APIFY_TOKEN",
  value: userApifyToken,
  workspaceId,
  externalUserId: "cus_123",
})

await miosa.connections.toolKeys.delete({ name: "APIFY_TOKEN", externalUserId: "cus_123" })
```

**Python**

```python
keys = miosa.connections.tool_keys.list(workspace_id=workspace_id, external_user_id="cus_123")

miosa.connections.tool_keys.set(
    "APIFY_TOKEN",
    user_apify_token,
    workspace_id=workspace_id,
    external_user_id="cus_123",
)
miosa.connections.tool_keys.delete("APIFY_TOKEN", external_user_id="cus_123")
```

**命令行界面**

```bash
miosa connections tool-keys list --external-user cus_123
miosa connections tool-keys set APIFY_TOKEN --external-user cus_123 --key-stdin <<< "$USER_TOKEN"
miosa connections tool-keys rm APIFY_TOKEN --external-user cus_123 --yes
```

只能设置工具服务器引用的名称。
其他都是 `UNKNOWN_TOOL_KEY` (404)，所以这不是通用的秘密存储。

## 哪个运行框架使用哪个连接

运行框架是在机器上运行的智能体：Claude Code、Codex、OpenCode 等。
每个都接受一组连接。

|运行框架（`runner`）|接受|
|---|---|
|克劳德代码 (`claude-code`) |克劳德计划、Anthropic 密钥、OpenRouter 密钥、DeepSeek 密钥、Amazon Bedrock |
|法典 (`codex`) | ChatGPT 计划、OpenAI 密钥、DeepSeek 密钥、Amazon Bedrock |
| OpenCode (`opencode`) | ChatGPT 计划、Anthropic 密钥、OpenAI 密钥、OpenRouter 密钥、LLM 网关密钥、Mistral 密钥、Mistral Vibe 计划 |
| Pi (`pi`) 和 Prime Agent (`prime`) | Claude 计划、ChatGPT 计划、Anthropic 密钥、OpenAI 密钥、OpenRouter 密钥、LLM 网关密钥、Mistral 密钥、Mistral Vibe 计划 |
|爱马仕特工 (`hermes`) | ChatGPT 计划、Claude 计划、Anthropic 密钥、OpenAI 密钥、OpenRouter 密钥 |
| Kimi 代码 (`kimi`) | Kimi计划，登月关键|
|米斯特拉尔氛围 (`mistral`) |米斯特拉尔氛围计划，米斯特拉尔钥匙 |
| OSA (`osa`) | Anthropic、OpenAI 或 OpenRouter 密钥，或您在 Agent Studio 中设置的模型密钥 |

有两个限制来自供应商，而不是 MIOSA：

- OpenCode 不采取克劳德计划。 Anthropic 禁止它，而 OpenCode 则删除了支持。
- Hermes 仅在购买额外使用量的 Claude Max 计划上使用 Claude 计划。 Pro 计划被 Anthropic 拒绝。

连接一次后，兼容运行框架上的任何运行都会建立连接。
您不必为每次运行选择一个连接。

要查看给定用户现在可以运行什么：

**打字稿**

```typescript
for (const harness of await miosa.harnesses.list({ externalUserId: "cus_123" })) {
  const usable = harness.connections.filter((c) => c.usable).map((c) => c.credential)
  console.log(harness.id, harness.ready ? "ready" : "no credential", usable)
}
// claude-code ready ["claude_subscription"]
// opencode    ready ["chatgpt_subscription"]
// pi          no credential []
```

**Python**

```python
for harness in miosa.harnesses.list(external_user_id="cus_123"):
    usable = [c["credential"] for c in harness["connections"] if c["usable"]]
    print(harness["id"], "ready" if harness["ready"] else "no credential", usable)
```

**命令行界面**

```bash
miosa connections harnesses --external-user cus_123
```

运行框架设置（哪种方法处于活动状态，默认模型）属于您（构建者）。
它们是按工作区设置的，并且对于所有最终用户来说都是相同的。
连接是他们的。
为最终用户创建的密钥无法更改运行框架设置。

## 选择运行框架使用的连接

一次采用一种方法的运行框架（Claude Code、Codex）使用一个连接。
需要多个（OpenCode、pi、Prime Agent、Kimi Code、Mistral Vibe、Hermes Agent）的工具使用您启用的那些。
这些是构建器的设置，按组织或每个工作区设置。

**打字稿**

```typescript
await miosa.agentSettings.updateHarness("claude-code", { authMethod: "claude_subscription", workspaceId })
await miosa.agentSettings.updateHarness("opencode", { enabledProviders: ["chatgpt_subscription", "openrouter"] })
```

**Python**

```python
miosa.agent_settings.update_harness("claude-code", auth_method="claude_subscription", workspace_id=workspace_id)
miosa.agent_settings.update_harness("opencode", enabled_providers=["chatgpt_subscription", "openrouter"])
```

**命令行界面**

```bash
miosa connections use claude-code claude                    # the Claude plan
miosa connections use opencode chatgpt openrouter           # several
miosa connections use opencode --all                        # every connected one
miosa connections use codex --model gpt-6.1-sol --effort high
```

然后使用您想要的模型在任何背带上运行：

```bash
miosa prompt --harness opencode --model gpt-6.1-sol --sandbox my-box "add a health check"
```

`miosa prompt --harness` 获取平台列出的每个工具，包括 CLI 发布后添加的工具。
`miosa connections harnesses opencode` 列出了该运行框架的型号以及可与所连接的设备配合使用的型号。

## 将其构建到您的产品中

设置页面中的“连接您的克劳德计划”按钮是对后端的两个请求。
登录 ID 通过浏览器传输，您的后端从其自己的会话中命名最终用户。

```typescript
// 1. Start. Send the browser only what it needs.
app.post("/api/connect/claude", async (req, res) => {
  const signin = await miosa.connections.connectSubscription({
    provider: "claude",
    externalUserId: req.user.id, // from YOUR session, never from the request body
  })
  res.json({ signinId: signin.id, url: signin.verificationUrl })
})

// 2. The user approves in a new tab and pastes the code into your page.
app.post("/api/connect/claude/code", async (req, res) => {
  const signin = miosa.connections.resumeSubscription({
    id: req.body.signinId,
    provider: "claude",
    externalUserId: req.user.id,
  })
  const status = await signin.submitCode(req.body.code)
  res.json({ connected: status.connected, email: status.accountEmail, plan: status.accountPlan })
})
```

`resumeSubscription` 通过 ID 选择登录。
因为它承载最终用户，所以属于其他人的登录 ID 是 `NOT_FOUND`，即使使用您的组织范围密钥也是如此。

带有可运行服务器的工作示例位于 SDK 存储库中的 `examples/connect-agent-accounts` 下。
它连接两个客户的 Claude 计划、ChatGPT 计划、密钥和工具服务器密钥，并显示一个客户的连接永远不会到达另一个客户的运行。

## 安全

- **仅限服务器。** 使用 `msk_*` 键从后端调用这些方法。
  切勿将该密钥放入浏览器中。
- **为任何靠近客户代码运行的内容创建最终用户密钥**。使用 `externalUserId` 创建的密钥始终充当该最终用户。
  它无法读取或更改工作区或组织的连接 (`403 SCOPE_FORBIDDEN`)，无法更改运行框架设置，也无法查看其他最终用户的登录。
- **任何调用都不会返回任何秘密**。密钥和计划令牌在静态时加密，绑定到它们所属的最终用户，并且仅在开始运行时解密。
- **审核。** 每次连接、替换、删除和使用都会使用最终用户的 ID 而不是值写入 [审核日志](/docs/security/audit-log)。
- **隔离。** 每次读取时都会根据您的组织检查连接。
  最终用户的 ID 对于其他组织来说没有任何意义。
- **范围。** 读取需要 `integrations:read`（加上连接列表的 `connections:read`）。
  更改需要 `integrations:write`（登录时需要 `connections:write`）。
- **计划是用户的。** 为最终用户连接的计划仅用于在该最终用户的计算机上运行。
  它绝不会用于为您的其他客户提供服务。
- **速率限制**适用于任何地方（[速率限制](/docs/rate-limits)）。

## 错误

|代码|状态 |意义|
|---|---|---|
| `SCOPE_FORBIDDEN` | 403 | 关键是为一个最终用户服务并要求更广泛的东西。|
| `MISSING_EXTERNAL_USER_ID` | 422 | `scope: "external_user"` 没有 `externalUserId`。|
| `CONFLICTING_SCOPE` | 422 | `scope` 与标识符不一致。|
| `INVALID_EXTERNAL_USER_ID` | 422 | id 长度超过 255 个字符或包含控制字符。|
| `USE_SIGNIN` | 422 | 无法制定计划；将其与 `connectSubscription` 连接。|
| `VALIDATION_FAILED` | 422 | 字段丢失或错误，例如空键。|
| `SIGNIN_CODE_REJECTED` | 422 | 粘贴的代码错误或已过期。再粘贴一下。|
| `SIGNIN_PROVIDER_ERROR` | 502 | 提供商的登录服务没有应答。重新开始。|
| `NOT_FOUND` | 404 | 在这个范围内没有任何联系。|
| `UNKNOWN_CREDENTIAL` | 404 | MIOSA 不知道任何凭证。|
| `UNKNOWN_TOOL_KEY` | 404 | 没有工具服务器引用该秘密名称。|
| `OWN_CREDENTIALS_REQUIRED` | 422 | 跑步开始时没有连接任何运行框架。|

## 底层的API

SDK 和 CLI 是薄包装器。
每条路线都接受 `workspace_id`、`external_user_id` 和可选的 `scope`。

|路线 |是否 |
|---|---|
| `GET /agents/credentials` |每个凭证及其在范围内的状态。 |
| `PUT /agents/credentials/{credential}` |设置或更换密钥。 |
| `DELETE /agents/credentials/{credential}` |在准确的范围内将其删除。 |
| `POST /agents/credentials/{credential}/test` |有效吗？ |
| `POST /agent-signins` |开始计划登录。返回链接和代码。 |
| `GET /agent-signins/{id}` |投票吧。 |
| `POST /agent-signins/{id}/code` |交出粘贴的代码。 |
| `DELETE /agent-signins/{id}` |取消它。 |
| `GET /agents/tool-keys` |您的工具服务器所需的密钥。 |
| `PUT /agents/tool-keys/{name}` |设置一个。 |
| `DELETE /agents/tool-keys/{name}` |删除一个。 |
| `GET /connections` |模型、应用程序和工具服务器位于一个列表中。 |
| `GET /agents/settings` |在范围内做好准备。 |

## 后续步骤

- [智能体驱动的构建](/docs/quickstart/agent-driven-build)：连接帐户后针对沙箱运行智能体。
- [所有权和归属](/docs/platform/attribution)：用您自己的客户 ID 标记每台机器。
- [产品中的智能体](/docs/platform/agents-in-your-product)：每个用户一台机器，有一个智能体在其中工作。
- [智能体操作指南](/docs/guides/agent-operating-guide)：调度工作并收集结果。
