<!-- BILINGUAL-EN-ZH -->
# Claude API - Python / Claude API（Python）

## Installation / 安装

```bash
pip install anthropic
```

## Client Initialization / 客户端初始化

```python
import anthropic

# Default - resolves credentials from the environment:
# ANTHROPIC_API_KEY, or ANTHROPIC_AUTH_TOKEN, or an `ant auth login` profile.
# Prefer this for local dev; don't hardcode a key.
client = anthropic.Anthropic()

# Explicit API key (only when you must inject a specific key)
client = anthropic.Anthropic(api_key="your-api-key")

# Async client
async_client = anthropic.AsyncAnthropic()
```

---

## Client Configuration / 客户端配置

### Per-request overrides / 单次请求覆盖

Use `with_options()` to override client settings for a single call without mutating the client:

使用 `with_options()` 为单次调用覆盖客户端设置，而无需修改客户端本身：

```python
client.with_options(timeout=5.0, max_retries=5).messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
```

### Timeouts / 超时

Default request timeout is 10 minutes. Pass a float (seconds) or an `anthropic.Timeout` for granular control. On timeout the SDK raises `anthropic.APITimeoutError` (and retries per `max_retries`).

默认请求超时为 10 分钟。可传入浮点数（秒）或 `anthropic.Timeout` 进行细粒度控制。超时后 SDK 会抛出 `anthropic.APITimeoutError`（并按 `max_retries` 重试）。

```python
client = anthropic.Anthropic(timeout=20.0)
client = anthropic.Anthropic(
    timeout=anthropic.Timeout(60.0, read=5.0, write=10.0, connect=2.0),
)
```

