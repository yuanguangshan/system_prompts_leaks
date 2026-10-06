<!-- BILINGUAL-EN-ZH -->
# Cost Optimization - Cutting Spend per Completed Task / 成本优化——降低每个已完成任务的开销

> **If you arrived via `/claude-api cost-optimize`:** this is the right file. Execute the steps below in order rather than summarizing the guide back to the user - presenting the profile, the ranked plan, and the findings IS part of the execution. Start with Step 0 (establish scope, quality bar, and baseline), and finish with Step 4's two deliverables: the cost profile and the changes.

> **如果你是通过 `/claude-api cost-optimize` 进入本文件的：**来对地方了。请按顺序执行以下步骤，而不是把本指南总结一遍复述给用户——呈现画像、排序列表与结论本身就是执行的一部分。从步骤 0（确定范围、质量标准与基线）开始，以步骤 4 的两项交付物（成本画像与变更）收尾。

API spend is optimized in units of **cost per completed task, not cost per token**. A model with a higher sticker price can be the cheaper option if it finishes the job in fewer turns, and a cheaper model that fails still bills its tokens, then the retry, then whatever the failure costs downstream. Every judgment below reads cost and quality together.

API 开销以**每个已完成任务的成本、而非每 token 的成本**为单位进行优化。标价更高的模型如果能在更少的轮次内完成工作，反而可能是更便宜的选择；而一个失败的廉价模型照样要为它的 token 计费，随后还有重试的费用，以及失败在下游造成的一切代价。下文的每项判断都将成本与质量放在一起衡量。

【评论】以"每完成任务成本"而非"每 token 成本"为优化单位，是本指南的核心立场：它把重试与失败的下游代价一并计入，避免只看 token 单价造成误判。

The levers divide into two kinds, and the order of the steps is load-bearing:

这些杠杆分为两类，而步骤的先后顺序是有实际效力的：

- **Free wins** - prompt caching, input-token hygiene (including a prompt audit), loop hygiene, output-token hygiene, batch processing - lower what you pay without lowering output quality. They go first, and caching stays on permanently.
  **免费收益**——提示词缓存、输入 token 卫生（包括提示词审计）、循环卫生、输出 token 卫生、批处理——在不降低输出质量的前提下降低支出。它们排在最前，且缓存要永久保持开启。
- **Tradeoffs** - budgets, effort, model choice, multi-model architectures - exchange cost for intelligence. They go last, because each one changes what the model can do, and overshooting costs quality that the free wins never touch.
  **权衡项**——预算、effort、模型选择、多模型架构——用成本换取智能。它们排在最后，因为每一项都会改变模型能做到什么，一旦过头，付出的质量代价是免费收益从不触及的。

**Where this workflow sits**: the `prompt-audit` subcommand (`shared/prompt-audit.md`) audits the prompt surface (prompts, skills, tool descriptions) alone; this workflow is the holistic cost pass - request shape, caching, loop structure, output, batching, effort, model - and runs that audit as one sub-lever of input hygiene (§ 2.2) rather than restating its patterns; and once the project has an eval, the levers become a hillclimb - one change at a time against the eval, keep or revert (Step 3).

**本工作流的定位**：`prompt-audit` 子命令（`shared/prompt-audit.md`）只审计提示词表面（提示词、技能、工具描述）；本工作流是整体性的成本处理——请求形态、缓存、循环结构、输出、批处理、effort、模型——并把该审计作为输入卫生的一个子杠杆（§ 2.2）来执行，而不是复述其模式；一旦项目有了评测（eval），这些杠杆就变成一次爬山——针对评测一次只改一处，保留或回退（步骤 3）。

Measured expectations below give the direction and rough size of each effect from Anthropic's published runs (sources at the end); the per-model figures live on those pages and change with each model release, so most are not restated here. They are directional, not guarantees - the validation loop in Step 3 is what makes a number true for this project - and fetching the Pricing and Cost Optimization pages is a required step, not background (Step 0 -> Fetch before you size). For measured figures, wherever a fetched page differs from what is quoted here, the page wins. For which history edits are valid, the Preserved thinking page governs (§ 2.1); the cost guide's and the cookbook's client-side prune recipes do not account for it.

下文的实测预期给出了 Anthropic 公开实验中每项效应的方向和大致量级（来源见文末）；分模型的数字位于那些页面上，且随每次模型发布而变化，因此此处大多不予复述。它们是方向性的，不是保证——步骤 3 的验证回路才是让数字对本项目成立的东西——而抓取 Pricing 与 Cost Optimization 页面是必经步骤，不是背景阅读（步骤 0 -> 定规模前先抓取）。就实测数字而言，凡抓取到的页面与此处引述不一致，以页面为准。至于哪些历史编辑是合法的，以 Preserved thinking 页面为准（§ 2.1）；成本指南与 cookbook 的客户端剪枝方案并未将其考虑在内。

---

## Step 0: Establish scope, quality bar, and baseline / 步骤 0：确定范围、质量标准与基线

**First, establish three things - from the request and the repository where they answer it, and from the user where they don't.** Unlike the prompt audit, this workflow is interactive by design: when context for a lever is missing, or a step would spend real money, work through it with the user rather than assuming. It is not expected to one-shot the audit. State all three at the top of the report (the baseline value itself may read "pending Step 1" at first).

**首先确定三件事——能从请求和仓库得到答案的从那里取，得不到的向用户问。**与提示词审计不同，本工作流在设计上就是交互式的：当某个杠杆缺少上下文，或某一步要花费真金白银时，与用户一起解决，而不是自行假设。不要求一次跑完整个审计。在报告开头陈述全部三项（基线值本身起初可以写"待步骤 1"）。

1. **Scope.** If the request names files or directories, that is the scope. Otherwise it is every place the project calls the Claude API - request builders, agent loops, batch jobs. Note distinct traffic classes (an interactive path and a nightly job are different workloads even on one key): the profile, the ranking, and every validation later run per class, and "cost per task" means nothing blended across classes. **Also establish which platform** the code targets (first-party Anthropic API, Claude Platform on AWS, Bedrock, Vertex, or Foundry) - feature availability varies, and it filters which levers are even on the table.
   **范围。**如果请求点名了文件或目录，那就是范围。否则就是项目中调用 Claude API 的每一处——请求构造器、代理循环、批处理作业。记下不同的流量类别（交互路径与夜间作业即便在同一把密钥上也是不同的工作负载）：画像、排序以及后续每项验证都按类别分别运行，"每任务成本"跨类别混合后毫无意义。**同时确定代码面向哪个平台**（Anthropic 一方 API、AWS 上的 Claude Platform、Bedrock、Vertex 还是 Foundry）——功能可用性各不相同，它会过滤掉哪些杠杆根本不在考虑之列。
2. **Quality bar.** Find the project's eval, test suite, or outcome checks for its LLM calls. If none exists, say so prominently in the report: without one, savings cannot be told apart from regressions. Do not stop - free wins are safe to propose regardless - but mark every tradeoff lever "needs an eval before applying", and ask the user what outcome check they can provide. An eval only validates the traffic class it covers: mark levers on uncovered paths the same way. If the only check is the user's own manual review, it gates free wins - it never clears a tradeoff. The full no-eval endgame - including a minimal eval recipe that unblocks tradeoffs - is in Step 3.
   **质量标准。**找到项目针对其 LLM 调用的评测、测试套件或结果校验。如果不存在，在报告中显著说明：没有它，节省与退化就无法区分。不要因此停下——免费收益无论如何都可以放心提出——但把每个权衡类杠杆标注为"应用前需要评测"，并询问用户能提供什么样的结果校验。评测只对它覆盖的流量类别有效：对未覆盖路径上的杠杆照此标注。如果唯一的校验是用户本人的人工审查，它可以约束免费收益——但永远不能放行权衡项。完整的无评测应对方案——包括一份能解锁权衡项的最小评测配方——在步骤 3。
3. **Baseline cost per task.** The baseline is whatever honest number is cheapest to obtain, in this order:
   **每任务基线成本。**基线就是以最廉价方式能得到的诚实数字，按以下优先顺序：
   - **From history, free**: with Admin API access, pull Step 1's usage and cost reports forward and compute the baseline from them - the reports supply the dollars, but the per-task denominator must come from the user or the application's own logs; or roll up the application's own logged `usage` objects per task, not per request - four token counts, each at its own rate: regular input, cache writes (1.25x input for the 5-minute duration, 2x for 1-hour), cache reads (a fraction of base input that differs by model), and output - multiplier structure as published on the pricing page; take the values from it when you fetch the rates, not from memory.
     **从历史记录，免费**：如有 Admin API 访问权限，把步骤 1 的用量与成本报告提前拉取过来并据此计算基线——报告给出金额，但每任务分母必须来自用户或应用自身的日志；或者按任务（而非按请求）汇总应用自己记录的 `usage` 对象——四项 token 计数，各按各的费率：普通输入、缓存写入（5 分钟时长为输入的 1.25 倍，1 小时为 2 倍）、缓存读取（基础输入的一个比例，因模型而异）、输出——倍率结构以定价页面公布者为准；抓取费率时以页面取值，不要凭记忆。
   - **From a baseline run, paid**: run the project's eval (or, with no eval, replay a representative sample of real requests) and roll up the same way. This spends real API money: state the expected cost - from Step 1's token estimates and live pricing, and "estimated - pending Step 1" is an acceptable first answer - **and get the user's approval before running it.** If the user declines the spend, estimate the baseline from the code and any bill figure they can read off the Console, label it an estimate, and continue.
     **从基线运行，付费**：运行项目的评测（没有评测时，重放一份有代表性的真实请求样本），并按同样方式汇总。这会花费真实的 API 资金：说明预期成本——依据步骤 1 的 token 估算与实时定价，"估算值——待步骤 1"是可以接受的第一版答案——**并在运行前取得用户批准。**如果用户拒绝这笔支出，就依据代码以及用户能从控制台读到的任何账单数字来估算基线，标注为估算值，然后继续。

   **Fetch before you size - this gates every rate and every measured figure in the audit.** Before any dollar amount, multiplier, or published figure is written into the report or into code, WebFetch two rows from `shared/live-sources.md`: **Pricing** (per-model rates and multipliers) and **Cost Optimization** (current measured expectations and the starting-model recommendation). Fetch both in one turn. When a lever that edits conversation history, `system`, or `tools` (§ 2.2, § 2.3, a model switch) enters the shortlist, also fetch the Preserved thinking page (URL in § 2.1) before proposing its diff, for which models and accounts run the check. Record in the report which pages were fetched and when, so a reader can see it happened, and cite the fetched page beside each figure taken from it. Remembered rates or figures, for any model, are not a source. If a fetch fails, say so in the report. For rates: with Admin API access, derive effective realized rates by dividing cost-report amounts by the usage report's matching token counts (same model, same token type); otherwise ask the user for the current rates. For measured expectations: size them in relative buckets with no figures. Only if none of these is available and the user still wants a number may one appear, and then labeled "unverified - from memory" every place it appears.

   **定规模前先抓取——这一步约束着审计中的每一条费率和每一个实测数字。**在任何金额、倍率或公开数字写入报告或代码之前，先用 WebFetch 抓取 `shared/live-sources.md` 中的两行：**Pricing**（分模型费率与倍率）和 **Cost Optimization**（当前实测预期与起始模型建议）。在同一轮中把两者都抓取。当某个会编辑对话历史、`system` 或 `tools` 的杠杆（§ 2.2、§ 2.3、模型切换）进入候选清单时，在提出其 diff 之前还要抓取 Preserved thinking 页面（URL 见 § 2.1），以确认哪些模型与账户会执行该检查。在报告中记录抓取了哪些页面、何时抓取，让读者能看到这一步确实发生过，并在每个取自页面的数字旁注明出处页面。凭记忆记下的任何模型的费率或数字都不是来源。如果抓取失败，在报告中说明。费率方面：如有 Admin API 访问权限，用成本报告金额除以用量报告中对应的 token 计数（同模型、同 token 类型）推导实际实现费率；否则向用户询问当前费率。实测预期方面：以不带数字的相对档位来定规模。只有当以上途径都不可用而用户仍想要一个数字时才可以给出，并且在其出现的每一处都标注"未经验证——凭记忆"。

