<!-- BILINGUAL-EN-ZH -->
# fleet-manager verbs (`scripts/fleet_manager.py`) / fleet-manager 动词（`scripts/fleet_manager.py`）

The verb contract is shared with `host-manager`: one JSON object per verb,
one exit-code table, one receipt line per write. `fleet-manager` adds a
machine to every target and the machine verbs. `<fleet> <verb> --help`
prints the flags of one verb.

动词契约与 `host-manager` 共享：每个动词对应一个 JSON 对象、一张退出码表，每次写入对应一行回执。`fleet-manager` 在每个目标上增加了机器部分，并新增机器类动词。`<fleet> <verb> --help` 可打印某个动词的标志。

```text
<fleet> = python3 <skill-dir>/scripts/fleet_manager.py
<fleet> [--mode herdr|tmux|auto] [--asked-by <who>] <verb> …
```

## Envelope / 信封结构

Every verb prints exactly one object on stdout, success or failure:

无论成功或失败，每个动词都在 stdout 上恰好打印一个对象：

| key | meaning |
| --- | --- |
| `outcome` | what happened, one word: a verb's own success word (`healthy`, `detected`, `context`, `listed`, `machines`, `status`, `read`, `dialog`, `attach_command`, `resources`, `waited`, `ready`, `sent`, `notified`, `not_shown`, `keys_sent`, `answered`, `opened`, `interrupted`, `closed`, `adopted`, `forgotten`, `connected`, `fetched`) or a failure word from the exit table |
| `provider` | what served the verb: `herdr`, `tmux`, or `null` when none was selected |
| `ref` | the target the verb acted on (a handle, an address, a machine label); `null` for fleet-wide reads |
| `capabilities` | tmux: `liveness`, `scrollback`, `guarded_input`, `attach_by_name`; Herdr: those four plus `agent_status`, `dialogs`, `prompt_readiness`, `wait`, `wait_for_output`, `workspace`, `tab`, `worktree`, `notify` |
| `progress` | one line per step already taken (also echoed to stderr) |
| `next` | the one command that moves things forward; empty when there is nothing to do (no success word is `ok`; every failure word carries one, `doctor` when nothing more specific applies) |
| `receipt` | write verbs only: `what`, `session` (the identity tuple), `who`, `when` (ISO-8601 UTC), and the same as one `line` (with the `HH:MM UTC` clock) |
| `error` | failures only: the first thing wrong |
| `text` | reads that render something (the board, a read, a dialog, the context digest) |

| 键 | 含义 |
| --- | --- |
| `outcome` | 发生了什么，用一个词表示：动词自身的成功词（`healthy`、`detected`、`context`、`listed`、`machines`、`status`、`read`、`dialog`、`attach_command`、`resources`、`waited`、`ready`、`sent`、`notified`、`not_shown`、`keys_sent`、`answered`、`opened`、`interrupted`、`closed`、`adopted`、`forgotten`、`connected`、`fetched`），或退出码表中的失败词 |
| `provider` | 由谁服务该动词：`herdr`、`tmux`，未选中任何提供方时为 `null` |
| `ref` | 动词作用的目标（一个句柄、一个地址、一个机器标签）；全舰队读取时为 `null` |
| `capabilities` | tmux：`liveness`、`scrollback`、`guarded_input`、`attach_by_name`；Herdr：上述四项外加 `agent_status`、`dialogs`、`prompt_readiness`、`wait`、`wait_for_output`、`workspace`、`tab`、`worktree`、`notify` |
| `progress` | 每个已执行的步骤对应一行（同时回显到 stderr） |
| `next` | 推动事情向前的那一条命令；无事可做时为空（没有任何成功词是 `ok`；每个失败词都对应一条，无更具体者时为 `doctor`） |
| `receipt` | 仅写入类动词：`what`、`session`（身份元组）、`who`、`when`（ISO-8601 UTC），以及合并为一行 `line` 的同一内容（使用 `HH:MM UTC` 时钟） |
| `error` | 仅失败时出现：最先出问题的地方 |
| `text` | 会渲染内容的读取（面板、一次读取、一个对话框、上下文摘要） |

Exit codes:

退出码：

- `0` ok.
  `0` 正常。
- `2` `usage` — the message names the flag or address.
  `2` `usage` —— 消息中指明该标志或地址。
- `3` `refused` / `no_such_session` / `identity_mismatch` / `session_live` /
  `name_taken` / `composer_not_empty` — refused by a guard, nothing changed
  (`no_such_session`: the handle names no live session, a gone session, not
  an unknown one).
  `3` `refused` / `no_such_session` / `identity_mismatch` / `session_live` / `name_taken` / `composer_not_empty` —— 被守卫拒绝，任何东西都未改动（`no_such_session`：句柄指代的是一个已消失的会话，而不是未知会话）。
- `4` `unsupported_by_provider` / `no_provider` — `next` names the
  alternative.
  `4` `unsupported_by_provider` / `no_provider` —— `next` 指出替代方案。
- `5` `needs_user_action` — an install command, a login or second factor,
  a confirmation; `next` is that step.
  `5` `needs_user_action` —— 需要一条安装命令、一次登录或第二因素、一次确认；`next` 即该步骤。
- `6` `provider_unreachable` / `failed` / `tmux_unavailable` /
  `herdr_unavailable` / `herdr_add_failed` / `composer_unreadable` — the
  guard could not look, so nothing was typed; nothing changed unless the
  line says `created: true`.
  `6` `provider_unreachable` / `failed` / `tmux_unavailable` / `herdr_unavailable` / `herdr_add_failed` / `composer_unreadable` —— 守卫未能查看，因此没有键入任何内容；除非 `line` 显示 `created: true`，否则没有任何改动。
- `7` `internal`.
  `7` `internal`（内部错误）。

One verb streams instead: `events` (one line per change, for one Monitor;
`events --once` is one object).

有一个动词改以流方式工作：`events`（每次变更一行，面向单个 Monitor；`events --once` 则输出一个对象）。

## Addresses / 地址

