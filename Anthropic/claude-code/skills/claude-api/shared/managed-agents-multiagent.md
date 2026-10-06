<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Multiagent Sessions / Managed Agents - 多智能体会话

A coordinator agent can delegate to other agents within one session. All agents **share the container and filesystem**; each runs in its own **thread** - a context-isolated event stream with its own conversation history, model, system prompt, tools, MCP servers, and skills (from that agent's own config). Threads are persistent: the coordinator can send a follow-up to a subagent it called earlier and that subagent retains its prior turns.

协调者智能体可以在一个会话内委托其他智能体。所有智能体**共享容器与文件系统**；每个智能体运行在自己的**线程**中——一条上下文隔离的事件流，拥有自己的对话历史、模型、系统提示词、工具、MCP 服务器和技能（来自该智能体自身的配置）。线程是持久的：协调者可以向之前调用过的子智能体发送后续消息，且该子智能体保留其先前的轮次。

The SDK sets the `managed-agents-2026-04-01` beta header automatically on all `client.beta.{agents,sessions}.*` calls; no additional header is required for multiagent.

SDK 会在所有 `client.beta.{agents,sessions}.*` 调用上自动设置 `managed-agents-2026-04-01` beta 头；多智能体无需额外请求头。

---

## When to use it - start with `self`, then add cheaper workers / 何时使用 - 从 `self` 开始，再添加更便宜的 worker

**If the agent's work splits into independent pieces** - several sources to research, many files or records to process, anything shaped like "look into N things, then summarize" - or one piece would fill its context with reading, **use a multiagent session instead of one long single-threaded loop.** Each delegated piece runs in its own thread with a fresh context window, threads run in parallel in the same container, and only each subagent's report comes back, so the coordinator's context stays small. There is no orchestration code to write: the coordinator is given delegation tools automatically and decides when to use them, and your client still creates one session and reads one stream.

**如果智能体的工作可以拆分为相互独立的部分**——多个待研究的来源、大量待处理的文件或记录，任何"调查 N 件事，然后汇总"形态的任务——或者某个部分会用阅读内容填满上下文，**请使用多智能体会话，而不是一条漫长的单线程循环。**每个被委托的部分都在自己的线程中运行，拥有全新的上下文窗口；线程在同一容器中并行运行；只有每个子智能体的报告会返回，因此协调者的上下文保持很小。无需编写任何编排代码：协调者会自动获得委托工具并自行决定何时使用，你的客户端仍然只需创建一个会话、读取一条流。

【评论】这一节带有明确的成本与上下文管理取向：用并行的较小模型摊薄阅读类 token 消耗，同时保护协调者的上下文窗口，是多智能体编排的常见设计动机。

**Step 1 - the smallest useful roster is the agent itself.** Add a `multiagent` block whose only entry is `{"type": "self"}`. The coordinator can then hand self-contained sub-tasks to copies of itself - same model, system prompt, and tools, minus the ability to delegate further - and combine what they report. Nothing else changes.

**第 1 步 - 最小的可用花名册就是智能体本身。**添加一个只含 `{"type": "self"}` 一个条目的 `multiagent` 块。协调者于是可以把自包含的子任务交给自己的副本——相同的模型、系统提示词和工具，只是失去进一步委托的能力——并汇总它们的报告。其他一切都不变。

```python
agent = client.beta.agents.create(
    name="Research assistant",
    description="Researches a question end to end. A copy can be spawned to own one well-scoped sub-question.",
    model="claude-opus-5-5",
    system="You are a research assistant. When a request splits into independent sub-questions, delegate each to a copy of yourself, one self-contained task per copy, then verify and combine their reports.",
    tools=[{"type": "agent_toolset_20260401"}],
    multiagent={"type": "coordinator", "agents": [{"type": "self"}]},  # the only change vs. a single agent
)

session = client.beta.sessions.create(agent=agent.id, environment_id=env.id)  # unchanged
```

**Step 2 - move the reading-heavy work to a cheaper model.** Delegated research work is mostly searching, reading, and extracting: many input tokens, little hard reasoning. Create a second agent on a smaller current-generation model (Claude Haiku 4.5, or Claude Sonnet 5.5 when the worker needs more judgment) with a narrow `system` prompt and only the tools it needs, and list it next to `self`. A roster entry is only a reference: the worker runs on its own `model`, `system`, and `tools`, and its tokens are billed at its own model's rates. The large model spends its tokens on planning, checking, and synthesis; the small model does the bulk reading.

**第 2 步 - 把阅读密集型工作转移到更便宜的模型。**被委托的研究工作大部分是搜索、阅读和提取：输入 token 多，硬推理少。在较小的当前一代模型上创建第二个智能体（Claude Haiku 4.5，或当 worker 需要更多判断力时的 Claude Sonnet 5.5），给它一个狭窄的 `system` 提示词和只需要的工具，并把它列在 `self` 旁边。花名册条目只是一个引用：worker 以它自己的 `model`、`system` 和 `tools` 运行，其 token 按它自己模型的费率计费。大模型把 token 花在规划、检查和综合上；小模型承担大量阅读。

```python
worker = client.beta.agents.create(
    name="Web researcher",
    description="Fast, low-cost, read-only researcher. Give it one well-scoped question; it searches, reads, and reports findings with sources.",
    model="claude-haiku-4-5",
    system="Answer exactly the question you are given. Search and read as much as you need, then report concise findings with a source URL or file path for every claim.",
    tools=[{
        "type": "agent_toolset_20260401",
        "default_config": {"enabled": False},
        "configs": [{"name": n, "enabled": True} for n in ("read", "glob", "grep", "web_fetch", "web_search")],
    }],
)

lead = client.beta.agents.create(
    name="Research lead",
    description="Plans and synthesizes research. A copy can be spawned to own one large sub-analysis.",
    model="claude-opus-5-5",
    system="Plan the work. Delegate each independent, reading-heavy question to Web researcher, one self-contained task per spawn, several in parallel. Keep verification and the final synthesis for yourself; spawn a copy of yourself only for a sub-analysis that needs your full capability.",
    tools=[{"type": "agent_toolset_20260401"}],
    multiagent={"type": "coordinator", "agents": [worker.id, {"type": "self"}]},
)
```

**Step 3 - add dedicated specialists.** When the sub-tasks call for different skills, give each its own agent - its own model, a narrow `system` prompt, and only the tools it needs - and roster them by ID next to `self`. Here the lead makes a change itself, sends the same review brief to several read-only reviewer threads for independent passes (one rostered agent can be spawned many times), and hands a test writer a self-contained brief; it then de-duplicates the findings, checks each against the code, and keeps the fix and the summary for itself.

**第 3 步 - 添加专职专家。**当子任务需要不同技能时，给每个子任务配置自己的智能体——自己的模型、狭窄的 `system` 提示词、只需要的工具——并按 ID 与 `self` 并列登入花名册。下例中，lead 自己完成修改，把同一份评审简报发给多个只读 reviewer 线程做独立审查（一个入册智能体可以被派生多次），并把一份自包含的简报交给 test writer；随后它对发现去重、逐条对照代码核实，把修复和总结留给自己。

```python
reviewer = client.beta.agents.create(
    name="Concurrency reviewer",
    description="Read-only reviewer for race conditions, deadlocks, lost updates, and retry/idempotency bugs. Give it the changed file paths and the invariants that must hold; it reports findings with file:line evidence. Spawn several on the same change for independent reviews.",
    model="claude-sonnet-5-5",
    system="Review only the files you are pointed at. Look for concurrency bugs: unsynchronized shared state, lock ordering, non-atomic read-modify-write, retries without idempotency. Report each finding as file:line, the interleaving that triggers it, and a suggested fix; say plainly if you found none.",
    tools=[{"type": "agent_toolset_20260401", "default_config": {"enabled": False},
            "configs": [{"name": n, "enabled": True} for n in ("read", "glob", "grep")]}],
)
test_writer = client.beta.agents.create(
    name="Test writer",
    description="Writes and runs tests. Give it the module path, the behavior to pin down, and the test command; it adds test files, runs them, and reports results with output.",
    model="claude-sonnet-5-5",
    system="Write focused tests for the behavior you are given, run them with the command you are given, and report pass/fail, the relevant output, and the paths of files you added. Do not edit non-test code; if the code under test looks wrong, report that instead.",
    tools=[{"type": "agent_toolset_20260401", "default_config": {"enabled": True},
            "configs": [{"name": n, "enabled": False} for n in ("web_fetch", "web_search")]}],
)
lead = client.beta.agents.create(
    name="Engineering lead",
    description="Plans and makes code changes and integrates specialist reports. A copy can be spawned to own one independent change.",
    model="claude-opus-5-5",
    system="Make the change yourself. Then, in parallel, send the changed paths and invariants to three Concurrency reviewers and the module path and test command to Test writer. Merge and de-duplicate the reviewers' findings, check each against the code before acting on it, fix, and have Test writer re-run. Keep design decisions and the final summary for yourself.",
    tools=[{"type": "agent_toolset_20260401"}],
    multiagent={"type": "coordinator", "agents": [reviewer.id, test_writer.id, {"type": "self"}]},
)
```

The same shape fits a pipeline of different specialists: a fast document extractor (for example on Claude Haiku 4.5) that writes one JSON file per input document, a verifier that checks each file against its source, and a lead that applies the corrections and writes the final table to `/mnt/session/outputs/`. Put the input and output paths in every task: threads share the container's filesystem, not each other's conversation.

同样的形态也适用于不同专家组成的流水线：一个快速的文档抽取器（例如运行在 Claude Haiku 4.5 上）为每份输入文档写出一个 JSON 文件，一个校验器把每个文件与其来源对照检查，以及一个 lead 应用修正并把最终表格写入 `/mnt/session/outputs/`。把输入和输出路径写进每个任务：线程共享的是容器的文件系统，而不是彼此的对话。

- **Good fits:** parallel research across sources; reading large amounts of material without filling the coordinator's context; specialists with narrow prompts and tool sets rather than one agent carrying every tool. **Poor fit:** a small single-step task - every delegation costs a round-trip and a re-briefing.
  **适配良好：**跨来源的并行研究；在不填满协调者上下文的前提下阅读大量材料；使用提示词与工具集狭窄的专家，而不是让一个智能体携带所有工具。**适配不佳：**小型的单步任务——每次委托都要付出一次往返和一次重新交代的代价。
- **Write `name` and `description` for the coordinator to read.** The coordinator chooses whom to spawn from each roster entry's name and description (the `self` entry is listed under the coordinator's own name), so say what each agent is good at and what to hand it. Names must be unique across the roster; don't name an agent `self`.
  **为协调者而写好 `name` 和 `description`。**协调者根据每个花名册条目的名称和描述决定派生谁（`self` 条目以协调者自己的名称列出），因此要说明每个智能体擅长什么、该把什么交给它。名称在整个花名册内必须唯一；不要把智能体命名为 `self`。
