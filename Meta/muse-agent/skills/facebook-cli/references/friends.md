<!-- BILINGUAL-EN-ZH -->
# Facebook Friends / Facebook 好友

List and search your Facebook friends by name or profile fields.

按姓名或资料字段列出并搜索你的 Facebook 好友。

## Commands / 命令

### List / Search Friends / 列出与搜索好友

```bash
# List the first page of close friends (20 per page)
facebook-cli me friends

# Search all friends by name
facebook-cli me friends --name "Sarah"

# Filter by city
facebook-cli me friends --city "San Francisco"

# Filter by workplace
facebook-cli me friends --work "Meta"

# Combine filters (AND by default — all must match)
facebook-cli me friends --city "New York" --work "Google"

# Combine filters with OR (any must match)
facebook-cli me friends --city "Seattle" --education "Stanford" --filter-mode OR

# Friends with birthdays in the next week (default when no number is given)
facebook-cli me friends --birthday-within-days

# Friends with birthdays in the next 30 days (ordered by upcoming birthday)
facebook-cli me friends --birthday-within-days 30

# Fetch the next page (cursor from the previous response's paging.cursors.after)
facebook-cli me friends --after <cursor>
```

| Flag | Required | Description |
|------|----------|-------------|
| `--name` / `-n` | No | Search all friends by name (case-insensitive substring match) |
| `--city` | No | Filter by current city (case-insensitive substring match) |
| `--hometown` | No | Filter by hometown (case-insensitive substring match) |
| `--work` | No | Filter by employer name or position title (case-insensitive substring match) |
| `--education` | No | Filter by school name (case-insensitive substring match) |
| `--filter-mode` | No | How to combine multiple filters: `AND` (default, all must match) or `OR` (any must match) |
| `--json-query` | No | JSON query for SocialGraphSearch (FindPeople format). Uses Social RAG search for interest/topic-based matching. **Not paginated** — returns a single result set with no `paging` cursor, so `--after` has no effect; `--limit` (max 20) still caps the result set. |
| `--birthday-within-days` | No | Return only friends whose birthday falls within the next N days (1–365). **Defaults to 7 (the next week) when passed with no value** (e.g. `--birthday-within-days`); omitting the flag entirely returns the normal friends list. Adds a `birthday_date` field and orders results by upcoming birthday. Privacy-aware (friends who hide their birthday are excluded). When set, the other filters are ignored. |
| `--limit` | No | Maximum number of friends per page (max 20; higher values are capped server-side) |
| `--after` | No | Pagination cursor — pass the `paging.cursors.after` value from the previous response to fetch the next page |

| 标志 | 必需 | 说明 |
|------|----------|-------------|
| `--name` / `-n` | 否 | 按姓名搜索全部好友（不区分大小写的子串匹配） |
| `--city` | 否 | 按当前城市过滤（不区分大小写的子串匹配） |
| `--hometown` | 否 | 按家乡过滤（不区分大小写的子串匹配） |
| `--work` | 否 | 按雇主名称或职位头衔过滤（不区分大小写的子串匹配） |
| `--education` | 否 | 按学校名称过滤（不区分大小写的子串匹配） |
| `--filter-mode` | 否 | 多个过滤条件的组合方式：`AND`（默认，必须全部匹配）或 `OR`（任一匹配即可） |
| `--json-query` | 否 | 用于 SocialGraphSearch 的 JSON 查询（FindPeople 格式）。基于兴趣/主题的匹配使用 Social RAG 搜索。**不分页**——返回单一结果集，没有 `paging` 游标，因此 `--after` 无效；`--limit`（最大 20）仍会限制结果集大小。 |
| `--birthday-within-days` | 否 | 只返回生日落在未来 N 天内（1–365）的好友。**不带值传入时默认 7（未来一周）**（例如 `--birthday-within-days`）；完全省略该标志则返回普通好友列表。会添加 `birthday_date` 字段并按即将到来的生日排序。注重隐私（隐藏生日的好友会被排除）。设置该标志时，其他过滤器被忽略。 |
| `--limit` | 否 | 每页最大好友数（最大 20；更高的值会在服务端被截断） |
| `--after` | 否 | 分页游标——传入上一个响应的 `paging.cursors.after` 值以抓取下一页 |

