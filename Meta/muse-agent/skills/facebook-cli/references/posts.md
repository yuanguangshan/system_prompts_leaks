<!-- BILINGUAL-EN-ZH -->
# Facebook Posts / Facebook 帖子

## Reading a Post / 读取帖子

```bash
facebook-cli post read --post-id <post-id>
facebook-cli post read --url '<canonical-facebook-url>'
```

Supply exactly one nonblank `--post-id` or `--url`. IDs may be numeric or bare  
PFBIDs. Output uses the normalized `social_posts_v1` format with the post's text,
permalink, and available media context.

必须且只能提供一个非空的 `--post-id` 或 `--url`。ID 可以是数字形式，也可以是不带前缀的 PFBID。输出采用规范化的 `social_posts_v1` 格式，包含帖子文本、永久链接以及可用的媒体上下文。

## Reading from a Facebook URL / 从 Facebook URL 读取

Resolve Facebook share wrappers such as `/share/<token>/` or
`/share/p/<token>/` before reading the content:

在读取内容之前，先解析 `/share/<token>/` 或 `/share/p/<token>/` 之类的 Facebook 分享包装链接：

```bash
facebook-cli link-sharing decode-url \
  --url 'https://www.facebook.com/share/p/<share-token>/'
```

The response includes `original_url`. If it is null, missing, or blank, report
that the link could not be resolved and stop. Do not use the share token as an ID.
A decoded URL identifies the content type; it does not confirm that content is
available to the account. The post reader does not decode share links itself.

响应中包含 `original_url`。如果它为 null、缺失或为空，报告该链接无法解析并停止。不要把分享令牌当作 ID 使用。解码后的 URL 标识的是内容类型；它并不确认该内容对该账户可用。帖子读取器本身不会解码分享链接。

For an already canonical URL, skip decoding. Pass the complete supplied URL or
decoded `original_url` to `post read --url` for these HTTP(S) URL patterns on
`facebook.com`, `www.facebook.com`, `m.facebook.com`, `mbasic.facebook.com`,
`web.facebook.com`, or `touch.facebook.com`:

对于已经是规范化形式的 URL，跳过解码。针对 `facebook.com`、`www.facebook.com`、`m.facebook.com`、`mbasic.facebook.com`、`web.facebook.com` 或 `touch.facebook.com` 上的这些 HTTP(S) URL 模式，把完整提供的 URL 或解码得到的 `original_url` 传给 `post read --url`：

| Content | Canonical URL patterns |
| --- | --- |
| Post | `/<profile>/posts/<post-id>` or `/posts/<post-id>` |
| Group post | `/groups/<group>/posts/<post-id>` or `/groups/<group>/permalink/<post-id>` |
| Post permalink | `/story.php?story_fbid=<post-id>` or `/permalink.php?story_fbid=<post-id>` |
| Photo | `/photo.php?fbid=<id>` or `/photo/?fbid=<id>` |
| Video or reel | `/reel/<id>`, `/videos/<id>`, or `/<profile>/videos/<id>` |
| Video query URL | `/watch/?v=<id>` or `/video.php?v=<id>` |

| 内容 | 规范化 URL 模式 |
| --- | --- |
| 帖子 | `/<profile>/posts/<post-id>` 或 `/posts/<post-id>` |
| 小组帖子 | `/groups/<group>/posts/<post-id>` 或 `/groups/<group>/permalink/<post-id>` |
| 帖子永久链接 | `/story.php?story_fbid=<post-id>` 或 `/permalink.php?story_fbid=<post-id>` |
| 照片 | `/photo.php?fbid=<id>` 或 `/photo/?fbid=<id>` |
| 视频或 reel | `/reel/<id>`、`/videos/<id>` 或 `/<profile>/videos/<id>` |
| 视频查询 URL | `/watch/?v=<id>` 或 `/video.php?v=<id>` |

```bash
facebook-cli post read --url '<original_url-from-decoder>'
```

Keep the URL intact, including its query parameters; do not extract an ID or
make a separate media-resolution request. The reader resolves supported
photo/video URLs to their containing post.
Standalone media may have no containing post. A failed read stops the chain:
report the failure without guessing an ID, retrying the media ID as a post ID,
or claiming that the content is readable from the URL alone.

保持 URL 原样，包括其查询参数；不要提取 ID，也不要单独发起媒体解析请求。读取器会把受支持的照片/视频 URL 解析到其所属帖子。独立媒体可能没有所属帖子。读取失败即中止整个链条：报告失败即可，不要猜测 ID，不要把媒体 ID 当帖子 ID 重试，也不要宣称仅凭 URL 就能读到内容。

