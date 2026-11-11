---
slug: "forge/sandboxes"
title: "Forge from a sandbox"
description: "Clone and push to Forge from inside a sandbox with plain git, using the sandbox's own identity."
section: "Services & projects"
order: 1036
---

Code written inside a sandbox, by you or by an agent, should land in a Forge repository with a plain `git push`.
No API key is copied into the sandbox for that.

> **Note: Rolling out**
>
> Everything on this page is rolling out now.
> Until it reaches your organization, a `git clone` or `git push` to Forge from inside a sandbox answers `Authentication failed`.

## How it works

A sandbox already carries its own token, scoped to its organization and workspace.
Forge accepts that token for the repositories of the sandbox's own workspace, and the sandbox writes a git credential helper for `forge.miosa.ai` when it boots.
So inside the sandbox, git just works:

```bash
git clone https://forge.miosa.ai/<org>/<repo>.git
cd <repo>
git checkout -b from-sandbox
echo "<p>Written in a sandbox.</p>" >> public/index.html
git commit -am "Edit from a sandbox"
git push -u origin from-sandbox
```

The sandbox reaches only the repositories of its own workspace, with the same branch policy as everyone else.
Then open a pull request from your machine, the CLI or the API, as in [pull requests](/docs/forge/pull-requests).

## Today, before it reaches you

Do not copy a personal API key into a sandbox to work around it: anyone and any agent that can read the sandbox can read the key.
If you need the files now, copy them out with `miosa files cp -r my-box:/workspace/site ./site` and push from your machine.
