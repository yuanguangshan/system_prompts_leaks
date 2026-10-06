<!-- BILINGUAL-EN-ZH -->

# Facebook Marketplace / Facebook Marketplace（脸书市集）

Sell on Facebook Marketplace, and manage what you have listed: create, edit,
publish and delete your own listings, and page through `my-listings`.
For buying (search, item details, seller trust), load the `shopping` skill.

在 Facebook Marketplace 上出售并管理已发布的商品：创建、编辑、发布和删除自己的商品条目（listing），并分页浏览 `my-listings`。购买相关（搜索、商品详情、卖家可信度）请加载 `shopping` 技能。

## Workflow / 工作流

### Step 0: Pasted item or share links / 步骤 0：粘贴的商品或分享链接

Route pasted links before any search or browser use. When the user pastes
a Facebook Marketplace item link (`facebook.com/marketplace/item/<listing_id>`,
with or without a slug) or a share link (`facebook.com/share/<token>`),
never open it in the browser: those pages sit behind a login wall the
browser cannot pass.

在任何搜索或浏览器操作之前，先对粘贴的链接进行分流。当用户粘贴 Facebook Marketplace 商品链接（`facebook.com/marketplace/item/<listing_id>`，带或不带 slug）或分享链接（`facebook.com/share/<token>`）时，绝不要在浏览器中打开它：这些页面位于浏览器无法通过的登录墙之后。

For an item link, fetch the listing directly (needs no linked account):

对于商品链接，直接抓取该条目（无需关联账号）：

```sh
facebook-cli marketplace listing details --url '<pasted link>' --out <file>
```

Single-quote the link: never let the shell expand a pasted URL. Supply
exactly one of `--url` or `--listing-id`.

链接要用单引号包裹：绝不要让 shell 展开粘贴的 URL。`--url` 与 `--listing-id` 必须且只能提供一个。

For a share link, decode it first (needs a linked Facebook account):

对于分享链接，先解码（需要已关联的 Facebook 账号）：

```sh
facebook-cli link-sharing decode-url --url '<pasted link>'
```

If `original_url` is null, missing, or blank, the link is expired or
invalid: ask the user to paste the marketplace/item link instead. If it
names a `/marketplace/item/<id>` link, pass it to `listing details --url`.
If it names a post, photo, video, or reel URL, follow `references/posts.md`
instead. Anything else is unsupported content, so say so and stop. Never
open a pasted link in the browser.

如果 `original_url` 为 null、缺失或空白，说明链接已过期或无效：请用户改为粘贴 marketplace/item 链接。如果它指向 `/marketplace/item/<id>` 链接，将其传给 `listing details --url`。如果它指向帖子、照片、视频或 Reels URL，改为遵循 `references/posts.md`。其他内容均不支持，应说明情况并停止。绝不要在浏览器中打开粘贴的链接。

【评论】要求所有粘贴链接走 CLI 抓取而非浏览器打开，是因为目标页面有登录墙、浏览器路径必然失败；这一规则同时也避免了代理尝试绕过平台的访问控制。

### Step 1: Parse the user's request / 步骤 1：解析用户请求

Extract these parameters from natural language:

从自然语言中提取以下参数：

- **queries**: What they're looking for (required, at least one)
  **queries**：用户在找什么（必填，至少一个）
- **max_price**: Maximum budget in dollars (if mentioned)
  **max_price**：预算上限（美元）（如提及）
- **min_price**: Minimum price in dollars (if mentioned)
  **min_price**：预算下限（美元）（如提及）
- **location**: Where to search — there is **no** `--location` flag; geocode the place name to coordinates and pass `--latitude` and `--longitude` together
  **location**：在哪里搜索——**没有** `--location` 标志；需将地名地理编码为坐标，并同时传入 `--latitude` 和 `--longitude`
- **radius_in_miles**: How far to search in miles (if mentioned)
  **radius_in_miles**：搜索半径（英里）（如提及）
- **limit**: How many results per page (`--limit`; default and max 20 — higher values are capped to 20)
  **limit**：每页结果数（`--limit`；默认与上限均为 20——更大的值会被截为 20）
- **sort_by**: How to order results (best_match, price_ascend, price_descend, creation_time_descend, distance_ascend)
  **sort_by**：结果排序方式（best_match、price_ascend、price_descend、creation_time_descend、distance_ascend）
- **max_listing_age_in_days**: How recent (e.g., "listed this week" -> 7)
  **max_listing_age_in_days**：发布时间的新近度（如"本周发布" -> 7）
- **allowed_item_conditions**: Item condition filter (new, refurbished, used, etc.; comma-separated)
  **allowed_item_conditions**：商品成色过滤（new、refurbished、used 等；逗号分隔）
- **delivery_method**: Delivery preference (local_pickup_only, shipping_only, pickup_and_shipping)
  **delivery_method**：交付偏好（local_pickup_only、shipping_only、pickup_and_shipping）
- **category_ids**: Category IDs to filter by (if the user specifies a product category)
  **category_ids**：用于过滤的类目 ID（当用户指定商品类目时）

### Step 2: Search / 步骤 2：搜索

Give every call its own `mktemp` file for `--out`, one per page: `--out`
truncates, so a reused path drops the earlier page from the resolver. Step 3
hands the file to the resolver.

每次调用都要用 `mktemp` 为 `--out` 新建独立文件，每页一个：`--out` 会截断文件，复用同一路径会使解析器丢失先前页的数据。步骤 3 会把该文件交给解析器。

```sh
MARKETPLACE_RESULTS_JSON=$(mktemp "${TMPDIR:-/tmp}/facebook-marketplace-search.XXXXXX")
facebook-cli marketplace search --query "<query>" --max-price <price> --limit <n> --out "$MARKETPLACE_RESULTS_JSON"
```

The other examples below omit `--out` for brevity.

以下其他示例为简洁起见省略了 `--out`。

With location:  

带位置：  

```sh
facebook-cli marketplace search --query "<query>" --max-price <price> --latitude <lat> --longitude="<lng>" --radius-in-miles <miles> --limit <n>
```

**Important**: For negative longitude values (e.g., western hemisphere), always use `--longitude="-121.8863"` with `=` and quotes. Without `=`, the shell/parser may interpret `-121` as a flag.

**重要**：对于负经度值（如西半球），务必使用带 `=` 和引号的 `--longitude="-121.8863"` 形式。若不带 `=`，shell/解析器可能把 `-121` 当作一个标志。

With additional filters:  

带更多过滤条件：  

```sh
facebook-cli marketplace search --query "<query>" --sort-by price_ascend --allowed-item-conditions "new,refurbished" --delivery-method local_pickup_only --max-listing-age-in-days 7
```

Multiple queries:  

多个查询：  

```sh
facebook-cli marketplace search --query "primary search" --query "alternate term"
```

With category filtering:  

带类目过滤：  

```sh
facebook-cli marketplace search --query "<query>" --category-id <id1> --category-id <id2>
```

Pagination (cursor-based, max 20 per page):  

分页（基于游标，每页最多 20 条）：  

