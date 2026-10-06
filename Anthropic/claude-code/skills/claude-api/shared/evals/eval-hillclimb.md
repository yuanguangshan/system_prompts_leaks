<!-- BILINGUAL-EN-ZH -->

# Hill-Climbing on an Eval / 在评测上做爬山优化

> **If you arrived via `/claude-api hillclimb`:** this is the right file. Work through the steps in order - Steps 0 and 0.5 are hard prerequisites; Steps 1-2 feed the plan sign-off you need before the loop starts. Don't summarize this guide; execute it.

> **如果你是通过 `/claude-api hillclimb` 来到这里的：**你找对了文件。请按顺序执行各步骤——步骤 0 和 0.5 是硬性前提；步骤 1-2 为循环开始前所需的计划签核提供输入。不要总结本指南；直接执行它。

This guide is for iteratively improving a Claude-powered app against a fixed eval: run the eval, read the failures, change something in the codebase, run again, and repeat until the score stops moving or the budget runs out. It picks up where eval-building leaves off - the user has a way to measure; now they want to move a number: usually the quality score up, but just as often cost or latency down while quality holds.

本指南用于针对一个固定评测迭代改进基于 Claude 的应用：运行评测、阅读失败案例、在代码库中修改一些内容、再次运行，并重复这一过程，直到分数不再变动或预算耗尽。它承接评测构建完成之后的阶段——用户已经有了度量手段；现在他们想推动某个数字：通常是提升质量分数，但同样常见的是在质量保持不变的前提下降低成本或延迟。

The loop itself is simple. What makes it work or not is discipline: reading the actual transcripts rather than pattern-matching on summary stats, knowing which change produced which effect, keeping a clean record of what was tried, and not fooling yourself by tuning on the same cases you score on. **The number that matters at the end is the test-set delta versus the starting point** - improvement on the cases you read while iterating is not the result, it's the process. Those are the things this guide is opinionated about. Everything else - what to change, how many rounds to run, when to stop - is the user's call, and you should ask rather than assume.

循环本身很简单。决定其成败的是纪律：阅读真实的对话记录而非对汇总统计做模式匹配，弄清哪个改动产生了哪个效果，对尝试过的内容保持干净的记录，并且不要靠在与评分相同的用例上调优来欺骗自己。**最后真正重要的数字是测试集相对起点的增量**——在你迭代时阅读的那些用例上的改进不是结果，而是过程。这些是本指南坚持立场的方面。其余一切——改什么、跑多少轮、何时停止——由用户决定，你应该询问而不是自行假设。

Stay recommendation-forward - propose a concrete default with every question so the user can just say "yes" - and treat the plan sign-off at the end of Step 2 as the minimum approval you need before the loop starts. How often to check in during the loop is one of the Step 2 questions; don't ask it separately here.

始终以建议为先——每个问题都附带一个具体的默认方案，让用户只需说"是"——并把步骤 2 末尾的计划签核视为循环开始前所需的最小批准。循环期间多久汇报一次是步骤 2 的问题之一；不要在这里单独询问。

> **Talking to the user.** These steps are your execution plan, not a script to narrate. Keep user-facing messages short and outcome-focused: what you ran, the score, what you'll try next, a path or link to open. Don't walk the user through which step you're on, which files you're writing, or internal bookkeeping unless they ask. One concise update per round is enough; put detail in `report.html` and `narrative.md`, not the chat. When you need a decision - what's in scope to change, which change to try next, whether to spend another round - use the `AskUserQuestion` tool rather than free-text prose: batch up to four related questions into one call, give each two to four concrete options with your recommendation listed first and labelled "(Recommended)", and don't add your own "Other" option - the tool appends a free-text one automatically. If `AskUserQuestion` isn't available (headless runs), fall back to one short question at a time.

> **与用户沟通。**这些步骤是你的执行计划，不是要朗读的脚本。面向用户的消息要简短并以结果为中心：你运行了什么、分数是多少、接下来尝试什么、可以打开的路径或链接。除非用户问起，否则不要向他们逐一说明你处于哪一步、正在写哪些文件或内部记账情况。每轮一条简明更新即可；细节放进 `report.html` 和 `narrative.md`，而不是聊天里。当你需要决策——什么在可修改范围内、下一步尝试哪个改动、是否再花一轮——使用 `AskUserQuestion` 工具而非自由文本：把最多四个相关问题合并到一次调用中，每个问题给出两到四个具体选项，你的推荐排在第一位并标注 "(Recommended)"，不要自行添加 "Other" 选项——该工具会自动附加一个自由文本选项。如果 `AskUserQuestion` 不可用（无头运行），退而一次只问一个简短问题。

> **Run the eval command in the background; keep the conversation free.** An eval run can take minutes to hours - don't make the user sit through it, but don't wrap it in a subagent either. Each round, do the quick parts yourself in the main session - apply the change, write `vN/change.*` - then launch the runner - the eval command itself, not an `Agent` - as **one `Bash` call with `run_in_background: true`** and a `timeout` that covers the run (normally at most 7200000 ms; with no `timeout` a background command is stopped after 30 minutes; if the runner is stopped at its limit, re-launch the same command - resume is idempotent). You'll get a completion notification when it exits; meanwhile stay available for the user's questions and, when useful, spawn the analyzer subagent (Step 4) - the only subagent this loop uses. When the runner finishes, **verify on disk before trusting the notification**: `vN/results.jsonl` should have N×R rows and `vN/summary.json` should exist; if rows are short, re-launch the same command (resume is idempotent at (case, rep), so it picks up where it stopped). Then regenerate `report.html` via the report builder (full or lite, per build-eval.md), tell the user the score and the path, and pick the next change - via `AskUserQuestion` if the Step 2 cadence has you checking in now, otherwise just state it and proceed. Before the first unattended round, show the user the exact runner command and ask them to allow it for the session, so a permission prompt can't stall a round nobody is watching. Be clear with the user about what that allowlist entry is: it is the security boundary for the loop - whatever that command executes runs under their standing approval - so keep it to the one runner command, and keep the runner's dependencies pinned (a lockfile, `npm ci`). The scaffold's harness-integrity gate is a change detector layered on top, not a second boundary: the runner hashes itself, any lockfile beside it or in the directory it is run from, plus `_state.json.harness_paths`, and exits 2 when the hash differs from the one last recorded with `--approve-harness`, so a round that edits a listed harness file - by accident or by an analyzer proposal - stops the next run until the user has seen the diff and said OK. It does not cover `node_modules/`, interpreter or SDK versions, or any file not listed in `harness_paths`, and it cannot stop an agent that can already write anywhere the user can; the allowlist scope is what does that. Never pass `--approve-harness` from an unattended round; it is the user's to run. If you're running headless (`-p` / SDK) there's no conversation to keep free. In a one-shot `-p` run, run the eval in the foreground, because background work is stopped when the run ends; in an SDK session that stays open, launch it in the background exactly as above. When the user asks how a run is going, answer from the runner's own progress line (the scaffold prints `k/N done ... ~Ns left` every 30 s and mirrors it to `vN/progress.txt`) - one line, not a narration.

> **在后台运行评测命令；保持会话空闲。**一次评测运行可能耗时几分钟到几小时——不要让用户干等，但也不要把它包进子代理。每轮先在主会话中亲自完成快速部分——应用改动、写 `vN/change.*`——然后启动运行器——即评测命令本身，而不是 `Agent`——作为**一次带 `run_in_background: true` 的 `Bash` 调用**，并设置覆盖整个运行的 `timeout`（通常最多 7200000 ms；不设 `timeout` 时后台命令会在 30 分钟后被停止；如果运行器在其上限处被停止，重新启动同一条命令即可——恢复是幂等的）。它退出时你会收到完成通知；在此期间保持可随时回答用户的问题，并在有用时启动分析器子代理（步骤 4）——这是本循环唯一使用的子代理。运行器结束后，**先在磁盘上核实再相信通知**：`vN/results.jsonl` 应有 N×R 行且 `vN/summary.json` 应存在；如果行数不足，重新启动同一条命令（恢复在 (case, rep) 粒度上幂等，因此它会从停下的地方继续）。然后用报告构建器重新生成 `report.html`（完整版或精简版，见 build-eval.md），告诉用户分数和路径，并挑选下一个改动——如果步骤 2 的节奏要求此刻汇报就通过 `AskUserQuestion`，否则直接说明并继续。在第一轮无人值守的运行之前，向用户展示确切的运行器命令并请他们为该会话放行，这样权限提示就不会卡住一轮无人看守的运行。要向用户讲清这条放行条目意味着什么：它是整个循环的安全边界——该命令执行的任何内容都在其常设批准下运行——因此只放行这一条运行器命令，并把运行器的依赖固定住（lockfile、`npm ci`）。脚手架的 harness 完整性门是叠加在其上的变更探测器，不是第二道边界：运行器会对自身、其旁边或运行目录中的任何 lockfile，以及 `_state.json.harness_paths` 做哈希，并在哈希与上次用 `--approve-harness` 记录的不一致时以退出码 2 结束，因此任何对所列 harness 文件的编辑——无论是意外的还是分析器提议的——都会让下一轮停止，直到用户查看差异并点头。它不覆盖 `node_modules/`、解释器或 SDK 版本，也不覆盖任何未列入 `harness_paths` 的文件；它也拦不住一个已经拥有与用户同等写入权限的代理——真正起作用的是放行范围本身。

【评论】这里把会话级命令放行清单明确定义为整个循环的安全边界，而 harness 哈希门只定位为"变更探测器"；职责分离的意图是既防外部注入篡改评测代码，也防代理在无人值守时扩大可执行范围。

---

## Step 0: Confirm there's a runnable eval / 步骤 0：确认存在可运行的评测

Ask the user:

询问用户：

> Do you have an eval script for this flow - something I can run from the command line that exercises the app against a fixed set of prompts and prints a score?

> 你是否已有针对该流程的评测脚本——我能从命令行运行、用一组固定提示词驱动应用并打印分数的东西？

If yes, ask for the command and where it writes its per-case results. Also ask whether the runner retries failed cases - and if so, which attempt's grade, transcript, and usage land in the results; score strict per attempt (a case that passed only on retry is a fail unless the user decides otherwise) and make sure retry attempts show up in cost accounting rather than being silently absorbed. Make sure the eval measures the outcome you're trying to improve, not just a behavior you assume correlates with it. If you're using a behavioral proxy because the real outcome is too expensive to measure every round, say so up front and run the winner on the real eval once at the end. **Each eval run must capture, per case: the full transcript, `model`, `usage` (input/output/cache token counts), and the grade dict** - not just an aggregate score. The transcript and the score for a case must come from the *same* model call; don't re-run the model separately to collect a transcript and then grade a different sample. Search the user's codebase for an existing runner or wrapper for this eval that already persists those fields before building anything new. Run it once to confirm it works and see the output shape (or work from a recent results file if one's handy). Then **read `shared/evals/eval-audit.md` and run it against the eval** - cases, runner, grader - reporting per its §6; an eval that wasn't built by `build-eval` hasn't had these checks, and Step 0.5 below is the must-pass subset, not the whole list.

如果有，询问命令是什么、逐用例结果写在哪里。还要询问运行器是否重试失败用例——如果是，结果中记录的是哪一次尝试的评分、对话记录和用量；按每次尝试严格评分（仅靠重试通过的用例记为失败，除非用户另行决定），并确保重试尝试计入成本核算，而不是被悄悄吸收。确保评测度量的是你想要改进的那个结果本身，而不是你假设与之相关的某种行为。如果因为真实结果每轮度量成本过高而使用行为代理指标，要事先说明，并在最后让获胜版本在真实评测上完整运行一次。**每次评测运行必须逐用例捕获：完整对话记录、`model`、`usage`（输入/输出/缓存 token 计数）以及评分字典**——而不只是一个汇总分数。一个用例的对话记录和分数必须来自*同一次*模型调用；不要为了收集记录单独重跑模型，然后再给另一个样本评分。在构建任何新东西之前，先在用户代码库中搜索是否已有能持久化这些字段的评测运行器或封装。运行一次以确认其可用并查看输出结构（如果手头有近期结果文件，也可以基于它工作）。然后**阅读 `shared/evals/eval-audit.md` 并将其用于审查该评测**——用例、运行器、评分器——按其 §6 汇报；不是由 `build-eval` 构建的评测没有做过这些检查，而下面的步骤 0.5 是必须通过的部分，并非完整清单。

If no, **stop here**. Hill-climbing without an eval is just editing and hoping. Route the user to `/claude-api build-eval` (read `shared/evals/build-eval.md` and run that flow), and come back when there's a script and a baseline number.

如果没有，**就此停止**。没有评测的爬山只是在盲目编辑并祈祷。把用户引导到 `/claude-api build-eval`（阅读 `shared/evals/build-eval.md` 并执行该流程），等有了脚本和基线数字再回来。

If the reason for hill-climbing is a model migration - the user wants to move to a newer Claude model and tune their prompts for it - read `shared/model-migration.md` alongside this guide so your proposed changes account for the new model's breaking changes and behavioral shifts.

如果爬山的起因是模型迁移——用户想换到更新的 Claude 模型并为它调整提示词——请连同本指南一起阅读 `shared/model-migration.md`，让你提议的改动考虑到新模型的破坏性变更和行为偏移。

---

## Step 0.5: Prove the eval can be climbed / 步骤 0.5：证明评测可被爬升

A runnable eval (Step 0) is not yet a *trustworthy* one. Before you spend a round, rule out the possibility that the harness is lying to you - a hill-climb on a broken measurement is worse than none, because you'll "improve" an artifact, declare victory, and ship nothing. `eval-audit.md` (loaded in Step 0, or by `build-eval` if that's how the eval was made) is the full checklist; the checks below are the ones that must pass before round 1, each of which has silently wrecked a run:

可运行的评测（步骤 0）还谈不上*可信*。在花费一轮之前，先排除测试框架在欺骗你的可能——在坏掉的度量上爬山比不爬更糟，因为你会"改进"一个假象、宣布胜利、却什么也没交付。`eval-audit.md`（步骤 0 中加载，或当评测由 `build-eval` 生成时由其加载）是完整清单；下面的检查是第 1 轮之前必须通过的项目，每一项都曾悄悄毁掉一次运行：

