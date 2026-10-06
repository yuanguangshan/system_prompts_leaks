<!-- BILINGUAL-EN-ZH -->
# Claude API - Java / Claude API - Java

> **Note:** The Java SDK supports the Claude API and beta tool use with annotated classes. Agent SDK is not yet available for Java.

> **注意：** Java SDK 通过注解类支持 Claude API 和测试版工具调用。Agent SDK 尚未提供 Java 版本。

## Package Reference / 包参考

Types are organized by package. If a class you need isn't shown in an example below, locate it via this table first - don't block on fetching SDK source over the network.

类型按包组织。如果你需要的类未在下面的示例中出现，请先通过此表定位 —— 不要卡在从网络获取 SDK 源码上。

| `import` prefix | Contains |
|---|---|
| `com.anthropic.client` / `com.anthropic.client.okhttp` | `AnthropicClient`, `AnthropicOkHttpClient` |
| `com.anthropic.models.messages` | non-beta request/response types - `MessageCreateParams`, `Model`, `Message`, `TextBlockParam`, `ContentBlockParam`, `ToolUseBlockParam`, `ToolResultBlockParam`, `CacheControlEphemeral`, `Tool*` (e.g. `ToolBash20250124`, `ToolTextEditor20250728`), `StopReason`, `StructuredMessage*` |
| `com.anthropic.models.messages.batches` | Batch API - `BatchResultsParams`, `MessageBatchIndividualResponse` |
| `com.anthropic.models.beta` | `AnthropicBeta` (beta-flag constants) |
| `com.anthropic.models.beta.messages` | beta-endpoint types - `MessageCreateParams`, `BetaMessage`, `BetaStopReason`, `BetaContextManagementConfig`, `BetaMcpToolset`, `BetaRequestMcpServerUrlDefinition`, `BetaTool*` |
| `com.anthropic.core` | `JsonValue`, `JsonField`, `JsonSchemaLocalValidation`, `com.anthropic.core.http.StreamResponse` |
| `com.anthropic.errors` | typed exceptions - `AnthropicServiceException`, `RateLimitException`, `NotFoundException`, etc. (see `shared/error-codes.md`) |

| `import` 前缀 | 包含内容 |
|---|---|
| `com.anthropic.client` / `com.anthropic.client.okhttp` | `AnthropicClient`、`AnthropicOkHttpClient` |
| `com.anthropic.models.messages` | 非 beta 请求/响应类型 —— `MessageCreateParams`、`Model`、`Message`、`TextBlockParam`、`ContentBlockParam`、`ToolUseBlockParam`、`ToolResultBlockParam`、`CacheControlEphemeral`、`Tool*`（例如 `ToolBash20250124`、`ToolTextEditor20250728`）、`StopReason`、`StructuredMessage*` |
| `com.anthropic.models.messages.batches` | Batch API —— `BatchResultsParams`、`MessageBatchIndividualResponse` |
| `com.anthropic.models.beta` | `AnthropicBeta`（beta 标志常量） |
| `com.anthropic.models.beta.messages` | beta 端点类型 —— `MessageCreateParams`、`BetaMessage`、`BetaStopReason`、`BetaContextManagementConfig`、`BetaMcpToolset`、`BetaRequestMcpServerUrlDefinition`、`BetaTool*` |
| `com.anthropic.core` | `JsonValue`、`JsonField`、`JsonSchemaLocalValidation`、`com.anthropic.core.http.StreamResponse` |
| `com.anthropic.errors` | 类型化异常 —— `AnthropicServiceException`、`RateLimitException`、`NotFoundException` 等（见 `shared/error-codes.md`） |

`client.messages()` uses `com.anthropic.models.messages.*`; `client.beta().messages()` uses `com.anthropic.models.beta.messages.*`. Both packages define a `MessageCreateParams` - import the one matching the client path you call.

`client.messages()` 使用 `com.anthropic.models.messages.*`；`client.beta().messages()` 使用 `com.anthropic.models.beta.messages.*`。两个包都定义了 `MessageCreateParams` —— 请导入与你调用的客户端路径相匹配的那一个。

### Key types per feature / 各功能的关键类型

Write from this table instead of `javap`/jar inspection. Endpoint column tells you whether to use `client.messages()` or `client.beta().messages()`.

