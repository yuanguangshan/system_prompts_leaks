<!-- BILINGUAL-EN-ZH -->
# Tool Use Concepts / 工具调用核心概念

This file covers the conceptual foundations of tool use with the Claude API. For language-specific code examples, see the `python/`, `typescript/`, or other language folders. For decision heuristics on which tools to expose, how to manage context in long-running agents, and caching strategy, see `agent-design.md`.

本文件介绍 Claude API 工具调用的概念基础。特定语言的代码示例请参见 `python/`、`typescript/` 或其他语言文件夹。关于应暴露哪些工具、如何在长时运行代理中管理上下文以及缓存策略的决策经验，请参见 `agent-design.md`。

## User-Defined Tools / 用户自定义工具

### Tool Definition Structure / 工具定义结构

> **Note:** When using the Tool Runner (beta), tool schemas are generated automatically from your function signatures (Python), Zod schemas (TypeScript), annotated classes (Java), `jsonschema` struct tags (Go), or `BaseTool` subclasses (Ruby). The raw JSON schema format below is for the manual approach - including PHP's `BetaRunnableTool`, which wraps a run closure around a hand-written schema - or SDKs without tool runner support.

> **注意：** 使用 Tool Runner（beta）时，工具 schema 会根据你的函数签名（Python）、Zod schema（TypeScript）、注解类（Java）、`jsonschema` 结构体标签（Go）或 `BaseTool` 子类（Ruby）自动生成。下方的原始 JSON schema 格式适用于手动方式——包括 PHP 的 `BetaRunnableTool`（它把运行闭包包在手写的 schema 外面）——以及不支持工具运行器的 SDK。

Each tool requires a name, description, and JSON Schema for its inputs:

每个工具都需要一个名称、一段描述，以及针对其输入的 JSON Schema：

```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state, e.g., San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["location"]
  }
}
```

**Best practices for tool definitions:**

**工具定义的最佳实践：**

- Use clear, descriptive names (e.g., `get_weather`, `search_database`, `send_email`)
  使用清晰、具有描述性的名称（例如 `get_weather`、`search_database`、`send_email`）
- Write detailed descriptions - Claude uses these to decide when to use the tool. Be **prescriptive about *when* to call it**, not just what it does (e.g. "Call this when the user asks about current prices or recent events"). On recent Opus models, which reach for tools more conservatively, trigger conditions in the description give measurable lift in should-call rate.
  编写详细的描述——Claude 依靠描述来决定何时使用该工具。要**明确规定*何时*调用它**，而不仅仅是它做什么（例如"当用户询问当前价格或近期事件时调用此工具"）。在较新的 Opus 模型上（这类模型调用工具更为保守），在描述中写明触发条件可以切实提升"应调用率"。
- Include descriptions for each property
  为每个属性编写描述
- Use `enum` for parameters with a fixed set of values
  对取值集合固定的参数使用 `enum`
- Mark truly required parameters in `required`; make others optional with defaults
  把真正必需的参数标记在 `required` 中；其余参数设为可选并提供默认值

---

### Eager input streaming (default for streaming requests with client tools) / 输入预流式（对于带客户端工具的流式请求为默认行为）

By default the API **buffers and validates each tool-input parameter** before it emits any `input_json_delta` for it. For a small `{"location": "Paris"}` that is invisible; for a tool that takes a file body, a code block, or a document, nothing arrives until the whole parameter is generated (a 20K-token parameter is a multi-minute silent gap on the stream). Setting `eager_input_streaming: true` on the tool turns off that buffering for that tool: fragments stream as they are generated, the first fragment arrives immediately, and the fragments are longer. The events are the same (`content_block_start` -> `input_json_delta` × N -> `content_block_stop`) - only the timing and the validation guarantee change.

默认情况下，API 在为某个工具输入参数发出任何 `input_json_delta` 之前，会**对该参数进行缓冲与校验**。对于 `{"location": "Paris"}` 这样的小参数，这一机制不可感知；但对于接收文件内容、代码块或文档的工具参数，在整个参数生成完毕之前不会有任何数据到达（一个 20K token 的参数意味着流上数分钟的静默空窗）。在工具上设置 `eager_input_streaming: true` 可为该工具关闭这种缓冲：片段随生成即流出，第一个片段立即到达，且片段更长。事件序列不变（`content_block_start` -> `input_json_delta` × N -> `content_block_stop`）——改变的只是时序与校验保证。

**Default rule:** when a request is streamed (`client.messages.stream(...)`, `stream=True`, tool runner with streaming on) and defines user-defined tools, set `eager_input_streaming: true` on each of those tools. Do not set it on non-streaming requests (it is ignored), on server tools (`web_search`, `code_execution`, `mcp_toolset`, etc. - it is not a valid field there), or when the client has no way to handle invalid JSON.

**默认规则：** 当请求以流式发出（`client.messages.stream(...)`、`stream=True`、开启了流式的工具运行器）并定义了用户自定义工具时，为其中每个工具设置 `eager_input_streaming: true`。不要在非流式请求上设置（会被忽略）、不要在服务端工具上设置（`web_search`、`code_execution`、`mcp_toolset` 等——该字段在那里不合法）、也不要在客户端无法处理非法 JSON 时设置。

【评论】默认的缓冲行为把 JSON 完整性校验放在服务端；开启 eager 模式换取更早的首字节到达，代价是把参数校验责任转移到客户端，文档用大量防御性条款来补偿这一转移。

```json
{
  "name": "write_file",
  "description": "Write text to a file",
  "eager_input_streaming": true,
  "input_schema": {
    "type": "object",
    "properties": {
      "path": {"type": "string"},
      "contents": {"type": "string", "description": "Full file contents"}
    },
    "required": ["path", "contents"]
  }
}
```

Tool runner: Python `@beta_tool(eager_input_streaming=True)` passes it through. TypeScript `betaZodTool()` has no option for it - spread the field onto the returned tool: `{ ...betaZodTool({ name, description, inputSchema, run }), eager_input_streaming: true }`. Go/Java/Ruby/C#/PHP: the field is on the tool param type (`EagerInputStreaming`, `.eagerInputStreaming(true)`, `eager_input_streaming:`).

工具运行器：Python 的 `@beta_tool(eager_input_streaming=True)` 可直接透传该选项。TypeScript 的 `betaZodTool()` 没有对应选项——把该字段展开附加到返回的工具上：`{ ...betaZodTool({ name, description, inputSchema, run }), eager_input_streaming: true }`。Go/Java/Ruby/C#/PHP：该字段位于工具参数类型上（`EagerInputStreaming`、`.eagerInputStreaming(true)`、`eager_input_streaming:`）。

**What you give up, and how to handle it.** Without buffering the API does not validate or coerce the parameter, so the accumulated `partial_json` can be (a) cut off at `max_tokens` mid-parameter or (b) invalid JSON the model emitted. Both SDKs accumulate it with a *tolerant* partial-JSON parser, so malformed input often comes back as a silently truncated object (an unescaped inner quote ends the string early; trailing garbage is dropped) rather than an exception. Do not rely on an exception. Always:

**你会失去什么，以及如何应对。** 没有缓冲时，API 不再对参数做校验或类型矫正，因此累积的 `partial_json` 可能出现两种情况：(a) 参数中途因 `max_tokens` 被截断，或 (b) 模型输出了非法 JSON。两个 SDK 都用*宽容的*部分 JSON 解析器来累积数据，因此格式错误的输入往往以静默截断的对象形式返回（未转义的内部引号会提前结束字符串；尾部的垃圾数据被丢弃），而不是抛出异常。不要依赖异常。务必做到：

1. **Validate the parsed input against the tool's schema before running the tool.** The typed runner helpers do this for you and never call `run` on input that fails: TS `betaZodTool` (Zod), Python `@beta_tool` on a function with typed parameters, Java's annotated classes, Go's struct tags, Ruby's `BaseTool`. The raw JSON-Schema helpers do **not** validate at runtime - TS `betaTool()` and PHP's `BetaRunnableTool` hand `run` whatever was parsed - so with those, validate inside `run` (or leave `eager_input_streaming` off for that tool). In a manual loop, validate yourself (`schema.safeParse(block.input)` in TS, a pydantic model or explicit type checks in Python) and treat a failure exactly like invalid JSON. A raw SSE / cURL client should `JSON.parse` the accumulated fragments strictly and then validate.
   **在运行工具之前，先对照工具 schema 校验解析出的输入。** 带类型的运行器辅助函数会替你完成这一步，且绝不会对校验失败的输入调用 `run`：TS 的 `betaZodTool`（Zod）、Python 中带类型参数函数上的 `@beta_tool`、Java 的注解类、Go 的结构体标签、Ruby 的 `BaseTool`。原始 JSON-Schema 辅助函数在运行时**不做**校验——TS 的 `betaTool()` 和 PHP 的 `BetaRunnableTool` 会把解析出的任何内容直接交给 `run`——因此使用它们时，请在 `run` 内部自行校验（或为该工具关闭 `eager_input_streaming`）。在手动循环中，请自行校验（TS 用 `schema.safeParse(block.input)`，Python 用 pydantic 模型或显式类型检查），并把校验失败完全当作非法 JSON 处理。裸 SSE / cURL 客户端应对累积的片段做严格的 `JSON.parse`，然后再校验。
2. Check `stop_reason == "max_tokens"` when a `tool_use` block is present: a truncated input usually parses as a valid partial object, so this is what catches it; retry with a higher `max_tokens` rather than running the tool. Also stop on `stop_reason == "refusal"` - a refusal can cut a `tool_use` off mid-input, so never execute that turn's tools.
   当存在 `tool_use` 块时检查 `stop_reason == "max_tokens"`：被截断的输入通常仍能解析为合法的部分对象，所以这一检查才能真正抓住问题；应以更高的 `max_tokens` 重试，而不是运行该工具。同时在 `stop_reason == "refusal"` 时停止——拒答可能在输入中途切断 `tool_use`，因此绝不要执行该轮的工具。
