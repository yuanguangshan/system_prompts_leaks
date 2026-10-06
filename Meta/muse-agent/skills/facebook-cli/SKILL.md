<!-- BILINGUAL-EN-ZH -->
---
name: "facebook_cli"
icon: "facebook"
title: "Facebook"
description: "Use when the user provides a Facebook URL or asks to read personal posts, comments, reactions, friends, timelines, profiles, stories, feeds, groups, events, or saved items, or to discover public events happening near a place, nearby, or in a local area on a date, or to create, edit, publish, or delete their own Marketplace listings. To find, browse, or buy Marketplace listings, use shopping instead. Use pages commands for managed Facebook Page discovery, insights, native draft editing/deletion, same-draft publication, approved posts and native scheduling."
metadata: { "includeInPrompt": true }
---

# Facebook CLI / Facebook 命令行工具

## Pasted links first / 粘贴链接优先

If the user pasted a Facebook Marketplace item link or a share link, never
open it with the browser: the browser cannot pass the Facebook login wall.
For an item link, run `facebook-cli marketplace listing details --url
'<pasted link>'`. For a share link, decode it first with `facebook-cli
link-sharing decode-url --url '<pasted link>'`, then follow
`references/marketplace.md` Step 0 for the decoded URL.

如果用户粘贴了 Facebook Marketplace 商品链接或分享链接，绝不要用浏览器打开它：浏览器无法通过 Facebook 的登录墙。对商品链接，运行 `facebook-cli marketplace listing details --url '<pasted link>'`。对分享链接，先用 `facebook-cli link-sharing decode-url --url '<pasted link>'` 解码，然后按 `references/marketplace.md` 的 Step 0 处理解码后的 URL。

## Quick Reference / 快速参考

The read-only post and Marketplace-only write restrictions below apply to
personal-profile activity. Managed Pages separately support draft, post and
scheduling writes gated by consent, Page access and operation approval.

下文的"帖子只读、Marketplace 仅限写自家商品"限制适用于个人主页活动。受管主页（Managed Pages）另行支持草稿、帖子与排期写入，并受用户同意、主页访问权限与操作审批的约束。

```
facebook-cli
├── pages                                      # Read references/pages.md first
│   ├── list [--limit N] [--after <cursor>]      # Managed Pages
│   ├── access --page-id <id>                   # Page access diagnostic
│   ├── account-insights --page-id <id>         # Page metrics
│   ├── drafts
│   │   ├── create --page-id <id> ...           # WRITE — prompts for approval
│   │   ├── show --page-id <id> --draft-id <id> # Read a native draft
│   │   ├── publish --page-id <id> ...          # WRITE — same draft; prompts for approval
│   │   ├── edit --page-id <id> ...             # WRITE — prompts for approval
│   │   └── delete --page-id <id> ...           # WRITE — prompts for approval
│   └── posts
│       ├── list --page-id <id>                 # Recent posts with metrics
│       ├── get --page-id <id> --post-ids <ids>  # Selected post metrics
│       ├── create --page-id <id> ...           # WRITE — publish or schedule; prompts for approval
│       ├── reschedule --page-id <id> ...       # WRITE — prompts for approval
│       └── cancel-schedule --page-id <id> ...  # WRITE — prompts for approval
├── post
│   ├── read (--post-id <id> | --url <url>) # Read by numeric ID/PFBID or canonical post/photo/video URL
│   ├── comments
│   │   └── read --post-id <id> [--limit N] [--after <cursor>]  # Read comments (paginated: data[] + paging.cursors.after; post link at summary.post_url)
│   └── reactions
│       └── read --post-id <id>  # Reaction summary only: summary.total_count + summary.reaction_counts (no individual reactor list)
├── link-sharing
│   └── decode-url --url <url>              # Resolve a share link; returns original_url
├── me                                         # Your Facebook name + profile ID
│   └── friends [--name] [--city] [--hometown] [--work] [--education] [--filter-mode AND|OR] [--json-query <json>] [--birthday-within-days [N]] [--limit N] [--after <cursor>]  # Paginated: data[] + paging.cursors.after
├── marketplace
│   ├── search --query "..." [--out <file>] [options]  # Search listings; pass the --out file to shopping.resolve_results, do not read it
│   ├── my-listings [--status active|pending|sold|draft] [--limit N] [--after <cursor>]  # Your own listings (active + inactive), paginated
│   ├── listing details (--listing-id <id> | --url '<item link>') [--out <file>]  # Item details
│   ├── listing create --title "..." --price N --photo /path [options]  # Create; goes LIVE only with photos+condition+category+location (location = --latitude & --longitude; no --location flag), else DRAFT
│   ├── listing edit --listing-id <id> --photo /path [options]  # Edit a listing (photos uploaded automatically)
│   ├── listing delete --listing-id <id>       # WRITE — permanently delete your listing; prompts for approval
│   ├── listing publish --listing-id <id>      # WRITE — take a DRAFT live; needs photos+condition+category+location (location = --latitude & --longitude); prompts for approval
│   └── seller-info --listing-id <id>          # Seller ratings/info
├── groups
│   ├── details --group-id <id-or-vanity> # Fetch name, About, visibility, history, tags, ordered rules, and member count
│   ├── search [--keywords "..."] [--role connected|admin|admod|any] [--after <cursor>]  # Paginated membership listing; keyword + any searches connected/private and public groups
│   └── posts --group-id <id>             # Browse posts in a group; --query filters by text
├── events
│   ├── search [--scope connected|discover] [--keywords "..."] [--location "..."] [--latitude N --longitude=-N] [--radius-in-miles N] [--category ...] [--start-date ...] [--end-date ...] [--limit N] [--after <cursor>]  # Paginated: data[] + paging.cursors.after; coordinates beat --location, which is only city-accurate
│   └── details --event-id <id>           # Read one event; ID from search `id` or `permalink_url`
├── saved
│   ├── list [--type post|video|link|product|reel|event|page] [--collection-id <id>] [--limit N] [--after <opaque>]
│   ├── add --savable-id <FBID> [--type ...] [--collection-id <id>]      # WRITE — auto-allowed
│   ├── remove --savable-id <FBID> [--type ...] [--collection-id <id>]   # WRITE — auto-allowed
│   └── collections
│       ├── list
│       └── create --name "..."                  # WRITE — auto-allowed
├── story
│   └── feed [--limit <n>]               # Fetch your story feed (story tray)
├── feed
│   ├── newsfeed [--limit <n>] [--after <cursor>] # Ranked algorithmic newsfeed (organic stories)
│   └── friends [--limit <n>] [--after <cursor>]  # Ranked friends-only feed
├── timeline
│   └── fetch --profile-id <id> [--limit N] [--after <cursor>]  # Fetch timeline posts; use next_cursor to page (check author_id vs owner_id)
└── profile
    └── info --profile-id <id>                 # Look up profile info
```


