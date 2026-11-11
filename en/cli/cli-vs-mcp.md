---
slug: "cli/cli-vs-mcp"
title: "CLI or MCP: which to use"
description: "The miosa CLI is for humans and scripts in a terminal; MCP gives AI clients the same capabilities as tools. Both use the same API key and scopes."
section: "CLI, SDKs & MCP"
order: 11000
---

CLI & MCP

Two ways into the same platform, with the same API key and the same scopes. Pick the one your client can run.

[CLI overview](/docs/cli)[MCP overview](/docs/mcp)[Install the CLI](/docs/cli/install)[MCP tools](/docs/mcp/tools)

- The **CLI** (`miosa`) is for humans, scripts and agents with a shell. You type commands, read text or JSON, and branch on exit codes.
- **MCP** gives AI clients (Claude Code, Codex, Cursor, Claude Desktop) the same capabilities as tools. The model calls tools by name instead of running shell commands.

Both use the **same API key and the same scopes**. A key that can create a sandbox from the CLI can create one over MCP, and the reverse. The only difference is the surface.

## Which should an agent use?

- **Has a shell? Use the CLI.** Claude Code, Codex, Cursor's terminal and any CI job can run `miosa`. The CLI is often the better choice: it reaches the whole platform, not just one tool surface; output is scriptable; and it composes with `jq`, pipes and files.
- **No shell? Use MCP.** Claude Desktop chat and other hosted chat products have no shell, so tools are the only path. MCP lets the model call capabilities by name.

Nothing is lost either way, because the key, the scopes and the resources are identical.

## Comparison

| | CLI | MCP |
|---|---|---|
| Who drives it | A person, a script, or an agent with a shell | An AI client without a shell (Claude Desktop), or one that prefers tools |
| Interface | `miosa <command> [flags]` | JSON-RPC tools over Streamable HTTP or stdio |
| Install | npm, Homebrew, install script, Releases | Hosted endpoint, no install, or `pip install miosa-mcp` |
| Output | Text or JSON, with exit codes | Structured tool results and images |
| Coverage | The whole platform, plus `miosa api` for any endpoint | The MCP tool surface |
| Best for | Agents with shell access, build loops, CI, one-off operations | Clients without a shell |
| Auth | `msk_u_*` key or a profile | `msk_u_*` key as a Bearer token |
| Scope model | Same key scopes | Same key scopes |

## Use both

A common split: an agent with a shell drives the platform with the CLI, and a client without a shell reaches the same account over MCP with the same key.

```bash
ID=$(miosa create agent-box --size small --wait --json | jq -r .id)   # CLI, from a shell agent
miosa exec "$ID" -- npm test
# the same account over MCP: exec(sandbox_id=ID, command="npm run build")
miosa list --json | jq -r '.data[].name'                              # CLI, read back
```

- [CLI overview](/docs/cli): Install, command groups, global flags and exit codes.
- [MCP overview](/docs/mcp): Connect Claude Code, Codex, Cursor or Claude Desktop.

> **Note**
>
> Both surfaces inherit the same tenant scoping, plan limits, credit checks and audit logging, because both sit in front of the MIOSA REST API.
