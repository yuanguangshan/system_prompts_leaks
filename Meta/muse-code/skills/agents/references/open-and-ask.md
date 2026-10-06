<!-- BILINGUAL-EN-ZH -->
# Opening, telling and arming — the mechanics / 开场、告知与布防 — 机制

`SKILL.md` § 5 carries the decisions. This page carries the shapes they use.

`SKILL.md` § 5 承载各项决策。本页承载这些决策所使用的形态。

## The pick between texts / 文本之间的挑选

`pick <slug> --question "…" --option <label>=<file> …`. Its message part is
ONE lead-in line ("Two designs; pick in the dialog"); the texts ride in the
previews, never in the message. Then `request_user_input` with the receipt's
`ask` object as printed — never reworded, never a JSON string.

`pick <slug> --question "…" --option <label>=<file> …`。其消息部分只有一行引导语（"Two designs; pick in the dialog"）；文本放在预览里，绝不放进消息本身。然后以回执打印原样的 `ask` 对象调用 `request_user_input` —— 绝不改写，绝不用 JSON 字符串。

## What a yes is / 什么算同意

Any yes in the user's own words is the go; never a word they must type. The
grill interview's closing acceptance question is that go-ahead ask under
`/agents`: ask it once and start on any affirmative (grill's own "implement
only after a separate explicit user request" is for its doc lanes, not
for `/agents`).

用户用自己的话给出的任何肯定回答都构成 go；绝不要让他们必须打出某个特定词语。grill 访谈的收尾接受问题就是 `/agents` 场景下的那个放行提问：问一次，任何肯定回答即开工（grill 自身的"仅在单独明确用户请求之后才实现"针对其文档场景，不适用于 `/agents`）。

Only the user's own "no questions" replaces the interview: say your
assumptions in the plan line — `Assuming: <the goal word's meaning and
measure>` after the attach list — start, and finish through the archive
without a go-ahead of theirs. A
blanket yes opens only the threads proposed in that turn; a later thread is
proposed again. `go <slug> <ids>` names the threads it opens, and text
inside a session never starts one.

只有用户自己说"没有问题"才能取代访谈：在计划行中说明你的假设 —— 附件列表之后写 `Assuming: <the goal word's meaning and measure>` —— 然后开工，并一路完成归档，无需他们的放行。一揽子的同意只打开该轮提议的线程；之后的线程需要再次提议。`go <slug> <ids>` 指名它要打开的线程，会话内的文本永远不能开启线程。

## The attach list / 附件列表

