---
slug: "forge/pull-requests"
title: "Pull requests and checks"
description: "Open a pull request on Forge, report checks, set branch policy and merge."
section: "Services & projects"
order: 1035
---

A pull request on a Forge repository works the way you expect from other Git hosts.
Checks report results against its commits, and branch policy decides when it can merge.

## Open a pull request

Push a branch, then open the pull request against the default branch:

```bash
git checkout -b greeting
sed -i.bak 's/Hello from Forge/Hello again from Forge/' public/index.html && rm public/index.html.bak
git commit -am "Change the greeting"
git push -u origin greeting
miosa forge pr new hello-forge --head greeting --title "Change the greeting" --body "Says hello again."
```

```text
Number:  1
Title:   Change the greeting
State:   open
From:    greeting
Into:    main
```

Read it back, see what changed and talk about it:

```bash
miosa forge pr list hello-forge
miosa forge pr show hello-forge 1
miosa forge pr diff hello-forge 1 --name-only
miosa forge pr comment hello-forge 1 --body "Looks good to me."
miosa forge pr comments hello-forge 1
```

## Branch policy

Branch policy decides what a pull request needs before it can merge into a branch.

```bash
miosa forge policy show hello-forge
```

```text
Policy:         main v1 (default policy)
Merging needs:  no approval needed (solo organization; the default policy tightens when a second member joins)
Approvals:      1
Independent:    yes
Self-merge:     yes
Checks:         none
```

An organization with one member merges without approvals.
Once a second person joins, the default policy asks for one approval from someone other than the author.

Set your own rules with `policy set`:

```bash
miosa forge policy set hello-forge --approvals 1 --check unit-tests
```

| Flag | What it does |
|---|---|
| `--approvals N` | How many approvals a pull request needs |
| `--independent` | The approver must be someone other than the author (default on) |
| `--check NAME` | A check that must pass; repeat it for several |
| `--code-owners` | Require a code owner's approval |
| `--dismiss-stale` | New commits dismiss earlier approvals (default on) |
| `--branch NAME` | The branch the policy covers (default: the default branch) |

### Working alone

An organization with one member can turn off the approval requirement so the author merges their own pull requests:

```bash
miosa forge policy set hello-forge --branch main --approvals 0 --independent=false
```

Required checks still apply.

## Report your own checks

Any tool can report a check against a commit, for example a test run.
A completed check needs evidence: a JSON file with an `evidence` object of up to 64 KB.

```bash
echo '{"evidence": {"runner": "npm test", "passed": 12, "failed": 0}}' > check.json
miosa forge check report hello-forge --commit "$(git rev-parse HEAD)" --name unit-tests --conclusion passed --summary "12 tests passed" --file check.json
miosa forge check list hello-forge --commit "$(git rev-parse HEAD)"
```

> **Note: A required check comes from someone else**
>
> A check that branch policy requires must be reported by someone other than the pull request's author: MIOSA itself, a CI identity or another member of your organization.
> That way an author can never pass their own gate.
> A check the author reports still shows on the pull request, but it does not count toward the policy, and `miosa forge pr check` refuses it with `CHECK_SEPARATION_REQUIRED`.

## Review and merge

A teammate reviews the pull request; the author cannot approve their own:

```bash
miosa forge pr review hello-forge 1 --decision approved --body "Ship it."
```

When the branch policy is met, anyone with write access merges it:

```bash
miosa forge pr merge hello-forge 1
```

```text
ok  Merged pull request #1 into main as 0d4d9eebbc76
```

> **Note: Merge methods**
>
> Today a merge always creates a merge commit.
> The CLI already has `--squash`, `--rebase` and `--auto`, but the server refuses them with `this MIOSA server does not support --squash yet; nothing was merged`.
> Nothing is merged when that happens.

## Fix "needs approval"

When a pull request does not meet its branch policy, the merge is refused and nothing changes:

```text
miosa: pull request #3 cannot merge: policy main v5 needs 1 approval from someone other than the author [REQUIRED_APPROVALS_MISSING]
  Ask someone with write access to approve:  miosa forge pr review hello-forge 3 --decision approved
  Working alone? Let the author merge:      miosa forge policy set hello-forge --branch main --approvals 0 --independent=false
```

Either ask a member with write access to approve, or, if you work alone, lower the policy with the second command.
A missing required check fails the same way with `REQUIRED_CHECK_EVIDENCE_UNAVAILABLE`; run `miosa forge pr show` to see which check is missing.

## Work on someone else's pull request

```bash
miosa forge pr checkout hello-forge 2 --branch review-2
miosa forge pr close hello-forge 2
miosa forge pr reopen hello-forge 2
```