`anthropic` 1.x is built on [`httpx2`](https://pypi.org/project/httpx2/), not `httpx`. `anthropic.Timeout` is `httpx2.Timeout`; if you import the HTTP library yourself, write `import httpx2 as httpx` - an object from the `httpx` package (`httpx.Timeout`, `httpx.Client`, transports, limits) is rejected or fails at request time. Existing `httpx`-era code is covered by the [v1 migration guide](https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md) and `/claude-api upgrade python`.

`anthropic` 1.x 构建在 [`httpx2`](https://pypi.org/project/httpx2/) 之上，而非 `httpx`。`anthropic.Timeout` 即 `httpx2.Timeout`；如果你自行导入 HTTP 库，应写 `import httpx2 as httpx`——来自 `httpx` 包的对象（`httpx.Timeout`、`httpx.Client`、传输层、连接限制）会被拒绝或在请求时失败。既有 `httpx` 时代代码可参考[《v1 迁移指南》](https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md)和 `/claude-api upgrade python`。

### Retries / 重试

The SDK auto-retries connection errors, 408, 409, 429, and >=500 with exponential backoff (default 2 retries). Set `max_retries` on the client or via `with_options()`; `max_retries=0` disables.

SDK 会自动以指数退避方式重试连接错误以及 408、409、429 和 >=500 状态码（默认重试 2 次）。可在客户端上或通过 `with_options()` 设置 `max_retries`；`max_retries=0` 表示禁用。

### Async performance (aiohttp backend) / 异步性能（aiohttp 后端）

For high-concurrency async workloads, install `anthropic[aiohttp]` and pass `DefaultAioHttpClient` instead of the default httpx2 backend:

对于高并发异步工作负载，可安装 `anthropic[aiohttp]` 并传入 `DefaultAioHttpClient` 替代默认的 httpx2 后端：

```python
from anthropic import AsyncAnthropic, DefaultAioHttpClient

async with AsyncAnthropic(http_client=DefaultAioHttpClient()) as client:
    ...
```

### Custom HTTP client (proxy, base URL) / 自定义 HTTP 客户端（代理、base URL）

Use `DefaultHttpxClient` / `DefaultAsyncHttpxClient` - not a raw `httpx2.Client` (and never a client from the `httpx` package) - so the SDK's default timeouts and connection limits are preserved:

请使用 `DefaultHttpxClient` / `DefaultAsyncHttpxClient`，而非裸的 `httpx2.Client`（且绝不要使用 `httpx` 包中的客户端），以保留 SDK 默认的超时和连接限制：

```python
from anthropic import Anthropic, DefaultHttpxClient

client = Anthropic(
    base_url="http://my.test.server.example.com:8083",  # or ANTHROPIC_BASE_URL env var
    http_client=DefaultHttpxClient(proxy="http://my.test.proxy.example.com"),
)
```

### Logging / 日志

Set `ANTHROPIC_LOG=debug` (or `info`) to enable SDK logging via the standard `logging` module.

设置 `ANTHROPIC_LOG=debug`（或 `info`）可通过标准 `logging` 模块启用 SDK 日志。

---

## Basic Message Request / 基本消息请求

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)
# response.content is a list of content block objects (TextBlock, ThinkingBlock,
# ToolUseBlock, ...). Check .type before accessing .text.
for block in response.content:
    if block.type == "text":
        print(block.text)
```

---

## System Prompts / 系统提示词

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    system="You are a helpful coding assistant. Always provide examples in Python.",
    messages=[{"role": "user", "content": "How do I read a JSON file?"}]
)
```

### Mid-conversation system messages (model-gated) / 对话中途系统消息（受模型支持限制）

For operator instructions that arrive mid-conversation (mode switches, injected state), append `{"role": "system", ...}` to `messages` instead of editing top-level `system` - this preserves the cached prefix and carries operator authority. Must follow a user message (or an `assistant` message ending in server-tool use), and must be either the last entry in `messages` or be followed by an `assistant` turn; cannot be `messages[0]`. Unsupported models return a 400 (`role 'system' is not supported on this model`). See `shared/prompt-caching.md` for when to use this vs. top-level `system`.

对于在对话中途到达的操作者指令（模式切换、注入的状态），应将 `{"role": "system", ...}` 追加到 `messages` 中，而不是修改顶层的 `system`——这样既能保留已缓存的前缀，又携带操作者权限。该消息必须跟在用户消息之后（或以服务器工具调用结尾的 `assistant` 消息之后），并且必须是 `messages` 的最后一项，或其后紧跟一个 `assistant` 回合；不能是 `messages[0]`。不支持的模型会返回 400（`role 'system' is not supported on this model`）。关于何时使用该方式与顶层 `system`，参见 `shared/prompt-caching.md`。
【评论】“携带操作者权限”值得注意：对话中途的 system 消息被赋予高于普通用户内容的指令层级，因此该设计同时规定了严格的插入位置约束，防止被当作普通消息滥用。

```python
response = client.messages.create(
    model=MODEL_ID,  # must support mid-conversation system messages
    max_tokens=16000,
    system=[{"type": "text", "text": STABLE_SYSTEM, "cache_control": {"type": "ephemeral"}}],
    messages=history + [
        {"role": "user", "content": user_message},
        {"role": "system", "content": "Terse mode enabled - keep responses under 40 words."},
    ],
)  # No beta header needed - use regular client.messages.create
```

---

## Vision (Images) / 视觉（图像）

### Base64 / Base64

```python
import base64

with open("image.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data
                }
            },
            {"type": "text", "text": "What's in this image?"}
        ]
    }]
)
```

### URL / URL

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/image.png"
                }
            },
            {"type": "text", "text": "Describe this image"}
        ]
    }]
)
```

---

## Prompt Caching / 提示词缓存

Cache large context to reduce costs (up to 90% savings). **Caching is a prefix match** - any byte change anywhere in the prefix invalidates everything after it. For placement patterns, architectural guidance (frozen system prompt, deterministic tool order, where to put volatile content), and the silent-invalidator audit checklist, read `shared/prompt-caching.md`.

缓存大段上下文以降低成本（最高节省 90%）。**缓存是前缀匹配**——前缀中任何位置的字节变化都会使其后所有内容失效。关于放置模式、架构指导（冻结的系统提示词、确定性的工具顺序、易变内容的位置）以及静默失效审计清单，请阅读 `shared/prompt-caching.md`。

### Automatic Caching (Recommended) / 自动缓存（推荐）

Use top-level `cache_control` to automatically cache the last cacheable block in the request - no need to annotate individual content blocks:

使用顶层的 `cache_control` 自动缓存请求中最后一个可缓存的块——无需逐个标注内容块：

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    cache_control={"type": "ephemeral"},  # auto-caches the last cacheable block
    system="You are an expert on this large document...",
    messages=[{"role": "user", "content": "Summarize the key points"}]
)
```

