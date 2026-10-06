<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Core Concepts / Managed Agents（托管智能体）- 核心概念

## Architecture / 架构

Managed Agents is built around four core concepts:

Managed Agents 围绕四个核心概念构建：

| Concept | Endpoint | What it is |
|---|---|---|
| **Agent** | `/v1/agents` | A persisted, versioned object defining the agent's capabilities and persona: model, system prompt, tools, MCP servers, skills. **Must be created before starting a session.** See the Agents section below. |
| **Session** | `/v1/sessions` | A stateful interaction with an agent. References a pre-created agent by ID + an environment + initial instructions. Produces an event stream. |
| **Environment** | `/v1/environments` | A template defining the configuration for container provisioning. |
| **Container** | N/A | An isolated compute instance where the agent's **tools** execute (bash, file ops, code). The agent loop does not run here - it runs on Anthropic's orchestration layer and acts on the container via tool calls. |

| 概念 | 端点 | 说明 |
|---|---|---|
| **Agent** | `/v1/agents` | 一个持久化、带版本的对象，定义智能体的能力与人设：模型、系统提示词、工具、MCP 服务器、技能。**必须在启动会话之前创建。**参见下文 Agents 一节。 |
| **Session** | `/v1/sessions` | 与智能体的一次有状态交互。按 ID 引用一个预先创建的智能体，外加一个环境与初始指令。会产生一个事件流。 |
| **Environment** | `/v1/environments` | 定义容器供给配置的模板。 |
| **Container** | N/A | 一个隔离的计算实例，智能体的**工具**在其中执行（bash、文件操作、代码）。智能体循环不在此处运行——它运行在 Anthropic 的编排层上，并通过工具调用对容器进行操作。 |

```
                       +-------------------------------------+
                       |  Anthropic orchestration layer      |
Agent (config) ------->|  (agent loop: Claude + tool calls)  |
                       +--------------+----------------------+
                                      | tool calls
                                      v
Environment (template) --> Container (tool execution workspace)
                                 |
                         Session -+
                                 +-- Resources (files, repos, memory stores - attached at startup)
                                 +-- Vault IDs (MCP credential references)
                                 +-- Conversation (event stream in/out)
```

> **Agent creation is a prerequisite.** Sessions reference a pre-created agent by ID - `model`/`system`/`tools` live on the agent object, never on the session. Every flow starts with `POST /v1/agents`.
> **创建智能体是前置条件。**会话按 ID 引用预先创建的智能体——`model`/`system`/`tools` 位于智能体对象上，绝不会位于会话上。每个流程都从 `POST /v1/agents` 开始。

---

## Session Lifecycle / 会话生命周期

```
rescheduling -> running <-> idle -> terminated
```

| Status         | Description                                                        |
| -------------- | ------------------------------------------------------------------ |
| `idle` | Agent has finished the current task, and is awaiting input. It's either waiting for input to continue working via a `user.message`, blocked awaiting a `user.custom_tool_result` or `user.tool_confirmation`, or paused because the session budget cap was reached. The `stop_reason` attached contains more information about why the Agent has stopped working. |
| `running` | Session has starting running, and the Agent is actively doing work. |
| `rescheduling` | Session is (re)scheduling after a retryable error has occurred, ready to be picked up by the orchestration system. |
| `terminated` | Session has ended and is in an irreversible, unusable state - **either on completion or because of an unrecoverable error**. Terminated does not by itself mean failure; fetch the session to tell the two apart. |

| 状态 | 说明 |
| --- | --- |
| `idle` | 智能体已完成当前任务，正在等待输入。它要么在等待通过 `user.message` 提供输入以继续工作，要么在阻塞等待 `user.custom_tool_result` 或 `user.tool_confirmation`，要么因会话预算上限已达到而暂停。所附带的 `stop_reason` 包含关于智能体为何停止工作的更多信息。 |
| `running` | 会话已开始运行，智能体正在积极工作。 |
| `rescheduling` | 会话在发生可重试错误后正在（重新）调度，等待编排系统接手处理。 |
| `terminated` | 会话已结束，处于不可逆、不可用的状态——**要么是正常完成，要么是因为发生了不可恢复的错误**。terminated 本身并不代表失败；需获取会话来区分这两种情况。 |

- Events can be sent when the session is `running` or `idle`. Messages are queued and processed in order. Exception: a session paused at its budget (`stop_reason: budget_reached`) accepts only **settle events** - events that resolve work already in progress (`user.tool_confirmation`, `user.tool_result`, `user.custom_tool_result`, `user.interrupt`) rather than starting new work - see § Session budgets.
  事件可以在会话处于 `running` 或 `idle` 时发送。消息会排队并按顺序处理。例外：因预算而暂停的会话（`stop_reason: budget_reached`）只接受**结算事件（settle events）**——即处理已在进行中的工作（`user.tool_confirmation`、`user.tool_result`、`user.custom_tool_result`、`user.interrupt`）而非开启新工作的事件——参见 § Session budgets（会话预算）。
- The agent transitions `idle -> running` when it receives a new event, then back to `idle` when done.
  智能体在收到新事件时从 `idle -> running` 转换，完成后再转回 `idle`。
- Errors surface as `session.error` events in the stream, not as a status value.
  错误以流中的 `session.error` 事件形式呈现，而不是作为状态值。

Every session has a live trace view in the Anthropic Console at `https://platform.claude.com/workspaces/{workspace}/sessions/{session_id}`. Print this URL immediately after creating a session so the user can watch tool calls and messages stream in real time. **`{workspace}` is the workspace the API key belongs to** - use `default` only when that's the org's Default workspace. The session response does **not** include a workspace field and the Console has no workspace-agnostic session route, so for non-default workspaces substitute the workspace's ID (visible in the Console URL bar, or expose it as a config value alongside the API key). A `default` link to a session that lives in another workspace lands on a **"Session not found"** page - the **Search workspaces** button there will locate it, but it is not an automatic redirect.

每个会话在 Anthropic Console 中都有一个实时追踪视图：`https://platform.claude.com/workspaces/{workspace}/sessions/{session_id}`。创建会话后应立即打印该 URL，让用户能实时观看工具调用与消息流入。**`{workspace}` 是 API key 所属的工作区**——只有当它就是组织的 Default 工作区时才使用 `default`。会话响应**不**包含 workspace 字段，Console 也没有与工作区无关的会话路由，因此对于非默认工作区，请替换为该工作区的 ID（可在 Console 的 URL 栏中看到，或将其作为配置值与 API key 一起暴露）。指向其他工作区会话的 `default` 链接会落在 **"Session not found"** 页面——该页面上的 **Search workspaces** 按钮可以找到它，但这不是自动重定向。

### Built-in session features / 内置会话功能

- **Context compaction** - if you approach max context, the API automatically condenses session history to keep the interaction going
  **上下文压缩**——当你接近最大上下文时，API 会自动压缩会话历史以保持交互继续
