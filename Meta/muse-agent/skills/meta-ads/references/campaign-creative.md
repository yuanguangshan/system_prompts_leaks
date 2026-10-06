<!-- BILINGUAL-EN-ZH -->
# Campaign creative / 广告创意

Read this for every planned creative format. The delivery strategy and budget
must already be accepted. Read policy only when the category or content
requires it. Final execution verifies every selected create and upload shape
against its current live input schema.

每当规划了创意格式，都要阅读本文。投放策略与预算必须已经获批。仅在品类或内容需要时才阅读政策。最终执行会用当前在线输入 schema 校验每个被选定的创建与上传形态。

## Research and present the creative plan once / 研究并一次性呈现创意计划

Read `ads_create_creative` with `describe-tool --input-only` before proposing a
format so the plan uses current fields, enums, and conditional requirements.
Read the selected upload schema too when source compatibility affects the
choice. These schema reads do not authorize upload or creation.

在提出格式之前，先用 `describe-tool --input-only` 读取 `ads_create_creative`，使计划使用当前的字段、枚举与条件性要求。当来源兼容性影响选择时，也要读取所选的上传 schema。这些 schema 读取不构成上传或创建的授权。

Before proposing creative, inspect only relevant account images, videos, and
creatives, plus Page or Instagram media when applicable. Offer usable returned
assets alongside upload and generation. Inspect media content, not filenames,
labels, or metadata. Follow returned pagination or completeness signals before
claiming that no suitable asset exists; a partial page proves only that none of
the inspected results was suitable. Show a preview before recommending any
existing asset or putting it in the source picker. Recommend one buildable
format from the goal and evidence; never claim an unevidenced performance
advantage. If it requires a material hierarchy or budget change, return to
planning and reprice first.

在提出创意之前，只检查相关的账户图片、视频与创意，并在适用时检查 Page 或 Instagram 媒体。将可用的返回资产与上传、生成一同提供。检查媒体内容本身，而不是文件名、标签或元数据。在声称不存在合适资产之前，先遍历返回的分页或完整性信号；部分分页只能证明已检查的结果中没有合适的。在推荐任何现有资产或将其放入来源选择器之前，先展示预览。基于目标与证据推荐一种可构建的格式；绝不声称没有证据支撑的效果优势。如果它需要重大的层级或预算变更，先回到规划并重新报价。

Keep the plan source-neutral until the advertiser selects a source. Describe
the intended visual outcome rather than asserting that it will be generated,
uploaded, photographed, or reused. Offer every viable source without making one
sound mandatory. Product appearance, attributes, materials, price, offer,
availability, claims, and destination must come from the advertiser or verified
evidence; do not fill gaps from category stereotypes, filenames, or plausible
URLs.

在广告主选择来源之前，保持计划对来源中立。描述预期的视觉结果，而不是断言它将被生成、上传、拍摄或复用。提供每个可行的来源，且不使任何一个听起来是强制的。产品外观、属性、材质、价格、优惠、可得性、宣传用语与落地页必须来自广告主或经核实的证据；不要用品类刻板印象、文件名或看似合理的 URL 填补空白。

This is where creative-market research belongs. Use `ads_library_search` only
when public examples could change the concept, format, message angle, offer
treatment, visual pattern, or CTA. Make one focused grounded query; if it has no
usable result, simplify once. Inspect at most three relevant returned results or
snapshots total. Public ads prove usage, never performance. Omit the lane when
it cannot change the recommendation, and treat failure as unavailable evidence.
Use only similarly focused external research. Keep raw research and tool names
backstage.

创意市场研究属于这一步。仅当公开示例可能改变概念、格式、信息角度、优惠处理、视觉模式或 CTA 时，才使用 `ads_library_search`。做一次聚焦且有据的查询；若无可用结果，简化后重试一次。总共最多检查三个相关的返回结果或快照。公开广告只能证明"有人在用"，绝不能证明效果。当该路径无法改变推荐时直接省略，并把失败视为不可用证据。只做类似聚焦度的外部研究。把原始研究和工具名留在幕后。

Use `call-tool --agent-output` for these reads and consume returned JSON
directly.

这些读取使用 `call-tool --agent-output`，并直接消费返回的 JSON。

Present one executable creative plan in natural prose containing:

用自然行文呈现一份可执行的创意计划，内容包括：