直接根据此表编写代码，而不是用 `javap`/jar 检查。端点列会告诉你该用 `client.messages()` 还是 `client.beta().messages()`。

| Feature | Endpoint | Key Java types / builder calls |
|---|---|---|
| User profiles | beta | `client.beta().userProfiles().create(...)` / `.retrieve(id)` / `.list()`. Pass the returned profile id on the beta `MessageCreateParams`. Requires a beta header - check the SDK's beta-headers reference for the current flag. |
| Agent Skills | beta | `BetaContainerParams`, `BetaSkillParams`, `BetaCodeExecutionTool20250825`. `.addBeta("code-execution-2025-08-25")` (Skills is out of beta - no `skills-2025-10-02`). Download the output via `client.beta().files().download(fileId)`. |
| Cache diagnostics | beta | `BetaDiagnosticsParam`, `BetaCacheControlEphemeral` |
| Context editing | beta | `.contextManagement(BetaContextManagementConfig.builder()...)`. The edit strategy is a `BetaClearToolUses20250919Edit` (or `BetaClearThinking20251015Edit`); its trigger is a `BetaInputTokensTrigger` built separately and passed to the edit's builder - there is no direct `.inputTokensTrigger(N)` shortcut on the edit builder. `javap` the edit and trigger classes for the exact setter names. |
| Memory tool | non-beta | `.addTool(MemoryTool20250818.builder().build())` from `com.anthropic.models.messages` |
| Programmatic tool calling | non-beta | `CodeExecutionTool20260120`, `Tool`, `ContentBlockParam` |
| Strict tool use | non-beta | `Tool`, `Tool.InputSchema` |
| Task budgets | beta | `.outputConfig(BetaOutputConfig.builder().taskBudget(BetaTokenTaskBudget.builder()...))` |
| Tool search | non-beta | `.addTool(ToolSearchToolRegex20251119.builder()...)` from `com.anthropic.models.messages` |
| Web search | non-beta | `WebSearchTool20260209` from `com.anthropic.models.messages` - the latest variant with dynamic filtering (Claude Fable 5.1 + Claude Opus 5.5 + Claude Opus 5 + Opus 4.8/4.7/4.6 + Claude Sonnet 5.5 + Claude Sonnet 5 + Sonnet 4.6). For older models or Vertex, use `WebSearchTool20250305` |

| 功能 | 端点 | 关键 Java 类型 / 构建器调用 |
|---|---|---|
| 用户画像 | beta | `client.beta().userProfiles().create(...)` / `.retrieve(id)` / `.list()`。把返回的 profile id 传给 beta 版 `MessageCreateParams`。需要 beta 标头 —— 当前标志请查阅 SDK 的 beta-headers 参考。 |
| Agent Skills | beta | `BetaContainerParams`、`BetaSkillParams`、`BetaCodeExecutionTool20250825`。`.addBeta("code-execution-2025-08-25")`（Skills 已脱离 beta —— 不再使用 `skills-2025-10-02`）。通过 `client.beta().files().download(fileId)` 下载输出。 |
| 缓存诊断 | beta | `BetaDiagnosticsParam`、`BetaCacheControlEphemeral` |
| 上下文编辑 | beta | `.contextManagement(BetaContextManagementConfig.builder()...)`。编辑策略是 `BetaClearToolUses20250919Edit`（或 `BetaClearThinking20251015Edit`）；其触发器是一个单独构建、传给编辑策略构建器的 `BetaInputTokensTrigger` —— 编辑策略构建器上没有直接的 `.inputTokensTrigger(N)` 快捷方法。用 `javap` 查看编辑策略和触发器类以获得确切的 setter 名称。 |
| 记忆工具 | 非 beta | 来自 `com.anthropic.models.messages` 的 `.addTool(MemoryTool20250818.builder().build())` |
| 程序化工具调用 | 非 beta | `CodeExecutionTool20260120`、`Tool`、`ContentBlockParam` |
| 严格工具调用 | 非 beta | `Tool`、`Tool.InputSchema` |
| 任务预算 | beta | `.outputConfig(BetaOutputConfig.builder().taskBudget(BetaTokenTaskBudget.builder()...))` |
| 工具搜索 | 非 beta | 来自 `com.anthropic.models.messages` 的 `.addTool(ToolSearchToolRegex20251119.builder()...)` |
| 网页搜索 | 非 beta | 来自 `com.anthropic.models.messages` 的 `WebSearchTool20260209` —— 带动态过滤的最新变体（Claude Fable 5.1 + Claude Opus 5.5 + Claude Opus 5 + Opus 4.8/4.7/4.6 + Claude Sonnet 5.5 + Claude Sonnet 5 + Sonnet 4.6）。较老的模型或 Vertex 请用 `WebSearchTool20250305` |