【评论】反复要求实时抓取官方页面、明令禁止凭记忆引用费率，是一种防幻觉设计：防止模型用训练时记错的旧价格推算节省额。

   For counting tokens in prompts and files, see `shared/token-counting.md` (`count_tokens` returns the count without running inference). Sanity-check an estimated baseline against any known monthly bill: divergence usually means multi-turn history growth the single-turn estimate missed.

   统计提示词与文件中的 token 数见 `shared/token-counting.md`（`count_tokens` 无需运行推理即返回计数）。把估算的基线与任何已知的月账单做一致性核对：偏差通常意味着单轮估算遗漏了多轮历史增长。

## Step 1: Profile where the tokens go / 步骤 1：画像 token 的去向

The profile can be measured or estimated. Measure when the organization's access allows it; fall back to reading the code. Either way, the levers that pay are decided by the workload's shape, not by the list of what exists.

画像可以是实测的，也可以是估算的。组织权限允许时就实测；否则退回到读代码。无论哪种方式，值得动的杠杆由工作负载的形态决定，而不是由现存事物的清单决定。

### Measure it - the Usage and Cost Admin API (preferred) / 实测——用量与成本 Admin API（首选）

If the user has an **Admin API key** (`sk-ant-admin01-...` - a different key type from the standard API key; not available for individual accounts - creation and scopes are covered in the Admin API docs, reachable from the **Usage and Cost Admin API** URL in `shared/live-sources.md`), pull the real numbers instead of estimating. These are report reads, not model calls - they consume no tokens. Full parameters and response schemas: the **Usage and Cost Admin API** URL in `shared/live-sources.md`.

如果用户有 **Admin API 密钥**（`sk-ant-admin01-...`——与标准 API 密钥不同的另一种密钥类型；个人账户不可用——创建方式与权限范围见 Admin API 文档，可从 `shared/live-sources.md` 中的 **Usage and Cost Admin API** URL 访问），就拉取真实数字而不是估算。这些是报告读取，不是模型调用——不消耗 token。完整参数与响应模式：`shared/live-sources.md` 中的 **Usage and Cost Admin API** URL。

- **Token profile**: `GET /v1/organizations/usage_report/messages` with `group_by[]=model` and `bucket_width=1d` (the default page is 7 daily buckets - raise `limit`, up to 31; the `group_by` dimensions also include `api_key_id`, `workspace_id`, `service_tier`, and `context_window`, among others). Each result splits into exactly the quantities the levers below act on: `uncached_input_tokens`, `cache_read_input_tokens`, `cache_creation.ephemeral_5m_input_tokens` / `ephemeral_1h_input_tokens`, and `output_tokens`.
  **token 画像**：`GET /v1/organizations/usage_report/messages`，带 `group_by[]=model` 与 `bucket_width=1d`（默认页为 7 个按天的桶——调高 `limit`，最多 31；`group_by` 维度还包括 `api_key_id`、`workspace_id`、`service_tier`、`context_window` 等）。每个结果恰好拆分为下文各杠杆所作用的量：`uncached_input_tokens`、`cache_read_input_tokens`、`cache_creation.ephemeral_5m_input_tokens` / `ephemeral_1h_input_tokens`，以及 `output_tokens`。
- **Dollar profile**: `GET /v1/organizations/cost_report` (daily granularity, USD as decimal strings in cents) with `group_by[]=description`; description-grouped results carry structured `model`, `cost_type`, `token_type`, and `service_tier` fields - `token_type` makes the cache split readable directly in dollars. Code execution appears under a `Code Execution Usage` description; Priority Tier costs are not included in this endpoint - track those through the usage endpoint's `service_tier` dimension.
  **金额画像**：`GET /v1/organizations/cost_report`（按天粒度，美元以"分"为单位的十进制字符串表示），带 `group_by[]=description`；按 description 分组的结果带有结构化的 `model`、`cost_type`、`token_type`、`service_tier` 字段——`token_type` 让缓存拆分可以直接以美元读出。代码执行出现在 `Code Execution Usage` 描述之下；Priority Tier 成本不包含在此端点中——请通过用量端点的 `service_tier` 维度跟踪。
- Data appears within about 5 minutes of a request completing; poll at most once per minute for sustained use.
  数据在请求完成约 5 分钟内出现；持续使用时每分钟至多轮询一次。
- Caveats by platform: Claude Enterprise (claude.ai) organizations use the Analytics API instead, and the endpoints are not currently available on Claude Platform on AWS - there, ask the user to read the totals off the Console's Usage and Cost pages and relay them. The Usage and Cost page does not list Amazon Bedrock, Google Cloud, or Microsoft Foundry traffic: if the code targets one of those, do not assume these reports cover it - ask whether a Console organization and Admin key exist for that traffic, and otherwise profile from the application's logged `usage` objects or the platform's own billing view.
  按平台的注意事项：Claude Enterprise（claude.ai）组织改用 Analytics API，而这些端点目前在 AWS 上的 Claude Platform 上不可用——在那里，请用户从控制台的 Usage 与 Cost 页面读出总数并转述。Usage and Cost 页面不列出 Amazon Bedrock、Google Cloud 或 Microsoft Foundry 流量：如果代码面向其中之一，不要假定这些报告覆盖了它——询问该流量是否存在对应的控制台组织与 Admin 密钥，否则从应用记录的 `usage` 对象或平台自身的账单视图做画像。

The measured profile answers directly: the real cache hit rate (`cache_read_input_tokens` against uncached input), how much traffic already rides the batch tier, the input/output balance, and where spend concentrates by model, key, and workspace. **Check that the measured footprint plausibly matches the audited code** (same models, a believable order of magnitude): the report covers the whole organization, and a key shared across projects blends their traffic - making per-project reads, including Step 3's post-cutover confirmation, unattributable. On a mismatch, reconcile against the code estimate, scope usage-report queries by `api_key_ids[]` / `workspace_ids[]` where the separation exists (the cost report takes neither filter - it segments only by workspace, via `group_by`), and recommend per-project keys or workspaces as a measurement prerequisite where it doesn't. Optimization effort follows the audited scope's spend, not the org blend.

实测画像直接回答：真实缓存命中率（`cache_read_input_tokens` 相对未缓存输入）、已有多少流量走批处理档、输入/输出平衡，以及支出按模型、密钥、工作区集中在哪里。**检查实测足迹与被审计的代码是否合理吻合**（相同模型、可信的数量级）：报告覆盖整个组织，跨项目共享的密钥会把它们的流量混在一起——使按项目读取（包括步骤 3 切换后的确认）无法归因。若有不符，与代码估算核对；在存在隔离之处用 `api_key_ids[]` / `workspace_ids[]` 限定用量报告查询（成本报告不接受这两种过滤——只能通过 `group_by` 按工作区分段）；在没有隔离之处，建议把按项目的密钥或工作区作为度量的前置条件。优化精力跟随被审计范围的支出，而不是组织整体的混合值。

### Estimate it from the code / 从代码估算

Without Admin API access (no Admin key, a Claude Enterprise organization, Claude Platform on AWS - whose feature availability `shared/claude-platform-on-aws.md` covers - or another cloud platform these reports do not cover) - and even with it, for the structural facts no usage report can show - read the request-building code:

在没有 Admin API 访问权限时（没有 Admin 密钥、Claude Enterprise 组织、AWS 上的 Claude Platform——其功能可用性见 `shared/claude-platform-on-aws.md`——或这些报告不覆盖的其他云平台）——即便有权限，对于那些任何用量报告都显示不了的结构性事实——去读构造请求的代码：

> **Per-model defaults, parameter support, and per-platform feature availability change across releases.** For any "what happens when `thinking`/`effort` is omitted", "does this model accept `effort`", "what levels does it support", or "is this feature available on Bedrock/Vertex/Foundry" question, read the answer from SKILL.md -> Thinking & Effort, `shared/models.md`, or `shared/platform-availability.md` (or the live Models API) - never assume, and never encode the answer in this guide.

> **分模型的默认值、参数支持与分平台功能可用性随版本而变。**任何"省略 `thinking`/`effort` 会怎样"、"这个模型是否接受 `effort`"、"支持哪些档位"或"该功能在 Bedrock/Vertex/Foundry 上是否可用"的问题，都从 SKILL.md -> Thinking & Effort、`shared/models.md` 或 `shared/platform-availability.md`（或实时 Models API）读取答案——绝不臆测，也绝不把答案硬编码进本指南。

- **Prefix**: how large are the system prompt and tool schemas, and is anything dynamic (timestamps, request IDs) interpolated into them?
  **前缀**：系统提示词与工具模式有多大？是否有动态内容（时间戳、请求 ID）被插入其中？
- **Reference material**: is documentation or a manual inlined into every request?
  **参考资料**：是否有文档或手册被内联进每个请求？
- **Tools**: how many schema tokens, and does every request need every tool?
  **工具**：模式占多少 token？每个请求是否都需要全部工具？
- **Loop**: how many turns deep, and do bulky tool results accumulate across them?
  **循环**：深达多少轮？庞大的工具结果是否跨轮累积？
- **Media**: are images, PDFs, or large files entering the context at full size?
  **媒体**：图片、PDF 或大文件是否以原始尺寸进入上下文？
- **Output**: how long are visible responses, and what is `max_tokens` set to?
  **输出**：可见回复有多长？`max_tokens` 设为多少？
- **Model and effort**: which model, which effort, and was either ever swept against an eval? Look up what the model does when both are omitted (SKILL.md -> Thinking & Effort) - an unset default that runs thinking is a hidden output-token line item, and because default effort differs by model, an unset `effort` can run a level higher or lower after a model change. Note too whether the model is a generation or two behind the current one in its tier - moving up is a lever (§ 2.7).
  **模型与 effort**：用的哪个模型、哪个 effort？两者是否曾对照评测扫过？查一下两者都省略时该模型的行为（SKILL.md -> Thinking & Effort）——一个会运行思考的未设置默认值是一条隐藏的输出 token 开销项；而且由于默认 effort 因模型而异，模型变更后未设置的 `effort` 可能落在更高或更低的档位。也注意该模型是否比所在层级当前型号落后一两代——升级也是一根杠杆（§ 2.7）。
- **Caching**: are there `cache_control` breakpoints already, and what do `cache_read_input_tokens` / `cache_creation_input_tokens` show in practice?
  **缓存**：是否已有 `cache_control` 断点？`cache_read_input_tokens` / `cache_creation_input_tokens` 在实践中表现如何？
- **Latency tolerance**: is a user waiting on every response, or can some work batch?
  **延迟容忍度**：是否每个响应都有用户在等？还是部分工作可以批处理？
- **Price modifiers**: does any request set `speed`, `inference_geo`, or another parameter billed at a premium over the standard rate (rates: the Pricing URL in `shared/live-sources.md`)? If a meaningful share of responses end in `stop_reason: "refusal"`, size that as its own line item - some refusals are billed (`shared/model-migration.md` -> `refusal` stop reason). The token counts alone show neither.
  **价格修饰符**：是否有请求设置了 `speed`、`inference_geo` 或其他按高于标准费率计费的参数（费率见 `shared/live-sources.md` 中的 Pricing URL）？如果相当比例的响应以 `stop_reason: "refusal"` 结束，把它单独列为一条开销项——有些拒答是计费的（`shared/model-migration.md` -> `refusal` stop reason）。仅看 token 计数两者都看不出来。

### Ask for the app's own usage logs first / 先索要应用自己的用量日志

Before ranking on estimates, **ask the user whether the application already logs `response.usage` per request** - and if so, to paste a representative day's worth. That turns cache hit rate, the input/output split, and thinking-token spend from guesses into measurements at zero API cost, and it decides which tier of the ranking table below applies. Two fields sharpen it where the response carries them: `usage.output_tokens_details.thinking_tokens` meters thinking spend directly, and a request that runs a server-side step such as compaction itemizes that step's tokens under `usage.iterations` - sum the iterations rather than reading the top-level counts alone. If the app doesn't log usage yet, note that adding it is itself a free-win diff (Step 3) and proceed on the code estimate.

在基于估算排序之前，**询问用户应用是否已按请求记录 `response.usage`**——如果有，请其粘贴有代表性的一天量。这会把缓存命中率、输入/输出拆分与思考 token 花费从猜测变成零 API 成本的实测，并决定下文排序表适用哪一档。响应携带时有两个字段能让它更精确：`usage.output_tokens_details.thinking_tokens` 直接计量思考花费；而运行服务端步骤（如压缩）的请求会在 `usage.iterations` 下分项列出该步骤的 token——把各次迭代求和，不要只读顶层计数。如果应用尚未记录用量，注明添加它本身就是一份免费收益 diff（步骤 3），然后按代码估算继续。

