<!-- BILINGUAL-EN-ZH -->
# Facebook Events / Facebook 活动

Search Facebook Events and read a single event's details.

搜索 Facebook 活动并读取单个活动的详情。

## Commands / 命令

### Search Events / 搜索活动

```bash
facebook-cli events search [--scope <scope>] [--keywords <text>] [--location <place>] [--latitude <lat> --longitude=<lng>] [--radius-in-miles <n>] [--category <category>] [--start-date <YYYY-MM-DD>] [--end-date <YYYY-MM-DD>] [--limit <n>] [--after <cursor>]
```

**Options: / 选项：**
- `--scope` (optional): `connected` (default) — events you are connected to (going, interested, invited, hosting); `discover` — popular nearby events from the recommendation backend
- `--scope`（可选）：`connected`（默认）——你已关联的活动（将参加、感兴趣、受邀请、主办）；`discover`——来自推荐后端的热门附近活动
- `--keywords` (optional): Free-text search, e.g. `"music"`, `"farmers market"`
- `--keywords`（可选）：自由文本搜索，例如 `"music"`、`"farmers market"`
- `--location` (optional): Place string, e.g. `"Seattle"`. Discover scope searches near this place instead of your location. The server geocodes it to a **city center**, so it cannot express a neighborhood or an address
- `--location`（可选）：地点字符串，例如 `"Seattle"`。discover 范围会以该地点（而非你的位置）为圆心搜索。服务器会把它地理编码为**市中心**，因此它无法表达某个街区或具体地址
- `--latitude` / `--longitude` (optional, discover scope only): Exact WGS-84 search center. Pass both or neither; one alone is an error. The pair takes precedence over `--location`. Use this whenever you have real coordinates — it is more accurate than any place string
- `--latitude` / `--longitude`（可选，仅限 discover 范围）：精确的 WGS-84 搜索中心。两者都传或都不传；只传一个是错误。这对参数优先于 `--location`。只要有真实坐标就使用它——它比任何地点字符串都更精确
- `--radius-in-miles` (optional, needs the coordinate pair): Search radius around the coordinates. Defaults to 25 miles
- `--radius-in-miles`（可选，需要坐标对）：围绕坐标的搜索半径。默认 25 英里
- `--category` (optional): Event category name, e.g. `MUSIC_AND_AUDIO`
- `--category`（可选）：活动类别名称，例如 `MUSIC_AND_AUDIO`
- `--start-date` / `--end-date` (optional): Calendar dates (`YYYY-MM-DD`); start must not be after end
- `--start-date` / `--end-date`（可选）：日历日期（`YYYY-MM-DD`）；开始日期不得晚于结束日期
- `--limit` (optional): Maximum events per page (default 10, max 25)
- `--limit`（可选）：每页活动数上限（默认 10，最大 25）
- `--after` (optional): Pagination cursor from the previous response (`paging.cursors.after`). Cursors are bound to their search scope — do not reuse a cursor with a different scope
- `--after`（可选）：来自上一次响应的分页游标（`paging.cursors.after`）。游标与其搜索范围绑定——不要把游标用于不同的范围

**Examples: / 示例：**  
```bash
# My upcoming events
facebook-cli events search

# Keyword search across popular nearby events
facebook-cli events search --scope discover --keywords "music" --limit 5

# Events near a place on/after a date (city-center accuracy)
facebook-cli events search --scope discover --location "Seattle" --start-date 2026-07-31 --limit 5

# Events near exact coordinates, within 5 miles (preferred when coordinates are known)
facebook-cli events search --scope discover --latitude 37.5072 --longitude=-122.2605 --radius-in-miles 5 --limit 5

# Next page (pass the cursor from the previous response)
facebook-cli events search --scope discover --keywords "music" --after "<cursor_from_previous_response>"
```

**Choosing a location: / 选择位置：**
1. If the turn context already carries the user's coordinates (for example a `message_location`
   tag), pass them straight to `--latitude` / `--longitude`. Do not reverse-geocode them into a
   place name first — that throws away the precision this command exists to use.