### Discovering type and member names / 发现类型与成员名称

If a class or builder method you need isn't in the tables above, `jar tf <anthropic-java-core jar> | grep -i <term>` or `javap -classpath <jar> com.anthropic.models....` is fast enough to locate names. **Do not compile and run a separate reflection program** to enumerate members - the first build is slow enough to be backgrounded in many environments, trapping you in a polling loop. Write the script with the names you found and let the compiler error (`cannot find symbol`) point at any wrong member.

如果你需要的类或构建器方法不在上表中，`jar tf <anthropic-java-core jar> | grep -i <term>` 或 `javap -classpath <jar> com.anthropic.models....` 就足以快速定位名称。**不要为了枚举成员而单独编译并运行一个反射程序** —— 首次构建在很多环境中慢到会被转入后台，使你陷入轮询循环。用你找到的名称直接编写脚本，让编译器错误（`cannot find symbol`）指出任何写错的成员。

【评论】"禁止编译反射程序"是一条面向执行代理的效率护栏：JVM 冷启动加首次构建可能远超交互等待时间，用 `jar tf`/`javap` 的静态枚举 + 编译器报错回环更快且更可控。

## Installation / 安装

Maven:

Maven：

```xml
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>anthropic-java</artifactId>
    <version>2.34.0</version>
</dependency>
```

Gradle:

Gradle：

```groovy
implementation("com.anthropic:anthropic-java:2.34.0")
```

## Client Initialization / 客户端初始化

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// Default (reads ANTHROPIC_API_KEY from environment)
AnthropicClient client = AnthropicOkHttpClient.fromEnv();

// Explicit API key
AnthropicClient client = AnthropicOkHttpClient.builder()
    .apiKey("your-api-key")
    .build();
```

---

## Basic Message Request / 基本消息请求

```java
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Message;

MessageCreateParams params = MessageCreateParams.builder()
    .model("claude-opus-5-5")  // .model(String) overload - works for every model id; typed Model.* constants lag model launches
    .maxTokens(16000L)
    .addUserMessage("What is the capital of France?")
    .build();

Message response = client.messages().create(params);
response.content().stream()
    .flatMap(block -> block.text().stream())
    .forEach(textBlock -> System.out.println(textBlock.text()));
```

---

## Thinking / 思考

**Adaptive thinking is the recommended mode for Claude 4.6+ models.** Claude decides dynamically when and how much to think. The builder has a direct `.thinking(ThinkingConfigAdaptive)` overload - no manual union wrapping.

**自适应思考（adaptive thinking）是 Claude 4.6+ 模型的推荐模式。** Claude 动态决定何时思考以及思考多少。构建器有直接的 `.thinking(ThinkingConfigAdaptive)` 重载 —— 无需手动包装 union。

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking (below). `ThinkingConfigEnabled.builder().budgetTokens(N)` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Claude Opus 5.5:** thinking is always on - omit `.thinking(...)` (or send `ThinkingConfigAdaptive`, which is equivalent); `ThinkingConfigDisabled` returns a 400 at every effort, as does a thinking budget. Control depth with `.outputConfig(OutputConfig.builder().effort(...))` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5:** thinking is on by default - omitting `.thinking(...)` runs adaptive (`ThinkingConfigAdaptive` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `ThinkingConfigDisabled` is accepted only at effort `HIGH` or lower; pairing it with `XHIGH`/`MAX` returns a 400.  
> **Older models:** Use `.thinking(ThinkingConfigEnabled.builder().budgetTokens(N).build())` (budget must be < `maxTokens`, min 1024).

> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6：** 使用自适应思考（见下文）。`ThinkingConfigEnabled.builder().budgetTokens(N)` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 和 4.7 上已被移除（发送则返回 400）；在 Opus 4.6 和 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5：** 思考始终开启 —— 省略 `.thinking(...)`（或发送等价的 `ThinkingConfigAdaptive`）；`ThinkingConfigDisabled` 在所有 effort 等级下都返回 400，思考预算同样如此。请改用 `.outputConfig(OutputConfig.builder().effort(...))` 控制深度 —— 该模型默认为 `medium`，而 Claude Opus 5 默认为 `high`。  
> **Claude Opus 5：** 思考默认开启 —— 省略 `.thinking(...)` 会运行自适应模式（`ThinkingConfigAdaptive` 等价）；这与 Opus 4.8/4.7 不同，在那里省略意味着不思考。`ThinkingConfigDisabled` 仅在 effort 为 `HIGH` 或更低时被接受；与 `XHIGH`/`MAX` 搭配会返回 400。  
> **较老的模型：** 使用 `.thinking(ThinkingConfigEnabled.builder().budgetTokens(N).build())`（预算必须 < `maxTokens`，最小 1024）。

```java
import com.anthropic.models.messages.ContentBlock;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.ThinkingConfigAdaptive;

