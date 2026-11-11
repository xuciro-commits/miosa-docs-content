---
slug: "forge/faq"
title: "Forge limits and FAQ"
description: "What Forge does and does not do yet: visibility, permissions, merge methods, issues, repository secrets and CI."
section: "Services & projects"
order: 1039
---

## Are repositories private?

Yes, by default.
A repository is `private`, `internal` or `public`, set with `repo create --visibility` or changed later with `repo update`.
Members of your organization get access through their organization role, and `grant set` gives one person an extra role on one repository.

## Which permissions does my API key need?

`forge:read` to clone, fetch and read, and `forge:write` to push, open and merge pull requests, report checks and publish releases.
`miosa login` gives a key with both.
A key without them gets `insufficient scope` and the name of the permission it needs.

## Can I use the GitHub CLI or another Git host?

Forge speaks standard Git over HTTPS, so any Git client works once `miosa forge auth setup-git` has run.
Pull requests, checks, policy and releases use `miosa forge`, the API or the SDKs.

## Can I import a repository from GitHub?

Yes, from the console.
An import copies the history once; a GitHub App connection keeps a repository mirrored.

## What Forge does not do

| Not supported | What to do |
|---|---|
| Issues | Track work in your own tool, or in pull request comments |
| Repository secrets | Set them on the deployment with `miosa deploy env-set` |
| CI that runs on push | Run your tests elsewhere and report the result with `miosa forge check report` |
| Squash, rebase and auto merge | Merges create a merge commit; the server refuses the other methods without merging |
| Pushing from a sandbox with its own identity | See [Forge from a sandbox](/docs/forge/sandboxes) |
| Raw file download, blame and Git LFS | Clone the repository |

> **Note**
>
> Forge does not do these today.

## Why was my merge refused?

The pull request does not meet the branch policy yet, and nothing was merged.
The error names what is missing and the command that fixes it; see [Fix "needs approval"](/docs/forge/pull-requests#fix-needs-approval).
