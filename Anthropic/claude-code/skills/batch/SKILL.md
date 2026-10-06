---
name: batch
description: Research and plan a large-scale change, then execute it in parallel across 5–30 isolated worktree agents that each open a PR.
when_to_use: Use when the user wants to make a sweeping, mechanical change across many files (migrations, refactors, bulk renames) that can be decomposed into independent parallel units.
disable-model-invocation: true
---
<!-- BILINGUAL-EN-ZH -->

# Batch: Parallel Work Orchestration / Batch：并行工作编排

You are orchestrating a large, parallelizable change across this codebase.

你正在本代码库中编排一项大型、可并行的变更。

## User Instruction / 用户指令

$ARGUMENTS

## Phase 1: Research and Plan (Plan Mode) / 阶段 1：研究与规划（计划模式）

Call the `EnterPlanMode` tool now to enter plan mode, then:

现在调用 `EnterPlanMode` 工具进入计划模式，然后：

1. **Understand the scope.** Launch one or more subagents (in the foreground — you need their results) to deeply research what this instruction touches. Find all the files, patterns, and call sites that need to change. Understand the existing conventions so the migration is consistent.

   **理解范围。** 启动一个或多个子代理（在前台运行 —— 你需要它们的结果），深入研究这条指令涉及的内容。找出所有需要更改的文件、模式与调用点。理解现有的约定，确保迁移方式一致。

2. **Decompose into independent units.** Break the work into 5–30 self-contained units. Each unit must:

   **分解为独立单元。** 将工作拆分为 5–30 个自包含的单元。每个单元必须：
   - Be independently implementable in an isolated git worktree (no shared state with sibling units)
     能够在隔离的 git worktree 中独立实现（与兄弟单元不共享状态）
   - Be mergeable on its own without depending on another unit's PR landing first
     能够独立合并，而不依赖其他单元的 PR 先行合入
   - Be roughly uniform in size (split large units, merge trivial ones)
     规模大致均匀（拆分过大的单元，合并过小的单元）

   Scale the count to the actual work: few files → closer to 5; hundreds of files → closer to 30. Prefer per-directory or per-module slicing over arbitrary file lists.

   根据实际工作量调整数量：文件少 → 接近 5 个；数百个文件 → 接近 30 个。优先按目录或模块划分，而不是随意罗列文件。

3. **Determine the e2e test recipe.** Figure out how a worker can verify its change actually works end-to-end — not just that unit tests pass. Look for:

   **确定端到端（e2e）测试方案。** 弄清工作代理如何验证其变更确实端到端可用 —— 而不只是单元测试通过。寻找：
   - A `claude-in-chrome` skill or browser-automation tool (for UI changes: click through the affected flow, screenshot the result)
     `claude-in-chrome` 技能或浏览器自动化工具（针对 UI 变更：点击走通受影响的流程并对结果截图）
   - A `tmux` or CLI-verifier skill (for CLI changes: launch the app interactively, exercise the changed behavior)
     `tmux` 或 CLI 验证技能（针对 CLI 变更：以交互方式启动应用，实际操作被变更的行为）
   - A dev-server + curl pattern (for API changes: start the server, hit the affected endpoints)
     开发服务器 + curl 模式（针对 API 变更：启动服务器，请求受影响的端点）
   - An existing e2e/integration test suite the worker can run
     工作代理可以运行的现成 e2e/集成测试套件

   If you cannot find a concrete e2e path, use the `AskUserQuestion` tool to ask the user how to verify this change end-to-end. Offer 2–3 specific options based on what you found (e.g., "Screenshot via chrome extension", "Run `bun run dev` and curl the endpoint", "No e2e — unit tests are sufficient"). Do not skip this — the workers cannot ask the user themselves.

   如果找不到具体的 e2e 验证路径，使用 `AskUserQuestion` 工具询问用户如何端到端验证此变更。根据你的发现提供 2–3 个具体选项（例如"通过 chrome 扩展截图"、"运行 `bun run dev` 并 curl 该端点"、"无 e2e —— 单元测试已足够"）。不要跳过这一步 —— 工作代理无法自行询问用户。

   Write the recipe as a short, concrete set of steps that a worker can execute autonomously. Include any setup (start a dev server, build first) and the exact command/interaction to verify.

   将该方案写成一份简短、具体、可由工作代理自主执行的步骤清单。包含所有准备工作（启动开发服务器、先构建）以及用于验证的确切命令/交互。