- **Prompt caching** - historical repeated tokens are cached, reducing processing time and cost
  **提示词缓存**——历史中重复出现的 token 会被缓存，降低处理时间与成本
- **Extended thinking** - on by default; `agent.thinking` events signal thinking progress and carry no thinking content
  **扩展思考**——默认开启；`agent.thinking` 事件标示思考进度，但不携带思考内容

### Session operations / 会话操作

| Operation | Notes |
|---|---|
| List / fetch | Paginated list or single resource by ID |
| Update | `title`, `metadata`, and the session-local `agent.tools`/`agent.mcp_servers` can be overridden (see § Updating the agent configuration mid-session). `budget` can only be changed or removed (see § Session budgets). `vault_ids` is create-only - update requests setting it are rejected. |
| Archive | Session becomes **read-only**. Not reversible. |
| Delete | Permanently deletes session, event history, container, and checkpoints. |

| 操作 | 说明 |
|---|---|
| 列表 / 获取 | 分页列表，或按 ID 获取单个资源 |
| 更新 | `title`、`metadata` 以及会话本地的 `agent.tools`/`agent.mcp_servers` 可以被覆盖（参见 § Updating the agent configuration mid-session）。`budget` 只能被修改或移除（参见 § Session budgets）。`vault_ids` 仅创建时可设——设置它的更新请求会被拒绝。 |
| 归档 | 会话变为**只读**。不可逆。 |
| 删除 | 永久删除会话、事件历史、容器与检查点。 |

These are ops/inspection calls - typically made from a terminal, not application code. From the shell (see `shared/anthropic-cli.md`):

这些是运维/检查类调用——通常从终端发出，而非写在应用代码里。在 shell 中（参见 `shared/anthropic-cli.md`）：

```sh
ant beta:sessions list --transform '{id,title,status,created_at}' --format jsonl
ant beta:sessions retrieve --session-id "$SID"
ant beta:sessions:events stream --session-id "$SID"   # watch events live
ant beta:sessions archive  --session-id "$SID"
ant beta:sessions delete   --session-id "$SID"
```

---

## Sessions / 会话（Sessions）

A session is a running agent instance inside an environment.

会话（session）是运行在环境（environment）内部的智能体实例。

### Session Object / 会话对象

Key fields returned by the API:

API 返回的关键字段：

| Field           | Type     | Description                                         |
| --------------- | -------- | --------------------------------------------------- |
| `type` | string | Always `"session"` |
| `id` | string | Unique session ID |
| `title` | string | Human-readable title |
| `status` | string | `idle`, `running`, `rescheduling`, `terminated` |
| `created_at` | string | ISO 8601 timestamp |
| `updated_at` | string | ISO 8601 timestamp |
| `archived_at` | string | ISO 8601 timestamp (nullable) |
| `environment_id` | string | Environment ID |
| `agent` | object | Agent configuration |
| `resources` | array | Attached files, repos, and memory stores |
| `metadata` | object | User-provided key-value pairs (max 8 keys) |
| `usage` | object | Cumulative usage: token counts, `server_tool_use` (web search/fetch request counts), `list_cost` (consumption priced at public list rates, as `{amount, currency}` with the amount an integer string in minor units - cents), and `active_seconds` (time with >=1 thread running; concurrent-thread overlap counted once - unlike `stats.active_seconds`, which sums per-thread time) |
| `budget` | object | The session's spend cap, when one was set at creation - see § Session budgets |
| `stats` | object | Timing statistics - `stats.active_seconds` sums per-thread time, unlike `usage.active_seconds` |

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 始终为 `"session"` |
| `id` | string | 唯一的会话 ID |
| `title` | string | 人类可读的标题 |
| `status` | string | `idle`、`running`、`rescheduling`、`terminated` |
| `created_at` | string | ISO 8601 时间戳 |
| `updated_at` | string | ISO 8601 时间戳 |
| `archived_at` | string | ISO 8601 时间戳（可为 null） |
| `environment_id` | string | 环境 ID |
| `agent` | object | 智能体配置 |
| `resources` | array | 附加的文件、仓库与记忆存储 |
| `metadata` | object | 用户提供的键值对（最多 8 个键） |
| `usage` | object | 累计用量：token 计数、`server_tool_use`（网页搜索/抓取请求次数）、`list_cost`（按公开挂牌费率计价的消费，形如 `{amount, currency}`，其中 amount 为最小货币单位（美分）的整数字符串），以及 `active_seconds`（至少 1 个线程在运行的时间；并发线程的重叠时间只计一次——不同于按线程逐一累加的 `stats.active_seconds`） |
| `budget` | object | 会话的消费上限（如果创建时设置了）——参见 § Session budgets |
| `stats` | object | 时间统计——`stats.active_seconds` 按线程累加时间，不同于 `usage.active_seconds` |

### Creating a session / 创建会话

**A session is meaningless without an agent.** Sessions reference a pre-created agent by ID. Create the agent first via `agents.create()`, then reference it:

**没有智能体，会话就毫无意义。**会话按 ID 引用预先创建的智能体。先通过 `agents.create()` 创建智能体，再引用它：

```ts
// 1. Create the agent (reusable, versioned)
const agent = await client.beta.agents.create(
  {
    name: "Coding Assistant",
    model: "claude-opus-5-5",
    system: "You are a helpful coding agent.",
    tools: [{ type: "agent_toolset_20260401"}],
  },
);

// 2. Start a session that references it
const session = await client.beta.sessions.create(
  {
    agent: agent.id,  // string shorthand -> latest version. Or: { type: "agent", id: agent.id, version: agent.version }
    environment_id: environmentId,
    title: "Hello World Session",
  },
);
```

> Tip: **Watch it live in Console.** While developing, print a link so you can click through to the session UI instead of parsing the event stream: `https://platform.claude.com/workspaces/{workspace}/sessions/{session.id}`. Use `default` for `{workspace}` only when the API key belongs to the org's Default workspace; otherwise substitute the workspace's ID (the session response does not carry it - read it from the Console URL bar or make it a config value). Include this `print`/`console.log` in code you generate for users who are iterating locally.
> 提示：**在 Console 中实时观看。**开发过程中，打印一个链接以便直接点击进入会话 UI，而不必解析事件流：`https://platform.claude.com/workspaces/{workspace}/sessions/{session.id}`。只有当 API key 属于组织的 Default 工作区时，`{workspace}` 才使用 `default`；否则请替换为该工作区的 ID（会话响应不携带它——从 Console 的 URL 栏读取，或将其做成配置值）。在你为本地迭代的用户生成的代码中，应包含这条 `print`/`console.log`。

**Session creation parameters:**

**会话创建参数：**

