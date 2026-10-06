---
name: "instagram"
description: "Read Instagram profiles, followers, posts, comments, likes, stories, feed, saved content, and account insights. Answer questions about posts, reels, and Instagram links. Manage interests and profile details, and publish stories, reels, posts, or carousels on request."
icon: "instagram"
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Instagram CLI / Instagram CLI 命令行工具

## Purpose / 用途
Read, manage, and answer questions about Instagram data.

读取、管理 Instagram 数据，并回答与 Instagram 数据相关的问题。

## Analyzing posts and reels / 分析帖子与 Reels

Before working with the content of an Instagram post or reel, read [Instagram content questions](references/content-questions.md) with `muse.read`. State what you checked, distinguish guesses from verified facts, and say what you could not verify.

在处理 Instagram 帖子或 Reels 的内容之前，先用 `muse.read` 阅读 [Instagram content questions](references/content-questions.md)。说明你核查了哪些内容，区分猜测与已验证的事实，并指出你无法验证的部分。

## Account Linking / 账号关联

Every `instagram-cli` command except `accounts`, `connect-url`, `disconnect-url`, and `--help` requires `--account-id`. For those commands, use the user's `user_fbid` from an `instagram-cli accounts` result available in your context. If none is available, run `instagram-cli accounts` first. Run `accounts` again after an auth or account error, or when the user says they connected, disconnected, or switched accounts.

除 `accounts`、`connect-url`、`disconnect-url` 和 `--help` 之外，每条 `instagram-cli` 命令都需要 `--account-id`。对于这些命令，请使用上下文中已有的 `instagram-cli accounts` 结果里用户本人的 `user_fbid`；如果没有可用的结果，先运行 `instagram-cli accounts`。在出现认证或账号错误之后，或者当用户表示已连接、已断开或已切换账号时，重新运行 `accounts`。

If several accounts are returned without a selection in the task, or the
selected linked account conflicts with the requested publishing destination,
resolve the account choice before running the affected command. Reuse an
explicit account choice already in the task without asking again. An agent
in live conversation asks the user about an unresolved choice. A subagent
returns an unresolved choice to its parent agent. A detached worker reports
an unresolved choice in its final message. Do not choose an account
arbitrarily.

如果返回了多个账号而任务中没有指定选择，或者所选的关联账号与请求的发布目标冲突，必须先解决账号选择问题，再运行受影响的命令。任务中已有的明确账号选择可直接复用，无需再次询问。实时对话中的代理就未决的选择询问用户；子代理把未决的选择返回给其父代理；独立工作节点在其最终消息中报告未决的选择。不要任意挑选一个账号。

【评论】该段按代理形态（实时对话、子代理、独立工作节点）规定了不同的账号选择上报路径，属于多代理协作中的确定性分派设计。

If a command says Instagram is not connected, check its `--account-id` against
the refreshed `accounts` result. If the selected account is returned with a
different `user_fbid`, retry with that value. Treat Instagram as unlinked only
when `accounts` returns an empty list or the command reports it is not
connected using the selected account's freshly verified `user_fbid`. If unlinked and the task requires an account, run `instagram-cli connect-url` and use its `connect_url` in the message below:

如果某条命令提示 Instagram 未连接，将其 `--account-id` 与刷新后的 `accounts` 结果核对。如果所选账号返回了不同的 `user_fbid`，用该值重试。只有当 `accounts` 返回空列表，或者命令使用所选账号刚验证过的 `user_fbid` 仍报告未连接时，才将 Instagram 视为未关联。如果未关联且任务需要账号，运行 `instagram-cli connect-url`，并在下面的消息中使用其 `connect_url`：

> Your Instagram account is not connected. To connect it, visit ``[Meta Accounts Center](`connect_url`)`` and link your Instagram account.

> 你的 Instagram 账号尚未连接。如需连接，请访问 ``[Meta Accounts Center](`connect_url`)`` 并关联你的 Instagram 账号。

A link to a post or reel does not need the user's account. Follow Instagram content questions for it.

处理帖子或 Reels 链接不需要用户的账号，此时仍须遵循 Instagram content questions 的指引。

Reading a public profile does not require connecting Instagram. If no account
is linked, use web search or a browser task for public-profile questions.
If browser inspection is still needed, use `browser.spawn_task` when
available. A subagent returns the known profile URL or username and
unresolved question to its parent agent only if browser inspection is still
needed and `browser.spawn_task` is unavailable.
The native `profile`, `user-profile`, and `post` commands still require a
linked account. Do not send a connect link for a public-only task unless the
user asks to connect.

读取公开主页不需要连接 Instagram。如果没有关联任何账号，公开主页相关的问题可改用网页搜索或浏览器任务处理。如果仍需浏览器检查，在可用时使用 `browser.spawn_task`。只有当仍需浏览器检查且 `browser.spawn_task` 不可用时，子代理才把已知的资料主页 URL 或用户名连同未决问题返回给其父代理。原生的 `profile`、`user-profile` 和 `post` 命令仍要求已关联账号。除非用户主动要求连接，否则不要为纯公开数据任务发送连接链接。

If the user asks to disconnect Instagram, run `instagram-cli disconnect-url` and send the user its `disconnect_url`:

如果用户要求断开 Instagram，运行 `instagram-cli disconnect-url`，并把其中的 `disconnect_url` 发送给用户：

> To disconnect your Instagram account, visit ``[Meta Accounts Center](`disconnect_url`)`` and remove the linked account.

> 如需断开你的 Instagram 账号，请访问 ``[Meta Accounts Center](`disconnect_url`)`` 并移除已关联的账号。

Use the URL returned by the corresponding command. Do not hardcode it.

使用对应命令返回的 URL，不要硬编码。

## Related skills / 相关技能
- Use the `instagram_messages` skill for inbox, thread, and message reads, DM search and sending, and top recipients. Follow its separate messaging connection and sending rules.
  收件箱、会话与消息的读取、私信搜索与发送、最常联系人等场景，请使用 `instagram_messages` 技能，并遵循其单独的消息连接与发送规则。
- Use `social.search` for general post discovery.
  一般性的帖子发现请使用 `social.search`。
- Use this skill for direct `account-insights` reports and for explaining or rewriting supplied metric values. Use `social_content_performance` for performance comparisons, trends, or content recommendations for the user's own accounts and posts. Use `social_competitor_analysis` for competitor, peer, or category comparisons. When every account in a performance comparison belongs to the user, use `social_content_performance`.
  直接的 `account-insights` 报告，以及解释或改写已提供的指标数值，请使用本技能。针对用户自己账号和帖子的表现对比、趋势或内容建议，请使用 `social_content_performance`；竞品、同行或品类对比，请使用 `social_competitor_analysis`。当表现对比中的所有账号都属于用户本人时，使用 `social_content_performance`。

