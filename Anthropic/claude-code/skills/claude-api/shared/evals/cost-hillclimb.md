<!-- BILINGUAL-EN-ZH -->
# Cost-reduction search: how to structure it / 降本搜索：如何组织

Guidance for running an eval-driven search whose central goal is **reducing cost** (at
equal-or-better quality) for a Claude-powered app - typically during a migration from an
older model and prompt to a current one. This is the procedure the hillclimb loop
(`eval-hillclimb.md`) follows when the user's Step 1 goal is cost; it assumes there is an
eval to measure quality against. Without one, use `shared/cost-optimization.md` instead - 
the no-eval checklist (caching -> input trim -> agent-loop hygiene -> output -> batch -> effort ->
model last). The lever order differs on purpose: with an eval you can *detect* that a
stronger model at lower effort is the cheaper cell, so the model × effort walk comes early
here; without one, a model swap is the riskiest change and belongs last. Where this file states a number, it
reflects measured behavior on representative benchmark evals except where a
practitioner report is noted as such, stated so you can anticipate the shape of
results; always re-measure on the user's own eval.

本指南用于组织一场以**降低成本**（质量持平或更好）为核心目标、由评测驱动的搜索，对象是 Claude 驱动的应用——通常发生在从旧模型与旧提示词向当前版本迁移的过程中。当用户在第 1 步设定的目标是成本时，爬坡循环（`eval-hillclimb.md`）遵循的就是这套流程；它假定存在可用于度量质量的评测。若没有评测，请改用 `shared/cost-optimization.md`——无评测清单（缓存 -> 输入精简 -> 代理循环卫生 -> 输出 -> 批处理 -> effort -> 模型最后）。两者的杠杆顺序刻意不同：有评测时，你能*探测出*"更强模型 + 更低 effort"才是更便宜的格子，所以模型 × effort 阶梯走在这里排得很早；没有评测时，换模型是风险最大的改动，应当放在最后。本文件给出的数字，除注明为从业者报告之处外，均反映在代表性基准评测上测得的行为，写明数字是为了让你对结果的形态有所预期；请始终在用户自己的评测上重新测量。

【评论】本文档的方法论核心是预登记（质量地板、门槛、证伪条件）与噪声条测量，用以约束多轮搜索中的选择偏差与过拟合——这是一套实验设计纪律，而非单纯的经验技巧。

## The search order / 搜索顺序

Work the levers in this order. Each position exists because running it later corrupts
or wastes the steps in between.

按此顺序操作各个杠杆。每个位置的存在都有原因：放到更晚执行会污染或浪费其间的步骤。

**Step 0 - Caching health: check first, re-verify after every lever change.**
Caching is configuration, not a search lever - its health gates the accuracy of every
cost measurement below, and its dominant failure mode is silent invalidation. Quick
health check: cache breakpoints are set; the cached prefix contains no dynamic content;
`cache_read_input_tokens` is nonzero in responses. Do not restate or re-derive caching
design here - follow the skill's prompt-caching guidance (shared/prompt-caching.md
in the shipped package; its load-bearing practices: re-verify cache health after
every change, not just at setup; know what a healthy cache loop looks like in the
usage fields; and when reads drop, hunt down the specific invalidator) - but
re-run this health check after EVERY model or effort decision below. Two scoping facts matter for every
step below: the cache is per-model, and effort participates in the prompt-cache key on
the Messages API - a mid-conversation effort switch costs one full prefix rewrite
(probe-measured on a current top-tier model against the raw API: 3/3 effort switches
re-billed the full prefix; 0/9 same-effort continuations did). Reports from first-party
product surfaces that effort can be switched without losing the cache do not transfer
to API customers - the serving path differs; trust an API-side probe. The upshot: a
lever change silently converts a healthy cache into a cost regression that looks like
model behavior, so re-verify cache health after every step of the model×effort walk
(Step 2), not only at setup.

**第 0 步——缓存健康状况：先检查，此后每次杠杆变更后重新验证。**
缓存是配置，不是搜索杠杆——它的健康状况决定了下文每一次成本测量的准确性，而其最主要的失效模式是静默失效。快速健康检查：缓存断点已设置；被缓存的前缀不含动态内容；响应中 `cache_read_input_tokens` 非零。不要在这里复述或重新推导缓存设计——遵循本技能的提示词缓存指南（交付包中的 shared/prompt-caching.md；其承重实践：每次变更后都重新验证缓存健康，而不只在搭建时；要知道健康的缓存循环在用量字段里长什么样；当缓存读取下降时，追查具体的失效原因）——但下文每一次模型或 effort 决策之后都要重跑这个健康检查。有两个与下文每一步都相关的作用域事实：缓存按模型隔离，且在 Messages API 上 effort 参与提示词缓存键——对话中途切换 effort 的代价是一次完整的前缀重写（在当前一款顶级模型上针对原始 API 做探针实测：3/3 次 effort 切换被重新计费完整前缀；0/9 次同 effort 续接没有）。来自第一方产品界面的"切换 effort 不丢缓存"的报告不能迁移到 API 客户——服务路径不同；要信 API 侧的探针。结论是：杠杆变更会把健康的缓存悄悄变成一次看起来像模型行为的成本回退，所以在模型×effort 阶梯走（第 2 步）的每一步之后都要重新验证缓存健康，而不是只在开始时。

**Step 1 - Audit the existing prompt and request config.**
Dated prompt content and legacy request parameters corrupt every later comparison: in
the harder arm of a measured migration (a heavily dated, extended prompt), the model
upgrade alone moved accuracy barely at all while auditing the same prompt recovered
more than ten times the gap; the milder arm of the same migration (lightly planted
cruft) measured a ~2× gap, so the multiple scales with how dated the prompt is. A
planted legacy thinking configuration (a dated budget-tokens shape) turned out to be
rejected outright - field-level, on every current model, probe-verified - and had to
be re-authored before any comparison could run at all; the period-correct form that
does run everywhere inflates cost instead of crashing, which is worse, because nothing
forces you to notice it. Newer models follow
legacy scaffolding MORE literally, so a model swap judged under a dated prompt can
mis-rank models - and can even *add* cost. The audit is cheap, runs once, and de-noises
everything after it. Run `prompt-audit` (the procedure ships as
shared/prompt-audit.md), and check request parameters against the current API
surface (see the model-migration guide), before any measurement you intend to keep.

**第 1 步——审计既有提示词与请求配置。**
过时的提示词内容与遗留请求参数会污染之后的所有比较：在一次实测迁移较难的那一臂（高度过时、长期堆叠的提示词）中，仅升级模型几乎没能提升准确率，而对同一提示词做审计挽回的差距是它的十倍以上；同一迁移较温和的一臂（轻度植入的杂物）测得约 2× 差距，可见倍数随提示词过时程度放大。一个植入的遗留 thinking 配置（过时的 budget-tokens 形态）结果是直接被拒绝——逐字段地、在所有当前模型上、经探针验证——必须先重写才能开始任何比较；而在哪里都能运行的"当期正确"形态不会崩溃，而是推高成本，这更糟，因为没有任何东西迫使你注意到它。较新的模型会更死板地遵循遗留脚手架，所以在过时提示词下评判的模型更换可能错排模型——甚至可能*增加*成本。审计很便宜、只跑一次，且能消除其后一切测量的噪声。在任何你打算保留的测量之前，先运行 `prompt-audit`（流程随包交付为 shared/prompt-audit.md），并对照当前 API 表面检查请求参数（见模型迁移指南）。

