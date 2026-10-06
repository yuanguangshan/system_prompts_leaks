<!-- BILINGUAL-EN-ZH -->
# Getting started: nothing → first session in five steps / 快速上手：从零到首个会话的五步

`<fleet>` is `python3 <skill-dir>/scripts/fleet_manager.py`, with the
directory taken from the read that delivered this skill (`skill-dir` when
present, else the directory of the SKILL.md path it shows). Every command prints one
JSON object; `text` is the human-readable part, `next` is the one command to
run when something is missing, and `<fleet> <verb> --help` lists a verb's
flags.

`<fleet>` 即 `python3 <skill-dir>/scripts/fleet_manager.py`，目录取自交付本技能的那次读取（有 `skill-dir` 时用它，否则用其所示 SKILL.md 路径所在目录）。每个命令打印一个 JSON 对象；`text` 是人类可读部分，`next` 是缺少某物时应运行的那一条命令，`<fleet> <verb> --help` 列出某个动词的标志。

## 1–4. Doctor, open, context, steer / 1–4. 体检、打开、上下文、操控

```text
<fleet> doctor                                   # provider, machines, next command
<fleet> open                                     # muse, this repository root, an auto name, this host
<fleet> open --engine claude --cwd ~/repo --name reviewer --prompt-file brief.md
<fleet> context                                  # the whole picture; read `text`, act on `groups`
<fleet> read s1                                  # what it is doing or said
<fleet> dialog s1                                # what it is asking (Herdr)
<fleet> approve s1                               # answer a y/N or numbered dialog
<fleet> send s1 "heads up: I'll close this in 5 min"   # a notification, nothing typed
<fleet> send s1 "run the tests" --type --wait    # typed in as a prompt; refused over a non-empty composer
```

`doctor` answers `provider: herdr` (server answering, started if it was
down) or `provider: tmux` (liveness, scrollback, guarded input,
open/stop/close only); `needs_user_action` means the tmux install needs a
password: run `next`, rerun. `open` answers `opened` with `handle` (`s1`),
`addr`, `identity` and a `receipt`. `send --type` with `submitted: false`
means `read` before sending again.

`doctor` 会应答 `provider: herdr`（服务器有响应，若原本未启动则将其启动）或 `provider: tmux`（存活检测、回滚缓冲、受保护输入，仅支持 open/stop/close）；`needs_user_action` 表示 tmux 安装需要密码：运行 `next`，然后重跑。`open` 会以 `opened` 应答，附带 `handle`（`s1`）、`addr`、`identity` 和一个 `receipt`。`send --type` 若返回 `submitted: false`，意味着再次发送前先 `read`。

## 5. Add a machine / 5. 添加一台机器

```text
<fleet> connect me@buildbox --label buildbox
```

One command, one `progress` line per step; it stops at the first thing
wrong with `next` naming the fix. Without Herdr here it opens one ssh
master of its own (one login, at most one second-factor prompt), checks
what runs there, starts a stopped Herdr server and forwards its socket, or
records the machine as tmux-only. With Herdr here it adds the machine
through Herdr first (`herdr machine add`, Herdr's own login, closed before
it returns), then reuses an answering forward or ssh master, else opens one
ssh master of its own: a fresh host costs Herdr's login plus at most one of
this skill's; `connect buildbox` again usually costs none. Then
`open buildbox --engine claude --cwd ~/repo`, and `context` covers both
machines. A report the remote session wrote comes home with
`<fleet> fetch buildbox /home/me/repo/report.md`; unchanged content copies
nothing. A machine that advertises MSP (`machines` lists it with `modes`
`msp` when `TBH_AGENTS_SESSION_PROTOCOL` is on) needs no `connect` at all:
`<fleet> open <host> --cwd /home/me/repo --prompt-file brief.md` opens a
session there in one call; never `ssh` an MSP host id.

一条命令，每步一行 `progress` 输出；它在第一个出错之处停下，并用 `next` 指明修复方法。本机没有 Herdr 时，它会自行打开一个 ssh master（一次登录，至多一次二次验证提示），检查那边在运行什么，启动已停止的 Herdr 服务器并转发其套接字，或将该机器记录为仅 tmux。本机有 Herdr 时，它先通过 Herdr 添加机器（`herdr machine add`，使用 Herdr 自己的登录，返回前关闭），然后复用有响应的转发或 ssh master，否则自行打开一个 ssh master：一台全新主机的代价是 Herdr 登录加上至多一次本技能的登录；再次 `connect buildbox` 通常毫无代价。然后运行 `open buildbox --engine claude --cwd ~/repo`，`context` 即覆盖两台机器。远程会话写出的报告用 `<fleet> fetch buildbox /home/me/repo/report.md` 取回；内容未变时不复制任何东西。通告 MSP 的机器（当 `TBH_AGENTS_SESSION_PROTOCOL` 开启时，`machines` 会以 `modes` `msp` 列出它）完全不需要 `connect`：`<fleet> open <host> --cwd /home/me/repo --prompt-file brief.md` 一次调用即可在那里打开会话；绝不要对 MSP 主机 id 使用 `ssh`。