- **Prove the eval can detect the win you're after.** From the baseline at its actual rep count, put three numbers in front of the user: the **noise floor** on the Step 1 target (paired-difference CI half-width at the current n × R - `eval-audit.md` §5 has the arithmetic), the **headroom** (ceiling minus baseline), and the **smallest improvement they'd act on**. If the noise floor is bigger than either, the loop cannot show a real in-scope win no matter how good the changes are - say so now, not after five rounds, and offer more reps, more cases, or a finer-grained metric before starting.
  **证明评测能检测出你追求的那种胜利。**以基线在其实际重复次数下的表现为准，把三个数字摆在用户面前：步骤 1 目标的**噪声底**（在当前 n × R 下配对差值置信区间的半宽——`eval-audit.md` §5 给出了计算方法）、**提升空间**（上限减去基线），以及**他们会采取行动的最小改进量**。如果噪声底比其中任何一个都大，那么无论改动多好，循环都无法展示一次真实的目标内获胜——现在就说出来，而不是五轮之后，并在开始前提供增加重复次数、增加用例或改用更细粒度指标的选项。

- **Prove the mechanism is actually wired.** Whatever the score depends on - a memory store the agent writes to, a tool it should call, a file it should read - run a one-off probe that it takes effect end-to-end before you trust any score: write a value and read it back through the same path the eval uses, or confirm the tool actually shows up in the agent's tool list. If the eval comes back as though the mechanism does nothing, check the wiring before concluding "the model can't do this."
  **证明机制确实接通了。**无论分数依赖什么——代理写入的记忆存储、它应当调用的工具、它应当读取的文件——在信任任何分数之前，先运行一次性探针验证其端到端生效：写入一个值并通过评测所用的同一路径读回，或确认该工具确实出现在代理的工具列表中。如果评测表现得像机制毫无作用，先检查接线，再下"模型做不到"的结论。

- **Recompute the headline number from raw per-case results.** Don't trust an aggregate field in a manifest - recompute the number you'll report from the per-instance values in `results.jsonl` yourself. Mean-vs-sum and similar aggregation mixups produce spectacular phantom results that look exactly like a breakthrough until you hand-check them. A too-good-to-be-true number is a measurement bug until a manual cross-check says otherwise; make the cross-check a step, not a lucky catch.
  **从原始逐用例结果重新计算头条数字。**不要相信清单文件里的汇总字段——自己从 `results.jsonl` 的逐实例值重新计算你要报告的数字。均值与求和混淆等类似的聚合错误会产生惊人的幻影结果，在你手工核查之前看起来就像一次突破。好得不真实的数字在人工交叉核对证明相反之前都是度量缺陷；把交叉核对做成一个步骤，而不是碰巧被抓住。

- **Spot-check the grading on the baseline failures.** For a handful of the lowest-scoring baseline cases, read the model's actual output, the judge's reasoning, and the expected value: did the judge grade fairly, and is the ground truth correct? A wrong rubric or wrong expected value will send every round chasing a harness fix for a measurement error. If you find one, fix the rubric/GT and re-grade the baseline in place (re-run the judge on the stored transcripts - no model re-run needed) before round 1. While you're there, triage *every* zero-scoring baseline case: classify each as harness error vs. grader verdict (read the per-case error field or errors sidecar, wherever the runner records failures), and exclude the harness-error cases from the scored denominator before round 1. Spot-check the graders your gates depend on: confirm each reads the artifact the agent actually writes, and that nothing outside the fixture can flip it (host repo state, pre-seeded files, wall clock) - a grader that escapes its fixture measures the environment, not the agent.
  **对基线失败用例的评分做抽查。**挑几个得分最低的基线用例，阅读模型的实际输出、评审的推理和期望值：评审打分是否公正，标准答案是否正确？错误的评分规则或错误的期望值会让每一轮都在为一次度量误差追修测试框架。如果发现这类问题，在第 1 轮之前修正评分规则/标准答案并就地重新评分基线（对已存储的对话记录重跑评审——无需重跑模型）。顺带对*每一个*零分基线用例做分诊：把每个用例归类为框架错误还是评分判定（读取逐用例错误字段或 errors 侧车文件，即运行器记录失败的任何位置），并在第 1 轮之前把框架错误用例从评分分母中剔除。抽查你的门控所依赖的评分器：确认每个评分器读取的是代理实际写出的产物，且夹具之外的任何东西（宿主仓库状态、预置文件、墙钟时间）都无法翻转其结果——一个逃出夹具的评分器度量的是环境，而不是代理。

- **Verify what served the requests and how the runner retries.** Run one smoke case and read `model` from the *response*, not your config - silent server-side substitution invalidates every comparison - and confirm request retries back off with jitter and are counted, not absorbed. `runner-scaffold.mjs` asserts both by default; a user-supplied runner needs the check (`eval-audit.md` §2-3).
  **核实实际服务请求的模型以及运行器的重试方式。**运行一个冒烟用例，从*响应*而不是你的配置中读取 `model`——服务端静默替换会使所有比较失效——并确认请求重试带抖动地退避且被计数，而不是被吸收。`runner-scaffold.mjs` 默认断言这两点；用户自带的运行器需要补上该检查（`eval-audit.md` §2-3）。

- **If the artifact you score is generated from the artifact you tune, measure its build variance first.** Some flows put a stochastic generation step between the lever and the score: the prompt you're iterating on *builds* something - a memory store, a retrieval index, a synthesized corpus - and the eval then scores reads against the built thing. When that build runs once per variant and every rep reads the same build, reps and their CIs measure only the noise of scoring a fixed build; the build's own run-to-run variance is sampled once per variant, invisible to every gate, and can be the larger term. Before round 1, rebuild the baseline artifact two or three times with the prompt *unchanged* and score each build the same way: the spread across those no-change rebuilds is the floor a one-edit effect has to clear. In one climb, three builds of the same prompt spanned ~7 points against rescore noise near ±1.4 on the train mean - every edit had been compared against a single baseline build, and the loop could not tell any of them from the default. If the build spread exceeds a plausible one-edit effect, build K times per variant and compare build-pooled means, or move the lever closer to the score; adding reps over one build can't see it.
  **如果你评分的产物是从你调优的产物生成的，先度量其构建方差。**有些流程在杠杆与分数之间放了一个随机生成步骤：你正在迭代的提示词会*构建*某个东西——记忆存储、检索索引、合成语料——然后评测对读取这个构建好的东西打分。当该构建每个变体只运行一次、每次重复都读取同一个构建时，重复次数及其置信区间只度量了给固定构建打分的噪声；构建自身的逐次运行方差每个变体只采样一次，对所有门控都不可见，却可能是更大的一项。在第 1 轮之前，保持提示词*不变*，把基线产物重建两三次，并以相同方式给每次构建打分：这些无改动重建之间的离散度就是单次编辑效应必须越过的下限。在一次爬山中，同一提示词的三次构建在训练均值上的差异约 7 分，而重评分噪声约 ±1.4——每次编辑都在与单一基线构建比较，循环无法把其中任何一次与默认值区分开。如果构建离散度超过一次编辑可能带来的效应，就每个变体构建 K 次并比较按构建合并的均值，或者把杠杆移到离分数更近的位置；在单一构建上增加重复次数看不见这种方差。

If the flow spawns subagents, also confirm the parameters you're iterating on - model, effort, prompt - actually reach every subagent and that traces capture their turns; a knob that silently doesn't propagate makes every round on it a no-op.

如果流程会派生子代理，还要确认你正在迭代的参数——模型、effort、提示词——确实传达到了每个子代理，且轨迹捕获了它们的轮次；一个悄悄不生效的旋钮会让围绕它的每一轮都变成空转。

If you have a choice of which eval or slice to climb on, **pick the one with the most signal per token**:

如果你可以选择在哪个评测或切片上爬山，**选择每个 token 信号量最大的那个**：

- **The mechanism must actually drive the score** - disable it and re-run; if the score barely drops, the eval isn't measuring what you're tuning, and no amount of tuning will show up.
  **机制必须真正驱动分数**——将其禁用并重跑；如果分数几乎不降，说明评测度量的不是你在调的东西，再怎么调优也不会体现出来。

- **Low run-to-run variance at small rep counts** - if the baseline's per-rep scores swing widely, a "win" is indistinguishable from variance. Raise reps or pick a calmer slice rather than chasing noise.
  **小重复次数下逐次运行的方差要低**——如果基线的逐次重复分数大幅波动，"获胜"与方差无法区分。提高重复次数或换一个更稳定的切片，而不是追逐噪声。

- **An inspectable mechanism** - prefer an eval where you can *see why* a variant won (an artifact it wrote and reused, a tool-call trace) over a black-box delta you can't attribute and that may not generalize.
  **可检视的机制**——优先选择那种你能*看清变体为何获胜*的评测（它写入并复用的产物、工具调用轨迹），而不是无法归因、未必可泛化的黑箱差值。

---

## Step 1: Agree on the goal, what to change, and how it's wired in / 步骤 1：就目标、修改对象及其接入方式达成一致

**First, the goal.** "Make the number go up" is only one of the things a hillclimb is for, and the loop behaves differently for each - so always ask before anything else, via `AskUserQuestion`, listing the metrics and perf fields the eval actually records:

**首先是目标。**"让数字上升"只是爬山的目标之一，不同目标下循环的行为也不同——所以务必先通过 `AskUserQuestion` 询问，并列出该评测实际记录的指标和性能字段：

> What should this hillclimb optimize?  
> 这次爬山应当优化什么？  
> - **Raise `<headline metric>`** (Recommended when the eval is new and the score has obvious room)  
>   - **提高 `<headline metric>`**（评测较新且分数有明显提升空间时推荐）  
> - **Cut cost per request** - hold `<headline metric>` within noise of baseline  
>   - **降低每请求成本**——使 `<headline metric>` 保持在基线噪声范围内  
> - **Cut latency** - hold `<headline metric>` within noise of baseline  
>   - **降低延迟**——使 `<headline metric>` 保持在基线噪声范围内  
> - **Move to `<other model>`** and recover `<headline metric>` on it
>   - **迁移到 `<other model>`** 并在其上恢复 `<headline metric>`