- **Say how to delegate in the coordinator's `system` prompt** - what to hand off and to whom, how many at once, what to keep for itself, and what is too small to be worth delegating (the *Delegating to subagents* sample prompt in `shared/model-migration.md` is a starting point). Subagents see none of the coordinator's conversation, so each task must carry the paths, constraints, and report format it needs. Spawning returns immediately; the subagent's report arrives in a later coordinator turn.
  **在协调者的 `system` 提示词中说明如何委托**——把什么交给谁、一次几个、什么留给自己、什么小到不值得委托（`shared/model-migration.md` 中的 *Delegating to subagents* 示例提示词可作为起点）。子智能体看不到协调者的任何对话，因此每个任务必须自带所需的路径、约束和报告格式。派生立即返回；子智能体的报告在协调者稍后的轮次中到达。
- **Web tool domain lists layer, never widen.** A roster agent's `web_search` / `web_fetch` calls are bound by its own `allowed_domains` / `blocked_domains`, by those of every agent that called it, and by the coordinator's current lists (allow-lists intersect, block-lists union). Keep each roster agent's allow-list inside the coordinator's - disjoint lists leave the tool present but every call fails `url_not_allowed`. See `shared/managed-agents-tools.md` § Web search & web fetch settings.
  **Web 工具的域名列表逐层叠加，只会收紧不会放宽。**花名册智能体的 `web_search` / `web_fetch` 调用同时受它自己的 `allowed_domains` / `blocked_domains`、调用链上每个智能体的同名列表、以及协调者当前列表的约束（允许列表取交集，阻止列表取并集）。把每个入册智能体的允许列表保持在协调者允许列表之内——列表不相交时工具仍然存在，但每次调用都会以 `url_not_allowed` 失败。见 `shared/managed-agents-tools.md` § Web search & web fetch settings。

  【评论】允许列表取交集、阻止列表取并集的叠加方式确保委托链上任何一环都无法放宽域名限制，属于防御纵深设计。
