<!-- BILINGUAL-EN-ZH -->
# Tool Use - Java / 工具使用 - Java

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

概念性概览（工具定义、工具选择、技巧）见 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Use (Beta) / 工具使用（Beta）

The Java SDK supports beta tool use with annotated classes. Tool classes implement `Supplier<String>` for automatic execution via `BetaToolRunner`.

Java SDK 通过注解类支持 beta 工具使用。工具类实现 `Supplier<String>`，以便由 `BetaToolRunner` 自动执行。

### Tool Runner (automatic loop) / 工具运行器（自动循环）

```java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.helpers.BetaToolRunner;
import com.fasterxml.jackson.annotation.JsonClassDescription;
import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import java.util.function.Supplier;

@JsonClassDescription("Get the weather in a given location")
static class GetWeather implements Supplier<String> {
    @JsonPropertyDescription("The city and state, e.g. San Francisco, CA")
    public String location;

    @Override
    public String get() {
        return "The weather in " + location + " is sunny and 72°F";
    }
}

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    MessageCreateParams.builder()
        .model("claude-opus-5-5")
        .maxTokens(16000L)
        .putAdditionalHeader("anthropic-beta", "structured-outputs-2025-11-13")
        .addTool(GetWeather.class)
        .addUserMessage("What's the weather in San Francisco?")
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
```

### Memory Tool / 记忆工具

The Java SDK provides `BetaMemoryToolHandler` for implementing the memory tool backend. You supply a handler that manages file storage, and the `BetaToolRunner` handles memory tool calls automatically.

Java SDK 提供 `BetaMemoryToolHandler` 用于实现记忆工具的后端。你提供一个管理文件存储的处理器，`BetaToolRunner` 则自动处理记忆工具调用。

```java
import com.anthropic.helpers.BetaMemoryToolHandler;
import com.anthropic.helpers.BetaToolRunner;
import com.anthropic.models.beta.messages.BetaMemoryTool20250818;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.ToolRunnerCreateParams;

// Implement BetaMemoryToolHandler with your storage backend (e.g., filesystem)
BetaMemoryToolHandler memoryHandler = new FileSystemMemoryToolHandler(sandboxRoot);

MessageCreateParams createParams = MessageCreateParams.builder()
    .model("claude-opus-5-5")
    .maxTokens(4096L)
    .addTool(BetaMemoryTool20250818.builder().build())
    .addUserMessage("Remember that my favorite color is blue")
    .build();

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    ToolRunnerCreateParams.builder()
        .betaMemoryToolHandler(memoryHandler)
        .initialMessageParams(createParams)
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
```

See the [shared memory tool concepts](../../shared/tool-use-concepts.md) for more details on the memory tool.

关于记忆工具的更多细节，见[共享记忆工具概念](../../shared/tool-use-concepts.md)。

### Non-Beta Tool Declaration (manual JSON schema) / 非 Beta 工具声明（手动 JSON schema）

`Tool.InputSchema.Properties` is a freeform `Map<String, JsonValue>` wrapper - build property schemas via `putAdditionalProperty`. `type: "object"` is the default. The builder has a direct `.addTool(Tool)` overload that wraps in `ToolUnion` automatically.

`Tool.InputSchema.Properties` 是一个自由格式的 `Map<String, JsonValue>` 包装——通过 `putAdditionalProperty` 构建属性 schema。`type: "object"` 是默认值。builder 有一个直接的 `.addTool(Tool)` 重载，会自动包装进 `ToolUnion`。

```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.messages.Tool;

Tool tool = Tool.builder()
    .name("get_weather")
    .description("Get the current weather in a given location")
    .inputSchema(Tool.InputSchema.builder()
        .properties(Tool.InputSchema.Properties.builder()
            .putAdditionalProperty("location", JsonValue.from(Map.of("type", "string")))
            .build())
        .required(List.of("location"))
        .build())
    .build();

MessageCreateParams params = MessageCreateParams.builder()
    .model("claude-opus-5-5")
    .maxTokens(16000L)
    .addTool(tool)
    .addUserMessage("Weather in Paris?")
    .build();
```

For manual tool loops, handle `tool_use` blocks in the response, send `tool_result` back, loop until `stop_reason` is `"end_turn"`. See [shared tool use concepts](../../shared/tool-use-concepts.md).

