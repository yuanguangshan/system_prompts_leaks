<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Overview / 托管代理 - 概览

Managed Agents provisions a container per session as the agent's workspace. The agent loop runs on Anthropic's orchestration layer; the container is where the agent's *tools* execute - bash commands, file operations, code. You create a persisted **Agent** config (model, system prompt, tools, MCP servers, skills), then start **Sessions** that reference it. The session streams events back to you; you send user messages and tool results in.

Managed Agents 为每个会话预配一个容器作为代理的工作区。代理循环运行在 Anthropic 的编排层上；容器则是代理*工具*执行的地方——bash 命令、文件操作、代码。你先创建持久化的 **Agent** 配置（模型、系统提示词、工具、MCP 服务器、技能），然后启动引用它的 **Session（会话）**。会话把事件流式传回给你；你把用户消息和工具结果发进去。

## Warning: THE MANDATORY FLOW: Agent (once) -> Session (every run) / 警告：强制流程：Agent（一次）-> Session（每次运行）

**Why agents are separate objects: versioning.** An agent is a persisted, versioned config - every update creates a new immutable version, and sessions pin to a version at creation time. This lets you iterate on the agent (tweak the prompt, add a tool) without breaking sessions already running, roll back if a change regresses, and A/B test versions side-by-side. None of that works if you `agents.create()` fresh on every run.

**为什么 Agent 是独立对象：版本化。** Agent 是持久化、带版本的配置——每次更新都会创建一个新的不可变版本，而会话在创建时即固定（pin）到某个版本。这让你可以在不破坏已运行会话的情况下迭代代理（调整提示词、添加工具），在变更造成退化时回滚，并对多个版本做并排 A/B 测试。如果你每次运行都新调用 `agents.create()`，这些好处统统不复存在。

Every session references a pre-created `/v1/agents` object. Create the agent once, store the ID, and reuse it across runs.

每个会话都引用一个预先创建的 `/v1/agents` 对象。代理只创建一次，保存 ID，并在多次运行间复用。

| Step | Call | Frequency |
|---|---|---|
| 1 | `POST /v1/agents` - `model`, `system`, `tools`, `mcp_servers`, `skills` live here | **ONCE.** Store `agent.id` **and** `agent.version`. |
| 2 | `POST /v1/sessions` - `agent: "agent_abc123"` or `{type: "agent", id, version}` | **Every run.** String shorthand uses latest version. |

| 步骤 | 调用 | 频率 |
|---|---|---|
| 1 | `POST /v1/agents` - `model`、`system`、`tools`、`mcp_servers`、`skills` 存放在这里 | **一次。** 保存 `agent.id` 和 `agent.version`。 |
| 2 | `POST /v1/sessions` - `agent: "agent_abc123"` 或 `{type: "agent", id, version}` | **每次运行。** 字符串简写形式使用最新版本。 |

If you're about to write `sessions.create()` with `model`, `system`, or `tools` on the session body - **stop**. Those fields live on `agents.create()`. The session takes a *pointer* only.

如果你正准备在 `sessions.create()` 的会话请求体里写 `model`、`system` 或 `tools`——**停**。这些字段属于 `agents.create()`。会话只接受*指针*。

**When generating code, separate setup from runtime.** `agents.create()` belongs in a setup script (or a guarded `if agent_id is None:` block), not at the top of the hot path. If the user's code calls `agents.create()` on every invocation, they're accumulating orphaned agents and paying the create latency for nothing. The correct shape is: define the agent as a version-controlled file and sync it with `ant apply`, which records the ID in `claude-lock.json` (see `shared/anthropic-cli.md`) - or use a guarded setup script that persists the returned ID (config file, env var, secrets manager) - and have every run load the ID and call `sessions.create()`.

**生成代码时，把设置与运行时分开。** `agents.create()` 应属于安装脚本（或有守卫的 `if agent_id is None:` 代码块），而不是热路径的开头。如果用户的代码每次调用都执行 `agents.create()`，他们就会不断累积孤儿代理，并白白支付创建延迟。正确的形态是：把代理定义为一个受版本控制的文件并用 `ant apply` 同步（ID 记录在 `claude-lock.json` 中，参见 `shared/anthropic-cli.md`）——或使用能持久化返回 ID 的带守卫安装脚本（配置文件、环境变量、密钥管理器）——然后让每次运行加载 ID 并调用 `sessions.create()`。

