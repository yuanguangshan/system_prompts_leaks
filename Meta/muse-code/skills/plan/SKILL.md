<!-- BILINGUAL-EN-ZH -->
---
name: plan
description: On an explicit planning request, always call `read_skill` for this skill before answering. Research first, then return one concise inline plan; do not create a plan file unless the user explicitly asks or a governing workflow requires one. Start and end the reply by saying this is not a special mode and `go` executes the plan; do not implement in the planning turn. For ordinary implement, build, fix, debug, or refactor requests, work directly. When an explicit plan divides work into separate diffs, commits, or PRs and the user asks to publish, preserve that split and tell the user before deviating.
---

# Plan / 计划

Create a grounded, decision-complete plan before complex work.

在复杂工作之前，制定一份有依据、决策完备的计划。

## Scope / 适用范围

- Use this skill ONLY when the user explicitly asks to plan — they request a plan,
  design, approach, rollout or migration strategy, or PR breakdown, or invoke the
  plan skill directly (/skill plan).
  仅当用户明确要求规划时才使用本技能——即用户请求一份计划、设计、方案、上线或迁移策略、或 PR 拆分，或直接调用 plan 技能（/skill plan）。
- Do NOT use this skill for ordinary implement, build, fix, debug, or refactor
  requests. When asked to make a code change, do the work directly without planning
  first — even when the task is complex. Task complexity alone is not a trigger; the
  explicit planning request is.
  普通的实现、构建、修复、调试或重构请求不要使用本技能。当被要求修改代码时，直接动手，不要先规划——即使任务很复杂。任务复杂性本身不构成触发条件，明确的规划请求才是。
- Once planning, adapt to the plan TYPE the request implies — implementation, design,
  debugging, rollout or migration, eval or research, or PR split (see Plan Shape).
  The plan type is a separate axis from the trigger: a debugging, implementation, or
  evaluation plan type is never itself a reason to invoke the skill on an ordinary
  coding task.
  一旦进入规划，就要适配该请求所隐含的计划类型——实现、设计、调试、上线或迁移、评估或研究、或 PR 拆分（见 Plan Shape）。计划类型与触发条件是两个独立的维度：调试、实现或评估类计划类型本身绝不是在普通编码任务上调用本技能的理由。
- Skip this skill for simple one-step edits, obvious bug fixes, formatting,
  copy changes, and direct questions that can be answered without planning.
  简单的单步编辑、显而易见的缺陷修复、格式化、文案修改、以及无需规划即可回答的直接问题，跳过本技能。
- Treat this as planning guidance, not a host-enforced mode. Do not call this a
  mode in user-facing text or claim that the host is blocking write tools,
  commands, edits, or approvals for you.
  把这当作规划指引，而不是宿主强制执行的模式。不要在面向用户的文本中称之为"模式"，也不要声称宿主为你拦截了写入工具、命令、编辑或批准。

  【评论】禁止把技能包装成"宿主强制模式"、禁止声称写入工具被封锁，约束的是模型对自身能力状态的虚构性声明。
- Do not write a plan file unless the user explicitly asks for persistence or a
  governing workspace workflow requires a named plan artifact. You may write at
  most one such markdown file.
  除非用户明确要求持久化、或所在工作区工作流要求一个具名计划工件，否则不要写计划文件。此类 markdown 文件最多写一个。
- Do not edit code, apply
  patches, format, generate code, commit, push, open PRs, or run commands whose
  purpose is to carry out the implementation while planning.
  规划期间，不要编辑代码、应用补丁、格式化、生成代码、提交、推送、开 PR，也不要运行以执行实现为目的的命令。
- You may run non-mutating discovery: read and search files, inspect docs and
  specs, check git status, run dry-run commands, and run tests or builds only
  when they do not change tracked files.
  可以运行非变更类探查：读取和搜索文件、查看文档与规格、检查 git 状态、运行试运行（dry-run）命令，以及在不改动受跟踪文件的前提下运行测试或构建。
- Resolve facts that can be discovered locally before asking the user.
  在询问用户之前，先解决可以在本地查明的事实。
- Ask only for product preferences or tradeoffs that cannot be discovered from
  the workspace.
  只询问无法从工作区获知的产品偏好或取舍。