## Account Linking / 账户连接

Before running any facebook-cli command, verify the user's Facebook account is connected by running `facebook-cli me`, except for `facebook-cli marketplace search`, `facebook-cli marketplace listing details`, and `facebook-cli marketplace seller-info`. If the command returns account info (name and profile ID), the account is connected — proceed normally. Cache this result for the rest of the conversation; do not re-run the check before every command. If any subsequent command fails with an auth or account error, re-run `facebook-cli me` to recheck account linking status.

在运行任何 facebook-cli 命令之前，先通过运行 `facebook-cli me` 验证用户的 Facebook 账户已连接，`facebook-cli marketplace search`、`facebook-cli marketplace listing details` 和 `facebook-cli marketplace seller-info` 除外。如果该命令返回了账户信息（名字与主页 ID），说明账户已连接——正常继续。把该结果缓存到本次会话结束；不要在每条命令前都重跑该检查。如果后续任何命令因鉴权或账户错误而失败，重跑 `facebook-cli me` 重新检查账户连接状态。

If the command fails or returns an error indicating no account is linked, the account is not connected. Get the connect URL by running `facebook-cli connect-url` (it outputs JSON with a `connect_url` field), then tell the user, substituting that URL:

如果该命令失败，或返回的错误表明没有账户被关联，说明账户未连接。通过运行 `facebook-cli connect-url` 获取连接 URL（它输出带 `connect_url` 字段的 JSON），然后把该 URL 代入后告诉用户：

> Your Facebook account is not connected. To connect it, visit ``[Meta Accounts Center](`connect_url`)`` and link your Facebook account.

> 你的 Facebook 账户尚未连接。要连接它，请访问 ``[Meta Accounts Center](`connect_url`)`` 并关联你的 Facebook 账户。

If the user asks to disconnect their Facebook account, run `facebook-cli disconnect-url` (it outputs JSON with a `disconnect_url` field) and direct them to that URL:

如果用户要求断开其 Facebook 账户的连接，运行 `facebook-cli disconnect-url`（它输出带 `disconnect_url` 字段的 JSON），并引导用户前往该 URL：

> To disconnect your Facebook account, visit ``[Meta Accounts Center](`disconnect_url`)`` and remove the linked account.

> 要断开你的 Facebook 账户，请访问 ``[Meta Accounts Center](`disconnect_url`)`` 并移除已关联的账户。

