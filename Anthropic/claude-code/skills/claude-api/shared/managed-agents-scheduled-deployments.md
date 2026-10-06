<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Scheduled Deployments / Managed Agents - 定时部署

A **scheduled deployment** runs an agent on a recurring cron schedule - each firing creates a session autonomously. Use it for predictable-cadence work: nightly triage, weekly compliance scans, hourly monitors.

**定时部署（scheduled deployment）**按周期性的 cron 计划运行一个智能体——每次触发都会自主创建一个会话。适用于节奏可预期的工作：每夜分诊、每周合规扫描、每小时监控。

Requires the `managed-agents-2026-04-01` beta header (the SDK sets it automatically for `client.beta.deployments.*` / `client.beta.deployment_runs.*` calls).

需要 `managed-agents-2026-04-01` beta 请求头（对 `client.beta.deployments.*` / `client.beta.deployment_runs.*` 调用，SDK 会自动设置）。

## Create a deployment / 创建部署

A deployment bundles everything a session needs (agent, environment, optional files / GitHub / memory stores / vaults) plus a `schedule` and the `initial_events` that kick off each run:

部署（deployment）打包了会话所需的一切（智能体、环境，可选的文件 / GitHub / 记忆存储 / 保险库），外加一个 `schedule` 以及启动每次运行的 `initial_events`：

- `agent` and `environment_id` are required - same shapes as `sessions.create` (see `shared/managed-agents-core.md`). A deployment targeting a **self-hosted** environment can attach `memory_store` resources (SDK worker required - `shared/managed-agents-self-hosted-sandboxes.md` § Memory stores); `file` and `github_repository` resources need a cloud environment. The Console deployment form doesn't offer memory stores for self-hosted environments - attach them via the API/SDK.
  `agent` 与 `environment_id` 为必填——结构与 `sessions.create` 相同（见 `shared/managed-agents-core.md`）。面向**自托管（self-hosted）**环境的部署可以附加 `memory_store` 资源（需要 SDK worker——见 `shared/managed-agents-self-hosted-sandboxes.md` 的"Memory stores"一节）；`file` 与 `github_repository` 资源则需要云环境。Console 的部署表单不为自托管环境提供记忆存储选项——请通过 API/SDK 附加。
