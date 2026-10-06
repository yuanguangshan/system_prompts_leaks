---
name: "spotify"
description: "Discover, search, and manage Spotify music, podcasts, and playlists, including deleting shows or episodes you created with Save to Spotify."
icon: "spotify"
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Spotify / Spotify

## Purpose / 目的
Use `spotify-api` to browse personalized Spotify content, search for music and podcasts, manage the user's library and playlists, check saveability of items, and start or control Spotify playback. For new playback during a voice conversation, follow **Voice playback** below instead of calling `spotify-api play`. Use `save-to-spotify` to manage shows and episodes you created through Save to Spotify.

使用 `spotify-api` 浏览个性化的 Spotify 内容、搜索音乐和播客、管理用户的资料库与播放列表、检查条目是否可保存，以及启动或控制 Spotify 播放。在语音对话中发起新的播放时，请遵循下方的**语音播放**部分，而不是调用 `spotify-api play`。使用 `save-to-spotify` 管理你通过 Save to Spotify 创建的节目和单集。

## Voice playback / 语音播放

Use this section only when someone asks to start new music during a voice
conversation. For pause, resume, stop, skip, previous, status, volume,
transfer, queue, login, search, library, or playlist requests, use the
`spotify-api` commands below.

仅当有人在语音对话中要求开始播放新音乐时才使用本节。对于暂停、恢复、停止、跳过、上一首、状态、音量、转移、队列、登录、搜索、资料库或播放列表等请求，请使用下方的 `spotify-api` 命令。

### Choose where to play / 选择播放位置

Use the first rule that matches:

使用第一条匹配的规则：

1. If the user names a playback device other than the device carrying this
   call, such as a phone, computer, TV, speaker, car, or console, or refers to
   another device named earlier with words like "there" or "the same speaker,"
   run `spotify-api devices`. Find the exact device, then use
   `spotify_connect` and its `spotify_device_id`. A specifically named phone
   always uses this rule, even if it carries an app-audio call. If there is no
   clear match, ask which device to use. Never use `auto` for that separate
   named or previously referenced device.
   如果用户指定的播放设备不是承载本次通话的设备（例如手机、电脑、电视、音箱、汽车或游戏主机），或者用“那里”“同一个音箱”之类的词指代先前提到的另一台设备，则运行 `spotify-api devices`，找到确切的设备，然后使用 `spotify_connect` 及其 `spotify_device_id`。被明确点名的手机始终适用本条规则，即使它承载的是 app-audio 通话。如果没有明确的匹配，请询问使用哪台设备。对于那台被单独点名或先前提及的设备，绝不要使用 `auto`。
2. Otherwise use `auto` for the device carrying this call. This includes
   "here", "these glasses", "this device", naming that same non-phone calling
   device, "play X", and "play X on Spotify". Spotify is the music service,
   not the device.
   否则，对承载本次通话的设备使用 `auto`。这包括“这里”“这副眼镜”“这台设备”、点名同一台非手机通话设备、“播放 X”以及“在 Spotify 上播放 X”。Spotify 是音乐服务，不是设备。

### Start the music / 开始播放

Call `muse.music` once with `action` set to `play` and the destination chosen
above. Do not call `spotify-api play`, `spotify-api wearable-play`, or
`muse.device.invoke` for this request.

调用一次 `muse.music`，将 `action` 设为 `play`，并使用上文选定的目标。对于此请求，不要调用 `spotify-api play`、`spotify-api wearable-play` 或 `muse.device.invoke`。

For `auto`, trusted host code checks the exact device that started this voice
call. If its live tool list contains `music_fulfillment`, the host calls that
command once on that device. If no supported call device can be resolved, the
host uses the active Spotify Connect device. If a resolved device disappears
or loses the command during dispatch, the host stops instead of switching
devices. Do not inspect or call device tools yourself.

对于 `auto`，受信任的宿主代码会检查发起本次语音通话的确切设备。如果其实时工具列表包含 `music_fulfillment`，宿主会在该设备上调用一次该命令。如果无法解析出受支持的通话设备，宿主会使用活跃的 Spotify Connect 设备。如果已解析的设备在分发过程中消失或失去该命令，宿主会直接停止，而不是切换设备。不要自行检查或调用设备工具。