- **Limits:** 1-20 roster entries (at most one `self`; each rostered agent can be spawned many times), one level of delegation (a roster member must not have its own `multiagent`), and at most 25 concurrent threads per session - archive finished threads if a long session needs more (see *Interrupting and archiving threads* below).
  **限制：**1-20 个花名册条目（至多一个 `self`；每个入册智能体可被派生多次）、仅一层委托（花名册成员不得自带 `multiagent`）、每个会话至多 25 个并发线程——长会话如需更多，请归档已完成的线程（见下文 *Interrupting and archiving threads*）。

The sections below are the reference for rosters, threads, events, and client-side handling; the platform guide is `https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration.md`.

以下各节是花名册、线程、事件与客户端处理的参考；平台指南见 `https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration.md`。

---

## Declare the roster on the coordinator / 在协调者上声明花名册

`multiagent` is a **top-level field** on `agents.create()` / `agents.update()` - **not** a `tools[]` entry. `agents` lists 1-20 roster entries. Nothing changes on `sessions.create()` - the roster is resolved from the coordinator's config.

`multiagent` 是 `agents.create()` / `agents.update()` 上的一个**顶层字段**——**不是** `tools[]` 条目。`agents` 列出 1-20 个花名册条目。`sessions.create()` 没有任何变化——花名册从协调者的配置中解析。

```python
orchestrator = client.beta.agents.create(
    name="Engineering lead",
    model="claude-opus-5-5",
    system="You coordinate engineering work. Delegate code review to the reviewer and test writing to the test agent.",
    tools=[{"type": "agent_toolset_20260401"}],
    multiagent={
        "type": "coordinator",
        "agents": [
            reviewer.id,                                            # bare string - latest version
            {"type": "agent", "id": test_writer.id, "version": 4},  # pinned version
            {"type": "self"},                                       # the coordinator itself
        ],
    },
)

session = client.beta.sessions.create(agent=orchestrator.id, environment_id=env.id)
```

| Roster entry | Shape | Notes |
|---|---|---|
| String shorthand | `"agent_abc123"` | References the latest version of a stored agent. |
| Agent reference | `{type: "agent", id, version?}` | Omit `version` to pin the latest at coordinator save time. |
| Self | `{type: "self"}` | The coordinator can spawn copies of itself. |
| Advisor | `{type: "advisor", model}` | A model the session's primary thread can consult mid-turn. At most one per roster. See § Advisor below. |

| 花名册条目 | 形态 | 说明 |
|---|---|---|
| 字符串简写 | `"agent_abc123"` | 引用某个已存储智能体的最新版本。 |
| 智能体引用 | `{type: "agent", id, version?}` | 省略 `version` 即固定为协调者保存时的最新版本。 |
| Self | `{type: "self"}` | 协调者可以派生自己的副本。 |
| Advisor | `{type: "advisor", model}` | 会话主线程可在轮次中途咨询的模型。每个花名册至多一个。见下文 § Advisor。 |

