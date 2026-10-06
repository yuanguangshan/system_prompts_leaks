---
name: doctor
description: Diagnose Muse Code product/runtime issues from installed binary evidence. Use ONLY when the user explicitly invokes the doctor skill, asks to debug/troubleshoot Muse Code itself, asks what happened earlier in the current Muse Code session, or explicitly selects an earlier Muse Code session. Do NOT use for ordinary repository code failures or history, benchmark tasks, implementation debugging, build/test hangs, or third-party project issues.
---
<!-- BILINGUAL-EN-ZH -->

# Diagnose / 诊断

Diagnose Muse Code as an installed product. Use this skill only when the user
explicitly invokes `doctor` or clearly asks to debug/troubleshoot Muse Code
itself from binary/runtime evidence. Assume the user has the binary, not the
source tree. Help them understand how the app works, collect the smallest safe
evidence set, identify the likely failing layer, and give the next safe action.

把 Muse Code 作为已安装产品来诊断。仅当用户显式调用 `doctor`、或明确要求基于二进制/运行时证据调试 Muse Code 本身时，才使用本技能。假定用户手里只有二进制，而非源码树。帮助用户理解应用如何工作，收集最小且安全的证据集，定位可能的失效层，并给出下一个安全的动作。

## Scope / 适用范围

- Use this skill ONLY when the user explicitly invokes `doctor`, asks to use
  the doctor skill, or clearly asks to debug/troubleshoot broken Muse Code
  product behavior: app, CLI, TUI, desktop, crash, provider/model, settings,
  auth, trust, skills, plugins, MCP, session, resume, export, trace, approvals,
  sandbox, update, or unexpected output.
  仅当用户显式调用 `doctor`、要求使用 doctor 技能、或明确要求调试 Muse Code 损坏的产品行为时，才使用本技能：应用、CLI、TUI、桌面端、崩溃、服务商/模型、设置、鉴权、信任、技能、插件、MCP、会话、resume、导出、trace、审批、沙箱、更新或意外输出。
- Use this skill when the user asks how Muse Code itself works, where Muse Code
  stores state, what a Muse Code log/session/trace means, or how to collect a
  Muse Code support bundle.
  当用户询问 Muse Code 本身如何工作、Muse Code 在哪里存储状态、Muse Code 的日志/会话/trace 是什么含义、或如何收集 Muse Code 支持包时，使用本技能。
- Use this skill when the user asks what happened earlier in the current
  Muse Code session or explicitly selects an earlier Muse Code session for
  evidence. This does not include ordinary repository history or an unspecified
  third-party agent session.
  当用户询问当前 Muse Code 会话早前发生了什么、或显式选择一个更早的 Muse Code 会话作为证据时，使用本技能。这不包括普通的仓库历史，也不包括未指明的第三方代理会话。
- Do NOT use this skill for ordinary repository engineering: code
  implementation, third-party project bugs, benchmark/eval tasks, build or test
  failures, command hangs, toolchain issues, CI failures, or local debugging
  inside a non-Muse Code codebase. Handle those with the normal engineering
  workflow unless the evidence points to Muse Code itself.
  不要把本技能用于普通的仓库工程：代码实现、第三方项目 bug、基准/评测任务、构建或测试失败、命令挂起、工具链问题、CI 失败、或在非 Muse Code 代码库内的本地调试。除非证据指向 Muse Code 本身，否则用正常工程工作流处理这些情况。
- Treat the user as a product user first, not as a repository engineer.
  首先把用户当作产品用户，而不是仓库工程师。
- For pure settings questions or explicitly requested settings edits with no
  product failure to investigate, use the `manage-settings` skill instead;
  Diagnose reads settings only as evidence for a failure it is investigating.
  对于纯设置问题、或与产品故障排查无关且被显式要求的设置修改，改用 `manage-settings` 技能；Diagnose 读取设置仅作为其正在排查的故障的证据。
- Do not create issues, branches, commits, PRs, install/enable/disable skills or
  plugins, change settings/auth/trust, upload logs, run live-provider/network
  checks, or edit code unless the user explicitly asks.
  除非用户明确要求，否则不要创建 issue、分支、提交、PR，不要安装/启用/禁用技能或插件，不要修改设置/鉴权/信任，不要上传日志，不要运行真实服务商/网络检查，也不要编辑代码。