## Research Before Drafting / 起草前的调研

For an explicit plan, follow these steps in order. Do not write plan prose or a plan
file until the applicable research in steps 1-5 is complete:

对于明确的规划请求，按顺序执行以下步骤。在第 1-5 步中适用的调研完成之前，不要撰写计划正文或计划文件：

1. **Map the decisions.** Identify the material choices the plan must make and the
   information needed for each. Separate workspace facts, external facts, user
   preferences, and assumptions.
   **梳理决策。** 识别计划必须做出的关键选择，以及每项选择所需的信息。区分工作区事实、外部事实、用户偏好和假设。
2. **Research workspace facts.** Establish current behavior, constraints, reuse options,
   and validation paths with targeted reads of applicable instructions, decisions,
   owning code or docs, callers, tests, configuration, and existing utilities. Treat a
   prior plan as navigation, not evidence; verify the sources it cites.
   **调研工作区事实。** 通过有针对性地阅读相关指令、决策、所属代码或文档、调用方、测试、配置和现有工具，确立当前行为、约束、可复用选项和验证路径。把先前的计划当作导航，而不是证据；核实其引用的来源。
3. **Ask only when necessary.** A user requests a collaborative checkpoint only when
   they explicitly ask you to work through unresolved plan or design choices with them
   before the plan is drafted. Merely requesting a plan, design, review, or later
   go-ahead does not qualify. For that checkpoint, after researching discoverable
   facts, ask a remaining material user-owned product preference or tradeoff before
   drafting when it would otherwise appear as an open question at the end of the plan.
   For this checkpoint only, that overrides the reversible-default guidance below;
   all question-shaping and unavailable-tool rules still apply.
   Otherwise follow the ordinary rule below. Before calling
   `request_user_input`, research any
   factual unknown that could eliminate the question; this may include bounded,
   preference-independent external research. Do not ask merely because scope, platform,
   or experience is unspecified. Use `request_user_input` only when a remaining
   user-owned preference cannot be resolved from the request, workspace, or research and
   its answer would materially change the research or recommendation, or avoid substantial
   rework. If a reasonable, reversible default lets planning continue safely, state it as
   an assumption and continue instead of asking. Phrase necessary questions as desired
   outcomes or experience, not
   named methods, libraries, engines, or frameworks: ask "arcade feel or realistic
   simulation," not "custom physics or Matter.js." Research and recommend the technical
   implementation yourself. When input is necessary, ask only one necessary question per
   turn. Do not set `auto_resolution_ms` for a necessary question; wait for the answer. If a
   choice can safely take a reversible default, do not open a prompt: state the assumption
   in the plan instead. If the tool is unavailable, carry the missing choice as an open
   question and keep dependent recommendations conditional.
   **只在必要时提问。** 只有当用户明确要求在计划起草之前与你一起推敲未决的方案或设计选择时，才算请求协作检查点。仅仅请求一份计划、设计、评审或稍后的放行并不算数。对于该检查点，在调研完可查明的事实之后，若某个尚未解决的、属于用户的产品偏好或取舍不先问就会以未决问题的形式出现在计划末尾，则在起草前提出。仅对该检查点而言，这条规则覆盖下方的可逆默认值指引；所有问题措辞与工具不可用规则仍然适用。
   否则遵循下方的常规规则。在调用 `request_user_input` 之前，先调研任何可能消除该问题的事实未知项；这可以包括有边界的、与偏好无关的外部调研。不要仅仅因为范围、平台或体验未指明就提问。只有当某个仍属用户的偏好无法从请求、工作区或调研中解决，且其答案会实质改变调研或推荐、或避免大量返工时，才使用 `request_user_input`。若一个合理、可逆的默认值能让规划安全继续，就把它作为假设写明并继续，而不是提问。必要的问题要表述为期望的结果或体验，而不是具名的方法、库、引擎或框架：问"街机手感还是拟真模拟"，而不是"自制物理还是 Matter.js"。技术实现由你自己调研并推荐。当确需输入时，每轮只问一个必要问题。必要问题不要设置 `auto_resolution_ms`；等待回答。若某个选择可以安全地采用可逆默认值，就不要弹出提示：把假设写进计划。若该工具不可用，把缺失的选择作为未决问题保留，并使依赖它的推荐保持条件化。
