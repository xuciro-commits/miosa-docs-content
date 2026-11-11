---
slug: "docker-host/overview"
title: "Docker主机"
description: "您自己的永远在线主机，运行从沙箱发布的应用程序，具有不可变版本、运行状况检查和回滚。以前是应用程序引擎。"
section: "Services & projects"
order: 1020
---

Docker Host（以前称为 App Engine）是您自己的一个盒子，用于运行您从沙箱发布的应用程序。
要运行项目的 Web 服务、工作人员、计划和静态站点，请改用 [Services](/docs/services/overview)。

## Docker 主机还是服务？

|你想要|使用 |
|---|---|
|项目中的 Web 服务、工作人员、计划或静态站点，具有版本、变量、域和回滚 | [服务](/docs/services/overview) |
|从沙箱发布到您运行的主机上的应用程序，具有不可变的版本 | Docker 主机（本节）|

这里没有任何内容被删除。
以下页面描述了 Docker 主机部署模型。

在 MIOSA 上部署意味着将沙箱转变为稳定的生产应用程序、站点或 API，由不可变版本、运行状况检查和 API 返回的公共 URL 提供支持。
发布后，部署 URL 保持稳定。
新发布更改它指向的版本。

## 1. 从沙箱发布

**命令行界面**

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --type docker_deploy \
  --wait \
  --json
```

每次发布时使用相同的 `--name`，以便 MIOSA 在相同的部署 ID 和 URL 后面创建一个新版本。

**打字稿**

```typescript
const result = await miosa.deployments.publish(deploymentId, {
  sourceSandboxId: sandbox.id,
  kind: "auto",                       // "static" | "dynamic" | "auto"
  environment: "production",          // "production" | "staging" | etc.
  outputPath: "/workspace",
  runCommand: "npm start",            // dynamic deployments only
  port: 3000,                         // dynamic only
})

console.log(result.deployment.public_url)
console.log(result.version.kind, result.version.state)
```

**Python**

```python
result = miosa.deployments.publish(
    deployment_id,
    source_sandbox_id=sandbox_id,
    kind="auto",              # "static" | "dynamic" | "auto"
    environment="production",
    output_path="/workspace",
    run_command="npm start",         # dynamic only
    port=3000,                       # dynamic only
)

print(result.deployment.public_url)
print(result.version.kind, result.version.state)
```

## 2. 升级或回滚

```bash
miosa deploy releases <deployment>
miosa deploy promote <deployment> <release>
miosa deploy rollback <deployment> --to <release-id> --yes --json
```

升级和回滚每个目标都是一个确切的不可变版本。

## 3. 证明它是实时的

```bash
miosa deploy get <deployment> --json
miosa deploy requests <deployment>
```

部署对象本身并不能证明。
公共 URL 必须路由到活动版本，对于 Docker Host 还必须路由到工作区主机和正在运行的容器。

## 选择你需要的

|你想要|从这里开始 |
|---|---|
|提供静态站点、SPA 或文档 | [发布](/docs/docker-host/publishing) |
|运行 API、SSR 或 WebSocket 服务器 | [动态运行时](/docs/docker-host/runtime-instances) |
|在一台主机上运行许多小应用程序 | [Docker 主机](/docs/docker-host/docker-deploy) |
|回滚到旧版本 | [回滚](/docs/docker-host/rollback) |
|附加自定义域 | [部署域](/docs/docker-host/domains) |
|了解发布背后的对象 | [部署模型](/docs/docker-host/deployment-model) |

## 下一步

- [发布](/docs/docker-host/publishing)：沙箱到版本的管道：源快照、构建模式和升级。
- [版本](/docs/docker-host/versions)：不可变的历史记录。列出、检查、升级和存档版本。
- [发布](/docs/docker-host/releases)：工件：静态文件和动态服务器工件。
- [回滚](/docs/docker-host/rollback)：在几秒钟内将部署重新指向旧版本，无需重建。
- [域](/docs/docker-host/domains)：托管域和自定义域。 DNS 验证、TLS 和路由缓存。
- [动态运行时](/docs/docker-host/runtime-instances)：动态部署 - 生产运行时、运行状况检查和协调器行为。
- [部署模型](/docs/docker-host/deployment-model)：管道、静态与动态、部署、版本和证明合约。
