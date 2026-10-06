<!-- BILINGUAL-EN-ZH -->
# Campaign review and execution / 广告系列审核与执行

Read this only after delivery strategy and creative are accepted, or while
recovering partial creation. Recheck coherence; material drift returns to the
earliest affected planning stage. Final review authorizes only the unchanged
paused hierarchy and never publication.

仅在交付策略与创意获得接受之后阅读本文，或在恢复部分已创建成果时阅读。重新检查一致性；出现实质性漂移时，回到受影响的最早规划阶段。最终审核只授权维持原样的暂停层级，绝不授权发布。

## Ground the executable request / 落实可执行请求

Before final review, obtain the live input schemas for
`ads_create_campaign`, `ads_create_ad_set`, `ads_create_creative`, and
`ads_create_ad` with one `describe-tool --input-only` command per selected tool.
Reuse a schema fetched during the current creative stage unless a failure or
capability update made it stale. Fetch the upload schema too when approved new media must be uploaded.
Use only names present in the conversation's current compact discovery result.

在最终审核之前，为 `ads_create_campaign`、`ads_create_ad_set`、`ads_create_creative` 与 `ads_create_ad` 获取实时输入 schema，每个所选工具执行一条 `describe-tool --input-only` 命令。若当前创意阶段已获取过 schema 且它未因失败或能力更新而过时，可复用之。当获批的新媒体需要上传时，也要获取上传 schema。只使用对话当前紧凑发现结果中出现的名称。

Build each intended argument object strictly from those schemas. Before final
review, read and apply the pre-write gate in
`campaign-delivery-compatibility.md` to the
exact hierarchy and arguments, including every conditional or mutually
exclusive rule in the live field descriptions rather than only each schema's
`required` list. Never guess a
field, enum, conditional requirement, destination URL, identity, or media
reference, and never use a write or a validation failure to discover the
contract. A field absent from a selected schema is unsupported for this plan.
Optional fields default to absent. A known value does not authorize every
optional field that could carry it; include only the reviewed setting's mapped
field or a schema-required dependency.
Do not carry a field from another Ads tool merely because it is common there.
An existing object's read response cannot prove that a create-schema field is
optional; if current reads and the live create contract conflict, stop at that
contract gap rather than approving or probing a write.
If a required value cannot be obtained from the advertiser or a successful
read, return to the earliest open decision before requesting final approval.
Copy field names and enum values exactly from the live schema; never translate a
human label into a guessed provider value. An identifier is not a URL. Do not
construct a URL pattern from an ID unless the live schema explicitly defines
that transformation, or the ad set's `destination_type` is `MESSENGER`,
`WHATSAPP`, or `INSTAGRAM_DIRECT`, where `campaign-creative.md` gives the link.

严格依据这些 schema 构建每个拟用的参数对象。在最终审核之前，阅读 `campaign-delivery-compatibility.md` 中的写前门控，并将其应用到确切的层级与参数上，包括实时字段描述中的每条条件规则或互斥规则，而不只是每个 schema 的 `required` 列表。绝不猜测字段、枚举、条件要求、目标 URL、身份或媒体引用，也绝不用一次写入或一次校验失败去试探契约。所选 schema 中不存在的字段，即为本计划不支持的字段。可选字段默认缺省。一个已知数值并不授权每一个可能承载它的可选字段；只纳入已审核设置所映射的字段或 schema 要求的依赖。
不要仅仅因为某字段在另一个 Ads 工具中常见就把它带过来。既有对象的读取响应无法证明某个 create schema 字段是可选的；若当前读取与实时 create 契约冲突，停在这个契约缺口处，而不是批准或试探一次写入。
若某个必需值既无法从广告主处获得、也无法通过一次成功读取获得，则在请求最终批准之前回到最早的未决决策。字段名与枚举值要逐字从实时 schema 复制；绝不把人类可读标签翻译成猜测的提供方取值。标识符不是 URL。除非实时 schema 明确定义了该变换，或广告组的 `destination_type` 为 `MESSENGER`、`WHATSAPP` 或 `INSTAGRAM_DIRECT`（链接由 `campaign-creative.md` 给出），否则不要从 ID 构造 URL 模式。

Only when that tool's schema exposes `advertiser_request`, use the advertiser's
complete current multi-turn wording required by `SKILL.md`; never add an
assistant-written reconstruction.

仅当该工具的 schema 暴露 `advertiser_request` 时，才使用 `SKILL.md` 所要求的广告主完整当前多轮原话；绝不添加助手代拟的转述。

Keep all campaign decisions and execution state in conversation context. Do
not call memory tools or persist campaign settings, IDs, media references,
attempts, or failures to `MEMORY.md` or another durable store.

