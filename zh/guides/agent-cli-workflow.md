---
slug: "guides/agent-cli-workflow"
title: "智能体 CLI 工作流程"
description: "智能体的完整 CLI 发布工作流程：发现、范围、沙箱开发、预览、发布、升级、恢复和回滚。"
section: "Agents"
order: 6002
---

当 AI 智能体、编码助理、CI 工作人员或操作员需要通过 MIOSA 的完整构建和发布路径时，请使用此页面。
CLI 是权威的快乐之路。与重建较低级别的 API 调用相比，更喜欢其 `--json` 输出。

## 从发现开始

```bash
miosa catalog --json
miosa deploy list --json
miosa deploy get <deployment> --json
```

`catalog` 报告平台目前提供的规范计算产品、模板、大小和准备情况。

`deploy list` 和 `deploy get` 返回您已拥有的部署及其状态、活动版本和公共 URL。

当这些命令可以返回当前清单时，请勿从旧记录中猜测资源 ID。

## 突变前解析范围

每个突变都必须解决一个明确的组织、工作空间、应用程序和环境。

不要仅仅因为标识符未解析而创建替换沙箱、部署、数据库或 Docker Host 主机。

首先检查目标资源。

```bash
miosa deploy get <deployment> --json
miosa deploy releases <deployment> --json
```

`deploy get` 显示部署的身份、环境、活动发布和路线。

`deploy releases` 列出了已经发布到它的不可变版本。

不要将凭据或机密提交到存储库。

秘密属于加密的 MIOSA 环境存储，而不是清单或 shell 历史记录。

## 在持久沙箱中开发

使用沙箱开发流程进行可变工作。

```bash
miosa create my-dev-box --template nextjs --wait
miosa console my-dev-box --cwd /workspace
miosa doctor
```

`create` 提供沙箱，并使用 `--wait` 进行阻止，直到准备好执行命令。

`console` 在沙箱上打开一个交互式 shell，并在稍后运行时恢复同一台计算机。

`doctor` 检查配置、连接和身份验证，并报告发现的内容。

依赖项安装必须使用严格锁定文件的命令，例如 `npm ci`。

## 制作前预览

```bash
miosa preview create my-dev-box 3000 --name candidate
miosa preview status my-dev-box <preview-id>
```

`preview create` 在沙箱上的公共 URL 上发布端口并打印该 URL。

`preview status` 报告是否正在服务。

预览永远不会促进生产。

从 JSON 响应中读取返回的 `url` 和 `status`。

使用返回的 URL。

不要构建沙箱、部署或自定义域主机名。

## 公布候选人

```bash
miosa deploy create my-app --from-sandbox my-dev-box --port 3000 --wait --json
```

`deploy create` 将沙箱、git 存储库或本地目录发布为新部署，并使用 `--wait` 进行阻止，直到其上线并打印其 URL。

每次发布都会生成一个不可变的版本，可以升级或回滚。

候选者不仅仅因为其构建成功而成为生产。

## 推动确切的发布

```bash
miosa deploy releases <deployment>
miosa deploy promote <deployment> <release>
```

促销的目标是一个确切的不可变版本。

它不得按时间戳选择“最新”或依赖可变版本标签。

当活动的不可变版本发生变化时，生产 URL 保持稳定。

如果候选版本无法构建，则当前活动版本仍然有效。

## 恢复中断的发布

HTTP 超时并不能证明后端操作失败。

重试之前重新读取部署，并且仅在前一次尝试被证明是终端时才重新发布。

```bash
miosa deploy get <deployment> --json
miosa deploy builds <deployment>
```

当较早的操作可能仍处于活动状态时，请勿启动重复的发布或升级。

## 故意回滚

```bash
miosa deploy rollback <deployment> --to <release-id> --yes --json
```

回滚还针对一个确切的不可变版本。

使用已知良好的版本并在激活后验证相同的路由、工件、数据和配置合同。

当较旧的应用程序版本可能针对较新的数据运行时，数据库迁移必须保持向后兼容。

## 另请参阅

- [特工安全和状态](/docs/guides/agent-safety)：解释复合健康信号并保护持久资源。
- [坐席操作指南](/docs/guides/agent-operating-guide)：坐席三步进入路径。
- [发布](/docs/docker-host/publishing)：候选人创建、验证、精确晋升和收据。
- [回滚](/docs/docker-host/rollback)：将部署重新指向旧版本，无需重建。
