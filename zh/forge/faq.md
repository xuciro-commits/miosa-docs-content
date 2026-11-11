---
slug: "forge/faq"
title: "伪造限制和常见问题解答"
description: "Forge 做了什么和没有做什么：可见性、权限、合并方法、问题、存储库机密和 CI。"
section: "Services & projects"
order: 1039
---

## 存储库是私有的吗？

是的，默认情况下。
存储库是 `private`、`internal` 或 `public`，使用 `repo create --visibility` 设置或稍后使用 `repo update` 进行更改。
您的组织的成员可以通过其组织角色进行访问，并且 `grant set` 为一个人在一个存储库上提供了额外的角色。

## 我的 API 密钥需要哪些权限？

`forge:read` 用于克隆、获取和读取，`forge:write` 用于推送、打开和合并拉取请求、报告检查和发布版本。
`miosa login` 给出了两者的键。
没有它们的密钥将获得 `insufficient scope` 及其所需的权限名称。

## 我可以使用 GitHub CLI 或其他 Git 主机吗？

Forge 通过 HTTPS 使用标准 Git，因此任何 Git 客户端在 `miosa forge auth setup-git` 运行后即可工作。
拉取请求、检查、策略和发布使用 `miosa forge`、API 或 SDK。

## 我可以从 GitHub 导入存储库吗？

是的，从控制台。
导入会复制一次历史记录； GitHub 应用程序连接保持存储库镜像。

## Forge 不做什么

|不支持 |该怎么办 |
|---|---|
|问题 |在您自己的工具中或在拉取请求评论中跟踪工作 |
|存储库的秘密 |使用 `miosa deploy env-set` | 在部署上设置它们
|推送运行的 CI |在其他地方运行测试并使用 `miosa forge check report` | 报告结果
|压缩、变基和自动合并 |合并创建合并提交；服务器拒绝其他方法而不合并|
|从具有自己身份的沙箱中推送 |请参阅[从沙箱中锻造](/docs/forge/sandboxes) |
|原始文件下载、责怪和 Git LFS |克隆存储库 |

> **注意**
>
> Forge 今天不做这些。

## 为什么我的合并被拒绝？

拉取请求尚不符合分支策略，并且没有合并任何内容。
错误指出了缺少的内容以及修复它的命令；请参阅[修复“需要批准”](/docs/forge/pull-requests#fix-needs-approval)。