Malformed or unsupported URLs, including undecoded share links, return HTTP 400.  
Supported URLs whose media or post is missing, private, or unreadable return  
HTTP 404. Neither result provides post content.

格式错误或不受支持的 URL（包括未解码的分享链接）返回 HTTP 400。受支持的 URL 若其媒体或帖子缺失、私有或不可读，返回 HTTP 404。两种结果都不提供帖子内容。

### Other Entities / 其他实体

Route other entities to their own readers:

把其他实体路由到各自的读取器：

- `/groups/<group-id>`: use `groups posts --group-id <group-id>` to browse posts.  
  If the group path contains a name instead of an ID, use `groups search` first.
- `/groups/<group-id>`：使用 `groups posts --group-id <group-id>` 浏览帖子。如果小组路径中是名称而不是 ID，先用 `groups search`。
- `/marketplace/item/<listing-id>`: use `marketplace listing details --url '<decoded-url>'` with the full URL intact.
- `/marketplace/item/<listing-id>`：使用 `marketplace listing details --url '<decoded-url>'`，保持完整 URL 原样。
- Event, game, and unrecognized URLs: report that this URL type is unsupported. Do not
  pass an arbitrary number from the URL to `post read`.
- 活动、游戏和无法识别的 URL：报告该 URL 类型不受支持。不要把 URL 中出现的任意数字传给 `post read`。

## Browsing a Profile's Timeline / 浏览个人主页时间线

To view recent posts for a profile, use `timeline fetch`:

要查看某个主页的近期帖子，使用 `timeline fetch`：

```bash
facebook-cli timeline fetch --profile-id <profile-id>
```

## Reading Comments or Reactions / 读取评论或表情反应

```bash
facebook-cli post comments read --post-id <post-id>
facebook-cli post reactions read --post-id <post-id>
```

## Operating Rules / 操作规则

1. When presenting normalized post, feed, or timeline results, include `post_caption` and `url` when present. A feed caption that matches `header_text`, or a caption derived from `media_summary`, is descriptive provider text rather than the author's quoted words. Do not fabricate links or text when these fields are absent.
2. 在呈现规范化后的帖子、信息流或时间线结果时，若 `post_caption` 和 `url` 存在则包含它们。与 `header_text` 一致的信息流标题，或由 `media_summary` 派生的标题，属于提供方的描述性文字，而不是作者的原话。这些字段缺失时，不要编造链接或文字。
2. Include the owner's name only when the user isn't asking about a specific person (e.g., browsing a feed). Omit it when the context already makes the author obvious.
3. 仅当用户不是在询问某个特定的人时（例如浏览信息流），才包含所有者姓名。当上下文已能明显看出作者时，省略它。
3. When summarizing what someone has been up to, ground every claim in a specific post. Do not fabricate or infer activities that are not evidenced by an actual post.
4. 在总结某人的近况时，每一句论断都要有具体帖子作依据。不要编造或推断没有实际帖子佐证的活动。
4. Use normalized `media_summary`, `media_ocr`, or `video_transcript` fields when present to provide media context.
5. 当 `media_summary`、`media_ocr` 或 `video_transcript` 等规范化字段存在时，用它们提供媒体上下文。
5. Media fields are pre-computed and may not be available for all posts. Never claim a post contains specific media content unless one of those fields confirms it.
6. 媒体字段是预先计算好的，未必对所有帖子可用。除非这些字段之一予以确认，绝不要宣称某条帖子包含特定的媒体内容。
6. Facebook video and reel posts may also carry shopping context: `shoppable_products` (a list of detected products, each with `product_name` and optional `descriptive_name`, `category`, `item_type`, `brand_name`, `color`, `style`, `gender`, `prominence`), `shoppability`, and `shoppable_product_category`. Use them to answer questions about what is shown or sold in a video instead of inferring products from the caption or transcript. They are absent on posts with no shopping content understanding, and on list endpoints (`profile posts`, `feed`) — only `post read` returns them.
7. Facebook 视频和 reel 帖子还可能携带购物上下文：`shoppable_products`（检测到的商品列表，每项含 `product_name` 和可选的 `descriptive_name`、`category`、`item_type`、`brand_name`、`color`、`style`、`gender`、`prominence`）、`shoppability` 和 `shoppable_product_category`。回答视频中展示或销售了什么之类的问题时使用它们，而不是从标题或文字记录中推断商品。在无购物内容理解的帖子上，以及在列表端点（`profile posts`、`feed`）上，这些字段不存在——只有 `post read` 会返回它们。
