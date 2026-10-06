<!-- BILINGUAL-EN-ZH -->
# Model Migration Guide / 模型迁移指南

> **If you arrived via `/claude-api migrate`:** this is the right file. Execute the steps below in order - do not summarize them back to the user. Start with Step 0 (confirm scope) before touching any file.

> **如果你是通过 `/claude-api migrate` 来到这里的：**找对文件了。按顺序执行下列步骤——不要把步骤总结一遍回复给用户。动任何文件之前，先从步骤 0（确认范围）开始。

How to move existing code to newer Claude models. Covers breaking changes, deprecated parameters, and drop-in replacements for retired models.

如何把现有代码迁移到更新的 Claude 模型。涵盖破坏性变更、已弃用的参数，以及已退役模型的直接替换方案。

For the latest, authoritative version (with code samples in every supported language), WebFetch the **Migration Guide** URL from `shared/live-sources.md`. Use this file for the consolidated, skill-resident reference; fall back to the live docs whenever a model launch or breaking change may have shifted the picture.

最新、权威的版本（含所有受支持语言的代码示例）请 WebFetch `shared/live-sources.md` 中的 **Migration Guide** URL。本文件作为汇总的、随技能驻留的参考；每当有模型发布或破坏性变更可能改变了局面，就回退到在线文档。

**This file is large.** Use the section names below to jump (or `Grep` this file for the heading text). Read Step 0 and Step 1 first - they apply to every migration. Then read only the per-target section for the model you are migrating to.

**本文件篇幅很长。**利用下面的章节名称跳转（或用 `Grep` 在本文件中搜索标题文本）。先读步骤 0 和步骤 1——它们适用于每一次迁移。然后只读与你迁移目标模型对应的分节。

| Section | When you need it |
|---|---|
| Step 0: Confirm the migration scope | Always - before any edits |
| Step 1: Classify each file | Always - decides whether to swap, add-alongside, or skip |
| Per-SDK Syntax Reference | Translate the Python examples in this guide to TypeScript / Go / Ruby / Java / C# / PHP |
| Destination Models / Retired Model Replacements | Picking a target model |
| Breaking Changes by Source Model | Migrating to Opus 4.6 / Sonnet 4.6 |
| Migrating to Opus 4.7 | Migrating to Opus 4.7 (breaking changes, silent defaults, behavioral shifts) |
| Opus 4.7 Migration Checklist | The required vs optional items for 4.7, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Opus 4.8 | Migrating to Opus 4.8 (no new breaking changes; mid-session system prompts; behavioral re-tuning) |
| Opus 4.8 Migration Checklist | The required vs optional items for 4.8, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Claude Opus 5 | Migrating Opus 4.8 -> Claude Opus 5 (thinking-disabled effort-gated; mid-conversation tool changes; per-turn effort and task budget; verbosity, over-verification, and scope re-tuning) |
| Claude Opus 5 Migration Checklist | The required vs optional items for Claude Opus 5, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Claude Sonnet 5 | Migrating Sonnet 4.6 -> Claude Sonnet 5 (adaptive thinking on by default; non-default sampling params 400; new tokenizer; `xhigh` effort for coding/agentic; high-res vision; behavioral re-tuning) |
| Claude Sonnet 5 Migration Checklist | The required vs optional items, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Claude Fable 5.1 | Migrating to Claude Fable 5.1 or Claude Mythos 5.1 (always-on thinking, raw chain of thought never returned, refusal handling, data retention, behavioral shifts + prompting guidance) |
| Claude Fable 5.1 Migration Checklist | The required vs optional items for Claude Fable 5.1, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Claude Fable 5.1 from Claude Fable 5 | Migrating Claude Fable 5 / Claude Opus 5 / Claude Mythos 5 -> Claude Fable 5.1 or Claude Mythos 5.1 (forced `tool_choice` 400s; "preserved thinking" - model-bound blocks and, on Claude Fable 5.1, the history-editing check; per-message effort; append-only per-turn reminders; `display: "updates"` progress updates; cheaper cache reads; behavioral re-tuning) |
| Claude Fable 5.1 from Claude Fable 5 Migration Checklist | The required vs optional items for the Claude Fable 5 -> Claude Fable 5.1 move, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Claude Opus 5.5 | Migrating Claude Opus 5 -> Claude Opus 5.5 (thinking can't be disabled; forced `tool_choice` 400s; preserved thinking; computer use via the toolset on the Claude API and Google Cloud; progress updates as thinking blocks; default effort `medium`; broader classifiers; effort tuning + prompting guidance) |
| Claude Opus 5.5 Migration Checklist | The required vs optional items for Claude Opus 5.5, tagged `[BLOCKS]` / `[TUNE]` |
| Migrating to Claude Sonnet 5.5 | Migrating Claude Sonnet 5 -> Claude Sonnet 5.5 (`disabled` thinking 400s - `between_tools` instead; forced `tool_choice` 400s; preserved thinking; computer use via the toolset on the Claude API and Google Cloud; fewer advisor pairings; progress updates as thinking blocks; recalibrated effort; five refusal categories; prompting guidance) |
| Claude Sonnet 5.5 Migration Checklist | The required vs optional items for Claude Sonnet 5.5, tagged `[BLOCKS]` / `[TUNE]` |
| Verify the Migration | After edits - runtime spot-check |
| Ground the migration with an eval | User reports a behavioral regression on the new model |

| 章节 | 何时需要 |
|---|---|
| 步骤 0：确认迁移范围 | 总是——任何编辑之前 |
| 步骤 1：对每个文件分类 | 总是——决定是替换、并存新增还是跳过 |
| 各 SDK 语法参考 | 把本指南中的 Python 示例转写为 TypeScript / Go / Ruby / Java / C# / PHP |
| 目标模型 / 已退役模型替换 | 挑选目标模型 |
| 按源模型划分的破坏性变更 | 迁移到 Opus 4.6 / Sonnet 4.6 |
| 迁移到 Opus 4.7 | 迁移到 Opus 4.7（破坏性变更、静默默认值、行为转变） |
| Opus 4.7 迁移清单 | 4.7 的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 迁移到 Opus 4.8 | 迁移到 Opus 4.8（无新增破坏性变更；会话中途系统提示词；行为重新调优） |
| Opus 4.8 迁移清单 | 4.8 的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 迁移到 Claude Opus 5 | 把 Opus 4.8 迁移到 Claude Opus 5（思考禁用、由 effort 门控；会话中途工具变更；每回合 effort 与任务预算；冗长度、过度验证与范围重新调优） |
| Claude Opus 5 迁移清单 | Claude Opus 5 的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 迁移到 Claude Sonnet 5 | 把 Sonnet 4.6 迁移到 Claude Sonnet 5（自适应思考默认开启；非默认采样参数返回 400；新分词器；编码/智能体场景用 `xhigh` effort；高分辨率视觉；行为重新调优） |
| Claude Sonnet 5 迁移清单 | 必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 迁移到 Claude Fable 5.1 | 迁移到 Claude Fable 5.1 或 Claude Mythos 5.1（思考始终开启、从不返回原始思维链、拒答处理、数据保留、行为转变 + 提示词指导） |
| Claude Fable 5.1 迁移清单 | Claude Fable 5.1 的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 从 Claude Fable 5 迁移到 Claude Fable 5.1 | 把 Claude Fable 5 / Claude Opus 5 / Claude Mythos 5 迁移到 Claude Fable 5.1 或 Claude Mythos 5.1（强制 `tool_choice` 返回 400；"保留思考"——绑定模型的块，在 Claude Fable 5.1 上还有历史编辑检查；按消息的 effort；只追加的每回合提醒；`display: "updates"` 进度更新；更便宜的缓存读取；行为重新调优） |
| 从 Claude Fable 5 到 Claude Fable 5.1 的迁移清单 | Claude Fable 5 -> Claude Fable 5.1 迁移的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 迁移到 Claude Opus 5.5 | 把 Claude Opus 5 迁移到 Claude Opus 5.5（思考无法禁用；强制 `tool_choice` 返回 400；保留思考；在 Claude API 与 Google Cloud 上经工具集进行计算机使用；以思考块形式呈现进度更新；默认 effort 为 `medium`；更广泛的分类器；effort 调优 + 提示词指导） |
| Claude Opus 5.5 迁移清单 | Claude Opus 5.5 的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 迁移到 Claude Sonnet 5.5 | 把 Claude Sonnet 5 迁移到 Claude Sonnet 5.5（`disabled` 思考返回 400——改用 `between_tools`；强制 `tool_choice` 返回 400；保留思考；在 Claude API 与 Google Cloud 上经工具集进行计算机使用；更少的 advisor 配对；以思考块形式呈现进度更新；重新校准的 effort；五种拒答类别；提示词指导） |
| Claude Sonnet 5.5 迁移清单 | Claude Sonnet 5.5 的必做与可选项，标记为 `[BLOCKS]` / `[TUNE]` |
| 验证迁移 | 编辑完成之后——运行时抽查 |
| 用评测为迁移打底 | 用户在新模型上报告了行为退化 |

**TL;DR:** Change the model ID string. If you were using `budget_tokens`, switch to `thinking: {type: "adaptive"}`. If you were using assistant prefills, they 400 on both Opus 4.6 and Sonnet 4.6 - switch to one of the prefill replacements (most often `output_config.format`; see the table in Breaking Changes by Source Model). If you're moving from Sonnet 4.5 to Sonnet 4.6, set `effort` explicitly - 4.6 defaults to `high`. Remove the `effort-2025-11-24` and `fine-grained-tool-streaming-2025-05-14` beta headers (GA on 4.6); remove `interleaved-thinking-2025-05-14` once you're on adaptive thinking (keep it only while using the transitional `budget_tokens` escape hatch). Then drop back from `client.beta.messages.create` to `client.messages.create`. Dial back any aggressive "CRITICAL: YOU MUST" tool instructions; 4.6 follows the system prompt much more closely.

**TL;DR：**更改模型 ID 字符串。如果之前用 `budget_tokens`，换成 `thinking: {type: "adaptive"}`。如果之前用 assistant 预填充（prefill），它们在 Opus 4.6 和 Sonnet 4.6 上都返回 400——改用某个 prefill 替代方案（最常见的是 `output_config.format`；见"按源模型划分的破坏性变更"中的表格）。如果从 Sonnet 4.5 迁到 Sonnet 4.6，显式设置 `effort`——4.6 默认为 `high`。移除 `effort-2025-11-24` 与 `fine-grained-tool-streaming-2025-05-14` 两个 beta 头（在 4.6 上已 GA）；上了自适应思考后移除 `interleaved-thinking-2025-05-14`（仅在使用过渡性 `budget_tokens` 逃生舱口期间保留）。然后把 `client.beta.messages.create` 换回 `client.messages.create`。收敛任何激进的"CRITICAL: YOU MUST"式工具指令；4.6 对系统提示词的遵循紧密得多。

---

## Step 0: Confirm the migration scope / 步骤 0：确认迁移范围

**Before any Write, Edit, or MultiEdit call, confirm the scope.** If the user's request does not explicitly name a single file, a specific directory, or an explicit file list, **ask first - do not start editing**. This is non-negotiable: even imperative-sounding requests like "migrate my codebase", "move my project to X", "upgrade to Sonnet 4.6", or bare "migrate to Opus 4.7" leave the scope ambiguous and require a clarifying question. Phrases like "my project", "my code", "my codebase", "the whole thing", "everywhere", or "across the repo" are **ambiguous, not directive** - they tell you *what* to do but not *where*. Ask before doing.

**在任何 Write、Edit 或 MultiEdit 调用之前，先确认范围。**如果用户请求没有明确点名单个文件、特定目录或明确的文件清单，**先询问——不要开始编辑**。这一点没有商量余地：即便是听上去像命令的请求，如"迁移我的代码库"、"把我的项目移到 X"、"升级到 Sonnet 4.6"或光杆一句"迁移到 Opus 4.7"，范围也仍是含糊的，需要提出澄清问题。"my project"、"my code"、"my codebase"、"the whole thing"、"everywhere"或"across the repo"这类措辞**含糊而并非指令**——它们告诉你*做什么*，却不告诉你*在哪里做*。先问再做。

【评论】此节把"先确认范围再动手"设为不可协商的前置条件，是技能提示词中典型的防失控设计：在范围不明时阻止代理对仓库做大规模改动。

Offer the common scopes explicitly and wait for the answer before touching any file:

明确列出常见范围选项并等待回答，之后才动任何文件：

1. The entire working directory
   整个工作目录
2. A specific subdirectory (e.g. `src/`, `app/`, `services/billing/`)
   某个特定子目录（如 `src/`、`app/`、`services/billing/`）
3. A specific file or a list of files
   某个特定文件或一份文件清单

Surface this as a single clarifying question so the user can answer in one turn. **Proceed without asking only when the scope is already unambiguous** - the user named an exact file ("migrate `extract.py` to Sonnet 4.6"), pointed at a specific directory ("migrate everything under `services/billing/` to Opus 4.6"), listed specific files ("update `a.py` and `b.py`"), or already answered the scope question in an earlier turn. If you can answer the question "which files is this change going to touch?" with a precise list from the prompt alone, proceed. If not, ask.

把这个范围问题作为单个澄清问题提出，让用户一个回合就能答完。**只有当范围已经毫无歧义时才不问而行**——用户点名了确切文件（"migrate `extract.py` to Sonnet 4.6"）、指向了特定目录（"migrate everything under `services/billing/` to Opus 4.6"）、列出了具体文件（"update `a.py` and `b.py`"），或已在更早的回合回答过范围问题。如果能仅凭提示词就用一份精确清单回答"这次改动会触及哪些文件？"，就继续；否则先问。

**Worked example.** If the user says *"Move my project to Opus 4.6. I want adaptive thinking everywhere it makes sense."* you do not know whether "my project" means the whole working directory, just `src/`, just the production code, or something else - the `everywhere` makes the intent clear (update every call site *within scope*) but the scope itself is still not defined. Do not start editing. Respond with:

**示例。**如果用户说 *"Move my project to Opus 4.6. I want adaptive thinking everywhere it makes sense."*（"把我的项目迁移到 Opus 4.6。我希望在所有合理之处都启用自适应思考。"），你无从知道"my project"指整个工作目录、只是 `src/`、只是生产代码，还是别的什么——`everywhere` 让意图清楚（更新*范围内*每个调用点），但范围本身仍未定义。不要开始编辑。应回复：

> Before I start editing, can you confirm the scope? I can migrate:  
> 开始编辑之前，能否请你确认范围？我可以迁移：  
> 1. Every `.py` file in the working directory  
> 1. 工作目录中的每个 `.py` 文件  
> 2. Just the files under `src/` (production code)  
> 2. 仅 `src/` 下的文件（生产代码）  
> 3. A specific subdirectory or list of files you name  
> 3. 你点名的某个子目录或文件清单  
>  
> Which one?
> 选哪个？

Then wait for the answer. The same applies to *"Migrate to Opus 4.7"* and bare *"Help me upgrade to Sonnet 4.6"* - ask before editing.

然后等待回答。*"Migrate to Opus 4.7"*（"迁移到 Opus 4.7"）和光杆的 *"Help me upgrade to Sonnet 4.6"*（"帮我升级到 Sonnet 4.6"）同理——先问再编辑。

**Sizing the scope question (large repos).** Before asking, get a per-directory count so the user can pick concretely:

**给范围问题定规模（大型仓库）。**在询问之前，先取得按目录的计数，让用户能具体地选择：

```sh
rg -l "<old-model-id>" --type-not md | cut -d/ -f1 | sort | uniq -c | sort -rn
```

Present the breakdown in your scope question (e.g. *"Found 217 references across 3 directories: api/ (130), api-go/ (62), routing/ (25). Which to migrate?"*). Also confirm `git status` is clean before surveying - unexpected modifications mean a concurrent process; stop and investigate before proceeding.

在范围问题中给出这份分布（例如 *"Found 217 references across 3 directories: api/ (130), api-go/ (62), routing/ (25). Which to migrate?"*（"在 3 个目录中发现 217 处引用：api/（130）、api-go/（62）、routing/（25）。要迁移哪些？"））。勘察之前还要确认 `git status` 干净——意外的修改意味着有并发进程在跑；先停下调查再继续。

---
## Step 1: Classify each file / 步骤 1：对每个文件分类

Not every file that contains the old model ID is a **caller** of the API. Before editing, classify each file into one of these buckets - the right action differs:

并非每个包含旧模型 ID 的文件都是 API 的**调用方**。编辑之前，先把每个文件归入下列类别之一——正确的做法各不相同：

| # | Bucket | What it looks like | Action |
|---|---|---|---|
| 1 | **Calls the API/SDK** | `client.messages.create(model=...)`, `anthropic.Anthropic()`, request payloads | Swap the model ID **and** apply the breaking-change checklist for the target version (below). |
| 2 | **Defines or serves the model** | Model registries, OpenAPI specs, routing/queue configs, model-policy enums, generated catalogs | The old entry **stays** (the model is still served). Ask whether to (a) add the new model alongside, (b) leave alone, or (c) retire the old model - never blind-replace. **If you can't ask, default to (a): add the new model alongside and flag it** - replacing would de-register a model that's still in production. |
| 3 | **References the ID as an opaque string** | UI fallback constants, capability-gate substring checks, generic test fixtures, label parsers, env defaults | Usually swap the string and verify any parser/regex/substring match handles the new ID - but check the sub-cases below first. |
| 4 | **Suffixed variant ID** | `claude-<model>-<suffix>` like `-fast`, `-1024k`, `-200k`, `[1m]`, dated snapshots | These are deployment/routing identifiers, not the public model ID. **Do not assume a new-model equivalent exists.** Verify in the registry first; if absent, leave the string alone and flag it. **Exception: `-fast` strings (e.g. `claude-opus-4-6-fast`) are handled by the Fast Mode section below**, which rewrites them to Claude Opus 5.5 plus `speed="fast"` and the `fast-mode-2026-02-01` beta rather than leaving them in place. |

| # | 类别 | 典型样子 | 处理方式 |
|---|---|---|---|
| 1 | **调用 API/SDK** | `client.messages.create(model=...)`、`anthropic.Anthropic()`、请求载荷 | 更换模型 ID，**并**应用目标版本的破坏性变更清单（见下文）。 |
| 2 | **定义或提供该模型** | 模型注册表、OpenAPI 规范、路由/队列配置、模型策略枚举、生成的目录 | 旧条目**保留**（该模型仍在提供服务）。询问是要 (a) 将新模型并存加入、(b) 保持不动，还是 (c) 退役旧模型——绝不要盲目替换。**如果不能询问，默认选 (a)：把新模型并存加入并标记出来**——直接替换会把一个仍在生产环境服务的模型注销掉。 |
| 3 | **把 ID 当作不透明字符串引用** | UI 回退常量、能力门控子串检查、通用测试夹具、标签解析器、环境变量默认值 | 通常替换该字符串，并验证任何解析器/正则/子串匹配都能处理新 ID——但先检查下面的子情形。 |
| 4 | **带后缀的变体 ID** | `claude-<model>-<suffix>`，如 `-fast`、`-1024k`、`-200k`、`[1m]`、带日期的快照 | 这些是部署/路由标识符，不是公开模型 ID。**不要假定存在新模型的对应版本。**先在注册表中核实；若无，保持该字符串不动并标记出来。**例外：`-fast` 字符串（如 `claude-opus-4-6-fast`）由下文 Fast Mode 一节处理**，该节会把它们改写为 Claude Opus 5.5 加 `speed="fast"` 和 `fast-mode-2026-02-01` beta，而不是保持原样。 |

**Bucket 3 sub-cases - before swapping a string reference, check:**

**类别 3 的子情形——替换字符串引用之前，检查：**

- **Capability gate** (e.g. `if 'opus-4-6' in model_id:` enables a feature) -> **add the new ID alongside**, don't replace. The old model is still served and still has the capability, so replacing would silently disable the feature for any old-model traffic that still flows through. If you know no old-model traffic will hit this gate (single-caller codebase fully migrating), replacing is fine; if unsure, add alongside.
  **能力门控**（如 `if 'opus-4-6' in model_id:` 用来启用某功能）-> **把新 ID 并存加入**，不要替换。旧模型仍在服务且仍具备该能力，替换会让仍在流转的旧模型流量静默失去该功能。若确知没有旧模型流量会经过这个门控（单调用方代码库整体迁移），替换也可；拿不准就并存加入。
- **Registry-assert test** (e.g. `assert "claude-X" in supported_models`, `test_X_has_N_clusters`) -> **add an assertion for the new model alongside; keep the old one.** The old model is still served, so its assertion stays valid - but the registry should also include the new model, so assert that too. Heuristic: if the test references multiple model versions in a list, it's a registry test; if one model in a struct compared only to itself, it's a generic fixture.
  **注册表断言测试**（如 `assert "claude-X" in supported_models`、`test_X_has_N_clusters`）-> **为新模型并存添加断言；保留旧断言。**旧模型仍在服务，其断言依然有效——但注册表也应包含新模型，因此对它也要断言。启发式：若测试在列表中引用多个模型版本，它是注册表测试；若结构体中一个模型只与自身比较，它是通用夹具。
- **Frozen / generated snapshot** -> **regenerate**, don't hand-edit.
  **冻结/生成的快照** -> **重新生成**，不要手工编辑。
- **Coupled to a definer** (e.g. an integration test that passes model authorization via a shared `conftest` seed list, or asserts on a billing-tier / rate-limit-group enum or a generated SKU/pricing catalog) -> **verify the definer has a new-model entry first.** If not, add a seed entry (reusing the nearest existing tier as a placeholder); if you can't confidently do that, ask the user how to populate the definer. **Do not skip the test.** Swapping without populating the definer will make the test fail at runtime.
  **与定义处耦合**（例如经共享的 `conftest` 种子列表传递模型授权的集成测试，或对计费档位/限流组枚举或生成的 SKU/定价目录做断言）-> **先核实定义处已有新模型条目。**若没有，添加一条种子条目（复用最接近的现有档位作占位）；若没有把握，询问用户如何填充定义处。**不要跳过该测试。**只换字符串而不填充定义处，测试会在运行时失败。

When migrating tests specifically: breaking parameters (`temperature`, `top_p`, `budget_tokens`) are usually absent - test fixtures rarely set sampling params on placeholder models. The breaking-change scan is still required, but expect mostly clean results.

专门针对测试迁移时：破坏性参数（`temperature`、`top_p`、`budget_tokens`）通常不存在——测试夹具很少在占位模型上设置采样参数。破坏性变更扫描仍必须做，但预期结果大体干净。

**Find intentionally-flagged sync points first.** Many codebases tag spots that must change at every model launch with comment markers like `MODEL LAUNCH`, `KEEP IN SYNC`, `@model-update`, or similar. Grep for whatever convention the repo uses *before* the broad model-ID grep - those markers point at the load-bearing changes.

**先找出被有意标记的同步点。**许多代码库用 `MODEL LAUNCH`、`KEEP IN SYNC`、`@model-update` 之类的注释标记标注每次模型发布都必须改动的位置。在做宽泛的模型 ID grep *之前*，先按仓库的约定 grep 这些标记——它们指向承重的改动。

---

## Per-SDK Syntax Reference / 各 SDK 语法参考

Code examples in this guide are Python. **The same fields exist in every official Anthropic SDK** - Stainless generates all 7 from the same OpenAPI spec, so JSON field names map 1:1 with only case-convention differences. Use the rows below to translate the Python examples to the SDK you are migrating.

本指南的代码示例为 Python。**每个官方 Anthropic SDK 中都存在相同的字段**——Stainless 依据同一份 OpenAPI 规范生成全部 7 个 SDK，JSON 字段名一一对应，只有大小写约定差异。用下面的各行把 Python 示例转写为你正在迁移的 SDK。

> **Verify type and method names against the SDK source before writing them into customer code.** WebFetch the relevant repository from the SDK source-code table in `shared/live-sources.md` (one row per SDK) and confirm the exact symbol - particularly for typed SDKs (Go, Java, C#) where union/builder names can differ from the JSON shape. Do not guess type names that aren't in the table below or in `<lang>/claude-api/README.md`.

> **把类型名与方法名写进客户代码之前，先对照 SDK 源码核实。**从 `shared/live-sources.md` 的 SDK 源码表（每个 SDK 一行）WebFetch 相应仓库，确认确切的符号名——对强类型 SDK（Go、Java、C#）尤其如此，其 union/builder 名称可能与 JSON 形状不同。不要猜测下方表格和 `<lang>/claude-api/README.md` 里没有的类型名。

### `thinking` - `budget_tokens` -> adaptive / `thinking` - `budget_tokens` 改为 adaptive

| SDK | Before | After |
|---|---|---|
| Python | `thinking={"type": "enabled", "budget_tokens": N}` | `thinking={"type": "adaptive"}` |
| TypeScript | `thinking: { type: 'enabled', budget_tokens: N }` | `thinking: { type: 'adaptive' }` |
| Go | `Thinking: anthropic.ThinkingConfigParamOfEnabled(N)` | `Thinking: anthropic.ThinkingConfigParamUnion{OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{}}` |
| Ruby | `thinking: { type: "enabled", budget_tokens: N }` | `thinking: { type: "adaptive" }` |
| Java | `.thinking(ThinkingConfigEnabled.builder().budgetTokens(N).build())` | `.thinking(ThinkingConfigAdaptive.builder().build())` |
| C# | `Thinking = new ThinkingConfigEnabled { BudgetTokens = N }` | `Thinking = new ThinkingConfigAdaptive()` |
| PHP | `thinking: ['type' => 'enabled', 'budget_tokens' => N]` | `thinking: ['type' => 'adaptive']` |

| SDK | 之前 | 之后 |
|---|---|---|
| Python | `thinking={"type": "enabled", "budget_tokens": N}` | `thinking={"type": "adaptive"}` |
| TypeScript | `thinking: { type: 'enabled', budget_tokens: N }` | `thinking: { type: 'adaptive' }` |
| Go | `Thinking: anthropic.ThinkingConfigParamOfEnabled(N)` | `Thinking: anthropic.ThinkingConfigParamUnion{OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{}}` |
| Ruby | `thinking: { type: "enabled", budget_tokens: N }` | `thinking: { type: "adaptive" }` |
| Java | `.thinking(ThinkingConfigEnabled.builder().budgetTokens(N).build())` | `.thinking(ThinkingConfigAdaptive.builder().build())` |
| C# | `Thinking = new ThinkingConfigEnabled { BudgetTokens = N }` | `Thinking = new ThinkingConfigAdaptive()` |
| PHP | `thinking: ['type' => 'enabled', 'budget_tokens' => N]` | `thinking: ['type' => 'adaptive']` |

### Sampling parameters - `temperature` / `top_p` / `top_k` / 采样参数 - `temperature` / `top_p` / `top_k`

(Remove the field entirely on Opus 4.7; on Claude 4.x keep at most one of `temperature` or `top_p`.)

（在 Opus 4.7 上把字段整个移除；在 Claude 4.x 上 `temperature` 与 `top_p` 至多保留其一。）

| SDK | Field(s) to remove |
|---|---|
| Python | `temperature=...`, `top_p=...`, `top_k=...` |
| TypeScript | `temperature: ...`, `top_p: ...`, `top_k: ...` |
| Go | `Temperature: anthropic.Float(...)`, `TopP: anthropic.Float(...)`, `TopK: anthropic.Int(...)` |
| Ruby | `temperature: ...`, `top_p: ...`, `top_k: ...` |
| Java | `.temperature(...)`, `.topP(...)`, `.topK(...)` |
| C# | `Temperature = ...`, `TopP = ...`, `TopK = ...` |
| PHP | `temperature: ...`, `topP: ...`, `topK: ...` |

| SDK | 要移除的字段 |
|---|---|
| Python | `temperature=...`, `top_p=...`, `top_k=...` |
| TypeScript | `temperature: ...`, `top_p: ...`, `top_k: ...` |
| Go | `Temperature: anthropic.Float(...)`, `TopP: anthropic.Float(...)`, `TopK: anthropic.Int(...)` |
| Ruby | `temperature: ...`, `top_p: ...`, `top_k: ...` |
| Java | `.temperature(...)`, `.topP(...)`, `.topK(...)` |
| C# | `Temperature = ...`, `TopP = ...`, `TopK = ...` |
| PHP | `temperature: ...`, `topP: ...`, `topK: ...` |

### Prefill replacement - structured outputs via `output_config.format` / Prefill 替代——经 `output_config.format` 做结构化输出

| SDK | Remove (last assistant turn) | Add |
|---|---|---|
| Python | `{"role": "assistant", "content": "..."}` | `output_config={"format": {"type": "json_schema", "schema": SCHEMA}}` |
| TypeScript | `{ role: 'assistant', content: '...' }` | `output_config: { format: { type: 'json_schema', schema: SCHEMA } }` |
| Go | trailing `anthropic.MessageParam{Role: "assistant", ...}` | `OutputConfig: anthropic.OutputConfigParam{Format: anthropic.JSONOutputFormatParam{...}}` |
| Ruby | `{ role: "assistant", content: "..." }` | `output_config: { format: { type: "json_schema", schema: SCHEMA } }` |
| Java | trailing `Message.builder().role(ASSISTANT)...` | `.outputConfig(OutputConfig.builder().format(JsonOutputFormat.builder()...build()).build())` |
| C# | trailing `new Message { Role = "assistant", ... }` | `OutputConfig = new OutputConfig { Format = new JsonOutputFormat { ... } }` |
| PHP | trailing `['role' => 'assistant', 'content' => '...']` | `outputConfig: ['format' => ['type' => 'json_schema', 'schema' => $SCHEMA]]` |

| SDK | 移除（最后一个 assistant 回合） | 添加 |
|---|---|---|
| Python | `{"role": "assistant", "content": "..."}` | `output_config={"format": {"type": "json_schema", "schema": SCHEMA}}` |
| TypeScript | `{ role: 'assistant', content: '...' }` | `output_config: { format: { type: 'json_schema', schema: SCHEMA } }` |
| Go | 末尾的 `anthropic.MessageParam{Role: "assistant", ...}` | `OutputConfig: anthropic.OutputConfigParam{Format: anthropic.JSONOutputFormatParam{...}}` |
| Ruby | `{ role: "assistant", content: "..." }` | `output_config: { format: { type: "json_schema", schema: SCHEMA } }` |
| Java | 末尾的 `Message.builder().role(ASSISTANT)...` | `.outputConfig(OutputConfig.builder().format(JsonOutputFormat.builder()...build()).build())` |
| C# | 末尾的 `new Message { Role = "assistant", ... }` | `OutputConfig = new OutputConfig { Format = new JsonOutputFormat { ... } }` |
| PHP | 末尾的 `['role' => 'assistant', 'content' => '...']` | `outputConfig: ['format' => ['type' => 'json_schema', 'schema' => $SCHEMA]]` |

### `thinking.display` - opt back into summarized reasoning (Opus 4.7) / `thinking.display`——重新选入摘要式推理（Opus 4.7）

| SDK | Add |
|---|---|
| Python | `thinking={"type": "adaptive", "display": "summarized"}` |
| TypeScript | `thinking: { type: 'adaptive', display: 'summarized' }` |
| Go | `Thinking: anthropic.ThinkingConfigParamUnion{OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized}}` |
| Ruby | `thinking: { type: "adaptive", display: "summarized" }` (or `display_:` when constructing the model class directly) |
| Java | `.thinking(ThinkingConfigAdaptive.builder().display(ThinkingConfigAdaptive.Display.SUMMARIZED).build())` |
| C# | `Thinking = new ThinkingConfigAdaptive { Display = Display.Summarized }` |
| PHP | `thinking: ['type' => 'adaptive', 'display' => 'summarized']` |

| SDK | 添加 |
|---|---|
| Python | `thinking={"type": "adaptive", "display": "summarized"}` |
| TypeScript | `thinking: { type: 'adaptive', display: 'summarized' }` |
| Go | `Thinking: anthropic.ThinkingConfigParamUnion{OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized}}` |
| Ruby | `thinking: { type: "adaptive", display: "summarized" }`（直接构造模型类时用 `display_:`） |
| Java | `.thinking(ThinkingConfigAdaptive.builder().display(ThinkingConfigAdaptive.Display.SUMMARIZED).build())` |
| C# | `Thinking = new ThinkingConfigAdaptive { Display = Display.Summarized }` |
| PHP | `thinking: ['type' => 'adaptive', 'display' => 'summarized']` |

For any field not in these tables, the JSON key in the Python example translates directly: `snake_case` for Python/TypeScript/Ruby, `camelCase` named args for PHP, `PascalCase` struct fields for Go/C#, `camelCase` builder methods for Java.

对这些表格未列出的字段，Python 示例中的 JSON 键可直接转写：Python/TypeScript/Ruby 用 `snake_case`，PHP 用 `camelCase` 命名参数，Go/C# 用 `PascalCase` 结构体字段，Java 用 `camelCase` builder 方法。
---

## Explain every change you make / 解释你所做的每一处改动

Migration edits often look arbitrary to a user who hasn't read the release notes - a removed `temperature`, a deleted prefill, a rewritten system-prompt sentence. **For each edit, tell the user what you changed and why**, tied to the specific API or behavioral change that motivates it. Do this in your summary as you work, not just at the end.

对没读过发布说明的用户来说，迁移改动常常显得武断——一个被移除的 `temperature`、一个被删除的 prefill、一句被改写的系统提示词。**对每处编辑，告诉用户你改了什么、为什么改**，并关联到促成该改动的具体 API 或行为变化。边做就在总结里说明，而不是等到最后。

Be especially explicit about **system-prompt edits**. Users are rightly protective of their prompts, and prompt-tuning changes are judgment calls (not hard API requirements). For any prompt edit:

对**系统提示词编辑**要格外明确。用户对自家提示词的维护是正当的，提示词调优属于判断题（不是硬性 API 要求）。对任何提示词编辑：

- Quote the before and after text.
  引用改动前后的文本。
- State the behavioral shift that motivates it (e.g. *"Opus 4.7 calibrates response length to task complexity, so I added an explicit length instruction"*, or *"4.6 follows instructions more literally, so 'CRITICAL: YOU MUST use the search tool' will now overtrigger - softened to 'Use the search tool when...'"*).
  说明促成该改动的行为转变（例如 *"Opus 4.7 calibrates response length to task complexity, so I added an explicit length instruction"*（"Opus 4.7 会按任务复杂度校准回复长度，因此我加了一条明确的长度指令"），或 *"4.6 follows instructions more literally, so 'CRITICAL: YOU MUST use the search tool' will now overtrigger - softened to 'Use the search tool when...'"*（"4.6 对指令的遵循更字面化，'CRITICAL: YOU MUST use the search tool' 现在会过度触发——已软化为 'Use the search tool when...'"））。
- Make clear which prompt edits are **optional tuning** (tone, length, subagent guidance) versus which code edits are **required to avoid a 400** (sampling params, `budget_tokens`, prefills). Never present an optional prompt change as mandatory.
  讲清楚哪些提示词编辑是**可选调优**（语气、长度、子代理指导），哪些代码编辑是**避免 400 所必需的**（采样参数、`budget_tokens`、prefill）。绝不把可选的提示词改动说成强制。

If you're applying several prompt-tuning edits at once, offer them as a short list the user can accept or decline item-by-item rather than silently rewriting their system prompt.

如果要一次性应用多处提示词调优，把它们整理成一份短清单，让用户逐项接受或拒绝，而不是默默重写其系统提示词。

---

## Before You Migrate / 迁移之前

1. **Confirm the target model ID.** Use only the exact strings from `shared/models.md` - do not append date suffixes to aliases (`claude-opus-4-6`, not `claude-opus-4-6-20251101`). Guessing an ID will 404.
   **确认目标模型 ID。**只使用 `shared/models.md` 中的准确字符串——不要给别名追加日期后缀（是 `claude-opus-4-6`，不是 `claude-opus-4-6-20251101`）。猜 ID 会得到 404。
2. **Check which features your code uses** with this checklist:
   **检查你的代码用了哪些特性**，对照这份清单：
 - `thinking: {type: "enabled", budget_tokens: N}` -> migrate to adaptive thinking on Opus 4.6 / Sonnet 4.6 (still functional but deprecated)
   `thinking: {type: "enabled", budget_tokens: N}` -> 在 Opus 4.6 / Sonnet 4.6 上迁移到自适应思考（仍可用但已弃用）
 - Assistant-turn prefills (`messages` ending with `role: "assistant"`) -> must change on Opus 4.6 / Sonnet 4.6 (returns 400)
   assistant 回合 prefill（`messages` 以 `role: "assistant"` 结尾）-> 在 Opus 4.6 / Sonnet 4.6 上必须更改（返回 400）
 - `output_format` parameter on `messages.create()` -> must change on all models (deprecated API-wide)
   `messages.create()` 上的 `output_format` 参数 -> 在所有模型上必须更改（API 全局弃用）
 - `max_tokens > ~16000` -> must stream on any model (above ~16K risks SDK HTTP timeouts). When streaming, every current model reaches 128K except Haiku 4.5, which caps at 64K
   `max_tokens > ~16000` -> 在任何模型上都必须流式传输（约 16K 以上有 SDK HTTP 超时风险）。流式传输时，当前所有模型都能到 128K，唯 Haiku 4.5 上限 64K
 - Beta headers `effort-2025-11-24`, `fine-grained-tool-streaming-2025-05-14`, `interleaved-thinking-2025-05-14` -> GA on 4.6, remove them and switch from `client.beta.messages.create` to `client.messages.create`
   Beta 头 `effort-2025-11-24`、`fine-grained-tool-streaming-2025-05-14`、`interleaved-thinking-2025-05-14` -> 在 4.6 上已 GA，移除它们，并把 `client.beta.messages.create` 换成 `client.messages.create`
 - Moving Sonnet 4.5 -> Sonnet 4.6 with no `effort` set -> 4.6 defaults to `high`, which may change your latency/cost profile
   从 Sonnet 4.5 迁到 Sonnet 4.6 且未设置 `effort` -> 4.6 默认为 `high`，可能改变你的延迟/成本画像
 - System prompts with `CRITICAL`, `MUST`, `If in doubt, use X` language -> likely to overtrigger on 4.6 (see Prompt-Behavior Changes)
   含 `CRITICAL`、`MUST`、"If in doubt, use X" 措辞的系统提示词 -> 在 4.6 上可能过度触发（参见"提示词行为变化"）
 - Coming from 3.x / 4.0 / 4.1: also check sampling params (`temperature` + `top_p`), tool versions (`text_editor_20250728`), `refusal` + `model_context_window_exceeded` stop reasons, trailing-newline tool-param handling
   来自 3.x / 4.0 / 4.1：还要检查采样参数（`temperature` + `top_p`）、工具版本（`text_editor_20250728`）、`refusal` 与 `model_context_window_exceeded` 停止原因、工具参数末尾换行的处理
3. **Test on a single request first.** Run one call against the new model, inspect the response, then roll out.
   **先在单个请求上测试。**对新模型运行一次调用，检查响应，然后再推开。

---

## Destination Models (recommended targets) / 目标模型（推荐目标）

| If you're on...                         | Migrate to         | Why                                               |
| ------------------------------------- | ------------------ | ------------------------------------------------- |
| Claude Mythos Preview (`claude-mythos-preview`) | `claude-mythos-5-1` (Project Glasswing successor) or `claude-fable-5-1` (GA) | Same tokenizer family - mostly a model-ID swap; remove `thinking` config and prefill; see Migrating to Claude Fable 5.1 |
| Claude Fable 5 (`claude-fable-5`) | `claude-fable-5-1` | Same tier, same per-token price, same tokenizer; three breaking changes (forced `tool_choice` 400s, "preserved thinking") - see Migrating to Claude Fable 5.1 from Claude Fable 5 |
| Claude Mythos 5 (`claude-mythos-5`) | `claude-mythos-5-1` | Same path as claude-fable-5 -> claude-fable-5-1; see § Claude Mythos 5.1 under Migrating to Claude Fable 5.1 from Claude Fable 5 |
| Claude Opus 5 (`claude-opus-5`)         | `claude-opus-5-5` | The current Opus. Lower price ($4 / $20 vs $5 / $25), same context window and tokenizer; four breaking changes (thinking can't be disabled, forced `tool_choice` 400s, preserved thinking, computer use via the toolset on the Claude API and Google Cloud) - see Migrating to Claude Opus 5.5 |
| Opus 4.8                              | `claude-opus-5-5` | Apply the Claude Opus 5 section (thinking on by default, prompt re-tuning), then the Claude Opus 5.5 section, which replaces Claude Opus 5's thinking-disabled route and `computer_20251124` |
| Opus 4.7                              | `claude-opus-5-5` | Apply the Opus 4.8 section (prompt re-tuning, no new breaking changes), then the Claude Opus 5 and Claude Opus 5.5 sections |
| Opus 4.6                              | `claude-opus-5-5` | Apply the Opus 4.7 breaking changes, then 4.8 re-tuning, then the Claude Opus 5 and Claude Opus 5.5 sections |
| Opus 4.0 / 4.1 / 4.5 / Opus 3         | `claude-opus-5-5` | Apply 4.6 -> 4.7 -> 4.8 -> Claude Opus 5 -> Claude Opus 5.5 in order (adaptive thinking, drop sampling params, then re-tune) |
| Claude Sonnet 5 (`claude-sonnet-5`)     | `claude-sonnet-5-5` | The current Sonnet. Same prices and tokenizer; five breaking changes (`disabled` thinking 400s - use `between_tools`, forced `tool_choice` 400s, preserved thinking, computer use via the toolset on the Claude API and Google Cloud, fewer advisor pairings) - see Migrating to Claude Sonnet 5.5 |
| Sonnet 4.6                            | `claude-sonnet-5-5` | Apply the Claude Sonnet 5 section (adaptive thinking on by default, new tokenizer), then the Claude Sonnet 5.5 section, which replaces its thinking-disabled route, forced `tool_choice` on Bedrock, `computer_20251124`, and effort advice |
| Sonnet 4.0 / 4.5 / 3.7 / 3.5          | `claude-sonnet-5-5` | Apply the Sonnet 4.6 changes first, then the Claude Sonnet 5 and Claude Sonnet 5.5 sections |
| Haiku 3 / 3.5                         | `claude-haiku-4-5` | Fastest and most cost-effective                   |

| 如果你目前在用... | 迁移到 | 原因 |
| --- | --- | --- |
| Claude Mythos Preview（`claude-mythos-preview`） | `claude-mythos-5-1`（Project Glasswing 后继）或 `claude-fable-5-1`（GA） | 同一分词器家族——基本只是换模型 ID；移除 `thinking` 配置和 prefill；参见"迁移到 Claude Fable 5.1" |
| Claude Fable 5（`claude-fable-5`） | `claude-fable-5-1` | 同档位、同单价、同分词器；三项破坏性变更（强制 `tool_choice` 返回 400、"保留思考"）——参见"从 Claude Fable 5 迁移到 Claude Fable 5.1" |
| Claude Mythos 5（`claude-mythos-5`） | `claude-mythos-5-1` | 与 claude-fable-5 -> claude-fable-5-1 同一路径；参见"从 Claude Fable 5 迁移到 Claude Fable 5.1"中的 § Claude Mythos 5.1 |
| Claude Opus 5（`claude-opus-5`） | `claude-opus-5-5` | 当前 Opus。价格更低（$4 / $20 对 $5 / $25），上下文窗口与分词器相同；四项破坏性变更（思考无法禁用、强制 `tool_choice` 返回 400、保留思考、在 Claude API 与 Google Cloud 上经工具集进行计算机使用）——参见"迁移到 Claude Opus 5.5" |
| Opus 4.8 | `claude-opus-5-5` | 先应用 Claude Opus 5 一节（思考默认开启、提示词重新调优），再应用 Claude Opus 5.5 一节，后者取代 Claude Opus 5 的思考禁用路线与 `computer_20251124` |
| Opus 4.7 | `claude-opus-5-5` | 先应用 Opus 4.8 一节（提示词重新调优、无新增破坏性变更），再应用 Claude Opus 5 与 Claude Opus 5.5 两节 |
| Opus 4.6 | `claude-opus-5-5` | 先应用 Opus 4.7 的破坏性变更，再做 4.8 的重新调优，然后应用 Claude Opus 5 与 Claude Opus 5.5 两节 |
| Opus 4.0 / 4.1 / 4.5 / Opus 3 | `claude-opus-5-5` | 按 4.6 -> 4.7 -> 4.8 -> Claude Opus 5 -> Claude Opus 5.5 的顺序依次应用（自适应思考、去掉采样参数，然后重新调优） |
| Claude Sonnet 5（`claude-sonnet-5`） | `claude-sonnet-5-5` | 当前 Sonnet。价格与分词器相同；五项破坏性变更（`disabled` 思考返回 400——改用 `between_tools`、强制 `tool_choice` 返回 400、保留思考、在 Claude API 与 Google Cloud 上经工具集进行计算机使用、更少的 advisor 配对）——参见"迁移到 Claude Sonnet 5.5" |
| Sonnet 4.6 | `claude-sonnet-5-5` | 先应用 Claude Sonnet 5 一节（自适应思考默认开启、新分词器），再应用 Claude Sonnet 5.5 一节，后者取代其思考禁用路线、Bedrock 上的强制 `tool_choice`、`computer_20251124` 以及 effort 建议 |
| Sonnet 4.0 / 4.5 / 3.7 / 3.5 | `claude-sonnet-5-5` | 先应用 Sonnet 4.6 的变更，再应用 Claude Sonnet 5 与 Claude Sonnet 5.5 两节 |
| Haiku 3 / 3.5 | `claude-haiku-4-5` | 最快、最具成本效益 |

Default to the latest Opus (Claude Opus 5.5) for the caller's tier unless they explicitly chose otherwise. The Sonnet target is Claude Sonnet 5.5 (`claude-sonnet-5-5`). The Opus migrations layer: if you're on an older Opus, apply each version's section in order up to your target (e.g. 4.5 -> 4.8 means the 4.6, 4.7, and 4.8 sections in sequence). A 4.7 -> 4.8 move has no new breaking changes - see Migrating to Opus 4.8 below.

除非调用方明确另有选择，默认迁移到其档位下最新的 Opus（Claude Opus 5.5）。Sonnet 的目标是 Claude Sonnet 5.5（`claude-sonnet-5-5`）。Opus 的迁移是分层叠加的：如果你在较旧的 Opus 上，按顺序依次应用各版本小节直到目标版本（例如 4.5 -> 4.8 意味着依次应用 4.6、4.7、4.8 三节）。4.7 -> 4.8 的迁移没有新增破坏性变更——参见下文"迁移到 Opus 4.8"。

---

## Retired Model Replacements / 已退役模型替换

These models return 404 - update immediately:

这些模型返回 404——立即更新：

| Retired model                 | Retired       | Drop-in replacement  |
| ----------------------------- | ------------- | -------------------- |
| `claude-3-7-sonnet-20250219`  | Feb 19, 2026  | `claude-sonnet-5-5` |
| `claude-3-5-haiku-20241022`   | Feb 19, 2026  | `claude-haiku-4-5`   |
| `claude-3-opus-20240229`      | Jan 5, 2026   | `claude-opus-4-8`    |
| `claude-3-5-sonnet-20241022`  | Oct 28, 2025  | `claude-sonnet-5-5` |
| `claude-3-5-sonnet-20240620`  | Oct 28, 2025  | `claude-sonnet-5-5` |
| `claude-3-sonnet-20240229`    | Jul 21, 2025  | `claude-sonnet-5-5` |
| `claude-2.1`, `claude-2.0`    | Jul 21, 2025  | `claude-sonnet-5-5` |

| 已退役模型 | 退役时间 | 直接替换 |
| --- | --- | --- |
| `claude-3-7-sonnet-20250219` | 2026年2月19日 | `claude-sonnet-5-5` |
| `claude-3-5-haiku-20241022` | 2026年2月19日 | `claude-haiku-4-5` |
| `claude-3-opus-20240229` | 2026年1月5日 | `claude-opus-4-8` |
| `claude-3-5-sonnet-20241022` | 2025年10月28日 | `claude-sonnet-5-5` |
| `claude-3-5-sonnet-20240620` | 2025年10月28日 | `claude-sonnet-5-5` |
| `claude-3-sonnet-20240229` | 2025年7月21日 | `claude-sonnet-5-5` |
| `claude-2.1`, `claude-2.0` | 2025年7月21日 | `claude-sonnet-5-5` |

## Deprecated Models (retiring soon) / 已弃用模型（即将退役）

| Model                         | Retires       | Replacement          |
| ----------------------------- | ------------- | -------------------- |
| `claude-3-haiku-20240307`     | Apr 19, 2026  | `claude-haiku-4-5`   |
| `claude-opus-4-20250514`      | June 15, 2026 | `claude-opus-4-8`    |
| `claude-sonnet-4-20250514`    | June 15, 2026 | `claude-sonnet-5-5` |

| 模型 | 退役时间 | 替代 |
| --- | --- | --- |
| `claude-3-haiku-20240307` | 2026年4月19日 | `claude-haiku-4-5` |
| `claude-opus-4-20250514` | 2026年6月15日 | `claude-opus-4-8` |
| `claude-sonnet-4-20250514` | 2026年6月15日 | `claude-sonnet-5-5` |

---

## Breaking Changes by Source Model / 按源模型划分的破坏性变更

### Migrating from Sonnet 4.5 to Sonnet 4.6 (effort default change) / 从 Sonnet 4.5 迁移到 Sonnet 4.6（effort 默认值变化）

Sonnet 4.5 had no `effort` parameter; Sonnet 4.6 defaults to `high`. If you just switch the model string and do nothing else, you may see noticeably higher latency and token usage. Set `effort` explicitly.

Sonnet 4.5 没有 `effort` 参数；Sonnet 4.6 默认为 `high`。如果只换模型字符串而不做别的，可能看到明显更高的延迟与 token 用量。请显式设置 `effort`。

**Recommended starting points:**

**推荐的起始点：**

| Workload                                          | Start at       | Notes                                                                                                    |
| ------------------------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------- |
| Chat, classification, content generation          | `low`          | With `thinking: {"type": "disabled"}` you'll see similar or better performance vs. Sonnet 4.5 no-thinking |
| Most applications (balanced)                      | `medium`       | The default sweet spot for quality vs. cost                                                              |
| Agentic coding, tool-heavy workflows              | `medium`       | Pair with adaptive thinking and a generous `max_tokens` (up to 128K with streaming - Sonnet 4.6's ceiling) |
| Autonomous multi-step agents, long-horizon loops  | `high`         | Scale down to `medium` if latency/tokens become a concern                                                 |
| Computer-use agents                               | `high` + adaptive | Sonnet 4.6's best computer-use accuracy is on adaptive + high                                          |

| 工作负载 | 起始值 | 说明 |
| --- | --- | --- |
| 聊天、分类、内容生成 | `low` | 配合 `thinking: {"type": "disabled"}`，对比 Sonnet 4.5 无思考模式性能相当或更好 |
| 大多数应用（均衡） | `medium` | 质量与成本权衡的默认最佳点 |
| 智能体编码、工具密集型工作流 | `medium` | 配合自适应思考与宽裕的 `max_tokens`（流式传输下最高 128K——Sonnet 4.6 的上限） |
| 自主多步智能体、长程循环 | `high` | 若延迟/token 成为顾虑，降到 `medium` |
| 计算机使用智能体 | `high` + adaptive | Sonnet 4.6 最佳的计算机使用准确率出现在 adaptive + high 上 |

For non-thinking chat workloads specifically:

专门针对非思考的聊天工作负载：

```python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8192,
    thinking={"type": "disabled"},
    output_config={"effort": "low"},
    messages=[{"role": "user", "content": "..."}],
)
```

**When to use Opus 4.6 instead:** hardest and longest-horizon problems - large code migrations, deep research, extended autonomous work. Sonnet 4.6 wins on fast turnaround and cost efficiency.

**何时改用 Opus 4.6：**最难、最长程的问题——大型代码迁移、深度研究、长时间自主工作。Sonnet 4.6 在快速周转与成本效率上占优。
### Migrating to Opus 4.6 / Sonnet 4.6 (from any older model) / 迁移到 Opus 4.6 / Sonnet 4.6（自任意更旧模型）

**1. Manual extended thinking is deprecated - use adaptive thinking.**

**1. 手动扩展思考已弃用——改用自适应思考。**

`thinking: {type: "enabled", budget_tokens: N}` (manual extended thinking with a fixed token budget) is deprecated on Opus 4.6 and Sonnet 4.6. Replace it with `thinking: {type: "adaptive"}`, which lets Claude decide when and how much to think. Adaptive thinking also enables interleaved thinking automatically (no beta header needed).

`thinking: {type: "enabled", budget_tokens: N}`（固定 token 预算的手动扩展思考）在 Opus 4.6 与 Sonnet 4.6 上已弃用。替换为 `thinking: {type: "adaptive"}`，让 Claude 自己决定何时思考、思考多少。自适应思考还会自动启用交错思考（无需 beta 头）。

```python
# Old (still works on older models, deprecated on 4.6)
response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    messages=[...]
)

# New (Opus 4.6 / Sonnet 4.6)
response = client.messages.create(
    model="claude-opus-4-6",  # or "claude-sonnet-4-6"
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # optional: low | medium | high | max
    messages=[...]
)
```

Adaptive thinking is the long-term target, and on internal evaluations it outperforms manual extended thinking. Move when you can.

自适应思考是长期目标，在内部评测中它优于手动扩展思考。能迁就迁。

**Transitional escape hatch:** manual extended thinking is still *functional* on Opus 4.6 and Sonnet 4.6 (deprecated, will be removed in a future release). If you need a hard ceiling while migrating - for example, to bound token spend on a runaway workload before you've tuned `effort` - you can keep `budget_tokens` around alongside an explicit `effort` value, then remove it in a follow-up. `budget_tokens` must be strictly less than `max_tokens`:

**过渡性逃生舱口：**手动扩展思考在 Opus 4.6 与 Sonnet 4.6 上仍*可用*（已弃用，将在未来版本中移除）。如果迁移期间需要硬上限——例如在调好 `effort` 之前约束某个失控工作负载的 token 花费——可以暂时保留 `budget_tokens` 并配合显式的 `effort` 值，随后再移除。`budget_tokens` 必须严格小于 `max_tokens`：

```python
# Transitional only - deprecated, plan to remove
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16384,
    thinking={"type": "enabled", "budget_tokens": 8192},  # must be < max_tokens
    output_config={"effort": "medium"},
    messages=[...],
)
```

If the user asks for a "thinking budget" on 4.6, the preferred answer is `effort` - use `low`, `medium`, `high`, or `max` rather than a token count.

如果用户在 4.6 上要一个"思考预算"，首选答案是 `effort`——用 `low`、`medium`、`high` 或 `max`，而不是 token 数。

**2. Effort parameter (Opus 4.5, Opus 4.6, Sonnet 4.6 only).**

**2. Effort 参数（仅 Opus 4.5、Opus 4.6、Sonnet 4.6）。**

Controls thinking depth and overall token spend. Goes inside `output_config`, not top-level. Default is `high`. `max` is supported on Fable 5, Opus 4.6 and later, Sonnet 5.5, Sonnet 5, and Sonnet 4.6 - it errors on Sonnet 4.5 and Haiku 4.5.

控制思考深度与整体 token 花费。放在 `output_config` 内部，而非顶层。默认值为 `high`。`max` 在 Fable 5、Opus 4.6 及之后、Sonnet 5.5、Sonnet 5 与 Sonnet 4.6 上受支持——在 Sonnet 4.5 与 Haiku 4.5 上会报错。

```python
output_config={"effort": "medium"}  # often the best cost / quality balance
```

### Migrating to the 4.6 family (Opus 4.6 and Sonnet 4.6) / 迁移到 4.6 家族（Opus 4.6 与 Sonnet 4.6）

**3. Assistant-turn prefills return 400 (Opus 4.6 and Sonnet 4.6).**

**3. assistant 回合 prefill 返回 400（Opus 4.6 与 Sonnet 4.6）。**

Prefilled responses on the final assistant turn are no longer supported on either Opus 4.6 or Sonnet 4.6 - both return a 400. Adding assistant messages *elsewhere* in the conversation (e.g., for few-shot examples) still works. Pick the replacement that matches what the prefill was doing:

在最后一个 assistant 回合上预填充响应在 Opus 4.6 与 Sonnet 4.6 上都不再受支持——两者都返回 400。在对话*其他位置*添加 assistant 消息（例如用于 few-shot 示例）仍然有效。按 prefill 原来的用途选择替代方案：

| Prefill was used for                               | Replacement                                                                                                                               |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Forcing JSON / YAML / schema output                | `output_config.format` with a `json_schema` - see example below                                                                           |
| Forcing a classification label                     | Tool with an enum field containing valid labels, or structured outputs                                                                    |
| Skipping preambles (`Here is the summary:\n`)      | System prompt instruction: *"Respond directly without preamble. Do not start with phrases like 'Here is...' or 'Based on...'."*           |
| Steering around bad refusals                       | Usually no longer needed - 4.6 refuses far more appropriately. Plain user-turn prompting is sufficient.                                   |
| Continuing an interrupted response                 | Move continuation into the user turn: *"Your previous response was interrupted and ended with `[last text]`. Continue from there."*     |
| Injecting reminders / context hydration            | Inject into the user turn instead. For complex agent harnesses, expose context via a tool call or during compaction.                      |

| Prefill 原用途 | 替代方案 |
| --- | --- |
| 强制 JSON / YAML / schema 输出 | `output_config.format` 配 `json_schema`——见下方示例 |
| 强制分类标签 | 带有效标签枚举字段的工具，或结构化输出 |
| 跳过开场白（`Here is the summary:\n`） | 系统提示词指令：*"Respond directly without preamble. Do not start with phrases like 'Here is...' or 'Based on...'."*（"直接回应，不要开场白。别以 'Here is...' 或 'Based on...' 之类的短语开头。"） |
| 绕开不当拒答 | 通常不再需要——4.6 的拒答恰当得多。普通的用户回合提示即可。 |
| 续写被中断的响应 | 把续写移入用户回合：*"Your previous response was interrupted and ended with `[last text]`. Continue from there."*（"你上一次的响应被中断，止于 `[last text]`。从那里继续。"） |
| 注入提醒 / 上下文补水 | 改为注入用户回合。对复杂的智能体框架，经工具调用或在压缩（compaction）时暴露上下文。 |

```python
# Old (fails on Opus 4.6 / Sonnet 4.6) - prefill forcing JSON shape
messages=[
    {"role": "user", "content": "Extract the name."},
    {"role": "assistant", "content": "{\"name\": \""},
]

# New - structured outputs replace the prefill
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    output_config={"format": {"type": "json_schema", "schema": {...}}},
    messages=[{"role": "user", "content": "Extract the name."}],
)
```

**4. Stream for `max_tokens > ~16K` (all models); only Haiku 4.5 caps lower, at 64K.**

**4. `max_tokens > ~16K` 时使用流式传输（所有模型）；只有 Haiku 4.5 上限更低，为 64K。**

Non-streaming requests hit SDK HTTP timeouts at high `max_tokens`, regardless of model - stream for anything above ~16K output. The streamable ceiling is 128K for every current model except Haiku 4.5, which caps at 64K.

高 `max_tokens` 下非流式请求会触发 SDK HTTP 超时，与模型无关——凡输出超过约 16K 就用流式传输。当前所有模型的可流式上限都是 128K，唯 Haiku 4.5 为 64K。

```python
with client.messages.stream(model="claude-opus-4-6", max_tokens=64000, ...) as stream:
    message = stream.get_final_message()
```

**5. Tool-call JSON escaping may differ (Opus 4.6 and Sonnet 4.6).**

**5. 工具调用 JSON 转义可能不同（Opus 4.6 与 Sonnet 4.6）。**

Both 4.6 models can produce tool call `input` fields with Unicode or forward-slash escaping. Always parse with `json.loads()` / `JSON.parse()` - never raw-string-match the serialized input.

两个 4.6 模型生成的工具调用 `input` 字段可能带 Unicode 或正斜杠转义。务必用 `json.loads()` / `JSON.parse()` 解析——绝不要对序列化后的输入做原始字符串匹配。

### All models / 所有模型

**6. `output_format` -> `output_config.format` (API-wide).**

**6. `output_format` -> `output_config.format`（API 全局）。**

The old top-level `output_format` parameter on `messages.create()` is deprecated. Use `output_config.format` instead. This is not 4.6-specific - applies to every model.

`messages.create()` 上旧的顶层 `output_format` 参数已弃用。改用 `output_config.format`。这并非 4.6 独有——适用于所有模型。

---

## Beta Headers to Remove on 4.6 / 4.6 上应移除的 Beta 头

Several beta headers that were required on 4.5 are now GA on 4.6 and should be removed. Leaving them in is harmless but misleading; removing them also lets you move from `client.beta.messages.create(...)` back to `client.messages.create(...)`.

几个在 4.5 上必需的 beta 头如今在 4.6 上已 GA，应当移除。留着无害但有误导性；移除它们还能让你从 `client.beta.messages.create(...)` 换回 `client.messages.create(...)`。

| Header                                    | Status on 4.6                                              | Action                                                  |
| ----------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------- |
| `effort-2025-11-24`                       | Effort parameter is GA                                     | Remove                                                  |
| `fine-grained-tool-streaming-2025-05-14`  | GA                                                         | Remove                                                  |
| `interleaved-thinking-2025-05-14`         | Adaptive thinking enables interleaved thinking automatically | Remove when using adaptive thinking; still functional on Sonnet 4.6 *with* manual extended thinking, but that path is deprecated |
| `token-efficient-tools-2025-02-19`        | Built in to all Claude 4+ models                           | Remove (no effect)                                      |
| `output-128k-2025-02-19`                  | Built in to Claude 4+ models                               | Remove (no effect)                                      |

| 头 | 4.6 上的状态 | 处理 |
| --- | --- | --- |
| `effort-2025-11-24` | effort 参数已 GA | 移除 |
| `fine-grained-tool-streaming-2025-05-14` | 已 GA | 移除 |
| `interleaved-thinking-2025-05-14` | 自适应思考会自动启用交错思考 | 使用自适应思考时移除；在 Sonnet 4.6 *配合*手动扩展思考时仍可用，但该路径已弃用 |
| `token-efficient-tools-2025-02-19` | 已内置于所有 Claude 4+ 模型 | 移除（无效果） |
| `output-128k-2025-02-19` | 已内置于 Claude 4+ 模型 | 移除（无效果） |

Once you remove all of these and finish moving to adaptive thinking, you can switch the SDK call site from the beta namespace back to the regular one:

全部移除并完成向自适应思考的迁移后，即可把 SDK 调用点从 beta 命名空间换回常规命名空间：

```python
# Before
response = client.beta.messages.create(
    model="claude-opus-4-5",
    betas=["interleaved-thinking-2025-05-14", "effort-2025-11-24"],
    ...
)

# After
response = client.messages.create(
    model="claude-opus-4-6",
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    ...
)
```
---

## Additional Changes When Coming from 3.x / 4.0 / 4.1 -> 4.6 / 从 3.x / 4.0 / 4.1 迁到 4.6 的额外变更

If you're jumping from Opus 4.1, Sonnet 4, Sonnet 3.7, or an older Claude 3.x model directly to 4.6, apply everything above *plus* the items in this section. Users already on Opus 4.5 / Sonnet 4.5 can skip this.

如果你从 Opus 4.1、Sonnet 4、Sonnet 3.7 或更旧的 Claude 3.x 模型直接跳到 4.6，请在应用上述所有内容之外*加上*本节的条目。已经在 Opus 4.5 / Sonnet 4.5 上的用户可跳过。

**1. Sampling parameters: `temperature` OR `top_p`, not both.**

**1. 采样参数：`temperature` 或 `top_p`，二选一。**

Passing both will error on every Claude 4+ model:

同时传两者在每个 Claude 4+ 模型上都会报错：

```python
# Old (3.x only - errors on 4+)
client.messages.create(temperature=0.7, top_p=0.9, ...)

# New
client.messages.create(temperature=0.7, ...)  # or top_p, not both
```

**2. Update tool versions.**

**2. 更新工具版本。**

Legacy tool versions are not supported on 4+. **Both the `type` and the `name` field change** - `text_editor_20250728` and `str_replace_based_edit_tool` are a pair; updating one without the other 400s. Also remove the `undo_edit` command from your text-editor integration:

旧工具版本在 4+ 上不受支持。**`type` 与 `name` 两个字段都要改**——`text_editor_20250728` 与 `str_replace_based_edit_tool` 是一对；只更新其一而不改另一个会返回 400。同时从文本编辑器集成中移除 `undo_edit` 命令：

| Old                                               | New                                                     |
| ------------------------------------------------- | ------------------------------------------------------- |
| `text_editor_20250124` + `str_replace_editor`     | `text_editor_20250728` + `str_replace_based_edit_tool`  |
| `code_execution_*` (earlier versions)             | `code_execution_20260521`                               |
| `undo_edit` command                               | *(no longer supported - delete call sites)*             |

| 旧 | 新 |
| --- | --- |
| `text_editor_20250124` + `str_replace_editor` | `text_editor_20250728` + `str_replace_based_edit_tool` |
| `code_execution_*`（更早版本） | `code_execution_20260521` |
| `undo_edit` 命令 | *（不再支持——删除调用点）* |

```python
# Before
tools = [{"type": "text_editor_20250124", "name": "str_replace_editor"}]

# After - BOTH fields change
tools = [{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}]
```

**3. Handle the `refusal` stop reason.**

**3. 处理 `refusal` 停止原因。**

Claude 4+ can return `stop_reason: "refusal"` on the response. If your code only handles `end_turn` / `tool_use` / `max_tokens`, add a branch:

Claude 4+ 可在响应中返回 `stop_reason: "refusal"`。如果你的代码只处理 `end_turn` / `tool_use` / `max_tokens`，请加一个分支：

```python
if response.stop_reason == "refusal":
    # Surface the refusal to the user; do not retry with the same prompt
    ...
```

**4. Handle the `model_context_window_exceeded` stop reason (4.5+).**

**4. 处理 `model_context_window_exceeded` 停止原因（4.5+）。**

Distinct from `max_tokens`: it means the model hit the *context window* limit, not the requested output cap. Handle both:

与 `max_tokens` 不同：它表示模型撞上的是*上下文窗口*限制，而非请求的输出上限。两者都要处理：

```python
if response.stop_reason == "model_context_window_exceeded":
    # Context window exhausted - compact or split the conversation
    ...
elif response.stop_reason == "max_tokens":
    # Requested output cap hit - retry with higher max_tokens or stream
    ...
```

**5. Trailing newlines preserved in tool call string parameters (4.5+).**

**5. 工具调用字符串参数保留末尾换行（4.5+）。**

4.5 and 4.6 preserve trailing newlines that older models stripped. If your tool implementations do exact string matching against tool-call `input` values (e.g., `if name == "foo"`), verify they still match when the model sends `"foo\n"`. Normalizing with `.rstrip()` on the receiving side is usually the simplest fix.

4.5 与 4.6 会保留旧模型会剥掉的末尾换行。如果你的工具实现对工具调用 `input` 值做精确字符串匹配（如 `if name == "foo"`），请验证模型发送 `"foo\n"` 时仍能匹配。在接收侧用 `.rstrip()` 归一化通常是最简单的修复。

**6. Haiku: rate limits reset between generations.**

**6. Haiku：速率限制在代际之间重置。**

Haiku 4.5 has its own rate-limit pool separate from Haiku 3 / 3.5. If you're ramping traffic as you migrate, check your tier's Haiku 4.5 limits at [API rate limits](https://platform.claude.com/docs/en/api/rate-limits) - a quota that comfortably served Haiku 3.5 traffic may need a tier bump for the same volume on 4.5.

Haiku 4.5 有独立于 Haiku 3 / 3.5 的限流池。如果迁移期间流量在爬坡，请在 [API 速率限制](https://platform.claude.com/docs/en/api/rate-limits)查看你档位的 Haiku 4.5 限制——一个从容服务 Haiku 3.5 流量的配额，在 4.5 上承载同样流量可能需要升档。

---

## Prompt-Behavior Changes (Opus 4.5 / 4.6, Sonnet 4.6) / 提示词行为变化（Opus 4.5 / 4.6、Sonnet 4.6）

These don't break your code, but prompts that worked on 4.5-and-earlier may over- or under-trigger on 4.6. Tune as needed. For a standing, model-general audit of dated prompt text beyond this migration - skills and tool descriptions included - read `shared/prompt-audit.md` (or invoke `/claude-api prompt-audit`).

这些不会弄坏你的代码，但在 4.5 及更早版本上有效的提示词在 4.6 上可能过度触发或触发不足。按需调优。若要在本次迁移之外对过时的提示词文本做经常性的、模型无关的审计——包括技能与工具描述——阅读 `shared/prompt-audit.md`（或调用 `/claude-api prompt-audit`）。

**1. Aggressive instructions cause overtriggering.** Opus 4.5 and 4.6 follow the system prompt much more closely than earlier models. Prompts written to *overcome* the old reluctance are now too aggressive:

**1. 激进的指令导致过度触发。**Opus 4.5 与 4.6 对系统提示词的遵循比早期模型紧密得多。为*克服*旧模型不情愿而写的提示词现在过于激进：

| Before (worked on 4.0 / 4.5)                | After (use on 4.6)                        |
| ------------------------------------------- | ----------------------------------------- |
| `CRITICAL: You MUST use this tool when...`  | `Use this tool when...`                   |
| `Default to using [tool]`                   | `Use [tool] when it would improve X`      |
| `If in doubt, use [tool]`                   | *(delete - no longer needed)*             |

| 之前（在 4.0 / 4.5 上有效） | 之后（在 4.6 上使用） |
| --- | --- |
| `CRITICAL: You MUST use this tool when...` | `Use this tool when...` |
| `Default to using [tool]` | `Use [tool] when it would improve X` |
| `If in doubt, use [tool]` | *（删除——不再需要）* |

If the model is now overtriggering a tool or skill, the fix is almost always to dial back the language, not to add more guardrails.

如果模型现在过度触发某个工具或技能，修复几乎总是收敛措辞，而不是增加更多护栏。

**2. Overthinking and excessive exploration (Opus 4.6).** At higher `effort` settings, Opus 4.6 explores more before answering. If that burns too many thinking tokens, lower `effort` first (`medium` is often the sweet spot) before adding prose instructions to constrain reasoning.

**2. 过度思考与过度探索（Opus 4.6）。**在更高的 `effort` 设置下，Opus 4.6 回答前会做更多探索。如果这烧掉太多思考 token，先降低 `effort`（`medium` 常是最佳点），再考虑加散文式指令去约束推理。

**3. Overeager subagent spawning (Opus 4.6).** Opus 4.6 has a strong preference for delegating to subagents. If you see it spawning a subagent for something a direct `grep` or `read` would solve, add guidance: *"Use subagents only for parallel or independent workstreams. For single-file reads or sequential operations, work directly."*

**3. 过度热衷的子代理派生（Opus 4.6）。**Opus 4.6 非常偏好把工作委派给子代理。如果看到它为一个直接 `grep` 或 `read` 就能解决的事情派生子代理，添加指导：*"Use subagents only for parallel or independent workstreams. For single-file reads or sequential operations, work directly."*（"只在并行或独立的工作流中使用子代理。单文件读取或顺序操作，直接动手。"）

**4. Overengineering (Opus 4.5 / 4.6).** Both models may add extra files, abstractions, or defensive error handling beyond what was asked. If you want minimal changes, prompt for it explicitly: *"Only make changes directly requested. Don't add helpers, abstractions, or error handling for scenarios that can't happen."*

**4. 过度工程（Opus 4.5 / 4.6）。**两个模型都可能超出所请，添加额外的文件、抽象或防御性错误处理。若想要最小改动，明确提示：*"Only make changes directly requested. Don't add helpers, abstractions, or error handling for scenarios that can't happen."*（"只做直接要求的改动。不要为不可能发生的场景添加辅助函数、抽象或错误处理。"）

**5. LaTeX math output (Opus 4.6).** Opus 4.6 defaults to LaTeX (`\frac{}{}`, `$...$`) for math and technical content. If you need plain text, instruct it explicitly: *"Format all math as plain text - no LaTeX, no `$`, no `\frac{}{}`. Use `/` for division and `^` for exponents."*

**5. LaTeX 数学输出（Opus 4.6）。**Opus 4.6 对数学与技术内容默认输出 LaTeX（`\frac{}{}`、`$...$`）。若需要纯文本，明确指示：*"Format all math as plain text - no LaTeX, no `$`, no `\frac{}{}`. Use `/` for division and `^` for exponents."*（"所有数学都用纯文本格式——不要 LaTeX、不要 `$`、不要 `\frac{}{}`。除法用 `/`，指数用 `^`。"）

**6. Skipped verbal summaries (4.6 family).** The 4.6 models are more concise and may skip the summary paragraph after a tool call, jumping straight to the next action. If you rely on those summaries for visibility, add: *"After completing a task that involves tool use, provide a brief summary of what you did."*

**6. 跳过口头总结（4.6 家族）。**4.6 模型更简洁，可能跳过工具调用后的总结段落，直接进行下一步动作。如果你依赖这些总结获得可见性，添加：*"After completing a task that involves tool use, provide a brief summary of what you did."*（"完成涉及工具使用的任务后，简要总结你做了什么。"）

**7. "Think" as a trigger word (Opus 4.5 with thinking disabled).** When `thinking` is off, Opus 4.5 is particularly sensitive to the word *think* and may reason more than you want. Use `consider`, `evaluate`, or `reason through` instead.

**7. "Think"作为触发词（关闭思考的 Opus 4.5）。**当 `thinking` 关闭时，Opus 4.5 对 *think* 一词特别敏感，可能推理得超出你的预期。改用 `consider`、`evaluate` 或 `reason through`。
---

## Model-ID Rename Quick Reference / 模型 ID 更名速查

| Old string (migration source)  | New string         |
| ------------------------------ | ------------------ |
| `claude-opus-5`             | `claude-opus-5-5`      |
| `claude-opus-4-8`              | `claude-opus-5-5`      |
| `claude-opus-4-7`              | `claude-opus-5-5`      |
| `claude-opus-4-6`              | `claude-opus-5-5`      |
| `claude-opus-4-5`              | `claude-opus-5-5`      |
| `claude-opus-4-1`              | `claude-opus-5-5`      |
| `claude-opus-4-0`              | `claude-opus-5-5`      |
| `claude-mythos-preview`        | `claude-mythos-5-1` (Project Glasswing) or `claude-fable-5-1` |
| `claude-fable-5`            | `claude-fable-5-1`     |
| `claude-mythos-5`           | `claude-mythos-5-1`    |
| `claude-sonnet-4-6`            | `claude-sonnet-5-5`     |
| `claude-sonnet-4-5`            | `claude-sonnet-5-5`     |
| `claude-sonnet-4-0`            | `claude-sonnet-5-5`     |
| `claude-sonnet-5`                | `claude-sonnet-5-5`     |

| 旧字符串（迁移源） | 新字符串 |
| --- | --- |
| `claude-opus-5` | `claude-opus-5-5` |
| `claude-opus-4-8` | `claude-opus-5-5` |
| `claude-opus-4-7` | `claude-opus-5-5` |
| `claude-opus-4-6` | `claude-opus-5-5` |
| `claude-opus-4-5` | `claude-opus-5-5` |
| `claude-opus-4-1` | `claude-opus-5-5` |
| `claude-opus-4-0` | `claude-opus-5-5` |
| `claude-mythos-preview` | `claude-mythos-5-1`（Project Glasswing）或 `claude-fable-5-1` |
| `claude-fable-5` | `claude-fable-5-1` |
| `claude-mythos-5` | `claude-mythos-5-1` |
| `claude-sonnet-4-6` | `claude-sonnet-5-5` |
| `claude-sonnet-4-5` | `claude-sonnet-5-5` |
| `claude-sonnet-4-0` | `claude-sonnet-5-5` |
| `claude-sonnet-5` | `claude-sonnet-5-5` |

Older aliases (`claude-opus-4-7`, `claude-opus-4-6`, `claude-opus-4-5`, `claude-sonnet-4-6`, `claude-sonnet-4-5`, etc.) are still active and can be pinned if you need time before upgrading - see `shared/models.md` for the full legacy list.

较旧的别名（`claude-opus-4-7`、`claude-opus-4-6`、`claude-opus-4-5`、`claude-sonnet-4-6`、`claude-sonnet-4-5` 等）仍然有效，如果你需要升级前的缓冲时间，可以钉住它们——完整旧版清单见 `shared/models.md`。

### Amazon Bedrock model IDs / Amazon Bedrock 模型 ID

If the code uses the `AnthropicBedrockMantle` client (Python `anthropic[bedrock]`, TypeScript `@anthropic-ai/bedrock-sdk`, Java `BedrockMantleBackend`, Go `bedrock.NewMantleClient`, etc.) or targets `https://bedrock-mantle.{region}.api.aws/anthropic`, it is running on **Claude in Amazon Bedrock**. All breaking changes in this guide apply unchanged there - it serves the same Messages API shape - but model IDs carry an `anthropic.` provider prefix:

如果代码使用 `AnthropicBedrockMantle` 客户端（Python `anthropic[bedrock]`、TypeScript `@anthropic-ai/bedrock-sdk`、Java `BedrockMantleBackend`、Go `bedrock.NewMantleClient` 等）或指向 `https://bedrock-mantle.{region}.api.aws/anthropic`，它运行在 **Claude in Amazon Bedrock** 上。本指南的所有破坏性变更在那里原样适用——它提供相同的 Messages API 形状——但模型 ID 带 `anthropic.` 提供方前缀：

| First-party ID | Bedrock ID |
|---|---|
| `claude-opus-4-8` | `anthropic.claude-opus-4-8` |
| `claude-opus-5` | `anthropic.claude-opus-5` |
| `claude-opus-5-5` | `anthropic.claude-opus-5-5` |
| `claude-fable-5-1` | `anthropic.claude-fable-5-1` |
| `claude-fable-5` | `anthropic.claude-fable-5` |
| `claude-mythos-5-1` | `anthropic.claude-mythos-5-1` (us-east-1 only, not publicly listed) |
| `claude-opus-4-7` | `anthropic.claude-opus-4-7` |
| `claude-sonnet-5` | `anthropic.claude-sonnet-5` |
| `claude-sonnet-5-5` | `anthropic.claude-sonnet-5-5` |
| `claude-haiku-4-5` | `anthropic.claude-haiku-4-5` |

| 第一方 ID | Bedrock ID |
|---|---|
| `claude-opus-4-8` | `anthropic.claude-opus-4-8` |
| `claude-opus-5` | `anthropic.claude-opus-5` |
| `claude-opus-5-5` | `anthropic.claude-opus-5-5` |
| `claude-fable-5-1` | `anthropic.claude-fable-5-1` |
| `claude-fable-5` | `anthropic.claude-fable-5` |
| `claude-mythos-5-1` | `anthropic.claude-mythos-5-1`（仅 us-east-1，未公开列出） |
| `claude-opus-4-7` | `anthropic.claude-opus-4-7` |
| `claude-sonnet-5` | `anthropic.claude-sonnet-5` |
| `claude-sonnet-5-5` | `anthropic.claude-sonnet-5-5` |
| `claude-haiku-4-5` | `anthropic.claude-haiku-4-5` |

When migrating a Bedrock file, apply the same rename-table row as first-party, then keep/add the `anthropic.` prefix. Do **not** generate a first-party `claude-*` ID for a Bedrock client - it will 400.

迁移 Bedrock 文件时，按第一方同样套用更名表的行，然后保留/添加 `anthropic.` 前缀。**不要**为 Bedrock 客户端生成第一方 `claude-*` ID——那会返回 400。

**Skip for Bedrock:** the `code_execution_*` tool-version checklist item and the **Task Budgets** section - neither is available on Bedrock (see `shared/platform-availability.md` for the per-feature table). Everything else in this guide - `effort`, adaptive/extended thinking, `output_config.format`, `thinking.display`, token counting - is available on Bedrock; fine-grained tool streaming (`eager_input_streaming`) is available on Bedrock's newer serving stack only (see the per-model note in `shared/platform-availability.md`).

**Bedrock 跳过项：**`code_execution_*` 工具版本清单条目和 **Task Budgets** 一节——两者在 Bedrock 上都不可用（逐特性表见 `shared/platform-availability.md`）。本指南其余一切——`effort`、自适应/扩展思考、`output_config.format`、`thinking.display`、token 计数——在 Bedrock 上都可用；细粒度工具流式传输（`eager_input_streaming`）仅在 Bedrock 较新的服务栈上可用（见 `shared/platform-availability.md` 中的按模型说明）。

> **Out of scope:** the legacy Amazon Bedrock integration (`InvokeModel` / `Converse` APIs with ARN-versioned IDs like `anthropic.claude-3-5-sonnet-20241022-v2:0`) uses a different request shape and model-ID format. This guide does not cover it; WebFetch the Bedrock page in `shared/live-sources.md` if the user is migrating between the two Bedrock integrations.

> **超出范围：**旧的 Amazon Bedrock 集成（`InvokeModel` / `Converse` API，使用形如 `anthropic.claude-3-5-sonnet-20241022-v2:0` 的 ARN 版本化 ID）使用不同的请求形状与模型 ID 格式。本指南不覆盖它；如果用户在两个 Bedrock 集成之间迁移，请 WebFetch `shared/live-sources.md` 中的 Bedrock 页面。

### Claude Platform on AWS / AWS 上的 Claude Platform

If the code uses `AnthropicAWS` / `AnthropicAws` / `anthropicaws.NewClient` / `AnthropicAwsClient` (or targets `https://aws-external-anthropic.{region}.api.aws`), it is running on **Claude Platform on AWS** - Anthropic-operated, same-day API parity. Model IDs are **bare first-party** strings; apply the rename table above **verbatim** and every breaking-change section in this guide unchanged. There is nothing to skip. Do **not** add an `anthropic.` prefix (that's Amazon Bedrock, a separate offering). See `shared/claude-platform-on-aws.md` for client/auth details.

如果代码使用 `AnthropicAWS` / `AnthropicAws` / `anthropicaws.NewClient` / `AnthropicAwsClient`（或指向 `https://aws-external-anthropic.{region}.api.aws`），它运行在 **Claude Platform on AWS** 上——由 Anthropic 运营、API 同日对齐。模型 ID 是**裸的第一方**字符串；**原封不动**地应用上面的更名表和本指南的每个破坏性变更节。没有需要跳过的内容。**不要**添加 `anthropic.` 前缀（那是 Amazon Bedrock，另一个产品）。客户端/认证细节见 `shared/claude-platform-on-aws.md`。

---

## Migration Checklist / 迁移清单

Every item is tagged: **`[BLOCKS]`** items cause a 400 error, infinite loop, silent timeout, or wrong tool selection if missed - apply these as code edits, not as suggestions. **`[TUNE]`** items are quality/cost adjustments.

每一项都有标记：**`[BLOCKS]`** 项一旦遗漏会导致 400 错误、无限循环、静默超时或错误的工具选择——把它们当作代码编辑来执行，而不是建议。**`[TUNE]`** 项是质量/成本调整。

For each file that calls `messages.create()` / equivalent SDK method:

对每个调用 `messages.create()` / 等价 SDK 方法的文件：

- [ ] **[BLOCKS]** Update the `model=` string to the new alias
      **[BLOCKS]** 把 `model=` 字符串更新为新别名
- [ ] **[BLOCKS]** Replace `budget_tokens` with `thinking={"type": "adaptive"}` (deprecated on Opus 4.6 / Sonnet 4.6)
      **[BLOCKS]** 用 `thinking={"type": "adaptive"}` 替换 `budget_tokens`（在 Opus 4.6 / Sonnet 4.6 上已弃用）
- [ ] **[BLOCKS]** Move `format` from top-level `output_format` into `output_config.format`
      **[BLOCKS]** 把 `format` 从顶层 `output_format` 移入 `output_config.format`
- [ ] **[BLOCKS]** Remove any assistant-turn prefills if targeting Opus 4.6 or Sonnet 4.6 (see the prefill replacement table)
      **[BLOCKS]** 若目标是 Opus 4.6 或 Sonnet 4.6，移除所有 assistant 回合 prefill（见 prefill 替代表）
- [ ] **[BLOCKS]** Switch to streaming if `max_tokens > ~16000` (otherwise SDK HTTP timeout)
      **[BLOCKS]** 若 `max_tokens > ~16000`，切换到流式传输（否则 SDK HTTP 超时）
- [ ] **[TUNE]** Verify tool-input handling parses JSON rather than raw-string-matching the serialized input (4.6 may escape Unicode / forward slashes differently; most SDKs already expose `block.input` as a parsed object)
      **[TUNE]** 核实工具输入处理是对 JSON 做解析，而不是对序列化输入做原始字符串匹配（4.6 对 Unicode / 正斜杠的转义可能不同；多数 SDK 已把 `block.input` 暴露为解析后的对象）
- [ ] **[TUNE]** Set `output_config={"effort": "..."}` explicitly - especially when moving Sonnet 4.5 -> Sonnet 4.6 (4.6 defaults to `high`)
      **[TUNE]** 显式设置 `output_config={"effort": "..."}`——尤其是从 Sonnet 4.5 迁到 Sonnet 4.6 时（4.6 默认为 `high`）
- [ ] **[TUNE]** Remove GA beta headers: `effort-2025-11-24`, `fine-grained-tool-streaming-2025-05-14`, `token-efficient-tools-2025-02-19`, `output-128k-2025-02-19`; remove `interleaved-thinking-2025-05-14` once on adaptive thinking
      **[TUNE]** 移除已 GA 的 beta 头：`effort-2025-11-24`、`fine-grained-tool-streaming-2025-05-14`、`token-efficient-tools-2025-02-19`、`output-128k-2025-02-19`；上了自适应思考后移除 `interleaved-thinking-2025-05-14`
- [ ] **[TUNE]** Switch `client.beta.messages.create(...)` -> `client.messages.create(...)` once all betas are removed
      **[TUNE]** 所有 beta 都移除后，把 `client.beta.messages.create(...)` 换成 `client.messages.create(...)`
- [ ] **[TUNE]** Review system prompt for aggressive tool language (`CRITICAL:`, `MUST`, `If in doubt`) and dial it back
      **[TUNE]** 审查系统提示词中的激进工具措辞（`CRITICAL:`、`MUST`、`If in doubt`）并收敛

**Extra items when coming from 3.x / 4.0 / 4.1:**
**来自 3.x / 4.0 / 4.1 时的额外条目：**
- [ ] **[BLOCKS]** Remove either `temperature` or `top_p` (passing both 400s on Claude 4+)
      **[BLOCKS]** 移除 `temperature` 或 `top_p` 之一（同时传两者在 Claude 4+ 上返回 400）
- [ ] **[BLOCKS]** Update text-editor tool `type` to `text_editor_20250728`
      **[BLOCKS]** 把文本编辑器工具的 `type` 更新为 `text_editor_20250728`
- [ ] **[BLOCKS]** Update text-editor tool `name` to `str_replace_based_edit_tool` - **changing only the `type` and keeping `name: "str_replace_editor"` returns a 400**
      **[BLOCKS]** 把文本编辑器工具的 `name` 更新为 `str_replace_based_edit_tool`——**只改 `type` 而保留 `name: "str_replace_editor"` 会返回 400**
- [ ] **[BLOCKS]** Update code-execution tool to `code_execution_20260521`
      **[BLOCKS]** 把代码执行工具更新为 `code_execution_20260521`
- [ ] **[BLOCKS]** Delete any `undo_edit` command call sites
      **[BLOCKS]** 删除所有 `undo_edit` 命令调用点
- [ ] **[TUNE]** Add handling for `stop_reason == "refusal"`
      **[TUNE]** 为 `stop_reason == "refusal"` 添加处理
- [ ] **[TUNE]** Add handling for `stop_reason == "model_context_window_exceeded"` (4.5+)
      **[TUNE]** 为 `stop_reason == "model_context_window_exceeded"` 添加处理（4.5+）
- [ ] **[TUNE]** Verify tool-param string matching tolerates trailing newlines (preserved on 4.5+)
      **[TUNE]** 核实工具参数字符串匹配能容忍末尾换行（4.5+ 上保留）
- [ ] **[TUNE]** If moving to Haiku 4.5: review rate-limit tier (separate pool from Haiku 3.x)
      **[TUNE]** 若迁移到 Haiku 4.5：审查限流档位（独立于 Haiku 3.x 的池）

**Verification:**
**验证：**
- [ ] Run one test request and inspect `response.stop_reason`, `response.usage`, and whether tool-use / thinking behavior matches expectations
      运行一次测试请求，检查 `response.stop_reason`、`response.usage`，以及工具使用 / 思考行为是否符合预期

For cached prompts: the render order and hash inputs did not change, so existing `cache_control` breakpoints keep working. However, **changing the model string invalidates the existing cache** - the first request on the new model will write the cache fresh.

对缓存的提示词：渲染顺序与哈希输入没有变化，因此现有 `cache_control` 断点继续有效。不过，**更改模型字符串会使现有缓存失效**——新模型上的第一个请求会重新写入缓存。
---

## Migrating to Opus 4.7 / 迁移到 Opus 4.7

> **Model ID `claude-opus-4-7` is authoritative as written here.** When the user asks to migrate to Opus 4.7, write `model="claude-opus-4-7"` exactly. Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entry exists in `shared/models.md`.

> **模型 ID `claude-opus-4-7` 以此处所写为准。**当用户要求迁移到 Opus 4.7 时，一字不差地写 `model="claude-opus-4-7"`。**不要** WebFetch 去核实——本指南就是迁移目标 ID 的事实来源。对应条目存在于 `shared/models.md` 中。

【评论】要求"不要 WebFetch 核实、以本文件为准"，把迁移目标 ID 的权威来源收口到技能自带文件，属于防幻觉设计；其代价是文件内容过期时，错误会被强制沿用。

Claude Opus 4.7 was Anthropic's most capable model at its launch and is now the previous-generation Opus (Opus 4.8 is current - see Migrating to Opus 4.8 below). It is highly autonomous and performs exceptionally well on long-horizon agentic work, knowledge work, vision tasks, and memory tasks. This section summarizes everything that was new at the 4.7 launch and remains the layered breaking-change path for callers coming from Opus 4.6 or older. It is layered on top of the 4.6 migration above - if the caller is jumping from Opus 4.5 or older, apply the 4.6 changes first, then this section, then the 4.8 section.

Claude Opus 4.7 是发布时 Anthropic 能力最强的模型，现在是上一代 Opus（当前为 Opus 4.8——见下文"迁移到 Opus 4.8"）。它高度自主，在长程智能体工作、知识工作、视觉任务与记忆任务上表现出色。本节总结 4.7 发布时的全部新内容，并且仍是来自 Opus 4.6 或更早的调用方的分层破坏性变更路径。它叠加在上文 4.6 迁移之上——如果调用方从 Opus 4.5 或更早跳过来，先应用 4.6 变更，再应用本节，然后是 4.8 节。

**TL;DR for someone already on Opus 4.6:** update the model ID to `claude-opus-4-7`, strip any remaining `budget_tokens` and sampling parameters (both 400 on Opus 4.7), give `max_tokens` extra headroom and re-baseline with `count_tokens()` against the new model, opt back into `thinking.display: "summarized"` if reasoning is surfaced to users, and re-tune `effort` - it matters more on 4.7 than on any prior Opus.

**对已在 Opus 4.6 上的用户，TL;DR 如下：**把模型 ID 更新为 `claude-opus-4-7`，剥掉残余的 `budget_tokens` 与采样参数（两者在 Opus 4.7 上都返回 400），给 `max_tokens` 留出额外余量并用 `count_tokens()` 对新模型重新定基，如果推理内容会呈现给用户则重新选入 `thinking.display: "summarized"`，并重新调优 `effort`——它在 4.7 上比任何先前的 Opus 都更重要。

### Breaking changes (will 400 on Opus 4.7) / 破坏性变更（在 Opus 4.7 上会返回 400）

**Extended thinking removed.**

**扩展思考已移除。**

`thinking: {type: "enabled", budget_tokens: N}` is no longer supported on Claude Opus 4.7 or later models and returns a 400 error. Switch to adaptive thinking (`thinking: {type: "adaptive"}`) and use the effort parameter to control thinking depth. Adaptive thinking is **off by default** on Claude Opus 4.7: requests with no `thinking` field run without thinking, matching Opus 4.6 behavior. Set `thinking: {type: "adaptive"}` explicitly to enable it.

`thinking: {type: "enabled", budget_tokens: N}` 在 Claude Opus 4.7 及之后的模型上不再受支持，会返回 400 错误。改用自适应思考（`thinking: {type: "adaptive"}`），并用 effort 参数控制思考深度。自适应思考在 Claude Opus 4.7 上**默认关闭**：不含 `thinking` 字段的请求将在无思考状态下运行，与 Opus 4.6 行为一致。要启用需显式设置 `thinking: {type: "adaptive"}`。

```python
# Before (Opus 4.6)
client.messages.create(
    model="claude-opus-4-6",
    max_tokens=64000,
    thinking={"type": "enabled", "budget_tokens": 32000},
    messages=[{"role": "user", "content": "..."}],
)

# After (Opus 4.7)
client.messages.create(
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # or "max", "xhigh", "medium", "low"
    messages=[{"role": "user", "content": "..."}],
)
```

If the caller wasn't using extended thinking, no change is required - thinking is off by default, or can be set explicitly with `thinking={"type": "disabled"}`.

如果调用方没有使用扩展思考，则无需改动——思考默认关闭，也可用 `thinking={"type": "disabled"}` 显式设置。

Delete `budget_tokens` plumbing entirely. For the replacement `effort` value, see **Choosing an effort level on Opus 4.7** below - there is no exact 1:1 mapping from `budget_tokens`.

把 `budget_tokens` 的整套管线彻底删除。替代的 `effort` 取值见下文**在 Opus 4.7 上选择 effort 级别**——从 `budget_tokens` 没有精确的一一对应。

**Sampling parameters removed.**

**采样参数已移除。**

The `temperature`, `top_p`, and `top_k` parameters are no longer accepted on Claude Opus 4.7. Requests that include them return a 400 error. Remove these fields from your request payloads. Prompting is the recommended way to guide model behavior on Claude Opus 4.7. If you were using `temperature = 0` for determinism, note that it never guaranteed identical outputs on prior models.

`temperature`、`top_p` 与 `top_k` 参数在 Claude Opus 4.7 上不再被接受，包含它们的请求返回 400 错误。从请求载荷中移除这些字段。在 Claude Opus 4.7 上，提示词是引导模型行为的推荐方式。如果你之前用 `temperature = 0` 换取确定性，请注意它在先前模型上也从未保证过完全相同的输出。

```python
# Before - errors on Opus 4.7
client.messages.create(temperature=0.7, top_p=0.9, ...)

# After
client.messages.create(...)  # no sampling params
```

- **If the intent was determinism** - use `effort: "low"` with a tighter prompt.
  **如果意图是确定性**——用 `effort: "low"` 配更紧凑的提示词。
- **If the intent was creative variance** - the prompt replacement depends on the use case; **ask the user** how they want variance elicited. If you can't ask, add a use-case-appropriate instruction along the lines of *"choose something off-distribution and interesting"* - e.g. for text generation, *"Vary your phrasing and structure across responses"*; for frontend/design, use the propose-4-directions approach under **Design and frontend coding** below.
  **如果意图是创意变化**——提示词替代取决于用例；**询问用户**希望如何引出变化。如果不能询问，按用例添加近似 *"choose something off-distribution and interesting"*（"选一个分布之外且有趣的"）的指令——例如文本生成用 *"Vary your phrasing and structure across responses"*（"在各次回复间变换措辞与结构"）；前端/设计用下文 **Design and frontend coding** 中的"提出 4 个方向"法。

### Choosing an effort level on Opus 4.7 / 在 Opus 4.7 上选择 effort 级别

`budget_tokens` controlled how much to *think*; `effort` controls how much to think *and* act, so there is no exact 1:1 mapping. **Use `xhigh` for best results in coding and agentic use cases, and a minimum of `high` for most intelligence-sensitive use cases.** Experiment with other levels to further tune token usage and intelligence:

`budget_tokens` 控制*思考*多少；`effort` 控制思考*和*行动的量，因此没有精确的一一对应。**编码与智能体用例用 `xhigh` 效果最佳；多数对智能敏感的用例至少用 `high`。**可试验其他级别进一步调整 token 用量与智能：

| Level | Use when | Notes |
| --- | --- | --- |
| `max` | Intelligence-demanding tasks worth testing at the ceiling | Can deliver gains in some use cases but may show diminishing returns from increased token usage; can be prone to overthinking |
| `xhigh` | **Most coding and agentic use cases** | The best setting for these; used as the default in Claude Code |
| `high` | Intelligence-sensitive use cases generally | Balances token usage and intelligence; recommended minimum for most intelligence-sensitive work |
| `medium` | Cost-sensitive use cases that need to reduce token usage while trading off intelligence | |
| `low` | Short, scoped tasks and latency-sensitive workloads that are not intelligence-sensitive | |

| 级别 | 何时使用 | 说明 |
| --- | --- | --- |
| `max` | 值得在顶格测试的高智能需求任务 | 在某些用例能带来增益，但随 token 用量增加可能收益递减；容易过度思考 |
| `xhigh` | **大多数编码与智能体用例** | 这些场景的最佳设置；Claude Code 中的默认值 |
| `high` | 一般的对智能敏感用例 | 在 token 用量与智能之间取得平衡；多数对智能敏感工作的推荐下限 |
| `medium` | 需要以牺牲部分智能换取更低 token 用量的成本敏感用例 | |
| `low` | 短小、范围明确的任务，以及对延迟敏感、对智能不敏感的工作负载 | |

### Silent default changes (no error, but behavior differs) / 静默默认值变化（不报错，但行为不同）

**Thinking content omitted by default.**

**思考内容默认省略。**

Thinking blocks still appear in the response stream on Claude Opus 4.7, but their `thinking` field is empty unless you explicitly opt in. This is a silent change from Claude Opus 4.6, where the default was to return summarized thinking text. To restore summarized thinking content on Claude Opus 4.7, set `thinking.display` to `"summarized"`. **The block-field name is unchanged** - it is still `block.thinking` on a `thinking`-type block; do not rename it.

在 Claude Opus 4.7 上，思考块仍会出现在响应流中，但除非显式选入，其 `thinking` 字段为空。这是相对 Claude Opus 4.6 的静默变化，后者的默认行为是返回摘要式思考文本。要在 Claude Opus 4.7 上恢复摘要式思考内容，把 `thinking.display` 设为 `"summarized"`。**块字段名未变**——`thinking` 类型块上仍是 `block.thinking`；不要改名。

**Detect this:** any code that reads `block.thinking` (or equivalent) from a `thinking`-type block and renders it in a UI, log, or trace. **The fix is the request parameter, not the response handling** - add `display: "summarized"` to the `thinking` parameter:

**如何发现：**任何从 `thinking` 类型块读取 `block.thinking`（或等价物）并在 UI、日志或 trace 中渲染的代码。**修复在请求参数，而非响应处理**——在 `thinking` 参数中加 `display: "summarized"`：

```python
thinking={"type": "adaptive", "display": "summarized"}  # "display" is new on Opus 4.7; values: "omitted" (default) | "summarized"
```

The default is `"omitted"` on Claude Opus 4.7. If thinking content was never surfaced anywhere, no change needed. If your product streams reasoning to users, the new default appears as a long pause before output begins; set `display: "summarized"` to restore visible progress during thinking.

Claude Opus 4.7 上默认值为 `"omitted"`。如果思考内容从未在任何地方呈现，无需改动。如果你的产品把推理流式呈现给用户，新默认表现为输出开始前的一段长停顿；设 `display: "summarized"` 以恢复思考期间的可见进展。

**Updated token counting.**

**token 计数已更新。**

Claude Opus 4.7 and Claude Opus 4.6 count tokens differently. The same input text produces a higher token count on Claude Opus 4.7 than on Claude Opus 4.6, and `/v1/messages/count_tokens` will return a different number of tokens for Claude Opus 4.7 than it did for Claude Opus 4.6. The token efficiency of Claude Opus 4.7 can vary by workload shape. Prompting interventions, `task_budget`, and `effort` can help control costs and ensure appropriate token usage. Keep in mind that these controls may trade off model intelligence. **Update your `max_tokens` parameters to give additional headroom, including compaction triggers.** Claude Opus 4.7 provides a 1M context window at standard API pricing with no long-context premium.

Claude Opus 4.7 与 Claude Opus 4.6 的 token 计数方式不同。同样的输入文本在 Claude Opus 4.7 上产生更高的 token 数，且 `/v1/messages/count_tokens` 对 Claude Opus 4.7 返回的 token 数也与 Claude Opus 4.6 不同。Claude Opus 4.7 的 token 效率随工作负载形态而变。提示词干预、`task_budget` 与 `effort` 有助于控制成本并保证合理的 token 用量。注意这些控制可能以模型智能为代价。**更新 `max_tokens` 参数以留出额外余量，包括压缩触发器。**Claude Opus 4.7 以标准 API 定价提供 1M 上下文窗口，无长上下文溢价。

What else to check:

还要检查什么：

- Client-side token estimators (tiktoken-style approximations) calibrated against 4.6
  针对 4.6 校准过的客户端 token 估算器（tiktoken 式近似）
- Cost calculators that multiply tokens by a fixed per-token rate
  用固定单价乘以 token 数的成本计算器
- Rate-limit retry thresholds keyed to measured token counts
  以实测 token 数为键的限流重试阈值

Re-baseline by re-running `client.messages.count_tokens()` against `claude-opus-4-7` on a representative sample of the caller's prompts. Do not apply a blanket multiplier. For cost-sensitive workloads, consider reducing `effort` by one level (e.g. `high` -> `medium`). For agentic loops, consider adopting Task Budgets (below).

对调用方提示词的代表性样本，对 `claude-opus-4-7` 重跑 `client.messages.count_tokens()` 来重新定基。不要套一刀切的乘数。对成本敏感的工作负载，考虑把 `effort` 降一级（如 `high` -> `medium`）。对智能体循环，考虑采用 Task Budgets（下文）。

### New feature: Task Budgets (beta) / 新特性：Task Budgets（beta）

Opus 4.7 introduces **task budgets** - tell Claude how many tokens it has for a full agentic loop (thinking + tool calls + final output). The model sees a running countdown and uses it to prioritize work and wrap up gracefully as the budget is consumed.

Opus 4.7 引入 **task budgets**（任务预算）——告诉 Claude 一整个智能体循环（思考 + 工具调用 + 最终输出）有多少 token 可用。模型会看到实时递减的倒计时，并据此排列工作优先级，随预算消耗体面收尾。

This is a **suggestion the model is aware of**, not a hard cap. It is distinct from `max_tokens`, which remains the enforced per-response limit and is *not* surfaced to the model. Use `task_budget` when you want the model to self-moderate; use `max_tokens` as a hard ceiling to cap usage.

这是**模型知晓的建议**，不是硬上限。它与 `max_tokens` 不同，后者仍是被强制执行的每响应限额，且*不会*呈现给模型。想让模型自我节制就用 `task_budget`；想硬性封顶用量就用 `max_tokens`。

Requires beta header `task-budgets-2026-03-13`:

需要 beta 头 `task-budgets-2026-03-13`：

```python
client.beta.messages.create(
    betas=["task-budgets-2026-03-13"],
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={
        "effort": "high",
        "task_budget": {"type": "tokens", "total": 128000},
    },
    messages=[...],
)
```

Set a generous budget for open-ended agentic tasks and tighten it for latency-sensitive ones. **Minimum `task_budget.total` is 20,000 tokens.** If the budget is too restrictive for the task, the model may complete it less thoroughly, referencing its budget as the constraint. **Do not add `task_budget` during a migration unless you are sure the budget value is right** - if you can run the workload and measure, do so; otherwise ask the user for the value rather than guessing. This is the primary lever for offsetting the token-counting shift on agentic workloads.

开放式智能体任务给宽裕的预算，对延迟敏感的任务收紧。**`task_budget.total` 最小为 20,000 token。**若预算对任务过紧，模型可能完成得不够彻底，并把预算当作约束来引用。**迁移期间不要添加 `task_budget`，除非你确信预算取值正确**——如果能运行工作负载并测量，就这么做；否则向用户询问取值，不要猜。这是在智能体工作负载上抵消 token 计数偏移的首要杠杆。

### Capability improvements / 能力提升

**High-resolution vision.** Opus 4.7 is the first Claude model with high-resolution image support. Maximum image resolution is **2576 pixels on the long edge** (up from 1568px on Opus 4.6 and prior). This unlocks gains on vision-heavy workloads, especially computer use and screenshot/artifact/document understanding. Coordinates returned by the model now map 1:1 to actual image pixels, so no scale-factor math is needed.

**高分辨率视觉。**Opus 4.7 是首个支持高分辨率图像的 Claude 模型。最大图像分辨率为**长边 2576 像素**（此前 Opus 4.6 及更早为 1568px）。这为视觉密集型工作负载解锁了增益，尤其是计算机使用与截图/产物/文档理解。模型返回的坐标现在与实际图像像素一一对应，无需再做缩放因子换算。

High-res support is **automatic on Opus 4.7** - no beta header, no client-side opt-in required. The model accepts larger inputs and returns pixel-accurate coordinates out of the box.

高分辨率支持在 Opus 4.7 上**自动生效**——无需 beta 头，无需客户端选入。模型开箱即接受更大输入并返回像素级精确的坐标。

**Token cost.** Full-resolution images on Opus 4.7 can use up to ~3× more image tokens than on prior models (up to ~4784 tokens per image, vs. the previous ~1,600-token cap). If the extra fidelity isn't needed, downsample client-side before sending to control cost - but **do not add downsampling by default during a migration**. If you're not sure whether the pipeline needs the fidelity, ask the user rather than guessing. Use `count_tokens()` on representative images on Opus 4.7 to re-baseline before reacting to any measured cost shift.

**token 成本。**Opus 4.7 上全分辨率图像耗用的图像 token 可达先前模型的约 3 倍（每张最高约 4784 token，此前上限约 1,600 token）。如果不需要这份额外保真度，发送前在客户端降采样以控制成本——但**迁移期间不要默认添加降采样**。若不确定管线是否需要该保真度，询问用户而非猜测。在 Opus 4.7 上对代表性图像用 `count_tokens()` 重新定基，再对测得的成本偏移做出反应。

Beyond resolution, Opus 4.7 also improves on low-level perception (pointing, measuring, counting) and natural-image bounding-box localization and detection.

除分辨率外，Opus 4.7 还改进了底层感知（指点、测量、计数）与自然图像的边界框定位与检测。

**Knowledge work.** Meaningful gains on tasks where the model visually verifies its own output - `.docx` redlining, `.pptx` editing, and programmatic chart/figure analysis (e.g. pixel-level data transcription via image-processing libraries). If prompts have scaffolding like *"double-check the slide layout before returning"*, try removing it and re-baselining.

**知识工作。**在模型以视觉校验自身输出的任务上有显著增益——`.docx` 批注修订、`.pptx` 编辑，以及程序化图表/图形分析（例如经图像处理库做像素级数据转录）。如果提示词中有 *"double-check the slide layout before returning"*（"返回前再检查一遍幻灯片布局"）之类的脚手架，试着移除并重新定基。

**Memory.** Opus 4.7 is better at writing and using file-system-based memory. If an agent maintains a scratchpad, notes file, or structured memory store across turns, that agent should improve at jotting down notes to itself and leveraging its notes in future tasks.

**记忆。**Opus 4.7 更擅长写入与使用基于文件系统的记忆。如果某个智能体跨回合维护草稿板、笔记文件或结构化记忆库，它在随手给自己记笔记以及在未来任务中利用这些笔记上应有提升。

**User-facing progress updates.** Opus 4.7 provides more regular, higher-quality interim updates during long agentic traces. If the system prompt has scaffolding like *"After every 3 tool calls, summarize progress"*, try removing it to avoid excessive user-facing text. If the length or contents of Opus 4.7's updates are not well-calibrated to your use case, explicitly describe what these updates should look like in the prompt and provide examples.

**面向用户的进度更新。**Opus 4.7 在长程智能体轨迹中提供更规律、质量更高的中期更新。如果系统提示词中有 *"After every 3 tool calls, summarize progress"*（"每 3 次工具调用后总结进度"）之类的脚手架，试着移除以避免过多的面向用户文本。如果 Opus 4.7 更新的长度或内容与你的用例不匹配，在提示词中明确描述这些更新应有的样子并给出示例。

### Real-time cybersecurity safeguards / 实时网络安全防护

Requests that involve prohibited or high-risk topics may lead to refusals.

涉及禁止或高风险主题的请求可能导致拒答。

### Fast Mode: Claude Opus 5.5 / Claude Opus 5 / Opus 4.8 only / Fast Mode：仅限 Claude Opus 5.5 / Claude Opus 5 / Opus 4.8

Fast mode is available on Claude Opus 5.5, Claude Opus 5, and Opus 4.8. Only surface this if the caller's code actually uses fast mode (e.g. `model="claude-opus-4-6-fast"`, or `speed="fast"` on an unsupported model); if the word "fast" does not appear in the code, say nothing about Fast Mode.

Fast mode 在 Claude Opus 5.5、Claude Opus 5 与 Opus 4.8 上可用。只有当调用方代码确实使用 fast mode（如 `model="claude-opus-4-6-fast"`，或在不受支持的模型上设 `speed="fast"`）时才提及；如果代码中没有出现 "fast" 一词，对 Fast Mode 只字不提。

When you see `model="claude-opus-4-6-fast"` (or any retired `-fast` model string), **the migration edit is** to move the fast-mode traffic onto Claude Opus 5.5, the current fast-capable default, at $8 / $40 per MTok (Claude Opus 5 and Opus 4.8 also work if the caller is staying on that tier):

看到 `model="claude-opus-4-6-fast"`（或任何已退役的 `-fast` 模型字符串）时，**迁移编辑是**把 fast mode 流量移到 Claude Opus 5.5——当前支持 fast 的默认模型，单价 $8 / $40 每 MTok（如果调用方停留在那一档，Claude Opus 5 与 Opus 4.8 也可用）：

```python
# Request fast mode on Claude Opus 5.5.
client.beta.messages.create(
    model="claude-opus-5-5", max_tokens=4096,
    speed="fast", betas=["fast-mode-2026-02-01"],
    messages=[...],
)
```

That is: switch the model to Claude Opus 5.5 (or Claude Opus 5 or Opus 4.8) and request fast mode the supported way, using the beta `client.beta.messages....` endpoint, the `fast-mode-2026-02-01` beta flag, and `speed="fast"` as a top-level request parameter (per-language form in SKILL.md § Fast Mode). Opus 4.7 fast mode has also been removed, so do not land on Opus 4.7 either. Do **not** leave the code on a retired `-fast` model string - the failure mode differs by version: `claude-opus-4-6-fast` is retired and the API **silently falls back** to standard Opus 4.6 (no error - the caller loses fast-mode speed without noticing); `claude-opus-4-7-fast` and `speed="fast"` on Opus 4.7 instead return an **API error** (hard failure - requests break outright rather than degrading). Either way, migrate to a supported fast-mode model (Claude Opus 5.5 by default) now.

也就是说：把模型切换到 Claude Opus 5.5（或 Claude Opus 5、Opus 4.8），并以受支持的方式请求 fast mode——使用 beta 的 `client.beta.messages....` 端点、`fast-mode-2026-02-01` beta 标志，以及作为顶层请求参数的 `speed="fast"`（各语言写法见 SKILL.md § Fast Mode）。Opus 4.7 的 fast mode 也已移除，所以也不要落在 Opus 4.7 上。**不要**把代码留在已退役的 `-fast` 模型字符串上——失败模式因版本而异：`claude-opus-4-6-fast` 已退役，API 会**静默回退**到标准 Opus 4.6（无报错——调用方在不知不觉中失去 fast mode 速度）；`claude-opus-4-7-fast` 及 Opus 4.7 上的 `speed="fast"` 则返回 **API 错误**（硬失败——请求直接中断而非降级）。无论哪种情况，现在就迁移到受支持的 fast mode 模型（默认 Claude Opus 5.5）。
### Behavioral shifts (prompt-tunable) / 行为转变（可通过提示词调优）

These don't break anything, but prompts tuned for Opus 4.6 may land differently. Opus 4.7 is more steerable than 4.6, so small prompt nudges usually close the gap.

这些不会弄坏任何东西，但为 Opus 4.6 调优的提示词可能产生不同效果。Opus 4.7 比 4.6 更可引导，小幅提示词点拨通常就能补齐差距。

**More literal instruction following.** Claude Opus 4.7 interprets prompts more literally and explicitly than Claude Opus 4.6, particularly at lower effort levels. It will not silently generalize an instruction from one item to another, and it will not infer requests you didn't make. The upside of this literalism is precision and less thrash. It generally performs better for API use cases with carefully tuned prompts, structured extraction, and pipelines where you want predictable behavior. A prompt and harness review may be especially helpful for migration to Claude Opus 4.7.

**更字面化的指令遵循。**Claude Opus 4.7 对提示词的解释比 Claude Opus 4.6 更字面、更就事论事，在较低 effort 档位尤甚。它不会把一条指令默默地从一项推广到另一项，也不会推断你没有提出的请求。这种字面化的好处是精确、更少反复。对于提示词经过精心调优的 API 用例、结构化抽取以及需要可预测行为的管线，它通常表现更好。一次提示词与框架（harness）审查对迁移到 Claude Opus 4.7 可能格外有帮助。

**Verbosity calibrates to task complexity.** Opus 4.7 scales response length to how complex it judges the task to be, rather than defaulting to a fixed verbosity - shorter answers on simple lookups, much longer on open-ended analysis. If the product depends on a particular length or style, tune the prompt explicitly. To reduce verbosity:

**冗长度随任务复杂度校准。**Opus 4.7 按自己对任务复杂度的判断伸缩回复长度，而不是默认固定冗长度——简单查询答案更短，开放式分析长得多。如果产品依赖特定的长度或风格，显式调优提示词。要降低冗长度：

> *"Provide concise, focused responses. Skip non-essential context, and keep examples minimal."*
> *"提供简洁、聚焦的回复。跳过非必要的上下文，示例从简。"*

If you see specific kinds of over-verbosity (e.g. over-explaining), add instructions targeting those. Positive examples showing the desired level of concision tend to be more effective than negative examples or instructions telling the model what not to do. Do **not** assume existing "be concise" instructions should be removed - test first.

如果看到特定种类的过度冗长（如过度解释），添加针对这些的指令。展示所需简洁程度的正面示例，往往比反面示例或告诉模型不要做什么的指令更有效。**不要**想当然地认为现有的"be concise"（保持简洁）类指令应当删除——先测试。

**Tone and writing style.** Opus 4.7 is more direct and opinionated, with less validation-forward phrasing and fewer emoji than Opus 4.6's warmer style. As with any new model, prose style on long-form writing may shift. If the product relies on a specific voice, re-evaluate style prompts against the new baseline. If a warmer or more conversational voice is wanted, specify it:

**语气与写作风格。**Opus 4.7 更直接、更有主见，相比 Opus 4.6 更温暖的风格，先给予认同的措辞更少、emoji 更少。与任何新模型一样，长文写作的文风可能变化。如果产品依赖特定的声音，对照新基线重新评估风格提示词。如果想要更温暖、更对话化的声音，明确指定：

> *"Use a warm, collaborative tone. Acknowledge the user's framing before answering."*
> *"使用温暖、协作的语气。回答之前先认可用户的表述。"*

**`effort` matters more than on any prior Opus.** Opus 4.7 respects `effort` levels more strictly, especially at the low end. At `low` and `medium` it scopes work to what was asked rather than going above and beyond - good for latency and cost, but on moderate tasks at `low` there is some risk of under-thinking.

**`effort` 比在任何先前 Opus 上都更重要。**Opus 4.7 对 `effort` 档位的遵循更严格，低档尤甚。在 `low` 和 `medium`，它把工作范围限定在所问内容上而不额外发挥——对延迟和成本有利，但在 `low` 档处理中等复杂度任务时存在思考不足的一定风险。

- If shallow reasoning shows up on complex problems, raise `effort` to `high` or `xhigh` rather than prompting around it.
  若复杂问题出现浅层推理，把 `effort` 提到 `high` 或 `xhigh`，而不是绕着它调提示词。
- If `effort` must stay `low` for latency, add targeted guidance: *"This task involves multi-step reasoning. Think carefully through the problem before responding."*
  若为延迟计 `effort` 必须保持 `low`，添加针对性指导：*"This task involves multi-step reasoning. Think carefully through the problem before responding."*（"本任务涉及多步推理。回应前把问题仔细想透。"）
- **At `xhigh` or `max`, set a large `max_tokens`** so the model has room to think and act across tool calls and subagents. Start at 64K and tune from there. (`xhigh` is a new effort level on Opus 4.7, between `high` and `max`.)
  **在 `xhigh` 或 `max`，设置较大的 `max_tokens`**，让模型在工具调用与子代理之间有思考和行动的空间。从 64K 起步再调。（`xhigh` 是 Opus 4.7 上新增的 effort 档位，介于 `high` 与 `max` 之间。）

Adaptive-thinking triggering is also steerable. If the model thinks more often than wanted - which can happen with large or complex system prompts - add: *"Thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multi-step reasoning. When in doubt, respond directly."*

自适应思考的触发也可引导。如果模型思考得比预期频繁——在系统提示词庞大或复杂时可能发生——添加：*"Thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multi-step reasoning. When in doubt, respond directly."*（"思考会增加延迟，只在能显著提升答案质量时使用——通常是要求多步推理的问题。拿不准就直接作答。"）

**Uses tools less often by default.** Opus 4.7 tends to use tools less often than 4.6 and to use reasoning more. This produces better results in most cases, but for products that rely on tools (search/retrieval, function-calling, computer-use steps), it can drop tool-use rate. Two levers:

**默认较少使用工具。**Opus 4.7 倾向于比 4.6 更少使用工具、更多使用推理。多数情况下这带来更好的结果，但对依赖工具的产品（搜索/检索、函数调用、计算机使用步骤），它可能拉低工具使用率。两个杠杆：

- **Raise `effort`** - `high` or `xhigh` show substantially more tool usage in agentic search and coding, and are especially useful for knowledge work.
  **提高 `effort`**——在智能体搜索与编码中，`high` 或 `xhigh` 的工具使用量明显更多，对知识工作尤其有用。
- **Prompt for it** - be explicit in tool descriptions or the system prompt about when and how to use the tool, and encourage the model to err on the side of using it more often:
  **用提示词引导**——在工具描述或系统提示词中明确何时以及如何使用该工具，并鼓励模型宁可多用：

> *"When the answer depends on information not present in the conversation, you MUST call the `search` tool before answering - do not answer from prior knowledge."*
> *"当答案依赖对话中不存在的信息时，回答前必须调用 `search` 工具——不要凭既有知识作答。"*

**Fewer subagents by default.** Opus 4.7 tends to spawn fewer subagents than 4.6. This is steerable - give explicit guidance on when delegation is desirable. For a coding agent, for example:

**默认更少子代理。**Opus 4.7 倾向于比 4.6 派生更少的子代理。这可以引导——就何时应当委派给出明确指导。例如对编码智能体：

> *"Do NOT spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see). Spawn multiple subagents in the same turn when fanning out across items or reading multiple files."*
> *"能在单次回复中直接完成的工作（例如重构一个你已经看得到的函数），不要派生子代理。跨多项展开或读取多个文件时，在同一回合派生多个子代理。"*

**Design and frontend coding.** Opus 4.7 has stronger design instincts than 4.6, with a consistent default house style: warm cream/off-white backgrounds (around `#F4F1EA`), serif display type (Georgia, Fraunces, Playfair), italic word-accents, and a terracotta/amber accent. This reads well for editorial, hospitality, and portfolio briefs, but will feel off for dashboards, dev tools, fintech, healthcare, or enterprise apps - and it appears in slide decks as well as web UIs.

**设计与前端编码。**Opus 4.7 的设计直觉比 4.6 更强，带有一致的默认家族风格：暖米白/乳白背景（约 `#F4F1EA`）、衬线展示字体（Georgia、Fraunces、Playfair）、斜体词缀点，以及陶土/琥珀色强调。这种风格适合编辑、酒店与作品集类需求，但对仪表盘、开发工具、金融科技、医疗或企业应用会显得不搭——而且它同样出现在幻灯片与 Web UI 中。

【评论】把"默认视觉风格"写进迁移文档，说明该模型的审美默认值相当稳定：泛泛的风格指令难以产生渐进扰动，只能整体切换。这属于对生成物风格一致性的工程观察。

The default is persistent. Generic instructions ("don't use cream," "make it clean and minimal") tend to shift the model to a different fixed palette rather than producing variety. Two approaches work reliably:

这个默认是顽固的。泛泛的指令（"别用米白"、"做得干净极简"）往往只是把模型切到另一套固定配色，而不是产生多样性。两种做法可靠：

1. **Specify a concrete alternative.** The model follows explicit specs precisely - give exact hex values, typefaces, and layout constraints.
   **指定一个具体的替代方案。**模型会精确遵循明确的规格——给出确切的十六进制色值、字体与布局约束。
2. **Have the model propose options before building.** This breaks the default and gives the user control:
   **让模型在构建前提出方案选项。**这能打破默认并交还用户控制权：

   > *"Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface - one-line rationale). Ask the user to pick one, then implement only that direction."*
   > *"构建之前，按本需求提出 4 个各不相同的视觉方向（每个格式为：背景色值 / 强调色值 / 字体——一行理由）。让用户挑一个，然后只实现该方向。"*

If the caller previously relied on `temperature` for design variety, use approach (2) - it produces meaningfully different directions across runs.

如果调用方此前靠 `temperature` 获得设计多样性，用做法 (2)——它能在多次运行间产生实质性不同的方向。

Opus 4.7 also requires less frontend-design prompting than previous models to avoid generic "AI slop" aesthetics. Where earlier models needed a lengthy anti-slop snippet, Opus 4.7 generates distinctive, creative frontends with a much shorter nudge. This snippet works well alongside the variety approaches above:

Opus 4.7 也比先前模型需要更少的前端设计提示词来避免千篇一律的"AI slop"美学。早期模型需要冗长的反套路片段，Opus 4.7 只需短得多的点拨就能生成有辨识度、有创意的前端。下面的片段与上述多样性做法配合良好：

> *"NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions."*
> *"绝不使用千篇一律的 AI 生成美学：被用滥的字体族（Inter、Roboto、Arial、系统字体）、俗套的配色（尤其是白色或深色背景上的紫色渐变）、可预测的布局与组件模式，以及缺乏语境个性的流水线设计。使用独特的字体、协调一致的配色与主题，并为主效果与微交互使用动画。"*

**Interactive coding products.** Opus 4.7's token usage and behavior can differ between autonomous, asynchronous coding agents with a single user turn and interactive, synchronous coding agents with multiple user turns. Specifically, it tends to use more tokens in interactive settings, primarily because it reasons more after user turns. This can improve long-horizon coherence, instruction following, and coding capabilities in long interactive coding sessions, but also comes with more token usage. To maximize both performance and token efficiency in coding products, use `effort: "xhigh"` or `"high"`, add autonomous features (like an auto mode), and reduce the number of human interactions required from users.

**交互式编码产品。**对"单个用户回合的自主异步编码智能体"与"多用户回合的交互式同步编码智能体"，Opus 4.7 的 token 用量与行为可能不同。具体而言，它在交互式场景下倾向于用更多 token，主要原因是它在每个用户回合后做更多推理。这可以提升长交互编码会话中的长程连贯性、指令遵循与编码能力，但也伴随更多 token 用量。要在编码产品中同时最大化性能与 token 效率，使用 `effort: "xhigh"` 或 `"high"`，添加自主特性（如 auto 模式），并减少需要用户介入的交互次数。

When limiting required user interactions, specify the task, intent, and relevant constraints upfront in the first human turn. Well-specified, clear, and accurate task descriptions upfront help maximize autonomy and intelligence while minimizing extra token usage after user turns - because Opus 4.7 is more autonomous than prior models, this usage pattern helps to maximize performance. In contrast, ambiguous or underspecified prompts conveyed progressively over multiple user turns tend to reduce token efficiency and sometimes performance.

在限制所需用户交互时，在第一个人类回合就前置说明任务、意图与相关约束。前置的、定义良好且准确的任务描述有助于最大化自主性与智能，同时把用户回合之后的额外 token 用量降到最低——由于 Opus 4.7 比先前模型更自主，这种使用模式有助于最大化性能。相反，跨多个用户回合逐步传达的含糊或欠定义提示词，往往降低 token 效率，有时也降低性能。

**Code review.** Opus 4.7 is meaningfully better at finding bugs than prior models, with both higher recall and precision. However, if a code-review harness was tuned for an earlier model, it may initially show *lower* recall - this is likely a harness effect, not a capability regression. When a review prompt says "only report high-severity issues," "be conservative," or "don't nitpick," Opus 4.7 follows that instruction more faithfully than earlier models did: it investigates just as thoroughly, identifies the bugs, and then declines to report findings it judges to be below the stated bar. Precision rises, but measured recall can fall even though underlying bug-finding has improved.

**代码审查。**Opus 4.7 找 bug 的能力明显强于先前模型，查全率与查准率都更高。不过，如果代码审查框架是为早期模型调优的，它起初可能表现出*更低*的查全率——这多半是框架效应，不是能力退化。当审查提示词说"只报告高严重度问题"、"保守一点"或"别吹毛求疵"时，Opus 4.7 比早期模型更忠实地遵循该指令：它照样彻底调查、照样找出 bug，然后拒报那些它判断低于所设门槛的发现。查准率上升，但测得的查全率可能下降，尽管底层的找 bug 能力提升了。

Recommended prompt language:

推荐的提示词措辞：

> *"Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them."*
> *"报告你发现的每一个问题，包括你不确定或认为严重度较低的。此阶段不要按重要性或置信度过滤——单独的验证步骤会做这件事。你在这个阶段的目标是覆盖面：让一个稍后被过滤掉的发现浮现，好过悄悄丢掉一个 bug。每条发现都附上置信度和估计严重度，便于下游过滤器排序。"*

This can be used without an actual second step, but moving confidence filtering out of the finding step often helps. If the harness has a separate verification/dedup/ranking stage, tell the model explicitly that its job at the finding stage is coverage, not filtering. If single-pass self-filtering is wanted, be concrete about the bar rather than using qualitative terms like "important" - e.g. *"report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences."* Iterate on prompts against a subset of evals to validate recall or F1 gains.

这条可以在没有真实第二步的情况下使用，但把置信度过滤从发现步骤中挪出去通常更有帮助。如果框架有单独的验证/去重/排序阶段，明确告诉模型它在发现阶段的职责是覆盖面，不是过滤。若要单遍自过滤，把门槛说具体，而不要用"重要"这类定性词——例如 *"report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences."*（"报告任何可能导致错误行为、测试失败或误导性结果的 bug；只省略纯样式或命名偏好之类的细枝末节。"）针对一部分评测迭代提示词，验证查全率或 F1 的增益。

**Computer use.** Computer use works across resolutions up to the new 2576px / 3.75MP maximum. Sending images at **1080p** provides a good balance of performance and cost. For particularly cost-sensitive workloads, **720p** or **1366×768** are lower-cost options with strong performance. Test to find the ideal settings for the use case; experimenting with `effort` can also help tune behavior.

**计算机使用。**计算机使用在最高 2576px / 3.75MP 的新上限内各分辨率下都能工作。以 **1080p** 发送图像是性能与成本的良好平衡。对成本特别敏感的工作负载，**720p** 或 **1366×768** 是性能依然强劲的低成本选项。请实际测试找到该用例的理想设置；试验 `effort` 也有助于调控行为。

---

## Opus 4.7 Migration Checklist / Opus 4.7 迁移清单

Every item is tagged: **`[BLOCKS]`** items cause a 400 error, infinite loop, silent truncation, or empty output if missed - apply these as code edits, not as suggestions. **`[TUNE]`** items are quality/cost adjustments - surface them to the user as recommendations.

每一项都有标记：**`[BLOCKS]`** 项一旦遗漏会导致 400 错误、无限循环、静默截断或空输出——把它们当作代码编辑来执行，而不是建议。**`[TUNE]`** 项是质量/成本调整——作为建议呈现给用户。

`[BLOCKS]` items prefixed with **"If..."** or **"At..."** are conditional. Before working through the list, **scan the file** for the conditions: does it surface thinking text to a UI/log? Does it set `output_config.effort` to `"x-high"` or `"max"`? Is it a security workload? Is it a multi-turn agentic loop? Apply only the items whose condition matches.

以 **"If..."** 或 **"At..."** 开头的 `[BLOCKS]` 项是条件项。逐条处理清单之前，先**扫描文件**看条件是否成立：它是否把思考文本呈现到 UI/日志？是否把 `output_config.effort` 设为 `"x-high"` 或 `"max"`？是否是安全类工作负载？是否是多回合智能体循环？只应用条件匹配的条目。

- [ ] **[BLOCKS]** Replace `thinking: {type: "enabled", budget_tokens: N}` with `thinking: {type: "adaptive"}` + `output_config.effort`; delete `budget_tokens` plumbing entirely
      **[BLOCKS]** 用 `thinking: {type: "adaptive"}` + `output_config.effort` 替换 `thinking: {type: "enabled", budget_tokens: N}`；把 `budget_tokens` 的整套管线彻底删除
- [ ] **[BLOCKS]** Strip `temperature`, `top_p`, `top_k` from request construction
      **[BLOCKS]** 从请求构造中剥掉 `temperature`、`top_p`、`top_k`
- [ ] **[BLOCKS]** If thinking content is surfaced to users or stored in logs: add `thinking.display: "summarized"` (otherwise the rendered text is empty)
      **[BLOCKS]** 若思考内容呈现给用户或写入日志：添加 `thinking.display: "summarized"`（否则渲染文本为空）
- [ ] **[BLOCKS]** At `output_config.effort` of `xhigh` or `max`: set `max_tokens` >= 64000 (otherwise output truncates mid-thought)
      **[BLOCKS]** 当 `output_config.effort` 为 `xhigh` 或 `max` 时：把 `max_tokens` 设为 >= 64000（否则输出在思考中途截断）
- [ ] **[TUNE]** Give `max_tokens` and compaction triggers extra headroom; re-run `count_tokens()` against `claude-opus-4-7` on representative prompts to re-baseline (no blanket multiplier)
      **[TUNE]** 给 `max_tokens` 与压缩触发器留出额外余量；对代表性提示词对 `claude-opus-4-7` 重跑 `count_tokens()` 重新定基（不要一刀切乘数）
- [ ] **[TUNE]** Re-baseline cost and rate-limit dashboards *before* reacting to measured shifts
      **[TUNE]** 在对测得的偏移做反应*之前*，重新定基成本与限流仪表盘
- [ ] **[TUNE]** Re-evaluate `effort` per route - use `xhigh` for coding/agentic and a minimum of `high` for most intelligence-sensitive work; it matters more on 4.7 than any prior Opus
      **[TUNE]** 按路由重新评估 `effort`——编码/智能体用 `xhigh`，多数对智能敏感的工作至少 `high`；它在 4.7 上比任何先前 Opus 都更重要
- [ ] **[TUNE]** Multi-turn agentic loops: adopt the API-native Task Budgets (`output_config.task_budget`, beta `task-budgets-2026-03-13`, minimum 20k tokens) - this is for capping *cumulative* spend across a loop; per-turn depth is `effort`
      **[TUNE]** 多回合智能体循环：采用 API 原生 Task Budgets（`output_config.task_budget`，beta `task-budgets-2026-03-13`，最小 20k token）——它用于封顶整个循环的*累计*花费；每回合的深度由 `effort` 控制
- [ ] **[TUNE]** Check for ambiguous or underspecified instructions that relied on 4.6 generalizing intent, and update them to be clearer or more precise - 4.7 follows them literally
      **[TUNE]** 检查那些依赖 4.6 自行推广意图的含糊或欠定义指令，把它们改得更清晰、更精确——4.7 会按字面执行
- [ ] **[TUNE]** Tool-use workloads: add explicit when/how-to-use guidance to tool descriptions (4.7 reaches for tools less often)
      **[TUNE]** 工具使用型工作负载：在工具描述中加入明确的何时/如何使用指导（4.7 较少主动用工具）
- [ ] **[TUNE]** Verbosity: test existing length instructions before changing them - 4.7 calibrates length to task complexity, so tune for the desired output rather than assuming a direction
      **[TUNE]** 冗长度：改动之前先测试现有长度指令——4.7 按任务复杂度校准长度，按期望输出调优，不要预设方向
- [ ] **[TUNE]** Remove forced-progress-update scaffolding (*"after every N tool calls..."*)
      **[TUNE]** 移除强制进度更新的脚手架（*"after every N tool calls..."*，即"每 N 次工具调用后……"）
- [ ] **[TUNE]** Remove knowledge-work verification scaffolding (*"double-check the slide layout..."*) and re-baseline
      **[TUNE]** 移除知识工作校验的脚手架（*"double-check the slide layout..."*，即"再检查一遍幻灯片布局……"）并重新定基
- [ ] **[TUNE]** Add tone instruction if a warmer / more conversational voice is needed; re-evaluate style prompts on writing-heavy routes
      **[TUNE]** 若需要更温暖/更对话化的声音，添加语气指令；在重写作的路由上重新评估风格提示词
- [ ] **[TUNE]** Subagent tool present: add explicit spawn / don't-spawn guidance
      **[TUNE]** 存在子代理工具时：添加明确的派生/不派生指导
- [ ] **[TUNE]** Frontend/design output: specify a concrete palette/typeface, or have the model propose 4 visual directions before building (the default cream/serif house style is persistent)
      **[TUNE]** 前端/设计输出：指定具体的配色/字体，或让模型在构建前提出 4 个视觉方向（默认的米白/衬线家族风格相当顽固）
- [ ] **[TUNE]** Interactive coding products: use `effort: "xhigh"` or `"high"`, add autonomous features (e.g. an auto mode) to reduce human interactions, and specify task/intent/constraints upfront in the first turn
      **[TUNE]** 交互式编码产品：使用 `effort: "xhigh"` 或 `"high"`，添加自主特性（如 auto 模式）以减少人工交互，并在第一回合前置说明任务/意图/约束
- [ ] **[TUNE]** Code-review harnesses: remove or loosen "only report high-severity" / "be conservative" filters and have the model report every finding with confidence + severity; move filtering to a downstream step (4.7 follows severity filters more literally, which can depress measured recall)
      **[TUNE]** 代码审查框架：移除或放宽"只报高严重度"/"保守一点"类过滤，让模型报告每一条发现并附置信度 + 严重度；把过滤挪到下游步骤（4.7 对严重度过滤的遵循更字面，可能压低测得的查全率）
- [ ] **[TUNE]** Vision-heavy pipelines (screenshots, charts, document understanding): leave images at native resolution up to 2576px long edge for the accuracy gain; remove any scale-factor math from coordinate handling (coords are now 1:1 with pixels). No beta header / opt-in needed - high-res is automatic on Opus 4.7.
      **[TUNE]** 视觉密集管线（截图、图表、文档理解）：图像保持原生分辨率、长边最高 2576px 以获得精度增益；从坐标处理中删掉一切缩放因子换算（坐标现在与像素 1:1）。无需 beta 头/选入——Opus 4.7 上高分辨率自动生效。
- [ ] **[TUNE]** Computer-use pipelines: send screenshots at 1080p for a good performance/cost balance (720p or 1366×768 for cost-sensitive workloads); experiment with `effort` to tune behavior
      **[TUNE]** 计算机使用管线：以 1080p 发送截图以取得良好的性能/成本平衡（成本敏感工作负载用 720p 或 1366×768）；试验 `effort` 调控行为
- [ ] **[TUNE]** Cost-sensitive image pipelines: full-res images on 4.7 use up to ~4784 tokens vs ~1,600 on prior models (~3×). Downsampling client-side before upload avoids the increase, but **do not downsample by default** - if you're unsure whether fidelity is needed, ask the user. Re-baseline with `count_tokens()` on representative images before reacting to cost shifts.
      **[TUNE]** 成本敏感的图像管线：4.7 上全分辨率图像最高耗用约 4784 token，而先前模型约 1,600（约 3 倍）。上传前在客户端降采样可避免增量，但**不要默认降采样**——不确定是否需要保真度时，询问用户。在对成本偏移做反应前，先对代表性图像用 `count_tokens()` 重新定基。
---

## Migrating to Opus 4.8 / 迁移到 Opus 4.8

> **Model ID `claude-opus-4-8` is authoritative as written here.** When the user asks to migrate to Opus 4.8, write `model="claude-opus-4-8"` exactly. Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entry exists in `shared/models.md`.

> **模型 ID `claude-opus-4-8` 以此处所写为准。**当用户要求迁移到 Opus 4.8 时，一字不差地写 `model="claude-opus-4-8"`。**不要** WebFetch 去核实——本指南就是迁移目标 ID 的事实来源。对应条目存在于 `shared/models.md` 中。

Claude Opus 4.8 is our most capable Opus-tier model - highly autonomous, with state-of-the-art long-horizon agentic execution, knowledge work, and memory. It is layered on top of the Opus 4.7 migration above. If the caller is jumping from Opus 4.6 or older, apply the 4.6 and 4.7 sections first, then this one.

Claude Opus 4.8 是我们最强的 Opus 档模型——高度自主，具备最先进的长程智能体执行、知识工作与记忆能力。它叠加在上文 Opus 4.7 迁移之上。如果调用方从 Opus 4.6 或更早跳过来，先应用 4.6 与 4.7 两节，再应用本节。

**No new breaking changes.** Opus 4.8 keeps the same request surface as Opus 4.7. The same calls that already work on 4.7 work unchanged on 4.8 - adaptive thinking only (`thinking: {type: "enabled", budget_tokens: N}` still 400s; use `{type: "adaptive"}`), sampling parameters (`temperature`, `top_p`, `top_k`) still rejected, last-assistant-turn prefills still 400, `thinking.display` still defaults to `"omitted"`, and the `low`/`medium`/`high`/`xhigh`/`max` effort levels, Task Budgets (beta), and high-resolution vision all behave as on 4.7. A 4.7 -> 4.8 migration is therefore **the model-ID swap plus prompt re-tuning** - there is no required code edit beyond the model string.

**无新增破坏性变更。**Opus 4.8 保有与 Opus 4.7 相同的请求面。在 4.7 上已能工作的调用在 4.8 上原样工作——仅支持自适应思考（`thinking: {type: "enabled", budget_tokens: N}` 仍返回 400；用 `{type: "adaptive"}`）、采样参数（`temperature`、`top_p`、`top_k`）仍被拒绝、最后一个 assistant 回合的 prefill 仍返回 400、`thinking.display` 仍默认 `"omitted"`，而 `low`/`medium`/`high`/`xhigh`/`max` 各 effort 档、Task Budgets（beta）与高分辨率视觉的行为都与 4.7 相同。因此 4.7 -> 4.8 迁移就是**换模型 ID 加提示词重新调优**——除模型字符串外没有必需的代码编辑。

**TL;DR for someone already on Opus 4.7:** swap the model ID to `claude-opus-4-8`. Nothing else is required to avoid an error. Then re-tune prompts for the behavioral shifts: 4.8 narrates *more* than 4.7 (add a silence-default if you want 4.7-like terseness), writes in a warmer, less hedged voice, is more deliberate and asks more often (add autonomy guidance to claw back ask-rate), and is more conservative about reaching for search, subagents, file-based memory, and custom tools (add explicit "when to use this" triggering). For long-horizon agentic work, give the full task specification up front in one well-specified turn and run at high effort.

**对已在 Opus 4.7 上的用户，TL;DR：**把模型 ID 换成 `claude-opus-4-8`。避免报错无需其他改动。然后针对行为转变重新调优提示词：4.8 比 4.7 叙述*更多*（想要 4.7 式的简洁就加一个静默默认），以更温暖、更少对冲的口吻写作，更审慎、提问更频繁（加自主性指导把提问率拉回来），并且对动用搜索、子代理、基于文件的记忆与自定义工具更保守（添加明确的"何时使用"触发条件）。对长程智能体工作，在一个定义良好的回合中前置给出完整任务规格，并以高 effort 运行。

### No new API breaking changes (inherited from 4.7) / 无新增 API 破坏性变更（继承自 4.7）

These all carry over from Opus 4.7 unchanged - apply them only if the caller is coming from Opus 4.6 or earlier (see the **Migrating to Opus 4.7** section above for the before/after and the SDK-specific syntax):

这些都自 Opus 4.7 原样继承——只有当调用方来自 Opus 4.6 或更早时才需应用（改动前后与各 SDK 语法见上文 **Migrating to Opus 4.7** 一节）：

- `thinking: {type: "enabled", budget_tokens: N}` -> 400. Use `thinking: {type: "adaptive"}` + `output_config.effort`.
  `thinking: {type: "enabled", budget_tokens: N}` -> 400。用 `thinking: {type: "adaptive"}` + `output_config.effort`。
- `temperature`, `top_p`, `top_k` -> 400. Remove them; steer with prompting.
  `temperature`、`top_p`、`top_k` -> 400。移除它们；用提示词引导。
- Last-assistant-turn prefills -> 400. Use `output_config.format` (structured outputs) or a system-prompt instruction.
  最后一个 assistant 回合的 prefill -> 400。用 `output_config.format`（结构化输出）或系统提示词指令。
- `thinking.display` defaults to `"omitted"`; set `"summarized"` if you surface reasoning to users.
  `thinking.display` 默认 `"omitted"`；若把推理呈现给用户，设 `"summarized"`。

If the caller is already on Opus 4.7 and these are clean, there is nothing to change here.

如果调用方已在 Opus 4.7 上且这些项都干净，这里没有要改的。

### New API feature: mid-session system prompts / 新 API 特性：会话中途系统提示词

You can deliver trusted instructions partway through a session by placing `{"role": "system", ...}` entries directly in the `messages` array - without editing the top-level system prompt and invalidating your prompt cache. Use it for things the application learns mid-session: the user delivered async context, a mode toggled (auto-approve enabled), files changed on disk, the remaining token budget dropped.

你可以把可信指令放在会话中段交付：直接在 `messages` 数组中放置 `{"role": "system", ...}` 条目——无需编辑顶层系统提示词、也不会使提示词缓存失效。用于应用在会话中途得知的事情：用户补送了异步上下文、某个模式切换（启用了自动批准）、磁盘上的文件变了、剩余 token 预算下降。

```python
messages=[
    {"role": "user", "content": [{"type": "tool_result", "tool_use_id": "...", "content": "..."}]},
    {"role": "system", "content": "This project's codebase is Go. Write code in Go."},
]
```

Phrase these as **context, not commands**. State the fact and let Claude act on it; avoid override-style language ("ignore what the user said", "regardless of the user's request", "disregard the previous instruction"). Claude is trained to protect users from instructions that appear to work against them, and that protection applies to the system role too. No beta header is required; available on Claude Opus 4.8. For cache-placement details and the older-model `<system-reminder>` fallback, see `shared/prompt-caching.md` and `shared/agent-design.md`.

把这些内容表述为**上下文，而非命令**。陈述事实，让 Claude 据此行动；避免覆盖式措辞（"ignore what the user said"、"regardless of the user's request"、"disregard the previous instruction"）。Claude 受过训练，会保护用户免受那些看似与其利益相悖的指令影响，这层保护同样适用于 system 角色。无需 beta 头；Claude Opus 4.8 上可用。缓存放置细节与旧模型的 `<system-reminder>` 回退方案，见 `shared/prompt-caching.md` 与 `shared/agent-design.md`。

【评论】要求"以上下文而非命令的口吻写系统消息"，是在顺应模型自身的安全对齐：模型被训练为抗拒看似与用户利益相悖的指令式内容，覆盖式措辞反而可能被拒绝执行。

### Capability improvements / 能力提升

**Long-horizon agentic execution.** Opus 4.8 is state-of-the-art at long, autonomous agentic work - complex refactors and overnight coding runs that complete without human correction. To get the most out of it, **give the full task specification up front in a single well-specified initial turn and run at high effort** (`effort: "high"` or `"xhigh"`). Its long-horizon coherence comes partly from reasoning more at each step; combined with a clear up-front goal, that more-intelligent planning often produces more efficient *and* more accurate output than prior frontier models. The "clear goal up front" principle maps to two product surfaces: in Claude Code, `/goal` sets direction for the run; with **Managed Agents (CMA)**, state what "done" looks like via an **Outcome** (`user.define_outcome` with a gradeable rubric - the harness runs an iterate -> grade -> revise loop), see `shared/managed-agents-outcomes.md`.

**长程智能体执行。**Opus 4.8 在长程自主智能体工作上是最先进的——无需人工纠错即可完成的复杂重构与通宵编码运行。要充分发挥，**在一个定义良好的初始回合中前置给出完整任务规格，并以高 effort 运行**（`effort: "high"` 或 `"xhigh"`）。它的长程连贯性部分来自每一步更多的推理；配合清晰的前置目标，这种更智能的规划常常产出比先前前沿模型更高效*也*更准确的输出。"前置清晰目标"原则对应两个产品面：在 Claude Code 中，`/goal` 为本次运行设定方向；在 **Managed Agents (CMA)** 中，通过 **Outcome** 说明"完成"长什么样（`user.define_outcome` 配可评分的量规——框架运行迭代 -> 评分 -> 修订循环），见 `shared/managed-agents-outcomes.md`。

**Effort is a dimension to test, not a fixed setting.** On prior models many reached for `xhigh` reflexively to maximize intelligence. Opus 4.8 has a higher intelligence ceiling, so **start at `high` as the default and iterate** rather than defaulting to `xhigh`. Sweep `medium`, `high`, and `xhigh` on your own eval set and weigh the intelligence <-> latency <-> cost tradeoff per route - the relationship isn't monotonic: higher effort up front often *reduces* turn count and total cost on agentic work, while for some tasks `medium` delivers equally good results in less time. Reserve `max` for extremely hard, latency-insensitive cases. The per-level effort table in the **Migrating to Opus 4.7** section above applies unchanged on 4.8.

**effort 是要测试的维度，不是固定设置。**在先前模型上，许多人条件反射地用 `xhigh` 最大化智能。Opus 4.8 的智能上限更高，所以**以 `high` 为默认起点并迭代**，而不是默认 `xhigh`。在你自己的评测集上扫 `medium`、`high`、`xhigh`，按路由权衡智能 <-> 延迟 <-> 成本的取舍——这层关系不是单调的：较高的前置 effort 在智能体工作上常常*降低*回合数与总成本，而某些任务用 `medium` 能以更少时间取得同样好的结果。`max` 留给极难、对延迟不敏感的情形。上文 **Migrating to Opus 4.7** 一节中的分档 effort 表在 4.8 上原样适用。

**Writing voice and clarity.** Testers consistently describe 4.8's prose as clearer, warmer, and less hedged than prior models, with fewer measurable AI vocal tics - especially at higher effort, where it approaches expert-level prose and structure. This is roughly the **opposite** direction from the 4.7 shift (4.7 was more clipped, direct, and less validation-forward). If you added style prompts to counter 4.7's terseness or to inject warmth, re-evaluate them against the new baseline before keeping them - they may now overcorrect. 4.8 is also a stronger thought partner: more thoughtful, more willing to push back, and more likely to infer the right answer from context.

**写作声音与清晰度。**测试者一致评价 4.8 的文字比先前模型更清晰、更温暖、更少对冲，可测得的 AI 口头禅更少——effort 更高时尤其如此，接近专家级行文与结构。这与 4.7 的转变方向大致**相反**（4.7 更简省、更直接、更少先认同）。如果你为对冲 4.7 的简省或注入温度而加了风格提示词，保留前先对照新基线重新评估——它们现在可能矫枉过正。4.8 也是更强的思考伙伴：更深思熟虑、更愿意反驳、更可能从上下文推断出正确答案。

**Code review and debugging.** Stronger real-bug finding and clearer explanations than 4.7 - one-shot fixes where 4.7 needed more, and correctly identifying intermittent flakes rather than declaring "fixed" after one clean run. The 4.7 caveat still applies: if a review harness says "only report high-severity issues" or "be conservative", 4.8 follows it literally and measured recall can drop even though underlying bug-finding improved. Tell the model to report everything and filter downstream (or review a second time) - see the **Code review** guidance in the 4.7 section for the recommended prompt.

**代码审查与调试。**真实 bug 的发现强于 4.7，解释更清晰——4.7 要改几轮的地方它能一次修好，还能正确识别间歇性 flake，而不是跑通一次就宣布"修好了"。4.7 的提醒仍然适用：如果审查框架说"只报高严重度问题"或"保守一点"，4.8 会按字面执行，测得的查全率可能下降，尽管底层找 bug 能力提升了。让模型报告一切、在下游过滤（或再审一遍）——推荐提示词见 4.7 节中的 **Code review** 指导。

### Behavioral shifts (prompt-tunable) / 行为转变（可通过提示词调优）

None of these break code, but prompts tuned for Opus 4.7 may land differently. 4.8 follows instructions well, so small, explicit nudges close the gap.

这些都不破坏代码，但为 Opus 4.7 调优的提示词可能产生不同效果。4.8 对指令遵循良好，小幅、明确的点拨即可补齐差距。

**Tool triggering is surface-dependent (search & knowledge).** 4.8's tool-triggering is more surface-dependent than in prior models: with a system prompt present it is high-precision / low-recall - web search triggers slightly more often but runs fewer rounds per trigger, while knowledge-retrieval tools (Drive, project knowledge, connected files) trigger *less* often. It searches when it's confident search is needed and otherwise answers from context, which can lower research depth on tasks that need it. Recover should-search rate with an explicit search-first instruction:

**工具触发依赖界面形态（搜索与知识）。**4.8 的工具触发比先前模型更依赖界面形态：有系统提示词在场时呈高查准/低查全——网页搜索触发略多但每次触发跑的轮数更少，而知识检索工具（Drive、项目知识、连接的文件）触发*更少*。它有把握需要搜索时才搜，否则就着上下文作答，这会降低需要深挖的任务的研究深度。用一个明确的"先搜索"指令把应搜率找回来：

> ```
> <search_first>
> For questions where current information would change the answer (recent events, current roles or prices, version-specific behavior, or anything the user flags as time-sensitive) search before answering rather than answering from memory. For open-ended research requests, begin searching immediately; do not ask a scoping question first unless the request is genuinely ambiguous about what to research.
> </search_first>
> ```

**Under-utilization of subagents, memory, and custom tools.** Separately from search, 4.8 is conservative about reaching for capabilities that need an explicit "decide to use this" step - file-based memory, subagent delegation, custom tools. It won't reach for complex or expensive capabilities unless reasonably sure they're needed. This is steerable since 4.8 follows instructions well - say *when* each capability applies, not just that it exists:

**子代理、记忆与自定义工具利用不足。**与搜索分开看，4.8 对动用那些需要显式"决定用它"一步的能力——基于文件的记忆、子代理委派、自定义工具——是保守的。除非相当确有必要，它不会动用复杂或昂贵的能力。这可以引导，因为 4.8 对指令遵循良好——说明每种能力*何时*适用，而不只说它存在：

> *"Before any task longer than a few turns, check your memory file for relevant prior context and write new findings to it as you go. When a task fans out across independent items (many files to read, many tests to run, many candidates to check), delegate to subagents rather than iterating serially."*
> *"在任何长于几个回合的任务之前，检查你的记忆文件中的相关先前上下文，并随做随写新发现。当任务跨独立事项展开（要读很多文件、要跑很多测试、要查很多候选）时，委派给子代理，而不是串行迭代。"*

The same lever works at the **tool-description** level, not just the system prompt: prescriptive descriptions that state *when* to call a tool (e.g. "Call this when the user asks about current prices or recent events") give meaningful lift on 4.8 over descriptions that only state what the tool does. Make the trigger condition part of each capability's own `description`.

同一个杠杆在**工具描述**层也有效，不只系统提示词：规定式描述写明*何时*调用工具（如"当用户问当前价格或近期事件时调用本工具"）在 4.8 上比只说工具做什么的描述有明显增益。把触发条件写进每种能力自己的 `description`。

**More user-facing narration.** 4.8 narrates more than 4.7 - more text between tool calls in long tool-calling sessions, and longer, more detailed end-of-task wrap-ups by default. If you previously added scaffolding to force interim status ("after every 3 tool calls, summarize progress"), **remove it** - 4.8 does this on its own. If the narration is too verbose for a coding agent, an explicit silence-default makes it behave like 4.7 with no loss of quality:

**更多面向用户的叙述。**4.8 比 4.7 叙述更多——长工具调用会话中工具调用之间的文字更多，任务收尾总结默认也更长更细。如果你此前加了强制中期汇报的脚手架（"每 3 次工具调用总结一次进度"），**移除它**——4.8 自己就会做。如果叙述对编码智能体而言太啰嗦，一个明确的静默默认能让它表现得像 4.7 且不损失质量：

> *"Default to silence between tool calls. Only write text when you find something, change direction, or hit a blocker - one sentence each. Do not narrate routine actions ('Now I'll...', 'Let me check...', 'Looking at...'). When done: one or two sentences on the outcome. Do not recap every file or test - the user has been following along."*
> *"工具调用之间默认静默。只有发现东西、改变方向或撞上阻塞时才写字——各一句。不要叙述例行动作（'现在我要……'、'让我看看……'、'正在看……'）。完成时：用一两句说明结果。不要复述每个文件或测试——用户一直在跟进。"*

For knowledge-work deliverables (reports, analysis readouts), verbosity responds very well to instructions in user preferences or the user turn - expose a verbosity preference rather than hard-coding a length.

对知识工作交付物（报告、分析汇报），冗长度对用户偏好或用户回合中的指令响应很好——暴露一个冗长度偏好，而不是把长度写死。

**More deliberate - asks more often.** 4.8 is more deliberate than prior Opus models. On minor decisions it would previously just make (a variable name, a default value, which of two equivalent approaches), it tends to pause and ask, and it often closes a completed task with "Want me to also...?" rather than doing the obvious next step or stopping cleanly. This is preferred for high-stakes or unfamiliar codebases, but bugs users when uncalibrated. Grant autonomy on the small stuff while keeping caution where it matters (in Claude Code testing this cut ask-rate by ~12 percentage points with no increase in over-reach):

**更审慎——更常提问。**4.8 比先前的 Opus 模型更审慎。在它此前会直接拿主意的小决定上（变量名、默认值、两个等价方案选哪个），它倾向于停下来问，并且常以"要不要我再……？"收尾一个已完成的任务，而不是做显而易见的下一步或干净利落地停下。对高风险或不熟悉的代码库这是优选，但不加校准时会烦扰用户。在小事情上给自主，在该谨慎处保持谨慎（在 Claude Code 测试中，这样把提问率降了约 12 个百分点且未增加越界）：

> *"For minor choices (naming, formatting, default values, which approach among equivalents), pick a reasonable option and note it rather than asking. For scope changes or destructive actions, still ask first."*
> *"对小选择（命名、格式、默认值、等价方案中选哪个），选一个合理的选项并注明，而不是发问。范围变更或破坏性操作，仍然先问。"*

**Verbose reasoning when thinking is disabled.** With `thinking: {type: "disabled"}`, 4.8 occasionally writes longer explanations of its reasoning into the visible response, which reads as verbose when the user wants a fast, quick answer. The simplest fix is to leave adaptive thinking on - set `thinking: {type: "adaptive"}` (the recommended setting; it adjusts how much to think per task). Note adaptive is **not** on when the field is omitted - like Opus 4.7, a request with no `thinking` field runs without thinking, so set it explicitly. If you need thinking off for latency or cost, scope it in the system prompt:

**关闭思考时的冗长推理。**在 `thinking: {type: "disabled"}` 下，4.8 偶尔把更长的推理解释写进可见响应，当用户想要快速利落的答案时显得啰嗦。最简单的修复是让自适应思考保持开启——设 `thinking: {type: "adaptive"}`（推荐设置；它按任务调整思考量）。注意省略该字段时自适应**并不**开启——与 Opus 4.7 一样，不含 `thinking` 字段的请求在无思考下运行，所以要显式设置。如果为延迟或成本需要关闭思考，在系统提示词中限定：

> *"Respond only with your final answer. Do not include exploratory reasoning, intermediate drafts, diffs you considered but rejected, or meta-commentary about your process."*
> *"只以最终答案回应。不要包含探索性推理、中间草稿、你考虑过但否决的 diff，或关于你过程的元评论。"*

### Opus 4.8 Migration Checklist / Opus 4.8 迁移清单

Every item is tagged: **`[BLOCKS]`** items cause a 400 error if missed; **`[TUNE]`** items are quality/cost adjustments - surface them to the user as recommendations.

每一项都有标记：**`[BLOCKS]`** 项一旦遗漏会导致 400 错误；**`[TUNE]`** 项是质量/成本调整——作为建议呈现给用户。

For a caller **already on Opus 4.7**, only the first item is required; everything else is `[TUNE]`. The conditional `[BLOCKS]` item applies only when coming from Opus 4.6 or earlier.

对**已在 Opus 4.7 上**的调用方，只有第一项是必需的；其余都是 `[TUNE]`。那条条件性 `[BLOCKS]` 项仅在来自 Opus 4.6 或更早时适用。

- [ ] **[BLOCKS]** Update the `model=` string to `claude-opus-4-8`
      **[BLOCKS]** 把 `model=` 字符串更新为 `claude-opus-4-8`
- [ ] **[BLOCKS]** *(only if coming from Opus 4.6 or earlier)* Apply the **Migrating to Opus 4.7** breaking changes first - `budget_tokens` -> adaptive thinking, strip `temperature`/`top_p`/`top_k`, remove last-assistant-turn prefills. These already 400 on 4.7 and continue to 400 on 4.8.
      **[BLOCKS]** *（仅当来自 Opus 4.6 或更早）*先应用 **Migrating to Opus 4.7** 的破坏性变更——`budget_tokens` 改自适应思考、剥掉 `temperature`/`top_p`/`top_k`、移除最后一个 assistant 回合的 prefill。这些在 4.7 上已返回 400，在 4.8 上继续返回 400。
- [ ] **[TUNE]** Long-horizon / agentic work: put the full task spec in one well-specified first turn and run at `high` or `xhigh` effort (Claude Code: `/goal`; Managed Agents: an Outcome with a gradeable rubric)
      **[TUNE]** 长程/智能体工作：把完整任务规格放进一个定义良好的首回合，并以 `high` 或 `xhigh` effort 运行（Claude Code：`/goal`；Managed Agents：带可评量规的 Outcome）
- [ ] **[TUNE]** Effort: sweep `medium` / `high` / `xhigh` on your eval set and pick per route by the intelligence <-> latency <-> cost tradeoff (default `high`, `xhigh` for coding/agentic)
      **[TUNE]** Effort：在评测集上扫 `medium` / `high` / `xhigh`，按智能 <-> 延迟 <-> 成本的取舍逐路由选择（默认 `high`，编码/智能体用 `xhigh`）
- [ ] **[TUNE]** Research depth & tool use: add a search-first instruction; add explicit triggering guidance for subagents, file-based memory, and custom tools (4.8 under-reaches for these by default) - in the system prompt *and* in each tool's own `description` (prescriptive "call this when..." descriptions give measurable lift)
      **[TUNE]** 研究深度与工具使用：加一条"先搜索"指令；为子代理、基于文件的记忆与自定义工具添加明确的触发指导（4.8 默认对这些不够主动）——系统提示词*和*每个工具自己的 `description` 里都要（规定式"何时调用本工具……"描述有可测增益）
- [ ] **[TUNE]** Narration: remove forced-progress scaffolding (*"after every N tool calls..."*); add a silence-default if a coding agent is too chatty
      **[TUNE]** 叙述：移除强制进度的脚手架（*"after every N tool calls..."*，即"每 N 次工具调用后……"）；编码智能体太话痨就加静默默认
- [ ] **[TUNE]** Autonomy: add small-decisions-don't-ask guidance to cut ask-rate, while keeping caution on scope changes / destructive actions
      **[TUNE]** 自主性：添加"小决定不发问"的指导以压低提问率，同时对范围变更/破坏性操作保持谨慎
- [ ] **[TUNE]** Writing voice: re-evaluate style prompts added to counter 4.7's directness - 4.8 is warmer and less hedged by default; re-baseline before keeping them
      **[TUNE]** 写作声音：重新评估为对冲 4.7 直率而加的风格提示词——4.8 默认更温暖、更少对冲；保留前先重新定基
- [ ] **[TUNE]** Code-review harnesses: keep the report-everything-filter-downstream pattern (4.8 follows "only high-severity" / "be conservative" filters literally, which can depress measured recall)
      **[TUNE]** 代码审查框架：保持"全报 + 下游过滤"模式（4.8 会按字面执行"只报高严重度"/"保守一点"过滤，可能压低测得的查全率）
- [ ] **[TUNE]** Thinking-disabled paths: add a final-answer-only instruction if reasoning leaks into the visible response
      **[TUNE]** 关闭思考的路径：若推理泄漏进可见响应，添加"只给最终答案"指令
- [ ] **[TUNE]** Consider mid-session system messages (`role:"system"` in `messages`; no beta header) for context the app learns mid-session, instead of rebuilding the top-level system prompt and invalidating the cache
      **[TUNE]** 对应用在会话中途得知的上下文，考虑用会话中途系统消息（`messages` 中的 `role:"system"`；无需 beta 头），而不是重建顶层系统提示词、使缓存失效
---

## Migrating to Claude Opus 5 / 迁移到 Claude Opus 5

> **Model ID `claude-opus-5` is authoritative as written here.** When the user asks to migrate to Claude Opus 5, write `model="claude-opus-5"` exactly. Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entry exists in `shared/models.md`.

> **模型 ID `claude-opus-5` 以此处所写为准。**当用户要求迁移到 Claude Opus 5 时，一字不差地写 `model="claude-opus-5"`。**不要** WebFetch 去核实——本指南就是迁移目标 ID 的事实来源。对应条目存在于 `shared/models.md` 中。

Claude Opus 5 is the successor to Claude Opus 4.8 in the Opus line, and is strongest on long-horizon agentic work and coding. It is layered on top of the Opus 4.8 migration above; if the caller is coming from Opus 4.7 or older, apply those sections first. Like Claude Fable 5.1, it ships with **elevated cybersecurity safeguards, and its safety classifiers can decline a request**: you get a normal HTTP 200 with `stop_reason: "refusal"` and a `stop_details` category, not an error. Benign security and life-sciences work occasionally trips them, so **check `stop_reason` before reading `response.content`** - code that indexes `content[0]` unconditionally breaks on a refusal. Cyber-category refusals route to Opus 4.8 as the recommended fallback, so a fallback strategy genuinely recovers the request rather than just relabelling the failure. The full refusal semantics (pre-output vs mid-stream billing, retry strategies, fallback credit) are in the Claude Fable 5.1 section below and apply here unchanged.

Claude Opus 5 是 Opus 线中 Claude Opus 4.8 的后继，在长程智能体工作与编码上最强。它叠加在上文 Opus 4.8 迁移之上；如果调用方来自 Opus 4.7 或更早，先应用那些小节。与 Claude Fable 5.1 一样，它带有**升级的网络安全防护，其安全分类器可以拒绝请求**：你得到的是正常的 HTTP 200，带 `stop_reason: "refusal"` 与 `stop_details` 类别，而不是错误。良性的安全与生命科学工作偶尔也会误触，所以**读取 `response.content` 之前先检查 `stop_reason`**——无条件索引 `content[0]` 的代码会在拒答时崩掉。网络类别的拒答按推荐回退路由到 Opus 4.8，因此回退策略能真正救回请求，而不只是给失败换个名目。完整的拒答语义（输出前与流中计费、重试策略、回退积分）在下文 Claude Fable 5.1 一节中，在此原样适用。

Existing prompts and evals should carry over with strong out-of-the-box performance. **It is a drop-in upgrade at Opus 4.8's pricing** - $5 per million input tokens, $25 per million output - with the same feature set: 1M context (default, no beta header), 128K max output, adaptive thinking, prompt caching, batch processing, the Files API, PDF support, vision, and the full server-side and client-side tool set. `claude-opus-5` is a fixed ID with no date suffix, same scheme as `claude-opus-4-8`.

现有提示词与评测应可平移，开箱即有强劲表现。**按 Opus 4.8 的定价，它是直接替换式升级**——输入每百万 $5、输出每百万 $25——特性集相同：1M 上下文（默认，无需 beta 头）、128K 最大输出、自适应思考、提示词缓存、批处理、Files API、PDF 支持、视觉，以及完整的服务端与客户端工具集。`claude-opus-5` 是无日期后缀的固定 ID，与 `claude-opus-4-8` 同一方案。

The migration is **the model-ID swap plus prompt re-tuning**, with two breaking changes covered below.

迁移就是**换模型 ID 加提示词重新调优**，另有两项破坏性变更，见下文。

**Availability at launch:** Claude API (`claude-opus-5`), Amazon Bedrock (`anthropic.claude-opus-5`), Google Cloud (`claude-opus-5`), and Microsoft Foundry. Opus 4.8 stays available on all four.

**发布时可用性：**Claude API（`claude-opus-5`）、Amazon Bedrock（`anthropic.claude-opus-5`）、Google Cloud（`claude-opus-5`）与 Microsoft Foundry。Opus 4.8 在这四处都继续可用。

**Rate limits are a separate bucket.** Opus 4.8/4.7/4.6/4.5 share one combined Opus limit; Claude Opus 5 does **not** draw from it. Shifting traffic over neither frees headroom on the old bucket nor inherits it - check your tier's Claude Opus 5 limits before moving volume.

**速率限制是独立的一桶。**Opus 4.8/4.7/4.6/4.5 共享一个合并的 Opus 限额；Claude Opus 5 **不**从中取用。把流量迁过去既不释放旧桶余量，也不继承它——搬量之前先查你档位的 Claude Opus 5 限制。

**TL;DR for someone already on Claude Opus 4.8:** swap the model ID. Then re-tune: Claude Opus 5 writes longer user-facing responses and longer files on disk (add explicit conciseness and deliverable-length instructions - `effort` does not reliably shorten visible output), verifies its own work without being told (**delete** your verification instructions and harness verification steps), and can expand task scope (add a scope-discipline instruction). Run a fresh effort sweep - `low` and `medium` are unusually strong here and are the primary cost/latency lever.

**对已在 Claude Opus 4.8 上的用户，TL;DR：**换模型 ID。然后重新调优：Claude Opus 5 写的面向用户响应更长、磁盘上的文件更长（加显式的简洁与交付物长度指令——`effort` 不能可靠地缩短可见输出），不经吩咐就会自查工作（**删除**你的校验指令与框架校验步骤），并且可能扩大任务范围（加范围纪律指令）。重新跑一轮 effort 扫描——`low` 与 `medium` 在这里出奇地强，是主要的成本/延迟杠杆。

### Breaking change 1: thinking is on by default / 破坏性变更 1：思考默认开启

A request that omits the `thinking` parameter **thinks** on Claude Opus 5, unlike Claude Opus 4.8 and Opus 4.7 where omitting it meant no thinking. `thinking: {type: "adaptive"}` remains valid and is equivalent to the default - the wire value didn't change, the default did.

在 Claude Opus 5 上，省略 `thinking` 参数的请求**会思考**——不同于 Claude Opus 4.8 与 Opus 4.7，在那里省略即不思考。`thinking: {type: "adaptive"}` 仍然有效且等价于默认——线上取值没变，变的是默认值。

This is a silent cost and truncation change, not just a behavior one: **`max_tokens` is a hard cap on thinking *plus* response text.** A workload that ran without thinking on Opus 4.8 and sized `max_tokens` tightly around its answer can now truncate mid-response. Revisit `max_tokens` on every route that never set `thinking`. To keep the old behavior, pass `thinking: {type: "disabled"}` - subject to the effort cap below.

这是一个静默的成本与截断变化，不只是行为变化：**`max_tokens` 是思考*加*响应文本的硬上限。**在 Opus 4.8 上无思考运行、把 `max_tokens` 按答案紧贴着配的工作负载，现在可能在响应中途截断。对每个从未设置 `thinking` 的路由重审 `max_tokens`。要保持旧行为，传 `thinking: {type: "disabled"}`——受下文 effort 上限约束。

Raw thinking tokens are **never returned** on Claude Opus 5; `display` defaults to `"omitted"`, and `display: "summarized"` gets you a summary. This also means a fallback model cannot read Claude Opus 5's thinking.

在 Claude Opus 5 上，原始思考 token **从不返回**；`display` 默认 `"omitted"`，`display: "summarized"` 可得摘要。这也意味着回退模型读不到 Claude Opus 5 的思考。

### Breaking change 2: disabling thinking is capped at `high` effort / 破坏性变更 2：禁用思考最高只到 `high` effort

Disabling thinking is available only at effort **`high` or lower**; `thinking: {type: "disabled"}` combined with `xhigh` or `max` returns a 400. Opus 4.8 accepts that combination, so audit any route that disables thinking before migrating.

禁用思考只在 effort **`high` 或更低**时可用；`thinking: {type: "disabled"}` 与 `xhigh` 或 `max` 组合会返回 400。Opus 4.8 接受该组合，所以迁移前审计每条禁用思考的路由。

**The check is per request.** Effort and thinking are validated independently on every call, so a later request that raises effort to `xhigh` while thinking is still disabled is rejected even though earlier requests in the same conversation succeeded.

**校验按请求进行。**effort 与思考在每次调用上独立校验，所以即便同一对话中先前的请求成功了，后面把 effort 提到 `xhigh` 而思考仍被禁用的请求也会被拒。

```python
# 400 on Claude Opus 5 - disabled thinking above `high`
client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    thinking={"type": "disabled"},
    output_config={"effort": "xhigh"},
    messages=[...],
)
```

**Migrating:** either enable thinking at `xhigh`/`max`, or lower effort to `high` or below. Given how well Claude Opus 5 performs at `low` and `medium`, a latency-sensitive route that previously ran `xhigh` + disabled thinking is usually better served by `medium` with thinking on than by keeping the disabled path.

**迁移：**要么在 `xhigh`/`max` 启用思考，要么把 effort 降到 `high` 或更低。鉴于 Claude Opus 5 在 `low` 与 `medium` 上的表现，一条此前跑 `xhigh` + 禁用思考的延迟敏感路由，通常更适合改成 `medium` 开思考，而不是守着禁用路径。

Everything else from the Opus 4.7/4.8 request surface is unchanged: `budget_tokens` still 400s (use `output_config.effort`), sampling parameters (`temperature`, `top_p`, `top_k`) are still rejected, last-assistant-turn prefills still 400, and `thinking.display` still defaults to `"omitted"`.

Opus 4.7/4.8 请求面的其余一切不变：`budget_tokens` 仍返回 400（用 `output_config.effort`）、采样参数（`temperature`、`top_p`、`top_k`）仍被拒、最后一个 assistant 回合的 prefill 仍返回 400、`thinking.display` 仍默认 `"omitted"`。

### Two failure modes when thinking is disabled / 禁用思考时的两种失败模式

**Are you affected?** Only if you explicitly set `thinking: {type: "disabled"}`. Thinking is on by default on Claude Opus 5 (see Breaking change 1 above), so an unmodified request never hits either of these - but code carrying a disabled-thinking setting forward from Opus 4.8, where it was the default behaviour, does.

**你会受影响吗？**只有当你显式设置 `thinking: {type: "disabled"}` 时。思考在 Claude Opus 5 上默认开启（见上文破坏性变更 1），所以未改动的请求不会碰到这两者——但从 Opus 4.8（在那里这是默认行为）把禁用思考设置一路带来的代码会。

Both are specific to `thinking: {type: "disabled"}` on Claude Opus 5, and for both the **primary recommendation is the same: turn thinking back on and use a lower `effort` to control cost and verbosity instead.** Disabling thinking is the more expensive lever in every sense - it is what triggers these, and `low`/`medium` effort already gets you most of the token and latency saving (see § Effort below).

两者都特定于 Claude Opus 5 上的 `thinking: {type: "disabled"}`，而且对两者的**首要建议相同：把思考开回来，改用更低的 `effort` 控制成本与冗长。**任何意义上，禁用思考都是更昂贵的杠杆——触发这些问题的正是它，而 `low`/`medium` effort 已能拿到大部分 token 与延迟节省（见下文 § Effort）。

**1. Tool calls can arrive as plain text.** The model occasionally writes a tool call into its user-facing text rather than emitting a structured `tool_use` block. **The turn completes normally and the call never runs** - there is no error and no `tool_use` block to catch, so a harness sees a successful turn that silently did nothing. Worse in an agentic loop: the bogus text stays in conversation history and skews later turns. Most common on tool-heavy workloads such as search.

**1. 工具调用可能以纯文本到达。**模型偶尔会把一次工具调用写进面向用户的文本，而不是发出结构化 `tool_use` 块。**回合正常完成，调用从未运行**——没有报错、也没有 `tool_use` 块可捕获，框架看到的是一个静默无所作为的"成功"回合。在智能体循环中更糟：假文本留在对话历史里，带偏后续回合。在搜索这类工具密集工作负载上最常见。

**2. `<thinking>` tags can leak into the visible response.** The model may emit `<thinking>` or other internal XML in its user-facing output.

**2. `<thinking>` 标签可能泄漏进可见响应。**模型可能在面向用户的输出中发出 `<thinking>` 或其他内部 XML。

If you cannot enable thinking, one instruction covers both failure modes - give the model explicit permission to talk before a tool call (the tool-as-text failure appears to come from suppressing the preamble it wants to write), and forbid internal tags generically:

如果无法启用思考，一条指令可覆盖两种失败模式——明确允许模型在工具调用前说句话（工具调用变文本的失败似乎源于压制了它想写的开场白），并一般性地禁止内部标签：

> *"When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response."*
> *"使用工具时，你可以先说一句简短的话。如果没有工具能表达用户所求，直说，不要猜。不要在你的响应中包含内部或系统 XML 标签。"*

Two counterintuitive rules for that instruction:

关于这条指令，有两个反直觉的规则：

- **Delete any instruction telling the model not to think or not to reason.** That kind of rule *increases* tag leakage rather than suppressing it.
  **删除任何叫模型不要思考、不要推理的指令。**那类规则反而*增加*标签泄漏，而非抑制。
- **Do not name thinking tags in the prompt.** Calling out `<thinking>` by name is measurably less effective than the generic "internal or system XML tags" wording above.
  **不要在提示词中点名思考标签。**直接点出 `<thinking>`，可测得地不如上文"内部或系统 XML 标签"的泛化措辞有效。

### New API features / 新 API 特性

Two additions, each behind its own beta header. Both are optional - a migrated request works without them.

两项新增，各自由自己的 beta 头把守。两者都可选——迁移后的请求没有它们也能工作。

**1. `fallbacks: "default"` - recommended for every caller.** Claude Opus 5's safety classifiers can decline a request; the `fallbacks` parameter re-runs a declined request on another model server-side instead of returning the refusal to you. Previously you named the substitute yourself (`"fallbacks": [{"model": "claude-opus-4-8"}]`). The new `"default"` mode picks Anthropic's recommended fallback automatically, routed **by refusal category** - cyber-category refusals go to Claude Opus 4.8.

**1. `fallbacks: "default"`——推荐每位调用方使用。**Claude Opus 5 的安全分类器可以拒绝请求；`fallbacks` 参数让被拒的请求由服务端在另一个模型上重跑，而不是把拒答返回给你。以前你要自己点名替代者（`"fallbacks": [{"model": "claude-opus-4-8"}]`）。新的 `"default"` 模式自动选择 Anthropic 推荐的回退，**按拒答类别**路由——网络类别的拒答交给 Claude Opus 4.8。

```http
POST /v1/messages
anthropic-beta: server-side-fallback-2026-07-01

{"model": "claude-opus-5", "fallbacks": "default", "max_tokens": 1024,
 "messages": [{"role": "user", "content": "Say OK."}]}
```

**Prefer `"default"` over pinning a model.** Different fallback models carry different classifiers, so the right substitute depends on *why* the request was declined - and `"default"` removes the migration you would otherwise owe when a pinned fallback model is deprecated. Note the header is `server-side-fallback-2026-07-01`, distinct from the `-2026-06-01` header that gates the array form; the array form's semantics (content blocks, `usage.iterations`, sticky routing) are unchanged and documented in the Claude Fable 5.1 refusal section below.

**优先用 `"default"`，不要钉死模型。**不同回退模型带不同的分类器，合适的替代取决于请求*为何*被拒——而且当被钉住的回退模型退役时，`"default"` 免去了你本要补的迁移。注意头是 `server-side-fallback-2026-07-01`，不同于把守数组形式的 `-2026-06-01` 头；数组形式的语义（content blocks、`usage.iterations`、粘性路由）不变，记录在下文 Claude Fable 5.1 拒答一节。

**2. Mid-conversation tool changes (beta `mid-conversation-tool-changes-2026-07-01`).** Change a conversation's tool set between turns without invalidating the prompt cache. Previously `tools` was fixed for the conversation's lifetime and any edit re-billed the whole prefix. Append a `{"role": "system", "content": [...]}` message carrying a `tool_addition` or `tool_removal` block:

**2. 会话中途工具变更（beta `mid-conversation-tool-changes-2026-07-01`）。**在回合之间变更对话的工具集，而不使提示词缓存失效。以前 `tools` 在对话生命周期内固定，任何修改都要为整个前缀重新计费。追加一条携带 `tool_addition` 或 `tool_removal` 块的 `{"role": "system", "content": [...]}` 消息：

```python
messages = [
    {"role": "user", "content": "What tools do you have for weather in Paris?"},
    {"role": "system", "content": [
        {"type": "tool_addition", "tool": {"type": "tool_reference", "name": "get_forecast"}},
    ]},
]
```

The added tool must already be declared in `tools[]` with `"defer_loading": True` - declared up front, but not loaded into context until a `tool_addition` surfaces it. A `tool_removal` block must sit either immediately before an assistant message or at the end of `messages`. To *change* a tool's definition, remove the old one on one request, then send the updated entry in `tools[]` on the next. See `shared/tool-use-concepts.md` § Mid-conversation tool changes.

新增的工具必须已在 `tools[]` 中以 `"defer_loading": True` 声明——预先声明，但在 `tool_addition` 让它露面之前不加载进上下文。`tool_removal` 块必须紧挨 assistant 消息之前，或位于 `messages` 末尾。要*更改*工具定义，先在一次请求中移除旧工具，再在下次请求的 `tools[]` 中发送更新后的条目。见 `shared/tool-use-concepts.md` § Mid-conversation tool changes。

> Warning: Earlier previews of this feature used a different beta header and different block shapes. Both are deprecated - if the code you're migrating carries anything other than `mid-conversation-tool-changes-2026-07-01` with `tool_addition` / `tool_removal` / `tool_reference`, update the header and the shapes together.

> 警告：该特性较早的预览版使用不同的 beta 头和不同的块形状。两者都已弃用——如果你正在迁移的代码携带 `mid-conversation-tool-changes-2026-07-01` 与 `tool_addition` / `tool_removal` / `tool_reference` 之外的任何东西，请把头与块形状一起更新。

> **SDK typings lag these blocks.** Pass them as plain dicts in Python (the SDK forwards unknown keys unchanged) or add a `@ts-expect-error` in TypeScript until the types catch up. `extra_body` / `extra_headers` work on `.stream()` exactly as on `.create()`.

> **SDK 类型定义滞后于这些块。**在 Python 中按普通 dict 传入（SDK 原样转发未知键），或在 TypeScript 中加 `@ts-expect-error`，直到类型跟上为止。`extra_body` / `extra_headers` 在 `.stream()` 上与在 `.create()` 上完全一样可用。
### Capability improvements / 能力提升

**Agentic coding.** Claude Opus 5 is a workhorse for agentic coding and is strongest on *difficult* tasks - multi-file features, larger refactors, end-to-end feature work. It completes tasks rather than leaving stubs or placeholders. The gap over prior models is smaller on easy single-turn edits, so evaluate it on the hard end of your workload. To get the most out of it, give the complete task specification up front and let it run; longer autonomous sessions with more parallel agents show the strongest results, short interactive edits the least.

**智能体编码。**Claude Opus 5 是智能体编码的主力，在*困难*任务上最强——多文件特性、较大重构、端到端特性开发。它把任务做完，而不是留下桩代码或占位符。在简单的单回合编辑上，对先前模型的领先幅度更小，所以要在你工作负载中难的那一端评估它。要充分发挥，前置给出完整任务规格然后放手让它跑；更长、并行智能体更多的自主会话结果最强，短促的交互式编辑最弱。

**Code review and bug-finding.** High precision *and* high recall - a high rate of real bugs per pass, with the extra findings mostly real rather than false positives. It stays accurate at lower effort, which makes a cheap fast pass at review time plus a thorough pass later a practical pattern.

**代码审查与找 bug。**高查准*且*高查全——每遍扫出真实 bug 的比率高，多出来的发现大多真实而非误报。它在较低 effort 下仍保持准确，这让"审查时便宜快速过一遍 + 之后再彻底过一遍"成为可行模式。

**Effort: the full ladder, and where to start.** Claude Opus 5 supports all five levels - `low`, `medium`, `high`, `xhigh`, `max` - with no beta header. The API default is `high`.

**Effort：完整阶梯与起点。**Claude Opus 5 支持全部五档——`low`、`medium`、`high`、`xhigh`、`max`——无需 beta 头。API 默认 `high`。

- **Start at `high` (the API default), then sweep down.** `low` and `medium` are unusually effective on this model - strong quality at a fraction of the tokens and latency on many workloads - so treat them as the primary cost/latency lever and reserve `high` and above for tasks where your evals show a quality difference. Effort defaults carried over from a prior model are usually not the right setting here; run a fresh sweep.
  **从 `high`（API 默认）起步，再向下扫。**`low` 与 `medium` 在这个模型上出奇地有效——在许多工作负载上以零头般的 token 与延迟拿下强劲质量——所以把它们当作主要的成本/延迟杠杆，`high` 及以上留给你的评测显示有质量差异的任务。从先前模型沿袭的 effort 默认值在这里通常不是正确设置；重新扫一轮。
- **`xhigh` and `max` are for measured wins, not a starting point.** `max` is the top tier for the deepest reasoning and worth testing where capability matters more than spend, but it can show diminishing returns and overthink simpler tasks.
  **`xhigh` 与 `max` 是给实测有赢面的场景，不是起点。**`max` 是最深推理的顶档，在能力比花费更重要的地方值得测试，但它可能收益递减，并把较简单的任务想过头。

At `xhigh` or `max`, **set a large `max_tokens`** so the model has room to think and act across tool calls and subagents. Start at 64K and tune.

在 `xhigh` 或 `max`，**设置较大的 `max_tokens`**，让模型在工具调用与子代理之间有思考和行动的空间。从 64K 起步再调。

**Lower prompt-cache minimum.** The minimum cacheable prompt is **512 tokens** on Claude Opus 5, down from 1024 on Opus 4.8. Prompts previously too short to cache now create entries with no code change - worth re-checking any prompt you'd written off as uncacheable. See `shared/prompt-caching.md`.

**提示词缓存门槛降低。**Claude Opus 5 上最小可缓存提示词为 **512 token**，Opus 4.8 为 1024。以前短到无法缓存的提示词现在不改代码即可生成缓存条目——值得复查你曾判定不可缓存的任何提示词。见 `shared/prompt-caching.md`。

**Fast mode.** `speed: "fast"` (beta header `fast-mode-2026-02-01`) is supported on Claude Opus 5, priced at $10 / $50 per MTok. It is a research preview on the **Claude API only** - including Managed Agents - and is **not** available on Amazon Bedrock, Google Cloud, or Microsoft Foundry. Fast mode draws on dedicated rate limits separate from the standard Opus pools.

**Fast mode。**`speed: "fast"`（beta 头 `fast-mode-2026-02-01`）在 Claude Opus 5 上受支持，定价 $10 / $50 每 MTok。它是**仅 Claude API** 上的研究预览——包括 Managed Agents——在 Amazon Bedrock、Google Cloud 或 Microsoft Foundry 上**不**可用。Fast mode 使用独立于标准 Opus 池的专用速率限制。

**Vision - give it tools, not more thinking.** Stronger on chart, document, and diagram understanding, and on UI and frontend visual replication. The highest-leverage change is **giving it tools to iteratively analyze, crop, and visually verify its own work**: on this model tool use is a markedly more cost-effective lever than raising thinking alone. Claude Opus 5 sits in the high-resolution tier alongside Opus 4.8 - 2576 px on the long edge, up to 4784 visual tokens per image - so coordinates map 1:1 to pixels and no scale-factor math is needed. Any prompt-side workaround you added for a prior model's vision limitations should be re-validated; several are now counterproductive.

**视觉——给它工具，而不是更多思考。**在图表、文档与示意图理解，以及 UI 与前端视觉复刻上更强。杠杆最高的改动是**给它工具去迭代分析、裁剪并视觉校验自己的工作**：在这个模型上，工具使用是比单纯提高思考明显更具成本效益的杠杆。Claude Opus 5 与 Opus 4.8 同属高分辨率档——长边 2576 px、每图最高 4784 视觉 token——因此坐标与像素 1:1 映射，无需缩放因子换算。你为先前模型视觉局限添加的任何提示词侧变通都应重新验证；有几条现在适得其反。

**Long context.** 1M-token context window as both the default *and* the maximum. Instruction following, tool calling, and reasoning stay strong across the full window.

**长上下文。**1M token 上下文窗口同时是默认*与*最大值。指令遵循、工具调用与推理在整个窗口内保持强劲。

**Office and document tasks.** Generates and edits complex multi-sheet Excel files with non-trivial formulas, and visually strong PowerPoint decks that follow slide-design best practices. It can be prompted to adhere to a specific style or template when one is required.

**Office 与文档任务。**能生成并编辑含非平凡公式的复杂多工作表 Excel 文件，以及遵循幻灯片设计最佳实践、视觉上出色的 PowerPoint 演示。在要求特定风格或模板时，可以通过提示词让它遵守。

**Multi-agent coordination.** Coordinates teams of subagents well - few cases of agents overwriting each other's work, and effective use of writer-verifier patterns. Workloads that benefit from multi-agent patterns are good fits. **Cost-sensitive workloads should cap multi-agent usage** - see the delegation section below, because this model reaches for subagents more readily than its predecessors.

**多智能体协调。**能很好地协调子代理团队——很少出现智能体互相覆盖工作的情况，且有效运用写者-校验者模式。受益于多智能体模式的工作负载是好标的。**成本敏感的工作负载应封顶多智能体用量**——见下文委派一节，因为这个模型比其前辈更轻易地动用子代理。

### Behavioral shifts (prompt-tunable) / 行为转变（可通过提示词调优）

**Longer user-facing responses.** Default response text is longer than on prior models. **`effort` is not the lever here** - changing it may move thinking volume without reliably changing visible output length. Prompting is: in testing, a short conciseness instruction cut user-facing response length by ~20%.

**更长的面向用户响应。**默认响应文本比先前模型更长。**`effort` 在这里不是杠杆**——改动它可能挪动思考量，却不能可靠地改变可见输出长度。提示词才是：测试中，一条简短的简洁指令把面向用户响应长度削了约 20%。

> *"Keep responses focused, brief, and concise to avoid overwhelming the person. Disclaimers and caveats are brief, with most of the response on the main answer; when asked to explain something, give a high-level summary unless an in-depth one is specifically requested."*
> *"保持回复聚焦、简短、精炼，以免压垮对方。免责与告戒从简，大部分篇幅留给主要答案；被要求解释某事时，先给高层摘要，除非明确要求深入。"*

For a long system prompt, pair that with a one-line reminder near the end:

对较长的系统提示词，在结尾附近配一行提醒：

> ```
> <tone_preference>
> Keep outputs reasonably concise.
> </tone_preference>
> ```

**More narration in agentic sessions** (the lever runs both ways - the same explicit-description technique tunes narration *up* or restyles it, if your product wants more). Claude Opus 5 narrates what it is about to do, and its per-message output in agentic sessions is longer than prior models'. It responds well to explicit guidance on *how* to communicate during a task rather than just *how much*. For coding agents, this block calibrates it:

**智能体会话中更多叙述**（这个杠杆双向可用——如果你的产品想要更多，同一套明确描述技术可以把叙述调*多*或改风格）。Claude Opus 5 会叙述它即将做什么，智能体会话中的每条消息输出比先前模型更长。它对"任务期间*如何*沟通"的明确指导响应良好，而不只是*多少*。对编码智能体，下面的块可为它校准：

> ```
> # Communicating with the user
> Your text output is what the user reads between tool calls; they usually can't see your thinking or the raw tool results. Write it for a teammate who stepped away and is catching up, not for a log file: they don't know the codenames or shorthand you created along the way, and they didn't watch your process unfold. Before your first tool call, say in a sentence what you're about to do; while working, give brief updates when you find something load-bearing or change direction.
>
> Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find" - the thing the user would ask for if they said "just give me the TLDR." Supporting detail and reasoning should come after, for readers who want them.
>
> Being readable and being concise are different things, and readable matters more. If the user has to reread your summary or ask you to explain, any time saved by brevity is gone. The way to keep output short is to be selective about what you include (drop details that don't change what the reader would do next), not to compress the writing into fragments, abbreviations, arrow chains like `A -> B -> fails`, or jargon. What you do include, write in complete sentences with the technical terms spelled out. Don't make the reader cross-reference labels or numbering you invented earlier; say what you mean in place.
>
> Match the response to the question: a simple question should be answered with a direct answer in prose, not headers and sections. Use tables only for short enumerable facts, with explanations in the surrounding prose rather than the cells. Calibrate to the user - a bit tighter for an expert, more explanatory for someone newer.
>
> Write code that reads like the surrounding code: match its comment density, naming, and idiom.
>
> Only write a code comment to state a constraint the code itself can't show - never to say where it came from, what the next line does, or why your change is correct; that's you talking to the reviewer, not the next reader, and it's noise the moment the PR merges.
> ```

**Longer written deliverables.** Separate from conversational verbosity: files Claude Opus 5 writes to disk - reports, Markdown documents, summaries - are often longer than on prior models. If your product ships Claude-authored documents, calibrate length explicitly:

**更长的书面交付物。**与对话冗长分开：Claude Opus 5 写到磁盘的文件——报告、Markdown 文档、总结——常比先前模型的长。如果你的产品发布 Claude 撰写的文档，显式校准长度：

> *"Match the length of written deliverables (especially Markdown files) to what the task needs: cover the substance, but do not pad documents with filler sections, redundant summaries, or boilerplate."*
> *"让书面交付物（尤其是 Markdown 文件）的长度匹配任务所需：覆盖实质内容，但不要用凑数章节、重复摘要或模板套话把文档灌水。"*

**Self-check instructions are the same trap.** Beyond harness scaffolding, per-prompt re-check phrasing - *"double-check your answer"*, *"re-verify before responding"* - triggers the same extra work. Note this **inverts a standard prompting best practice**: "ask Claude to self-check" is generally sound advice and is wrong here, so a prompt library that applies it uniformly needs a carve-out for this model rather than a global rule.

**自查指令是同一个坑。**除框架脚手架外，逐提示词的复查措辞——*"double-check your answer"*（"再检查一遍你的答案"）、*"re-verify before responding"*（"回应前再核一遍"）——触发同样的额外工作。注意这**颠倒了标准的提示词最佳实践**："让 Claude 自查"一般是有益的建议，在这里却不对，所以统一套用该建议的提示词库需要为这个模型开例外，而不是定为全局规则。

**Over-verification - delete your verification scaffolding.** Claude Opus 5 verifies its own work without being asked. Instructions that *tell* it to verify ("include a final verification step for virtually any non-trivial task", "use a subagent to verify") now cause over-verification. **Removing them reduces over-verification with no capability regression** - this is a delete, not a rewrite. The same applies to harness-level scaffolding: separate verification steps carried over from prior models are likely redundant now.

**过度验证——删掉你的校验脚手架。**Claude Opus 5 不用吩咐就会自查工作。*叫*它验证的指令（"为几乎任何非平凡任务加一个最终验证步骤"、"用子代理验证"）现在造成过度验证。**移除它们能减少过度验证且无能力退化**——这是删除，不是改写。框架层脚手架同理：从先前模型沿袭下来的独立验证步骤如今多半冗余。

**Task scope expansion.** It can add steps the user didn't request, or apply its own judgment about what the task should be without making that clear. In testing, this instruction reduced scope changes to nearly zero without producing excessive clarifying questions:

**任务范围扩张。**它可能添加用户没有要求的步骤，或按自己的判断决定任务该是什么而不言明。测试中，这条指令把范围变更降到接近零，且没有产生过多的澄清提问：

> *"Deliver what the user asked for, at the scope they intended. Interpret ambiguity the way a careful colleague would: make routine judgment calls yourself, and check in only when different readings would lead to materially different work. If you conclude the ask is mistaken or a better approach exists, say so in a sentence and keep going with the task as asked - don't quietly narrow, widen, or transform it. Finish the whole task, not just the easy part of it - only report completion when it's fully done. If you genuinely can't complete something, do the rest and state plainly what's missing and why. Stop short of actions or changes that are clearly beyond what the user's ask implies."*
> *"交付用户所求，范围如其本意。像一位细致的同事那样解读含糊之处：例行判断自己做主，只在不同的读法会导致实质性不同工作时才来确认。如果你断定这个要求有误或存在更好的做法，用一句话说明，然后继续按所求完成任务——不要悄悄收窄、扩大或改造它。完成整个任务，而不只是其中容易的部分——只有真正做完才报告完成。如果确实无法完成某部分，把其余做完，并直白说明缺什么、为什么。止步于明显超出用户所求含义的行动或改动。"*

The revised wording adds a **finish-the-whole-task** clause - report completion only when the work is actually done, and if something genuinely can't be finished, do the rest and say plainly what is missing. That covers premature "done" claims, which scope-discipline wording alone did not.

修订后的措辞加了**完成整个任务**条款——只有工作真正做完才报告完成，若确有无法完成的部分，做完其余并直白说明缺了什么。这覆盖了过早宣称"完成"的情况，单靠范围纪律措辞管不住这一点。

**Delegates to subagents more readily - the opposite of Opus 4.8.** This is a direction change worth flagging: Opus 4.8 *under*-reached for subagents and needed prompting to delegate. Claude Opus 5 reaches for them freely, which multiplies cost and latency - each subagent re-establishes context, re-explores, reports back, and then the coordinator re-reads the report. If your harness supports subagents, **any "delegate more" guidance you added for Opus 4.8 should come out**, and you likely want an explicit cap. A deterministic ceiling on spawn count is the reliable lever; this block reduces delegation and token spend:

**更轻易委派给子代理——与 Opus 4.8 相反。**这是一个值得标注的方向变化：Opus 4.8 对子代理*利用不足*，需要提示词才肯委派。Claude Opus 5 随手就派，这会放大成本与延迟——每个子代理要重建上下文、重新探索、汇报，协调者还要重读汇报。如果你的框架支持子代理，**你为 Opus 4.8 添加的任何"多委派"指导都应删掉**，而且你多半想要一个显式上限。对派生数量设确定性天花板是可靠的杠杆；下面的块能减少委派与 token 花费：

> ```
> ## Delegating to subagents
> Subagents multiply cost and time: each one re-establishes context, re-explores, and reports back, and you then re-read its report. Delegate rarely and only when the payoff clearly exceeds that overhead.
>
> Do use subagents for:
> - Large tasks that are genuinely independent and parallelizable. For example, wide multi-file investigations.
>
> Do NOT use subagents for:
> - Work you could finish yourself in a handful of tool calls. For example: a few file reads, a handful of edits, a simple search task, relatively simple verification.
> - Review, verification, or to double check your work. Verification belongs in your main agent loop.
>
> Use of parallel or multiple subagents:
> - Do not use multiple subagents on a single small task. Parallel subagents are for genuinely independent, sizeable tracks (unrelated modules, a wide multi-file investigation), not for splitting one modest job into pieces.
> - If the task can be completed with one subagent, choose one subagent over multiple subagents. Keep spawn counts low.
> - Never use more than 20 parallel agents unless the user explicitly requests it.
>
> When delegating to subagents:
> - Brief the subagent precisely the first time. Avoid launching, waiting, and re-briefing.
> - If you delegate, commit to the delegation. Never redo the subagent's work and do not re-derive its findings once it reports back.
> - If you launch multiple agents for independent work, send them in a single message with multiple tool uses so they run concurrently.
> ```

Note the interaction with over-verification below: "do not use subagents to verify" and "delete your verification scaffolding" are the same underlying fix seen from two angles.

注意它与下文过度验证的联动："不要用子代理验证"与"删掉你的校验脚手架"是同一个底层修复的两个视角。

**Narrates self-corrections more than prior models.** It flags and explains its own earlier mistakes at length, which reads as thrash in a user-facing product. Scope corrections to the ones that actually change the user's outcome:

**比先前模型更多叙述自我纠正。**它会详尽地指出并解释自己先前的错误，在面向用户的产品里读起来像反复折腾。把纠正限定在真正改变用户结果的那类：

> ```
> # Corrections
> Avoid unnecessary or excessive self-correction. Only correct an earlier statement in your user-facing text when the error would change the user's code, conclusions, or decisions. State corrections plainly and concisely, and continue the task; combine multiple corrections rather than enumerating them all. For slips that change nothing for the user, simply make the correction and move on - no need to note it explicitly. Don't add apologies or preambles, don't be overly self-critical, and don't ruminate or give a detailed account of the mistake or tally past errors. Sometimes, other agents will report incorrect or misleading results - don't always take them at face value immediately. If other agents correct your statements and they are right, then simply update your approach without narrating too much about the correction to the user. This instruction does not apply to thinking blocks.
>
> A follow-up question about your earlier work is not, by itself, a signal that you got something wrong - answer what was asked. A statement that was accurate needs no correction: don't re-audit how you phrased it, how you verified it, or limits you already stated. When the user does point to a real error, correct it plainly as above.
> ```

The second paragraph matters as much as the first: a plain follow-up question can otherwise trigger a re-audit of work that was correct.

第二段与第一段同样重要：否则一个普通的追问就可能触发对本来正确工作的重新审计。

**Time to first token (TTFT).** Claude Opus 5 sometimes thinks before its first visible block, which raises TTFT - a problem for user-facing chat and voice, where the pause reads as latency. This one-line instruction reduces pre-first-block thinking significantly:

**首 token 时间（TTFT）。**Claude Opus 5 有时会在首个可见块之前思考，抬高 TTFT——对面向用户的聊天与语音是个问题，因为停顿会被读作延迟。这条单行指令能显著减少首块前的思考：

> *"Latency-sensitive; begin your visible answer immediately."*
> *"对延迟敏感；立即开始输出可见答案。"*

Apply it only where first-token latency is user-visible; on background and agentic routes the pre-answer thinking is usually worth keeping.

只在首 token 延迟对用户可见之处应用；在后台与智能体路由上，答案前的思考通常值得保留。

**Severity filters still depress measured recall.** Unchanged from 4.7/4.8: if a review harness says "only report high-severity issues" or "be conservative", Claude Opus 5 follows it literally. Ask it to report everything with confidence and severity, and filter in a separate pass - see the **Code review** guidance in the Opus 4.7 section for the recommended prompt.

**严重度过滤仍会压低测得的查全率。**与 4.7/4.8 相同：如果审查框架说"只报高严重度问题"或"保守一点"，Claude Opus 5 会按字面执行。让它把一切连同置信度与严重度一起报告，并在单独一遍里过滤——推荐提示词见 Opus 4.7 节的 **Code review** 指导。
### Claude Opus 5 Migration Checklist / Claude Opus 5 迁移清单

**`[BLOCKS]`** items cause a 400 error if missed; **`[TUNE]`** items are quality/cost adjustments - surface them to the user as recommendations.

**`[BLOCKS]`** 项一旦遗漏会导致 400 错误；**`[TUNE]`** 项是质量/成本调整——作为建议呈现给用户。

- [ ] **[BLOCKS]** Update the `model=` string to `claude-opus-5`
      **[BLOCKS]** 把 `model=` 字符串更新为 `claude-opus-5`
- [ ] **[BLOCKS]** Any route combining `thinking: {type: "disabled"}` with `effort` of `xhigh` or `max`: enable thinking, or lower effort to `high` or below. Validated per request, so audit every call site, not just the first
      **[BLOCKS]** 任何把 `thinking: {type: "disabled"}` 与 `xhigh` 或 `max` effort 组合的路由：启用思考，或把 effort 降到 `high` 或更低。校验按请求进行，所以要审计每个调用点，不只是第一个
- [ ] **[BLOCKS]** Every route that never set `thinking`: it now thinks, and `max_tokens` caps thinking + response text together. Raise `max_tokens` or pass `thinking: {type: "disabled"}` at effort `high` or below - otherwise responses truncate mid-answer
      **[BLOCKS]** 每条从未设置 `thinking` 的路由：它现在会思考，且 `max_tokens` 一起封顶思考 + 响应文本。调高 `max_tokens`，或在 `high` 及以下 effort 传 `thinking: {type: "disabled"}`——否则响应会在答案中途截断
- [ ] **[BLOCKS]** *(only if coming from Opus 4.7 or earlier)* Apply the **Migrating to Opus 4.7** breaking changes first - `budget_tokens` -> adaptive thinking, strip `temperature`/`top_p`/`top_k`, remove last-assistant-turn prefills
      **[BLOCKS]** *（仅当来自 Opus 4.7 或更早）*先应用 **Migrating to Opus 4.7** 的破坏性变更——`budget_tokens` 改自适应思考、剥掉 `temperature`/`top_p`/`top_k`、移除最后一个 assistant 回合的 prefill
- [ ] **[TUNE]** Effort: start at `high` (the API default) and sweep down - `low`/`medium` are unusually strong on this model and are the primary cost/latency lever; reserve `xhigh`/`max` for tasks where you've measured a quality difference. Prior-model defaults rarely transfer. At `xhigh`/`max`, set `max_tokens` to at least 64K
      **[TUNE]** Effort：从 `high`（API 默认）起步并向下扫——`low`/`medium` 在这个模型上出奇地强，是主要的成本/延迟杠杆；`xhigh`/`max` 留给你实测出质量差异的任务。先前模型的默认值很少能直接沿用。在 `xhigh`/`max`，把 `max_tokens` 设为至少 64K
- [ ] **[TUNE]** Re-check prompts you'd written off as uncacheable - the minimum drops to 512 tokens (from 1024 on Opus 4.8)
      **[TUNE]** 复查你曾判定不可缓存的提示词——门槛降到 512 token（Opus 4.8 为 1024）
- [ ] **[TUNE]** Rate limits: Claude Opus 5 is a separate bucket from the combined Opus 4.x pool - confirm your tier's limits before shifting volume
      **[TUNE]** 速率限制：Claude Opus 5 独立于合并的 Opus 4.x 池——搬量之前确认你档位的限制
- [ ] **[TUNE]** Fast mode (`speed: "fast"`, `fast-mode-2026-02-01`, $10/$50) is Claude-API-only - drop it on Bedrock, Google Cloud, and Foundry routes
      **[TUNE]** Fast mode（`speed: "fast"`、`fast-mode-2026-02-01`、$10/$50）仅限 Claude API——在 Bedrock、Google Cloud 与 Foundry 路由上去掉
- [ ] **[TUNE]** Verbosity: add a conciseness instruction (and a `<tone_preference>` tag for long system prompts). Do **not** try to shorten output by lowering `effort` - it doesn't reliably work
      **[TUNE]** 冗长度：加一条简洁指令（长系统提示词再加一个 `<tone_preference>` 标签）。**不要**试图靠调低 `effort` 缩短输出——不可靠
- [ ] **[TUNE]** Agentic sessions: add a "Communicating with the user" block to calibrate inter-tool-call narration
      **[TUNE]** 智能体会话：加一个"Communicating with the user"块来校准工具调用之间的叙述
- [ ] **[TUNE]** Claude-authored files: add a deliverable-length instruction
      **[TUNE]** Claude 撰写的文件：加一条交付物长度指令
- [ ] **[TUNE]** **Delete** verification instructions from prompts and verification steps from the harness - including per-prompt *"double-check your answer"* phrasing, which inverts the usual self-check best practice on this model
      **[TUNE]** 从提示词中**删除**校验指令、从框架中删除校验步骤——包括逐提示词的 *"double-check your answer"* 措辞，它在这个模型上颠倒了通常的自查最佳实践
- [ ] **[TUNE]** Add the scope-discipline instruction if the model expands task scope
      **[TUNE]** 若模型扩张任务范围，加范围纪律指令
- [ ] **[TUNE]** Vision pipelines: re-validate prompt-side workarounds written for a prior model's vision limitations
      **[TUNE]** 视觉管线：重新验证为先前模型视觉局限所写的提示词侧变通
- [ ] **[TUNE]** Consider mid-conversation tool changes (`mid-conversation-tool-changes-2026-07-01`) - changes the tool set between turns without invalidating the prompt cache. Note per-turn `effort` / `task_budget` were **not** in this launch (per-message `effort` shipped later, beta `mid-conversation-output-config-2026-07-01`, and works on Claude Opus 5 too - see Migrating to Claude Fable 5.1 from Claude Fable 5 § New API features; `task_budget` stays request-level)
      **[TUNE]** 考虑会话中途工具变更（`mid-conversation-tool-changes-2026-07-01`）——在回合间变更工具集而不使提示词缓存失效。注意逐回合的 `effort` / `task_budget` **不在**本次发布中（逐消息的 `effort` 后来才发布，beta `mid-conversation-output-config-2026-07-01`，在 Claude Opus 5 上也可用——见"从 Claude Fable 5 迁移到 Claude Fable 5.1"§ New API features；`task_budget` 保持在请求级）
- [ ] **[TUNE]** Subagent-capable harnesses: this model delegates *more* readily than Opus 4.8 - remove any "delegate more" guidance you added for 4.8 and add an explicit cap
      **[TUNE]** 支持子代理的框架：这个模型委派比 Opus 4.8 *更*积极——删掉你为 4.8 加的任何"多委派"指导，并加一个显式上限
- [ ] **[TUNE]** User-facing products: add the corrections instruction if self-correction narration reads as thrash
      **[TUNE]** 面向用户的产品：若自我纠正叙述读起来像反复折腾，加纠正指令
- [ ] **[TUNE]** TTFT-sensitive routes (chat, voice): add *"Latency-sensitive; begin your visible answer immediately"* to reduce pre-first-block thinking; skip on background/agentic routes
      **[TUNE]** TTFT 敏感路由（聊天、语音）：加 *"Latency-sensitive; begin your visible answer immediately"*（"对延迟敏感；立即开始输出可见答案"）以减少首块前的思考；后台/智能体路由跳过
- [ ] **[TUNE]** Any route running `thinking: {type: "disabled"}`: prefer turning thinking on at `low`/`medium` effort. Disabled thinking can emit tool calls as plain text (the call silently never runs) and leak `<thinking>` tags into output. If you must stay thinking-off, delete any don't-think/don't-reason rule and add the combined *"When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response"* - do not name `<thinking>` tags in the prompt
      **[TUNE]** 任何跑 `thinking: {type: "disabled"}` 的路由：优先改为在 `low`/`medium` effort 上开启思考。禁用思考可能把工具调用当纯文本发出（调用静默地从未运行），并把 `<thinking>` 标签泄漏进输出。若必须保持关思考，删掉任何"不要思考/不要推理"规则，并加合并指令 *"When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response"*（"使用工具时，你可以先说一句简短的话。如果没有工具能表达用户所求，直说，不要猜。不要在你的响应中包含内部或系统 XML 标签"）——不要在提示词中点名 `<thinking>` 标签
- [ ] **[TUNE]** Vision pipelines: give it crop/analyze/verify tools - cheaper and more effective than raising thinking
      **[TUNE]** 视觉管线：给它裁剪/分析/校验工具——比提高思考更便宜也更有效
- [ ] **[TUNE]** Handle `stop_reason: "refusal"` before reading `content`, and opt into `fallbacks: "default"` (`server-side-fallback-2026-07-01`) rather than pinning a model - cyber-category refusals route to Claude Opus 4.8
      **[TUNE]** 读取 `content` 前处理 `stop_reason: "refusal"`，并选择 `fallbacks: "default"`（`server-side-fallback-2026-07-01`）而不是钉死模型——网络类别的拒答路由到 Claude Opus 4.8
- [ ] **[TUNE]** Long-horizon / agentic work: give the complete task spec up front in one turn rather than building it up across interactive turns
      **[TUNE]** 长程/智能体工作：在一个回合中前置给出完整任务规格，而不是跨交互回合逐步搭起来

---

## Migrating to Claude Sonnet 5 / 迁移到 Claude Sonnet 5

> **Model ID `claude-sonnet-5` is authoritative as written here.** When the user asks to migrate to Claude Sonnet 5, write `model="claude-sonnet-5"` exactly. Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entry exists in `shared/models.md`.

> **模型 ID `claude-sonnet-5` 以此处所写为准。**当用户要求迁移到 Claude Sonnet 5 时，一字不差地写 `model="claude-sonnet-5"`。**不要** WebFetch 去核实——本指南就是迁移目标 ID 的事实来源。对应条目存在于 `shared/models.md` 中。

Claude Sonnet 5 substantially improves on Sonnet 4.6 for coding and agentic work, reaching what was previously Opus-tier quality on many tasks. Its API surface aligns with Opus 4.7/4.8: manual extended thinking is removed (adaptive or disabled only, adaptive is the default), and non-default sampling parameters are rejected. This section is layered on top of the Sonnet 4.6 migration above - if the caller is jumping from Sonnet 4.5 or older, apply the 4.6 changes first, then this one.

Claude Sonnet 5 在编码与智能体工作上大幅超越 Sonnet 4.6，在许多任务上达到此前 Opus 档的质量。其 API 面与 Opus 4.7/4.8 对齐：手动扩展思考被移除（只能自适应或禁用，自适应为默认），非默认采样参数被拒。本节叠加在上文 Sonnet 4.6 迁移之上——如果调用方从 Sonnet 4.5 或更早跳过来，先应用 4.6 变更，再应用本节。

**TL;DR for someone already on Sonnet 4.6:** swap the model ID to `claude-sonnet-5`. Replace any remaining `thinking: {type: "enabled", budget_tokens: N}` with `thinking: {type: "adaptive"}` (the transitional escape hatch is gone - it now 400s), and note that omitting `thinking` now runs adaptive (4.6 ran thinking-off). Strip non-default `temperature`/`top_p`/`top_k`. Re-run `count_tokens()` against `claude-sonnet-5` - the new tokenizer produces ~30% more tokens for the same text, so token-budgeted limits and cost baselines shift (per-token pricing is also lower than Sonnet 4.6: $2/$10 vs $3/$15 per MTok). `effort` defaults to `high`, the same as Sonnet 4.6 - raise to `xhigh` for the hardest coding and agentic tasks (Claude Sonnet 5 supports the full `low`/`medium`/`high`/`xhigh`/`max` range), and give `max_tokens` headroom at `xhigh`/`max` (the new tokenizer means a Sonnet-4.6-tuned `max_tokens` may truncate equivalent output). Then re-tune prompts: Claude Sonnet 5 interprets instructions more literally than 4.6 - holdover style/tone directives now apply at face value; it is more agentic by default and reaches for tools and self-verification loops more readily (with thinking disabled it is less tool-eager - add an explicit nudge); it gives better in-progress updates by default (drop forced "summarize every N tool calls" scaffolding); and code-review harnesses with conservative-reporting instructions may see lower recall (tell it to report everything and filter downstream).

**对已在 Sonnet 4.6 上的用户，TL;DR：**把模型 ID 换成 `claude-sonnet-5`。把残余的 `thinking: {type: "enabled", budget_tokens: N}` 换成 `thinking: {type: "adaptive"}`（过渡逃生舱口没了——现在返回 400），并注意省略 `thinking` 现在会跑自适应（4.6 是思考关闭）。剥掉非默认的 `temperature`/`top_p`/`top_k`。对 `claude-sonnet-5` 重跑 `count_tokens()`——新分词器对同样文本多产出约 30% 的 token，所以按 token 预算设的限额与成本基线都会移动（单 token 定价也低于 Sonnet 4.6：$2/$10 对 $3/$15 每 MTok）。`effort` 默认 `high`，与 Sonnet 4.6 相同——最难的编码与智能体任务升到 `xhigh`（Claude Sonnet 5 支持完整的 `low`/`medium`/`high`/`xhigh`/`max` 范围），并在 `xhigh`/`max` 给 `max_tokens` 留余量（新分词器意味着按 Sonnet 4.6 调好的 `max_tokens` 可能截断同等输出）。然后重新调优提示词：Claude Sonnet 5 对指令的解释比 4.6 更字面——遗留的风格/语气指令如今按字面生效；它默认更智能体化，更轻易动用工具与自验证循环（思考关闭时则较少主动用工具——加一条明确的点拨）；它默认给出更好的进行中更新（去掉强制的"每 N 次工具调用总结一次"脚手架）；带保守报告指令的代码审查框架可能查全率下降（让它全部上报、在下游过滤）。
### Breaking changes (will 400 on Claude Sonnet 5) / 破坏性变更（在 Claude Sonnet 5 上会返回 400）

These bring the Sonnet line onto the same request surface as Opus 4.7/4.8. See the **Per-SDK Syntax Reference** above for the language-specific spelling of each.

这些变更把 Sonnet 线带上了与 Opus 4.7/4.8 相同的请求面。每种变更在各语言中的写法见上文 **Per-SDK Syntax Reference**。

**1. Extended thinking removed - adaptive only.** `thinking: {type: "enabled", budget_tokens: N}` returns a 400. The transitional escape hatch that still worked on Sonnet 4.6 is gone. Use adaptive thinking with an effort hint:

**1. 扩展思考已移除——仅自适应。**`thinking: {type: "enabled", budget_tokens: N}` 返回 400。在 Sonnet 4.6 上仍可用的过渡逃生舱口没了。改用自适应思考加 effort 提示：

```python
# Before - deprecated on Sonnet 4.6, now errors on Claude Sonnet 5
thinking={"type": "enabled", "budget_tokens": 10000}

# After
thinking={"type": "adaptive"},
output_config={"effort": "high"},  # or "xhigh" for the hardest coding/agentic tasks
```

To turn thinking off entirely, set `thinking: {type: "disabled"}` - but see *Adaptive vs. disabled* below before doing so.

要完全关掉思考，设 `thinking: {type: "disabled"}`——但动手前先看下文 *Adaptive vs. disabled*。

**2. Sampling parameters rejected.** Setting `temperature`, `top_p`, or `top_k` to a non-default value returns a 400; omitting the parameter, or passing its default, is still accepted. The safest migration is to omit them entirely and steer with prompting. If the caller was relying on `temperature=0` for determinism, note in the migration comment that it never guaranteed identical outputs.

**2. 采样参数被拒。**把 `temperature`、`top_p` 或 `top_k` 设为非默认值返回 400；省略该参数或传其默认值仍被接受。最稳妥的迁移是完全省略它们，用提示词引导。如果调用方靠 `temperature=0` 换确定性，在迁移注释中说明它从未保证过完全相同的输出。

```python
# Before
client.messages.create(model="claude-sonnet-4-6", temperature=0.2, ...)

# After - omit entirely
client.messages.create(model="claude-sonnet-5", ...)
```

**3. Bedrock only: forced `tool_choice` requires `thinking: {type: "disabled"}`.** On Amazon Bedrock, pass `thinking: {type: "disabled"}` alongside `tool_choice: {type: "tool", name: ...}` or `tool_choice: {type: "any"}`. The Claude API and Vertex AI do not require this.

**3. 仅 Bedrock：强制 `tool_choice` 需要 `thinking: {type: "disabled"}`。**在 Amazon Bedrock 上，`tool_choice: {type: "tool", name: ...}` 或 `tool_choice: {type: "any"}` 须搭配 `thinking: {type: "disabled"}` 一起传。Claude API 与 Vertex AI 不要求。

**Not a request-shape error, but handle it: cybersecurity safeguards.** Claude Sonnet 5 is substantially more cyber-capable than Sonnet 4.6, so - like Opus 4.7/4.8 - requests touching prohibited or high-risk topics may be refused. Handle it as a content outcome (see the `refusal` stop-reason guidance in the Claude Fable 5.1 section if the caller needs a fallback path).

**不是请求形状错误，但要处理：网络安全防护。**Claude Sonnet 5 的网络能力远超 Sonnet 4.6，因此——与 Opus 4.7/4.8 一样——触及禁止或高风险主题的请求可能被拒。把它当作内容层面的结果处理（调用方需要回退路径时，见 Claude Fable 5.1 一节中的 `refusal` 停止原因指导）。

**Unchanged from Sonnet 4.6:** assistant-turn prefills still return a 400 (use `output_config.format` or a system-prompt instruction); the 1M-token context window, the 128k max-output ceiling, prompt caching, batch processing, the Files API, PDF support, vision, and the full server- and client-side tool set all carry over.

**与 Sonnet 4.6 相同：**assistant 回合 prefill 仍返回 400（用 `output_config.format` 或系统提示词指令）；1M token 上下文窗口、128k 最大输出上限、提示词缓存、批处理、Files API、PDF 支持、视觉，以及完整的服务端与客户端工具集全部延续。

### Silent default change: adaptive thinking on when `thinking` is omitted / 静默默认变化：省略 `thinking` 时自适应思考开启

On Sonnet 4.6, a request with no `thinking` field runs **without** thinking. On Claude Sonnet 5, the same request runs with **adaptive thinking**. This is not an error - but callers who never set `thinking` will now see thinking output (and spend thinking tokens) where they didn't before. `max_tokens` is a hard limit on total output (thinking + response text), so a workload that ran thinking-off on Sonnet 4.6 by omission may now truncate. Either set `thinking: {type: "disabled"}` explicitly to keep the old behavior, or revisit `max_tokens` to leave room for thinking.

在 Sonnet 4.6 上，不含 `thinking` 字段的请求**不**思考运行。在 Claude Sonnet 5 上，同样的请求以**自适应思考**运行。这不是错误——但从未设置 `thinking` 的调用方如今会在以前没有的地方看到思考输出（并花费思考 token）。`max_tokens` 是总输出（思考 + 响应文本）的硬上限，所以在 Sonnet 4.6 上因省略而思考关闭的工作负载现在可能截断。要么显式设 `thinking: {type: "disabled"}` 保持旧行为，要么重审 `max_tokens` 给思考留空间。

### Silent default change: `thinking.display` defaults to `"omitted"` / 静默默认变化：`thinking.display` 默认 `"omitted"`

`thinking.display` defaults to `"omitted"` on Claude Sonnet 5 (matching Opus 4.7/4.8 and Claude Fable 5.1); on Sonnet 4.6 it defaulted to `"summarized"`. With the default, `thinking` blocks stream with empty text - to a streaming UI this looks like a long pause before output. Combined with the adaptive-on-by-default change above, a Sonnet 4.6 caller who omits `thinking` entirely now gets adaptive thinking *and* empty-text thinking blocks. If you stream reasoning to users, set `thinking: {type: "adaptive", display: "summarized"}` explicitly. `display` controls visibility only - thinking happens and is billed the same under every setting.

`thinking.display` 在 Claude Sonnet 5 上默认 `"omitted"`（与 Opus 4.7/4.8 及 Claude Fable 5.1 一致）；Sonnet 4.6 默认 `"summarized"`。在该默认下，`thinking` 块以空文本流出——对流式 UI 来说像是输出前的一段长停顿。叠加上面自适应默认开启的变化，完全省略 `thinking` 的 Sonnet 4.6 调用方现在会同时得到自适应思考*和*空文本思考块。如果把推理流式呈现给用户，显式设 `thinking: {type: "adaptive", display: "summarized"}`。`display` 只控制可见性——无论哪个设置，思考都会发生且计费相同。

### New tokenizer (~30% more tokens) / 新分词器（token 多约 30%）

Claude Sonnet 5 uses the same new tokenizer as Opus 4.7/4.8. The same input text produces approximately 30% more tokens than on Sonnet 4.6. No request/response shape changes and no code edits are required, but **everything measured or budgeted in tokens shifts**: `usage` fields and `count_tokens()` results for the same text are higher, the 1M context window holds less text, and a `max_tokens` limit tuned for Sonnet 4.6 may truncate equivalent output. Per-token pricing is $2/$10 per MTok (Sonnet 4.6 is $3/$15), so the cost of an equivalent request differs in both directions: more tokens at a lower rate. Re-run `count_tokens()` against `claude-sonnet-5` rather than reusing counts measured against earlier models, and re-baseline cost dashboards before reacting to measured shifts.

Claude Sonnet 5 使用与 Opus 4.7/4.8 相同的新分词器。同样的输入文本比 Sonnet 4.6 多产出约 30% 的 token。请求/响应形状不变、无需代码编辑，但**一切以 token 计量或预算的东西都移动了**：同样文本的 `usage` 字段与 `count_tokens()` 结果更高，1M 上下文窗口装下的文本更少，为 Sonnet 4.6 调好的 `max_tokens` 限制可能截断同等输出。单 token 定价为 $2/$10 每 MTok（Sonnet 4.6 为 $3/$15），所以同等请求的成本双向变化：token 更多、单价更低。对 `claude-sonnet-5` 重跑 `count_tokens()`，不要复用对早期模型测得的计数，并在对测得偏移做反应前重新定基成本仪表盘。

### Choosing an effort level on Claude Sonnet 5 / 在 Claude Sonnet 5 上选择 effort 级别

`effort` defaults to `high` when not set (same as Sonnet 4.6 and Opus 4.8). Claude Sonnet 5 supports the full `low`/`medium`/`high`/`xhigh`/`max` range - the first Sonnet-tier model with `xhigh`. **Keep the `high` default for most work and raise to `xhigh` for the hardest coding and agentic tasks**:

未设置时 `effort` 默认 `high`（与 Sonnet 4.6 和 Opus 4.8 相同）。Claude Sonnet 5 支持完整的 `low`/`medium`/`high`/`xhigh`/`max` 范围——首个支持 `xhigh` 的 Sonnet 档模型。**多数工作保持 `high` 默认，最难的编码与智能体任务升到 `xhigh`**：

| Level    | When to use on Claude Sonnet 5 |
| -------- | ----- |
| `max`    | Tasks needing the absolute highest capability with no token constraint. Can deliver gains in some use cases but may show diminishing returns and is sometimes prone to overthinking - test before committing |
| `xhigh`  | The hardest coding and agentic use cases - the recommended setting for those |
| `high`   | The default; balances token usage and intelligence for most use cases |
| `medium` | Cost-saving step-down from the default - comparable to Sonnet 4.6 at `high` |
| `low`    | Short, scoped tasks and latency-sensitive workloads that aren't intelligence-sensitive (chat, simple lookups) |

| 级别 | 在 Claude Sonnet 5 上何时使用 |
| --- | --- |
| `max` | 需要绝对最高能力且无 token 约束的任务。在某些用例能带来增益，但可能收益递减，有时容易过度思考——采用前先测试 |
| `xhigh` | 最难的编码与智能体用例——这些场景的推荐设置 |
| `high` | 默认值；对多数用例平衡 token 用量与智能 |
| `medium` | 相对默认的成本节省降档——大致相当于 `high` 档的 Sonnet 4.6 |
| `low` | 短小、范围明确的任务，以及对延迟敏感、对智能不敏感的工作负载（聊天、简单查询） |

As a rough cross-model mapping when migrating: Claude Sonnet 5 at `medium` is comparable in intelligence to Sonnet 4.6 at `high`, and Claude Sonnet 5 at `high` is comparable to Sonnet 4.6 at `max`. When benchmarking, match by observed thinking length rather than effort name.

迁移时的粗略跨模型映射：`medium` 档的 Claude Sonnet 5 智能上大致相当于 `high` 档的 Sonnet 4.6，`high` 档的 Claude Sonnet 5 大致相当于 `max` 档的 Sonnet 4.6。做基准测试时，按观察到的思考长度匹配，而不是按 effort 名称。

Claude Sonnet 5 **respects effort levels strictly, especially at the low end**. At `low` and `medium` it scopes its work to what was asked rather than going above and beyond - good for latency and cost, but on moderately complex tasks at `low` there is some risk of under-thinking. If you observe shallow reasoning on complex problems, **raise effort to `high` or `xhigh` rather than prompting around it**. If you must keep effort at `low` for latency, add targeted guidance:

Claude Sonnet 5 **对 effort 档位的遵循严格，低档尤甚**。在 `low` 和 `medium`，它把工作范围限定在所问内容上而不额外发挥——对延迟和成本有利，但在 `low` 档处理中等复杂度任务时存在思考不足的一定风险。若在复杂问题上观察到浅层推理，**把 effort 提到 `high` 或 `xhigh`，而不是绕着它调提示词**。若为延迟必须保持 `low`，添加针对性指导：

> *"This task involves multi-step reasoning. Think carefully through the problem before responding."*
> *"本任务涉及多步推理。回应之前把问题仔细想透。"*

**Leave `max_tokens` headroom at `xhigh`/`max`.** Set a large output token budget (up to the 128k cap, unchanged from Sonnet 4.6) so the model has room for thinking and tool calls. On long tasks, adaptive thinking can use a large share of the budget; if the budget is tight you may see a response that is almost entirely thinking followed by a truncated answer and `stop_reason: "max_tokens"` - raise `max_tokens` or drop to `medium`. Because Claude Sonnet 5 uses the new tokenizer (~30% more tokens for the same text), `max_tokens` limits tuned for Sonnet 4.6 may truncate equivalent output.

**在 `xhigh`/`max` 给 `max_tokens` 留余量。**设一个较大的输出 token 预算（最高 128k 上限，与 Sonnet 4.6 相同），让模型有空间思考和调用工具。长任务上，自适应思考可能占预算的大头；预算太紧时，你会看到几乎全是思考、随后答案被截断且 `stop_reason: "max_tokens"` 的响应——调高 `max_tokens` 或降到 `medium`。由于 Claude Sonnet 5 使用新分词器（同样文本 token 多约 30%），为 Sonnet 4.6 调好的 `max_tokens` 限制可能截断同等输出。

### Adaptive vs. disabled thinking / 自适应思考与禁用思考

Leave adaptive thinking on. Claude Sonnet 5 calibrates thinking spend to task complexity; the small added latency is usually worth the quality gain. If the caller was running Sonnet 4.6 with thinking off, **try adaptive + `effort: "low"` first** rather than `thinking: {type: "disabled"}`.

让自适应思考保持开启。Claude Sonnet 5 按任务复杂度校准思考花费；小幅增加的延迟通常换来值得的质量增益。如果调用方在 Sonnet 4.6 上是关思考运行的，**先试自适应 + `effort: "low"`**，而不是 `thinking: {type: "disabled"}`。

The triggering behavior for adaptive thinking is steerable. If the model emits thinking blocks more often than wanted (which can happen with large or complex system prompts), prompt it directly - and measure the effect on quality:

自适应思考的触发行为可以引导。如果模型发出思考块的频率高于预期（在系统提示词庞大或复杂时可能发生），直接用提示词引导——并测量对质量的影响：

> *"Thinking adds latency and should only be used when it will meaningfully improve answer quality, typically for problems that require multi-step reasoning. When in doubt, respond directly."*
> *"思考会增加延迟，只在能显著提升答案质量时使用，通常是要求多步推理的问题。拿不准就直接作答。"*

Conversely, if you're running hard workloads at `medium` and seeing under-thinking, the first lever is to raise effort; if you need finer control, prompt for it directly.

反过来，如果你在 `medium` 档跑困难工作负载并看到思考不足，第一杠杆是提高 effort；需要更细的控制就直接用提示词引导。

### Capability improvements / 能力提升

**Coding and agentic tasks.** The largest gains over Sonnet 4.6 are in coding and agentic tasks. Claude Sonnet 5 performs well out of the box on existing Sonnet 4.6 prompts.

**编码与智能体任务。**相对 Sonnet 4.6 的最大增益在编码与智能体任务上。Claude Sonnet 5 在现有 Sonnet 4.6 提示词上开箱表现良好。

**High-resolution vision.** Claude Sonnet 5 is the first Sonnet-tier model with high-resolution image support: maximum **2576 pixels on the long edge** (up from 1568px on Sonnet 4.6). High-res images can use up to ~3× more image tokens than on Sonnet 4.6 (4784 vs 1568 tokens per image at the limit) - if the added fidelity isn't needed, downsample before sending to control token costs. No beta header or opt-in required.

**高分辨率视觉。**Claude Sonnet 5 是首个支持高分辨率图像的 Sonnet 档模型：最大**长边 2576 像素**（Sonnet 4.6 为 1568px）。高分辨率图像耗用的图像 token 可达 Sonnet 4.6 的约 3 倍（上限处每图 4784 对 1568 token）——若不需要这份额外保真度，发送前降采样以控制 token 成本。无需 beta 头或选入。

**Computer use.** Supports the `computer_20251124` tool version (beta header `computer-use-2025-11-24`). Capability works across resolutions up to the 2576px / 3.75MP maximum; sending screenshots at **1080p** provides a good balance of performance and cost. For particularly cost-sensitive workloads, **720p** or **1366×768** are lower-cost options with strong performance. Test to find the ideal settings for the use case; experimenting with `effort` can also help tune behavior.

**计算机使用。**支持 `computer_20251124` 工具版本（beta 头 `computer-use-2025-11-24`）。该能力在最高 2576px / 3.75MP 上限内各分辨率下都能工作；以 **1080p** 发送截图是性能与成本的良好平衡。对成本特别敏感的工作负载，**720p** 或 **1366×768** 是性能依然强劲的低成本选项。请实际测试找到该用例的理想设置；试验 `effort` 也有助于调控行为。

### Behavioral shifts (prompt-tunable) / 行为转变（可通过提示词调优）

None of these break code, but prompts tuned for Sonnet 4.6 may land differently. Claude Sonnet 5 follows instructions closely, so small explicit directives close the gap.

这些都不破坏代码，但为 Sonnet 4.6 调优的提示词可能产生不同效果。Claude Sonnet 5 对指令遵循紧密，小幅明确的指示即可补齐差距。

**Response length and verbosity.** Claude Sonnet 5 calibrates response length to task complexity rather than defaulting to a fixed verbosity - usually shorter on simple lookups, longer on open-ended analysis. If a product depends on a particular verbosity, tune the prompt. To decrease verbosity:

**回复长度与冗长度。**Claude Sonnet 5 按任务复杂度校准回复长度，而不是默认固定冗长度——简单查询通常更短，开放式分析更长。如果产品依赖特定冗长度，调优提示词。要降低冗长度：

> *"Provide concise, focused responses. Skip non-essential context, and keep examples minimal."*
> *"提供简洁、聚焦的回复。跳过非必要的上下文，示例从简。"*

If you see specific kinds of verbosity (e.g. over-explaining), add targeted instructions to prevent them. Positive examples showing the desired concision tend to be more effective than telling the model what not to do.

如果看到特定种类的冗长（如过度解释），添加针对性指令防止。展示所需简洁度的正面示例，往往比告诉模型不要做什么更有效。

**Tool use triggering.** Claude Sonnet 5 is more agentic than Sonnet 4.6 by default and will reach for tools and run self-verification loops more readily. **With thinking disabled**, the model is less likely to reach for tools or consider searching - if the harness relies on tool calls with thinking off, add an explicit nudge in the system prompt. `effort` is also a lever: `high` and `xhigh` show substantially more tool usage in agentic search and coding. For scenarios where you want more tool use, also explicitly instruct when and how to use the tools (e.g. if web-search is under-used, describe in the prompt why and how it should be called).

**工具使用触发。**Claude Sonnet 5 默认比 Sonnet 4.6 更智能体化，更轻易动用工具并运行自验证循环。**思考关闭时**，模型较不愿动用工具或考虑搜索——如果框架在关思考时依赖工具调用，在系统提示词中加一条明确的点拨。`effort` 也是杠杆：在智能体搜索与编码中，`high` 与 `xhigh` 的工具使用量明显更多。想要更多工具使用的场景，还要明确指示何时以及如何使用工具（如网页搜索用得少时，在提示词中说明为何以及如何调用它）。

**User-facing progress updates.** Claude Sonnet 5 provides regular, higher-quality updates to the user throughout long agentic traces by default. If the harness has scaffolding to force interim status messages ("After every 3 tool calls, summarize progress"), **try removing it**. If the length or content of the updates isn't well-calibrated to the use case, describe what they should look like in the prompt and provide an example.

**面向用户的进度更新。**Claude Sonnet 5 默认会在长程智能体轨迹中定期向用户提供更高质量的更新。如果框架有强制中期状态消息的脚手架（"每 3 次工具调用总结一次进度"），**试着移除它**。如果更新的长度或内容与用例不匹配，在提示词中描述应有的样子并给出示例。

**More literal instruction following.** Claude Sonnet 5 interprets prompts literally and explicitly, particularly at lower effort levels. It does not silently generalize an instruction from one item to another, and it does not infer requests that weren't made. The upside is precision - better for carefully tuned prompts, structured extraction, and pipelines that need predictable behavior. If an instruction should apply broadly, **state the scope explicitly** ("Apply this formatting to every section, not just the first one"). The same literalism means style/tone directives carried over from Sonnet 4.6 may now over-apply - re-baseline holdover lines like "be concise" before keeping them.

**更字面化的指令遵循。**Claude Sonnet 5 对提示词的解释字面而直接，在较低 effort 档位尤甚。它不会把一条指令默默地从一项推广到另一项，也不会推断未被提出的请求。好处是精确——对精心调优的提示词、结构化抽取和需要可预测行为的管线更有利。若某条指令应广泛适用，**明确陈述范围**（"把此格式应用到每一节，而不只是第一节"）。同样的字面化意味着从 Sonnet 4.6 沿袭的风格/语气指令现在可能过度适用——保留"be concise"这类遗留句子前先重新定基。

**Tone and writing style.** Prose style on long-form writing may shift. If a product relies on a specific voice, re-evaluate style prompts against the new baseline. For a warmer or more conversational voice:

**语气与写作风格。**长文写作的文风可能变化。如果产品依赖特定的声音，对照新基线重新评估风格提示词。想要更温暖、更对话化的声音：

> *"Use a warm, collaborative tone. Acknowledge the user's framing before answering."*
> *"使用温暖、协作的语气。回答之前先认可用户的表述。"*

Because `temperature`/`top_p`/`top_k` are not accepted on Claude Sonnet 5, callers who previously relied on `temperature` for stylistic variety must use system-prompt instructions instead.

由于 Claude Sonnet 5 不接受 `temperature`/`top_p`/`top_k`，此前靠 `temperature` 获得风格多样性的调用方必须改用系统提示词指令。

**Code review harnesses.** A review harness tuned for an earlier model may initially see lower recall on Claude Sonnet 5. This is likely a harness effect, not a capability regression: when a review prompt says "only report high-severity issues" / "be conservative" / "don't nitpick," Claude Sonnet 5 follows that instruction more faithfully than earlier models did - it investigates just as thoroughly, identifies the bugs, and then doesn't report findings it judges below the stated bar. Precision typically rises, but measured recall can fall even though underlying bug-finding ability has improved. Recommended prompt language:

**代码审查框架。**为早期模型调优的审查框架起初在 Claude Sonnet 5 上可能看到更低的查全率。这多半是框架效应，不是能力退化：当审查提示词说"只报高严重度问题"/"保守一点"/"别吹毛求疵"时，Claude Sonnet 5 比早期模型更忠实地遵循该指令——它照样彻底调查、照样找出 bug，然后不报告它判断低于所设门槛的发现。查准率通常上升，但测得的查全率可能下降，尽管底层的找 bug 能力提升了。推荐的提示词措辞：

> *"Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them."*
> *"报告你发现的每一个问题，包括你不确定或认为严重度较低的。此阶段不要按重要性或置信度过滤——单独的验证步骤会做这件事。你在这个阶段的目标是覆盖面：让一个稍后被过滤掉的发现浮现，好过悄悄丢掉一个真实的 bug。每条发现都附上置信度和估计严重度，便于下游过滤器排序。"*

This works even without an actual second step, but moving confidence filtering out of the finding stage often helps. If you do want single-pass self-filtering, be concrete about where the bar is rather than using qualitative terms like "important" - e.g. "report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences." Iterate against a subset of evals to validate recall/F1 gains.

即使没有真实的第二步，这也有效，但把置信度过滤从发现阶段挪出去通常更有帮助。若确实要单遍自过滤，把门槛说具体，而不要用"重要"这类定性词——例如"报告任何可能导致错误行为、测试失败或误导性结果的 bug；只省略纯样式或命名偏好之类的细枝末节。"针对一部分评测迭代，验证查全率/F1 增益。

**Design and frontend defaults.** Claude Sonnet 5 may settle into a consistent default visual style on open-ended frontend and design briefs. Generic instructions ("don't use that color," "make it clean and minimal") tend to shift it to a different fixed palette rather than producing variety. Two approaches work reliably: **specify a concrete alternative** (the model follows explicit specs precisely - give the palette, typography, layout, and spacing), or **have the model propose options before building** (e.g. "Before building, propose 4 distinct visual directions tailored to this brief - bg hex / accent hex / typeface plus a one-line rationale - ask the user to pick one, then implement only that direction"). Because `temperature` isn't accepted on Claude Sonnet 5, the propose-then-pick approach is the recommended way to get meaningfully different design directions across runs. To steer away from generic AI-aesthetic patterns, a short directive in the system prompt also helps:

**设计与前端默认。**在开放式的前端与设计需求上，Claude Sonnet 5 可能落入一致的默认视觉风格。泛泛的指令（"别用那个颜色"、"做得干净极简"）往往只是把它切到另一套固定配色，而不是产生多样性。两种做法可靠：**指定一个具体的替代方案**（模型精确遵循明确的规格——给出配色、字体、布局与间距），或**让模型在构建前提出方案选项**（如"构建之前，按本需求提出 4 个各不相同的视觉方向——背景色值 / 强调色值 / 字体加一行理由——让用户挑一个，然后只实现该方向"）。由于 Claude Sonnet 5 不接受 `temperature`，"先提议后挑选"是跨运行获得实质性不同设计方向的推荐方式。要把风格从泛型 AI 美学模式上引开，系统提示词里一条简短指示也有帮助：

> *"NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions."*
> *"绝不使用千篇一律的 AI 生成美学：被用滥的字体族（Inter、Roboto、Arial、系统字体）、俗套的配色（尤其是白色或深色背景上的紫色渐变）、可预测的布局与组件模式，以及缺乏语境个性的流水线设计。使用独特的字体、协调一致的配色与主题，并为主效果与微交互使用动画。"*

**Interactive coding products.** Token usage and behavior can differ between autonomous, asynchronous coding agents (single user turn) and interactive, synchronous coding agents (multiple user turns). To maximize both performance and token efficiency, use `effort: "xhigh"` or `"high"`, add autonomous features like an auto mode, and reduce the number of human interactions required. Specify task, intent, and constraints upfront in the first turn - well-specified initial prompts maximize autonomy and intelligence while minimizing extra token usage after user turns; ambiguous or progressively-revealed prompts tend to reduce token efficiency and sometimes performance.

**交互式编码产品。**自主异步编码智能体（单个用户回合）与交互式同步编码智能体（多个用户回合）的 token 用量与行为可能不同。要同时最大化性能与 token 效率，使用 `effort: "xhigh"` 或 `"high"`，添加 auto 模式之类的自主特性，并减少所需的人工交互次数。在第一回合前置说明任务、意图与约束——定义良好的初始提示词最大化自主性与智能，同时把用户回合之后的额外 token 用量降到最低；含糊或逐步揭示的提示词往往降低 token 效率，有时也降低性能。
### Claude Sonnet 5 Migration Checklist / Claude Sonnet 5 迁移清单

Every item is tagged: **`[BLOCKS]`** items cause a 400 error or truncated output if missed; **`[TUNE]`** items are quality/cost adjustments - surface them to the user as recommendations.

每一项都有标记：**`[BLOCKS]`** 项一旦遗漏会导致 400 错误或输出截断；**`[TUNE]`** 项是质量/成本调整——作为建议呈现给用户。

- [ ] **[BLOCKS]** Update the `model=` string to `claude-sonnet-5`
      **[BLOCKS]** 把 `model=` 字符串更新为 `claude-sonnet-5`
- [ ] **[BLOCKS]** Replace `thinking: {type: "enabled", budget_tokens: N}` with `thinking: {type: "adaptive"}` + `output_config.effort` - the Sonnet 4.6 transitional escape hatch is gone
      **[BLOCKS]** 用 `thinking: {type: "adaptive"}` + `output_config.effort` 替换 `thinking: {type: "enabled", budget_tokens: N}`——Sonnet 4.6 的过渡逃生舱口没了
- [ ] **[BLOCKS]** Strip `temperature`, `top_p`, `top_k` from request construction (use system-prompt instructions for tone/variety instead)
      **[BLOCKS]** 从请求构造中剥掉 `temperature`、`top_p`、`top_k`（语气/多样性改用系统提示词指令）
- [ ] **[BLOCKS]** Bedrock only: pass `thinking: {type: "disabled"}` alongside forced `tool_choice` (`{type: "tool"}` / `{type: "any"}`) - not required on the Claude API or Vertex AI
      **[BLOCKS]** 仅 Bedrock：强制 `tool_choice`（`{type: "tool"}` / `{type: "any"}`）须搭配 `thinking: {type: "disabled"}`——Claude API 与 Vertex AI 不要求
- [ ] **[BLOCKS]** At `effort: "xhigh"` or `"max"`: set a large `max_tokens` (up to 128k, unchanged from Sonnet 4.6) so the model has room for thinking and tool calls - Sonnet-4.6-tuned limits may truncate equivalent output under the new tokenizer (symptom: `stop_reason: "max_tokens"`)
      **[BLOCKS]** 当 `effort: "xhigh"` 或 `"max"`：设较大的 `max_tokens`（最高 128k，与 Sonnet 4.6 相同），让模型有空间思考和调用工具——新分词器下，按 Sonnet 4.6 调好的限制可能截断同等输出（症状：`stop_reason: "max_tokens"`）
- [ ] **[TUNE]** Thinking-field omitted: adaptive is now the default (4.6 ran thinking-off) - either set `thinking: {type: "disabled"}` to preserve the old behavior, or revisit `max_tokens` for the added thinking spend
      **[TUNE]** 省略 thinking 字段：自适应现在是默认（4.6 是思考关闭）——要么设 `thinking: {type: "disabled"}` 保持旧行为，要么为新增的思考花费重审 `max_tokens`
- [ ] **[TUNE]** `thinking.display` defaults to `"omitted"` (4.6 defaulted to `"summarized"`): if you stream reasoning to users, set `thinking: {type: "adaptive", display: "summarized"}` explicitly - the default streams empty-text thinking blocks (long pause before output)
      **[TUNE]** `thinking.display` 默认 `"omitted"`（4.6 默认 `"summarized"`）：若把推理流式呈现给用户，显式设 `thinking: {type: "adaptive", display: "summarized"}`——默认会流出空文本思考块（输出前的长停顿）
- [ ] **[TUNE]** New tokenizer: re-run `count_tokens()` against `claude-sonnet-5` (~30% more tokens for the same text); revisit `max_tokens` and compaction triggers sized close to expected output length; re-baseline cost dashboards before reacting (per-token pricing is lower than Sonnet 4.6: $2/$10 vs $3/$15 per MTok)
      **[TUNE]** 新分词器：对 `claude-sonnet-5` 重跑 `count_tokens()`（同样文本 token 多约 30%）；重审贴近预期输出长度设定的 `max_tokens` 与压缩触发器；在对偏移做反应前重新定基成本仪表盘（单 token 定价低于 Sonnet 4.6：$2/$10 对 $3/$15 每 MTok）
- [ ] **[TUNE]** Effort: keep the `high` default; raise to `xhigh` for the hardest coding/agentic tasks; `medium` is a cost-saving step-down (~ Sonnet 4.6 at `high`); reserve `low` for short, latency-sensitive, non-intelligence-sensitive tasks. If shallow reasoning shows up at `low`/`medium`, raise effort rather than prompting around it
      **[TUNE]** Effort：保持 `high` 默认；最难的编码/智能体任务升到 `xhigh`；`medium` 是成本节省降档（约等于 `high` 档的 Sonnet 4.6）；`low` 留给短小、延迟敏感、对智能不敏感的任务。若 `low`/`medium` 出现浅层推理，提高 effort 而不是绕着它调提示词
- [ ] **[TUNE]** Thinking-off callers: try `thinking: {type: "adaptive"}` + `effort: "low"` instead of `disabled`; if `disabled` must stay, add an explicit tool-triggering nudge (the model is less tool-eager with thinking off)
      **[TUNE]** 关思考的调用方：先试 `thinking: {type: "adaptive"}` + `effort: "low"` 而不是 `disabled`；若必须保持 `disabled`，加一条明确的工具触发点拨（思考关闭时模型较不愿用工具）
- [ ] **[TUNE]** Tool usage: more agentic than 4.6 by default (reaches for tools and self-verification more readily) - `effort` is a lever (`high`/`xhigh` for more tool use); add explicit when/how triggering instructions for under-used tools
      **[TUNE]** 工具使用：默认比 4.6 更智能体化（更轻易动用工具与自验证）——`effort` 是杠杆（要更多工具使用用 `high`/`xhigh`）；对使用不足的工具添加明确的何时/如何触发指令
- [ ] **[TUNE]** Drop forced progress-update scaffolding ("after every N tool calls, summarize") - the default updates are higher quality; describe the desired update shape if it still needs tuning
      **[TUNE]** 去掉强制进度更新的脚手架（"每 N 次工具调用总结一次"）——默认更新质量更高；仍需调优时描述期望的更新形态
- [ ] **[TUNE]** Re-baseline holdover style/tone/scope directives - instructions are followed literally; state the scope explicitly when one should apply broadly
      **[TUNE]** 重新定基遗留的风格/语气/范围指令——指令按字面执行；某条应广泛适用时明确陈述范围
- [ ] **[TUNE]** Verbosity-sensitive routes: tune response length via prompt (positive examples > "don't" instructions)
      **[TUNE]** 对冗长度敏感的路由：经提示词调校回复长度（正面示例优于"不要"类指令）
- [ ] **[TUNE]** Code-review harnesses with conservative-reporting instructions ("only high-severity", "don't nitpick"): switch to a coverage-first prompt (report everything with confidence + severity) and filter downstream - measured recall can otherwise fall even though bug-finding improved
      **[TUNE]** 带保守报告指令（"只报高严重度"、"别吹毛求疵"）的代码审查框架：切换到覆盖优先的提示词（全部上报并附置信度 + 严重度）并在下游过滤——否则即使找 bug 能力提升，测得的查全率也可能下降
- [ ] **[TUNE]** Open-ended frontend/design briefs: specify a concrete spec, or have the model propose 3-4 visual directions and pick one (the recommended substitute for `temperature`-driven variety)
      **[TUNE]** 开放式前端/设计需求：指定具体规格，或让模型提出 3-4 个视觉方向再挑一个（`temperature` 驱动多样性的推荐替代）
- [ ] **[TUNE]** Interactive coding products: use `effort: "xhigh"`/`"high"`, add autonomous features (e.g. auto mode), and put task/intent/constraints in the first turn
      **[TUNE]** 交互式编码产品：使用 `effort: "xhigh"`/`"high"`，添加自主特性（如 auto 模式），并把任务/意图/约束放进第一回合
- [ ] **[TUNE]** Vision-heavy / computer-use pipelines: leave images at native resolution up to 2576px long edge for the accuracy gain (downsample to control image-token cost if fidelity isn't needed); for computer use, 1080p screenshots are a good performance/cost balance with `computer_20251124`
      **[TUNE]** 视觉密集/计算机使用管线：图像保持原生分辨率、长边最高 2576px 以获得精度增益（不需要保真度时降采样以控制图像 token 成本）；计算机使用场景，1080p 截图配合 `computer_20251124` 是良好的性能/成本平衡
- [ ] **[TUNE]** Security workloads: add handling for safeguard refusals (cyber-capable topics may now be declined where Sonnet 4.6 answered)
      **[TUNE]** 安全类工作负载：为防护性拒答添加处理（此前 Sonnet 4.6 会回答的网络能力类主题现在可能被拒）

---

## Migrating to Claude Fable 5.1 / 迁移到 Claude Fable 5.1

> **Model IDs `claude-fable-5-1` and `claude-mythos-5-1` are authoritative as written here.** When the user asks to migrate to Claude Fable 5.1, write `model="claude-fable-5-1"` exactly; a Mythos Preview migrator in Project Glasswing writes `model="claude-mythos-5-1"` (everyone else: `claude-fable-5-1`). Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entries exist in `shared/models.md`.

> **模型 ID `claude-fable-5-1` 与 `claude-mythos-5-1` 以此处所写为准。**当用户要求迁移到 Claude Fable 5.1 时，一字不差地写 `model="claude-fable-5-1"`；Project Glasswing 中的 Mythos Preview 迁移者写 `model="claude-mythos-5-1"`（其他人：`claude-fable-5-1`）。**不要** WebFetch 去核实——本指南就是迁移目标 ID 的事实来源。对应条目存在于 `shared/models.md` 中。

Claude Fable 5.1 is Anthropic's most capable widely released model - for the most demanding reasoning and long-horizon agentic work. **Claude Mythos 5.1** (`claude-mythos-5-1`) offers the same capabilities and pricing through Project Glasswing (participation is the only way to access it), and succeeds the invitation-only **Claude Mythos Preview** (`claude-mythos-preview`). Everything in this section applies to both models except where § Claude Mythos 5.1 below says otherwise (the history-editing check, platform availability, and safeguards that depend on the access program). Mythos Preview migrators in Project Glasswing target `claude-mythos-5-1`; everyone else targets `claude-fable-5-1`. 1M token context window by default (the maximum is also the default), up to 128K output tokens per request.

Claude Fable 5.1 是 Anthropic 能力最强的广泛发布模型——面向最苛刻的推理与长程智能体工作。**Claude Mythos 5.1**（`claude-mythos-5-1`）经 Project Glasswing 提供相同的能力与定价（参与是唯一的获取途径），并接替仅限邀请的 **Claude Mythos Preview**（`claude-mythos-preview`）。本节的一切对两个模型都适用，除非下文 § Claude Mythos 5.1 另有说明（历史编辑检查、平台可用性，以及依赖访问计划的安全防护）。Project Glasswing 中的 Mythos Preview 迁移者以 `claude-mythos-5-1` 为目标；其他人以 `claude-fable-5-1` 为目标。默认 1M token 上下文窗口（最大值即默认值），每次请求最高 128K 输出 token。

**Migrate to Claude Fable 5.1 only when the user explicitly chose it.** It is not the default Opus upgrade path - pricing is above Opus-tier. For "upgrade to the latest model" requests, the target is `claude-opus-5-5`.

**只有当用户明确选择了 Claude Fable 5.1 时才迁移过去。**它不是默认的 Opus 升级路径——定价高于 Opus 档。"升级到最新模型"类请求的目标是 `claude-opus-5-5`。

【评论】这条规则把默认升级目标钉在性价比更高的 Opus 档，防止代理在用户未点名时擅自选择更昂贵的模型——属于技能层面对成本越权的约束。

### Breaking changes (vs Opus-tier and Mythos Preview) / 破坏性变更（相对 Opus 档与 Mythos Preview）

> Claude Fable 5.1 carries three further breaking changes introduced after Claude Fable 5: forced `tool_choice` (`any` / `tool`) returns a 400, thinking blocks are bound to the producing model, and editing earlier turns invalidates thinking blocks. They are covered in § Migrating to Claude Fable 5.1 from Claude Fable 5 below - apply that section on top of this one when coming from Opus-tier or older.

> Claude Fable 5.1 还带有 Claude Fable 5 之后引入的三项破坏性变更：强制 `tool_choice`（`any` / `tool`）返回 400、思考块绑定产出它的模型、编辑较早回合会使思考块失效。它们在下文 § Migrating to Claude Fable 5.1 from Claude Fable 5 中覆盖——来自 Opus 档或更早时，先应用本节，再应用那一节。

1. **Thinking is always on - remove all `thinking` configuration.** Adaptive thinking applies automatically whenever the `thinking` parameter is unset (an explicit `{type: "adaptive"}` is also accepted). Any other configuration is rejected: `thinking: {type: "disabled"}` and `{type: "enabled", budget_tokens: N}` both return a 400. `budget_tokens` has no replacement - the `output_config.effort` parameter is a separate output-level control, not a thinking budget.
   **思考始终开启——移除一切 `thinking` 配置。**只要 `thinking` 参数未设置，自适应思考就自动生效（也接受显式的 `{type: "adaptive"}`）。任何其他配置都被拒绝：`thinking: {type: "disabled"}` 与 `{type: "enabled", budget_tokens: N}` 都返回 400。`budget_tokens` 没有替代——`output_config.effort` 参数是独立的输出级控制，不是思考预算。

   ```python
   # Before (Mythos Preview / older models)
   client.messages.create(
       model="claude-mythos-preview",
       max_tokens=16000,
       thinking={"type": "enabled", "budget_tokens": 10000},
       messages=[...],
   )

   # After (Claude Fable 5.1) - no thinking field at all
   client.messages.create(
       model="claude-fable-5-1",
       max_tokens=16000,
       output_config={"effort": "high"},
       messages=[...],
   )
   ```

2. **Assistant prefill is not supported.** Replace last-assistant-turn prefills with structured outputs (`output_config.format`) or system prompt instructions - same replacement patterns as the 4.6-family prefill removal above. (One exception: the fallback-credit prefill claim - the server accepts the echoed assistant message when redeeming a credit; see the refusal section below.)
   **不支持 assistant 预填充。**用结构化输出（`output_config.format`）或系统提示词指令替换最后一个 assistant 回合的 prefill——与上文 4.6 家族移除 prefill 的替代模式相同。（一个例外：回退积分的 prefill 主张——兑换积分时服务器接受回显的 assistant 消息；见下文拒答一节。）

3. **Interleaved scratchpad is not supported** (Mythos Preview migrators only). Inter-tool reasoning is returned in thinking blocks instead, which adaptive thinking produces automatically between tool calls.
   **不支持交错草稿板**（仅 Mythos Preview 迁移者）。工具间推理改为在思考块中返回，自适应思考会在工具调用之间自动产生。

### Thinking output on Claude Fable 5.1 and Claude Mythos 5.1 / Claude Fable 5.1 与 Claude Mythos 5.1 上的思考输出

On Claude Fable 5.1 and Claude Mythos 5.1, the raw chain of thought is never returned. What you receive are **regular `thinking` blocks**, not encrypted blobs or `redacted_thinking`: `display: "summarized"` returns a readable summary of the reasoning, and with `"omitted"` - the default, same as Opus 4.8/4.7 - responses still include `thinking` blocks but the `thinking` field is an empty string. `display` controls visibility only; thinking happens and is billed the same under every setting. When continuing a conversation on the same model, pass thinking blocks back to the API **unchanged** (the standard multi-turn pattern; dropping or editing them breaks the turn).

在 Claude Fable 5.1 与 Claude Mythos 5.1 上，原始思维链从不返回。你收到的是**常规 `thinking` 块**，不是加密数据块或 `redacted_thinking`：`display: "summarized"` 返回推理的可读摘要，而 `"omitted"`——默认值，与 Opus 4.8/4.7 相同——响应仍包含 `thinking` 块，但 `thinking` 字段是空字符串。`display` 只控制可见性；无论哪个设置，思考都会发生且计费相同。在同一模型上继续对话时，把思考块**原样**传回 API（标准多回合模式；丢弃或编辑它们会破坏回合）。

When continuing on the same model, pass each thinking block back **exactly as received - including blocks whose `thinking` text is empty**. The API rejects blocks whose content has been *modified*, not blocks you have read; displaying the summary is fine, editing or reconstructing blocks is not.

在同一模型上继续时，把每个思考块**完全按收到时的样子**传回——包括 `thinking` 文本为空的块。API 拒绝的是内容被*修改*过的块，不是你读取过的块；展示摘要没问题，编辑或重构块不行。

Regular thinking blocks aren't origin-locked - they replay across models fine (the server renders them into the target model's prompt). Fable-tier thinking is the exception: a Claude Fable 5.1 / Claude Mythos 5.1 block is read only by that pair (apart from Claude Mythos 5.1, no other model can read a Claude Fable 5.1 block - see Migrating to Claude Fable 5.1 from Claude Fable 5), and a thinking block from Claude Fable 5/Claude Mythos 5 replayed to a different model is **dropped from the prompt** rather than rendered (except by Claude Fable 5.1 / Claude Mythos 5.1, which read these blocks) - typically silently (early-access builds hard-rejected with `invalid_request_error`; that broke workflows and was reverted before launch, but the new behavior is still rolling out, so don't build logic that depends on either outcome). The drop happens before the prompt is priced, so a dropped block **lowers `usage.input_tokens`** - you aren't billed for it, and there's nothing to strip for cost. Don't strip *regular* thinking blocks either: removing them can trigger ordering/signature 400s. Two rules for replay bodies stand regardless: fallback-credit retries must echo the refused body **unchanged**, and `fallback` blocks from a mid-output fallback stay where they appeared.

常规思考块不锁定来源——它们跨模型回放没问题（服务端把它们渲染进目标模型的提示词）。Fable 档的思考是例外：Claude Fable 5.1 / Claude Mythos 5.1 的块只有这一对能读（除 Claude Mythos 5.1 外，没有其他模型能读 Claude Fable 5.1 的块——见 Migrating to Claude Fable 5.1 from Claude Fable 5），而来自 Claude Fable 5/Claude Mythos 5 的思考块回放给其他模型时会**从提示词中丢弃**而非渲染（能读这些块的 Claude Fable 5.1 / Claude Mythos 5.1 除外）——通常静默（早期访问版本曾以 `invalid_request_error` 硬拒；那破坏了工作流，发布前已回退，但新行为仍在推开，所以不要构建依赖任一结果的逻辑）。丢弃发生在提示词计价之前，所以被丢弃的块**降低 `usage.input_tokens`**——你不会为它付费，也没有为省钱而要剥掉的东西。*常规*思考块同样不要剥：移除它们可能触发顺序/签名 400。无论怎样，回放体有两条规则：回退积分重试必须**原样**回显被拒请求体，输出中途回退产生的 `fallback` 块保持在原位。

Related: a request that tries to elicit the model's internal reasoning *in the response text* can be refused with `stop_details.category: "reasoning_extraction"` - applications needing reasoning visibility should read the summarized `thinking` blocks instead of prompting for reasoning.

相关：试图让模型*在响应文本中*吐出内部推理的请求，可能被以 `stop_details.category: "reasoning_extraction"` 拒绝——需要推理可见性的应用应改为读取摘要式 `thinking` 块，而不是用提示词索取推理。

### Tokenizer - unchanged from Opus 4.8 / 分词器——与 Opus 4.8 相同

Claude Fable 5.1 uses the **same tokenizer as Claude Opus 4.8** (the tokenizer introduced with Opus 4.7). Token counts are roughly unchanged when migrating from Opus 4.7/4.8 or from `claude-mythos-preview`; per-token pricing differs.

Claude Fable 5.1 使用**与 Claude Opus 4.8 相同的分词器**（随 Opus 4.7 引入的分词器）。从 Opus 4.7/4.8 或 `claude-mythos-preview` 迁移时 token 计数大致不变；单 token 定价不同。

- Coming **from Opus 4.7/4.8 or `claude-mythos-preview`**: token counts are roughly unchanged. Re-baseline cost and latency on your own workloads for the per-token price difference.
  来自 **Opus 4.7/4.8 或 `claude-mythos-preview`**：token 计数大致不变。按单 token 价差，在自己的工作负载上重新定基成本与延迟。
- Coming **from Opus 4.6, Sonnet, Haiku, or older**: the Opus 4.7 tokenizer tokenizes the same content to roughly 1×-1.35× as many tokens (varies by content and workload shape). Do not reuse token counts, context-window budgets, or `max_tokens` settings measured on the old model; re-baseline with `count_tokens`.
  来自 **Opus 4.6、Sonnet、Haiku 或更早**：Opus 4.7 分词器把同样内容切成约 1×-1.35× 的 token（随内容与工作负载形态而变）。不要复用在旧模型上测得的 token 计数、上下文窗口预算或 `max_tokens` 设置；用 `count_tokens` 重新定基。

To measure the difference on your own prompts, call `count_tokens` once with your current model and once with `model: "claude-fable-5-1"`, and compare the two `input_tokens` values.

要在自己的提示词上测量差异，先用当前模型调用一次 `count_tokens`，再用 `model: "claude-fable-5-1"` 调用一次，比较两个 `input_tokens` 值。
### `refusal` stop reason - handle before reading content / `refusal` 停止原因——读取内容前处理

Claude Fable 5.1 runs safety classifiers on incoming requests, targeting research biology and most cybersecurity content (Claude Fable 5.1 is not intended for those domains); benign adjacent work - security tooling, life-sciences tasks - can occasionally trigger false positives, which is why the fallback patterns below matter even for legitimate workloads. (Most Claude consumer surfaces ship with built-in Opus 4.8 fallbacks; API callers configure their own.) A declined request returns a **successful HTTP 200** with `stop_reason: "refusal"`, plus a `stop_details` object with the policy category (values such as `"cyber"`, `"bio"`, `"reasoning_extraction"`, `"frontier_llm"`, or `null` - treat `null` as a permanent valid state; see the refusal category table in the public docs for the full set). **Branch on `stop_reason`, never on `stop_details`** - `stop_details` is informational and can be `null` even on a refusal, and `explanation` is not guaranteed present. Note that classifier blocks and ordinary model refusals (the model itself declining) both surface as `stop_reason: "refusal"`; `stop_details.category` tells you which class you're handling, and therefore whether retrying on a fallback model is the right response. The classifier can fire **before any output** (empty `content` array; counts against rate limits - for billing, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)) or **mid-stream** after partial output (input and already-streamed output are billed at normal rates - discard the partial output rather than treating it as complete). Code that reads `response.content[0]` unconditionally will break - check `stop_reason` first:

Claude Fable 5.1 对进入的请求运行安全分类器，针对研究生物学与大多数网络安全内容（Claude Fable 5.1 并非面向这些领域）；邻近的良性工作——安全工具、生命科学任务——偶尔也会触发误报，这正是下面的回退模式对合法工作负载也很重要的原因。（多数 Claude 消费端自带内建的 Opus 4.8 回退；API 调用方自行配置。）被拒的请求返回**成功的 HTTP 200**，带 `stop_reason: "refusal"`，外加含政策类别的 `stop_details` 对象（取值如 `"cyber"`、`"bio"`、`"reasoning_extraction"`、`"frontier_llm"` 或 `null`——把 `null` 当作永久有效的状态；完整集合见公开文档中的拒答类别表）。**按 `stop_reason` 分支，绝不要按 `stop_details`**——`stop_details` 只是信息性的，即使在拒答时也可能是 `null`，且不保证有 `explanation`。注意分类器拦截与普通模型拒答（模型自己拒绝）都以 `stop_reason: "refusal"` 呈现；`stop_details.category` 告诉你在处理哪一类，从而判断在回退模型上重试是否是正确响应。分类器可以在**任何输出之前**触发（`content` 数组为空；计入速率限制——计费见[拒答如何计费](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)），也可以在部分输出后的**流中**触发（输入与已流出部分按正常费率计费——把部分输出丢弃，不要当成完整结果）。无条件读取 `response.content[0]` 的代码会崩——先检查 `stop_reason`：

```python
response = client.messages.create(model="claude-fable-5-1", max_tokens=1024, messages=[...])
if response.stop_reason == "refusal":
    # classifiers declined; content is empty (pre-output) or partial (mid-stream)
    handle_refusal()
else:
    print(response.content[0].text)
```

**Default to opting in.** Fallbacks are not automatic on the API - a request without them simply stops on a refusal. Migrated and new Claude Fable 5.1 code should ship with pattern 1 below (pattern 2 on providers without server-side support) from day one, not as a later hardening step: emit the opt-in in the code, tell the user it's there, and remove it only if they decline.

**默认选入。**API 上回退不是自动的——没有回退的请求在拒答时直接停止。迁移的与新建的 Claude Fable 5.1 代码应从第一天就带上下面的模式 1（无服务端支持的提供方用模式 2），而不是日后加固时再加：在代码中写入选入，告诉用户它在那里，只有用户拒绝时才移除。

Three ways to retry a refused request on another model, in order of preference:

把被拒请求在另一个模型上重试的三种方式，按优先顺序：

**1. Server-side `fallbacks` parameter (beta; Claude API and Claude Platform on AWS) - preferred.** One round trip, a plain client, no client-side logic. Name substitute models (the supported fallback targets are `claude-opus-4-8` and `claude-opus-5`, expansion expected); on a policy decline the API runs the next model on the same request and returns its answer, with credit-style repricing applied automatically. A `stop_reason: "refusal"` on the final response means the whole chain refused.

**1. 服务端 `fallbacks` 参数（beta；Claude API 与 Claude Platform on AWS）——首选。**一次往返、普通客户端、无客户端逻辑。点名替代模型（受支持的回退目标是 `claude-opus-4-8` 与 `claude-opus-5`，预计会扩充）；政策拒绝时，API 在同一请求上运行下一个模型并返回其答案，自动按积分式重新计价。最终响应上的 `stop_reason: "refusal"` 意味着整条链都拒绝了。

```python
response = client.beta.messages.create(
    model="claude-fable-5-1",
    max_tokens=1024,
    betas=["server-side-fallback-2026-06-01"],
    fallbacks=[{"model": "claude-opus-4-8"}],
    messages=[{"role": "user", "content": "Hello, Claude"}],
)

# Switch points: one fallback block per model that ran and declined this turn
for block in response.content:
    if block.type == "fallback":
        print(f"{block.from_.model} declined; {block.to.model} continued")

# Served-by signal: a fallback_message in usage.iterations means a fallback model
# ran; pair it with stop_reason to confirm the fallback served the response
# (a fallback model can also refuse). Covers sticky turns too.
fallback_ran = any(
    entry.type == "fallback_message" for entry in response.usage.iterations or []
)
if fallback_ran and response.stop_reason != "refusal":
    print(f"Served by {response.model}")
```

Key semantics:

关键语义：

- **Header depends on the form you use.** The **array** form (`fallbacks: [{...}]`) requires exactly `server-side-fallback-2026-06-01` - other `server-side-fallback-*` values reject it with a 400, and that header carries the *earliest* date of the series (`-2026-06-09` and `-2026-06-02` were earlier previews), so do not "correct" it to a newer-looking date. The **`"default"` scalar** form uses `server-side-fallback-2026-07-01` instead - see § New API features under Migrating to Claude Opus 5. Pairing either header with the other form 400s. Rejected on the Batches API; available on the Claude API and Claude Platform on AWS; not on Amazon Bedrock, Vertex AI, or Microsoft Foundry (use pattern 2 there - the SDK middleware). Entries may override `max_tokens` per hop (bounding that attempt's own output independently of the top-level `max_tokens`); `thinking`, `output_config`, and `speed` overrides are rolling out (`speed` additionally requires its beta) - until your requests accept them, include only `model` and `max_tokens` in each entry. Entries must be distinct and must be in the requested model's `allowed_fallback_models` (published on `/v1/models` when the `server-side-fallback-2026-06-01` beta header is set - not yet visible under the `fallback-credit-*` header alone, and not exposed on Amazon Bedrock, Vertex AI, or Microsoft Foundry). The request *with an entry's overrides merged in* must be valid as a direct request to that entry's model.
  **头取决于你用哪种形式。****数组**形式（`fallbacks: [{...}]`）要求恰好是 `server-side-fallback-2026-06-01`——其他 `server-side-fallback-*` 取值会以 400 拒绝，而且这个头带的是该系列的*最早*日期（`-2026-06-09` 与 `-2026-06-02` 是更早的预览），所以不要"纠正"成看起来更新的日期。**`"default"` 标量**形式改用 `server-side-fallback-2026-07-01`——见 Migrating to Claude Opus 5 下的 § New API features。任一头与另一形式配对都会 400。Batches API 上被拒；Claude API 与 Claude Platform on AWS 上可用；Amazon Bedrock、Vertex AI 与 Microsoft Foundry 上不可用（在那里用模式 2——SDK 中间件）。条目可逐跳覆盖 `max_tokens`（独立于顶层 `max_tokens` 约束该次尝试自身的输出）；`thinking`、`output_config` 与 `speed` 覆盖正在推开（`speed` 另需其 beta）——在你的请求接受它们之前，每个条目只写 `model` 与 `max_tokens`。条目必须互不相同，且必须在所请求模型的 `allowed_fallback_models` 中（设置 `server-side-fallback-2026-06-01` beta 头时发布在 `/v1/models` 上——仅带 `fallback-credit-*` 头时还看不到，Amazon Bedrock、Vertex AI 与 Microsoft Foundry 上也不暴露）。*合并了条目覆盖后*的请求，作为对该条目模型的直接请求必须有效。
- **Triggers on policy declines only** - rate limits, overloads, and server errors on the requested model are returned as-is, never falling back.
  **只在政策拒绝时触发**——所请求模型的速率限制、过载与服务器错误原样返回，绝不回退。
- **Reading the response:** a `fallback` content block (`{"type": "fallback", "from": {"model": ...}, "to": {"model": ...}}`) marks each switch point in `content`; the served-by signal is a `fallback_message` entry in `usage.iterations` (don't rely on the block - sticky-served turns have none). Top-level `model` names the model that produced the message.
  **读取响应：**`fallback` 内容块（`{"type": "fallback", "from": {"model": ...}, "to": {"model": ...}}`）在 `content` 中标记每个切换点；served-by 信号是 `usage.iterations` 中的 `fallback_message` 条目（别依赖块——粘性服务的回合没有块）。顶层 `model` 给出产出该消息的模型。
- **Billing:** `usage.iterations` is the per-attempt source of truth; top-level `usage` covers only the attempt that produced the returned message. Declined-before-output attempts are reported (for whether they're billed, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)); an attempt that declines mid-stream bills at normal rates, and fallback attempts bill at the fallback model's rates. Each attempt claims the rate limits of the model that ran it - if the fallback model is rate-limited or overloaded, the fallback attempt is not made and the preceding refusal is returned instead with `stop_details.recommended_model` naming a model to retry directly (the recommendation is a hint, not a guarantee, and is `null` when no recommendation is available) - size fallback-model limits for expected refusal volume.
  **计费：**`usage.iterations` 是逐次尝试的事实来源；顶层 `usage` 只覆盖产出返回消息的那次尝试。输出前被拒的尝试会被报告（是否计费见[拒答如何计费](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)）；流中拒绝的尝试按正常费率计费，回退尝试按回退模型的费率计费。每次尝试占用运行它的模型的速率限制——如果回退模型被限流或过载，回退尝试不会发起，而是返回先前的拒答，并带 `stop_details.recommended_model` 点名可直接重试的模型（该推荐只是提示，不是保证，无推荐时为 `null`）——按预期的拒答量为回退模型的限额做容量规划。
- **Sticky routing:** once a conversation falls back, later requests with `fallbacks` (streaming and non-streaming - on a stream the decision is made before it opens, so `message_start` already names the fallback model) are served directly by the fallback model for ~1 hour (best-effort; org-scoped content-hash record, not message content; not recorded for ZDR orgs). Handle the requested model being tried again at any time.
  **粘性路由：**一旦对话发生回退，其后带 `fallbacks` 的请求（流式与非流式——流上的决定在打开流之前做出，所以 `message_start` 已点名回退模型）会在约 1 小时内由回退模型直接服务（尽力而为；按组织范围的内容哈希记录，不含消息内容；ZDR 组织不记录）。要处理所请求模型随时被重新尝试的情况。
- **Echoing fallback turns back:** after a mid-output fallback, omit `thinking`, `redacted_thinking`, and `tool_use` blocks - plus any `server_tool_use` block without its matching `server_tool_result`, and any other unrecognized model-internal block type - that appear *before* the final `fallback` block; text blocks, paired server-tool blocks, and everything after the boundary echo normally. The `fallback` block itself is an ignored audit marker (keep or drop). Streaming: the retry happens on the same stream and already-received content is never invalidated - a pre-output block is seamless (`message_start` names the fallback model; the `fallback` block arrives as an ordinary `content_block_start`, first in `content` - there is no special SSE event type; note `message_start` arrives only after the declined attempt, so time-to-first-byte includes it), and a mid-stream block keeps the partial, marks the boundary with the block, and continues - only the partial's `text` blocks are passed to the fallback model as continuation context (other block types stay in `content` but aren't part of it). Non-streaming mid-output declines omit the declined partial entirely.
  **回显回退回合：**输出中途回退之后，省略出现在最后一个 `fallback` 块*之前*的 `thinking`、`redacted_thinking` 与 `tool_use` 块——外加没有配对 `server_tool_result` 的 `server_tool_use` 块，以及其他无法识别的模型内部块类型；文本块、成对的服务端工具块以及边界之后的一切正常回显。`fallback` 块本身是被忽略的审计标记（留着或去掉都行）。流式：重试发生在同一条流上，已接收的内容永不失效——输出前的块无缝（`message_start` 点名回退模型；`fallback` 块作为普通 `content_block_start` 到达，位于 `content` 首位——没有特殊的 SSE 事件类型；注意 `message_start` 在被拒尝试之后才到，所以首字节时间包含它），流中的块保留部分内容、用块标记边界并继续——只有该部分的 `text` 块作为续写上下文传给回退模型（其他块类型留在 `content` 中但不参与）。非流式的输出中途回退则完全省略被拒的部分内容。

**2. SDK client-side middleware - for providers without server-side fallbacks (Amazon Bedrock, Vertex AI, Microsoft Foundry).** Register it on the client and every `client.beta.messages` request (streaming included) retries refusals automatically, splicing the fallback model's events onto the open stream in the same wire shape as pattern 1 (a `fallback` content block at each boundary, per-hop `usage.iterations`). It is also a beta surface: the middleware sends the `fallback-credit-2026-07-01` header by default (the earlier `-2026-06-01` value is still accepted) so retries are repriced via credit tokens (override with its `betas` option). `BetaFallbackState` pins follow-up turns to the model that accepted (the client-side analog of sticky routing) - reuse one state object per conversation:

**2. SDK 客户端中间件——用于没有服务端回退的提供方（Amazon Bedrock、Vertex AI、Microsoft Foundry）。**把它注册到客户端上，每个 `client.beta.messages` 请求（包括流式）都会自动重试拒答，把回退模型的事件按模式 1 的相同线上形状拼接到打开的流上（每个边界一个 `fallback` 内容块、逐跳的 `usage.iterations`）。它同样是 beta 面：中间件默认发送 `fallback-credit-2026-07-01` 头（较早的 `-2026-06-01` 取值仍被接受），重试经积分 token 重新计价（用其 `betas` 选项覆盖）。`BetaFallbackState` 把后续回合钉在接受的模型上（粘性路由的客户端对应物）——每个会话复用同一个状态对象：

```python
from anthropic import Anthropic, BetaFallbackState, BetaRefusalFallbackMiddleware

client = Anthropic(middleware=[BetaRefusalFallbackMiddleware([{"model": "claude-opus-4-8"}])])
state = BetaFallbackState()  # pins follow-ups to the model that accepted
with state:
    response = client.beta.messages.create(model="claude-fable-5-1", max_tokens=1024, messages=messages)
```

Create **one state per conversation** - it is the pinning scope; sharing one across conversations pins unrelated threads together, and a conversation without a state is never pinned. Per-language naming (from the GA SDK examples - don't improvise):

每个会话创建**一个状态**——它是钉定范围；跨会话共享一个会把不相关的线程钉在一起，而没有状态的会话永远不被钉定。各语言命名（来自 GA SDK 示例——不要即兴发挥）：

- **TypeScript**: `betaRefusalFallbackMiddleware([...])` in the client's `middleware` array; pass `{ fallbackState: state }` (a `BetaFallbackState`) as a request option.
  **TypeScript**：在客户端的 `middleware` 数组中放 `betaRefusalFallbackMiddleware([...])`；把 `{ fallbackState: state }`（一个 `BetaFallbackState`）作为请求选项传入。
- **Go**: `option.WithMiddleware(betafallback.BetaRefusalFallbackMiddleware([]anthropic.BetaFallbackParam{{Model: ...}}))` (package `lib/betafallback`); state via `betafallback.WithBetaFallbackState(&betafallback.BetaFallbackState{})` passed as a request option. Server-side equivalents: `Fallbacks: []anthropic.BetaFallbackParam{...}` + `anthropic.AnthropicBetaServerSideFallback2026_06_01`.
  **Go**：`option.WithMiddleware(betafallback.BetaRefusalFallbackMiddleware([]anthropic.BetaFallbackParam{{Model: ...}}))`（包 `lib/betafallback`）；状态经 `betafallback.WithBetaFallbackState(&betafallback.BetaFallbackState{})` 作为请求选项传入。服务端等价物：`Fallbacks: []anthropic.BetaFallbackParam{...}` + `anthropic.AnthropicBetaServerSideFallback2026_06_01`。
- **C#**: it's a *handler* - `new AnthropicClient { Handlers = [new BetaRefusalFallbackHandler { Fallbacks = [new(Model.ClaudeOpus4_8)] }] }` (namespace `Anthropic.Helpers`); state via `BetaFallbackState.Create()` scoped per call with `using (fallbackState.Use()) { ... }`. Server-side equivalents: `Fallbacks = [new(Model.ClaudeOpus4_8)]` + `AnthropicBeta.ServerSideFallback2026_06_01`.
  **C#**：它是一个 *handler*——`new AnthropicClient { Handlers = [new BetaRefusalFallbackHandler { Fallbacks = [new(Model.ClaudeOpus4_8)] }] }`（命名空间 `Anthropic.Helpers`）；状态经 `BetaFallbackState.Create()` 创建，按调用用 `using (fallbackState.Use()) { ... }` 限定作用域。服务端等价物：`Fallbacks = [new(Model.ClaudeOpus4_8)]` + `AnthropicBeta.ServerSideFallback2026_06_01`。

For languages not listed (Java, Ruby, PHP) - or for a full runnable program in any language - each public SDK repo ships a fallbacks example under `examples/` (e.g. `examples/fallbacks.py`, `examples/refusal-fallback/`): WebFetch the repo from `shared/live-sources.md` § SDK Repositories rather than improvising the binding.

对未列出的语言（Java、Ruby、PHP）——或任何语言的完整可运行程序——每个公开 SDK 仓库都在 `examples/` 下带有一个 fallbacks 示例（如 `examples/fallbacks.py`、`examples/refusal-fallback/`）：从 `shared/live-sources.md` § SDK Repositories WebFetch 相应仓库，而不是即兴编写绑定。

**3. Hand-rolled retry + fallback credit (raw HTTP, or SDKs without the middleware).** Detect the refusal via `stop_reason` and re-send the conversation as-is on a model with broader availability such as `claude-opus-4-8` (no stripping required either way: Claude Fable 5's thinking blocks are silently ignored by models other than Claude Fable 5.1 / Claude Mythos 5.1, which read them, and Claude Fable 5.1's own blocks are dropped by the API for any other model - breaking change 2 in § Migrating to Claude Fable 5.1 from Claude Fable 5); keep using the fallback model for subsequent turns. **Fallback credit** (beta: Claude API, Claude Platform on AWS, Amazon Bedrock, Vertex AI, and Microsoft Foundry) makes those retries cheaper. Prompt caches are per-model, so a plain retry pays cold cache-writes on the new model. With the `fallback-credit-2026-07-01` beta header (send it on both the original request and the retry; `-2026-06-01` is still accepted, and `server-side-fallback-2026-07-01` grants the same fields), a refusal's `stop_details` carries `fallback_credit_token` (opaque; `null` when unavailable) and `fallback_has_prefill_claim`. Echo the token as the top-level `fallback_credit_token` request parameter on the retry (typed in the GA SDKs; on a pre-GA SDK pass it via `extra_body`) and the previously-cached span bills at cache-read rates - the retry costs what it would have if the conversation had been on that model all along. Rules: the retry body must match the refused request **exactly** in every prompt-shaping field (`system`, `messages`, `tools`, `tool_choice`, `thinking` - do **not** strip thinking blocks when redeeming a credit - the server handles them); the retry model must be in the refused model's `allowed_fallback_models`; the token expires in 5 minutes; Batches results carry no tokens. If `fallback_has_prefill_claim` is `true`, append one assistant message echoing the refused response's `content` - the retry model continues from where the refused model stopped (and completed server-tool work isn't re-run). When echoing, strip trailing whitespace from a final `text` block (the prefill validator rejects it; the credit match tolerates that edit), after omitting any unpaired `tool_use` blocks. On a 400, fall back to the unchanged body with the token; on a 400 naming `fallback_credit_token`, retry without it (credit forfeited).

**3. 手写重试 + 回退积分（原始 HTTP，或没有中间件的 SDK）。**经 `stop_reason` 检测拒答，把对话原样重发到可用性更广的模型，如 `claude-opus-4-8`（两种情况都无需剥离：Claude Fable 5 的思考块被 Claude Fable 5.1 / Claude Mythos 5.1 以外的模型静默忽略，能读它们的是这一对；而 Claude Fable 5.1 自己的块会被 API 为任何其他模型丢弃——见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中的破坏性变更 2）；后续回合继续使用回退模型。**回退积分**（beta：Claude API、Claude Platform on AWS、Amazon Bedrock、Vertex AI 与 Microsoft Foundry）让这些重试更便宜。提示词缓存按模型隔离，普通重试要为新模型付冷缓存写入。带上 `fallback-credit-2026-07-01` beta 头（原始请求与重试都要发；`-2026-06-01` 仍被接受，`server-side-fallback-2026-07-01` 授予相同字段），拒答的 `stop_details` 会带 `fallback_credit_token`（不透明；不可用时为 `null`）与 `fallback_has_prefill_claim`。在重试中把该 token 作为顶层 `fallback_credit_token` 请求参数回传（GA SDK 中有类型；pre-GA SDK 经 `extra_body` 传），先前已缓存的区间按缓存读取费率计费——重试的成本相当于对话一直在这个模型上的成本。规则：重试体在每个提示词塑造字段上必须与被拒请求**完全**一致（`system`、`messages`、`tools`、`tool_choice`、`thinking`——兑换积分时**不要**剥离思考块——服务端会处理它们）；重试模型必须在被拒模型的 `allowed_fallback_models` 中；token 5 分钟过期；Batches 结果不带 token。若 `fallback_has_prefill_claim` 为 `true`，追加一条回显被拒响应 `content` 的 assistant 消息——重试模型从被拒模型停止之处继续（已完成的服务端工具工作不会重跑）。回显时，去掉最后一个 `text` 块的末尾空白（prefill 校验器会拒收；积分匹配容忍这个编辑），并在其之前省去任何未配对的 `tool_use` 块。遇 400 时，改用带 token 的原样请求体；遇点名 `fallback_credit_token` 的 400，去掉它重试（积分作废）。

**Migrating code built on the v1 preview.** If the code you're editing carries any of these markers, it targets the discontinued early-access surface - migrate it to the v2 shapes above, and ship the header and parameter changes together (the v1 parameter shape under the v2 header is a 400):

**迁移构建在 v1 预览上的代码。**如果你正在编辑的代码带有这些标记中的任何一个，它针对的是已停用的早期访问面——把它迁移到上述 v2 形状，并把头与参数改动一起交付（v2 头下的 v1 参数形状是 400）：

| v1 marker (replace) | v2 |
|---|---|
| `server-side-fallback-2026-06-09` / `-2026-06-02` header | `server-side-fallback-2026-06-01` (array form; the `"default"` scalar form uses `-2026-07-01`) |
| `fallback: {model, on_partial}` single object | `fallbacks: [{model, ...}]` array (1-3); `on_partial` no longer exists - partial-output behavior is fixed (streams keep the partial; non-streaming omits it). Unknown keys in an entry are a 400 |
| Top-level `response.fallback` object (`from_model`, `reason`) | Never emitted - read `fallback` content blocks (switch points, no `reason` field) and `usage.iterations` (served-by) |
| `event: fallback` SSE with discard indices | No dedicated event; streamed content is never invalidated - the switch arrives as an ordinary `content_block_start`/`stop` pair of type `fallback` |
| `fallback_primary` / `fallback_retry` iteration types | Blocked attempts are plain `message` entries; the serving attempt is `fallback_message` |
| `reason: "sticky"` | No reason field - sticky turns carry no block; detect via `fallback_message` in `usage.iterations` + `response.model` |
| `recommended_model` meaning "primary served the refusal" | Now populated only when the fallback attempt *couldn't run* (rate-limited/overloaded) - its presence means a direct retry on that model may succeed, not that it refused too |

| v1 标记（替换） | v2 |
| --- | --- |
| `server-side-fallback-2026-06-09` / `-2026-06-02` 头 | `server-side-fallback-2026-06-01`（数组形式；`"default"` 标量形式用 `-2026-07-01`） |
| `fallback: {model, on_partial}` 单对象 | `fallbacks: [{model, ...}]` 数组（1-3 个）；`on_partial` 不复存在——部分输出行为已固定（流保留部分内容；非流式省略它）。条目中的未知键是 400 |
| 顶层 `response.fallback` 对象（`from_model`、`reason`） | 永不出现——读取 `fallback` 内容块（切换点，无 `reason` 字段）与 `usage.iterations`（served-by） |
| 带 discard 索引的 `event: fallback` SSE | 无专用事件；流式内容永不失效——切换以普通的 `fallback` 类型 `content_block_start`/`stop` 对到达 |
| `fallback_primary` / `fallback_retry` 迭代类型 | 被拦的尝试是普通 `message` 条目；服务的尝试是 `fallback_message` |
| `reason: "sticky"` | 无 reason 字段——粘性回合不带块；经 `usage.iterations` 中的 `fallback_message` + `response.model` 检测 |
| `recommended_model` 表示"主模型处理了拒答" | 现在只在回退尝试*未能运行*（限流/过载）时填充——它的存在意味着直接在该模型上重试可能成功，而不是它也拒绝了 |

### Data retention requirement / 数据保留要求

Claude Fable 5.1 requires **30-day data retention** and is not available under zero data retention. Requests from an organization whose data-retention configuration doesn't meet the requirement return `400 invalid_request_error` - if a migration suddenly 400s with no obvious request problem, check the org's retention configuration before debugging the payload. On Amazon Bedrock, Google Vertex AI, and Microsoft Foundry, data-retention requirements are set by each platform.

Claude Fable 5.1 要求 **30 天数据保留**，在零数据保留下不可用。数据保留配置不满足要求的组织发出的请求返回 `400 invalid_request_error`——如果迁移突然开始 400 而请求本身看不出问题，先查组织的数据保留配置，再去调试载荷。在 Amazon Bedrock、Google Vertex AI 与 Microsoft Foundry 上，数据保留要求由各平台自行设定。

### What carries over unchanged / 原样延续的部分

Same Messages API and tool-use patterns as Opus-tier and Mythos Preview. Supported at launch: `output_config.effort` (`low`/`medium`/`high`/`xhigh`/`max`), Task Budgets (beta, `task-budgets-2026-03-13` header - on Claude Fable 5.1 confirm at launch), compaction (beta, `compact-2026-01-12` header), the memory tool, tool-call clearing via context editing, and high-resolution vision (no downscaling cap, as on Opus 4.7+).

与 Opus 档和 Mythos Preview 相同的 Messages API 与工具使用模式。发布时支持：`output_config.effort`（`low`/`medium`/`high`/`xhigh`/`max`）、Task Budgets（beta，`task-budgets-2026-03-13` 头——在 Claude Fable 5.1 上发布时确认）、压缩（beta，`compact-2026-01-12` 头）、memory 工具、经上下文编辑的工具调用清除，以及高分辨率视觉（与 Opus 4.7+ 一样无降采样上限）。
### Behavioral shifts (prompt-tunable) / 行为转变（可通过提示词调优）

None of these are API-breaking, but they're where migrated workloads feel different. Claude Fable 5.1's biggest gains are on work *above* what prior models could do (long-horizon autonomous runs, first-shot implementations of well-specified systems, end-to-end enterprise deliverables - financial analysis, spreadsheets, slides, docs - code review/debugging and repository-history search, vision on dense or degraded images - it's explicitly trained to use bash and crop tools on flipped/blurry/noisy inputs - navigating ambiguity, parallel sub-agent delegation and collaboration - it reliably sustains ongoing communications with long-running sub-agents and peer agents; note bug-finding gains exclude security-focused analysis, where the cyber classifiers apply) - don't evaluate it only on workloads older models already handled.

这些都不是 API 破坏性的，但它们是迁移后的工作负载感到不同的地方。Claude Fable 5.1 最大的增益在*超出*先前模型能力的工作上（长程自主运行、对定义良好的系统一次到位的实现、端到端企业交付物——财务分析、电子表格、幻灯片、文档——代码审查/调试与仓库历史搜索、密集或退化图像上的视觉——它被显式训练在翻转/模糊/噪点输入上使用 bash 与裁剪工具——在含糊中导航、并行子代理委派与协作——它能可靠地维持与长时运行子代理及同侪代理的持续通信；注意找 bug 的增益不包括安全聚焦的分析，那里适用网络分类器）——不要只在旧模型已能处理的工作负载上评估它。

**Longer turns by default - the biggest structural shift.** Individual requests on hard tasks can run many minutes at higher effort (a 15-minute single request is normal when the task involves gathering context, building, and self-verifying). Before migrating, plan timeouts, streaming, and user-facing progress indicators; structure work so callers check in on runs asynchronously rather than blocking inside one request. On ambiguous tasks Claude Fable 5.1 may need a small nudge to avoid overplanning:

**默认回合更长——最大的结构性变化。**困难任务上的单个请求在高 effort 下可运行许多分钟（任务涉及收集上下文、构建与自验证时，单请求 15 分钟是正常的）。迁移之前，规划好超时、流式与面向用户的进度指示；把工作结构化为让调用方异步查看运行，而不是阻塞在单个请求内。在含糊任务上，Claude Fable 5.1 可能需要一点小点拨以避免过度规划：

> When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.
> 当你有足够的信息可以行动时，就行动。不要重新推导对话中已确立的事实，不要重新翻案用户已经做出的决定，也不要在面向用户的消息中叙述你不会走的选项。如果你在权衡一个选择，给出推荐，而不是穷举式盘点。这不适用于思考块。

**Consider all effort levels.** `output_config.effort` is the primary intelligence/latency/cost control. Recommended defaults: `high` for most tasks, `xhigh` for the most capability-sensitive workloads, `medium`/`low` for routine work. Lower effort settings - including `low` - still perform very well on Claude Fable 5.1, often exceeding the `xhigh` or even `max` performance of previous models. Reduce effort if a task completes correctly but takes longer than necessary, or for a quicker interactive working style. At higher effort on routine work, Claude Fable 5.1 can gather context and deliberate beyond what the task needs (the flip side: higher effort buys excellent verification behavior and the most rigorous outputs). To prevent unrequested tidying or refactoring at higher effort:

**考虑所有 effort 档位。**`output_config.effort` 是主要的智能/延迟/成本控制。推荐默认：多数任务用 `high`，对能力最敏感的工作负载用 `xhigh`，例常工作用 `medium`/`low`。较低的 effort 设置——包括 `low`——在 Claude Fable 5.1 上仍表现很好，常超过先前模型在 `xhigh` 甚至 `max` 的表现。如果任务完成正确但耗时超过必要，或想要更快的交互节奏，就降低 effort。例常工作在高 effort 下，Claude Fable 5.1 收集上下文与斟酌可能超出任务所需（另一面：更高的 effort 换来出色的校验行为与最严谨的输出）。要防止高 effort 下未经要求的整理或重构：

> Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements - do the simplest thing that works well. Avoid premature abstraction. Avoid half-finished implementations either. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.
> 不要超出任务所需添加特性、重构或引入抽象。修 bug 不需要顺带清理周边，一次性操作通常不需要辅助函数。不要为假想的未来需求做设计——做能良好工作的最简单的事。避免过早抽象。也不要半途而废的实现。不要为不可能发生的场景添加错误处理、回退或校验。信任内部代码与框架保证。只在系统边界（用户输入、外部 API）校验。能直接改代码时，不要用特性开关或向后兼容垫片。

**Instruction following is strong - use it.** Claude Fable 5.1 is very responsive to explicit communication-style sections in system prompts; invest in them rather than fighting output style downstream. Un-steered - especially at higher effort - it can elaborate beyond what the task needs: heavily-structured PR descriptions, sections on alternatives that weren't chosen, comments narrating what the next line does. You don't need to enumerate these behaviors by name; a brief instruction is just as effective:

**指令遵循很强——用起来。**Claude Fable 5.1 对系统提示词中明确的沟通风格小节响应很好；在这里投入，而不是在下游对抗输出风格。不加引导时——高 effort 下尤甚——它可能铺陈超出任务所需：结构繁复的 PR 描述、讲未选方案的小节、叙述下一行做什么的注释。你不必逐一点名这些行为；一条简短指令同样有效：

> Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find" - the thing the user would ask for if they said "just give me the TLDR." Supporting detail and reasoning come after. Being readable and being concise are different things, and readability matters more. The way to keep output short is to be selective about what you include (drop details that don't change what the reader would do next), not to compress the writing into fragments, abbreviations, arrow chains like A -> B -> fails, or jargon.
> 先说结果。完成后的第一句话应回答"发生了什么"或"你发现了什么"——也就是用户说"直接给我 TLDR"时想要的东西。支持性细节与推理放在后面。易读与简短是两回事，易读更重要。让输出变短的方式是对收录内容有所取舍（丢掉不改变读者下一步行动的细节），而不是把文字压缩成碎片、缩写、A -> B -> fails 之类的箭头链或行话。

**Ground progress claims on long runs.** Require progress claims to be audited against tool results - in testing this nearly eliminated fabricated status reports on tasks designed to elicit them:

**为长程运行中的进度声明提供依据。**要求进度声明对照工具结果接受审计——测试中，这几乎消灭了在为诱发它们而设计的任务上编造的状态报告：

> Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.
> 在报告进度之前，把每条声明对照本次会话中的工具结果审计一遍。只报告你能指出证据的工作；某事尚未验证就明说。如实报告结果：测试失败就说失败并附输出；某步骤被跳过就说跳过；某事完成且经验证，就平实地陈述，不加对冲。

**State boundaries explicitly.** Claude Fable 5.1 sometimes takes unrequested-but-adjacent actions (e.g. composing an email straight to drafts, creating backup git branches). Define what it should *not* do:

**明确边界。**Claude Fable 5.1 有时会采取未被要求但相邻的行动（如直接把邮件写进草稿箱、创建备份 git 分支）。定义它*不应*做什么：

> When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state - restarts, deletes, config edits - check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
> 当用户在描述问题、提问或自言自语而非要求改动时，交付物是你的评估。报告发现然后停下。在用户要求修复之前不要动手修。在运行改变系统状态的命令——重启、删除、配置修改——之前，核实证据确实支持那个具体动作。与已知故障模式相似的症状可能有不同的成因。

**Let it delegate - asynchronously.** Parallel sub-agents are dependable on Claude Fable 5.1 - instead of suppressing delegation (a common prior-model guardrail), use sub-agents frequently and give explicit guidance on *when* delegation is desirable. Sub-agents that communicate **asynchronously** with the orchestrator outperform spawn-and-block: long-lived agents keep their context instead of re-establishing it per subtask (cache-read savings), the orchestrator isn't bottlenecked on the slowest sub-agent, and context persists across subtasks.

**让它委派——异步地。**并行子代理在 Claude Fable 5.1 上可靠——不要压制委派（常见于先前模型的护栏），而要频繁使用子代理，并就*何时*应当委派给出明确指导。与编排者**异步**通信的子代理优于"派生并阻塞"：长寿命的代理保有自己的上下文而不必每个子任务重建（缓存读取节省），编排者不会被最慢的子代理卡住，上下文也能跨子任务延续。

> Delegate independent subtasks to sub-agents and keep working while they run. Intervene if a sub-agent goes off track or is missing relevant context.
> 把独立的子任务委派给子代理，并在它们运行时继续工作。若某个子代理跑偏或缺少相关上下文，再介入。

**Give it a memory surface.** Claude Fable 5.1 performs notably better when it can write learnings somewhere for future reference - even a plain `.md` file. Tell it where, tell it to consult that file in future sessions, and give it a format:

**给它一个记忆面。**当 Claude Fable 5.1 能把学到的东西写到某处以供将来参考时——哪怕只是一个普通 `.md` 文件——它的表现明显更好。告诉它写在哪里，告诉它未来会话要查阅该文件，并给它一个格式：

> Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.
> 每个文件存一条经验，顶部放一行摘要。修正与确认有效的做法都记，包括它们为何重要。仓库或聊天历史已有的记录不要重复保存；更新已有笔记而不是另建重复；被证明错误的笔记就删掉。

**Rare: early stopping.** Deep into long sessions it can occasionally end a turn with a text-only statement of intent ("I'll now run X") without the tool call, or ask permission it doesn't need. A "continue" recovers it interactively; for autonomous pipelines add a system reminder:

**少见：提前停。**在很长的会话深处，它偶尔会用纯文字的意图声明（"我现在去运行 X"）结束回合而不发起工具调用，或请求并不需要的许可。交互场景一句"继续"即可救回；自主管线则加一条系统提醒：

> You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking 'Want me to...?' or 'Shall I...?' will block the work. For reversible actions that follow from the original request, proceed without asking. Offering follow-ups after the task is done is fine; asking permission after already discussing with the user before doing the work is not. Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ('I'll...', 'let me know when...'), do that work now with tool calls. End your turn only when the task is complete or you are blocked on input only the user can provide.
> 你在自主运行。用户并未实时观看，也无法在任务中途回答问题，因此问"要我……吗？"或"我可以……吗？"会阻塞工作。对于源于原始请求且可逆的行动，直接进行而不发问。任务完成后再提供后续选项没问题；动手之前已与用户讨论过却仍请求许可则不行。结束回合之前，检查你的最后一段。如果它是一个计划、一段分析、一个问题、一列后续步骤，或对尚未完成工作的承诺（"我将……"、"到时候告诉我……"），现在就用工具调用把那些工作做掉。只有任务完成或你被只有用户才能提供的输入阻塞时，才结束回合。

**Rare: context anxiety.** In very long sessions it can worry about running out of context - suggesting a new session or trimming its own work - most often when the harness surfaces a remaining-token countdown. Avoid showing explicit context-budget counts; if you must:

**少见：上下文焦虑。**在极长的会话中，它可能担心上下文用尽——建议新开会话或删减自己的工作——最常见于框架展示剩余 token 倒计时之时。避免展示明确的上下文预算计数；若必须展示：

> You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits - continue the work.
> 你的上下文余量充足。不要因上下文限制而停止、总结或建议新开会话——继续工作。

**Give the reason, not just the request.** Claude Fable 5.1 performs better when it understands the intent behind a request - it connects the task to relevant information rather than inferring intent on its own. This matters most for long-running agents juggling context from disparate workstreams:

**给出理由，而不只是请求。**当 Claude Fable 5.1 理解请求背后的意图时表现更好——它会把任务与相关信息连接起来，而不是自行推断意图。这对在不同工作流之间调度上下文的长时运行智能体最重要：

> I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].
> 我在做[更大的任务]，是为了[它服务谁]。他们需要[这份输出能带来什么]。带着这个背景：[请求]。

**Readability in long agentic sessions.** Deep into extended conversations (many tool calls, large working context) Claude Fable 5.1 can produce text users find hard to follow - dense arrow-chain shorthand, implementation-level detail, references to thinking the user never saw. A communication-style addendum strongly mitigates this; adapt:

**长智能体会话中的可读性。**在长对话深处（大量工具调用、庞大工作上下文），Claude Fable 5.1 可能产出用户难以跟随的文字——密集的箭头链速记、实现层细节、提及用户从未见过的思考。一段沟通风格附录能显著缓解；按需改编：

> Terse shorthand is fine between tool calls (that's you thinking out loud, and brevity there is good). Your final summary is different: it's for a reader who didn't see any of that. If you've been working for a while without the user watching - overnight, across many tool calls, since they last spoke - your final message is their first look at any of it. Write it as a re-grounding, not a continuation of your working thread: the outcome first, then the one or two things you need from them, each explained as if new. The vocabulary you built up while working is yours, not theirs; leave it behind unless you re-introduce it. When you write the summary at the end, drop the working shorthand. Write complete sentences. Spell out terms instead of abbreviating them. Don't use arrow chains, hyphen-stacked compounds, or labels you made up earlier - the reader doesn't have the context to decode them. When you mention files, commits, flags, or other identifiers, give each one its own plain-language clause saying what it is or what changed - never pack several into one parenthesized run or slash-separated list. Open with the outcome: one sentence on what happened or what you found. Then the supporting detail. If you have to choose between short and clear, choose clear.
> 工具调用之间用简短速记没问题（那是你在自言自语，简洁是好事）。你的最终总结不同：它面向的读者没看到那些。如果你已经工作了一段时间而用户没有旁观——过夜、跨许多工具调用、自上次对话以来——你的最终消息就是他们第一次看到这一切。把它写成一次重新铺垫，而不是你工作线程的延续：先说结果，再说你需要他们做的一两件事，每一件都当作新事物解释。你工作中积累的词汇是你的，不是他们的；除非重新介绍，否则不要带走。写结尾总结时，丢掉工作速记。写完整的句子。术语拼写完整而不是缩写。不要用箭头链、连字符堆叠的复合词或你此前自造的标签——读者没有解码它们的上下文。提到文件、提交、标志或其他标识符时，每个都单独用一个平实语言的从句说明它是什么或改了什么——绝不要把几个塞进一个括号串或斜杠分隔列表。以结果开篇：一句话说发生了什么或你发现了什么。然后是支持细节。如果必须在短与清晰之间取舍，选清晰。

### Long-running agent recommendations / 长时运行智能体建议

- **Make self-verification explicit.** For long-running builds, instruct it to establish and run its own checking harness on a cadence ("Establish a method for checking your own work as you build; run it every [interval], verifying against the specification with sub-agents"). Separate fresh-context verifier sub-agents tend to outperform self-critique.
  **把自验证写明。**对长时运行的构建，指示它建立并按节奏运行自己的检查框架（"建立一套边构建边检查自己工作的方法；每 [间隔] 运行一次，用子代理对照规格验证"）。独立的全新上下文校验子代理通常胜过自我批评。
- **De-prescribe migrated prompts and skills.** Prompts and skills written for prior models are often too prescriptive for Claude Fable 5.1 and *reduce* output quality. After migrating, A/B the workload with older step-by-step scaffolding removed - prefer stating the goal and constraints over enumerating the steps. Claude Fable 5.1 is also good at updating skills on the fly from what it learns mid-task - let it.
  **给迁移来的提示词与技能松绑。**为先前模型写的提示词与技能对 Claude Fable 5.1 往往过于规定化，会*降低*输出质量。迁移后，把旧的逐步脚手架去掉对工作负载做 A/B——优先陈述目标与约束，而不是罗列步骤。Claude Fable 5.1 也擅长依据任务中习得的东西即时更新技能——放手让它做。
- **Start at the top of your difficulty range.** The teams with the best early-access outcomes gave it their hardest unsolved problems first - have it scope the problem, ask questions, then execute.
  **从你难度范围的最顶端开始。**早期访问成效最好的团队把最难的无解问题先交给它——让它先划定问题范围、提问，然后执行。
- **Add a `send_to_user` tool for verbatim mid-task delivery.** When an asynchronous agent must deliver something the user sees *exactly as written* mid-run (a deliverable, a progress update with specific numbers, a direct answer), give it a client-side tool whose input you render directly in the UI - tool inputs are never summarized, so content arrives intact. Return a simple acknowledgement as the tool result:
  **加一个 `send_to_user` 工具用于任务中途原样交付。**当异步智能体必须在运行中途交付用户*逐字看到*的东西（交付物、带具体数字的进度更新、直接答案）时，给它一个客户端工具，其输入由你直接渲染进 UI——工具输入从不被摘要，内容完整到达。工具结果返回一个简单的确认即可：

```json
{
  "name": "send_to_user",
  "description": "Display a message directly to the user. Use this for progress updates, partial results, or content the user must see exactly as written before the task finishes.",
  "input_schema": {
    "type": "object",
    "properties": {
      "message": { "type": "string", "description": "The content to display to the user." }
    },
    "required": ["message"]
  }
}
```

For agents that only narrate routine progress, the model's default progress narration is typically adequate without this tool.

对只叙述例行进度的智能体，模型的默认进度叙述通常就够，不需要这个工具。

### Claude Fable 5.1 Migration Checklist / Claude Fable 5.1 迁移清单

- [ ] **[BLOCKS]** Also apply the Claude Fable 5.1 from Claude Fable 5 Migration Checklist below - it carries the three breaking changes introduced after Claude Fable 5 (forced `tool_choice` 400s, model-bound thinking blocks, the history-editing check), which this checklist predates
      **[BLOCKS]** 还要应用下文"从 Claude Fable 5 迁移到 Claude Fable 5.1"的迁移清单——它带有 Claude Fable 5 之后引入的三项破坏性变更（强制 `tool_choice` 返回 400、模型绑定思考块、历史编辑检查），本清单早于它们
- [ ] **[BLOCKS]** Update the `model=` string to `claude-fable-5-1` (`claude-mythos-5-1` for Mythos Preview migrators in Project Glasswing)
      **[BLOCKS]** 把 `model=` 字符串更新为 `claude-fable-5-1`（Project Glasswing 中的 Mythos Preview 迁移者用 `claude-mythos-5-1`）
- [ ] **[BLOCKS]** Remove `thinking: {type: "disabled"}` (errors on Claude Fable 5.1)
      **[BLOCKS]** 移除 `thinking: {type: "disabled"}`（在 Claude Fable 5.1 上报错）
- [ ] **[BLOCKS]** Replace assistant prefill with structured outputs or system prompt instructions
      **[BLOCKS]** 用结构化输出或系统提示词指令替换 assistant prefill
- [ ] **[BLOCKS]** Confirm the org meets the 30-day data-retention requirement (ZDR orgs get `400 invalid_request_error` on every request; ZDR only if expressly authorized by Anthropic, or enable 30-day retention for one workspace)
      **[BLOCKS]** 确认组织满足 30 天数据保留要求（ZDR 组织每个请求都会得到 `400 invalid_request_error`；ZDR 仅在 Anthropic 明确授权时可用，或为一个工作区启用 30 天保留）
- [ ] **[BLOCKS]** Remove all other `thinking` configuration (`{type: "enabled", budget_tokens: N}` returns a 400, same as on Opus 4.7/4.8); control depth with `output_config.effort` instead
      **[BLOCKS]** 移除所有其他 `thinking` 配置（`{type: "enabled", budget_tokens: N}` 返回 400，与 Opus 4.7/4.8 相同）；改用 `output_config.effort` 控制深度
- [ ] **[BLOCKS]** If thinking content is surfaced to users or stored in logs: add `thinking: {type: "adaptive", display: "summarized"}` (the default is `"omitted"` - otherwise the rendered text is empty)
      **[BLOCKS]** 若思考内容呈现给用户或写入日志：添加 `thinking: {type: "adaptive", display: "summarized"}`（默认为 `"omitted"`——否则渲染文本为空）
- [ ] **[TUNE]** Re-baseline cost and latency on your own workloads - token counts are roughly unchanged from Opus 4.7/4.8 and Mythos Preview (same tokenizer); per-token pricing differs. Coming from Opus 4.6, Sonnet, Haiku, or older, token counts differ - use `count_tokens` with each model to compare
      **[TUNE]** 在自己的工作负载上重新定基成本与延迟——相对 Opus 4.7/4.8 与 Mythos Preview 的 token 计数大致不变（同一分词器）；单 token 定价不同。来自 Opus 4.6、Sonnet、Haiku 或更早则 token 计数不同——对每个模型用 `count_tokens` 比较
- [ ] **[TUNE]** Add `stop_reason == "refusal"` handling before reading `response.content` (pre-output: empty, billing per [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed); mid-stream: billed at normal rates - discard the partial); opt into a fallback by default - server-side `fallbacks` (Claude API and Claude Platform on AWS: `fallbacks: "default"` with `server-side-fallback-2026-07-01`, or the array form with `server-side-fallback-2026-06-01`) where available, otherwise the SDK middleware or fallback credit (`fallback-credit-2026-07-01`, exact body); a bare client-side replay (history as-is; models other than Claude Fable 5.1 / Claude Mythos 5.1 drop Fable's thinking blocks) is the floor, not the recommendation
      **[TUNE]** 在读取 `response.content` 前添加 `stop_reason == "refusal"` 处理（输出前：内容为空，计费见[拒答如何计费](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)；流中：按正常费率计费——丢弃部分内容）；默认选入回退——可用处用服务端 `fallbacks`（Claude API 与 Claude Platform on AWS：`fallbacks: "default"` 配 `server-side-fallback-2026-07-01`，或 `server-side-fallback-2026-06-01` 的数组形式），否则用 SDK 中间件或回退积分（`fallback-credit-2026-07-01`，请求体完全一致）；裸的客户端重放（历史原样；Claude Fable 5.1 / Claude Mythos 5.1 以外的模型会丢弃 Fable 的思考块）是底线，不是推荐
- [ ] **[TUNE]** If you surfaced thinking text to users, plan for the thinking output change - the raw chain of thought is never returned; render the `display: "summarized"` summary (per the [BLOCKS] item above); pass blocks back unchanged on the same model; other models drop them from the prompt (unbilled; Claude Mythos 5.1 instead reads them)
      **[TUNE]** 若你把思考文本呈现给用户，为思考输出的变化做规划——原始思维链从不返回；渲染 `display: "summarized"` 的摘要（按上文 [BLOCKS] 条目）；同一模型上把块原样传回；其他模型会把它们从提示词中丢弃（不计费；Claude Mythos 5.1 则会读取）
- [ ] **[TUNE]** Plan for minutes-long turns: timeouts, streaming, async check-ins, progress UX (see Behavior changes above)
      **[TUNE]** 为数分钟长的回合做规划：超时、流式、异步查看、进度 UX（见上文行为变化）
- [ ] **[TUNE]** Run an effort sweep including low/medium for routine workloads; add the no-tidying instruction if higher effort produces unrequested refactors
      **[TUNE]** 跑一轮 effort 扫描，对例常工作负载纳入 low/medium；若更高 effort 产生未经请求的重构，加"不整理"指令
- [ ] **[TUNE]** A/B with prior-model scaffolding removed - over-prescriptive prompts/skills reduce Claude Fable 5.1 output quality
      **[TUNE]** 去掉先前模型的脚手架做 A/B——过度规定化的提示词/技能会降低 Claude Fable 5.1 的输出质量

---

## Migrating to Claude Fable 5.1 from Claude Fable 5 / 从 Claude Fable 5 迁移到 Claude Fable 5.1

> **Model IDs `claude-fable-5-1` and `claude-mythos-5-1` are authoritative as written here.** When the user asks to migrate to Claude Fable 5.1, write `model="claude-fable-5-1"` exactly; a Project Glasswing participant migrating from Claude Mythos 5 writes `model="claude-mythos-5-1"`. Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entries exist in `shared/models.md`.

> **模型 ID `claude-fable-5-1` 与 `claude-mythos-5-1` 以此处所写为准。**当用户要求迁移到 Claude Fable 5.1 时，一字不差地写 `model="claude-fable-5-1"`；从 Claude Mythos 5 迁移的 Project Glasswing 参与者写 `model="claude-mythos-5-1"`。**不要** WebFetch 去核实——本指南就是迁移目标 ID 的事实来源。对应条目存在于 `shared/models.md` 中。

Claude Fable 5.1 succeeds Claude Fable 5 in the same tier at the same per-token price, with stronger long-running agentic coding, multistep research, and document / spreadsheet / slide work. **Claude Mythos 5.1** (`claude-mythos-5-1`) is the same model for Project Glasswing participants (see § Claude Mythos 5.1 below for how it differs). Same 1M token context window (default and maximum), same 128K max output, same tokenizer as Claude Fable 5 (token counts unchanged; coming from a pre-Opus-4.7 model, expect roughly 30% more tokens - follow the tokenizer guidance in § Migrating to Claude Fable 5.1 above). Available on the Claude API, Amazon Bedrock (`anthropic.claude-fable-5-1`), Claude Platform on AWS, Google Cloud, and Microsoft Foundry (Anthropic-hosted). Existing Claude Fable 5 prompts should perform well out of the box.

Claude Fable 5.1 在同一档位、相同单 token 价格上接替 Claude Fable 5，长时运行智能体编码、多步研究与文档/电子表格/幻灯片工作更强。**Claude Mythos 5.1**（`claude-mythos-5-1`）是 Project Glasswing 参与者可用的同一模型（差异见下文 § Claude Mythos 5.1）。同样的 1M token 上下文窗口（默认即最大），同样的 128K 最大输出，与 Claude Fable 5 相同的分词器（token 计数不变；来自 Opus 4.7 之前的模型则预期 token 多约 30%——遵循上文 § Migrating to Claude Fable 5.1 中的分词器指导）。可在 Claude API、Amazon Bedrock（`anthropic.claude-fable-5-1`）、Claude Platform on AWS、Google Cloud 与 Microsoft Foundry（Anthropic 托管）上使用。现有 Claude Fable 5 提示词应可开箱表现良好。

**Migrate to Claude Fable 5.1 only when the user explicitly chose it** - same rule as Claude Fable 5: it is not the default Opus upgrade path. For "upgrade to the latest model" requests, the target is `claude-opus-5-5`: the docs position Claude Opus 5.5 as the default for most work, including complex agentic coding, and Claude Fable 5.1 as the step up for the hardest long-running agentic and research tasks, or where evals on Claude Opus 5.5 at higher effort still fall short.

**只有当用户明确选择了 Claude Fable 5.1 时才迁移过去**——与 Claude Fable 5 相同的规则：它不是默认的 Opus 升级路径。"升级到最新模型"类请求的目标是 `claude-opus-5-5`：文档把 Claude Opus 5.5 定位为多数工作（包括复杂智能体编码）的默认，把 Claude Fable 5.1 定位为最困难的长时运行智能体与研究任务的进阶选择，或在 Claude Opus 5.5 较高 effort 下评测仍不达标时的选择。
**What changes, in one line:** three breaking changes (forced tool choice 400s; thinking blocks are preserved only for the model that produced them or a newer one; thinking blocks are preserved only in the conversation that produced them - the docs group the last two as "preserved thinking"), five additions (per-message effort, turn-scoped system messages, progress updates between tool calls, a lower cache-read price, content provenance), and agent-loop behavior that differs in three prompt-tunable ways. Read the path that matches the source model: from Claude Fable 5, everything below applies directly; from Claude Opus 5, also read § Coming from Claude Opus 5; from Opus 4.8 or earlier, apply § Migrating to Claude Fable 5.1 above first (Opus 4.7 or earlier: the Claude Opus 5 section before that), then this one.

一句话概括变更内容：三项破坏性变更（强制工具选择返回 400；思维块仅对产生它们的模型或更新的模型保留；思维块仅在产生它们的对话中保留——文档将后两项合称为"保留思维"（preserved thinking）），五项新增（每消息努力度、轮次作用域系统消息、工具调用之间的进度更新、更低的缓存读取价格、内容来源标识），以及代理循环在三处可通过提示词调整的方面出现的行为差异。请按源模型选择对应路径：从 Claude Fable 5 迁移，以下内容直接适用；从 Claude Opus 5 迁移，还需阅读 § Coming from Claude Opus 5；从 Opus 4.8 或更早版本迁移，先应用上文 § Migrating to Claude Fable 5.1（Opus 4.7 或更早：先看其前的 Claude Opus 5 一节），再应用本节。

### Breaking change 1: forced tool use is rejected / 破坏性变更 1：强制工具使用被拒绝

`tool_choice: {"type": "any"}` and `tool_choice: {"type": "tool", "name": "..."}` return a 400 `invalid_request_error` on Claude Fable 5.1 and Claude Mythos 5.1 - on the Messages API, the Message Batches API, and the token-counting endpoint:

在 Claude Fable 5.1 与 Claude Mythos 5.1 上，`tool_choice: {"type": "any"}` 与 `tool_choice: {"type": "tool", "name": "..."}` 会返回 400 `invalid_request_error`——涵盖 Messages API、Message Batches API 与 token 计数端点：

```text
tool_choice: type "tool" and "any" are not supported for this model.
```

This is a model-specific restriction, not a consequence of always-on thinking (Claude Fable 5 and Claude Opus 5 also think by default and still accept forced tool choice). `{"type": "auto"}` (the default) and `{"type": "none"}` are unchanged. `disable_parallel_tool_use: true` still works with `auto` but now means *at most* one call - the "exactly one tool" guarantee it gave in combination with `any`/`tool` is gone.

这是模型特有的限制，并非默认开启思维（always-on thinking）的后果（Claude Fable 5 与 Claude Opus 5 同样默认思考，但仍接受强制工具选择）。`{"type": "auto"}`（默认值）与 `{"type": "none"}` 保持不变。`disable_parallel_tool_use: true` 与 `auto` 搭配仍然有效，但现在的含义是*至多*一次调用——它此前与 `any`/`tool` 组合时提供的"恰好一次工具调用"保证已不复存在。

Migrate by intent:

按迁移意图区分：

- **Steering toward a tool:** keep `tool_choice: {"type": "auto"}` (or omit it) and state in the prompt when the tool applies ("Use the `get_weather` tool to answer"). Claude Fable 5.1 follows explicit tool instructions reliably, and thinking first improves the arguments it passes. If the *application* (not the user) requires a specific call on the current turn of a multi-turn conversation, append a `role: "system"` message after the latest `user` turn that names the tool, says the call is required for this turn, and tells Claude to open its response with it - and keep that message in the history on later requests.
  **引导模型使用某个工具：**保留 `tool_choice: {"type": "auto"}`（或省略它），并在提示词中说明该工具何时适用（"Use the `get_weather` tool to answer"）。Claude Fable 5.1 能可靠地遵循明确的工具指令，先思考还能改进它传入的参数。如果是*应用*（而非用户）在多轮对话的当前轮要求特定调用，则在最新一条 `user` 轮之后追加一条 `role: "system"` 消息，写明该工具、说明本轮必须进行该调用，并让 Claude 以该调用开始其响应——且在后续请求中把这条消息保留在历史里。
- **Guaranteeing schema-valid arguments:** the argument-validity guarantee `any` gave you comes back with strict tool use - `strict: true` on the tool definition (with `additionalProperties: false` in the schema) under `auto`. (In a CMEK organization, structured outputs including `strict: true` aren't available on Fable models - rely on the instruction alone.)
  **保证参数符合 schema：**`any` 曾提供的参数有效性保证由严格工具使用（strict tool use）找回——在 `auto` 下，在工具定义上设置 `strict: true`（schema 中含 `additionalProperties: false`）。（在 CMEK 组织中，包括 `strict: true` 在内的结构化输出在 Fable 模型上不可用——只能依赖指令本身。）
- **Extracting structured data:** if the forced call existed only to get JSON back, replace it with structured outputs (`output_config.format`) - see the prefill-replacement table under Breaking Changes by Source Model for the `messages.parse()` / `output_config.format` shapes.
  **提取结构化数据：**如果强制调用只是为了拿回 JSON，请改用结构化输出（`output_config.format`）——`messages.parse()` / `output_config.format` 的具体形态见 Breaking Changes by Source Model 下的预填充替换表。
- **Advisor tool:** a Claude Fable 5.1 or Claude Mythos 5.1 *executor* rejects forced `tool_choice` too, so nudge the advisor call from the prompt instead (see `shared/tool-use-concepts.md` § Advisor).
  **Advisor 工具：**Claude Fable 5.1 或 Claude Mythos 5.1 的*执行器*（executor）同样拒绝强制的 `tool_choice`，因此应改为从提示词中引导 advisor 调用（见 `shared/tool-use-concepts.md` § Advisor）。

```python
# Before - 400 on Claude Fable 5.1
response = client.messages.create(
    model="claude-fable-5",
    max_tokens=4096,
    tools=[get_weather_tool],
    tool_choice={"type": "tool", "name": "get_weather"},
    messages=[{"role": "user", "content": "Check Tokyo, then summarize."}],
)

# After - let it think, name the tool, keep the schema guarantee with strict tool use
get_weather_tool["strict"] = True   # schema must set additionalProperties: false
response = client.messages.create(
    model="claude-fable-5-1",
    max_tokens=4096,
    tools=[get_weather_tool],
    tool_choice={"type": "auto"},
    messages=[{"role": "user", "content": "Use the get_weather tool to check Tokyo, then summarize."}],
)
```

### Breaking change 2: thinking blocks are preserved only for the model that produced them, or a newer one / 破坏性变更 2：思维块仅对产生它们的模型或更新的模型保留

Every `thinking` block records which model produced it. Claude Fable 5.1 and Claude Mythos 5.1 read each other's blocks and those from Claude Opus 5.5, Claude Opus 5, Claude Fable 5, Claude Mythos 5, and earlier models that don't encrypt their reasoning in the signature (Opus 4.8 and earlier Opus, Sonnet, Haiku 4.5) - so a conversation that *moves onto* `claude-fable-5-1` keeps its earlier reasoning. They don't read Mythos Preview's blocks. **The binding is one-way: apart from Claude Mythos 5.1, no other model can read a Claude Fable 5.1 block.**

每个 `thinking` 块都记录了产生它的模型。Claude Fable 5.1 与 Claude Mythos 5.1 能读取彼此的块，以及来自 Claude Opus 5.5、Claude Opus 5、Claude Fable 5、Claude Mythos 5 与更早的不在签名中加密推理的模型（Opus 4.8 及更早的 Opus、Sonnet、Haiku 4.5）的块——因此*切换到* `claude-fable-5-1` 的对话会保留其先前的推理。它们读取不了 Mythos Preview 的块。**这种绑定是单向的：除 Claude Mythos 5.1 外，没有任何其他模型能读取 Claude Fable 5.1 的块。**

When a request carries a block the receiving model can't read - a router switch, a client-side retry on another model, a classifier refusal fallback (server-side or SDK middleware) - the API drops it before the model sees it: the request succeeds, the dropped block doesn't count toward `input_tokens` and isn't billed, and the target model re-plans without that reasoning (expect higher cost and latency on the first turn after a switch). A dropped block changes the cached prefix from its position onward on that request. Without the `thinking-binding-controls-2026-08-01` beta header the drop is silent; with it, the response carries a top-level `input_transformations` array naming each dropped block with `reason: "model_binding_mismatch"` (shape below).

当请求携带接收模型无法读取的块时——路由切换、客户端在另一模型上重试、分类器拒答回退（服务端或 SDK 中间件）——API 会在模型看到之前将其丢弃：请求成功，被丢弃的块不计入 `input_tokens`、不计费，目标模型在没有该推理的情况下重新规划（切换后的第一轮预计成本与延迟更高）。被丢弃的块会使该请求从其位置开始的缓存前缀发生变化。不带 `thinking-binding-controls-2026-08-01` beta 请求头时，丢弃是静默的；带上它，响应会携带顶层的 `input_transformations` 数组，以 `reason: "model_binding_mismatch"` 标明每个被丢弃的块（结构见下文）。

Keep passing thinking blocks back unchanged when you switch models - the API drops what the target can't read, unbilled, so there are no input tokens to save by stripping; removing blocks yourself can trigger ordering/signature 400s, and a fallback-credit retry must echo the refused body unchanged.

切换模型时请继续原样回传思维块——API 会丢弃目标模型读取不了的部分，且不计费，因此剥离它们省不下任何输入 token；自行移除块可能触发顺序/签名相关的 400，而回退积分（fallback-credit）重试必须原样回显被拒的请求体。

### Breaking change 3: thinking blocks are preserved only in the conversation that produced them / 破坏性变更 3：思维块仅在产生它们的对话中保留

The published docs file this and breaking change 2 together under *preserved thinking* ("pass blocks back unchanged and let the API decide which the model can use"); this one is the conversation check - editing earlier turns invalidates every later thinking block. The API field names for it say `prefix_mismatch_behavior` / `prefix_binding_mismatch` - the same check. **Claude Mythos 5.1 does not run this check** (breaking change 2, the model-binding check, still applies to it, and editing history still restarts the prompt cache).

官方文档将本项与破坏性变更 2 归入*保留思维*（preserved thinking）名下（"原样回传块，由 API 决定模型能用哪些"）；本项是其中的对话检查——编辑较早的轮次会使之后所有思维块失效。其对应的 API 字段名为 `prefix_mismatch_behavior` / `prefix_binding_mismatch`——是同一项检查。**Claude Mythos 5.1 不运行这项检查**（破坏性变更 2 的模型绑定检查对它仍然适用，且编辑历史仍会重启提示词缓存）。

To find and fix these edits in an existing harness - capture its requests, diff them, measure the drops, then one diff per cause - follow `shared/preserved-thinking-migration.md` (the `preserved-thinking-migration` subcommand). This section holds the rules that guide applies.

要在现有 harness 中找出并修复这类编辑——捕获其请求、做 diff、测量丢弃量、再按成因逐一 diff——请遵循 `shared/preserved-thinking-migration.md`（`preserved-thinking-migration` 子命令）。本节收录了该指南所依据的规则。

A Claude Fable 5.1 thinking block's `signature` also records the conversation prefix that produced it - the top-level `system` prompt, the set of tools in `tools`, and every message before the block (with server-side compaction, the prefix starts at the most recent compaction block) - plus a chain to the previous thinking block across turns (earlier thinking blocks aren't part of the prefix, but each block records the one before it, which is why blocks can be removed from the *front* of the history and not from the middle). When the transcript comes back, the API checks that this prefix is unchanged. Claude Code, claude.ai, Managed Agents, and the Agent SDK keep the prefix intact for you; **if your code builds the `messages` array itself, check it before migrating** (the three-step check is below). **Who is enforced:** new accounts **created on or after August 31, 2026**, on every platform. Enforcement scope is decided per model - Claude Opus 5.5 also enforces it for new accounts only - so make your application compatible regardless of your account's age: the same patterns keep the prompt cache warm, and you can test against the check from any account by sending `prefix_mismatch_behavior`. For accounts created earlier the API *records* the mismatch but acts on it only when the request sets `thinking.block_binding.prefix_mismatch_behavior` - **any value, including `"error"`, opts the request into enforcement**, which is also how you test from an older organization (the beta header alone does not opt in: it lets you set the field and, on a request that leaves the field unset, lists each failing block in `input_transformations` as `thinking_mismatch_allowed` while the model still receives it). If you ship a tool or framework that people run with their own API key, test with the field set: your users on new organizations are enforced before you are. To see whether your own organization is enforced by default, send a request that edits history without the beta header - a 400 that names the header means it is. Platform note: the opt-in controls (the beta header, `prefix_mismatch_behavior`, `input_transformations`) are available under the same beta name on the Claude API, Claude Platform on AWS, Amazon Bedrock, and Google Cloud Vertex AI (Bedrock: the `anthropic_beta` body field; Vertex: the `anthropic-beta` HTTP header - the SDKs' `betas` parameter does the right thing on each); Microsoft Foundry is unconfirmed. Wherever an endpoint rejects the header or the beta name, the opt-in test path doesn't apply and recovery is strip-and-retry (`shared/platform-availability.md` has the matrix).

Claude Fable 5.1 思维块的 `signature` 还记录了产生它的对话前缀——顶层 `system` 提示词、`tools` 中的工具集合，以及该块之前的每一条消息（使用服务端压缩时，前缀从最近的压缩块开始）——外加一条跨轮次指向前一个思维块的链（较早的思维块不属于前缀，但每个块都会记录它前面的那个块，这就是为什么块可以从历史*开头*移除而不能从中间移除）。当对话记录回传时，API 会检查这一前缀是否未变。Claude Code、claude.ai、Managed Agents 与 Agent SDK 会替你保持前缀完整；**如果你的代码自行构建 `messages` 数组，请在迁移前先行检查**（三步检查见下文）。**谁会被强制执行：**在所有平台上，**创建于 2026 年 8 月 31 日或之后**的新账户。执行范围按模型决定——Claude Opus 5.5 同样只对新账户执行——因此无论你的账户多老，都要让应用保持兼容：同样的模式能让提示词缓存保持温热，而且任何账户都可以通过发送 `prefix_mismatch_behavior` 来针对这项检查做测试。对更早创建的账户，API 会*记录*这种不匹配，但只有当请求设置了 `thinking.block_binding.prefix_mismatch_behavior` 时才会据此行动——**任何取值，包括 `"error"`，都会让该请求加入强制执行**，这也是从较老组织进行测试的方法（仅 beta 请求头并不会加入强制执行：它只是允许你设置该字段，并且在字段未设置时，把每个未通过检查的块以 `thinking_mismatch_allowed` 列入 `input_transformations`，而模型仍会收到它）。如果你发布的是让别人用自己的 API key 运行的工具或框架，请带上该字段测试：新组织上的用户会先于你被强制执行。要确认你自己的组织是否默认被强制执行，可以不带 beta 请求头发送一个会编辑历史的请求——如果返回的 400 指名了该请求头，即表示已被强制执行。平台说明：这些加入执行的控制项（beta 请求头、`prefix_mismatch_behavior`、`input_transformations`）在 Claude API、Claude Platform on AWS、Amazon Bedrock 与 Google Cloud Vertex AI 上以相同的 beta 名称提供（Bedrock：请求体中的 `anthropic_beta` 字段；Vertex：`anthropic-beta` HTTP 请求头——SDK 的 `betas` 参数在两者上都会做正确处理）；Microsoft Foundry 尚未确认。凡是拒绝该请求头或 beta 名称的端点，加入执行的测试路径不适用，恢复方式是剥离后重试（矩阵见 `shared/platform-availability.md`）。
【评论】以账户创建日期划定强制执行范围是一种常见的灰度发布策略：新账户默认收紧，旧账户通过显式设置字段选择加入，以降低存量集成被一次性破坏的风险。

**What invalidates every later thinking block:**

**会使之后所有思维块失效的操作：**

- Editing, reordering, or removing an earlier turn while keeping later ones - including deleting old tool results (use server-side tool-result clearing instead).
  在保留后续轮次的同时编辑、重排或删除较早的轮次——包括删除旧的工具结果（应改用服务端工具结果清除）。
- Injecting per-request text into an earlier turn (a reminder, a status line, a token count) that you remove or rebuild on the next request.
  向较早轮次注入每次请求都会变化的文本（提醒、状态行、token 计数），并在下一次请求中移除或重建它。
- Rebuilding the top-level `system` prompt or `tools` array between requests in the same conversation.
  在同一对话的各次请求之间重建顶层 `system` 提示词或 `tools` 数组。
- Removing a thinking block from anywhere other than the start of the run (see below).
  从运行开头以外的任何位置移除思维块（见下文）。
- An image or document URL in an earlier turn that serves different bytes on a later request - the bytes are bound, not the URL string, so a rotating signed URL for the same file is fine; for content referenced across turns, upload it once with the Files API and send the `file_id`, or send base64.
  较早轮次中的图片或文档 URL 在后续请求中返回了不同的字节——绑定的是字节而非 URL 字符串，因此同一文件的轮换签名 URL 没有问题；跨轮次引用的内容，用 Files API 上传一次并发送 `file_id`，或发送 base64。

**What keeps later blocks valid:** append-only histories, including appended `role: "system"` messages and cleared turn-scoped (`clear_at`) messages or reminder text blocks left in place; removing a *leading* run of thinking blocks, oldest first (the first block in the conversation - or the first after the most recent compaction block - then the next, and so on); reordering `tools` without changing them (bound as a name-sorted set; confirm at launch) and adding a `defer_loading: true` tool nothing has referenced yet; changing any request parameter outside `system` / `tools` / `messages` (`max_tokens`, `output_config` incl. `effort`, `tool_choice`, `metadata`); adding, moving, or removing `cache_control` markers; a rotating signed URL that returns the same bytes; server-side compaction and context editing, including thinking-block clearing (they don't count as edits, because the check compares the conversation *as you sent it*, not the server's edited copy; after a compaction the checked prefix starts from the compaction block).

**能让后续块保持有效的操作：**只追加（append-only）的历史，包括追加的 `role: "system"` 消息，以及被清除的轮次作用域（`clear_at`）消息或原地保留的提醒文本块；移除*开头*连续的一段思维块，从最旧的开始（对话中的第一个块——或最近一个压缩块之后的第一个块——然后是下一个，依此类推）；在不改变工具本身的情况下重排 `tools`（按名称排序的集合绑定；发布时需确认），以及添加一个尚未被任何内容引用的 `defer_loading: true` 工具；更改 `system` / `tools` / `messages` 之外的任何请求参数（`max_tokens`、含 `effort` 的 `output_config`、`tool_choice`、`metadata`）；添加、移动或移除 `cache_control` 标记；返回相同字节的轮换签名 URL；服务端压缩与上下文编辑，包括思维块清除（它们不算编辑，因为检查比较的是*你发送时*的对话，而非服务器编辑后的副本；压缩之后，被检查的前缀从压缩块开始）。

**Where the check is enforced, a request that replays an invalidated block is rejected** with a 400 `invalid_request_error`, decided before any output. Retrying the same body fails the same way; the token-counting endpoint runs the same check. (In the Message Batches API the *unset* default drops failing blocks instead of failing the item - set `"error"` explicitly if you want batch items to error.)

**在强制执行该检查的地方，重放已失效块的请求会被拒绝**，返回 400 `invalid_request_error`，且在任何输出之前即作出判定。用相同请求体重试会以同样方式失败；token 计数端点运行同样的检查。（在 Message Batches API 中，*未设置*的默认值是丢弃未通过的块而不是让该条目失败——如果你希望批处理条目报错，请显式设置 `"error"`。）

```text
messages.5.content.0: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block". That setting requires the `thinking-binding-controls-2026-08-01` value in the `anthropic-beta` header.
```

The last sentence appears only when the request didn't send the beta header; the message can end with one more sentence naming the first message that changed - the actionable diagnostic. (A tampered or undecryptable signature is a different failure: the same leading clause with *no* "bound to a different conversation" sentence, always a 400, and `prefix_mismatch_behavior` doesn't apply.) Two recoveries:

最后一句只在请求未发送 beta 请求头时出现；错误消息还可能以另一句结尾，指出第一条发生变化的消息——这是最具可操作性的诊断信息。（被篡改或无法解密的签名是另一种失败：同样的开头子句，但*没有*"绑定到不同对话"那一句，始终是 400，且 `prefix_mismatch_behavior` 不适用。）有两种恢复方式：

1. **Strip every `thinking` and `redacted_thinking` block from the history** (each turn's `text` and `tool_use` blocks stay), then retry once - the no-beta path. The model answers that turn without the reasoning those blocks carried. Dropping thinking once, at a boundary such as a compaction, has little effect; an integration that invalidates its own history on every request loses that reasoning and restarts the prompt cache each time, which can raise cost per task. Treat this as a one-time recovery, not a steady-state pattern.
   **从历史中剥离所有 `thinking` 与 `redacted_thinking` 块**（每轮的 `text` 与 `tool_use` 块保留），然后重试一次——这是不带 beta 的路径。模型回答该轮时将没有这些块所承载的推理。在压缩这类边界处丢弃一次思维影响很小；但每次请求都使自己历史失效的集成会丢失这些推理并每次重启提示词缓存，从而可能抬高每个任务的成本。请将其视为一次性恢复手段，而非稳态模式。
2. **Ask the API to drop instead of erroring:**
   **让 API 丢弃而不是报错：**

```http
POST /v1/messages
anthropic-beta: thinking-binding-controls-2026-08-01

{"model": "claude-fable-5-1", "max_tokens": 4096,
 "thinking": {"type": "adaptive", "block_binding": {"prefix_mismatch_behavior": "drop_block"}},
 "messages": [ ...full history with thinking blocks replayed verbatim... ]}
```

`thinking.block_binding.prefix_mismatch_behavior` takes `"error"` or `"drop_block"`. On an enforced account the default is `"error"` with or without the header (the header only lets you set the field, and adds `input_transformations` to responses). On an account that isn't enforced, an unset field lets failing blocks through to the model, and with the header each one is listed in `input_transformations` as an entry of type `thinking_mismatch_allowed`. In the Message Batches API the unset default on an enforced account drops the failing blocks instead of failing the item; a Batches item fails as `errored` only with `prefix_mismatch_behavior: "error"` set. Set the field explicitly. With `"drop_block"` the API drops the first mismatched block **and every thinking block after it** (up to the next compaction block, if any - including blocks in an assistant turn whose `tool_use` is still waiting on its `tool_result`), the request proceeds, and each drop is reported in the response's top-level `input_transformations` array:

`thinking.block_binding.prefix_mismatch_behavior` 接受 `"error"` 或 `"drop_block"`。在被强制执行的账户上，无论是否带该请求头，默认值都是 `"error"`（该请求头只是让你能设置这个字段，并让响应附带 `input_transformations`）。在未被强制执行的账户上，字段未设置时未通过的块会直达模型；带上请求头时，每个这样的块都会以 `thinking_mismatch_allowed` 类型的条目列入 `input_transformations`。在 Message Batches API 中，被强制执行账户上未设置的默认值是丢弃未通过的块而不是让条目失败；只有显式设置 `prefix_mismatch_behavior: "error"` 时，Batches 条目才会以 `errored` 失败。请显式设置该字段。使用 `"drop_block"` 时，API 会丢弃第一个不匹配的块**以及其后的所有思维块**（直到下一个压缩块为止，如有的话——包括其 `tool_use` 仍在等待 `tool_result` 的 assistant 轮中的块），请求继续进行，每次丢弃都会在响应顶层的 `input_transformations` 数组中报告：

```js
"input_transformations": [
  {"type": "thinking_dropped", "path": "messages.1.content.0", "reason": "prefix_binding_mismatch"}
]
```

The drop applies to *that request only*: keep sending `"drop_block"` for the rest of the session, or remove the failing blocks from the history yourself. On a `"type": "thinking_dropped"` entry, `reason` is `"prefix_binding_mismatch"` (your history changed) or `"model_binding_mismatch"` (the conversation switched models - not a bug in your code). The other entry type, `thinking_mismatch_allowed` (always `reason: "prefix_binding_mismatch"`), marks a block that failed the prefix check but still reached the model, on a request the API doesn't enforce. Ignore entries whose `type` or `reason` you don't recognize, because later checks add values. With the header, every response from a thinking-capable model carries the array (empty when no block was dropped and none failed the prefix check, never `null`); without it the field is absent. When streaming it arrives on the `message` object in `message_start` (and again in the final `message_delta` after a mid-stream server-side fallback). Sending `block_binding` without the header is a 400 ending in `block_binding: Extra inputs are not permitted`. The object is accepted alongside `thinking.type: "adaptive"` and `"enabled"`, and models that don't enforce the conversation check accept it and report only model-check drops, so one request body works across models. The launch SDKs type it in the beta namespace (`client.beta.messages.create(..., thinking={"type": "adaptive", "block_binding": {"prefix_mismatch_behavior": "drop_block"}}, betas=["thinking-binding-controls-2026-08-01"])`; typed enum names such as `PrefixMismatchBehavior` are open at launch - fall back to `extra_body` / a cast if the field isn't typed yet). Some older tooling spells the field `block_binding.mismatch_behavior` - an undocumented alias; write the canonical name and never send both.

丢弃只适用于*该次请求*：会话其余部分继续发送 `"drop_block"`，或者自行从历史中移除未通过的块。对于 `"type": "thinking_dropped"` 条目，`reason` 为 `"prefix_binding_mismatch"`（你的历史发生了变化）或 `"model_binding_mismatch"`（对话切换了模型——并非你代码中的 bug）。另一种条目类型 `thinking_mismatch_allowed`（`reason` 恒为 `"prefix_binding_mismatch"`）标记的是未通过前缀检查但仍然到达了模型的块，出现在 API 不强制执行的请求上。不认识的 `type` 或 `reason` 条目请忽略，因为后续的检查会新增取值。带请求头时，来自支持思维模型的所有响应都携带该数组（没有块被丢弃且没有块未通过前缀检查时为空，绝不会是 `null`）；不带时该字段不存在。流式传输时，它出现在 `message_start` 的 `message` 对象上（在流中途发生服务端回退后，最终 `message_delta` 中会再次出现）。不带请求头发送 `block_binding` 会得到 400，错误信息以 `block_binding: Extra inputs are not permitted` 结尾。该对象可与 `thinking.type: "adaptive"` 及 `"enabled"` 一同接受，而不执行对话检查的模型会接受它并只报告模型检查的丢弃，因此同一请求体可跨模型使用。发布版 SDK 在 beta 命名空间中为它提供了类型（`client.beta.messages.create(..., thinking={"type": "adaptive", "block_binding": {"prefix_mismatch_behavior": "drop_block"}}, betas=["thinking-binding-controls-2026-08-01"])`；`PrefixMismatchBehavior` 等类型化枚举名称在发布时尚未确定——若字段尚未类型化，回退到 `extra_body` / 强制转换）。一些较旧的工具把它拼作 `block_binding.mismatch_behavior`——这是一个未写入文档的别名；请写规范名称，且绝不两者同发。

**The three-step check for an existing integration:**

**针对现有集成的三步检查：**

1. Capture the exact request bodies it sends over a few normal turns, including a compaction or a tool change if the product has them. For each pair of consecutive requests, compare the `system` prompt, the `tools` array, and the shared prefix of `messages` - they should be byte-identical up to the newly appended turns.
   捕获它在几个正常轮次中发送的确切请求体，若产品有压缩或工具变更，也一并纳入。对每一对相邻请求，比较 `system` 提示词、`tools` 数组与 `messages` 的共享前缀——直到新追加的轮次为止，它们应当逐字节相同。
2. Run a normal multi-turn session against `claude-fable-5-1` with the `thinking-binding-controls-2026-08-01` header and `prefix_mismatch_behavior: "drop_block"`, and log `input_transformations` on every response. An empty array on every turn means the history is intact; a `prefix_binding_mismatch` entry means something before the block at `path` changed since the previous request; a `model_binding_mismatch` entry means the conversation switched models. This works from any organization wherever the controls beta is accepted (see the platform note above: if a partner endpoint rejects the beta name as unknown, use strip-and-retry; Foundry unconfirmed), because setting the field opts the request into enforcement. In CI, set `"error"` instead so an edit fails the run.
   带 `thinking-binding-controls-2026-08-01` 请求头与 `prefix_mismatch_behavior: "drop_block"`，对 `claude-fable-5-1` 跑一次正常的多轮会话，并记录每次响应的 `input_transformations`。每一轮都是空数组说明历史完整；出现 `prefix_binding_mismatch` 条目说明 `path` 所指块之前的内容自上一个请求后发生了变化；出现 `model_binding_mismatch` 条目说明对话切换了模型。凡是接受该 controls beta 的地方，这一方法在任何组织都可行（见上文平台说明：如果合作方端点将 beta 名称当作未知而拒绝，就用剥离后重试；Foundry 未确认），因为设置该字段即让请求加入强制执行。在 CI 中请改设 `"error"`，让历史编辑直接导致运行失败。
3. Choose a production setting and **set it explicitly** rather than relying on the default (see the defaults note above): `"error"` if a prefix mismatch can only mean a bug in your code, or `"drop_block"` to degrade instead of fail - and monitor the 400s or the `input_transformations` entries either way. On an account that isn't enforced (created before 2026-08-31), sending the header with the field unset is a way to monitor production traffic, not one of the two settings above: failing blocks still reach the model, and each one shows up only as a `thinking_mismatch_allowed` entry in `input_transformations`.
   选定一个生产环境取值并**显式设置**，不要依赖默认值（默认值说明见上文）：如果前缀不匹配只可能是你代码中的 bug，用 `"error"`；想降级而不是失败，用 `"drop_block"`——无论哪种都要监控 400 或 `input_transformations` 条目。在未被强制执行的账户上（2026-08-31 之前创建），发送请求头但字段留空是一种监控生产流量的手段，而不是上述两个取值之一：未通过的块仍会到达模型，每个块只会以 `thinking_mismatch_allowed` 条目出现在 `input_transformations` 中。

**Making a harness compatible - replace each transcript edit with its append-only form:**

**让 harness 兼容——把每种对话记录编辑替换为它的只追加形式：**

| You were doing | Do this instead |
|---|---|
| Editing the system prompt mid-session | Freeze the top-level `system` at session start; append a `{"role": "system", "content": "..."}` message at the point where the change becomes true (GA, no header; see `shared/prompt-caching.md` § Mid-conversation system messages). It gets system-prompt authority and becomes part of the prefix later blocks are locked to. |
| Editing the `tools` array mid-session | Declare the full set in `tools` at session start (`defer_loading: true` on the ones that start hidden) and send `tool_addition` / `tool_removal` blocks in a `role: "system"` message (beta `mid-conversation-tool-changes-2026-07-01`; `shared/tool-use-concepts.md` § Mid-conversation tool changes). |
| Injecting a per-turn reminder and deleting it next request | Send it as a turn-scoped system message (`clear_at: "next_user_message"`, addition 2 below) after the `tool_result` message and leave it in the history; without that beta, a text block after the `tool_result` blocks in the same user message, earlier copies left in place. |
| Deleting old tool results / snipping old turns client-side | Server-side context editing (tool-result clearing, thinking clearing) or compaction - they don't count as edits (the check compares the conversation as you sent it). |
| Compacting | Prefer server-side compaction (beta `compact-2026-01-12`; its `instructions` parameter takes your own summarization prompt) or context editing - neither counts as an edit. Client-side, **simple compaction** is the recommended shape: when the conversation grows too long, summarize it into a single message, start the next request with that summary plus the new user turn, and replay nothing else - no earlier turns, no earlier thinking blocks. Nothing carried over is tied to the old transcript; Claude models are trained on long-horizon tasks with this scheme and it performs comparably to more elaborate ones. Any compaction resets the cache, and don't compact in the middle of a tool round (an assistant turn whose `tool_use` is still waiting on its `tool_result` should go back with its thinking intact). Thinking from before the summary isn't carried forward, so the summary is all the model has of that work - tell the summarizer what to retain (the compaction prompt under Behavioral shifts, or server-side compaction's `instructions`). |
| Referencing an image/document by URL across turns | Upload once to the Files API and send the `file_id`, or send base64. |

| 你此前在做的 | 请改为这样做 |
|---|---|
| 会话中途编辑系统提示词 | 在会话开始时冻结顶层 `system`；在变更成立之处追加一条 `{"role": "system", "content": "..."}` 消息（正式可用，无需请求头；见 `shared/prompt-caching.md` § Mid-conversation system messages）。它获得系统提示词级别的权威，并成为后续块所锁定前缀的一部分。 |
| 会话中途编辑 `tools` 数组 | 会话开始时在 `tools` 中声明完整集合（对起初隐藏的工具设置 `defer_loading: true`），并在一条 `role: "system"` 消息中发送 `tool_addition` / `tool_removal` 块（beta `mid-conversation-tool-changes-2026-07-01`；见 `shared/tool-use-concepts.md` § Mid-conversation tool changes）。 |
| 注入每轮提醒并在下一次请求时删除 | 把它作为轮次作用域系统消息（`clear_at: "next_user_message"`，见下文新增项 2）在 `tool_result` 消息之后发送，并把它留在历史里；不带该 beta 时，作为同一用户消息中 `tool_result` 块之后的文本块，较早的副本原地保留。 |
| 客户端删除旧工具结果 / 剪掉旧轮次 | 服务端上下文编辑（工具结果清除、思维清除）或压缩——它们不算编辑（检查比较的是你发送时的对话）。 |
| 压缩（compaction） | 优先使用服务端压缩（beta `compact-2026-01-12`；其 `instructions` 参数接受你自己的摘要提示词）或上下文编辑——两者都不算编辑。客户端侧，**简单压缩**是推荐形态：当对话过长时，把它摘要成一条消息，下一次请求以该摘要加新用户轮开始，其余一概不重放——不重放更早的轮次，也不重放更早的思维块。任何带入的内容都不与旧对话记录绑定；Claude 模型就是按这一方案在长程任务上训练的，其表现与更复杂的方案相当。任何压缩都会重置缓存，且不要在工具轮中间压缩（`tool_use` 仍在等待 `tool_result` 的 assistant 轮应带着其思维原样回传）。摘要之前的思维不会向前携带，因此摘要是模型对那段工作的全部记忆——告诉摘要器要保留什么（见 Behavioral shifts 下的压缩提示词，或服务端压缩的 `instructions`）。 |
| 跨轮次通过 URL 引用图片/文档 | 用 Files API 上传一次并发送 `file_id`，或发送 base64。 |

Two client-side compaction shapes **break** under the check. *Keep-tail compaction* (summarize older turns, keep the most recent turns verbatim) fails on the retained turns: their thinking blocks were created with the full history present, so replaying them after the summary returns a 400 even though the retained turns are unchanged - strip the thinking blocks from the retained turns (text and tool calls can stay) or set `"drop_block"`. *Background (async) compaction* (compact off the critical path and swap the summary in while the conversation continues) fails the same way but affects more of the transcript: by the time the summary lands, several newer turns exist above the swap point and all of their thinking blocks predate it - send `"drop_block"` on every request that still carries pre-swap thinking blocks (or strip those blocks yourself; `input_transformations` on the first response after the swap lists exactly which ones), or compact synchronously. The same applies to the compaction beta's `pause_after_compaction` flow if you re-insert assistant turns after the compaction block: remove their `thinking` blocks or send `"drop_block"`. Snipping individual turns out of the *middle* of the transcript invalidates every later thinking block, and no client-side shape avoids it - use a mid-conversation system message for the instruction change you were making, or server-side context editing for selective removal.

两种客户端压缩形态在该检查下会**失效**。*保留尾部压缩*（keep-tail compaction，把较早轮次做成摘要、最近几轮逐字保留）会在被保留的轮次上失败：它们的思维块是在完整历史在场时创建的，因此在摘要之后重放它们，即使被保留的轮次未变也会返回 400——从被保留轮次中剥离思维块（文本和工具调用可以保留），或者设置 `"drop_block"`。*后台（异步）压缩*（在不阻塞关键路径的地方压缩，并在对话继续进行时换入摘要）以同样方式失败但影响更大的记录范围：等摘要落地时，换入点之上已存在若干更新的轮次，它们的思维块全部早于该点——在每个仍携带换入前思维块的请求上发送 `"drop_block"`（或自行剥离那些块；换入后第一个响应的 `input_transformations` 会准确列出是哪些），或者改为同步压缩。如果你在压缩块之后重新插入 assistant 轮，同样的结论也适用于压缩 beta 的 `pause_after_compaction` 流程：移除它们的 `thinking` 块或发送 `"drop_block"`。从记录*中间*剪掉个别轮次会使之后所有思维块失效，且没有任何客户端形态可以规避——你原本想做的指令变更请改用会话中途系统消息，选择性移除请改用服务端上下文编辑。

### What carries over unchanged from Claude Fable 5 / 从 Claude Fable 5 原样沿用的部分

The API surface, limits, per-token pricing, tokenizer, always-on adaptive thinking, refusal handling, and `stop_details` categories all match Claude Fable 5: no `thinking` config other than `{type: "adaptive"}` (`disabled` and `budget_tokens` both 400), `display` defaults to `"omitted"` and the raw chain of thought is never returned, interleaved thinking is automatic (no header), no assistant prefill, no non-default sampling parameters, 512-token minimum cacheable prompt, mid-conversation system messages and tool changes supported. The `refusal` stop reason must be handled before reading `content` - the classifiers cover the same categories as Claude Fable 5 (a broader set than Claude Opus 5's cyber-only classifiers), so expect `stop_details.category` values `"bio"` and `"reasoning_extraction"` as well as `"cyber"`. Deltas:

API 表面、限额、按 token 计价、分词器、默认开启的自适应思维、拒答处理与 `stop_details` 类别都与 Claude Fable 5 一致：除 `{type: "adaptive"}` 外不接受任何 `thinking` 配置（`disabled` 与 `budget_tokens` 都返回 400），`display` 默认为 `"omitted"` 且原始思维链绝不返回，交错思维自动启用（无需请求头），不支持 assistant 预填充，不支持非默认采样参数，可缓存提示词最少 512 token，支持会话中途系统消息与工具变更。`refusal` 停止原因必须在读取 `content` 之前处理——分类器覆盖与 Claude Fable 5 相同的类别（比 Claude Opus 5 仅覆盖 cyber 的分类器集合更广），因此除了 `"cyber"`，还要预期 `stop_details.category` 出现 `"bio"` 与 `"reasoning_extraction"`。差异点：

- **Fallbacks:** server-side `fallbacks` (`"default"`, or the array form) and the SDK middleware work as on Claude Fable 5; the permitted targets are `claude-opus-4-8` and `claude-opus-5`, and per-category routing is applied server-side and not published (some categories decline with no fallback). The fallback model can't read Claude Fable 5.1's thinking blocks, so the API drops them (breaking change 2). Fallback credit works as on Claude Fable 5: Claude Fable 5.1 and Claude Mythos 5.1 both mint a `fallback_credit_token` on refusals (the token is `null` when no credit is available for a refusal, so always handle `null`), redeemable on either permitted target (pattern 3 of the refusal section in § Migrating to Claude Fable 5.1 above); the credit refunds the prompt-cache cost of switching models. A mid-stream refusal is billed at normal rates; for a refusal before any output, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed).
  **回退（Fallbacks）：**服务端 `fallbacks`（`"default"` 或数组形式）与 SDK 中间件的行为与 Claude Fable 5 上相同；允许的目标是 `claude-opus-4-8` 与 `claude-opus-5`，按类别的路由在服务端应用且不公开（某些类别会直接拒答而无回退）。回退模型读取不了 Claude Fable 5.1 的思维块，因此 API 会丢弃它们（破坏性变更 2）。回退积分与 Claude Fable 5 上相同：Claude Fable 5.1 与 Claude Mythos 5.1 在拒答时都会铸造 `fallback_credit_token`（当某次拒答没有可用积分时该 token 为 `null`，因此务必处理 `null`），可在任一允许目标上兑换（见上文 § Migrating to Claude Fable 5.1 拒答小节的模式 3）；该积分会退还切换模型产生的提示词缓存成本。流中途的拒答按正常费率计费；输出之前的拒答见 [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。
- **Data retention:** Claude Fable 5.1 and Claude Mythos 5.1 are Covered Models like Claude Fable 5 - 30-day retention required, **not available under zero data retention unless expressly authorized by Anthropic**. As on Claude Fable 5, a request from an organization or workspace without 30-day retention returns `400 invalid_request_error` ("In order to access this model, your organization or workspace must have data retention enabled.") - check the retention configuration before debugging the payload. (An earlier draft of the launch docs described a 404 with the model hidden from `/v1/models`; the final wording is the 400. If you do see a 404 on the ID, check retention before anything else.) A ZDR organization that needs the model should contact its Anthropic account team (the "expressly authorized" path) or enable 30-day retention for one workspace; a ZDR org that *can* already reach the model has such an authorization, not proof the requirement is gone. (Earlier drafts of the launch docs described a time-bound enterprise exemption through 2026-12-31; that sentence was removed on Aug 28 - don't cite it.)
  **数据保留：**与 Claude Fable 5 一样，Claude Fable 5.1 与 Claude Mythos 5.1 属于受覆盖模型（Covered Models）——要求 30 天保留，**除非获得 Anthropic 明确授权，否则零数据保留（ZDR）下不可用**。与 Claude Fable 5 相同，没有 30 天保留的组织或工作区发出的请求会返回 `400 invalid_request_error`（"In order to access this model, your organization or workspace must have data retention enabled."）——调试负载之前先检查保留配置。（发布文档的早期草稿描述的是 404 且模型从 `/v1/models` 中隐藏；最终措辞是 400。如果你确实在访问该 ID 时看到 404，先检查保留配置再查其他。）需要该模型的 ZDR 组织应联系其 Anthropic 客户团队（"明确授权"路径），或为某一个工作区启用 30 天保留；*已经*能访问该模型的 ZDR 组织是拥有此类授权，而不是证明该要求已取消。（发布文档的早期草稿曾描述一项到 2026-12-31 为止的限期企业豁免；该句已于 8 月 28 日删除——不要引用它。）
- **Priority Tier:** not supported on Claude Fable 5.1 or Claude Mythos 5.1 (Claude Fable 5 is). A Claude Fable 5 caller on Priority Tier loses it on migration.
  **Priority Tier：**Claude Fable 5.1 与 Claude Mythos 5.1 不支持（Claude Fable 5 支持）。使用 Priority Tier 的 Claude Fable 5 调用方在迁移后会失去它。
- **Rate limits:** Claude Fable 5.1 shares one "Fable 5.x" pool with Claude Fable 5 (combined traffic; the Mythos models share a separate pool on the same terms) - re-baseline headroom if you run both during the migration.
  **速率限制：**Claude Fable 5.1 与 Claude Fable 5 共用一个 "Fable 5.x" 池（流量合并计算；Mythos 系列模型按相同条款共享另一个独立池）——如果迁移期间两者都在运行，请重新评估余量基准。
- **Pricing:** $10 / $50 per MTok, 5-minute cache writes $12.50, 1-hour cache writes $20, batch $5 / $25 - all as Claude Fable 5 - except **cache reads at $0.25 per MTok** (0.025x base input; Claude Mythos 5.1 shares this rate, Claude Opus 5.5 reads at 0.05x, and every other model at 0.1x): a quarter of the Claude Fable 5 rate and half of Claude Opus 5's. Long agentic sessions that re-read a cached prefix get most of the saving; caching break-even math in `shared/prompt-caching.md` shifts accordingly - and because a miss is now much more expensive relative to a hit, keeping the cache warm matters more: per-message effort and turn-scoped system messages exist partly for that, and for idle gaps of 5-60 minutes a `max_tokens: 0` keep-alive re-send on the default 5-minute TTL is usually cheaper than the 1-hour TTL (send it with `stream` off; not with structured outputs or Batches - see `shared/prompt-caching.md` § Choosing the TTL). Expect cost per task at or under the Claude Fable 5 figures in `shared/cost-optimization.md`.
  **定价：**每百万 token（MTok）$10 / $50，5 分钟缓存写入 $12.50，1 小时缓存写入 $20，批处理 $5 / $25——全部与 Claude Fable 5 相同——除了**缓存读取为每 MTok $0.25**（基础输入价的 0.025 倍；Claude Mythos 5.1 同享此费率，Claude Opus 5.5 按 0.05 倍读取，其他所有模型为 0.1 倍）：是 Claude Fable 5 费率的四分之一、Claude Opus 5 的一半。会反复重读缓存前缀的长代理会话获得大部分节省；`shared/prompt-caching.md` 中的缓存盈亏平衡计算随之变化——由于未命中相对命中现在贵得多，保持缓存温热更加重要：每消息努力度与轮次作用域系统消息的存在部分就是为了这个，而对 5-60 分钟的空闲间隔，在默认 5 分钟 TTL 上重发一次 `max_tokens: 0` 的保活请求通常比 1 小时 TTL 更便宜（发送时关闭 `stream`；不要与结构化输出或 Batches 一起使用——见 `shared/prompt-caching.md` § Choosing the TTL）。预期每个任务的成本达到或低于 `shared/cost-optimization.md` 中 Claude Fable 5 的数字。
- **Tool surface:** the same tool versions as Claude Fable 5 - code execution `code_execution_20250825` / `_20260120` / `_20260521` (programmatic tool calling needs `_20260120` or later), tool search (`tool_search_tool_regex_20251119`, `_bm25_20251119`), computer use `computer_20251124`, browser use, structured outputs, web fetch with dynamic filtering (`web_fetch_20260318`), and the advisor tool (as executor or advisor; Claude Fable 5.1 / Claude Mythos 5.1 advisors return the encrypted `advisor_redacted_result`). Task budgets: beta (`task-budgets-2026-03-13`, 20k minimum) - confirm at launch.
  **工具表面：**与 Claude Fable 5 相同的工具版本——代码执行 `code_execution_20250825` / `_20260120` / `_20260521`（程序化工具调用需要 `_20260120` 或更高版本）、工具搜索（`tool_search_tool_regex_20251119`、`_bm25_20251119`）、计算机使用 `computer_20251124`、浏览器使用、结构化输出、带动态过滤的 web 抓取（`web_fetch_20260318`），以及 advisor 工具（可作执行器或 advisor；Claude Fable 5.1 / Claude Mythos 5.1 的 advisor 返回加密的 `advisor_redacted_result`）。任务预算：beta（`task-budgets-2026-03-13`，最低 20k）——发布时需确认。
- **Content provenance (new, no request change):** text from Claude Fable 5.1 and Claude Mythos 5.1 carries Anthropic's statistical text watermark on every platform (no extra tokens or hidden characters, nothing about your org). Supported image, audio, and video files Claude produces in the code-execution sandbox carry signed C2PA Content Credentials when downloaded through the Files API on the Claude API - the manifest adds a few kilobytes, so the downloaded file's size and checksum differ from the file inside the container; text, PDF, and office files aren't signed. Platform scope beyond the Claude API is open at launch.
  **内容来源（新增，无需更改请求）：**来自 Claude Fable 5.1 与 Claude Mythos 5.1 的文本在每个平台上都带有 Anthropic 的统计性文本水印（不增加额外 token 或隐藏字符，不包含任何关于你组织的信息）。Claude 在代码执行沙箱中生成、并通过 Claude API 的 Files API 下载的受支持图片、音频与视频文件带有签名的 C2PA Content Credentials——清单会增加几千字节，因此下载文件的大小与校验和不同于容器内的文件；文本、PDF 与办公文件不带签名。Claude API 之外的平台范围在发布时待定。
- **1M context on Bedrock / Google Cloud and the batch 300k-output beta:** open at launch - confirm before promising either on a partner platform.
  **Bedrock / Google Cloud 上的 1M 上下文与批处理 300k 输出 beta：**发布时待定——在合作方平台上承诺任一项之前请先确认。

### New API features / 新增 API 特性

Three additions, each behind a beta header. All optional - a migrated request works without them - but the first two are how a harness stays cache-friendly and keeps its thinking preserved, so read them before touching an agent loop.

三项新增，各自需要一个 beta 请求头。全部可选——迁移后的请求不带它们也能工作——但前两项是 harness 保持缓存友好并保住思维不被丢弃的手段，因此在改动代理循环之前请先阅读它们。

**1. Per-message effort - beta `mid-conversation-output-config-2026-07-01`.** On Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5.5, Claude Opus 5, and Claude Sonnet 5.5 (with thinking on) (Claude API and Google Cloud; Claude Platform on AWS / Bedrock / Foundry not confirmed, and Claude Opus 5 is excluded on Bedrock), a `role: "system"` message with empty content and `output_config: {effort: ...}` changes effort from that point on without invalidating the prompt cache - raise it for a hard step, lower it for routine ones:

**1. 每消息努力度（per-message effort）——beta `mid-conversation-output-config-2026-07-01`。**在 Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Opus 5 与 Claude Sonnet 5.5（开启思维时）（Claude API 与 Google Cloud；Claude Platform on AWS / Bedrock / Foundry 未确认，且 Bedrock 上不含 Claude Opus 5）上，一条内容为空并带 `output_config: {effort: ...}` 的 `role: "system"` 消息会从该点起改变努力度而不使提示词缓存失效——难步骤调高，常规步骤调低：

```http
POST /v1/messages
anthropic-beta: mid-conversation-output-config-2026-07-01

{"model": "claude-fable-5-1", "max_tokens": 4096,
 "output_config": {"effort": "high"},
 "messages": [
   {"role": "user", "content": "Plan the migration."},
   {"role": "assistant", "content": "Here's the plan: ..."},
   {"role": "system", "content": [], "output_config": {"effort": "low"}},
   {"role": "user", "content": "Now rename the config file."}
 ]}
```

Values are the level names (`low`, `medium`, `high`, `xhigh`, `max`). The new level takes effect from the next `user` turn and holds until a later `role: "system"` message changes it. An effort-only message carries no text, so the placement rules for mid-conversation system messages don't apply - it can sit anywhere in `messages`, including first or between an assistant turn and the next user turn. Lowering effort this way is reliable; raising works best for large jumps (e.g. `low` to `xhigh`). On Claude Fable 5.1 prefer this form over changing the top-level value between requests: a top-level change restarts the cache *and* steers the model less reliably (its earlier replies were written at the previous level and it tends to stay consistent with them) - though a top-level change does not invalidate thinking blocks. Unsupported models, Claude Fable 5 included, 400: `output_config.effort requires a model that supports per-turn effort; this model does not`. The older spellings `mid-conversation-effort-2026-08-01` and `per-turn-control-2026-07-01` still resolve to the same feature but are undocumented - don't write new code with them. The beta is open to any organization that sends the header. This supersedes the "per-turn effort is not in this launch" note in the Claude Opus 5 checklist.

取值为级别名称（`low`、`medium`、`high`、`xhigh`、`max`）。新级别从下一条 `user` 轮开始生效，并保持到之后的某条 `role: "system"` 消息改变它为止。纯努力度消息不含文本，因此会话中途系统消息的位置规则不适用——它可以放在 `messages` 的任何位置，包括第一条，或 assistant 轮与下一条用户轮之间。用这种方式调低努力度是可靠的；调高则在大跨度跳跃（例如 `low` 到 `xhigh`）时效果最好。在 Claude Fable 5.1 上，比起在请求之间修改顶层值，更推荐这种形式：修改顶层值会重启缓存，*而且*对模型的引导较不可靠（它此前的回复是按上一级别写的，倾向于与它们保持一致）——不过修改顶层值不会使思维块失效。不支持的模型（包括 Claude Fable 5）返回 400：`output_config.effort requires a model that supports per-turn effort; this model does not`。较早的写法 `mid-conversation-effort-2026-08-01` 与 `per-turn-control-2026-07-01` 仍解析到同一特性但未写入文档——不要在新代码中使用它们。该 beta 对任何发送请求头的组织开放。此项取代 Claude Opus 5 检查清单中"每轮努力度不在本次发布中"的说明。

**2. Turn-scoped mid-conversation system messages - beta `mid-conversation-system-clear-at-2026-08-21`.** A harness often needs to tell the model something that is only true for one turn ("check your inbox before running code", "the user can't see that tool output"). Injecting the reminder and deleting it next request is a history edit - it restarts the prompt cache and, on Claude Fable 5.1, invalidates every later thinking block. Instead give a `role: "system"` message `clear_at: "next_user_message"`: its text carries system-prompt authority for the current turn, then stops rendering once a later `user` message exists. **Keep sending it back verbatim** - it stays in `messages`, so nothing earlier changes, the cache keeps matching, later thinking blocks stay valid, and a cleared message costs no input tokens.

**2. 轮次作用域的会话中途系统消息——beta `mid-conversation-system-clear-at-2026-08-21`。**harness 常常需要告诉模型一些只对某一轮成立的事（"运行代码前先检查收件箱"、"用户看不到那条工具输出"）。注入提醒再在下一次请求删除它属于历史编辑——它会重启提示词缓存，并且在 Claude Fable 5.1 上使之后所有思维块失效。替代做法是给一条 `role: "system"` 消息设置 `clear_at: "next_user_message"`：其文本在当前轮拥有系统提示词级别的权威，一旦出现更晚的 `user` 消息就停止渲染。**请把它原样持续回传**——它保留在 `messages` 里，因此更早的内容没有变化，缓存持续匹配，后续思维块保持有效，且已清除的消息不占输入 token。

```http
POST /v1/messages
anthropic-beta: mid-conversation-system-clear-at-2026-08-21

{"model": "claude-fable-5-1", "max_tokens": 4096,
 "tools": [...],
 "messages": [
   {"role": "user", "content": "Run the analysis script."},
   {"role": "assistant", "content": [{"type": "tool_use", "id": "toolu_01", "name": "bash",
    "input": {"command": "python analyze.py"}}]},
   {"role": "user", "content": [{"type": "tool_result", "tool_use_id": "toolu_01",
    "content": "Analysis complete; report written."}]},
   {"role": "system", "clear_at": "next_user_message",
    "content": "Results have landed in your inbox; check it before running more code."}
 ]}
```

The main use is a per-turn reminder in a tool loop: append the message after each `tool_result` user message you want it in view for, and **leave every earlier copy where it is** - a `tool_result`-only user message counts as the next user message, so the earlier copies are already cleared (rendering nothing, costing nothing, still part of the prefix the thinking is tied to) and the model reads only the newest one. Rules: `clear_at` takes `"never"` (the default) or `"next_user_message"`; a turn-scoped message is `text`-only (no `tool_addition`/`tool_removal` blocks, no `output_config`), takes no `cache_control` (put the breakpoint on the preceding user turn), and follows the normal placement rules - one followed directly by another `user` message is a 400, so put all of a tool round's results in one user message and the reminders after it. Deleting, rewording, rebuilding from current state, or changing the `clear_at` of a copy already sent is an edit like any other. Same models and platforms as mid-conversation system messages (on Bedrock and Google Cloud pass the beta value the way that platform passes betas); the launch SDKs may not type the field yet - send it via `extra_body` / a cast. **Without the beta**, append the reminder as a `text` block after the `tool_result` blocks in the same user message and leave earlier copies in place - the model acts on the newest one.

主要用途是工具循环中的每轮提醒：把它追加到每个你希望它处于视野中的 `tool_result` 用户消息之后，并且**把更早的每一份副本都留在原地**——只含 `tool_result` 的用户消息就算作下一条用户消息，因此更早的副本已被清除（不渲染、不计费、仍是思维所绑定前缀的一部分），模型只读最新那份。规则：`clear_at` 接受 `"never"`（默认）或 `"next_user_message"`；轮次作用域消息只能是 `text`（不能有 `tool_addition`/`tool_removal` 块、不能有 `output_config`），不能带 `cache_control`（把断点放在其前的用户轮上），并遵循正常的位置规则——紧跟另一条 `user` 消息的系统消息会得到 400，因此把一个工具轮的全部结果放进一条用户消息，提醒放在其后。删除、改写、按当前状态重建或更改已发送副本的 `clear_at` 同样属于编辑。与"会话中途系统消息"相同的模型与平台（在 Bedrock 与 Google Cloud 上，按该平台传递 beta 的方式传递 beta 值）；发布版 SDK 可能尚未为该字段提供类型——通过 `extra_body` / 强制转换发送。**不带该 beta 时**，把提醒作为同一用户消息中 `tool_result` 块之后的 `text` 块追加，并把更早的副本留在原地——模型会按最新的那条行动。

**3. Progress updates between tool calls - `thinking.display: "updates"`, beta `thinking-display-updates-2026-08-18`.** Between tool calls, Claude Fable 5.1, Claude Mythos 5.1, and Claude Fable 5 write short progress updates - what it just found, what it will do next - each returned as its own `thinking` block with its own signature immediately before the tool call it introduces, separate from any reasoning block at the same point. Under the default `display: "omitted"` those blocks come back empty, like reasoning, which is why a long agentic turn can look silent for minutes. Request `display: "updates"` and the progress updates come back as text while reasoning stays hidden:

**3. 工具调用之间的进度更新——`thinking.display: "updates"`，beta `thinking-display-updates-2026-08-18`。**在工具调用之间，Claude Fable 5.1、Claude Mythos 5.1 与 Claude Fable 5 会写下简短的进度更新——刚发现了什么、接下来要做什么——每条都作为独立的 `thinking` 块连同自己的签名返回，紧跟在它所引出的工具调用之前，与同一点上的任何推理块相互独立。在默认的 `display: "omitted"` 下，这些块与推理一样返回为空，这就是为什么长的代理轮可能看起来沉默数分钟。请求 `display: "updates"`，进度更新就会以文本返回，而推理保持隐藏：

```http
POST /v1/messages
anthropic-beta: thinking-display-updates-2026-08-18

{"model": "claude-fable-5-1", "max_tokens": 4096,
 "thinking": {"type": "adaptive", "display": "updates"},
 "tools": [...],
 "messages": [{"role": "user", "content": "Review the PRs open against our billing service."}]}
```

How to consume them: under `"updates"` **any `thinking` block with non-empty text is a progress update** (normally a sentence or two) - render it as a status line; render nothing for an empty block (a progress block can come back empty under any `display` value). When streaming, a progress block streams its text as `thinking_delta` events before the `tool_use` block it introduces - treat a block as a progress update as soon as a `thinking_delta` carries non-empty text; a pause of several seconds before the block opens is normal. A response can contain zero of them and the model can skip any gap, so build for zero-or-more. When a response stops on `max_tokens`, `model_context_window_exceeded`, or `stop_sequence` soon after a tool call or result, its last block can be a progress block standing in for unfinished work whose text is exactly `This part of the response was interrupted before it finished.` - to continue, pass the assistant turn back unchanged and append a new `user` message (with a `tool_result` for each `tool_use` in that turn). The update is billed at its full length in `usage.output_tokens`, not the summary's. Echo progress blocks back unchanged like any thinking block. `"summarized"` returns their text too, mixed with the reasoning summaries. Available on every platform - on Bedrock, Google Cloud, and Foundry pass the beta value the way that platform passes beta headers; without it `"updates"` is rejected as an unknown `display` value.

如何消费它们：在 `"updates"` 下，**任何文本非空的 `thinking` 块都是进度更新**（通常一两句话）——把它渲染为状态行；空块则什么都不渲染（在任何 `display` 取值下进度块都可能返回为空）。流式传输时，进度块在它引出的 `tool_use` 块之前以 `thinking_delta` 事件流式输出文本——只要某个 `thinking_delta` 携带非空文本就把它当作进度更新；块开始前的数秒停顿是正常的。一个响应可以一条都没有，模型也可能跳过任何间隙，因此按"零或多个"来设计。当响应在工具调用或结果之后不久因 `max_tokens`、`model_context_window_exceeded` 或 `stop_sequence` 停止时，其最后一个块可能是一个代表未完成工作的进度块，文本恰好是 `This part of the response was interrupted before it finished.`——要继续，原样回传该 assistant 轮并追加一条新的 `user` 消息（为该轮中每个 `tool_use` 提供对应的 `tool_result`）。进度更新按其完整长度计入 `usage.output_tokens` 计费，而非摘要的长度。进度块与其他思维块一样原样回传。`"summarized"` 也会返回它们的文本，与推理摘要混在一起。每个平台都可用——在 Bedrock、Google Cloud 与 Foundry 上，按该平台传递 beta 请求头的方式传递 beta 值；不带它时 `"updates"` 会作为未知的 `display` 取值被拒绝。

### Coming from Claude Opus 5 / 从 Claude Opus 5 迁移

Beyond the three breaking changes: `thinking: {type: "disabled"}` 400s at **any** effort (on Claude Opus 5 it was accepted at `high` or lower) - remove it, control spend with lower effort, and revisit `max_tokens`. Text the model wrote *between tool calls* on Claude Opus 5 came back as `text` blocks; on Claude Fable 5.1 it comes back as progress-update `thinking` blocks, empty under the default `"omitted"` - set `display: "updates"` (or `"summarized"`) if your UI rendered that narration. The classifier set is broader (`bio`, `reasoning_extraction` in addition to `cyber`). ZDR is lost (Claude Opus 5 is available under ZDR). Pricing goes from $5 / $25 to $10 / $50 per MTok, with cache reads at half Claude Opus 5's rate; the 512-token cache minimum is unchanged. Per-message effort already works on Claude Opus 5, so an Claude Opus 5 harness that uses it needs no change there.

除三项破坏性变更之外：`thinking: {type: "disabled"}` 在**任何**努力度下都返回 400（在 Claude Opus 5 上 `high` 及以下可接受）——移除它，用更低的努力度控制开销，并重新审视 `max_tokens`。模型在 Claude Opus 5 上*写在工具调用之间*的文本以 `text` 块返回；在 Claude Fable 5.1 上则以进度更新 `thinking` 块返回，在默认 `"omitted"` 下为空——如果你的 UI 渲染过那段叙述，请设置 `display: "updates"`（或 `"summarized"`）。分类器集合更广（在 `cyber` 之外加上 `bio`、`reasoning_extraction`）。ZDR 丢失（Claude Opus 5 可在 ZDR 下使用）。定价从每 MTok $5 / $25 涨到 $10 / $50，缓存读取为 Claude Opus 5 费率的一半；512 token 缓存下限不变。每消息努力度在 Claude Opus 5 上已可用，因此使用它的 Claude Opus 5 harness 无需为此改动。

From Opus 4.8 or earlier: apply § Migrating to Claude Fable 5.1 above first (Opus 4.7 or earlier: the Claude Opus 5 section before that), then this one - and budget time for the history-editing check: integrations written for Opus 4.8 and earlier often truncate old turns, strip or rebuild earlier messages, or refresh the `system` prompt each request, and Opus 4.8 never objected. Review prompts near the 512-token caching minimum.

从 Opus 4.8 或更早版本迁移：先应用上文 § Migrating to Claude Fable 5.1（Opus 4.7 或更早：先看其前的 Claude Opus 5 一节），再应用本节——并为历史编辑检查预留时间：为 Opus 4.8 及更早版本编写的集成常常截断旧轮次、剥离或重建较早的消息，或每次请求刷新 `system` 提示词，而 Opus 4.8 从不介意。请检查接近 512 token 缓存下限的提示词。

### Claude Mythos 5.1 / Claude Mythos 5.1

`claude-mythos-5-1` is the same model as Claude Fable 5.1 - same capabilities, limits, and per-token pricing (including the $0.25/MTok cache-read rate), and the same API behavior except that it does not run the history-editing check (breaking change 3) - offered only to approved Project Glasswing customers, and the only model besides Claude Fable 5.1 that reads Claude Fable 5.1's thinking blocks (it also reads Claude Mythos 5's; not the reverse). Confirm the organization's access with the account team before switching IDs. Two differences from a Claude Mythos 5 migrator's point of view: **Claude Mythos 5.1 runs safeguards** that depend on the access program the organization is approved under (Claude Mythos 5 ran none) - handle `stop_reason: "refusal"`, read `stop_details.category`, and set up fallback as on Claude Fable 5.1 (same targets, `claude-opus-4-8` and `claude-opus-5`; it mints a `fallback_credit_token` on refusals as Claude Fable 5.1 does) - and it is **not offered on Claude Platform on AWS** (Claude API, Amazon Bedrock as `anthropic.claude-mythos-5-1` in us-east-1 only and not publicly listed, Google Cloud, Microsoft Foundry). It shares the Mythos rate-limit pool with Claude Mythos 5. Whether Claude Mythos 5 access carries over automatically is open at launch.

`claude-mythos-5-1` 与 Claude Fable 5.1 是同一个模型——相同的能力、限额与按 token 定价（包括 $0.25/MTok 的缓存读取费率），API 行为也相同，只是它不运行历史编辑检查（破坏性变更 3）——仅向获得批准的 Project Glasswing 客户提供，并且是除 Claude Fable 5.1 之外唯一能读取 Claude Fable 5.1 思维块的模型（它也能读取 Claude Mythos 5 的；反之不行）。切换模型 ID 之前请与客户团队确认组织的访问权限。从 Claude Mythos 5 迁移者的视角看有两处差异：**Claude Mythos 5.1 会运行安全防护**，具体取决于组织获批加入的访问计划（Claude Mythos 5 不运行任何防护）——要处理 `stop_reason: "refusal"`、读取 `stop_details.category`，并按 Claude Fable 5.1 的方式配置回退（目标相同，`claude-opus-4-8` 与 `claude-opus-5`；它也会像 Claude Fable 5.1 那样在拒答时铸造 `fallback_credit_token`）——而且它**不在 Claude Platform on AWS 上提供**（Claude API、Amazon Bedrock 上仅 us-east-1 区域有 `anthropic.claude-mythos-5-1` 且未公开列出、Google Cloud、Microsoft Foundry）。它与 Claude Mythos 5 共享 Mythos 速率限制池。Claude Mythos 5 的访问权限是否自动沿用，在发布时待定。

### Capability improvements versus Claude Fable 5 / 相对 Claude Fable 5 的能力提升

The gap is widest at higher effort levels. Six areas: **agentic coding over long sessions** (multi-file features, large refactors and migrations, debugging, code review across sessions that run for hours); **knowledge work with documents, spreadsheets, and slides** (from a first question to a finished document, live-formula spreadsheet, or deck built from a blank page); **research and search** (multistep web research that follows up on what it finds); **vision** (dense charts, filings, and tables nested in PDFs - strongest when it has tools to crop and zoom); **long-context retrieval** deep in the 1M window; and **computer use** (operating a browser and desktop applications more reliably, recovering from failed steps). Multilingual performance is on par with Claude Fable 5. Held pending confirmation at launch: that it expands a request's scope partway through less often, and that an instruction given once at the start of a long session persists better - if the latter holds, remove instruction repetition inserted every few turns for Claude Fable 5 and re-test (the per-turn batching nudge below is a separate case: it targets one behavior on the next turn, so keep it where measurements show it helps).

差距在更高努力度级别上最大。六个方面：**长会话上的代理式编码**（多文件特性、大型重构与迁移、调试、跨数小时会话的代码评审）；**围绕文档、电子表格与幻灯片的知识工作**（从第一个问题到成稿文档、带活性公式的电子表格或从空白页建起的演示文稿）；**研究与搜索**（会对发现进行追问的多步网络研究）；**视觉**（密集图表、申报文件与嵌在 PDF 中的表格——在有裁剪与缩放工具时最强）；**1M 窗口深处的长上下文检索**；以及**计算机使用**（更可靠地操作浏览器与桌面应用、从失败步骤中恢复）。多语言表现与 Claude Fable 5 相当。待发布确认的事项：它更少在中途扩大请求范围；以及在长会话开始时给出一次的指令能更持久地保持——如果后者成立，可以移除为 Claude Fable 5 每隔几轮插入的指令重复并重新测试（下文的每轮批量调用提示是另一回事：它针对下一轮的单一行为，测量显示有帮助就保留）。

### Behavioral shifts (prompt-tunable) / 行为变化（可通过提示词调整）

None of these are API-breaking. The behavioral guidance in § Migrating to Claude Fable 5.1 above (longer turns, grounding progress claims, stating boundaries, delegation, memory surfaces, the readability addendum) still applies; these are the Claude Fable 5.1-specific deltas. Three of them show up without any code change: it batches implied tool calls less, narrates less between tool calls, and answers from memory more at `low` effort.

这些都不是 API 层面的破坏。上文 § Migrating to Claude Fable 5.1 中的行为指导（更长的轮次、让进度陈述有据可依、声明边界、委托、记忆载体、可读性附则）仍然适用；这里是 Claude Fable 5.1 特有的差异。其中三项无需任何代码改动即会显现：它更少把隐含的工具调用打包执行、在工具调用之间更少叙述、在 `low` 努力度下更多凭记忆作答。

**Effort.** Start with `high` (the default) and re-run your effort sweep even if you ran one on Claude Fable 5 - level names don't correspond to the same amount of thinking across models. The gains over Claude Fable 5 show up across levels and are largest at the higher settings; at `medium`, results roughly match Claude Fable 5 at lower cost, so step down to `medium` or `low` where your evals show quality holds. At `high` and above set a large `max_tokens` - it is a hard limit on total output (thinking plus response). At `low`, Claude Fable 5.1 is often competitive with Opus and Sonnet on cost per task while performing better - evaluate low Fable effort against below-frontier usage before reaching for a cheaper model. Per-message effort (addition 1) lets one conversation mix levels without a cache reset.

**努力度（Effort）。**从 `high`（默认值）开始，即使你在 Claude Fable 5 上做过努力度扫描也要重做一遍——级别名称在不同模型上对应的思考量并不相同。相对 Claude Fable 5 的收益在各个级别上都有体现，且在更高设置上最大；在 `medium` 下，结果大致与 Claude Fable 5 相当而成本更低，因此在你的评测显示质量仍达标之处可以降到 `medium` 或 `low`。在 `high` 及以上要设置较大的 `max_tokens`——它是对总输出（思考加回复）的硬上限。在 `low` 下，Claude Fable 5.1 在每任务成本上往往与 Opus 和 Sonnet 相当而表现更好——在伸手换更便宜的模型之前，先把 Fable 低努力度与低于前沿的用法做对比评估。每消息努力度（新增项 1）让同一个对话能混合多个级别而不重置缓存。

**Long deliverables at `xhigh` and `max`.** At `xhigh`, and especially `max`, the model thinks more before it starts writing. When one request asks for a long deliverable - a full rewrite of a long document, a large table, a complete code file - it may draft much of it in its thinking and then write it out again as the reply: a longer wait and roughly double the output tokens. Simplest fix: run those requests at `high` (the recommended start anyway) and move up only where you've measured a quality gain. If you do run them at `xhigh`/`max`, set `max_tokens` to leave room for the thinking *and* the reply, and append this to the end of the user message - it makes the thinking much shorter on prose and code requests (replace the bracket with the request's actual `max_tokens`, e.g. 64,000). Like every appended per-request note on this model (addition 2 above), leave each earlier copy in place byte-for-byte on later requests, each keeping the value it was sent with - removing or rebuilding one is a history edit that invalidates the thinking blocks after it:

**`xhigh` 与 `max` 下的长交付物。**在 `xhigh`，尤其是 `max` 下，模型在动笔之前思考更多。当单个请求要求一份长交付物——对长文档的完整重写、一张大表、一个完整的代码文件——它可能在思考中起草大部分内容，然后再在回复中重写一遍：等待更久，输出 token 约为两倍。最简单的解法：把这些请求放在 `high` 上运行（本来也是推荐的起点），只在测量到质量收益时才上调。如果确实要在 `xhigh`/`max` 上运行，把 `max_tokens` 设为给思考*和*回复都留出空间，并在用户消息末尾追加下面这段——它能让散文与代码请求的思考大幅变短（把方括号替换为该请求实际的 `max_tokens`，如 64,000）。与本模型上每一条追加的每次请求附注一样（上文新增项 2），后续请求要把每份更早的副本逐字节留在原地，各自保留发送时的取值——移除或重建其中一份属于历史编辑，会使它之后的思维块失效：

> Everything Claude produces in one reply, including any reasoning or drafting it does before the reply, counts toward a single limit of about [max_tokens] tokens. If that limit is reached before the reply is finished, the person receives a cut-off response and has to start over. Composing an entire output or deliverable in full as reasoning and then again as a reply would double the length of the turn without improving the result, so Claude doesn't do that.  
> Instead, when the person has asked for a long or effort-intensive deliverable such as a multi-section document, a large table or dataset, or a complete code file, Claude spends extra effort on understanding the request, checking the inputs Claude's answer depends on, settling the structure and other difficult decisions, and otherwise using the reasoning space to reason and the output space to write an output. If Claude plans well then it should not need to draft its output multiple times (and Claude is pretty good at planning, so this should not be an issue).
> Claude 在一次回复中产出的一切，包括它在回复之前所做的任何推理或起草，都计入约 [max_tokens] token 的同一个上限。如果在回复完成之前达到该上限，用户会收到一段被截断的响应，不得不从头再来。把整份输出或交付物先作为推理完整写一遍、再作为回复完整写一遍，会在不改善结果的情况下使轮次长度翻倍，所以 Claude 不这样做。  
> 反之，当用户要求长篇或高努力度的交付物——例如多章节文档、大表格或数据集、完整的代码文件——Claude 会把额外的努力花在理解请求、核对答案所依赖的输入、敲定结构与其他困难决策上，总体上是把推理空间用于推理、把输出空间用于写出输出。如果 Claude 规划得当，就不需要把输出起草多遍（而 Claude 相当擅长规划，所以这不应成为问题）。

**Batch independent tool calls in agent loops.** When a request explicitly names several things to fetch, Claude Fable 5.1 issues those calls in parallel; standard function calling is unaffected. In long agent loops where the next independent reads are only *implied* (custom coding agents, bash-and-editor harnesses, computer use) it may issue one call per turn where Claude Fable 5 batched several - same answers, more round trips and wall-clock. Measure first: track the share of assistant turns with more than one tool call, and add the nudge only if that share is low (over-batching shows up as calls issued before results they depend on). Placement matters more than wording - one sentence near the end of the current request moves the number far more than the same text in the system prompt or a tool description. Each time you send tool results back, append the sentence after that user message as a turn-scoped system message (`clear_at: "next_user_message"`, addition 2) - or, without that beta, as a `text` block after the `tool_result` blocks in the same user message - **appending a fresh copy each turn and leaving the earlier copies in place byte-for-byte**; rewriting earlier turns to remove them restarts the cache and, on this model, invalidates the thinking blocks after them. Keep the word "privately" - without it the model sometimes answers the reminder ("nothing further is needed") instead of the user in its final reply:

**在代理循环中把相互独立的工具调用打包。**当请求明确点名要取几样东西时，Claude Fable 5.1 会并行发出这些调用；标准函数调用不受影响。而在下一批独立读取只是*隐含*的长代理循环中（自研编码代理、bash 加编辑器类 harness、计算机使用），它可能每轮只发一次调用，而 Claude Fable 5 会打包几次——答案相同，但往返次数与墙上时钟时间更多。先测量：统计含多个工具调用的 assistant 轮占比，只有当该占比偏低时才加提示（打包过度表现为在其依赖的结果就绪之前就发出调用）。位置比措辞更重要——靠近当前请求结尾的一句话对数字的影响远大于同一段文字放在系统提示词或工具描述里。每次回传工具结果时，在该用户消息之后把这句话作为轮次作用域系统消息追加（`clear_at: "next_user_message"`，新增项 2）——或不带该 beta，作为同一用户消息中 `tool_result` 块之后的 `text` 块——**每轮追加一份新副本，并把更早的副本逐字节留在原地**；改写较早的轮次来移除它们会重启缓存，并且在本模型上使它们之后的思维块失效。保留 "privately" 一词——没有它，模型有时会在最终回复中回答提醒本身（"无需更多操作"）而不是回答用户：

> First privately list what you need next; then request every item that doesn't depend on another's result in this one response.
> 先私下列出你接下来需要什么；然后在这一次响应中请求每一项不依赖其他结果的内容。

**User-facing progress updates.** Claude Fable 5.1 writes fewer user-facing updates during long tool-calling turns than Claude Fable 5 - more so at higher effort and in longer tool chains. Users see the agent go quiet for minutes, or a final message that describes only the last step; its agentic coding summaries are shorter too. In order: (1) **request `display: "updates"`** (addition 3) - if you aren't, the model's between-tool notes aren't reaching you; (2) **remove prompt text written for update-eager older models** ("hold all findings for the final response", "don't narrate") *before* adding anything; (3) if you still want more - pair programming, human-in-the-loop - add a short, specific system-prompt line saying when you want user-facing text:

**面向用户的进度更新。**在长的工具调用轮中，Claude Fable 5.1 写给用户的更新比 Claude Fable 5 少——努力度越高、工具链越长越明显。用户会看到代理沉默数分钟，或者最后一条消息只描述了最后一步；它的代理式编码摘要也更短。按顺序：(1) **请求 `display: "updates"`**（新增项 3）——如果你还没设，模型在工具之间写的说明根本没送到你手上；(2) 在添加任何内容*之前*，**删除为热衷更新的旧模型写的提示词文本**（"hold all findings for the final response"、"don't narrate"）；(3) 如果你仍想要更多——结对编程、人在回路——在系统提示词里加一句简短、具体的说明，讲明你何时想要面向用户的文本：

> Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own - what you found, what you did, and what's next - so a reader who only sees the last message has the full picture.
> 开始之前，用一行说明你即将做什么；工作过程中简短的更新有助于用户跟上进展。结尾给出一段能独立成立的简短回顾——你发现了什么、做了什么、接下来是什么——让只看到最后一条消息的读者也能掌握全貌。

Relatedly, **if the harness collapses or hides tool output, tell the model** - otherwise Claude Fable 5.1 may run commands to "show" the user output they cannot see. Deliver it as a turn-scoped system message (`clear_at: "next_user_message"`, addition 2), or without the beta alongside the tool results in the same user message, left in place on later requests:

与此相关，**如果 harness 会折叠或隐藏工具输出，请告诉模型**——否则 Claude Fable 5.1 可能为了"展示"用户看不到的输出而运行命令。用轮次作用域系统消息交付（`clear_at: "next_user_message"`，新增项 2），或不带该 beta、与工具结果一起放在同一用户消息中，并在后续请求中原样保留：

> Only you see that command's output - the user's terminal shows at most a few lines of it. If the user needs to read any of it, put it in your reply.
> 只有你能看到那条命令的输出——用户的终端最多显示其中几行。如果用户需要阅读其中任何内容，请把它写进你的回复。

**Writing density.** Claude Fable 5.1's writing is generally preferred, but prose can be denser than Claude Fable 5's - longer sentences, fewer paragraph breaks. Defining "mannered prose" as an anti-pattern has helped; style instructions placed in the first user turn of a session hold better than the same text in the system prompt:

**写作密度。**Claude Fable 5.1 的写作总体上更受青睐，但行文可能比 Claude Fable 5 更密——句子更长，分段更少。把"套路化行文（mannered prose）"定义为反模式已有帮助；放在会话第一条用户轮中的风格指令，比同样的文字放在系统提示词里更持久：

> Mannered prose substitutes metaphor and flourish for direct statement. Instead of "a parameter worth varying," the mannered writer produces "a dial worth turning." Instead of "this point still matters," they write "this point earns its keep." The phrases exist to display the writer, not to convey the idea, and readers can tell. That is why mannered prose irritates: it makes the reader work harder so the writer can perform. It is also imprecise. Metaphors drag in connotations the writer did not choose and cannot control. The fix is to say what you mean. When a literal phrase is available, use it.
> 套路化行文用比喻和辞藻代替直陈。不写"a parameter worth varying"，而写"a dial worth turning"；不写"this point still matters"，而写"this point earns its keep"。这些措辞的存在是为了展示作者本人而不是传达观点，读者分辨得出来。这正是套路化行文令人烦躁的原因：它让读者更费劲，好让作者表演。它还不精确。比喻会拖进作者没有选择也无法控制的联想。解法是直接说你是什么意思。有字面说法可用时，就用字面说法。
【评论】这段反"套路化行文"的提示词属于纯风格层面的行为校准，不依赖任何 API 特性，可作为提示词工程中风格控制的参考示例。

The short form - "Please remove all mannered prose." - also tends to work.

短版本——"Please remove all mannered prose."——往往也有效。

**Formatting.** Where earlier models over-used bullets and bold in chat, Claude Fable 5.1 does the opposite: less bold, fewer headers, lists, and quotation marks. **If the prompt contains anti-formatting language, remove it** or replace it with a rule that says when formatting is appropriate:

**格式。**此前的模型在聊天中过度使用项目符号和粗体，Claude Fable 5.1 恰好相反：更少粗体、更少标题、列表和引号。**如果提示词中含有反格式化措辞，请删掉**，或替换成一条说明何时适合使用格式的规则：

> Use lists and bullet points when asked to, or when the content is multifaceted enough that they help with clarity. If the person explicitly requests minimal formatting, always format your responses without bullet points, headers, lists, or bold emphasis, as requested. In conversational, personal, or emotional exchanges, keep to plain prose.
> 在被要求时，或内容足够多面、列表有助于清晰呈现时，使用列表和项目符号。如果用户明确要求极简格式，就按其要求，始终以不带项目符号、标题、列表或粗体强调的方式组织回复。在对话式、私人化或情绪化的交流中，保持朴素行文。

When summarizing documents, Claude Fable 5.1 is more likely than Claude Fable 5 to reproduce passages of source text without marking them as quotations. The fix is one complete example of a correct response in the system prompt - the user's request, the response, and a sentence saying why the response is correct. Replace the two `[web_search: ...]` lines with your own tool name so the model reads them as templated tool output, not as literal desired output:

在总结文档时，Claude Fable 5.1 比 Claude Fable 5 更容易不加引注标记地照抄源文段落。解法是在系统提示词中给出一个完整的正确响应示例——用户的请求、响应，以及一句说明该响应为何正确的话。把两行 `[web_search: ...]` 替换成你自己的工具名，让模型把它们读作模板化的工具输出，而不是字面上的期望输出：

```xml
<example>
<user>look up how the Riverton Ledger and the Coast Dispatch each covered the Harbor Bridge closure and compare their reporting</user>
<response>
[web_search: Harbor Bridge closure Riverton Ledger]
[web_search: Harbor Bridge closure Coast Dispatch]
Both outlets agree on the basics: the bridge closed on March 3 after inspectors found cracked welds, and the state expects repairs to take about eight months. Where they differ is emphasis. The Ledger treats it as a local-economy story. The Dispatch frames it as a funding failure; its editorial calls the closure "entirely foreseeable." Read together, the Ledger explains who is affected now and the Dispatch explains how it came to this - neither account alone gives the whole picture.
</response>
<rationale>CORRECT: The response is organized around where the two outlets agree and differ, not as a walk through either article. Each outlet's reporting is conveyed in one or two sentences of the assistant's own indirect speech. One short marked phrase from one source; every other claim is reworded. The response is still specific and complete.</rationale>
</example>
```

**Maximizing long-horizon execution.** Claude Fable 5.1 is capable of very long autonomous runs, but on complex asynchronous workloads it needs a nudge not to stop at *describing* the next step ("Next, I'll ...") or asking permission for a step the request already covered ("Shall I apply this?"). Users experience it as having to reply "continue" - fine for pair programming, but it caps the model's long-horizon capability. Two system-prompt additions together mitigated this; apply both unless context is tight, in which case the first keeps most of the effect. The opening sentence of the first ("The user is not watching") is load-bearing - keep it as written; if the product needs stops for specific confirmations, add a sentence listing them. This prompt can make the model less likely to clarify ambiguous requests. With either block the model writes slightly more code - mostly extra tests in files it's already editing - so pair them with the "ground progress claims" audit instruction in § Migrating to Claude Fable 5.1 above and the test-coverage line below. If your existing prompt asks the model to test or check its work before reporting, **keep it** when migrating - the Claude Opus 5 guidance to delete verification instructions doesn't apply here (tentative: rests on a small number of reports).

**最大化长程执行。**Claude Fable 5.1 能够进行很长的自主运行，但在复杂的异步工作负载上，它需要一点推动才不会停在*描述*下一步（"Next, I'll ..."）或为请求已经覆盖的步骤请求许可（"Shall I apply this?"）。用户的体验是不得不回复"continue"——结对编程时无妨，但这限制了模型的长程能力。两条系统提示词增补一起使用可缓解此问题；除非上下文紧张，否则两条都用，紧张时第一条能保留大部分效果。第一条的开头句（"The user is not watching"）是承重句——照原样保留；如果产品需要为特定确认而停下，加一句列出这些确认点。这段提示词可能降低模型澄清含糊请求的倾向。无论用哪一条，模型的代码产出都会略增——多为它正在编辑的文件里补的测试——因此请把它们与上文 § Migrating to Claude Fable 5.1 中"让进度陈述有据可依"的审计指令以及下文的测试覆盖条款搭配使用。如果你现有的提示词要求模型在报告前测试或检查自己的工作，迁移时**保留它**——Claude Opus 5 那条"删除验证指令"的指导在这里不适用（初步结论：依据的样本较少）。

> You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking 'Want me to...?' or 'Shall I...?' will block the work. For reversible actions that follow from the original request, proceed without asking. Stop only for destructive actions or genuine scope changes the user must decide. Offering follow-ups after the task is done is fine; asking permission before doing the work is not.  
>  
> Exception: when the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one.  
>  
> Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ('I'll...', 'let me know when...'), do that work now with tool calls. That includes retrying after errors and gathering missing information yourself. Do not stop because the context or session is long. End your turn only when the task is complete or you are blocked on input only the user can provide.  
>  
> Before running a command that changes system state (such as restarts, deletes, or config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
> 你在自主运行。用户并不实时旁观，也无法在任务中途回答问题，因此问"Want me to...?"或"Shall I...?"会阻塞工作。对于由原始请求引出的可逆操作，直接进行、无需询问。只在破坏性操作或用户必须亲自决定的真正范围变更时停下。任务完成后提供后续建议没有问题；做事之前请求许可则不然。  
>  
> 例外：当用户是在描述问题、提问或自言自语，而非请求改动时，交付物是你的评估。报告发现后即停止；在他们开口要求之前不要动手修复。  
>  
> 在结束你的轮次之前，检查最后一段。如果它是一个计划、一段分析、一个问题、一列后续步骤，或关于你尚未完成工作的承诺（"I'll..."、"let me know when..."），现在就用工具调用把那些工作做完。其中包括出错后重试以及自行补齐缺失信息。不要因为上下文或会话很长就停下。只有当任务完成，或你被只有用户才能提供的输入阻塞时，才结束轮次。  
>  
> 在运行会改变系统状态的命令（重启、删除、配置修改等）之前，核实证据确实支持那个具体动作。某个模式匹配上已知故障的信号，成因可能完全不同。

The second tells it to hold the scope the user set:

第二条让它守住用户设定的范围：

> \# Delivering work  
> The user's request - or the plan they approved - sets the scope, and the scope is the deliverable: don't quietly narrow, widen, or swap it. Read ambiguity the way a careful colleague would: make routine judgment calls yourself, and check in only when different readings would lead to materially different work. If you see a real problem with the task as specified, say so in a sentence or two and keep building under stated assumptions; if the user hears the concern and reaffirms, that is their decision, so deliver the full request.  
>  
> If a question comes up partway, first do everything that doesn't depend on the answer; then state the assumption you made, or - when going ahead on a wrong guess would be unsafe or would make the work useless - put the question at the end of a turn that also delivers that progress. If one part turns out to be blocked, complete every other part in full and say exactly what you left out and why - the whole task is the deliverable, and scaling it down is the user's call, not yours. A step you have decided on is something to run, not to announce: describing the next step and ending the turn leaves it undone until the user replies.  
>  
> Keep changes to what the request needs. Something else you notice worth doing - cleanup or documentation the task didn't call for, a change to a file the task didn't require - is a suggestion to make at the end, not a change to make; actions clearly beyond what the ask implies, and risky or destructive ones, still need the user's go-ahead.
> \# 交付工作  
> 用户的请求——或他们批准的计划——设定了范围，而范围就是交付物：不要悄悄收窄、扩大或调换它。像一位谨慎的同事那样解读含糊之处：例行判断自己做，只有当不同解读会导致实质不同的工作时才去确认。如果你发现按规格写就的任务确有真实问题，用一两句话说出来，然后在声明的假设下继续构建；如果用户听闻顾虑后仍坚持，那是他们的决定，因此要交付完整的请求。  
>  
> 如果中途冒出问题，先做所有不依赖答案的事；然后说明你所做的假设，或者——当按错误猜测推进会不安全或会让工作白费时——把问题放在一个同时交付了上述进展的轮次末尾。如果某一部分被阻塞，把其余每一部分完整做完，并准确说明你省略了什么、为什么——整个任务才是交付物，缩小范围是用户的决定，不是你的。你已经定下的步骤是要去执行的，不是要去宣布的：描述下一步然后结束轮次，会让它在用户回复之前一直无人执行。  
>  
> 改动以请求所需为限。你注意到的其他值得做的事——任务没有要求的清理或文档、任务没有涉及的文件的改动——是留到最后提出的建议，不是要做的改动；明显超出请求所蕴含范围的操作，以及有风险或破坏性的操作，仍然需要用户点头。

(The published snippets use em dashes and ellipsis characters where this file has hyphens and three periods; the bundled skill is ASCII-only, and the difference has no effect on the model.)

（官方发布的片段在本文档使用连字符和三个句点的地方使用破折号和省略号字符；随包发布的 skill 仅限 ASCII 字符，这一差异对模型没有影响。）

**Scope and test coverage.** Asked to implement an open-ended feature, Claude Fable 5.1 delivers what was asked and sometimes more - fixing nearby code, writing extra tests, committing scratch checks as permanent test files. It responds well to explicit instructions about what to leave out; with this prompt the guide's authors saw far fewer unrequested additions and much less committed test code with no measurable change in task success (an earlier, shorter form - "keep verification scripts outside the repository, e.g. under `/tmp`, and delete any you did add" - still works if you only care about test sprawl):

**范围与测试覆盖。**被要求实现一个开放式特性时，Claude Fable 5.1 会交付所求，有时还会多给——顺手修周边代码、多写测试、把临时检查脚本作为永久测试文件提交。它对"不要做什么"的明确指令响应良好；用这段提示词，指南作者看到的未经请求的附加改动少得多、提交的测试代码也少得多，而任务成功率没有可测的变化（一个更早、更短的版本——"keep verification scripts outside the repository, e.g. under `/tmp`, and delete any you did add"——如果你只在乎测试蔓延，依然有效）：

> If, while working or testing, you find a pre-existing bug, a performance concern, or behavior the task doesn't mention, don't fix, optimize or extend it in this change unless the requested behavior cannot work without it; report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption in your summary, and don't build for the other readings as well. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or this repository already keeps tests for this kind of change, sized like the neighboring test files - roughly one focused test per stated behavior - and don't turn scratch checks into additional permanent test files. This is about extras only: implement every behavior the task asks for, completely.
> 如果在工作或测试过程中，你发现一个预先存在的 bug、一个性能隐患，或任务没有提及的行为，除非所请求的行为离开它就无法工作，否则不要在本次改动中修复、优化或扩展它；在总结中作为后续事项报告。当任务含糊时，实现其措辞与周边代码最直接支持的那种解读，在总结中说明这一假设，不要同时为其他解读做构建。用你喜欢的任何方式验证你的工作；临时脚本和快速检查无需保留。只在任务要求提交测试、或本仓库本就为这类改动保留测试的地方提交测试，体量与相邻的测试文件相称——大约每个既述行为一个聚焦的测试——并且不要把临时检查变成额外的永久测试文件。这只针对附加部分：任务要求的每一个行为都要完整实现。

**Search triggering at low effort.** At `low` effort, Claude Fable 5.1 calls a search or retrieval tool less often than Claude Fable 5 and answers from memory more - most visibly for named products, models, and tools it recognizes but has out-of-date knowledge of. Raising effort for those turns (per-message effort, feature 1) is often the simplest fix. Otherwise tell it in the system prompt that recognizing a name is not the same as knowing its current state, and that such names should be searched as the user wrote them:

**低努力度下的搜索触发。**在 `low` 努力度下，Claude Fable 5.1 调用搜索或检索工具的频率低于 Claude Fable 5，更多凭记忆作答——对于它认识但知识已过时的具名产品、模型和工具最为明显。为这些轮次调高努力度（每消息努力度，特性 1）往往是最简单的解法。否则，在系统提示词中告诉它：认出一个名字不等于了解其现状，这类名字应按用户书写的形式进行搜索：

> When a query centers on a name you do not confidently recognize, or recognize from a fast-moving area like AI models and developer tools where the landscape shifts within months, the name itself is the thing to verify: search before answering, and include the name as the user wrote it in at least one query alongside any reformulations. This holds even when you have some background on it - partial background is exactly what makes an out-of-date answer sound authoritative, so familiarity is not a reason to skip the search.
> 当查询围绕一个你没有把握认出的名字，或一个你认识但来自 AI 模型、开发者工具这类格局数月一变的快速演进领域的名字时，需要核实的正是名字本身：先搜索再作答，并且至少在一条查询中按用户书写的形式包含该名字，与其他改写并列。即使你对它有一些背景了解，这条也成立——一知半解恰恰是过时答案显得权威的原因，因此熟悉不是跳过搜索的理由。

**Vision: let it crop, zoom, and verify.** Claude Fable 5.1's pure vision is better out of the box, and it is best when it can iteratively analyze, crop, and visually verify its own work. For complex inputs - dense charts, filings, tables nested in PDFs, video - run it as an agent with a container that holds the raw images/videos and basic image-processing libraries (PIL, OpenCV) preinstalled. If a container is too much overhead, most of the uplift comes from a single crop tool that takes a bounding box and returns that region cropped and enlarged (the recipe is in the Claude Opus 5 section's vision guidance) - this scales test-time compute with image tokens instead of effort. At `low` effort the model may answer from an overall impression without calling it, so check the logs for the call and raise effort on image turns if it's missing.

**视觉：让它裁剪、缩放并验证。**Claude Fable 5.1 的纯视觉能力开箱即有提升，而当它能迭代地分析、裁剪并对自己的工作做视觉验证时表现最好。对于复杂输入——密集图表、申报文件、嵌在 PDF 中的表格、视频——把它当作代理来运行，配一个预装原始图片/视频和基础图像处理库（PIL、OpenCV）的容器。如果容器开销太大，大部分提升来自一个单一的裁剪工具：接收边界框、返回裁剪放大后的区域（配方见 Claude Opus 5 一节的视觉指导）——这用图像 token 而不是努力度来扩展测试时算力。在 `low` 努力度下，模型可能凭整体印象作答而不调用该工具，因此要检查日志确认调用是否存在，若缺失则在图像轮次上调高努力度。

**Safeguard false positives.** The classifiers produce fewer false positives than Claude Fable 5's did at launch, and finding vulnerabilities in source code is permitted; a blocked request still returns `stop_reason: "refusal"`, so keep the refusal handling and fallbacks in place. Three situations make false positives more likely: compile-check phrasing (ask "Are there any bugs in this program?" rather than "Does this program compile without errors?"); lesser-known programming languages (give the model context on what the language is and how it works, e.g. its docs); and tools that return base64-encoded data into the model's context (remove them).

**安全防护误报。**这些分类器比 Claude Fable 5 发布时的分类器误报更少，且在源代码中寻找漏洞是被允许的；被拦截的请求仍返回 `stop_reason: "refusal"`，因此要保留拒答处理与回退机制。三种情况更容易出现误报：编译检查式措辞（问"Are there any bugs in this program?"而不是"Does this program compile without errors?"）；小众编程语言（给模型提供该语言是什么、如何工作的背景，例如其文档）；以及把 base64 编码数据返回到模型上下文中的工具（移除它们）。

**Whole-file rewrites.** Claude Fable 5.1 is more likely than Claude Fable 5 to rewrite an entire file where a targeted edit would do - same result, more output tokens and time. Appending this to the system prompt (or the first user message - equally effective) restored targeted edits for small and medium changes:

**整文件重写。**在定向编辑就够用的场合，Claude Fable 5.1 比 Claude Fable 5 更倾向于重写整个文件——结果相同，但输出 token 与时间更多。把下面这句追加到系统提示词（或第一条用户消息——效果相同）后，中小改动恢复了定向编辑：

> The number of tokens used to edit files is best minimized, all else being equal. Therefore, when it will not affect the end result, try to surgically edit a file rather than rewrite the entire thing.
> 在其他条件相同时，编辑文件所用的 token 越少越好。因此，在不影响最终结果的前提下，尽量对文件做外科手术式编辑，而不是整个重写。

**Summarization prompt for client-side compaction.** Claude Fable 5.1 responds well to being told explicitly what to retain in a compaction summary. Server-side compaction already does this; if you compact on the client (the simple-compaction shape under breaking change 3), this summarization instruction has been effective. The final sentence is load-bearing when the summarization request still carries the conversation's `tools` (breaking change 3 means you can't drop them for one request): without it the model occasionally calls a tool instead of writing the summary.

**客户端压缩的摘要提示词。**Claude Fable 5.1 对明确告知压缩摘要要保留什么响应良好。服务端压缩已经内置了这一点；如果你在客户端压缩（破坏性变更 3 下的简单压缩形态），这条摘要指令已被证明有效。最后一句在摘要请求仍然携带对话的 `tools` 时是承重句（破坏性变更 3 意味着无法仅为一次请求就丢掉它们）：没有它，模型偶尔会去调用工具而不是写摘要。

> Summarize the transcript inside `<summary></summary>` tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary - stated exactly; (4) exactly where things stand now - what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct - names, numbers, dates, exact wording, links or references - kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced - as long as nothing in the six items above is dropped. Do not call any tools while writing this summary; respond with text only.
> 在 `<summary></summary>` 标签内总结这段对话记录。在摘要中纳入相关信息，使这段对话可以由一个新的上下文窗口接续，而无需重做工作、也无需重新提供相关约束或背景。务必保留：(1) 出现过的任何困难或问题，以及它们是如何处理或解决的；(2) 提出过、尝试过或搁置过的任何可能性、选项或方法，以及原因；(3) 任何被要求、决定、同意、排除，或被确立为偏好、约束、边界的内容——按原话精确记录；(4) 当前进展的确切状态——迄今为止覆盖、敲定或完成了什么；(5) 任何仍悬而未决、未解决、已承诺或预计接下来会发生的事；(6) 难以重构的具体细节——名称、数字、日期、确切措辞、链接或引用——精确保留。这些内容宁可长也要完整；其余部分保持简练。对两种声音区别对待：用户说过、要求过、分享过或确立过的内容要仔细保留、贴近其原话；你自己的解释和推理可以大幅压缩到结论或产出了什么——只要上面六项一项不落。写这份摘要时不要调用任何工具；只用文本回复。

**Non-blocking sub-agents in coding.** If the coding agent delegates to sub-agents, Claude Fable 5.1 finishes sooner when the lead is not forced to stop and wait for each one - lower average time to completion at similar quality, token usage, and cost. Have the tool that starts a sub-agent return immediately and deliver the sub-agent's result to the lead in a later user message when ready; the model will still often *choose* to wait, so also give it a separate tool that waits for its sub-agents. The time savings come from the cases where the lead carries on with other work. (This extends the asynchronous-delegation guidance in § Migrating to Claude Fable 5.1 above.)

**编码中的非阻塞子代理。**如果编码代理会把工作委托给子代理，当主代理不必停下来逐个等待时，Claude Fable 5.1 完成得更快——在质量、token 用量与成本相近的情况下，平均完成时间更短。让启动子代理的工具立即返回，并在子代理结果就绪后于稍后的用户消息中把结果交给主代理；模型仍常常*选择*等待，因此还要给它一个单独的、用于等待其子代理的工具。时间节省来自主代理继续做其他工作的那些情形。（这是对上文 § Migrating to Claude Fable 5.1 中异步委托指导的扩展。）

### Claude Fable 5.1 from Claude Fable 5 Migration Checklist / 从 Claude Fable 5 迁移到 Claude Fable 5.1 检查清单

- [ ] **[BLOCKS]** Update the `model=` string to `claude-fable-5-1` (`claude-mythos-5-1` for Project Glasswing participants coming from Claude Mythos 5; confirm access first)
  **[BLOCKS]** 把 `model=` 字符串更新为 `claude-fable-5-1`（来自 Claude Mythos 5 的 Project Glasswing 参与者用 `claude-mythos-5-1`；先确认访问权限）
- [ ] **[BLOCKS]** Remove `tool_choice: {type: "any"}` and `{type: "tool", name: ...}` (400, also on `count_tokens` and Batches) - `auto` plus the instruction in the `user` turn (or an appended `role: "system"` message when the application requires the call), `strict: true` for schema-valid arguments, structured outputs for JSON extraction; delete any retry-on-missing-tool loop that depended on forcing
  **[BLOCKS]** 移除 `tool_choice: {type: "any"}` 与 `{type: "tool", name: ...}`（返回 400，`count_tokens` 与 Batches 上同样）——`auto` 加上 `user` 轮中的指令（或当应用需要该调用时追加一条 `role: "system"` 消息）、`strict: true` 保证参数符合 schema、结构化输出用于 JSON 提取；删除任何依赖强制调用的"缺失工具重试"循环
- [ ] **[BLOCKS]** Coming from an Opus-tier or older model (not from Claude Fable 5): apply the Claude Fable 5.1 Migration Checklist above (the Opus-tier -> Fable migration) first, plus § Coming from Claude Opus 5 - `thinking: {type: "disabled"}` now 400s at any effort, between-tool narration moves into `thinking` blocks, ZDR is lost, price doubles
  **[BLOCKS]** 如果来自 Opus 档或更早的模型（而非 Claude Fable 5）：先应用上文 Claude Fable 5.1 迁移检查清单（Opus 档 -> Fable 迁移），再加上 § Coming from Claude Opus 5——`thinking: {type: "disabled"}` 现在任何努力度都返回 400，工具调用之间的叙述移入 `thinking` 块，ZDR 丢失，价格翻倍
- [ ] **[BLOCKS]** Data retention: 30-day retention required (Covered Model; ZDR only if expressly authorized by Anthropic) - a ZDR org gets `400 invalid_request_error` on every request, as on Claude Fable 5; check the retention configuration before debugging the payload
  **[BLOCKS]** 数据保留：要求 30 天保留（受覆盖模型；仅在获得 Anthropic 明确授权时才支持 ZDR）——与 Claude Fable 5 一样，ZDR 组织的每个请求都会得到 `400 invalid_request_error`；调试负载之前先检查保留配置
- [ ] **[BLOCKS]** Keep passing `thinking` blocks back unchanged on every turn, including empty ones and `redacted_thinking` - the history-editing check rejects edited history
  **[BLOCKS]** 每一轮都把 `thinking` 块原样回传，包括空块和 `redacted_thinking`——历史编辑检查会拒绝被编辑过的历史
- [ ] **[BLOCKS]** Preserved thinking / the history-editing check (new accounts created on/after 2026-08-31 on every platform, and any request that sets `prefix_mismatch_behavior`; enforcement scope is decided per model, and Claude Opus 5.5 also enforces it for new accounts only; Claude Mythos 5.1 doesn't run it): stop editing history between requests - freeze the top-level `system`, use `role: "system"` messages for mid-session instructions, `tool_addition`/`tool_removal` for tool changes, turn-scoped (`clear_at`) system messages - or, without that beta, retained user-message text blocks - appended after the tool results and never deleted, for per-turn reminders, server-side context editing / compaction (summary-only if client-side) for trimming, `file_id` for cross-turn files. Run the three-step check (`prefix_mismatch_behavior: "drop_block"` + log `input_transformations`; fix every `prefix_binding_mismatch`, `model_binding_mismatch` after a model switch is expected; `"error"` in CI; the controls beta is also available on Claude Platform on AWS, Bedrock and Vertex, Foundry unconfirmed - `shared/platform-availability.md`), then pick a production setting and monitor it. If you ship a tool others run with their own key, test with the field set. Keep-tail and background compaction need `"drop_block"` (per request - keep sending it) or stripped thinking on the retained turns; never compact mid tool round
  **[BLOCKS]** 保留思维 / 历史编辑检查（在所有平台上创建于 2026-08-31 或之后的新账户，以及任何设置了 `prefix_mismatch_behavior` 的请求；执行范围按模型决定，Claude Opus 5.5 也只对新账户执行；Claude Mythos 5.1 不运行该检查）：停止在请求之间编辑历史——冻结顶层 `system`，会话中途的指令用 `role: "system"` 消息，工具变更用 `tool_addition`/`tool_removal`，每轮提醒用轮次作用域（`clear_at`）系统消息——或不带该 beta、用保留的用户消息文本块——追加在工具结果之后且永不删除；裁剪用服务端上下文编辑 / 压缩（客户端则仅摘要）；跨轮文件用 `file_id`。运行三步检查（`prefix_mismatch_behavior: "drop_block"` 并记录 `input_transformations`；修复每一个 `prefix_binding_mismatch`，模型切换后的 `model_binding_mismatch` 属预期；CI 中用 `"error"`；该 controls beta 亦可在 Claude Platform on AWS、Bedrock 与 Vertex 上使用，Foundry 未确认——见 `shared/platform-availability.md`），然后选定生产取值并监控。如果你发布让别人用自己的 key 运行的工具，请带上该字段测试。保留尾部压缩与后台压缩需要对保留轮次设置 `"drop_block"`（按请求——持续发送）或剥离其思维块；绝不在工具轮中间压缩
- [ ] **[TUNE]** Fallbacks: keep server-side `fallbacks` (targets `claude-opus-4-8` / `claude-opus-5`; routing unpublished) or the SDK middleware; the fallback model can't read 5.1 thinking blocks (dropped, unbilled); fallback credit works as on Claude Fable 5 for both Claude Fable 5.1 and Claude Mythos 5.1 (handle a `null` token)
  **[TUNE]** 回退：保留服务端 `fallbacks`（目标 `claude-opus-4-8` / `claude-opus-5`；路由不公开）或 SDK 中间件；回退模型读取不了 5.1 的思维块（被丢弃、不计费）；回退积分对 Claude Fable 5.1 与 Claude Mythos 5.1 都与 Claude Fable 5 上相同（要处理 `null` token）
- [ ] **[TUNE]** Adopt `thinking: {type: "adaptive", display: "updates"}` with `thinking-display-updates-2026-08-18` (all platforms) if users watch long tool-calling turns; render non-empty `thinking` blocks as status lines, handle the interrupted-response sentinel, echo them back unchanged
  **[TUNE]** 如果用户会观看长的工具调用轮，采用 `thinking: {type: "adaptive", display: "updates"}` 加 `thinking-display-updates-2026-08-18`（所有平台）；把非空 `thinking` 块渲染为状态行，处理响应被中断的哨兵文本，并原样回传这些块
- [ ] **[TUNE]** Adopt per-message effort (`mid-conversation-output-config-2026-07-01`; also on Claude Opus 5 and Claude Opus 5.5) where a loop mixes hard and routine steps - lowering is reliable, raising wants a big jump; re-run the effort sweep (`high` default; `medium` as cost control; `xhigh`/`max` only for capability-sensitive work; `low` often beats below-frontier models on cost per task); size `max_tokens` for `high`+
  **[TUNE]** 在一个循环混合困难与常规步骤的地方采用每消息努力度（`mid-conversation-output-config-2026-07-01`；Claude Opus 5 与 Claude Opus 5.5 也支持）——调低可靠，调高需要大跨度；重做努力度扫描（默认 `high`；`medium` 用于成本控制；`xhigh`/`max` 仅用于能力敏感的工作；`low` 在每任务成本上常胜过低于前沿的模型）；`high` 及以上要相应设置 `max_tokens`
- [ ] **[TUNE]** Agent loops: measure the share of multi-tool-call turns and add the "privately list what you need next" nudge (fresh copy each turn, earlier copies kept) if it's low; remove "hold findings for the final response" / anti-narration text and anti-formatting rules before adding the progress-update and formatting snippets
  **[TUNE]** 代理循环：测量多工具调用轮的占比，若偏低则添加"先私下列出接下来需要什么"的提示（每轮新副本，更早副本保留）；在添加进度更新与格式化片段之前，先删除"把发现留到最终回复"/反叙述文本与反格式化规则
- [ ] **[TUNE]** Add the autonomy + scope prompts for unattended runs; the hidden-tool-output note if the harness collapses tool output; the targeted-edit and scope/test-coverage prompts for coding agents; the long-deliverable note (with the real `max_tokens`) for `xhigh`/`max` requests; the compaction summarization prompt if you summarize client-side; the mannered-prose instruction for prose-heavy work; the name-verification line for search products; a crop tool (or an image-processing container) for vision
  **[TUNE]** 为无人值守运行添加自主性 + 范围提示词；harness 会折叠工具输出时添加隐藏工具输出说明；编码代理添加定向编辑与范围/测试覆盖提示词；`xhigh`/`max` 请求添加长交付物说明（带真实 `max_tokens`）；客户端做摘要时添加压缩摘要提示词；散文密集的工作添加反套路行文指令；搜索产品添加名称核实条款；视觉任务添加裁剪工具（或图像处理容器）
- [ ] **[TUNE]** Priority Tier is not supported on Claude Fable 5.1; rate limits share the Fable 5.x pool with Claude Fable 5 - re-baseline headroom; cache reads cost a quarter of the Claude Fable 5 rate (re-check caching break-even; for 5-60 minute idle gaps a `max_tokens: 0` keep-alive on the 5-minute TTL usually beats the 1-hour TTL - sent with `stream` off; not with structured outputs or Batches); tokenizer unchanged from Claude Fable 5, so re-baseline token counts only if you weren't on Claude Fable 5
  **[TUNE]** Claude Fable 5.1 不支持 Priority Tier；速率限制与 Claude Fable 5 共享 Fable 5.x 池——重新评估余量基准；缓存读取价格为 Claude Fable 5 的四分之一（重新检查缓存盈亏平衡；对 5-60 分钟的空闲间隔，5 分钟 TTL 上 `max_tokens: 0` 的保活请求通常优于 1 小时 TTL——发送时关闭 `stream`；不要与结构化输出或 Batches 一起使用）；分词器与 Claude Fable 5 相同，因此只有此前不在 Claude Fable 5 上时才需重新评估 token 计数
- [ ] **[TUNE]** Supported image, audio, and video files produced in the code-execution sandbox carry a C2PA manifest when downloaded through the Files API on the Claude API - size and checksum differ from the in-container file (text, PDF, and office files aren't signed; platform scope beyond the Claude API open at launch); adjust integrity checks for signed media only
  **[TUNE]** 在代码执行沙箱中生成的受支持图片、音频与视频文件，经 Claude API 的 Files API 下载时带有 C2PA 清单——大小与校验和不同于容器内文件（文本、PDF 与办公文件不带签名；Claude API 之外的平台范围发布时待定）；只需对带签名的媒体调整完整性检查

---

## Migrating to Claude Opus 5.5 / 迁移到 Claude Opus 5.5

> **Model ID `claude-opus-5-5` is authoritative as written here.** When the user asks to migrate to Claude Opus 5.5, write `model="claude-opus-5-5"` exactly. Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entry exists in `shared/models.md`.
> **模型 ID `claude-opus-5-5` 以此处所写为准。**当用户要求迁移到 Claude Opus 5.5 时，请精确写出 `model="claude-opus-5-5"`。**不要**用 WebFetch 去验证——本指南是迁移目标 ID 的事实来源。`shared/models.md` 中存在对应条目。
【评论】"不要用 WebFetch 验证"是一条防漂移指令：迁移目标 ID 以本指南为准，避免模型在运行时抓取到过期或错误的模型名。

Claude Opus 5.5 succeeds Claude Opus 5 in the Opus line for long-running agentic coding and knowledge work, **at a lower price** - $4 / $20 per MTok input / output against Claude Opus 5's $5 / $25. Same 1M token context window (default and maximum), same 128K max output, same tokenizer as Claude Opus 5 (token counts unchanged; coming from a pre-Opus-4.7 model, follow the tokenizer guidance in the Claude Opus 5 section). Knowledge cutoff June 2026. Available at launch on the Claude API (`claude-opus-5-5`), Amazon Bedrock (`anthropic.claude-opus-5-5`), Claude Platform on AWS, Google Cloud, and Microsoft Foundry (all as `claude-opus-5-5`; on Foundry under both hosting options, "Hosted on Anthropic" and "Hosted on Azure"); Claude Opus 5 stays available on all of them. Existing Claude Opus 5 prompts should perform well out of the box; the Claude Opus 5 prompting patterns below remain a reasonable starting point.

Claude Opus 5.5 在 Opus 产品线中接替 Claude Opus 5，面向长时间运行的代理式编码与知识工作，**价格更低**——每 MTok 输入/输出 $4 / $20，而 Claude Opus 5 为 $5 / $25。同样 1M token 上下文窗口（默认与最大），同样 128K 最大输出，与 Claude Opus 5 相同的分词器（token 计数不变；若来自 Opus 4.7 之前的模型，请遵循 Claude Opus 5 一节中的分词器指导）。知识截止日期为 2026 年 6 月。发布时可在 Claude API（`claude-opus-5-5`）、Amazon Bedrock（`anthropic.claude-opus-5-5`）、Claude Platform on AWS、Google Cloud 与 Microsoft Foundry 上使用（均以 `claude-opus-5-5`；在 Foundry 上两种托管选项皆可——"Hosted on Anthropic" 与 "Hosted on Azure"）；Claude Opus 5 在所有这些平台上继续可用。现有 Claude Opus 5 提示词应当开箱即有良好表现；下文的 Claude Opus 5 提示词模式仍是合理的起点。

**Claude Opus 5.5 is the default Opus migration target.** This section is layered on top of the Claude Opus 5 migration above - a caller coming from Opus 4.8 or older applies § Migrating to Claude Opus 5 first (Opus 4.7 or older: the sections before that), with two exceptions to what that section (and the earlier ones) say: Claude Opus 5's "thinking can be disabled at `high` or below" does not carry over, and neither does the acceptance of the earlier `computer_20251124` tool outside Amazon Bedrock (breaking change 4 below). Coming from Claude Sonnet 5: the request surface already matches (adaptive thinking, no sampling parameters, no prefill) - apply this section on top of the Claude Sonnet 5 code, re-baselining for Opus-tier pricing and rate limits.

**Claude Opus 5.5 是默认的 Opus 迁移目标。**本节叠加在上文 Claude Opus 5 迁移之上——来自 Opus 4.8 或更早的调用方先应用 § Migrating to Claude Opus 5（Opus 4.7 或更早：其前面的章节），但该节（以及更早章节）所说内容有两处例外：Claude Opus 5 的"思维可在 `high` 或以下关闭"不再沿用，`computer_20251124` 旧工具在 Amazon Bedrock 之外的接受也不沿用（见下方破坏性变更 4）。来自 Claude Sonnet 5：请求表面已经匹配（自适应思维、无采样参数、无预填充）——在 Claude Sonnet 5 代码之上应用本节，并按 Opus 档定价与速率限制重新校准。

**What changes, in one line:** four breaking changes for code running on Claude Opus 5 (thinking can't be disabled; forced `tool_choice` 400s; thinking blocks are tied to the model and the conversation - "preserved thinking"; on the Claude API and Google Cloud the `computer_20251124` tool 400s - use the computer toolset), one response-shape change that fails no request (text between tool calls comes back in `thinking` blocks), a **default effort of `medium`** where Claude Opus 5's is `high`, and a broader safety-classifier set (`bio` and `reasoning_extraction` join `cyber`). The first three breaking changes are the same mechanisms Claude Fable 5.1 introduced - the sections below give the Claude Opus 5.5 specifics and point at § Migrating to Claude Fable 5.1 from Claude Fable 5 for the shared mechanics rather than repeating them. Everything else in the Claude Opus 5 request surface carries over: mid-conversation system messages and per-message effort (which some of the tips below use), mid-conversation tool changes, task budgets, compaction, the 512-token minimum cacheable prompt, batch, the Files API, PDF support, vision, and the server-side and client-side tools.

**一句话概括变更内容：**对运行在 Claude Opus 5 上的代码有四项破坏性变更（思维无法关闭；强制 `tool_choice` 返回 400；思维块与模型及对话绑定——"保留思维"；在 Claude API 与 Google Cloud 上 `computer_20251124` 工具返回 400——改用计算机工具集），一处不导致任何请求失败的响应形态变化（工具调用之间的文本以 `thinking` 块返回），**默认努力度为 `medium`**（Claude Opus 5 为 `high`），以及更广的安全分类器集合（`bio` 与 `reasoning_extraction` 加入 `cyber`）。前三项破坏性变更与 Claude Fable 5.1 引入的是相同机制——下文各节给出 Claude Opus 5.5 的具体细节，共享机制指向 § Migrating to Claude Fable 5.1 from Claude Fable 5 而不重复。Claude Opus 5 请求表面的其余一切照旧沿用：会话中途系统消息与每消息努力度（下文部分技巧会用到）、会话中途工具变更、任务预算、压缩、512 token 最小可缓存提示词、批处理、Files API、PDF 支持、视觉，以及服务端与客户端工具。

### Breaking change 1: thinking can't be disabled / 破坏性变更 1：思维无法关闭

On Claude Opus 5, thinking is on by default and `thinking: {type: "disabled"}` is accepted at effort `high` or below. On Claude Opus 5.5 thinking is **always on**: `{"type": "disabled"}` and `{"type": "enabled", "budget_tokens": N}` both return a 400 `invalid_request_error` at every effort level, with no beta header involved:

在 Claude Opus 5 上，思维默认开启，且 `thinking: {type: "disabled"}` 在 `high` 或以下的努力度可被接受。在 Claude Opus 5.5 上思维**始终开启**：`{"type": "disabled"}` 与 `{"type": "enabled", "budget_tokens": N}` 在每个努力度级别都返回 400 `invalid_request_error`，不涉及任何 beta 请求头：

```text
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

(`"thinking.type.enabled" is not supported for this model. ...` for the budget form.) Omit the `thinking` field or send `{type: "adaptive"}`, which is equivalent. **Effort is now the control for how much the model thinks, and therefore for latency and cost** (§ Choosing an effort level below). Migrate a route that disables thinking or sets a budget as follows:

（预算形式返回 `"thinking.type.enabled" is not supported for this model. ...`。）省略 `thinking` 字段或发送 `{type: "adaptive"}`，两者等价。**努力度现在是控制模型思考多少的旋钮，因而也是控制延迟与成本的旋钮**（见下文 § Choosing an effort level）。按如下步骤迁移关闭思维或设置预算的路由：

1. **Remove the `thinking` field** (or set it to `{type: "adaptive"}`).
   **移除 `thinking` 字段**（或设为 `{type: "adaptive"}`）。
2. **If time to first token matters, set `output_config.effort` to `low`.** At `low` the model keeps its thinking short; how often it skips thinking altogether depends on the prompts. Measure, and move to `medium` if quality drops. A system-prompt line such as *"Answer directly without deliberating."* can reduce thinking further (and with it TTFT and cost) - measure quality on your own use case before keeping it, since less thinking can cost accuracy.
   **如果首 token 时间重要，把 `output_config.effort` 设为 `low`。**在 `low` 下模型保持思考简短；它在多大程度上完全跳过思考取决于提示词。先测量，若质量下降再移到 `medium`。类似 *"Answer directly without deliberating."* 的系统提示词语句可以进一步减少思考（并随之降低 TTFT 与成本）——保留之前请先在自己的用例上测量质量，因为更少的思考可能牺牲准确率。
3. **Size `max_tokens` for the thinking as well as the reply.** Thinking counts toward `max_tokens` even though its text isn't returned under the default `display`, so a limit sized for a no-thinking route cuts replies off. For long agentic coding turns, 64K has worked well.
   **为思考以及回复设置 `max_tokens`。**尽管在默认 `display` 下思考文本不会返回，思考仍计入 `max_tokens`，因此按无思维路由设定的上限会截断回复。对长的代理式编码轮，64K 效果良好。
4. **Read the response by block `type`, not position.** A response can begin with one or more `thinking` blocks; under the default `display: "omitted"` they come back with an empty `thinking` string. Set `display: "summarized"` for a readable summary of the reasoning. Pass `thinking` blocks back unmodified in tool-use loops.
   **按块的 `type` 而非位置读取响应。**响应可能以一个或多个 `thinking` 块开头；在默认 `display: "omitted"` 下它们返回为空的 `thinking` 字符串。设置 `display: "summarized"` 可获得可读的推理摘要。在工具使用循环中原样回传 `thinking` 块。

```python
# Before - accepted on Claude Opus 5, 400 on Claude Opus 5.5
client.messages.create(
    model="claude-opus-5",
    max_tokens=16000,
    thinking={"type": "disabled"},
    messages=[{"role": "user", "content": "..."}],
)

# After - thinking is always on; effort is the control
client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    output_config={"effort": "low"},
    messages=[{"role": "user", "content": "..."}],
)
```

**Prompts written for thinking disabled.** Three follow-ups if the Claude Opus 5 integration ran with thinking off: (a) start at `low` and measure, as above; (b) **remove instructions that stood in for thinking** - a prompt that asked the model to write its reasoning into the response text as a substitute for thinking should go, and the reasoning read from `display: "summarized"` blocks instead; a prompt that pushes the model to reproduce its internal reasoning in the response can be **declined** with `stop_details.category: "reasoning_extraction"`; (c) re-test the two thinking-disabled mitigations from § Two failure modes when thinking is disabled under Claude Opus 5 - both address artifacts that appeared only with thinking off, so check whether the combined "brief sentence before a tool call / say so if no tool fits / no internal XML tags" instruction is still needed, and **delete any rule telling the model not to think either way** (it can't comply, and such rules increase tag leakage).

**为关闭思维而写的提示词。**如果 Claude Opus 5 集成是在思维关闭状态下运行的，有三项后续工作：(a) 从 `low` 开始并测量，如上所述；(b) **移除代替思考的指令**——如果某个提示词让模型把推理写进响应文本以代替思考，应当删去，改为从 `display: "summarized"` 块读取推理；如果某个提示词推动模型在响应中复现其内部推理，可能被以 `stop_details.category: "reasoning_extraction"` **拒答**；(c) 重新测试 § Two failure modes when thinking is disabled under Claude Opus 5 中的两项思维关闭缓解措施——两者针对的都是仅在思维关闭时出现的伪影，因此检查"工具调用前一句话 / 没有合适工具就明说 / 不写内部 XML 标签"这条组合指令是否仍有必要，并**删除任何一条告诉模型无论哪种方式都不要思考的规则**（它无法遵守，且这类规则会加重标签泄漏）。

### Breaking change 2: forced tool use is rejected / 破坏性变更 2：强制工具使用被拒绝

As on Claude Fable 5.1: `tool_choice: {"type": "any"}` and `{"type": "tool", "name": "..."}` return a 400 `invalid_request_error` (`tool_choice: type "tool" and "any" are not supported for this model.`) on the Messages API, the Message Batches API, and the token-counting endpoint, where Claude Opus 5 accepts both. `{"type": "auto"}` (the default) and `{"type": "none"}` are unchanged; `disable_parallel_tool_use: true` still works with `auto` but now means *at most* one call. Migrate by intent - the full patterns (steering from the prompt, `strict: true` for schema-valid arguments, structured outputs for extraction, the advisor tool) are under § Breaking change 1: forced tool use is rejected in § Migrating to Claude Fable 5.1 from Claude Fable 5. The two most common:

与 Claude Fable 5.1 相同：`tool_choice: {"type": "any"}` 与 `{"type": "tool", "name": "..."}` 返回 400 `invalid_request_error`（`tool_choice: type "tool" and "any" are not supported for this model.`），涵盖 Messages API、Message Batches API 与 token 计数端点，而这些地方 Claude Opus 5 两者都接受。`{"type": "auto"}`（默认值）与 `{"type": "none"}` 不变；`disable_parallel_tool_use: true` 与 `auto` 搭配仍然有效，但现在意味着*至多*一次调用。按迁移意图处理——完整模式（从提示词引导、`strict: true` 保证参数符合 schema、结构化输出用于提取、advisor 工具）见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中 § Breaking change 1: forced tool use is rejected。最常见的两种：

- **Steering toward a tool:** `tool_choice: {"type": "auto"}` plus the expectation in the prompt ("Use the `get_weather` tool to answer"), with `strict: true` on the tool definition (schema sets `additionalProperties: false`) so the arguments match the schema. Because `auto` does not guarantee a call, **check that one was made and retry if it wasn't.**
  **引导模型使用某个工具：**`tool_choice: {"type": "auto"}` 加提示词中的期望（"Use the `get_weather` tool to answer"），并在工具定义上设 `strict: true`（schema 设 `additionalProperties: false`）使参数符合 schema。由于 `auto` 不保证一定调用，**要检查调用是否发生，未发生则重试。**
- **Extracting structured data:** if the forced call existed only to get JSON back, replace it with structured outputs (`output_config.format`).
  **提取结构化数据：**如果强制调用只是为了拿回 JSON，改用结构化输出（`output_config.format`）。

```python
# Before - 400 on Claude Opus 5.5
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=1024,
    tools=tools,
    tool_choice={"type": "tool", "name": "get_weather"},
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
)

# After - auto + strict tool use, steering in the prompt, and a check that the call happened
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    tools=[{**tool, "strict": True} for tool in tools],
    tool_choice={"type": "auto"},
    messages=[{"role": "user", "content": "What's the weather in Paris? Use the get_weather tool."}],
)
if not any(block.type == "tool_use" for block in response.content):
    ...  # retry, or fall back to a text answer
```

### Breaking change 3: thinking blocks are tied to the model and the conversation / 破坏性变更 3：思维块与模型及对话绑定

Both halves of "preserved thinking" from Claude Fable 5.1 apply to Claude Opus 5.5; the mechanics (what invalidates a block, the `drop_block` request shape, `input_transformations`, the three-step audit, the append-only replacements table, which client-side compaction shapes break) are under § Breaking change 2 and § Breaking change 3 in § Migrating to Claude Fable 5.1 from Claude Fable 5 and apply verbatim. What is specific to Claude Opus 5.5:

Claude Fable 5.1 的"保留思维"两部分都适用于 Claude Opus 5.5；其机制（什么会使块失效、`drop_block` 请求形态、`input_transformations`、三步审计、只追加替换表、哪些客户端压缩形态会失效）见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中 § Breaking change 2 与 § Breaking change 3，逐字适用。Claude Opus 5.5 特有的部分：

- **Model binding - who reads whose blocks.** Claude Opus 5.5 reads thinking blocks from Claude Opus 5 and earlier Opus, Sonnet, and Haiku models - a conversation that *moves onto* `claude-opus-5-5` keeps its reasoning - but **not** from any Fable or Mythos model. In the other direction, on the Claude API only Claude Fable 5.1 and Claude Mythos 5.1 read a Claude Opus 5.5 block; **no other model does** - so a router switch, a client-side retry on another model, or a classifier-refusal fallback (server-side or SDK middleware) to Claude Opus 5 / Claude Opus 4.8 runs the turns after the switch without Claude Opus 5.5's reasoning. The API drops what the target can't read before the model sees it: the request succeeds, dropped blocks aren't billed, and with the `thinking-binding-controls-2026-08-01` header the drop is reported in `input_transformations` with `reason: "model_binding_mismatch"`. Whether Claude Fable 5.1 / Claude Mythos 5.1 also read Claude Opus 5.5's blocks on Amazon Bedrock and Google Cloud is not documented - the docs state it for the Claude API only. Keep passing blocks back unchanged when you switch models; don't strip them yourself.
  **模型绑定——谁读谁的块。**Claude Opus 5.5 读取来自 Claude Opus 5 与更早 Opus、Sonnet、Haiku 模型的思维块——*切换到* `claude-opus-5-5` 的对话保留其推理——但**不**读取任何 Fable 或 Mythos 模型的块。反方向，在 Claude API 上只有 Claude Fable 5.1 与 Claude Mythos 5.1 能读 Claude Opus 5.5 的块；**没有其他模型能读**——因此路由切换、客户端在另一模型上重试，或到 Claude Opus 5 / Claude Opus 4.8 的分类器拒答回退（服务端或 SDK 中间件），会让切换之后的轮次在没有 Claude Opus 5.5 推理的情况下运行。API 会在模型看到之前丢弃目标读取不了的部分：请求成功，被丢弃的块不计费，带 `thinking-binding-controls-2026-08-01` 请求头时，丢弃会以 `reason: "model_binding_mismatch"` 报告在 `input_transformations` 中。Claude Fable 5.1 / Claude Mythos 5.1 在 Amazon Bedrock 与 Google Cloud 上是否也读 Claude Opus 5.5 的块，文档没有说明——文档只针对 Claude API 陈述了这一点。切换模型时请继续原样回传块；不要自行剥离。
- **Conversation binding - who is enforced.** Same posture as Claude Fable 5.1: on every platform the prefix check (the `system` prompt, the `tools` array, and every earlier message must be byte-identical to when the block was produced) is enforced by default for accounts **created on or after August 31, 2026, 00:00 UTC** - a replayed block after such an edit is a 400. Older accounts opt in by setting `thinking.block_binding.prefix_mismatch_behavior` (`"error"` or `"drop_block"`, beta `thinking-binding-controls-2026-08-01`). Claude Code, claude.ai, Managed Agents, and the Agent SDK keep the prefix intact; **if your code builds `messages` itself, run the three-step check before migrating** - and do it now even on an exempt account, because it also raises prompt-cache hit rates. The three edits that break the prefix and their append-only replacements: a per-turn reminder injected and later deleted, or a system prompt changed mid-session (append a mid-conversation `role: "system"` message instead; for a one-turn reminder the `clear_at: "next_user_message"` form under beta `mid-conversation-system-clear-at-2026-08-21` is a limited beta - without it, append the reminder as a text block after the `tool_result` blocks and leave earlier copies in place); tools added or removed mid-session (declare the full set at session start and send `tool_addition` / `tool_removal` blocks, beta `mid-conversation-tool-changes-2026-07-01`); and compaction that summarizes older turns while replaying newer ones with their thinking blocks (use server-side compaction or context editing - the on-demand `compaction` parameter under beta `compact-2026-09-04`, offered on the Claude API, Claude Platform on AWS, Google Cloud, and Microsoft Foundry but not yet Amazon Bedrock, is designed to keep the retained turns' blocks valid after the swap - or client-side *simple* compaction that replaces the whole history with a summary and replays no earlier thinking, or set `drop_block`). Two compaction details that follow Claude Fable 5.1: a threshold-compaction request with custom `instructions` summarizes from the visible conversation only - earlier thinking blocks are not part of the summarizer's input, so tell it what the summary must retain (on-demand compaction's summarizer reads earlier thinking with or without `instructions`); and any assistant turn you re-insert after a compaction block needs its `thinking` / `redacted_thinking` blocks removed, or `drop_block` set.
  **对话绑定——谁被强制执行。**与 Claude Fable 5.1 相同的立场：在所有平台上，前缀检查（`system` 提示词、`tools` 数组与每条更早的消息必须与块产生时逐字节相同）对**创建于 2026 年 8 月 31 日 00:00 UTC 或之后**的账户默认强制执行——在这类编辑之后重放块会得到 400。更早的账户通过设置 `thinking.block_binding.prefix_mismatch_behavior`（`"error"` 或 `"drop_block"`，beta `thinking-binding-controls-2026-08-01`）选择加入。Claude Code、claude.ai、Managed Agents 与 Agent SDK 会保持前缀完整；**如果你的代码自行构建 `messages`，请在迁移前运行三步检查**——即使账户被豁免也要现在就做，因为它还能提高提示词缓存命中率。三种破坏前缀的编辑及其只追加替代：注入后又删除的每轮提醒，或会话中途更改系统提示词（改为追加会话中途 `role: "system"` 消息；单轮提醒的 `clear_at: "next_user_message"` 形式在 beta `mid-conversation-system-clear-at-2026-08-21` 下是受限 beta——没有它，把提醒作为 `tool_result` 块之后的文本块追加并保留更早副本）；会话中途增删工具（会话开始时声明完整集合并发送 `tool_addition` / `tool_removal` 块，beta `mid-conversation-tool-changes-2026-07-01`）；以及摘要较早轮次却带着思维块重放较新轮次的压缩（用服务端压缩或上下文编辑——beta `compact-2026-09-04` 下的按需 `compaction` 参数在 Claude API、Claude Platform on AWS、Google Cloud 与 Microsoft Foundry 上提供但尚不在 Amazon Bedrock 上，其设计目标就是在换入后保持被保留轮次块的有效性——或客户端*简单*压缩，用摘要替换全部历史且不重放更早思维，或设置 `drop_block`）。两项沿用 Claude Fable 5.1 的压缩细节：带自定义 `instructions` 的阈值压缩请求只从可见对话做摘要——更早的思维块不在摘要器输入中，因此要告诉它摘要必须保留什么（按需压缩的摘要器无论有无 `instructions` 都会读取更早的思维）；以及在压缩块之后重新插入的任何 assistant 轮都需要移除其 `thinking` / `redacted_thinking` 块，或设置 `drop_block`。

```http
POST /v1/messages
anthropic-beta: thinking-binding-controls-2026-08-01

{"model": "claude-opus-5-5", "max_tokens": 64000,
 "thinking": {"type": "adaptive", "block_binding": {"prefix_mismatch_behavior": "drop_block"}},
 "messages": [ ...full history with thinking blocks replayed verbatim... ]}
```

### Breaking change 4: computer use only through the computer toolset / 破坏性变更 4：计算机使用只能通过计算机工具集

> **Re-check before promising computer use on a partner platform.** Which platforms offer the toolset can change - WebFetch the Computer Use page from `shared/live-sources.md` and read its Compatibility section.
> **在合作方平台上承诺计算机使用之前请重新核实。**哪些平台提供工具集可能变化——用 WebFetch 抓取 `shared/live-sources.md` 中的 Computer Use 页面并阅读其 Compatibility 一节。

Claude Opus 5 accepts computer use both as the `computer_toolset_20260801` toolset and, with the `computer-use-2025-11-24` beta header, as the earlier `computer_20251124` tool. **On the Claude API and Google Cloud, Claude Opus 5.5 accepts only the toolset**: a `tools` entry of type `computer_20251124` returns a 400 `invalid_request_error` that names the rejected type and then lists the accepted ones after `Did you mean one of` (it begins `'claude-opus-5-5' does not support tool types: computer_20251124.`). The toolset is GA on the Claude API and Google Cloud with no beta header. On Amazon Bedrock, Claude Opus 5.5 still accepts `computer_20251124` (with its beta header), as Claude Opus 5 does - keep that version there. For other platforms, check the computer use tool's Compatibility section (`shared/tool-use-concepts.md` § Computer Use has the toolset summary). This is more than a `tools`-entry swap, so make and test the change on Claude Opus 5 first (it accepts both forms):

Claude Opus 5 既接受 `computer_toolset_20260801` 工具集形式的计算机使用，也带 `computer-use-2025-11-24` beta 请求头接受较早的 `computer_20251124` 工具。**在 Claude API 与 Google Cloud 上，Claude Opus 5.5 只接受工具集**：类型为 `computer_20251124` 的 `tools` 条目返回 400 `invalid_request_error`，错误信息指名被拒绝的类型，然后在 `Did you mean one of` 之后列出接受的类型（开头是 `'claude-opus-5-5' does not support tool types: computer_20251124.`）。工具集在 Claude API 与 Google Cloud 上已正式可用（GA），无需 beta 请求头。在 Amazon Bedrock 上，Claude Opus 5.5 仍接受 `computer_20251124`（带其 beta 请求头），与 Claude Opus 5 一样——在那里保留该版本。其他平台请查看计算机使用工具的 Compatibility 一节（`shared/tool-use-concepts.md` § Computer Use 有工具集摘要）。这不止是换一个 `tools` 条目，因此请先在 Claude Opus 5 上完成并测试改动（它两种形式都接受）：

- **Request:** drop the beta header and the beta client namespace; the entry is `{"type": "computer_toolset_20260801"}` with **no `name`** and no `display_width_px` / `display_height_px`; an optional `configs` map turns individual member tools on or off (`{"zoom": {"enabled": false}}`). All 17 members, `zoom` included, are on by default. The entry can't share a request with a `computer_20251124` entry or another tool named `computer`.
  **请求：**去掉 beta 请求头与 beta 客户端命名空间；条目是 `{"type": "computer_toolset_20260801"}`，**没有 `name`**，也没有 `display_width_px` / `display_height_px`；可选的 `configs` 映射可开或关个别成员工具（`{"zoom": {"enabled": false}}`）。全部 17 个成员（包括 `zoom`）默认开启。该条目不能与 `computer_20251124` 条目或另一个名为 `computer` 的工具同处一个请求。
- **Agent loop:** Claude's calls are `tool_use` blocks whose `name` is the member (`screenshot`, `left_click`, `type`, `zoom`, ...) - **the action is the block's `name`, not `input.action`** - carrying `"toolset_name": "computer"`, and there can be **several per turn** (a batch action), each its own block. Return one `tool_result` per `tool_use`, matched by `tool_use_id`, all in the next `user` message, **every one echoing `"toolset_name": "computer"`** (a result that omits it is rejected); only `screenshot` and `zoom` results need an image, a short `OK` is enough for the rest. Coordinates are in the pixel space of the full screenshots you return, also after a `zoom`. Screenshots must already fit the model's image limits (the toolset takes no display dimensions and the API doesn't downscale for you).
  **代理循环：**Claude 的调用是 `tool_use` 块，其 `name` 为成员名（`screenshot`、`left_click`、`type`、`zoom` 等）——**动作是块的 `name`，不是 `input.action`**——携带 `"toolset_name": "computer"`，且**每轮可以有多个**（批量动作），每个是独立的块。每个 `tool_use` 返回一个 `tool_result`，按 `tool_use_id` 匹配，全部放在下一条 `user` 消息中，**每一个都要回显 `"toolset_name": "computer"`**（省略它的结果会被拒绝）；只有 `screenshot` 与 `zoom` 的结果需要图片，其余一个简短的 `OK` 即可。坐标位于你返回的完整截图的像素空间中，`zoom` 之后亦然。截图必须已经符合模型的图片限制（工具集不接受显示尺寸，API 也不会替你缩小）。

```python
# Before - 400 on Claude Opus 5.5
client.beta.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    betas=["computer-use-2025-11-24"],
    tools=[{"type": "computer_20251124", "name": "computer",
            "display_width_px": 1024, "display_height_px": 768}],
    messages=[{"role": "user", "content": "Open the display settings."}],
)

# After - no beta header; the toolset entry takes no name or display size
client.messages.create(
    model="claude-opus-5-5",
    max_tokens=4096,
    tools=[{"type": "computer_toolset_20260801"}],
    messages=[{"role": "user", "content": "Open the display settings."}],
)
```

Integrations already on the toolset, and the browser use toolset (`browser_toolset_20260801`), need no change.

已经在用工具集的集成，以及浏览器使用工具集（`browser_toolset_20260801`），无需改动。

### Text between tool calls comes back in thinking blocks / 工具调用之间的文本以思维块返回

On Claude Opus 5, the short notes the model writes between tool calls (what it just found, what it's doing next) come back as `text` blocks. On Claude Opus 5.5, as on Claude Fable 5.1, notes longer than a sentence or two come back as **progress-update `thinking` blocks**, at most one before each tool call, and under the default `display: "omitted"` their text is empty - no request fails, but a client that renders only `text` blocks goes quiet for the length of a long agentic turn. Fix: set `thinking.display: "updates"` (beta `thinking-display-updates-2026-08-18`) to get a short summary of each note as text while reasoning stays hidden (`"summarized"` returns both, mixed), render each non-empty `thinking` block ahead of the `tool_use` it precedes, and pass the blocks back unchanged - the consumption rules (streaming `thinking_delta`, the interrupted-work sentinel, zero-or-more per response) are under addition 3 in § New API features of § Migrating to Claude Fable 5.1 from Claude Fable 5:

在 Claude Opus 5 上，模型写在工具调用之间的简短说明（刚发现了什么、接下来做什么）以 `text` 块返回。在 Claude Opus 5.5 上，与 Claude Fable 5.1 一样，超过一两句的说明以**进度更新 `thinking` 块**返回，每次工具调用之前至多一条，且在默认 `display: "omitted"` 下文本为空——没有请求会失败，但只渲染 `text` 块的客户端会在长代理轮期间一直沉默。解法：设置 `thinking.display: "updates"`（beta `thinking-display-updates-2026-08-18`），以文本获取每条说明的简短摘要而推理保持隐藏（`"summarized"` 两者都返回、混合在一起），把每个非空 `thinking` 块渲染在它前面的 `tool_use` 之前，并原样回传这些块——消费规则（流式 `thinking_delta`、工作被中断的哨兵文本、每响应零或多个）见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中 § New API features 的新增项 3：

```http
POST /v1/messages
anthropic-beta: thinking-display-updates-2026-08-18

{"model": "claude-opus-5-5", "max_tokens": 64000,
 "thinking": {"type": "adaptive", "display": "updates"},
 "tools": [...],
 "messages": [{"role": "user", "content": "Review the PRs open against our billing service."}]}
```

Three levers on what users see:

面向用户可见内容有三个调节手段：

1. **Receive them** - `display: "updates"` as above (a `"summarized"` display returns them too, mixed with the reasoning summaries).
   **接收它们**——如上设置 `display: "updates"`（`"summarized"` 显示也会返回它们，与推理摘要混在一起）。
2. **If the model may need to hand the user something *verbatim*** partway through a long turn - a code snippet, an exact value - give it a simple tool for sending the user a message and tell it to reserve the tool for that content. Declare the tool in `tools` from the **first** request of the session: adding it later edits the conversation's prefix and invalidates earlier thinking blocks (breaking change 3).
   **如果模型可能需要在长轮中途把某些内容*逐字*交给用户**——一段代码、一个精确值——给它一个向用户发送消息的简单工具，并告诉它把该工具留给这类内容。在会话的**第一个**请求中就在 `tools` 里声明该工具：之后添加会编辑对话前缀并使更早思维块失效（破坏性变更 3）。
3. **For more frequent or predictable updates** - a one-line statement of intent before the first tool call and a short recap at the end - say so in the system prompt: when you want user-facing text and what it should contain. The model follows such instructions reasonably well; this helps most in pair programming and other human-in-the-loop work.
   **想要更频繁或可预期的更新**——第一次工具调用前一句意图说明、结尾一段简短回顾——在系统提示词中说明：你何时想要面向用户的文本以及它应包含什么。模型对这类指令的遵循相当好；这在结对编程与其他人在回路的工作中帮助最大。

### Choosing an effort level - the default is `medium`, and the levels don't map 1:1 from Claude Opus 5 / 选择努力度级别——默认值为 `medium`，且级别与 Claude Opus 5 不再一一对应

Effort is the main control for how much Claude Opus 5.5 thinks, and with adaptive-only thinking it is the first setting to adjust when trading off intelligence, latency, and cost. Two things change from Claude Opus 5:

努力度是控制 Claude Opus 5.5 思考多少的主要旋钮；在只有自适应思维的情况下，它是权衡智能、延迟与成本时首先调整的设置。相对 Claude Opus 5 有两处变化：

- **The API default is `medium`** (Claude Opus 5 and earlier Opus models default to `high`), so a request that omits `effort` now runs one level lower than it did. **Set `effort` explicitly** and re-run the sweep rather than carrying the Claude Opus 5 setting over. Effort names don't mean the same amount of thinking across models: in Anthropic's testing, Claude Opus 5.5 at `medium` exceeds Claude Opus 5 at `high` on coding and knowledge-work evaluations, and on several coding evaluations `low` comes close to it at much lower cost. Start at `medium` and test the neighboring levels; reserve `xhigh` and `max` for work where you have measured a quality gain (all five levels are supported; `max` is uncapped).
  **API 默认值是 `medium`**（Claude Opus 5 与更早的 Opus 模型默认 `high`），因此省略 `effort` 的请求现在会比以前低一个级别运行。**显式设置 `effort`** 并重做扫描，而不是照搬 Claude Opus 5 的设置。努力度名称在不同模型上不代表相同的思考量：在 Anthropic 的测试中，Claude Opus 5.5 在 `medium` 下在编码与知识工作评测上超过 `high` 下的 Claude Opus 5，而且在若干编码评测上 `low` 以低得多的成本接近该表现。从 `medium` 开始并测试相邻级别；把 `xhigh` 与 `max` 留给已测量到质量收益的工作（五个级别都支持；`max` 不设上限）。
- **At a given level, Claude Opus 5.5 tends to think more per turn than Claude Opus 5**, especially at `xhigh` and `max`. If you keep the `effort` value you set for Claude Opus 5, expect longer turns and more output tokens. To get less thinking, **lower the effort level before adding "think less" instructions** - lowering effort reduces thinking, cost, and latency more reliably than prompting does. Set `max_tokens` with room for the thinking as well as the reply (thinking counts toward it even though the text isn't returned - a limit sized for Claude Opus 5 with thinking off can cut replies off; 64K is a reasonable starting point for long agentic coding turns).
  **在同一级别下，Claude Opus 5.5 每轮思考得比 Claude Opus 5 多**，在 `xhigh` 与 `max` 尤其如此。如果你沿用为 Claude Opus 5 设置的 `effort` 值，预期轮次更长、输出 token 更多。想让思考更少，**先调低努力度级别，再加"少思考"类指令**——调低努力度比提示词更可靠地降低思考、成本与延迟。设置 `max_tokens` 时给思考与回复都留出空间（即使文本不返回，思考也计入其中——按关闭思维的 Claude Opus 5 设定的上限可能截断回复；对长代理式编码轮，64K 是合理的起点）。

Change effort for individual turns without invalidating the prompt cache with a **per-message effort change** (beta `mid-conversation-output-config-2026-07-01`; the request shape is addition 1 under § New API features of § Migrating to Claude Fable 5.1 from Claude Fable 5, and Claude Opus 5 already supports it). Changing the **top-level** `effort` between requests does invalidate the prompt cache.

用**每消息努力度变更**（beta `mid-conversation-output-config-2026-07-01`；请求形态见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中 § New API features 的新增项 1，且 Claude Opus 5 已支持）在不使提示词缓存失效的情况下为个别轮次改变努力度。在请求之间修改**顶层** `effort` 会使提示词缓存失效。

### Safeguards - a broader classifier set than Claude Opus 5 / 安全防护——比 Claude Opus 5 更广的分类器集合

Claude Opus 5.5 runs cybersecurity **and biology** safety classifiers similar to Claude Fable 5.1's; coming from Claude Opus 5, the biology classifier is new. Everyday health and educational questions are unaffected, but requests the classifier treats as dual-use biology research (virology, toxicology, molecular design) are declined; on the cybersecurity side, finding vulnerabilities in source code is allowed. Separately - also new relative to Claude Opus 5 - a request that tries to get the model to reproduce its internal reasoning in the response text can be declined with `stop_details.category: "reasoning_extraction"`; if a prompt does this (for example, to get visible reasoning with thinking off), remove the instruction, set `display: "summarized"`, and read the `thinking` blocks. **`reasoning_extraction` declines are not retried on a fallback model.**

Claude Opus 5.5 运行与 Claude Fable 5.1 类似的网络安全**与生物**安全分类器；来自 Claude Opus 5 的话，生物分类器是新的。日常健康与教育类问题不受影响，但被分类器视为两用生物学研究（病毒学、毒理学、分子设计）的请求会被拒绝；在网络安全方面，在源代码中寻找漏洞是被允许的。另外——相对 Claude Opus 5 也是新的——试图让模型在响应文本中复现其内部推理的请求可能被以 `stop_details.category: "reasoning_extraction"` 拒绝；如果某个提示词这样做（例如为了在思维关闭时获得可见推理），移除该指令、设置 `display: "summarized"` 并读取 `thinking` 块。**`reasoning_extraction` 拒答不会在回退模型上重试。**

A classifier decline arrives as a normal HTTP 200 with `stop_reason: "refusal"` and a `stop_details` object naming the category (`"cyber"`, `"bio"`, `"reasoning_extraction"`, ...; branch on `stop_reason`, treat `stop_details` as informational - the full handling is § `refusal` stop reason under § Migrating to Claude Fable 5.1). A refusal before any output still counts against your rate limits; for whether it is billed, see [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed). Retry on another model with server-side fallbacks - `fallbacks: "default"` under beta `server-side-fallback-2026-07-01` retries on the model Anthropic recommends for that category, the array form under `server-side-fallback-2026-06-01` names your own targets (§ New API features under § Migrating to Claude Opus 5 has both shapes; read Claude Opus 5.5's permitted targets from `allowed_fallback_models` on its `/v1/models` entry, as § `refusal` stop reason describes - expect Claude Opus 5 / claude-opus-4-8), the SDK middleware on platforms without server-side fallback, or your own retry. A fallback model runs without Claude Opus 5.5's thinking blocks (breaking change 3). **Ship the opt-in from day one**, as the Claude Fable 5.1 section says.

分类器拒答以正常 HTTP 200 到达，带 `stop_reason: "refusal"` 与指名类别的 `stop_details` 对象（`"cyber"`、`"bio"`、`"reasoning_extraction"` 等；按 `stop_reason` 分支，把 `stop_details` 当作参考信息——完整处理见 § Migrating to Claude Fable 5.1 中的 § `refusal` stop reason）。输出之前的拒答仍计入你的速率限制；是否计费见 [How refusals are billed](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。用服务端回退在另一模型上重试——beta `server-side-fallback-2026-07-01` 下的 `fallbacks: "default"` 在 Anthropic 为该类别推荐的模型上重试，`server-side-fallback-2026-06-01` 下的数组形式指定你自己的目标（§ Migrating to Claude Opus 5 的 § New API features 有两种形态；按 § `refusal` stop reason 的描述，从 Claude Opus 5.5 的 `/v1/models` 条目上的 `allowed_fallback_models` 读取其允许目标——预期为 Claude Opus 5 / claude-opus-4-8），在没有服务端回退的平台上用 SDK 中间件，或用你自己的重试。回退模型在没有 Claude Opus 5.5 思维块的情况下运行（破坏性变更 3）。**从第一天起就上线选择加入**，如 Claude Fable 5.1 一节所说。

The classifiers can still flag benign requests - the fallback opt-in is what keeps a false positive from becoming an outage. (The docs give no prompt change that avoids Claude Opus 5.5's false positives; the **Safeguard false positives** tips under § Migrating to Claude Fable 5.1 from Claude Fable 5 are documented for Claude Fable 5.1 only - do not offer them for Claude Opus 5.5.)

分类器仍可能误伤良性请求——回退选择加入正是防止误报演变为服务中断的手段。（文档没有给出避免 Claude Opus 5.5 误报的提示词修改；§ Migrating to Claude Fable 5.1 from Claude Fable 5 下的 **Safeguard false positives** 技巧仅针对 Claude Fable 5.1 记录——不要为 Claude Opus 5.5 提供它们。）

### What carries over unchanged from Claude Opus 5 - and what to re-check / 从 Claude Opus 5 原样沿用的部分——以及需要重新检查的部分

- **Feature set:** per-message effort (beta), mid-conversation system messages (no header) and tool changes (beta), task budgets, compaction (including the on-demand `compaction` parameter, beta `compact-2026-09-04`, which has its own docs), prompt caching with the 512-token minimum, batch processing (up to 300K output tokens with the `output-300k-2026-03-24` beta), the Files API, PDF support, vision, structured outputs, strict tool use, and the same server-side and client-side tools - except computer use, which needs the toolset (breaking change 4). Programmatic tool calling lists the model.
  **特性集合：**每消息努力度（beta）、会话中途系统消息（无请求头）与工具变更（beta）、任务预算、压缩（包括有自己的文档的按需 `compaction` 参数、beta `compact-2026-09-04`）、带 512 token 下限的提示词缓存、批处理（带 `output-300k-2026-03-24` beta 最多 300K 输出 token）、Files API、PDF 支持、视觉、结构化输出、严格工具使用，以及相同的服务端与客户端工具——计算机使用除外，它需要工具集（破坏性变更 4）。程序化工具调用已列出该模型。
- **Pricing:** $4 / $20 per MTok; 5-minute cache writes $5 and 1-hour cache writes $8; **cache reads $0.20 per MTok (0.05x base input)**; batch $2 / $10. The cache-read discount is deeper than Claude Opus 5's, so long agentic sessions that re-read a cached prefix save more, and a miss costs relatively more - keeping the cache warm (per-message effort, append-only histories, the keep-alive patterns in `shared/prompt-caching.md`) matters more.
  **定价：**每 MTok $4 / $20；5 分钟缓存写入 $5，1 小时缓存写入 $8；**缓存读取每 MTok $0.20（基础输入的 0.05 倍）**；批处理 $2 / $10。缓存读取折扣比 Claude Opus 5 更深，因此重读缓存前缀的长代理会话节省更多，而未命中的相对代价更高——保持缓存温热（每消息努力度、只追加历史、`shared/prompt-caching.md` 中的保活模式）更加重要。
- **Rate limits:** a separate pool from Claude Opus 5's, with its own per-tier numbers. Re-check your tier's Claude Opus 5.5 limits before moving volume.
  **速率限制：**与 Claude Opus 5 不同的独立池，各档位数字独立。迁移流量前先重新检查你所在档位的 Claude Opus 5.5 限额。
- **Priority Tier:** not supported (as Claude Opus 5).
  **Priority Tier：**不支持（与 Claude Opus 5 相同）。
- **Fast mode:** research preview on the Claude API only (not Bedrock, Claude Platform on AWS, Google Cloud, or Foundry), `speed: "fast"` under beta `fast-mode-2026-02-01`, at **$8 / $40 per MTok** (2x the standard price, the same multiple as Claude Opus 5's $10 / $50).
  **快速模式：**仅在 Claude API 上的研究预览（不在 Bedrock、Claude Platform on AWS、Google Cloud 或 Foundry），beta `fast-mode-2026-02-01` 下 `speed: "fast"`，**每 MTok $8 / $40**（标准价的 2 倍，与 Claude Opus 5 的 $10 / $50 同一倍数）。
- **Data retention / ZDR:** nothing new is documented - treat Claude Opus 5.5 as Claude Opus 5 here.
  **数据保留 / ZDR：**没有新的文档说明——此处把 Claude Opus 5.5 当作 Claude Opus 5 对待。
- **SDK constants:** `Model.ClaudeOpus5_5` (C#), `anthropic.ModelClaudeOpus5_5` (Go), `Model.CLAUDE_OPUS_5_5` (Java, PHP), `Anthropic::Model::CLAUDE_OPUS_5_5` (Ruby) - published with each SDK's launch release; the bare string `"claude-opus-5-5"` works everywhere before then.
  **SDK 常量：**`Model.ClaudeOpus5_5`（C#）、`anthropic.ModelClaudeOpus5_5`（Go）、`Model.CLAUDE_OPUS_5_5`（Java、PHP）、`Anthropic::Model::CLAUDE_OPUS_5_5`（Ruby）——随各 SDK 的发布版本一起提供；在此之前，裸字符串 `"claude-opus-5-5"` 在所有地方都可用。

### Capability improvements versus Claude Opus 5 / 相对 Claude Opus 5 的能力提升

**Cheaper per solved task, not just per token.** On many coding, analysis, and vision tasks, Claude Opus 5.5 at its default effort matched or beat Claude Opus 5 while using fewer tokens, and its price per token is 20% lower than Claude Opus 5's (60% lower for cache reads) - so expect the cost per solved task to be significantly lower for most tasks, and re-baseline `shared/cost-optimization.md`'s Claude Opus 5 figures rather than scaling them by the list price alone.

**每个解出任务更便宜，而不只是每 token。**在许多编码、分析与视觉任务上，Claude Opus 5.5 在默认努力度下匹配或超过 Claude Opus 5，同时使用更少 token，且其每 token 价格比 Claude Opus 5 低 20%（缓存读取低 60%）——因此预期大多数任务每个解出任务的成本显著更低，请对 `shared/cost-optimization.md` 的 Claude Opus 5 数字重新设定基准，而不是仅按牌价缩放。

- **Agentic coding and code review:** the largest measured gains are on multistep work in a real codebase (carrying a change through a large repository until its tests pass) - in Anthropic's testing, at its default `medium` effort it matched or beat Claude Opus 5's `high`-effort results on such tasks, in fewer steps and with about half the tokens - and on code review, where it catches more bugs with fewer false alarms. It explains its changes in plain language, which makes its work easier to review and trust. It tends to finish the same task with fewer tokens.
  **代理式编码与代码评审：**实测收益最大的是真实代码库中的多步工作（把一个改动带过大型仓库直到测试通过）——在 Anthropic 的测试中，默认 `medium` 努力度下它在这类任务上匹配或超过 `high` 努力度的 Claude Opus 5，步数更少、token 约为一半——代码评审方面，它以更少误报捕获更多 bug。它用平实语言解释自己的改动，使其工作更易评审与信任。它倾向用更少 token 完成同样的任务。
- **Knowledge work:** a more reliable analyst - much less likely than Claude Opus 5 to state a figure or cite a source the inputs don't support (citations pointed at the right source much more often in one customer's measurement); at `medium` it produced better long analytical deliverables than Claude Opus 5 at `high` with roughly 40% fewer output tokens; better at building and auditing financial models; more detail-oriented on large inputs (a date in a long thread that falls on the wrong weekday, a chart in a deck that doesn't match the figures) **without more false-positive flags**.
  **知识工作：**更可靠的分析师——比 Claude Opus 5 更少说出输入不支持的数据或引用（在一家客户的测量中，引用指向正确来源的频率大幅提高）；在 `medium` 下它产出的长篇分析交付物优于 `high` 下的 Claude Opus 5，输出 token 少约 40%；更擅长搭建与审计财务模型；对大输入更注重细节（长线程中落在错误星期几的日期、幻灯片中与数字不符的图表）**而不会带来更多误报标记**。
- **Communication and writing:** clearer prose, most noticeably in how it reports on agentic work - its updates while it works and its summary when it finishes say plainly what it did, what it found, and what it needs from you, with less jargon and fewer stock phrases.
  **沟通与写作：**行文更清晰，最明显的是它对代理工作的汇报——工作时的更新与完成时的总结直白说明它做了什么、发现了什么、需要你什么，行话与套话更少。
- **Charts, diagrams, screenshots, and computer use:** reads visual material more accurately at every effort level without extra tooling - in Anthropic's testing, even at `low` it read charts more accurately than Claude Opus 5 at its highest effort, at roughly a tenth of the output tokens per chart (Claude Opus 5 read charts well only by running code to crop, zoom, and measure) - values on dense charts, and meaning that depends on position rather than text (which boxes an arrow connects, what changed between two diagram versions, exactly when a meeting starts and ends in a calendar screenshot). Also more reliable at multistep computer use from screenshots: at its default effort it matched the success rate Claude Opus 5 reached only at a much higher effort setting, in fewer steps and with roughly 40% fewer tokens.
  **图表、示意图、截图与计算机使用：**在每个努力度级别上都能在无额外工具的情况下更准确地读取视觉材料——在 Anthropic 的测试中，即便在 `low` 下，它读图的准确度也超过最高努力度下的 Claude Opus 5，每张图输出 token 约为十分之一（Claude Opus 5 只能靠运行代码裁剪、缩放与测量才能读好图表）——密集图表上的数值，以及取决于位置而非文字的含义（箭头连接哪些框、两个版本的示意图之间改了什么、日历截图中会议的确切起止时间）。在基于截图的多步计算机使用上也更可靠：默认努力度下它达到了 Claude Opus 5 只在更高得多的努力度设置下才达到的成功率，步数更少、token 少约 40%。

### Behavioral shifts (prompt-tunable) / 行为变化（可通过提示词调整）

- **Re-evaluate Claude Opus 5-specific instructions.** Instructions tuned for Claude Opus 5's behavior (the verbosity, over-verification, and scope prompts under § Behavioral shifts of § Migrating to Claude Opus 5) may no longer be needed - keep them as the starting point, then re-test each on your own evals rather than carrying them over untouched.
  **重新评估 Claude Opus 5 专属指令。**为 Claude Opus 5 行为调校的指令（§ Migrating to Claude Opus 5 中 § Behavioral shifts 下的冗长、过度验证与范围提示词）可能不再需要——保留它们作为起点，然后在自有评测上逐条重测，而不是原样照搬。
- **Progress updates:** covered above - render the `thinking` blocks, and ask in the system prompt for the cadence you want.
  **进度更新：**上文已覆盖——渲染 `thinking` 块，并在系统提示词中说明你想要的节奏。
- **Frontend design defaults:** asked for frontend work without design direction, it falls back on a few default styles, and a general instruction such as "avoid a generic AI look" mostly swaps one default for another. **It responds well to instructions that name specific patterns to avoid.** Work iteratively - look at which styles the first result used instead, and extend the list:
  **前端设计默认值：**在没有设计方向的情况下要求前端工作，它会退回几种默认样式，而"避免千篇一律的 AI 风格"这类笼统指令大多只是把一种默认换成另一种。**它对点名要避免的具体模式的指令响应良好。**请迭代进行——看第一次结果改用了哪些样式，再扩充清单：
  > *"Output a vanilla HTML/CSS personal website with placeholder data. Do not use a cream or off-white background, italic accent words in headlines, numbered "01/02/03" section labels, monospace labels, or pill-shaped buttons."*
  > *"输出一个原生 HTML/CSS 个人网站，使用占位数据。不要使用奶油色或米白色背景、标题中的斜体强调词、"01/02/03"编号小节标签、等宽字体标签，或药丸形按钮。"*
- **Ingesting complex visual inputs:** it reads charts, diagrams, and screenshots considerably more precisely out of the box, so **harness scaffolding built for visual inputs on earlier models may no longer be needed - re-test it.** For the densest inputs, two things still add accuracy: higher-resolution images (most of all for technical drawings), and image-processing tools - run it as an agent with a container holding the raw images and libraries such as PIL and OpenCV so it can crop, zoom, measure, and verify; if a container is too much overhead, a cropping tool alone still helps. It uses these tools more effectively at higher effort levels; without tools, raising effort improves its reading of technical drawings but does little for charts.
  **摄取复杂视觉输入：**它开箱即能明显更精确地读取图表、示意图与截图，因此为早期模型的视觉输入搭建的 harness 脚手架可能不再需要——请重测。对最密集的输入，仍有两件事能提高准确度：更高分辨率的图片（对技术图纸最有用），以及图像处理工具——把它当作代理来运行，配一个装有原始图片和 PIL、OpenCV 等库的容器，让它能裁剪、缩放、测量与验证；如果容器开销太大，仅一个裁剪工具仍有帮助。努力度级别越高，它对这些工具的使用越有效；没有工具时，提高努力度对技术图纸的读取有改善，对图表帮助不大。
- **Long turns:** at `xhigh` / `max`, turns run longer than on Claude Opus 5 - plan timeouts, streaming, and progress UX accordingly, and lower effort before prompting for brevity.
  **长轮次：**在 `xhigh` / `max` 下，轮次比 Claude Opus 5 更长——相应规划超时、流式与进度 UX，并先用降低努力度的方式代替提示词要求简洁。

### Claude Opus 5.5 Migration Checklist / Claude Opus 5.5 迁移检查清单

- [ ] **[BLOCKS]** Model ID -> `claude-opus-5-5` (Bedrock: `anthropic.claude-opus-5-5`). Coming from Opus 4.8 or older, the Claude Opus 5 checklist first - except that disabling thinking is not an option.
  **[BLOCKS]** 模型 ID -> `claude-opus-5-5`（Bedrock：`anthropic.claude-opus-5-5`）。来自 Opus 4.8 或更早的，先做 Claude Opus 5 检查清单——除了关闭思维已不可选。
- [ ] **[BLOCKS]** Remove `thinking: {type: "disabled"}` and `{type: "enabled", budget_tokens}` on every route - both 400 at every effort level. Choose an effort level instead; size `max_tokens` for thinking plus the reply; read content blocks by `type`; pass `thinking` blocks back unmodified.
  **[BLOCKS]** 在每条路由上移除 `thinking: {type: "disabled"}` 与 `{type: "enabled", budget_tokens}`——两者在每个努力度级别都返回 400。改为选择努力度级别；为思考加回复设置 `max_tokens`；按 `type` 读取内容块；原样回传 `thinking` 块。
- [ ] **[BLOCKS]** Replace `tool_choice` `any` / `tool` with `auto` plus `strict: true` (steering in the prompt, and a check that the call happened) or structured outputs - on `count_tokens` and Batches too.
  **[BLOCKS]** 把 `tool_choice` 的 `any` / `tool` 换成 `auto` 加 `strict: true`（在提示词中引导，并检查调用是否发生）或结构化输出——`count_tokens` 与 Batches 上同样。
- [ ] **[BLOCKS]** Computer use on the Claude API and Google Cloud: declare `{"type": "computer_toolset_20260801"}` (no beta header, no `name` / display size) instead of `computer_20251124` (Amazon Bedrock still accepts `computer_20251124`), and update the agent loop for member `tool_use` blocks (action = block `name`), batch actions, and `toolset_name` on every result; confirm the toolset is offered on your platform. Test on Claude Opus 5 first.
  **[BLOCKS]** Claude API 与 Google Cloud 上的计算机使用：声明 `{"type": "computer_toolset_20260801"}`（无 beta 请求头、无 `name` / 显示尺寸）代替 `computer_20251124`（Amazon Bedrock 仍接受 `computer_20251124`），并为成员 `tool_use` 块（动作 = 块 `name`）、批量动作与每个结果上的 `toolset_name` 更新代理循环；确认工具集在你的平台上可用。先在 Claude Opus 5 上测试。
- [ ] **[BLOCKS]** If the harness builds `messages` itself: run the preserved-thinking three-step check (§ Migrating to Claude Fable 5.1 from Claude Fable 5) - accounts created on or after 2026-08-31 are enforced by default on every platform; set `prefix_mismatch_behavior` explicitly under `thinking-binding-controls-2026-08-01` and replace every history edit with its append-only form. Declare from the first request any tool the session may need later.
  **[BLOCKS]** 如果 harness 自行构建 `messages`：运行保留思维三步检查（§ Migrating to Claude Fable 5.1 from Claude Fable 5）——创建于 2026-08-31 或之后的账户在所有平台上默认被强制执行；在 `thinking-binding-controls-2026-08-01` 下显式设置 `prefix_mismatch_behavior`，并把每一处历史编辑替换为只追加形式。会话稍后可能需要的工具，从第一个请求起就声明。
- [ ] **[BLOCKS]** Handle `stop_reason: "refusal"` before reading `content` (new `bio` and `reasoning_extraction` categories) and ship a fallback opt-in; `reasoning_extraction` is not retried on a fallback.
  **[BLOCKS]** 在读取 `content` 之前处理 `stop_reason: "refusal"`（新增 `bio` 与 `reasoning_extraction` 类别），并上线回退选择加入；`reasoning_extraction` 不在回退上重试。
- [ ] **[TUNE]** Set `effort` explicitly - the default is `medium`, one level below Claude Opus 5's `high` - and re-run the sweep including `low` / `medium`; lower effort before adding "think less" prompts; reserve `xhigh` / `max` for measured gains; use per-message effort (beta) to vary it without a cache reset.
  **[TUNE]** 显式设置 `effort`——默认值是 `medium`，比 Claude Opus 5 的 `high` 低一级——并重做包括 `low` / `medium` 在内的扫描；在添加"少思考"提示词之前先降低努力度；把 `xhigh` / `max` 留给实测收益；用每消息努力度（beta）在不重置缓存的情况下调整。
- [ ] **[TUNE]** If the UI showed text between tool calls: `display: "updates"` (beta) or `"summarized"`, render non-empty `thinking` blocks; give the model a send-message tool (declared at session start) for verbatim mid-turn content; ask in the system prompt for the update cadence you want.
  **[TUNE]** 如果 UI 曾显示工具调用之间的文本：设置 `display: "updates"`（beta）或 `"summarized"`，渲染非空 `thinking` 块；给模型一个发消息工具（会话开始时声明）用于轮中途的逐字内容；在系统提示词中说明你想要的更新节奏。
- [ ] **[TUNE]** If a router or fallback can move the conversation to another model, expect it to run without Claude Opus 5.5's thinking blocks (only Claude Fable 5.1 / Claude Mythos 5.1 on the Claude API keep them).
  **[TUNE]** 如果路由器或回退可能把对话移到另一个模型，预期它将在没有 Claude Opus 5.5 思维块的情况下运行（在 Claude API 上只有 Claude Fable 5.1 / Claude Mythos 5.1 能保留它们）。
- [ ] **[TUNE]** If thinking was disabled on Claude Opus 5: start at `low`, remove reasoning-in-the-response instructions, re-test the thinking-disabled mitigation instruction, delete any "don't think" rule.
  **[TUNE]** 如果在 Claude Opus 5 上关闭了思维：从 `low` 开始，移除"把推理写进响应"类指令，重测思维关闭缓解指令，删除任何"不要思考"规则。
- [ ] **[TUNE]** Re-test visual-input scaffolding (may be unnecessary now); for frontend work, name the specific default patterns to avoid rather than asking for "no generic look"; re-evaluate Claude Opus 5-specific verbosity / verification / scope instructions.
  **[TUNE]** 重测视觉输入脚手架（现在可能不再需要）；前端工作要点名要避免的具体默认模式，而不是要求"不要千篇一律"；重新评估 Claude Opus 5 专属的冗长 / 验证 / 范围指令。
- [ ] **[TUNE]** Re-baseline cost and latency at the chosen effort level: $4 / $20, cache reads $0.20 per MTok; separate rate-limit pool; no Priority Tier; fast mode is Claude API only at $8 / $40.
  **[TUNE]** 在选定的努力度级别重新校准成本与延迟基准：$4 / $20，缓存读取每 MTok $0.20；独立速率限制池；无 Priority Tier；快速模式仅 Claude API，$8 / $40。

---

## Migrating to Claude Sonnet 5.5 / 迁移到 Claude Sonnet 5.5

> **Model ID `claude-sonnet-5-5` is authoritative as written here.** When the user asks to migrate to Claude Sonnet 5.5, write `model="claude-sonnet-5-5"` exactly (no date suffix; `anthropic.claude-sonnet-5-5` on Amazon Bedrock). Do **not** WebFetch to verify - this guide is the source of truth for migration target IDs. The corresponding entry exists in `shared/models.md`.
> **模型 ID `claude-sonnet-5-5` 以此处所写为准。**当用户要求迁移到 Claude Sonnet 5.5 时，请精确写出 `model="claude-sonnet-5-5"`（无日期后缀；Amazon Bedrock 上为 `anthropic.claude-sonnet-5-5`）。**不要**用 WebFetch 去验证——本指南是迁移目标 ID 的事实来源。`shared/models.md` 中存在对应条目。

Claude Sonnet 5.5 succeeds Claude Sonnet 5 in the Sonnet line **at the same prices** - $2 / $10 per MTok input / output, 5-minute cache writes $2.50, 1-hour cache writes $4, cache reads $0.20, and Claude Sonnet 5's batch rates. Same tokenizer as Claude Sonnet 5 (token counts unchanged), 1M token context window, 128K max output (up to 300K on the Message Batches API with the `output-300k-2026-03-24` beta header). Available at launch on the Claude API (`claude-sonnet-5-5`), Amazon Bedrock (`anthropic.claude-sonnet-5-5`), Claude Platform on AWS, Google Cloud, and Microsoft Foundry (all three as `claude-sonnet-5-5`). On Foundry it is hosted on Azure only, with Global Standard deployments only, so the features Foundry doesn't offer when hosted on Azure are unavailable for it there - code execution, programmatic tool calling, Agent Skills, the Files API, and the newer web search and web fetch versions (`shared/platform-availability.md`). The SDK constants listed below may be missing from the SDK version a project pins; the bare string `"claude-sonnet-5-5"` works in every SDK (with the `anthropic.` prefix on Amazon Bedrock). Existing Claude Sonnet 5 prompts should perform well without changes; for the hardest long-horizon work, an Opus model is the better choice.

Claude Sonnet 5.5 在 Sonnet 产品线中接替 Claude Sonnet 5，**价格相同**——每 MTok 输入/输出 $2 / $10，5 分钟缓存写入 $2.50，1 小时缓存写入 $4，缓存读取 $0.20，批处理费率与 Claude Sonnet 5 相同。与 Claude Sonnet 5 相同的分词器（token 计数不变），1M token 上下文窗口，128K 最大输出（带 `output-300k-2026-03-24` beta 请求头时在 Message Batches API 上最高 300K）。发布时可在 Claude API（`claude-sonnet-5-5`）、Amazon Bedrock（`anthropic.claude-sonnet-5-5`）、Claude Platform on AWS、Google Cloud 与 Microsoft Foundry 上使用（后三者均为 `claude-sonnet-5-5`）。在 Foundry 上它仅托管于 Azure，且仅提供 Global Standard 部署，因此 Foundry 在 Azure 托管下不提供的特性对它也不可用——代码执行、程序化工具调用、Agent Skills、Files API，以及较新的 web 搜索与 web 抓取版本（`shared/platform-availability.md`）。下文列出的 SDK 常量在项目锁定的 SDK 版本中可能缺失；裸字符串 `"claude-sonnet-5-5"` 在每个 SDK 中都可用（Amazon Bedrock 上带 `anthropic.` 前缀）。现有 Claude Sonnet 5 提示词应当无需改动即有良好表现；对最难的长程工作，Opus 模型是更好的选择。

**Claude Sonnet 5.5 is the default Sonnet migration target.** It is layered on top of the Claude Sonnet 5 migration above - a caller coming from Sonnet 4.6 or earlier applies § Migrating to Claude Sonnet 5 first (and, from Sonnet 4.5 or earlier, the older sections it points to), **except these parts of it, which this section replaces:** its `thinking: {type: "disabled"}` route (breaking change 1 below); its Bedrock-only forced `tool_choice` with thinking disabled, both halves of which 400 here (breaking changes 1 and 2); its `computer_20251124` tool version, which 400s on the Claude API and Google Cloud (breaking change 4); and all of its effort advice - including the `high` default with `xhigh` for the hardest work, the level mapping against Sonnet 4.6, the effort suggestions in its behavioral notes, and the prompts to make the model think more or less - because the levels are recalibrated here (§ Choosing an effort level). A caller coming from Claude Haiku 4.5 applies the same sections, including the Sonnet 4.5-or-earlier changes (prefill, explicit effort, beta headers, `output_format`, parsing tool input with a JSON parser) but not the Sonnet 4-or-earlier ones, then replaces `claude-haiku-4-5-20251001` or its alias, re-baselines cost at the higher price per token, and reviews prompts that were too short to cache on Claude Haiku 4.5.

**Claude Sonnet 5.5 是默认的 Sonnet 迁移目标。**它叠加在上文 Claude Sonnet 5 迁移之上——来自 Sonnet 4.6 或更早的调用方先应用 § Migrating to Claude Sonnet 5（来自 Sonnet 4.5 或更早的，再应用其所指的更早章节），**但其中被本节取代的这些部分除外：**其 `thinking: {type: "disabled"}` 路由（下方破坏性变更 1）；其仅限 Bedrock 的"思维关闭下的强制 `tool_choice`"，其两部分在这里都返回 400（破坏性变更 1 与 2）；其 `computer_20251124` 工具版本，在 Claude API 与 Google Cloud 上返回 400（破坏性变更 4）；以及它的全部努力度建议——包括最难工作用 `xhigh`、默认 `high` 的设定、与 Sonnet 4.6 的级别对照、其行为说明中的努力度建议，以及让模型多思考或少思考的提示词——因为级别在这里被重新校准（§ Choosing an effort level）。来自 Claude Haiku 4.5 的调用方应用同样的章节，包括 Sonnet 4.5 或更早的变更（预填充、显式努力度、beta 请求头、`output_format`、用 JSON 解析器解析工具输入），但不包括 Sonnet 4 或更早的章节，然后替换 `claude-haiku-4-5-20251001` 或其别名，按更高的每 token 价格重新校准成本，并检查在 Claude Haiku 4.5 上因太短而无法缓存的提示词。

**What changes:** five breaking changes for code running on Claude Sonnet 5 (`disabled` thinking 400s - turn thinking off with `between_tools` instead; forced `tool_choice` 400s; thinking blocks are tied to the model and the conversation - "preserved thinking"; on the Claude API and Google Cloud the `computer_20251124` tool 400s - use the computer toolset; the advisor tool rejects Claude Opus 4.8, Claude Opus 4.7, and Claude Sonnet 5 advisors), one response-shape change that fails no request (text between tool calls comes back in `thinking` blocks), **recalibrated effort levels** (the default stays `high`, but a level no longer produces the same amount of thinking as on Claude Sonnet 5), and safety classifiers that decline in five categories. Breaking changes 2 and 3 are the same mechanisms Claude Fable 5.1 and Claude Opus 5.5 introduced - the shared mechanics are under § Migrating to Claude Fable 5.1 from Claude Fable 5, and this section gives the Claude Sonnet 5.5 specifics.

**变更内容：**对运行在 Claude Sonnet 5 上的代码有五项破坏性变更（`disabled` 思维返回 400——改用 `between_tools` 关闭思维；强制 `tool_choice` 返回 400；思维块与模型及对话绑定——"保留思维"；在 Claude API 与 Google Cloud 上 `computer_20251124` 工具返回 400——改用计算机工具集；advisor 工具拒绝 Claude Opus 4.8、Claude Opus 4.7 与 Claude Sonnet 5 的 advisor），一处不导致任何请求失败的响应形态变化（工具调用之间的文本以 `thinking` 块返回），**重新校准的努力度级别**（默认值仍为 `high`，但同一级别不再产生与 Claude Sonnet 5 上相同的思考量），以及在五个类别中拒答的安全分类器。破坏性变更 2 与 3 与 Claude Fable 5.1 和 Claude Opus 5.5 引入的是相同机制——共享机制见 § Migrating to Claude Fable 5.1 from Claude Fable 5，本节给出 Claude Sonnet 5.5 的具体细节。

### Breaking change 1: `disabled` thinking returns a 400 - turn thinking off with `between_tools` / 破坏性变更 1：`disabled` 思维返回 400——改用 `between_tools` 关闭思维

On Claude Sonnet 5, thinking is on by default and `thinking: {type: "disabled"}` turns it off. On Claude Sonnet 5.5, `{"type": "disabled"}` returns a 400 `invalid_request_error`:

在 Claude Sonnet 5 上，思维默认开启，`thinking: {type: "disabled"}` 可将其关闭。在 Claude Sonnet 5.5 上，`{"type": "disabled"}` 返回 400 `invalid_request_error`：

```text
"thinking.type.disabled" is not supported for this model. Use "thinking.type.between_tools" for the lowest thinking setting, or "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

`thinking: {"type": "between_tools"}` is the lowest thinking setting on this model: the model does no extended thinking, and the short progress updates it writes between tool calls come back as `thinking` blocks with their summary text. It needs no beta header and works on every platform that offers the model. Its limits, each a 400 `invalid_request_error` when crossed:

`thinking: {"type": "between_tools"}` 是本模型上最低的思考设置：模型不做扩展思考，它写在工具调用之间的简短进度更新以 `thinking` 块连同其摘要文本返回。它不需要 beta 请求头，在提供该模型的每个平台上都可用。其限制（超出即返回 400 `invalid_request_error`）：

- **Effort `high` or below only.** At `xhigh` or `max`, use adaptive thinking (omit `thinking` or send `{"type": "adaptive"}`). The error reads `output_config.effort 'xhigh' is not supported when thinking is disabled on this model. Use effort 'high' or below, or enable thinking.` - "enable thinking" in this and the next error means adaptive thinking; `{"type": "enabled"}` is itself a 400.
  **仅限 `high` 或以下的努力度。**在 `xhigh` 或 `max` 下，使用自适应思维（省略 `thinking` 或发送 `{"type": "adaptive"}`）。错误信息为 `output_config.effort 'xhigh' is not supported when thinking is disabled on this model. Use effort 'high' or below, or enable thinking.`——此错误与下一个错误中的"启用思维"指的是自适应思维；`{"type": "enabled"}` 本身就是 400。
- **No other field inside `thinking`.** `display`, `budget_tokens`, or `block_binding` sent with `between_tools` is rejected, and manual budgets (`{"type": "enabled", "budget_tokens": N}`) return a 400 as well.
  **`thinking` 内不能有其他字段。**与 `between_tools` 一起发送的 `display`、`budget_tokens` 或 `block_binding` 会被拒绝，手动预算（`{"type": "enabled", "budget_tokens": N}`）同样返回 400。
- **Effort can't change mid-conversation.** A per-message `output_config.effort` that differs from the level in effect is rejected (`messages.N: output_config.effort 'low' differs from the 'high' in effect before it; ...`). To vary effort per turn, use adaptive thinking.
  **努力度不能在对话中途改变。**与现行级别不同的每消息 `output_config.effort` 会被拒绝（`messages.N: output_config.effort 'low' differs from the 'high' in effect before it; ...`）。要逐轮变化努力度，请用自适应思维。
- **Claude Sonnet 5.5 only.** Any other model rejects it: `"thinking.type.between_tools" is not supported for this model.` - so client-side code that re-sends the same body to another model (a router or a retry) must drop the field first.
  **仅限 Claude Sonnet 5.5。**任何其他模型都拒绝它：`"thinking.type.between_tools" is not supported for this model.`——因此把同一请求体重发给另一模型的客户端代码（路由器或重试）必须先去掉该字段。

Migrate a route that disables thinking in this order:

按以下顺序迁移关闭思维的路由：

1. **Try adaptive thinking at `low` effort first.** At `low` the model keeps its thinking short and skips it on most simple requests. Measure time to first token at the median and the 95th percentile on the caller's own traffic, and compare quality.
   **先尝试 `low` 努力度下的自适应思维。**在 `low` 下模型保持思考简短，并且在大多数简单请求上跳过思考。在调用方自己的流量上测量中位数与 95 分位的首 token 时间，并比较质量。
2. **Otherwise send `between_tools` at `high` effort or below.** Remove any instruction that tells the model not to think - such instructions make it more likely to write internal XML tags in its visible output.
   **否则在 `high` 或以下努力度发送 `between_tools`。**移除任何告诉模型不要思考的指令——这类指令使它更可能在可见输出中写内部 XML 标签。
3. **Read the response by block `type`, not position.** With adaptive thinking, a response can begin with a `thinking` block whose `thinking` field is empty under the default `display: "omitted"`.
   **按块的 `type` 而非位置读取响应。**使用自适应思维时，响应可能以 `thinking` 块开头，其在默认 `display: "omitted"` 下 `thinking` 字段为空。
4. **Pass `thinking` blocks back unchanged** with the rest of the assistant turn, including the progress-update blocks `between_tools` returns - a block sent back gives the model the full note it wrote, not the summary.
   **把 `thinking` 块原样回传**，与 assistant 轮的其余部分一起，包括 `between_tools` 返回的进度更新块——回传的块让模型得到它所写说明的完整文本，而非摘要。
5. **Size `max_tokens` for thinking as well as the reply.** Thinking counts toward `max_tokens` even when its text isn't returned; for long agentic coding turns, 64,000 is a reasonable starting point.
   **为思考以及回复设置 `max_tokens`。**即使思考文本不返回，思考也计入 `max_tokens`；对长代理式编码轮，64,000 是合理的起点。

```python
# Before - accepted on Claude Sonnet 5, 400 on Claude Sonnet 5.5
client.messages.create(
    model="claude-sonnet-5",
    max_tokens=16000,
    thinking={"type": "disabled"},
    messages=[{"role": "user", "content": "..."}],
)

# After, preferred - adaptive thinking (the default) at low effort; measure latency and quality
client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=16000,
    output_config={"effort": "low"},
    messages=[{"role": "user", "content": "..."}],
)

# After, when the route must stay thinking-off - the lowest thinking setting, at high effort or below
client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=16000,
    thinking={"type": "between_tools"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "..."}],
)
```

In Python, TypeScript, PHP, and Ruby, write `between_tools` as a plain value in the `thinking` object (Ruby: `type: :between_tools`). In the typed SDKs, send the `thinking` object as a raw override until their types include `between_tools`: C# `Thinking = new ThinkingConfigParam(JsonSerializer.SerializeToElement(new { type = "between_tools" }))`; Go `params.SetExtraFields(map[string]any{"thinking": map[string]any{"type": "between_tools"}})` on the `MessageNewParams` value; Java `.putAdditionalBodyProperty("thinking", JsonValue.from(Map.of("type", "between_tools")))`.

在 Python、TypeScript、PHP 与 Ruby 中，把 `between_tools` 作为 `thinking` 对象中的普通值书写（Ruby：`type: :between_tools`）。在类型化 SDK 中，在其类型包含 `between_tools` 之前，把 `thinking` 对象作为原始覆盖发送：C# `Thinking = new ThinkingConfigParam(JsonSerializer.SerializeToElement(new { type = "between_tools" }))`；Go 在 `MessageNewParams` 值上 `params.SetExtraFields(map[string]any{"thinking": map[string]any{"type": "between_tools"}})`；Java `.putAdditionalBodyProperty("thinking", JsonValue.from(Map.of("type", "between_tools")))`。

### Breaking change 2: forced tool use is rejected / 破坏性变更 2：强制工具使用被拒绝

As on Claude Fable 5.1 and Claude Opus 5.5: `tool_choice: {"type": "any"}` and `{"type": "tool", "name": "..."}` return a 400 `invalid_request_error` (`tool_choice: type "tool" and "any" are not supported for this model.`), including on the token-counting endpoint, where Claude Sonnet 5 accepts both. `{"type": "auto"}` (the default) and `{"type": "none"}` are unchanged. Migrate by intent - the patterns are under § Breaking change 1: forced tool use is rejected in § Migrating to Claude Fable 5.1 from Claude Fable 5:

与 Claude Fable 5.1 及 Claude Opus 5.5 相同：`tool_choice: {"type": "any"}` 与 `{"type": "tool", "name": "..."}` 返回 400 `invalid_request_error`（`tool_choice: type "tool" and "any" are not supported for this model.`），包括 token 计数端点，而 Claude Sonnet 5 两者都接受。`{"type": "auto"}`（默认值）与 `{"type": "none"}` 不变。按迁移意图处理——模式见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中 § Breaking change 1: forced tool use is rejected：

- **Steering toward a tool:** `tool_choice: {"type": "auto"}` plus a prompt that says when the tool applies, with `strict: true` on the tool definition for schema-valid arguments. Because `auto` does not guarantee a call, check that one was made and retry if it wasn't.
  **引导模型使用某个工具：**`tool_choice: {"type": "auto"}` 加说明该工具何时适用的提示词，工具定义上设 `strict: true` 保证参数符合 schema。由于 `auto` 不保证一定调用，要检查调用是否发生，未发生则重试。
- **Extracting structured data:** if the forced call existed only to get JSON back, use structured outputs (`output_config.format`).
  **提取结构化数据：**如果强制调用只是为了拿回 JSON，使用结构化输出（`output_config.format`）。

### Breaking change 3: thinking blocks are tied to the model and the conversation / 破坏性变更 3：思维块与模型及对话绑定

- **Model binding - who reads whose blocks.** Claude Sonnet 5.5 reads thinking blocks from Claude Sonnet 5, Claude Opus 4.8, Claude Haiku 4.5, and earlier models - a conversation that moves from Claude Sonnet 5 onto `claude-sonnet-5-5` keeps its reasoning - but **not** from Claude Opus 5, Claude Opus 5.5, or any Claude Fable or Claude Mythos model. **No other model reads Claude Sonnet 5.5 blocks**, so a router switch, a retry on another model, or a refusal fallback runs the turns after the switch without its reasoning. The API drops what the target can't read before the model sees it: the request succeeds, dropped blocks aren't billed, and with the `thinking-binding-controls-2026-08-01` beta header the drop is reported in a top-level `input_transformations` array. Keep passing blocks back unchanged; don't strip them yourself.
  **模型绑定——谁读谁的块。**Claude Sonnet 5.5 读取来自 Claude Sonnet 5、Claude Opus 4.8、Claude Haiku 4.5 与更早模型的思维块——从 Claude Sonnet 5 切换到 `claude-sonnet-5-5` 的对话保留其推理——但**不**读取 Claude Opus 5、Claude Opus 5.5 或任何 Claude Fable / Claude Mythos 模型的块。**没有其他模型读取 Claude Sonnet 5.5 的块**，因此路由切换、在另一模型上重试或拒答回退，会让切换之后的轮次在没有其推理的情况下运行。API 会在模型看到之前丢弃目标读取不了的部分：请求成功，被丢弃的块不计费，带 `thinking-binding-controls-2026-08-01` beta 请求头时，丢弃会报告在顶层 `input_transformations` 数组中。请继续原样回传块；不要自行剥离。
- **Conversation binding - the history-editing check.** The API checks that the `system` prompt, the `tools`, and every earlier message are unchanged since a block was produced. It is enforced by default for accounts created on or after August 31, 2026, 00:00 UTC, on the Claude API and Amazon Bedrock (Google Cloud is not confirmed - check the preserved thinking docs before promising either way): on those accounts a request that replays a block after such an edit is a 400. To drop the affected blocks instead, send the `thinking-binding-controls-2026-08-01` beta header and set `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`; on older accounts, setting that field to either value opts the request in. **`block_binding` works only with `thinking: {"type": "adaptive"}`** - with `between_tools`, keep the history append-only, or strip the thinking blocks from the edited turn on. The append-only replacements (mid-conversation `role: "system"` messages, which Claude Sonnet 5.5 supports and Claude Sonnet 5 doesn't; `tool_addition` / `tool_removal`; turn-scoped reminders; server-side compaction or context editing) and the three-step check are in § Migrating to Claude Fable 5.1 from Claude Fable 5. With threshold compaction, thinking blocks from before a `compaction` block aren't carried forward, so the summary is all the model has of that earlier work - if you write your own `instructions`, say what the summary must retain - and remove the `thinking` and `redacted_thinking` blocks from any assistant turn you re-insert after a compaction block (or set `drop_block`). On-demand compaction (beta `compact-2026-09-04`) can keep the kept turns' blocks valid after you replace the summarized turns with the returned `compaction` block.
  **对话绑定——历史编辑检查。**API 检查 `system` 提示词、`tools` 与每条更早的消息自块产生以来未变。它对创建于 2026 年 8 月 31 日 00:00 UTC 或之后的账户在 Claude API 与 Amazon Bedrock 上默认强制执行（Google Cloud 未确认——承诺之前请查阅保留思维文档）：在这些账户上，在这类编辑之后重放块的请求会得到 400。要改为丢弃受影响的块，发送 `thinking-binding-controls-2026-08-01` beta 请求头并设置 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`；在更早的账户上，把该字段设为任一取值即让请求选择加入。**`block_binding` 只能与 `thinking: {"type": "adaptive"}` 一起使用**——用 `between_tools` 时，请保持历史只追加，或从被编辑的轮次起剥离思维块。只追加替代形式（会话中途 `role: "system"` 消息，Claude Sonnet 5.5 支持而 Claude Sonnet 5 不支持；`tool_addition` / `tool_removal`；轮次作用域提醒；服务端压缩或上下文编辑）与三步检查见 § Migrating to Claude Fable 5.1 from Claude Fable 5。使用阈值压缩时，`compaction` 块之前的思维块不会向前携带，因此摘要是模型对那段较早工作的全部记忆——如果你自己写 `instructions`，说明摘要必须保留什么——并且从你在压缩块之后重新插入的任何 assistant 轮中移除 `thinking` 与 `redacted_thinking` 块（或设置 `drop_block`）。按需压缩（beta `compact-2026-09-04`）可以在你用返回的 `compaction` 块替换被摘要轮次之后，保持被保留轮次块的有效性。
- **Account binding.** On Amazon Bedrock and Google Cloud at launch, a block Claude Sonnet 5.5 produced works only in the account that produced it, or in an account linked to it; another account's blocks are dropped and the request succeeds (on Google Cloud, with the `thinking-binding-controls-2026-08-01` header, each drop is listed in `input_transformations` with `reason: "organization_binding_mismatch"`). Blocks from earlier models aren't affected.
  **账户绑定。**在发布时的 Amazon Bedrock 与 Google Cloud 上，Claude Sonnet 5.5 产生的块只在产生它的账户或与之关联的账户中有效；其他账户的块会被丢弃且请求成功（在 Google Cloud 上，带 `thinking-binding-controls-2026-08-01` 请求头时，每次丢弃会以 `reason: "organization_binding_mismatch"` 列入 `input_transformations`）。更早模型的块不受影响。

### Breaking change 4: computer use needs the toolset on the Claude API and Google Cloud / 破坏性变更 4：在 Claude API 与 Google Cloud 上计算机使用需要工具集

On the Claude API and Google Cloud, Claude Sonnet 5.5 accepts computer use only as the `computer_toolset_20260801` toolset; `computer_20251124` returns a 400 (on the Claude API the message begins `'claude-sonnet-5-5' does not support tool types: computer_20251124.`). On Amazon Bedrock it still accepts `computer_20251124`. No platform accepts `computer_20250124`.

在 Claude API 与 Google Cloud 上，Claude Sonnet 5.5 只接受 `computer_toolset_20260801` 工具集形式的计算机使用；`computer_20251124` 返回 400（在 Claude API 上消息以 `'claude-sonnet-5-5' does not support tool types: computer_20251124.` 开头）。在 Amazon Bedrock 上它仍接受 `computer_20251124`。没有任何平台接受 `computer_20250124`。

| Version sent today | Starting models that send it | Send on the Claude API and Google Cloud | Send on Amazon Bedrock |
|---|---|---|---|
| `computer_20251124` | Claude Sonnet 5, Sonnet 4.6 | `computer_toolset_20260801` | `computer_20251124` |
| `computer_20250124` | Sonnet 4.5, Haiku 4.5, Sonnet 4 | `computer_toolset_20260801` | `computer_20251124` |

| 当前发送的版本 | 发送它的起始模型 | 在 Claude API 与 Google Cloud 上发送 | 在 Amazon Bedrock 上发送 |
|---|---|---|---|
| `computer_20251124` | Claude Sonnet 5、Sonnet 4.6 | `computer_toolset_20260801` | `computer_20251124` |
| `computer_20250124` | Sonnet 4.5、Haiku 4.5、Sonnet 4 | `computer_toolset_20260801` | `computer_20251124` |

The toolset's request shape and agent-loop changes (no beta header, no `name` or display size, the action is the member `tool_use` block's `name`, several calls per turn, `"toolset_name": "computer"` echoed on every result) are in `shared/tool-use-concepts.md` § Computer Use and § Migrating to Claude Opus 5.5 -> Breaking change 4. Code that already sends the toolset needs no change. Two more things to check in the agent loop: don't prune old screenshots on the client - removing an earlier screenshot invalidates every later thinking block (breaking change 3), so resize screenshots to 2000 px or less per side and use server-side tool result clearing instead, or, with adaptive thinking, keep `prefix_mismatch_behavior: "drop_block"` set from the first prune on; and replace the `fine-grained-tool-streaming-2025-05-14` header with `eager_input_streaming: true` on each tool that needs it - the header returns a 400 alongside a computer use or browser use toolset entry.

工具集的请求形态与代理循环改动（无 beta 请求头、无 `name` 或显示尺寸、动作是成员 `tool_use` 块的 `name`、每轮多次调用、每个结果都回显 `"toolset_name": "computer"`）见 `shared/tool-use-concepts.md` § Computer Use 与 § Migrating to Claude Opus 5.5 -> Breaking change 4。已经在发送工具集的代码无需改动。代理循环中还要检查两件事：不要在客户端裁剪旧截图——移除较早的截图会使之后所有思维块失效（破坏性变更 3），因此把截图每边缩放到 2000 px 或更小，并改用服务端工具结果清除；或者在使用自适应思维时，从第一次裁剪起就保持设置 `prefix_mismatch_behavior: "drop_block"`；以及把 `fine-grained-tool-streaming-2025-05-14` 请求头替换为需要它的每个工具上的 `eager_input_streaming: true`——该请求头与计算机使用或浏览器使用工具集条目同时出现会返回 400。

### Breaking change 5: the advisor tool accepts fewer advisors / 破坏性变更 5：advisor 工具接受的 advisor 更少

With the advisor tool (beta), a Claude Sonnet 5.5 executor needs one of these advisors: Claude Opus 5, Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, or Claude Mythos 5.1. Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, Claude Sonnet 5, and Sonnet 4.6 advisors return a 400. Every accepted advisor returns its advice encrypted, as an `advisor_redacted_result` block, so the advice text isn't readable in the response. Because the executor rejects forced `tool_choice`, nudge a consult from the prompt rather than forcing the `advisor` tool.

使用 advisor 工具（beta）时，Claude Sonnet 5.5 执行器需要以下 advisor 之一：Claude Opus 5、Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5 或 Claude Mythos 5.1。Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5 与 Sonnet 4.6 的 advisor 返回 400。每个被接受的 advisor 都以加密的 `advisor_redacted_result` 块返回其建议，因此建议文本在响应中不可读。由于执行器拒绝强制的 `tool_choice`，请从提示词中引导咨询，而不是强制 `advisor` 工具。

### Text between tool calls comes back in thinking blocks / 工具调用之间的文本以思维块返回

On Claude Sonnet 5, text the model writes between tool calls comes back as `text` blocks. On Claude Sonnet 5.5, notes longer than a sentence or two come back as **progress-update `thinking` blocks**, empty under the default `display: "omitted"`; shorter remarks stay `text`. No request fails, but a client that renders only `text` blocks goes quiet between tool calls. With adaptive thinking, set `thinking.display: "updates"` (beta `thinking-display-updates-2026-08-18`) to get the updates alone, or `"summarized"` to get them mixed with reasoning summaries; render each non-empty `thinking` block before the `tool_use` block that follows it, and pass the blocks back unchanged. With `between_tools`, the notes come back with their text and no `display` field is needed (or allowed).

在 Claude Sonnet 5 上，模型写在工具调用之间的文本以 `text` 块返回。在 Claude Sonnet 5.5 上，超过一两句的说明以**进度更新 `thinking` 块**返回，在默认 `display: "omitted"` 下为空；更短的说明仍是 `text`。没有请求会失败，但只渲染 `text` 块的客户端会在工具调用之间沉默。使用自适应思维时，设置 `thinking.display: "updates"`（beta `thinking-display-updates-2026-08-18`）只获取更新，或 `"summarized"` 让它们与推理摘要混在一起；把每个非空 `thinking` 块渲染在它后面的 `tool_use` 块之前，并原样回传这些块。使用 `between_tools` 时，说明连同其文本返回，不需要（也不允许）`display` 字段。

If the interface doesn't render `thinking` blocks and the model may need to show the user something word for word mid-turn (a code snippet, a question), give it a simple tool for sending the user a message, tell it to reserve that tool for such content, and declare it in the **first** request of the session so the `tools` list doesn't change later (breaking change 3). Remove older instructions such as "hold all findings for the final response"; if updates are then wanted at predictable points, add a line that says when user-facing text is wanted and what it should contain - the model follows it - for example:

如果界面不渲染 `thinking` 块，而模型可能需要在轮中途逐字展示某些内容（一段代码、一个问题），给它一个向用户发送消息的简单工具，告诉它把该工具留给这类内容，并在会话的**第一个**请求中声明它，使 `tools` 列表之后不再变化（破坏性变更 3）。移除"把所有发现留到最终回复"之类的旧指令；如果之后希望在可预期的时点获得更新，加一句说明何时需要面向用户的文本及其应包含的内容——模型会遵循——例如：

> *"Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own so a reader who only sees the last message has the full picture."*
> *"开始之前，用一行说明你即将做什么；工作过程中简短的更新有助于用户跟上进展。结尾给出一段能独立成立的简短回顾，让只看到最后一条消息的读者也能掌握全貌。"*

If long tool-calling turns still go quiet for too long, the harness can prompt an update: count consecutive tool-calling steps with no user-facing text or progress update, and after several in a row (for example five) append a one-turn reminder after the latest tool results as a turn-scoped system message (`clear_at: "next_user_message"`, beta `mid-conversation-system-clear-at-2026-08-21` - § Migrating to Claude Fable 5.1 from Claude Fable 5). If the turn stays quiet, stop after two or three reminders; text after every tool result can make the model suspect a prompt injection (§ Behavioral shifts -> Mid-turn user messages). Because the reminder is appended, never inserted and later deleted, the cache and preserved thinking stay intact. In Anthropic's testing at `high` effort, with a send-message tool available, it made the model update the user more often and shortened its longest silent stretches, with no measurable change in task quality:

如果长工具调用轮仍然沉默太久，harness 可以主动提示更新：统计没有面向用户文本或进度更新的连续工具调用步数，连续数次（例如五次）后，在最新工具结果之后追加一条单轮提醒，作为轮次作用域系统消息（`clear_at: "next_user_message"`，beta `mid-conversation-system-clear-at-2026-08-21`——见 § Migrating to Claude Fable 5.1 from Claude Fable 5）。如果轮次依然沉默，提醒两三次后停止；每个工具结果后都跟文本可能让模型怀疑是提示词注入（§ Behavioral shifts -> Mid-turn user messages）。由于提醒是追加的、绝不插入后删除，缓存与保留思维保持完好。在 Anthropic `high` 努力度下的测试中，在有发消息工具可用的情况下，它让模型更频繁地向用户更新并缩短了最长的沉默区间，任务质量没有可测的变化：

> *"The user hasn't heard from you in a while - say in a few words what you're doing, then continue."*
> *"用户有一阵子没听到你的消息了——用几句话说明你在做什么，然后继续。"*

### Choosing an effort level - recalibrated levels, default still `high` / 选择努力度级别——级别重新校准，默认值仍为 `high`

Claude Sonnet 5.5 supports `low`, `medium`, `high`, `xhigh`, and `max`; the Claude API default is `high`. The levels are **recalibrated**: a level doesn't produce the same amount of thinking as the same level on Claude Sonnet 5, so re-run the effort sweep against the caller's own evals rather than carrying the Claude Sonnet 5 setting over, and set `output_config.effort` explicitly.

Claude Sonnet 5.5 支持 `low`、`medium`、`high`、`xhigh` 与 `max`；Claude API 默认值为 `high`。级别被**重新校准**：同一级别产生的思考量与 Claude Sonnet 5 上的同一级别不同，因此要在调用方自己的评测上重做努力度扫描，而不是照搬 Claude Sonnet 5 的设置，并显式设置 `output_config.effort`。

- **Starting points:** `medium` for agentic coding and multistep tool use; `low` for chat, content generation, classification, extraction, and search. Reserve `xhigh` and `max` for work with a measured quality gain - at those levels thinking can't be turned off. At `low`, on long agentic tasks the model is more likely than at higher levels to stop and check in with the user before finishing, or to skip verifying a change (§ Behavioral shifts -> Verification on coding tasks).
  **起点：**代理式编码与多步工具使用用 `medium`；聊天、内容生成、分类、提取与搜索用 `low`。把 `xhigh` 与 `max` 留给有实测质量收益的工作——在那些级别上思维无法关闭。在 `low` 下，长代理任务上模型比更高级别更可能在完成前停下来向用户确认，或跳过对改动的验证（§ Behavioral shifts -> Verification on coding tasks）。
- **Judge cost per completed task, not per token.** In Anthropic's testing it finishes agentic coding and multistep tool-use work in far fewer model requests than Claude Sonnet 5, and generates output faster. On most agentic coding evals it scored higher at `medium` than Claude Sonnet 5 did at `high`, typically at under a fifth of the cost; on computer use at `high` it completed substantially more tasks than Claude Sonnet 5 at its highest effort, with under a third of the tokens.
  **按完成的任务而非每 token 判断成本。**在 Anthropic 的测试中，它完成代理式编码与多步工具使用工作所需的模型请求数远少于 Claude Sonnet 5，且输出生成更快。在大多数代理式编码评测上，它在 `medium` 下的得分高于 Claude Sonnet 5 在 `high` 下的得分，成本通常低于五分之一；在 `high` 的计算机使用上，它完成的任务明显多于最高努力度下的 Claude Sonnet 5，token 少于三分之一。
- **To get less thinking, lower the effort level.** From `medium` up, the model thinks briefly before almost every reply, even a greeting, which adds to the time before the first visible token, and a system-prompt request to think less has almost no effect at those levels. At `low`, it skips thinking on most simple requests.
  **想让思考更少，调低努力度级别。**从 `medium` 往上，模型在几乎每次回复前都会简短思考，哪怕只是打招呼，这增加了首个可见 token 之前的时间，而在这些级别上系统提示词里"少思考"的请求几乎无效。在 `low` 下，它在大多数简单请求上跳过思考。
- **Keep the cache warm when varying effort.** Changing the top-level `effort` between requests invalidates the prompt cache; a per-message effort change (beta `mid-conversation-output-config-2026-07-01`; request shape under § New API features of § Migrating to Claude Fable 5.1 from Claude Fable 5) keeps it - for example, run an interactive session at `low` and raise effort to `high` for a hard problem. Per-message effort needs adaptive thinking; with `between_tools` it is a 400.
  **变化努力度时保持缓存温热。**在请求之间修改顶层 `effort` 会使提示词缓存失效；每消息努力度变更（beta `mid-conversation-output-config-2026-07-01`；请求形态见 § Migrating to Claude Fable 5.1 from Claude Fable 5 中 § New API features）则保持——例如，交互会话用 `low`，遇到难题时把努力度升到 `high`。每消息努力度需要自适应思维；用 `between_tools` 时会返回 400。

### Safeguards and fallback / 安全防护与回退

Claude Sonnet 5.5 declines in more categories than Claude Sonnet 5. A decline is a normal HTTP 200 with `stop_reason: "refusal"` and a `stop_details` category: `"cyber"` (could enable cyber harm, such as malware or exploit development - finding vulnerabilities in source code is allowed), `"bio"` (could enable biological harm), `"frontier_llm"` (could assist the development of competing AI models), `"reasoning_extraction"` (asks the model to reproduce its internal reasoning in the response text), and `"general_harms"` (another usage-policy area - benign work can also trigger it). Branch on `stop_reason` before reading `content` (handling: § `refusal` stop reason under § Migrating to Claude Fable 5.1). Server-side fallback (`fallbacks: "default"`, beta `server-side-fallback-2026-07-01`, Claude API only) retries `"cyber"` and `"frontier_llm"` declines on Claude Sonnet 5; it doesn't retry `"bio"`, `"reasoning_extraction"`, or `"general_harms"` declines. The SDK middleware or your own retry are the alternatives; a fallback model runs without Claude Sonnet 5.5's thinking blocks (breaking change 3), and a client-side retry must drop `between_tools` (breaking change 1); with server-side fallback, a `between_tools` request that falls back to Claude Sonnet 5 runs there with `thinking: {"type": "disabled"}`. Whether a refusal that arrives before any output is billed depends on its refusal category, and it counts against rate limits either way. Real-time cyber safeguards are new for code coming from Sonnet 4.6, Sonnet 4.5, and Haiku 4.5 - for legitimate security work, point the user to the [Cyber Verification Program](https://support.claude.com/en/articles/14604842-real-time-cyber-safeguards-on-claude-opus-and-sonnet). The biology safeguards are the same as Claude Sonnet 5's and leave everyday health and educational questions unaffected; if the `bio` classifier gets in the way of an organization's life-sciences work, it can apply to the Life Sciences Verification Program. If a prompt asks the model to write out its reasoning as a substitute for thinking, remove that instruction (it invites `reasoning_extraction` declines) and read `display: "summarized"` blocks instead.

Claude Sonnet 5.5 拒答的类别比 Claude Sonnet 5 多。拒答是正常的 HTTP 200，带 `stop_reason: "refusal"` 与 `stop_details` 类别：`"cyber"`（可能助长网络危害，如恶意软件或漏洞利用开发——在源代码中寻找漏洞是被允许的）、`"bio"`（可能助长生物危害）、`"frontier_llm"`（可能协助竞争性 AI 模型的开发）、`"reasoning_extraction"`（要求模型在响应文本中复现其内部推理）、`"general_harms"`（其他使用政策领域——良性工作也可能触发）。在读取 `content` 之前按 `stop_reason` 分支（处理方式见 § Migrating to Claude Fable 5.1 中的 § `refusal` stop reason）。服务端回退（`fallbacks: "default"`，beta `server-side-fallback-2026-07-01`，仅 Claude API）会把 `"cyber"` 与 `"frontier_llm"` 拒答在 Claude Sonnet 5 上重试；它不会重试 `"bio"`、`"reasoning_extraction"` 或 `"general_harms"` 拒答。SDK 中间件或你自己的重试是替代方案；回退模型在没有 Claude Sonnet 5.5 思维块的情况下运行（破坏性变更 3），客户端重试必须去掉 `between_tools`（破坏性变更 1）；使用服务端回退时，回退到 Claude Sonnet 5 的 `between_tools` 请求会在那里以 `thinking: {"type": "disabled"}` 运行。输出之前到达的拒答是否计费取决于其拒答类别，且无论是否计费都计入速率限制。实时网络安全防护对来自 Sonnet 4.6、Sonnet 4.5 与 Haiku 4.5 的代码是新的——对正当安全工作，可引导用户前往 [Cyber Verification Program](https://support.claude.com/en/articles/14604842-real-time-cyber-safeguards-on-claude-opus-and-sonnet)。生物安全防护与 Claude Sonnet 5 相同，日常健康与教育类问题不受影响；如果 `bio` 分类器妨碍了组织的生命科学工作，可以申请 Life Sciences Verification Program。如果某个提示词让模型写出其推理以代替思考，移除该指令（它会招致 `reasoning_extraction` 拒答），改为读取 `display: "summarized"` 块。

### What carries over, and what is new versus Claude Sonnet 5 / 相对 Claude Sonnet 5 的沿用与新增

- **New on Claude Sonnet 5.5:** mid-conversation system messages (no beta header), mid-conversation tool changes (beta), per-message effort (beta), task budgets (beta `task-budgets-2026-03-13` - but see § Behavioral shifts on interactive sessions), and tool definitions inside a mid-conversation message (beta `inline-tools-2026-09-15`) - none is available on Claude Sonnet 5.
  **Claude Sonnet 5.5 新增：**会话中途系统消息（无 beta 请求头）、会话中途工具变更（beta）、每消息努力度（beta）、任务预算（beta `task-budgets-2026-03-13`——但交互会话请见 § Behavioral shifts），以及会话中途消息内的工具定义（beta `inline-tools-2026-09-15`）——这些在 Claude Sonnet 5 上都不可用。
- **Prompt caching:** the minimum cacheable prompt is 512 tokens, down from 1,024 on Claude Sonnet 5 (check the prompt caching docs before quoting the exact value).
  **提示词缓存：**最小可缓存提示词为 512 token，低于 Claude Sonnet 5 的 1,024（引用确切数值前请查阅提示词缓存文档）。
- **Carries over:** batch processing, the Files API, PDF support, vision, structured outputs, strict tool use, threshold compaction, compaction on demand (beta `compact-2026-09-04`), and the server-side and client-side tools (computer use as in breaking change 4; on Microsoft Foundry, only what it offers when hosted on Azure).
  **沿用：**批处理、Files API、PDF 支持、视觉、结构化输出、严格工具使用、阈值压缩、按需压缩（beta `compact-2026-09-04`），以及服务端与客户端工具（计算机使用见破坏性变更 4；Microsoft Foundry 上仅限其 Azure 托管时提供的部分）。
- **Rate limits and tiers:** its own rate-limit pool, separate from Claude Sonnet 5's and from the combined Sonnet 4.x pool - re-check the tier's Claude Sonnet 5.5 limits before moving volume. Priority Tier is not supported (as on Claude Sonnet 5).
  **速率限制与档位：**独立的速率限制池，与 Claude Sonnet 5 及合并的 Sonnet 4.x 池都分开——迁移流量前重新检查所在档位的 Claude Sonnet 5.5 限额。Priority Tier 不支持（与 Claude Sonnet 5 相同）。
- **SDK constants** (where the pinned SDK has them; the bare string always works): `Model.ClaudeSonnet5_5` (C#), `anthropic.ModelClaudeSonnet5_5` (Go), `Model.CLAUDE_SONNET_5_5` (Java), `Model::CLAUDE_SONNET_5_5` (PHP), `Anthropic::Model::CLAUDE_SONNET_5_5` (Ruby).
  **SDK 常量**（在锁定的 SDK 具备时；裸字符串始终可用）：`Model.ClaudeSonnet5_5`（C#）、`anthropic.ModelClaudeSonnet5_5`（Go）、`Model.CLAUDE_SONNET_5_5`（Java）、`Model::CLAUDE_SONNET_5_5`（PHP）、`Anthropic::Model::CLAUDE_SONNET_5_5`（Ruby）。

### Behavioral shifts (prompt-tunable) / 行为变化（可通过提示词调整）

None of these break code. Re-evaluate Claude Sonnet 5-specific prompt instructions against them after the effort sweep:

这些都不会破坏代码。在努力度扫描之后，对照它们重新评估 Claude Sonnet 5 专属的提示词指令：

- **Remove workarounds for what got better.** The model is strongest on multistep agentic coding in a real repository, uses connected tools more reliably in agentic workflows, declines fewer benign requests, and holds a system-prompt role more reliably when a user pastes in a competing persona. If the Claude Sonnet 5 prompts carry workarounds for these - refusal steering, tool-call retry shims, instructions like "do not be lazy" - remove them and re-run the evals before tuning anything else.
  **移除对已改善之处的变通手段。**该模型在真实代码库中的多步代理式编码上最强，在代理式工作流中更可靠地使用联网工具，更少拒答良性请求，并且在用户粘贴竞争性人设时更可靠地守住系统提示词角色。如果 Claude Sonnet 5 提示词为这些携带了变通手段——拒答引导、工具调用重试垫片、"do not be lazy" 之类指令——先移除它们并重跑评测，再调整其他任何东西。
- **Tool use in chat and knowledge work.** On chat and knowledge-work tasks the model sometimes answers from its own knowledge or from public web results when a connected tool, skill, or internal search would serve better, and can hold off on tools until asked directly. Remove language that discourages tool use ("only use tools when strictly necessary", "minimize tool calls" - it follows these literally). Where the product should prefer connected sources (enterprise search, account research, support agents), add: *"Use the search tool to check specifics that may have changed since your training, such as what is allowed, required or charged, even when you feel confident. For researched work such as a report or a comparison, gather current sources rather than writing from your training knowledge."*
  **聊天与知识工作中的工具使用。**在聊天与知识工作任务上，当联网工具、技能或内部搜索更能满足需要时，模型有时会改用自己的知识或公开网络结果作答，并且可能在被直接要求之前按兵不动。移除劝阻工具使用的措辞（"only use tools when strictly necessary"、"minimize tool calls"——它会照字面执行）。在产品应当优先使用联网来源之处（企业搜索、账号调研、支持代理），添加：*"Use the search tool to check specifics that may have changed since your training, such as what is allowed, required or charged, even when you feel confident. For researched work such as a report or a comparison, gather current sources rather than writing from your training knowledge."*
- **Mid-turn user messages and task budgets.** The model pays close attention to where text sits relative to a tool result: a message the user typed mid-task that arrives as a mid-conversation system message placed directly after a tool result, or inside a `tool_result` block, can be read as a prompt-injection attempt (the model says so and usually ignores it or waits for confirmation). Task budgets can cause this, because the budget countdown arrives as a system message after every tool result, and so can any harness text added after the tool results on every step (a token countdown, per-step instructions or context); an occasional one-turn reminder arrives far less often - if one draws this reaction, send it less often. Deliver mid-turn user input as a user turn - a text block in the user message that carries the `tool_result` blocks, after the last `tool_result`; keep harness notices (budget countdowns, background-task completions, reminders) in a separate mid-conversation system message that follows, never in the same block as the user's words; never put user text inside a `tool_result` block; and on interactive sessions don't use a task budget (control cost with effort and `max_tokens`; keep task budgets for unattended agentic loops).
  **轮中途用户消息与任务预算。**模型非常在意文本相对于工具结果的位置：用户在任务中途输入、以紧跟工具结果的会话中途系统消息到达、或位于 `tool_result` 块内的消息，可能被读作提示词注入企图（模型会这样说，并且通常忽略它或等待确认）。任务预算可能引发这种情况，因为预算倒计时作为系统消息出现在每个工具结果之后；每一步都在工具结果之后添加 harness 文本（token 倒计时、每步指令或上下文）也会如此；偶发的单轮提醒引发频率低得多——如果某条提醒引出了这种反应，就降低其发送频率。把轮中途用户输入作为用户轮交付——放在携带 `tool_result` 块的用户消息中、最后一条 `tool_result` 之后的文本块；把 harness 通知（预算倒计时、后台任务完成、提醒）放在其后单独的会话中途系统消息里，绝不与用户的话放在同一个块中；绝不把用户文本放进 `tool_result` 块；交互会话不要使用任务预算（用努力度与 `max_tokens` 控制成本；任务预算留给无人值守的代理循环）。
【评论】这里是防御与误伤的双向作用：模型对"紧跟工具结果的系统消息"保持注入警惕是安全设计，但对合法的产品消息也可能误伤，因此消息位置约定成为集成契约的一部分。
- **Verification on coding tasks.** Mostly at `low` effort, the model sometimes reports a code change as done without a check that exercises it (no `npm install`, so the tests and type-checker never ran; only a syntax check; stopping silently when a build tool is missing). If a coding agent runs at `low`, or changes are reported complete without test or build output, add to the system prompt: *"When you change code that can be run, built, or type-checked, run a real check that exercises the change before reporting it done: the project's tests, type-checker, or build, or the changed command itself. A syntax-only check, or a check command that failed to start, does not count; if all that is missing is the project's declared dependencies, install them with its own package manager (e.g. npm install, pip install -r requirements.txt) unless told not to. Only if no real check can run here, say which one you did not run and why instead of reporting the change as done."*
  **编码任务上的验证。**主要在 `low` 努力度下，模型有时会在没有运行过真正检查的情况下报告代码改动已完成（没有 `npm install`，测试与类型检查从未运行；只做了语法检查；缺少构建工具时静默停止）。如果编码代理以 `low` 运行，或改动在没有测试或构建输出的情况下被报告完成，在系统提示词中添加：*"When you change code that can be run, built, or type-checked, run a real check that exercises the change before reporting it done: the project's tests, type-checker, or build, or the changed command itself. A syntax-only check, or a check command that failed to start, does not count; if all that is missing is the project's declared dependencies, install them with its own package manager (e.g. npm install, pip install -r requirements.txt) unless told not to. Only if no real check can run here, say which one you did not run and why instead of reporting the change as done."*
- **Tolerant tool-call handling.** The model occasionally calls a declared tool by a name that differs only in letter case (`bash` for `Bash`), or passes a known parameter under a slightly different name. Don't treat that as fatal: accept the call when the match is unambiguous, or return a `tool_result` with `is_error: true` that states the exact expected name - the model usually corrects the call on its next turn.
  **宽容的工具调用处理。**模型偶尔会以仅大小写不同的名称调用已声明的工具（用 `bash` 指代 `Bash`），或以略有不同的名称传递已知参数。不要把这当作致命错误：在匹配明确时接受该调用，或返回一个 `is_error: true` 的 `tool_result`，写明确切期望的名称——模型通常会在下一轮更正调用。
- **Dense charts and technical drawings.** Give the model a way to crop, zoom, or run code on the image; it reads them markedly more accurately with such tools. On charts the tools help at every effort level and more than raising effort does (with tools at `high` it read charts more accurately than without them at `max`, at a fraction of the cost); on technical drawings they help only from `high` up, most at `xhigh` and `max`.
  **密集图表与技术图纸。**给模型提供对图像裁剪、缩放或运行代码的手段；有这些工具时它读取得明显更准确。对图表，这些工具在每个努力度级别都有帮助，且比提高努力度更有效（`high` 加工具时它读图的准确度超过 `max` 无工具时，成本只是零头）；对技术图纸，它们只在 `high` 及以上有帮助，在 `xhigh` 与 `max` 上帮助最大。
- **Follow-up turns in multi-turn chat.** If the model should treat its earlier answers as settled rather than going back over them when it thinks about a new message, add at the end of the system prompt: *"Once Claude has answered something, Claude treats that answer as done. On later turns Claude's thinking goes to what the person is asking now, and Claude doesn't go back over an earlier answer unless the person asks about it or points out a problem with it."* Leave it out where earlier work should keep being re-examined - long analyses, or agentic tasks where a later step can reveal a mistake in an earlier one.
  **多轮聊天中的后续轮次。**如果希望模型把较早的回答视为已定，而不是在思考新消息时回头翻旧账，在系统提示词末尾添加：*"Once Claude has answered something, Claude treats that answer as done. On later turns Claude's thinking goes to what the person is asking now, and Claude doesn't go back over an earlier answer unless the person asks about it or points out a problem with it."* 在较早工作应当持续被重新审视的场合省略它——长篇分析，或后续步骤可能暴露较早步骤错误的代理式任务。

### Claude Sonnet 5.5 Migration Checklist / Claude Sonnet 5.5 迁移检查清单

- [ ] **[BLOCKS]** Update the `model=` string to `claude-sonnet-5-5` (`anthropic.claude-sonnet-5-5` on Amazon Bedrock); no date suffix.
  **[BLOCKS]** 把 `model=` 字符串更新为 `claude-sonnet-5-5`（Amazon Bedrock 上为 `anthropic.claude-sonnet-5-5`）；无日期后缀。
- [ ] **[BLOCKS]** Replace `thinking: {type: "disabled"}`: try adaptive thinking at `low` effort first; where the route must stay thinking-off, send `{type: "between_tools"}` at effort `high` or below, with no other field in `thinking` and no per-message effort change; drop it from any request client-side code re-sends to another model.
  **[BLOCKS]** 替换 `thinking: {type: "disabled"}`：先尝试 `low` 努力度下的自适应思维；路由必须保持无思维的地方，在 `high` 或以下努力度发送 `{type: "between_tools"}`，`thinking` 中不带其他字段、不做每消息努力度变更；从客户端代码会重发给其他模型的任何请求中去掉它。
- [ ] **[BLOCKS]** Coming from Sonnet 4.6 or earlier or from Haiku 4.5: replace `budget_tokens` with an effort level and remove non-default `temperature` / `top_p` / `top_k`; from Sonnet 4.5 or earlier or Haiku 4.5, also replace assistant prefills.
  **[BLOCKS]** 来自 Sonnet 4.6 或更早或来自 Haiku 4.5：用努力度级别替换 `budget_tokens`，并移除非默认的 `temperature` / `top_p` / `top_k`；来自 Sonnet 4.5 或更早或 Haiku 4.5 的，还要替换 assistant 预填充。
- [ ] **[BLOCKS]** Read content blocks by `type` (a response can begin with `thinking` blocks) and pass `thinking` blocks back unchanged; size `max_tokens` for thinking plus the reply.
  **[BLOCKS]** 按 `type` 读取内容块（响应可能以 `thinking` 块开头）并原样回传 `thinking` 块；为思考加回复设置 `max_tokens`。
- [ ] **[BLOCKS]** Replace `tool_choice` `any` / `tool` with `auto` plus `strict: true` (steering in the prompt, and a check that the call happened) or structured outputs - on `count_tokens` too.
  **[BLOCKS]** 把 `tool_choice` 的 `any` / `tool` 换成 `auto` 加 `strict: true`（在提示词中引导，并检查调用是否发生）或结构化输出——`count_tokens` 上同样。
- [ ] **[BLOCKS]** If the harness builds `messages` itself: keep it append-only and run the preserved-thinking three-step check (§ Migrating to Claude Fable 5.1 from Claude Fable 5) - new accounts are enforced by default on the Claude API and Amazon Bedrock; `block_binding` needs adaptive thinking.
  **[BLOCKS]** 如果 harness 自行构建 `messages`：保持只追加并运行保留思维三步检查（§ Migrating to Claude Fable 5.1 from Claude Fable 5）——新账户在 Claude API 与 Amazon Bedrock 上默认被强制执行；`block_binding` 需要自适应思维。
- [ ] **[BLOCKS]** On the Claude API and Google Cloud, move computer use to `computer_toolset_20260801` and update the agent loop; on Amazon Bedrock send `computer_20251124`.
  **[BLOCKS]** 在 Claude API 与 Google Cloud 上，把计算机使用移到 `computer_toolset_20260801` 并更新代理循环；在 Amazon Bedrock 上发送 `computer_20251124`。
- [ ] **[BLOCKS]** Advisor tool: pair the executor with an accepted advisor (not Claude Opus 4.8 / 4.7 / 4.6, Claude Sonnet 5, or Sonnet 4.6) and expect encrypted `advisor_redacted_result` advice.
  **[BLOCKS]** Advisor 工具：让执行器搭配被接受的 advisor（不能用 Claude Opus 4.8 / 4.7 / 4.6、Claude Sonnet 5 或 Sonnet 4.6），并预期加密的 `advisor_redacted_result` 建议。
- [ ] **[BLOCKS]** Handle `stop_reason: "refusal"` before reading `content` (five categories) and configure fallback; only `cyber` and `frontier_llm` declines are retried by server-side fallback.
  **[BLOCKS]** 在读取 `content` 之前处理 `stop_reason: "refusal"`（五个类别）并配置回退；服务端回退只重试 `cyber` 与 `frontier_llm` 拒答。
- [ ] **[TUNE]** Re-run the effort sweep and set `effort` explicitly (start at `medium` for agentic coding, `low` for chat); lower effort rather than prompting for less thinking; use per-message effort to vary it without losing the cache.
  **[TUNE]** 重做努力度扫描并显式设置 `effort`（代理式编码从 `medium` 开始，聊天用 `low`）；想少思考就降低努力度而不是用提示词；用每消息努力度在不丢失缓存的情况下调整。
- [ ] **[TUNE]** If the UI showed text between tool calls: `display: "updates"` (beta) or `"summarized"` with adaptive thinking, render non-empty `thinking` blocks; declare any send-message tool at session start; remove "hold all findings" instructions; if turns still go quiet, a turn-scoped reminder after about five silent steps, at most two or three times.
  **[TUNE]** 如果 UI 曾显示工具调用之间的文本：自适应思维下用 `display: "updates"`（beta）或 `"summarized"`，渲染非空 `thinking` 块；会话开始时声明任何发消息工具；移除"把发现都留着"类指令；如果轮次仍然沉默，约五步沉默后加一条轮次作用域提醒，至多两三次。
- [ ] **[TUNE]** Prompts: remove workarounds (refusal steering, tool-call retry shims, "do not be lazy") and tool-discouraging language; deliver mid-turn user input as a user turn and drop task budgets on interactive sessions; add the verification paragraph for low-effort coding agents; accept or correct near-miss tool names instead of failing; give crop/zoom/code tools for dense charts and drawings; the settled-answers line for multi-turn chat where it fits.
  **[TUNE]** 提示词：移除变通手段（拒答引导、工具调用重试垫片、"do not be lazy"）与劝阻工具使用的措辞；把轮中途用户输入作为用户轮交付，交互会话取消任务预算；为低努力度编码代理添加验证段落；对近似正确的工具名接受或纠正而不是失败；为密集图表与图纸提供裁剪/缩放/运行代码工具；在合适的多轮聊天中添加"回答即定案"条款。
- [ ] **[TUNE]** Re-baseline cost and latency at the chosen effort level (same prices as Claude Sonnet 5; own rate-limit pool; no Priority Tier).
  **[TUNE]** 在选定的努力度级别重新校准成本与延迟基准（价格与 Claude Sonnet 5 相同；独立速率限制池；无 Priority Tier）。

---

## Verify the Migration / 验证迁移

After updating, spot-check that the new model is actually being used. Replace `YOUR_TARGET_MODEL` with the model string you migrated to (e.g. `claude-fable-5-1`, `claude-opus-5-5`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-sonnet-5-5`, `claude-sonnet-5`, `claude-sonnet-4-6`, `claude-haiku-4-5`) and keep the assertion prefix in sync:

更新之后，抽查确认新模型确实在被使用。把 `YOUR_TARGET_MODEL` 替换为你迁移到的模型字符串（如 `claude-fable-5-1`、`claude-opus-5-5`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7`、`claude-sonnet-5-5`、`claude-sonnet-5`、`claude-sonnet-4-6`、`claude-haiku-4-5`），并保持断言前缀同步：

```python
YOUR_TARGET_MODEL = "claude-opus-5-5"  # or "claude-opus-5", "claude-opus-4-7", "claude-sonnet-5-5", "claude-sonnet-5", "claude-sonnet-4-6", "claude-haiku-4-5"
response = client.messages.create(model=YOUR_TARGET_MODEL, max_tokens=64, messages=[...])
assert response.model.startswith(YOUR_TARGET_MODEL), response.model
```

Prefix collision: `claude-opus-5-5` starts with `claude-opus-5`, so when your target is `claude-opus-5`, also assert `not response.model.startswith("claude-opus-5-5")`. The same holds for Sonnet: `claude-sonnet-5-5` starts with `claude-sonnet-5`, so a `claude-sonnet-5` check also asserts `not response.model.startswith("claude-sonnet-5-5")`.

前缀冲突：`claude-opus-5-5` 以 `claude-opus-5` 开头，因此当你的目标是 `claude-opus-5` 时，还要断言 `not response.model.startswith("claude-opus-5-5")`。Sonnet 同理：`claude-sonnet-5-5` 以 `claude-sonnet-5` 开头，因此 `claude-sonnet-5` 检查还要断言 `not response.model.startswith("claude-sonnet-5-5")`。

For rate-limit headroom changes, pricing, or capability deltas (vision, structured outputs, effort support), query the Models API:

有关速率限制余量变化、定价或能力差异（视觉、结构化输出、努力度支持），查询 Models API：

```python
m = client.models.retrieve(YOUR_TARGET_MODEL)
m.max_input_tokens, m.max_tokens
m.capabilities["effort"]["max"]["supported"]
```

See `shared/models.md` for the full capability lookup pattern.

完整的能力查询模式见 `shared/models.md`。

---

## Ground the migration with an eval / 用评测为迁移提供依据

A spot-check confirms the new model answers; it doesn't confirm the app still behaves the way the user wants. When the user reports a behavioral regression on the new model - e.g. *"it refuses things the old one handled fine"*, *"tool calls dropped off after the swap"*, *"responses got twice as long"* - don't tune the prompt by feel. Read `shared/evals/build-eval.md` and build a small eval that captures the regression, then read `shared/evals/eval-hillclimb.md` to iterate the prompt or harness against that eval until the score moves. Grounding the fix in an eval keeps the migration decision honest and leaves the user with a regression test for the next model swap.

抽查只能确认新模型能回答；它不能确认应用仍按用户期望的方式运行。当用户在新模型上报告行为退化时——例如 *"it refuses things the old one handled fine"*、*"tool calls dropped off after the swap"*、*"responses got twice as long"*——不要凭感觉调提示词。阅读 `shared/evals/build-eval.md`，构建一个能捕捉该退化的小型评测，然后阅读 `shared/evals/eval-hillclimb.md`，针对该评测迭代提示词或 harness，直到分数变动。把修复建立在评测上能让迁移决策保持诚实，并为下一次模型切换给用户留下一个回归测试。