**Estimating cache hit rate without usage data.** If the app logs request timestamps, simulate the TTL walk: sort timestamps, count a hit whenever the gap to the previous request is <= TTL (reads refresh the entry), and run it for each cache TTL the platform offers (see `shared/prompt-caching.md`) - the difference between durations is the longer-TTL lever's ceiling on the user's real traffic. If only aggregate volume is known, approximate with Poisson arrivals: hit rate ~ `1 - e^(-lambda·TTL)` where lambda is requests per second. Either beats comparing average gap to TTL, which ignores burstiness.

**在没有用量数据时估算缓存命中率。**如果应用记录请求时间戳，就模拟 TTL 行走：对时间戳排序，凡与前一请求的间隔 <= TTL 就计一次命中（读取会刷新条目），并对平台提供的每种缓存 TTL 各跑一遍（见 `shared/prompt-caching.md`）——两种时长之差就是更长 TTL 杠杆在用户真实流量上的上限。如果只知道总量，用泊松到达近似：命中率 ~ `1 - e^(-lambda·TTL)`，其中 lambda 是每秒请求数。这两种方法都优于把平均间隔与 TTL 直接比较——后者忽略了突发性。

### Rank the levers / 对杠杆排序

Before touching code, size each lever the profile makes applicable so the shortlist can be ordered. **How you quote the size depends on what data you have** - an estimate and a measurement must not look the same in the report:

在改动代码之前，先给画像判定适用的每个杠杆定出规模，让候选清单可以排序。**引用规模的方式取决于你手上有什么数据**——估算与实测在报告中绝不能呈现得一样：

| Data available | Quote each ceiling as |
|---|---|
| Admin API usage/cost report | **Dollar range**, labeled `measured` |
| App-side `usage` logs, or a user-reported bill total only | **% of current bill**, with dollars only as a parenthetical "(~ $Y at your reported $X/mo)" - the % is the claim; the $ is the user's own arithmetic |
| Neither (pure code read) | **Relative buckets** - "largest / medium / small", or an order-of-magnitude band - no specific figures |

| 可用数据 | 上限的引用方式 |
|---|---|
| Admin API 用量/成本报告 | **美元区间**，标注为 `measured` |
| 仅有应用侧 `usage` 日志，或用户报出的账单总额 | **当前账单的百分比**，美元只作括注"(~ $Y at your reported $X/mo)"——百分比是主张；美元是用户自己的算术 |
| 两者皆无（纯读代码） | **相对档位**——"最大/中等/较小"或数量级区间——不给具体数字 |

**Before sizing, drop any lever the target platform doesn't support** (`shared/platform-availability.md` is the single source of truth - do not assume 1P availability carries to Bedrock, Vertex, Foundry, or Claude Platform on AWS). A lever that can't ship on the user's platform isn't worth ranking; list it under "skipped" with the availability reason instead.

**定规模之前，先剔除目标平台不支持的杠杆**（`shared/platform-availability.md` 是唯一权威来源——不要假定一方 API 的可用性会延伸到 Bedrock、Vertex、Foundry 或 AWS 上的 Claude Platform）。在用户平台上无法落地的杠杆不值得排序；把它连同可用性原因列在"已跳过"之下。

Within whichever unit applies, size each lever from the measured (or estimated) spend components and the measured expectations described in Step 2 and quantified on the Cost Optimization page - use the copy fetched in Step 0 for any per-model figure (if that fetch failed, carry the ceiling as a relative bucket) - for example:

在适用的单位体系内，从实测（或估算）的支出分量、步骤 2 所述并在 Cost Optimization 页面上量化的实测预期出发，给每个杠杆定规模——任何分模型数字都用步骤 0 抓取的副本（若那次抓取失败，则把上限按相对档位携带）——例如：

- **Caching ceiling**: the spend on input that is shared and byte-stable across requests - the would-be prefix - re-billed at the cache-read rate of the model in use - a fraction of base input that differs by model (the Pricing URL in `shared/live-sources.md`; `shared/prompt-caching.md` § API reference carries the break-even arithmetic). Blend the measured `uncached_input_tokens` with the code profile here: unique per-request payload can never cache, so on a workload that is mostly payload (or already well cached) this ceiling is honestly small. Sanity-bound the result against the published agent-loop range (a several-fold reduction at high hit rates - current figures: the Cost Optimization URL in `shared/live-sources.md`).
  **缓存上限**：跨请求共享且逐字节稳定的输入——即潜在前缀——本可按所用模型的缓存读取费率重新计价的那部分支出——该费率是基础输入的一个比例，因模型而异（见 `shared/live-sources.md` 中的 Pricing URL；`shared/prompt-caching.md` § API reference 给出盈亏平衡算术）。此处把实测的 `uncached_input_tokens` 与代码画像融合：每请求独有、无法缓存的负载永远不命中，因此在以负载为主（或已缓存良好）的工作负载上，这个上限确实就小。把结果与公布的代理循环区间做一致性约束（高命中率下可降低数倍——当前数字见 `shared/live-sources.md` 中的 Cost Optimization URL）。
- **Batch ceiling**: 50% of the spend on standard-tier traffic that no one is waiting on. The model-grouped profile cannot see that split - segment first: group by `service_tier` to find what already batches, use a finer `bucket_width` to spot scheduled spikes, and ask the user which traffic can wait.
  **批处理上限**：无人等候的标准档流量支出的一半。按模型分组的画像看不到这一拆分——先分段：按 `service_tier` 分组找出已在批处理的部分，用更细的 `bucket_width` 发现定时出现的峰值，并询问用户哪些流量可以等。
- **Input-hygiene ceiling**: the share of input spend going to reference material, tool schemas, or oversized media that the § 2.2 levers would remove or defer.
  **输入卫生上限**：输入支出中流向 § 2.2 杠杆会移除或延后的参考资料、工具模式或超大媒体的那一份。
- **Effort/model ceiling**: the published tradeoff curves (the Cost Optimization page) applied to the biggest spend concentrations - carried as a range, since the quality cost is unknown until the eval runs.
  **effort/模型上限**：把公开的权衡曲线（Cost Optimization 页面）套用到最大的支出集中处——以区间形式携带，因为质量代价在评测运行前是未知的。

Ceilings that claim the same tokens (caching an inlined document versus deleting it) are mutually exclusive: compute each ceiling unconditionally, rank, then deflate each for its overlap with the levers above it, so the shortlist can never sum past the bill.

对同一批 token 提出主张的上限（缓存一份内联文档与删掉它互斥）：先无条件计算每个上限，排序，然后按其与更上方杠杆的重叠逐级折减，使候选清单之和永远不会超过账单。