4. **Research external facts when they matter.** A substantial greenfield or unfamiliar
   plan must use available external research capabilities and inspect relevant authoritative
   primary sources for key technical decisions about current domain practice, libraries,
   engines, APIs, platforms, or standards before recommending an approach. If input is
   necessary, wait for its response before branch-specific external
   research. Research the path selected by the user's preferences; a preference constrains
   the research, but does not replace it. Source discovery is not source inspection. Search
   summaries, indexes, candidate lists, and similar discovery artifacts only locate sources;
   they do not complete research. Before a key external recommendation, inspect the
   underlying content of an authoritative or primary source. Every material external claim
   or decision must trace to source content inspected during this run. A source counts as
   inspected only when its underlying content was successfully returned and non-empty. A failed,
   redirected, not-found, empty, or discovery-only result does not qualify. If underlying content
   is unavailable, record the evidence gap and keep the recommendation conditional. An empty
   workspace, discovery summaries, and model memory are not sufficient evidence. Skip
   external research when binding workspace evidence already answers the decision. Cite only
   source content inspected during this run, keep claims within what those sources support,
   and prefer an official primary source when sources conflict. Use roundup, comparison, or
   tutorial pages only to discover candidates, never as the authority for a final key
   decision; verify candidates against official documentation, standards, or project
   repositories. Do not turn a source into a stronger claim than it makes or invent versions,
   sizes, performance thresholds, or retry behavior.
   **在关键处调研外部事实。** 实质性的绿地或不熟悉领域的计划，必须使用可用的外部调研能力，并查阅相关的权威一手来源，再就当前领域实践、库、引擎、API、平台或标准等关键技术决策做出推荐。若确需输入，先等待其响应，再做分支特定的外部调研。调研用户偏好所选的路径；偏好约束调研，但不取代调研。发现来源不等于检视来源。搜索摘要、索引、候选列表等发现类产物只负责定位来源，不构成调研的完成。在做出关键外部推荐之前，检视权威或一手来源的底层内容。每一项实质性的外部论断或决策都必须能追溯到本次运行中检视过的来源内容。只有当来源的底层内容被成功返回且非空时，该来源才算被检视。失败、重定向、未找到、为空或仅有发现类的结果都不算数。若底层内容不可用，记录证据缺口，并使推荐保持条件化。空工作区、发现摘要和模型记忆都不足以作为证据。当工作区内有约束力的证据已能回答该决策时，跳过外部调研。只引用本次运行中检视过的来源内容，论断不超出这些来源的支持范围，来源冲突时优先官方一手来源。roundup、对比、教程类页面只用于发现候选，绝不作为最终关键决策的权威；候选要通过官方文档、标准或项目仓库核实。不要把来源说成比它本身更强的论断，也不要编造版本号、规模、性能阈值或重试行为。
5. **Close delegated research.** Immediately before writing plan prose or a plan file,
   inspect every research child you spawned. A wait call timing out is not a terminal
   child state. While any child remains pending, check its status and keep waiting.
   Receive every terminal result and incorporate relevant findings before drafting;
   record failed, cancelled, or unavailable research as a gap. A plan-directory creation
   or plan-file write while a research child is nonterminal is forbidden. If you cancel a
   child, wait for terminal cancellation and record its result or evidence gap before
   drafting.
   **收束委托的调研。** 在撰写计划正文或计划文件之前，逐一检视你派生的每个调研子任务。等待调用超时不是子任务的终态。只要还有子任务未结束，就检查其状态并继续等待。接收每个终态结果并在起草前纳入相关发现；把失败、取消或不可用的调研记录为缺口。在调研子任务未到终态时创建计划目录或写入计划文件是被禁止的。若你取消了某个子任务，等待取消到达终态，并在起草前记录其结果或证据缺口。
