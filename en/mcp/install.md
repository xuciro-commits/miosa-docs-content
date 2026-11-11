---
slug: "mcp/install"
title: "Install and authenticate"
description: "Choose the hosted MIOSA MCP endpoint or the local stdio package, create a key, and set organization context."
section: "CLI, SDKs & MCP"
order: 11023
---

There are two ways to run the MIOSA MCP server. Both hit the same MIOSA REST API. The only difference is whether the JSON-RPC bridge runs in MIOSA's infrastructure (hosted) or on your machine (stdio).

## Hosted (recommended)

No install. Point any MCP client at the public endpoint with your key as a Bearer token.

```text
https://api.miosa.ai/api/v1/mcp
```

```bash
claude mcp add --transport http miosa \
  https://api.miosa.ai/api/v1/mcp \
  --header "Authorization: Bearer msk_u_your_key_here"
```

See the [overview](/docs/mcp) for the config file form for Claude Code, Codex, Cursor and Claude Desktop.

## Local stdio

Use stdio when you need to wrap the MCP layer with custom local logic, or your client does not support remote MCP.

```bash
pip install miosa-mcp
```

Requires Python 3.10 or newer. Package version 0.4.3.

The standalone package registers its own tool set and its own names: 194 tools with the prefix-first scheme (`sandbox_create`, `sandbox_exec`, `computer_create`). It is a separate server from the hosted endpoint, which registers 151 tools with a different scheme (`create_sandbox`, `exec`, `computer_create`). The [tools reference](/docs/mcp/tools) documents the hosted server; the local package's tool names follow the same pattern as its own `list_tools` output.

Add to `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "miosa": {
      "command": "python",
      "args": ["-m", "miosa_mcp"],
      "env": {
        "MIOSA_API_KEY": "msk_u_your_key_here",
        "MIOSA_TENANT": "optional-organization-slug"
      }
    }
  }
}
```

## Connect OSA

OSA is MIOSA's open-source agent. It runs on your machine or on a MIOSA computer, and works with your Claude or ChatGPT subscription. It reads MCP servers from `~/.osa/mcp.json`, so the hosted endpoint is a `url` plus an `Authorization` header there. The full setup, including the `/mcp add` command, is in the [overview](/docs/mcp#osa).

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_your_key_here" }
    }
  }
}
```

## Connect Hermes

Hermes Agent reads `mcp_servers` from `~/.hermes/config.yaml` and resolves `${MIOSA_API_KEY}` from `~/.hermes/.env`. See the [overview](/docs/mcp#hermes) for the block.

## Install or serve from the CLI

The `miosa` CLI can configure or run the MCP server for you, so you do not have to edit a client config by hand.

```bash
miosa mcp install                                  # Claude Code, user scope
miosa mcp install --client cursor                  # Cursor
miosa mcp install --client gemini                  # Gemini CLI
miosa mcp install --client claude --scope project  # a shared .mcp.json
miosa mcp install --print                          # show the change, write nothing
```

`miosa mcp install` does not write OSA or Hermes config; use the two sections above for those. It writes the hosted endpoint and your API key into the client's config file, keeping the other servers it already lists. It takes `--client` (`claude`, `cursor`, `gemini` or `manual`), `--name` (default `miosa`), `--scope` (`user` or `project`, for Claude), `--url`, `--force` and `--print`.

For clients that need a stdio server, run the bridge over standard input:

```bash
miosa mcp serve
```

`miosa mcp serve` reads newline-delimited JSON-RPC 2.0 messages, forwards each to MIOSA MCP, and writes one JSON line per response. Point a client at it with:

```json
{
  "mcpServers": {
    "miosa": {"command": "miosa", "args": ["mcp", "serve"]}
  }
}
```

List the tools the server exposes:

```bash
miosa mcp tools
```

`miosa mcp serve` and `miosa mcp tools` both take `--url` (default: the API base URL plus `/mcp`). The Python package also exposes a `miosa-mcp` entry point for the same stdio role.

See [Advanced MCP setup](/docs/mcp/advanced) for the per-client configuration matrix, the environment variable reference (local server only), and troubleshooting.

## Get a key

Create a user API key at [miosa.ai/dashboard/api-keys](https://miosa.ai/dashboard/api-keys). It looks like `msk_u_<hex>`. Treat it like a password.

From the CLI:

```bash
miosa api-key create mcp-key --preset agent
miosa api-key create mcp-key --preset agent --quiet-key   # print only the secret
```

## Organization context

The hosted endpoint scopes every request to the organization that owns the API key, so there is no organization-switch tool there. The local stdio package (`miosa-mcp`) is the one that registers organization helpers and reads `MIOSA_TENANT` for an initial organization:

```text
organization_list()                       # organizations available to you
organization_current()                    # the organization in use
organization_switch(organization="acme")  # switch by UUID or slug
```

These tools belong to the local package, not the hosted server; the [tools reference](/docs/mcp/tools) lists the hosted tools.

> **Note**
>
> The key is checked by the same auth plug that protects every REST endpoint, so the MCP surface inherits tenant scoping, plan limits, credit checks and audit logging.

## Verify

```bash
claude mcp list
```

You should see `miosa` with a connected status. If a call returns `401`, the key is invalid, expired or revoked. A `429` means you hit the plan's rate limit (same limit as the REST API; the window is per minute).

You can also list the catalogue over the wire:

```bash
curl -sS -X POST https://api.miosa.ai/api/v1/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer msk_u_your_key_here" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | jq .
```
