<!-- BILINGUAL-EN-ZH -->

# Managed Agents - Endpoint Reference / Managed Agents - 端点参考

All endpoints require `x-api-key` and `anthropic-version: 2023-06-01` headers. Managed Agents endpoints additionally require the `anthropic-beta` header.

所有端点都需要 `x-api-key` 和 `anthropic-version: 2023-06-01` 请求头。Managed Agents 端点还额外要求 `anthropic-beta` 请求头。

> Most users should define agents and environments as version-controlled files synced with `ant apply` - see `shared/anthropic-cli.md`. The endpoints below are the underlying API that the CLI and SDKs drive.
> 大多数用户应当把代理和环境定义为受版本控制的文件，用 `ant apply` 同步——见 `shared/anthropic-cli.md`。下面的端点是 CLI 与 SDK 所驱动的底层 API。

## Beta Headers / Beta 请求头

```
anthropic-beta: managed-agents-2026-04-01
```

The SDK adds this header automatically for all `client.beta.{agents,environments,sessions,vaults,deployments,deployment_runs}.*` calls. Memory store endpoints (`client.beta.memory_stores.*`) use `agent-memory-2026-07-22` instead, which the SDK also sets; sending both headers on a memory store request returns a 400. The Files and Skills APIs are out of beta and need no beta header.

对于所有 `client.beta.{agents,environments,sessions,vaults,deployments,deployment_runs}.*` 调用，SDK 会自动添加这个请求头。记忆存储端点（`client.beta.memory_stores.*`）改用 `agent-memory-2026-07-22`，SDK 也会自动设置；对记忆存储请求同时发送两个请求头会返回 400。Files 与 Skills API 已结束测试期，不需要 beta 请求头。

---

## SDK Method Reference / SDK 方法参考

All resources are under the `beta` namespace. Python and TypeScript share identical method names.

所有资源都在 `beta` 命名空间下。Python 与 TypeScript 的方法名完全一致。

| Resource | Python / TypeScript (`client.beta.*`) | Go (`client.Beta.*`) |
| --- | --- | --- |
| Agents | `agents.create` / `retrieve` / `update` / `list` / `archive` | `Agents.New` / `Get` / `Update` / `List` / `Archive` |
| Agent Versions | `agents.versions.list` | `Agents.Versions.List` |
| Environments | `environments.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Environments.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| Environment Work (self-hosted) | `environments.work.poller` / `stats` / `stop` | See `shared/managed-agents-self-hosted-sandboxes.md` |
| Sessions | `sessions.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Sessions.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| Session Events | `sessions.events.list` / `send` / `stream` | `Sessions.Events.List` / `Send` / `StreamEvents` |
| Session Threads | `sessions.threads.list` / `retrieve` / `archive`; `sessions.threads.events.list` / `stream` | `Sessions.Threads.List` / `Get` / `Archive`; `Sessions.Threads.Events.List` / `StreamEvents` |
| Session Resources | `sessions.resources.add` / `retrieve` / `update` / `list` / `delete` | `Sessions.Resources.Add` / `Get` / `Update` / `List` / `Delete` |
| Deployments | `deployments.create` / `update` / `pause` / `unpause` / `archive` / `run` | Not yet documented - WebFetch the SDK repo (`shared/live-sources.md`) |
| Deployment Runs | `deployment_runs.list` / `retrieve` (TS: `deploymentRuns.*`) | Not yet documented - WebFetch the SDK repo (`shared/live-sources.md`) |
| Vaults | `vaults.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Vaults.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| Credentials | `vaults.credentials.create` / `retrieve` / `update` / `list` / `delete` / `archive` / `mcp_oauth_validate` | `Vaults.Credentials.New` / `Get` / `Update` / `List` / `Delete` / `Archive` / `McpOauthValidate` |
| Memory Stores | `memory_stores.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `MemoryStores.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| Memories | `memory_stores.memories.create` / `retrieve` / `update` / `list` / `delete` | `MemoryStores.Memories.New` / `Get` / `Update` / `List` / `Delete` |
| Memory Versions | `memory_stores.memory_versions.list` / `retrieve` / `redact` | `MemoryStores.MemoryVersions.List` / `Get` / `Redact` |

| 资源 | Python / TypeScript (`client.beta.*`) | Go (`client.Beta.*`) |
| --- | --- | --- |
| 代理 | `agents.create` / `retrieve` / `update` / `list` / `archive` | `Agents.New` / `Get` / `Update` / `List` / `Archive` |
| 代理版本 | `agents.versions.list` | `Agents.Versions.List` |
| 环境 | `environments.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Environments.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 环境工作（自托管） | `environments.work.poller` / `stats` / `stop` | 见 `shared/managed-agents-self-hosted-sandboxes.md` |
| 会话 | `sessions.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Sessions.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 会话事件 | `sessions.events.list` / `send` / `stream` | `Sessions.Events.List` / `Send` / `StreamEvents` |
| 会话线程 | `sessions.threads.list` / `retrieve` / `archive`; `sessions.threads.events.list` / `stream` | `Sessions.Threads.List` / `Get` / `Archive`; `Sessions.Threads.Events.List` / `StreamEvents` |
| 会话资源 | `sessions.resources.add` / `retrieve` / `update` / `list` / `delete` | `Sessions.Resources.Add` / `Get` / `Update` / `List` / `Delete` |
| 部署 | `deployments.create` / `update` / `pause` / `unpause` / `archive` / `run` | 暂无文档 - 请 WebFetch SDK 仓库（`shared/live-sources.md`） |
| 部署运行 | `deployment_runs.list` / `retrieve`（TS：`deploymentRuns.*`） | 暂无文档 - 请 WebFetch SDK 仓库（`shared/live-sources.md`） |
| 保管库 | `vaults.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Vaults.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 凭证 | `vaults.credentials.create` / `retrieve` / `update` / `list` / `delete` / `archive` / `mcp_oauth_validate` | `Vaults.Credentials.New` / `Get` / `Update` / `List` / `Delete` / `Archive` / `McpOauthValidate` |
| 记忆存储 | `memory_stores.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `MemoryStores.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 记忆 | `memory_stores.memories.create` / `retrieve` / `update` / `list` / `delete` | `MemoryStores.Memories.New` / `Get` / `Update` / `List` / `Delete` |
| 记忆版本 | `memory_stores.memory_versions.list` / `retrieve` / `redact` | `MemoryStores.MemoryVersions.List` / `Get` / `Redact` |

**Naming quirks to watch for:**

**需要注意的命名怪癖：**
- Agents and Session Threads have **no delete** - only `archive`. Archive is **permanent**: the agent becomes read-only, new sessions cannot reference it, and there is no unarchive. Confirm with the user before archiving a production agent. Environments, Sessions, Vaults, Credentials, and Memory Stores have both `delete` and `archive`; Session Resources, Files, Skills, and Memories are `delete`-only; Memory Versions have neither - only `redact`.
  代理与会话线程**没有 delete**——只有 `archive`。归档是**永久的**：代理变为只读，新会话不能引用它，也没有取消归档。归档生产环境的代理前请先与用户确认。环境、会话、保管库、凭证和记忆存储同时有 `delete` 和 `archive`；会话资源、文件、技能和记忆只有 `delete`；记忆版本两者皆无——只有 `redact`。