If `muse.music` fails, report the failure. Do not try another device or
playback route. If Spotify was disconnected, let the tool show the connection
flow. When the user connects and asks again, call `muse.music` again.

如果 `muse.music` 失败，请报告该失败。不要尝试其他设备或播放路径。如果 Spotify 已断开连接，让工具展示连接流程。当用户完成连接并再次请求时，再次调用 `muse.music`。

## Tooling / 工具
Use the installed CLI directly from `PATH`.

直接使用 `PATH` 中已安装的 CLI。

#### Connection / 连接
- `spotify-api status` — check OAuth connection status (`status`, `connect_url`, `disconnect_url`)
  `spotify-api status` — 检查 OAuth 连接状态（`status`、`connect_url`、`disconnect_url`）
- `spotify-api disconnect` — disconnect Spotify (may return a confirmation URL)
  `spotify-api disconnect` — 断开 Spotify 连接（可能返回一个确认 URL）
- `spotify-api authorize-url` — get the active environment's connect URL
  `spotify-api authorize-url` — 获取当前活跃环境的连接 URL

#### Browse & Discover / 浏览与发现
- `spotify-api experience --id <spotify_uri_or_name> [--language <lang>]` — get experience by ID (e.g. artist/album/show page). Large sections may include a `next` URL for more results.
  `spotify-api experience --id <spotify_uri_or_name> [--language <lang>]` — 按 ID 获取体验页（例如艺人/专辑/节目页面）。较大的分节可能包含用于获取更多结果的 `next` URL。
- `spotify-api next-page --url <section_next_url> [--language <lang>]` — fetch the next pagination URL returned as `sections[].next` or `next` by a prior Spotify response. Prefer this over guessing section IDs.
  `spotify-api next-page --url <section_next_url> [--language <lang>]` — 获取先前 Spotify 响应以 `sections[].next` 或 `next` 形式返回的下一个分页 URL。优先使用此方式，而不是猜测分节 ID。

#### Search / 搜索
- `spotify-api search --query <text> [--search-type TRACKS,ALBUMS,ARTISTS,PLAYLISTS,EPISODES,PODCASTS] [--language <lang>]` — search for content. For an exact song or album lookup, always format the query as `"<song or album> by <artist>"` so missing originals are distinguished from covers and similarly named content. Use `PODCASTS` to find shows (not `SHOWS` which is invalid). Use `experience --id <show_uri>` to list episodes of a found show.
  `spotify-api search --query <text> [--search-type TRACKS,ALBUMS,ARTISTS,PLAYLISTS,EPISODES,PODCASTS] [--language <lang>]` — 搜索内容。要精确查找某首歌曲或某张专辑时，务必将查询格式化为 `"<song or album> by <artist>"`，以便将缺失的原版与翻唱及名称相近的内容区分开。查找节目请使用 `PODCASTS`（`SHOWS` 是无效值）。使用 `experience --id <show_uri>` 列出找到的节目的单集。
- Search responses include `spotify_search_url` for opening the same query directly in Spotify. When an exact requested item is unavailable, `catalog_fallback.message` is the fully rendered approved response and must be relayed verbatim; the object also provides `requested_content_label`, `requested_content_url`, `artist_name`, and `artist_url`.
  搜索响应包含 `spotify_search_url`，可直接在 Spotify 中打开同一查询。当请求的确切条目不可用时，`catalog_fallback.message` 是已完整渲染的官方批准回复，必须逐字转达；该对象还提供 `requested_content_label`、`requested_content_url`、`artist_name` 和 `artist_url`。

#### Filter values / 过滤值
- Valid library/browse filter values are `ALBUMS`, `ARTISTS`, `PLAYLISTS`, `EPISODES`, `PODCASTS`, `SHOWS`, and `PODCASTS_AND_SHOWS`.
  资料库/浏览的有效过滤值为 `ALBUMS`、`ARTISTS`、`PLAYLISTS`、`EPISODES`、`PODCASTS`、`SHOWS` 和 `PODCASTS_AND_SHOWS`。