2. 如果本轮上下文已带有用户的坐标（例如 `message_location` 标签），把它们直接传给 `--latitude` / `--longitude`。不要先把它们逆地理编码为地名——那样会丢弃本命令专门用来利用的精度。
2. If the user named a place, geocode it with `map.geocode` and pass the coordinates, or pass the
   place string to `--location` when city-level accuracy is enough.
3. 如果用户说出了地名，用 `map.geocode` 对其地理编码并传入坐标；当城市级精度已足够时，也可以把地点字符串传给 `--location`。
3. If nothing establishes a location, ask the user. Discover scope with no location falls back to
   a server-side guess, and the response never says which place it used.
4. 如果没有任何信息能确定位置，询问用户。无位置的 discover 范围会退回到服务端的猜测，且响应绝不会说明它用的是哪个地点。

**Negative coordinates:** write `--longitude=-122.2605` with the `=`. A bare `--longitude -122.2605`
is misread as another flag and fails with `error: unexpected argument '-1' found`. The same applies
to `--latitude` in the southern hemisphere.

**负坐标：** 写 `--longitude=-122.2605` 时要带 `=`。裸写的 `--longitude -122.2605` 会被误读为另一个标志，并报错 `error: unexpected argument '-1' found`。南半球的 `--latitude` 同理。

**Response fields per event: / 每个活动的响应字段：**
- `id`: Numeric raw event ID (global, not app-scoped — matches the ID in `permalink_url`)
- `id`：数字形式的原始活动 ID（全局，非应用范围——与 `permalink_url` 中的 ID 一致）
- `name`: Event name
- `name`：活动名称
- `permalink_url`: Shareable link, always `https://www.facebook.com/events/<id>/`
- `permalink_url`：可分享链接，始终为 `https://www.facebook.com/events/<id>/`
- `description`: Event description (may be null)
- `description`：活动描述（可能为 null）
- `start_time` / `end_time`: Timestamps with offset (e.g. `2026-08-01T11:00:00-0400`)
- `start_time` / `end_time`：带时区偏移的时间戳（例如 `2026-08-01T11:00:00-0400`）
- `location`: Venue or address text (may be null)
- `location`：场馆或地址文本（可能为 null）
- `category`: Category name (may be null)
- `category`：类别名称（可能为 null）
- `going_count` / `interested_count`: Attendance counts
- `going_count` / `interested_count`：出席人数统计
- `timezone`: IANA timezone (may be null)
- `timezone`：IANA 时区（可能为 null）
- `privacy`: `public`, `private`, etc. (may be null)
- `privacy`：`public`、`private` 等（可能为 null）
- `is_online`: Whether the event is online
- `is_online`：活动是否在线举行
- `online_url`: Third-party URL where an online event is hosted. Details responses only — search results omit this key entirely rather than sending null
- `online_url`：在线活动举办所在的第三方 URL。仅详情响应提供——搜索结果会完全省略该键，而不是发送 null

The response is paginated: events live under `data`, with the next cursor at `paging.cursors.after` when more pages exist.

响应是分页的：活动位于 `data` 之下，当还有更多页时，下一页游标位于 `paging.cursors.after`。

### Read Event Details / 读取活动详情

```bash
facebook-cli events details --event-id <id>
```

**Options: / 选项：**
- `--event-id` (required): Raw numeric event ID — the `id` from search results, or the number embedded in the event's `permalink_url`
- `--event-id`（必填）：原始数字活动 ID——搜索结果中的 `id`，或活动 `permalink_url` 中内嵌的数字

This is the only events route that populates `online_url`. A malformed ID returns 400; an event that does not exist or is not visible to you returns 404 (the two cases are indistinguishable by design).

这是唯一会填充 `online_url` 的活动路由。格式错误的 ID 返回 400；不存在或对你不可见的活动返回 404（按设计这两种情况无法区分）。

**Examples: / 示例：**  
```bash
# Read one event (ID taken from a search result)
facebook-cli events details --event-id 1679757800076464
```

**Response fields:** same as search, plus `online_url` (null when the event has no third-party online URL).

**响应字段：** 与搜索相同，外加 `online_url`（当活动没有第三方在线 URL 时为 null）。
