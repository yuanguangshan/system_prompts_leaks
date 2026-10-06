<!-- BILINGUAL-EN-ZH -->
# Claude API - TypeScript / Claude API（TypeScript）

| Feature | Namespace | Key types / call |
|---|---|---|
| User profiles | beta | `client.beta.userProfiles.create(...)` / `.retrieve(id)` / `.list()`. Pass the returned profile id on `client.beta.messages.create`. Requires a beta header - check the SDK's beta-headers reference for the current flag. |

| 功能 | 命名空间 | 关键类型 / 调用 |
|---|---|---|
| 用户画像（User profiles） | beta | `client.beta.userProfiles.create(...)` / `.retrieve(id)` / `.list()`。将返回的画像 id 传给 `client.beta.messages.create`。需要 beta 头——请查阅 SDK 的 beta-headers 参考以获取当前标志。 |

## Installation / 安装

```bash
npm install @anthropic-ai/sdk
```

> **Reading local files (ESM):** `__dirname` and `__filename` are **undefined** in ES modules - using either throws `ReferenceError: __dirname is not defined` at runtime. For cwd-relative reads, pass the bare relative path (`fs.readFileSync("./sample.png")`). For script-relative paths, derive the directory from `import.meta.url`: `const here = path.dirname(fileURLToPath(import.meta.url))`. Never write `path.join(__dirname, ...)` in an ESM `.ts` file.

> **读取本地文件（ESM）：** `__dirname` 和 `__filename` 在 ES 模块中**未定义**——使用其中任何一个都会在运行时抛出 `ReferenceError: __dirname is not defined`。相对于当前工作目录的读取，直接传裸相对路径（`fs.readFileSync("./sample.png")`）；相对于脚本所在目录的路径，从 `import.meta.url` 推导目录：`const here = path.dirname(fileURLToPath(import.meta.url))`。绝不要在 ESM 的 `.ts` 文件中写 `path.join(__dirname, ...)`。

## Client Initialization / 客户端初始化

```typescript
import Anthropic from "@anthropic-ai/sdk";

// Default - resolves credentials from the environment:
// ANTHROPIC_API_KEY, or ANTHROPIC_AUTH_TOKEN, or an `ant auth login` profile.
// Prefer this for local dev; don't hardcode a key.
const client = new Anthropic();

// Explicit API key (only when you must inject a specific key)
const client = new Anthropic({ apiKey: "your-api-key" });
```

---

## Basic Message Request / 基础消息请求

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [{ role: "user", content: "What is the capital of France?" }],
});
// response.content is ContentBlock[] - a discriminated union. Narrow by .type
// before accessing .text (TypeScript will error on content[0].text without this).
for (const block of response.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

---

## System Prompts / 系统提示词

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  system:
    "You are a helpful coding assistant. Always provide examples in Python.",
  messages: [{ role: "user", content: "How do I read a JSON file?" }],
});
```

### Mid-conversation system messages (model-gated) / 对话中途的系统消息（因模型而异）

For operator instructions that arrive mid-conversation (mode switches, injected state), append `{role: "system", ...}` to `messages` instead of editing top-level `system` - this preserves the cached prefix and carries operator authority. Must follow a user message (or an `assistant` message ending in server-tool use), and must be either the last entry in `messages` or be followed by an `assistant` turn; cannot be `messages[0]`. Unsupported models return a 400 (`role 'system' is not supported on this model`). See `shared/prompt-caching.md` for when to use this vs. top-level `system`.

对于在对话中途到达的操作方指令（模式切换、注入的状态），应将 `{role: "system", ...}` 追加到 `messages`，而不是编辑顶层的 `system`——这样可保留已缓存的前缀，并携带操作方权限。它必须紧跟在 user 消息之后（或以服务器工具使用结尾的 `assistant` 消息之后），并且必须是 `messages` 的最后一项，或后面跟着一个 `assistant` 回合；不能是 `messages[0]`。不支持的模型返回 400（`role 'system' is not supported on this model`）。关于何时使用这种方式与顶层 `system` 的取舍，参见 `shared/prompt-caching.md`。

【评论】"对话中途系统消息"是不常见的设计：它把系统级指令嵌入 messages 数组，文中给出的理由是保留缓存前缀并携带操作方权限；对位置和模型支持的严格限制也表明这仍属受控特性。

```typescript
// No beta header needed - use regular client.messages.create.
const response = await client.messages.create({
  model: MODEL_ID, // must support mid-conversation system messages
  max_tokens: 16000,
  system: [
    { type: "text", text: STABLE_SYSTEM, cache_control: { type: "ephemeral" } },
  ],
  messages: [
    ...history,
    { role: "user", content: userMessage },
    { role: "system", content: "Terse mode enabled - keep responses under 40 words." },
  ],
});
```

---

## Vision (Images) / 视觉（图像）

### URL / URL

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "url", url: "https://example.com/image.png" },
        },
        { type: "text", text: "Describe this image" },
      ],
    },
  ],
});
```

