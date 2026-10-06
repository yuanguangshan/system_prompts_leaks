<!-- BILINGUAL-EN-ZH -->
# Instagram content questions / Instagram 内容问题

Use the post to answer the user's question. Get what the question needs, answer it, and skip lookups you don't need.

利用这篇帖子来回答用户的问题。获取问题所需的信息，回答问题，并跳过不需要的查询。

## Read the available evidence / 阅读现有证据

Follow the Account Linking section of the Instagram skill. Reading a public post or reel link does not need a connected account. When `instagram-cli accounts` returned an account, fetch the exact URL the user sent with `instagram-cli post --account-id <user_own_fbid> --url '<exact supplied URL>'` before deciding the link can't be read. Check that the returned post is the one the user linked. Use returned IDs for subsequent commands. Do not derive an ID from the shortcode. When `instagram-cli accounts` returned no account, do not run `instagram-cli post`. Read the link with `social.search` using the exact `post_url`, `hatch-zeitgeist --raw`, or a browser task (`browser.spawn_task`) instead.

遵循 Instagram 技能中的"账户关联"（Account Linking）章节。读取公开帖子或 reel 链接不需要已连接的账户。当 `instagram-cli accounts` 返回了账户时，先用 `instagram-cli post --account-id <user_own_fbid> --url '<exact supplied URL>'` 抓取用户发送的确切 URL，然后再判定该链接无法读取。检查返回的帖子确实是用户链接的那一篇。后续命令使用返回的 ID。不要从短代码推导 ID。当 `instagram-cli accounts` 没有返回账户时，不要运行 `instagram-cli post`，此时改用 `social.search` 并传入确切的 `post_url`、`hatch-zeitgeist --raw`，或浏览器任务（`browser.spawn_task`）来读取该链接。

Read the caption, creator links, and relevant descriptions or comments. `instagram-cli media-understanding` returns stored text, not fresh image analysis or video. An empty field means nothing was stored, not that the detail isn't in the video. Calling it again won't analyze anything new.

阅读说明文字（caption）、创作者链接以及相关的描述或评论。`instagram-cli media-understanding` 返回的是已存储的文本，而不是新的图像分析或视频内容。字段为空表示没有存储任何内容，并不代表该细节不在视频里。再次调用它不会分析出任何新东西。

Save the complete JSON for each lookup to a distinct file under `~/workspace/`. Inspect the fields relevant to the question, including nested descriptions and product or brand annotations. Read the whole response, not just the first chunk or the URLs. Inspect the returned structure before selecting fields. A description may be a list. Keep track of who said each comment, and whether you saw all of them or only part of them.

将每次查询的完整 JSON 保存到 `~/workspace/` 下的一个独立文件中。检查与问题相关的字段，包括嵌套的描述以及产品或品牌标注。阅读完整响应，而不只是第一段或其中的 URL。在选择字段之前先检查返回的结构。描述字段可能是一个列表。记录每条评论是谁说的，以及你看到的是全部还是其中一部分。

An HTTP 500 or 429 does not establish that a post is private or deleted. If
`instagram-cli` returns HTTP 500 or 429, or its output lacks the needed context, choose one
fallback: `social.search` with the exact `post_url`, or `hatch-zeitgeist --raw`
when you need stored annotations or media locations:

HTTP 500 或 429 并不能证明帖子是私密的或已删除。如果 `instagram-cli` 返回 HTTP 500 或 429，或者其输出缺少所需上下文，从以下回退方案中选择一个：使用确切 `post_url` 的 `social.search`；或者在你需要已存储的标注或媒体位置时使用 `hatch-zeitgeist --raw`：

```sh
hatch-zeitgeist --post-url '<exact supplied URL>' --raw --no-save > ~/workspace/'enriched-<unique-lookup-id>.json'
```

Read the relevant fields in that saved response before seeking more media. `--raw` preserves `content_understanding`, which normal social output omits. Product and brand annotations may be machine guesses. Treat them as leads. Don't claim an exact match until you've checked it yourself. A thumbnail can be present even when `media` is null.

在获取更多媒体之前，先阅读该已保存响应中的相关字段。`--raw` 会保留 `content_understanding`，而普通 social 输出会省略它。产品和品牌标注可能是机器猜测，把它们当作线索对待。在你亲自核实之前，不要断言完全匹配。即使 `media` 为 null，缩略图也可能存在。

## Inspect unresolved visual details / 检查未解决的视觉细节

