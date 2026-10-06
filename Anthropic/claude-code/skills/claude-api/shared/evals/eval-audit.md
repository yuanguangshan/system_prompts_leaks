<!-- BILINGUAL-EN-ZH -->
# Eval health checklist / 评估健康检查清单

This file is loaded whenever an eval is being **built** (`build-eval.md`) or **climbed on** (`eval-hillclimb.md`). It has two jobs. When you are writing the eval, every item below is a construction requirement - the runner, grader, and case set you produce should satisfy it by default, not after someone flags it. When the user brings an existing eval, it is the verification pass you run before building anything on top of it. Either way, run it once more before the first full paid pass and before round 1 of a hillclimb.

每当一个评估正在被**构建**（`build-eval.md`）或被**爬坡调优**（`eval-hillclimb.md`）时，都会加载本文件。它有两个职责。当你编写评估时，下面每一项都是构建要求——你产出的运行器（runner）、评分器和用例集应当默认满足它们，而不是等别人指出后才满足。当用户带来一个现有评估时，它是你在其上构建任何东西之前运行的验证环节。无论哪种情况，在第一次完整付费运行之前和 hillclimb 第 1 轮之前，都请再运行一遍本清单。

Before trusting an eval to tell you which model, prompt, or configuration is better, check that the eval itself is sound. A broken eval produces confident-looking numbers that point in the wrong direction, and a hillclimb over a broken eval just multiplies the misdirection: you will "improve" an artifact and ship nothing. In practice the most surprising eval results usually turn out to be bugs in the eval rather than facts about the model, so an hour of auditing up front routinely saves days of chasing phantom differences.

在信任一个评估来告诉你哪个模型、提示词或配置更好之前，先检查评估本身是否可靠。一个有缺陷的评估会产生看似可信却指向错误方向的数字，而在有缺陷的评估上做 hillclimb 只会把误导放大：你会"改进"一个假象而什么也没交付。实践中，最令人意外的评估结果往往是评估本身的 bug，而不是关于模型的事实，因此事先花一小时审计通常能省下数天追逐幻影差异的时间。

The checks are grouped into **task design** (are the cases right?), **harness design** (is the scaffolding right?), **metrics hygiene** (are cost and latency measured correctly?), **grader design** (is the scoring right?), and **can it detect the change you're after** (is there enough signal for the decision?). They are written as direct instructions: for each, look at the eval's actual code, config, and data, not its README. The final section, **Reporting findings to the user**, covers how to communicate what you find; the checks are declarative, but the report to the human is observations and suggestions, since the eval's author almost always has context that justifies choices an outsider would flag.

这些检查分为五组：**任务设计**（用例对不对？）、**评测框架设计**（脚手架对不对？）、**指标卫生**（成本和延迟测得对不对？）、**评分器设计**（打分对不对？）以及**能否检测到你要找的变化**（支撑决策的信号够不够？）。它们以直接指令的形式书写：对每一项，请查看评估的实际代码、配置和数据，而不是它的 README。最后一节**向用户报告发现**讲的是如何沟通你的发现；检查本身是指令式的，但呈给人的报告应是观察与建议，因为评估的作者几乎总有局外人看不到、却足以解释其选择的背景。

【评论】本清单把"读实际代码与数据、而非 README"作为审计原则；评估文档与实现脱节是常见的误差来源，这一点与一般代码审计的做法一致。

Before auditing further, run the eval once end-to-end on a handful of cases, or find a recent results file. A surprising number of eval-quality discussions turn out to be about code that does not currently run.

在进一步审计之前，先在少量用例上端到端运行一次评估，或找到一份近期的结果文件。相当多的评估质量讨论，最后发现讨论的其实是当前根本跑不起来的代码。

## 1. Task design / 1. 任务设计

These checks concern the cases themselves: what is being asked, what counts as correct, and whether the set as a whole can distinguish between the systems being compared.

这些检查关注用例本身：在问什么、什么算正确，以及整个用例集能否区分被比较的系统。

### Auditing case sets at scale / 大规模审计用例集

The harness and grader are code you can read end to end; the case set may be hundreds of items you cannot. Do not try to read every case inline. Work in three tiers:

评测框架和评分器是你可以从头读到尾的代码；用例集却可能是数百条你读不过来的条目。不要试图逐条通读所有用例。分三个层级工作：

**Tier 1: programmatic checks over the full set.** Write a short script that loads every case and reports: exact- and near-duplicate rate; label or category balance; prompt-length and expected-answer-length distributions; schema validity and missing-field counts; obviously malformed rows. Cheap, exhaustive, and catches skew, duplicates, truncation, and broken rows regardless of set size.

**第一层级：对全集做程序化检查。**写一个简短脚本加载每个用例并报告：精确重复与近似重复率；标签或类别均衡度；提示词长度与期望答案长度分布；schema 有效性与缺失字段计数；明显畸形的行。成本低、覆盖全，且无论集合规模如何都能发现偏斜、重复、截断和坏行。

