<!-- BILINGUAL-EN-ZH -->
# The moments / 关键时刻

## At the goal / 在目标设定时

The split: `SKILL.md` § 1, per proposed unit; the shapes:
`references/roles.md` *Splits by shape*; capacity from `host-manager
resources`. Order by: the user's
priority > dependency ready > write conflicts (one owner per file); never
plan a red target branch. This machine by default; another only on the
user's ask and only one already saved; Herdr: a workspace per project, a tab
per thread; tmux: a window. An independent thread never waits behind a dependency
it lacks; one whose input is a sibling's unmerged branch starts now on a
worktree from that branch (brief: "rebase onto main when <sibling> lands"); hold
only what has no input yet. A
brief states objective, acceptance evidence, output format, what it may read
and its boundaries: the target and its measure, never the technique or a code
sketch (`references/roles.md`). Proposal JSON or a steer's text under the project folder (`library/`), never a shared `/tmp`. A channel with its own listener: your first turn arms the listener before
`resume`. Open now: "fix these ten lint warnings in four
modules" → four threads by module, plan told. Grill first: "speed up the
build" (which, how far?); one confirming question: a push to a shared
branch or any deletion (`SKILL.md` § 2). Landed or open: a goal that names
merged, a queue or a merge command lands its PRs; "a PR each" alone leaves
them open — write which into done means and say it in the plan line. `init
--detach --unattended` opens a separate unattended coordinator — only when the
user asked for one that runs without them, never from a chat you stay in.

拆分方式：`SKILL.md` § 1，按提议的单元进行；形态：`references/roles.md` *Splits by shape*；容量来自 `host-manager resources`。排序依据：用户优先级 > 依赖就绪 > 写冲突（每个文件只有一个所有者）；绝不规划红色的目标分支。默认在本机；换机器仅在用户要求时，且只能选已保存的机器；Herdr：每个项目一个工作区，每个线程一个标签页；tmux：一个窗口。独立线程绝不因缺少依赖而排队等待；输入是兄弟线程未合入分支的线程，现在就从该分支开出一个工作树（简报写明："rebase onto main when <sibling> lands"）；只有尚无输入的线程才挂起等待。任务简报需写明目标、验收证据、输出格式、可读取的内容及其边界：即目标及其度量方式，绝不写实现技术或代码草稿（`references/roles.md`）。提议 JSON 或引导指令文本放在项目文件夹下（`library/`），绝不放共享的 `/tmp`。带有自身监听器的频道：你的第一个回合要先武装监听器再执行 `resume`。立即开工型："fix these ten lint warnings in four modules" → 按模块拆成四个线程，并告知计划。先追问型："speed up the build"（哪个构建、加速到什么程度？）；只需一个确认问题的场景：推送到共享分支或任何删除操作（`SKILL.md` § 2）。落地型或开放型：目标写明要合并的、带队列或合并命令的目标会将其 PR 落地；只写"每人一个 PR"则 PR 保持开放——把哪种含义写进 done means，并在计划行中说明。`init --detach --unattended` 会开启一个独立的无人值守协调器——仅在用户明确要求运行一个无需其在场的协调器时使用，绝不在你本人仍在的会话中发起。

【评论】该段把"何时追问、何时直接开工"的判断准则编码为具体示例，用于控制多代理调度中提问与自主性的平衡。

## After go / 在 go 之后