If the session was created with `agent_with_overrides` (see `shared/managed-agents-core.md` -> Override agent configuration for a session), those overrides apply to the **coordinator and its `self` copies**. Roster agents referenced by ID always use their own as-created configuration - overrides do not propagate to them.

如果会话是用 `agent_with_overrides` 创建的（见 `shared/managed-agents-core.md` -> Override agent configuration for a session），这些覆盖作用于**协调者及其 `self` 副本**。按 ID 引用的花名册智能体始终使用它们创建时的配置——覆盖不会传播到它们。

The coordinator's thread receives delegation tools for working the roster: `list_agents` (see the roster) and `send_to_agent` (task or message a member). Up to **20 unique agents** in the roster; the coordinator may spawn **multiple copies** of each. **One level of delegation only** - and it is enforced rather than silently flattened: rostering an agent that itself carries a `multiagent.agents` roster fails the create or update with a validation error.

协调者的线程获得操作花名册的委托工具：`list_agents`（查看花名册）和 `send_to_agent`（向成员发送任务或消息）。花名册最多 **20 个不同的智能体**；协调者可以派生每个智能体的**多个副本**。**仅允许一层委托**——而且是强制执行而非静默展平：把一个自带 `multiagent.agents` 花名册的智能体登入花名册，会导致 create 或 update 以校验错误失败。

**Inference geo pins must be roster-uniform.** When agents pin an inference geography (`model.inference_geo` - see `shared/managed-agents-core.md` § Pinning inference geography), the coordinator's pin and every roster member's must all be the same value or all be unset. A mismatched roster is a 400 validation error, both when the agent is saved and when a session-create `model` override changes any of the pins.

**推理地域锚定必须在花名册内一致。**当智能体锚定推理地域时（`model.inference_geo`——见 `shared/managed-agents-core.md` § Pinning inference geography），协调者的锚定与每个花名册成员的锚定必须全部相同或全部未设置。不一致的花名册是 400 校验错误，在保存智能体时、以及在会话创建时用 `model` 覆盖改变任一锚定的情况下都会触发。

---

## Threads / 线程

The session-level event stream is the **primary thread** - it shows the coordinator's trace plus a condensed view of subagent activity (thread status transitions and cross-thread messages, not every subagent tool call). Drill into a specific subagent via the per-thread endpoints:

会话级事件流就是**主线程**——它显示协调者的轨迹加上子智能体活动的精简视图（线程状态转换与跨线程消息，而非子智能体的每次工具调用）。通过按线程的端点深入某个具体的子智能体：

| Operation | HTTP | SDK (`client.beta.sessions.threads.*`) |
|---|---|---|
| List threads | `GET /v1/sessions/{sid}/threads` | `.list(session_id)` |
| Retrieve one | `GET /v1/sessions/{sid}/threads/{tid}` | `.retrieve(thread_id, session_id=...)` |
| Archive | `POST /v1/sessions/{sid}/threads/{tid}/archive` | `.archive(thread_id, session_id=...)` |
| List thread events | `GET /v1/sessions/{sid}/threads/{tid}/events` | `.events.list(thread_id, session_id=...)` |
| Stream thread events | `GET /v1/sessions/{sid}/threads/{tid}/stream` | `.events.stream(thread_id, session_id=...)` |

| 操作 | HTTP | SDK（`client.beta.sessions.threads.*`） |
|---|---|---|
| 列出线程 | `GET /v1/sessions/{sid}/threads` | `.list(session_id)` |
| 获取单个 | `GET /v1/sessions/{sid}/threads/{tid}` | `.retrieve(thread_id, session_id=...)` |
| 归档 | `POST /v1/sessions/{sid}/threads/{tid}/archive` | `.archive(thread_id, session_id=...)` |
| 列出线程事件 | `GET /v1/sessions/{sid}/threads/{tid}/events` | `.events.list(thread_id, session_id=...)` |
| 流式读取线程事件 | `GET /v1/sessions/{sid}/threads/{tid}/stream` | `.events.stream(thread_id, session_id=...)` |

Each `SessionThread` carries `id`, `status` (`running` | `idle` | `rescheduling` | `terminated`), `agent` (a resolved snapshot of the agent config - `id`, `name`, `model`, `system`, `tools`, `skills`, `mcp_servers`, `version` - except advisor threads, whose `agent` is the two-field advisor form `{"type": "advisor", "model": ...}` - see § Advisor), `parent_thread_id` (null for the primary thread, which is included in the list), `archived_at`, and optional `stats`/`usage`. Per-thread `usage.list_cost` figures do **not** sum to the session total - the session figure additionally includes session running time and each figure is rounded independently; the session-level `usage.list_cost` is authoritative. **Session status aggregates thread statuses** - if any thread is `running`, `session.status` is `running`. Max **25 concurrent threads** (advisor threads are exempt - see § Advisor). When draining a per-thread stream, break on `session.thread_status_idle` (and check its `stop_reason` as you would for the session-level idle).