**To change the agent's behavior, use `POST /v1/agents/{id}` - don't create a new one.** (For an agent managed with `ant apply`, edit its file and re-run instead - an update made outside the files makes the next `ant apply` refuse to run.) Each update bumps the version; running sessions keep their pinned version, new sessions get the latest (or pin explicitly via `{type: "agent", id, version}`). See `shared/managed-agents-core.md` -> Agents -> Versioning. To change `tools`/`mcp_servers` on **one running session** without touching the agent object, use `sessions.update()` (`vault_ids` attaches at session create only) - see `shared/managed-agents-core.md` -> Updating the agent configuration mid-session.

**要改变代理的行为，用 `POST /v1/agents/{id}`——不要新建。**（对用 `ant apply` 管理的代理，应改为编辑其文件并重新运行——在文件之外做的更新会让下一次 `ant apply` 拒绝执行。）每次更新都会递增版本；运行中的会话保持其固定的版本，新会话拿到最新版本（或通过 `{type: "agent", id, version}` 显式固定）。参见 `shared/managed-agents-core.md` -> Agents -> Versioning。要在**不触碰代理对象**的前提下更改**某个运行中会话**的 `tools`/`mcp_servers`，使用 `sessions.update()`（`vault_ids` 只能在会话创建时附加）——参见 `shared/managed-agents-core.md` -> Updating the agent configuration mid-session。

## Beta Headers / Beta 请求头

Managed Agents is in beta. The SDK sets required beta headers automatically:

Managed Agents 处于 beta 阶段。SDK 会自动设置所需的 beta 请求头：

| Beta Header                    | What it enables                                      |
| ------------------------------ | ---------------------------------------------------- |
| `managed-agents-2026-04-01`    | Agents, Environments, Sessions, Events, Session Resources, Session Threads, Outcomes, Multiagent, Vaults, Credentials, Deployments |
| `agent-memory-2026-07-22`      | Memory Stores (replaces `managed-agents-2026-04-01` on memory store endpoints) |

| Beta 请求头 | 启用的功能 |
| ------------------------------ | ---------------------------------------------------- |
| `managed-agents-2026-04-01` | Agents、Environments、Sessions、Events、Session Resources、Session Threads、Outcomes、Multiagent、Vaults、Credentials、Deployments |
| `agent-memory-2026-07-22` | Memory Stores（在记忆存储端点上取代 `managed-agents-2026-04-01`） |

**Which beta header goes where:** The SDK sets `managed-agents-2026-04-01` automatically on `client.beta.{agents,environments,sessions,vaults,deployments,deployment_runs}.*` calls and `agent-memory-2026-07-22` on `client.beta.memory_stores.*` calls. Don't add `managed-agents-2026-04-01` to a memory store call: sending both headers on a memory store request returns a 400 (attaching a memory store to a session is a session call and still uses `managed-agents-2026-04-01`). The Files and Skills APIs are out of beta and need no beta header; requests that still send `files-api-2025-04-14` or `skills-2025-10-02` keep working but get the old beta response shapes. **Exception - session-scoped file listing:** filtering `files.list` by `scope_id` requires `managed-agents-2026-04-01`, which `client.beta.files` does not add, so pass `betas: ["managed-agents-2026-04-01"]` explicitly on `client.beta.files.list({scope_id: session.id})` (on raw HTTP, send `anthropic-beta: managed-agents-2026-04-01`; in the `ant` CLI, add `--beta managed-agents-2026-04-01` to `ant beta:files list --scope-id`). See `shared/managed-agents-environments.md` -> Session outputs.

**哪个 beta 请求头用在哪里：** SDK 在 `client.beta.{agents,environments,sessions,vaults,deployments,deployment_runs}.*` 调用上自动设置 `managed-agents-2026-04-01`，在 `client.beta.memory_stores.*` 调用上自动设置 `agent-memory-2026-07-22`。不要把 `managed-agents-2026-04-01` 加到记忆存储调用上：在记忆存储请求上同时发送两个请求头会返回 400（把记忆存储挂到会话上属于会话调用，仍然使用 `managed-agents-2026-04-01`）。Files 和 Skills API 已结束 beta，不需要 beta 请求头；仍发送 `files-api-2025-04-14` 或 `skills-2025-10-02` 的请求继续可用，但会拿到旧的 beta 响应结构。**例外——会话级文件列表：** 按 `scope_id` 过滤 `files.list` 需要 `managed-agents-2026-04-01`，而 `client.beta.files` 不会自动添加该请求头，因此需在 `client.beta.files.list({scope_id: session.id})` 上显式传入 `betas: ["managed-agents-2026-04-01"]`（原始 HTTP 时发送 `anthropic-beta: managed-agents-2026-04-01`；`ant` CLI 中在 `ant beta:files list --scope-id` 后加 `--beta managed-agents-2026-04-01`）。参见 `shared/managed-agents-environments.md` -> Session outputs。