- `initial_events` must contain at least one starting event - a `user.message` **or** a `user.define_outcome`. Same default as sessions: a scheduled run that produces a deliverable (the weekly report, the compliance scan's findings file, a dataset) starts with `user.define_outcome` plus a drafted starter rubric (`shared/managed-agents-outcomes.md`); use `user.message` only when the run is genuinely conversational or has no checkable output. (A deployment's `initial_events` also accepts `system.message`, which a session's does not.)
  `initial_events` 必须包含至少一个起始事件——`user.message` **或** `user.define_outcome`。与会话相同的默认规则：产出交付物的定时运行（周报、合规扫描的结果文件、数据集）应以 `user.define_outcome` 加上起草好的初始评分标准（rubric）开始（见 `shared/managed-agents-outcomes.md`）；只有当运行确实是对话式的、或没有可检查的输出时才使用 `user.message`。（部署的 `initial_events` 还接受 `system.message`，而会话不接受。）
- `schedule` takes a cron `expression` and an IANA `timezone`. Minute-level granularity is the maximum.
  `schedule` 接受 cron `expression` 与 IANA `timezone`。最高粒度为分钟级。

```bash
curl -fsSL https://api.anthropic.com/v1/deployments \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01" \
  -H "content-type: application/json" \
  -d @- <<EOF
{
  "name": "Weekly compliance scan",
  "agent": "$AGENT_ID",
  "environment_id": "$ENVIRONMENT_ID",
  "initial_events": [
    {"type": "user.message", "content": [{"type": "text", "text": "Run the weekly compliance scan."}]}
  ],
  "schedule": {
    "type": "cron",
    "expression": "0 20 * * 5",
    "timezone": "America/New_York"
  }
}
EOF
```

```python
deployment = client.beta.deployments.create(
    name="Weekly compliance scan",
    agent=agent.id,
    environment_id=environment.id,
    initial_events=[
        {
            "type": "user.message",
            "content": [{"type": "text", "text": "Run the weekly compliance scan."}],
        },
    ],
    schedule={
        "type": "cron",
        "expression": "0 20 * * 5",
        "timezone": "America/New_York",
    },
)
```

The response is a deployment object (`depl_` ID prefix). Check `schedule.upcoming_runs_at` - the next fire times - to confirm the schedule parses the way you intended:

响应是一个部署对象（ID 前缀为 `depl_`）。检查 `schedule.upcoming_runs_at`——即接下来的触发时间——以确认计划按你的意图被解析：

```json
{
  "id": "depl_01xyz",
  "status": "active",
  "paused_reason": null,
  "schedule": {
    "type": "cron",
    "expression": "0 20 * * 5",
    "timezone": "America/New_York",
    "last_run_at": null,
    "upcoming_runs_at": ["2026-05-09T00:00:00Z", "2026-05-16T00:00:00Z", "2026-05-23T00:00:00Z"]
  }
}
```

`upcoming_runs_at` reflects the exact configured schedule, but **execution is jittered to distribute load: up to 15% of the interval between runs, floored at 5 seconds and capped at 9 minutes.** An hourly deployment can therefore fire up to 9 minutes late; don't build a downstream deadline that assumes the listed timestamp. Maximum **1000 scheduled deployments per organization** (contact Anthropic support for more).

`upcoming_runs_at` 反映的是精确配置的计划，但**实际执行会加入抖动（jitter）以分散负载：最多为两次运行间隔的 15%，下限 5 秒，上限 9 分钟。**因此每小时的部署最多可能晚 9 分钟触发；不要构建假设所列时间戳准点的下游截止时间。每个组织最多 **1000 个定时部署**（如需更多请联系 Anthropic 支持）。

【评论】抖动机制意味着 `upcoming_runs_at` 只是名义触发时间而非保证执行时间，这是分布式定时系统中常见的负载削峰设计，集成方需自行容忍延迟。

### Cron and timezone semantics / Cron 与时区语义

- **Expression:** standard POSIX cron (`minute hour day-of-month month day-of-week`).
  **表达式：**标准 POSIX cron（`分 时 日 月 星期`）。
- **Timezone:** IANA identifier (e.g. `"America/Los_Angeles"`).
  **时区：**IANA 标识符（例如 `"America/Los_Angeles"`）。
- **DST:** literal wall-clock matching - `"0 20 * * *"` in `America/New_York` fires at 8:00 PM local regardless of EST/EDT.
  **夏令时（DST）：**按字面墙钟时间匹配——`America/New_York` 中的 `"0 20 * * *"` 在当地时间 20:00 触发，无论 EST 还是 EDT。

> Warning: **DST edge:** wall-clock times that don't exist on a spring-forward day (e.g. 2AM) are **skipped**; times that occur twice on a fall-back day **fire twice**. Schedule outside the 1-3AM local window, or use UTC, when missed or duplicate executions are unacceptable.

> 警告：**夏令时边界：**在春季拨快那一天不存在的墙钟时间（例如凌晨 2 点）会被**跳过**；在秋季拨回那一天出现两次的时间会**触发两次**。当无法接受遗漏或重复执行时，请避开当地凌晨 1-3 点的时间段排程，或使用 UTC。

## Deployment budgets / 部署预算

A deployment accepts the same `budget` object as a session (`{type: "limit", max_list_cost: {amount, currency}}` - minor-unit cents string, `USD` only; see `shared/managed-agents-core.md` § Session budgets). The cap is **copied onto each session at fire time**, and that session then behaves exactly like any budgeted session.

部署接受与会话相同的 `budget` 对象（`{type: "limit", max_list_cost: {amount, currency}}`——最小单位为美分的字符串，仅支持 `USD`；见 `shared/managed-agents-core.md` 的"Session budgets"一节）。该上限会在**触发时复制到每个会话上**，之后该会话的行为与任何带预算的会话完全一致。

Deployment budget update semantics differ from a session's:

部署预算的更新语义与会话不同：

- `budget` is accepted on **create and update** - it is not create-only.
  `budget` 在**创建和更新时均可接受**——并非仅限创建时。
- `budget: null` on update **clears** it, and a cleared budget **can be re-added later** - there is no one-way door.
  更新时传 `budget: null` 会**清除**预算，且被清除的预算**之后可以重新添加**——不存在单向门。
- A change applies **from the next fired session** - sessions already running keep the cap they were created with (change those via their own session update).
  变更**从下一个被触发的会话起生效**——已在运行中的会话保留其创建时的上限（如需修改请通过这些会话自身的更新操作）。

## Deployment runs / 部署运行记录

Every trigger attempt - successful or not - writes a **deployment run** record (`drun_` prefix), so you can audit failures independent of the session lifecycle. A successful run carries the created `session_id`; follow that session via the event stream (`shared/managed-agents-events.md`) or webhooks (`shared/managed-agents-webhooks.md`) as usual. A failed run carries an `error` whose `type` explains why session creation was rejected.

每一次触发尝试——无论成功与否——都会写入一条**部署运行（deployment run）**记录（ID 前缀为 `drun_`），因此你可以独立于会话生命周期审计失败。成功的运行携带所创建的 `session_id`；可像往常一样通过事件流（`shared/managed-agents-events.md`）或 webhook（`shared/managed-agents-webhooks.md`）跟踪该会话。失败的运行携带一个 `error`，其 `type` 说明会话创建被拒绝的原因。

```python
# All runs for a deployment
for run in client.beta.deployment_runs.list(deployment_id=deployment.id):
    print(run.created_at, run.session_id or run.error.type)

# Failures only
for run in client.beta.deployment_runs.list(deployment_id=deployment.id, has_error=True):
    print(run.created_at, run.error.type, run.error.message)
```

```typescript
for await (const run of client.beta.deploymentRuns.list({
  deployment_id: deployment.id,
  has_error: true,
})) {
  console.log(run.created_at, run.error?.type, run.error?.message);
}
```

Raw HTTP: `GET /v1/deployment_runs?deployment_id=...&has_error=true`. To retrieve a single run by ID, `GET /v1/deployment_runs/{deployment_run_id}` (SDK: `client.beta.deployment_runs.retrieve(run_id)`) - a `deployment_run.*` webhook event carries the run ID as its `data.id`.

原始 HTTP：`GET /v1/deployment_runs?deployment_id=...&has_error=true`。要按 ID 获取单条运行记录，使用 `GET /v1/deployment_runs/{deployment_run_id}`（SDK：`client.beta.deployment_runs.retrieve(run_id)`）——`deployment_run.*` webhook 事件会把运行 ID 作为其 `data.id` 携带。

A failed run looks like:

一条失败的运行记录形如：

```json
{
  "type": "deployment_run",
  "id": "drun_01abc124",
  "deployment_id": "depl_01xyz",
  "trigger_context": { "type": "schedule", "scheduled_at": "2026-05-09T00:00:00Z" },
  "session_id": null,
  "error": { "type": "environment_archived", "message": "environment `env_01abc` is archived" },
  "agent": { "type": "agent", "id": "agent_01ghi789", "version": 3 },
  "created_at": "2026-05-09T00:00:01Z"
}
```

Error types include `environment_archived`, `agent_archived`, `vault_not_found`, `session_rate_limited`, and `service_unavailable`.

错误类型包括 `environment_archived`、`agent_archived`、`vault_not_found`、`session_rate_limited` 与 `service_unavailable`。

The outcome of each **scheduled** run (started/succeeded/failed) and each deployment lifecycle change (created/updated/paused/unpaused/archived/deleted) is also delivered as a webhook event - see `shared/managed-agents-webhooks.md` for the `deployment.*` and `deployment_run.*` event types - so you can react without polling. Manual runs do **not** emit `deployment_run.*` webhook events.

每次**定时**运行的结果（started/succeeded/failed）以及每次部署生命周期变更（created/updated/paused/unpaused/archived/deleted）也会以 webhook 事件的形式送达——`deployment.*` 与 `deployment_run.*` 事件类型见 `shared/managed-agents-webhooks.md`——因此无需轮询即可响应。手动运行**不会**发出 `deployment_run.*` webhook 事件。

## Lifecycle: pause / unpause / archive / 生命周期：暂停 / 恢复 / 归档

| Operation | SDK | Effect |
|---|---|---|
| Pause | `client.beta.deployments.pause(id)` | Suppresses scheduled triggers go-forward. Sessions already running continue. **Manual runs are still permitted while paused.** Sets `paused_reason: {"type": "manual"}`. |
| Unpause | `client.beta.deployments.unpause(id)` | Resumes from the next scheduled occurrence. **Missed triggers are not backfilled.** Clears `paused_reason`. |
| Archive | `client.beta.deployments.archive(id)` | **Terminal** - the schedule stops and the deployment can no longer be modified. Use pause for anything reversible. |

| 操作 | SDK | 效果 |
|---|---|---|
| 暂停 | `client.beta.deployments.pause(id)` | 从此抑制定时触发。已在运行的会话继续。**暂停期间仍允许手动运行。**会设置 `paused_reason: {"type": "manual"}`。 |
| 恢复 | `client.beta.deployments.unpause(id)` | 从下一个计划时点起恢复。**错过的触发不会被补跑。**会清除 `paused_reason`。 |
| 归档 | `client.beta.deployments.archive(id)` | **终态**——计划停止，部署不再可修改。任何可逆的操作都应使用暂停。 |

Raw HTTP: `POST /v1/deployments/{deployment_id}/pause` (likewise `/unpause`, `/archive`).

原始 HTTP：`POST /v1/deployments/{deployment_id}/pause`（`/unpause`、`/archive` 同理）。

### Failure behavior / 失败行为

- **Rate-limited:** recorded immediately as a `session_rate_limited` run, **no retry** - the schedule simply tries again at the next occurrence. (Rate limits on API calls *inside* a session are handled by the session itself.)
  **触发限流：**立即记录为一条 `session_rate_limited` 运行，**不重试**——计划只会在下一个时点再试。（会话*内部* API 调用的限流由会话自身处理。）
- **Other failed runs** (e.g. `environment_archived`, `vault_not_found`, `service_unavailable`): the run records the `error.type` - monitor runs and fix the referenced resource, or pause the deployment.
  **其他失败的运行**（例如 `environment_archived`、`vault_not_found`、`service_unavailable`）：运行记录会记下 `error.type`——请监控运行记录并修复所引用的资源，或暂停部署。
- **Agent archived:** the deployment is automatically **archived** (terminal) in the same operation. **Agent deleted:** the next scheduled trigger detects the missing agent and archives the deployment then. Either way no deployment run is recorded, and no further sessions are created.
  **智能体被归档：**部署会在同一操作中被自动**归档**（终态）。**智能体被删除：**下一个定时触发会检测到智能体缺失并届时归档部署。两种情况下都不会记录部署运行，也不会再创建会话。

## Manual runs / 手动运行

`POST /v1/deployments/{deployment_id}/run` (SDK: `client.beta.deployments.run(id)`) creates a session immediately and writes a run with `trigger_context.type: "manual"`. Use it to **test a deployment before committing to the schedule** - and remember it works even while the deployment is paused.

`POST /v1/deployments/{deployment_id}/run`（SDK：`client.beta.deployments.run(id)`）会立即创建一个会话，并写入一条 `trigger_context.type: "manual"` 的运行记录。用它来**在正式启用计划前测试部署**——并且注意即使部署处于暂停状态它也能运行。

【评论】"暂停期间手动运行仍然可用"与"错过的触发不补跑"这两条组合起来，实际上给出了一套低风险的上线流程：先暂停、手动验证、再恢复。
