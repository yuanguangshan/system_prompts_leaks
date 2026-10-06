<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Outcomes / Managed Agents - 成果（Outcomes）

An **outcome** elevates a session from *conversation* to *work*: you state what "done" looks like, and the harness runs an iterate -> grade -> revise loop until the artifact meets the rubric, hits `max_iterations`, or is interrupted. A separate **grader** (independent context window) scores each iteration against your rubric and feeds per-criterion gaps back to the agent.

**成果（outcome）** 把会话从*对话*提升为*工作*：你陈述"完成"是什么样子，框架（harness）随即运行"迭代 -> 评分 -> 修订"循环，直到产物满足量规、达到 `max_iterations` 或被中断为止。一个独立的**评分器（grader）**（拥有独立的上下文窗口）依据你的量规为每次迭代打分，并把逐项标准的差距反馈给智能体。

The SDK sets the `managed-agents-2026-04-01` beta header automatically on all `client.beta.sessions.*` calls; no additional header is required for outcomes.

SDK 会在所有 `client.beta.sessions.*` 调用上自动设置 `managed-agents-2026-04-01` beta 头；使用 outcomes 无需额外的头。

**Codegen default:** when generating Managed Agents client code for a task with a deliverable (an artifact, a report, a PR, a dataset), kick the session off with `user.define_outcome` rather than `user.message` - draft a starter rubric from the task description and mark it for the user to tune. Reserve plain `user.message` kickoffs for genuinely conversational sessions.

**代码生成默认规则：** 在为带有交付物（产物、报告、PR、数据集）的任务生成 Managed Agents 客户端代码时，用 `user.define_outcome` 而不是 `user.message` 来启动会话——根据任务描述起草一份初始量规，并标注出来供用户调整。朴素的 `user.message` 启动方式仅保留给真正的对话式会话。

---

## The `user.define_outcome` event / `user.define_outcome` 事件

Outcomes are not a field on `sessions.create()`. You create a normal session, then send a `user.define_outcome` event. The agent starts working on receipt - **do not also send a `user.message`** to kick it off.

Outcomes 不是 `sessions.create()` 上的字段。你先创建一个普通会话，然后发送一个 `user.define_outcome` 事件。智能体在收到后即开始工作——**不要再发送 `user.message`** 来启动它。

You can collapse both calls into one by passing a single `user.define_outcome` in the session's `initial_events` array - same event, same rules, one round trip (see `shared/managed-agents-core.md` -> Seeding a session with `initial_events`). More than one `user.define_outcome` in that array, or one without a `rubric`, rejects the whole create with a 400.

你可以在会话的 `initial_events` 数组中传入单个 `user.define_outcome`，把两次调用合并为一次——同一事件、同一规则、一次往返（参见 `shared/managed-agents-core.md` -> Seeding a session with `initial_events`）。该数组中出现多个 `user.define_outcome`，或某个缺少 `rubric`，都会使整个创建请求以 400 被拒绝。

```python
session = client.beta.sessions.create(
    agent=AGENT_ID,
    environment_id=ENVIRONMENT_ID,
    title="Financial analysis on Costco",
)

client.beta.sessions.events.send(
    session_id=session.id,
    events=[
        {
            "type": "user.define_outcome",
            "description": "Build a DCF model for Costco in .xlsx",
            "rubric": {"type": "text", "content": RUBRIC_MD},
            # or: "rubric": {"type": "file", "file_id": rubric.id}
            "max_iterations": 5,  # optional; default 3, max 20
        }
    ],
)
```

| Field | Type | Notes |
|---|---|---|
| `type` | `"user.define_outcome"` | |
| `description` | string | The task. This is what the agent works toward - no separate `user.message` needed. |
| `rubric` | `{type: "text", content}` \| `{type: "file", file_id}` | **Required.** Markdown with explicit, independently gradeable criteria. Upload once via `client.files.upload(...)` to reuse across sessions. |
| `max_iterations` | int | Optional. Default **3**, max **20**. |

| 字段 | 类型 | 说明 |
|---|---|---|
| `type` | `"user.define_outcome"` | |
| `description` | string | 任务内容。这就是智能体努力的目标——无需单独发送 `user.message`。 |
| `rubric` | `{type: "text", content}` \| `{type: "file", file_id}` | **必填。** Markdown 格式，包含明确、可独立评分的标准。可通过 `client.files.upload(...)` 上传一次，跨会话复用。 |
| `max_iterations` | int | 可选。默认 **3**，最大 **20**。 |

The event is echoed back on the stream with a server-assigned `outcome_id` and `processed_at`.

该事件会在流上回显，附带服务端分配的 `outcome_id` 和 `processed_at`。

