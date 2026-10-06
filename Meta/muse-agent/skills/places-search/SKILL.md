---
name: "places_search"
title: "Places Search"
description: "Find, compare, and share details on physical places near the user or in a specified area, including restaurants, cafes, bars, hotels, parks, attractions, shops, and businesses with local services. Not for itineraries, choosing a city or region, dated events or showtimes, or directions."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Places / 地点

Use `browser.search` for all place discovery and name/address lookup. Run
`places <command>` through `muse.exec` for richer details and photo-based place
detection. Never load or search for a `places` tool namespace. If a search
comes back empty or weak, refine the `browser.search` query per the rules
below. Print CLI JSON to stdout for you to read, not to show the user. Use
`--help` when needed. No sign-in is required.

所有地点发现以及名称/地址查询都使用 `browser.search`。通过 `muse.exec` 运行 `places <command>` 以获取更丰富的详情和基于照片的地点识别。绝不要加载或搜索 `places` 工具命名空间。若某次搜索结果为空或较弱，按下述规则优化 `browser.search` 查询。CLI JSON 打印到 stdout 供你自己阅读，不要展示给用户。需要时使用 `--help`。无需登录。

## Common flows / 常见流程

### Find places / 查找地点

Use `browser.search` for standing places people can visit or reference on a
map: restaurants, cafes, bars, hotels, parks, trails, attractions, shops, and
local services. A follow-up like "anything cheaper?" or "Italian instead?"
refines the preceding search.

人们可以到访或在地图上查阅的常设地点（餐厅、咖啡馆、酒吧、酒店、公园、步道、景点、商铺和本地服务商）都使用 `browser.search`。"有更便宜的吗？"或"换成意大利菜？"这类后续请求是对前一次搜索的细化。

- Put the complete user intent in `primary_query.query`. Include the place
  category or venue name, area, radius, and ranking preference in natural
  language. For example, search for "best sushi restaurants in San Francisco
  within five miles", not just "sushi".
  把完整的用户意图放进 `primary_query.query`。用自然语言包含地点类别或场所名称、区域、半径和排序偏好。例如，搜索 "best sushi restaurants in San Francisco within five miles"，而不只是 "sushi"。
- Put "near me" or "nearby" in a browser.search query only when
  `message_location` or `last_seen_location` arrived with this turn; the
  search service cannot see a location you learned any other way, so those
  words can land results in the wrong city. When the location came from the
  user, a device read, or their home on file, put the area's name in the
  query. If no location is available, ask the user; never invent or guess a
  location.
  仅当 `message_location` 或 `last_seen_location` 随本轮一起到来时，才把 "near me" 或 "nearby" 写进 browser.search 查询；搜索服务看不到你以其他方式获知的位置，这些词可能把结果落到错误的城市。当位置来自用户、设备读取或存档的家庭住址时，把区域名称写进查询。若没有任何可用位置，向用户询问；绝不臆造或猜测位置。
- One call covers one area. When the user names multiple cities or
  neighborhoods, run a separate call for each and cover each in the answer.
  一次调用覆盖一个区域。当用户点出多个城市或街区时，对每个分别运行一次调用，并在回答中逐一覆盖。
- For category discovery, include the category and area in the query. For a
  named venue, include its exact name and enough area context to disambiguate
  it.
  类别发现时，把类别和区域写进查询。具名场所则写进其准确名称和足够的区域上下文以消歧。
- For "best," trending, insider, or nuanced requests, use the web evidence and
  place results returned together by `browser.search`. Ground every named venue
  in a returned result.
  "最好的"、热门、内部人士或有细微差别的请求，使用 `browser.search` 一并返回的网络证据和地点结果。点名的每个场所都要有返回结果作依据。
- When comparable `avg_rating` values from `places details` are available for
  every shortlisted place, order them highest first but never print the rating.
  Otherwise order by relevance to the user's constraints.
  当入围的每个地点都有可比较的 `places details` `avg_rating` 数值时，按从高到低排序，但绝不打印评分。否则按与用户约束的相关度排序。
- Avoid showing permanently or temporarily closed places unless specifically
  asked for.
  除非被明确要求，避免展示永久或临时歇业的地点。
- Judge the returned result set rather than following rank blindly. Set aside
  wrong-category or wrong-area results. If the set is weak, retry with an exact
  venue name, a narrower subtype, or clearer area language.
  对返回的结果集自行判断，不要盲从排名。剔除类别错误或区域错误的结果。若结果集较弱，用确切的场所名称、更窄的子类或更明确的区域措辞重试。

### Get richer details / 获取更丰富的详情

- `places details` takes numeric place IDs only. It cannot search by name or
  address; use `browser.search` for that.
  `places details` 只接受数字形式的地点 ID。它不能按名称或地址搜索；那类需求使用 `browser.search`。
- Call `places details` only when `browser.search` explicitly returns an all-digit
  `place_id` for that result. Never derive an ID from a venue name or URL.
  仅当 `browser.search` 为该结果明确返回了全数字的 `place_id` 时才调用 `places details`。绝不要从场所名称或 URL 推导 ID。
- Prefer the `browser.search` response when it already contains enough detail. Use
  `places details` for richer or fresher hours, prices, photos, reviews, and
  offerings.
  `browser.search` 的响应已包含足够详情时优先使用它。`places details` 用于更丰富或更新的营业时间、价格、照片、评论和提供内容。
- If `browser.search` returns no numeric ID, retry once with the exact venue
  name and area. If that still returns no ID, answer from cited search evidence
  without details or a map.
  若 `browser.search` 没有返回数字 ID，用确切的场所名称和区域重试一次。若仍无 ID，就基于有引用的搜索证据作答，不给详情也不给地图。