- Do not print secrets, raw prompts, raw model payloads, auth tokens, API keys,
  cookies, bearer headers, or full session logs. Prefer redacted exports,
  key/value presence checks, and concise summaries.
  不要输出机密、原始提示词、原始模型载荷、鉴权令牌、API 密钥、cookie、bearer 请求头或完整会话日志。优先使用脱敏导出、键/值存在性检查和简洁摘要。
- If a surface has no standalone app log, say so and use session logs, crash
  reports, trace inspection, or export evidence instead of inventing a path.
  如果某个界面没有独立的应用日志，就如实说明，改用会话日志、崩溃报告、trace 检查或导出证据，不要编造路径。

## Mental Model / 心智模型

Explain the relevant product path before asking for logs:

在索要日志之前，先解释相关的产品路径：

- The binary reads settings/auth/trust from the user's config directory and
  writes sessions, crashes, model catalog cache, and memory under the data
  directory.
  二进制从用户的配置目录读取设置/鉴权/信任，并在数据目录下写入会话、崩溃、模型目录缓存和记忆。
- A Muse Code session is the main handle for resume, trace inspection, export,
  and support. Prefer a session id or session log path over screenshots of
  terminal output.
  Muse Code 会话是 resume、trace 检查、导出和支持的主要句柄。相比终端输出的截图，优先使用会话 id 或会话日志路径。
- Provider/auth failures are often config, environment, model catalog, network,
  or credential problems. Separate those before blaming the model.
  服务商/鉴权失败往往是配置、环境、模型目录、网络或凭据问题。在归咎于模型之前，先把这些问题分开排查。
- Skills/plugins/MCP are loaded product capabilities. Diagnose discovery,
  activation, trust, validation, and runtime errors separately.
  技能/插件/MCP 是已加载的产品能力。分别诊断发现、激活、信任、校验和运行时错误。
- A trace/export explains what the binary saw and did. It is evidence, not a
  transcript to paste raw.
  trace/导出说明二进制看到了什么、做了什么。它是证据，不是可以原样粘贴的会话转写。

## Triage Questions / 分诊问题

1. Name the failing surface and exact symptom.
   说出失效的界面和确切症状。
2. Record the command, cwd, session id/path if provided, whether the user wants
   to inspect or continue that session, approximate time, provider/model if
   relevant, and whether the issue reproduces.
   记录命令、cwd、（如提供的）会话 id/路径、用户是想检查还是继续该会话、大致时间、相关的服务商/模型，以及问题是否可复现。
3. Ask for one missing handle only when it blocks a safe local check. Prefer:
   exact command, session id/path, time window, and whether they can reproduce.
   只有当缺失的句柄阻碍某项安全的本地检查时才索要。优先索取：确切命令、会话 id/路径、时间窗口，以及用户能否复现。
4. If the user only wants an explanation, explain first and avoid running checks.
   如果用户只想要解释，先做解释，避免运行检查。

## Product Evidence Map / 产品证据地图

Collect the smallest read-only set that explains the issue. Adapt the map to the
symptom; do not run every row by default.

收集能解释问题的最小只读集合。按症状调整该地图；默认不要逐行全跑。

The config/data roots below are Muse Code's entire local state surface. A
config, log, or session file outside them is not Muse Code state —
never present one as the product's active configuration, logs, or session
evidence.
Variables such as `CODEX_HOME` matter only for explicitly requested
import/compat evidence and stay attributed to the product that owns them.

下述配置/数据根目录就是 Muse Code 全部的本地状态表面。位于它们之外的配置、日志或会话文件不属于 Muse Code 状态——绝不要把这样的文件当作产品的现行配置、日志或会话证据来呈现。
诸如 `CODEX_HOME` 之类的变量仅在显式要求的导入/兼容证据时才相关，并且始终归属于拥有它们的产品。

1. Build/provenance: `muse --version`; also note `command -v muse` when
   multiple copies may exist.
   构建/来源：`muse --version`；当可能存在多份副本时，也记录 `command -v muse` 的结果。