### Manual Cache Control / 手动缓存控制

For fine-grained control, add `cache_control` to specific content blocks:

如需细粒度控制，可向特定内容块添加 `cache_control`：

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    system=[{
        "type": "text",
        "text": "You are an expert on this large document...",
        "cache_control": {"type": "ephemeral"}  # default TTL is 5 minutes
    }],
    messages=[{"role": "user", "content": "Summarize the key points"}]
)

# With explicit TTL (time-to-live)
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    system=[{
        "type": "text",
        "text": "You are an expert on this large document...",
        "cache_control": {"type": "ephemeral", "ttl": "1h"}  # 1 hour TTL
    }],
    messages=[{"role": "user", "content": "Summarize the key points"}]
)
```

### Verifying Cache Hits / 验证缓存命中

```python
print(response.usage.cache_creation_input_tokens)  # tokens written to cache (~1.25x cost)
print(response.usage.cache_read_input_tokens)      # tokens served from cache (~0.1x cost)
print(response.usage.input_tokens)                 # uncached tokens (full cost)
```

If `cache_read_input_tokens` is zero across repeated identical-prefix requests, a silent invalidator is at work - `datetime.now()` or a UUID in the system prompt, unsorted `json.dumps()`, or a varying tool set. See `shared/prompt-caching.md` for the full audit table.

如果在多次相同前缀的请求中 `cache_read_input_tokens` 始终为零，说明存在静默失效因素——系统提示词中的 `datetime.now()` 或 UUID、未排序的 `json.dumps()`，或不断变化的工具集。完整审计表参见 `shared/prompt-caching.md`。

---

## Extended Thinking / 扩展思考

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking. `budget_tokens` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6：**请使用自适应思考。`budget_tokens` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 和 4.7 上已移除（发送则返回 400）；在 Opus 4.6 和 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5:** thinking is always on - omit `thinking` (or send `{"type": "adaptive"}`, which is equivalent); `{"type": "disabled"}` returns a 400 at every effort, as does a thinking budget. Control depth with `output_config.effort` instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5.5：**思考始终开启——省略 `thinking`（或发送等效的 `{"type": "adaptive"}`）；在任何 effort 档位下 `{"type": "disabled"}` 都会返回 400，指定思考预算亦然。请改用 `output_config.effort` 控制深度——该模型默认为 `medium`，而 Claude Opus 5 默认为 `high`。  
> **Claude Opus 5:** thinking is on by default - omitting `thinking` runs adaptive (`{"type": "adaptive"}` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `{"type": "disabled"}` is accepted only at effort `high` or lower; pairing it with `xhigh`/`max` returns a 400.  
> **Claude Opus 5：**思考默认开启——省略 `thinking` 即运行自适应模式（`{"type": "adaptive"}` 等效），这与 Opus 4.8/4.7 不同，在后者上省略意味着不思考。`{"type": "disabled"}` 仅在 effort 为 `high` 或更低时被接受；与 `xhigh`/`max` 搭配会返回 400。  
> **Older models:** Use `thinking: {type: "enabled", budget_tokens: N}` (must be < `max_tokens`, min 1024).
> **更早的模型：**使用 `thinking: {type: "enabled", budget_tokens: N}`（必须小于 `max_tokens`，最小 1024）。

```python
# Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / 4.7 / 4.6: adaptive thinking (recommended)
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    thinking={"type": "adaptive", "display": "summarized"},  # display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
    output_config={"effort": "high"},  # low | medium | high | xhigh | max
    messages=[{"role": "user", "content": "Solve this step by step..."}]
)

# Access thinking and response
for block in response.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
```

---

## Error Handling / 错误处理

```python
import anthropic

try:
    response = client.messages.create(...)
except anthropic.BadRequestError as e:
    print(f"Bad request: {e.message}")
