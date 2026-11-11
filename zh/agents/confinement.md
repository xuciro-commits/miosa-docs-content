---
slug: "agents/confinement"
title: "智能体禁闭"
description: "智能体在其机器内部运行，并且不能使用 MIOSA 自己的平台密钥，除非环境授予它一个范围内的密钥。"
section: "Agents"
order: 6009
---

智能体在其机器内运行并停留在那里。
它无法使用 MIOSA 自己的平台密钥来访问平台的其余部分。

## 平台键为空

每个沙箱都使用平台密钥及其在环境中的 ID 启动，因此简单的命令可以调用 MIOSA。

运行框架运行没有得到它。
当 MIOSA 启动运行框架时，它会在启动时清空平台密钥和 ID，并在运行脚本中取消设置它们。
因此，Claude Code、Codex、OSA、pi、OpenCode、Prime Agent、Kimi Code、Mistral Vibe 和 Hermes Agent 无法借用 MIOSA 的密钥。

## 授予具有环境的作用域密钥

当您希望智能体有意访问 MIOSA 时，[环境](/docs/cli/environments) 可以为其授予范围内的机器密钥。
该密钥绑定到机器，而不是您的帐户。

## 保留什么

- 保留运行本身提供的值。
- 环境补助金不受影响。

## 相关

- [隔离和权限](/docs/optimal/isolation)：托管聊天可以触及的内容。
- [运行时环境](/docs/guides/runtime-env)：按范围和目标划分的变量。
