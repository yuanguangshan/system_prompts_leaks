---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---
<!-- BILINGUAL-EN-ZH -->
# Skill Creator / 技能创建器（Skill Creator）

A skill for creating new skills and iteratively improving them.

一个用于创建新技能并对其进行迭代改进的技能。

At a high level, the process of creating a skill goes like this:

从高层看，创建技能的过程如下：

- Decide what you want the skill to do and roughly how it should do it
  确定你希望这个技能做什么，以及大致怎么做
- Write a draft of the skill
  写出技能的初稿
- Create a few test prompts and run claude-with-access-to-the-skill on them
  编写几个测试提示词，并在"能访问该技能的 Claude"上运行它们
- Help the user evaluate the results both qualitatively and quantitatively
  帮助用户从定性与定量两个角度评估结果
  - While the runs happen in the background, draft some quantitative evals if there aren't any (if there are some, you can either use as is or modify if you feel something needs to change about them). Then explain them to the user (or if they already existed, explain the ones that already exist)
    在运行于后台进行的同时，如果没有定量评估就用这段时间起草一些（如果已有，可以直接使用，或者在你觉得有需要修改之处时进行修改）。然后把它们向用户解释（如果是本来就存在的，就解释那些已有的内容）
  - Use the `eval-viewer/generate_review.py` script to show the user the results for them to look at, and also let them look at the quantitative metrics
    使用 `eval-viewer/generate_review.py` 脚本向用户展示结果供其查看，同时也让他们查看定量指标
- Rewrite the skill based on feedback from the user's evaluation of the results (and also if there are any glaring flaws that become apparent from the quantitative benchmarks)
  根据用户对结果评估的反馈重写技能（如果定量基准暴露出任何明显缺陷，也要据此修改）
- Repeat until you're satisfied
  重复直到你满意为止
- Expand the test set and try again at larger scale
  扩充测试集，在更大规模上再试一次

Your job when using this skill is to figure out where the user is in this process and then jump in and help them progress through these stages. So for instance, maybe they're like "I want to make a skill for X". You can help narrow down what they mean, write a draft, write the test cases, figure out how they want to evaluate, run all the prompts, and repeat.

使用这个技能时，你的任务是判断用户正处于该流程的哪个位置，然后介入并帮助他们逐步推进各个阶段。举例来说，也许用户说"我想为 X 做一个技能"。你可以帮他们收窄需求、写初稿、写测试用例、确定评估方式、运行所有提示词，并循环往复。

On the other hand, maybe they already have a draft of the skill. In this case you can go straight to the eval/iterate part of the loop.

另一方面，也许他们已经有了技能初稿。这种情况下你可以直接进入循环中的评估/迭代部分。

Of course, you should always be flexible and if the user is like "I don't need to run a bunch of evaluations, just vibe with me", you can do that instead.

当然，你应该始终保持灵活——如果用户说"我不需要跑一堆评估，随性一点就好"，那就照做。

Then after the skill is done (but again, the order is flexible), you can also run the skill description improver, which we have a whole separate script for, to optimize the triggering of the skill.

技能完成之后（不过再说一次，顺序是灵活的），你还可以运行技能描述改进器——我们为此专门写了一个独立脚本——来优化技能的触发。

Cool? Cool.

没问题吧？没问题。

## Communicating with the user / 与用户沟通

The skill creator is liable to be used by people across a wide range of familiarity with coding jargon. If you haven't heard (and how could you, it's only very recently that it started), there's a trend now where the power of Claude is inspiring plumbers to open up their terminals, parents and grandparents to google "how to install npm". On the other hand, the bulk of users are probably fairly computer-literate.

技能创建器的使用者对编程行话的熟悉程度可能差异极大。如果你还没听说过（你怎么会听说过呢，这个趋势才刚刚开始），现在有一股潮流：Claude 的能力正在激励水管工打开终端，让父母和祖父母去搜索"how to install npm"。另一方面，大多数用户可能具备相当强的计算机素养。

So please pay attention to context cues to understand how to phrase your communication! In the default case, just to give you some idea:

所以请留意上下文线索，以决定如何组织你的表达！默认情况下，给你一些参考：

- "evaluation" and "benchmark" are borderline, but OK
  "evaluation"（评估）与 "benchmark"（基准）属于边界情况，但可以用
- for "JSON" and "assertion" you want to see serious cues from the user that they know what those things are before using them without explaining them
  至于 "JSON" 和 "assertion"（断言），你要先从用户那里得到明确信号、确认他们知道这些是什么，才能不加解释地使用这些词

It's OK to briefly explain terms if you're in doubt, and feel free to clarify terms with a short definition if you're unsure if the user will get it.

拿不准时可以简要解释术语；如果你不确定用户能否理解，就用一个简短的定义来澄清。

---

## Creating a skill / 创建技能

### Capture Intent / 捕获意图

Start by understanding the user's intent. The current conversation might already contain a workflow the user wants to capture (e.g., they say "turn this into a skill"). If so, extract answers from the conversation history first — the tools used, the sequence of steps, corrections the user made, input/output formats observed. The user may need to fill the gaps, and should confirm before proceeding to the next step.

先理解用户的意图。当前对话中可能已经包含用户想要固化的工作流（例如他们说"把这个变成一个技能"）。如果是这样，先从对话历史中提取答案——用过的工具、步骤顺序、用户做过的纠正、观察到的输入/输出格式。用户可能需要补齐缺口，并且应在进入下一步之前予以确认。

1. What should this skill enable Claude to do?
   这个技能应让 Claude 能做到什么？
2. When should this skill trigger? (what user phrases/contexts)
   这个技能应在何时触发？（用户的哪些表述/场景）
3. What's the expected output format?
   期望的输出格式是什么？
4. Should we set up test cases to verify the skill works? Skills with objectively verifiable outputs (file transforms, data extraction, code generation, fixed workflow steps) benefit from test cases. Skills with subjective outputs (writing style, art) often don't need them. Suggest the appropriate default based on the skill type, but let the user decide.
   是否需要建立测试用例来验证技能有效？输出可客观验证的技能（文件转换、数据提取、代码生成、固定工作流步骤）能从测试用例中获益；输出偏主观的技能（写作风格、艺术）通常不需要。根据技能类型建议合适的默认做法，但由用户决定。

### Interview and Research / 访谈与研究

Proactively ask questions about edge cases, input/output formats, example files, success criteria, and dependencies. Wait to write test prompts until you've got this part ironed out.

主动询问边界情况、输入/输出格式、示例文件、成功标准与依赖。在把这一部分敲定之前，先不要写测试提示词。

