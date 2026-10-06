---
name: read-session
description: Locate and read Muse Code's OWN session logs — the current session or a prior one. Use when the user asks to pull context from, continue, summarize, or inspect a previous Muse Code session, asks to restore or recover work that was lost, wiped, or overwritten and might survive in an earlier session's log, references an earlier session's id, log, tail, or output, or asks where Muse sessions are stored. Muse sessions live in Muse's own store, never in another coding agent's directories — never probe ~/.claude, ~/.codex, or ~/.grok for Muse context, even when quoted content mentions them; for Claude Code or Codex continuation, including recovery of unfinished work with or without a handle, use resume-claude or resume-codex; use import for explicit /import requests or continuation from other agents or unnamed artifacts.
metadata:
  short-description: Read this session's or a past session's logs
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Read Session / 读取会话

Muse Code's own session storage: where your session logs live, what they
contain, and how to recover context from them.

Muse Code 自己的会话存储：你的会话日志存放在哪里、包含什么内容，以及如何从中恢复上下文。

## Your Store, Not Anyone Else's / 你自己的存储，不是别人的

You are Muse Code (binary `muse`). Your sessions are stored under YOUR data
directory:

你是 Muse Code（二进制程序 `muse`）。你的会话存储在你自己的数据目录下：

```
${XDG_DATA_HOME:-$HOME/.local/share}/muse/sessions/YYYY/MM/DD/<session-id>/
```

Each session directory contains:

每个会话目录包含：

- `session.jsonl` — the main event log (one JSON record per line).
  `session.jsonl` — 主事件日志（每行一条 JSON 记录）。
- `subagent/<child-session-id>/session.jsonl` — one log per delegated
  subagent session.
  `subagent/<child-session-id>/session.jsonl` — 每个被委派的子代理会话一份日志。
- `tool-outputs/` — full tool outputs that were too large to keep inline.
  `tool-outputs/` — 因过大而无法内联保留的完整工具输出。