3. Still guard the exceptions the SDKs do raise for JSON they cannot parse at all: **Python** raises `ValueError` from the stream iterator (wrap the `with client.messages.stream(...)` block); **TypeScript** materializes the input at `content_block_stop`, so the error rejects whatever you are awaiting at that moment - the `for await (const event of stream)` loop if you iterate events, otherwise `await stream.finalMessage()` - so wrap the whole consumption of the stream (iteration and final read together), as the Python guidance wraps the whole `with` block; the **tool runners** surface the same error from their iteration (wrap the loop). Catch only that error - rethrow the SDK's typed API errors (`RateLimitError`, `AuthenticationError`, ...) so an auth or rate-limit failure is not mistaken for bad JSON - and cap retries.
   对于 SDK 在遇到完全无法解析的 JSON 时仍会抛出的异常，也要加以防护：**Python** 会从流迭代器抛出 `ValueError`（把 `with client.messages.stream(...)` 块包进异常处理）；**TypeScript** 在 `content_block_stop` 时实例化输入，因此该错误会拒绝（reject）你当时正在 await 的东西——如果你在迭代事件，就是 `for await (const event of stream)` 循环，否则就是 `await stream.finalMessage()`——所以要包住流的整个消费过程（迭代与最终读取一起包住），正如 Python 的建议是包住整个 `with` 块；**工具运行器**会从其迭代中抛出同样的错误（包住循环）。只捕获这一个错误——SDK 的带类型 API 错误（`RateLimitError`、`AuthenticationError` 等）要重新抛出，以免认证或限流失败被误判为非法 JSON——并限制重试次数。
4. When validation fails and you still hold the `tool_use` block (manual loop after `finalMessage()` / `get_final_message()`, raw SSE), do not run the tool; return the raw text to Claude as an error result so it can retry:
   当校验失败而你仍持有 `tool_use` 块时（在 `finalMessage()` / `get_final_message()` 之后的手动循环、裸 SSE），不要运行工具；把原始文本作为错误结果返回给 Claude，让它重试：

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01...",
  "is_error": true,
  "content": "{\"INVALID_JSON\": \"<the unparseable input you received>\"}"
}
```

When the SDK raised before the block completed (Python stream, either tool runner), there is no `tool_use_id` to answer, so re-issue the request instead.

如果 SDK 在块完成之前就抛出了异常（Python 流、任一工具运行器），此时没有可应答的 `tool_use_id`，因此应改为重新发起请求。

Build that wrapper with the JSON library (not string concatenation) so quotes in the bad input are escaped. With the buffered default the server would have delivered the same broken parameter as a single string value instead; eager mode moves that failure to the client, it does not create it.

请使用 JSON 库（而不是字符串拼接）来构造该包装对象，这样坏输入中的引号才能被正确转义。在默认的缓冲模式下，服务端本会把同样损坏的参数作为单个字符串值传递过来；eager 模式只是把这一失败点移到了客户端，并不是它制造了失败。

**Availability:** Claude API, Claude Platform on AWS, Vertex AI, and Microsoft Foundry for all current models (`shared/platform-availability.md`). On Amazon Bedrock only the newer serving stack accepts the field (Opus 4.7 / 4.8 / 5, Fable 5, Sonnet 4.6 / 5); older Bedrock deployments (Opus 4.5 / 4.6, Sonnet 4.0 / 4.5, Haiku 4.5) return 400 on the unknown field - drop it there. `shared/platform-availability.md` is the source of truth for this list. Any proxy or gateway in front of the API may likewise reject it; if the user's code points at a custom `base_url`, leave it off unless they confirm the upstream is the real API.

**可用性：** Claude API、AWS 上的 Claude Platform、Vertex AI 以及 Microsoft Foundry 上的所有当前模型均支持（见 `shared/platform-availability.md`）。在 Amazon Bedrock 上，只有较新的服务栈接受该字段（Opus 4.7 / 4.8 / 5、Fable 5、Sonnet 4.6 / 5）；较旧的 Bedrock 部署（Opus 4.5 / 4.6、Sonnet 4.0 / 4.5、Haiku 4.5）会对未知字段返回 400——在这些部署上请去掉该字段。`shared/platform-availability.md` 是此列表的最终权威来源。位于 API 前面的任何代理或网关同样可能拒绝该字段；如果用户的代码指向自定义 `base_url`，除非其确认上游是真正的 API，否则不要启用该字段。

---

### Tool Choice Options / 工具选择选项

Control when Claude uses tools:

控制 Claude 何时使用工具：

| Value                             | Behavior                                      |
| --------------------------------- | --------------------------------------------- |
| `{"type": "auto"}`                | Claude decides whether to use tools (default) |
| `{"type": "any"}`                 | Claude must use at least one tool             |
| `{"type": "tool", "name": "..."}` | Claude must use the specified tool            |
| `{"type": "none"}`                | Claude cannot use tools                       |

| 取值                              | 行为                                |
| --------------------------------- | ----------------------------------- |
| `{"type": "auto"}`                | Claude 自行决定是否使用工具（默认） |
| `{"type": "any"}`                 | Claude 必须至少使用一个工具         |
| `{"type": "tool", "name": "..."}` | Claude 必须使用指定的工具           |
| `{"type": "none"}`                | Claude 不能使用工具                 |

Any `tool_choice` value can also include `"disable_parallel_tool_use": true` to force Claude to use at most one tool per response. By default, Claude may request multiple tool calls in a single response.

任何 `tool_choice` 取值都可以附带 `"disable_parallel_tool_use": true`，迫使 Claude 在每次响应中最多使用一个工具。默认情况下，Claude 可以在单个响应中请求多个工具调用。

**Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5.5, and Claude Sonnet 5.5 reject forced tool use:** `{"type": "any"}` and `{"type": "tool", "name": ...}` return a 400 there (`tool_choice: type "tool" and "any" are not supported for this model.` - on `count_tokens` and Batches too). It is a model-specific restriction (Claude Fable 5 and Claude Opus 5 accept them). Because `auto` does not guarantee a call, check that one was made and retry if it wasn't. Use `{"type": "auto"}` and state the expectation in the prompt ("Use the get_weather tool to answer") - `strict: true` on the tool keeps the schema-valid-arguments guarantee `any` gave you - or structured outputs (`output_config.format`) when the forced call only existed to extract JSON. `auto` and `none` are unaffected; `disable_parallel_tool_use` with `auto` still means at most one call (the "exactly one" combination with `any`/`tool` is gone). Combining `tool_choice` `any` with `strict: true` applies only on models that support forced tool use. See `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5.

**Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5 与 Claude Sonnet 5.5 拒绝强制工具调用：** 在这些模型上，`{"type": "any"}` 与 `{"type": "tool", "name": ...}` 会返回 400（错误信息为 `tool_choice: type "tool" and "any" are not supported for this model.`——`count_tokens` 和 Batches 上同样如此）。这是模型级别的限制（Claude Fable 5 与 Claude Opus 5 仍然接受它们）。由于 `auto` 不保证一定发生调用，请检查是否确实发生了调用，没有则重试。使用 `{"type": "auto"}` 并在提示词中说明预期（"Use the get_weather tool to answer"）——工具上的 `strict: true` 可以保留 `any` 原本给你的"参数符合 schema"保证——如果强制调用只是为了提取 JSON，也可以改用结构化输出（`output_config.format`）。`auto` 与 `none` 不受影响；`disable_parallel_tool_use` 配合 `auto` 仍表示最多一次调用（与 `any`/`tool` 组合出的"恰好一次"已不存在）。`tool_choice` `any` 与 `strict: true` 的组合只适用于支持强制工具调用的模型。参见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5。

【评论】新一代模型取消服务端强制工具调用、改由提示词表达调用意图，是"硬约束"向"模型自主性"的取舍变化；依赖 `any` 保证结构化参数的存量代码迁移时需要自行补上校验。

---

### Tool Runner vs Manual Loop / 工具运行器与手动循环

**Tool Runner (Recommended):** The SDK's tool runner handles the agentic loop automatically - it calls the API, detects tool use requests, executes your tool functions, feeds results back to Claude, and repeats until Claude stops calling tools. Available in Python, TypeScript, Java, Go, Ruby, PHP, and C# SDKs (beta). The Python SDK also provides MCP conversion helpers (`anthropic.lib.tools.mcp`) to convert MCP tools, prompts, and resources for use with the tool runner - see `python/claude-api/tool-use.md` for details. **Default to the tool runner** for any custom-tool agent.

**Tool Runner（推荐）：** SDK 的工具运行器会自动处理代理式循环——它调用 API、检测工具使用请求、执行你的工具函数、把结果回传给 Claude，并不断重复直到 Claude 停止调用工具。可用于 Python、TypeScript、Java、Go、Ruby、PHP 和 C# SDK（beta）。Python SDK 还提供 MCP 转换辅助工具（`anthropic.lib.tools.mcp`），可将 MCP 工具、提示词和资源转换为工具运行器可用的形式——详见 `python/claude-api/tool-use.md`。对于任何自定义工具代理，**默认使用工具运行器**。

**The tool runner is not a black box - "I need control" is rarely a reason to drop to the manual loop.** Each iteration yields the assistant message *before* the tools run and lets you intervene, so most "fine-grained control" needs are covered without hand-writing the loop:

**工具运行器并不是黑盒——"我需要控制权"很少是退回手动循环的理由。** 每次迭代都会在工具运行*之前*产出助手消息并允许你介入，因此大多数"细粒度控制"需求无需手写循环即可满足：

- **Human-in-the-loop approval / gating** - gate in the tool's run function (return a "user declined" result instead of executing), or inspect the tool call in the yielded message and override the pending request with `set_messages_params()` / `setMessagesParams()` / `append_messages()` / `pushMessages()` to allow or deny *before* the tool executes. The runner runs your function automatically only if you don't intervene.
  **人工审批 / 门控**——在工具的 run 函数内做门控（返回"用户已拒绝"的结果而不执行），或检查产出消息中的工具调用，并用 `set_messages_params()` / `setMessagesParams()` / `append_messages()` / `pushMessages()` 在工具执行*之前*覆盖待处理的请求以允许或拒绝。只有你不干预时，运行器才会自动执行你的函数。