- Session resources use `add` (not `create`).
  会话资源用 `add`（不是 `create`）。
- Go's event stream is `StreamEvents` (not `Stream`).
  Go 的事件流是 `StreamEvents`（不是 `Stream`）。
- The self-hosted worker class is `EnvironmentWorker` from `anthropic.lib.environments` / `@anthropic-ai/sdk/helpers/beta/environments` / `anthropic-sdk-go/lib/environments`; `client.beta.environments.work.worker(...)` is a factory that returns the same class, alongside the `environments.work.poller/stats/stop` client methods.
  自托管的 worker 类是来自 `anthropic.lib.environments` / `@anthropic-ai/sdk/helpers/beta/environments` / `anthropic-sdk-go/lib/environments` 的 `EnvironmentWorker`；`client.beta.environments.work.worker(...)` 是一个返回同一类的工厂，与 `environments.work.poller/stats/stop` 客户端方法并列。

【评论】"归档即终态、不可取消归档"是不可逆操作设计；文档据此要求在归档生产代理前先征得用户确认，属于对破坏性 API 的防护性约定。

**Agent shorthand:** `agent` on session create accepts three forms - a bare string (`agent="agent_abc123"`, latest version), a pinned reference `{type: "agent", id, version}`, or `{type: "agent_with_overrides", id, version?, model?, system?, tools?, mcp_servers?, skills?}` to override those fields for this session only (see `shared/managed-agents-core.md` -> Override agent configuration for a session).

**代理简写：** 会话创建时的 `agent` 接受三种形式——裸字符串（`agent="agent_abc123"`，最新版本）、钉住版本的引用 `{type: "agent", id, version}`，或 `{type: "agent_with_overrides", id, version?, model?, system?, tools?, mcp_servers?, skills?}`，仅为该会话覆盖那些字段（见 `shared/managed-agents-core.md` -> Override agent configuration for a session）。

**Model shorthand:** `model` on agent create accepts either a bare string (`model="claude-opus-5-5"` - uses `standard` speed) or the full config object, which takes `speed`, `effort`, and `inference_geo` alongside `id`: `{id: "claude-opus-5-5", speed: "fast"}`, `{id: "claude-opus-5-5", effort: "high"}`, `{id: "claude-opus-5-5", inference_geo: "us"}`. `effort` accepts a level string (`low`/`medium`/`high`/`xhigh`/`max`) or `{type: "<level>"}`, and in a per-session `model` override it sets the session's effort level (the agent's own `effort` isn't carried over, and a `model` override without `effort` runs at that model's default effort level). `inference_geo` (`"us"` | `"global"`) pins the geography serving the agent's model requests, and is also applied in a per-session `model` override. See `shared/managed-agents-core.md` -> Effort on the agent model / Pinning inference geography. Note: `speed: "fast"` is supported on Claude Opus 5.5, Claude Opus 5, and Opus 4.8 - on the Claude API only, which includes Managed Agents but not Amazon Bedrock, Google Cloud, or Microsoft Foundry. Opus 4.7 fast mode has been removed; `speed: "fast"` on Opus 4.7 returns an error.

**模型简写：** 代理创建时的 `model` 接受裸字符串（`model="claude-opus-5-5"`——使用 `standard` 速度）或完整配置对象，后者在 `id` 之外还接受 `speed`、`effort` 和 `inference_geo`：`{id: "claude-opus-5-5", speed: "fast"}`、`{id: "claude-opus-5-5", effort: "high"}`、`{id: "claude-opus-5-5", inference_geo: "us"}`。`effort` 接受级别字符串（`low`/`medium`/`high`/`xhigh`/`max`）或 `{type: "<level>"}`；在按会话的 `model` 覆盖中，它会设置该会话的 effort 级别（代理自身的 `effort` 不会被带过来，不带 `effort` 的 `model` 覆盖按该模型的默认 effort 级别运行）。`inference_geo`（`"us"` | `"global"`）钉住服务该代理模型请求的地理区域，在按会话的 `model` 覆盖中同样生效。见 `shared/managed-agents-core.md` -> Effort on the agent model / Pinning inference geography。注意：`speed: "fast"` 在 Claude Opus 5.5、Claude Opus 5 和 Opus 4.8 上受支持——仅在 Claude API 上，包括 Managed Agents，但不包括 Amazon Bedrock、Google Cloud 或 Microsoft Foundry。Opus 4.7 的 fast 模式已被移除；在 Opus 4.7 上使用 `speed: "fast"` 会返回错误。

---

## Agents / 代理

**Step one of every flow.** Sessions require a pre-created agent - there is no inline agent config under `managed-agents-2026-04-01`.

**每个流程的第一步。** 会话需要预先创建好的代理——在 `managed-agents-2026-04-01` 下没有内联的代理配置。

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/agents` | ListAgents | List agents |
| `POST` | `/v1/agents` | CreateAgent | Create a saved agent configuration |
| `GET` | `/v1/agents/{agent_id}` | GetAgent | Get agent details |
| `POST` | `/v1/agents/{agent_id}` | UpdateAgent | Update agent configuration. `version` is **optional**: supply it (>= 1) for optimistic concurrency - a mismatch returns 409 - or omit it for an unconditional last-write-wins update. |
| `POST` | `/v1/agents/{agent_id}/archive` | ArchiveAgent | Archive an agent. Makes it **read-only**; existing sessions continue, new sessions cannot reference it. No unarchive - this is the terminal state. |
| `GET` | `/v1/agents/{agent_id}/versions` | ListAgentVersions | List agent versions |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/agents` | ListAgents | 列出代理 |
| `POST` | `/v1/agents` | CreateAgent | 创建一个已保存的代理配置 |
| `GET` | `/v1/agents/{agent_id}` | GetAgent | 获取代理详情 |
| `POST` | `/v1/agents/{agent_id}` | UpdateAgent | 更新代理配置。`version` 是**可选的**：提供它（>= 1）可启用乐观并发——不匹配返回 409——或省略它做无条件的最后写入胜出更新。 |
| `POST` | `/v1/agents/{agent_id}/archive` | ArchiveAgent | 归档代理。使其变为**只读**；现有会话继续，新会话不能引用它。没有取消归档——这是终态。 |
| `GET` | `/v1/agents/{agent_id}/versions` | ListAgentVersions | 列出代理版本 |

