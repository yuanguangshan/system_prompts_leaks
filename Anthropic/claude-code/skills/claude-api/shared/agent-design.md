<!-- BILINGUAL-EN-ZH -->
# Agent Design Patterns / 代理设计模式

This file covers decision heuristics for building agents on the Claude API: which primitives to reach for, how to design your tool surface, and how to manage context and cost over long runs. For per-tool mechanics and code examples, see `tool-use-concepts.md` and the language-specific folders.

本文件介绍在 Claude API 上构建代理的决策启发式：该选用哪些原语、如何设计工具面、以及如何在长时间运行中管理上下文与成本。各工具的具体机制与代码示例见 `tool-use-concepts.md` 和各语言专属文件夹。

---

## Model Parameters / 模型参数

| Parameter | When to use it | What to expect |
| --- | --- | --- |
| **Adaptive thinking** (`thinking: {type: "adaptive"}`) | When you want Claude to control when and how much to think. | Claude determines thinking depth per request and automatically interleaves thinking between tool calls. No token budget to tune. |
| **Effort** (`output_config: {effort: ...}`) | When adjusting the tradeoff between thoroughness and token efficiency. | Lower effort -> fewer and more-consolidated tool calls, less preamble, terser confirmations. `medium` is often a favorable balance. Use `max` when correctness matters more than cost. |

| 参数 | 何时使用 | 预期效果 |
| --- | --- | --- |
| **自适应思考**（`thinking: {type: "adaptive"}`） | 当你想让 Claude 自行控制何时思考、思考多少时。 | Claude 按每次请求确定思考深度，并自动在工具调用之间穿插思考。无需调节 token 预算。 |
| **Effort**（`output_config: {effort: ...}`） | 当需要调节彻底性与 token 效率之间的权衡时。 | 更低的 effort -> 更少、更整合的工具调用，更少的前言，更简短的确认。`medium` 通常是较好的平衡。当正确性比成本更重要时使用 `max`。 |

See `SKILL.md` §Thinking & Effort for model support and parameter details.

模型支持情况与参数细节见 `SKILL.md` §Thinking & Effort。

---

## Designing Your Tool Surface / 设计你的工具面

### Bash vs. dedicated tools / Bash 与专用工具

Claude doesn't know your application's security boundary, approval policy, or UX surface. Claude emits tool calls; your harness handles them. The shape of those tool calls determines what the harness can do.

Claude 并不知道你的应用的安全边界、审批策略或 UX 界面。Claude 发出工具调用；由你的执行框架（harness）处理它们。这些工具调用的形状决定了框架能做什么。

A **bash tool** gives Claude broad programmatic leverage - it can perform almost any action. But it gives the harness only an opaque command string, the same shape for every action. Promoting an action to a **dedicated tool** gives the harness an action-specific hook with typed arguments it can intercept, gate, render, or audit.

**bash 工具**给 Claude 提供广泛的编程杠杆——它几乎能执行任何操作。但它交给框架的只是一个不透明的命令字符串，对所有操作都是同一种形状。把一个操作提升为**专用工具**，则给框架一个针对该操作的钩子，带有类型化的参数，框架可以拦截、门控、渲染或审计它。

**When to promote an action to a dedicated tool:**

**何时把操作提升为专用工具：**

- **Security boundary.** Actions that require gating are natural candidates. Reversibility is a useful criterion: hard-to-reverse actions (external API calls, sending messages, deleting data) can be gated behind user confirmation. A `send_email` tool is easy to gate; `bash -c "curl -X POST ..."` is not.
  **安全边界。**需要门控的操作是天然候选。可逆性是一个有用的判据：难以逆转的操作（外部 API 调用、发送消息、删除数据）可以置于用户确认之后。`send_email` 工具易于门控；`bash -c "curl -X POST ..."` 则不然。
- **Staleness checks.** A dedicated `edit` tool can reject writes if the file changed since Claude last read it. Bash can't enforce that invariant.
  **过期检查。**专用的 `edit` 工具可以在文件自 Claude 上次读取后发生变更时拒绝写入。Bash 无法强制这一不变式。
