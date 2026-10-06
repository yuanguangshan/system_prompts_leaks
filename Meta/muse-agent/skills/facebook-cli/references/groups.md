<!-- BILINGUAL-EN-ZH -->
# Facebook Groups / Facebook 小组

Fetch group details, search groups, and browse posts in Facebook Groups.

获取小组详情、搜索小组，以及浏览 Facebook 小组中的帖子。

## Commands / 命令

### Group Details / 小组详情

```bash
facebook-cli groups details --group-id "<id-or-vanity>"
```

`--group-id` accepts either a numeric group FBID or the vanity name from a  
Facebook group URL such as  
`https://www.facebook.com/groups/<group-id-or-vanity>`. Use the path segment
after `/groups/`; do not pass the whole URL.

`--group-id` 接受数字形式的群组 FBID，或来自 Facebook 小组 URL（如 `https://www.facebook.com/groups/<group-id-or-vanity>`）中的个性化名称。使用 `/groups/` 之后的路径段；不要传入整个 URL。

**Examples: / 示例：**

```bash
# Fetch by numeric FBID
facebook-cli groups details --group-id "<group-id>"

# Fetch by vanity name
facebook-cli groups details --group-id "<group-vanity-name>"
```

**Response fields: / 响应字段：**

- `name`: Group name
- `name`：小组名称
- `about`: Group About text
- `about`：小组"简介"文本
- `visibility`: `OPEN`, `CLOSED`, or `SECRET`
- `visibility`：`OPEN`、`CLOSED` 或 `SECRET`
- `history`: Localized creation and name-change history, when visible
- `history`：本地化的创建与更名历史（可见时）
- `tags`: Admin-selected group tags
- `tags`：管理员选择的小组标签
- `rules`: Group rules as title/description objects in admin-defined order
- `rules`：小组规则，以标题/描述对象表示，按管理员定义的顺序排列
- `member_count`: Approximate member count
- `member_count`：约略成员数

Missing groups and groups that are not visible to the linked Facebook account
return the same not-found error.

不存在的小组与已关联 Facebook 账户不可见的小组，返回相同的未找到错误。

### Search Groups / 搜索小组

```bash
facebook-cli groups search [--keywords <text>] [--role <role>] [--sort-by <sort>] [--status <status>] [--limit <n>] [--after <cursor>]
```

**Options: / 选项：**
- `--keywords` (optional): Search by name or topic. Omit to list your own groups
- `--keywords`（可选）：按名称或主题搜索。省略则列出你自己的小组
- `--role` (optional): Controls search scope:
- `--role`（可选）：控制搜索范围：
  - `connected` (default) — only groups you're a member of
  - `connected`（默认）——仅限你所属的小组
  - `admin` — groups where you're an admin
  - `admin`——你担任管理员的小组
  - `admod` — groups where you're admin or moderator
  - `admod`——你担任管理员或版主的小组
  - `any` — search connected groups (including private groups) first, then
    public discovery results (requires `--keywords`)
  - `any`——先搜索已关联小组（含私密小组），再搜索公开发现结果（需要 `--keywords`）
- `--sort-by` (optional): `MOST_RELEVANT`, `LARGEST`, `LAST_VISITED`, `RECENT_ACTIVITY`, `ALPHABETICAL`
- `--sort-by`（可选）：`MOST_RELEVANT`、`LARGEST`、`LAST_VISITED`、`RECENT_ACTIVITY`、`ALPHABETICAL`
- `--status` (optional): `any`, `weekly_active`, `non_archived` (default), `archived`
- `--status`（可选）：`any`、`weekly_active`、`non_archived`（默认）、`archived`
- `--limit` (optional): Maximum number of results per page (default 25 with  
  `--keywords`, 10 otherwise; max 25)
- `--limit`（可选）：每页结果数上限（带 `--keywords` 时默认 25，否则为 10；最大 25）
- `--after` (optional): Cursor from `paging.cursors.after` in the previous
  response. Supported only when `--keywords` is omitted
- `--after`（可选）：来自上一次响应 `paging.cursors.after` 的游标。仅在省略 `--keywords` 时受支持

