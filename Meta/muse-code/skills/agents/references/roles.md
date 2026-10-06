<!-- BILINGUAL-EN-ZH -->
# Roles: four brief templates / 角色：四个 brief 模板

A role only pre-fills the brief. There is no role verb, no `kind` beyond
`work` and `follow`, nothing the helper reads: paste a template into the
thread's `brief`, fill the blanks, and that text alone directs the thread.

角色只是预填简报（brief）。没有角色动词，除 `work` 与 `follow` 之外没有别的 `kind`，也没有任何助手要读的东西：把模板粘进线程的 `brief`，填好空格，仅凭那段文字指挥线程。

**Naming.** `name` = `<Role>`, `<Role> <given>` or `<Role> (<engine>)` —
`Researcher Ada`, `Tester (codex)`, `Implementer core.py`; `id` = the role
word or `role-given` (`researcher`, `researcher-ada`), so the user's word
matches the id. Every line you write leads with the name and puts the id in
brackets once, when the user may need to type it.

**命名。** `name` = `<Role>`、`<Role> <given>` 或 `<Role> (<engine>)`——如 `Researcher Ada`、`Tester (codex)`、`Implementer core.py`；`id` = 角色词或 `role-given`（`researcher`、`researcher-ada`），使用户的用词与 id 对得上。你写的每一行都以名字开头，并在用户可能需要键入时用方括号标注一次 id。

| `id` | `name` | the line the user reads |
| --- | --- | --- |
| `researcher` | `Researcher (muse)` | `Researcher (muse) [researcher] — reads the parser, read-only` |

| `id` | `name` | 用户读到的那一行 |
| --- | --- | --- |
| `researcher` | `Researcher (muse)` | `Researcher (muse) [researcher] — reads the parser, read-only` |

**Ownership.** Every writer's proposal row carries `owns`, the files it may
change; the brief tells each thread its own list and its siblings'. When two
scopes touch one file, split by files, not by feature, or order the second
thread on the first's branch.

**所有权。** 每个写入者的提案行都携带 `owns`，即可更改的文件；brief 告诉每个线程它自己的清单与同侪的清单。当两个范围触及同一个文件时，按文件切分而不是按功能切分，或者把第二个线程排在第一个线程的分支之上。

