<!-- BILINGUAL-EN-ZH -->

# Facebook Comments / Facebook 评论

Read comments on Facebook posts.

读取 Facebook 帖子上的评论。

## Command / 命令

```bash
facebook-cli post comments read --post-id <post-id> [--limit <n>] [--after <cursor>]
```

**Options:**
**选项：**

- `--post-id` (required): Post ID to read comments from
  `--post-id`（必填）：要读取评论的帖子 ID
- `--limit` (optional): Maximum number of comments per page (max 20; higher values are capped server-side)
  `--limit`（可选）：每页评论数上限（最大 20；更高的值会被服务端截断）
- `--after` (optional): Pagination cursor — pass the `paging.cursors.after` value from the previous response to fetch the next page
  `--after`（可选）：分页游标 —— 传入上一次响应中的 `paging.cursors.after` 值以获取下一页

## Response Fields / 响应字段

The response is cursor-paginated:
响应采用游标分页：

- `data`: Array of comment objects, each with:
  `data`：评论对象数组，每个对象包含：
  - `id`: Comment ID
    `id`：评论 ID
  - `author_name`: Name of the comment author
    `author_name`：评论作者姓名
  - `text`: Comment text content
    `text`：评论文本内容
  - `created_time`: Unix timestamp of when the comment was created
    `created_time`：评论创建时间的 Unix 时间戳
  - `reply_count`: Number of replies to this comment
    `reply_count`：该评论的回复数
  - `comment_url`: Direct Facebook URL to this specific comment. Use this when the user wants to see or share a particular comment.
    `comment_url`：指向这条评论的 Facebook 直达 URL。当用户想查看或分享某条特定评论时使用它。
- `summary.post_url`: Permalink to the original post on Facebook. Always present — use this to link the user back to the post. (This moved from the old top-level `post_url` into `summary` when the endpoint became paginated.)
  `summary.post_url`：原帖在 Facebook 上的永久链接。始终存在 —— 用它把用户带回原帖。（该端点改为分页后，此字段从旧版顶层 `post_url` 移入了 `summary`。）
- `paging.cursors.after`: Opaque cursor for the next page. **Present only when more comments exist** — its absence means you've reached the end. Pass it back via `--after` to continue.
  `paging.cursors.after`：下一页的不透明游标。**仅当还有更多评论时才存在** —— 缺失即表示已到末尾。通过 `--after` 传回它以继续获取。

When presenting comments to the user, always include `summary.post_url` so they can navigate to the original post, and mention that each comment has a direct `comment_url` link. To gather more than one page, follow `paging.cursors.after` with `--after` until it's absent.

向用户展示评论时，始终附上 `summary.post_url` 以便用户前往原帖，并说明每条评论都有直达的 `comment_url` 链接。要获取多页内容时，跟随 `paging.cursors.after` 并配合 `--after`，直到该字段不再出现。

## Operating Rules / 操作规则

1. When presenting comments, always include `summary.post_url` for the parent post and `comment_url` for each comment. Never show raw comment IDs or post IDs — always use the URLs.
   展示评论时，始终附上父帖的 `summary.post_url` 和每条评论的 `comment_url`。绝不展示原始评论 ID 或帖子 ID —— 一律使用 URL。
2. When the user asks about "recent" or "latest" comments, always state the date range you used in your response (e.g., "Here are comments from the past 7 days").
   当用户问及“最近”或“最新”评论时，在回复中始终说明你所采用的时间范围（例如，“以下是过去 7 天的评论”）。
3. When organizing comments across multiple posts, present them grouped per post with clear separation.
   跨多个帖子整理评论时，按帖子分组呈现，分组之间有清晰的分隔。
4. Include timestamps (`created_time`) for comments when presenting them.
   展示评论时附上时间戳（`created_time`）。
5. When cross-referencing commenters across posts, accurately identify only people who appear in multiple threads. Do not fabricate commenter names.
   跨帖子交叉比对评论者时，只准确识别确实出现在多个讨论串中的人。不得虚构评论者姓名。

【评论】要求明示所采用的时间范围，可防止把有限抓取的结果笼统当作“最新”呈现给用户，属于减少误导的输出规范。
