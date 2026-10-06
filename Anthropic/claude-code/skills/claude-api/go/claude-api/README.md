<!-- BILINGUAL-EN-ZH -->
# Claude API - Go / Claude API - Go

> **Note:** The Go SDK supports the Claude API and beta tool use with `BetaToolRunner`. Agent SDK is not yet available for Go.

> **注意：** Go SDK 通过 `BetaToolRunner` 支持 Claude API 与 beta 工具调用。Agent SDK 尚未提供 Go 版本。

## Installation / 安装

```bash
go get github.com/anthropics/anthropic-sdk-go
```

## Client Initialization / 客户端初始化

```go
import (
    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/option"
)

// Default (uses ANTHROPIC_API_KEY env var)
client := anthropic.NewClient()

// Explicit API key
client := anthropic.NewClient(
    option.WithAPIKey("your-api-key"),
)
```

---

## Model IDs / 模型 ID

`anthropic.Model` is an alias for `string`, so pass the model as its plain id: `Model: "claude-opus-5-5"`. Default to Claude Opus 5.5 unless the user specifies otherwise; if they ask for Fable or the most powerful model, use `"claude-fable-5-1"`; if they ask for a cheaper tier, use the current generation - `"claude-sonnet-5-5"` or `"claude-haiku-4-5"` (see `shared/models.md` for the full resolution table).

`anthropic.Model` 是 `string` 的别名，因此直接以普通 id 传入模型：`Model: "claude-opus-5-5"`。除非用户另行指定，默认使用 Claude Opus 5.5；如果用户要求 Fable 或最强大的模型，使用 `"claude-fable-5-1"`；如果用户要求更便宜的档位，使用当前一代的 `"claude-sonnet-5-5"` 或 `"claude-haiku-4-5"`（完整的解析表见 `shared/models.md`）。

The SDK also ships typed `anthropic.ModelClaude*` constants, but they lag model launches - a given SDK release may have constants only for previous-generation models. Do not pick a model because it has a typed constant; the string id works for every model on every SDK version. Check the SDK release notes before assuming a typed constant exists for a current model.

SDK 也提供带类型的 `anthropic.ModelClaude*` 常量，但它们滞后于模型发布——某个 SDK 版本可能只包含上一代模型的常量。不要因为某个模型有类型化常量就选它；字符串 id 在任何 SDK 版本上都适用于所有模型。在假定当前模型存在类型化常量之前，先查阅 SDK 的发布说明。

---

## Basic Message Request / 基本消息请求

```go
response, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
    Model:     "claude-opus-5-5",
    MaxTokens: 16000,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("What is the capital of France?")),
    },
})
if err != nil {
    log.Fatal(err)
}
for _, block := range response.Content {
    switch variant := block.AsAny().(type) {
    case anthropic.TextBlock:
        fmt.Println(variant.Text)
    }
}
```

---

## Thinking / 思考

Enable Claude's internal reasoning by setting `Thinking` in `MessageNewParams`. The response will contain `ThinkingBlock` content before the final `TextBlock`.

通过在 `MessageNewParams` 中设置 `Thinking` 来启用 Claude 的内部推理。响应中会在最终的 `TextBlock` 之前包含 `ThinkingBlock` 内容。

**Adaptive thinking is the recommended mode for Claude 4.6+ models.** Claude decides dynamically when and how much to think. Combine with the `effort` parameter for cost-quality control.

**自适应思考（adaptive thinking）是 Claude 4.6+ 模型的推荐模式。**由 Claude 动态决定何时思考、思考多少。可与 `effort` 参数配合进行成本-质量调控。

Derived from `anthropic-sdk-go/message.go` (`ThinkingConfigParamUnion`, `ThinkingConfigAdaptiveParam`).

改编自 `anthropic-sdk-go/message.go`（`ThinkingConfigParamUnion`、`ThinkingConfigAdaptiveParam`）。

