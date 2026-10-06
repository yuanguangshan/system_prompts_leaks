<!-- BILINGUAL-EN-ZH -->
# Campaign delivery compatibility / 广告系列投放兼容性

Use this gate while choosing delivery and again on the exact create arguments
before final review. It prevents a plausible setting in isolation from becoming
an invalid campaign when combined with the others.

在选择投放方式时使用这道闸门，并在最终评审前对确切的创建参数再次使用。它防止单独看似乎合理的设置，与其他设置组合后变成无效的广告系列。

Treat each ad set's business outcome, campaign objective, optimization goal,
destination, required signal or promoted object, and billing event as one
decision. Settle the tuple only when every leg agrees:

把每个广告组的业务结果、广告系列目标（objective）、优化目标（optimization goal）、目标去向（destination）、所需信号或推广对象（promoted object）以及计费事件当作一个整体决策。只有每一条腿都一致时才敲定这个元组：

- the objective supports the optimization goal;
  objective 支持该优化目标；
- the destination can produce the event being optimized;
  目标去向能够产生被优化的事件；
- every required Page, form, messaging identity, dataset/pixel, and event was
  returned for the selected account and belongs to the intended role. No tool
  lists an account's apps, so an app ID and store URL come from the advertiser
  or an existing ad set's promoted object; never invent either;
  所需的每个主页（Page）、表单、消息身份、数据集/像素（pixel）和事件都已为所选账户返回，且属于预期角色。没有工具会列出账户的应用，因此应用 ID 和商店 URL 必须来自广告主或既有广告组的推广对象；绝不凭空编造两者；
- the promoted-object shape contains exactly the fields that goal and
  destination require; and
  推广对象的结构恰好包含该目标与目标去向所要求的字段；并且
- the billing event and budget/bid level are supported by the same setup.
  计费事件与预算/出价层级由同一套设置支持。

The plan must describe the result it actually optimizes. A plan cannot promise
purchases while optimizing visits, call a website visit an on-ad lead, or call
Page Likes a Traffic result. An upstream event is a different delivery path;
present and settle it as such rather than relabelling the advertiser's outcome.

计划必须描述它实际优化的结果。计划不能一边优化访问一边承诺购买，不能把网站访问称作广告内线索（on-ad lead），也不能把主页点赞称作 Traffic 结果。上游事件是另一条投放路径；按其本来的样子呈现并敲定，而不是给广告主的结果改贴标签。

## Common recommendation families / 常用推荐族

Use these families for ordinary recommendations. Codes are internal planning
values; use advertiser-facing names in responses.

普通推荐使用这些推荐族。代码是内部规划值；在回复中使用面向广告主的名称。

| Intended result | Compatible delivery family |
|---|---|
| Awareness or reach | `OUTCOME_AWARENESS` with `REACH`, `IMPRESSIONS`, or another live-supported awareness goal |
| Website visits | `OUTCOME_TRAFFIC` with `LANDING_PAGE_VIEWS` or `LINK_CLICKS`, a verified website URL, and only tracking fields supported by that goal |
| Website purchases or conversion value | `OUTCOME_SALES` with `OFFSITE_CONVERSIONS` or `VALUE`, a website destination, and a returned eligible conversion source plus a supported event |
| Website lead conversions | `OUTCOME_LEADS` with `OFFSITE_CONVERSIONS`, a website destination, and a returned eligible conversion source plus the selected lead event |
| Instant-form leads | `OUTCOME_LEADS` with `LEAD_GENERATION` or `QUALITY_LEAD`, an on-ad destination, and the returned Page and form |
| Messaging conversations | An objective matching the stated outcome with `CONVERSATIONS`, a matching Messenger, WhatsApp, or Instagram Direct destination, and its returned identity; the creative uses the channel's message button and standard link per `campaign-creative.md`, never an advertiser-supplied URL |
| Page Likes | `OUTCOME_ENGAGEMENT` with `PAGE_LIKES`, the Page destination and returned Page; use a live-supported billing event, normally impressions |
| Post engagement | `OUTCOME_ENGAGEMENT` with `POST_ENGAGEMENT`, an on-post destination, and the returned Page/post or supported new-ad shape |
| Profile visits | `OUTCOME_TRAFFIC` with `VISIT_INSTAGRAM_PROFILE` or `PROFILE_VISIT` and the matching returned profile destination |
| Video views | `OUTCOME_AWARENESS` or `OUTCOME_ENGAGEMENT` with a live-supported video-view goal and destination |
| App installs or app events | `OUTCOME_APP_PROMOTION` with an app goal, app destination, and the advertiser-supplied app ID and store URL or event identity |
| Calls | A goal-matched objective with a live-supported call goal, phone destination, and returned Page/number inputs |

