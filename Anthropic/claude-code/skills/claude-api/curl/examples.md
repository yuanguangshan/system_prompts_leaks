<!-- BILINGUAL-EN-ZH -->
# Claude API - cURL / Raw HTTP / Claude API - cURL / 原始 HTTP

Use these examples when the user needs raw HTTP requests or is working in a language without an official SDK.

当用户需要原始 HTTP 请求、或所使用的语言没有官方 SDK 时，使用这些示例。

## Setup / 准备

```bash
export ANTHROPIC_API_KEY="your-api-key"
```

---

## Basic Message Request / 基本消息请求

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 16000,
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
```

### Parsing the response / 解析响应

Use `jq` to extract fields from the JSON response. Do not use `grep`/`sed` -  
JSON strings can contain any character and regex parsing will break on quotes,
escapes, or multi-line content.

使用 `jq` 从 JSON 响应中提取字段。不要使用 `grep`/`sed`——
JSON 字符串可以包含任意字符，正则解析会在引号、
转义符或多行内容上出错。

```bash
# Capture the response, then extract fields
response=$(curl -s https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model":"claude-opus-5-5","max_tokens":16000,"messages":[{"role":"user","content":"Hello"}]}')

# Print the first text block (-r strips the JSON quotes)
echo "$response" | jq -r '.content[0].text'

# Read usage fields
input_tokens=$(echo "$response" | jq -r '.usage.input_tokens')
output_tokens=$(echo "$response" | jq -r '.usage.output_tokens')

# Read stop reason (for tool-use loops)
stop_reason=$(echo "$response" | jq -r '.stop_reason')

# Extract all text blocks (content is an array; filter to type=="text")
echo "$response" | jq -r '.content[] | select(.type == "text") | .text'
```


---

## Streaming (SSE) / 流式传输（SSE）

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 64000,
    "stream": true,
    "messages": [{"role": "user", "content": "Write a haiku"}]
  }'
```

The response is a stream of Server-Sent Events:

响应是一个 Server-Sent Events 流：

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

---

## Tool Use / 工具使用

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 16000,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [{"role": "user", "content": "What is the weather in Paris?"}]
  }'
```

When Claude responds with a `tool_use` block, send the result back:

当 Claude 以 `tool_use` 块响应时，把结果回传：

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 16000,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [
      {"role": "user", "content": "What is the weather in Paris?"},
      {"role": "assistant", "content": [
        {"type": "text", "text": "Let me check the weather."},
        {"type": "tool_use", "id": "toolu_abc123", "name": "get_weather", "input": {"location": "Paris"}}
      ]},
      {"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": "toolu_abc123", "content": "72°F and sunny"}
      ]}
    ]
  }'
```

---

## Prompt Caching / 提示词缓存

Put `cache_control` on the last block of the stable prefix. See `shared/prompt-caching.md` for placement patterns and the silent-invalidator audit checklist.

把 `cache_control` 放在稳定前缀的最后一个块上。放置模式与"静默失效"审计清单见 `shared/prompt-caching.md`。

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 16000,
    "system": [
      {"type": "text", "text": "<large shared prompt...>", "cache_control": {"type": "ephemeral"}}
    ],
    "messages": [{"role": "user", "content": "Summarize the key points"}]
  }'
```

For 1-hour TTL: `"cache_control": {"type": "ephemeral", "ttl": "1h"}`. Top-level `"cache_control"` on the request body auto-places on the last cacheable block. Verify hits via the response `usage.cache_creation_input_tokens` / `usage.cache_read_input_tokens` fields.

1 小时 TTL：`"cache_control": {"type": "ephemeral", "ttl": "1h"}`。请求体顶层的 `"cache_control"` 会自动放到最后一个可缓存的块上。通过响应的 `usage.cache_creation_input_tokens` / `usage.cache_read_input_tokens` 字段核验命中情况。

---

## Extended Thinking / 扩展思考

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking. `budget_tokens` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Claude Opus 5.5:** thinking is always on - omit `thinking` (or send `{"type": "adaptive"}`, which is equivalent); `{"type": "disabled"}` returns a 400 at every effort, as does a thinking budget. Control depth with `output_config.effort` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5:** thinking is on by default - omitting `"thinking"` runs adaptive (`{"type": "adaptive"}` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `{"type": "disabled"}` is accepted only at effort `high` or lower; pairing it with `xhigh`/`max` returns a 400.  
> **Older models:** Use `"type": "enabled"` with `"budget_tokens": N` (must be < `max_tokens`, min 1024).

> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 与 Sonnet 4.6：**使用自适应思考（adaptive thinking）。`budget_tokens` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 与 4.7 上已移除（传入则返回 400）；在 Opus 4.6 与 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5：**思考始终开启——省略 `thinking`（或发送 `{"type": "adaptive"}`，二者等价）；`{"type": "disabled"}` 在任何 effort 档位都返回 400，思考预算同样如此。改用 `output_config.effort` 控制深度——该模型默认 `medium`，而 Claude Opus 5 默认 `high`。  
> **Claude Opus 5：**思考默认开启——省略 `"thinking"` 即运行为 adaptive（`{"type": "adaptive"}` 等价），不同于 Opus 4.8/4.7 上省略意味着不思考。`{"type": "disabled"}` 仅在 effort 为 `high` 或更低时被接受；与 `xhigh`/`max` 搭配返回 400。  
> **更旧的模型：**使用 `"type": "enabled"` 加 `"budget_tokens": N`（必须 < `max_tokens`，最小 1024）。

```bash
# Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / 4.7 / 4.6: adaptive thinking (recommended)
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 16000,
    "thinking": {
      "type": "adaptive",
      "display": "summarized"
    },
    "output_config": {
      "effort": "high"
    },
    "messages": [{"role": "user", "content": "Solve this step by step..."}]
  }'