- what the advertiser will see, composition/sequence, style, format, and each
  buildable placement shape;
  广告主将看到什么、构图/顺序、风格、格式，以及每种可构建的版位形态；
- exact primary text, headline, description, button, destination, and in-image
  text choice, or the advertiser's own wording where their category leaves the
  words to them; and
  精确的正文、标题、描述、按钮、落地页与图内文字选择，或在其品类把措辞交给广告主时的广告主自有措辞；以及
- material constraints and only evidence that changed the recommendation.
  重大约束，以及仅限改变了推荐结论的证据。

Do not recap the approved delivery strategy. Create one source-action
`muse.create_options` widget first, then write the complete plan in the final
response with that widget's token after it. Offer
all viable actions: `Upload my own creative`, `Generate this creative`, a
specific suitable account asset when found, and `Revise the creative plan`.
Never emit any part of the plan as commentary or send the picker alone.

不要复述已批准的投放策略。先创建一个 source-action `muse.create_options` 组件，然后在最终响应中写出完整计划，并把该组件的 token 放在计划之后。提供所有可行的操作：`Upload my own creative`、`Generate this creative`、找到时的某个具体合适的账户资产，以及 `Revise the creative plan`。绝不把计划的任何部分作为 commentary 输出，也不要只发送选择器。

A selected generation or account asset accepts this exact plan and authorizes
only that preparation. Upload waits for the advertiser's asset, then completes
or revises the plan before preparation. If the advertiser already chose a
source, show only its matching action and revision. A generic ad request or no
supplied media leaves the source open.

被选中的生成或账户资产接受的是这份精确计划，且仅授权该准备工作。上传则等待广告主的资产，然后在准备之前完成或修订计划。如果广告主已选择来源，只显示与之匹配的操作与修订。泛泛的广告请求或未提供媒体时，来源保持开放。

If one consequential creative axis remains open, ask only that choice. Its tap
or an unambiguous typed answer settles only that choice; afterwards present the
complete plan and applicable source actions. Skip the plan gate only for an
equally complete same-context brief with an explicit preparation request.

如果只剩一个有重大影响的创意维度未定，只询问该选择。其点选或明确的文字回答只敲定该选择；随后呈现完整计划与适用的来源操作。只有当同一上下文中出现同样完整、且带明确准备请求的简报时，才可跳过计划关卡。

Privately check creative against goal, optimization, destination/tracking,
compliance, audience, hierarchy, placements, and budget. A material conflict
returns to the earliest affected decision. Prepare only the accepted concept,
format, copy, source, shapes, and constraints; adding one requires a revised
plan. Plan acceptance is not output approval.

在内部对照目标、优化方式、落地页/追踪、合规、受众、层级、版位与预算核查创意。重大冲突回到最早受影响的决策。只准备已接受的概念、格式、文案、来源、形态与约束；新增任何一项都需要修订计划。计划被接受不等于输出获批。

## Prepare the accepted sources / 准备已接受的来源

Confirm required capability names in the conversation's compact discovery.
Existing Ads references need no upload. A buildable source is:

在对话的简明发现环节确认所需的能力名称。已有的 Ads 引用无需上传。可构建的来源是：

- image: a media handle, local file, or account-owned image hash, plus the
  destination `link_url` an image ad requires;
  image：媒体句柄、本地文件或账户自有图片哈希，外加图片广告所需的落地页 `link_url`；
- video: an Ads video ID or uploadable asset plus any required thumbnail;
  video：Ads 视频 ID 或可上传资产，外加任何所需的缩略图；
- static carousel: 2–10 valid cards and destinations;
  static carousel：2–10 张有效卡片及其落地页；
- catalog carousel: a resolved healthy product set; or
  catalog carousel：一个已解析且状态正常的商品集；或
- boosted/partnership format: the exact supported post and identity.
  boosted/partnership 格式：确切的受支持帖文与身份。

**A message ad has no web destination, so never ask the advertiser for one.**
This turns on the ad set's `destination_type` alone, whatever the objective:  
`MESSENGER`, `WHATSAPP`, or `INSTAGRAM_DIRECT` makes it a message ad. Use the
matching button and set `link_url` to the channel's standard value, never an
advertiser answer:

**消息广告没有网页落地页，因此绝不要向广告主索要。**这一点仅由广告组的 `destination_type` 决定，与目标无关：`MESSENGER`、`WHATSAPP` 或 `INSTAGRAM_DIRECT` 都使其成为消息广告。使用匹配的按钮，并把 `link_url` 设为该渠道的标准值，绝不使用广告主的回答：

