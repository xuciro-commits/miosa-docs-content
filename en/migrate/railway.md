---
slug: "migrate/railway"
title: "Moving from Railway"
description: "Bring a Railway project to MIOSA with one reviewed plan, every environment in its own Deploy target, services created stopped, and an exact list of what carries over and what you finish by hand."
section: "Services & projects"
order: 1013
---

`miosa import railway` reads a Railway project through Railway's public API and writes a plan.
The plan lists the MIOSA project, one Deploy target per Railway environment, and in each target the services, databases, variable names and shared variables it would create.
It also lists everything it cannot carry across.
Nothing is created until you review the plan and apply it, and an apply starts nothing unless you ask.

This page is the reference for what carries over, what does not, and what to do about the secrets.

## Before you start

- A Railway token: an account, workspace or project token from Railway's account settings.
  `--project` takes the project's id, or its name when the login can list it; if Railway refuses, the import prints Railway's message and asks for the id.
  MIOSA reads it, never stores it, and never writes it to the plan.
  If you are logged in with the Railway CLI, `--via railway-cli` reads through that session instead, so no token passes through MIOSA.
- The MIOSA CLI, `@miosa/cli` 2.13.4 or newer:

```bash
npm install -g @miosa/cli
miosa --version
```

- GitHub connected to MIOSA, when your services build from GitHub repositories:

```bash
miosa connections add apps github
```

- Assume every secret in the Railway project is compromised.
  Railway disclosed on 2026-10-21 that its secrets were exposed, so the import copies names, not values, unless you explicitly ask otherwise.

## The steps

```bash
export RAILWAY_TOKEN=...                                          # or --token-file, or a prompt
miosa import railway --project acme-api --environment all         # review only; nothing is created
miosa import railway --project acme-api --environment all --out plan.json
miosa import railway --apply --plan plan.json --region us-east    # creates everything, stopped
miosa service vars set acme-api api STRIPE_KEY=<new value> --deploy-target production
miosa service deploy acme-api api --deploy-target production
```

`--environment` takes one environment, several (`production,staging`), or `all`.
The default is `production`.

The review prints what will be created in each Deploy target, where each setting came from, which variables need a value, and a "Finish by hand" list with the next step for each item.
Every field that cannot be carried appears there; nothing is dropped silently.

## What apply does

- It creates the project, then one Deploy target per Railway environment, reusing a target that already exists.
- It creates every service in the Deploy target of its environment, so `production` and `development` keep separate services, variables and settings.
- It creates every service **stopped**.
  Nothing builds or runs until you deploy it with `miosa service deploy`, or apply with `--deploy`.
- `--deploy` starts every service that has a source, after the GitHub copies are ready.
  It refuses while any variable has no value, so a service never starts with an empty secret.
- It does **not** attach custom domains unless you pass `--attach-domains`.
  Attaching a live domain during the import would move its traffic before the MIOSA services run.
  Without the flag, the review lists each domain as skipped and the summary prints the command that attaches it later.
- It links a GitHub repository, private or public, through your GitHub connection.
  MIOSA copies the repository into Forge and builds from the copy, so no credential is stored on the service.
- It checks everything before creating anything.
  If GitHub is not connected, it stops with `Connect GitHub first: miosa connections add apps github`.
  If your connection cannot read a repository, it names the repository and stops.
  If your organization cannot build services from a repository yet, it says so and stops.
  In each case nothing is created.
- Running apply again creates only what is missing: every project, Deploy target and service is looked up first.
- It gives every service a name and slug MIOSA accepts.
  A package name such as `@acme/web` becomes `acme-web`, and references to it follow.
  A slug longer than 40 characters is shortened with a short hash of the name, the same on every run.

## What carries over

| Railway | MIOSA |
|---|---|
| Project | Project |
| Environment | A Deploy target of the same name, with its own services and variables |
| Service | Web service, Worker or Schedule, in its environment's Deploy target |
| Repository, branch, root directory | The service source; a GitHub repository is linked through your GitHub connection |
| Dockerfile path | `source.dockerfile`, relative to the root directory |
| Build command | `source.build_command` |
| Start command | `start_command` |
| Pre-deploy command (for example `alembic upgrade head`, `drizzle-kit migrate`) | `pre_deploy_command` |
| Health check path and timeout | `healthcheck_path` and `healthcheck_timeout_s` |
| Restart policy and retries | `restart_policy` and `max_retries` |
| Replicas | `replicas` (1 to 50) |
| Cron schedule | A Schedule service |
| `railway.json`, `railway.toml` | Read from the repository through Railway; the file wins over the service settings, as on Railway |
| Variables | Names, build-visible as on Railway; `${{Service.VAR}}` references, rewritten where MIOSA's names differ |
| Shared variables | Shared variables of the same Deploy target (names) |
| Postgres, Redis, MySQL template images and plugins | A database service with a managed database |
| Domain target port | The service port, which the health check and the public address use |
| Custom domains | Listed; attached with `--attach-domains`, and served from MIOSA once their DNS points here |

Each setting is taken from the first place that has it: the repository's config file, then the service settings, then what the latest deployment ran.
The plan records which one, for every setting.

### References

MIOSA uses the same `${{Service.NAME}}` reference syntax as Railway.
The import rewrites the names that differ:

