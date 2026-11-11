---
slug: "opencomputers/hosts"
title: "Hosts and capabilities"
description: "Inspect enrolled hosts, read their runtime and services, run commands on one host or a group, manage files, apps, compose projects, secrets, SSH keys, and clusters."
section: "Computers"
order: 3015
---

A host is one enrolled machine.
Every host is addressed by its host ID, and every host command takes `<host>` as its first argument.
The host ID is stable; the machine's IP address and hostname are not the identity MIOSA uses.

```bash
miosa host list --tag studio
miosa host get <host>
```

`miosa host list` takes `--tag` to narrow to hosts carrying a tag.
`miosa host get <host>` returns the host record with its platform, region, tags, and reported capabilities.
Aliases: `miosa hosts`, `miosa host`.

## What a host reports

| Command | What it tells you |
|---|---|
| `miosa host get <host>` | The host record: state, platform, region, tags, capabilities |
| `miosa host metrics <host>` | CPU, memory, and disk of the host |
| `miosa host runtime <host>` | The container runtime the host is running |
| `miosa host services <host>` | The services running on the host |
| `miosa host jobs <host>` | Recent commands dispatched to the host |

Registration alone proves neither readiness nor feature parity.
Treat a fresh heartbeat plus a completed operation as the evidence that a host is usable for the capability you need.

> **Caution: Check the capability, not the interface**
>
> A common MIOSA interface does not mean every host supports every managed operation.
> Direct execution runs within the host user's operating-system permissions; it does not automatically place each command in an isolated microVM.
> Registration does not turn an arbitrary host into a sandbox worker pool or an Docker Host host.

## Run a command

Commands dispatched to a host are jobs.

```bash
miosa host job list <host>
miosa host job get <host> <job>
miosa host job cancel <host> <job>
```

`miosa host job` has aliases `jobs` and `exec`, so `miosa host exec <host> -- uname -a` is the same surface.
Cancelling a job is not proof that the remote process stopped; confirm the outcome before retrying work with side effects.

The REST equivalent is the [exec endpoint](/docs/api-reference/open-computers#run-a-job), which streams SSE by default and returns collected output when you send `"stream": false`.

### Run on every host in a group

A group is a named set of hosts. Running a command or installing an app on a group runs it on every member.

```bash
miosa host group new ci --description "CI workers"
miosa host group add ci <host>
miosa host group exec ci --cmd npm --arg test --cwd /srv/app --timeout-ms 120000
miosa host group get ci
miosa host group remove ci <host>
miosa host group rm ci --yes
```

Deleting a group leaves its hosts in place.

## Files on a host

Paths are on the host's own file system, not a sandbox workspace.

```bash
miosa host fs ls <host> --path /srv/app
miosa host fs stat <host> --path /srv/app/package.json
miosa host fs mkdir <host> --path /srv/app/logs
miosa host fs rm <host> --path /srv/app/old.log --yes
```

`miosa host fs ls` defaults to the agent user's home directory.
The same operations are available over REST under `/opencomputers/hosts/{id}/fs/*` - see the [OpenComputers API](/docs/api-reference/open-computers#file-system).

## Tags

Tags group hosts for selection and batch work.

```bash
miosa host tag add <host> gpu
miosa host tag list
miosa host tag rm <host> gpu
```

`miosa host list --tag gpu` then selects hosts without you tracking IDs by hand.

## Secrets

Host secrets are encrypted key-value pairs that the host agent can use at runtime.
Values are never returned by a list call.

```bash
miosa host secret list <host>
miosa host secret set <host> GITHUB_TOKEN --description "CI token"
miosa host secret set <host> DEPLOY_KEY --key-stdin < deploy.key
miosa host secret rm <host> GITHUB_TOKEN --yes
```

`secret set` prompts for the value or reads it from standard input with `--key-stdin`, so the value does not land in your shell history.

## SSH keys

Only public keys are handled.

```bash
miosa host ssh-key add <host> --name roberto --public-key "ssh-ed25519 AAAA..."
miosa host ssh-key import <host> <github-user>
miosa host ssh-key list <host>
miosa host ssh-key rm <host> <key> --yes
```

`ssh-key import` copies the public keys a GitHub user has published.

## Apps

The catalog lists what MIOSA can install; install puts one on a host.

```bash
miosa host app catalog
miosa host app install <host> redis
miosa host app list <host>
miosa host app health <host> redis
miosa host app start <host> redis
miosa host app uninstall <host> redis --yes
```

Check `miosa host app health` after an install rather than assuming the app came up.

## Compose projects

A compose project is a docker-compose file the host keeps running.

```bash
miosa host compose list <host>
miosa host compose up <host> app --build --pull
miosa host compose down <host> app --remove-volumes
miosa host compose rm <host> app --yes
```

`--build` builds images first and `--pull` pulls them first.
`--remove-volumes` deletes the project's volumes, so confirm what you intend to lose before using it on a stateful project.

## Containers

Containers on a host are covered in [Containers](/docs/opencomputers/containers).

## Clusters

A cluster is a group of your hosts that serve one model together, exposed as an OpenAI-compatible endpoint.

```bash
miosa host cluster list
miosa host cluster get <cluster>
miosa host cluster start <cluster>
miosa host cluster stop <cluster>
miosa host cluster rm <cluster> --yes
```

Clusters are created over the API with `host_ids`, `name`, and `model`; see [Inference clusters](/docs/api-reference/open-computers#inference-clusters).

## Remove a host

```bash
miosa host rm <host> --yes
```

Removing a host ends its ability to authenticate.
It does not erase the machine's files and does not prove that its processes stopped.

- [Containers](/docs/opencomputers/containers): Run images on a host and stream logs and stats.

- [Security model](/docs/opencomputers/security): What the host agent can reach, and what MIOSA can see.

- [OpenComputers API](/docs/api-reference/open-computers): Hosts, jobs, files, tunnels, agents, clusters, and secrets over REST.