If `social_content_performance` or `social_competitor_analysis` is unavailable,
use this skill's documented reads for the parts of the request they support.
Use `account-insights` only for the user's linked accounts. For competitors,
use public `profile` and `posts` data under the Account Linking rules. Do not
substitute the user's account totals for competitor metrics or invent missing
values. State which requested metrics or comparisons remain unavailable.

如果 `social_content_performance` 或 `social_competitor_analysis` 不可用，就用本技能已有的读取能力覆盖请求中它们支持的部分。`account-insights` 只能用于用户已关联的账号；对于竞品，应在账号关联规则的约束下使用公开的 `profile` 和 `posts` 数据。不要用用户自己的账号总量替代竞品指标，也不要编造缺失的数值。须说明哪些请求的指标或对比仍无法提供。

## Running commands / 运行命令

Run `instagram-cli` commands with `exec`.

使用 `exec` 运行 `instagram-cli` 命令。

Use flag names exactly as documented. Unknown flags can be ignored without
an error.

标志（flag）名称必须与文档完全一致。未知的标志可以直接忽略，不报错误。

For `fetch-post-comments` and `fetch-post-likers`, use `--post-ids` even for one
post. For `save-post`, `unsave-post`, and `media-understanding`, use
`--media-ids` even for one item. The singular flags `--post-id` and `--media-id`
are unsupported.

对于 `fetch-post-comments` 和 `fetch-post-likers`，即使只处理一个帖子也要使用 `--post-ids`。对于 `save-post`、`unsave-post` 和 `media-understanding`，即使只处理一个条目也要使用 `--media-ids`。单数形式的 `--post-id` 和 `--media-id` 不受支持。

Reuse results already in your context. When a script will process a result,
redirect the first call's output to a file under `~/workspace/` and read that
file. Do not rerun a successful command with the same arguments in the same
task.

优先复用上下文中已有的结果。当脚本要处理某个结果时，把第一次调用的输出重定向到 `~/workspace/` 下的文件，再读取该文件。同一任务中不要用相同参数重复运行已成功的命令。

The Instagram API enforces rate limits. If a request returns
`429 Too Many Requests`, stop all `instagram-cli` calls for this task, including
calls issued by scripts. For a post or reel link, continue with the fallbacks in
Instagram content questions. Otherwise, report what you completed and what
remains.

Instagram API 实施限流。如果某个请求返回 `429 Too Many Requests`，立即停止本任务的所有 `instagram-cli` 调用，包括脚本发出的调用。对于帖子或 Reels 链接，可按 Instagram content questions 中的回退方案继续；其他情况则报告已完成的内容与剩余的部分。

【评论】把 429 视为任务级熔断信号而非单次重试提示，是较为保守的限流处理策略，可降低账号被进一步限制的风险。

In scripts, check the exit status of each `instagram-cli` call. Stop at the
first 429, and do not treat a failed call as an empty result.

在脚本中检查每条 `instagram-cli` 调用的退出状态。遇到第一个 429 即停止，且不得把失败的调用当作空结果处理。

If a bulk read repeatedly returns HTTP 500, stop issuing calls for that bulk
read, including from scripts. An HTTP 500 alone does not identify a rate limit.
State how much you read and what remains.

如果某次批量读取反复返回 HTTP 500，停止发起该批量读取的调用，包括来自脚本的调用。仅凭 HTTP 500 不能判定为限流。须说明已读取的量与剩余的量。

For requests covering an entire list, fetch every page while pagination makes
progress. Stop before requesting a cursor already used for that list. Also
stop when a page reports more pages but adds no new items. Apply these checks
inside pagination scripts. If you stop before completing the requested scope,
state how much you read and what remains.

对于覆盖整个列表的请求，在分页仍有进展时持续抓取每一页。不要请求该列表已使用过的游标；当某页报告还有更多页却没有新增条目时也应停止。这些检查要写入分页脚本内部。如果未完成请求范围就停止，须说明已读取的量与剩余的量。

When a request needs a separate `user-profile`, `profile`, or `post` call for
each of more than 50 accounts or posts, start with the 50 most recent. Stop
after those 50. Report their results and how many remain for a later batch.
Instagram rate-limits these calls after a few dozen.

当请求需要对超过 50 个账号或帖子逐一单独调用 `user-profile`、`profile` 或 `post` 时，先处理最近的 50 个，之后停下，报告这 50 个的结果以及留待下一批处理的数量。Instagram 在几十次调用后就会对这类调用限流。

Perform write actions only when the user explicitly asks for the change.
This includes interest updates, publishing, and profile changes.

只在用户明确要求更改时执行写操作，包括兴趣更新、发布和资料修改。

`instagram-cli` does not support following, unfollowing, removing followers,
blocking, liking, commenting, replying to comments, deleting posts, archiving
posts, pinning posts, or editing captions.

`instagram-cli` 不支持关注、取关、移除粉丝、拉黑、点赞、评论、回复评论、删除帖子、归档帖子、置顶帖子或编辑说明文字。

### Post output / 帖子输出

`posts`, `feed`, and `post` return the same compact post collection used by
`social.search`. Read `posts[]` directly. Do not guess a provider GraphQL path or
pipe these commands through `jq`:

`posts`、`feed` 和 `post` 返回与 `social.search` 相同的紧凑帖子集合。直接读取 `posts[]`，不要猜测服务端的 GraphQL 路径，也不要把这些命令通过 `jq` 管道处理：

```json
{
  "format": "social_posts_v1",
  "count": 1,
  "posts": [{
    "rank": 1,
    "post_id": "...",
    "url": "https://www.instagram.com/...",
    "platform": "instagram",
    "post_created_at": {"utc":"...","user_local":"...","user_timezone":"..."},
    "username": "account",
    "post_caption": "..."
  }],
  "next_cursor": "...",
  "has_next_page": true
}
```

Prefer `post_created_at.user_local` when presenting a post time. The response
retains `created_at` for compatibility. Fields unavailable in the provider
response are omitted. A feed item can omit `url` or `created_at`. Before
quoting, dating, or linking a selected feed item, run `post --id <post_id>` and use
the returned record. A valid partial response includes the provider's reason in
optional `provider_error`. Do not describe it as a complete page. If required
post data is absent, the CLI fails and includes that reason when the provider
supplied one. Without a provider reason, it reports the schema mismatch rather
than returning an empty post list.