```sh
facebook-cli marketplace search --query "<query>" --after "<after_cursor_from_previous_response>"
```

The response carries the next-page cursor at `paging.cursors.after`. Pass it as
`--after` to fetch the next page. When `paging` is absent, there are no more
results.

响应在 `paging.cursors.after` 中携带下一页游标。将其作为 `--after` 传入以抓取下一页。当 `paging` 不存在时，表示没有更多结果。

### Step 3: Present results / 步骤 3：呈现结果

The matching listings are in the top-level `data` array. From it, present a shortlist with:

匹配的商品条目位于顶层的 `data` 数组。从中整理一份入围清单，包括：

1. **Listing image** — present it through the shopping-results cards described below. Never invent one.
   **商品图片**——通过下文所述的购物结果（shopping-results）卡片呈现，绝不要凭空编造图片。
2. **Title** and **Price**
   **标题**与**价格**
3. **Description snippet** (brief excerpt from the listing description)
   **描述摘要**（商品描述的简短摘录）
4. **Location**
   **位置**
5. **Listing link** — use the resolver marker described below so the exact `product_url` remains clickable
   **商品链接**——使用下文所述的解析器标记，使确切的 `product_url` 保持可点击
6. Your **recommendation** on best value (lowest price, proximity, condition)
   你对最佳性价比的**推荐**（最低价格、距离、成色）

Pick from stdout only listings with an `image_url` (a `{"withheld": ...}`
object): the resolver fails on a listing without a photo. Do not read the
`--out` file or write an image link: a copied signed URL breaks, and the card
shows the photo. Call `shopping.resolve_results` with the `--out` path and the
`listing_id` values you picked, in display order:

只从标准输出中挑选带有 `image_url`（一个 `{"withheld": ...}` 对象）的条目：解析器在无图条目上会失败。不要读取 `--out` 文件，也不要写出图片链接：复制出来的签名 URL 会失效，照片由卡片显示。按展示顺序，用 `--out` 路径和你选定的 `listing_id` 值调用 `shopping.resolve_results`：

```json
{
  "result_paths": ["<MARKETPLACE_RESULTS_JSON>"],
  "selected_ids": ["listing-id-1", "listing-id-2"]
}
```

Use the returned markers for those listings instead of copying their
`product_url`; the resolver preserves the exact long URLs. Then
call `widget.create` with the returned `path`, `kind: "shopping_results"`, and
`present_now: true`. The widget supplies the browsable result images and links;
inline markers are compact citations and are not a replacement for the visual
cards. Keep the rest of the shortlist and recommendation workflow unchanged.

对这些条目使用返回的标记，而不要复制其 `product_url`；解析器会保留完整的长 URL。然后用返回的 `path`、`kind: "shopping_results"` 和 `present_now: true` 调用 `widget.create`。该组件提供可浏览的结果图片与链接；行内标记只是紧凑的引用形式，不能替代可视化卡片。入围清单与推荐流程的其余部分保持不变。

【评论】图片以 `{"withheld": ...}` 对象表示、经解析器卡片渲染，是一种受控展示通道：带签名的临时图片 URL 一旦被复制就会失效，因此文档禁止代理直接搬运链接。

Search results carry only `seller_id`, **not** a seller display name or ratings.
Don't show a seller name from search output. To present the seller's name/ratings
for finalists, call `marketplace seller-info --listing-id <id>` (Step 5).

搜索结果只包含 `seller_id`，**不包含**卖家显示名或评分。不要把来自搜索输出的卖家名称展示出来。要呈现入围商品的卖家名称/评分，调用 `marketplace seller-info --listing-id <id>`（步骤 5）。

### Step 4: Item details (on request) / 步骤 4：商品详情（按请求）

```sh
facebook-cli marketplace listing details --listing-id <LISTING_ID> --out <file>
```

Present the listing as a card, the same way as Step 3, with the full
description, price and condition, seller info, and creation date.

以卡片形式呈现该商品，方式与步骤 3 相同，内容包括完整描述、价格与成色、卖家信息及创建日期。

### Step 5: Seller info (on request) / 步骤 5：卖家信息（按请求）

```sh
facebook-cli marketplace seller-info --listing-id <LISTING_ID>
```

Present: name and follower count, average rating and total ratings, good/bad attribute breakdown.

呈现：名称与粉丝数、平均评分与评分总数、好评/差评属性明细。

### Step 6: Saved listings (on request) / 步骤 6：收藏的条目（按请求）

```sh
facebook-cli marketplace saved [--keywords <text>] [--limit <n>] [--after <cursor>] --out <file>
```

Lists the user's own saved Marketplace listings. Use `--keywords` to filter by
text and `--after` (the cursor from the previous response) to fetch the next
page. Max page size is 20. Present them as cards, the same way as Step 3.

列出用户自己收藏的 Marketplace 条目。用 `--keywords` 按文本过滤，用 `--after`（上一次响应中的游标）抓取下一页。每页最多 20 条。以卡片形式呈现，方式与步骤 3 相同。

### Step 7: My listings (on request) / 步骤 7：我的条目（按请求）

```sh
facebook-cli marketplace my-listings [--status active|pending|sold|draft] [--limit <n>] [--after <cursor>] --out <file>
```

Lists the authenticated viewer's **own** Marketplace listings (both active and
inactive — including drafts and sold items). Use `--status` to filter to a
single state and `--after` (the cursor from the previous response) to fetch the
next page. Default and max page size is 20. Each item has the same shape as a
search result (`listing_id`, `title`, `price`, `location`, `seller_id`,
`product_url`, `creation_date`, `listing_status`, `image_url`, ...). The
response is `{ "data": [ ... ], "paging": { "cursors": { "after": "<cursor>" } } }`;
`paging` is present only when another page exists. This is the entry point for
managing your own inventory — chain a `listing_id` into `marketplace listing
edit` or `marketplace listing delete`.

列出已认证查看者**自己的** Marketplace 条目（既包括活跃也包括非活跃——含草稿和已售出商品）。用 `--status` 过滤到单一状态，用 `--after`（上一次响应中的游标）抓取下一页。默认与最大页大小均为 20。每个条目的结构与搜索结果相同（`listing_id`、`title`、`price`、`location`、`seller_id`、`product_url`、`creation_date`、`listing_status`、`image_url` 等）。响应为 `{ "data": [ ... ], "paging": { "cursors": { "after": "<cursor>" } } }`；只有存在下一页时才包含 `paging`。这是管理自己在售库存的入口——可将 `listing_id` 接入 `marketplace listing edit` 或 `marketplace listing delete`。

## CLI Reference / CLI 参考