except anthropic.AuthenticationError:
    print("Invalid API key")
except anthropic.PermissionDeniedError:
    print("API key lacks required permissions")
except anthropic.NotFoundError:
    print("Invalid model or endpoint")
except anthropic.RateLimitError as e:
    retry_after = int(e.response.headers.get("retry-after", "60"))
    print(f"Rate limited. Retry after {retry_after}s.")
except anthropic.APIStatusError as e:
    if e.status_code >= 500:
        print(f"Server error ({e.status_code}). Retry later.")
    else:
        print(f"API error: {e.message}")
except anthropic.APIConnectionError:
    print("Network error. Check internet connection.")
```

---

## Response Helpers / 响应辅助方法

Every response object exposes `_request_id` (populated from the `request-id` header) - log it when reporting failures to Anthropic. Despite the underscore prefix, this property is public.

每个响应对象都暴露 `_request_id`（取自 `request-id` 响应头）——向 Anthropic 报告故障时请记录该值。尽管以下划线为前缀，该属性是公开的。

```python
message = client.messages.create(...)
print(message._request_id)       # req_018EeWyXxfu5pfWkrYcMdjWG
print(message.to_json())          # serialize the Pydantic model
print(message.to_dict())          # plain dict
```

To access raw headers or other response metadata, use `.with_raw_response`:

如需访问原始响应头或其他响应元数据，请使用 `.with_raw_response`：

```python
raw = client.messages.with_raw_response.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
print(raw.headers.get("request-id"))
message = raw.parse()  # the Message object messages.create() would have returned
```

---

## Multi-Turn Conversations / 多轮对话

The API is stateless - send the full conversation history each time.

该 API 是无状态的——每次都要发送完整的对话历史。

```python
class ConversationManager:
    """Manage multi-turn conversations with the Claude API."""

    def __init__(self, client: anthropic.Anthropic, model: str, system: str = None):
        self.client = client
        self.model = model
        self.system = system
        self.messages = []

    def send(self, user_message: str, **kwargs) -> str:
        """Send a message and get a response."""
        self.messages.append({"role": "user", "content": user_message})

        response = self.client.messages.create(
            model=self.model,
            max_tokens=kwargs.get("max_tokens", 16000),
            system=self.system,
            messages=self.messages,
            **kwargs
        )

        assistant_message = next(
            (b.text for b in response.content if b.type == "text"), ""
        )
        self.messages.append({"role": "assistant", "content": assistant_message})

        return assistant_message

# Usage
conversation = ConversationManager(
    client=anthropic.Anthropic(),
    model="claude-opus-5-5",
    system="You are a helpful assistant."
)

response1 = conversation.send("My name is Alice.")
response2 = conversation.send("What's my name?")  # Claude remembers "Alice"
```

**Rules:** / **规则：**

- Consecutive same-role messages are allowed - the API combines them into a single turn
  允许连续相同角色的消息——API 会将其合并为单个回合
- First message must be `user`
  首条消息必须是 `user`
- `role: "system"` messages are allowed mid-conversation on supporting models (no beta header needed) - see § Mid-conversation system messages above
  在支持的模型上，`role: "system"` 消息可以出现在对话中途（无需 beta 请求头）——参见上文“对话中途系统消息”一节

---

### Compaction (long conversations) / 压缩（长对话）

> **Beta, Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6.** When conversations approach the 200K context window, compaction automatically summarizes earlier context server-side. The API returns a `compaction` block; you must pass it back on subsequent requests - append `response.content`, not just the text.
> **Beta，适用于 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6。** 当对话接近 200K 上下文窗口时，压缩（compaction）会在服务端自动摘要较早的上下文。API 会返回一个 `compaction` 块；你必须在后续请求中将其传回——应追加完整的 `response.content`，而不只是文本。

```python
import anthropic

client = anthropic.Anthropic()
messages = []

def chat(user_message: str) -> str:
    messages.append({"role": "user", "content": user_message})

    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="claude-opus-5-5",
        max_tokens=16000,
        messages=messages,
        context_management={
            "edits": [{"type": "compact_20260112"}]
        }
    )

    # Append full content - compaction blocks must be preserved
    messages.append({"role": "assistant", "content": response.content})

    return next(block.text for block in response.content if block.type == "text")

