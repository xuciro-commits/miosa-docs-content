---
slug: "sdks/java"
title: "Java SDK"
description: "Install, authenticate and use the miosa Java SDK 0.5.1 - sandboxes, exec, files, snapshots, computers, agent runs and thrown exceptions."
section: "CLI, SDKs & MCP"
order: 11019
---

The official Java SDK for MIOSA. Coordinates `ai.miosa:miosa-sdk`, version 0.5.1. Requires Java 11 or newer. Built on `java.net.http.HttpClient` and Jackson.

## Install

**Maven**

```xml
<dependency>
    <groupId>ai.miosa</groupId>
    <artifactId>miosa-sdk</artifactId>
    <version>0.5.1</version>
</dependency>
```

**Gradle**

```groovy
implementation 'ai.miosa:miosa-sdk:0.5.1'
```

**Gradle (Kotlin DSL)**

```kotlin
implementation("ai.miosa:miosa-sdk:0.5.1")
```

## Authentication

```java
import ai.miosa.sdk.MiosaClient;

MiosaClient client = new MiosaClient("msk_u_...");
MiosaClient fromEnv = MiosaClient.fromEnv();
```

Constructors: `new MiosaClient(apiKey)`, `(apiKey, baseUrl)` and `(apiKey, baseUrl, timeoutMs, maxRetries)`. Defaults: base URL `https://api.miosa.ai/api/v1`, timeout 30s, 3 retries. An empty key throws `IllegalArgumentException`.

`MiosaClient.fromEnv()` reads `MIOSA_API_KEY`, then `MIOSA_ACCESS_TOKEN`, `MIOSA_BASE_URL` and `MIOSA_TENANT`. The string constructors ignore the environment. A builder adds tenant, user agent and app info:

```java
MiosaClient client = MiosaClient.builder("msk_u_...")
    .tenant("acme")
    .appInfo("Acme", "3.2")
    .build();
client.withTenant("other-org");
```

## Sandboxes

Sandbox calls live on `miosa.sandboxes()`.

```java
Computer sbx = client.sandboxes().create("my-box", "small");
boolean ready = client.sandboxes().waitUntilReady(sbx.getId(), WaitOptions.builder().build());
```

| Method | What it does |
|---|---|
| `sandboxes().create(name)` | Create (`miosa-sandbox`, `small`) |
| `sandboxes().create(name, size)` | Create with a size |
| `sandboxes().create(name, templateType, size)` | Create with template and size |
| `sandboxes().create(options)` | Create from a full parameter map |
| `sandboxes().get(id)` | Get one |
| `sandboxes().list(query)` / `list()` | List |
| `sandboxes().listPage(query)` | Paged list |
| `sandboxes().delete(id)` | Delete |
| `sandboxes().extend(id, seconds)` | Extend the time to live |
| `sandboxes().stop(id)` / `pause(id)` / `resume(id)` | Lifecycle |
| `sandboxes().usage(id)` | Usage |
| `sandboxes().waitUntilReady(id, WaitOptions)` | Wait until ready |

Sizes are `xs`, `small`, `medium`, `large`, `xl`. Passing raw `cpu_count` / `memory_mb` / `disk_size_mb` requires all three and an exact match to a named contract, else `IllegalArgumentException`.

## Exec and files

```java
CommandResult r = client.sandboxExec().run(sandboxId, "node -v", Map.of());
client.sandboxExec().stream(sandboxId, "npm run dev", Map.of(), ev -> System.out.println(ev));

computer.bash("ls -la");
computer.python("print(2 + 2)");
computer.exec().bash("ls -la", 30);
computer.exec().spawn("bash", SpawnOptions.builder().tty(true).build());
```

Files are computer-scoped. The base `FilesResource` is small; extended writes live in `filesExtended()`.

