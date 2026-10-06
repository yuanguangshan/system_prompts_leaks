---
name: "peloton"
description: "Connect to Peloton to browse fitness classes, check schedules, and book workouts."
icon: "peloton"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Peloton / Peloton

## Purpose / 用途
Connect to Peloton to browse on-demand and live fitness classes, manage schedules, and book workouts.

连接 Peloton，浏览点播与直播健身课程、管理日程并预约训练。

## Tooling / 工具
Use the installed CLI directly from `PATH`.

直接使用 `PATH` 中已安装的 CLI。

Core auth commands:

核心认证命令：

- `peloton status`
- `peloton authorize-url`
- `peloton disconnect`

### Class Browsing / 课程浏览
- `peloton ride-archived [--limit 10] [--page 0] [--fitness-discipline cycling] [--duration 1800] [--sort-by popularity] [--difficulty-level beginner] [--has-closed-captions true] [--super-genre-id <ID>] [--browse-category cycling] [--class-type-id <ID>] [--instructor <ID>] [--desc true]`
- `peloton ride-live [--limit 10] [--schedule-type live] [--exclude-complete true] [--days 7] [--fitness-discipline cycling] [--super-genre-id <ID>] [--browse-category cycling] [--class-type-id <ID>] [--instructor <ID>] [--desc true]`
- `peloton search-class --query "<text>" --limit 10 --page 0 [--include-raw] [--include-schema]` — search classes by natural-language or partner-provided terms such as "seated strength". Always pass `--limit` and `--page`; use `--limit 10 --page 0` unless the user asks for a different local result window. Use `--include-raw` or `--include-schema` only when inspecting the response shape.
  `peloton search-class --query "<text>" --limit 10 --page 0 [--include-raw] [--include-schema]` —— 通过自然语言或合作方提供的词语（如 "seated strength"）搜索课程。务必传入 `--limit` 和 `--page`；除非用户要求不同的本地结果窗口，否则使用 `--limit 10 --page 0`。仅在对响应结构进行检查时才使用 `--include-raw` 或 `--include-schema`。

Response structure: class browsing commands return a presentation-ready `body.classes[]` array. `search-class` also returns local-window fields: `body.count`, `body.total`, `body.page`, `body.limit`, `body.has_more`, and `body.pagination_mode: "local_window"`. Peloton search does not support API pagination; `--limit` and `--page` only slice the returned result set locally. For different results, refine the query instead of trying to fetch another provider page. Raw provider arrays may be present only for debug commands that request them.

响应结构：课程浏览命令返回可直接呈现的 `body.classes[]` 数组。`search-class` 还会返回本地窗口字段：`body.count`、`body.total`、`body.page`、`body.limit`、`body.has_more` 和 `body.pagination_mode: "local_window"`。Peloton 搜索不支持 API 分页；`--limit` 和 `--page` 只在本地对返回的结果集进行切片。要获得不同的结果，请优化查询本身，而不是试图获取提供商的下一页。原始提供商数组仅可能出现在请求它们的调试命令中。

Class reads preserve Peloton's raw epoch fields and add semantic
`class_scheduled_start_at`, `class_starts_at`, `class_ends_at`, and
`class_originally_aired_at` values with UTC and user-local forms when those
source fields are present. Prefer the semantic fields.

课程读取保留 Peloton 的原始 epoch 字段，并在这些源字段存在时，追加带 UTC 和用户本地时间形式的语义化 `class_scheduled_start_at`、`class_starts_at`、`class_ends_at` 与 `class_originally_aired_at` 值。优先使用语义化字段。

#### Filter IDs / 过滤器 ID
- `peloton metadata-mappings` — returns all filter taxonomies: instructors, class_types, equipment, fitness_disciplines, difficulty_levels, and content_formats. Use the returned IDs for `--super-genre-id`, `--class-type-id`, and `--instructor` filters instead of hardcoding values.
  `peloton metadata-mappings` —— 返回所有过滤器的分类体系：instructors、class_types、equipment、fitness_disciplines、difficulty_levels 和 content_formats。在 `--super-genre-id`、`--class-type-id` 和 `--instructor` 过滤器中使用返回的 ID，不要硬编码取值。

### Scheduling / 日程安排
- `peloton schedule-event --ride-id <ID> --scheduled-start-time <EPOCH>` — schedule an on-demand class
  —— 安排一节点播课程
- `peloton schedule-event --join-token <TOKEN>` — schedule a live class
  —— 安排一节直播课程
- `peloton delete-scheduled-event --join-token <TOKEN>` — remove a scheduled class
  —— 移除一个已安排的课程
- `peloton reschedule-event --join-token <TOKEN> --scheduled-start-time <EPOCH>` — move to new time
  —— 改期到新时间

### Deeplinks / 深度链接
- `peloton deeplink --class-id <ID>` — generate a link to view the class in Peloton
  —— 生成一个在 Peloton 中查看该课程的链接

## Auth / 认证
Peloton is an OAuth-backed skill.

Peloton 是一个基于 OAuth 的技能。

For normal class reads, call the requested read command directly; do not add a `peloton status` preflight. If a command reports `not_connected`, or the user asks to manage the connection:

正常的课程读取直接调用所请求的读取命令；不要附加 `peloton status` 预检。若某命令报告 `not_connected`，或用户要求管理连接：

1. Run `peloton status`.
   运行 `peloton status`。
2. If status is `not_connected`, complete the link flow first.
   若状态为 `not_connected`，先完成连接流程。