展示帖子时间时优先使用 `post_created_at.user_local`；响应中保留 `created_at` 仅为兼容目的。服务端响应中不可得的字段会被省略，feed 条目可能缺少 `url` 或 `created_at`。在引用、标注日期或链接某个选定的 feed 条目之前，先运行 `post --id <post_id>` 并使用返回的记录。有效的部分响应会在可选的 `provider_error` 中给出服务端原因，不要将其描述为完整的一页。如果所需的帖子数据缺失，CLI 会失败，并在服务端给出原因时附带该原因；若服务端未给出原因，CLI 会报告模式不匹配，而不是返回空的帖子列表。

Post rows can include the author's `username`, `author_name`, `author_bio`,
`follower_count`, and `verified`. Use these returned fields instead of a
separate profile lookup for the same information, unless the request needs
missing or refreshed profile information.

帖子行可以包含作者的 `username`、`author_name`、`author_bio`、`follower_count` 和 `verified`。除非请求需要缺失或需要刷新的资料信息，否则应直接使用这些返回字段获取同类信息，而不必单独查询资料。

The `posts`, `feed`, and `post` responses can include per-post counts in
`likes` and `comments`, but do not expose per-post views, reach, saves, or
shares. Do not assign account-level totals to individual posts.

`posts`、`feed` 和 `post` 的响应可以包含按帖子计的 `likes` 和 `comments` 计数，但不提供按帖子的浏览、触达、收藏或分享数据。不要把账号层面的总量套用到单个帖子上。

### Global options / 全局选项
- `--account-id <user_own_fbid>` selects the account. Use the user's own `user_fbid` from Account Linking. Do not pass another user's ID.
  `--account-id <user_own_fbid>` 用于选择账号。使用账号关联一节中用户本人的 `user_fbid`，不要传入其他用户的 ID。
- `--retries <N>` retries transient failures. The default is 0. Nonzero values are unsupported for `post-story`, `post-feed`, `set-profile-picture`, and `profile-banner`.
  `--retries <N>` 对瞬时失败进行重试，默认值为 0。对于 `post-story`、`post-feed`、`set-profile-picture` 和 `profile-banner`，不支持非零值。
- `--after <cursor>` supplies the pagination cursor from the previous response, except for `own-stories-archive`, which uses `--max-id`. Omit the cursor to fetch the first page.
  `--after <cursor>` 提供上一次响应中的分页游标；`own-stories-archive` 例外，它使用 `--max-id`。省略游标即抓取第一页。

For `fetch-post-comments` and `fetch-post-likers`, pagination is per
`post_groups[]` entry. To fetch another page for an entry with `has_next_page`
set to true, pass only its `media_id` as `--post-ids` and its `end_cursor` as  
`--after`.

对于 `fetch-post-comments` 和 `fetch-post-likers`，分页按 `post_groups[]` 条目进行。当某条目的 `has_next_page` 为 true 时，若要抓取该条目的下一页，只把它的 `media_id` 作为 `--post-ids` 传入，并把它的 `end_cursor` 作为 `--after` 传入。

## Commands / 命令

Available commands:

可用命令：

- `accounts`
- `connect-url`
- `disconnect-url`
- `profile`
- `current-interests`
- `posts`
- `followers`
- `following`
- `close-friends`
- `feed`
- `activity-notifications`
- `user-profile`
- `tagged-posts`
- `post`
- `fetch-post-comments`
- `fetch-post-likers`
- `saved-posts`
- `saved-collections`
- `create-saved-collection`
- `rename-saved-collection`
- `save-post`
- `unsave-post`
- `own-stories`
- `own-stories-archive`
- `stories-tray`
- `story-media`
- `location-search`
- `media-understanding`
- `account-insights`
- `recently-liked-posts`
- `recently-commented-posts`
- `update-interests`
- `post-feed`
- `post-story`
- `set-profile-picture`
- `update-bio`
- `profile-banner`

### Accounts / 账号列表
List the Instagram accounts linked to the user.

列出已与用户关联的 Instagram 账号。

```sh
instagram-cli accounts
```

### Profile / 资料主页
Fetch the Instagram profile bio and follower/following counts. Omit `--username` and `--profile-url` to fetch the user's own profile. Otherwise, provide a username or profile URL to fetch another user's profile. When only counts are needed, use this command instead of fetching the follower or following lists.

获取 Instagram 资料主页的简介及粉丝/关注计数。省略 `--username` 和 `--profile-url` 即获取用户本人的资料；否则提供用户名或主页 URL，以获取其他用户的资料。当只需要计数时，使用本命令而非抓取粉丝或关注列表。

When using `--username`, pass one username per call. Read the result from `profiles[]`, including
`bio` and `website`. For `profile` and `user-profile`, an empty `profiles` list
alone does not establish that a handle is available or reserved, or that the
tool failed.

使用 `--username` 时，每次调用只传一个用户名。从 `profiles[]` 读取结果，包括 `bio` 和 `website`。对于 `profile` 和 `user-profile`，仅凭空的 `profiles` 列表不能证明某个句柄可用或已被占用，也不能证明工具失败。

```sh
instagram-cli profile --account-id <user_own_fbid>
instagram-cli profile --account-id <user_own_fbid> --username <username>
instagram-cli profile --account-id <user_own_fbid> --username @<username>
instagram-cli profile --account-id <user_own_fbid> --profile-url https://instagram.com/<username>
```

### Update bio / 更新简介
Update the user's Instagram profile bio. `--bio` is required and may contain at
most 150 characters. Pass an empty string to clear the bio.

更新用户的 Instagram 资料简介。`--bio` 为必填，最多 150 个字符；传入空字符串即可清空简介。

```sh
instagram-cli update-bio --account-id <user_own_fbid> --bio "<bio>"
```

### Muse profile banner / Muse 资料横幅
Add or remove the user's Muse banner on their Instagram profile. This shows as a
pill on the user's profile beneath their bio. This does not change their profile
picture or bio.

在用户的 Instagram 资料页上添加或移除 Muse 横幅。它以胶囊样式显示在用户资料页简介下方，不会更改用户的头像或简介。

```sh
instagram-cli profile-banner --account-id <user_own_fbid> --action add
instagram-cli profile-banner --account-id <user_own_fbid> --action remove
```

Do not supply media or a profile URL. Report completion only when the result
contains `success: true`; `success: false` is not a completed update. Automatic
retries are disabled; after an ambiguous failure, do not claim success or
silently repeat the mutation.

不要提供媒体文件或主页 URL。只有当结果包含 `success: true` 时才报告完成；`success: false` 不算完成更新。自动重试已被禁用；在结果不明确的失败之后，不要声称成功，也不要悄悄重复该变更操作。

### Current interests / 当前兴趣
Fetch topics the user is interested in or uninterested in. Show the topic and its type without distinguishing inferred from explicit interests.  

获取用户感兴趣或不感兴趣的主题。展示主题及其类型时不区分推断兴趣与显式兴趣。  

