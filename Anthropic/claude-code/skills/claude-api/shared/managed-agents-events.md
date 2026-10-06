<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Events & Steering / Managed Agents - 事件与转向控制

## Events / 事件

### Sending Events / 发送事件

Send events to a session via `POST /v1/sessions/{id}/events`.

通过 `POST /v1/sessions/{id}/events` 向会话发送事件。

| Event Type                | When to Send                                        |
| ------------------------- | --------------------------------------------------- |
| `user.message`            | Send a user message |
| `user.interrupt`          | Interrupt the agent while it's running |
| `user.tool_confirmation`  | Approve/deny a tool call that paused for approval (`always_ask`, or `auto` when the server reached no determination) |
| `user.custom_tool_result` | Provide result for a custom tool call |
| `user.define_outcome`     | Start a rubric-graded iterate loop - see `shared/managed-agents-outcomes.md` |
| `system.message`          | Append privileged system-level context for this turn and every turn after it; see § Adding system context mid-session |

| 事件类型 | 发送时机 |
| --- | --- |
| `user.message` | 发送一条用户消息 |
| `user.interrupt` | 在智能体运行时中断它 |
| `user.tool_confirmation` | 批准/拒绝因等待审批而暂停的工具调用（`always_ask`，或服务器未作出判定时的 `auto`） |
| `user.custom_tool_result` | 为自定义工具调用提供结果 |
| `user.define_outcome` | 启动基于评分标准的迭代循环 - 见 `shared/managed-agents-outcomes.md` |
| `system.message` | 为本轮及之后每一轮追加特权系统级上下文；见 § Adding system context mid-session |

#### Adding system context mid-session (`system.message`) / 会话中途添加系统上下文（`system.message`）

The `system` field on the agent definition sets the top-level system prompt and is fixed for the session's lifetime. A `system.message` event **appends** to the session's system context as a `role: "system"` turn - it does not replace that prompt. The content applies to the accompanying turn and all subsequent turns. Use it for a different persona, revised constraints, or runtime-fetched context that should shape behavior going forward:

智能体定义中的 `system` 字段设定顶层系统提示词，并在会话的整个生命周期内固定不变。`system.message` 事件会以 `role: "system"` 轮次的形式**追加**到会话的系统上下文中——它不会替换该提示词。其内容作用于随附的一轮以及之后的所有轮次。可用于切换人设、修订约束，或注入应在后续行为中生效的运行时获取的上下文：

【评论】系统上下文采用"追加而非替换"的设计，使运行中会话的原始系统提示词无法被后续事件整体覆盖，从机制上收窄了运行时指令被滥用的影响面。

```python
client.beta.sessions.events.send(
    session.id,
    events=[
        {
            "type": "system.message",
            "content": [
                {"type": "text", "text": "The user's current timezone is America/New_York."},
            ],
        },
    ],
)
```

Constraints:

约束：

- **Model-gated: Claude Fable 5.1, Claude Mythos 5.1, Claude Fable 5, Claude Mythos 5, Claude Opus 5.5, Claude Opus 5, Claude Opus 4.8, and Claude Sonnet 5.5 (not Claude Sonnet 5).** Only the agent's **primary** model is checked - `system.message` lands on the primary thread only, so subagent models are not considered. On an unsupported primary model the event is rejected with a `model_does_not_support_mid_conversation_system` validation error.
  **受模型限制：Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8 和 Claude Sonnet 5.5（不支持 Claude Sonnet 5）。** 只检查智能体的**主**模型——`system.message` 仅落在主线程上，因此不考虑子智能体模型。在不支持的主模型上，该事件会被拒绝并返回 `model_does_not_support_mid_conversation_system` 校验错误。
- **While the session is idle with `stop_reason: requires_action`** (blocked on `user.custom_tool_result` / `user.tool_confirmation`), a `system.message` is accepted **only when it trails a tool result event in the same request**. Sent on its own - or alongside a `user.message` - it is rejected until the pending tool events are resolved.
  **当会话处于空闲且 `stop_reason: requires_action` 时**（正阻塞在 `user.custom_tool_result` / `user.tool_confirmation` 上），`system.message` **只有在同一请求中跟随工具结果事件之后**才会被接受。单独发送——或与 `user.message` 一起发送——都会被拒绝，直到待处理的工具事件被解决。
- `content` accepts 1-1000 text items.
  `content` 接受 1-1000 个文本项。

### Receiving Events / 接收事件

Three methods:

共三种方法：

1. **Streaming (SSE)**: `GET /v1/sessions/{id}/events/stream` - real-time Server-Sent Events. **Long-lived** - the server sends periodic heartbeats to keep the connection alive.
   1. **流式（SSE）**：`GET /v1/sessions/{id}/events/stream` - 实时 Server-Sent Events。**长连接** - 服务器会定期发送心跳以保持连接存活。
2. **Polling**: `GET /v1/sessions/{id}/events` - paginated event list (query params: `limit` default 1000, `page`). **Returns immediately** - this is a plain paginated GET, not a long-poll.
   2. **轮询**：`GET /v1/sessions/{id}/events` - 分页事件列表（查询参数：`limit` 默认 1000、`page`）。**立即返回** - 这是一个普通的分页 GET，不是长轮询。