Always read these URLs from the command output rather than hardcoding them.

始终从命令输出中读取这些 URL，而不是硬编码。

## Post IDs and URLs / 帖子 ID 与 URL

When using `post read`, `post comments read`, or `post reactions read`, these formats are accepted for `--post-id`:
- `123456789012345` — raw numeric post ID
- `pfbid02...` — PFBID format

使用 `post read`、`post comments read` 或 `post reactions read` 时，`--post-id` 接受以下格式：
- `123456789012345` — 原始数字帖子 ID
- `pfbid02...` — PFBID 格式

Canonical post, photo, video, and reel URLs go directly to `post read --url`.
For share links, call `link-sharing decode-url --url <url>` first, then pass
the nonblank canonical `original_url` intact to the reader. Supply exactly one
of `--url` or `--post-id`. Stop on a null, missing, or blank decoded URL or a
failed read; do not guess IDs or retry a media ID as a post ID. See
[posts.md](references/posts.md) for supported URL patterns and other entities.

规范的帖子、图片、视频和 reel URL 直接交给 `post read --url`。对分享链接，先调用 `link-sharing decode-url --url <url>`，然后把非空的规范 `original_url` 原封不动传给读取器。`--url` 与 `--post-id` 恰好提供其一。遇到解码 URL 为 null、缺失或空白，或读取失败，就停止；不要猜测 ID，也不要把媒体 ID 当帖子 ID 重试。受支持的 URL 模式与其他实体见 [posts.md](references/posts.md)。

## Composability / 命令组合

When the user asks "what can I do on Facebook?" or at the start of a session, generate 3-5 contextual suggestions that combine multiple commands into useful read-only workflows.

当用户问"我在 Facebook 上能做什么？"或在会话开始时，生成 3-5 条结合上下文的建议，把多条命令组合成实用的只读工作流。

### How outputs chain across commands / 各命令的输出如何串联

Most chains are mechanical: an ID in one command's output is the `--id` flag of
the next. `me` and `me friends` produce `profile_id` for `profile info` and
`timeline fetch`. Timelines, feeds, and searches produce `post_id` for
`post read`, `post comments read`, and `post reactions read`. `marketplace
search` produces `listing_id` for `listing details` and `seller-info`. `groups
search` produces `group_id` for `groups details` and `groups posts`. `events
search` produces `id` (also embedded in `permalink_url`) for `events details`.
`story feed` returns story buckets with story URLs and owner info.

大多数链条是机械的：一条命令输出中的 ID 就是下一条命令的 `--id` 旗标。`me` 与 `me friends` 产出供 `profile info` 与 `timeline fetch` 使用的 `profile_id`。时间线、feed 和搜索产出供 `post read`、`post comments read` 与 `post reactions read` 使用的 `post_id`。`marketplace search` 产出供 `listing details` 与 `seller-info` 使用的 `listing_id`。`groups search` 产出供 `groups details` 与 `groups posts` 使用的 `group_id`。`events search` 产出供 `events details` 使用的 `id`（也内嵌于 `permalink_url`）。`story feed` 返回带故事 URL 与所有者信息的故事分组。

The chains that are not mechanical:

并非机械式的链条如下：

- **My listings -> manage**: `marketplace my-listings` produces your own `listing_id` -> `marketplace listing edit` (update), `marketplace listing delete` (remove), or `marketplace listing publish` (take a draft live). Use `--status` to scope to active/pending/sold/draft and `--after` to page.

  **我的商品 -> 管理**：`marketplace my-listings` 产出你自己的 `listing_id` -> `marketplace listing edit`（更新）、`marketplace listing delete`（移除）或 `marketplace listing publish`（让草稿上线）。用 `--status` 把范围限定在 active/pending/sold/draft，用 `--after` 翻页。

- **Create draft -> complete -> publish**: `marketplace listing create` missing any of photos/condition/category/location saves a draft -> `marketplace listing edit` to add the missing field(s) -> `marketplace listing publish` to take it live. If publish fails with missing fields, the error names exactly which to add via edit, then retry.

  **创建草稿 -> 补全 -> 发布**：`marketplace listing create` 若缺少 photos/condition/category/location 中的任一项，会保存为草稿 -> 用 `marketplace listing edit` 补上缺失字段 -> 用 `marketplace listing publish` 让其上线。如果发布因缺字段而失败，错误信息会明确指出要通过编辑补哪些字段，补上后重试。

- **Groups posts + friends cross-reference**: When the user asks about friends' activity in a group, call `me friends` to get the friend list, then compare against post authors from `groups posts` — do not ask the user to provide their friend list manually.

  **群组帖子 + 好友交叉引用**：当用户问某群中好友的动态时，调用 `me friends` 获取好友列表，再与 `groups posts` 的帖子作者比对——不要让用户手工提供好友列表。