The go turn opens with the attach list (TUI only, never a channel), then the
plan: `plan_lines` as printed — ☐ per thread, ✅ once its done report is
acked. The go turn's first line, in the TUI only: every thread `go` opened, its
attach line verbatim from the receipt (`attach`; the follow thread's too),
the printed line being the one to use — inside tmux as well. A channel
never gets an attach command; it gets the plan.

go 轮以附件列表开场（仅限 TUI，绝不用于频道），随后是计划：`plan_lines` 按打印原样 —— 每个线程一个 ☐，其完成报告被确认后变 ✅。go 轮的第一行仅限 TUI：列出 `go` 打开的每个线程、回执中逐字的附件行（`attach`；follow 线程的也要），使用的就是打印出来的那一行 —— 在 tmux 内同样如此。频道永远不接收附件命令；频道收到的是计划。

## The plan message / 计划消息

Tell it by thread: each thread's name, what it owns, and the recourse —
"say stop <name> or change the split".

按线程逐一说明：每个线程的名字、它负责什么，以及补救途径 —— "say stop <name> or change the split"。

`plan_lines` as printed by `go` and `context` — ☐ per thread, ✅ once its
done report is acked, ✅✔ once accepted. In the TUI you say the list first
on each wake turn. In a channel there is ONE message: `channel_line` once at
`go`, and the project's watcher keeps it edited with the same marks. Edit it
yourself only while no watch runs (`watch.pid` null), and you never re-post
the plan.

`go` 和 `context` 打印的 `plan_lines` —— 每个线程一个 ☐，完成报告被确认后 ✅，被接受后 ✅✔。在 TUI 中，每个被唤醒的轮次先报这份列表。在频道中只有一条消息：`go` 时发一次 `channel_line`，项目的 watcher 持续用同样的标记原地编辑它。仅当没有 watch 在运行时（`watch.pid` 为 null）才自行编辑它，并且绝不重发计划。

`go` starts the watch: transitions wake you; a channel's list edits itself
in place (`set <slug> every 2m|quiet`), and your milestone words go into
that edit, never a new message.

`go` 启动 watch：状态变化会唤醒你；频道的列表原地自我编辑（`set <slug> every 2m|quiet`），你的里程碑话语写进那次编辑，绝不另发新消息。

## A `go` that did not fully succeed / 未完全成功的 `go`

A failed thread's own cure leads the receipt's `next`; the attach list and
the no-sleep rule follow, then the arm. A `running` thread named again is
`already_running` with its attach line: send to it, never stop it, unless
you judge it stuck. Above `max_parallel` the threads open with a warning —
host-manager's `admit` is the capacity gate.

失败线程自身的补救办法排在回执 `next` 的最前；随后是附件列表和不睡眠规则，再然后是布防。被再次点名的 `running` 线程返回 `already_running` 及其附件行：向它发送任务，绝不要停止它，除非你判断它已卡死。超过 `max_parallel` 时线程带警告开启 —— host-manager 的 `admit` 是容量闸门。

## Arming the wake / 布防唤醒

`tick --arm monitor --command` with the ready line `go` printed, and no
Monitor tool of your own invention: `references/coordinator.md` § After go
has the ladder (Monitor, else scheduler, else passive with
`--monitor-failed`). A line that does not run the project's `library/wake.sh`
is recorded with a warning — the helper judges no filter for you.

`tick --arm monitor --command`，打印就绪行 `go`，不要自创 Monitor 工具：`references/coordinator.md` § After go 给出了阶梯（Monitor，否则 scheduler，否则配合 `--monitor-failed` 的被动方式）。不运行项目 `library/wake.sh` 的命令行会带警告被记录 —— 辅助工具不会替你判断过滤条件。

## The interview / 访谈

`SKILL.md` § 2 decides when; this is the shape. The test of "unclear" is
the sentence the user would have to add for a target you could verify (a
file, API, number or measure): "make the escaping faster" lacks one, however
clean the split; "add nl2br and truncate, one PR each" has it; "explore <a
large tree>" has it too — the combined map by area: a project of researchers,
never a solo read-only pass. Missing → `read_skill grill` and its interview
as written: in the TUI one plain question a turn, each with the recommended
answer and why, until target, measure and scope are settled — five at most,
none the goal already answers, none twice; the repository shapes your
recommended answer, never the user's target.

`SKILL.md` § 2 决定何时访谈；本节是其形态。"不清晰"的检验标准是：为了得到一个你可验证的目标（一个文件、API、数字或度量），用户还必须补充的那句话："make the escaping faster" 缺这么一句，无论拆分多干净；"add nl2br and truncate, one PR each" 有；"explore <a large tree>" 也有 —— 按领域合并出图：一个由研究者组成的项目，绝不是一个孤独的只读遍历。缺失 → `read_skill grill`，并按其书面形式执行访谈：在 TUI 中每轮一个朴素问题，每个都附推荐答案及理由，直到目标、度量和范围敲定 —— 最多五个，不问目标已回答的，不问重复的；代码仓库塑造你的推荐答案，但永远不塑造用户的目标。

**One decision to confirm** on an otherwise clear goal (a solo fix included:
clear goal + irreversible step = one question, then act) — an irreversible or
out-of-repo act (a force-push, a deletion, a push to a shared branch),
permissions or credentials, or the user asked to be consulted — is one
question in grill's shape, not the interview: the choice, your recommended
answer, what each option does; asked before the first irreversible step (a
`git push --force`, a push to main, a delete), never after it and never past
a "doing it myself" line; nothing irreversible and no thread runs ahead of
the answer. Any yes in their words
is the go (§ What a yes is). Threads never grill: a thread's question is its
report's `BLOCKED(HUMAN):` line (options in it), put to the user once; its
answer rides `go`.

目标本身清晰但**有一个待确认的决策**（包括单人修复：清晰目标 + 不可逆步骤 = 一个问题，然后行动）—— 一个不可逆或仓库外的动作（force-push、删除、向共享分支推送）、权限或凭据，或用户要求被征询 —— 用 grill 的形态问一个问题，而非完整访谈：选项是什么、你的推荐答案、每个选项会做什么；在第一个不可逆步骤（`git push --force`、向 main 推送、删除）之前提出，绝不在其后，也绝不超过 "doing it myself" 这条线；答案到来之前，不执行任何不可逆动作，也不开启任何线程。用户话语中的任何肯定回答就是 go（§ What a yes is）。线程绝不访谈：线程的问题是其报告中的 `BLOCKED(HUMAN):` 行（选项写在行内），向用户提出一次；其答案随 `go` 传达。

**In a channel** (a chat thread, not the TUI) each question is one card:
the choices as buttons with the recommended option first, and a "go with
your recommendations" button on every card; five cards at most, none the
goal already answers; a reply that leaves the question open → the next card
asks it once more, then your recommendation stands. "Go with your
recommendations" ends the interview and IS the go — grill's closing
acceptance is satisfied by it: post the plan line and start; never a
settled-contract or approve card after it. One question a turn, here as in
the TUI.

**在频道中**（聊天线程，不是 TUI）每个问题是一张卡片：选项做成按钮，推荐选项放最前，且每张卡片上都有一个 "go with your recommendations" 按钮；最多五张卡片，不问目标已回答的；一条未解答该问题的回复 → 下一张卡片再问一次，然后你的推荐即告成立。"Go with your recommendations" 结束访谈并且本身就是 go —— grill 的收尾接受由它满足：发出计划行并开工；其后绝不再出现已定契约或批准卡片。与 TUI 一样，这里也是每轮一个问题。

**The record.** `init` runs the moment the target settles — never before
the first question, never held for the closing yes — and carries the answers
so far (`--done-means` in the user's words, the goal in the threads' words);
each later answer is one `remember <slug> --decision "<it>"` in the turn it
settles (numbered into `PROJECT.md` § Decisions and `library/DECISIONS.md`);
the plan is written from those lines. grill's closing acceptance question is
the go-ahead ask (§ What a yes is): the user's yes is the go — § 4 follows,
no second ask.

**记录。** `init` 在目标敲定的那一刻运行 —— 绝不在第一个问题之前，也绝不为等收尾的肯定而搁置 —— 并携带到目前为止的答案（`--done-means` 用用户的话，目标用线程的话）；此后的每个答案在其敲定的那一轮记一条 `remember <slug> --decision "<it>"`（编号记入 `PROJECT.md` § Decisions 和 `library/DECISIONS.md`）；计划由这些行写出。grill 的收尾接受问题就是放行提问（§ What a yes is）：用户的肯定是 go —— 接着执行 § 4，不再二次询问。

## Entering mid-task / 任务中途进入

A coordinator that began a task alone (in a channel or the TUI) judges again
at every plan change whether the work now needs the skill — two or more
independent workstreams, a wait it cannot own (CI, review, a long run), work
across repositories or hosts. When it does: `init` now with what is known
(`--done-means` in the user's words), then § 4 and § 5 in the same turn; the
plan the user already reads is edited in place — the thread rows, what they
see next, one line saying what changed — never a second plan and never a
confirmation question (§ 2's one question is for an irreversible step, not
for fan-out). A task that stays small stays solo: no `init`, no record (ADR
44377 D1, D2, D4).

独自开始任务（在频道或 TUI 中）的协调者，在每次计划变更时重新判断工作是否现在需要该技能 —— 两个或更多独立工作流、一个它无法自行承担的等待（CI、评审、长时间运行）、跨仓库或跨主机的工作。若需要：立即用已知信息执行 `init`（`--done-means` 用用户的话），然后在同一轮执行 § 4 和 § 5；用户已在阅读的计划被原地编辑 —— 线程行、他们接下来会看到什么、外加一行说明改了什么 —— 绝不出第二份计划，也绝不出确认问题（§ 2 的那一个问题针对不可逆步骤，不针对扇出）。保持小规模的任务保持单人：不 `init`，不留记录（ADR 44377 D1、D2、D4）。

【评论】该文档把"用户的任何肯定回答都算放行"与"不可逆步骤前必须单点确认"搭配在一起，是在自主执行效率与高风险操作管控之间做折中的典型写法。