**Examples: / 示例：**  
```bash
# List my groups
facebook-cli groups search

# Search for groups about hiking
facebook-cli groups search --keywords "hiking"

# My largest groups
facebook-cli groups search --sort-by LARGEST --limit 5

# Groups I admin
facebook-cli groups search --role admin

# Search connected and public groups
facebook-cli groups search --keywords "hiking" --role any

# Next page while listing memberships
facebook-cli groups search --role connected --after "<cursor_from_previous_response>"
```

**Response:** Each page keeps results in `groups`; `paging.cursors.after` is
present when another page is available.

**响应：** 每页的结果保存在 `groups` 中；当还有下一页时会出现 `paging.cursors.after`。

**Group fields: / 小组字段：**
- `group_id`: Numeric group ID
- `group_id`：数字形式的群组 ID
- `name`: Group name
- `name`：小组名称
- `member_count`: Number of members
- `member_count`：成员数量
- `privacy`: `Public` or `Private`
- `privacy`：`Public` 或 `Private`
- `group_url`: Direct link to the group (e.g. `https://www.facebook.com/groups/<group-id>/`)
- `group_url`：小组的直接链接（例如 `https://www.facebook.com/groups/<group-id>/`）

### Search Posts in a Group / 在小组内搜索帖子

```bash
facebook-cli groups posts --group-id <id> [--query <text>] [--sort-by <sort>] [--limit <n>] [--after <cursor>] [--min-timestamp <unix>] [--max-timestamp <unix>]
```

**Options: / 选项：**
- `--group-id` (required): Group ID to search posts in
- `--group-id`（必填）：要在其中搜索帖子的小组 ID
- `--query` (optional): Text query to filter posts by content
- `--query`（可选）：按内容筛选帖子的文本查询
- `--sort-by` (optional): `MOST_RECENT_ACTIVITY` (default), `MOST_REACTS`, `NEW_POSTS`
- `--sort-by`（可选）：`MOST_RECENT_ACTIVITY`（默认）、`MOST_REACTS`、`NEW_POSTS`
- `--limit` (optional): Maximum number of results (default 10, max 25)
- `--limit`（可选）：结果数上限（默认 10，最大 25）
- `--after` (optional): Pagination cursor from a previous response — pass it to fetch the next page (max page size 20)
- `--after`（可选）：来自上一次响应的分页游标——传入它以获取下一页（每页最大 20 条）
- `--min-timestamp` (optional): Lower bound on post time, Unix seconds (exclusive) — only posts newer than this
- `--min-timestamp`（可选）：帖子时间下界，Unix 秒（不含）——只返回晚于该时间的帖子
- `--max-timestamp` (optional): Upper bound on post time, Unix seconds (inclusive) — only posts at or before this
- `--max-timestamp`（可选）：帖子时间上界，Unix 秒（含）——只返回不晚于该时间的帖子

> **Note:** Pagination (`--after`) is only supported with `MOST_RECENT_ACTIVITY` and `NEW_POSTS` (max page size 20). `MOST_REACTS` does not return a pagination cursor, so it can't be paged.

> **注意：** 分页（`--after`）仅在 `MOST_RECENT_ACTIVITY` 和 `NEW_POSTS` 下受支持（每页最大 20 条）。`MOST_REACTS` 不返回分页游标，因此无法分页。

**Examples: / 示例：**  
```bash
# Recent posts in a group
facebook-cli groups posts --group-id "<group-id>"

# Most popular posts
facebook-cli groups posts --group-id "<group-id>" --sort-by MOST_REACTS --limit 5

# Newest posts first
facebook-cli groups posts --group-id "<group-id>" --sort-by NEW_POSTS --limit 5

# Search for posts about a topic
facebook-cli groups posts --group-id "<group-id>" --query "recipe"

# Next page (pass the cursor from the previous response)
facebook-cli groups posts --group-id "<group-id>" --after "<cursor_from_previous_response>"

# Posts within a time window (Unix seconds)
facebook-cli groups posts --group-id "<group-id>" --min-timestamp 1750000000 --max-timestamp 1752000000
```