3. **Webhooks**: Anthropic POSTs session state transitions to your HTTPS endpoint - thin payloads (IDs only), HMAC-signed, Console-registered. See `shared/managed-agents-webhooks.md`.
   3. **Webhook**：Anthropic 会将会话状态变更 POST 到你的 HTTPS 端点——载荷精简（仅含 ID）、经 HMAC 签名、需在 Console 中注册。见 `shared/managed-agents-webhooks.md`。

**No-code inspection - the Console session viewer** (Console sidebar -> **Managed Agents** -> **Sessions**; Developers and Admins only). Point users here for debugging before they parse the stream themselves: a session list (ID, name, status, agent, tokens in/out, cost; filter by status/created, search by ID); a **timeline minimap** with one lane per thread in multiagent sessions; the **transcript** grouped by model request (thinking, tool calls with inputs/results, streaming text) with a **Filter events** box (matches ID, type, tool name, or text; Enter steps between matches) and copy/download-as-JSON (filtered export when a filter is active); and an **Inspector** side panel (toggle with `d`) with five tabs - **Session** (details, metadata, cumulative-cost chart vs. budget), **Events** (raw events in server order, JSON per event, plus a **Deltas** view for messages that streamed while the page was open), **Tools** (every configured tool with call counts, failures, median duration; jump to any call), **Resources** (mounted files, repos, memory stores with per-session memory changes, `/mnt/session/outputs` files, skills under `/workspace/skills`), **Threads** (status, context size, cost per thread; context-size chart for the current thread; switch threads). Deep-link with `?event={event_id}` on the session URL - handy to include in error reports alongside the Console link from `shared/managed-agents-core.md`.

**无代码检查 - Console 会话查看器**（Console 侧边栏 -> **Managed Agents** -> **Sessions**；仅限 Developers 和 Admins）。在用户自行解析事件流之前，可将他们引导到这里进行调试：会话列表（ID、名称、状态、智能体、输入/输出 token 数、成本；可按状态/创建时间过滤、按 ID 搜索）；多智能体会话中每个线程一条泳道的**时间线缩略图**；按模型请求分组的**对话记录**（思考过程、带输入/结果的工具调用、流式文本），配有 **Filter events** 搜索框（匹配 ID、类型、工具名称或文本；按 Enter 可在匹配项之间跳转）以及复制/下载为 JSON（启用过滤时导出过滤后的内容）；还有一个 **Inspector** 侧边面板（用 `d` 键切换），包含五个标签页——**Session**（详情、元数据、累计成本与预算对比图）、**Events**（按服务器顺序排列的原始事件、每个事件的 JSON，以及页面打开期间流式传输消息的 **Deltas** 视图）、**Tools**（每个已配置工具的调用次数、失败次数、中位耗时；可跳转到任一调用）、**Resources**（挂载的文件、仓库、内存存储及其会话级内存变更、`/mnt/session/outputs` 文件、`/workspace/skills` 下的技能）、**Threads**（每个线程的状态、上下文大小、成本；当前线程的上下文大小图表；可切换线程）。可在会话 URL 上用 `?event={event_id}` 进行深度链接——便于在错误报告中连同 `shared/managed-agents-core.md` 提供的 Console 链接一并列出。

All **persisted** events carry `id`, `type`, and `processed_at` (ISO 8601), set when the event finishes processing. On events you send, `processed_at` is `null` while the event is still queued behind earlier ones - **except** `user.define_outcome`, `user.custom_tool_result`, and `user.tool_result`, which are processed on receipt and echoed back with `processed_at` already populated. The stream-only `event_start` / `event_delta` preview events (see § Live previews) carry only the `id` of the event they preview.

所有**持久化**事件都带有 `id`、`type` 和 `processed_at`（ISO 8601），在事件处理完成时设置。对于你发送的事件，当事件仍在队列中排在更早事件之后时，`processed_at` 为 `null`——**但** `user.define_outcome`、`user.custom_tool_result` 和 `user.tool_result` 除外，它们在收到时即被处理并回显，`processed_at` 已填充。仅流中存在的 `event_start` / `event_delta` 预览事件（见 § Live previews）只携带它们所预览事件的 `id`。

> Warning: **Robust polling (raw HTTP).** If you bypass the SDK and roll your own poll loop, don't rely on `requests` or `httpx` timeouts as wall-clock caps - they're **per-chunk** read timeouts, reset every time a byte arrives. A trickling response (heartbeats, a wedged chunked-encoding body, a misbehaving proxy) can keep the call blocked indefinitely even with `timeout=(5, 60)` or `httpx.Timeout(120)`. Neither library has a "total wall-clock" timeout built in. For a hard deadline: track `time.monotonic()` at the loop level and break/cancel if a single request exceeds your budget (e.g. via a watchdog thread, or `asyncio.wait_for()` around async httpx). **Prefer the SDK** - `client.beta.sessions.events.stream()` and `client.beta.sessions.events.list()` handle timeout + retry sanely.  
>  
> 警示：**稳健的轮询（原始 HTTP）。**如果你绕过 SDK 自行编写轮询循环，不要把 `requests` 或 `httpx` 的超时当作墙钟时间上限——它们是**按分块**计的读取超时，每收到一个字节就会重置。一个缓慢滴漏的响应（心跳、卡住的 chunked 编码响应体、行为异常的代理）即使设置了 `timeout=(5, 60)` 或 `httpx.Timeout(120)`，也可能让调用无限期阻塞。这两个库都没有内置"总墙钟时间"超时。若需要硬性截止期限：在循环层跟踪 `time.monotonic()`，当单个请求超出预算时中断/取消（例如用看门狗线程，或在异步 httpx 外包一层 `asyncio.wait_for()`）。**优先使用 SDK**——`client.beta.sessions.events.stream()` 和 `client.beta.sessions.events.list()` 对超时与重试的处理更稳妥。
>
> If `GET /v1/sessions/{id}/events` (paginated) ever hangs after headers, you've likely hit `GET /v1/sessions/{id}/events/stream` by mistake or a server-side stall - report it; don't treat it as a client-config problem.
>
> 如果 `GET /v1/sessions/{id}/events`（分页）在响应头之后挂起，你很可能是误请求了 `GET /v1/sessions/{id}/events/stream`，或遇到了服务器端停滞——请上报，不要当作客户端配置问题。

