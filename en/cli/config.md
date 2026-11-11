---
slug: "cli/config"
title: "Config and profiles"
description: "Named sign-ins and defaults in ~/.miosa/config.toml, plus the config commands, options and environment variables."
section: "CLI, SDKs & MCP"
order: 11011
---

Settings live in `~/.miosa/config.toml` (mode 0600). The API key is never printed by `config`; sign in with `miosa login`.

## Config commands

```bash
miosa config list            # effective settings of the current profile (alias ls)
miosa config get <key>       # print one setting
miosa config set <key> <value>
miosa config path            # print the config file path
```

Keys: `api_url`, `current_sandbox`, `default_workspace`.

```toml
api_url           = "https://api.miosa.ai/api/v1"
api_key           = "msk_u_..."
default_workspace = "default"
current_sandbox   = "my-box"
```

## Profiles

A profile is a named sign-in with its own API URL, organization and defaults. Several profiles let you keep accounts or environments apart.

```bash
miosa login --profile work
miosa profile list
miosa profile use work
miosa --profile default list
```

| Command | What it does |
|---|---|
| `miosa profile add <name>` | Create an empty profile (then `miosa login --profile <name>`) |
| `miosa profile list` | List profiles (alias `ls`) |
| `miosa profile show [name]` | Show a profile's settings (the key is masked) |
| `miosa profile use <name>` | Make a profile the default for new commands |
| `miosa profile rm <name>` | Delete a profile and its stored key |

`miosa profile add` takes `--api-url` and `--workspace`.

### Which profile is in effect

1. `--profile`
2. `MIOSA_PROFILE`
3. the one set with `miosa profile use`
4. `default`

## Credential resolution

1. `--api-key`
2. `MIOSA_API_KEY`
3. the active profile in `~/.miosa/config.toml`

## Environment variables

```text
MIOSA_API_KEY          API key
MIOSA_BASE_URL         API base URL
MIOSA_PROFILE          Profile to use
MIOSA_ORG              Organization to bill
MIOSA_OUTPUT           Output format (text or json)
MIOSA_RETRIES          Retries on 429 and transient 5xx
MIOSA_NO_UPDATE_NOTICE Set to 1 to silence the new-version notice
```

## Sign out

```bash
miosa logout                # remove the stored key of the active profile
miosa logout --all          # delete the config file (all profiles)
```

> **Note**
>
> `miosa doctor` checks configuration, connectivity and authentication, and prints a fix for each failure.
