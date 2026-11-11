---
slug: "cli/scripting"
title: "JSON output and scripting"
description: "Machine-readable output, streaming, error shapes, exit codes and CI patterns for the miosa CLI."
section: "CLI, SDKs & MCP"
order: 11012
---

## Machine-readable output

`--json` (or `-o json`, or `MIOSA_OUTPUT=json`) prints machine-readable JSON on stdout.

```bash
miosa list --json | jq -r '.data[].name'
miosa create ci-box --wait --json | jq -r .id
```

Output is never switched to JSON automatically when piped. Commands that stream print one JSON object per line.

## Errors

Errors go to stderr with the API error code. With `--json` they are an `{"error":{...}}` object.

```bash
if ! miosa exec my-box --json -- ./flaky.sh > out.json 2> err.json; then
  jq -r '.error.code' err.json
fi
```

Destructive commands need `--yes` when there is no terminal.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Error |
| 2 | Usage |
| 3 | Not signed in, or forbidden |
| 4 | Not found |
| 5 | Conflict or invalid input |
| 6 | Rate limited (after retries) |
| 7 | Server or network error |

`miosa exec` and `miosa ssh` exit with the remote command's status, so they drop straight into a pipeline:

```bash
miosa exec ci-box -- npm test || echo "tests failed with $?"
```

## Retries and timeouts

`--retries` (default 3) retries on 429 and transient 5xx. `--timeout` bounds a request in seconds, and for `exec` and `ssh` it also bounds the command (1 to 300).

## A CI pattern

```bash
export MIOSA_API_KEY="$MIOSA_CI_KEY"
miosa create "ci-${CI_PIPELINE_ID}" --size small --ttl 1h --wait --json | jq -r .id
miosa exec "ci-${CI_PIPELINE_ID}" -- ./build.sh
status=$?
miosa destroy "ci-${CI_PIPELINE_ID}" --yes
exit $status
```

## Raw calls from scripts

`miosa api` uses the same credentials and prints the body:

```bash
miosa api GET /sandboxes/ci-box/ports --json | jq -r '.data[].port'
miosa api POST /webhooks -d @hook.json
```

> **Tip**
>
> For non-interactive use, set `MIOSA_API_KEY` and avoid browser login entirely. See [Config and profiles](/docs/cli/config) for profiles and credential order.