### Event Types (Received) / 事件类型（接收）

Event types use dot notation, grouped by namespace:

事件类型采用点号命名法，按命名空间分组：

| Event Type | Description |
| --- | --- |
| `agent.message` | Agent text output |
| `agent.thinking` | Progress signal that the agent is thinking - it does **not** carry the thinking content |
| `agent.tool_use` | Agent used a built-in tool (`agent_toolset_20260401`). Carries `evaluated_permission` (`allow`/`ask`/`deny`) and usually `evaluation` - see `shared/managed-agents-tools.md` § `evaluated_permission` and `evaluation` |
| `agent.tool_result` | Result from a built-in tool |
| `agent.mcp_tool_use` | Agent used an MCP tool. Carries `evaluated_permission` and usually `evaluation`, same as `agent.tool_use` |
| `agent.mcp_tool_result` | Result from an MCP tool |
| `agent.custom_tool_use` | Agent invoked a custom tool - session goes idle, you respond with `user.custom_tool_result` |
| `agent.thread_context_compacted` | Conversation context was compacted |
| `session.status_idle` | Agent has finished the current task, and is awaiting input. It's either waiting for input to continue working via a `user.message`, blocked awaiting a `user.custom_tool_result` or `user.tool_confirmation`, or paused because the session budget cap was reached. The `stop_reason` attached contains more information about why the Agent has stopped working. |
| `session.status_running` | Session has starting running, and the Agent is actively doing work. |
| `session.status_rescheduled` | Session is (re)scheduling after a retryable error has occurred, ready to be picked up by the orchestration system. |
| `session.status_terminated` | Session ended and is irreversibly unusable - **on completion or on error**, not error-only. |
| `session.updated` | A session update changed at least one field - carries only the changed fields (a budget removal carries `budget: null`) |
| `session.usage` | Snapshot of the session's cumulative usage and tracked list cost - see § Reaching a session budget below |
| `session.error` | Error occurred during processing |
| `span.model_request_start` | Model inference started |
| `span.model_request_end` | Model inference completed |
| `span.outcome_evaluation_start` / `_ongoing` / `_end` | Grader progress for outcome-oriented sessions - see `shared/managed-agents-outcomes.md` |
| `session.thread_created` | Subagent thread spawned (multiagent), or an advisor consultation started (thread name `anthropic.advisor`) - see `shared/managed-agents-multiagent.md` |
| `session.thread_status_running` / `_idle` / `_rescheduled` / `_terminated` | Thread status transitions - mostly seen in multiagent sessions, but a single-agent session's primary thread also emits `_idle` when pausing at a session budget (§ Reaching a session budget). `_idle` carries `stop_reason`. |
| `agent.thread_message_sent` / `_received` | Cross-thread message, carries `to_session_thread_id` / `from_session_thread_id` (multiagent) |

| 事件类型 | 描述 |
| --- | --- |
| `agent.message` | 智能体的文本输出 |
| `agent.thinking` | 表示智能体正在思考的进度信号——它**不**携带思考内容 |
| `agent.tool_use` | 智能体使用了内置工具（`agent_toolset_20260401`）。携带 `evaluated_permission`（`allow`/`ask`/`deny`），通常还有 `evaluation`——见 `shared/managed-agents-tools.md` § `evaluated_permission` and `evaluation` |
| `agent.tool_result` | 内置工具的结果 |
| `agent.mcp_tool_use` | 智能体使用了 MCP 工具。携带 `evaluated_permission`，通常还有 `evaluation`，同 `agent.tool_use` |
| `agent.mcp_tool_result` | MCP 工具的结果 |
| `agent.custom_tool_use` | 智能体调用了自定义工具——会话转为空闲，你需以 `user.custom_tool_result` 响应 |
| `agent.thread_context_compacted` | 对话上下文已被压缩 |
| `session.status_idle` | 智能体已完成当前任务，正在等待输入。它或者在通过 `user.message` 等待继续工作的输入，或者阻塞等待 `user.custom_tool_result` 或 `user.tool_confirmation`，或者因会话预算上限已达而暂停。随附的 `stop_reason` 包含智能体为何停止工作的更多信息。 |
| `session.status_running` | 会话已开始运行，智能体正在积极工作。 |
| `session.status_rescheduled` | 发生可重试错误后会话正在（重新）调度，等待编排系统接手。 |
| `session.status_terminated` | 会话已结束且不可逆地不可用——**在完成或出错时**，而非仅在出错时。 |
| `session.updated` | 一次会话更新至少改变了一个字段——只携带被改变的字段（移除预算时携带 `budget: null`） |
| `session.usage` | 会话累计用量与记录清单成本的快照——见下文 § Reaching a session budget |
| `session.error` | 处理过程中发生错误 |
| `span.model_request_start` | 模型推理开始 |
| `span.model_request_end` | 模型推理完成 |
| `span.outcome_evaluation_start` / `_ongoing` / `_end` | 面向成果会话的评分进度——见 `shared/managed-agents-outcomes.md` |
| `session.thread_created` | 子智能体线程被派生（多智能体），或一次顾问咨询开始（线程名 `anthropic.advisor`）——见 `shared/managed-agents-multiagent.md` |
| `session.thread_status_running` / `_idle` / `_rescheduled` / `_terminated` | 线程状态转换——主要见于多智能体会话，但单智能体会话的主线程在会话预算处暂停时也会发出 `_idle`（§ Reaching a session budget）。`_idle` 携带 `stop_reason`。 |
| `agent.thread_message_sent` / `_received` | 跨线程消息，携带 `to_session_thread_id` / `from_session_thread_id`（多智能体） |

