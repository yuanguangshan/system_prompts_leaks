---
name: "ticketmaster"
description: "Search Ticketmaster events and seats with pricing. Returns Buy-now links to Ticketmaster checkout; it cannot complete a purchase itself."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Ticketmaster / Ticketmaster 票务

## Purpose / 目的
Search events and get smart Ticketmaster seat recommendations.

搜索活动并获取智能的 Ticketmaster 座位推荐。

For a user-facing event-ticket search or purchase, first read  
`/opt/hatch/skills/booking/SKILL.md`,  
`/opt/hatch/skills/booking/references/tickets.md`, and
`/opt/hatch/skills/booking/references/presentation.md`. Those files define the
routing, presentation, browser checkout, and commitment rules. Use this file
for the Ticketmaster CLI contract. Do not call `seat-view-carousel`, create
HTML, or use `widget.create` for a booking flow. Use the compact Markdown ticket
table and optional verified Markdown images.
Do not call `create_options` for ticket choices or checkout decisions.

面向用户的活动门票搜索或购买，请先阅读  
`/opt/hatch/skills/booking/SKILL.md`、  
`/opt/hatch/skills/booking/references/tickets.md` 与
`/opt/hatch/skills/booking/references/presentation.md`。这些文件定义了路由、呈现、浏览器结账与承诺规则。本文件只用于 Ticketmaster CLI 的契约。在预订流程中不要调用 `seat-view-carousel`、不要创建 HTML、也不要使用 `widget.create`。使用紧凑的 Markdown 门票表格和可选的已验证 Markdown 图片。
不要为票档选择或结账决策调用 `create_options`。

## Tooling / 工具

```sh
ticketmaster <subcommand> [options]
```

#### search-events / 搜索活动
Search for events by keyword (required), location, and date range. Supports `--country-code` (default: US), `--page` (0-indexed), and `--sort` (e.g. `date,asc`, `relevance,desc`, `name,asc`). `--start-date` and `--end-date` accept ISO 8601 values; bare `YYYY-MM-DD` dates are normalized by the CLI.

按关键词（必填）、位置和日期范围搜索活动。支持 `--country-code`（默认：US）、`--page`（从 0 开始）与 `--sort`（例如 `date,asc`、`relevance,desc`、`name,asc`）。`--start-date` 与 `--end-date` 接受 ISO 8601 值；裸 `YYYY-MM-DD` 日期由 CLI 归一化。

```sh
ticketmaster search-events --keyword "Taylor Swift" --city "Los Angeles" --size 10
ticketmaster search-events --keyword "Lakers" --start-date 2026-05-01 --end-date 2026-06-01 --state-code CA --sort date,asc
```

#### event-details / 活动详情
Get full details for a specific event ID returned by `search-events`.

获取 `search-events` 返回的某个活动 ID 的完整详情。

```sh
ticketmaster event-details --event-id vvG1IZ_AKnSEae
```

#### top-picks / 优选座位
Get seat recommendations sorted by price or quality. Default `--selection Any` returns Standard + Resale + Platinum tickets. Use `--selection Standard` to exclude resale. Returns `venue_map_url` and per-pick `snapshot_image_url` when available. Preserve those URLs for verified Markdown links or images.

获取按价格或质量排序的座位推荐。默认 `--selection Any` 返回 Standard + Resale + Platinum 票。使用 `--selection Standard` 排除转售票。在可用时返回 `venue_map_url` 和每个推荐档位的 `snapshot_image_url`。保留这些 URL 以用于已验证的 Markdown 链接或图片。

```sh
ticketmaster top-picks --event-id vvG1IZ_AKnSEae --quantity 2 --sort listprice
ticketmaster top-picks --event-id vvG1IZ_AKnSEae --quantity 4 --sections "MEZZ,ORCH" --sort quality
ticketmaster top-picks --event-id vvG1IZ_AKnSEae --quantity 2 --price-min 50 --price-max 150 --limit 10
ticketmaster top-picks --event-id vvG1IZ_AKnSEae --quantity 2 --selection Standard --sort listprice
ticketmaster top-picks --event-id vvG1IZ_AKnSEae --quantity 2 --areas "10,11" --ticket-type-id 000000000001
```

#### seat-view-carousel / 座位视图轮播
Generate a carousel HTML widget from top-picks JSON. Takes the full top-picks JSON output as `--input`. Outputs raw HTML to stdout.

从 top-picks JSON 生成轮播 HTML 部件。以完整的 top-picks JSON 输出作为 `--input`。向 stdout 输出原始 HTML。

```sh
ticketmaster seat-view-carousel --input '<top-picks JSON>' --title "Lakers vs Thunder" --subtitle "Paycom Center · May 13 · 8:30 PM"
```

## Output / 输出

