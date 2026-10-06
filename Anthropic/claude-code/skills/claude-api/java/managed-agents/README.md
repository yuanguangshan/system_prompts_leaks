<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Java / 托管智能体 - Java

> **Bindings not shown here:** This README covers the most common managed-agents flows for Java. If you need a class, method, namespace, field, or behavior that isn't shown, WebFetch the Java SDK repo **or the relevant docs page** from `shared/live-sources.md` rather than guess. Do not extrapolate from cURL shapes or another language's SDK.

> **此处未展示的绑定：** 本 README 涵盖 Java 最常见的托管智能体流程。如果你需要某个未展示的类、方法、命名空间、字段或行为，请通过 WebFetch 从 `shared/live-sources.md` 中获取 Java SDK 仓库**或相关文档页面**，而不要凭猜测行事。不要从 cURL 形态或其他语言的 SDK 进行外推。

> **Agents are persistent - create once, reference by ID.** Store the agent ID returned by `client.beta().agents().create` and pass it to every subsequent `client.beta().sessions().create`; do not call `agents().create` in the request path. **Recommended:** define agents and environments as version-controlled files synced with `ant apply` - see `shared/anthropic-cli.md` (its live-docs URL is in `shared/live-sources.md`). The CLI owns the control plane (create/update); your code owns the data plane (sessions with the stored ID). The examples below show in-code creation for when you must provision programmatically; in production the create call belongs in setup, not in the request path.

> **智能体是持久存在的 - 创建一次，通过 ID 引用。** 请保存 `client.beta().agents().create` 返回的智能体 ID，并在后续每次 `client.beta().sessions().create` 时传入该 ID；不要在请求路径中调用 `agents().create`。**推荐做法：** 将智能体和环境定义为受版本控制的文件，并通过 `ant apply` 同步 - 参见 `shared/anthropic-cli.md`（其实时文档 URL 见 `shared/live-sources.md`）。CLI 负责控制平面（创建/更新）；你的代码负责数据平面（使用已保存的 ID 创建会话）。以下示例展示了必须在代码中以编程方式置备时的创建方式；在生产环境中，创建调用应属于初始化设置阶段，而不应出现在请求路径中。
【评论】此处体现了控制平面与数据平面分离的架构约束：智能体对象被视为持久资源而非请求期产物，避免在请求路径中重复创建。

## Installation / 安装

```xml
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>anthropic-java</artifactId>
</dependency>
```

## Client Initialization / 客户端初始化

```java
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// Default (uses ANTHROPIC_API_KEY env var)
var client = AnthropicOkHttpClient.fromEnv();
```

---

## Create an Environment / 创建环境

```java
import com.anthropic.models.beta.environments.BetaCloudConfigParams;
import com.anthropic.models.beta.environments.BetaUnrestrictedNetwork;
import com.anthropic.models.beta.environments.EnvironmentCreateParams;

var environment = client.beta().environments().create(EnvironmentCreateParams.builder()
    .name("my-dev-env")
    .config(BetaCloudConfigParams.builder()
        .networking(BetaUnrestrictedNetwork.builder().build())
        .build())
    .build());
System.out.println("Environment ID: " + environment.id()); // env_...
```

---

## Create an Agent (required first step) / 创建智能体（必需的第一步）

> Warning: **There is no inline agent config.** Model, system, and tools live on the agent object, not the session. Always start with `client.beta().agents().create()` - the session takes either `.agent(agent.id())` or the typed `BetaManagedAgentsAgentParams.builder()...build()`.

> 警告：**不存在内联的智能体配置。** 模型、系统提示词和工具都挂在智能体对象上，而不是会话上。始终从 `client.beta().agents().create()` 开始 - 会话要么接受 `.agent(agent.id())`，要么接受类型化的 `BetaManagedAgentsAgentParams.builder()...build()`。

### Minimal / 最小示例