每个 `SessionThread` 携带 `id`、`status`（`running` | `idle` | `rescheduling` | `terminated`）、`agent`（智能体配置的已解析快照——`id`、`name`、`model`、`system`、`tools`、`skills`、`mcp_servers`、`version`——advisor 线程除外，其 `agent` 为两字段的 advisor 形式 `{"type": "advisor", "model": ...}`——见 § Advisor）、`parent_thread_id`（主线程为 null，主线程包含在列表中）、`archived_at`，以及可选的 `stats`/`usage`。按线程的 `usage.list_cost` **不会**加总等于会话总量——会话数字额外包含会话运行时间，且每个数字独立舍入；会话级 `usage.list_cost` 才是权威值。**会话状态聚合线程状态**——只要任一线程是 `running`，`session.status` 就是 `running`。最多 **25 个并发线程**（advisor 线程豁免——见 § Advisor）。消费按线程的流时，以 `session.thread_status_idle` 作为退出条件（并像会话级 idle 一样检查其 `stop_reason`）。

**A session budget is one shared cap across all threads** - no per-thread caps. Each thread's consumption is priced at its own served model, and threads pause independently (`stop_reason: budget_reached`) as the shared cap is reached; one thread can pause while another finishes its in-flight request. A thread waiting on `requires_action` outranks the cap at the session level. See `shared/managed-agents-core.md` § Session budgets.

**会话预算是所有线程共享的一个上限**——没有按线程的上限。每个线程的消耗按其服务的模型计价，线程在共享上限达到时各自独立暂停（`stop_reason: budget_reached`）；一个线程可以暂停，而另一个线程完成其进行中的请求。在会话层面，等待 `requires_action` 的线程优先于上限。见 `shared/managed-agents-core.md` § Session budgets。

---

## Multiagent events (on the session stream) / 多智能体事件（在会话流上）

| Event | Payload highlights | Meaning |
|---|---|---|
| `session.thread_created` | `session_thread_id`, `agent_name` | A new thread was created. |
| `session.thread_status_running` | `session_thread_id`, `agent_name` | Thread started activity. |
| `session.thread_status_idle` | `session_thread_id`, `agent_name`, **`stop_reason`** | Thread is awaiting input - or paused at the session's shared budget (`stop_reason: budget_reached`). Inspect `stop_reason` (same shape as `session.status_idle.stop_reason`). |
| `session.thread_status_rescheduled` | `session_thread_id`, `agent_name` | Thread is rescheduling after a retryable error. |
| `session.thread_status_terminated` | `session_thread_id`, `agent_name` | Thread ended - completed its work and self-terminated (advisor consultation threads - see § Advisor), was archived, or hit a terminal error. |
| `agent.thread_message_sent` | `to_session_thread_id`, `to_agent_name`, `content` | *This* thread sent a message to another thread. On the primary stream: the coordinator sent a task or follow-up to an agent. |
| `agent.thread_message_received` | `from_session_thread_id`, `from_agent_name`, `content` | A message arrived on *this* thread from another. On the primary stream: an agent sent a report or question to the coordinator. |

| 事件 | 载荷要点 | 含义 |
|---|---|---|
| `session.thread_created` | `session_thread_id`, `agent_name` | 新线程已创建。 |
| `session.thread_status_running` | `session_thread_id`, `agent_name` | 线程开始活动。 |
| `session.thread_status_idle` | `session_thread_id`, `agent_name`, **`stop_reason`** | 线程正在等待输入——或已在会话共享预算处暂停（`stop_reason: budget_reached`）。请检查 `stop_reason`（与 `session.status_idle.stop_reason` 形状相同）。 |
| `session.thread_status_rescheduled` | `session_thread_id`, `agent_name` | 线程在可重试错误后正在重新调度。 |
| `session.thread_status_terminated` | `session_thread_id`, `agent_name` | 线程已结束——完成工作并自行终止（advisor 咨询线程——见 § Advisor）、被归档，或遇到终止性错误。 |
| `agent.thread_message_sent` | `to_session_thread_id`, `to_agent_name`, `content` | *本*线程向另一个线程发送了消息。在主流上：协调者向某个智能体发送了任务或后续消息。 |
| `agent.thread_message_received` | `from_session_thread_id`, `from_agent_name`, `content` | 一条消息从另一个线程到达*本*线程。在主流上：某个智能体向协调者发送了报告或提问。 |

> **Direction is relative to the thread whose stream carries the event**, not to the coordinator. The same delegated task is an `agent.thread_message_sent` on the primary stream and an `agent.thread_message_received` on the child's own stream. Reading `_received` as "a subagent finished" is wrong once you're reading a child stream.
>
> **方向相对于承载该事件的流所属的线程**，而非相对于协调者。同一个被委托的任务，在主流上是 `agent.thread_message_sent`，在子线程自己的流上则是 `agent.thread_message_received`。一旦你在读子线程流，把 `_received` 读成"某个子智能体完成了"就是错的。

---

## Previewing a subagent's text / 预览子智能体的文本

Each thread's stream accepts the same `event_deltas[]` parameter as the session-level stream, so you can watch a subagent's text as the model generates it:

每个线程的流接受与会话级流相同的 `event_deltas[]` 参数，因此你可以在模型生成时实时查看子智能体的文本：

```
GET /v1/sessions/{sid}/threads/{tid}/stream?event_deltas%5B%5D=agent.message
```