- **Saved -> content**: `saved list` produces `savable_id` plus `type`. Chain by category:
  - `post` -> `post read`, `post comments read`, `post reactions read`
  - `page` -> `profile info --profile-id <savable_id>`
  - `product` -> `marketplace listing details --listing-id <savable_id>`
  - `event` -> `events details --event-id <savable_id>`

  **收藏 -> 内容**：`saved list` 产出 `savable_id` 与 `type`。按类别串联：
  - `post` -> `post read`、`post comments read`、`post reactions read`
  - `page` -> `profile info --profile-id <savable_id>`
  - `product` -> `marketplace listing details --listing-id <savable_id>`
  - `event` -> `events details --event-id <savable_id>`

- **Saved -> unsave**: `saved list` produces `savable_id` (the content ID) -> `saved remove --savable-id <savable_id> [--type <type>]`

  **收藏 -> 取消收藏**：`saved list` 产出 `savable_id`（内容 ID）-> `saved remove --savable-id <savable_id> [--type <type>]`

- **Collections -> create flow**: `saved collections create` produces `collection_id` for future use

  **收藏夹 -> 创建流程**：`saved collections create` 产出 `collection_id` 供后续使用

### Suggestion guidelines / 建议准则

1. **Phrase as user-centric actions** — "See which of your Marketplace listings still have no photos", not "Run marketplace my-listings then listing edit".

   **以用户视角的动作来措辞**——"看看你的哪些 Marketplace 商品还没有图片"，而不是"运行 marketplace my-listings 然后执行 listing edit"。

2. **Vary suggestions across sessions** — rotate across skill categories.

   **不同会话间轮换建议**——在技能类别之间轮换。

3. **Adapt to context**:
   - **Profile-aware**: If the user's profile mentions hobbies or interests, suggest related workflows (e.g., cyclist -> "Catch up on what your cycling friends have posted").
   - **Time-aware**: Near holidays or weekends, suggest checking timeline for what friends are up to, or tidying up your own Marketplace drafts.
   - **Session-aware**: After publishing a listing -> "Check whether it went live or stayed a draft"; after viewing a post -> "See the reaction counts and what people are saying in the comments".

   **因情境而异**：
   - **主页感知**：如果用户的主页提到爱好或兴趣，建议相关工作流（例如骑行者 -> "看看你的骑行好友最近发了什么"）。
   - **时间感知**：临近节假日或周末时，建议查看时间线了解好友近况，或整理自己的 Marketplace 草稿。
   - **会话感知**：发布商品之后 -> "检查它是上线了还是停留在草稿"；看完一个帖子后 -> "看看表情回应计数，以及大家在评论里说什么"。

4. **Only suggest what's real** — every suggestion must be achievable using the commands documented here. Most personal-profile tools are **read-only** — do not suggest unsupported posts, reactions, or comments. Managed Pages support only the documented draft, post, and scheduling writes. Marketplace writes support only creating, editing, deleting, and publishing the user's own listings.

   **只建议真实可行的事**——每条建议都必须能用本文档所列的命令实现。大多数个人主页工具是**只读的**——不要建议不受支持的发帖、表情回应或评论。受管主页只支持文档所列的草稿、帖子与排期写入。Marketplace 写入只支持对用户自己的商品进行创建、编辑、删除与发布。

5. **Keep it conversational** — short bulleted list with one-line descriptions. Offer to walk through any of them.

   **保持对话感**——用一行描述的简短列表。主动提出可以带用户走一遍其中任何一条。

## References / 参考文件

One file per feature area, each named for the area it covers:  
[references/friends.md](references/friends.md),  
[references/posts.md](references/posts.md),  
[references/pages.md](references/pages.md),  
[references/comments.md](references/comments.md),  
[references/reactions.md](references/reactions.md),  
[references/marketplace.md](references/marketplace.md),  
[references/timeline.md](references/timeline.md),  
[references/profile.md](references/profile.md),  
[references/story.md](references/story.md),  
[references/feed.md](references/feed.md),  
[references/groups.md](references/groups.md),  
[references/events.md](references/events.md),  
[references/saved.md](references/saved.md).

每个功能领域各有一个文件，文件名对应该领域所覆盖的内容：  
[references/friends.md](references/friends.md)，  
[references/posts.md](references/posts.md)，  
[references/pages.md](references/pages.md)，  
[references/comments.md](references/comments.md)，  
[references/reactions.md](references/reactions.md)，  
[references/marketplace.md](references/marketplace.md)，  
[references/timeline.md](references/timeline.md)，  
[references/profile.md](references/profile.md)，  
[references/story.md](references/story.md)，  
[references/feed.md](references/feed.md)，  
[references/groups.md](references/groups.md)，  
[references/events.md](references/events.md)，  
[references/saved.md](references/saved.md)。

