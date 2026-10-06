---
name: greenfield-project-scaffolding
description: Use only when all three gates already hold. (1) This same user turn explicitly says to start, implement, build, scaffold, or go ahead with a new project now; prior answers, planning, decisions, and reminders never authorize. (2) No named path or confirmed suitable here/current-folder target resolves placement. (3) Inspection is forbidden, or permitted top-level inspection has already proved the current root home-like, general-purpose, or falsely empty. Do not load this skill merely to check eligibility. Exclude existing work, bug fixes, component edits, server or verification work, and standalone, paste-ready, single-file, or snippet delivery.
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->
# Greenfield Project Scaffolding / 全新项目脚手架搭建

Choose the project root and its normal layout before the first write.

在第一次写入之前，先选定项目根目录及其常规布局。

1. Confirm current-turn authorization and a complete enough request.
   Authorization is satisfied only when the current user turn itself explicitly asks
   to start, implement, build, scaffold, or go ahead with a new project now. Answers
   to prior questions, planning or decision turns, and reminders that work remains do
   not authorize this workflow without those implementation-now words in the same
   user turn. If
   authorization is absent, stop without project-shape work or a file write. If product
   behavior or delivery constraints are missing, visibly truncated, or materially
   ambiguous, ask one focused clarification question and stop until the answer resolves
   the gap. Do not invoke this workflow merely because the deliverable is an application
   or game.
   确认当前轮次的授权以及足够完整的请求。只有当前用户轮次本身明确要求现在就开始、实现、构建、搭建或推进一个新项目时，授权才算满足。对先前问题的回答、规划或决策轮次，以及"还有工作未完成"的提醒，只要同一用户轮次中没有那些"现在就实现"的字眼，均不构成本工作流的授权。若授权缺失，停止执行，不做项目形态的工作，也不写任何文件。若产品行为或交付约束缺失、明显被截断或存在实质性歧义，提出一个聚焦的澄清问题然后停止，直到答案弥补缺口。不要仅仅因为交付物是一个应用或游戏就调用本工作流。
2. Resolve explicit placement before treating it as unknown. `here`, `this folder`,
   `the current folder`, `this directory`, and `the current directory` count as targets
   only when permitted facts confirm that the current directory is a suitable intended
   project root. A home-like, general-purpose, or false-empty current directory is not
   made suitable by deictic wording. Preserve an explicit named path and an existing
   project root. Existing repositories, projects, or notebooks remain excluded.
   在把放置位置视为未知之前，先解析明确的指定。`here`、`this folder`、`the current folder`、`this directory` 和 `the current directory` 只有在允许获取的事实确认当前目录是合适的预期项目根时，才可作为目标。类似主目录、通用用途或伪空的当前目录，不会因为指示性措辞而变得合适。保留明确的具名路径和既有的项目根。既有的仓库、项目或 notebook 仍在排除之列。
3. Honor inspection limits. When the user forbids inspecting existing files, honor that restriction.
   Use only the request, `pwd`, and path semantics. Do not list or read nearby content.
   When an explicit no-peek constraint leaves placement unresolved, create a
   user-meaningful dedicated subdirectory from the request and path semantics. Skip all
   discovery steps below.
   遵守检查限制。当用户禁止检查既有文件时，遵守该限制。仅使用请求本身、`pwd` 和路径语义。不要列出或读取附近内容。当明确的不窥探约束导致放置位置无法确定时，根据请求和路径语义创建一个对用户有意义的专用子目录，并跳过下述所有发现步骤。
4. Otherwise reuse the permitted inspection that triggered this workflow. Use the
   request, `pwd`, and observed top-level entries and project markers. Do not repeat
   an already completed read solely for this workflow. Inspect only placement facts
   that remain missing and are allowed by the user. Determine whether the user named
   a target and whether the current directory is already the intended project root.
   Do not adopt an unrelated nearby repository merely because it exists.
   否则复用触发本工作流的已允许检查。使用请求、`pwd` 以及已观察到的顶层条目和项目标记。不要仅为本工作流而重复已完成的读取。只检查仍然缺失且用户允许的放置事实。判断用户是否指定了目标，以及当前目录是否已是预期的项目根。不要仅仅因为附近存在某个不相关的仓库就采用它。