**Output:** JSON with a `data` array plus a `paging` object. Each friend in `data` has `friend_id`, `name`, `profile_url`, and optionally `match_context` (with `groups`, `pages`, `profile_details` explaining why they matched). When `--birthday-within-days` is used, each friend also has `birthday_date` (`YYYY-MM-DD`), and results are ordered by upcoming birthday ascending. Without any filters, returns the first page (20) of close friends — paginate with `--after` for more. With `--json-query`, uses SocialGraphSearch for richer matching.

**输出：** JSON，含一个 `data` 数组和一个 `paging` 对象。`data` 中每个好友都有 `friend_id`、`name`、`profile_url`，以及可选的 `match_context`（其中的 `groups`、`pages`、`profile_details` 解释其匹配原因）。使用 `--birthday-within-days` 时，每个好友还有 `birthday_date`（`YYYY-MM-DD`），结果按即将到来的生日升序排列。不带任何过滤器时，返回密友第一页（20 个）——用 `--after` 翻页获取更多。使用 `--json-query` 时，用 SocialGraphSearch 做更丰富的匹配。

**Pagination:** results are cursor-paginated (forward-only). `paging.cursors.after` (when present) is the cursor for the next page; its absence means there are no more results. Pass it back via `--after`. (Note: a cursor is only valid for the same filter mode it came from — don't reuse an `after` from one mode after switching filters.) **Exception:** `--json-query` (SocialGraphSearch) is **not paginated** — it returns a single result set with no `paging` cursor, so `--after` has no effect; `--limit` (max 20) still caps how many results come back.

**分页：** 结果采用游标分页（只能向前）。`paging.cursors.after`（若存在）是下一页的游标；不存在则说明没有更多结果。通过 `--after` 把它传回去。（注意：游标只对它来源的那个过滤模式有效——切换过滤器后不要复用另一个模式的 `after`。）**例外：** `--json-query`（SocialGraphSearch）**不分页**——它返回单一结果集，没有 `paging` 游标，因此 `--after` 无效；`--limit`（最大 20）仍会限制返回的结果数量。

### Upcoming birthdays via `--birthday-within-days` / 通过 `--birthday-within-days` 查看即将到来的生日

Use `--birthday-within-days N` to find friends with birthdays in the next N days (1–365). This is the right tool for "whose birthday is coming up?", "any birthdays this week?", or "who should I wish happy birthday?". It uses a privacy-aware birthday lookup, so friends who hide their birthday are excluded.

用 `--birthday-within-days N` 查找未来 N 天内（1–365）过生日的好友。对于"谁快过生日了？"、"这周有生日吗？"或"我该祝谁生日快乐？"这类问题，这就是正确的工具。它使用注重隐私的生日查询，隐藏生日的好友会被排除。

**Default window:** when the user asks about upcoming birthdays without naming a timeframe, pass the flag with no value — it defaults to the next 7 days (a week). Only pass an explicit number when the user specifies one ("this month" → `30`, "next 90 days" → `90`).

**默认窗口：** 当用户问及即将到来的生日而未指明时间范围时，不带值传入该标志——默认为未来 7 天（一周）。只有当用户给出明确数字时才传入明确数值（"这个月" → `30`，"接下来 90 天" → `90`）。

```bash
# Birthdays in the next week (default — no value needed)
facebook-cli me friends --birthday-within-days

# Birthdays in an explicit window
facebook-cli me friends --birthday-within-days 30
```

```json
{
  "data": [
    {
      "friend_id": "123456789",
      "name": "Jane Smith",
      "profile_url": "https://facebook.com/jane.smith",
      "birthday_date": "2026-06-14"
    }
  ],
  "paging": { "cursors": { "after": "<cursor>" } }
}
```

`birthday_date` is in `YYYY-MM-DD` format. Results are ordered by date ascending starting from today, one page at a time (page size 20) — follow `paging.cursors.after` with `--after` for more. The other filters (`--name`, `--city`, `--hometown`, `--work`, `--education`, `--json-query`) are ignored when `--birthday-within-days` is set.

`birthday_date` 为 `YYYY-MM-DD` 格式。结果从今天起按日期升序排列，一次一页（页大小 20）——跟随 `paging.cursors.after` 用 `--after` 获取更多。设置了 `--birthday-within-days` 时，其他过滤器（`--name`、`--city`、`--hometown`、`--work`、`--education`、`--json-query`）都会被忽略。

### SocialGraphSearch via `--json-query` / 通过 `--json-query` 使用 SocialGraphSearch

Use `--json-query` to search friends by interests, sports, topics, etc. First get your Facebook ID with `facebook-cli me`, then use it in the `friend_by` filter:

用 `--json-query` 按兴趣、运动、主题等搜索好友。先用 `facebook-cli me` 获取你的 Facebook ID，再把它用于 `friend_by` 过滤器：

> **No pagination:** `--json-query` returns a single result set with no `paging` cursor, so `--after` has no effect. `--limit` (max 20) still caps how many results are returned; to go beyond that, refine the query filters rather than paging.

> **不分页：** `--json-query` 返回单一结果集，没有 `paging` 游标，因此 `--after` 无效。`--limit`（最大 20）仍会限制返回的结果数量；要超出该数量，应细化查询过滤器而不是翻页。

```bash
facebook-cli me friends --json-query '{
  "intent": "FindPeople",
  "target": "users",
  "filters": {
    "friend_by": ["YOUR_FB_ID"],
    "topic": ["hiking"]
  }
}'
```

**Important:** Use `topic` as the filter key for interests (NOT `interests`). The `friend_by` value must be the user's Facebook ID from `facebook-cli me`.

**重要：** 兴趣类搜索要用 `topic` 作为过滤器键（不是 `interests`）。`friend_by` 的值必须是 `facebook-cli me` 返回的用户 Facebook ID。

#### Supported filters / 支持的过滤器

| Filter | Description | Example | match_context category |
|--------|-------------|---------|----------------------|
| `friend_by` | Facebook ID of viewer (required) | `["YOUR_FB_ID"]` | — |
| `topic` | Interests/topics/hobbies | `["hiking"]`, `["seattle seahawks"]` | groups, pages |
| `sports` | Sports teams/activities | `["pickleball"]`, `["soccer"]` | pages |
| `company` | Employer | `["Meta"]`, `["Google"]` | profile_details |
| `job` | Job title | `["engineer"]` | profile_details |
| `college` | College/university | `["Stanford"]` | profile_details |
| `school` | Any school | `["MIT"]` | profile_details |
| `location` | Current city | `["Seattle"]` | profile_details |
| `hometown` | Hometown | `["Mumbai"]` | profile_details |
| `movies`, `music`, `tv_shows`, `books`, `games`, `podcasts` | Entertainment | `["Inception"]` | pages |

| 过滤器 | 说明 | 示例 | match_context 类别 |
|--------|-------------|---------|----------------------|
| `friend_by` | 查看者的 Facebook ID（必填） | `["YOUR_FB_ID"]` | — |
| `topic` | 兴趣/主题/爱好 | `["hiking"]`、`["seattle seahawks"]` | groups, pages |
| `sports` | 运动队/运动活动 | `["pickleball"]`、`["soccer"]` | pages |
| `company` | 雇主 | `["Meta"]`、`["Google"]` | profile_details |
| `job` | 职位头衔 | `["engineer"]` | profile_details |
| `college` | 学院/大学 | `["Stanford"]` | profile_details |
| `school` | 任何学校 | `["MIT"]` | profile_details |
| `location` | 当前城市 | `["Seattle"]` | profile_details |
| `hometown` | 家乡 | `["Mumbai"]` | profile_details |
| `movies`, `music`, `tv_shows`, `books`, `games`, `podcasts` | 娱乐 | `["Inception"]` | pages |

**Mapping user intent to filter key:**
- "friends who like hiking" / "friends interested in cooking" → `topic`
- "friends who play soccer" / "friends into basketball" → `sports`
- "friends who work at Meta" → `company`
- "friends in Seattle" / "friends living in NYC" → `location`
- "friends who went to Stanford" → `college`
- "friends who watch Breaking Bad" → `tv_shows`

**把用户意图映射到过滤器键：**
- "friends who like hiking" / "friends interested in cooking" → `topic`
  "喜欢徒步的好友" / "对烹饪感兴趣的好友" → `topic`
- "friends who play soccer" / "friends into basketball" → `sports`
  "踢足球的好友" / "爱打篮球的好友" → `sports`
- "friends who work at Meta" → `company`
  "在 Meta 工作的好友" → `company`
- "friends in Seattle" / "friends living in NYC" → `location`
  "在 Seattle 的好友" / "住在纽约的好友" → `location`
- "friends who went to Stanford" → `college`
  "上过 Stanford 的好友" → `college`
- "friends who watch Breaking Bad" → `tv_shows`
  "看《绝命毒师》的好友" → `tv_shows`

#### Response with match context / 带匹配上下文的响应

When using `--json-query`, results include `match_context` explaining why each friend matched. The context strings contain XML-tagged data that should be parsed for display:

使用 `--json-query` 时，结果包含解释每个好友匹配原因的 `match_context`。上下文字符串含有带 XML 标签的数据，展示前应先解析：

```json
{
  "friend_id": "123456789",
  "name": "Jane Smith",
  "profile_url": "https://facebook.com/jane.smith",
  "match_context": {
    "groups": [
      {
        "id": "111222333",
        "context": "<GROUP_NAME>Hiking Enthusiasts</GROUP_NAME><GROUP_DESCRIPTION>A group for open discussion of hiking and backpacking topics...</GROUP_DESCRIPTION>"
      }
    ],
    "pages": [
      {
        "id": "444555666",
        "context": "<PAGE_NAME>Home Cooking Tips</PAGE_NAME><PAGE_CATEGORY>Kitchen/Cooking</PAGE_CATEGORY><PAGE_DESCRIPTION>Easy recipes for home cooks...</PAGE_DESCRIPTION>"
      }
    ],
    "profile_details": [
      {
        "source_id": "777888999",
        "context": "Works at Acme Corp"
      }
    ]
  }
}
```

**Parsing context strings:**
- **groups**: Extract name from `<GROUP_NAME>...</GROUP_NAME>`, description from `<GROUP_DESCRIPTION>...</GROUP_DESCRIPTION>`
- **pages**: Extract name from `<PAGE_NAME>...</PAGE_NAME>`, category from `<PAGE_CATEGORY>...</PAGE_CATEGORY>`
- **profile_details**: Plain text, no XML tags (e.g. "Works at Acme Corp", "Currently located in Seattle, Washington", "Attended school State University")

**解析上下文字符串：**
- **groups**: Extract name from `<GROUP_NAME>...</GROUP_NAME>`, description from `<GROUP_DESCRIPTION>...</GROUP_DESCRIPTION>`
  **groups**：名称取自 `<GROUP_NAME>...</GROUP_NAME>`，描述取自 `<GROUP_DESCRIPTION>...</GROUP_DESCRIPTION>`
- **pages**: Extract name from `<PAGE_NAME>...</PAGE_NAME>`, category from `<PAGE_CATEGORY>...</PAGE_CATEGORY>`
  **pages**：名称取自 `<PAGE_NAME>...</PAGE_NAME>`，类别取自 `<PAGE_CATEGORY>...</PAGE_CATEGORY>`
- **profile_details**: Plain text, no XML tags (e.g. "Works at Acme Corp", "Currently located in Seattle, Washington", "Attended school State University")
  **profile_details**：纯文本，无 XML 标签（例如 "Works at Acme Corp"、"Currently located in Seattle, Washington"、"Attended school State University"）

**Presenting results:** When showing match context to the user, extract the readable names and present them naturally:
- "Jane Smith — member of Hiking Enthusiasts, Trail Runners groups"
- "John Doe — follows Home Cooking Tips, Chef's Table pages"
- "Alex Lee — Works at Acme Corp"

**呈现结果：** 向用户展示匹配上下文时，提取可读的名称并自然地表述：
- "Jane Smith — member of Hiking Enthusiasts, Trail Runners groups"
  "Jane Smith——Hiking Enthusiasts、Trail Runners 小组的成员"
- "John Doe — follows Home Cooking Tips, Chef's Table pages"
  "John Doe——关注 Home Cooking Tips、Chef's Table 主页"
- "Alex Lee — Works at Acme Corp"
  "Alex Lee——就职于 Acme Corp"

### Your Identity / 你的身份

```bash
facebook-cli me
```

**Output:** JSON with `ok`, `name`, and `profile_id` — your Facebook name and profile ID. This grounds the agent on who the Facebook user is. Use the profile ID with other commands like `profile info --profile-id` (for full profile details) or `timeline fetch --profile-id` (for your posts).

**输出：** JSON，含 `ok`、`name` 和 `profile_id`——你的 Facebook 姓名与个人资料 ID。这能让代理明确这个 Facebook 用户是谁。把该资料 ID 用于其他命令，如 `profile info --profile-id`（获取完整资料详情）或 `timeline fetch --profile-id`（获取你的帖子）。

## Operating Rules / 操作规则

1. Use `me friends` to search for people by name or profile fields. There is no general profile search endpoint — `me friends` is the only people lookup.
   用 `me friends` 按姓名或资料字段搜索人物。没有通用的人物搜索端点——`me friends` 是唯一的人物查询途径。
2. Use the dedicated filter flags (`--city`, `--hometown`, `--work`, `--education`) to filter friends directly — no need to search by name first and then manually filter results.
   用专用过滤标志（`--city`、`--hometown`、`--work`、`--education`）直接过滤好友——无需先按姓名搜索再手工过滤结果。
3. For interest-based friend search (e.g. "which friends like hiking?"), use `--json-query` with a FindPeople template using `topic` as the filter key. Do not scan friends' posts, timelines, groups, or page likes to infer interests.
   基于兴趣的好友搜索（例如"哪些好友喜欢徒步？"）用 `--json-query` 加 FindPeople 模板，以 `topic` 作为过滤器键。不要扫描好友的帖子、时间线、小组或主页点赞来推断兴趣。
   【评论】"不要扫描好友内容来推断兴趣"是一条隐私边界：把代理的检索范围限定在平台显式提供的搜索接口内，而不是对社交内容做自主挖掘。
4. When multiple friends match, list them with distinguishing details (name, city, workplace, profile link) and ask the user to clarify.
   当多个好友匹配时，附带区分性细节（姓名、城市、工作单位、资料链接）列出，并请用户澄清。
5. Always include a profile link (`profile_url`) for each friend in results.
   结果中每个好友都要附上资料链接（`profile_url`）。
6. After disambiguation, proceed with the original request. If the user asked "show me posts from Liz" and you listed matching friends, once the user picks one (or if only one match exists for a nickname like "Liz" → "Liz Taylor"), fetch and display the posts — do not stop at listing friends.
   消歧之后，继续原来的请求。若用户要求"给我看 Liz 的帖子"而你列出了匹配的好友，一旦用户选定其中一个（或者该昵称只有一个匹配，如 "Liz" → "Liz Taylor"），就抓取并展示帖子——不要停留在列出好友这一步。
7. When searching by name, expand common nicknames to their full forms and search for both. For example, "Liz" → also search "Elizabeth"; "Mike" → also search "Michael"; "Bob" → also search "Robert"; "Bill" → also search "William"; "Teddy" → also search "Theodore". Combine results from both searches before presenting to the user.
   按姓名搜索时，把常见昵称展开为全名并两者都搜。例如 "Liz" → 同时搜 "Elizabeth"；"Mike" → 同时搜 "Michael"；"Bob" → 同时搜 "Robert"；"Bill" → 同时搜 "William"；"Teddy" → 同时搜 "Theodore"。在呈现给用户之前合并两次搜索的结果。
