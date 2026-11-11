---
slug: "services/logs"
title: "Services logs"
description: "Read a service's runtime logs per replica, the build log of a release, and the events that led to its state."
section: "Services & projects"
order: 1016
---

The service panel has a **Logs** tab with three views.

| View | Shows |
|---|---|
| Runtime | The newest lines each replica printed, with the replica's state. |
| Build | The log of the build that produced a release, with whether it finished. |
| Events | The state transitions of a release and the reason when it failed. |

A service that runs a container image has no build, so its Build view is empty.

## Runtime logs

Runtime logs are per replica.
Each replica reports its state and its lines, or an error when its log could not be read.
Each replica is listed separately, and the `replica` parameter reads one.

```text
GET /api/v1/projects/:id/services/:sid/logs?tail=200&replica=<id>
```

## Build logs

```text
GET /api/v1/projects/:id/services/:sid/builds/latest/logs?tail=200
GET /api/v1/projects/:id/services/:sid/builds/:build_id/logs?after=<offset>
```

The response carries the text, the offset to continue from, and `complete` once the build has finished.
Ask again with `after` set to the last offset to follow a running build.

## Events and failure reasons

```bash
miosa service logs shop API
```

The CLI prints the newest release's state transitions and its failure reason.
It does not stream runtime output.

## Where to go next

- [Releases and rollback](/docs/services/releases): the release states the events describe.
- [Replicas and scaling](/docs/services/replicas): what a replica is.