> **Writing rubrics.** Use explicit, gradeable criteria ("CSV has a numeric `price` column"), not vibes ("data looks good") - the grader scores each criterion independently, so vague criteria produce noisy loops. If you don't have a rubric, have Claude analyze a known-good artifact and turn that analysis into one. When generating code for a user who supplied no rubric, draft one yourself from their task description - 5-10 concrete criteria covering the artifact's format, required content, and quality floor - and comment it as a starter rubric to tune; never omit the outcome because the rubric wasn't handed to you.

> **撰写量规。** 使用明确、可评分的标准（"CSV 含有数值型的 `price` 列"），而不要用感觉（"数据看起来不错"）——评分器会对每条标准独立打分，模糊的标准会产生噪音很大的循环。如果你没有量规，可以让 Claude 分析一个已知良好的产物，并把该分析转化为量规。在为未提供量规的用户生成代码时，根据其任务描述自己起草一份——5 到 10 条具体标准，覆盖产物的格式、必备内容与质量下限——并以注释标明这是供调整的初始量规；绝不要因为用户没给你量规就省略 outcome。

---

## Outcome-specific events / outcome 专属事件

These appear on the standard event stream (`sessions.events.stream` / `.list`) alongside the usual `agent.*` / `session.*` events.

这些事件出现在标准事件流（`sessions.events.stream` / `.list`）上，与常见的 `agent.*` / `session.*` 事件并列。

| Event | Payload highlights | Meaning |
|---|---|---|
| `span.outcome_evaluation_start` | `outcome_id`, `iteration` (0-indexed) | Grader began scoring iteration *N*. |
| `span.outcome_evaluation_ongoing` | `outcome_id` | Heartbeat while the grader runs. Grader reasoning is opaque - you see *that* it's working, not *what* it's thinking. |
| `span.outcome_evaluation_end` | `outcome_evaluation_start_id`, `outcome_id`, `iteration`, `result`, `explanation`, `usage` | Grader finished one iteration. `result` drives what happens next (table below). |

| 事件 | 载荷要点 | 含义 |
|---|---|---|
| `span.outcome_evaluation_start` | `outcome_id`、`iteration`（从 0 开始） | 评分器开始为第 *N* 次迭代打分。 |
| `span.outcome_evaluation_ongoing` | `outcome_id` | 评分器运行期间的心跳。评分器的推理是不透明的——你只能看到它*正在工作*，看不到它*在想什么*。 |
| `span.outcome_evaluation_end` | `outcome_evaluation_start_id`、`outcome_id`、`iteration`、`result`、`explanation`、`usage` | 评分器完成一次迭代。`result` 决定接下来发生什么（见下表）。 |

### `span.outcome_evaluation_end.result`

| `result` | Next |
|---|---|
| `satisfied` | Session -> `idle`. Terminal for this outcome. |
| `needs_revision` | Agent starts another iteration. |
| `max_iterations_reached` | No further grader cycles. Agent may run one final revision, then session -> `idle`. |
| `failed` | Session -> `idle`. Rubric fundamentally doesn't match the task (e.g. description and rubric contradict). |
| `interrupted` | Emitted whenever a `user.interrupt` arrives while an outcome is active - **even if evaluation hadn't started**. In that case `outcome_evaluation_start_id` is an empty string rather than an event ID, so don't use it as a lookup key without checking. (Except an interrupt sent while paused at the session budget, which is accepted and ignored - see `shared/managed-agents-events.md` § Reaching a session budget.) |

| `result` | 后续 |
|---|---|
| `satisfied` | 会话 -> `idle`。该 outcome 终结。 |
| `needs_revision` | 智能体开始下一次迭代。 |
| `max_iterations_reached` | 不再进行评分循环。智能体可执行最后一次修订，然后会话 -> `idle`。 |
| `failed` | 会话 -> `idle`。量规与任务根本不匹配（例如 description 与量规互相矛盾）。 |
| `interrupted` | 只要 outcome 处于活动状态时到达 `user.interrupt` 就会发出——**即使评估尚未开始**。此时 `outcome_evaluation_start_id` 是空字符串而不是事件 ID，因此在未检查之前不要把它当作查找键。（例外：在会话因预算暂停期间发送的中断会被接受但忽略——参见 `shared/managed-agents-events.md` § Reaching a session budget。） |

```json
{
  "type": "span.outcome_evaluation_end",
  "id": "sevt_01jkl...",
  "outcome_evaluation_start_id": "sevt_01def...",
  "outcome_id": "outc_01a...",
  "result": "satisfied",
  "explanation": "All 12 criteria met: revenue projections use 5 years of historical data, ...",
  "iteration": 0,
  "usage": { "input_tokens": 2400, "output_tokens": 350, "cache_creation_input_tokens": 0, "cache_read_input_tokens": 1800 },
  "processed_at": "2026-03-25T14:03:00Z"
}
```

---

## Checking status & retrieving deliverables / 检查状态与获取交付物