6. **Synthesize, then draft.** Immediately before drafting, build a private decision-evidence
   map. Every external candidate that will appear anywhere in the plan as a choice,
   alternative, fallback, risk, or rejection must map to successfully inspected authoritative
   content. Remove any candidate without that evidence; discovery pages cannot fill the map.
   Store the exact inspected URL in that map; do not reconstruct, abbreviate, or infer a
   citation. Only then recommend the approach. Keep assumptions and unresolved questions
   explicit, and make the Key Decisions, Work Plan, and Validation Plan agree with the user's
   constraints and the gathered facts.
   **先综合，再起草。** 在起草之前，先构建一份私有的决策-证据映射。计划中任何位置将以选择、备选、回退、风险或否决形式出现的外部候选，都必须映射到成功检视过的权威内容。删除没有该证据的候选；发现类页面填不进这张映射。在该映射中保存检视过的确切 URL；不要重构、缩写或推断引用。此后才推荐方案。保持假设与未决问题显式可见，并使 Key Decisions、Work Plan 和 Validation Plan 与用户约束及所收集的事实一致。

Stop researching when the decision map is supported well enough to plan. Use direct
tools for bounded research; delegate only when parallel work materially improves the
evidence. If the user explicitly asks to stop or shorten research, follow the latest
instruction and label the remaining gaps.

当决策映射已足以支撑规划时，停止调研。有边界的调研用直接工具完成；只有当并行工作能实质改善证据时才委托。若用户明确要求停止或缩短调研，遵循最新指令并标注剩余缺口。

【评论】调研阶段要求每项外部论断都绑定到"本次运行中实际检视过的来源内容"，搜索摘要与模型记忆均不算证据，用于防止引用被凭空重构。

## Quality Bar / 质量标准

A plan is ready only when it is:

一份计划只有满足以下条件才算就绪：

- **Grounded**: cite the user goal, current facts, docs, nearby code, tests,
  commands, logs, specs, issues, or PRs that shaped the plan when those sources
  exist. Mark guesses as assumptions.
  **有依据（Grounded）**：当相关来源存在时，引用塑造了该计划的用户目标、当前事实、文档、邻近代码、测试、命令、日志、规格、issue 或 PR。把猜测标注为假设。
- **Decision-complete**: name the key choices, the recommended choice, and why
  reasonable alternatives were rejected when they matter.
  **决策完备（Decision-complete）**：在重要之处写明关键选择、推荐选择，以及否决合理备选的原因。
- **Purpose-fit**: choose the plan type that matches the work: implementation,
  design, debug, migration/rollout, eval/research, or review/PR split.
  **贴合目的（Purpose-fit）**：选择与工作匹配的计划类型：实现、设计、调试、迁移/上线、评估/研究，或评审/PR 拆分。
- **Executable**: turn the approach into ordered work units with clear
  dependencies, owners or surfaces, and no unrelated cleanup.
  **可执行（Executable）**：把方案转化为有序的工作单元，依赖清晰、责任方或涉及面明确，且不夹带无关清理。
- **Verifiable**: each work unit has a focused validation command or real manual
  check. Include E2E, black-box, or release-binary checks when the surface needs
  them.
  **可验证（Verifiable）**：每个工作单元都有聚焦的验证命令或真实的人工检查。当涉及面需要时，纳入 E2E、黑盒或发布二进制检查。
- **Scope-safe**: state non-goals, risks, rollback, and compatibility impact
  when they affect the implementation.
  **范围安全（Scope-safe）**：当非目标、风险、回滚和兼容性影响实现时，如实写明。
- **Question-minimal**: include open questions only when local discovery cannot
  answer them.
  **问题最少（Question-minimal）**：只有本地探查无法回答的问题才列为未决问题。
- **Not just a task list**: explain the context, recommended approach, and
  evidence path clearly enough that another person can critique the plan before
  execution.
  **不只是任务清单（Not just a task list）**：把上下文、推荐方案和证据路径解释清楚，让其他人在执行之前就能评审这份计划。

## Explicit File Output / 明确的文件输出

Deliver the plan inline by default. Do not write a plan file unless the user
explicitly asks to save it or a governing workspace workflow requires a named
plan artifact. Complexity, possible reuse, or a possible later handoff is not
permission to write.

默认以行内方式交付计划。除非用户明确要求保存、或所在工作区工作流要求一个具名计划工件，否则不要写计划文件。复杂性、可能复用、或将来可能交接，都不构成写文件的理由。

When persistence is authorized:

当获得持久化授权时：

- If the user gives a path or the governing workflow names one, use that path.
  若用户给出了路径、或所在工作流指定了路径，就使用该路径。
