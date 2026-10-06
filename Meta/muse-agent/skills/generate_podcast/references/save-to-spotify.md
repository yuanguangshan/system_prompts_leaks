<!-- BILINGUAL-EN-ZH -->
# Add a generated episode to the user's Spotify / 将生成的单集添加到用户的 Spotify

Reference for the `generate_podcast` skill's "Add to your Spotify" handoff. This is a **personal**
save — an episode the user generated into their **own** Spotify library/show — via the bundled
`save-to-spotify` CLI. It is not RSS feed publishing. Only do this when the user explicitly asks;
never offer it proactively.

`generate_podcast` 技能"添加到你的 Spotify"交接环节的参考文档。这是一次**个人**保存——即用户把生成的单集存入其**自己的** Spotify 资料库/节目——通过内置的 `save-to-spotify` CLI 完成。它不是 RSS 订阅源发布。仅在用户明确要求时执行；绝不主动提议。

【评论】"绝不主动提议"这一限制将此类写入用户外部账户的操作严格限定为用户发起，属于能力默认收敛的设计。

## Prefer `podcast-helper save-to-spotify` for the upload / 上传优先使用 `podcast-helper save-to-spotify`

Drive the actual save through `podcast-helper save-to-spotify --slug <slug> [--show-id <id> |
--new-show "<title>"] [--title ...] [--summary ...] [--image ...]`. The helper uploads via the
`save-to-spotify` CLI below **and** records the resulting `spotify_show_url`/`spotify_show_id` into the
manifest feed (and the Postgres catalog), so the show link surfaces on the podcasts library page. It
returns `episode_id`, `episode_uri`, `spotify_show_id`, and `spotify_show_url`. Use the raw CLI
directly for read/status and deletion steps. The rest of this doc describes that underlying CLI.

实际的保存通过 `podcast-helper save-to-spotify --slug <slug> [--show-id <id> |
--new-show "<title>"] [--title ...] [--summary ...] [--image ...]` 驱动。该辅助工具会通过下文的 `save-to-spotify` CLI 上传，**并且**把生成的 `spotify_show_url`/`spotify_show_id` 记录进 manifest feed（以及 Postgres 目录），使节目链接能显示在播客资料库页面上。它返回 `episode_id`、`episode_uri`、`spotify_show_id` 和 `spotify_show_url`。读取/状态与删除步骤直接使用原始 CLI。本文其余部分描述的正是这个底层 CLI。

## Tooling / 工具

Always call the bundled wrapper on PATH; never install the Spotify CLI, download binaries, or set
alternate paths. If `save-to-spotify` is missing, report it as a Muse installation issue. Use
`--json` on every call so results are machine-readable; it is a global flag — place it before the
subcommand (`save-to-spotify --json <command>`).

始终调用 PATH 上自带的包装器；绝不安装 Spotify CLI、下载二进制文件或设置替代路径。若 `save-to-spotify` 缺失，将其报告为 Muse 安装问题。每次调用都使用 `--json` 以便结果可被机器读取；它是全局标志——放在子命令之前（`save-to-spotify --json <command>`）。

```sh
save-to-spotify --json <command> [options]
```

Commands / 命令：

- `save-to-spotify --json shows`
- `save-to-spotify --json shows create --title "<title>" --summary "<desc>"`
- `save-to-spotify --json shows get <show-id>`
- `save-to-spotify --json shows delete <show-id>`
- Uploads are supported only through `podcast-helper save-to-spotify`; the raw wrapper rejects them
  unless the helper has already claimed and fsynced its durable upload receipt.
  上传仅支持通过 `podcast-helper save-to-spotify` 进行；除非辅助工具已认领并 fsync 其持久化上传回执，原始包装器会拒绝上传。
- `save-to-spotify --json episodes --show-id <show-id>`
- `save-to-spotify --json episodes status <episode-id> --wait`
- `save-to-spotify --json episodes delete <episode-id> --show-id <show-id>`
- `save-to-spotify --json timeline set --episode-id <id> --from-file <timeline.json>`

`update` and `token` are disabled by the wrapper; do not use them.

`update` 与 `token` 已被包装器禁用；不要使用。

## Auth (shared Spotify connection — connect in Settings) / 认证（共享 Spotify 连接——在设置中连接）

The tool reuses the user's existing **Spotify** connection, following the same backend as the `spotify`
playback skill. Gatekeeper-enabled users use authd-owned public PKCE; the existing brokered path remains the
rollback outside the rollout. Connect and disconnect happen in  
**Settings → Connections → Spotify** — this tool issues no connect link, holds no token, and has no
`auth` subcommands. Infer connection state from the commands themselves: if `shows` (or any command)
fails with a "connect Spotify in Settings" / token error, tell the user to **connect Spotify in
Settings → Connections → Spotify**, then retry. Never ask the user for Spotify passwords, client
secrets, or tokens.