手动工具循环时，处理响应中的 `tool_use` 块，回发 `tool_result`，循环直到 `stop_reason` 为 `"end_turn"`。见[共享工具使用概念](../../shared/tool-use-concepts.md)。

### Building `MessageParam` with Content Blocks (Tool Result Round-Trip) / 用内容块构建 `MessageParam`（工具结果往返）

`MessageParam.Content` is an inner union class (string | list). Use the builder's `.contentOfBlockParams(List<ContentBlockParam>)` alias - there is NO separate `MessageParamContent` class with a static `ofBlockParams`:

`MessageParam.Content` 是一个内部 union 类（string | list）。使用 builder 的 `.contentOfBlockParams(List<ContentBlockParam>)` 别名——并不存在带静态 `ofBlockParams` 方法的独立 `MessageParamContent` 类：

```java
import com.anthropic.models.messages.MessageParam;
import com.anthropic.models.messages.ContentBlockParam;
import com.anthropic.models.messages.ToolResultBlockParam;

List<ContentBlockParam> results = List.of(
    ContentBlockParam.ofToolResult(ToolResultBlockParam.builder()
        .toolUseId(toolUseBlock.id())
        .content(yourResultString)
        .build())
);

MessageParam toolResultMsg = MessageParam.builder()
    .role(MessageParam.Role.USER)
    .contentOfBlockParams(results)   // builder alias for Content.ofBlockParams(...)
    .build();
```

---

## Structured Output / 结构化输出

The class-based overload auto-derives the JSON schema from your POJO and gives you a typed `.text()` return - no manual schema, no manual parsing.

基于类的重载会从你的 POJO 自动推导 JSON schema，并返回类型化的 `.text()`——无需手动写 schema，无需手动解析。

```java
import com.anthropic.models.messages.StructuredMessageCreateParams;

record Book(String title, String author) {}
record BookList(List<Book> books) {}

StructuredMessageCreateParams<BookList> params = MessageCreateParams.builder()
    .model("claude-opus-5-5")
    .maxTokens(16000L)
    .outputConfig(BookList.class)  // returns a typed builder
    .addUserMessage("List 3 classic novels")
    .build();

client.messages().create(params).content().stream()
    .flatMap(cb -> cb.text().stream())
    .forEach(typed -> {
        // typed.text() returns BookList, not String
        for (Book b : typed.text().books()) System.out.println(b.title());
    });
```

Supports Jackson annotations: `@JsonPropertyDescription`, `@JsonIgnore`, `@ArraySchema(minItems=...)`. Manual schema path: `OutputConfig.builder().format(JsonOutputFormat.builder().schema(...).build())`.

支持 Jackson 注解：`@JsonPropertyDescription`、`@JsonIgnore`、`@ArraySchema(minItems=...)`。手动 schema 路径：`OutputConfig.builder().format(JsonOutputFormat.builder().schema(...).build())`。

---

## Anthropic-Defined Tools / Anthropic 定义的工具

Version-suffixed types; `name`/`type` auto-set by builder. Direct `.addTool()` overloads exist for most tool types; where one is missing (newer or less-common tools - see the advisor note below), wrap via the union type's static factory: `.addTool(BetaToolUnion.of<ToolName>(builder...build()))`. Web search and code execution are server-executed; bash and text editor are client-executed (you handle the `tool_use` locally - see `shared/tool-use-concepts.md`).

带版本后缀的类型；`name`/`type` 由 builder 自动设置。大多数工具类型都有直接的 `.addTool()` 重载；缺少重载时（较新或较少用的工具——见下方 advisor 说明），通过 union 类型的静态工厂包装：`.addTool(BetaToolUnion.of<ToolName>(builder...build()))`。网页搜索与代码执行在服务端执行；bash 与文本编辑器在客户端执行（你在本地处理 `tool_use`——见 `shared/tool-use-concepts.md`）。