**Step 2 - Model × effort: walk the staircase, don't sweep the grid.**
The cheapest configuration is frequently a *stronger* model at *lower* effort: the
stronger model tends to spend fewer tokens on the same task (a measured direction,
not a fixed magnitude), so it can win on cost as well as score. That
configuration is invisible to any procedure that fixes the model at default effort and
then tunes effort - and a full factorial grid finds it only by paying for every cell,
most of them in the expensive corner. Treat model × effort as one surface and search
it with a *staircase walk*: lay the cells out with model tier on one axis and effort
on the other. Cost rises with effort within each row - across tiers the measured
cost bands can overlap (see the bottom-up exception below) - and quality rises
(weakly) along both axes, so the cells that clear a pre-registered quality floor
form an upper-right region, the cheapest acceptable cell sits near that region's
lower-left boundary, and a monotone
walk from the top-left corner traces that boundary in roughly (tiers + effort notches)
cells instead of (tiers × effort notches) - and the cells it skips are the expensive
corner. (On cheap evals the walk buys pre-registrable discipline more than dollars:
one measured program's full grid cost ~$20 all-in.) Scope note: the "stronger model at lower effort" pattern is measured at list
prices on ordinary-sized tasks and is benchmark-dependent - the walk verifies it on
the user's own eval rather than assuming it.

**第 2 步——模型 × effort：走阶梯，不要扫全网格。**
最便宜的配置常常是*更强*的模型配*更低*的 effort：更强的模型往往在相同任务上花更少的 token（这是实测出的方向，不是固定幅度），因此它可以同时在成本与得分上胜出。这一配置对任何"把模型固定在默认 effort 再调 effort"的流程都不可见——而全因子网格只有为每个格子付钱才能找到它，其中多数格子位于昂贵的角落。把模型 × effort 当作一个整体曲面来对待，用*阶梯走*搜索它：把格子铺开，模型档位在一个轴、effort 在另一个轴。每行之内成本随 effort 上升——跨档位的实测成本带可能重叠（见下文的自底向上例外）——质量沿两个轴都（弱）上升，因此通过预登记质量地板的格子构成一个右上区域，最便宜的可接受格子位于该区域左下边界附近，而从左上角出发的单调行走用大约（档位数 + effort 档数）个格子就能描出这条边界，而不是（档位数 × effort 档数）个——它跳过的正是昂贵的角落。（在廉价的评测上，阶梯走买到的主要是可预登记的纪律而非美元：一个实测项目的全网格总花费约 $20。）范围说明："更强模型 + 更低 effort"模式是按目录价、在普通规模任务上测得的，且依赖具体基准——阶梯走会在用户自己的评测上验证它，而不是假定它成立。

*Set up the grid.* Rows are model tiers ordered by capability; columns are effort
notches (low -> high). Two regularities make the walk work, and one of them is the
load-bearing assumption to name in the plan: within a row, cost rises with effort
(measured in every row of both programs behind this guide); and at fixed effort,
quality rises with tier (held in every measured pair - where it bends, the walk
mis-prunes, which is part of what the registered confirm exists to catch). Quality
along the *effort* axis is only weakly monotone - in one measured family the high
notch drew below low on single-run screens (a tie within the measured ~±6 noise
band, while costing 1.7× the tokens) - so the walk never leans on it. The frontier tier above the
default top row, where one exists, sits *outside* the grid as an extension: it has
repeatedly priced above the top row's medium cell (~7× the eventual winner in one
measured migration), so treat it as a quality probe, not a cost candidate - the
cost-plausibility screen below is the test that decides this per workload.

*搭好网格。*行是按能力排序的模型档位；列是 effort 档（低 -> 高）。两个规律使阶梯走得以成立，其中之一是必须在计划中点名的承重假设：同一行内，成本随 effort 上升（在本指南背后的两个项目的每一行都测得）；固定 effort 时，质量随档位上升（在每一对实测组合中都成立——若该规律弯曲，阶梯走会错误剪枝，这正是登记确认环节要抓的问题之一）。沿 *effort* 轴的质量只是弱单调——在一个实测家族中，高档在单次运行初筛上的得分低于低档（处于实测约 ±6 噪声带内的平局，同时多花 1.7× 的 token）——所以阶梯走绝不依赖它。默认顶行之上的前沿档位（若存在）作为扩展放在网格*之外*：它的定价反复高于顶行的 medium 格（一次实测迁移中约为最终赢家的 7×），因此把它当作质量探针，而不是成本候选——下文的成本合理性初筛才是按具体工作负载做此判断的检验。

*Round-0 diagnostic (before the grid spends anything).* From the baseline repeats
already run for the noise bar, read two signals out of the transcripts: output-token
share and turns per task. The third signal - effort sensitivity - costs one or two
cheap cells on the *old* model at a different effort. Output-heavy, multi-turn, and
effort-sensitive -> expect the winner near the top rows' low cells and budget repeats
there. Input-heavy, single-shot, effort-flat -> the exception class, where per-token
price dominates; the entry cell is the same, but expect to step down quickly and
budget repeats for the bottom rows. The diagnostic sets where repeats get spent,
never where the walk enters.

*第 0 轮诊断（在网格花钱之前）。*从已经为噪声条跑过的基线重复中，从转录里读出两个信号：输出 token 占比与每任务轮数。第三个信号——effort 敏感性——需要花一两个便宜格子：在*旧*模型上换一个 effort 跑。输出重、多轮、effort 敏感 -> 预期赢家在顶行的低档格子附近，把重复预算放在那里。输入重、单发、effort 不敏感 -> 属于例外类，每 token 单价主导；入口格子相同，但预期很快向下走，并把重复预算留给底部几行。诊断决定的是重复预算花在哪里，从不决定入口在哪里。