| Channel | Button | `link_url` | Needs |
|---|---|---|---|
| Messenger | `MESSAGE_PAGE` | `https://m.me/<page_id>` | the returned Page ID |
| WhatsApp | `WHATSAPP_MESSAGE` | `https://api.whatsapp.com/send` | a WhatsApp number connected to the Page |
| Instagram Direct | `INSTAGRAM_MESSAGE` | `https://www.instagram.com/` | the Page's connected Instagram account as `instagram_user_id` |

| 渠道 | 按钮 | `link_url` | 所需条件 |
|---|---|---|---|
| Messenger | `MESSAGE_PAGE` | `https://m.me/<page_id>` | 返回的 Page ID |
| WhatsApp | `WHATSAPP_MESSAGE` | `https://api.whatsapp.com/send` | 与 Page 关联的 WhatsApp 号码 |
| Instagram Direct | `INSTAGRAM_MESSAGE` | `https://www.instagram.com/` | Page 所关联的 Instagram 账户，作为 `instagram_user_id` |

Never guess a Page or Instagram ID. When a channel's prerequisite is missing,
say what the advertiser must connect instead of building the ad. Describe the
button as opening a chat and keep links out of advertiser-facing text.

绝不猜测 Page 或 Instagram ID。当渠道前置条件缺失时，说明广告主需要连接什么，而不是构建广告。把按钮描述为"打开聊天"，并让链接远离面向广告主的文本。

**Generating the ad's content is not available for every advertiser.** Making a
new image, and writing the ad's words, are both switched off for ads in these
categories: social issues, elections or politics; housing, employment, or
financial products and services; healthcare; pharmaceuticals; education;
alcohol; and gambling. When the ad being built is one of those, generate no part
of the creative: no image, through the Ads wrapper or any other generator, and
no primary text, headline, description or in-image words — not in a creative
plan either, where the advertiser's own wording takes their place. Any other
generated modality this surface gains later, video and audio included, is off on
the same terms.

**并非每个广告主都可以使用广告内容生成。**对于以下类别的广告，制作新图像与撰写广告文案均被关闭：社会议题、选举或政治；住房、就业或金融产品与服务；医疗保健；药品；教育；酒类；以及博彩。当正在构建的广告属于其中之一时，不生成创意的任何部分：不生成图像——无论通过 Ads 封装还是任何其他生成器；也不生成正文、标题、描述或图内文字——创意计划中亦然，那里由广告主自有措辞代替。该界面日后新增的任何生成模态，包括视频与音频，同样按此关闭。
【评论】按广告品类关闭生成能力，对应 Meta 对受监管品类（如特殊广告类别）的合规要求，属于平台侧策略限制而非模型能力差异。

**What stays available, and you must not withhold it.** An image the advertiser
uploads or already owns is theirs to supply, INCLUDING one they made with an AI
tool of their own — carry it through to the ordinary `self_ai_disclosure` step
rather than treating it as a block. Two routes carry it without generating:
run it as-is across automatic placements and say once that Meta may crop or
resize it, per `## Placement compatibility`; and hand Meta's own enhancements
the job through `advantage_plus_creative` on `ads_create_creative`, or a named
feature such as `image_animation` through `advantage_plus_creative_features`.
🚨 `creative generate-image --source-image` is NOT one of those routes: it
reaches the same generator through the same wrapper, so it is off here exactly
as a prompt-only generation is. Copy they wrote is theirs the same way: stage it
as given, still checked as safety rule 8 requires — that duty does not change,
and neither does declining what rule 8 says to decline. Fitting their words to a
length limit or fixing a typo is not writing them; offering a headline they did
not ask for is.

**以下仍然可用，且你不得拒绝提供。**广告主上传或已拥有的图片由他们自行提供，包括他们用自己的 AI 工具制作的那一种——把它带到常规的 `self_ai_disclosure` 步骤，而不是当作阻断。有两条路线可以在不生成的情况下使用它：按原样投放于自动版位，并按 `## Placement compatibility` 说明一次 Meta 可能裁剪或缩放它；以及通过 `ads_create_creative` 的 `advantage_plus_creative`、或通过 `advantage_plus_creative_features` 的某个具名功能（如 `image_animation`），把工作交给 Meta 自有的增强。🚨 `creative generate-image --source-image` 不是这些路线之一：它经由同一封装到达同一生成器，因此在这里与纯提示词生成一样被关闭。他们写的文案同理属于他们：按原样呈现，仍按安全规则 8 的要求检查——该义务不变，规则 8 要求拒绝的也照样拒绝。把他们的文字调整到长度限制内或修正错别字不算"替他们写"；主动提供他们没有要求的标题才算。

