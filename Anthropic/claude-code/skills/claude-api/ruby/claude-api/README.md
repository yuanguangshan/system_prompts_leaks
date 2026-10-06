<!-- BILINGUAL-EN-ZH -->
# Claude API - Ruby / Claude API - Ruby 版

> **Note:** The Ruby SDK supports the Claude API. A tool runner is available in beta via `client.beta.messages.tool_runner()`. Agent SDK is not yet available for Ruby.

> **注意：** Ruby SDK 支持 Claude API。工具运行器（tool runner）已以 Beta 形式提供，可通过 `client.beta.messages.tool_runner()` 使用。Agent SDK 尚未支持 Ruby。

## Installation / 安装

```bash
gem install anthropic
```

## Client Initialization / 客户端初始化

```ruby
require "anthropic"

# Default (uses ANTHROPIC_API_KEY env var)
client = Anthropic::Client.new

# Explicit API key
client = Anthropic::Client.new(api_key: "your-api-key")
```

---

## Basic Message Request / 基础消息请求

```ruby
message = client.messages.create(
  model: :"claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)
# content is an array of polymorphic block objects (TextBlock, ThinkingBlock,
# ToolUseBlock, ...). .type is a Symbol - compare with :text, not "text".
# .text raises NoMethodError on non-TextBlock entries.
message.content.each do |block|
  puts block.text if block.type == :text
end
```

---

