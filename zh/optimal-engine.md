---
slug: "optimal-engine"
title: "Optimal Engine"
description: "Optimal Engine 是工作区背后的自托管内存和知识引擎 - 它保留源证据，构建经过审查的内存，并通过 HTTP、CLI 和 MCP 将引用的上下文返回给人员和智能体。"
section: "Optimal Engine"
order: 8000
---

最优引擎是一个适用于个人、团队或公司的记忆和知识引擎。
它保留您提供的来源，将其转化为某人已审阅的记忆，并将引用的上下文返回给人员和智能体。
它是在 Apache 2.0 许可证下开源的，位于 [github.com/Miosa-osa/OptimalEngine](https://github.com/Miosa-osa/OptimalEngine)。

Optimal Engine不是最佳聊天智能体。
[Optimal](/docs/optimal) 是您在控制台中聊天的智能体。
Optimal Engine 是它记住的存储：您自己运行的服务器，或者 MIOSA 上托管的服务器。

## 它的作用

|步骤|引擎的作用 |
|---|---|
|组织|保留组织、工作区和类型节点（例如人员、产品和项目）的树 |
|捕捉|存储原始文本、文件、聊天会话、Notion 导出以及从 Slack、GitHub 和 Linear 等应用程序提取的记录 |
|记住|对每个输入进行分类，提取声明，并仅在审查或置信规则之后将声明提升为事实 |
|检索 |使用全文排名进行搜索，在设置嵌入模型时添加矢量搜索，并返回其来源的上下文 |
|操作|为智能体提供每日简报、开环、协议，并允许、询问或拒绝他们可以做什么的规则 |
|项目|写出与 Markdown 文件、wiki 页面、工作区包和 HTTP API 相同的状态 |

引擎不使用语言模型编写答案。
`ask` 返回上下文和答案应来自的来源，并且您的智能体或模型编写答案。

## 两种运行方式

| |自托管 |在 MIOSA 上 |
|---|---|---|
|它在哪里运行 |您的机器、服务器或容器| MIOSA 工作区中的私人计算机 |
|谁来运营 |您：安装、密钥、备份、升级 | MIOSA：配置、备份、升级、运行状况检查 |
|搜索 |全文以及使用您自己的嵌入模型的矢量搜索 |全文|
|如何访问它 | `http://127.0.0.1:4200` 或您自己的域 | **Memory** 页面、Optimal 聊天、`miosa engine` 命令和工作区中的机器 |
|可用性 |任何人，来自存储库 |每个组织启用 |

自托管不需要 MIOSA 的任何帮助。
请参阅[自托管快速入门](/docs/optimal-engine/quickstart)或[MIOSA](/docs/optimal-engine/miosa)上的Optimal Engine。

> **注意：当前版本**
>
> 这些页面描述了 Optimal Engine 0.3.10，API 版本 `v1`。
> 使用 `GET /api/version` 检查正在运行的引擎报告的内容。

## 下一步去哪里

- [概念](/docs/optimal-engine/concepts)：工作空间、节点、来源、声明、事实和上下文包。
- [快速入门](/docs/optimal-engine/quickstart)：运行引擎，存储源，并在大约十分钟内搜索它。
- [安装和配置](/docs/optimal-engine/install)：要求、Docker、设置、存储和 API 密钥。
- [记忆和检索](/docs/optimal-engine/memory-and-retrieval)：如何搜索、询问和审查索赔。
- [API 参考](/docs/optimal-engine/api)：身份验证、错误和主要路由示例。
- [MIOSA](/docs/optimal-engine/miosa) 上的Optimal Engine：工作区内存、托管引擎和 CLI 的密钥。
