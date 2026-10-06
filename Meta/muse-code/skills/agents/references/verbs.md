<!-- BILINGUAL-EN-ZH -->
# agents verb contract (`scripts/agents.py`) / agents 动词契约（`scripts/agents.py`）

The one helper this skill ships. It keeps a **project folder** — goal,
memory, threads, inbox — and turns the coordinator's decisions into
mechanics: it opens threads through `host-manager` (this machine) or
`fleet-manager` (a saved machine), records who each thread is, files
events once, and computes the groups a status answer is made of. It decides
nothing: which work becomes a thread, where it runs, when it is done and
what to remember are the coordinator's calls (`references/coordinator.md`).

这个 skill 附带的唯一辅助脚本。它维护一个**项目文件夹**——目标、记忆、线程、收件箱——并把协调者的决策转化为机制：通过 `host-manager`（本机）或 `fleet-manager`（一台已保存的机器）开启线程，记录每个线程是谁，对事件只归档一次，并计算出一条状态答复由哪些分组构成。它不做任何决策：哪些工作变成线程、在哪里运行、何时算完成、要记住什么，都由协调者判断（`references/coordinator.md`）。

```
python3 scripts/agents.py <verb> …
```

The six verbs the coordinator types most, as typed (the rest of this page
has every flag):

协调者最常输入的六个动词，按输入原样给出（本页其余部分列有全部旗标）：

```
propose <slug> --threads-json -   # stdin {"threads": [{"id","name","brief","worktree","owns":[…],"test_command","unattended",…}]}
tick <slug> --arm monitor --command "<the line go printed>"
follow <slug> --pr <url> [--pr <url>…]
stop <slug> <id> [<id>…]
ack <slug> <id> [--progress NN --basis "<why>"]
inbox drain <slug>
```

The helper is stdlib Python and runs by path from the skill directory. It
finds `host-manager` and `fleet-manager` as sibling skills
(`../host-manager/scripts/lane_runtime.py`,
`../fleet-manager/scripts/fleet_manager.py`); `MUSE_AGENTS_HOST_MANAGER` and
`MUSE_AGENTS_FLEET_MANAGER` name other commands (tests inject fakes there).
A fleet-manager counts only when its `open` verb answers `--help` (one probe
per run): a rename-only file is "found, but no `open` verb" in `doctor` and
no `remote_threads` capability; a helper that crashes or hangs on that
probe is reported by `doctor` (`fail`) and, on a verb that needs remote
sessions now, is a tool failure (exit 6 with its last stderr line), never
"no verb". Projects live under `~/.muse/projects/`; `MUSE_PROJECTS_HOME`
overrides the directory.

这个辅助脚本只用标准库 Python，按路径从 skill 目录运行。它把 `host-manager` 和 `fleet-manager` 作为相邻 skill 查找（`../host-manager/scripts/lane_runtime.py`、`../fleet-manager/scripts/fleet_manager.py`）；`MUSE_AGENTS_HOST_MANAGER` 和 `MUSE_AGENTS_FLEET_MANAGER` 可指定其他命令（测试在其中注入假件）。一个 fleet-manager 只有在其 `open` 动词能应答 `--help` 时才算数（每次运行探测一次）：一个只会改名的文件在 `doctor` 中是"找到了，但没有 `open` 动词"，且不具备 `remote_threads` 能力；在该探测上崩溃或挂起的辅助脚本会被 `doctor` 报告（`fail`），而在当下需要远程会话的动词上则是工具失败（exit 6 并附其最后一行 stderr），绝不会是"没有动词"。项目位于 `~/.muse/projects/` 之下；`MUSE_PROJECTS_HOME` 可覆盖该目录。

## One JSON shape / 单一 JSON 形状

Every verb prints **one JSON object** on stdout, success or error — the
same envelope `host-manager` and `fleet-manager` print (two text modes
are the exceptions: `tick --wake-line` prints one WAKE line, `pick` the
message to post, then a `--- dialog ---` line and the dialog spec):

每个动词在 stdout 上打印**一个 JSON 对象**，无论成功还是出错——与 `host-manager` 和 `fleet-manager` 打印的信封相同（两种文本模式是例外：`tick --wake-line` 打印一行 WAKE；`pick` 打印要发布的消息，随后是一行 `--- dialog ---` 和对话规格）：

| key | meaning |
| --- | --- |
| `outcome` | what happened, one word |
| `provider` | the session provider a thread line is about (`herdr`, `tmux`), or `null` |
| `ref` | `<slug>` for a project line, `<slug>/<thread>` for a thread line, or `null` |
| `capabilities` | what this installation can do: `local_threads`, `remote_threads` (fleet-manager found), `unattended_flag` (host-manager's `open` advertises `--unattended` — D16's spelling, under which a plain `open` is the engine's own prompts), `trusted_flag` (its `open` advertises `--trusted`: the engine's own trust record pre-seeded for a Claude Code or Codex thread, Amendment 6), `hooks_flag` (its `open` advertises `--hooks-approved`: the coordinator's hooks acceptance seeded into a thread's own settings file, Amendment 9) |
| `progress` | one line per step, in order; also written to stderr as it happens |
| `next` | the one thing to do next — a command with the flags in force, or "end the turn"; always present, and a repeated read is never the answer (§ What `next` means) |
| `error` | on a failure only: the first thing that is wrong |
| `receipt` | on a write verb only: `what`, `project`, `thread` (when one), `who`, `when`; `go` returns `receipts`, one per thread it opened |

| 键 | 含义 |
| --- | --- |
| `outcome` | 发生了什么，一个词 |
| `provider` | 该线程行所指的会话提供者（`herdr`、`tmux`），或 `null` |
| `ref` | 项目行为 `<slug>`，线程行为 `<slug>/<thread>`，或 `null` |
| `capabilities` | 本安装能做什么：`local_threads`、`remote_threads`（找到了 fleet-manager）、`unattended_flag`（host-manager 的 `open` 宣告 `--unattended`——D16 的拼法，在该拼法下不带修饰的 `open` 即引擎自身的提示）、`trusted_flag`（其 `open` 宣告 `--trusted`：为 Claude Code 或 Codex 线程预置引擎自身的信任记录，修正案 6）、`hooks_flag`（其 `open` 宣告 `--hooks-approved`：协调者的 hooks 验收被预置进线程自己的设置文件，修正案 9） |
| `progress` | 每步一行，按顺序；发生时也写入 stderr |
| `next` | 接下来要做的一件事——带现行旗标的命令，或"结束本轮"；始终存在，且"再次读取"永远不是答案（见 § What `next` means） |
| `error` | 仅失败时出现：第一个出错之处 |
| `receipt` | 仅写动词有：`what`、`project`、`thread`（如有）、`who`、`when`；`go` 返回 `receipts`，其开启的每个线程一条 |

Stdout carries slugs, ids, names and paths; never an environment value.
Verb-specific keys are listed per verb below. Every write verb takes
`--asked-by WHO` (default: `MUSE_AGENTS_ASKED_BY`, else the login name); it
goes into the receipt and, for a remote thread, in front of the
fleet-manager verb.

stdout 只承载 slug、id、名称和路径；绝不承载环境变量值。各动词专属的键在下方按动词列出。每个写动词都接受 `--asked-by WHO`（默认：`MUSE_AGENTS_ASKED_BY`，否则取登录名）；它会进入回执，对远程线程则置于 fleet-manager 动词之前。

## Environment / 环境变量

