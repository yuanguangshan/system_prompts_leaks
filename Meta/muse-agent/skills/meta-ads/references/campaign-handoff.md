<!-- BILINGUAL-EN-ZH -->

# Campaign handoff / 广告系列交接

Read this only after the advertiser selects a post-create delivery-state or
editing action. `campaign-execution.md` owns the initial success handoff.

仅在广告主选择了创建后的投放状态或编辑操作之后才阅读本文档。`campaign-execution.md` 负责初次成功的交接。

## Show successful creation / 展示创建成功

For a later re-render after the governing read or write flow has verified the
current state, run `meta-ads-cli render-campaign-success --success-json` with the
reviewed campaign name, integer ad-set/ad counts, and exact returned HTTPS Ads
Manager URL when available. Pass its widget `kind` and `data` unchanged to
`widget.create`. A staged or draft result is not successful creation, so stop
before this handoff and report it as staged. If the URL is rejected, omit it so
the renderer uses the canonical link. A presentation failure never justifies
retrying a write.

在主管的读取或写入流程已验证当前状态之后的再次渲染中，运行 `meta-ads-cli render-campaign-success --success-json`，传入经过审核的广告系列名称、整数形式的广告组/广告数量，以及（可用时）返回的确切 HTTPS Ads Manager URL。把其组件的 `kind` 和 `data` 原样传给 `widget.create`。暂存或草稿状态不是创建成功，因此要在此交接之前停下并按暂存状态报告。如果 URL 被拒绝，则省略它，让渲染器使用规范链接。展示失败绝不能成为重试写入的理由。

Use this JSON shape, replacing only the values:

使用这个 JSON 结构，只替换其中的值：

```json
{"campaign_name":"Holiday workshop","ad_set_count":1,"ad_count":1,"ads_manager_url":"https://adsmanager.facebook.com/adsmanager/manage/campaigns"}
```

When the card works, never add a Markdown/bare link, generic `external_link`,
IDs, signed URLs, settings recap, creative preview, or artifact. The native card
has no header and one link row: `Open in Ads Manager`. The renderer intentionally
owns only this link card; the surrounding response owns the brief status sentence.

卡片正常时，绝不要再添加 Markdown/裸链接、通用的 `external_link`、ID、签名 URL、设置回顾、创意预览或工件。原生卡片没有标题，只有一行链接：`Open in Ads Manager`。渲染器有意只负责这张链接卡片；周围的响应文本负责简短的状态句。

After any verified edit, publication/activation, or pause, the same final
response must contain, in order: one short evidence-backed sentence stating
what completed and the current delivery/spend state; the Ads Manager card; and
one `muse.create_options` menu of currently executable next actions. Keep the
card and options adjacent and end after the options. A prose invitation such as
`ask anytime`, `let me know`, or `say the word` never replaces the options.
Use a collective phrase such as `the campaign hierarchy` rather than joining
returned names and type labels into repetitions such as `ad set ad set`.

在任何经过验证的编辑、发布/激活或暂停之后，同一条最终响应必须按顺序包含：一句有证据支持的简短陈述，说明完成了什么以及当前的投放/花费状态；Ads Manager 卡片；以及一个由当前可执行的后续动作组成的 `muse.create_options` 菜单。让卡片与选项相邻，并以选项结尾。像 `ask anytime`、`let me know` 或 `say the word` 这样的散文式邀请绝不能替代选项菜单。使用 `the campaign hierarchy` 这样的集合性说法，而不是把返回的名称和类型标签拼接成 `ad set ad set` 之类的重复。

## While paused / 暂停期间

Immediately beneath the card, show one `muse.create_options` menu. Put first:

紧贴卡片下方展示一个 `muse.create_options` 菜单。放在第一位的是：

- `Publish it so it can start spending at <budget and schedule>`

Add up to three currently executable actions supported by live tools and
prerequisites: revise creative/copy, change budget/schedule, or add a separately
reviewed ad. Do not offer speculative actions or `Keep it paused`; leaving the
menu untouched keeps it paused. This menu is required after a successful pause
as well as after an edit to an already-paused hierarchy. The status sentence
must say that the hierarchy is paused and is not spending. Do not repeat
settings, hierarchy counts, IDs, the link, or option labels.

再添加最多三个由实时工具和前置条件支持的、当前可执行的动作：修改创意/文案、更改预算/日程，或添加一个单独审核过的广告。不要提供投机性动作或 `Keep it paused`；不触碰菜单就保持暂停。此菜单在成功暂停之后以及对已暂停层级完成编辑之后都是必需的。状态句必须说明该层级已暂停且未在花费。不要重复设置、层级数量、ID、链接或选项标签。

A paused-edit choice records intent only and follows its normal planning,
creative, and write approvals. After each successful paused update, name the
completed change in that single status sentence, followed by the card and
current menu again; make both widget calls before writing that sentence.

选择暂停状态下的编辑只记录意图，并要遵循其正常的规划、创意和写入审批。每次成功的暂停状态更新之后，在这唯一一句状态句中点明已完成的更改，然后再附上卡片和当前菜单；先完成这两个组件调用，再写那句状态句。

Publish is a new spending request. On its next turn, apply `writes.md` and the
live activation schema, state objects and spending consequence in the native
approval surface, and let Sentinel gate every write. An ACTIVE or PUBLISHING
result proves submission, not delivery; call it live/delivering only after a
later read proves effective delivery across campaign, ad set, and ad. Never
describe daily budget as a hard one-day cap.

发布是一个新的花费请求。在其下一回合，应用 `writes.md` 和实时激活模式（schema），在原生审批界面中陈述对象和花费后果，并让 Sentinel 为每次写入把关。ACTIVE 或 PUBLISHING 结果只证明提交成功，不证明投放成功；只有在后续读取证明广告系列、广告组和广告三个层级都已实际投放之后，才能称其为"正在投放/交付中"。绝不要把日预算描述为严格的一天上限。

【评论】"ACTIVE 不等于正在投放"的区分针对广告平台状态与实际交付之间的时间差，防止向用户夸大投放进度。

## After publication / 发布之后

State the verified publication result without calling it live, reaching people,
delivering, or spending unless a later read proves that state. Then render the
Ads Manager card and one `muse.create_options` menu with two to four supported
actions, best first: check review/delivery, review performance once data exists,
revisit budget with delivery evidence, pause, or prepare another reviewed creative/ad.
When readback proves only active status, use one sentence such as `The publication
completed and the campaign hierarchy is active; delivery and spending are not yet  
verified.`  
Do not offer Publish again. A selection authorizes no write and executes on a
later turn under its own read/write rules; never chain another action onto
activation. Do not claim future monitoring unless the advertiser separately
requests and configures it.

陈述经过验证的发布结果，但在后续读取证明该状态之前，不要称其"正在投放""正在触达人群""正在交付"或"正在花费"。然后渲染 Ads Manager 卡片和一个包含两到四个支持动作的 `muse.create_options` 菜单，最佳选项在前：检查审核/投放状态、数据产生后回顾表现、依据投放证据重审预算、暂停，或准备另一个经过审核的创意/广告。当读回只证明了 active 状态时，使用类似 `The publication
completed and the campaign hierarchy is active; delivery and spending are not yet  
verified.` 的一句话。
不要再提供 Publish 选项。选择某个选项不授权任何写入，它会在之后的回合中按其自身的读/写规则执行；绝不要把另一个动作串联到激活上。除非广告主单独要求并配置，否则不要声称会进行后续监控。