- **Error interception** - inspect the tool result before it returns to Claude (`generate_tool_call_response()` / `generateToolResponse()`); stop early or handle it yourself.
  **错误拦截**——在工具结果返回给 Claude 之前进行检查（`generate_tool_call_response()` / `generateToolResponse()`）；可提前停止或自行处理。
- **Result modification** - mutate the tool result before it goes back (e.g. add `cache_control` for prompt caching, or transform the output).
  **结果修改**——在结果回传前进行修改（例如为提示词缓存添加 `cache_control`，或对输出做变换）。
- **Per-turn retries / param changes** - e.g. bump `max_tokens` and re-run a truncated turn; bound the whole loop with `max_iterations`.
  **按轮重试 / 参数调整**——例如调大 `max_tokens` 后重跑被截断的一轮；用 `max_iterations` 限制整个循环。
- **Streaming and automatic compaction** are both supported.
  **流式与自动压缩**均受支持。

These hooks are SDK helper features, not separate API parameters - for the exact method names and worked examples, WebFetch the per-language SDK repo listed in `shared/live-sources.md` -> *Claude API SDK Repositories* (the tool-runner helpers live in each repo's `tools.md` / `helpers.md`). The bundled `python/claude-api/tool-use.md` and `typescript/claude-api/tool-use.md` show the basic tool-runner setup.

这些钩子是 SDK 辅助功能，而非独立的 API 参数——确切的方法名与完整示例请通过 WebFetch 获取 `shared/live-sources.md` -> *Claude API SDK Repositories* 中列出的各语言 SDK 仓库（工具运行器辅助函数位于各仓库的 `tools.md` / `helpers.md`）。随附的 `python/claude-api/tool-use.md` 与 `typescript/claude-api/tool-use.md` 展示了工具运行器的基本配置。

**Don't drop to a manual loop because of these misconceptions:**

**不要因为以下误解而退回手动循环：**

- The tool runner does not require Zod/Pydantic - `betaTool()` (TS) and `@beta_tool` (Python) accept raw JSON Schema; other SDKs use plain structs/maps/classes.
  工具运行器并不要求 Zod/Pydantic——`betaTool()`（TS）与 `@beta_tool`（Python）接受原始 JSON Schema；其他 SDK 使用普通的结构体/映射/类。
- The runner makes detecting the final turn *easier*, not harder - iteration ends when Claude stops calling tools, and the last yielded message is the final response. Most SDKs also offer a one-shot variant (`runner.until_done()` / `runner.runUntilDone()` / `RunToCompletion()`).
  运行器让检测最终一轮变得*更容易*而非更难——当 Claude 停止调用工具时迭代结束，最后产出的消息就是最终响应。多数 SDK 还提供一次性变体（`runner.until_done()` / `runner.runUntilDone()` / `RunToCompletion()`）。
- Confirmation/approval gates work with the runner (see Security below).
  确认/审批门控与运行器兼容（见下文安全说明）。

**Manual Agentic Loop:** Reach for this only when you want to own the *entire* loop - you need control the runner does not expose (e.g., a custom transport, request shapes the SDK cannot build, per-token streaming on SDKs whose runner does not support it), you'd rather not take the beta dependency, or your control flow doesn't fit the runner's per-turn hooks (e.g. interleaving unrelated work mid-loop). Approval gates, logging, interception, result modification, and conditional execution do **not** require it - the tool runner covers those (above). Loop until `stop_reason == "end_turn"`, always append the full `response.content` to preserve tool_use blocks, and ensure each `tool_result` includes the matching `tool_use_id`.

**手动代理循环：** 只有当你想完全拥有*整个*循环时才考虑它——你需要运行器未暴露的控制能力（例如自定义传输层、SDK 无法构造的请求形态、在不支持流式的 SDK 运行器上做逐 token 流式输出），你不想引入 beta 依赖，或者你的控制流程无法套入运行器的按轮钩子（例如在循环中途插入无关工作）。审批门控、日志、拦截、结果修改与条件执行都**不需要**手动循环——工具运行器已覆盖这些（见上）。循环直到 `stop_reason == "end_turn"`，始终完整追加 `response.content` 以保留 tool_use 块，并确保每个 `tool_result` 都带有匹配的 `tool_use_id`。

**Stop reasons for server-side tools:** When using server-side tools (code execution, web search, etc.), the API runs a server-side sampling loop. If this loop reaches its default limit of 10 iterations, the response will have `stop_reason: "pause_turn"`. To continue, re-send the user message and assistant response and make another API request - the server will resume where it left off. Do NOT add an extra user message like "Continue." - the API detects the trailing `server_tool_use` block and knows to resume automatically.

**服务端工具的停止原因：** 使用服务端工具（代码执行、网页搜索等）时，API 会在服务端运行一个采样循环。如果该循环达到默认的 10 次迭代上限，响应将带有 `stop_reason: "pause_turn"`。要继续，重新发送用户消息和助手响应并再次发起 API 请求——服务端会从中断处继续。不要添加"Continue."之类的额外用户消息——API 会检测到尾部的 `server_tool_use` 块并自动知道要继续。

```python
# Handle pause_turn in your agentic loop
if response.stop_reason == "pause_turn":
    messages = [
        {"role": "user", "content": user_query},
        {"role": "assistant", "content": response.content},
    ]
    # Make another API request - server resumes automatically
    response = client.messages.create(
        model="claude-opus-5-5", messages=messages, tools=tools
    )
```

**Note:** the SDK tool runners do not auto-resume `pause_turn` (as of `@anthropic-ai/sdk` 0.110.0 / `anthropic` 0.116.0) - a paused turn ends the runner and is returned as the final message, with no error. In TypeScript you can resume inside the iteration body (push the paused assistant turn back onto the runner); in Python the runner cannot be resumed mid-loop - restart a new runner with the paused turn appended, or handle `pause_turn` in a manual loop. See each language's `tool-use.md` for the pattern.

**注意：** SDK 的工具运行器不会自动恢复 `pause_turn`（截至 `@anthropic-ai/sdk` 0.110.0 / `anthropic` 0.116.0）——暂停的一轮会结束运行器并作为最终消息返回，不报错误。在 TypeScript 中，你可以在迭代体内恢复（把暂停的助手轮推回运行器）；在 Python 中，运行器无法在循环中途恢复——应把暂停的轮次追加后重启一个新的运行器，或在手动循环中处理 `pause_turn`。具体模式见各语言的 `tool-use.md`。

Set a `max_continuations` limit (e.g., 5) to prevent infinite loops. For the full guide, see: `https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons`

设置 `max_continuations` 上限（例如 5）以防止无限循环。完整指南见：`https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons`

> **Security:** The tool runner executes your tool functions automatically whenever Claude requests them. For tools with side effects (sending emails, modifying databases, financial transactions), validate inputs and gate destructive operations behind human approval. **Both** the tool runner and the manual loop support this - with the tool runner, gate inside the tool's run function (prompt the user and return a "user declined" result instead of executing), or inspect the tool call in each yielded message and take over message history with `set_messages_params()` / `setMessagesParams()` to allow or deny *before* the tool runs (it executes your function automatically only if you don't intervene); with the manual loop you gate inline before calling the function.

> **安全：** 只要 Claude 发出请求，工具运行器就会自动执行你的工具函数。对于有副作用的工具（发送邮件、修改数据库、金融交易），请校验输入，并把破坏性操作置于人工审批之后。工具运行器与手动循环**都**支持这一点——使用工具运行器时，在工具的 run 函数内做门控（提示用户并返回"用户已拒绝"的结果而不执行），或检查每条产出消息中的工具调用，并用 `set_messages_params()` / `setMessagesParams()` 接管消息历史，在工具运行*之前*允许或拒绝（只有你不干预时它才会自动执行你的函数）；使用手动循环时，在调用函数前内联做门控。

---

### Handling Tool Results / 处理工具结果

When Claude uses a tool, the response contains a `tool_use` block. You must:

Claude 使用工具时，响应中会包含一个 `tool_use` 块。你必须：

1. Execute the tool with the provided input
   用提供的输入执行该工具
2. Send the result back in a `tool_result` message
   以 `tool_result` 消息把结果回传
3. Continue the conversation
   继续对话

**Error handling in tool results:** When a tool execution fails, set `"is_error": true` and provide an informative error message. Claude will typically acknowledge the error and either try a different approach or ask for clarification.

**工具结果中的错误处理：** 当工具执行失败时，设置 `"is_error": true` 并提供有信息量的错误消息。Claude 通常会确认该错误，然后尝试其他方法或请求澄清。

**Multiple tool calls:** Claude can request multiple tools in a single response. Handle them all before continuing - send all results back in a single `user` message.

**多个工具调用：** Claude 可以在单个响应中请求多个工具。请先处理完所有调用再继续——在单条 `user` 消息中回传全部结果。

---

## Server-Side Tools: Code Execution / 服务端工具：代码执行

The code execution tool lets Claude run code in a secure, sandboxed container. Unlike user-defined tools, server-side tools run on Anthropic's infrastructure - you don't execute anything client-side. Just include the tool definition and Claude handles the rest.

代码执行工具让 Claude 能在安全、沙箱化的容器中运行代码。与用户自定义工具不同，服务端工具运行在 Anthropic 的基础设施上——你不需要在客户端执行任何东西。只需包含工具定义，剩下的由 Claude 处理。

### Key Facts / 关键事实

- Runs in an isolated container (1 CPU, 5 GiB RAM, 5 GiB disk)
  运行在隔离容器中（1 个 CPU、5 GiB 内存、5 GiB 磁盘）
- No internet access (fully sandboxed)
  无互联网访问（完全沙箱化）
- Python 3.11 with data science libraries pre-installed
  预装数据科学库的 Python 3.11
- Containers persist for 30 days and can be reused across requests
  容器保留 30 天，可跨请求复用
- Free when used with web search/web fetch tools; otherwise $0.05/hour after 1,550 free hours/month per organization
  与网页搜索/网页抓取工具一起使用时免费；否则每个组织每月 1,550 免费小时之后按 $0.05/小时计费