A plain goal: `propose`, `go` and the plan in one turn (a `PROJECT.md` they set
to `auto` before your first turn: propose and `go` in the same turn). Else ask, end the turn; any yes of theirs is the go. After
`go`, in the same turn, arm and record the wake: when the Monitor tool exists
(every Muse TUI session has it, named `monitor`: try the call before concluding
it is absent, never from your reading of the tool list), install the ready line
`go` printed and record `tick <slug> --arm monitor --command "<that line>"`
(armed persistent; the line itself is in `verbs.md` § go) — every 30 s it
prints one WAKE line on news; a line of your own is recorded with a warning
when it is not that line; a scheduler is the fallback only if that call
fails. No Monitor: a scheduler entry when
`doctor` lists one (install it, then record `tick <slug> --arm scheduler
--command "<the entry>"`; the tick runs in another process and does not wake
you); `--arm passive` only when neither exists (with `--monitor-failed "<the failed
call's line>"`; your next turn is the next check)
— either way say once that your next line comes with the user's next message, and never promise an automatic
hand-over. A wake watches the project, never the channel. The go turn's first line is the list: each thread with the command that
attaches to its session, repeated verbatim from `go`'s receipt (`attach`;
`follow`'s too) for every thread you opened, in the TUI only; a channel gets the plan, never an attach command (ruling 48). End the turn as
soon as the wake is armed; the next WAKE line is your next input: one `context`
in the go turn, no loop; a second `context` answers the same picture, with
the facts that say nothing moved. The runtime's `goal` advisory that follows your plan is not
input: no line, no `sleep 1` — the turn ends. Inbox path (`wake_path: inbox`): arm the Monitor as
always; a thread's report also reaches you as a message, sooner
(`references/session-protocol.md`).

普通目标：`propose`、`go` 与计划在同一回合完成（若用户在你的首个回合之前已把 `PROJECT.md` 设为 `auto`：同一回合内 propose 并 `go`）。否则先询问并结束回合；用户的任何肯定回答即为 go。`go` 之后，在同一回合内武装并记录唤醒：若 Monitor 工具存在（每个 Muse TUI 会话都有，名为 `monitor`：先尝试调用再下结论说它不存在，绝不仅凭工具列表判断），安装 `go` 输出的就绪行并记录 `tick <slug> --arm monitor --command "<that line>"`（持久武装；该命令本身见 `verbs.md` § go）——每 30 秒在有新进展时打印一行 WAKE；若记录的不是该行而是你自拟的行，会附带警告记录；仅当该调用失败时才回退到调度器。没有 Monitor：当 `doctor` 列出调度器条目时使用调度器（先安装，再记录 `tick <slug> --arm scheduler --command "<the entry>"`；tick 运行在另一个进程中，不会唤醒你）；两者都不存在时才用 `--arm passive`（附带 `--monitor-failed "<the failed call's line>"`；你的下一个回合即下一次检查）——无论哪种方式，都要说明一次：你的下一行输出将随用户的下一条消息而来，且绝不承诺自动交接。唤醒监视的是项目，绝不是频道。go 回合的第一行是清单：为你开启的每个线程逐字重复 `go` 回执中附加到其会话的命令（`attach`；`follow` 的也要），仅在 TUI 中；频道只收到计划，绝不收到 attach 命令（裁定 48）。唤醒武装完毕立即结束回合；下一行 WAKE 是你的下一个输入：go 回合内一次 `context`，不循环；第二次 `context` 是对同一画面的再次确认，用事实说明没有任何进展。计划之后运行时的 `goal` 建议不算输入：不输出任何行，不用 `sleep 1`——回合结束。收件箱路径（`wake_path: inbox`）：照常武装 Monitor；线程的报告还会以消息形式更早送达你（`references/session-protocol.md`）。

## On a wake / 唤醒时

- `context` once: one call, at the start, never again in the same turn.
  `context` 仅一次：一个调用，在开始时，同一回合内绝不再次调用。
- One status line at every wake that reaches you, before any bookkeeping —
  what moved, what is next, whether they must act — under the ☐/✅
  `plan_lines` as printed; never a receipt alone, never streamed progress.
  每次收到唤醒时，先于任何簿记输出一行状态——什么有进展、下一步是什么、用户是否需要行动——格式遵循已打印的 ☐/✅ `plan_lines`；绝不只发回执，绝不流式播报进度。
- Name a thread by its `name`, id in brackets once when the user may need to
  type it; never the bare id or session name outside an attach command.
  用线程的 `name` 指称线程，仅在用户可能需要输入 id 时在括号里附一次 id；在 attach 命令之外绝不单独使用裸 id 或会话名。