把所有广告系列决策与执行状态保存在对话上下文中。不要调用记忆工具，也不要把广告系列设置、ID、媒体引用、尝试或失败持久化到 `MEMORY.md` 或其他持久存储。

The campaign, ad-set, and ad create tools enforce `PAUSED` server-side; a
creative has no delivery status. Follow each live schema and do not add a
`status` field when it is not exposed. Keep a private mapping from every
reviewed value to the exact argument that will create it, including:

广告系列、广告组与广告的创建工具在服务端强制 `PAUSED`；创意没有投放状态。遵循每个实时 schema，在未暴露 `status` 字段时不要添加它。为每个已审核的值保留一份到"将创建它的确切参数"的私有映射，包括：

- account, Page and optional Instagram identity;
  账户、主页与可选的 Instagram 身份；
- objective, optimization, destination, targeting, placements, and required
  special-ad-category declarations and countries, including
  `targeting_as_signal: 0` on every ad set of a housing, employment, or
  financial products and services campaign (why: the Special Ad Category
  section of `references/writes.md`);
  目标（objective）、优化方式、目的地、定向、版位，以及所需的特殊广告类别声明与国家/地区，包括在住房、就业或金融产品与服务类广告系列的每个广告组上设置 `targeting_as_signal: 0`（原因见 `references/writes.md` 的 Special Ad Category 一节）；
- hierarchy, budget, currency and schedule;
  层级、预算、币种与排期；
- creative format, source, copy, CTA and disclosure; and
  创意格式、素材来源、文案、CTA 与披露声明；以及
- all parent/reference dependencies that will come from successful results.
  所有将来自成功结果的父级/引用依赖。

On every budget- or bid-bearing write whose schema exposes `account_currency`,
pass the exact currency returned for the selected account. Never infer it from
geography or silently convert an amount. Write budget at exactly one hierarchy
level: when the campaign carries its budget, omit ad-set budget fields; when ad
sets carry budget, omit campaign budget fields.

在每次涉及预算或出价、且其 schema 暴露 `account_currency` 的写入中，传入为所选账户返回的确切币种。绝不从地理位置推断币种，也绝不静默换算金额。预算只写在恰好一个层级上：广告系列带预算时，省略广告组预算字段；广告组带预算时，省略广告系列预算字段。

`render-campaign-summary --summary-json` accepts exactly `campaign_name`,
`goal`, `optimization`, `destination`, `structure` (`ad_set_count` and
`ad_count`), `budget`, `schedule`, `audience_and_geography`,
`audience_description`, and `placements`. Schedule declares an
`after_publishing` or scheduled start and a `no_end` or scheduled end, with a
time only for a scheduled boundary. Placements declares exactly one mode:
`automatic`, `platform_restricted` with platforms, or `manual` with positions.
Its values must describe the intended create arguments; do not use the renderer
to introduce or omit a material setting. Creative, category, and paused state
were already reviewed elsewhere and do not appear as summary rows.

`render-campaign-summary --summary-json` 恰好接受 `campaign_name`、`goal`、`optimization`、`destination`、`structure`（`ad_set_count` 与 `ad_count`）、`budget`、`schedule`、`audience_and_geography`、`audience_description` 与 `placements`。schedule 声明 `after_publishing` 或定时开始，以及 `no_end` 或定时结束，只有定时边界才带时间。placements 恰好声明一种模式：`automatic`、带平台的 `platform_restricted`，或带具体位置的 `manual`。其取值必须描述拟用的创建参数；不要借渲染器引入或省略实质性设置。创意、类别与暂停状态已在别处审核过，不作为汇总行出现。

Use this JSON shape, replacing only the values:

使用这个 JSON 形状，仅替换其中的值：

```json
{"campaign_name":"Holiday workshop","goal":"Awareness","optimization":"Impressions","destination":"https://example.com/workshop","structure":{"ad_set_count":1,"ad_count":1},"budget":"$10/day at campaign level","schedule":{"start":"scheduled","start_time":"December 1, 2026","end":"scheduled","end_time":"December 23, 2026"},"audience_and_geography":"United States","audience_description":"Broad Advantage+ audience","placements":{"mode":"automatic"}}
```

## Render final review / 渲染最终审核

Call `meta-ads-cli render-campaign-summary` once with those exact settled
values, then pass its compact widget `kind` and `data` unchanged to  
`widget.create`.  
Create these exact options with `muse.create_options`:

用那些确切的已定值调用 `meta-ads-cli render-campaign-summary` 一次，然后把它紧凑部件的 `kind` 与 `data` 原样传给  
`widget.create`。  
用 `muse.create_options` 创建这两个确切的选项：