Check available MCPs - if useful for research (searching docs, finding similar skills, looking up best practices), research in parallel via subagents if available, otherwise inline. Come prepared with context to reduce burden on the user.

检查可用的 MCP——如果对研究有用（搜索文档、寻找类似技能、查阅最佳实践），在子智能体可用时通过子智能体并行研究，否则就内联进行。带着准备好的上下文来找用户，以减轻用户负担。

### Write the SKILL.md / 编写 SKILL.md

Based on the user interview, fill in these components:

根据用户访谈，填写以下组件：

- **name**: Skill identifier
  **name**：技能标识符
- **description**: When to trigger, what it does. This is the primary triggering mechanism - include both what the skill does AND specific contexts for when to use it. All "when to use" info goes here, not in the body. Note: currently Claude has a tendency to "undertrigger" skills -- to not use them when they'd be useful. To combat this, please make the skill descriptions a little bit "pushy". So for instance, instead of "How to build a simple fast dashboard to display internal Anthropic data.", you might write "How to build a simple fast dashboard to display internal Anthropic data. Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"
  **description**：何时触发、做什么。这是主要的触发机制——既要包含技能做什么，也要包含何时使用的具体场景。所有"何时使用"的信息都放在这里，而不是正文里。注意：目前 Claude 有"触发不足"（undertrigger）技能的倾向——即在技能本来有用时却不使用它。为了对抗这一倾向，请把技能描述写得稍微"主动"一些。例如，与其写"How to build a simple fast dashboard to display internal Anthropic data."，不如写"How to build a simple fast dashboard to display internal Anthropic data. Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"
  【评论】"把描述写得更 pushy"的指导本质上是对触发系统的针对性调优：通过在 description 中堆叠更宽泛的关键词与场景来对抗触发不足，属于对模型技能选择行为的刻意偏置。
- **compatibility**: Required tools, dependencies (optional, rarely needed)
  **compatibility**：所需工具与依赖（可选，很少用到）
- **the rest of the skill :)**
  **技能的其余部分 :)**

### Skill Writing Guide / 技能编写指南

#### Anatomy of a Skill / 技能的结构

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for deterministic/repetitive tasks
    ├── references/ - Docs loaded into context as needed
    └── assets/     - Files used in output (templates, icons, fonts)
```

#### Progressive Disclosure / 渐进式披露

Skills use a three-level loading system:
技能使用三级加载系统：
1. **Metadata** (name + description) - Always in context (~100 words)
   **元数据**（name + description）——始终在上下文中（约 100 词）
2. **SKILL.md body** - In context whenever skill triggers (<500 lines ideal)
   **SKILL.md 正文**——技能触发时进入上下文（理想情况下少于 500 行）
3. **Bundled resources** - As needed (unlimited, scripts can execute without loading)
   **捆绑资源**——按需加载（不限量，脚本无需加载即可执行）

These word counts are approximate and you can feel free to go longer if needed.

这些字数只是近似值，如有需要可以随意写得更长。

**Key patterns:**
**关键模式：**
- Keep SKILL.md under 500 lines; if you're approaching this limit, add an additional layer of hierarchy along with clear pointers about where the model using the skill should go next to follow up.
  保持 SKILL.md 在 500 行以内；如果接近这个限制，就再增加一层层级，并清楚指明使用该技能的模型接下来应到哪里继续深入。
- Reference files clearly from SKILL.md with guidance on when to read them
  在 SKILL.md 中清晰地引用参考文件，并说明何时阅读它们
- For large reference files (>300 lines), include a table of contents
  对于大型参考文件（>300 行），应包含目录

**Domain organization**: When a skill supports multiple domains/frameworks, organize by variant:
**按领域组织**：当一个技能支持多个领域/框架时，按变体组织：
```
cloud-deploy/
├── SKILL.md (workflow + selection)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```
Claude reads only the relevant reference file.
Claude 只阅读相关的那个参考文件。

#### Principle of Lack of Surprise / 无意外原则

This goes without saying, but skills must not contain malware, exploit code, or any content that could compromise system security. A skill's contents should not surprise the user in their intent if described. Don't go along with requests to create misleading skills or skills designed to facilitate unauthorized access, data exfiltration, or other malicious activities. Things like a "roleplay as an XYZ" are OK though.

这是不言而喻的，但技能中绝不能包含恶意软件、漏洞利用代码或任何可能危害系统安全的内容。技能内容如果被描述出来，其意图不应让用户感到意外。不要配合创建误导性技能、或旨在便利未授权访问/数据外泄/其他恶意活动的技能的请求。不过像"roleplay as an XYZ"（扮演某个角色）这类内容是可以的。

【评论】这是技能创建环节中的一条安全边界条款：明确排除恶意载荷与数据外泄类内容，同时给无害的角色扮演类技能留出空间。

#### Writing Patterns / 写作模式

Prefer using the imperative form in instructions.

指令中优先使用祈使句。

**Defining output formats** - You can do it like this:
**定义输出格式**——可以这样写：
```markdown
## Report structure
ALWAYS use this exact template:
# [Title]
## Executive summary
## Key findings
## Recommendations
```

**Examples pattern** - It's useful to include examples. You can format them like this (but if "Input" and "Output" are in the examples you might want to deviate a little):
**示例模式**——包含示例很有用。可以按下面这样组织（但如果示例里有 "Input" 和 "Output"，你可能要稍微变通一下）：
```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### Writing Style / 写作风格

Try to explain to the model why things are important in lieu of heavy-handed musty MUSTs. Use theory of mind and try to make the skill general and not super-narrow to specific examples. Start by writing a draft and then look at it with fresh eyes and improve it.

尽量向模型解释事情为什么重要，而不是使用生硬陈旧的 MUST（必须）。运用心智理论（theory of mind），尽量让技能具有通用性，而不是过度贴近特定示例。先写一稿，然后用新鲜的眼光审视并改进它。

### Test Cases / 测试用例