## Reading Guide / 阅读指南

| User wants to...                       | Read these files                                        |
| -------------------------------------- | ------------------------------------------------------- |
| **Get started from scratch / "help me set up an agent"** | `shared/managed-agents-onboarding.md` - guided interview (WHERE->WHO->WHAT->WATCH), then emit code |
| Understand how the API works           | `shared/managed-agents-core.md`                         |
| See the full endpoint reference        | `shared/managed-agents-api-reference.md`                |
| **Create an agent** (required first step) | `shared/managed-agents-core.md` (Agents section) + language file |
| Update/version an agent                | `shared/managed-agents-core.md` (Agents -> Versioning) - update, don't re-create |
| Create a session                       | `shared/managed-agents-core.md` + `{lang}/managed-agents/README.md` (cURL/C#: `curl/managed-agents.md`) |
| Configure tools and permissions        | `shared/managed-agents-tools.md`                        |
| Restrict which sites `web_search` / `web_fetch` can reach; localize search; cap fetched content | `shared/managed-agents-tools.md` (§ Web search & web fetch settings) - `allowed_domains` / `blocked_domains` / `user_location` / `max_content_tokens` on the toolset `configs` entry; **not** the environment's `networking` |
| Set up MCP servers                     | `shared/managed-agents-tools.md` (MCP Servers section)  |
| Stream events / handle tool_use        | `shared/managed-agents-events.md` + language file       |
| Get notified of session state changes via webhook (no polling) | `shared/managed-agents-webhooks.md` - Console-registered endpoint, HMAC verify, thin payload + fetch |
| Define an outcome / rubric-graded iterate loop | `shared/managed-agents-outcomes.md` - `user.define_outcome` event, grader, `span.outcome_evaluation_*` events |
| Coordinate multiple agents / subagents / threads | `shared/managed-agents-multiagent.md` - `multiagent: {type: "coordinator", agents: [...]}` on the agent, session threads, cross-posted tool confirmations |
| Set up environments                    | `shared/managed-agents-environments.md` + language file |
| Run tool execution in your own infra / VPC (self-hosted sandbox) | `shared/managed-agents-self-hosted-sandboxes.md` - `config:{type:"self_hosted"}`, `ANTHROPIC_ENVIRONMENT_KEY`, `EnvironmentWorker.run()` / `ant beta:worker poll` |
| Upload files / attach repos            | `shared/managed-agents-environments.md` (Resources)     |
| Give agents persistent memory across sessions | `shared/managed-agents-memory.md` - memory stores, `memory_store` session resource, preconditions, versions/redact. On self-hosted sandboxes: `shared/managed-agents-self-hosted-sandboxes.md` § Memory stores (SDK worker syncs a local copy) |
| Inspect a session without code (transcript, per-tool stats, cost, threads) | `shared/managed-agents-events.md` - Console session viewer note; deep link `?event={event_id}` |
| Keep agents/environments/skills as version-controlled files (`ant apply`); drive the API from the shell | `shared/anthropic-cli.md` - `ant apply`, `claude-lock.json`, `--transform`, `@file` inlining |
| Store credentials (MCP auth, API keys for CLIs/SDKs) | `shared/managed-agents-tools.md` (Vaults section) - `mcp_oauth` / `static_bearer` / `environment_variable` |
| Call a non-MCP API / CLI that needs a secret | `shared/managed-agents-tools.md` (Vaults section) - `environment_variable` credential, substituted at egress. If that doesn't fit (e.g. self-hosted sandboxes), `shared/managed-agents-client-patterns.md` Pattern 9 keeps the secret host-side via a custom tool |
| Run an agent on a recurring cron schedule | `shared/managed-agents-scheduled-deployments.md` - deployments, deployment runs, pause/auto-pause |
| Cap a session's spend with a hard dollar budget | `shared/managed-agents-core.md` (§ Session budgets) - `budget` at session create, `budget_reached` pause, change/remove to resume. Deployments: `shared/managed-agents-scheduled-deployments.md` § Deployment budgets |
| Pin where model inference runs (data residency) | `shared/managed-agents-core.md` (§ Pinning inference geography) - `model.inference_geo` on the agent, per-session override, roster uniformity |
| Load skills from the codebase instead of uploading | `shared/managed-agents-tools.md` (§ Skills from a GitHub repository) - root `.claude/skills` discovery at session start |
| Give the session an advisor to consult mid-turn | `shared/managed-agents-multiagent.md` (§ Advisor) - `{type: "advisor", model}` roster entry, consultation threads, plaintext vs redacted delivery |

| 用户想... | 阅读这些文件 |
| -------------------------------------- | ------------------------------------------------------- |
| **从零开始 / "帮我搭一个代理"** | `shared/managed-agents-onboarding.md` - 引导式访谈（WHERE->WHO->WHAT->WATCH），然后生成代码 |
| 了解 API 的工作原理 | `shared/managed-agents-core.md` |
| 查看完整的端点参考 | `shared/managed-agents-api-reference.md` |
| **创建代理**（必需的第一步） | `shared/managed-agents-core.md`（Agents 部分）+ 语言文件 |
| 更新/版本化代理 | `shared/managed-agents-core.md`（Agents -> Versioning）——更新，而不是重建 |
| 创建会话 | `shared/managed-agents-core.md` + `{lang}/managed-agents/README.md`（cURL/C#：`curl/managed-agents.md`） |
| 配置工具与权限 | `shared/managed-agents-tools.md` |
| 限制 `web_search` / `web_fetch` 可访问的站点；本地化搜索；限制抓取内容量 | `shared/managed-agents-tools.md`（§ Web search & web fetch settings）——toolset 的 `configs` 条目上的 `allowed_domains` / `blocked_domains` / `user_location` / `max_content_tokens`；**而不是**环境的 `networking` |
| 设置 MCP 服务器 | `shared/managed-agents-tools.md`（MCP Servers 部分） |
| 流式接收事件 / 处理 tool_use | `shared/managed-agents-events.md` + 语言文件 |
| 通过 webhook 接收会话状态变化通知（无需轮询） | `shared/managed-agents-webhooks.md` - Console 注册的端点、HMAC 校验、精简载荷 + 回取 |
| 定义成果 / 按评分标准迭代的循环 | `shared/managed-agents-outcomes.md` - `user.define_outcome` 事件、评分器、`span.outcome_evaluation_*` 事件 |
| 协调多个代理 / 子代理 / 线程 | `shared/managed-agents-multiagent.md` - 代理上的 `multiagent: {type: "coordinator", agents: [...]}`、会话线程、交叉转发的工具确认 |
| 设置环境 | `shared/managed-agents-environments.md` + 语言文件 |
| 在你自己的基础设施 / VPC 中运行工具执行（自托管沙箱） | `shared/managed-agents-self-hosted-sandboxes.md` - `config:{type:"self_hosted"}`、`ANTHROPIC_ENVIRONMENT_KEY`、`EnvironmentWorker.run()` / `ant beta:worker poll` |
| 上传文件 / 挂载仓库 | `shared/managed-agents-environments.md`（Resources） |
| 让代理拥有跨会话的持久记忆 | `shared/managed-agents-memory.md` - 记忆存储、`memory_store` 会话资源、前置条件、版本/脱敏。自托管沙箱场景：`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores（SDK worker 同步本地副本） |
| 无需代码查看会话（记录、逐工具统计、成本、线程） | `shared/managed-agents-events.md` - Console 会话查看器说明；深链 `?event={event_id}` |
| 把代理/环境/技能保存为受版本控制的文件（`ant apply`）；从 shell 驱动 API | `shared/anthropic-cli.md` - `ant apply`、`claude-lock.json`、`--transform`、`@file` 内联 |
| 存储凭据（MCP 鉴权、CLI/SDK 的 API key） | `shared/managed-agents-tools.md`（Vaults 部分）- `mcp_oauth` / `static_bearer` / `environment_variable` |
| 调用需要密钥的非 MCP API / CLI | `shared/managed-agents-tools.md`（Vaults 部分）- `environment_variable` 凭据，出站时替换。若不适用（如自托管沙箱），`shared/managed-agents-client-patterns.md` Pattern 9 通过自定义工具把密钥留在宿主侧 |
| 按周期性 cron 计划运行代理 | `shared/managed-agents-scheduled-deployments.md` - deployments、deployment runs、暂停/自动暂停 |
| 用硬性金额预算限制会话支出 | `shared/managed-agents-core.md`（§ Session budgets）- 会话创建时的 `budget`、`budget_reached` 暂停、变更/移除预算以恢复。部署场景：`shared/managed-agents-scheduled-deployments.md` § Deployment budgets |
| 固定模型推理的运行地点（数据驻留） | `shared/managed-agents-core.md`（§ Pinning inference geography）- 代理上的 `model.inference_geo`、按会话覆盖、成员一致性 |
| 从代码库加载技能而不是上传 | `shared/managed-agents-tools.md`（§ Skills from a GitHub repository）- 会话启动时发现根目录 `.claude/skills` |
| 给会话配置一个可在回合中途咨询的顾问 | `shared/managed-agents-multiagent.md`（§ Advisor）- `{type: "advisor", model}` 成员条目、咨询线程、明文与脱敏交付 |

## Common Pitfalls / 常见陷阱

- **Agent FIRST, then session - NO EXCEPTIONS** - the session's `agent` field accepts **only** a string ID or `{type: "agent", id, version}`. `model`, `system`, `tools`, `mcp_servers`, `skills` are **top-level fields on `POST /v1/agents`**, never on `sessions.create()`. If the user hasn't created an agent, that is step zero of every example.
  - **先有 Agent，再有会话——无一例外**——会话的 `agent` 字段**只**接受字符串 ID 或 `{type: "agent", id, version}`。`model`、`system`、`tools`、`mcp_servers`、`skills` 是 `POST /v1/agents` 的**顶层字段**，绝不出现在 `sessions.create()` 上。如果用户还没创建代理，那每个示例的第零步都是创建代理。
- **Agent ONCE, not every run** - `agents.create()` is a setup step. Store the returned `agent_id` and reuse it; don't call `agents.create()` at the top of your hot path. If the agent's config needs to change, `POST /v1/agents/{id}` - each update creates a new version, and sessions can pin to a specific version for reproducibility.
  - **Agent 只创建一次，而不是每次运行**——`agents.create()` 是设置步骤。保存返回的 `agent_id` 并复用；不要在热路径开头调用 `agents.create()`。如果代理配置需要变更，用 `POST /v1/agents/{id}`——每次更新创建一个新版本，会话可以固定到特定版本以保证可复现性。
- **MCP auth goes through vaults** - the agent's `mcp_servers` array declares `{type, name, url}` only (no auth). Credentials live in vaults (`client.beta.vaults.credentials.create`) and attach to sessions via `vault_ids`. Anthropic auto-refreshes OAuth tokens using the stored refresh token. Vaults also hold `environment_variable` credentials for non-MCP services (CLIs, SDKs, direct API calls) - substituted at egress, never visible in the sandbox.
  - **MCP 鉴权经由 vault（保险库）**——代理的 `mcp_servers` 数组只声明 `{type, name, url}`（不含鉴权）。凭据存放在 vault 中（`client.beta.vaults.credentials.create`），并通过 `vault_ids` 附加到会话。Anthropic 使用存储的 refresh token 自动刷新 OAuth token。vault 还保存非 MCP 服务（CLI、SDK、直接 API 调用）使用的 `environment_variable` 凭据——出站时替换，在沙箱中永不可见。
- **Reconcile resources before the first run** - a session with a clear ask but a missing tool, credential, data mount, or context will discover the gap mid-run, then flail and give up. Before creating the session, check that every action in the task maps to a configured tool/MCP server, every MCP server has a vault credential, and every referenced file/host is mounted/reachable. When helping a user set one up, run the reconciliation in `shared/managed-agents-onboarding.md` -> §3 Pre-flight viability check.
  - **首次运行前先核对资源**——一个诉求明确但缺少工具、凭据、数据挂载或上下文的会话，会在运行中途才发现缺口，然后挣扎并放弃。创建会话之前，确认任务中的每个动作都映射到已配置的工具/MCP 服务器，每个 MCP 服务器都有 vault 凭据，每个被引用的文件/主机都已挂载/可达。帮助用户搭建时，执行 `shared/managed-agents-onboarding.md` -> §3 Pre-flight viability check 中的核对。
- **Stream to get events** - `GET /v1/sessions/{id}/events/stream` is the primary way to receive agent output in real-time.
  - **用流接收事件**——`GET /v1/sessions/{id}/events/stream` 是实时接收代理输出的主要方式。
- **SSE stream has no replay - reconnect with consolidation** - if the stream drops while a `agent.tool_use`, `agent.mcp_tool_use`, or `agent.custom_tool_use` is pending resolution (`user.tool_confirmation` for the first two, `user.custom_tool_result` for the last one), the session deadlocks (client disconnects -> session idles -> reconnect happens -> no client resolution happens). On every (re)connect: open stream with `GET /v1/sessions/{id}/events/stream` , fetch `GET /v1/sessions/{id}/events`, dedupe by event ID, then proceed. See `shared/managed-agents-events.md` -> Reconnecting after a dropped stream.
  - **SSE 流没有重放——重连时需合并去重**——如果流在某个 `agent.tool_use`、`agent.mcp_tool_use` 或 `agent.custom_tool_use` 待解决期间断开（前两者需 `user.tool_confirmation`，最后者需 `user.custom_tool_result`），会话会死锁（客户端断开 -> 会话空闲 -> 重新连接 -> 始终没有客户端解决结果）。每次（重）连接时：用 `GET /v1/sessions/{id}/events/stream` 打开流，抓取 `GET /v1/sessions/{id}/events`，按事件 ID 去重，然后继续。参见 `shared/managed-agents-events.md` -> Reconnecting after a dropped stream。
  
  【评论】SSE 无重放意味着断线重连的客户端必须自行从完整事件列表补齐状态，否则待确认的工具调用会永久挂起——这是把容错责任放在客户端一侧的协议设计。
- **Don't trust HTTP-library timeouts as wall-clock caps** - `requests` `timeout=(c, r)` and `httpx.Timeout(n)` are *per-chunk* read timeouts; they reset every byte, so a trickling connection can block indefinitely. For a hard deadline on raw-HTTP polling, track `time.monotonic()` at the loop level and bail explicitly. Prefer the SDK's `sessions.events.stream()` / `sessions.events.list()` over hand-rolled HTTP. See `shared/managed-agents-events.md` -> Receiving Events.
  - **不要把 HTTP 库超时当作墙钟上限**——`requests` 的 `timeout=(c, r)` 和 `httpx.Timeout(n)` 是*逐块*读取超时；每收到一个字节就重置，因此一个缓慢滴流的连接可以无限期阻塞。要对原始 HTTP 轮询设置硬性截止时间，应在循环层面跟踪 `time.monotonic()` 并显式退出。优先使用 SDK 的 `sessions.events.stream()` / `sessions.events.list()`，而不是手写 HTTP。参见 `shared/managed-agents-events.md` -> Receiving Events。
- **Messages queue** - you can send events while the session is `running` or `idle`; they're processed in order. No need to wait for a response before sending the next message. Exception: a session paused at its budget (`stop_reason: budget_reached`) accepts only settle events - change or remove the budget to resume (`shared/managed-agents-core.md` § Session budgets).
  - **消息排队**——会话处于 `running` 或 `idle` 时都可以发送事件；它们按顺序处理。无需等到响应再发下一条消息。例外：因预算暂停的会话（`stop_reason: budget_reached`）只接受结算类事件——变更或移除预算才能恢复（`shared/managed-agents-core.md` § Session budgets）。
- **Environment `config.type` is `"cloud"` or `"self_hosted"`** - `cloud` runs the container on Anthropic's infrastructure; `self_hosted` moves tool execution to your own (see `shared/managed-agents-self-hosted-sandboxes.md`).
  - **环境的 `config.type` 是 `"cloud"` 或 `"self_hosted"`**——`cloud` 把容器跑在 Anthropic 的基础设施上；`self_hosted` 把工具执行移到你自己的设施中（参见 `shared/managed-agents-self-hosted-sandboxes.md`）。
- **Archive is permanent on every resource** - archiving an agent, environment, session, vault, credential, or memory store makes it read-only with no unarchive. For agents, environments, and memory stores specifically, archived resources cannot be referenced by new sessions (existing sessions continue). Do not call `.archive()` on a production agent, environment, or memory store as cleanup - **always confirm with the user before archiving**.
  - **归档对所有资源都是永久的**——归档代理、环境、会话、vault、凭据或记忆存储会使其变为只读，且无法取消归档。具体到代理、环境和记忆存储，已归档的资源无法被新会话引用（已有会话继续运行）。不要把对生产环境的代理、环境或记忆存储调用 `.archive()` 当作清理手段——**归档前务必与用户确认**。
  
  【评论】对编码代理写入"归档前必须与用户确认"，是把不可逆破坏性操作的人工确认权显式保留给用户的一类防护条款。
