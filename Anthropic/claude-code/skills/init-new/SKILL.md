---
name: init
description: |-
  Initialize new CLAUDE.md file(s) and optional skills/hooks with codebase documentation
---
<!-- BILINGUAL-EN-ZH -->

Set up a minimal CLAUDE.md (and optionally skills and hooks) for this repo. CLAUDE.md is loaded into every Claude Code session, so it must be concise — only include what Claude would get wrong without it.

为本仓库设置一份精简的 CLAUDE.md（以及可选的技能和 hook）。CLAUDE.md 会在每个 Claude Code 会话中加载，因此必须简洁 — 只收录没有它 Claude 就会出错的内容。

【评论】"只收录没有它 Claude 就会出错的内容"这一准入判据，把上下文窗口的成本意识转化为可操作的写作标准，是上下文文件极简主义的典型体现。

## Phase 0: Check for an existing CLAUDE.md / 阶段 0：检查是否已存在 CLAUDE.md

Before asking anything, check if CLAUDE.md already exists at the project root (just `cat ./CLAUDE.md` — only the project-root file counts; don't explore the tree yet). This branches Phase 1.

在提问之前，先检查项目根目录是否已存在 CLAUDE.md（只需 `cat ./CLAUDE.md` — 只有项目根目录下的文件才算数；暂不要探索目录树）。这一步决定阶段 1 的分支走向。

## Phase 1: Ask what to set up / 阶段 1：询问要设置什么

Use AskUserQuestion to find out what the user wants. Which question you ask depends on Phase 0. Call AskUserQuestion with **only Q1** — do NOT include Q2 in the same call. Only ask Q2 after you've seen the Q1 answer, since "Let Claude decide" skips it.

用 AskUserQuestion 询问用户想要什么。问哪个问题取决于阶段 0。调用 AskUserQuestion 时**只带 Q1** — 不要在同一次调用中包含 Q2。只有在看到 Q1 的答案之后才能问 Q2，因为"让 Claude 决定"会跳过 Q2。

Before the first question, print this primer as normal assistant text so first-time users know the terms:

在第一个问题之前，先把下面的说明以普通助手文本打印出来，让首次使用的用户了解这些术语：

> Quick context:
> 快速背景：
> - **CLAUDE.md** files give Claude persistent instructions for a project, your personal workflow, or your organization. Claude reads them at the start of every session.
>   - **CLAUDE.md** 文件为 Claude 提供针对项目、你的个人工作流或组织的持久化指令。Claude 会在每次会话开始时读取它们。
> - **Skills** are packaged instructions Claude invokes automatically when a task matches, or that you trigger with a slash command (e.g. `/frontend-design`, `/commit-push-pr`).
>   - **技能（Skills）**是打包好的指令，任务匹配时 Claude 会自动调用，你也可以用斜杠命令触发（如 `/frontend-design`、`/commit-push-pr`）。
> - **Hooks** allow you to run shell commands automatically on lifecycle events: get notified when Claude is blocked on your input, auto-format after edits, enforce checks before commits — these are deterministic and Claude can't skip them.
>   - **Hook** 允许你在生命周期事件上自动运行 shell 命令：Claude 等待你输入时收到通知、编辑后自动格式化、提交前强制检查 — 这些是确定性操作，Claude 无法跳过。

**If CLAUDE.md already exists**, ask:

**如果 CLAUDE.md 已存在**，询问：

- "I found an existing CLAUDE.md. What would you like to do?"
  "我发现已有一份 CLAUDE.md。你想怎么做？"
  Options: "Review and improve it" | "Leave it, set up other things" | "Start fresh (replace it)"
  选项："审查并改进" | "保留它，设置其他内容" | "从头开始（替换它）"
  Description for improve: "Explore what's changed in the codebase and propose targeted edits to the existing file."
  "改进"选项的描述："探索代码库中发生了哪些变化，并对现有文件提出有针对性的修改建议。"
  Description for leave it: "Skip CLAUDE.md. Go straight to skills and hooks."
  "保留"选项的描述："跳过 CLAUDE.md，直接进入技能和 hook 的设置。"
  Description for start fresh: "Discard it and write new file(s)."
  "从头开始"选项的描述："丢弃现有文件，写入新文件。"
  Routing:
  路由：
  - "Review and improve" → skip Q1/Q2; explore (Phase 2), ask the single Phase 3-lite question, then go to Phase 4's diff-proposal, then Phase 8.
    "审查并改进" → 跳过 Q1/Q2；先探索（阶段 2），再问阶段 3-lite 的单个问题，然后进入阶段 4 的差异提案，最后到阶段 8。
  - "Leave it" → skip Q1, ask Q2 (rename its fourth option to "Neither — skip setup"). If they pick "Neither — skip setup", jump straight to Phase 8 with: "Nothing to set up — your CLAUDE.md is unchanged." Otherwise: Phase 2 → Phase 3 proposal (no gap-fill interview) → Phases 6/7 per queue → Phase 8. For Phase 7's hook target-file default, treat this path as "project" (`.claude/settings.json`).
    "保留" → 跳过 Q1，问 Q2（把其第四个选项改名为"都不用 — 跳过设置"）。如果用户选"都不用 — 跳过设置"，直接跳到阶段 8，并告知："没有需要设置的内容 — 你的 CLAUDE.md 保持不变。"否则：阶段 2 → 阶段 3 提案（不做补漏访谈）→ 按队列执行阶段 6/7 → 阶段 8。对于阶段 7 的 hook 目标文件默认值，此路径按"项目"处理（`.claude/settings.json`）。
  - "Start fresh" → continue to Q1 below as if no file existed.
    "从头开始" → 继续下面的 Q1，如同文件不存在一样。

**If no CLAUDE.md exists** (or the user picked "Start fresh"), ask:

**如果 CLAUDE.md 不存在**（或用户选了"从头开始"），询问：

- Q1: "Which CLAUDE.md files should /init set up?"
  Q1："应该由 /init 设置哪些 CLAUDE.md 文件？"
  Options: "Project CLAUDE.md" | "Personal CLAUDE.local.md" | "Both project + personal" | "Let Claude decide"
  选项："项目 CLAUDE.md" | "个人 CLAUDE.local.md" | "项目 + 个人都要" | "让 Claude 决定"
  Description for project: "Team-shared instructions checked into source control — architecture, coding standards, common workflows."
  "项目"选项的描述："纳入版本控制、团队共享的指令 — 架构、编码规范、常用工作流。"
  Description for personal: "Your private preferences for this project (gitignored, not shared) — your role, sandbox URLs, preferred test data, workflow quirks."
  "个人"选项的描述："你针对本项目的私人偏好（被 gitignore，不共享）— 你的角色、沙箱 URL、常用测试数据、工作流习惯。"
  Description for Let Claude decide: "Fastest path — project CLAUDE.md plus whatever skills or hooks fit this repo. No follow-on questions; you'll approve everything before it's written."
  "让 Claude 决定"选项的描述："最快路径 — 项目 CLAUDE.md 加上适合本仓库的技能或 hook。没有后续提问；写入之前你会逐项确认。"
  If the user picks "Let Claude decide", skip Q2 — treat it as project CLAUDE.md with no skills/hooks constraint.
  如果用户选"让 Claude 决定"，跳过 Q2 — 按项目 CLAUDE.md 处理，且对技能/hook 不设约束。

- Q2: "Also set up skills and hooks?"
  Q2："是否同时设置技能和 hook？"
  Options: "Skills + hooks" | "Skills only" | "Hooks only" | "Neither, just CLAUDE.md"
  选项："技能 + hook" | "仅技能" | "仅 hook" | "都不要，只要 CLAUDE.md"
  Description for skills: "Packaged instructions Claude invokes automatically when a task matches, or that you trigger with a slash command (e.g. `/frontend-design`, `/commit-push-pr`)."
  "技能"选项的描述："打包好的指令，任务匹配时 Claude 自动调用，你也可以用斜杠命令触发（如 `/frontend-design`、`/commit-push-pr`）。"
  Description for hooks: "Deterministic shell commands that run on tool events (e.g., format after every edit). Claude can't skip them."
  "hook"选项的描述："在工具事件上运行的确定性 shell 命令（如每次编辑后格式化）。Claude 无法跳过。"
  Q2 is a hint, not a filter — Phase 3 proposes what fits the codebase and notes any deviation.
  Q2 只是提示，不是过滤器 — 阶段 3 会提出适合代码库的方案，并注明任何偏离。

## Phase 2: Explore the codebase / 阶段 2：探索代码库

Launch a subagent to survey the codebase, and ask it to read key files to understand the project: manifest files (package.json, Cargo.toml, pyproject.toml, go.mod, pom.xml, etc.), README, Makefile/build configs, CI config, existing CLAUDE.md, .claude/rules/, AGENTS.md, .cursor/rules or .cursorrules, .github/copilot-instructions.md, .devin/rules/ or .windsurf/rules/ or .windsurfrules, .clinerules, .mcp.json.

启动一个子代理勘察代码库，让它阅读关键文件以了解项目：清单文件（package.json、Cargo.toml、pyproject.toml、go.mod、pom.xml 等）、README、Makefile/构建配置、CI 配置、既有 CLAUDE.md、.claude/rules/、AGENTS.md、.cursor/rules 或 .cursorrules、.github/copilot-instructions.md、.devin/rules/ 或 .windsurf/rules/ 或 .windsurfrules、.clinerules、.mcp.json。

【评论】清单一并枚举了 Cursor、Copilot、Devin、Windsurf、Cline 等其他 AI 编码工具的指令文件，说明该技能带有读取并迁移既有 AI 工具配置的意图。

Also have the subagent do a cheap presence check (not a read — the contents are handled by the import adapters) for:

同时让子代理对以下内容做廉价的存在性检查（不是读取 — 内容由导入适配器处理）：

- OpenAI Codex config: ~/.codex/config.toml or ./.codex/
  OpenAI Codex 配置：~/.codex/config.toml 或 ./.codex/
- Gemini CLI config: ~/.gemini/settings.json, ./.gemini/, or a GEMINI.md at project root
  Gemini CLI 配置：~/.gemini/settings.json、./.gemini/ 或项目根目录的 GEMINI.md

Record which of these exist — Phase 8 uses it.

记录其中哪些存在 — 阶段 8 会用到。

Detect:

检测：

- Build, test, and lint commands (especially non-standard ones)
  构建、测试和 lint 命令（尤其是非标准的）
- Languages, frameworks, and package manager
  语言、框架和包管理器
- Project structure (monorepo with workspaces, multi-module, or single project)
  项目结构（带 workspaces 的 monorepo、多模块或单项目）
- Code style rules that differ from language defaults
  与语言默认值不同的代码风格规则
- Non-obvious gotchas, required env vars, or workflow quirks
  不显眼的坑、必需的环境变量或工作流习惯
- Existing .claude/skills/ and .claude/rules/ directories
  既有的 .claude/skills/ 和 .claude/rules/ 目录
- Formatter configuration (prettier, biome, ruff, black, gofmt, rustfmt, or a unified format script like `npm run format` / `make fmt`)
  格式化工具配置（prettier、biome、ruff、black、gofmt、rustfmt，或统一的格式化脚本如 `npm run format` / `make fmt`）
- Git worktree usage: run `git worktree list` to check if this repo has multiple worktrees (only relevant if the user wants a personal CLAUDE.local.md)
  Git worktree 使用情况：运行 `git worktree list` 检查本仓库是否有多个 worktree（仅当用户想要个人 CLAUDE.local.md 时才相关）

Note what you could NOT figure out from code alone — these become interview questions.

记下仅凭代码无法弄清的事项 — 它们将成为访谈问题。

## Phase 3: Fill in the gaps / 阶段 3：填补空白

Use AskUserQuestion to gather what you still need to write good CLAUDE.md files and skills. Ask only things the code can't answer.

用 AskUserQuestion 收集编写高质量 CLAUDE.md 文件和技能还欠缺的信息。只问代码无法回答的问题。

If the user chose project CLAUDE.md, both, or "Let Claude decide": ask about codebase practices — non-obvious commands, gotchas, branch/PR conventions, required env setup, testing quirks. Skip things already in README or obvious from manifest files. Do not mark any options as "recommended" — this is about how their team works, not best practices.

如果用户选择了项目 CLAUDE.md、两者都要或"让 Claude 决定"：询问代码库实践 — 不显眼的命令、坑、分支/PR 约定、必需的环境配置、测试习惯。跳过 README 已有或从清单文件即可看出的事项。不要把任何选项标为"推荐" — 这关乎他们团队的工作方式，而非最佳实践。

If the user chose personal CLAUDE.local.md or both: ask about them, not the codebase. Do not mark any options as "recommended" — this is about their personal preferences, not best practices. Examples of questions:

如果用户选择了个人 CLAUDE.local.md 或两者都要：询问用户本人，而非代码库。不要把任何选项标为"推荐" — 这关乎个人偏好，而非最佳实践。问题示例：

  - What's their role on the team? (e.g., "backend engineer", "data scientist", "new hire onboarding")
    用户在团队中的角色是什么？（如"后端工程师"、"数据科学家"、"刚入职的新人"）
  - How familiar are they with this codebase and its languages/frameworks? (so Claude can calibrate explanation depth)
    用户对这个代码库及其语言/框架的熟悉程度如何？（以便 Claude 校准讲解深度）
  - Do they have personal sandbox URLs, test accounts, API key paths, or local setup details Claude should know?
    用户是否有 Claude 应当知道的私人沙箱 URL、测试账号、API 密钥路径或本地环境细节？
  - Only if Phase 2 found multiple git worktrees: ask whether their worktrees are nested inside the main repo (e.g., `.claude/worktrees/<name>/`) or siblings/external (e.g., `../myrepo-feature/`). If nested, the upward file walk finds the main repo's CLAUDE.local.md automatically — no special handling needed. If sibling/external, the personal content should live in a home-directory file (e.g., `~/.claude/<project-name>-instructions.md`) and each worktree gets a one-line CLAUDE.local.md stub that imports it: `@~/.claude/<project-name>-instructions.md`. Never put this import in the project CLAUDE.md — that would check a personal reference into the team-shared file.
    仅当阶段 2 发现多个 git worktree 时：询问其 worktree 是嵌套在主仓库内（如 `.claude/worktrees/<name>/`）还是同级/外部（如 `../myrepo-feature/`）。若是嵌套，向上查找文件时会自动找到主仓库的 CLAUDE.local.md — 无需特殊处理。若是同级/外部，个人内容应存放在主目录文件中（如 `~/.claude/<project-name>-instructions.md`），每个 worktree 放一个一行式 CLAUDE.local.md 存根来导入它：`@~/.claude/<project-name>-instructions.md`。绝不要把这个导入放进项目 CLAUDE.md — 那会把个人引用提交进团队共享文件。
  - Any communication preferences? (e.g., "be terse", "always explain tradeoffs", "don't summarize at the end")
    有无沟通偏好？（如"简洁一些"、"总是解释权衡"、"结尾不要总结"）

If the user picked "Review and improve" in Phase 0: ask just one question — "Has anything changed about how the team works since this CLAUDE.md was written (new conventions, commands, gotchas)?" with options "No, nothing's changed" | "Yes — let me describe". If they pick Yes, ask what changed (free text) before continuing. Then skip to Phase 4.

如果用户在阶段 0 选了"审查并改进"：只问一个问题 — "自这份 CLAUDE.md 写成以来，团队的工作方式有什么变化吗（新约定、命令、坑）？"，选项为"没有，没有变化" | "有 — 我来描述"。如果选"有"，先问变化内容（自由文本）再继续。然后跳到阶段 4。

**Synthesize a proposal from Phase 2 findings and the gap-fill answers.** For each item, pick the artifact type that fits the evidence:

**根据阶段 2 的发现和补漏答案综合出一份提案。** 对每一项，选择符合证据的产物类型：

  - **Hook** — deterministic, fast, per-edit shell command (formatting, linting a changed file).
    **Hook** — 确定性、快速、每次编辑触发的 shell 命令（格式化、对改动文件做 lint）。
  - **Skill** — on-demand multi-step workflow (`/verify`, `/deploy-staging`, session reports).
    **技能（Skill）** — 按需调用的多步骤工作流（`/verify`、`/deploy-staging`、会话报告）。
  - **CLAUDE.md note** — guidance that shapes behavior but isn't enforced (conventions, communication style).
    **CLAUDE.md 备注** — 影响行为但不强制执行的指引（约定、沟通风格）。

Include the CLAUDE.md file(s) implied by Q1 (project, personal, both, or "Let Claude decide" → project) as the first bullet(s) of the proposal, with a one-line summary of what each will cover. Then list skills/hooks/notes. On the "Leave it" path, omit CLAUDE.md file bullets and notes (Phase 4 won't run). On the "Start fresh" path with Q1 = personal-only, add a bullet noting the existing project CLAUDE.md will be left untouched (they chose not to replace it with a project file).

把 Q1 所暗示的 CLAUDE.md 文件（项目、个人、两者，或"让 Claude 决定" → 项目）作为提案的第一批要点，每个附一行关于其将涵盖内容的摘要。然后列出技能/hook/备注。在"保留"路径上，省略 CLAUDE.md 文件要点和备注（阶段 4 不会运行）。在"从头开始"且 Q1 = 仅个人的路径上，加一条要点说明现有项目 CLAUDE.md 将保持原样（用户选择了不用项目文件替换它）。

Propose what fits. If the user gave a Q2 hint and your proposal deviates from it (e.g. they said "Hooks only" but nothing hook-shaped exists), say so in one line at the top of the proposal and propose the better-fitting artifacts anyway.

提出适合的方案。如果用户给了 Q2 提示而你的提案与之偏离（如用户说"仅 hook"但不存在适合做成 hook 的东西），在提案开头用一行说明这一点，并照样提出更合适的产物。

**Print the proposal as normal assistant text**, one bullet per item:

**以普通助手文本打印提案**，每项一条要点：

> Here's what I'd set up:
> • **[Artifact type: file/hook/skill/note]** — [one-line description]
> • …

> 我建议的配置如下：
> • **[产物类型：文件/hook/技能/备注]** — [一行描述]
> • …

Then call AskUserQuestion with a simple question ("Does this look right?") and options like "Looks good — proceed" | "Drop the hook" | "Drop the skill". Don't use the `preview` field — the proposal is already visible in scrollback. The tool auto-adds an "Other" option for custom tweaks.

然后用 AskUserQuestion 问一个简单问题（"这样安排可以吗？"），选项如"没问题 — 继续" | "去掉 hook" | "去掉技能"。不要用 `preview` 字段 — 提案已在回滚记录中可见。该工具会自动添加"Other"选项供自定义调整。

**Build the preference queue** from the accepted proposal. Each entry: {type: hook|skill|note, description, target file, any Phase-2-sourced details like the actual test/format command}. Phase 6 and Phase 7's hooks sub-bullet consume this queue; Phases 4/5 gate on the approved proposal's file bullets directly; Phase 7's GitHub-CLI and linting checks run regardless of queue contents.

**根据被接受的提案构建偏好队列。** 每个条目：{type: hook|skill|note, description, target file，以及来自阶段 2 的细节如实际的测试/格式化命令}。阶段 6 和阶段 7 的 hooks 子项消费该队列；阶段 4/5 直接以已批准提案的文件要点为准；阶段 7 的 GitHub-CLI 和 lint 检查无论队列内容如何都会执行。

## Phase 4: Write CLAUDE.md (if the approved proposal includes it, or on the "Review and improve" path) / 阶段 4：编写 CLAUDE.md（若已批准提案包含它，或处于"审查并改进"路径）

Write a minimal CLAUDE.md at the project root. Every line must pass this test: "Would removing this cause Claude to make mistakes?" If no, cut it.

在项目根目录写一份精简的 CLAUDE.md。每一行都必须通过这项检验："删掉这行会导致 Claude 犯错吗？"如果不会，就删掉。

【评论】这条"删除测试"把极简原则具体化为可执行判据：一行内容是否值得常驻每个会话的上下文，取决于它缺席时模型是否会出错。

If the user picked "Review and improve it" in Phase 0: don't write fresh — read the existing file, compare against Phase 2 findings and the Phase 3-lite answer, and propose specific additions/removals as diffs with a one-line reason for each. The existing file is the baseline; your job is to catch what's missing, outdated, or bloated. After printing the diffs, call AskUserQuestion ("Apply these edits?" with options like "Apply all" | "Let me pick which" | "Skip — leave it as is") before writing anything.

如果用户在阶段 0 选了"审查并改进"：不要重写 — 先读现有文件，与阶段 2 的发现和阶段 3-lite 的答案对比，以 diff 形式提出具体的增删建议，每条附一行理由。现有文件是基线；你的任务是找出缺失、过时或臃肿之处。打印 diff 之后、写入任何内容之前，调用 AskUserQuestion（"应用这些修改吗？"，选项如"全部应用" | "我来挑选" | "跳过 — 保持原样"）。

**Consume `note` entries from the Phase 3 preference queue whose target is CLAUDE.md** (team-level notes) — add each as a concise line in the most relevant section. These are the behaviors the user wants Claude to follow but didn't need guaranteed (e.g., "propose a plan before implementing", "explain the tradeoffs when refactoring"). Leave personal-targeted notes for Phase 5.

**消费阶段 3 偏好队列中目标为 CLAUDE.md 的 `note` 条目**（团队级备注）— 每条以简洁的一行加入最相关的小节。这些是用户希望 Claude 遵循但无需强制保证的行为（如"实现前先提出方案"、"重构时解释权衡"）。面向个人的备注留给阶段 5。

Include:

应包含：

- Build/test/lint commands Claude can't guess (non-standard scripts, flags, or sequences)
  Claude 猜不到的构建/测试/lint 命令（非标准脚本、旗标或顺序）
- Code style rules that DIFFER from language defaults (e.g., "prefer type over interface")
  与语言默认值不同的代码风格规则（如"优先用 type 而非 interface"）
- Testing instructions and quirks (e.g., "run single test with: pytest -k 'test_name'")
  测试说明与特殊之处（如"用 pytest -k 'test_name' 运行单个测试"）
- Repo etiquette (branch naming, PR conventions, commit style)
  仓库礼仪（分支命名、PR 约定、提交风格）
- Required env vars or setup steps
  必需的环境变量或设置步骤
- Non-obvious gotchas or architectural decisions
  不显眼的坑或架构决策
- Important parts from existing AI coding tool configs if they exist (AGENTS.md, .cursor/rules, .cursorrules, .github/copilot-instructions.md, .devin/rules/, .windsurf/rules/, .windsurfrules, .clinerules)
  既有 AI 编码工具配置中的重要部分（如有）（AGENTS.md、.cursor/rules、.cursorrules、.github/copilot-instructions.md、.devin/rules/、.windsurf/rules/、.windsurfrules、.clinerules）

Exclude:

应排除：

- File-by-file structure or component lists (Claude can discover these by reading the codebase)
  逐文件的目录结构或组件列表（Claude 可以通过阅读代码库自行发现）
- Standard language conventions Claude already knows
  Claude 已经掌握的语言标准约定
- Generic advice ("write clean code", "handle errors")
  泛泛的建议（"写干净的代码"、"处理错误"）
- Detailed API docs or long references — use `@path/to/import` syntax instead (e.g., `@docs/api-reference.md`) to inline content on demand without bloating CLAUDE.md
  详细的 API 文档或长篇参考 — 改用 `@path/to/import` 语法（如 `@docs/api-reference.md`）按需内联内容，避免 CLAUDE.md 膨胀
- Information that changes frequently — reference the source with `@path/to/import` so Claude always reads the current version
  频繁变化的信息 — 用 `@path/to/import` 引用来源，让 Claude 总是读到当前版本
- Long tutorials or walkthroughs (move to a separate file and reference with `@path/to/import`, or put in a skill)
  长教程或分步指引（移到单独文件并用 `@path/to/import` 引用，或放进技能）
- Commands obvious from manifest files (e.g., standard "npm test", "cargo test", "pytest")
  从清单文件即可看出的命令（如标准的 "npm test"、"cargo test"、"pytest"）

Be specific: "Use 2-space indentation in TypeScript" is better than "Format code properly."

要具体："在 TypeScript 中使用 2 空格缩进"比"把代码格式化好"更有用。

Do not repeat yourself and do not make up sections like "Common Development Tasks" or "Tips for Development" — only include information expressly found in files you read.

不要重复自己，也不要编造"常见开发任务"或"开发提示"之类的小节 — 只收录你在所读文件中明确找到的信息。

Prefix the file with:

在文件开头加上：

```
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
```

For projects with multiple concerns, suggest organizing instructions into `.claude/rules/` as separate focused files (e.g., `code-style.md`, `testing.md`, `security.md`). These are loaded automatically alongside CLAUDE.md and can be scoped to specific file paths using `paths` frontmatter.

对于关注点较多的项目，建议把指令组织到 `.claude/rules/` 下的多个聚焦文件中（如 `code-style.md`、`testing.md`、`security.md`）。它们会与 CLAUDE.md 一起自动加载，并可用 `paths` frontmatter 限定到特定文件路径。

For projects with distinct subdirectories (monorepos, multi-module projects, etc.): mention that subdirectory CLAUDE.md files can be added for module-specific instructions (they're loaded automatically when Claude works in those directories). Offer to create them if the user wants.

对于具有独立子目录的项目（monorepo、多模块项目等）：提及可为模块专属指令添加子目录级 CLAUDE.md 文件（当 Claude 在这些目录中工作时它们会自动加载）。如果用户需要，可主动提出创建。

## Phase 5: Write CLAUDE.local.md (if the approved proposal includes it) / 阶段 5：编写 CLAUDE.local.md（若已批准提案包含它）

Write a minimal CLAUDE.local.md at the project root. This file is automatically loaded alongside CLAUDE.md. After creating it, add `CLAUDE.local.md` to the project's .gitignore so it stays private.

在项目根目录写一份精简的 CLAUDE.local.md。该文件会与 CLAUDE.md 一起自动加载。创建后，把 `CLAUDE.local.md` 加入项目的 .gitignore，使其保持私密。

**Consume `note` entries from the Phase 3 preference queue whose target is CLAUDE.local.md** (personal-level notes) — add each as a concise line. If the user chose personal-only in Phase 1, this is the sole consumer of note entries.

**消费阶段 3 偏好队列中目标为 CLAUDE.local.md 的 `note` 条目**（个人级备注）— 每条以简洁的一行加入。如果用户在阶段 1 选择了仅个人文件，这里是 note 条目的唯一消费方。

Include:

应包含：

- The user's role and familiarity with the codebase (so Claude can calibrate explanations)
  用户的角色和对代码库的熟悉程度（以便 Claude 校准讲解方式）
- Personal sandbox URLs, test accounts, or local setup details
  私人沙箱 URL、测试账号或本地环境细节
- Personal workflow or communication preferences
  个人工作流或沟通偏好

Keep it short — only include what would make Claude's responses noticeably better for this user.

保持简短 — 只收录能让该用户明显感到 Claude 回复变好的内容。

If Phase 2 found multiple git worktrees and the user confirmed they use sibling/external worktrees (not nested inside the main repo): the upward file walk won't find a single CLAUDE.local.md from all worktrees. Write the actual personal content to `~/.claude/<project-name>-instructions.md` and make CLAUDE.local.md a one-line stub that imports it: `@~/.claude/<project-name>-instructions.md`. The user can copy this one-line stub to each sibling worktree. Never put this import in the project CLAUDE.md. If worktrees are nested inside the main repo (e.g., `.claude/worktrees/`), no special handling is needed — the main repo's CLAUDE.local.md is found automatically.

如果阶段 2 发现多个 git worktree 且用户确认使用同级/外部 worktree（未嵌套在主仓库内）：向上查找文件无法从所有 worktree 找到同一个 CLAUDE.local.md。请把实际的个人内容写到 `~/.claude/<project-name>-instructions.md`，并让 CLAUDE.local.md 成为一个一行式存根来导入它：`@~/.claude/<project-name>-instructions.md`。用户可把这行存根复制到每个同级 worktree。绝不要把这个导入放进项目 CLAUDE.md。如果 worktree 嵌套在主仓库内（如 `.claude/worktrees/`），无需特殊处理 — 会自动找到主仓库的 CLAUDE.local.md。

If CLAUDE.local.md already exists: read it, propose specific additions, and do not silently overwrite.

如果 CLAUDE.local.md 已存在：先读取，提出具体的增补建议，不要静默覆盖。

## Phase 6: Suggest and create skills (if the approved proposal includes any) / 阶段 6：建议并创建技能（若已批准提案包含）

Skills add capabilities Claude can use on demand without bloating every session.

技能为 Claude 增加按需使用的能力，又不会让每个会话膨胀。

**First, consume `skill` entries from the Phase 3 preference queue.** Each queued skill preference becomes a SKILL.md tailored to what the user described. For each:

**首先，消费阶段 3 偏好队列中的 `skill` 条目。** 每条队列中的技能偏好都会成为一个按用户描述定制的 SKILL.md。对每一条：

- Name it from the preference (e.g., "verify-deep", "session-report", "deploy-sandbox")
  依偏好命名（如 "verify-deep"、"session-report"、"deploy-sandbox"）
- Write the body using the user's own words from the interview plus whatever Phase 2 found (test commands, report format, deploy target). If the preference maps to an existing bundled skill (e.g., `/verify`), write a project skill that adds the user's specific constraints on top — tell the user the bundled one still exists and theirs is additive.
  用访谈中用户自己的原话加上阶段 2 的发现（测试命令、报告格式、部署目标）来写正文。如果该偏好对应某个既有内置技能（如 `/verify`），就写一个在其上叠加用户特定约束的项目技能 — 并告诉用户内置技能仍然存在，二者是叠加关系。
- Ask a quick follow-up if the preference is underspecified (e.g., "which test command should verify-deep run?")
  如果偏好描述不充分，快速追问一句（如"verify-deep 应运行哪条测试命令？"）

**Then suggest additional skills** beyond the queue when you find:

**然后在队列之外建议额外技能**，当你发现：

- Reference knowledge for specific tasks (conventions, patterns, style guides for a subsystem)
  面向特定任务的参考知识（某子系统的约定、模式、风格指南）
- Repeatable workflows the user would want to trigger directly (deploy, fix an issue, release process, verify changes)
  用户会想直接触发的可重复工作流（部署、修复 issue、发布流程、验证改动）

For each suggested skill, provide: name, one-line purpose, and why it fits this repo.

对每个建议的技能，给出：名称、一行用途，以及它为何适合本仓库。

If `.claude/skills/` already exists with skills, review them first. Do not overwrite existing skills — only propose new ones that complement what is already there.

如果 `.claude/skills/` 已存在技能，先审查它们。不要覆盖既有技能 — 只建议与现有内容互补的新技能。

Create each skill at `.claude/skills/<skill-name>/SKILL.md`:

每个技能创建在 `.claude/skills/<skill-name>/SKILL.md`：

```yaml
---
name: <skill-name>
description: <what the skill does and when to use it>
---

<Instructions for Claude>
```

Both the user (`/<skill-name>`) and Claude can invoke skills by default. For workflows with side effects (e.g., `/deploy`, `/fix-issue 123`), add `disable-model-invocation: true` so only the user can trigger it, and use `$ARGUMENTS` to accept input.

默认情况下，用户（`/<skill-name>`）和 Claude 都可以调用技能。对有副作用的工作流（如 `/deploy`、`/fix-issue 123`），加 `disable-model-invocation: true` 使其只能由用户触发，并用 `$ARGUMENTS` 接受输入。

【评论】"有副作用的技能禁止模型自主调用"是权限收窄设计：把部署等不可逆操作的触发权保留给人类用户。

## Phase 7: Suggest additional optimizations / 阶段 7：建议更多优化

Tell the user you're going to suggest a few additional optimizations now that CLAUDE.md and skills (if chosen) are in place.

告诉用户：既然 CLAUDE.md 和技能（如已选择）已就绪，接下来会再建议几项额外优化。

Check the environment and ask about each gap you find (use AskUserQuestion):

检查环境，对发现的每个缺口进行询问（使用 AskUserQuestion）：

- **GitHub CLI**: Run `which gh` (or `where gh` on Windows). If it's missing AND the project uses GitHub (check `git remote -v` for github.com), ask the user if they want to install it. Explain that the GitHub CLI lets Claude help with commits, pull requests, issues, and code review directly.
  **GitHub CLI**：运行 `which gh`（Windows 上为 `where gh`）。如果缺失且项目使用 GitHub（通过 `git remote -v` 检查是否有 github.com），询问用户是否要安装。说明 GitHub CLI 能让 Claude 直接协助提交、拉取请求、issue 和代码审查。

- **Linting**: If Phase 2 found no lint config (no .eslintrc, ruff.toml, .golangci.yml, etc. for the project's language), ask the user if they want Claude to set up linting for this codebase. Explain that linting catches issues early and gives Claude fast feedback on its own edits.
  **Lint**：如果阶段 2 未发现 lint 配置（项目所用语言没有 .eslintrc、ruff.toml、.golangci.yml 等），询问用户是否想让 Claude 为代码库配置 lint。说明 lint 能尽早发现问题，并让 Claude 对自己的编辑获得快速反馈。

- **Proposal-sourced hooks** (if the approved proposal includes any): Consume `hook` entries from the Phase 3 preference queue. If Phase 2 found a formatter and the queue has no formatting hook, offer format-on-edit as a fallback.
  **来自提案的 hook**（若已批准提案包含）：消费阶段 3 偏好队列中的 `hook` 条目。如果阶段 2 发现了格式化工具而队列中没有格式化 hook，可提供"编辑后格式化"作为兜底建议。

  For each hook preference (from the queue or the formatter fallback):

  对每条 hook 偏好（来自队列或格式化兜底）：

  1. Target file: default based on the Phase 1 CLAUDE.md choice — project → `.claude/settings.json` (team-shared, committed); personal → `.claude/settings.local.json`. Only ask if the user chose "both" in Phase 1 or the preference is ambiguous. Ask once for all hooks, not per-hook.
     目标文件：默认依据阶段 1 的 CLAUDE.md 选择 — 项目 → `.claude/settings.json`（团队共享、提交入库）；个人 → `.claude/settings.local.json`。仅当用户在阶段 1 选了"两者都要"或偏好不明确时才询问。对所有 hook 只问一次，而不是每个 hook 问一次。

  2. Pick the event and matcher from the preference:
     依据偏好选择事件和匹配器：
     - "after every edit" → `PostToolUse` with matcher `Write|Edit`
       "每次编辑之后" → `PostToolUse`，匹配器 `Write|Edit`
     - "when Claude finishes" / "before I review" → `Stop` event (fires at the end of every turn — including read-only ones)
       "Claude 完成时" / "在我审查之前" → `Stop` 事件（每轮结束时触发 — 包括只读轮次）
     - "before running bash" → `PreToolUse` with matcher `Bash`
       "运行 bash 之前" → `PreToolUse`，匹配器 `Bash`
     - "before committing" (literal git-commit gate) → **not a hooks.json hook.** Matchers can't filter Bash by command content, so there's no way to target only `git commit`. Route this to a git pre-commit hook (`.git/hooks/pre-commit`, husky, pre-commit framework) instead — offer to write one. If the user actually means "before I review and commit Claude's output", that's `Stop` — probe to disambiguate.
       "提交之前"（字面意义的 git 提交门禁）→ **不是 hooks.json hook。** 匹配器无法按命令内容过滤 Bash，因此没有办法只针对 `git commit`。应改为路由到 git pre-commit 钩子（`.git/hooks/pre-commit`、husky、pre-commit 框架）— 可主动提出代写一个。如果用户实际是指"在我审查并提交 Claude 的产出之前"，那是 `Stop` — 需追问以消歧。
     Probe if the preference is ambiguous.
     偏好不明确时进行追问。

  3. **Load the hook reference** (once per `/init` run, before the first hook): invoke the Skill tool with `skill: 'update-config'` and args starting with `[hooks-only]` followed by a one-line summary of what you're building — e.g., `[hooks-only] Constructing a PostToolUse/Write|Edit format hook for .claude/settings.json using ruff`. This loads the hooks schema and verification flow into context. Subsequent hooks reuse it — don't re-invoke.
     **加载 hook 参考**（每次 `/init` 运行一次，在第一个 hook 之前）：以 `skill: 'update-config'` 调用 Skill 工具，args 以 `[hooks-only]` 开头，后跟一行关于你要构建内容的摘要 — 如 `[hooks-only] Constructing a PostToolUse/Write|Edit format hook for .claude/settings.json using ruff`。这会把 hooks 架构和验证流程加载进上下文。后续 hook 复用它 — 不要重复调用。

  4. Follow the skill's **"Constructing a Hook"** flow: dedup check → construct for THIS project → pipe-test raw → wrap → write JSON → `jq -e` validate → live-proof (for `Pre|PostToolUse` on triggerable matchers) → cleanup → handoff. Target file and event/matcher come from steps 1–2 above.
     遵循该技能的 **"Constructing a Hook"** 流程：查重 → 为本项目构建 → 原始管道测试 → 包装 → 写 JSON → `jq -e` 校验 → 实测验证（对可触发的匹配器上的 `Pre|PostToolUse`）→ 清理 → 交接。目标文件与事件/匹配器来自上述第 1–2 步。

Act on each "yes" before moving on.

每得到一个"是"，先落实再继续。

## Phase 8: Summary and next steps / 阶段 8：总结与后续步骤

Recap what was set up — which files were written and the key points included in each. Remind the user these files are a starting point: they should review and tweak them, and can run `/init` again anytime to re-scan.

概述设置了什么 — 写入了哪些文件、每个文件包含哪些要点。提醒用户这些文件只是起点：应自行审阅和调整，并可随时再次运行 `/init` 重新扫描。

Then tell the user that you'll be introducing a few more suggestions for optimizing their codebase and Claude Code setup based on what you found. Present these as a single, well-formatted to-do list where every item is relevant to this repo. Put the most impactful items first.

然后告诉用户：基于本次发现，还会给出几条优化代码库和 Claude Code 配置的补充建议。以单一、格式良好的待办列表呈现，且每条都与本仓库相关。影响最大的条目放前面。

When building the list, work through these checks and include only what applies:

构建列表时，逐项核对这些检查，只收录适用的：

- If Phase 2 found Codex or Gemini CLI config: offer to import it now — tell the user to reply `/import` to scan and list what's importable (MCP servers, slash commands, subagents, skills, instructions), then `/import --yes=<digest>` (the scan output names the digest) to apply the user-level items. Do NOT read the foreign-agent config files or write Claude Code config yourself — the deterministic import (triggered by `--yes`) applies the same safe-name and path-traversal guards as the terminal picker. If `/import` isn't available on this surface, tell the user to run `claude import` from a terminal instead. Put this first — it saves re-entering config they already have.
  如果阶段 2 发现了 Codex 或 Gemini CLI 配置：主动提出现在导入 — 告诉用户回复 `/import` 以扫描并列出可导入内容（MCP 服务器、斜杠命令、子代理、技能、指令），然后回复 `/import --yes=<digest>`（扫描输出会给出 digest）以应用用户级条目。不要自行读取外部代理配置文件或改写 Claude Code 配置 — 由 `--yes` 触发的确定性导入会应用与终端选择器相同的安全名称和路径穿越防护。如果当前界面不支持 `/import`，让用户改在终端运行 `claude import`。这条放在最前 — 可免去重复录入用户已有的配置。

【评论】"不要自行读取外部代理配置，改由确定性导入命令处理"体现了对竞品配置文件的隔离处理：把敏感的迁移操作限制在带安全校验的受控路径中。

- If frontend code was detected (React, Vue, Svelte, etc.): `/plugin install frontend-design@claude-plugins-official` gives Claude design principles and component patterns so it produces polished UI; `/plugin install playwright@claude-plugins-official` lets Claude launch a real browser, screenshot what it built, and fix visual bugs itself.
  如果检测到前端代码（React、Vue、Svelte 等）：`/plugin install frontend-design@claude-plugins-official` 为 Claude 提供设计原则和组件模式，使其产出精致的 UI；`/plugin install playwright@claude-plugins-official` 让 Claude 能启动真实浏览器、对构建结果截图并自行修复视觉 bug。
- If you found gaps in Phase 7 (missing GitHub CLI, missing linting) and the user said no: list them here with a one-line reason why each helps.
  如果在阶段 7 发现缺口（缺 GitHub CLI、缺 lint）而用户拒绝了：在此列出，每条附一行说明其价值。
- If tests are missing or sparse: suggest setting up a test framework so Claude can verify its own changes.
  如果测试缺失或稀少：建议搭建测试框架，让 Claude 能验证自己的改动。
- To help you create skills and optimize existing skills using evals, Claude Code has an official skill-creator plugin you can install. Install it with `/plugin install skill-creator@claude-plugins-official`, then run `/skill-creator <skill-name>` to create new skills or refine any existing skill. (Always include this one.)
  为帮助你借助评测（eval）创建技能并优化既有技能，Claude Code 提供可安装的官方 skill-creator 插件。用 `/plugin install skill-creator@claude-plugins-official` 安装，然后运行 `/skill-creator <skill-name>` 来创建新技能或打磨既有技能。（此条必须包含。）
- Browse official plugins with `/plugin` — these bundle skills, agents, hooks, and MCP servers that you may find helpful. You can also create your own custom plugins to share them with others. (Always include this one.)
  用 `/plugin` 浏览官方插件 — 它们打包了技能、代理、hook 和 MCP 服务器，或许对你有帮助。你也可以创建自己的自定义插件分享给他人。（此条必须包含。）