### Tool Definition / 工具定义

The tool requires no schema - just declare it in the `tools` array:

该工具不需要 schema——只需在 `tools` 数组中声明：

```json
{
  "type": "code_execution_20260120",
  "name": "code_execution"
}
```

Claude automatically gains access to `bash_code_execution` (run shell commands) and `text_editor_code_execution` (create/view/edit files).

Claude 会自动获得 `bash_code_execution`（运行 shell 命令）和 `text_editor_code_execution`（创建/查看/编辑文件）的访问权限。

### Pre-installed Python Libraries / 预装 Python 库

- **Data science**: pandas, numpy, scipy, scikit-learn, statsmodels
  **数据科学**：pandas、numpy、scipy、scikit-learn、statsmodels
- **Visualization**: matplotlib, seaborn
  **可视化**：matplotlib、seaborn
- **File processing**: openpyxl, xlsxwriter, pillow, pypdf, pdfplumber, python-docx, python-pptx
  **文件处理**：openpyxl、xlsxwriter、pillow、pypdf、pdfplumber、python-docx、python-pptx
- **Math**: sympy, mpmath
  **数学**：sympy、mpmath
- **Utilities**: tqdm, python-dateutil, pytz, sqlite3
  **工具库**：tqdm、python-dateutil、pytz、sqlite3

Additional packages can be installed at runtime via `pip install`.

其他包可在运行时通过 `pip install` 安装。

### Supported File Types for Upload / 支持上传的文件类型

| Type   | Extensions                         |
| ------ | ---------------------------------- |
| Data   | CSV, Excel (.xlsx/.xls), JSON, XML |
| Images | JPEG, PNG, GIF, WebP               |
| Text   | .txt, .md, .py, .js, etc.          |

| 类型   | 扩展名                              |
| ------ | ----------------------------------- |
| 数据   | CSV、Excel（.xlsx/.xls）、JSON、XML |
| 图像   | JPEG、PNG、GIF、WebP                |
| 文本   | .txt、.md、.py、.js 等              |

### Container Reuse / 容器复用

Reuse containers across requests to maintain state (files, installed packages, variables). Extract the `container_id` from the first response and pass it to subsequent requests.

跨请求复用容器以保持状态（文件、已安装的包、变量）。从第一个响应中提取 `container_id`，并在后续请求中传入。

### Response Structure / 响应结构

The response contains interleaved text and tool result blocks:

响应中交错包含文本和工具结果块：

- `text` - Claude's explanation
  `text`——Claude 的解释
- `server_tool_use` - What Claude is doing
  `server_tool_use`——Claude 正在做什么
- `bash_code_execution_tool_result` - Code execution output (check `return_code` for success/failure)
  `bash_code_execution_tool_result`——代码执行输出（检查 `return_code` 判断成功/失败）
- `text_editor_code_execution_tool_result` - File operation results
  `text_editor_code_execution_tool_result`——文件操作结果

> **Security:** Always sanitize filenames with `os.path.basename()` / `path.basename()` before writing downloaded files to disk to prevent path traversal attacks. Write files to a dedicated output directory.

> **安全：** 在把下载的文件写入磁盘之前，务必用 `os.path.basename()` / `path.basename()` 清理文件名，以防路径遍历攻击。请把文件写入专用的输出目录。

---

## Server-Side Tools: Web Search and Web Fetch / 服务端工具：网页搜索与网页抓取

Web search and web fetch let Claude search the web and retrieve page content. They run server-side - just include the tool definitions and Claude handles queries, fetching, and result processing automatically.

网页搜索与网页抓取让 Claude 能搜索网络并获取页面内容。它们在服务端运行——只需包含工具定义，Claude 会自动处理查询、抓取和结果处理。

### Tool Definitions / 工具定义

```json
[
  { "type": "web_search_20260209", "name": "web_search" },
  { "type": "web_fetch_20260209", "name": "web_fetch" }
]
```

### Dynamic Filtering (Claude Opus 5.5 / Claude Opus 5 / Fable 5 / Opus 4.8 / Opus 4.7 / Opus 4.6 / Claude Sonnet 5.5 / Sonnet 5 / Sonnet 4.6) / 动态过滤（Claude Opus 5.5 / Claude Opus 5 / Fable 5 / Opus 4.8 / Opus 4.7 / Opus 4.6 / Claude Sonnet 5.5 / Sonnet 5 / Sonnet 4.6）

The `web_search_20260209` and `web_fetch_20260209` versions support **dynamic filtering** - Claude writes and executes code to filter search results before they reach the context window, improving accuracy and token efficiency. Dynamic filtering is built into these tool versions and activates automatically; you do not need to separately declare the `code_execution` tool or pass any beta header.

`web_search_20260209` 与 `web_fetch_20260209` 版本支持**动态过滤**——Claude 会编写并执行代码，在搜索结果进入上下文窗口之前对其进行过滤，从而提升准确性与 token 效率。动态过滤内建于这些工具版本并自动激活；你无需单独声明 `code_execution` 工具，也无需传递任何 beta 头。

```json
{
  "tools": [
    { "type": "web_search_20260209", "name": "web_search" },
    { "type": "web_fetch_20260209", "name": "web_fetch" }
  ]
}
```

Without dynamic filtering, the previous `web_search_20250305` version is also available.

如不需要动态过滤，也可使用先前的 `web_search_20250305` 版本。

> **Note:** Only include the standalone `code_execution` tool when your application needs code execution for its own purposes (data analysis, file processing, visualization) independent of web search. Including it alongside `_20260209` web tools creates a second execution environment that can confuse the model.

> **注意：** 只有当你的应用出于自身目的（数据分析、文件处理、可视化）需要代码执行、且独立于网页搜索时，才应包含独立的 `code_execution` 工具。把它与 `_20260209` 网页工具一同包含会创建第二个执行环境，可能使模型产生混淆。

---

## Server-Side Tools: Programmatic Tool Calling / 服务端工具：程序化工具调用

With standard tool use, each tool call is a round trip: Claude calls, the result enters Claude's context, Claude reasons, then calls the next tool. Chained calls accumulate latency and tokens - most of that intermediate data is never needed again.

在标准工具使用方式下，每次工具调用都是一次往返：Claude 发起调用，结果进入 Claude 的上下文，Claude 推理，然后再调用下一个工具。链式调用会不断累积延迟和 token——其中大部分中间数据之后再也不会被用到。

