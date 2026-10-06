<!-- BILINGUAL-EN-ZH -->
# Tool Use - C# / 工具使用 - C# 版

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

关于概念性概览（工具定义、工具选择、技巧），参见 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Use / 工具使用

### Defining a tool / 定义工具

`Tool` (NOT `ToolParam`) with an `InputSchema` record. `InputSchema.Type` is auto-set to `"object"` by the constructor - don't set it. `ToolUnion` has an implicit conversion from `Tool`, triggered by the collection expression `[...]`.

使用带有 `InputSchema` 记录的 `Tool`（而不是 `ToolParam`）。`InputSchema.Type` 会由构造函数自动设为 `"object"` —— 无需手动设置。`ToolUnion` 存在来自 `Tool` 的隐式转换，由集合表达式 `[...]` 触发。

```csharp
using System.Text.Json;
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = "claude-opus-5-5",
    MaxTokens = 16000,
    Tools = [
        new Tool {
            Name = "get_weather",
            Description = "Get the current weather in a given location",
            InputSchema = new() {
                Properties = new Dictionary<string, JsonElement> {
                    ["location"] = JsonSerializer.SerializeToElement(
                        new { type = "string", description = "City name" }),
                },
                Required = ["location"],
            },
        },
    ],
    Messages = [new() { Role = Role.User, Content = "Weather in Paris?" }],
};
```

Derived from `anthropic-sdk-csharp/src/Anthropic/Models/Messages/Tool.cs` and `ToolUnion.cs:799` (implicit conversion).

派生自 `anthropic-sdk-csharp/src/Anthropic/Models/Messages/Tool.cs` 与 `ToolUnion.cs:799`（隐式转换）。

See [shared tool use concepts](../../shared/tool-use-concepts.md) for the loop pattern.  
关于循环模式，参见 [shared tool use concepts](../../shared/tool-use-concepts.md)。  
### Converting response content to the follow-up assistant message / 将响应内容转换为后续的 assistant 消息

When echoing Claude's response back in the assistant turn, **there is no `.ToParam()` helper** - manually reconstruct each `ContentBlock` variant as its `*Param` counterpart. Do NOT use `new ContentBlockParam(block.Json)`: it compiles and serializes, but `.Value` stays `null` so `TryPick*`/`Validate()` fail (degraded JSON pass-through, not the typed path).

在 assistant 轮次中回传 Claude 的响应时，**没有 `.ToParam()` 辅助方法** —— 需手动将每个 `ContentBlock` 变体重构为其对应的 `*Param` 类型。不要使用 `new ContentBlockParam(block.Json)`：它能编译和序列化，但 `.Value` 会保持为 `null`，导致 `TryPick*`/`Validate()` 失败（退化为 JSON 透传，而非类型化路径）。

```csharp
using Anthropic.Models.Messages;

Message response = await client.Messages.Create(parameters);

// No .ToParam() - reconstruct per variant. Implicit conversions from each
// *Param type to ContentBlockParam mean no explicit wrapper.
List<ContentBlockParam> assistantContent = [];
List<ContentBlockParam> toolResults = [];
foreach (ContentBlock block in response.Content)
{
    if (block.TryPickText(out TextBlock? text))
    {
        assistantContent.Add(new TextBlockParam { Text = text.Text });
    }
    else if (block.TryPickThinking(out ThinkingBlock? thinking))
    {
        // Signature MUST be preserved - the API rejects tampering
        assistantContent.Add(new ThinkingBlockParam
        {
            Thinking = thinking.Thinking,
            Signature = thinking.Signature,
        });
    }
    else if (block.TryPickRedactedThinking(out RedactedThinkingBlock? redacted))
    {
        assistantContent.Add(new RedactedThinkingBlockParam { Data = redacted.Data });
    }
    else if (block.TryPickToolUse(out ToolUseBlock? toolUse))
    {
        // ToolUseBlock has required Caller; ToolUseBlockParam.Caller is optional - don't copy it
        assistantContent.Add(new ToolUseBlockParam
        {
            ID = toolUse.ID,
            Name = toolUse.Name,
            Input = toolUse.Input,
        });
        // Execute the tool; collect ONE result per tool_use block - the API
        // rejects the follow-up if any tool_use ID lacks a matching tool_result.
        string result = ExecuteYourTool(toolUse.Name, toolUse.Input);
        toolResults.Add(new ToolResultBlockParam
        {
            ToolUseID = toolUse.ID,
            Content = result,
        });
    }
}

// Follow-up: prior messages + assistant echo + user tool_result(s)
List<MessageParam> followUpMessages =
[
    .. parameters.Messages,
    new() { Role = Role.Assistant, Content = assistantContent },
    new() { Role = Role.User, Content = toolResults },
];
```