The answer sets three things for the rest of the loop: the **primary target** the analyzer is pointed at each round (Step 4), the metric the **stopping condition** is phrased against (Step 2), and the **guardrails** - every other recorded metric becomes a must-not-regress-outside-noise constraint rather than something to improve. If the goal is cost or latency, make sure `cost_usd` / `latency_s` is in `_state.json`'s `perf_fields` whether or not it was picked as a display column in build-eval - the goal forces the column. Record the goal in `_state.json` (e.g. `"approve_each_round": false,  
  "goal": {"target": "cost_usd", "direction": "lower", "hold": ["accuracy"]}`) so a resumed session doesn't silently revert to climbing the score.

这个答案为循环余下部分设定三件事：每一轮分析器所指向的**主要目标**（步骤 4）、**停止条件**所针对的指标（步骤 2），以及**护栏**——其余每个被记录的指标都变成"不得在噪声范围外回退"的约束，而不是要改进的对象。如果目标是成本或延迟，确保 `cost_usd` / `latency_s` 在 `_state.json` 的 `perf_fields` 中，无论它在 build-eval 中是否被选为显示列——目标强制要求该列。把目标记录到 `_state.json`（例如 `"approve_each_round": false,  
  "goal": {"target": "cost_usd", "direction": "lower", "hold": ["accuracy"]}`），这样恢复的会话不会悄悄回退成爬分数。

Then ask which part of the app is on the table:

然后询问应用的哪一部分可以改动：

> What do you want me to iterate on? For example:  
> 你想让我迭代什么？例如：  
> - The system prompt  
>   系统提示词  
> - A specific skill or instruction file the agent reads  
>   代理读取的某个特定技能或指令文件  
> - Tool descriptions  
>   工具描述  
> - Model choice or API parameters (`effort`, `thinking`, `max_tokens`)  
>   模型选择或 API 参数（`effort`、`thinking`、`max_tokens`）  
> - The agent loop / harness code itself  
>   代理循环 / harness 代码本身  
> - All of the above - whatever moves the target  
>   以上全部——任何能推动目标的改动  
>  
> And is there anything that's off-limits - parts of the prompt or code I should leave alone even if I think changing them would help?
> 是否有不可触碰的部分——即使我认为改动会有帮助，也不应去动的提示词或代码？

Record the answer. This defines what you'll be editing each round; the off-limits list is a hard constraint. Also have the user name the **harness paths** - the runner script, grader, and any file the eval command executes - and record them in `_state.json.harness_paths` (repo-relative); the runner refuses to start when any of them changed since the user last ran it with `--approve-harness`, so an unreviewed edit to a listed file surfaces as a stopped round rather than a silently different eval (the gate detects changes to the files it digests - the session allowlist for the runner command is the boundary itself). If the user says "whatever moves it," that's fine - but still ask about off-limits, because there's almost always something (a compliance disclaimer, a tone requirement, a tool that's contractually required). For a cost or latency goal, model choice and API parameters (`effort`, `max_tokens`, caching) are usually the biggest levers - make sure they're explicitly in or out.

记录答案。它定义了你每一轮要编辑的内容；禁区清单是硬约束。还要让用户指明 **harness 路径**——运行器脚本、评分器以及评测命令执行的任何文件——并记录到 `_state.json.harness_paths`（仓库相对路径）；当其中任何一个自用户上次以 `--approve-harness` 运行后发生了变化，运行器会拒绝启动，因此对所列文件的未审查改动会表现为一轮被停止，而不是评测被悄悄改变（该门检测的是其摘要所覆盖文件的变更——运行器命令的会话放行清单本身才是边界）。如果用户说"怎么有用怎么来"，没问题——但仍要询问禁区，因为几乎总有某些东西（合规免责声明、语气要求、合同要求必须保留的工具）。对于成本或延迟目标，模型选择和 API 参数（`effort`、`max_tokens`、缓存）通常是最大的杠杆——确保明确它们是在范围内还是范围外。

If the target is a skill or instruction file, confirm **how it reaches the model during the eval**: is it appended directly into the system prompt (which isolates "is the content good?"), or loaded through the app's real skill-discovery path (which also tests "does the model find and use it?")? Both are valid and they measure different things - ask which one the user wants, and make sure the eval runner matches.

如果目标是一个技能或指令文件，确认**它在评测期间如何送达模型**：是直接追加到系统提示词中（这隔离了"内容本身好不好"的问题），还是通过应用真实的技能发现路径加载（这同时测试"模型能否找到并使用它"）？两者都有效且度量对象不同——询问用户想要哪一种，并确保评测运行器与之匹配。

Two things you can choose freely unless the user objects: the model that *reads transcripts and proposes changes* doesn't have to be the model under test - using a stronger model for diagnosis is often worth it - and the proposed changes don't have to be prose. If a concrete helper script, a code snippet, or a worked example would guide the model better than another paragraph of instructions, write that instead.

有两件事你可以自由决定，除非用户反对：*阅读对话记录并提出改动*的模型不必是被测模型——用更强的模型做诊断通常值得——而且提出的改动不必是散文式文本。如果一段具体的辅助脚本、代码片段或完整示例比又一段指令更能引导模型，就写那个。

---

## Step 2: Agree on a stopping condition (and a budget, if cost matters) / 步骤 2：商定停止条件（若成本重要，还有预算）

Ask via `AskUserQuestion` how many rounds to run before checking back in:

通过 `AskUserQuestion` 询问在回来汇报之前运行多少轮：

> One pass over the full set is N cases × R reps on `<model>`, roughly ~Y minutes. How many rounds before I check back in?  
> 对全集跑一遍是 `<model>` 上的 N 用例 × R 重复，大约 ~Y 分钟。在我回来汇报之前运行多少轮？  
> - **Until plateau (Recommended):** keep going until the Step 1 target improves by less than delta for K consecutive rounds (K >= 3 - two flat rounds is too few to call a plateau), or a guardrail metric regresses outside noise, then report.  
>   - **直到平台期（推荐）：**持续运行，直到步骤 1 目标连续 K 轮的提升小于 delta（K >= 3——两轮持平太少，不足以称为平台期），或某个护栏指标在噪声范围外回退，然后汇报。  
> - **One round at a time:** propose, run, report, ask again. Pick this to steer each change.  
>   - **一次一轮：**提议、运行、汇报、再询问。想要逐个把控改动就选这个。  
> - **N rounds:** run N, report, ask whether to continue. (User types N.)
>   - **N 轮：**运行 N 轮，汇报，询问是否继续。（用户输入 N。）

This is both the stopping condition and the check-in cadence - the loop runs autonomously between check-ins.

这既是停止条件也是汇报节奏——两次汇报之间循环自主运行。

Set delta above the noise floor from Step 0.5 - a stopping threshold finer than the eval can resolve never fires honestly.

把 delta 设得高于步骤 0.5 的噪声底——比评测分辨力还细的停止阈值永远不会诚实地触发。

**If reducing cost is itself the Step 1 goal** - not just a ceiling on this loop - read `shared/evals/cost-hillclimb.md` before planning rounds: it narrows this guide's loop to the cost objective, with a lever search order (caching health -> prompt audit -> a model × effort staircase walk -> prompt climb on the frozen model -> a down-left re-probe -> a registered joint confirm), pre-registered adoption gates, and cost-specific stopping rules. (`shared/cost-optimization.md` is the no-eval checklist for the same goal; inside this loop, follow `cost-hillclimb.md`.)

**如果降低成本本身就是步骤 1 的目标**——而不只是本循环的上限——在规划各轮之前先阅读 `shared/evals/cost-hillclimb.md`：它把本指南的循环收窄到成本目标，带有杠杆搜索顺序（缓存健康状况 -> 提示词审计 -> 模型 × effort 阶梯遍历 -> 在冻结模型上做提示词爬山 -> 向左下方重探 -> 预登记的联合确认）、预登记的采纳门，以及成本专属的停止规则。（`shared/cost-optimization.md` 是同一目标的无评测清单；在本循环内请遵循 `cost-hillclimb.md`。）

**If the user asks what the loop will cost or gives you a budget**, also present the **total loop cost** - baseline plus every planned round - and get a ceiling. Estimate it from real numbers, don't guess:

**如果用户询问循环将花费多少或给了你预算**，还要呈现**循环总成本**——基线加上每个计划中的轮次——并确定一个上限。用真实数字估算，不要猜：

1. From a recent results file (or a small sample run if none exists), sum the `usage` fields across all cases, including judge calls if model-graded.
   从近期结果文件（若没有则做一次小样本运行）出发，把所有用例的 `usage` 字段求和，若采用模型评审还要包括评审调用。

2. Multiply by the per-token prices for the user's provider - the Current Models table in `SKILL.md` is first-party; ask or look up if they're on Bedrock/Vertex/etc. Cached reads are ~10× cheaper than base input. This gives the cost of one pass at one rep.
   乘以用户所用服务商的每 token 价格——`SKILL.md` 中的 Current Models 表是第一方价格；如果用户在 Bedrock/Vertex 等平台上，请询问或查询。缓存读取比基础输入便宜约 10 倍。这得到单次重复一遍全集的成本。

3. Measure wall-clock for that pass - time a real run end-to-end and scale; don't estimate.
   实测该遍的墙钟时间——完整计时一次真实运行再缩放；不要估算。

4. Add a rough allowance for the per-round analysis turns - typically small relative to the eval itself.
   为每轮的分析轮次加上粗略余量——相对于评测本身通常很小。

Present `(N + 1) × R × $X` and ask what total spend they're comfortable with. If they hesitate, offer the levers: fewer rounds, fewer reps, a cheaper judge model, or trim to the discriminating cases (rank by cross-rep variance from the baseline run, keep the top K, then run the full set on baseline + winner at the end to confirm). Whatever they pick, the default inside the loop is still to run the **same** set every round; never silently subset it to fit. **The N, R, and case count you present must be what you actually run** - if reps/split later push the total above the approved ceiling, mention it before proceeding.

呈现 `(N + 1) × R × $X` 并询问他们能接受的总花费。如果他们犹豫，提供可调项：更少的轮次、更少的重复、更便宜的评审模型，或裁剪到有区分度的用例（按基线运行中的跨重复方差排序，保留前 K 个，最后在基线 + 获胜者上跑全集以确认）。无论他们选什么，循环内的默认仍是每轮跑**同一**集合；绝不为凑数悄悄取子集。**你呈现的 N、R 和用例数必须是你实际运行的**——如果重复数/切分后来使总额超出已批准的上限，先说明再继续。

When the goal is cost and the candidate change is prompt text, price the instruction itself first - its per-request input tokens × requests per case, against the predicted saving; some candidates disqualify on paper before you spend a round.

当目标是成本且候选改动是提示词文本时，先给这条指令本身定价——它的每请求输入 token 数 × 每用例请求数，对照预测的节省；有些候选在纸面上就被淘汰，不必花一轮。

### Get the plan approved / 获得计划批准

Before you touch any files, confirm the plan with the user and get a clear yes. Three pieces:

在触碰任何文件之前，与用户确认计划并获得明确的同意。三部分：

- **A scope table** - two columns, "will change" and "won't touch," populated from Step 1. The user should be able to glance at it and know exactly which files and knobs are in play.
  **范围表**——两列，"将修改"与"不会触碰"，内容来自步骤 1。用户应能一眼看清哪些文件和旋钮在起作用。

- **Who applies changes** - by default you apply each round's change and the user reviews the result; offer the alternative of **showing each round's diff for a yes/no before it runs** (recommend it when the artifact is customer-facing copy, legal/medical/regulated content, or anything the user said a human must own). Record the choice as `_state.json.approve_each_round`.
  **由谁应用改动**——默认由你应用每轮的改动、用户审查结果；也可提供另一种选择：**在每轮运行前展示该轮 diff 以获得是/否确认**（当产物是面向客户的文案、法律/医疗/受监管内容，或任何用户说过必须由人负责的东西时，推荐这种方式）。将该选择记录为 `_state.json.approve_each_round`。

- **A short approach paragraph** - how you intend to run the loop. For example: *"Each round I'll have a fresh analyzer read the train traces and propose one change; I'll apply it, rerun the full set, and post a status table. I'll spend rounds only on changes big enough to show above the eval's noise floor - a rewritten section or a new rule, not a reworded line. Early on I'll try different levers (tool descriptions, the system prompt, `effort`) to find where the headroom is. Running 8 rounds max."*
  **一段简短的方法说明**——你打算如何运行循环。例如：*"每轮我会让一个全新的分析器阅读训练轨迹并提出一项改动；我来应用它、重跑全集并发布状态表。我只把轮次花在大到能越过评测噪声底的改动上——重写一节或新增一条规则，而不是改写一行措辞。早期我会尝试不同的杠杆（工具描述、系统提示词、`effort`）以找到提升空间所在。最多运行 8 轮。"*

Don't make the user ask for this; produce it by default. It's what lets them trust the loop enough to let it run without checking in every twenty minutes.

不要等用户索要；默认就产出。正是它让用户对这个循环足够信任，从而放心让它运行而不必每二十分钟来确认一次。

---

## Step 3: Set up state, split the data, and take a baseline / 步骤 3：建立状态、切分数据并采集基线

The loop will run for multiple rounds, possibly across multiple sessions. Keep state on disk so it survives interruption and so the user can see the history. If the user already has a results directory and file layout from a prior eval, **keep theirs** - what matters is that each run produces the per-case data from Step 0 (full transcript, `model`, `usage`, grade dict, all from the same model call); the layout below is the default when starting fresh. Otherwise, create a working directory - `.claude/hillclimb/<flow-name>/` is a reasonable default, but put it wherever fits their repo - with this layout. **The report generator reads this tree directly**, so the file names and field names below are an exact contract, not a suggestion:

循环会运行多轮，可能跨越多个会话。把状态保存在磁盘上，使其能经受中断并让用户看到历史。如果用户已有先前评测留下的结果目录和文件布局，**沿用他们的**——关键是每次运行都要产出步骤 0 要求的逐用例数据（完整对话记录、`model`、`usage`、评分字典，全部来自同一次模型调用）；下面的布局是全新开始时的默认方案。否则，创建一个工作目录——`.claude/hillclimb/<flow-name>/` 是合理的默认值，放在适合其仓库的任何位置即可——并采用如下布局。**报告生成器直接读取这棵目录树**，因此下面的文件名和字段名是精确契约，不是建议：

```
.claude/hillclimb/<flow>/
  _state.json            # loop state + report config - shape below
  metrics.md             # free-text rubric (markdown) - what each metric means; the full viewer renders it
  narrative.md           # model-authored running exec summary - rewritten after every round; final version in Step 5
  trajectory/
    scores.tsv           # derived by the report builder (full or lite): per-case mean of the primary metric, one column per round
  baseline/
    results.jsonl        # one JSON object per (case, rep) - shape below
    summary.json         # per-variant header - shape below
    traces/
      <id>_rep<k>.json   # full transcript, one file per rep, every case (the analyzer is handed only the train ones - Step 4)
  v1/
    change.md            # what changed this round and why (from the analyzer)
    change.patch         # the actual diff applied to the codebase
    results.jsonl  summary.json  traces/<id>_rep<k>.json
  v2/
    ...
```

**`_state.json`** - everything here is optional except the split ids; `metrics` and `perf_fields` let you override what the report infers from the data. `best.round` is 0-indexed (0 = baseline). If the eval has multiple metrics, **the first `binary`-kind metric (else `metrics[0]`) is the report's headline** - order the list accordingly. `goal` records the Step 1 answer - the metric or perf field the loop is optimizing, its direction, and the metrics it must hold - so `best` is picked against the goal, not blindly against the headline:

**`_state.json`**——除切分 id 外这里的一切都是可选的；`metrics` 和 `perf_fields` 允许你覆盖报告从数据中推断的内容。`best.round` 从 0 开始计数（0 = 基线）。如果评测有多个指标，**第一个 `binary` 类型的指标（否则为 `metrics[0]`）是报告的头条**——相应地安排列表顺序。`goal` 记录步骤 1 的答案——循环正在优化的指标或性能字段、其方向，以及必须保持的指标——因此 `best` 是针对目标挑选的，而不是盲目对头条：

```js
{ "current_round": 2, "reps": 2,
  "goal": {"target": "pass", "direction": "higher", "hold": ["cost_usd"]},
  "approve_each_round": false,
  "best": {"round": 1, "test_score": 0.70},
  "train_ids": ["case_01", ...], "test_ids": [...],
  "harness_paths": ["eval/run-eval.mjs", "eval/grade.mjs"],
  "harness_sha": "...written by the runner on --approve-harness; never edit by hand...",
  "metrics": [
    {"id": "pass",      "kind": "binary", "label": "Pass"},
    {"id": "quality",   "kind": "judge"},
    {"id": "verbosity", "kind": "float",  "better": "lower"}
  ],
  "perf_fields": [
    {"id": "cost_usd",  "label": "Cost",    "unit": "$"},
    {"id": "latency_s", "label": "Latency", "unit": "s"}
  ],
  "prices": { "my-custom-model": {"in": 2.0, "out": 8.0} } }
```

**`results.jsonl`** - one line per case per rep, **appended as each case completes** so a crash doesn't lose finished work. `grade` can be a bool, a number, or a `{metric_id: number}` dict; `explanation` is an optional `{metric_id: "judge reasoning"}` sibling for judge-kind metrics. Put the full prompt text in `prompt` (the report shows it). `tags` is ordered - `tags[0]` is the primary grouping key. Record `model` from the response, not from config - `cost_usd` is derived from each row's `model` × `usage` (plus `judge_model` × `judge_usage` when present), so a model swap can't carry a stale rate and the runner doesn't compute cost at all. The full viewer does that derivation when it is on disk; otherwise compute `cost_usd` per row yourself when you report, with the recipe in build-eval.md § Before the first paid call (the "If the user asks what this will cost" bullets: Current Models prices in `SKILL.md`; cache writes at 1.25× input, cache reads at 0.1× input) - that is where the status table's `$/run` and `spend` come from. The adapter is forgiving on input: `prompt_id` may also be spelled `id` or `case_id`, and if `_state.json` omits `metrics` they're inferred from the union of grade keys. Perf keys are read by exact name:

**`results.jsonl`**——每个用例每次重复一行，**随每个用例完成即追加**，这样崩溃不会丢失已完成的工作。`grade` 可以是布尔值、数字或 `{metric_id: number}` 字典；`explanation` 是评审类指标可选的 `{metric_id: "judge reasoning"}` 同级字段。把完整提示词文本放进 `prompt`（报告会展示它）。`tags` 有序——`tags[0]` 是主分组键。从响应而不是配置中记录 `model`——`cost_usd` 由每行的 `model` × `usage` 推导（如有评审则加上 `judge_model` × `judge_usage`），这样换模型不会沿用过期价格，运行器本身完全不计算成本。完整查看器在磁盘上时会做该推导；否则你在报告时自己按行计算 `cost_usd`，方法见 build-eval.md § Before the first paid call（"如果用户问这要花多少钱"要点：`SKILL.md` 中的 Current Models 价格；缓存写入按 1.25× 输入价，缓存读取按 0.1× 输入价）——状态表的 `$/run` 和 `spend` 列即来源于此。适配器对输入宽容：`prompt_id` 也可拼作 `id` 或 `case_id`，若 `_state.json` 省略 `metrics`，则从评分键的并集推断。性能键按精确名称读取：

```json
{ "prompt_id": "case_17", "rep": 0, "prompt": "...full prompt text...",
  "tags": ["topic-a", "hard"],
  "grade": {"pass": 1, "quality": 7.2, "verbosity": 3.0},
  "explanation": {"quality": "Cites two sources; balanced."},
  "model": "claude-opus-5-5",
  "latency_s": 12.4,
  "tool_calls": 2, "web_searches": 1,
  "usage": {"input_tokens": 1200, "output_tokens": 480} }
