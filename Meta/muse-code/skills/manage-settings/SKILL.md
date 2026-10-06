---
name: manage-settings
description: Any explicit Muse Code setting question or change (model, reasoning effort, /settings) requires a silent read_skill call for bundled:manage-settings as FIRST ACTION—no assistant text or other tool first. Never use for repository, app, eval, or other non-Muse configuration.
---
<!-- BILINGUAL-EN-ZH -->

# Manage Settings / 管理设置

Explain saved settings and defaults from the workspace's configuration, keep
them distinct from live/effective state, and make small settings edits only when
the user explicitly asks for that edit.

解释来自工作区配置的已保存设置与默认值，将其与实时/生效状态明确区分，并且只在用户明确要求某项编辑时才进行小规模的设置修改。

## Scope / 适用范围

- Manage Settings is exclusively for Muse Code-owned settings.
  Manage Settings 仅用于 Muse Code 自有的设置。
- Never read, name, explain, or modify another agent's config, rules, skills,
  home directories, or compatibility environment.
  绝不读取、提及、解释或修改其他智能体的配置、规则、技能、主目录或兼容性环境。
- Use this skill for questions about product settings, config paths, reminder
  toggles, TUI settings, provider/model settings, and tool or policy settings.
  本技能用于回答关于产品设置、配置路径、提醒开关、TUI 设置、provider/模型设置以及工具或策略设置的问题。
- "Product settings" here means Muse Code itself. Repository settings,
  application settings screens, library configuration, and eval-task config
  work stay in the ordinary task workflow and MUST NOT load this skill.
  这里的"产品设置"指 Muse Code 本身。仓库设置、应用设置界面、库配置以及评测任务配置工作仍走普通任务工作流，绝不能加载本技能。
- Prefer the current workspace and current process environment over general
  defaults.
  相较于通用默认值，优先使用当前工作区和当前进程环境。
- Do not create issues, branches, commits, PRs, plugins, or skills.
  不要创建 issue、分支、提交、PR、插件或技能。
- Do not install, enable, disable, trust, activate, or run plugins or skills
  unless the user explicitly asks for that action.
  除非用户明确要求该操作，否则不要安装、启用、禁用、信任、激活或运行插件或技能。
- Do not print secrets. If a setting might contain a token, key, credential, or
  opaque auth value, describe whether it is present without revealing the value.
  不要打印机密。如果某个设置可能包含令牌、密钥、凭据或不透明的鉴权值，只描述其是否存在，绝不透露其值。

## Decide Before Any Tool Call / 任何工具调用之前先决策

- Manage Settings owns persistent saved settings, not current runtime state.
  Manage Settings 管辖的是持久化的已保存设置，而非当前运行时状态。
- An explicit request to change a setting is a saved-settings edit request; the
  user does not need to say "saved default" or name `settings.json`.
  更改设置的明确请求即是已保存设置的编辑请求；用户无需说"保存的默认值"或指名 `settings.json`。
- For questions about controls, supported tiers, or timing, answer from the
  stable contract below. Read config when the user asks for a saved value or a
  persistent setting change.
  关于控件、支持的档位或时机的问题，依据下文的稳定契约回答。当用户询问某个已保存的值或要求持久化修改设置时才读取配置。
- The first settings-file CONTENT access MUST be `read_file` on the resolved
  active path. When the absolute path is not already known, one read-only Bash
  call may resolve it from the environment, but that call may print only the
  resulting path and MUST NOT touch the filesystem. Use this shape without
  adding `ls`, `test`, `stat`, `cat`, or another command:
  `python3 -c 'import os; print(os.path.join(os.environ.get("XDG_CONFIG_HOME") or os.path.join(os.environ["HOME"], ".config"), "muse", "settings.json"))'`.
  YOLO mode alone never authorizes direct Bash file access.
  对设置文件内容的首次访问必须是对已解析的活动路径执行 `read_file`。当绝对路径尚不可知时，可以用一次只读 Bash 调用从环境中解析它，但该调用只能打印最终得到的路径，且绝不能触碰文件系统。使用如下形式，不得附加 `ls`、`test`、`stat`、`cat` 或其他命令：
  `python3 -c 'import os; print(os.path.join(os.environ.get("XDG_CONFIG_HOME") or os.path.join(os.environ["HOME"], ".config"), "muse", "settings.json"))'`。
  仅凭 YOLO 模式绝不能授权直接用 Bash 访问文件。