3. Before running `peloton authorize-url`, ask for explicit confirmation:
   在运行 `peloton authorize-url` 之前，先请求明确确认：
   - state the provider name: `peloton`
     说明提供商名称：`peloton`
   - warn: `This stores a persistent access token for Peloton. Your assistant will have ongoing access until you disconnect it locally or revoke it in Peloton settings.`
     警告：`This stores a persistent access token for Peloton. Your assistant will have ongoing access until you disconnect it locally or revoke it in Peloton settings.`
   - never show raw OAuth scope strings or scope names in the connection message
     绝不在连接消息中显示原始的 OAuth scope 字符串或 scope 名称
4. Run `peloton authorize-url` once. When `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Peloton](<connect_url>)`; do not paste the raw URL separately.
   只运行一次 `peloton authorize-url`。当返回 `connect_url` 时，用返回的 URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Peloton](<connect_url>)`；不要单独粘贴原始 URL。
5. Do not call `peloton authorize-url` again while waiting for the user to approve the link or for the callback to complete; use `peloton status` to check pending connection state.
   在等待用户批准链接或回调完成期间，不要再次调用 `peloton authorize-url`；用 `peloton status` 检查连接的待定状态。
6. After callback completion, run `peloton status` again and continue only when status is `connected`. If status is still `not_connected`, say the connection is still pending or failed and ask whether to generate a new link.
   回调完成后，再次运行 `peloton status`，仅当状态为 `connected` 时才继续。若状态仍为 `not_connected`，告知用户连接仍在等待或已失败，并询问是否生成新链接。

【评论】认证流程要求在生成授权链接前向用户披露"持久访问令牌"的后果、隐藏 OAuth scope 细节，并禁止重复生成链接，属于对第三方连接权限的谨慎披露与防滥用设计。

Credential storage:

凭据存储：

- Never print `client_secret`, `access_token`, or `refresh_token`.
  绝不打印 `client_secret`、`access_token` 或 `refresh_token`。

## Operating Rules / 运行规则
1. For account linking, follow the Auth section.
   账号连接遵循 Auth 部分。
2. Do not show raw internal identifiers, difficulty scores, star ratings, rating counts, or class URLs in user-facing results unless the user explicitly asks for technical/debug details. Use titles, instructor names, and times for fallback text.
   除非用户明确要求技术/调试细节，否则不要在面向用户的结果中显示原始内部标识符、难度分数、星级评分、评分数量或课程 URL。回退文本使用课程标题、教练姓名和时间。
3. For natural-language class searches, call `peloton search-class` with `--limit 10 --page 0`. Use `ride-archived` only when the user requests its structured filters or unsearched archived browsing. Present the returned `body.classes[]` with `widget.create`, using `kind: "list"` and a list title and `items` inside `data`. Put each class title in `title`, instructor and `duration_minutes` in `subtitle`, and a relevant class time in `tertiary_title` when available. Copy `thumbnail_url` into `image_url` when present. Use `type: "link"` with `data.url` copied from the returned HTTP(S) `deeplink_url`; use `type: "generic"` when no such link is returned. Place the returned `embed_token` in the reply. Do not duplicate the class list in plain text. If there are no classes, say so; if widget creation fails, use concise Markdown bullets with title, instructor, and duration. Do not print class IDs or image CDN URLs as text.
   对于自然语言课程搜索，用 `--limit 10 --page 0` 调用 `peloton search-class`。仅当用户请求其结构化过滤器或未检索的点播浏览时才使用 `ride-archived`。用 `widget.create` 呈现返回的 `body.classes[]`，采用 `kind: "list"`，并在 `data` 内给出列表标题和 `items`。每个课程标题放入 `title`，教练与 `duration_minutes` 放入 `subtitle`，可用时把相关课程时间放入 `tertiary_title`。`thumbnail_url` 存在时复制到 `image_url`。使用 `type: "link"`，其 `data.url` 从返回的 HTTP(S) `deeplink_url` 复制；没有此类链接时使用 `type: "generic"`。把返回的 `embed_token` 放入回复中。不要在纯文本中重复课程列表。若没有课程，如实说明；若组件创建失败，改用包含标题、教练和时长的简洁 Markdown 要点。不要把课程 ID 或图片 CDN URL 作为文本打印。
4. For other class browsing commands, use the same list presentation with `body.classes[]`. Use raw provider arrays only for requested debugging. Do not construct class URLs yourself.
   其他课程浏览命令同样使用基于 `body.classes[]` 的列表呈现。原始提供商数组仅用于用户要求的调试。不要自行构造课程 URL。
5. Use the CLI-provided `duration_minutes` field for displaying class duration.
   显示课程时长时使用 CLI 提供的 `duration_minutes` 字段。
6. For live classes, default to a 7-day window using `--days 7 --exclude-complete true`.
   直播课程默认使用 7 天窗口，即 `--days 7 --exclude-complete true`。
7. Scheduling and deleting scheduled classes may proceed from a clear, unambiguous user request without an additional confirmation.
   只要用户请求清晰、无歧义，安排课程和删除已安排课程即可执行，无需额外确认。
8. Token refresh on expired-token errors is automatic. If API calls still fail after auto-refresh, re-check `peloton status` and re-link if needed.
   令牌过期错误会自动刷新。若自动刷新后 API 调用仍然失败，重新检查 `peloton status` 并在需要时重新连接。
