---
slug: "cli/api"
title: "Raw API access"
description: "Call any MIOSA API endpoint from the CLI with your stored credentials, using miosa api."
section: "CLI, SDKs & MCP"
order: 11010
---

`miosa api` sends a request to the MIOSA API with your stored credentials and prints the response body. The path is relative to the API base URL (`/api/v1`). Use it for anything the CLI has no command for yet.

```bash
miosa api [METHOD] <path> [flags]
```

The method defaults to GET, or POST when `--data` is given.

```bash
miosa api /credits/balance
miosa api GET /sandboxes/my-box/ports
miosa api POST /webhooks -d '{"url":"https://example.com/hook","events":["sandbox.ready"]}'
miosa api PATCH /sandboxes/my-box/tags -d @tags.json
echo '{"name":"x"}' | miosa api POST /workspaces -d -
```

| Flag | Meaning |
|---|---|
| `-X, --method string` | HTTP method (default GET, or POST with `--data`) |
| `-d, --data string` | Request body: JSON, `@file`, or `-` for stdin |
| `-H, --header stringArray` | Extra header, `'Name: value'` (repeatable) |
| `-i, --include` | Print the status line and response headers first |

A failing status exits non-zero (see exit codes) and still prints the body.

## When to reach for the raw API

| Need | Use |
|---|---|
| Code/build/test/runtime workspace | `miosa create`, `miosa exec`, `miosa files`, `miosa services`, `miosa url` |
| Full GUI/browser computer | `miosa api /computers ...` or an SDK (`client.computers`) |
| Docker Host appliance | SDK or API Docker Host endpoints; publish from a sandbox to Docker Host |
| Product/template readiness | `miosa catalog` |
| Anything else | `miosa api <METHOD> <path>` |

> **Tip**
>
> `miosa api` uses the same profile, key and organization as every other command. Combine it with `--json` and `jq` for scripting. See [JSON output and scripting](/docs/cli/scripting).