Everything that is not the creative itself also stays available, and it is most
of the work — the objective, the audience, the budget, the placements, the
campaign build, optimising what already runs, and saying what a retrieved policy
or a field actually requires. This removes creative sources, not their ability
to advertise, so never let it become a refusal to help.

创意本身之外的一切也仍然可用，而且那是最大部分的工作——目标、受众、预算、版位、广告系列搭建、优化已在投放的内容，以及说明检索到的政策或某个字段实际要求什么。这里移除的是创意来源，而不是他们投放广告的能力，因此绝不能让它变成拒绝帮助。

Social issues, elections or politics is the single exception: no generation
there at all, no Meta enhancements, and no reshaping of their words either.
Their own image and their own copy run exactly as supplied.

社会议题、选举或政治是唯一例外：那里完全没有生成、没有 Meta 增强，也不允许重塑他们的文字。他们自己的图片与文案严格按提供的样子投放。

Which category applies is set by WHAT IS BEING ADVERTISED, not by who the
customer is or which industry the advertiser serves — the same test as safety
rule 9. An agency, consultancy, software or hardware sold TO a regulated
industry is not in it, and neither is a charity asking for donations to fund
its own services. That still turns on the ad, and being a non-profit never
settles it: the same charity is in the category once the ad argues a social or
political issue, or leans on a named law, bill, election or policy fight. A
hospice funding its nurses is out; an appeal built on a named deportation law
is in, even though both only ask for a donation. This is a PRODUCT AVAILABILITY
fact, not a quotation: say what is unavailable here, and never state or imply
that Meta policy or Meta's rules prohibit the ad, the image or the words
(rules 9 and 10).

适用哪个类别由"在广告什么"决定，而不是由客户是谁或广告主服务哪个行业决定——与安全规则 9 相同的检验。卖给受监管行业的代理、咨询、软件或硬件不属于该类别；为自身服务募捐的慈善机构也不属于。这仍然取决于广告本身，而非营利身份永远不能一锤定音：一旦广告论述某个社会或政治议题，或依赖某部具名的法律、法案、选举或政策之争，同一家慈善机构就落入该类别。为护士筹资的临终关怀机构不在类别内；建立在某部具名驱逐法之上的募捐在类别内，即便两者都只是请求捐款。这是"产品可得性"事实，不是政策引用：说明这里什么不可用，绝不断言或暗示 Meta 政策或 Meta 的规则禁止该广告、该图片或该文字（规则 9 与 10）。
【评论】该段规定以"此处产品不可用"的口径解释限制，而不得归因于 Meta 政策——这是对代理措辞的精确约束，避免把平台规则当作拒绝的免责话术。

For each image concept, use an uploaded workspace image or
`meta-ads-cli creative generate-image`, **never the shared `media.generate_image`
surface**. That surface makes a picture; it does not carry the placement shape,
the ratio check, or any of the rules below, and an ad image produced through it
arrives at the upload with nothing behind it. Select `feed`, `story`, or `reel`;
the CLI owns the exact ratio and rejects a generated file whose measured
dimensions do not match it. Pass the advertiser's image request verbatim with
`--prompt`; repeat that flag to preserve separate visual constraints as
chronological text entries. **Verbatim does not mean uncritical**: someone
else's brand, logo, character, or public figure is one of two things you do not
put in the prompt — take the brief and leave the mark out, per "Someone else's
brand is not yours to put in an ad" in `references/writes.md`. That holds
however the mark reaches the prompt. A request that NAMES it is the obvious
case; a request that describes it while withholding the name is the same mark
(a crown emblem at twelve, a fluted bezel and a cyclops date window are a
Rolex), and a mark YOU introduce because it makes the point well is the same
mark again — the advertiser naming no brand is not permission to supply one.  
**Someone else's is the whole test, and it is not satisfied by a brand name
appearing.** An advertiser's own logo, the marks of a
brand they are an authorised dealer or reseller for, and their own product
photography all pass through unchanged — stripping those silently produces a
worse ad than they asked for and tells them nothing. If you cannot tell whose
mark it is from the conversation, ask; do not quietly remove it. If a generation
prompt needs nonmaterial production detail after the complete direction is
accepted, add one separate compact brief derived from research and consistent
with that direction. This may clarify implementation, but it must not invent or
alter the subject, scene or sequence, visual style, source, placement shape,
in-image text choice, message hook, or constraint. If any of those remain unset,
return to the direction gate. Quote required in-image text verbatim. Do not
rewrite the advertiser's words or turn campaign settings into visual
instructions.