```

Two more optional per-row keys the report understands: `attachments` (a list of `{kind: "image"|"file"|"url", ref, alt?}` shown alongside the prompt) and `meta` (an arbitrary dict surfaced in the transcript header).

报告还能理解另外两个可选的逐行键：`attachments`（`{kind: "image"|"file"|"url", ref, alt?}` 的列表，随提示词一同显示）和 `meta`（在对话记录头部展示的任意字典）。

**`summary.json`** - per-variant header for the report, not aggregate stats (the report recomputes those from `results.jsonl`). All optional; `target` is one of `"system_prompt" | "skill" | "tools" | "code"`:

**`summary.json`**——报告的逐变体头部信息，不是汇总统计（报告会从 `results.jsonl` 重新计算）。全部可选；`target` 是 `"system_prompt" | "skill" | "tools" | "code"` 之一：

```json
{ "description": "Enable web_search tool",
  "target": "system_prompt",
  "suspicious": "val lift not replicated on a second seed" }
```

**`traces/<id>_rep<k>.json`** - the verbatim conversation as a JSON list of `{role, content, thinking?, name?}` turns where `role` is `system | user | assistant | tool_call | tool_result`.

**`traces/<id>_rep<k>.json`**——逐字对话，以 `{role, content, thinking?, name?}` 轮次的 JSON 列表表示，其中 `role` 为 `system | user | assistant | tool_call | tool_result`。

The `traces/` directories are what the analyzer reads between rounds. Keeping them per-round with one file per rep means that by round 3 the analyzer can diff `baseline/traces/case_17_rep0.json` against `v2/traces/case_17_rep0.json` and see exactly what behavior changed. `trajectory/scores.tsv` is the at-a-glance cross-round view: one row per case, one column per round, cell = that case's mean primary-metric score - derived by the report builder (full or lite, identically) from the `results.jsonl` files, so the runner doesn't write it and it can't drift.

`traces/` 目录是分析器在轮次之间阅读的内容。按轮保存、每次重复一个文件，意味着到第 3 轮时分析器可以把 `baseline/traces/case_17_rep0.json` 与 `v2/traces/case_17_rep0.json` 做差分，精确看到行为变了什么。`trajectory/scores.tsv` 是一眼可览的跨轮视图：每个用例一行、每轮一列，单元格 = 该用例的主指标均值——由报告构建器（完整版或精简版，方式相同）从 `results.jsonl` 文件推导，因此运行器不写它，它也不会漂移。

**Make artifacts visible.** If the flow consumes or produces visual or structured artifacts - input images or PDFs, computer-use screenshots, generated HTML/SVG/plots, files the model wrote - make sure a human reviewing a case can actually see them, not just a filename - rendered in the Transcripts tab with the full viewer, opened from the referenced path otherwise. The full viewer has prebuilt slots for this; the runner just fills them:

**让产物可见。**如果流程消费或产生视觉或结构化产物——输入图像或 PDF、计算机使用截图、生成的 HTML/SVG/图表、模型写的文件——确保人工审查用例时能真正看到它们，而不只是一个文件名——用完整查看器时在 Transcripts 标签页中渲染，否则从引用路径打开。完整查看器为此预留了现成槽位；运行器只需填充：

- **Input artifacts** - on the `results.jsonl` row: `"attachments": [{"kind":"pdf","ref":"baseline/inputs/case_3.pdf","alt":"source doc"}]`. Renders above the first user turn.
  **输入产物**——放在 `results.jsonl` 行上：`"attachments": [{"kind":"pdf","ref":"baseline/inputs/case_3.pdf","alt":"source doc"}]`。渲染在第一个用户轮次上方。

- **Output artifacts** - on the trace turn that produced them: `{"role":"assistant","content":"...","attachments":[{"kind":"html","ref":"v2/out/case_3.html"}]}`. Renders below that turn. Write the artifact to disk under the variant dir and `ref` it relative to the flow root.
  **输出产物**——放在产生它的轨迹轮次上：`{"role":"assistant","content":"...","attachments":[{"kind":"html","ref":"v2/out/case_3.html"}]}`。渲染在该轮次下方。把产物写到变体目录下，`ref` 使用相对流程根目录的路径。

- **Rich content in the response text** - fenced ` ```html `, ` ```svg `, ` ```json ` blocks in an assistant turn's `content` get a "> Render" toggle automatically; nothing extra to write.
  **响应文本中的富内容**——助手轮次 `content` 中的围栏 ` ```html `、` ```svg `、` ```json ` 代码块会自动获得 "> Render" 切换开关；无需额外编写。

The full viewer renders `image`/`svg` inline, `html` in a sandboxed scrollable iframe, `pdf` in the browser's native viewer, `json`/`text` in a `<pre>` - each with a Hide/Show toggle. Anything else (`file`: docx, pptx, ...) shows as a download chip. Paths under ~2 MB are inlined into `report.html`; larger ones stay as download links so the report doesn't balloon. **One slot per artifact**: when a turn has `attachments`, the viewer suppresses its inline-render toggle for fenced blocks in that turn's text - so put the output in `attachments` once and the response text stays plain source. The lite report renders none of this - it links each trace file, and the analyzer and the user open the `ref`'d files directly - so keep every `ref` relative to the flow root either way. If a flow needs something the prebuilt viewer doesn't cover, edit `report/frontend/atoms.jsx` (`ArtifactView`) and rebuild via `frontend/scripts/build.sh` (EAP install only - the CLI doesn't extract the frontend).

完整查看器内联渲染 `image`/`svg`，`html` 放入沙箱化的可滚动 iframe，`pdf` 用浏览器原生查看器，`json`/`text` 放入 `<pre>`——每个都带隐藏/显示切换。其余（`file`：docx、pptx 等）显示为下载芯片。约 2 MB 以下的路径内联进 `report.html`；更大的保持为下载链接，以免报告膨胀。**每个产物一个槽位**：当某个轮次带 `attachments` 时，查看器会取消该轮次文本中围栏代码块的内联渲染开关——所以把输出放一次到 `attachments`，响应文本保持纯源码。精简报告不渲染这些——它链接每个轨迹文件，由分析器和用户直接打开被 `ref` 引用的文件——因此无论哪种方式都要让每个 `ref` 相对流程根目录。如果流程需要预置查看器未覆盖的功能，编辑 `report/frontend/atoms.jsx`（`ArtifactView`）并通过 `frontend/scripts/build.sh` 重建（仅限 EAP 安装——CLI 不解包前端）。

**Split the prompt set so the analyzer can't overfit to the number you report.** The analyzer reads transcripts to propose changes - it will, by design, fix the specific cases it sees. The score you report must come from cases it never read. How you carve that depends on how many cases you have; pick the lightest structure that gives a held-out number you can trust:

**切分提示词集合，使分析器无法对你报告的数字过拟合。**分析器阅读对话记录来提出改动——按设计，它会修复它看到的特定用例。你报告的分数必须来自它从未读过的用例。如何切分取决于你有多少用例；选择能给出可信保留数字的最轻量结构：

- **Default - train / test.** *Train* is the set whose transcripts the analyzer reads each round. *Test* is everything else: scored every round alongside train, never opened by the analyzer; its score picks the winning round and is the headline. **Draw the split at random, stratified by `tags[0]` - never by baseline score.** Train needs enough failures to show a pattern (a handful to a couple dozen), but get them by making train big enough, not by hand-picking the worst cases: a train slice selected for low scores means the analyzer only ever sees pathological cases and tunes for the tail, and those cases regress toward the mean on re-run anyway - a healthy train gain with a flat test set is the signature. After the baseline, check that train and test means agree within noise; if they don't, re-draw before round 1. The analyzer can still *focus* on the failures within train.
  **默认——训练 / 测试。***Train* 是分析器每轮阅读其对话记录的集合。*Test* 是其余全部：每轮与训练集一起评分，但从不被分析器打开；其分数选出获胜轮并构成头条。**随机抽取切分，按 `tags[0]` 分层——绝不按基线分数。**训练集需要足够多的失败来呈现模式（几个到两三个），但要通过把训练集做得足够大来获得，而不是手工挑选最差的用例：专挑低分用例的训练切片意味着分析器只会看到病态用例并为尾部调优，而这些用例重跑时本就会向均值回归——训练集健康提升而测试集持平正是其特征。基线之后，检查训练与测试均值是否在噪声范围内一致；如果不一致，在第 1 轮之前重新抽取。分析器仍可以*聚焦*训练集内的失败用例。

- **Small set, or a cross-case metric** (pairwise ordering, ranking - anything that fragments inside a slice). Don't split. Score the whole set every round, lean on **reps** to tighten the noise, and have the analyzer read a few targeted failure transcripts rather than a fixed slice. Label per-round scores in the report as **directional**: an improvement that holds across reps is real signal, but with no held-out set the headline is an iterate-on number, not a publish number.
  **小集合，或跨用例指标**（成对排序、排名——任何在切片内部会碎片化的东西）。不要切分。每轮对整个集合评分，依靠**重复次数**收紧噪声，让分析器阅读若干有针对性的失败对话记录而非固定切片。在报告中把逐轮分数标注为**方向性**：跨重复保持的改进是真实信号，但在没有保留集的情况下，头条是一个用于迭代的数字，不是可发布的数字。

- **Large set** (~150+ cases) where you want a final number untouched by round selection: optionally carve a third *validation* slice - scored each round to pick the winner - and hold *test* back until the end. For most evals the extra bookkeeping isn't worth it.
  **大集合**（约 150+ 用例）且你想要一个不受轮次选择影响的最终数字：可选地再切出第三个*验证*切片——每轮评分用于挑选获胜者——并把*测试*留到最后。对大多数评测来说，额外的簿记不值得。

Be honest with the user about what their split can and can't tell them. For a binary pass-rate the 95% CI half-width is roughly `1/sqrt(n·reps)` - 25 test cases at 2 reps is about ±14 points, 50 at 2 reps about ±10 - so show that number for their set and let them choose; reps and test-size are two knobs on the same dial. Whatever they pick, record the IDs in `_state.json` exactly as they appear in `baseline/results.jsonl`'s `prompt_id` (a runner that sanitizes ids for file paths keys its rows on the sanitized form), fix the split once, and don't change it.

就切分能告诉他们什么、不能告诉什么，对用户保持诚实。对二元通过率，95% 置信区间半宽约为 `1/sqrt(n·reps)`——25 个测试用例 2 次重复约 ±14 分，50 个用例 2 次重复约 ±10——所以把这个数字展示给他们的集合让他们选择；重复数和测试集大小是同一个旋钮上的两个刻度。无论他们选什么，把 ID 按其在 `baseline/results.jsonl` 的 `prompt_id` 中的原样记录到 `_state.json`（会把 id 清洗成文件路径的运行器以清洗后的形式作为行的键），一次性固定切分，并且不再更改。

**Reps per prompt** is the other knob. Model outputs vary run-to-run, and repeating each prompt tightens the estimate - especially worth it on a small set, or whenever the Step 2 budget has room. Offer it and let the user pick the count; if they choose more than one, record results as "k of N reps passed" rather than a single pass/fail. Don't default this silently in either direction. Reps tighten only the noise they re-sample - a runner that reuses a once-built generated artifact across reps leaves its build variance untouched (see the Step 0.5 rebuild check).

**每个提示词的重复次数**是另一个旋钮。模型输出逐次运行各不相同，重复每个提示词能收紧估计——在小集合上或步骤 2 预算有富余时尤其值得。提供该选项并让用户选数量；如果他们选择多于一次，把结果记录为"N 次重复中通过 k 次"而不是单一的通过/失败。不要在任何方向上悄悄设默认值。重复只收紧它们重新采样的噪声——跨重复复用一次性构建产物的运行器不会触及构建方差（见步骤 0.5 的重建检查）。

**Isolate ground truth from the model under test.** If the eval has reference answers, rubrics, or expected outputs, make sure they are **not reachable from the model's context** - not in its system prompt, not in a tool result it can read, not in a file its code-execution or bash tool can open, not in a fixture its container mounts. Keep them in the grader only. This has to be structural; "don't look at the answers" in a prompt is not a defense, and an agent under optimization pressure will eventually find a `cat evals.json` that wins the game without playing it. If the user's existing eval stores prompts and answers in the same file, split it before the first round. And if the eval derives from a public benchmark and the agent has web or network tools, the answers are also reachable *through the internet* - public mirrors of the benchmark often include solutions. Screen every passing transcript for fetches of the benchmark's repo or solution mirrors (and consider blocking egress to them); treat a pass accompanied by a solution-hosting fetch as invalid, and don't assume any automated contamination check will catch it.

**把标准答案与被测模型隔离开。**如果评测有参考答案、评分规则或期望输出，确保它们**无法从模型上下文到达**——不在其系统提示词中，不在其可读取的工具结果中，不在其代码执行或 bash 工具能打开的文件中，不在其容器挂载的夹具中。把它们只保存在评分器里。这必须是结构性的；提示词里写"不要看答案"不是防线，处于优化压力下的代理最终会找到一条 `cat evals.json` 的路径，不玩游戏就赢下游戏。如果用户现有的评测把提示词和答案存在同一个文件里，在第 1 轮之前拆开。如果评测源自公开基准且代理有网页或网络工具，答案也能*通过互联网*到达——基准的公开镜像常常包含解答。筛查每个通过的对话记录中对基准仓库或解答镜像的抓取（并考虑阻断到它们的外联）；把伴随解答镜像抓取的通过视为无效，并且不要指望任何自动化污染检查会抓住它。

【评论】该段强调用结构性隔离而非提示词约束来保护标准答案，并覆盖公开基准经互联网泄漏的途径——针对的是优化压力下代理"绕过任务、直接读取答案"的作弊路径。

**Handling non-text payloads in traces.** Don't embed raw base64 or binary blobs in the trace text - they bloat the file and render as noise. Instead, write each image or binary payload to a sidecar file and put a markdown reference at the point in the turn where it appeared - `![](<path from flow root>)` for an image, a link for anything else - so the report shows a thumbnail and the analyzer can still open the real file. Only fall back to a bare placeholder like `<[binary: 12kB]>` when the payload genuinely isn't worth keeping.

**处理轨迹中的非文本负载。**不要在轨迹文本中嵌入原始 base64 或二进制块——它们会让文件膨胀并渲染成噪声。改为把每个图像或二进制负载写到侧车文件，并在轮次中它出现的位置放一个 markdown 引用——图像用 `![](<path from flow root>)`，其他用链接——这样报告显示缩略图，分析器仍能打开真实文件。只有当负载确实不值得保留时，才退回使用 `<[binary: 12kB]>` 这类裸占位符。

**Baseline.** Run the unmodified app across **the full set** at the chosen rep count, and record it in `baseline/` and in `_state.json`. Record an environment fingerprint with it - repo commit, lockfile hash, local patches or stubs, disabled tools, model id - and re-verify the fingerprint before each round's run: a changed fingerprint means re-baseline, because the comparison is broken either way. If time or an environment change later separates the baseline from a round, score a no-change control alongside the candidate and gate against the control, not the stale baseline number - identical-code drift of a few points between measurement days is common; where a stochastic build sits between the lever and the score, the control must re-run the build, not just the scorer. The control re-run also estimates the same-environment flip rate for free; don't set a keep gate inside that noise floor (Step 2's threshold sizing). (If cost matters and the pinned reps/split push the total above the Step 2 ceiling, mention it before this run.) You can run the report builder (full or lite - Step 4 has the invocation) on the flow directory now if you want a look: with only a single variant on disk (just `baseline/`, or no `_state.json` yet) the full viewer shows only the Evals and Transcripts tabs - the Summary tab auto-hides until there's a second variant to compare.

**基线。**让未修改的应用在选定重复次数下跑**全集**，记录到 `baseline/` 和 `_state.json`。同时记录环境指纹——仓库提交、lockfile 哈希、本地补丁或存根、禁用的工具、模型 id——并在每轮运行前重新核验指纹：指纹变了就意味着重新采集基线，因为无论哪种情况比较都已失效。如果时间或环境变化后来把基线与某一轮隔开，就在候选旁边加评一个无改动对照，并以对照而非过期的基线数字做门控——相同代码在不同测量日之间漂移几分很常见；当随机构建位于杠杆与分数之间时，对照必须重跑构建，而不只是评分器。对照重跑还顺带估计了同环境翻转率；不要在那个噪声底之内设置保留门（步骤 2 的阈值定标）。（如果成本重要且固定的重复数/切分使总额超出步骤 2 上限，在本次运行前先说明。）如果你想先看看，现在就可以在流程目录上运行报告构建器（完整版或精简版——调用方式见步骤 4）：磁盘上只有一个变体时（只有 `baseline/`，或还没有 `_state.json`），完整查看器只显示 Evals 和 Transcripts 两个标签页——Summary 标签页会自动隐藏，直到有第二个变体可比。

---

## Step 4: The loop / 步骤 4：循环

Each round is: **analyze -> apply -> run -> record**. Two rules are non-negotiable throughout: only the train split's transcripts are ever read, and neither the eval set nor the budget changes without going back to the user.

每一轮是：**分析 -> 应用 -> 运行 -> 记录**。两条规则全程不可协商：只读训练切分的对话记录，且不回头找用户就绝不改动评测集合或预算。

**Analyze.** If the next change is already clear - the user named a specific fix, or the last round's result points straight at one - skip the analyzer and write `vN/change.md` directly. Otherwise spawn **one fresh analyzer subagent** (the Claude Code Task tool) and give it: the previous round's **train-split trace files** - the runner writes a trace for every case, so don't hand over the `traces/` directory; list the exact `vN/traces/<id>_rep<k>.json` paths whose `<id>` is in `_state.json.train_ids` (or copy them to a scratch directory) and tell the analyzer not to read anything else under `traces/` - the **train rows of `results.jsonl`** so it sees every metric's per-case score, the **train rows of `trajectory/scores.tsv`** so it sees how each case has moved across every prior round, the current version of the artifact being iterated on, the scope and off-limits list from Step 1, and **which metric this round is targeting** - on round 1 that's the Step 1 goal, along with the guardrail metrics it must hold. The analyzer scans the scores to pick which transcripts to read, reads however it likes - sort by the target metric, diff low vs high scorers, correlate across metrics, spot cases stuck flat across rounds - and returns a proposed change with a short rationale that cites the specific traces motivating it. Save that rationale to `vN/change.md`. A fresh subagent each round keeps the outer session from accumulating transcript content in its own context.

**分析。**如果下一个改动已经明确——用户点名了具体修复，或上一轮的结果直指某个改动——跳过分析器，直接写 `vN/change.md`。否则派生**一个全新的分析器子代理**（Claude Code Task 工具）并给它：上一轮的**训练切分轨迹文件**——运行器为每个用例都写轨迹，所以不要直接交出 `traces/` 目录；列出 `<id>` 在 `_state.json.train_ids` 中的那些确切路径 `vN/traces/<id>_rep<k>.json`（或把它们复制到临时目录），并告诉分析器不要读 `traces/` 下的其他任何内容——`results.jsonl` 的**训练行**，让它看到每个指标的逐用例分数；`trajectory/scores.tsv` 的**训练行**，让它看到每个用例在之前每一轮如何变动；被迭代产物的当前版本；步骤 1 的范围与禁区清单；以及**本轮针对哪个指标**——第 1 轮即步骤 1 的目标，连同其必须保持的护栏指标。分析器扫描分数以挑选要读的对话记录，阅读方式随它——按目标指标排序、对比高低分者、跨指标做相关、找出多轮持平的用例——然后返回一项提议改动及引用了佐证轨迹的简短理由。把该理由保存到 `vN/change.md`。每轮全新的子代理避免外层会话把对话记录内容积累进自己的上下文。

The outer session **does not read transcripts itself** - for choosing changes it sees scores only. That separation is the data-isolation guarantee: nothing from the held-out test set can leak into a proposed change, because the thing proposing changes never sees anything but train.

外层会话**自己不读对话记录**——在选择改动时它只看分数。这种分离就是数据隔离保证：保留测试集的任何内容都不可能泄入提议的改动，因为提出改动的东西除训练集外什么也看不到。

【评论】训练/测试隔离借鉴自机器学习实践，这里把它扩展到提出改动的模型自身的阅读行为：只让分析器读训练轨迹，从流程上杜绝保留集信息回流进改动提议。

Three steers to pass the analyzer. First, **generalize, don't memorize**: the change should describe the failure *behavior*, not the failure *content* - pasting specific nouns or phrases from train cases into the prompt is the fastest route to an overfit change that helps train and does nothing held-out. Second, transcripts show what a block makes the model *do*, not what it prevents or enables without visible action - when proposing to remove something, name which cases you expect to regress, not just which recover. Third, it's fine - especially early on - for the analyzer to propose trying a different lever entirely (a tool description, an API parameter) rather than another wording of the same sentence; exploring where the headroom is can be worth a round.

给分析器三条引导。第一，**泛化，不要背题**：改动应描述失败的*行为*，而非失败的*内容*——把训练用例中的具体名词或短语粘贴进提示词是制造过拟合改动的最快路径，它帮训练集、对保留集毫无作用。第二，对话记录展示的是一段文字让模型*做什么*，而不是它在没有可见动作时阻止或启用了什么——提议删除某内容时，要指出你预期哪些用例会回退，而不只是哪些会恢复。第三，分析器提议彻底换一个杠杆（工具描述、API 参数）而不是同一句话的另一种措辞，这完全没问题——尤其是在早期；探明提升空间在哪里值得一轮。

**Spend a round only on a change the eval can see**, whether the analyzer proposes it or you write it directly. (A fix the user asked for by name still gets its round; say so in that round's status message if you expect it to land inside the noise floor.) A change whose effect is smaller than the Step 0.5 noise floor is kept or reverted largely by chance. So fix the targeted behavior at its root (rewrite the section that causes it, add the missing rule or capability) rather than rewording a line; effect is the measure, not diff length - one missing fact, a new tool or a different `effort` setting can be the whole fix. A fix can gain at most what the cases showing the behavior now lose: if no single behavior loses enough to clear the floor, treat it as a stall instead of inflating the change - run Step 4.5's categorization now, and in that round's status message offer more reps or cases, which is what lowers the floor. A root fix is still one change - one hypothesis about why cases fail, in one patch, kept or reverted whole, however many places it touches - not unrelated fixes bundled together (only Step 4.5's breadth pass does that, for gaps each too small to measure alone).

**只为评测看得见的改动花一轮**，无论它是分析器提出的还是你直接写的。（用户点名的修复同样有资格占一轮；如果你预计它会落在噪声底之内，在该轮状态消息里说明。）效应小于步骤 0.5 噪声底的改动，保留或回退基本靠运气。所以要从根源修复目标行为（重写导致它的那一节，补上缺失的规则或能力），而不是改写一行的措辞；效应才是尺度，不是 diff 长度——一个缺失的事实、一个新工具或不同的 `effort` 设置就可能构成整个修复。一项修复最多能赢得当前展现该行为的用例所损失的总和：如果没有任何单一行为的损失足以越过噪声底，就把它当作停滞处理而不是夸大改动——现在就执行步骤 4.5 的归类，并在该轮状态消息中提供增加重复或用例的选项，那才是降低噪声底的办法。根源修复仍是一项改动——关于用例为何失败的一个假设、一个补丁、整体保留或整体回退，无论它触及多少处——而不是把不相关的修复捆绑在一起（只有步骤 4.5 的广度遍历才这样做，针对每个都小到无法单独度量的缺口）。

A sketch of the analyzer's prompt:

分析器提示词的草稿：

> Here is the current `<artifact>` we are iterating on. Below are the train-split rows from `results.jsonl` (each case's full `grade` dict) and the corresponding transcripts. **This round's target is `<metric>` (`<higher|lower>` is better); `<guardrail metrics>` must not regress.** The off-limits list is: `<...>`. Find the cases doing worst on `<metric>`, plus a couple doing best for contrast; name the single behavior that most often costs it, and propose **one** concrete change to the artifact as a unified diff; it may touch several places, but every hunk must serve that one behavior. Say how many cases show the behavior and how much `<metric>` it costs across them - a fix can gain no more than that. The noise floor on `<metric>` is about `<±X>`: if a full fix would still land inside it, say so instead of proposing a change; otherwise fix the behavior at its root (rewrite the section that causes it, add the missing rule or capability) rather than rewording a line. For every trace you cite as evidence, **quote the relevant lines verbatim** and append its trace file path (`<vN>/traces/<case_id>_rep<k>.json`) - and, if `report.html` was built by the full viewer, the deep link `report.html#tab=transcript&ex=<case_id>&cmp=<vN>&rrep=<k>` - so the user can open that exact rep with one click. Do not reference test cases.