`s3` (a handle the board minted) or `machine[:server]/<ref>`: `machine` is
`local` or a machine label; `server` is a Herdr session name (default: the
profile's); `ref` is a pane id (`w1:p2`), a unique live agent name, a human
label (the pane's label, its terminal title, its tab's or its workspace's
label — exact first, then a unique case-insensitive match; two matches name
the candidates), or a tmux session name. Pane ids, names and labels are
server-local: never drop the machine part.

`s3`（面板铸造的句柄）或 `machine[:server]/<ref>`：`machine` 是 `local` 或一个机器标签；`server` 是一个 Herdr 会话名（默认：profile 的会话名）；`ref` 是一个窗格 id（`w1:p2`）、一个唯一的存活代理名、一个人工标签（窗格的标签、其终端标题、其标签页或工作区的标签——先精确匹配，再唯一的大小写不敏感匹配；出现两个匹配时会列出候选），或一个 tmux 会话名。窗格 id、名称与标签都是服务器局部的：绝不省略机器部分。

A handle-shaped address nobody minted (`send s17 …`, when a human named the
session `s17`) is not refused as an unknown handle: it is resolved like any
other name — one fleet read, every session's name and the labels a human
sees, exact first then a unique case-insensitive match; two answers name the
candidates, none is `no_such_session`. A handle this skill minted always
wins over a session named like it.

一个无人铸造却形似句柄的地址（当有人把会话命名为 `s17` 时的 `send s17 …`）不会被当作未知句柄而拒绝：它像其他任何名称一样被解析——一次全舰队读取，覆盖每个会话的名称与人工可见的标签，先精确匹配再唯一的大小写不敏感匹配；得到两个答案时列出候选，而不是返回 `no_such_session`。本技能铸造的句柄总是优先于恰好同名的会话。

A handle carries the identity tuple
`(provider, machine, server, ref, cwd, engine)` — `server` is the Herdr
socket path or the tmux socket label the verb actually used, stored on the
handle at open/adopt/list and compared like every other field — and every
write verb checks the whole tuple first: a restored pane is not the prior
process and a pane id or name can be reissued to a new one; a mismatch is
`identity_mismatch` (exit 3) naming `adopt <address>`, which takes the
session now under the address on purpose. `list` shows such a handle as
`drift (…)` and never re-points it at the stranger.

句柄携带身份元组 `(provider, machine, server, ref, cwd, engine)`——`server` 是该动词实际使用的 Herdr 套接字路径或 tmux 套接字标签，在 open/adopt/list 时存入句柄并与其他字段一样参与比较——每个写入动词都先检查整个元组：恢复出来的窗格不再是原先的进程，而且窗格 id 或名称可能被重新分配给新进程；不匹配即为 `identity_mismatch`（退出码 3），并指名 `adopt <address>`，用于有意接管现在位于该地址下的会话。`list` 会把这样的句柄显示为 `drift (…)`，且绝不把它重新指向陌生的会话。

【评论】身份元组在每次写入前整体校验，是对"窗格 id 被重新分配给新进程"这类竞态的防护设计。

## Read verbs (allow-listable) / 读取类动词（可列入允许清单）

| verb | does |
| --- | --- |
| `doctor [--no-start] [--no-install]` | `healthy` (or `needs_user_action` / `no_provider` / `provider_unreachable`): `checks` rows (`python`, `herdr`, `tmux`, `machines_directory`, `provider`), every machine's reachability, the one next command; starts a stopped Herdr server, installs tmux when that needs no password |
| `detect [--no-start] [--no-install]` | the provider decision for this host: `reason` (`herdr_reachable`, `herdr_started`, `herdr_down`, `tmux_available`, `tmux_installed`, `requested`, `no_provider`), `providers` rows (`installed`, `reachable`, `server`, `version`, `capabilities`) |
| `context [--reset]` | one digest: `sessions` rows (each with `identity` and `group`: `waiting-on-you`, `ready-for-review`, `working`, `idle`; an MSP host's sessions come from one `muse sessions --host <host>` per online host of yours, four at a time under a per-host deadline (`FLEET_MANAGER_MSP_LIST_TIMEOUT_S`, 8 s) — a pending request is `waiting-on-you`, a running turn `working`, else `idle` — and carry `provider: msp`; a host that does not answer, or not in time, is `unreachable` with the transport's own message or "did not answer within N s (listing pending; ask again)", never "0 sessions"; strangers' hosts and sessions are never read or rendered), `groups`, machines with reachability (`connected`, `stale`, `unreachable`, `disabled`, or `unverified` for a directory row whose `connect` stopped before verification — its `next` is that `connect`, never an outage), `coverage`/`unknowns`, `resources` (`cpu_count`, `load_1m`, `memory_available_mb`, `disk_free_mb`, `sampled_at`), `changed` since the last call, `items` (`outage`, `recovered`; the cadence is under Outage cadence below), `text` |
| `list [<machine>] [--dialogs] [--hint]` | the board (`text`), the compact `inventory`, and `rows` (one per server with `agents`, `state`, `next_step`); blocked first; every machine, connected or not. A Herdr row carries `labels` — the names a human sees (`pane` from the snapshot's panes, `tab`, `workspace`; empty ones and a tab's or workspace's own number left out) — and the board and `context` print the pane's name, else its tab's, in quotes beside the session, so the name a human gave a pane is on the board they read; a workspace label (usually what `open --label` wrote) stays in the row and addressable |
| `machines` | Herdr's saved list plus `~/.config/muse/machines.toml`, plus your MSP hosts (`muse hosts` over the transport CLI, with `TBH_AGENTS_SESSION_PROTOCOL` on; provider `msp`, source `msp`, the host's transport id as label and target): this machine's own advert, the hosts of your own login family (ids carrying your login as a dash-bounded token, `msp-<user>-<host>` and the like), every host you saved with `connect <host id>` or that an ssh row's label already names, every host holding a session this skill opened or adopted, and any host you name in the turn — never the rest of a shared directory, whose size is `msp.directory` (`advertising`, `shown`, `not_shown`) and one `notes` line; each with `provider`, `modes` (the modes it supports — an MSP host whose id an ssh-registered row already carries as its label or id is that one row with `msp` added — its sessions are read through its ssh provider, `open` on it prefers msp), `reachability` (an MSP host: `connected` when `muse hosts` says online, `unreachable` when offline — never a login), `note`, `next`; `shadowed` names directory rows Herdr also saves; with the flag on, `msp` (`state`, `note`, `hosts`) and, when the source is not available, one `notes` line saying why (no CLI, an older build without the `muse` verbs, a transport that does not answer) |
| `status <addr>` | the identity tuple, `live`, agent status, `identity_ok` / `identity_drift` for a handle; a session that no longer exists is `no_such_session` (exit 3) — "gone", while `provider_unreachable` (exit 6) is "unknown"; an agentless Herdr pane this skill holds (a shell pane, an adopted raw pane) is `status: no_agent` with `liveness_only: true`; an MSP session answers through host-manager's `status --mode msp --ref` (`group` beside `status`) |
| `read <addr> [--lines N] [--chars N] [--tail] [--source visible\|recent\|recent-unwrapped]` | the session's recent output (`text`, `lines`, the native `status`); `--tail` is the last `--lines` rows (default 60) of its visible screen as the engine drew them (blank rows and rule lines squeezed), chrome and all — you read it as you would in attach; an MSP session has no screen: `--tail` and the plain read are the same transport tail, `status` its `group` |
| `dialog <addr> [--lines N]` | what a blocked session is asking (Herdr) |
| `resources [<machine>]` | load, CPUs, memory, disk under `$HOME` on this host and every reachable machine (portable shell probes down the ladder) |
| `wait <addr> [--until s1,s2] [--duration S]` | Herdr `agent wait` on one session (never a poll): `waited` with `reached` true or false and the last `status`; default states idle, done or blocked; default duration 600 s, enforced by the helper (Herdr's own `--timeout` is a backstop 5 s behind it); a Herdr-side error is a failure envelope — `no_such_session` (exit 3) for a target Herdr does not know, exit 6 otherwise — never read out of Herdr's prose |
| `events [--interval S] [--once] [--kinds …] [--duration S] [--replay-baseline]` | the fleet stream for one Monitor (Herdr `events.subscribe` per reachable server; unreachable machines re-probed every `--interval`) |

| 动词 | 作用 |
| --- | --- |
| `doctor [--no-start] [--no-install]` | `healthy`（或 `needs_user_action` / `no_provider` / `provider_unreachable`）：`checks` 各行（`python`、`herdr`、`tmux`、`machines_directory`、`provider`）、每台机器的可达性、那一条 `next` 命令；会启动停着的 Herdr 服务器，并在无需密码时安装 tmux |
| `detect [--no-start] [--no-install]` | 本主机的提供方决策：`reason`（`herdr_reachable`、`herdr_started`、`herdr_down`、`tmux_available`、`tmux_installed`、`requested`、`no_provider`）、`providers` 各行（`installed`、`reachable`、`server`、`version`、`capabilities`） |
| `context [--reset]` | 一份摘要：`sessions` 各行（每行含 `identity` 与 `group`：`waiting-on-you`、`ready-for-review`、`working`、`idle`；你的 MSP 主机的会话来自对每台在线主机各执行一次 `muse sessions --host <host>`，每次四条、受每主机时限约束（`FLEET_MANAGER_MSP_LIST_TIMEOUT_S`，8 秒）——待处理的请求为 `waiting-on-you`，正在运行的轮次为 `working`，否则为 `idle`——并带有 `provider: msp`；不应答或未按时应答的主机为 `unreachable`，附传输层自身的消息或 "did not answer within N s (listing pending; ask again)"，绝不会是 "0 sessions"；陌生人的主机与会话绝不读取、绝不渲染）、`groups`、带可达性的机器（`connected`、`stale`、`unreachable`、`disabled`，或对 `connect` 在验证完成前中断的目录行标注 `unverified`——其 `next` 就是那次 `connect`，绝不是故障）、`coverage`/`unknowns`、`resources`（`cpu_count`、`load_1m`、`memory_available_mb`、`disk_free_mb`、`sampled_at`）、距上次调用以来 `changed` 的内容、`items`（`outage`、`recovered`；节奏见下文 Outage cadence 一节）、`text` |
| `list [<machine>] [--dialogs] [--hint]` | 面板（`text`）、紧凑的 `inventory` 以及 `rows`（每台服务器一行，含 `agents`、`state`、`next_step`）；阻塞者优先；涵盖每台机器，无论是否连接。Herdr 行携带 `labels`——人工可见的名称（`pane` 来自快照中的窗格、`tab`、`workspace`；空名称以及标签页或工作区自身的编号不列入）——且面板和 `context` 会在会话旁用引号打印窗格名，无窗格名时打印其标签页名，让人为窗格起的名字出现在他们所读的面板上；工作区标签（通常是 `open --label` 写入的内容）保留在行内且保持可寻址 |
| `machines` | Herdr 保存的列表加上 `~/.config/muse/machines.toml`，再加上你的 MSP 主机（通过传输 CLI 执行 `muse hosts`，并开启 `TBH_AGENTS_SESSION_PROTOCOL`；provider 为 `msp`，source 为 `msp`，以主机的传输 id 作为标签与目标）：包括本机自身的通告、你自己登录家族的主机（id 中以短横线界定的标记携带你的登录名，如 `msp-<user>-<host>` 等）、你用 `connect <host id>` 保存过的每台主机或 ssh 行标签已指名的主机、持有本技能所开或所接管会话的每台主机，以及你在本轮中点名的任何主机——绝不包含共享目录中的其余部分，其规模以 `msp.directory`（`advertising`、`shown`、`not_shown`）和一行 `notes` 表示；每台主机附 `provider`、`modes`（它支持的模式——若某 MSP 主机的 id 已被某条 ssh 注册行作为标签或 id 携带，则它就是那一行并加上 `msp`——其会话经由其 ssh 提供方读取，对其执行 `open` 时优先 msp）、`reachability`（MSP 主机：`muse hosts` 显示在线则为 `connected`，离线则为 `unreachable`——绝不归因于登录）、`note`、`next`；`shadowed` 指出 Herdr 也保存了的目录行；开关打开时还有 `msp`（`state`、`note`、`hosts`），以及当来源不可用时用一行 `notes` 说明原因（无 CLI、没有 `muse` 动词的旧版本、传输不应答） |
| `status <addr>` | 身份元组、`live`、代理状态、句柄的 `identity_ok` / `identity_drift`；不再存在的会话是 `no_such_session`（退出码 3）——即"已消失"，而 `provider_unreachable`（退出码 6）是"未知"；本技能持有的无代理 Herdr 窗格（shell 窗格、接管的原始窗格）为 `status: no_agent` 且 `liveness_only: true`；MSP 会话通过 host-manager 的 `status --mode msp --ref` 应答（`group` 显示在 `status` 旁） |
| `read <addr> [--lines N] [--chars N] [--tail] [--source visible\|recent\|recent-unwrapped]` | 会话的近期输出（`text`、`lines`、原生的 `status`）；`--tail` 是其可见屏幕的最后 `--lines` 行（默认 60），按引擎绘制的样子呈现（压缩空行与分隔线），界面装饰一并包含——就像在 attach 中那样阅读；MSP 会话没有屏幕：`--tail` 与普通读取是同一个传输层尾部，`status` 即其 `group` |
| `dialog <addr> [--lines N]` | 被阻塞的会话正在询问什么（Herdr） |
| `resources [<machine>]` | 本主机及每台可达机器上的负载、CPU、内存、`$HOME` 所在磁盘（沿阶梯逐级使用可移植的 shell 探测） |
| `wait <addr> [--until s1,s2] [--duration S]` | 对单个会话执行 Herdr `agent wait`（绝不是轮询）：返回 `waited`，附 `reached` 真或假以及最后的 `status`；默认状态为 idle、done 或 blocked；默认时长 600 秒，由本助手强制执行（Herdr 自身的 `--timeout` 作为落后 5 秒的兜底）；Herdr 侧错误是一个失败信封——Herdr 不认识的目标为 `no_such_session`（退出码 3），否则为退出码 6——绝不从 Herdr 的自然语言文本中推断 |
| `events [--interval S] [--once] [--kinds …] [--duration S] [--replay-baseline]` | 面向单个 Monitor 的全舰队事件流（每台可达服务器一个 Herdr `events.subscribe`；不可达机器每个 `--interval` 重新探测一次） |

### `fetch <machine> <path>` / 单文件取回

One report or library file home from a machine, by content hash.

从机器取回一份报告或一个库文件回家，依据内容哈希。

- The machine is asked for the file's sha256 first; the copy happens only
  when the home copy is missing or its own sha256 differs (the last hash is
  also kept in the state file). Success is `fetched` with `copied`,
  `unchanged`, `sha256`, `bytes`, `home`, `rung`.
  先询问该机器文件的 sha256；只有当本地副本缺失或其自身 sha256 不同时才执行复制（最近一次哈希也保存在状态文件中）。成功返回 `fetched`，附 `copied`、`unchanged`、`sha256`、`bytes`、`home`、`rung`。
- The copy lands under `<state dir>/home/<machine>/<path>` and nowhere
  else: no destination flag (the agent's cwd is usually a repository; a
  human moves the file).
  副本落在 `<state dir>/home/<machine>/<path>` 下，别无他处：没有目标标志（代理的 cwd 通常是某个仓库；由人来移动该文件）。
- A symlink is `refused`; a directory or missing path is `usage`; above the
  copy cap, `usage` with the cap named; `local` is `usage` (read it in
  place).
  符号链接是 `refused`；目录或不存在的路径是 `usage`；超过复制上限时返回 `usage` 并指明上限；`local` 是 `usage`（就地读取即可）。
- A box without `sha256sum`/`shasum`, a copy that does not decode, or bytes
  whose hash differs write nothing (`provider_unreachable`).
  没有 `sha256sum`/`shasum` 的机器、无法解码的副本、或哈希不一致的字节，都不会写入任何东西（`provider_unreachable`）。
- Code never travels this way: it comes home through a PR.
  代码绝不走这条路：它通过 PR 回家。

## Steer verbs (allow-listable by subcommand) / 操纵类动词（可按子命令列入允许清单）

| verb | does |
| --- | --- |
| `send <addr> <text> [--automated]` | a notification only a human watching the pane sees (Herdr `notification show`; tmux `display-message`); never types, and the agent never receives it — never `sent`: `notified` when Herdr reports it shown, `not_shown` for a tmux status-line message (it reaches only an attached client) or a Herdr `shown: false`; either says `nothing was typed` in `message` and its `next` is the `--type` form |
| `approve <addr> [--key K] [--force]` / `deny <addr> …` | answers a recognised y/N or numbered dialog (Herdr); refused when the session is not blocked or the dialog is unreadable (`--key` after reading it). `approve` reads the dialog first: Enter confirms the highlighted choice (a `❯`/`›` row, or a bare `>` only on a numbered row — Codex's `> You are in <dir>` banner is never the choice), so when that choice is not the affirmative one it refuses (`code: agent_blocked`, `highlighted`) and `next` is the attach command. When Herdr calls the session idle but the screen shows a dialog (Codex's directory trust, a fresh Muse trust prompt in some drives), the refusal carries `code: agent_blocked` and `next` is the one command that answers it: `approve <addr> --force` when the affirmative choice is highlighted, else the attach command |
| `send <addr> --keys <key…>` | named keys for any other dialog (Herdr), under the composer guard; a row of the dialog's own choice block is not held text, so keys go through to answer it — a line a person typed is held text whatever shape it has |

| 动词 | 作用 |
| --- | --- |
| `send <addr> <text> [--automated]` | 一种只有正盯着窗格的人才能看到的通知（Herdr `notification show`；tmux `display-message`）；绝不键入任何内容，代理也永远收不到——成功词绝不是 `sent`：Herdr 报告已显示时为 `notified`，tmux 状态行消息（只会到达已连接的客户端）或 Herdr `shown: false` 时为 `not_shown`；两种情况都在 `message` 中写明 `nothing was typed`，且其 `next` 是 `--type` 形式 |
| `approve <addr> [--key K] [--force]` / `deny <addr> …` | 回答一个可识别的 y/N 或编号对话框（Herdr）；当会话并非阻塞状态或对话框不可读时拒绝（读完对话框后才定 `--key`）。`approve` 先读取对话框：Enter 确认高亮的选项（`❯`/`›` 所在行，或仅编号行上的裸 `>`——Codex 的 `> You are in <dir>` 横幅绝不是选项），因此当该选项不是肯定项时它会拒绝（`code: agent_blocked`，附 `highlighted`），`next` 是 attach 命令。当 Herdr 认为会话空闲而屏幕上却显示对话框（Codex 的目录信任、某些驱动器上 Muse 新出现的信任提示）时，拒绝携带 `code: agent_blocked`，且 `next` 是回答它的那一条命令：肯定项高亮时为 `approve <addr> --force`，否则为 attach 命令 |
| `send <addr> --keys <key…>` | 为其他任何对话框发送指定的按键（Herdr），受输入行守卫约束；对话框自身选项块中的一行不算被占用的文本，因此按键可以直达以回答它——而人键入的一行无论什么形状都是被占用的文本 |

## Guarded verbs (permission prompt) / 受守卫动词（需权限批准）

| verb | does |
| --- | --- |
| `adopt <machine[:server]>/<ref> [--name N]` | mints a handle for a session this skill did not open, with its identity recorded (`--name` is a Herdr agent rename); a name another session already answers to is `name_taken` (exit 3) naming the holder, and nothing is adopted or renamed |
| `attach <addr>` | the command a human runs to sit in front of the session; nothing is executed |
| `stop <addr>` | interrupts the current turn (ctrl-c); the session stays; an agentless Herdr pane gets `pane send-keys c-c` into its shell; an MSP session has no ctrl-c — `unsupported_by_provider` naming `close` |
| `close <addr> [--confirm "<the human's words>"]` | closes the pane / kills the tmux session; a live session (working, blocked, a non-shell program in any of its panes) is `session_live` (exit 3) without `--confirm`; on an MSP session host-manager's `close --confirm` ends its work (the running turn interrupted, its tasks stopped) and the receipt says the host keeps the row listed idle until it unloads it — gone is the host's own `not_found` (`no_such_session`); a repeat with the words is `closed` again |
| `forget <label> [--confirm "<words>"]` | drops a `machines.toml` row (`forgotten`); `session_live` while it has live sessions unless confirmed; a Herdr-saved machine is Herdr's (`next` is `herdr machine remove <id>` — Herdr 0.9.0 takes the id from `machine list --json`, not the label) |

| 动词 | 作用 |
| --- | --- |
| `adopt <machine[:server]>/<ref> [--name N]` | 为一个非本技能开启的会话铸造句柄，并记录其身份（`--name` 是 Herdr 的代理重命名）；若另一个会话已应答同名，则为 `name_taken`（退出码 3）并指名持有者，且不做任何接管或重命名 |
| `attach <addr>` | 供人坐到会话面前所运行的命令；不执行任何东西 |
| `stop <addr>` | 中断当前轮次（ctrl-c）；会话保留；无代理的 Herdr 窗格会向其 shell 发送 `pane send-keys c-c`；MSP 会话没有 ctrl-c——返回 `unsupported_by_provider` 并指名 `close` |
| `close <addr> [--confirm "<the human's words>"]` | 关闭窗格 / 杀掉 tmux 会话；存活会话（工作中、被阻塞、任一窗格内有非 shell 程序）在无 `--confirm` 时返回 `session_live`（退出码 3）；在 MSP 会话上，host-manager 的 `close --confirm` 结束其工作（正在运行的轮次被中断，其任务被停止），且回执说明主机把该行保持为空闲列出状态直至卸载它——gone 则是主机自身的 `not_found`（`no_such_session`）；带着那句话重复执行会再次返回 `closed` |
| `forget <label> [--confirm "<words>"]` | 删除一条 `machines.toml` 行（`forgotten`）；其下还有存活会话时返回 `session_live`，除非已确认；Herdr 保存的机器归 Herdr 管（`next` 是 `herdr machine remove <id>`——Herdr 0.9.0 从 `machine list --json` 取 id，而不是标签） |

### `send <addr> <text> --type [--wait] [--until …] [--timeout ms] [--automated] [--no-verify] [--verify-seconds S] [--steer]` / 以提示词形式键入文本

On an MSP session `--type` is a message the agent receives as its next turn
(`delivery: message`) and `--steer` steers the running turn instead
(`delivery: steer`), both through host-manager's `send`; `--wait`, `--until`,
`--timeout` and the verify flags are listed under `not_applied`. A bare
`send` or `--keys` there is `unsupported_by_provider`: no pane, no composer.

在 MSP 会话上，`--type` 是代理将作为其下一轮收到的消息（`delivery: message`），而 `--steer` 则改为操纵正在运行的轮次（`delivery: steer`），两者都经由 host-manager 的 `send`；`--wait`、`--until`、`--timeout` 与各校验标志列在 `not_applied` 之下。在那里，裸 `send` 或 `--keys` 是 `unsupported_by_provider`：没有窗格，没有输入行。

Types the text into the session as a prompt: the form for the agent, whether
an instruction, a steer, a question or a reminder (`--type`; relayed text
adds `--automated`). A permission prompt in every
caller: the shipped allow-list template carries no text-`send` glob because
a glob cannot separate the notification form from `--type` and an allow
match wins (the notification form itself stays allow-listable by
subcommand).

把文本作为提示词键入会话：这是面向代理的形式，无论是指令、操纵、提问还是提醒（`--type`；转发的文本加 `--automated`）。对每个调用方都是一次权限提示：随附的允许清单模板不含针对文本 `send` 的 glob，因为 glob 无法把通知形式与 `--type` 区分开，而允许匹配优先（通知形式本身仍可按子命令列入允许清单）。

【评论】模板拒绝为文本 `send` 提供 glob，是因为允许匹配优先于询问；通配符会把"仅通知"的授权放大为"代写提示词"的授权。

- Refused (`composer_not_empty`, exit 3) when the composer already holds
  text — when the composer is not empty, wait or tell the human, never claim
  delivery — and refused on a blocked session (answer the dialog first). On
  Herdr, "held text" that is a dialog's choice row while Herdr calls the
  session idle is the dialog shape instead: `refused` with `code:
  agent_blocked`, the `dialog` rows, and `next` the one command that answers
  it (`approve <addr> --force`, or the attach command when the highlighted
  choice is not the affirmative one). On tmux a
  dialog on screen is `refused` with `code: agent_blocked` and the `dialog`
  rows; `next` is the attach command — a person answers it in attach, the
  helper has no key verb. A dialog is host-manager's reading: a trust,
  permission or confirmation phrase (`press enter to continue`, `Enter to
  confirm`), a y/n wait on the last line, a numbered selector or an
  unnumbered yes/no pair with a `>`/`›`/`❯` cursor on one row (Claude
  Code's folder trust), or a decision question over numbered choices. The
  dialog scan runs on the whole visible screen before the composer row is
  judged, whatever row the cursor sits on (a redrawn Codex parks it on a
  blank row under its trust dialog). A pane whose program exited
  (`Pane is dead`) is `no_such_session`, never held text; `next` is `close`.
  当输入行中已有文本时拒绝（`composer_not_empty`，退出码 3）——输入行非空时，等待或告知人，绝不声称已送达——并且在被阻塞的会话上也拒绝（先回答对话框）。在 Herdr 上，当 Herdr 认为会话空闲而所谓"被占用文本"其实是对话框的选项行时，按对话框形状处理：返回 `refused`，附 `code: agent_blocked` 与 `dialog` 各行，`next` 是回答它的那一条命令（`approve <addr> --force`，或当高亮选项不是肯定项时为 attach 命令）。在 tmux 上，屏幕上的对话框返回 `refused`，附 `code: agent_blocked` 与 `dialog` 各行；`next` 是 attach 命令——由人在 attach 中回答，本助手没有按键动词。对话框由 host-manager 判定：信任、权限或确认语句（`press enter to continue`、`Enter to confirm`）、最后一行上的 y/n 等待、编号选择器、某一行上带 `>`/`›`/`❯` 光标的无编号 yes/no 对（Claude Code 的目录信任），或对编号选项提出的决策性问题。对话框扫描在判定输入行之前运行于整个可见屏幕，无论光标停在哪个行（重绘的 Codex 会把光标停在其信任对话框下方的空行上）。程序已退出的窗格（`Pane is dead`）是 `no_such_session`，绝不是被占用文本；`next` 是 `close`。
- A swallowed prompt gets one Enter and `needed_enter: true`.
  被吞掉的提示词补一次 Enter 并返回 `needed_enter: true`。
- **`typed` is submitted, not taken:** a submitted line's `next` is `read
  <addr> --tail`; run it a few seconds later and tell the user in one line
  what the pane shows — took it and is doing X / no reaction yet, read
  again in N s / for a shell pane, what the command printed and that it
  exited. Never leave a steer at `typed`; `submitted: false` keeps `read
  <addr> before any retry` as `next` instead.
  **`typed` 表示已提交，不等于已接受：**已提交行的 `next` 是 `read <addr> --tail`；几秒后运行它，并用一行话告诉用户窗格显示什么——已接受并正在做 X / 尚无反应，N 秒后再读 / 对 shell 窗格，命令打印了什么以及它已退出。绝不让操纵停留在 `typed`；`submitted: false` 时 `next` 改为 `read <addr> before any retry`。
- `--automated` prefixes `[automated, not the user, approves nothing]`.
  `--automated` 会在文本前加 `[automated, not the user, approves nothing]` 前缀。
- An agentless Herdr pane (`open --engine bash`, an adopted raw pane) is a
  shell, not an agent: the line runs in its shell through Herdr's own
  `pane run` (what `open` types its command with), never `agent prompt`;
  the command line is judged from the last screen row (free when a prompt
  character ends it, `composer_not_empty` otherwise); one line per send;
  `--automated` rides as a trailing `# …` comment so the command still
  runs; the envelope says `via: pane run`, `status: no_agent`,
  `liveness_only: true`.
  无代理的 Herdr 窗格（`open --engine bash`、接管的原始窗格）是 shell 而非代理：该行经由 Herdr 自身的 `pane run` 在其 shell 中运行（`open` 就是用它键入命令的），绝不是 `agent prompt`；命令行按最后一屏行判定（以提示符字符结尾即为空闲，否则 `composer_not_empty`）；每次发送一行；`--automated` 作为结尾的 `# …` 注释搭车，命令仍能运行；信封注明 `via: pane run`、`status: no_agent`、`liveness_only: true`。
- On tmux the text is typed, Enter follows as its own step after the text
  landed (an Enter in the same burst is a pasted newline to the Muse TUI),
  and `submitted` is what the pane shows afterwards: `needed_enter` when a
  second Enter was pressed; `submitted: false` with `read … before any
  retry` as `next` when the line still sits in the composer. That read is
  keyed by the session's engine, like the guard before it, so an engine's
  empty-composer chrome (Codex's placeholder, Claude's transcript echo) is
  never read as a line the pane kept.
  在 tmux 上先键入文本，Enter 在文本落定后作为独立步骤跟随（同一批次里的 Enter 对 Muse TUI 而言是粘贴的换行）；`submitted` 取决于其后窗格的显示：若按了第二次 Enter 则为 `needed_enter`；若该行仍留在输入行中，则 `submitted: false` 且 `next` 为 `read … before any retry`。这次读取按会话的引擎区分键，如同其前的守卫一样，因此引擎空输入行的界面装饰（Codex 的占位符、Claude 的转录回显）绝不会被误读为窗格保留的一行。

### `open [<machine[:server]>] [--engine K] [--cwd D] [--name N] [--prompt-file PATH|-] [--worktree PATH] [--engine-arg=FLAG …] [--purpose TEXT] [--label TEXT] [--exact-name] [--unattended] [--timeout ms]` / 打开新会话

Zero required arguments: `local`, `muse`, the repository root (else the
current directory), an auto name (`<dir>-<n>`; a taken name gets `-2`/`-3`,
`--exact-name` refuses with `name_taken`). Success is `opened` with
`identity`, `created: true` and a `receipt`.

无必填参数：机器默认 `local`、引擎默认 `muse`、目录默认仓库根（否则当前目录）、名称自动生成（`<dir>-<n>`；被占用则依次得 `-2`/`-3`，`--exact-name` 会以 `name_taken` 拒绝）。成功返回 `opened`，附 `identity`、`created: true` 和一份 `receipt`。

- Herdr: `workspace create` (or `worktree open --path` for `--worktree`),
  `agent start --kind` (engine args after `--`; a kind Herdr does not
  manage runs as the command in the pane), then the brief with
  `agent prompt --wait`. A shell kind (`bash`, `zsh`, `sh`, …) runs through
  `pane run` and nothing waits for an agent Herdr will never detect: the
  answer is `opened` with a handle, `status: no_agent`, `liveness_only:
  true`, `via: pane run`; the brief is not sent (`send --type` runs a
  line). `list`/`context` show the pane as a liveness-only row (the tmux
  shape) for as long as this skill holds its handle; any other command that
  never becomes an agent is still `failed` (exit 6) with the pane left to
  read and close.
  Herdr：先 `workspace create`（`--worktree` 时为 `worktree open --path`），再 `agent start --kind`（引擎参数放在 `--` 之后；Herdr 不管理的 kind 会作为命令在窗格中运行），然后用 `agent prompt --wait` 发送任务简报。shell 类 kind（`bash`、`zsh`、`sh` 等）经由 `pane run` 运行，且不会等待 Herdr 永远检测不到的代理：结果是带句柄的 `opened`，附 `status: no_agent`、`liveness_only: true`、`via: pane run`；简报不发送（`send --type` 负责运行一行）。只要本技能还持有其句柄，`list`/`context` 就把该窗格显示为仅存活行（tmux 形状）；其他任何永远不会变成代理的命令仍是 `failed`（退出码 6），窗格留着供读取与关闭。
- tmux: `new-session -d` under `env -u HERDR_* -u TMUX` (a Muse engine gets
  `--workspace <cwd>`); an engine that exits at once is `failed` (exit 6)
  with its last line, nothing left behind; `--worktree`/`--label` are
  `unsupported_by_provider`; the brief is not sent (no readiness signal).
  tmux：在 `env -u HERDR_* -u TMUX` 之下执行 `new-session -d`（Muse 引擎加 `--workspace <cwd>`）；立即退出的引擎为 `failed`（退出码 6）并附其最后一行，不留残余；`--worktree`/`--label` 为 `unsupported_by_provider`；简报不发送（没有就绪信号）。
- `--unattended` adds `--yolo` for a Muse engine and nothing for any other
  engine (pass that engine's own flag with `--engine-arg`); default is the
  engine's normal permission prompts; the envelope carries `unattended` and
  `posture` (the flags actually added).
  `--unattended` 对 Muse 引擎追加 `--yolo`，对其他引擎不加任何东西（用 `--engine-arg` 传该引擎自己的标志）；默认是引擎正常的权限提示；信封携带 `unattended` 与 `posture`（实际追加的标志）。
- An MSP host (`modes` carries `msp`): host-manager's `open --host <host>
  --cwd <dir>` through the transport, the brief as the first turn; `--cwd`
  is required (`usage` naming it: that machine's directories are not this
  one's), `--engine` other than `muse`, `--worktree`, `--label` and
  `--engine-arg` are `unsupported_by_provider`. Success carries `mode:
  msp`, `mode_line`, `attach` (the transport's tail command), the handle,
  `session_receipt` (the start and the brief) and `via: host-manager open
  --mode msp`; a host that stopped advertising or answering between the
  listing and the open is `provider_unreachable` (exit 6) with `created:
  false`, and every other machine answers as before. Every `open` receipt,
  on every provider, carries `mode` (`herdr` | `tmux` | `msp`).
  MSP 主机（`modes` 含 `msp`）：经由传输执行 host-manager 的 `open --host <host> --cwd <dir>`，简报作为第一轮；`--cwd` 必填（`usage` 并指名它：那台机器的目录不是本机的目录），`muse` 以外的 `--engine`、`--worktree`、`--label` 与 `--engine-arg` 均为 `unsupported_by_provider`。成功携带 `mode: msp`、`mode_line`、`attach`（传输的 tail 命令）、句柄、`session_receipt`（启动与简报）以及 `via: host-manager open --mode msp`；在列目录与打开之间停止通告或应答的主机为 `provider_unreachable`（退出码 6）且 `created: false`，其他所有机器照旧应答。每个 `open` 回执，在任何提供方上都携带 `mode`（`herdr` | `tmux` | `msp`）。
- After a timeout, run `list` before any second `open`: the session is
  usually there under its name, and a second `open` makes a second session.
  超时之后，先运行 `list` 再考虑第二次 `open`：会话通常已在，用其名字即可找到；第二次 `open` 会造出第二个会话。
- No `--dry-run`: `progress` names each step as it happens and `close`
  undoes the result.
  没有 `--dry-run`：`progress` 随每步发生即时指名，`close` 可撤销结果。

### `connect <ssh-target|label> [--label N] [--mode herdr|tmux] [--session S] [--no-login]` / 连接远程机器

One command, one `progress` line per step; it stops at the first thing
wrong with `next`. `via` says `herdr` or `ssh-master`; `remote` carries what
that path observed.

一条命令，每步一行 `progress`；在第一处出错即停止并给出 `next`。`via` 标明 `herdr` 或 `ssh-master`；`remote` 携带该路径观测到的内容。

- With Herdr on this host: `herdr machine add <target> --label N
  [--remote-session S]` (the target first: Herdr 0.9.0's parser rejects
  options-first argv) records the machine (Herdr's list is authoritative,
  no directory row) with Herdr's own login, which Herdr closes before the
  add returns. Then an answering forward is reused with no ssh; else a
  master that already answers (this skill's, or one the user's ssh config
  keeps) carries the forward with no login; else this skill opens its one
  master, and the remote server is started or forwarded. Login accounting:
  a fresh host costs Herdr's login plus at most one of this skill's; a
  reconnect costs at most one and usually none.
  本机有 Herdr 时：执行 `herdr machine add <target> --label N [--remote-session S]`（目标在前：Herdr 0.9.0 的解析器拒绝选项在前的 argv）记录机器（Herdr 的列表是权威，不写目录行），登录用 Herdr 自己的登录，Herdr 会在 add 返回前关闭它。随后，已应答的转发直接复用、不再走 ssh；否则由一个已在应答的 master（本技能的，或用户 ssh 配置中保留的）承载转发、无需登录；否则本技能开启自己唯一的那条 master，并启动或转发远端服务器。登录核算：新主机花费 Herdr 登录加至多一次本技能的登录；重连至多一次，通常为零。
- A failed add is `herdr_add_failed` (exit 6) with the Herdr command as
  `next`, never a silent tmux fallback. An add that exits 0 but whose
  `herdr machine list` read-back fails is `provider_unreachable` with
  `connect <label>` as `next` (never a second add).
  add 失败是 `herdr_add_failed`（退出码 6），`next` 为该 Herdr 命令，绝不静默回退 tmux。add 退出码为 0 但 `herdr machine list` 读回失败的，是 `provider_unreachable`，`next` 为 `connect <label>`（绝不是第二次 add）。
- Without Herdr, or with `--mode tmux`: save to `machines.toml`, open
  the ssh master (the one interactive login, one second factor), verify the
  remote provider, start a stopped remote Herdr server, forward its socket,
  record. A box without tmux gets it installed when that needs no password,
  else the install command as `next` (exit 5).
  没有 Herdr 或指定 `--mode tmux` 时：保存到 `machines.toml`，打开 ssh master（唯一的一次交互式登录、一次第二因素），验证远端提供方，启动停着的远端 Herdr 服务器，转发其套接字，记录。没有 tmux 的机器在无需密码时安装 tmux，否则以安装命令作为 `next`（退出码 5）。
- A saved label: re-forward over the live master, starting a stopped remote
  Herdr server over it first (one progress line, never a question).
  已保存的标签：在活着的 master 上重新转发，先经它启动停着的远端 Herdr 服务器（一行 progress，绝不是提问）。
- An MSP host: nothing to log in to (a transport id is not an ssh target);
  `connected` when `muse hosts` lists it online, `provider_unreachable`
  when offline, `next` is `open <host> --cwd <dir>`; a host that is not
  yours is saved to `machines.toml` as provider `msp`, so the digest reads
  it with yours from then on, and `forget <host id>` drops that row (one
  of your own hosts is not a row: it leaves when it stops advertising).
  MSP 主机：没有可登录的东西（传输 id 不是 ssh 目标）；`muse hosts` 列为在线则 `connected`，离线则 `provider_unreachable`，`next` 是 `open <host> --cwd <dir>`；不属于你的主机以 provider `msp` 保存进 `machines.toml`，此后摘要会把它与你的主机一起读取，`forget <host id>` 删除该行（你自己的主机不是一行：它停止通告时自然离开）。
- `--label <other>` for a target the directory holds verified tmux-only
  under another label is `usage` naming the saved label. `--no-login` never
  opens a login (exit 5 or 6 with the command instead).
  对目录中在另一标签下已验证为仅 tmux 的目标使用 `--label <other>` 是 `usage` 并指名已保存的标签。`--no-login` 绝不打开登录（改为退出码 5 或 6 并附命令）。

## Provider rule and remote ladder / 提供方规则与远程阶梯

Herdr when installed and its server answers (started when down, never
installed); tmux otherwise (installed when that needs no password; else the
exact command in `next`); `--mode` overrides; Windows: `no_provider`. A
machine that advertises MSP is provider `msp` whatever the pin: its
sessions are host-manager's mode C, reached through host-manager's helper
(`FLEET_MANAGER_HOST_MANAGER`, else the `host-manager` skill beside this
one) — one call of it per verb, the two skills one record of the session.
No `ssh` ever goes to an MSP host id, and no session or command id is minted
here: host-manager's provider does that. `resources` does not read an MSP
host (no shell; `reachability: unsupported`), `events` does not watch one
(`context` does).

已安装且其服务器应答时用 Herdr（停着就启动，绝不安装）；否则用 tmux（无需密码时安装；否则把确切命令放进 `next`）；`--mode` 可覆盖；Windows 为 `no_provider`。通告 MSP 的机器无论钉定什么都以 `msp` 为提供方：其会话是 host-manager 的模式 C，经由 host-manager 的助手（`FLEET_MANAGER_HOST_MANAGER`，否则用本技能旁边的 `host-manager` 技能）访问——每个动词只调用它一次，两个技能共享同一份会话记录。`ssh` 绝不会发往 MSP 主机 id，这里也不铸造任何会话或命令 id：那是 host-manager 提供方的职责。`resources` 不读取 MSP 主机（没有 shell；`reachability: unsupported`），`events` 不监视它（由 `context` 负责）。

A machine is reached down one ladder: the forwarded Herdr socket (a
`fleet-manager@<home host>` pane on that server runs the command and its
output comes back through the pane) → the existing ssh ControlMaster
(`BatchMode=yes`, `ControlMaster=no`, never a fresh login) → `unreachable`
with `connect` as the next command. Output comes home through the pane or
the master and stops above `FLEET_MANAGER_COPY_CAP_BYTES` (4 MiB) on either
rung with the cap named; this host owns the record.

到达一台机器只经一条阶梯：转发的 Herdr 套接字（该服务器上的 `fleet-manager@<home host>` 窗格运行命令，其输出经窗格返回）→ 既有的 ssh ControlMaster（`BatchMode=yes`、`ControlMaster=no`，绝不重新登录）→ `unreachable`，`next` 为 `connect`。输出经窗格或 master 回家，在任一级上超过 `FLEET_MANAGER_COPY_CAP_BYTES`（4 MiB）即止并指明上限；记录归本主机所有。

Outage cadence: a machine that stops answering
is skipped for two minutes (a successful `connect <label>` ends that window
early) and keeps its last group in `context` as `stale`; one `outage` item
after ten minutes, one `recovered` item when it returns.

中断节奏：停止应答的机器被跳过两分钟（成功的 `connect <label>` 会提前结束该窗口），并在 `context` 中把其最后的分组保持为 `stale`；十分钟后产生一条 `outage` 项，恢复时产生一条 `recovered` 项。

Copying home is lazy. No verb copies a file per round: `context` and `list`
pull status only (one snapshot per machine). `fetch` is the only copy, by
content hash, and a repeated `fetch` of an unchanged file is one round trip
and no bytes. Symlinks are never followed; code comes home through a PR.

拷贝回家是惰性的。没有任何动词每轮都拷贝文件：`context` 与 `list` 只拉取状态（每台机器一个快照）。`fetch` 是唯一的拷贝，按内容哈希进行，对未变文件的重复 `fetch` 只花一次往返、零字节。符号链接绝不被跟随；代码通过 PR 回家。

## Environment / 环境变量

`TBH_AGENTS_SESSION_PROTOCOL` (host-manager's flag: on lists MSP hosts and opens on them; off is the old path byte for byte), the transport CLI override host-manager honours (named in its `references/mode-msp.md`), `FLEET_MANAGER_HOST_MANAGER` (host-manager's skill directory; default: the sibling `host-manager`), `FLEET_MANAGER_MSP_HELPER_TIMEOUT_S` (300: one host-manager call), `FLEET_MANAGER_MSP_LIST_TIMEOUT_S` (8: one host's session listing inside a digest; past it the host is "not answered yet"); `HERDR_BIN_PATH`, `HERDR_SOCKET_PATH` (Herdr's own); `FLEET_MANAGER_SSH`, `_DIR` (forwards and masters, `/tmp/fleet-manager-<uid>`), `_STATE` (`~/.local/share/muse/fleet-manager/state.json`), `_MACHINES` (`~/.config/muse/machines.toml`), `_SESSION`, `_REMOTE_SOCKET`, `_TMUX_SOCKET` (a private tmux server), `_ASKED_BY`, `_CONNECT_TIMEOUT_S` (25; a login step gets at least the smaller of 5 s and that, however little budget is left), `_SERVER_WAIT_S` (12: how long a just-started remote server may take to answer over the forward), `_COPY_CAP_BYTES`, `_CALL_TIMEOUT_S` / `_SSH_TIMEOUT_S` / `_API_TIMEOUT_S`, `_NOW` (injected clock for tests).

`TBH_AGENTS_SESSION_PROTOCOL`（host-manager 的开关：开时列出 MSP 主机并可在其上打开；关时走旧路径、逐字节一致）、host-manager 认可的传输 CLI 覆盖项（在其 `references/mode-msp.md` 中指名）、`FLEET_MANAGER_HOST_MANAGER`（host-manager 的技能目录；默认：同级的 `host-manager`）、`FLEET_MANAGER_MSP_HELPER_TIMEOUT_S`（300：一次 host-manager 调用）、`FLEET_MANAGER_MSP_LIST_TIMEOUT_S`（8：摘要内一台主机的会话列举时限；超时后该主机为"尚未应答"）；`HERDR_BIN_PATH`、`HERDR_SOCKET_PATH`（Herdr 自身）；`FLEET_MANAGER_SSH`、`_DIR`（转发与 master，`/tmp/fleet-manager-<uid>`）、`_STATE`（`~/.local/share/muse/fleet-manager/state.json`）、`_MACHINES`（`~/.config/muse/machines.toml`）、`_SESSION`、`_REMOTE_SOCKET`、`_TMUX_SOCKET`（一个私有 tmux 服务器）、`_ASKED_BY`、`_CONNECT_TIMEOUT_S`（25；登录步骤至少获得 5 秒与该值中较小者，无论剩余预算多少）、`_SERVER_WAIT_S`（12：刚启动的远端服务器经转发应答可等待的时长）、`_COPY_CAP_BYTES`、`_CALL_TIMEOUT_S` / `_SSH_TIMEOUT_S` / `_API_TIMEOUT_S`、`_NOW`（测试用注入时钟）。