The stream also echoes back user-sent events (`user.message`, `user.interrupt`, `user.tool_confirmation`, `user.tool_result`, `user.custom_tool_result`, `user.define_outcome`) - except a `user.interrupt` sent while the session is paused at its budget, which is accepted and ignored and never appears (§ Reaching a session budget).

事件流还会回显用户发送的事件（`user.message`、`user.interrupt`、`user.tool_confirmation`、`user.tool_result`、`user.custom_tool_result`、`user.define_outcome`）——但会话在预算处暂停期间发送的 `user.interrupt` 除外，该事件会被接受并忽略，永不出现（§ Reaching a session budget）。

Stream-only delta preview events (`event_start`, `event_delta`) are the one exception to the `{domain}.{action}` naming convention - see § Live previews below; they never appear in `GET /v1/sessions/{id}/events`.

仅流中存在的增量预览事件（`event_start`、`event_delta`）是 `{domain}.{action}` 命名约定的唯一例外——见下文 § Live previews；它们从不出现在 `GET /v1/sessions/{id}/events` 中。

---

## Live previews / 实时预览

By default, assistant text reaches the stream as buffered `agent.message` events - emitted only after the model request that produced them finishes. **Live previews** let you render that text incrementally while the model is still generating. The buffered `agent.message` is always the authoritative record; a client that ignores previews still receives a complete, correct stream. The wire format is **not** Messages-API streaming: the delta type is `content_delta`, not `content_block_delta`, so Messages-API accumulator code does not carry over unchanged.

默认情况下，助手文本以缓冲的 `agent.message` 事件形式到达事件流——仅在产生它的模型请求结束后才发出。**实时预览**让你能在模型仍在生成时增量渲染该文本。缓冲的 `agent.message` 始终是权威记录；忽略预览的客户端仍会收到完整、正确的事件流。其线上格式**不是** Messages API 流式格式：增量类型是 `content_delta` 而非 `content_block_delta`，因此 Messages API 的累加器代码无法原样沿用。

**Opt in per stream connection** by adding the `event_deltas[]` query parameter, repeated once per event type to preview. Accepted values: `agent.message`, `agent.thinking` - any other value returns a 400, as does a request with more than 100 values. **Both stream endpoints accept it:** the session-level stream (`GET /v1/sessions/{id}/events/stream`) and each session thread's own stream (`GET /v1/sessions/{sid}/threads/{tid}/stream`). In a shell, quote the URL or percent-encode the brackets as `%5B%5D` - bare `[]` is a glob pattern.

**按流连接选择性开启**：添加 `event_deltas[]` 查询参数，每个要预览的事件类型重复一次。接受的值：`agent.message`、`agent.thinking`——任何其他值都会返回 400，超过 100 个值的请求同样返回 400。**两个流端点都接受该参数：**会话级流（`GET /v1/sessions/{id}/events/stream`）和每个会话线程自己的流（`GET /v1/sessions/{sid}/threads/{tid}/stream`）。在 shell 中，请给 URL 加引号或将方括号百分号编码为 `%5B%5D`——裸 `[]` 是通配符模式。

**Previews are thread-scoped.** A connection previews only the thread it is reading. A child thread's previews are delivered on that child's stream and are *never* cross-posted to the session-level stream, whose previews stay scoped to the primary thread. To watch a subagent's text as the model generates it, open that subagent's thread stream - see `shared/managed-agents-multiagent.md`. Run one accumulator instance per connection.

**预览以线程为作用域。**一条连接只预览它正在读取的线程。子线程的预览在该子线程自己的流上投递，*绝不会*跨投到会话级流，会话级流的预览始终局限于主线程。要在子智能体文本生成时实时查看，请打开该子智能体的线程流——见 `shared/managed-agents-multiagent.md`。每条连接运行一个累加器实例。

```python
stream = client.beta.sessions.events.stream(
    session_id=session.id,
    event_deltas=["agent.message"],
)
```

When a previewed event begins, the stream emits an `event_start` carrying the upcoming event's `type` and `id`; for `agent.message` it's followed by `event_delta` events carrying incremental text:

当被预览的事件开始时，流会发出一个携带即将到来事件的 `type` 和 `id` 的 `event_start`；对于 `agent.message`，其后跟随携带增量文本的 `event_delta` 事件：

```js
{"type": "event_start", "event": {"type": "agent.message", "id": "sevt_01abc..."}}
{"type": "event_delta", "event_id": "sevt_01abc...", "delta": {"type": "content_delta", "index": 0, "content": {"type": "text", "text": "Here is the summary"}}}
```