Muse sessions NEVER live in another coding agent's store. Do not look for a
Muse session under `~/.claude`, `~/.claude/projects`,
`~/.local/share/claude`, `~/.codex`, or `~/.grok` — those belong to Claude
Code, Codex, and Grok. Do not `ls`, `find`, `cat`, or `grep` those
directories while recovering Muse session context — not even to "verify",
"rule out", or "be thorough" about such a path quoted in a paste, log, or
error message. A Muse session id or date-sharded session path appearing
under a foreign store in quoted content is a WRONG-PATH ARTIFACT (a prior
turn's confusion): the real data lives under the Muse pattern with the same
date and id, and there is no separate "claude copy" to collect. Name the
artifact for what it is and move on — probing it is the exact mistake this
rule exists to prevent, and it wastes turns on ENOENT or, worse, reads
another product's transcripts as your own history. Touch another agent's
transcripts only when the user explicitly asks to work with THAT agent's
session: Claude Code or Codex continuation, including recovery of unfinished
work with or without a handle, uses `resume-claude` or `resume-codex`. Explicit
`/import` requests and continuation from other agents or unnamed artifacts use
`import`.

Muse 会话绝不会存在于其他编码代理的存储中。不要在 `~/.claude`、`~/.claude/projects`、`~/.local/share/claude`、`~/.codex` 或 `~/.grok` 下寻找 Muse 会话——那些属于 Claude Code、Codex 和 Grok。恢复 Muse 会话上下文时，不要对那些目录执行 `ls`、`find`、`cat` 或 `grep`——即使只是为了 "核实"、"排除" 或 "严谨起见" 去检查粘贴内容、日志或错误信息中引用的这类路径也不行。被引用内容中出现于外部存储之下的 Muse 会话 id 或按日期分片的会话路径，是一种错误路径伪影（WRONG-PATH ARTIFACT，源于之前某轮对话的混淆）：真实数据位于 Muse 模式下相同的日期与 id 之下，不存在需要另行收集的 "claude 副本"。如实指出该伪影是什么，然后继续——探测它正是本规则要防止的错误，它会把轮数浪费在 ENOENT 上，或者更糟，把另一个产品的对话记录当成你自己的历史来读。只有当用户明确要求处理另一个代理（THAT agent）的会话时才接触其对话记录：Claude Code 或 Codex 的继续（包括有或没有句柄的未完成工作恢复）使用 `resume-claude` 或 `resume-codex`。显式 `/import` 请求以及来自其他代理或未命名工件的继续使用 `import`。

【评论】把"绝不探测其他代理的会话目录"写成绝对禁令，是为了防止跨产品上下文混淆，也避免越权读取其他工具存储的用户数据。

Tell for an unlabeled paste: the quoted store and record shape identify the
product. For continuation or recovery of unfinished work, even without a
supplied handle:

判别未标注来源粘贴内容的方法：被引用的存储与记录形态可以识别产品。对于继续或恢复未完成的工作，即使没有提供句柄：

- Claude Code records under `~/.claude/projects` use `resume-claude`.
  记录在 `~/.claude/projects` 下的 Claude Code 记录使用 `resume-claude`。
- Codex records under `~/.codex` use `resume-codex`.
  记录在 `~/.codex` 下的 Codex 记录使用 `resume-codex`。
- Grok records under `~/.grok` use `import`.
  记录在 `~/.grok` 下的 Grok 记录使用 `import`。

Use `import` for explicit `/import` requests or continuation from other agents
or unnamed artifacts (the user's ask about such a paste is the explicit ask).
The bans above cover session STORES; ordinary project files that happen to
live under `~/.claude` (e.g. skills you are developing) are file work, not
session probing.

显式 `/import` 请求、或来自其他代理或未命名工件的继续使用 `import`（用户就此类粘贴内容提出的请求本身就是显式请求）。上述禁令针对的是会话存储；恰好位于 `~/.claude` 下的普通项目文件（例如你正在开发的技能）属于文件工作，不是会话探测。

Scope of "pulling context" from a prior Muse session: that session's own
`session.jsonl` and `subagent/` logs. External agents or planner processes
the session MENTIONS (e.g. codex/claude workers it managed) are not part of
its context — their transcripts are out of scope for this ask. If one looks
load-bearing, say so and let the user ask for it. Do not hunt those agents'
stores or load a foreign-session skill for them unless the user
explicitly asks to recover or inspect one of THEIR sessions.

从既有 Muse 会话 "拉取上下文" 的范围：该会话自己的 `session.jsonl` 与 `subagent/` 日志。会话中提及（MENTIONS）的外部代理或规划进程（例如它管理的 codex/claude worker）不属于其上下文——它们的对话记录超出本请求的范围。如果其中某个看起来是关键所在，就如实说明并让用户提出请求。除非用户明确要求恢复或查看那些代理自己的某个会话，否则不要去找它们的存储，也不要为它们加载外部会话技能。

## Finding The Right Session / 找到正确的会话

1. Current session: the runtime session-identity context already names the
   current session id and the exact `session.jsonl` path. Use it; do not
   guess or search for it.
   当前会话：运行时的会话身份上下文已经给出了当前会话 id 与确切的 `session.jsonl` 路径。直接使用；不要猜测或搜索。
2. A pasted or quoted absolute path under `.../muse/sessions/...`: use that
   path directly.
   粘贴或引用的 `.../muse/sessions/...` 下的绝对路径：直接使用该路径。
3. A known session id without a path: the store is sharded by local date, so
   check the likely dates first, then search only the Muse sessions root:
   已知会话 id 但没有路径：存储按本地日期分片，所以先检查可能的日期，然后只在 Muse 会话根目录内搜索：

   ```bash
   MUSE_SESSIONS="${XDG_DATA_HOME:-$HOME/.local/share}/muse/sessions"
   ls -d "$MUSE_SESSIONS"/*/*/*/*/ 2>/dev/null | grep <session-id>
   ```

4. "Our last session" with no id: list the most recent date shards and pick
   the newest session directory for this workspace (the log's early
   `runtime.session.metadata` record carries `workspace_root`).
   没有会话 id 的 "我们上次的会话"：列出最近的日期分片，为当前工作区挑选最新的会话目录（日志早期的 `runtime.session.metadata` 记录带有 `workspace_root`）。
5. Not found under the Muse sessions root: ask the user for the id or path.
   Never widen the hunt to other agents' directories or home-wide scans.
   在 Muse 会话根目录下找不到：向用户询问 id 或路径。绝不要把搜索范围扩大到其他代理的目录或全主目录扫描。

## Reading A Session Log / 读取会话日志

Each `session.jsonl` line is an event-log envelope:

`session.jsonl` 的每一行都是一个事件日志信封：

```json
{"schema_version":1,"id":"…","stream":{"kind":"session","id":"<session-id>"},
 "sequence":42,"recorded_at":1771088000123456,"record_type":"event",
 "durability":"durable","causation_id":null,"payload_type":"runtime.session",
 "payload_schema_version":1,"payload":{"kind":"run","run_id":"…",
 "event":{"kind":"assistant_message_committed","text":"…"}}}
```

The useful `payload.event.kind` values for context recovery:

对上下文恢复有用的 `payload.event.kind` 取值：

- `started` — what the user asked: every submitted prompt is recorded here
  (text in `payload.event.prompt`). A `user_prompt_display` record follows
  only when a differing user-facing form exists (an attachment placeholder
  such as `[Image 1]`, a composer form that differs from the sent text) —
  prefer its text when quoting the user. A prompt typed while a run was
  active is instead an
  `inbox_item_queued` whose `source.source` is `"user_steer"` (text in
  `payload.event.payload.prompt`, or in `payload.event.body` when the record
  carries no payload); background and scheduled runtime deliveries share
  that kind, so never quote those as the user.
  `started` — 用户问了什么：每个提交的提示词都记录在此（文本在 `payload.event.prompt`）。只有当存在不同的面向用户的形态时（如 `[Image 1]` 这样的附件占位符、与发送文本不同的输入框表单），才会跟随一条 `user_prompt_display` 记录——引用用户的话时优先用它的文本。运行进行中键入的提示词则是 `inbox_item_queued`，其 `source.source` 为 `"user_steer"`（文本在 `payload.event.payload.prompt`，若记录不带 payload 则在 `payload.event.body`）；后台与定时运行时投递也共用该类型，因此绝不要把它们当作用户的话来引用。
- `assistant_message_committed` — what the agent concluded (decisions,
  summaries, handoffs usually live here, late in the log).
  `assistant_message_committed` — 代理得出了什么结论（决策、总结、交接通常在这里，位于日志后部）。
- `assistant_tool_calls_committed` / `tool_result_batch_committed` — what was
  actually done and what it returned.
  `assistant_tool_calls_committed` / `tool_result_batch_committed` — 实际做了什么以及返回了什么。
- `terminal` — turn boundaries.
  `terminal` — 轮次边界。

Read discipline for long logs: read the TAIL first (later records matter
most), then only enough earlier evidence to understand context. Use bounded
`tail`/`grep` slices; never load a whole multi-megabyte log into context.
Subagent findings live in `subagent/<id>/session.jsonl`, not the main log.
For product debugging of the current session, prefer the doctor skill's
session-evidence helper.

长日志的读取纪律：先读尾部（TAIL，越靠后的记录越重要），再只读理解上下文所需的尽量少的早期证据。使用有界的 `tail`/`grep` 切片；绝不要把整份数 MB 的日志载入上下文。子代理的发现位于 `subagent/<id>/session.jsonl`，不在主日志中。对当前会话做产品调试时，优先使用 doctor 技能的会话证据辅助工具。

## Restoring Lost Work / 恢复丢失的工作

When the user asks to restore or recover lost, wiped, or overwritten work
and a prior session's log holds the only copy of that content (as recorded
tool-call mutations), reconstruct each requested file exactly; do not guess
between versions:

当用户要求恢复丢失、被清除或被覆盖的工作，而既有会话日志持有该内容的唯一副本（以记录下来的工具调用变更形式）时，精确重建每个被要求的文件；不要在版本之间猜测：

1. Enumerate every mutated workspace path first — `write_file`,
   `edit_file`, `apply_patch`, and shell writes — and build the complete
   file list before restoring anything. Restore the full scope of the
   ask: for a general "restore what was lost" ask that means every lost
   file, while an explicitly narrower ask wins as stated.
   先枚举每个被变更的工作区路径——`write_file`、`edit_file`、`apply_patch` 与 shell 写入——并在恢复任何东西之前建立完整的文件清单。按请求的完整范围恢复：对于笼统的 "恢复丢失的东西"，意味着恢复每一个丢失的文件；而明确收窄的请求则按其表述执行。
2. A file's final state is its mutation history replayed in record order
   (`sequence`/`recorded_at`): the LAST `write_file` content for the path
   in record order — or the newest full-file `apply_patch`/shell write
   when that came later — with every LATER `edit_file` delta applied in
   the same order (patch hunks and shell edits likewise). Skip an
   `edit_file`/`apply_patch` delta whose tool result reported an error;
   an errored shell command may still have mutated the file first, so
   check whether later records reflect its write. Recency comes from record
   order, never by content length or size: the longest version is often a
   superseded draft, and refactors make the final version SHORTER.
   Reconstructions built from `write_file` records alone silently drop
   every later edit.
   文件的最终状态是其变更历史按记录顺序（`sequence`/`recorded_at`）重放的结果：按记录顺序该路径最后一次（LAST）`write_file` 的内容——或时间上更晚的整文件 `apply_patch`/shell 写入——再按相同顺序应用其后的每个 `edit_file` 增量（patch 块与 shell 编辑同理）。跳过工具结果报告了错误的 `edit_file`/`apply_patch` 增量；出错的 shell 命令可能仍已先行修改了文件，因此要检查后续记录是否反映了它的写入。新近性来自记录顺序，绝不要以内容长度或大小判断：最长的版本往往是被取代的草稿，而重构会让最终版本更短。仅凭 `write_file` 记录重建会静默丢弃其后所有编辑。
   【评论】"新近性看记录顺序而非内容长度"针对的是模型常按篇幅推断版本新旧这类启发式偏误。
3. Restore the exact recorded content, not a paraphrase. Re-typing from
   memory, rewording, "improving", or summarizing content that is
   recoverable verbatim is data loss, not a restore — extract the bytes
   from the record and write those. (A user asking only to summarize a
   prior session is not a restore; this rule governs restoring files.)
   恢复确切记录的内容，而不是转述。对可逐字恢复的内容凭记忆重打、改写、"改进" 或总结，是数据损失，不是恢复——从记录中提取字节并写入这些字节。（用户只要求总结某个既有会话不属于恢复；本规则管辖的是恢复文件。）
4. Report the result per file: which records it was restored from (the
   last full write plus the deltas applied), and what was restored and
   what was not, with a reason for anything skipped.
   按文件报告结果：它从哪些记录恢复（最后一次完整写入加上所应用的增量）、恢复了什么、没有恢复什么，以及任何跳过项的原因。
5. If two candidate final versions are genuinely ambiguous (e.g.
   divergent edit branches after a fork or resume), present both
   candidates with their record timestamps and let the user pick; never
   silently prefer the longer one.
   如果两个候选最终版本确实存在歧义（例如 fork 或 resume 之后分叉的编辑分支），把两个候选连同其记录时间戳一并列出，让用户选择；绝不静默偏向较长的那一个。

## First-Party Commands / 一方命令

Prefer these over hand-rolled parsing when they fit:

适用时优先使用这些命令，而不是手工解析：

- `muse resume <session-id>` or `muse resume --last` — interactive
  continuation.
  `muse resume <session-id>` 或 `muse resume --last` — 交互式继续。
- `muse exec --session-id <session-id> "<follow-up>"` — headless
  continuation, only on explicit request.
  `muse exec --session-id <session-id> "<follow-up>"` — 无头继续，仅在明确要求时使用。
- `muse export --session <id-or-session.jsonl> --redacted --out <file>` —
  a shareable, redacted export.
  `muse export --session <id-or-session.jsonl> --redacted --out <file>` — 可分享的、已脱敏的导出。
- `muse trace inspect --session-log <session.jsonl> --render-mode compact` —
  model-call level inspection.
  `muse trace inspect --session-log <session.jsonl> --render-mode compact` — 模型调用级别的检查。

Treat session logs as read-only evidence: never modify, move, or delete them.

把会话日志视为只读证据：绝不修改、移动或删除它们。