| 预期结果 | 兼容的投放族 |
|---|---|
| 认知度或触达 | `OUTCOME_AWARENESS` 配 `REACH`、`IMPRESSIONS` 或其他线上支持（live-supported）的认知类目标 |
| 网站访问 | `OUTCOME_TRAFFIC` 配 `LANDING_PAGE_VIEWS` 或 `LINK_CLICKS`、一个经过验证的网站 URL，以及仅限该目标支持的追踪字段 |
| 网站购买或转化价值 | `OUTCOME_SALES` 配 `OFFSITE_CONVERSIONS` 或 `VALUE`、一个网站目标去向，以及一个返回的合格转化来源加一个受支持的事件 |
| 网站线索转化 | `OUTCOME_LEADS` 配 `OFFSITE_CONVERSIONS`、一个网站目标去向，以及一个返回的合格转化来源加所选的线索事件 |
| 即时表单线索 | `OUTCOME_LEADS` 配 `LEAD_GENERATION` 或 `QUALITY_LEAD`、一个广告内目标去向，以及返回的主页和表单 |
| 消息会话 | 与所述结果匹配的 objective 配 `CONVERSATIONS`、匹配的 Messenger、WhatsApp 或 Instagram Direct 目标去向及其返回的身份；创意按 `campaign-creative.md` 使用该频道的消息按钮和标准链接，绝不用广告主提供的 URL |
| 主页点赞 | `OUTCOME_ENGAGEMENT` 配 `PAGE_LIKES`、主页目标去向和返回的主页；使用线上支持的计费事件，通常是展示（impressions） |
| 帖子互动 | `OUTCOME_ENGAGEMENT` 配 `POST_ENGAGEMENT`、帖子内目标去向，以及返回的主页/帖子或受支持的新广告结构 |
| 主页访问 | `OUTCOME_TRAFFIC` 配 `VISIT_INSTAGRAM_PROFILE` 或 `PROFILE_VISIT` 以及匹配的已返回主页目标去向 |
| 视频观看 | `OUTCOME_AWARENESS` 或 `OUTCOME_ENGAGEMENT` 配线上支持的视频观看目标和目标去向 |
| 应用安装或应用事件 | `OUTCOME_APP_PROMOTION` 配应用目标、应用目标去向，以及广告主提供的应用 ID 和商店 URL 或事件身份 |
| 来电 | 目标匹配的 objective 配线上支持的来电目标、电话目标去向，以及返回的主页/号码输入 |

These are recommendation families, not a substitute for the current tool
contract. An account-gated or unlisted combination is settled only when current
tool evidence proves the complete combination and its required inputs. A field
being present in a schema does not prove that every combination using it is
valid.

这些是推荐族，不能替代当前的工具契约。受账户限制或未列出的组合，只有当前的工具证据证明完整组合及其所需输入时才算敲定。某个字段出现在 schema 中，并不能证明使用它的每个组合都有效。

## Rules the tuple does not show / 元组显示不出来的规则

These combinations pass every individual field check and are still rejected:

以下组合能通过每一项单独的字段检查，但仍会被拒绝：

- **One goal per lowest-cost campaign budget.** When the campaign carries the
  budget with the lowest-cost bid strategy, every ad set in it, including one
  added later, uses the same optimization goal. Before adding an ad set, read
  the existing ad sets' goal; a different goal needs its own campaign.
  **每个最低成本预算的广告系列只能有一个优化目标。** 当广告系列携带采用最低成本出价策略的预算时，其中的每个广告组（包括后来添加的）都使用同一优化目标。在添加广告组之前，先读取既有广告组的目标；不同的目标需要自己的广告系列。