4. **Write the plan.** In your plan file, include:

   **撰写计划。** 在计划文件中包含：
   - A summary of what you found during research
     研究阶段发现的摘要
   - A numbered list of work units — for each: a short title, the list of files/directories it covers, and a one-line description of the change
     工作单元的编号清单 —— 每个单元包括：简短标题、覆盖的文件/目录列表，以及一行变更描述
   - The e2e test recipe (or "skip e2e because …" if the user chose that)
     e2e 测试方案（或用户选择跳过时的"跳过 e2e，因为……"）
   - The exact worker instructions you will give each agent (the shared template)
     你将下发给每个代理的确切工作指令（共享模板）

5. Call `ExitPlanMode` to present the plan for approval.

   调用 `ExitPlanMode` 提交计划以供批准。

## Phase 2: Spawn Workers (After Plan Approval) / 阶段 2：启动工作代理（计划批准后）

Once the plan is approved, spawn one background agent per work unit using the `Agent` tool. **All agents must use `isolation: "worktree"` and `run_in_background: true`.** Launch them all in a single message block so they run in parallel.

计划获批后，使用 `Agent` 工具为每个工作单元启动一个后台代理。**所有代理必须使用 `isolation: "worktree"` 和 `run_in_background: true`。** 在单个消息块中一次性启动所有代理，使它们并行运行。

For each agent, the prompt must be fully self-contained. Include:

对每个代理，其提示词必须完全自包含。包括：
- The overall goal (the user's instruction)
  总体目标（用户的指令）
- This unit's specific task (title, file list, change description — copied verbatim from your plan)
  本单元的具体任务（标题、文件列表、变更描述 —— 从计划中逐字复制）
- Any codebase conventions you discovered that the worker needs to follow
  你发现的、工作代理需要遵循的代码库约定
- The e2e test recipe from your plan (or "skip e2e because …")
  计划中的 e2e 测试方案（或"跳过 e2e，因为……"）
- The worker instructions below, copied verbatim:
  下面的工作指令，逐字复制：

```
After you finish implementing the change:
1. **Code review** — Invoke the `Skill` tool with `skill: "code-review"` to find correctness bugs (it reports findings; it does not edit code). Fix any findings it surfaces before continuing.
2. **Run unit tests** — Run the project's test suite (check for package.json scripts, Makefile targets, or common commands like `npm test`, `bun test`, `pytest`, `go test`). If tests fail, fix them.
3. **Test end-to-end** — Follow the e2e test recipe from the coordinator's prompt (below). If the recipe says to skip e2e for this unit, skip it.
4. **Commit and push** — Commit all changes with a clear message, push the branch, and create a PR with `gh pr create`. Use a descriptive title. If `gh` is not available or the push fails, note it in your final message.
5. **Report** — End with a single line: `PR: <url>` so the coordinator can track it. If no PR was created, end with `PR: none — <reason>`.
```

Use `subagent_type: "general-purpose"` unless a more specific agent type fits.

除非有更贴合的代理类型，否则使用 `subagent_type: "general-purpose"`。

## Phase 3: Track Progress / 阶段 3：跟踪进度

After launching all workers, render an initial status table:

启动所有工作代理后，渲染一张初始状态表：

| # | Unit | Status | PR |
|---|------|--------|----|
| 1 | `<title>` | running | — |
| 2 | `<title>` | running | — |

| # | 单元 | 状态 | PR |
|---|------|--------|----|
| 1 | `<title>` | running | — |
| 2 | `<title>` | running | — |

As background-agent completion notifications arrive, parse the `PR: <url>` line from each agent's result and re-render the table with updated status (`done` / `failed`) and PR links. Keep a brief failure note for any agent that did not produce a PR.

随着后台代理完成通知的到达，从每个代理的结果中解析 `PR: <url>` 行，并以更新后的状态（`done` / `failed`）和 PR 链接重新渲染表格。对未产出 PR 的代理，保留一条简短的失败备注。

When all agents have reported, render the final table and a one-line summary (e.g., "22/24 units landed as PRs").

当所有代理都已汇报后，渲染最终表格和一行总结（例如"22/24 个单元已落地为 PR"）。
