---
slug: "cli"
title: "CLI overview"
description: "The miosa command line tool. Install, sign in, and drive sandboxes, computers, agents, deployments and every platform resource from the terminal."
section: "CLI, SDKs & MCP"
order: 11001
---

CLI & MCP

The fastest way to control all of MIOSA, for you and your agents. Everything the console does, scriptable, with JSON output.

`npm i -g @miosa/cli`

The miosa CLI on macOS, Linux and Windows. The fastest path inside a container or CI job.

- [Install and authenticate](/docs/cli/install): npm, Homebrew, script or release archive, then sign in.
- [Use from agents](/docs/cli#use-the-cli-from-agents): Let OSA or any agent with a shell drive the platform.
- [Command reference](/docs/cli#command-groups): Every command group, flag and exit code.
- [CLI or MCP](/docs/cli/cli-vs-mcp): Which one to give an agent, and why both work.

`miosa` covers the whole platform: sandboxes and computers, files and previews, agents and their runs, deployments, databases, storage, API keys, webhooks, billing and the organization you work in.

## Quick start

```bash
npm i -g @miosa/cli               # or Homebrew, the install script, or a release archive
miosa --version                   # prints 2.3.1
miosa login                       # browser device-code sign in
miosa whoami                      # user, organization, plan and scopes
miosa create my-box --wait        # provision a sandbox
miosa exec my-box -- echo hello   # run a command
miosa destroy my-box --yes
```

- [Install and authenticate](/docs/cli/install): npm, Homebrew, install script, releases, login and API keys.
- [Machines and lifecycle](/docs/cli/machines): create, list, use, stop, fork, recover, destroy.
- [Run commands and move files](/docs/cli/work-inside): exec, ssh, console, files, services, previews, ports.
- [Devices](/docs/cli/devices): Any runtime you can run work on: sandboxes, computers and more.
- [JSON output and scripting](/docs/cli/scripting): --json, streaming output, exit codes, CI use.

## Use the CLI from agents

The CLI is not only for humans. [OSA](/docs/mcp#osa), MIOSA's open-source agent, runs shell commands, so it can drive `miosa` from your machine or from a MIOSA computer. `osa run "..."` runs it headless with no TUI. Any agent with shell access can do the same, and drive the entire platform with `miosa`: it covers every resource the API and the SDKs expose, not just a tool surface, and it composes with the shell and the tools the agent already has.

- **Install inside a sandbox or CI.** `npm i -g @miosa/cli` is the fastest path in a container or CI job. Sandbox images ship Node, so a fresh agent machine has the CLI in one command.
- **Authenticate with a key.** Set `MIOSA_API_KEY` and every command is authenticated with no browser. Never pass a key as an argument.
- **Get JSON and exit codes.** Add `--json` for machine-readable output and branch on the exit code. `exec` and `ssh` exit with the remote command's status.

```bash
export MIOSA_API_KEY="msk_u_..."
npm i -g @miosa/cli

# An agent provisioning, using and cleaning up a machine, all scriptable
ID=$(miosa create agent-box --size small --ttl 1h --wait --json | jq -r .id)
miosa exec "$ID" -- npm ci
miosa preview create "$ID" 3000 --visibility public --json | jq -r .url
miosa destroy "$ID" --yes
```

An agent running in Claude Code or Codex with shell access uses exactly these commands: provision a sandbox, run a build, publish a preview, read the JSON, clean up. No MCP setup required.

### When the CLI beats MCP

Use the **CLI** when the agent already has a shell. It reaches the whole platform rather than one tool surface, its output is scriptable and composes with `jq`, pipes and files, and `miosa api <METHOD> <path>` covers any endpoint the high-level commands do not. Use **MCP** when the client has no shell, or when you want the model to choose tools rather than write commands. See [CLI or MCP: which to use](/docs/cli/cli-vs-mcp).

## Command groups

Every top-level command, grouped the way the CLI organises them.

| Group | Commands |
|---|---|
| Account | `login`, `logout`, `whoami`, `org`, `limits`, `profile`, `config`, `doctor`, `open`, `update`, `version` |
| Lifecycle | `create`, `list`, `info`, `use`, `destroy`, `stop`, `pause`, `resume`, `fork`, `recover`, `extend`, `tag`, `wait`, `usage`, `computer`, `device` |
| Work inside | `exec`, `ssh`, `console`, `files`, `process` (`ps`), `services`, `logs`, `events`, `ports`, `preview`, `url`, `proxy`, `desktop` |
| State | `snapshot`, `checkpoint`, `restore`, `env`, `policy` |
| Sandbox resources | `sandbox` (`resources`, `resize`) |
| Agents | `prompt`, `run`, `chat`, `agent`, `connections`, `mcp` |
| Platform | `deploy`, `batch`, `host`, `db`, `storage`, `volume`, `workflow`, `schedule`, `function`, `template`, `project`, `workspace`, `forge`, `catalog`, `regions`, `egress`, `cloud`, `gateway` |
| Access and billing | `api-key`, `service-account`, `webhook`, `member`, `billing`, `audit`, `alerts`, `notifications` |
| Raw | `api <METHOD> <path>` with `-d` (`@file` or `-`) and `-H` |

## Machine arguments

Anywhere a sandbox is expected you may give its name, its id, `current` (set with `miosa use`) or `self` (inside a sandbox). Commands that act on a computer take `--computer`.

```bash
miosa exec my-box -- npm test      # by name
miosa exec sbox_abc123 -- npm test # by id
miosa use my-box                   # then omit the name
miosa exec -- npm test
```

## Global flags

These apply to every command.

```text
--api-key string      API key (overrides MIOSA_API_KEY and config)
--api-url string      API base URL (overrides MIOSA_BASE_URL and config)
--profile string      Config profile to use (MIOSA_PROFILE)
--org string          Organization to bill for this command (MIOSA_ORG)
--json                Machine-readable JSON (same as -o json)
-o, --output string   text (default) or json (MIOSA_OUTPUT)
-q, --quiet           Suppress informational output
--retries int         Retries on 429 and transient 5xx (default 3, MIOSA_RETRIES)
--timeout int         Request timeout in seconds; for exec and ssh the command timeout
--no-update-notice    Silence the new-version notice
```

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Error |
| 2 | Usage |
| 3 | Not signed in, or forbidden |
| 4 | Not found |
| 5 | Conflict or invalid input |
| 6 | Rate limited (after retries) |
| 7 | Server or network error |

`miosa exec` and `miosa ssh` exit with the remote command's status.