```java
import com.anthropic.models.messages.WebSearchTool20260209;
import com.anthropic.models.messages.ToolBash20250124;
import com.anthropic.models.messages.ToolTextEditor20250728;
import com.anthropic.models.messages.CodeExecutionTool20260120;

.addTool(WebSearchTool20260209.builder()
    .maxUses(5L)                              // optional
    .allowedDomains(List.of("example.com"))   // optional
    .build())
.addTool(ToolBash20250124.builder().build())
.addTool(ToolTextEditor20250728.builder().build())
.addTool(CodeExecutionTool20260120.builder().build())
```

Also available: `WebFetchTool20260209`, `MemoryTool20250818`, `ToolSearchToolBm25_20251119`. For the advisor tool, use `BetaAdvisorTool20260301` in the beta namespace with `.addBeta("advisor-tool-2026-03-01")` (server-side; advisor model >= executor model). There is no direct `.addTool(BetaAdvisorTool20260301)` overload on the beta builder - wrap it via the `BetaToolUnion` static factory for the advisor type; if `javac` rejects the specific factory method name, `javap com.anthropic.models.beta.messages.BetaToolUnion | grep -i advisor` shows the exact one.

另有可用工具：`WebFetchTool20260209`、`MemoryTool20250818`、`ToolSearchToolBm25_20251119`。advisor 工具使用 beta 命名空间中的 `BetaAdvisorTool20260301`，并配合 `.addBeta("advisor-tool-2026-03-01")`（服务端执行；advisor 模型版本须 >= 执行模型）。beta builder 上没有直接的 `.addTool(BetaAdvisorTool20260301)` 重载——请通过 advisor 类型对应的 `BetaToolUnion` 静态工厂包装；如果 `javac` 拒绝特定的工厂方法名，可用 `javap com.anthropic.models.beta.messages.BetaToolUnion | grep -i advisor` 查看确切的方法名。

### Beta namespace (MCP, compaction) / Beta 命名空间（MCP、压缩）

For beta-only features use `com.anthropic.models.beta.messages.*` - class names have a `Beta` prefix AND live in the beta package. The beta `MessageCreateParams.Builder` has direct `.addTool(BetaToolBash20250124)` overloads AND `.addMcpServer()`:

仅 beta 提供的功能使用 `com.anthropic.models.beta.messages.*`——类名带 `Beta` 前缀且位于 beta 包中。beta 的 `MessageCreateParams.Builder` 既有直接的 `.addTool(BetaToolBash20250124)` 重载，也有 `.addMcpServer()`：

```java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaToolBash20250124;
import com.anthropic.models.beta.messages.BetaCodeExecutionTool20260120;
import com.anthropic.models.beta.messages.BetaRequestMcpServerUrlDefinition;

MessageCreateParams params = MessageCreateParams.builder()
    .model("claude-opus-5-5")
    .maxTokens(16000L)
    .addBeta("mcp-client-2025-11-20")
    .addTool(BetaToolBash20250124.builder().build())
    .addTool(BetaCodeExecutionTool20260120.builder().build())
    .addMcpServer(BetaRequestMcpServerUrlDefinition.builder()
        .name("my-server")
        .url("https://example.com/mcp")
        .build())
    .addUserMessage("...")
    .build();

client.beta().messages().create(params);
```

`BetaTool*` types are NOT interchangeable with non-beta `Tool*` - pick one namespace per request.

`BetaTool*` 类型与非 beta 的 `Tool*` 不可互换——每次请求只能选一个命名空间。

**Reading server-tool blocks in the response:** `ServerToolUseBlock` has `.id()`, `.name()` (enum), and `._input()` returning raw `JsonValue` - there is NO typed `.input()`. For code execution results, unwrap two levels:

**读取响应中的服务端工具块：**`ServerToolUseBlock` 具有 `.id()`、`.name()`（枚举）以及返回原始 `JsonValue` 的 `._input()`——没有类型化的 `.input()`。对代码执行结果，需要解包两层：

```java
for (ContentBlock block : response.content()) {
    block.serverToolUse().ifPresent(stu -> {
        System.out.println("tool: " + stu.name() + " input: " + stu._input());
    });
    block.codeExecutionToolResult().ifPresent(r -> {
        r.content().resultBlock().ifPresent(result -> {
            System.out.println("stdout: " + result.stdout());
            System.out.println("stderr: " + result.stderr());
            System.out.println("exit: " + result.returnCode());
        });
    });
}
```

---