| Field           | Type     | Required | Description                                    |
| --------------- | -------- | -------- | ---------------------------------------------- |
| `agent`         | string or object | **Yes** | Three forms: string shorthand `"agent_abc123"` (latest version); pinned `{type: "agent", id, version}`; or `{type: "agent_with_overrides", id, version?, ...}` to override `model`/`system`/`tools`/`mcp_servers`/`skills` for this session only - see § Override agent configuration for a session |
| `environment_id`| string   | **Yes**  | Environment ID                                 |
| `title`         | string   | No       | Human-readable name (appears in logs/dashboards) |
| `resources`     | array    | No       | Files, GitHub repos, or memory stores, attached to the container at startup. Memory stores are session-create-only (not addable via `resources.add()`). |
| `initial_events`| array    | No       | Events to send at creation, processed in order - collapses create + first send into one call. See § Seeding a session with `initial_events` below. |
| `vault_ids`     | array    | No       | Vault IDs (`vlt_*`) - MCP credentials with auto-refresh + `environment_variable` secrets substituted at egress. See `shared/managed-agents-tools.md` -> Vaults. |
| `budget`        | object   | No       | Hard dollar cap on the session's spend: `{type: "limit", max_list_cost: {amount, currency}}`. **Create-only** - can be changed or removed later, never added. See § Session budgets. |
| `metadata`      | object   | No       | User-provided key-value pairs                  |

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `agent` | string or object | **是** | 三种形式：字符串简写 `"agent_abc123"`（最新版本）；固定版本的 `{type: "agent", id, version}`；或 `{type: "agent_with_overrides", id, version?, ...}`，仅为该会话覆盖 `model`/`system`/`tools`/`mcp_servers`/`skills`——参见 § Override agent configuration for a session |
| `environment_id`| string | **是** | 环境 ID |
| `title` | string | 否 | 人类可读的名称（出现在日志/仪表板中） |
| `resources` | array | 否 | 文件、GitHub 仓库或记忆存储，在启动时附加到容器。记忆存储只能在创建会话时提供（无法通过 `resources.add()` 追加）。 |
| `initial_events`| array | 否 | 创建时发送的事件，按顺序处理——将创建与首次发送合并为一次调用。参见下文 § Seeding a session with `initial_events`。 |
| `vault_ids` | array | 否 | Vault ID（`vlt_*`）——支持自动刷新的 MCP 凭据，出口处会替换 `environment_variable` 机密。参见 `shared/managed-agents-tools.md` -> Vaults。 |
| `budget` | object | 否 | 会话支出的硬性金额上限：`{type: "limit", max_list_cost: {amount, currency}}`。**仅创建时可设**——之后可以修改或移除，但无法再新增。参见 § Session budgets。 |
| `metadata` | object | 否 | 用户提供的键值对 |

#### Seeding a session with `initial_events` / 用 `initial_events` 预置会话

Creating a session without `initial_events` registers the session in `idle` and starts no work; the sandbox is provisioned when the session first needs it. Passing a **non-empty** `initial_events` array starts the agent loop in the same call - the session is **created directly in `running`**, never passing through `idle`. A client that waits for an `idle -> running` transition to know work began will wait forever; check `status` on the create response instead.

不带 `initial_events` 创建会话时，会话以 `idle` 状态注册且不启动任何工作；沙箱在会话首次需要时才供给。传入**非空**的 `initial_events` 数组会在同一次调用中启动智能体循环——会话**直接以 `running` 状态创建**，绝不经过 `idle`。靠等待 `idle -> running` 转换来判断工作已开始的客户端将永远等待；应改为检查创建响应中的 `status`。

【评论】"直接以 running 创建、不经过 idle"是一个容易踩中的状态机细节：按传统"等待 running 出现"来轮询的客户端会永远挂起，文档特意提醒改为读取创建响应中的 status。

```python
session = client.beta.sessions.create(
    agent=AGENT_ID,
    environment_id=ENVIRONMENT_ID,
    initial_events=[
        {"type": "user.message", "content": [{"type": "text", "text": "Review the auth module."}]},
    ],
)
```

- **Only `user.message` and `user.define_outcome` are accepted**, max **50** events. The tool-result kinds (`user.tool_confirmation`, `user.tool_result`, `user.custom_tool_result`) are rejected because no agent turn exists yet, and `user.interrupt` because there is no turn to stop. Unlike a scheduled deployment's `initial_events`, a session's does **not** accept `system.message`.
  **只接受 `user.message` 与 `user.define_outcome`**，最多 **50** 个事件。工具结果类事件（`user.tool_confirmation`、`user.tool_result`、`user.custom_tool_result`）会被拒绝，因为此时还不存在任何智能体回合；`user.interrupt` 也被拒绝，因为没有可停止的回合。与定时部署的 `initial_events` 不同，会话的 `initial_events` **不**接受 `system.message`。
- Each event is validated and persisted before the create response returns, in list order, with a server-assigned ID - exactly as if you had posted it to the send-events endpoint immediately after creation. Per-event content rules are the same as on that endpoint.
  每个事件都会在创建响应返回之前按列表顺序完成校验并持久化，并获得服务器分配的 ID——效果与你在创建后立即把事件投递到发送事件端点完全一致。单事件的内容规则与该端点相同。
- **The events are not echoed on the create response.** Read them back with `sessions.events.list(session.id)` if you need their server-assigned IDs.
  **创建响应不会回显这些事件。**如果需要它们的服务器分配 ID，可用 `sessions.events.list(session.id)` 读回。
- **Validation is all-or-nothing:** if any event fails, the whole request is rejected and no session is created. An empty list is equivalent to omitting the field.
  **校验是全有或全无的：**任何一个事件失败，整个请求都会被拒绝，且不会创建会话。空列表等价于省略该字段。
- Rejections: more than one `user.define_outcome` -> 400; a `user.define_outcome` without a `rubric` -> 400; more than 100 file-sourced `document` content blocks across the whole list -> 400; a request body over 32 MB -> 413.
  拒绝情形：多于一个 `user.define_outcome` -> 400；`user.define_outcome` 缺少 `rubric` -> 400；整个列表中来自文件的 `document` 内容块超过 100 个 -> 400；请求体超过 32 MB -> 413。

An outcome-driven session is therefore a single call - pass one `user.define_outcome` in `initial_events` instead of creating the session and then sending the event (see `shared/managed-agents-outcomes.md`).

因此，结果驱动（outcome-driven）的会话只需一次调用——在 `initial_events` 中传入一个 `user.define_outcome`，而不是先创建会话再发送事件（参见 `shared/managed-agents-outcomes.md`）。

**Agent configuration fields** (passed to `agents.create()`, not `sessions.create()`):

**智能体配置字段**（传递给 `agents.create()`，而非 `sessions.create()`）：

