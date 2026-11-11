---
slug: "guides/agent-operating-guide"
title: "智能体操作指南"
description: "使用 MIOSA 的 AI 智能体的 CLI 和发布工作流程。"
section: "Agents"
order: 6000
---

当 AI 智能体、编码助理、CI 工作人员或操作员需要通过 MIOSA 构建和发布软件时，请使用此页面。
CLI 是权威的快乐路径 - 更喜欢它的 `--json` 输出而不是重建较低级别的 API 调用。

## 1. 发现当前库存

```bash
miosa catalog --json
miosa deploy list --json
miosa deploy get <deployment> --json
```

`catalog` 报告平台目前提供的规范计算产品、模板、大小和准备情况。
`deploy list` 和 `deploy get` 返回您已拥有的部署。
当这些命令可以返回当前清单时，请勿从旧记录中猜测资源 ID。

## 2. 在持久沙箱中开发

```bash
miosa create my-dev-box --template nextjs --wait
miosa console my-dev-box --cwd /workspace
miosa doctor
miosa preview create my-dev-box 3000 --name candidate
```

`create` 提供沙箱，并使用 `--wait` 进行阻止，直到准备就绪。
`console` 打开交互式 shell，并在稍后运行时恢复同一台计算机。
`doctor` 检查配置、连接和身份验证。
预览版在公共 URL 上发布端口，并且从不推广生产。

## 3. 发布确切的版本

**命令行界面**

```bash
miosa deploy create my-app --from-sandbox my-dev-box --port 3000 --wait --json
miosa deploy releases <deployment>
miosa deploy promote <deployment> <release>
```

**打字稿**

```typescript
const result = await miosa.deployments.publish(deploymentId, {
  sourceSandboxId: sandbox.id,
  kind: "auto",                       // "static" | "dynamic" | "auto"
  environment: "production",
  outputPath: "/workspace",
  runCommand: "npm start",            // dynamic deployments only
  port: 3000,                         // dynamic only
})

console.log(result.deployment.public_url)
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
```

候选者不仅仅因为其构建成功而成为生产。
促销的目标是一个确切的不可变版本，并且在活动版本发生变化时，生产 URL 保持稳定。

## 选择你需要的

|你想要|从这里开始 |
|---|---|
|完整的 CLI 工作流程 | [智能体 CLI 工作流程](/docs/guides/agent-cli-workflow) |
|解读状态并保护持久资源 | [特工安全和状态](/docs/guides/agent-safety) |
|智能体可以做什么 | [MIOSA 智能体可以做什么](/docs/guides/agent-capabilities) |
|配置智能体| [配置智能体](/docs/guides/configure-agent) |
|了解部署对象 | [部署模型](/docs/docker-host/deployment-model) |

## 下一步

- [智能体 CLI 工作流程](/docs/guides/agent-cli-workflow)：发现、范围、沙箱开发、预览、发布、升级、恢复和回滚。
- [智能体安全和状态](/docs/guides/agent-safety)：复合健康信号、持久资源和能力边界。
- [发布](/docs/docker-host/publishing)：了解候选人创建、验证、准确晋升和收据。
- [回滚](/docs/docker-host/rollback)：将部署重新指向旧版本，无需重建。
- [可用性](/docs/availability)：检查功能是否可用、集成、可自托管、门控或计划。
- [你可以构建什么](/docs/use-cases)：探索智能体产品、创意系统、垂直平台和基础设施产品。
- [应用程序清单](/docs/develop/templates)：定义沙箱、服务、依赖项、准备情况、机密和数据要求。
