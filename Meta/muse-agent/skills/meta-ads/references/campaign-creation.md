<!-- BILINGUAL-EN-ZH -->
# Meta Ads — complete campaign creation / Meta Ads — 完整广告系列创建

Use this controller for a new campaign hierarchy. Adding one child to an
existing hierarchy is a standalone write under `references/writes.md`.

本控制器用于全新广告系列层级结构。向既有层级添加一个子对象属于独立写入操作，见 `references/writes.md`。

## Route only the current stage / 仅路由当前阶段

Full research is the default. Before capability discovery or identity
resolution, ensure the current request explains what is being advertised or
tested well enough to proceed without inventing how it works. Otherwise ask only
the necessary product- or test-scope questions in one natural turn. Minimize
intake only after this gate passes. `campaign-planning.md` owns identity,
advertiser-owned blockers, and evidence-backed outcome, optimization, target, or
spend choices. Use guided mode only when explicitly requested; never ask which
mode. Retain valid decisions across mode changes.

完整研究是默认模式。在能力发现或身份解析之前，先确认当前请求对所广告或所测试内容的说明已足够充分，可以在不臆测其运作方式的前提下继续；否则仅在一个自然轮次中提出必要的产品或测试范围问题。只有通过这道门槛之后才开始精简信息采集。`campaign-planning.md` 负责身份、广告主自有阻塞项，以及有证据支撑的结果、优化、定向或花费选择。仅在用户明确要求时使用引导模式；绝不询问使用哪种模式。有效决策在模式切换后予以保留。

【评论】此段设置了一个"先理解再采集"的门槛，防止系统在信息不足时虚构产品细节，属于典型的防幻觉设计。

| Current need | Read now | Stage complete when |
|---|---|---|
| Full-research product brief, identity, research, delivery plan, or plan approval | `campaign-planning.md`; load `campaign-budget.md` after non-amount pricing inputs and advertiser constraints stabilize | All delivery decisions were presented together and accepted. |
| Explicit step-by-step planning or its separate budget steer | `campaign-guided.md`; load `campaign-budget.md` only when it directs | Its current steer was answered; budget is accepted before creative. |
| Creative plan, source preparation, or media approval | `campaign-creative.md` | The complete plan authorized its source action and every prepared ad passed the media gate. |
| Final review, paused creation, initial Ads Manager handoff, or partial recovery | `campaign-execution.md` | Every reviewed object exists, the campaign is verified paused, and the handoff is shown. |
| A selected post-create publish or editing action | `campaign-handoff.md` | The requested delivery-state or follow-up action is complete, or the campaign remains paused. |
| The account rejected writes as not available, or the advertiser will build it in Ads Manager | `campaign-manual-setup.md`, in place of `campaign-execution.md` and `campaign-handoff.md` | The accepted plan was handed over as a setup guide. |

| 当前需求 | 现在读取 | 阶段完成条件 |
|---|---|---|
| 完整研究的产品简介、身份、调研、投放方案或方案批准 | `campaign-planning.md`；待非金额定价输入与广告主约束稳定后加载 `campaign-budget.md` | 所有投放决策已一并呈现并获接受。 |
| 明确的逐步规划或其单独的预算引导 | `campaign-guided.md`；仅在其指示时加载 `campaign-budget.md` | 当前引导已获回应；预算在创意之前获接受。 |
| 创意方案、素材准备或媒体批准 | `campaign-creative.md` | 完整方案已授权其素材准备操作，且每个准备好的广告均通过媒体门禁。 |
| 最终审查、暂停创建、初次移交 Ads Manager 或部分恢复 | `campaign-execution.md` | 每个已审查对象均存在，广告系列已确认处于暂停状态，且已展示移交结果。 |
| 创建后选定的发布或编辑操作 | `campaign-handoff.md` | 所请求的投放状态或后续操作已完成，或广告系列保持暂停。 |
| 账户以不可用为由拒绝写入，或广告主将在 Ads Manager 中自行构建 | `campaign-manual-setup.md`（替代 `campaign-execution.md` 与 `campaign-handoff.md`）| 已接受的方案作为配置指南完成移交。 |

Load only the first incomplete stage. A strategy-only request stops after its
plan is accepted unless the advertiser asks to continue.

只加载第一个未完成的阶段。仅要策略的请求在其方案获接受后即停止，除非广告主要求继续。

## Decision and interaction contract / 决策与交互契约

Keep one `next_open_decision`. Full research may stop only for a product brief,
identity, binding constraint, advertiser-owned blocker, whole-plan approval,
creative source, media approval, or final-create approval. Guided mode resolves
one consequential setting at a time.

始终只保留一个 `next_open_decision`。完整研究仅可因以下事项暂停：产品简介、身份、约束性条件、广告主自有阻塞项、整体方案批准、创意素材来源、媒体批准或最终创建批准。引导模式一次只解决一个关键设置。