5. Reject a risky root before project work. When permitted inspection proves that the
   current directory is home-like, general-purpose, or contradicts an empty/current-root
   premise, treat it as requiring a dedicated child. Treat a home-like, general-purpose,
   or false-empty current root as closed to project artifacts. The first project mutation
   must establish the dedicated child. Keep every scaffold path and scaffold command
   working directory beneath the chosen root.
   在开展项目工作之前拒绝有风险的根目录。当允许的检查证明当前目录类似主目录、属通用用途，或与"空目录/当前根"的前提相矛盾时，将其视为需要专用子目录。类似主目录、通用用途或伪空的当前根对项目工件关闭。第一次项目变更必须先建立该专用子目录。所有脚手架路径和脚手架命令的工作目录都要保持在所选根之下。

   If this skill is loaded late after current-task project artifacts already exist at
   the rejected root, stop further project mutation; relocate only those current-task
   artifacts into the dedicated child before any other project mutation. Relocate rather
   than duplicate them. Do not copy, synchronize, or mirror them back to the rejected
   root. Continue exclusively in the dedicated child. Do not inspect, move, or delete
   unrelated content while repairing the task's paths.
   若本技能加载过晚、当前任务的工件已存在于被拒绝的根目录中，停止进一步的项目变更；在进行任何其他项目变更之前，仅把这些当前任务的工件迁移到专用子目录中。是迁移，而不是复制。不要把它们复制、同步或镜像回被拒绝的根目录。此后只在专用子目录中继续。修复任务路径时，不要检查、移动或删除无关内容。

   Choose exactly one root:
   选定且仅选定一个根目录：
   - Preserve an explicit named path or existing project root.
     保留明确的具名路径或既有的项目根。
   - Use the current directory when permitted facts confirm it is suitable and the
     request makes it already the intended project root.
     当允许获取的事实确认其合适、且请求已把当前目录当作预期项目根时，使用当前目录。
   - When the current directory is a home-like or general-purpose directory, or
     conflicts with the request's premise, and no target was named, create one
     user-meaningful dedicated subdirectory derived from the requested product.
     当当前目录类似主目录或属通用用途、或与请求的前提冲突，且未指定目标时，依据所请求的产品创建一个对用户有意义的专用子目录。
   Do not add another wrapper directory inside the chosen root.
   不要在所选根内再添加一层包装目录。
6. Use the ecosystem-native layout inside that root. Keep entry points, package files,
   source, tests, and assets where that ecosystem normally puts them. Create a
   directory only when the ecosystem convention or concrete near-term contents need
   it; do not create a directory that would contain only one file as decoration.
   Avoid placeholder files and speculative layers. For a playable browser app or game
   using no-build vanilla HTML/JavaScript that was not explicitly requested as
   standalone or single-file, keep the entry HTML in the dedicated project root and
   put substantial behavior in at least one separate authored `.js` or `.mjs` source file.
   For a framework or typed stack, use the normal source modules for that stack; do not
   force a root HTML file or a token JavaScript file. When the request explicitly chooses
   a standalone, single-file, paste-ready, or offline artifact shape, preserve it and
   defer artifact and verification mechanics to `bundled:browser-app-delivery`.
   在该根目录内使用生态系统原生的布局。入口文件、包文件、源代码、测试和资产放在该生态系统通常放置的位置。只有当生态系统约定或具体的近期内容需要时才创建目录；不要为了装饰而创建只装一个文件的目录。避免占位文件和投机性的分层。对于使用无构建原生 HTML/JavaScript、且未被明确要求做成独立单文件的可玩浏览器应用或游戏，把入口 HTML 保留在专用项目根中，并把实质性行为放进至少一个单独编写的 `.js` 或 `.mjs` 源文件。对于框架或带类型的栈，使用该栈正常的源码模块；不要强行生成根级 HTML 文件或象征性的 JavaScript 文件。当请求明确选择独立、单文件、即贴即用或离线的工件形态时，保持该形态，并把工件与验证机制交由 `bundled:browser-app-delivery` 处理。
7. Scaffold the smallest complete runnable shape the request needs, then use the
   applicable delivery or verification workflow. Report the chosen root clearly.
   搭建请求所需的最小完整可运行形态，然后使用适用的交付或验证工作流。清晰地报告所选的根目录。
8. Before handoff, audit known paths and paths created or changed for this task.
   Keep its scaffold artifacts under exactly one chosen project root. Relocate or
   remove only current-task artifacts outside it; do not inspect, move, or delete
   unrelated content.
   交接之前，审计已知路径以及为本任务创建或更改的路径。把本任务的脚手架工件保持在唯一一个所选项目根之下。只迁移或移除位于其外的当前任务工件；不要检查、移动或删除无关内容。

【评论】该技能要求授权必须来自"当前这一轮"用户消息，先前轮次的同意一概无效，这是一种防止智能体在多轮对话中自行扩大行动范围的约束设计。

Do not invoke this workflow for existing repositories, projects, or notebooks, bug fixes,
repository maintenance, component-only edits. Preserve an explicit standalone,
single-file, snippet, or paste delivery request without turning it into a project.

对既有仓库、项目或 notebook、bug 修复、仓库维护、仅组件级的修改，不要调用本工作流。保留明确的独立、单文件、代码片段或即贴即用交付请求，不要把它变成一个项目。