```

---

## Refusal Fallbacks (Claude Fable 5.1) - opt in by default / 拒答回退（Claude Fable 5.1）- 默认需主动启用

On `claude-fable-5-1`, safety classifiers may decline a request (HTTP 200 with `stop_reason: "refusal"`). Fallbacks are **opt-in**: without them the request simply stops. Include the `fallbacks` parameter and its beta header by default - on a policy decline the API re-runs the same request on the fallback model inside the same call. A mid-stream decline is billed at normal rates, and the rescue bills at the fallback model's own rates; for a decline before any output, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed).

在 `claude-fable-5-1` 上，安全分类器可能拒绝一个请求（HTTP 200 加 `stop_reason: "refusal"`）。回退（fallback）是**主动启用（opt-in）**的：不启用时请求就直接停止。默认带上 `fallbacks` 参数及其 beta 请求头——在策略性拒绝时，API 会在同一次调用内改用回退模型重跑同一请求。流中途的拒绝按正常费率计费，救援部分按回退模型自身的费率计费；对尚未产生任何输出的拒绝，见[拒答如何计费](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。

【评论】"opt-in by default"这一标题措辞略有歧义：正文明确回退需要显式传参才启用，即默认并不开启，需要调用方自行选择加入。

```bash
response=$(curl -s https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: server-side-fallback-2026-06-01" \
  -d '{
    "model": "claude-fable-5-1",
    "max_tokens": 16000,
    "fallbacks": [{"model": "claude-opus-4-8"}],
    "messages": [{"role": "user", "content": "Hello"}]
  }')

# Which model produced the message
echo "$response" | jq -r '.model'

# Refusal on the final response means the whole chain refused
echo "$response" | jq -r '.stop_reason'

# Switch points: one fallback block per model that ran and declined this turn
echo "$response" | jq -r '.content[] | select(.type == "fallback") | "\(.from.model) declined; \(.to.model) continued"'

# Served-by signal - covers sticky turns, which carry no fallback block.
# Pair with stop_reason: the fallback model can itself refuse.
if [ "$(echo "$response" | jq -r '.stop_reason')" != "refusal" ] && \
   echo "$response" | jq -e '[.usage.iterations[]? | select(.type == "fallback_message")] | length > 0' > /dev/null; then
  echo "fallback model served this turn"
fi
```

The header must be exactly `server-side-fallback-2026-06-01` **for this array form**; the newer `fallbacks: "default"` scalar form uses `server-side-fallback-2026-07-01` instead (see `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features), and pairing either header with the other form returns a 400. The parameter is rejected on the Batches API and unavailable on Amazon Bedrock, Vertex AI, and Microsoft Foundry. Full semantics (sticky routing, billing, streaming, echoing fallback turns back): `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason.

**对本数组形式而言**，请求头必须严格是 `server-side-fallback-2026-06-01`；较新的 `fallbacks: "default"` 标量形式改用 `server-side-fallback-2026-07-01`（见 `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features），任一请求头与另一种形式搭配都返回 400。该参数在 Batches API 上被拒绝，且在 Amazon Bedrock、Vertex AI 与 Microsoft Foundry 上不可用。完整语义（粘性路由、计费、流式传输、把回退轮次回传）：`shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason。

---

## Required Headers / 必需请求头

| Header              | Value              | Description                |
| ------------------- | ------------------ | -------------------------- |
| `Content-Type`      | `application/json` | Required                   |
| `x-api-key`         | Your API key       | Authentication             |
| `anthropic-version` | `2023-06-01`       | API version                |
| `anthropic-beta`    | Beta feature IDs   | Required for beta features |

| 请求头              | 值              | 说明                |
| ------------------- | ------------------ | -------------------------- |
| `Content-Type`      | `application/json` | 必需                   |
| `x-api-key`         | 你的 API 密钥       | 身份认证             |
| `anthropic-version` | `2023-06-01`       | API 版本                |
| `anthropic-beta`    | Beta 功能 ID   | 使用 beta 功能时必需 |