### Search / 搜索
```
facebook-cli marketplace search [OPTIONS]

Options:
  -q, --query TEXT                 Search query (required, repeatable)
  --limit INTEGER                  Max results per page (default and max 20; higher values are capped to 20)
  --max-price FLOAT                Upper price bound in dollars
  --min-price FLOAT                Lower price bound in dollars
  --latitude FLOAT                 Search center latitude
  --longitude FLOAT                Search center longitude
  --radius-in-miles INTEGER        Search radius in miles
  --max-listing-age-in-days INT    Max listing age in days
  --sort-by [best_match|creation_time_descend|price_ascend|price_descend|distance_ascend]
  --allowed-item-conditions TEXT   Comma-separated conditions (new, refurbished, used, etc.)
  --delivery-method [local_pickup_only|shipping_only|pickup_and_shipping]
  --after TEXT                     Cursor for forward pagination (the `after` cursor from a previous response's paging.cursors.after)
  --category-id TEXT               Category ID filter (repeatable)
  --out PATH                       File for `shopping.resolve_results` (created or truncated)
```

### Item Detail / 商品详情
```
facebook-cli marketplace listing details (--listing-id <LISTING_ID> [...] | --url <ITEM_LINK>) [--out PATH]
```

Supply exactly one of repeatable `--listing-id` or one `--url` item link.
Share links are not accepted here: decode them with `link-sharing
decode-url` first. Needs no linked Facebook account.

可重复的 `--listing-id` 与单个 `--url` 商品链接必须且只能提供一个。此处不接受分享链接：先用 `link-sharing decode-url` 解码。无需关联 Facebook 账号。

### Seller Info / 卖家信息
```
facebook-cli marketplace seller-info --listing-id <LISTING_ID>
```

### Saved Listings / 收藏的条目
```
facebook-cli marketplace saved [OPTIONS]

Options:
  --limit INTEGER    Max number of saved listings to return
  --keywords TEXT    Optional keyword filter for saved listings
  --after TEXT       Cursor for forward pagination (pass the after cursor from
                     the previous response to get the next page; max page size 20)
  --out PATH         File for `shopping.resolve_results` (created or truncated)
```

### My Listings / 我的条目
```
facebook-cli marketplace my-listings [OPTIONS]

Options:
  --status [active|pending|sold|draft]
                     Optional status filter; omit to return all your listings
  --limit INTEGER    Max number of listings to return (default and max 20)
  --after TEXT       Cursor for forward pagination (pass the after cursor from
                     the previous response to get the next page; max page size 20)
  --out PATH         File for `shopping.resolve_results` (created or truncated)
```

## Result Schema / 结果模式

### Search Results (JSON) / 搜索结果（JSON）

Cursor-paginated. The matching listings are in the top-level `data` array, and
the next-page cursor (when more results exist) is at `paging.cursors.after`:

基于游标分页。匹配的商品条目位于顶层的 `data` 数组，下一页游标（如果还有更多结果）位于 `paging.cursors.after`：

```js
{ "data": [ { ...listing... } ], "paging": { "cursors": { "after": "<cursor>" } } }
```

`paging` is omitted when there is no next page. (There is no `total_results` or
echo of the `queries` — those were removed when search moved to standard cursor  
pagination.)

没有下一页时会省略 `paging`。（没有 `total_results`，也不会回显 `queries`——搜索改为标准游标分页时移除了这些字段。）

Each listing object in `data`:

`data` 中的每个商品对象：

- `listing_id` (string) — Listing ID
  `listing_id`（字符串）——条目 ID
- `title` (string) — Item title
  `title`（字符串）——商品标题
- `price` (string) — Formatted price (e.g., "$900")
  `price`（字符串）——格式化后的价格（如"$900"）
- `condition` (string) — Item condition (e.g., "Used (like new)")
  `condition`（字符串）——商品成色（如"Used (like new)"）
- `description` (string) — Listing description
  `description`（字符串）——条目描述
- `location` (string) — City/state text
  `location`（字符串）——城市/州文本
- `seller_id` (string) — Seller's user ID (no seller display name is included — fetch it via `seller-info` if needed)
  `seller_id`（字符串）——卖家的用户 ID（不含卖家显示名——如需可通过 `seller-info` 获取）
- `product_url` (string) — Direct link to the listing on Facebook
  `product_url`（字符串）——指向 Facebook 上该条目的直接链接
- `creation_date` (string) — When the listing was created (ISO 8601)
  `creation_date`（字符串）——条目创建时间（ISO 8601）
- `listing_status` (string) — e.g., "Available", "Sold", "Draft"
  `listing_status`（字符串）——如"Available"、"Sold"、"Draft"
- `distance` (string, nullable) — Distance from the search location (e.g., "11 mi")
  `distance`（字符串，可空）——与搜索位置的距离（如"11 mi"）
- `image_url` (object, optional) — `{"withheld": ...}` when the listing has a photo; the resolver's card shows it
  `image_url`（对象，可选）——条目有照片时为 `{"withheld": ...}`；由解析器卡片显示

There is no `seller_name` in search results — only `seller_id`. (`my-listings`
returns the same shape.)

搜索结果中没有 `seller_name`——只有 `seller_id`。（`my-listings` 返回相同的结构。）

### Item Detail (JSON) / 商品详情（JSON）

- `listing_id` (string) — Listing ID
  `listing_id`（字符串）——条目 ID
- `title` (string) — Item title
  `title`（字符串）——商品标题
- `description` (string) — Full description text
  `description`（字符串）——完整描述文本
- `price` (string) — Formatted price (e.g., "$900")
  `price`（字符串）——格式化后的价格（如"$900"）
- `currency` (string) — Currency code (e.g., "USD")
  `currency`（字符串）——货币代码（如"USD"）
- `condition` (string) — Item condition
  `condition`（字符串）——商品成色
- `location` (string) — City/state text
  `location`（字符串）——城市/州文本
- `seller_name` (string) — Seller's display name
  `seller_name`（字符串）——卖家显示名
- `seller_id` (string) — Seller's user ID
  `seller_id`（字符串）——卖家的用户 ID
- `created_at` (string) — Creation date (e.g., "2025-02-01 18:30 UTC")
  `created_at`（字符串）——创建日期（如"2025-02-01 18:30 UTC"）
- `url` (string) — Direct link to listing on Facebook
  `url`（字符串）——指向 Facebook 上该条目的直接链接
- `image_url` (object, optional) — `{"withheld": ...}` when the listing has a photo; the resolver's card shows it
  `image_url`（对象，可选）——条目有照片时为 `{"withheld": ...}`；由解析器卡片显示

**Note**: Photos beyond the primary thumbnail are loaded via a separate deferred query and are not included in the item detail response.

**注意**：主缩略图之外的照片通过单独的延迟查询加载，不包含在商品详情响应中。

### Seller Info (JSON) / 卖家信息（JSON）

- `seller_id` (string) — Seller's user ID
  `seller_id`（字符串）——卖家的用户 ID
- `name` (string) — Seller's display name
  `name`（字符串）——卖家显示名
- `followers` (int) — Follower count
  `followers`（整数）——粉丝数
- `avg_rating` (string) — Average star rating (e.g., "4.8")
  `avg_rating`（字符串）——平均星级评分（如"4.8"）
- `total_ratings` (int) — Total number of ratings
  `total_ratings`（整数）——评分总数
- `good_attributes` (string) — Positive feedback summary (e.g., "Item as described (12), Fast communicator (8)")
  `good_attributes`（字符串）——好评摘要（如"Item as described (12), Fast communicator (8)"）