- `Yes, create paused campaign`
  `Yes, create paused campaign`（是的，创建暂停的广告系列）
- `No, make changes`
  `No, make changes`（不，需要修改）

In the final response, place the review token first, ask `Proceed with this
campaign? It will be created paused. After success I'll give you a direct Ads
Manager link.`, then place the options token. Add no duplicate summary or
second question. The exact affirmative option or an unambiguous typed
instruction to create the unchanged reviewed hierarchy accepts this decision;
do not treat agreement to another decision as final approval.

在最终答复中，先放审核令牌，再问 `Proceed with this campaign? It will be created paused. After success I'll give you a direct Ads Manager link.`，然后放选项令牌。不要添加重复的汇总或第二个问题。确切的肯定选项、或一条含义无歧义的"创建这个保持原样的已审核层级"的文字指示，才视为接受本决定；不要把对其他决定的同意当作最终批准。

## Create after approval / 批准后创建

After that affirmative acceptance, execute only the reviewed arguments:

在那次肯定接受之后，只执行已审核的参数：

1. Upload each approved new asset with `ads_creative_upload_media`; retain only
   its returned account-owned media reference. Existing Ads references need no
   upload. Complete and verify every required upload before the first Ads-object
   write; media preparation is a gate, not work to parallelize with campaign
   creation.
   用 `ads_creative_upload_media` 上传每个获批的新素材；只保留其返回的账户自有媒体引用。已有的 Ads 引用无需上传。在第一次 Ads 对象写入之前完成并核验所有必需的上传；媒体准备是一道门控，不是可与广告系列创建并行的工作。
2. Call `ads_create_campaign` once and retain its returned campaign ID.
   调用 `ads_create_campaign` 一次，并保留其返回的广告系列 ID。
3. Call `ads_create_ad_set` exactly once for each reviewed ad set with that
   campaign ID, and `ads_create_creative` exactly once for each reviewed
   creative with its approved identity/media. Run only dependency-independent
   calls in parallel.
   用该广告系列 ID 为每个已审核的广告组恰好调用一次 `ads_create_ad_set`，并用其获批的身份/媒体为每个已审核的创意恰好调用一次 `ads_create_creative`。只有相互无依赖的调用才并行执行。
4. Call `ads_create_ad` exactly once for each reviewed ad, only after its
   paired ad-set and creative IDs exist.
   为每个已审核的广告恰好调用一次 `ads_create_ad`，且仅在其配对的广告组与创意 ID 都已存在之后才调用。

Creation succeeds when every expected campaign, ad set and ad create result
returns its ID and `status` `PAUSED`, and every creative create returns its ID.
Those successful create results are the verification; do not read the hierarchy
back after a clean create. Never infer a child's safe state from a paused
ancestor: each object's own result must show `PAUSED`. A `DRAFT` result is
staged, not created, and is reported under `references/writes.md`. An
unexpectedly `ACTIVE` create result is an immediate stop: pause it when an
exposed update can, then reconcile and report without creating descendants.
When a result that reports success lacks its ID or status, make one fresh read
of that object with the live read tool, fetching its schema first; that read is
reconciliation, not a retry. A missing object or non-`PAUSED` status after that
read blocks the success card and publication options.

当每个预期的广告系列、广告组与广告的创建结果都返回其 ID 和 `status` `PAUSED`，且每个创意的创建都返回其 ID 时，创建即告成功。这些成功的创建结果本身就是验证；创建干净完成后不要再读回层级。绝不从处于暂停状态的祖先推断子对象的安全状态：每个对象自己的结果必须显示 `PAUSED`。`DRAFT` 结果是暂存而非已创建，按 `references/writes.md` 报告。意外出现 `ACTIVE` 的创建结果是立即停止信号：在暴露的更新能力可以做到时将其暂停，然后进行对账并报告，不再创建任何后代对象。当某个报告成功的结果缺少 ID 或状态时，用实时读取工具对该对象做一次全新读取（先获取其 schema）；这次读取是对账，不是重试。该读取之后若对象缺失或状态非 `PAUSED`，则阻止成功卡片与发布选项。

【评论】所有创建类写入都在服务端强制 `PAUSED`，再配合"用户明确批准后才发布"的流程，构成防止广告在未经同意的情况下开始花费的双保险。

Run every Ads operation as its own `exec` call. Never retry a create merely to
recover missing output. If an argument or material decision changes after
approval, stop, update only affected decisions, reprice when required, and
render a new final review.

