<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Webhooks / 托管代理——Webhook

Anthropic can POST to your HTTPS endpoint when a Managed Agents resource changes state - an alternative to holding an SSE stream or polling. Payloads are **thin** (event type + resource IDs only); on receipt, fetch the resource for current state. Every delivery is HMAC-signed.

当 Managed Agents 资源改变状态时，Anthropic 可以向你的 HTTPS 端点发起 POST——这是保持 SSE 流或轮询之外的另一种选择。负载是**薄的**（仅含事件类型和资源 ID）；收到后请拉取资源获取当前状态。每次投递都经 HMAC 签名。

> **Direction matters.** This page covers *Anthropic -> you* notifications about session/vault state. It does **not** cover *third-party -> you* webhooks that *trigger* a session (e.g. a GitHub push handler that calls `sessions.create()`) - that's ordinary application code on your side with no Anthropic-specific wire format.

> **方向很重要。**本页介绍的是*Anthropic -> 你*关于会话/保管库状态的通知。它**不**介绍*第三方 -> 你*的、用于*触发*会话的 webhook（例如调用 `sessions.create()` 的 GitHub push 处理器）——那只是你一侧的普通应用代码，不涉及 Anthropic 特定的报文格式。

---

## Register an endpoint (Console only) / 注册端点（仅限 Console）

Console -> **Manage -> Webhooks**. There is no programmatic endpoint-management API yet. Secret rotation is supported from the same page.

Console -> **Manage -> Webhooks**。目前没有编程式的端点管理 API。密钥轮换也可在同一页面操作。

| Field | Constraint |
|---|---|
| URL | HTTPS on port 443, publicly resolvable hostname |
| Event types | Subscribe per `data.type` - an endpoint receives only the types it is subscribed to |
| Signing secret | `whsec_`-prefixed, 32 bytes, **shown once at creation** - store it |

| 字段 | 约束 |
|---|---|
| URL | 443 端口上的 HTTPS，公开可解析的主机名 |
| 事件类型 | 按 `data.type` 订阅——端点只接收其订阅的类型 |
| 签名密钥 | `whsec_` 前缀、32 字节，**仅在创建时显示一次**——务必保存 |

---

## Verify the signature / 校验签名

Every delivery carries the `webhook-id`, `webhook-timestamp`, and `webhook-signature` headers. **Use the SDK's `client.beta.webhooks.unwrap()`** - it verifies the signature, rejects payloads more than ~5 minutes old, and returns the parsed event. It reads the `whsec_` secret from `ANTHROPIC_WEBHOOK_SIGNING_KEY`. Pass the headers through untouched; don't hand-roll verification against a single `X-Webhook-Signature` header, which is not the wire format.

每次投递都带有 `webhook-id`、`webhook-timestamp` 和 `webhook-signature` 头。**请使用 SDK 的 `client.beta.webhooks.unwrap()`**——它会校验签名、拒绝超过约 5 分钟的旧负载，并返回解析后的事件。它从 `ANTHROPIC_WEBHOOK_SIGNING_KEY` 读取 `whsec_` 密钥。头信息原样透传；不要针对单个 `X-Webhook-Signature` 头自行实现校验，那不是线上格式。

```python
import anthropic
from flask import Flask, request

client = anthropic.Anthropic()  # reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env
app = Flask(__name__)


@app.route("/webhook", methods=["POST"])
def webhook():
    try:
        event = client.beta.webhooks.unwrap(
            request.get_data(as_text=True),
            headers=dict(request.headers),
        )
    except Exception:
        return "invalid signature", 400

    if event.id in seen_event_ids:  # dedupe retries - id is per-event, not per-delivery
        return "", 204
    seen_event_ids.add(event.id)

    match event.data.type:
        case "session.status_idled":
            session = client.beta.sessions.retrieve(event.data.id)
            notify_user(session)
        case "vault_credential.refresh_failed":
            alert_oncall(event.data.id)

    return "", 204
```

Pass the **raw request body** to `unwrap()` - frameworks that re-serialize JSON (Express `.json()`, Flask `.get_json()`) change the bytes and break the MAC. For other languages, look up the `beta.webhooks.unwrap` binding in the SDK repo (`shared/live-sources.md`); don't hand-roll verification.

