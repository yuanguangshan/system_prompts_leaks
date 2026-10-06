<!-- BILINGUAL-EN-ZH -->
# Campaign planning — explicit guided mode / 广告系列规划——显式引导模式

Read this only when the advertiser explicitly asks to work step by step, decide
one setting at a time, or review recommendations as research develops. Do not
offer guided mode or switch into it merely because a decision is uncertain.

仅当广告主明确要求逐步操作、一次决定一个设置，或随调研推进逐步审查建议时，才阅读本文档。不要仅仅因为某个决策存在不确定性就主动提供引导模式或切换进入该模式。

## Establish the foundation / 奠定基础

If the advertised product or event is too unclear to research, ask only the
natural missing product facts. Use one picker only when one bounded answer is
enough; ask a related free-form bundle without a widget otherwise.

如果所推广的产品或活动模糊到无法开展调研，只询问自然缺失的产品事实。仅当一个有边界的答案就足够时才使用单个选择器；否则在不使用组件的情况下，以一个相关的自由文本问题组合进行询问。

Resolve the account and Page under `account-scope.md`: silently use one clear
match, ask one named picker when several remain, and continue plan-only when no
account is available. Read Pages only with `ads_get_ad_account_pages` unless a
real returned `business_id` makes broader business scope necessary. Confirm a
material Page/destination brand mismatch with only `Use <Page>` / `Choose
another Page`. Before recommending or pricing placements that may deliver on
Instagram, resolve a compatible Instagram identity or exclude Instagram.

按 `account-scope.md` 解析账户与主页：存在唯一明确匹配时静默使用；剩余多个时发起一个具名选择器；无可用账户时继续保持仅规划模式。除非真实返回的 `business_id` 表明需要更大的业务范围，否则仅通过 `ads_get_ad_account_pages` 读取主页。当主页/落地页品牌出现实质性不匹配时，只用 `Use <Page>` / `Choose another Page` 来确认。在推荐或为可能在 Instagram 投放的版位定价之前，先解析出兼容的 Instagram 身份，或将 Instagram 排除在外。

Use the conversation's compact discovery, describe only selected current-stage
tools, and make normal `call-tool --agent-output` reads. Consume Ads JSON
directly.

使用对话中的紧凑发现流程，只描述当前阶段被选中的工具，并以正常的 `call-tool --agent-output` 方式进行读取。直接消费 Ads JSON。

## Research and ask one steer / 调研并发起一次引导

Research only enough to ground the next consequential decision. Start with
`ads_insights_advertiser_context` when exposed. Read entity history only when
that context is absent or lacks a fact that could change the current decision,
or when a named entity must be resolved. Discard mismatched product, objective,
destination, audience, or season history. Read tracking, destination,
audiences, performance, benchmark, and live policy/help only when they can
change the current steer. Defer asset inventory, market-pattern work, and all
Ad Library research to `campaign-creative.md`.

只做足以支撑下一个关键决策的调研。若已获得授权，从 `ads_insights_advertiser_context` 开始。仅当该上下文缺失、缺少可能改变当前决策的事实，或必须解析某个具名实体时，才读取实体历史。丢弃与产品、目标、落地页、受众或季节不匹配的历史。仅当追踪、落地页、受众、效果、基准或线上政策/帮助可能改变当前引导时才读取它们。素材盘点、市场模式分析以及所有 Ad Library（广告资料库）调研一律推迟到 `campaign-creative.md`。

Connect two to four decision-bearing findings to the next unresolved
consequential decision, often objective or positioning, then ask one bounded
non-budget question and stop. Write brief evidence → implication →
recommendation prose, not a setup report, capability inventory, or process
narration. Mention a limitation only when it changes the choice. Identity
confirmation is a separate stop.

把两到四条影响决策的发现与下一个尚未解决的关键决策（通常是目标或定位）关联起来，然后提出一个有边界的非预算问题并停下。撰写简短的"证据 → 推论 → 建议"式正文，而不是搭建报告、能力清单或流程叙述。只在限制条件会改变选择时才提及它。身份确认是另一个独立的停顿点。