## Troubleshooting / 故障排查

| you see | it means | do |
| --- | --- | --- |
| `unsupported_by_provider` / `no_provider` (exit 4) | this provider cannot do that verb, or there is none | run the verb `next` names (tmux has no dialogs: `read`, then `send --type`) |
| `needs_user_action` (exit 5) | a step needs you: a password, a second factor, a confirmation | do the step in `next`, rerun |
| `provider_unreachable` (exit 6) with `connect …` | the ssh master is down or the Herdr socket does not answer | run the `connect` line in `next` with a terminal attached |
| `herdr_add_failed` (exit 6) | Herdr could not add the machine (its own output says why) | run the `herdr machine add …` line in `next`; a tmux-only box: `connect <target> --mode tmux` |
| `no_such_session` (exit 3) | the handle names a session that is gone | `list`; the session ended or was closed |
| `identity_mismatch` (exit 3) "not the session it was minted for" | the pane id or name now belongs to another process | `list`, use the new handle |
| `composer_not_empty` (exit 3) | someone is typing in that session | `read`, wait or finish the line, retry |
| `session_live` (exit 3) "is live; close refused" | the session still works | `stop` it first, or `close … --confirm "<the user's words>"` |
| a machine shows `stale` in `context` | it stopped answering, inside the skip window (`verbs.md`, Outage cadence) | nothing yet; the last group is kept; an `outage` item follows |
| `herdr machine list` fails | the Herdr client catalog is broken | `herdr machine list --json` by hand; `local/...` addresses keep working |

| 你看到 | 它意味着 | 应对 |
| --- | --- | --- |
| `unsupported_by_provider` / `no_provider`（退出码 4） | 该提供方无法执行该动词，或根本没有提供方 | 运行 `next` 所指名的动词（tmux 没有对话框：先 `read`，再 `send --type`） |
| `needs_user_action`（退出码 5） | 某一步需要你：密码、二次验证或确认 | 完成 `next` 中的步骤，然后重跑 |
| `provider_unreachable`（退出码 6）且带有 `connect …` | ssh master 已宕机或 Herdr 套接字无响应 | 在接好终端的情况下运行 `next` 中的 `connect` 行 |
| `herdr_add_failed`（退出码 6） | Herdr 未能添加该机器（其自身输出会说明原因） | 运行 `next` 中的 `herdr machine add …` 行；仅 tmux 的机器：`connect <target> --mode tmux` |
| `no_such_session`（退出码 3） | 该句柄所指的会话已不存在 | `list`；会话已结束或已被关闭 |
| `identity_mismatch`（退出码 3）"not the session it was minted for" | 该窗格 id 或名称现在属于另一个进程 | `list`，改用新的句柄 |
| `composer_not_empty`（退出码 3） | 有人在那个会话中正在输入 | `read`，等待或等那一行输入完成，然后重试 |
| `session_live`（退出码 3）"is live; close refused" | 会话仍在运行 | 先 `stop`，或 `close … --confirm "<the user's words>"` |
| 某台机器在 `context` 中显示 `stale` | 它停止应答，处于跳过窗口内（`verbs.md`，中断节奏） | 暂时无需处理；最后一组结果被保留；随后会出现一个 `outage` 条目 |
| `herdr machine list` 失败 | Herdr 客户端目录已损坏 | 手动运行 `herdr machine list --json`；`local/...` 地址继续可用 |

State lives in `~/.local/share/muse/fleet-manager/state.json` (handles,
outages, the last `context`, the last hash of every fetched file), fetched
files under `~/.local/share/muse/fleet-manager/home/<machine>/`, machines in
`~/.config/muse/machines.toml`, forwards and ssh masters under
`/tmp/fleet-manager-<uid>/`. Deleting the state file loses handles only; the
next `list` mints new ones.

状态保存在 `~/.local/share/muse/fleet-manager/state.json`（句柄、中断记录、最近一次 `context`、每个已取回文件的最近一次哈希），取回的文件位于 `~/.local/share/muse/fleet-manager/home/<machine>/` 下，机器记录在 `~/.config/muse/machines.toml`，转发与 ssh master 位于 `/tmp/fleet-manager-<uid>/` 下。删除状态文件只会丢失句柄；下一次 `list` 会铸造新的句柄。
