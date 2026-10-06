<!-- BILINGUAL-EN-ZH -->
---
name: workflow-authoring
description: |-
  Reference for writing a Workflow tool script (script API and gotchas, resume, quality patterns, worked examples). Load before authoring a script for a workflow the user already opted into; it does not itself authorize running one.
---

# Workflow authoring reference / 工作流编写参考

A workflow structures work across many agents — to be comprehensive (decompose and cover in parallel), to be confident (independent perspectives and adversarial checks before committing), or to take on scale one context can't hold (migrations, audits, broad sweeps). The script is where you encode that structure: what fans out, what verifies, what synthesizes.

工作流将工作结构化地分布到多个智能体上 — 目的可以是求全面（分解后并行覆盖）、求确信（在提交结论前进行独立视角与对抗性核查），或承接单个上下文无法容纳的规模（迁移、审计、大范围扫查）。脚本就是你编码该结构的地方：什么被扇出、什么做验证、什么做汇总。

When you do call it, the right move is often **hybrid**: scout inline first (list the files, find the channels, scope the diff) to discover the work-list, then call Workflow to pipeline over it. You don't need to know the shape before the *task* — only before the *orchestration step*.

当你确实要调用它时，正确的做法往往是**混合式**：先在内联侦察（列出文件、找到频道、确定 diff 范围）以发现工作清单，再调用 Workflow 对其做流水线处理。在*任务*开始前你不需要知道工作的形态 — 只需要在*编排步骤*开始前知道。

Common single-phase workflows you can chain across turns:

可以在多轮之间串联的常见单阶段工作流：

- **Understand** — parallel readers over relevant subsystems → structured map
  **理解** — 对相关子系统并行阅读 → 结构化地图
- **Design** — judge panel of N independent approaches → scored synthesis
  **设计** — 对 N 个独立方案做评审团式打分 → 综合合成
- **Review** — dimensions → find → adversarially verify (the review-changes example)
  **评审** — 多维度 → 发现 → 对抗性核验（review-changes 示例）
- **Research** — multi-modal sweep → deep-read → synthesize
  **研究** — 多模式扫查 → 深度阅读 → 综合
- **Migrate** — discover sites → transform each (worktree isolation) → verify
  **迁移** — 发现位置 → 逐一转换（worktree 隔离）→ 核验

For larger work, run several in sequence — read each result before deciding the next phase. You stay in the loop; each workflow is one well-scoped fan-out.

对更大的工作，按顺序运行多个工作流 — 在决定下一阶段前先读取每个结果。你始终留在决策环中；每个工作流都是一次范围良好的扇出。

**Ultracode.** When a system-reminder confirms ultracode is on, that opt-in is standing: author and run a workflow for every substantive task by default. The goal is the most exhaustive, correct answer you can produce — token cost is not a constraint. For multi-phase work (understand → design → implement → review), that often means several workflows in sequence — one per phase — so you stay in the loop between them. The quality patterns below (adversarial verify, multi-modal sweep, completeness critic, loop-until-dry) are the tools; pick what fits the task. Lean toward orchestrating with workflows and adversarially verifying your findings — unless the work is trivial or already verified. Solo only on conversational turns or trivial mechanical edits. When a reminder says ultracode is off, revert to the opt-in rule in the Workflow tool description.

**Ultracode。** 当系统提醒确认 ultracode 已开启时，该选择授权是长期有效的：默认为每个实质性任务编写并运行工作流。目标是产出你能给出的最详尽、最正确的答案 — token 成本不是约束。对多阶段工作（理解 → 设计 → 实现 → 评审），这通常意味着按顺序运行多个工作流 — 每阶段一个 — 从而你在它们之间保持知情。下面的质量模式（对抗性核验、多模式扫查、完备性批评者、循环直到无新发现）就是工具箱；挑选适合任务的即可。倾向于用工作流编排并对发现做对抗性核验 — 除非工作微不足道或已经过核验。只有纯对话轮次或琐碎的机械性编辑才单独完成。当提醒表明 ultracode 已关闭时，恢复为 Workflow 工具描述中的选择授权规则。

Pass the script inline via `script` — do not Write it to a file first. Every invocation automatically persists its script to a file under the session directory and returns the path in the tool result. To iterate on a workflow, edit that file with Write/Edit and re-invoke Workflow with `{scriptPath: "<path>"}` instead of resending the full script.