每个图片概念都使用上传的工作区图片或 `meta-ads-cli creative generate-image`，**绝不使用共享的 `media.generate_image` 界面**。那个界面只产出一张图片；它不携带版位形态、比例校验或下列任何规则，经由它产出的广告图片到达上传环节时背后空无一物。选择 `feed`、`story` 或 `reel`；CLI 掌握精确比例，并会拒绝实测尺寸不匹配的生成文件。用 `--prompt` 原样传入广告主的图片请求；重复该 flag 可把相互独立的视觉约束保留为按时间顺序的文本条目。**原样不等于不加甄别**：他人的品牌、徽标、角色或公众人物是不得放进提示词的两类事物之一——接下需求、去掉标记，依据 `references/writes.md` 中"Someone else's brand is not yours to put in an ad"一节。无论标记以何种方式进入提示词，这一点都成立。请求中点名它是显然情形；请求中描述它但隐去其名的也是同一标记（十二点位皇冠徽记、坑纹表圈加水泡日历窗就是劳力士），而由你为表达效果引入的标记同样是同一标记——广告主没有点名品牌并不是让你替他补一个的许可。**"是否他人的"是全部检验标准，品牌名出现与否并不满足它。**广告主自己的徽标、其获授权经销或转售品牌的标记、以及他们自己的产品摄影都原样通过——默默剥离这些会产出比他们要求的更差的广告，且什么都不告知。如果从对话中无法判断标记属于谁，就询问；不要悄悄移除。如果在完整方向被接受之后，生成提示词还需要非实质性的制作细节，可添加一条独立的紧凑简报，它须源自研究且与该方向一致。它可以澄清实现，但不得发明或更改主体、场景或顺序、视觉风格、来源、版位形态、图内文字选择、信息钩子或约束。若其中任何一项仍未确定，回到方向关卡。所需图内文字必须逐字引用。不要改写广告主的文字，也不要把广告系列设置变成视觉指令。

**The second is a restricted good.** Do not put one in a generation prompt as
the SUBJECT of the picture: tobacco in any form including cigars and vapes,
alcohol, cannabis and other drugs, weapons, or gambling. That holds whoever is
asking and whatever they lawfully sell — a licensed dispensary, a glassware
brand and a shooting range are all still asking you to draw the restricted
thing. **Only when new generation is available for this ad at all**, which the
category block above decides first, take the brief and shoot around it: the
room, the people, the occasion, the craft, the packaging they supply. When the
ad is itself an alcohol or gambling ad, that block has already closed
generation, so there is no scene to offer and the advertiser's own image is the
only route — never offer to draw around a subject for an ad that may not be
drawn for at all.

**第二类是受限商品。**不要在生成提示词中把以下事物作为画面主体：任何形式的烟草（包括雪茄与电子烟）、酒类、大麻及其他毒品、武器，或博彩。无论谁在问、无论他们合法售卖什么，这条都成立——持牌药房、玻璃器皿品牌和射击场都仍然是在请你画那个受限物。**仅当此广告本来就可以使用新图像生成时**（由上方类别条款先行决定），才接下需求并绕开拍摄：房间、人物、场合、手艺、他们提供的包装。当广告本身就是酒类或博彩广告时，类别条款已关闭生成，没有场景可提供，广告主自有图片是唯一路线——绝不为一个根本不许画图的广告提议"绕开主体来画"。

**The advertiser's own seed is the exception**, on the terms the category block
sets out above: carried as-is across placements, or handed to Meta's own
enhancements. Offer that route in the same breath, so this redirects the request
rather than refusing it.

**广告主自己的素材是例外**，按上方类别条款给出的条件处理：跨版位原样携带，或交给 Meta 自有增强。要在同一句话里提供该路线，使这是对请求的改道而不是拒绝。