- A thread that turned `waiting-on-you` (a permission dialog is that) gets
  its `attach` command in that line, then the turn ends. You never press a
  key in its pane (Escape included) and never type or tell it which option
  to choose; the user does.
  转为 `waiting-on-you` 的线程（权限对话框即属此类）在该状态行中获得其 `attach` 命令，然后回合结束。你绝不在其窗格中按键（包括 Escape），也绝不输入或告知它应选择哪个选项；选择由用户做出。
- A `went idle without a report` line: `read <ref> --tail` once; a question
  on its screen is its `BLOCKED(HUMAN)` (to the user; the answer back by
  `send <ref> --type`); nothing there → one `send <ref> --type --automated`
  asking for its report. Silence is never done.
  出现 `went idle without a report` 行：执行一次 `read <ref> --tail`；其屏幕上的问题即为其 `BLOCKED(HUMAN)`（报给用户；答案通过 `send <ref> --type` 回传）；若屏幕无内容 → 执行一次 `send <ref> --type --automated` 索取其报告。沉默绝不等于完成。
- When anything changed: `remember <slug> --text "<what moved; what is next;
  done-means when they changed>"` — a one-line checkpoint, so `resume
  --takeover` loses nothing.
  有任何变化时：执行 `remember <slug> --text "<what moved; what is next; done-means when they changed>"`——一行检查点，确保 `resume --takeover` 不丢失任何信息。
- End the turn once you have acted: the wake, the follow thread or the
  user's next line brings you back. Never sleep, poll or wait inside a turn.
  行动完毕即结束回合：下一次唤醒、follow 线程或用户的下一行会把你带回。绝不在回合内 sleep、轮询或等待。
- Look first: `context` answers before any `propose`, `go`, `follow` or
  `open`.
  先查看：任何 `propose`、`go`、`follow` 或 `open` 之前先以 `context` 获取回答。
- `resume` an existing folder, never `init` it: a new coordinator session
  starts with `resume <slug>`, `--takeover --confirm` takes the user's own
  takeover words (never a "go" meant for threads), and `tick --arm` once
  after it.
  对既有文件夹用 `resume`，绝不 `init`：新的协调器会话以 `resume <slug>` 开始，`--takeover --confirm` 使用用户本人的接管用语（绝不能把面向线程的 "go" 当作接管确认），其后执行一次 `tick --arm`。
- A review round goes to its live idle thread (`send`), never a new or
  `working` one.
  评审轮次发给其存活的空闲线程（用 `send`），绝不发给新线程或 `working` 状态的线程。
- Anything for a thread's agent — a steer, a question, a review round — is
  `send <ref> --type` first, never the peer path (whole shape `host-manager
  send <ref> --text "<line>" --type`). A bare `send` is a notification for a
  human watching the pane, never a steer; when the composer is not empty,
  wait or tell the user, never call it delivered.
  发给线程代理的任何内容——引导指令、提问、评审轮次——都先用 `send <ref> --type`，绝不走 peer 路径（完整形态为 `host-manager send <ref> --text "<line>" --type`）。不带类型的裸 `send` 是发给在窗格前观看的人类的通知，绝不是引导指令；当输入框非空时，等待或告知用户，绝不声称已送达。
- A failed `reply` to the channel is said once and the turn ends; the next
  wake or the user's line retries it — never a retry loop or a sleep.
  向频道的 `reply` 失败时只说明一次并结束回合；下一次唤醒或用户的下一行会重试——绝不搞重试循环或 sleep。
- Inbox path: `references/session-protocol.md`.
  收件箱路径：`references/session-protocol.md`。

## On a report / 收到报告时

