<!-- BILINGUAL-EN-ZH -->
# Streaming - Python / 流式传输 - Python

## Quick Start / 快速开始

```python
with client.messages.stream(
    model="claude-opus-5-5",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Async / 异步

```python
async with async_client.messages.stream(
    model="claude-opus-5-5",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Low-level: `stream=True` / 底层方式：`stream=True`

`messages.stream()` (above) is the recommended helper - it accumulates state and exposes `text_stream` / `get_final_message()`. If you only need the raw event iterator and want lower memory use, pass `stream=True` to `messages.create()` instead:

`messages.stream()`（上文）是推荐的辅助方法 —— 它会累积状态并提供 `text_stream` / `get_final_message()`。如果你只需要原始事件迭代器并希望降低内存占用，可以改为向 `messages.create()` 传入 `stream=True`：

```python
for event in client.messages.create(
    model="claude-opus-5-5",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True,
):
    print(event.type)
```

No final-message accumulation is done for you in this form.

这种形式下不会替你累积最终消息。

---

## Handling Different Content Types / 处理不同内容类型

Claude may return text, thinking blocks, or tool use. Handle each appropriately:

Claude 可能返回文本、思考块或工具调用。请分别妥善处理：

> **Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / Opus 4.7 / Opus 4.6:** Use `thinking: {type: "adaptive"}`. On Claude Opus 5.5 and Claude Opus 5 adaptive is also what you get by omitting `thinking` entirely (Claude Opus 5.5 accepts no other setting - `disabled` and `budget_tokens` both 400). On older models, use `thinking: {type: "enabled", budget_tokens: N}` instead.

> **Fable 5 / Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 / Opus 4.7 / Opus 4.6：** 使用 `thinking: {type: "adaptive"}`。在 Claude Opus 5.5 和 Claude Opus 5 上，完全省略 `thinking` 参数得到的同样是 adaptive（Claude Opus 5.5 不接受任何其他设置 —— `disabled` 和 `budget_tokens` 都返回 400）。在较老的模型上，请改用 `thinking: {type: "enabled", budget_tokens: N}`。

```python
with client.messages.stream(
    model="claude-opus-5-5",
    max_tokens=64000,
    thinking={"type": "adaptive", "display": "summarized"},  # display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
    messages=[{"role": "user", "content": "Analyze this problem"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\n[Thinking...]")
            elif event.content_block.type == "text":
                print("\n[Response:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
```

---

## Streaming with Tool Use / 流式传输与工具调用

The Python tool runner supports streaming: pass `stream=True` to `client.beta.messages.tool_runner(...)` and each iteration yields a stream you consume event-by-event, with `get_final_message()` for the accumulated message per turn (see `shared/tool-use-concepts.md` -> Tool Runner vs Manual Loop). Declare tool-runner tools with `@beta_tool(eager_input_streaming=True)` so their inputs stream as they are generated (default rule: `shared/tool-use-concepts.md` -> Eager input streaming). The runner never calls your function on unparseable input; the `ValueError` surfaces while you iterate the per-turn stream, so wrap the `for ... in runner` loop and, on failure, restart a new runner from a history you mirror while iterating, in this order for each yielded stream: take `message = stream.get_final_message()`, append it (the assistant turn), then check its `stop_reason` - on `max_tokens` with a `tool_use` present or on `refusal`, stop right there and never call `generate_tool_call_response()` for that turn (it executes the tools) - and only for a turn that continues append `runner.generate_tool_call_response()` (the matching `tool_result` user turn), exactly as `tool-use.md` does to resume `pause_turn`. The Python runner exposes no `params` read, a consumed runner cannot be iterated again, and a history missing the tool-result half of a continued turn is rejected by the API. `pause_turn` you resume yourself; a truncated text answer is simply the final message.

Python 工具运行器（tool runner）支持流式传输：向 `client.beta.messages.tool_runner(...)` 传入 `stream=True`，每次迭代会产出一个流，你逐事件消费该流，并用 `get_final_message()` 获取每轮累积的消息（见 `shared/tool-use-concepts.md` -> Tool Runner vs Manual Loop）。用 `@beta_tool(eager_input_streaming=True)` 声明工具运行器工具，使其输入在生成时即流式传输（默认规则见 `shared/tool-use-concepts.md` -> Eager input streaming）。运行器绝不会在输入无法解析时调用你的函数；`ValueError` 会在你迭代每轮流时浮出，因此请包住 `for ... in runner` 循环，一旦失败，从你在迭代过程中自行镜像维护的历史重启一个新的运行器。对每个产出的流按此顺序处理：先取 `message = stream.get_final_message()`，把它（即 assistant 轮）追加进历史，然后检查其 `stop_reason` —— 若为 `max_tokens` 且存在 `tool_use`，或为 `refusal`，就在此停止，绝不要为该轮调用 `generate_tool_call_response()`（它会执行工具）；只有继续的轮次才追加 `runner.generate_tool_call_response()`（即对应的 `tool_result` 用户轮），与 `tool-use.md` 恢复 `pause_turn` 的做法完全一致。Python 运行器不提供 `params` 读取；已消费完毕的运行器不能再次迭代；历史中缺少继续轮次的工具结果那一半会被 API 拒绝。`pause_turn` 由你自己恢复；被截断的文本答案就是最终消息。

Use the manual-loop pattern below only when you're not using the tool runner and need per-token streaming with tools. Set `eager_input_streaming: True` on each user-defined tool. With eager streaming the server no longer validates the input: the Python SDK's tolerant parser returns a partial object for a truncated input (check `stop_reason == "max_tokens"`) and can return a silently truncated one for malformed JSON (validate the parsed input before running the tool); only JSON it cannot parse at all raises `ValueError` **from the stream iterator**, so that guard wraps the stream, not the final-message read. Schema validation is not path validation: the model-supplied `path` is untrusted output, so confine it to a project root before writing (`shared/tool-use-concepts.md` -> the text-editor security note):

仅当你没有使用工具运行器、且需要工具场景下逐 token 的流式传输时，才使用下面的手动循环模式。在每个用户自定义工具上设置 `eager_input_streaming: True`。启用急切流式传输后，服务器不再校验输入：Python SDK 的宽容解析器对被截断的输入会返回部分对象（检查 `stop_reason == "max_tokens"`），对格式错误的 JSON 也可能返回被静默截断的对象（运行工具前先校验解析出的输入）；只有完全无法解析的 JSON 才会**从流迭代器**抛出 `ValueError`，因此该防护应包住流本身，而不是最终消息的读取。Schema 校验不等于路径校验：模型提供的 `path` 是不可信输出，写入前必须把它限制在项目根目录之内（`shared/tool-use-concepts.md` -> the text-editor security note）：

【评论】此段把"输入流式传输"与安全权衡写在一起：急切流式放弃了服务端输入校验，转由调用方自行校验并限制 `path`，属于把安全责任从服务端移交到客户端代码的典型设计。

```python
import json
from pathlib import Path

ROOT = Path.cwd().resolve()

tools = [
    {
        "name": "write_file",
        "description": "Write text to a file at the given path",
        "eager_input_streaming": True,  # stream large inputs as generated
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"},
                "contents": {"type": "string"},
            },
            "required": ["path", "contents"],
        },
    }
]

messages = [{"role": "user", "content": task}]
json_retries = 0

while True:
    try:
        with client.messages.stream(
            model="claude-opus-5-5",
            max_tokens=64000,
            tools=tools,
            messages=messages,
        ) as stream:
            for event in stream:
                if event.type == "text":
                    print(event.text, end="", flush=True)
                elif event.type == "input_json":
                    # Tool input fragment - arrives immediately with eager streaming
                    print(event.partial_json, end="", flush=True)
            response = stream.get_final_message()
        json_retries = 0  # the cap is on consecutive failures of one turn
    except ValueError:
        # JSON the SDK could not parse at all. It raised before the tool_use
        # block completed, so there is no tool_use_id to answer; re-issue the
        # turn (bounded). API errors are not ValueError and propagate.
        json_retries += 1
        if json_retries > 2:
            raise
        continue

    # Server-side tool hit its iteration limit: append the turn and re-send
    if response.stop_reason == "pause_turn":
        messages.append({"role": "assistant", "content": response.content})
        continue

    tool_uses = [b for b in response.content if b.type == "tool_use"]
    if response.stop_reason == "refusal" or not tool_uses:
        # end_turn, a text-only answer, or a refusal (which can cut a
        # tool_use off mid-input): nothing to run
        break
    if response.stop_reason == "max_tokens":
        # A truncated tool input parses as a valid partial object; don't run it.
        raise RuntimeError("tool input truncated; retry with a higher max_tokens")

    # The SDK's tolerant parser can return a silently truncated or mistyped
    # input (for example at an unescaped inner quote), so validate first.
    tool_results = []
    for block in tool_uses:
        args = block.input
        if not (isinstance(args, dict) and isinstance(args.get("path"), str)
                and isinstance(args.get("contents"), str)):
            tool_results.append({"type": "tool_result", "tool_use_id": block.id, "is_error": True,
                                 "content": json.dumps({"INVALID_JSON": json.dumps(args)})})
            continue
        # `path` is untrusted model output: resolve it and reject anything that
        # escapes the project root (`..`, absolute paths, symlinks) before the
        # write - schema validation alone does not check this.
        target = (ROOT / args["path"]).resolve()
        if not target.is_relative_to(ROOT):
            tool_results.append({"type": "tool_result", "tool_use_id": block.id, "is_error": True,
                                 "content": "path escapes the project root"})
            continue
        tool_results.append({"type": "tool_result", "tool_use_id": block.id,
                             "content": run_tool(block.name, {**args, "path": str(target)})})
    messages.append({"role": "assistant", "content": response.content})
    messages.append({"role": "user", "content": tool_results})
```

---

## Getting the Final Message / 获取最终消息

```python
with client.messages.stream(
    model="claude-opus-5-5",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    # Get full message after streaming
    final_message = stream.get_final_message()
    print(f"\n\nTokens used: {final_message.usage.output_tokens}")
```

---

## Streaming with Progress Updates / 带进度更新的流式传输

```python
def stream_with_progress(client, **kwargs):
    """Stream a response with progress updates."""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\n\n[Tokens used: {total_tokens}]")
    return "".join(content_parts)
```

---

## Error Handling in Streams / 流中的错误处理

```python
try:
    with client.messages.stream(
        model="claude-opus-5-5",
        max_tokens=64000,
        messages=[{"role": "user", "content": "Write a story"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\nConnection lost. Please retry.")
except anthropic.RateLimitError:
    print("\nRate limited. Please wait and retry.")
except anthropic.APIStatusError as e:
    print(f"\nAPI error: {e.status_code}")
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

| 事件类型 | 描述 | 触发时机 |
| --------------------- | --------------------------- | --------------------------------- |
| `message_start` | 包含消息元数据 | 开头时一次 |
| `content_block_start` | 新内容块开始 | 文本/tool_use 块开始时 |
| `content_block_delta` | 增量内容更新 | 每个 token/数据块 |
| `content_block_stop` | 内容块完成 | 块结束时 |
| `message_delta` | 消息级更新 | 包含 `stop_reason`、用量 |
| `message_stop` | 消息完成 | 结尾时一次 |

## Best Practices / 最佳实践

1. **Always flush output** - Use `flush=True` to show tokens immediately

1. **始终刷新输出** —— 使用 `flush=True` 立即显示 token

2. **Handle partial responses** - If the stream is interrupted, you may have incomplete content

2. **处理部分响应** —— 如果流被中断，你可能拿到不完整的内容

3. **Track token usage** - The `message_delta` event contains usage information

3. **跟踪 token 用量** —— `message_delta` 事件包含用量信息

4. **Use timeouts** - Set appropriate timeouts for your application

4. **使用超时** —— 为你的应用设置合适的超时

5. **Default to streaming** - Use `.get_final_message()` to get the complete response even when streaming, giving you timeout protection without needing to handle individual events

5. **默认使用流式** —— 即使在流式模式下也用 `.get_final_message()` 获取完整响应，从而获得超时保护，而无需逐个处理事件

6. **Large `max_tokens` without streaming raises `ValueError`** - The SDK refuses non-streaming requests it estimates will exceed ~10 minutes (idle connections drop). Pass `stream=True` / use `messages.stream()`, or explicitly override `timeout`, to suppress the guard.

6. **大的 `max_tokens` 在非流式下会抛出 `ValueError`** —— SDK 会拒绝其预计将超过约 10 分钟的非流式请求（空闲连接会被断开）。传入 `stream=True` / 使用 `messages.stream()`，或显式覆盖 `timeout`，可关闭该防护。