2. Config/data roots: `$XDG_CONFIG_HOME/muse` or `$HOME/.config/muse`;
   `$XDG_DATA_HOME/muse` or `$HOME/.local/share/muse`.
   配置/数据根目录：`$XDG_CONFIG_HOME/muse` 或 `$HOME/.config/muse`；`$XDG_DATA_HOME/muse` 或 `$HOME/.local/share/muse`。
3. Local files: `settings.json`, `auth.json`, and `trust.json`; read settings
   when relevant, but report auth/trust presence and provider names only.
   本地文件：`settings.json`、`auth.json` 和 `trust.json`；相关时读取设置，但只报告 auth/trust 的存在性和服务商名称。
4. Data dirs: `sessions`, `memory`, `model-catalog`, and `crashes` under the
   data dir. For crashes, summarize report metadata and file path, not session
   content.
   数据目录：数据目录下的 `sessions`、`memory`、`model-catalog` 和 `crashes`。对崩溃，只概述报告元数据和文件路径，不要概述会话内容。
5. Session evidence: prefer `muse export --redacted --out <file>` for the
   latest workspace session, or `muse export --session <id-or-session.jsonl>
   --redacted --out <file>` when the user provides a handle.
   会话证据：对最近的工作区会话优先使用 `muse export --redacted --out <file>`；当用户提供句柄时使用 `muse export --session <id-or-session.jsonl> --redacted --out <file>`。
6. Continuation handles: if the user provides a Muse Code session id and wants
   to continue that session, point them to `muse resume <session-id>` or
   `muse resume --last` for interactive continuation. Use
   `muse exec --session-id <session-id> "<follow-up>"` only on explicit
   request for headless continuation. Add `--allow-workspace-switch` to the
   `muse exec --session-id` command only after confirming the session
   belongs to another workspace; interactive `muse resume` does not take
   this flag. If an exit, fork, or handoff message printed
   `muse resume <session-id>`, treat that command as the canonical handle.
   续接句柄：如果用户提供了 Muse Code 会话 id 并想继续该会话，交互式续接指向 `muse resume <session-id>` 或 `muse resume --last`。仅当用户明确要求无头（headless）续接时，才使用 `muse exec --session-id <session-id> "<follow-up>"`。只有在确认该会话属于另一个工作区之后，才给 `muse exec --session-id` 命令加 `--allow-workspace-switch`；交互式 `muse resume` 不接受该标志。如果退出、分叉或交接消息打印了 `muse resume <session-id>`，就把该命令视为规范句柄。
7. Trace evidence: use `muse trace inspect --session-log <session.jsonl>
   --render-mode compact`; add `--run-id <uuid>` or `--all-runs` for multi-run
   logs; use `--format json` only when structured analysis is needed.
   trace 证据：使用 `muse trace inspect --session-log <session.jsonl> --render-mode compact`；对多 run 日志加 `--run-id <uuid>` 或 `--all-runs`；仅在需要结构化分析时使用 `--format json`。
8. User support bundle: use `/feedback` when available; otherwise prefer a
   redacted export, trace inspection, crash metadata, and concise reproduction
   steps.
   用户支持包：可用时使用 `/feedback`；否则优先提供脱敏导出、trace 检查、崩溃元数据和简洁的复现步骤。
9. Skills/plugins/MCP: use `muse skills list --enabled-only --json` and safe
   `muse plugins ... --help` or validation commands when the symptom points
   there.
   技能/插件/MCP：当症状指向那里时，使用 `muse skills list --enabled-only --json` 以及安全的 `muse plugins ... --help` 或校验命令。
10. Environment: check relevant non-secret variables by presence/value only, such
    as `MUSE_MODEL`, base-url variables with credentials redacted, XDG dirs,
    `CODEX_HOME` for import/compat issues, and telemetry variables by presence
    only.
    环境：对相关非机密变量只检查存在性/取值，例如 `MUSE_MODEL`、凭据已脱敏的 base-url 变量、XDG 目录、用于导入/兼容问题的 `CODEX_HOME`，以及只查存在性的遥测变量。

## Use Current Session Evidence First / 优先使用当前会话证据

For a question about what happened earlier in the current Muse Code session,
use the sibling `scripts/session-evidence.py` helper before export or full trace
inspection. After `read_skill` gives the physical Diagnose package path, run:

对于关于当前 Muse Code 会话早前发生了什么的问题，在导出或完整 trace 检查之前，先使用同目录的 `scripts/session-evidence.py` 辅助脚本。当 `read_skill` 给出 Diagnose 包的物理路径后，运行：

```bash
python3 <doctor-skill-dir>/scripts/session-evidence.py --session-log <current-session.jsonl> --workspace "$PWD"
```

The runtime session-identity context already contains the exact current log
path. Do not ask the user for a path already present there, do not guess a
latest session, and do not search the session store first. A host may instead
provide `MUSE_CURRENT_SESSION_LOG`, in which case the helper can run without a
selector.

运行时的会话身份上下文已包含确切的当前日志路径。不要向用户索要其中已有的路径，不要猜测最近的会话，也不要先搜索会话存储。宿主也可能改为提供 `MUSE_CURRENT_SESSION_LOG`，此时辅助脚本无需选择器即可运行。

For an explicitly selected earlier Muse Code session, use the exact path or id:

对于显式选择的更早 Muse Code 会话，使用确切的路径或 id：

```bash
python3 <doctor-skill-dir>/scripts/session-evidence.py --session-log <explicit-session.jsonl> --workspace "$PWD"
python3 <doctor-skill-dir>/scripts/session-evidence.py --session-id <explicit-session-id> --workspace "$PWD"
```

Both explicit earlier-session forms are current-workspace scoped and fail
closed on unknown workspace metadata, a mismatch, or ambiguity. Use `--kind`,
`--path`, `--tool`, `--run-id`, or sequence bounds to narrow follow-up evidence.
Projected events retain their source stream, and the default bound reserves
evidence for both the main session and child sessions so a busy child cannot
erase the parent timeline. Always compare durable actions with assistant claims,
especially across compaction and child activity. Use a
redacted export or compact trace only when this bounded projection is
insufficient. Never paste the raw session log into model context.

两种显式早期会话形式都以当前工作区为范围，并在工作区元数据未知、不匹配或含糊时失败即停（fail closed）。使用 `--kind`、`--path`、`--tool`、`--run-id` 或序列边界来收窄后续证据。投影出的事件保留其来源流，且默认边界为主会话和子会话都保留证据，避免繁忙的子会话抹掉父会话时间线。始终把持久化动作与助手的声称做对比，尤其是在压缩（compaction）和子活动之后。只有当这一有界投影不够用时，才使用脱敏导出或紧凑 trace。绝不要把原始会话日志粘贴进模型上下文。

【评论】"绝不要把原始会话日志粘贴进模型上下文"同时服务于隐私（日志可能含凭据与私密内容）和上下文窗口预算两方面；有界投影是折中后的取证方式。

## Diagnose Live Session Ownership Safely / 安全诊断活动会话的属主

Use this path when resume says a session is already open or the original
terminal no longer accepts input:

当 resume 提示会话已打开、或原始终端不再接受输入时，走这条路径：

1. Select the exact session first and run `scripts/session-evidence.py` as
   above. Bound the output and compare its latest durable activity timestamps;
   do not start with a store-wide process or file search.
   先选定确切的会话，并如上文运行 `scripts/session-evidence.py`。对输出设置边界，并比较其最新的持久化活动时间戳；不要从全库范围的进程或文件搜索开始。
2. Treat `.session.lock` as an inode-backed kernel lease, not a marker file:
   file existence is not lock ownership, and `flock` protects an open inode.
   Unlinking a contended pathname can let another process create and lock a new
   inode while the original writer still owns the old inode.
   把 `.session.lock` 当作基于 inode 的内核租约，而不是标记文件：文件存在不等于持有锁，`flock` 保护的是打开的 inode。对一个被争用的路径名做 unlink（解除链接），可能让另一个进程创建并锁住新 inode，而原写者仍持有旧 inode。
3. Probe the exact lock read-only with a non-blocking exclusive `flock`. Open it
   without truncation, report only `acquirable`, `contended`, `missing`, or the
   read error, then close it immediately. Never resume the session as a probe.
   用非阻塞的独占 `flock` 以只读方式探测确切的锁。打开时不得截断，只报告 `acquirable`、`contended`、`missing` 或读取错误，然后立即关闭。绝不要把执行 resume 当作探测手段。