```sh
instagram-cli current-interests --account-id <user_own_fbid>
```

### Update interests / 更新兴趣
Mark a topic as "interested" so the user sees more of it on Instagram, or as "not interested" so the user sees less of it.

将某个主题标记为"interested"（感兴趣），让用户在 Instagram 上看到更多相关内容；或标记为"not interested"（不感兴趣），让用户看到更少相关内容。

【评论】该命令提供对 Instagram 兴趣画像（推断兴趣与显式兴趣）的直接管理接口，这类画像通常也是内容与广告个性化的输入。

```sh
instagram-cli update-interests --account-id <user_own_fbid> --text "<topic>" --interested true
instagram-cli update-interests --account-id <user_own_fbid> --text "<topic>" --interested false
```

### Posts / 帖子
Omit `--username` and `--user-id` to fetch the user's own posts. Otherwise, provide one or more usernames or one or more user IDs to fetch other users' posts. Do not mix usernames and user IDs in the same command.  
Pass the previous response's `next_cursor` as `--after`. Omit `--after` on the first request.  
Narrow results with `--since`, `--until`, `--sort-order asc|desc`, `--limit`, and repeated or comma-separated `--post-types` values such as `POST`, `REEL`, `STORY`, or `HIGHLIGHT`.

省略 `--username` 和 `--user-id` 即抓取用户本人的帖子；否则提供一个或多个用户名或一个或多个用户 ID，以抓取其他用户的帖子。同一条命令中不要混用用户名与用户 ID。  
将上一次响应的 `next_cursor` 作为 `--after` 传入；首次请求省略 `--after`。  
使用 `--since`、`--until`、`--sort-order asc|desc`、`--limit` 以及重复或逗号分隔的 `--post-types` 值（如 `POST`、`REEL`、`STORY` 或 `HIGHLIGHT`）缩小结果范围。

The response can contain more posts than `--limit` requests. Attribute each
post to its returned `username`, which can differ from the requested account.
When the request is for posts and reels only, pass `--post-types POST,REEL`.
Do not infer profile-grid order or pin status from the order of returned
posts.

响应包含的帖子数可能多于 `--limit` 请求的数量。每条帖子按其返回的 `username` 归属，该用户名可能与请求的账号不同。当请求只针对帖子和 Reels 时，传入 `--post-types POST,REEL`。不要根据返回帖子的顺序推断主页九宫格顺序或置顶状态。

```sh
instagram-cli posts --account-id <user_own_fbid>
instagram-cli posts --account-id <user_own_fbid> --after <cursor>
instagram-cli posts --account-id <user_own_fbid> --username <username>
instagram-cli posts --account-id <user_own_fbid> --username <username>,<username>
instagram-cli posts --account-id <user_own_fbid> --user-id <FBID>
instagram-cli posts --account-id <user_own_fbid> --user-id <FBID>,<FBID>
instagram-cli posts --account-id <user_own_fbid> --username <username> --since 2026-01-01 --until 2026-02-01 --sort-order asc --limit 25 --post-types POST,REEL
```

### Followers / 粉丝
Fetch the user's own follower list. `--count` is optional and defaults to `200`.
Read the returned `users[]` list. To fetch another page when `has_more` is
true, pass the previous response's `next_max_id` as `--after`. Omit `--after`
on the first request.

抓取用户本人的粉丝列表。`--count` 可选，默认为 `200`。读取返回的 `users[]` 列表。当 `has_more` 为 true 时，将上一次响应的 `next_max_id` 作为 `--after` 传入以抓取下一页；首次请求省略 `--after`。

For `followers` and `following`, `users[]` entries include `id`, `username`,
`full_name`, and `profile_pic_url`. Compare list membership by `id` without
per-account profile lookups.

对于 `followers` 和 `following`，`users[]` 条目包含 `id`、`username`、`full_name` 和 `profile_pic_url`。比较列表成员关系时按 `id` 判断，无需逐个查询资料。

```sh
instagram-cli followers --account-id <user_own_fbid>
instagram-cli followers --account-id <user_own_fbid> --count 25
instagram-cli followers --account-id <user_own_fbid> --after <cursor>
```

### Following / 关注
Fetch the user's own following list. Other users' following lists are not supported. `--count` is optional and defaults to `200`.  
Read the returned `users[]` list. To fetch another page when
`page_info.has_next_page` is true, pass the previous response's
`page_info.end_cursor` as `--after`. Omit `--after` on the first request.

抓取用户本人的关注列表，不支持获取其他用户的关注列表。`--count` 可选，默认为 `200`。  
读取返回的 `users[]` 列表。当 `page_info.has_next_page` 为 true 时，将上一次响应的 `page_info.end_cursor` 作为 `--after` 传入以抓取下一页；首次请求省略 `--after`。

```sh
instagram-cli following --account-id <user_own_fbid>
instagram-cli following --account-id <user_own_fbid> --count 25
instagram-cli following --account-id <user_own_fbid> --after <cursor>
```

### Close friends / 亲密朋友
Fetch the user's current Instagram close friends list.

获取用户当前的 Instagram 亲密朋友（Close Friends）列表。

This is the user's own curated Close Friends list on Instagram. It is not ranked or inferred. To get an inferred list of people the user interacts with most, use the `top-recipients` command from the `instagram_messages` skill instead.

这是用户自己在 Instagram 上手动维护的亲密朋友列表，并非排序或推断的结果。若要获取与用户互动最多的人的推断列表，请改用 `instagram_messages` 技能中的 `top-recipients` 命令。

```sh
instagram-cli close-friends --account-id <user_own_fbid>
```

### Feed / 动态流
Following feed:  

关注流：  

```sh
instagram-cli feed --account-id <user_own_fbid> --variant following
```

Close-friends feed:  

亲密朋友流：  

```sh
instagram-cli feed --account-id <user_own_fbid> --variant favorites
```

`--variant` is required. Its supported values are `following` and `favorites`.
The ranked home timeline is not readable through this surface. Omitting
`--variant` returns an error.

`--variant` 为必填，支持的取值为 `following` 和 `favorites`。经排序的主页时间线无法通过此接口读取；省略 `--variant` 会返回错误。

For all feed variants, pass the previous response's `next_cursor` as `--after`. Omit `--after` on the first request.

对所有 feed 变体，都将上一次响应的 `next_cursor` 作为 `--after` 传入；首次请求省略 `--after`。

```sh
instagram-cli feed --account-id <user_own_fbid> --variant following --after <cursor>
```

### Activity notifications / 动态通知
Fetch `notifications` and `priority_notifications` without marking them seen.
Paginate with `page_info.end_cursor` as `--after`.

