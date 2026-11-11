---
slug: "services/releases"
title: "Services releases"
description: "Deploy a service, follow the release state machine, read a failure reason, promote or roll back, and cancel a release."
section: "Services & projects"
order: 1012
---

A release is one build-and-serve attempt of a service.
It is immutable: it freezes the source, the settings and the built image it ran with.

## Deploy

```bash
miosa service deploy shop API                        # rebuild the service's own source
miosa service deploy shop API --dir . --wait         # upload a directory
miosa service deploy shop API --repo https://github.com/acme/api --branch main
miosa service deploy shop API --image ghcr.io/acme/api:1.4 --port 8080
```

Every release takes an `Idempotency-Key`; the CLI sends one so a retried command returns the first release instead of starting a second.

`--wait` follows the release and exits non-zero if it fails, printing the reason.

## The state machine

| State | Meaning |
|---|---|
| `queued` | Accepted, waiting for a worker. |
| `building` | The source is being built. |
| `built` | The image exists; the release is about to start replicas. |
| `pre_deploy` | `pre_deploy_command` is running in the new replica. |
| `starting` | Replicas are booting. |
| `health_check` | The health path is being probed. |
| `active` | Serving. |
| `sleeping` | Idle sleep stopped the replicas; the next request wakes them. |
| `draining` | An earlier release is stopping. |
| `crashed` | A replica crashed and the restart policy did not recover it. |
| `failed` | Terminal failure, with a reason. |
| `canceled` | Canceled by a user. |
| `stopped` | Stopped by `stop`. |

A release that never leaves a state before its deadline is settled by the reaper with `deadline_exceeded`.

## Read a failure

```bash
miosa service releases shop API
miosa service logs shop API
miosa service logs shop API --release 6
```

`logs` prints every transition and the failure reason of the newest release, or of the release you name.
The release events carry the reason a release failed.
The build log and each replica's runtime lines are in the console's Logs tab; see [Logs](/docs/services/logs). The CLI does not stream them.

What a running replica printed is a separate read:

```text
GET /projects/:id/services/:sid/logs                    # the active release
GET /projects/:id/services/:sid/deployments/:did/logs   # one release
```

Each returns the last lines of every replica's output, with your secret values masked.
`tail` sets the number of lines (1 to 1000, default 200) and `replica` narrows it to one replica.
A replica that cannot be read says so in its own entry and does not hide the others.
It is a snapshot, not a stream.

Common reason codes:

| Code | What to do |
|---|---|
| `build_failed` | Read the build output; the command exited non-zero. |
| `clone_failed` | The repository or branch was not reachable, or the credential is wrong. |
| `image_pull_failed` | The image reference is wrong, or the registry secret is missing. |
| `healthcheck_timeout` | The app did not answer its health path in time. Check `start_command` and the port. |
| `pre_deploy_failed` | `pre_deploy_command` exited non-zero; nothing serves. |
| `no_capacity` | No host had room for the shape in that region. Try another region or a smaller `limits`. |
| `reference_missing` | A variable references something that does not exist. `miosa service vars effective` lists them. |
| `reference_cycle` | Two variables reference each other. |
| `deadline_exceeded` | The release sat in one state too long and the reaper settled it. |
| `quota_exceeded` | A plan limit was reached. |

## Promote and roll back

Rollback serves an earlier release's built image with its frozen source and settings.
Nothing is rebuilt, so it is fast.

```bash
miosa service rollback shop API 3 --yes
miosa service promote  shop API 3 --yes   # the same call
```

The Services API has one operation for this and calls it rollback.
The CLI accepts `promote` as its name for "serve this release again"; there is no separate promote route.

## Cancel and remove

```bash
miosa service cancel shop API 7     # a queued or building release
miosa service release shop API 3    # one release
```

A release that already finished can be removed from the history; one that is still running cannot.

## Stop, start, restart

```bash
miosa service stop    shop API --yes   # stop every active release now
miosa service start   shop API         # start a stopped service
miosa service restart shop API         # queue a release that restarts it
```

`stop` fails with `SERVICES_NOT_STOPPED` if a replica cannot be stopped, so a service is never reported stopped while it still runs.
`start` on a service that is already running or starting is `SERVICE_CONFLICT`.

## Rollout and rollback of several services

Each service releases independently.
For a coordinated change, apply a template or a change set, which stages every service's change and deploys them together.
