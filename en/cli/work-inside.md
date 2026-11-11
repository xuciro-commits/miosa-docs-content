---
slug: "cli/work-inside"
title: "Run commands and move files"
description: "exec, ssh, console, files, background processes, services, logs, ports and public previews in a sandbox."
section: "CLI, SDKs & MCP"
order: 11004
---

## exec

```bash
miosa exec [name|id] -- <cmd> [args...] [flags]
```

Run a command inside a sandbox and print its output. `miosa` exits with the remote command's exit code, so it drops straight into scripts and CI. The `--` separator is required before the command.

```bash
miosa exec my-box -- echo hello
miosa exec -- ls -la /home                              # uses the current sandbox
miosa exec my-box --cwd my-repo -- npm run build
miosa exec my-box -- bash -c 'for i in 1 2 3; do echo $i; done'
```

`--cwd` sets the working directory, relative to the sandbox home directory (or absolute).
`exec` streams output until the command finishes.
For work that runs a long time, start it in the background so it keeps running after you disconnect.

A sandbox that is still starting refuses commands. `exec` retries with backoff until it is ready, up to 90 seconds.

## ssh

```bash
miosa ssh my-box
miosa ssh my-box -- bash -lc "cd app && npm test"
miosa ssh my-box -- bash -s < ./setup.sh
```

`miosa ssh` runs the system `ssh` through the authenticated tunnel with a one-time key and a short-lived certificate (15 minutes, valid for that sandbox only). Where the server has no certificate authority it authorizes the one-time key in the sandbox instead. The key, certificate and host file live in a private temporary directory removed when ssh ends.

Standard input is piped through, up to 80KB. `--api` runs the command over the MIOSA API instead, and so does a machine without `ssh`. There is no interactive login shell yet.

`ssh user@host` and `scp` against a sandbox directly do not work: there is no public SSH endpoint.

## console

```bash
miosa console my-box
```

An interactive shell in a sandbox using the miosa-terminal-v1 WebSocket relay, with a real PTY, resize handling and the remote exit code.

## files

```bash
miosa files [command]
```

Remote paths are written `<sandbox>:<path>`. The sandbox may be a name or id, and can be omitted for the current sandbox. Relative remote paths start in `/home/user`.

```bash
miosa files ls my-box:/workspace
miosa files cp ./app.py my-box:/workspace/app.py
miosa files cp -r ./src my-box:/workspace/
miosa files cp my-box:/workspace/out.json ./out.json
miosa files cat my-box:/workspace/out.json
cat config.yaml | miosa files write my-box:/workspace/config.yaml
```

| Command | What it does |
|---|---|
| `files ls [-l] <path>` | List a directory (`-l` shows modification times) |
| `files cp [-r] <src> <dst>` | Copy local to remote, remote to local, or remote to remote |
| `files cat <path>` | Print a file |
| `files write <path>` | Write stdin to a file, creating parent directories |
| `files mkdir <path>` | Create a directory and its parents |
| `files mv <from> <to>` | Move or rename (alias `rename`) |
| `files rm [-r] [-f] <path>...` | Delete files or directories |
| `files chmod <mode> <path>` | Change permissions (octal, for example `755`) |
| `files stat <path>` | Type, size, mode and modification time |

A directory needs `-r`. A path that does not exist is an error unless `-f` is given.

## Background processes

```bash
miosa process start --name build --cwd /workspace/app -- npm run build
miosa process list
miosa ps
miosa process logs <pid> --tail 50
miosa process kill <pid>
miosa process get <pid>
```

`miosa ps` is the same as `miosa process list`.

## Daemons

A sandbox daemon is a named long-lived process: start it with a name, then read
its output, restart it or stop it by that name.

```bash
miosa daemon create my-box --name web --command "npm run dev" --cwd /workspace/app
miosa daemon list my-box
miosa daemon logs my-box web
miosa daemon restart my-box web
miosa daemon stop my-box web
```

Named services belong to a Computer, where systemd owns them; `miosa services`
is the project Services command and does not take sandbox flags.

## Logs and events

```bash
miosa logs my-box -n 100 -f     # system log, follow
miosa events my-box             # stream lifecycle events
```

## Ports, previews and URLs

```bash
miosa ports my-box
miosa preview create my-box 3000 --visibility public
miosa preview list
miosa url my-box
```

`miosa preview` publishes a sandbox port at a public URL and is where visibility
is decided: `--visibility private|public`, `--ttl`, `--name`. `miosa url` prints
the URL a sandbox's endpoint answers on, and `miosa preview unshare` revokes a
public link.

## Proxy

```bash
miosa proxy my-box 5432:5432
```

Forwards local ports to sandbox ports over a WebSocket tunnel (any TCP service; the key needs `sandboxes:exec`).

## Desktop

```bash
miosa desktop start my-box
miosa desktop url my-box
```

> **Note**
>
> Running a command over the API and over ssh have the same exit-code behaviour: both exit with the remote command's status.