## Operating Rules / 操作规则

1. **Open a reference doc only when you need it.** The Quick Reference above covers the common calls. Read the matching file under [references/](references/) when you need a flag it does not list, a response field, or a pagination detail, when a command fails and you need the full contract, or when a rule below points you at one. Resolving a pasted Marketplace or share link always uses `references/marketplace.md` Step 0.

   **仅在需要时才打开参考文档。** 上面的快速参考覆盖了常见调用。当你需要它未列出的某个旗标、某个响应字段或某个分页细节时，当某条命令失败而你需要完整契约时，或当下文某条规则指向它时，阅读 [references/](references/) 下对应的文件。解析粘贴的 Marketplace 或分享链接，始终使用 `references/marketplace.md` 的 Step 0。

2. **Output field requirements (quick reference):**
   - **Posts:** normalized feed, timeline, and post reads preserve `created_at` and add `post_created_at` with UTC and user-local forms. Prefer `post_created_at.user_local`.
   - **Comments:** always include comment timestamps (`created_time`) and comment permalink URLs (`comment_url`) for every comment shown.
   - **Reactions:** always include a per-type count breakdown (e.g., "3 Love, 2 Like, 1 Wow") and post permalink URLs.
   - **Friends:** when searching by nickname, also search the full name (e.g., "Liz" → also search "Elizabeth"; "Mike" → "Michael"; "Bob" → "Robert").
   - **Cross-references:** when identifying people across multiple posts, include links to the specific posts and comments/reactions.

   **输出字段要求（快速参考）：**
   - **帖子：** 规范化的 feed、timeline 与 post 读取会保留 `created_at`，并添加包含 UTC 与用户本地两种形式的 `post_created_at`。优先使用 `post_created_at.user_local`。
   - **评论：** 展示的每条评论都要带上评论时间戳（`created_time`）与评论永久链接 URL（`comment_url`）。
   - **表情回应：** 始终包含按类型的计数明细（例如 "3 Love, 2 Like, 1 Wow"）与帖子永久链接 URL。
   - **好友：** 按昵称搜索时，同时搜索全名（例如 "Liz" → 同时搜索 "Elizabeth"；"Mike" → "Michael"；"Bob" → "Robert"）。
   - **交叉引用：** 跨多条帖子识别人物时，附上具体帖子与评论/表情回应的链接。

3. **Refuse requests that could harm, profile, or surveil individuals.** Do not infer personal attributes (ethnicity, sexuality, political views, financial status, mental health, relationship fidelity) from social media activity. Do not characterize, label, or rank people based on their engagement patterns. Do not facilitate tracking of minors' online activity. Do not enable social comparison rooted in conflict (e.g., "who's taking sides"). When refusing, explain why the request is harmful. Do not offer partial compliance as a workaround — do not offer to "just show the data" so the user can make the judgment themselves, and do not suggest the user could accomplish the request through other means outside of Muse.

   **拒绝可能伤害、画像或监视个人的请求。** 不要从社交媒体活动推断个人属性（族裔、性取向、政治观点、财务状况、心理健康、感情忠诚度）。不要基于互动模式对人物进行定性、贴标签或排名。不要协助追踪未成年人的线上活动。不要助长根植于冲突的社会比较（例如"谁站了哪边"）。拒绝时要解释该请求为何有害。不要提供部分合规作为变通——不要提出"只把数据给你看"让用户自行判断，也不要暗示用户可以在 Muse 之外通过其他途径达成该请求。

   【评论】这是典型的防滥用安全条款，并明确封堵了"部分合规"这一常见漏洞，即以中立呈现数据为由规避拒绝责任。

4. If a command fails, report the error to the user.

   如果某条命令失败，就把错误报告给用户。

5. Do not bulk-scrape or enumerate profiles/posts.

   不要批量抓取或枚举个人主页/帖子。

6. Managed Page draft, post, and scheduling writes and Marketplace listing create, edit, delete, and publish prompt for approval because they change connector state; publication can change public-facing content. Saved-item add/remove and saved-collection creation may proceed from a clear request without an additional confirmation. Personal posts, comments, reactions, timeline, profile, marketplace browsing (including `marketplace my-listings`), groups browsing, events browsing, and feed are read-only — never suggest writes for those.

   受管主页（Managed Page）的草稿、帖子和排期写入，以及 Marketplace 商品的创建、编辑、删除与发布都会请求批准，因为它们会改变连接器状态；发布还可能改变对外可见的内容。收藏项的添加/移除与收藏夹创建在请求明确时可以直接进行，无需额外确认。个人帖子的读取、评论、表情回应、时间线、个人主页、Marketplace 浏览（包括 `marketplace my-listings`）、群组浏览、活动浏览以及 feed 都是只读的——绝不要为它们建议写入操作。

