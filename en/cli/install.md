---
slug: "cli/install"
title: "Install and authenticate"
description: "Install the miosa CLI from npm, Homebrew, the install script or a release archive on macOS, Linux and Windows, verify 2.3, then sign in."
section: "Get started"
order: 5
---

`miosa` is the official MIOSA command line tool. There is one `miosa` CLI: version 2.3. It runs on macOS, Linux and Windows.

## 1. Install

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/miosa-cli-go/main/install.sh | sh
```

macOS and Linux. Installs to `~/.local/bin/miosa` for non-root users and `/usr/local/bin/miosa` for root. Override with `INSTALL_DIR=/usr/local/bin`.

**Windows**

```bash
npm i -g @miosa/cli
```

Installs the `miosa` 2.3 CLI. The package resolves the right native build for your platform, including Windows x64. Works the same on macOS, Linux and Windows.

The installer does these steps:

1. Detects your OS and CPU.
2. Downloads the matching `miosa` binary and checks its SHA-256.
3. Installs the binary onto your `$PATH`.

Then sign in with `miosa login` (step 3 below): it opens your browser, and the key for this machine is stored in `~/.miosa/config.toml`.

Prefer another install path?

**Homebrew**

```bash
brew install --cask Miosa-osa/homebrew-tap/miosa
```

macOS only.

**Manual**

Download the archive for your platform from [GitHub Releases](https://github.com/Miosa-osa/miosa-cli-go/releases/latest) and place the binary in a directory on your `$PATH`.

```text
miosa_2.3.1_darwin_arm64.tar.gz
miosa_2.3.1_darwin_amd64.tar.gz
miosa_2.3.1_linux_arm64.tar.gz
miosa_2.3.1_linux_amd64.tar.gz
miosa_2.3.1_windows_amd64.zip
```

## 2. Verify

```bash
miosa --version
miosa version
```

These print `2.3.1`. If they print `1.x`, run `npm i -g @miosa/cli@latest` or `brew upgrade` to move onto the 2.x line.

## Windows

The npm install is the simplest path on Windows: `npm i -g @miosa/cli` resolves the `win32-x64` build (`@miosa/cli-win32-x64`) and puts `miosa` on your PATH.

To install by hand, download `miosa_2.3.1_windows_amd64.zip` from [GitHub Releases](https://github.com/Miosa-osa/miosa-cli-go/releases/latest), extract `miosa.exe`, and add its folder to your `PATH`. In PowerShell:

```powershell
$dir = "$env:USERPROFILE\miosa"
Expand-Archive -Path "$env:USERPROFILE\Downloads\miosa_2.3.1_windows_amd64.zip" -DestinationPath $dir
[Environment]::SetEnvironmentVariable("Path", "$env:Path;$dir", "User")