## Extended Thinking / 扩展思考

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking. `budget_tokens` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 与 Sonnet 4.6：** 使用自适应思考（adaptive thinking）。`budget_tokens` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 与 4.7 上已被移除（传入则返回 400）；在 Opus 4.6 与 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5:** thinking is always on - omit `thinking` (or send `{ type: "adaptive" }`, which is equivalent); `{ type: "disabled" }` returns a 400 at every effort, as does a thinking budget. Control depth with `output_config.effort` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5.5：** 思考始终开启 —— 省略 `thinking`（或发送 `{ type: "adaptive" }`，二者等价）；`{ type: "disabled" }` 在任何 effort 档位下都会返回 400，思考预算同样如此。应改用 `output_config.effort` 控制思考深度 —— 该模型默认为 `medium`，而 Claude Opus 5 默认为 `high`。  
> **Claude Opus 5:** thinking is on by default - omitting `thinking:` runs adaptive (`{ type: "adaptive" }` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `{ type: "disabled" }` is accepted only at effort `high` or lower; pairing it with `xhigh`/`max` returns a 400.  
> **Claude Opus 5：** 思考默认开启 —— 省略 `thinking:` 即运行为自适应模式（与 `{ type: "adaptive" }` 等价），这与 Opus 4.8/4.7 不同，后两者省略该参数意味着不进行思考。`{ type: "disabled" }` 仅在 effort 为 `high` 或更低时被接受；与 `xhigh`/`max` 搭配会返回 400。  
> **Older models:** Use `thinking: { type: "enabled", budget_tokens: N }` (must be < `max_tokens`, min 1024).
> **更早的模型：** 使用 `thinking: { type: "enabled", budget_tokens: N }`（必须小于 `max_tokens`，最小值为 1024）。

```ruby
message = client.messages.create(
  model: :"claude-opus-5-5",
  max_tokens: 16000,
  thinking: { type: "adaptive" },
  messages: [{ role: "user", content: "Solve: 27 * 453" }]
)

message.content.each do |block|
  case block.type
  when :thinking then puts "Thinking: #{block.thinking}"
  when :text then puts "Response: #{block.text}"
  end
end
```

---

## Prompt Caching / 提示词缓存

`system_:` (trailing underscore - avoids shadowing `Kernel#system`) takes an array of text blocks; set `cache_control` on the last block. Plain hashes work via the `OrHash` type alias. For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

`system_:`（尾部下划线用于避免遮蔽 `Kernel#system`）接受一个文本块数组；在最后一个块上设置 `cache_control`。通过 `OrHash` 类型别名，普通哈希亦可使用。关于放置模式与静默失效审计清单，参见 `shared/prompt-caching.md`。

```ruby
message = client.messages.create(
  model: :"claude-opus-5-5",
  max_tokens: 16000,
  system_: [
    { type: "text", text: long_system_prompt, cache_control: { type: "ephemeral" } }
  ],
  messages: [{ role: "user", content: "Summarize the key points" }]
)
```

For 1-hour TTL: `cache_control: { type: "ephemeral", ttl: "1h" }`. There's also a top-level `cache_control:` on `messages.create` that auto-places on the last cacheable block.

若需 1 小时 TTL：`cache_control: { type: "ephemeral", ttl: "1h" }`。`messages.create` 上还有一个顶层 `cache_control:`，会自动放置在最后一个可缓存的块上。

Verify hits via `message.usage.cache_creation_input_tokens` / `message.usage.cache_read_input_tokens`.

可通过 `message.usage.cache_creation_input_tokens` / `message.usage.cache_read_input_tokens` 验证缓存命中情况。

---

## Stop Details / 停止详情

When `stop_reason` is `:refusal`, the response includes structured `stop_details`:

当 `stop_reason` 为 `:refusal`（拒答）时，响应中会包含结构化的 `stop_details`：

```ruby
if message.stop_reason == :refusal && message.stop_details
  puts "Category: #{message.stop_details.category}"     # e.g. :cyber, :bio, :reasoning_extraction, :frontier_llm, or nil - see docs for the full set
  puts "Explanation: #{message.stop_details.explanation}"
end
```

**Refusal fallbacks (Claude Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. New `claude-fable-5-1` code should include the server-side `fallbacks` parameter (beta header `server-side-fallback-2026-06-01`, `fallbacks: [{model: "claude-opus-4-8"}]` on the beta messages call) by default. The exact Ruby binding (and the client-side middleware for providers without server-side support) is not documented here - WebFetch the Ruby SDK repo's `examples/` from `shared/live-sources.md`; full semantics in `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason.

**拒答回退（Claude Fable 5.1）—— 默认应选择启用。** 回退机制需要显式启用：不启用时，被拒答的请求会直接停止。新的 `claude-fable-5-1` 代码默认应包含服务端 `fallbacks` 参数（Beta 头为 `server-side-fallback-2026-06-01`，在 Beta 消息调用上设置 `fallbacks: [{model: "claude-opus-4-8"}]`）。确切的 Ruby 绑定（以及为不支持服务端能力的服务商准备的客户端中间件）未在此处说明 —— 请依据 `shared/live-sources.md` 通过 WebFetch 获取 Ruby SDK 仓库的 `examples/`；完整语义见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason。

【评论】默认建议开启服务端回退意味着拒答后可自动切换到另一模型继续处理，这一设计改变了"拒答即终止"的默认行为，集成时需留意。

---

## Beta Features / Beta 功能

`betas:` is only valid on `client.beta.messages.create`, not the non-beta path.

`betas:` 仅在 `client.beta.messages.create` 上有效，非 Beta 路径不支持。

### Task budgets / 任务预算

```ruby
response = client.beta.messages.create(
  model: :"claude-opus-5-5",
  max_tokens: 16000,
  output_config: { task_budget: { type: :tokens, total: 64_000 } },
  tools: [...],
  messages: [...],
  betas: ["task-budgets-2026-03-13"]
)
```

---

## Error Type / 错误类型

`APIStatusError` exposes a `.type` field for programmatic error classification:

`APIStatusError` 暴露了一个 `.type` 字段，用于以编程方式进行错误分类：

```ruby
begin
  client.messages.create(...)
rescue Anthropic::Errors::APIStatusError => e
  puts e.type  # :rate_limit_error, :overloaded_error, etc.
end
```
