---
slug: "docker-host/deploy-from-github"
title: "Deploy from GitHub"
description: "Build and publish a GitHub repository with miosa deploy create --git - connect GitHub, public and private repositories, the branch, logs, redeploys, and the webhook that rebuilds on every push."
section: "Services & projects"
order: 1026
---

`miosa deploy create --git` clones a repository, builds the branch you name, and publishes it at a stable URL.
The same deployment rebuilds when you push, once you add its webhook to the repository.

## Connect GitHub

A public repository needs no connection.
A private repository is cloned with the GitHub connection of the person who creates the deployment, so connect GitHub first:

```bash
miosa connections add apps github
```

The command opens GitHub's consent page and waits until you approve.
In the console the same connection is under **Connections**, **Apps**.

GitHub shows MIOSA only the organizations that approved it.
If an organization's repositories are missing, grant access on GitHub under **Settings**, **Applications**, **Authorized OAuth Apps**, or ask an organization owner to approve MIOSA.

## Deploy a repository

```bash
miosa deploy create --git https://github.com/octocat/Spoon-Knife --branch main
```

The command waits until the deployment is live and prints its URL, for example `https://spoon-knife.<your-org>.miosa.app`.
Pass `--name` to choose the deployment name; it defaults to the repository name.
Pass `--no-wait` to return as soon as the deployment is created.

Name the branch with `--branch`.
The deployment builds that branch now and on every push to it.

A private repository works the same way once GitHub is connected.
Without the connection, or with a branch that does not exist, the build fails at the clone step:

```text
publishing failed at build: The source could not be cloned (git exited 128). (SOURCE_CLONE_FAILED)
```

Connect GitHub, check the branch name, and run `miosa deploy redeploy <name>`.

## Logs and builds

```bash
miosa deploy builds my-app     # every build and its state
miosa deploy logs my-app       # why the last build failed, and the app's output
```

A static site has no runtime log: it serves the built files and runs no process.

## Redeploy

```bash
miosa deploy redeploy my-app
```

Redeploy builds the branch again from GitHub and releases it.
Use it after you fix a failed build, or to pick up a change without a push.

## Rebuild on every push

When `deploy create --git` creates the deployment, it prints a webhook secret once:

```text
Add this GitHub webhook so pushes redeploy (the secret is shown once):
  URL:          https://api.miosa.ai/api/v1/integrations/github/webhook
  Content type: application/json
  Secret:       <webhook secret>
  Events:       Just the push event
```

Add it to the repository under **Settings**, **Webhooks**, **Add webhook**, with exactly those values.
GitHub signs every delivery with the secret.
MIOSA checks the signature, finds the deployment for that repository and branch, and builds the pushed commit.
A delivery with no signature is refused with 401, and one with a wrong signature with 403.
Pushes to other branches are ignored.

The secret is also the `webhook_secret` field of the create response (`--json`).
Store it like a password: anyone with it can trigger a build.
To replace it, call `POST /api/v1/deployments/{id}/webhook-secret` and update the webhook on GitHub with the new value.

## Related

- [Publishing](/docs/docker-host/publishing) explains how a build becomes a version.
- [Domains](/docs/docker-host/domains) attaches your own domain to the deployment.
- [Rollback](/docs/docker-host/rollback) makes an earlier version live again.
