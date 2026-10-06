<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Common Client Patterns / 托管代理——常见客户端模式

Patterns you'll write on the client side when driving a Managed Agent session, grounded in working SDK examples.

在驱动托管代理（Managed Agent）会话时，你将在客户端侧编写的各种模式，全部基于可运行的 SDK 示例。

Code samples are TypeScript - other languages follow the same shape; see `{lang}/managed-agents/README.md` (cURL and C#: `curl/managed-agents.md`) for equivalents.

代码示例为 TypeScript——其他语言结构相同；等效实现参见 `{lang}/managed-agents/README.md`（cURL 与 C# 见 `curl/managed-agents.md`）。

---

## 1. Lossless stream reconnect / 无损流式重连

**Problem:** SSE has no replay. If the connection drops mid-session, a naive reconnect re-opens the stream from "now" and you silently miss every event emitted in between.

**问题：** SSE 不支持重放。如果连接在会话中途断开，朴素的重新连接会从"当前"时刻重新打开流，你会无声地错过其间发出的所有事件。

**Solution:** on reconnect, fetch the full event history via `events.list()` *before* consuming the live stream, and dedupe on event ID as the live stream catches up.

**解决方案：** 重连时，*先*通过 `events.list()` 获取完整事件历史，再消费实时流，并在实时流追赶上来时按事件 ID 去重。

```ts
const seenEventIds = new Set<string>()
const stream = await client.beta.sessions.events.stream(session.id)

// Stream is now open and buffering server-side. Read history first.
for await (const event of client.beta.sessions.events.list(session.id)) {
  seenEventIds.add(event.id)
  handle(event)
}

// Tail the live stream. Dedupe only gates handle() - terminal checks must run
// even for already-seen events, or a terminal event that was in the history
// response gets skipped by `continue` and the loop never exits.
for await (const event of stream) {
  if (!seenEventIds.has(event.id)) {
    seenEventIds.add(event.id)
    handle(event)
  }
  if (event.type === 'session.status_terminated') break
  if (event.type === 'session.status_idle' && event.stop_reason.type !== 'requires_action') break
}
```

---

## 2. `processed_at` - queued vs processed / `processed_at`——排队与已处理

Every event on the stream carries `processed_at` (ISO 8601), set when the event finishes processing. For client-sent events (`user.message`, `user.interrupt`, `user.tool_confirmation`) it's `null` while the event is queued behind earlier ones, and populated once the agent processes it - so the same event appears on the stream twice, once with `null` and once with a timestamp. (Exception: a `user.interrupt` sent while the session is paused at its budget is accepted and ignored - it never appears at all; see `shared/managed-agents-events.md` § Reaching a session budget.)

流上的每个事件都带有 `processed_at`（ISO 8601），在事件处理完成时设置。对客户端发送的事件（`user.message`、`user.interrupt`、`user.tool_confirmation`）而言，当事件排在先前事件之后排队时该值为 `null`，代理处理完成后才会填充——因此同一事件会在流上出现两次，一次为 `null`，一次带时间戳。（例外：在会话因预算而暂停期间发送的 `user.interrupt` 会被接受但忽略——它根本不会出现；参见 `shared/managed-agents-events.md` § Reaching a session budget。）

**Three event types skip the queued phase:** `user.define_outcome`, `user.custom_tool_result`, and `user.tool_result` are processed on receipt and echoed back with `processed_at` already populated. A pending -> acknowledged UI that assumes "first sighting is always `null`" will never clear for these - treat a populated `processed_at` on first sighting as immediately acknowledged.

**三类事件会跳过排队阶段：** `user.define_outcome`、`user.custom_tool_result` 和 `user.tool_result` 在收到时即被处理，回显时 `processed_at` 已填充。如果"待处理 -> 已确认"的 UI 假设"首次出现时必为 `null`"，那么对这些事件该状态将永远不会清除——应在首次出现时就把已填充的 `processed_at` 视为立即已确认。

```ts
for await (const event of stream) {
  if (event.type === 'user.message') {
    if (event.processed_at == null) onQueued(event.id)
    else onProcessed(event.id, event.processed_at)
  }
}
```

Use this to drive pending -> acknowledged UI state for anything you send. How you map a locally-rendered optimistic message to the server-assigned `event.id` is application-specific (typically via the return value of `events.send()` or FIFO ordering).

用它为你发送的任何内容驱动"待处理 -> 已确认"的 UI 状态。如何把本地渲染的乐观更新消息映射到服务器分配的 `event.id` 属于应用特定逻辑（通常通过 `events.send()` 的返回值或 FIFO 顺序实现）。

---

## 3. Interrupt a running session / 中断正在运行的会话

Send `user.interrupt` as a normal event. The session keeps running until it reaches a safe boundary, then goes idle.

把 `user.interrupt` 作为普通事件发送。会话会继续运行直到到达安全边界，然后进入空闲状态。

```ts
await client.beta.sessions.events.send(session.id, {
  events: [{ type: 'user.interrupt' }],
})

// Drain until the session is truly done - see Pattern 5 for the full gate.
for await (const event of stream) {
  if (event.type === 'session.status_terminated') break
  if (
    event.type === 'session.status_idle' &&
    event.stop_reason.type !== 'requires_action'
  ) break
}
```

Reference: `interrupt.ts` - sends the interrupt the moment it sees `span.model_request_start`, drains to idle, then verifies via `sessions.retrieve()`.

参考：`interrupt.ts`——在看到 `span.model_request_start` 的瞬间发送中断，排空至空闲，然后通过 `sessions.retrieve()` 验证。

---

## 4. `tool_confirmation` round-trip / `tool_confirmation` 往返流程

When a call evaluates to `ask` - the tool has `permission_policy: { type: 'always_ask' }`, or it has `{ type: 'auto' }` and the server reached no determination - the `agent.tool_use` / `agent.mcp_tool_use` event carries `evaluated_permission === 'ask'` and the session goes idle waiting for a decision. Respond with `user.tool_confirmation`.

当某次调用的评估结果为 `ask` 时——即工具设置了 `permission_policy: { type: 'always_ask' }`，或设置为 `{ type: 'auto' }` 且服务器未能作出判定——`agent.tool_use` / `agent.mcp_tool_use` 事件会携带 `evaluated_permission === 'ask'`，会话进入空闲等待决策。用 `user.tool_confirmation` 作出响应。

```ts
for await (const event of stream) {
  if ((event.type === 'agent.tool_use' || event.type === 'agent.mcp_tool_use') && event.evaluated_permission === 'ask') {
    await client.beta.sessions.events.send(session.id, {
      events: [{
        type: 'user.tool_confirmation',
        tool_use_id: event.id,         // not a toolu_ id - use event.id
        result: 'allow',               // or 'deny'
        // deny_message: '...',        // optional, only with result: 'deny'
      }],
    })
  }
}
```

Key points:

要点：

- `tool_use_id` is `event.id` (typically `sevt_...`), **not** a `toolu_...` ID.
  `tool_use_id` 是 `event.id`（通常为 `sevt_...`），**而不是** `toolu_...` 形式的 ID。
- `result` is `'allow' | 'deny'`. Use `deny_message` to tell the model *why* you denied - it gets surfaced back to the agent.
  `result` 为 `'allow' | 'deny'`。用 `deny_message` 告诉模型你拒绝的*原因*——它会回传给代理。
- Multiple pending tools: respond once per `agent.tool_use` / `agent.mcp_tool_use` event with `evaluated_permission === 'ask'`.
  多个待处理工具：对每个带 `evaluated_permission === 'ask'` 的 `agent.tool_use` / `agent.mcp_tool_use` 事件各响应一次。
- Gate on `evaluated_permission === 'ask'`, not on the policy you configured - it covers `always_ask` and `auto`-indeterminate alike. Calls the server **denies** under `auto` (`evaluated_permission === 'deny'`, `evaluation.evaluated_permission.reason_code === 'high_risk'`) never enter this flow: the agent gets an error tool result and the session keeps running; sending a confirmation for one is a 400.
  以 `evaluated_permission === 'ask'` 作为判断条件，而不是你配置的策略——它同时覆盖 `always_ask` 和 `auto` 下无法判定两种情况。服务器在 `auto` 下**拒绝**的调用（`evaluated_permission === 'deny'`，`evaluation.evaluated_permission.reason_code === 'high_risk'`）不会进入此流程：代理会收到一个错误的工具结果且会话继续运行；为这类调用发送确认会得到 400 错误。
- Log `event.evaluation` for audit (`type` + `reason_code`), and tolerate a `type` or `reason_code` you don't recognize - branch on known values, pass unknown ones through.
  记录 `event.evaluation` 以便审计（`type` + `reason_code`），并容忍你不认识的 `type` 或 `reason_code`——对已知值分支处理，未知值直接透传。

Reference: `tool-permissions.ts`.

参考：`tool-permissions.ts`。

---

## 5. Correct idle-break gate / 正确的空闲退出判断

Do not break on `session.status_idle` alone. The session goes idle transiently - e.g. between parallel tool executions, while waiting for a `user.tool_confirmation`, or while awaiting a `user.custom_tool_result`. Break when idle with a non-`requires_action` `stop_reason` (terminal, or `budget_reached` - resumable only by a budget update, so break unless you intend to change or remove the budget), or on `session.status_terminated`.

不要仅凭 `session.status_idle` 就退出循环。会话会短暂进入空闲——例如并行工具执行之间、等待 `user.tool_confirmation` 时、或等待 `user.custom_tool_result` 时。应在空闲且 `stop_reason` 不是 `requires_action` 时退出（终止态，或 `budget_reached`——只能通过更新预算恢复，因此除非你打算修改或移除预算，否则应退出），或在 `session.status_terminated` 时退出。

```ts
for await (const event of stream) {
  handle(event)
  if (event.type === 'session.status_terminated') break
  if (event.type === 'session.status_idle') {
    if (event.stop_reason.type === 'requires_action') continue // waiting on you - handle it
    break // end_turn, retries_exhausted, or budget_reached - see list below
  }
}
```

`stop_reason.type` values on `session.status_idle`:

`session.status_idle` 上 `stop_reason.type` 的取值：

- `requires_action` - agent is waiting on a client-side event (tool confirmation, custom tool result). Handle it, don't break. **Self-hosted exception:** if the session went `requires_action`-idle with no pending `agent.tool_use` / `agent.mcp_tool_use` (`ask`) or `agent.custom_tool_use` to answer, the worker failed the claimed work item (typically a memory-store mount error, logged only on the worker host). Don't `continue` forever on that - surface it, fix the host, and send `user.interrupt` to re-queue the work (`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores -> Troubleshooting).
  `requires_action`——代理正在等待客户端侧事件（工具确认、自定义工具结果）。处理它，不要退出循环。**自托管例外：** 如果会话进入 `requires_action` 空闲，却没有任何待应答的 `agent.tool_use` / `agent.mcp_tool_use`（`ask`）或 `agent.custom_tool_use`，说明 worker 未能完成其认领的工作项（通常是内存存储挂载错误，日志只记录在 worker 主机上）。不要对此无限 `continue`——把问题暴露出来，修复主机，并发送 `user.interrupt` 重新排队该工作（`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores -> Troubleshooting）。
- `retries_exhausted` - terminal failure. Break, then check `sessions.retrieve()` for the error state.
  `retries_exhausted`——终态失败。退出循环，然后通过 `sessions.retrieve()` 查看错误状态。
- `end_turn` - normal completion.
  `end_turn`——正常完成。
- `budget_reached` - the session hit its spend cap and paused. Not terminal and not resumable by any event: change (typically raise) or remove the session's `budget` to resume, or treat it as done. A `session.usage` event with the final cost immediately precedes this idle. See `shared/managed-agents-core.md` § Session budgets.
  `budget_reached`——会话达到消费上限并暂停。既非终止态也无法通过任何事件恢复：需修改（通常是调高）或移除会话的 `budget` 才能恢复，或视其为已结束。此空闲状态之前会紧邻出现一个携带最终费用的 `session.usage` 事件。参见 `shared/managed-agents-core.md` § Session budgets。

---

## 6. Post-idle status-write race / 空闲之后的状态写入竞态

The SSE stream emits `session.status_idle` slightly before the session's queryable status reflects it. Clients that break on idle and immediately call `sessions.delete()` or `sessions.archive()` will intermittently 400 with "cannot delete/archive while running."

SSE 流发出 `session.status_idle` 的时间略早于会话可查询状态实际更新之时。在空闲时退出循环并立即调用 `sessions.delete()` 或 `sessions.archive()` 的客户端，会间歇性地收到 400 错误："cannot delete/archive while running."

Poll before cleanup:

清理前先轮询：

```ts
let s
for (let i = 0; i < 10; i++) {
  s = await client.beta.sessions.retrieve(session.id)
  if (s.status !== 'running') break
  await new Promise(r => setTimeout(r, 200))
}
if (s?.status !== 'running') {
  await client.beta.sessions.archive(session.id)
} // else: still running after 2s - don't archive, let it settle or escalate
```

---

## 7. Stream-first, then send / 先开流，再发送

Always open the stream **before** sending the kickoff event. Otherwise the agent may process the event and emit the first events before your consumer is attached, and you'll miss them.

始终在发送启动事件**之前**先打开流。否则代理可能在你的消费者挂载之前就处理事件并发出最初的事件，导致你错过它们。

```ts
const stream = await client.beta.sessions.events.stream(session.id)
await client.beta.sessions.events.send(session.id, {
  events: [{ type: 'user.message', content: [{ type: 'text', text: 'Hello' }] }],
})
for await (const event of stream) { /* ... */ }
```

The `Promise.all([stream, send])` shape works too, but stream-first is simpler and has the same effect - the stream starts buffering the moment it's opened.

`Promise.all([stream, send])` 的写法也可行，但先开流更简单且效果相同——流一打开就开始缓冲。

---

## 8. File-mount gotchas / 文件挂载的陷阱

**The mounted resource has a different `file_id` than the file you uploaded.** Session creation makes a session-scoped copy.

**挂载资源的 `file_id` 与你上传的文件不同。** 创建会话时会生成一份会话作用域的副本。

```ts
const uploaded = await client.beta.files.upload({ file, purpose: 'agent_resource' })
// uploaded.id         -> the original file
const session = await client.beta.sessions.create({
  /* ... */
  resources: [{ type: 'file', file_id: uploaded.id, mount_path: '/workspace/data.csv' }],
})
// session.resources[0].file_id !== uploaded.id  <- different IDs
```

Delete the original via `files.delete(uploaded.id)`; the session-scoped copy is garbage-collected with the session. `mount_path` must be absolute - see `shared/managed-agents-environments.md`.

通过 `files.delete(uploaded.id)` 删除原文件；会话作用域的副本会随会话一起被垃圾回收。`mount_path` 必须是绝对路径——参见 `shared/managed-agents-environments.md`。

---

## 9. Secrets for non-MCP APIs and CLIs - keep them host-side via custom tools / 非 MCP API 与 CLI 的密钥——通过自定义工具保留在主机侧

**Problem:** you want the agent to call a third-party API or run a CLI that needs a secret (API key, token, service-account credential), but you can't or don't want to hand the secret to a vault.

**问题：** 你想让代理调用需要密钥（API key、token、服务账号凭据）的第三方 API 或运行 CLI，但你不能或不想把该密钥交给保险库（vault）。

**First check:** for cloud environments, the first-class answer is now a vault `environment_variable` credential - the agent's shell sees an opaque placeholder and the real secret is substituted at egress. See `shared/managed-agents-tools.md` -> Vaults. Use this pattern instead when that doesn't fit: **self-hosted sandboxes** (env-var credentials not yet supported there), clients that reject the placeholder via local format validation, secrets that must never leave your infrastructure, or calls that need host-side binaries.

**先行检查：** 对云环境而言，现在的一等方案是保险库 `environment_variable` 凭据——代理的 shell 看到的是一个不透明的占位符，真实密钥在出口处替换。参见 `shared/managed-agents-tools.md` -> Vaults。当该方案不适用时改用本模式：**自托管沙箱**（尚不支持环境变量凭据）、会通过本地格式校验拒绝占位符的客户端、绝不能离开你基础设施的密钥，或需要主机侧二进制文件的调用。

**Solution:** move the authenticated call to your side. Declare a custom tool on the agent; when the agent emits `agent.custom_tool_use`, your orchestrator (the process reading the SSE stream) executes the call with its own credentials and responds with `user.custom_tool_result`. The container never sees the key.

**解决方案：** 把需要认证的调用移到你这一侧。在代理上声明一个自定义工具；当代理发出 `agent.custom_tool_use` 时，你的编排器（即读取 SSE 流的进程）用它自己的凭据执行该调用，并以 `user.custom_tool_result` 响应。容器永远看不到密钥。

```ts
// Agent template: declare the tool, no credentials
tools: [{ type: 'custom', name: 'linear_graphql', input_schema: { /* query, vars */ } }]

// Orchestrator: handle the call with host-side creds
for await (const event of stream) {
  if (event.type === 'agent.custom_tool_use' && event.name === 'linear_graphql') {
    const result = await linear.request(event.input.query, event.input.vars) // host's key
    await client.beta.sessions.events.send(session.id, {
      events: [{
        type: 'user.custom_tool_result',
        custom_tool_use_id: event.id,
        content: [{ type: 'text', text: JSON.stringify(result) }],
      }],
    })
  }
}
```

Same shape works for `gh` CLI, local eval scripts, or anything else that needs host-side auth or binaries.

同样的结构也适用于 `gh` CLI、本地评估脚本，或任何其他需要主机侧认证或二进制文件的场景。

**Security note:** this does not expose a public endpoint. `agent.custom_tool_use` arrives on the SSE stream your orchestrator already holds open with your Anthropic API key, and `user.custom_tool_result` goes back via `events.send()` under the same key. Your orchestrator is a client, not a server - nothing unauthenticated is listening.

**安全说明：** 这不会暴露任何公共端点。`agent.custom_tool_use` 到达所用的 SSE 流，是你的编排器已用你的 Anthropic API key 保持打开的流；`user.custom_tool_result` 也通过同一密钥经 `events.send()` 送回。你的编排器是客户端而非服务器——没有任何未经认证的东西在监听。

【评论】该模式让密钥只存在于宿主进程、只经已认证的 SSE 通道往返，避免向沙箱容器分发凭据；是一种"凭据不落沙箱"的常见编排设计。

**Do not embed API keys in the system prompt or user messages as a workaround.** Prompts and messages are stored in the session's event history, returned by `events.list()`, and included in compaction summaries - a secret placed there is durably persisted and readable via the API for the life of the session.

**不要把 API key 嵌入系统提示词或用户消息作为变通手段。** 提示词和消息会存入会话的事件历史、由 `events.list()` 返回，并被纳入压缩（compaction）摘要——放在那里的密钥会被持久保存，并在会话存续期间可经 API 读取。

【评论】此处点明了一个容易被忽视的风险面：会话事件历史与压缩摘要都会持久化提示词内容，因此任何进入提示词的敏感信息都会随会话存档而扩大暴露面。