`event_start` and `event_delta` have no `id` or `processed_at` of their own - the only identifier they carry is the `id` of the event they preview. For `agent.thinking`, **only** the `event_start` is emitted (a "thinking has started" signal) - no deltas follow, and the buffered `agent.thinking` that concludes the preview carries no thinking content either. It is a progress signal, not a content carrier; there is nothing to read out of it.

`event_start` 和 `event_delta` 自身没有 `id` 或 `processed_at`——它们携带的唯一标识符是被预览事件的 `id`。对于 `agent.thinking`，**只**发出 `event_start`（一个"思考已开始"信号）——没有后续增量，且作为预览收尾的缓冲 `agent.thinking` 同样不携带思考内容。它是一个进度信号，而非内容载体；从中读不出任何内容。

**Accumulate-and-reconcile pattern.** Treat the preview as a scratch buffer keyed by `(event_id, index)`. On `event_start`, create an empty entry for the announced `id`. On each `event_delta`, append `delta.content.text` to `(event_id, delta.index)` and render the running text. When the buffered `agent.message` arrives, match it by `id`, **discard the accumulated preview**, and render the message's content instead. The identifiers always line up: `event_start.event.id`, every `event_delta.event_id`, and the buffered event's `id` are the same value. On a normal turn the order is fixed: `session.status_running` -> `span.model_request_start` -> `event_start` -> `event_delta`* -> buffered `agent.message` -> `span.model_request_end`. If the turn errors or is interrupted the buffered event may never arrive, but `span.model_request_end` still does - close any unreconciled preview when you see it. Python/TypeScript/Go SDKs ship an accumulator helper that implements this; in other SDKs apply the manual pattern to the generated event types.

**累加-对账模式。**把预览当作以 `(event_id, index)` 为键的暂存缓冲区。收到 `event_start` 时，为公告的 `id` 创建一个空条目。每收到一个 `event_delta`，把 `delta.content.text` 追加到 `(event_id, delta.index)` 并渲染当前累积文本。当缓冲的 `agent.message` 到达时，按 `id` 匹配，**丢弃累积的预览**，改为渲染消息内容。这些标识符总是对得上：`event_start.event.id`、每个 `event_delta.event_id` 与缓冲事件的 `id` 是同一个值。正常一轮的顺序是固定的：`session.status_running` -> `span.model_request_start` -> `event_start` -> `event_delta`* -> 缓冲的 `agent.message` -> `span.model_request_end`。如果该轮出错或被中断，缓冲事件可能永远不到达，但 `span.model_request_end` 仍会到达——看到它时就关闭任何未对账的预览。Python/TypeScript/Go SDK 附带实现了该模式的累加器助手；在其他 SDK 中，请将该手动模式应用于代码生成的事件类型。

**Two guarantees the pattern relies on:** concatenating a preview's deltas in arrival order, keyed by `(event_id, index)`, yields a *prefix* of `content[index].text` in the buffered event (a prefix, not necessarily the whole text - deltas may be shed under load); and a connection emits at most one `event_start` per `event_id`, with the buffered event as the last thing that connection delivers for that `id`.

**该模式依赖的两条保证：**按到达顺序、以 `(event_id, index)` 为键拼接一个预览的各增量，得到的*是*缓冲事件中 `content[index].text` 的*前缀*（是前缀，不一定是全部文本——高负载下增量可能被丢弃）；且每条连接对每个 `event_id` 至多发出一个 `event_start`，缓冲事件是该连接为该 `id` 投递的最后一项。

**Limitations:**

**限制：**

- **Best effort** - under load the server may shed deltas for an event; you receive a contiguous prefix and then no further deltas for that event. The buffered `agent.message` still arrives complete. Never treat an accumulated preview as final.
  **尽力而为** - 高负载下服务器可能丢弃某事件的增量；你会收到一段连续前缀，之后该事件不再有增量。缓冲的 `agent.message` 仍会完整到达。绝不要把累积的预览当作最终结果。
- **No replay on reconnect** - deltas are delivered only to the connection that opted in, while it's open; this holds for the session-level stream and each thread stream alike. A connection opened after a model request started receives no deltas for that in-flight event. After a drop, follow the consolidation pattern in § Reconnecting after a dropped stream - the history fetch returns any buffered events emitted during the gap; missed deltas cannot be re-requested.
  **重连后不重放** - 增量只投递给已选择开启且保持打开的那条连接；会话级流与各线程流皆然。在模型请求开始之后才打开的连接，收不到该进行中事件的增量。断线后，请遵循 § Reconnecting after a dropped stream 中的合并模式——历史拉取会返回间隔期间发出的缓冲事件；错过的增量无法重新请求。
- **One thread, text only** - previews cover assistant text on the thread the connection is reading. Tool use, tool results, MCP results, and activity on any *other* thread are never previewed on that connection.
  **仅单线程、仅文本** - 预览只覆盖连接正在读取的线程上的助手文本。工具使用、工具结果、MCP 结果以及任何*其他*线程上的活动都不会在该连接上预览。
- **Never persisted** - `event_start` / `event_delta` exist only on the live SSE stream, never in `GET /v1/sessions/{id}/events` or any thread's event history.
  **从不持久化** - `event_start` / `event_delta` 只存在于实时 SSE 流中，从不出现在 `GET /v1/sessions/{id}/events` 或任何线程的事件历史中。

**Troubleshooting:**

**故障排查：**

