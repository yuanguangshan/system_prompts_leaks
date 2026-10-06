<!-- BILINGUAL-EN-ZH -->
# Streaming - TypeScript / 流式传输 - TypeScript

## Quick Start / 快速开始

```typescript
const stream = client.messages.stream({
  model: "claude-opus-5-5",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Write a story" }],
});

for await (const event of stream) {
  if (
    event.type === "content_block_delta" &&
    event.delta.type === "text_delta"
  ) {
    process.stdout.write(event.delta.text);
  }
}
```

---

## Handling Different Content Types / 处理不同的内容类型

> **Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / Opus 4.7 / Opus 4.6:** Use `thinking: {type: "adaptive"}`. On Claude Opus 5.5 and Claude Opus 5 adaptive is also what you get by omitting `thinking` entirely (Claude Opus 5.5 accepts no other setting - `disabled` and `budget_tokens` both 400). On older models, use `thinking: {type: "enabled", budget_tokens: N}` instead.

> **Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / Opus 4.7 / Opus 4.6：**使用 `thinking: {type: "adaptive"}`。在 Claude Opus 5.5 与 Claude Opus 5 上，完全省略 `thinking` 得到的也是 adaptive（Claude Opus 5.5 不接受其他设置——`disabled` 与 `budget_tokens` 都返回 400）。在更旧的模型上，改用 `thinking: {type: "enabled", budget_tokens: N}`。

```typescript
const stream = client.messages.stream({
  model: "claude-opus-5-5",
  max_tokens: 64000,
  thinking: { type: "adaptive", display: "summarized" }, // display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
  messages: [{ role: "user", content: "Analyze this problem" }],
});

for await (const event of stream) {
  switch (event.type) {
    case "content_block_start":
      switch (event.content_block.type) {
        case "thinking":
          console.log("\n[Thinking...]");
          break;
        case "text":
          console.log("\n[Response:]");
          break;
      }
      break;
    case "content_block_delta":
      switch (event.delta.type) {
        case "thinking_delta":
          process.stdout.write(event.delta.thinking);
          break;
        case "text_delta":
          process.stdout.write(event.delta.text);
          break;
      }
      break;
  }
}
```

---

## Streaming with Tool Use (Tool Runner) / 流式 + 工具使用（Tool Runner）

Use the tool runner with `stream: true`. The outer loop iterates over tool runner iterations (messages), the inner loop processes stream events. `betaZodTool()` has no option for `eager_input_streaming`, so spread it onto the returned tool - without it the API buffers each tool-input parameter and `input_json_delta` arrives in one burst at the end (default rule: `shared/tool-use-concepts.md` -> Eager input streaming):

在工具运行器上使用 `stream: true`。外层循环迭代工具运行器的各轮迭代（消息），内层循环处理流事件。`betaZodTool()` 没有提供 `eager_input_streaming` 选项，因此要把它展开附加到返回的工具上——不加的话，API 会缓冲每个工具输入参数，`input_json_delta` 会在最后一次性到达（默认规则见：`shared/tool-use-concepts.md` -> Eager input streaming）：

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const client = new Anthropic();

const getWeather = {
  ...betaZodTool({
    name: "get_weather",
    description: "Get current weather for a location",
    inputSchema: z.object({
      location: z.string().describe("City and state, e.g., San Francisco, CA"),
    }),
    run: async ({ location }) => `72°F and sunny in ${location}`,
  }),
  eager_input_streaming: true, // stream tool input as it is generated
};

let runner = client.beta.messages.toolRunner({
  model: "claude-opus-5-5",
  max_tokens: 64000,
  tools: [getWeather],
  messages: [
    { role: "user", content: "What's the weather in Paris and London?" },
  ],
  stream: true,
});