4. Read the lock body's PID only as a hint. A tool sandbox or PID namespace may
   not see the host process; `ps`/`kill -0` absence inside it cannot prove that
   the host owner died. Request a host-shell check when that distinction matters.
   锁体内记录的 PID 只作提示。工具沙箱或 PID 命名空间可能看不到宿主进程；在其中 `ps`/`kill -0` 查不到该进程，不能证明宿主属主已死亡。当这一区别很关键时，应请求宿主 shell 检查。
5. Check the bounded session evidence for prior file mutation involving
   `.session.lock`, especially `rm`, unlink, replacement, truncation, or
   recreation. If the path is missing while old activity advances, stop resume
   attempts and preserve evidence.
   在有界会话证据中检查此前是否发生过涉及 `.session.lock` 的文件变动，尤其是 `rm`、unlink、替换、截断或重建。如果该路径缺失而旧活动仍在推进，停止 resume 尝试并保全证据。

Never remove, replace, truncate, or recreate `.session.lock` as diagnosis or
repair. Never signal the owner from Diagnose. Do not request takeover or use an
owner-control endpoint. Ask the user to exit the owning process normally, then
explicitly retry ordinary resume.

绝不要把删除、替换、截断或重建 `.session.lock` 作为诊断或修复手段。绝不要从 Diagnose 向属主发送信号。不要请求接管，也不要使用属主控制端点。请用户正常退出属主进程，然后显式重试普通 resume。

Classify the result before recommending an action:

在建议动作之前，先对结果分类：

| Evidence | Classification | Safe next action |
| --- | --- | --- |
| Lease is acquirable | No current kernel owner; a leftover pathname is harmless | Retry normal resume; do not clean the file for tidiness |
| contended lease with advancing session activity | Live owner and active runtime; terminal attachment may be the failed layer | Preserve the owner; inspect terminal/PTY evidence or exit it normally |
| Contended lease with bounded activity idle | Live kernel owner, runtime/terminal health unknown | Ask the user to exit the owning process normally, then explicitly retry ordinary resume |
| Legacy owner-control artifacts are present | Diagnostic leftovers from an old binary; the kernel lease remains authoritative | Treat them as diagnostic only; exit the owning process normally before ordinary resume retry |
| Lock path was unlinked or replaced while old activity continues | Unsafe prior mutation with possible dual writers | Stop further resume attempts, preserve both inode/session timelines, and escalate; do not recreate the lock |

| 证据 | 分类 | 安全的下一步动作 |
| --- | --- | --- |
| 租约可获取 | 当前没有内核属主；遗留的路径名无害 | 重试正常的 resume；不要为了整洁去清理该文件 |
| 争用租约且会话活动持续推进 | 属主存活且运行时活跃；终端接入可能是失效层 | 保全属主；检查终端/PTY 证据或让其正常退出 |
| 争用租约且有界活动空闲 | 内核属主存活，运行时/终端健康状态未知 | 请用户正常退出属主进程，然后显式重试普通 resume |
| 存在旧式属主控制产物 | 旧二进制的诊断遗留物；内核租约仍是权威 | 仅将其视为诊断信息；在普通 resume 重试之前先让属主进程正常退出 |
| 锁路径被解除链接或替换而旧活动仍在继续 | 不安全的先前变动，可能存在双写者 | 停止进一步的 resume 尝试，保全两条 inode/会话时间线并上报；不要重建锁 |

【评论】这一节的核心是把"锁"从文件系统语义纠正为内核语义：`flock` 绑定的是 inode 而非路径名，因此"文件不见了"和"锁已释放"是两回事，误删锁文件反而可能制造双写者。

A contended lease proves a live kernel owner, not that its TUI, terminal,
provider, or runtime is healthy. Pin the failing layer from activity plus
host-visible evidence before proposing a product fix.

争用中的租约只能证明内核属主存活，不能证明其 TUI、终端、服务商或运行时是健康的。在提出产品修复之前，先用活动加上宿主可见的证据钉死失效层。

## Narrowing Loop / 收窄循环

Work from evidence:

从证据出发工作：

1. State the most likely layer in one or two concrete sentences.
   用一两句具体的话陈述最可能的失效层。