通过 `script` 内联传入脚本 — 不要先把它 Write 到文件。每次调用都会自动把脚本持久化到会话目录下的一个文件中，并在工具结果中返回该路径。要迭代工作流，用 Write/Edit 编辑该文件，然后以 `{scriptPath: "<path>"}` 重新调用 Workflow，而不是重发完整脚本。

Every script must begin with `export const meta = {...}`:

每个脚本必须以 `export const meta = {...}` 开头：

  export const meta = {
    name: 'find-flaky-tests',
    description: 'Find flaky tests and propose fixes',   // one-line, shown in permission dialog
    phases: [                                            // one entry per phase() call
      { title: 'Scan', detail: 'grep test logs for retries' },
      { title: 'Fix', detail: 'one agent per flaky test' },
    ],
  }
  // script body starts here — use agent()/parallel()/pipeline()/phase()/log()
  phase('Scan')
  const flaky = await agent('grep CI logs for retry markers', {schema: FLAKY_SCHEMA})
  ...

The `meta` object must be a PURE LITERAL — no variables, function calls, spreads, or template interpolation. Required fields: `name`, `description`. Optional: `whenToUse` (shown in the workflow list), `phases`. Use the SAME phase titles in meta.phases as in phase() calls — titles are matched exactly; a phase() call with no matching meta entry just gets its own progress group.

`meta` 对象必须是纯字面量 — 不允许变量、函数调用、展开运算或模板插值。必填字段：`name`、`description`。可选字段：`whenToUse`（显示在工作流列表中）、`phases`。meta.phases 中使用的阶段标题必须与 phase() 调用中的完全一致 — 标题是精确匹配的；没有对应 meta 条目的 phase() 调用只会获得自己的进度分组。
【评论】要求 meta 为纯字面量属于静态可校验的设计约束：编排框架可在脚本执行前解析并展示阶段结构。

Script body hooks:

脚本主体钩子：

