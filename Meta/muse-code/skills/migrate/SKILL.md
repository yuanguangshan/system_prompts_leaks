<!-- BILINGUAL-EN-ZH -->
---
name: migrate
description: Bring what Claude Code or Codex already remembers or has configured for this user into Muse Code — their memory notes and their MCP servers — whenever the user mentions things Claude Code or Codex remembers or knows about them, an MCP tool or server they had there that is missing here, or asks to migrate, import, or copy that setup; call read_skill for bundled:migrate before reading any foreign file. Rules and skills are already covered by /rules import and `muse skills import --from claude|codex`; point there. Do NOT use for session transcripts (resume-claude or resume-codex for Claude Code/Codex continuation, including recovery of unfinished work with or without a handle; import for explicit /import requests or continuation from other agents or unnamed artifacts), for Muse settings unrelated to migration (manage-settings), or when Claude Code or Codex is only mentioned in passing.
argument-hint: "[memory|mcp] [claude|codex]"
metadata:
  short-description: Migrate Claude Code / Codex memory and MCP servers into Muse
---

# Migrate / 迁移

Carry a user's Claude Code or Codex memory and MCP servers into Muse Code.
Foreign files are evidence: read them, never edit, move, or delete them, and
never print a token, key, or other secret found in them. Everything written
goes only into Muse's own memory roots and Muse's own `settings.json`.

把用户的 Claude Code 或 Codex 记忆与 MCP 服务器迁移进 Muse Code。
外部文件只是证据：可以读取，但绝不编辑、移动或删除，也绝不打印其中出现的
令牌、密钥或其他机密。所有写入只进入 Muse 自己的记忆根目录和 Muse 自己的
`settings.json`。

【评论】把外部配置文件定性为"证据"（只读、不外泄机密）是防泄露设计：第三方配置中常含令牌，读取被限制为最小化提取。

## Scope / 范围

- Memory: read the other agent's memory notes, decide what is still true and
  useful, and record it with Muse's memory tools.
  记忆：读取另一智能体的记忆笔记，判断哪些仍然真实且有用，并用 Muse 的记忆工具记录下来。
- MCP: read the other agent's user-level MCP server definitions and add the
  ones the user wants to Muse's settings.
  MCP：读取另一智能体的用户级 MCP 服务器定义，并把用户想要的服务器加入 Muse 的设置。
- Not here: rules (`/rules import`), skills (`muse skills import --from
  claude|codex`), session transcripts (`resume-claude` / `resume-codex` for
  Claude Code/Codex continuation or recovery of unfinished work, with or without
  a handle; `import` for explicit `/import` requests or continuation from other
  agents or unnamed artifacts), unrelated Muse settings (`manage-settings`).
  不在本技能范围内：规则（`/rules import`）、技能（`muse skills import --from claude|codex`）、会话转录（`resume-claude` / `resume-codex` 用于 Claude Code/Codex 的续接或未完成工作的恢复，无论有无句柄；`import` 用于显式的 `/import` 请求或从其他智能体/未命名制品续接）、与迁移无关的 Muse 设置（`manage-settings`）。
