---
name: "media_library"
description: "Search and inspect the user's photo library, including connected device galleries. Use for photo requests and whenever a photo could ground or personalize a response; lookups of uploaded photos are cheap, so check opportunistically and move on if nothing fits."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Media Library / 媒体库

## Purpose / 目的
Find the user's photos in Jarvis's uploaded library or a connected device gallery. The `media-library` CLI covers uploaded photos only.

在 Jarvis 的已上传媒体库或已连接的设备相册中查找用户的照片。`media-library` 命令行工具只覆盖已上传的照片。

## Device Photos / 设备照片

Search connected galleries for explicit gallery requests, or when uploaded or attached images are insufficient for a personal photo request.

当有明确的相册请求，或已上传/已附加的图片不足以满足个人照片请求时，搜索已连接的相册。

1. **Discover.** Call `device.list`, then `device.describe`. Choose a reachable device advertising `photos.search` in `commands`: user-named first, then `is_request_origin`, then another phone. Clarify ambiguity.
   **发现。** 调用 `device.list`，再调用 `device.describe`。选择一台可达且在 `commands` 中声明了 `photos.search` 的设备：优先用户指名的设备，其次是 `is_request_origin`，再其次是另一部手机。有歧义时先澄清。
2. **Search.** Invoke `photos.search`, narrowing with its advertised filters; albums mean folders. Respect permission denial and retry after required setup succeeds. Follow pagination and report incomplete coverage rather than claiming no matches.
   **搜索。** 调用 `photos.search`，用它声明的过滤器收窄范围；相册即文件夹。尊重权限拒绝，在所需设置完成后重试。遵循分页，如果覆盖不完整要如实报告，而不是声称没有匹配。
3. **Upload.** Search returns metadata. For viewing or comparison, upload a shortlist using the advertised commands. Wait for completion and account for partial failures; acceptance is not completion.
   **上传。** 搜索返回的是元数据。如需查看或对比，使用其声明的命令上传一份入围清单。等待完成并处理部分失败的情况；被接受不等于已完成。
4. **Locate.** After completion, run `/opt/hatch/bin/media-library recent --limit 20` for paths/IDs. Match capture dates, filenames, and GPS against device results; use `get` for details and date/GPS searches for older or already-uploaded photos. Filenames or recency alone cannot prove identity. Read confidently matched images; report uncertainty rather than guessing paths.
   **定位。** 完成后，运行 `/opt/hatch/bin/media-library recent --limit 20` 获取路径/ID。将拍摄日期、文件名和 GPS 与设备结果比对；对较旧或已上传的照片用 `get` 查看详情、用日期/GPS 搜索。仅凭文件名或新旧无法证明是同一张。读取已确信匹配的图像；宁可报告不确定，也不要猜测路径。

## Tooling / 工具
Use `exec` to run `/opt/hatch/bin/media-library <subcommand> [options]` for uploaded photos.

对已上传的照片，用 `exec` 运行 `/opt/hatch/bin/media-library <subcommand> [options]`。

Subcommands:

子命令：

- `stats` — library overview (total count, date range, description status)
  `stats` — 媒体库概览（总数、日期范围、描述状态）
- `search [--query "<terms>"] [--after YYYY-MM-DD] [--before YYYY-MM-DD] [--lat <f64>] [--long <f64>] [--radius <km>] [--has-gps] [--path <prefix>] [--limit N]` — when `--query` is present, FTS ranked by BM25; otherwise structured filtering sorted by taken date
  `search [--query "<terms>"] [--after YYYY-MM-DD] [--before YYYY-MM-DD] [--lat <f64>] [--long <f64>] [--radius <km>] [--has-gps] [--path <prefix>] [--limit N]` — 当 `--query` 存在时，为按 BM25 排序的全文检索；否则为按拍摄日期排序的结构化过滤
- `recent [--limit N]` — latest uploads by upload time (default 20, max 50)
  `recent [--limit N]` — 按上传时间排列的最新上传（默认 20，最大 50）
- `scan [--ids <id1,id2,...>] [--query "<terms>"] [--after YYYY-MM-DD] [--before YYYY-MM-DD] [--limit N]` — richer summaries with full descriptions; requires at least one narrowing option (any of the flags above, including `--limit` alone)
  `scan [--ids <id1,id2,...>] [--query "<terms>"] [--after YYYY-MM-DD] [--before YYYY-MM-DD] [--limit N]` — 带完整描述的更丰富摘要；要求至少一个收窄选项（上述任一标志，包括单独的 `--limit`）
- `get <media_id>` — full detail for one photo (JSON)
  `get <media_id>` — 单张照片的完整详情（JSON）

Output contract:

输出契约：

- `stats`: plain text with `Total photos`, `Date range`, and description status counts (`ready`, `pending`, `failed`, `rejected`).
  `stats`：纯文本，包含 `Total photos`、`Date range` 和描述状态计数（`ready`、`pending`、`failed`、`rejected`）。
- `search`: each result line: `<media_id> | <path> | <date> | [<location> |] <summary_short>`, with optional `match:` snippet below.
  `search`：每行结果为 `<media_id> | <path> | <date> | [<location> |] <summary_short>`，下方可选附有 `match:` 摘录。
- `recent`: same compact format as `search`, without match snippets.
  `recent`：与 `search` 相同的紧凑格式，但没有匹配摘录。