| You see | What it means |
| --- | --- |
| Buffered events but no `event_start` / `event_delta` | This connection didn't opt in (`event_deltas[]` is per connection, not per session), or the turn ran on a different thread. List `GET /v1/sessions/{sid}/threads` to find which one ran. |
| 404 on the stream URL | Wrong path or ID, or the request carries no managed-agents beta header - the thread endpoints are beta-gated, so without it they don't exist. The thread path is `/threads/{tid}/stream`, **not** `/threads/{tid}/events/stream` (which doesn't exist) and not `/events/stream` (session level only). |
| 400 naming `event_deltas` | Only `agent.message` and `agent.thinking` are accepted, max 100 values. |

| 现象 | 含义 |
| --- | --- |
| 有缓冲事件但没有 `event_start` / `event_delta` | 该连接未选择开启（`event_deltas[]` 按连接而非按会话生效），或该轮跑在另一个线程上。列出 `GET /v1/sessions/{sid}/threads` 找出是哪一个。 |
| 流 URL 返回 404 | 路径或 ID 错误，或请求未携带 managed-agents beta 头——线程端点受 beta 门控，没有该头它们就不存在。线程路径是 `/threads/{tid}/stream`，**不是** `/threads/{tid}/events/stream`（不存在），也不是 `/events/stream`（仅会话级）。 |
| 返回 400 并点名 `event_deltas` | 只接受 `agent.message` 和 `agent.thinking`，最多 100 个值。 |

---

## Steering Patterns / 转向控制模式

Practical patterns for driving a session via the events surface.

通过事件接口驱动会话的实用模式。

### Stream-first ordering / 流优先的顺序

**Open the stream before sending events.** The stream only delivers events that occur *after* it's opened - it does not replay current state or historical events. If you send a message first and open the stream second, early events (including fast status transitions) arrive buffered in a single batch and you lose the ability to react to them in real time.

**先开流，再发事件。**流只投递在它打开*之后*发生的事件——不重放当前状态或历史事件。如果你先发消息再开流，早期事件（包括快速的状态转换）会以单个批次缓冲到达，你将失去实时响应它们的能力。

```ts
// Correct - stream and send concurrently
const [response] = await Promise.all([
  streamEvents(sessionId),   // opens SSE connection
  sendMessage(sessionId, text),
]);

// Wrong - events before stream opens arrive as a single buffered batch
await sendMessage(sessionId, text);
const response = await streamEvents(sessionId);
```

**For full history,** use `GET /v1/sessions/{id}/events` (paginated list) - the stream only gives you live events from connection onward.

**要获取完整历史，**请使用 `GET /v1/sessions/{id}/events`（分页列表）——流只提供从连接建立起的实时事件。

### Reconnecting after a dropped stream / 流断开后的重连

**The SSE stream has no replay.** If your connection drops (httpx read timeout, network blip) and you reconnect, you only get events emitted *after* reconnection. Any events emitted during the gap are lost from the stream.

**SSE 流没有重放。**如果连接断开（httpx 读取超时、网络抖动）后重连，你只会得到重连*之后*发出的事件。间隔期间发出的任何事件都会从流中丢失。

**The consolidation pattern:** on every (re)connect, overlap the stream with a history fetch and dedupe by event ID:

**合并模式：**每次（重）连接时，让流与一次历史拉取重叠，并按事件 ID 去重：

```python
def connect_with_consolidation(client, session_id):
    # 1. Open the SSE stream first
    stream = client.beta.sessions.events.stream(session_id=session_id)

    # 2. Fetch history to cover any gap
    history = client.beta.sessions.events.list(
        session_id=session_id,
    )

    # 3. Yield history first, then stream - dedupe by event.id
    seen = set()
    for ev in history.data:
        seen.add(ev.id)
        yield ev
    for ev in stream:
        if ev.id not in seen:
            seen.add(ev.id)
            yield ev
```

### Message queuing / 消息排队

**You don't have to wait for a response before sending the next message.** User events are queued server-side and processed in order. This is useful for chat bridges where the user sends rapid follow-ups:

**发送下一条消息前不必等待响应。**用户事件在服务器端排队并按顺序处理。这对用户快速连发后续消息的聊天桥接场景很有用：

```ts
// All three go into one session; agent processes them in order
await sendMessage(sessionId, "Summarize the README");
await sendMessage(sessionId, "Actually also check the CONTRIBUTING guide");
await sendMessage(sessionId, "And compare the two");
// Stream once - agent responds to all three as a coherent turn
```

Events can be sent up to the Session at any time. There is no need to wait on a specific session status to enqueue new events via `client.beta.sessions.events.send()`. One exception: a session paused at its budget (`stop_reason: budget_reached`) accepts only settle events - a `user.message` there is a 400. See § Reaching a session budget.

事件可以随时发送给会话。无需等待特定会话状态即可通过 `client.beta.sessions.events.send()` 将新事件入队。一个例外：在预算处暂停的会话（`stop_reason: budget_reached`）只接受结算类事件——此时发送 `user.message` 会返回 400。见 § Reaching a session budget。

### Interrupt / 中断

A `user.interrupt` event **jumps the queue** (ahead of any pending user messages) and forces the session into `idle`. Exception: while the session is paused at its budget, an interrupt is accepted and ignored - it is never persisted and changes nothing (§ Reaching a session budget). Use this for "stop" / "nevermind" / "cancel" commands:

`user.interrupt` 事件会**插队**（越过任何待处理的用户消息）并强制会话进入 `idle`。例外：当会话在预算处暂停时，中断会被接受并忽略——它从不持久化，也不改变任何东西（§ Reaching a session budget）。用于"停止" / "算了" / "取消"类命令：

```ts
await client.beta.sessions.events.send(sessionId, {
  events: [{ type: 'user.interrupt' }],
});
```

The agent stops mid-task. It does not see the interrupt as a message - it just halts. Send a follow-up `user` event to explain what to do instead. If an outcome is active, the interrupt also marks `span.outcome_evaluation_end.result: "interrupted"` (see `shared/managed-agents-outcomes.md`) - though not at a budget pause, where the interrupt is accepted and ignored (see § Reaching a session budget).

智能体会在任务中途停止。它不会把中断看作一条消息——只是就地停止。请再发送一条后续 `user` 事件说明接下来该做什么。如果有成果评估在进行，中断还会标记 `span.outcome_evaluation_end.result: "interrupted"`（见 `shared/managed-agents-outcomes.md`）——但在预算暂停处除外，那里中断会被接受并忽略（见 § Reaching a session budget）。

**The interrupted turn ends with `stop_reason: end_turn`** - the same value a turn that finishes on its own carries. There is no interruption-specific stop reason, so a drain loop can't distinguish the two from `stop_reason` alone; track that you sent the interrupt.

**被中断的一轮以 `stop_reason: end_turn` 结束**——与自行完成的一轮携带的值相同。不存在中断专用的停止原因，因此消费循环无法仅凭 `stop_reason` 区分两者；请自行记录你发送过中断。

