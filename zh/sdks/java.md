---
slug: "sdks/java"
title: "Java SDK"
description: "安装、验证并使用 miosa Java SDK 0.5.1——沙箱、exec、文件、快照、计算机、智能体运行和抛出的异常。"
section: "CLI, SDKs & MCP"
order: 11019
---

MIOSA 官方 Java SDK。坐标为 `ai.miosa:miosa-sdk`，版本 0.5.1。需要 Java 11 或更高版本。基于 `java.net.http.HttpClient` 和 Jackson 构建。

## 安装

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

**Gradle（Kotlin DSL）**

```kotlin
implementation("ai.miosa:miosa-sdk:0.5.1")
```

## 身份验证

```java
import ai.miosa.sdk.MiosaClient;

MiosaClient client = new MiosaClient("msk_u_...");
MiosaClient fromEnv = MiosaClient.fromEnv();
```

构造函数：`new MiosaClient(apiKey)`、`(apiKey, baseUrl)` 和 `(apiKey, baseUrl, timeoutMs, maxRetries)`。默认值：基础 URL 为 `https://api.miosa.ai/api/v1`，超时 30 秒，重试 3 次。空密钥会抛出 `IllegalArgumentException`。

`MiosaClient.fromEnv()` 依次读取 `MIOSA_API_KEY`、`MIOSA_ACCESS_TOKEN`、`MIOSA_BASE_URL` 和 `MIOSA_TENANT`。字符串构造函数会忽略环境变量。构建器可添加租户、用户智能体和应用信息：

```java
MiosaClient client = MiosaClient.builder("msk_u_...")
    .tenant("acme")
    .appInfo("Acme", "3.2")
    .build();
client.withTenant("other-org");
```

## 沙箱

沙箱调用位于 `miosa.sandboxes()`。

```java
Computer sbx = client.sandboxes().create("my-box", "small");
boolean ready = client.sandboxes().waitUntilReady(sbx.getId(), WaitOptions.builder().build());
```

| 方法 | 作用 |
|---|---|
| `sandboxes().create(name)` | 创建（`miosa-sandbox`、`small`） |
| `sandboxes().create(name, size)` | 按指定规格创建 |
| `sandboxes().create(name, templateType, size)` | 按模板和规格创建 |
| `sandboxes().create(options)` | 从完整参数映射创建 |
| `sandboxes().get(id)` | 获取单个 |
| `sandboxes().list(query)` / `list()` | 列出 |
| `sandboxes().listPage(query)` | 分页列出 |
| `sandboxes().delete(id)` | 删除 |
| `sandboxes().extend(id, seconds)` | 延长存活时间 |
| `sandboxes().stop(id)` / `pause(id)` / `resume(id)` | 生命周期操作 |
| `sandboxes().usage(id)` | 用量 |
| `sandboxes().waitUntilReady(id, WaitOptions)` | 等待就绪 |

规格为 `xs`、`small`、`medium`、`large`、`xl`。传入原始 `cpu_count` / `memory_mb` / `disk_size_mb` 时，三者必须全部提供，并且与某个具名契约完全匹配，否则会抛出 `IllegalArgumentException`。

## Exec 与文件

```java
CommandResult r = client.sandboxExec().run(sandboxId, "node -v", Map.of());
client.sandboxExec().stream(sandboxId, "npm run dev", Map.of(), ev -> System.out.println(ev));

computer.bash("ls -la");
computer.python("print(2 + 2)");
computer.exec().bash("ls -la", 30);
computer.exec().spawn("bash", SpawnOptions.builder().tty(true).build());
```

文件以计算机为范围。基础 `FilesResource` 功能较少；扩展写入位于 `filesExtended()`。

```java
computer.files().upload("./notes.txt", "/home/user/notes.txt");
byte[] data = computer.files().download("/home/user/notes.txt");
List<FileInfo> entries = computer.files().list("/home/user");
computer.files().delete("/home/user/old");

computer.filesExtended().writeFiles(List.of(new FileWrite("/home/user/a.txt", "hi")));
Iterator<FileWatchEvent> it = computer.filesExtended().watchDir("/home/user");
```

`FilesResource` 仅提供 `upload`、`download`、`list`、`export` 和 `delete`。**此 SDK 尚未提供：** 按路径读取（请使用 `download`），以及基础文件资源上的 `mkdir` / `rename`。

## 快照

```java
Snapshot cp = computer.checkpoints().create("before deps");
Map<String, Object> snaps = client.snapshotHistory().list(Map.of("limit", 50));
```

| 接口面 | 方法 |
|---|---|
| `computer.checkpoints()` | `create`、`list`、`get`、`delete`、`restore` |
| `client.snapshotHistory()` | `list`、`groups`、`get`、`fork`、`delete`、`deleteMany`、`deleteHistory`、`tree`、`awaitTree`、`download` |
| `client.snapshotsStandalone()` | `list`、`get` |
| `client.namedSnapshots()` | `list`、`create`、`get`、`delete`、`deploy` |

## 计算机

```java
Computer computer = client.computers().create("dev-box", "miosa-desktop", "small");
computer.start();
```

| 方法 | 作用 |
|---|---|
| `computers().create(name, templateType, size)` | 创建（另有元数据和归属信息重载） |
| `computers().list()` / `list(page, perPage, status)` | 列出 |
| `computers().get(id)` | 获取单个 |
| `computers().delete(id)` | 删除 |
| `computers().viewerPassword(id)` / `rotateViewerPassword(id)` | 查看器密码 |

`Computer` 句柄还提供 `start`、`stop`、`destroy`、`refresh`、`wait(seconds)`、`clone`、`resize`、`move`、`previewUrl(port)`、`publicUrl`、`screenshot`、`screenshotBase64`、`click`、`rightClick`、`doubleClick`、`type`、`key`、`scroll`、`drag`、`launch`、`windows`、`cursor`、`focusWindow`、`bash`、`python`、`desktop()`、`files()`、`exec()`、`checkpoints()`、`services()`、`env()`、`terminal()`。

## 智能体

此 SDK 没有智能体定义的 CRUD 资源。智能体相关接口面仅包括设置和运行。

```java
client.agentSettings().get(workspaceId);
client.runs().run(Map.of("instruction", "...", "target_kind", "sandbox", "target_id", sbx.getId()));
```

| 接口面 | 方法 |
|---|---|
| `client.agentSettings()` | `get`、`setDefaultHarness`、`updateHarness` |
| `client.agentCredentials()` / `agentModels()` / `agentSignins()` / `agentChats()` | 设置和登录 |
| `client.runs()` | 运行分发和读取 |
| `client.connections()` | 模型提供商和工具连接 |

**此 SDK 尚未提供：** 创建、列出、更新和删除智能体定义。

## 错误

方法会抛出继承自 `MiosaException` 的非受检异常：

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

`MiosaException` 提供 `getStatusCode()`、`getCode()`、`getBody()`、`getRequestId()`、`getDetails()`、`isRetryable()`、`getRetryAfterMs()`、`getKind()` 和 `isStarting()`。子类：`AuthException`、`InsufficientCreditsException`、`NotFoundException`、`ConflictException`、`RateLimitException`、`ValidationException`、`ServerException`、`NetworkException`、`TimeoutException`、`ResourceStartingException`、`SnapshotInUseException`。请根据 `getKind()` 分支处理；`ErrorKind` 会先按 API 代码分类，再按状态码分类。