# Compaction triggers automatically when context grows large
print(chat("Help me build a Python web scraper"))
print(chat("Add support for JavaScript-rendered pages"))
print(chat("Now add rate limiting and error handling"))
```

---

## Stop Reasons / 停止原因

The `stop_reason` field in the response indicates why the model stopped generating:

响应中的 `stop_reason` 字段指示模型停止生成的原因：

| Value | Meaning |
|-------|---------|
| `end_turn` | Claude finished its response naturally |
| `max_tokens` | Hit the `max_tokens` limit - increase it or use streaming |
| `stop_sequence` | Hit a custom stop sequence |
| `tool_use` | Claude wants to call a tool - execute it and continue |
| `pause_turn` | Model paused and can be resumed (agentic flows) |
| `refusal` | Claude refused for safety reasons - check `stop_details` |

| 值 | 含义 |
|-------|---------|
| `end_turn` | Claude 自然完成回复 |
| `max_tokens` | 达到 `max_tokens` 上限——增大该值或改用流式输出 |
| `stop_sequence` | 命中自定义停止序列 |
| `tool_use` | Claude 希望调用工具——执行该工具后继续 |
| `pause_turn` | 模型已暂停且可恢复（智能体流程） |
| `refusal` | Claude 因安全原因拒答——请查看 `stop_details` |

### Structured Stop Details / 结构化停止详情

When `stop_reason` is `"refusal"`, the response includes a `stop_details` object with structured information about the refusal:

当 `stop_reason` 为 `"refusal"` 时，响应会包含一个 `stop_details` 对象，其中带有关于拒答的结构化信息：

```python
if response.stop_reason == "refusal" and response.stop_details:
    print(f"Category: {response.stop_details.category}")   # e.g. "cyber", "bio", "reasoning_extraction", "frontier_llm", or None - see docs for the full set
    print(f"Explanation: {response.stop_details.explanation}")
```

### Refusal Fallbacks (Claude Fable 5.1) - opt in by default / 拒答回退（Claude Fable 5.1）——默认需显式启用

Fallbacks are **opt-in**: without them a refused request simply stops. Include the server-side `fallbacks` parameter in `claude-fable-5-1` code by default - on a policy decline the API re-runs the same request on the fallback model inside the same call. A mid-stream decline is billed at normal rates, and the rescue bills at the fallback model's own rates, with cache repricing applied automatically; for a decline before any output, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed).

回退机制是**可选择加入（opt-in）**的：不配置时，被拒的请求会直接停止。在 `claude-fable-5-1` 的代码中默认应包含服务端 `fallbacks` 参数——发生策略性拒绝时，API 会在同一次调用内改用回退模型重新执行相同请求。流中途的拒绝按正常费率计费，救援请求按回退模型自身的费率计费，并自动应用缓存重新计价；关于在任何输出之前的拒绝如何计费，参见[拒答如何计费](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。

```python
response = client.beta.messages.create(
    model="claude-fable-5-1",
    max_tokens=16000,
    betas=["server-side-fallback-2026-06-01"],
    fallbacks=[{"model": "claude-opus-4-8"}],
    messages=[{"role": "user", "content": "..."}],
)

# Switch points: one fallback block per model that ran and declined this turn
for block in response.content:
    if block.type == "fallback":
        print(f"{block.from_.model} declined; {block.to.model} continued")

# Served-by signal - covers sticky turns, which carry no fallback block.
# Pair with stop_reason: the fallback model can itself refuse.
fallback_ran = any(
    entry.type == "fallback_message" for entry in response.usage.iterations or []
)
if fallback_ran and response.stop_reason != "refusal":
    print(f"Served by {response.model}")