获取 `notifications` 和 `priority_notifications`，且不会将其标记为已读。分页时将 `page_info.end_cursor` 作为 `--after` 传入。

```sh
instagram-cli activity-notifications --account-id <user_own_fbid> [--after <cursor>]
```

### Other user's profile / 其他用户的资料
Fetch another user's profile by user ID, username, or profile URL. Prefer `--user-id` when you already have a user FBID from prior tool output.  
When using `--username`, pass one username per call. Read the result from `profiles[]`, including
`bio` and `website`.

通过用户 ID、用户名或主页 URL 获取其他用户的资料。如果此前的工具输出中已有用户 FBID，优先使用 `--user-id`。  
使用 `--username` 时，每次调用只传一个用户名。从 `profiles[]` 读取结果，包括 `bio` 和 `website`。

```sh
instagram-cli user-profile --account-id <user_own_fbid> --user-id <user_fbid>
instagram-cli user-profile --account-id <user_own_fbid> --username @<username>
instagram-cli user-profile --account-id <user_own_fbid> --profile-url https://instagram.com/<username>
```

### Tagged posts / 被标记的帖子
Fetch posts a user has been tagged in. Omit `--user-id` to fetch the user's own tagged posts. Otherwise, pass the target user's FBID as `--user-id`.  
Narrow results with `--since`, `--until`, `--sort-order`, and `--limit`.

抓取某用户被标记其中的帖子。省略 `--user-id` 即抓取用户本人被标记的帖子；否则将目标用户的 FBID 作为 `--user-id` 传入。  
使用 `--since`、`--until`、`--sort-order` 和 `--limit` 缩小结果范围。

```sh
instagram-cli tagged-posts --account-id <user_own_fbid>
instagram-cli tagged-posts --account-id <user_own_fbid> --after <cursor>
instagram-cli tagged-posts --account-id <user_own_fbid> --user-id <user_fbid>
instagram-cli tagged-posts --account-id <user_own_fbid> --user-id <user_fbid>,<user_fbid> --since 2026-01-01 --until 2026-02-01 --sort-order asc --limit 25
```

### Post by ID or URL / 按 ID 或 URL 获取帖子
Fetch a single post by its media ID or full Instagram post/reel URL. Provide
exactly one of `--id` or `--url`.

通过媒体 ID 或完整的 Instagram 帖子/Reels URL 获取单条帖子。`--id` 与 `--url` 必须且只能提供一个。

```sh
instagram-cli post --account-id <user_own_fbid> --id <media_id>_<owner_id>
instagram-cli post --account-id <user_own_fbid> --id <media_id>
instagram-cli post --account-id <user_own_fbid> --url https://www.instagram.com/p/<shortcode>/
instagram-cli post --account-id <user_own_fbid> --url https://www.instagram.com/reels/<shortcode>
```

### Post comments / 帖子评论
Fetch comments for one or more posts by media ID. Batch IDs in a comma-separated `--post-ids` list. Narrow results with `--since`, `--until`, `--sort-order`, `--limit`, and repeated or comma-separated `--author-ids`.  
Read comment text from `post_groups[].comments[].comment_text`.

按媒体 ID 获取一个或多个帖子的评论。多个 ID 以逗号分隔放入 `--post-ids` 列表。使用 `--since`、`--until`、`--sort-order`、`--limit` 以及重复或逗号分隔的 `--author-ids` 缩小结果范围。  
评论正文从 `post_groups[].comments[].comment_text` 读取。

```sh
instagram-cli fetch-post-comments --account-id <user_own_fbid> --post-ids <media_id>
instagram-cli fetch-post-comments --account-id <user_own_fbid> --post-ids <media_id>,<media_id>
instagram-cli fetch-post-comments --account-id <user_own_fbid> --post-ids <media_id> --limit 25 --after <cursor>
instagram-cli fetch-post-comments --account-id <user_own_fbid> --post-ids <media_id> --since 2026-01-01 --until 2026-02-01 --author-ids <user_fbid>
```

### Post likers / 帖子点赞者
Fetch likers for one or more posts by media ID. Batch IDs in a comma-separated `--post-ids` list. Narrow results with `--since`, `--until`, `--sort-order`, `--limit`, and repeated or comma-separated `--reactor-ids`.  
Read the likers from `post_groups[].reactors[]`.

按媒体 ID 获取一个或多个帖子的点赞用户。多个 ID 以逗号分隔放入 `--post-ids` 列表。使用 `--since`、`--until`、`--sort-order`、`--limit` 以及重复或逗号分隔的 `--reactor-ids` 缩小结果范围。  
点赞用户从 `post_groups[].reactors[]` 读取。

```sh
instagram-cli fetch-post-likers --account-id <user_own_fbid> --post-ids <media_id>
instagram-cli fetch-post-likers --account-id <user_own_fbid> --post-ids <media_id>,<media_id>
instagram-cli fetch-post-likers --account-id <user_own_fbid> --post-ids <media_id> --limit 25 --after <cursor>
instagram-cli fetch-post-likers --account-id <user_own_fbid> --post-ids <media_id> --since 2026-01-01 --until 2026-02-01 --reactor-ids <user_fbid>
```

### Saved posts / 收藏的帖子
Fetch the user's saved posts. To fetch the posts in one collection, pass `--collection-id` with a `collection_id` from `saved-collections`.  
Without `--collection-id`, `saved-posts` supports `--since`, `--until`, `--sort-order`, and `--limit`.

抓取用户收藏的帖子。要抓取某个收藏合集内的帖子，将 `saved-collections` 返回的 `collection_id` 作为 `--collection-id` 传入。  
不传 `--collection-id` 时，`saved-posts` 支持 `--since`、`--until`、`--sort-order` 和 `--limit`。

```sh
instagram-cli saved-posts --account-id <user_own_fbid>
instagram-cli saved-posts --account-id <user_own_fbid> --after <cursor>
instagram-cli saved-posts --account-id <user_own_fbid> --since 2026-01-01 --until 2026-02-01 --sort-order asc --limit 25
instagram-cli saved-posts --account-id <user_own_fbid> --after <cursor> --collection-id "<collection_id>"
```

### Saved collections / 收藏合集
Fetch the user's saved collections (folders/categories of saved posts).

获取用户的收藏合集（已收藏帖子的文件夹/分类）。

```sh
instagram-cli saved-collections --account-id <user_own_fbid>
instagram-cli saved-collections --account-id <user_own_fbid> --after <cursor>
```

### Manage saved posts and collections / 管理收藏的帖子与合集