- These filter values are content categories, not field projections. Never use field names such as `title`, `items.title`, `sections`, or `items` as `--filter` values.
  这些过滤值是内容类别，不是字段投影。绝不要将 `title`、`items.title`、`sections` 或 `items` 等字段名用作 `--filter` 的值。
- `TRACKS` is explicitly rejected for library filtering; use unfiltered `spotify-api library` or `spotify-api search --search-type TRACKS` to find tracks.
  资料库过滤明确拒绝 `TRACKS`；查找曲目请使用未过滤的 `spotify-api library` 或 `spotify-api search --search-type TRACKS`。

#### Library / 资料库
- `spotify-api library [--filter ALBUMS|ARTISTS|PLAYLISTS|EPISODES|PODCASTS|SHOWS|PODCASTS_AND_SHOWS] [--language <lang>]` — browse user's library. `TRACKS` is not supported as a library filter; use unfiltered `library` or `search --search-type TRACKS` instead.
  `spotify-api library [--filter ALBUMS|ARTISTS|PLAYLISTS|EPISODES|PODCASTS|SHOWS|PODCASTS_AND_SHOWS] [--language <lang>]` — 浏览用户资料库。资料库过滤不支持 `TRACKS`；请改用未过滤的 `library` 或 `search --search-type TRACKS`。
- `spotify-api save --uri <spotify_uri>` — save item to library (track, album, artist, show, episode, playlist)
  `spotify-api save --uri <spotify_uri>` — 将条目保存到资料库（曲目、专辑、艺人、节目、单集、播放列表）

#### Delete Save to Spotify shows or episodes / 删除 Save to Spotify 创建的节目或单集
- When the user asks to delete a podcast, show, or episode Muse added through Save to Spotify, use the installed `save-to-spotify` CLI. These are managed by the CLI, not `spotify-api`, `podcasters.spotify.com`, or `creators.spotify.com`.
  当用户要求删除 Muse 通过 Save to Spotify 添加的播客、节目或单集时，使用已安装的 `save-to-spotify` CLI。这些内容由该 CLI 管理，而非 `spotify-api`、`podcasters.spotify.com` 或 `creators.spotify.com`。
- If `save-to-spotify` reports that show and episode management is unavailable, explain the limitation directly; do not ask the user to reconnect or retry.
  如果 `save-to-spotify` 报告节目和单集管理不可用，直接说明该限制；不要要求用户重新连接或重试。
- Always use JSON mode. Start with `save-to-spotify --json shows`; this inventory contains the shows created through Save to Spotify. Match by title and ask the user to disambiguate if more than one show matches.
  始终使用 JSON 模式。从 `save-to-spotify --json shows` 开始；该清单包含通过 Save to Spotify 创建的节目。按标题匹配，如果多个节目匹配，请用户加以区分。
- Inspect the selected show with `save-to-spotify --json shows get <show-id>`. List its episodes when needed with `save-to-spotify --json episodes --show-id <show-id>` and match episodes by title.
  使用 `save-to-spotify --json shows get <show-id>` 查看选定的节目。需要时用 `save-to-spotify --json episodes --show-id <show-id>` 列出其单集，并按标题匹配单集。
- To delete one episode, resolve its title and owning show unambiguously, then run  
  `save-to-spotify --json episodes delete <episode-id> --show-id <show-id>`.
  要删除单个单集，需无歧义地确定其标题及其所属节目，然后运行
  `save-to-spotify --json episodes delete <episode-id> --show-id <show-id>`。
- To delete the whole show, resolve the show unambiguously and tell the user that all of its episodes will be removed, then run `save-to-spotify --json shows delete <show-id>`. This command deletes the show and its episodes; do not delete each episode first.
  要删除整个节目，需无歧义地确定该节目，并告知用户其所有单集都将被移除，然后运行 `save-to-spotify --json shows delete <show-id>`。该命令会删除节目及其所有单集；不要先逐个删除单集。