> 这是我们正在迭代的当前 `<artifact>`。下面是 `results.jsonl` 的训练切分行（每个用例的完整 `grade` 字典）及对应的对话记录。**本轮目标是 `<metric>`（以 `<higher|lower>` 为更优方向）；`<guardrail metrics>` 不得回退。**禁区清单为：`<...>`。找出 `<metric>` 上表现最差的用例，外加几个表现最好的作对照；指出最常导致失分的单一行为，并以统一 diff 的形式对产物提出**一项**具体改动；它可以触及多处，但每个 hunk 都必须服务于这一个行为。说明有多少用例展现该行为、它在这些用例上共损失多少 `<metric>`——修复最多只能赢回这么多。`<metric>` 的噪声底约为 `<±X>`：如果完整修复仍会落在噪声底之内，就直接说明而不是提议改动；否则从根源修复该行为（重写导致它的那一节，补上缺失的规则或能力），而不是改写一行措辞。你引用为证据的每条轨迹都要**逐字引用相关行**，并附上其轨迹文件路径（`<vN>/traces/<case_id>_rep<k>.json`）——如果 `report.html` 由完整查看器构建，还要附上深链 `report.html#tab=transcript&ex=<case_id>&cmp=<vN>&rrep=<k>`——让用户一键打开确切的那个重复。不要引用测试用例。

When reading `trajectory/scores.tsv` to pick focus cases, filter to **train rows only** - feeding test-row movement back into the change proposal is a leak of the held-out signal.

阅读 `trajectory/scores.tsv` 挑选聚焦用例时，只筛**训练行**——把测试行的变动反馈进改动提议就是对保留信号的泄漏。