Programmatic tool calling lets Claude compose those calls into a script. The script runs in the code execution container; when it invokes a tool, the container pauses, the call executes, and the result returns to the running code (not to Claude's context). The script processes it with normal control flow. Only the final output returns to Claude. Use it when chaining many tool calls or when intermediate results are large and should be filtered before reaching the context window.

程序化工具调用让 Claude 能把这些调用组合成一段脚本。脚本在代码执行容器中运行；当脚本调用某个工具时，容器暂停，调用被执行，结果返回给正在运行的代码（而不是进入 Claude 的上下文）。脚本用普通控制流处理结果。只有最终输出会返回给 Claude。当需要链式调用许多工具、或中间结果很大且应在进入上下文窗口前被过滤时，使用该功能。

For full documentation, use WebFetch:

完整文档请用 WebFetch 获取：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling`
  URL：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling`

---

## Server-Side Tools: Tool Search / 服务端工具：工具搜索

The tool search tool lets Claude dynamically discover tools from large libraries without loading all definitions into the context window. Use it when you have many tools but only a few are relevant to any given request. Discovered tool schemas are appended to the request, not swapped in - this preserves the prompt cache (see `agent-design.md` §Caching for Agents).

工具搜索工具让 Claude 能从大型工具库中动态发现工具，而不必把所有定义都载入上下文窗口。当你有很多工具但任何给定请求只涉及其中少数几个时使用它。被发现的工具 schema 是追加到请求中，而不是替换——这能保持提示词缓存有效（见 `agent-design.md` §Caching for Agents）。

For full documentation, use WebFetch:

完整文档请用 WebFetch 获取：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool`
  URL：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool`

---

## Mid-conversation tool changes (Beta) / 对话中途的工具变更（Beta）

**Beta header `mid-conversation-tool-changes-2026-07-01`; Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, and Claude Sonnet 5.5 - not Claude Sonnet 5; not available on Microsoft Foundry (availability: `shared/platform-availability.md`).** Normally `tools` is fixed for a conversation's lifetime - editing it changes the very front of the prompt prefix and invalidates the entire cache (see `prompt-caching.md` § Invalidation hierarchy). This feature lets you add and remove tools between turns while the cached prefix survives.

**Beta 头 `mid-conversation-tool-changes-2026-07-01`；支持 Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1 与 Claude Sonnet 5.5——不支持 Claude Sonnet 5；Microsoft Foundry 上不可用（可用性见 `shared/platform-availability.md`）。** 通常 `tools` 在对话生命周期内是固定的——修改它会改变提示词前缀的最前端，使整个缓存失效（见 `prompt-caching.md` § Invalidation hierarchy）。该功能让你能在轮次之间添加和移除工具，同时缓存的前缀得以保留。

Both operations are content blocks on a `{"role": "system", ...}` message appended to `messages[]`, and both reference a tool by name via a `tool_reference`:

两种操作都是追加到 `messages[]` 的 `{"role": "system", ...}` 消息上的内容块，并且都通过 `tool_reference` 按名称引用工具：

```python
# Removal - must sit immediately before an assistant message, or last in messages.
{"role": "system", "content": [
    {"type": "tool_removal", "tool": {"type": "tool_reference", "name": "get_weather"}},
]}

# Addition - surfaces a tool declared up front with defer_loading.
{"role": "system", "content": [
    {"type": "tool_addition", "tool": {"type": "tool_reference", "name": "get_forecast"}},
]}
```

**A tool you plan to add must already be declared in `tools[]` with `"defer_loading": True`.** Deferred tools are known to the request but not loaded into the model's context until a `tool_addition` surfaces them:

**计划添加的工具必须已以 `"defer_loading": True` 声明在 `tools[]` 中。** 延迟加载的工具对请求而言是已知的，但在 `tool_addition` 使其显现之前不会载入模型的上下文：

```python
tools = [
    {"name": "get_weather", "description": "Get weather",
     "input_schema": {"type": "object", "properties": {"city": {"type": "string"}}}},
    {"name": "get_forecast", "description": "Get 5-day forecast",
     "input_schema": {"type": "object", "properties": {"city": {"type": "string"}}},
     "defer_loading": True},
]
```

**To change a tool's definition**, do it across two requests: send a `tool_removal` for the old definition on the first, then carry the conversation forward with the updated entry in `tools[]` on the next.

**要修改某个工具的定义**，需跨两个请求进行：第一个请求发送针对旧定义的 `tool_removal`，下一个请求在 `tools[]` 中携带更新后的条目并延续对话。

> Warning: Earlier previews used a different beta header and different block shapes; both are deprecated. Use `mid-conversation-tool-changes-2026-07-01` with `tool_addition` / `tool_removal` / `tool_reference`.

> 警告：更早的预览版使用了不同的 beta 头和不同的块结构；两者均已弃用。请使用 `mid-conversation-tool-changes-2026-07-01` 配合 `tool_addition` / `tool_removal` / `tool_reference`。

SDK typings lag these blocks - pass them as plain dicts in Python, or add a `@ts-expect-error` in TypeScript.

SDK 的类型定义滞后于这些块——在 Python 中以普通 dict 传入，或在 TypeScript 中加上 `@ts-expect-error`。

**Choosing between this and tool search:** tool search is for *discovery* - Claude finds what it needs from a large library on its own. Mid-conversation tool changes are for *control* - your application decides the tool set has changed (a mode switch, a resource that became available, a capability you want to revoke) and says so explicitly.

**在本功能与工具搜索之间做选择：** 工具搜索面向*发现*——由 Claude 自己从大型库中找到所需工具。对话中途的工具变更面向*控制*——由你的应用判定工具集发生了变化（模式切换、某个资源变为可用、要撤销某项能力）并明确声明。

---

## Agent Skills (Messages API) / 代理技能（Messages API）

Agent Skills package task-specific instructions and files that Claude loads when relevant (e.g., the Anthropic pre-built `pptx`, `xlsx`, `pdf`, `docx` skills). On the **Messages API**, skills are enabled via the `container` parameter alongside the code-execution tool - this is **not** the Managed Agents surface and does **not** use `client.beta.agents` / `sessions` / `environments`. Availability: see `shared/platform-availability.md`.

代理技能（Agent Skills）打包了任务相关的指令与文件，Claude 会在相关时加载它们（例如 Anthropic 预构建的 `pptx`、`xlsx`、`pdf`、`docx` 技能）。在 **Messages API** 上，技能通过 `container` 参数配合代码执行工具启用——这**不是** Managed Agents 接口，也**不**使用 `client.beta.agents` / `sessions` / `environments`。可用性见 `shared/platform-availability.md`。

Required on each request:

每个请求都需要：

1. `client.beta.messages.create(...)` with the `code-execution-2025-08-25` beta flag (Skills is out of beta - no `skills-2025-10-02` header needed).
   在 `client.beta.messages.create(...)` 中带上 `code-execution-2025-08-25` beta 标志（Skills 已脱离 beta——无需 `skills-2025-10-02` 头）。
2. `container={"skills": [{"type": "anthropic", "skill_id": "<id>", "version": "latest"}]}` - the skills list selects which skills are available inside the execution container.
   `container={"skills": [{"type": "anthropic", "skill_id": "<id>", "version": "latest"}]}`——技能列表选定执行容器内可用的技能。
3. `tools=[{"type": "code_execution_20260521", "name": "code_execution"}]` - skills execute via code execution in the container.
   `tools=[{"type": "code_execution_20260521", "name": "code_execution"}]`——技能通过容器内的代码执行来运行。

```python
response = client.beta.messages.create(
    model="claude-opus-5-5", max_tokens=16000,
    betas=["code-execution-2025-08-25"],
    container={"skills": [{"type": "anthropic", "skill_id": "pptx", "version": "latest"}]},
    tools=[{"type": "code_execution_20260521", "name": "code_execution"}],
    messages=[{"role": "user", "content": "Create a 3-slide presentation on X"}],
)
```

Generated files (`.pptx`, `.xlsx`, ...) are written inside the container; the response carries a file ID for each. Download by passing that ID to the Files API (`client.files.download(file_id)` / `GET /v1/files/{id}/content`).

生成的文件（`.pptx`、`.xlsx` 等）写在容器内；响应中为每个文件携带一个文件 ID。把该 ID 传给 Files API 即可下载（`client.files.download(file_id)` / `GET /v1/files/{id}/content`）。

List available skills via `GET /v1/skills` (no beta header).

通过 `GET /v1/skills` 列出可用技能（无需 beta 头）。

---

## MCP Connector (Beta) / MCP 连接器（Beta）

The MCP connector lets Claude call tools hosted on a remote MCP server directly from the Messages API - Anthropic makes the MCP connection server-side. Requires beta flag `mcp-client-2025-11-20` on `client.beta.messages.create(...)`. Availability: see `shared/platform-availability.md`.

MCP 连接器让 Claude 能直接从 Messages API 调用托管在远程 MCP 服务器上的工具——由 Anthropic 在服务端建立 MCP 连接。需要在 `client.beta.messages.create(...)` 上使用 beta 标志 `mcp-client-2025-11-20`。可用性见 `shared/platform-availability.md`。

**Two parameters are required together:**

**两个参数必须同时提供：**

- `mcp_servers` - array of server connection definitions: `[{"type": "url", "url": "<server URL>", "name": "<server-name>", "authorization_token": "<optional>"}]`
  `mcp_servers`——服务器连接定义的数组：`[{"type": "url", "url": "<server URL>", "name": "<server-name>", "authorization_token": "<optional>"}]`
- `tools` - must include an `mcp_toolset` entry that references the server by name: `[{"type": "mcp_toolset", "mcp_server_name": "<server-name>"}]`
  `tools`——必须包含一个按名称引用服务器的 `mcp_toolset` 条目：`[{"type": "mcp_toolset", "mcp_server_name": "<server-name>"}]`

The `mcp_server_name` in the toolset must match a `name` in `mcp_servers`. Omitting the `mcp_toolset` entry is rejected as a validation error - every server in `mcp_servers` must be referenced by exactly one toolset.

工具集中的 `mcp_server_name` 必须与 `mcp_servers` 中的某个 `name` 匹配。省略 `mcp_toolset` 条目会被作为校验错误拒绝——`mcp_servers` 中的每个服务器都必须被恰好一个工具集引用。

```python
client.beta.messages.create(
    model="claude-opus-5-5", max_tokens=1024,
    betas=["mcp-client-2025-11-20"],
    mcp_servers=[{"type": "url", "url": "https://example/sse", "name": "example-mcp"}],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "example-mcp"}],
    messages=[...],
)
```

Go uses the typed constant `anthropic.AnthropicBetaMCPClient2025_11_20`; the older `...2025_04_04` constant is deprecated.

Go 使用带类型的常量 `anthropic.AnthropicBetaMCPClient2025_11_20`；较旧的 `...2025_04_04` 常量已弃用。

Optional toolset fields: `default_config` (defaults for all tools, e.g. `{"enabled": false}` for allowlist mode) and `configs` (per-tool overrides keyed by tool name).

工具集的可选字段：`default_config`（所有工具的默认配置，例如白名单模式下用 `{"enabled": false}`）和 `configs`（以工具名为键的逐工具覆盖配置）。

---

## Tool Use Examples / 工具使用示例

You can provide sample tool calls directly in your tool definitions to demonstrate usage patterns and reduce parameter errors. This helps Claude understand how to correctly format tool inputs, especially for tools with complex schemas.

你可以在工具定义中直接提供示例工具调用，以演示使用模式并减少参数错误。这有助于 Claude 理解如何正确格式化工具输入，对具有复杂 schema 的工具尤其如此。

For full documentation, use WebFetch:

完整文档请用 WebFetch 获取：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use`
  URL：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use`

---

## Client-Side Tools: Computer Use / 客户端工具：计算机使用

Computer use lets Claude interact with a desktop environment (screenshots, mouse, keyboard). It is a client-side tool - your application provides the environment and executes the actions Claude requests; Anthropic processes the screenshots and action requests in real time but does not host the environment or retain the data.

计算机使用让 Claude 能与桌面环境交互（截图、鼠标、键盘）。它是客户端工具——由你的应用提供环境并执行 Claude 请求的操作；Anthropic 实时处理截图和操作请求，但不托管环境，也不保留数据。

**Two request shapes.** The current one is the **computer toolset** - GA on the Claude API and Google Cloud, no beta header: one `tools` entry `{"type": "computer_toolset_20260801"}` with **no `name`** and no display dimensions, plus an optional `configs` map to turn member tools off (`{"zoom": {"enabled": false}}`; all 17 members, `zoom` included, are on by default). Claude's calls are `tool_use` blocks whose `name` is the member (`screenshot`, `left_click`, `type`, `zoom`, ...) carrying `"toolset_name": "computer"`, often several per turn; return one `tool_result` per call in the next `user` message, **each echoing `"toolset_name": "computer"`** (only `screenshot` / `zoom` need an image; `OK` suffices for the rest). Coordinates are in the pixel space of the full screenshots you return, also after a `zoom`, and screenshots must already fit the model's image limits. The earlier `computer_20251124` tool (beta `computer-use-2025-11-24`, a `name: "computer"` entry with `display_width_px` / `display_height_px`, actions in `input.action`) keeps working on the models and platforms that offer it - Bedrock, Claude Platform on AWS, and Foundry offer only the earlier beta versions today - and the two forms can't share a request. **Claude Opus 5.5 accepts only the toolset**: `computer_20251124` returns a 400 there (`shared/model-migration.md` -> Migrating to Claude Opus 5.5 -> Breaking change 4 has the request and agent-loop changes; test them on Claude Opus 5, which accepts both). **Claude Sonnet 5.5 accepts only the toolset on the Claude API and Google Cloud** (`computer_20251124` returns a 400 there; Amazon Bedrock still accepts it, and no platform accepts `computer_20250124`) - see `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5 -> Breaking change 4.

**两种请求形态。** 当前形态是**计算机工具集（computer toolset）**——在 Claude API 和 Google Cloud 上已正式发布（GA），无需 beta 头：一个 `tools` 条目 `{"type": "computer_toolset_20260801"}`，**不带 `name`**、不带显示尺寸，另加一个可选的 `configs` 映射用于关闭成员工具（`{"zoom": {"enabled": false}}`；全部 17 个成员——包括 `zoom`——默认开启）。Claude 的调用是 `tool_use` 块，其 `name` 为成员工具名（`screenshot`、`left_click`、`type`、`zoom` 等）并携带 `"toolset_name": "computer"`，一轮中常有多个；在下一个 `user` 消息中为每个调用返回一个 `tool_result`，**每个都要回显 `"toolset_name": "computer"`**（只有 `screenshot` / `zoom` 需要附图像；其余返回 `OK` 即可）。坐标处于你返回的完整截图的像素空间内（`zoom` 之后也是如此），且截图本身必须已符合模型的图像限制。较早的 `computer_20251124` 工具（beta `computer-use-2025-11-24`，一个 `name: "computer"` 条目，带 `display_width_px` / `display_height_px`，操作位于 `input.action`）在提供它的模型和平台上仍可继续使用——Bedrock、AWS 上的 Claude Platform 与 Foundry 目前只提供较早的 beta 版本——而且两种形态不能出现在同一请求中。**Claude Opus 5.5 只接受工具集**：在该模型上 `computer_20251124` 返回 400（请求与代理循环的改动见 `shared/model-migration.md` -> Migrating to Claude Opus 5.5 -> Breaking change 4；可在同时接受两种形态的 Claude Opus 5 上测试）。**在 Claude API 和 Google Cloud 上，Claude Sonnet 5.5 只接受工具集**（该处 `computer_20251124` 返回 400；Amazon Bedrock 仍接受它，且没有任何平台接受 `computer_20250124`）——见 `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5 -> Breaking change 4。

For full documentation (member reference, batch actions, scaling, the `computer_20251124` migration steps), use WebFetch:

完整文档（成员参考、批量操作、缩放、`computer_20251124` 迁移步骤）请用 WebFetch 获取：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/computer-use/overview`
  URL：`https://platform.claude.com/docs/en/agents-and-tools/computer-use/overview`

---

## Context Editing / 上下文编辑

Context editing clears stale tool results and thinking blocks from the transcript as a long-running agent accumulates turns. Unlike compaction (which summarizes), context editing prunes - the cleared content is removed, not replaced. Use it when old tool outputs are no longer relevant and you want to keep the transcript lean without losing the conversation structure.

上下文编辑会在长时运行的代理不断累积轮次时，从对话记录中清除过期的工具结果和思考块。与压缩（compaction，做摘要）不同，上下文编辑是修剪——被清除的内容直接移除，不做替换。当旧的工具输出不再相关、而你想保持对话记录精简又不丢失对话结构时使用它。

**Beta.** Use `client.beta.messages.*` with beta `context-management-2025-06-27`. Configure via `context_management.edits` with a strategy type of `clear_tool_uses_20250919` (clear old tool results; optional `clear_tool_inputs: true` also clears the tool_use params) or `clear_thinking_20251015` (clear thinking blocks). These are **not** the compaction types - `compact_20260112` with beta `compact-2026-01-12` is the separate compaction feature.

**Beta。** 使用 `client.beta.messages.*` 并带上 beta `context-management-2025-06-27`。通过 `context_management.edits` 配置，策略类型为 `clear_tool_uses_20250919`（清除旧工具结果；可选的 `clear_tool_inputs: true` 还会清除 tool_use 参数）或 `clear_thinking_20251015`（清除思考块）。这些**不是**压缩类型——配合 beta `compact-2026-01-12` 的 `compact_20260112` 才是独立的压缩功能。

For full documentation, use WebFetch:

完整文档请用 WebFetch 获取：

- URL: `https://platform.claude.com/docs/en/build-with-claude/context-editing`
  URL：`https://platform.claude.com/docs/en/build-with-claude/context-editing`

---

## Server-Side Tools: Advisor (Beta) / 服务端工具：Advisor（Beta）

The advisor tool pairs a faster, lower-cost **executor** model (the top-level `model` on the request) with a higher-intelligence **advisor** model (the `model` field inside the tool definition) that provides strategic guidance mid-generation. The executor does most of the token generation; the advisor is consulted for planning. Availability: see `shared/platform-availability.md`.

Advisor 工具把一个更快、更便宜的**执行器**模型（请求顶层的 `model`）与一个智能更高的**顾问（advisor）**模型（工具定义内的 `model` 字段）配对，后者在生成过程中提供战略性指导。执行器承担大部分 token 生成；顾问用于规划咨询。可用性见 `shared/platform-availability.md`。

### Tool Definition / 工具定义

```json
{
  "type": "advisor_20260301",
  "name": "advisor",
  "model": "claude-opus-4-8"
}
```

Optional fields on the tool definition:

工具定义上的可选字段：

- `max_uses` - cap on advisor consultations per request. Exceeding it makes the `advisor_tool_result` block's `content` the error object `{"type": "advisor_tool_result_error", "error_code": "max_uses_exceeded"}` - the third member of the content union in the payload-shape table below.
  `max_uses`——每次请求顾问咨询次数的上限。超出后，`advisor_tool_result` 块的 `content` 会变为错误对象 `{"type": "advisor_tool_result_error", "error_code": "max_uses_exceeded"}`——即下方载荷形态表中 content 联合类型的第三个成员。
- `max_tokens` - bounds the advisor's total output (thinking + text) per call. At the cap the result block carries `stop_reason: "max_tokens"` and a truncation note is appended to the advice the executor sees; the server also emits a remaining-tokens budget block in the advisor's prompt so it self-shapes toward the cap.
  `max_tokens`——限制顾问每次调用的总输出（思考 + 文本）。达到上限时，结果块带有 `stop_reason: "max_tokens"`，并在执行器看到的建议后附加截断说明；服务端还会在顾问的提示词中发出一个剩余 token 预算块，使其自觉向该上限靠拢。
- `caching` - cache-control for the advisor's own prompt, same shape as a cache breakpoint: `"caching": {"type": "ephemeral", "ttl": "5m"}` (`ttl` is `"5m"` or `"1h"`, default `"5m"`). Each call writes a cache entry at that TTL so later calls in the conversation read the stable prefix. Omitted = advisor prompt not cached.
  `caching`——顾问自身提示词的缓存控制，形状与缓存断点相同：`"caching": {"type": "ephemeral", "ttl": "5m"}`（`ttl` 为 `"5m"` 或 `"1h"`，默认 `"5m"`）。每次调用都会按该 TTL 写入一个缓存条目，因此对话中后续调用可读取稳定前缀。省略 = 顾问提示词不缓存。

**The advisor model must be at least as capable as the executor.** An invalid pairing returns `400 invalid_request_error`. Valid pairs:

**顾问模型的能力必须不低于执行器。** 无效配对会返回 `400 invalid_request_error`。有效配对：

| Executor (request `model`) | Valid advisor (tool `model`) |
|---|---|
| `claude-haiku-4-5` / `claude-sonnet-4-6` | `claude-mythos-5-1`, `claude-fable-5-1`, `claude-mythos-5`, `claude-fable-5`, `claude-opus-5-5`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-opus-4-6`, `claude-sonnet-5-5`, `claude-sonnet-5`, or `claude-sonnet-4-6` |
| `claude-sonnet-5` | `claude-mythos-5-1`, `claude-fable-5-1`, `claude-mythos-5`, `claude-fable-5`, `claude-opus-5-5`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-sonnet-5-5`, or `claude-sonnet-5` |
| `claude-opus-4-6` | `claude-mythos-5-1`, `claude-fable-5-1`, `claude-mythos-5`, `claude-fable-5`, `claude-opus-5-5`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-opus-4-6`, `claude-sonnet-5-5`, or `claude-sonnet-5` |
| `claude-opus-4-7` / `claude-opus-4-8` | `claude-mythos-5-1`, `claude-fable-5-1`, `claude-mythos-5`, `claude-fable-5`, `claude-opus-5-5`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, or `claude-sonnet-5-5` |
| `claude-opus-5-5` / `claude-opus-5` / `claude-fable-5` / `claude-mythos-5` | `claude-mythos-5-1`, `claude-fable-5-1`, `claude-mythos-5`, `claude-fable-5`, `claude-opus-5-5`, or `claude-opus-5` |
| `claude-fable-5-1` / `claude-mythos-5-1` | `claude-mythos-5-1` or `claude-fable-5-1` - and these executors (like `claude-opus-5-5`) reject forced `tool_choice`, so nudge the advisor call from the prompt |
| `claude-sonnet-5-5` | `claude-mythos-5-1`, `claude-fable-5-1`, `claude-mythos-5`, `claude-fable-5`, `claude-opus-5-5`, `claude-opus-5`, or `claude-sonnet-5-5` - Claude Opus 4.8 / 4.7 / 4.6, Claude Sonnet 5, and Sonnet 4.6 advisors return a 400; every accepted advisor returns the encrypted `advisor_redacted_result`, and this executor rejects forced `tool_choice`, so nudge the advisor call from the prompt |

| 执行器（请求的 `model`） | 有效顾问（工具的 `model`） |
|---|---|
| `claude-haiku-4-5` / `claude-sonnet-4-6` | `claude-mythos-5-1`、`claude-fable-5-1`、`claude-mythos-5`、`claude-fable-5`、`claude-opus-5-5`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7`、`claude-opus-4-6`、`claude-sonnet-5-5`、`claude-sonnet-5` 或 `claude-sonnet-4-6` |
| `claude-sonnet-5` | `claude-mythos-5-1`、`claude-fable-5-1`、`claude-mythos-5`、`claude-fable-5`、`claude-opus-5-5`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7`、`claude-sonnet-5-5` 或 `claude-sonnet-5` |
| `claude-opus-4-6` | `claude-mythos-5-1`、`claude-fable-5-1`、`claude-mythos-5`、`claude-fable-5`、`claude-opus-5-5`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7`、`claude-opus-4-6`、`claude-sonnet-5-5` 或 `claude-sonnet-5` |
| `claude-opus-4-7` / `claude-opus-4-8` | `claude-mythos-5-1`、`claude-fable-5-1`、`claude-mythos-5`、`claude-fable-5`、`claude-opus-5-5`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7` 或 `claude-sonnet-5-5` |
| `claude-opus-5-5` / `claude-opus-5` / `claude-fable-5` / `claude-mythos-5` | `claude-mythos-5-1`、`claude-fable-5-1`、`claude-mythos-5`、`claude-fable-5`、`claude-opus-5-5` 或 `claude-opus-5` |
| `claude-fable-5-1` / `claude-mythos-5-1` | `claude-mythos-5-1` 或 `claude-fable-5-1`——且这些执行器（与 `claude-opus-5-5` 一样）拒绝强制 `tool_choice`，因此需从提示词中引导顾问调用 |
| `claude-sonnet-5-5` | `claude-mythos-5-1`、`claude-fable-5-1`、`claude-mythos-5`、`claude-fable-5`、`claude-opus-5-5`、`claude-opus-5` 或 `claude-sonnet-5-5`——Claude Opus 4.8 / 4.7 / 4.6、Claude Sonnet 5 与 Sonnet 4.6 作为顾问会返回 400；所有被接受的顾问都返回加密的 `advisor_redacted_result`，且该执行器拒绝强制 `tool_choice`，因此需从提示词中引导顾问调用 |

> Warning: **The advisor's payload shape differs by advisor model.** The response block is always `advisor_tool_result`; what varies is its **`content`**, a discriminated union:  
>  
> | `content` type | Fields | When |  
> |---|---|---|  
> | `advisor_result` | `text`, `stop_reason` | Advisor returns plaintext (e.g. Opus 4.8) |  
> | `advisor_redacted_result` | `encrypted_content`, `stop_reason` | Advisor returns encrypted output - Claude Opus 5.5, Claude Opus 5, Claude Fable 5.1, Claude Mythos 5.1, Claude Fable 5, Claude Mythos 5, Claude Sonnet 5.5 |  
> | `advisor_tool_result_error` | `error_code` | Consultation failed - `max_uses_exceeded`, `prompt_too_long`, `too_many_requests`, `overloaded`, `unavailable`, `execution_time_exceeded`, or `model_not_found` |  
>  
> So switch on `advisor_tool_result.content` type, not on the block type. Code that reads `.text` unconditionally gets nothing back from an Claude Opus 5.5 or Claude Opus 5 advisor, because the payload is under `encrypted_content` instead - and you cannot read it, only replay it.

> 警告：**顾问的载荷形态因顾问模型而异。** 响应块始终是 `advisor_tool_result`；变化的是它的 **`content`**，一个可辨识联合（discriminated union）：  
>  
> | `content` 类型 | 字段 | 出现时机 |  
> |---|---|---|  
> | `advisor_result` | `text`、`stop_reason` | 顾问返回明文（例如 Opus 4.8） |  
> | `advisor_redacted_result` | `encrypted_content`、`stop_reason` | 顾问返回加密输出——Claude Opus 5.5、Claude Opus 5、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Sonnet 5.5 |  
> | `advisor_tool_result_error` | `error_code` | 咨询失败——`max_uses_exceeded`、`prompt_too_long`、`too_many_requests`、`overloaded`、`unavailable`、`execution_time_exceeded` 或 `model_not_found` |  
>  
> 因此应依据 `advisor_tool_result.content` 的类型分支处理，而不是依据块类型。无条件读取 `.text` 的代码在 Claude Opus 5.5 或 Claude Opus 5 顾问那里什么都拿不到，因为载荷在 `encrypted_content` 之下——而且你无法读取它，只能原样回放。

【评论】`advisor_redacted_result` 的加密输出意味着调用方无法检查顾问的实际内容，只能把它回放给执行器模型——这是一种对客户端屏蔽模型间中间通信的设计。

Call via `client.beta.messages.create(...)` with `betas=["advisor-tool-2026-03-01"]` (or the `anthropic-beta: advisor-tool-2026-03-01` header). In multi-turn conversations, append the full `response.content` - including any `advisor_tool_result` blocks - back to `messages` on the next turn. If you remove the advisor tool from `tools` on a later turn while the history still contains `advisor_tool_result` blocks, the API returns a 400.

通过 `client.beta.messages.create(...)` 调用并带上 `betas=["advisor-tool-2026-03-01"]`（或 `anthropic-beta: advisor-tool-2026-03-01` 头）。在多轮对话中，下一轮要把完整的 `response.content`——包括任何 `advisor_tool_result` 块——追加回 `messages`。如果后续某一轮从 `tools` 中移除了 advisor 工具，而历史中仍包含 `advisor_tool_result` 块，API 会返回 400。

> **Advisor on Managed Agents:** CMA sessions support an advisor too, configured as a `{"type": "advisor", "model"}` entry in the agent's multiagent roster rather than as a tool definition - no `max_uses`/`max_tokens`/`caching` options, and advice is delivered as thread events on the session's event stream rather than `advisor_tool_result` blocks. See `shared/managed-agents-multiagent.md` -> Advisor.

> **Managed Agents 上的 Advisor：** CMA 会话同样支持顾问，配置为代理 multiagent 名册中的 `{"type": "advisor", "model"}` 条目而非工具定义——没有 `max_uses`/`max_tokens`/`caching` 选项，且建议以会话事件流上的线程事件形式传递，而不是 `advisor_tool_result` 块。见 `shared/managed-agents-multiagent.md` -> Advisor。

---

## Client-Side Tools: Memory / 客户端工具：记忆

The memory tool enables Claude to store and retrieve information across conversations through a memory file directory. Claude can create, read, update, and delete files that persist between sessions.

记忆工具让 Claude 能通过一个记忆文件目录跨对话存储和检索信息。Claude 可以创建、读取、更新和删除在会话之间持久化的文件。

### Key Facts / 关键事实

- Client-side tool - you control storage via your implementation
  客户端工具——存储由你的实现控制
- Supports commands: `view`, `create`, `str_replace`, `insert`, `delete`, `rename`
  支持的命令：`view`、`create`、`str_replace`、`insert`、`delete`、`rename`
- Operates on files in a `/memories` directory
  操作 `/memories` 目录中的文件
- The Python, TypeScript, and Java SDKs provide helper classes/functions for implementing the memory backend
  Python、TypeScript 和 Java SDK 提供用于实现记忆后端的辅助类/函数

> **Security:** Never store API keys, passwords, tokens, or other secrets in memory files. Be cautious with personally identifiable information (PII) - check data privacy regulations (GDPR, CCPA) before persisting user data. The reference implementations have no built-in access control; in multi-user systems, implement per-user memory directories and authentication in your tool handlers.

> **安全：** 绝不要在记忆文件中存储 API 密钥、密码、令牌或其他秘密。对个人身份信息（PII）要谨慎——持久化用户数据前先确认数据隐私法规（GDPR、CCPA）。参考实现没有内置访问控制；在多用户系统中，请在工具处理器中实现按用户隔离的记忆目录和身份验证。

【评论】记忆目录跨会话持久化用户相关数据，属于高敏存储位置；若提示词注入能驱动模型调用该工具，可能带来跨会话的数据污染或泄露风险，因此按用户隔离与鉴权需要由调用方自行实现。

For full implementation examples, use WebFetch:

完整实现示例请用 WebFetch 获取：

- Docs: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`
  Docs：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`

---

## Client-Side Tools: Bash and Text Editor / 客户端工具：Bash 与文本编辑器

The bash and text editor tools are **Anthropic-defined, schema-less** tools. Declare them by `type` and `name` only - the input schema is built into the model and cannot be modified. **Do not pass an `input_schema`**, and do not define a custom tool that happens to be named `"bash"` - that creates a user-defined tool without the built-in behavior.

bash 与文本编辑器工具是 **Anthropic 定义的无 schema** 工具。只需通过 `type` 和 `name` 声明——输入 schema 内置于模型中，不可修改。**不要传 `input_schema`**，也不要定义恰好名为 `"bash"` 的自定义工具——那会创建一个没有内置行为的用户自定义工具。

Both are **client-executed**: Claude returns a `tool_use` block, your code performs the action locally, and you send back a `tool_result`. The API is stateless; your application maintains the shell session or filesystem between turns.

两者都由**客户端执行**：Claude 返回 `tool_use` 块，你的代码在本地执行操作，然后你回传 `tool_result`。API 是无状态的；shell 会话或文件系统由你的应用在轮次之间维护。

### Bash tool declaration / Bash 工具声明

```json
{"type": "bash_20250124", "name": "bash"}
```

| Language | Declaration |
|---|---|
| Python / TypeScript / Ruby / cURL | plain object `{"type": "bash_20250124", "name": "bash"}` |
| Go | `anthropic.ToolUnionParam{OfBashTool20250124: &anthropic.ToolBash20250124Param{}}` |
| Java | `.addTool(ToolBash20250124.builder().build())` from `com.anthropic.models.messages` |
| C# | `Tools = [new ToolBash20250124()]` from `Anthropic.Models.Messages` |
| PHP | `tools: [new \Anthropic\Messages\ToolBash20250124()]` |

| 语言 | 声明方式 |
|---|---|
| Python / TypeScript / Ruby / cURL | 普通对象 `{"type": "bash_20250124", "name": "bash"}` |
| Go | `anthropic.ToolUnionParam{OfBashTool20250124: &anthropic.ToolBash20250124Param{}}` |
| Java | 来自 `com.anthropic.models.messages` 的 `.addTool(ToolBash20250124.builder().build())` |
| C# | 来自 `Anthropic.Models.Messages` 的 `Tools = [new ToolBash20250124()]` |
| PHP | `tools: [new \Anthropic\Messages\ToolBash20250124()]` |

Claude's `tool_use.input` contains either `{"command": "<string>"}` or `{"restart": true}`. Check for `restart` first (reset the session, return a confirmation string); otherwise run `command` and return combined stdout + stderr.

Claude 的 `tool_use.input` 包含 `{"command": "<string>"}` 或 `{"restart": true}`。先检查 `restart`（重置会话，返回确认字符串）；否则运行 `command` 并返回合并后的 stdout + stderr。

> **Security - commands are untrusted model output.** Run in an isolated environment (container, VM, or restricted user); apply an **allowlist** of permitted executables and reject shell operators (`&&`, `|`, `;`, `` ` ``, `$()`); set timeouts and resource limits; log every command. A blocklist is not sufficient.

