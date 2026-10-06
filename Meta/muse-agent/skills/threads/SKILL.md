<!-- BILINGUAL-EN-ZH -->
---
name: "threads"
description: "Read and manage the user's Threads account: profile, posts, feed, saved posts, activity, insights, social graph, search, trends, and a specific post by URL or ID. Can tune feed ranking and publish posts on request."
icon: "threads"
metadata: { "includeInPrompt": true }
---

# Threads CLI / Threads 命令行工具

## Purpose / 用途
Read and manage authenticated Threads data using the `threads-cli` companion
CLI. Reads, `dear-algo-whisper`, and approved `publish-post` invocations are
available wherever the Threads skill is supported.

使用配套命令行工具 `threads-cli` 读取和管理经过身份验证的 Threads 数据。在支持 Threads 技能的所有场景中，读取操作、`dear-algo-whisper` 以及获得批准的 `publish-post` 调用均可用。

Use the separate `threads_messages` skill for Threads inboxes and message threads.

Threads 收件箱和私信会话请使用单独的 `threads_messages` 技能。

## Account Linking / 账户关联

Before running any other command, verify the user's Threads account is connected by running `threads-cli accounts`. If the command returns account info (one or more accounts with `id`), the account is connected — proceed normally. Cache this result for the rest of the conversation; do not re-run the check before every command. If any subsequent command fails with an auth or account error, re-run `threads-cli accounts` to recheck account linking status.

在运行任何其他命令之前，先通过运行 `threads-cli accounts` 验证用户的 Threads 账户是否已连接。若该命令返回账户信息（一个或多个带 `id` 的账户），则账户已连接——正常继续。将该结果缓存至本次对话结束；不要在每条命令前都重新运行该检查。若后续任何命令出现认证或账户错误，重新运行 `threads-cli accounts` 以复查账户关联状态。

If the command fails or returns an empty result indicating no account is linked, the account is not connected. Get the connect URL by running `threads-cli connect-url` (it outputs JSON with a `connect_url` field), then tell the user, substituting that URL:

若该命令失败或返回空结果（表明没有已关联的账户），则账户未连接。运行 `threads-cli connect-url` 获取连接 URL（它输出包含 `connect_url` 字段的 JSON），然后代入该 URL 告知用户：

> Your Threads account is not connected. To connect it, visit ``[Meta Accounts Center](`connect_url`)`` and link your Threads account.
> 你的 Threads 账户尚未连接。要连接，请访问 ``[Meta Accounts Center](`connect_url`)`` 并关联你的 Threads 账户。

If the user asks to disconnect their Threads account, run `threads-cli disconnect-url` (it outputs JSON with a `disconnect_url` field) and direct them to that URL:

若用户要求断开 Threads 账户的连接，运行 `threads-cli disconnect-url`（它输出包含 `disconnect_url` 字段的 JSON），并引导用户前往该 URL：

> To disconnect your Threads account, visit ``[Meta Accounts Center](`disconnect_url`)`` and remove the linked account.
> 要断开 Threads 账户的连接，请访问 ``[Meta Accounts Center](`disconnect_url`)`` 并移除已关联的账户。

Always read these URLs from the command output rather than hardcoding them.

