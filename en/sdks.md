---
slug: "sdks"
title: "SDKs overview"
description: "Official MIOSA client libraries for TypeScript, Python, Go, Elixir, Java and Rust, with package names, versions, install commands and status."
section: "CLI, SDKs & MCP"
order: 11014
---

Official client libraries wrap the MIOSA REST API. Every SDK does authentication, sandboxes, exec and files, snapshots, computers, agents and typed errors. Coverage varies by language; each page marks what a given SDK does not implement yet.

## Languages

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

## What every SDK shares

- **Auth** with an API key (`msk_u_*`) or an access token, read from the constructor or from `MIOSA_API_KEY`.
- **Sandboxes** with create, list, get and delete.
- **Exec** to run a command, and **files** to read and write inside a machine.
- **Snapshots** and **computers**.
- **Typed errors** carrying a status, a code and a request id.

## Differences to expect

- **Agent definitions** are full CRUD in Elixir; TypeScript has definition CRUD plus separate run APIs; Python, Go and Java expose runs and settings but not definition CRUD. See each page.
- **Sandbox exec in Go, Java and Elixir** runs through a dedicated sandbox exec route; the handle-level convenience methods differ by language.
- **File method names differ** between the sandbox surface and the computer surface even inside one SDK. Each page lists the exact names.