把**原始请求体**传给 `unwrap()`——会重新序列化 JSON 的框架（Express 的 `.json()`、Flask 的 `.get_json()`）会改变字节并破坏 MAC。其他语言请在 SDK 仓库中查阅 `beta.webhooks.unwrap` 绑定（`shared/live-sources.md`）；不要自行实现校验。

---

## Payload envelope / 负载信封

```json
{
  "type": "event",
  "id": "whe_9d5c1f7e...",
  "created_at": "2026-03-18T14:05:22Z",
  "data": {
    "type": "session.status_idled",
    "id": "session_01XYZ...",
    "organization_id": "8a3d2f1e-...",
    "workspace_id": "c7b0e4d9-..."
  }
}
```

Switch on `data.type`, fetch the resource by `data.id`, return any **2xx** to acknowledge. `created_at` is when the *event occurred*, not when the delivery was attempted - the `webhook-timestamp` header is the clock for the attempt (see Delivery behavior).

按 `data.type` 分支处理，按 `data.id` 拉取资源，返回任意 **2xx** 以确认。`created_at` 是*事件发生*的时间，而不是尝试投递的时间——`webhook-timestamp` 头才是投递尝试的时钟（见投递行为一节）。

The top-level `id` is the same value as the `webhook-id` header, and it is per *event*, not per delivery - every retry carries it unchanged. Dedupe on it.

顶层 `id` 与 `webhook-id` 头的值相同，它按*事件*计，而不是按投递计——每次重试都原样携带。以此去重。

---

## Supported `data.type` values / 支持的 `data.type` 值

| `data.type` | Fires when |
|---|---|
| `session.status_scheduled` | Session created and ready to accept events |
| `session.status_run_started` | Agent execution kicked off (every transition to `running`) |
| `session.status_idled` | Agent awaiting input (tool approval, custom tool result, or next message) - or paused at its session budget. The webhook payload is thin - list the session's events and check the latest `session.status_idle` event's `stop_reason` (the session object itself has no `stop_reason` field): if it is `budget_reached`, further `user.message` events return a 400 and only a budget change/removal resumes the session (`shared/managed-agents-core.md` § Session budgets) |
| `session.status_rescheduled` | A transient error occurred; the session is retrying automatically |
| `session.status_terminated` | Session ended - **on completion or on error**, not error-only |
| `session.thread_created` | Multiagent: coordinator opened a new subagent thread, or the session's advisor is being consulted (`shared/managed-agents-multiagent.md` -> Advisor) |
| `session.thread_idled` | Child threads only: a subagent thread is waiting for input - or paused because the session reached its budget cap. When the whole session pauses at the cap, a `session.status_idled` webhook also fires and the stream's `session.status_idle` event carries `stop_reason: budget_reached` - unless another thread is waiting on a tool ask, which outranks the cap at the session level (`shared/managed-agents-core.md` § Session budgets). |
| `session.thread_terminated` | A thread ended - child completed its work, or the thread was archived. **Child threads only**; the primary thread's end surfaces as `session.status_terminated` |
| `session.outcome_evaluation_ended` | Outcome grader finished one iteration |
| `session.updated` | Session properties changed (name, configuration) |
| `session.deleted` | Session permanently deleted - no object left to fetch; treat the event itself as final |
| `vault.archived` | Vault was archived |
| `vault.created` | Vault was created |
| `vault.deleted` | Vault was deleted - a `vault_credential.deleted` also fires per underlying credential. No object left to fetch; treat the event itself as final |
| `vault_credential.archived` | Credential archived, directly or via vault archival |
| `vault_credential.created` | Vault credential was created |
| `vault_credential.deleted` | Credential deleted, directly or via vault deletion. No object left to fetch; treat the event itself as final |
| `vault_credential.refresh_failed` | MCP OAuth vault credential failed to refresh |
| `agent.created` | Agent created |
| `agent.updated` | A new agent version was published. Updates that do not create a new version do **not** fire this. |
| `agent.archived` | Agent archived |
| `agent.deleted` | Agent permanently deleted - no object left to fetch; treat the event itself as final |
| `deployment.created` | Scheduled deployment created |
| `deployment.updated` | Deployment properties changed (e.g. schedule edited) |
| `deployment.paused` | Deployment paused - by request, or automatically when a scheduled run fails with a **non-recoverable** error (archived agent, missing environment). Recoverable failures, including rate limits, do **not** auto-pause. |
| `deployment.unpaused` | Deployment unpaused; schedule resumes |
| `deployment.archived` | Deployment archived - directly, or as a result of agent archival/deletion |
| `deployment.deleted` | Deployment permanently deleted - no object left to fetch; treat the event itself as final |
| `deployment_run.started` | A **scheduled** run started. Manual runs do **not** emit `deployment_run.*` events. |
| `deployment_run.succeeded` | Scheduled run created its session. Same `data.id` (the run ID) as the run's `.started` event - fetch the deployment run for its `session_id`, then subscribe to the session events to follow the work. |
| `deployment_run.failed` | Scheduled run did not create a session. Same `data.id` as the run's `.started` event - fetch the deployment run for `error.type` / `error.message`. |
| `environment.created` | Environment created |
| `environment.updated` | Environment updated with at least one changed field. A no-op update emits nothing. |
| `environment.archived` | Environment archived. Re-archiving an already-archived environment emits nothing. |
| `environment.deleted` | Environment deleted, including delete of an already-archived one. No object left to fetch; treat the event itself as final |
| `memory_store.created` | Memory store created - by you, or by an Anthropic-operated process that clones one of your stores |
| `memory_store.archived` | Memory store archived. Re-archiving an already-archived store emits nothing. |
| `memory_store.deleted` | Memory store deleted, including delete of an already-archived one. Cascades to its memories and versions **without** per-memory events - this single event is the signal. No object left to fetch; treat it as final |