- A `{"status":"deleted"}` response means Spotify accepted the deletion, but its listings can take about a minute to update. Poll the relevant inventory every 5–10 seconds for up to 60 seconds: use `episodes --show-id <show-id>` for an episode or `shows` for a show. Confirm completion only after the deleted ID is absent. If it is still listed after 60 seconds, tell the user the deletion was accepted and is still propagating; do not send the delete again.
  收到 `{"status":"deleted"}` 响应表示 Spotify 已接受删除，但其列表更新可能需要约一分钟。以每 5–10 秒一次的频率轮询相关清单，最长 60 秒：单集使用 `episodes --show-id <show-id>`，节目使用 `shows`。只有在被删除的 ID 不再出现时才确认完成。如果 60 秒后仍在列表中，告知用户删除已被接受且仍在传播中；不要再次发送删除命令。
- Show and episode IDs are internal command handles. Refer to content by title in user-facing replies. A CLI deletion removes the Spotify copy only; it does not delete Muse's local audio or an independently published RSS feed.
  节目和单集 ID 是内部命令句柄。在面向用户的回复中以标题指代内容。CLI 删除只移除 Spotify 上的副本；不会删除 Muse 的本地音频或独立发布的 RSS 源。

【评论】删除能力被收敛到专用的 `save-to-spotify` CLI，并要求轮询确认传播完成，这是一种边界设计，避免经由通用 API 误删内容或遗漏独立发布的 RSS 源。

#### Collections (Playlists) / 合集（播放列表）
- `spotify-api create-collection --name <name>` — create a new playlist
  `spotify-api create-collection --name <name>` — 创建新的播放列表
- `spotify-api add-to-collection --collection-uri <uri> --uris <uri1,uri2,...> [--position-type BEFORE_UID|AFTER_UID --position-uid <uid>] [--revision-id <rev>]` — add items to a playlist
  `spotify-api add-to-collection --collection-uri <uri> --uris <uri1,uri2,...> [--position-type BEFORE_UID|AFTER_UID --position-uid <uid>] [--revision-id <rev>]` — 向播放列表添加条目
- `spotify-api update-collection --collection-uri <uri> --name <new_name>` — rename a playlist
  `spotify-api update-collection --collection-uri <uri> --name <new_name>` — 重命名播放列表

#### Playback Control / 播放控制
- `spotify-api play [--context-uri <uri>] [--uid <uid>] [--target-device-id <id>]` — start playback outside a voice conversation (optionally of a specific album/playlist/context, starting from a specific item UID, on a specific device). For new playback during voice, follow **Voice playback** above and use `muse.music` instead.
  `spotify-api play [--context-uri <uri>] [--uid <uid>] [--target-device-id <id>]` — 在语音对话之外开始播放（可选：播放特定的专辑/播放列表/上下文、从特定条目 UID 开始、在特定设备上播放）。语音对话中的新播放请遵循上方的**语音播放**部分并改用 `muse.music`。
- `spotify-api pause` — pause playback on the active device
  `spotify-api pause` — 在活跃设备上暂停播放
- `spotify-api resume` — resume paused playback on the active device
  `spotify-api resume` — 在活跃设备上恢复已暂停的播放
- `spotify-api skip` — skip to the next item
  `spotify-api skip` — 跳到下一个条目
- `spotify-api previous` — go to the previous item
  `spotify-api previous` — 回到上一个条目
- `spotify-api seek --position-ms <ms>` — seek to a position in the currently playing item
  `spotify-api seek --position-ms <ms>` — 在当前播放条目中跳转到指定位置
- `spotify-api set-volume --volume-percent <0-100> [--target-device-id <id>]` — set playback volume
  `spotify-api set-volume --volume-percent <0-100> [--target-device-id <id>]` — 设置播放音量
- `spotify-api transfer --target-device-id <id>` — transfer playback to a different device
  `spotify-api transfer --target-device-id <id>` — 将播放转移到另一台设备
- `spotify-api now-playing` — get the current playback state (track, progress, device, etc.)
  `spotify-api now-playing` — 获取当前播放状态（曲目、进度、设备等）
- `spotify-api devices` — list available Spotify Connect devices
  `spotify-api devices` — 列出可用的 Spotify Connect 设备
- `spotify-api get-queue` — get the current playback queue
  `spotify-api get-queue` — 获取当前播放队列
- `spotify-api add-to-queue --item-uri <spotify_uri>` — add an item to the playback queue
  `spotify-api add-to-queue --item-uri <spotify_uri>` — 向播放队列添加条目

