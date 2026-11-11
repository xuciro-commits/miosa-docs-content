---
slug: "services/config-file"
title: "Service settings as code"
description: "Put a service's settings in a file at the repository root, so the repository says how the service runs and the console shows where each value came from."
section: "Services & projects"
order: 1008
---

A service can carry its settings in a file at the repository root. The same file
works for Docker Host and for Services:

- `miosa.app.yml`, `miosa.app.yaml` or `miosa.app.json`, and
- `miosa.json`.

`railway.json` is read as a compatible fallback, so a project imported from
Railway needs no edit first.

## Rules

**Only the fields the file names change.** A file with one key leaves every other
setting as the project has it. A file is an addition, not a replacement.

**The API still validates.** The merged settings go through the same validation
as a console save, so a file cannot write a value the API would refuse.

**The file wins over the console for the fields it names.** The console shows
which fields came from the file.

## Fields

| File key | Setting | Applies to |
|---|---|---|
| `deploy.start` / `startCommand` / `start_command` | Start command | all |
| `deploy.preDeploy` / `preDeployCommand` | Pre-deploy command | all |
| `deploy.healthcheckPath` / `healthcheck_path` | Health check path | web services |
| `deploy.healthcheckTimeout` | Health check timeout, seconds | web services |
| `deploy.restartPolicy` / `restartPolicyType` | `always`, `on_failure` or `never` | all |
| `deploy.maxRetries` / `restartPolicyMaxRetries` | Retries before giving up | all |
| `deploy.replicas` / `numReplicas` | Replicas | all |
| `deploy.cron` / `cronSchedule` | Five-field schedule | cron services |
| `deploy.region` | `us-east`, `us-west` or `us-mia` | all |
| `deploy.sleep` / `sleepApplication` | Sleep when idle, five minutes by default | web services with a public address |
| `build_command` | Build command, Railpack builds only | static services |
| `output_dir` | Directory to serve, relative to the app | static services |

`variables` are read as names; values are not taken from the file.

## What the file cannot carry

Keys with no equivalent are reported in the plan and in the console rather than
dropped quietly: `build.builder`, `build.dockerfilePath`, and anything unknown.

> **Note: Where the value came from**
>
> Every field the file sets is recorded against the file and key that set it, so
> "why is the start command this?" has an answer months later.