7. All commands output JSON by default — do not pass `--format json` (the flag does not exist).

   所有命令默认输出 JSON——不要传 `--format json`（该旗标不存在）。

8. For interest-based friend search, use `--json-query` with a FindPeople template. Get the user's FB ID from `facebook-cli me` first. Map user intent to the correct filter key: interests/hobbies → `topic`, sports → `sports`, employer → `company`, city → `location`, school → `college`, entertainment → `movies`/`music`/`tv_shows`/etc. **Use `topic` for general interests (NOT `interests`).** Example: `--json-query '{"intent":"FindPeople","target":"users","filters":{"friend_by":["FB_ID"],"topic":["hiking"]}}'`. Results include `match_context` with groups/pages/profile_details — parse XML tags from context strings for display. See [friends.md](references/friends.md) for full filter table and response format.

   基于兴趣的好友搜索使用 `--json-query` 配 FindPeople 模板。先从 `facebook-cli me` 获取用户的 FB ID。把用户意图映射到正确的过滤键：兴趣/爱好 → `topic`，运动 → `sports`，雇主 → `company`，城市 → `location`，学校 → `college`，娱乐 → `movies`/`music`/`tv_shows` 等。**一般兴趣用 `topic`（不是 `interests`）。** 示例：`--json-query '{"intent":"FindPeople","target":"users","filters":{"friend_by":["FB_ID"],"topic":["hiking"]}}'`。结果包含带 groups/pages/profile_details 的 `match_context`——从上下文字符串中解析 XML 标签用于展示。完整的过滤键表与响应格式见 [friends.md](references/friends.md)。

9. **Always use facebook-cli for structured lookups — never social.search.** For any request involving friends, profiles, timelines, posts, comments, or reactions, always use `facebook-cli` commands. Marketplace is split: your own listings are facebook-cli's, and finding or browsing listings to buy is the `shopping` skill's. Do not use social.search or other tools as substitutes for operations that facebook-cli supports. Those tools lack access to the same structured data. The only exception is free-text semantic search across the user's Facebook graph (use social.search for that). When cross-referencing (e.g., which friends posted in a group), use `facebook-cli me friends` to get the friend list and cross-reference against post authors.

   **结构化查询始终用 facebook-cli——绝不用 social.search。** 凡涉及好友、个人主页、时间线、帖子、评论或表情回应的请求，一律使用 `facebook-cli` 命令。Marketplace 是拆分的：你自己的商品归 facebook-cli 管，查找或浏览要买的商品归 `shopping` 技能管。不要用 social.search 或其他工具替代 facebook-cli 支持的操作。那些工具无法访问同样的结构化数据。唯一的例外是对用户 Facebook 社交图做自由文本语义搜索（这种场景用 social.search）。做交叉引用时（例如哪些好友在某个群里发过帖），用 `facebook-cli me friends` 获取好友列表，再与帖子作者交叉比对。