```

A `stop_reason: "refusal"` on the final response means the whole chain refused. The header must be exactly `server-side-fallback-2026-06-01` **for this array form**; the newer `fallbacks: "default"` scalar form uses `server-side-fallback-2026-07-01` instead (see `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features), and pairing either header with the other form returns a 400. The parameter is rejected on the Batches API and unavailable on Amazon Bedrock, Vertex AI, and Microsoft Foundry - register the client-side `BetaRefusalFallbackMiddleware` on the client there instead. Full semantics (sticky routing, billing, streaming, echoing fallback turns back): `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason.

最终响应上出现 `stop_reason: "refusal"` 意味着整条回退链都拒绝了。**对于这种数组形式**，请求头必须严格为 `server-side-fallback-2026-06-01`；较新的 `fallbacks: "default"` 标量形式则使用 `server-side-fallback-2026-07-01`（见 `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features），任一请求头与另一种形式搭配都会返回 400。Batches API 会拒绝该参数，且该参数在 Amazon Bedrock、Vertex AI 和 Microsoft Foundry 上不可用——在这些平台上应改为在客户端注册 `BetaRefusalFallbackMiddleware`。完整语义（粘性路由、计费、流式传输、回传回退回合）：`shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason。
【评论】标题中的“默认需显式启用”与正文“代码中默认包含该参数”并不矛盾：API 层面需要显式传入参数才生效，而文档建议代码模板默认携带该参数，两者描述的是不同层面。

---

## Cost Optimization Strategies / 成本优化策略

### 1. Use Prompt Caching for Repeated Context / 1. 对重复上下文使用提示词缓存

```python
# Automatic caching (simplest - caches the last cacheable block)
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    cache_control={"type": "ephemeral"},
    system=large_document_text,  # e.g., 50KB of context
    messages=[{"role": "user", "content": "Summarize the key points"}]
)

# First request: full cost
# Subsequent requests: ~90% cheaper for cached portion
```

### 2. Choose the Right Model / 2. 选择合适的模型

```python
# Default to Opus for most tasks
response = client.messages.create(
    model="claude-opus-5-5",  # $4.00/$20.00 per 1M tokens
    max_tokens=16000,
    messages=[{"role": "user", "content": "Explain quantum computing"}]
)

# Use Sonnet for high-volume production workloads
standard_response = client.messages.create(
    model="claude-sonnet-5-5",  # $2.00/$10.00 per 1M tokens
    max_tokens=16000,
    messages=[{"role": "user", "content": "Summarize this document"}]
)

# Use Haiku only for simple, speed-critical tasks
simple_response = client.messages.create(
    model="claude-haiku-4-5",  # $1.00/$5.00 per 1M tokens
    max_tokens=256,
    messages=[{"role": "user", "content": "Classify this as positive or negative"}]
)
```

### 3. Use Token Counting Before Requests / 3. 请求前进行 token 计数

```python
count_response = client.messages.count_tokens(
    model="claude-opus-5-5",
    messages=messages,
    system=system
)

estimated_input_cost = count_response.input_tokens * 0.000004  # $4/1M tokens
print(f"Estimated input cost: ${estimated_input_cost:.4f}")
```

---

## Retry with Exponential Backoff / 使用指数退避重试

> **Note:** The Anthropic SDK automatically retries rate limit (429) and server errors (5xx) with exponential backoff. You can configure this with `max_retries` (default: 2). Only implement custom retry logic if you need behavior beyond what the SDK provides.
> **注意：** Anthropic SDK 会自动对限流（429）和服务器错误（5xx）进行指数退避重试。可通过 `max_retries` 配置（默认：2）。只有当需要 SDK 未提供的行为时，才需自行实现重试逻辑。

```python
import time
import random
import anthropic

def call_with_retry(
    client: anthropic.Anthropic,
    max_retries: int = 5,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    **kwargs
):
    """Call the API with exponential backoff retry."""
    last_exception = None

    for attempt in range(max_retries):
        try:
            return client.messages.create(**kwargs)
        except anthropic.RateLimitError as e:
            last_exception = e
        except anthropic.APIStatusError as e:
            if e.status_code >= 500:
                last_exception = e
            else:
                raise  # Client errors (4xx except 429) should not be retried

        delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
        print(f"Retry {attempt + 1}/{max_retries} after {delay:.1f}s")
        time.sleep(delay)

    raise last_exception
```