| `data.type` | 触发时机 |
|---|---|
| `session.status_scheduled` | 会话已创建，可开始接收事件 |
| `session.status_run_started` | 代理执行已启动（每次转入 `running` 时） |
| `session.status_idled` | 代理等待输入（工具审批、自定义工具结果或下一条消息）——或因会话预算而暂停。webhook 负载很薄——列出该会话的事件，查看最新 `session.status_idle` 事件的 `stop_reason`（会话对象本身没有 `stop_reason` 字段）：若为 `budget_reached`，后续 `user.message` 事件会返回 400，只有更改/移除预算才能恢复会话（`shared/managed-agents-core.md` § Session budgets） |
| `session.status_rescheduled` | 发生了暂时性错误；会话正在自动重试 |
| `session.status_terminated` | 会话已结束——**完成或出错时都会触发**，不限于出错 |
| `session.thread_created` | 多代理：协调者开启了新的子代理线程，或正在咨询会话的顾问（advisor）（`shared/managed-agents-multiagent.md` -> Advisor） |
| `session.thread_idled` | 仅限子线程：某个子代理线程在等待输入——或因会话达到预算上限而暂停。当整个会话在上限处暂停时，也会发出一个 `session.status_idled` webhook，且流中的 `session.status_idle` 事件带有 `stop_reason: budget_reached`——除非另有线程在等待工具询问（tool ask），它在会话层级的优先级高于预算上限（`shared/managed-agents-core.md` § Session budgets）。 |
| `session.thread_terminated` | 某个线程已结束——子线程完成了工作，或该线程被归档。**仅限子线程**；主线程的结束表现为 `session.status_terminated` |
| `session.outcome_evaluation_ended` | 成果评分器完成了一次迭代 |
| `session.updated` | 会话属性发生变化（名称、配置） |
| `session.deleted` | 会话被永久删除——已无对象可取；把该事件本身视为最终状态 |
| `vault.archived` | 保管库（vault）被归档 |
| `vault.created` | 保管库已创建 |
| `vault.deleted` | 保管库被删除——每个底层凭证还会各发出一个 `vault_credential.deleted`。已无对象可取；把该事件本身视为最终状态 |
| `vault_credential.archived` | 凭证被归档，直接归档或随保管库归档 |
| `vault_credential.created` | 保管库凭证已创建 |
| `vault_credential.deleted` | 凭证被删除，直接删除或随保管库删除。已无对象可取；把该事件本身视为最终状态 |
| `vault_credential.refresh_failed` | MCP OAuth 保管库凭证刷新失败 |
| `agent.created` | 代理已创建 |
| `agent.updated` | 发布了新的代理版本。不产生新版本的更新**不会**触发本事件。 |
| `agent.archived` | 代理被归档 |
| `agent.deleted` | 代理被永久删除——已无对象可取；把该事件本身视为最终状态 |
| `deployment.created` | 定时部署已创建 |
| `deployment.updated` | 部署属性发生变化（如修改了计划） |
| `deployment.paused` | 部署被暂停——应请求暂停，或当定时运行因**不可恢复**错误（代理已归档、环境缺失）而失败时自动暂停。可恢复的失败（包括速率限制）**不会**导致自动暂停。 |
| `deployment.unpaused` | 部署已恢复暂停；计划继续执行 |
| `deployment.archived` | 部署被归档——直接归档，或因代理归档/删除所致 |
| `deployment.deleted` | 部署被永久删除——已无对象可取；把该事件本身视为最终状态 |
| `deployment_run.started` | 一次**定时**运行已启动。手动运行**不会**发出 `deployment_run.*` 事件。 |
| `deployment_run.succeeded` | 定时运行已创建其会话。`data.id`（运行 ID）与该运行的 `.started` 事件相同——取部署运行详情获得其 `session_id`，再订阅会话事件以跟进工作。 |
| `deployment_run.failed` | 定时运行未能创建会话。`data.id` 与该运行的 `.started` 事件相同——取部署运行详情查看 `error.type` / `error.message`。 |
| `environment.created` | 环境已创建 |
| `environment.updated` | 环境已更新且至少有一个字段变化。无实质变更的更新不发出任何事件。 |
| `environment.archived` | 环境被归档。对已归档环境再次归档不发出任何事件。 |
| `environment.deleted` | 环境被删除，包括删除已归档的环境。已无对象可取；把该事件本身视为最终状态 |
| `memory_store.created` | 记忆库已创建——由你创建，或由克隆你某个记忆库的 Anthropic 运营进程创建 |
| `memory_store.archived` | 记忆库被归档。对已归档记忆库再次归档不发出任何事件。 |
| `memory_store.deleted` | 记忆库被删除，包括删除已归档的记忆库。会级联删除其记忆和版本，且**不会**发出逐条记忆的事件——这一个事件就是全部信号。已无对象可取；把它视为最终状态 |

