---
slug: "docker-host/environments"
title: "环境"
description: "生产、暂存、预览/<source>：部署的变量和秘密范围。"
section: "Services & projects"
order: 1030
---

**部署环境**是环境变量、机密和数据服务绑定应用的范围。大多数项目至少有两个：`production` 和 `staging`。 `preview/feature-abc` 等自定义环境对于审核应用程序工作流程很常见。

> **注：阶段状态**
>
> 一流的 `deployment_environments` 记录是正在进行的部署重构的一部分。如今，环境变量通过 `deployment_env_vars` 确定每个部署的范围。这里的形状和行为是稳态目标。

## 默认值

每个项目在创建时都会有两个环境：

- `production`：真实用户看到的内容
- `staging`：升级到生产之前的内部测试目标

您可以根据需要创建其他环境（`preview/&lt;source&gt;`、`qa`、`canary` 等）。

## 哪些环境范围

|项目 |范围 |
|---|---|
|环境变量 |每个环境 |
|秘密|每个环境 |
|数据服务绑定（DATABASE_URL、REDIS_URL 等）|每个环境 |
|域名 |每个环境（生产获得顶点；按惯例，登台获得 `staging.*` 子域） |
|版本推广|每个环境，首先将版本提升到暂存阶段，然后再提升到生产环境 |

相同的部署可以在不同的环境中运行不同的版本。常见模式：
```text

production:   version 17 active
staging:      version 18 active   ← being tested
preview/pr-42: version 19 active  ← single-feature preview
```

## 环境变量
```http
POST /api/v1/deployments/:id/env
GET  /api/v1/deployments/:id/env
```

环境变量在静态时加密存储（ChaCha20-Poly1305），并在开发期间注入沙箱中，并在启动时注入运行时实例中。默认情况下，列表端点返回值的预览，而不是原始值。

**打字稿**

```typescript
await miosa.deployments.env.set(deploymentId, {
  environment: "production",
  vars: {
    STRIPE_KEY: "sk_live_...",
    NEXT_PUBLIC_API_URL: "https://api.example.net",
  },
})

const preview = await miosa.deployments.env.list(deploymentId, {
  environment: "production",
})
// preview returns: { STRIPE_KEY: "sk_live_••••", NEXT_PUBLIC_API_URL: "https://api…" }
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/deployments/$DEP/env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{
    "environment": "production",
    "vars": { "STRIPE_KEY": "sk_live_..." }
  }'
```

## 秘密与环境变量

如今，没有严格的 UI 区别：环境变量无论如何都会在静态时加密。约定是：

- **环境变量**：可以作为预览检索的配置（截断值）。
- **秘密**：相同的加密，但标记为 `is_secret: true`，因此预览始终显示 `••••`，甚至对您自己的管理员也是如此。

未来状态，秘密将支持轮换和访问事件审核日志记录。

## 数据服务绑定

当您通过[Data](/docs/data/overview)创建Postgres/Redis/bucket时，您将其绑定到特定的部署环境。 MIOSA 在启动时将凭证注入到 env 中：
```text

production env:    DATABASE_URL=postgres://prod...
staging env:       DATABASE_URL=postgres://staging...
```

您不必为一个应用程序提供两个数据库 URL；而是为一个应用程序提供两个数据库 URL。您发送一个连接字符串变量，MIOSA 根据运行时启动的环境选择正确的值。

## 下一步去哪里

- [发布](/docs/docker-host/publishing)：在发布时选择环境
- [Domains](/docs/docker-host/domains)：附加暂存子域
- [数据](/docs/data/overview)：环境范围的凭据