- Use `edit_file` or `write_file` for the smallest safe JSON change. If those
  tools reject the same app-private config path, only when the startup security
  context explicitly says YOLO mode is already active (approval bypassed,
  shell sandbox off, and workspace trusted), a structured shell JSON edit may
  be used after the file tools reject that exact path in this turn. Use a JSON
  parser plus atomic replace and reread; never use text substitution.
  使用 `edit_file` 或 `write_file` 进行最小幅度的安全 JSON 修改。如果这些工具拒绝了同一个应用私有配置路径，则只有当启动安全上下文明确说明 YOLO 模式已激活（审批被绕过、shell 沙箱关闭、工作区已受信任）时，才能在本轮文件工具拒绝该确切路径之后使用结构化的 shell JSON 编辑。应使用 JSON 解析器加原子替换并重新读取；绝不使用文本替换。
- A shell file-access fallback must operate on that one resolved path. It MUST NOT test a
  second root, branch on file existence, use `$HOME` after resolving an
  `XDG_CONFIG_HOME` path, list the directory, or `cat` the whole settings file.
  Emit only the requested safe key values and the minimal preservation evidence.
  shell 文件访问后备方案必须只作用于那一个已解析的路径。它绝不能探测第二个根目录、依据文件存在与否分支、在解析了 `XDG_CONFIG_HOME` 路径后再使用 `$HOME`、列出目录内容，或 `cat` 整个设置文件。只输出被请求的安全键值和最少量的保全证据。
- Never ask the user to change security mode just to edit settings.
  绝不为了编辑设置而要求用户更改安全模式。
- Never recommend disabling sandbox, approvals, or another security control to
  inspect or change settings.
  绝不建议为查看或更改设置而禁用沙箱、审批或其他安全控制。
- Renderer settings such as reasoning summaries and verbose output cannot be
  simulated in the assistant response. Explain the owning control instead of
  promising equivalent behavior.
  推理摘要和详细输出等渲染器设置无法在助手回复中模拟。应解释拥有该功能的控件，而不是承诺等效行为。

## Read Before Answering / 回答前先读取

First classify the request. Questions about product controls, supported tiers,
or timing use the stable contract below and do not require a settings-file read.
Read the file for a saved-value question or an explicit setting change. A
request to change a setting means a persistent file edit; it does not authorize
a claim about current runtime state.

先对请求分类。关于产品控件、支持档位或时机的问题使用下文的稳定契约回答，无需读取设置文件。涉及已保存值的询问或明确的设置更改则要读取文件。更改设置的请求意味着一次持久化文件编辑；它并不授权对当前运行时状态作出断言。

Before explaining or editing a saved value:

在解释或编辑某个已保存的值之前：

1. Identify the settings file. Prefer `$XDG_CONFIG_HOME/muse/settings.json`
   when `XDG_CONFIG_HOME` is set; otherwise use `$HOME/.config/muse/settings.json`.
   确定设置文件。当 `XDG_CONFIG_HOME` 已设置时优先使用 `$XDG_CONFIG_HOME/muse/settings.json`；否则使用 `$HOME/.config/muse/settings.json`。
2. Read that file when it exists. After the optional path-only resolver, the
   first content access MUST use `read_file`. If it rejects the app-private path
   and startup evidence already says YOLO mode is active, the structured shell
   JSON fallback above may read the same path.
   若文件存在则读取它。在可选的仅路径解析之后，首次内容访问必须使用 `read_file`。如果它拒绝了该应用私有路径、且启动证据已表明 YOLO 模式处于激活状态，则上文的结构化 shell JSON 后备方案可以读取同一路径。
3. A missing file is the normal first-write state. For a read-only question,
   say which path was checked, explain the relevant default, report that no file
   changed, and stop. For an explicit unambiguous change, create the smallest
   valid JSON document with `schema_version: 1` and only the requested setting;
   the old value is absent plus its documented default. Do not create a file for
   an ambiguous request.
   文件缺失是正常的首次写入前状态。对只读问题，说明检查了哪个路径、解释相关默认值、报告未更改任何文件，然后停止。对明确无歧义的更改，创建包含 `schema_version: 1` 且只含被请求设置的最小有效 JSON 文档；旧值视为"不存在"，并附其文档化的默认值。对含糊的请求不要创建文件。