**Incidental is not the subject.** A glass of wine beside a plated dish, a pub
in the background of a high-street scene: the picture is not about the
restricted good and you must not strip it out. Judge what the image is OF, not
whether the item appears in frame.

**偶然出现不是主体。**一盘菜旁的一杯酒、街景背景中的一间酒馆：画面并非关于那个受限商品，你不得把它剥离。判断图像"拍的是什么"，而不是该物品是否出现在画面里。

Resolve new assets as uploadable sources, but do not upload them during this
stage; creation uses Ads hashes/IDs. Do not invent IDs, turn a thumbnail into
video, flatten carousel cards, or silently substitute formats or sources. A
missing source returns to the plan. `campaign-execution.md` reuses these current
schemas and fetches the remaining create schemas after media approval, before
rendering the final campaign review.

把新资产解析为可上传来源，但不要在本阶段上传；创建使用 Ads 哈希/ID。不要发明 ID、把缩略图变成视频、压平轮播卡片，或悄悄替换格式或来源。缺失来源则回到计划。`campaign-execution.md` 会复用这些当前 schema，并在媒体获批之后、渲染最终广告系列评审之前获取其余的创建 schema。

For generated images, use `meta-ads-cli creative generate-image` with the
accepted `feed`, `story`, or `reel` placement, verbatim advertiser request, and
`--output-dir workspace/your_files`. Repeat `--prompt` for separate constraints;
use `--source-image` only to adapt the selected workspace image. The wrapper
owns dimensions and rejects a wrong ratio.

对生成图片，使用 `meta-ads-cli creative generate-image`，带上已接受的 `feed`、`story` 或 `reel` 版位、原样的广告主请求，以及 `--output-dir workspace/your_files`。对相互独立的约束重复 `--prompt`；仅在适配所选工作区图片时使用 `--source-image`。封装层掌管尺寸并拒绝错误比例。

For image generation, leave someone else's logo, brand, character, public
figure, or recognizable likeness out, including from a remix. This does not ban
using a supplied licensed asset: the advertiser's own marks and authorized
third-party source assets follow `writes.md`; ask when rights are unclear rather
than silently stripping an advertiser's own brand. A compact production brief
may add only nonmaterial detail consistent with the accepted plan. It must not
change subject, scene/sequence, style, source, shape, text, hook, or constraint.
Quote in-image text verbatim.

图像生成时，把他人的徽标、品牌、角色、公众人物或可识别肖像排除在外，混编（remix）也不例外。这并不禁止使用提供的已获授权资产：广告主自有标记与获授权的第三方来源资产遵循 `writes.md`；权利不明确时先询问，而不是默默剥离广告主自己的品牌。紧凑的制作简报只能添加与已接受计划一致的非实质细节。它不得更改主体、场景/顺序、风格、来源、形态、文字、钩子或约束。图内文字必须逐字引用。

Retain generated `local_path`, dimensions, and `media_handle` when returned.
Prefer original media handles for other assets and do not convert/edit files
merely to retry. Intentional edits are new assets. Do not upload any candidate
before both creative approval and final campaign approval.

返回时保留生成结果的 `local_path`、尺寸与 `media_handle`。其他资产优先使用原始媒体句柄，不要仅为重试而转换/编辑文件。有意编辑即视为新资产。在创意批准与最终广告系列批准都完成之前，不上传任何候选。

## Placement compatibility / 版位兼容性

Before approval, verify that the selected Page or Instagram identity supports
the proposed placements and that every source meets their format and aspect-
ratio requirements. When one asset serves automatic placements, state once in
the creative plan that Meta may crop or resize it. Do not imply that one asset
provides placement-specific creative.

批准之前，核实所选 Page 或 Instagram 身份支持拟议版位，且每个来源都满足其格式与宽高比要求。当一个资产服务于自动版位时，在创意计划中说明一次"Meta 可能裁剪或缩放它"。不要暗示单一资产能提供版位专属创意。

Use exact placement-specific media only through a supported mapping. Otherwise
revise the placements or hierarchy and reprice when delivery changes. A failed
adaptation reopens the creative plan rather than silently substituting a source
or format.

仅通过受支持的映射使用精确的版位专属媒体。否则修订版位或层级，并在投放变化时重新报价。适配失败应重新打开创意计划，而不是悄悄替换来源或格式。

