---
slug: "cli/agents"
title: "Agents, chats and runs"
description: "Run agents on a sandbox or computer, manage saved agent definitions, chats and durable runs, and see how runs stream."
section: "CLI, SDKs & MCP"
order: 11007
---

## Bring your own model

Agent runs use your own model-provider connections, never MIOSA platform keys. Mint a key that can do this with `miosa api-key create NAME --preset cli`. Add a connection with `miosa connections add models` (the key is read from a prompt or stdin, never from an argument). A run without a usable connection fails with a message that names this command.

```bash
miosa api-key create agent-key --preset cli
miosa connections add models
```

## Prompt

Run an agent on a sandbox or computer and follow it.

```bash
miosa prompt --sandbox my-box "fix the failing test"
miosa prompt --new-machine --harness codex "write a README"
miosa prompt --sandbox my-box --continue
```

| Flag | Meaning |
|---|---|
| `-s, --sandbox string` | Sandbox to run in (default: the current sandbox) |
| `--computer string` | Computer to run in |
| `--agent string` | Run a saved agent (name or id) instead of an ad hoc one |
| `--harness string` | `claude-code`, `codex` or `osa` (default: the agent's, or `claude-code`) |
| `--model string` | Model for the harness (default: the harness default) |
| `--new-machine` | Provision a machine for this run |
| `--machine string` | Kind of new machine: `sandbox` or `computer` |
| `--size string` | Size of a new machine |
| `--template string` | Template of a new machine |
| `--chat string` | Chat id; continues the chat if it has runs |
| `--new-chat` | Start a chat (a new chat id is printed) |
| `--reuse string` | New machine lifetime: `run` (one per run) or `chat` (one per `--chat`, files persist) |
| `--after-run string` | What happens to a new machine when the run ends: `destroy`, `pause` or `keep` |
| `--max-time int` | Stop the run after this many seconds |
| `--no-follow` | Print the run id and return without waiting |
| `--notify-email` | Email you when the run finishes |
| `--notify-webhook string` | Webhook id to notify when the run finishes |

## Saved agents

An agent is a saved, versioned definition: harness, model, instructions, the machine it runs on, and what it may do. `<agent>` is a name or an id.

```bash
miosa agent new reviewer --harness claude-code --instructions-file review.md
miosa agent publish reviewer --file config.yaml
miosa agent list
miosa agent diff reviewer 2 3
miosa agent run reviewer "review PR 42" --sandbox my-box
miosa agent harnesses
```

`miosa agent new` takes `--harness` (`claude-code`, `codex`, `osa` or `custom`), `--model`, `--instructions` or `--instructions-file`, `--overdrive` (run the harness without approval prompts), `--timeout-sec`, `--description`, and a full `--file` (YAML or JSON).

Other subcommands: `agent get`, `agent versions`, `agent version`, `agent update`, `agent duplicate`, `agent rm`, `agent runs`, `agent templates`, `agent triggers`, `agent approvals`, `agent authority`, `agent defaults`, `agent harness-versions`.

## Runs

List, follow and stop agent runs.

```bash
miosa run list --status running
miosa run get <id>
miosa run follow <id>
miosa run events <id>
miosa run logs <id>
miosa run messages <id>
miosa run outputs <id>
miosa run files <id>
miosa run diagnostics <id>
miosa run download <id> -O out.tar
miosa run steer --sandbox my-box "also add tests"   # queue the next turn for the running agent
miosa run interrupt --sandbox my-box                # stop the agent attached to a machine
miosa run stop <id>
miosa run usage --group-by model
```

`miosa run list` filters with `--agent-id`, `--chat`, `--harness`, `--source`, `--status`, `--target`, `--workspace-id` and `--limit`.

## Chats

Chats are agent conversations that remember.

```bash
miosa chat list
miosa chat start --sandbox my-box --harness claude-code
miosa chat show <chat-id>
miosa chat context <chat-id>     # how full the context is
miosa chat compact <chat-id>     # compact it so the chat can go on
```

## Connections

Model providers, apps and tool servers your agents use.

```bash
miosa connections add models
miosa connections list --kind models
miosa connections test <name>
miosa connections share <name>
miosa connections rm <name>
```

`miosa connections add` takes `--key-stdin` (never a key on the command line), `--label`, `--workspace-id`, `--url` (for an MCP tool server) and `--header-env HEADER=ENV_VAR` (repeatable).

> **Note**
>
> Runs are the canonical agent execution contract. `miosa prompt`, `miosa agent run` and `miosa chat` all create runs; `miosa run` reads and controls them.