```sh
instagram-cli create-saved-collection --account-id <user_own_fbid> --name "<name>"
instagram-cli rename-saved-collection --account-id <user_own_fbid> --collection-id <collection_id> --name "<name>"

instagram-cli save-post --account-id <user_own_fbid> --media-ids <media_id>
instagram-cli save-post --account-id <user_own_fbid> --media-ids <media_id>,<media_id> --collection-id <collection_id>

instagram-cli unsave-post --account-id <user_own_fbid> --media-ids <media_id> --collection-id <collection_id>
instagram-cli unsave-post --account-id <user_own_fbid> --media-ids <media_id>
```

Saving with a collection both saves the post and adds it to that collection. Unsaving with a collection only removes it from that collection. Omitting `--collection-id` unsaves it globally.

带合集的收藏会同时收藏帖子并将其加入该合集；带合集的取消收藏只会把帖子移出该合集。省略 `--collection-id` 则全局取消收藏。

### Own stories / 自己的快拍
Fetch the user's own currently posted Instagram stories.

获取用户当前已发布的 Instagram 快拍（stories）。

```sh
instagram-cli own-stories --account-id <user_own_fbid>
```

### Stories archive / 快拍存档
Fetch the user's own Instagram story archive. Use the previous response's `next_max_id` as `--max-id` to paginate. Omit `--max-id` on the first request. When an item has a non-empty video URL, use it instead of the image URL.

获取用户本人的 Instagram 快拍存档。分页时将上一次响应的 `next_max_id` 作为 `--max-id` 传入；首次请求省略 `--max-id`。当条目带有非空的视频 URL 时，使用视频 URL 而非图片 URL。

```sh
instagram-cli own-stories-archive --account-id <user_own_fbid>
instagram-cli own-stories-archive --account-id <user_own_fbid> --max-id <cursor>
```

### Stories tray / 快拍托盘
Fetch the user's stories tray, containing available stories from accounts the user follows. `--count` is optional and defaults to `200`. The response contains only media IDs. To get the permalink for a specific story, pass its ID to `story-media`.

获取用户的快拍托盘，其中包含用户所关注账号的可用快拍。`--count` 可选，默认为 `200`。响应只包含媒体 ID。要获取某条快拍的永久链接，将其 ID 传给 `story-media`。

```sh
instagram-cli stories-tray --account-id <user_own_fbid>
instagram-cli stories-tray --account-id <user_own_fbid> --count 25
```

Use `story-media` only for stories the user explicitly requested. Do not fetch media for every story in the tray by default.

仅对用户明确请求的快拍使用 `story-media`；默认不要为托盘中的每条快拍抓取媒体。

### Story media / 快拍媒体
Fetch media metadata for one or more story items by ID, including the permalink and posting time. Batch IDs in a comma-separated `--ids` list.

按 ID 获取一个或多个快拍条目的媒体元数据，包括永久链接和发布时间。多个 ID 以逗号分隔放入 `--ids` 列表。

```sh
instagram-cli story-media --account-id <user_own_fbid> --ids <story_id>
instagram-cli story-media --account-id <user_own_fbid> --ids <story_id>,<story_id>
```

### Location search / 位置搜索
Search Instagram locations with required `--search-query`. Optionally pass
both `--latitude` and `--longitude` to narrow the results.

使用必填的 `--search-query` 搜索 Instagram 位置。可选地同时传入 `--latitude` 和 `--longitude` 以缩小结果范围。

```sh
instagram-cli location-search --account-id <user_own_fbid> --search-query "<query>"
instagram-cli location-search --account-id <user_own_fbid> \
  --search-query "<query>" --latitude <latitude> --longitude <longitude>
```

### Media files for publishing / 用于发布的媒体文件
Media and cover files for `post-story`, `post-feed`, and `set-profile-picture`
must be under `~/workspace/`. Before passing a file from outside
`~/workspace/`, copy it into `~/workspace/`. Do not use a `/tmp` path.

`post-story`、`post-feed` 和 `set-profile-picture` 所用的媒体与封面文件必须位于 `~/workspace/` 之下。传入来自 `~/workspace/` 之外的文件前，先将其复制到 `~/workspace/`。不要使用 `/tmp` 路径。

### Publish story / 发布快拍
Publish one JPEG, PNG, static WebP, MP4, or MOV file, preferably vertical 9:16
at 1080 x 1920 px. Do not letterbox the media. `--file` is required and
limited to 100 MB. Directly posted videos require a matching JPEG, PNG, or
static WebP `--cover`. Videos edited with stickers or text generate their
cover automatically.

发布一个 JPEG、PNG、静态 WebP、MP4 或 MOV 文件，最好是 1080 x 1920 像素的竖版 9:16。不要为媒体加黑边（letterbox）。`--file` 为必填，上限 100 MB。直接发布的视频需要配套的 JPEG、PNG 或静态 WebP `--cover`；经贴纸或文字编辑过的视频会自动生成封面。

Use `post-story --draft` with `--file` and `--editor-json` for all visible
Story text and supported native stickers. First run
`instagram-cli post-story --help` and follow its detailed parameter, text,
sticker, geometry, and media contract. Draft mode renders under
`~/workspace/instagram/stories/` without posting. Render revisions from the
same clean media. Record every returned draft output and cover path.

所有可见的快拍文字与受支持的原生贴纸，都应使用 `post-story --draft` 配合 `--file` 和 `--editor-json` 完成。先运行 `instagram-cli post-story --help`，并遵循其详细的参数、文字、贴纸、几何布局与媒体约定。草稿模式只在 `~/workspace/instagram/stories/` 下渲染，不会发布。修订版本应基于同一份未修改的原始媒体重新渲染。记录每次返回的草稿输出与封面路径。

Edited Stories using `--editor-json` require user approval of the rendered draft
before publishing. If approval is pending, an agent in live conversation shows
the preview inline and asks for confirmation, a subagent returns the preview to
its parent agent, and a detached worker returns the preview through normal result
delivery. Publish an edited Story from a delegated or detached task only when
that task carries the user's approval of the rendered draft.

使用 `--editor-json` 编辑的快拍在发布前需要用户批准渲染出的草稿。若批准尚未完成：实时对话中的代理会内联展示预览并请求确认；子代理把预览返回给其父代理；独立工作节点通过常规结果交付通道返回预览。只有当委派任务或独立任务携带用户对该渲染草稿的批准时，才能从这些任务中发布编辑后的快拍。

【评论】将"用户批准渲染稿"设为发布前置条件，并按代理形态规定批准信息的传递路径，是把人类审批环节嵌入多代理任务链的一种机制。

After approval, rerun the same command without `--draft`. It renders the approved
edit in `~/workspace/instagram/stories/` and posts that output.
Once publishing succeeds, delete every earlier draft and draft cover by its
exact recorded path. Keep the last user-approved draft and the final published
render. Do not use a glob.

