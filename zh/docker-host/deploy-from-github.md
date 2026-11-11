---
slug: "docker-host/deploy-from-github"
title: "从 GitHub 部署"
description: "使用 miosa deploy create --git 构建并发布 GitHub 存储库 - 连接 GitHub、公共和私有存储库、分支、日志、重新部署以及每次推送时重建的 Webhook。"
section: "Services & projects"
order: 1026
---

`miosa deploy create --git` 克隆存储库，构建您命名的分支，并将其发布到稳定的 URL。
将其 Webhook 添加到存储库后，在推送时会重建相同的部署。

## 连接 GitHub

公共存储库不需要连接。
私有存储库是通过创建部署的人的 GitHub 连接进行克隆的，因此首先连接 GitHub：

```bash
miosa connections add apps github
```

该命令将打开 GitHub 的同意页面并等待您批准。
在控制台中，相同的连接位于 **Connections**、**Apps** 下。

GitHub 仅向 MIOSA 显示批准它的组织。
如果组织的存储库丢失，请在 GitHub 上的**设置**、**应用程序**、**授权 OAuth 应用程序**下授予访问权限，或要求组织所有者批准 MIOSA。

## 部署存储库

```bash
miosa deploy create --git https://github.com/octocat/Spoon-Knife --branch main
```

该命令会等待部署生效并打印其 URL，例如 `https://spoon-knife.<your-org>.miosa.app`。
通过`--name`选择部署名称；它默认为存储库名称。
创建部署后，传递 `--no-wait` 返回。

用 `--branch` 命名分支。
部署现在和每次推送时都会构建该分支。

连接 GitHub 后，私有存储库的工作方式相同。
如果没有连接，或者分支不存在，构建将在克隆步骤失败：

```text
publishing failed at build: The source could not be cloned (git exited 128). (SOURCE_CLONE_FAILED)
```

连接GitHub，检查分支名称，然后运行`miosa deploy redeploy <name>`。

## 日志和构建

```bash
miosa deploy builds my-app     # every build and its state
miosa deploy logs my-app       # why the last build failed, and the app's output
```

静态站点没有运行时日志：它提供构建的文件并且不运行任何进程。

## 重新部署

```bash
miosa deploy redeploy my-app
```

Redeploy 再次从 GitHub 构建分支并发布。
在修复失败的构建后使用它，或者在不推送的情况下获取更改。

## 每次推送时重建

当 `deploy create --git` 创建部署时，它会打印一次 Webhook 密钥：

```text
Add this GitHub webhook so pushes redeploy (the secret is shown once):
  URL:          https://api.miosa.ai/api/v1/integrations/github/webhook
  Content type: application/json
  Secret:       <webhook secret>
  Events:       Just the push event
```

将其添加到存储库中的 **Settings**、**Webhooks**、**Add webhook** 下，并使用这些值。
GitHub 使用该秘密对每次交付进行签名。
MIOSA 检查签名，查找该存储库和分支的部署，并构建推送的提交。
没有签名的投递将被拒绝（401），签名错误的投递将被拒绝（403）。
对其他分支的推送将被忽略。

秘密也是创建响应的 `webhook_secret` 字段 (`--json`)。
像密码一样存储它：任何拥有它的人都可以触发构建。
要替换它，请调用 `POST /api/v1/deployments/{id}/webhook-secret` 并使用新值更新 GitHub 上的 Webhook。

## 相关

- [Publishing](/docs/docker-host/publishing) 解释了构建如何成为版本。
- [Domains](/docs/docker-host/domains) 将您自己的域附加到部署。
- [回滚](/docs/docker-host/rollback)使早期版本再次上线。