该工具复用用户已有的 **Spotify** 连接，后端与 `spotify` 播放技能相同。启用了 Gatekeeper 的用户使用 authd 持有的公共 PKCE；既有 broker 路径在推广期外仍作为回退方案。连接与断开都在  
**Settings → Connections → Spotify** 中进行——本工具不发放连接链接、不持有令牌，也没有 `auth` 子命令。连接状态应从命令本身推断：若 `shows`（或任何命令）因"connect Spotify in Settings"/令牌错误而失败，告知用户在 **Settings → Connections → Spotify 中连接 Spotify**，然后重试。绝不向用户索要 Spotify 密码、客户端密钥或令牌。

## Upload flow / 上传流程

1. `shows` — list existing shows (this also confirms Spotify is connected; on a token error, point the
   user to Settings → Connections → Spotify). Ask whether to reuse one or create a new one
   (`shows create` or `upload --new-show`). Don't silently pick.
   `shows`——列出现有节目（这一步同时确认 Spotify 已连接；出现令牌错误时，引导用户前往 Settings → Connections → Spotify）。询问是复用已有节目还是新建（`shows create` 或 `upload --new-show`）。绝不擅自替用户选择。
2. Run `podcast-helper save-to-spotify ...` — confirm the title, show, and summary with the user first
   (this is an approved write to their account). The helper claims a durable receipt before launch; a
   deterministic wrapper preflight rejection is safe to retry only when the wrapper positively reports that
   no external attempt began. Every other failure leaves the receipt uncertain and blocks a blind retry.
   运行 `podcast-helper save-to-spotify ...`——先与用户确认标题、节目和摘要（这是对其账户的一次写入，须获批准）。辅助工具在启动前先认领一份持久化回执；只有当包装器明确报告外部尝试尚未开始时，确定性的包装器预检拒绝才可安全重试。其他任何失败都会使回执状态不确定，并禁止盲目重试。
3. `episodes status <episode-id> --wait` — poll until `READY`. Processing is server-side and can take
   a few minutes.
   `episodes status <episode-id> --wait`——轮询直到 `READY`。处理在服务端进行，可能需要几分钟。
4. Optionally `timeline set --episode-id <id> --from-file timeline.json` once READY.
   可选：READY 后执行 `timeline set --episode-id <id> --from-file timeline.json`。
5. Tell the user it's on their Spotify and may take a few minutes to appear in the app. Use the show
   and episode titles plus plain-language readiness (`ready` or `still processing`); do not quote the
   CLI's raw identifiers or JSON.
   告知用户内容已在其 Spotify 上，可能需要几分钟才会出现在应用中。使用节目和单集标题加上通俗的就绪表述（`ready` 或 `still processing`）；不要引用 CLI 的原始标识符或 JSON。

## Deletion flow / 删除流程

Shows and episodes created through Save to Spotify are deleted with this CLI. They are not managed in
`podcasters.spotify.com` or `creators.spotify.com`, and `spotify-api` cannot delete them.

通过 Save to Spotify 创建的节目和单集用本 CLI 删除。它们不受 `podcasters.spotify.com` 或 `creators.spotify.com` 管理，`spotify-api` 也无法删除它们。

1. Run `save-to-spotify --json shows`. Treat this as the authoritative inventory of the user's Save to
   Spotify shows. Match by title; if multiple shows match, ask the user which one they mean.
   运行 `save-to-spotify --json shows`。将其视为用户 Save to Spotify 节目的权威清单。按标题匹配；若有多个节目匹配，询问用户指的是哪一个。
2. Run `save-to-spotify --json shows get <show-id>` to verify the selected title and episode count.
   运行 `save-to-spotify --json shows get <show-id>` 以核实所选标题和单集数量。
3. For one episode, run `save-to-spotify --json episodes --show-id <show-id>`, match the requested title,
   confirm the deletion, then run  
   `save-to-spotify --json episodes delete <episode-id> --show-id <show-id>`.
   删除单个单集：运行 `save-to-spotify --json episodes --show-id <show-id>`，匹配所请求的标题，确认删除，然后运行  
   `save-to-spotify --json episodes delete <episode-id> --show-id <show-id>`。
4. For a whole show, confirm the show title and that all its episodes will be removed, then run
   `save-to-spotify --json shows delete <show-id>`. The show command deletes its episodes too; do not
   delete each episode first.
   删除整档节目：确认节目标题及其所有单集将被移除，然后运行 `save-to-spotify --json shows delete <show-id>`。节目命令会一并删除其单集；不要先逐个删除单集。
5. Treat `{"status":"deleted"}` as acceptance, not proof that Spotify's listings have converged. Poll
   every 5–10 seconds for up to 60 seconds: query `episodes --show-id <show-id>` after an episode deletion
   or `shows` after a show deletion. The deletion is confirmed only when the deleted ID is absent.
   将 `{"status":"deleted"}` 视为已受理，而非 Spotify 列表已收敛的证明。每 5–10 秒轮询一次、最长 60 秒：删除单集后查询 `episodes --show-id <show-id>`，删除节目后查询 `shows`。只有被删 ID 不再出现时，删除才算确认。