Use `muse.create_options` for every bounded choice. Call it before writing any
advertiser-facing text (`SKILL.md` rule 19); text written before the call is
hidden commentary. After it returns, write the final response: all context and
the question, then its returned `embed_token` alone on the final line, and stop.
A tap submits only its `selectedText`; an unambiguous typed answer to the same
unchanged choice is equivalent. Neither answers another question. Ask related
free-form product facts together; never replace bounded approval with `say the
word`.

每个有界选择都要使用 `muse.create_options`。在撰写任何面向广告主的文本之前先调用它（`SKILL.md` 规则 19）；调用之前写出的文本属于隐藏评论。调用返回后，撰写最终回复：先给出全部上下文与问题，再在其返回的 `embed_token` 单独置于最后一行，然后停止。点击只提交其 `selectedText`；针对同一未变更选择的明确文字输入答案与之等效。两者均不能回答另一个问题。相关的自由格式产品事实应一并询问；绝不用"说个词"式的提问替代有界批准。

Render options only after every selected read, background command, browser task,
and todo for that decision is terminal. Once the token is sent, leave no pending
work that can resume before the advertiser answers. Never invent, abbreviate, or
print an `<embed_token placeholder>`; if `muse.create_options` fails, no options
were shown.

只有在该决策相关的所有读取、后台命令、浏览器任务和待办都到达终态之后，才渲染选项。令牌发出后，不得留下任何会在广告主回答前恢复执行的挂起工作。绝不发明、缩写或打印 `<embed_token placeholder>`；若 `muse.create_options` 失败，则未展示任何选项。

Complete entity resolution and every other tool first. Call
`muse.create_options` alone as the final tool call, never in parallel. After it
succeeds, compose the response with its exact token and call nothing else. A
polled background result is not terminal for this purpose; wait for its automatic
completion notification before creating the widget.

先完成实体解析和所有其他工具调用。`muse.create_options` 必须单独作为最后一次工具调用，绝不并行。成功后，使用其返回的精确令牌撰写回复，且不再调用任何其他工具。轮询到的后台结果不算终态；需等待其自动完成通知后方可创建该组件。

These approvals remain distinct:

以下各类批准彼此独立：

- identity does not approve research or budget;
  身份确认不批准调研或预算；
- strategy approval accepts only the shown delivery plan;
  策略批准只接受所展示的投放方案；
- a creative-axis choice does not approve the complete creative plan;
  创意维度选择不批准完整创意方案；
- a source action authorizes only that preparation;
  素材来源操作只授权该次准备；
- media approval accepts only the shown media paired with the unchanged plan;
  媒体批准只接受与未变更方案配对展示的媒体；
- final review authorizes only the unchanged paused hierarchy; and
  最终审查只授权未变更的暂停层级；以及
- publication is a later spending decision.
  发布是之后的又一项花费决策。

【评论】将各类批准细分为互不可替代的独立确认，可防止用户的单次点击被扩大解释为对整个流程的授权。

## Capability and research ordering / 能力与研究顺序

Follow the discovery contract in `SKILL.md`: names once, only current-stage
descriptors, and `--agent-output` on normal calls. Do not fetch later-stage
create descriptors during research.

遵循 `SKILL.md` 中的发现契约：名称只取一次、仅取当前阶段的描述符、普通调用附带 `--agent-output`。研究期间不得获取后期创建阶段的描述符。

For full research, establish the product brief before discovery. Then resolve
identity sequentially: fetch the account descriptor, run the exact protected
account command required by `SKILL.md`, select the account, fetch the Page
descriptor, and read `ads_get_ad_account_pages` for that account. Later reads
may proceed only after this scope is known. Targeting resolution follows
objective and compliance; budget pricing follows the final audience,
geography, placements, and hierarchy.

完整研究模式下，先确立产品简介再进行发现。随后按顺序解析身份：获取账户描述符、运行 `SKILL.md` 要求的受保护账户命令、选定账户、获取 Page 描述符，并读取该账户的 `ads_get_ad_account_pages`。只有在明确该范围之后才能进行后续读取。定向解析跟随目标与合规要求；预算定价跟随最终受众、地域、版位和层级结构。

Ads Manager creation requires:

在 Ads Manager 中创建需要：

- `ads_get_ad_accounts` and `ads_get_ad_account_pages`;
  `ads_get_ad_accounts` 与 `ads_get_ad_account_pages`；
- `ads_targeting_search` for a creation-bound interest, place, or language that
  is not already canonical;
  对尚非规范条目的、与创建相关的兴趣、地点或语言使用 `ads_targeting_search`；