**Apply.** (If `approve_each_round` is set, show the diff and one-line rationale and wait for a yes before running; a no counts as a reverted round with the user's reason recorded in `change.md`.) Before applying, do a **de-fluff pass** on the proposed change: cut anything that's a platitude, a restatement of default behavior, or advice with no operational content ("be careful," "think step by step"). Do this every round - fluff accumulates one reasonable-sounding sentence at a time. Then edit the actual files in the user's codebase and save the diff to `vN/change.patch` so it can be reverted cleanly (for a brand-new artifact with no prior version, diff against `/dev/null`). That patch is the round's record of what changed (and what the full viewer's diff drawer renders - click a variant row in Summary), so cut it against the user's real source paths - not a scratch copy under `.claude/hillclimb/` - so each hunk reads as an edit they can apply directly to their repo; if the loop is iterating on a temporary copy, diff the original file instead. Also snapshot the full post-change artifact - the system prompt, skill file, or whatever you're iterating on - to `vN/` (e.g., `vN/skill.md`) so each round is inspectable on its own without replaying patches. If the patch touches a path in `_state.json.harness_paths`, the next run will stop for `--approve-harness`: show the user the diff and wait for their OK rather than running the round unattended.

**应用。**（若设置了 `approve_each_round`，展示 diff 和一行理由，等待同意后再运行；拒绝则计为一轮回退，用户原因记录在 `change.md`。）应用之前，对提议的改动做一次**去水分处理**：删掉任何属于陈词滥调、重述默认行为或没有可操作内容的建议（"小心一点"、"一步步思考"）。每轮都做——水分会以一次一句貌似合理的话的速度累积。然后编辑用户代码库中的真实文件，并把 diff 保存到 `vN/change.patch` 以便能干净回退（对没有先前版本的全新产物，对 `/dev/null` 做 diff）。该补丁是本轮的变更记录（也是完整查看器 diff 抽屉渲染的内容——在 Summary 中点击某个变体行即可看到），所以要针对用户的真实源码路径生成——而不是 `.claude/hillclimb/` 下的临时副本——让每个 hunk 都像是能直接应用到其仓库的编辑；如果循环迭代的是临时副本，就改为 diff 原文件。同时把改动后的完整产物快照——系统提示词、技能文件或任何你正在迭代的东西——保存到 `vN/`（例如 `vN/skill.md`），让每一轮无需重放补丁即可单独检视。如果补丁触及 `_state.json.harness_paths` 中的路径，下一次运行会因 `--approve-harness` 而停止：向用户展示 diff 并等待其确认，而不是无人值守地跑完这一轮。

**Run.** Run the full eval - every case, at the chosen rep count. Running the entire set every round keeps every score in the history directly comparable. Keep the runner's concurrency maxed out (up to the rate limit) - the loop's cadence is gated on how fast each pass finishes - provided retries back off with jitter (Step 0.5); in a shared-quota environment, maxed-out concurrency with a hot retry loop converts someone else's burst into your zero-scores. Write per-case results to `vN/results.jsonl`, the aggregate to `vN/summary.json`, and every case's transcript to `vN/traces/<id>_rep<k>.json` (the train/test wall is enforced at hand-off - the Analyze step passes only the train files - not by withholding test traces from disk, which the report and Step 5's before/after pairs need). (`trajectory/scores.tsv` is regenerated by the report builder - full or lite - from those files; the runner doesn't write it.)

**运行。**跑完整评测——所有用例，按选定的重复次数。每轮跑整个集合让历史中的每个分数都直接可比。让运行器的并发保持打满（直至速率上限）——循环的节奏取决于每遍完成的速度——前提是重试带抖动退避（步骤 0.5）；在共享配额环境中，打满的并发加上高频重试循环会把别人的突发流量变成你的零分。把逐用例结果写入 `vN/results.jsonl`，汇总写入 `vN/summary.json`，每个用例的对话记录写入 `vN/traces/<id>_rep<k>.json`（训练/测试之墙在交接环节执行——分析步骤只传训练文件——而不是靠扣住测试轨迹不落盘，报告和步骤 5 的前后配对需要它们）。(`trajectory/scores.tsv` 由报告构建器——完整版或精简版——从这些文件重新生成；运行器不写它。)

Three gates before a round's full pass - each costs at most one case, against a pass that costs all of them:

一轮完整跑批之前的三道闸门——每道最多耗费一个用例，与之相对，一次失败的全量跑批会耗费全部用例：

- **Premise-probe config levers.** When the round's change is a config-surface lever (a model parameter, a tool config, `effort` - anything validated server-side at create or call time) rather than prompt content, run one case first and confirm the lever is accepted - and, where the response exposes it, echoed back. A loud validation error is the cheap outcome; the expensive one is a runner that degrades the config error into retries or scored zeros and runs the full set anyway.
  **对配置杠杆先做前提探针。**当本轮改动是配置层面的杠杆（模型参数、工具配置、`effort`——任何在创建或调用时由服务端校验的东西）而非提示词内容时，先跑一个用例确认该杠杆被接受——并且在响应暴露它的场合被回显。响亮的校验错误是便宜的结局；昂贵的是把配置错误降级成重试或零分、却照样跑完整集的运行器。

- **Print the resolved scope.** Before the pass, have the runner print what it actually resolved - case count × reps × model × estimated cost - and compare it to the approved plan. A dry run that resolves a different case count than the plan is a stop, not a warning.
  **打印解析后的范围。**跑批之前，让运行器打印它实际解析到的内容——用例数 × 重复数 × 模型 × 估计成本——并与已批准的计划比对。试运行解析出的用例数与计划不同就是停止信号，不是警告。

- **Canary before an unattended or expensive pass** - run one case and compare its error and latency profile to baseline's; if degraded, hold rather than burn the round.
  **无人值守或昂贵的跑批之前先放金丝雀**——跑一个用例，将其错误和延迟特征与基线比较；若已劣化，宁可暂停也不要烧掉这一轮。

Infra health per round - retries, timeouts, served-model mismatches - is in `errors.jsonl`; a round whose error profile differs grossly from baseline's is void-and-rerun, not a comparable data point.

每轮的基础设施健康状况——重试、超时、实际服务模型不符——记录在 `errors.jsonl` 中；错误特征与基线截然不同的一轮应作废重跑，而不是当作可比数据点。

**Record and report.** Update `_state.json` (round number, and `best` if this round's test score beats it). A number you'll report - including privately-authored held-out cases and their raw results - must land in the recorded `vN/` layout, never only in a run log. If the session or machine is ephemeral (a CI runner, a remote session), also copy each round's `vN/` to storage that survives it. Don't report a round's score until every case has landed - partial reads can show a sign that flips when the batch finishes; if you must report mid-run, flag it as `N/total`. After every round, write the status table (row layout below) at the top of `narrative.md` and keep it current - it is the per-round record, on disk even when the run is headless - then report in chat a one-line headline ("v3 test 0.71 -> 0.74, change: `<one line>`"). With the full viewer, follow the headline with a pointer to `report.html#tab=summary` instead of the table - the Summary tab is exactly this table, sortable and with the diff drawer one click away. With the lite report, which shows only the primary metric, paste the markdown table under the headline, since it is what carries the guardrail, perf, `$/run` and `spend` columns. `$/run` and `spend` come from `cost_usd` (Step 3: derived by the full viewer when it is on disk, otherwise computed by you from each row's `model` × `usage`). If tracking spend against a budget, cumulative spend is `sum(cost_usd)` over every `vN/results.jsonl`, plus the billed `usage` recorded on failed attempts in each variant's `errors.jsonl` sidecar (failed spend is still spend) - never a maintained counter. The row layout:

**记录与报告。**更新 `_state.json`（轮次编号，以及若本轮测试分数更优则更新 `best`）。你要报告的任何数字——包括自行编写的保留用例及其原始结果——必须落在被记录的 `vN/` 布局中，绝不能只存在于运行日志里。如果会话或机器是临时的（CI 运行器、远程会话），还要把每轮的 `vN/` 复制到能在其销毁后存活的存储。在每个用例都落盘之前不要报告该轮分数——部分读取可能显示出一个在批次完成时翻转的信号；若必须中途报告，标注为 `N/total`。每轮之后，把状态表（行布局见下）写在 `narrative.md` 顶部并保持最新——它是逐轮记录，即使在无头运行时也在磁盘上——然后在聊天里报告一行标题（"v3 test 0.71 -> 0.74, change: `<one line>`"）。使用完整查看器时，标题后给一个指向 `report.html#tab=summary` 的指引而不是表格本身——Summary 标签页就是这个表，可排序，且 diff 抽屉一键可达。使用只显示主指标的精简报告时，把 markdown 表格贴在标题下方，因为护栏、性能、`$/run` 和 `spend` 列全靠它承载。`$/run` 和 `spend` 来自 `cost_usd`（步骤 3：完整查看器在磁盘上时由其推导，否则由你按每行的 `model` × `usage` 计算）。如果对照预算跟踪花费，累计花费是所有 `vN/results.jsonl` 的 `sum(cost_usd)`，加上各变体 `errors.jsonl` 侧车中失败尝试记录的计费 `usage`（失败的花费也是花费）——绝不是维护一个计数器。行布局：

> | round | change (one line)    | test  | train | s/turn | out toks      | tool calls   | $/run          | spend  |  
> |-------|----------------------|-------|-------|--------|---------------|--------------|----------------|--------|  
> | 0     | baseline             | 0.62  | 0.60  | 19.8   | 480           | 3.1          | $2.60          |  2.60  |  
> | 1     | when-to-search rule  | 0.70  | 0.73  | 19.5   | 492 (1.0×)    | 4.2 (1.4×) (up) | $2.97 (1.1×)   |  9.70  |  
> | 2     | effort=medium        | 0.68  | 0.71  | 11.2   | 310 (0.6×) (down)  | 3.0 (1.0×)   | $1.40 (0.5×) (down) | 12.90  |  
>  
> Best so far: round 1 (test 0.70). Next: combine round-1 rule with `effort=medium` and re-check latency.

> | 轮次 | 改动（一行） | 测试 | 训练 | 秒/轮次 | 输出 token | 工具调用 | $/运行 | 累计花费 |  
> |------|--------------|------|------|---------|------------|----------|--------|----------|  
> | 0 | 基线 | 0.62 | 0.60 | 19.8 | 480 | 3.1 | $2.60 | 2.60 |  
> | 1 | 何时搜索规则 | 0.70 | 0.73 | 19.5 | 492 (1.0×) | 4.2 (1.4×)（升） | $2.97 (1.1×) | 9.70 |  
> | 2 | effort=medium | 0.68 | 0.71 | 11.2 | 310 (0.6×)（降） | 3.0 (1.0×) | $1.40 (0.5×)（降） | 12.90 |  
>  
> 目前最佳：第 1 轮（测试 0.70）。下一步：把第 1 轮的规则与 `effort=medium` 结合，并复查延迟。

After each round is scored, also **rewrite `narrative.md`** - a model-authored running exec summary of every harness change and its effect so far, not just this round's. Read all of the `vN/change.md` rationales and `vN/summary.json` results and write one short paragraph that says which variant is currently winning and *why*, in terms of the changes ("v4 has the best recall, but v3's prompt tightening traded a little recall for precision and nets the higher overall score"). Overwrite it wholesale each round - it's a snapshot of the story so far, not an append-only log. Keep the current status table above the paragraph. The full viewer's NARRATIVE panel renders this file, and without the full viewer `narrative.md` is the file to open - either way, keeping it current means the user (or a resumed session) can read the state of play at any point in one screen.

每轮评分之后，还要**重写 `narrative.md`**——一份由模型撰写的、关于至今每次 harness 改动及其效应的滚动执行摘要，而不只是本轮的。阅读所有 `vN/change.md` 的理由和 `vN/summary.json` 的结果，写出一小段话说明当前哪个变体领先以及*为什么*，用改动的语言来表述（"v4 召回最好，但 v3 收紧提示词用少量召回换取了精确率，总分反而更高"）。每轮整体覆写——它是故事至今的快照，不是只增不改的日志。把当前状态表放在这段话上方。完整查看器的 NARRATIVE 面板渲染该文件；没有完整查看器时，`narrative.md` 就是要打开的文件——无论哪种方式，保持其最新意味着用户（或恢复的会话）可以在一屏之内读到当前局势。

Alongside the status table, regenerate the **HTML report** so the user can drill in visually: run the report builder on `.claude/hillclimb/<flow>/` - `shared/evals/report/build-report.mjs` when it is on disk (EAP install), else `shared/evals/report/build-report-lite.mjs` (both paths relative to this skill's base directory), with `node` or `bun` (the selection line and the no-runtime fallback are in build-eval.md §Report builder). The full viewer writes a self-contained `report.html` into the flow directory (exec-summary table and trend charts, side-by-side transcript comparison, and the per-round diffs); the lite builder writes a summary table plus per-case rows with the primary metric per round and links to each trace file. Both write `trajectory/scores.tsv`. It reads the same `summary.json` / `results.jsonl` / `traces/` / `change.*` files you just wrote, so there is nothing extra to produce - run it **after the round's runner has exited**, not while it's still appending (the partial variant's row would show scores from however many cases have landed so far - the full viewer badges it as a partial `N/M cases` run, the lite report just shows the lower case count - don't publish either). **The report is there for when the user wants it, not news to deliver:** give its path once after the baseline and again in the final summary, end each round's status message with the path as a bare last line, and otherwise don't bring it up - no remarks on its size, its notes, or that they should open it - unless the build exits non-zero or they ask. If they ask for more than it shows (with the lite report that could be a chart, the diff on the page, or a dashboard), build that as an extra page beside `report.html`, never in place of it, per `shared/evals/report/SCHEMA.md` §Pages beyond `report.html`; don't offer one unprompted. **Re-apply the Step 0.5 spot-checks to the new round's row - in the status table and, with the full viewer, the Summary tab - before pointing the user at it**: every metric and perf column present, plausible, and consistent with this round's change - a `$0.00` cost, `0.0s` latency (or the `usage` / `latency_s` fields behind them missing from the rows), or a column that didn't move the way the change predicts is a runner bug, not a result. Also spot-check that one transcript renders as distinct turn cards rather than a single text blob (full viewer) or that a linked trace file is a JSON list of `{role, content}` turns (lite). The rest are full-viewer features: `build-report.mjs <flow> --check` flags the common trace-format mistakes without rendering; `build-report.mjs --index .claude/hillclimb/` writes an `index.html` linking every child flow's report when several flows run in parallel; and a run directory of a different shape takes a small adapter per `shared/evals/report/SCHEMA.md` - typically a few dozen lines: assemble `Turn[]` from your raw content blocks, compute `RepResult.perf` from `usage`, declare your metrics - handed to `render()` directly.

在状态表之外，重新生成 **HTML 报告**以便用户可视化下钻：在 `.claude/hillclimb/<flow>/` 上运行报告构建器——磁盘上有 `shared/evals/report/build-report.mjs` 时用它（EAP 安装），否则用 `shared/evals/report/build-report-lite.mjs`（两个路径均相对本技能的基础目录），用 `node` 或 `bun` 运行（选择逻辑与无运行时回退见 build-eval.md §Report builder）。完整查看器把自包含的 `report.html` 写入流程目录（执行摘要表与趋势图、并排对话记录对比、逐轮 diff）；精简构建器写一份汇总表加逐用例行（含每轮主指标）以及指向每个轨迹文件的链接。两者都写 `trajectory/scores.tsv`。它读取的正是你刚写的那些 `summary.json` / `results.jsonl` / `traces/` / `change.*` 文件，因此没有额外要产出的东西——**在该轮运行器退出之后**再运行它，而不是在它仍在追加时（部分变体的行会只显示已落盘用例的分数——完整查看器会把它标记为部分的 `N/M cases` 运行，精简报告只会显示较低的用例数——两者都不要发布）。**报告是为用户想看时准备的，不是要推送的新闻：**在基线之后和最终总结中各给一次路径，每轮状态消息以路径作为孤零零的最后一行收尾，其余时候不要提它——不评论它的大小、它的备注，也不要催用户打开——除非构建以非零退出或用户询问。如果用户想要它没展示的东西（精简报告下可能是图表、页面上的 diff 或仪表盘），把它构建为 `report.html` 旁边的额外页面，绝不要替代它，依 `shared/evals/report/SCHEMA.md` §Pages beyond `report.html`；不要主动提出。**在把新一轮的行指给用户之前，把步骤 0.5 的抽查重新应用到该行——状态表中，以及完整查看器的 Summary 标签页中**：每个指标和性能列都在、合理、且与本轮改动一致——`$0.00` 成本、`0.0s` 延迟（或其背后的 `usage` / `latency_s` 字段在行中缺失），或某列没有按改动预测的方向移动，都是运行器缺陷，不是结果。再抽查一条对话记录能渲染成独立的轮次卡片而非一整块文本（完整查看器），或链接的轨迹文件是 `{role, content}` 轮次的 JSON 列表（精简版）。其余是完整查看器的功能：`build-report.mjs <flow> --check` 无需渲染即可标记常见的轨迹格式错误；`build-report.mjs --index .claude/hillclimb/` 在多个流程并行时写出链接每个子流程报告的 `index.html`；形状不同的运行目录需要一个按 `shared/evals/report/SCHEMA.md` 编写的小适配器——通常几十行：从你的原始内容块组装 `Turn[]`，从 `usage` 计算 `RepResult.perf`，声明你的指标——直接交给 `render()`。

**Every metric that informs your recommendation must be on the record - on the rows, in the status table, in the report - before you make the call.** If, mid-loop, you compute a new metric in a scratch script and it changes which variant you'd pick, stop and fold it in: add it to the runner's grader so every row carries it, declare it in `_state.json` under `metrics`, re-grade the existing variants in place so the comparison is apples-to-apples, and rebuild the report and the status table - *then* recommend. The user must be able to verify every number behind your recommendation from the flow directory alone (`report.html`, `narrative.md`, the `vN/` files), without your chat history. For metrics that are inherently aggregates - inter-rep consistency, or anything else computed across cases rather than per-row - the same rule holds: put a per-variant comparison table in `metrics.md` so the record carries the comparison (the full viewer renders it), not just your prose description of it.

**每个影响你建议的指标都必须先进入记录——在行里、在状态表里、在报告里——然后你才能下结论。**如果在循环中途你在临时脚本里算出一个新指标，而它会改变你会选的变体，停下来把它并入：把它加进运行器的评分器让每行都携带它，在 `_state.json` 的 `metrics` 下声明它，就地重新评分现有变体使比较同口径，并重建报告和状态表——*然后*再提建议。用户必须能仅凭流程目录（`report.html`、`narrative.md`、各 `vN/` 文件）核实你建议背后的每个数字，而不需要你的聊天历史。对本质上就是聚合值的指标——重复间一致性，或任何跨用例而非逐行计算的量——同样的规则成立：在 `metrics.md` 中放一个逐变体对比表，让记录承载对比（完整查看器会渲染它），而不只是你对它的文字描述。

If the grader is a model-as-judge and the score jumps by more than the change could plausibly explain, **treat it as suspicious before treating it as good news**: spot-check a handful of outputs by hand, show the user, and confirm the judge isn't rewarding a surface pattern the change happened to introduce. Record the concern as a one-line `suspicious` string in that round's `summary.json` so the concern stays attached to the variant (the full viewer shows it as a warning badge). An LLM judge being gamed looks exactly like a breakthrough until you check.

如果评分器是模型评审，而分数的跳升超过了改动所能合理解释的幅度，**先当作可疑对待，再当作好消息**：手工抽查若干输出，展示给用户，确认评审没有在奖励改动恰好引入的某种表面模式。把这个疑虑作为一行 `suspicious` 字符串记录进该轮的 `summary.json`，让疑虑附着在该变体上（完整查看器会显示为警告徽章）。被钻空子的 LLM 评审在你核查之前看起来完全像一次突破。

**Separate "did the mechanism engage" from "did it help."** Track a leading indicator of the mechanism firing - how often the agent wrote to memory, called the tool, produced the artifact - as its own column, distinct from the score. It's the in-loop counterpart to the Step 0.5 wiring probe: the probe proved the mechanism *can* work; this proves it *did* this round. A score that moved while the engagement rate didn't is probably noise or a harness artifact - find out which before stacking another change on top.

**把"机制是否触发"与"它是否有帮助"分开。**把机制触发的领先指标——代理写入记忆、调用工具、产出产物的频率——作为独立于分数的一列来跟踪。它是步骤 0.5 接线探针在循环内的对应物：探针证明了机制*能*工作；这个证明它本轮*确实*工作了。分数动了而触发率没动，多半是噪声或框架伪影——在往上叠下一个改动之前先弄清是哪个。

If train went up and test didn't, the change overfit to the cases the analyzer read - revert it and try a different angle next round. If a round regresses on train too, revert before the next round rather than stacking changes on top of it. If train went *down* but test went *up*, treat it as noise at low rep counts - keep the change only if the pattern repeats on a second run. On ties, prefer the later round.

如果训练升了而测试没升，改动对分析器读过的用例过拟合——回退它，下一轮换个角度。如果某轮连训练也回退，在下一轮之前回退它，而不是在上面继续叠改动。如果训练*下降*而测试*上升*，在小重复次数下当作噪声处理——只有该模式在第二次运行中重复出现才保留改动。平局时，偏向更晚的一轮。

**Decide.** Check the stopping condition from Step 2. Treat the budget as a **guide, not a wall**: as you approach it with the score still climbing or an obvious idea untried, say so and offer to extend rather than stopping cold. Conversely, if several consecutive rounds haven't moved the score *outside noise* - point estimates drifting but intervals overlapping - you're likely at a plateau even though the numbers look like they're climbing: run Step 4.5's categorization before another content round. To tighten a variant's interval, append more reps to its `results.jsonl` (and baseline's, for a fair comparison) and rebuild - the report recomputes from whatever rows are there; no new round directory needed. Otherwise, once the Step 1 goal has plateaued, offer to change the target before stopping: pick the guardrail metric or perf field with the most headroom that hasn't been tried, and run another round with the analyzer pointed at it under the constraint of not regressing what's already won - but ask first, since the user set the goal and may consider it done. Only stop when no metric has obvious room, or report best-so-far and ask. If the user asked for check-ins and you've hit the interval, report and wait. Otherwise, loop.

**决策。**检查步骤 2 的停止条件。把预算当作**指南，而不是墙**：当你逼近预算而分数仍在爬升或还有明显的想法没试，说明这一点并提出延长，而不是戛然而止。反过来，如果连续几轮都没把分数移出*噪声范围*——点估计在漂移但区间重叠——即使数字看起来还在爬，你多半已处于平台期：再开下一轮内容改动之前先执行步骤 4.5 的归类。要收紧某个变体的区间，向其 `results.jsonl` 追加更多重复（基线同样追加，以保证公平比较）并重建——报告会从现有行重新计算；不需要新的轮次目录。否则，一旦步骤 1 的目标进入平台期，在停止之前提议更换目标：挑选提升空间最大且尚未尝试的护栏指标或性能字段，让分析器指向它再跑一轮，约束是不回退已经赢得的东西——但先询问，因为目标是用户定的，他们可能认为已经完成。只有当没有任何指标还有明显空间时才停止，或者报告当前最佳并询问。如果用户要求定期汇报且已到间隔，报告并等待。否则，继续循环。

---

## Step 4.5: When the loop stalls, categorize before grinding / 步骤 4.5：循环停滞时，先归类再硬磨

The analyze -> apply -> run loop assumes each failure is caused by the artifact you're tuning. Once the easy content gaps are filled, that stops being true - remaining failures increasingly come from the grader, the harness, the artifact's structure, or plain variance, and another content round can't move them. The tell is **two or three consecutive rounds where the test score hasn't cleared the noise band** despite changes that should have helped. When that happens, stop iterating content and spend one round categorizing instead.

分析 -> 应用 -> 运行的循环假设每个失败都由你正在调优的产物导致。一旦容易补的内容缺口补完，这就不再成立——剩余失败越来越多来自评分器、harness、产物的结构或纯粹的方差，再来一轮内容改动也无济于事。征兆是**连续两三轮测试分数都没能越过噪声带**，尽管改动本应有所帮助。发生这种情况时，停止迭代内容，改用一轮来归类。

Spawn a fresh analyzer subagent (same isolation as Step 4's Analyze) to bucket every remaining train-split failure by root cause, reading each transcript far enough to tell which:

派生一个全新的分析器子代理（与步骤 4 的分析同样的隔离），按根因给每个剩余的训练切分失败归类，把每条对话记录读到足以分辨的程度：

| Bucket | Tell | What to do instead of another content round |
|---|---|---|
| **Artifact gap** | Model never had the fact it needed; transcript shows it guessing or searching | This is the loop's home turf - keep going |
| **Grader disagreement** | Model's output looks correct to you but the grader marks it wrong; or the prompt and the rubric ask for different things | Fix the grader, then re-grade *every* variant in place from stored outputs. Before overwriting, compare old vs new grades - how many cases moved, and did the variant ranking change? If the previous best is still the best and its lead over baseline held, keep going. If the ranking flipped or the lead collapsed to noise, the prior rounds were tuned to the wrong signal: show the before/after table and propose restarting the loop from baseline. |
| **Harness / infra** | Case errored before the model produced a scorable output - auth failure, timeout, rate-limit, env setup. Some harnesses *score* the failure instead of erroring it: zero-scored cases whose transcripts carry infra markers (retries exhausted, stall ceilings, empty outputs) belong here too | Fix the harness; exclude errored cases from the denominator until then. For scored-in zeros, decide the handling rule before comparing scores |
| **Structural** | The content exists in the artifact but the model didn't reach it; or the same review finding recurs across rounds; or one dimension (a language, a provider) underperforms regardless of which feature you target | Reorganize - consolidate duplicated facts into one table, split a monolith file, fix the routing - rather than adding more of the unreached content |
| **Variance** | Pass<->fail flips between identical-code runs are as large as the round-over-round delta | You're at the noise floor on this lever. Report best-so-far; offer to raise reps or change target |

| 归类 | 征兆 | 不再跑内容轮时的替代做法 |
|---|---|---|
| **产物缺口** | 模型从未拥有它需要的事实；对话记录显示它在猜测或搜索 | 这是循环的主场——继续即可 |
| **评分器分歧** | 模型输出在你看来正确但评分器判错；或提示词与评分规则要求的对象不同 | 修复评分器，然后基于已存输出就地重新评分*每个*变体。覆写之前，比较新旧评分——多少用例移动了，变体排名是否改变？如果此前的最佳仍是最佳、且对基线的领先保持，就继续。如果排名翻转或领先缩水成噪声，说明之前的轮次是对着错误信号调优的：展示前后对比表，提议从基线重启循环。 |
| **Harness / 基础设施** | 用例在模型产出可评分输出之前就出错——鉴权失败、超时、限流、环境搭建。有些框架把失败*计分*而不是记为错误：对话记录带有基础设施标记（重试耗尽、停滞上限、空输出）的零分用例也归此类 | 修复 harness；在此之前把出错用例从分母中剔除。对被计分为零的情况，在比较分数之前先定好处理规则 |
| **结构性** | 内容在产物里但模型没有到达；或同一条审查发现跨轮反复出现；或某一维度（某种语言、某个提供商）无论针对哪个特性都表现不佳 | 重新组织——把重复的事实合并进一张表、拆分单一巨型文件、修正路由——而不是添加更多没被读到的内容 |
| **方差** | 相同代码的运行之间通过<->失败翻转的幅度与逐轮差值一样大 | 你已处于该杠杆的噪声底。报告当前最佳；提议提高重复次数或更换目标 |

A failure that fits none of these is itself a signal: the artifact you're tuning may not be the bottleneck for that slice - offer to change target rather than forcing it into a bucket.

不属于以上任何一类的失败本身就是信号：你正在调优的产物可能不是那个切片的瓶颈——提议更换目标，而不是硬塞进某个桶。

Write the bucket counts to `vN/change.md` in place of a content diff for that round, and tell the user: N of the remaining M failures aren't artifact gaps - here's what each cluster needs. Then dispatch per bucket rather than running another content round against all of them.

把各桶计数写进 `vN/change.md` 以替代该轮的内容 diff，并告诉用户：剩余 M 个失败中有 N 个不是产物缺口——每个簇各需要什么。然后按桶分派，而不是对着全部再跑一轮内容改动。

Two patterns this surfaces that the per-round analyzer can't:

这一做法能浮现两种逐轮分析器看不出的模式：

- **The long tail.** The analyzer's "single behavior that most often costs the grade" is worst-bucket-first and never reaches a tail of many small buckets each costing one or two cases. If categorization shows a dozen dimensions each contributing <=2 failures and none of them have artifact coverage, a one-shot **breadth pass** - draft minimal coverage for every uncovered dimension in parallel, apply all at once - covers more ground in one round than the serial loop will in ten. This deliberately breaks one-change-per-round: the dimensions are independent, the question is coverage not attribution, and no single one would move the score enough to measure on its own.
  **长尾。**分析器的"最常导致失分的单一行为"按最差桶优先，永远够不到由许多小桶组成、每个只损失一两个用例的尾部。如果归类显示十几个维度各贡献 <=2 个失败且都没有产物覆盖，一次性的**广度遍历**——并行地为每个未覆盖维度起草最小覆盖，一次性全部应用——一轮覆盖的面比串行循环十轮还多。这刻意打破了一轮一改动的原则：这些维度相互独立，问题是覆盖而非归因，且任何单独一个都不足以让分数移动到可度量的程度。

- **Mid-run grader drift.** Step 0.5 proved the eval was trustworthy at the start. A rubric that's subtly wrong for one feature, or a canonical answer that's gone stale, won't show up as an implausible jump - it shows up as a feature that won't move no matter what content you add. When one bucket resists three rounds of content that looks correct to you, re-read its rubric before writing round four - and if you do change it, re-grade everything, quantify the shift, and decide with the user whether the existing rounds still stand.
  **中途评分器漂移。**步骤 0.5 证明了评测在起点是可信的。对某个特性微妙错误的评分规则，或已过时的标准答案，不会表现为一次不可信的跳升——它表现为无论你添加什么内容都不动的某个特性。当同一个桶顶住三轮在你看来正确的内容改动时，在写第四轮之前重读它的评分规则——如果你确实改了它，重新评分一切，量化这次位移，并与用户一起决定既有轮次是否仍然成立。

---

## Step 5: Report and hand back / 步骤 5：报告并交还

When the loop ends, put the codebase at the version that won on test. The headline is the **test-score delta, baseline vs winner** - you already have both numbers from the per-round runs. (If Step 3 chose no split, report the whole-set delta and label it **directional**; if it chose the optional three-way split, run the held-back test slice now on baseline and winner only.) Rewrite `narrative.md` one last time as the final status table (kept on top, as in Step 4) followed by the four-part exec summary - **Recommended change**, **Versus baseline**, **Why trust this**, **What else was tried** - then regenerate `report.html` one last time (full or lite builder, as in Step 4); it is the artifact you point the user at for per-case detail. In parallel, produce a short text report (in the user's PR description if they want a PR, or as a markdown file otherwise) - this is a **companion** to `report.html`, not a replacement, so point at it for transcripts (full viewer) or trace files (lite) and per-case detail rather than duplicating them inline. It covers:

循环结束时，把代码库置于在测试上获胜的版本。头条是**测试分数增量，基线对获胜者**——这两个数字你从逐轮运行中已经有了。（如果步骤 3 选择不切分，报告全集增量并标注为**方向性**；如果选择了可选的三分切分，现在只在基线和获胜者上跑留存的测试切片。）最后一次重写 `narrative.md`：以最终状态表开头（保持置顶，同步骤 4），后接四部分执行摘要——**推荐改动**、**对比基线**、**为何可信**、**还尝试了什么**——然后最后一次重新生成 `report.html`（完整版或精简版构建器，同步骤 4）；它是你指给用户看逐用例细节的产物。同时，产出一份简短的文字报告（用户想要 PR 就放在 PR 描述里，否则作为 markdown 文件）——它是 `report.html` 的**伴生**，不是替代，因此对话记录（完整查看器）或轨迹文件（精简版）及逐用例细节应指向它，而不是在行内重复。它涵盖：

- **Headline:** test score at baseline -> test score at the winning round, each with a confidence interval, and the delta. This is the result. Train improvement is supporting detail, not the claim. **If the test delta is within noise of zero - the CIs overlap, or a paired test over cases isn't significant - say so plainly and recommend not merging.** An honest "this didn't move the needle, here's what I'd try with more budget" is more useful to the user than a dressed-up marginal gain.
  **头条：**基线测试分数 -> 获胜轮测试分数，各带置信区间，以及增量。这才是结果。训练提升是支撑细节，不是主张。**如果测试增量在零的噪声范围内——置信区间重叠，或跨用例的配对检验不显著——直说，并建议不要合并。**一句诚实的"这没有推动数字，如果预算更多我会尝试这些"比一份包装起来的边际收益对用户更有用。

- **Per-round table:** round, one-line description of the change, train score, test score, and the guardrail columns with their baseline ratios. Flag only the deltas that clear noise; suppress or grey out the ones that don't, so the user's eye lands on what actually moved. The train vs test columns side by side are the generalization-gap trajectory - if they diverge round over round, say so explicitly.
  **逐轮表：**轮次、改动的一行描述、训练分数、测试分数，以及带基线比值的护栏列。只标记越过噪声的增量；压暗或置灰没越过的，让用户的目光落在真正移动的东西上。训练与测试两列并排就是泛化差距的轨迹——如果它们逐轮发散，明确指出。

- The changes that are actually applied to the codebase right now, each with its one-sentence "why" from `change.md`, and each tagged **`[REQUIRED]`** (fixes something broken - e.g., a parameter that errors on the target model) or **`[TUNE]`** (a judgment call that improved the score but that the user could reasonably decline). This lets the user accept the diff selectively.
  此刻实际已应用到代码库的改动，每条附其来自 `change.md` 的一句话"理由"，并分别标注 **`[REQUIRED]`**（修复损坏的东西——例如在目标模型上报错的参数）或 **`[TUNE]`**（提升了分数的判断性选择，但用户可以合理拒绝）。这让用户可以按需选择性接受 diff。

- **A failure taxonomy, when zeros have mixed causes:** how many failures were refusals, harness or serving errors, and timeouts, versus genuine capability misses - a single rate hides it. And when the loop compared models, classify failures per model before quoting a gap: a failure mode only one model triggers (say, a tool-calling convention the harness rejects from that model) is a harness bug confounding the comparison, not a capability difference; report the gap with and without those attempts.
  **失败分类学，当零分成因混合时：**多少失败是拒答、框架或服务错误、超时，多少是真实的能力缺口——单一比率会掩盖这一点。当循环比较过多个模型时，在引用差距之前先按模型分类失败：只有某一个模型触发的失败模式（比如该模型发起的某种工具调用约定被 harness 拒绝）是混淆比较的 harness 缺陷，不是能力差异；引用差距时要分别给出包含与剔除这些尝试的版本。

- **A second-model check, if the artifact will serve more than one model:** re-run the winner once on the other model(s) before recommending it, and report per-model numbers - failures are model-dependent, and a win measured on one model doesn't transfer by default.
  **第二模型检查，如果产物将服务多个模型：**在推荐获胜者之前先在另一个（或另几个）模型上重跑一次，并按模型报告数字——失败依赖模型，在一个模型上测得的获胜默认不可迁移。

- **Two or three before/after transcript pairs** - the same prompt under baseline and under the winning version, side by side - so the user can see the quality change with their own eyes rather than taking the number on faith. Pick cases that illustrate the behavior the changes were targeting.
  **两到三组前后对话记录配对**——同一提示词在基线与获胜版本下、并排呈现——让用户亲眼看到质量变化，而不是只信数字。挑选能说明改动所针对行为的用例。

- **What you'd try next.** Proactively list the concrete levers still on the table - "`effort=medium` looked promising on latency but I didn't re-tune the prompt for it; the `summarize` tool description is still vague; judge could move to `claude-sonnet-5-5`" - rather than waiting for the user to ask whether there's more. Include anything that seemed to need a bigger change than the target allowed.
  **你接下来会尝试什么。**主动列出仍在台面上的具体杠杆——"`effort=medium` 在延迟上看起来有戏但我没有为它重调提示词；`summarize` 的工具描述仍然含糊；评审可以换成 `claude-sonnet-5-5`"——而不是等用户来问还有没有。包括那些看起来需要的改动超出目标允许范围的内容。

The report is what lets the user trust the diff enough to merge it. Be specific about *why* each change helps; "reworded the system prompt" is not enough.

这份报告让用户对 diff 产生足以合并的信任。要具体说明*为什么*每项改动有帮助；"改写了系统提示词"是不够的。

If the eval and flow directory aren't already committed, offer the same three-way choice as `build-eval.md` §Make it durable (commit eval / commit eval + transcripts / don't), with fresh file and size counts now that there are multiple rounds. Recommend committing the eval - the harness change going into this PR is only half the value; the other half is being able to re-baseline on the next model without rebuilding the eval.

如果评测与流程目录尚未提交，提供与 `build-eval.md` §Make it durable 相同的三选一（提交评测 / 提交评测 + 对话记录 / 不提交），并在已有多个轮次之后给出新的文件与大小统计。建议提交评测——进入这个 PR 的 harness 改动只价值一半；另一半是能够在下一个模型上重新采集基线而不必重建评测。

---

## Failure modes to avoid / 应避免的失败模式

- **Touching the held-out set.** The split is the only thing standing between a real improvement and an overfit one. Don't open test transcripts, and don't let test-case content inform a proposed change - the analyzer reads train, the headline comes from test, and that wall is the result's credibility.
  **触碰保留集。**切分是真实改进与过拟合改进之间的唯一屏障。不要打开测试对话记录，也不要让测试用例的内容影响改动提议——分析器读训练，头条来自测试，这堵墙就是结果的可信度。

- **Leaving ground truth reachable.** If the model-under-test can read the expected answers from disk, the loop will eventually find that path and "win" without improving anything. Isolate answers structurally; don't rely on instructions. On a public benchmark, "reachable" includes the internet - a web-enabled agent can fetch a solutions mirror (see Step 3).
  **让标准答案保持可达。**如果被测模型能从磁盘读到期望答案，循环最终会找到那条路径并在毫无改进的情况下"获胜"。在结构上隔离答案；不要依赖指令。在公开基准上，"可达"包括互联网——有网页能力的代理可以抓取解答镜像（见步骤 3）。

- **Trusting an implausible jump.** When a model-graded score leaps further than the change could reasonably explain, the likeliest cause is the judge being gamed, not the app getting better. Spot-check by hand before celebrating.
  **相信不可信的跳升。**当模型评审的分数跃升幅度超过改动能合理解释的范围时，最可能的原因是评审被钻了空子，而不是应用变好了。庆祝之前先手工抽查。

- **Climbing on an untrustworthy eval.** Skipping Step 0.5 means you might be tuning against a disconnected mechanism or a mis-aggregated headline number - "improving" an artifact and shipping nothing. Prove the mechanism is wired and the headline recomputes from raw per-case results *before* the first round.
  **在不可信的评测上爬山。**跳过步骤 0.5 意味着你可能在对着一个没接通的机制或聚合错误的头条数字调优——"改进"了产物却什么也没交付。在第一轮*之前*证明机制已接通、头条数字能从原始逐用例结果重算出来。

- **Grinding content at a wall.** Adding more content for a feature that hasn't moved in three rounds, without first asking whether the failure is the grader's, the harness's, or structural. Step 4.5 is the check.
  **对着墙硬磨内容。**为一个三轮都没动的特性继续添加内容，却不先问失败是评分器的、harness 的还是结构性的。步骤 4.5 就是这道检查。

- **Averaging refusals into the score.** A safety refusal is not a capability failure, and a harness-killed attempt is neither. Record a failure class per attempt (refusal / harness-or-serving error / timeout / genuine failure), report refusal-zeros separately from capability-zeros, and pre-register the scrub predicate for anomalous zeros (e.g. completed with score 0 at a wall-clock and request count far below the task's normal floor), reporting raw and scrubbed - decide the rule before the scores exist.
  **把拒答平均进分数。**安全拒答不是能力失败，被框架杀死的尝试两者都不是。为每次尝试记录失败类别（拒答 / 框架或服务错误 / 超时 / 真实失败），把拒答零分与能力零分分开报告，并为异常零分预登记清洗谓词（例如在远低于该任务正常下限的墙钟时间和请求数内以 0 分完成），同时报告原始与清洗后的结果——在分数存在之前定好规则。

- **Letting the platform shift under the loop.** If the serving platform or harness changes execution semantics mid-run - environment reuse, batching, defaults - rounds stop being comparable. Pin the runner and its dependencies for the whole loop, and verify the semantics you depend on from run artifacts (e.g. one environment per case) rather than assuming them.
  **让平台在循环之下发生漂移。**如果服务平台或 harness 在运行中途改变执行语义——环境复用、批处理、默认值——各轮就不再可比。在整个循环期间固定运行器及其依赖，并从运行产物中核实你所依赖的语义（例如每个用例一个环境），而不是假定它们成立。

- **Climbing on a gain you can't explain.** If the score moved but you can't point to the behavior that moved it - a shift in how often the mechanism engaged, specific flips in the traces - the "win" is as likely noise or a measurement artifact as a real improvement. Tie every delta to a mechanism; distrust the ones you can't.
  **建立在你无法解释的收益之上。**如果分数动了但你指不出是哪个行为让它动——机制触发频率的变化、轨迹中的具体翻转——这个"获胜"与噪声或度量伪影的可能性一样大，与真实改进的可能性也一样大。把每个增量都挂钩到一个机制上；挂钩不上的就不要相信。

- **Overgeneralizing from one case.** Seeing a pattern in a single failure and rewriting the whole prompt around it is the most common way to make the score go down. The analyzer should cite the traces that motivate a change, describe the *behavior* rather than pasting the *content* of the failures into the prompt, and scale the change to how many cases show the behavior, not to one vivid failure.
  **从单个用例过度泛化。**在单个失败中看到一种模式就围绕它重写整个提示词，是让分数下降的最常见方式。分析器应引用支撑改动的轨迹，描述*行为*而不是把失败的*内容*粘贴进提示词，并按展现该行为的用例数量来定改动规模，而不是按一个鲜活的失败。

- **Untracked bundling.** Stacking several unrelated edits into one round's change means that if it helps you won't know which part did the work, and if it hurts you won't know which part to revert. One idea per round: one hypothesis about why cases fail, even when its patch touches several places.
  **不加追踪的捆绑。**把几个不相关的编辑叠进一轮的改动意味着：如果它有帮助，你不知道是哪部分起了作用；如果它有害，你不知道该回退哪部分。每轮一个想法：一个关于用例为何失败的假设，即使其补丁触及多处。

- **Changes too small to see.** A change whose best-case gain sits inside the noise floor is kept or reverted largely by chance, and a string of them spends full passes learning nothing (Step 4).
  **小到看不见的改动。**最好情况下的收益仍落在噪声底之内的改动，保留或回退基本靠运气，一连串这样的改动会耗掉一轮又一轮的完整跑批却一无所获（步骤 4）。

- **Accumulating fluff.** Without the per-round de-fluff pass, prompts grow a sediment of vague, unfalsifiable advice that costs tokens and dilutes the instructions that matter.
  **累积水分。**没有每轮的去水分处理，提示词会长出一层模糊、不可证伪的建议沉积物，既费 token 又稀释真正重要的指令。

- **Losing state.** If the session is interrupted, the next session should be able to read `_state.json` and the `vN/` directories and pick up exactly where this one left off. Write state after every round, not at the end. If the proposer/analyzer runs as its own long-lived session, its working artifacts - the traces and scores it was handed, and its rationale for each change - belong under the flow directory too, so an interrupted loop can reconstruct the proposer, not just the scores.
  **丢失状态。**如果会话被中断，下一个会话应能读取 `_state.json` 和各 `vN/` 目录并从此会话中断的地方精确接续。每轮结束都写状态，而不是最后才写。如果提议者/分析器作为自己的长驻会话运行，它的工作产物——交给它的轨迹和分数，以及它对每项改动的理由——也应放在流程目录之下，这样被中断的循环能重建提议者本身，而不只是分数。

- **Editing off-limits content.** The user told you what not to touch in Step 1. A change that improves the score but violates a constraint the user stated is not an improvement.
  **编辑禁区内容。**用户在步骤 1 已告诉过你什么不能碰。提升分数但违反用户明示约束的改动不是改进。

- **Treating the budget as a wall (or ignoring it)** - when cost is a guardrail. Recompute cumulative spend from the `results.jsonl` files - plus the billed usage on any `errors.jsonl` failed attempts - and check it against the Step 2 ceiling every round. If you're going to exceed it, ask first. Equally, don't stop cold at the limit while test is clearly still climbing without *offering* to continue.
  **把预算当墙（或无视预算）**——当成本是护栏时。从各 `results.jsonl` 文件重新计算累计花费——加上 `errors.jsonl` 中失败尝试的计费用量——并每轮对照步骤 2 的上限核查。如果要超限，先询问。同样，当测试明显仍在爬升时，不要在上限处戛然而止而不*提出*可以继续。
