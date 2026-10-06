<!-- BILINGUAL-EN-ZH -->
# Facebook Timeline / Facebook 时间线

Fetch posts from a Facebook user's timeline/feed.

从某个 Facebook 用户的时间线/信息流（timeline/feed）中获取帖子。

## Command / 命令

Fetch timeline posts for a user. Use `me` first to get your own profile ID.

获取某个用户的时间线帖子。先用 `me` 获取你自己的 profile ID。

```bash
# Fetch a user's timeline (use `facebook-cli me` to get your own profile_id)
facebook-cli timeline fetch --profile-id 123456789

# Fetch up to 10 posts, then request the next page using the returned next_cursor
facebook-cli timeline fetch --profile-id 123456789 --limit 10
facebook-cli timeline fetch --profile-id 123456789 --limit 10 --after '<next_cursor>'
```

**Options:** / **选项：**

- `--profile-id` (required): Profile ID to fetch timeline for.
  `--profile-id`（必需）：要获取其时间线的 profile ID。
- `--limit`: Maximum posts in this page. The server defaults to 20 and clamps
  larger values to 20.
  `--limit`：本页的最大帖子数。服务器默认为 20，并将更大的值截断为 20。
- `--after`: Opaque `next_cursor` returned by the previous page. `--cursor` is
  accepted as an alias.
  `--after`：上一页返回的不透明 `next_cursor`。也接受 `--cursor` 作为别名。

## Output / 输出

The command returns the compact `social_posts_v1` collection shared with
social search. Read `posts[]` directly; do not guess a provider response path
or pipe the command through `jq`. Each post carries `post_id`, `url`,
`platform`, `created_at`, `username`, `post_caption`, author/owner identity,
and available media fields. `post_caption` prefers authored text and falls back
to an available media summary. Pagination uses `next_cursor` and
`has_next_page`. When `has_next_page` is true, pass `next_cursor` unchanged as
`--after` to fetch the next page for the same profile.

该命令返回与社交搜索共享的紧凑 `social_posts_v1` 集合。直接读取 `posts[]`；不要猜测提供方的响应路径，也不要把命令通过 `jq` 管道处理。每个帖子带有 `post_id`、`url`、`platform`、`created_at`、`username`、`post_caption`、作者/所有者身份信息，以及可用的媒体字段。`post_caption` 优先使用作者撰写的文字，退而使用可用的媒体摘要。分页使用 `next_cursor` 和 `has_next_page`。当 `has_next_page` 为 true 时，将 `next_cursor` 原样作为 `--after` 传入，以获取同一 profile 的下一页。

### Author vs Owner (CRITICAL) / 作者与所有者的区别（关键）

A timeline contains BOTH posts written by the profile owner AND posts written by others on their wall. You MUST distinguish between these:

时间线既包含 profile 所有者本人写的帖子，也包含其他人在其留言墙（wall）上写的帖子。你必须区分这两者：

- **Post by the profile owner**: `author_id == owner_id` (the person wrote their own post)
  **profile 所有者的帖子**：`author_id == owner_id`（该人自己写的帖子）
- **Wall post by someone else**: `author_id != owner_id` — a friend or other person posted ON the profile owner's wall/timeline
  **他人写的留言墙帖子**：`author_id != owner_id`——朋友或其他人在 profile 所有者的留言墙/时间线上发的帖子

## Example Output / 输出示例

```json
{
  "format": "social_posts_v1",
  "count": 1,
  "posts": [{
    "rank": 1,
    "post_id": "pfbid02...",
    "url": "https://www.facebook.com/...",
    "platform": "facebook",
    "created_at": "2024-03-02T17:00:00+00:00",
    "post_caption": "Check out this sunset!",
    "author_name": "Michael Santoro",
    "author_id": "123456789",
    "owner_name": "Michael Santoro",
    "owner_id": "123456789",
    "media_summary": "The image shows a sunset over the ocean."
  }],
  "next_cursor": "...",
  "has_next_page": true
}
```

## Operating Rules / 操作规则

1. **Only fetch timelines for IDs that came from an earlier command.** Pass a `--profile-id` only when you obtained it from earlier facebook-cli output in this conversation — `me` (your own), `me friends`, feed/group post authors, reactors/commenters, saved items, or another timeline post's `author_id`/`owner_id`. Do **not** accept a raw numeric profile ID the user typed or pasted directly, and do **not** guess or construct IDs. If the user supplies a bare ID with no context, identify the person first via `me friends --name` and use the ID from that result.
   **只获取来自先前命令的 ID 所对应的时间线。** 仅当某个 `--profile-id` 是你在本会话中从早先的 facebook-cli 输出中获得的才传入——来源可以是 `me`（你自己的）、`me friends`、信息流/群组帖子作者、点赞者/评论者、收藏项，或另一条时间线帖子的 `author_id`/`owner_id`。**不要**接受用户直接输入或粘贴的原始数字 profile ID，也**不要**猜测或构造 ID。如果用户提供了没有上下文的裸 ID，先通过 `me friends --name` 确认该用户身份，再使用该结果中的 ID。