`ToolResultBlockParam` has no tuple constructor - use the object initializer. `Content` is a string-or-list union; a plain `string` implicitly converts.

`ToolResultBlockParam` 没有元组构造函数 —— 请使用对象初始化器。`Content` 是"字符串或列表"的联合类型；普通 `string` 可隐式转换。

---

## Structured Output / 结构化输出

```csharp
OutputConfig = new OutputConfig {
    Format = new JsonOutputFormat {
        Schema = new Dictionary<string, JsonElement> {
            ["type"] = JsonSerializer.SerializeToElement("object"),
            ["properties"] = JsonSerializer.SerializeToElement(
                new { name = new { type = "string" } }),
            ["required"] = JsonSerializer.SerializeToElement(new[] { "name" }),
        },
    },
},
```

`JsonOutputFormat.Type` is auto-set to `"json_schema"` by the constructor. `Schema` is `required`.

`JsonOutputFormat.Type` 会由构造函数自动设为 `"json_schema"`。`Schema` 是必填的。

---

## Anthropic-Defined Tools / Anthropic 定义的工具

Web search, bash, text editor, and code execution are Anthropic-defined tools with built-in schemas. Web search and code execution are server-executed; bash and text editor are client-executed (you handle the `tool_use` locally - see `shared/tool-use-concepts.md`). Type names are version-suffixed; constructors auto-set `name`/`type`. **Wrap each in `new ToolUnion(...)` explicitly.**

Web 搜索、bash、文本编辑器与代码执行都是带内置 schema 的 Anthropic 定义工具。Web 搜索与代码执行由服务端执行；bash 与文本编辑器由客户端执行（你在本地处理 `tool_use` —— 参见 `shared/tool-use-concepts.md`）。类型名带有版本后缀；构造函数会自动设置 `name`/`type`。**每一个都要显式包装在 `new ToolUnion(...)` 中。**

```csharp
Tools = [
    new ToolUnion(new WebSearchTool20260209()),
    new ToolUnion(new ToolBash20250124()),
    new ToolUnion(new ToolTextEditor20250728()),
    new ToolUnion(new CodeExecutionTool20260120()),
],
```

Also available: `new ToolUnion(new WebFetchTool20260209())`, `new ToolUnion(new MemoryTool20250818())`. `WebSearchTool20260209` optionals: `AllowedDomains`, `BlockedDomains`, `MaxUses`, `UserLocation`.

另有可用项：`new ToolUnion(new WebFetchTool20260209())`、`new ToolUnion(new MemoryTool20250818())`。`WebSearchTool20260209` 的可选参数：`AllowedDomains`、`BlockedDomains`、`MaxUses`、`UserLocation`。

---

## Tool Runner (Beta) / 工具运行器（Beta）

The C# SDK provides a `BetaToolRunner` for automatic tool execution loops. Define tools with raw JSON schemas, and the runner handles the API call -> tool execution -> result feedback loop.

C# SDK 提供了 `BetaToolRunner` 用于自动化的工具执行循环。用原始 JSON schema 定义工具，运行器会处理 API 调用 -> 工具执行 -> 结果反馈的循环。

```csharp
using Anthropic.Models.Beta.Messages;

// Define tools and create params as shown in the Tool Use section above,
// but using the beta namespace types (BetaToolUnion, etc.)
var runner = client.Beta.Messages.ToolRunner(betaParams);

await foreach (BetaMessage message in runner)
{
    foreach (var block in message.Content)
    {
        if (block.TryPickText(out var text))
        {
            Console.WriteLine(text.Text);
        }
    }
}
```

---