4. If the allowed read path is unavailable or the read fails for any reason
   other than not found (permission denied, sandbox block, malformed JSON, I/O
   error), say which path was checked and which failure happened, explain the
   relevant default behavior instead of inventing a saved value, and stop — do
   not hunt for substitute configuration files, probe writability, or try a
   different config root.
   如果允许的读取路径不可用，或读取因"未找到"之外的任何原因失败（权限被拒、沙箱拦截、JSON 格式错误、I/O 错误），应说明检查了哪个路径、发生了哪种失败，解释相关的默认行为而不是编造一个已保存的值，然后停止 —— 不要寻找替代配置文件、探测可写性，或尝试其他配置根目录。
5. Tie the answer to the concrete key path, such as
   `runtime_capabilities["plugin:tbh-reminders:reminder:skill-reminder"].enabled`.
   将回答锚定到具体的键路径，例如
   `runtime_capabilities["plugin:tbh-reminders:reminder:skill-reminder"].enabled`。

## Where Muse Code Keeps Its Settings / Muse Code 的设置存放位置

This skill answers from the settings surface only:

本技能只依据设置表面回答：

- Config root: `$XDG_CONFIG_HOME/muse`, else `$HOME/.config/muse` — holds
  `settings.json` (saved settings and defaults), `auth.json`, and `trust.json`; report
  auth/trust presence only, never contents.
  配置根目录：`$XDG_CONFIG_HOME/muse`，否则 `$HOME/.config/muse` —— 其中保存 `settings.json`（已保存设置与默认值）、`auth.json` 和 `trust.json`；对 auth/trust 只报告其是否存在，绝不报告其内容。

That config root is the whole answer surface for this skill. A configuration
file outside it is not Muse Code configuration — do not hunt for substitute
configuration files, and never present an unrelated file as this session's
active or effective configuration, even when the Muse settings file is
unreadable.

该配置根目录是本技能全部的回答范围。其之外的配置文件不属于 Muse Code 配置 —— 不要寻找替代配置文件，也绝不把无关文件当作本次会话的活动或生效配置来呈现，即使 Muse 设置文件不可读也是如此。

Resolve exactly one root. When `XDG_CONFIG_HOME` is set, `$HOME/.config/muse` is
outside the active root: never read, list, test-write, or report it as a fallback
or schema source.

只解析唯一的一个根目录。当 `XDG_CONFIG_HOME` 已设置时，`$HOME/.config/muse` 位于活动根目录之外：绝不读取、列出、试写它，也绝不把它报告为后备或 schema 来源。

Session state, logs, crashes, and what-happened questions are not settings
questions: hand those to the `doctor` skill instead of reading data-dir
files from here.

会话状态、日志、崩溃和"发生了什么"类问题不属于设置问题：把它们交给 `doctor` 技能，而不是从这里读取数据目录文件。

## Saved Versus Live State / 已保存状态与实时状态

A `settings.json` read-back proves saved configuration only; it never proves
current TUI, in-flight run, active tool surface, or provider-request state. A
verified config write changes the permanent saved setting only. It is guaranteed
to be loaded on the next Muse Code launch; do not claim that the current process
reloaded it. Relaunch Muse Code, not the terminal application, when the user
wants the saved value to become the new runtime default.

读回 `settings.json` 只能证明已保存的配置；它永远不能证明当前的 TUI、进行中的运行、活动工具表面或 provider 请求状态。经验证的配置写入只改变持久化的已保存设置。它保证在下一次 Muse Code 启动时被加载；不要声称当前进程已重新加载它。当用户希望已保存的值成为新的运行时默认值时，应重新启动 Muse Code，而不是终端应用。

Ordinary conversation does not invoke a TUI settings control. In the interactive
TUI, `/models`, `/effort`, and `/settings` are user-side controls; never claim
that you executed one. Telling the assistant `/effort` or describing a desired
setting in chat does not run that control.

普通对话不会触发 TUI 设置控件。在交互式 TUI 中，`/models`、`/effort` 和 `/settings` 是用户侧的控件；绝不能声称你执行了其中之一。在聊天中对助手说 `/effort` 或描述期望的设置，并不会运行该控件。

