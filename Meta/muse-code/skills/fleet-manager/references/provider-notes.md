<!-- BILINGUAL-EN-ZH -->
# Provider notes: verified behaviours by version / 提供方说明：按版本核验的行为

What this skill relies on, with how each item was verified. A row without a
verification is an assumption and says so. Re-verify a row when the provider
version changes; the helper prints the version it saw (`doctor`, `list`).

本技能所依赖的内容，以及每一项的核验方式。没有核验的行属于假设，并会如实标注。提供方版本变化时需重新核验；辅助工具会打印其观测到的版本（`doctor`、`list`）。

## Herdr / Herdr

| version | behaviour | verified how |
| --- | --- | --- |
| 0.9.0 | The socket API answers one request per connection and closes it; `events.subscribe` holds the connection open and streams `{"event","data"}` lines. | The helper's fake server mirrors it; live `events` runs (#36636 evidence) |
| 0.9.0 | Closing a workspace or tab emits ONE `workspace_closed` / `tab_closed` and no per-pane `pane_closed`; the helper folds the container event. | Measured live (#36636); `test_fleet_lifecycle.py` pins it |
| 0.9.0 | `pane.agent_status_changed` is scoped per pane: a pane that appears after the subscription needs its own subscription (the helper re-opens the stream). | Measured live (#36636) |
| 0.9.0 | `agent start` refuses a kind it does not manage with exit 2 and plain-text stderr `unsupported interactive agent kind: <kind>` (no JSON); the helper then runs the kind as the command in the pane and waits for detection. | Measured live (#36636) |
| 0.9.0 | `agent prompt` on a blocked agent is refused with `agent_blocked` before any input; `--wait` from a non-working state needs an observed working/blocked state within 5000 ms, else `agent_prompt_stalled`. | `herdr agent prompt --help` text and live drives |
| 0.9.0 | A freshly started agent can take a prompt into its composer without submitting (Muse 1.0.3, Claude Code): the helper verifies and presses Enter once (`needed_enter: true`). | Measured live (#31985) |
| 0.9.0 | A restored pane is not the prior process, and a pane id can be reissued after a server restart; the ids do not reliably restart from `w1` — after `herdr session stop <name>` and a restart over the master, a named session kept counting (`w5` → `w6`). The identity tuple (`server`, `ref`, `cwd`, `engine`) exists because of the reuse, not the numbering. | the herdr-projects stage-2/3 findings (restart from `w1`); the #38715 round-8 lane FM2 drive on a named session (continued numbering) |
| 0.9.0 | A saved machine whose ssh master is up can refuse a second ssh session (`Session open refused by peer`): the Herdr bridge holds the single slot. Reach it over the master (`ssh -O forward`), never with a fresh `ssh <host>`. | Measured on the fleet (#31985, the SLOT_FACT text) |
| 0.9.0 | `pane wait-output <pane> --regex <re> --timeout <ms>` searches the recent snapshot immediately, then polls; `--source recent-unwrapped` is available. | `herdr pane wait-output --help` |
| 0.9.0 | `notification show <TITLE> --body <TEXT>` shows a server-wide notification (no per-pane target). | `herdr notification show --help` |
| 0.9.0 | `worktree open --path <PATH> --label <TEXT> --no-focus` opens an existing git worktree as a workspace. | `herdr worktree open --help` |
| 0.9.0 | `workspace create --label <TEXT> --cwd <PATH> --no-focus` returns the root pane; `--env KEY=VALUE` exists but the helper never passes environment (launch-environment inheritance stays deferred). | `herdr workspace create --help`; `test_fleet.py` pins no `--env` |
| 0.9.1 | A prompt that arrives while text is half-typed merges with and submits it. The helper reads the composer first and refuses (exit 3) when it holds text. | Reported by the herdr-projects notes and adopted by the identity-and-safety rule; NOT re-verified here (this host runs 0.9.0) — treat as an assumption until a 0.9.1 drive confirms it |
| 0.9.0 | A fresh Muse session sitting on its workspace-trust dialog was `blocked` in one drive and `idle` in the next two (`agent wait --until blocked` returned idle); `dialog` shows the prompt either way. When Herdr did not flag it, `approve --force` answers what `dialog` showed. | Loopback black-box on this host, three runs |
| 0.9.0 | `machine add <SSH_TARGET> --label <LABEL> [--remote-session <NAME>]` prepares the remote server and saves the profile; the target comes FIRST — options-first argv gets `usage: herdr machine add <ssh-target> --label <label> [--remote-session <name>]` and exit 2, although `--help` prints clap-style `[OPTIONS] --label <LABEL> <SSH_TARGET>`. Its ssh runs over a private control socket (`-F /tmp/herdr-ssh-<pid>-0/config -S /tmp/herdr-ssh-<pid>-0/ctl`) that it `-O exit`s before returning, so nothing of its login survives for `connect` to ride. Its prompts and output go to the terminal, so `connect` routes them to stderr and keeps stdout one object. | Both argv orders run against the real binary in an isolated HOME (the fake in `test_fleet.py` rejects options-first the same way); the ssh argv captured with a logging `ssh` on PATH |
| any | An 0.8.x client has no `machine` verb (`unknown command`): the fleet is `local` only. | `list_machines` handles it; not re-verified since 0.9.0 |

| 版本 | 行为 | 核验方式 |
| --- | --- | --- |
| 0.9.0 | 套接字 API 对每个连接只应答一个请求然后关闭；`events.subscribe` 保持连接打开并流式输出 `{"event","data"}` 行。 | 辅助工具的假服务器复刻了该行为；实时 `events` 运行（#36636 证据） |
| 0.9.0 | 关闭工作区或标签页只会发出一条 `workspace_closed` / `tab_closed`，而没有按窗格的 `pane_closed`；辅助工具会折叠该容器事件。 | 实测（#36636）；`test_fleet_lifecycle.py` 将其固定 |
| 0.9.0 | `pane.agent_status_changed` 按窗格限定作用范围：订阅之后才出现的窗格需要自己的订阅（辅助工具会重新打开流）。 | 实测（#36636） |
| 0.9.0 | `agent start` 对其不管理的 kind 会拒绝执行，以退出码 2 和纯文本 stderr `unsupported interactive agent kind: <kind>`（无 JSON）；辅助工具随后把该 kind 作为命令在窗格中运行并等待检测。 | 实测（#36636） |
| 0.9.0 | 对被阻塞的代理执行 `agent prompt` 会在任何输入之前被以 `agent_blocked` 拒绝；从非工作状态使用 `--wait` 需要在 5000 ms 内观测到 working/blocked 状态，否则报 `agent_prompt_stalled`。 | `herdr agent prompt --help` 文本与实时驱动 |
| 0.9.0 | 新启动的代理可以把提示词放入其输入框而不提交（Muse 1.0.3、Claude Code）：辅助工具会核验并按一次回车（`needed_enter: true`）。 | 实测（#31985） |
| 0.9.0 | 恢复的窗格不是先前的进程，且窗格 id 在服务器重启后可能被重新签发；id 并不可靠地从 `w1` 重新开始——在 `herdr session stop <name>` 并通过 master 重启后，一个具名会话的编号继续递增（`w5` → `w6`）。身份元组（`server`、`ref`、`cwd`、`engine`）的存在是因为复用，而不是因为编号。 | herdr-projects 阶段 2/3 的发现（从 `w1` 重启）；#38715 第 8 轮 FM2 泳道对具名会话的驱动（编号延续） |
| 0.9.0 | ssh master 已建立的已保存机器可能拒绝第二个 ssh 会话（`Session open refused by peer`）：Herdr 桥占用了唯一的槽位。应通过 master 访问（`ssh -O forward`），绝不要用全新的 `ssh <host>`。 | 在机群上实测（#31985，SLOT_FACT 文本） |
| 0.9.0 | `pane wait-output <pane> --regex <re> --timeout <ms>` 先立即搜索最近的快照，然后轮询；`--source recent-unwrapped` 可用。 | `herdr pane wait-output --help` |
| 0.9.0 | `notification show <TITLE> --body <TEXT>` 显示服务器级通知（没有按窗格的目标）。 | `herdr notification show --help` |
| 0.9.0 | `worktree open --path <PATH> --label <TEXT> --no-focus` 把既有的 git worktree 作为工作区打开。 | `herdr worktree open --help` |
| 0.9.0 | `workspace create --label <TEXT> --cwd <PATH> --no-focus` 返回根窗格；`--env KEY=VALUE` 存在，但辅助工具从不传递环境变量（启动环境继承仍被搁置）。 | `herdr workspace create --help`；`test_fleet.py` 固定了不使用 `--env` |
| 0.9.1 | 在文本输入到一半时到达的提示词会与其合并并提交。辅助工具会先读取输入框，若其中已有文本则拒绝（退出码 3）。 | 由 herdr-projects 笔记报告并被身份与安全规则采纳；此处未重新核验（本主机运行 0.9.0）——在 0.9.1 驱动确认之前视为假设 |
| 0.9.0 | 停留在工作区信任对话框上的全新 Muse 会话，在一次驱动中为 `blocked`，在随后两次中为 `idle`（`agent wait --until blocked` 返回 idle）；无论哪种情况 `dialog` 都会显示该提示。当 Herdr 未标记它时，`approve --force` 依 `dialog` 所示内容作答。 | 本主机上的回环黑盒测试，三次运行 |
| 0.9.0 | `machine add <SSH_TARGET> --label <LABEL> [--remote-session <NAME>]` 准备远程服务器并保存配置；目标必须放在最前——选项在前的 argv 会得到 `usage: herdr machine add <ssh-target> --label <label> [--remote-session <name>]` 与退出码 2，尽管 `--help` 打印的是 clap 风格的 `[OPTIONS] --label <LABEL> <SSH_TARGET>`。其 ssh 通过私有控制套接字运行（`-F /tmp/herdr-ssh-<pid>-0/config -S /tmp/herdr-ssh-<pid>-0/ctl`），返回前会执行 `-O exit`，因此其登录不会为 `connect` 留下任何可复用的通道。它的提示与输出直接进入终端，因此 `connect` 把它们路由到 stderr，并保持 stdout 为单一对象。 | 两种 argv 顺序都在隔离 HOME 中对真实二进制运行（`test_fleet.py` 中的假实现以同样方式拒绝选项在前的顺序）；ssh argv 用 PATH 上带日志功能的 `ssh` 捕获 |
| any | 0.8.x 客户端没有 `machine` 动词（`unknown command`）：机群仅支持 `local`。 | `list_machines` 处理了该情况；自 0.9.0 起未再核验 |

## tmux / tmux

| version | behaviour | verified how |
| --- | --- | --- |
| 3.x | `list-panes -a -F …` with `#{window_activity}` (epoch seconds) is the liveness signal: activity within 60 s counts as `working`, else `idle`. No agent state exists. | tmux format docs; the tmux suite |
| 3.x | `new-session -d -s <name>` refuses an existing name (`duplicate session`); the helper refuses too and names `status` as the next command. `open` sets `remain-on-exit on` in the same call, so an engine that exits at once leaves a dead pane whose last line is the reason: the helper kills it and answers `failed` (exit 6), never `opened`. A session whose panes all exited later shows `status: exited`, `liveness: dead`. | tmux behaviour; the tmux suite |
| 3.x | The identity engine is `#{pane_start_command}` (first word, after the `env -u …` wrapper `open` adds), else `#{default-shell}`: stable for the pane's life. `#{pane_current_command}` is reported as `program` and decides liveness (`close` refuses while any pane of the session runs a non-shell program). `list-panes` uses a fresh random field separator per read, so no path or title can shift fields. | the tmux suite |
| 3.x | `open` starts the engine under `env -u HERDR_ENV -u HERDR_PANE_ID -u HERDR_SOCKET_PATH -u TMUX …`, so a session is nobody's pane; a Muse engine gets `--workspace <cwd>`, plus `--yolo` only with `open --unattended` (default: the engine's own permission prompts; owner ruling 2026-09-19). `--worktree` and `--label` are `unsupported_by_provider` on tmux. | the tmux suite |
| 3.x | `capture-pane -p -J -S -<n>` is the scrollback read. The composer check for guarded input reads the line under the cursor, for every engine alike: empty or a bare prompt (ends in a prompt character) is free, and so is a prompt glyph followed by exactly one of Muse's idle tips, whole on one row or soft-wrapped over the rows below it (the dim text the TUI draws in an EMPTY composer after a turn settles; the list is the TUI's `prompt_hint.rs` table, shared with host-manager); any other text is half-typed input and nothing is typed over it; a check that cannot run counts as busy. `send-keys -l -- <text>` and `display-message … -- <text>` so text that starts with a dash is text. | the tmux suite |
| 3.7b | A pane-level command (`send-keys`, `capture-pane`, `display-message`) with `-t =<session>` alone does not resolve ("can't find pane"); `-t =<session>:` (exact session, its current window) does. `kill-session -t =<session>` resolves as a session target. | Probed on this host's tmux 3.7b while building the suite |
| 3.x | `display-message -t =<session>: -d 0 <text>` is the notification form of `send`. | tmux behaviour |
| 3.x | `send-keys -l <text>` then `send-keys Enter` types a guarded line; `send-keys C-c` is `stop`; `kill-session -t =<session>` is `close` (refused while a non-shell program runs, unless confirmed). | the tmux suite |
| 3.x | `-L <socket>` selects a private server: tests and black-box runs never touch the user's own sessions (`FLEET_MANAGER_TMUX_SOCKET`). | the tmux suite |

| 版本 | 行为 | 核验方式 |
| --- | --- | --- |
| 3.x | `list-panes -a -F …` 配合 `#{window_activity}`（epoch 秒）是活性信号：60 秒内的活动计为 `working`，否则为 `idle`。不存在代理状态。 | tmux format 文档；tmux 测试套件 |
| 3.x | `new-session -d -s <name>` 会拒绝已存在的名称（`duplicate session`）；辅助工具同样拒绝并提示下一步用 `status`。`open` 在同一次调用中设置 `remain-on-exit on`，因此立即退出的引擎会留下一个死窗格，其最后一行就是原因：辅助工具会杀掉它并回答 `failed`（退出码 6），绝不会回答 `opened`。窗格随后全部退出的会话显示 `status: exited`、`liveness: dead`。 | tmux 行为；tmux 测试套件 |
| 3.x | 身份引擎取 `#{pane_start_command}`（第一个词，取自 `open` 添加的 `env -u …` 包装之后），否则取 `#{default-shell}`：在窗格生命周期内保持稳定。`#{pane_current_command}` 作为 `program` 上报并决定活性（当会话中任一窗格运行非 shell 程序时，`close` 会拒绝）。`list-panes` 每次读取都使用新的随机字段分隔符，因此任何路径或标题都无法移动字段。 | tmux 测试套件 |
| 3.x | `open` 在 `env -u HERDR_ENV -u HERDR_PANE_ID -u HERDR_SOCKET_PATH -u TMUX …` 下启动引擎，因此会话不属于任何人的窗格；Muse 引擎会获得 `--workspace <cwd>`，且只有 `open --unattended` 才附加 `--yolo`（默认：使用引擎自己的权限提示；所有者裁定 2026-09-19）。在 tmux 上 `--worktree` 与 `--label` 为 `unsupported_by_provider`。 | tmux 测试套件 |
| 3.x | `capture-pane -p -J -S -<n>` 用于读取回滚缓冲。对受保护输入的输入框检查读取光标所在行，对所有引擎一视同仁：空行或纯提示符（以提示符字符结尾）即为空闲；提示符符号后恰好跟着一条 Muse 空闲提示的情况同样算空闲（整条在一行，或软换行到下方几行；这是 TUI 在一轮结束后于空输入框中绘制的暗色文字；该清单即 TUI 的 `prompt_hint.rs` 表，与 host-manager 共享）；任何其他文字都是输入到一半的内容，不会在其上键入任何东西；无法运行的检查按忙对待。`send-keys -l -- <text>` 与 `display-message … -- <text>` 保证以破折号开头的文字仍被当作文字。 | tmux 测试套件 |
| 3.7b | 窗格级命令（`send-keys`、`capture-pane`、`display-message`）只带 `-t =<session>` 时无法解析（"can't find pane"）；`-t =<session>:`（确切会话，其当前窗口）可以。`kill-session -t =<session>` 作为会话目标可以解析。 | 构建测试套件时在本主机的 tmux 3.7b 上探测 |
| 3.x | `display-message -t =<session>: -d 0 <text>` 是 `send` 的通知形式。 | tmux 行为 |
| 3.x | `send-keys -l <text>` 然后 `send-keys Enter` 键入受保护的一行；`send-keys C-c` 是 `stop`；`kill-session -t =<session>` 是 `close`（当非 shell 程序在运行时会被拒绝，除非已确认）。 | tmux 测试套件 |
| 3.x | `-L <socket>` 选择私有服务器：测试与黑盒运行绝不接触用户自己的会话（`FLEET_MANAGER_TMUX_SOCKET`）。 | tmux 测试套件 |

## MSP (host-manager's mode C) / MSP（host-manager 的模式 C）

| version | behaviour | verified how |
| --- | --- | --- |
| transport CLI 2026-09 | `muse hosts --json` rows carry the host id, `msp_ready`, `availability` (online\|offline) and no `authorized` key (absent is authorized); a row without `msp_ready` is not a machine. The operator's directory can be wide: 20 advertising hosts on one devserver, 65 for the verifier. | the operator rows of the 2026-09-27 MSP acceptance and this PR's live rows |
| transport CLI 2026-09 | `muse sessions --host <host> --json` answers `result.sessions[]` of `{target: "<host>/<session id>", session: {sessionId, title, workspaceRoot, status: idle\|running\|notLoaded, activeTurnId}}`; `muse show <target>` adds `pendingRequests[]`. One `sessions --host` per online host per digest (in parallel), one `show` per running session. `--all-hosts` is a catalog scan with a 64-host budget: past it the answer is exit 4, `status: partial`, no rows and one `errors[]` entry per host — never the board's source. | live 2026-09-27 (row 24; verifier V-43535 raw envelope); the host-manager fake mirrors both shapes |
| host-manager | `open --host H --cwd P --name N [--prompt-file]` is `muse start` then the brief as `muse send --busy queue`; `send --steer` is `muse steer --turn <running>`; `close --confirm` is `muse interrupt --turn` then `muse task stop-all`, and the host keeps the session listed idle (no per-session end verb); `stop` on a session host-manager did not record is `no_such_session`, so this skill's `stop` names `close` instead. | host-manager's `test_msp_provider.py`; this skill's `test_msp.py` |

| 版本 | 行为 | 核验方式 |
| --- | --- | --- |
| 传输 CLI 2026-09 | `muse hosts --json` 的行带有主机 id、`msp_ready`、`availability`（online\|offline），并且没有 `authorized` 键（缺省即已授权）；没有 `msp_ready` 的行不是机器。操作员的目录可能很大：一台 devserver 上有 20 个通告主机，验证者那里有 65 个。 | 2026-09-27 MSP 验收的操作员行与本 PR 的实时行 |
| 传输 CLI 2026-09 | `muse sessions --host <host> --json` 以 `result.sessions[]` 应答，元素为 `{target: "<host>/<session id>", session: {sessionId, title, workspaceRoot, status: idle\|running\|notLoaded, activeTurnId}}`；`muse show <target>` 额外给出 `pendingRequests[]`。每份摘要对每台在线主机一次 `sessions --host`（并行执行），对每个运行中的会话一次 `show`。`--all-hosts` 是一次带 64 台主机预算的目录扫描：超出后应答为退出码 4、`status: partial`、无行，且每台主机一条 `errors[]` 条目——它绝不能作为看板的数据来源。 | 2026-09-27 实时（第 24 行；验证者 V-43535 原始信封）；host-manager 的假实现复刻了两种形态 |
| host-manager | `open --host H --cwd P --name N [--prompt-file]` 是先 `muse start` 再以 `muse send --busy queue` 发送任务简报；`send --steer` 是 `muse steer --turn <running>`；`close --confirm` 是先 `muse interrupt --turn` 再 `muse task stop-all`，且主机让该会话保持为 idle 列示（没有按会话的结束动词）；对 host-manager 未记录的会话执行 `stop` 会得到 `no_such_session`，因此本技能的 `stop` 改用 `close`。 | host-manager 的 `test_msp_provider.py`；本技能的 `test_msp.py` |