| Railway reference | Becomes |
|---|---|
| `${{Redis.REDISHOST}}`, `REDISPORT`, `REDISPASSWORD` | `${{Redis.REDIS_HOST}}`, `REDIS_PORT`, `REDIS_PASSWORD` |
| `${{Redis.REDISUSER}}` | `default` |
| `${{Redis.REDIS_URL}}` | unchanged |
| `${{Postgres.POSTGRES_USER}}`, `POSTGRES_PASSWORD`, `POSTGRES_DB` | `${{Postgres.PGUSER}}`, `PGPASSWORD`, `PGDATABASE` |
| `${{Postgres.DATABASE_URL}}`, `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE` | unchanged |
| `${{api.RAILWAY_PRIVATE_DOMAIN}}` | `${{api.MIOSA_PRIVATE_DOMAIN}}` (`api.miosa.internal`) |
| `RAILWAY_SERVICE_NAME`, `RAILWAY_SERVICE_ID`, `RAILWAY_ENVIRONMENT_NAME`, `RAILWAY_ENVIRONMENT_ID`, `RAILWAY_PROJECT_ID`, `RAILWAY_PROJECT_NAME`, `RAILWAY_REGION` | the `MIOSA_` name |

A service is unreachable by its private domain on MIOSA today, because private networking between services is off.
The review flags every variable that reaches another service that way; point it at an address MIOSA can reach (for SMTP, your mail provider).

A reference with no MIOSA equivalent, such as `RAILWAY_PUBLIC_DOMAIN`, `REDIS_PUBLIC_URL` or `DATABASE_PUBLIC_URL`, is listed in the review.
That variable is created without a value, and you set it after the import.

Railway's own variables (`RAILWAY_*`) are not imported; MIOSA injects its own `MIOSA_*` names.

### Build-visible variables

Railway exposes every service variable to the build, so a framework inlines `NEXT_PUBLIC_*`, `VITE_*` and `PUBLIC_*` values when it builds.
The import keeps that: every imported variable and shared variable is created build-visible, and the review says how many.
`MIOSA_*` and `RAILPACK_*` names stay runtime only, because MIOSA reserves them.
A variable created without a value becomes build-visible when you set it with `--build`; the summary prints that command.

On MIOSA a build-visible variable reaches the build as a BuildKit secret, so it is never stored in an image layer.
The exception is a Dockerfile that declares an `ARG` of the same name: that variable is passed as a build argument, as on Railway, and a `RUN` step that uses it is recorded in the image history.

To create every variable runtime only, pass `--runtime-only-vars` when you write the plan.
You can change one later:

```bash
miosa service vars set <project> <service> NEXT_PUBLIC_API_URL=https://api.example.com --build
miosa service vars set <project> <service> STRIPE_KEY=<new value> --no-build
```

## What does not carry over

| Railway | What to do |
|---|---|
| Variable values | Set new values on MIOSA; see below |
| Region outside the US (for example `asia-southeast1`, `europe-west4`) | MIOSA runs in `us-east`, `us-west` and `us-mia`; the review lists every such service and apply needs `--region` |
| Builder (Nixpacks) | MIOSA builds the Dockerfile if the repository has one, else detects the stack with Railpack; check the first build |
| Watch patterns | MIOSA builds on every push; the review lists them |
| Sleep on idle | Not carried; set the `sleep` setting after the import |
| Database data and volume contents | Not copied; the review lists each volume and its size, and you move the data |
| Service volumes | Listed with the `miosa volume create` command and the mount path to attach |
| `*.up.railway.app` domains | Railway-owned DNS; MIOSA assigns `<label>.<org>.miosa.app` |
| A service uploaded with the Railway CLI (no repository or image) | Created without a source; upload the code with `miosa service deploy <project> <service> --dir <path>` |
| MongoDB plugins | Create a MongoDB database with `miosa db create --engine mongodb` |

## Rotate the secrets

Apply refuses to run while a variable has no value, unless you pass `--allow-missing-values`, which creates those variables empty.
Services stay stopped either way, so nothing starts with a missing or leaked secret.

1. Assume every secret in the Railway project is compromised.
2. Do not copy values during the import; set new ones on MIOSA.
3. Rotate at the provider first (database password, Stripe, SMTP, API keys), then paste the new value into MIOSA.
4. Deploy after the new values are set.
5. Delete the old Railway project once the MIOSA one serves traffic.

```bash
miosa service vars set <project> <service> STRIPE_KEY=<new value> --deploy-target production
miosa project vars set <project> APP_NAME=<value> --deploy-target production
miosa service deploy <project> <service> --deploy-target production
```

`--copy-variables` exists for a test project where nothing is sensitive.
It prints the same warning, and it is never the default.

## If you only have a config file

`--from-file` reads a `railway.json` or `railway.toml` locally, with no token and no network call:

```bash
miosa import railway --from-file ./railway.toml --service-name api
```

The plan has no source, because a config file has none; add one to the plan before you apply.

## Undo

Every id the apply creates is recorded as it is created.

```bash
miosa import railway --cleanup --state .miosa/import-railway-acme-api.json --yes
```

`--cleanup` prints exactly what it will delete first, and deletes only those ids.
An interrupted apply continues with `--resume`.

## Railway terms, MIOSA names

| Railway says | MIOSA calls it |
|---|---|
| Project | Project |
| Environment | Deploy target |
| Service | Service (Web service, Worker, Private service, Schedule) |
| Plugin, database template | Database service |
| Deploy | Release |
| Variables | Variables |
| Shared variables | Shared variables |
| Volume | Volume |
| Domain | Public address or custom domain |