> **There is deliberately no `memory_store.updated`.** Individual memories and memory versions emit no webhook events at all, and neither do an environment's self-hosted work items. If you need per-memory change tracking, poll the memory-versions endpoints (`shared/managed-agents-memory.md`).

> **刻意不设 `memory_store.updated`。**单条记忆和记忆版本完全不发出 webhook 事件，环境的自托管工作项也同样不发出。如果需要逐条记忆的变更追踪，请轮询 memory-versions 端点（`shared/managed-agents-memory.md`）。

> These are **webhook** `data.type` values - a separate namespace from SSE event types (`session.status_idle`, `span.outcome_evaluation_end`, etc. in `shared/managed-agents-events.md`). Don't reuse SSE constants in webhook handlers.

> 这些是 **webhook** 的 `data.type` 值——与 SSE 事件类型（`shared/managed-agents-events.md` 中的 `session.status_idle`、`span.outcome_evaluation_end` 等）是两个独立的命名空间。不要在 webhook 处理器中复用 SSE 常量。

---

## Delivery behavior & pitfalls / 投递行为与陷阱

- **Duplicates.** An endpoint can receive the same event more than once; every attempt carries the same top-level `event.id` (= the `webhook-id` header). Dedupe on it.
  **重复投递。**同一事件可能不止一次到达端点；每次尝试都带有相同的顶层 `event.id`（即 `webhook-id` 头）。以此为去重键。