```java
import com.anthropic.models.beta.agents.AgentCreateParams;
import com.anthropic.models.beta.agents.BetaManagedAgentsAgentToolset20260401Params;
import com.anthropic.models.beta.sessions.BetaManagedAgentsAgentParams;
import com.anthropic.models.beta.sessions.SessionCreateParams;

// 1. Create the agent (reusable, versioned)
var agent = client.beta().agents().create(AgentCreateParams.builder()
    .name("Coding Assistant")
    .model("claude-opus-5-5")
    .system("You are a helpful coding assistant.")
    .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
        .type(BetaManagedAgentsAgentToolset20260401Params.Type.AGENT_TOOLSET_20260401)
        .build())
    .build());

// 2. Start a session
var session = client.beta().sessions().create(SessionCreateParams.builder()
    .agent(BetaManagedAgentsAgentParams.builder()
        .type(BetaManagedAgentsAgentParams.Type.AGENT)
        .id(agent.id())
        .version(agent.version())
        .build())
    .environmentId(environment.id())
    .title("Quickstart session")
    .build());
System.out.println("Session ID: " + session.id());
System.out.println("Trace: https://platform.claude.com/workspaces/default/sessions/" + session.id()); // swap 'default' for your workspace ID if the API key is not in the Default workspace
```

### Updating an Agent / 更新智能体

Updates create new versions; the agent object is immutable per version.

更新会创建新版本；智能体对象在每个版本内是不可变的。

```java
import com.anthropic.models.beta.agents.AgentUpdateParams;

var updatedAgent = client.beta().agents().update(agent.id(), AgentUpdateParams.builder()
    .version(agent.version())
    .system("You are a helpful coding agent. Always write tests.")
    .build());
System.out.println("New version: " + updatedAgent.version());

// List all versions
for (var version : client.beta().agents().versions().list(agent.id()).autoPager()) {
    System.out.println("Version " + version.version() + ": " + version.updatedAt());
}

// Archive the agent
var archived = client.beta().agents().archive(agent.id());
System.out.println("Archived at: " + archived.archivedAt().orElseThrow());
```

---

## Send a User Message / 发送用户消息

```java
import com.anthropic.models.beta.sessions.events.BetaManagedAgentsUserMessageEventParams;
import com.anthropic.models.beta.sessions.events.EventSendParams;

client.beta().sessions().events().send(session.id(), EventSendParams.builder()
    .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
        .type(BetaManagedAgentsUserMessageEventParams.Type.USER_MESSAGE)
        .addTextContent("Review the auth module")
        .build())
    .build());
```

> Tip: **Stream-first:** Open the stream *before* (or concurrently with) sending the message. The stream only delivers events that occur after it opens - stream-after-send means early events arrive buffered in one batch. See [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns).