// With eager input streaming the SDK parses each tool input when its block
// closes. The runner validates it against the Zod schema and never calls
// run() on input that fails; JSON it cannot parse at all rejects the
// iteration. Re-issue only for that case - API errors are rethrown - with a
// cap on consecutive failures. A consumed runner cannot be iterated again,
// so the retry builds a new one from runner.params, which holds the
// conversation so far (the failed turn was never appended), so completed
// tool calls are not re-run.
//
// The runner does not apply the stop-reason rules for you: check
// stop_reason after each turn before the runner runs that turn's tools.
class TruncatedToolInput extends Error {}

for (let attempt = 0; ; attempt++) {
  try {
    // Outer loop: each tool runner iteration
    for await (const messageStream of runner) {
      // Inner loop: stream events for this iteration
      for await (const event of messageStream) {
        switch (event.type) {
          case "content_block_delta":
            switch (event.delta.type) {
              case "text_delta":
                process.stdout.write(event.delta.text);
                break;
              case "input_json_delta":
                // Tool input fragment - arrives immediately with eager streaming
                process.stdout.write(event.delta.partial_json);
                break;
            }
            break;
        }
      }
      const message = await messageStream.finalMessage();
      attempt = 0; // the turn completed; the cap is on consecutive failures
      // A truncated tool input can still pass schema validation, so stop
      // before the runner executes it; a refusal can cut a tool_use off
      // mid-input, so never run that turn's tools. pause_turn is not
      // auto-resumed by the runner: see tool-use.md -> Server tools.
      const hasToolUse = message.content.some((b) => b.type === "tool_use");
      if (message.stop_reason === "max_tokens" && hasToolUse) {
        throw new TruncatedToolInput("tool input truncated; retry with a higher max_tokens");
      }
      if (message.stop_reason === "refusal") break;
      // max_tokens on a plain text answer just ends the loop with the
      // truncated text; the runner returns it as the final message.
    }
    break;
  } catch (err) {
    if (err instanceof Anthropic.APIError || err instanceof TruncatedToolInput || attempt >= 2) {
      throw err;
    }
    console.error("tool input was not parseable JSON, re-issuing the turn");
    runner = client.beta.messages.toolRunner({ ...runner.params });
  }
}
```

With `betaZodTool` the runner validates each tool input against the Zod schema before calling `run` (a `betaTool()` JSON-Schema tool is not validated at runtime - validate inside `run`), which catches malformed input (missing or mistyped fields) the tolerant parser let through; JSON it cannot parse at all rejects the `for await` loop, so wrap it, rethrow API errors, and re-issue with a new runner built from `runner.params` and a cap on consecutive failures - the `tool_use` block never completed, so there is no `tool_use_id` to answer with an `is_error` result. The stop-reason rules are yours to apply, not the runner's: check each turn's `stop_reason` after `finalMessage()` - stop on `max_tokens` when the turn carries a `tool_use` (a truncated input can pass schema validation; a truncated text answer is just returned), stop on `refusal`, and resume `pause_turn` yourself (the runner does not; see `tool-use.md` -> Server tools with the tool runner and `shared/tool-use-concepts.md` -> Eager input streaming).

使用 `betaZodTool` 时，运行器会在调用 `run` 之前用 Zod schema 校验每个工具输入（`betaTool()` 这类 JSON-Schema 工具在运行时不做校验——请在 `run` 内部自行校验），这能捕获宽容解析器放过的畸形输入（缺失或类型错误的字段）；完全无法解析的 JSON 会使 `for await` 循环以拒绝结束，因此要把循环包起来、重新抛出 API 错误，并用基于 `runner.params` 构建的新运行器重新发起该轮，同时限制连续失败次数——`tool_use` 块从未完成，所以不存在可用 `is_error` 结果应答的 `tool_use_id`。停止原因（stop-reason）规则要由你自己应用，运行器不会代劳：在 `finalMessage()` 之后检查每一轮的 `stop_reason`——当该轮带有 `tool_use` 时在 `max_tokens` 处停止（被截断的输入仍可能通过 schema 校验；被截断的纯文本回答会直接返回），在 `refusal` 处停止，并自行恢复 `pause_turn`（运行器不会恢复；见 `tool-use.md` 的"Server tools with the tool runner"以及 `shared/tool-use-concepts.md` 的"Eager input streaming"）。

【评论】这段强调 SDK 的宽容解析（tolerant parser）可能放过结构不完整的工具输入，而截断输入仍可能通过 schema 校验，因此把 stop-reason 检查的责任明确留给调用方——这是在流式工具循环中容易踩坑的点。

---

## Getting the Final Message / 获取最终消息

```typescript
const stream = client.messages.stream({
  model: "claude-opus-5-5",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Hello" }],
});