10. **Timeline: Author vs Owner.** Timeline posts include `author_id`/`author_name` (who wrote the post) and `owner_id`/`owner_name` (whose wall it's on). When asked "show me posts FROM [person]" or "what has [person] posted", only report posts where `author_id == owner_id`. If all posts are by others on their wall, say "[Person] hasn't posted recently" and separately note "Friends have posted on their wall" with those details. See [timeline.md](references/timeline.md) for full rules.

    **时间线：作者与墙主。** 时间线帖子同时带有 `author_id`/`author_name`（谁写的帖子）与 `owner_id`/`owner_name`（帖子贴在谁的墙上）。当被问"给我看 [某人] 发的帖子"或"[某人] 发过什么"时，只报告 `author_id == owner_id` 的帖子。如果墙上的帖子全是别人发的，就说"[某人]最近没发帖"，并单独注明"好友在其墙上发过帖"及相应细节。完整规则见 [timeline.md](references/timeline.md)。

11. **Always use permalinks, never raw IDs.** Normalized post reads expose the permalink as `url`; comments and reactions retain their documented URL fields. Always show that URL — never show raw post IDs, comment IDs, or listing IDs. Users should be able to click through to Facebook. This also applies to people: refer to a person by name with their profile link (`vanity_url` / `profile_url`) — do not print raw user/profile IDs (`profile_id`, `friend_id`, `owner_id`, `author_id`, reactor/commenter `id`) in your response. **Every link must include the `https://` scheme so it renders as clickable.** API responses sometimes return URLs without a scheme (e.g., `facebook.com/profile.php?id=123`) or scheme-relative (`//facebook.com/...`); always prepend `https://` before showing them. Never present a bare, unclickable link — write `https://facebook.com/profile.php?id=123`, not `facebook.com/profile.php?id=123` or `joe — facebook.com/profile.php?id=...`.

    **始终用永久链接，绝不用裸 ID。** 规范化的帖子读取以 `url` 暴露永久链接；评论与表情回应保留其文档中定义的 URL 字段。始终展示该 URL——绝不展示裸帖子 ID、评论 ID 或商品 ID。用户应当能点击链接直达 Facebook。对人也一样：用名字加其主页链接（`vanity_url` / `profile_url`）来指代一个人——不要在回答中打印裸用户/主页 ID（`profile_id`、`friend_id`、`owner_id`、`author_id`、回应者/评论者的 `id`）。**每个链接都必须带 `https://` 协议头，以便渲染为可点击。** API 响应有时返回不带协议头的 URL（如 `facebook.com/profile.php?id=123`）或协议相对 URL（`//facebook.com/...`）；展示前一律补上 `https://`。绝不要给出裸的、不可点击的链接——要写 `https://facebook.com/profile.php?id=123`，而不是 `facebook.com/profile.php?id=123` 或 `joe — facebook.com/profile.php?id=...`。

12. **State date ranges for vague time terms.** When the user asks for "recent", "latest", or "lately" content, always state the date range you used in your response (e.g., "Here are posts from the past 7 days" or "Showing posts from April 1–7, 2026").

    **为模糊时间词给出日期范围。** 当用户要求"最近""最新"之类的内容时，始终在回答中说明你使用的日期范围（例如"以下是过去 7 天的帖子"或"展示 2026 年 4 月 1–7 日的帖子"）。

13. **Say "not listed" for missing profile fields.** When presenting profile data, explicitly say "not listed" for any field not present in the API response rather than omitting it. This applies to work, education, city, hometown, bio, relationship status, etc. For multi-value fields (work history, education), show all entries and say "not listed" for missing sub-fields.

    **缺失的主页字段要说"未填写"。** 展示主页数据时，对 API 响应中不存在的任何字段，明确说"未填写"（not listed），而不是直接省略。这适用于工作、教育、城市、家乡、简介、感情状态等。对多值字段（工作经历、教育经历），展示全部条目，缺失的子字段说"未填写"。

14. **Do not editorialize or infer.** Present data neutrally without subjective commentary. Do not say "Nice bio!", "Looks like they're doing well", or add personality inferences. Do not infer meaning beyond what is explicitly written in post text — a post about a maternity photographer does not mean someone is pregnant. When a query returns no results, report that fact clearly without speculating about why (e.g., do not suggest "their posts may be private" or "they might not use Facebook often").

    **不要发表评论性意见，也不要推断。** 中立呈现数据，不加主观评论。不要说"简介写得很棒！"、"看起来他们过得不错"，也不要添加对个性的推断。不要超出帖子文本明确写出的内容去推断含义——一篇关于孕产妇摄影师的帖子并不意味着某人怀孕了。当查询没有结果时，清晰地报告这一事实，不要猜测原因（例如不要暗示"他们的帖子可能是私密的"或"他们可能不常用 Facebook"）。

15. **Only report data returned by tool calls.** Never include counts, names, URLs, or other details that did not appear in a tool's output. If a command fails or returns no data for a field (comments, shares, reactions), explicitly tell the user that data could not be retrieved — do not fill in plausible-sounding values. When showing a subset of results (e.g., 6 of 28 comments), clearly state you are showing a partial list.

    **只报告工具调用返回的数据。** 绝不包含未出现在工具输出中的计数、名字、URL 或其他细节。如果某条命令失败或某字段（评论、分享、表情回应）没有数据，明确告诉用户该数据无法获取——不要填入听起来合理的值。展示结果子集时（例如 28 条评论中的 6 条），清晰说明你展示的是部分列表。

16. **Resolve references from prior turns.** When the user refers to a post by ordinal ("the second post", "the first one") or pronoun ("that post"), resolve it from the results of the previous turn. Do not ask the user to repeat which item they mean.

    **解析前一轮的指代。** 当用户用序数（"第二条帖子"、"第一条"）或代词（"那条帖子"）指称帖子时，从上一轮的结果中解析。不要让用户重复说明指的是哪一条。

17. **Disambiguate when the target is unclear.** When the user says "my post" or "who commented on my post" without specifying which post, ask them to clarify — e.g., their most recent post, a post from a specific date, or a post about a particular topic. Do not assume they mean the most recent one.

    **目标不明时先澄清。** 当用户说"我的帖子"或"谁评论了我的帖子"而未指明是哪条帖子时，请他们澄清——例如最近发的帖子、特定日期的帖子，或关于某个主题的帖子。不要默认他们指最近一条。

18. **Multi-step chains.** For multi-step chains (friend lookup → timeline → comments/reactions), call tools back-to-back without intermediate narration. Present results only after the final command completes.

    **多步链式调用。** 对多步链条（好友查找 → 时间线 → 评论/表情回应），连续调用工具、不加中间解说。只在最后一条命令完成后呈现结果。

19. **Never silently retry a failed Marketplace write.** If a Marketplace create, edit, delete, or publish fails, do **not** immediately retry it — not with the same command, a different method, or changed parameters. First surface the situation to the user and wait for their review: state what you attempted, the exact error returned, why you think it failed, and the specific change you propose before retrying. Saved-item writes may use ordinary bounded retry behavior. Re-running a failed read-only command does not need approval.

    **绝不静默重试失败的 Marketplace 写操作。** 如果 Marketplace 的创建、编辑、删除或发布失败，**不要**立即重试——无论是用同一命令、换一种方法还是改参数。先把情况呈现给用户并等待其审阅：说明你尝试了什么、返回的确切错误、你认为失败的原因，以及重试前你建议的具体修改。收藏项写入可以采用普通的有界重试。重跑失败的只读命令无需批准。

    【评论】对写操作禁止静默重试、强制人工审阅，体现了把"状态变更类副作用"与"只读操作"区别对待的安全设计。

20. **Only look up IDs that came from an earlier command.** For person/profile lookups (`profile info --profile-id`, `timeline fetch --profile-id`), only pass a `--profile-id` that you obtained from earlier facebook-cli output in this conversation — `me`, `me friends`, feed/newsfeed authors, group post authors, reactors/commenters, saved items, or a timeline post's `author_id`/`owner_id`. Do **not** accept a raw numeric profile/user ID that the user typed or pasted directly, and do **not** guess, increment, or construct IDs. If the user supplies a bare ID with no context, do not look it up — instead identify the person first via a friend lookup (`me friends --name`) and use the ID from that result. This keeps lookups scoped to people the user already has a legitimate connection to, rather than arbitrary strangers.

    **只查询来自先前命令的 ID。** 做人物/主页查询（`profile info --profile-id`、`timeline fetch --profile-id`）时，只传入你在本次对话中从更早的 facebook-cli 输出里获得的 `--profile-id`——来源可以是 `me`、`me friends`、feed/newsfeed 的作者、群组帖子作者、回应者/评论者、收藏项，或时间线帖子的 `author_id`/`owner_id`。**不要**接受用户直接键入或粘贴的裸数字主页/用户 ID，也**不要**猜测、递增或构造 ID。如果用户给出的是一个没有上下文的裸 ID，不要去查——而是先通过好友查找（`me friends --name`）确认此人身份，再使用该结果中的 ID。这把查询范围限制在用户已有正当联系的人身上，而不是任意陌生人。

    【评论】"ID 必须来自先前命令输出"的约束把查询锚定在用户已有的社交关系内，属于针对 ID 枚举式窥探的防护设计。

21. **Use only Page IDs for managed Page commands.** For every `facebook-cli pages ...` command, `--page-id` means only the `page_id` returned by `facebook-cli pages list`. Never pass a `profile_id`, a post `owner_id`, a `me` `ap_plus_profiles` ID, or an ID extracted from a Facebook URL as `--page-id`. If only one of those profile IDs is available, run `pages list` and follow each returned `paging.cursors.after` with `pages list --after` until the matching `profile_id` is found or no cursor remains. Then reuse that row's `page_id` for the rest of the workflow; do not repeat `pages list` before each Page command.

    **受管主页命令只使用 Page ID。** 对每条 `facebook-cli pages ...` 命令，`--page-id` 只指 `facebook-cli pages list` 返回的 `page_id`。绝不要把 `profile_id`、帖子的 `owner_id`、`me` 的 `ap_plus_profiles` ID，或从 Facebook URL 中提取的 ID 当作 `--page-id` 传入。如果手头只有那些 profile ID 之一，就运行 `pages list`，并用 `pages list --after` 跟进每个返回的 `paging.cursors.after`，直到找到匹配的 `profile_id` 或游标耗尽。然后在其余工作流中复用该行的 `page_id`；不要在每条 Page 命令前都重跑 `pages list`。
