---
slug: "services/variables"
title: "服务变量"
description: "设置服务的变量、引用另一服务的变量、在同一环境中共享变量，并生成密钥。"
section: "Services & projects"
order: 1009
---

变量以密封形式静态存储。
读取只返回遮盖后的预览，绝不返回值本身。
唯一的明文读取方式是 `reveal`，它需要 `env:reveal` 和操作者自己的凭证，并且对密封变量会报错。

## 设置变量

```bash
miosa service vars set shop API NODE_ENV=production
miosa service vars list shop API
```

TypeScript：

```typescript
await miosa.services.setVariables(serviceId, { NODE_ENV: "production" });
const variables = await miosa.services.variables(serviceId);
```

Python：

```python
miosa.services.set_variables(service_id, {"NODE_ENV": "production"})
variables = miosa.services.variables(service_id)
```

写入会按名称合并。删除时提供名称：

```bash
miosa service vars unset shop API NODE_ENV,OLD_FLAG
```

## 引用

值可以是一个引用。
MIOSA 会在发布启动时解析它，因此服务始终能看到当前值。

| 表达式 | 解析为 |
|---|---|
| `${{Postgres.DATABASE_URL}}` | 同一环境中另一服务的变量 |
| `${{shared.API_BASE}}` | 环境共享变量 |
| `${{NAME}}` | 同一服务的另一个变量 |
| `${{secret(32)}}` | 生成的 32 字符随机密钥 |

```bash
miosa service vars set shop API DATABASE_URL='${{Postgres.DATABASE_URL}}'
```

在 shell 中为值加引号，让 `${{...}}` 原样传给 MIOSA。

如果存在循环或格式错误的表达式，系统会一次性列出所有问题并报错，且不会写入任何内容。
对尚不存在的服务或变量的引用可以保存，但会阻止部署，并显示在 `problems` 下。

## 共享变量

共享变量属于环境，并对该环境中的每个服务可见。

```bash
miosa project vars set shop API_BASE=https://api.acme.dev
miosa project vars list shop
miosa project vars unset shop API_BASE
```

服务使用 `${{shared.API_BASE}}` 读取共享变量。

## 更改何时需要发布

写入变量不会改变正在运行的发布。
MIOSA 会把这次写入导致服务文档过期的内容记录为 `pending_changes`：

```json
{"needs": "rebuild", "variables": ["DATABASE_URL"], "because": "Web.DATABASE_URL", "at": "..."}
```

- 当构建可见变量发生变化时为 `needs: rebuild`，因为仅重新部署会让构建时已内联的值继续随发布上线。
- 其他所有变量为 `needs: redeploy`，它们在发布启动时读取。
- `because` 为 `own`、`shared`，或传递该更改的引用：更改 `Web.DATABASE_URL` 会标记所有在其表达式中使用该变量的服务，更改共享变量会标记所有读取该共享变量的服务。

```bash
miosa service get shop API
```

部署会清除标记，后续的运行时更改不会将重建降级：

```bash
miosa service deploy shop API
```

## 检查部署将收到什么

```bash
miosa service vars effective shop API
```

这会显示发布将收到的每个变量，引用已解析而值仍被遮盖，并附上 `problems` 列表。
只要该列表不为空，发布就会被阻止。

## 读取单个值

```bash
miosa service vars reveal shop API DATABASE_URL
```

Reveal 用于以本人身份操作的人员。
密封变量不会被揭示；每次揭示都会记录审计，包含变量名称，绝不包含其值。

## 构建时变量

变量可以标记为构建可见，这样构建过程也能看到它。
平台提供的值绝不会发送到构建。
以 `MIOSA_` 或 `RAILPACK_` 开头的名称为保留名称。