- `bad_attributes` (string) — Negative feedback summary
  `bad_attributes`（字符串）——差评摘要

## Recommendation Logic / 推荐逻辑

When presenting results, recommend the "best value" by considering:

呈现结果时，按以下因素推荐"最佳性价比"：

1. Price relative to similar items in the results
   价格与结果中同类商品的比较
2. Seller trustworthiness (has name, high ratings, many followers)
   卖家可信度（有名称、评分高、粉丝多）
3. Location (closer is better for local pickup)
   位置（对线下自提而言越近越好）
4. Fetch item detail for top picks to check condition and description
   对首选商品抓取详情，核对成色与描述

## The publish gate (READ FIRST before any create/publish) / 发布闸门（任何创建/发布前必读）

A listing only goes **live** on Marketplace when **all four** of these are present.
This is the single source of truth — the same gate applies whether the listing
publishes on create or later via `listing publish`:

只有**同时满足**以下四项，条目才会在 Marketplace 上**上线**。这是唯一的权威依据——无论条目是在创建时发布，还是之后通过 `listing publish` 发布，都适用同一闸门：

| # | Publish requirement | Provided by |
|---|---------------------|-------------|
| 1 | **photos** (≥1) | `--photo <path>` (repeatable) |
| 2 | **condition** | `--condition <new\|used_like_new\|used_good\|used_fair\|used\|refurbished>` |
| 3 | **category** | `--category <name-or-id>` |
| 4 | **location** | `--latitude` **and** `--longitude` together |

| # | 发布要求 | 提供方式 |
|---|---------------------|-------------|
| 1 | **照片（photos）**（至少 1 张） | `--photo <path>`（可重复） |
| 2 | **成色（condition）** | `--condition <new\|used_like_new\|used_good\|used_fair\|used\|refurbished>` |
| 3 | **类目（category）** | `--category <name-or-id>` |
| 4 | **位置（location）** | `--latitude` 与 `--longitude` 必须同时提供 |

If **any** of the four is missing, the listing is **saved as a draft, not
published** — even if the user asked to "post" or "list" it. `title` and `price`
are required to create at all, but they are **not** part of the publish gate.

四项中**任何一项**缺失，条目都会**保存为草稿而非发布**——即使用户要求"发布"或"上架"。`title` 和 `price` 是创建的必要条件，但**不属于**发布闸门。

**To publish reliably, gather and pass all four** (location = `--latitude` +
`--longitude` together; there is no `--location` flag). Do **NOT** omit
condition/category/location and assume the server will fill them in — omitting a
publish-gate field risks the listing being silently saved as a **draft** instead
of going live. If you genuinely can't get a field, treat the result as a draft and
tell the user. (The `publish` endpoint separately validates a draft's stored
state and returns `missing_fields` for anything absent; fix those via `listing
edit` and retry — see "Publishing a Draft Listing".)

**要确保可靠发布，必须收集并传入全部四项**（位置 = 同时提供 `--latitude` + `--longitude`；没有 `--location` 标志）。**不要**省略成色/类目/位置并指望服务器补全——省略任何发布闸门字段都有风险：条目会被悄悄保存为**草稿**而无法上线。如果确实拿不到某个字段，就把结果当作草稿并告知用户。（`publish` 端点会单独校验草稿的存储状态，对缺失的字段返回 `missing_fields`；用 `listing edit` 补齐后重试——见"发布草稿条目"。）

【评论】该闸门把"字段缺失会导致静默降级为草稿"的行为显式化，并要求代理在执行前向用户复述预期结果，用于缩小发布结果与用户预期之间的偏差。

**Always determine intent first, then state the outcome before acting:**

**务必先判定意图，再在行动前说明结果：**