```go
// There is no ThinkingConfigParamOfAdaptive helper - construct the union
// struct-literal directly and take the address of the variant.
// display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
adaptive := anthropic.ThinkingConfigAdaptiveParam{Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized}
params := anthropic.MessageNewParams{
    Model:     "claude-opus-5-5",
    MaxTokens: 16000,
    Thinking:  anthropic.ThinkingConfigParamUnion{OfAdaptive: &adaptive},
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("How many r's in strawberry?")),
    },
}

resp, err := client.Messages.New(context.Background(), params)
if err != nil {
    log.Fatal(err)
}

// ThinkingBlock(s) precede TextBlock in content
for _, block := range resp.Content {
    switch b := block.AsAny().(type) {
    case anthropic.ThinkingBlock:
        fmt.Println("[thinking]", b.Thinking)
    case anthropic.TextBlock:
        fmt.Println(b.Text)
    }
}
```

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking (above). `ThinkingConfigParamOfEnabled(budgetTokens)` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Claude Opus 5.5:** thinking is always on - leave `Thinking` unset (or send the adaptive union, which is equivalent); `OfDisabled` returns a 400 at every effort, as does `ThinkingConfigParamOfEnabled`. Control depth with `Effort` under `OutputConfig` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5:** thinking is on by default - leaving `Thinking` unset runs adaptive (the adaptive union is equivalent), unlike Opus 4.8/4.7 where leaving it unset meant no thinking.  
> **Older models:** Use `anthropic.ThinkingConfigParamOfEnabled(N)` (budget must be < `MaxTokens`, min 1024).

> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 与 Sonnet 4.6：**使用自适应思考（见上）。`ThinkingConfigParamOfEnabled(budgetTokens)` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 与 4.7 上已被移除（发送则返回 400）；在 Opus 4.6 与 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5：**思考始终开启——保持 `Thinking` 不设置即可（或发送自适应联合类型，二者等价）；`OfDisabled` 在任何 effort 下都返回 400，`ThinkingConfigParamOfEnabled` 同样如此。请改用 `OutputConfig` 下的 `Effort` 控制深度——该模型默认为 `medium`，而 Claude Opus 5 默认为 `high`。  
> **Claude Opus 5：**思考默认开启——保持 `Thinking` 不设置即运行为自适应（自适应联合类型与之等价）；这一点不同于 Opus 4.8/4.7，后者不设置意味着不思考。  
> **更早的模型：**使用 `anthropic.ThinkingConfigParamOfEnabled(N)`（预算必须 < `MaxTokens`，最小 1024）。

To disable: `anthropic.ThinkingConfigParamUnion{OfDisabled: &anthropic.ThinkingConfigDisabledParam{}}`. On Claude Opus 5 that is accepted only at effort `high` or lower - pairing it with `xhigh`/`max` returns a 400; on Claude Opus 5.5 it returns a 400 at every effort (lower `Effort` instead).

禁用方式：`anthropic.ThinkingConfigParamUnion{OfDisabled: &anthropic.ThinkingConfigDisabledParam{}}`。在 Claude Opus 5 上，仅当 effort 为 `high` 或更低时才被接受——与 `xhigh`/`max` 搭配会返回 400；在 Claude Opus 5.5 上，任何 effort 下都返回 400（应改为调低 `Effort`）。

---

## Prompt Caching / 提示词缓存

`System` is `[]TextBlockParam`; set `CacheControl` on the last block to cache tools + system together. For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

`System` 是 `[]TextBlockParam`；在最后一个块上设置 `CacheControl` 即可将工具与系统提示词一起缓存。放置模式与"静默失效"排查清单见 `shared/prompt-caching.md`。

```go
System: []anthropic.TextBlockParam{{
    Text:         longSystemPrompt,
    CacheControl: anthropic.NewCacheControlEphemeralParam(), // default 5m TTL
}},
```

For 1-hour TTL: `anthropic.CacheControlEphemeralParam{TTL: anthropic.CacheControlEphemeralTTLTTL1h}`. There's also a top-level `CacheControl` on `MessageNewParams` that auto-places on the last cacheable block.

1 小时 TTL 的写法：`anthropic.CacheControlEphemeralParam{TTL: anthropic.CacheControlEphemeralTTLTTL1h}`。`MessageNewParams` 上还有一个顶层 `CacheControl`，会自动放置在最后一个可缓存的块上。

Verify hits via `resp.Usage.CacheCreationInputTokens` / `resp.Usage.CacheReadInputTokens`.

通过 `resp.Usage.CacheCreationInputTokens` / `resp.Usage.CacheReadInputTokens` 验证缓存命中。

---

## Stop Details / 停止详情

When `StopReason` is `anthropic.StopReasonRefusal`, the response includes structured `StopDetails`:

当 `StopReason` 为 `anthropic.StopReasonRefusal`（拒答）时，响应会包含结构化的 `StopDetails`：

```go
if resp.StopReason == anthropic.StopReasonRefusal {
    fmt.Println("Category:", resp.StopDetails.Category)     // e.g. "cyber", "bio", "reasoning_extraction", "frontier_llm", or "" - see docs for the full set
    fmt.Println("Explanation:", resp.StopDetails.Explanation)
}
```