- `ads_create_campaign`, `ads_create_ad_set`, `ads_create_creative`, and
  `ads_create_ad`, with every selected input schema current before final review;
  a creative schema read during the current creative stage may be reused;
  `ads_create_campaign`、`ads_create_ad_set`、`ads_create_creative` 与 `ads_create_ad`，且每个选定的输入 schema 在最终审查前均为最新；当前创意阶段读取过的创意 schema 可复用；
- every identity, destination, format, and source required by those schemas;  
  and
  上述 schema 要求的每一项身份、目标端、格式和素材来源；以及
- `ads_creative_upload_media` for each new accepted asset after final approval.
  Existing Ads references need no upload.
  最终批准后，对每个新接受的素材使用 `ads_creative_upload_media`。既有 Ads 参考无需上传。

If a required creation capability is absent, create no partial hierarchy and
do not render final-create review or its approval options. Keep the flow at plan
or creative review, explain that execution is unavailable, and offer only a
supported revision.
When an accepted placement-specific static-image plan is unavailable, explain
the supported single-image alternative without exposing internal field names
and ask whether to revise. Never silently downgrade the accepted plan.
Capability does not authorize generation or upload; `campaign-creative.md`
owns those gates.

若缺少所需的创建能力，不得创建不完整的层级结构，也不得渲染最终创建审查或其批准选项。流程应停留在方案或创意审查环节，说明执行不可用，并仅提供受支持的修改选项。
当已接受的版位专属静态图片方案不可用时，在不暴露内部字段名的前提下说明受支持的单图替代方案，并询问是否修改。绝不悄悄降级已接受的方案。能力存在并不等于授权生成或上传；这些门禁由 `campaign-creative.md` 管理。

## Private strategy ledger / 私有策略台账

Keep the ledger only in conversation state—never `MEMORY.md`, a file, database
record, persisted strategy object, or runtime API. Track independently:

台账只保存在会话状态中——绝不写入 `MEMORY.md`、文件、数据库记录、持久化策略对象或运行时 API。独立追踪以下内容：

- account, Page, and Instagram identity;
  账户、Page 与 Instagram 身份；
- goal, objective, optimization, destination, and tracking;
  目标、objective、优化方式、目标端与追踪；
- compliance, geography, audience, placements, and hierarchy;
  合规、地域、受众、版位与层级结构；
- budget amount and cadence, derivation basis, cost source/confidence,
  projected volume, schedule, planning mode, and strategy approval;
  预算金额与节奏、推导依据、成本来源/置信度、预估量、排期、规划模式与策略批准；
- creative plan, source, prepared media, and media approval; and
  创意方案、素材来源、准备好的媒体与媒体批准；以及
- final approval, returned Ads IDs, handoff, and delivery state.
  最终批准、返回的 Ads ID、移交与投放状态。

Classify each decision as `settled` (value, basis, provenance), `assumed` (safe
default and reason), or `open` (exact decision and who or what can resolve it).
Settle only fields explicitly supplied by the advertiser or supported by a
successful result. A supported skill-owned default remains `assumed` and must
be shown for approval; it is not evidence. An answer to one question leaves
omitted sibling fields open, and any execution-critical field with neither a
settled value nor a safe executable default blocks strategy or final approval.
Failed research is unavailable evidence, not a negative advertiser fact.

将每项决策分类为 `settled`（值、依据、来源）、`assumed`（安全默认值及理由）或 `open`（确切决策及由谁或什么来解决）。只有广告主明确提供或由成功结果支撑的字段才可定为 settled。技能自带的安全默认值仍属 `assumed`，必须展示以供批准；它不构成证据。对一个问题的回答不会使被省略的同级字段变为已定；任何既无 settled 值也无安全可执行默认值的关键执行字段都会阻止策略或最终批准。调研失败属于不可用证据，而非关于广告主的负面事实。

【评论】把决策状态划分为 settled/assumed/open 三级，并明确"默认值不等于证据"，是防止系统将推测当作用户确认的约束设计。

Invalidate only dependants:

仅使依赖项失效：

```text
identity/destination
  -> tracking + objective
  -> compliance
  -> audience + structure + placements
  -> budget
  -> strategy approval
  -> creative plan
  -> prepared media
  -> final approval
```

Retain unaffected siblings and upstream work. Material execution drift returns
to the earliest invalidated decision; reprice only after new inputs stabilize.
A DRAFT is staged, not created. Sentinel remains the native write gate after
either form of conversational acceptance; no widget, artifact, typed reply, or
conversation state replaces it.

保留未受影响的同级项与上游工作。执行出现实质性偏差时回退到最早失效的决策；只有在新输入稳定后才重新定价。DRAFT 只是暂存，并非创建。无论经过哪种形式的对话确认，Sentinel 仍是原生写入门禁；任何组件、工件、文字回复或会话状态都不能取代它。

【评论】即便用户已在对话中确认，系统仍要求经过原生写入门禁（Sentinel）才实际执行，属于双重确认机制。
