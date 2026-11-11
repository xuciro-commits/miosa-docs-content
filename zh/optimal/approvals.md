---
slug: "optimal/approvals"
title: "批准"
description: "决定哪些最佳步骤等待您，在聊天中批准或拒绝它们，允许对话的其余部分执行该步骤，并让 API 密钥批准。"
section: "Optimal"
order: 7005
---

Optimal 在行动之前先询问。
当某个步骤需要您做出决定时，聊天会暂停，并且批准卡会出现在作曲家上方。

## 批准或拒绝

批准卡列出了 Optimal 想要采取的步骤。

- **批准** 让这一步骤运行。
- **Decline** 拒绝它，Optimal 会以另一种方式继续或告诉您它不能做什么。
- **允许此聊天** 批准它并允许运行相同类型的步骤，而无需询问其余的聊天。
  它仅显示可以记住的步骤。

当有多张卡在等待时，最新的首先显示“N more waiting”。

## 选择 Optimal 之前询问的内容

在“首选项”中的“批准”下，“先询问”设置工作区中等待您的步骤：

|设置|等待的步骤|
|---|---|
|每一步|每一步都用到一个工具|
|仅更改|改变一些事情的步骤|
|仅删除 |删除某些内容的步骤 |

默认值是**仅更改**。
该设置是您的：它适用于您在此工作区、每台设备上的聊天。

## 自动批准

**自动批准**开关位于最佳聊天的聊天标题中。
当它打开时，Optimal 会不间断地运行此工作区中所需的每个工具。
将其关闭以返回**之前询问**设置。

在空聊天中，相同的开关位于作曲家下方。
自动批准仅适用于最佳大脑。

## 允许在此聊天中

您通过 **允许此聊天** 允许的每个步骤都列在聊天标题中的 **...** > **此聊天中允许 (N)** 下。
**删除**退一步询问。
当不允许任何事情时，列表中会显示“在此聊天中未经询问，不允许任何事情”。

## 每一个待批准的项目

最佳侧栏中的**批准**列出了工作区中聊天的批准。
每行都说明它来自哪个聊天，您可以搜索列表。
当要求批准的回合结束时，待批准的批准就会过期。

## 登录站点

当 Optimal 需要在浏览器中登录网站时，它会显示安全登录卡，而不是在聊天中询问。
您输入的内容会直接进入登录页面，不会保存在聊天中。
勾选**保存下次**以保留登录状态；已保存的登录信息列于 **首选项** 下的 **登录** 下，仅包含屏蔽名称。
**现在不行**跳过它。

## API 的批准

仅当密钥具有 **批准或拒绝 Optimal 的更改** 权限 (`osa:approve`) 时，从 API 密钥创建的 Optimal 令牌才能批准或拒绝。
令牌的 `can_decide_approvals` 字段表示是否可以。
如果没有它，决策将路由答案 `403 approval_scope_required`，因此除非您允许，否则脚本无法批准自己的更改。

```bash
# Pending approvals in one chat
curl -s "https://optimal.miosa.ai/api/v1/approvals?session_id=$CHAT&status=pending" \
  -H "Authorization: Bearer $TOKEN"

# Approve one
curl -s -X POST https://optimal.miosa.ai/api/v1/approvals/$APPROVAL/approve \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"reason": "Looks right"}'
```

`/reject` 拒绝。
决定两次答案`409 already_decided`。
请参阅[最佳 API](/docs/optimal/api#approvals)。

> **注意：自动批准跟随您**
>
> 自动批准和**先询问**适用于每台设备上工作区中的最佳聊天。
> 平台政策仍然适用。

## 相关

- [隔离和权限](/docs/optimal/isolation)：Optimal 可以达到的范围。
- [任务、工件和历史记录](/docs/optimal/workspace)：其他最佳侧边栏页面。