**Refusal fallbacks (Claude Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. In `claude-fable-5-1` code, include `Fallbacks: []anthropic.BetaFallbackParam{{Model: "claude-opus-4-8"}}` plus the `anthropic.AnthropicBetaServerSideFallback2026_06_01` beta on `client.Beta.Messages.New` by default - a policy decline is re-served by the fallback model inside the same call. Not available on Amazon Bedrock, Vertex AI, or Microsoft Foundry - register the client-side middleware there: `option.WithMiddleware(betafallback.BetaRefusalFallbackMiddleware(...))` from `lib/betafallback`, with per-conversation state via `betafallback.WithBetaFallbackState(&betafallback.BetaFallbackState{})`. Full semantics (billing, sticky routing, streaming) and a runnable example: `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason, and the Go SDK repo's `examples/` (WebFetch via `shared/live-sources.md`).

**拒答回退（Claude Fable 5.1）——默认即启用。**回退功能是可选加入的：不配置时，被拒答的请求就直接停止。在 `claude-fable-5-1` 的代码中，默认包含 `Fallbacks: []anthropic.BetaFallbackParam{{Model: "claude-opus-4-8"}}`，并在 `client.Beta.Messages.New` 上加上 `anthropic.AnthropicBetaServerSideFallback2026_06_01` beta——策略性拒绝会在同一次调用内由回退模型重新应答。Amazon Bedrock、Vertex AI 与 Microsoft Foundry 上不可用——这些平台请注册客户端中间件：使用 `lib/betafallback` 中的 `option.WithMiddleware(betafallback.BetaRefusalFallbackMiddleware(...))`，并经由 `betafallback.WithBetaFallbackState(&betafallback.BetaFallbackState{})` 维护会话级状态。完整语义（计费、粘性路由、流式）与可运行示例：`shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason，以及 Go SDK 仓库的 `examples/`（WebFetch 经由 `shared/live-sources.md`）。

【评论】“标题写 opt in by default、正文又说 fallbacks are opt-in”存在措辞上的张力：实际语义是“在示例代码中默认包含回退配置”，但该功能本身仍需开发者显式传入，并非服务端默认行为。

---

## PDF / Document Input / PDF / 文档输入

`NewDocumentBlock` generic helper accepts any source type. `MediaType`/`Type` are auto-set.

`NewDocumentBlock` 通用辅助函数接受任意来源类型。`MediaType`/`Type` 会自动设置。

```go
b64 := base64.StdEncoding.EncodeToString(pdfBytes)

msg := anthropic.NewUserMessage(
    anthropic.NewDocumentBlock(anthropic.Base64PDFSourceParam{Data: b64}),
    anthropic.NewTextBlock("Summarize this document"),
)
```

Other sources: `URLPDFSourceParam{URL: "https://..."}`, `PlainTextSourceParam{Data: "..."}`.

其他来源：`URLPDFSourceParam{URL: "https://..."}`、`PlainTextSourceParam{Data: "..."}`。

---

## Context Editing / Compaction (Beta) / 上下文编辑 / 压缩（Beta）

Use `Beta.Messages.New` with `ContextManagement` on `BetaMessageNewParams`. There is no `NewBetaAssistantMessage` - use `.ToParam()` for the round-trip.

使用 `Beta.Messages.New` 并在 `BetaMessageNewParams` 上配置 `ContextManagement`。没有 `NewBetaAssistantMessage`——往返回填请使用 `.ToParam()`。

```go
params := anthropic.BetaMessageNewParams{
    Model:     "claude-opus-5-5",
    MaxTokens: 16000,
    Betas:     []anthropic.AnthropicBeta{"compact-2026-01-12"},
    ContextManagement: anthropic.BetaContextManagementConfigParam{
        Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
            {OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
        },
    },
    Messages: []anthropic.BetaMessageParam{ /* ... */ },
}

resp, err := client.Beta.Messages.New(ctx, params)
if err != nil {
    log.Fatal(err)
}

// Round-trip: append response to history via .ToParam()
params.Messages = append(params.Messages, resp.ToParam())

// Read compaction blocks from the response
for _, block := range resp.Content {
    if c, ok := block.AsAny().(anthropic.BetaCompactionBlock); ok {
        fmt.Println("compaction summary:", c.Content)
    }
}
```

Other edit types: `BetaClearToolUses20250919EditParam`, `BetaClearThinking20251015EditParam` - these need `Betas: []anthropic.AnthropicBeta{"context-management-2025-06-27"}`, not `compact-2026-01-12`.

其他编辑类型：`BetaClearToolUses20250919EditParam`、`BetaClearThinking20251015EditParam`——它们需要 `Betas: []anthropic.AnthropicBeta{"context-management-2025-06-27"}`，而不是 `compact-2026-01-12`。
