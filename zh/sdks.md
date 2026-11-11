---
slug: "sdks"
title: "SDK 概览"
description: "MIOSA 官方 TypeScript、Python、Go、Elixir、Java 和 Rust 客户端库，包含包名、版本、安装命令和状态。"
section: "CLI, SDKs & MCP"
order: 11014
---

官方客户端库封装了 MIOSA REST API。每个 SDK 都涵盖身份验证、沙箱、exec 与文件、快照、计算机、智能体以及类型化错误。各语言的覆盖范围有所不同；每个页面都会标注某个 SDK 尚未实现的功能。

## 语言

[TypeScript](/docs/sdks/typescript) `@miosa/sdk`

3.6.0

`npm install @miosa/sdk`

[Python](/docs/sdks/python) `miosa`

2.15.0

`pip install miosa`

[Go](/docs/sdks/go) `github.com/Miosa-osa/miosa-go/v2`

v2.2.0

`go get github.com/Miosa-osa/miosa-go/v2`

[Elixir](/docs/sdks/elixir) `miosa`

1.1.0

`{:miosa, "~> 1.1"}`

[Java](/docs/sdks/java) `ai.miosa:miosa-sdk`

0.5.1

`implementation 'ai.miosa:miosa-sdk:0.5.1'`

[Rust](/docs/sdks/rust) `miosa`

0.1.0

`cargo add miosa`

## 所有 SDK 的共同点

- 使用 API key（`msk_u_*`）或访问令牌进行**身份验证**，可从构造函数或 `MIOSA_API_KEY` 读取。
- **沙箱**：创建、列出、获取和删除。
- **Exec** 用于运行命令，**文件**用于在机器内读写。
- **快照**和**计算机**。
- **类型化错误**，携带状态、代码和请求 ID。

## 预期差异

- **智能体定义**：Elixir 提供完整的 CRUD；TypeScript 提供定义 CRUD 以及独立的运行 API；Python、Go 和 Java 暴露运行和设置，但不提供定义 CRUD。详见各语言页面。
- **Go、Java 和 Elixir 中的沙箱 exec** 通过专用的沙箱 exec 路由执行；句柄级便捷方法因语言而异。
- **文件方法名不同**：即使同一个 SDK 内，沙箱接口和计算机接口的方法名也不一样。每个页面都会列出具体名称。