| variable | meaning |
| --- | --- |
| `MUSE_PROJECTS_HOME` | the projects directory (default `~/.muse/projects`); passed into every local thread |
| `MUSE_AGENTS_HOST_MANAGER` / `MUSE_AGENTS_FLEET_MANAGER` | the sibling helpers' commands (default: the sibling skills' `scripts/`) |
| `MUSE_AGENTS_ASKED_BY` | the default for `--asked-by` |
| `MUSE_AGENTS_TMUX` | the tmux command the coordinator's own sessions live on (`tmux -L <socket>`); rides as host-manager's global `--tmux` on every call, so threads open, are found and are stopped on that server; unset, host-manager's default |
| `MUSE_AGENTS_PROJECT`, `MUSE_AGENTS_THREAD`, `MUSE_AGENTS_ROLE` | set on a thread by `go`; `remember` refuses `MUSE_AGENTS_ROLE=thread`. `MUSE_AGENTS_ROLE=launcher` is set by a program that runs `init` for a coordinator session it opens afterwards (a launcher): `init` records no coordinator (`coordinator: null`) and that session's first `resume` binds |
| `MUSE_EXPERIMENTAL_AGENTS=on` | passed by `init --detach`, `go` and `follow` into every local session they open (the skill was visible here, so the new session must see it too — ADR 38715 D15); fleet-manager's `open` takes no `--env`, so a remote thread gets it only from its machine's own environment (a `progress` line says so) |
| `AGENTS_TEST_LAUNCHER_ARGV` | tests only, under the seams switch: a JSON list standing in for the launching session's command line |
| `AGENTS_TEST_COORDINATING_PID` | tests only, under the seams switch: the pid that stands in for the coordinating process (the suite pins its own, so no test's identity follows who ran `run.sh` and whether that shell outlived the first tests — #42960) |
| `AGENTS_TOOL_TIMEOUT_S` | the failure bound on one sibling-helper call (default 120); never a wait |
| `AGENTS_NOW` | a fixed clock for tests |
| `MUSE_AGENTS_TEST_SEAMS` | `1` enables the test-only seams; any other value ignores them (`ignored_env` in `progress`) |
| `AGENTS_TEST_HOLD` | tests only, under the switch above: `<point>:<ready>:<go>` — at `inbox-put` or `accept-judged` (both inside the lock, the latter after the `done`/`proposed` guards) the helper opens the go pipe, writes `reached` to the ready pipe and blocks reading the go pipe, so a suite can prove the lock is held and what a second writer does meanwhile |

| 变量 | 含义 |
| --- | --- |
| `MUSE_PROJECTS_HOME` | 项目目录（默认 `~/.muse/projects`）；传入每个本地线程 |
| `MUSE_AGENTS_HOST_MANAGER` / `MUSE_AGENTS_FLEET_MANAGER` | 相邻辅助脚本的命令（默认：相邻 skill 的 `scripts/`） |
| `MUSE_AGENTS_ASKED_BY` | `--asked-by` 的默认值 |
| `MUSE_AGENTS_TMUX` | 协调者自身会话所在的 tmux 命令（`tmux -L <socket>`）；每次调用都作为 host-manager 的全局 `--tmux` 携带，使线程在该服务器上开启、被找到、被停止；未设置时用 host-manager 的默认值 |
| `MUSE_AGENTS_PROJECT`、`MUSE_AGENTS_THREAD`、`MUSE_AGENTS_ROLE` | 由 `go` 设置在线程上；`remember` 拒绝 `MUSE_AGENTS_ROLE=thread`。`MUSE_AGENTS_ROLE=launcher` 由一个为协调者会话执行 `init`、随后开启该会话的程序（启动器）设置：`init` 不记录协调者（`coordinator: null`），而该会话的第一次 `resume` 完成绑定 |
| `MUSE_EXPERIMENTAL_AGENTS=on` | 由 `init --detach`、`go` 和 `follow` 传入它们开启的每个本地会话（这个 skill 在此处可见，新会话也必须看见——ADR 38715 D15）；fleet-manager 的 `open` 不接受 `--env`，因此远程线程只能从其所在机器自身的环境获得它（有一条 `progress` 行说明这一点） |
| `AGENTS_TEST_LAUNCHER_ARGV` | 仅测试用，需 seams 开关：一个 JSON 列表，顶替发起会话的命令行 |
| `AGENTS_TEST_COORDINATING_PID` | 仅测试用，需 seams 开关：顶替协调进程的 pid（测试套件固定自己的 pid，因此任何测试的身份都不随谁运行了 `run.sh`、该 shell 是否比最早的测试活得更久而变化——#42960） |
| `AGENTS_TOOL_TIMEOUT_S` | 单次相邻辅助脚本调用的失败上界（默认 120）；绝不是一个等待 |
| `AGENTS_NOW` | 测试用的固定时钟 |
| `MUSE_AGENTS_TEST_SEAMS` | `1` 启用仅测试的 seams；其他任何值忽略它们（`progress` 中的 `ignored_env`） |
| `AGENTS_TEST_HOLD` | 仅测试用，需上述开关：`<point>:<ready>:<go>`——在 `inbox-put` 或 `accept-judged`（都在锁内，后者在 `done`/`proposed` 守卫之后）处，辅助脚本打开 go 管道、向 ready 管道写入 `reached`、阻塞读取 go 管道，使测试套件能证明锁被持有，以及第二个写入者在此期间会怎样 |

`state.json`, the inbox and every thread record are written under one lock
file (`<slug>/.lock`, `flock`): threads run `report` and `inbox put` while
the coordinator runs `tick`, `context` and `resume`; each read-modify-write
pair is atomic; `tick` and the live `follow` path re-read a record under
the lock after their (unlocked) liveness probe, `follow`'s reopen path
derives the fresh record from a locked re-read, so a report filed meanwhile
is kept, and `accept` judges `done`/`proposed` under the lock.

`state.json`、收件箱和每条线程记录都在同一把锁文件（`<slug>/.lock`，`flock`）之下写入：线程运行 `report` 和 `inbox put` 的同时，协调者运行 `tick`、`context` 和 `resume`；每对读-改-写都是原子的；`tick` 和活的 `follow` 路径会在其（不加锁的）存活探测之后于锁内重读记录，`follow` 的重开路径从一次加锁重读推导出新鲜记录，从而保住其间归档的报告；`accept` 在锁内判定 `done`/`proposed`。

【评论】用一个 `flock` 锁文件串起所有读写，是典型的多写者一致性设计：辅助脚本与协调者并发访问同一份状态时，靠锁保证读-改-写的原子性。

## One exit-code table / 单一退出码表

| code | meaning |
| --- | --- |
| `0` | ok |
| `2` | usage — the message names the flag or argument |
| `3` | refused by a guard: no such project or thread (`"archived"` when the slug's folder is in the archive: the line carries `"archive_path"` and `"archived_at"`, `next` reads it), a slug already taken, a thread not named or not proposed, a live coordinator or live threads without confirmation, a memory write from a thread, an acceptance of a thread that is already done or never ran, a hand-written Monitor line refused at arm time (`arm_not_persistent`; `next` is the ready line). Nothing changed |
| `4` | unsupported here: no host-manager beside this skill, or a machine named without fleet-manager; `next` names the alternative |
| `5` | stopped for a step only a human can take (host-manager or fleet-manager stopped with 5); `next` is that step |
| `6` | evidence unavailable: a provider could not answer, a thread could not be opened. Nothing changed unless the line says `created: true` |
| `7` | internal — report the line; nothing changed |

| 码 | 含义 |
| --- | --- |
| `0` | 正常 |
| `2` | 用法错误——消息会指出是哪个旗标或参数 |
| `3` | 被守卫拒绝：不存在这样的项目或线程（slug 的文件夹已进归档时为 `"archived"`：该行携带 `"archive_path"` 与 `"archived_at"`，`next` 会读取它）、slug 已被占用、线程未命名或未提议、存在活的协调者或活的线程而未经确认、来自线程的记忆写入、对已 done 或从未运行过的线程的验收、arm 时拒绝手写的 Monitor 行（`arm_not_persistent`；`next` 是就绪行）。什么都没有改变 |
| `4` | 此处不支持：skill 旁没有 host-manager，或指名了机器却没有 fleet-manager；`next` 给出替代方案 |
| `5` | 因只有人才能执行的步骤而停下（host-manager 或 fleet-manager 以 5 退出）；`next` 就是那个步骤 |
| `6` | 证据不可得：提供者无法应答，线程无法开启。除非该行写明 `created: true`，否则什么都没有改变 |
| `7` | 内部错误——报告这一行；什么都没有改变 |

A host-manager or fleet-manager exit code is passed through unchanged with
the underlying line under `underlying`.

host-manager 或 fleet-manager 的退出码原样透传，底层的那一行放在 `underlying` 之下。

## What `next` means / `next` 的含义

`next` coaches the turn, never the loop: it names the one thing to do after
the verb, and it never answers `context <slug>` or `overview <slug>` after a
write. Judgement stays `SKILL.md`'s.

`next` 指导的是这一轮，而不是循环：它说出该动词之后要做的一件事，并且在一次写操作之后永远不会以 `context <slug>` 或 `overview <slug>` 作答。判断力仍归 `SKILL.md`。

【评论】"工具只给下一步、判断留给协调者提示词"是这一族设计的核心分工：脚本负责确定性机制，LLM 层负责节奏与取舍。

| after | `next` |
| --- | --- |
| `doctor` | the first turn's `init` |
| `init` | `propose <slug> --threads-json -` when `--done-means` was given; else write `## Done means`, then that proposal line |
| `init` on a request that begins `/agents` or `agents:` | the same, prefixed: under /agents you do the work yourself unless it needs several lanes at once or a long wait (then threads, or one follow thread); the plan line names the choice and why (ADR 38715 Amendment 10, #42241; it replaced the several-threads default of owner rulings 30/31) |
| `context` | write `## Done means`; `propose …` when no thread exists; the inbox moves in order (`follow <slug> --pr <url>` for a `PR:` report, the `BLOCKED(HUMAN)` question, `ack`, verify-then-`accept` of the thread that opened a merged PR (the follow thread ends on its own final report), reopen-or-not for a gone thread), then end the turn; the facts say so (`since_last_context_s`, `unchanged_for_s`, `context_call`) when `changed` is empty on a turn's first look; `end the turn now — context says how long nothing has moved and what brings you back; else answer the user and end the turn |
| `propose` | show the list, end the turn, `go <slug> <ids>` on the user's later go — or that `go` line alone under `start_threads: auto`; each ends `a thread is a host-manager session, opened by `go` alone; never `subagent_spawn` for project work` (owner ruling 52: the in-process subagent path has no report and no wake) |
| `go` | repeat `text`; `tick <slug> --arm monitor\|scheduler --command "…"` (or `--arm passive --monitor-failed "<line>"` when neither exists; the host-manager sentence above rides on this `next` too) and end the turn when no wake is recorded; else repeat `text` (each thread's attach command) in the TUI — in a channel `channel_line` alone: an attach command never reaches a channel (owner ruling 48) — then `no sleep of any length (the wake brings the reports)` and the end-turn line last, on every path — a first go, a later go under an armed wake (a thread opened after a pick, under a wake already armed), a reopen: `end the turn now; the Monitor wakes you.` — a note to the coordinator, no timestamp, never posted (it was pasted into plans) (a scheduler: `… the scheduler ticks in another process and the user's next message brings you back`; passive: `… nothing wakes you (passive) — the user's next message brings you back`) |
| `tick --arm`, `tick` | the same end-turn line as `go` after an arm (`end the turn now; the Monitor wakes you.`, or the tier's own words); a plain `tick` ends the turn and looks again on the next wake — or, when no wake is recorded, the arm line; when `changes` is non-empty, `remember <slug> --text "<what moved; what is next>"` first (the checkpoint of a changing wake) |
| `follow` | repeat `text` to the user (the follow thread with its attach command); end the turn; `no sleep of any length (the wake brings the reports)`; the follow thread reports here — the same no-sleep words ride `go` (a first go, a reopen and a partial go with a live thread — newly opened or already running — alike; on a partial go the failed thread's cure leads, then the attach-list words and the no-sleep words, then the arm and the end-turn line last) and the `context` move that relays a `BLOCKED(HUMAN)` answer (#41850: `sleep 240/280/90` after each of those); a `queued` follow receipt ends `never land it yourself; no sleep of any length (the wake brings the reports)` |
| `report`, `inbox put` | filed for the coordinator's next wake (a `PR:` report names the `follow` line the coordinator runs in the turn it reads it) |
| `ack` of a done report (a `PR:` line or a done STATUS) | `verify now; when done-means is met, `accept <slug> <id>` in this wake (that closes its session); else say what is missing` — one per done thread acked, `then end the turn` last (#41851: acked done reports sat unaccepted, sessions lingering, until the user asked) |
| `ack` of a progress report, `accept`, `remember`, `stop`, `inbox drain` | continue with what you have; end the turn when nothing else is pending — `inbox drain` first says `<n> report(s) unread — <names>` when a report is still neither acked nor accepted: the drain empties the inbox, not your reading |
| `relay --asked` | wait for the user's answer; on it, `relay <slug> <id> --fingerprint <fp> --answer "<their answer>"`; end the turn |
| `relay --answer` (delivered) | the answer is with the thread; the question closes on its next report; end the turn; `no sleep of any length (the wake brings the reports)` |
| `relay --answer` (failed) | NOT delivered, with the reason: the relay stays owed — the same `relay … --answer` again once the send can land; say what was NOT delivered, never that it was forwarded |
| `overview` | answer the user from `text` (a `waiting-on-you` line carries the thread's attach command again), then end the turn |
| `pick` | post everything above the `--- dialog ---` line verbatim as your message, then ask with the JSON below it as printed (its options carry the texts too); end the turn |
| `resume` | arm your own wake |
| a refusal | the command that clears it |

| 之后 | `next` |
| --- | --- |
| `doctor` | 第一轮的 `init` |
| `init` | 给了 `--done-means` 时是 `propose <slug> --threads-json -`；否则先写 `## Done means`，再写那条提议行 |
| 对以 `/agents` 或 `agents:` 开头的请求执行 `init` | 同上，但加前缀：在 /agents 下你自己动手做，除非它需要多条泳道同时进行或一次长时间等待（那就用线程，或一个 follow 线程）；计划行写出这个选择及理由（ADR 38715 Amendment 10，#42241；它取代了业主裁决 30/31 的多线程默认） |
| `context` | 写 `## Done means`；不存在线程时 `propose …`；收件箱按顺序处理（`PR:` 报告用 `follow <slug> --pr <url>`、`BLOCKED(HUMAN)` 提问、`ack`、对开启了已合并 PR 的线程先验证后 `accept`（follow 线程在其自己的最终报告后结束）、对已消失的线程决定是否重开），然后结束本轮；当本轮首次查看时 `changed` 为空，由事实说话（`since_last_context_s`、`unchanged_for_s`、`context_call`）；`end the turn now — context says how long nothing has moved and what brings you back; else answer the user and end the turn` |
| `propose` | 展示列表，结束本轮，待用户随后发话再 `go <slug> <ids>`——或在 `start_threads: auto` 下只给那条 `go` 行；每条都以后面这句收尾：线程是 host-manager 会话，只由 `go` 开启；项目工作绝不用 `subagent_spawn`（业主裁决 52：进程内 subagent 路径既无报告也无唤醒） |
| `go` | 重复 `text`；`tick <slug> --arm monitor\|scheduler --command "…"`（两者都不存在时用 `--arm passive --monitor-failed "<line>"`；上面那句 host-manager 的话也搭在这条 `next` 上），没有记录到唤醒时结束本轮；否则在 TUI 里重复 `text`（每个线程的 attach 命令）——在频道中只给 `channel_line`：attach 命令绝不进入频道（业主裁决 48）——然后是 `no sleep of any length (the wake brings the reports)`，结束本轮的话放最后，每条路径都如此——首次 go、在已布防唤醒下的再次 go（pick 之后开启的线程，唤醒已布防）、重开：`end the turn now; the Monitor wakes you.`——给协调者的一句备注，无时间戳，绝不发布（它曾被粘贴进计划）（scheduler 则是：`… the scheduler ticks in another process and the user's next message brings you back`；passive 则是：`… nothing wakes you (passive) — the user's next message brings you back`） |
| `tick --arm`、`tick` | 布防之后与 `go` 相同的结束本轮行（`end the turn now; the Monitor wakes you.`，或该层级自己的措辞）；不带布防的 `tick` 结束本轮，下次唤醒再看——没有记录到唤醒时则是布防行；当 `changes` 非空时，先 `remember <slug> --text "<what moved; what is next>"`（变动唤醒的检查点） |
| `follow` | 向用户重复 `text`（follow 线程及其 attach 命令）；结束本轮；`no sleep of any length (the wake brings the reports)`；follow 线程在此报告——同样的不睡眠话术也搭在 `go` 上（首次 go、重开、以及带一个活线程的部分 go——新开启或已在运行——都一样；部分 go 时先说失败线程的补救，再说 attach 列表的话与不睡眠的话，然后是布防，结束本轮的话放最后）以及转发 `BLOCKED(HUMAN)` 回答的那个 `context` 步骤（#41850：每步之后 `sleep 240/280/90`）；`queued` 的 follow 回执以后面这句收尾：绝不要自己去落地它；任何长度的睡眠都不要（唤醒会带来报告） |
| `report`、`inbox put` | 为协调者的下一次唤醒归档（`PR:` 报告会写出协调者读到它的那一轮要运行的 `follow` 行） |
| 对 done 报告的 `ack`（一行 `PR:` 或 done STATUS） | `verify now; when done-means is met, `accept <slug> <id>` in this wake (that closes its session); else say what is missing`——每个被 ack 的 done 线程一条，`then end the turn` 放最后（#41851：被 ack 的 done 报告一直无人验收、会话一直挂着，直到用户开口） |
| 对进度报告的 `ack`、`accept`、`remember`、`stop`、`inbox drain` | 用手上已有的继续；没有别的事待办时结束本轮——`inbox drain` 在有报告既未被 ack 也未被 accept 时会先说 `<n> report(s) unread — <names>`：清空的是收件箱，不是你的阅读 |
| `relay --asked` | 等用户的回答；拿到后执行 `relay <slug> <id> --fingerprint <fp> --answer "<他们的回答>"`；结束本轮 |
| `relay --answer`（已送达） | 回答已随线程送达；问题在其下一次报告时关闭；结束本轮；`no sleep of any length (the wake brings the reports)` |
| `relay --answer`（失败） | 未送达，附原因：这次转发仍然欠着——等发送可以落地时再次执行同样的 `relay … --answer`；说出什么"未"送达，绝不说"已转发" |
| `overview` | 用 `text` 回答用户（`waiting-on-you` 行会再次携带该线程的 attach 命令），然后结束本轮 |
| `pick` | 把 `--- dialog ---` 行之上的全部内容原样作为你的消息发布，然后按其下打印的 JSON 原样提问（其选项也带有那些文本）；结束本轮 |
| `resume` | 布防你自己的唤醒 |
| 一次拒绝 | 解除该拒绝的那条命令 |

`--help` on the helper and on every verb lists the flags with one line each;
the helper's source is not for reading.

辅助脚本本身和每个动词的 `--help` 都会逐行列出旗标；辅助脚本的源码不是给人读的。

## The project folder / 项目文件夹

```
~/.muse/projects/<slug>/
  PROJECT.md              goal, request (the message as typed), "done means", scope, repositories, standing instructions, settings
  MEMORY.md               shared memory — written only through `remember`
  TASKS.md                the user's checklist (pointers; issues and PRs are the truth)
  state.json              schema agents-project/v1: coordinator, repos (as init recorded them; a thread inherits trust only from one the user also trusted in Muse's trust.json — init says which lack a record), wake arm, cursors; on the inbox path only (absent = the Monitor path, FR-41038-5): `wake_path: inbox` (recorded at init and kept until archive — ADR 41038 D5; the Monitor wake is the floor on both paths until #41228), `inbox_target` / `inbox_label` / `inbox_reason` (the report target: this session's Muse session id, the workspace label it was resolved under, the reason when none)
    tracking.json           schema agents-tracking/v1: the progress ledger — workstreams, thread → workstream, one observation per live thread per tick, pending questions (below)
  threads/<id>/record.json   schema agents-thread/v1 (below)
  threads/<id>/brief.md      what the thread was told at open
  threads/<id>/report.md     the thread's own report (whole-file rewrite; the brief names this path)
  threads/<id>/leftover/     report files archive moved out of a landed checkout
  threads/<id>/remember.md   the report's `## Remember` section, held for the coordinator
  library/                 files threads produce for the project; wake.sh, the helper's own wake loop
  inbox/new/<seq>-<hash>.json   unprocessed events
  inbox/done/<seq>-<hash>.json  processed events (kept for idempotency)
```

`PROJECT.md` is human-editable Markdown. `init` writes these headings and
`context` reads them: `## Goal`, `## Request`, `## Done means`, `## Scope`,
`## Repositories`, `## Standing instructions`, `## Settings`. Settings are
`key: value` lines under `## Settings`:

`PROJECT.md` 是人可编辑的 Markdown。`init` 写入这些标题、`context` 读取它们：`## Goal`、`## Request`、`## Done means`、`## Scope`、`## Repositories`、`## Standing instructions`、`## Settings`。设置是 `## Settings` 之下的 `key: value` 行：

| key | default | meaning |
| --- | --- | --- |
| `max_parallel` | `4` | threads `go` will hold open at once |
| `start_threads` | `propose` | `propose` (wait for `go`) or `auto` (the user said so) |
| `unattended` | `inherit` | posture of new threads: `inherit` = this coordinator's own approval posture (approvals off → unattended); `true`/`false` on the user's words |
| `follow_every` | `5m` | the follow thread's cadence, written into its brief |
| `stuck_scans` | unset | the project's own flat-run length in wake rounds — ticks at which another thread moved, never raw 30 s ticks — that raises a thread's `flat` flag; unset = the number alone (`quiet_for_s`), no flag |

| 键 | 默认 | 含义 |
| --- | --- | --- |
| `max_parallel` | `4` | `go` 同时保持打开的线程数 |
| `start_threads` | `propose` | `propose`（等 `go`）或 `auto`（用户明确说过） |
| `unattended` | `inherit` | 新线程的姿态：`inherit` = 本协调者自身的审批姿态（审批关闭 → 无人值守）；`true`/`false` 按用户的原话 |
| `follow_every` | `5m` | follow 线程的节奏，写入其 brief |
| `stuck_scans` | 未设置 | 项目自己的平坦运行长度，以唤醒轮计——以"其他线程有移动的 tick"计，绝不是原始的 30 秒 tick——超过该长度会置起线程的 `flat` 旗标；未设置 = 只看数值本身（`quiet_for_s`），不设旗标 |

`MEMORY.md` is `## <date> <heading>` entries. `TASKS.md` is `- [ ]` lines.

`MEMORY.md` 是 `## <date> <heading>` 条目。`TASKS.md` 是 `- [ ]` 行。

**Thread record** (`threads/<id>/record.json`, `schema: agents-thread/v1`):
`id`, `name`, `kind` (`work` | `follow`), `status`, `brief` (the coordinator's
text), `cwd`, `repo`, `worktree` (the branch name from the proposal, or
null), `worktree_path` (the checkout `go` made for it, or null), `machine`
(`local` or a label), `provider`, `ref` (the provider's bare ref: the
address is composed from `machine` and `ref`, once), `server`, `engine` (set
at `propose` — the proposal's, else the coordinator's own — and confirmed
from the open receipt's identity), `engine_args` (the proposal's list),
`identity` (the provider's tuple as host-manager or fleet-manager reports
it; a remote thread's `status` answer is compared to it field by field, and
`identity_drift` lists the fields that differed, `key: 'was' -> 'now'`
each, as fleet-manager's `status` serves them, when a session under the
thread's name was not the one opened for it), `model`,
`effort`, `unattended`, `posture_source` (which rule decided the requested
posture — `thread`, `project`, `inherited`, `unreadable`; FR-38715-15),
`allow_list_path` (the engine's own rules file the helper wrote into the
thread's checkout; `null` when none), `posture_applied` (what the open receipt reports:
the helper's own word when it names one, e.g. `engine_default`; else from
its posture flag list — `unattended` when a flag applied, `attended` when
none did and none was asked, `unknown` when unattended was asked and the
receipt carries no flag, as host-manager adds it for Muse alone;
`provider_default` when the helper took no `--unattended`), `proposed_at`, `amended_at`
(`propose --replace`), `opened_at`, `receipt` (host-manager's), `open_note`
(the provider's note on open, when it gave one), `attempts` (failed opens),
`prs` (`[{url, last_event, head, at}]`), `report` (`{digest, at, status_line, blocked_line, decisions_line, pr, remember, progress}` — `progress` is the thread's `Progress: NN% — <basis>` line as `{percent, basis}`, null without one), `acked_digest`, `calibrated` (`{value, basis, rung, at}` from `ack --progress`, null until then), `workstream` and `workstream_ref` (the proposal's title and reference, null without), `evidence` (`[]` until `accept`),
`ended_at`, `stop_receipt` and `session_ended_at` (when `stop` or `agents.py archive`
ended its session), `last_agent_status` (the last screen or Herdr verdict
`tick` saw, so a thread that turns waiting-on-you files once per transition).

**线程记录**（`threads/<id>/record.json`，`schema: agents-thread/v1`）：`id`、`name`、`kind`（`work` | `follow`）、`status`、`brief`（协调者的文本）、`cwd`、`repo`、`worktree`（提案中的分支名，或 null）、`worktree_path`（`go` 为它做的检出，或 null）、`machine`（`local` 或一个标签）、`provider`、`ref`（提供者的裸 ref：地址由 `machine` 和 `ref` 组装，只组装一次）、`server`、`engine`（在 `propose` 时设置——用提案的，否则用协调者自己的——并由开启回执的身份确认）、`engine_args`（提案的列表）、`identity`（host-manager 或 fleet-manager 报告的提供者元组；当该线程名下的会话并非为它开启的那个时，远程线程的 `status` 应答会与它逐字段比对，`identity_drift` 列出有差异的字段，每个为 `key: 'was' -> 'now'`，按 fleet-manager 的 `status` 所给）、`model`、`effort`、`unattended`、`posture_source`（决定所请求姿态的是哪条规则——`thread`、`project`、`inherited`、`unreadable`；FR-38715-15）、`allow_list_path`（辅助脚本写进线程检出里的引擎自有规则文件；没有则为 `null`）、`posture_applied`（开启回执所报告的内容：回执自己点名时用它自己的词，如 `engine_default`；否则按其姿态旗标列表——有旗标生效为 `unattended`，无旗标生效且无人要求为 `attended`，要求了 unattended 而回执不带旗标为 `unknown`——host-manager 只为 Muse 附加它；辅助脚本未带 `--unattended` 为 `provider_default`）、`proposed_at`、`amended_at`（`propose --replace`）、`opened_at`、`receipt`（host-manager 的）、`open_note`（提供者开启时的备注，如给出）、`attempts`（失败的开启）、`prs`（`[{url, last_event, head, at}]`）、`report`（`{digest, at, status_line, blocked_line, decisions_line, pr, remember, progress}`——`progress` 是线程的 `Progress: NN% — <basis>` 行，形如 `{percent, basis}`，没有则为 null）、`acked_digest`、`calibrated`（`ack --progress` 得来的 `{value, basis, rung, at}`，此前为 null）、`workstream` 与 `workstream_ref`（提案的标题与引用，没有则为 null）、`evidence`（`accept` 之前为 `[]`）、`ended_at`、`stop_receipt` 与 `session_ended_at`（当 `stop` 或 `agents.py archive` 结束其会话时）、`last_agent_status`（`tick` 看到的最后一条屏幕或 Herdr 判定，于是一次转为 waiting-on-you 的线程每次转变只归档一次）。

**Thread status** is a recorded fact: `proposed` → `running` → (`exited` when
the provider proves it gone after a report) → `done` (only `accept`); `stopped`
(by `stop` or `agents.py archive`); `orphaned` (gone or identity failed with no report,
set by `tick` or `resume`). A thread's own "done" is a claim in its report,
never a status. A status is about the record; the session may outlive it
(a `done` thread's engine keeps running until `stop` or `agents.py archive` ends it).

**线程状态**是记录在案的事实：`proposed` → `running` →（当提供者在一次报告之后证明其已消失时为 `exited`）→ `done`（只有 `accept` 能给出）；`stopped`（由 `stop` 或 `agents.py archive` 设置）；`orphaned`（已消失、或身份比对失败且无报告，由 `tick` 或 `resume` 设置）。线程自己说的"done"只是其报告中的一个声明，绝不是状态。状态是关于记录的；会话可能比它活得更久（`done` 线程的引擎会一直运行，直到 `stop` 或 `agents.py archive` 结束它）。

**Groups** (computed by `context` and `overview` from facts, in this order,
first match wins):

**分组**（由 `context` 和 `overview` 从事实计算得出，按此顺序，首个匹配胜出）：

| group | fact |
| --- | --- |
| `done` | `status = done` (evidence recorded) |
| `orphaned` | `status = orphaned`, or a live check answers gone/mismatch and there is no report newer than `opened_at` |
| `waiting-on-you` | the report's `BLOCKED(HUMAN):` line is newer than `acked_digest`, or the provider reports the agent `blocked` (Herdr's own status, read from host-manager's `list` row — its `status` verb answers liveness only), or (tmux, this machine) the visible screen shows a dialog or permission prompt (`evidence: screen`, `screen: prompt on screen`) |
| `ready-for-review` | a report exists whose digest is not `acked_digest` |
| `landing` | any `prs[].last_event` in `enqueued`, `queued`, `merging` |
| `unreachable` | the provider answered `transport_unreachable` for a running thread (ADR 41038 § Failure modes): its transport is down and it keeps running there — never `orphaned`, never listed under `unknowns`; the row carries `unreachable` (the provider's words) and its attach line, and `text` lists it with the attach command; `tick` records nothing for it |
| `working` | live and the provider reports `working` or no status at all (liveness only: an engine Herdr does not detect), or (tmux) the screen shows an activity line (`evidence: screen`); a tmux screen that says nothing readable leaves liveness alone as the verdict and the row says `evidence: liveness` |
| `idle` | live and the provider reports any other status (`idle`, `done`, `unknown` — the groups host-manager's own `list` row gives them), or (tmux) the screen shows an empty composer with nothing running, or `status = exited` with its report acked, or `status = stopped` |
| `proposed` | `status = proposed` — written by `propose`, not yet opened by `go` |

| 分组 | 事实 |
| --- | --- |
| `done` | `status = done`（证据已记录） |
| `orphaned` | `status = orphaned`，或存活检查应答为消失/不匹配，且没有比 `opened_at` 更新的报告 |
| `waiting-on-you` | 报告的 `BLOCKED(HUMAN):` 行比 `acked_digest` 更新，或提供者报告 agent `blocked`（Herdr 自己的状态，从 host-manager 的 `list` 行读取——其 `status` 动词只答存活），或（tmux、本机）可见屏幕显示对话框或权限提示（`evidence: screen`、`screen: prompt on screen`） |
| `ready-for-review` | 存在一条报告，其 digest 不是 `acked_digest` |
| `landing` | 任一 `prs[].last_event` 处于 `enqueued`、`queued`、`merging` |
| `unreachable` | 提供者对运行中的线程应答 `transport_unreachable`（ADR 41038 § Failure modes）：其传输通道断了但线程仍在那边运行——绝不是 `orphaned`，也绝不列入 `unknowns`；该行携带 `unreachable`（提供者的原话）及其 attach 行，`text` 带着 attach 命令列出它；`tick` 不为它记录任何东西 |
| `working` | 存活且提供者报告 `working` 或根本没有状态（仅存活：一个 Herdr 检测不到的引擎），或（tmux）屏幕显示活动行（`evidence: screen`）；一个读不出可读内容的 tmux 屏幕就只以存活作为判定，该行写 `evidence: liveness` |
| `idle` | 存活且提供者报告任何其他状态（`idle`、`done`、`unknown`——host-manager 自己的 `list` 行给出的那些分组），或（tmux）屏幕显示空的输入框且无东西在运行，或 `status = exited` 且其报告已被 ack，或 `status = stopped` |
| `proposed` | `status = proposed`——由 `propose` 写入，尚未被 `go` 开启 |

A `group` on host-manager's `read` line (the msp provider's own state:
`working|waiting-on-you|idle|unknown`) decides before any screen reading;
`unknown` falls through to it (before it was read, every msp
thread read `working` by liveness alone). A tmux or msp provider's `status` knows liveness only, so for a live local thread the
helper reads the visible screen once (host-manager `read <ref> --tail`) and
judges it: known dialog and permission-prompt lines, the engine's activity
line, an empty composer. A thread whose provider could not answer has no
group: it is listed under `unknowns` with `unknown_reason` (the provider's
own words) and named in `text` with its machine and last known status.

host-manager `read` 行上的 `group`（msp 提供者自己的状态：`working|waiting-on-you|idle|unknown`）先于任何屏幕读取做决定；`unknown` 会落到它这里（在读取它之前，每个 msp 线程仅凭存活都读作 `working`）。tmux 或 msp 提供者的 `status` 只知道存活，因此对活的本地线程，辅助脚本会读一次可见屏幕（host-manager `read <ref> --tail`）并加以判断：已知的对话框与权限提示行、引擎的活动行、空的输入框。提供者无法应答的线程没有分组：它被列在 `unknowns` 之下，带 `unknown_reason`（提供者的原话），并在 `text` 中连同其机器与最后已知状态一起点名。

## The tracking ledger / 跟踪台账

`tracking.json` (schema `agents-tracking/v1`) is the project's progress
ledger — pointers and checkpoints, never task truth (issues, PRs and
artifacts stay authoritative), written under the same project lock as
`state.json` by atomic replace, rebuilt from the records when missing, and
loud when it cannot be read: every verb that touches it then answers
`tracking_error` (the path and the cure — move the file aside; the next tick
seeds a new ledger from the records and the earlier series is lost) and the
status table is that one line, never an empty table. It holds:

`tracking.json`（schema `agents-tracking/v1`）是项目的进度台账——指针与检查点，绝不是任务真相（issue、PR 和工件仍是权威），与 `state.json` 在同一把项目锁下以原子替换写入，缺失时从记录重建，读不了时大声报错：凡是触及它的动词此时都应答 `tracking_error`（路径与补救——把文件挪开；下一次 tick 会从记录播种一本新台账，此前的序列丢失），且状态表就是那一行，绝不是空表。它持有：

- `workstreams`: `{<id>: {id, title, source_ref}}` — `id` is the title's
  slug, `title` and `source_ref` the proposal's; `threads`: `{<thread id>:
  <workstream id> | null}`. Both are rebuilt from the records (`workstream`,
  `workstream_ref`) on every write: a workstream groups the status table
  and nothing else — never a session name or a label segment.
  `workstreams`：`{<id>: {id, title, source_ref}}`——`id` 是标题的 slug，`title` 与 `source_ref` 来自提案；`threads`：`{<thread id>: <workstream id> | null}`。两者在每次写入时从记录（`workstream`、`workstream_ref`）重建：工作线只为状态表分组，不作他用——绝不是会话名或标签段。
- `observations`: `{<thread id>: [point, …]}`, append-only — one point per
  running thread per `tick` (unchanged values included: a flat series is the
  stuck evidence), one point at `ack` and `accept` for that thread, and one
  transition point when a thread's status changed since its last point
  (stopped, exited, orphaned, done — the series ends there). A point:
  `at`, `group`, `status`, `live`, `self_reported` (`{value, basis, source:
  report}` or null), `artifact_rung` (`{value, basis}`), `calibrated`
  (`{value, basis, rung, at}` or null), `progress` (the one shown value,
  below), `current_action` (the report's `STATUS:` line), `pending_question`
  (an open entry's fingerprint or null), `blocker` (the report's
  `BLOCKED(HUMAN):` line), `evidence` (PR URLs with their last event, then
  the `accept` evidence), `flags`.
  `observations`：`{<thread id>: [point, …]}`，只追加——每个运行中的线程每个 `tick` 一个点（数值未变也记：平坦序列正是卡住的证据）、该线程在 `ack` 和 `accept` 时各一个点、以及当线程状态自其上一个点之后发生变化时的一个转变点（stopped、exited、orphaned、done——序列到此为止）。一个点：`at`、`group`、`status`、`live`、`self_reported`（`{value, basis, source: report}` 或 null）、`artifact_rung`（`{value, basis}`）、`calibrated`（`{value, basis, rung, at}` 或 null）、`progress`（所显示的那一个值，见下）、`current_action`（报告的 `STATUS:` 行）、`pending_question`（未决条目的指纹或 null）、`blocker`（报告的 `BLOCKED(HUMAN):` 行）、`evidence`（PR URL 及其最后事件，然后是 `accept` 证据）、`flags`。
- `pending_questions`: `[{fingerprint, thread, text, surfaced_at, status,
  closed_at, relay_state, asked_at, relay}]` — one per question: a report's
  `BLOCKED(HUMAN):` line (`report:<id>:<digest>`) or a dialog on the screen
  (`dialog:<id>:<stamp>`, once per transition). Open until the thread moves
  on — the next report supersedes the line, the screen leaves the dialog, or
  the thread is done — which is the answer's delivery as seen from here;
  `ack` closes nothing (the user still has the question). A report question
  carries its relay lifecycle (#44029, § `relay`): `relay_state` is
  `blocked-unasked` at open, `asked-relay-owed` once the ask (or an answer)
  is recorded, `relayed-awaiting-worker` once the relay's send is delivered;
  `asked_at` is the recorded ask and `relay` the recorded receipt
  (`{thread, fingerprint, answer, at, send}`), both `null` until then. A
  dialog entry's three relay fields are `null`: it is answered at its
  screen, never relayed.
  `pending_questions`：`[{fingerprint, thread, text, surfaced_at, status, closed_at, relay_state, asked_at, relay}]`——每个问题一条：报告的 `BLOCKED(HUMAN):` 行（`report:<id>:<digest>`）或屏幕上的一个对话框（`dialog:<id>:<stamp>`，每次转变一条）。保持打开，直到线程继续前进——下一次报告取代该行、屏幕离开对话框、或线程完成——从这里的视角看，这正是回答的送达；`ack` 什么都不关闭（问题仍在用户手里）。报告问题携带自己的转发生命周期（#44029，§ `relay`）：`relay_state` 在打开时为 `blocked-unasked`，记下提问（或一个回答）后为 `asked-relay-owed`，转发的发送送达后为 `relayed-awaiting-worker`；`asked_at` 是记录的提问，`relay` 是记录的回执（`{thread, fingerprint, answer, at, send}`），两者在此之前都是 `null`。对话框条目的三个转发字段都是 `null`：它在自己的屏幕上被回答，从不转发。

**Artifact rung** — the evidence value from the record's facts alone,
never from a claim: `proposed` 0; opened with no PR 25 (`no PR yet`); a PR
open (any event but a landing or `merged`) 50 (`PR open`); `enqueued`,
`queued` or `merging` 90 (`landing`); `merged` 95; `status: done` 100
(`accepted`). Several PRs take the lowest, the basis counting the merged
ones. **Shown value** (`progress` on a row and a point): the coordinator's
`calibrated` value while the rung it was judged against still holds — evidence
that moved since (a merge, an accept) supersedes it until the next `ack
--progress` — else the rung itself; never above the rung, and a
`self_reported` value never raises it (the user sees one value; the two raw
tracks stay in the ledger for the coordinator).

**工件档位（artifact rung）**——只从记录的事实得出的证据值，绝不出自声明：`proposed` 0；已开启但无 PR 25（`no PR yet`）；PR 已开（除 landing 或 `merged` 外的任何事件）50（`PR open`）；`enqueued`、`queued` 或 `merging` 90（`landing`）；`merged` 95；`status: done` 100（`accepted`）。多个 PR 取最低档，basis 计入已合并的那些。**显示值**（行与点上的 `progress`）：只要被评判时的档位仍然成立，就用协调者的 `calibrated` 值——此后移动了的证据（一次合并、一次验收）会取代它，直到下一次 `ack --progress`——否则就是档位本身；绝不高于档位，`self_reported` 的值也绝不抬高它（用户只看到一个值；两条原始轨迹都留在台账里供协调者查看）。

【评论】进度显示刻意以"证据档位"封顶、线程自报值只作参考，这是对 agent 自报进度不可信这一问题的防御性设计：对外只呈现可验证的里程碑。

**Series facts** on every `context`/`overview` row: `elapsed_s` (since the
open), `quiet_for_s` (since the series last changed — the first point of the
trailing unchanged run; since the open before any point),
`pending_question` (the open entry or null) and `flags`, candidate facts the
coordinator judges: `flat` (the trailing unchanged run spans at least the
project's `stuck_scans` wake rounds — ticks at which another thread's point
changed, never raw ticks; no setting, no flag),
`regressed` (the thread's own `Progress:` fell below its earlier one, or the
artifact rung fell below its peak — a lost artifact; never the coordinator's
own downward `ack --progress`, which is its judgment, not the thread falling
back). No universal threshold is coded, and the
helper estimates no finish time: the facts are the series, the rung and the
elapsed time, and the reading is yours.

每个 `context`/`overview` 行上的**序列事实**：`elapsed_s`（自开启起）、`quiet_for_s`（自序列上次变化起——即末尾未变段的第一个点；尚无任何点时则自开启起）、`pending_question`（未决条目或 null）与 `flags`，供协调者判断的候选事实：`flat`（末尾未变段至少横跨项目的 `stuck_scans` 个唤醒轮——以其他线程的点发生变化的 tick 计，绝不是原始 tick；未设置则无此旗标）、`regressed`（线程自己的 `Progress:` 跌到其早前值之下，或工件档位跌到其峰值之下——丢了工件；绝不是协调者自己向下调的 `ack --progress`，那是协调者的判断，不是线程倒退）。没有编码任何普适阈值，辅助脚本也不估计完成时间：事实就是序列、档位与已耗时间，解读归你。

**Status table** (`overview --table`; `overview`'s `table`; the tail of
`context`'s `text` every wake — rendered by the helper, never hand-typed):
`<slug>: N threads · L landed[ · A accepted]` — `landed` counts threads with
a `merged` PR event, `accepted` the `done` ones without (shown only when
any; "2 of 3 landed" echoed to a user who said do not
merge); under each workstream title (`other:` for
the rest; no headers when no thread has one) one row per thread — the
name-first cell `<name> (<engine>) [<id>] · PR <n>` (the session name only
inside an attach command), the bar `██████░░░░ 60% <basis>` of the shown
value, the ETA with its basis, the group and `quiet <time>` — then at most
two footer lines (`landed L of N[ · accepted A] · inbox K pending · wake <tier>`; `flags:
…` when any) and one `Needs you: <name> (<engine>) [<id>] — <question>` line
per open question, a dialog's with its attach command. `overview` also
answers `needs_you` (the open entries), as `context` does.

**状态表**（`overview --table`；`overview` 的 `table`；每次唤醒时 `context` 的 `text` 末尾——由辅助脚本渲染，绝不手打）：`<slug>: N threads · L landed[ · A accepted]`——`landed` 计有 `merged` PR 事件的线程，`accepted` 计没有该事件的 `done` 线程（仅在非零时显示；对说过不要合并的用户会回显"2 of 3 landed"）；在每个工作线标题之下（其余的归 `other:`；没有线程带工作线时则无表头）每线程一行——名字在前的单元格 `<name> (<engine>) [<id>] · PR <n>`（会话名只出现在 attach 命令里）、显示值的进度条 `██████░░░░ 60% <basis>`、带 basis 的 ETA、分组与 `quiet <时间>`——然后至多两行页脚（`landed L of N[ · accepted A] · inbox K pending · wake <tier>`；有旗标时 `flags: …`）以及每个未决问题一行 `Needs you: <name> (<engine>) [<id>] — <question>`，对话框的附其 attach 命令。`overview` 也应答 `needs_you`（未决条目），与 `context` 相同。

## Inbox events and idempotency keys / 收件箱事件与幂等键

An event is `{schema: "agents-event/v1", key, kind, at, thread, text, data}`.
`inbox put` files it once per `key`; a second `put` with a key already in
`inbox/new/` or `inbox/done/` is `duplicate` (exit 0, `deduplicated: true`,
nothing written). Key shapes, one per source:

一个事件是 `{schema: "agents-event/v1", key, kind, at, thread, text, data}`。`inbox put` 按 `key` 只归档一次；第二次 `put` 带的键已存在于 `inbox/new/` 或 `inbox/done/` 时是 `duplicate`（exit 0，`deduplicated: true`，不写任何东西）。键的形状，每个来源一种：

| source | key |
| --- | --- |
| a thread report | `report:<thread>:<sha256 of report body, 12 hex>` |
| a PR event | `pr:<head sha>:<event>` (`checks_failed`, `review`, `enqueued`, `merged`, `conflict`, …) |
| a thread status change (`tick`) | `thread:<thread>:<status>:<opened_at>` |
| a thread that turned waiting-on-you (`tick`) | `thread:<thread>:waiting-on-you:<stamp>` — text carries its attach command; once per transition |
| an outside subscription | `sub:<source>:<seq>` |
| the user or the coordinator by hand | `user:<any text>` |

| 来源 | 键 |
| --- | --- |
| 一份线程报告 | `report:<thread>:<sha256 of report body, 12 hex>` |
| 一个 PR 事件 | `pr:<head sha>:<event>`（`checks_failed`、`review`、`enqueued`、`merged`、`conflict`，…） |
| 一次线程状态变化（`tick`） | `thread:<thread>:<status>:<opened_at>` |
| 一个转为 waiting-on-you 的线程（`tick`） | `thread:<thread>:waiting-on-you:<stamp>`——text 携带其 attach 命令；每次转变一条 |
| 一个外部订阅 | `sub:<source>:<seq>` |
| 用户或协调者手工 | `user:<any text>` |

On the inbox path (ADR 41038 D1; `wake_path: inbox`) a `report` event and a `pr …:merged` event are also sent to the coordinator's session as an `agents-message/v1` message — the fast path beside the Monitor's WAKE line, which still names the same event — and a copy filed with `inbox put --message` uses the message's own `key:`: a copy delivered twice, or one the Monitor already named, files once.

在收件箱路径上（ADR 41038 D1；`wake_path: inbox`），一个 `report` 事件和一个 `pr …:merged` 事件还会作为 `agents-message/v1` 消息发送到协调者的会话——这是 Monitor 的 WAKE 行之外的快速路径，WAKE 行仍会点名同一事件——而用 `inbox put --message` 归档的副本使用消息自己的 `key:`：一份被投递两次的副本、或一份 Monitor 已点名过的副本，只归档一次。

## `doctor` / `doctor`（体检）

```
doctor [<slug>]
```

`healthy` (0) or the first thing missing (`needs_user_action` 5,
`no_host_manager` 4, `sandbox_blocked` 5). `checks` rows: `python`,
`projects_home` (writable), `host_manager` (found, its own `doctor`
outcome), `tmux` (only when host-manager's `list` fails after a healthy
doctor: `fail` with `sandboxed: true` when the socket answers `Operation
not permitted` — a sandboxed shell, which cannot open, read or stop
sessions; `warn` with the reason otherwise), `fleet_manager` (found or
absent — absent is `warn`, remote threads unavailable), `git`, `gh` (absent
is `warn`: the follow thread needs it), `wake` (`candidates`: which of
`monitor`, `systemd-run`, `launchctl`, `crontab` exist here — facts, the
coordinator picks and arms). `capabilities` carries `unattended_flag` when
host-manager's `open --help` advertises it (one probe). `sandbox_blocked`
(5): `next` names the escalated shell (`sandbox_permissions:
require_escalated`) or a coordinator session without the sandbox; `init`,
`propose` and `context` keep working from the sandboxed shell. Under
`sandbox_blocked`, run `init` and every later verb in the escalated shell
(the recorded coordinator must be the shell that runs `go`): this shell
cannot reach the tmux socket. With a slug: the project's folder, its
coordinator record and wake arm (`wake_path: inbox` on the inbox path only).

`healthy`（0）或第一个缺失项（`needs_user_action` 5、`no_host_manager` 4、`sandbox_blocked` 5）。`checks` 行：`python`、`projects_home`（可写）、`host_manager`（已找到，及其自己的 `doctor` 结果）、`tmux`（仅在 host-manager 的 `list` 在一次健康的 doctor 之后仍失败时出现：套接字应答 `Operation not permitted` 时为 `fail` 并带 `sandboxed: true`——一个沙盒 shell，无法开启、读取或停止会话；否则 `warn` 并附原因）、`fleet_manager`（已找到或缺失——缺失为 `warn`，远程线程不可用）、`git`、`gh`（缺失为 `warn`：follow 线程需要它）、`wake`（`candidates`：`monitor`、`systemd-run`、`launchctl`、`crontab` 哪些在此存在——只是事实，由协调者挑选并布防）。host-manager 的 `open --help` 宣告它时，`capabilities` 携带 `unattended_flag`（一次探测）。`sandbox_blocked`（5）：`next` 点名提权 shell（`sandbox_permissions: require_escalated`）或一个没有沙盒的协调者会话；`init`、`propose` 和 `context` 在沙盒 shell 里继续可用。在 `sandbox_blocked` 之下，`init` 和此后每个动词都要在提权 shell 里运行（被记录的协调者必须是运行 `go` 的那个 shell）：这个 shell 够不到 tmux 套接字。带 slug 时：给出项目的文件夹、其协调者记录与唤醒布防（仅收件箱路径上为 `wake_path: inbox`）。

## `init` / 初始化

```
init - [--done-means TEXT] [--slug S] [--repo DIR …] [--max-parallel N]
       [--goal "<the task in the threads' words>"]
       [--unattended|--attended] [--start-threads propose|auto] [--engine muse|claude|codex] [--detach] [--asked-by WHO] <<'EOF'
<the user's message, whole>
EOF
```

`init -` reads the task from stdin, the way `propose --threads-json -`
reads its JSON: a quoted heredoc keeps the user's backticks and `$(…)` as
text; never inside double quotes: a backtick or `$(…)` in the user's words
runs as a command in their clone; never edit the message to make it
shell-safe. The user's words are the whole message — every numbered
requirement, quoted string and sentence that defines a PR, merged, a test
command or an origin, not its title: the follow brief quotes them from `##
Goal`, a paraphrase loses them. `init "<message>"` in double quotes let the shell run a backticked
`python -m pytest -q` inside the user's clone and store its output as the
goal, and the other coordinator stripped all 70 backticks to avoid that. Positional words are still accepted; a message among
them that carries a backtick or `$(` answers `warnings:
[{kind: `task_on_the_command_line`, text, next}]` with `next` = `pass the
message on stdin (init -)`, the line's own `next` opening with the warning —
recorded as received, never refused. `init -` with empty stdin is `usage`
(2), nothing created.

`init -` 从 stdin 读取任务，与 `propose --threads-json -` 读其 JSON 的方式相同：带引号的 heredoc 把用户的反引号和 `$(…)` 保持为文本；绝不要放进双引号里：用户话语中的反引号或 `$(…)` 会在其克隆里作为命令运行；绝不要为了让消息 shell 安全而编辑它。用户的话就是整条消息——每一条编号的要求、每一个引号字符串、每一句定义 PR、合并、测试命令或来源（而非其标题）的话：follow 简报会从 `## Goal` 引用它们，一改写就丢。用双引号的 `init "<message>"` 曾让 shell 在用户克隆里运行了一段反引号包裹的 `python -m pytest -q` 并把其输出存成目标，于是另一位协调者删掉了全部 70 个反引号来避免此事。位置参数仍然接受；其中带有反引号或 `$(` 的消息会应答 `warnings: [{kind: `task_on_the_command_line`, text, next}]`，`next` = `pass the message on stdin (init -)`，该行自己的 `next` 以这条警告开头——按收到的样子记录，绝不拒绝。`init -` 遇到空 stdin 是 `usage`（2），什么都不创建。

【评论】这段是在提示词注入之外再防一层 shell 注入：用户原话里的反引号与 `$()` 在双引号上下文中会被当命令执行，文档以事故复盘的口吻强制"stdin + 带引号 heredoc"这一条通道。

Plain `init` makes the folder and records you, this session, as
coordinator; its answer carries `doctor`'s checks. Keep `--slug` short — 24 characters at most: the slug prefixes every
session name (`<slug>-<id>`), and a 63-character name wraps at 80 columns
and eats the Monitor row. Creates the folder: `PROJECT.md` with `## Goal` from `--goal` — the task in
the threads' own words, yours to write: `## Goal` is what every thread
reads, and a thread that reads the sentences addressed to you ("propose the
threads and wait for my go", "you coordinate", "do not do the work
yourself") becomes a coordinator, so leave them
out. Without the flag the message stands as the goal, unchanged. The
message as typed is kept whole under `## Request` either way —
`## Done means` from `--done-means` (the evidence that ends the project;
empty without the flag — the coordinator then writes it before any
thread), `## Repositories` from `--repo` (default: the repository root walked
up from the current directory), `## Settings` from the flags; empty
`MEMORY.md`, `TASKS.md`, `threads/`, `library/`, `inbox/`. No other heading
is written. A second clone is a second `--repo`: the helper counts two
checkouts as one repository only when they share a `.git` common directory
(a worktree does; another clone does not). Records this session as the coordinator
(`state.json.coordinator`: `kind: session` with `MUSE_LANE_BACKEND` /
`MUSE_LANE_REF` when present, else `kind: process` with `user`, `host`,
`pid` — the coordinating process, the nearest ancestor of the helper that
is neither a shell nor a python interpreter (a `python3` that is a launcher
forking the real interpreter is the helper's own per-call parent, never the
coordinator) — its `command` and `started` stamp, which `resume` and
`context` check on the same host through `/proc` on Linux, `ps` elsewhere). A caller that runs `init`
for a coordinator session it opens afterwards says so with
`MUSE_AGENTS_ROLE=launcher`: no identity in its own process tree is that
session's — the walk climbed to the launcher's own TUI and the opened
session's `resume` met a live coordinator — so
nothing is recorded (`coordinator: null`, a `progress`
line says so, `next` = `resume <slug>` from that session) and the first
`resume` binds. `--slug` defaults
to a slug of the task's first words; a taken slug is `slug_taken` (3) with
`next` = `resume <slug>`. A task without `--slug` that starts with
`resume`, `continue`, `reopen` or `pick` (or is one word) and names an
existing project is `resume_instead` (3, nothing created, `next` =
`resume <that slug>`): a fresh session asked to pick a project up never
makes a stray one. The coordinator is recorded as the session
host-manager opened it in (`MUSE_LANE_BACKEND`/`MUSE_LANE_REF`); else, inside
tmux, as the TUI's own pane (`$TMUX`, `$TMUX_PANE`) — from its sandboxed tool
shell and its escalated shell alike, which see different processes but one
pane (a default-mode coordinator's auto-approved `init`
recorded the pane, its approved `go` came as a process, and the first `go`
was refused) — probed through host-manager `list` on that tmux server (a live
row naming the pane); outside the sandbox the process (pid, start stamp)
rides on the pane record, and a project that recorded a process accepts the
same process seen from its pane; else as a process (user, host, pid, start
stamp) probed through `/proc` on Linux, `ps` elsewhere; inside a sandboxed tool shell (a PID namespace,
where the coordinating process is `pid 2 sh` and no host `ps` knows it)
with no pane that pid is never recorded: the coordinator is `opaque` —
nobody can prove they are it, so `resume` from any session answers
`coordinator_unknown` until the human's words take over; a `progress` line
says which. `--detach` opens a coordinator session through
host-manager `open` with a starter that loads this skill and runs `resume
<slug>`; the receipt carries that session's identity. `--detach` requires
`--unattended` (`usage`, 2, nothing created, without it — even when this
session itself runs approvals-off: nobody sits in a detached coordinator,
so that word stays the user's own, never inherited): nobody answers a
detached session's prompts, so it opens with `--unattended`
(`coordinator.posture_applied: unattended`; `provider_default` with a
`progress` line under a host-manager that predates D16). The detached
coordinator gets the same settings as the session that opened it (owner
ruling 2026-09-20; ADR 38715 D15/D16): `--env MUSE_EXPERIMENTAL_AGENTS=on`
and the launching session's own `--model`, `--reasoning-effort`,
`--provider`, `--preset`, `--base-url` and tool-call switches as
`--engine-arg=--flag=value` tokens, read from that session's command line
(the coordinating process's argv; never its posture flags, workspace,
worktree, resume or prompt as engine args — the approvals-off flag there is
read for the threads' inherited posture, Amendment 6); where the command line cannot be read (a
sandboxed tool shell, a launcher that is not a Muse session) nothing is
guessed. `initialized` (0): `slug`, `path`, `coordinator`, `settings`,
`created: true`, and with `--detach` `launch_settings` (`env`,
`engine_args`, `engine_args_source`: `launcher argv` or the reason none
could be read; `binary`, `binary_source` and, when PATH `muse` stood in,
`binary_fallback`: the detached coordinator runs this session's own
binary — `MUSE_BIN`, else this session's command resolved — never a
stale `muse` on PATH without saying so). The line also carries what the first turn used to fetch
in four more calls (`doctor`, `context`, a
`PROJECT.md` read and edit around every `init`): doctor's `checks` and
`health` (`{"outcome": "healthy"}`, or doctor's refusal as `outcome`,
`error`, `next` — `no_host_manager`, `sandbox_blocked`, host-manager's own
word — reported, never raised: the folder is made either way), and the
picture `context` would return for the new project (`project`, `memory`,
`tasks`, `threads`, `inbox`, `coordinator`, `wake`, `host_manager`, `text`;
`changed` and `since_last_context_s` are `null` — `init` moves no context
cursor, so the go turn's `context` is still a first call). `next` is the
proposal line (`propose <slug> --threads-json -`) when `--done-means` was
given, else `write `## Done means` in PROJECT.md …, then propose …`; with
`--detach` or a launcher, the resume line as before. A goal turn is
`init --done-means "…"` then `propose`: no separate `doctor` or `context`.

不带修饰的 `init` 创建文件夹，并把"你"——本会话——记录为协调者；其应答携带 `doctor` 的检查。`--slug` 保持简短——至多 24 个字符：slug 是每个会话名（`<slug>-<id>`）的前缀，63 个字符的名字会在 80 列处折行、吃掉 Monitor 行。创建文件夹：`PROJECT.md`，其 `## Goal` 来自 `--goal`——用线程自己的口吻写的任务，由你来写：每个线程读到的都是 `## Goal`，而一个读到那些对你说话的句子（"propose the threads and wait for my go"、"you coordinate"、"do not do the work yourself"）的线程会变成协调者，所以要把它们去掉。不给该旗标时，消息原样作为目标。无论哪种方式，按输入原样的消息都完整保存在 `## Request` 之下——`## Done means` 来自 `--done-means`（结束项目的证据；不给旗标则为空——协调者随后会在任何线程之前写它）、`## Repositories` 来自 `--repo`（默认：从当前目录向上走到的仓库根）、`## Settings` 来自各旗标；另建空的 `MEMORY.md`、`TASKS.md`、`threads/`、`library/`、`inbox/`。不写其他标题。第二个克隆是第二个 `--repo`：只有当两个检出共享一个 `.git` 公共目录时，辅助脚本才把它们算作一个仓库（worktree 是；另一个克隆不是）。把本会话记录为协调者（`state.json.coordinator`：存在时为 `kind: session` 带 `MUSE_LANE_BACKEND` / `MUSE_LANE_REF`，否则为 `kind: process` 带 `user`、`host`、`pid`——协调进程，即辅助脚本向上找到的既非 shell 也非 python 解释器的最近祖先（一个 fork 出真正解释器的 `python3` 启动器只是辅助脚本每次调用的父进程，绝不是协调者）——及其 `command` 与 `started` 时间戳，`resume` 和 `context` 在同一主机上于 Linux 经 `/proc`、其他平台经 `ps` 加以核对）。为一个随后才开启的协调者会话运行 `init` 的调用方用 `MUSE_AGENTS_ROLE=launcher` 说明这一点：它自己进程树里没有任何身份属于那个会话——向上游走到了启动器自己的 TUI，而新开会话的 `resume` 遇到一个活协调者——所以什么都不记录（`coordinator: null`，一条 `progress` 行说明这一点，`next` = 从那个会话执行 `resume <slug>`），由第一次 `resume` 完成绑定。`--slug` 默认取任务开头几个词的 slug；slug 已被占用是 `slug_taken`（3），`next` = `resume <slug>`。不带 `--slug` 且以 `resume`、`continue`、`reopen` 或 `pick` 开头（或是单个词）并指名了一个已有项目的任务是 `resume_instead`（3，什么都不创建，`next` = `resume <那个 slug>`）：一个被要求接手项目的全新会话绝不会顺手造出一个杂散项目。协调者被记录为 host-manager 开启它时所用的那个会话（`MUSE_LANE_BACKEND`/`MUSE_LANE_REF`）；否则，在 tmux 内，记为 TUI 自己的 pane（`$TMUX`、`$TMUX_PANE`）——从其沙盒工具 shell 与其提权 shell 记录的结果一致，两者看到的进程不同但 pane 是同一个（一个 default 模式协调者自动批准的 `init` 记录了 pane，其批准的 `go` 却以进程身份到来，于是第一次 `go` 被拒）——通过 host-manager 在该 tmux 服务器上的 `list` 探测（一行点名该 pane 的活行）；沙盒之外则进程（pid、启动时间戳）附在 pane 记录上，且记录了进程的项目也接受从其 pane 看到的同一进程；否则记为一个进程（user、host、pid、启动时间戳），Linux 经 `/proc`、其他平台经 `ps` 探测；在沙盒工具 shell 内（一个 PID 命名空间，协调进程是 `pid 2 sh`，没有任何主机 `ps` 认识它）且没有 pane 时，那个 pid 绝不被记录：协调者是 `opaque`——没有人能证明自己是它，因此任何会话的 `resume` 都应答 `coordinator_unknown`，直到用户的原话接管；一条 `progress` 行说明是哪种。`--detach` 通过 host-manager `open` 开启一个协调者会话，starter 加载这个 skill 并运行 `resume <slug>`；回执携带那个会话的身份。`--detach` 要求 `--unattended`（不给则是 `usage`、2、什么都不创建——即使本会话自身就是审批关闭运行：没有人坐在一个分离的协调者里，所以这个词必须始终是用户自己的原话，绝不被继承）：没有人应答分离会话的提示，因此它以 `--unattended` 开启（`coordinator.posture_applied: unattended`；在早于 D16 的 host-manager 下为 `provider_default` 并带一条 `progress` 行）。分离的协调者拿到与开启它的会话相同的设置（业主裁决 2026-09-20；ADR 38715 D15/D16）：`--env MUSE_EXPERIMENTAL_AGENTS=on`，以及发起会话自己的 `--model`、`--reasoning-effort`、`--provider`、`--preset`、`--base-url` 与工具调用开关，作为 `--engine-arg=--flag=value` 记号，从该会话的命令行读取（协调进程的 argv；绝不把它的姿态旗标、工作区、worktree、resume 或提示当作引擎参数——其中的审批关闭旗标被读出来用于线程的继承姿态，修正案 6）；命令行读不到时（沙盒工具 shell、一个不是 Muse 会话的启动器）什么都不猜。`initialized`（0）：`slug`、`path`、`coordinator`、`settings`、`created: true`，带 `--detach` 时还有 `launch_settings`（`env`、`engine_args`、`engine_args_source`：`launcher argv` 或读不到的原因；`binary`、`binary_source`，以及当 PATH 上的 `muse` 顶替过时还有 `binary_fallback`：分离的协调者运行本会话自己的二进制——`MUSE_BIN`，否则解析本会话的命令——绝不悄悄用 PATH 上的陈旧 `muse`）。该行还携带第一轮原本要用四次额外调用才能取到的东西（`doctor`、`context`、围绕每次 `init` 的一次 `PROJECT.md` 读与编辑）：doctor 的 `checks` 与 `health`（`{"outcome": "healthy"}`，或以 `outcome`、`error`、`next` 呈现的 doctor 拒绝——`no_host_manager`、`sandbox_blocked`、host-manager 自己的词——只报告、不抛出：无论如何文件夹都会创建），以及 `context` 会为新项目返回的图景（`project`、`memory`、`tasks`、`threads`、`inbox`、`coordinator`、`wake`、`host_manager`、`text`；`changed` 与 `since_last_context_s` 为 `null`——`init` 不移动上下文游标，因此 go 轮的 `context` 仍是首次调用）。`next` 在给了 `--done-means` 时是提议行（`propose <slug> --threads-json -`），否则是 `write `## Done means` in PROJECT.md …, then propose …`；带 `--detach` 或启动器时，照旧是 resume 行。一个目标轮是 `init --done-means "…"` 然后 `propose`：不再单独跑 `doctor` 或 `context`。

【评论】对"目标文本里出现对你说话的句子会让线程变成协调者"的告警，是一处针对角色混淆（role confusion）的防提示词注入设计：写入 `## Goal` 的内容会流向每个 worker 线程，因此指令性语句必须剥离。

**Wake path (ADR 41038 D5).** `init` reads `TBH_AGENTS_SESSION_PROTOCOL` once
(1/on/true/yes, any case); no later verb reads the flag, so a project keeps
its path until archive. Off, or unset: today's behaviour byte for byte — no
key is written, and a missing `wake_path` reads as the Monitor path. On: the
helper resolves this session in the local Muse session list (`muse
session-message list --json`; `MUSE_AGENTS_SESSION_LIST` overrides the
command): exactly one row under this process's workspace label (or the
lane's own `session_name`) is the report target, recorded as
`inbox_target` (with `inbox_label`), `wake_path` is `inbox`, and a
`progress` line says so; the wake loop is written and the wake arm stays
today's — the Monitor wake is the floor on both paths until #41228 (round 21
B1: a worktree thread's message parks behind the coordinator's admission
card and dies with the thread). No such row — a closed list
(`external_agent_ingress_closed`), none or two under the label — opens the
project on the monitor path with a warning `inbox_wake_unavailable`
(`warnings[]`, its `text` naming the reason, its `next` the Monitor arm). On
the inbox path the `init`, `context` and `resume` lines carry `wake_path:
inbox` (`resume` also `inbox_target` and `resubscribed`); on the monitor
path they are today's lines and today's `state.json`, key for key
(FR-41038-5).

**唤醒路径（ADR 41038 D5）。** `init` 只读一次 `TBH_AGENTS_SESSION_PROTOCOL`（1/on/true/yes，大小写不限）；后续动词都不再读这个旗标，因此项目保持其路径直到归档。关闭或未设置：与今天的行为逐字节相同——不写任何键，缺失的 `wake_path` 按 Monitor 路径解读。开启：辅助脚本在本地 Muse 会话列表（`muse session-message list --json`；`MUSE_AGENTS_SESSION_LIST` 可覆盖该命令）中解析本会话：在本进程的工作区标签（或泳道自己的 `session_name`）之下恰好一行者即为报告目标，记录为 `inbox_target`（连同 `inbox_label`），`wake_path` 为 `inbox`，并由一条 `progress` 行说明；唤醒循环照写，唤醒布防也保持今天的——直到 #41228，Monitor 唤醒始终是两条路径的下限（第 21 轮 B1：一个 worktree 线程的消息停在协调者的准入卡片后面，随线程一起消亡）。没有这样一行——列表已关闭（`external_agent_ingress_closed`）、该标签之下一行都没有或有两行——则以一条 `inbox_wake_unavailable` 警告（`warnings[]`，其 `text` 写明原因，其 `next` 是 Monitor 布防）让项目走上 monitor 路径。在收件箱路径上，`init`、`context` 和 `resume` 行携带 `wake_path: inbox`（`resume` 还携带 `inbox_target` 与 `resubscribed`）；在 monitor 路径上它们就是今天的行和今天的 `state.json`，一键不差（FR-41038-5）。

## `context` / 上下文

```
context <slug>
```

The whole picture in one call, for one coordinator turn. It changes nothing but its own `changed` cursor in `state.json`
and, like `tick`, types a PR queued on the follow record once that thread's composer is free (`follow_delivered`, the
URLs typed this call; see `follow`)
(`last_context` when the caller is the recorded coordinator, or when that coordinator has no stable identity — `opaque`,
nobody can be told from it; a session that is not it reads the same picture on a cursor of its own under
`context_cursors.<kind|identity keys>` — the keys `resume`/`not_coordinator` compare; opaque callers, which have none, get one
row per user and host — so
a second TUI's `resume` or a shell's `context` never consumes the coordinator's deltas: each caller sees a change once; at
most eight stranger rows are kept, the oldest look dropped first, and an evicted caller's next call reads as a first call):

一次调用取回全貌，供协调者的一轮使用。它只改动 `state.json` 中自己的 `changed` 游标，此外什么也不改；并且像 `tick` 一样，一旦该线程的输入框空闲，就把一个已排队的 PR 键入 follow 记录（`follow_delivered`，本次调用键入的 URL；见 `follow`）（当调用方就是被记录的协调者、或该协调者没有稳定身份——`opaque`，无法据以区分任何人——时为 `last_context`；不是它的会话在 `context_cursors.<kind|identity keys>` 之下用自己的游标读取同一图景——`resume`/`not_coordinator` 比较这些键；没有身份的 opaque 调用方按每个 user 和 host 得到一行——于是第二个 TUI 的 `resume` 或某个 shell 的 `context` 绝不会消费协调者的增量：每个调用方对每个变化只看一次；至多保留八行陌生者行，最老的查看先丢弃，被逐出的调用方下一次调用按首次调用解读）：

- `project`: `slug`, `path`, `goal`, `done_means` (empty string when the
  coordinator has not written it — write it first), `scope`, `repositories`,
  `standing_instructions`, `settings`;
  `project`：`slug`、`path`、`goal`、`done_means`（协调者尚未写时为空字符串——先写它）、`scope`、`repositories`、`standing_instructions`、`settings`；
- `memory`: the `MEMORY.md` headings with their dates (the index, not the
  text);
  `memory`：`MEMORY.md` 的标题及其日期（只是索引，不是正文）；
- `tasks`: the `TASKS.md` lines with `done: true|false`;
  `tasks`：`TASKS.md` 的行，带 `done: true|false`；
- `threads`: one row per thread: the record fields (`attach` included) plus `live` (host-manager
  or fleet-manager `status`, `null` when it could not answer), `agent_status`
  (Herdr's, or the tmux screen verdict), `group`, `evidence` (`screen` or `liveness` for a tmux thread), `screen` (what the screen showed), and the ledger facts — `artifact_rung`, `self_reported`, `calibrated`, `progress` (the one shown value), `elapsed_s`, `quiet_for_s`, `eta`, `flags`, `pending_question` (§ The tracking ledger); a
  provider that cannot answer narrows `coverage`, lists the thread under
  `unknowns` instead of grouping it and puts its words in `unknown_reason`;
  `threads`：每线程一行：记录字段（含 `attach`）外加 `live`（host-manager 或 fleet-manager 的 `status`，答不上来时为 `null`）、`agent_status`（Herdr 的，或 tmux 屏幕判定）、`group`、`evidence`（tmux 线程为 `screen` 或 `liveness`）、`screen`（屏幕显示了什么），以及台账事实——`artifact_rung`、`self_reported`、`calibrated`、`progress`（所显示的那一个值）、`elapsed_s`、`quiet_for_s`、`eta`、`flags`、`pending_question`（§ The tracking ledger）；答不上来的提供者会缩小 `coverage`、把该线程列入 `unknowns` 而不分组，并把它的原话放进 `unknown_reason`；
- `inbox`: `pending` (the events in `inbox/new/`, oldest first) and
  `pending_count`;
  `inbox`：`pending`（`inbox/new/` 中的事件，最旧在前）与 `pending_count`；
- `worker_notes` and `repainted` (FR-43932-6): the herdr worker gate's
  pending notes for this project — a worker under a live coordinator
  reports `unknown` instead of `idle`/`blocked`, and each gated report
  leaves one note (`pane`, `server`, `machine`, `slug`, `kind`, `at`,
  `seq`) in its project dir (the coordinator's own, or the thread
  machine's for a split-machine thread, read through the copy channel);
  work a note like any other thread event, and relay a note's question
  with `relay`, never an answer of your own. When the recorded
  coordinator is dead, a note older than 45 minutes is repainted here
  (a fresh `idle` for its pane, `seq` one more than the note's) and
  listed under `repainted` (`delivered: false` names a machine whose
  socket did not answer — the note stays pending);
  `worker_notes` 与 `repainted`（FR-43932-6）：herdr worker 门为该项目留下的待处理备注——活协调者之下的 worker 报告 `unknown` 而不是 `idle`/`blocked`，每份被拦的报告在其项目目录留下一条备注（`pane`、`server`、`machine`、`slug`、`kind`、`at`、`seq`；协调者自己的目录，分机线程则经复制通道读线程机器上的）；像对待其他线程事件一样处理一条备注，并用 `relay` 转发备注里的问题，绝不用你自己编的回答。当被记录的协调者已死时，超过 45 分钟的备注会在此被重绘（为其 pane 生成一条新的 `idle`，`seq` 比该备注大一号）并列在 `repainted` 之下（`delivered: false` 表示某台机器的套接字没有应答——该备注仍待处理）；
- `coordinator`: the recorded coordinator and whether it is this process;
  `coordinator`：被记录的协调者，以及它是否是本进程；
- `wake`: the recorded arm (`tier`, `command`, `armed_at`, `armed_by`,
  `means` — what that tier does for the coordinator session, repeated in
  `text` —, `env` and `tick_command`) or `null` — a `null` arm means nothing
  will wake this project; the coordinator arms one (`tick --arm`) or says so;
  `wake`：被记录的布防（`tier`、`command`、`armed_at`、`armed_by`、`means`——该层级为协调者会话做什么，在 `text` 中重复——、`env` 与 `tick_command`）或 `null`——`null` 布防意味着没有什么会唤醒这个项目；协调者要么布防一个（`tick --arm`），要么明说没有；
- `changed`: what moved since this caller's last `context` call (thread
  groups, new events, memory entries, tasks); the first call has nothing to
  compare against and says so;
  `changed`：自该调用方上次 `context` 调用以来移动了什么（线程分组、新事件、记忆条目、任务）；首次调用没有可比对象，并如实说明；
- `plan_lines` and `channel_line`: the ☐/✅ list to post under the wake
  line, as printed (§ `go`);
  `plan_lines` 与 `channel_line`：要贴在唤醒行之下的 ☐/✅ 列表，按打印原样（§ `go`）；
- `since_last_context_s`: whole seconds since this caller's previous
  `context` call (`null` on the first); when nothing moved and it is under a minute, `text`
  ends `nothing moved since <n> s ago` — a fact, not a refusal;
  `since_last_context_s`：距该调用方上次 `context` 调用的整秒数（首次为 `null`）；当没有东西移动且不足一分钟时，`text` 以 `nothing moved since <n> s ago` 收尾——这是一个事实，不是拒答；
- `unchanged_for_s`: whole seconds since this caller's picture last differed
  from the look before it (`null` on the first call, `0` on a call that
  reports a change): how long nothing has moved, as a number (coordinators
  slept in-turn 50-75 % of their active time);
  `unchanged_for_s`：距该调用方的图景上次与上一次查看不同的整秒数（首次调用为 `null`，报告了变化的调用为 `0`）：没有东西移动了多久，作为一个数字（协调者在轮内睡眠占其活跃时间的 50-75%）；
- `host_manager`: the exact host-manager command prefix this helper uses for
  its own calls (`--tmux` included when `MUSE_AGENTS_TMUX` is set), so your
  own `send`/`read`/`resources` calls reach the same server; `null` when no
  host-manager is beside the skill;
  `host_manager`：本辅助脚本自己调用所用的 host-manager 命令前缀，一字不差（设置了 `MUSE_AGENTS_TMUX` 时含 `--tmux`），使你自己的 `send`/`read`/`resources` 调用到达同一服务器；skill 旁没有 host-manager 时为 `null`；
- `text`: the same picture in a few lines, safe to read to the user — each
  thread row `  - <name> [<id>] (<machine>): <status line, or the status
  word>`, the name first and the id in brackets once; a
  thread without a group is a line under `unknown` with its machine, the
  provider's reason and its last known status — a machine that is down reads as a machine down, never as a missing row, then the status table (below);
  `text`：同一图景的几行文本，可以安全读给用户——每线程行 `  - <name> [<id>] (<machine>): <status line, or the status word>`，名字在前、id 只在方括号中出现一次；没有分组的线程是 `unknown` 之下的一行，带其机器、提供者的理由与最后已知状态——一台宕机的机器读作机器宕机，绝不是一行缺失，然后是状态表（见下）；
- `needs_you`: the open `pending_question` entries, oldest first (a `BLOCKED(HUMAN):` report no tick has ledgered yet included);
  `needs_you`：未决的 `pending_question` 条目，最旧在前（尚未被任何 tick 台账化的 `BLOCKED(HUMAN):` 报告也算在内）；
- `next`: the inbox moves in order (never `inbox drain`: this call read
  them); a `ready-for-review`
  or `waiting-on-you` thread whose report event is already read gets the
  same moves (`follow`, the question, `ack`) from its record, whatever
  `changed` says — never `nothing moved; end the turn` while a report is
  unread.
  `next`：收件箱按顺序处理（绝不是 `inbox drain`：本次调用已经读过它们）；一条报告事件已被读过的 `ready-for-review` 或 `waiting-on-you` 线程，从其记录得到同样的步骤（`follow`、提问、`ack`），无论 `changed` 说什么——只要还有报告未读，就绝不是 `nothing moved; end the turn`。
- `tracking_error`: only when `tracking.json` cannot be read — the path and the cure; the table is that line then.
  `tracking_error`：仅当 `tracking.json` 读不了时出现——路径与补救；那时状态表就是那一行。

Every call answers the whole picture. A look with nothing moved carries the
facts that say so — `since_last_context_s` (since this caller's previous
`context`), `unchanged_for_s`, `context_call` (this cursor's looks since
anything moved) — and, from the second, the `text` line
`nothing moved since <n> s ago — look <N> since anything did; <what brings
you back>` (the Monitor with its arm time, the scheduler or passive words,
or the arm hint when no wake is recorded). Whether that is a second look in
one turn is yours to read: the helper keeps no clock-based same-turn guess
(text and a hint, no guard).

每次调用都应答全貌。没有东西移动的一次查看携带说明这一点的事实——`since_last_context_s`（自该调用方上次 `context` 起）、`unchanged_for_s`、`context_call`（该游标自任何东西移动以来的查看次数）——并且从第二次起还有 `text` 行 `nothing moved since <n> s ago — look <N> since anything did; <what brings you back>`（Monitor 带其布防时间、scheduler 或 passive 的措辞、没有记录布防时则是布防提示）。这算不算同一轮里的第二次查看由你解读：辅助脚本不保留任何基于时钟的同轮猜测（只有文本和提示，没有守卫）。

`no_such_project` (3) names the folder that is missing; `next` is `init`.
`"archived"` (3) when the slug was archived: `"archive_path"`, `"archived_at"`,
`next` reads the archived `PROJECT.md` (every project verb answers this
way for an archived slug).

`no_such_project`（3）点名缺失的文件夹；`next` 是 `init`。slug 已归档时为 `"archived"`（3）：`"archive_path"`、`"archived_at"`，`next` 会读取归档的 `PROJECT.md`（对已归档的 slug，每个项目动词都这样应答）。

A Monitor nobody installed (`tick --arm monitor`
recorded, no Monitor tool call, two reports unread ten minutes): when the
record says `tier: monitor`, `library/wake.lock/pid` names no live process
and the arm is older than 15 s (the loop writes its pid in its first
second), `text` leads with `wake: monitor armed on record, but no loop is
running` and `next` opens with `call monitor(<the recorded command>) now`
(the recorded line when it is a `monitor(...)` line, else the ready line
wrapping it). A fact, never a disarm: the record stands.

一个没人安装的 Monitor（记录着 `tick --arm monitor`、没有 Monitor 工具调用、两份报告十分钟未读）：当记录写着 `tier: monitor`、`library/wake.lock/pid` 点不到活进程、且布防已超过 15 秒（循环在第一秒写入自己的 pid）时，`text` 以 `wake: monitor armed on record, but no loop is running` 开头，`next` 以 `call monitor(<被记录的命令>) now` 开头（记录本身是 `monitor(...)` 行时用它，否则用包着它的就绪行）。这是一个事实，绝不是解除布防：记录照旧成立。

## `propose` / 提议

```
propose <slug> (--threads-json PATH|-)
```

Records the coordinator's proposal — threads the user has not yet approved.
The JSON is `{"threads": [{id, name, brief, cwd?, repo?, worktree?, machine?,
engine?, engine_args?, model?, effort?, unattended?, kind?, owns?, test_command?, workstream?}]}`: `id` is `[a-z0-9-]{1,32}` and
unique in the project (a taken id is `thread_exists`, 3); `brief` is the one page the thread is told; `workstream` (a title, or `{title, source_ref}`) groups the thread in the status table and nothing else — never a session name or a label — and `propose` writes the assignment to `tracking.json` (anything but a title is `usage`, 2); `machine` defaults to `local`; `worktree` is a
**branch name**: `go` gives a local thread its own checkout of that branch
(created from `HEAD` when the branch is new) beside the repository and
records the path in `worktree_path` (`null` = work in `cwd`; two threads
that write one repository at once each name a branch); for a thread on a
machine the value is passed to fleet-manager `open --worktree` unchanged
(there it names an existing worktree on that machine, Herdr only); a
`worktree` that is not a non-empty string (JSON `true`, a number, a list)
is `usage` (2) naming the field and the accepted shape — judged at
`propose`, so `go` never sees it — and so is a local
thread's string git would not take as a branch name (`branch: feat/x`, a
space, `..`, a leading `-`; `git check-ref-format --branch` judges it, its
words on the line — the string passed and every `go`
failed with git's advice hint as the reason); a thread on a machine hands
its value to fleet-manager unjudged; an `unattended` of `true` or `false` is
the thread's own posture either way, above the project setting and the
coordinator's own (a `false` amendment fell through to
inherit under a `--yolo` coordinator), absent is no word (recorded `null`)
and anything else is `usage` (2); the word is read while the record is
still the proposal's or was opened on it (`posture_source` unset or
`thread`) — a reopened no-word thread re-derives its posture, and a record
proposed before this rule but never opened reads its stored `false` as
attended; `engine`
is `muse|claude|codex|shell` — what host-manager's or fleet-manager's `open
--engine` starts for the thread — and defaults to the coordinator's own
engine (the session this helper runs inside; `muse` when it cannot be
learned): a Claude Code coordinator's threads are Claude Code sessions
unless the proposal says otherwise; any other value is `usage` (2) naming
the four; `engine_args` is a JSON list of strings, the engine's own
arguments, passed one `--engine-arg` each (a flag it names is the thread's
own choice: the coordinator's inherited copy of that flag is dropped, so
the engine never sees it twice); `model` and `effort` are Muse
flags — on a thread whose engine is not `muse` either is `usage` (2),
pointing at `engine_args` — and the launcher's own Muse flags (`--model`,
`--reasoning-effort`, …) are inherited by a Muse thread alone, while the
`agents` gate env reaches every engine; `effort`
is one of the engine's tiers (`none|minimal|low|medium|high|xhigh|max|ultra`;
any other value is `usage`, 2, naming them, before anything is written —
the engine exits at once on an unknown tier and every open fails); `model`
is an id the engine can open here: one the catalog it cached at its last
start lists (`model-catalog/` under its data root, every provider and
profile) or the launching session's own `--model`; any other value is
`usage`, 2, naming the accepted ids, before anything is written (the engine
checks nothing itself: a name it cannot resolve dies at the thread's first
call, after the thread opened, and the thread ends `orphaned` with no
report); when neither source is readable a set `model` is `usage` too,
saying so — leave it unset and the thread inherits the coordinator's;
`kind` defaults to `work` — a `kind: follow` row with no PR is recorded but
never opened by `go`; the line's `text` and a `progress` line say so and name
`follow <slug> --pr <url>`, which opens the follow thread at the first `PR:`
line (opened from the proposal, it polled from t=0 with
nothing to follow); `owns` is a JSON list of paths (one string is
taken as a list of one) — the files this thread alone writes; `go` writes
them into every brief of the project (`You own: …` for the thread, `Siblings
own: <id>: …` for the others), so two threads that both name `CHANGES.rst`
are seen before they conflict; `test_command` is one
string, the command that runs the repository's tests, written into the
brief as `Test command: …` (either field of a wrong type is `usage`, 2,
before anything is written). Each thread is written with `status: proposed`.
`--replace` rewrites a thread that is still `proposed` in place — the
requester changed the split, the brief is narrowed — keeping its
`proposed_at`, its `attempts`, and — for a local thread only — its
`worktree_path` when the rewrite names the same branch, the same repository
(`repo`, else `cwd`, compared canonically on both sides through `repo_root`,
so a record written with a trailing slash or a symlink still matches) and
the same machine (the checkout a failed open made is this thread's own;
another branch, repository or machine drops it and a `progress` line,
printed only when the batch is written, names the checkout left behind) —
and stamping `amended_at`. A thread on another machine keeps `repo` as
spelled, never resolved on this disk, and never keeps a checkout on this
disk. The thread's own checkout, given as `repo` or `cwd`, names the same
repository.
A work thread's brief ends at its push and the `PR:` line; the landing
words and the follow row belong to `references/roles.md` § Implementer, and
the receipt says whose the landing is. A `--threads-json PATH`
outside the project folder (a shared `/tmp/threads.json` was overwritten by
another coordinator within minutes) is read and
answered with a `warnings` entry too (`kind: threads_json_outside_project`,
`next` = `pass the JSON on stdin (--threads-json -)`). A warning refuses
nothing; `warnings` is absent when there is none.
A thread that ran is `thread_exists` (3) with or without the flag. `proposed` (0) with `threads`
(id, name, machine, `replaced`) and `next` = `show the list to the user;
plain goal (one reading, nothing irreversible or outside the repository before
the first report) → go <slug> <ids…> in this turn and tell the user the plan;
else ask whether to open them, END the turn, and on any yes of theirs
("unattended" is a posture, not consent) → go <slug> <ids…>` (the bare `go` line
under `start_threads: auto`; 2 of 4 goal turns went
propose → go on their own). Nothing is
opened. A local thread that names neither `cwd` nor `repo` takes the
project's first recorded repository (`state.json.repos[0]`, when it still
is a directory) as both — never the process's own directory (a coordinator running from repo A gave project B's threads
worktrees of A) — and a `progress` line says so; a project whose state
predates `repos` keeps the process-directory default.

记录协调者的提议——用户尚未批准的线程。JSON 是 `{"threads": [{id, name, brief, cwd?, repo?, worktree?, machine?, engine?, engine_args?, model?, effort?, unattended?, kind?, owns?, test_command?, workstream?}]}`：`id` 为 `[a-z0-9-]{1,32}` 且在项目内唯一（已被占用的 id 是 `thread_exists`，3）；`brief` 是告知线程的那一页；`workstream`（一个标题，或 `{title, source_ref}`）只为该线程在状态表中分组，不作他用——绝不是会话名或标签——并且 `propose` 把该指派写入 `tracking.json`（标题以外的任何东西都是 `usage`，2）；`machine` 默认 `local`；`worktree` 是一个**分支名**：`go` 为本地线程在仓库旁给出该分支自己的检出（分支是新建时从 `HEAD` 创建）并把路径记入 `worktree_path`（`null` = 在 `cwd` 中工作；同时写同一个仓库的两个线程各自指名一个分支）；对机器上的线程，该值原样传给 fleet-manager 的 `open --worktree`（在那里它指名该机器上一个已存在的 worktree，仅 Herdr）；不是非空字符串的 `worktree`（JSON `true`、数字、列表）是 `usage`（2），点名该字段与接受的形状——在 `propose` 时判定，因此 `go` 永远看不到它——本地线程的、git 不会当作分支名的字符串同理（`branch: feat/x`、空格、`..`、前导 `-`；由 `git check-ref-format --branch` 判定，判定的话照单全收——曾有一个通过了校验的字符串，之后每次 `go` 都以 git 的建议提示为原因而失败）；机器上的线程把它的值不加判断地交给 fleet-manager；`true` 或 `false` 的 `unattended` 无论如何都是线程自己的姿态，高于项目设置和协调者自身的（一次 `false` 修正曾在 `--yolo` 协调者之下落回了 inherit），缺席即不置一词（记录为 `null`），其他任何值都是 `usage`（2）；这个词只在记录仍是提案的、或正是基于它开启的时候被读取（`posture_source` 未设置或为 `thread`）——重开的无词线程重新推导其姿态，而在这条规则之前提议、却从未开启的记录把它存的 `false` 读作 attended；`engine` 是 `muse|claude|codex|shell`——host-manager 或 fleet-manager 的 `open --engine` 为线程启动的东西——默认为协调者自己的引擎（本辅助脚本所在的会话；学不到时为 `muse`）：Claude Code 协调者的线程就是 Claude Code 会话，除非提案另有说法；其他任何值都是 `usage`（2），点名这四个；`engine_args` 是字符串的 JSON 列表，引擎自己的参数，每个以一次 `--engine-arg` 传递（它点名的旗标是线程自己的选择：协调者继承来的那份旗标被丢弃，引擎绝不会看到两次）；`model` 与 `effort` 是 Muse 旗标——在引擎不是 `muse` 的线程上，任何一个都是 `usage`（2），指向 `engine_args`——启动器自己的 Muse 旗标（`--model`、`--reasoning-effort`，…）只由 Muse 线程继承，而 `agents` 门环境变量到达每个引擎；`effort` 是引擎的层级之一（`none|minimal|low|medium|high|xhigh|max|ultra`；任何其他值都是 `usage`，2，点名它们，在任何写入之前——引擎遇到未知层级会立刻退出，每次开启都会失败）；`model` 是引擎在此能开启的一个 id：其上次启动时缓存的目录（其数据根下的 `model-catalog/`，涵盖每个 provider 和 profile）所列出的，或发起会话自己的 `--model`；任何其他值都是 `usage`，2，点名接受的 id，在任何写入之前（引擎自己什么也不检查：一个它解析不了的名字会在线程的第一次调用时死掉——那时线程已开启——线程最终 `orphaned` 且没有报告）；两个来源都读不到时，设置了 `model` 也是 `usage`，并说明原因——留空则线程继承协调者的；`kind` 默认 `work`——没有 PR 的 `kind: follow` 行会被记录但绝不被 `go` 开启；该行的 `text` 和一条 `progress` 行说明这一点并点名 `follow <slug> --pr <url>`，它在第一条 `PR:` 行处开启 follow 线程（若从提案直接开启，它会从 t=0 起轮询却无物可跟）；`owns` 是路径的 JSON 列表（单个字符串按只有一个元素的列表处理）——只有这个线程写的文件；`go` 把它们写进项目的每个 brief（该线程是 `You own: …`，其他线程是 `Siblings own: <id>: …`），于是两个都点名 `CHANGES.rst` 的线程在冲突之前就被看见；`test_command` 是一个字符串，运行仓库测试的命令，以 `Test command: …` 写进 brief（任一字段类型错误都是 `usage`，2，在任何写入之前）。每个线程以 `status: proposed` 写入。`--replace` 原地重写一个仍是 `proposed` 的线程——请求者改了拆分、brief 被收窄——保留其 `proposed_at`、其 `attempts`，以及——仅对本地线程——当重写指名同一分支、同一仓库（`repo`，否则 `cwd`，两侧都经 `repo_root` 规范化比较，因此带尾斜杠或符号链接写入的记录仍然匹配）且同一机器（失败的 open 留下的检出属于本线程自己；另一分支、仓库或机器则丢弃它，一条仅在该批写入时打印的 `progress` 行点名被留下的检出）时保留其 `worktree_path`——并盖上 `amended_at`。另一台机器上的线程按拼写保留 `repo`，绝不在本磁盘上解析，也绝不在本磁盘上保留检出。线程自己的检出，以 `repo` 或 `cwd` 给出时，指名的是同一仓库。
工作线程的 brief 止于它的 push 和 `PR:` 行；落地的话术与 follow 行属于 `references/roles.md` § Implementer，回执会说明落地归谁。项目文件夹之外的 `--threads-json PATH`（一份共享的 `/tmp/threads.json` 在几分钟内就被另一个协调者覆盖了）仍会被读取，并以一条 `warnings` 条目应答（`kind: threads_json_outside_project`，`next` = `pass the JSON on stdin (--threads-json -)`）。警告不拒绝任何事；没有警告时就没有 `warnings`。
运行过的线程无论带不带该旗标都是 `thread_exists`（3）。`proposed`（0），带 `threads`（id、name、machine、`replaced`）与 `next` = `show the list to the user; plain goal (one reading, nothing irreversible or outside the repository before the first report) → go <slug> <ids…> in this turn and tell the user the plan; else ask whether to open them, END the turn, and on any yes of theirs ("unattended" is a posture, not consent) → go <slug> <ids…>`（`start_threads: auto` 之下为光杆 `go` 行；4 个目标轮中有 2 个自己走了 propose → go）。什么都不开启。既不指名 `cwd` 也不指名 `repo` 的本地线程把项目记录的第一个仓库（`state.json.repos[0]`，当它仍是目录时）同时当作两者——绝不是进程自己的目录（一个从仓库 A 运行的协调者曾把 A 的 worktree 给了项目 B 的线程）——并由一条 `progress` 行说明；状态早于 `repos` 的项目保留进程目录默认值。

## `go` / 启动

```
go <slug> <thread-id…> [--asked-by WHO]
```

Opens the named threads, one host-manager or fleet-manager `open` each: a
remote thread first gets its worker-gate record copy on its machine
(FR-43932-5: `<projects-home>/<slug>/thread-copy-<id>.json` through the
copy channel, before the prompt is delivered, so the thread's first
stop can already gate; a failed open removes it again) — the copy is
what lets the thread's own Herdr hook report `unknown` to its
coordinator instead of showing the user a waiting dot. What opens is a
`proposed` thread, or a `stopped`, `orphaned` or `done` work thread, which reopens
in place — same id, same record and checkout (its own branch reused dirty
or clean: the dirt is the thread's own work; another branch checked out
there is still `worktree_failed`), a new session, its `attempts`, evidence and
PR rows kept, `ended_at`/`session_ended_at`/`stop_receipt`/`identity_drift`
cleared, `reopened_at` (a record field) stamped, `reopens` counted on the
record (and so on every `context`/`overview` row) and `reopened: true` on its
receipt (a stuck session is not reopened beside: a `running` thread answers
`already_running` and is stopped only on your own judgement). A `done` work thread
named in `go` reopens in place the same way (its session ended at `accept`;
owner ruling 55, #41777): its accepted `report` and `acked_digest` are
cleared, so the plan shows ☐ again and its next report is a new one; a
done thread `go` does not name stays done. Routing (owner ruling 58): a
follow-up about work a thread owns goes back to that thread through this
reopen — its PR, branch or findings; a PR to babysit is the follow
thread's (`follow --pr`) — and the coordinator does the work itself only
when no thread owns it. The follow thread never
reopens through `go`: `follow <slug> --pr <url>` reopens it itself, outside
`max_parallel` and with its record refreshed, and the refusal's `next`
names that. A proposed `kind: follow` row with no PR to follow is skipped —
`threads.<id>: skipped`, a `text` line, and `next` led by `the follow thread
opens at the first PR: line via follow <slug> --pr <url>` — while the other
ids open; when only such rows are named the call is `empty_follow` (3,
nothing opens, the same `next`). Ids are
required (`usage`, 2, with none): the user's `go` names
threads, so no thread starts on a bare word. Guards, in order, before
anything opens: an id in another status (`exited`) is
`no_such_thread` (3); a `running` id beside ids that can open is skipped
— `threads.<id>: already_running`, a receipt with `already_running: true`,
a `text` line `<name> [<id>]: already running — attach: …` — and the rest
open (a retried go told the coordinator to stop a
healthy thread); when every named id runs already the call is
`already_running` (0): the same receipts and `text`, `next` = the
`host-manager send <ref> --text "<line>" --type --automated` line for each
(a stuck one is `stop`ped on your judgement, never on this receipt); opening
more than `settings.max_parallel` would hold open is a `warnings` entry
(`over_max_parallel`, naming the live threads and the set that fits) and the
threads open: host-manager's `admit` is the capacity gate (D16), and your
`go` is not overruled by this skill's own setting; a
`machine` other than `local` without a fleet-manager whose `open` verb
answers `--help` (one probe per run) is `unsupported` (4) — a rename-only
fleet-manager has the file but no session verbs.

开启被点名的线程，每个一次 host-manager 或 fleet-manager 的 `open`：远程线程先在其机器上拿到它的 worker 门记录副本（FR-43932-5：经复制通道传送 `<projects-home>/<slug>/thread-copy-<id>.json`，在提示词投递之前，使线程的第一次 stop 就能被门拦；开启失败则再次移除该副本）——正是这份副本让线程自己的 Herdr 钩子向协调者报告 `unknown`，而不是给用户看一个等待中的圆点。被开启的是一个 `proposed` 线程，或一个 `stopped`、`orphaned` 或 `done` 的工作线程，后者原地重开——同一 id、同一记录与检出（复用它自己的分支，脏或干净都行：脏是线程自己的工作；另一个分支被检出到那里则仍是 `worktree_failed`）、一个新会话，其 `attempts`、证据与 PR 行保留，`ended_at`/`session_ended_at`/`stop_receipt`/`identity_drift` 清空，盖上 `reopened_at`（一个记录字段），记录上 `reopens` 计数（因而每个 `context`/`overview` 行上也有），回执上 `reopened: true`（卡住的会话绝不在旁边另开一个：`running` 线程应答 `already_running`，只有凭你自己的判断才能停它）。`go` 点名的 `done` 工作线程以同样方式原地重开（其会话在 `accept` 时结束；业主裁决 55，#41777）：其已验收的 `report` 与 `acked_digest` 被清空，于是计划里重新显示 ☐，它的下一份报告是一份新报告；`go` 未点名的 done 线程保持 done。路由（业主裁决 58）：关于某线程所拥有工作的后续，经这次重开回到那个线程——它的 PR、分支或发现；要照看的 PR 是 follow 线程的事（`follow --pr`）——只有当没有线程拥有这项工作时，协调者才自己做。follow 线程绝不通过 `go` 重开：`follow <slug> --pr <url>` 自己重开它，在 `max_parallel` 之外并刷新其记录，拒绝的 `next` 会点名这一点。没有 PR 可跟的提议型 `kind: follow` 行被跳过——`threads.<id>: skipped`、一行 `text`、以及以 `the follow thread opens at the first PR: line via follow <slug> --pr <url>` 开头的 `next`——同时其他 id 照常开启；当点名的只有这类行时，调用是 `empty_follow`（3，什么都不开启，同样的 `next`）。id 是必需的（没有则是 `usage`，2）：用户的 `go` 点名线程，因此没有线程靠一句光杆命令启动。守卫，按顺序，在任何开启之前：处于另一状态（`exited`）的 id 是 `no_such_thread`（3）；在可开启 id 之列的 `running` id 被跳过——`threads.<id>: already_running`、一条 `already_running: true` 的回执、一行 `text` `<name> [<id>]: already running — attach: …`——其余照常开启（一次重试的 go 曾叫协调者去停一个健康的线程）；当所有被点名的 id 都已在运行时，调用是 `already_running`（0）：同样的回执与 `text`，`next` = 对每一个给出 `host-manager send <ref> --text "<line>" --type --automated` 行（卡住的那个由你判断后 `stop`，绝不凭这条回执）；开启数将超过 `settings.max_parallel` 能同时保持的量是一条 `warnings` 条目（`over_max_parallel`，点名活线程与放得下的那批）且线程照常开启：host-manager 的 `admit` 才是容量门（D16），你的 `go` 不会被这个 skill 自己的设置否决；`local` 以外的 `machine` 而没有一个其 `open` 动词能应答 `--help` 的 fleet-manager（每次运行探测一次）是 `unsupported`（4）——一个只会改名的 fleet-manager 有那个文件却没有会话动词。

Per thread, in order. A local thread with `worktree` first gets its
checkout: `git worktree add <repo>-threads/<slug>-<id> <branch>` (with
`-b <branch>` when the branch does not exist yet; never `--force`); the
record's `cwd` becomes that path and `worktree_path` records it. A
directory already at the path is reused only when this thread's own
earlier open made it (the record names the path) and it is a clean
checkout of the branch; anything else there — an earlier project's
leftover with the same slug and id, a dirty tree — and a checkout git
refuses (the branch is checked out elsewhere, the parent directory cannot
be written) are `worktree_failed` for that thread — nothing opens for it,
it stays `proposed` with the attempt, its detail (git's own `fatal:` line,
never the `hint:` advice after it) and the thread's `engine` logged — the `progress` lines name the engine the thread runs at the open and in a failure (a helper text that blames the muse binary for a non-Muse thread gets the thread's own engine named after it). Then the brief file is assembled
(`threads/<id>/brief.md`: the project goal and done-means, the standing
instructions, `## The project around you` — the base block the helper
reads at open time (`Repository:`; `Checkout: <cwd> on branch <branch> from
<short base sha>` from one bounded `git rev-parse` each, the branch being
`worktree` when set; `Remote: origin <url>` when the checkout has one; `You
own:` / `Siblings own:` from the proposals' `owns`; `Test command:` from
`test_command`; a thread on a machine gets the repository and branch as
spelled and no git probe), then `MEMORY.md` and `TASKS.md` pasted verbatim
when each body is under 2 KB, `is empty` when it has no body (never "read
it" for an empty file: every thread opened on that read), else
named with its entry count and size, then the sibling threads as they stand
at that moment (id, name, state, the first sentence of its brief, at most 200 characters — never the whole brief) — the coordinator's `brief`, the thread rules the helper writes (the brief's `## How to work` section;
`threads/<id>/brief.md` has the exact text), the directory to work in, and the exact
`report` command with `--file threads/<id>/report.md` under the projects
home — the report lives in the project folder, never in the checkout); then `open --name <slug>-<id> --cwd <cwd> --purpose …
--prompt-file <brief> --exact-name --env MUSE_AGENTS_PROJECT=<slug> --env
MUSE_AGENTS_THREAD=<id> --env MUSE_AGENTS_ROLE=thread [--env
MUSE_PROJECTS_HOME=…] --env MUSE_EXPERIMENTAL_AGENTS=on [--engine-arg=--model=<m>]
[--engine-arg=--reasoning-effort=<e>] [--engine-arg=<inherited>…] [--unattended]` runs on host-manager
(`local`), or `open <machine> --name … --cwd … --purpose … --exact-name
[--worktree <value>] [--engine-arg=--model=<m>] [--engine-arg=…]
--engine-arg=<the brief text> [--unattended]` on fleet-manager: a tmux
machine has no readiness signal for a prompt file (fleet-manager sends
nothing and says so), so the brief rides as the engine's last argument, the
way the local launcher passes it, on every provider. A `shell` thread takes
no brief as argv on either side (host-manager would run `bash '<the brief
text>'`): its open carries no `--prompt-file` and
no brief argument, the record says `brief_delivery: file` with `brief_path`
(`threads/<id>/brief.md`), a `progress` line names it, and the coordinator
types the line that reads it; every other thread records `brief_delivery:
prompt-file` (local) or `engine-arg` (remote). Posture (ADR 38715 D16 as
amended by Amendment 6), local and remote alike: the requested posture is
the thread's own `unattended` (`true` or `false` — an explicit `false` is
the thread's attended word), else the project setting when the
user set it (`true`/`false`), else (`inherit`) the coordinator's own — read
from the coordinating session's command line, where Muse
`--yolo`/`--disable-approval`, Claude Code `--dangerously-skip-permissions`
(or `--permission-mode bypassPermissions`) and Codex
`--dangerously-bypass-approvals-and-sandbox`/`--ask-for-approval never` mean
approvals off, any other line means attended, and a line that cannot be
read (a sandboxed tool shell, a launcher that is no known engine) means
attended, said in a `progress` line; the record's `posture_source` names
the rule. `--unattended` is passed when that posture is unattended and the
helper's `open` advertises the flag (`unattended_flag`); nothing is passed
for `attended` (the engine's own prompts; a prompt the thread sits on
surfaces as `waiting-on-you`). An attended local thread in a checkout the
helper made gets the engine's own rules file first, scoped to that
checkout — Claude Code `.claude/settings.local.json` (edits inside it, the
record's `test_command`, git on its branch; kept out of `git status`
through the repository's `info/exclude`); Muse and Codex get none (Muse's
persistent prefix rule at the first prompt is already workspace-scoped;
Codex reads no per-directory settings and its `workspace-write` sandbox
already confines writes; said once in a `progress` line) — recorded as
`allow_list_path`; a file the helper did not write is left alone and said,
and a checkout that cannot take the file opens with the engine's own
prompts, said (`references/allow-list.md` § Thread allow-list). `go`'s `text` says whose posture the threads got
(`threads run with my permission posture: <posture> (<why>)`, then any
thread whose own word differs). The record's `posture_applied` is `unattended` or
`attended` when the helper advertises the flag (D16: a plain `open` is the
engine's own prompts), else `provider_default` with a `progress` line (a
helper that predates D16; its default is not known here).

每线程，按顺序。带 `worktree` 的本地线程先拿到自己的检出：`git worktree add <repo>-threads/<slug>-<id> <branch>`（分支尚不存在时加 `-b <branch>`；绝不用 `--force`）；记录的 `cwd` 变成该路径，`worktree_path` 记下它。路径上已存在的目录仅当是本线程自己更早的开启所建（记录点名该路径）且是该分支的干净检出时才复用；那里的其他任何东西——更早项目留下的同 slug 同 id 的残余、一棵脏树——以及 git 拒绝的检出（分支已在别处检出、父目录不可写）都是该线程的 `worktree_failed`——它什么也不开启，保持 `proposed` 并留下这次尝试，其细节（git 自己的 `fatal:` 行，绝不是其后的 `hint:` 建议）与线程的 `engine` 一并记录——`progress` 行在开启时与失败时点名线程运行的引擎（一个把非 Muse 线程的失败归咎于 muse 二进制的辅助脚本文本，会在其后附上该线程自己的引擎名）。然后组装 brief 文件（`threads/<id>/brief.md`：项目目标与 done-means、常设指令、`## The project around you`——辅助脚本在开启时读取的基础块（`Repository:`；`Checkout: <cwd> on branch <branch> from <short base sha>`，各由一次有界的 `git rev-parse` 得出，设置了 `worktree` 时分支即它；检出有远程时为 `Remote: origin <url>`；`You own:` / `Siblings own:` 来自提案的 `owns`；`Test command:` 来自 `test_command`；机器上的线程按拼写得到仓库与分支、不做 git 探测），然后 `MEMORY.md` 与 `TASKS.md` 在各自正文不足 2 KB 时原文粘贴、无正文时写 `is empty`（空文件绝不说"去读它"：每个线程都会为这一次读取开启）、否则按条目数与大小点名，然后是彼时兄弟姐妹线程的现状（id、name、状态、其 brief 的第一句，至多 200 字符——绝不是整个 brief）——协调者的 `brief`、辅助脚本写的线程规则（brief 的 `## How to work` 节；`threads/<id>/brief.md` 有确切文本）、要工作的目录，以及带上 `--file threads/<id>/report.md`、位于项目根之下的确切 `report` 命令——报告活在项目文件夹里，绝不在检出里）；然后 `open --name <slug>-<id> --cwd <cwd> --purpose … --prompt-file <brief> --exact-name --env MUSE_AGENTS_PROJECT=<slug> --env MUSE_AGENTS_THREAD=<id> --env MUSE_AGENTS_ROLE=thread [--env MUSE_PROJECTS_HOME=…] --env MUSE_EXPERIMENTAL_AGENTS=on [--engine-arg=--model=<m>] [--engine-arg=--reasoning-effort=<e>] [--engine-arg=<inherited>…] [--unattended]` 在 host-manager 上运行（`local`），或 `open <machine> --name … --cwd … --purpose … --exact-name [--worktree <value>] [--engine-arg=--model=<m>] [--engine-arg=…] --engine-arg=<the brief text> [--unattended]` 在 fleet-manager 上：一台 tmux 机器没有提示文件的就绪信号（fleet-manager 不发送并明说），所以 brief 作为引擎的最后一个参数搭车，如同本地启动器传它的方式，在每个提供者上都如此。`shell` 线程在两侧都不把 brief 作为 argv（host-manager 会运行 `bash '<the brief text>'`）：它的开启不带 `--prompt-file` 也不带 brief 参数，记录写 `brief_delivery: file` 与 `brief_path`（`threads/<id>/brief.md`），一条 `progress` 行点名它，由协调者键入读取它的那行命令；其他每个线程记录 `brief_delivery: prompt-file`（本地）或 `engine-arg`（远程）。姿态（ADR 38715 D16 经修正案 6 修订），本地远程一致：所请求的姿态是线程自己的 `unattended`（`true` 或 `false`——显式 `false` 是线程的 attended 之词），否则用户设置过的项目设置（`true`/`false`），否则（`inherit`）协调者自己的——从协调会话的命令行读取，其中 Muse 的 `--yolo`/`--disable-approval`、Claude Code 的 `--dangerously-skip-permissions`（或 `--permission-mode bypassPermissions`）与 Codex 的 `--dangerously-bypass-approvals-and-sandbox`/`--ask-for-approval never` 意为审批关闭，任何其他命令行意为 attended，读不到的命令行（沙盒工具 shell、一个不是已知引擎的启动器）也意为 attended，由一条 `progress` 行说明；记录的 `posture_source` 点名是哪条规则。该姿态为 unattended 且辅助脚本的 `open` 宣告该旗标（`unattended_flag`）时才传 `--unattended`；attended 则什么都不传（引擎自己的提示；线程停在提示上会以 `waiting-on-you` 浮现）。辅助脚本所建检出中的 attended 本地线程先得到引擎自己的规则文件，限定于该检出——Claude Code 的 `.claude/settings.local.json`（允许其内编辑、记录的 `test_command`、其分支上的 git；经仓库的 `info/exclude` 避开 `git status`）；Muse 与 Codex 不给（Muse 首条提示上的持久前缀规则本就按工作区限定；Codex 不读按目录的设置且其 `workspace-write` 沙盒已限定写入；由一条 `progress` 行说明一次）——记录为 `allow_list_path`；非辅助脚本所写的文件原样保留并说明，收不下该文件的检出以引擎自己的提示开启，同样说明（`references/allow-list.md` § Thread allow-list）。`go` 的 `text` 说明线程们拿到了谁家的姿态（`threads run with my permission posture: <posture> (<why>)`，然后是任何自有措辞不同的线程）。辅助脚本宣告该旗标时（D16：不带修饰的 `open` 即引擎自己的提示），记录的 `posture_applied` 是 `unattended` 或 `attended`，否则为 `provider_default` 并带一条 `progress` 行（一个早于 D16 的辅助脚本；其默认值在此不可知）。

Every thread gets the coordinator's own settings too (owner ruling
2026-09-20, coordinators and threads alike): the `agents` gate pair on a
local open, and the launching session's engine args as `init --detach`
carries them, except a flag the record sets itself (`model`, `effort`),
which wins; the line's `launch_settings` names the gate pair, the source,
and under `engine_args` what each thread id was actually given. A local
Muse thread runs the coordinator's own executable, handed to host-manager
as `--engine <path>` (`MUSE_BIN` when the session that launched this one set it, else the
coordinator's command resolved); `launch_settings.binary` names it, and a
fallback to `muse` by name is said in `binary_fallback` and a `progress`
line, never silently. A remote thread keeps the bare `muse` of its machine.

每个线程也拿到协调者自己的设置（业主裁决 2026-09-20，协调者与线程一视同仁）：本地开启上的 `agents` 门对，以及发起会话的引擎参数（如 `init --detach` 携带的那样），但记录自己设置的旗标（`model`、`effort`）除外，以它为准；该行的 `launch_settings` 点名门对、来源，并在 `engine_args` 之下给出每个线程 id 实际拿到了什么。本地 Muse 线程运行协调者自己的可执行文件，以 `--engine <path>` 交给 host-manager（发起本会话的那个会话设置了 `MUSE_BIN` 时用它，否则解析协调者的命令）；`launch_settings.binary` 点名它，按名字回退到 `muse` 会在 `binary_fallback` 和一条 `progress` 行中说明，绝不悄悄进行。远程线程保留其机器上的光杆 `muse`。

The record takes the provider, server, identity and receipt the helper
returned, the **bare** ref (`identity.ref`; fleet-manager's `ref` is the
address `<machine>/<ref>`, which the helper composes itself, once, for
every later `status`/`close`), a `note` the helper gave (in `progress` and
`open_note`), and `status: running`. A repeated id in one `go` opens once.
Every open's `--purpose` ends in `[<instance>]`, six hex digits naming this
project folder (`context.project.instance`; two projects with one slug —
two homes, two coordinators on one host — differ). When `open` answers
`name_taken` for the exact name, host-manager `list` is asked whose the
session is: live, in this thread's directory, with this instance's marker
means it is this thread's own (an open whose record write died) and the
record adopts it (`running`, `adopted: true`; `posture_applied` stays what
the open recorded unless the row states a posture); anything else under the name
— another project with this slug, a stale name — is never adopted or
touched: the attempt is logged and the thread opens under
`<slug>-<id>-<instance>` (a `progress` line says so); that name is this
project's alone, so `name_taken` on it is this thread's own earlier open
and is adopted the same way. A retried `go` whose own checkout is no longer
reusable (dirty, another branch) adopts this thread's live session in it
rather than stranding it.

记录拿下辅助脚本返回的 provider、server、identity 与回执、**裸** ref（`identity.ref`；fleet-manager 的 `ref` 是地址 `<machine>/<ref>`，由辅助脚本自己组装，只组装一次，供此后每次 `status`/`close` 使用）、辅助脚本给出的一条 `note`（在 `progress` 与 `open_note` 中），以及 `status: running`。同一次 `go` 里重复的 id 只开启一次。每次开启的 `--purpose` 以 `[<instance>]` 结尾，六个十六进制位点名这个项目文件夹（`context.project.instance`；同 slug 的两个项目——同一主机上的两个家、两个协调者——得以区分）。当 `open` 对确切名字应答 `name_taken` 时，会询问 host-manager 的 `list` 这个会话是谁的：活着的、在本线程的目录里、带着本 instance 的标记，意味着它就是本线程自己的（一次记录写入死掉的开启），记录将其收养（`running`、`adopted: true`；除非该行写明姿态，`posture_applied` 保持 open 所记录的）；该名字下的其他任何东西——同 slug 的另一个项目、一个陈旧名字——绝不被收养或触碰：尝试被记录，线程改为在 `<slug>-<id>-<instance>` 之下开启（一条 `progress` 行说明）；那个名字为本项目独有，因此它之上的 `name_taken` 就是本线程自己更早的开启，以同样方式收养。重试的 `go` 若自己的检出已不再可复用（脏了、换分支），则收养该线程在其中活着的会话，而不是把它晾在那里。

`started` (0) when every named thread opened or was adopted; `partial` (6)
when some did — `threads` says per id `opened`, `adopted`, `worktree_failed`
or the underlying outcome; nothing is retried. A failed open's `progress`
line and its `attempts[].detail` carry the helper's reason (host-manager's
`message`, e.g. "the lane exited within 1s of launch"; fleet-manager's
`error`; else the last stderr line), never the bare outcome twice, and
`attempts[].next` the helper's own cure when it named one; `partial`'s
`next` is the first failed thread's cure (`<id>: <cure>`), followed by the
wake line only when another thread opened in the same call. The same
reason fills `error` when `init --detach`, `follow` or `agents.py archive`
pass a helper's failure through. `receipts`: one per thread, each with its
`attach`. `mode_line` (host-manager's `mode=<x> (<why>)`: why the thread
landed in tmux, Herdr or msp) rides on the `opened as` progress line, the
record and the `go` receipt, so the coordinator can say it; when the open receipt's `progress` says `msp not chosen: <reason>`
(the session protocol on, msp unusable here) that reason is folded into the
line verbatim — `mode=tmux (msp not chosen: <reason>; tmux; no Herdr server)`
— so a flag-on user reads why the threads are tmux (#41827).

所有被点名线程都开启或被收养时为 `started`（0）；部分如此时为 `partial`（6）——`threads` 按 id 说明 `opened`、`adopted`、`worktree_failed` 或底层结果；不重试任何东西。失败开启的 `progress` 行与其 `attempts[].detail` 携带辅助脚本的原因（host-manager 的 `message`，如"泳道在启动后 1 秒内退出"；fleet-manager 的 `error`；否则是最后一行 stderr），绝不把光秃秃的结果重复两遍，`attempts[].next` 则是辅助脚本自己点名的补救；`partial` 的 `next` 是第一个失败线程的补救（`<id>: <cure>`），只有当同一次调用有别的线程开启时才在其后附上唤醒行。当 `init --detach`、`follow` 或 `agents.py archive` 透传辅助脚本的失败时，同一原因填入 `error`。`receipts`：每线程一条，各带其 `attach`。`mode_line`（host-manager 的 `mode=<x> (<why>)`：线程为何落在 tmux、Herdr 或 msp）搭在 `opened as` progress 行、记录与 `go` 回执上，使协调者能说出口；当开启回执的 `progress` 说 `msp not chosen: <reason>`（会话协议开着、msp 在此不可用）时，该原因原样折进这一行——`mode=tmux (msp not chosen: <reason>; tmux; no Herdr server)`——于是开了旗标的用户能读懂线程为何是 tmux（#41827）。

`plan_lines` (on `go`, `context`, `tick`, `ack` and `accept` alike, every
look included) is the plan as the user sees it, ready to post as printed: one ☐ per opened
thread, name first, then what it owns (else its brief's first line); ✅ once
its done report is acked — a report with a `PR:` line, or a STATUS opening
with done/finished/complete(d)/merged/pushed/landed, whose digest is
`acked_digest` (a progress report acked stays ☐); ✅✔ once accepted (the
merge decision keeps its own mark: ✅ on `accept` alone flipped one run in
three, because coordinators ack on the wake and accept at archive); a
proposed-but-unopened thread is on `propose`'s `plan_lines` alone — the
plan told before the threads open — and not on the others. `channel_line`
beside it is the one channel `reply` call that posts the list, in the
conversation coordinator's own shape — `reply --to <lane>
--replace-last <<'MSG'` … `MSG`, the lines on stdin (a backtick in a brief
line never runs), `--replace-last` editing the plan while it is the lane's
newest message (post nothing else while the project runs, so it stays the
newest; never a re-post); the lane
fills `<lane>` (channel row: the per-thread list never
reached the channel and the ticked list came as a new message; composed by
the model, the ✅ re-post landed three times in seven).

`plan_lines`（在 `go`、`context`、`tick`、`ack` 与 `accept` 上都一样，每次查看都在）是用户看到的那份计划，按打印原样即可发布：每个已开启线程一个 ☐，名字在前，然后是它拥有什么（否则是其 brief 的第一行）；✅ 在其 done 报告被 ack 之后——一份带 `PR:` 行的报告，或一份以 done/finished/complete(d)/merged/pushed/landed 开头的 STATUS，其 digest 为 `acked_digest`（被 ack 的进度报告保持 ☐）；✅✔ 在被验收之后（合并决定保留自己的记号：只在 `accept` 上打 ✅ 曾三次里错一次，因为协调者在唤醒时 ack、在归档时才 accept）；提议而未开启的线程只出现在 `propose` 的 `plan_lines` 上——线程开启前预告的计划——不出现在其他动词上。旁边的 `channel_line` 是发布该列表的那一次 channel `reply` 调用，用对话协调者自己的形状——`reply --to <lane> --replace-last <<'MSG'` … `MSG`，各行走 stdin（brief 行里的反引号绝不会运行），`--replace-last` 在计划仍是泳道最新消息时编辑它（项目运行期间不要发布其他东西，使它保持最新；绝不重发）；`<lane>` 由泳道填（channel 行：逐线程列表曾从未到达频道、打勾列表以一条新消息到来；由模型即兴拼装时，✅ 重发七次里来了三次）。

A local thread whose `cwd` is the worktree this helper made (`worktree_path`)
from one of the coordinator's repositories — the ones `init` recorded
(`state.json.repos`) whose root the user already trusted, in Muse's own
`trust.json` (read-only; never the cwd of the process running `go`) or on
the coordinating session's own command line for this run (`--trust-workspace`
or `--yolo` on the line, and the workspace it trusts — `--workspace <root>`,
else the repository around the coordinator's cwd — is that root; a `--yolo`
session's trust is never persisted, ADR 38715 Amendment 6 item 4; an
explicit `untrusted` record in the store wins over the line) — one `.git` common
directory — inherits that trust and the hooks acceptance that goes with it:
a Muse thread opens with `--engine-arg=--trust-workspace` (project skills,
rules and hooks load for the run), a Claude Code or Codex thread
opens with host-manager `open --trusted` (the engine's own per-directory
trust record pre-seeded, no skip-permission flag) when `open --help`
advertises it (`trusted_flag`), else a `progress` line says the engine's
dialog stands; the record carries `trust_workspace: true` and `trust_reason`
once the open succeeded — a failed open records neither trust field, an
adopted session records `null`; a `progress` line — because the user
trusted that repository, and nobody answers a thread's trust prompt.

一个 `cwd` 是本辅助脚本从协调者的某个仓库所建 worktree（`worktree_path`）的本地线程——即 `init` 记录的那些（`state.json.repos`）、其根用户已在 Muse 自己的 `trust.json` 中信任过的（只读；绝不是运行 `go` 的进程的 cwd）、或本轮协调会话自己的命令行上信任过的（行上有 `--trust-workspace` 或 `--yolo`，且它信任的工作区——`--workspace <root>`，否则是协调者 cwd 周围的仓库——就是那个根；`--yolo` 会话的信任绝不持久化，ADR 38715 修正案 6 第 4 条；存储中显式的 `untrusted` 记录压过命令行）——同一个 `.git` 公共目录——继承那份信任及随之而来的 hooks 接受：Muse 线程以 `--engine-arg=--trust-workspace` 开启（项目 skills、规则与 hooks 为本次运行加载），Claude Code 或 Codex 线程在 `open --help` 宣告时以 host-manager 的 `open --trusted` 开启（预置引擎自己的按目录信任记录，不用跳过权限旗标）（`trusted_flag`），否则一条 `progress` 行说明引擎的对话框照旧；开启成功后记录携带 `trust_workspace: true` 与 `trust_reason`——失败的开启两个信任字段都不记，被收养的会话记 `null`；要一条 `progress` 行——因为用户信任的是那个仓库，而没有人会去应答线程的信任提示。

Installed-hook acceptance rides separately and to EVERY local thread (ADR
38715 Amendment 9, owner ruling 64): when the coordinator's own Muse
settings file exists (`$XDG_CONFIG_HOME`/`~/.config`, `muse` or `tbh`,
`settings.json`) and host-manager's `open --help` advertises
`--hooks-approved` (`hooks_flag`), the thread's `open` carries that file
and host-manager seeds the thread's own settings file from it where the two
differ — a hook the user changed or refused still prompts; without the flag
a `progress` line says the engine's own hooks review stands. A coordinator
whose launcher vouches for every session below it (its environment carries
`MUSE_AGENTS_LAUNCHER_TRUST=inherit`, which only such a launcher sets) opens
EVERY local thread, any engine and any directory, with host-manager `open
--trusted` as well — the checkout's own trust record, so no thread of such a
session shows a startup dialog of any kind (owner ruling 65); the record
says `trust_reason: a thread of a session its launcher vouches for`. A work
thread at a repository root itself, a worktree of a recorded-but-untrusted
or trusted-but-unrecorded repository, another clone's worktree and any other
directory keep the engine's own prompt (`trust_workspace: false`; the follow
thread's rule is under `follow`). Owner rulings 2026-09-20 (spec
FR-38715-9).

已安装钩子的接受单独搭乘、且面向每个本地线程（ADR 38715 修正案 9，业主裁决 64）：当协调者自己的 Muse 设置文件存在（`$XDG_CONFIG_HOME`/`~/.config`，`muse` 或 `tbh`，`settings.json`）且 host-manager 的 `open --help` 宣告 `--hooks-approved`（`hooks_flag`）时，线程的 `open` 携带该文件，host-manager 在两者不同的处以它为线程自己的设置文件播种——用户改过或拒绝过的钩子仍会提示；没有该旗标则一条 `progress` 行说明引擎自己的 hooks 审查照旧。启动者为其下每个会话作保的协调者（其环境携带 `MUSE_AGENTS_LAUNCHER_TRUST=inherit`，只有这样的启动者会设置）以 host-manager 的 `open --trusted` 开启每一个本地线程，任何引擎、任何目录——检出自己的信任记录，于是这种会话的任何线程都不显示任何形式的启动对话框（业主裁决 65）；记录写 `trust_reason: a thread of a session its launcher vouches for`。位于仓库根本身的工作线程、已记录但未受信或已受信但未记录的仓库的 worktree、另一克隆的 worktree 以及任何其他目录都保留引擎自己的提示（`trust_workspace: false`；follow 线程的规则在 `follow` 之下）。业主裁决 2026-09-20（规格 FR-38715-9）。

Every opened or adopted thread records `attach`: the exact command that
puts the human in front of its session, as host-manager's or
fleet-manager's own `attach` verb answers it (`tmux -L <server> attach -t
=<name>` with the server flags in force, `herdr agent attach <ref>`, the
machine's form for a remote thread), never composed here; `null` with a
`progress` line when the helper has no such verb; and `attach_inside_tmux`:
the same tmux command with `switch-client` for `attach` (server flags kept;
`null` for another provider's command) — `tmux attach` refuses to nest and a
tmux user is usually inside one (a dead end). Every `text` line
names both: the form for where the caller sits first (`$TMUX` set in the
helper's environment: `attach: <switch-client form> (you are inside tmux;
from outside: <attach form>)`; else `attach: <attach form> (inside tmux:
<switch-client form>)`), and the per-thread receipt carries both fields. `go` returns `attach`
(per id) and `text`: one line per thread — `<name> [<id>]: <first brief
line> — attach: …`, the name first — for the user's post-go message (owner ruling
2026-09-20: a session the user cannot reach feels gone); `next` opens with
"repeat `text` to the user" at every open — a single thread, a reopen — not
only the first go. It also writes
`library/wake.sh` (the helper's wake loop, with this session's tmux
environment and projects home) and returns `monitor_line`, the ready
Monitor line on it — `monitor(command="sh <project>/library/wake.sh",
persistent=true, wake_delay_ms=0, show_lines=true, description="agents
<slug>")`; with no wake armed, `next` says to install that line as printed
and record it (coordinators spent four to five calls composing a
line, wrote it to shared /tmp and chose a two-minute period). The inbox
path (`wake_path: inbox`) changes none of this — the loop and the ready line
are the wake floor (#41228); an `inbox_target` missing since `resume` is
resolved again first.

每个被开启或收养的线程都记录 `attach`：把人放到其会话面前的确切命令，按 host-manager 或 fleet-manager 自己的 `attach` 动词所答（`tmux -L <server> attach -t =<name>` 带现行服务器旗标、`herdr agent attach <ref>`、远程线程用该机器的形式），绝不在此拼装；辅助脚本没有这个动词时为 `null` 并附一条 `progress` 行；还有 `attach_inside_tmux`：同一条 tmux 命令但以 `switch-client` 替代 `attach`（保留服务器旗标；其他提供者的命令为 `null`）——`tmux attach` 拒绝嵌套，而 tmux 用户通常就在 tmux 里（一条死路）。每行 `text` 两者都点名：先按调用方所在位置的形式（辅助脚本环境里 `$TMUX` 已设：`attach: <switch-client form> (you are inside tmux; from outside: <attach form>)`；否则 `attach: <attach form> (inside tmux: <switch-client form>)`），每线程回执也带这两个字段。`go` 返回 `attach`（按 id）与 `text`：每线程一行——`<name> [<id>]: <first brief line> — attach: …`，名字在前——供用户在 go 之后的消息使用（业主裁决 2026-09-20：一个用户够不到的会话感觉就像没了）；`next` 在每次开启时——单线程、重开——都以"向用户重复 `text`"开头，不只是第一次 go。它还写 `library/wake.sh`（辅助脚本的唤醒循环，带本会话的 tmux 环境与项目根）并返回 `monitor_line`，即其上的就绪 Monitor 行——`monitor(command="sh <project>/library/wake.sh", persistent=true, wake_delay_ms=0, show_lines=true, description="agents <slug>")`；没有布防唤醒时，`next` 说按打印原样安装那一行并记录它（协调者们曾花四到五次调用拼一行、写进共享 /tmp、还选了两分钟的周期）。收件箱路径（`wake_path: inbox`）不改变其中任何一点——循环与就绪行是唤醒的下限（#41228）；`resume` 起缺失的 `inbox_target` 先被重新解析。

## `follow` / 跟进

```
follow <slug> --pr URL [--pr URL…] [--cwd DIR] [--unattended] [--asked-by WHO]
```

The one follow thread per project (`kind: follow` — found by kind, whatever
its id: a proposal's `kind: follow` row under another id is that thread, and
`follow --pr` opens it in place under its own id with its own checkout, never
the proposal's `cwd`, and never a second follow thread beside a live one
(a gone one is replaced under its own id): it babysits
PRs to merge on a `settings.follow_every` cadence — with a local origin (no
GitHub) a PR is a branch pushed to origin and landed means merged into
origin's main; the follow thread drives it, working in its own clone
under the project folder and never checking out, merging or creating
branches in the user's clone; the coordinator and the work threads never
land; its merge gate is the suite green, never a list of accepted
failures — event-first (it reacts to
the first failed check, review threads, conflicts and the queue), self-reviews
each head, enqueues once, verifies the merge on the target branch, and exits
after its report when nothing it follows is open. When no live follow thread
exists, `follow` opens one through host-manager `open` with the follow brief
— its `## Your task` opens with `What this project calls a PR and merged (the
goal's own words): …`, the `## Goal` sentences that mention a PR or a merge
(a branch on a bare origin, a merge into its main), when there are any (the follow thread spent 12-18 turns re-deriving them), and says that on
a target branch with no merge automation the landing is the follow thread's
(merge each PR and push the target by explicit refspec once green — the one
case a thread pushes the target — then file `merged`, never `enqueued`; a
follow thread once stalled 746 s on "I may not push main") — and the PR list
(`following`, 0, `created: true`), its posture — ADR 38715 D16 as amended by Amendment 6: `--unattended` on this verb, else the project setting when the user set it, else the coordinator's own posture (`inherit`; attended when its line cannot be read), recorded as `posture_source`; never inferred from the work threads' postures (per-thread `unattended: true` under an attended project left the follow thread on its staged merge prompts — so put the user's word where the follow thread is decided; `posture_applied` on the record says which, and `go` says it in advance: `follow_posture`, and a `text` line `follow thread: opens <posture> (<why>)` before it exists, `follow thread: live, opened <posture_applied>` — `provider_default` for a pre-D16 record — after a liveness probe says it runs, `state unknown …` when its provider cannot answer; `--unattended` on a live follow thread changes nothing and says so in a `progress` line) and
`--engine-arg=--trust-workspace` for its own directory — a detached
checkout of the coordinator's first recorded repository (`state.json.repos`,
whatever process opens it; a gone one falls back to the caller's repository,
said in a `progress` line when that default is used) that the helper makes
beside the work threads' worktrees, `<repo>-threads/<slug>-follow` (`git
worktree add --detach`; reused when a checkout of that repository already
stands there; the record's `cwd` and `worktree_path` name it, so `agents.py archive`
removes it once landed and clean) — never the user's clone itself (the follow thread merged and pushed from the user's clone and
moved their HEAD four times); when git can make none (a repository with no
commit yet) it opens in the repository after all and a `progress` line says
its HEAD may move — unless `--cwd` names another or the previous
follow record already carried one (a reopen keeps its directory while it
still exists; a gone one falls back to the default above — the first
recorded repository, else the caller's repository — and says so in a
`progress` line; a relative `--cwd` is stored resolved, and a record saved
relative by an earlier helper is treated like a gone one; a project whose
state predates `repos` keeps the pre-field default, the caller's
repository, with no notice) — when that directory is inside one of the coordinator's
repositories that the user trusted in Muse's `trust.json` (a tick or a
detached coordinator that opens it has nobody to answer the trust prompt;
`trust_workspace: true` and `trust_reason` on its record once the open
succeeded; a `--cwd` outside that repository keeps the engine's prompt; a
`--cwd` that is not a directory is refused before any record is written
(`usage`, 2); a work thread gets the flag only
for a checkout this helper made from the coordinator's repository, see
`go`); when one
is live, the new URLs are typed into it as one automated line — host-manager
`send <ref> --type --automated`, never the peer path, which Muse opens only
to a session that has messaged this one first, and a thread this helper
opened never has (`following`, 0, `created: false`, `delivered`, `delivery:
typed`, the receipt on the line — a notification is never delivery, #38715
ruling 18). Every `following` line carries `attach` (the follow thread's
attach command as recorded at its open) and `text`, its one line for the
user — id, brief, attach — like `go`'s; `next` says to repeat it. The record keeps every URL
either way, each row with `delivered: true|false` (a row a thread's own
`inbox put --kind pr` creates is `delivered: true` — it knows that PR; a row
written before this field reads as delivered). The record is the follow
thread's PR list: its brief names the record path and says to read it at
the start of every round, so a PR handed over while it works reaches it
whether or not the typed line does (the line typed
into a mid-turn thread sat unsent on its composer for the whole session).
Before typing, one `read --tail` judges the composer: while the engine runs
or a dialog stands, or while text already sits on the composer, nothing is
typed; that, or a send that is not a clean `sent` receipt at exit 0 —
`composer_not_empty`, `typed_unsubmitted` / `composer_not_cleared` (the text
still on the composer after Enter), any non-zero exit — is `queued` (0) with
`queued` (the URLs), `why`, `underlying` (the send's line, when one ran),
the rows marked `delivered: false`, and `next`: end the turn; the follow
thread reads its record's PR list every round, and the next `tick` (the wake
loop's round included), `context` or `follow` types it once the composer is
free (`follow_delivered` on those lines names the URLs typed; a `follow` call
with a new URL types the queued ones with it); never land it yourself (one
action; no manual `send` line, no attend-the-thread alternative and no
re-run chore — offered three options, a coordinator
merged and pushed main itself). When the composer verdict comes from a
thread whose screen shows it idle — no activity line, no dialog — the answer
is `composer_stale` (6) instead: the thread finished and an earlier
unsubmitted line sits on its composer, which no wait or retry clears (three `follow` calls over 14 minutes were told "mid-turn") —
a line that already sat on the idle thread's composer before any send is
the same case (it read `queued` for 17 minutes and
nothing freed it; `underlying` is then a `composer_not_empty` verdict this
call made itself, the line's text on it); `next` names the line's first 60
characters, says `host-manager send --type` cannot clear or type over it,
and names the stop-then-follow reopen (`stop <slug> follow`, then the same
`follow`; the reopened thread's brief carries the URL) and, again, never land
it yourself. The retry every `tick` and `context` make types nothing on top
of such a line and leads its `text` and `next` with `follow: idle behind an
unsubmitted line ("<its first 60 characters>") that no round clears: stop
<slug> follow, then follow <slug> --pr …`. The typed nudge itself is
`new PR(s) queued in your record: <n>; read <record.json path>` — a count
and the record, never a URL: the Muse composer does not submit a line that
carries a `file://` token (the root cause of the undelivered
hand-offs), and the record is the list the brief tells the
thread to read every round. A screen that says nothing lets the send itself answer.
`delivered: true` is never answered on a receipt that did not submit. A live follow thread
whose recorded `cwd` is no longer a directory is `follow_thread_lost` (6):
nothing is typed into it, the URLs are recorded `delivered: false`, `next`
names the stop-then-follow reopen; `tick` and `overview`/`context` show such a
thread as `lost — its recorded checkout <cwd> is gone` (the WAKE line too)
instead of `running` (the follow thread removed its own
checkout and two PRs were "delivered" into it while the rows said running). After a clean `sent` receipt the helper re-reads the composer once (host-manager `read --tail`): the pasted line still there is `queued` too, `underlying` = `typed_unsubmitted`/`composer_not_cleared` (`sent` answered while the composer held the line, a 991 s stall). A gone record whose own live session the
reopen adopts (`adopted: true`) gets the URLs its brief did not carry typed
the same way — adoption alone never counts as delivery (`queued` with `adopted: true` when the composer is not free). The follow
thread files PR events with `inbox put --kind pr --key pr:<head>:<event>`.
No `--pr` is `usage` (2). A `done` or `stopped` follow thread reopens in
place — done is per PR list, never terminal for the follow thread (`thread_done` left the coordinator building landing threads of its
own): same record, a new session, its `evidence`, `attempts`, report and PR
rows kept, the new URLs appended, `ended_at`/`session_ended_at`/`stop_receipt`/
`identity_drift` cleared and `reopened_at` stamped (`following`, 0,
`created: false`, `reopened: true`; the brief lists only the PRs not yet
merged). A running follow thread whose provider cannot answer is
`follow_unknown` (6) — nothing reopened or recorded, `next` is `tick <slug>`. A gone follow thread (`exited`, `orphaned`) is reopened
as a fresh record (no report, evidence or end stamp; `created: true`) carrying exactly the
PRs the old record had not seen merged plus the new URLs. `--cwd` is the directory the follow thread works in
(default: the coordinator's first recorded repository, `state.json.repos[0]`;
a reopen keeps the previous record's). The follow thread is one per project and
outside `max_parallel`.

每个项目只有一个 follow 线程（`kind: follow`——按 kind 查找，无论其 id：提案中在另一个 id 之下的 `kind: follow` 行就是它，`follow --pr` 以其自己的 id 和自己的检出原地开启它，绝不是提案的 `cwd`，也绝不在一个活线程旁边再开第二个 follow 线程（没了的在原 id 下替换）：它以 `settings.follow_every` 的节奏照看待合并的 PR——本地 origin（无 GitHub）时，PR 是一条推到 origin 的分支，landed 意为合入 origin 的 main；follow 线程驱动这件事，在项目文件夹下自己的克隆里工作，绝不在用户的克隆里检出、合并或创建分支；协调者与工作线程绝不落地；它的合并门是测试套件变绿，绝不是一份已接受失败的清单——事件优先（它对第一个失败的检查、评审线程、冲突和队列作出反应），自查每个 head，只入队一次，在目标分支上验证合并，在其所跟之物都不再开放时于报告后退出。没有活的 follow 线程时，`follow` 经 host-manager 的 `open` 以 follow 简报开启一个——其 `## Your task` 以 `What this project calls a PR and merged (the goal's own words): …` 开头，即 `## Goal` 中提到 PR 或合并的句子（bare origin 上的一条分支、合入其 main），如有（follow 线程曾花 12-18 轮重新推导它们），并说明在没有合并自动化的目标分支上，落地归 follow 线程（变绿后合并每个 PR 并以显式 refspec 推送目标——线程推目标分支的唯一情形——然后归档 `merged`，绝不是 `enqueued`；一个 follow 线程曾在"I may not push main"上卡了 746 秒）——以及 PR 列表（`following`，0，`created: true`），其姿态——ADR 38715 D16 经修正案 6 修订：本动词上的 `--unattended`，否则用户设置过的项目设置，否则协调者自己的姿态（`inherit`；其命令行读不到时为 attended），记录为 `posture_source`；绝不从工作线程的姿态推断（attended 项目之下逐线程 `unattended: true` 曾让 follow 线程停在它的暂存合并提示上——所以把用户的话放在决定 follow 线程的地方；记录上的 `posture_applied` 说明是哪种，`go` 提前说明：`follow_posture`，以及一行 `text`：它存在之前 `follow thread: opens <posture> (<why>)`，一次存活探测说明它在运行之后 `follow thread: live, opened <posture_applied>`——早于 D16 的记录为 `provider_default`——其提供者答不上来时 `state unknown …`；活的 follow 线程上的 `--unattended` 什么也不改，并在一条 `progress` 行里说明）以及为其自己目录的 `--engine-arg=--trust-workspace`——协调者第一个被记录仓库（`state.json.repos`，无论哪个进程开启它；没了的回退到调用方的仓库，使用该默认时一条 `progress` 行说明）的 detached 检出，由辅助脚本建在工作线程的 worktree 旁边，`<repo>-threads/<slug>-follow`（`git worktree add --detach`；该仓库的检出已立于彼处时复用；记录的 `cwd` 与 `worktree_path` 点名它，因此 `agents.py archive` 在落地且干净后移除它）——绝不是用户的克隆本身（follow 线程曾从用户的克隆里合并并推送，挪动了他们的 HEAD 四次）；git 建不出时（一个还没有 commit 的仓库）它索性在仓库里开启，一条 `progress` 行说明其 HEAD 可能移动——除非 `--cwd` 点名了别处，或上一个 follow 记录已带有一个（重开在其仍存在时保留其目录；没了的回退到上面的默认——第一个被记录的仓库，否则调用方的仓库——并在一条 `progress` 行中说明；相对的 `--cwd` 以解析后的形式存储，早期辅助脚本按相对保存的记录按没了的对待；状态早于 `repos` 的项目保留字段出现前的默认，即调用方的仓库，且不另行通知）——当该目录位于用户在 Muse 的 `trust.json` 中信任过的协调者仓库之一之内时（开启它的 tick 或分离协调者没有人去应答信任提示；开启成功后其记录上有 `trust_workspace: true` 与 `trust_reason`；该仓库之外的 `--cwd` 保留引擎的提示；不是目录的 `--cwd` 在写任何记录之前被拒（`usage`，2）；工作线程只对辅助脚本从协调者仓库建的检出拿该旗标，见 `go`）；当已有一个活着时，新 URL 作为一行自动化命令键入其中——host-manager 的 `send <ref> --type --automated`，绝不走 peer 路径——Muse 只对先给本会话发过消息的会话开放 peer 路径，而本辅助脚本开启的线程从来没有（`following`，0，`created: false`，`delivered`，行上的回执 `delivery: typed`——通知绝不是送达，#38715 裁决 18）。每行 `following` 携带 `attach`（开启时记录的 follow 线程 attach 命令）与 `text`，给用户的那一行——id、brief、attach——与 `go` 的相同；`next` 说要重复它。无论哪种方式，记录都保留每个 URL，每行带 `delivered: true|false`（线程自己的 `inbox put --kind pr` 建的行是 `delivered: true`——它认识那个 PR；该字段出现前写的行按已送达解读）。记录就是 follow 线程的 PR 清单：它的 brief 点名记录路径并要求每轮开始时读它，于是它工作期间交来的 PR 能到达它，无论键入的行到没到（键入一个轮中线程的那行在整个会话里都未发送地停在它的输入框上）。键入之前，一次 `read --tail` 判断输入框：引擎在运行、或有对话框立着、或输入框上已有文字时，什么都不键入；这种情况，或一次没有以 exit 0 干净 `sent` 回执收场的发送——`composer_not_empty`、`typed_unsubmitted` / `composer_not_cleared`（Enter 之后文字仍在输入框上）、任何非零退出——是 `queued`（0），带 `queued`（那些 URL）、`why`、`underlying`（发送的行，若运行过）、标为 `delivered: false` 的行，以及 `next`：结束本轮；follow 线程每轮读它记录的 PR 清单，下一次 `tick`（唤醒循环的轮次也算）、`context` 或 `follow` 在输入框空闲时键入它（这些行上的 `follow_delivered` 点名被键入的 URL；带新 URL 的 `follow` 调用会连同新的一起键入排队者）；绝不自己去落地（一个动作；没有手动的 `send` 行、没有去盯线程的替代方案、也没有重跑的杂务——曾给过三个选项，一位协调者自己合并并推了 main）。当输入框判定来自一个屏幕显示空闲的线程——没有活动行、没有对话框——应答改为 `composer_stale`（6）：线程已收工，一条更早的未提交行停在它的输入框上，等待与重试都清不掉它（14 分钟里的三次 `follow` 调用都被答以"mid-turn"）——在任何发送之前就已停在空闲线程输入框上的一行是同一情形（它以 `queued` 读了 17 分钟而没有什么释放它；此时 `underlying` 是本次调用自己做出的 `composer_not_empty` 判定，行上的文字即那条行）；`next` 点名该行的前 60 个字符，说明 `host-manager send --type` 清不掉也盖不掉它，并点名先停后重开（`stop <slug> follow`，然后同样的 `follow`；重开线程的 brief 带着该 URL），并且再次强调绝不自己去落地。每次 `tick` 与 `context` 做的重试不在这样的行上再键入任何东西，并让其 `text` 与 `next` 以 `follow: idle behind an unsubmitted line ("<its first 60 characters>") that no round clears: stop <slug> follow, then follow <slug> --pr …` 开头。键入的轻推本身是 `new PR(s) queued in your record: <n>; read <record.json path>`——一个计数和记录，绝不是 URL：Muse 输入框不提交带 `file://` 记号的行（未送达交接的根源），而记录正是 brief 告诉线程每轮去读的清单。什么也不说的屏幕让发送自己作答。没有提交过的回执绝不应答 `delivered: true`。记录的 `cwd` 已不是目录的活 follow 线程是 `follow_thread_lost`（6）：什么都不键入进去，URL 记为 `delivered: false`，`next` 点名先停后重开；`tick` 与 `overview`/`context` 把这样的线程显示为 `lost — its recorded checkout <cwd> is gone`（WAKE 行也是）而不是 `running`（follow 线程删掉了自己的检出，两个 PR 在行写着 running 时被"送达"了进去）。干净的 `sent` 回执之后，辅助脚本把输入框重读一次（host-manager 的 `read --tail`）：粘贴的行还在那里也算 `queued`，`underlying` = `typed_unsubmitted`/`composer_not_cleared`（输入框还握着那行时 `sent` 就已应答，一次 991 秒的停摆）。其活会话被重开收养的没了记录（`adopted: true`）以同样方式得到其 brief 未带的 URL——收养本身绝不算送达（输入框不空闲时为带 `adopted: true` 的 `queued`）。follow 线程以 `inbox put --kind pr --key pr:<head>:<event>` 归档 PR 事件。没有 `--pr` 是 `usage`（2）。`done` 或 `stopped` 的 follow 线程原地重开——done 是按 PR 清单说的，对 follow 线程绝不是终态（`thread_done` 曾让协调者自己搭起落地线程）：同一记录、一个新会话，其 `evidence`、`attempts`、报告与 PR 行保留，追加新 URL，清空 `ended_at`/`session_ended_at`/`stop_receipt`/`identity_drift` 并盖上 `reopened_at`（`following`，0，`created: false`，`reopened: true`；brief 只列尚未合并的 PR）。提供者答不上来的运行中 follow 线程是 `follow_unknown`（6）——不重开也不记录，`next` 是 `tick <slug>`。没了的 follow 线程（`exited`、`orphaned`）作为全新记录重开（无报告、证据或结束戳；`created: true`），恰好携带旧记录未见合并的 PR 加上新 URL。`--cwd` 是 follow 线程工作的目录（默认：协调者第一个被记录的仓库 `state.json.repos[0]`；重开保留上一个记录的）。follow 线程每项目一个，且在 `max_parallel` 之外。

## `report` / 报告

```
report <slug> <thread-id> (--file PATH|-)
```

A thread's report, written as a whole (`threads/<id>/report.md`; the
brief's `Report:` line is `--file -`, the report on stdin — one shell call,
no file the thread writes itself, the helper keeps the copy — four attach-and-approve trips per project came from editor-tool writes of
that file). The helper
reads the first `PR: <url>` line, wherever the report puts it (a `PR:` line that is empty or says `none`, `n/a`, `-`, `no` or `nothing` is no PR: no row, no rung, no follow hint — `PR: none` became a PR row at rung 50), the last `STATUS:` line, the
last `BLOCKED(HUMAN):` line (a value of `none`, `n/a`, `-`, `no` or `nothing`,
any case, a trailing period allowed, or nothing at all is no block:
`blocked_line` is `null`) and the last `DECISIONS:` line — what the thread
decided that the brief did not fix: a version, a public name, files outside
its list (the same no-decision values, or no line at all, is `null`; a report
from before the line reads as before — where a
removal version and three extra modules passed through unquestioned) — files one inbox event (`kind: report`, key
`report:<thread>:<digest>`), adds a work thread's `PR:` URL to its record's
`prs` as an open row (the follow thread's `PR:` line adds no row: its rows
are the URLs it follows), and moves a `## Remember` section, when present,
to `threads/<id>/remember.md` — never into `MEMORY.md`. A report identical to
the last one is `reported` with `deduplicated: true`. `reported` (0) with
`digest`, `status_line`, `blocked_line`, `decisions_line`, `pr`,
`remember: true|false`, `self_reported` (the last `Progress: NN% — <basis>` line as `{percent, basis}`; no line, or one that does not parse — no percent, over 100 — is `null`, silently: the brief asks for it, the helper never invents it; it is recorded as said and shown only under the artifact rung, § The tracking ledger). `context`'s `next` for a pending report with a decision says `relay <id>'s DECISIONS to the user: …` before the `ack`. `no_such_thread` (3) for an unknown id.

一份线程的报告，整体写入（`threads/<id>/report.md`；brief 的 `Report:` 行是 `--file -`，报告走 stdin——一次 shell 调用，没有线程自己写的文件，副本由辅助脚本保管——每个项目四次 attach-审批往返来自用编辑器工具写那个文件）。辅助脚本读取第一条 `PR: <url>` 行，无论报告把它放在哪（空或写着 `none`、`n/a`、`-`、`no`、`nothing` 的 `PR:` 行不算 PR：没有行、没有档位、没有 follow 提示——`PR: none` 曾在档位 50 上变成一行 PR），最后一条 `STATUS:` 行，最后一条 `BLOCKED(HUMAN):` 行（值为 `none`、`n/a`、`-`、`no`、`nothing`——大小写不限、允许句尾句号——或干脆没有，都算没有阻塞：`blocked_line` 为 `null`）以及最后一条 `DECISIONS:` 行——线程自行决定而 brief 未曾固定的事：一个版本、一个公开名字、清单之外的文件（同样的无决定值，或干脆没有该行，为 `null`；该行出现之前的报告按之前的样子解读——曾有一个删除版本和三个额外模块无人质疑地通过了）——归档一条收件箱事件（`kind: report`，键 `report:<thread>:<digest>`），把工作线程的 `PR:` URL 作为开放行加进其记录的 `prs`（follow 线程的 `PR:` 行不加行：它的行就是它跟的 URL），并把 `## Remember` 节（如存在）移到 `threads/<id>/remember.md`——绝不进 `MEMORY.md`。与上一份相同的报告是 `reported` 且 `deduplicated: true`。`reported`（0），带 `digest`、`status_line`、`blocked_line`、`decisions_line`、`pr`、`remember: true|false`、`self_reported`（最后一条 `Progress: NN% — <basis>` 行，形如 `{percent, basis}`；没有该行、或一行解析不了——无百分比、超过 100——则默默为 `null`：brief 请求它，辅助脚本从不编造；它按所说的记录，且只在工件档位之下显示，§ The tracking ledger）。对一份带决定的待处理报告，`context` 的 `next` 会在 `ack` 之前说 `relay <id>'s DECISIONS to the user: …`。未知 id 是 `no_such_thread`（3）。

On the inbox path the verb, run from the thread's own shell, then sends the
report as one `agents-message/v1` session message to the coordinator's
session (`muse session-message send --json --target <its id>`, the body on
stdin; `MUSE_AGENTS_SESSION_SEND` overrides the command): first line `WAKE
<slug>: <name> reported: <text>` (the wake line's own words), then the schema
line and `project:`, `thread:`, `kind:`, `key:`, `text:`, `report:` lines, a
blank line, the report text verbatim. The line carries `message` —
`delivered`, `target`, `body`, the CLI's `receipt`, or `error`; `held: true`
when the target session holds it for its own admission (the CLI's
`pending`); `send_with_tool` (`{tool: send_session_message, target, body}`)
when the CLI refused it — today's runtime admits a session message from the
session's own model tool, not from a shell (`unverified_target_receipt`,
`causal_metadata_invalid`; #41210), so `next` then leads with `send
\`message.body\` to session <target> with your send_session_message tool`
and the thread, whose brief says the same, sends it itself; the helper's
own attempt is bounded to a few seconds, so the receipt always comes back
inside the thread's tool window. Any other undelivered send leads `next`
with `not delivered …`. Either way the file and
the inbox event stand, nothing is typed, nothing is retried. A deduplicated
report sends nothing.

在收件箱路径上，该动词从线程自己的 shell 运行，随后把报告作为一条 `agents-message/v1` 会话消息发送到协调者的会话（`muse session-message send --json --target <its id>`，正文走 stdin；`MUSE_AGENTS_SESSION_SEND` 可覆盖该命令）：第一行 `WAKE <slug>: <name> reported: <text>`（唤醒行自己的话），然后是 schema 行和 `project:`、`thread:`、`kind:`、`key:`、`text:`、`report:` 各行，一个空行，报告原文。该行携带 `message`——`delivered`、`target`、`body`、CLI 的 `receipt`，或 `error`；目标会话把消息扣下等自己的准入时为 `held: true`（CLI 的 `pending`）；CLI 拒绝时为 `send_with_tool`（`{tool: send_session_message, target, body}`）——今天的运行时只承认来自会话自己的模型工具的会话消息，不承认来自 shell 的（`unverified_target_receipt`、`causal_metadata_invalid`；#41210），所以此时 `next` 以 `send \`message.body\` to session <target> with your send_session_message tool` 开头，而 brief 里写着同样内容的线程自己发送；辅助脚本自己的尝试以几秒为界，因此回执总能在线程的工具窗口内返回。其他任何未送达的发送让 `next` 以 `not delivered …` 开头。无论哪种方式，文件与收件箱事件都成立，不键入任何东西，不重试任何东西。去重的报告什么都不发送。

## `ack` / 确认

```
ack <slug> <thread-id…> [--progress NN --basis TEXT]
```

The coordinator has read the current report: `acked_digest` becomes the
report's digest, so the thread leaves `ready-for-review` /
`waiting-on-you` until the report changes. `acked` (0); `no_report` (3) when
there is nothing to ack. Several ids like `stop` (`ack
<slug> a b c d` was `usage`, then `--help`, then a per-id loop): every id is
checked before any write — one without a report is `no_report` naming it and
nothing is acked — and the line answers `threads` (`digest` per id), `ref` =
`<slug>/<id,id,…>`; one id keeps its shape (`digest` at the top).

协调者已读过当前报告：`acked_digest` 变为该报告的 digest，于是线程离开 `ready-for-review` / `waiting-on-you`，直到报告再变。`acked`（0）；没有可 ack 的东西时为 `no_report`（3）。多个 id 与 `stop` 相同（`ack <slug> a b c d` 曾是 `usage`，然后是 `--help`，然后是一个逐 id 循环）：任何写入之前先检查每个 id——一个没有报告的是 `no_report` 并点名它，什么都不 ack——该行应答 `threads`（每个 id 的 `digest`），`ref` = `<slug>/<id,id,…>`；单个 id 保持其形状（`digest` 在顶层）。

The line carries what the coordinator says and does with the report: `decisions_line` (the report's, `null` without
one) and, when there is one, `text` = `Decisions: <line>` (several ids: one
line each, `Decisions: <id> — <line>`) with `next` opening `say this to the
user in this turn: …`; then `warnings: [{kind: `pr_without_follow`, thread,
text, next}]` for every work row with an unmerged PR that no running follow
thread carries, `next` = `follow <slug> --pr <url> — the landing is the
follow thread's, never yours; you run no git in any clone` (the follow
thread's own report, and a PR the follow thread already carries or has
merged, warn of nothing); `next` closes with the end-of-turn words. Facts,
never a refusal: the ack lands either way.

该行携带协调者对报告说的话与要做的事：`decisions_line`（报告的，没有则 `null`），以及有时一行 `text` = `Decisions: <line>`（多个 id：每人一行，`Decisions: <id> — <line>`），`next` 以 `say this to the user in this turn: …` 开头；然后对每个带着未合并 PR、且没有任何运行中的 follow 线程承载它的工作行，给 `warnings: [{kind: `pr_without_follow`, thread, text, next}]`，`next` = `follow <slug> --pr <url> — the landing is the follow thread's, never yours; you run no git in any clone`（follow 线程自己的报告，以及 follow 线程已承载或已合并的 PR，不警告任何东西）；`next` 以结束本轮的话收尾。是事实，不是拒绝：ack 无论如何都落定。

`ack <slug> <id> --progress NN --basis "<why>"` — never above the row's `artifact_rung`; a lower self-report may pull it down — also records the coordinator's judged value — one thread, a whole percent, a basis (what was verified, or the thread's own lower report) — as `calibrated` `{value, basis, rung, at}` and answers it; a value above the thread's artifact rung is `usage` (2) naming the cap and its basis (`--progress 80 is above the artifact rung 50 (PR open)`), and nothing is acked; several ids with `--progress`, or `--progress` without `--basis`, is `usage` too. The ack writes the thread's ledger point (§ The tracking ledger).

`ack <slug> <id> --progress NN --basis "<why>"`——绝不超过该行的 `artifact_rung`；更低的自报可以把它拉低——同时记录协调者判定的值——一个线程、整数百分比、一个 basis（验证过什么，或线程自己更低的报告）——为 `calibrated` `{value, basis, rung, at}` 并应答它；高于线程工件档位的值是 `usage`（2），点名上限及其 basis（`--progress 80 is above the artifact rung 50 (PR open)`），且什么都不 ack；多个 id 带 `--progress`，或 `--progress` 没带 `--basis`，也是 `usage`。这次 ack 写下线程的台账点（§ The tracking ledger）。

A few seconds after a typed steer (`host-manager send <ref> --type`), ONE
`read <ref> --tail`: say what the pane shows (took it / no reaction yet),
never leave it at `typed`; the outcome comes with the next wake.

键入的转向（`host-manager send <ref> --type`）之后几秒，做一次 `read <ref> --tail`：说出 pane 显示了什么（接了 / 还没反应），绝不停在 `typed`；结果随下一次唤醒到来。

## `remember` / 记忆

```
remember <slug> (--text TEXT | --from-thread ID… | --decision TEXT) [--heading H]
```

Appends one entry to `MEMORY.md` (`## <date> <heading>` and the text).
`--from-thread` takes `threads/<id>/remember.md` (the report's `## Remember`)
and removes it once appended; several ids take several sections in one
call, every one checked before any is appended (one
without a section is `nothing_to_remember`, 3, nothing written) — the
answer then carries `blocks` (`heading`, `deduplicated` per id),
`entries` and `receipts`; one id or `--text` keeps the one-block shape
below (plus `blocks`). The coordinator is the only writer: a call from
a thread — `MUSE_AGENTS_ROLE=thread` in the caller's environment — is
`not_coordinator` (3) and writes nothing; the thread's words stay in its
report for the coordinator to take. `remembered` (0) with `heading`,
`entries` (the new count) and `deduplicated`: a block byte-identical to one
`MEMORY.md` already holds under the same heading is not appended again
(`deduplicated: true`, `entries` unchanged; the thread's `remember.md` is
removed either way). Task state — issues, PRs,
`TASKS.md` — is never memory: `MEMORY.md` keeps what the project learned,
not what is open.

向 `MEMORY.md` 追加一条（`## <date> <heading>` 与文本）。`--from-thread` 取 `threads/<id>/remember.md`（报告的 `## Remember`）并在追加后移除它；多个 id 在一次调用中取多节，任何追加之前每一节都先检查（一个没有该节的是 `nothing_to_remember`，3，什么都不写）——此时应答带 `blocks`（每 id 的 `heading`、`deduplicated`）、`entries` 与 `receipts`；单个 id 或 `--text` 保持下方的一块形状（外加 `blocks`）。协调者是唯一写者：来自线程的调用——调用方环境里 `MUSE_AGENTS_ROLE=thread`——是 `not_coordinator`（3），什么都不写；线程的话留在它的报告里，供协调者取用。`remembered`（0），带 `heading`、`entries`（新增计数）与 `deduplicated`：与 `MEMORY.md` 在同一标题下已有内容逐字节相同的一块不再追加（`deduplicated: true`，`entries` 不变；线程的 `remember.md` 无论如何都移除）。任务状态——issue、PR、`TASKS.md`——绝不是记忆：`MEMORY.md` 保存项目学到的东西，不是尚在开放中的东西。

`--decision "<text>"` records one settled answer instead: the next `D<n>`
line in `PROJECT.md` § Decisions and in `library/DECISIONS.md` (the threads'
copy), numbered by the helper. `remembered` (0) with `decision` (the label)
and `decisions` (every line, in order; `context`'s `project.decisions` says
the same). It takes neither `--text` nor `--from-thread` in the same call.

`--decision "<text>"` 改为记录一条已敲定的答案：`PROJECT.md` § Decisions 与 `library/DECISIONS.md`（线程的副本）中的下一条 `D<n>` 行，由辅助脚本编号。`remembered`（0），带 `decision`（标签）与 `decisions`（每一行，按顺序；`context` 的 `project.decisions` 说的相同）。同一次调用里它既不接受 `--text` 也不接受 `--from-thread`。

A failed `host-manager send` is not a relay: do its receipt's `next`
(`--type`); still failing, say what was NOT delivered, never that it was
forwarded (moved here from SKILL.md step 6.2).

一次失败的 `host-manager send` 不是转发：执行其回执的 `next`（`--type`）；仍然失败，就说出什么"未"送达，绝不说"已转发"（从 SKILL.md 步骤 6.2 移到这里）。

## `relay` / 转发

```
relay <slug> <thread-id> [--fingerprint FP] (--asked | --answer TEXT) [--asked-by WHO]
```

The recorded relay of a `BLOCKED(HUMAN):` answer (#44029). A report question's
lifecycle is distinct in the ledger (§ The tracking ledger):
`blocked-unasked` → `asked-relay-owed` → `relayed-awaiting-worker` → closed
(on the worker's next report, as before). `ack` alone moves a question past
none of these states: it reads the report, it does not ask, answer or relay.

一条 `BLOCKED(HUMAN):` 回答的记录式转发（#44029）。报告问题在台账中的生命周期是独立的（§ The tracking ledger）：`blocked-unasked` → `asked-relay-owed` → `relayed-awaiting-worker` → 关闭（照旧，在 worker 的下一次报告上）。仅凭 `ack` 不会让问题越过这些状态中的任何一个：它读报告，不提问、不回答、不转发。

- `--asked` records that the coordinator put the question to the user once:
  `asked` (0) with `relay_state: asked-relay-owed` and `asked_at`; nothing is
  sent. `--asked` on a question already relayed is `already_relayed` (3).
  `--asked` 记录协调者已把问题问过用户一次：`asked`（0），带 `relay_state: asked-relay-owed` 与 `asked_at`；什么都不发送。对已转发过的问题 `--asked` 是 `already_relayed`（3）。
- `--answer TEXT` (`-` reads stdin) records the user's answer on the ledger
  entry first, then performs the send — host-manager
  `send <ref> --type --automated --text <answer>` on this machine,
  fleet-manager `send <addr> <text> --type --automated` on a machine — and
  records the receipt: `relay` = `{thread, fingerprint, answer, at, send:
  {outcome, delivery, receipt}}`. Delivered only on `deliver_to_follow`'s
  criterion: a clean `sent`/`typed` line at exit 0 WITH a receipt, and the
  submission confirmed — on this machine a post-send re-read whose composer
  no longer holds the answer (a swallowed Enter is not delivery), on a
  machine fleet-manager's own post-send verify: its `submitted` verdict
  (`submitted: false` is not delivery): `relayed` (0), `relay_state:
  relayed-awaiting-worker`; the question stays open until the worker's
  next report closes it. The whole check → record → send → record runs
  under the project lock, so two concurrent relays cannot both type it.
  `--answer TEXT`（`-` 读 stdin）先把用户的答案记在台账条目上，然后执行发送——本机为 host-manager `send <ref> --type --automated --text <answer>`，机器上为 fleet-manager `send <addr> <text> --type --automated`——并记录回执：`relay` = `{thread, fingerprint, answer, at, send: {outcome, delivery, receipt}}`。只有满足 `deliver_to_follow` 的判据才算送达：exit 0 的干净 `sent`/`typed` 行且带回执，且提交得到确认——本机为发送后重读且其输入框不再握着答案（被吞掉的 Enter 不是送达），机器上为 fleet-manager 自己的发送后验证：其 `submitted` 判定（`submitted: false` 不是送达）——`relayed`（0），`relay_state: relayed-awaiting-worker`；问题保持打开，直到 worker 的下一次报告关闭它。整个检查 → 记录 → 发送 → 记录都在项目锁下运行，因此两个并发的转发不可能都键入它。
- A failed send is `relay_failed` (6): the entry stays `asked-relay-owed`
  with the answer and the failed `send` recorded, `next` leads with what was
  NOT delivered and the same `relay … --answer` retry (the answer shell-quoted;
  no re-ask of the user is needed: their answer is on the entry), and
  `context`'s `next` and the WAKE line keep naming the relay as owed until
  it lands.
  失败的发送是 `relay_failed`（6）：条目保持 `asked-relay-owed`，答案与失败的 `send` 都已记录，`next` 以什么"未"送达与同样的 `relay … --answer` 重试开头（答案经 shell 引用；无需再问用户：他们的答案已在条目上），并且 `context` 的 `next` 与 WAKE 行会继续把这次转发点名为欠着，直到它落地。
- No open report question for the thread — or a `--fingerprint` that names
  none, or a `dialog:` fingerprint (a screen dialog is answered at its
  screen, never relayed) — is `no_open_question` (3) and sends nothing.
  Without `--fingerprint` the thread's one open report question is meant.
  该线程没有打开的报告问题——或一个什么也点不中的 `--fingerprint`，或一个 `dialog:` 指纹（屏幕对话框在它的屏幕上回答，绝不转发）——是 `no_open_question`（3），什么都不发送。不给 `--fingerprint` 时指线程唯一打开的报告问题。
- Only the recorded coordinator relays (`not_coordinator`, 3, as `accept`).
  只有被记录的协调者能转发（`not_coordinator`，3，与 `accept` 相同）。

While a relay is owed, `context`'s `next` names it (`relay owed for <id>
(<fingerprint>): …`) even when the report was acked and nothing else moved —
never `nothing moved; end the turn` — and `tick --wake-line` names
`<Name> relay owed: <question>` for a thread its report segment did not
already name. `needs_you` (in `context` and `overview`) and the status
table's `Needs you:` line carry the entry with its `relay_state`.

转发还欠着时，`context` 的 `next` 点名它（`relay owed for <id> (<fingerprint>): …`），即使报告已被 ack 且没有别的移动——绝不是 `nothing moved; end the turn`——而 `tick --wake-line` 对一个其报告段尚未点名过的线程点名 `<Name> relay owed: <question>`。`needs_you`（在 `context` 与 `overview` 中）与状态表的 `Needs you:` 行携带该条目及其 `relay_state`。

A session opened outside this helper — a raw `agentcloudctl` or
host-manager session with no thread record — has no pending question, no
wake and no relay: `relay` refuses it (`no_such_thread` /
`no_open_question`). Coordinators must not treat such a session as a
managed thread; open it through `propose`/`go` when its questions must be
asked and relayed.

一个在本辅助脚本之外开启的会话——一个没有线程记录的裸 `agentcloudctl` 或 host-manager 会话——没有待处理问题、没有唤醒、没有转发：`relay` 拒绝它（`no_such_thread` / `no_open_question`）。协调者绝不能把这样的会话当作受管线程；当它的问题必须被提问并转发时，通过 `propose`/`go` 开启它。

## `accept` / 验收

```
accept <slug> <thread-id…> --evidence TEXT… [--asked-by WHO]
```

Marks a thread `done` on evidence the coordinator verified — a merged PR
URL, a commit on the target branch, an artifact path, a test run. Several
ids take the one evidence line for all of them, like `stop`: every id is judged before any is written (an unknown id
is `no_such_thread`, a done or never-run one its refusal below, nothing
written), the answer carries `threads` (`evidence`, `decisions_line` per
id) and `receipts`, and `next` relays every thread's decision; one id
keeps the shape below (plus `threads`). Without
`--evidence` it is `usage` (2): a report that says "done" is a claim.
`accepted` (0) with `evidence` and `decisions_line` (the report's, `null`
without one); the receipt records who accepted and when, and `decisions`
when the thread decided something on its own — `text` is then `Decisions:
<line>` and `next` opens with `say this to the user in this turn: …`, and a
`pr_without_follow` warning with its `follow --pr` line follows when no
running follow thread carries the thread's unmerged PR (the same shape as
`ack`, above): the acceptance lands either way;
`next` is `every thread is done: archive <slug>, then end the turn` when
no thread is left in another status, else it names what is still open
before the archive (`<id> (<status>)`, the follow thread included) after
the end-of-turn words.
For a remote thread the answer also carries `copies` (FR-43932-5): each
accepted thread's record-copy disposition — `removed`, `tombstone-left`
(the copy was rewritten `status: accepted` before removal, and the
removal failed; the tombstone already ungates the pane), or
`tombstone-failed` (both writes failed; the copy keeps gating until
its stamp ages past 45 minutes, and the failure is named here, never
silent).
`accept` ends the accepted thread's session the way `stop` does (host-manager
`stop`, fleet-manager `close`; `stop_receipt` and `session_ended_at` on the
record; `session_ended` on each `threads` row and receipt, with
`session_ended_why` when `false` — `no live session`, a stranger's
session's `identity_drift`, or the provider's refusal); the clone, branch, record and evidence stay,
and `go <slug> <id>` reopens the thread in place (§ go; owner ruling 55,
#41777). A provider refusal never un-accepts: `accepted` (0) all the same,
`session_ended: false`, the refusal under that row's `stop_error`
(`outcome`, `error`, `next`), and `next` opens with `stop <slug> <id…>`.
A second `accept` on a done thread is `thread_done` (3) and appends
nothing (a reopened thread is accepted again, its evidence appended); a
thread that never ran (`proposed`) is `not_started` (3). The accept writes each thread's terminal ledger point — `accepted`, 100, the evidence beside it (§ The tracking ledger).

在协调者验证过的证据之上把线程标为 `done`——一个已合并的 PR URL、目标分支上的一个 commit、一个工件路径、一次测试运行。多个 id 共用同一条证据行，与 `stop` 相同：任何写入之前每个 id 都先判定（未知 id 是 `no_such_thread`，done 或从未运行的是其下文的拒绝，什么都不写），应答带 `threads`（每 id 的 `evidence`、`decisions_line`）与 `receipts`，`next` 转达每个线程的决定；单个 id 保持下方形状（外加 `threads`）。没有 `--evidence` 是 `usage`（2）：一份说"done"的报告只是一个声明。`accepted`（0），带 `evidence` 与 `decisions_line`（报告的，没有则 `null`）；回执记录谁在何时验收，以及线程自行决定过什么时的 `decisions`——此时 `text` 为 `Decisions: <line>`，`next` 以 `say this to the user in this turn: …` 开头，且当没有运行中的 follow 线程承载该线程未合并的 PR 时，跟着一条带其 `follow --pr` 行的 `pr_without_follow` 警告（与上文 `ack` 相同的形状）：验收无论如何都落定；当没有线程处于其他状态时 `next` 为 `every thread is done: archive <slug>, then end the turn`，否则在结束本轮的话之后点名归档前仍然开放的东西（`<id> (<status>)`，follow 线程算在内）。
对远程线程，应答还带 `copies`（FR-43932-5）：每个被验收线程的记录副本处置——`removed`、`tombstone-left`（移除前副本已被改写为 `status: accepted`，而移除失败；墓碑已解除该 pane 的门禁）或 `tombstone-failed`（两次写入都失败；副本继续门禁直到其时间戳老化超过 45 分钟，失败在此点名，绝不沉默）。
`accept` 以 `stop` 的方式结束被验收线程的会话（host-manager 的 `stop`，fleet-manager 的 `close`；记录上有 `stop_receipt` 与 `session_ended_at`；每条 `threads` 行与回执上有 `session_ended`，为 `false` 时带 `session_ended_why`——`no live session`、陌生人会话的 `identity_drift`、或提供者的拒绝）；克隆、分支、记录与证据保留，`go <slug> <id>` 原地重开该线程（§ go；业主裁决 55，#41777）。提供者的拒绝绝不撤销验收：照样 `accepted`（0）、`session_ended: false`，拒绝放在该行的 `stop_error` 之下（`outcome`、`error`、`next`），`next` 以 `stop <slug> <id…>` 开头。对 done 线程的第二次 `accept` 是 `thread_done`（3）且不追加任何东西（重开的线程可以再次被验收，其证据被追加）；从未运行过的线程（`proposed`）是 `not_started`（3）。这次验收写下每个线程的终局台账点——`accepted`，100，旁边是证据（§ The tracking ledger）。

## `inbox` / 收件箱

```
inbox put <slug> --kind KIND --key KEY [--thread ID] [--text TEXT] [--json JSON]
inbox put <slug> --message (PATH|-)
inbox drain <slug> [--ids ID…]
```

`put` files one event under its key (`filed`, 0, `created: true`) or answers
`duplicate` (0, `deduplicated: true`) when the key was seen — in `new/` or
`done/`. `--thread` is resolved before anything is written:
`no_such_thread` (3) for an unknown `--thread` files no event. A `pr` event with `--thread` updates that record's `prs[]`
(`last_event`, `head`). Your own `context` already reads (drains) the events it returned, so a wake
needs no `drain` call of its own; a stranger's `context`
reads nothing, as its own cursor already promises. `drain` stays for the
events you want cleared by hand and for a caller that is not the recorded
coordinator. `drain` returns the pending events (all, or the named
ids) and moves them to `inbox/done/` — the coordinator's "processed"
(`drained`, 0, `events`). An empty inbox is `drained` with `events: []`.
The line also carries `unacked_reports` (thread ids whose newest report
is neither acked nor accepted, newest first) and, when there are any,
`next` opens `<n> report(s) unread — <Name> [<id>], …: read
threads/<id>/report.md, then ack …` — a drain empties the inbox, not your
reading; an unread report stays on the WAKE line (four reports drained
together, three acted on, the fourth never named again: #38715).

`put` 在其键下归档一个事件（`filed`，0，`created: true`），或当该键已被见过——在 `new/` 或 `done/` 中——时应答 `duplicate`（0，`deduplicated: true`）。`--thread` 在写任何东西之前解析：未知的 `--thread` 是 `no_such_thread`（3），不归档任何事件。带 `--thread` 的 `pr` 事件更新该记录的 `prs[]`（`last_event`、`head`）。你自己的 `context` 已经读取（排空）它返回的事件，因此唤醒不需要自己的 `drain` 调用；陌生人的 `context` 什么都不读，正如它自己的游标已承诺的那样。`drain` 留给你想手工清掉的事件，以及不是被记录协调者的调用方。`drain` 返回待处理事件（全部，或点名的 id）并把它们移到 `inbox/done/`——协调者的"已处理"（`drained`，0，`events`）。空收件箱是带 `events: []` 的 `drained`。该行还携带 `unacked_reports`（最新报告既未被 ack 也未被 accept 的线程 id，最新在前），且当有时，`next` 以 `<n> report(s) unread — <Name> [<id>], …: read threads/<id>/report.md, then ack …` 开头——排空的是收件箱，不是你的阅读；一份未读报告留在 WAKE 行上（四份报告一起被排空，三份被处理，第四份从此再未被点名：#38715）。

`--message` files a message that arrived by session delivery (ADR 41038 D1:
a thread with no folder here) under the message's own `key:`: `filed` (0;
for a `report` the text after the header is written to
`threads/<id>/report.md` and read the way `report` reads it, `digest` on the
line) or `duplicate` (0, `deduplicated: true`) when the key was seen — the
same copy delivered twice files once. A body that is not an
`agents-message/v1` message, or one for another project, is `usage` (2); a
`thread:` the project lacks is `no_such_thread` (3); nothing is written
either way. On the inbox path a `put --kind pr` whose key ends `:merged`
also sends the event to the coordinator's session (`message` on the line);
every other pr event is the follow thread's own and wakes nobody.

`--message` 归档一条经会话投递到来的消息（ADR 41038 D1：一个在此没有文件夹的线程），在该消息自己的 `key:` 之下：`filed`（0；对 `report`，头部之后的文本写入 `threads/<id>/report.md` 并按 `report` 的方式读取，行上给 `digest`），或当该键已被见过时 `duplicate`（0，`deduplicated: true`）——同一份被投递两次的副本只归档一次。不是 `agents-message/v1` 消息的正文、或属于另一项目的消息是 `usage`（2）；项目没有的 `thread:` 是 `no_such_thread`（3）；两种情况都不写任何东西。在收件箱路径上，键以 `:merged` 结尾的 `put --kind pr` 还会把该事件发送到协调者的会话（行上的 `message`）；其他每个 pr 事件都是 follow 线程自己的，不唤醒任何人。

## `tick` / 节拍

```
tick <slug> [--arm monitor|scheduler|passive [--command CMD] [--one-shot] [--monitor-failed LINE]] [--disarm] [--asked-by WHO]
```

The coordinator's cadence mechanics, safe to run from a timer:

协调者的节拍机制，可以从定时器安全运行：

- refreshes every live remote thread's worker-gate copy stamp
  (FR-43932-5: `coordinator_seen_at` on this round's 30-second cadence,
  update-in-place through the copy channel — a live coordinator keeps
  its workers gated by cadence, not attention; a dead machine's copies
  simply age, and the tick never stalls on one);
  刷新每个活的远程线程的 worker 门副本时间戳（FR-43932-5：本轮的 `coordinator_seen_at`，以 30 秒的节奏经复制通道原地更新——活的协调者以节奏而非注意力让它的 worker 保持被门控；一台死机器上的副本只是老化，tick 绝不在其上停摆）；
- refreshes every open thread's liveness through its provider; a thread that
  is gone after a report becomes `exited`, gone or mismatched without a
  report becomes `orphaned`, and each change files one event
  (`thread:<id>:<status>:<opened_at>`), so a repeated tick files nothing new.
  A remote session fleet-manager no longer lists (`no_such_session` from
  `status` on a reachable machine) is gone; one whose live identity (`cwd`,
  `engine`, …) no longer matches the record is a stranger under the
  thread's name — gone for this thread (`identity_drift` on the record),
  never stopped; a machine that cannot answer leaves the thread unknown;
  经其提供者刷新每个开放线程的存活；一份报告之后消失的线程变为 `exited`，没有报告而消失或不匹配的变为 `orphaned`，每次变化归档一个事件（`thread:<id>:<status>:<opened_at>`），因此重复的 tick 不归档任何新东西。fleet-manager 不再列出的远程会话（可达机器上 `status` 的 `no_such_session`）就是没了；活身份（`cwd`、`engine`，…）不再匹配记录的是一个顶着线程名字的陌生人——对本线程即没了（记录上的 `identity_drift`），绝不被停止；一台答不上来的机器让线程保持未知；
- reads each live local thread's verdict the way `context` does (Herdr's own
  status, or host-manager's `read --tail` judgment plus the activity mark)
  and files one event when a thread turns waiting-on-you
  (`thread:<id>:waiting-on-you:<stamp>`, text with its attach command), so
  the watched `text` changes once for a dialog or permission prompt too;
  `last_agent_status` on the record keeps it to once per transition; and
  one when a live thread has read `idle` (an empty composer, nothing
  running; Herdr's or the msp provider's own `idle`) on two rounds in a row
  with no report the coordinator still has to read — none filed, or its
  newest acked and not done — `thread:<id>:idle:<spell>` (the record's count of idle spells), text `<id> went idle without a report — a question may be waiting on its screen; read it once` with its attach command, once per idle spell (`idle_rounds` on the record; one idle round is a fresh TUI's empty composer, not news). A
  question typed on a thread's screen, or a thread that stopped without
  reporting, was silence before this (AUDIT-AGENTS-WAKE, #38715: four
  minutes of `--wake-line` printed nothing while the question sat there);
  以 `context` 的方式读取每个活本地线程的判定（Herdr 自己的状态，或 host-manager 的 `read --tail` 判断加活动标记），并在线程转为 waiting-on-you 时归档一个事件（`thread:<id>:waiting-on-you:<stamp>`，text 带其 attach 命令），于是被监视的 `text` 对一个对话框或权限提示也只变化一次；记录上的 `last_agent_status` 把它限制为每次转变一次；以及当一个活线程连续两轮读作 `idle`（空输入框，无物运行；Herdr 或 msp 提供者自己的 `idle`）且没有协调者仍需阅读的报告——从未归档过、或其最新已 ack 且未 done——时归档一个：`thread:<id>:idle:<spell>`（记录的空闲段计数），text 为 `<id> went idle without a report — a question may be waiting on its screen; read it once` 并带其 attach 命令，每个空闲段一次（记录上的 `idle_rounds`；一个空闲轮只是新 TUI 的空输入框，不是新闻）。一个键到线程屏幕上的问题、或一个没报告就停下的线程，在此之前就是沉默（AUDIT-AGENTS-WAKE，#38715：四分钟的 `--wake-line` 什么也没打印，而问题就立在那里）；
- writes the tracking ledger (§ The tracking ledger): one point per running thread from what this round already holds (the record, the liveness verdict, the report, the PR events), a transition point for a thread whose status changed, the workstreams and the pending questions synced; the wake loop's `--wake-line` tick is a round too. A ledger that cannot be read is `tracking_error` on the line and a `tracking: …` progress line; the tick's own work still lands;
  写跟踪台账（§ The tracking ledger）：每个运行中的线程从本轮已握有的东西（记录、存活判定、报告、PR 事件）得一个点，状态变化的线程得一个转变点，工作线与待处理问题同步；唤醒循环的 `--wake-line` tick 也是一轮。读不了的台账是该行上的 `tracking_error` 与一行 `tracking: …` progress 行；tick 自己的工作仍然落定；
- every round, `--wake-line` included, types a PR queued on the follow record once that thread's composer is free (see `follow`; `follow_delivered` on the JSON line, nothing on the wake line: a delivery is no news);
  每一轮（`--wake-line` 也算）在该线程输入框空闲时把一个排队的 PR 键入 follow 记录（见 `follow`；JSON 行上有 `follow_delivered`，唤醒行上没有：送达不是新闻）；
- `--wake-line`: the wake loop's mode (`library/wake.sh` runs it every 30 s);
  each pending event renders as `<name> reported: <text>` (a pr event
  `<name> pr <event>: <text>`), the name from the thread's record and the
  key kept in the JSON `inbox` — the JSON tick's `text` rows read
  `  - <name> [<id>] reported: <text>  (<key>)` — no
  JSON; one line `WAKE <slug>: <segment>[; <segment>…]`, the event segments
  newest first whatever their kind — `<name> reported: <text>`, `<name> pr
  <event>: <text>`, `<name> thread: <text>` (a thread that turned
  waiting-on-you) — then `unknown: <ids>` and `<name> lost: …`, and one
  report per thread, its newest (an older unread report of the same thread
  stays in the JSON `inbox`, off the line: the first digest led every WAKE
  while the later reports were the news, past the
  ellipsis at 80 columns; the JSON `tick` is not curated),
  when there is news, nothing otherwise, identical while the news is, so the
  loop's last-line guard prints it once and again when the same news returns
  after a quiet spell; the records are news too: every thread whose newest
  report is neither acked nor accepted (`ready-for-review`, `waiting-on-you`)
  is one `<name> reported: <text>` segment on every tick until `ack`/`accept`,
  whether or not its event is still in the inbox (an `accept` shrinks the
  line, so the loop wakes you again for what is left — a drained, unread
  report is never silence); one segment per thread: its `reported` words win
  over its `moved` watch events (a ready-for-review or blocked transition
  repeats that report), else only its newest `moved` shows, and `pr` events
  stay; the line holds `WAKE_LINE_BYTES` (500) — whole segments newest
  first, then `+N more unread` for the ones that did not fit, so a thread is
  never silently dropped past the note's fold (the JSON `tick` keeps every
  event whole); a tick the loop cannot run at all (exit 126/127) is one
  WAKE line saying so; on an archived or unknown project it prints nothing and exits 0 (a loop installed before the
  folder went away must not print the refusal as news); a `pr` event the follow thread filed about a PR it
  follows is not news until it says `merged` — checks, reviews, conflicts
  and the queue are the follow thread's own to act on (three wakes in five
  minutes on an unread count), and the JSON
  `tick` still lists it. `--arm`/`--disarm` with it is `usage`;
  `--wake-line`：唤醒循环的模式（`library/wake.sh` 每 30 秒运行它）；每个待处理事件渲染为 `<name> reported: <text>`（pr 事件为 `<name> pr <event>: <text>`），名字来自线程的记录，键保存在 JSON `inbox` 中——JSON tick 的 `text` 行读作 `  - <name> [<id>] reported: <text>  (<key>)`——唤醒行上没有 JSON；一行 `WAKE <slug>: <segment>[; <segment>…]`，事件段最新在前、不分种类——`<name> reported: <text>`、`<name> pr <event>: <text>`、`<name> thread: <text>`（一个转为 waiting-on-you 的线程）——然后是 `unknown: <ids>` 与 `<name> lost: …`，且每线程只一份报告、取其最新（同一线程更早的未读报告留在 JSON `inbox` 里、不上行：第一个 digest 曾引领每一次 WAKE 而后来的报告才是新闻，还在 80 列的省略号之外；JSON `tick` 不做策展）——有新闻时输出，否则什么都不输出，新闻不变时输出不变，于是循环的末行守卫打印它一次，并在安静间歇后同一新闻回来时再打印一次；记录也是新闻：每个其最新报告既未被 ack 也未被 accept（`ready-for-review`、`waiting-on-you`）的线程在每次 tick 上都是一个 `<name> reported: <text>` 段，直到 `ack`/`accept`，无论其事件是否还在收件箱里（一次 `accept` 会缩短该行，于是循环为剩下的再唤醒你——被排空而未读的报告绝不是沉默）；每线程一个段：其 `reported` 的话压过其 `moved` 监视事件（ready-for-review 或阻塞转变会重复那份报告），否则只显示其最新的 `moved`，而 `pr` 事件保留；该行有 `WAKE_LINE_BYTES`（500）——整段最新在前，然后对放不下的加 `+N more unread`，于是线程绝不会被无声地丢过备注的折叠处（JSON `tick` 把每个事件完整保留）；循环完全跑不动的 tick（exit 126/127）是一行说明此事的 WAKE 行；对已归档或未知的项目它什么都不打印并 exit 0（一个在文件夹消失前安装的循环绝不能把拒绝当新闻打印）；follow 线程为其所跟 PR 归档的 `pr` 事件在说出 `merged` 之前不是新闻——检查、评审、冲突与队列是 follow 线程自己要处理的（五分钟里因未读计数而来的三次唤醒），而 JSON `tick` 仍然列出它。带它的 `--arm`/`--disarm` 是 `usage`；
- `--arm monitor` with a command that does not run the project's own
  `library/wake.sh` records the arm and adds one `warnings` entry
  (`not_the_ready_line`: it may never wake you, with the ready line as its
  `next`). The helper runs no simulation of the line: the ready line is
  printed by `go`, `context`, `tick` and the script's own header, and
  judging a hand-written filter is yours;
  `--arm monitor` 带一条并不运行项目自己的 `library/wake.sh` 的命令时，记录布防并加一条 `warnings` 条目（`not_the_ready_line`：它可能永远不会唤醒你，就绪行是其 `next`）。辅助脚本不对该行做任何模拟：就绪行由 `go`、`context`、`tick` 和脚本自己的头部打印，判断一个手写的过滤器是你的事；
- `library/wake.sh` speaks for one loop per project: it takes `library/wake.lock/pid`, a second loop on the same
  project (a second Monitor installed) stays silent and takes over only when the pid there is gone, and once the
  project folder is gone (archived) every loop on it, speaking or silent, exits 0 within one tick, so the Monitor
  ends by itself (owner ruling 26, #38715; supersedes the stay-alive loop:
  three Monitors meant every WAKE three times). `sh wake.sh --once` runs one
  round by hand. `--arm monitor` while the recorded arm is a monitor whose loop pid is alive is `already_armed` (0,
  nothing re-recorded) with `wake` (the standing arm), `loop_pid`, and `next` saying to leave the second Monitor alone
  — its loop is silent on its own, and a stop is a note that costs a turn; the arm records `loop_pid` when the loop
  has started;
  `library/wake.sh` 每个项目只为一个循环代言：它占用 `library/wake.lock/pid`，同一项目上的第二个循环（安装的第二个 Monitor）保持沉默，只有当那里的 pid 没了才接管，而一旦项目文件夹没了（已归档），其上每个循环、说话的或沉默的，都在一个 tick 内 exit 0，于是 Monitor 自行结束（业主裁决 26，#38715；取代常驻循环：三个 Monitor 意味着每次 WAKE 来三次）。`sh wake.sh --once` 手工跑一轮。当被记录的布防是一个循环 pid 还活着的 monitor 时，`--arm monitor` 是 `already_armed`（0，不重录），带 `wake`（现行的布防）、`loop_pid`，以及一条说别去动第二个 Monitor 的 `next`——它的循环自己就沉默，停它是一句要花一轮的备注；循环已启动时布防记录 `loop_pid`；
- `--arm` records who wakes this project: `tier`, `command` (for `monitor`,
  the Monitor line the coordinator installed; for `scheduler`, the unit or
  cron line), `persistent`, `armed_by` (this coordinator), `armed_at`;
  `--disarm` clears it. Recording is not scheduling — the coordinator
  installs the Monitor or the scheduler entry itself and records it here so
  `context` and `resume` can say whether a wake exists and whose it is;
  `--arm monitor` or `--arm scheduler` without `--command` is `usage` (2,
  nothing recorded) — an arm with no entry behind it wakes nobody; `--arm
  passive` without `--monitor-failed "<one line copied from the failed
  monitor( result>"` is `usage` (2, nothing recorded; `error` names the
  Monitor tool and the ready line, `next` says to call it, then record
  `--arm monitor`): passive is never the first try — the Monitor is what
  wakes you, and a passive arm with no call tried left every report unread
  (the owner's own run; ruling 52, #38715).
  With the line in hand it records `wake.monitor_failed` beside the tier. A
  monitor is persistent by default: a `--command` line that says
  `persistent=false` is `arm_not_persistent` (3, nothing recorded) unless
  `--one-shot` says one wake is all that is wanted — a timed monitor ends
  after its window, and re-arming it with the same cursor replays every
  line the source kept since; a replayed line carries the key it had the
  first time, so the inbox files it once (`duplicate`) and no `go` is
  read twice; the persistence rule reads only the line's own `persistent=`,
  never quoted inner text;
  `--arm` 记录谁唤醒这个项目：`tier`、`command`（`monitor` 为协调者安装的 Monitor 行；`scheduler` 为 unit 或 cron 行）、`persistent`、`armed_by`（本协调者）、`armed_at`；`--disarm` 清除它。记录不是排程——协调者自己安装 Monitor 或调度器条目，并在这里记录它，使 `context` 与 `resume` 能说出唤醒是否存在、是谁的；没有 `--command` 的 `--arm monitor` 或 `--arm scheduler` 是 `usage`（2，什么都不记录）——背后没有条目的布防谁也唤不醒；没有 `--monitor-failed "<从失败的 monitor( 结果复制的一行>"` 的 `--arm passive` 是 `usage`（2，什么都不记录；`error` 点名 Monitor 工具与就绪行，`next` 说先调用它，然后记录 `--arm monitor`）：passive 绝不是第一次尝试——Monitor 才是唤醒你的东西，而一个没试过调用的 passive 布防曾让所有报告无人读（业主自己的运行；裁决 52，#38715）。手里有那行时，它在 tier 旁记录 `wake.monitor_failed`。monitor 默认持久：写着 `persistent=false` 的 `--command` 行是 `arm_not_persistent`（3，什么都不记录），除非 `--one-shot` 说明只想要一次唤醒——定时 monitor 在其窗口后结束，用同一游标重新布防会重放源保留的每一行；重放的行带着它第一次的键，于是收件箱只归档一次（`duplicate`），也没有哪个 `go` 被读两次；持久性规则只读该行自己的 `persistent=`，绝不读引号内的文本；
- `ticked` (0) with `changes` (per thread: from, to), `inbox_pending`,
  `inbox` (every unread event: key, kind, thread, text), `text` (the unread
  events first, then the changes, then the wake line — the line a Monitor
  watches: it changes exactly when the inbox or a thread does, whoever ran
  the previous tick, so a thread's report is a change it sees once) and
  `wake`; `next` is `inbox drain <slug>` while anything is unread; a
  `receipt` (`what: arm|disarm`) when the arm changed;
  `ticked`（0），带 `changes`（每线程：从、到）、`inbox_pending`、`inbox`（每个未读事件：key、kind、thread、text）、`text`（先未读事件，再变化，然后是唤醒行——Monitor 监视的那一行：恰在收件箱或某线程变化时它才变化，无论上一轮 tick 是谁跑的，于是线程的报告是它只看一次的变化）与 `wake`；还有未读时 `next` 为 `inbox drain <slug>`；布防变化时有一条 `receipt`（`what: arm|disarm`）；
- the arm also records `means` (what the tier does for the coordinator
  session: a Monitor wakes it; a scheduler runs `tick` in another process —
  the folder refreshes, the session is not woken and speaks on the user's
  next message; passive runs nothing), `env` (the `MUSE_AGENTS_TMUX`,
  `TMUX_TMPDIR` and `MUSE_PROJECTS_HOME` the arm ran under) and
  `tick_command` (the line a scheduler should run: that environment in
  front of `python3 agents.py tick <slug>`), and answers `tick_command` on
  the line. A later verb whose caller sets no `MUSE_AGENTS_TMUX` /
  `TMUX_TMPDIR` fills them from the arm with a `progress` line, so a tick
  from a bare scheduler environment probes the same tmux server instead of
  an empty directory; a caller that names its own tmux is respected. A
  tmux that cannot answer leaves the thread unknown — never gone: so does a
  `status` answer that is `unreachable`, carries a tmux connect error, has
  no boolean `live`, is not a `status` line, carries an `error` beside its
  verdict (a listing the helper could not read, or
  a non-JSON answer; only a positive, parseable not-live verdict from a
  reachable provider moves a thread), or answers — live or not — from a tmux server other
  than the one the record names (a bare environment with no
  `MUSE_AGENTS_TMUX` and no arm to fill it from probes the default server;
  whatever that server holds, a same-named stranger included, the thread's
  own server was never asked and nothing of the stranger is read). Such
  threads are listed under `unknowns` with `unknown_reasons`, nothing is
  recorded or filed, and `text` carries one `unknown:` line naming the tick
  command that reaches their server (the arm's `tick_command`, else one
  composed from the recorded server whenever this environment's
  `MUSE_AGENTS_TMUX` is absent or names another server — never a repeat of
  the probe that just failed);
  布防还记录 `means`（该层级为协调者会话做什么：Monitor 唤醒它；scheduler 在另一个进程里跑 `tick`——文件夹刷新，会话不被唤醒，在用户下一条消息时开口；passive 什么都不跑）、`env`（布防运行时的 `MUSE_AGENTS_TMUX`、`TMUX_TMPDIR` 与 `MUSE_PROJECTS_HOME`）与 `tick_command`（调度器应跑的行：那个环境放在 `python3 agents.py tick <slug>` 前面），并在行上应答 `tick_command`。后续动词的调用方没设 `MUSE_AGENTS_TMUX` / `TMUX_TMPDIR` 时，会从布防补齐它们并附一条 `progress` 行，于是来自光杆调度器环境的 tick 探测的是同一台 tmux 服务器而不是一个空目录；点名了自己 tmux 的调用方被尊重。一台答不上来的 tmux 让线程保持未知——绝不是没了：`unreachable` 的 `status` 应答、带 tmux 连接错误、没有布尔 `live`、不是 `status` 行、判定旁带着 `error`（辅助脚本读不了的列表，或非 JSON 应答；只有来自可达提供者的、可解析的明确 not-live 判定才移动线程）、或——无论活没活——从记录点名之外的另一台 tmux 服务器应答，都一样（没有 `MUSE_AGENTS_TMUX` 也没有布防可补的光杆环境探测默认服务器；那台服务器上有什么——同名的陌生人也在内——线程自己的服务器从未被问过，陌生人的任何东西都不读）。这样的线程列在 `unknowns` 之下并带 `unknown_reasons`，不记录也不归档任何东西，`text` 带一行 `unknown:` 点名能到达它们服务器的 tick 命令（布防的 `tick_command`，否则在本环境的 `MUSE_AGENTS_TMUX` 缺席或指向另一台服务器时、从被记录服务器拼出的一条——绝不是把刚失败的那次探测再来一遍）；
- `--arm` and `--disarm` run only from the recorded coordinator
  (`not_coordinator`, 3, nothing recorded); a plain `tick` is anyone's.
  `--arm` 与 `--disarm` 只能从被记录的协调者运行（`not_coordinator`，3，什么都不记录）；光杆的 `tick` 谁都可以跑。

The same missing-loop fact rides on every `tick` that is not the loop's own
`--wake-line`: `text` leads with `wake: monitor armed on record, but no loop
is running` and `next` is `call monitor(<the recorded command>) now, then
end the turn` (an arm made this call is inside the grace window). The
earlier `try monitor(<the ready line>) first; passive only after a failed
call` hint on an early passive arm is retired: the arm refuses instead
(above), the failed call's line in hand.

同样的缺循环事实搭在每一次不是循环自己 `--wake-line` 的 `tick` 上：`text` 以 `wake: monitor armed on record, but no loop is running` 开头，`next` 为 `call monitor(<被记录的命令>) now, then end the turn`（本次调用所做的布防在宽限窗口之内）。更早的"先试 `monitor(<就绪行>)`；只在调用失败后才 passive"提示已在早期 passive 布防上退役：布防改为直接拒绝（如上），手里握着失败调用的行。

## `overview` / 总览

"status?" → `overview <slug> --table`, shown as is, never as bullets; a
`flat` row → ONE `host-manager read <name> --tail`, said in that thread's
line, no scanning on other wakes (moved here from SKILL.md
step 6, with the `ack` cap and the `init` message rules).
Deep read on a trigger only — quiet past `stuck_scans`, a doubted
self-report, the user asks, before `accept`: `read <ref> --lines 200`
once, never every wake (moved here from coordinator.md § On a wake for
the went-idle bullet's byte room).

"status?" → `overview <slug> --table`，按原样展示，绝不变为要点；一行 `flat` → 一次 `host-manager read <name> --tail`，在该线程的行里说明，其他唤醒上不再扫描（从 SKILL.md 步骤 6 移到这里，连同 `ack` 上限与 `init` 消息规则）。深读只在触发条件下——安静超过 `stuck_scans`、一次存疑的自报、用户发问、`accept` 之前：`read <ref> --lines 200` 一次，绝不是每次唤醒（从 coordinator.md § On a wake 移到这里，为 went-idle 条目腾字节）。

```
overview <slug> [--table]
```

The groups as text, one line per thread under its group heading, in the
order `waiting-on-you`, `ready-for-review`, `working`, `landing`, `idle`,
`orphaned`, `done`, `proposed`, then `unknown` (threads whose provider did
not answer: machine, reason, last known status), the inbox count and the
wake arm. `overview` (0) with `groups`, `unknowns`, `text`, `table` (the status table, § The tracking ledger) and `needs_you`; `overview <slug> --table` makes `text` the table — the answer to "status?", read verbatim. No HTML view exists.

以文本呈现各分组，每组标题之下每线程一行，顺序为 `waiting-on-you`、`ready-for-review`、`working`、`landing`、`idle`、`orphaned`、`done`、`proposed`，然后 `unknown`（提供者未应答的线程：机器、原因、最后已知状态）、收件箱计数与唤醒布防。`overview`（0），带 `groups`、`unknowns`、`text`、`table`（状态表，§ The tracking ledger）与 `needs_you`；`overview <slug> --table` 让 `text` 成为那张表——"status?" 的答案，按原文读出。不存在 HTML 视图。

## `pick` / 选择

```
pick <slug> --question "<q>" --option "<label>=<path>" [--option "<label>=<path>" …]
```

A pick between texts (two advocate notes, candidates, a `DECISIONS:` line),
printed ready to post: the message part is ONE lead-in line naming the
labels ("Two texts to choose between — A, B — each in full in its option's
preview; pick in the dialog.", plus the file paths when a preview is cut) —
the dialog is the carrier (six rounds of prose left the
message a bare lead-in in 2/3 runs while the previews carried every text
3/3) — then the line `--- dialog ---` and one JSON object — `ask` (the `request_user_input` arguments object: one
question, cut to 500 characters, the ask tool's cap, with its `options`) and `next`, nothing twice. Each option carries its `label` (cut to 60
characters) and, unless the file is blank, its text twice in the ask tool's
own per-option fields: `description`, the text as one line cut to 240
characters, and `preview` (`format: markdown`, `content` the text cut to
2000 characters); a cut description ends in `… (full text in the preview)`,
a cut preview in `… (cut at the dialog's cap)` — never "in the message
above": the model skipped that message in three rounds.
Post the lead-in line, then call `request_user_input` with the `ask`
object below the separator as its arguments, exactly as printed —
`{"questions": [{"id": "pick", "header": "Pick", "question", "options"}]}`,
an object whose `questions` is an array, never a JSON string (the model rebuilt the call, passed `questions` as a string,
and the retry dropped the previews) — descriptions and previews never
reworded: the user reads each text in the
dialog, never labels alone and never the texts after the choice (#41037:
the pick was made blind while the texts sat outside the dialog; #41227
and the model ran the verb and posted a lead-in with nothing under
it, so the texts moved into the dialog for good). A relative `<path>` is under the
project folder (`library/a.md`); an absolute path is taken as given. Two or
three options — the ask tool's cap per question (#41234); a fourth is
`usage` (2) naming the limit: ask in two rounds. Nothing else refuses: a file that
cannot be read prints `(missing: <path>)` in its option with a `warning:`
line on stderr, and the verb exits 0 either way. Not the JSON envelope:
stdout is the message, the separator, the JSON; a slug with no project
folder still works for absolute paths.

在几段文本之间做选择（两份倡导备注、几个候选、一条 `DECISIONS:` 行），打印成可立即发布的样子：消息部分是一行引入语，点名各标签（"Two texts to choose between — A, B — each in full in its option's preview; pick in the dialog."，预览被截时附上文件路径）——对话框才是载体（六轮散文里 2/3 的运行把消息变成了一句光杆引入语，而预览 3/3 地带上了每段文本）——然后是 `--- dialog ---` 行和一个 JSON 对象——`ask`（`request_user_input` 的参数对象：一个问题，截到 500 字符——ask 工具的上限——带其 `options`）与 `next`，没有东西重复两遍。每个选项带其 `label`（截到 60 字符），并且除非文件为空，在 ask 工具自己的按选项字段里把文本带两遍：`description`（文本作为一行截到 240 字符）与 `preview`（`format: markdown`，`content` 为截到 2000 字符的文本）；被截的 description 以 `… (full text in the preview)` 收尾，被截的 preview 以 `… (cut at the dialog's cap)` 收尾——绝不说"见上面的消息"：模型曾三轮跳过那条消息。发布引入行，然后以分隔符之下的 `ask` 对象为参数调用 `request_user_input`，与打印的一字不差——`{"questions": [{"id": "pick", "header": "Pick", "question", "options"}]}`，一个 `questions` 为数组的对象，绝不是 JSON 字符串（模型曾重建调用、把 `questions` 作为字符串传入，重试把预览丢了）——description 与 preview 绝不换措辞：用户在对话框里读每段文本，绝不是只读标签，也绝不是选完才读文本（#41037：文本在对话框外面时选择是盲选；#41227 模型跑了动词、只发布了一句下面什么都没有的引入语，于是文本被永久移进了对话框）。相对的 `<path>` 在项目文件夹之下（`library/a.md`）；绝对路径按原样接受。两个或三个选项——ask 工具每问题的上限（#41234）；第四个是 `usage`（2），点名该限制：分两轮问。其他都不拒绝：读不了的文件在其选项里打印 `(missing: <path>)` 并在 stderr 上给一行 `warning:`，动词无论如何都 exit 0。不是 JSON 信封：stdout 就是消息、分隔符、JSON；没有项目文件夹的 slug 对绝对路径仍然可用。

## `set` / 设置

```
set <slug> mode herdr|tmux|msp|auto [--asked-by WHO]
set <slug> engine muse|claude|codex|auto
set <slug> every 2m|90s|quiet|auto
set <slug> heartbeat 5|1h|off
set <slug> sink "<command>"|none
```

`mode`: the project's mode pin (ADR 41038 D3 rule 1): every later `go` passes it to
host-manager as `open --mode <x>`, which uses it or refuses with the reason
(`mode_unavailable`, `mode_unreachable`) and never substitutes; `auto`
removes the pin and the ladder decides (Herdr when its server runs, else
tmux; `msp` — behind `TBH_AGENTS_SESSION_PROTOCOL` — only when pinned or
by host, or when this machine advertises MSP; the pin overrides that arm
both ways, and flag off an `msp` pin is refused naming the flag). Written as `mode: <x>` under
`## Settings` in PROJECT.md, beside the other settings, once. Running
threads keep the mode they opened with (`running_keep_mode`); a project never
switches mid-flight. Say the pin in one line when the user asked for it; a
pin is the user's word, never yours.

`mode`：项目的模式钉（ADR 41038 D3 规则 1）：此后每次 `go` 都把它作为 `open --mode <x>` 传给 host-manager，后者用它或带理由拒绝（`mode_unavailable`、`mode_unreachable`）而绝不替换；`auto` 拔钉，由阶梯决定（Herdr 服务器在跑则 Herdr，否则 tmux；`msp`——藏在 `TBH_AGENTS_SESSION_PROTOCOL` 之后——只在被钉住或按主机、或本机宣告 MSP 时；钉子双向压过那条臂，而对一个 `msp` 钉子关旗标会被拒绝并点名该旗标）。以 `mode: <x>` 写入 PROJECT.md 的 `## Settings` 之下、其他设置旁边，一次。运行中的线程保持其开启时的模式（`running_keep_mode`）；项目绝不在飞行中途切换。用户要求时用一行说明这个钉；钉是用户的话，绝不是你的。

`engine` (#43739, spec 38715-agents FR-43739-4): the project's default worker
engine — `init --engine codex` writes it when the user names the workers'
engine ("do this with codex workers"); `set <slug> engine claude` pins or
changes it later; `auto` removes the line. Written as `engine: <x>` under
`## Settings`. A proposal thread that names no `engine` takes it; a thread's
own `engine` still wins; without the line a thread runs the coordinator's
own engine, else muse (QA r10 ENGINES D1). Threads already proposed or
running keep theirs. host-manager applies the engine's own posture and
trust as for any thread; Muse flags (`model`, `effort`) stay Muse-only.

`engine`（#43739，规格 38715-agents FR-43739-4）：项目的默认 worker 引擎——用户点名 worker 引擎时（"do this with codex workers"）由 `init --engine codex` 写入；`set <slug> engine claude` 之后钉住或更改它；`auto` 移除该行。以 `engine: <x>` 写在 `## Settings` 之下。未点名 `engine` 的提案线程取它；线程自己的 `engine` 仍然胜出；没有该行时线程运行协调者自己的引擎，否则 muse（QA r10 ENGINES D1）。已提议或运行中的线程保持其自己的。host-manager 照对任何线程那样施用该引擎自己的姿态与信任；Muse 旗标（`model`、`effort`）仍只属 Muse。

`every`, `heartbeat`, `sink` (#41802): the watcher's settings, in the same
section (`every: 120s` | `every: quiet`, `heartbeat: 5`, `sink: <command>`),
re-read by a running watcher before its every sleep — no restart. `every`
is the pace of the "still working" edits ("update me every 2 min" → `2m`;
"less often" → a longer value; "quiet until done" → `quiet` (`off` is the
same word); `auto` removes the line and the cadence table decides); under
10 s is `usage`. `heartbeat` makes a cadence tick also file a `watch`
inbox event every N minutes — for a TUI user who wants a periodic line;
off by default, `off` removes it. `sink` is the command each rendered list
is piped to (§ `watch`); `none` removes it. Outcome `set` with `setting`,
`value` (`null` after `auto`/`off`/`none`), a receipt; `usage` (2) for
another setting or value.

`every`、`heartbeat`、`sink`（#41802）：观察者的设置，在同一节里（`every: 120s` | `every: quiet`、`heartbeat: 5`、`sink: <command>`），由运行中的观察者在每次 every 睡眠前重读——无需重启。`every` 是"仍在工作"编辑的节奏（"update me every 2 min" → `2m`；"less often" → 更长的值；"quiet until done" → `quiet`（`off` 是同一个词）；`auto` 移除该行、由节奏表决定）；低于 10 秒是 `usage`。`heartbeat` 让一次节拍 tick 也每 N 分钟归档一个 `watch` 收件箱事件——给想要周期性一行的 TUI 用户；默认关闭，`off` 移除它。`sink` 是每份渲染出的列表经由管道送往的命令（§ `watch`）；`none` 移除它。结果 `set` 带 `setting`、`value`（`auto`/`off`/`none` 之后为 `null`）与一条回执；其他设置或值是 `usage`（2）。

## `watch` / 观察

```
watch <slug> [--sink "<command>"]                                  # the project's threads (what `go` starts)
watch <slug> --source cmd --step "<label>" [--sink "<command>"] -- <command…>   # one long step of your own
watch <slug> --stop
```

The project's watcher: ONE long-lived child per project, started by `go`
(pid on `state.json` `watch`, bound to the process by its start stamp (`ps`'s `lstart` spelling, read from `/proc` on Linux) —
a recycled pid is a stranger's: never signalled, treated as gone; a second `go` keeps it; one that died while
threads run is restarted by the next `go`, `context` or `tick` — the wake
loop's round included; `watch_restarted` on that line), ended by `agents.py archive`,
by `--stop`, or by itself once every row is terminal (done, accepted,
failed, stopped). It never wakes you itself. Every **20 s** (`WATCH_POLL_S`)
it reads the project's own records and each running thread's session — the
same facts `context`/`overview` compute — and renders one row per opened
thread:

项目的观察者：每项目一个长命子进程，由 `go` 启动（pid 记在 `state.json` 的 `watch` 上，经其启动时间戳绑定到进程（`ps` 的 `lstart` 拼法，Linux 上从 `/proc` 读取）——被回收的 pid 是陌生人的：绝不发信号，按没了处理；第二次 `go` 保留它；线程运行中死掉的一个由下一次 `go`、`context` 或 `tick` 重启——唤醒循环的轮次也算；该行上有 `watch_restarted`），由 `agents.py archive`、`--stop` 或其自身在每一行都到终态（done、accepted、failed、stopped）时结束。它自己绝不唤醒你。每 **20 秒**（`WATCH_POLL_S`）它读取项目自己的记录与每个运行中线程的会话——与 `context`/`overview` 计算的相同事实——并为每个已开启线程渲染一行：

```
☐ running · ⛔ blocked (the fact is the question) · ✅ done (report acked) · ✅✔ accepted · ✖ failed (session gone, checkout gone) · ⛔ stopped
<mark> <name> — <what it owns> [· <elapsed>] [· <one fact: the report's STATUS line, else its newest commit>]
```

(The marks are `plan_lines`' own, so your post and the watcher's edit are
one list; a proposed thread is not on it, as on `plan_lines`.)

（这些记号是 `plan_lines` 自己的，因此你的发布与观察者的编辑是同一份列表；提议中的线程不在其上，正如不在 `plan_lines` 上。）

Two clocks. A **state transition** between two samples (running →
blocked, → done, → failed, …) rewrites the list at once and restarts the
cadence interval; a transition into blocked/done/accepted/failed/stopped,
or out of blocked, also files one `watch` inbox event
(`watch:<id>:<state>:<n>`, text `<from> → <to> — <fact>`), which the wake
loop prints as `<Name> moved: …` — the Monitor wakes you (on the inbox path
the message is sent too). A running row's new report the coordinator has
not acked is a transition into `ready-for-review`: one event per digest
(`watch:<id>:ready-for-review:<digest>`, text `running → ready-for-review —
<STATUS>`), no edit of its own (the next cadence rewrite carries the fact);
a row whose state moved in the same sample files that transition's event
alone. The start batch files nothing and is not news to
the sink: you just opened them (a lane that cannot fold an edit posts
nothing for it). Otherwise a **"still working"** rewrite
follows the cadence table (`WATCH_CADENCE`), keyed by time since the watch
started, `every` overriding it:

两个时钟。两次采样之间的一次**状态转变**（running → blocked、→ done、→ failed，…）立即重写列表并重启节奏间隔；转入 blocked/done/accepted/failed/stopped 或转出 blocked 的转变还归档一个 `watch` 收件箱事件（`watch:<id>:<state>:<n>`，text 为 `<from> → <to> — <fact>`），唤醒循环把它打印为 `<Name> moved: …`——Monitor 唤醒你（收件箱路径上消息也会发送）。运行中行的一份协调者尚未 ack 的新报告是向 `ready-for-review` 的转变：每个 digest 一个事件（`watch:<id>:ready-for-review:<digest>`，text 为 `running → ready-for-review — <STATUS>`），没有自己的编辑（下一次节奏重写会带上这个事实）；同一采样中状态移动的行只归档那一转变的事件。启动批次什么都不归档，对 sink 也不是新闻：是你刚打开它们的（一条不能折叠编辑的泳道不为它发布任何东西）。否则，**"仍在工作"**的重写遵循节奏表（`WATCH_CADENCE`），以观察启动以来的时间为键，`every` 压过它：

| elapsed | edit every |
| --- | --- |
| under 10 min | 1 min |
| 10–30 min | 2 min |
| 30–60 min | 5 min |
| 1–2 h | 10 min |
| after 2 h | 15 min |

| 已耗时 | 编辑间隔 |
| --- | --- |
| 不足 10 分钟 | 1 分钟 |
| 10–30 分钟 | 2 分钟 |
| 30–60 分钟 | 5 分钟 |
| 1–2 小时 | 10 分钟 |
| 2 小时之后 | 15 分钟 |

Where the list goes: always `library/status.md` (`# <slug> — updated <t>`,
then the rows; `context`/`overview` return `watch` = `{pid, status_file,
updated_at, every, heartbeat, sink}`); with `--sink` (else the `sink`
setting) also to that command's stdin, one call per rewrite — a channel's
plan message, edited in place by id (the channel's own sink command, set
by the program that created the project). The sink edits only the
coordinator's plan post: until that message exists it answers `skipped:
no_plan` with no id, the watcher logs the wait once and the status file
alone carries the list — the watcher never creates the plan message and
never edits an acknowledgement (#41959). Sink contract: stdin is the
rendered rows; argv gets `--message-id <id>` (from its own previous answer),
`--news` on a transition batch, `--row` for a `--source cmd` step (rewrite
one row, not the block), `--stamp` to answer without editing; stdout is one
JSON line `{"message_id", "edited_at_ms"}`; a non-zero exit is logged once
and never stops the watch; an answer `{"outcome": "refused", "reason": …}`
with exit 3 (not applicable: a card as the target) is learned once — no
further sink calls, the status file keeps the list. **Backoff**: before a cadence rewrite the watcher
asks the sink for the stamp; a stamp newer than its own last edit (you
ticked or reworded the list) skips that tick and restarts the interval;
no stamp, no backoff. `--source cmd` watches one command of your own
instead (its row `☐ <label> · <elapsed> · <last log line>`, `✅ <label>` or
`✖ <label> · exit <n>` on exit, `⛔ · stopped` on `--stop`; the output is
teed; the exit code is the command's; no inbox event — the step's own
completion wakes the lane).

列表去哪里：总是 `library/status.md`（`# <slug> — updated <t>`，然后是各行；`context`/`overview` 返回 `watch` = `{pid, status_file, updated_at, every, heartbeat, sink}`）；带 `--sink`（否则是 `sink` 设置）时还送到该命令的 stdin，每次重写一次调用——频道的计划消息，按 id 原地编辑（频道自己的 sink 命令，由创建项目的程序设置）。sink 只编辑协调者的计划帖：那条消息存在之前它应答 `skipped: no_plan` 且不带 id，观察者把等待记录一次，状态文件单独承载列表——观察者绝不创建计划消息，也绝不编辑一条确认（#41959）。Sink 契约：stdin 是渲染出的行；argv 得到 `--message-id <id>`（来自它自己上一次的应答）、转变批次上的 `--news`、`--source cmd` 步骤的 `--row`（重写一行，不是整块）、不编辑只应答的 `--stamp`；stdout 是一行 JSON `{"message_id", "edited_at_ms"}`；非零退出记录一次且绝不停掉观察；exit 3 的应答 `{"outcome": "refused", "reason": …}`（不适用：目标是卡片）学习一次——不再有 sink 调用，状态文件保留列表。**退避**：节奏重写之前观察者向 sink 询问时间戳；比它自己上次编辑更新的时间戳（你 tick 过或改写过列表）跳过该 tick 并重启间隔；没有时间戳就没有退避。`--source cmd` 改为监视你自己的一条命令（其行 `☐ <label> · <elapsed> · <last log line>`，退出时 `✅ <label>` 或 `✖ <label> · exit <n>`，`--stop` 时 `⛔ · stopped`；输出被 tee；退出码是该命令的；没有收件箱事件——步骤自己的完成唤醒泳道）。

Outcomes: `watch_ended` (0; `ended`: done | failed | stopped, `edits`,
`transitions`, `events`, `elapsed`) — the JSON line is the watcher's last
stdout line, under `library/watch.log` when `go` started it;
`already_watching` (0, `pid`) for a second `watch` on a live one;
`watch_stopped` (0, `pid`) / `no_watch` (0) for `--stop`; `usage` (2) for
`--source cmd` without a command. The watcher composes no prose: the mark,
the name, what it owns, elapsed and one fact from the source; when you are
awake you say what it means.

结果：`watch_ended`（0；`ended`：done | failed | stopped，`edits`、`transitions`、`events`、`elapsed`）——JSON 行是观察者的最后一行 stdout，由 `go` 启动时在 `library/watch.log` 之下；对活着的观察者再 `watch` 一次是 `already_watching`（0，`pid`）；`--stop` 为 `watch_stopped`（0，`pid`）/ `no_watch`（0）；`--source cmd` 没带命令是 `usage`（2）。观察者不写散文：记号、名字、它拥有什么、已耗时间与来自源的一个事实；你醒着时由你说明它意味着什么。

## `resume` / 恢复

```
resume <slug> [--takeover --confirm "<the human's words>"]
```

A session whose identity (`MUSE_LANE_BACKEND` / `MUSE_LANE_REF`) is one of
the project's own thread records is `coordinator_is_thread` (3, `thread`
named, nothing recorded, with or without `--takeover`): a coordinator is
never one of its threads. A new coordinator session takes the project over from the folder. It first
asks whether the previous coordinator is live — a session through its
provider, a process identity on the same host through `/proc` on Linux, `ps` elsewhere (its `pid`, and
its `started` stamp, so a reused pid is not a live coordinator): a live one
is `coordinator_live` (3) — two coordinators would race — and a session
whose provider cannot answer, a tmux pane its server cannot judge, or an
`opaque` sandboxed coordinator is `coordinator_unknown` (6) — it may still be
live — unless `--takeover --confirm` carries the human's words; a process
identity that is gone proceeds with a `progress` line, and one on another
host (or without a pid, written by an older helper) cannot be checked and
proceeds with a `progress` line saying so. A second session on the same
host is therefore never the same coordinator by accident. Then it reconciles every
open thread by identity (gone or mismatched without a report → `orphaned`;
gone after a report → `exited`; live → unchanged; a provider that cannot
answer leaves the thread as it was and lists it under `unknowns`), clears
the wake arm (a Monitor bound to the previous session is not this one's —
re-arm with `tick --arm`), records this session as the coordinator, and
returns the same picture `context` does. `resumed` (0) with `threads`,
`unknowns`, `previous_coordinator`, `wake: null`, `previous_stopped`. A
`--takeover` of a coordinator that `init --detach` opened and that is live
or cannot be checked stops that session on the same words, before this
session is recorded (`previous_stopped: true`; a stop that fails passes
through and records nothing, so the retry still finds it; one that answers
`no_such_session` after a probe that could not answer proceeds with "not
found on this server" — host-manager `stop` looks on the caller's server —
while `no_such_session` after a probe that said live is that contradiction:
`no_such_session` (3, nothing recorded) with the helper's own `next` — stop
the coordinator on its own tmux server, then `resume --takeover --confirm
"<the human's words>"` again): nobody sits in it and it would run on
beside the new coordinator, and never two live coordinators; a person's own
live session is never stopped by a takeover. The session recorded by `init --detach`
resuming itself (its starter runs `resume`) keeps `detached` and its posture
on the record, so `context` still says so and `agents.py archive` from
another session finds and stops it.

身份（`MUSE_LANE_BACKEND` / `MUSE_LANE_REF`）是项目自己线程记录之一的会话是 `coordinator_is_thread`（3，点名 `thread`，什么都不记录，带不带 `--takeover` 都一样）：协调者绝不是自己的线程之一。一个新的协调者会话从文件夹接管项目。它先询问上一个协调者是否还活着——经其提供者的会话，同一主机上经 Linux 的 `/proc`、其他平台的 `ps` 的进程身份（其 `pid`，及其 `started` 时间戳，因此被复用的 pid 不是一个活协调者）：活着的是 `coordinator_live`（3）——两个协调者会赛跑——而提供者答不上来的会话、其服务器无法判定的 tmux pane、或 `opaque` 的沙盒协调者是 `coordinator_unknown`（6）——它可能还活着——除非 `--takeover --confirm` 带着人的原话；已经没了的进程身份带着一条 `progress` 行继续，另一台主机上的（或没有 pid、由更老的辅助脚本写的）无法核对，带着说明此事的 `progress` 行继续。因此同一主机上的第二个会话绝不会碰巧就是同一个协调者。然后它按身份对账每个开放线程（没了或不匹配且无报告 → `orphaned`；报告之后没了 → `exited`；活着 → 不变；答不上来的提供者让线程保持原样并列入 `unknowns`），清除唤醒布防（绑定上一个会话的 Monitor 不是这个的——用 `tick --arm` 重新布防），把本会话记录为协调者，并返回与 `context` 相同的图景。`resumed`（0），带 `threads`、`unknowns`、`previous_coordinator`、`wake: null`、`previous_stopped`。对 `init --detach` 开启的、活着或无法核对的协调者 `--takeover`，在同样的原话上停掉那个会话，在本会话被记录之前（`previous_stopped: true`；一次失败的 stop 会透传并不记录任何东西，因此重试仍找得到它；一次答不上来的探测之后的 `no_such_session` 带着"not found on this server"继续——host-manager 的 `stop` 在调用方的服务器上找——而说了活着的探测之后的 `no_such_session` 就是那个矛盾：`no_such_session`（3，什么都不记录），带辅助脚本自己的 `next`——在其自己的 tmux 服务器上停掉协调者，然后再次 `resume --takeover --confirm "<人的原话>"`）：没有人坐在里面，否则它会跑着伴随新协调者，且绝不要两个活协调者；人自己的活会话绝不被接管停掉。`init --detach` 记录的会话恢复它自己（其 starter 运行 `resume`）时，在记录上保留 `detached` 及其姿态，于是 `context` 仍如此说明，且来自另一会话的 `agents.py archive` 能找到并停掉它。

A session that binds a project a launcher made (`init` under
`MUSE_AGENTS_ROLE=launcher` recorded nobody; the lane it opened runs the first
`resume`) is recorded `detached: true`, like an `init --detach` coordinator:
nobody sits in it, so `agents.py archive --confirm` from any session stops it
and names it, and `resume --takeover --confirm` stops it before recording the
taker (a shell's archive was `not_coordinator` and a
takeover left two lane sessions running against archived projects). A
person's own session that ran `init` itself is never marked so.

绑定启动者所建项目（在 `MUSE_AGENTS_ROLE=launcher` 之下的 `init` 没记录任何人；它开启的泳道跑第一次 `resume`）的会话被记录为 `detached: true`，如同 `init --detach` 的协调者：没有人坐在里面，因此来自任何会话的 `agents.py archive --confirm` 都能停掉它并点名它，而 `resume --takeover --confirm` 在记录接管者之前停掉它（一次 shell 的 archive 曾是 `not_coordinator`，而一次接管留下两个泳道会话跑在已归档项目上）。自己运行过 `init` 的人自己的会话绝不被如此标记。

On the inbox path `resume` keeps `wake_path: inbox` whatever
`TBH_AGENTS_SESSION_PROTOCOL` says now (a `progress` line says so) and
re-subscribes — this session is resolved as the report target and recorded
as `inbox_target` (on the line too, with `wake_path`); `resubscribed` lists
the running threads whose next `report` reaches it through the folder; a
remote thread's report filed by the coordinator itself sends nothing. The wake arm is cleared
and the loop rewritten as on the monitor path, and `next` is the same arm
line. A monitor project resumed with the flag on stays on the monitor path
and says so.

在收件箱路径上，无论 `TBH_AGENTS_SESSION_PROTOCOL` 现在怎么说，`resume` 都保持 `wake_path: inbox`（一条 `progress` 行说明）并重新订阅——本会话被解析为报告目标并记录为 `inbox_target`（行上也有，连同 `wake_path`）；`resubscribed` 列出其下一次 `report` 将经文件夹到达它的运行中线程；由协调者自己归档的远程线程报告不发送任何东西。唤醒布防被清除，循环照 monitor 路径那样重写，`next` 是同一布防行。带着旗标恢复的 monitor 项目留在 monitor 路径上并如实说明。

## `stop` / 停止

```
stop <slug> <thread-id…> [--asked-by WHO]
```

Ends one or several threads' sessions (several ids like `go`; every id is
checked before any session ends — an unknown one is `no_such_thread`, 3,
and nothing stops) through host-manager `stop` or fleet-manager
`close` (the engine's own quit, a wait, a close of what lingers), whatever
the record's status: a `running` record becomes `stopped`; a `done`,
`exited`, `orphaned` or `stopped` record whose engine still runs keeps its
status and only loses the session (`ended: true`, `stop_receipt` on the
record). `ended` is this helper's word — a live session was ended;
`lingered` is the provider's, passed through unchanged (`true`: the engine
ignored its quit gesture and was closed; `null` when no session was
asked to quit). A thread that is already gone is `stopped` with `ended:
false` (a remote session fleet-manager no longer lists is gone the same
way); a session under the thread's name whose identity no longer matches
the record is a stranger's — the record becomes `orphaned` (`exited` when
a report exists) with `identity_drift`, nothing is sent or closed
(`ended: false`); a `proposed` one never had a session. The caller is checked before anything
else: from a session that is not the recorded coordinator every `stop` is
`not_coordinator` (3), a `proposed` thread's included. A provider that answers
`no_such_session` for a session it just reported live (a wrong address, a
provider in trouble) is that refusal, passed through (3): nothing is
stamped stopped while a session may still run; `next` is `tick`, then
`stop` again. One id answers as before: `ended`, `lingered`, `status` and
one `receipt` at the top (plus `threads`, the same words under the id).
Several ids answer `threads` (one row per id: `ended`, `lingered`,
`status`, `identity_drift` when set), `stopped` (the ids whose live session
was ended) and `receipts`, `ref` = `<slug>/<id,id,…>`; a provider refusal
for one of them rides under that id (`outcome`, `error`, `next`) while the
others are ended, and the line is `partial` (6) naming it, `next` its cure
(`stop <slug> a b c` was `usage` in 3/3 projects).

结束一个或多个线程的会话（多个 id 与 `go` 相同；任何会话结束之前每个 id 都先检查——未知的是 `no_such_thread`，3，什么都不停）——通过 host-manager 的 `stop` 或 fleet-manager 的 `close`（引擎自己的退出、一次等待、对残留物的关闭），无论记录的状态：`running` 记录变为 `stopped`；引擎仍在运行的 `done`、`exited`、`orphaned` 或 `stopped` 记录保持其状态，只失去会话（`ended: true`，记录上有 `stop_receipt`）。`ended` 是本辅助脚本的话——一个活会话被结束了；`lingered` 是提供者的，原样透传（`true`：引擎无视了它的退出示意而被关闭；没有要求任何会话退出时为 `null`）。已经消失的线程是 `stopped` 且 `ended: false`（fleet-manager 不再列出的远程会话以同样方式算没了）；顶着线程名字而身份不再匹配记录的会话是陌生人的——记录变为 `orphaned`（有报告时 `exited`）并带 `identity_drift`，不发送也不关闭任何东西（`ended: false`）；`proposed` 的从来没有会话。调用方先于一切被检查：从一个不是被记录协调者的会话，每个 `stop` 都是 `not_coordinator`（3），`proposed` 线程的也算。对一个刚报告还活着的会话应答 `no_such_session` 的提供者（一个错的地址、一个陷入麻烦的提供者）就是那个拒绝，原样透传（3）：会话可能还在跑时什么都不盖 stopped 戳；`next` 是 `tick`，然后再次 `stop`。单个 id 照旧应答：顶层 `ended`、`lingered`、`status` 与一条 `receipt`（外加 `threads`，id 之下同样的话）。多个 id 应答 `threads`（每 id 一行：`ended`、`lingered`、`status`、有则 `identity_drift`）、`stopped`（活会话被结束的那些 id）与 `receipts`，`ref` = `<slug>/<id,id,…>`；其中一个的提供者拒绝搭在该 id 之下（`outcome`、`error`、`next`）而其余被结束，该行是 `partial`（6）并点名它，`next` 是其补救（`stop <slug> a b c` 曾在 3/3 个项目上是 `usage`）。

## `agents.py archive` / 归档

```
archive <slug> [--confirm "<the human's words>"] [--asked-by WHO]
```

Completion cleanup, in this order, each a `progress` line. The first
`progress` line, and the line's `text`, is `Monitor: do not work_stop it — it
ends by itself within a tick; the empty `Monitor event` that follows is not
input (one line at most: `Monitor ended; project archived.`, never the
close-out again)` (three coordinators stopped it
after reading the receipt; the first line survives a `head`); the receipt's
`monitor_ended_line` is that one line, `Monitor ended; project archived.`,
verbatim — the ended-Monitor turn's whole text, and `next` says so ("say
nothing" was spoken aloud instead, twice with the close-out repeated). Once the
live-session guard below passes, disarm the wake first — the arm cleared
and `library/wake.sh` removed before any thread is stopped, so the stops
are news to nobody (a Monitor WAKE fired after the
archive and cost a turn; `disarmed` names the tier, `null` when none was
armed, on the line and the receipt); the loop a Monitor still runs is left
alone — it exits 0 within one tick once the folder is gone, so the Monitor
ends by itself (owner ruling 26, #38715: the item stayed `watching`;
supersedes the stay-alive loop), and the empty `Monitor event:
agents <slug>` the runtime then delivers (a `stream_ended` item; the TUI shows
`Monitor "agents <slug>" — ended`) is the Monitor ending, not new input: say
nothing and end that turn (a `work_stop` is the same note one turn earlier);
`next` says so; then stop every
session that is still live — every thread's, whatever its status (`done`
threads' engines included), and a coordinator `init --detach` opened when
it is not the session running the verb; one whose provider cannot answer
counts as live; a session this helper already ended (`session_ended_at`)
is not asked about again; a `done` thread's live session (one `accept`
could not end, #41777) is ended without
the words — its work was accepted on evidence, and its idle engine is what
`threads_live` refused every project whose threads were still live (`threads_live`,
3, naming the live sessions of threads not yet done, `coordinator` among
them, when any is live and there is no `--confirm`; `next` names `accept`,
the `--confirm` line and the `stop <slug> <ids>` line);
for each thread `worktree_path` that git reports clean and
whose HEAD is merged into the repository's default branch — its copy on
`origin` after one bounded `git fetch` when the repository has an origin
(a local `main` nobody pulled is stale), else the local branch; a failed
fetch is a `progress` line naming the copy compared — `git worktree remove
<path>` — **never `--force`**; a landed checkout whose only untracked files
are the thread's report (`report*.md`, `report*.txt` or `AGENTS-REPORT.md`
at its root) counts as clean: they move to `threads/<id>/leftover/` first
(a `progress` line each), nothing is deleted; a dirty or unmerged worktree
is left in place and named in `kept` (`not merged into origin/main`); move the folder to
`~/.muse/projects/.archive/<slug>-<stamp>/` (the records stay readable;
nothing is deleted). Branches and PRs are never touched. `"archived"` (0)
with `stopped` (thread ids whose session it ended), `coordinator`, which
names the detached session from the record — `stopped (<provider>:<ref>)`;
`not live (<provider>:<ref>)` when it was gone or its stop found no session;
`none` when the project had no detached coordinator and another session
archives it; `this session (<provider>:<ref>)` when the recorded coordinator
with a session identity runs the verb itself (the verb never stops the
session it runs in, so a detached coordinator that archives its own project
stays open until the user closes it); `this TUI session` when the recorded
coordinator running the verb has no session identity (a process or pane) — `removed_worktrees`, `kept`,
`"archive_path"`, and a receipt. A reader that closes the pipe early
(`agents.py archive <slug> 2>&1 | head -5`) never stops the verb: the dropped lines
stay in `progress`, the folder moves, the receipt is written and the exit
code is the verb's own. A slug is one path component of letters,
digits, `.`, `_` and `-` (`usage`, 2, otherwise) on every verb.

完成后的清理，按此顺序，每步一条 `progress` 行。第一条 `progress` 行，以及该行的 `text`，是 `Monitor: do not work_stop it — it ends by itself within a tick; the empty `Monitor event` that follows is not input (one line at most: `Monitor ended; project archived.`, never the close-out again)`（三位协调者在读完回执后停掉了它；第一行能在 `head` 下幸存）；回执的 `monitor_ended_line` 就是那一行 `Monitor ended; project archived.`，一字不差——结束的 Monitor 轮的整段文本，`next` 也如此说明（"什么都不说"曾被大声说出，两次还带着重复的收尾）。一旦下文的活动会话守卫通过，先解除唤醒——布防被清除、`library/wake.sh` 在任何线程被停止之前移除，于是那些停止对谁都构不成新闻（一次 Monitor WAKE 曾在归档之后响起并花掉一轮；行上与回执上的 `disarmed` 点名层级，没有布防过时为 `null`）；Monitor 仍在跑的循环原样留下——文件夹没了之后它在一个 tick 内 exit 0，于是 Monitor 自行结束（业主裁决 26，#38715：条目曾保持 `watching`；取代常驻循环），而运行时随之投递的空 `Monitor event: agents <slug>`（一个 `stream_ended` 条目；TUI 显示 `Monitor "agents <slug>" — ended`）是 Monitor 的结束，不是新输入：什么都不说并结束那一轮（`work_stop` 是早一轮的同一备注）；`next` 如此说明；然后停掉每个仍然活着的会话——每个线程的，无论其状态（`done` 线程的引擎也算），以及 `init --detach` 开启的、不是正在运行本动词的那个会话的协调者；提供者答不上来的算活着；本辅助脚本已经结束过的会话（`session_ended_at`）不再被询问；一个 `done` 线程的活会话（一次 `accept` 没能结束它，#41777）在没有那些话的情况下被结束——它的工作已凭证据被验收，而它空闲的引擎正是 `threads_live` 所拒的：每个线程仍活着的项目（`threads_live`，3，当任何一个活着且没有 `--confirm` 时，点名未 done 线程的活会话，`coordinator` 也在其中；`next` 点名 `accept`、`--confirm` 行与 `stop <slug> <ids>` 行）；对每线程 `worktree_path`：git 报告干净、且其 HEAD 已合并进仓库默认分支——仓库有 origin 时（一次有限的 `git fetch` 之后）是其 `origin` 上的副本（没人拉取过的本地 `main` 是陈旧的），否则本地分支——`git worktree remove <path>`——**绝不用 `--force`**；一个已落地的检出，其仅有的未跟踪文件是该线程的报告（其根部的 `report*.md`、`report*.txt` 或 `AGENTS-REPORT.md`）也算干净：它们先移到 `threads/<id>/leftover/`（各一条 `progress` 行），什么都不删除；脏的或未合并的 worktree 原地保留并记在 `kept` 里（`not merged into origin/main`）；把文件夹移到 `~/.muse/projects/.archive/<slug>-<stamp>/`（记录保持可读；什么都不删除）。分支与 PR 绝不被触碰。`"archived"`（0），带 `stopped`（其会话被它结束的线程 id）、`coordinator`——从记录点名分离会话——`stopped (<provider>:<ref>)`；已消失或其 stop 没找到会话时 `not live (<provider>:<ref>)`；项目没有分离协调者而由另一会话归档时 `none`；带会话身份的被记录协调者自己运行本动词时 `this session (<provider>:<ref>)`（本动词绝不停止它运行于其中的会话，因此归档自己项目的分离协调者保持打开，直到用户关闭它）；运行本动词的被记录协调者没有会话身份（进程或 pane）时 `this TUI session`——`removed_worktrees`、`kept`、`"archive_path"`，以及一条回执。提前关闭管道的读取者（`agents.py archive <slug> 2>&1 | head -5`）绝不会停掉该动词：被丢弃的行留在 `progress` 里，文件夹照样移动，回执照样被写，退出码是动词自己的。slug 是字母、数字、`.`、`_` 与 `-` 的单一路径成分（否则 `usage`，2），对所有动词皆然。

## Allow-list / 允许清单

Read and record verbs — `doctor`, `context`, `overview`, `propose`, `report`,
`ack`, `inbox put`, `inbox drain`, `tick` without `--arm` — may sit on an
agent's allow-list by subcommand, never the bare helper. `init --detach`,
`go`, `follow`, `stop`, `agents.py archive`, `resume --takeover`, `remember`, `accept`
and `tick --arm` start, end, take over or record authority and stay on the
permission prompt. `go`, `follow`, `accept`, `stop`, `remember`, `tick --arm`,
`tick --disarm`, `agents.py archive` and `propose --replace` also check the
caller: a session that is not the recorded coordinator
(`state.json.coordinator`) is `not_coordinator` (3, nothing changed; `next`
is `resume <slug>` when that coordinator is gone or is a process identity
that cannot be checked — the same cases a plain `resume` proceeds past —
and `resume <slug> --takeover --confirm` when it is live or a session whose
provider cannot answer). Read and record verbs never check. A detached
project's `stop` and `agents.py archive` stay open to any session: nobody
sits in a detached coordinator, both only end things, and `agents.py archive` stops
the detached session itself on the human's words. An `opaque` recorded
coordinator (a sandboxed tool shell with no tmux pane) can be told from
nobody, itself included, so these verbs proceed with a `progress` line
saying so; `resume --takeover --confirm` on the human's words settles it. The Claude Code settings template and what a prefix
rule cannot express (`tick --arm`, `init --detach`):
`references/allow-list.md`.

读取与记录动词——`doctor`、`context`、`overview`、`propose`、`report`、`ack`、`inbox put`、`inbox drain`、不带 `--arm` 的 `tick`——可以按子命令进入 agent 的允许清单，绝不是光杆辅助脚本。`init --detach`、`go`、`follow`、`stop`、`agents.py archive`、`resume --takeover`、`remember`、`accept` 与 `tick --arm` 会启动、结束、接管或记录权威，因此留在权限提示上。`go`、`follow`、`accept`、`stop`、`remember`、`tick --arm`、`tick --disarm`、`agents.py archive` 与 `propose --replace` 还检查调用方：一个不是被记录协调者（`state.json.coordinator`）的会话是 `not_coordinator`（3，什么都没变；该协调者已消失、或是一个无法核对的进程身份时 `next` 为 `resume <slug>`——与光杆 `resume` 会继续越过的相同情形——而其活着或提供者答不上来时为 `resume <slug> --takeover --confirm`）。读取与记录动词从不检查。分离项目的 `stop` 与 `agents.py archive` 对任何会话保持开放：没有人坐在分离协调者里，两者都只做结束的事，且 `agents.py archive` 会凭人的原话停掉分离会话本身。一个 `opaque` 的被记录协调者（一个没有 tmux pane 的沙盒工具 shell）谁也无法分辨——包括它自己——因此这些动词带着说明此事的 `progress` 行继续；凭人的原话的 `resume --takeover --confirm` 一锤定音。Claude Code 设置模板与前缀规则表达不了的东西（`tick --arm`、`init --detach`）：见 `references/allow-list.md`。

【评论】允许清单按"只读/记录"与"启动/结束/接管"划分权限粒度，并对写权威动词追加协调者身份检查——这是最小权限原则在 agent 工具链上的落地方式。

## Contract suite / 契约套件

The helper's contract suite ships with its source tree beside the skill:
stdlib unittest, offline, a fake host-manager and a fake fleet-manager
(injected through `MUSE_AGENTS_HOST_MANAGER` / `MUSE_AGENTS_FLEET_MANAGER`),
a per-test `MUSE_PROJECTS_HOME`, real `git` for the worktree arm, no sleeps.

辅助脚本的契约套件随其源码树一起放在 skill 旁：标准库 unittest、离线运行、一个假 host-manager 与一个假 fleet-manager（经 `MUSE_AGENTS_HOST_MANAGER` / `MUSE_AGENTS_FLEET_MANAGER` 注入）、每测试独立的 `MUSE_PROJECTS_HOME`、worktree 臂用真的 `git`、没有睡眠。