- **Subscription scope.** An event reaches only endpoints subscribed to its type **at the moment it is emitted**. An event emitted while nothing was subscribed is never delivered, and subscribing later does not backfill - subscribe before you need the type.
  **订阅范围。**事件只送达**在事件发出那一刻**已订阅其类型的端点。发出时无人订阅的事件永远不会投递，事后订阅也不补发——在需要该类型之前先订阅。
- **No ordering guarantee.** Events are not delivered in occurrence order: `session.status_idled` may arrive before `session.outcome_evaluation_ended`, and a `.deleted` can arrive before the `.archived` for the same resource. **Drive state from the resource you fetch, not from arrival order.**
  **无顺序保证。**事件不按发生顺序投递：`session.status_idled` 可能先于 `session.outcome_evaluation_ended` 到达，同一资源的 `.deleted` 也可能先于 `.archived` 到达。**状态以你取回的资源为准，而不是以到达顺序为准。**
- **Retries: up to three attempts** per endpoint per event, with jittered exponential backoff between 5 and 120 seconds. A response that triggers auto-disable is never retried. **After the last attempt fails the event is dropped** - not queued, and with no signal that it was lost. Webhooks are not a durable log: if you must observe every transition, reconcile by listing or fetching the resource.
  **重试：**每个端点对每个事件最多尝试三次，采用 5 到 120 秒之间带抖动的指数退避。会触发自动停用的响应绝不重试。**最后一次尝试失败后事件即被丢弃**——不入队，也没有任何丢失信号。webhook 不是持久日志：如果必须观察到每一次状态迁移，请通过列举或拉取资源来对账。
- **`webhook-timestamp` is re-stamped on every attempt**, so retries don't fail the SDK's five-minute freshness check. It times the *delivery attempt*; use the payload's `created_at` for when the event occurred.
  **`webhook-timestamp` 在每次尝试时都会重新打戳**，因此重试不会导致 SDK 的五分钟新鲜度检查失败。它记录的是*投递尝试*的时间；事件发生时间请用负载中的 `created_at`。
- **Auto-disable - three triggers**, each setting `disabled_reason`, all reversible from Console (events emitted while disabled are **not** replayed):
  **自动停用——三种触发条件**，各自设置 `disabled_reason`，都可在 Console 中恢复（停用期间发出的事件**不会**重放）：
  - A `3xx` response. Redirects are never followed; disables immediately, on the first attempt. Reason: `auto-disabled: endpoint URL returned a redirect (3xx)`.
    返回 `3xx` 响应。重定向从不跟随；首次尝试即立即停用。原因：`auto-disabled: endpoint URL returned a redirect (3xx)`。
  - The URL resolves to a non-public IP at connect time. Disables immediately. Reason: `auto-disabled: endpoint URL resolved to an invalid address`.
    连接时 URL 解析到非公网 IP。立即停用。原因：`auto-disabled: endpoint URL resolved to an invalid address`。
  - Continuous failure for a sustained period. Reason: `auto-disabled after sustained delivery failures`. **The trigger is duration, not a delivery count** - a single `2xx` resets the window, so one flaky event can't disable the endpoint.
    持续一段时间连续失败。原因：`auto-disabled after sustained delivery failures`。**触发条件是持续时间，而不是投递次数**——一次 `2xx` 就会重置时间窗，因此单个不稳定事件无法停用端点。
- **Thin payload is intentional.** Don't expect `stop_reason` (list the session's events for that - the session object has no `stop_reason` field), `outcome_evaluations`, credential secrets, etc. on the webhook body - fetch the resource.
  **负载刻意做薄。**不要指望 webhook 请求体里会有 `stop_reason`（这需要列出会话事件——会话对象没有 `stop_reason` 字段）、`outcome_evaluations`、凭证机密等内容——去拉取资源。

【评论】"URL 解析到非公网 IP 即自动停用"是一条内置的防 SSRF 措施，阻止 webhook 被用作探测内网地址的跳板；签名校验、五分钟新鲜度窗口与按事件 ID 去重则构成投递完整性的标准组合。
