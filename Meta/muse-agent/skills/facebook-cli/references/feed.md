<!-- BILINGUAL-EN-ZH -->
# Facebook Feed / Facebook 动态

Browse your ranked Facebook feeds — the algorithmic newsfeed and the friends-only feed.

浏览经过排序的 Facebook 动态——算法推荐的信息流和仅好友动态流。

## Commands / 命令

### Newsfeed / 信息流

Fetch the ranked algorithmic newsfeed (organic stories only, no ads).

获取经过排序的算法信息流（仅自然内容，无广告）。

```bash
facebook-cli feed newsfeed [--limit <n>] [--after <cursor>]
```

| Flag | Required | Description |
|------|----------|-------------|
| `--limit` | No | Maximum stories to return (default 10, max 10) |
| `--after` | No | Pagination cursor from a previous response's `next_cursor` (`--cursor` accepted as a back-compat alias) |

| 参数 | 是否必填 | 说明 |
|------|----------|------|
| `--limit` | 否 | 返回动态的最大条数（默认 10，上限 10） |
| `--after` | 否 | 来自上一次响应 `next_cursor` 的分页游标（也接受 `--cursor` 作为向后兼容别名） |

**Examples:**  
**示例：**  
```bash
# Fetch your newsfeed
facebook-cli feed newsfeed

# Fetch up to 10 stories (the max)
facebook-cli feed newsfeed --limit 10

# Fetch next page
facebook-cli feed newsfeed --limit 10 --after <after_cursor_from_previous>
```

### Friends Feed / 好友动态

Fetch the ranked friends-only feed (stories from friends only).

获取经过排序的仅好友动态流（仅来自好友的内容）。

```bash
facebook-cli feed friends [--limit <n>] [--after <cursor>]
```

| Flag | Required | Description |
|------|----------|-------------|
| `--limit` | No | Maximum stories to return (default 10, max 10) |
| `--after` | No | Pagination cursor from a previous response's `next_cursor` (`--cursor` accepted as a back-compat alias) |

| 参数 | 是否必填 | 说明 |
|------|----------|------|
| `--limit` | 否 | 返回动态的最大条数（默认 10，上限 10） |
| `--after` | 否 | 来自上一次响应 `next_cursor` 的分页游标（也接受 `--cursor` 作为向后兼容别名） |

**Examples:**  
**示例：**  
```bash
# Fetch friends feed
facebook-cli feed friends

# Fetch 5 stories from friends
facebook-cli feed friends --limit 5
```

## Response Fields / 响应字段

Both commands return the compact `social_posts_v1` collection shared with
social search:

两个命令都返回与社交搜索共享的紧凑集合 `social_posts_v1`：

- `posts`: Ranked feed stories with `post_id`, `url`, `platform`, `created_at`,
  `username`, `post_caption`, `header_text`, and available media/engagement
  fields. `post_caption` prefers authored text, then a media summary, then the
  provider-generated story header also emitted as `header_text`. Treat a media
  summary or story header as a description, never as the author's quoted words.
  `posts`：经过排序的信息流动态，包含 `post_id`、`url`、`platform`、`created_at`、`username`、`post_caption`、`header_text` 以及可用的媒体/互动字段。`post_caption` 优先取作者亲自撰写的文字，其次取媒体摘要，再次取提供方生成的动态标题（该标题同时以 `header_text` 输出）。要把媒体摘要或动态标题当作描述，绝不能当作作者的引语。
- `next_cursor`: Cursor for the next page (pass as `--after`)
  `next_cursor`：下一页的游标（作为 `--after` 传入）
- `has_next_page`: Whether another page is available
  `has_next_page`：是否还有下一页

Read `posts[]` directly. Do not guess a provider response path or pipe the
command through `jq`. A provider schema mismatch fails instead of returning an
empty collection.

直接读取 `posts[]`。不要猜测提供方响应路径，也不要把命令通过 `jq` 管道处理。提供方模式不匹配时会直接失败，而不是返回空集合。

## Operating Rules / 操作规则

1. Use `feed newsfeed` for the standard ranked feed. Use `feed friends` for friends-only content.
   标准排序信息流使用 `feed newsfeed`。仅好友内容使用 `feed friends`。
2. Always check `next_cursor` — if present, more pages are available. Offer to fetch the next page.
   始终检查 `next_cursor` —— 如果存在，说明还有更多页。主动提出可以获取下一页。
3. Feed stories produce `post_id` values that can be used with `post comments read` and `post reactions read`.
   信息流动态产生的 `post_id` 可用于 `post comments read` 和 `post reactions read`。
4. Present stories in the order returned (they are ML-ranked by relevance).
   按返回顺序呈现动态（它们已由机器学习按相关性排序）。
5. Do not editorialize or add subjective commentary on post content.
   不要对帖子内容发表主观评论或添加主观解读。
6. When presenting stories, always include `url` so the user can view them on Facebook.
   呈现动态时始终附上 `url`，方便用户在 Facebook 上查看。