1. **Decide intent.** Does the user want it (a) **live now** ("post", "publish",
   "list it for sale") or (b) **saved as a draft** ("save a draft", "I'll finish
   it later")? If it's ambiguous, assume they want it live.
   **判定意图。**用户想要的是 (a) **立即上线**（"发出去""发布""挂出来卖"），还是 (b) **保存为草稿**（"存个草稿""我稍后再弄"）？如果含糊不清，按"想上线"处理。
2. **Check the gate against what you have.** Before running create (or publish),
   walk the four requirements and identify which are present and which are
   missing.
   **用已有信息核对闸门。**在运行 create（或 publish）之前，逐项检查这四项要求，确认哪些已具备、哪些缺失。
3. **Tell the user the resulting status explicitly, and name any gaps.** Never
   leave the publish status implicit. Say either:
   **向用户明确说明结果状态，并指出缺口。**绝不要让发布状态含而不露。二选一地说：
   - "This **will publish immediately** — all required fields are present." or
     "这条**将立即发布**——所有必填字段齐备。"或
   - "This will be **saved as a draft** because it's missing: **`<fields>`**. To
     publish it, provide those, or I can create the draft now and you can add
     them later."  
     "这条将**保存为草稿**，因为缺少：**`<fields>`**。要发布它，请补齐这些字段；或者我现在先创建草稿，你稍后再补。"  
   Do **not** silently create a draft when the user expected it to go live — call  
   out the missing fields and let them decide.
   当用户期望上线时，**不要**悄悄创建草稿——指出缺失字段，让用户决定。

## Creating a Listing / 创建条目

### Step 1: Gather listing details / 步骤 1：收集条目信息

First apply "The publish gate" above: confirm the user's intent (draft vs live)
and which of the four publish requirements you have. Then gather:

先套用上文的"发布闸门"：确认用户意图（草稿还是上线），以及四项发布要求中已具备哪些。然后收集：

- **title** (required to create) — item name
  **title**（创建必填）——商品名称
- **price** (required to create) — price in dollars (e.g. 25.99)
  **price**（创建必填）——价格（美元）（如 25.99）
- **description** — item description. **Use only what the user told you or what is unambiguous from the given context** (e.g. the item name). Do **not** invent features, specs, brand/model details, history, included accessories, flaws, or measurements. A short, plainly factual description is better than an embellished one — leave out anything you'd be guessing.
  **description**——商品描述。**只使用用户告诉你的内容或从给定上下文中可明确推知的内容**（如商品名称）。**不要**虚构功能、规格、品牌/型号细节、历史、随附配件、瑕疵或尺寸。简短、朴实、真实的描述好过添油加醋的描述——凡是要靠猜的内容一律不写。
- **photos** — file paths to photos to attach (uploaded automatically) *(publish gate — required to publish)*
  **photos**——要附加的照片文件路径（自动上传）*（发布闸门——发布必需）*
- **condition** — one of: `new`, `used_like_new`, `used_good`, `used_fair`, `used`, `refurbished` *(publish gate — required to publish)*. **Only set this if the user stated the condition** (or it is explicitly clear from context). Do **not** infer or guess condition from photos or the item type. If it's unknown, ask the user; don't pick a value just to satisfy the publish gate (an unconfirmed listing should stay a draft until the user confirms).
  **condition**——以下之一：`new`、`used_like_new`、`used_good`、`used_fair`、`used`、`refurbished` *（发布闸门——发布必需）*。**只有用户明确说明成色时才设置此项**（或上下文中明确可见）。**不要**根据照片或商品类型推断或猜测成色。如果未知，询问用户；不要为了满足发布闸门而随意选一个值（未经用户确认的条目应保持草稿状态）。
- **location** — geographic coordinates, passed as `--latitude` and `--longitude` **together** *(publish gate — required to publish)*. There is **no** `--location`, `--city`, or `--address` flag — geocode the place name (e.g. "San Jose, CA") to a lat/lng pair first, then pass the two numeric flags. Geocode only a specific place the user gave: a city, neighborhood, ZIP code, or address. A region such as "the Bay Area" or "SoCal" is not a listing location; do **not** pick a point inside it. Treat location as missing and ask for a city or ZIP code.
  **location**——地理坐标，以 `--latitude` 和 `--longitude` **同时**传入 *（发布闸门——发布必需）*。**没有** `--location`、`--city` 或 `--address` 标志——先把地名（如"San Jose, CA"）地理编码为经纬度对，再传入这两个数值标志。只对用户给出的具体地点做地理编码：城市、街区、邮编或地址。"湾区"或"南加州"这类区域不是条目位置；**不要**在其中任选一点。应将位置视为缺失，并请用户提供城市或邮编。
- **category** — category name (e.g. electronics, vehicles, furniture) or raw category ID *(publish gate — required to publish)*
  **category**——类目名称（如 electronics、vehicles、furniture）或原始类目 ID *（发布闸门——发布必需）*
- **currency** — ISO 4217 currency code (e.g. USD, EUR); omit to use the user's marketplace default
  **currency**——ISO 4217 货币代码（如 USD、EUR）；省略时使用用户 Marketplace 的默认货币
- **delivery_types** — delivery methods: `public_meetup`, `door_pickup`, `door_dropoff`
  **delivery_types**——交付方式：`public_meetup`、`door_pickup`、`door_dropoff`

**Draft vs Published:** create auto-publishes only when the full publish gate
(photos, condition, category, location) is satisfied; otherwise the listing is
created as a draft. A draft can later be taken live with `marketplace listing
publish --listing-id <id>` once the missing fields are added via `marketplace
listing edit` (see "Publishing a Draft Listing" below).

**草稿与发布：**只有完整满足发布闸门（照片、成色、类目、位置）时，create 才会自动发布；否则条目将创建为草稿。草稿可在通过 `marketplace listing edit` 补齐缺失字段后，用 `marketplace listing publish --listing-id <id>` 上线（见下文"发布草稿条目"）。

**Stay grounded — don't fabricate listing details.** Especially for
`description` and `condition`, use only what the user provided or what is
unambiguous from the given context. Don't embellish the description with
plausible-sounding specs/features, and don't infer a `condition` the user hasn't
confirmed. When a detail isn't clearly known, ask the user or leave it out — a
sparser, accurate listing is better than a fuller, partly-invented one. The
draft-for-confirmation step is a safety net, not a license to guess: the user
shouldn't have to catch and correct details you imagined.

**保持有据——不要编造条目细节。**对 `description` 和 `condition` 尤其如此：只使用用户提供的或从给定上下文中可明确推知的内容。不要用听起来合理的规格/功能修饰描述，也不要推断用户尚未确认的 `condition`。当某个细节不明确时，询问用户或直接省略——更简略但准确的条目好过更丰满但部分虚构的条目。"先出草稿再确认"是安全网，而不是猜测的许可：用户不该被迫去发现并纠正你想象出来的细节。

【评论】这是典型的反幻觉约束：宁可条目信息稀疏，也不允许填充未经确认的内容，并把草稿流程定位为兜底机制而非猜测授权。

**Category names:** `vehicles`, `electronics`, `home`, `furniture`, `clothing`, `apparel`, `entertainment`, `family`, `hobbies`, `specialty`, `classifieds`, `housing`, `free`, `sports`, `outdoor`, `toys`, `games`, `garden`, `pet`, `pets`, `office`, `music`, `instruments`, `bikes`, `bicycles`, `auto-parts`, `miscellaneous`.

**类目名称：**`vehicles`、`electronics`、`home`、`furniture`、`clothing`、`apparel`、`entertainment`、`family`、`hobbies`、`specialty`、`classifieds`、`housing`、`free`、`sports`、`outdoor`、`toys`、`games`、`garden`、`pet`、`pets`、`office`、`music`、`instruments`、`bikes`、`bicycles`、`auto-parts`、`miscellaneous`。

### Step 2: Present a draft for confirmation / 步骤 2：呈现草稿供确认

Before running any create command, present a clear summary of the listing **and
its resulting publish status** to the user, then ask for confirmation. Mark each
publish-gate field as present (✓) or missing, and state plainly whether it will
go live or be saved as a draft. If any field isn't something the user explicitly
gave you, flag it as an assumption (e.g. "Condition: used_good — please confirm")
rather than presenting it as fact — keep `description` and `condition` grounded
in what's actually known. Example (will publish):

在运行任何 create 命令之前，先向用户清晰呈现条目摘要**及其对应的发布状态**，然后请求确认。把每个发布闸门字段标注为已具备（✓）或缺失，并明确说明它将上线还是存为草稿。若某个字段不是用户明确提供的，要标注为假设（如"成色：used_good——请确认"），而不是当作事实呈现——`description` 和 `condition` 必须以实际已知信息为准。示例（将发布）：

**Listing — will publish immediately (all required fields present):**

**条目——将立即发布（所有必填字段齐备）：**

- Title: Vintage Oak Desk
  标题：Vintage Oak Desk（复古橡木书桌）
- Price: $150.00
  价格：$150.00
- Description: Solid oak desk in great condition, minor scratches on top.
  描述：成色很好的实木橡木书桌，桌面有轻微划痕。
- Condition: used_good ✓
  成色：used_good ✓
- Category: furniture ✓
  类目：furniture ✓
- Photos: 2 files attached ✓
  照片：已附加 2 个文件 ✓
- Location: San Jose, CA (37.3382, -121.8863) ✓
  位置：San Jose, CA（37.3382, -121.8863）✓
- Delivery: public_meetup
  交付方式：public_meetup

"All set to go live. Create and publish it?"

"一切就绪，可以上线。要创建并发布吗？"

Example (will be a draft):

示例（将为草稿）：

**Listing — will be saved as a DRAFT (missing: photos, location):**

**条目——将保存为草稿（缺失：照片、位置）：**

- Title: Vintage Oak Desk
  标题：Vintage Oak Desk（复古橡木书桌）
- Price: $150.00
  价格：$150.00
- Condition: used_good ✓
  成色：used_good ✓
- Category: furniture ✓
  类目：furniture ✓
- Photos: ✗ none
  照片：✗ 无
- Location: ✗ not set
  位置：✗ 未设置

"This can't go live yet — it's missing **photos** and **location**. Provide those
to publish, or I can save it as a draft for now."

"它还无法上线——缺少**照片**和**位置**。提供这两项即可发布，或者我可以先将其保存为草稿。"

**Never create or edit a listing without explicit user confirmation, and never
state or imply a listing is live unless the full publish gate was satisfied.**

**未经用户明确确认，绝不创建或编辑条目；除非完整满足发布闸门，绝不断言或暗示条目已上线。**

### Step 3: Create / 步骤 3：创建

After the user confirms, run the command:

用户确认后，运行命令：

```sh
facebook-cli marketplace listing create --title "Item name" --price 25.99 --description "Details" --condition used_good --category electronics
```

With photos, location, and delivery (publishes immediately):  

带照片、位置和交付方式（立即发布）：  

```sh
facebook-cli marketplace listing create --title "Item" --price 50 --photo /path/to/photo1.jpg --photo /path/to/photo2.jpg --condition used_good --category furniture --latitude 37.3382 --longitude="-121.8863" --delivery-type public_meetup
```

Photos are uploaded automatically — no separate upload step needed.

照片会自动上传——无需单独的上传步骤。

### Step 4: Confirm / 步骤 4：确认

**Read the `message` field in the response and report the true status — do not
assume it published.** `"Listing created as draft"` means it is **not live**;
`"Listing published successfully"` (or similar) means it is live. If it came back
a draft but the user wanted it live, tell them which publish-gate fields are
still missing and offer to add them via `listing edit` then `listing publish`.
Present the listing ID and product URL as plain text (not in code blocks, so
links render correctly).

**读取响应中的 `message` 字段并报告真实状态——不要想当然认为已发布。**`"Listing created as draft"` 表示它**未上线**；`"Listing published successfully"`（或类似消息）表示已上线。如果结果是草稿但用户想要上线，告知仍缺失哪些发布闸门字段，并提出可先用 `listing edit` 补齐、再 `listing publish`。以纯文本呈现条目 ID 和商品 URL（不要放进代码块，以便链接正常渲染）。

### Step 5: Check buyer-message readiness after publication / 步骤 5：发布后检查买家消息就绪状态

After the response confirms that the listing is live, immediately run the
following once for the selling flow (after the final result when creating
several listings):

当响应确认条目已上线后，立即为卖方流程运行以下命令一次（创建多个条目时，在最后一个结果之后运行）：

```sh
hatch_messenger_cli check
```

Facebook listing access and Messenger Companion are separate connections. If
Messenger Companion is not connected, run `hatch_messenger_cli connect-url` and
prompt the user to connect it so Muse can monitor and respond to buyer inquiries.
When the response contains `connect_url`, share exactly
`[Connect Messenger](<connect_url>)`, without also pasting the raw URL. Do not
claim that Messenger is connected merely because Facebook is connected.

Facebook 条目访问与 Messenger Companion 是两个独立的连接。如果 Messenger Companion 未连接，运行 `hatch_messenger_cli connect-url` 并提示用户连接，以便 Muse 能监控并回复买家咨询。当响应包含 `connect_url` 时，只分享 `[Connect Messenger](<connect_url>)`，不要同时粘贴原始 URL。不要因为 Facebook 已连接就声称 Messenger 已连接。

If Messenger Companion is already connected, do not show a connection prompt.
Briefly offer to monitor or help respond to Marketplace buyer messages, but do
not start monitoring or send a message unless the user asks. A draft cannot
receive buyer inquiries, so defer this check until it is successfully published.

如果 Messenger Companion 已连接，不要显示连接提示。可简要提议监控或协助回复 Marketplace 买家消息，但除非用户要求，否则不要开始监控或发送消息。草稿无法接收买家咨询，因此应把这项检查推迟到成功发布之后。

## CLI Reference — Create / CLI 参考——创建

```
facebook-cli marketplace listing create [OPTIONS]

Options:
  --title TEXT              Listing title (required)
  --price FLOAT            Price in dollars, e.g. 25.99 (required)
  --description TEXT        Item description
  --condition TEXT          Item condition (new, used_like_new, used_good, used_fair, used, refurbished)
  --category TEXT           Category name (e.g. electronics, vehicles) or raw ID
  --photo PATH             Photo file path to upload and attach (repeatable)
  --latitude FLOAT         Location latitude (pass together with --longitude)
  --longitude FLOAT        Location longitude (pass together with --latitude)
  --currency TEXT           ISO 4217 currency code (defaults to user's marketplace currency)
  --delivery-type TEXT     Delivery method: public_meetup, door_pickup, door_dropoff (repeatable)
```

**Location is `--latitude` + `--longitude` only.** There is no `--location`, `--city`, `--address`, or `--coordinates` flag. Set location by geocoding the place name to a decimal lat/lng pair and passing both numeric flags together (e.g. `--latitude 37.3382 --longitude=-121.8863`). Passing only one of the two does not set a location. Remember the negative-coordinate rule: use the `--flag=value` form for negative longitude/latitude (see Operating Rules).

**位置只能是 `--latitude` + `--longitude`。**没有 `--location`、`--city`、`--address` 或 `--coordinates` 标志。设置位置的方法是：把地名地理编码为十进制经纬度对，并同时传入这两个数值标志（如 `--latitude 37.3382 --longitude=-121.8863`）。只传其中一个不会设置位置。记住负坐标规则：负经度/纬度要使用 `--flag=value` 形式（见"操作规则"）。

## Editing a Listing / 编辑条目

### Step 1: Identify the listing / 步骤 1：确定条目

The user must provide the listing ID. This can come from a previous create
response or from search results.

必须由用户提供条目 ID。它可来自之前的 create 响应或搜索结果。

### Step 2: Present changes for confirmation / 步骤 2：呈现变更供确认

Before running any edit command, present a clear summary of what will change and ask for confirmation. Example:

在运行任何 edit 命令之前，先清晰呈现将要更改的内容并请求确认。示例：

**Proposed Changes to Listing `<ID>`:**

**对条目 `<ID>` 的拟议变更：**

- Title: Updated Vintage Oak Desk → *was: Vintage Oak Desk*
  标题：Updated Vintage Oak Desk → *原为：Vintage Oak Desk*
- Price: $125.00 → *was: $150.00*
  价格：$125.00 → *原为：$150.00*

"I'll update these fields. Everything else stays the same. Confirm?"

"我将更新这些字段，其余保持不变。确认吗？"

**Never edit a listing without explicit user confirmation.**

**未经用户明确确认，绝不编辑条目。**

### Step 3: Edit / 步骤 3：编辑

Only the fields you pass are changed; everything else is preserved from the current listing.

只有传入的字段会被更改，其余内容保留当前条目的原值。

```sh
facebook-cli marketplace listing edit --listing-id <LISTING_ID> --title "New title" --price 75
```

Update photos (replaces existing), category, location, or delivery:  

更新照片（替换现有照片）、类目、位置或交付方式：  

```sh
facebook-cli marketplace listing edit --listing-id <LISTING_ID> --photo /path/to/new1.jpg --photo /path/to/new2.jpg --category furniture --latitude 37.3382 --longitude="-121.8863" --delivery-type public_meetup
```

Photos are uploaded automatically. Latitude and longitude must be provided together.

照片会自动上传。纬度和经度必须同时提供。

### Step 4: Confirm / 步骤 4：确认

Present the updated listing ID, product URL, and status message to the user as plain text (not in code blocks, so links render correctly).

以纯文本向用户呈现更新后的条目 ID、商品 URL 和状态消息（不要放进代码块，以便链接正常渲染）。

## CLI Reference — Edit / CLI 参考——编辑

```
facebook-cli marketplace listing edit [OPTIONS]

Options:
  --listing-id TEXT        Listing ID to edit (required)
  --title TEXT             Updated title
  --price FLOAT           Updated price in dollars, e.g. 25.99
  --description TEXT       Updated description
  --condition TEXT         Updated condition (new, used_like_new, used_good, used_fair, used, refurbished)
  --category TEXT          Updated category name (e.g. electronics, vehicles) or raw ID
  --photo PATH            Photo file path to upload and attach (repeatable, replaces existing)
  --latitude FLOAT        Updated location latitude (with --longitude)
  --longitude FLOAT       Updated location longitude (with --latitude)
  --currency TEXT          ISO 4217 currency code (defaults to listing's current currency)
  --delivery-type TEXT    Delivery method: public_meetup, door_pickup, door_dropoff (repeatable, replaces existing)
```

## Deleting a Listing / 删除条目

Permanently deletes one of the user's own Marketplace listings. **This cannot be
undone** — the listing is removed from Marketplace and cannot be recovered.

永久删除用户自己的某个 Marketplace 条目。**此操作不可撤销**——条目将从 Marketplace 移除且无法恢复。

### Step 1: Identify the listing / 步骤 1：确定条目

The user must provide the listing ID, or you can list their listings first with
`marketplace my-listings` and use the `listing_id` from the result. Only delete a
listing the user explicitly identified.

必须由用户提供条目 ID，或者先用 `marketplace my-listings` 列出用户的条目并使用结果中的 `listing_id`。只删除用户明确指定的条目。

### Step 2: Confirm before deleting / 步骤 2：删除前确认

Before running the delete command, confirm with the user exactly which listing
will be deleted (show its title and listing link) and that deletion is
permanent. **Never delete a listing without explicit user confirmation.**

在运行 delete 命令之前，与用户确认要删除的具体条目（展示其标题和条目链接），并说明删除是永久性的。**未经用户明确确认，绝不删除条目。**

### Step 3: Delete / 步骤 3：删除

```sh
facebook-cli marketplace listing delete --listing-id <LISTING_ID>
```

### Step 4: Confirm / 步骤 4：确认

On success the response is `{ "listing_id": "<id>", "message": "Listing deleted
successfully" }`. Report the outcome to the user as plain text. If it fails
(e.g. the listing was not found or you do not own it), surface the exact error
and do not retry without the user's approval.

成功时响应为 `{ "listing_id": "<id>", "message": "Listing deleted successfully" }`。以纯文本向用户报告结果。如果失败（如条目不存在或不属于你），如实呈现确切的错误信息，且未经用户同意不要重试。

## CLI Reference — Delete / CLI 参考——删除

```
facebook-cli marketplace listing delete [OPTIONS]

Options:
  --listing-id TEXT        Listing ID to delete (required)
```

## Publishing a Draft Listing / 发布草稿条目

Takes an existing **draft** listing live on Marketplace. This is the companion
to create: `marketplace listing create` saves an incomplete listing as a draft,
and publish takes it live once it has everything required.

将已有的**草稿**条目在 Marketplace 上线。它是 create 的配套操作：`marketplace listing create` 会把不完整的条目保存为草稿，publish 则在条目具备全部所需内容后将其上线。

### Step 1: Identify the draft / 步骤 1：确定草稿

Get the draft's `listing_id` — from a previous `marketplace listing create`
response, or by listing drafts with `marketplace my-listings --status draft`.

获取草稿的 `listing_id`——可来自之前的 `marketplace listing create` 响应，或用 `marketplace my-listings --status draft` 列出草稿。

### Step 2: Check the publish gate before calling / 步骤 2：调用前检查发布闸门

This command publishes against the **same four-field publish gate** as create
(see "The publish gate" above): **photos, condition, category, location**. Before
calling publish, check the draft's fields (from `my-listings` / `listing
details`) and:

此命令的发布遵循与 create **相同的四字段发布闸门**（见上文"发布闸门"）：**照片、成色、类目、位置**。在调用 publish 之前，检查草稿的字段（来自 `my-listings` / `listing details`），并且：

- If a publish-gate field (photos, condition, category, location) is missing,
  **tell the user what's missing first** and add it via `marketplace listing
  edit` (with their confirmation of the values) — don't just fire publish and let
  it bounce.
  如果某个发布闸门字段（照片、成色、类目、位置）缺失，**先把缺失项告诉用户**，并通过 `marketplace listing edit` 补齐（字段值须经用户确认）——不要直接发出 publish 让它报错。
- Don't assume a field is "already set" — if you didn't pass it on create and
  aren't sure it's present, treat it as missing and supply it before publishing.
  不要假设某字段"已经设置"——如果创建时没有传入且不确定它是否存在，就当作缺失处理，并在发布前补上。

### Step 3: Publish / 步骤 3：发布

```sh
facebook-cli marketplace listing publish --listing-id <LISTING_ID>
```

If the draft is still missing a publish-gate field, the command fails with HTTP
400 and a `missing_fields` list naming exactly what's absent. That error is
self-explanatory: add the named fields with `marketplace listing edit` (e.g.
`--photo`, `--condition`, `--category`, `--latitude`/`--longitude`), confirm the
values with the user, then retry publish — surface the missing fields to the user
rather than guessing values.

如果草稿仍缺某个发布闸门字段，命令会以 HTTP 400 失败，并给出 `missing_fields` 列表，明确指出缺失项。该错误本身已说明问题：用 `marketplace listing edit` 补齐所指字段（如 `--photo`、`--condition`、`--category`、`--latitude`/`--longitude`），与用户确认取值后重试 publish——把缺失字段告知用户，而不要猜测取值。

Publishing is **idempotent**: re-publishing an already-live listing returns 200
with `"message": "Listing is already published"` rather than erroring, so retries
are safe.

发布是**幂等的**：对已上线的条目重新发布会返回 200 和 `"message": "Listing is already published"`，而不是报错，因此重试是安全的。

### Step 4: Confirm / 步骤 4：确认

Read the `message`: `"Listing published successfully"` means it is now live;
`"Listing is already published"` means it was live to begin with (safe no-op).
Report the true status and present the `product_url` as plain text (not in a code
block, so it renders as a clickable link). Don't claim it published if the call
errored on missing fields. After either successful live result, follow **Creating
a Listing — Step 5: Check buyer-message readiness after publication**.

读取 `message`：`"Listing published successfully"` 表示现已上线；`"Listing is already published"` 表示它本来就已上线（安全的无操作）。报告真实状态，并以纯文本呈现 `product_url`（不要放进代码块，以便渲染为可点击链接）。如果调用因缺失字段而报错，不要声称已发布。无论哪种成功上线的结果之后，都执行**创建条目——步骤 5：发布后检查买家消息就绪状态**。

## CLI Reference — Publish / CLI 参考——发布

```
facebook-cli marketplace listing publish [OPTIONS]

Options:
  --listing-id TEXT        Draft listing ID to publish (required)
```

The draft must have photos, condition, category, and location. On a missing
field the response includes a `missing_fields` array; fix via `listing edit` and
retry. Re-publishing a live listing is a safe no-op success.

草稿必须具备照片、成色、类目和位置。字段缺失时，响应会包含 `missing_fields` 数组；用 `listing edit` 修复后重试。对已上线条目重新发布是安全的无操作成功。

## Operating Rules / 操作规则

1. **Always quote `--query` values** — e.g., `--query "road bike 51cm"`. Unquoted multi-word queries break argument parsing.
   **始终给 `--query` 的值加引号**——如 `--query "road bike 51cm"`。不带引号的多词查询会破坏参数解析。
2. Omit `--limit` for the default page (20), or set `--limit <n>` (max 20; higher values are capped server-side). For "show me more", paginate with `--after` rather than asking for a bigger limit.
   省略 `--limit` 使用默认页（20 条），或设置 `--limit <n>`（最大 20；更大的值会被服务端截断）。要"看更多"，用 `--after` 分页，而不是要求更大的 limit。
3. If the user mentions a city name, convert it to lat/lng before searching.
   如果用户提到城市名，先转换为经纬度再搜索。
4. If no location is specified, check stored memories for the user's default location and search radius. If found, use those values and tell the user which defaults you applied (e.g., "Using your default location: San Jose, CA (30-mile radius)"). If no memory exists, omit location params.
   如果未指定位置，检查已存储的记忆中是否有用户的默认位置和搜索半径。如有，使用这些值并告知用户应用了哪些默认值（如"使用你的默认位置：San Jose, CA（30 英里半径）"）。如果没有相关记忆，省略位置参数。
5. If the user's request is too vague to form a meaningful search query (e.g., "something nice" without specifying a product type or category), ask a clarifying question before searching. You need at least a specific product type or category to run a useful search.
   如果用户的请求过于模糊、无法构成有意义的搜索查询（如只说"想要个好东西"而未指明商品类型或类目），先提问澄清再搜索。至少需要具体的商品类型或类目才能执行有效搜索。
6. Present prices prominently — buyers care most about price.
   突出呈现价格——买家最关心的就是价格。
7. Present listing images only through the resolved `shopping_results` widget, for search, item details, `saved` and `my-listings`. Never invent image URLs.
   搜索、商品详情、`saved` 和 `my-listings` 的条目图片只能通过解析后的 `shopping_results` 组件呈现。绝不编造图片 URL。
8. Always include a clickable listing link. Use the resolver marker. Without the resolver, use the `product_url` field (fall back to `https://www.facebook.com/marketplace/item/<listing_id>/`), or `url` for item details.
   始终附上可点击的条目链接。优先使用解析器标记；没有解析器时，使用 `product_url` 字段（回退到 `https://www.facebook.com/marketplace/item/<listing_id>/`），商品详情则用 `url`。
9. To get seller ratings, use `marketplace seller-info --listing-id` with the listing ID.
   要获取卖家评分，用条目 ID 调用 `marketplace seller-info --listing-id`。
10. Search and my-listings are cursor-paginated: take the next-page cursor from `paging.cursors.after` and pass it as `--after` to fetch more. Absence of `paging` means there are no further results.
    搜索和 my-listings 均为游标分页：从 `paging.cursors.after` 取下一页游标，作为 `--after` 传入以获取更多。`paging` 不存在表示没有更多结果。
11. **Always present a draft before creating or editing a listing, and always confirm before deleting one.** Show the user a clear summary of what will be posted/changed (for create/edit) or which listing will be removed (for delete) and wait for explicit confirmation before running the command. Deletion is permanent and cannot be undone.  
    **创建或编辑条目前务必先呈现草稿，删除前务必先确认。**向用户清晰展示将要发布/更改的内容（创建/编辑）或将移除的条目（删除），并等待明确确认后再运行命令。删除是永久性的，不可撤销。  
11a. **Publish gate + intent (prevents the #1 confusion).** Before any create or publish, establish draft-vs-live intent, check the publish gate — **all four of photos, condition, category, location must be provided to go live** (location = `--latitude`+`--longitude`, not a `--location` flag; do not omit any expecting the server to default them) — and state the resulting status ("will publish" vs "will be a draft, missing: …") before running the command; afterward, read the response `message` and report the true status. Full details in **"The publish gate (READ FIRST)"** section above.
    11a. **发布闸门 + 意图（防止最常见的困惑）。**在任何 create 或 publish 之前，先确定"草稿还是上线"的意图，核对发布闸门——**照片、成色、类目、位置四项全部齐备才能上线**（位置 = `--latitude`+`--longitude`，而非 `--location` 标志；不要指望服务器为省略项填默认值）——并在运行命令前说明结果状态（"将发布"还是"将存为草稿，缺少：……"）；运行后读取响应的 `message` 并报告真实状态。完整细节见上文**"发布闸门（必读）"**一节。
12. **Never put listing results (URLs, titles, status) in code blocks.** Use plain text so that links render as clickable.
    **绝不要把条目结果（URL、标题、状态）放进代码块。**使用纯文本，链接才能渲染为可点击。
13. **Negative coordinates: use the `--flag=value` form.** Western/southern locations have negative longitude/latitude (e.g. NYC is `-73.97781`). Write `--longitude=-73.97781` (with `=`), not `--longitude -73.97781`. A bare negative value is misread as another flag and the command fails with `error: unexpected argument '-7' found`. The same applies to any numeric flag that can be negative (`--latitude`, `--min-price`, `--max-price`).
    **负坐标：使用 `--flag=value` 形式。**西/南半球的经纬度为负值（如纽约为 `-73.97781`）。应写成 `--longitude=-73.97781`（带 `=`），而不是 `--longitude -73.97781`。裸的负值会被误读为另一个标志，导致命令失败并报 `error: unexpected argument '-7' found`。任何可为负的数值标志都同理（`--latitude`、`--min-price`、`--max-price`）。
14. **Location is only `--latitude` + `--longitude`.** For both search and create/edit, there is no `--location`, `--city`, `--address`, or `--coordinates` flag. Geocode any place name to a decimal lat/lng pair yourself and pass `--latitude` and `--longitude` **together** — never pass a place name to a flag, and never pass just one of the two. For create/edit, the lat/lng pair is what satisfies the "location" requirement for publishing.
    **位置只能是 `--latitude` + `--longitude`。**无论搜索还是创建/编辑，都没有 `--location`、`--city`、`--address` 或 `--coordinates` 标志。自行把地名地理编码为十进制经纬度对，并**同时**传入 `--latitude` 和 `--longitude`——绝不要把地名传给标志，也绝不要只传两者之一。对于创建/编辑，这对经纬度就是满足发布"位置"要求的依据。