**Status** - either watch the stream for `span.outcome_evaluation_end`, or poll the session and read `outcome_evaluations`:

**状态**——要么在流上监听 `span.outcome_evaluation_end`，要么轮询会话并读取 `outcome_evaluations`：

```python
session = client.beta.sessions.retrieve(session.id)
for ev in session.outcome_evaluations:
    print(f"{ev.outcome_id}: {ev.result}")  # outc_01a...: satisfied
```

**Deliverables** - the agent writes to `/mnt/session/outputs/`. Once idle, fetch via the Files API with `scope_id=session.id`. This is the same session-outputs mechanism documented in `shared/managed-agents-environments.md` -> Session outputs (including the `managed-agents-2026-04-01` header that `files.list` needs for `scope_id`).

**交付物**——智能体会写入 `/mnt/session/outputs/`。会话空闲后，通过 Files API 以 `scope_id=session.id` 获取。这与 `shared/managed-agents-environments.md` -> Session outputs 中记载的是同一套会话输出机制（包括 `files.list` 使用 `scope_id` 时所需的 `managed-agents-2026-04-01` 头）。

---

## Interaction rules & pitfalls / 交互规则与陷阱

- **One outcome at a time.** Chain by sending the next `user.define_outcome` only after the previous one's terminal `span.outcome_evaluation_end` (`satisfied` / `max_iterations_reached` / `failed` / `interrupted`). The session retains history across chained outcomes.
  **一次只进行一个 outcome。** 串接方式为：只有在上一个 outcome 出现终结性的 `span.outcome_evaluation_end`（`satisfied` / `max_iterations_reached` / `failed` / `interrupted`）之后，才发送下一个 `user.define_outcome`。会话会在串接的 outcome 之间保留历史。
- **Steering is allowed but optional.** You *may* send `user.message` events mid-outcome to nudge direction, but the agent already knows to keep working until terminal - don't send "keep going" prompts. (Exception: a session paused at its budget (`stop_reason: budget_reached`) accepts only settle events - a steering `user.message`, or a chained `user.define_outcome`, is a 400 there; see `shared/managed-agents-events.md` § Reaching a session budget.)
  **允许转向但并非必需。** 你*可以*在 outcome 进行中发送 `user.message` 事件来微调方向，但智能体已经知道要持续工作到终结状态——不要发送"继续干"之类的提示。（例外：因预算暂停的会话（`stop_reason: budget_reached`）只接受结算类事件——转向用的 `user.message` 或串接的 `user.define_outcome` 在那里会返回 400；参见 `shared/managed-agents-events.md` § Reaching a session budget。）
- **`user.interrupt` pauses the current outcome** - it marks `result: "interrupted"` and leaves the session `idle`, ready for a new outcome or conversational turn. (Exception: sent while paused at the session budget, the interrupt is accepted and ignored and the outcome stays active - see `shared/managed-agents-events.md` § Reaching a session budget.)
  **`user.interrupt` 会暂停当前 outcome**——它将结果标记为 `result: "interrupted"` 并让会话回到 `idle`，可开始新的 outcome 或对话轮次。（例外：在会话因预算暂停期间发送时，该中断会被接受但忽略，outcome 保持活动——参见 `shared/managed-agents-events.md` § Reaching a session budget。）
- **After terminal, the session is reusable** - continue conversationally or define a new outcome.
  **终结之后会话可复用**——可以继续对话，也可以定义新的 outcome。
- **Outcome != session-create field.** Don't put `outcome`, `rubric`, or `description` on `sessions.create()` - outcomes are always sent as a `user.define_outcome` event.
  **Outcome 不是会话创建字段。** 不要把 `outcome`、`rubric` 或 `description` 放到 `sessions.create()` 上——outcome 总是以 `user.define_outcome` 事件发送。
- **Idle-break gate is unchanged.** In your drain loop, keep using `event.type === 'session.status_idle' && event.stop_reason?.type !== 'requires_action'` - do **not** gate on `span.outcome_evaluation_end` alone (on `needs_revision` the session keeps running). See `shared/managed-agents-client-patterns.md` Pattern 5.
  **空闲退出判据不变。** 在你的排空循环中，继续使用 `event.type === 'session.status_idle' && event.stop_reason?.type !== 'requires_action'`——**不要**仅以 `span.outcome_evaluation_end` 作为退出判据（出现 `needs_revision` 时会话仍在运行）。参见 `shared/managed-agents-client-patterns.md` Pattern 5。

For the raw HTTP shapes and per-language SDK bindings beyond Python, WebFetch `https://platform.claude.com/docs/en/managed-agents/define-outcomes.md` (see `shared/live-sources.md`).

原始 HTTP 形态以及 Python 之外各语言的 SDK 绑定，请 WebFetch `https://platform.claude.com/docs/en/managed-agents/define-outcomes.md`（参见 `shared/live-sources.md`）。