| Field         | Type     | Required | Description                                    |
| ------------- | -------- | -------- | ---------------------------------------------- |
| `name`        | string   | **Yes**  | Human-readable name (1-256 chars)              |
| `model`       | string or object | **Yes** | Claude model ID (bare string, or an object taking `id`, `speed`, `effort`, and `inference_geo`). All Claude 4.5+ models supported. See § Effort on the agent model and § Pinning inference geography below. |
| `system`      | string   | No       | System prompt - defines the agent's behavior (up to 100K chars) |
| `tools`       | array    | No       | Encompasses three kinds: (1) pre-built Claude Agent tools (`agent_toolset_20260401`), (2) MCP tools (`mcp_toolset`), and (3) custom client-side tools. Max 128. |
| `mcp_servers` | array    | No       | MCP server connections - standardized third-party capabilities (e.g. GitHub, Asana). Max 20, unique names. See `shared/managed-agents-tools.md` -> MCP Servers. |
| `skills`      | array    | No       | Customized "best-practices" context with progressive disclosure. Max 20. See `shared/managed-agents-tools.md` -> Skills. |
| `description` | string   | No       | Description of the agent (up to 2048 chars)    |
| `multiagent`  | object   | No       | `{type: "coordinator", agents: [...]}` - roster this agent may delegate to. See `shared/managed-agents-multiagent.md`. |
| `metadata`    | object   | No       | Arbitrary key-value pairs (max 16, keys <=64 chars, values <=512 chars) |

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | **是** | 人类可读的名称（1-256 字符） |
| `model` | string or object | **是** | Claude 模型 ID（裸字符串，或接受 `id`、`speed`、`effort`、`inference_geo` 的对象）。支持所有 Claude 4.5+ 模型。参见下文 § Effort on the agent model 与 § Pinning inference geography。 |
| `system` | string | 否 | 系统提示词——定义智能体的行为（最多 100K 字符） |
| `tools` | array | 否 | 包含三类：(1) 预构建的 Claude Agent 工具（`agent_toolset_20260401`），(2) MCP 工具（`mcp_toolset`），(3) 自定义客户端工具。最多 128 个。 |
| `mcp_servers` | array | 否 | MCP 服务器连接——标准化的第三方能力（如 GitHub、Asana）。最多 20 个，名称唯一。参见 `shared/managed-agents-tools.md` -> MCP Servers。 |
| `skills` | array | 否 | 带渐进式披露的定制"最佳实践"上下文。最多 20 个。参见 `shared/managed-agents-tools.md` -> Skills。 |
| `description` | string | 否 | 智能体的描述（最多 2048 字符） |
| `multiagent` | object | 否 | `{type: "coordinator", agents: [...]}`——该智能体可委派的名册。参见 `shared/managed-agents-multiagent.md`。 |
| `metadata` | object | 否 | 任意键值对（最多 16 个，键 <=64 字符，值 <=512 字符） |

### Session budgets / 会话预算

A **session budget** is an optional hard spend ceiling set at session creation. The platform continuously prices everything the session consumes at **public list rates** (the session's **list cost**) and stops issuing new model requests once that total reaches the cap. A session at its budget **pauses and goes `idle` with `stop_reason: budget_reached`** - it is not terminated; history and sandbox are preserved, and changing or removing the budget resumes the paused work automatically.

**会话预算（session budget）**是在创建会话时可选设置的硬性消费上限。平台按**公开挂牌费率**持续为会话消耗的一切计价（即会话的 **list cost**），一旦总额达到上限就停止发出新的模型请求。触及预算的会话会**暂停并进入 `idle` 状态，`stop_reason: budget_reached`**——它并不会被终止；历史与沙箱都会保留，修改或移除预算后会自动恢复被暂停的工作。

【评论】预算按公开挂牌费率（list rates）而非合同折扣价计算，因此会话触顶时实际计费支出可能低于上限值——文档特意点明 list cost 与账单金额是两个不同的口径。

```python
session = client.beta.sessions.create(
    agent=AGENT_ID,
    environment_id=ENVIRONMENT_ID,
    budget={
        "type": "limit",
        "max_list_cost": {"amount": "2500", "currency": "USD"},  # minor units: "2500" = $25.00
    },
)
```

- `type` is always `"limit"`. `max_list_cost.amount` is the amount in **minor units of the currency (cents), as an integer string** with no leading zeros, > 0 - `"2500"` is $25.00, `"50"` is fifty cents. A string rather than a number so no float rounding is ever applied; decimal forms such as `"25.00"` are rejected. `max_list_cost.currency` is uppercase ISO-4217; **`USD` is the only supported currency.**
  `type` 始终为 `"limit"`。`max_list_cost.amount` 是以**货币最小单位（美分）表示的整数字符串**，无前导零且大于 0——`"2500"` 表示 $25.00，`"50"` 表示五十美分。用字符串而非数字是为了绝不应用浮点舍入；`"25.00"` 这类小数形式会被拒绝。`max_list_cost.currency` 为大写 ISO-4217；**`USD` 是唯一支持的币种。**
- **What counts toward list cost:** model tokens at each served model's list price, web searches at $10 per 1,000, and session running time at $0.08/hour. List cost is *not* your contracted price - with negotiated discounts, the session hits the cap when the list-price total does, and billed spend may be lower.
  **计入 list cost 的项目：**按各服务模型挂牌价计算的模型 token、按每 1,000 次 $10 计费的网页搜索，以及按每小时 $0.08 计费的会话运行时间。list cost *不*是你的合同价——在有协商折扣的情况下，会话在挂牌价总额达到上限时触顶，实际计费支出可能更低。
- **Enforcement is a pre-request gate:** before every model request the platform checks whether consumed list cost has reached the cap and pauses the thread if it has; the request that crosses the cap completes, so the final figure can exceed the cap by at most one model request per running thread. Treat the budget as a bound on new work, not an exact stop.
  **执行方式是请求前闸门：**每次模型请求发出前，平台都会检查已消耗的 list cost 是否已达上限，若已达到则暂停线程；跨过上限的那次请求会执行完毕，因此最终数字最多可能超出上限"每个运行中线程一次模型请求"。应把预算视为对新工作的约束，而不是精确的停止点。
- The reported `list_cost` is **rounded to the nearest cent** while enforcement compares exact amounts - rounding can move the reported figure up to half a cent in either direction from the exact amount, so a session whose reported `list_cost` equals its cap may not yet be paused. Treat `stop_reason: budget_reached` (or the 400 on `user.message`), not the reported figure, as the signal that the cap was reached.
  上报的 `list_cost` 会**四舍五入到最接近的美分**，而执行时比较的是精确金额——舍入可能使上报数字相对精确金额上下偏移最多半美分，因此上报 `list_cost` 恰好等于上限的会话可能尚未暂停。应以 `stop_reason: budget_reached`（或 `user.message` 返回的 400）而非上报数字作为触及上限的信号。
- **Create-only.** Adding a budget to a session created without one is a 400. Updates accept exactly two changes: **change the cap** (the new value can be higher or lower than the old cap, but must be strictly greater than the consumed list cost, else 400: `budget.max_list_cost must be greater than the session's consumed list cost`) or **remove** (`budget: null` - the `session.updated` event carries `budget: null` rather than a separate flag). Because the consumed cost usually sits a fraction past the old cap when the session pauses, base the new value on the session's reported `usage.list_cost`, not the old `max_list_cost`. **Removal is one-way**: a removed budget can never be re-added; to keep a cap, change it instead.
  **仅创建时可设。**给创建时未设预算的会话添加预算会返回 400。更新只接受两种变更：**修改上限**（新值可高于或低于旧上限，但必须严格大于已消耗的 list cost，否则返回 400：`budget.max_list_cost must be greater than the session's consumed list cost`）或**移除**（`budget: null`——`session.updated` 事件携带 `budget: null` 而非单独的标志位）。由于会话暂停时已消耗成本通常恰好略超旧上限，新值应基于会话上报的 `usage.list_cost` 而非旧的 `max_list_cost` 来确定。**移除是单向的**：预算一旦移除就永远无法重新添加；若想保留上限，应选择修改而非移除。
- **At the cap, only settle events are accepted** - events that resolve work already in progress rather than starting new work: `user.tool_confirmation`, `user.tool_result`, `user.custom_tool_result`, `user.interrupt`. A `user.interrupt` sent while the session is paused at its budget (all threads paused at the cap) is accepted and ignored: it does not appear in the event list and changes nothing. Raise or remove the budget to continue. Anything that starts new work (e.g. `user.message`) is a 400 naming that list. No event resumes the session - only a budget change/removal does.
  **触及上限时只接受结算事件**——即处理已在进行中的工作而非开启新工作的事件：`user.tool_confirmation`、`user.tool_result`、`user.custom_tool_result`、`user.interrupt`。会话因预算暂停期间（所有线程都在上限处暂停）发送的 `user.interrupt` 会被接受但被忽略：它不会出现在事件列表中，也不改变任何状态。要继续工作，需提高或移除预算。任何开启新工作的事件（如 `user.message`）都会返回 400，并在错误信息中列出上述名单。没有任何事件能恢复会话——只有修改或移除预算可以。
  【评论】"结算事件之外一律拒绝、interrupt 被静默接受并忽略"的设计，保证了外部调用方无法通过发送事件绕过支出上限，属于消费控制层面的防滥用约束。
- **Multiagent:** one budget shared across all threads, no per-thread caps. Threads pause independently; each thread's consumption is priced at its own served model. A pending tool ask outranks the cap: a session with one thread at `requires_action` and another at `budget_reached` reports `requires_action` at the session level - answer it as usual (settle events aren't blocked).
  **多智能体：**所有线程共享一个预算，没有按线程的上限。线程独立暂停；每个线程的消耗按其各自服务的模型计价。待处理的工具请求优先于上限：当一个线程处于 `requires_action` 而另一个处于 `budget_reached` 时，会话级别上报 `requires_action`——照常应答即可（结算事件不会被阻止）。