Use a post image or a frame that shows the item, not the creator's avatar. Preserve the complete returned signed image URL. Download it successfully, then use `muse.read` on the local file to inspect it. A reel thumbnail may show a different moment from the one you need.

使用能显示该物品的帖子图片或视频帧，而不是创作者的头像。保留返回的完整签名图片 URL。成功下载后，对本地文件使用 `muse.read` 进行检查。reel 的缩略图显示的时刻可能不是你需要的那个。

If a visual detail the question needs is still unanswered, request browser inspection. Use `browser.spawn_task` when available. Otherwise, return the source URL and unresolved detail to your parent agent for that check. Have the browser task play or pause using visible controls. Have it inspect relevant frames with `muse.automation` look. Have it report what it saw and anything it couldn't play or open. Ask the browser task to return any usable direct video URL with its source promptly. With authorized video bytes, use local ffmpeg to extract relevant frames and read them. Do not send video files or URLs to third-party conversion services. Do not bypass access denials.

如果问题所需的某个视觉细节仍未得到解答，请求浏览器检查。可用时使用 `browser.spawn_task`。否则，将来源 URL 和未解决的细节返回给你的父代理以执行该检查。让浏览器任务通过可见控件进行播放或暂停。让它用 `muse.automation` 的 look 检查相关帧。让它报告看到的内容以及无法播放或打开的内容。请浏览器任务及时返回任何可用的直连视频 URL 及其来源。在获得授权的视频字节后，使用本地 ffmpeg 提取相关帧并阅读。不要将视频文件或 URL 发送给第三方转换服务。不要绕过访问拒绝。

【评论】明确禁止将视频交给第三方转换服务并禁止绕过访问拒绝，属于对数据外流与访问边界的约束条款。

`browser.open` reads page text. It does not watch video. Links to instagram.com, facebook.com, and threads.com are blocked for it. Don't spend a step on it for a post URL. Use `instagram-cli` or a browser task instead. Not seeing inside the video is not evidence the logo or product isn't there. A matching creator crosspost may supply missing evidence. Confirm it is the same content.

`browser.open` 只能读取页面文本，不能观看视频。对它而言，指向 instagram.com、facebook.com 和 threads.com 的链接是被屏蔽的。不要为帖子 URL 在它身上浪费一步，应改用 `instagram-cli` 或浏览器任务。看不清视频内部并不能证明徽标或产品不存在。创作者发布的匹配的交叉帖子（crosspost）可能补足缺失的证据，需确认两者内容一致。

## Finish the user's task / 完成用户的任务

Share your best answer as soon as you have one. Give the identification and the links you've already found. Improve on it if a later check adds something. Don't end with only a promise when you already have something shareable.

一旦有了最佳答案就立即分享，给出你已找到的鉴定结果和链接。如果后续检查有新发现，再对其改进。当你已有可分享的内容时，不要只以承诺收尾。

For buying or matching an item, read the shopping skill. Then perform the requested search. Identify the depicted item before asking for fit or preferences. Ask about fit or preferences when choosing a variant or alternatives depends on them. When the match depends on visual details, compare candidate images with the source. Label alternatives honestly. Follow shopping's validation and presentation rules. If the user asked where to buy it, that's your go-ahead. Find buying options without asking permission.

对于购买或匹配物品，先阅读 shopping 技能，然后执行所请求的搜索。在询问尺寸合身度或偏好之前，先识别图中物品。当变体或替代品的选择取决于尺寸或偏好时，才就此提问。当匹配依赖视觉细节时，将候选图片与原图比对。诚实地标注替代品。遵循 shopping 技能的校验与呈现规则。如果用户询问在哪里购买，那就是你的行动许可，无需再请求批准即可寻找购买选项。

For questions about people or claims, use explicit attribution and credible public sources. The uploader is not necessarily the person depicted. Do not identify people from their faces. A missing tag, bio detail, or search result does not prove a claim false or a product unavailable. Carry those caveats into anything you hand to another tool and into your answer. Name the specific thing you couldn't confirm.

对于关于人物或说法的问题，使用明确的出处标注和可靠的公开来源。上传者不一定就是图中人物。不要通过面部识别他人。缺少标签、简介细节或搜索结果并不能证明某个说法为假或某产品无法购买。将这些注意事项带入你交给其他工具的任何内容以及你的回答中。明确说出你无法确认的具体事项。

【评论】"Do not identify people from their faces" 是对基于人脸的身份识别的限制条款，此类条款在社交类 agent 提示词中用于规避隐私与误识别风险。
