---
name: "shopify"
description: >
  Set up and operate a Shopify store with Shopify's official MCP server (the
  installed `shopify` command, NOT the npm Shopify CLI). Use when someone wants
  to sell online, start a store or business, or manage a Shopify store — even
  if they do not say "Shopify": "sell candles online", "open my first store",
  "add products", "check my orders", "set inventory", "make a discount",
  "show my sales". Before connection it can show relevant mock.shop sample
  catalogs, suggest business names, check domains, search shopify.dev, and
  validate GraphQL. The connect flow can create an account and store for a new
  user. Keywords: Shopify MCP, ecommerce, sample products, starter catalog,
  ShopifyQL, first store, create product.
icon: "shopify"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->
# Shopify / Shopify

The installed `shopify` command wraps Shopify's official MCP server. It is not
the Shopify CLI from npm; the only verbs are the four below. Tool names and
schemas come from `shopify list-tools`, never from memory — other Shopify
connectors expose tools this one does not.

已安装的 `shopify` 命令封装了 Shopify 官方 MCP 服务器。它不是 npm 的 Shopify CLI；仅有的动词就是下面四个。工具名与 schema 来自 `shopify list-tools`，绝不凭记忆——其他 Shopify 连接器会暴露本连接器没有的工具。

```text
shopify status
shopify authorize-url
shopify list-tools
shopify call-tool --name <tool-name> --arguments-json '<JSON object>'
```

## Connecting / 连接

1. `shopify status`. If a store is connected, go straight to work.
   `shopify status`。若已有店铺连接，直接开始工作。
2. Otherwise `shopify authorize-url` and share **only** the returned
   `connect_url` with the user. Nothing else in that response is for them.
   否则运行 `shopify authorize-url`，并且只把返回的 `connect_url` 分享给用户。该响应中的其他内容都不是给他们的。
3. When they say they're done, `shopify status`, then
   `call-tool --name get-shop-info` to confirm which store is connected.
   当用户说已完成时，运行 `shopify status`，再运行 `call-tool --name get-shop-info` 确认连接的是哪个店铺。