Commands return JSON from Ticketmaster. Do not expect an
`ok` wrapper around endpoint output.

命令返回来自 Ticketmaster 的 JSON。不要期望端点输出外面还包着 `ok` 包装器。

The CLI maps snake_case query params to Ticketmaster camelCase API params. The
response body is the raw Ticketmaster payload unless this CLI adds explicit
convenience fields for local rendering.

CLI 将 snake_case 查询参数映射为 Ticketmaster 的 camelCase API 参数。响应体是原始的 Ticketmaster 载荷，除非本 CLI 为本地渲染添加了明确的便捷字段。

**search-events**: raw Ticketmaster Discovery search response. Events are in
`_embedded.events[]`; pagination is in `page`. For each event, use `id` as the
event ID for follow-up calls, `name` for the title, `url` for the public
Ticketmaster page, `dates.start.localDate`, `dates.start.localTime`, and
`dates.start.dateTime` for timing, `_embedded.venues[0]` for venue/city/state,
and `priceRanges[]` when Ticketmaster includes search-level pricing.
Each event may add `tmol_available`. When it is `false`, `url` and the
`tmMarketPlace` outlet URL are omitted and `box_office_url` contains a sanitized
venue link or `null`. An absent `tmol_available` means availability is unknown.

**search-events**：原始的 Ticketmaster Discovery 搜索响应。活动位于 `_embedded.events[]`；分页位于 `page`。对每个活动，用 `id` 作为后续调用的事件 ID，`name` 作为标题，`url` 作为公开的 Ticketmaster 页面，`dates.start.localDate`、`dates.start.localTime` 与 `dates.start.dateTime` 表示时间，`_embedded.venues[0]` 表示场馆/城市/州，当 Ticketmaster 附带搜索级价格时用 `priceRanges[]`。
每个活动可能附加 `tmol_available`。当其为 `false` 时，`url` 与 `tmMarketPlace` 外部销售 URL 被省略，`box_office_url` 包含一个经清理的场馆链接或 `null`。`tmol_available` 缺失意味着可用性未知。

**event-details**: raw Ticketmaster Discovery event object. Read fields
directly from the event: `id`, `name`, `url`, `dates.start.*`,
`dates.timezone`, `dates.status.code`, `priceRanges[]`, `classifications[]`,
`images[]`, `_embedded.venues[]`, `_embedded.attractions[]`, and `_links`.

**event-details**：原始的 Ticketmaster Discovery 活动对象。直接从活动对象读取字段：`id`、`name`、`url`、`dates.start.*`、`dates.timezone`、`dates.status.code`、`priceRanges[]`、`classifications[]`、`images[]`、`_embedded.venues[]`、`_embedded.attractions[]` 与 `_links`。

Timed event reads add `event_starts_at` / `event_ends_at` with canonical UTC
and user-local forms, plus runtime-generated `retrieved_at`. Prefer the
user-local form when answering; date-only event fields remain dates.

定时活动读取会附加 `event_starts_at` / `event_ends_at`（含标准 UTC 与用户本地时间两种形式），以及运行时生成的 `retrieved_at`。回答时优先使用用户本地时间形式；仅日期的活动字段仍保持为日期。

**top-picks**: raw Ticketmaster Top Picks response with CLI convenience fields
added for carousel rendering when available.

**top-picks**：原始的 Ticketmaster Top Picks 响应，在可用时附加了用于轮播渲染的 CLI 便捷字段。

Top-picks contains `picks[]`, `_embedded.offer[]`, `eventDetails`, and `page`.
Seat-level pricing lives in `_embedded.offer[]`, keyed by `offerId`; each pick
references offers through `picks[].offers[]`. Use the first referenced matching
offer as the primary displayed price. `_embedded.offer[].totalPrice` is the
price **per ticket**, not the total for the requested quantity. Multiply it by
the requested quantity and show that computed full-party amount as the primary
price; label the per-ticket amount separately only when useful. Do not label one
ticket's `totalPrice` as the pair or party total. If
`eventDetails.allInclusivePricing` is true, say the displayed per-ticket price
and computed party total include fees and show
`eventDetails.listingsDisclaimer` when present. You may show `faceValue` only
as additional context, not as the main price.

Top-picks 包含 `picks[]`、`_embedded.offer[]`、`eventDetails` 与 `page`。座位级定价位于 `_embedded.offer[]`，以 `offerId` 为键；每个推荐档位通过 `picks[].offers[]` 引用报价。使用第一个被引用的匹配报价作为主要显示价格。`_embedded.offer[].totalPrice` 是**每张票**的价格，不是所请求数量的总价。将其乘以所请求的数量，并把计算出的全组总额作为主要价格显示；仅在有用时才单独标注单张票价。不要把一张票的 `totalPrice` 标成两张或全组总价。若 `eventDetails.allInclusivePricing` 为 true，需说明所显示的单张票价与计算出的全组总额均已含费用，并在存在时展示 `eventDetails.listingsDisclaimer`。`faceValue` 只能作为补充信息展示，不能作为主要价格。