*Cost-plausibility screen (what "cost-plausible" means).* Project each candidate
tier's low cell from billing texture at matched effort - the tier's token prices
times a low-effort texture, which the round-0 diagnostic already bought on the old
model; if only the baseline's own-effort texture exists, discount it by the measured
round-0 effort ratio before applying the bar. (This is rule 4's effort-matching
requirement applied at screen time: a profile at a different effort overstates a low
cell by roughly the effort ratio, and at the screen the overstatement lands on the
silent-exclusion side.) A tier moves out to the above-grid extension only when the
evidence is overdetermined: no projection within the documented error band brings
its low cell in under the next tier down's medium cell - a bare point estimate
cannot establish "can't"; measured projection errors have run ~2× optimistic and
2.8× pessimistic - AND no measured or reported token-economy evidence suggests the
tier closes that gap in this workload's regime (smarter tiers spend fewer tokens per task, but how much is regime- and
pair-dependent, and the measured economies so far are single-draw readings - treat
them as direction, not magnitude). The screen is one-sided by design: when in doubt the tier stays in
the grid, because the walk makes inclusion errors cheap (the entry probe is that
tier's cheapest cell, and a fail prunes a whole column) while exclusion errors are
silent (only the on-fail extension trigger can catch one). Record the screen's
verdict and its basis in the pre-registration.

*成本合理性初筛（"成本上说得通"的含义）。*按匹配 effort 下的计费纹理投影每个候选档位的低档格子——该档位的 token 单价乘以低 effort 纹理，后者已由第 0 轮诊断在旧模型上买到；若只有基线自身 effort 的纹理，先按实测的第 0 轮 effort 比率打折，再套门槛。（这是规则 4 的 effort 匹配要求在初筛时刻的应用：用不同 effort 的画像会把低格子高估大约一个 effort 比率，而在初筛中这种高估落在"静默排除"一侧。）只有当证据过度充分确定时，才把一个档位移到网格上方的扩展区：文档化的误差带内没有任何投影能让它的低格子低于下一档的 medium 格——光靠一个点估计无法确立"不可能"；实测投影误差曾达约 2× 乐观与 2.8× 悲观——*并且*没有任何实测或已报告的 token 节约证据表明该档位会在此工作负载的形态中弥合这一差距（更聪明的档位每任务花更少 token，但幅度取决于形态与模型对，且迄今实测的节约都是单次抽读——把它们当方向，别当幅度）。这个初筛按设计是单侧的：存疑时档位留在网格内，因为阶梯走让"错留"很便宜（入口探针就是该档位最便宜的格子，一次失败即可剪掉一整列），而"错删"是静默的（只有 on-fail 扩展触发器能抓到）。把初筛的裁定及其依据登记进预登记文档。

*Enter at the highest cost-plausible new-generation tier at low effort.* Three
reasons. Only the top-left corner gives an unambiguous walk - a pass prunes in one
direction and a fail prunes in the other, while from the bottom-left corner a fail
leaves two uphill directions and no way to choose without probing both. The entry
probe is the top tier's cheapest cell, so it is budget-bounded. And it doubles as the
regime test: if the whole top row fails the floor, the eval sits at or above the
models' capability frontier - upgrading is a quality story, not a cost story; stop
searching for savings and say so. (One honest cost: the top row *before the first
pass* has no incumbent, so its rightward walk is cost-unbounded by construction.
That is the regime probe's price.)

*从成本上说得通的最高新生代档位、以低 effort 进入。*三个理由。只有左上角给出无歧义的行走——通过则向一个方向剪枝，失败则向另一个方向剪枝；而从左下角出发，一次失败会留下两个向上的方向，不把两个都探一遍就无法选择。入口探针是顶档最便宜的格子，因此预算有界。而且它兼作形态检验：如果整行顶行都不过地板，说明评测位于或超出模型的能力前沿——升级是质量故事，不是成本故事；停止寻找节省并如实说明。（一个诚实的代价：第一次通过*之前*，顶行没有在位者，因此它向右的行走按构造没有成本上界。这就是形态探针的价格。）

*The walk.*

*行走本身。*

- **Pass at (tier k, effort e):** record the cell as the incumbent with cost c*, drop
  the rest of row k, and step *down* a tier - from here on, only into cells projected
  under c*. The row-drop needs no quality assumption at all: every cell rightward of
  a pass costs more than the pass, so it cannot beat the incumbent whether or not it
  clears the floor. That makes the rule robust to effort non-monotonicity.
  **在（档位 k，effort e）通过：**把该格记录为成本 c* 的在位者，丢弃第 k 行其余部分，并*向下*走一档——此后只走进投影在 c* 之下的格子。整行丢弃完全不需要质量假设：通过格右侧的每个格子都比通过更贵，所以无论是否过地板，它都不可能击败在位者。这让该规则对 effort 非单调性保持稳健。
- **Fail at (tier k, effort e):** presume every lower tier at effort e also fails
  (the tier-monotonicity assumption above) and step *right*. Once an incumbent
  exists, step only into cells projected under c*, and cap the rightward walk at one
  or two notches - quality in effort is too weakly monotone to chase further.
  **在（档位 k，effort e）失败：**假定同 effort 下所有更低档位也失败（即上述档位单调性假设），并*向右*走。在位者存在后，只走进投影在 c* 之下的格子，且向右最多走一两档——effort 维度上的质量太弱单调，不值得追更远。
- **Overlap probes:** after a fail-then-pass on tier k, cells of tier k-1 projected
  under c* may still be probed. This overlap - the smaller model working hard against
  the smarter model barely trying - is the only place the walk branches.
  **重叠探针：**在档位 k 上先失败后通过之后，投影在 c* 之下的档位 k-1 的格子仍可探测。这种重叠——小模型拼命干活对抗几乎不费力的更聪明模型——是阶梯走唯一会分叉的地方。
- **Extension trigger (the one upward move):** if the top row's low cell fails and
  the row's passing cells price near the above-grid extension tier, probe the
  extension's low cell before stopping. This is also the only recovery path for a
  tier the screen wrongly excluded - project it effort-matched (rule 4).
  **扩展触发器（唯一的向上动作）：**如果顶行的低档格子失败、且该行通过格子的价格接近网格上方的扩展档位，在停止前探测扩展档位的低档格子。这也是初筛错删档位的唯一恢复路径——按 effort 匹配做投影（规则 4）。
- **Stop** when no unprobed cell projects under c* and the extension trigger is
  quiet. The incumbent is the answer.
  当没有任何未探测格子投影在 c* 之下、且扩展触发器保持安静时**停止**。在位者就是答案。

In one sentence: enter at the highest cost-plausible new-generation tier at low
effort; step down on pass, right on fail; prune by incumbent cost; the frontier
tier's low cell is the on-fail extension above the grid, and the old model at low is
the round-0 diagnostic below it.

一句话总结：从成本上说得通的最高新生代档位、以低 effort 进入；通过向下走，失败向右走；按在位者成本剪枝；前沿档位的低档格子是网格之上的失败扩展，其下的旧模型低 effort 是第 0 轮诊断。

【评论】阶梯走本质上是利用成本与质量的（弱）单调性假设所做的带剪枝贪心搜索；文中同时为假设失效的情形准备了补救机制——重叠探针、噪声带内复测与扩展触发器。

*Decision machinery the walk cannot run without.*

*阶梯走离不开的决策机制。*

1. **The floor is pre-registered before round 1** - for example the baseline's best
   repeat, with the rationale stated - never the baseline mean read after the fact.
   In one measured migration the registered floor was set above the baseline mean
   precisely because a one-repeat score at or just under that mean is as likely
   below baseline as at it - such a read is not admissible evidence of parity.
   **地板在第 1 轮之前预登记**——例如基线最好的一次重复，并写明理由——绝不是事后读出的基线均值。在一次实测迁移中，登记的地板之所以设在基线均值之上，正是因为单次重复的得分等于或略低于该均值时，其真实水平在基线之下与恰在基线上的可能性相当——这种读数不能作为"持平"的可采信证据。
2. **Promotion needs n >= 2 near the floor.** Any cell about to become the incumbent
   whose margin over the floor is inside the noise bar gets a second run before c*
   moves. The errors are asymmetric and the expensive one is the false *pass*: a
   lucky pass sets c* and prunes the true winner, a lower tier failing says nothing
   about whether the pass above it was real, and the final confirm catches the error
   only after the pruned cells are gone. (Measured: a winner that passed at 11 of 20
   at one repeat drew 6 of 20 later on the identical configuration.) A false fail
   just sends the walk one cell right - cheaper, and the next rule covers it.
   **接近地板的晋升需要 n >= 2。**任何即将成为在位者、且其相对地板的优势落在噪声带之内的格子，在 c* 移动前都要再跑一次。两类误差不对称，更昂贵的是假*通过*：一次侥幸的通过会设定 c* 并剪掉真正的赢家；更低档位的失败说明不了它上方那次通过是否为真；而最终确认只能在格子被剪掉之后才抓到这个错误。（实测：一个在单次重复中以 20 题对 11 通过的赢家，之后在完全相同的配置上只得到 20 题对 6。）假失败只是让行走向右移一格——代价更低，且下一条规则会兜住它。
3. **Fails inside the noise bar of the floor get re-tested** before the walk steps
   right on their account.
   **落在地板噪声带内的失败要先复测**，然后才能据此让行走向右。
4. **Pruning is calibrated deferral, not deletion.** Project a cell's cost from
   billing texture - the tier's token prices times a measured token profile MATCHED
   TO THE CANDIDATE'S EFFORT (a low-effort candidate projects from a low-effort
   cell's banked texture; the incumbent's profile at a different effort overstates
   the candidate by roughly the effort ratio - a measured case read 2.8× too high on
   an incumbent-at-higher-effort basis, and the effort-matched projection reversed
   the verdict to cheaper-than-incumbent) - and never from priors alone: early
   projections in one measured program ran ~2× optimistic, and optimistic
   projections *under*-prune. A pruned cell is deferred; re-admit it if the
   projections recalibrate.
   **剪枝是经过校准的推迟，不是删除。**用计费纹理投影一个格子的成本——该档位的 token 单价乘以一份与候选者 effort 相匹配的实测 token 画像（低 effort 候选者从低 effort 格子的既有纹理投影；用不同 effort 的在位者画像会高估候选者大约一个 effort 比率——一个实测案例在"更高 effort 的在位者"基础上读高了 2.8×，而 effort 匹配的投影把裁定反转为比在位者更便宜）——绝不仅凭先验：一个实测项目早期的投影乐观了约 2×，而乐观的投影会*剪得不够*。被剪掉的格子只是被推迟；若投影重新校准，就重新纳入。

*What the walk looks like on real evals.* Replaying the two measured migrations
behind this guide: on the ordinary-workload eval the walk reaches the eventual winner
in two cells - the entry cell passes at low, and the tier below passes at low once
the prompt is clean (under the surviving cruft it failed there, and was rescued by
the Step 4 re-probe - see that step's caveat) - and then asks the one question the
actual program never did, the next tier further down; the frontier-tier cell that
program did run plays the declared quality-probe role below. On the frontier-hard eval the entry cell fails the floor, and the same
tier's medium cell passes - one point over the floor, inside the noise bar, which is
exactly the case rule 2 above exists for: it gets a second run before it promotes to
incumbent. (The confirm's later 11, 6, 11 spread on that identical configuration is
the demonstration - a one-point margin at one repeat can be a 6.) The walk then steps down to the
mid tier's low cell - a probe the actual program never fired (projected cheaper than
the cells it did) - and on a fail would step right into the mid tier's medium cell,
which the program did fire: it came in under the incumbent's cost and failed badly.
There the walk stops, pruning without running it the mid tier's high cell, which the
actual program paid real money to learn was priced above the incumbent. (That program's winner read -46% vs the same model's high setting and -43% vs the
old-model baseline on fresh-run means, where the single cheapest selection pass read
-50%/-48% on the same comparisons: report the fresh-run means, never the favorable
end of a spread. Both arms billed on an internal page-counted route; on public
breakpoint billing the reductions run a few points smaller - ~-40% on the baseline
comparison - and the arms compare like-for-like either way.) The frontier-hard case is also the cautionary half: the winner's
pre-registered confirm FAILED its stability clause - the identical configuration drew
11 of 20 on the selection pass and then 11, 6, and 11 across the three fresh confirm
runs - so the honest verdict was "cost cut firm,
mean quality comparable within noise, NOISIER than baseline", and the headline had to
be the confirm's number, not the selection round's. Which regime you are in decides
whether upgrading is a cost lever or a quality lever; the entry cell is what tells
you, in one bounded probe.

*阶梯走在真实评测上是什么样子。*回放本指南背后的两次实测迁移：在普通工作负载的评测上，阶梯走用两个格子就到达最终赢家——入口格子以低 effort 通过，提示词清干净后低一档的档位也以低 effort 通过（在残留杂物下它当时在那里失败，靠第 4 步的再探针挽回——见该步的注意事项）——然后它会问那个实际项目从未问过的问题：再往下一档；该项目实际跑过的那个前沿档位格子则扮演下文声明的质量探针角色。在前沿困难的评测上，入口格子没过地板，而同档位的 medium 格子通过——只超地板一分、在噪声带内，这正是上文规则 2 存在的情形：它在晋升为在位者之前又跑了一次。（该确认在完全相同配置上随后 11、6、11 的分布就是演示——单次重复的一分优势可能是 6。）阶梯走随后向下走到中档的低档格子——实际项目从未发射过的探针（投影比它跑过的格子更便宜）——若失败会向右走进中档的 medium 格，而项目确实跑过这一格：它落在在位者成本之下，且败得很惨。在那里阶梯走停止，不跑就剪掉中档的高档格——实际项目花真金白银才知道它定价比在位者高。（该项目的赢家按 fresh-run 均值读数相对同模型高档为 -46%、相对旧模型基线为 -43%，而单次最便宜的选择轮在同一比较上读出 -50%/-48%：要报告 fresh-run 均值，绝不报告分布的有利一端。两臂都按内部按页计数的路由计费；在公共断点计费下，降幅要小几个百分点——基线比较约 -40%——但无论如何两臂都是同口径比较。）前沿困难案例也是警示性的一半：赢家的预登记确认未通过其稳定性条款——完全相同的配置在选择轮抽到 20 题对 11，随后三次全新确认运行分别抽到 11、6、11——所以诚实的结论是"成本削减扎实、均值质量在噪声内相当、但比基线*更噪*"，标题必须用确认轮的数字，而不是选择轮的。处于哪种形态决定了升级是成本杠杆还是质量杠杆；入口格子用一个有界探针就能告诉你答案。

*Per-cell honesty rules (they apply to every probe in the walk):*

*每格诚实规则（适用于行走中的每一次探测）：*

- **Single-run reads are pass/fail evidence, not rankings.** A single eval pass can
  swing several points on sampling alone (a recorded small-set example: ~8 points,
  ±6 across repeats). The honest single-run signals are *telemetry* - realized
  thinking tokens, output tokens, and tool rounds per case - not small score deltas.
  Measure the noise bar BEFORE the walk (repeat runs of the incumbent config, or
  pass@k over existing results files), so the floor margin and the promotion rule
  have a number to work with.
  **单次读数是通过/失败证据，不是排名。**仅凭采样，一次评测通过就可能摆动好几分（一个记录在案的小集合例子：约 8 分，重复间 ±6）。诚实的单次信号是*遥测*——每个案例实际发生的 thinking token、输出 token 与工具轮数——而不是小的分差。在行走之前先测噪声条（对在位配置重复运行，或对既有结果文件做 pass@k），让地板优势与晋升规则有数字可用。
- **Confirm the effort dial is alive before crediting a rightward step.** Effort
  curves are per-model-family and not always monotonic: measured cases include a
  family where medium beat high, and a score curve with a knee that endpoint
  sampling cannot see. If a step right does not change realized thinking tokens, the
  dial is dead for that family - further rightward cells are the same cell at a
  higher price, so treat the row as exhausted.
  **在把向右一步记功之前，先确认 effort 旋钮还活着。**effort 曲线按模型家族而异，且不总是单调：实测案例中有一个家族 medium 胜过 high，还有一条带拐点、端点采样看不见的得分曲线。如果向右一步没有改变实际发生的 thinking token，说明该家族的旋钮是死的——更右侧的格子是同一个格子的更高价格版本，把该行当作已穷尽。
- **Walk cost readings are cache-cold.** Cells never share cache - the cache is
  per-model and effort participates in the cache key (Step 0) - so production cost
  will be cheaper than walk cost by the cache rate; either warm each cell or
  annotate the readings. Keep effort fixed within any session whose cost is being
  measured, or cache invalidation noise lands in the effort arm's numbers.
  **行走的成本读数是缓存冷的。**格子之间从不共享缓存——缓存按模型隔离，且 effort 参与缓存键（第 0 步）——所以生产成本会比行走成本低出缓存率那部分；要么把每个格子焐热，要么给读数加注。在被测成本的会话内保持 effort 固定，否则缓存失效噪声会落进 effort 臂的数字里。
- **State the billing basis per cell.** Eval-harness billing routes can differ from
  what an API customer pays - some internal routes count cache in fixed-size pages
  where the public API bills exact tokens from breakpoints - and the same run can
  differ materially in reported cost across routes. Check which route the ledger
  rides before quoting absolute costs; ratios between cells on the same route are
  more robust than absolutes.
  **逐格声明计费口径。**评测框架的计费路由可能与 API 客户实际支付的不同——一些内部路由按固定大小的页计缓存，公共 API 按断点的精确 token 计费——同一次运行在不同路由上报告的成本可能差异巨大。引用绝对成本前，先确认账本走的是哪条路由；同一路由上格子之间的比值比绝对值更稳健。

*Declare a quality probe, or the ceiling goes unmeasured.* By construction the walk
never fires the expensive corner, so a migration that passes early never learns what
the top tier at high effort would have bought. If that number is wanted - it usually
is, once - run the top-right cell, or the above-grid frontier tier at low, as ONE
declared quality-reference cell outside the cost walk, marked as such in the plan.
Skippable on tight budgets.

*声明一个质量探针，否则天花板无人测量。*按构造，阶梯走永远不会发射昂贵的角落，因此早早通过的迁移永远不知道顶档配高 effort 本可以买到什么。如果想要这个数字——通常想要一次——就把右上格子、或网格上方的前沿档位低档，作为一格*声明的*质量参照格跑一次，放在成本行走之外，并在计划中如此标记。预算紧张时可跳过。

*When entering from the bottom is defensible.* Two cases. (1) Steady-state tuning - 
already on a current-generation model with a tuned prompt, just trimming: sweep
effort downward from where you are, but ALWAYS add the single next-tier-up-at-low
probe. The blind spot it closes: a bottom-up sweep that finds quality fine and cost
high at a lower tier never escalates, so the cheaper-better cell one tier up at low
is never tested. The measured billing overlap is why this is unsafe to skip: in one
program the top tier's low cell drew per-run costs both below and above the mid
tier's medium cell across two draws (~0.8× and ~1.5× its cost, the mid cell itself
a single draw) while solving more cases in both - adjacent tiers' measured cost
bands overlap, so tier order cannot be trusted to give cost order
(overlap evidence for the hazard, not an observed firing of the blind spot itself:
in that program the mid tier's cell also failed the floor, so even a bottom-up sweep
would have escalated). (2) A total budget of a cell or two plus a round-0 diagnostic
reading "exception class": go straight to the same-tier successor at low and accept
the risk of missing the inversion.