每个 Ads 操作都用它自己的一次 `exec` 调用运行。绝不要为了找回缺失的输出而重试一次创建。若批准之后某参数或实质性决策发生变化，停止，只更新受影响的决策，必要时重新报价，并渲染一次新的最终审核。

`render-campaign-success --success-json` accepts exactly `campaign_name`,
`ad_set_count`, `ad_count`, and optional `ads_manager_url`. After verified
success, call it, pass its returned list widget unchanged to `widget.create`,
then show one `muse.create_options` menu.
Put `Publish it so it can start spending at <budget and schedule>` first and add
up to three currently executable paused edits supported by the discovered tools
and prerequisites. Do not offer `Keep it paused`; leaving the menu untouched
keeps it paused. In the final response, state in one short evidence-backed
sentence that creation completed and the hierarchy remains paused and is not
spending, then embed the Ads Manager card and the options consecutively. End
after the options. Do not repeat settings, hierarchy counts, IDs, the link, or
the option labels. Use a collective phrase such as `the campaign hierarchy`
rather than joining returned names and type labels. A prose invitation such as
`ask anytime`, `let me know`, or `say the word` never replaces the options.

`render-campaign-success --success-json` 恰好接受 `campaign_name`、`ad_set_count`、`ad_count` 与可选的 `ads_manager_url`。在验证成功之后调用它，把它返回的列表部件原样传给 `widget.create`，然后展示一个 `muse.create_options` 菜单。
把 `Publish it so it can start spending at <budget and schedule>` 放在第一位，再添加最多三个当前可执行、且有所发现的工具与前提条件支持的暂停状态下编辑项。不要提供 `Keep it paused` 选项；不碰菜单即保持暂停。在最终答复中，用一句简短、有证据支撑的话说明创建已完成、层级仍处于暂停且未在花费，然后连续嵌入 Ads Manager 卡片与选项。选项之后即结束。不要重复设置、层级计数、ID、链接或选项标签。使用诸如 `the campaign hierarchy` 的集合性说法，而不是把返回的名称与类型标签拼接起来。诸如 `ask anytime`、`let me know` 或 `say the word` 之类的文字性邀请绝不能替代选项。

## Failure and recovery / 失败与恢复

On failure, reconcile current Ads state before deciding whether another write
is safe. Report only objects proved to exist and their verified status, plus the
first incomplete stage. If local or generated media exists but no campaign,
ad set, creative, or ad exists, say `No Ads objects were created`, not `nothing
was created`.

失败时，先对账当前 Ads 状态，再判断再一次写入是否安全。只报告被证实存在的对象及其经过核验的状态，外加第一个未完成的阶段。若本地或生成的媒体存在，但没有任何广告系列、广告组、创意或广告存在，要说 `No Ads objects were created`，而不是 `nothing was created`。

Describe the returned failure without assigning an unproved provider, outage,
or infrastructure cause. Say when reconciliation occurred rather than claiming
each attempt was verified. Account-owned uploaded media is existing state even
when no delivery object was created.

描述返回的失败时，不要归因于未经证实的提供方、故障中断或基础设施原因。说明对账何时发生，而不要声称每次尝试都经过验证。账户自有且已上传的媒体属于既有状态，即使没有创建任何投放对象。

Never say `everything validated`, `the last step`, or equivalent unless every
schema, identity, asset, argument, approval, write, and verification required by
that statement actually completed. A validation, schema, unsupported-shape,
missing-input, policy, or eligibility rejection requires corrected grounded
input or a supported revision; do not offer an unchanged retry. A local
pre-dispatch validation failure may be corrected from its exact diagnostic and
reissued because no write occurred. For an explicitly transient write failure,
or an ambiguous failure reconciled to no object, make at most one additional
dispatch for that intended object in the current turn. Reconciliation is the
safety prerequisite, not another retry allowance. If that retry fails, stop;
a later explicit request or changed environment may begin a new attempt only
after current state is reconciled.

绝不说 `everything validated`、`the last step` 或同义表述，除非该陈述所需的每一项 schema、身份、素材、参数、批准、写入与验证都确实完成了。校验、schema、不支持形状、缺失输入、政策或资格类的拒绝，需要经纠正的、有依据的输入或一次受支持的修订；不要提供原样重试。本地分派前校验失败可以按其确切诊断纠正后重新发出，因为没有发生写入。对于明确瞬时的写入失败，或对账后确认没有产生对象的含糊失败，在当前回合内对该拟建对象最多再分派一次。对账是安全前提，不是额外的重试额度。若该次重试仍失败，停止；只有在对当前状态完成对账之后，后续的明确请求或环境变化才可以开启新一轮尝试。
