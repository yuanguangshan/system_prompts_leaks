<!-- BILINGUAL-EN-ZH -->

# Shopping Backends / 购物后端

Load this file only when you need backend-specific filters.

仅当你需要特定后端的筛选项时才加载本文件。

## Facebook Marketplace / Facebook Marketplace

Use for local, secondhand, pickup, strict-budget, and deal-hunting requests.

适用于本地、二手、自提、严格预算和寻找优惠类的请求。

Base search flow:

基本搜索流程：

```sh
MARKETPLACE_RESULTS_JSON=$(mktemp "${TMPDIR:-/tmp}/facebook-marketplace-search.XXXXXX")

facebook-cli marketplace search --query "<item>" --limit <N> --out "$MARKETPLACE_RESULTS_JSON"
```

Useful flags:

实用参数：

- `--max-price`, `--min-price` in dollars
  `--max-price`、`--min-price`：价格上下限，单位为美元
- `--latitude` and `--longitude="<lng>"` together for location search; quote negative coordinates with `=`
  `--latitude` 和 `--longitude="<lng>"` 成对使用进行位置搜索；负坐标要用 `=` 形式书写
- `--radius-in-miles` for local distance
  `--radius-in-miles`：本地搜索的距离半径
- `--sort-by best_match|price_ascend|price_descend|creation_time_descend|distance_ascend`
  排序方式：最佳匹配、价格升序、价格降序、发布时间倒序、距离升序
- `--allowed-item-conditions new,refurbished,used` for condition filtering
  `--allowed-item-conditions new,refurbished,used`：按成色筛选（全新、翻新、二手）
- `--delivery-method local_pickup_only|shipping_only|pickup_and_shipping`
  配送方式：仅本地自提、仅快递、自提加快递
- `--max-listing-age-in-days` for recency
  `--max-listing-age-in-days`：筛选近期发布的商品
- `--limit <N>` for page size (default and max 20; higher values are capped)
  `--limit <N>`：每页数量（默认和上限均为 20；更大的值会被截断）
- `--after <cursor>` to continue a search — pass the `paging.cursors.after` value from the previous response (absent `paging` means no more results)
  `--after <cursor>`：继续上一次搜索——传入上一次响应中的 `paging.cursors.after` 值（缺少 `paging` 字段表示没有更多结果）

Call `shopping.resolve_results`:

调用 `shopping.resolve_results`：

```json
{
  "result_paths": ["<MARKETPLACE_RESULTS_JSON>"],
  "selected_ids": ["listing-id-1", "listing-id-2"]
}
```

The resolver maps Marketplace `listing_id` values into shopping result cards.  
Present the returned `path` with `widget.create` using
`kind: "shopping_results"` and `data.path`.

解析器会把 Marketplace 的 `listing_id` 值映射为购物结果卡片。使用 `widget.create` 呈现返回的 `path`，参数为 `kind: "shopping_results"` 和 `data.path`。

To present catalog and Marketplace picks together:

若要将目录商品与 Marketplace 的选择一起呈现：

```json
{
  "result_paths": ["<CATALOG_RESULTS_JSON>", "<MARKETPLACE_RESULTS_JSON>"],
  "selected_ids": ["catalog-id-1", "listing-id-1"]
}
```

Present the returned `path` with `widget.create` using
`kind: "shopping_results"` and `data.path`.

使用 `widget.create` 呈现返回的 `path`，参数为 `kind: "shopping_results"` 和 `data.path`。

【评论】该流程规定 Marketplace 结果必须先经 `shopping.resolve_results` 归一化为统一的结果卡片，再由 `widget.create` 渲染，命令行输出不直接展示给用户。