6. If the item is still listed after 60 seconds, tell the user the deletion was accepted and can take
   about a minute to propagate. Do not issue the delete again. Otherwise report the deleted title, not
   the internal ID. Clarify that this removes the Spotify copy only; it does not delete Muse's local
   audio or an independently published RSS feed.
   若 60 秒后条目仍在列表中，告知用户删除已被受理、传播约需一分钟。不要再次发起删除。否则报告被删的标题，而非内部 ID。说明这只移除 Spotify 上的副本；不会删除 Muse 的本地音频或独立发布的 RSS 订阅源。

Supported audio: `.mp3`, `.m4a`, `.wav`, `.ogg`. Cover images: **JPEG or PNG only, ≤ 1 MB** — other
formats (e.g. `.webp`) are rejected with `unsupported image extension`; convert first
(`ffmpeg -i cover.webp cover.jpg`) and pass the converted file to `--image`.

支持的音频格式：`.mp3`、`.m4a`、`.wav`、`.ogg`。封面图片：**仅限 JPEG 或 PNG，≤ 1 MB**——其他格式（如 `.webp`）会被以 `unsupported image extension` 拒绝；先转换（`ffmpeg -i cover.webp cover.jpg`），再把转换后的文件传给 `--image`。

**Readiness & retries.** A returned `episode_uri` means Spotify *accepted* the upload; reaching `READY`
is a separate server-side processing step. Spotify occasionally returns `503` or leaves an episode in
`NOT_READY` during backend slowdowns. If it hasn't reached `READY` after several minutes, that's a
Spotify-side stall (not a Muse error): tell the user it was accepted but is still processing on
Spotify's side, and retry the upload later rather than polling indefinitely.

**就绪与重试。** 返回 `episode_uri` 意味着 Spotify *已接受*上传；达到 `READY` 是另一个独立的服务端处理步骤。在后端变慢期间，Spotify 偶尔会返回 `503` 或让单集停留在 `NOT_READY`。若几分钟后仍未达到 `READY`，那是 Spotify 侧的停滞（不是 Muse 的错误）：告知用户上传已被接受、仍在 Spotify 侧处理，稍后重试上传，而不是无限轮询。

## Rules / 规则

1. Always use `save-to-spotify --json`; never install or invoke the Spotify CLI directly.
   始终使用 `save-to-spotify --json`；绝不安装或直接调用 Spotify CLI。
2. Connect is Settings-only: on a token / "not connected" error, tell the user to connect Spotify in
   Settings → Connections → Spotify, then retry — this tool has no `auth` subcommands or connect link.
   连接只能在设置中完成：出现令牌/"not connected"错误时，告知用户在 Settings → Connections → Spotify 中连接 Spotify，然后重试——本工具没有 `auth` 子命令或连接链接。
3. Confirm title, target show, and summary before `upload` — it writes to the user's account.
   `upload` 前确认标题、目标节目和摘要——它会写入用户的账户。
4. After `upload`, poll `episodes status --wait` until `READY` before setting a timeline or telling the
   user it's live.
   `upload` 之后，先轮询 `episodes status --wait` 直到 `READY`，再设置时间线或告知用户已上线。
5. Only save audio the user generated or provided; respect third-party rights.
   只保存用户生成或提供的音频；尊重第三方权利。
6. Do not use `update` or `token` (disabled).
   不要使用 `update` 或 `token`（已禁用）。
7. Covers must be JPEG/PNG ≤ 1 MB (convert `.webp` or others first).
   封面必须是 ≤ 1 MB 的 JPEG/PNG（先转换 `.webp` 或其他格式）。
8. A returned `episode_uri` = accepted; a `NOT_READY`/`503` that never reaches `READY` is a Spotify-side
   stall — surface it and retry later, don't loop indefinitely.
   返回 `episode_uri` 即已受理；始终未达到 `READY` 的 `NOT_READY`/`503` 属于 Spotify 侧停滞——如实告知并稍后重试，不要无限循环。
9. **Never expose internal Spotify identifiers.** Show IDs, episode IDs, and `spotify:show:` /
   `spotify:episode:` URIs are opaque handles used only as arguments to later CLI commands. Never print,
   echo, or mention them in confirmations, upload summaries, readiness updates, or error explanations.
   Refer to shows and episodes by title instead.
   **绝不暴露内部 Spotify 标识符。**节目 ID、单集 ID 以及 `spotify:show:` / `spotify:episode:` URI 是不透明句柄，仅用作后续 CLI 命令的参数。绝不在确认信息、上传摘要、就绪更新或错误解释中打印、回显或提及它们。改用标题指称节目和单集。
10. Confirm every deletion. A show deletion removes all of that show's episodes.
    每次删除都须确认。删除节目会移除该节目的所有单集。
11. Never direct the user to Spotify creator websites for this content; use the CLI inventory and delete
    commands above.
    就此类内容绝不引导用户前往 Spotify 创作者网站；使用上文的 CLI 清单和删除命令。
12. Never claim an item disappeared based only on `{"status":"deleted"}`. Verify absence with a bounded
    read loop, or clearly say the accepted deletion is still propagating after the one-minute timeout.
    绝不仅凭 `{"status":"deleted"}` 就声称条目已消失。用有界读取循环核实其不再出现，或在超时一分钟后明确说明已受理的删除仍在传播。