### Base64 / Base64

```typescript
import fs from "fs";

const imageData = fs.readFileSync("image.png").toString("base64");

const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "base64", media_type: "image/png", data: imageData },
        },
        { type: "text", text: "What's in this image?" },
      ],
    },
  ],
});
```

---

## Prompt Caching / 提示词缓存

**Caching is a prefix match** - any byte change anywhere in the prefix invalidates everything after it. For placement patterns, architectural guidance (frozen system prompt, deterministic tool order, where to put volatile content), and the silent-invalidator audit checklist, read `shared/prompt-caching.md`.

**缓存是前缀匹配**——前缀中任何位置的一个字节变化都会使其后的所有缓存失效。关于放置模式、架构指导（冻结的系统提示词、确定性的工具顺序、易变内容的放置位置）以及静默失效审计清单，请阅读 `shared/prompt-caching.md`。

### Automatic Caching (Recommended) / 自动缓存（推荐）

Use top-level `cache_control` to automatically cache the last cacheable block in the request:

使用顶层 `cache_control` 自动缓存请求中最后一个可缓存的内容块：

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  cache_control: { type: "ephemeral" }, // auto-caches the last cacheable block
  system: "You are an expert on this large document...",
  messages: [{ role: "user", content: "Summarize the key points" }],
});
```

### Manual Cache Control / 手动缓存控制

For fine-grained control, add `cache_control` to specific content blocks:

如需细粒度控制，请为特定内容块添加 `cache_control`：

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral" }, // default TTL is 5 minutes
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// With explicit TTL (time-to-live)
const response2 = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral", ttl: "1h" }, // 1 hour TTL
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});
```

### Verifying Cache Hits / 验证缓存命中

```typescript
console.log(response.usage.cache_creation_input_tokens); // tokens written to cache (~1.25x cost)
console.log(response.usage.cache_read_input_tokens);     // tokens served from cache (~0.1x cost)
console.log(response.usage.input_tokens);                // uncached tokens (full cost)
```

If `cache_read_input_tokens` is zero across repeated identical-prefix requests, a silent invalidator is at work - `Date.now()` or a UUID in the system prompt, non-deterministic key ordering, or a varying tool set. See `shared/prompt-caching.md` for the full audit table.

如果在多次前缀完全相同的请求中 `cache_read_input_tokens` 都为零，说明存在静默失效因素——系统提示词中的 `Date.now()` 或 UUID、非确定性的键排序，或变化的工具集合。完整的审计表见 `shared/prompt-caching.md`。

---