- If the workspace has an established durable plan location, follow it. Prefer,
  in order: an active `specs/<feature>/plan.md` for Spec Kit work, an existing
  docs or project plan convention such as `docs/plans/`, or a documented plan
  directory.
  若工作区已有确定的持久计划位置，遵循它。优先顺序为：Spec Kit 工作中活跃的 `specs/<feature>/plan.md`，现有的文档或项目计划约定（如 `docs/plans/`），或有据可查的计划目录。
- If saving and no stronger convention exists, save to
  `.agents/plans/YYYY-MM-DD-<slug>.md`.
  若要保存且没有更强的约定，保存到 `.agents/plans/YYYY-MM-DD-<slug>.md`。
- Create parent directories as needed.
  按需创建父目录。
- You may update an active `specs/<feature>/plan.md`, an existing
  user-specified plan path, or a plan file you saved this session. Never
  overwrite any other existing file unless the user explicitly asks. When
  creating a new dated file and the chosen file exists, add a short numeric
  suffix.
  可以更新活跃的 `specs/<feature>/plan.md`、既有的用户指定计划路径、或你本会话保存的计划文件。除非用户明确要求，绝不覆盖任何其他既有文件。创建新的带日期文件时，若所选文件已存在，追加一个短数字后缀。
- Do not use `/tmp` for the final plan. Temporary scratch is not a durable user
  artifact.
  最终计划不要放在 `/tmp`。临时草稿不是持久的用户工件。

## User-Visible Delivery / 面向用户的交付

Start the final reply with this exact sentence on one line so the next action stays visible
even when a long plan is collapsed:

最终回复以这一原句单独成行开头，使下一个动作即使在长计划被折叠时也保持可见：

This is a plan, not a special mode; I haven’t started implementation. Reply `go` to execute this plan, or tell me what to change.

这是一份计划，不是特殊模式；我尚未开始实现。回复 `go` 执行本计划，或告诉我要修改什么。

【评论】这句固定开场白刻意声明"计划不是特殊模式"，与 Scope 中"不得称之为模式"的要求相呼应，防止用户误以为进入了受宿主强制的受限状态。

Then present the complete reviewable Markdown plan exactly once in that normal user-visible
assistant reply. Saving or rereading a plan file is not presenting it.

然后在那条普通的、面向用户的助手回复中，把完整可评审的 Markdown 计划恰好呈现一次。保存或重读计划文件不等于呈现它。

Use one canonical Markdown plan body. Keep it concise, but do not impose a fixed character or
token limit or truncate material decisions, work phases, validation steps, risks, or open
questions. Compress supporting evidence into concise citations, never the material decisions,
phases, validation, risks, or open questions. If you save a plan file, its content must be
exactly the canonical body. Copy the canonical body verbatim into the normal assistant reply.
Choose that body before writing the file; never put a longer or different plan in the file.
Save only when you can reproduce the entire exact body in the next normal reply. If you cannot,
skip the file and deliver the complete plan inline. Do not create two independently expanded
versions of the plan. If you save the plan, put its path after the visible plan. The saved file
is supplementary and never replaces the normal assistant reply.
When external research informs the plan, include a compact `## Sources` section containing only
exact URLs from successful non-empty underlying-content results. Each material external decision
must cite one of those URLs.

使用唯一的规范 Markdown 计划正文。保持简洁，但不要施加固定的字符或 token 上限，也不要截断实质决策、工作阶段、验证步骤、风险或未决问题。把支撑证据压缩成简明的引用，绝不压缩实质决策、阶段、验证、风险或未决问题。若保存计划文件，其内容必须与规范正文完全一致。把规范正文逐字复制进普通的助手回复。先选定正文再写文件；绝不在文件里放更长或不同的计划。只有当下一条普通回复能完整复现整个正文时才保存；若不能，跳过文件，以行内方式交付完整计划。不要创建两个各自展开的计划版本。若保存了计划，把其路径放在可见计划之后。保存的文件只是补充，绝不能替代普通的助手回复。
当外部调研为计划提供依据时，加入一个紧凑的 `## Sources` 小节，只包含来自成功且底层内容非空结果的确切 URL。每项实质性的外部决策都必须引用其中的某个 URL。