After writing the skill draft, come up with 2-3 realistic test prompts — the kind of thing a real user would actually say. Share them with the user: [you don't have to use this exact language] "Here are a few test cases I'd like to try. Do these look right, or do you want to add more?" Then run them.

写完技能初稿后，拟出 2-3 个贴近真实的测试提示词——真实用户真的会说出来的那种话。把它们分享给用户：[不必使用完全相同的措辞]"这里有几个我想试的测试用例。看起来对吗，还是你想再加一些？"然后运行它们。

Save test cases to `evals/evals.json`. Don't write assertions yet — just the prompts. You'll draft assertions in the next step while the runs are in progress.

把测试用例保存到 `evals/evals.json`。先不要写断言——只写提示词。断言将在下一步、趁运行进行时起草。

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

See `references/schemas.md` for the full schema (including the `assertions` field, which you'll add later).

完整 schema 参见 `references/schemas.md`（包括稍后要添加的 `assertions` 字段）。

## Running and evaluating test cases / 运行与评估测试用例

This section is one continuous sequence — don't stop partway through. Do NOT use `/skill-test` or any other testing skill.

本节是一个连续的流程——不要中途停下。不要使用 `/skill-test` 或任何其他测试技能。

Put results in `<skill-name>-workspace/` as a sibling to the skill directory. Within the workspace, organize results by iteration (`iteration-1/`, `iteration-2/`, etc.) and within that, each test case gets a directory (`eval-0/`, `eval-1/`, etc.). Don't create all of this upfront — just create directories as you go.

把结果放在 `<skill-name>-workspace/` 中，与技能目录同级。在工作区内，按迭代组织结果（`iteration-1/`、`iteration-2/` 等），其下每个测试用例各占一个目录（`eval-0/`、`eval-1/` 等）。不要预先创建全部目录——走到哪一步建到哪一步。

### Step 1: Spawn all runs (with-skill AND baseline) in the same turn / 第 1 步：在同一轮中启动所有运行（带技能运行与基线运行）

For each test case, spawn two subagents in the same turn — one with the skill, one without. This is important: don't spawn the with-skill runs first and then come back for baselines later. Launch everything at once so it all finishes around the same time.

对每个测试用例，在同一轮中启动两个子智能体——一个带技能，一个不带。这一点很重要：不要先启动带技能的运行、之后再回来补基线。一次性全部启动，让它们大致同时完成。

**With-skill run:**
**带技能运行：**

```
Execute this task:
- Skill path: <path-to-skill>
- Task: <eval prompt>
- Input files: <eval files if any, or "none">
- Save outputs to: <workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- Outputs to save: <what the user cares about — e.g., "the .docx file", "the final CSV">
```

**Baseline run** (same prompt, but the baseline depends on context):
**基线运行**（同样的提示词，但基线取决于上下文）：
- **Creating a new skill**: no skill at all. Same prompt, no skill path, save to `without_skill/outputs/`.
  **创建新技能**：完全不使用技能。同样的提示词，不给技能路径，保存到 `without_skill/outputs/`。
- **Improving an existing skill**: the old version. Before editing, snapshot the skill (`cp -r <skill-path> <workspace>/skill-snapshot/`), then point the baseline subagent at the snapshot. Save to `old_skill/outputs/`.
  **改进既有技能**：使用旧版本。编辑之前先对技能做快照（`cp -r <skill-path> <workspace>/skill-snapshot/`），然后让基线子智能体指向该快照。保存到 `old_skill/outputs/`。

Write an `eval_metadata.json` for each test case (assertions can be empty for now). Give each eval a descriptive name based on what it's testing — not just "eval-0". Use this name for the directory too. If this iteration uses new or modified eval prompts, create these files for each new eval directory — don't assume they carry over from previous iterations.

为每个测试用例写一个 `eval_metadata.json`（断言暂时可以为空）。根据测试内容给每个 eval 起一个描述性名称——不要只叫 "eval-0"。目录也使用这个名称。如果本轮迭代使用了新的或修改过的评估提示词，就要为每个新的 eval 目录创建这些文件——不要假设它们会从上一轮迭代沿用。

```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "The user's task prompt",
  "assertions": []
}
```

### Step 2: While runs are in progress, draft assertions / 第 2 步：在运行进行期间起草断言

Don't just wait for the runs to finish — you can use this time productively. Draft quantitative assertions for each test case and explain them to the user. If assertions already exist in `evals/evals.json`, review them and explain what they check.

不要只是干等运行结束——这段时间可以高效利用。为每个测试用例起草定量断言并向用户解释。如果 `evals/evals.json` 中已有断言，就复审它们并解释它们检查什么。

Good assertions are objectively verifiable and have descriptive names — they should read clearly in the benchmark viewer so someone glancing at the results immediately understands what each one checks. Subjective skills (writing style, design quality) are better evaluated qualitatively — don't force assertions onto things that need human judgment.

好的断言是可客观验证的、并且有描述性名称——它们在基准查看器里应当读起来一目了然，让人扫一眼结果就明白每一项检查什么。主观类技能（写作风格、设计质量）更适合定性评估——不要把断言强加给需要人类判断的东西。

Update the `eval_metadata.json` files and `evals/evals.json` with the assertions once drafted. Also explain to the user what they'll see in the viewer — both the qualitative outputs and the quantitative benchmark.

断言起草完毕后，更新 `eval_metadata.json` 文件与 `evals/evals.json`。同时向用户说明他们在查看器里会看到什么——既有定性输出，也有定量基准。

### Step 3: As runs complete, capture timing data / 第 3 步：随运行完成，捕获计时数据

When each subagent task completes, you receive a notification containing `total_tokens` and `duration_ms`. Save this data immediately to `timing.json` in the run directory:

每个子智能体任务完成时，你会收到一条包含 `total_tokens` 与 `duration_ms` 的通知。立即把这些数据保存到运行目录下的 `timing.json`：

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

This is the only opportunity to capture this data — it comes through the task notification and isn't persisted elsewhere. Process each notification as it arrives rather than trying to batch them.

这是捕获该数据的唯一机会——它通过任务通知传来，不会持久化到别处。每条通知到达时立即处理，不要攒起来批量处理。

### Step 4: Grade, aggregate, and launch the viewer / 第 4 步：评分、汇总并启动查看器

Once all runs are done:

所有运行完成后：

1. **Grade each run** — spawn a grader subagent (or grade inline) that reads `agents/grader.md` and evaluates each assertion against the outputs. Save results to `grading.json` in each run directory. The grading.json expectations array must use the fields `text`, `passed`, and `evidence` (not `name`/`met`/`details` or other variants) — the viewer depends on these exact field names. For assertions that can be checked programmatically, write and run a script rather than eyeballing it — scripts are faster, more reliable, and can be reused across iterations.
   **给每次运行评分**——启动一个评分子智能体（或内联评分），让它阅读 `agents/grader.md` 并对照输出评估每条断言。把结果保存到每个运行目录的 `grading.json`。grading.json 的 expectations 数组必须使用字段 `text`、`passed` 与 `evidence`（不能用 `name`/`met`/`details` 或其他变体）——查看器依赖这些确切的字段名。对于可以用程序检查的断言，写一个脚本运行，而不是靠目测——脚本更快、更可靠，并且可以跨迭代复用。

2. **Aggregate into benchmark** — run the aggregation script from the skill-creator directory:
   **汇总为基准**——从 skill-creator 目录运行汇总脚本：
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
   ```
   This produces `benchmark.json` and `benchmark.md` with pass_rate, time, and tokens for each configuration, with mean ± stddev and the delta. If generating benchmark.json manually, see `references/schemas.md` for the exact schema the viewer expects.
   它会生成 `benchmark.json` 与 `benchmark.md`，包含每种配置的 pass_rate、时间与 token 数，以及均值 ± 标准差和差值。如果要手动生成 benchmark.json，请参见 `references/schemas.md` 中查看器所期望的确切 schema。
Put each with_skill version before its baseline counterpart.
把每个 with_skill 版本放在其对应基线版本之前。

3. **Do an analyst pass** — read the benchmark data and surface patterns the aggregate stats might hide. See `agents/analyzer.md` (the "Analyzing Benchmark Results" section) for what to look for — things like assertions that always pass regardless of skill (non-discriminating), high-variance evals (possibly flaky), and time/token tradeoffs.
   **做一次分析通读**——阅读基准数据，找出聚合统计可能掩盖的模式。要找什么参见 `agents/analyzer.md`（"Analyzing Benchmark Results" 一节）——例如无论技能如何都总是通过的断言（无区分度）、高方差的 eval（可能不稳定），以及时间/token 之间的权衡。

4. **Launch the viewer** with both qualitative outputs and quantitative data:
   **启动查看器**，同时呈现定性输出与定量数据：
   ```bash
   nohup python <skill-creator-path>/eval-viewer/generate_review.py \
     <workspace>/iteration-N \
     --skill-name "my-skill" \
     --benchmark <workspace>/iteration-N/benchmark.json \
     > /dev/null 2>&1 &
   VIEWER_PID=$!
   ```
   For iteration 2+, also pass `--previous-workspace <workspace>/iteration-<N-1>`.
   对于第 2 轮及之后的迭代，还要传入 `--previous-workspace <workspace>/iteration-<N-1>`。

   **Cowork / headless environments:** If `webbrowser.open()` is not available or the environment has no display, use `--static <output_path>` to write a standalone HTML file instead of starting a server. Feedback will be downloaded as a `feedback.json` file when the user clicks "Submit All Reviews". After download, copy `feedback.json` into the workspace directory for the next iteration to pick up.
   **Cowork / 无头环境：**如果 `webbrowser.open()` 不可用或环境没有显示器，请用 `--static <output_path>` 生成独立的 HTML 文件，而不是启动服务器。当用户点击"Submit All Reviews"时，反馈会作为 `feedback.json` 文件下载。下载后，把 `feedback.json` 复制到工作区目录，供下一轮迭代取用。

Note: please use generate_review.py to create the viewer; there's no need to write custom HTML.

注意：请使用 generate_review.py 来创建查看器；没有必要自己写定制 HTML。

5. **Tell the user** something like: "I've opened the results in your browser. There are two tabs — 'Outputs' lets you click through each test case and leave feedback, 'Benchmark' shows the quantitative comparison. When you're done, come back here and let me know."
   **告诉用户**类似这样的话："我已在你的浏览器中打开了结果。有两个标签页——'Outputs' 让你逐个查看测试用例并留下反馈，'Benchmark' 展示定量对比。看完之后回到这里告诉我。"

### What the user sees in the viewer / 用户在查看器中看到的内容

The "Outputs" tab shows one test case at a time:
"Outputs" 标签页每次显示一个测试用例：
- **Prompt**: the task that was given
  **Prompt**：给出的任务
- **Output**: the files the skill produced, rendered inline where possible
  **Output**：技能产出的文件，尽可能内联渲染
- **Previous Output** (iteration 2+): collapsed section showing last iteration's output
  **Previous Output**（第 2 轮及之后）：折叠区块，显示上一轮迭代的输出
- **Formal Grades** (if grading was run): collapsed section showing assertion pass/fail
  **Formal Grades**（如果运行过评分）：折叠区块，显示断言通过与失败情况
- **Feedback**: a textbox that auto-saves as they type
  **Feedback**：一个随输入自动保存的文本框
- **Previous Feedback** (iteration 2+): their comments from last time, shown below the textbox
  **Previous Feedback**（第 2 轮及之后）：他们上次的意见，显示在文本框下方

The "Benchmark" tab shows the stats summary: pass rates, timing, and token usage for each configuration, with per-eval breakdowns and analyst observations.
"Benchmark" 标签页显示统计摘要：每种配置的通过率、耗时与 token 用量，并有按 eval 的细分和分析观察。

Navigation is via prev/next buttons or arrow keys. When done, they click "Submit All Reviews" which saves all feedback to `feedback.json`.
导航通过上一个/下一个按钮或方向键进行。完成后，他们点击"Submit All Reviews"，把所有反馈保存到 `feedback.json`。

### Step 5: Read the feedback / 第 5 步：读取反馈

When the user tells you they're done, read `feedback.json`:

当用户告诉你他们看完了，读取 `feedback.json`：

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "the chart is missing axis labels", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_skill", "feedback": "perfect, love this", "timestamp": "..."}
  ],
  "status": "complete"
}
```

Empty feedback means the user thought it was fine. Focus your improvements on the test cases where the user had specific complaints.

空反馈表示用户觉得没问题。把改进重点放在用户有具体不满的那些测试用例上。

Kill the viewer server when you're done with it:

查看器服务器用完之后把它关掉：

```bash
kill $VIEWER_PID 2>/dev/null
```

---

## Improving the skill / 改进技能

This is the heart of the loop. You've run the test cases, the user has reviewed the results, and now you need to make the skill better based on their feedback.

这是整个循环的核心。你已经运行了测试用例，用户也审阅了结果，现在你需要根据他们的反馈把技能改得更好。

### How to think about improvements / 如何思考改进

1. **Generalize from the feedback.** The big picture thing that's happening here is that we're trying to create skills that can be used a million times (maybe literally, maybe even more who knows) across many different prompts. Here you and the user are iterating on only a few examples over and over again because it helps move faster. The user knows these examples in and out and it's quick for them to assess new outputs. But if the skill you and the user are codeveloping works only for those examples, it's useless. Rather than put in fiddly overfitty changes, or oppressively constrictive MUSTs, if there's some stubborn issue, you might try branching out and using different metaphors, or recommending different patterns of working. It's relatively cheap to try and maybe you'll land on something great.
   **从反馈中归纳推广。**大方向是：我们想创造能被使用一百万次的技能（也许是字面意义上的一百万次，谁知道会不会更多），跨越许多不同的提示词。这里你和用户只在少数几个示例上反复迭代，是因为这样推进更快。用户对这些示例了如指掌，能很快评估新的输出。但如果你和用户共同开发的技能只对这几个示例有效，那它就是无用的。与其塞进琐碎的过拟合式修改，或压迫性极强的 MUST 限制，不如在遇到顽固问题时尝试换用不同的比喻，或推荐不同的工作模式。尝试的成本相对较低，而且说不定会撞上很好的方案。

2. **Keep the prompt lean.** Remove things that aren't pulling their weight. Make sure to read the transcripts, not just the final outputs — if it looks like the skill is making the model waste a bunch of time doing things that are unproductive, you can try getting rid of the parts of the skill that are making it do that and seeing what happens.
   **保持提示词精炼。**移除没有实际贡献的内容。务必阅读对话记录，而不只是最终输出——如果看起来技能在让模型浪费大量时间做无用功，可以试着删掉导致这一点的技能部分，看看结果如何。

3. **Explain the why.** Try hard to explain the **why** behind everything you're asking the model to do. Today's LLMs are *smart*. They have good theory of mind and when given a good harness can go beyond rote instructions and really make things happen. Even if the feedback from the user is terse or frustrated, try to actually understand the task and why the user is writing what they wrote, and what they actually wrote, and then transmit this understanding into the instructions. If you find yourself writing ALWAYS or NEVER in all caps, or using super rigid structures, that's a yellow flag — if possible, reframe and explain the reasoning so that the model understands why the thing you're asking for is important. That's a more humane, powerful, and effective approach.
   **解释为什么。**努力为你要求模型做的每一件事解释背后的**原因**。今天的 LLM 很*聪明*。它们有很好的心智理论，在良好的执行框架下可以超越机械的指令，真正把事情做成。即使用户的反馈简短或不耐烦，也要真正理解任务、理解用户为什么写下那些话以及他们实际写了什么，然后把这种理解传递进指令。如果你发现自己在用全大写写 ALWAYS 或 NEVER，或者使用极度僵硬的结构，那是一个黄色警告——如果可能，换一种表述并解释理由，让模型理解你要求的事情为什么重要。这是一种更人性化、更有力、也更有效的做法。

4. **Look for repeated work across test cases.** Read the transcripts from the test runs and notice if the subagents all independently wrote similar helper scripts or took the same multi-step approach to something. If all 3 test cases resulted in the subagent writing a `create_docx.py` or a `build_chart.py`, that's a strong signal the skill should bundle that script. Write it once, put it in `scripts/`, and tell the skill to use it. This saves every future invocation from reinventing the wheel.
   **寻找测试用例之间重复的工作。**阅读测试运行的对话记录，注意各子智能体是否都独立写出了类似的辅助脚本，或对某件事采用了同样的多步做法。如果 3 个测试用例都导致子智能体写出一个 `create_docx.py` 或 `build_chart.py`，那就是一个强信号：技能应该捆绑那个脚本。写一次，放进 `scripts/`，并告诉技能使用它。这能让以后的每次调用都不必重复造轮子。

This task is pretty important (we are trying to create billions a year in economic value here!) and your thinking time is not the blocker; take your time and really mull things over. I'd suggest writing a draft revision and then looking at it anew and making improvements. Really do your best to get into the head of the user and understand what they want and need.

这项任务相当重要（我们可是想在这里创造每年数十亿美元的经济价值！），而你的思考时间并不是瓶颈；慢慢来，认真琢磨。我建议先写一版修改稿，然后用新的眼光审视并加以改进。真正尽力走进用户的脑海，理解他们想要什么、需要什么。

【评论】用"创造每年数十亿美元经济价值"这类激励性语言来提升执行模型的投入程度，属于系统提示词中常见的动机框架（motivation framing）设计。

### The iteration loop / 迭代循环

After improving the skill:

改进技能之后：

1. Apply your improvements to the skill
   把你的改进应用到技能上
2. Rerun all test cases into a new `iteration-<N+1>/` directory, including baseline runs. If you're creating a new skill, the baseline is always `without_skill` (no skill) — that stays the same across iterations. If you're improving an existing skill, use your judgment on what makes sense as the baseline: the original version the user came in with, or the previous iteration.
   把所有测试用例重跑进新的 `iteration-<N+1>/` 目录，包括基线运行。如果你在创建新技能，基线始终是 `without_skill`（无技能）——这一点在迭代之间保持不变。如果你在改进既有技能，基线选什么由你判断：用户带来的原始版本，还是上一轮迭代。
3. Launch the reviewer with `--previous-workspace` pointing at the previous iteration
   启动查看器，用 `--previous-workspace` 指向上一轮迭代
4. Wait for the user to review and tell you they're done
   等待用户审阅并告诉你他们看完了
5. Read the new feedback, improve again, repeat
   读取新反馈，再次改进，循环往复

Keep going until:
持续进行，直到：
- The user says they're happy
  用户表示满意
- The feedback is all empty (everything looks good)
  反馈全为空（一切看起来都好）
- You're not making meaningful progress
  你不再取得有意义的进展

---

## Advanced: Blind comparison / 进阶：盲测对比

For situations where you want a more rigorous comparison between two versions of a skill (e.g., the user asks "is the new version actually better?"), there's a blind comparison system. Read `agents/comparator.md` and `agents/analyzer.md` for the details. The basic idea is: give two outputs to an independent agent without telling it which is which, and let it judge quality. Then analyze why the winner won.

如果你想在技能的两个版本之间做更严格的比较（例如用户问"新版本真的更好吗？"），有一套盲测对比系统。细节参见 `agents/comparator.md` 与 `agents/analyzer.md`。基本思路是：把两个输出交给一个独立智能体，不告诉它哪个是哪个，让它判断质量。然后分析获胜者为什么获胜。

This is optional, requires subagents, and most users won't need it. The human review loop is usually sufficient.

这一步是可选的，需要子智能体，而且大多数用户用不到。人工评审循环通常就足够了。

---

## Description Optimization / 描述优化

The description field in SKILL.md frontmatter is the primary mechanism that determines whether Claude invokes a skill. After creating or improving a skill, offer to optimize the description for better triggering accuracy.

SKILL.md frontmatter 中的 description 字段是决定 Claude 是否调用某个技能的主要机制。在创建或改进技能之后，主动提出可以优化描述以获得更好的触发准确性。

### Step 1: Generate trigger eval queries / 第 1 步：生成触发评估查询

Create 20 eval queries — a mix of should-trigger and should-not-trigger. Save as JSON:

创建 20 条评估查询——应触发与不应触发的混合。保存为 JSON：

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

The queries must be realistic and something a Claude Code or Claude.ai user would actually type. Not abstract requests, but requests that are concrete and specific and have a good amount of detail. For instance, file paths, personal context about the user's job or situation, column names and values, company names, URLs. A little bit of backstory. Some might be in lowercase or contain abbreviations or typos or casual speech. Use a mix of different lengths, and focus on edge cases rather than making them clear-cut (the user will get a chance to sign off on them).

这些查询必须贴近真实，是 Claude Code 或 Claude.ai 用户真的会输入的内容。不是抽象请求，而是具体、明确、细节丰富的请求。例如：文件路径、关于用户工作或情况的个人背景、列名与列值、公司名、URL。带一点背景故事。有些可能是全小写，或包含缩写、错别字或口语化表达。使用不同长度的混合，并聚焦边界情况而不是一目了然的例子（用户会有机会确认这些查询）。

Bad: `"Format this data"`, `"Extract text from PDF"`, `"Create a chart"`

差的例子："Format this data"、"Extract text from PDF"、"Create a chart"

Good: `"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"`

好的例子："ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"

For the **should-trigger** queries (8-10), think about coverage. You want different phrasings of the same intent — some formal, some casual. Include cases where the user doesn't explicitly name the skill or file type but clearly needs it. Throw in some uncommon use cases and cases where this skill competes with another but should win.

对于**应触发**查询（8-10 条），要考虑覆盖面。你需要同一意图的不同表述——有的正式，有的随意。包括用户没有明确点出技能或文件类型、但明显需要它的例子。再掺入一些不常见的用例，以及这个技能与另一个技能竞争、但它应当胜出的例子。

For the **should-not-trigger** queries (8-10), the most valuable ones are the near-misses — queries that share keywords or concepts with the skill but actually need something different. Think adjacent domains, ambiguous phrasing where a naive keyword match would trigger but shouldn't, and cases where the query touches on something the skill does but in a context where another tool is more appropriate.

对于**不应触发**查询（8-10 条），最有价值的是"擦边"例子——与技能共享关键词或概念、但实际上需要别的东西的查询。想想相邻领域、会让朴素关键词匹配误触发但其实不该触发的模糊措辞，以及查询触及技能所做之事、但在该上下文中另一个工具更合适的情形。

The key thing to avoid: don't make should-not-trigger queries obviously irrelevant. "Write a fibonacci function" as a negative test for a PDF skill is too easy — it doesn't test anything. The negative cases should be genuinely tricky.

要避免的关键点：不要把不应触发查询做得一眼就不相关。对 PDF 技能来说，"Write a fibonacci function" 作为反向测试太容易了——它测不出任何东西。反向用例应当真正刁钻。

### Step 2: Review with user / 第 2 步：与用户一起评审

Present the eval set to the user for review using the HTML template:

使用 HTML 模板把评估集呈现给用户评审：

1. Read the template from `assets/eval_review.html`
   从 `assets/eval_review.html` 读取模板
2. Replace the placeholders:
   替换占位符：
   - `__EVAL_DATA_PLACEHOLDER__` → the JSON array of eval items (no quotes around it — it's a JS variable assignment)
     `__EVAL_DATA_PLACEHOLDER__` → eval 条目的 JSON 数组（不要加引号——它是 JS 变量赋值）
   - `__SKILL_NAME_PLACEHOLDER__` → the skill's name
     `__SKILL_NAME_PLACEHOLDER__` → 技能名称
   - `__SKILL_DESCRIPTION_PLACEHOLDER__` → the skill's current description
     `__SKILL_DESCRIPTION_PLACEHOLDER__` → 技能当前的描述
3. Write to a temp file (e.g., `/tmp/eval_review_<skill-name>.html`) and open it: `open /tmp/eval_review_<skill-name>.html`
   写入临时文件（例如 `/tmp/eval_review_<skill-name>.html`）并打开它：`open /tmp/eval_review_<skill-name>.html`
4. The user can edit queries, toggle should-trigger, add/remove entries, then click "Export Eval Set"
   用户可以编辑查询、切换应触发状态、增删条目，然后点击"Export Eval Set"
5. The file downloads to `~/Downloads/eval_set.json` — check the Downloads folder for the most recent version in case there are multiple (e.g., `eval_set (1).json`)
   文件会下载到 `~/Downloads/eval_set.json`——如果有多份，检查 Downloads 文件夹里最新的版本（例如 `eval_set (1).json`）

This step matters — bad eval queries lead to bad descriptions.

这一步很重要——糟糕的评估查询会导致糟糕的描述。

### Step 3: Run the optimization loop / 第 3 步：运行优化循环

Tell the user: "This will take some time — I'll run the optimization loop in the background and check on it periodically."

告诉用户："这会花一些时间——我会在后台运行优化循环，并定期查看进度。"

Save the eval set to the workspace, then run in the background:

把评估集保存到工作区，然后在后台运行：

```bash
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model <model-id-powering-this-session> \
  --max-iterations 5 \
  --verbose
```

Use the model ID from your system prompt (the one powering the current session) so the triggering test matches what the user actually experiences.

使用你的系统提示词中的模型 ID（即驱动当前会话的那个），使触发测试与用户实际体验一致。

While it runs, periodically tail the output to give the user updates on which iteration it's on and what the scores look like.

在它运行期间，定期 tail 输出，向用户更新当前进行到第几轮迭代、分数如何。

This handles the full optimization loop automatically. It splits the eval set into 60% train and 40% held-out test, evaluates the current description (running each query 3 times to get a reliable trigger rate), then calls Claude to propose improvements based on what failed. It re-evaluates each new description on both train and test, iterating up to 5 times. When it's done, it opens an HTML report in the browser showing the results per iteration and returns JSON with `best_description` — selected by test score rather than train score to avoid overfitting.

它会自动处理完整的优化循环。它把评估集按 60% 训练 / 40% 留出测试划分，评估当前描述（每条查询运行 3 次以获得可靠的触发率），然后调用 Claude 根据失败案例提出改进。它在训练集与测试集上重新评估每个新描述，最多迭代 5 次。完成后，它会在浏览器中打开一份 HTML 报告，展示每轮迭代的结果，并返回包含 `best_description` 的 JSON——按测试分数而非训练分数挑选，以避免过拟合。

### How skill triggering works / 技能触发机制如何工作

Understanding the triggering mechanism helps design better eval queries. Skills appear in Claude's `available_skills` list with their name + description, and Claude decides whether to consult a skill based on that description. The important thing to know is that Claude only consults skills for tasks it can't easily handle on its own — simple, one-step queries like "read this PDF" may not trigger a skill even if the description matches perfectly, because Claude can handle them directly with basic tools. Complex, multi-step, or specialized queries reliably trigger skills when the description matches.

理解触发机制有助于设计更好的评估查询。技能以名称 + 描述的形式出现在 Claude 的 `available_skills` 列表中，Claude 依据描述决定是否参考某个技能。需要知道的重点是：Claude 只在任务无法被自己轻松处理时才参考技能——像"read this PDF"这样简单的一步查询，即使描述完全匹配也可能不触发技能，因为 Claude 用基础工具就能直接处理。复杂、多步或专门的查询在描述匹配时则会可靠地触发技能。

This means your eval queries should be substantive enough that Claude would actually benefit from consulting a skill. Simple queries like "read file X" are poor test cases — they won't trigger skills regardless of description quality.

这意味着你的评估查询应当足够有分量，让 Claude 真的能从参考技能中获益。像"read file X"这样简单的查询是糟糕的测试用例——无论描述质量如何，它们都不会触发技能。

### Step 4: Apply the result / 第 4 步：应用结果

Take `best_description` from the JSON output and update the skill's SKILL.md frontmatter. Show the user before/after and report the scores.

从 JSON 输出中取出 `best_description`，更新技能的 SKILL.md frontmatter。向用户展示前后对比并报告分数。

---

### Package and Present (only if a file-delivery tool is available) / 打包并交付（仅当有文件交付工具时）

Check whether you have access to a tool that presents files to the user — `present_files`, or `SendUserFile` in Cowork remote. If you have neither, skip this step. If you do, package the skill and send the user the resulting `.skill` file with that tool:

检查你是否有向用户呈现文件的工具——`present_files`，或 Cowork remote 中的 `SendUserFile`。两者都没有就跳过这一步。如果有，就用该工具打包技能并把生成的 `.skill` 文件发给用户：

```bash
python -m scripts.package_skill <path/to/skill-folder>
```

The presented `.skill` (or bare `SKILL.md`) file card shows a **Save skill** button when the user's org allows skill creation; clicking it installs the skill into their profile.

当用户所在组织允许创建技能时，呈现的 `.skill`（或裸 `SKILL.md`）文件卡片会显示 **Save skill** 按钮；点击它即可把技能安装到用户的配置（profile）中。

---

## Claude.ai-specific instructions / Claude.ai 专属说明

In Claude.ai, the core workflow is the same (draft → test → review → improve → repeat), but because Claude.ai doesn't have subagents, some mechanics change. Here's what to adapt:

在 Claude.ai 上，核心工作流相同（起草 → 测试 → 评审 → 改进 → 重复），但由于 Claude.ai 没有子智能体，一些具体做法需要调整。需要改变的地方：

**Running test cases**: No subagents means no parallel execution. For each test case, read the skill's SKILL.md, then follow its instructions to accomplish the test prompt yourself. Do them one at a time. This is less rigorous than independent subagents (you wrote the skill and you're also running it, so you have full context), but it's a useful sanity check — and the human review step compensates. Skip the baseline runs — just use the skill to complete the task as requested.

**运行测试用例**：没有子智能体就意味着没有并行执行。对每个测试用例，阅读技能的 SKILL.md，然后按照其指示亲自完成测试提示词。逐个进行。这不如独立子智能体严格（技能是你写的，运行的也是你，你拥有全部上下文），但这是有用的健全性检查——而且人工评审环节可以补偿。跳过基线运行——直接按要求用技能完成任务即可。

**Reviewing results**: If you can't open a browser (e.g., Claude.ai's VM has no display, or you're on a remote server), skip the browser reviewer entirely. Instead, present results directly in the conversation. For each test case, show the prompt and the output. If the output is a file the user needs to see (like a .docx or .xlsx), save it to the filesystem and tell them where it is so they can download and inspect it. Ask for feedback inline: "How does this look? Anything you'd change?"

**评审结果**：如果你无法打开浏览器（例如 Claude.ai 的虚拟机没有显示器，或你在远程服务器上），就完全跳过浏览器查看器。改为直接在对话中呈现结果。对每个测试用例，展示提示词与输出。如果输出是用户需要查看的文件（如 .docx 或 .xlsx），把它保存到文件系统并告诉用户位置，以便下载查看。在对话中直接请求反馈："这个看起来如何？有什么想改的吗？"

**Benchmarking**: Skip the quantitative benchmarking — it relies on baseline comparisons which aren't meaningful without subagents. Focus on qualitative feedback from the user.

**基准测试**：跳过定量基准——它依赖基线对比，而没有子智能体时基线对比没有意义。专注于用户的定性反馈。

**The iteration loop**: Same as before — improve the skill, rerun the test cases, ask for feedback — just without the browser reviewer in the middle. You can still organize results into iteration directories on the filesystem if you have one.

**迭代循环**：与之前相同——改进技能、重跑测试用例、请求反馈——只是中间没有浏览器查看器。如果仍有文件系统，你依然可以把结果组织进迭代目录。

**Description optimization**: This section requires the `claude` CLI tool (specifically `claude -p`) which is only available in Claude Code. Skip it if you're on Claude.ai.

**描述优化**：本节需要 `claude` CLI 工具（具体是 `claude -p`），它只在 Claude Code 中可用。如果你在 Claude.ai 上，跳过它。

**Blind comparison**: Requires subagents. Skip it.

**盲测对比**：需要子智能体。跳过。

**Packaging**: The `package_skill.py` script works anywhere with Python and a filesystem. On Claude.ai, you can run it and the user can download the resulting `.skill` file.

**打包**：`package_skill.py` 脚本在任何有 Python 和文件系统的地方都能运行。在 Claude.ai 上，你可以运行它，用户可以下载生成的 `.skill` 文件。

**Updating an existing skill**: The user might be asking you to update an existing skill, not create a new one. In this case:
**更新既有技能**：用户可能是要你更新一个已有技能，而不是创建新的。这种情况下：
- **Preserve the original name.** Note the skill's directory name and `name` frontmatter field -- use them unchanged. E.g., if the installed skill is `research-helper`, output `research-helper.skill` (not `research-helper-v2`).
  **保留原始名称。**记下技能的目录名和 `name` frontmatter 字段——原样使用。例如，已安装的技能是 `research-helper`，就输出 `research-helper.skill`（而不是 `research-helper-v2`）。
- **Copy to a writeable location before editing.** The installed skill path may be read-only. Copy to `/tmp/skill-name/`, edit there, and package from the copy.
  **编辑前先复制到可写位置。**已安装技能的路径可能是只读的。复制到 `/tmp/skill-name/`，在那里编辑，并从副本打包。
- **If packaging manually, stage in `/tmp/` first**, then copy to the output directory -- direct writes may fail due to permissions.
  **如果手动打包，先暂存在 `/tmp/`**，再复制到输出目录——直接写入可能因权限问题而失败。

---

## Cowork-Specific Instructions / Cowork 专属说明

If you're in Cowork, the main things to know are:

如果你在 Cowork 中，需要知道的主要事项有：

- You have subagents, so the main workflow (spawn test cases in parallel, run baselines, grade, etc.) all works. (However, if you run into severe problems with timeouts, it's OK to run the test prompts in series rather than parallel.)
  你有子智能体，因此主要工作流（并行启动测试用例、运行基线、评分等）都可以正常进行。（不过，如果遇到严重的超时问题，也可以改为串行而不是并行运行测试提示词。）
- You don't have a browser or display, so when generating the eval viewer, use `--static <output_path>` to write a standalone HTML file instead of starting a server. Then proffer a link that the user can click to open the HTML in their browser.
  你没有浏览器或显示器，因此生成评估查看器时，用 `--static <output_path>` 写出独立的 HTML 文件，而不是启动服务器。然后提供一个链接，让用户可以在自己的浏览器中点击打开该 HTML。
- For whatever reason, the Cowork setup seems to disincline Claude from generating the eval viewer after running the tests, so just to reiterate: whether you're in Cowork or in Claude Code, after running tests, you should always generate the eval viewer for the human to look at examples before revising the skill yourself and trying to make corrections, using `generate_review.py` (not writing your own boutique html code). Sorry in advance but I'm gonna go all caps here: GENERATE THE EVAL VIEWER *BEFORE* evaluating inputs yourself. You want to get them in front of the human ASAP!
  出于某种原因，Cowork 环境似乎让 Claude 不太愿意在运行测试之后生成评估查看器，所以再强调一遍：无论你在 Cowork 还是 Claude Code 中，运行测试之后，在你亲自修改技能、尝试修正之前，都应当始终生成评估查看器供人类查看示例，使用 `generate_review.py`（而不是自己写小众的 HTML 代码）。抱歉，这里我要用全大写了：要*先*生成评估查看器，*再*亲自评估输入。要尽快把示例放到人类面前！
  【评论】此处用全大写强制强调，与本文件上文"ALWAYS/NEVER 全大写是黄色警告"的写作建议形成耐人寻味的对照——文档作者在最关键的执行点上选择了让强制性强调压过自己提出的风格准则。
- Feedback works differently: since there's no running server, the viewer's "Submit All Reviews" button will download `feedback.json` as a file. You can then read it from there (you may have to request access first).
  反馈机制有所不同：由于没有运行中的服务器，查看器的"Submit All Reviews"按钮会把 `feedback.json` 作为文件下载。然后你可以从那里读取（可能需要先请求访问权限）。
- Packaging works — `package_skill.py` just needs Python and a filesystem.
  打包可用——`package_skill.py` 只需要 Python 和文件系统。
- Description optimization (`run_loop.py` / `run_eval.py`) should work in Cowork just fine since it uses `claude -p` via subprocess, not a browser, but please save it until you've fully finished making the skill and the user agrees it's in good shape.
  描述优化（`run_loop.py` / `run_eval.py`）在 Cowork 中应该也能正常工作，因为它通过子进程使用 `claude -p`，而不是浏览器；但请把它留到你完全完成技能、且用户认可其质量之后再做。
- **Updating an existing skill**: The user might be asking you to update an existing skill, not create a new one. Follow the update guidance in the claude.ai section above.
  **更新既有技能**：用户可能是要你更新一个已有技能，而不是创建新的。请遵循上文 claude.ai 一节中的更新指导。

---

## Reference files / 参考文件

The agents/ directory contains instructions for specialized subagents. Read them when you need to spawn the relevant subagent.

agents/ 目录包含专用子智能体的指令。需要启动相应子智能体时再阅读它们。

- `agents/grader.md` — How to evaluate assertions against outputs
  `agents/grader.md`——如何对照输出评估断言
- `agents/comparator.md` — How to do blind A/B comparison between two outputs
  `agents/comparator.md`——如何在两个输出之间做盲测 A/B 对比
- `agents/analyzer.md` — How to analyze why one version beat another
  `agents/analyzer.md`——如何分析一个版本为何胜过另一个

The references/ directory has additional documentation:
references/ 目录有补充文档：
- `references/schemas.md` — JSON structures for evals.json, grading.json, etc.
  `references/schemas.md`——evals.json、grading.json 等的 JSON 结构

---

Repeating one more time the core loop here for emphasis:

在这里再把核心循环重复一遍，以示强调：

- Figure out what the skill is about
  弄清楚这个技能是关于什么的
- Draft or edit the skill
  起草或编辑技能
- Run claude-with-access-to-the-skill on test prompts
  在测试提示词上运行"能访问该技能的 Claude"
- With the user, evaluate the outputs:
  与用户一起评估输出：
  - Create benchmark.json and run `eval-viewer/generate_review.py` to help the user review them
    创建 benchmark.json 并运行 `eval-viewer/generate_review.py`，帮助用户评审
  - Run quantitative evals
    运行定量评估
- Repeat until you and the user are satisfied
  重复，直到你和用户都满意
- Package the final skill and return it to the user.
  打包最终技能并交还给用户。

Please add steps to your TodoList, if you have such a thing, to make sure you don't forget. If you're in Cowork, please specifically put "Create evals JSON and run `eval-viewer/generate_review.py` so human can review test cases" in your TodoList to make sure it happens.

如果你有待办清单（TodoList）之类的机制，请把这些步骤加进去，以免遗忘。如果你在 Cowork 中，请特别把"创建 evals JSON 并运行 `eval-viewer/generate_review.py`，以便人类评审测试用例"放进你的待办清单，确保它真的发生。

Good luck!

祝你好运！