- **Models without a list price can't be budgeted:** a budgeted create whose agent (or any roster agent, including the advisor's model) uses an unpriced model is a 400. If a running budgeted session's usage comes to include one, changing the budget is rejected - remove the budget to resume.
  **没有挂牌价的模型无法纳入预算：**创建带预算的会话时，若其智能体（或名册中任何智能体，包括 advisor 的模型）使用无价模型，会返回 400。如果一个正在运行的带预算会话的用量后来包含了无价模型，修改预算会被拒绝——需移除预算才能恢复。
- Stream behavior at the cap and the `session.usage` event: `shared/managed-agents-events.md` § Reaching a session budget.
  触及上限时的流行为与 `session.usage` 事件：`shared/managed-agents-events.md` § Reaching a session budget。
- Scheduled deployments can carry a budget too - copied onto each fired session, with different update semantics (clearable and re-addable): `shared/managed-agents-scheduled-deployments.md` § Deployment budgets.
  定时部署也可以携带预算——会复制到每个被触发的会话上，但更新语义不同（可清除、可重新添加）：`shared/managed-agents-scheduled-deployments.md` § Deployment budgets。

> **Not the same thing as Messages-API task budgets.** Session budgets are hard, dollar-denominated, platform-enforced caps on one session. `task_budget` on the Messages API is an advisory, token-denominated budget the model uses to pace itself within one agentic loop.
> **与 Messages API 的 task budget 不是一回事。**会话预算是平台对单个会话实施的、以美元计价的硬性上限。Messages API 上的 `task_budget` 则是建议性的、以 token 计价的预算，供模型在单个智能体循环内自我调节节奏。

---

## Agents / 智能体（Agents）

**This is where every Managed Agents flow begins.** The agent object is a persisted, versioned configuration - you create it once, then reference it by ID every time you start a session. No agent -> no session.

**这里是所有 Managed Agents 流程的起点。**智能体对象是持久化、带版本的配置——创建一次，之后每次启动会话都按 ID 引用它。没有智能体就没有会话。

### Agent Object / 智能体对象

The API is **flat** - `model`, `system`, `tools` etc. are top-level fields, not wrapped in an `agent:{}` sub-object.

API 是**扁平的**——`model`、`system`、`tools` 等都是顶层字段，不包在 `agent:{}` 子对象里。

| Field              | Type     | Required | Description                                        |
| ------------------ | -------- | -------- | -------------------------------------------------- |
| `name`             | string   | Yes      | Human-readable name                                |
| `model`            | string or object | Yes | Claude model ID - bare string, or `{id, speed?, effort?, inference_geo?}` |
| `system`           | string   | No       | System prompt                                      |
| `tools`            | array    | No       | Agent toolset / MCP toolset / custom tools         |
| `mcp_servers`      | array    | No       | MCP server connections                             |
| `skills`           | array    | No       | Skill references (max 20)                          |
| `description`      | string   | No       | Description of the agent                           |
| `multiagent`       | object   | No       | Coordinator roster - see `shared/managed-agents-multiagent.md` |
| `metadata`         | object   | No       | Arbitrary key-value pairs                          |

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | 是 | 人类可读的名称 |
| `model` | string or object | 是 | Claude 模型 ID——裸字符串，或 `{id, speed?, effort?, inference_geo?}` |
| `system` | string | 否 | 系统提示词 |
| `tools` | array | 否 | 智能体工具集 / MCP 工具集 / 自定义工具 |
| `mcp_servers` | array | 否 | MCP 服务器连接 |
| `skills` | array | 否 | 技能引用（最多 20 个） |
| `description` | string | 否 | 智能体的描述 |
| `multiagent` | object | 否 | 协调者名册——参见 `shared/managed-agents-multiagent.md` |
| `metadata` | object | 否 | 任意键值对 |

### Lifecycle: create once, run many, update in place / 生命周期：一次创建，多次运行，就地更新

The agent is a **persistent resource**, not a per-run parameter. The intended pattern:

智能体是**持久资源**，而不是每次运行的参数。预期的模式：

```
+- setup (once) ---------+     +- runtime (every invocation) -+
| agents.create()        |     | sessions.create(             |
|   -> store agent_id    | ---> |   agent={type:..., id: ID}   |
|     in config/env/db   |     | )                            |
+------------------------+     +------------------------------+
```