> **安全——命令是不可信的模型输出。** 在隔离环境中运行（容器、虚拟机或受限用户）；对可执行文件使用**白名单**，拒绝 shell 操作符（`&&`、`|`、`;`、`` ` ``、`$()`）；设置超时与资源限制；记录每条命令。仅靠黑名单是不够的。

### Text editor tool declaration / 文本编辑器工具声明

```json
{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}
```

Optional field: `max_characters` to cap `view` output. Java exposes a typed `ToolTextEditor20250728` builder (`com.anthropic.models.messages`); other statically-typed SDKs follow the same naming pattern - see the Anthropic-Defined Tools section in `{lang}/claude-api/tool-use.md` for the exact class.

可选字段：`max_characters`，用于限制 `view` 的输出量。Java 提供带类型的 `ToolTextEditor20250728` 构建器（`com.anthropic.models.messages`）；其他静态类型 SDK 沿用同样的命名模式——确切的类名见 `{lang}/claude-api/tool-use.md` 的 Anthropic-Defined Tools 一节。

> **Security - `path` is untrusted model output. Confine every file operation to a fixed project root.** Before executing any command, resolve the model-supplied `path` to its canonical form and verify it remains within your project root; reject the request if it escapes (`..`, symlinks, absolute paths outside the root, URL-encoded traversal like `%2e%2e%2f`). Use your language's built-in path utilities (e.g., Python `pathlib.Path.resolve()` then check `.is_relative_to(root)`). Never call `open()` / `writeFile` / `unlink` directly on the raw `path` value.

> **安全——`path` 是不可信的模型输出。把所有文件操作限制在固定的项目根目录内。** 执行任何命令之前，先把模型提供的 `path` 解析为规范形式，并验证它仍位于项目根目录之内；如果越界（`..`、符号链接、根目录之外的绝对路径、`%2e%2e%2f` 之类的 URL 编码遍历），则拒绝请求。使用你所用语言内置的路径工具（例如 Python 中先 `pathlib.Path.resolve()` 再检查 `.is_relative_to(root)`）。绝不要直接对原始 `path` 值调用 `open()` / `writeFile` / `unlink`。

`tool_use.input.command` is one of:

`tool_use.input.command` 是以下之一：

| `command` | Other inputs | Action |
|---|---|---|
| `view` | `path`, optional `view_range` | Return file contents or directory listing |
| `create` | `path`, `file_text` | Create/overwrite file with `file_text`. Create a backup if the file already exists. |
| `str_replace` | `path`, `old_str`, `new_str` | Replace exactly one occurrence; error if 0 or >1 matches |
| `insert` | `path`, `insert_line`, `insert_text` | Insert `insert_text` after line `insert_line` (0 = beginning of file) |

| `command` | 其他输入 | 操作 |
|---|---|---|
| `view` | `path`，可选 `view_range` | 返回文件内容或目录列表 |
| `create` | `path`、`file_text` | 用 `file_text` 创建/覆盖文件。若文件已存在则先创建备份。 |
| `str_replace` | `path`、`old_str`、`new_str` | 精确替换一处匹配；匹配数为 0 或大于 1 时报错 |
| `insert` | `path`、`insert_line`、`insert_text` | 在第 `insert_line` 行之后插入 `insert_text`（0 表示文件开头） |

For both tools, on error return `{"type": "tool_result", "tool_use_id": "...", "content": "<error text>", "is_error": true}` so Claude can recover.

对这两个工具，出错时返回 `{"type": "tool_result", "tool_use_id": "...", "content": "<error text>", "is_error": true}`，让 Claude 能够恢复。

---

## Structured Outputs / 结构化输出

Structured outputs constrain Claude's responses to follow a specific JSON schema, guaranteeing valid, parseable output. This is not a separate tool - it enhances the Messages API response format and/or tool parameter validation.

结构化输出约束 Claude 的响应遵循特定的 JSON schema，保证输出有效且可解析。它不是一个独立的工具——它是对 Messages API 响应格式和/或工具参数校验的增强。

Two features are available:

有两个可用特性：

- **JSON outputs** (`output_config.format`): Control Claude's response format
  **JSON 输出**（`output_config.format`）：控制 Claude 的响应格式
- **Strict tool use** (`strict: true`): Guarantee valid tool parameter schemas
  **严格工具调用**（`strict: true`）：保证工具参数 schema 有效

**Supported models:** Claude Fable 5, Claude Mythos 5, Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5.5, Claude Opus 5, Claude Opus 4.8, Claude Sonnet 5.5, Claude Sonnet 5, and Claude Haiku 4.5. Legacy models (Claude Opus 4.5, Claude Opus 4.1) also support structured outputs.

**支持的模型：** Claude Fable 5、Claude Mythos 5、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Sonnet 5.5、Claude Sonnet 5 与 Claude Haiku 4.5。旧模型（Claude Opus 4.5、Claude Opus 4.1）也支持结构化输出。

> **Recommended:** Use `client.messages.parse()` which automatically validates responses against your schema. When using `messages.create()` directly, use `output_config: {format: {...}}`. The `output_format` convenience parameter is also accepted by some SDK methods (e.g., `.parse()`), but `output_config.format` is the canonical API-level parameter.

> **推荐：** 使用 `client.messages.parse()`，它会自动按你的 schema 校验响应。直接使用 `messages.create()` 时，请用 `output_config: {format: {...}}`。某些 SDK 方法（例如 `.parse()`）也接受 `output_format` 便捷参数，但 `output_config.format` 才是规范的 API 级参数。

### JSON Schema Limitations / JSON Schema 局限

**Supported:**

**支持：**

- Basic types: object, array, string, integer, number, boolean, null
  基本类型：object、array、string、integer、number、boolean、null
- `enum`, `const`, `anyOf`, `allOf`, `$ref`/`$def`
  `enum`、`const`、`anyOf`、`allOf`、`$ref`/`$def`
- String formats: `date-time`, `time`, `date`, `duration`, `email`, `hostname`, `uri`, `ipv4`, `ipv6`, `uuid`
  字符串格式：`date-time`、`time`、`date`、`duration`、`email`、`hostname`、`uri`、`ipv4`、`ipv6`、`uuid`
- `additionalProperties: false` (required for all objects)
  `additionalProperties: false`（所有对象都要求此项）

**Not supported:**

**不支持：**

- Recursive schemas
  递归 schema
- Numerical constraints (`minimum`, `maximum`, `multipleOf`)
  数值约束（`minimum`、`maximum`、`multipleOf`）
- String constraints (`minLength`, `maxLength`)
  字符串约束（`minLength`、`maxLength`）
- Complex array constraints
  复杂数组约束
- `additionalProperties` set to anything other than `false`
  `additionalProperties` 设为 `false` 以外的任何值

The Python and TypeScript SDKs automatically handle unsupported constraints by removing them from the schema sent to the API and validating them client-side.

Python 和 TypeScript SDK 会自动处理不受支持的约束：把它们从发送给 API 的 schema 中移除，并在客户端进行校验。

### Important Notes / 重要注意事项

- **First request latency**: New schemas incur a one-time compilation cost. Subsequent requests with the same schema use a 24-hour cache.
  **首次请求延迟**：新 schema 会产生一次性的编译开销。相同 schema 的后续请求使用 24 小时缓存。
- **Refusals**: If Claude refuses for safety reasons (`stop_reason: "refusal"`), the output may not match your schema.
  **拒答**：如果 Claude 因安全原因拒答（`stop_reason: "refusal"`），输出可能不符合你的 schema。
- **Token limits**: If `stop_reason: "max_tokens"`, output may be incomplete. Increase `max_tokens`.
  **token 限制**：如果 `stop_reason: "max_tokens"`，输出可能不完整。请调大 `max_tokens`。
- **Incompatible with**: Citations (returns 400 error), message prefilling.
  **不兼容于**：引用（Citations，返回 400 错误）、消息预填充。
- **Works with**: Batches API, streaming, token counting, extended thinking.
  **兼容于**：Batches API、流式、token 计数、扩展思考。

---

## Tips for Effective Tool Use / 高效工具使用技巧

1. **Provide detailed descriptions**: Claude relies heavily on descriptions to understand when and how to use tools
   **提供详细描述**：Claude 高度依赖描述来理解何时以及如何使用工具
2. **Use specific tool names**: `get_current_weather` is better than `weather`
   **使用具体的工具名**：`get_current_weather` 优于 `weather`
3. **Validate inputs**: Always validate tool inputs before execution
   **校验输入**：执行前始终校验工具输入
4. **Handle errors gracefully**: Return informative error messages so Claude can adapt
   **妥善处理错误**：返回有信息量的错误消息，让 Claude 能够调整
5. **Limit tool count**: Too many tools can confuse the model - keep the set focused
   **限制工具数量**：过多工具会使模型混淆——保持工具集聚焦
6. **Test tool interactions**: Verify Claude uses tools correctly in various scenarios
   **测试工具交互**：验证 Claude 在各种场景下都能正确使用工具

For detailed tool use documentation, use WebFetch:

详细的工具调用文档请用 WebFetch 获取：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview`
  URL：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview`
