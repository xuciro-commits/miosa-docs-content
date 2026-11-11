---
slug: "migrating-from-miosa-sandbox"
title: "从 miosa-sandbox 迁移"
description: "将已弃用的 @miosa/sandbox、miosa-sandbox 和 sandboxes-go 包移至规范的 MIOSA SDK 上。"
section: "Optional"
order: 17030
---

独立沙箱包是已弃用的兼容性包装器。现在，每个包都委托给规范的 MIOSA SDK，因此最快的迁移通常是安装规范包并更改一个导入。

|已弃用的包 |规范替代 |
|---|---|
| `@miosa/sandbox` (JavaScript) | `@miosa/sdk` |
| `miosa-sandbox` (Python) | `miosa` |
| `sandboxes-go`（转到）| `github.com/Miosa-osa/miosa-go/v2` |

## JavaScript

```bash
npm install @miosa/sdk
```

```typescript
// Before
import { MIOSA } from '@miosa/sandbox';
const client = new MIOSA({ apiKey: process.env.MIOSA_API_KEY! });

// After
import { Miosa } from '@miosa/sdk';
const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
```

`@miosa/sandbox` 重新导出规范客户端，因此在迁移时它会继续安装，但新代码应直接导入 `@miosa/sdk`。重命名的类映射为：

|旧进口|规范替代 |
|---|---|
| `MIOSA` | `Miosa` |
| `SandboxClient` | `Sandboxes` |
| `ComputerClient` | `Computers` |
| `CreateSandboxOptions` | `SandboxCreateParams` |
| `CreateComputerOptions` | `ComputerCreateParams` |

## Python

```bash
pip install miosa
```

```python
# Before
from miosa_sandbox import MIOSA
with MIOSA(api_key="msk_...") as client:
    with client.sandboxes.create() as sandbox:
        print(sandbox.exec("python3 -c 'print(1 + 1)'").stdout)

# After
from miosa import Miosa
with Miosa(api_key="msk_...") as client:
    with client.sandboxes.create() as sandbox:
        print(sandbox.exec("python3 -c 'print(1 + 1)'").stdout)
```

`MIOSA` 和 `AsyncMIOSA` 保存在兼容包中，并返回规范的 `miosa` 资源类；异步类变为 `AsyncMiosa`。

## Go

```bash
go get github.com/Miosa-osa/miosa-go/v2
```

```go
// Before
import sandboxes "github.com/miosa-ai/sandboxes-go"
client, _ := sandboxes.New(sandboxes.Config{APIKey: os.Getenv("MIOSA_API_KEY")})

// After
import miosa "github.com/Miosa-osa/miosa-go/v2"
client := miosa.NewClient("msk_...")
```

> **注：日落窗**
>
> `sandboxes-go` 软件包修复了错误，但没有新功能，日落日期为 2026 年 11 月 8 日。 JavaScript 和 Python 兼容性包的存在只是为了让旧的集成在迁移时继续安装。

## 预期的行为差异

- **一个客户端，一个表面。** 在规范的 SDK 中，沙箱与计算机、文件、快照和智能体运行一起存在于客户端 (`client.sandboxes`) 上。到达单独沙箱客户端的代码现在将通过单个客户端。
- **所有权选择器是一流的。** `workspace_id`、`project_id` 和 `external_user_id` 可以直接在创建时传递，并在返回的句柄上公开。
- **`size` 和 `resource_contract`。** 当未提供大小时，兼容性包请求规范的 `small` 合同：2 个 vCPU、4096 MiB 内存和 10240 MiB 磁盘。该响应在同步和异步句柄上公开 `size`、`resource_contract` 和 `usage()`。
- **规范名称。** 在沙盒映像上使用 `client.sandboxes`，将 `client.computers` 与 `miosa-desktop` 一起使用，并在 Docker 主机设备上使用 `client.dockerDeploy`。 `computer` 是面向用户的产品名称； `miosa-desktop` 是它启动的模板。

## 验证迁移

兼容性包和规范 SDK 使用相同的 API，因此检查是行为性的：

```python
from miosa import Miosa

with Miosa(api_key="msk_...") as client:
    sbx = client.sandboxes.create(template_id="miosa-sandbox", size="small")
    print(sbx.state, sbx.resource_contract)
    print(sbx.exec("python3 -c 'print(1 + 1)'").stdout)   # 2
    sbx.destroy()
```

如果打印 `2` 和 `small` 合同，则迁移已完成。

## 另请参阅

- [TypeScript SDK](/docs/sdks/typescript)：规范的 JavaScript 和 TypeScript 客户端。
- [Python SDK](/docs/sdks/python)：规范的 Python 客户端，同步和异步。
- [Go SDK](/docs/sdks/go)：规范的 Go 客户端，版本 2。