**Checkout.** `worktree: null` means the thread works in your own checkout
unless its `cwd` names another, and runs no git that writes there (no
`fetch`, `worktree add` or checkout in the user's clone). A build or a
branch takes a `worktree` of its own, or a clone under `<project>/library/`.

**检出。** `worktree: null` 表示线程在你的检出中工作，除非其 `cwd` 指定了别的位置，且不运行任何会写入那里的 git（在用户克隆中不运行 `fetch`、`worktree add` 或 checkout）。构建或分支工作要有自己的 `worktree`，或用 `<project>/library/` 下的克隆。

**The brief.** Open it with one plain line saying what this thread does —
that line is what siblings see in `## The project around you` and what the
user sees after `go`; the template goes below it. State the same four
things every time: objective, output format, what the thread may read, its
boundaries. Never repeat or contradict the helper's own `## How to work`
block (scratch paths, who lands, what to push). Never write "no questions"
or "run unattended": an unattended thread asks by `BLOCKED(HUMAN):` in its
report, with its options in that line, and never opens a TUI dialog —
nobody watches its pane, so the coordinator puts the question to the user.
When a design note is on main, the brief points at it and never calls
itself canonical over it.

**简报。** 以一句平实的话开头，说明本线程做什么——同侪在 `## The project around you` 中看到的、用户在 `go` 之后看到的就是这一行；模板放在它下面。每次都陈述同样四件事：目标、输出格式、线程可读什么、其边界。绝不重复或抵触助手自己的 `## How to work` 块（草稿路径、谁合入、推送什么）。绝不写"没有问题"或"无人值守运行"：无人值守线程通过报告中的 `BLOCKED(HUMAN):` 提问，选项写在该行里，且绝不打开 TUI 对话框——没有谁盯着它的窗格，由协调者把问题转给用户。当设计说明在 main 上时，brief 指向它，绝不自称比它更权威。

【评论】`BLOCKED(HUMAN):` 协议把"无人值守"重新定义为结构化的阻塞上报而非静默自行其是，是协调型多代理系统中的人机回路设计。

**Report.** `report <slug> <id> --file -` with the text on stdin: the helper
writes the file, so no editor dialog stalls an attended thread.

**报告。** `report <slug> <id> --file -`，文本走 stdin：文件由助手写出，编辑器对话框就不会卡住有人值守的线程。

**Timing.** A writer opens at `go`. A read-only role (reviewer, tester) is
proposed now and opened only in the turn that reads what it reviews or
tests — a `PR:` line, a report — never at `go` with nothing to look at.

**时机。** 写入者在 `go` 时打开。只读角色（评审者、测试者）现在提出，只在读取它所评审或测试之物的那一回合打开——一条 `PR:` 行或一份报告——绝不在 `go` 时无所可看就打开。

**Verification.** `test_command` on the proposal row is the goal's own
verification command, verbatim. Never plan a red target branch: the slice
that changes behaviour an existing test covers carries that test's update,
and a merge gate is the suite green, never a list of accepted failures.

**验证。** 提案行上的 `test_command` 是目标自身的验证命令，逐字照抄。绝不计划一个红色的目标分支：改动某既有测试所覆盖行为的切片要携带该测试的更新，合并门禁是测试套件全绿，绝不是一份"可接受失败"清单。

**Odds and ends.** A network or permission prompt inside a thread is one
`BLOCKED(HUMAN):` line to the coordinator, never a self-deny. Two threads
may carry one template (a best-of-two on two branches); a thread may carry
none; a thread proposed before the user's pick is proposed again with the
pick in its brief; a typed line is a steer, not a brief.

**零碎事项。** 线程内的网络或权限提示是给协调者的一行 `BLOCKED(HUMAN):`，绝不是自行拒绝。两个线程可以携带同一模板（两个分支上的二选一取优）；线程也可以不带模板；在用户做出选择之前提议的线程要带着该选择重新提议；用户键入的一行是方向修正，不是 brief。

**Splits by shape.** The rule is `SKILL.md` § 1; its
shapes: implementation → by owned files or modules (one owner per file); a
hunt → one investigator who lands the fix (a second only for another
hypothesis), plus at most one measurement thread; exploration or research →
by area or independent question (crate group, subsystem, hypothesis, source),
one researcher each, you combine the maps or answers; a small tree or a
dependent question → one thread; anything else → the same test. A brief that says "after X exists" is proposed now and opened in the
turn that reads X, never at `go`. The measurement thread's
first task runs at `go` on the unfixed baseline, at once; its numbers go to
the summary — the follow lands the PR on the PR's own green, never on that
thread's loop, unless the goal names the measurement as the merge condition.
A brief naming failures a
sibling will fix plans a red main: the slice carries the test update, or the
tests thread opens now on that branch and both land together.

**按形状切分。** 规则在 `SKILL.md` § 1；其形状：实现 → 按拥有的文件或模块（每个文件一个所有者）；追捕 → 一名落地修复的调查者（仅在另一假设时才有第二名），外加至多一个测量线程；探索或研究 → 按领域或独立问题（crate 组、子系统、假设、来源）划分，各一名研究者，由你合并地图或答案；小树或依赖性问题 → 一个线程；其余情形 → 同样的检验。说"在 X 存在之后"的 brief 现在提出，并在读取 X 的那一回合打开，绝不在 `go`。测量线程的第一个任务在 `go` 时于未修复的基线上立即运行；其数字进入摘要——follow 线程在 PR 自己变绿时落地 PR，绝不等那个线程的循环，除非目标把测量列为合并条件。brief 中点名"同侪将修复的失败"就是在计划一个红色 main：切片要携带测试更新，或者测试线程现在就在该分支上打开、两者一起落地。

## Implementer / 实现者

- Objective: <the change; the issue or spec it comes from; done means for
  this thread — a merged PR, a passing test>.
  目标：<改动；其来源 issue 或规格；对本线程"完成"意味着什么——已合并的 PR、通过的测试>。
- Output: a PR on branch `<worktree>`; the report's first line `PR: <url>`;
  a `STATUS:` line; the tests run, named, with their result; a `DECISIONS:`
  line for every value the brief did not fix (a version, a public name, a
  file outside `owns`) — or `BLOCKED(HUMAN):` when the user must choose.
  输出：`<worktree>` 分支上的一个 PR；报告首行 `PR: <url>`；一行 `STATUS:`；运行过的测试，具名并带结果；brief 未固定的每个取值（一个版本号、一个公开名称、`owns` 之外的文件）各一行 `DECISIONS:`——当必须由用户选择时则 `BLOCKED(HUMAN):`。
- Owns: `owns: [<files>]` on the proposal row — the brief then tells this
  thread and its siblings who owns what; touch nothing outside your list.
  所有：提案行上的 `owns: [<files>]`——brief 随后告知本线程与同侪谁拥有什么；不碰清单之外的任何东西。
- May read: the repository in your checkout, the project's `MEMORY.md` and
  `TASKS.md`, the linked issue and spec. `MEMORY.md` and other threads'
  checkouts are read-only.
  可读：你检出中的仓库、项目的 `MEMORY.md` 与 `TASKS.md`、关联的 issue 与规格。`MEMORY.md` 与其他线程的检出是只读的。
- Boundaries: edit only your checkout and the files you own; a failing test
  before the fix for a behaviour change; no rebase, no force-push; open the
  PR and hand it to the report — the follow thread drives it to merge;
  anything the user must decide goes on a `BLOCKED(HUMAN):` line.
  边界：只编辑你的检出与你拥有的文件；行为变更要先有修复前失败的测试；不 rebase、不 force-push；开 PR 并写入报告——由 follow 线程推进到合并；任何必须由用户决定的事都写在一行 `BLOCKED(HUMAN):` 上。
- Record: `worktree: "<branch>"`, `cwd: <repo>`.
  记录：`worktree: "<branch>"`、`cwd: <repo>`。

A work thread's brief ends at its push and the `PR:` line — "land it when
green", "merge it yourself", "push main without waiting on me", "if checks
fail, fix and push again" are the follow thread's brief only; a follow row is
never proposed or opened at `go`: it exists through `follow --pr` at the
first `PR:` line.

工作线程的 brief 止于其推送与 `PR:` 行——"变绿就合入"、"自己合并"、"不用等我直接推 main"、"检查失败就修复再推"只属于 follow 线程的 brief；follow 行绝不在 `go` 时提议或打开：它通过首个 `PR:` 行处的 `follow --pr` 而存在。

## Reviewer / 评审者

- Objective: review <PR urls or a diff> for <one lens: correctness,
  security, spec conformance>. Findings, not approval.
  目标：从<一个视角：正确性、安全、规格符合度>评审<PR url 或 diff>。交付发现，而不是批准。
- Output: one finding per line — file, why it matters, the smallest fix —
  tagged Blocker / Should / Nit; "no findings" is a valid report.
  输出：每行一条发现——文件、为何要紧、最小修复——标注 Blocker / Should / Nit；"无发现"也是有效报告。
- May read: the PR, its checks, the spec and issue, the repository at the
  PR's head; `MEMORY.md`.
  可读：PR 及其检查、规格与 issue、PR 头部的仓库；`MEMORY.md`。
- Boundaries: never approve, request changes on or resolve a thread of a
  sibling's PR — findings go to the coordinator, who decides; never push to
  it; no edits to any checkout.
  边界：绝不对同侪的 PR 批准、请求修改或解决会话串——发现交给协调者裁决；绝不向其推送；不编辑任何检出。
- Record: `worktree: null` when it only reads; a branch when it must build.
  Opened in the turn that reads the `PR:` line it reviews, not at `go`.
  记录：仅读取时 `worktree: null`；必须构建时用一个分支。在读取其所评审的 `PR:` 行的那一回合打开，不在 `go`。

## Tester / 测试者

- Objective: exercise <the change> black-box at <the surface: CLI, TUI,
  API>; the scenarios, listed.
  目标：在<某个表面：CLI、TUI、API>上黑盒演练<该改动>；列出各场景。
- Output: a table per scenario — steps, expected, observed, verdict — with
  the exact commands; logs and recordings as files under
  `<project>/library/`.
  输出：每个场景一张表——步骤、预期、观察、判定——附确切命令；日志与录像作为文件放在 `<project>/library/` 下。
- May read: the built product, the spec's scenarios, `MEMORY.md`; the
  repository read-only.
  可读：构建出的产品、规格中的场景、`MEMORY.md`；仓库只读。
- Boundaries: no code change unless the brief says so; a defect is a report
  line with reproduction steps, not a fix; never mark a scenario passed on a
  claim.
  边界：brief 未说明就不改代码；缺陷是带复现步骤的报告行，不是修复；绝不在仅有声明时把场景标记为通过。
- Record: `worktree: null`; a branch only when asked to fix as well.
  Opened in the turn that reads the `PR:` line or report it tests, not at
  `go`.
  记录：`worktree: null`；仅在被同时要求修复时才用分支。在读取它所测试的 `PR:` 行或报告的那一回合打开，不在 `go`。

## Researcher (read-only) / 研究者（只读）

- Objective: answer <the question> with evidence; say what the coordinator
  will decide with it.
  目标：用证据回答<问题>；说明协调者将据此决定什么。
- Output: report only — the answer first, then evidence with paths and
  links; longer material as files under `<project>/library/`; a
  `## Remember` section with the one or two lines the project should keep.
  输出：仅报告——先答案，再证据（带路径与链接）；较长材料作为文件放在 `<project>/library/` 下；一个 `## Remember` 部分，写项目应保留的一两行。
- May read: the repository, docs, issues, the web when the brief allows it;
  `MEMORY.md` and the sibling briefs.
  可读：仓库、文档、issue，brief 允许时的网络；`MEMORY.md` 与同侪 brief。
- Boundaries: read, never edit. `worktree: null`, no branch, no commit, no
  PR. Your `cwd` is the coordinator's own checkout: a file you change there
  changes the coordinator's tree — refuse any step that writes to it and say
  so in the report.
  边界：只读，绝不编辑。`worktree: null`，无分支、无提交、无 PR。你的 `cwd` 是协调者自己的检出：你在那里改动的文件会改动协调者的树——拒绝任何写入它的步骤，并在报告中说明。
- Record: `worktree: null`, `cwd: <the checkout to read>`.
  记录：`worktree: null`、`cwd: <要读取的检出>`。