## Sessions / 会话

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions` | ListSessions | List sessions (paginated) |
| `POST` | `/v1/sessions` | CreateSession | Create a new session |
| `GET` | `/v1/sessions/{session_id}` | GetSession | Get session details |
| `POST` | `/v1/sessions/{session_id}` | UpdateSession | Update session `metadata`/`title`, `agent.tools`/`agent.mcp_servers` (session-local override; session must be `idle`), or `budget` - change the cap (higher or lower; the new value must exceed the consumed list cost) or remove it with `null`; removal is one-way, and a budget can never be added post-create. `vault_ids` is create-only (rejected on update). See `shared/managed-agents-core.md` -> Updating the agent configuration mid-session / Session budgets. |
| `DELETE` | `/v1/sessions/{session_id}` | DeleteSession | Delete a session |
| `POST` | `/v1/sessions/{session_id}/archive` | ArchiveSession | Archive a session |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions` | ListSessions | 列出会话（分页） |
| `POST` | `/v1/sessions` | CreateSession | 创建新会话 |
| `GET` | `/v1/sessions/{session_id}` | GetSession | 获取会话详情 |
| `POST` | `/v1/sessions/{session_id}` | UpdateSession | 更新会话的 `metadata`/`title`、`agent.tools`/`agent.mcp_servers`（会话本地覆盖；会话必须处于 `idle`），或 `budget`——更改上限（调高或调低；新值必须超过已消耗的清单成本）或用 `null` 移除；移除是单向的，预算在创建后绝不能再添加。`vault_ids` 仅创建时可用（更新时被拒绝）。见 `shared/managed-agents-core.md` -> Updating the agent configuration mid-session / Session budgets。 |
| `DELETE` | `/v1/sessions/{session_id}` | DeleteSession | 删除会话 |
| `POST` | `/v1/sessions/{session_id}/archive` | ArchiveSession | 归档会话 |

## Events / 事件

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/events` | ListEvents | List events (polling, paginated) |
| `POST` | `/v1/sessions/{session_id}/events` | SendEvents | Send events (user message, tool result) |
| `GET` | `/v1/sessions/{session_id}/events/stream` | StreamEvents | Stream events via SSE. Optional `event_deltas[]=agent.message` / `agent.thinking` opts in to live-preview `event_start`/`event_delta` events - see `shared/managed-agents-events.md` § Live previews. |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/events` | ListEvents | 列出事件（轮询、分页） |
| `POST` | `/v1/sessions/{session_id}/events` | SendEvents | 发送事件（用户消息、工具结果） |
| `GET` | `/v1/sessions/{session_id}/events/stream` | StreamEvents | 通过 SSE 流式获取事件。可选的 `event_deltas[]=agent.message` / `agent.thinking` 可选择加入实时预览的 `event_start`/`event_delta` 事件——见 `shared/managed-agents-events.md` § Live previews。 |

## Session Threads / 会话线程

Per-subagent event streams in multiagent sessions. See `shared/managed-agents-multiagent.md`.

多代理会话中每个子代理的事件流。见 `shared/managed-agents-multiagent.md`。

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/threads` | ListThreads | List threads (paginated) |
| `GET` | `/v1/sessions/{session_id}/threads/{thread_id}` | GetThread | Retrieve one thread (carries `agent` snapshot, `status`, `parent_thread_id`, `stats`, `usage`) |
| `POST` | `/v1/sessions/{session_id}/threads/{thread_id}/archive` | ArchiveThread | Archive a thread |
| `GET` | `/v1/sessions/{session_id}/threads/{thread_id}/events` | ListThreadEvents | List past events for one thread (paginated) |
| `GET` | `/v1/sessions/{session_id}/threads/{thread_id}/stream` | StreamThreadEvents | Stream one thread via SSE (SDK: `threads.events.stream`) |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/threads` | ListThreads | 列出线程（分页） |
| `GET` | `/v1/sessions/{session_id}/threads/{thread_id}` | GetThread | 获取单个线程（携带 `agent` 快照、`status`、`parent_thread_id`、`stats`、`usage`） |
| `POST` | `/v1/sessions/{session_id}/threads/{thread_id}/archive` | ArchiveThread | 归档一个线程 |
| `GET` | `/v1/sessions/{session_id}/threads/{thread_id}/events` | ListThreadEvents | 列出单个线程的历史事件（分页） |
| `GET` | `/v1/sessions/{session_id}/threads/{thread_id}/stream` | StreamThreadEvents | 通过 SSE 流式获取单个线程（SDK：`threads.events.stream`） |

## Session Resources / 会话资源

| Method   | Path                                                    | Operation        | Description                              |
| -------- | ------------------------------------------------------- | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/resources` | ListResources | List resources attached to session |
| `POST` | `/v1/sessions/{session_id}/resources` | AddResource | Attach `file` or `github_repository` resource (SDK method: `add`, not `create`). `memory_store` resources attach at session-create time only. Self-hosted environments accept **only** `memory_store` (at create); `file` / `github_repository` are rejected there. |
| `GET` | `/v1/sessions/{session_id}/resources/{resource_id}` | GetResource | Get a single resource |
| `POST` | `/v1/sessions/{session_id}/resources/{resource_id}` | UpdateResource | Update resource |
| `DELETE` | `/v1/sessions/{session_id}/resources/{resource_id}` | DeleteResource | Remove resource from session |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------------- | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/resources` | ListResources | 列出附加到会话的资源 |
| `POST` | `/v1/sessions/{session_id}/resources` | AddResource | 附加 `file` 或 `github_repository` 资源（SDK 方法：`add`，不是 `create`）。`memory_store` 资源只能在会话创建时附加。自托管环境**只**接受 `memory_store`（在创建时）；`file` / `github_repository` 在那里会被拒绝。 |
| `GET` | `/v1/sessions/{session_id}/resources/{resource_id}` | GetResource | 获取单个资源 |
| `POST` | `/v1/sessions/{session_id}/resources/{resource_id}` | UpdateResource | 更新资源 |
| `DELETE` | `/v1/sessions/{session_id}/resources/{resource_id}` | DeleteResource | 从会话移除资源 |

## Environments / 环境

| Method   | Path                                                             | Operation            | Description                         |
| -------- | ---------------------------------------------------------------- | -------------------- | ----------------------------------- |
| `POST`   | `/v1/environments`                                     | CreateEnvironment    | Create environment                  |
| `GET`    | `/v1/environments`                                     | ListEnvironments     | List environments                   |
| `GET`    | `/v1/environments/{environment_id}`                    | GetEnvironment       | Get environment details             |
| `POST`   | `/v1/environments/{environment_id}`                    | UpdateEnvironment    | Update environment                  |
| `DELETE` | `/v1/environments/{environment_id}`                    | DeleteEnvironment    | Delete environment. Returns 204. |
| `POST`   | `/v1/environments/{environment_id}/archive`            | ArchiveEnvironment   | Archive environment. Makes it **read-only**; existing sessions continue, new sessions cannot reference it. No unarchive - this is the terminal state. |
| `GET`    | `/v1/environments/{environment_id}/work/stats`         | WorkQueueStats       | Self-hosted work-queue depth/pending/workers. `x-api-key` auth. See `shared/managed-agents-self-hosted-sandboxes.md`. |
| `POST`   | `/v1/environments/{environment_id}/work/{work_id}/stop` | StopWork            | Self-hosted: stop a claimed work item. `x-api-key` auth. |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ---------------------------------------------------------------- | -------------------- | ----------------------------------- |
| `POST`   | `/v1/environments`                                     | CreateEnvironment    | 创建环境                  |
| `GET`    | `/v1/environments`                                     | ListEnvironments     | 列出环境                   |
| `GET`    | `/v1/environments/{environment_id}`                    | GetEnvironment       | 获取环境详情             |
| `POST`   | `/v1/environments/{environment_id}`                    | UpdateEnvironment    | 更新环境                  |
| `DELETE` | `/v1/environments/{environment_id}`                    | DeleteEnvironment    | 删除环境。返回 204。 |
| `POST`   | `/v1/environments/{environment_id}/archive`            | ArchiveEnvironment   | 归档环境。使其变为**只读**；现有会话继续，新会话不能引用它。没有取消归档——这是终态。 |
| `GET`    | `/v1/environments/{environment_id}/work/stats`         | WorkQueueStats       | 自托管工作队列的深度/待处理/worker 数。`x-api-key` 认证。见 `shared/managed-agents-self-hosted-sandboxes.md`。 |
| `POST`   | `/v1/environments/{environment_id}/work/{work_id}/stop` | StopWork            | 自托管：停止一个已被认领的工作项。`x-api-key` 认证。 |