for await (const event of stream) {
  // Process events...
}

const finalMessage = await stream.finalMessage();
console.log(`Tokens used: ${finalMessage.usage.output_tokens}`);
```

---

## Stream Event Types / 流事件类型

| Event Type            | Description                 | When it fires                     |
| --------------------- | --------------------------- | --------------------------------- |
| `message_start`       | Contains message metadata   | Once at the beginning             |
| `content_block_start` | New content block beginning | When a text/tool_use block starts |
| `content_block_delta` | Incremental content update  | For each token/chunk              |
| `content_block_stop`  | Content block complete      | When a block finishes             |
| `message_delta`       | Message-level updates       | Contains `stop_reason`, usage     |
| `message_stop`        | Message complete            | Once at the end                   |

| 事件类型              | 说明                 | 触发时机                     |
| --------------------- | --------------------------- | --------------------------------- |
| `message_start`       | 包含消息元数据   | 开头时一次             |
| `content_block_start` | 新内容块开始 | 文本/tool_use 块开始时 |
| `content_block_delta` | 增量内容更新  | 每个 token/块              |
| `content_block_stop`  | 内容块完成      | 块结束时             |
| `message_delta`       | 消息级更新       | 包含 `stop_reason` 与用量     |
| `message_stop`        | 消息完成            | 结尾时一次                   |

## Best Practices / 最佳实践

1. **Always flush output** - Use `process.stdout.write()` for immediate display
   **始终刷新输出**——使用 `process.stdout.write()` 立即显示
2. **Handle partial responses** - If the stream is interrupted, you may have incomplete content
   **处理部分响应**——如果流被中断，你可能得到不完整的内容
3. **Track token usage** - The `message_delta` event contains usage information
   **跟踪 token 用量**——`message_delta` 事件包含用量信息
4. **Use `finalMessage()`** - Get the complete `Anthropic.Message` object even when streaming. Don't wrap `.on()` events in `new Promise()` - `finalMessage()` handles all completion/error/abort states internally
   **使用 `finalMessage()`**——即使在流式传输时也能拿到完整的 `Anthropic.Message` 对象。不要把 `.on()` 事件包进 `new Promise()`——`finalMessage()` 会在内部处理所有完成/错误/中止状态
5. **Buffer for web UIs** - Consider buffering a few tokens before rendering to avoid excessive DOM updates
   **为 Web UI 做缓冲**——考虑在渲染前缓冲若干 token，以避免过多的 DOM 更新
6. **Use `stream.on("text", ...)` for deltas** - The `text` event provides just the delta string, simpler than manually filtering `content_block_delta` events
   **用 `stream.on("text", ...)` 获取增量**——`text` 事件只提供增量字符串，比手动过滤 `content_block_delta` 事件更简单
7. **For agentic loops with streaming** - See the [Streaming Manual Loop](./tool-use.md#streaming-manual-loop) section in tool-use.md for combining `stream()` + `finalMessage()` with a tool-use loop
   **流式场景下的智能体循环**——将 `stream()` + `finalMessage()` 与工具使用循环结合，见 tool-use.md 的 [Streaming Manual Loop](./tool-use.md#streaming-manual-loop) 一节

## Raw SSE Format / 原始 SSE 格式

If using raw HTTP (not SDKs), the stream returns Server-Sent Events:

如果使用原始 HTTP（而非 SDK），流返回 Server-Sent Events：

```
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","type":"message",...}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":12}}

event: message_stop
data: {"type":"message_stop"}
```