Before a store is connected, `search_docs_chunks` (searches https://shopify.dev),
`validate_graphql_codeblocks`, `find-mock-shop-catalogs`, `generate-domain-names`,
and `generate-business-names` work. `get-storefront-generation` is usable only
with a `generationUUID` created by Shopify's storefront-generation widget;
`claim-storefront-preview` is a widget callback and must not be called directly.
Every other tool, including catalog import, needs a store. Do not ask someone
to connect merely to explore sample catalogs, names, domains, or documentation.

店铺连接之前，`search_docs_chunks`（搜索 https://shopify.dev）、`validate_graphql_codeblocks`、`find-mock-shop-catalogs`、`generate-domain-names` 与 `generate-business-names` 可用。`get-storefront-generation` 只能配合 Shopify storefront-generation widget 创建的 `generationUUID` 使用；`claim-storefront-preview` 是 widget 回调，绝不能直接调用。其他所有工具（包括目录导入）都需要店铺。不要为了浏览示例目录、名称、域名或文档就要求用户连接。

【评论】这里把工具划分为"连接前可用 / 仅限 widget / 需要店铺"三类，并禁止模型直接调用 widget 专属回调——是对工具调用面的人为收窄，防止绕过前端流程。

## Sample catalogs / 示例目录

When someone is starting a store, offer relevant mock.shop sample catalogs
before asking them to connect:

当有人要开店铺时，在要求连接之前先提供相关的 mock.shop 示例目录：

1. Call `find-mock-shop-catalogs` with what they sell, in their own words. If
   they have not said, ask that one question first.
   用他们自己的话描述的商品调用 `find-mock-shop-catalogs`。如果他们还没说，先问这一个问题。
2. Present the best matches with their descriptions, product and collection
   counts, currency, and `storefrontUrl`. Clearly label each link as the source
   sample storefront, then ask which catalog they want.
   呈现最匹配的结果，附描述、商品与集合数量、货币及 `storefrontUrl`。清楚标注每个链接是来源示例店面，然后询问他们想要哪个目录。
3. Once connected, confirm before calling `import-mock-shop-catalog` with the
   selected `subdomain`.
   连接之后，用所选 `subdomain` 调用 `import-mock-shop-catalog` 前先确认。
4. Verify the destination records with `search_products` and
   `search_collections`, then report created, reused, skipped, unpublished, or
   partial results and any currency mismatch.
   用 `search_products` 与 `search_collections` 验证目标记录，然后报告新建、复用、跳过、未发布或部分完成的结果以及任何货币不匹配。

An import copies the first two collections and up to eight products from each,
at most sixteen products, and publishes newly imported records to the Online
Store. Repeating the same import is safe: it fills missing records and
memberships without overwriting previously imported products or merchant edits.
If Shopify reports `partial: true`, wait briefly and repeat the same import.
These are sample records to edit or replace before launch, not supplier stock.

一次导入会复制前两个集合及各自最多八个商品，合计最多十六个商品，并把新导入的记录发布到 Online Store。重复同一导入是安全的：它补齐缺失的记录与成员关系，不会覆盖此前导入的商品或商家的修改。若 Shopify 报告 `partial: true`，稍等片刻并重复同一导入。这些是要在上线前编辑或替换的示例记录，不是供应商库存。

## Users without a store / 没有店铺的用户

The same `connect_url` works for someone who has never used Shopify. Shopify's
sign-in page lets them create an account and a new store, then returns them to
the connection. Tell them this up front so they don't go create a store
separately first. Once connected, offer a first-store sequence and take it one
step at a time:

同一个 `connect_url` 也适用于从未用过 Shopify 的人。Shopify 的登录页让他们创建账户与新店铺，然后返回连接流程。提前告知这一点，免得他们先去单独创建店铺。连接之后，提供首店流程，一步一步来：

1. `get-shop-info` to learn the store's name, currency, and plan.
   `get-shop-info` 了解店铺名称、货币与套餐。
2. Import the sample catalog they selected, or use `create-product` for their
   own products (title, description, price, images by URL). Ask what they sell;
   don't invent a catalogue.
   导入他们选定的示例目录，或用 `create-product` 创建他们自己的商品（标题、描述、价格、按 URL 提供的图片）。问他们卖什么；不要凭空编造商品目录。
3. `create-collection`, then `add-to-collection` to group them.
   `create-collection`，然后用 `add-to-collection` 归组。
4. If they track stock, call `get-inventory-levels` before `set-inventory` to
   resolve the inventory item, location, and current quantity.
   如果他们跟踪库存，先调用 `get-inventory-levels` 再 `set-inventory`，以解析库存条目、位置与当前数量。
5. `create-discount` for a launch promotion, only if they want one.
   仅当他们想要时，才用 `create-discount` 做上线促销。

Business-name and domain results may include `signupUrl` links for starting a
store. Use those for exploration; when the user is ready to connect Muse to
their store, share the connector's `connect_url` instead.

商家名称与域名结果可能包含用于开店的 `signupUrl` 链接。探索阶段用它们；当用户准备好把 Muse 连接到店铺时，改为分享连接器的 `connect_url`。

Point them at the Shopify admin (the domain from `get-shop-info`) for anything
this server doesn't cover: themes, checkout, payments, shipping, and connecting
or buying domains.

本服务器未覆盖的事项（主题、结账、支付、物流以及连接或购买域名），指引他们去 Shopify admin（`get-shop-info` 中的域名）。

## Common tasks / 常见任务

| Task | Tools |
| --- | --- |
| Store details | `get-shop-info` |
| Business names and available domains | `generate-business-names`, `generate-domain-names` |
| Sample catalogs | `find-mock-shop-catalogs`, then `import-mock-shop-catalog` with the selected `subdomain`; this imports catalog data, not storefront design |
| Storefront previews | `get-storefront-generation` only for a generation already created by Shopify's widget |
| Products | `search_products`, `get-product`, `create-product`, `update-product`, `bulk-update-product-status` |
| Collections | `search_collections`, `get-collection`, `create-collection`, `update-collection`, `add-to-collection` |
| Inventory | `get-inventory-levels`, `set-inventory` |
| Orders and customers | `list-orders`, `get-order`, `list-customers` |
| Discounts | `create-discount` |
| Reports and trends | `run-analytics-query` (ShopifyQL; the description has examples) |
| Anything else in Admin | `graphql_schema` → `validate_graphql_codeblocks` → `graphql_query` or `graphql_mutation`, in that order, every time |
| How Shopify works | `search_docs_chunks` |
| Another store | `switch-shop`, then `get-shop-info` |

| 任务 | 工具 |
| --- | --- |
| 店铺详情 | `get-shop-info` |
| 商家名称与可用域名 | `generate-business-names`、`generate-domain-names` |
| 示例目录 | `find-mock-shop-catalogs`，然后用所选 `subdomain` 调用 `import-mock-shop-catalog`；这导入的是目录数据，不是店面设计 |
| 店面预览 | `get-storefront-generation` 仅用于已由 Shopify widget 创建的 generation |
| 商品 | `search_products`、`get-product`、`create-product`、`update-product`、`bulk-update-product-status` |
| 集合 | `search_collections`、`get-collection`、`create-collection`、`update-collection`、`add-to-collection` |
| 库存 | `get-inventory-levels`、`set-inventory` |
| 订单与客户 | `list-orders`、`get-order`、`list-customers` |
| 折扣 | `create-discount` |
| 报告与趋势 | `run-analytics-query`（ShopifyQL；描述中有示例） |
| Admin 中的其他一切 | `graphql_schema` → `validate_graphql_codeblocks` → `graphql_query` 或 `graphql_mutation`，每次都按此顺序 |
| Shopify 如何运作 | `search_docs_chunks` |
| 另一个店铺 | `switch-shop`，然后 `get-shop-info` |

## Rules / 规则

- Authenticated commands refresh an expired access token automatically. Do not
  tell the user to disconnect and reconnect for ordinary expiry. If Shopify
  reports that the refresh grant was revoked or reauthorization is required,
  run `shopify authorize-url`; do not require a disconnect first unless the
  returned result explicitly says it is necessary.
  已认证的命令会自动刷新过期的访问令牌。普通过期不要让用户断开重连。若 Shopify 报告刷新授权已被撤销或需要重新授权，运行 `shopify authorize-url`；除非返回结果明确说必须先断开，否则不要求先断开。
- Inspect the returned result as well as the command status: Shopify can report
  a tool-level failure inside a successful MCP response.
  除命令状态外还要检查返回结果：Shopify 可能在成功的 MCP 响应内部报告工具级失败。
- Confirm with the user before any tool that writes: create, update, set,
  bulk, discounts, catalog import, preview claims, `graphql_mutation`. Catalog
  import publishes sample products and collections to the Online Store; report
  partial imports and currency mismatches (prices are copied without conversion).
  任何写入类工具之前先与用户确认：create、update、set、bulk、折扣、目录导入、预览认领、`graphql_mutation`。目录导入会把示例商品与集合发布到 Online Store；要报告部分导入与货币不匹配（价格按原样复制，不做换算）。
- A catalog import does not change the store name, theme, navigation, homepage
  sections, or pre-existing products. The `storefrontUrl` returned by
  `find-mock-shop-catalogs` previews the source mock catalog, not the destination
  store after import. Never say the destination storefront will look like that
  preview or offer its homepage as proof. After importing, verify the new records
  with `search_products` and `search_collections`, then report that the merchant
  must configure their theme in Shopify Admin if they want the homepage to feature
  the imported catalog.
  目录导入不会更改店铺名称、主题、导航、首页版块或既有商品。`find-mock-shop-catalogs` 返回的 `storefrontUrl` 预览的是来源示例目录，不是导入后的目标店铺。绝不说目标店面会像那个预览一样，也不要拿它的首页当证据。导入之后，用 `search_products` 与 `search_collections` 验证新记录，然后告知商家：若想让首页展示导入的目录，需要在 Shopify Admin 中自行配置主题。
- `switch-shop` must be followed by another tool call (the requested action,
  or `get-shop-info`) to finish the switch.
  `switch-shop` 之后必须再跟一次工具调用（所请求的操作，或 `get-shop-info`）才算完成切换。
- `get-storefront-generation` only polls a `generationUUID` already created by
  Shopify's storefront-generation widget; it cannot start a generation or alter
  an existing store. `claim-storefront-preview` is called only by that widget,
  never directly by the model. Neither tool applies a mock catalog's theme to a
  connected store.
  `get-storefront-generation` 只轮询 Shopify storefront-generation widget 已创建的 `generationUUID`；它不能发起 generation，也不能改动既有店铺。`claim-storefront-preview` 只由该 widget 调用，绝不由模型直接调用。两个工具都不会把示例目录的主题应用到已连接的店铺。
- Muse shows structured data, not Shopify's widgets. Summarize results in
  the reply; don't refer to a card, chart, or preview the user can't see.
  Muse 展示结构化数据，不是 Shopify 的 widget。在回复中总结结果；不要提及用户看不到的卡片、图表或预览。