**Previews are thread-scoped.** A child's previews are delivered only on that child's stream and never cross-posted to the session-level stream, whose previews stay scoped to the primary thread. So watching a subagent live means opening its thread stream - the session stream will not show it, no matter what you pass.

**预览以线程为作用域。**子线程的预览只在它自己的流上投递，绝不会跨投到会话级流，会话级流的预览始终局限于主线程。因此实时查看子智能体意味着打开它的线程流——会话流不会显示它，无论你传什么参数。

> Warning: **Only plain assistant text previews.** A subagent's *reply to its coordinator* rides `agent.thread_message_sent` and is never previewed. A worker that does nothing but report back therefore streams no deltas at all, even with a correct opt-in on the right thread. To get a live preview out of a subagent, its prompt has to make it write the answer as a plain assistant message in its own thread first, and only then report to the coordinator. Run one accumulator per connection, and exit the read loop on `session.thread_status_idle`. Opt-in, accumulate, and reconcile details: `shared/managed-agents-events.md` -> Live previews.
>
> 警示：**只预览普通的助手文本。**子智能体*对协调者的回复*走 `agent.thread_message_sent`，从不被预览。因此一个只负责汇报的 worker 即使在正确的线程上正确开启了预览，也完全不会流出任何增量。要从子智能体拿到实时预览，它的提示词必须让它先在自己的线程里以普通助手消息的形式写出答案，然后再向协调者汇报。每条连接运行一个累加器，读到 `session.thread_status_idle` 时退出读取循环。开启、累加与对账细节见 `shared/managed-agents-events.md` -> Live previews。

---

## Advisor / Advisor（顾问）

An `{"type": "advisor", "model": "<model id>"}` roster entry gives the session's **primary thread** an advisor: a model it can consult mid-turn for strategic guidance (planning an approach, getting unstuck, reviewing work before finishing). The entry has exactly two fields - `type` and `model` - and can sit alongside any other roster forms; a roster with no other entries works too. The advisor is also available as a server tool on the Messages API (`advisor_20260301` - see `shared/tool-use-concepts.md` -> Advisor); the Managed Agents surface differs in configuration and delivery: the roster entry has **no `max_uses`, `max_tokens`, or `caching` fields**, and advice arrives through thread events rather than `advisor_tool_result` blocks.

`{"type": "advisor", "model": "<model id>"}` 花名册条目为会话的**主线程**提供一个 advisor：一个可在轮次中途咨询战略指引的模型（规划方法、摆脱卡顿、完成前审查工作）。该条目只有两个字段——`type` 和 `model`——可以与任何其他花名册形式并列；只有这一个条目的花名册也可以。advisor 在 Messages API 上也可作为服务器工具使用（`advisor_20260301`——见 `shared/tool-use-concepts.md` -> Advisor）；Managed Agents 这一层在配置与投递上有所不同：花名册条目**没有 `max_uses`、`max_tokens` 或 `caching` 字段**，且建议通过线程事件而非 `advisor_tool_result` 块到达。

```python
agent = client.beta.agents.create(
    name="Backend engineer",
    model="claude-sonnet-5-5",
    system="You implement backend features end to end.",
    multiagent={
        "type": "coordinator",
        "agents": [{"type": "advisor", "model": "claude-opus-5-5"}],
    },
)
```