**Tier 2: stratified sample for a close read.** Draw twenty to fifty cases, stratified across `tags[0]` if it exists, otherwise uniformly at random, and apply the per-case checks below to those. Recommend the user read a handful themselves as well - a second pair of human eyes on raw cases catches things no checklist does. (This is what the build-eval inputs sign-off is for; the report's per-case table is the surface.)

**第二层级：分层抽样细读。**抽取二十到五十个用例，若存在 `tags[0]` 则按其分层，否则均匀随机抽取，并对它们应用下文的逐用例检查。同时建议用户自己也读几个——人眼对原始用例的第二遍审视能发现任何清单都发现不了的问题。（这正是 build-eval 输入签核的用途；报告的逐用例表格是其呈现面。）

**Tier 3: per-case LLM auditor.** For sets beyond a few hundred items, run one isolated model call per case with a tight audit prompt, collect a structured verdict, and aggregate. Ask before running it - the cost is roughly N cheap-model calls - and offer it explicitly: "I can run a per-case auditor over all N cases, ~$X. Want me to?"

**第三层级：逐用例 LLM 审计器。**对超过数百条的集合，对每个用例运行一次隔离的模型调用，配合精炼的审计提示词，收集结构化判定并汇总。运行前先询问——成本大约是 N 次廉价模型调用——并明确提议："我可以对全部 N 个用例运行逐用例审计器，约 $X。需要吗？"

A per-case auditor prompt that works well (adapt field names to the eval's schema):

一个效果良好的逐用例审计器提示词（字段名请按评估的 schema 调整）：

```
You are auditing a single case from an evaluation suite. Given the prompt, the reference answer, and a description of how the grader decides pass/fail, flag any of the following. Be conservative - only flag when reasonably confident.

PROMPT:
{prompt}

REFERENCE ANSWER:
{gold}

GRADER BEHAVIOUR:
{grader_description}

For each issue answer yes/no with a one-line reason if yes:
- ambiguous: could two careful experts reasonably disagree on the correct answer?
- gold_suspect: does the reference answer look wrong, incomplete, or arguable?
- answerable_from_memory: could a well-read model answer this without doing the intended work?
- grader_too_strict: are there clearly correct answers the grader as described would reject?
- grader_too_lenient: are there clearly wrong answers the grader as described would accept?
- trivially_cheatable: is there a shortcut that satisfies the grader without solving the task?
- other: anything else that would make this case's result misleading.

Return JSON: {"case_id": "...", "flags": {"ambiguous": {"flagged": bool, "reason": "..."}, ...}, "overall": "ok" | "review" | "broken"}
```

Cluster by flag type, surface the top issues with example case IDs, and feed them into the report (§6).

按标记类型聚类，列出主要问题及示例用例 ID，并纳入报告（§6）。

The per-case checks (apply to the tier-2 sample):

逐用例检查（应用于第二层级样本）：

- **Unambiguous success criteria.** Would two independent domain experts, shown the same output, agree on pass vs fail? If the criteria admit reasonable disagreement ("write a *good* summary"), scores reflect grader opinion as much as model capability. Note the dual failure: under-specified (a required output, filename, format left unstated) or over-specified (the prompt is a step-by-step recipe, leaving nothing for the model to decide).

  **无歧义的成功标准。**把同一输出展示给两位独立的领域专家，他们能否就通过与不通过达成一致？如果标准允许合理的分歧（"写一个*好的*总结"），得分就会同等地反映评分者意见和模型能力。注意两类失败：欠规定（必需的输出、文件名、格式未说明）或过度规定（提示词成了逐步操作菜谱，模型没有任何决策空间）。

- **Reference solution exists and passes.** Does each case ship with at least one gold answer that actually passes the grader? A 0% pass rate across all variants is more often a broken case than a hard one. Spot-check by running the reference through the grader.

  **存在且能通过的参考解。**每个用例是否至少附带一个确实能通过评分器的标准答案？所有变体 0% 通过率，更可能是用例坏了而不是题目难。抽查方法：把参考答案跑一遍评分器。

- **Ground-truth labels are correct.** Sample ten cases and independently re-derive the expected answers. Widely used benchmarks routinely carry meaningful label error; wrong labels cap measurable accuracy for reasons that have nothing to do with the model.

  **真实标签（ground truth）正确。**抽样十个用例，独立重新推导期望答案。广泛使用的基准测试也常带有可观的标签错误；错误标签会封顶可测得的准确率，而这与模型毫无关系。

- **Where did the ground truth come from?** Ask, and record the answer as a tag: human-written, human-verified, or **a model's outputs - and which model**. If the expected outputs are a model's outputs, reference-match scoring rewards *imitating that model*, not being right; this is worst in a migration, where gold derived from the incumbent makes the incumbent look best by construction and penalises a successor for every stylistic difference. Prefer a rubric or pairwise judge over reference similarity in that case, or have a human verify a sample of the references first. Never use model A's outputs as gold when the question is A vs B.

  **真实标签来自哪里？**询问并把答案记录为标签：人工撰写、人工校验，或**某模型的输出——以及是哪个模型**。如果期望输出是某个模型的输出，参照匹配式打分奖励的是*模仿那个模型*，而不是答对；这在迁移场景中最糟：由现有系统派生的标准答案在构造上就让现有系统显得最好，并因每一点风格差异惩罚继任者。这种情况下应优先使用评分细则（rubric）或成对评审，而非参照相似度，或先让人工校验一部分参考答案。当问题就是 A 与 B 对比时，绝不要用模型 A 的输出当标准答案。

  【评论】标准答案取自被比较模型之一，会同时引入自我偏好与构造性偏置；这类循环性偏置在 LLM 评审研究（如 self-preference 文献）中有系统记录。

- **No annotation artifacts.** Could a trivial baseline score well from surface patterns - question length, keywords, option order - without solving the task? If a no-op or majority-class baseline scores well above chance, the eval is partly measuring the artifact.

  **无标注伪迹（annotation artifacts）。**一个平凡基线能否不解题、仅凭表层模式——问题长度、关键词、选项顺序——取得高分？如果空操作或多类基线得分远高于随机水平，评估就部分地在测量伪迹本身。

- **Label leakage in the prompt.** Does the expected answer, or a near-paraphrase, appear anywhere the model can see - the prompt, few-shot examples, system message, a tool description, a file the agent can read? Common in few-shot setups assembled by copy-pasting from the golden set.

  **提示词中的标签泄漏。**期望答案或其近似改写是否出现在模型可见的任何位置——提示词、few-shot 示例、系统消息、工具描述、智能体可读的文件？在从标准答案集复制粘贴组装 few-shot 的场景中很常见。

- **Answerable from memory.** For cases about real, named entities, can the model answer from parametric memory even though the intent is to test retrieval or tool use? If the goal is whether the model can *do the work*, subjects need to be obscure or synthetic enough that recall alone doesn't carry it.

  **凭记忆即可作答。**对于关于真实命名实体的用例，尽管本意是测试检索或工具使用，模型能否仅凭参数化记忆作答？如果目标是考察模型能否*完成这项工作*，题材就必须足够冷门或合成，使单凭记忆无法应付。

- **Difficulty comes from the problem, not the prompt.** Are hard-looking cases just worded obscurely? Then the score measures prompt-deciphering. Suggest stating the problem plainly and letting the problem itself be hard.

  **难度来自问题本身，而非提示词。**看似困难的用例是否只是表述晦涩？那样的话得分测量的是解读提示词的能力。建议把问题平实地陈述出来，让难度来自问题本身。

- **Agentic cases: symptom, not investigation.** For cases that ask an agent to diagnose or fix something, how much of the investigation is handed over in the prompt? If it already includes the log line, the failing test name, or the file, the eval measures whether the model can read a hint, not find one. Give the agent what a user would plausibly report and let it fetch the rest.

  **智能体型用例：给症状，而不是给调查结果。**对于要求智能体诊断或修复问题的用例，提示词交出了多少调查工作？如果其中已经包含日志行、失败测试名或问题文件，评估测量的就是模型能否读懂提示，而不是找到提示。只给智能体用户大概率会报告的内容，其余让它自己去找。

- **Realistic distribution and interaction shape.** Compare a handful of cases to production traffic. Also check the *shape*: a single-turn eval won't capture effects that only appear in long multi-turn or agentic settings, and vice versa. Name any obvious divergence up front so readers can calibrate how far results transfer.

  **真实的分布与交互形态。**把若干用例与生产流量对比。还要检查*形态*：单轮评估捕捉不到只在长多轮或智能体场景中出现的效果，反之亦然。事先点名任何明显的偏差，让读者能校准结果的可迁移范围。

- **Difficulty headroom.** If results exist, look at the spread. If the baseline already scores ~95%+, the eval cannot discriminate at the top and a hillclimb will mostly move cost or latency - useful, but say so in advance. If everything scores ~0%, there is more often a case or grader bug than a genuinely impossible task.

  **难度余量。**如果已有结果，看分布。如果基线已得分约 95% 以上，评估在顶部失去区分度，hillclimb 将主要移动成本或延迟——有用，但要提前说明。如果所有得分都约 0%，更常见的原因是用例或评分器 bug，而不是任务真的不可能。

- **Saturated evals and what they end up measuring.** Near the ceiling, remaining variance is dominated by format quirks, grader tie-breaking, or mild reward-hacking rather than capability. Flag that the last few points may no longer measure what the eval was built for; suggest harder items.

  **饱和的评估及其最终测量的东西。**接近天花板时，剩余方差主要来自格式怪癖、评分器平局裁决或轻度奖励作弊，而非能力。要指出最后那几分可能已不再测量该评估本要测量的东西；建议加入更难的条目。

- **Class balance.** For classification-style evals, check the label distribution; report the majority-class baseline alongside model scores. When both positives and negatives exist, prefer precision/recall/specificity to accuracy alone.

  **类别均衡。**对分类式评估，检查标签分布；把多数类基线与模型得分并列报告。当正负两类都存在时，优先使用精确率/召回率/特异度，而非单看准确率。

- **Both-directions coverage.** An eval for "does the agent search when it should" also needs "does the agent *not* search when it shouldn't"; otherwise always-search scores perfectly and one-sided evals produce one-sided optimisation. Same for refusals, tool use, escalation.

  **双向覆盖。**评估"该搜索时是否搜索"的评估，也需要"不该搜索时是否*没有*搜索"；否则永远搜索的策略得满分，单侧评估造成单侧优化。拒答、工具使用、升级（escalation）同理。

- **One capability per case (when diagnosis matters).** A case that needs retrieval *and* reasoning *and* formatting shows 0 whenever any one breaks. Fine for a headline number; flag it when the user wants to know *why* variants differ.

  **每个用例对应一种能力（当需要诊断时）。**同时需要检索*和*推理*和*格式化的用例，任何一环断了都显示 0。作为头条数字没问题；当用户想知道变体*为什么*不同时要指出来。

- **Inverted items as a smoke test.** Where a clearly weaker variant outscores a clearly stronger one on an item, it is far more often a case or grader bug than a real inversion - a good place to look closely.

  **以倒置条目做冒烟测试。**当明显更弱的变体在某条目上反超明显更强者时，这更可能是用例或评分器 bug，而非真实的倒置——值得仔细查看的地方。

- **Staleness.** If cases reference live facts (prices, dates, API responses, library versions), when were the gold answers last verified? A currently-correct answer gets marked wrong against a stale key.

  **时效性。**如果用例引用易变事实（价格、日期、API 响应、库版本），标准答案上次校验是什么时候？当答案键过期时，当前正确的答案会被判错。

- **For generated cases: fix the generator, not the filter.** When cases come from a pipeline, problems in the output are symptoms of something upstream; patching individual items leaves siblings of the same bug. Adjust the generator and regenerate.

  **对生成的用例：修生成器，而不是修过滤器。**当用例来自流水线时，输出中的问题是上游问题的症状；逐条修补会留下同一 bug 的兄弟条目。调整生成器并重新生成。

## 2. Harness design / 2. 评测框架设计

These checks concern the code around the model call. The central failure mode is **conflation**: any time a non-model artifact - an infra error, a truncated response, a broken tool, a retry delay - lands in the same column as a genuine model result, the eval attributes to the model something that belongs to the plumbing.

这些检查关注模型调用周边的代码。核心失败模式是**混淆（conflation）**：任何时候，只要一个非模型产物——基础设施错误、被截断的响应、损坏的工具、重试延迟——落进与真实模型结果相同的列，评估就把本属于管道问题的东西归到了模型头上。

- **Infra failures distinguished from model failures.** How does the runner handle a timeout, an API or rate-limit error after retries, an unparseable output, a response cut off at `max_tokens`, a tool that threw, a grader that itself failed? If any of these are silently scored as 0 (or as pass) and mixed in with real answers, the headline is contaminated. Attempts that never produced a scorable output go to an `errors.jsonl` sidecar with a failure class (harness/serving error, timeout, served-model mismatch) - never into `results.jsonl`, where they'd occupy the `(case, rep)` slot, block resume, and score plumbing as a model failure. Rows that did produce output carry `stop_reason` and `status: truncated` when the response hit `max_tokens`, so a clipped answer is counted and shown but not averaged in as wrong. Refusals are a graded outcome, not an error - record them as their own metric so refusal-zeros and capability-zeros aren't summed.

  **基础设施失败与模型失败相区分。**运行器如何处理超时、重试后仍出现的 API 或限流错误、无法解析的输出、在 `max_tokens` 处被截断的响应、抛异常的工具、自身失败的评分器？如果其中任何一种被静默记 0 分（或记通过）并与真实答案混在一起，头条数字就被污染了。从未产生可打分输出的尝试应写入 `errors.jsonl` 侧车文件并带上失败类别（评测框架/服务错误、超时、所服务模型不匹配）——绝不能进 `results.jsonl`，否则它们会占据 `(case, rep)` 槽位、阻塞断点续跑，并把管道问题记成模型失败。确实产生了输出的行，当响应触到 `max_tokens` 时携带 `stop_reason` 和 `status: truncated`，使被裁剪的答案被计数和展示，但不作为错误计入均值。拒答是被评分的结果，不是错误——把它记录为独立指标，使拒答零分与能力零分不被加总。

- **"No answer" is not "negative answer."** Does the grader distinguish the model *asserting a negative* ("no vulnerabilities found") from the model *failing to produce an answer* (empty, crashed, truncated, unparseable)? If both land on the same label, a runner that errors on every input scores identically to one that carefully found nothing. Look for this in detection, classification, and retrieval evals where "none" is a valid answer.

  **"没有答案"不是"否定性答案"。**评分器是否区分模型*断言否定*（"未发现漏洞"）与模型*未能给出答案*（空、崩溃、截断、无法解析）？如果两者落到同一标签，一个对每个输入都报错的运行器将与一个认真查了却一无所获的运行器得分相同。在"无"是合法答案的检测、分类和检索类评估中要特别留意。

- **Clean, isolated state per trial.** Does each (case, rep) start from a fresh environment - no files, rows, git history, env vars, or cached results left from a previous trial? Shared state leaks one case's side effects into another's score, lets an agent read hints from an earlier run, and makes results order-dependent.

  **每次试验的状态干净且隔离。**每个 (case, rep) 是否从全新环境开始——没有上个试验遗留的文件、数据行、git 历史、环境变量或缓存结果？共享状态会把一个用例的副作用泄入另一个的得分，让智能体读到早前运行的提示，并使结果依赖执行顺序。

- **Environment complete and functional.** Does the environment actually have what the task requires - dependencies, fixtures, reachable services? A case that fails for every variant because a package is missing measures the environment. Distinguish from deliberate obstacles.

  **环境完整且可用。**环境是否真的具备任务所需的东西——依赖、夹具（fixtures）、可达的服务？因缺一个包而让所有变体都失败的用例，测量的只是环境。要与有意设置的障碍相区分。

- **Deterministic setup.** Unseeded randomness, unordered iteration that reaches the model or grader, timestamp-dependent paths, stochastic simulators without a fixed seed - these add run-to-run variance unrelated to the system under test. Pin seeds, sort anything whose order matters, and use the sampling parameters you intend for production.

  **确定性的设置。**未固定种子的随机性、以无序迭代到达模型或评分器、依赖时间戳的路径、没有固定种子的随机模拟器——这些会引入与被测系统无关的运行间方差。固定种子，对顺序有影响的任何东西排序，并使用你在生产中打算使用的采样参数。

- **Scaffold limitations separated from model limitations.** A missing tool, a tight step budget, an early-give-up retry policy, or a template that drops context all look like capability gaps from outside. Where practical, vary the scaffold holding the model fixed (or vice versa) to attribute results to the right layer.

  **脚手架局限与模型局限相区分。**缺失的工具、过紧的步数预算、过早放弃的重试策略、会丢弃上下文的模板——从外部看都像能力缺口。在可行处，固定模型而改变脚手架（或反过来），把结果归因到正确的层。

- **Token and context limits won't clip any case.** Compare the longest prompt and longest plausible correct answer against the configured context window and `max_tokens`. Truncation is easy to misread as the model choosing to stop; it must surface as `status: truncated`, not as a wrong answer.

  **Token 与上下文限制不会裁剪任何用例。**把最长提示词和最长的合理正确答案，与配置的上下文窗口和 `max_tokens` 对比。截断很容易被误读成模型主动停止；它必须以 `status: truncated` 呈现，而不是作为错误答案。

- **Transient errors retried with jittered backoff, and retries recorded.** Unretried 429/529s show up as spurious failures and can make one variant or one time of day look worse; a zero-delay retry loop is worse - it multiplies cost invisibly and can turn one 429 into a torn-down batch. Back off with jitter, cap attempts, and record the attempt count per row so retries can be excluded from latency and "attempts run vs attempts scored" is visible in the data, not just the bill. If the runner re-runs whole failed *cases*, decide which attempt's grade lands - default strict (passed-only-on-retry is a fail) - and count every attempt's usage.

  **瞬时错误以带抖动的退避重试，并记录重试。**未重试的 429/529 会表现为虚假失败，并可能让某个变体或某个时段显得更差；零延迟的重试循环更糟——它隐形地放大成本，并可能把一个 429 变成一整批被拆除。请带抖动退避、限制尝试次数，并按行记录尝试次数，使重试可从延迟统计中剔除，且"运行尝试数与计分尝试数"在数据中可见，而不只在账单上。如果运行器会重跑整个失败的*用例*，要决定取哪次尝试的成绩——默认从严（仅重试时通过算失败）——并把每次尝试的用量都计入。

- **A hard per-case wall-clock ceiling, independent of stream liveness.** A hung streaming connection can emit keepalives indefinitely, defeating inactivity timers; only a ceiling on total case time reclaims the worker slot. When it fires the attempt goes to `errors.jsonl` as a timeout, never a zero.

  **硬性的每用例墙钟上限，独立于流是否活跃。**挂起的流式连接可以无限发出 keepalive，从而骗过不活动计时器；只有对用例总时长设上限才能收回 worker 槽位。触发时该尝试作为超时进入 `errors.jsonl`，绝不记为零分。

- **The model that served the request is the model you asked for.** Read `model` from the *response* on a smoke case, then assert it on every call - beyond documented alias->snapshot resolution, a mismatch (a provider fallback, a capacity reroute) fails the attempt loudly. A score served by the wrong model measures nothing, and silent substitution may not surface anywhere else; where the provider exposes usage or billing records, cross-check the aggregate once.

  **服务请求的模型就是你要的模型。**先在冒烟用例上从*响应*读取 `model`，然后在每次调用上断言它——在文档说明的别名->快照解析之外，任何不匹配（提供商回退、容量改道）都要让该尝试响亮地失败。由错误模型产出的分数什么也测不了，而静默替换可能不会在任何其他地方浮现；在提供商提供用量或账单记录处，对总量做一次交叉核对。

- **Eval config matches production config.** Diff the system prompt, tool definitions, model version, sampling parameters, and scaffolding in the eval against what actually ships. The runner must call the app's real entry point; a re-implemented call silently measures a different setup.

  **评估配置与生产配置一致。**把评估中的系统提示词、工具定义、模型版本、采样参数和脚手架，与实际交付的内容做 diff。运行器必须调用应用的真实入口点；重新实现的调用会静默地测量另一套配置。

- **Full per-case trajectories saved.** Every message, tool call and result, and error, per (case, rep) - plus the grader's own inputs and outputs - so a surprising score can be traced to a fact about the model or a bug in the eval without re-running. This is the single highest-leverage habit for a debuggable eval, and it is what the report's Transcripts tab renders (full viewer) or its per-case rows link to (lite).

  **保存完整的逐用例轨迹。**每个 (case, rep) 的每条消息、每次工具调用与结果、以及错误——再加上评分器自身的输入与输出——使意外的分数无需重跑就能追溯到关于模型的事实或评估中的 bug。这是打造可调试评估的最高杠杆习惯，也是报告的 Transcripts 标签页（完整查看器）所渲染、或其逐用例行（精简版）所链接的内容。

- **Multiple trials with variance reported.** A single rep is a point estimate with no error bar; differences smaller than the run-to-run spread are not meaningful. The runner must support reps and reported numbers must carry intervals.

  **多次试验并报告方差。**单次 rep 是没有误差棒的点估计；小于运行间波动的差异没有意义。运行器必须支持多次 rep，报告的数字必须带区间。

- **Reproducible over time.** Dependencies pinned, case set and grader versioned together, environment specified. Scores from before and after a grader change are not comparable.

  **可随时间复现。**依赖固定版本，用例集与评分器一起做版本化，环境写明。评分器变更前后的分数不可比较。

- **Harness tested on known-good and known-bad.** Before the first full pass, run (a) an oracle - the reference answers, or a variant that *should* score near 100% - and (b) a null baseline - empty output, a constant answer, or the majority class - through the whole pipeline. If the oracle doesn't pass, the harness or grader is broken; if the null doesn't fail, the grader is too lenient. Two runs, minutes, and it catches most wiring bugs before they cost a full pass.

  **评测框架在已知好与已知坏的输入上测试过。**第一次完整运行之前，把两样东西跑过整条流水线：(a) 一个 oracle——参考答案，或一个*应该*接近 100% 的变体；(b) 一个零基线——空输出、恒定答案或多数类。如果 oracle 不通过，说明评测框架或评分器坏了；如果零基线不失败，说明评分器太宽松。两次运行、几分钟，就能在接错线的 bug 花掉一整次完整运行之前抓住大部分。

## 3. Metrics hygiene / 3. 指标卫生

Pass rate alone rarely answers the user's real question, which is some form of "what quality can I get for what cost and latency?" Check that each perf metric reflects the model under test rather than the rig around it.

仅有通过率很少能回答用户的真实问题，后者总是某种形式的"以怎样的成本和延迟能换来怎样的质量？"。要检查每项性能指标反映的是被测模型，而不是围绕它的装置。

- **Token accounting from the API, not estimated.** Input, output, cache-read and cache-write tokens per row from the response's `usage` block. String-length estimates are off by enough to reverse a cost comparison.

  **Token 计量来自 API，而非估算。**每行的输入、输出、缓存读与缓存写 token 都取自响应的 `usage` 块。按字符串长度估算的误差足以反转一次成本比较。

- **Cost derived from recorded tokens and the row's actual model** - including cache rates - never a flat assumed rate; and the judge's cost recorded separately (`judge_model`, `judge_usage`) so it neither hides nor dampens differences between variants.

  **成本由记录的 token 和该行实际模型推导**——包括缓存费率——绝不用一刀切的假设费率；评审模型的成本单独记录（`judge_model`、`judge_usage`），使它既不掩盖也不稀释变体之间的差异。

- **Cache hit rate comparable across variants.** If one variant runs warm-cache and another cold, cost and latency differences are partly an artifact of run order. Flag comparisons where cache-read share differs materially.

  **缓存命中率在变体间可比。**如果一个变体跑热缓存、另一个跑冷缓存，成本与延迟差异就部分是运行顺序的产物。对缓存读占比差异显著的比较要作出标记。

- **Latency measured against the right boundaries.** Time the final successful request only; keep client-side retries, backoff sleeps, local queueing behind a semaphore, and post-processing out of the model-latency column (record total wall-clock separately if useful). Otherwise whichever variant hit more transient errors looks slower.

  **延迟按正确的边界测量。**只对最终成功的请求计时；把客户端重试、退避睡眠、信号量后的本地排队和后处理排除在模型延迟列之外（如有用，可单独记录总墙钟时间）。否则哪个变体遇到的瞬时错误多，哪个就显得慢。

- **Per-call breakdown for agentic evals.** Record tokens, cost, and timing per model call and per tool call, not just per episode, or a slow tool is indistinguishable from a slow model.

  **智能体评估要有逐调用分解。**按每次模型调用和每次工具调用记录 token、成本与耗时，而不是只按整个回合记录，否则慢的工具与慢的模型无法区分。

- **Perf reported alongside quality**, per variant, as absolute numbers first - so quality-vs-cost and quality-vs-latency trade-offs are visible rather than implied.

  **性能与质量并列报告**，按变体，先给绝对数——使质量-成本与质量-延迟的权衡可见，而非隐含。

## 4. Grader design / 4. 评分器设计

These checks concern the function that turns an output into a score - exact match, unit test, end-state check, or LLM judge.

这些检查关注把输出变成分数的函数——精确匹配、单元测试、终态检查，或 LLM 评审。

- **Prompt-grader agreement.** Does the grader reward what the prompt asks for? Common drift: prompt says "at least X", grader passes only on strictly more; prompt asks for an explanation, grader checks only the number. This penalises models that follow instructions.

  **提示词与评分器一致。**评分器奖励的是提示词所要求的东西吗？常见漂移：提示词说"至少 X"，评分器只在严格多于时才通过；提示词要求解释，评分器只检查数字。这会惩罚遵守指令的模型。

- **Grades outcomes, not paths.** Does the grader reward reaching the right answer or taking a particular route? Requiring an exact tool-call sequence, phrasing, or intermediate step fails a model that solved it a different valid way. Check the answer is correct and appropriately grounded without dictating the trajectory.

  **评结果，不评路径。**评分器奖励的是到达正确答案，还是走了特定路线？要求精确的工具调用序列、措辞或中间步骤，会让用另一种有效方式解题的模型失败。要检查答案正确且恰当地有据可依，而不规定轨迹。

- **For agents that act on an environment, grade the end state, not the transcript.** Run the task in a disposable workspace, then score what it left behind programmatically - tests pass, the diff applies cleanly, expected files/rows/values are present, nothing off-limits was touched, step and tool-call counts within budget - and layer a rubric or pairwise judge only for the taste dimensions a check can't see (readability, minimality of the diff, quality of the PR description). A judge reading a coding transcript is grading the narration; the environment is the answer.

  **对作用于环境的智能体，评终态，而不是评对话记录。**在一次性工作区运行任务，然后以编程方式为它留下的东西打分——测试通过、diff 可干净应用、预期的文件/行/值存在、没有触碰禁区、步数与工具调用次数在预算内——只对检查看不到的品味维度（可读性、diff 的最小性、PR 描述质量）叠加评分细则或成对评审。评审模型读编码对话记录时评的是解说；环境才是答案。

- **Not overly rigid.** For exact/substring graders: whitespace, casing, `4` vs `4.0`, markdown fences, units, thousands separators, a sentence wrapped around the answer. Normalise both sides or accept a small set of equivalent forms.

  **不过度僵硬。**对精确/子串评分器：空白、大小写、`4` 与 `4.0`、markdown 代码围栏、单位、千位分隔符、把答案包在句子中间。要么对两侧做归一化，要么接受一个小的等价形式集合。

- **Not too lenient.** For test-based graders: are the tests thorough enough to catch wrong answers? Write a deliberately wrong-but-plausible answer and confirm it fails.

  **不过于宽松。**对基于测试的评分器：测试是否足够周全，能抓住错误答案？写一个故意错误但貌似合理的答案，确认它失败。

- **Cheat-resistant.** How could a model satisfy the grader without solving the task - hard-coding the expected output, reading the answer key, special-casing on test names, an empty string a lenient regex accepts, a degenerate policy that technically optimises the metric, injecting instructions into the judge's input? Models under optimisation pressure find these. Close them off.

  **抗作弊。**模型如何能不解题而让评分器满意——硬编码期望输出、读答案键、对测试名做特判、宽松正则会接受的空字符串、在技术上优化了指标的退化策略、向评审的输入注入指令？处于优化压力下的模型会找到这些。把它们堵上。

- **Ground truth not reachable by the model under test.** Not in a file in the sandbox, a checked-out repo, leftover commit history, a grader prompt it can see, or the open web if it has search. This has to be structural; "don't look" in a prompt is not a defence.

  **真实答案不被被测模型触及。**不在沙箱中的文件里、检出的仓库里、残留的提交历史里、它可见的评分器提示词里，也不在开放互联网上（如果它有搜索）。这必须是结构性的；提示词里写"不许看"不是防御。

  【评论】"指令式约束不算防御"与系统提示词对抗提示词注入的思路一致：敏感内容的隔离必须在架构层面成立，不能只靠对模型的嘱咐。

- **Spot-check the failures.** Read a handful of outputs the grader marked wrong. If more than roughly one in ten look like grader errors, fix the grader before any full pass - otherwise you are partly measuring which variant matches the grader's blind spots.

  **抽查判错的样本。**读几个被评分器判错的输出。如果大约每十个中就有一个以上像评分器错误，先修评分器再谈完整运行——否则你部分上是在测量哪个变体恰好匹配评分器的盲区。

- **Deterministic, or with measured variance.** Run the grader on the same output twice. If the result changes, there is grader variance on top of model variance; measure and report it.

  **确定性，或测得其方差。**对同一输出把评分器跑两遍。如果结果变了，那么在模型方差之外还有评分器方差；测出来并报告它。

- **Atomic checks over holistic scores.** Score independent properties as separate metrics (`{correct, formatted, concise}`) rather than one blended number - more reproducible, easier to calibrate, and diagnostic. Prefer a separate judge call per property.

  **原子检查优于整体评分。**把相互独立的属性作为单独指标打分（`{correct, formatted, concise}`），而不是揉成一个混合数字——更可复现、更易校准、且有诊断价值。尽量为每个属性单独调用一次评审。

- **Aggregation matches the question.** Mean is right for typical-case quality; for rare high-stakes behaviours (data deletion, irreversible actions), fail-on-any or worst-case reflects what matters better than a mean diluted by easy cases.

  **聚合方式与问题匹配。**均值适合典型情况质量；对罕见的高风险行为（数据删除、不可逆操作），任一失败即失败或最坏情况，比被简单用例稀释的均值更能反映要害。

- **Partial credit and penalties don't make a degenerate policy optimal.** If penalties for trying and stumbling outweigh the reward for succeeding, "do nothing" wins.

  **部分得分与惩罚不应使退化策略最优。**如果尝试和跌倒招致的惩罚超过成功的奖励，"什么都不做"就成了赢家。

- **Handles large outputs.** The grader must not truncate, time out, or crash on the longest output a model might produce; a grader crash is `status: error`, not a model failure.

  **能处理大输出。**评分器不得在模型可能产生的最长输出上截断、超时或崩溃；评分器崩溃是 `status: error`，不是模型失败。

### When the grader is an LLM judge / 当评分器是 LLM 评审时

- **Position bias.** In pairwise comparison, randomise A/B per case (or score both orders and average).
  **位置偏差。**成对比较中，按用例随机化 A/B（或对两种顺序都打分再平均）。
- **Verbosity bias.** Tell the judge not to reward length for its own sake, or judges reliably prefer longer answers.
  **冗长偏差。**要告诉评审不要为长度本身给分，否则评审会稳定地偏好更长的答案。
- **Self-preference.** A judge from the same family as a model under test tends to prefer outputs that resemble its own; avoid the exact model under test as its own judge, and consider a different family or a small jury for close calls.
  **自我偏好。**与被测模型同门的评审往往偏好与自己相似的输出；避免让被测模型给自己当评审，并在难分高下的场合考虑换一个家族或用小型评审团。
- **Label deference.** Don't tell the judge which response is the "reference", "baseline", or "human" one.
  **标签顺从。**不要告诉评审哪个回答是"参考"、"基线"或"人类"的。
- **Concrete rubric, not vibes.** Specific checkable properties, not "which is better?"; treat candidate text as untrusted data, not instructions; use structured output so the parse is deterministic.
  **具体的评分细则，不凭感觉。**用具体可检查的属性，而不是"哪个更好？"；把候选文本当作不可信数据，而不是指令；使用结构化输出使解析具有确定性。
- **Calibrated against human labels.** Validate the judge on a few dozen cases a human labelled independently and report agreement. Well below ~90% on clear-cut cases means the judge prompt needs another iteration before its scores can steer changes - and this number is what lets a cautious owner trust a judge-graded taste metric at all.
  **对照人工标签校准。**在几十个人工独立标注的用例上验证评审并报告一致率。在清晰用例上明显低于约 90%，意味着评审提示词需要再迭代一轮，它的分数才能用来指导改动——而且正是这个数字让谨慎的负责人敢于信任一个由评审打分的品味型指标。
- **Tested on known negatives.** Feed the judge an empty string, "I don't know", and a confident answer to the wrong question; confirm it fails all three.
  **在已知坏样本上测试过。**给评审喂空字符串、"我不知道"、以及对错误问题的自信回答；确认三者都被判失败。

## 5. Can it detect the change you're after? / 5. 它能否检测到你要找的变化？

An eval can be correct on every item above and still be useless for the decision at hand because it lacks the resolution to see the effect. Check this *before* the first full pass and again before round 1 of a hillclimb - discovering it after several paid rounds is the expensive way.

一个评估可以在上述每一项上都合格，却仍对手头的决策没用——因为它缺乏看清该效应所需的分辨率。在第一次完整运行*之前*检查这一点，并在 hillclimb 第 1 轮之前再查一次——在几个付费轮次之后才发现，是最昂贵的路径。

- **Noise floor vs headroom vs the smallest change worth acting on.** From the baseline run at its actual rep count, compute the noise floor on the target metric - the half-width of the paired-difference 95% CI at the current n × R (for a pass-rate, roughly `1/sqrt(n·R)`: 25 cases × 2 reps ~ ±14 points, 100 × 2 ~ ±7). Put it next to the **headroom** (ceiling minus baseline) and the **smallest improvement the user would actually ship on**. If the noise floor exceeds either, say so plainly now, with the numbers, and offer the levers in order of cost: more reps (cheapest, and paired designs make them go further), more cases, a pairwise or continuous metric instead of a binary one, or trimming to discriminating cases for iteration with a full-set confirm at the end. Cases and reps are two knobs on the same dial; budget them together when the set is sized, not after.

  **噪声底 vs 余量 vs 值得行动的最小变化。**从基线运行（按其实际 rep 数）出发，计算目标指标上的噪声底——当前 n × R 下成对差异 95% 置信区间的半宽（对通过率约为 `1/sqrt(n·R)`：25 用例 × 2 次 rep ≈ ±14 分，100 × 2 ≈ ±7）。把它与**余量**（天花板减基线）和**用户真正会为之交付的最小改进**并列。如果噪声底超过其中任何一个，现在就直说，并给出数字，按成本从低到高提供可调手段：更多 reps（最便宜，且成对设计能放大其效果）、更多用例、用成对或连续指标替代二元指标，或裁剪到有区分度的用例用于迭代、最后用全集确认。用例数与 rep 数是同一个旋钮上的两个档位；在确定集合规模时就一起预算，而不是事后。

- **Train and test are the same population.** If the eval will be hillclimbed with a split, the split must be drawn at random (stratified by `tags[0]`), never by baseline score. A train slice hand-picked from the worst-scoring cases guarantees two things: the analyzer only ever sees pathological cases, so its fixes target the tail rather than the population; and selecting on low baseline scores buys regression to the mean - those cases "improve" on re-run by chance alone. The symptom is a healthy train gain with a flat held-out set. Check at baseline that train and test means agree within noise; if they don't, re-draw before round 1. The analyzer can still *focus* on failures within train.

  **训练集与测试集是同一总体。**如果评估将用切分来做 hillclimb，切分必须随机抽取（按 `tags[0]` 分层），绝不能按基线分数抽。从得分最差的用例中手工挑出的训练切片必然带来两件事：分析器只会看到病态用例，其修复针对的是尾部而非总体；以及按低基线分数选择会引入向均值回归——那些用例重跑时"改善"纯属概率。症状是训练集增益可观而保留集纹丝不动。在基线处检查训练与测试均值在噪声范围内一致；若不一致，第 1 轮之前重新抽取。分析器仍然可以*聚焦*训练集内的失败。

- **The mechanism is wired.** Whatever the score is supposed to depend on - a tool, a memory store, an instruction file - disable it and confirm the score drops; enable it and confirm it engages in the transcripts. If the score barely moves either way, the eval isn't measuring the lever you plan to pull.

  **机制确已接通。**无论分数理应依赖什么——一个工具、一个内存存储、一个指令文件——禁用它并确认分数下降；启用它并确认它在轨迹中实际生效。如果分数怎么都不怎么动，评估就没有在测量你打算拉的那根杆。

- **The headline recomputes from raw rows.** Recompute the number you will report from per-case `grade` values yourself; don't trust an aggregate field. Mean-vs-sum and per-rep-vs-per-case mixups produce phantom breakthroughs.

  **头条数字可由原始行重算。**自己从逐用例 `grade` 值重算将要报告的数字；不要信任聚合字段。均值与加总混淆、逐 rep 与逐用例混淆，会制造幻影级突破。

## 6. Reporting findings to the user / 6. 向用户报告发现

The checks above are directives to you; the report you hand the human should not read as one. The person who built the eval almost always has context you lack - a constraint, a deadline, a deliberate trade-off - and the purpose is to surface things worth a second look, not to grade their work.

上述检查是给你的指令；你交给人的报告不应读起来也像指令。评估的作者几乎总有你欠缺的背景——一个约束、一个期限、一个有意的权衡——报告的目的是浮出值得再看一眼的东西，而不是给他们的工作打分。

- **Frame findings as observations and suggestions.** "Something worth looking at is...", "you might consider...", "one thing that can cause trouble here is..." over "this is wrong." State what you observed, why it might matter, and one concrete change, then let the user decide.
  **把发现表述为观察与建议。**优先用"有一处值得看看的是……"、"你可以考虑……"、"这里有个容易惹麻烦的东西是……"，而不是"这是错的"。说明你观察到了什么、为什么可能重要、以及一个具体改动，然后让用户决定。
- **Distinguish severity.** Lead with things likely to make the numbers actively misleading - infra errors scored as failures, "no answer" conflated with "negative", reachable ground truth, gold derived from a model under comparison, a split selected by score, a noise floor larger than the effect sought, a non-deterministic judge - then things that add noise or limit generality without flipping conclusions.
  **区分严重程度。**先说可能让数字主动误导人的事——基础设施错误被记为失败、"没有答案"与"否定性答案"混同、真实答案可被触及、标准答案取自被比较的模型之一、按分数选的切分、噪声底大于要找的效应、不确定的评审——再说那些只增加噪声或限制普适性、但不翻转结论的事。
- **Be specific and cite evidence.** The file, function, case ID, or transcript line. "Case 14's expected answer looks stale - the library changed its default in v3" is actionable; "some labels may be stale" is not.
  **具体并引用证据。**给出文件、函数、用例 ID 或轨迹行。"用例 14 的期望答案看起来过期了——该库在 v3 改了默认值"是可行动的；"有些标签可能过期了"不是。
- **Say when things are fine.** An audit that finds nothing wrong is a valid result. Don't manufacture concerns.
  **没问题也要说。**审计一无所获也是有效结果。不要制造忧虑。
- **Don't be preachy or exhaustive.** Report the handful of things that matter for the decision the user is making; listing every deviation from an ideal buries them.
  **不说教、不求全。**报告对用户正在做的决策重要的那几件事；罗列每一点与理想的偏差反而会淹没它们。
- **Offer to fix, not just flag.** Where a finding is a small change - add a `status` field, pin a seed, normalise before comparing, randomise A/B, re-draw the split - make it.
  **提供修复，而不只是标记。**当发现对应一个小改动时——加一个 `status` 字段、固定一个种子、比较前先归一化、随机化 A/B、重抽切分——直接做好。

Two practices worth suggesting regardless of what the audit finds: treat the eval as a living suite (new production failure modes become cases, saturated items are hardened, the judge is re-calibrated when it drifts); and periodically have a strong model read the cases, rubric, and a few graded transcripts and ask where a reasonable person would disagree with the label - the tier-3 auditor is the scaled-up version of that.

无论审计结果如何，都值得建议两个做法：把评估当作活的套件（新的生产失败模式变成新用例，饱和条目被加固，评审漂移时重新校准）；以及定期让一个强模型阅读用例、评分细则和若干已评分的轨迹，问它哪里讲道理的人会不同意该标签——第三层级审计器就是这一做法的放大版。