#### Known unavailable or conditional commands / 已知不可用或有条件限制的命令
- `home`, `recommendations`, `check-saved`, `remove`, `section-items`, and `reorder-collection` are not exposed by `spotify-api` because they are unsupported or unreliable with the current Partner API responses/scopes. Use `search`, `library`, `experience`, `next-page`, and playlist create/add/update instead. This does not apply to shows and episodes created through Save to Spotify; delete those with `save-to-spotify` as described above.
  `home`、`recommendations`、`check-saved`、`remove`、`section-items` 和 `reorder-collection` 未由 `spotify-api` 提供，因为当前的 Partner API 响应/权限范围不支持它们或其结果不可靠。请改用 `search`、`library`、`experience`、`next-page` 以及播放列表的创建/添加/更新。该限制不适用于通过 Save to Spotify 创建的节目和单集；如上所述，用 `save-to-spotify` 删除它们。
- `play` and `add-to-queue` require an active Spotify Connect device; use `spotify-api devices` first and prefer an explicit `--target-device-id` where supported. `set-volume` should also prefer `--target-device-id`.
  `play` 和 `add-to-queue` 需要一台活跃的 Spotify Connect 设备；先使用 `spotify-api devices`，并在支持时优先使用显式的 `--target-device-id`。`set-volume` 也应优先使用 `--target-device-id`。


## Auth / 认证
`spotify-api` owns the Spotify connection workflow.

`spotify-api` 负责 Spotify 连接工作流。

Auth contract:

认证契约：
- Run `spotify-api status` first.
  先运行 `spotify-api status`。
- If not connected, run `spotify-api authorize-url`. Present the returned URL as a hyperlink with the text `[Connect to Spotify](<connect_url>)`; do not show or paste the raw URL.
  如果未连接，运行 `spotify-api authorize-url`。将返回的 URL 以超链接形式呈现，链接文字为 `[Connect to Spotify](<connect_url>)`；不要展示或粘贴原始 URL。
- If playlist changes stop working after a reconnect, re-link so the token is minted with the latest requested scopes.
  如果重新连接后播放列表更改仍然失效，请重新关联，以便令牌以最新请求的权限范围签发。
- If the user wants to disconnect, run `spotify-api disconnect`. When `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share only this Markdown link: `[Disconnect Spotify](<disconnect_url>)`; explain that the user must open it to confirm. When no URL is returned and the command succeeds, `spotify-api status` may be used to verify the disconnection.
  如果用户想断开连接，运行 `spotify-api disconnect`。当存在 `disconnect_url` 时，用返回的 URL 替换 `<disconnect_url>`，并只分享这个 Markdown 链接：`[Disconnect Spotify](<disconnect_url>)`；说明用户必须打开它进行确认。当没有返回 URL 且命令成功时，可用 `spotify-api status` 验证已断开。
- Do not pass secrets on the command line.
  不要在命令行上传递机密信息。

## Operating Rules / 操作规则
1. Verify connection with `spotify-api status` before `spotify-api` data calls. For Save to Spotify management, start with `save-to-spotify --json shows`; if it reports a token or connection error, ask the user to connect Spotify in Settings → Connections → Spotify, then retry. If it reports that management is unavailable, do not present reconnection as a fix.
   在调用 `spotify-api` 获取数据前，先用 `spotify-api status` 验证连接。对于 Save to Spotify 管理，从 `save-to-spotify --json shows` 开始；如果它报告令牌或连接错误，请用户在 Settings → Connections → Spotify 中连接 Spotify，然后重试。如果它报告管理不可用，不要把重新连接当作解决办法。
2. Browse with `search`, `library`, and `experience`. Extract relevant items; never dump full responses.
   使用 `search`、`library` 和 `experience` 浏览。提取相关条目；绝不原样倾倒完整响应。
3. When a section includes `next`, call `spotify-api next-page --url <next>` to fetch additional pages. Continue following `next` until it is absent or the user has enough results.
   当某个分节包含 `next` 时，调用 `spotify-api next-page --url <next>` 获取更多页。持续跟随 `next`，直到它不存在或结果已足够。
4. The documented Save to Spotify show and episode deletions may proceed from a clear, unambiguous user request without an additional confirmation. Do not promise unsupported `spotify-api` cleanup (unsave/remove, playlist deletion, remove-from-playlist, or reordering).
   文档所述的 Save to Spotify 节目和单集删除，在用户明确且无歧义的请求下即可执行，无需额外确认。不要承诺 `spotify-api` 不支持的清理操作（取消保存/移除、删除播放列表、从播放列表移除或重新排序）。
5. Direct `spotify-api` playback requires an active Spotify Connect device. Run `spotify-api devices` or `spotify-api now-playing` first and prefer an explicit `--target-device-id` where supported. Voice playback through `muse.music` with `auto` may instead use the calling device's live `music_fulfillment` capability.
   直接使用 `spotify-api` 播放需要一台活跃的 Spotify Connect 设备。先运行 `spotify-api devices` 或 `spotify-api now-playing`，并在支持时优先使用显式的 `--target-device-id`。通过 `muse.music` 且目标为 `auto` 的语音播放，则可改用通话设备实时的 `music_fulfillment` 能力。
6. Every response referencing existing Spotify content must include a Spotify deep link. Use `spotify_url`, or construct `https://open.spotify.com/{type}/{id}` from `spotify_uri`. A Save to Spotify deletion confirmation is the exception: the resource no longer exists, so name the deleted title without exposing its internal ID or constructing a dead link.
   每个引用了现存 Spotify 内容的响应都必须包含一个 Spotify 深度链接。使用 `spotify_url`，或由 `spotify_uri` 构造 `https://open.spotify.com/{type}/{id}`。Save to Spotify 的删除确认是例外：资源已不存在，因此只说出被删除的标题，不暴露其内部 ID，也不构造失效链接。
