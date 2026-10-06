<!-- BILINGUAL-EN-ZH -->

# Facebook Reactions / Facebook 表情回应

Read the reaction summary on Facebook posts.

读取 Facebook 帖子上的表情回应（reaction）摘要。

## Command / 命令

```bash
facebook-cli post reactions read --post-id <post-id>
```

**Options:**
- `--post-id` (required): Post ID to read reactions from
  `--post-id`（必需）：要读取表情回应的帖子 ID

The endpoint returns the reaction **summary** only (total count and per-type
counts). An individual reactor list is not available from this command.

该端点只返回表情回应的 **summary**（摘要）：总计数和按类型的计数。单个回应者的列表无法通过此命令获取。

## Response Fields / 响应字段

The response carries the aggregates under `summary`:

响应在 `summary` 下携带聚合数据：

- `summary.total_count`: Total number of reactions on the post
  `summary.total_count`：帖子上的表情回应总数
- `summary.reaction_counts`: Array of `{ reaction_type, count }` — breakdown by type (Like, Love, Wow, Haha, Sad, Angry, Thankful, Care)
  `summary.reaction_counts`：`{ reaction_type, count }` 数组——按类型细分（Like、Love、Wow、Haha、Sad、Angry、Thankful、Care）

`data` is an empty array (no individual reactor list is returned).

`data` 是空数组（不返回单个回应者列表）。

## Operating Rules / 操作规则

1. When presenting reactions, include the breakdown by type (Like, Love, Haha, Wow, Sad, Angry) from `summary.reaction_counts`, plus the total from `summary.total_count`. Always include the post permalink — never show raw post IDs. Individual reactor names ("who reacted"/"who liked it") are not available from this command; if the user asks who reacted, say that the reactor list isn't available and offer the per-type counts instead.
   呈现表情回应时，要包含来自 `summary.reaction_counts` 的按类型细分（Like、Love、Haha、Wow、Sad、Angry），以及来自 `summary.total_count` 的总数。始终附带帖子永久链接——绝不显示原始帖子 ID。单个回应者的姓名（"谁回应了"/"谁点赞了"）无法通过此命令获取；如果用户问是谁回应的，应说明回应者列表不可用，并改为提供按类型的计数。
2. When the user asks about "recent" or "latest" reactions, always state the date range you used in your response (e.g., "Here are posts with reactions from the past 7 days").
   当用户询问"最近"或"最新"的表情回应时，始终在回答中说明所使用的日期范围（例如"以下是过去 7 天内有回应的帖子"）。
3. When comparing reactions across posts, present clear numerical comparisons grounded in actual data. Include post links for each post being compared.
   跨帖子比较表情回应时，基于实际数据给出清晰的数值比较，并为每个被比较的帖子附上链接。
4. Do not use subjective language like "very popular" or "went viral" without grounding in specific numbers.
   没有具体数字支撑时，不要使用"非常火"、"刷屏了"之类的主观说法。
5. When the user asks to filter reactions by type using informal terms (e.g., "funny ones", "sad reactions", "the laughing ones"), clarify which Facebook reaction type they mean before proceeding. Map common terms: "funny" / "laughing" → Haha, "sad" → Sad, "angry" / "mad" → Angry, "hearts" / "love" → Love. If the mapping is ambiguous, ask the user to confirm.
   当用户用非正式说法按类型过滤表情回应时（例如"好笑的那些"、"悲伤的回应"、"笑的那些"），先澄清用户指的是哪种 Facebook 表情回应类型再继续。常见说法映射："好笑"/"笑" → Haha，"悲伤" → Sad，"生气"/"愤怒" → Angry，"爱心"/"喜欢" → Love。如果映射不明确，请用户确认。

【评论】"绝不显示原始帖子 ID"与第 4 条"禁止无数字支撑的主观说法"共同体现了克制的呈现原则：只转述可验证的聚合数据，不放大也不臆测。