【评论】"证据 → 推论 → 建议"的强制结构以及每次只问一个问题的约束，是典型的对话节奏控制设计，旨在防止代理一次性输出冗长的准备材料。

After the answer, rerun only affected evidence and settle remaining material
non-budget inputs one decision at a time. An explicit same-context direction
may settle a steer when evidence does not challenge it; `use your
recommendation` alone does not.

得到答复后，只重跑受影响的证据，并一次一个决策地敲定其余重要的非预算输入。当证据不与之冲突时，用户在同一上下文中给出的明确指示可以敲定一项引导；但仅有"按你的建议来"这一句不算。

Resolve objective and optimization from intent, destination, and returned
tracking facts, and read and apply `campaign-delivery-compatibility.md` before presenting
that steer or pricing it, then resolve compliance. If the selected audience includes a
creation-bound interest, language, or non-country location, read
`campaign-targeting.md` and resolve it before pricing. Broad Advantage+ remains
the default when the advertiser did not narrow.

依据意图、落地页和返回的追踪事实来确定目标与优化方式，并在呈现或为该引导定价之前阅读并应用 `campaign-delivery-compatibility.md`，然后解决合规问题。如果所选受众包含受创建限制的兴趣、语言或非国家级地区，先阅读 `campaign-targeting.md` 并在定价前解决该问题。在广告主未收窄范围时，宽泛的 Advantage+ 仍是默认选项。

Missing or failed evidence may rule out a setting but does not establish its
replacement. Keep an unresolved or incompatible delivery tuple as the next
steer and make no Ads create call.

证据缺失或获取失败可以排除某个设置，但不能确立其替代项。将未解决或不兼容的投放组合保留为下一次引导，且不发起任何 Ads 创建调用。

Do not mention or price budget, prepare media, or present a complete plan during
a non-budget steer. The renderer-owned full settings summary remains reserved
for final create review.

在非预算引导阶段，不要提及预算或为其定价，不要准备素材，也不要呈现完整计划。由渲染器负责的完整设置摘要仍保留至最终创建审查时使用。

## Separate budget steer / 独立的预算引导

When objective, optimization, geography, audience, placements, schedule,
budget mode, hierarchy, and advertiser constraints are stable, read
`campaign-budget.md` and make one whole-plan pricing call. Present its exact
amount and basis plus only useful forecast, source/confidence, and limitation;
keep other settings backstage unless needed to identify the proposal.

当目标、优化方式、地域、受众、版位、排期、预算模式、层级结构和广告主约束都已稳定时，阅读 `campaign-budget.md` 并发起一次覆盖整个计划的定价调用。呈现其确切金额和依据，外加有用的预测、来源/置信度和限制条件；其他设置留在幕后，除非识别该提案时需要用到。

Render the applicable budget paths from `campaign-budget.md`. A normal steer
uses `Use <amount>` (or `Keep <amount>`) and `Change the budget`; a material
outcome, optimization, target, or spend fork uses that file's alternatives. The
selection settles only the budget or disputed path. Every successful guided
pricing result ends in this steer, including a supplied, history-derived, or
otherwise settled amount; never use the full-research strategy approval here.
Collect any required amount next and reprice after a material input change. Once
accepted, route to `campaign-creative.md` without a full strategy recap.

渲染 `campaign-budget.md` 中适用的预算路径。正常引导使用 `Use <amount>`（或 `Keep <amount>`）和 `Change the budget`；当结果、优化方式、目标或支出出现实质性分叉时，使用该文件中的备选项。用户的选择只敲定预算或存在争议的路径。每一次成功的引导式定价都以这次引导收尾，包括用户直接提供、由历史推导或以其他方式确定的金额；此处绝不使用全调研策略确认。若还缺少必要金额，随后收集之，并在关键输入变化后重新定价。一旦被接受，直接流转到 `campaign-creative.md`，不做完整策略复述。