7. Reference Spotify by name ("on Spotify" / "via Spotify") whenever you surface content or confirm an action.
   每当呈现内容或确认某个操作时，都要点名 Spotify（“on Spotify” / “via Spotify”）。
8. Flag explicit content: when `is_explicit: true`, show `[E]` next to the title.
   标记露骨内容：当 `is_explicit: true` 时，在标题旁显示 `[E]`。
9. After a direct `spotify-api` playback change (`play`, `skip`, `previous`, `resume`), follow up with `now-playing` and name the track plus creator; never confirm with only a device name. For `muse.music`, use its result and do not issue a second playback action.
   在直接的 `spotify-api` 播放变更（`play`、`skip`、`previous`、`resume`）之后，跟进 `now-playing` 并说出曲目及创作者；绝不只用设备名称进行确认。对于 `muse.music`，使用其结果，不要再发出第二个播放操作。
10. A successful playback response means the action took effect. If playback still errors after CLI retries, surface it once in plain user-facing language. If an action is not available, point the user to the Spotify app rather than speculating.
    播放响应成功即表示操作已生效。如果 CLI 重试后播放仍然出错，用平实的用户语言说明一次。如果某个操作不可用，引导用户前往 Spotify 应用，而不要臆测。
11. If an exact song or album by an artist is missing from search, or a known Spotify item cannot be resolved for playlist or playback actions, do not substitute a cover, tribute, karaoke, or similarly named item. Relay `catalog_fallback.message` verbatim; it already renders the approved Muse copy with Markdown links to the requested content search and the exact artist page. Do not add a cause, preamble, follow-up, or alternative wording; blame the user's account; suggest reconnecting; or claim the item was removed from Spotify.
    如果搜索结果中缺少某艺人的确切歌曲或专辑，或者在播放列表或播放操作中无法解析出某个已知的 Spotify 条目，不要用翻唱、致敬、卡拉OK或名称相近的条目替代。逐字转达 `catalog_fallback.message`；它已完整渲染了 Muse 的官方批准文案，其中包含指向所请求内容搜索和确切艺人页面的 Markdown 链接。不要补充原因、开场白、后续追问或替代表述；不要归咎于用户的账户；不要建议重新连接；也不要声称该条目已从 Spotify 移除。

【评论】第 11 条要求逐字转达预渲染的 `catalog_fallback.message` 并禁止任何补充或变通表述，属于脚本化回复设计，用于保证官方话术一致并抑制模型在条目缺失时的自行发挥。