始终从命令输出读取这些 URL，不要硬编码。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
threads-cli <target> [options]
```

Targets:
- `accounts`
- `profile`
- `activity-feed`
- `liked-media`
- `saved-posts`
- `insights-overview`
- `post-insights`
- `top-posts`
- `user-profile`
- `profile-threads`
- `profile-replies`
- `profile-media`
- `followers`
- `following`
- `post`
- `fetch-post-comments` (alias: `comments`)
  `fetch-post-comments`（别名：`comments`）
- `fetch-post-likers` (alias: `likers`)
  `fetch-post-likers`（别名：`likers`）
- `feedback-hub-overview`
- `feedback-hub-tab`
- `trends`
- `search`
- `feed`
- `publish-post` (hidden; explicit confirmation required)
  `publish-post`（隐藏；需要明确确认）
- `dear-algo-whisper`

### Post output / 帖子输出

`feed` and `post` use the same compact `social_posts_v1` presentation as
`instagram-cli`. Use the filtered result as authoritative; do not reconstruct
omitted fields. An explicit error-only post row is omitted and its message is
surfaced as `provider_error`.

`feed` 和 `post` 使用与 `instagram-cli` 相同的紧凑 `social_posts_v1` 呈现格式。以过滤后的结果为准，不要重构被省略的字段。明确的仅错误帖子行会被省略，其消息以 `provider_error` 呈现。

### Global options / 全局选项
- `--account-id <threads_account_id>` **(required for all commands except `accounts`)** — select which Threads account to operate on. The value must be the authenticated user's own `id` field from the `accounts` response. Always call `accounts` first. If it returns multiple entries, ask the user which account to use.
  `--account-id <threads_account_id>` **（除 `accounts` 外的所有命令都必需）** — 选择要操作哪个 Threads 账户。该值必须是 `accounts` 响应中经过身份验证的用户自己的 `id` 字段。始终先调用 `accounts`。若返回多个条目，询问用户使用哪个账户。
- `--retries <N>` — retry transient failures (default: 0). This is not
  supported for `dear-algo-whisper` or `publish-post`, because a
  retry could duplicate a mutation.
  `--retries <N>` — 重试瞬时故障（默认：0）。`dear-algo-whisper` 和 `publish-post` 不支持该选项，因为重试可能导致变更重复。

## Commands / 命令

### Accounts / 账户
List Threads accounts associated with the currently authenticated user.

列出与当前已认证用户关联的 Threads 账户。

```sh
threads-cli accounts
```

### Profile / 个人资料
Fetch the current user's own Threads profile.

获取当前用户自己的 Threads 个人资料。

```sh
threads-cli profile --account-id <threads_account_id>
```

### Activity Feed / 活动动态
Fetch the user's activity notifications.

获取用户的活动通知。

```sh
threads-cli activity-feed --account-id <threads_account_id>
threads-cli activity-feed --account-id <threads_account_id> --first 20
threads-cli activity-feed --account-id <threads_account_id> --category-filter text_post_app_mentions
threads-cli activity-feed --account-id <threads_account_id> --after <cursor>
```

Category filters: `text_post_app_conversations`, `text_post_app_following`, `text_post_app_private_follow_requests`, `text_post_app_mentions`, `text_post_app_replies`, `text_post_app_user_follows`, `text_post_app_quote_posts`, `text_post_app_reposts`.

类别过滤器：`text_post_app_conversations`、`text_post_app_following`、`text_post_app_private_follow_requests`、`text_post_app_mentions`、`text_post_app_replies`、`text_post_app_user_follows`、`text_post_app_quote_posts`、`text_post_app_reposts`。

### Liked Media / 已点赞媒体
Fetch posts you've liked.
This uses the shared engagement query and supports `--since`, `--until`, `--sort-order`, `--limit`, and `--after`.

获取你点赞过的帖子。
该命令使用共享的互动查询，支持 `--since`、`--until`、`--sort-order`、`--limit` 和 `--after`。

```sh
threads-cli liked-media --account-id <threads_account_id>
threads-cli liked-media --account-id <threads_account_id> --limit 20
threads-cli liked-media --account-id <threads_account_id> --since 2026-03-01 --until 2026-03-20
threads-cli liked-media --account-id <threads_account_id> --sort-order asc
threads-cli liked-media --account-id <threads_account_id> --after <cursor>
```

### Saved Posts / 已保存帖子
Fetch your saved posts.
This uses the shared engagement query and supports `--since`, `--until`, `--sort-order`, `--limit`, and `--after`.

获取你保存的帖子。
该命令使用共享的互动查询，支持 `--since`、`--until`、`--sort-order`、`--limit` 和 `--after`。

```sh
threads-cli saved-posts --account-id <threads_account_id>
threads-cli saved-posts --account-id <threads_account_id> --limit 20
threads-cli saved-posts --account-id <threads_account_id> --since 2026-03-01 --until 2026-03-20
threads-cli saved-posts --account-id <threads_account_id> --after <cursor>
```

### Insights Overview / 数据概览
Fetch account-level insights (views, likes, quotes, replies, reposts, traffic sources, demographics).

获取账户级数据洞察（浏览、点赞、引用、回复、转发、流量来源、受众人口统计）。

```sh
threads-cli insights-overview --account-id <threads_account_id> --start-date 2026-03-13 --end-date 2026-03-20
threads-cli insights-overview --account-id <threads_account_id> --start-date 2026-03-13 --end-date 2026-03-20 --sections followers
```

Section values: `summary`, `views`, `interactions`, `followers`, `demographics`, `all`.

区块取值：`summary`、`views`、`interactions`、`followers`、`demographics`、`all`。

The typed insights backend uses the Threads account selected by
`--account-id`. Do not retry an account-binding failure through `/gq`; run
`accounts` again and pass one of the returned account IDs.

结构化数据后端使用 `--account-id` 所选定的 Threads 账户。不要通过 `/gq` 重试账户绑定失败；重新运行 `accounts` 并传入返回的某个账户 ID。

### Post-Level Insights / 帖子级数据
Fetch insights for a specific post.

获取特定帖子的数据洞察。

```sh
threads-cli post-insights --account-id <threads_account_id> --post-id 3856993780407305605
```

### Top Posts / 热门帖子
Fetch the most-viewed posts and the service-defined top three most-liked posts
over a date range. `--count` controls most-viewed posts and is bounded to 50.

获取日期范围内浏览最多的帖子，以及服务端定义的点赞最多的前三名帖子。`--count` 控制浏览最多帖子的数量，上限为 50。

```sh
threads-cli top-posts --account-id <threads_account_id> --start-date 2026-03-13 --end-date 2026-03-20
threads-cli top-posts --account-id <threads_account_id> --start-date 2026-03-13 --end-date 2026-03-20 --count 5
```

### Other User's Profile / 他人资料
Fetch another user's profile by numeric Threads user FBID. This command does
not accept usernames or profile URLs.

通过数字 Threads 用户 FBID 获取其他用户的资料。该命令不接受用户名或主页 URL。

```sh
threads-cli user-profile --account-id <threads_account_id> --user-id 12345678
```

### Profile Threads / 用户帖子
Fetch threads by numeric Threads user FBID. `--limit` is accepted as a
compatibility alias for `--first`.

通过数字 Threads 用户 FBID 获取其帖子。`--limit` 被接受为 `--first` 的兼容别名。

```sh
threads-cli profile-threads --account-id <threads_account_id> --user-id 12345678
threads-cli profile-threads --account-id <threads_account_id> --user-id 12345678 --first 10 --after <cursor>
```

### Profile Replies / 用户回复
Fetch a user's replies.

获取某用户的回复。

```sh
threads-cli profile-replies --account-id <threads_account_id> --user-id 12345678
threads-cli profile-replies --account-id <threads_account_id> --user-id 12345678 --first 10 --after <cursor>
```

### Profile Media / 用户媒体
Fetch a user's media posts (images, videos, carousels).

获取某用户的媒体帖子（图片、视频、轮播）。

```sh
threads-cli profile-media --account-id <threads_account_id> --user-id 12345678
threads-cli profile-media --account-id <threads_account_id> --user-id 12345678 --first 10 --after <cursor>
```

### Followers / 粉丝
Fetch a user's followers list.

获取某用户的粉丝列表。

```sh
threads-cli followers --account-id <threads_account_id> --user-id 12345678
threads-cli followers --account-id <threads_account_id> --user-id 12345678 --first 10 --after <cursor>
```

### Following / 关注
Fetch a user's following list.

获取某用户的关注列表。

```sh
threads-cli following --account-id <threads_account_id> --user-id 12345678
threads-cli following --account-id <threads_account_id> --user-id 12345678 --first 10 --after <cursor>
```

### Post by URL or ID / 按 URL 或 ID 获取帖子
Fetch a specific post. Use `fetch-post-comments` for its replies.

获取特定帖子。其回复请使用 `fetch-post-comments`。

Exactly one of `--url` or `--post-id` (alias `--id`) is required.

必须且只能提供 `--url` 或 `--post-id`（别名 `--id`）之一。

When the user supplies a Threads permalink or share link (`threads.com` or
`threads.net`), use `--url` first and pass the original URL unchanged. WWW
resolves it and enforces access.

当用户提供 Threads 永久链接或分享链接（`threads.com` 或 `threads.net`）时，优先使用 `--url` 并原样传递原始 URL，由 WWW 解析并执行访问控制。

```sh
threads-cli post --account-id <threads_account_id> --url https://www.threads.com/@carnage4life/post/DdU_q-9mLbt
threads-cli post --account-id <threads_account_id> --url https://www.threads.com/share/HCmFh1x9l/
```

Use `--post-id` when you already have a known numeric post FBID:

当你已有已知的数字帖子 FBID 时，使用 `--post-id`：

```sh
threads-cli post --account-id <threads_account_id> --post-id 3856993780407305605
```

### Fetch Post Comments / 获取帖子评论
Fetch comments for one or more post media IDs.
Use `--since`, `--until`, `--sort-order`, `--limit`, and repeated/comma-separated `--author-id` filters when needed.

获取一个或多个帖子媒体 ID 的评论。
需要时使用 `--since`、`--until`、`--sort-order`、`--limit` 以及重复/逗号分隔的 `--author-id` 过滤器。

```sh
threads-cli fetch-post-comments --account-id <threads_account_id> --post-ids 3856993780407305605
threads-cli fetch-post-comments --account-id <threads_account_id> --post-ids 3856993780407305605,3856993780407305606 --limit 20 --after <cursor>
threads-cli fetch-post-comments --account-id <threads_account_id> --post-ids 3856993780407305605 --since 2026-03-01 --until 2026-03-20 --author-id 12345678
threads-cli comments --account-id <threads_account_id> --post-ids 3856993780407305605
```

### Fetch Post Likers / 获取帖子点赞者
Fetch users who liked one or more post media IDs.
Use `--since`, `--until`, `--sort-order`, `--limit`, and repeated/comma-separated `--reactor-id` filters when needed.

获取点赞一个或多个帖子媒体 ID 的用户。
需要时使用 `--since`、`--until`、`--sort-order`、`--limit` 以及重复/逗号分隔的 `--reactor-id` 过滤器。

```sh
threads-cli fetch-post-likers --account-id <threads_account_id> --post-ids 3856993780407305605
threads-cli fetch-post-likers --account-id <threads_account_id> --post-ids 3856993780407305605 --limit 20 --after <cursor>
threads-cli fetch-post-likers --account-id <threads_account_id> --post-ids 3856993780407305605 --since 2026-03-01 --until 2026-03-20 --reactor-id 12345678
threads-cli likers --account-id <threads_account_id> --post-ids 3856993780407305605
```

### Feedback Hub Overview / 互动中心概览
Fetch a post's engagement summary (likes, reposts, quotes counts).

获取帖子的互动摘要（点赞、转发、引用计数）。

```sh
threads-cli feedback-hub-overview --account-id <threads_account_id> --post-id 3856993780407305605
```

### Feedback Hub Tab / 互动中心标签页
Fetch paginated lists of users who liked, reposted, or quoted a post.

分页获取点赞、转发或引用某帖子的用户列表。

```sh
threads-cli feedback-hub-tab --account-id <threads_account_id> --post-id 3856993780407305605 --tab-type like
threads-cli feedback-hub-tab --account-id <threads_account_id> --post-id 3856993780407305605 --tab-type repost --first 10
threads-cli feedback-hub-tab --account-id <threads_account_id> --post-id 3856993780407305605 --tab-type quote --after <cursor>
```

### Trends / 趋势
Fetch trending topics on Threads.

获取 Threads 上的热门话题。

```sh
threads-cli trends --account-id <threads_account_id>
threads-cli trends --account-id <threads_account_id> --first 10
```

### Search / 搜索
Search Threads by keyword, or dive deeper into a trend.

按关键词搜索 Threads，或深入探索某个趋势。

```sh
threads-cli search --account-id <threads_account_id> --query "AI news"
threads-cli search --account-id <threads_account_id> --query "AI news" --recent 1
threads-cli search --account-id <threads_account_id> --query "trending topic" --trend-fbid 987654
threads-cli search --account-id <threads_account_id> --query "AI" --first 10 --after <cursor>
```

Use `--recent 1` for "Recent" tab results instead of "Top".

使用 `--recent 1` 获取"最近"（Recent）标签页的结果，而不是"热门"（Top）。

### Feed / 信息流
Fetch the ranked feed (For You or Following).

获取排序后的信息流（For You 或 Following）。

For You feed:  
For You 信息流：  
```sh
threads-cli feed --account-id <threads_account_id> --variant for_you
```

Following feed:  
Following 信息流：  
```sh
threads-cli feed --account-id <threads_account_id> --variant following
threads-cli feed --account-id <threads_account_id> --variant following --sort-by recent
```

For all feed variants, use `--after` for pagination:  
所有信息流变体均使用 `--after` 分页：  
```sh
threads-cli feed --account-id <threads_account_id> --variant for_you --after <cursor>
```

### Publish a post / 发布帖子

`publish-post` is a hidden write command. It publishes a normal Threads post
in one approved operation and returns the post ID. There is no public draft
step and no creation handle to pass between commands.

`publish-post` 是一条隐藏的写入命令。它在一个获批准的操作中发布一条普通 Threads 帖子并返回帖子 ID。没有公开的草稿步骤，也没有可在命令之间传递的创建句柄。

Text must contain at least one non-whitespace character when no media is
present. Every byte of accepted text, including leading or trailing whitespace
and Unicode, is preserved through confirmation and publication. A post may
include one to twenty ordered local images or videos from the Hatch workspace,
optionally reply to a specific post, and set who can reply. If a file is outside
the workspace, copy it into the workspace first; never use a `/tmp` path.

无媒体时，文本必须包含至少一个非空白字符。被接受的文本的每一个字节——包括首尾空白和 Unicode——在确认与发布过程中都保持不变。帖子可以包含一至二十个来自 Hatch 工作区的有序本地图片或视频，可以选择回复特定帖子，并可设置谁可以回复。若文件在工作区之外，先把它复制进工作区；绝不使用 `/tmp` 路径。

Run `threads-cli accounts` first and pass the selected account's numeric `id`
unchanged as `--account-id`. If multiple accounts are returned, ask which one
to use. Account and reply targets are canonical positive decimal FBIDs with no
sign, leading zero, or surrounding whitespace; never derive one from a URL or
username.

先运行 `threads-cli accounts`，并把所选账户的数字 `id` 原样作为 `--account-id` 传入。若返回多个账户，询问使用哪一个。账户与回复目标均为规范的十进制正数 FBID，不带符号、前导零或首尾空白；绝不从 URL 或用户名推导。

```sh
threads-cli publish-post --account-id <threads_account_fbid> --text "Hello Threads"
threads-cli publish-post --account-id <threads_account_fbid> --text "Caption" --media-item '{"file":"/workspace/Launch photo.jpg","alt_text":"Description"}'
threads-cli publish-post --account-id <threads_account_fbid> --text "Mixed media" --media-item '{"file":"/workspace/first.jpg","alt_text":"First image"}' --media-item '{"file":"/workspace/second.mp4","cover":"/workspace/second-cover.jpg"}'
threads-cli publish-post --account-id <threads_account_fbid> --text "Reply video" --media-item '{"file":"/workspace/video.mp4","cover":"/workspace/video-cover.jpg"}' --reply-to-post-fbid <post_fbid> --reply-control mentioned_only
```

Reply control accepts exactly `everyone`, `accounts_you_follow`,
`mentioned_only`, `parent_post_author_only`, or `followers_only`. The old
`following`, `mentioned`, and `followers` spellings are invalid. Never silently
translate an older spelling.

回复控制只接受 `everyone`、`accounts_you_follow`、`mentioned_only`、`parent_post_author_only` 或 `followers_only`。旧拼写 `following`、`mentioned` 和 `followers` 无效，绝不悄悄转换旧拼写。

`--media-item` is repeatable from one to twenty times. Each JSON value has
exactly `file`, optional `cover`, and optional exact `alt_text`. Supported media
files are JPEG, PNG, static WebP, MP4, and MOV, at most 100 MB each. Media type
is inferred from the extension. Every MP4/MOV requires a JPEG, PNG, or WebP
`cover`; images reject `cover`. File order is display order, and each cover is
kept with its video item. Omit all media items for a text-only post. One item
publishes an image or video; two to twenty items publish one carousel in the
supplied order.

`--media-item` 可重复一至二十次。每个 JSON 值恰好包含 `file`、可选的 `cover` 和可选的精确 `alt_text`。支持的媒体文件为 JPEG、PNG、静态 WebP、MP4 和 MOV，每个不超过 100 MB。媒体类型由扩展名推断。每个 MP4/MOV 都需要 JPEG、PNG 或 WebP 的 `cover`；图片则拒绝 `cover`。文件顺序即展示顺序，每个封面与其视频项保持配对。纯文本帖子省略所有媒体项。单个项发布一张图片或一个视频；两到二十个项按给定顺序发布一个轮播。

After approval, the CLI reads every declared file through its privsep input and
multipart-uploads it to the private Threads staging endpoint. A single item is
staged with its exact text, alt text, and reply settings, then `publish-post`
receives only the private returned creation ID. For a carousel, the CLI stages
each ordered item with empty text, no reply settings, and
`is_carousel_item=true`, validates every distinct returned creation ID, then
publishes one `media_type=CAROUSEL` parent with the ordered `children`, exact
text, and optional reply fields. Paths and creation IDs remain internal.

获批之后，CLI 通过其 privsep 输入读取每个声明的文件，并分段上传到私有的 Threads 暂存端点。单个项会连同其精确文本、替代文本和回复设置一起暂存，随后 `publish-post` 只收到私有的返回创建 ID。对于轮播，CLI 以空文本、无回复设置和 `is_carousel_item=true` 暂存每个有序项，校验每个不同的返回创建 ID，然后发布一个 `media_type=CAROUSEL` 的父帖，附带有序的 `children`、精确文本和可选回复字段。路径与创建 ID 保持内部不外露。

【评论】文件经 privsep 隔离读取、创建 ID 只在私有端点之间流转，限制了敏感路径与句柄在会话中的暴露面。

The CLI validates the complete post before approval. One `content.post`
approval covers the account, human reply context, reply control, exact text,
and all ordered media. Confirmation shows each item from an authenticated
workspace preview resource with a safe basename and optional exact alt text; it
also shows every video cover as an adjacent image attachment immediately after
its video. The twenty-media maximum can therefore produce forty visual
attachments. Confirmation never substitutes raw identifiers. Missing or unsafe
names become `Threads image` or `Threads video`.
Names must be nonempty after trimming, at most 255 Unicode scalar characters,
and contain no slash, backslash, control character, numeric-only stem,
hexadecimal digest stem, or provider/upload identifier. Alt text must be
nonempty after trimming, at most 1024 Unicode scalar characters, and contain no
control character. Truncation fails closed.

CLI 在批准之前校验完整帖子。一次 `content.post` 批准覆盖账户、真人回复上下文、回复控制、精确文本和全部有序媒体。确认界面通过经过身份验证的工作区预览资源展示每一项，带安全的基本文件名和可选的精确替代文本；每个视频封面都作为紧随其视频之后的图片附件展示。因此，二十个媒体的上限可能产生四十个视觉附件。确认界面绝不以原始标识符替代显示。缺失或不安全的名称显示为 `Threads image` 或 `Threads video`。
名称在去除首尾空白后必须非空、不超过 255 个 Unicode 标量字符，且不得包含斜杠、反斜杠、控制字符、纯数字词干、十六进制摘要词干或服务商/上传标识符。替代文本在去除首尾空白后必须非空、不超过 1024 个 Unicode 标量字符，且不得包含控制字符。发生截断时一律失败（fail closed）。

【评论】"截断即失败"（fail closed）保证确认界面呈现的内容与实际发布的内容严格一致，宁可拒绝也不降级展示。

After approval, every write uses zero retries. A child failure or duplicate
handle stops immediately without creating later children or publishing the
parent. Any later attempt is a new invocation requiring new explicit approval.
Creation IDs are accepted only from the private staging response.

获批之后，所有写入都零重试。子项失败或句柄重复会立即停止，不会创建后续子项或发布父帖。之后的任何尝试都是新的调用，需要新的明确批准。创建 ID 只接受来自私有暂存响应的值。

### Post write safety / 发帖写入安全

1. Never publish until the explicit Hatch confirmation window is approved. A
   decline or dismissal ends the action.
   在明确的 Hatch 确认窗口获得批准之前绝不发布。拒绝或关闭窗口即终止该操作。
2. Never use transport retries. A child or parent failure is terminal and
   ambiguous. Do not continue a failed carousel. Any later attempt is a new
   invocation with a new explicit approval.
   绝不使用传输层重试。子项或父项失败是终结性且状态不明的。不要继续一个已失败的轮播。之后的任何尝试都是新的调用，需要新的明确批准。
3. Show exact post text, exact reply control, all ordered media items, and every
   video cover described above. Resolve account and reply labels on a
   best-effort basis for up to 30 seconds. If lookup times out or returns an
   error, malformed data, or a missing/unsafe label, show a non-identifying  
   `Unverified Threads …`  
   placeholder and continue to explicit approval; never substitute its raw
   identifier. Never show account, reply, child, creation, or post IDs or local
   paths in normal prose.
   展示精确的帖子文本、精确的回复控制、全部有序媒体项以及上述每个视频封面。账户与回复标签最多用 30 秒尽力解析。若查询超时、返回错误、畸形数据或缺失/不安全的标签，则显示不具识别性的  
   `Unverified Threads …`  
   占位符并继续进入明确批准；绝不以原始标识符替代。正常行文中绝不显示账户、回复、子项、创建或帖子 ID，也不显示本地路径。
4. Post text is trimmed only to test emptiness. Every byte of a nonempty value,
   including leading/trailing whitespace and Unicode, is preserved in both the
   structured confirmation preview and dispatched request. If the full text
   cannot fit without preview truncation, the write is rejected.
   帖子文本只在测试是否为空时做修剪。非空值的每一个字节——包括首尾空白和 Unicode——在结构化确认预览与实际发出的请求中均保持不变。若完整文本无法在不截断预览的情况下容纳，则拒绝该写入。
5. This write command is available wherever the Threads skill is supported,
   but every invocation still requires its own explicit confirmation.
   该写入命令在支持 Threads 技能的所有场景均可用，但每次调用仍需要其自身的明确确认。
6. Respect authorization, account-selection, and quota failures. Do not bypass
   them through an alternate or undocumented command path.
   尊重授权、账户选择和配额失败。不要通过替代或未公开的命令路径绕过它们。
7. Read the published post back and compare it with the approved snapshot when
   the exact publication result matters.
   当发布结果的精确性很重要时，回读已发布的帖子并与批准的快照比较。

### Dear Algo Whisper / 致算法低语
Send a message directly to the Threads ranking algorithm to modify what the user sees in their feed (e.g. "show me less politics", "more cat content"). This command is available to all supported users, and a clear user request may proceed without an additional confirmation.

直接向 Threads 排序算法发送消息，修改用户在信息流中看到的内容（例如"少给我看政治内容""多来点猫咪内容"）。该命令对所有受支持的用户可用，且明确的用户请求无需额外确认即可执行。

```sh
threads-cli dear-algo-whisper --account-id <threads_account_id> --message "show me less politics"
```

【评论】"dear-algo-whisper" 提供了一个以自然语言直接干预推荐排序的接口，把部分排序控制权交还给用户，这在推荐系统中较为少见。

## Operating rules / 运行规则
1. This skill reads Threads data the authenticated user can access — their own account plus posts they supply links to — not general Threads search.
   本技能读取已认证用户可访问的 Threads 数据——其自己的账户加上其提供链接的帖子——而非一般性的 Threads 搜索。
2. **Always call `accounts` first** to obtain the account `id` for `--account-id`. Cache this value for subsequent commands in the same conversation. If `accounts` returns no entries, direct the user to Meta Accounts Center to link a Threads account (refer to the "Account Linking" section for details).
   **始终先调用 `accounts`** 以获取用于 `--account-id` 的账户 `id`。在同一对话中为后续命令缓存该值。若 `accounts` 未返回任何条目，引导用户前往 Meta Accounts Center 关联 Threads 账户（详情参见"账户关联"一节）。
3. **Minimize API calls to avoid rate limiting.** The Threads API enforces strict rate limits — excessive calls will result in `429 Too Many Requests` errors that are not retried. Follow these principles:
   **尽量减少 API 调用以避免触发限流。** Threads API 执行严格的速率限制——过量调用会导致 `429 Too Many Requests` 错误，且不会重试。遵循以下原则：
   - **Batch your information gathering.** Before making calls, plan which data you actually need. Don't fetch data speculatively.
     **批量收集信息。** 在调用之前，先规划你真正需要哪些数据。不要凭猜测抓取数据。
   - **Reuse data from earlier responses.** If you already fetched `accounts` or `profile`, extract IDs and usernames from that response instead of calling again.
     **复用早前响应中的数据。** 若已经获取过 `accounts` 或 `profile`，从该响应中提取 ID 和用户名，而不是再次调用。
   - **Avoid redundant pagination.** Only paginate (`--after`) when the user explicitly needs more results. Don't automatically fetch all pages.
     **避免多余的分页。** 只有当用户明确需要更多结果时才分页（`--after`）。不要自动抓取所有页。
   - **Never call the same command twice with identical arguments** in one conversation unless the previous call failed or the user explicitly asks for a refresh.
     **在同一对话中绝不用完全相同的参数调用同一命令两次**，除非上一次调用失败或用户明确要求刷新。
4. Avoid requests to persistently / frequently poll these commands.
   避免持续/频繁轮询这些命令的请求。
5. Use numeric Threads FBIDs for account, user, post, and other entity identifiers. Do not derive an identifier from a Threads URL or pass a URL where an FBID is required. The one exception is `post --url`, which takes the original Threads URL as-is and lets WWW resolve it.
   账户、用户、帖子及其他实体标识符使用数字 Threads FBID。不要从 Threads URL 推导标识符，也不要在需要 FBID 的地方传入 URL。唯一的例外是 `post --url`，它原样接受原始 Threads URL 并由 WWW 解析。
6. Treat U18 enforcement as a server-side invariant. Do not reconstruct filtered fields, retry through `/gq`, or otherwise bypass server results.
   把 U18（未成年人保护）强制视为服务端不变量。不要重构被过滤的字段、不要通过 `/gq` 重试，也不要以其他方式绕过服务端结果。
7. Use the structured, filtered media result returned by each named read command; do not reconstruct omitted provider fields.
   使用每个具名读取命令返回的结构化、已过滤的媒体结果；不要重构被省略的服务商字段。

【评论】把未成年人内容限制声明为服务端不变量、并禁止客户端绕过，是内容安全中"不信任客户端"的典型做法。

## Output / 输出
The CLI prints decoded JSON to stdout. `publish-post` prints exactly
`{"post_id":"<string>"}`. Treat the value as a machine-only workflow handle
and never repeat it in user-facing prose or an approval preview.

CLI 将解码后的 JSON 打印到 stdout。`publish-post` 恰好打印 `{"post_id":"<string>"}`。把该值当作仅供机器使用的工作流句柄，绝不在面向用户的行文或批准预览中重复它。
