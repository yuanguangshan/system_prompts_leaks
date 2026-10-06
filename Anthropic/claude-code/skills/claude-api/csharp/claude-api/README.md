<!-- BILINGUAL-EN-ZH -->
# Claude API - C# / Claude API - C#

> **Note:** The C# SDK is the official Anthropic SDK for C#. Tool use is supported via the Messages API with a beta `BetaToolRunner` for automatic tool execution loops. The SDK also supports Microsoft.Extensions.AI IChatClient integration with function invocation and Managed Agents (beta).

> **注意：** C# SDK 是 Anthropic 官方的 C# SDK。工具使用通过 Messages API 支持，并提供测试版的 `BetaToolRunner` 用于自动工具执行循环。该 SDK 还支持 Microsoft.Extensions.AI IChatClient 集成（含函数调用）以及 Managed Agents（beta）。

## Namespace Reference / 命名空间参考

Types are organized by namespace. If a type you need isn't shown in an example below, locate it via this table first - don't block on fetching SDK source over the network.

类型按命名空间组织。如果你需要的类型没有出现在下面的示例中，先通过本表定位——不要卡在从网络获取 SDK 源码上。

| `using` | Contains |
|---|---|
| `Anthropic` | `AnthropicClient`, top-level options |
| `Anthropic.Models.Messages` | non-beta request/response types - `MessageCreateParams`, `Model`, `Role`, `ContentBlock`, `TextBlock`, `ToolUseBlock`, `ToolResultBlockParam`, `Tool*` (tool definition classes) |
| `Anthropic.Models.Beta.Messages` | beta-endpoint equivalents - `MessageCreateParams`, `BetaMessage`, `BetaTool*`, `Speed`, `BetaRequestMcpServerUrlDefinition`, context-editing/compaction configs |
| `Anthropic.Models.Beta` | shared beta constants |
| `Anthropic.Models.Beta.Files` | Files API types |
| `Anthropic.Models.Messages.Batches` | Batch API types |
| `Anthropic.Helpers.Beta` | `BetaToolRunner`, beta helper utilities |
| `Anthropic.Exceptions` | `AnthropicApiException`, `AnthropicRateLimitException`, `Anthropic5xxException`, etc. - see `shared/error-codes.md` |
| `Anthropic.Bedrock` / `Anthropic.Vertex` / `Anthropic.Foundry` / `Anthropic.Aws` | platform clients (separate NuGet packages): `AnthropicBedrockMantleClient`, `AnthropicFoundryClient`, `AnthropicAwsClient` |

| `using` | 包含内容 |
|---|---|
| `Anthropic` | `AnthropicClient`、顶层选项 |
| `Anthropic.Models.Messages` | 非 beta 请求/响应类型——`MessageCreateParams`、`Model`、`Role`、`ContentBlock`、`TextBlock`、`ToolUseBlock`、`ToolResultBlockParam`、`Tool*`（工具定义类） |
| `Anthropic.Models.Beta.Messages` | beta 端点对应类型——`MessageCreateParams`、`BetaMessage`、`BetaTool*`、`Speed`、`BetaRequestMcpServerUrlDefinition`、上下文编辑/压缩配置 |
| `Anthropic.Models.Beta` | 共享的 beta 常量 |
| `Anthropic.Models.Beta.Files` | Files API 类型 |
| `Anthropic.Models.Messages.Batches` | Batch API 类型 |
| `Anthropic.Helpers.Beta` | `BetaToolRunner`、beta 辅助工具集 |
| `Anthropic.Exceptions` | `AnthropicApiException`、`AnthropicRateLimitException`、`Anthropic5xxException` 等——见 `shared/error-codes.md` |
| `Anthropic.Bedrock` / `Anthropic.Vertex` / `Anthropic.Foundry` / `Anthropic.Aws` | 平台客户端（独立 NuGet 包）：`AnthropicBedrockMantleClient`、`AnthropicFoundryClient`、`AnthropicAwsClient` |

`client.Messages.*` uses non-beta types; `client.Beta.Messages.*` uses the `Anthropic.Models.Beta.Messages` types. Both namespaces define a `MessageCreateParams` - pick the one matching the client path you call.

`client.Messages.*` 使用非 beta 类型；`client.Beta.Messages.*` 使用 `Anthropic.Models.Beta.Messages` 中的类型。两个命名空间都定义了 `MessageCreateParams`——请选择与你调用的客户端路径匹配的那个。