For `type: "self_hosted"`, `config` is the bare `{"type": "self_hosted"}` - `networking` and `packages` do not apply. (`networking` never governs `web_search` / `web_fetch` in either type - those are restricted per-tool with `allowed_domains` / `blocked_domains` in the agent toolset; see `shared/managed-agents-tools.md`.)

对于 `type: "self_hosted"`，`config` 就是裸的 `{"type": "self_hosted"}`——`networking` 和 `packages` 不适用。（两种类型中 `networking` 都不管辖 `web_search` / `web_fetch`——那些是在代理工具集中按工具用 `allowed_domains` / `blocked_domains` 限制的；见 `shared/managed-agents-tools.md`。）

## Deployments / 部署

Scheduled deployments (`depl_` IDs) run an agent on a recurring cron schedule - each firing creates a session. See `shared/managed-agents-scheduled-deployments.md` for the conceptual guide (cron/DST semantics, failure behavior, lifecycle).

定时部署（`depl_` ID）按周期性的 cron 计划运行一个代理——每次触发都会创建一个会话。概念指南（cron/夏令时语义、失败行为、生命周期）见 `shared/managed-agents-scheduled-deployments.md`。

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/deployments`                                | CreateDeployment | Create a scheduled deployment            |
| `POST`   | `/v1/deployments/{deployment_id}`                | UpdateDeployment | Update deployment configuration (see `shared/managed-agents-scheduled-deployments.md`) |
| `POST`   | `/v1/deployments/{deployment_id}/pause`          | PauseDeployment  | Suppress scheduled triggers (reversible; manual runs still allowed) |
| `POST`   | `/v1/deployments/{deployment_id}/unpause`        | UnpauseDeployment | Resume from the next occurrence (no backfill) |
| `POST`   | `/v1/deployments/{deployment_id}/archive`        | ArchiveDeployment | **Terminal** - schedule stops, deployment becomes immutable |
| `POST`   | `/v1/deployments/{deployment_id}/run`            | RunDeployment    | Trigger a manual run immediately (`trigger_context.type: "manual"`); works while paused |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/deployments`                                | CreateDeployment | 创建定时部署            |
| `POST`   | `/v1/deployments/{deployment_id}`                | UpdateDeployment | 更新部署配置（见 `shared/managed-agents-scheduled-deployments.md`） |
| `POST`   | `/v1/deployments/{deployment_id}/pause`          | PauseDeployment  | 抑制定时触发（可逆；仍允许手动运行） |
| `POST`   | `/v1/deployments/{deployment_id}/unpause`        | UnpauseDeployment | 从下一次触发起恢复（不回填） |
| `POST`   | `/v1/deployments/{deployment_id}/archive`        | ArchiveDeployment | **终态**——计划停止，部署变为不可变 |
| `POST`   | `/v1/deployments/{deployment_id}/run`            | RunDeployment    | 立即触发一次手动运行（`trigger_context.type: "manual"`）；暂停期间也可用 |

## Deployment Runs / 部署运行

Each trigger attempt (scheduled or manual) writes a `deployment_run` record (`drun_` IDs) carrying either the created `session_id` or an `error.type` (`environment_archived`, `agent_archived`, `vault_not_found`, `session_rate_limited`, `service_unavailable`).

每次触发尝试（定时或手动）都会写入一条 `deployment_run` 记录（`drun_` ID），携带已创建的 `session_id` 或一个 `error.type`（`environment_archived`、`agent_archived`、`vault_not_found`、`session_rate_limited`、`service_unavailable`）。

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET`    | `/v1/deployment_runs?deployment_id=...`          | ListDeploymentRuns | List runs for a deployment (paginated; filter failures with `has_error=true`) |
| `GET`    | `/v1/deployment_runs/{deployment_run_id}`        | GetDeploymentRun   | Retrieve a single run by ID (a `deployment_run.*` webhook event carries this as `data.id`) |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET`    | `/v1/deployment_runs?deployment_id=...`          | ListDeploymentRuns | 列出某个部署的运行（分页；用 `has_error=true` 过滤失败） |
| `GET`    | `/v1/deployment_runs/{deployment_run_id}`        | GetDeploymentRun   | 按 ID 获取单次运行（`deployment_run.*` webhook 事件将其作为 `data.id` 携带） |

## Vaults / 保管库

Vaults store credentials that Anthropic manages on your behalf - MCP credentials (OAuth with auto-refresh, or static bearer tokens) and `environment_variable` credentials substituted into outbound requests at egress. Attach to sessions via `vault_ids`. See `managed-agents-tools.md` §Vaults for the conceptual guide and credential shapes.

保管库存储由 Anthropic 代你管理的凭证——MCP 凭证（自动刷新的 OAuth，或静态 bearer token）和在出口处替换进外发请求的 `environment_variable` 凭证。通过 `vault_ids` 附加到会话。概念指南与凭证形状见 `managed-agents-tools.md` §Vaults。

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/vaults`                                     | CreateVault      | Create a vault                           |
| `GET`    | `/v1/vaults`                                     | ListVaults       | List vaults                              |
| `GET`    | `/v1/vaults/{vault_id}`                          | GetVault         | Get vault details                        |
| `POST`   | `/v1/vaults/{vault_id}`                          | UpdateVault      | Update vault                             |
| `DELETE` | `/v1/vaults/{vault_id}`                          | DeleteVault      | Delete vault                             |
| `POST`   | `/v1/vaults/{vault_id}/archive`                  | ArchiveVault     | Archive vault                            |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/vaults`                                     | CreateVault      | 创建保管库                           |
| `GET`    | `/v1/vaults`                                     | ListVaults       | 列出保管库                              |
| `GET`    | `/v1/vaults/{vault_id}`                          | GetVault         | 获取保管库详情                        |
| `POST`   | `/v1/vaults/{vault_id}`                          | UpdateVault      | 更新保管库                             |
| `DELETE` | `/v1/vaults/{vault_id}`                          | DeleteVault      | 删除保管库                             |
| `POST`   | `/v1/vaults/{vault_id}/archive`                  | ArchiveVault     | 归档保管库                            |

## Credentials / 凭证

Credentials are individual secrets stored inside a vault.

凭证是存储在保管库内的单个机密。

