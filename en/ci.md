---
slug: "ci"
title: "CI on MIOSA"
description: "Run GitHub Actions jobs on MIOSA. Connect the MIOSA CI app, use runs-on miosa in a workflow, and each job runs in its own sandbox, billed per second."
section: "Services & projects"
order: 1040
---

CI on MIOSA runs your GitHub Actions jobs in MIOSA sandboxes.
You connect the MIOSA CI GitHub app, give it your repositories, change `runs-on: ubuntu-latest` to `runs-on: miosa` in a workflow and push.
Each job then runs in a new sandbox, billed per second, and the sandbox is deleted when the job ends.

The same state is in the console under **CI on MIOSA**, in the `miosa ci` commands and at `GET /api/v1/ci`.

## Set up

There are four steps.
The console page and `miosa ci status` show which are done.

1. **Install the MIOSA CI app on GitHub.**
   In the console, open **CI on MIOSA** and select **Connect GitHub**.
   From a terminal, run `miosa ci connect`; it opens the same link and waits until the connection appears.
2. **Give it access to your repositories.**
   On GitHub, choose all repositories or the ones that should run on MIOSA.
   You can change this later in the app's settings on GitHub.
3. **Use `runs-on: miosa` in a workflow.**
4. **A job passes on MIOSA.**

One GitHub account belongs to one MIOSA organization at a time.
If it is already connected to another organization, disconnect it there first.

The link GitHub returns you through is valid for 30 minutes.
If it expires, select **Connect GitHub** again.

## Change the workflow

Set the label of the job to `miosa`:

```yaml
jobs:
  test:
    runs-on: miosa
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

The label must be exactly `miosa`.
Jobs with other labels keep running where they ran before.
You can move one job at a time.

## Isolation

A CI sandbox is created with none of your organization's MIOSA environment variables, secret files, database links, Git keys or model keys.
It has only what the runner image contains and what GitHub sends the runner.

Keep job secrets in GitHub Actions secrets and pass them to a step the way you do today:

```yaml
- run: ./deploy.sh
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
```

Each job gets its own sandbox.
Nothing carries over from one job to the next.

> **Note: Public repositories**
>
> MIOSA refuses jobs from public repositories by default.
> Anyone can open a pull request on a public repository, and the job it starts would run on your organization's bill.
> The job is shown in **Recent jobs** with the reason.

## Billing

A CI sandbox is billed per second, like any other sandbox, from when it is created until it is deleted.
The bill shows this usage as **CI**, separate from your other sandboxes.

## Troubleshooting

### The job stays "Queued" on GitHub

- The workflow must say exactly `runs-on: miosa`.
- The MIOSA CI app must have access to that repository.
  Add it in the app's settings on GitHub.
- Run `miosa ci status` to see which setup step is not done, and `miosa ci jobs` to see whether the job reached MIOSA.
- A public repository is refused by default.

### The repository is public

Public repositories are refused unless MIOSA has allowed them for your organization.
Run public repositories on GitHub's own runners, or ask support.

### A step cannot see a MIOSA environment variable

That is intended.
Put the value in a GitHub Actions secret and pass it to the step with `env:`.

### Jobs start slowly or wait

Every job gets a new sandbox, and the runner registers in it before the first step starts.
In **Recent jobs**, **Queued** means MIOSA has not started the sandbox yet and **Starting** means it is being prepared.
If a job stays in either state or fails, the row shows the reason.
A sandbox limit or an empty credit balance on the organization can stop new sandboxes.

### A step needs a tool that `ubuntu-latest` has

The runner image has the GitHub Actions runner and the common tools from `ubuntu-latest`, not every one.
Install what is missing in a step, for languages with `actions/setup-node`, `actions/setup-python` or `actions/setup-go`.

### Debug a failing job

Open the run from the link in **Recent jobs**; the step logs are on GitHub.
The sandbox is deleted when the job ends, so there is nothing to open afterwards.
On GitHub, choose **Re-run jobs** and enable debug logging for more output.
`miosa ci jobs --json` gives the status, conclusion, duration and failure reason of each job.

## CLI

| Command | What it does |
|---|---|
| `miosa ci connect` | Opens the GitHub install link and waits until the connection appears |
| `miosa ci status` | Shows the four setup steps and the isolation note |
| `miosa ci jobs` | Lists recent jobs |
| `miosa ci disconnect <installation>` | Disconnects a GitHub account; running jobs finish |

Every command takes `--json`.

## API

| Request | What it does |
|---|---|
| `GET /api/v1/ci` | Setup state: `available`, connected installations, the four `steps`, the `last_job` and the `isolation` note |
| `POST /api/v1/ci/connect-intents` | Returns the GitHub `install_url`, valid for 30 minutes |
| `GET /api/v1/ci/jobs?limit=20` | Recent jobs with `status`, `conclusion`, `duration_seconds`, `run_url` and `failure_reason` |
| `DELETE /api/v1/ci/installations/:installation_id` | Disconnects a GitHub account; running jobs finish |

A job `status` is `queued`, `starting`, `running`, `completed`, `failed` or `cancelled`.
Errors use the platform error shape.
The codes are `CI_NOT_AVAILABLE`, `CI_INSTALLATION_NOT_FOUND`, `CI_INSTALLATION_OWNED_ELSEWHERE` and `CI_STATE_INVALID`.

```bash
curl -s https://api.miosa.ai/api/v1/ci \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```
