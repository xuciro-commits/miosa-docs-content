---
slug: "forge/quickstart"
title: "Forge quickstart"
description: "Create a Forge repository, push to it with plain git and open your first pull request, in five minutes."
section: "Services & projects"
order: 1034
---

This takes about five minutes.
You need the `miosa` CLI (version 2.3.10 or later) and `git`.

## 1. Sign in

```bash
miosa login
miosa whoami
```

`miosa login` gives the CLI a key with the Forge permissions, `forge:read` and `forge:write`.
If you use an API key you created yourself instead, tick both permissions when you create it under **API keys** in the console, then `export MIOSA_API_KEY=msk_...`.
Without them every `miosa forge` command answers `insufficient scope`.

## 2. Let git use your MIOSA login

```bash
miosa forge auth setup-git
```

```text
ok  git now uses your MIOSA login for https://forge.miosa.ai
```

This adds a credential helper for `forge.miosa.ai` to your global git config.
Git then signs in with your MIOSA key, so you never paste a password or token.

## 3. Create a repository and clone it

```bash
miosa forge repo create hello-forge
miosa forge repo clone hello-forge
cd hello-forge
```

```text
ok  Created repository "hello-forge" in the current organization
  Visibility: private
  Clone URL:  https://forge.miosa.ai/<org>/hello-forge.git
```

A new repository is private and starts empty, so git warns that you cloned an empty repository.

## 4. Commit and push

```bash
mkdir -p public
printf '<!doctype html>\n<title>Hello from Forge</title>\n<h1>Hello from Forge</h1>\n' > public/index.html
git add .
git commit -m "First page"
git push -u origin main
```

That is plain git: any client, editor or tool that speaks Git over HTTPS works the same way.

## 5. Open a pull request

```bash
git checkout -b greeting
sed -i.bak 's/Hello from Forge/Hello again from Forge/' public/index.html && rm public/index.html.bak
git commit -am "Change the greeting"
git push -u origin greeting
miosa forge pr new hello-forge --head greeting --title "Change the greeting"
miosa forge pr merge hello-forge 1
```

```text
ok  Merged pull request #1 into main as 0d4d9eebbc76
```

An organization with one member can merge its own pull requests.
Once a teammate joins, a pull request needs their approval first; see [branch policy](/docs/forge/pull-requests#branch-policy).

## Clean up

```bash
miosa forge repo list
miosa forge repo delete <repository-id> --yes
```

`repo delete` takes the repository id that `repo list` shows, and it is permanent.

## Next

- [Pull requests, checks and branch policy](/docs/forge/pull-requests)
- [Every forge command](/docs/forge/cli)