- `agent(prompt: string, opts?: {label?: string, phase?: string, schema?: object, effort?: string, isolation?: 'worktree', agentType?: string}): Promise<any>` — spawn a subagent. Without schema, returns its final text as a string. With schema (a JSON Schema), the subagent is forced to call a StructuredOutput tool and agent() returns the validated object — no parsing needed. Returns null if the user skips the agent mid-run or the subagent dies on a terminal API error after retries (filter with .filter(Boolean)). opts.label overrides the display label. opts.phase explicitly assigns this agent to a progress group (use this inside pipeline()/parallel() stages to avoid races on the global phase() state — same phase string → same group box). opts.effort overrides the reasoning effort for this agent call ('low' | 'medium' | 'high' | 'xhigh' | 'max') — omit to inherit the session effort; use 'low' for cheap mechanical stages and higher tiers only for the hardest verify/judge stages. opts.isolation: 'worktree' runs the agent in a fresh git worktree — EXPENSIVE (~200-500ms setup + disk per agent), use ONLY when agents mutate files in parallel and would otherwise conflict; the worktree is auto-removed if unchanged. opts.agentType uses a custom subagent type (e.g. 'general-purpose', 'code-reviewer') instead of the default workflow subagent — resolved from the same registry as the Agent tool; composes with schema (the custom agent's system prompt gets a StructuredOutput instruction appended).
  `agent(prompt: string, opts?: {label?: string, phase?: string, schema?: object, effort?: string, isolation?: 'worktree', agentType?: string}): Promise<any>` — 生成一个子智能体。不带 schema 时，将其最终文本作为字符串返回。带 schema（一个 JSON Schema）时，子智能体被强制调用 StructuredOutput 工具，agent() 返回通过校验的对象 — 无需解析。如果用户在运行中途跳过该智能体，或子智能体在重试后因终端性 API 错误而死亡，则返回 null（用 .filter(Boolean) 过滤）。opts.label 覆盖显示标签。opts.phase 将该智能体显式指派到某个进度分组（在 pipeline()/parallel() 的阶段内使用它，以避免对全局 phase() 状态的竞态 — 相同的 phase 字符串 → 相同的分组框）。opts.effort 覆盖该智能体调用的推理力度（'low' | 'medium' | 'high' | 'xhigh' | 'max'）— 省略则继承会话的推理力度；廉价的机械性阶段用 'low'，只有最难的核验/评审阶段才用更高档位。opts.isolation: 'worktree' 在一个全新的 git worktree 中运行该智能体 — 代价高昂（每个智能体约 200-500ms 的设置开销 + 磁盘占用），只在多个智能体并行修改文件且否则会冲突时使用；未发生更改的 worktree 会被自动移除。opts.agentType 使用自定义子智能体类型（如 'general-purpose'、'code-reviewer'）代替默认的工作流子智能体 — 从与 Agent 工具相同的注册表解析；可与 schema 组合（自定义智能体的系统提示词会被附加一条 StructuredOutput 指令）。
- `pipeline(items, stage1, stage2, ...): Promise<any[]>` — run each item through all stages independently, NO barrier between stages. Item A can be in stage 3 while item B is still in stage 1. This is the DEFAULT for multi-stage work. Wall-clock = slowest single-item chain, not sum-of-slowest-per-stage. Every stage callback receives (prevResult, originalItem, index) — use originalItem/index in later stages to label work without threading context through stage 1's return value. A stage that throws drops that item to `null` and skips its remaining stages.
  `pipeline(items, stage1, stage2, ...): Promise<any[]>` — 让每个条目独立地流经所有阶段，阶段之间没有屏障。条目 A 可以在第 3 阶段，而条目 B 还在第 1 阶段。这是多阶段工作的默认选择。总耗时 = 最慢的单条目链路，而不是各阶段最慢耗时之和。每个阶段回调接收 (prevResult, originalItem, index) — 在后续阶段使用 originalItem/index 来标注工作，而不必让上下文穿过阶段 1 的返回值。抛出异常的阶段会把该条目置为 `null` 并跳过其剩余阶段。
- `parallel(thunks: Array<() => Promise<any>>): Promise<any[]>` — run tasks concurrently. This is a BARRIER: awaits all thunks before returning. A thunk that throws (or whose agent errors) resolves to `null` in the result array — the call itself never rejects, so `.filter(Boolean)` before using the results. Use ONLY when you genuinely need all results together.
  `parallel(thunks: Array<() => Promise<any>>): Promise<any[]>` — 并发运行任务。这是一个屏障（BARRIER）：返回前等待所有 thunk 完成。抛出异常（或其智能体出错）的 thunk 在结果数组中解析为 `null` — 调用本身绝不 reject，因此使用结果前先 `.filter(Boolean)`。只在确实需要同时拿到全部结果时使用。
- log(message: string): void — emit a progress message to the user (shown as a narrator line above the progress tree)
  log(message: string): void — 向用户发出一条进度消息（显示为进度树上方的叙述行）
- phase(title: string): void — start a new phase; subsequent agent() calls are grouped under this title in the progress display
  phase(title: string): void — 开始一个新阶段；后续的 agent() 调用在进度显示中归入该标题之下
- args: any — the value passed as Workflow's `args` input, verbatim (undefined if not provided). Pass arrays/objects as actual JSON values in the tool call, NOT as a JSON-encoded string — `args: ["a.ts", "b.ts"]`, not `args: "[\"a.ts\", ...]"` (a stringified list reaches the script as one string, so `args.filter`/`args.map` throw). Use this to parameterize named workflows — e.g. pass a research question, target path, or config object directly instead of via a side-channel file.
  args: any — 作为 Workflow 的 `args` 输入传入的值，原样传递（未提供时为 undefined）。在工具调用中把数组/对象作为实际 JSON 值传入，而不是 JSON 编码的字符串 — 用 `args: ["a.ts", "b.ts"]`，而不是 `args: "[\"a.ts\", ...]"`（字符串化的列表到达脚本时是一个字符串，`args.filter`/`args.map` 会抛错）。用它来参数化命名工作流 — 例如直接传入研究问题、目标路径或配置对象，而不是通过旁路文件。
- budget: {total: number|null, spent(): number, remaining(): number} — the turn's token target from the user's "+500k"-style directive. `budget.total` is null if no target was set. `budget.spent()` returns output tokens spent this turn across the main loop and all workflows — the pool is shared, not per-workflow. `budget.remaining()` returns `max(0, total - spent())`, or `Infinity` if no target. The target is a HARD ceiling, not advisory: once `spent()` reaches `total`, further `agent()` calls throw. Use for dynamic loops: `while (budget.total && budget.remaining() > 50_000) { ... }`, or static scaling: `const FLEET = budget.total ? Math.floor(budget.total / 100_000) : 5`.
  budget: {total: number|null, spent(): number, remaining(): number} — 本轮的 token 目标，来自用户 "+500k" 之类的指令。未设置目标时 `budget.total` 为 null。`budget.spent()` 返回本轮在主循环和所有工作流上花费的输出 token — 池是共享的，不按工作流划分。`budget.remaining()` 返回 `max(0, total - spent())`，无目标时为 `Infinity`。该目标是硬上限，不是建议值：一旦 `spent()` 达到 `total`，后续 `agent()` 调用会抛错。用于动态循环：`while (budget.total && budget.remaining() > 50_000) { ... }`，或静态缩放：`const FLEET = budget.total ? Math.floor(budget.total / 100_000) : 5`。
- `workflow(nameOrRef: string | {scriptPath: string}, args?: any): Promise<any>` — run another workflow inline as a sub-step and return whatever it returns. Pass a name to invoke a saved workflow (same registry as {name: "..."}), or {scriptPath} to run a script file you Wrote earlier. The child shares this run's concurrency cap, agent counter, abort signal, and token budget — its agents appear under a "▸ name" group in /workflows and its tokens count toward budget.spent(). The args param becomes the child's `args` global. Nesting is one level only: workflow() inside a child throws. Throws on unknown name / unreadable scriptPath / child syntax error; catch to handle gracefully.
  `workflow(nameOrRef: string | {scriptPath: string}, args?: any): Promise<any>` — 将另一个工作流作为子步骤内联运行，并返回其返回值。传名称以调用已保存的工作流（与 {name: "..."} 相同的注册表），或传 {scriptPath} 以运行你先前 Write 的脚本文件。子工作流共享本次运行的并发上限、智能体计数器、中止信号和 token 预算 — 它的智能体在 /workflows 中显示在 "▸ name" 分组下，其 token 计入 budget.spent()。args 参数成为子工作流的 `args` 全局值。嵌套只允许一层：在子工作流内调用 workflow() 会抛错。未知名称 / 不可读的 scriptPath / 子脚本语法错误时抛错；可捕获以优雅处理。

Subagents are told their final text IS the return value (not a human-facing message), so they return raw data. For structured output, use the schema option — validation happens at the tool-call layer so the model retries on mismatch.
Schemas need {type: 'object', properties: {...}} at root and required ⊆ properties; unsatisfiable ones throw at agent().

子智能体会被告知它们的最终文本就是返回值（不是面向人类的消息），因此它们返回原始数据。要获得结构化输出，使用 schema 选项 — 校验发生在工具调用层，因此模型在不匹配时会重试。
schema 在根层需要 {type: 'object', properties: {...}}，且 required ⊆ properties；无法满足的 schema 会在 agent() 处抛错。

Workflow agents can reach all session-connected MCP tools via ToolSearch — schemas load on demand per agent. Caveat: interactively-authenticated MCP servers (e.g. claude.ai) may be absent in headless/cron runs.

工作流智能体可以通过 ToolSearch 访问所有已连接会话的 MCP 工具 — schema 按智能体按需加载。注意事项：以交互方式认证的 MCP 服务器（如 claude.ai）在无头/cron 运行中可能缺席。

Subagents get the same CLAUDE.md files injected at start that you did (except built-in agent types that omit them, such as Explore and Plan) — don't tell them to re-read those or paste their rules into the prompt; name the specific rule a stage needs, if any.

子智能体在启动时会获得与你相同的 CLAUDE.md 文件注入（省略这些文件的内置智能体类型除外，如 Explore 和 Plan）— 不要让它们重读这些文件或把其中的规则粘贴进提示词；如果某阶段需要特定规则，直接点名该规则。

Scripts are plain JavaScript, NOT TypeScript — type annotations (`: string[]`), interfaces, and generics fail to parse. The script body runs in an async context — use await directly. Standard JS built-ins (JSON, Math, Array, etc.) are available — EXCEPT `Date.now()`/`Math.random()`/argless `new Date()`, which throw (they would break resume); pass timestamps in via `args`, stamp results after the workflow returns, and for randomness vary the agent prompt/label by index. No filesystem or Node.js API access.

脚本是纯 JavaScript，不是 TypeScript — 类型注解（`: string[]`）、接口和泛型无法解析。脚本主体运行在 async 上下文中 — 直接使用 await。标准 JS 内置对象（JSON、Math、Array 等）可用 — 但 `Date.now()`/`Math.random()`/无参 `new Date()` 除外，它们会抛错（会破坏断点续跑）；时间戳通过 `args` 传入，或在工作流返回后再标注结果，需要随机性时按索引改变智能体的提示词/标签。无文件系统或 Node.js API 访问权限。
【评论】禁用时间与随机 API 是为断点续跑服务的确定性约束：重放时脚本必须产生与首次一致的调用序列。

DEFAULT TO pipeline(). Only reach for a barrier (parallel between stages) when you genuinely need ALL prior-stage results together.

默认使用 pipeline()。只有当你确实需要同时拿到上一阶段的全部结果时，才使用屏障（阶段间的 parallel）。

A barrier is correct ONLY when stage N needs cross-item context from all of stage N-1:

只有当阶段 N 需要来自阶段 N-1 全部结果的跨条目上下文时，屏障才是正确的：

- Dedup/merge across the full result set before expensive downstream work
  在昂贵的下游工作之前，对完整结果集做去重/合并
- Early-exit if the total count is zero ("0 bugs found → skip verification entirely")
  总数为零时提前退出（"发现 0 个 bug → 完全跳过核验"）
- Stage N's prompt references "the other findings" for comparison
  阶段 N 的提示词引用"其他发现"进行比较

A barrier is NOT justified by:

以下理由不能证明屏障的合理性：

- "I need to flatten/map/filter first" — do it inside a pipeline stage: pipeline(items, stageA, r => transform([r]).flat(), stageB)
  "我需要先 flatten/map/filter" — 在 pipeline 阶段内部做：pipeline(items, stageA, r => transform([r]).flat(), stageB)
- "The stages are conceptually separate" — that's what pipeline() models. Separate stages ≠ synchronized stages.
  "这些阶段在概念上是分开的" — 这正是 pipeline() 所建模的东西。分开的阶段 ≠ 同步的阶段。
- "It's cleaner code" — barrier latency is real. If 5 finders run and the slowest takes 3× the fastest, a barrier wastes 2/3 of the fast finders' idle time.
  "这样代码更整洁" — 屏障的延迟是真实存在的。如果 5 个查找器中最慢的耗时是最快的 3 倍，屏障会浪费掉快查找器 2/3 的空闲时间。

Smell test: if you wrote

坏味道检验：如果你写了

  const a = await parallel(...)
  const b = transform(a)        // flatten, map, filter — no cross-item dependency
  const c = await parallel(b.map(...))
that middle transform doesn't need the barrier. Rewrite as a pipeline with the transform inside a stage. When in doubt: pipeline.

那么中间那个 transform 并不需要屏障。改写为 pipeline，把 transform 放进某个阶段内部。拿不准时：用 pipeline。

Concurrent agent() calls are capped at min(16, available CPUs - 2) per workflow — excess calls queue and run as slots free up. You can still pass 100 items to parallel()/pipeline() and they all complete; only ~10 run at any moment. Total agent count across a workflow's lifetime is capped at 1000 — a runaway-loop backstop set far above any real workflow. A single parallel()/pipeline() call accepts at most 4096 items; passing more is an explicit error, not a silent truncation.

每个工作流的并发 agent() 调用上限为 min(16, 可用 CPU 数 - 2) — 超出的调用会排队，在空位释放后运行。你仍然可以把 100 个条目传给 parallel()/pipeline()，它们都会完成；只是任一时刻只有约 10 个在运行。一个工作流生命周期内的智能体总数上限为 1000 — 这是为失控循环设置的后备保险，远高于任何真实工作流的需要。单次 parallel()/pipeline() 调用最多接受 4096 个条目；传更多会显式报错，而不是静默截断。

When a barrier IS correct — dedup across all findings before expensive verification:

屏障正确的场景 — 在昂贵的核验之前对全部发现去重：

  const all = await parallel(DIMENSIONS.map(d => () => agent(d.prompt, {schema: FINDINGS_SCHEMA})))
  const deduped = dedupeByFileAndLine(all.filter(Boolean).flatMap(r => r.findings))  // <-- genuinely needs ALL at once
  const verified = await parallel(deduped.map(f => () => agent(verifyPrompt(f), {schema: VERDICT_SCHEMA})))

Loop-until-count pattern — accumulate to a target:

循环直到达到数量模式 — 向目标累积：

  const bugs = []
  while (bugs.length < 10) {
    const result = await agent("Find bugs in this codebase.", {schema: BUGS_SCHEMA})
    bugs.push(...result.bugs)
    log(`${bugs.length}/10 found`)
  }

Loop-until-budget pattern — scale depth to the user's "+500k" directive. Guard on budget.total: with no target set, remaining() is Infinity and the loop would run straight to the 1000-agent cap.

循环直到预算耗尽模式 — 深度随用户 "+500k" 之类的指令伸缩。要以 budget.total 作为守卫条件：未设置目标时 remaining() 为 Infinity，循环会一路跑满 1000 个智能体的上限。

  const bugs = []
  while (budget.total && budget.remaining() > 50_000) {
    const result = await agent("Find bugs in this codebase.", {schema: BUGS_SCHEMA})
    bugs.push(...result.bugs)
    log(`${bugs.length} found, ${Math.round(budget.remaining()/1000)}k remaining`)
  }

Composing patterns — exhaustive review (find → dedup vs seen → diverse-lens panel → loop-until-dry):

组合模式 — 详尽评审（查找 → 与 seen 去重 → 多视角评审团 → 循环直到无新发现）：

  const seen = new Set(), confirmed = []
  let dry = 0
  while (dry < 2) {                                              // loop-until-dry
    const found = (await parallel(FINDERS.map(f => () =>          // barrier: collect all finders this round
      agent(f.prompt, {phase: 'Find', schema: BUGS})))).filter(Boolean).flatMap(r => r.bugs)
    const fresh = found.filter(b => !seen.has(key(b)))           // dedup vs ALL seen — plain code, not an agent
    if (!fresh.length) { dry++; continue }
    dry = 0; fresh.forEach(b => seen.add(key(b)))
    const judged = await parallel(fresh.map(b => () =>           // every fresh bug judged concurrently...
      parallel(['correctness','security','repro'].map(lens => () =>   // ...each by 3 distinct lenses
        agent(`Judge "${b.desc}" via the ${lens} lens — real?`, {phase: 'Verify', schema: VERDICT})))
        .then(vs => ({ b, real: vs.filter(Boolean).filter(v => v.real).length >= 2 }))))
    confirmed.push(...judged.filter(v => v.real).map(v => v.b))
  }
  return confirmed
  // dedup vs `seen`, NOT `confirmed` — else judge-rejected findings reappear every round and it never converges.

Quality patterns — common shapes; pick by task and compose freely:

质量模式 — 常见形态；按任务挑选，自由组合：

- Adversarial verify: spawn N independent skeptics per finding, each prompted to REFUTE. Kill if ≥majority refute. Prevents plausible-but-wrong findings from surviving.
  对抗性核验：为每个发现生成 N 个独立的怀疑者，每个都被提示去反驳（REFUTE）。若 ≥ 多数反驳则剔除。防止看似合理但错误的发现存活下来。
【评论】"不确定时默认已反驳"式的对抗性核验是保守方向的验证设计：把举证责任交给发现的提出方，以压低假阳性率。

    const votes = await parallel(Array.from({length: 3}, () => () =>
      agent(`Try to refute: ${claim}. Default to refuted=true if uncertain.`, {schema: VERDICT})))
    const survives = votes.filter(Boolean).filter(v => !v.refuted).length >= 2
- Perspective-diverse verify: when a finding can fail in more than one way, give each verifier a distinct lens (correctness, security, perf, does-it-reproduce) instead of N identical refuters — diversity catches failure modes redundancy can't.
  视角多样化核验：当一个发现可能以多种方式出错时，给每个核验者一个不同的视角（正确性、安全、性能、能否复现），而不是 N 个相同的反驳者 — 多样性能捕捉到冗余捕捉不到的失败模式。
- Judge panel: generate N independent attempts from different angles (e.g. MVP-first, risk-first, user-first), score with parallel judges, synthesize from the winner while grafting the best ideas from runners-up. Beats one-attempt-iterated when the solution space is wide.
  评审团：从不同角度生成 N 个独立尝试（如 MVP 优先、风险优先、用户优先），由并行评审打分，从胜出者合成，同时嫁接亚军方案的最佳想法。当解空间很宽时优于单方案反复迭代。
- Loop-until-dry: for unknown-size discovery (bugs, issues, edge cases), keep spawning finders until K consecutive rounds return nothing new. Simple counters (while count < N) miss the tail.
  循环直到无新发现：对未知规模的发现任务（bug、问题、边界情况），持续生成查找器，直到连续 K 轮没有新发现。简单的计数器（while count < N）会漏掉尾部。
- Multi-modal sweep: parallel agents each searching a different way (by-container, by-content, by-entity, by-time). Each is blind to what the others surface; useful when one search angle won't find everything.
  多模式扫查：并行的智能体各自用不同方式搜索（按容器、按内容、按实体、按时间）。每个智能体都看不到其他智能体的产出；当单一搜索角度无法找全内容时有用。
- Completeness critic: a final agent that asks "what's missing — modality not run, claim unverified, source unread?" What it finds becomes the next round of work.
  完备性批评者：一个最终智能体，负责追问"还缺什么 — 哪种方式没跑、哪个论断未核验、哪个来源未读？"它发现的内容成为下一轮工作。
- No silent caps: if a workflow bounds coverage (top-N, no-retry, sampling), `log()` what was dropped — silent truncation reads as "covered everything" when it didn't.
  不做静默截断：如果工作流限制了覆盖范围（top-N、不重试、抽样），用 `log()` 记录被丢弃的内容 — 静默截断会让人误以为"已覆盖全部"，实际并没有。

Scale to what the user asked for. "find any bugs" → a few finders, single-vote verify. "thoroughly audit this" or "be comprehensive" → larger finder pool, 3–5 vote adversarial pass, synthesis stage. When unsure, lean toward thoroughness for research/review/audit requests and toward brevity for quick checks.

按用户要求的规模伸缩。"找找有没有 bug" → 少量查找器、单票核验。"彻底审计这个"或"要全面" → 更大的查找器池、3–5 票的对抗性核验、合成阶段。拿不准时，研究/评审/审计类请求倾向于彻底，快速检查类倾向于简洁。

These patterns aren't exhaustive — compose novel harnesses when the task calls for it (tournament brackets, self-repair loops, staged escalation, whatever fits).

这些模式并非穷尽 — 当任务需要时可以组合出新的执行框架（锦标赛对阵、自修复循环、分阶段升级，任何合适的形式）。

Use this tool for multi-step orchestration where control flow should be deterministic (loops, conditionals, fan-out) rather than model-driven.

当控制流应当是确定性的（循环、条件、扇出）而非模型驱动时，使用此工具进行多步骤编排。

## Resume / 断点续跑

The tool result includes a runId. To resume after a pause, kill, or script edit, relaunch with Workflow({scriptPath, resumeFromRunId}) — the longest unchanged prefix of agent() calls returns cached results instantly; the first edited/new call and everything after it runs live. Same script + same args → 100% cache hit. Before diagnosing why a completed workflow returned an empty or unexpected result, Read `<transcriptDir>`/journal.jsonl — it records each agent's actual return value; do not assume cached results are non-empty. Date.now()/Math.random()/new Date() are unavailable in scripts (they would break this) — stamp results after the workflow returns, or pass timestamps via args. Fallback when no journal is available: Read agent-`<id>`.jsonl files in the transcript directory and hand-author a continuation script.

工具结果中包含 runId。要在暂停、终止或脚本编辑之后续跑，用 Workflow({scriptPath, resumeFromRunId}) 重新启动 — agent() 调用中最长的未改变前缀会立即返回缓存结果；第一个被编辑/新增的调用及其之后的一切会真实运行。相同脚本 + 相同 args → 100% 缓存命中。在诊断一个已完成工作流为何返回空或意外结果之前，先 Read `<transcriptDir>`/journal.jsonl — 它记录了每个智能体的实际返回值；不要假设缓存结果非空。脚本中不可用 Date.now()/Math.random()/new Date()（它们会破坏该机制）— 在工作流返回后再标注结果，或通过 args 传入时间戳。没有 journal 可用时的后备方案：Read 转录目录中的 agent-`<id>`.jsonl 文件，手工编写续跑脚本。