2. Say why the next check will confirm or reject that hypothesis, then run that
   one safe local check.
   说明下一项检查为何能证实或否定该假设，然后只运行那一项安全的本地检查。
3. Update the hypothesis from the result and continue only while the next check
   is still relevant and safe.
   根据结果更新假设，只在下一项检查仍然相关且安全时继续。
4. If a live provider, live network, destructive command, or upload is required,
   state why and get explicit user approval first.
   如果需要真实服务商调用、真实网络、破坏性命令或上传，先说明原因并取得用户的明确批准。
5. If customization or local state is suspected, compare against an isolated
   temporary XDG config/data profile only after explaining that it will not read
   or mutate the user's real settings.
   如果怀疑是定制或本地状态问题，只有在说明其不会读取或改动用户真实设置之后，才用一个隔离的临时 XDG 配置/数据档案做对比。

## Common Diagnosis Paths / 常见诊断路径

- Startup/auth: binary path -> version -> config load -> auth provider present
  -> model/catalog selection -> first network boundary.
  启动/鉴权：二进制路径 -> 版本 -> 配置加载 -> 鉴权服务商存在 -> 模型/目录选择 -> 第一处网络边界。
- Provider/model: selected provider/model -> base URL with credentials redacted
  -> auth presence -> model catalog cache -> trace request/error summary.
  服务商/模型：所选服务商/模型 -> 凭据已脱敏的 base URL -> 鉴权存在性 -> 模型目录缓存 -> trace 请求/错误摘要。
- Session/resume: session id/path -> workspace match -> session log exists ->
  resume/export/trace command -> whether the user wants inspection or
  continuation.
  会话/resume：会话 id/路径 -> 工作区匹配 -> 会话日志存在 -> resume/export/trace 命令 -> 用户想要检查还是续接。
- Skills/plugins/MCP: list/discovery -> activation/trust -> validation output ->
  runtime trace or startup diagnostic.
  技能/插件/MCP：列表/发现 -> 激活/信任 -> 校验输出 -> 运行时 trace 或启动诊断。
- Desktop/TUI: packaged app/binary version -> session id -> UI-visible symptom
  -> crash/report metadata -> trace/export evidence. Say when source-only tests
  cannot prove a packaged-app issue.
  桌面端/TUI：打包应用/二进制版本 -> 会话 id -> UI 可见症状 -> 崩溃/报告元数据 -> trace/导出证据。当仅靠源码测试无法证明打包应用的问题时，要如实说明。

## Fix Boundary / 修复边界

- Settings-only fix: propose the exact change and apply it only after explicit
  user request; preserve unknown settings fields and verify with a read-back or
  focused command.
  仅设置的修复：提出确切的修改，且仅在用户明确请求后才应用；保留未知的设置字段，并通过回读或聚焦的命令进行验证。
- Workspace code fix: switch to the RED-before-GREEN engineering loop only when
  the user explicitly asks to fix code in the current repo. Reproduce first,
  edit surgically, rerun the same check, and report incomplete if it still
  fails.
  工作区代码修复：仅当用户明确要求修复当前仓库的代码时，才切换到 RED-before-GREEN 工程循环。先复现，精准修改，重跑同一检查，若仍失败则报告为未完成。
- Product bug report: if the evidence points to Muse Code itself and no local fix
  is safe, give a concise support bundle with symptom, version, config/data
  paths, session/crash paths, redacted export/trace evidence, likely cause, and
  next action.
  产品 bug 报告：如果证据指向 Muse Code 本身、且没有安全的本地修复，给出一份简洁的支持包，包含症状、版本、配置/数据路径、会话/崩溃路径、脱敏的导出/trace 证据、可能原因和下一步动作。

## Completion Report / 完成报告

Include:

报告需包含：

- Symptom and affected surface.
  症状与受影响的界面。
- Product mental model relevant to this failure.
  与本次故障相关的产品心智模型。
- Evidence collected, paths inspected, commands and outcomes.
  收集的证据、检查过的路径、命令及其结果。
- Redactions applied.
  已应用的脱敏处理。
- Most likely cause and confidence.
  最可能的原因及置信度。
- Fix made, proposed, or not made.
  已做、已建议或未做的修复。
- Remaining uncertainty and the next safest check.
  剩余的不确定性与下一个最安全的检查。