> 提示：**流式优先：** 在发送消息*之前*（或同时）打开流。流只会递送在它打开之后发生的事件 - 先发送后开流意味着早期事件会以一批缓冲的形式到达。参见 [转向模式](../../shared/managed-agents-events.md#steering-patterns)。

---

## Stream Events (SSE) / 流式接收事件（SSE）

```java
import com.anthropic.models.beta.sessions.events.StreamEvents;

// Open the stream first, then send the user message
try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
    client.beta().sessions().events().send(session.id(), EventSendParams.builder()
        .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
            .type(BetaManagedAgentsUserMessageEventParams.Type.USER_MESSAGE)
            .addTextContent("Summarize the repo README")
            .build())
        .build());

    for (var event : (Iterable<StreamEvents>) stream.stream()::iterator) {
        if (event.isAgentMessage()) {
            event.asAgentMessage().content().forEach(block -> System.out.print(block.text()));
        } else if (event.isAgentToolUse()) {
            System.out.println("\n[Using tool: " + event.asAgentToolUse().name() + "]");
        } else if (event.isSessionStatusIdle()) {
            break;
        } else if (event.isSessionError()) {
            System.out.println("\n[Error]");
            break;
        }
    }
}
```

### Reconnecting and Tailing / 重连与尾随监听

When reconnecting mid-session, list past events first to dedupe, then tail live events. The cross-variant `id` field is read from the raw `_json()` value:

在会话中途重连时，先列出历史事件以去重，然后再尾随监听实时事件。跨变体通用的 `id` 字段需从原始的 `_json()` 值中读取：

```java
import com.anthropic.core.JsonValue;
import java.util.HashSet;
import java.util.Map;
import java.util.Optional;

try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
    // Stream is open and buffering. List history before tailing live.
    var seenEventIds = new HashSet<String>();
    for (var past : client.beta().sessions().events().list(session.id()).autoPager()) {
        Optional<Map<String, JsonValue>> obj = past._json().orElseThrow().asObject();
        seenEventIds.add(obj.orElseThrow().get("id").asStringOrThrow());
    }

    // Tail live events, skipping anything already seen
    for (var event : (Iterable<StreamEvents>) stream.stream()::iterator) {
        Optional<Map<String, JsonValue>> obj = event._json().orElseThrow().asObject();
        if (!seenEventIds.add(obj.orElseThrow().get("id").asStringOrThrow())) continue;
        if (event.isAgentMessage()) {
            event.asAgentMessage().content().forEach(block -> System.out.print(block.text()));
        } else if (event.isSessionStatusIdle()) {
            break;
        }
    }
}
```

---

## Provide Custom Tool Result / 提供自定义工具结果

> Note: The Java managed-agents bindings for `user.custom_tool_result` are not yet documented in this skill or in the apps source examples. Refer to `shared/managed-agents-events.md` for the wire format and the `anthropic-java` repository for the corresponding params types.

> 注意：`user.custom_tool_result` 的 Java 托管智能体绑定尚未在本技能或 apps 源码示例中提供文档。线上格式请参见 `shared/managed-agents-events.md`，对应的参数类型请参见 `anthropic-java` 仓库。

---

## Poll Events / 轮询事件

```java
for (var event : client.beta().sessions().events().list(session.id()).autoPager()) {
    System.out.println(event.type() + ": " + event);
}
```

---

## Upload a File / 上传文件

```java
import com.anthropic.models.beta.files.FileUploadParams;
import com.anthropic.models.beta.sessions.BetaManagedAgentsFileResourceParams;
import java.nio.file.Path;

var dataCsv = Path.of("data.csv");

var file = client.beta().files().upload(FileUploadParams.builder()
    .file(dataCsv)
    .build());
System.out.println("File ID: " + file.id());

// Mount in a session
var session = client.beta().sessions().create(SessionCreateParams.builder()
    .agent(agent.id())
    .environmentId(environment.id())
    .addResource(BetaManagedAgentsFileResourceParams.builder()
        .type(BetaManagedAgentsFileResourceParams.Type.FILE)
        .fileId(file.id())
        .mountPath("/workspace/data.csv")
        .build())
    .build());
```

### Add and Manage Resources on an Existing Session / 在现有会话上添加和管理资源

```java
import com.anthropic.models.beta.sessions.resources.ResourceAddParams;
import com.anthropic.models.beta.sessions.resources.ResourceDeleteParams;

// Attach an additional file to an open session
var resource = client.beta().sessions().resources().add(session.id(), ResourceAddParams.builder()
    .betaManagedAgentsFileResourceParams(BetaManagedAgentsFileResourceParams.builder()
        .type(BetaManagedAgentsFileResourceParams.Type.FILE)
        .fileId(file.id())
        .build())
    .build());
System.out.println(resource.id()); // "sesrsc_01ABC..."

// List resources on the session - entries are a discriminated union
var listed = client.beta().sessions().resources().list(session.id());
for (var entry : listed.data()) {
    if (entry.isFile()) {
        var fileResource = entry.asFile();
        System.out.println(fileResource.id() + " " + fileResource.type());
    } else if (entry.isGitHubRepository()) {
        var repoResource = entry.asGitHubRepository();
        System.out.println(repoResource.id() + " " + repoResource.type());
    }
}

// Detach a resource
client.beta().sessions().resources().delete(resource.id(), ResourceDeleteParams.builder()
    .sessionId(session.id())
    .build());
```

---

## List and Download Session Files / 列出并下载会话文件

> Note: Listing and downloading files an agent wrote during a session is not yet documented for Java in this skill or in the apps source examples. See `shared/managed-agents-events.md` and the `anthropic-java` repository for the file list/download bindings.

> 注意：针对 Java，列出和下载智能体在会话期间写入的文件这一功能尚未在本技能或 apps 源码示例中提供文档。文件列表/下载绑定请参见 `shared/managed-agents-events.md` 与 `anthropic-java` 仓库。

---

## Session Management / 会话管理

```java
// List environments
var environments = client.beta().environments().list();

// Retrieve a specific environment
var env = client.beta().environments().retrieve(environment.id());

// Archive an environment (read-only, existing sessions continue)
client.beta().environments().archive(environment.id());

// Delete an environment (only if no sessions reference it)
client.beta().environments().delete(environment.id());

// Delete a session
client.beta().sessions().delete(session.id());
```

---

## MCP Server Integration / MCP 服务器集成

```java
import com.anthropic.models.beta.agents.BetaManagedAgentsMcpToolsetParams;
import com.anthropic.models.beta.agents.BetaManagedAgentsUrlMcpServerParams;

// Agent declares MCP server (no auth here - auth goes in a vault)
var agent = client.beta().agents().create(AgentCreateParams.builder()
    .name("GitHub Assistant")
    .model("claude-opus-5-5")
    .addMcpServer(BetaManagedAgentsUrlMcpServerParams.builder()
        .type(BetaManagedAgentsUrlMcpServerParams.Type.URL)
        .name("github")
        .url("https://api.githubcopilot.com/mcp/")
        .build())
    .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
        .type(BetaManagedAgentsAgentToolset20260401Params.Type.AGENT_TOOLSET_20260401)
        .build())
    .addTool(BetaManagedAgentsMcpToolsetParams.builder()
        .type(BetaManagedAgentsMcpToolsetParams.Type.MCP_TOOLSET)
        .mcpServerName("github")
        .build())
    .build());

// Session attaches vault(s) containing credentials for those MCP server URLs
var session = client.beta().sessions().create(SessionCreateParams.builder()
    .agent(BetaManagedAgentsAgentParams.builder()
        .type(BetaManagedAgentsAgentParams.Type.AGENT)
        .id(agent.id())
        .version(agent.version())
        .build())
    .environmentId(environment.id())
    .addVaultId(vault.id())
    .build());
```

See `shared/managed-agents-tools.md` §Vaults for creating vaults and adding credentials.

创建保管库和添加凭据请参见 `shared/managed-agents-tools.md` 的 §Vaults 一节。

---

## Vaults / 保管库（Vaults）

```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.beta.vaults.VaultCreateParams;
import com.anthropic.models.beta.vaults.credentials.BetaManagedAgentsMcpOAuthCreateParams;
import com.anthropic.models.beta.vaults.credentials.BetaManagedAgentsMcpOAuthRefreshParams;
import com.anthropic.models.beta.vaults.credentials.BetaManagedAgentsMcpOAuthRefreshUpdateParams;
import com.anthropic.models.beta.vaults.credentials.BetaManagedAgentsMcpOAuthUpdateParams;
import com.anthropic.models.beta.vaults.credentials.CredentialCreateParams;
import com.anthropic.models.beta.vaults.credentials.CredentialUpdateParams;
import java.time.OffsetDateTime;

// Create a vault
var vault = client.beta().vaults().create(VaultCreateParams.builder()
    .displayName("Alice")
    .metadata(VaultCreateParams.Metadata.builder()
        .putAdditionalProperty("external_user_id", JsonValue.from("usr_abc123"))
        .build())
    .build());
System.out.println(vault.id()); // "vlt_01ABC..."

// Add an OAuth credential
var credential = client.beta().vaults().credentials().create(vault.id(),
    CredentialCreateParams.builder()
        .displayName("Alice's Slack")
        .auth(BetaManagedAgentsMcpOAuthCreateParams.builder()
            .type(BetaManagedAgentsMcpOAuthCreateParams.Type.MCP_OAUTH)
            .mcpServerUrl("https://mcp.slack.com/mcp")
            .accessToken("xoxp-...")
            .expiresAt(OffsetDateTime.parse("2026-04-15T00:00:00Z"))
            .refresh(BetaManagedAgentsMcpOAuthRefreshParams.builder()
                .tokenEndpoint("https://slack.com/api/oauth.v2.access")
                .clientId("1234567890.0987654321")
                .scope("channels:read chat:write")
                .refreshToken("xoxe-1-...")
                .clientSecretPostTokenEndpointAuth("abc123...")
                .build())
            .build())
        .build());

// Rotate the credential (e.g., after a token refresh)
client.beta().vaults().credentials().update(credential.id(),
    CredentialUpdateParams.builder()
        .vaultId(vault.id())
        .auth(BetaManagedAgentsMcpOAuthUpdateParams.builder()
            .type(BetaManagedAgentsMcpOAuthUpdateParams.Type.MCP_OAUTH)
            .accessToken("xoxp-new-...")
            .expiresAt(OffsetDateTime.parse("2026-05-15T00:00:00Z"))
            .refresh(BetaManagedAgentsMcpOAuthRefreshUpdateParams.builder()
                .refreshToken("xoxe-1-new-...")
                .build())
            .build())
        .build());

// Archive a vault
client.beta().vaults().archive(vault.id());
```
【评论】凭据与智能体声明分离：智能体只声明 MCP 服务器 URL，会话通过附加保管库提供凭据，避免将令牌直接写入智能体配置。

---

## GitHub Repository Integration / GitHub 仓库集成

Mount a GitHub repository as a session resource (a vault holds the GitHub MCP credential):

将 GitHub 仓库挂载为会话资源（保管库中保存 GitHub MCP 凭据）：

```java
import com.anthropic.models.beta.sessions.BetaManagedAgentsGitHubRepositoryResourceParams;

var session = client.beta().sessions().create(SessionCreateParams.builder()
    .agent(agent.id())
    .environmentId(environment.id())
    .addVaultId(vault.id())
    .addResource(BetaManagedAgentsGitHubRepositoryResourceParams.builder()
        .type(BetaManagedAgentsGitHubRepositoryResourceParams.Type.GITHUB_REPOSITORY)
        .url("https://github.com/org/repo")
        .mountPath("/workspace/repo")
        .authorizationToken("ghp_your_github_token")
        .build())
    .build());
```

Multiple repositories on the same session:

同一会话挂载多个仓库：

```java
import java.util.List;

var resources = List.of(
    BetaManagedAgentsGitHubRepositoryResourceParams.builder()
        .type(BetaManagedAgentsGitHubRepositoryResourceParams.Type.GITHUB_REPOSITORY)
        .url("https://github.com/org/frontend")
        .mountPath("/workspace/frontend")
        .authorizationToken("ghp_your_github_token")
        .build(),
    BetaManagedAgentsGitHubRepositoryResourceParams.builder()
        .type(BetaManagedAgentsGitHubRepositoryResourceParams.Type.GITHUB_REPOSITORY)
        .url("https://github.com/org/backend")
        .mountPath("/workspace/backend")
        .authorizationToken("ghp_your_github_token")
        .build());
```

Rotating a repository's authorization token:

轮换仓库的授权令牌：

```java
import com.anthropic.models.beta.sessions.resources.ResourceUpdateParams;

var listed = client.beta().sessions().resources().list(session.id());
var repoResourceId = listed.data().get(0).asGitHubRepository().id();

client.beta().sessions().resources().update(repoResourceId, ResourceUpdateParams.builder()
    .sessionId(session.id())
    .authorizationToken("ghp_your_new_github_token")
    .build());
```
