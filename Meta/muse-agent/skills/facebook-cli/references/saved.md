<!-- BILINGUAL-EN-ZH -->

# Saved / 收藏

Read and manage Facebook saved items and collections.

读取和管理 Facebook 收藏内容与收藏集。

## Commands / 命令

### `saved list`

```
facebook-cli saved list [--type <category>] [--collection-id <id>] [--limit N] [--after <opaque>]
```

Lists saved items, most recent first.

列出已收藏的内容，按最近优先排序。

- `--type` (optional): filter by category. One of `post`, `video`, `link`, `product`, `reel`, `event`, `page`.
  `--type`（可选）：按类别过滤。取值为 `post`、`video`、`link`、`product`、`reel`、`event`、`page` 之一。
- `--collection-id` (optional): list items from a specific collection.
  `--collection-id`（可选）：列出特定收藏集中的条目。
- `--limit` (optional): maximum number of items per page (max 20).
  `--limit`（可选）：每页条目数上限（最大 20）。
- `--after` (optional): opaque next-page cursor from a previous response's `paging.cursors.after` (`--cursor` accepted as a back-compat alias).
  `--after`（可选）：来自上一次响应 `paging.cursors.after` 的不透明下一页游标（`--cursor` 作为向后兼容的别名被接受）。

**Response shape** / **响应结构**：

```json
{
  "data": [
    {
      "id": "<save-relationship-id>",
      "savable_id": "<content-id>",
      "type": "post",
      "saved_time": 1719500000,
      "title": "...",
      "description": "...",
      "permalink": "https://www.facebook.com/..."
    }
  ],
  "paging": { "cursors": { "before": "...", "after": "..." } }
}
```

To fetch the next page, pass `paging.cursors.after` as `--after`.

要获取下一页，把 `paging.cursors.after` 作为 `--after` 传入。

### `saved add`

```
facebook-cli saved add --savable-id <FBID> [--type <category>] [--collection-id <id>]
```

Saves an item. A clear user request may proceed without an additional approval.

收藏一个条目。明确的用户请求可以无需额外批准直接执行。

- `--savable-id` (required): Facebook ID of the content (post, page, listing, etc.) — sent as the `id` field.
  `--savable-id`（必需）：内容的 Facebook ID（帖子、主页、商品列表等）——作为 `id` 字段发送。
- `--type` (optional): defaults to `post` server-side. One of `post`, `video`, `link`, `product`, `reel`, `event`, `page`.
  `--type`（可选）：服务端默认为 `post`。取值为 `post`、`video`、`link`、`product`、`reel`、`event`、`page` 之一。
- `--collection-id` (optional): add the item to a specific collection. Omit to save to the default All Saves bucket.
  `--collection-id`（可选）：将条目添加到特定收藏集。省略则保存到默认的"All Saves"收藏夹。

**Response** / **响应**：`{ "id": "<savable-id>", "saved": true }`

### `saved remove`

```
facebook-cli saved remove --savable-id <FBID> [--type <category>] [--collection-id <id>]
```

Removes a previously-saved item. A clear user request may proceed without an additional approval.

移除先前收藏的条目。明确的用户请求可以无需额外批准直接执行。

- `--savable-id` (required): Facebook ID of the saved content (post, page, listing, etc.) — the `savable_id` from `saved list`, sent as the `id` field.
  `--savable-id`（必需）：已收藏内容的 Facebook ID（帖子、主页、商品列表等）——即 `saved list` 返回的 `savable_id`，作为 `id` 字段发送。
- `--type` (optional): defaults to `post` server-side. Pass the item's category to disambiguate.
  `--type`（可选）：服务端默认为 `post`。传入条目的类别以消除歧义。
- `--collection-id` (optional): remove the item only from this collection. Omit to fully unsave.
  `--collection-id`（可选）：仅从该收藏集中移除条目。省略则完全取消收藏。

This command only removes the save relationship — it does not delete the underlying post/video/page.

此命令仅移除收藏关系——不会删除底层的帖子/视频/主页。

【评论】该条款明确区分"取消收藏"与"删除内容"两种操作，是防止代理误删用户数据的保护性设计。

**Response** / **响应**：`{ "id": "<savable-id>", "unsaved": true }`

### `saved collections list`

```
facebook-cli saved collections list
```

Lists collections.

列出收藏集。

**Response shape** / **响应结构**：

```json
{
  "data": [
    { "id": "<collection-id>", "name": "...", "item_count": 5 }
  ]
}
```

### `saved collections create`

```
facebook-cli saved collections create --name "My Collection"
```

Creates an empty collection. A clear user request may proceed without an additional approval.

创建一个空收藏集。明确的用户请求可以无需额外批准直接执行。

**Response** / **响应**：`{ "collection_id": "<id>", "name": "<name>" }`

## Operating notes / 操作说明

- Pagination: `saved list` returns `paging.cursors.after`; pass it as `--after` to fetch the next page. Do not auto-paginate without user direction.
  分页：`saved list` 返回 `paging.cursors.after`；将其作为 `--after` 传入以获取下一页。未经用户指示不要自动翻页。
- Saved item IDs — use `savable_id` for unsave and chaining: every item from `saved list` has two IDs. `id` is the save-relationship entry; `savable_id` is the underlying content. `saved remove` identifies the item by content, so pass the `savable_id` to `saved remove --savable-id <savable_id>` (add `--type <type>` to disambiguate; default is POST). Omitting `--collection-id` fully unsaves the item; passing it removes the item only from that collection. `saved remove` never deletes the post/video/page itself. When chaining to other commands (e.g. `post comments read`, `timeline fetch`), also use `savable_id`.
  收藏条目 ID——取消收藏与后续串联操作使用 `savable_id`：`saved list` 返回的每个条目有两个 ID。`id` 是收藏关系条目；`savable_id` 是底层内容。`saved remove` 按内容标识条目，因此要把 `savable_id` 传给 `saved remove --savable-id <savable_id>`（可加 `--type <type>` 消除歧义；默认为 POST）。省略 `--collection-id` 会完全取消收藏该条目；传入则仅从该收藏集中移除。`saved remove` 绝不会删除帖子/视频/主页本身。在与其他命令串联时（如 `post comments read`、`timeline fetch`），也应使用 `savable_id`。
- Saved-item writes are auto-allowed by default. Resolve the exact item or collection before changing it.
  收藏类写操作默认自动允许。在更改之前先确认确切的条目或收藏集。
