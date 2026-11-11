---
slug: "forge/pull-requests"
title: "拉取请求和检查"
description: "在 Forge 上打开拉取请求、报告检查、设置分支策略并合并。"
section: "Services & projects"
order: 1035
---

Forge 存储库上的拉取请求的工作方式与您期望从其他 Git 主机发出的方式相同。
根据提交检查报告结果，分支策略决定何时可以合并。

## 打开拉取请求

推送一个分支，然后针对默认分支打开拉取请求：

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

读回来，看看发生了什么变化并讨论它：

```bash
miosa forge pr list hello-forge
miosa forge pr show hello-forge 1
miosa forge pr diff hello-forge 1 --name-only
miosa forge pr comment hello-forge 1 --body "Looks good to me."
miosa forge pr comments hello-forge 1
```

## 分行政策

分支策略决定拉取请求在合并到分支之前需要什么。

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

一个只有一名成员的组织未经批准就合并。
一旦第二个人加入，默认策略会要求作者以外的其他人批准。

使用 `policy set` 设置您自己的规则：

```bash
miosa forge policy set hello-forge --approvals 1 --check unit-tests
```

|旗帜|它有什么作用 |
|---|---|
| `--approvals N` |拉取请求需要多少批准 |
| `--independent` |审批者必须是作者以外的人（默认开启）|
| `--check NAME` |必须通过的检查；重复几次|
| `--code-owners` |需要代码所有者的批准 |
| `--dismiss-stale` |新提交会驳回先前的批准（默认开启）|
| `--branch NAME` |策略覆盖的分支（默认：默认分支）|

### 独自工作

拥有一名成员的组织可以关闭批准要求，以便作者合并他们自己的拉取请求：

```bash
miosa forge policy set hello-forge --branch main --approvals 0 --independent=false
```

所需的检查仍然适用。

## 报告您自己的支票

任何工具都可以报告针对提交的检查，例如测试运行。
完成的检查需要证据：一个包含最大 64 KB 的 `evidence` 对象的 JSON 文件。

```bash
echo '{"evidence": {"runner": "npm test", "passed": 12, "failed": 0}}' > check.json
miosa forge check report hello-forge --commit "$(git rev-parse HEAD)" --name unit-tests --conclusion passed --summary "12 tests passed" --file check.json
miosa forge check list hello-forge --commit "$(git rev-parse HEAD)"
```

> **注意：所需的支票来自其他人**
>
> 分支策略要求的检查必须由拉取请求作者以外的其他人报告：MIOSA 本身、CI 身份或组织的其他成员。
> 这样作者就永远无法跨过自己的大门。
> 作者报告的检查仍然显示在拉取请求上，但它不计入策略，并且 `miosa forge pr check` 使用 `CHECK_SEPARATION_REQUIRED` 拒绝它。

## 审核并合并

队友审查拉取请求；作者不能批准自己的：

```bash
miosa forge pr review hello-forge 1 --decision approved --body "Ship it."
```

当满足分支策略时，任何具有写访问权限的人都会合并它：

```bash
miosa forge pr merge hello-forge 1
```

```text
ok  Merged pull request #1 into main as 0d4d9eebbc76
```

> **注意：合并方法**
>
> 如今，合并总是会创建合并提交。
> CLI 已经有 `--squash`、`--rebase` 和 `--auto`，但服务器用 `this MIOSA server does not support --squash yet; nothing was merged` 拒绝它们。
> 当发生这种情况时，不会合并任何内容。

## 修复“需要批准”

当拉取请求不满足其分支策略时，合并将被拒绝并且不会发生任何变化：

```text
miosa: pull request #3 cannot merge: policy main v5 needs 1 approval from someone other than the author [REQUIRED_APPROVALS_MISSING]
  Ask someone with write access to approve:  miosa forge pr review hello-forge 3 --decision approved
  Working alone? Let the author merge:      miosa forge policy set hello-forge --branch main --approvals 0 --independent=false
```

请求具有写入权限的成员批准，或者，如果您单独工作，请使用第二个命令降低策略。
缺少必需的检查会以与 `REQUIRED_CHECK_EVIDENCE_UNAVAILABLE` 相同的方式失败；运行 `miosa forge pr show` 查看缺少哪个检查。

## 处理别人的拉取请求

```bash
miosa forge pr checkout hello-forge 2 --branch review-2
miosa forge pr close hello-forge 2
miosa forge pr reopen hello-forge 2
```