Current-runtime controls are outside this skill's mutation scope. `/models`,
`/effort`, and `/settings` are user-side controls; mention the relevant control
when the user also wants an immediate current-session change, but never claim
that you executed it. For Meta, the persistent effort tiers are `minimal`,
`low`, `medium`, `high`, `xhigh`, `max`, and `ultra`. `high` is the default Meta
baseline; `xhigh` is the opt-in premium precision tier. `ultra` remains the
saved client selection, uses `max` reasoning on the Meta wire (ADR 19425 D63), and currently
enables proactive workflow/delegation guidance when that tool surface is
available. It may proactively run multi-agent workflows and increase token
usage quickly. Do not claim the proposed 64-slot unconfigured-root default is
current; that capacity change is still a target.

当前运行时控件不在本技能的修改范围之内。`/models`、`/effort` 和 `/settings` 是用户侧控件；当用户还希望立即改变当前会话时，可以提及相关控件，但绝不能声称你执行了它。对 Meta 而言，持久化的努力档位（effort tier）为 `minimal`、`low`、`medium`、`high`、`xhigh`、`max` 和 `ultra`。`high` 是 Meta 的默认基线；`xhigh` 是需要主动选择的高级精确档。`ultra` 仍是客户端保存的选择，在 Meta 线路上使用 `max` 推理（ADR 19425 D63），且在相应工具表面可用时当前会启用主动式工作流/委派指导。它可能主动运行多智能体工作流并快速增加 token 用量。不要声称拟议的 64 槽位未配置根默认值已是现状；该容量变更仍是一个目标。

You may honor a behavioral preference for the current task without claiming a
product setting changed only when it actually governs your response or tool
choices. For example, answer more briefly or avoid subagents when asked, while
leaving the corresponding product setting untouched. Do not pretend to emulate
renderer behavior such as reasoning summaries or verbose-output repainting, or
provider, permission, or other external-setting behavior. Do not invent
reasoning-effort tiers, budgets, ratios, mappings, or effects; use only the
stable facts above. Do not infer token budget, quality, speed, or generic
workflow effects from tier names. In particular, do not say a higher provider
tier performs more, deeper, longer, or better reasoning. The only current
workflow delta named by this contract is `ultra`'s proactive guidance; it does
not prove that provider reasoning itself is deeper or better.

只有当某个行为偏好确实支配着你的回复或工具选择时，你才可以在当前任务中遵从它，而不声称产品设置发生了变化。例如，被要求时回答得更简短或避免使用子代理，同时保持相应的产品设置原封不动。不要假装模拟渲染器行为（如推理摘要或详细输出重绘），或 provider、权限及其他外部设置的行为。不要编造推理努力档位、预算、比率、映射或效果；只使用上述稳定事实。不要从档位名称推断 token 预算、质量、速度或一般性工作流效果。尤其不要说更高的 provider 档位会进行更多、更深、更长或更好的推理。本契约点名的唯一现行工作流差异是 `ultra` 的主动式指导；它并不能证明 provider 推理本身更深或更好。

## Persistent Writes On Explicit Change / 明确更改时的持久化写入

Do not claim that a saved-file edit changed the current interactive TUI. An
explicit request such as "use xhigh", "turn summaries off", or "set verbose
output to less" authorizes the corresponding persistent setting edit when the
request is unambiguous. If the requested value is ambiguous, ask which saved
value to use and STOP without a mutation. A clarification timeout,
auto-cancellation, dismissal, empty reply, or missing reply is not authorization:
leave the setting unchanged. Never pick a value because it seems more common,
likely, stronger, or closer to a default.

不要声称对已保存文件的编辑改变了当前交互式 TUI。"使用 xhigh"、"关闭摘要"或"把详细输出设为 less"这类明确请求，在该请求无歧义时授权相应的持久化设置编辑。如果请求的值含糊，应询问应使用哪个已保存的值并停止（STOP），不做任何修改。澄清超时、自动取消、关闭、空回复或缺失回复都不构成授权：保持该设置不变。绝不要因为某个值看起来更常见、更可能、更强或更接近默认值而选择它。

【评论】"澄清超时或沉默不构成授权"是一条值得注意的安全条款：它防止智能体在用户未回应时擅自替用户做出持久化决定。

Use these stable key contracts:

使用这些稳定的键契约：

- `reasoning_effort` is the top-level persistent effort tier. Meta accepts
  `minimal`, `low`, `medium`, `high`, `xhigh`, `max`, or `ultra`.
  `reasoning_effort` 是顶层的持久化努力档位。Meta 接受 `minimal`、`low`、`medium`、`high`、`xhigh`、`max` 或 `ultra`。