*什么时候自底向上进入是可辩护的。*两种情形。（1）稳态调优——已经在一个提示词调好的当前代模型上、只想再修剪：从当前位置向下扫 effort，但务必补上那一格"上一档配低 effort"的探针。它弥补的盲区是：自底向上的扫掠在一个更低档位上发现"质量尚可、成本偏高"时不会向上升级，于是高一档低 effort 那个更便宜更好的格子永远不会被测到。实测的计费重叠正说明了为什么跳过它不安全：在一个项目中，顶档低档格子两次抽得的单次成本一次低于、一次高于中档 medium 格（约其成本的 0.8× 与 1.5×，中档格子本身只有单次抽读），同时它在两次中都解出更多案例——相邻档位的实测成本带相互重叠，所以档位顺序不能被信任为代表成本顺序（这是关于该风险的重叠证据，不是盲区实际触发的观察：在该项目中，中档格子同样没过地板，因此即便自底向上扫掠也会升级）。（2）总预算只有一两格、且第 0 轮诊断读出"例外类"：直接去同代后继档位的低档，接受错过反转的风险。

(For what the effort knob is and when the top of the range earns its cost, see the
skill's effort-level guidance - a pending skill update, not yet in the shipped
package; this section is about how to SEARCH it.)

（effort 旋钮是什么、以及区间顶端何时物有所值，见本技能的 effort 分级指南——一次待发布的技能更新，尚未进入交付包；本节讲的是如何*搜索*它。）