【评论】原文特意强调 `totalPrice` 是单张票价而非总价，并要求乘以数量后展示全组总额——这是为防止代理把单价误报为总价而设的防错性规定。

Ticketmaster may return both camelCase and snake_case duplicates. Use this
precedence when reading fields:
- Checkout URL: `pick.redirect_url || pick.redirectUrl`
  结账 URL：`pick.redirect_url || pick.redirectUrl`
- Seat image: `pick.snapshot_image_url || pick.snapshotImageUrl || pick.snapshotURL`
  座位图片：`pick.snapshot_image_url || pick.snapshotImageUrl || pick.snapshotURL`
- Price: `pick.total_price || _embedded.offer[offerId].totalPrice`
  价格：`pick.total_price || _embedded.offer[offerId].totalPrice`

The CLI may add these convenience fields for carousel rendering:
`venue_map_url`, `snapshotURL`, `snapshot_image_url`, `redirect_url`,
`total_price`, `face_value`, and `currency`.

CLI 可能为轮播渲染添加以下便捷字段：`venue_map_url`、`snapshotURL`、`snapshot_image_url`、`redirect_url`、`total_price`、`face_value` 与 `currency`。

Each pick may include: `{ type, selection, section, row, seats, area, quality, descriptions, listingDetails, offers, snapshotImageUrl, redirectUrl }`.

每个推荐档位可能包含：`{ type, selection, section, row, seats, area, quality, descriptions, listingDetails, offers, snapshotImageUrl, redirectUrl }`。

**seat-view-carousel**: Legacy HTML output. Do not use it in a booking flow.
Images are Ticketmaster-hosted, for example
`https://app.ticketmaster.com/maps/geometry/...`. Checkout URLs must be valid
HTTPS URLs; `https://ticketmaster.evyy.net/...` affiliate redirect URLs with
preselected seats are valid when returned.

**seat-view-carousel**：遗留的 HTML 输出。不要在预订流程中使用它。图片由 Ticketmaster 托管，例如 `https://app.ticketmaster.com/maps/geometry/...`。结账 URL 必须是有效的 HTTPS URL；带有预选座位的 `https://ticketmaster.evyy.net/...` 联盟跳转 URL 在返回时是有效的。

Do not generate or render the carousel or a buy-now list widget in a booking
flow. Put three to five exact ticket groups in the booking skill's Markdown
table. A verified `snapshot_image_url` may appear below the table with ordinary
Markdown image syntax and an exact section label. Keep each valid checkout URL
as a compact Markdown link in the relevant option or next action.

在预订流程中不要生成或渲染轮播或立即购买列表部件。把三到五个精确票档放入预订技能的 Markdown 表格。已验证的 `snapshot_image_url` 可以用普通 Markdown 图片语法加精确的看台标签显示在表格下方。将每个有效的结账 URL 以紧凑 Markdown 链接的形式保留在相应选项或下一步操作中。

## Auth / 认证
No user setup is required for Ticketmaster service access.

访问 Ticketmaster 服务无需用户做任何设置。

## Operating Rules / 操作规则
1. Use `search-events` first to find the event ID before calling `top-picks`.
   先使用 `search-events` 找到活动 ID，再调用 `top-picks`。
2. When the user asks for "cheap" tickets, use `--sort listprice`. When they want "best" seats, use `--sort quality`.
   当用户要"便宜"的票时，使用 `--sort listprice`。当他们想要"最好"的座位时，使用 `--sort quality`。
3. When searching for a seating level, do not guess the section name. Venues use inconsistent names for floor, mezzanine, orchestra, pit, and upper sections. Run a broad `--sort quality --limit 20` query with no `--sections` filter. Use the returned section names and area labels in later `--sections` queries.
   搜索某个座位层级时，不要猜测看台名称。各场馆对内场（floor）、楼座（mezzanine）、池座（orchestra）、乐池（pit）和上层看台的命名并不一致。先运行不带 `--sections` 过滤的宽泛查询 `--sort quality --limit 20`。在后续 `--sections` 查询中使用返回的看台名称和区域标签。