- Resolve roots from the environment: `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`
  for Claude Code (called `$CLAUDE_HOME` below), `${CODEX_HOME:-$HOME/.codex}`
  for Codex (an empty `CODEX_HOME` means unset), and
  `${XDG_CONFIG_HOME:-$HOME/.config}/muse` for Muse's own config.
  从环境中解析根目录：Claude Code 用 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`（下文称 `$CLAUDE_HOME`），Codex 用 `${CODEX_HOME:-$HOME/.codex}`（`CODEX_HOME` 为空表示未设置），Muse 自身配置用 `${XDG_CONFIG_HOME:-$HOME/.config}/muse`。

## Where Claude Code keeps memory / Claude Code 把记忆存放在哪里

- Directory: `$CLAUDE_HOME/projects/<slug>/memory/`. `<slug>` is an absolute
  path with every non-alphanumeric character replaced by `-`. Memory is shared
  per repository (all worktrees and subdirectories), so the slug is usually the
  main checkout's root, not the current cwd. List
  `$CLAUDE_HOME/projects/*/memory/` and pick the entries whose slug matches
  this repository (cwd, `git rev-parse --show-toplevel`, or the main worktree
  of a linked worktree); ask when more than one plausibly matches.
  目录：`$CLAUDE_HOME/projects/<slug>/memory/`。`<slug>` 是把绝对路径中所有非字母数字字符替换为 `-` 的结果。记忆按仓库共享（覆盖所有工作树与子目录），因此 slug 通常是主检出的根路径，而非当前 cwd。列出 `$CLAUDE_HOME/projects/*/memory/`，挑选 slug 与本仓库匹配的条目（依据 cwd、`git rev-parse --show-toplevel` 或链接工作树的主工作树）；当多个条目都可能匹配时应询问。
- A settings key `autoMemoryDirectory` in `~/.claude/settings.json` or the
  project's `.claude/settings*.json` relocates that directory.
  `~/.claude/settings.json` 或项目 `.claude/settings*.json` 中的设置键 `autoMemoryDirectory` 可重新定位该目录。
- `MEMORY.md` is the index: one line per note, `- [Title](file.md) — hook`.
  Read it first; it says what exists.
  `MEMORY.md` 是索引：每条笔记一行，形式为 `- [Title](file.md) — hook`。先读它；它说明已有哪些内容。
- Each note is `<name>.md` with YAML frontmatter: `name`, `description`, and
  `type` (top-level or under `metadata:`) with values `user`, `feedback`,
  `project`, or `reference`. These are the same four types Muse uses.
  每条笔记是带 YAML frontmatter 的 `<name>.md`：包含 `name`、`description` 与 `type`（位于顶层或 `metadata:` 之下），取值为 `user`、`feedback`、`project` 或 `reference`。这与 Muse 使用的四种类型相同。

## Where Codex keeps memory / Codex 把记忆存放在哪里

- Codex memory is opt-in (`[features] memories = true` in
  `$CODEX_HOME/config.toml`). When it has run, the durable files are in
  `$CODEX_HOME/memories/`: `MEMORY.md` (registry), `memory_summary.md`
  (user profile, preferences, tips, index), `rollout_summaries/*.md` (one per
  past thread, each naming its cwd), and `extensions/*/`.
  Codex 记忆是可选功能（`$CODEX_HOME/config.toml` 中的 `[features] memories = true`）。启用并运行后，持久化文件位于 `$CODEX_HOME/memories/`：`MEMORY.md`（登记表）、`memory_summary.md`（用户画像、偏好、提示、索引）、`rollout_summaries/*.md`（每个历史会话一个文件，各自标注其 cwd）以及 `extensions/*/`。
- If that directory is missing but `$CODEX_HOME/memories_1.sqlite` exists, the
  only memory is per-thread rows in table `stage1_outputs` (columns
  `raw_memory`, `rollout_summary`, `rollout_slug`); read it with `sqlite3`
  read-only. An empty table means Codex has nothing to migrate; say so.
  若该目录不存在但 `$CODEX_HOME/memories_1.sqlite` 存在，则唯一的记忆是表 `stage1_outputs` 中按会话存储的行（列为 `raw_memory`、`rollout_summary`、`rollout_slug`）；用 `sqlite3` 以只读方式读取。空表说明 Codex 没有可迁移的内容；应如实说明。
- Codex memory is global, not per project. Use the cwd named in each rollout
  summary to decide whether a fact belongs to this project.
  Codex 记忆是全局的，不按项目划分。用每份 rollout 摘要中标注的 cwd 判断某条事实是否属于本项目。
- `$CODEX_HOME/AGENTS.md` and `history.jsonl` are rules and prompt history,
  not memory.
  `$CODEX_HOME/AGENTS.md` 与 `history.jsonl` 是规则和提示词历史，不是记忆。

## Recording memory in Muse / 在 Muse 中记录记忆

- Muse's memory tools are `read_memory`, `add_memory`, and `edit_memory`.
  Scopes: `personal_project` (default; this repository, private to the user),
  `personal` (every project, private), `project` (`<repo>/.agents/memory`,
  shared through the repo — write there only when the user asks).
  Muse 的记忆工具是 `read_memory`、`add_memory` 与 `edit_memory`。作用域：`personal_project`（默认；本仓库，仅用户私有）、`personal`（所有项目，私有）、`project`（`<repo>/.agents/memory`，通过仓库共享——仅当用户要求时才写入）。
- Read Muse's own `MEMORY.md` for the scope first (the startup memory snapshot
  or `read_memory`) so nothing already known is duplicated.
  先读取 Muse 自身对应作用域的 `MEMORY.md`（启动记忆快照或 `read_memory`），避免重复记录已知内容。
- Do not copy blindly. Keep facts that are still true and would change how
  Muse works for this user; drop notes tied to Claude Code or Codex internals
  (their tool names, session ids, their own bugs) unless the user wants them.
  Rewrite "Claude Code" or "Codex" wording only where it names the agent that
  should act, never where it names the tool the fact is about.
  不要盲目复制。保留仍然真实、且会改变 Muse 为该用户工作方式的事实；丢弃与 Claude Code 或 Codex 内部细节绑定的笔记（它们的工具名、会话 id、它们自身的 bug），除非用户想要。仅在"Claude Code"或"Codex"指代应当行动的智能体时才改写措辞，在其指代事实所涉工具时绝不改写。
- Write each kept note with `add_memory`: keep the original `type`, give a
  one-line `description`, use a relative `.md` path (letters, digits, `-`,
  `_`; no `..`, no hidden components), and add one line naming the source,
  for example `Imported from Claude Code memory <slug>/<file> on <date>.`
  Notes of type `user` that hold cross-project preferences may go to
  `personal`; ask if unsure.
  用 `add_memory` 写入每条保留的笔记：保留原 `type`，写一行式 `description`，使用相对 `.md` 路径（字母、数字、`-`、`_`；不允许 `..`，不允许隐藏目录成分），并加一行注明来源，例如 `Imported from Claude Code memory <slug>/<file> on <date>.`。持有跨项目偏好的 `user` 类型笔记可放入 `personal`；不确定时先询问。
- Add one index line per note to the scope's `MEMORY.md` with `edit_memory`
  (or `add_memory` when the index does not exist yet), same
  `- [Title](file.md) — hook` shape, so the next session sees it in the
  startup snapshot.
  用 `edit_memory` 在该作用域的 `MEMORY.md` 中为每条笔记添加一行索引（索引尚不存在时用 `add_memory`），保持相同的 `- [Title](file.md) — hook` 形式，让下一个会话能在启动快照中看到它。

## Where Claude Code keeps MCP servers / Claude Code 把 MCP 服务器存放在哪里

- User level: `$CLAUDE_CONFIG_DIR/.claude.json` when `CLAUDE_CONFIG_DIR` is
  set, else `$HOME/.claude.json`; top-level key `mcpServers`. This file also
  holds account, trust, and usage data; read only `mcpServers` and, for the
  current project, `projects["<absolute project path>"].mcpServers` (local
  scope). Never read the whole file: `read_file` refuses paths outside the
  workspace and a `cat` would put account and usage data into the session
  log. Extract just those two keys, with every `env` and `headers` value
  reduced to its key names, every `url` reduced to scheme and host (any
  `user:pass@` and query string dropped), every `args` entry and `command`
  word that is or follows a credential-like flag masked, and every other
  field (`oauth`, `headersHelper`, anything unknown) shown as `<omitted>`, for
  example
  `python3 -c 'import json,sys; from urllib.parse import urlsplit; d=json.load(open(sys.argv[1])); projects=d.get("projects", {}); local=next((s for s in (projects.get(k, {}).get("mcpServers", {}) for k in sys.argv[2:] if k) if s), {}); secretish=lambda a: any(w in str(a).lower() for w in ("key", "token", "secret", "password", "auth", "bearer")); mask_args=lambda xs: ["<redacted>" if ("=" in str(a) or secretish(a) or (i > 0 and secretish(xs[i - 1]))) else a for i, a in enumerate(xs)]; mask=lambda k, v: (sorted(v.keys()) if k in ("env", "headers") and isinstance(v, dict) else f"{urlsplit(v).scheme}://{urlsplit(v).hostname}/..." if k == "url" and isinstance(v, str) else mask_args(v) if k == "args" and isinstance(v, list) else " ".join(str(x) for x in mask_args(v.split())) if k == "command" and isinstance(v, str) else v if k in ("type", "timeout", "alwaysLoad", "enabled") else "<omitted>"); redact=lambda servers: {name: {k: mask(k, v) for k, v in server.items()} for name, server in servers.items()}; print(json.dumps({"user": redact(d.get("mcpServers", {})), "local": redact(local)}, indent=1))' "${CLAUDE_CONFIG_DIR:-$HOME}/.claude.json" "$(dirname "$(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null)")" "$(git rev-parse --show-toplevel 2>/dev/null)" "$PWD"`.
  The `projects[...]` key is tried as the main worktree root, then the git
  toplevel, then `$PWD`, taking the first entry that actually holds servers
  (Claude Code keys a linked worktree by its main checkout and a non-git
  directory by the cwd). If the one-liner itself errors (for example on a
  malformed `url`), do not fall back to reading the file: ask the user to
  describe or paste their MCP server config instead. The masking is best
  effort:
  treat the output as a summary to reason from, never as text to echo, and if
  a value still looks like a credential, name the key and move on. Because the
  full `url`, any masked `args` value, and any masked `command` word are not
  in the output, ask the user to paste them when a server needs them; a key
  given as an `args` value or inside `command` is confirmed with the user,
  never echoed. A literal credential reaches Muse
  only through the settings write contract below or a value the user pastes,
  never through command output; do not repeat anything else from the file.
  用户级：设置了 `CLAUDE_CONFIG_DIR` 时为 `$CLAUDE_CONFIG_DIR/.claude.json`，否则为 `$HOME/.claude.json`；顶层键 `mcpServers`。该文件还保存账户、信任与用量数据；只读取 `mcpServers`，以及当前项目对应的 `projects["<absolute project path>"].mcpServers`（local 作用域）。绝不读取整个文件：`read_file` 会拒绝工作区之外的路径，而 `cat` 会把账户与用量数据写进会话日志。只提取这两个键，并把每个 `env` 与 `headers` 的值缩减为其键名、每个 `url` 缩减为协议与主机（丢弃任何 `user:pass@` 与查询串）、每个属于或跟随类凭据旗标的 `args` 条目与 `command` 词做掩码，其余所有字段（`oauth`、`headersHelper` 及一切未知字段）显示为 `<omitted>`，例如上面保留原文的单行命令。`projects[...]` 键依次按主工作树根、git 顶层、`$PWD` 尝试，取第一个确实含服务器的条目（Claude Code 用主检出来键控链接工作树，用 cwd 键控非 git 目录）。若单行命令本身报错（例如 `url` 格式非法），不要退回读取文件：改为请用户描述或粘贴其 MCP 服务器配置。掩码是尽力而为：把输出当作供推理的摘要，绝不当作可回显的文本；若某个值看起来仍像凭据，只点名该键然后继续。由于完整 `url`、被掩码的 `args` 值与被掩码的 `command` 词不在输出中，当某个服务器需要它们时请用户粘贴；以 `args` 值或 `command` 内形式给出的密钥须与用户确认，绝不回显。字面凭据只能通过下文的设置写入契约或用户粘贴的值进入 Muse，绝不能通过命令输出；不得复述文件中的其他任何内容。
  【评论】该条目先用掩码化提取避免机密进入会话日志，又在命令出错时宁可询问用户也不读原文件，属于纵深防御式的最小权限处理。
- Project level: `<project root>/.mcp.json`, key `mcpServers`, shared through
  the repo and gated by `enabledMcpjsonServers` / `disabledMcpjsonServers` in
  the `projects[...]` entry. Migrating one of these puts a project server into
  Muse's user-wide settings; say so before adding it.
  项目级：`<project root>/.mcp.json`，键 `mcpServers`，通过仓库共享，并受 `projects[...]` 条目中 `enabledMcpjsonServers` / `disabledMcpjsonServers` 的门控。迁移其中之一会把项目级服务器放进 Muse 的用户级设置；添加前应说明这一点。
- Skip servers Claude Code itself does not run: `enabled: false`, names in a
  `disabledMcpjsonServers` list, and anything under managed policy
  (`/etc/claude-code/managed-mcp.json`, plugin caches).
  跳过 Claude Code 自身不会运行的服务器：`enabled: false`、列入 `disabledMcpjsonServers` 清单的名称，以及受托管策略约束的一切（`/etc/claude-code/managed-mcp.json`、插件缓存）。
- Entry fields: `type` (`stdio` when absent, `http` or `streamable-http`,
  `sse`, `ws`, `sdk`), `command`, `args`, `env`, `url`, `headers`,
  `headersHelper`, `oauth`, `timeout`, `alwaysLoad`. Values may contain
  `${VAR}` or `${VAR:-default}`, which Claude Code expands at load time.
  条目字段：`type`（缺省时为 `stdio`，另有 `http` 或 `streamable-http`、`sse`、`ws`、`sdk`）、`command`、`args`、`env`、`url`、`headers`、`headersHelper`、`oauth`、`timeout`、`alwaysLoad`。值中可含 `${VAR}` 或 `${VAR:-default}`，Claude Code 会在加载时展开。

## Where Codex keeps MCP servers / Codex 把 MCP 服务器存放在哪里

- `$CODEX_HOME/config.toml`, tables `[mcp_servers.<name>]`. A trusted project
  may add more in `<root>/.codex/config.toml`; admin layers in `/etc/codex/`
  are not the user's and are skipped.
  `$CODEX_HOME/config.toml`，表 `[mcp_servers.<name>]`。受信任的项目可在 `<root>/.codex/config.toml` 中追加；`/etc/codex/` 中的管理员层不属于用户，予以跳过。
- stdio fields: `command`, `args`, `env` (literal map), `env_vars` (names of
  variables Codex forwards from its own environment), `cwd`. HTTP fields:
  `url`, `bearer_token_env_var`, `http_headers`, `env_http_headers`,
  `http_headers_helper`, `auth`. Shared: `enabled`, `required`,
  `startup_timeout_sec`, `tool_timeout_sec`, `enabled_tools`,
  `disabled_tools`, approval settings. Skip a server with `enabled = false`;
  Codex does not run it either.
  stdio 字段：`command`、`args`、`env`（字面映射）、`env_vars`（Codex 从自身环境转发的变量名）、`cwd`。HTTP 字段：`url`、`bearer_token_env_var`、`http_headers`、`env_http_headers`、`http_headers_helper`、`auth`。共享字段：`enabled`、`required`、`startup_timeout_sec`、`tool_timeout_sec`、`enabled_tools`、`disabled_tools`、审批设置。`enabled = false` 的服务器予以跳过；Codex 同样不会运行它。

## Adding MCP servers to Muse / 向 Muse 添加 MCP 服务器

- File: `${XDG_CONFIG_HOME:-$HOME/.config}/muse/settings.json`, key
  `mcpServers` (camelCase). Never write a `mcp_servers` key next to it: when
  both keys are present the loader drops the whole MCP member, so every server
  the user already had stops loading while the rest of the settings still
  apply. If the file already carries the legacy `mcp_servers` key, rename it
  to `mcpServers` (keeping its entries) in the same edit before adding
  servers. Read the file first; a missing file is created as
  `{"schema_version": 1, "mcpServers": {...}}`. Preserve every other key.
  Servers Muse already defines under the same name are left alone.
  文件：`${XDG_CONFIG_HOME:-$HOME/.config}/muse/settings.json`，键 `mcpServers`（驼峰式）。绝不在其旁再写一个 `mcp_servers` 键：两个键同时存在时，加载器会丢弃整个 MCP 成员，导致用户已有的所有服务器停止加载，而设置的其他部分仍然生效。若文件已带有旧式 `mcp_servers` 键，应在同一次编辑中先将其更名为 `mcpServers`（保留其条目）再添加服务器。先读取文件；文件缺失时创建为 `{"schema_version": 1, "mcpServers": {...}}`。保留所有其他键。Muse 已以同名定义的服务器保持不动。
  【评论】这里把加载器"双键并存即整体丢弃"的实现行为固化为操作禁忌，并给出同次编辑内改名的补救流程。
- Write contract (same as `manage-settings`): resolve the one path from the
  environment, then use `edit_file` or `write_file` for the smallest JSON
  change. If those tools reject the app-private path, fall back to a shell
  edit only when the startup security context already says YOLO mode is
  active (approval bypassed, shell sandbox off, workspace trusted), and then
  only through a JSON parser plus atomic replace and reread; never use text
  substitution. Otherwise stop and hand the user the exact `mcpServers` block
  to paste. Never ask the user to change a security mode for this edit.
  写入契约（与 `manage-settings` 相同）：从环境中解析该唯一路径，然后用 `edit_file` 或 `write_file` 做最小的 JSON 修改。若这些工具拒绝该应用私有路径，则仅当启动安全上下文已表明 YOLO 模式生效（审批被绕过、shell 沙箱关闭、工作区受信任）时，才退回用 shell 编辑，且只能通过 JSON 解析器加原子替换并回读完成；绝不使用文本替换。否则停止，把确切的 `mcpServers` 块交给用户粘贴。绝不为此编辑要求用户更改安全模式。
- Entry shape:
  条目形式：

  ```json
  "github": {
    "type": "stdio",
    "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {"GITHUB_TOKEN": "<literal value>"},
    "mode": "optional"
  }
  ```

  ```json
  "docs": {"type": "streamable-http", "url": "https://example/mcp",
           "headers": {"Authorization": "Bearer <literal value>"}, "mode": "optional"}
  ```

- Mapping: Claude `stdio` (or a bare `command`) and Codex `command` become
  `type: "stdio"` with `command`, `args`, `env`. Claude `http` /
  `streamable-http` and Codex `url` become `type: "streamable-http"` with `url`
  and `headers` (Codex `http_headers`). Never write the extraction output
  itself into settings: a `<redacted>` entry, a `url` reduced to
  `scheme://host/...`, or a key-names-only `env`/`headers` list is a summary,
  not a value. Where the summary hid something (the mask also hides harmless
  entries such as `--keyring` or `--config=...`, and the cut drops a query
  such as `?tenant=acme`), treat it like a `${VAR}` value: ask the user for
  it by position or key name only, write it only when they supply or approve
  it, and leave the server out rather than write the placeholder or the
  shortened `url`. Codex `enabled` and `tool_timeout_sec`
  copy verbatim under the same names and take effect. Codex
  `startup_timeout_sec`, `enabled_tools`, and `disabled_tools` also copy under
  the same names but are stored, not enforced yet (spec 12857): Muse's startup
  budget is fixed and every tool stays exposed, so list them in the completion
  report as settings the user still has to check. Always add
  `"mode": "optional"` so a server that fails here cannot block Muse startup
  (Muse's canonical resolver parses `required`, but the startup gate is fed
  from the typed settings carrier, which knows only `mode` — default required
  — and drops an unknown `required`, so `required: false` alone leaves the
  server in the default required mode; and never write both on one server,
  because `required` plus `mode` is an ambiguous-alias fault that drops the
  whole user MCP settings member, not just that server (spec 12857)).
  映射：Claude 的 `stdio`（或仅有 `command`）与 Codex 的 `command` 变为 `type: "stdio"`，带 `command`、`args`、`env`。Claude 的 `http` / `streamable-http` 与 Codex 的 `url` 变为 `type: "streamable-http"`，带 `url` 和 `headers`（对应 Codex 的 `http_headers`）。绝不把提取输出本身写进设置：`<redacted>` 条目、被缩减为 `scheme://host/...` 的 `url`、或仅含键名的 `env`/`headers` 清单都是摘要而非值。凡摘要隐藏了内容之处（掩码也会隐藏无害条目如 `--keyring` 或 `--config=...`，截断会丢掉 `?tenant=acme` 这样的查询串），将其视同 `${VAR}` 值处理：仅按位置或键名向用户询问，只在用户提供或批准后才写入；宁可不上该服务器，也不要写入占位符或被缩短的 `url`。Codex 的 `enabled` 与 `tool_timeout_sec` 按原名原样复制并生效。Codex 的 `startup_timeout_sec`、`enabled_tools` 与 `disabled_tools` 也按原名复制，但只是存储、尚未强制生效（spec 12857）：Muse 的启动预算固定且所有工具保持暴露，因此要在完成报告中把它们列为用户仍需检查的设置。始终添加 `"mode": "optional"`，使在此处失败的服务器无法阻塞 Muse 启动（Muse 的规范解析器会解析 `required`，但启动门控读取的是类型化设置载体，它只认识 `mode`——默认为 required——并丢弃未知的 `required`，因此仅有 `required: false` 会让服务器停留在默认的 required 模式；而且绝不在同一服务器上同时写两者，因为 `required` 加 `mode` 是"歧义别名"故障，会丢弃整个用户 MCP 设置成员而不只是该服务器（spec 12857））。