| Method   | Path                                                              | Operation          | Description                  |
| -------- | ----------------------------------------------------------------- | ------------------ | ---------------------------- |
| `POST`   | `/v1/vaults/{vault_id}/credentials`                               | CreateCredential   | Create a credential          |
| `GET`    | `/v1/vaults/{vault_id}/credentials`                               | ListCredentials    | List credentials in vault    |
| `GET`    | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | GetCredential      | Get credential metadata      |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | UpdateCredential   | Update credential            |
| `DELETE` | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | DeleteCredential   | Delete credential            |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}/archive`       | ArchiveCredential  | Archive credential           |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}/mcp_oauth_validate` | McpOauthValidate | Validate an MCP OAuth credential |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ----------------------------------------------------------------- | ------------------ | ---------------------------- |
| `POST`   | `/v1/vaults/{vault_id}/credentials`                               | CreateCredential   | 创建凭证          |
| `GET`    | `/v1/vaults/{vault_id}/credentials`                               | ListCredentials    | 列出保管库中的凭证    |
| `GET`    | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | GetCredential      | 获取凭证元数据      |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | UpdateCredential   | 更新凭证            |
| `DELETE` | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | DeleteCredential   | 删除凭证            |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}/archive`       | ArchiveCredential  | 归档凭证           |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}/mcp_oauth_validate` | McpOauthValidate | 验证一个 MCP OAuth 凭证 |

## Memory Stores / 记忆存储

Workspace-scoped persistent memory that survives across sessions. Attach to a session via a `{"type": "memory_store", "memory_store_id": ...}` entry in `resources[]` (session-create time only). See `shared/managed-agents-memory.md` for the conceptual guide, the FUSE-mount agent interface, preconditions, and versioning.

工作区范围的持久记忆，跨会话存活。通过 `resources[]` 中的 `{"type": "memory_store", "memory_store_id": ...}` 条目附加到会话（仅会话创建时）。概念指南、FUSE 挂载的代理接口、前置条件与版本化见 `shared/managed-agents-memory.md`。

| Method   | Path                                             | Operation          | Description                              |
| -------- | ------------------------------------------------ | ------------------ | ---------------------------------------- |
| `POST`   | `/v1/memory_stores`                              | CreateMemoryStore  | Create a store (`name`, `description`, `metadata`) |
| `GET`    | `/v1/memory_stores`                              | ListMemoryStores   | List stores (`include_archived`, `created_at_{gte,lte}`) |
| `GET`    | `/v1/memory_stores/{memory_store_id}`            | GetMemoryStore     | Get store details                        |
| `POST`   | `/v1/memory_stores/{memory_store_id}`            | UpdateMemoryStore  | Update store                             |
| `DELETE` | `/v1/memory_stores/{memory_store_id}`            | DeleteMemoryStore  | Delete store                             |
| `POST`   | `/v1/memory_stores/{memory_store_id}/archive`    | ArchiveMemoryStore | Archive store. Makes it **read-only**; existing sessions continue, new sessions cannot reference it. No unarchive. |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ------------------ | ---------------------------------------- |
| `POST`   | `/v1/memory_stores`                              | CreateMemoryStore  | 创建存储（`name`、`description`、`metadata`） |
| `GET`    | `/v1/memory_stores`                              | ListMemoryStores   | 列出存储（`include_archived`、`created_at_{gte,lte}`） |
| `GET`    | `/v1/memory_stores/{memory_store_id}`            | GetMemoryStore     | 获取存储详情                        |
| `POST`   | `/v1/memory_stores/{memory_store_id}`            | UpdateMemoryStore  | 更新存储                             |
| `DELETE` | `/v1/memory_stores/{memory_store_id}`            | DeleteMemoryStore  | 删除存储                             |
| `POST`   | `/v1/memory_stores/{memory_store_id}/archive`    | ArchiveMemoryStore | 归档存储。使其变为**只读**；现有会话继续，新会话不能引用它。没有取消归档。 |

## Memories / 记忆

Individual text documents inside a store (<= 100KB each). `create` creates at a `path` and returns `409` (`memory_path_conflict_error`, with `conflicting_memory_id`) if the path is occupied; `update` mutates by `mem_...` ID (rename and/or content). Only `update` accepts a `precondition` (`{"type": "content_sha256", "content_sha256": ...}`) - on mismatch returns `409` (`memory_precondition_failed_error`). List endpoints accept `view: "basic"|"full"` (controls whether `content` is populated; `retrieve` defaults to `full`).

存储内的单个文本文档（每个 <= 100KB）。`create` 在一个 `path` 处创建，若路径已被占用则返回 `409`（`memory_path_conflict_error`，附 `conflicting_memory_id`）；`update` 按 `mem_...` ID 变更（重命名和/或内容）。只有 `update` 接受 `precondition`（`{"type": "content_sha256", "content_sha256": ...}`）——不匹配时返回 `409`（`memory_precondition_failed_error`）。列表端点接受 `view: "basic"|"full"`（控制是否填充 `content`；`retrieve` 默认为 `full`）。

| Method   | Path                                                              | Operation      | Description                              |
| -------- | ----------------------------------------------------------------- | -------------- | ---------------------------------------- |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memories`                    | ListMemories   | Returns `Memory \| MemoryPrefix`; filter by `path_prefix`, `depth` |
| `POST`   | `/v1/memory_stores/{memory_store_id}/memories`                    | CreateMemory   | Create at `path` (SDK: `memories.create`); `409 memory_path_conflict_error` if occupied |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memories/{memory_id}`        | GetMemory      | Read one memory (defaults to `view="full"`) |
| `PATCH`  | `/v1/memory_stores/{memory_store_id}/memories/{memory_id}`        | UpdateMemory   | Change `content`, `path`, or both by ID; optional `precondition` |
| `DELETE` | `/v1/memory_stores/{memory_store_id}/memories/{memory_id}`        | DeleteMemory   | Delete (optional `expected_content_sha256`) |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ----------------------------------------------------------------- | -------------- | ---------------------------------------- |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memories`                    | ListMemories   | 返回 `Memory \| MemoryPrefix`；按 `path_prefix`、`depth` 过滤 |
| `POST`   | `/v1/memory_stores/{memory_store_id}/memories`                    | CreateMemory   | 在 `path` 处创建（SDK：`memories.create`）；若被占用则 `409 memory_path_conflict_error` |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memories/{memory_id}`        | GetMemory      | 读取一条记忆（默认 `view="full"`） |
| `PATCH`  | `/v1/memory_stores/{memory_store_id}/memories/{memory_id}`        | UpdateMemory   | 按 ID 更改 `content`、`path` 或两者；可选 `precondition` |
| `DELETE` | `/v1/memory_stores/{memory_store_id}/memories/{memory_id}`        | DeleteMemory   | 删除（可选 `expected_content_sha256`） |

## Memory Versions / 记忆版本

Immutable per-mutation snapshots (`memver_...`) - the audit and rollback surface. `operation` in `created` / `modified` / `deleted`.

每次变更一份的不可变快照（`memver_...`）——审计与回滚的界面。`operation` 取值为 `created` / `modified` / `deleted`。