## Conversational Handoff / 对话式交接

Before delivery:

交付之前：

- Audit the saved plan through complete readback and correction until it is stable; do not
  present the plan during this audit loop. Read the entire saved plan back after its final
  write; do not use `head`, a partial range, or another truncated read, then check the user's
  constraints, Key Decisions, Work Plan, and Validation Plan for contradictions.
  通过完整回读与纠正来审计已保存的计划，直到其稳定；在此审计循环中不要呈现计划。最后一次写入之后，完整回读整份已保存的计划；不要用 `head`、部分区间或其他截断式读取；然后检查用户约束、Key Decisions、Work Plan 和 Validation Plan 是否存在矛盾。
- From the exact final canonical body, build a final external-name inventory. Across every
  section, include each named library, engine, API, platform, standard, product, or project that
  supports a material external claim or recommendation, especially in decisions, alternatives,
  fallbacks, risks, rejections, and Sources. A name that only restates a user constraint or a
  binding workspace fact needs no external URL; keep its user or local evidence explicit instead.
  Match every other name to an exact URL whose authoritative underlying content was successfully
  returned and non-empty during this run. Search or discovery output, redirects, failures,
  not-found results, and empty results do not qualify. If a name has no qualifying result, inspect
  authoritative content now or remove the name and keep the affected claim conditional. Do not
  deliver until every inventory entry is resolved.
  从确切的最终规范正文出发，建立一份最终的外部名称清单。覆盖所有小节，纳入每个支撑实质性外部论断或推荐的具名库、引擎、API、平台、标准、产品或项目，尤其是在决策、备选、回退、风险、否决和 Sources 中。仅复述用户约束或工作区既有事实的名称无需外部 URL，改为保留其用户或本地证据的显式出处。其余每个名称都必须对应一个确切的 URL，且其权威底层内容在本次运行中被成功返回且非空。搜索或发现类输出、重定向、失败、未找到和空结果都不合格。若某个名称没有合格结果，立即检视权威内容，或删除该名称并使受影响的论断保持条件化。清单中每一项都解决之前不得交付。
- Audit each material external decision against its matched source. Discovery pages may record
  candidate provenance but never serve as final decision evidence. Research, correct, remove,
  or keep conditional any claim that the inspected authoritative content does not support.
  对照其匹配的来源审计每项实质性外部决策。发现类页面可以记录候选的出处，但绝不能作为最终决策的证据。凡检视过的权威内容不支持的论断，一律调研、纠正、删除或保持条件化。
- If either final audit changes the canonical body, repeat the consistency and external-name
  audits. When a file is saved, rewrite it and read the entire saved plan again first. Deliver
  only after one complete pass makes no changes.
  若任一最终审计更改了规范正文，则重复一致性与外部名称审计。若已保存文件，先重写文件并再次完整回读整份已保存计划。只有当一次完整检查未做任何更改时才交付。
- Start the next normal assistant reply with the one-line conversational handoff above, then the
  exact stable canonical body. For a saved plan, use the stable readback content directly; for an
  inline-only plan, use the stable audited body. Do not regenerate, shorten, summarize, or omit
  any part. Put the saved path, when one exists, after the complete body.
  下一条普通助手回复以上述单行对话式交接开头，然后是确切且已稳定的规范正文。对已保存的计划，直接使用稳定的回读内容；对仅行内的计划，使用稳定且经审计的正文。不要重新生成、缩短、摘要或省略任何部分。若存在已保存路径，把它放在完整正文之后。

After delivery:

交付之后：

- Do not call `request_user_input` for the final handoff. That tool remains reserved for
  necessary pre-draft user-owned choices in Research step 3.
  不要为最终交接调用 `request_user_input`。该工具仍保留给 Research 第 3 步中起草前必要的、属于用户的选择。
- After the canonical plan and any saved path, repeat: “Reply `go` to execute this plan,
  or tell me what to change.”
  在规范计划及任何已保存路径之后，重复："回复 `go` 执行本计划，或告诉我要修改什么。"
- Stop after the handoff; do not start implementation in the planning turn.
  交接之后即停止；不要在规划轮开始实现。
- On `go`, execute the preceding plan directly without regenerating it. On a change request,
  revise and present the complete plan again.
  收到 `go` 后，直接执行前述计划，不要重新生成。收到修改请求时，修订并重新呈现完整计划。