2. **Always distinguish author from timeline owner.** When `author_id != owner_id`, clearly state that the post was written by `author_name` on `owner_name`'s wall — it is NOT a post by the timeline owner. Refer to people by name — do not print raw `owner_id`/`author_id` values in your response.
   **始终区分作者与时间线所有者。** 当 `author_id != owner_id` 时，明确说明该帖子是 `author_name` 写在 `owner_name` 留言墙上的——它不是时间线所有者本人发的帖子。用姓名称呼相关人物——不要在回复中打印原始 `owner_id`/`author_id` 值。
3. **When asked "show me posts FROM [person]" or "what has [person] posted"**: Filter results to only posts where `author_id == owner_id`. If the person has no self-authored posts in the results, say so clearly: "[Person] hasn't posted recently." Then optionally mention: "However, friends have posted on their wall" and summarize those separately.
   **当被问及"给我看 [某人] 发的帖子"或"[某人] 发过什么"时**：把结果过滤为仅保留 `author_id == owner_id` 的帖子。如果该人在结果中没有自己发的帖子，清楚说明："[某人] 最近没有发帖。"然后可以补充："不过，朋友们在 TA 的留言墙上发过帖子"，并单独概述这些帖子。
4. **When asked "what's on [person]'s timeline"**: Show all posts, but clearly label each one — "Posted by [author_name]" for self-authored posts, and "[author_name] posted on [owner_name]'s wall" for wall posts by others.
   **当被问及"[某人] 的时间线上有什么"时**：展示所有帖子，但为每条清楚标注——本人发的帖子标注"由 [author_name] 发布"，他人的留言墙帖子标注"[author_name] 发布在 [owner_name] 的留言墙上"。
5. When presenting timeline posts, include `post_caption` and `url` when present. Do not fabricate links or text when these fields are absent.
   展示时间线帖子时，若存在 `post_caption` 和 `url` 则一并给出。这些字段缺失时，不要编造链接或文字。
6. When summarizing a timeline, ground every claim in a specific post and include a link to that post when available. Do not fabricate or infer activities beyond what the posts say.
   概述时间线时，每个论断都要落在具体帖子上，并在可行时附上该帖子的链接。不要在帖子内容之外编造或推断活动。
7. Use `media_summary` to describe an image or video and `media_ocr` or `video_transcript` only when the normalized post includes them.
   使用 `media_summary` 描述图片或视频；只有当规范化后的帖子包含 `media_ocr` 或 `video_transcript` 时才使用它们。
8. Media fields are pre-computed and may be absent. Never claim a post contains specific media content unless one of those fields confirms it.
   媒体字段是预先计算好的，可能缺失。除非这些字段之一予以确认，绝不要声称某个帖子包含特定的媒体内容。
9. A media-only post can carry its available summary in `post_caption`; present it as a description, not as the author's quoted words.
   纯媒体帖子的可用摘要可能放在 `post_caption` 中；呈现时应将其作为描述，而不是作者的引语。
10. When the user asks for "recent", "latest", or "lately" posts, always state the date range you used in your response (e.g., "Here are posts from the past 7 days" or "Showing posts from April 1–7, 2026").
    当用户要求"最近"（recent/latest/lately）的帖子时，始终在回复中说明你所使用的日期范围（例如"以下是过去 7 天的帖子"或"显示 2026 年 4 月 1 日至 7 日的帖子"）。
11. When the user references a post by ordinal position (e.g., "the second post", "the first one"), resolve it from the results of the previous turn. Do not ask the user to repeat which post they mean.
    当用户以序数位置指代某个帖子（如"第二条帖子"、"第一条"）时，从上一轮的结果中解析它。不要让用户重复说明指的是哪条帖子。
12. Do not infer meaning beyond what is explicitly written in post text. A post about a maternity photographer does not mean the user is pregnant. Report content literally without speculation.
    不要在帖子文字明确表述之外推断含义。一篇关于孕味摄影师的帖子并不意味着用户怀孕了。按字面报告内容，不做臆测。

【评论】规则 1 只接受来自工具输出的 ID、明确拒绝用户直接输入的 ID，属于针对提示词注入与身份混淆的输入校验设计；规则 12 的"按字面报告"则是不做推断的反幻觉约束。