| Method   | Path                                                                          | Operation             | Description                              |
| -------- | ----------------------------------------------------------------------------- | --------------------- | ---------------------------------------- |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memory_versions`                         | ListMemoryVersions    | Newest-first; filter by `memory_id`, `operation`, `session_id`, `api_key_id`, `created_at_{gte,lte}` |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memory_versions/{version_id}`            | GetMemoryVersion      | List fields + full `content`             |
| `POST`   | `/v1/memory_stores/{memory_store_id}/memory_versions/{version_id}/redact`     | RedactMemoryVersion   | Clear `content`/`content_sha256`/`content_size_bytes`/`path`; preserve actor + timestamps |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ----------------------------------------------------------------------------- | --------------------- | ---------------------------------------- |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memory_versions`                         | ListMemoryVersions    | 最新优先；按 `memory_id`、`operation`、`session_id`、`api_key_id`、`created_at_{gte,lte}` 过滤 |
| `GET`    | `/v1/memory_stores/{memory_store_id}/memory_versions/{version_id}`            | GetMemoryVersion      | 列出字段 + 完整 `content`             |
| `POST`   | `/v1/memory_stores/{memory_store_id}/memory_versions/{version_id}/redact`     | RedactMemoryVersion   | 清除 `content`/`content_sha256`/`content_size_bytes`/`path`；保留操作者与时间戳 |

## Files / 文件

| Method   | Path                                             | Operation        | Description                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/files`                            | UploadFile       | Upload a file                            |
| `GET`    | `/v1/files`                            | ListFiles        | List files                               |
| `GET`    | `/v1/files/{file_id}`                  | GetFile          | Get file metadata (SDK method: `retrieve_metadata`) |
| `GET`    | `/v1/files/{file_id}/content`          | DownloadFile     | Download file content                    |
| `DELETE` | `/v1/files/{file_id}`                  | DeleteFile       | Delete a file                            |