- `tui.reasoning_summaries` is a persistent boolean; absent defaults to `true`.
  `tui.reasoning_summaries` 是一个持久化布尔值；缺省时默认为 `true`。
- `tui.verbose_output` is one of `less`, `edits`, or `more`; absent defaults to
  `edits` (shown as `edits & writes`).
  `tui.verbose_output` 取 `less`、`edits` 或 `more` 之一；缺省时默认为 `edits`（显示为 `edits & writes`）。
- `tui.terminal_background` is one of `auto`, `light`, or `dark`; absent
  defaults to `auto` (detect from the startup probe). `light`/`dark` override
  the detected terminal background for theme resolution at the next startup.
  `tui.terminal_background` 取 `auto`、`light` 或 `dark` 之一；缺省时默认为 `auto`（从启动探测中检测）。`light`/`dark` 会在下次启动时覆盖检测到的终端背景，用于主题解析。

When a write is requested:

当被要求写入时：

1. Read the current settings first.
   先读取当前设置。
2. Make the smallest JSON change that satisfies the request. If the read proves
   the file is missing, create a minimal object containing `schema_version: 1`
   and only the requested setting. A denied or malformed file is not a missing
   file and MUST NOT be replaced.
   进行满足请求的最小 JSON 修改。如果读取证实文件缺失，创建一个包含 `schema_version: 1` 且只含被请求设置的最小对象。被拒绝或格式错误的文件不属于缺失文件，绝不能被替换。
3. Preserve unrelated keys, formatting-sensitive values, and unknown fields.
   保留无关的键、对格式敏感的值和未知字段。
4. Re-read the file to prove the saved value changed. Do not treat the reread as
   live/effective-state evidence.
   重新读取文件以证明已保存的值发生了变化。不要把这次重读当作实时/生效状态的证据。
5. Report the old saved value (or absent plus its effective default), the new
   saved value, that it is permanent, that the current session is unchanged,
   and that it takes effect on the next Muse Code launch. Relaunch Muse Code; do
   not tell the user to restart the terminal application.
   报告旧的已保存值（或"不存在"及其生效默认值）、新的已保存值、该更改是持久的、当前会话保持不变、以及它将在下一次 Muse Code 启动时生效。重新启动 Muse Code；不要让用户重启终端应用。

For a YOLO shell fallback, an earlier successful shell read does not replace the
required `edit_file` or `write_file` attempt. Try the file mutation tool on the
same path first; use the atomic structured shell mutation only after that tool's
explicit denial in this turn.

对于 YOLO shell 后备方案，更早的成功 shell 读取不能替代必需的 `edit_file` 或 `write_file` 尝试。先在同一路径上尝试文件修改工具；只有在该工具在本轮明确拒绝之后，才使用原子化的结构化 shell 修改。

If the allowed access path above still cannot read or write the active config,
report that no saved setting changed and stop. Do not probe another config root
or bypass a security control.

如果上述允许的访问路径仍无法读取或写入活动配置，应报告没有任何已保存设置被更改并停止。不要探测其他配置根目录，也不要绕过安全控制。

## Completion Report / 完成报告

For read-only answers, include:

对只读回答，应包含：

- the settings path checked;
  检查过的设置路径；
- the key path or default that answers the question;
  回答该问题的键路径或默认值；
- whether any file was changed.
  是否更改了任何文件。

For a verified write, use this user-facing summary shape:

对经验证的写入，使用以下面向用户的汇总格式：

```text
Saved changes
- <key>: <old saved value or absent + default> -> <new saved value>

Permanent: yes
Current session: unchanged
Effective: next Muse Code launch
Reload: required - relaunch Muse Code; no terminal-application restart
Verified: `settings.json` reread at <path>
```

For a setting such as `reasoning_effort` with higher-precedence startup input,
append `unless a launch flag overrides the saved value` to the
Effective line. Name any setting intentionally left unchanged.

对诸如 `reasoning_effort` 这类存在更高优先级启动输入的设置，在 Effective 行后附加 `unless a launch flag overrides the saved value`。指出任何有意保持不变的设置。

If the read, write, or verification fails, use:

如果读取、写入或验证失败，使用：

```text
Changed: no
Reason: <exact path and failure>
Reload: not applicable
```
