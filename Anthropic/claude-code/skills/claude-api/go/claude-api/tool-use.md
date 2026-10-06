<!-- BILINGUAL-EN-ZH -->
# Tool Use - Go / 工具使用 - Go

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

概念性概述（工具定义、工具选择、相关技巧）请参见 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Use / 工具使用

### Tool Runner (Beta - Recommended) / 工具运行器（Beta - 推荐）

**Beta:** The Go SDK provides `BetaToolRunner` for automatic tool use loops via the `toolrunner` package.

**Beta：** Go SDK 通过 `toolrunner` 包提供 `BetaToolRunner`，用于自动化的工具调用循环。

```go
import (
    "context"
    "fmt"
    "log"

    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/toolrunner"
)

// Define tool input with jsonschema tags for automatic schema generation
type GetWeatherInput struct {
    City string `json:"city" jsonschema:"required,description=The city name"`
}

// Create a tool with automatic schema generation from struct tags
weatherTool, err := toolrunner.NewBetaToolFromJSONSchema(
    "get_weather",
    "Get current weather for a city",
    func(ctx context.Context, input GetWeatherInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
        return anthropic.BetaToolResultBlockParamContentUnion{
            OfText: &anthropic.BetaTextBlockParam{
                Text: fmt.Sprintf("The weather in %s is sunny, 72°F", input.City),
            },
        }, nil
    },
)
if err != nil {
    log.Fatal(err)
}

// Create a tool runner that handles the conversation loop automatically
runner := client.Beta.Messages.NewToolRunner(
    []anthropic.BetaTool{weatherTool},
    anthropic.BetaToolRunnerParams{
        BetaMessageNewParams: anthropic.BetaMessageNewParams{
            Model:     "claude-opus-5-5",
            MaxTokens: 16000,
            Messages: []anthropic.BetaMessageParam{
                anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("What's the weather in Paris?")),
            },
        },
        MaxIterations: 5,
    },
)

// Run until Claude produces a final response
message, err := runner.RunToCompletion(context.Background())
if err != nil {
    log.Fatal(err)
}

// RunToCompletion returns *BetaMessage; content is []BetaContentBlockUnion.
// Narrow via AsAny() switch - note the Beta-namespace types (BetaTextBlock,
// not TextBlock):
for _, block := range message.Content {
    switch block := block.AsAny().(type) {
    case anthropic.BetaTextBlock:
        fmt.Println(block.Text)
    }
}
```

**Key features of the Go tool runner:**

**Go 工具运行器的主要特性：**

- Automatic schema generation from Go structs via `jsonschema` tags
  通过 `jsonschema` 标签从 Go 结构体自动生成 schema
- `RunToCompletion()` for simple one-shot usage
  `RunToCompletion()` 用于简单的一次性用法
- `All()` iterator for processing each message in the conversation
  `All()` 迭代器用于逐条处理对话中的消息
- `NextMessage()` for step-by-step iteration
  `NextMessage()` 用于逐步迭代
- Streaming variant via `NewToolRunnerStreaming()` with `AllStreaming()`
  通过 `NewToolRunnerStreaming()` 配合 `AllStreaming()` 提供流式变体

### Manual Loop / 手动循环

Prefer the tool runner above. For interception, validation, logging, or human-in-the-loop approval, gate inside the tool's run function or step the runner with `NextMessage()`/`All()` and inspect each message (the runner's public `Params` field lets you adjust the next request) - a manual loop is not required. Drop to a manual loop only when you need control the runner does not expose: define tools with `ToolParam`, check `StopReason`, execute tools yourself, and feed `tool_result` blocks back.

优先使用上面的工具运行器。若需拦截、校验、日志记录或人在回路（human-in-the-loop）审批，可以在工具的 run 函数内部设置门控，或用 `NextMessage()`/`All()` 逐步驱动运行器并检查每条消息（运行器公开的 `Params` 字段允许你调整下一个请求）——无需手动循环。只有当需要运行器未暴露的控制能力时才退回手动循环：用 `ToolParam` 定义工具、检查 `StopReason`、自行执行工具并回填 `tool_result` 块。

Derived from `anthropic-sdk-go/examples/tools/main.go`.

改编自 `anthropic-sdk-go/examples/tools/main.go`。

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"

    "github.com/anthropics/anthropic-sdk-go"
)