MessageCreateParams params = MessageCreateParams.builder()
    .model("claude-opus-5-5")
    .maxTokens(16000L)
    // display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
    .thinking(ThinkingConfigAdaptive.builder().display(ThinkingConfigAdaptive.Display.SUMMARIZED).build())
    .addUserMessage("Solve this step by step: 27 * 453")
    .build();

for (ContentBlock block : client.messages().create(params).content()) {
    block.thinking().ifPresent(t -> System.out.println("[thinking] " + t.thinking()));
    block.text().ifPresent(t -> System.out.println(t.text()));
}
```

`ContentBlock` narrowing: `.thinking()` / `.text()` return `Optional<T>` - use `.ifPresent(...)` or `.stream().flatMap(...)`. Alternative: `isThinking()` / `asThinking()` boolean+unwrap pairs (throws on wrong variant).

`ContentBlock` 类型收窄：`.thinking()` / `.text()` 返回 `Optional<T>` —— 请用 `.ifPresent(...)` 或 `.stream().flatMap(...)`。另一种方式：`isThinking()` / `asThinking()` 这类布尔判断加解包的成对方法（用错变体时抛异常）。

---

## Effort Parameter / Effort 参数

Effort is nested inside `OutputConfig` - there is NO `.effort()` directly on `MessageCreateParams.Builder`.

Effort 嵌套在 `OutputConfig` 内 —— `MessageCreateParams.Builder` 上**没有**直接的 `.effort()` 方法。

```java
import com.anthropic.models.messages.OutputConfig;

.outputConfig(OutputConfig.builder()
    .effort(OutputConfig.Effort.HIGH)  // or LOW, MEDIUM, XHIGH, MAX
    .build())
```

Combine with `Thinking = ThinkingConfigAdaptive` for cost-quality control.

与 `Thinking = ThinkingConfigAdaptive` 组合使用可实现成本-质量调控。

---

## Prompt Caching / 提示词缓存

System message as a list of `TextBlockParam` with `CacheControlEphemeral`. Use `.systemOfTextBlockParams(...)` - the plain `.system(String)` overload can't carry cache control. For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

系统消息以带 `CacheControlEphemeral` 的 `TextBlockParam` 列表传入。请使用 `.systemOfTextBlockParams(...)` —— 普通 `.system(String)` 重载无法携带缓存控制。放置模式和静默失效排查清单见 `shared/prompt-caching.md`。

```java
import com.anthropic.models.messages.TextBlockParam;
import com.anthropic.models.messages.CacheControlEphemeral;

.systemOfTextBlockParams(List.of(
    TextBlockParam.builder()
        .text(longSystemPrompt)
        .cacheControl(CacheControlEphemeral.builder()
            .ttl(CacheControlEphemeral.Ttl.TTL_1H)  // optional; also TTL_5M
            .build())
        .build()))