4. When a valid checkout URL (`redirect_url || redirectUrl`) and a displayed price are present, include a compact `[Review on Ticketmaster](<redirect_url>)` Markdown link for that option. Ticketmaster affiliate links such as `https://ticketmaster.evyy.net/...` are valid checkout URLs. If the checkout URL is absent/invalid or pricing is unavailable, do not fabricate a checkout link or present the seat as directly purchasable.
   Bind each link to the same pick used for that row and verify its offer/seat
   parameters before sending. Do not reuse one option's URL for another option;
   omit an unverified link instead.
   当存在有效的结账 URL（`redirect_url || redirectUrl`）和所显示的价格时，为该选项附上一个紧凑的 `[Review on Ticketmaster](<redirect_url>)` Markdown 链接。诸如 `https://ticketmaster.evyy.net/...` 的 Ticketmaster 联盟链接是有效的结账 URL。若结账 URL 缺失/无效或价格不可用，不要伪造结账链接，也不要把该座位描述为可直接购买。
   每个链接都要绑定到该行所用的同一个推荐档位，并在发送前核验其报价/座位参数。不要把一个选项的 URL 复用给另一个选项；未经验证的链接宁可不加。
5. For resale tickets (`selection: "resale"`), show the `listingDetails` description when present and note these are verified resale tickets.
   对于转售票（`selection: "resale"`），在存在时展示 `listingDetails` 描述，并说明这些是经核验的转售票。
6. If the selected offer has a `limit` object, respect its `min` and `max` quantity bounds.
   若所选报价带有 `limit` 对象，遵守其 `min` 与 `max` 数量边界。
7. If `eventDetails.allInclusivePricing` is true, note that the displayed price includes fees. Show `eventDetails.listingsDisclaimer` and `eventDetails.importantInformation` when present.
   若 `eventDetails.allInclusivePricing` 为 true，注明所显示价格已含费用。在存在时展示 `eventDetails.listingsDisclaimer` 与 `eventDetails.importantInformation`。
8. After `top-picks` returns, follow the `Event tickets` section in  
   `/opt/hatch/skills/booking/references/presentation.md`.
   `top-picks` 返回后，遵循  
   `/opt/hatch/skills/booking/references/presentation.md` 中的 `Event tickets` 一节。
9. `priceRanges[]` from search-events is often absent or incomplete. Do not tell the user "no pricing available" based on search-events alone. Use `top-picks` to get current pricing.
   search-events 的 `priceRanges[]` 常常缺失或不完整。不要仅凭 search-events 就告诉用户"无可用价格"。使用 `top-picks` 获取当前价格。
10. Use the raw Discovery `id` from `_embedded.events[]` for `event-details` and prefer it for `top-picks` calls. `top-picks` accepts either the Discovery ID (for example `vvG...`) or the numeric geometry/eventDetails ID (for example `0900...`); the response may echo the numeric ID as `eventDetails.id`.
    在 `event-details` 中使用 `_embedded.events[]` 的原始 Discovery `id`，`top-picks` 调用也优先使用它。`top-picks` 既接受 Discovery ID（例如 `vvG...`），也接受数字型的 geometry/eventDetails ID（例如 `0900...`）；响应可能以 `eventDetails.id` 回显数字 ID。
11. If a search event has `tmol_available: false`, it is not sold on Ticketmaster. Say so, offer its non-null `box_office_url`, and never run `top-picks` on it. If `box_office_url` is null, say Ticketmaster Discovery supplied no official purchase link. If `tmol_available` is absent, do not claim the event is offsite.
    若某个搜索到的活动为 `tmol_available: false`，则它不在 Ticketmaster 上销售。如实说明，提供其非空的 `box_office_url`，且绝不对它运行 `top-picks`。若 `box_office_url` 为 null，说明 Ticketmaster Discovery 未提供官方购买链接。若 `tmol_available` 缺失，不要断言该活动在站外销售。
12. When the user selects tickets, use the browser to keep the exact checkout alive and prepare it to final review. Ask whether the user has a Ticketmaster account and wants to sign in before checkout. Continue as a guest when permitted. Hand off the exact link only when authentication, blocked automation, or another real limitation prevents the main agent from continuing.
    当用户选定票档时，使用浏览器保持该精确结账会话活跃并将其推进到最终确认。询问用户是否拥有 Ticketmaster 账号以及是否想在结账前登录。在允许时以访客身份继续。仅当身份验证、被阻止的自动化或其他真实限制使主代理无法继续时，才交出精确链接。
13. If a response carries an `error`, drop it silently: skip that event or pick, and do not display it or surface the error text.
    若响应带有 `error`，静默丢弃：跳过该活动或推荐档位，不显示它，也不把错误文本抛给用户。
    【评论】与常见"向用户报告错误"的做法不同，这里要求对单条错误静默降级，以保证结果列表的整洁与转化路径的连贯。
14. Do not quote a price from memory or an earlier turn. Re-run `top-picks` and quote only its current result. A price-monitoring cron must fetch current inventory on every run.
    不要凭记忆或更早的回合引用价格。重新运行 `top-picks` 并只引用其当前结果。价格监控 cron 每次运行都必须抓取当前库存。