| 方法 | 路径 | 操作 | 说明 |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/files`                            | UploadFile       | 上传一个文件                            |
| `GET`    | `/v1/files`                            | ListFiles        | 列出文件                               |
| `GET`    | `/v1/files/{file_id}`                  | GetFile          | 获取文件元数据（SDK 方法：`retrieve_metadata`） |
| `GET`    | `/v1/files/{file_id}/content`          | DownloadFile     | 下载文件内容                    |
| `DELETE` | `/v1/files/{file_id}`                  | DeleteFile       | 删除一个文件                            |

## Skills / 技能

| Method   | Path                                                            | Operation          | Description                  |
| -------- | --------------------------------------------------------------- | ------------------ | ---------------------------- |
| `POST`   | `/v1/skills`                                          | CreateSkill        | Create a skill               |
| `GET`    | `/v1/skills`                                          | ListSkills         | List skills                  |
| `GET`    | `/v1/skills/{skill_id}`                               | GetSkill           | Get skill details            |
| `DELETE` | `/v1/skills/{skill_id}`                               | DeleteSkill        | Delete a skill               |
| `POST`   | `/v1/skills/{skill_id}/versions`                      | CreateVersion      | Create skill version         |
| `GET`    | `/v1/skills/{skill_id}/versions`                      | ListVersions       | List skill versions          |
| `GET`    | `/v1/skills/{skill_id}/versions/{version}`            | GetVersion         | Get skill version            |
| `DELETE` | `/v1/skills/{skill_id}/versions/{version}`            | DeleteVersion      | Delete skill version         |

| 方法 | 路径 | 操作 | 说明 |
| -------- | --------------------------------------------------------------- | ------------------ | ---------------------------- |
| `POST`   | `/v1/skills`                                          | CreateSkill        | 创建一个技能               |
| `GET`    | `/v1/skills`                                          | ListSkills         | 列出技能                  |
| `GET`    | `/v1/skills/{skill_id}`                               | GetSkill           | 获取技能详情            |
| `DELETE` | `/v1/skills/{skill_id}`                               | DeleteSkill        | 删除一个技能               |
| `POST`   | `/v1/skills/{skill_id}/versions`                      | CreateVersion      | 创建技能版本         |
| `GET`    | `/v1/skills/{skill_id}/versions`                      | ListVersions       | 列出技能版本          |
| `GET`    | `/v1/skills/{skill_id}/versions/{version}`            | GetVersion         | 获取技能版本            |
| `DELETE` | `/v1/skills/{skill_id}/versions/{version}`            | DeleteVersion      | 删除技能版本         |

---

## Request/Response Schema Quick Reference / 请求/响应模式速查

### CreateAgent Request Body / CreateAgent 请求体

**Always start here.** `model`, `system`, `tools`, `mcp_servers`, `skills` are top-level fields on this object - they do NOT go on the session.

**永远从这里开始。** `model`、`system`、`tools`、`mcp_servers`、`skills` 是这个对象的顶层字段——它们不放在会话上。

```json
{
  "name": "string (required, 1-256 chars)",
  "model": "claude-opus-5-5 (required - bare string, or {id, speed?, effort?, inference_geo?} object)",
  "description": "string (optional, up to 2048 chars)",
  "system": "string (optional, up to 100,000 chars)",
  "tools": [
    { "type": "agent_toolset_20260401" }
  ],
  "skills": [
    { "type": "anthropic", "skill_id": "xlsx" },
    { "type": "custom", "skill_id": "skill_abc123", "version": "1" }
  ],
  "mcp_servers": [
    {
      "type": "url",
      "name": "github",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  ],
  "multiagent": {
    "type": "coordinator",
    "agents": [
      "agent_abc123",
      { "type": "agent", "id": "agent_def456", "version": 4 },
      { "type": "self" }
    ]
  },
  "metadata": {
    "key": "value (max 16 pairs, keys <=64 chars, values <=512 chars)"
  }
}
```

> Limits: `tools` max 128, `skills` max 20, `mcp_servers` max 20 (unique names). `multiagent.agents` 1-20 entries (string ID | `{type:"agent",id,version?}` | `{type:"self"}` | `{type:"advisor",model}`, at most one advisor) - see `shared/managed-agents-multiagent.md`.
> 限制：`tools` 最多 128，`skills` 最多 20，`mcp_servers` 最多 20（名字唯一）。`multiagent.agents` 为 1-20 个条目（字符串 ID | `{type:"agent",id,version?}` | `{type:"self"}` | `{type:"advisor",model}`，advisor 至多一个）——见 `shared/managed-agents-multiagent.md`。

### CreateSession Request Body / CreateSession 请求体

```json
{
  "agent": "agent_abc123 (required - string shorthand for latest version, or {type: \"agent\", id, version} object)",
  "environment_id": "env_abc123 (required)",
  "title": "string (optional)",
  "resources": [
    {
      "type": "github_repository",
      "url": "https://github.com/owner/repo (required)",
      "authorization_token": "ghp_... (required)",
      "mount_path": "/workspace/repo (optional - defaults to /workspace/<repo-name>)",
      "checkout": { "type": "branch", "name": "main" }
    }
  ],
  "initial_events": [
    { "type": "user.message", "content": [{ "type": "text", "text": "Review the auth module." }] }
  ],
  "vault_ids": ["vlt_abc123 (optional - vault credentials: MCP auth + environment variables)"],
  "budget": {
    "type": "limit",
    "max_list_cost": { "amount": "2500", "currency": "USD" }
  },
  "metadata": {
    "key": "value"
  }
}
```

> The `agent` field accepts a string ID, `{type: "agent", id, version}`, or `{type: "agent_with_overrides", id, version?, ...}` for session-local overrides of `model`/`system`/`tools`/`mcp_servers`/`skills`. Outside the overrides form, those fields live on the agent, not here. An `effort` inside a `model` override is applied (the agent's own `effort` isn't carried over, and a `model` override without `effort` runs at that model's default effort level). An `inference_geo` inside a `model` override **is** applied (omitting it clears the agent's pin for this session).  
>  
> **`budget`** (optional, create-only) is a hard dollar cap on the session's list-priced spend; `amount` is an integer string in minor units (cents - `"2500"` = $25.00), `USD` only. It can be changed or removed later via session update, never added. See `shared/managed-agents-core.md` -> Session budgets.  
>  
> **`initial_events`** (optional, max 50) sends events at creation and starts the agent loop in the same call. Only `user.message` and `user.define_outcome` are accepted - no `system.message`, and none of the tool-result kinds. Validation is all-or-nothing. See `shared/managed-agents-core.md` -> Seeding a session with `initial_events`.  
>  
> **`checkout`** accepts `{type: "branch", name: "..."}` or `{type: "commit", sha: "..."}`. Omit for the repo's default branch.
> `agent` 字段接受字符串 ID、`{type: "agent", id, version}`，或用于对 `model`/`system`/`tools`/`mcp_servers`/`skills` 做会话本地覆盖的 `{type: "agent_with_overrides", id, version?, ...}`。在覆盖形式之外，那些字段放在代理上，不放在这里。`model` 覆盖内的 `effort` 会被应用（代理自身的 `effort` 不会被带过来，不带 `effort` 的 `model` 覆盖按该模型的默认 effort 级别运行）。`model` 覆盖内的 `inference_geo` **会**被应用（省略它会清掉代理在本会话的区域钉住）。  
>  
> **`budget`**（可选，仅创建时）是对会话按清单价支出的硬性美元上限；`amount` 是以最小单位（美分——`"2500"` = $25.00）计的整数字符串，仅支持 `USD`。之后可通过会话更新更改或移除，绝不能再添加。见 `shared/managed-agents-core.md` -> Session budgets。  
>  
> **`initial_events`**（可选，最多 50）在创建时发送事件并在同一次调用中启动代理循环。只接受 `user.message` 和 `user.define_outcome`——不接受 `system.message`，也不接受任何工具结果类型。校验是全有或全无。见 `shared/managed-agents-core.md` -> Seeding a session with `initial_events`。  
>  
> **`checkout`** 接受 `{type: "branch", name: "..."}` 或 `{type: "commit", sha: "..."}`。省略则用仓库的默认分支。

### CreateEnvironment Request Body / CreateEnvironment 请求体

```json
{
  "name": "string (required)",
  "description": "string (optional)",
  "config": {
    "type": "cloud | self_hosted",
    "networking": {
      "type": "unrestricted | limited (union - see SDK types)"
    },
    "packages": { }
  },
  "metadata": { "key": "value" }
}
```

### CreateDeployment Request Body / CreateDeployment 请求体

```json
{
  "name": "Weekly compliance scan",
  "agent": "agent_abc123 (required - same shapes as CreateSession)",
  "environment_id": "env_abc123 (required)",
  "initial_events": [
    { "type": "user.message", "content": [{ "type": "text", "text": "Run the weekly compliance scan." }] }
  ],
  "schedule": {
    "type": "cron",
    "expression": "0 20 * * 5",
    "timezone": "America/New_York"
  }
}
```

> Optional session config (`resources`, `vault_ids`, etc.) is supported the same way as on CreateSession, including `budget` - copied onto each fired session; unlike a session's, it can be added where none exists and re-added after clearing (see `shared/managed-agents-scheduled-deployments.md` § Deployment budgets). Response includes `status`, `paused_reason`, and `schedule.upcoming_runs_at` (next fire times). See `shared/managed-agents-scheduled-deployments.md`.
> 可选的会话配置（`resources`、`vault_ids` 等）以与 CreateSession 相同的方式支持，包括 `budget`——复制到每个被触发的会话上；与会话上的不同，它可以在原本没有时添加，也可在清除后重新添加（见 `shared/managed-agents-scheduled-deployments.md` § Deployment budgets）。响应包含 `status`、`paused_reason` 和 `schedule.upcoming_runs_at`（下次触发时间）。见 `shared/managed-agents-scheduled-deployments.md`。

### SendEvents Request Body / SendEvents 请求体

```json
{
  "events": [
    {
      "type": "user.message",
      "content": [
        {
          "type": "text",
          "text": "Hello"
        }
      ]
    }
  ]
}
```

> `system.message` events (append system-level context for this turn and later ones) use the same envelope with `type: "system.message"` - supported on Claude Fable 5.1, Claude Mythos 5.1, Claude Fable 5, Claude Mythos 5, Claude Opus 5.5, Claude Opus 5, Claude Opus 4.8, and Claude Sonnet 5.5 (not Claude Sonnet 5), checked against the agent's *primary* model only; see `shared/managed-agents-events.md` § Adding system context mid-session.
> `system.message` 事件（为本回合及之后的回合追加系统级上下文）使用同样的信封、`type: "system.message"`——在 Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8 和 Claude Sonnet 5.5 上受支持（Claude Sonnet 5 不支持），且只按代理的*主*模型检查；见 `shared/managed-agents-events.md` § Adding system context mid-session。

### Define Outcome Event / Define Outcome 事件

```json
{
  "type": "user.define_outcome",
  "description": "Build a DCF model for Costco in .xlsx",
  "rubric": { "type": "file", "file_id": "file_01..." },
  "max_iterations": 5
}
```

> `rubric` is required: `{type: "text", content}` or `{type: "file", file_id}`. `max_iterations` default 3, max 20. Echoed back with `outcome_id` + `processed_at`. See `shared/managed-agents-outcomes.md`.
> `rubric` 是必需的：`{type: "text", content}` 或 `{type: "file", file_id}`。`max_iterations` 默认 3，最大 20。会随 `outcome_id` + `processed_at` 回显。见 `shared/managed-agents-outcomes.md`。

### Tool Result Event / 工具结果事件

```json
{
  "type": "user.custom_tool_result",
  "custom_tool_use_id": "sevt_abc123",
  "content": [{ "type": "text", "text": "Result data" }],
  "is_error": false
}
```

---

## Error Handling / 错误处理

Managed Agents endpoints use the standard Anthropic API error format. Errors are returned with an HTTP status code and a JSON body containing `type`, `error`, and `request_id`:

Managed Agents 端点使用标准的 Anthropic API 错误格式。错误以一个 HTTP 状态码和一个包含 `type`、`error`、`request_id` 的 JSON 体返回：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Description of what went wrong"
  },
  "request_id": "req_011CRv1W3XQ8XpFikNYG7RnE"
}
```

Include the `request_id` when reporting issues to Anthropic - it lets us trace the request end-to-end. The inner `error.type` is one of the following:

向 Anthropic 报告问题时请附上 `request_id`——它让我们能端到端追踪请求。内层 `error.type` 是以下之一：

| Status | Error type | Description |
|---|---|---|
| 400 | `invalid_request_error` | The request was malformed or missing required parameters |
| 401 | `authentication_error` | Invalid or missing API key |
| 403 | `permission_error` | The API key doesn't have permission for this operation |
| 404 | `not_found_error` | The requested resource doesn't exist |
| 409 | `invalid_request_error` | The request conflicts with the resource's current state (e.g., sending to an archived session) |
| 413 | `request_too_large` | The request body exceeds the maximum allowed size |
| 429 | `rate_limit_error` | Too many requests - check rate limit headers for retry timing |
| 500 | `api_error` | An internal server error occurred |
| 529 | `overloaded_error` | The service is temporarily overloaded - retry with backoff |

| 状态 | 错误类型 | 说明 |
|---|---|---|
| 400 | `invalid_request_error` | 请求格式错误或缺少必需参数 |
| 401 | `authentication_error` | API 密钥无效或缺失 |
| 403 | `permission_error` | API 密钥没有该操作的权限 |
| 404 | `not_found_error` | 所请求的资源不存在 |
| 409 | `invalid_request_error` | 请求与资源的当前状态冲突（例如发送到已归档的会话） |
| 413 | `request_too_large` | 请求体超过允许的最大尺寸 |
| 429 | `rate_limit_error` | 请求过多——查看速率限制响应头确定重试时机 |
| 500 | `api_error` | 发生了内部服务器错误 |
| 529 | `overloaded_error` | 服务暂时过载——带退避重试 |

Note that `409 Conflict` carries `error.type: "invalid_request_error"` (there is no separate `conflict_error` type); inspect both the HTTP status and the `message` to distinguish conflicts from other invalid requests.

注意 `409 Conflict` 携带的是 `error.type: "invalid_request_error"`（没有单独的 `conflict_error` 类型）；要区分冲突与其他无效请求，需同时查看 HTTP 状态码和 `message`。

---

## Pagination / 分页

Most Managed Agents list endpoints use the `page` / `next_page` cursor scheme:

大多数 Managed Agents 列表端点使用 `page` / `next_page` 游标方案：

| Field | Where | Notes |
|---|---|---|
| `limit` | query | Max items per page |
| `page` | query | Opaque cursor from a previous response - pass a `next_page` or `prev_page` value here |
| `order` | query | `asc` / `desc` on endpoints that support sorting. A cursor encodes the `order` of the request that produced it - reusing it with a different `order` returns 400. Other params (filters, `limit`) can change between paginated requests. |
| `next_page` | response | Cursor for the next page; `null` when there are no more results |
| `prev_page` | response | Cursor for the previous page on endpoints that support backward pagination - currently **only `GET /v1/sessions`**. `null` on the first page. On endpoints that don't support it, the field is **absent** (not `null`). |

| 字段 | 位置 | 说明 |
|---|---|---|
| `limit` | query | 每页最大条数 |
| `page` | query | 来自先前响应的不透明游标——把 `next_page` 或 `prev_page` 的值传到这里 |
| `order` | query | 支持排序的端点上的 `asc` / `desc`。游标编码了产生它的请求的 `order`——用不同的 `order` 重用它返回 400。其他参数（过滤器、`limit`）可以在分页请求之间变化。 |
| `next_page` | response | 下一页的游标；没有更多结果时为 `null` |
| `prev_page` | response | 支持向后分页的端点上的上一页游标——目前**只有 `GET /v1/sessions`**。首页上为 `null`。在不支持它的端点上，该字段**不存在**（而不是 `null`）。 |

Every SDK exposes an auto-paginating iterator that follows `next_page`. In Python and TypeScript, iterate the list result directly; the other SDKs expose the iterator via a separate method (iterating the plain list result returns one page). SDK auto-pagination is **forward-only** - to go back a page, read `prev_page` from the response and pass it back as the `page` parameter yourself.

每个 SDK 都暴露遵循 `next_page` 的自动分页迭代器。在 Python 和 TypeScript 中直接迭代列表结果即可；其他 SDK 通过单独的方法暴露迭代器（迭代普通列表结果只返回一页）。SDK 自动分页是**仅向前**的——要回退一页，需自己从响应读取 `prev_page` 并作为 `page` 参数传回。

> Warning: Some endpoints use a **different** cursor scheme: Message Batches, Files, Models, and several Admin API endpoints take `after_id`/`before_id` and return `has_more`/`first_id`/`last_id` instead of `page`/`next_page`. Some `page`-scheme endpoints (e.g. `GET /v1/skills`) also return a `has_more` boolean alongside `next_page`. Check the endpoint's reference page for its exact pagination fields.
> 警告：一些端点使用**不同的**游标方案：Message Batches、Files、Models 以及若干 Admin API 端点接受 `after_id`/`before_id`，返回 `has_more`/`first_id`/`last_id` 而不是 `page`/`next_page`。一些 `page` 方案的端点（例如 `GET /v1/skills`）还会在 `next_page` 旁边返回 `has_more` 布尔值。请查阅端点的参考页确认其确切的分页字段。

---

## Rate Limits / 速率限制

Managed Agents endpoints have per-organization request-per-minute (RPM) limits, separate from your [Messages API token limits](https://platform.claude.com/docs/en/api/rate-limits). Model inference inside a session still draws from your organization's standard ITPM/OTPM limits.

Managed Agents 端点有按组织计的每分钟请求数（RPM）限制，与你 的[Messages API token 限制](https://platform.claude.com/docs/en/api/rate-limits)相互独立。会话内的模型推理仍消耗你组织的标准 ITPM/OTPM 限制。

| Endpoint group | Scope | RPM | Max concurrent |
|---|---|---|---|
| Create operations (Agents, Sessions, Vaults) | organization | 300 | - |
| All other operations (Agents, Sessions, Vaults) | organization | 600 | - |
| All operations (Environments) | organization | 60 | 5 |

| 端点组 | 范围 | RPM | 最大并发 |
|---|---|---|---|
| 创建类操作（Agents、Sessions、Vaults） | 组织 | 300 | - |
| 所有其他操作（Agents、Sessions、Vaults） | 组织 | 600 | - |
| 所有操作（Environments） | 组织 | 60 | 5 |

Files and Skills endpoints use the standard tier-based [rate limits](https://platform.claude.com/docs/en/api/rate-limits).

Files 与 Skills 端点使用标准的按层级[速率限制](https://platform.claude.com/docs/en/api/rate-limits)。

When a limit is exceeded the API returns `429` with a `rate_limit_error` (see [Error Handling](#error-handling) for the response envelope) and a `retry-after` header indicating how many seconds to wait before retrying. The Anthropic SDK reads this header and retries automatically.

超限时 API 返回 `429` 和 `rate_limit_error`（响应信封见[错误处理](#error-handling)），并附带一个 `retry-after` 响应头，指明重试前要等待的秒数。Anthropic SDK 会读取该响应头并自动重试。