(Claude Opus 5.5 is the default advisor choice. It is a redacted advisor - the agent reads its advice server-side, but the client sees `[{"type": "redacted"}]`; see *Plaintext vs redacted delivery* below. For client-readable advice, a plaintext advisor such as `claude-opus-4-8` is valid only when the agent's own model is `claude-opus-4-8` or below - agents on Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Fable 5.1, or Claude Mythos 5.1 can only pair with redacted advisors, so client-readable advice is not available for them (pairing table: `shared/tool-use-concepts.md`).)

（Claude Opus 5.5 是默认的 advisor 选择。它是一个脱敏的 advisor——智能体在服务器端读取其建议，但客户端看到的是 `[{"type": "redacted"}]`；见下文 *Plaintext vs redacted delivery*。若要客户端可读的建议，`claude-opus-4-8` 这类明文 advisor 只有在智能体自身模型为 `claude-opus-4-8` 或更低时才有效——运行在 Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5、Claude Fable 5.1 或 Claude Mythos 5.1 上的智能体只能搭配脱敏 advisor，因此无法获得客户端可读的建议（配对表见 `shared/tool-use-concepts.md`）。）

**Rules:**

**规则：**

- **At most one advisor entry per roster.** The entry occupies the reserved roster name `anthropic.advisor` - a roster that also lists a member literally named `anthropic.advisor` is a 400. In responses, the advisor entry is echoed **last** in the roster regardless of submitted position.
  **每个花名册至多一个 advisor 条目。**该条目占用保留的花名册名称 `anthropic.advisor`——同时列出字面名称为 `anthropic.advisor` 的成员的花名册会返回 400。在响应中，无论提交位置如何，advisor 条目都会回显在花名册**末尾**。
- **Pairing is validated at agent save:** the advisor model must meet a minimum capability bar, and the agent's own model must not be more capable than its advisor (equals can pair). Invalid pairing -> 400. The valid pairs mirror the Messages advisor tool's executor<->advisor table (`shared/tool-use-concepts.md`).
  **配对在保存智能体时校验：**advisor 模型必须达到最低能力门槛，且智能体自身模型不得强于其 advisor（相等可以配对）。无效配对 -> 400。有效配对镜像 Messages advisor 工具的 executor<->advisor 表（`shared/tool-use-concepts.md`）。
- **Only the primary thread consults it.** The advisor is not a roster agent: invisible to the coordinator's `list_agents` tool, unreachable via `send_to_agent`, and roster agents cannot consult it.
  **只有主线程可以咨询它。**advisor 不是花名册智能体：对协调者的 `list_agents` 工具不可见，无法通过 `send_to_agent` 触达，花名册智能体也不能咨询它。

**How consultations work.** Each consultation runs as a platform-spawned thread named `anthropic.advisor` that terminates itself when done; the advice is delivered to the primary thread as an `agent.thread_message_received` event. Typical event order (the reserved name rides `agent_name` on lifecycle events and `from_agent_name` on the delivery):

**咨询如何运作。**每次咨询都作为一个由平台派生的、名为 `anthropic.advisor` 的线程运行，完成后自行终止；建议以 `agent.thread_message_received` 事件投递到主线程。典型事件顺序（保留名称在生命周期事件上出现在 `agent_name`，在投递事件上出现在 `from_agent_name`）：

1. `session.thread_created`
   1. `session.thread_created`
2. `session.thread_status_running`
   2. `session.thread_status_running`
3. `agent.thread_message_received` - the advice
   3. `agent.thread_message_received` - 建议内容
4. `session.thread_status_idle` (`stop_reason: end_turn`)
   4. `session.thread_status_idle`（`stop_reason: end_turn`）
5. `session.thread_status_terminated`
   5. `session.thread_status_terminated`

No `agent.tool_use` and no `agent.thread_message_sent` are emitted for a consultation, and **the advice delivery is not guaranteed to precede the advisor thread's idle/terminated events** - don't treat those as "advice already delivered."

咨询不会发出 `agent.tool_use` 和 `agent.thread_message_sent`，而且**不保证建议投递先于 advisor 线程的 idle/terminated 事件**——不要把这些当作"建议已投递"。

**Plaintext vs redacted delivery.** Whether your client can read the advice is the advisor model's policy, mirroring the Messages advisor tool's result variants: models that return plaintext there deliver readable text content here; models that return redacted results deliver `[{"type": "redacted"}]` as the message content on every client surface, while the agent still reads the full advice server-side. Advisor thinking is never surfaced. Clients cannot send `redacted` blocks themselves - an event containing one is a 400.

**明文与脱敏投递。**你的客户端能否读取建议取决于 advisor 模型的策略，镜像 Messages advisor 工具的结果变体：在那里返回明文的模型在这里投递可读的文本内容；返回脱敏结果的模型在每个客户端表面都以 `[{"type": "redacted"}]` 作为消息内容，而智能体仍在服务器端读取完整建议。advisor 的思考过程从不暴露。客户端不能自行发送 `redacted` 块——包含它的事件是 400。

**Failure and interruption.** A failed consultation - or one abandoned via a `user.interrupt` carrying the advisor thread's `session_thread_id` - never fails the agent's turn: the agent continues after a generic notice. A session-level `user.interrupt` during a consultation halts the whole session as usual (every thread, primary included), terminating the advisor thread with no advice delivered.

**失败与中断。**一次失败的咨询——或一次通过携带 advisor 线程 `session_thread_id` 的 `user.interrupt` 放弃的咨询——绝不会让智能体的轮次失败：智能体会在一条通用通知后继续。咨询期间的会话级 `user.interrupt` 会照常暂停整个会话（包括主线程在内的每个线程），advisor 线程被终止且不投递建议。

**Threads, billing, caching.** Advisor threads are **exempt from the 25-concurrent-thread limit**. They appear in the session's thread list with `agent` set to the advisor form as configured (`{"type": "advisor", "model": ...}`) and `parent_thread_id` set to the primary thread. Consultations are billed at the advisor model's rates; their tokens appear in the advisor thread's usage and the session's totals. Advisor-side prompt caching is automatic - nothing to configure.

**线程、计费、缓存。**advisor 线程**豁免 25 并发线程上限**。它们出现在会话的线程列表中，`agent` 为配置的 advisor 形式（`{"type": "advisor", "model": ...}`），`parent_thread_id` 指向主线程。咨询按 advisor 模型的费率计费；其 token 计入 advisor 线程的用量和会话总量。advisor 侧的提示词缓存是自动的——无需配置。

**Removing the advisor:** update the agent with a roster that omits the entry; if the advisor is the roster's only entry, clear the roster with `"multiagent": null`.

**移除 advisor：**用一个省略该条目的花名册更新智能体；如果 advisor 是花名册唯一的条目，用 `"multiagent": null` 清空花名册。

---

## Tool permissions and custom tools from subagent threads / 来自子智能体线程的工具权限与自定义工具

When a subagent needs your client (a tool call that paused for approval - `always_ask`, or `auto` with no determination - or a custom tool result), the request is **cross-posted to the primary thread** with `session_thread_id` identifying the originating thread - so you only need to watch the session stream. Reply with `user.tool_confirmation` (carrying `tool_use_id`) or `user.custom_tool_result` (carrying `custom_tool_use_id`), and **echo the `session_thread_id` from the originating event** (the SDK param type and docstring expect it). The server also routes by the tool-use ID, so the echo is belt-and-suspenders rather than load-bearing - but include it.

当子智能体需要你的客户端介入时（一次因等待审批而暂停的工具调用——`always_ask`，或服务器未作出判定的 `auto`——或一个自定义工具结果），该请求会**跨投到主线程**，并以 `session_thread_id` 标识来源线程——因此你只需监听会话流。用 `user.tool_confirmation`（携带 `tool_use_id`）或 `user.custom_tool_result`（携带 `custom_tool_use_id`）回复，并**回显来源事件中的 `session_thread_id`**（SDK 参数类型和文档字符串要求它）。服务器还会按工具使用 ID 路由，因此该回显是双保险而非关键依赖——但仍应包含。

```python
for event_id in stop.event_ids:
    pending = events_by_id[event_id]
    confirmation = {
        "type": "user.tool_confirmation",
        "tool_use_id": event_id,
        "result": "allow",
    }
    if pending.session_thread_id is not None:
        confirmation["session_thread_id"] = pending.session_thread_id
    client.beta.sessions.events.send(session.id, events=[confirmation])
```

The same pattern applies to `user.custom_tool_result`.

同样的模式适用于 `user.custom_tool_result`。

**`auto` in multiagent sessions.** Only your `user.message` events on the primary thread can lead the server to allow a call it would otherwise deny under `auto`; nothing in a subagent's thread carries that weight (your client posts no messages there, and the coordinator's messages to the subagent carry none). A call the server denies under `auto` is **not** cross-posted - its event and the error tool result appear only on the subagent's own thread stream, and the subagent keeps running.