## Extended Thinking / 扩展思考

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking. `budget_tokens` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Claude Opus 5.5:** thinking is always on - omit `thinking` (or send `{ type: "adaptive" }`, which is equivalent); `{ type: "disabled" }` returns a 400 at every effort, as does a thinking budget. Control depth with `output_config.effort` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5:** thinking is on by default - omitting `thinking` runs adaptive (`{ type: "adaptive" }` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `{ type: "disabled" }` is accepted only at effort `high` or lower; pairing it with `xhigh`/`max` returns a 400.  
> **Older models:** Use `thinking: {type: "enabled", budget_tokens: N}` (must be < `max_tokens`, min 1024).

> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6：** 使用自适应思考（adaptive thinking）。`budget_tokens` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 和 4.7 上已移除（发送则返回 400）；在 Opus 4.6 和 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5：** 思考始终开启——省略 `thinking`（或发送等效的 `{ type: "adaptive" }`）；在任何 effort 档位下发送 `{ type: "disabled" }` 都会返回 400，发送思考预算同样如此。请改用 `output_config.effort` 控制深度——该模型默认 `medium`，而 Claude Opus 5 默认 `high`。  
> **Claude Opus 5：** 思考默认开启——省略 `thinking` 时运行为自适应（等效于 `{ type: "adaptive" }`），这与 Opus 4.8/4.7 不同（在那些模型上省略意味着不思考）。`{ type: "disabled" }` 仅在 effort 为 `high` 或更低时被接受；与 `xhigh`/`max` 搭配会返回 400。  
> **更早的模型：** 使用 `thinking: {type: "enabled", budget_tokens: N}`（必须小于 `max_tokens`，最小 1024）。

```typescript
// Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / 4.7 / 4.6: adaptive thinking (recommended)
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  thinking: { type: "adaptive", display: "summarized" }, // display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
  output_config: { effort: "high" }, // low | medium | high | xhigh | max
  messages: [
    { role: "user", content: "Solve this math problem step by step..." },
  ],
});

for (const block of response.content) {
  if (block.type === "thinking") {
    console.log("Thinking:", block.thinking);
  } else if (block.type === "text") {
    console.log("Response:", block.text);
  }
}
```

---

## Error Handling / 错误处理

Use the SDK's typed exception classes - never check error messages with string matching:

使用 SDK 的类型化异常类——绝不要用字符串匹配来检查错误消息：

```typescript
import Anthropic from "@anthropic-ai/sdk";

try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.BadRequestError) {
    console.error("Bad request:", error.message);
  } else if (error instanceof Anthropic.AuthenticationError) {
    console.error("Invalid API key");
  } else if (error instanceof Anthropic.RateLimitError) {
    console.error("Rate limited - retry later");
  } else if (error instanceof Anthropic.APIError) {
    console.error(`API error ${error.status}:`, error.message);
  }
}
```

All classes extend `Anthropic.APIError` with a typed `status` field. Check from most specific to least specific. See [shared/error-codes.md](../../shared/error-codes.md) for the full error code reference.

所有类都继承自 `Anthropic.APIError`，并带有类型化的 `status` 字段。检查时应从最具体的类到最不具体的类依次进行。完整的错误代码参考见 [shared/error-codes.md](../../shared/error-codes.md)。

---

## Multi-Turn Conversations / 多轮对话

The API is stateless - send the full conversation history each time. Use `Anthropic.MessageParam[]` to type the messages array:

API 是无状态的——每次都要发送完整的对话历史。使用 `Anthropic.MessageParam[]` 为消息数组指定类型：

```typescript
const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "My name is Alice." },
  { role: "assistant", content: "Hello Alice! Nice to meet you." },
  { role: "user", content: "What's my name?" },
];

const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: messages,
});
```

**Rules:**

**规则：**

- Consecutive same-role messages are allowed - the API combines them into a single turn
  允许连续相同角色的消息——API 会将其合并为单个回合
- First message must be `user`
  第一条消息必须是 `user`
- Use SDK types (`Anthropic.MessageParam`, `Anthropic.Message`, `Anthropic.Tool`, etc.) for all API data structures - don't redefine equivalent interfaces
  所有 API 数据结构都使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Message`、`Anthropic.Tool` 等）——不要重新定义等效的接口

---

### Compaction (long conversations) / 压缩（长对话）

> **Beta, Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6.** When conversations approach the 200K context window, compaction automatically summarizes earlier context server-side. The API returns a `compaction` block; you must pass it back on subsequent requests - append `response.content`, not just the text.

> **Beta 功能，适用于 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6。** 当对话接近 200K 上下文窗口时，压缩（compaction）会在服务器端自动摘要较早的上下文。API 会返回一个 `compaction` 块；你必须在后续请求中把它传回去——追加完整的 `response.content`，而不是只追加文本。

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const messages: Anthropic.Beta.BetaMessageParam[] = [];

async function chat(userMessage: string): Promise<string> {
  messages.push({ role: "user", content: userMessage });

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 16000,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }],
    },
  });

  // Append full content - compaction blocks must be preserved
  messages.push({ role: "assistant", content: response.content });

  const textBlock = response.content.find(
    (b): b is Anthropic.Beta.BetaTextBlock => b.type === "text",
  );
  return textBlock?.text ?? "";
}

// Compaction triggers automatically when context grows large
console.log(await chat("Help me build a Python web scraper"));
console.log(await chat("Add support for JavaScript-rendered pages"));
console.log(await chat("Now add rate limiting and error handling"));
```

---

## Stop Reasons / 停止原因

The `stop_reason` field in the response indicates why the model stopped generating:

响应中的 `stop_reason` 字段指示模型停止生成的原因：

| Value           | Meaning                                                         |
| --------------- | --------------------------------------------------------------- |
| `end_turn`      | Claude finished its response naturally                          |
| `max_tokens`    | Hit the `max_tokens` limit - increase it or use streaming       |
| `stop_sequence` | Hit a custom stop sequence                                      |
| `tool_use`      | Claude wants to call a tool - execute it and continue           |
| `pause_turn`    | Model paused and can be resumed (agentic flows)                 |
| `refusal`       | Claude refused for safety reasons - check `stop_details`        |

| 值 | 含义 |
| --------------- | --------------------------------------------------------------- |
| `end_turn` | Claude 自然结束其回复 |
| `max_tokens` | 达到 `max_tokens` 上限——增大该值或改用流式输出 |
| `stop_sequence` | 命中自定义停止序列 |
| `tool_use` | Claude 想要调用工具——执行该工具并继续 |
| `pause_turn` | 模型已暂停且可恢复（智能体流程） |
| `refusal` | Claude 因安全原因拒答——检查 `stop_details` |

### Structured Stop Details / 结构化停止详情

When `stop_reason` is `"refusal"`, the response includes a `stop_details` object with structured information about the refusal:

当 `stop_reason` 为 `"refusal"` 时，响应会包含一个 `stop_details` 对象，其中带有关于拒答的结构化信息：

```typescript
if (response.stop_reason === "refusal" && response.stop_details) {
  console.log(`Category: ${response.stop_details.category}`); // e.g. "cyber", "bio", "reasoning_extraction", "frontier_llm", or null - see docs for the full set
  console.log(`Explanation: ${response.stop_details.explanation}`);
}
```

### Refusal Fallbacks (Claude Fable 5.1) - opt in by default / 拒答回退（Claude Fable 5.1）——默认启用

Fallbacks are **opt-in**: without them a refused request simply stops. Include the server-side `fallbacks` parameter in `claude-fable-5-1` code by default - on a policy decline the API re-runs the same request on the fallback model inside the same call. A mid-stream decline is billed at normal rates, and the rescue bills at the fallback model's own rates, with cache repricing applied automatically; for a decline before any output, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed).

回退是**可选启用**的：不启用时，被拒绝的请求会直接停止。在 `claude-fable-5-1` 的代码中默认加入服务端 `fallbacks` 参数——当出现策略性拒绝时，API 会在同一次调用内改用回退模型重新运行同一请求。流式传输中途的拒绝按正常费率计费，救助运行按回退模型自身的费率计费，并自动应用缓存重新计价；对于尚未产生任何输出前的拒绝，参见[拒答如何计费](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。

【评论】服务端回退把"策略性拒绝"自动转到另一个模型重跑同一请求，并同时改变计费方式，属于较激进的可用性与合规折中设计。

```typescript
const response = await client.beta.messages.create({
  model: "claude-fable-5-1",
  max_tokens: 16000,
  betas: ["server-side-fallback-2026-06-01"],
  fallbacks: [{ model: "claude-opus-4-8" }],
  messages: [{ role: "user", content: "..." }],
});

// Switch points: one fallback block per model that ran and declined this turn
for (const block of response.content) {
  if (block.type === "fallback") {
    console.log(`${block.from.model} declined; ${block.to.model} continued`);
  }
}

// Served-by signal - covers sticky turns, which carry no fallback block.
// Pair with stop_reason: the fallback model can itself refuse.
const fallbackRan = (response.usage.iterations ?? []).some(
  (entry) => entry.type === "fallback_message",
);
if (fallbackRan && response.stop_reason !== "refusal") {
  console.log(`Served by ${response.model}`);
}
```

A `stop_reason: "refusal"` on the final response means the whole chain refused. The header must be exactly `server-side-fallback-2026-06-01` **for this array form**; the newer `fallbacks: "default"` scalar form uses `server-side-fallback-2026-07-01` instead (see `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features), and pairing either header with the other form returns a 400. The parameter is rejected on the Batches API and unavailable on Amazon Bedrock, Vertex AI, and Microsoft Foundry - register the client-side `betaRefusalFallbackMiddleware` on the client there instead. Full semantics (sticky routing, billing, streaming, echoing fallback turns back): `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason.

最终响应上出现 `stop_reason: "refusal"` 意味着整条回退链都拒绝了。**对于这种数组形式**，请求头必须严格为 `server-side-fallback-2026-06-01`；较新的 `fallbacks: "default"` 标量形式改用 `server-side-fallback-2026-07-01`（参见 `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features），把任一请求头与另一种形式搭配都会返回 400。Batches API 会拒绝该参数，且该参数在 Amazon Bedrock、Vertex AI 和 Microsoft Foundry 上不可用——在这些平台上应改为在客户端注册 `betaRefusalFallbackMiddleware`。完整语义（粘性路由、计费、流式传输、回传回退回合）：`shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason。

---

## Cost Optimization Strategies / 成本优化策略

### 1. Use Prompt Caching for Repeated Context / 1. 对重复上下文使用提示词缓存

```typescript
// Automatic caching (simplest - caches the last cacheable block)
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  cache_control: { type: "ephemeral" },
  system: largeDocumentText, // e.g., 50KB of context
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// First request: full cost
// Subsequent requests: ~90% cheaper for cached portion
```

### 2. Use Token Counting Before Requests / 2. 在请求前使用令牌计数

```typescript
const countResponse = await client.messages.countTokens({
  model: "claude-opus-5-5",
  messages: messages,
  system: system,
});

const estimatedInputCost = countResponse.input_tokens * 0.000004; // $4/1M tokens
console.log(`Estimated input cost: $${estimatedInputCost.toFixed(4)}`);
```