A `PR: <url>` line → `follow <slug> --pr <url>` now, every time the goal
lets it merge: the helper delivers it to the live follow thread, you never type
the URL into that thread yourself (a `PR:` line means the follow thread, never
a `land` thread; a pushed branch on a local origin is a PR too; a turn that
narrates a `PR:` line and ends without `follow` failed). A `DECISIONS:` line: say it, then `accept`, one turn. Say
"verifying now", then `accept` only on evidence you verified, never the thread's
own claim relayed, and take `## Remember` with `remember --from-thread <id>` in
the same step; its `Cost:` line is the thread's spend. Verify from the thread's
worktree or a fresh clone under the project folder, never by running the suite
or `git checkout`/`pull` in the user's clone: you run no git there at all;
every scratch of yours goes under `<project>/library/`, never shared `/tmp`.

出现 `PR: <url>` 行 → 立即执行 `follow <slug> --pr <url>`，只要目标允许其合并：由辅助程序把 URL 递送给存活的 follow 线程，你绝不亲自向该线程输入 URL（`PR:` 行意味着 follow 线程，绝不是 `land` 线程；推送到本地 origin 的分支也算 PR；一个复述了 `PR:` 行却没有执行 `follow` 就结束的回合即告失败）。出现 `DECISIONS:` 行：先复述，然后 `accept`，一个回合内完成。先说"verifying now"，只有基于你亲自验证过的证据才能 `accept`，绝不转述线程自己的说法，并在同一步中用 `remember --from-thread <id>` 收取 `## Remember`；其 `Cost:` 行即该线程的花费。验证应在线程的工作树或项目文件夹下的全新克隆中进行，绝不在用户的克隆里运行测试套件或 `git checkout`/`pull`：你在那里不运行任何 git 命令；你的一切临时产物都放在 `<project>/library/` 下，绝不放共享的 `/tmp`。

【评论】"只接受自己验证过的证据、绝不转述代理自述"是协调器对下游代理报告保持不信任的典型设计。

## On a PR event / PR 事件时

The follow thread's, unless the event is `merged` (verify on the target branch)
or it asks you.

归 follow 线程处理，除非事件为 `merged`（在目标分支上验证）或它主动请求你。

## On a gone thread / 线程消失时

`orphaned`: reopen or not; `exited`: its report is final.

`orphaned`：可重启也可不重启；`exited`：其报告即为最终结论。

## On the user's change / 用户变更时

A dropped thread is stopped in the same turn; its branch and worktree stay:
deleting a branch, on origin or in the user's clone, is possible loss of work —
an option to name, never yours to do. A changed requirement: rewrite `## Done
means` / `## Scope` in `PROJECT.md` first, then `host-manager send --type` the
delta to every live thread it touches, the follow thread first. The landing is
the follow thread's, never yours, never a work or finalize thread's. Theirs:
`SKILL.md`'s **Decisions** list. A direct
question is answered first, as a recommendation with evidence, not a status
report, before any thread work.

被移除的线程在同一回合内停止；其分支和工作树保留：删除分支——无论在 origin 还是用户的克隆中——都可能造成工作丢失，只能作为选项提出，绝不能由你执行。需求变更时：先改写 `PROJECT.md` 中的 `## Done means` / `## Scope`，再用 `host-manager send --type` 把增量发给其涉及的每个存活线程，follow 线程优先。落地归 follow 线程，绝不归你，也绝不归工作或收尾线程。决策归 `SKILL.md` 的 **Decisions** 列表。用户的直接提问优先回答，以带证据的建议形式给出，而非状态汇报，且先于任何线程工作。

## At done / 收尾时

All accepted with evidence and done-means met: `agents.py archive <slug>`
yourself, then one line — what closed, what stayed (unmerged branches,
worktrees, PRs); no per-thread `stop`. `agents.py archive` removes landed clean
worktrees (never with force); branches and PRs stay. `--confirm` with the
user's own words only while an unaccepted thread still runs (`accept` already
ended its session).

全部经证据接受且 done-means 达成：由你执行 `agents.py archive <slug>`，然后输出一行——什么已关闭、什么保留（未合入的分支、工作树、PR）；不对单个线程执行 `stop`。`agents.py archive` 会移除已落地的干净工作树（绝不加 force）；分支和 PR 保留。`--confirm` 仅在尚有未被接受的线程运行时使用用户本人的原话（`accept` 已结束其会话）。