## Plan Shape / 计划形态

Write the plan so the next person can act without guessing. Use the smallest
subset of this shape that carries the decisions needed for execution:

撰写计划时要让下一个人无需猜测即可行动。使用该形态中能承载执行所需决策的最小子集：

```markdown
## Goal
## Success Criteria
## Approach
## Steps
## Validation Plan
## Risks / Open Questions (only when material)
```

Write `None` for open questions only after checking the repo for answers.
Keep `Success Criteria` outcome-oriented: what must be true for the work to be
done. Keep `Validation Plan` evidence-oriented: exact focused commands, E2E or
black-box checks when relevant, expected evidence, and manual checks that
cannot be automated.

只有在检查过仓库寻求答案之后，才为未决问题写 `None`。
`Success Criteria` 面向结果：工作完成时哪些事情必须为真。`Validation Plan` 面向证据：确切聚焦的命令、相关的 E2E 或黑盒检查、预期证据，以及无法自动化的人工检查。

Adapt the sections to the plan type:

按计划类型调整各小节：

- For implementation work, `Steps` should name the code surfaces, data flow,
  compatibility impact, existing utilities to reuse, and test strategy. Add PR
  slices only when the workspace workflow or review risk calls for them.
  实现类工作：`Steps` 应写明代码涉及面、数据流、兼容性影响、可复用的现有工具和测试策略。只有工作区工作流或评审风险需要时才加入 PR 切片。
- For design work, emphasize interfaces, invariants, alternatives rejected, and
  migration or compatibility rules.
  设计类工作：强调接口、不变量、被否决的备选方案，以及迁移或兼容性规则。
- For debugging work, list hypotheses, observations needed, instrumentation or
  logs to inspect, reproduction steps, and the evidence that will confirm or
  falsify each hypothesis.
  调试类工作：列出假设、所需观察、要检视的插桩或日志、复现步骤，以及将证实或证伪每个假设的证据。
- For rollout or migration work, include phases, gates, rollback, data safety,
  monitoring, and user-visible impact.
  上线或迁移类工作：纳入阶段、门禁、回滚、数据安全、监控和用户可见影响。
- For eval or research work, include the benchmark/question, corpus or sources,
  comparison arms, success metrics, confounders, and reproducibility evidence.
  评估或研究类工作：纳入基准/问题、语料或来源、对照组、成功指标、混淆因素和可复现性证据。

Do not invent an issue, spec, PR, or file path just to fill a section. If the
workspace rules require them, cite the real item or state the missing prerequisite
as an open question or blocker. Do not repeat background the user already knows or
add empty sections. Keep the plan decision-complete enough that an implementer does
not need to invent scope, interfaces, or verification.

不要为了填满某个小节而编造 issue、规格、PR 或文件路径。若工作区规则要求它们，引用真实条目，或把缺失的前提作为未决问题或阻碍写明。不要重复用户已知的背景，也不要添加空洞小节。保持计划决策完备，让实现者无需自行发明范围、接口或验证方式。

## Workflow / 工作流

1. Classify whether planning is actually needed. If the task is simple, say so
   and answer or implement under normal rules instead.
   判断是否真的需要规划。若任务简单，如实说明，并改为按常规规则回答或实现。
2. Complete Research Before Drafting when it applies. Ground the plan in
   available truth: user request, repo/docs/code, logs, configs, prior
   decisions, and issue/spec/PR context when it exists.
   在适用时完成"起草前的调研"。把计划锚定在可获得的事实上：用户请求、仓库/文档/代码、日志、配置、既往决策，以及存在时的 issue/规格/PR 上下文。
3. Identify the plan type and the smallest viable path that proves the approach.
   识别计划类型，以及能证明该方案的最小可行路径。
4. Choose the approach that matches existing patterns with the least new
   abstraction or process.
   选择与现有模式匹配、新增抽象或流程最少的方案。
5. Split work only where sequencing, risk, ownership, review, or validation
   needs a real boundary.
   只有在时序、风险、归属、评审或验证需要真实边界的地方才拆分工作。
6. Map every work unit to validation evidence or a real manual check.
   把每个工作单元映射到验证证据或真实的人工检查。