# open a new terminal, then
miosa --version
```

`SetEnvironmentVariable` with the `User` scope makes the change permanent without admin rights; it only affects terminals opened after you run it. For the current session only, use `$env:Path += ";$dir"` instead.

Windows builds are x64 only. There is no windows/arm64 archive.

## Upgrading from 1.x

1.x was the Node CLI. 2.x is the same `miosa` command with a rewritten command surface, and the installs above now all give you 2.3.x.

What changed:

- **Sandbox commands lost the `sandbox` prefix.** `miosa sandbox create` is now `miosa create`.
- **Deployment work moved under `miosa deploy`.** The old `miosa app` and top-level `releases`, `rollback` and `builds` commands are now `miosa deploy` subcommands. New in 2.2: `miosa deploy create` publishes a sandbox, a git repository or a local directory.
- **Contexts became profiles.** `miosa context` is now `miosa profile`, stored in `~/.miosa/config.toml`.
- **Restored in 2.1.** Several 1.x command families came back, renamed: `miosa device` (any runtime kind), `miosa mcp`, and the `miosa host` families for containers, meshes, GitHub Actions runners, backups and tunnels.
- **Still removed from the public CLI:** the internal OSA tooling `actions`, `blueprint`, `changes`, `drift`, `evidence`, `capabilities` and `command-overview`. Action authority is server-side, and `miosa --help` replaces `command-overview`.

Renamed commands:

| 1.x | 2.3 |
|---|---|
| `miosa sandbox create` | `miosa create` |
| `miosa sandbox exec` | `miosa exec` |
| `miosa sandbox stop` / `resume` | `miosa stop` / `miosa resume` |
| `miosa sandbox fork` | `miosa fork` |
| `miosa sandbox wait` | `miosa wait` |
| `miosa sandbox prompt` | `miosa prompt` |
| `miosa sandbox write-file` | `miosa files write` |
| `miosa sandbox dev` | `miosa console` |
| `miosa sandbox doctor` | `miosa doctor` |
| `miosa sandbox publish` | `miosa deploy` |
| `miosa ls` | `miosa list` |
| `miosa rm` | `miosa destroy` |
| `miosa cp` | `miosa files cp` |
| `miosa watch` | `miosa events` |
| `miosa context` | `miosa profile` |
| `miosa app link` / `promote` / `rollback` | `miosa deploy` (`promote`, `rollback`) |
| `miosa releases` | `miosa deploy releases` |
| `miosa rollback` | `miosa deploy rollback` |
| `miosa builds` | `miosa deploy builds` |
| `miosa deploy destroy` | `miosa deploy rm` |
| `miosa deploy domain` | `miosa deploy domains` / `miosa deploy domain-add` |
| `miosa deploy env` | `miosa deploy env` / `miosa deploy env-set` |
| `miosa databases` | `miosa db` |
| `miosa api-keys` | `miosa api-key` |
| `miosa checkpoints` | `miosa checkpoint` |
| `miosa templates` | `miosa template` |
| `miosa schedules` / `miosa cron` | `miosa schedule` |
| `miosa connectors` | `miosa connections` |
| `miosa workspaces` / `miosa teams` | `miosa workspace` |
| `miosa devices` | `miosa device` |
| `miosa opencomputers` | `miosa host` |
| `miosa containers` | `miosa host container` |
| `miosa meshes` | `miosa host mesh` |
| `miosa gha-runners` | `miosa host gha-runner` |
| `miosa backups` | `miosa host backup` (snapshots under `miosa host backup snapshot`) |
| `miosa tunnel open` | `miosa host tunnel` |
| `miosa mcp ...` | `miosa mcp ...` (restored in 2.1) |
| `miosa capabilities` | `miosa catalog` |
| `miosa command-overview` | `miosa --help` |
| `miosa actions ...` | removed (authority is server-side) |

## Update

```bash
miosa update            # download and replace the binary
miosa update --check    # report only
miosa version
```

`miosa update` verifies the SHA-256 of the download against the release `checksums.txt`, and refuses to replace a Homebrew-managed binary (use `brew upgrade` there). A one-line notice appears at most once a day when a newer version exists; silence it with `--no-update-notice` or `MIOSA_NO_UPDATE_NOTICE=1`.

## Shell completion

```bash
miosa completion zsh      # bash, fish, powershell
```

Sandbox names, profiles, templates and agents complete from the live API.

## 3. Sign in

```bash
miosa login               # opens the browser, device-code sign in
miosa whoami              # organization, plan, credits, and the scopes your key holds
```

`miosa whoami` shows the user, organization, workspace, plan and the scopes the key holds.

### Sign in with a key

`miosa login --with-key` asks for a key with hidden input, and `--key-stdin` reads it from stdin. A key is never accepted as an argument.

```bash
miosa login --with-key
echo "$MIOSA_KEY" | miosa login --key-stdin
```

### Credential resolution

Credentials resolve in this order:

1. `--api-key`
2. `MIOSA_API_KEY`
3. the active profile in `~/.miosa/config.toml`

> **Tip**
>
> CI and other non-interactive environments use `MIOSA_API_KEY`. See [Config and profiles](/docs/cli/config) for multi-account setups.

## Sign out

```bash
miosa logout
```

Removes the stored key of the active profile. `miosa logout --all` deletes the config file for every profile.

## Diagnose

```bash
miosa doctor
```

Checks configuration, connectivity and authentication, and prints a fix for each failure.

- [Config and profiles](/docs/cli/config): Named sign-ins, api_url, workspace defaults.
- [Raw API access](/docs/cli/api): miosa api for anything the high-level commands do not cover.