- `scan`: text blocks per item: `media_id`, `path`, `date`, `location`, `summary`, `detail`.
  `scan`：每项一个文本块：`media_id`、`path`、`date`、`location`、`summary`、`detail`。
- `get`: JSON with `media_id`, `rel_path`, `mime_type`, `size_bytes`, `taken_at_local`, `taken_at_local_date`, `gps_latitude`, `gps_longitude`, `location_text`, `location_json`, `exif_json`, `description_status`, `description` (nested: `summary_short`, `summary_full`, `people_text`, `activity_text`, `objects_text`, `ocr_text`, `location_hint_text`).
  `get`：JSON，包含 `media_id`、`rel_path`、`mime_type`、`size_bytes`、`taken_at_local`、`taken_at_local_date`、`gps_latitude`、`gps_longitude`、`location_text`、`location_json`、`exif_json`、`description_status`、`description`（嵌套：`summary_short`、`summary_full`、`people_text`、`activity_text`、`objects_text`、`ocr_text`、`location_hint_text`）。

The `path` field in results is relative to the home directory.

结果中的 `path` 字段相对于主目录。

## Operating Rules / 操作规则

1. **Workflow.** `stats` on first use → `search` or `recent` → `scan` for richer detail on candidates → `get` for full info → `read` the image only if the user wants to see it or the description is insufficient. Most queries can be answered from descriptions alone.
   **工作流。** 首次使用先 `stats` → `search` 或 `recent` → 对候选结果用 `scan` 获取更丰富的细节 → 用 `get` 获取完整信息 → 仅当用户想看图像或描述不足时才 `read` 图像。大多数查询仅凭描述即可回答。
2. **Stats interpretation.** If total is 0, the uploaded library is empty or not set up yet — mention this briefly and check a connected device gallery, if available, for personal photo requests. If total is low (under ~20), note the library is small. If many are pending, text search will miss undescribed items — prefer `recent` or date-filtered searches instead.
   **统计解读。** 如果总数为 0，说明已上传媒体库为空或尚未设置——简要提及这一点，并在可用时检查已连接的设备相册来满足个人照片请求。如果总数偏低（约 20 以下），说明媒体库较小。如果大量条目处于 pending 状态，文本搜索会漏掉未描述的条目——改用 `recent` 或按日期过滤的搜索。
3. **Opportunistic vs primary use.** When the media library may help answer the question, do a quick probe — `stats` plus a `search` or `recent` — and move on if nothing useful comes back. When the user is explicitly asking to find, show, identify, date, or locate a photo, treat the media library as a primary source and iterate harder with multiple searches, `scan`, and `get`.
   **机会性使用与主要用途。** 当媒体库可能有助于回答问题时，做一次快速探测——`stats` 加一次 `search` 或 `recent`——如果没有有用的结果就继续。当用户明确要求查找、展示、识别、确定日期或定位某张照片时，把媒体库当作主要来源，用多次搜索、`scan` 和 `get` 更下功夫地迭代。
4. **Uploaded-library searches are cheap local PostgreSQL lookups** (typically milliseconds). Don't hesitate to run many queries — try different terms, filters, and angles until you find what the user is looking for.
   **已上传媒体库的搜索是廉价的本地 PostgreSQL 查询**（通常毫秒级）。尽管多跑几次查询——换不同的词、过滤器和角度，直到找到用户要找的东西。
5. **FTS semantics.** Bare terms are joined with implicit OR, ranked by BM25 (more matching terms = higher rank). AND/OR/NOT are supported. Terms are prefix-matched after punctuation stripping. Adding more OR terms makes results *broader*, not narrower — to narrow, use AND, date filters, or GPS filters instead.
   **全文检索语义。** 裸词之间以隐式 OR 连接，按 BM25 排序（匹配词越多排名越高）。支持 AND/OR/NOT。词项在去除标点后按前缀匹配。增加更多 OR 词会使结果*更宽*而非更窄——要收窄，请改用 AND、日期过滤或 GPS 过滤。
6. **Start with the user's words**, not guessed content. Broaden with synonyms, context from memory or conversation, or related terms only after direct queries come up short or return irrelevant results.
   **从用户原话出发**，而不是猜测的内容。只有在直接查询结果不足或无关时，才用同义词、记忆或对话中的上下文、相关词来扩展。
7. **Location.** Geocoded city/county, state, and country names are in the search index — `search --query "Tokyo"` works. For GPS proximity, call `map.geocode` with the place as `query`, then use the returned coordinates with `search --lat <lat> --long <lon> --radius <km>` (scale: `0.1` for a spot, `5` for a neighborhood, `50` for a metro area).
   **位置。** 已地理编码的城市/县、州和国家名称在搜索索引中——`search --query "Tokyo"` 可用。对于 GPS 邻近搜索，先以地点为 `query` 调用 `map.geocode`，然后用返回的坐标配合 `search --lat <lat> --long <lon> --radius <km>`（尺度：一个地点用 `0.1`，一个街区用 `5`，一个都市圈用 `50`）。
8. **People.** Use physical descriptions ("woman in red dress"), not names.
   **人物。** 使用外貌描述（"穿红裙子的女人"），而不是名字。
9. **Memory.** Cross-reference memory for dates and locations when the user mentions events or trips.
   **记忆。** 当用户提到事件或旅行时，用记忆交叉核对日期和位置。
10. **When finding a specific photo**, show what you found and confirm it's the right one.
    **查找特定照片时**，展示你找到的内容并确认是正确的那张。