批准后，去掉 `--draft` 重新运行同一条命令；它会在 `~/workspace/instagram/stories/` 下渲染已批准的编辑并发布该输出。发布成功后，按先前记录的精确路径删除所有更早的草稿与草稿封面，保留最后一次经用户批准的草稿和最终发布的渲染结果。不要使用通配符。

Only mention, location, and link native stickers are supported. Refuse other
native sticker types. For a location sticker, select a result with
`location-search` and include its `location_id` in the location object passed
to `--editor-json`.
Mention objects require `user_fbid`. Link objects require `url`.

仅支持提及（mention）、位置（location）和链接（link）三类原生贴纸，遇到其他原生贴纸类型应拒绝。位置贴纸需先用 `location-search` 选定一个结果，并把它的 `location_id` 写入传给 `--editor-json` 的位置对象。提及对象需要 `user_fbid`；链接对象需要 `url`。

The final command automatically uploads the rendered output and its video cover:

最终命令会自动上传渲染结果及其视频封面：

```sh
EDITOR_JSON='[
  {"type":"text","text":"<text>","style":"poster","x":0.5,"y":0.2},
  {"type":"mention","text":"@<username>","user_fbid":"<user_fbid>","style":"default","x":0.5,"y":0.4},
  {"type":"location","text":"<location_name>","location_id":"<location_id>","style":"default","x":0.5,"y":0.65},
  {"type":"link","text":"<link_text>","url":"<url>","style":"default","x":0.5,"y":0.82}
]'

instagram-cli post-story --draft --account-id <user_own_fbid> \
  --file <clean-media> --editor-json "$EDITOR_JSON"

instagram-cli post-story --account-id <user_own_fbid> --file <clean-media> \
  --editor-json "$EDITOR_JSON"
```

### Publish feed post / 发布 Feed 帖子
Publish media to the user's feed. `post-feed` selects the post type from the
files supplied:

将媒体发布到用户的 Feed。`post-feed` 根据提供的文件选择帖子类型：

- One JPEG or PNG image or static WebP image publishes a regular image post.
  Prefer portrait 4:5 at 1080 x 1350 px. Square 1:1 at 1080 x 1080 px is also
  suitable.
  单张 JPEG 或 PNG 图片或静态 WebP 图片发布为普通图片帖子，推荐 1080 x 1350 像素的竖版 4:5，1080 x 1080 像素的 1:1 方图同样适用。
- One MP4/MOV publishes a reel shared to the feed and profile grid. Use vertical
  9:16 video, preferably 1080 x 1920 px, and pass a matching JPEG, PNG, or
  static WebP `--cover`.
  单个 MP4/MOV 发布为同时分享到 Feed 和主页九宫格的 Reels。使用竖版 9:16 视频（推荐 1080 x 1920 像素），并传入配套的 JPEG、PNG 或静态 WebP `--cover`。
- Two or more JPEG, PNG, static WebP, MP4, or MOV files publish a carousel in
  the order supplied. Image, video, and mixed image/video carousels are
  supported. Keep all items at the same aspect ratio and dimensions, preferably
  4:5 at 1080 x 1350 px.
  两个及以上的 JPEG、PNG、静态 WebP、MP4 或 MOV 文件按提供顺序发布为轮播（carousel），支持纯图片、纯视频及图文混排轮播。所有条目应保持相同的宽高比与尺寸，推荐 1080 x 1350 像素的 4:5。

Each file may be at most 100 MB. `--caption` is optional, may include hashtags
and textual @mentions, and is limited to 2,200 characters. Whitespace-only
captions are treated as absent. `--mentions` takes a JSON array of user tags.
Each tag requires `user_fbid`. Optional `x` and `y` coordinates run from 0 to 1
and default to 0.5. On a carousel, the CLI applies the supplied user tags to
every item.

每个文件上限 100 MB。`--caption` 可选，可包含话题标签和文本形式的 @提及，上限 2,200 个字符；仅含空白字符的说明视为未提供。`--mentions` 接受一个用户标签 JSON 数组，每个标签都需要 `user_fbid`；可选的 `x` 和 `y` 坐标取值范围为 0 到 1，默认 0.5。在轮播中，CLI 会把提供的用户标签应用到每一个条目。

For video media, `--video-thumbnail-playback-offset-ms` optionally selects a
non-negative video frame for the final thumbnail. Each video requires a JPEG,
PNG, or static WebP `--cover`, shown until frame extraction finishes. For a
carousel, repeat `--cover` once per video in the same order as the video files.
Image items do not take covers. Do not pass either video option for an
image-only post or carousel.

对于视频媒体，`--video-thumbnail-playback-offset-ms` 可选地指定用于最终缩略图的非负视频帧位置。每个视频都需要 JPEG、PNG 或静态 WebP 的 `--cover`，在帧提取完成前显示该封面。轮播中按视频文件的顺序为每个视频重复一次 `--cover`。图片条目不需要封面。纯图片帖子或轮播不要传这两个视频选项。

```sh
# Image post
instagram-cli post-feed --account-id <user_own_fbid> \
  --file ~/workspace/<image>.jpg --caption '<caption>' \
  --mentions '[{"user_fbid":"<tagged_user_fbid>","x":0.5,"y":0.5}]'

# Reel shared to feed
instagram-cli post-feed --account-id <user_own_fbid> \
  --file ~/workspace/<reel>.mp4 --cover ~/workspace/<cover>.jpg \
  --video-thumbnail-playback-offset-ms 1200 \
  --caption '<caption>'

# Mixed carousel. File order is display order and cover order follows videos.
instagram-cli post-feed --account-id <user_own_fbid> \
  --file ~/workspace/<first>.jpg --file ~/workspace/<second>.mp4 \
  --cover ~/workspace/<second-cover>.jpg \
  --caption '<caption>'
```

Do not start the same publish again while its `post-feed` command is running
or a scheduled job for that publish is still in progress. Wait for the
command or job's result.

当 `post-feed` 命令仍在运行、或该发布对应的计划任务仍在进行时，不要再次发起同一发布，应等待该命令或任务的结果。

Do not use `--retries` for posting because an automatic retry could publish a
duplicate. Instagram also enforces a per-account daily publish limit.

发布时不要使用 `--retries`，因为自动重试可能导致重复发布。Instagram 还对每个账号实施每日发布次数上限。

【评论】此处把自动重试明确列为重复发布的风险来源，与上文的 429 熔断策略共同体现了对写操作幂等性的保守处理。

### Set profile picture / 设置头像
Pass exactly one square (1:1) JPEG or PNG, preferably at least 320 x 320 px and
no larger than 1080 x 1080 px. The file may be at most 100 MB.

传入且只传入一张 1:1 方形 JPEG 或 PNG，最好不小于 320 x 320 像素且不大于 1080 x 1080 像素。文件上限 100 MB。

