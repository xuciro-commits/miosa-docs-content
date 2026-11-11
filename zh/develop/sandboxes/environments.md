---
slug: "develop/sandboxes/environments"
title: "环境变量"
description: "在创建时设置沙箱级环境，按范围和目标继承帐户级运行时环境，并了解优先级、每个沙箱环境 API、秘密文件和秘密，以及值如何到达执行。"
section: "Sandboxes"
order: 2017
---

环境变量从几层之一到达沙箱：按范围解析的帐户级运行时环境、沙箱本身设置的值、绑定到不透明占位符的机密或在单个执行中传递的 `env`。本页介绍了每一层的含义、冲突如何解决以及如何阅读现场设置。

## 当应用值时

- 创建时，在第一个用户命令运行之前应用变量和机密。
- 每次恢复时，沙箱都会重新注入当前设置，因此恢复的沙箱会看到创建后设置的值，而不仅仅是启动时使用的值。
- 秘密值仅到达沙箱。 API、CLI 和 SDK 返回名称、元数据和简短预览，而不是明文值。
- 可以跨组织共享或分叉的快照首先会清除智能体密钥和秘密值。

> **注意：安装脚本**
>
> 创建时传递的安装脚本在沙箱准备就绪后在后台运行一次。
> 它不会延迟 `ready`。

## 账户级运行时环境

平台所有者定义一次环境变量，并让沙箱、计算机和智能体运行继承它们。运行时 env 静态加密，并且永远不会由 API、CLI 或 SDK 以明文形式返回。响应仅包括元数据和简短预览。

MIOSA 按此顺序解析运行时环境。更具体的值会覆盖具有相同名称的更广泛的值：

|优先级 |范围 |适用于 |
|---:|---|---|
| 1 |租户|租户下的每个工作区、项目、沙箱、计算机和智能体 |
| 2 |工作空间 |该工作区中的每个项目、沙箱、计算机和智能体 |
| 3 |项目|归因于该项目的沙箱、计算机、部署和智能体运行 |
| 4 |资源 |直接在资源上设置每沙箱或每计算机环境 |
| 5 |运行| Env 直接传递给单个 exec 或智能体运行 |

每个运行时环境变量也有一个 **目标**。在相同范围内，特定于目标的值会覆盖一组 `all`：

|目标|当 | 时使用它
|---|---|
| `all` |该值对于每个运行时表面都是安全且有用的 |
| `sandbox` |该值应该只出现在沙箱 exec/build/agent 会话中 |
| `computer` |该值应该只出现在桌面计算机命令和智能体中 |
| `agent` |该值应注入到在沙箱或计算机上运行的提示驱动智能体中 |
| `deployment` |生产部署/运行时启动 |

变量名称必须是与 `[A-Z][A-Z0-9_]*` 匹配的大写 shell 安全名称。

> **注意：运行时环境是智能体配置堆栈的一层**
>
> 运行时环境按范围和目标携带继承的环境变量和提供者密钥。 [智能体运行时配置文件](/docs/api-reference/agent-runtime-profiles)拥有运行时选择、工具、策略和非秘密默认值。 [连接器](/docs/security/connect-accounts)智能体提供商身份。使用运行时环境作为直接提供程序环境变量、MIOSA 应智能体访问的连接器以及配置文件以附加正确的默认值。

## 沙盒级环境

创建时传递的环境变量会在启动时注入到虚拟机中，并对所有进程可见。一旦沙箱运行，它们就是只读的； `env.list()` 返回现场设置。

**Python**

```python
# Set at create time
sbx = miosa.sandboxes.create(
    env={"DATABASE_URL": "postgres://...", "API_SECRET": "..."},
)

# Read the live env inside a running sandbox
env = sbx.env.list()
print(env)  # {"DATABASE_URL": "postgres://...", ...}
```

**打字稿**

```typescript
const sbx = await miosa.sandboxes.create({
  env: { DATABASE_URL: "postgres://...", API_SECRET: "..." },
});

const env = await sbx.env.list();
```

**curl**

```bash
# Pass at create time (in the POST body)

# Read
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/env \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## 每个沙箱环境 API

`/api/v1/sandboxes/{id}/env` API 管理附加到沙箱的键值存储。变量在静态时进行加密，并在启动时注入沙箱进程环境中。 API 从不返回纯文本值，仅返回键名称和元数据。

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/sandboxes/{id}/env` |列出所有环境变量（仅名称，无值）|
| `PUT` | `/api/v1/sandboxes/{id}/env` |批量设置或更新环境变量 |
| `DELETE` | `/api/v1/sandboxes/{id}/env/{key}` |删除单个环境变量 |

`PUT` 是批量更新插入：更新现有密钥，创建新密钥，并且请求中不存在的密钥保持不变。

```bash
curl -X PUT https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vars": [
      {"key": "DATABASE_URL", "value": "postgres://user:pass@host/db"},
      {"key": "LOG_LEVEL", "value": "info"}
    ]
  }'
```

> **注意：API 参考具有完整的合同**
>
> 请参阅 [Sandbox Env API](/docs/api-reference/sandbox-env) 了解范围、响应形状和错误表，以及 [Runtime Env](/docs/api-reference/runtime-env) 了解继承环境端点。

## 秘密对于您根本不希望在虚拟机内部存在的凭据，请将它们存储为机密。每个使用秘密的沙箱都会看到一个不透明的占位符环境变量；真正的密钥永远不会存在于虚拟机内部，并且交换发生在出口处。

```python
sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

在沙箱内，`os.environ["OPENAI_API_KEY"]` 返回一个不透明标记，例如 `miosa-tok-7f2a...`。当您的代码进行出站调用时，VM 主机上的 MIOSA 智能体会看到 `Authorization` 标头中的占位符，并在转发之前将其与真实密钥交换。轮换是即时的：每个沙箱都会在下一个出站请求中获取新值，并且占位符令牌不会更改。

机密的范围可以是 `tenant`、`workspace`、`user`、`external_user` 或 `external_workspace`，并且智能体会为发出请求的资源选择最具体的匹配。

> **警告：秘密应该存在的地方**
>
> 不要将长期存在的秘密存储在沙箱 `env` 值中：这些秘密存储在沙箱记录中，并且对虚拟机内的任何进程都可见。每次执行时传递它们、使用秘密存储或使用连接器。请参阅 [Secrets](/docs/security/secrets) 了解完整的工作流程。

## 一个值如何到达 exec

当命令运行时，MIOSA会分层解析有效环境并将其交给进程：

1. 沙箱范围和目标的继承运行时环境在启动时具体化。
2. 沙箱级 env（创建时 `env` 或每沙箱 env API）会覆盖具有相同名称的继承值。
3. 秘密显示为通过 `expose_as_env` 公开的占位符环境变量。
4. 传递给单个 exec 的 `env` 仅在该命令的顶部合并。

```python
result = sbx.exec.run(
    "pytest tests/ -x",
    {"cwd": "/workspace", "env": {"PYTHONPATH": "/workspace"}, "timeout_sec": 120},
)
```

> **注意：相关**
>
> [Create and connect](/docs/develop/sandboxes/create) 显示在创建时传递 `env` 的位置，[Connections (OAuth)](/docs/security/connect-accounts) 涵盖智能体提供者凭据。