7. If the user or workflow requested a saved plan, briefly report its path. Do not
   announce the absence of a file for the normal inline case.
   若用户或工作流要求保存计划，简要报告其路径。正常的行内场景不要宣告"没有文件"。
8. Highlight the highest-risk validation step.
   突出风险最高的验证步骤。
9. Present the plan in the User-Visible Delivery form and stop for the user's
   ordinary next message before implementation.
   以"面向用户的交付"形式呈现计划，然后停下，等待用户的下一条普通消息，再进入实现。

## Exit / 退出

- After generating the plan, present the conversational handoff, the complete canonical
  Markdown, and the repeated handoff in one normal assistant reply.
  生成计划后，在同一条普通助手回复中呈现对话式交接、完整规范 Markdown，以及重复的交接语。
- Stop for the user's ordinary next message. Do not treat planning text as permission to edit.
  停下等待用户的下一条普通消息。不要把规划文本当作编辑许可。
- If the user says go, continue, implement, or otherwise starts execution, use the
  preceding plan directly: leave planning and do the work under the normal workspace
  rules, carrying the plan's structural commitments (see Executing The Plan).
  若用户说 go、continue、implement 或以其他方式开始执行，直接使用前述计划：离开规划状态，按常规工作区规则开展工作，并携带计划的结构性承诺（见 Executing The Plan）。
- Re-check the latest user message before editing so stale plan assumptions do
  not override a new instruction.
  编辑之前复查最新的用户消息，以免过时的计划假设覆盖新指令。

## Executing The Plan / 执行计划

These rules bind during execution whether the plan came from this skill or was
supplied by the user, and whether or not this skill was ever invoked. If you
loaded this skill at publish time, only this section applies — the planning
restrictions above do not.

无论计划来自本技能还是由用户提供，也无论是否调用过本技能，这些规则在执行期间都有效。若你在发布时加载了本技能，则只有本节适用——上述规划限制不适用。

- A plan's structural commitments survive into execution: the diff/commit/PR
  split and the unit ordering stay binding while the work is carried out and
  published.
  计划的结构性承诺延续到执行阶段：在工作执行与发布期间，diff/提交/PR 拆分和单元顺序保持约束力。
- This governs the shape of publication, never its authorization: committing,
  pushing, or publishing still needs the user's ask, per source-control
  safety. A plan step that says "commit" does not by itself authorize a
  commit.
  这只约束发布的形态，绝不构成发布的授权：提交、推送或发布仍需用户的明确要求，遵循源码控制安全。计划步骤里写了"commit"本身并不授权提交。

  【评论】"计划里写了 commit 也不构成提交授权"——把计划内容与操作授权严格分离，防止计划文本被当作执行许可。
- When the user has you publish work the plan divides into N separate diffs,
  commits, or PRs, publish exactly that split: commit each planned unit
  separately, in the planned order, even when one combined commit would
  satisfy the literal request ("commit this", "publish it").
  当用户让你发布一份被计划拆分为 N 个独立 diff、提交或 PR 的工作时，严格按该拆分发布：按计划顺序逐个提交每个计划单元，即使一次合并提交也能满足字面请求（"commit this""publish it"）。
- Work already finished in one mixed working tree still gets split at publish
  time: group the changed files by plan unit and commit unit by unit. If one
  file's changes span units, split at the hunk level (`git add -p`) or assign
  the file to the earliest unit and say so.
  已经在同一个混合工作树中完成的工作，在发布时仍要拆分：按计划单元分组已更改文件，逐单元提交。若某个文件的更改跨越多个单元，就在 hunk 级拆分（`git add -p`），或把该文件归入最早的单元并说明。
- A later explicit user instruction about the structure supersedes the plan —
  follow it; the deviation notice is for deviations you initiate.
  用户后续关于结构的明确指令优先于计划——遵循它；偏差告知针对的是你主动发起的偏差。
- Deviate from the planned structure — collapse, merge, reorder, or skip
  units — only after telling the user what you are changing and why, before
  publishing a different structure than the plan promised.
  只有在告诉用户你要改变什么及其原因之后，才可以偏离计划结构——合并、归并、重排或跳过单元；即先于发布任何与计划承诺不同的结构之前完成告知。