```sh
instagram-cli set-profile-picture --account-id <user_own_fbid> \
  --file ~/workspace/<profile>.jpg
```

### Media understanding / 媒体理解
Fetch media descriptions for one or more Instagram media FBIDs, including narrative summary and semantic understanding. Batch IDs in a comma-separated `--media-ids` list.

按 FBID 获取一个或多个 Instagram 媒体的描述，包括叙事性摘要和语义理解。多个 ID 以逗号分隔放入 `--media-ids` 列表。

```sh
instagram-cli media-understanding --account-id <user_own_fbid> --media-ids <media_id>
instagram-cli media-understanding --account-id <user_own_fbid> --media-ids <media_id>,<media_id>
```

### Recently liked posts / 最近赞过的帖子
Fetch posts the user has recently liked.

获取用户最近点赞的帖子。

```sh
instagram-cli recently-liked-posts --account-id <user_own_fbid>
instagram-cli recently-liked-posts --account-id <user_own_fbid> --limit 5
instagram-cli recently-liked-posts --account-id <user_own_fbid> --limit 10 --after <cursor>
instagram-cli recently-liked-posts --account-id <user_own_fbid> --since 2026-01-01 --until 2026-02-01 --sort-order asc
```

### Recently commented posts / 最近评论过的帖子
Fetch posts the user has recently commented on.

获取用户最近评论过的帖子。

```sh
instagram-cli recently-commented-posts --account-id <user_own_fbid>
instagram-cli recently-commented-posts --account-id <user_own_fbid> --limit 5
instagram-cli recently-commented-posts --account-id <user_own_fbid> --limit 10 --after <cursor>
instagram-cli recently-commented-posts --account-id <user_own_fbid> --since 2026-01-01 --until 2026-02-01 --sort-order asc
```

### Account insights / 账号洞察

When reporting, explaining, or rewriting Instagram metrics, use the relevant
source definitions from [Account insights metrics](references/account-insights-metrics.md).
Read them with `muse.read` when the exact definitions are absent from the current
context, including on follow-ups. A previous summary does not replace them.
This includes captions based on supplied values. Use supplied values directly
unless the request needs missing or refreshed account data.

在报告、解释或改写 Instagram 指标时，使用 [Account insights metrics](references/account-insights-metrics.md) 中相关的原始定义。当当前上下文（包括后续追问）中没有确切定义时，用 `muse.read` 阅读它们；先前的摘要不能替代这些定义。基于所提供数值撰写文案时同样适用。除非请求需要缺失或需要刷新的账号数据，否则直接使用所提供的数值。

`account-insights` returns account-level totals. The default window is the
last 30 days. Choose another with `--period last_7_days`, `last_30_days`,
`last_90_days`, `this_month`, or `this_year`, or with `--start-date` and
`--end-date`. Paired `--start-time` and `--end-time` values select Unix-second
bounds and take precedence over periods and dates. A supported period takes
precedence over dates. Date-only bounds use UTC midnight. Use one supported
form and state its time window when presenting the values. The response does
not include the window.

`account-insights` 返回账号层面的总量。默认窗口为最近 30 天；可通过 `--period last_7_days`、`last_30_days`、`last_90_days`、`this_month` 或 `this_year`，或通过 `--start-date` 和 `--end-date` 选择其他窗口。成对的 `--start-time` 和 `--end-time` 以 Unix 秒为边界，优先于时间段和日期；受支持的时间段优先于日期；仅日期的边界按 UTC 午夜计。展示数值时只使用一种受支持的形式，并说明其时间窗口。响应本身不包含窗口信息。

A metric can be absent or `null`. Otherwise, it is a list of entries with a
`value` field. When `dimension_values` is present, interpret its codes using
the metric definitions before reporting the breakdown.

某个指标可能缺失或为 `null`；否则它是一个含 `value` 字段的条目列表。当存在 `dimension_values` 时，先按指标定义解读其中的代码，再报告细分数据。

Report each field by its Instagram name:

按 Instagram 的名称报告每个字段：

- `accounts_reached` (or `reach`) is **Accounts reached**.
  `accounts_reached`（或 `reach`）即 **触达账号数**。
- `viewers` is **Viewers**.
  `viewers` 即 **观看者数**。
- `content_views` is **Views**.
  `content_views` 即 **浏览量**。
- `engaged_accounts` is **Accounts engaged**.
  `engaged_accounts` 即 **互动账号数**。
- `total_interactions` is **Content interactions**.
  `total_interactions` 即 **内容互动数**。

When reporting Accounts reached, Viewers, or Accounts engaged, identify the
values as estimated account counts. One person can have multiple accounts.
Keep the account unit in reports and captions, including when addressing the
audience directly. State the reporting window in the text containing the
values.

报告触达账号数、观看者数或互动账号数时，应说明这些数值是账号数的估算值，因为一个人可能拥有多个账号。报告和文案中都要保留"账号"这一单位，包括直接面向受众的表述。包含这些数值的文字须注明统计窗口。

Views, interactions, profile visits, and taps count events. Keep those units
rather than converting their totals to accounts or people. Label a metric as
estimated when its definition says so. Do not add that label to other metrics.

浏览、互动、主页访问和点击统计的是事件次数，应保留这些单位，不要把总量换算成账号数或人数。当某个指标的定义本身注明为估算值时，才标注"估算"，其他指标不得添加该标注。

If requested copy uses the wrong unit, correct it in the publishable text.
A separate note does not correct a misleading caption.

如果待发布文案使用了错误的单位，直接在可发布文本中改正；单独附加说明并不能纠正误导性的说明文字。

## Output / 输出
The CLI prints JSON to stdout. Post reads use the normalized collection above.
Other commands retain their provider-specific decoded JSON. When presenting
results to the user, focus on meaningful content (names, text, images or
video, links, captions).

CLI 将 JSON 打印到标准输出。帖子读取使用上文归一化后的集合，其他命令保留各自服务端解码后的 JSON。向用户呈现结果时，聚焦有意义的内容（名称、文字、图片或视频、链接、说明文字）。

Include what you found in your response before asking a question or offering
more help. Do not end with only a question, an offer, or a promise of later
work.

在提问或提议进一步帮助之前，先把发现的内容写进回复。不要仅以一个问题、一个提议或对后续工作的承诺收尾。

Do not expose FBIDs, opaque IDs, cursors, Unix timestamps, or implementation
terminology to the user. Use usernames, display names, and plain-language
descriptions. Keep IDs only in tool calls.

不要向用户暴露 FBID、不透明 ID、游标、Unix 时间戳或实现层面的术语；使用用户名、显示名和通俗描述。ID 只保留在工具调用中。
