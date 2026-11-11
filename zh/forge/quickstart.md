---
slug: "forge/quickstart"
title: "Forge 快速入门"
description: "创建一个 Forge 存储库，使用普通 git 推送到它并在五分钟内打开您的第一个拉取请求。"
section: "Services & projects"
order: 1034
---

这大约需要五分钟。
您需要 `miosa` CLI（版本 2.3.10 或更高版本）和 `git`。

## 1. 登录

```bash
miosa login
miosa whoami
```

`miosa login` 为 CLI 提供具有 Forge 权限的密钥，`forge:read` 和 `forge:write`。
如果您使用自己创建的 API 密钥，请在控制台中的 **API 密钥** 下创建它时勾选这两个权限，然后勾选 `export MIOSA_API_KEY=msk_...`。
如果没有它们，每个 `miosa forge` 命令都会回答 `insufficient scope`。

## 2. 让 git 使用您的 MIOSA 登录名

```bash
miosa forge auth setup-git
```

```text
ok  git now uses your MIOSA login for https://forge.miosa.ai
```

这会将 `forge.miosa.ai` 的凭证助手添加到您的全局 git 配置中。
然后，Git 使用您的 MIOSA 密钥登录，因此您无需粘贴密码或令牌。

## 3. 创建存储库并克隆它

```bash
miosa forge repo create hello-forge
miosa forge repo clone hello-forge
cd hello-forge
```

```text
ok  Created repository "hello-forge" in the current organization
  Visibility: private
  Clone URL:  https://forge.miosa.ai/<org>/hello-forge.git
```

新的存储库是私有的并且开始时是空的，因此 git 会警告您克隆了一个空的存储库。

## 4. 提交并推送

```bash
mkdir -p public
printf '<!doctype html>\n<title>Hello from Forge</title>\n<h1>Hello from Forge</h1>\n' > public/index.html
git add .
git commit -m "First page"
git push -u origin main
```

这就是普通的 git：任何通过 HTTPS 使用 Git 的客户端、编辑器或工具都以相同的方式工作。

## 5. 打开拉取请求

```bash
git checkout -b greeting
sed -i.bak 's/Hello from Forge/Hello again from Forge/' public/index.html && rm public/index.html.bak
git commit -am "Change the greeting"
git push -u origin greeting
miosa forge pr new hello-forge --head greeting --title "Change the greeting"
miosa forge pr merge hello-forge 1
```

```text
ok  Merged pull request #1 into main as 0d4d9eebbc76
```

拥有一名成员的组织可以合并自己的拉取请求。
一旦队友加入，拉取请求首先需要他们的批准；请参阅[分支政策](/docs/forge/pull-requests#branch-policy)。

## 清理

```bash
miosa forge repo list
miosa forge repo delete <repository-id> --yes
```

`repo delete` 采用 `repo list` 显示的存储库 ID，并且它是永久的。

## 下一步

- [拉取请求、检查和分支策略](/docs/forge/pull-requests)
- [每个锻造命令](/docs/forge/cli)