- **Rendering.** Some actions benefit from custom UI. Claude Code promotes question-asking to a tool so it can render as a modal, present options, and block the agent loop until answered.
  **渲染。**某些操作受益于自定义 UI。Claude Code 把"提问"提升为工具，从而能以模态框渲染、呈现选项，并阻塞代理循环直到获得回答。
- **Scheduling.** Read-only tools like `glob` and `grep` can be marked parallel-safe. When the same actions run through bash, the harness can't tell a parallel-safe `grep` from a parallel-unsafe `git push`, so it must serialize.
  **调度。**`glob`、`grep` 这类只读工具可以标记为可并行。同样的操作经 bash 执行时，框架无法区分可并行的 `grep` 与不可并行的 `git push`，只能串行化。

**Rule of thumb:** Start with bash for breadth. Promote to dedicated tools when you need to gate, render, audit, or parallelize the action.

**经验法则：**先用 bash 求广度。当需要对操作进行门控、渲染、审计或并行化时，提升为专用工具。

---

## Anthropic-Provided Tools / Anthropic 提供的工具

| Tool | Side | When to use it | What to expect |
| --- | --- | --- | --- |
| **Bash** | Client | Claude needs to execute shell commands. | Claude emits commands; your harness executes them. Reference implementation provided. |
| **Text editor** | Client | Claude needs to read or edit files. | Claude views, creates, and edits files via your implementation. Reference implementation provided. |
| **Computer use** | Client or Server | Claude needs to interact with GUIs, web apps, or visual interfaces. | Claude takes screenshots and issues mouse/keyboard commands. Can be self-hosted (you run the environment) or Anthropic-hosted. |
| **Code execution** | Server | Claude needs to run code in a sandbox you don't want to manage. | Anthropic-hosted container with built-in file and bash sub-tools. No client-side execution. |
| **Web search / fetch** | Server | Claude needs information past its training cutoff (news, current events, recent docs) or the content of a specific URL. | Claude issues a query or URL; Anthropic executes it and returns results with citations. |
| **Memory** | Client | Claude needs to save context across sessions. | Claude reads/writes a `/memories` directory. You implement the storage backend. |

| 工具 | 执行侧 | 何时使用 | 预期效果 |
| --- | --- | --- | --- |
| **Bash** | 客户端 | Claude 需要执行 shell 命令。 | Claude 发出命令；由你的框架执行。提供参考实现。 |
| **Text editor** | 客户端 | Claude 需要读取或编辑文件。 | Claude 经由你的实现查看、创建与编辑文件。提供参考实现。 |
| **Computer use** | 客户端或服务端 | Claude 需要与 GUI、Web 应用或可视化界面交互。 | Claude 截屏并发出鼠标/键盘命令。可自托管（由你运行环境）或由 Anthropic 托管。 |
| **Code execution** | 服务端 | Claude 需要在你不想自己维护的沙箱中运行代码。 | Anthropic 托管的容器，内置文件与 bash 子工具。无客户端执行。 |
| **Web search / fetch** | 服务端 | Claude 需要超出其训练截止时间的信息（新闻、时事、最新文档）或特定 URL 的内容。 | Claude 发出查询或 URL；由 Anthropic 执行并返回带引用的结果。 |
| **Memory** | 客户端 | Claude 需要跨会话保存上下文。 | Claude 读写一个 `/memories` 目录。由你实现存储后端。 |