**Against an already-`idle` session an interrupt is normally a no-op.** The exception is a session on a self-hosted environment whose worker failed the claimed work item (a memory-store mount error, for instance): it sits `idle` with `stop_reason: requires_action` and no error event, and `user.interrupt` re-queues the work for the next worker claim (`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores -> Troubleshooting).

**对已经处于 `idle` 的会话，中断通常是无操作。**例外是自托管环境上 worker 认领的工作项失败的会话（例如内存存储挂载错误）：它停留在 `idle`，`stop_reason: requires_action` 且没有错误事件，此时 `user.interrupt` 会把该工作重新入队等待下一次 worker 认领（`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores -> Troubleshooting）。

**In a multiagent session, omitting `session_thread_id` interrupts every non-archived thread, including the primary** - it is not primary-only. Pass `session_thread_id` to stop one thread. See `shared/managed-agents-multiagent.md`.

**在多智能体会话中，省略 `session_thread_id` 会中断每个未归档的线程，包括主线程**——并非只针对主线程。传入 `session_thread_id` 可只停止一个线程。见 `shared/managed-agents-multiagent.md`。

> **Note**: Interrupt events may have empty IDs in the current implementation. When troubleshooting, use the `processed_at` timestamp along with surrounding event IDs. (Not applicable to an interrupt sent at the budget cap - that event is never persisted, so there is nothing to locate.)
>
> **注意**：当前实现中，中断事件的 ID 可能为空。排查问题时，请结合 `processed_at` 时间戳与周围事件的 ID 定位。（对预算上限处发送的中断不适用——该事件从不持久化，因此无从定位。）

### Reaching a session budget / 达到会话预算

A session created with a budget (see `shared/managed-agents-core.md` § Session budgets) pauses instead of overspending. Before every model request the platform checks whether consumed list cost has reached the cap and pauses the thread if it has, and the session goes idle with `stop_reason: budget_reached` rather than terminating. On the stream, the pause arrives as three events, in order:

带预算创建的会话（见 `shared/managed-agents-core.md` § Session budgets）会暂停而不是超支。每次模型请求前，平台会检查已消耗的清单成本是否达到上限，若已达到则暂停该线程，会话转入空闲并带 `stop_reason: budget_reached`，而不是终止。在流上，暂停以三个事件按顺序到达：

1. `session.thread_status_idle` with `stop_reason: budget_reached`, for each thread as it pauses. When a thread's final request both crosses the cap and finishes its turn, that thread reports `stop_reason: end_turn` while the session still reports `budget_reached` - key on the **session-level** `stop_reason`, not thread-level ones, to detect the pause.
   1. `session.thread_status_idle` 带 `stop_reason: budget_reached`，每个线程暂停时各一条。当线程的最后一次请求既越过上限又完成了它的轮次时，该线程报告 `stop_reason: end_turn`，而会话仍报告 `budget_reached`——要检测暂停，请以**会话级** `stop_reason` 为准，而非线程级的。
2. `session.usage` - a snapshot of the session's cumulative usage and tracked list cost.
   2. `session.usage` - 会话累计用量与记录清单成本的快照。
3. `session.status_idle` with `stop_reason: budget_reached`. The `session.usage` event always immediately precedes this idle.
   3. `session.status_idle` 带 `stop_reason: budget_reached`。`session.usage` 事件总是紧邻此次空闲之前。

While at the cap the session accepts **only settle events** (`user.tool_confirmation`, `user.tool_result`, `user.custom_tool_result`, `user.interrupt`); anything that starts new work, including `user.message`, is a 400 naming that list. A `user.interrupt` sent while the session is paused at its budget (all threads paused at the cap) is accepted and ignored: it does not appear in the event list and changes nothing. Raise or remove the budget to continue. When one thread waits on a tool ask and another is paused at the cap, the session-level `stop_reason` is `requires_action`, not `budget_reached` - settling the ask doesn't trigger a model request, so respond as usual.

处于上限时，会话**只接受结算类事件**（`user.tool_confirmation`、`user.tool_result`、`user.custom_tool_result`、`user.interrupt`）；任何会开启新工作的事件，包括 `user.message`，都会返回 400 并点名该列表。在会话因预算暂停（所有线程都停在上限）期间发送的 `user.interrupt` 会被接受并忽略：它不出现在事件列表中，也不改变任何东西。要继续，请调高或移除预算。当一个线程在等待工具审批而另一个线程停在上限时，会话级 `stop_reason` 是 `requires_action` 而非 `budget_reached`——结算该审批不会触发模型请求，因此照常响应即可。

【评论】预算暂停期间"接受但忽略"中断事件的设计保持了事件确认语义的统一，但调用方无法从回执上察觉该事件实际未生效。

**No event resumes a session paused at its cap.** Update the session's budget instead: change it to a value above the consumed list cost (higher or lower than the old cap), or remove it with `"budget": null`. An accepted update resumes the paused work automatically.

**没有任何事件能恢复停在上限的会话。**请改为更新会话的预算：把它改为高于已消耗清单成本的值（无论高于还是低于旧上限），或用 `"budget": null` 移除。被接受的更新会自动恢复暂停的工作。

**`session.usage`** carries the session's cumulative token totals, `list_cost` (`{amount, currency}`, rounded to the nearest cent), `active_seconds` (concurrent-thread overlap counted once - the figure runtime cost is priced on), `server_tool_use` counts (`web_search_requests`, and `web_fetch_requests` - informational, currently always 0 since web fetch is not metered), and an echo of the session's `budget` when one is set. It appears in the events list and the session stream - a stream reader sees the final cost of the work that hit the cap without an extra fetch; child threads' own streams do not carry it. The same totals live on the session object's `usage` field, and each thread's own `usage` carries per-thread `list_cost` and `active_seconds` - but per-thread costs do **not** sum to the session total: the session figure additionally includes session running time and each figure is rounded independently, so the session figure is the authoritative one. To enforce a spend limit, set a budget rather than polling usage and interrupting the session yourself - the platform's gate runs before each model request.

**`session.usage`** 携带会话的累计 token 总量、`list_cost`（`{amount, currency}`，四舍五入到分）、`active_seconds`（并发线程重叠只计一次——即运行时成本计价所依据的数字）、`server_tool_use` 计数（`web_search_requests` 和 `web_fetch_requests`——仅供参考，由于 web fetch 未计量，目前始终为 0），以及会话设置了预算时对其 `budget` 的回显。它出现在事件列表和会话流中——流读取者无需额外拉取即可看到触及上限那项工作的最终成本；子线程自己的流不携带它。同样的总量也存在于会话对象的 `usage` 字段上，每个线程自己的 `usage` 携带按线程的 `list_cost` 和 `active_seconds`——但按线程的成本**不会**加总等于会话总量：会话数字额外包含会话运行时间，且每个数字独立舍入，因此会话数字才是权威值。要执行支出限制，请设置预算而不是轮询用量并自行中断会话——平台的闸门在每次模型请求前运行。

### Event payloads / 事件载荷

some events carry useful metadata beyond the status change itself:

一些事件除状态变更本身外还携带有用的元数据：

`session.status_idle` - includes a `stop_reason` field which elaborates on why the session stopped and what type of further action is required by the user.  

`session.status_idle` - 包含一个 `stop_reason` 字段，详述会话为何停止以及需要用户采取何种后续操作。  

```json
{
  "id": "sevt_456",
  "processed_at": "2026-04-07T04:27:43.197Z",
  "stop_reason": {
    "event_ids": [
      "sevt_123"
    ],
    "type": "requires_action"
  },
  "type": "status_idle"
}
```

`span.model_request_end` contains a `model_usage` field for cost tracking and efficiency analysis:

`span.model_request_end` 包含用于成本跟踪与效率分析的 `model_usage` 字段：

```json
{
  "type": "span.model_request_end",
  "id": "sevt_456",
  "is_error": false,
  "model_request_start_id": "sevt_123",
  "model_usage": {
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 6656,
    "input_tokens": 3571,
    "output_tokens": 727
  },
  "processed_at": "2026-04-07T04:11:32.189Z"
}
```

**`agent.thread_context_compacted`** - emitted when the conversation history was summarized to fit context. Includes `pre_compaction_tokens` so you know how much was squeezed:

**`agent.thread_context_compacted`** - 当对话历史被摘要以适配上下文时发出。包含 `pre_compaction_tokens`，让你知道压缩掉了多少：

```json
{
  "id": "sevt_abc123",
  "processed_at": "2026-03-24T14:05:15.787Z",
  "type": "agent.thread_context_compacted"
}
```

### Archive / 归档

When done with a session, archive it to free resources:

会话用完后，归档它以释放资源：

```ts
await client.beta.sessions.archive(sessionId);
```

> Archiving a **session** is routine cleanup - sessions are per-run and disposable. **Do not generalize this to agents or environments**: those are persistent, reusable resources, and archiving them is permanent (no unarchive; new sessions cannot reference them). See `shared/managed-agents-overview.md` -> Common Pitfalls.
>
> 归档**会话**是常规清理——会话按次运行、用完即弃。**不要把这一点推广到智能体或环境**：它们是持久、可复用的资源，归档它们是永久性的（无法取消归档；新会话无法引用它们）。见 `shared/managed-agents-overview.md` -> Common Pitfalls。