## Collect one prepared-media approval / 收集一次成品媒体批准

Preparation cannot change plan values. Before this gate, use
`meta-ads-cli describe-tool --name ads_create_creative --input-only` (or its
current result) and resolve `self_ai_disclosure` as directed by the schema.
Give the disclosure choices enough context to stand on their own: include a
clear question and a brief explanation of AI labeling in the final response
that carries the options token, written after the options call.
Use natural wording that fits the conversation.
Wait for the answer before presenting media for approval.

准备工作不能更改计划取值。在此关卡之前，使用 `meta-ads-cli describe-tool --name ads_create_creative --input-only`（或其当前结果），并按 schema 的指引处理 `self_ai_disclosure`。给披露选项足够的上下文使其能独立成立：在携带选项 token 的最终响应中（写在选项调用之后）包含一个清晰的问题和一段关于 AI 标注的简短说明。使用契合对话的自然措辞。在呈现媒体供批准之前先等待答案。

Show one representative image per concept, preferring 4:5 when Feed is present.
Show multiple images only when subject, composition, text, or message differs
materially; aspect ratio alone is not another concept. Attach video or list
carousel cards as applicable. The final response contains one brief declarative
sentence that the prepared media is ready for review, with the chosen AI
declaration and labeling caveat in plain language, then the media, then
exactly these options directly beneath it:

每个概念展示一张代表性图片，存在 Feed 时优先 4:5。仅当主体、构图、文字或信息有实质差异时才展示多张图片；仅宽高比不同不构成另一个概念。按需附上视频或列出轮播卡片。最终响应包含一句简短的陈述句，说明成品媒体已可评审，并以平实语言带上所选的 AI 声明与标注提醒，然后是媒体，然后是紧随其下、恰好如下的这些选项：

- `Approve this creative` (or `Approve these creatives` for multiple ads)
  `Approve this creative`（多支广告时为 `Approve these creatives`）
- `Revise the creative`
  `Revise the creative`

Then stop. Add no direction, evidence, copy, CTA, destination, placement note,
adaptation caveat, or other recap. A failure may explain its blocker.

然后停止。不添加方向、证据、文案、CTA、落地页、版位说明、适配提醒或其他复述。失败可以解释其阻塞点。

The affirmative option or an unambiguous typed approval of the shown,
unchanged media passes this single creative gate. It accepts the media paired
with the unchanged plan, not the campaign or any write. A revision repeats the
gate. Do not add copy approval or promise a rendered ad preview.

肯定选项、或对所展示且未更改媒体的明确文字批准，通过这一个创意关卡。它接受的是与未更改计划配对的媒体，而不是广告系列或任何写操作。修订则重复该关卡。不要附加文案批准，也不要承诺渲染后的广告预览。

## Carry approved media into execution / 将已批准媒体带入执行

Do not upload during this stage. Retain exactly one accepted source per planned
creative for `campaign-execution.md`. Upload approved new media only after final
campaign approval; use an existing account-owned image hash or ready video ID
directly. Carry a new image as:

本阶段不上传。为 `campaign-execution.md` 的每个计划创意恰好保留一个已接受的来源。仅在最终广告系列批准之后上传已批准的新媒体；已有的账户自有图片哈希或就绪视频 ID 可直接使用。新图片按以下方式携带：

- generated image: its returned `media_handle`, or its exact `local_path` as
  `file` when no handle was returned;
  生成图片：其返回的 `media_handle`；若未返回句柄，则以其精确 `local_path` 作为 `file`；
- other asset: known `media_handle`, including a matching prior upload-only
  result; if needed, read its native `output_path` and match the attachment;
  其他资产：已知的 `media_handle`，包括匹配的先前仅上传结果；必要时读取其原生 `output_path` 并匹配附件；
- otherwise a local `file`; download external URLs first.
  否则用本地 `file`；外部 URL 先下载。

Never use `snapshot_id`, pass a URL, compute a hash, switch source after
failure, or use another upload entrypoint. Creation takes only account-owned
image hashes, ready video IDs with thumbnails, or approved image sources that
`ads_creative_upload_media` uploads after final approval.

绝不使用 `snapshot_id`、传 URL、计算哈希、失败后更换来源，或使用其他上传入口。创建只接受账户自有图片哈希、带缩略图的就绪视频 ID，或在最终批准后由 `ads_creative_upload_media` 上传的已批准图片来源。