```

There's also a top-level `.cacheControl(CacheControlEphemeral)` on `MessageCreateParams.Builder` and on `Tool.builder()`.

`MessageCreateParams.Builder` 和 `Tool.builder()` 上还有顶层的 `.cacheControl(CacheControlEphemeral)`。

Verify hits via `response.usage().cacheCreationInputTokens()` / `response.usage().cacheReadInputTokens()`.

通过 `response.usage().cacheCreationInputTokens()` / `response.usage().cacheReadInputTokens()` 验证缓存命中。

---

## Token Counting / Token 计数

```java
import com.anthropic.models.messages.MessageCountTokensParams;

long tokens = client.messages().countTokens(
    MessageCountTokensParams.builder()
        .model("claude-opus-5-5")
        .addUserMessage("Hello")
        .build()
).inputTokens();
```

---

## PDF / Document Input / PDF / 文档输入

`DocumentBlockParam` builder has source shortcuts. Wrap in `ContentBlockParam.ofDocument()` and pass via `.addUserMessageOfBlockParams()`.

`DocumentBlockParam` 构建器有来源快捷方法。用 `ContentBlockParam.ofDocument()` 包装，并通过 `.addUserMessageOfBlockParams()` 传入。

```java
import com.anthropic.models.messages.DocumentBlockParam;
import com.anthropic.models.messages.ContentBlockParam;
import com.anthropic.models.messages.TextBlockParam;

DocumentBlockParam doc = DocumentBlockParam.builder()
    .source(Base64PdfSource.builder().data(base64String).build())
    // or .source(UrlPdfSource.builder().url("https://...").build())
    .title("My Document")        // optional
    .build();
```

For **Files API** document references, use the beta path and beta types - see `files-api.md`: `BetaRequestDocumentBlock.builder().source(BetaFileDocumentSource.builder().fileId(id).build())`.

**Files API** 文档引用请使用 beta 路径和 beta 类型 —— 见 `files-api.md`：`BetaRequestDocumentBlock.builder().source(BetaFileDocumentSource.builder().fileId(id).build())`。

```java
.addUserMessageOfBlockParams(List.of(
    ContentBlockParam.ofDocument(doc),
    ContentBlockParam.ofText(TextBlockParam.builder().text("Summarize this").build())))
```

---

## Stop Details / 停止详情

When `stopReason()` is `"refusal"`, the response includes structured `stopDetails()`:

当 `stopReason()` 为 `"refusal"` 时，响应包含结构化的 `stopDetails()`：

```java
response.stopDetails().ifPresent(details -> {
    System.out.println("Category: " + details.category());
    System.out.println("Explanation: " + details.explanation());
});
```

**Refusal fallbacks (Claude Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. New `claude-fable-5-1` code should include the server-side `fallbacks` parameter (beta header `server-side-fallback-2026-06-01`, fallback model `claude-opus-4-8`, on the beta messages call) by default. The exact Java builder methods (and the client-side middleware for providers without server-side support) are not documented here - WebFetch the Java SDK repo's `examples/` from `shared/live-sources.md`; full semantics in `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason.

**拒答回退（Claude Fable 5.1）—— 默认应主动启用。** 回退（fallbacks）是可选项：不启用时，被拒答的请求会直接停止。面向 `claude-fable-5-1` 的新代码默认应包含服务端 `fallbacks` 参数（beta 标头 `server-side-fallback-2026-06-01`，回退模型 `claude-opus-4-8`，加在 beta messages 调用上）。确切的 Java 构建器方法（以及针对不支持服务端回退的提供商的客户端中间件）未在本文档中说明 —— 请通过 WebFetch 访问 `shared/live-sources.md` 中 Java SDK 仓库的 `examples/`；完整语义见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason。

---

## Error Type / 错误类型

`AnthropicServiceException` exposes `.errorType()` returning `Optional<ErrorType>` for programmatic error classification:

`AnthropicServiceException` 提供 `.errorType()`，返回 `Optional<ErrorType>`，用于程序化的错误分类：

```java
try {
    client.messages().create(params);
} catch (AnthropicServiceException e) {
    e.errorType().ifPresent(type ->
        System.out.println("Error type: " + type)  // RATE_LIMIT_ERROR, OVERLOADED_ERROR, etc.
    );
}
```

---

---