- Batch several IDs into one call when comparing places. Pass `--motivation`
  with the user's intent so quotes and photos are ranked for it.
  比较多个地点时，把几个 ID 合并进一次调用。传入带有用户意图的 `--motivation`，使报价和照片按该意图排序。
- `places details` returns `{"<place_id>": {"details": {…, "rating":
  {"avg_rating", "num_rating"}}}}`, keyed by id.
  `places details` 返回 `{"<place_id>": {"details": {…, "rating":
  {"avg_rating", "num_rating"}}}}`，以 id 为键。

### Show places on a map / 在地图上展示地点

Before creating a map, run `places details` for the selected numeric IDs. Create
one map if at least one returned details record has both a numeric place ID and
coordinates. This applies to recommendations, comparisons, named-place
lookups, and other uses where a map could ground the user. See Response
formatting for which places go on it.

创建地图之前，先为选定的数字 ID 运行 `places details`。只要至少一条返回的详情记录同时具有数字地点 ID 和坐标，就创建一张地图。这适用于推荐、比较、具名地点查询以及其他地图能为用户提供依据的场景。哪些地点上地图，见 Response formatting。

Create one `local_map` widget with `widget.create` and this payload:

用 `widget.create` 创建一个 `local_map` 组件，载荷如下：

```json
{
  "kind": "local_map",
  "data": {
    "elements": [
      {
        "kind": "rich_place",
        "place_id": "<numeric ID>",
        "name": "<name from the same result>",
        "coordinate": {
          "latitude": 0.0,
          "longitude": 0.0
        }
      }
    ]
  }
}
```

- Replace the example coordinates with the values from the same keyed
  `places details` record. Use its canonical name. Omit a place whose details
  or coordinates are missing; never infer an ID or geocode a name.
  用同一条键匹配的 `places details` 记录中的数值替换示例坐标。使用其规范名称。详情或坐标缺失的地点直接省略；绝不推断 ID，也不要对名称做地理编码。
- If creation fails, answer in text. Do not retry or build an HTML or image map.
  若创建失败，用文本作答。不要重试，也不要构建 HTML 或图片地图。

### Identify a place from a photo / 从照片识别地点

Use `places detect` when captured frames and GPS need to resolve which place the
user is at. It returns ranked candidates as `{place_id, place_name, confidence}`.
Pass a returned ID to `places details` for more. The optional 4th `--location`
field is the connected Wi-Fi BSSID; include it verbatim when known to improve
Home/Work matching, and never echo a raw BSSID back to the user.

当需要用抓拍帧和 GPS 判定用户所在地点时，使用 `places detect`。它以 `{place_id, place_name, confidence}` 返回排序后的候选。把返回的 ID 传给 `places details` 获取更多信息。可选的第 4 个 `--location` 字段是所连 Wi-Fi 的 BSSID；已知时原样传入以改进家/公司匹配，但绝不要把原始 BSSID 回显给用户。

## Response formatting / 回复格式

- Never name a place that did not come back from a tool.
  绝不点名任何不是由工具返回的地点。
- Match the framing to the ask. When the user wants a recommendation, give them
  two to three. When the request is broad, offer a few named results as a
  starting point.
  叙述方式与提问匹配。用户要推荐时，给两到三个。请求宽泛时，给出几个具名结果作为起点。
- Name two to three places in text from the merged result set. Write each on
  its own line in exactly this shape: `**Name**, a sentence or two on why you
  chose it.` Open with one short line framing the answer before the places.
  从合并后的结果集中用文本点名两到三个地点。每个单独一行，严格采用这个形状：`**Name**, a sentence or two on why you chose it.`（名称后接一两句推荐理由）。在地点之前先用一行短句框定回答。
- Send the text as one message. Never add a second message after the map.
  文本作为一条消息发送。绝不在地图之后再发第二条消息。
- No addresses, raw place IDs, coordinates, star ratings, review counts, or
  price levels in text.
  文本中不出现地址、原始地点 ID、坐标、星级评分、评论数或价位。
- After a recommendation, note there are more options without itemizing; after
  a broad answer, offer to narrow to a recommendation.
  推荐之后，提示还有更多选项但不必逐一列举；宽泛回答之后，主动提出可以收窄成一份推荐。
- Cite factual claims sourced from `browser.search` using its normal citation
  format.
  来自 `browser.search` 的事实性论断，按其常规引用格式标注来源。
- Resolve every name you plan to use, closing line included, through
  `browser.search` before creating the map. Never name an unresolved place.
  在创建地图之前，把你打算使用的每个名称（包括收尾句）都通过 `browser.search` 确认过。绝不点名未经确认的地点。
- Build the map from the most relevant places returned that have a place ID
  and coordinates. Don't rely solely on named places in your text as an
  indication of what places should appear on the map.
  用返回结果中最相关、且具有地点 ID 和坐标的地点构建地图。不要只凭文本中点名的地点来决定地图上应出现哪些地点。
- The places you named come first on the map, in the order you named them,
  ahead of everything else.
  你点名的地点排在地图最前，顺序与你点名的顺序一致，优先于其他一切。
- Create one map per answer. Include the returned `embed_token`.
  每个回答只创建一张地图。包含返回的 `embed_token`。
- Never narrate or reference the map. No "I mapped things out for you"; "Take a look at the map", etc.
  绝不叙述或提及地图。不要说 "I mapped things out for you"、"Take a look at the map" 之类的话。

## Limits / 限制

- Do not write directions, turn-by-turn steps, maps links, or navigation
  controls.
  不写路线指引、逐向导航步骤、地图链接或导航控件。
- Do not estimate travel times or distances. Use only values returned by a tool.
  不估算行程时间或距离。只使用工具返回的数值。