**Anti-pattern:** calling `agents.create()` at the top of every script run. This accumulates orphaned agent objects, pays create latency on every invocation, and defeats the versioning model. If you see `agents.create()` in a function that's called per-request or per-cron-tick, that's wrong - hoist it to one-time setup and persist the ID.

**反模式：**在每次脚本运行的开头调用 `agents.create()`。这会累积孤儿智能体对象、每次调用都付出创建延迟，并破坏版本化模型。如果你在按请求或按 cron 周期调用的函数里看到 `agents.create()`，那就是错的——应把它提升到一次性初始化中并持久化该 ID。

> **Recommended - define agents and environments as files and sync them with `ant apply`.** The split is **CLI for the control plane, SDK for the data plane**: agents and environments are relatively static resources you manage with `ant` (version-controlled files, synced by hand or from CI); sessions are dynamic and driven by your application through the SDK. See `shared/anthropic-cli.md` -> *Version-controlled Managed Agents resources* for the file layout, `claude-lock.json`, and the CI flow. The SDK `agents.create()` call shown elsewhere in this doc is the in-code equivalent - use it when you need to provision programmatically, but prefer files + `ant apply` for anything a human maintains.
> **推荐做法——把智能体和环境定义为文件，并用 `ant apply` 同步。**分工是**CLI 管控制平面，SDK 管数据平面**：智能体和环境是相对静态的资源，用 `ant` 管理（纳入版本控制的文件，手动或从 CI 同步）；会话是动态的，由你的应用通过 SDK 驱动。文件布局、`claude-lock.json` 与 CI 流程参见 `shared/anthropic-cli.md` -> *Version-controlled Managed Agents resources*。本文其他地方展示的 SDK `agents.create()` 调用是代码内的等价做法——需要以编程方式供给时可用它，但凡是人工维护的内容，优先使用文件 + `ant apply`。

### Effort on the agent model / 智能体模型的 effort 设置

Pass `model` as an object to set the effort level: `{"id": "claude-opus-5-5", "effort": "high"}`. `effort` accepts a level string (`low`, `medium`, `high`, `xhigh`, `max`) or an object such as `{"type": "high"}`. The create/update response echoes it in object form and fills in omitted `model` fields with their defaults.

将 `model` 作为对象传入即可设置 effort 级别：`{"id": "claude-opus-5-5", "effort": "high"}`。`effort` 接受级别字符串（`low`、`medium`、`high`、`xhigh`、`max`）或形如 `{"type": "high"}` 的对象。创建/更新响应会以对象形式回显它，并用默认值补全省略的 `model` 字段。

> Warning: **A per-session `model` override replaces the agent's `model` object in full, so the agent's own `effort` isn't carried over.** To run the session at a specific effort level, set `effort` inside the override's `model` object. A level the model doesn't support returns a 400 error, and a `model` override without `effort` runs at that model's default effort level.
> 警告：**按会话的 `model` 覆盖会完整替换智能体的 `model` 对象，因此智能体自身的 `effort` 不会被沿用。**要让会话以特定 effort 级别运行，请在覆盖的 `model` 对象内设置 `effort`。模型不支持的级别会返回 400 错误，而不带 `effort` 的 `model` 覆盖则以该模型的默认 effort 级别运行。

The same object form carries `speed` for fast mode: `{"id": "claude-opus-5-5", "speed": "fast"}`.

同样的对象形式也可携带 `speed` 以启用快速模式：`{"id": "claude-opus-5-5", "speed": "fast"}`。

### Pinning inference geography (`inference_geo`) / 固定推理地理位置（`inference_geo`）

The `model` object also takes `inference_geo` to pin the geography that serves the agent's model requests: `{"id": "claude-opus-5-5", "inference_geo": "us"}`. Accepts `"us"` or `"global"` - and unlike the Messages API, where `inference_geo` is a top-level request parameter, here it is always nested inside `model`, never top-level. When unset, each model request follows the workspace's default inference geo at the time it's served.

`model` 对象还可接受 `inference_geo`，用于固定为智能体的模型请求提供服务的地理位置：`{"id": "claude-opus-5-5", "inference_geo": "us"}`。接受 `"us"` 或 `"global"`——与 Messages API 不同（那里 `inference_geo` 是顶层请求参数），这里它总是嵌套在 `model` 内部，绝不会是顶层。未设置时，每个模型请求遵循其被服务时工作区的默认推理地理位置。

- **Validated at every stage:** the pin is checked against the workspace's `allowed_inference_geos` when the agent is saved, when a session is created from it, and on every turn the session serves. If the workspace allowlist later narrows so the pin is no longer allowed, new sessions can't be created from the agent and **running sessions refuse further turns** - pins are never grandfathered (workspaces rely on them for compliance).
  **在每个阶段都会校验：**保存智能体时、从中创建会话时、以及会话服务的每一轮，都会对照工作区的 `allowed_inference_geos` 检查该固定值。如果工作区白名单后来收窄、使该固定值不再被允许，则无法再从该智能体创建新会话，且**正在运行的会话会拒绝后续轮次**——固定值永不豁免（工作区依赖它满足合规要求）。
- Setting `inference_geo` on a model that doesn't support geographic inference pinning returns a 400.
  在不支持地理推理固定的模型上设置 `inference_geo` 会返回 400。
- **Fixed for a session's lifetime** - the pin can't change mid-session. Set it on the agent, or set/clear it for one session with a `model` override at session create (see § Override agent configuration for a session).
  **在会话生命周期内固定**——该固定值无法在会话中途更改。可在智能体上设置，或在创建会话时通过 `model` 覆盖为单个会话设置/清除（参见 § Override agent configuration for a session）。
- **Multiagent rosters must be geo-uniform:** the coordinator's pin and every roster member's must all be the same value or all be unset - see `shared/managed-agents-multiagent.md`.
  **多智能体名册必须地理一致：**协调者的固定值与每个名册成员的固定值必须全部相同或全部未设置——参见 `shared/managed-agents-multiagent.md`。
- Like `effort`, an `inference_geo` inside a per-session `model` override **is applied** - and because overrides replace the `model` object in full, an override that *omits* `inference_geo` clears the agent's pin for that session.
  与 `effort` 一样，按会话 `model` 覆盖中的 `inference_geo` **会被应用**——并且由于覆盖会完整替换 `model` 对象，*省略* `inference_geo` 的覆盖会在该会话中清除智能体的固定值。

### Versioning / 版本管理

Each `POST /v1/agents/{id}` (update) creates a new immutable version - a sequential integer, starting at 1 and incrementing on each update. The agent's history is append-only - you can't edit a past version.

每次 `POST /v1/agents/{id}`（更新）都会创建一个新的不可变版本——一个顺序整数，从 1 开始，每次更新递增。智能体的历史是只追加的——你无法编辑过去的版本。

**`version` on update is optional.** Supply it for optimistic concurrency, or omit it to apply the update unconditionally:

**更新时 `version` 是可选的。**提供它可实现乐观并发控制，或省略它以无条件应用更新：

| `version` | Behavior | Fits |
|---|---|---|
| Supplied (must be >= 1) | 409 if it doesn't match the agent's current version - **even when the fields you send already equal the stored values**. Re-read and retry. | Interactive callers; the recommended default |
| Omitted | Applies unconditionally. The most recent update silently replaces any concurrent one, with no error to either caller. | Hand-rolled sync loops - e.g. a CI script pushing checked-in agent definitions with `agents.update()`, where the loop owns the agent |

| `version` | 行为 | 适用场景 |
|---|---|---|
| 提供了（必须 >= 1） | 若与智能体当前版本不匹配则返回 409——**即使你发送的字段值已与存储值相等**。重新读取并重试。 | 交互式调用方；推荐的默认选择 |
| 省略 | 无条件应用。最近的更新会静默替换任何并发的更新，双方调用方都不会收到错误。 | 手写的同步循环——例如用 `agents.update()` 推送签入的智能体定义的 CI 脚本，此时该循环拥有这个智能体 |

**Update semantics.** Omitted fields are preserved. Scalar fields (`model`, `system`, `name`, `description`) are replaced; `system` and `description` can be cleared with `null`, while `model` and `name` cannot. Array fields (`tools`, `mcp_servers`, `skills`) are replaced wholesale - `null` or `[]` clears them. **`effort` is the sole exception inside a `model` object you supply:** if the model `id` is unchanged, omitting `effort` leaves the stored level alone; if you change the `id`, an omitted `effort` resets to the new model's default. Other `model` fields are replaced along with the object - **supplying `model` without `inference_geo` clears the agent's inference geo pin.**

**更新语义。**省略的字段保持不变。标量字段（`model`、`system`、`name`、`description`）会被替换；`system` 和 `description` 可用 `null` 清除，`model` 和 `name` 则不行。数组字段（`tools`、`mcp_servers`、`skills`）会被整体替换——`null` 或 `[]` 会将其清空。**在你提供的 `model` 对象内部，`effort` 是唯一的例外：**如果模型 `id` 未变，省略 `effort` 会保留已存储的级别；如果你更改了 `id`，省略的 `effort` 会重置为新模型的默认值。其他 `model` 字段会随对象一起被替换——**提供不带 `inference_geo` 的 `model` 会清除智能体的推理地理固定值。**

**Why version:**

**为什么要版本化：**

- **Reproducibility** - pin a session to a known-good config: `{type: "agent", id, version: 3}`
  **可复现性**——把会话固定到一个已知良好的配置：`{type: "agent", id, version: 3}`
- **Safe iteration** - update the agent without breaking sessions already running on the old version
  **安全迭代**——更新智能体而不会破坏仍在旧版本上运行的会话
- **Rollback** - if a new system prompt regresses, pin new sessions back to the prior version while you debug
  **回滚**——如果新的系统提示词出现退化，可在调试期间把新会话固定回先前版本

**`version` is optional.** Omit it (or use the string shorthand `agent="agent_abc123"`) to get the latest version at session-creation time. Pass it explicitly (`{type: "agent", id, version: N}`) to pin for reproducibility.

**`version` 是可选的。**省略它（或使用字符串简写 `agent="agent_abc123"`）可在创建会话时获取最新版本。显式传入它（`{type: "agent", id, version: N}`）可固定版本以保证可复现性。

**Getting the version to pin:** `agents.create()` and `agents.update()` both return `version` in the response. Store it alongside `agent_id`. To fetch the current latest for an existing agent: `GET /v1/agents/{id}` -> `.version`.

**获取要固定的版本：**`agents.create()` 和 `agents.update()` 都会在响应中返回 `version`。把它与 `agent_id` 一起保存。要获取既有智能体当前的最新版本：`GET /v1/agents/{id}` -> `.version`。

**When to update vs create new:** Update (`POST /v1/agents/{id}`) when it's conceptually the same agent with tweaked behavior (better prompt, extra tool). Create a new agent when it's a different persona/purpose. Rule of thumb: if you'd give it the same `name`, update.

**何时更新 vs 创建新的：**当概念上是同一个智能体、只是行为微调（更好的提示词、额外的工具）时，选择更新（`POST /v1/agents/{id}`）。当人设/用途不同时，创建新智能体。经验法则：如果你会给它相同的 `name`，就更新。

### Agent Endpoints / 智能体端点

| Operation        | Method   | Path                                  |
| ---------------- | -------- | ------------------------------------- |
| Create           | `POST`   | `/v1/agents`                          |
| List             | `GET`    | `/v1/agents`                          |
| Get              | `GET`    | `/v1/agents/{id}`                     |
| Update           | `POST`   | `/v1/agents/{id}`                     |
| Archive          | `POST`   | `/v1/agents/{id}/archive`             |

| 操作 | 方法 | 路径 |
| --- | --- | --- |
| 创建 | `POST` | `/v1/agents` |
| 列表 | `GET` | `/v1/agents` |
| 获取 | `GET` | `/v1/agents/{id}` |
| 更新 | `POST` | `/v1/agents/{id}` |
| 归档 | `POST` | `/v1/agents/{id}/archive` |

> Warning: **Archive is permanent.** Archiving makes the agent read-only: existing sessions continue to run, but **new sessions cannot reference it**, and there is no unarchive. Since agents have no `delete`, this is the terminal lifecycle state. Never archive a production agent as routine cleanup - confirm with the user first.
> 警告：**归档是永久的。**归档使智能体变为只读：现有会话继续运行，但**新会话无法再引用它**，且没有取消归档。由于智能体没有 `delete`，这就是生命周期的终态。绝不要把归档生产环境智能体当作例行清理——先与用户确认。

【评论】"先与用户确认"这类面向代码生成智能体的行为指令被直接写进 API 参考文档，因为该操作不可逆且没有删除或恢复的兜底手段。

### Using an Agent in a Session / 在会话中使用智能体

Reference the agent by string ID (latest version) or by object with an explicit version:

通过字符串 ID（最新版本）或带显式版本的对象来引用智能体：

```python
# String shorthand - uses the agent's latest version
session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=environment_id,
)

# Or pin to a specific version (int)
session = client.beta.sessions.create(
    agent={"type": "agent", "id": agent.id, "version": agent.version},
    environment_id=environment_id,
)
```

### Override agent configuration for a session / 为单个会话覆盖智能体配置

The third `agent` form, `agent_with_overrides`, replaces parts of the agent's configuration for **a single session** - try a different model or grant an extra tool without versioning the agent. Pass `id` (and optionally `version`; omitted = latest, same default as the other two forms) plus any of `model`, `system`, `tools`, `mcp_servers`, `skills`:

第三种 `agent` 形式 `agent_with_overrides` 会为**单个会话**替换智能体配置的一部分——无需对智能体进行版本化即可尝试不同模型或授予额外工具。传入 `id`（以及可选的 `version`；省略即最新版本，与其他两种形式的默认一致），外加 `model`、`system`、`tools`、`mcp_servers`、`skills` 中的任意若干项：

```python
session = client.beta.sessions.create(
    agent={
        "type": "agent_with_overrides",
        "id": agent.id,
        "model": "claude-opus-5-5",   # replace the agent's model for this session
        "system": None,           # clear the system prompt for this session
    },
    environment_id=environment_id,
)
```

Each overridable field follows tri-state rules:

每个可覆盖字段都遵循三态规则：

- **Omit** -> the session inherits the value from the referenced agent version.
  **省略** -> 会话从被引用的智能体版本继承该值。
- **`null` (or `[]` for list fields)** -> the session runs with that field cleared. Applies in full to `system` and `skills`. Three exceptions: `model` is never clearable (`model: null` -> 400 `agent_model_required`); clearing `tools` returns 400 when the session's effective `skills` is non-empty (skills require the `read` tool); and clearing `mcp_servers` returns 400 when the effective `tools` still contains an `mcp_toolset` referencing one of the agent's servers - override `tools` in the same request to drop those entries, then clear `mcp_servers`.
  **`null`（列表字段为 `[]`）** -> 会话在清除该字段的状态下运行。完全适用于 `system` 和 `skills`。有三个例外：`model` 永远不可清除（`model: null` -> 400 `agent_model_required`）；当会话的有效 `skills` 非空时清除 `tools` 会返回 400（技能需要 `read` 工具）；当有效 `tools` 仍包含引用智能体某个服务器的 `mcp_toolset` 时清除 `mcp_servers` 会返回 400——先在同一请求中覆盖 `tools` 以移除那些条目，然后再清除 `mcp_servers`。
- **A value** -> replaces the agent's value **in full**. Overrides never merge - a `tools` override must list every tool the session should have. A `model` override also replaces the agent's `model` object in full: the agent's own `effort` isn't carried over, so set `effort` inside the override's `model` object to run the session at a specific effort level (a level the model doesn't support returns a 400 error, and a `model` override without `effort` runs at that model's default effort level). An `inference_geo` inside a `model` override **is** applied - and because the object is replaced in full, an override that omits it clears the agent's pin, so the session follows the workspace's default inference geo. The overridden value is validated against the workspace's `allowed_inference_geos` at session create.
  **提供一个值** -> 完整替换智能体的值。覆盖从不合并——`tools` 覆盖必须列出会话应拥有的每一个工具。`model` 覆盖同样会完整替换智能体的 `model` 对象：智能体自身的 `effort` 不会被沿用，因此要让会话以特定 effort 级别运行，需在覆盖的 `model` 对象内设置 `effort`（模型不支持的级别返回 400 错误，不带 `effort` 的 `model` 覆盖则以该模型的默认 effort 级别运行）。`model` 覆盖内的 `inference_geo` **会**被应用——并且由于对象被完整替换，省略它的覆盖会清除智能体的固定值，会话将遵循工作区的默认推理地理位置。被覆盖的值会在创建会话时对照工作区的 `allowed_inference_geos` 进行校验。

Overrides are session-local: they do **not** modify the agent resource or create a new agent version. The response's `agent` object reflects the post-override configuration, while its `id` and `version` still identify the base agent - so you can trace a session back to its base. In multiagent sessions, overrides apply to the coordinator and its `{type: "self"}` copies; roster agents referenced by ID always use their own as-created configuration (see `shared/managed-agents-multiagent.md`).

覆盖是会话本地的：它们**不会**修改智能体资源，也不会创建新的智能体版本。响应中的 `agent` 对象反映覆盖后的配置，而其 `id` 和 `version` 仍标识基础智能体——因此你可以把会话追溯回其基础智能体。在多智能体会话中，覆盖适用于协调者及其 `{type: "self"}` 副本；按 ID 引用的名册智能体始终使用各自创建时的配置（参见 `shared/managed-agents-multiagent.md`）。

### Updating the agent configuration mid-session / 会话中途更新智能体配置

`sessions.update()` can change `agent.tools` and `agent.mcp_servers` (including permission policies and the per-tool web settings - `allowed_domains` / `blocked_domains` etc., see `shared/managed-agents-tools.md` § Web search & web fetch settings) on an **existing** session. Updated domain lists apply to the rest of the session. This is a **session-local override** - it does not create a new agent version and does not propagate back to the agent object. The provided arrays are **full replacements**; to append one tool, `GET` the session, modify, and `POST` back. The session must be `idle` - interrupt first if running. `vault_ids` is **create-only**: the update param exists in the SDK but is rejected by the API ("Not yet supported") - attach vaults when you create the session.

`sessions.update()` 可以在**现有**会话上更改 `agent.tools` 与 `agent.mcp_servers`（包括权限策略与每工具的网页设置——`allowed_domains` / `blocked_domains` 等，参见 `shared/managed-agents-tools.md` § Web search & web fetch settings）。更新后的域名列表适用于会话的剩余部分。这是**会话本地的覆盖**——它不会创建新的智能体版本，也不会传播回智能体对象。提供的数组是**完整替换**；要追加一个工具，需 `GET` 该会话、修改后再 `POST` 回去。会话必须处于 `idle`——若正在运行需先中断。`vault_ids` 是**仅创建时可设**：该更新参数在 SDK 中存在，但会被 API 拒绝（"Not yet supported"）——请在创建会话时附加 vault。

Among the agent-configuration fields, only `tools` and `mcp_servers` can change after a session is created - to run with a `model`, `system`, or `skills` other than the agent's values, use `agent_with_overrides` at create time (above). (`title`, `metadata`, and `budget` have their own session-update paths - see § Session operations / § Session budgets.) The agent's model configuration - including its `inference_geo` pin - and its configured `system` field are fixed for the session's lifetime; you can still **append system-level context between turns** by sending a `system.message` event (see `shared/managed-agents-events.md` § Adding system context mid-session).

在智能体配置字段中，只有 `tools` 和 `mcp_servers` 可以在会话创建后更改——若要以不同于智能体取值的 `model`、`system` 或 `skills` 运行，请在创建时使用 `agent_with_overrides`（见上文）。（`title`、`metadata` 和 `budget` 有各自的会话更新路径——参见 § Session operations / § Session budgets。）智能体的模型配置——包括其 `inference_geo` 固定值——及其配置的 `system` 字段在会话生命周期内固定不变；你仍可通过发送 `system.message` 事件**在轮次之间追加系统级上下文**（参见 `shared/managed-agents-events.md` § Adding system context mid-session）。

```python
client.beta.sessions.update(
    session.id,
    agent={
        "tools": [
            {"type": "agent_toolset_20260401"},
            {"type": "mcp_toolset", "mcp_server_name": "linear"},
        ],
        "mcp_servers": [{"type": "url", "name": "linear", "url": "https://mcp.linear.app/sse"}],
    },
)
```