```java
computer.files().upload("./notes.txt", "/home/user/notes.txt");
byte[] data = computer.files().download("/home/user/notes.txt");
List<FileInfo> entries = computer.files().list("/home/user");
computer.files().delete("/home/user/old");

computer.filesExtended().writeFiles(List.of(new FileWrite("/home/user/a.txt", "hi")));
Iterator<FileWatchEvent> it = computer.filesExtended().watchDir("/home/user");
```

`FilesResource` has only `upload`, `download`, `list`, `export` and `delete`. **Not yet in this SDK:** a read-by-path (use `download`) and `mkdir` / `rename` on the base files resource.

## Snapshots

```java
Snapshot cp = computer.checkpoints().create("before deps");
Map<String, Object> snaps = client.snapshotHistory().list(Map.of("limit", 50));
```

| Surface | Methods |
|---|---|
| `computer.checkpoints()` | `create`, `list`, `get`, `delete`, `restore` |
| `client.snapshotHistory()` | `list`, `groups`, `get`, `fork`, `delete`, `deleteMany`, `deleteHistory`, `tree`, `awaitTree`, `download` |
| `client.snapshotsStandalone()` | `list`, `get` |
| `client.namedSnapshots()` | `list`, `create`, `get`, `delete`, `deploy` |

## Computers

```java
Computer computer = client.computers().create("dev-box", "miosa-desktop", "small");
computer.start();
```

| Method | What it does |
|---|---|
| `computers().create(name, templateType, size)` | Create (also metadata and attribution overloads) |
| `computers().list()` / `list(page, perPage, status)` | List |
| `computers().get(id)` | Get one |
| `computers().delete(id)` | Delete |
| `computers().viewerPassword(id)` / `rotateViewerPassword(id)` | Viewer password |

The `Computer` handle adds `start`, `stop`, `destroy`, `refresh`, `wait(seconds)`, `clone`, `resize`, `move`, `previewUrl(port)`, `publicUrl`, `screenshot`, `screenshotBase64`, `click`, `rightClick`, `doubleClick`, `type`, `key`, `scroll`, `drag`, `launch`, `windows`, `cursor`, `focusWindow`, `bash`, `python`, `desktop()`, `files()`, `exec()`, `checkpoints()`, `services()`, `env()`, `terminal()`.

## Agents

There is no agent definition CRUD resource in this SDK. Agent surfaces are settings and runs only.

```java
client.agentSettings().get(workspaceId);
client.runs().run(Map.of("instruction", "...", "target_kind", "sandbox", "target_id", sbx.getId()));
```

| Surface | Methods |
|---|---|
| `client.agentSettings()` | `get`, `setDefaultHarness`, `updateHarness` |
| `client.agentCredentials()` / `agentModels()` / `agentSignins()` / `agentChats()` | Settings and sign-in |
| `client.runs()` | Run dispatch and reads |
| `client.connections()` | Model provider and tool connections |

**Not yet in this SDK:** creating, listing, updating and deleting agent definitions.

## Errors

Methods throw unchecked exceptions extending `MiosaException`:

```java
import ai.miosa.sdk.exceptions.MiosaException;
import ai.miosa.sdk.exceptions.RateLimitException;

try {
    client.sandboxes().get("missing");
} catch (RateLimitException e) {
    System.out.println(e.getRetryAfter());
} catch (MiosaException e) {
    System.out.println(e.getStatusCode() + " " + e.getCode() + " " + e.getRequestId());
}
```

`MiosaException` carries `getStatusCode()`, `getCode()`, `getBody()`, `getRequestId()`, `getDetails()`, `isRetryable()`, `getRetryAfterMs()`, `getKind()` and `isStarting()`. Subclasses: `AuthException`, `InsufficientCreditsException`, `NotFoundException`, `ConflictException`, `RateLimitException`, `ValidationException`, `ServerException`, `NetworkException`, `TimeoutException`, `ResourceStartingException`, `SnapshotInUseException`. Branch on `getKind()`, an `ErrorKind` classified by API code first, then status.