### Key types per feature / 各功能的关键类型

Write from this table instead of reflecting the SDK assembly. Endpoint column tells you whether to use `client.Messages.*` or `client.Beta.Messages.*`.

直接依据本表编写代码，而不是对 SDK 程序集做反射。Endpoint 列会告诉你该用 `client.Messages.*` 还是 `client.Beta.Messages.*`。

| Feature | Endpoint | Key C# types (namespace per table above) |
|---|---|---|
| User profiles | beta | `client.Beta.UserProfiles.Create(...)` / `.Retrieve(id)` / `.List()`. Pass the returned profile id on the beta messages call. Requires a beta header - check the SDK's beta-headers reference for the current flag. |
| Agent Skills | beta | `BetaContainerParams` (with `Skills = [new BetaSkillParams { ... }]`), `BetaCodeExecutionTool20250825`. `Betas = ["code-execution-2025-08-25"]` (Skills is out of beta - no `skills-2025-10-02`). Download the output via `client.Beta.Files.Download(fileId)`. |
| Advisor tool | beta | `BetaAdvisorTool20260301` - may not be in all SDK releases yet |
| Cache diagnostics | beta | `Diagnostics = new() { PreviousMessageID = ... }`, `BetaCacheControlEphemeral`, `BetaContentBlockParam` |
| Context editing | beta | `ContextManagement = new BetaContextManagementConfig { Edits = [new BetaClearToolUses20250919Edit()] }`. `Betas = ["context-management-2025-06-27"]` (not `compact-2026-01-12` - that's for `BetaCompact20260112Edit`). |
| Memory tool | non-beta | `Tools = [new ToolUnion(new MemoryTool20250818())]` |
| Programmatic tool calling | non-beta | `CodeExecutionTool20260120`, `ToolResultBlockParam`, `ContentBlockParam` |
| Task budgets | beta | `BetaOutputConfig` with `TaskBudget = new BetaTokenTaskBudget { ... }` |
| Tool search | non-beta | `new ToolUnion(new ToolSearchToolRegex20251119 { Type = ToolSearchToolRegex20251119Type.ToolSearchToolRegex20251119 })` - `Type` must be set explicitly. |
| Web search | non-beta | `new ToolUnion(new WebSearchTool20260209())` - the latest variant with dynamic filtering (Claude Fable 5.1 + Claude Opus 5.5 + Claude Opus 5 + Opus 4.8/4.7/4.6 + Claude Sonnet 5.5 + Claude Sonnet 5 + Sonnet 4.6). For older models or Vertex, use `WebSearchTool20250305()` |

| 功能 | 端点 | 关键 C# 类型（命名空间见上表） |
|---|---|---|
| 用户画像（User profiles） | beta | `client.Beta.UserProfiles.Create(...)` / `.Retrieve(id)` / `.List()`。在 beta messages 调用中传入返回的 profile id。需要 beta 标头——当前标志请查阅 SDK 的 beta-headers 参考。 |
| Agent Skills | beta | `BetaContainerParams`（配合 `Skills = [new BetaSkillParams { ... }]`）、`BetaCodeExecutionTool20250825`。`Betas = ["code-execution-2025-08-25"]`（Skills 已正式发布——无需 `skills-2025-10-02`）。输出通过 `client.Beta.Files.Download(fileId)` 下载。 |
| Advisor 工具 | beta | `BetaAdvisorTool20260301`——可能尚未包含在所有 SDK 版本中 |
| 缓存诊断 | beta | `Diagnostics = new() { PreviousMessageID = ... }`、`BetaCacheControlEphemeral`、`BetaContentBlockParam` |
| 上下文编辑 | beta | `ContextManagement = new BetaContextManagementConfig { Edits = [new BetaClearToolUses20250919Edit()] }`。`Betas = ["context-management-2025-06-27"]`（不是 `compact-2026-01-12`——那对应 `BetaCompact20260112Edit`）。 |
| 记忆工具 | 非 beta | `Tools = [new ToolUnion(new MemoryTool20250818())]` |
| 程序化工具调用 | 非 beta | `CodeExecutionTool20260120`、`ToolResultBlockParam`、`ContentBlockParam` |
| 任务预算 | beta | `BetaOutputConfig` 配合 `TaskBudget = new BetaTokenTaskBudget { ... }` |
| 工具搜索 | 非 beta | `new ToolUnion(new ToolSearchToolRegex20251119 { Type = ToolSearchToolRegex20251119Type.ToolSearchToolRegex20251119 })`——`Type` 必须显式设置。 |
| 网页搜索 | 非 beta | `new ToolUnion(new WebSearchTool20260209())`——支持动态过滤的最新变体（Claude Fable 5.1 + Claude Opus 5.5 + Claude Opus 5 + Opus 4.8/4.7/4.6 + Claude Sonnet 5.5 + Claude Sonnet 5 + Sonnet 4.6）。旧模型或 Vertex 请使用 `WebSearchTool20250305()` |

### Discovering type and member names / 发现类型与成员名称

If a type or member you need isn't in the tables above, `strings ~/.nuget/packages/anthropic/*/lib/*/Anthropic.dll | grep -i <term>` is fast and sufficient for locating class and property names. **Do not escalate to a `dotnet run` reflection probe** to dump members precisely - the first compile is slow enough to be backgrounded in many environments, trapping you in a polling loop. Instead, write `Program.cs` using the names `strings | grep` found; if a member name is wrong the compiler error (`error CS1061: 'X' does not contain a definition for 'Y'`) points at it in a few seconds, faster than any reflection probe.

如果你需要的类型或成员不在上面的表中，`strings ~/.nuget/packages/anthropic/*/lib/*/Anthropic.dll | grep -i <term>` 速度快且足以定位类名和属性名。**不要升级为 `dotnet run` 反射探测**来精确转储成员——首次编译在许多环境中慢到会被转入后台，把你困进轮询循环。正确做法是用 `strings | grep` 找到的名称直接编写 `Program.cs`；如果成员名写错了，编译器错误（`error CS1061: 'X' does not contain a definition for 'Y'`）几秒内就会指出问题，比任何反射探测都快。

【评论】这一节是面向自动化代理的效率指令：把编译器报错当作最快的反馈回路，并显式禁止高成本的反射探测，以避免代理陷入轮询等待。

Note that `strings` will not surface wire-format snake_case field names (`output_tokens`, `stop_reason`) - those are stored in the DLL differently. **C# properties are the PascalCase equivalent of the wire field** (`response.Usage.OutputTokens`, `response.StopReason`). If you know the wire field name from the docs, write the PascalCase property and compile; do not probe for the snake_case string.

请注意，`strings` 不会显示出线上格式的 snake_case 字段名（`output_tokens`、`stop_reason`）——它们在 DLL 中的存储方式不同。**C# 属性是对应线上字段的 PascalCase 形式**（`response.Usage.OutputTokens`、`response.StopReason`）。如果你从文档得知的是线上字段名，直接写 PascalCase 属性并编译；不要去探测 snake_case 字符串。

### Minimal working skeleton / 最小可运行骨架

**Write a plain `Program.cs` body** - `using` statements followed by top-level statements, as below. Do **not** add a `#!/usr/bin/env dotnet` shebang or `#:package Anthropic@*` directive: those are .NET file-based-app syntax and fail with `CS1024: Preprocessor directive expected` when the file is compiled via an existing `.csproj`. The standard project setup (per the [C# quickstart](https://platform.claude.com/docs/en/get-started): `dotnet new console` -> `dotnet add package Anthropic` -> edit `Program.cs` -> `dotnet run`) provides the `.csproj` and package reference.

**编写普通的 `Program.cs` 正文**——如上所述，`using` 语句后跟顶级语句。**不要**添加 `#!/usr/bin/env dotnet` shebang 或 `#:package Anthropic@*` 指令：那些是 .NET 基于文件的应用语法，当文件通过现有 `.csproj` 编译时会报 `CS1024: Preprocessor directive expected` 错误。标准的项目设置（参见 [C# 快速入门](https://platform.claude.com/docs/en/get-started)：`dotnet new console` -> `dotnet add package Anthropic` -> 编辑 `Program.cs` -> `dotnet run`）会提供 `.csproj` 和包引用。

Start from this - it compiles as-is. Fill in the feature-specific fields; do not spend turns running reflection or XML-doc inspection to discover type names first.

从这里开始——它可以原样编译通过。填入功能相关的字段即可；不要把回合花在先通过反射或 XML 文档检查去发现类型名上。

```csharp
using System;
using Anthropic;
using Anthropic.Models.Messages;       // or Anthropic.Models.Beta.Messages for beta endpoints

AnthropicClient client = new();

var message = await client.Messages.Create(new MessageCreateParams
{
    Model = "claude-opus-5-5",
    MaxTokens = 1024,
    Messages = [ new() { Role = Role.User, Content = "Hello, Claude" } ],
});

Console.WriteLine(message);
```

For beta features (anything behind an `anthropic-beta` header), use the beta client path and namespace - same overall shape:

对于 beta 功能（任何位于 `anthropic-beta` 标头之后的功能），使用 beta 客户端路径和命名空间——整体结构相同：

```csharp
using System;
using Anthropic;
using Anthropic.Models.Beta.Messages;

AnthropicClient client = new();

var response = await client.Beta.Messages.Create(new MessageCreateParams
{
    Model = "claude-opus-5-5",
    MaxTokens = 4096,
    Betas = ["<beta-flag>"],
    Messages = [ new() { Role = Role.User, Content = "..." } ],
    // Tools = new BetaToolUnion[] { new BetaSomeTool { ... } },   // for tool features
});

Console.WriteLine(response);
```

If a type name the feature needs isn't in this file, write it following the naming pattern in the Namespace Reference above and fix from compiler output - producing a `Program.cs` and iterating beats researching.

如果某功能需要的类型名不在本文件中，按照上文命名空间参考中的命名模式写出它，再依据编译器输出修正——先产出 `Program.cs` 并迭代，胜过反复调研。

### Common C# compile errors / 常见 C# 编译错误

- **CS8803 (top-level statements must precede type declarations):** put any `record`/`class`/`struct` definitions **after** the last top-level statement, at the end of the file. A record defined above `var client = new AnthropicClient()` will not compile.
  **CS8803（顶级语句必须位于类型声明之前）：** 把所有 `record`/`class`/`struct` 定义放在最后一条顶级语句**之后**，即文件末尾。定义在 `var client = new AnthropicClient()` 之前的 record 无法编译。
- **`await foreach` on a `Task<...Page>`:** `client.Models.List()` returns a `Task<ModelListPage>`, which is not directly async-enumerable. Await it first, then iterate: `var page = await client.Models.List(); foreach (var m in page.Items) {...}`. For auto-pagination, check whether the page type exposes `AutoPagingEachAsync()` or similar before reaching for `await foreach`.
  **对 `Task<...Page>` 使用 `await foreach`：** `client.Models.List()` 返回 `Task<ModelListPage>`，它不能直接异步枚举。先 await 再迭代：`var page = await client.Models.List(); foreach (var m in page.Items) {...}`。若要自动分页，先用 `await foreach` 之前检查页面类型是否提供 `AutoPagingEachAsync()` 或类似方法。

## Installation / 安装

```bash
dotnet add package Anthropic
```

## Client Initialization / 客户端初始化

```csharp
using Anthropic;

// Default (uses ANTHROPIC_API_KEY env var)
AnthropicClient client = new();

// Explicit API key (use environment variables - never hardcode keys)
AnthropicClient client = new() {
    ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
};
```

---

## Basic Message Request / 基础消息请求

```csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = "claude-opus-5-5",
    MaxTokens = 16000,
    Messages = [new() { Role = Role.User, Content = "What is the capital of France?" }]
};
var response = await client.Messages.Create(parameters);

// ContentBlock is a union wrapper. .Value unwraps to the variant object,
// then OfType<T> filters to the type you want. Or use the TryPick* idiom
// shown in the Thinking section below.
foreach (var text in response.Content.Select(b => b.Value).OfType<TextBlock>())
{
    Console.WriteLine(text.Text);
}
```

---

## Thinking / 思考（Thinking）

**Adaptive thinking is the recommended mode for Claude 4.6+ models.** Claude decides dynamically when and how much to think.

**自适应思考（adaptive thinking）是 Claude 4.6+ 模型的推荐模式。**何时思考、思考多少由 Claude 动态决定。

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking (below). `new ThinkingConfigEnabled { BudgetTokens = N }` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6：**使用自适应思考（见下文）。`new ThinkingConfigEnabled { BudgetTokens = N }` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 和 4.7 上已被移除（发送则返回 400）；在 Opus 4.6 和 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5:** thinking is always on - omit `Thinking` (or send `ThinkingConfigAdaptive`, which is equivalent); `ThinkingConfigDisabled` returns a 400 at every effort, as does a thinking budget. Control depth with `OutputConfig.Effort` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5.5：**思考始终开启——省略 `Thinking`（或发送等价的 `ThinkingConfigAdaptive`）；`ThinkingConfigDisabled` 在任何 effort 级别都返回 400，思考预算同样如此。应改用 `OutputConfig.Effort` 控制深度——该模型默认 `medium`，而 Claude Opus 5 默认 `high`。  
> **Claude Opus 5:** thinking is on by default - omitting `Thinking` runs adaptive (`ThinkingConfigAdaptive` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `ThinkingConfigDisabled` is accepted only at effort `high` or lower; pairing it with `xhigh`/`max` returns a 400.  
> **Claude Opus 5：**思考默认开启——省略 `Thinking` 即运行自适应模式（`ThinkingConfigAdaptive` 等价），这与 Opus 4.8/4.7 不同，后者省略即意味着不思考。`ThinkingConfigDisabled` 仅在 effort 为 `high` 或更低时被接受；与 `xhigh`/`max` 搭配会返回 400。  
> **Older models:** Use `new ThinkingConfigEnabled { BudgetTokens = N }` (budget must be < `MaxTokens`, min 1024).  
> **更旧的模型：**使用 `new ThinkingConfigEnabled { BudgetTokens = N }`（预算必须小于 `MaxTokens`，最小 1024）。

```csharp
using Anthropic.Models.Messages;

var response = await client.Messages.Create(new MessageCreateParams
{
    Model = "claude-opus-5-5",
    MaxTokens = 16000,
    // ThinkingConfigParam? implicitly converts from the concrete variant classes -
    // no wrapper needed.
    // display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
    Thinking = new ThinkingConfigAdaptive { Display = Display.Summarized },
    Messages =
    [
        new() { Role = Role.User, Content = "Solve: 27 * 453" },
    ],
});

// ThinkingBlock(s) precede TextBlock in Content. TryPick* narrows the union.
foreach (var block in response.Content)
{
    if (block.TryPickThinking(out ThinkingBlock? t))
    {
        Console.WriteLine($"[thinking] {t.Thinking}");
    }
    else if (block.TryPickText(out TextBlock? text))
    {
        Console.WriteLine(text.Text);
    }
}
```

Alternative to `TryPick*`: `.Select(b => b.Value).OfType<ThinkingBlock>()` (same LINQ pattern as the Basic Message example).

`TryPick*` 的替代方案：`.Select(b => b.Value).OfType<ThinkingBlock>()`（与基础消息示例相同的 LINQ 模式）。

---

## Context Editing / Compaction (Beta) / 上下文编辑与压缩（Beta）

**Beta-namespace prefix is inconsistent** (source-verified against `src/Anthropic/Models/Beta/Messages/*.cs` @ 12.9.0). No prefix: `MessageCreateParams`, `MessageCountTokensParams`, `Role`, `Speed`. **Everything else has the `Beta` prefix**: `BetaMessageParam`, `BetaMessage`, `BetaContentBlock`, `BetaToolUseBlock`, all block param types. The unprefixed `Role` WILL collide with `Anthropic.Models.Messages.Role` if you import both namespaces (CS0104). Safest: import only Beta; if mixing, alias the beta `Role`:

**Beta 命名空间的前缀并不一致**（已对照 `src/Anthropic/Models/Beta/Messages/*.cs` @ 12.9.0 源码核实）。无前缀：`MessageCreateParams`、`MessageCountTokensParams`、`Role`、`Speed`。**其余所有类型都带 `Beta` 前缀**：`BetaMessageParam`、`BetaMessage`、`BetaContentBlock`、`BetaToolUseBlock`，以及所有 block 参数类型。如果同时导入两个命名空间，无前缀的 `Role` 会与 `Anthropic.Models.Messages.Role` 冲突（CS0104）。最安全的做法是只导入 Beta；若确需混用，为 beta 的 `Role` 起别名：

```csharp
using Anthropic.Models.Beta.Messages;
using NonBeta = Anthropic.Models.Messages;  // only if you also need non-beta types
// Now: MessageCreateParams, BetaMessageParam, Role (beta's), NonBeta.Role (if needed)
```

`BetaMessage.Content` is `IReadOnlyList<BetaContentBlock>` - a 15-variant discriminated union. Narrow with `TryPick*`. **Response `BetaContentBlock` is NOT assignable to param `BetaContentBlockParam`** - there's no `.ToParam()` in C#. Round-trip by converting each block:

`BetaMessage.Content` 是 `IReadOnlyList<BetaContentBlock>`——一个 15 个变体的可辨识联合。用 `TryPick*` 收窄类型。**响应中的 `BetaContentBlock` 不能赋给参数类型 `BetaContentBlockParam`**——C# 中没有 `.ToParam()`。需要把每个 block 逐个转换以完成往返：

```csharp
using Anthropic.Models.Beta.Messages;

var betaParams = new MessageCreateParams   // no Beta prefix - see unprefixed list above
{
    Model = "claude-opus-5-5",
    MaxTokens = 16000,
    Betas = ["compact-2026-01-12"],
    ContextManagement = new BetaContextManagementConfig
    {
        Edits = [new BetaCompact20260112Edit()],
    },
    Messages = messages,
};
BetaMessage resp = await client.Beta.Messages.Create(betaParams);

foreach (BetaContentBlock block in resp.Content)
{
    if (block.TryPickCompaction(out BetaCompactionBlock? compaction))
    {
        // Content is nullable - compaction can fail server-side
        Console.WriteLine($"compaction summary: {compaction.Content}");
    }
}

// Context-edit metadata lives on a separate nullable field
if (resp.ContextManagement is { } ctx)
{
    foreach (var edit in ctx.AppliedEdits)
        Console.WriteLine($"cleared {edit.ClearedInputTokens} tokens");
}

// ROUND-TRIP: BetaMessageParam.Content is BetaMessageParamContent (a string|list
// union). It implicit-converts from List<BetaContentBlockParam>, NOT from the
// response's IReadOnlyList<BetaContentBlock>. Convert each block:
List<BetaContentBlockParam> paramBlocks = [];
foreach (var b in resp.Content)
{
    if (b.TryPickText(out var t)) paramBlocks.Add(new BetaTextBlockParam { Text = t.Text });
    else if (b.TryPickCompaction(out var c)) paramBlocks.Add(new BetaCompactionBlockParam { Content = c.Content });
    // ... other variants as needed
}
messages.Add(new BetaMessageParam { Role = Role.Assistant, Content = paramBlocks });
```

All 15 `BetaContentBlock.TryPick*` variants: `Text`, `Thinking`, `RedactedThinking`, `ToolUse`, `ServerToolUse`, `WebSearchToolResult`, `WebFetchToolResult`, `CodeExecutionToolResult`, `BashCodeExecutionToolResult`, `TextEditorCodeExecutionToolResult`, `ToolSearchToolResult`, `McpToolUse`, `McpToolResult`, `ContainerUpload`, `Compaction`.

全部 15 个 `BetaContentBlock.TryPick*` 变体：`Text`、`Thinking`、`RedactedThinking`、`ToolUse`、`ServerToolUse`、`WebSearchToolResult`、`WebFetchToolResult`、`CodeExecutionToolResult`、`BashCodeExecutionToolResult`、`TextEditorCodeExecutionToolResult`、`ToolSearchToolResult`、`McpToolUse`、`McpToolResult`、`ContainerUpload`、`Compaction`。

**`BetaToolUseBlock.Input` is `IReadOnlyDictionary<string, JsonElement>`** - index by key then call the `JsonElement` extractor:

**`BetaToolUseBlock.Input` 是 `IReadOnlyDictionary<string, JsonElement>`**——按键索引后调用 `JsonElement` 的取值方法：

```csharp
if (block.TryPickToolUse(out BetaToolUseBlock? tu))
{
    int a = tu.Input["a"].GetInt32();
    string s = tu.Input["name"].GetString()!;
}
```

---

## Effort Parameter / Effort 参数

Effort is nested under `OutputConfig`, NOT a top-level property. `ApiEnum<string, Effort>` has an implicit conversion from the enum, so assign `Effort.High` directly.

Effort 嵌套在 `OutputConfig` 之下，不是顶层属性。`ApiEnum<string, Effort>` 支持从枚举隐式转换，直接赋值 `Effort.High` 即可。

```csharp
OutputConfig = new OutputConfig { Effort = Effort.High },
```

Values: `Effort.Low`, `Effort.Medium`, `Effort.High`, `Effort.Max`. Combine with `Thinking = new ThinkingConfigAdaptive()` for cost-quality control.

可选值：`Effort.Low`、`Effort.Medium`、`Effort.High`、`Effort.Max`。与 `Thinking = new ThinkingConfigAdaptive()` 组合使用可实现成本与质量的权衡。

---

## Prompt Caching / 提示词缓存

`System` takes `MessageCreateParamsSystem?` - a union of `string` or `List<TextBlockParam>`. There is no `SystemTextBlockParam`; use plain `TextBlockParam`. The implicit conversion needs the concrete `List<TextBlockParam>` type (array literals won't convert). For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

`System` 接受 `MessageCreateParamsSystem?`——`string` 或 `List<TextBlockParam>` 的联合类型。不存在 `SystemTextBlockParam`，直接使用 `TextBlockParam`。隐式转换要求具体的 `List<TextBlockParam>` 类型（数组字面量无法转换）。关于放置模式与静默失效排查清单，见 `shared/prompt-caching.md`。

```csharp
System = new List<TextBlockParam> {
    new() {
        Text = longSystemPrompt,
        CacheControl = new CacheControlEphemeral(),  // auto-sets Type = "ephemeral"
    },
},
```

Optional `Ttl` on `CacheControlEphemeral`: `new() { Ttl = Ttl.Ttl1h }` or `Ttl.Ttl5m`. `CacheControl` also exists on `Tool.CacheControl` and top-level `MessageCreateParams.CacheControl`.

`CacheControlEphemeral` 上有可选的 `Ttl`：`new() { Ttl = Ttl.Ttl1h }` 或 `Ttl.Ttl5m`。`CacheControl` 也存在于 `Tool.CacheControl` 和顶层的 `MessageCreateParams.CacheControl` 上。

Verify hits via `response.Usage.CacheCreationInputTokens` / `response.Usage.CacheReadInputTokens`.

通过 `response.Usage.CacheCreationInputTokens` / `response.Usage.CacheReadInputTokens` 验证缓存命中。

---

## Token Counting / Token 计数

```csharp
MessageTokensCount result = await client.Messages.CountTokens(new MessageCountTokensParams {
    Model = "claude-opus-5-5",
    Messages = [new() { Role = Role.User, Content = "Hello" }],
});
long tokens = result.InputTokens;
```

`MessageCountTokensParams.Tools` uses a different union type (`MessageCountTokensTool`) than `MessageCreateParams.Tools` (`ToolUnion`) - if you're passing tools, the compiler will tell you when it matters.

`MessageCountTokensParams.Tools` 使用的联合类型（`MessageCountTokensTool`）与 `MessageCreateParams.Tools`（`ToolUnion`）不同——如果你传入了工具，编译器会在有关系时提醒你。

---

## PDF / Document Input / PDF 与文档输入

`DocumentBlockParam` takes a `DocumentBlockParamSource` union: `Base64PdfSource` / `UrlPdfSource` / `PlainTextSource` / `ContentBlockSource`. `Base64PdfSource` auto-sets `MediaType = "application/pdf"` and `Type = "base64"`.

`DocumentBlockParam` 接受 `DocumentBlockParamSource` 联合类型：`Base64PdfSource` / `UrlPdfSource` / `PlainTextSource` / `ContentBlockSource`。`Base64PdfSource` 会自动设置 `MediaType = "application/pdf"` 和 `Type = "base64"`。

```csharp
new MessageParam {
    Role = Role.User,
    Content = new List<ContentBlockParam> {
        new DocumentBlockParam { Source = new Base64PdfSource { Data = base64String } },
        new TextBlockParam { Text = "Summarize this PDF" },
    },
}
```

---

## Fast Mode (Beta) / 快速模式（Beta）

```csharp
var response = await client.Beta.Messages.Create(new MessageCreateParams {
    Model = "claude-opus-5-5", MaxTokens = 4096,
    Speed = Speed.Fast,
    Betas = ["fast-mode-2026-02-01"],
    Messages = [...],
});
```

## Models API / Models API

```csharp
var page = await client.Models.List();
foreach (var m in page.Items) { Console.WriteLine(m.ID); }
var one = await client.Models.Retrieve("claude-opus-5-5");
```

## Long Output (128k) + Prefill / 长输出（128k）与预填充

Set `MaxTokens = 128000` on `client.Messages` and use the streaming path (see `streaming.md`). On Claude 4+ models, 128k output is native - no `output-128k-*` beta header or beta namespace is needed.

在 `client.Messages` 上设置 `MaxTokens = 128000` 并使用流式路径（见 `streaming.md`）。在 Claude 4+ 模型上，128k 输出是原生能力——无需 `output-128k-*` beta 标头或 beta 命名空间。

**Prefilling the assistant message** (putting a trailing `Role.Assistant` message in the input) is **not supported** on Claude Fable 5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6 - requests return a 400. Use `OutputConfig.Format` (structured outputs) instead.

**预填充助手消息**（在输入末尾放入一条 `Role.Assistant` 消息）在 Claude Fable 5.1、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6 上**不受支持**——请求会返回 400。请改用 `OutputConfig.Format`（结构化输出）。

## Stop Details / 停止原因详情

When `StopReason` is `"refusal"`, the response includes structured `StopDetails`:

当 `StopReason` 为 `"refusal"`（拒答）时，响应包含结构化的 `StopDetails`：

```csharp
if (response.StopReason == "refusal" && response.StopDetails is { } details)
{
    Console.WriteLine($"Category: {details.Category}");
    Console.WriteLine($"Explanation: {details.Explanation}");
}
```

**Refusal fallbacks (Claude Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. In `claude-fable-5-1` code, include `Fallbacks = [new(Model.ClaudeOpus4_8)]` plus the `AnthropicBeta.ServerSideFallback2026_06_01` beta on the beta messages call by default - a policy decline is re-served by the fallback model inside the same call. Not available on Amazon Bedrock, Vertex AI, or Microsoft Foundry - use the client-side handler there: `new AnthropicClient { Handlers = [new BetaRefusalFallbackHandler { Fallbacks = [new(Model.ClaudeOpus4_8)] }] }` (namespace `Anthropic.Helpers`), with per-conversation state via `BetaFallbackState.Create()` scoped with `using (fallbackState.Use()) { ... }`. Full semantics (billing, sticky routing, streaming) and a runnable example: `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason, and the C# SDK repo's `examples/` (WebFetch via `shared/live-sources.md`).

**拒答回退（Claude Fable 5.1）——默认需主动启用。**回退是可选功能：不启用时，被拒的请求直接终止。在 `claude-fable-5-1` 代码中，默认应在 beta messages 调用中加入 `Fallbacks = [new(Model.ClaudeOpus4_8)]` 以及 `AnthropicBeta.ServerSideFallback2026_06_01` beta——策略性拒绝会在同一次调用内由回退模型重新处理。Amazon Bedrock、Vertex AI 或 Microsoft Foundry 上不可用——这些平台请使用客户端侧处理器：`new AnthropicClient { Handlers = [new BetaRefusalFallbackHandler { Fallbacks = [new(Model.ClaudeOpus4_8)] }] }`（命名空间 `Anthropic.Helpers`），并通过 `BetaFallbackState.Create()` 配合 `using (fallbackState.Use()) { ... }` 作用域维护每轮对话的状态。完整语义（计费、粘性路由、流式）与可运行示例：`shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason，以及 C# SDK 仓库的 `examples/`（WebFetch 见 `shared/live-sources.md`）。

【评论】"refusal 回退"让策略性拒绝的请求在同一次调用内由回退模型重新处理，属于把安全判定与可用性编排结合起来的较新机制。

---

## Managed Agents (Beta) / Managed Agents（Beta）

The C# SDK supports Managed Agents via `client.Beta.Agents`, `client.Beta.Sessions`, `client.Beta.Environments`, and related namespaces. See `shared/managed-agents-overview.md` for the architecture and `curl/managed-agents.md` for the wire-level reference.

C# SDK 通过 `client.Beta.Agents`、`client.Beta.Sessions`、`client.Beta.Environments` 及相关命名空间支持 Managed Agents。架构说明见 `shared/managed-agents-overview.md`，线路级参考见 `curl/managed-agents.md`。