**Client-side** tools are defined by Anthropic (name, schema, Claude's usage pattern) but executed by your harness. Anthropic provides reference implementations. **Server-side** tools run entirely on Anthropic infrastructure - declare them in `tools` and Claude handles the rest.

**客户端**工具由 Anthropic 定义（名称、schema、Claude 的使用模式），但由你的框架执行。Anthropic 提供参考实现。**服务端**工具完全运行在 Anthropic 基础设施上——在 `tools` 中声明即可，其余由 Claude 处理。

---

## Composing Tool Calls: Programmatic Tool Calling / 组合工具调用：程序化工具调用

With standard tool use, each tool call is a round trip: Claude calls the tool, the result lands in Claude's context, Claude reasons about it, then calls the next tool. Three sequential actions (read profile -> look up orders -> check inventory) means three round trips. Each adds latency and tokens, and most of the intermediate data is never needed again.

在标准工具调用下，每次工具调用都是一次往返：Claude 调用工具，结果落入 Claude 的上下文，Claude 对其推理，然后调用下一个工具。三个顺序操作（读取资料 -> 查询订单 -> 检查库存）意味着三次往返。每次都增加延迟与 token，且大部分中间数据此后再也用不上。

**Programmatic tool calling (PTC)** lets Claude compose those calls into a script instead. The script runs in the code execution container. When the script calls a tool, the container pauses, the call is executed (client-side or server-side), and the result returns to the running code - not to Claude's context. The script processes it with normal control flow (loops, filters, branches). Only the script's final output returns to Claude.

**程序化工具调用（PTC）**让 Claude 把这些调用组合成一段脚本。脚本运行在代码执行容器中。当脚本调用某个工具时，容器暂停，调用被执行（客户端或服务端），结果返回给正在运行的代码——而不是 Claude 的上下文。脚本用普通控制流（循环、过滤、分支）处理它。只有脚本的最终输出返回给 Claude。

| When to use it | What to expect |
| --- | --- |
| Many sequential tool calls, or large intermediate results you want filtered before they hit the context window. | Claude writes code that invokes tools as functions. Runs in the code execution container. Token cost scales with final output, not intermediate results. |

| 何时使用 | 预期效果 |
| --- | --- |
| 大量顺序工具调用，或希望中间结果在进入上下文窗口之前先被过滤。 | Claude 编写以函数形式调用工具的代码。运行在代码执行容器中。token 成本随最终输出伸缩，而非随中间结果。 |

---

## Scaling the Tool and Instruction Set / 扩展工具集与指令集

| Feature | When to use it | What to expect |
| --- | --- | --- |
| **Tool search** | Many tools available, but only a few relevant per request. Don't want all schemas in context upfront. | Claude searches the tool set and loads only relevant schemas. Tool definitions are appended, not swapped - preserves cache (see Caching below). |
| **Skills** | Task-specific instructions Claude should load only when relevant. | Each skill is a folder with a `SKILL.md`. The skill's description sits in context by default; Claude reads the full file when the task calls for it. |

| 特性 | 何时使用 | 预期效果 |
| --- | --- | --- |
| **工具搜索（Tool search）** | 可用工具很多，但每次请求只涉及少数几个；不希望所有 schema 预先占据上下文。 | Claude 搜索工具集并只加载相关的 schema。工具定义是追加而非替换——保持缓存不失效（见下文"缓存"）。 |
| **技能（Skills）** | 只有在相关时 Claude 才应加载的任务专属指令。 | 每个技能是一个含 `SKILL.md` 的文件夹。默认只有技能描述在上下文中；当任务需要时 Claude 才读取完整文件。 |

Both patterns keep the fixed context small and load detail on demand.

两种模式都让固定上下文保持精简，并按需加载细节。

---

## Long-Running Agents: Managing Context / 长时运行代理：管理上下文

| Pattern | When to use it | What to expect |
| --- | --- | --- |
| **Context editing** | Context grows stale over many turns (old tool results, completed thinking). | Tool results and thinking blocks are cleared based on configurable thresholds. Keeps the transcript lean without summarizing. |
| **Compaction** | Conversation likely to reach or exceed the context window limit. | Earlier context is summarized into a compaction block server-side. See `SKILL.md` §Compaction for the critical `response.content` handling. |
| **Memory** | State must persist across sessions (not just within one conversation). | Claude reads/writes files in a memory directory. Survives process restarts. |

| 模式 | 何时使用 | 预期效果 |
| --- | --- | --- |
| **上下文编辑（Context editing）** | 上下文随多轮对话变得陈旧（旧工具结果、已完成的思考）。 | 基于可配置的阈值清除工具结果与思考块。无需摘要即可保持记录精简。 |
| **压缩（Compaction）** | 对话可能达到或超出上下文窗口上限。 | 较早的上下文在服务端被摘要为压缩块。关键的 `response.content` 处理见 `SKILL.md` §Compaction。 |
| **记忆（Memory）** | 状态必须跨会话持久保存（不限于一次对话之内）。 | Claude 在记忆目录中读写文件。可在进程重启后保留。 |

**Choosing between them:** Context editing and compaction operate within a session - editing prunes stale turns, compaction summarizes when you're near the limit. Memory is for cross-session persistence. Many long-running agents use all three.

**如何选择：**上下文编辑与压缩都在会话之内运作——编辑修剪陈旧轮次，压缩在接近上限时做摘要。记忆用于跨会话持久化。许多长时运行的代理三者并用。

---

## Caching for Agents / 代理的缓存

**Read `prompt-caching.md` first.** It covers the prefix-match invariant, breakpoint placement, the silent-invalidator audit, and why changing tools or models mid-session breaks the cache. This section covers only the agent-specific workarounds for those constraints.

**先阅读 `prompt-caching.md`。**它涵盖前缀匹配不变式、断点放置、静默失效排查，以及为何在会话中途更换工具或模型会破坏缓存。本节只介绍针对这些约束的代理侧变通方案。

| Constraint (from `prompt-caching.md`) | Agent-specific workaround |
| --- | --- |
| Editing the system prompt mid-session invalidates the cache. | Append a `{"role": "system", ...}` message to `messages[]` instead (no beta header; on supporting models - see `prompt-caching.md` § Mid-conversation system messages). The cached prefix stays intact, and the model treats it as an operator-authority instruction rather than user text. On models that don't support it, fall back to a `<system-reminder>` text block in the user turn. |
| Switching models mid-session invalidates the cache. | Spawn a **subagent** with the cheaper model for the sub-task; keep the main loop on one model. On Managed Agents that is a `multiagent` roster entry - see `managed-agents-multiagent.md`. |
| Adding/removing tools mid-session invalidates the cache. | Use **tool search** for dynamic discovery - it appends tool schemas rather than swapping them, so the existing prefix is preserved. |

| 约束（来自 `prompt-caching.md`） | 代理侧变通方案 |
| --- | --- |
| 在会话中途编辑系统提示词会使缓存失效。 | 改为向 `messages[]` 追加一条 `{"role": "system", ...}` 消息（无需 beta 请求头；在支持的模型上——见 `prompt-caching.md` § Mid-conversation system messages）。缓存前缀保持完好，且模型会将其视为操作者权限的指令而非用户文本。在不支持的模型上，退而在用户回合中放一个 `<system-reminder>` 文本块。 |
| 在会话中途切换模型会使缓存失效。 | 为子任务用一个更便宜的模型生成**子代理（subagent）**；主循环保持单一模型。在 Managed Agents 上就是一个 `multiagent` roster 条目——见 `managed-agents-multiagent.md`。 |
| 在会话中途增删工具会使缓存失效。 | 使用**工具搜索**做动态发现——它追加工具 schema 而非替换，因此既有前缀得以保留。 |

For multi-turn breakpoint placement, use the combination in `prompt-caching.md` § Automatic vs explicit breakpoints: one explicit breakpoint on the static system prefix plus top-level automatic caching for the conversation tail (where automatic caching is available).

多轮对话的断点放置采用 `prompt-caching.md` § Automatic vs explicit breakpoints 中的组合：静态系统前缀上放一个显式断点，加上会话尾部的顶层自动缓存（在自动缓存可用的地方）。

---

For live documentation on any of these features, see `live-sources.md`.

这些特性的在线文档见 `live-sources.md`。