func main() {
    client := anthropic.NewClient()

    // 1. Define tools. ToolParam.InputSchema uses a map, no struct tags needed.
    addTool := anthropic.ToolParam{
        Name:        "add",
        Description: anthropic.String("Add two integers"),
        InputSchema: anthropic.ToolInputSchemaParam{
            Properties: map[string]any{
                "a": map[string]any{"type": "integer"},
                "b": map[string]any{"type": "integer"},
            },
        },
    }
    // ToolParam must be wrapped in ToolUnionParam for the Tools slice
    tools := []anthropic.ToolUnionParam{{OfTool: &addTool}}

    messages := []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("What is 2 + 3?")),
    }

    for {
        resp, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
            Model:     "claude-opus-5-5",
            MaxTokens: 16000,
            Messages:  messages,
            Tools:     tools,
        })
        if err != nil {
            log.Fatal(err)
        }

        // 2. Append the assistant response to history BEFORE processing tool calls.
        //    resp.ToParam() converts Message -> MessageParam in one call.
        messages = append(messages, resp.ToParam())

        // 3. Walk content blocks. ContentBlockUnion is a flattened struct;
        //    use block.AsAny().(type) to switch on the actual variant.
        toolResults := []anthropic.ContentBlockParamUnion{}
        for _, block := range resp.Content {
            switch variant := block.AsAny().(type) {
            case anthropic.TextBlock:
                fmt.Println(variant.Text)
            case anthropic.ToolUseBlock:
                // 4. Parse the tool input. Use variant.JSON.Input.Raw() to get the
                //    raw JSON - block.Input is json.RawMessage, not the parsed value.
                var in struct {
                    A int `json:"a"`
                    B int `json:"b"`
                }
                if err := json.Unmarshal([]byte(variant.JSON.Input.Raw()), &in); err != nil {
                    log.Fatal(err)
                }
                result := fmt.Sprintf("%d", in.A+in.B)
                // 5. NewToolResultBlock(toolUseID, content, isError) builds the
                //    ContentBlockParamUnion for you. block.ID is the tool_use_id.
                toolResults = append(toolResults,
                    anthropic.NewToolResultBlock(block.ID, result, false))
            }
        }

        // 6. Exit when Claude stops asking for tools
        if resp.StopReason != anthropic.StopReasonToolUse {
            break
        }

        // 7. Tool results go in a user message (variadic: all results in one turn)
        messages = append(messages, anthropic.NewUserMessage(toolResults...))
    }
}
```

**Key API surface:**

**关键 API 一览：**

| Symbol | Purpose |
|---|---|
| `resp.ToParam()` | Convert `Message` response -> `MessageParam` for history |
| `block.AsAny().(type)` | Type-switch on `ContentBlockUnion` variants |
| `variant.JSON.Input.Raw()` | Raw JSON string of tool input (for `json.Unmarshal`) |
| `anthropic.NewToolResultBlock(id, content, isError)` | Build `tool_result` block |
| `anthropic.NewUserMessage(blocks...)` | Wrap tool results as a user turn |
| `anthropic.StopReasonToolUse` | `StopReason` constant to check loop termination |
| `anthropic.ToolUnionParam{OfTool: &t}` | Wrap `ToolParam` in the union for `Tools:` |

| 符号 | 用途 |
|---|---|
| `resp.ToParam()` | 将 `Message` 响应转换为 `MessageParam` 以便追加到历史 |
| `block.AsAny().(type)` | 对 `ContentBlockUnion` 的变体做类型 switch |
| `variant.JSON.Input.Raw()` | 工具输入的原始 JSON 字符串（用于 `json.Unmarshal`） |
| `anthropic.NewToolResultBlock(id, content, isError)` | 构建 `tool_result` 块 |
| `anthropic.NewUserMessage(blocks...)` | 将工具结果包装为用户回合 |
| `anthropic.StopReasonToolUse` | 用于判断循环终止的 `StopReason` 常量 |
| `anthropic.ToolUnionParam{OfTool: &t}` | 将 `ToolParam` 包装进联合类型以用于 `Tools:` |

---

## Anthropic-Defined Tools / Anthropic 定义的工具

Version-suffixed struct names with `Param` suffix. `Name`/`Type` are `constant.*` types - zero value marshals correctly, so `{}` works. Wrap in `ToolUnionParam` with the matching `Of*` field. Web search and code execution are server-executed; bash and text editor are client-executed (you handle the `tool_use` locally - see `shared/tool-use-concepts.md`).

结构体名带版本后缀，并再加 `Param` 后缀。`Name`/`Type` 是 `constant.*` 类型——零值即可正确序列化，因此写 `{}` 即可。使用对应的 `Of*` 字段包装进 `ToolUnionParam`。web search 与 code execution 由服务端执行；bash 与 text editor 由客户端执行（由你在本地处理 `tool_use`——参见 `shared/tool-use-concepts.md`）。

```go
Tools: []anthropic.ToolUnionParam{
    {OfWebSearchTool20260209: &anthropic.WebSearchTool20260209Param{}},
    {OfBashTool20250124: &anthropic.ToolBash20250124Param{}},
    {OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
    {OfCodeExecutionTool20260120: &anthropic.CodeExecutionTool20260120Param{}},
},
```

Also available: `WebFetchTool20260209Param`, `ToolSearchToolBm25_20251119Param`, `ToolSearchToolRegex20251119Param`. For the advisor and memory tools, use `BetaAdvisorTool20260301Param` / `BetaMemoryTool20250818Param` in the beta namespace on `client.Beta.Messages.New`.

另有：`WebFetchTool20260209Param`、`ToolSearchToolBm25_20251119Param`、`ToolSearchToolRegex20251119Param`。advisor 与 memory 工具请使用 beta 命名空间中的 `BetaAdvisorTool20260301Param` / `BetaMemoryTool20250818Param`，挂在 `client.Beta.Messages.New` 上。

### Advisor tool (beta) / Advisor 工具（beta）

Server-side - no tool_result round-trip. The advisor model must be >= the executor (top-level) model; invalid pairs return 400.

服务端执行——无需 tool_result 往返。advisor 模型必须 >= 执行方（顶层）模型；无效组合返回 400。

```go
response, err := client.Beta.Messages.New(ctx, anthropic.BetaMessageNewParams{
    Model:     "claude-sonnet-5-5", // executor
    MaxTokens: 4096,
    Tools: []anthropic.BetaToolUnionParam{
        {OfAdvisorTool20260301: &anthropic.BetaAdvisorTool20260301Param{
            Model: "claude-opus-5-5", // advisor
        }},
    },
    Messages: []anthropic.BetaMessageParam{ /* ... */ },
    Betas:    []anthropic.AnthropicBeta{anthropic.AnthropicBetaAdvisorTool2026_03_01},
})
```

---
