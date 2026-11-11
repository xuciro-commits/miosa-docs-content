---
slug: "opencomputers/gha-runners"
title: "GitHub Actions runners"
description: "Register self-hosted GitHub Actions runners on your own hosts for one repository, and stream their activity from MIOSA."
section: "Computers"
order: 3018
---

A runner is a GitHub Actions runner the host keeps registered for one repository.
MIOSA installs and supervises the runner process on your machine, so a workflow job runs on hardware you control.

Use this when a build needs a GPU, a large local cache, software that only exists on your machine, or a network path that GitHub-hosted runners cannot reach.

## Register a runner

**miosa CLI**

```bash
miosa host gha-runner list <host>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token <token>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token-secret GH_TOKEN --ephemeral
miosa host gha-runner get <host> <runner>
```

Aliases: `miosa host gha-runners`, `miosa host runner`, `miosa host runners`, and `create` or `add` for `new`.

**REST API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/gha-runners" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"repo_url":"https://github.com/acme/app","name":"gpu-builder","labels":["gpu","linux"],"pat_secret_id":"<secret-id>"}'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `repo_url` | string | Yes | Repository URL; must be on `github.com` or a `*.github.com` host |
| `name` | string | Yes | Runner name, 1 to 128 characters; unique per host |
| `labels` | string[] | No | Runner labels; default `[]` |
| `ephemeral` | boolean | No | Deregister the runner after one job; default `false` |
| `pat_secret_id` | string | One of | ID of an existing host secret holding a GitHub token |
| `pat_value` | string | One of | Inline token; MIOSA stores it as a host secret for you |

The CLI flag names differ from the API field names: `--repo` maps to `repo_url`, `--token` to `pat_value` (sent to the host and not stored), and `--token-secret` to `pat_secret_id`.
`--label` is repeatable, and `--name` defaults to a generated name.

> **Caution: MIOSA never returns the token**
>
> The GitHub token is not stored in a readable form and is never returned by any read.
> Prefer `pat_secret_id` or `--token-secret` so the value lives in a [host secret](/docs/opencomputers/hosts#secrets) rather than in a command you might paste elsewhere.

> **Note: GitHub Enterprise Server is not supported**
>
> `repo_url` must point at `github.com`. A GHES URL is rejected with a `422`.

## Runner states

| State | Meaning |
|---|---|
| `provisioning` | MIOSA is installing and starting the runner |
| `idle` | Registered and waiting for a job |
| `running_job` | Executing a workflow job |
| `offline` | Registered but not reachable |
| `failed` | Installation or registration failed |
| `removed` | Deregistered |

`current_job` is set while the runner is executing, and `last_activity_at` tells you when it last did anything.

## Watch a runner

**miosa CLI**

```bash
miosa host gha-runner events <host> <runner>
miosa host gha-runner events <host> <runner> --json
```

Streams events as the runner picks up and finishes jobs. Ctrl-C stops the stream.

**REST API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/gha-runners/$RUNNER_ID/events" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Named SSE events: `progress`, `ready`, `status`, `error`, and `removed`.

## Repair and remove

```bash
miosa host gha-runner refresh <host> <runner>
miosa host gha-runner rm <host> <runner> --yes
```

| Action | REST | Notes |
|---|---|---|
| Refresh | `POST .../gha-runners/{runner_id}/refresh` | Reinstalls and restarts the runner; returns `{"ok": true}` |
| Remove | `DELETE .../gha-runners/{runner_id}` | Stops the runner and deregisters it from GitHub; returns `204` |

`refresh` is the fix for a runner stuck in `failed` or `offline` after an upgrade, rather than deleting and re-registering.
Both actions can return `429` if the GitHub API rate-limits the registration calls.

## Reference it in a workflow

Point a workflow job at the runner's labels:

```yaml
jobs:
  build:
    runs-on: [self-hosted, gpu, linux]
```

A job runs on your machine only while the host agent is connected and the runner is `idle`.
If the host goes offline mid-job, the workflow fails on GitHub's side; check `miosa host gha-runner get` and the [runner events](/docs/opencomputers/gha-runners#watch-a-runner) before retrying.
