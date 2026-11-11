---
slug: "opencomputers/connect"
title: "Connect your first host"
description: "Install the host agent, enroll a machine, verify a command on that exact host, and configure service startup for Linux, macOS, or Windows."
section: "Computers"
order: 3014
---

Enroll a machine you operate as an OpenComputer.
Your first success is a completed `hostname` command on that exact machine, followed by a verified service restart.
A saved key or an online badge alone is not the finish line.

## Before you start

- Outbound HTTPS and WebSocket access on port 443, including access to GitHub release downloads and `api.miosa.ai`.
- Shell access as the user that should run the agent, plus `curl` and a POSIX shell on Linux or macOS.
- A MIOSA account with access to OpenComputers, and either the `miosa` CLI or a platform API key authorized for the intended organization.
- `sudo` access and systemd if you follow the Linux background-service section.

Choose the agent's OS user deliberately: its permissions determine which local files and services the host can reach.
See the [security model](/docs/opencomputers/security) before connecting sensitive workloads.

> **Note: Prebuilt targets are not universal**
>
> The prebuilt Linux agent target is x86-64.
> On Linux ARM64, or on a platform with no prebuilt agent, follow the [OSA source installation instructions](https://github.com/Miosa-osa/OSA#install-in-one-command) rather than assuming the installer supports it.
> An operating system appearing in registration does not guarantee desktop support for every architecture.

## 1. Install the agent on the machine

**Linux / macOS**

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/OSA/main/scripts/install.sh -o /tmp/osa-install.sh
sh /tmp/osa-install.sh
osa version
```

If `osa` is not found, open a new shell and run `osa version` again.
Follow any platform-specific instructions printed by the installer.
For a pinned release, use the version-selection instructions in the [OSA installation guide](https://github.com/Miosa-osa/OSA#install-in-one-command).

**Windows**

Install the OSA agent from the release assets in the [OSA repository](https://github.com/Miosa-osa/OSA), then confirm the agent answers in a new terminal:

```powershell
osa version
```

Registration accepts `--platform windows`.
Desktop streaming depends on the agent build for your platform; a registered Windows or macOS host does not automatically have a working desktop.

## 2. Register the host and save its key

**miosa CLI**

```bash
miosa host register studio --platform linux
```

`--platform` takes `macos`, `linux`, or `windows`.
`register` creates the host record and prints the host ID and the host key once.
Add `--quiet-key` to print only the key, and `--region` to pin a region.

**MIOSA UI**

Open OpenComputers in MIOSA, choose **Connect a computer**, and register the machine in the intended organization.
Give it a recognizable name, such as `lab-server`, and select the operating system.
Save the returned host ID and one-time host key.

The **host key** authenticates this machine's agent.
Your **platform API key** authorizes REST requests from your terminal or backend.
They are different credentials and are not interchangeable.

## 3. Point the agent at MIOSA

On the machine itself, configure the connection with the OSA agent:

```bash
osa opencomputers login
osa opencomputers enable --no-profile
```

Paste the host key when prompted by `login`.
The prompt accepts the credential as terminal input; use a private terminal session.
Configuration is written under the agent user's `~/.osa` directory, including `open_computers.toml` and the `open_computers.ed25519` identity file.
Preserve that identity when updating the agent.

`login` saves the key and `enable` enables host mode.
Neither command proves that a live session has started.

## 4. Start the host session

Start the headless agent in the machine's terminal:

```bash
OSA_OPEN_COMPUTERS_ENABLED=true osa serve
```

Keep it running while completing the first check from another terminal.
The agent initiates an authenticated outbound WebSocket connection to MIOSA.
No inbound management port or public IP is required.

Open the host in MIOSA, or list it from your terminal, and confirm a fresh heartbeat and the capability needed for command execution:

```bash
miosa host list --tag studio
miosa host get <host>
```

If the host does not connect, follow [Host stays offline](/docs/opencomputers/troubleshooting#host-stays-offline).

## 5. Complete a command on the selected host

**miosa CLI**

```bash
miosa host jobs <host>
miosa host job list <host>
miosa host fs ls <host> --path /tmp
```

`miosa host jobs <host>` lists recent jobs. `miosa host job get <host> <job>` shows one job and its output.

**REST API**

In a separate trusted terminal, set your platform API key and the returned host ID:

```bash
export MIOSA_API_KEY="<your-platform-api-key>"
export HOST_ID="<your-host-id>"
```

Confirm the host record:

```bash
curl --fail-with-body -sS "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

The response contains a `host` object.
Confirm its ID, name, heartbeat, and capabilities correspond to this machine.
Then run a read-only command:

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/exec" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cmd":"hostname","stream":false,"timeout_ms":10000}'
```

Expected result, with your own job ID and hostname:

```json
{
  "job_id": "<job-id>",
  "exit_code": 0,
  "stdout": "lab-server\n",
  "stderr": ""
}
```

Compare `stdout` with `hostname` run directly on the machine.
With `stream: false` the API collects output; the default is `stream: true`, which returns SSE events until a terminal result.
For longer commands, use the [streaming exec contract](/docs/api-reference/open-computers#run-a-job) instead of this short collected-output check.

Use the IDs from your own request and the same authorized organization context.
An accepted request is not proof that the command completed.
Keep the API key in your backend or terminal environment.

## 6. Run the agent as a service

After the command passes, stop the foreground `osa serve` process before starting a second instance.

**Linux (systemd)**

The unit runs under the same user and home directory that hold the configured identity.
Run these commands as that user, not from a root login shell.
If this unit already exists, inspect it before replacing it; preserve any site-specific restrictions.

```bash
OSA_HOST_BIN="$(command -v osa)"
OSA_HOST_USER="$(id -un)"

sudo tee /etc/systemd/system/osa-opencomputers.service > /dev/null <<EOF_UNIT
[Unit]
Description=MIOSA OSA OpenComputers Host
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=${OSA_HOST_USER}
Environment=HOME=${HOME}
Environment=OSA_OPEN_COMPUTERS_ENABLED=true
ExecStart=${OSA_HOST_BIN} serve
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF_UNIT

sudo systemctl daemon-reload
sudo systemctl enable --now osa-opencomputers.service
systemctl status osa-opencomputers.service --no-pager
```

**Other service managers**

Use your platform's service manager under the same agent user, and keep two things with the service:

- the `OSA_OPEN_COMPUTERS_ENABLED=true` environment variable, so the agent starts in host mode;
- the `~/.osa` directory, including `open_computers.toml` and the `open_computers.ed25519` identity file.

A launch agent on macOS and a Windows service both work this way.
Then confirm a fresh heartbeat in MIOSA.

Expect an active service, then verify a new heartbeat in MIOSA and repeat the `hostname` request.
An enabled service should start at boot, but verify that behavior after a scheduled reboot before relying on unattended work.

## 7. Disconnect or retire the host

For temporary maintenance, stop dispatching new work, inspect running jobs, and stop the agent:

```bash
miosa host jobs <host>
sudo systemctl stop osa-opencomputers.service
```

The unit remains enabled for the next boot.
For retirement, disable startup, log the agent out, and revoke the host:

```bash
sudo systemctl disable --now osa-opencomputers.service
osa opencomputers logout
miosa host rm <host> --yes
```

Revocation and logout do not erase the machine's files, and they do not prove that every child process has stopped.
Verify active work before removing access or retrying it elsewhere.

- [Hosts and capabilities](/docs/opencomputers/hosts): Inspect a host and run work on one machine or a group.

- [OpenComputers troubleshooting](/docs/opencomputers/troubleshooting): Diagnose offline hosts, failed commands, identity rejection, and tunnels.