Present the ranked shortlist with the profile evidence behind each number - labeled as ranked by savings ceiling, not application order (Step 2's § 2.x numbering decides the sequence) - and say where the list stops: a lever whose ceiling is a small fraction of the bill - or would not repay the approved runs and effort needed to validate it - does not earn an eval cycle, and most levers will not earn a place on any given workload (the "Workload shape -> lever" table near the end of this file is the map for matching profile to levers). On a small bill the honest shortlist may be empty: "nothing here is worth changing" is a successful finding, not a failure - report it plainly. Expected savings are planning numbers, not results - Step 3's measurements are the results.

呈现排好序的候选清单时，附上每个数字背后的画像证据——标注为按节省上限排序、而非应用顺序（步骤 2 的 § 2.x 编号决定顺序）——并说明清单到哪里为止：一个上限只占账单很小比例——或收回不了验证它所需的已批准运行与精力——的杠杆不配得到一轮评测，而且大多数杠杆在任一给定工作负载上都得不到位置（本文件末尾附近的"工作负载形态 -> 杠杆"表就是把画像对应到杠杆的地图）。账单很小时，诚实的候选清单可能是空的："这里没有值得改的"是一个成功结论，不是失败——如实报告。预期节省是规划数字，不是结果——步骤 3 的测量才是结果。

## Step 2: Work the levers in order / 步骤 2：按顺序运用杠杆

Free wins may be applied directly when the request asked for edits (a bare subcommand invocation has not asked - propose). Tradeoff levers (2.6 onward) are always presented with their measured quality cost and applied only on the user's explicit acceptance - never trade accuracy for cost silently. And every run that exercises the model - the baseline, each lever's validation pass - spends real API money: get explicit approval before each one, with the expected cost, or once as a Step 3 measurement budget that covers them.

当请求本身要求改动时，免费收益可以直接应用（单纯调用子命令不算要求——只提出建议）。权衡类杠杆（2.6 起）总是连同其实测质量代价一起呈现，并且只在用户明确接受后才应用——绝不在暗中用准确率换成本。而每一次真正驱动模型的运行——基线、每个杠杆的验证轮——都花费真实 API 资金：每次之前取得明确批准并说明预期成本，或者一次性取得覆盖它们的步骤 3 测量预算。

【评论】本节把"花钱必须先获用户明确批准"设为硬性门槛，将成本控制与权限控制绑定，防止代理自行产生大额 API 支出。

Pricing multipliers quoted below (cache write rates, batch discount) are current as of writing - confirm against the Pricing URL in `shared/live-sources.md` before computing any ceiling. The cache-read rate differs by model and is deliberately not quoted here.

下文引用的定价倍率（缓存写入费率、批处理折扣）以撰写时为准——计算任何上限之前先对照 `shared/live-sources.md` 中的 Pricing URL 确认。缓存读取费率因模型而异，此处刻意不予引述。

### 2.1 Prompt caching - first, and it stays on / 2.1 提示词缓存——第一顺位，且永久开启

Every turn of an agentic task resends the entire growing conversation - system prompt, tool definitions, every prior turn - so a 40-turn task sends its first turn 40 times and task cost grows with roughly the square of turn count. Caching does not stop the resending; it reprices everything already cached to the model's cache-read rate, a small fraction of base input.

代理型任务的每一轮都重发整段不断增长的对话——系统提示词、工具定义、此前每一轮——因此一个 40 轮的任务会把第 1 轮发送 40 次，任务成本大致随轮数的平方增长。缓存并不阻止重发；它把已缓存的一切按模型的缓存读取费率重新计价，那是基础输入的一个很小比例。

For design and placement - the prefix-match invariant, classifying inputs by stability, breakpoint patterns, the anti-pattern table - **read `shared/prompt-caching.md` and follow its workflow**; do not improvise `cache_control` markers. Points that matter specifically for cost:

设计与摆放——前缀匹配不变量、按稳定性分类输入、断点模式、反模式表——**读 `shared/prompt-caching.md` 并遵循其工作流**；不要即兴发挥 `cache_control` 标记。专门与成本相关的要点：

- **Measured expectation**: the largest single lever on every model and benchmark Anthropic measured - it cut agent-loop cost several-fold at high hit rates.
  **实测预期**：在 Anthropic 实测过的每个模型与基准上都是最大的单根杠杆——高命中率下把代理循环成本降低数倍。
- **Explicit breakpoints when many independent conversations share a static prefix** (or prefix layers change at different rates). Automatic caching only amortizes within one conversation; in the cookbook's worked example, one explicit breakpoint on the static system prefix roughly halved cost per task across a queue of independent tasks. The robust shape for agent loops - one explicit breakpoint on the static prefix plus top-level automatic caching for the tail - and the cases where automatic alone is a pure surcharge are in `shared/prompt-caching.md` § Automatic vs explicit breakpoints.
  **当许多独立会话共享静态前缀时使用显式断点**（或前缀各层以不同速率变化时）。自动缓存只在单个会话内摊销；在 cookbook 的示例中，静态系统前缀上的一个显式断点把一队列独立任务的每任务成本大致砍半。代理循环的稳健形态——静态前缀上一个显式断点，加上对尾部的顶层自动缓存——以及仅用自动缓存纯属额外支出的情形，见 `shared/prompt-caching.md` § Automatic vs explicit breakpoints。
- **Use the 1-hour cache duration when the loop waits on humans between turns.** It writes at 2x instead of 1.25x, so it pays only through the misses it prevents, and on gaps longer than an hour it costs more than the default. Decide from the distribution of start-to-start gaps between requests (generation time counts against the TTL), not their average - the table in `shared/prompt-caching.md` § Choosing the TTL, which also covers the `max_tokens: 0` keep-alive that can beat the longer duration for pauses well short of an hour, more so where the model's cache reads are cheapest.
  **当循环在轮次之间等待人工时，使用 1 小时缓存时长。**它以 2 倍而非 1.25 倍写入，因此只有靠它避免的未命中才能回本，而间隔超过一小时时它比默认更贵。依据请求起始间隔的分布（生成时间计入 TTL）而非平均值来决定——见 `shared/prompt-caching.md` § Choosing the TTL 中的表；该节还覆盖了 `max_tokens: 0` 保活，对于远短于一小时的停顿它可能胜过更长时长，在模型缓存读取最便宜的场合尤其如此。
- **Audit for mid-task cache-breakers**: dynamic content above a breakpoint; changing `thinking` or top-level `effort` between requests (always invalidates the messages cache, and on some models the tools+system cache too - `shared/prompt-caching.md` § Invalidation hierarchy); setting or changing a structured-output format; switching `speed`; changing a task budget mid-task; every context-editing pass; a thinking block the API drops (the messages cache changes from that block onward); switching models mid-conversation (caches are per-model).
  **审计任务中途的缓存破坏者**：断点上方的动态内容；请求之间改变 `thinking` 或顶层 `effort`（总是使 messages 缓存失效，在某些模型上还使 tools+system 缓存失效——`shared/prompt-caching.md` § Invalidation hierarchy）；设置或更改结构化输出格式；切换 `speed`；任务中途更改任务预算；每一次上下文编辑；被 API 丢弃的思考块（messages 缓存自该块起改变）；会话中途切换模型（缓存按模型隔离）。
- **Not breakers, and where to put a change that is one**: setting effort explicitly to the model's default is the same as omitting it; a per-message effort change (beta) keeps the cache only where the model and the `thinking` setting the code sends both accept it, and returns a 400 otherwise (which: the per-message section of the **Effort Parameter** URL in `shared/live-sources.md`); and most edits to `system` or `tools` have an append-only form that keeps the prefix: a mid-conversation system message (a `role: "system"` message inside `messages`) instead of editing `system`, `tool_addition` / `tool_removal` blocks instead of editing `tools`, and a turn-scoped reminder (`clear_at`) for text that should not persist. The table in `shared/prompt-caching.md` § Invalidation hierarchy lists each with its model availability. When a model or top-level effort change is wanted anyway, make it where the cache is already going to break: on the first request after a compaction (not the one that triggers it), or between conversations.
  **不属于破坏者的情形，以及确需变更时的摆放位置**：把 effort 显式设为模型默认值与省略它等价；每消息 effort 变更（beta）只在模型与代码所发 `thinking` 设置都接受时保持缓存，否则返回 400（说明见 `shared/live-sources.md` 中 **Effort Parameter** URL 的 per-message 一节）；而且对 `system` 或 `tools` 的大多数编辑都有保持前缀的仅追加形式：用会话中途系统消息（`messages` 内一条 `role: "system"` 消息）代替编辑 `system`，用 `tool_addition` / `tool_removal` 块代替编辑 `tools`，对不应持久化的文本用轮次范围的提醒（`clear_at`）。`shared/prompt-caching.md` § Invalidation hierarchy 中的表列出了各情形及其模型可用性。反正要换模型或顶层 effort 时，把它放在缓存本来就即将断裂的位置：压缩之后的第一条请求（而不是触发压缩的那条），或会话之间。
- **On models that run the preserved-thinking check, an edit to `system`, `tools`, or earlier messages breaks the model's earlier reasoning as well as the cache.** Each thinking block is tied to the conversation that produced it, and the edits that invalidate it are the same edits that restart the cache, so one discipline serves both: keep `system` and `tools` fixed for the session and treat `messages` as append-only. `cache_control` markers, `effort`, `max_tokens`, and `tool_choice` are outside the check. A diff that moves content out of `system` or `tools` is still an edit for conversations in flight when it deploys: have the diff keep sending the stored `system` and `tools` bytes to those conversations and the new bytes to new conversations only. Where the harness cannot do that, say in the report that in-flight conversations take one cold miss and, where the check is enforced, are rejected until their thinking blocks are dropped (next bullet).
  **在运行 preserved-thinking 检查的模型上，编辑 `system`、`tools` 或较早的消息既破坏模型此前的推理，也破坏缓存。**每个思考块都与产生它的对话绑定，使其失效的编辑与重启缓存的编辑是同一批，因此一套纪律同时服务两者：会话期间保持 `system` 与 `tools` 固定，并把 `messages` 当作仅追加。`cache_control` 标记、`effort`、`max_tokens`、`tool_choice` 不在检查范围内。把内容移出 `system` 或 `tools` 的 diff，对部署时仍在进行中的会话而言仍是编辑：让 diff 对那些会话继续发送原有 `system` 与 `tools` 字节，只对新会话发送新字节。在 harness 无法做到之处，在报告中说明：进行中的会话将承受一次冷未命中，且在强制执行检查之处，其思考块被丢弃前会被拒绝（见下一条）。
- **What an invalid edit costs**: by default the request is rejected where the check is enforced. Opting into dropping the failing blocks instead (the Preserved thinking page has the field, and the beta header that both it and the response's list of dropped blocks need) leaves them unbilled, but the session can still cost more, because the model may think again to rebuild them. Which models and accounts are checked changes with releases - read it from that page (`https://platform.claude.com/docs/en/build-with-claude/preserved-thinking.md`), never from this guide - and hand off to the `preserved-thinking-migration` subcommand (`shared/preserved-thinking-migration.md`) to find and fix a harness's edits.
  **一次失效编辑的代价**：在强制执行检查之处，默认行为是请求被拒绝。改为选择丢弃未通过检查的块（Preserved thinking 页面有该字段，以及它与响应中已丢弃块列表都需要的 beta 头）可以让它们不计费，但会话仍可能花得更多，因为模型可能重新思考以重建它们。哪些模型与账户会被检查随版本变化——从那个页面读取（`https://platform.claude.com/docs/en/build-with-claude/preserved-thinking.md`），绝不从本指南读取——并交接给 `preserved-thinking-migration` 子命令（`shared/preserved-thinking-migration.md`）去发现并修复某个 harness 的编辑。
- **Short prefixes**: the minimum cacheable prefix is per-model, so after any model change re-check whether a short prompt still caches, or now does - table in `shared/prompt-caching.md` § API reference.
  **短前缀**：最小可缓存前缀因模型而异，因此任何模型变更后要重新检查一个短提示词是否仍能缓存、或现在能够缓存了——表见 `shared/prompt-caching.md` § API reference。
- **Verify from usage, not from code review - and re-verify after every prompt-assembly change**: on a warmed-up loop, `cache_read_input_tokens` should dominate regular `input_tokens`, and `cache_creation_input_tokens` should be roughly one turn's worth, not the whole conversation (the Cost Optimization page gives the current bar for how much of its input a healthy loop reads from the cache). If it isn't, hunt for a cache-breaker with the healthy-loop signature in `shared/prompt-caching.md` § Verifying cache hits - unless the workload's input is mostly unique per-request payload (which can never cache), or the misses are concurrent-batch artifacts (§ 2.5); neither is a breaker, and neither has a fix. To localize the breaker, reach for cache diagnostics first where the platform has it (availability in `shared/platform-availability.md`): include a `diagnostics` object on every request (`previous_message_id: null` on the first, the previous response's `id` after that), and the response's `diagnostics` object names where the two requests diverged - no payload logging needed. `shared/prompt-caching.md` § Verifying cache hits also sends a beta header on every request; the Cache diagnostics page (`https://platform.claude.com/docs/en/build-with-claude/cache-diagnostics.md`) says that header is no longer required, that requests that still send it work as before, and that the `diagnostics` object is the opt-in, so a harness that already sends the header can keep it, and every request needs the object either way. That page has the current shape. On other platforms, fall back to the payload-diff method in the same section.
  **从用量验证，而不是从代码审查验证——并且每次提示词组装变更后重新验证**：在已预热的循环上，`cache_read_input_tokens` 应当压过普通 `input_tokens`，`cache_creation_input_tokens` 应当约为单轮的量，而不是整段对话（Cost Optimization 页面给出健康循环从缓存读取其输入比例的现行标准）。若不是，就按 `shared/prompt-caching.md` § Verifying cache hits 中的健康循环特征去猎取缓存破坏者——除非工作负载的输入以每请求独有的负载为主（那永远无法缓存），或未命中是并发批处理的伴生物（§ 2.5）；两者都不是破坏者，也都没有修复办法。要定位破坏者，平台有缓存诊断时优先用它（可用性见 `shared/platform-availability.md`）：在每个请求上携带 `diagnostics` 对象（首个为 `previous_message_id: null`，其后为上一响应的 `id`），响应的 `diagnostics` 对象会指出两个请求在哪里分岔——不需要记录负载。`shared/prompt-caching.md` § Verifying cache hits 还在每个请求上发送一个 beta 头；Cache diagnostics 页面（`https://platform.claude.com/docs/en/build-with-claude/cache-diagnostics.md`）说明该头已不再必需、仍发送它的请求照旧工作、`diagnostics` 对象才是选择加入项，因此已发送该头的 harness 可以保留它，而无论哪种方式每个请求都需要该对象。该页面有当前形态。在其他平台上，退回同一节的负载 diff 方法。
- **The cache probe, when there is no usage history to read**: a scratch script for the project's own stack that sends one representative request twice, byte-identical; prints all four usage meters (`input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`) for both; and exits non-zero if the second request's `cache_read_input_tokens` is zero. Ship it alongside the caching diff so the user can run the before/after themselves. It spends real tokens and may execute the project's tools - run it only under the standing approval rule, and point it at a scratch environment if the request's tools mutate state.
  **没有用量历史可读时的缓存探针**：一个针对项目自身技术栈的临时脚本，把同一代表性请求逐字节相同地发送两次；打印两者的全部四项用量计量（`input_tokens`、`cache_creation_input_tokens`、`cache_read_input_tokens`、`output_tokens`）；并在第二个请求的 `cache_read_input_tokens` 为零时以非零码退出。随缓存 diff 一起交付，让用户能自己跑前后对比。它花费真实 token 并可能执行项目的工具——只在常设批准规则下运行，且当请求的工具会改动状态时指向临时环境。

### 2.2 Input tokens - progressive disclosure / 2.2 输入 token——渐进式披露

Send the model what the task needs, let it fetch the rest. Each sub-lever has a skip-when; the caveat at the end of this section governs all of them.

把任务需要的发给模型，其余让它自己去取。每个子杠杆都有"何时跳过"；本节末尾的告示约束全部子杠杆。

- **Large reference document in every prompt** -> move it behind a tool or skill so the model retrieves sections on demand. Skip when most calls consult most of it anyway - a document in the cached prefix is cheap - or when the eval shows misses on cases that hinge on rules the model now has to go looking for.
  **每个提示词里都带大型参考文档** -> 把它挪到工具或技能之后，让模型按需检索章节。当大多数调用反正要用到大部分内容时跳过——缓存前缀中的文档很便宜；或当评测显示在依赖模型须自行寻找规则的用例上出现失误时跳过。
- **Tool recaps in the system prompt** -> delete them. Tool schemas already render into the request; prose restating them only inflates the prefix.
  **系统提示词中的工具复述** -> 删掉。工具模式本就会渲染进请求；复述它们的散文只会撑大前缀。
- **Many or heavy tool schemas** -> tool search with `defer_loading` on rarely-used tools, so definitions load only when needed. Pays once schemas run past roughly 10K tokens (MCP servers reach that fast); below that the search step is overhead. Measurement gotcha: the token-counting endpoint rejects server tools - read billed input off a `max_tokens: 1` request instead (a paid, if tiny, model call: it sits under the standing approval rule).
  **工具模式多或重** -> 对不常用的工具用 `defer_loading` 做工具搜索，让定义按需加载。模式超过约 10K token 后才划算（MCP 服务器很快达到）；低于此值搜索步骤就是开销。度量陷阱：token 计数端点拒绝服务器工具——改从 `max_tokens: 1` 的请求读取计费输入（一次付费的、虽然极小的模型调用：受常设批准规则约束）。
- **Images and PDFs at full resolution** -> pre-downscale to what the task needs. Vision inputs are tokenized by pixel area at roughly one token per 28×28 patch, so cost scales with resolution, not information content; 1280×720 is a safe default that caps an image near 1,200 tokens (current formula - verify via the Vision docs in `shared/live-sources.md`).
  **全分辨率的图片与 PDF** -> 预先降采样到任务所需。视觉输入按像素面积 token 化，约每 28×28 补丁一个 token，因此成本随分辨率而非信息量增长；1280×720 是安全默认值，能把一张图片封顶在约 1,200 token（当前公式——经 `shared/live-sources.md` 中的 Vision 文档核实）。
- **Large tables and artifacts inlined** -> Files API plus code execution: mount the file, let the model compute in the sandbox, and only the answer enters context. Skip when there is nothing to extract or compute - the sandbox round-trip only adds tokens (and sandbox container time bills hourly beyond a free allowance).
  **内联的大表格与工件** -> Files API 加代码执行：挂载文件，让模型在沙箱中计算，只有答案进入上下文。当没有可提取或可计算的内容时跳过——沙箱往返只会增加 token（且沙箱容器时间在超出免费额度后按小时计费）。
- **Fetched web pages** -> dynamic filtering in the web fetch tool keeps boilerplate out of the context.
  **抓取的网页** -> 网页抓取工具中的动态过滤把样板文本挡在上下文之外。
- **Chained tool calls whose intermediates don't matter** -> programmatic tool calling runs the calls from code so only the filtered result enters context; its documentation reports 24% fewer input tokens on agentic search benchmarks, with a higher score.
  **中间结果无关紧要的链式工具调用** -> 程序化工具调用从代码中执行这些调用，只有过滤后的结果进入上下文；其文档报告在代理搜索基准上输入 token 减少 24%，且得分更高。
- **Broad data-dump tools** -> prefer narrow accessors (`get_policy(claim_id)` over `get_all_policies()`), and give list tools `limit`/`fields`/`date_range` parameters.
  **宽口径的数据倾倒工具** -> 优先窄口径访问器（用 `get_policy(claim_id)` 而非 `get_all_policies()`），并给列表工具加上 `limit`/`fields`/`date_range` 参数。
- **Unbounded user-supplied input** -> the token-counting endpoint as an ingestion gate (`shared/token-counting.md`): count first, then truncate, summarize, or route oversize payloads to the Files API.
  **无上界的用户提供输入** -> 用 token 计数端点作为摄取闸门（`shared/token-counting.md`）：先计数，再截断、摘要，或把超大负载路由到 Files API。
- **The prompt text itself** -> run the `prompt-audit` subcommand (`shared/prompt-audit.md`) as part of this step; its pattern tables are the reference for dated prompt text (this guide deliberately does not restate them), and its report and proposed diff fold into this workflow's deliverables. Skip when the prompt surface is small and recently audited. Prompts written for an older model make a newer one over-work: in Anthropic's support-desk evaluation, prompts carried over unaudited cost noticeably more per ticket on the newer model for no change in accuracy, and the audited versions were cheaper and, in one migration, more accurate (current figures: the Cost Optimization URL in `shared/live-sources.md`).
  **提示词文本本身** -> 在本步骤中运行 `prompt-audit` 子命令（`shared/prompt-audit.md`）；其模式表是判断提示词文本是否过时的参照（本指南刻意不复述），其报告与建议 diff 并入本工作流的交付物。提示词表面小且近期已审计时跳过。为旧模型写的提示词会让新模型过度劳动：在 Anthropic 的客服台评估中，未经审计照搬的提示词在新模型上每工单成本明显更高而准确率无变化，审计后的版本更便宜，且在一次迁移中更准确（当前数字见 `shared/live-sources.md` 中的 Cost Optimization URL）。

**Caveat for the whole section**: a smaller prefix is not automatically a cheaper task. Deferring context means the model may spend discovery turns fetching what it previously read inline. Validate against the eval - on the cookbook's workload, wrapping the manual in a tool matched the explicit-breakpoint config on cost and gave back accuracy. These changes edit `system` or `tools`, so apply them to new conversations only where the harness can, and otherwise say in the report what a conversation already in flight pays when they deploy: a restarted cache and, on models that run the preserved-thinking check, invalid thinking blocks in its history (§ 2.1). Tools declared up front with `defer_loading` are the form that stays valid.

**本节整体告示**：更小的前缀不自动等于更便宜的任务。延后上下文意味着模型可能花发现轮去取此前内联可读的内容。对照评测验证——在 cookbook 的工作负载上，把手册包进工具在成本上与显式断点配置打平，但回吐了一些准确率。这些变更会编辑 `system` 或 `tools`，因此在 harness 能做到之处只应用于新会话；做不到时，在报告中说明已在进行中的会话在其部署时要付出什么：缓存重启，以及在运行 preserved-thinking 检查的模型上历史中失效的思考块（§ 2.1）。用 `defer_loading` 预先声明的工具是保持有效的形式。

### 2.3 Agent-loop hygiene - keep long loops from compounding / 2.3 代理循环卫生——防止长循环复利放大

Only relevant when the profile shows deep loops with bulky accumulating results; short loops never trigger these and the added machinery is pure overhead.

只在画像显示存在带庞大累积结果的深循环时才相关；短循环永远不会触发这些，新增机制纯属开销。

- **Context editing** (clearing old tool uses or thinking) **is a context-window tool, not a savings lever.** Every clearing pass rewrites the cached conversation, which works against prompt caching - in the run measured for the platform docs, context editing cost more than it saved. Use it to make room in the window; set the trigger high enough that clears stay infrequent, and clear in a few large batches rather than every turn (`clear_at_least` sets the minimum a clear must remove, so a clear is skipped unless it removes enough to be worth the cache write). On models that run the preserved-thinking check it is also the safe way to clear: the check compares the conversation as sent, so server-side clearing and compaction do not count as edits, where the same trim done client-side does.
  **上下文编辑**（清除旧的工具使用或思考）**是上下文窗口工具，不是省钱杠杆。**每一轮清除都重写已缓存的对话，与提示词缓存相抵触——在平台文档实测的那次运行中，上下文编辑的成本超过了它的节省。用它来在窗口中腾地方；把触发阈值设得足够高，让清除保持低频，并以少数几个大批次清除而不是每轮清除（`clear_at_least` 设定一次清除必须移除的最小量，因此移除量不足以抵偿缓存写入时就跳过清除）。在运行 preserved-thinking 检查的模型上，它也是安全的清除方式：该检查按发送时的对话进行比较，因此服务端清除与压缩不算编辑，而客户端做同样的裁剪就算。
- **Compaction** (the server-side summarize-and-continue edit) pays only in sessions long enough to need it; on the long run measured for the platform docs it took roughly a third off the bill, and on the short run it saved nothing. Where the platform offers on-demand compaction the docs prefer it to the token-threshold form (status and platforms: the **Compaction** URL in `shared/live-sources.md`). A custom `instructions` string replaces the default summarization prompt rather than adding to it, so write it to keep the task-critical state the default would have kept; and the summarization call is billed like any other request (it shows under `usage.iterations`).
  **压缩**（服务端的摘要续跑编辑）只在长得足以需要它的会话中才划算；在平台文档实测的长运行中它把账单削掉约三分之一，在短运行中分文未省。平台提供按需压缩之处，文档更推荐它而非 token 阈值形式（状态与平台：`shared/live-sources.md` 中的 **Compaction** URL）。自定义 `instructions` 字符串是替换而非追加默认摘要提示词，因此要写成能保留默认本会保留的任务关键状态；并且摘要调用照常计费（显示在 `usage.iterations` 之下）。
- **Client-side pruning at natural boundaries - gated on models that run the preserved-thinking check**: collapsing bulky tool results to one-line extracts when a work phase completes rewrites earlier turns. On those models (the Preserved thinking page says which - URL in § 2.1) that invalidates every later thinking block, and the `preserved-thinking-migration` guide lists it as having no workaround (`shared/preserved-thinking-migration.md` § Severity tiers). There, size tool results and images before the first request that carries them and clear later through the server-side edits above; client-side, what stays valid is a compaction that replays nothing earlier (one summary message), or a prune after which the client drops every later thinking block, at the cost of that reasoning. Elsewhere the prune works as before: keep the message array byte-identical between prunes so each prune is one cold cache miss rather than a new miss every turn.
  **自然边界上的客户端剪枝——在运行 preserved-thinking 检查的模型上受限制**：在工作阶段完成时把庞大的工具结果折叠成单行摘录会重写较早的轮次。在那些模型上（Preserved thinking 页面说明是哪些——URL 见 § 2.1），这会使之后每个思考块失效，且 `preserved-thinking-migration` 指南将其列为没有变通办法（`shared/preserved-thinking-migration.md` § Severity tiers）。在那里，在首次携带工具结果与图片的请求之前就定好其规模，之后通过上述服务端编辑清除；客户端侧，保持有效的形式是不重放任何更早内容的压缩（单条摘要消息），或剪枝后客户端丢弃其后所有思考块——以损失那段推理为代价。在其他地方剪枝照旧有效：两次剪枝之间保持消息数组逐字节相同，使每次剪枝只是一次冷缓存未命中，而不是每轮新添一次未命中。
- **Subagents for self-contained bulky steps**: a nested loop absorbs its own heavy tool results and hands back one line, optionally on a cheaper model. Skip when the deciding model needs the intermediate context to judge well - and note the subagent starts a fresh prefix with no cache shared with the parent.
  **用子代理处理自成体系的庞大步骤**：嵌套循环吸收自己的重型工具结果，只交回一行，可选更便宜的模型。当决策模型需要中间上下文才能判断好时跳过——并注意子代理以全新前缀启动，与父级不共享缓存。

### 2.4 Output tokens / 2.4 输出 token

- **`max_tokens` is a backstop, not a tuning knob.** The model never sees it; hitting it cuts the response off mid-thought with `stop_reason: "max_tokens"`. In Anthropic's coding runs a tight cap cut off a large share of attempts, few of them solved - capped runs spent less per attempt and bought proportionally fewer solves, so cost per solved task didn't improve. Set it to 64,000 for agentic work (also the published starting point at `xhigh` or `max` effort; up to the model's maximum output, listed in `shared/models.md`, where a single cut-off attempt is costly), stream responses that large, and treat `stop_reason: max_tokens` as a failed attempt rather than retrying at the same cap. Before setting it, check the target model's section of the Effort page (the **Effort Parameter** URL in `shared/live-sources.md`): a `max_tokens` value recommended there replaces the 64,000.
  **`max_tokens` 是保险丝，不是调参旋钮。**模型看不到它；触到它就以 `stop_reason: "max_tokens"` 把响应在思路中途切断。在 Anthropic 的编码实验中，紧的上限切掉了很大一部分尝试，其中解出的很少——受限运行每次尝试花得更少，但按比例买到的解也更少，因此每解出任务的成本并未改善。代理型工作设为 64,000（也是 `xhigh` 或 `max` effort 公布的起始点；上至模型最大输出，见 `shared/models.md`，适用于单次被切断的尝试代价高昂之处），以流式传输这么大的响应，并把 `stop_reason: max_tokens` 当作一次失败的尝试，而不是在同一上限下重试。设置之前，查 Effort 页面中目标模型的小节（`shared/live-sources.md` 中的 **Effort Parameter** URL）：该处推荐的 `max_tokens` 值取代 64,000。
- **To shorten visible responses**, specify the exact output shape in the prompt, ideally with an example. To shorten reasoning, that is the effort parameter (§ 2.6) - not `max_tokens`, and not switching thinking off unchecked: some current models reject a disabled or budgeted `thinking` configuration with a 400, so read SKILL.md -> Thinking & Effort for the target model before proposing any `thinking` change. Where thinking is always on, effort is the main control over thinking spend, and a workload that ran without thinking can bill more output tokens after moving there.
  **要缩短可见回复**，在提示词中明确规定确切的输出形状，最好附一个示例。要缩短推理，那是 effort 参数（§ 2.6）的事——不是 `max_tokens`，也不是不加检查就关掉思考：一些现行模型会以 400 拒绝被禁用或带预算的 `thinking` 配置，因此在提出任何 `thinking` 变更前先读 SKILL.md -> Thinking & Effort 中目标模型的部分。在思考常开之处，effort 是控制思考花费的主要手段，而一个原本不思考的工作负载迁移过去后可能开出更多的输出 token 账单。
- **Stop sequences as content-aware early exits**: register a sentinel the model emits when it cannot proceed (for example `<CANNOT_REVIEW>`), so it stops instead of spending tokens explaining.
  **把停止序列用作内容感知的提前退出**：注册一个模型在无法继续时发出的哨兵串（例如 `<CANNOT_REVIEW>`），让它停下来，而不是花 token 去解释。

### 2.5 Batch processing / 2.5 批处理

50% off **every token in the request, including cache reads and writes** - the discounts stack. The second-largest free lever after caching for unattended agent work - evaluation runs, backfills, scheduled jobs.

**请求中的每个 token 都打五折，包括缓存读与写**——折扣可叠加。对于无人值守的代理工作——评测运行、回填、定时作业——这是仅次于缓存的第二大免费杠杆。

- Results arrive asynchronously within 24 hours; that window is an expiry, not an SLA. Keep user-facing work synchronous.
  结果在 24 小时内异步到达；该窗口是过期时限，不是 SLA。面向用户的工作保持同步。
- Batch requests are single-shot - no mid-batch tool loop. A tool loop can sometimes be flattened into one batchable request by pre-fetching its inputs up front; in the cookbook's worked example that ran at roughly half the interactive config's cost, but it is an architecture decision, not a parameter - it changes how the model reasons (the flattened run held its pass rate less firmly), and cache hits inside a concurrent batch are best-effort.
  批处理请求是一次性的——没有批中途的工具循环。工具循环有时可以预先抓取其输入，从而压平成一个可批处理的请求；在 cookbook 的示例中它以约为交互式配置一半的成本运行，但这是架构决策，不是参数——它改变模型的推理方式（压平后的运行保持通过率的能力更弱），且并发批处理内的缓存命中是尽力而为。
- Not available for Managed Agents sessions (current mechanics and availability: the **Batch Processing** URL in `shared/live-sources.md`).
  Managed Agents 会话不可用（当前机制与可用性：`shared/live-sources.md` 中的 **Batch Processing** URL）。

### 2.6 Effort and budgets - the first tradeoffs / 2.6 effort 与预算——最初的权衡项

From here down, every lever trades capability for cost. Sweep on the eval, one change at a time.

从这里往下，每根杠杆都在用能力换成本。在评测上扫描，一次只改一处。

- **Sweep effort before touching the model** (on models that expose an effort parameter - check `shared/models.md` or the **Effort Parameter** URL in `shared/live-sources.md`). Use only the levels the model accepts with the `thinking` setting the code sends - some settings reject the top levels with a 400 (read that model's whole row in SKILL.md -> Thinking & Effort, not only its Effort levels cell). Effort scales thinking and tool-call depth without changing the model. Test each level in a separate session - changing top-level effort mid-session invalidates the cache and distorts the comparison. Sweep mechanics that keep the comparison honest:
  **在动模型之前先扫 effort**（在暴露 effort 参数的模型上——查 `shared/models.md` 或 `shared/live-sources.md` 中的 **Effort Parameter** URL）。只使用模型在代码所发 `thinking` 设置下接受的档位——某些设置会以 400 拒绝最高档（读 SKILL.md -> Thinking & Effort 中该模型整行，而不只是它的 Effort levels 单元格）。effort 在不更换模型的情况下缩放思考与工具调用深度。每个档位在独立会话中测试——会话中途改顶层 effort 会使缓存失效并扭曲比较。保持比较诚实的扫描要点：
  - Cells are byte-identical except `output_config.effort`; same model throughout. Complete every sample request at one setting before starting the next, in a stable order, so cache reads are comparable across settings - and if the cache meters still differ materially between settings, say so and weight the read toward output-side cost.
    除 `output_config.effort` 外各单元格逐字节相同；全程同一模型。一个设置下跑完所有样本请求再开始下一个设置，顺序稳定，使缓存读取在各设置间可比——若各设置间缓存计量仍有实质差异，如实说明，并把解读权重偏向输出侧成本。
  - Include a hard case the user knows about: curves are flattest on easy tasks, and the hard tail is where higher effort earns its cost.
    纳入一个用户知道的困难用例：曲线在简单任务上最平，而困难的尾部才是更高 effort 赚回其成本的地方。
  - **Side-effect gate**: if replaying a sample request executes tools that mutate real state, point the replay at a scratch environment or stub those tools first; a sweep is never worth a production mutation. If that isn't possible, sweep only the requests that are safe to replay and say so.
    **副作用闸门**：如果重放样本请求会执行改动真实状态的工具，把重放指向临时环境，或先把那些工具替换为桩；扫描永远不值得换来一次生产环境改动。做不到时，只扫可以安全重放的请求，并如实说明。
  - Read the curve as flat (the lower setting does this workload's work), steep (the higher setting is earning its cost - now a measured number rather than a fear), or mixed (name which tasks flipped - those are the candidates for the re-run-failures policy below). Differences of a task or two of pass rate, or cents of mean cost, are within noise on single runs; the remedy is repeat trials at the settings in contention, offered with their cost.
    把曲线读成平的（较低设置就能干完该工作负载的活）、陡的（较高设置正在赚回其成本——如今是实测数字而非担忧）、或混合的（点名哪些任务翻转了——它们是下文重跑失败策略的候选对象）。通过率差一两个任务、或平均成本差几分钱，在单次运行中都在噪声之内；补救办法是在有争议的设置上重复试验，并连同其成本一并报出。
  - The curve is per-workload *and* per-model. Keep the sample and the outcome check where the report says they live, and re-sweep after a model migration, a major prompt change, or a workload shift. Default effort differs by model, and the token allocation behind each level can change between models, so after a migration run a fresh sweep from the new model's own default (SKILL.md -> Thinking & Effort) rather than carrying a level over.
    这条曲线因工作负载*且*因模型而异。把样本与结果校验保存在报告所述的位置，并在模型迁移、重大提示词变更或工作负载变化之后重新扫描。默认 effort 因模型而异，各档位背后的 token 分配也可能随模型改变，因此迁移之后要从新模型自己的默认值（SKILL.md -> Thinking & Effort）起跑一轮全新扫描，而不是把档位直接带过去。

  What to expect by workload shape:

  按工作负载形态可以预期：

  - Research and knowledge work: nearly flat curves - in Anthropic's runs `low` gave up a few points for a third to a half off cost per task, `medium` matched `high`'s accuracy for noticeably less, and `high` bought nothing measurable over `medium`. Lower effort is also faster.
    研究与知识型工作：曲线近乎平坦——在 Anthropic 的实验中，`low` 用几个百分点的让步换来每任务成本降三分之一到一半，`medium` 以明显更低的成本追平 `high` 的准确率，而 `high` 相对 `medium` 没有买到可测量的提升。更低的 effort 也更快。
  - Long-horizon coding: a real tradeoff - each step down gave up a few points of pass rate (single digits in the published runs) for a substantially lower cost per task, and the highest settings bought little for a large multiple of the cost.
    长程编码：真实的权衡——每往下一步都以几个百分点的通过率（公开实验中为个位数）换来显著更低的每任务成本，而最高档位以数倍的成本只买到很少。
  - Hard work does not automatically need high effort: on one deep-research benchmark the published scores were nearly level across `low`, `medium`, and `high` while cost per task climbed. Only the sweep says whether more effort still buys accuracy on this workload.
    难的活不会自动需要高 effort：在某深度研究基准上，公开分数在 `low`、`medium`、`high` 之间几乎持平，而每任务成本一路攀升。更多 effort 在该工作负载上是否还买得到准确率，只有扫描能回答。
  - Current curves per model: the Cost Optimization URL in `shared/live-sources.md`.
    各模型的当前曲线：`shared/live-sources.md` 中的 Cost Optimization URL。
- **Re-run failures at higher effort** - when the workload has a usable failure signal (tests, a checker, a validator). Run everything at a low setting and re-run only the failures at a higher one: in Anthropic's coding runs this held the pass rate of running everything at the higher setting, or slightly beat it, for a little over half the cost, counting the failed cheap attempts. Use this for the saving, not the lift, and price in the checker and the doubled wall-clock on failures.
  **以更高 effort 重跑失败项**——在工作负载有可用的失败信号时（测试、检查器、校验器）。全部以低设置运行，只把失败项以更高设置重跑：在 Anthropic 的编码实验中，这保住了全部以高设置运行的通过率、或略微胜出，而成本略多于一半（把失败的廉价尝试算在内）。用它来省钱，而不是提分，并把检查器成本与失败项上翻倍的墙钟时间计进去。
- **Tell the model that time matters, and show it the elapsed time**, when wall-clock time matters: the published runs finished sooner and cost less per task for a small score cost. Append the clock as a new message after the newest turn (a mid-conversation system message where the model accepts one), never by rewriting `system` - recipe and figures: the Cost Optimization URL in `shared/live-sources.md`. Validate it like any other tradeoff.
  **告诉模型时间很重要，并把已用时间给它看**——在墙钟时间要紧时：公开实验中它更早完成、每任务成本更低，只付出很小的分数代价。把时钟作为新消息追加在最新一轮之后（模型接受处用会话中途系统消息），绝不通过改写 `system`——配方与数字见 `shared/live-sources.md` 中的 Cost Optimization URL。像其他任何权衡项一样验证它。
- **Task budgets** (the model sees the budget and paces itself - this is the budget control that saves money): set from the loop's 90th-percentile token usage, then tighten. The budget is advisory - it steers the model rather than stopping it - so verify adherence on the workload. Measured on coding: pass rate fell a few points as the budget tightened while cost per task fell by a much larger share - budgets bought efficiency at a price in pass rate that grows as they tighten. Budgets below the documented floor are rejected; very tight budgets can produce refusal-like behavior; set the budget once on the first request - a mid-task change invalidates the cache. Check model availability before wiring it in (beta, and not available on every current model) - parameter shape, the streaming requirement, and supported models are in this skill's SKILL.md -> Task Budgets (Quick Reference) and `shared/model-migration.md` -> Task Budgets.
  **任务预算**（模型看得见预算并自我调节——这才是真正省钱的预算控制）：从循环的第 90 百分位 token 用量起设，然后收紧。预算是建议性的——它引导模型而非阻止模型——因此要在工作负载上核实依从性。编码任务上的实测：预算收紧时通过率掉几个点，而每任务成本降幅大得多——预算买到效率，代价是随收紧而增大的通过率损失。低于文档下限的预算会被拒绝；过紧的预算可能产生类似拒答的行为；预算只在第一条请求上设一次——任务中途更改会使缓存失效。接线之前先查模型可用性（beta，并非每个现行模型都有）——参数形状、流式要求与受支持模型见本技能的 SKILL.md -> Task Budgets (Quick Reference) 与 `shared/model-migration.md` -> Task Budgets。
- **Backstops that don't save per-task money but cap the damage**: a Managed Agents session budget is a hard dollar stop; a workspace spend limit is the final backstop on the whole workspace.
  **不省每任务的钱、但能封顶损失的后备手段**：Managed Agents 会话预算是硬性美元止损；工作区支出限额是整个工作区的最终后备。

### 2.7 Model selection - last, deliberately / 2.7 模型选择——刻意放在最后

Model choice constrains the intelligence ceiling, which is why it comes after every lever that doesn't. (The exception is when the project has an eval and you are running the hillclimb loop: there `shared/evals/cost-hillclimb.md` walks model x effort early, because an eval can detect the case where a stronger model at lower effort is the cheaper cell. Without an eval that case is invisible and a model swap stays the riskiest change - keep it last.)

模型选择约束智能上限，这正是它排在所有不约束智能上限的杠杆之后的原因。（例外是项目有评测且你在跑爬山回路时：那里 `shared/evals/cost-hillclimb.md` 会提前走模型 x effort，因为评测能发现"更强模型配更低 effort 才是更便宜单元格"的情形。没有评测时该情形不可见，换模型仍是最危险的变更——留在最后。）

- **Check for an upgrade before a step-down.** If the workload runs a model a generation or two behind the current one in its tier, the cheapest lever can be the model string: in the published runs the cheapest upgrade was often the new model at a lower effort setting. That direction is not guaranteed - on one research benchmark the same upgrade cost more per task - so measure it on this workload before assuming it saves. It is still a migration - run the `migrate` subcommand for the breaking changes, re-run the prompt audit (§ 2.2), and re-sweep effort from the new model's default.
  **降档之前先查升级。**如果工作负载运行的模型比所在层级当前型号落后一两代，最便宜的杠杆可能就是模型字符串：公开实验中最便宜的升级往往是新模型配更低的 effort 档。这个方向并无保证——在某研究基准上同样的升级每任务反而更贵——所以在假定它能省钱之前先在该工作负载上测。它仍是一次迁移——运行 `migrate` 子命令处理破坏性变更，重跑提示词审计（§ 2.2），并从新模型的默认值重新扫 effort。
- **Price candidates in cost per completed task on your own traffic**, including the larger model at reduced effort - per-token price lists do not predict the ranking. In Anthropic's runs the ranking flipped by workload: the frontier model at `low` effort out-solved a smaller model for less per solved task on one benchmark, and on a coding subset that the frontier model and the model one tier below it both largely saturate, the lower of the two at its default matched the frontier model for a fraction of the cost. Which model to start from is not this guide's call: take the current starting-point recommendation and per-task comparisons from the Cost Optimization URL in `shared/live-sources.md`, and current rates from the Pricing URL there. At the other end, the smallest model answered knowledge questions at a fraction of the cost per question of that same model one tier below the frontier, with markedly lower accuracy - it fits high-volume work with checkable outputs, not long agentic loops.
  **在自己的流量上以每完成任务的成本为候选模型定价**，包括降档 effort 下的更大模型——每 token 价目表预测不了排名。在 Anthropic 的实验中，排名随工作负载翻转：在一个基准上，前沿模型以 `low` effort 以更低的每解出任务成本胜过更小的模型；而在前沿模型与其下一档模型都基本饱和的编码子集上，两者中较低档的模型以其默认设置以零头成本追平前沿模型。从哪个模型起步不由本指南决定：从 `shared/live-sources.md` 中的 Cost Optimization URL 取当前起点建议与每任务对比，费率从那里的 Pricing URL 取。在另一端，最小模型以不足前沿模型下一档模型每问题成本零头的价格回答知识问题，但准确率明显更低——它适合产出可核查的高量工作，不适合长代理循环。
- **Price the tail, not the median.** Compare models on the hardest tenth of the workload: on the typical task every model looks similar and the cheapest looks best, but the bill is decided by the tasks the cheap model fails - and the tail is where the money goes even when nothing fails (on one 20-problem research run, two problems carried 43% of the spend).
  **给尾部定价，而不是给中位数定价。**在工作负载最难的那十分之一上比较模型：在典型任务上每个模型看起来都差不多、最便宜的看起来最好，但账单由廉价模型失败的任务决定——而且即使什么都没失败，钱也花在尾部（在一次 20 题的研究运行中，两道题占掉了 43% 的支出）。
- **The stepping-down method**: sweep effort on the current model first; if `low` passes the eval, drop one model tier, **confirm which parameters and effort levels the target tier supports** (SKILL.md -> Thinking & Effort) - including the `thinking` configuration the current code sends, which the target may reject - reset effort to that tier's default - not a hardcoded level; the default and the supported range vary by model - and re-sweep down from there (on a tier without `effort` support, evaluate at its single default only). One notch at a time, against the eval - and when there is no cheaper tier, the lever is exhausted; say so rather than inventing a step. Current model lineup and discovery: `shared/models.md`; for model-swap mechanics and per-target breaking changes, the `migrate` subcommand (`shared/model-migration.md`). Switch between conversations, not inside one: a mid-conversation switch cold-starts the cache and, where the new model cannot read the old one's thinking blocks, runs on without that reasoning (`shared/preserved-thinking-migration/causes.md` § Switching models mid-conversation).
  **逐级下调法**：先在当前模型上扫 effort；若 `low` 通过评测，就降一个模型档位，**确认目标档位支持哪些参数与 effort 档位**（SKILL.md -> Thinking & Effort）——包括当前代码发送的 `thinking` 配置，目标可能拒绝它——把 effort 重置为该档位的默认值——不是硬编码的档位；默认值与支持范围因模型而异——再从那里向下重新扫描（在没有 `effort` 支持的档位上，只用其唯一默认值评估）。一次一档，对照评测——当没有更便宜的档位时，这根杠杆就到头了；如实说明，而不是硬造一步。当前模型阵容与查询方式：`shared/models.md`；换模型的机制与各目标的破坏性变更：`migrate` 子命令（`shared/model-migration.md`）。在会话之间切换，不要在会话内切换：会话中途切换会让缓存冷启动，且在新模型读不到旧模型思考块之处，没有那段推理也照样运行（`shared/preserved-thinking-migration/causes.md` § Switching models mid-conversation）。
- **Two models can beat one, in exactly two measured shapes** - both are architecture changes; validate like one, and any design that routes a single conversation between models inherits the model-switch costs above:
  **两个模型可以胜过一个模型，实测有效的恰好是两种形态**——两者都是架构变更；按架构变更来验证，而任何在模型之间路由单个会话的设计都继承上述换模型成本：
  - **Advisor** (a cheaper executor runs the loop and consults a frontier model on hard decisions): pays when the capability gap between the two models is wide and the executor actually consults. The consult rate is the fragile variable - lowering effort can drop a pairing from consulting on most tasks to almost none, and then it scores below the executor alone - and gating the consult well requires a cheap signal; asking the executor to recognize the hard cases itself demands the very judgment it's missing. Benchmark first: on Anthropic's coding benchmark the flagship pairing landed about on the executor model's own effort curve - the advisor bought roughly what more effort did - so sweep effort and price the stronger model alone before adding the advisor.
    **顾问**（更便宜的执行者跑循环，在困难决策上咨询前沿模型）：当两模型能力差距大且执行者真的会去咨询时才划算。咨询率是脆弱的变量——降低 effort 可能让一对组合从大多数任务都咨询跌到几乎不咨询，此时其得分低于执行者单独运行——而把咨询闸门调好需要一个廉价信号；让执行者自己识别困难案例，要的恰恰是它缺的那份判断力。先跑基准：在 Anthropic 的编码基准上，旗舰组合大致落在执行者模型自己的 effort 曲线上——顾问买到的约等于更多 effort 买到的——因此在加顾问之前，先扫 effort、单独给更强模型定价。
  - **Orchestrator** (a frontier model plans and delegates bulk work to cheaper workers): buys something only when there is bulk to hand off - many independent pieces, ideally too many for one context window. On work larger than any context window it cost about half as much as the frontier model solo, for a lower score; on routine search work it paid as tail insurance (about half the average cost, less still at the expensive tail) but reversed on the harder full set. When the work is one dependent chain, or fits in a single context, the orchestrator pays for a plan, a handoff, and a merge that a single model gets for free - in every such case measured, the coordinator's model alone at lower effort came out ahead.
    **编排器**（前沿模型规划并把批量工作委派给更廉价的工人）：只有在有批量可交出——许多独立件，理想情况下多到一个上下文窗口装不下——时才买到东西。在大于任何上下文窗口的工作上，它的成本约为前沿模型单干的约一半，得分更低；在常规检索工作上它作为尾部保险是划算的（约一半的平均成本，昂贵的尾部还更低），但在更难的完整集上反而亏了。当工作是一条依赖链，或能装进单个上下文时，编排器为规划、交接与合并付费，而单个模型免费就有这些——在实测的每一个此类案例中，协调者自己的模型配更低 effort 都更划算。

## Step 3: Apply, measure, keep or revert - one lever at a time / 步骤 3：应用、测量、保留或回退——一次一根杠杆

- Work down the ranked shortlist to decide which levers earn a diff - but **apply shortlisted levers in the § 2 order** (free wins -> effort/budgets -> model), not in savings-rank order: the ranking decides inclusion and where the eval budget goes; the § 2.x numbering decides sequence. Each lever that earns a place becomes **its own diff** (one lever per diff, so a revert is clean and effects attribute), applied and then measured: re-run the eval covering that lever's traffic class, and read pass rate and cost per task together against the previous kept configuration (the baseline for the first lever only). A lever that saves money and gives back accuracy is not an optimization - revert it and record why. A lever touching a path no eval covers cannot be validated by the eval you have: a free win there is measured on cost only, and said so; a tradeoff there stays an unapplied proposal (Step 0.2's marking rule).
  沿排好序的候选清单向下走，决定哪些杠杆配得到 diff——但**入清单的杠杆按 § 2 顺序应用**（免费收益 -> effort/预算 -> 模型），而不是按节省排名顺序：排名决定取舍与评测预算的去向；§ 2.x 编号决定顺序。每根赢得位置的杠杆成为**它自己的 diff**（一个 diff 一根杠杆，这样回退干净、效果可归因），先应用后测量：重跑覆盖该杠杆流量类别的评测，把通过率与每任务成本放在一起、对照上一个保留的配置来读（只有第一根杠杆对照基线）。省钱但回吐准确率的杠杆不是优化——回退它并记录原因。触及任何评测都不覆盖路径的杠杆无法用现有评测验证：那里的免费收益只按成本测量，并如实说明；那里的权衡项保持为未应用的建议（步骤 0.2 的标注规则）。
- **Ask for the measurement budget once, not per run.** Present the validation plan with its total expected runs and cost - an effort sweep is several configurations at several trials each - and get it approved as a budget; within an approved budget, individual runs need no fresh approval. A shadow-run on live traffic roughly doubles production spend while it runs: it is its own approval.
  **测量预算只申请一次，而不是每次运行都申请。**把验证计划连同预期的总运行次数与总成本一并呈报——一次 effort 扫描是数个配置各若干次试验——作为预算取得批准；在已批准预算内，单次运行无需再批。对线上流量的影子运行在其运行期间大约使生产支出翻倍：它需要单独批准。
- **Never keep or revert on a one-case swing.** Repeat trials within the approved budget until the decision clears the noise. The published bar - around fifty cases and at least five trials per configuration - is the standard for the production cutover; a smaller project eval is acceptable for per-lever decisions when trials are repeated. And validating a caching diff needs a warm cache: run the sample sequentially and measure from the second request on, or the 1.25x writes dominate and the free win reads as a regression.
  **绝不在单例波动上做保留或回退。**在已批准预算内重复试验，直到决策超出噪声。公开标准——约五十个用例、每个配置至少五次试验——是生产切换的标准；规模更小的项目评测在重复试验的前提下可用于每杠杆决策。验证缓存 diff 需要热缓存：顺序运行样本并从第二个请求起测量，否则 1.25 倍的写入占主导，免费收益会被读成退化。
- **When the user can provide no outcome check at all**: free wins become cost-only-measured diffs (or proposals, if no spend is approved), tradeoffs stay unapplied proposals carrying the published expectations, and offer a manual before/after spot-check of a handful of real answers - the user's review gates free wins, never a tradeoff. For an effort sweep specifically, a cost-only run is still worth offering: the same matrix with no pass-rate column, reporting per task the outputs at each setting laid side by side - exactly what the user needs in front of them to judge quality themselves. State plainly in the report which mode ran, and do not invent a grader to fill the gap. If the application doesn't log usage, adding `response.usage` logging is itself a free-win diff, and it is the measurement channel for everything after it when there is no Admin API key.
  **当用户完全提供不了结果校验时**：免费收益变成只测成本的 diff（未批准支出时则为建议），权衡项保持为携带公开预期的未应用建议，并提出对少数真实答案做人工前后抽检——用户审查约束免费收益，永远不放行权衡项。对 effort 扫描而言，只测成本的运行仍值得提供：同一矩阵去掉通过率列，按任务并排呈现各设置的输出——这正是用户自行判断质量所需的案头材料。在报告中如实说明运行的是哪种模式，不要发明一个评分器来填补空缺。如果应用不记录用量，添加 `response.usage` 日志本身就是一份免费收益 diff，并且在没有 Admin API 密钥时它是此后一切测量的通道。
- **Minimal eval recipe** - the cheapest thing that clears a tradeoff lever, so "needs an eval" is a next step rather than a dead end. Offer to build it with the user:
  **最小评测配方**——能放行一根权衡杠杆的最廉价方案，让"需要评测"成为下一步而不是死胡同。提出与用户一起搭建：
  - **Inputs**: a fixed set of ~20-30 real requests pulled from production logs or written by the user - enough for per-lever keep/revert decisions (the ~50-case bar above is for the final production cutover). Freeze them; every config runs the identical set.
    **输入**：一组固定的约 20-30 条真实请求，取自生产日志或由用户编写——足以支持每杠杆的保留/回退决策（前述约 50 例标准是给最终生产切换的）。冻结它们；每个配置跑完全相同的集合。
  - **Judgment per output**: whichever is cheapest for the workload - golden answers to diff against, a short rubric the user scores each output on, or an automated checker (tests pass, JSON validates, required fields present). A model-graded judge is acceptable when nothing cheaper exists, but it is itself an approved API spend.
    **每输出的判定**：取对该工作负载最廉价的——可 diff 的标准答案、用户给每个输出打分的简短量规、或自动化检查器（测试通过、JSON 有效、必填字段齐全）。在没有更廉价方案时，模型评分裁判可以接受，但它本身就是一笔已批准的 API 支出。
  - **Runner**: a script that runs the frozen inputs through one config, records each output plus `response.usage`, and reports pass rate and cost per task. Each config is one invocation; the sweep is a loop over configs.
    **运行器**：一个脚本，把冻结输入跑过一个配置，记录每个输出及 `response.usage`，并报告通过率与每任务成本。每个配置一次调用；扫描就是遍历配置的循环。
  - **Cost and approval**: estimate it (inputs × configs × baseline cost per task) and get the user's go-ahead before running - this is real API spend under the standing approval rule.
    **成本与批准**：先估算（输入数 × 配置数 × 每任务基线成本）并在运行前取得用户同意——这是常设批准规则下的真实 API 支出。
- Keep-or-revert is decided locally, on the eval evidence. Shadow-run the winning configuration on live traffic before cutover, keep the eval running after it, and confirm the savings in the usage and cost reports **after** cutover - only where the traffic is attributable (Step 1's shared-key caveat applies to the confirmation read too).
  保留还是回退在本地依据评测证据决定。切换前让获胜配置在线上流量影子运行，切换后让评测继续跑，并在切换**之后**在用量与成本报告中确认节省——只在流量可归因之处（步骤 1 的共享密钥告诫同样适用于这次确认读取）。
- Expect most levers not to fit any given workload. On the cookbook's worked example, most didn't earn a place - tool schemas too small for tool search, loops too short for editing or compaction, no numeric work for code execution - and the levers that came closest on cost each gave back a correct answer. The profile from Step 1 exists so optimization isn't blind.
  预期大多数杠杆不适合任一给定工作负载。在 cookbook 的示例中，大多数没赢得位置——工具模式太小用不上工具搜索、循环太短用不上编辑或压缩、没有数值计算用不上代码执行——而在成本上最接近可行的那几根杠杆各自回吐了一个正确答案。步骤 1 的画像之所以存在，就是为了让优化不是盲目的。
- Plot configurations as score versus cost per task and take the Pareto frontier - that is what the cutover decision reads from.
  把各配置画成得分对每任务成本的散点并取帕累托前沿——切换决策就从那里读出。

## Workload shape -> lever / 工作负载形态 -> 杠杆

Adapted from the cookbook's takeaways table, for mapping a profile to levers (row 1's watch-out is extended):

改编自 cookbook 的要点表，用于把画像对应到杠杆（第 1 行的注意事项有所扩展）：

| Where the cost is | Reach for | Skip it or watch out when |
|---|---|---|
| Same system prompt and tools re-billed on every call | Prompt caching with auto first, then an explicit breakpoint on the static prefix when many independent conversations share it or prefix layers change at different rates, and the 1-hour TTL or a keep-alive where gaps run past five minutes (§ 2.1) | Anything dynamic sits above the breakpoint - move that content into the user turn. And a cache that already reads well needs nothing: concurrent-batch misses (§ 2.5) aren't breakers, and a 1-hour TTL doesn't reach calls that are hours apart |
| Large reference document in every prompt | Move it behind a tool or skill | Each call needs most of the document rather than a section, or the eval shows misses on cases that hinge on rules the model has to go looking for |
| Many or heavy tool schemas | Tool search with `defer_loading` | Under roughly 10K schema tokens, where the search step is overhead |
| Images, PDFs, or large files in context | Downscale images to what the task needs, and use the Files API plus code execution for tables and PDFs | There is nothing to extract or compute so the sandbox only adds tokens |
| Unbounded user-supplied input | Token counting as an ingestion gate | |
| Bulky results piling up across a long loop | Context editing or compaction server-side; a client-side prune at natural boundaries, gated on models that run the preserved-thinking check (§ 2.3) | Loops are short or the cleared content is still needed, and note that every edit breaks the cache from that point - and a client-side edit of earlier turns also invalidates later thinking on models that run the preserved-thinking check |
| One self-contained step with bulky intermediates | Subagent, optionally on a cheaper model | The deciding model needs that intermediate context to judge well |
| Long visible responses | Specify the output shape with an example, with `max_tokens` as a backstop and a stop-sequence sentinel for early exits | |
| Thinking and tool calls dominate, and the eval has headroom | Lower `effort` first; check for a newer model in the tier (§ 2.7), then drop a model tier and re-sweep effort | Always a direct capability trade, so step down one notch at a time against the eval |
| Mostly routine cases with a few hard ones | Advisor tool on a cheaper driver | There is no cheap signal to gate the consult, leaving the driver to spot hard cases itself |
| No one is waiting on the response | Batch API, flattening a tool loop into one request by pre-fetching its inputs if you have to | A user is waiting, or when flattening changes how the model reasons |

| 成本在哪里 | 用什么 | 何时跳过或当心 |
|---|---|---|
| 每次调用都重复计费同样的系统提示词与工具 | 先自动提示词缓存；当许多独立会话共享静态前缀或前缀各层变化速率不同时，在静态前缀上加显式断点；间隔常超过五分钟处用 1 小时 TTL 或保活（§ 2.1） | 断点上方有任何动态内容——把那些内容挪进用户轮。已经读得很好的缓存什么都不需要：并发批处理未命中（§ 2.5）不是破坏者，而 1 小时 TTL 也够不着相隔数小时的调用 |
| 每个提示词都带大型参考文档 | 把它挪到工具或技能之后 | 每次调用需要文档的大部分而非一节，或评测显示在依赖模型须自行寻找规则的用例上出现失误 |
| 工具模式多或重 | 用 `defer_loading` 做工具搜索 | 模式 token 约在 10K 以下，此时搜索步骤是开销 |
| 上下文中的图片、PDF 或大文件 | 图片降采样到任务所需；表格与 PDF 用 Files API 加代码执行 | 没有可提取或可计算的内容，沙箱只会增加 token |
| 无上界的用户提供输入 | 用 token 计数作摄取闸门 | |
| 庞大结果在长循环中不断堆积 | 服务端上下文编辑或压缩；自然边界上的客户端剪枝，在运行 preserved-thinking 检查的模型上受限制（§ 2.3） | 循环短或被清除内容仍有用；注意每次编辑都从该点起破坏缓存——且在运行 preserved-thinking 检查的模型上，对较早轮次的客户端编辑还会使其后的思考失效 |
| 单个自成体系步骤带庞大中间结果 | 子代理，可选更便宜的模型 | 决策模型需要那段中间上下文才能判断好 |
| 可见回复过长 | 用示例规定输出形状，`max_tokens` 作后备，停止序列哨兵作提前退出 | |
| 思考与工具调用占主导且评测有余量 | 先降 `effort`；查层级内有无更新模型（§ 2.7），再降一档模型并重扫 effort | 永远是直接的能力交换，因此对照评测一次只降一档 |
| 常规案例居多、少数困难 | 更廉价驱动模型上配顾问工具 | 没有廉价信号为咨询设闸门，只能靠驱动模型自己发现困难案例 |
| 没有人在等响应 | Batch API，必要时预先抓取输入把工具循环压平成一个请求 | 有用户在等，或压平会改变模型推理方式时 |

## Step 4: Deliverables / 步骤 4：交付物

1. **The cost profile and plan**: the Step 0 assumptions (scope, quality bar, baseline), the Step 1 token profile, and the levers chosen with the measured expectation each one carries - plus the levers deliberately skipped and why, so the next person doesn't re-litigate them. Label the shortlist table as ranked by savings ceiling, not application order, so it can't be misread as the diff sequence.
   **成本画像与计划**：步骤 0 的假设（范围、质量标准、基线）、步骤 1 的 token 画像，以及所选杠杆及各自携带的实测预期——外加刻意跳过的杠杆及原因，免得下一个人重新争论一遍。给候选清单表标注"按节省上限排序、非应用顺序"，以免被误读为 diff 顺序。
2. **The changes**: one diff per lever so effects attribute - applied and measured (expected versus measured cost per task, pass rate held or not) where the user approved the runs; left as proposals carrying their expected savings and published quality cost where they didn't, or where a tradeoff lever still needs an eval. When nothing cleared the ranking floor, this deliverable is "no changes recommended" - a successful outcome; say it plainly rather than manufacturing a lever.
   **变更**：一根杠杆一个 diff，让效果可归因——用户批准运行之处，已应用并测量（预期与实测的每任务成本对比、通过率是否守住）；未批准之处、或权衡杠杆仍需评测之处，保留为携带预期节省与公开质量代价的建议。当没有任何东西过得了排名底线时，这项交付物就是"不建议变更"——这是一个成功结果；如实说出，而不是硬造一根杠杆。

**Report skeleton** (section order and required columns - keep the rest flexible):

**报告骨架**（小节顺序与必需列为固定项——其余保持灵活）：

- **Scope / quality bar / baseline / platform / pages fetched, with dates** (Step 0 assumptions)
  **范围 / 质量标准 / 基线 / 平台 / 已抓取页面（附日期）**（步骤 0 假设）
- **Token profile** (Step 1)
  **token 画像**（步骤 1）
- **Ranked shortlist** - table columns: `Lever | Type (free win / tradeoff) | Savings ceiling | Data source (measured / usage logs / code estimate)`. Ceiling is in the unit tier the data supports (Step 1 -> Rank the levers). Caption the table "ranked by savings ceiling, not application order."
  **排序列表**——表列：`Lever | Type (free win / tradeoff) | Savings ceiling | Data source (measured / usage logs / code estimate)`。上限以数据支持的单位档呈现（步骤 1 -> Rank the levers）。给表加注"按节省上限排序、非应用顺序"。
- **Proposed changes** - one diff per lever, numbered in § 2 application order (free wins -> effort/budgets -> model), each tagged *applied and measured* / *proposed* / *needs an eval*
  **建议变更**——一根杠杆一个 diff，按 § 2 应用顺序编号（免费收益 -> effort/预算 -> 模型），每项标注*已应用并测量* / *建议* / *需要评测*
- **Levers skipped** and why (including any dropped for platform availability)
  **已跳过杠杆**及原因（包括因平台可用性被剔除的）
- **Next step / approvals needed** - measurement budget ask, eval prerequisite, or "no changes recommended"
  **下一步 / 所需批准**——测量预算申请、评测前置条件、或"不建议变更"

## Sources and live references / 来源与实时引用

The measured results above come from two published Anthropic sources (and the Admin API facts in Step 1 from a third); Step 0 requires the first and the Pricing page before anything is sized; the cookbook and Admin API docs are fetched when the user needs the full write-ups or schemas:

上文的实测结果来自两个 Anthropic 公开来源（步骤 1 的 Admin API 事实来自第三个）；在任何东西定规模之前，步骤 0 要求抓取第一个与 Pricing 页面；当用户需要完整论述或模式时才抓取 cookbook 与 Admin API 文档：

- The platform guide **Optimizing for cost and intelligence** - WebFetch the Cost Optimization URL in `shared/live-sources.md`.
  平台指南 **Optimizing for cost and intelligence**——WebFetch `shared/live-sources.md` 中的 Cost Optimization URL。
- The cookbook **Cost optimization on the Claude API** (`https://platform.claude.com/cookbook/cost-optimization-cost-optimization`) - a runnable end-to-end worked example of this workflow.
  cookbook **Cost optimization on the Claude API**（`https://platform.claude.com/cookbook/cost-optimization-cost-optimization`）——本工作流可运行的端到端完整示例。
- The **Usage and Cost Admin API** docs - the URL in `shared/live-sources.md`; the endpoint reference pages linked from that page carry the full parameter and response schemas.
  **Usage and Cost Admin API** 文档——URL 见 `shared/live-sources.md`；从该页面链接的端点参考页载有完整参数与响应模式。
- Per-model prices: always the **Pricing** URL in `shared/live-sources.md`, never remembered rates.
  分模型价格：永远以 `shared/live-sources.md` 中的 **Pricing** URL 为准，绝不凭记忆的费率。
- The **Preserved thinking** page (URL in § 2.1) - required whenever the plan includes a lever that edits history, `system`, or `tools`.
  **Preserved thinking** 页面（URL 见 § 2.1）——只要计划包含会编辑历史、`system` 或 `tools` 的杠杆就必须抓取。