- **A lifetime budget needs an end.** When the ad set's own budget or its
  parent campaign's budget is a lifetime budget, the ad set needs an end time
  more than 24 hours after its start. Read the parent's budget before adding an
  ad set to an existing campaign. Without an end date, ask for one; never
  switch the advertiser's lifetime budget to a daily one yourself
  (`references/writes.md` treats budget cadence as their decision).
  **总预算（lifetime budget）需要结束时间。** 当广告组自己的预算或其父广告系列的预算是总预算时，广告组需要一个比开始时间晚 24 小时以上的结束时间。把广告组加进既有广告系列之前，先读取父级预算。没有结束日期时，向广告主索要；绝不要自作主张把广告主的总预算改成日预算（`references/writes.md` 把预算周期视为广告主的决定）。
- **An app ad targets the app's platform.** When the promoted object names an
  app, restrict targeting to that app's operating system: an App Store app is
  iOS and a Google Play app is Android. Leaving the operating system open is
  rejected as a mismatch. Take the value from the live schema or a targeting
  search, never a guessed string.
  **应用广告定向到该应用的平台。** 当推广对象指明某个应用时，把定向限制到该应用的操作系统：App Store 的应用是 iOS，Google Play 的应用是 Android。把操作系统留空会作为不匹配被拒绝。取值来自线上 schema 或定向搜索，绝不用猜测的字符串。
- **iOS 14+ attribution needs an iOS 14+ campaign.** Send the ad set's
  `campaign_attribution` (`SKAN` or `AEM`) only under a campaign created in
  this conversation with `is_skadnetwork_attribution: true`, or one the
  advertiser says is an iOS 14+ campaign. Reads cannot show that setting on an
  existing campaign, so otherwise omit it. Take field names from the live
  schema.
  **iOS 14+ 归因需要 iOS 14+ 广告系列。** 只有在本对话中创建的、带有 `is_skadnetwork_attribution: true` 的广告系列之下，或广告主确认是 iOS 14+ 广告系列之下，才发送广告组的 `campaign_attribution`（`SKAN` 或 `AEM`）。读取操作无法在既有广告系列上显示该设置，所以其他情况一律省略。字段名取自线上 schema。

## Resolve incompatibility before pricing / 定价之前先解决不兼容

When supplied values cross families, preserve each stated intent as a
constraint and show one bounded choice between executable paths. For example,
a request for website Traffic optimized for Page Likes becomes a choice between
website visits and Page Likes; neither setting silently overwrites the other.
Missing required conversion signals likewise becomes a choice between restoring
that measurement and accepting a clearly named upstream delivery path.

当提供的取值跨族时，把每项已陈述的意图保留为约束，并给出一个在可执行路径之间的有边界选择。例如，"网站 Traffic 优化主页点赞"的请求变成"网站访问"与"主页点赞"之间的选择；任何一个设置都不会悄悄覆盖另一个。缺少所需转化信号同样变成一个选择：要么恢复该测量，要么接受一条被明确点名的上游投放路径。

Until the tuple is compatible, keep it open. Do not price it, present a complete
strategy, plan creative against it, render final review, or call an Ads create
tool.

在元组变得兼容之前，保持其未定状态。不要为其定价、不要呈现完整策略、不要据此规划创意、不要渲染最终评审，也不要调用 Ads 创建工具。

## Recheck the exact create arguments / 复查确切的创建参数

After fetching the selected live create schemas, apply their conditional and
mutual-exclusion descriptions—not only their `required` arrays—to the exact
arguments. Confirm the campaign objective and budget ownership against every ad
set's optimization goal, destination, promoted object, billing event and bid
fields; then confirm targeting/placements, one creative family with all of its
identity/media/link requirements, and the intended ad-set/creative references.

在获取所选的线上创建 schema 之后，把它们的条件描述和互斥描述 —— 而不仅是 `required` 数组 —— 应用到确切参数上。对照每个广告组的优化目标、目标去向、推广对象、计费事件和出价字段，确认广告系列目标与预算归属；然后确认定向/版位、一个创意族及其全部身份/媒体/链接要求，以及预期的广告组/创意引用。

The live descriptors win if this reference and the current contract differ.
Return to the earliest affected planning decision and obtain a revised approval;
do not rely on create-time defaulting or auto-correction, and do not use a
failed create as compatibility discovery.

若本参考文档与当前契约不一致，以线上描述符为准。回到最早受影响的规划决策并取得修订后的批准；不要依赖创建时的默认值或自动纠正，也不要把一次失败的创建当作兼容性探测手段。
