---
slug: "forge/sandboxes"
title: "从沙箱中锻造"
description: "使用沙箱自己的身份，使用普通 git 从沙箱内部克隆并推送到 Forge。"
section: "Services & projects"
order: 1036
---

由您或智能体在沙箱内编写的代码应该以简单的 `git push` 形式存放在 Forge 存储库中。
为此，不会将 API 密钥复制到沙箱中。

> **注意：推出**
>
> 此页面上的所有内容现已推出。
> 在它到达您的组织之前，从沙箱内部进行锻造的 `git clone` 或 `git push` 会回答 `Authentication failed`。

## 它是如何工作的

沙箱已经带有自己的令牌，其范围仅限于其组织和工作空间。
Forge 接受沙箱自己工作区的存储库的令牌，并且沙箱在启动时为 `forge.miosa.ai` 写入一个 git 凭证帮助程序。
所以在沙箱内，git 可以正常工作：

```bash
git clone https://forge.miosa.ai/<org>/<repo>.git
cd <repo>
git checkout -b from-sandbox
echo "<p>Written in a sandbox.</p>" >> public/index.html
git commit -am "Edit from a sandbox"
git push -u origin from-sandbox
```

沙箱仅到达其自己工作区的存储库，与其他人具有相同的分支策略。
然后从您的计算机、CLI 或 API 打开拉取请求，如 [拉取请求](/docs/forge/pull-requests) 中所示。

## 今天，在您收到之前

不要将个人 API 密钥复制到沙箱中来解决它：可以读取沙箱的任何人和任何智能体都可以读取该密钥。
如果您现在需要这些文件，请使用 `miosa files cp -r my-box:/workspace/site ./site` 将它们复制出来并从您的计算机中推送。