**多智能体会话中的 `auto`。**只有你在主线程上的 `user.message` 事件才能让服务器在 `auto` 下放行它原本会拒绝的调用；子智能体线程中的任何内容都没有这个分量（你的客户端不在那里发消息，协调者发给子智能体的消息也不携带该语义）。服务器在 `auto` 下拒绝的调用**不会**被跨投——它的事件和错误工具结果只出现在子智能体自己的线程流上，子智能体继续运行。

---

## Interrupting and archiving threads / 中断与归档线程

- **`user.interrupt` without `session_thread_id` interrupts every non-archived thread in the session, including the primary** - it is not a primary-only stop. Pass `session_thread_id` to target one thread.
  **不带 `session_thread_id` 的 `user.interrupt` 会中断会话中每个未归档的线程，包括主线程**——并非只停止主线程。传入 `session_thread_id` 可只针对一个线程。
- **Against a child thread blocked on `requires_action`**, the interrupt closes each pending tool call with an *error* tool result (`"Tool execution was interrupted before completion. Please retry."`) and re-emits `session.thread_status_idle` with `stop_reason: end_turn` directly - the model is not sampled. Against a thread already `idle`, the interrupt is a no-op - with one exception: a session on a self-hosted environment whose worker failed the claimed work item (e.g. a memory-store mount error) sits `idle`, and a `user.interrupt` re-queues that work so the next worker claim retries (`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores -> Troubleshooting).
  **对阻塞在 `requires_action` 上的子线程**，中断会以*错误*工具结果（`"Tool execution was interrupted before completion. Please retry."`）关闭每个待处理的工具调用，并直接重新发出 `session.thread_status_idle`、`stop_reason: end_turn`——不再采样模型。对已经 `idle` 的线程，中断是无操作——有一个例外：自托管环境上 worker 认领的工作项失败的会话（例如内存存储挂载错误）停留在 `idle`，`user.interrupt` 会把该工作重新入队，下一次 worker 认领时重试（`shared/managed-agents-self-hosted-sandboxes.md` § Memory stores -> Troubleshooting）。
- **Archive requires the thread to be idle, and `requires_action` counts as idle** - a thread parked on a pending tool call can be archived directly. Only a *running* thread must be interrupted first.
  **归档要求线程处于 idle，而 `requires_action` 算作 idle**——停在待处理工具调用上的线程可以直接归档。只有*正在运行*的线程必须先中断。

---

## Pitfalls / 陷阱

- **Don't put the roster on `sessions.create()` or in `tools[]`.** `multiagent` is a top-level agent field; update the coordinator, then start a session that references it.
  **不要把花名册放在 `sessions.create()` 上或 `tools[]` 里。**`multiagent` 是智能体的顶层字段；先更新协调者，再启动引用它的会话。
- **Don't assume shared context.** Threads share the filesystem but not conversation history or tools. If the coordinator needs a subagent to act on something, it must say so in the delegated message (or write it to disk).
  **不要假设上下文共享。**线程共享文件系统，但不共享对话历史或工具。如果协调者需要子智能体对某物采取行动，必须在委托消息中说明（或写入磁盘）。
- **Depth > 1 is a validation error.** Rostering an agent that itself carries a `multiagent.agents` roster fails the create or update - only the session's coordinator delegates.
  **深度超过 1 是校验错误。**把一个自带 `multiagent.agents` 花名册的智能体登入花名册会导致 create 或 update 失败——只有会话的协调者可以委托。

For per-language bindings beyond Python, WebFetch `https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration.md` (see `shared/live-sources.md`).

Python 之外的其他语言绑定，请 WebFetch `https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration.md`（见 `shared/live-sources.md`）。