- Muse does not expand `${VAR}` and starts stdio servers with only a small
  fixed allowlist of the user's environment (HOME, PATH, USER, LANG, TERM and
  similar) plus the literal `env` map, so a Claude `${VAR}` value or a Codex
  `env_vars`, `env_http_headers`, or `bearer_token_env_var` entry outside that
  allowlist has no automatic equivalent. Tell the user which variable each
  server needs; write the literal value only when the user provides or
  approves it, and never echo it back. Leave the server out rather than guess.
  Muse 不展开 `${VAR}`，且仅以用户环境的一个小型固定白名单（HOME、PATH、USER、LANG、TERM 等）加上字面 `env` 映射来启动 stdio 服务器，因此白名单之外的 Claude `${VAR}` 值或 Codex 的 `env_vars`、`env_http_headers`、`bearer_token_env_var` 条目没有自动等价物。告诉用户每个服务器需要哪个变量；仅当用户提供或批准时才写入字面值，且绝不回显。宁可不上该服务器，也不要猜测。
- Not supported in Muse, skip and report: `sse`, `ws`, `sdk` transports,
  `oauth`, `headersHelper`, `http_headers_helper`, `auth = "oauth"`. Drop
  Claude `timeout` and `alwaysLoad`, Codex `cwd`, and approval settings with a
  note.
  Muse 不支持、应跳过并报告：`sse`、`ws`、`sdk` 传输，`oauth`、`headersHelper`、`http_headers_helper`、`auth = "oauth"`。附带说明地丢弃 Claude 的 `timeout` 与 `alwaysLoad`、Codex 的 `cwd` 以及审批设置。
- After the edit, re-read the file to confirm the saved value. The change is
  permanent and takes effect on the next Muse Code launch; the current session
  does not load new servers.
  编辑后重新读取文件以确认保存的值。更改是永久的，并在下次 Muse Code 启动时生效；当前会话不会加载新服务器。

## Completion report / 完成报告

- Sources read (paths only) and what each held.
  已读取的来源（仅路径）及各自包含的内容。
- Memory: notes written per scope with their paths, notes skipped and why.
  记忆：按作用域列出已写入的笔记及其路径、已跳过的笔记及原因。
- MCP: servers added, servers skipped with the reason, variables the user
  still has to fill in (names only), and copied fields Muse stores but does
  not enforce yet (`startup_timeout_sec`, `enabled_tools`, `disabled_tools`).
  MCP：已添加的服务器、已跳过的服务器及原因、用户仍需填写的变量（仅名称），以及 Muse 已存储但尚未强制生效的字段（`startup_timeout_sec`、`enabled_tools`、`disabled_tools`）。
- Reminder that MCP changes apply on the next launch.
  提醒：MCP 更改在下次启动时生效。
- Confirmation that no foreign file was modified and no secret was printed.
  确认未修改任何外部文件、未打印任何机密。