**Step 3 - Prompt-hillclimb on the frozen model.**
Prompt wins do not transfer across models - measured gains of +30 and +15 points on two
model families were each model-specific, and one newer model's failure mode was not
prompt-addressable at all. Hillclimbing the prompt before the model is frozen wastes
the climb. Run the loop per the hillclimb guide, with the cost-specific rules below.

**第 3 步——在冻结的模型上做提示词爬坡。**
提示词收益不能跨模型迁移——在两个模型家族上分别实测到 +30 与 +15 分的收益各自都是模型特定的，且一个更新模型的失败模式完全无法用提示词解决。在模型冻结之前爬坡提示词会浪费整场爬坡。按爬坡指南运行循环，并附加下文的成本专用规则。

Do not assume the prompt is where the cost lives. The Step 1 audit tells you what is
*wrong* with a prompt; only measurement tells you what the wrongness *costs*. In one
measured case a dated opener full of turn-inflating ritual (forced plan files,
re-read-after-every-edit, full test suite after every change) audited as an obvious
cost win - and the cleaned opener failed to save anything: both cleanup cells landed
above the CEILINGS of their pre-registered 80% cost intervals (the cleaned cell's
point prediction was a ~32% cut), and inside the incumbent's own identical-config cost
spread measured later (so "cost more" is within noise; "missed its pre-registered
cost interval entirely" is the solid finding). The engagement census (below) showed the
forced-reasoning ritual was fully ignored while a plan-file instruction was genuinely
obeyed in 17 of 20 runs - and deleting all of it saved nothing, because per-run cost
was bound by turn count and context growth, not by opener text. That is sharper than
"models ignore dead text": even the obeyed scaffolding was not where the cost lived.
Pre-register the falsifier before the cleanup round - "if the cleaned prompt does not
come in under a named cost bar, the prompt lever is exhausted here, say so" - so a
no-win closes the lever with a recorded finding instead of inviting another round of
edits at the same dead wall.

不要假定成本就藏在提示词里。第 1 步的审计告诉你提示词*哪里不对*；只有测量才告诉你这种不对*花了多少钱*。在一个实测案例中，一段满是增加回合的仪式的过时开场白（强制计划文件、每次编辑后重读、每次改动后跑全量测试套件）被审计为明显的成本赢点——而清理后的开场白什么也没省下：两个清理格子都落在了各自预登记的 80% 成本区间的上限之上（清理格子的点预测是约 32% 的削减），并且落在后来实测的在位者同配置成本散布之内（所以"成本更高"在噪声之内；"完全错过预登记成本区间"才是扎实的发现）。参与度普查（见下）显示强制推理仪式被完全无视，而一条计划文件指令在 20 次运行中有 17 次被真正遵守——但把它们全删了也没省下什么，因为单次运行成本由轮数与上下文增长决定，而不是由开场白文本决定。这比"模型无视死文本"更尖锐：连被遵守的脚手架也不是成本所在。在清理轮之前预登记证伪条件——"如果清理后的提示词没有落在某个指名的成本条之下，提示词杠杆在此耗尽，如实说明"——这样一次无收益就会以一条被记录的发现关闭该杠杆，而不是招致在同一堵死墙上再来一轮编辑。

【评论】该段的核心教训是：审计判断的"明显问题"与实际成本构成可以完全脱节，提示词层面的直觉需要用带预登记区间的测量来检验。

**Step 4 - After the prompt climb, re-probe one cell down-left.**
One effort notch lower, or one tier lower at the effort that just passed. Cleanup can
make a previously failing cheaper cell viable, so the walk's verdict on those cells
expires when the prompt changes: in one measured migration the mid tier's low cell
went from 0.64 under the dated prompt to 0.98 after the transcript-driven cleanup on
the frozen model. Honest caveat: that rescue came
from the round-3 transcript-driven cleanup; whether the lighter pre-grid mechanical
audit (Step 1) alone recovers such cells is untested - the earlier program's arc
suggests it recovers much of the gap (audited cells scored far above swap-only cells
on the same eval), but treat that as suggestive, not measured, for this specific
re-probe. One cell, not a re-opened search: if it passes under the incumbent's cost,
it becomes the configuration the confirm tests; if not, the incumbent stands.

**第 4 步——提示词爬坡之后，向左下一格再探一次。**
低一档 effort，或同一 effort 下降一档。清理可以让此前失败的更便宜格子变得可行，所以行走对这些格子的裁定随提示词变更而过期：在一次实测迁移中，中档低档格子的得分从过时提示词下的 0.64 升到冻结模型上转录驱动清理后的 0.98。诚实的注意事项：那次挽回来自第 3 轮的转录驱动清理；仅凭更早的网格前机械审计（第 1 步）能否挽回这类格子并未经过检验——更早那个项目的轨迹提示它能挽回大部分差距（同一评测上被审计过的格子得分远高于仅换模型的格子），但对这一特定的再探针而言，把它当作启发性线索，而非实测结论。只探一格，不重开搜索：如果它在在位者成本之下通过，它就成为确认环节要检验的配置；否则在位者维持原状。

**Step 5 - Final effort re-sample = the registered joint confirm.**
After the prompt climb (and the Step 4 re-probe, if it promoted a cheaper cell),
re-sample effort around the chosen point at n>=3 and make that
run the pre-registered confirm of the full (model, prompt, effort) configuration - the
three adoption gates below, registered before it fires, on held-out cases if any exist.
This is the number to report. (The prompt winner was selected at the earlier effort
point; the direct prompt×effort interaction is unmeasured, so a prompt tuned under rich
thinking may not hold at lower effort - the joint confirm is the insurance.)

**第 5 步——最终 effort 重采样 = 登记的联合确认。**
提示词爬坡（以及第 4 步再探针，若它晋升了更便宜的格子）之后，在所选点附近以 n>=3 重采样 effort，并让这次运行成为完整（模型、提示词、effort）配置的预登记确认——即下方三道采纳门槛，在它发射之前登记，若存在留出案例则用留出集。这才是要报告的数字。（提示词赢家是在较早的 effort 点选出的；提示词×effort 的直接交互未被测量，因此在丰富 thinking下调优的提示词未必在更低 effort 下依然成立——联合确认就是这份保险。）

**Step 6 - Multi-model topologies only behind a task-shape preflight - usually never.**
Across every measured comparison, one strong model at the right effort beat every team
shape on the cost-score plane: cheap tokens pay by *substitution* (the cheap model does
the work instead), never by *addition* (a helper alongside a strong lead) - a strong
lead pays roughly an order of magnitude in its own tokens to consume cheap help. The
bar for any topology candidate is the model×effort frontier from Step 2 ("does this
beat what the effort dial gives for free?"). Documented exceptions worth a preflight:
the executor is constrained to be cheap or non-Claude (then one up-front plan call by
the strong model, with zero mid-run interaction, can pay); the executor is genuinely
weak (advisors pay below the lead's tier, with a floor); or the task has a visible,
checkable artifact (verification transfers; capability does not).

**第 6 步——多模型拓扑只在任务形态预检之后才考虑——通常是从不。**
在所有实测比较中，一个配对 effort 的强模型在成本-得分平面上击败了所有团队形态：便宜 token 的收益来自*替代*（便宜模型代做工作），从不来自*追加*（在强主力旁边加助手）——一个强主力大约要付出高一个数量级的自身 token 才能消费便宜的帮助。任何拓扑候选的门槛都是第 2 步的模型×effort 前沿（"它是否胜过 effort 旋钮免费给出的东西？"）。值得做预检的已记录例外：执行者被限定为便宜或非 Claude（此时强模型做一次零中途交互的事前计划调用可能划算）；执行者确实弱（顾问在主力档位之下支付，且有地板）；或任务有可见、可检查的产物（校验可迁移，能力不可迁移）。

## Adoption gates - register before round 1 / 采纳门槛——第 1 轮之前登记

A candidate change (prompt edit, effort cut, model swap) is adopted only if ALL three
pre-registered gates pass:

一项候选变更（提示词编辑、effort 削减、模型更换）只有在全部三道预登记门槛都通过时才被采纳：

1. **Quality band** - held-out score within a named band of the incumbent (state the
   band before running).
   **质量带**——留出得分落在在位者的某个指名带宽之内（运行前声明带宽）。
2. **Cost margin** - strictly cheaper beyond a registered margin, measured at the
   stated pricing basis.
   **成本边际**——在登记的边际之外严格更便宜，按声明的计费口径测量。
3. **Mechanism** - the *predicted* mechanism appears in the measurements (e.g. "this
   edit removes duplicate lookups" must show up as fewer tool calls, not just a lower
   bill). A cost tie with the right mechanism and a cost win with the wrong mechanism
   are both rejections: the first is an edit that didn't bite, the second is an
   unexplained confound that will not survive contact with production.
   **机制**——*预测的*机制出现在测量中（例如"这次编辑移除了重复查询"必须体现为更少的工具调用，而不只是一张更低的账单）。机制正确但成本打平、与机制错误但成本获胜，都要拒绝：前者是没有咬合的编辑，后者是经不起生产环境接触的、无法解释的混杂因素。

The final joint confirm (Step 5 of the search order) reports against these same gates;
three sequential selections, each made on the data that chose it, overstate the
combined win, so the confirm's number - not the per-round selection scores - is the
headline.

最终联合确认（搜索顺序第 5 步）对照同样的门槛报告；三次顺序选择、每一次都基于选出它的数据，会夸大合计收益，因此标题必须是确认的数字，而不是逐轮选择得分。

## Measurement discipline / 测量纪律

- **Noise bar first.** Before round 1, answer "how big must a delta be to be believed?"
  with a number, from repeat runs of the unchanged config. The same baseline repeats
  feed the round-0 diagnostic of the model×effort walk (Step 2): read output-token
  share and turns per task out of their transcripts while measuring the bar. If the tuned artifact is
  itself a stochastic generation (e.g. a built index or wiki, not a fixed prompt),
  measure *build* variance with a no-change rebuild control before judging any edit.
  **先测噪声条。**第 1 轮之前，用数字回答"偏差要多大才可信？"——依据对未变配置的重复运行。同一批基线重复同时供模型×effort 行走的第 0 轮诊断（第 2 步）使用：在测噪声条的同时，从它们的转录中读出输出 token 占比与每任务轮数。如果被调优的产物本身就是随机生成物（例如构建出的索引或 wiki，而不是固定提示词），先在判断任何编辑之前用"无变更重建"对照测量*构建*方差。
- **Selection set != holdout.** The split whose score picks winners each round is a
  selection set, even if the guide calls it "test". Pre-register confirm runs for the
  headline and expect train->holdout shrinkage.
  **选择集 ≠ 留出集。**每轮凭分数挑选赢家的那部分数据是选择集，哪怕指南把它叫作 "test"。为标题数字预登记确认运行，并预期 train->holdout 缩水。
- **Pricing basis discipline.** Lock and state the pricing basis up front (which price
  sheet, whether cache-adjusted, promo vs standard). The same run's reported cost can
  diverge severalfold across bases - cached input bills at a tenth of the fresh-input
  price, so cache-adjusted and flat accountings of one run separate fast at high
  cache rates. Do paper arithmetic with the rate card before spending:
  it can exclude whole configurations with zero eval runs.
  **计费口径纪律。**预先锁定并声明计费口径（哪张价目表、是否按缓存调整、促销价还是标准价）。同一次运行报告的成本在不同口径下可相差数倍——缓存输入按新鲜输入价格的十分之一计费，所以在高缓存率下，同一次运行的缓存调整核算与平价核算会迅速分开。花钱之前先用价目表做纸面算术：它可以在零评测运行的情况下排除整类配置。
- **Register cost in the objective.** An optimizer optimizes exactly what is
  registered: a quality-only climb raised cost per deliverable by 75% in one measured
  search. If the goal is cost-subject-to-quality, the gates above ARE the objective - 
  write them into the plan sign-off.
  **把成本登记进目标。**优化器优化的恰是登记的东西：一次实测搜索中，仅质量的爬坡把每交付物的成本推高了 75%。如果目标是"质量约束下的成本"，那么上述门槛就是目标——把它们写进计划的签核。
- **One lever per round, frozen arm.** Move exactly one axis per round so wins and
  regressions are attributable.
  **每轮一个杠杆，冻结对照臂。**每轮恰好移动一个轴，收益与回退才可归因。
- **Make the optimizer predict before it measures.** Require, in each round's
  proposal, a point estimate and an 80% interval for every cell - on score AND cost - 
  plus named falsifiers ("if X happens, the lever is dead; say so"). Compare outcomes
  to intervals after each round, and shift and widen the next round's intervals after
  misses. This turns every round into a correction of the optimizer's own predictions: in
  one measured search the optimizer's round-2 cells both landed just below its solved
  intervals; it said so, re-centered, and the falsifier it registered for round 3 is
  what caught the prompt-lever no-win cleanly.
  **让优化器在测量之前先预测。**要求每一轮的提案对每个格子给出点估计与 80% 区间——得分与成本都要——外加指名的证伪条件（"若 X 发生，杠杆已死；如实说明"）。每轮之后把结果与区间比对，未命中后移动并加宽下一轮的区间。这把每一轮都变成对优化器自身预测的修正：一次实测搜索中，优化器第 2 轮的两个格子都恰好落在其已解出的区间之下；它如实说明、重新定心，而它为第 3 轮登记的证伪条件正是干净利落抓到提示词杠杆无收益的东西。
- **Verify serving identity and wiring before believing any arm.** Record the model id
  from the *response*, not the config; confirm usage fields are present per case; run
  on an eval surface that reports them; disable any auto-retry scoring that passes on
  either attempt. A result without wiring receipts is not evidence.
  **在相信任何一臂之前，先核实服务身份与接线。**从*响应*记录模型 id，而不是从配置；确认每个案例都有用量字段；在会报告这些字段的评测表面上运行；禁用任何"两次尝试任一通过即算过"的自动重试计分。没有接线回执的结果不是证据。
- **Audit graders before believing persistent failures.** Re-grading has shrunk a
  claimed +9-point win to +3 in a measured case. When a case fails every round, suspect
  the grader before grinding prompt content at it.
  **在相信持续性失败之前，先审计评分器。**一次实测中，重新评分把一个声称 +9 分的收益缩到了 +3。当某个案例每一轮都失败时，先怀疑评分器，再死磕提示词内容。
- **Routers price only on the full traffic frame.** A difficulty-router evaluated on a
  hard subset self-defeats (everything routes to the big model and you pay the routing
  overhead for nothing); its savings exist only on the full distribution, and are
  paper-only until the predictor is tested.
  **路由器只在完整流量框架下才有价格优势。**在困难子集上评测难度路由器是自我挫败（一切都路由到大模型，而路由开销白付）；它的节省只存在于完整分布上，且在预测器被检验之前只是纸面数字。

## What drives prompt cost (measured mechanics) / 什么在驱动提示词成本（实测机制）

- **Cost scales with extra actions triggered, not prompt length.** In one measured
  decomposition, a single extra tool round added roughly a third of the per-case
  cost, while longer-but-inert prompt text was nearly free - *when cached*.
  **成本随触发的额外动作扩展，而不是随提示词长度。**在一次实测分解中，一个额外的工具轮大约增加三分之一的单案例成本，而更长但不起作用的提示词文本几乎免费——*在缓存命中时*。
- **Census engagement before trimming.** Before editing scaffold instructions, count
  in existing transcripts the artifacts each instruction demands (plan-file writes,
  forced reasoning blocks, capped or repeated reads, per-edit suite runs, narration
  phrases) - and subtract the prompt's own occurrences of each marker, or static text
  masquerades as engagement. Near-zero corrected counts mean the model is ignoring
  that text: dead weight, nearly free while cached, and deleting it will not cut cost.
  Expect mixed pictures - in the measured case the forced-reasoning ritual counted
  zero everywhere while a plan-file instruction was engaged in 17 of 20 runs. A
  two-cell ablation (cleaned opener vs cleaned-plus-ritual) is cheap and settles
  whether a suspect block is load-bearing: here the two cells landed within 2% on cost
  and tied exactly on the held quality gate (the partial-credit diagnostic moved, a
  reminder that "tied" is metric-relative) - the ritual was dead weight at the scale
  the test could detect.
  **修剪之前先做参与度普查。**在编辑脚手架指令之前，先在既有转录中清点每条指令所要求的产物（计划文件写入、强制推理块、封顶或重复读取、每次编辑后的套件运行、旁白短语）——并减去提示词自身对每个标记的出现次数，否则静态文本会伪装成参与。修正后接近零的计数意味着模型在无视那段文本：死重，缓存命中时几乎免费，删掉它不会削减成本。预期混合图景——实测案例中强制推理仪式在所有地方计数为零，而一条计划文件指令在 20 次运行中有 17 次被遵守。一次两格消融（只清理开场白 vs 清理加仪式）很便宜，足以判定一个可疑块是否承重：此处两个格子在成本上相差 2% 以内，且在留出质量门槛上恰好打平（部分得分诊断移动了——提醒"打平"是相对于度量而言的）——在该检验可探测的尺度上，那段仪式是死重。
- **The expensive patterns are action-triggering instructions.** "Verify twice"
  (+48% per-case cost via duplicate lookups and re-deliberation) and "be maximally
  thorough" (+39% via unneeded tool calls) together cost roughly twice as much as all
  other measured cost-adding patterns combined. Audit for instructions that trigger
  redundant actions before trimming words.
  **昂贵的模式是触发动作的指令。**"Verify twice"（通过重复查询与重新深思，单案例成本 +48%）与"be maximally thorough"（通过不必要的工具调用 +39%）两者合计的成本约为其他所有实测增本模式总和的两倍。在修剪文字之前，先审计触发冗余动作的指令。
- **Charge a prompt edit its own token mass at the real cache-adjusted price.** A
  standing directive that rides every request must net positive against its own mass:
  one measured 650-character directive produced exactly the predicted behavior change
  and still only tied on cost, because its per-request mass canceled the saving.
  **按真实缓存调整后的价格，让一次提示词编辑为自己的 token 质量买单。**一条随每个请求出现的常设指令，必须净收益大于其自身质量：一条实测的 650 字符指令精确产生了预测的行为改变，成本上仍只打平，因为它的每请求质量抵消了节省。
- **Brevity caps save money through shorter replies** - a reply-quality tradeoff to
  surface to the user, not a free win. Flag the median reply-length change alongside
  the cost saving.
  **简洁上限通过更短的回复省钱**——这是一个要向用户摆上台面的回复质量权衡，不是免费的胜利。把回复长度的中位数变化与成本节省一起标出。
- **Output tokens are the latency lever too.** In latency-bound products, output-token
  prompting rises in priority: one customer self-reported ~11% output-token cuts with
  quality flat-or-up (a practitioner report, not a benchmark measurement), and streamed
  tokens are directly perceived latency.
  **输出 token 也是延迟杠杆。**在延迟受限的产品中，输出 token 提示的优先级上升：一位客户自报约 11% 的输出 token 削减且质量持平或更好（从业者报告，非基准测量），而流式 token 是被直接感知的延迟。

## Stopping rules / 停止规则

- **Prompt rounds:** wins come in rounds 1-2; stop when two consecutive variants fail
  to beat the incumbent beyond the noise band; cap at ~3-4 rounds per model.
  **提示词轮次：**收益出现在第 1-2 轮；当连续两个变体都无法在噪声带之外击败在位者时停止；每个模型上限约 3-4 轮。
- **Effort:** savings saturate stepwise (each step down saves less while variance
  grows). Stop inside the noise band. Remember lowered effort doesn't fail fixed cases - 
  failures MOVE between runs ("shallower thinking fails wherever the margin is thin"),
  so effort-cut decisions need aggregate non-inferiority over multiple runs, never
  per-case reads.
  **Effort：**节省逐级饱和（每往下一步省得越少，而方差增大）。在噪声带内停止。记住降低 effort 不会让固定案例失败——失败会在运行之间*移动*（"更浅的思考在边际薄的地方失败"），所以削减 effort 的决策需要多次运行上的总体非劣性，绝不要看单案例读数。
- **The model×effort walk:** stops itself - when no unprobed cell projects under the
  incumbent's cost and the extension trigger is quiet, the incumbent is the answer.
  Do not keep probing "to be sure";
  the declared quality-reference cell is the sanctioned way to buy information
  outside the walk.
  **模型×effort 行走：**它会自行停止——当没有任何未探测格子投影在在位者成本之下、且扩展触发器安静时，在位者就是答案。不要"为了保险"继续探测；声明的质量参照格是在行走之外购买信息的正当方式。
- **Overall:** when the joint confirm passes its gates, ship; when it fails, report the
  best gated configuration honestly rather than re-searching on the confirm data.
  **总体：**联合确认通过全部门槛，就发布；失败时，诚实报告最佳的通过门槛的配置，而不是拿确认数据重新搜索。
