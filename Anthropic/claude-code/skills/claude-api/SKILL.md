---
name: claude-api
description: |-
  Reference for the Claude API / Anthropic SDK — model ids, pricing, params, streaming, tool use, MCP, agents, caching, token counting, model migration.
  TRIGGER — read BEFORE opening the target file; don't skip because it "looks like a one-liner" — whenever: the prompt names Claude/Anthropic in any form (Claude, Anthropic, Fable, Opus, Sonnet, Haiku, `anthropic`, `@anthropic-ai`, `claude-*`, `us.anthropic.*`, `[1m]`); the user asks about an LLM (pricing/model choice/limits/caching) — never answer from memory; OR the task is LLM-shaped with provider unstated (agent/MCP/tool-definition/multi-agent/RAG/LLM-judge/computer-use; generate/summarize/extract/classify/rewrite/converse over NL; debugging refusals/cutoffs/streaming/tool-calls/tokens).
  SKIP only when another provider is being worked on (overrides all triggers): OpenAI/GPT/Gemini/Llama/Mistral/Cohere/Ollama named in the query; OR `grep -rE 'openai|langchain_openai|google.generativeai|genai|mistralai|cohere|ollama'` over the project hits (run this grep FIRST if no provider named — don't Read the file).
---
<!-- BILINGUAL-EN-ZH -->

# Building LLM-Powered Applications with Claude / 使用 Claude 构建大模型驱动的应用

This skill helps you build LLM-powered applications with Claude. Choose the right surface based on your needs, detect the project language, then read the relevant language-specific documentation.

本技能帮助你使用 Claude 构建大模型驱动的应用。请根据需求选择合适的接入层，检测项目语言，然后阅读相应的语言专属文档。

【评论】front matter 中的 TRIGGER/SKIP 描述采用激进的触发式设计：只要提示词中出现任何 Claude/Anthropic 相关记号即强制加载本技能，并要求先跑 grep 排除其他厂商，属于典型的"抢占式技能路由"写法。

## Before You Start / 开始之前

Scan the target file (or, if no target file, the prompt and project) for non-Anthropic provider markers - `import openai`, `from openai`, `langchain_openai`, `OpenAI(`, `gpt-4`, `gpt-5`, file names like `agent-openai.py` or `*-generic.py`, or any explicit instruction to keep the code provider-neutral. If you find any, stop and tell the user that this skill produces Claude/Anthropic SDK code; ask whether they want to switch the file to Claude or want a non-Claude implementation. Do not edit a non-Anthropic file with Anthropic SDK calls. (Exception: the `prompt-audit` subcommand is non-interactive and does not stop here - it records non-Anthropic provider markers in its report's stated assumptions and never proposes switching a non-Anthropic file to the Anthropic SDK.)

扫描目标文件（若无目标文件，则扫描提示词与项目）以查找非 Anthropic 厂商的标记——`import openai`、`from openai`、`langchain_openai`、`OpenAI(`、`gpt-4`、`gpt-5`、诸如 `agent-openai.py` 或 `*-generic.py` 的文件名，或任何要求保持代码厂商中立的明确指令。一旦发现，立即停止并告知用户：本技能产出的是 Claude/Anthropic SDK 代码；询问用户是想把该文件切换为 Claude，还是想要非 Claude 的实现。不要用 Anthropic SDK 调用去修改非 Anthropic 文件。（例外：`prompt-audit` 子命令是非交互式的，不会在此停下——它会把非 Anthropic 厂商标记记录在报告的"既定假设"中，且绝不会提议把非 Anthropic 文件切换到 Anthropic SDK。）

## Output Requirement / 输出要求

When the user asks you to add, modify, or implement a Claude feature, your code must call Claude through one of:

当用户要求你添加、修改或实现某个 Claude 功能时，你的代码必须通过以下方式之一调用 Claude：

1. **The official Anthropic SDK** for the project's language (`anthropic`, `@anthropic-ai/sdk`, `com.anthropic.*`, etc.). This is the default whenever a supported SDK exists for the project.
   该项目语言对应的 **Anthropic 官方 SDK**（`anthropic`、`@anthropic-ai/sdk`、`com.anthropic.*` 等）。只要项目存在受支持的 SDK，这就是默认选择。
2. **Raw HTTP** (`curl`, `requests`, `fetch`, `httpx`, etc.) - only when the user explicitly asks for cURL/REST/raw HTTP, the project is a shell/cURL project, or the language has no official SDK.
   **原始 HTTP**（`curl`、`requests`、`fetch`、`httpx` 等）——仅在用户明确要求 cURL/REST/原始 HTTP、项目本身是 shell/cURL 项目，或该语言没有官方 SDK 时使用。

Never mix the two - don't reach for `requests`/`fetch` in a Python or TypeScript project just because it feels lighter. Never fall back to OpenAI-compatible shims.

绝不混用两者——不要仅因为感觉更轻量就在 Python 或 TypeScript 项目里改用 `requests`/`fetch`。绝不退而使用 OpenAI 兼容的垫片层。

**Never guess SDK usage.** Function names, class names, namespaces, method signatures, and import paths must come from explicit documentation - either the `{lang}/` files in this skill or the official SDK repositories or documentation links listed in `shared/live-sources.md`. If the binding you need is not explicitly documented in the skill files, WebFetch the relevant SDK repo from `shared/live-sources.md` before writing code. Do not infer Ruby/Java/Go/PHP/C# APIs from cURL shapes or from another language's SDK.

**绝不凭猜测使用 SDK。** 函数名、类名、命名空间、方法签名与导入路径必须来自明确的文档——要么是本技能中的，要么是 `{lang}/` 文件，或 `shared/live-sources.md` 中列出的官方 SDK 仓库或文档链接。如果你需要的绑定在技能文件中没有明确记载，请在写代码前先 WebFetch `shared/live-sources.md` 中相应的 SDK 仓库。不要从 cURL 请求形态或其他语言的 SDK 去推断 Ruby/Java/Go/PHP/C# 的 API。

**If WebFetch or repository access fails** (network restricted, timeouts, clone blocked): do not keep retrying - write code from the patterns and namespace/package tables in the `{lang}/` file, run the compiler or interpreter on it, and iterate on the error output. For statically-typed SDKs (C#, Java, Go) a compile-fix loop against local errors reaches working code faster than blocked network research.

**如果 WebFetch 或仓库访问失败**（网络受限、超时、clone 被阻断）：不要反复重试——改为依据 `{lang}/` 文件中的模式与命名空间/包表编写代码，运行编译器或解释器，并根据错误输出迭代。对静态类型 SDK（C#、Java、Go）而言，针对本地错误做"编译-修复"循环，比被阻断的网络查阅更快得到可用代码。

【评论】此处把"网络查证失败"的处理路径明确导向本地编译反馈循环，是一种应对沙箱断网环境的兜底策略。

## Defaults / 默认设置

Unless the user requests otherwise:

除非用户另有要求：

For the Claude model version, please use Claude Opus 5.5, which you can access via the exact model string `claude-opus-5-5`. Please default to using adaptive thinking (`thinking: {type: "adaptive"}`) for anything remotely complicated. And finally, please default to streaming for any request that may involve long input, long output, or high `max_tokens` - it prevents hitting request timeouts. Use the SDK's `.get_final_message()` / `.finalMessage()` helper to get the complete response if you don't need to handle individual stream events. When a streaming request defines user-defined (client) tools, set `eager_input_streaming: true` on each of those tools so large tool inputs (file contents, code, documents) stream as they are generated instead of arriving in one burst after the server finishes buffering them; the client then owns validation: the SDKs' tolerant parsers can return a silently truncated input instead of raising, so validate each parsed tool input against its schema before running it (the typed runner helpers such as `betaZodTool` / typed `@beta_tool` do this; `betaTool()` JSON-Schema tools and manual loops must validate themselves), treat a failure like invalid JSON (`INVALID_JSON` error `tool_result` when you hold the block, re-issue otherwise), check `max_tokens` / `refusal` stop reasons before running tools, and catch only the SDK's JSON error, never its typed API errors - pattern in `shared/tool-use-concepts.md` -> Eager input streaming. Leave it off for non-streaming requests, for server tools, and when the request goes through a proxy or an older Bedrock model deployment that rejects the field.

关于 Claude 模型版本，请使用 Claude Opus 5.5，可通过精确的模型字符串 `claude-opus-5-5` 访问。凡是稍微复杂的任务，请默认使用自适应思考（`thinking: {type: "adaptive"}`）。最后，凡可能涉及长输入、长输出或较高 `max_tokens` 的请求，请默认使用流式输出——这可以避免触发请求超时。如果不需要逐个处理流事件，可使用 SDK 的 `.get_final_message()` / `.finalMessage()` 辅助方法获取完整响应。当流式请求定义了用户自定义（客户端）工具时，请为其中每个工具设置 `eager_input_streaming: true`，使大型工具输入（文件内容、代码、文档）在生成过程中即以流式传输，而不是在服务器完成缓冲后一次性到达；此时校验责任由客户端承担：SDK 宽容的解析器可能返回被静默截断的输入而不抛出异常，因此在运行每个解析出的工具输入之前，必须先对照其 schema 进行校验（`betaZodTool` 等类型化运行器辅助方法以及带类型的 `@beta_tool` 已内置此校验；`betaTool()` JSON-Schema 工具和手写循环必须自行校验），将失败视同无效 JSON 处理（持有该块时返回 `INVALID_JSON` 错误的 `tool_result`，否则重新发起请求），在运行工具前检查 `max_tokens` / `refusal` 停止原因，并且只捕获 SDK 的 JSON 错误、绝不捕获其类型化 API 错误——模式见 `shared/tool-use-concepts.md` -> Eager input streaming。非流式请求、服务器端工具，以及请求经过代理或会拒绝该字段的旧版 Bedrock 模型部署时，请保持该选项关闭。

【评论】默认模型被固定写死为某个具体模型字符串，且流式/思考等参数偏好一并列出——技能以硬编码默认值方式消除模型选择上的歧义。
## Warning: API Drift - Your Training Prior May Be Stale / 警告：API 漂移——你的训练先验可能已过时

Several common Claude API shapes changed in 2025-2026. If you recall a pattern from training, verify it against the `{lang}/` files in this skill before writing - the rows below are the most frequent drift points:

多个常见的 Claude API 写法在 2025-2026 年间发生了变化。如果你回忆起训练中学到的某个模式，请在写代码之前先对照本技能中的 `{lang}/` 文件进行核实——下表是最常见的漂移点：

| Area | Stale prior | Current API |
|---|---|---|
| Extended thinking | `thinking: {type: "enabled", budget_tokens: N}` | On Claude 4.6+ models: `thinking: {type: "adaptive"}`. `budget_tokens` is deprecated on Opus 4.6 / Sonnet 4.6 and **rejected with a 400** on Fable 5/5.1 / Sonnet 5.5 / Sonnet 5 / Opus 5.5 / 5 / 4.8 / 4.7. Pre-4.6 models still use `budget_tokens`. |
| Web search / web fetch tool type | `web_search_20250305`, `web_fetch_20250910` | `web_search_20260209`, `web_fetch_20260209` (dynamic filtering) on Opus 5.5/5/4.8/4.7/4.6, Sonnet 5.5, Sonnet 5, and Sonnet 4.6. Older models keep the basic variants; on Vertex AI only basic `web_search_20250305` is available (web fetch is not on Vertex) - see the Server Tools QR below. |
| PHP parameter names | snake_case wire names as named args (`max_tokens`) | Top-level named args are camelCase (`maxTokens`). Nested array keys vary by feature (e.g. `'taskBudget'`, `'skillID'`, `'mcp_server_name'`) - copy the exact key from the documented example; do not bulk-convert. |
| Managed Agents credentials | Keep secrets host-side via custom tools (the only option before vaults shipped) | Vault `environment_variable` credentials - stored by Anthropic, substituted at egress, never visible in the sandbox (`shared/managed-agents-tools.md` -> Vaults). Host-side custom tools remain the fallback for self-hosted sandboxes. |
| Files API / Skills | `client.beta.files.*` / `client.beta.skills.*` with beta `files-api-2025-04-14` / `skills-2025-10-02` | Out of beta: `client.files.*` / `client.skills.*`, no beta header. In current SDKs `client.beta.files` / `client.beta.skills` have breaking shape changes from previous versions, matching the stable namespaces - migrate per `shared/live-sources.md` -> Files API / Skills Guide. |

| 领域 | 过时的记忆 | 当前 API |
|---|---|---|
| 扩展思考 | `thinking: {type: "enabled", budget_tokens: N}` | 在 Claude 4.6 及以上模型上：`thinking: {type: "adaptive"}`。`budget_tokens` 在 Opus 4.6 / Sonnet 4.6 上已弃用，在 Fable 5/5.1 / Sonnet 5.5 / Sonnet 5 / Opus 5.5 / 5 / 4.8 / 4.7 上会被 **400 错误拒绝**。4.6 之前的模型仍使用 `budget_tokens`。 |
| 网络搜索 / 网络抓取工具类型 | `web_search_20250305`、`web_fetch_20250910` | 在 Opus 5.5/5/4.8/4.7/4.6、Sonnet 5.5、Sonnet 5、Sonnet 4.6 上为 `web_search_20260209`、`web_fetch_20260209`（支持动态过滤）。较旧的模型保留基础变体；在 Vertex AI 上只有基础的 `web_search_20250305` 可用（Vertex 上没有 web fetch）——见下方的 Server Tools 速查表。 |
| PHP 参数名 | 以 snake_case 线上名称作为命名参数（`max_tokens`） | 顶层命名参数为 camelCase（`maxTokens`）。嵌套数组的键随功能而异（例如 `'taskBudget'`、`'skillID'`、`'mcp_server_name'`）——请从文档示例中原样复制确切的键，不要批量转换。 |
| Managed Agents 凭据 | 通过自定义工具在宿主侧保存密钥（vaults 推出前的唯一选项） | Vault `environment_variable` 凭据——由 Anthropic 存储，在出口处替换，沙箱内永远不可见（`shared/managed-agents-tools.md` -> Vaults）。自托管沙箱仍以宿主侧自定义工具为后备方案。 |
| Files API / Skills | `client.beta.files.*` / `client.beta.skills.*`，附 beta 头 `files-api-2025-04-14` / `skills-2025-10-02` | 已结束 beta：`client.files.*` / `client.skills.*`，无需 beta 头。在当前 SDK 中，`client.beta.files` / `client.beta.skills` 相比旧版本有破坏性结构变化，与稳定命名空间保持一致——按 `shared/live-sources.md` -> Files API / Skills Guide 迁移。 |

【评论】用表格逐项对比"模型记忆中的旧写法"与当前 API，是技能文档对抗训练数据过时（训练截止日期滞后）问题的常见手段。

The `{lang}/` files in this skill are authoritative over recalled patterns.

本技能中的 `{lang}/` 文件优先于你回忆起来的任何模式。

---

## Subcommands / 子命令

If the User Request at the bottom of this prompt is a bare subcommand string (no prose), search every **Subcommands** table in this document - including any in sections appended below - and follow the matching Action column directly. This lets users invoke specific flows via `/claude-api <subcommand>`. If no table in the document matches, treat the request as normal prose.

如果本提示词底部的 User Request 是一个裸的子命令字符串（不含散文说明），请搜索本文档中的每个 **Subcommands** 表格——包括下文附加章节中的表格——并直接遵循匹配行的 Action 列。这使用户能够通过 `/claude-api <subcommand>` 调用特定流程。如果文档中没有任何表格匹配，则将该请求当作普通散文处理。

| Subcommand | Action |
|---|---|
| `migrate` | Migrate existing Claude API code to a newer model. **Read `shared/model-migration.md` immediately** and follow it in order: Step 0 (confirm scope - ask which files/directories before any edit), Step 1 (classify each file), then the per-target breaking-changes section. Do not summarize the guide - execute it. If the user did not name a target model, ask which model to migrate to in the same turn as the scope question. After the per-target changes are applied, audit the in-scope prompt text, tool descriptions, and request code against `shared/prompt-audit.md` - prompting written for the source model is part of every migration, and it does not announce itself. |
| `prompt-audit` | Audit existing prompts, tool descriptions, skills, and agent configuration files (`CLAUDE.md`, rule files, commands, subagents) for dated patterns ("cruft"): text written for older models, and instructions the repository has outgrown or that contradict each other. **Read `shared/prompt-audit.md` immediately** and follow it in order: Step 0 (establish scope and target model from the request and the repository - state the assumptions in the report, do not stop to ask), inventory, provenance, then the pattern scan. Produce both deliverables in full - the audit report (findings with `file:line`, pattern, why it's obsolete, confidence) and a proposed diff - without pausing for confirmation; apply edits only if the request explicitly asked for them. Do not summarize the guide - execute it. |
| `upgrade` | Upgrade the project's Anthropic SDK dependency across a major version - currently the Python SDK, `anthropic` 0.x -> 1.x. Trailing words may name the language and/or a scope (`upgrade python`, `upgrade python sdk src/`). **Read `python/claude-api/sdk-upgrade.md` immediately** and follow it in order: Step 0 (confirm scope, then establish the current and target versions - a published 1.x must exist before you write a pin), the Step 1 inventory, each numbered section, then verification and the report. Do not summarize the guide - execute it. If the detected or named language has no `sdk-upgrade.md` in this skill, say that no major-version upgrade guide is bundled for that SDK yet and point the user at that SDK's CHANGELOG (repositories in `shared/live-sources.md`); do not improvise one from the Python guide. This is not model migration - to move code to a newer Claude model, use `migrate`. |
| `cost-optimize` | Reduce what existing Claude API code costs to run, without sacrificing output quality. **Read `shared/cost-optimization.md` immediately** and follow it in order: Step 0 (establish scope, quality bar, and baseline), the token profile - measured through the Usage and Cost Admin API when the user has an Admin API key, from the app's own `response.usage` logs when it has those (ask), or estimated from the code otherwise - then a savings-ranked shortlist of levers (quoted in dollars, % of bill, or relative buckets depending on which of those data sources you have), free wins (caching, input-token hygiene, loop hygiene, output-token hygiene, batch) before tradeoffs (budgets, effort, model choice, multi-model); any lever that earns a place becomes its own diff - proposed by default, applied and measured against the eval covering the traffic it touches when the user asks and approves - and "no changes recommended" is a valid outcome. Two standing rules: every run that exercises the model spends real money, so get the user's approval first; and when context for a lever is missing, work through it interactively with the user - this workflow is not expected to one-shot the audit. Do not summarize the guide - execute it; presenting the profile and the ranked plan to the user is part of executing it. |
| `build-eval` | Help the user build an eval set for their Claude-powered app. **Read `shared/evals/build-eval.md` immediately** and run its interview: Step 0 (what's being evaluated), Step 1 (source the prompts - existing eval / transcripts / synthesized), Step 2 (grading method), Step 3 (runnable script + measured cost). Get the user's explicit sign-off on the inputs, the grading method, and the cost before producing the eval. |
| `preserved-thinking-migration` | Make an existing integration compatible with preserved thinking - the check that keeps a thinking block valid only in the conversation that produced it. **Read `shared/preserved-thinking-migration.md` immediately** and follow it in order: Step 0 (scope, traffic classes, platform and model, enforcement status, quality bar, baseline), Step 0.5 (prove the check is running with the three-request self-test), Step 1 (capture request bodies, diff consecutive pairs with `shared/preserved-thinking-migration/prefix_diff.py`, scan the code for the causes, name each edit and whether it is deliberate), Step 2 (replay a test slice with `prefix_mismatch_behavior: "drop_block"` under the `thinking-binding-controls-2026-08-01` header, count new dropped blocks per conversation, read the diagnosis header when present), Step 3 (one cause per diff in order of reasoning lost - proposed by default, applied when the user asks - then re-measure, keep or revert; the three-arm protocol when an eval exists), the model-switch section (in `shared/preserved-thinking-migration/causes.md`, with the cause table and the keep list) when the harness routes between models, Step 4 (the break profile and the changes). Two standing rules: every replay spends real money, so get the user's approval for the measurement budget first; and "no changes recommended" - the slice replayed thinking and nothing was dropped - is a valid outcome. Causes that have an append-only form only under a newer beta (keep-tail and background compaction: `compact-2026-09-04`; same-name tool changes: `inline-tools-2026-09-15`) are, where that beta is not available, measured and decided, not rewritten. For the *why* (the three-step check, the append-only edit table) it chains to `shared/model-migration.md` -> Breaking change 3; do not summarize the guide - execute it. |
| `hillclimb` | Iteratively improve the user's app against an existing eval. **Read `shared/evals/eval-hillclimb.md` immediately** and follow it: Step 0 (confirm a runnable eval exists - if not, route to `build-eval`), Step 1 (what to change / what's off-limits), Step 2 (budget + stopping condition from measured per-run cost), get the plan approved, then the read->propose->apply->run->record loop with on-disk state and a train/validation/test split. |

| 子命令 | 动作 |
|---|---|
| `migrate` | 将现有的 Claude API 代码迁移到更新的模型。**立即阅读 `shared/model-migration.md`** 并按顺序执行：Step 0（确认范围——在任何编辑前询问涉及哪些文件/目录）、Step 1（对每个文件分类），然后是与目标模型对应的破坏性变更章节。不要概述指南——要执行它。如果用户未指明目标模型，请在询问范围问题的同一轮中追问要迁移到哪个模型。完成针对目标的修改后，依据 `shared/prompt-audit.md` 审计范围内的提示词文本、工具描述与请求代码——为源模型撰写的提示词是每次迁移的一部分，而且它不会自我声明。 |
| `prompt-audit` | 审计现有提示词、工具描述、技能与智能体配置文件（`CLAUDE.md`、规则文件、命令、子智能体）中过时的模式（"cruft"，陈旧残留）：为旧模型撰写的文本，以及仓库已经不再适用或彼此矛盾的指令。**立即阅读 `shared/prompt-audit.md`** 并按顺序执行：Step 0（从请求与仓库中确定范围与目标模型——在报告中陈述假设，不要停下来询问）、清单、来源梳理，然后进行模式扫描。完整产出两份交付物——审计报告（含 `file:line`、模式、过时原因、置信度的发现）与建议的差异补丁——中途不停下来等确认；仅当请求明确要求时才应用编辑。不要概述指南——要执行它。 |
| `upgrade` | 跨主版本升级项目的 Anthropic SDK 依赖——目前为 Python SDK，`anthropic` 0.x -> 1.x。末尾附带的词可以指明语言和/或范围（`upgrade python`、`upgrade python sdk src/`）。**立即阅读 `python/claude-api/sdk-upgrade.md`** 并按顺序执行：Step 0（确认范围，然后确定当前版本与目标版本——在写入版本锁定前必须已存在已发布的 1.x）、Step 1 清单、各编号章节，然后是验证与报告。不要概述指南——要执行它。如果检测到或指明的语言在本技能中没有对应的 `sdk-upgrade.md`，说明该 SDK 尚未附带主版本升级指南，并引导用户查阅该 SDK 的 CHANGELOG（仓库见 `shared/live-sources.md`）；不要基于 Python 指南自行编造。这不是模型迁移——要将代码迁移到更新的 Claude 模型，请使用 `migrate`。 |
| `cost-optimize` | 在不牺牲输出质量的前提下，降低现有 Claude API 代码的运行成本。**立即阅读 `shared/cost-optimization.md`** 并按顺序执行：Step 0（确定范围、质量标准与基线），随后是 token 画像——用户有 Admin API 密钥时通过 Usage and Cost Admin API 实测，应用自身有 `response.usage` 日志时从日志取数（需询问），否则从代码估算——然后给出按节省额排序的优化手段候选清单（依据掌握的数据源，分别以美元、账单百分比或相对档位表述），先做免费收益（缓存、输入 token 卫生、循环卫生、输出 token 卫生、批处理），再考虑权衡类手段（预算、力度、模型选择、多模型）；任何入选的手段都形成独立的 diff——默认只提出建议，当用户要求并批准后，针对覆盖相关流量的 eval 进行应用并测量——而"不建议任何修改"也是有效结论。两条长期规则：每次实际调用模型都会花费真金白银，因此必须先获得用户批准；当某个手段缺少背景信息时，与用户交互式推进——本工作流并不要求一次完成审计。不要概述指南——要执行它；向用户呈现 token 画像与排序后的计划本身就是执行的一部分。 |
| `build-eval` | 帮助用户为其 Claude 应用构建评测集。**立即阅读 `shared/evals/build-eval.md`** 并执行其访谈流程：Step 0（评测对象是什么）、Step 1（确定提示词来源——现有评测 / 对话记录 / 合成）、Step 2（评分方法）、Step 3（可运行脚本 + 实测成本）。在产出评测之前，须获得用户对输入、评分方法与成本的明确确认。 |
| `preserved-thinking-migration` | 使现有集成兼容 preserved thinking（保留思考）——即保证思考块仅在产生它的对话中有效的那项检查。**立即阅读 `shared/preserved-thinking-migration.md`** 并按顺序执行：Step 0（范围、流量类别、平台与模型、强制执行状态、质量标准、基线），Step 0.5（用三次请求自测证明该检查正在生效），Step 1（捕获请求体，用 `shared/preserved-thinking-migration/prefix_diff.py` 对相邻请求对做差分，扫描代码寻找成因，指明每处编辑及其是否属有意为之），Step 2（在 `thinking-binding-controls-2026-08-01` 头之下以 `prefix_mismatch_behavior: "drop_block"` 重放测试切片，统计每个对话新增的丢弃块数，若存在诊断头则读取），Step 3（每个 diff 对应一个成因，按推理损失程度排序——默认只提出建议，用户要求时才应用——然后重新测量，保留或回退；存在 eval 时采用三臂协议），当框架在多个模型间路由时，执行模型切换章节（位于 `shared/preserved-thinking-migration/causes.md`，含成因表与保留清单），Step 4（中断特征与修改内容）。两条长期规则：每次重放都花费真金白银，因此必须先获得用户对测量预算的批准；并且"不建议任何修改"——切片重放后思考保留、未丢弃任何内容——也是有效结论。仅在更新的 beta 下才有只追加形式的成因（keep-tail 与后台压缩：`compact-2026-09-04`；同名工具变更：`inline-tools-2026-09-15`），在该 beta 不可用之处，对它们进行测量与决策，而不是改写。有关*原因*（三步检查、只追加编辑表），本流程衔接 `shared/model-migration.md` -> Breaking change 3；不要概述指南——要执行它。 |
| `hillclimb` | 基于现有 eval 迭代改进用户的应用。**立即阅读 `shared/evals/eval-hillclimb.md`** 并遵循它：Step 0（确认存在可运行的 eval——若无，转用 `build-eval`）、Step 1（可改动什么/什么不可动）、Step 2（依据实测的单次运行成本确定预算与停止条件），获得计划批准后，进入读取->提出->应用->运行->记录的循环，使用磁盘上的状态并划分训练/验证/测试集。 |

【评论】将迁移、审计、升级、成本优化等完整工作流以子命令表格形式内嵌，每个子命令又强制先读独立的指南文件，体现了"技能 = 路由层 + 外挂手册"的组织方式。

---

## Language Detection / 语言检测

Before reading code examples, determine which language the user is working in (exception: for the `prompt-audit` subcommand, skip this section's ask steps - the audit is non-interactive and its inventory is language-agnostic; when no language is inferable, proceed without asking and state the assumption in the report):

在阅读代码示例之前，先判断用户使用的是哪种语言（例外：对于 `prompt-audit` 子命令，跳过本节的询问步骤——该审计是非交互式的，其清单与语言无关；当无法推断语言时，不做询问直接继续，并在报告中陈述该假设）：

1. **Look at project files** to infer the language:
   **查看项目文件**以推断语言：

 - `*.py`, `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` -> **Python** - read from `python/`
   存在 `*.py`、`requirements.txt`、`pyproject.toml`、`setup.py`、`Pipfile` -> **Python**——从 `python/` 读取
 - `*.ts`, `*.tsx`, `package.json`, `tsconfig.json` -> **TypeScript** - read from `typescript/`
   存在 `*.ts`、`*.tsx`、`package.json`、`tsconfig.json` -> **TypeScript**——从 `typescript/` 读取
 - `*.js`, `*.jsx` (no `.ts` files present) -> **TypeScript** - JS uses the same SDK, read from `typescript/`
   存在 `*.js`、`*.jsx`（且没有 `.ts` 文件）-> **TypeScript**——JS 使用同一个 SDK，从 `typescript/` 读取
 - `*.java`, `pom.xml`, `build.gradle` -> **Java** - read from `java/`
   存在 `*.java`、`pom.xml`、`build.gradle` -> **Java**——从 `java/` 读取
 - `*.kt`, `*.kts`, `build.gradle.kts` -> **Java** - Kotlin uses the Java SDK, read from `java/`
   存在 `*.kt`、`*.kts`、`build.gradle.kts` -> **Java**——Kotlin 使用 Java SDK，从 `java/` 读取
 - `*.scala`, `build.sbt` -> **Java** - Scala uses the Java SDK, read from `java/`
   存在 `*.scala`、`build.sbt` -> **Java**——Scala 使用 Java SDK，从 `java/` 读取
 - `*.go`, `go.mod` -> **Go** - read from `go/`
   存在 `*.go`、`go.mod` -> **Go**——从 `go/` 读取
 - `*.rb`, `Gemfile` -> **Ruby** - read from `ruby/`
   存在 `*.rb`、`Gemfile` -> **Ruby**——从 `ruby/` 读取
 - `*.cs`, `*.csproj` -> **C#** - read from `csharp/`
   存在 `*.cs`、`*.csproj` -> **C#**——从 `csharp/` 读取
 - `*.php`, `composer.json` -> **PHP** - read from `php/`
   存在 `*.php`、`composer.json` -> **PHP**——从 `php/` 读取

2. **If multiple languages detected** (e.g., both Python and TypeScript files):
   **如果检测到多种语言**（例如同时存在 Python 与 TypeScript 文件）：

 - Check which language the user's current file or question relates to
   检查用户当前文件或问题与哪种语言相关
 - If still ambiguous, ask: "I detected both Python and TypeScript files. Which language are you using for the Claude API integration?"
   若仍不明确，则询问："I detected both Python and TypeScript files. Which language are you using for the Claude API integration?"（我检测到 Python 和 TypeScript 文件。你的 Claude API 集成使用的是哪种语言？）

3. **If language can't be inferred** (empty project, no source files, or unsupported language):
   **如果无法推断语言**（空项目、没有源文件，或语言不受支持）：

 - Use AskUserQuestion with options: Python, TypeScript, Java, Go, Ruby, cURL/raw HTTP, C#, PHP
   使用 AskUserQuestion，选项为：Python、TypeScript、Java、Go、Ruby、cURL/raw HTTP、C#、PHP
 - If AskUserQuestion is unavailable, default to Python examples and note: "Showing Python examples. Let me know if you need a different language."
   如果 AskUserQuestion 不可用，默认展示 Python 示例并注明："Showing Python examples. Let me know if you need a different language."（正在展示 Python 示例。如需其他语言请告知。）

4. **If unsupported language detected** (Rust, Swift, C++, Elixir, etc.):
   **如果检测到不受支持的语言**（Rust、Swift、C++、Elixir 等）：

 - Suggest cURL/raw HTTP examples from `curl/` and note that community SDKs may exist
   建议使用 `curl/` 中的 cURL/原始 HTTP 示例，并说明可能存在社区 SDK
 - Offer to show Python or TypeScript examples as reference implementations
   并可提供 Python 或 TypeScript 示例作为参考实现

5. **If user needs cURL/raw HTTP examples**, read from `curl/`.
   **如果用户需要 cURL/原始 HTTP 示例**，从 `curl/` 读取。

### Language-Specific Feature Support / 各语言功能支持

Every SDK language above supports both the beta Tool Runner and Managed Agents (beta) - Python (`@beta_tool` decorator), TypeScript (`betaZodTool` + Zod), Java (annotated classes), Go (`BetaToolRunner` in the `toolrunner` pkg), Ruby (`BaseTool` + `tool_runner`), C# (`BetaToolRunner` + raw JSON schema), PHP (`BetaRunnableTool` + `toolRunner()`); code entry points are in the Tool Use Patterns quick reference below. cURL is raw HTTP (no SDK features) and supports Managed Agents.

上述每种 SDK 语言都同时支持 beta 版 Tool Runner 与 Managed Agents（beta）——Python（`@beta_tool` 装饰器）、TypeScript（`betaZodTool` + Zod）、Java（注解类）、Go（`toolrunner` 包中的 `BetaToolRunner`）、Ruby（`BaseTool` + `tool_runner`）、C#（`BetaToolRunner` + 原始 JSON schema）、PHP（`BetaRunnableTool` + `toolRunner()`）；代码入口见下方的 Tool Use Patterns 速查表。cURL 属于原始 HTTP（无 SDK 特性），但也支持 Managed Agents。

> **Managed Agents code examples**: see the reading guide in the `## Managed Agents (Beta)` section below.

> **Managed Agents 代码示例**：见下方 `## Managed Agents (Beta)` 章节中的阅读指南。

---

## Which Surface Should I Use? / 我该选用哪种接入层？

> **Start simple.** Default to the simplest tier that meets your needs. Single API calls and workflows handle most use cases - only reach for agents when the task genuinely requires open-ended, model-driven exploration. "Simplest" means the least code you own: for a hosted, scheduled, or memory-backed agent, Managed Agents is usually the simplest option (no loop code, no state files, no scheduler), even though it's a bigger platform.

> **从简单开始。** 默认选择能满足需求的最简层级。单次 API 调用与工作流即可覆盖大多数用例——只有当任务确实需要开放式、模型驱动的探索时才诉诸智能体。"最简"指的是你自己所拥有的代码量最少：对于托管型、定时型或带记忆的智能体，Managed Agents 通常是最简单的选项（无需循环代码、无需状态文件、无需调度器），尽管它是一个更庞大的平台。

| Use Case                                        | Tier            | Recommended Surface       | Why                                                          |
| ----------------------------------------------- | --------------- | ------------------------- | ------------------------------------------------------------ |
| Classification, summarization, extraction, Q&A  | Single LLM call | **Claude API**            | One request, one response                                    |
| Batch processing or embeddings                  | Single LLM call | **Claude API**            | Specialized endpoints                                        |
| Multi-step pipelines with code-controlled logic | Workflow        | **Claude API + tool use** | You orchestrate the loop                                     |
| Custom agent with your own tools                | Agent           | **Claude API + tool use** | Maximum flexibility                                          |
| Server-managed stateful agent with workspace    | Agent           | **Managed Agents**        | Anthropic runs the loop and hosts the tool-execution sandbox |
| Persisted, versioned agent configs              | Agent           | **Managed Agents**        | Agents are stored objects; sessions pin to a version         |
| Long-running multi-turn agent with file mounts  | Agent           | **Managed Agents**        | Per-session containers, SSE event stream, Skills + MCP       |
| Agent that runs on a schedule (cron, "every night") | Agent       | **Managed Agents** - scheduled deployments | Deployments fire sessions autonomously; no client-side scheduler |
| Agent work that must meet a quality bar ("until it's right") | Agent | **Managed Agents** - outcomes | A separate grader iterates the agent against your rubric until it passes |

| 用例 | 层级 | 推荐接入层 | 原因 |
| --- | --- | --- | --- |
| 分类、摘要、抽取、问答 | 单次 LLM 调用 | **Claude API** | 一次请求，一次响应 |
| 批处理或向量化（embeddings） | 单次 LLM 调用 | **Claude API** | 有专门的端点 |
| 由代码控制逻辑的多步流水线 | 工作流 | **Claude API + tool use** | 循环由你来编排 |
| 使用自有工具的自定义智能体 | 智能体 | **Claude API + tool use** | 灵活性最大 |
| 由服务器管理的带工作区有状态智能体 | 智能体 | **Managed Agents** | Anthropic 负责运行循环并托管工具执行沙箱 |
| 需要持久化、版本化的智能体配置 | 智能体 | **Managed Agents** | 智能体是存储对象；会话固定到某个版本 |
| 长时运行、多轮、带文件挂载的智能体 | 智能体 | **Managed Agents** | 每会话独立容器、SSE 事件流、Skills + MCP |
| 按计划运行的智能体（cron、"每晚"） | 智能体 | **Managed Agents**——scheduled deployments | 部署自主触发会话；无需客户端调度器 |
| 必须达到质量标准的智能体工作（"直到做对为止"） | 智能体 | **Managed Agents**——outcomes | 由独立的评分器按你的评分规则迭代该智能体，直至通过 |

> **Note:** Managed Agents is the right choice when you want Anthropic to run the agent loop *and* host the container where tools execute - file ops, bash, code execution all run in the per-session workspace. If you want to host the compute yourself or run your own custom tool runtime, Claude API + tool use is the right choice - use the tool runner for the agentic loop - its per-turn hooks still give you approval gates, logging, error interception, and conditional execution (see `shared/tool-use-concepts.md`) - or the manual loop when you want to own the entire loop yourself.

> **注意：** 当你既想让 Anthropic 运行智能体循环，*又*想让它托管工具执行的容器时，Managed Agents 是正确选择——文件操作、bash、代码执行都在每会话的工作区内运行。如果你想自己托管算力或运行自己的自定义工具运行时，Claude API + tool use 才是正确选择——智能体循环可使用 tool runner——其每轮钩子仍能为你提供审批门、日志、错误拦截与条件执行（见 `shared/tool-use-concepts.md`）——若想完全自己掌控整个循环，则用手写循环。

> **Cloud-provider access.** **Claude Platform on AWS** is Anthropic-operated with same-day API parity - see `shared/claude-platform-on-aws.md` for client setup. For per-feature availability on **Claude Platform on AWS**, **Amazon Bedrock**, **Google Vertex AI**, and **Microsoft Foundry**, see `shared/platform-availability.md` - that table is the single source of truth in this skill; do not infer availability from anywhere else.

> **云厂商接入。** **Claude Platform on AWS** 由 Anthropic 运营，API 与官方当天保持一致——客户端配置见 `shared/claude-platform-on-aws.md`。各功能在 **Claude Platform on AWS**、**Amazon Bedrock**、**Google Vertex AI** 与 **Microsoft Foundry** 上的可用性，见 `shared/platform-availability.md`——该表是本技能中唯一的事实来源；不要从其他任何地方推断可用性。

### Building an Agent: Four Approaches / 构建智能体：四种方法

Once you've decided you actually need an agent (open-ended, model-driven tool use), there are four distinct ways to build one. Two independent questions separate them: **who supplies the harness** (the agent loop + context management) and **who supplies the deployment** (the infra the agent runs on). The Tool Runner and the Claude Agent SDK both supply a *harness only* - you still host and deploy them yourself - which is why they're easy to conflate. Managed Agents (CMA) is the only option that supplies **both** the harness *and* managed deployment; the manual loop supplies neither.

一旦确定确实需要智能体（开放式、模型驱动的工具使用），就有四种截然不同的构建方式。两个独立的问题将它们区分开：**由谁提供执行框架**（harness，即智能体循环 + 上下文管理）与**由谁提供部署**（智能体运行所在的基础设施）。Tool Runner 与 Claude Agent SDK 都只提供*执行框架*——你仍需自行托管和部署——这正是二者容易被混淆的原因。Managed Agents（CMA）是唯一同时提供执行框架*和*托管部署的选项；手写循环则两者都不提供。

| # | Approach | You write | Harness & deployment | Tools available | Use when |
|---|----------|-----------|----------------------|-----------------|----------|
| 1 | **Claude API - manual loop** | The `while stop_reason == "tool_use"` loop yourself | You build the harness; you host | Only tools you define | You want to own the *entire* loop - no beta dependency, or a control flow the Tool Runner's per-turn hooks don't fit |
| 2 | **Claude API - Tool Runner** (`client.beta.messages.tool_runner` + `@beta_tool` / `betaZodTool`) | Just the tool functions | SDK supplies the loop (**harness only**); you host | Only tools you define | A custom-tool agent without hand-writing the loop (most cases). Per-turn hooks still give you approval gates, error interception, result modification (e.g. `cache_control`), retries, streaming, and compaction |
| 3 | **Managed Agents** (REST, beta) | Agent config + your tool results | Anthropic supplies the harness **and** hosts a per-session sandbox (**harness + deployment**) | Anthropic-hosted sandbox (bash, files, code exec) + Skills/MCP + your tools | You want Anthropic to run the loop *and* host the per-session workspace; persisted/versioned configs; long-running sessions |
| 4 | **Claude Agent SDK** - *separate product* (`claude-agent-sdk` / `@anthropic-ai/claude-agent-sdk`) | A prompt + options | SDK supplies the Claude Code harness + built-in tools (**harness only**); you host | Built-in Read/Write/Edit/Bash/Glob/Grep/WebSearch/WebFetch + MCP + subagents | You want a batteries-included coding/filesystem agent running on your own infra |

| # | 方法 | 你要写什么 | 执行框架与部署 | 可用工具 | 适用场景 |
|---|----------|-----------|----------------------|-----------------|----------|
| 1 | **Claude API——手写循环** | 自行编写 `while stop_reason == "tool_use"` 循环 | 执行框架由你构建；托管也由你负责 | 仅限你定义的工具 | 你想*完全掌控*整个循环——不依赖 beta，或 Tool Runner 的每轮钩子无法满足你的控制流 |
| 2 | **Claude API——Tool Runner**（`client.beta.messages.tool_runner` + `@beta_tool` / `betaZodTool`） | 只写工具函数 | SDK 提供循环（**仅执行框架**）；托管由你负责 | 仅限你定义的工具 | 无需手写循环的自定义工具智能体（多数情况）。每轮钩子仍提供审批门、错误拦截、结果修改（如 `cache_control`）、重试、流式与压缩 |
| 3 | **Managed Agents**（REST，beta） | 智能体配置 + 你的工具结果 | Anthropic 提供执行框架**并**托管每会话沙箱（**框架 + 部署**） | Anthropic 托管的沙箱（bash、文件、代码执行）+ Skills/MCP + 你的工具 | 你想让 Anthropic 既运行循环*又*托管每会话工作区；需要持久化/版本化配置；长时运行会话 |
| 4 | **Claude Agent SDK**——*独立产品*（`claude-agent-sdk` / `@anthropic-ai/claude-agent-sdk`） | 一段提示词 + 选项 | SDK 提供 Claude Code 执行框架 + 内置工具（**仅执行框架**）；托管由你负责 | 内置 Read/Write/Edit/Bash/Glob/Grep/WebSearch/WebFetch + MCP + 子智能体 | 你想要一个开箱即用的编码/文件系统智能体跑在自己的基础设施上 |

The harness/deployment split is the key mental model: options 1, 2, and 4 all **leave deployment to you**; only option 3 (CMA) adds managed deployment. Options 1-3 are what this skill generates; option 4 is a different library with its own docs - see the disambiguation below.

执行框架/部署的切分是关键的思维模型：选项 1、2、4 都**把部署留给你**；只有选项 3（CMA）增加了托管部署。本技能生成的是选项 1-3；选项 4 是另一个拥有独立文档的库——见下文的辨析。

> **Tool Runner != Claude Agent SDK.** These sound alike but are different packages:  
> **Tool Runner != Claude Agent SDK。** 二者名字相似，但属于不同的包：  
> - **Tool Runner** is part of the regular Anthropic API SDK (`anthropic` / `@anthropic-ai/sdk`), reached via `client.beta.messages.tool_runner`. It automates the request -> execute -> loop cycle *for tools you define*. No built-in tools, no filesystem access, no sandbox - you supply every tool and host the compute. It is option 2 above, a thin helper over `POST /v1/messages`.  
> - **Tool Runner** 属于常规 Anthropic API SDK（`anthropic` / `@anthropic-ai/sdk`）的一部分，通过 `client.beta.messages.tool_runner` 使用。它针对*你定义的工具*自动完成 请求 -> 执行 -> 循环 的周期。没有内置工具、没有文件系统访问、没有沙箱——每个工具都由你提供，算力由你托管。即上面的选项 2，是 `POST /v1/messages` 之上的一层薄封装。  
> - **Claude Agent SDK** (`claude-agent-sdk` / `@anthropic-ai/claude-agent-sdk`) is Claude Code packaged as a library. It ships built-in tools (file read/write/edit, bash, grep, web search), the full agent loop, context management, hooks, subagents, permissions, and sessions. You call `query(prompt, options)` and it drives everything.  
> - **Claude Agent SDK**（`claude-agent-sdk` / `@anthropic-ai/claude-agent-sdk`）是以库形式打包的 Claude Code。它自带内置工具（文件读/写/编辑、bash、grep、网络搜索）、完整的智能体循环、上下文管理、钩子、子智能体、权限与会话。你只需调用 `query(prompt, options)`，其余全部由它驱动。  
>  
> Both are **harness-only - you host and deploy them.** The difference is scope of harness: the Tool Runner loops over tools *you* define (with per-turn hooks for approval, interception, result modification, and retries - but no built-in tools); the Agent SDK is the full Claude Code harness with built-in tools. Neither provides managed deployment - that's what **Managed Agents (CMA)** adds (Anthropic hosts the loop and a per-session sandbox).  
> 两者都**只提供执行框架——托管与部署都由你负责。**区别在于框架的范围：Tool Runner 只循环执行*你*定义的工具（带有用于审批、拦截、结果修改与重试的每轮钩子——但没有内置工具）；Agent SDK 则是带内置工具的完整 Claude Code 框架。两者都不提供托管部署——这正是 **Managed Agents（CMA）**补充的部分（Anthropic 托管循环与每会话沙箱）。  
>  
> **This skill covers the Claude API and Managed Agents (options 1-3); it does not generate Claude Agent SDK code.** If the user actually wants the Claude Agent SDK, point them to its docs (`code.claude.com/docs/en/agent-sdk`) - don't substitute the API Tool Runner for it, or vice-versa.
> **本技能覆盖 Claude API 与 Managed Agents（选项 1-3）；不生成 Claude Agent SDK 代码。**如果用户实际想要的是 Claude Agent SDK，请引导其查阅该文档（`code.claude.com/docs/en/agent-sdk`）——不要用 API Tool Runner 替代它，反之亦然。

### Should I Build an Agent? / 我该构建智能体吗？

Before choosing the agent tier, check all four criteria:

在选择智能体层级之前，先核对以下四项标准：

- **Complexity** - Is the task multi-step and hard to fully specify in advance? (e.g., "turn this design doc into a PR" vs. "extract the title from this PDF")
  **复杂度**——任务是否多步骤且难以事先完整规定？（例如："把这份设计文档变成一个 PR" 对比 "从这份 PDF 中提取标题"）
- **Value** - Does the outcome justify higher cost and latency?
  **价值**——产出是否足以证明更高的成本与延迟是值得的？
- **Viability** - Is Claude capable at this task type?
  **可行性**——Claude 在这类任务上是否有足够能力？
- **Cost of error** - Can errors be caught and recovered from? (tests, review, rollback)
  **错误代价**——错误能否被发现并从中恢复？（测试、评审、回滚）

If the answer is "no" to any of these, stay at a simpler tier (single call or workflow).

只要其中任何一项答案为"否"，就停留在更简单的层级（单次调用或工作流）。

---

## Architecture / 架构

Everything goes through `POST /v1/messages`. Tools and output constraints are features of this single endpoint - not separate APIs.

一切都经由 `POST /v1/messages`。工具与输出约束都是这一个端点的特性——并非独立的 API。

**User-defined tools** - You define tools (via decorators, Zod schemas, or raw JSON), and the SDK's tool runner handles calling the API, executing your functions, and looping until Claude is done. For full control, you can write the loop manually.

**用户自定义工具**——你定义工具（通过装饰器、Zod schema 或原始 JSON），SDK 的 tool runner 负责调用 API、执行你的函数并循环直到 Claude 完成。如需完全控制，可以手写该循环。

**Server-side tools** - Anthropic-hosted tools that run on Anthropic's infrastructure. Code execution is fully server-side (declare it in `tools`, Claude runs code automatically). Computer use can be server-hosted or self-hosted.

**服务器端工具**——由 Anthropic 托管、运行在 Anthropic 基础设施上的工具。代码执行完全在服务器端进行（在 `tools` 中声明即可，Claude 会自动运行代码）。计算机使用（computer use）可以由服务器托管，也可以自行托管。

**Structured outputs** - Constrains the Messages API response format (`output_config.format`) and/or tool parameter validation (`strict: true`). The recommended approach is `client.messages.parse()` which validates responses against your schema automatically. Note: the old `output_format` parameter is deprecated; use `output_config: {format: {...}}` on `messages.create()`.

**结构化输出**——约束 Messages API 的响应格式（`output_config.format`）和/或工具参数校验（`strict: true`）。推荐使用 `client.messages.parse()`，它会自动按你的 schema 校验响应。注意：旧的 `output_format` 参数已弃用；请在 `messages.create()` 上使用 `output_config: {format: {...}}`。

**Supporting endpoints** - Batches (`POST /v1/messages/batches`), Files (`POST /v1/files`), Token Counting (`POST /v1/messages/count_tokens` - see `shared/token-counting.md`), and Models (`GET /v1/models`, `GET /v1/models/{id}` - live capability/context-window discovery) feed into or support Messages API requests.

**辅助端点**——批处理（`POST /v1/messages/batches`）、文件（`POST /v1/files`）、token 计数（`POST /v1/messages/count_tokens`——见 `shared/token-counting.md`）与模型（`GET /v1/models`、`GET /v1/models/{id}`——实时能力/上下文窗口发现）为 Messages API 请求提供输入或支撑。

---

## Current Models (cached: 2026-09-25) / 当前模型（缓存于 2026-09-25）

| Model             | Model ID            | Context        | Input $/1M | Output $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
| Claude Fable 5.1    | `claude-fable-5-1`      | 1M             | $10.00     | $50.00      |
| Claude Mythos 5.1 (Project Glasswing only) | `claude-mythos-5-1` | 1M | $10.00     | $50.00      |
| Claude Fable 5 | `claude-fable-5` | 1M             | $10.00     | $50.00      |
| Claude Opus 5.5 | `claude-opus-5-5` | 1M | $4.00 | $20.00 |
| Claude Opus 5     | `claude-opus-5`       | 1M             | $5.00      | $25.00      |
| Claude Opus 4.8 | `claude-opus-4-8`  | 1M             | $5.00      | $25.00      |
| Claude Opus 4.7   | `claude-opus-4-7`   | 1M             | $5.00      | $25.00      |
| Claude Opus 4.6   | `claude-opus-4-6`   | 1M             | $5.00      | $25.00      |
| Claude Sonnet 5.5 | `claude-sonnet-5-5` | 1M | $2.00 | $10.00 |
| Claude Sonnet 5   | `claude-sonnet-5`   | 1M             | $2.00      | $10.00      |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M             | $3.00      | $15.00      |
| Claude Haiku 4.5  | `claude-haiku-4-5`  | 200K           | $1.00      | $5.00       |

| 模型 | 模型 ID | 上下文 | 输入 $/1M | 输出 $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
| Claude Fable 5.1    | `claude-fable-5-1`      | 1M             | $10.00     | $50.00      |
| Claude Mythos 5.1（仅限 Project Glasswing） | `claude-mythos-5-1` | 1M | $10.00     | $50.00      |
| Claude Fable 5 | `claude-fable-5` | 1M             | $10.00     | $50.00      |
| Claude Opus 5.5 | `claude-opus-5-5` | 1M | $4.00 | $20.00 |
| Claude Opus 5     | `claude-opus-5`       | 1M             | $5.00      | $25.00      |
| Claude Opus 4.8 | `claude-opus-4-8`  | 1M             | $5.00      | $25.00      |
| Claude Opus 4.7   | `claude-opus-4-7`   | 1M             | $5.00      | $25.00      |
| Claude Opus 4.6   | `claude-opus-4-6`   | 1M             | $5.00      | $25.00      |
| Claude Sonnet 5.5 | `claude-sonnet-5-5` | 1M | $2.00 | $10.00 |
| Claude Sonnet 5   | `claude-sonnet-5`   | 1M             | $2.00      | $10.00      |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M             | $3.00      | $15.00      |
| Claude Haiku 4.5  | `claude-haiku-4-5`  | 200K           | $1.00      | $5.00       |

**Partner pricing:** The prices above are Anthropic first-party API rates - they also apply to Claude on Microsoft Foundry, which is billed through the Microsoft Marketplace at standard API rates. Claude on Amazon Bedrock and Vertex AI is partner-operated with separate pricing - see [Bedrock](https://aws.amazon.com/bedrock/pricing/) or [Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/pricing#claude-models). For WebFetch, use the Pricing row in `shared/live-sources.md`.

**合作伙伴定价：**上述价格为 Anthropic 第一方 API 费率——同样适用于 Microsoft Foundry 上的 Claude，后者通过 Microsoft Marketplace 按标准 API 费率计费。Amazon Bedrock 与 Vertex AI 上的 Claude 由合作伙伴运营，定价独立——见 [Bedrock](https://aws.amazon.com/bedrock/pricing/) 或 [Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/pricing#claude-models)。如需 WebFetch，请使用 `shared/live-sources.md` 中的 Pricing 行。

**ALWAYS use `claude-opus-5-5` unless the user explicitly names a different model.** This is non-negotiable. Do not use `claude-sonnet-5-5`, `claude-sonnet-5`, or any other model unless the user literally says "use sonnet" or "use haiku". Never downgrade for cost - that's the user's decision, not yours. A request that describes a Sonnet by attribute ("cheapest Sonnet", "cheaper Sonnet", "newest Sonnet", "latest Sonnet") resolves to `claude-sonnet-5-5`. Where a second, cheaper model is in play alongside the main one (worker or sub-agent threads, bulk extractors, LLM judges, the executor under an advisor) - because the user asked for one or a guide in this skill calls for it - or the user says "sonnet" or "haiku" without a version, that means the current generation from the table above (`claude-sonnet-5-5`, `claude-haiku-4-5`); previous-generation IDs such as `claude-sonnet-5` are only for users who name that version. Use `claude-fable-5-1` only when the user explicitly asks for Claude Fable 5.1, "fable", or Anthropic's most capable model - it has different API behavior than the Opus family (see below) and pricing that exceeds Opus-tier. **Use only the exact model ID strings from the table - they are complete as-is; never append date suffixes** (`claude-opus-5-5`, never `claude-opus-5-5-20260401` or any other date-suffixed variant you might recall from training data). If the user requests an older model not in the table (e.g., "opus 4.5", "sonnet 3.7"), read `shared/models.md` for the exact ID - do not construct one yourself.

**除非用户明确指名其他模型，否则一律使用 `claude-opus-5-5`。**这一点没有商量余地。除非用户字面上说出 "use sonnet" 或 "use haiku"，否则不要使用 `claude-sonnet-5-5`、`claude-sonnet-5` 或任何其他模型。绝不为了省钱而降级——那是用户的决定，不是你的。按属性描述 Sonnet 的请求（"cheapest Sonnet"、"cheaper Sonnet"、"newest Sonnet"、"latest Sonnet"）解析为 `claude-sonnet-5-5`。当主模型之外还要用第二个更便宜的模型（worker 或子智能体线程、批量抽取器、LLM 评审、advisor 之下的执行器）——因为用户要求，或本技能中某指南有此要求——或者用户未带版本号地说 "sonnet" 或 "haiku"，那指的是上表中的当前世代（`claude-sonnet-5-5`、`claude-haiku-4-5`）；`claude-sonnet-5` 这类上一代 ID 仅用于点名该版本的用户。只有当用户明确要求 Claude Fable 5.1、"fable" 或 Anthropic 能力最强的模型时才使用 `claude-fable-5-1`——它的 API 行为与 Opus 家族不同（见下文），且定价高于 Opus 档。**只使用表中精确的模型 ID 字符串——它们本身就完整；绝不要追加日期后缀**（用 `claude-opus-5-5`，绝不要用 `claude-opus-5-5-20260401` 或你从训练数据中回忆起的任何其他带日期后缀的变体）。如果用户要求表中没有的旧模型（如 "opus 4.5"、"sonnet 3.7"），请阅读 `shared/models.md` 获取确切 ID——不要自行拼造。

【评论】"非商量的默认模型 + 禁止日期后缀"是针对模型幻觉与训练数据过时的双重防御：既锁定默认选择权，又阻止模型从记忆中补全带日期的 ID。

### Claude Fable 5.1 (`claude-fable-5-1`) - most capable widely released model / Claude Fable 5.1（`claude-fable-5-1`）——能力最强的大范围发布模型

Claude Fable 5.1 is Anthropic's most capable widely released model, for the most demanding reasoning and long-horizon agentic work; everything below also applies to **Claude Mythos 5.1** (`claude-mythos-5-1`, Project Glasswing - same capabilities, pricing, and API surface; it runs safeguards that depend on the access program, so the `refusal` handling below applies there too; successor to Claude Mythos 5, which ran no safety classifiers). 1M context window (the maximum is also the default), 128K max output. Key API differences from Opus-tier - see `shared/model-migration.md` -> Migrating to Claude Fable 5.1 for details:

Claude Fable 5.1 是 Anthropic 能力最强的大范围发布模型，面向要求最高的推理与长程智能体工作；下文所有内容同样适用于 **Claude Mythos 5.1**（`claude-mythos-5-1`，Project Glasswing——能力、定价与 API 表面相同；它运行依赖接入计划的安全防护，因此下文的 `refusal` 处理同样适用于它；是 Claude Mythos 5 的后继者，后者不运行任何安全分类器）。1M 上下文窗口（最大值即默认值），128K 最大输出。与 Opus 档的关键 API 差异——详情见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1：

- **Thinking is always on** - omit the `thinking` parameter entirely (or send `{type: "adaptive"}`). Any other explicit configuration is rejected: `{type: "disabled"}` and `{type: "enabled", budget_tokens: N}` both return a 400. Control depth with `output_config.effort` (supports `low` through `xhigh` and `max`).
  **思考始终开启**——完全省略 `thinking` 参数（或发送 `{type: "adaptive"}`）。任何其他显式配置都会被拒绝：`{type: "disabled"}` 与 `{type: "enabled", budget_tokens: N}` 均返回 400。通过 `output_config.effort` 控制深度（支持 `low` 至 `xhigh` 与 `max`）。
- **The raw chain of thought is never returned** - responses carry regular `thinking` blocks (not `redacted_thinking`): `display: "summarized"` returns a readable summary, `"omitted"` (the default) leaves the `thinking` field as an empty string. Replay rules: pass thinking blocks back unchanged on the same model; other models drop them silently (unbilled - nothing to strip; Claude Mythos 5.1 instead reads them); details in `shared/model-migration.md`.
  **原始思维链永不返回**——响应携带常规 `thinking` 块（而非 `redacted_thinking`）：`display: "summarized"` 返回可读摘要，`"omitted"`（默认值）则把 `thinking` 字段留为空字符串。回传规则：在同一模型上原样传回思考块；其他模型会静默丢弃它们（不计费——无需剥离；Claude Mythos 5.1 则会读取它们）；详情见 `shared/model-migration.md`。
- **Tokenizer** - same tokenizer as Opus 4.8 (introduced with Opus 4.7). Token counts are roughly unchanged when migrating from Opus 4.7/4.8; per-token pricing differs. Coming from Opus 4.6, Sonnet, Haiku, or older, re-baseline with `count_tokens` (the Opus 4.7 tokenizer uses ~1×-1.35× as many tokens).
  **分词器**——与 Opus 4.8 相同（随 Opus 4.7 引入）。从 Opus 4.7/4.8 迁移时 token 数大致不变；每 token 定价不同。来自 Opus 4.6、Sonnet、Haiku 或更早模型时，请用 `count_tokens` 重新建立基线（Opus 4.7 分词器使用的 token 数约为原来的 1×-1.35×）。
- **`refusal` stop reason - handle it, and opt into fallbacks by default** - safety classifiers may decline a request (HTTP 200, `stop_reason: "refusal"`, with a `stop_details` category); always check `stop_reason` before reading `content`. **When you write `claude-fable-5-1`, `claude-opus-5-5`, `claude-opus-5`, or `claude-sonnet-5-5` code, include the server-side `fallbacks` parameter by default** (for `claude-sonnet-5-5`, only the `"default"` form and only on the Claude API; on other platforms use the SDK middleware below, except when the request sends `between_tools`: only Claude Sonnet 5.5 accepts it and the middleware re-sends the same request body on the fallback model, so write the retry yourself and send it without `between_tools` - see `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5 -> Safeguards and fallback). Simplest form: `betas: ["server-side-fallback-2026-07-01"]` + `fallbacks: "default"`, which routes by refusal category so you never maintain a model list. (The older array form - `betas: ["server-side-fallback-2026-06-01"]` + `fallbacks: [{"model": "claude-opus-4-8"}]` - still works; Claude API and Claude Platform on AWS - on Bedrock, Vertex and Foundry, use the SDKs' client-side `BetaRefusalFallbackMiddleware` + `BetaFallbackState`). Tell the user you've enabled it; drop it only if they decline. Full semantics (billing, mid-stream refusals, credit repricing) in `shared/model-migration.md` -> refusal section. **Per-language code examples in `{lang}/claude-api/README.md` § Refusal Fallbacks cover the array form only** - for the `"default"` mode, follow the raw-HTTP shape in `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features and swap `fallbacks: [{...}]` for `fallbacks: "default"` plus the `-2026-07-01` header; the rest of the request is unchanged.
  **`refusal` 停止原因——要处理它，并默认启用后备模型**——安全分类器可能拒绝某个请求（HTTP 200、`stop_reason: "refusal"`，并带 `stop_details` 类别）；读取 `content` 之前务必先检查 `stop_reason`。**编写 `claude-fable-5-1`、`claude-opus-5-5`、`claude-opus-5` 或 `claude-sonnet-5-5` 的代码时，默认包含服务器端 `fallbacks` 参数**（对 `claude-sonnet-5-5`，只用 `"default"` 形式且仅在 Claude API 上；其他平台使用下述 SDK 中间件，但请求发送 `between_tools` 时除外：只有 Claude Sonnet 5.5 接受它，而中间件会在后备模型上重发同一请求体，因此请自行编写重试逻辑并发送不带 `between_tools` 的请求——见 `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5 -> Safeguards and fallback）。最简形式：`betas: ["server-side-fallback-2026-07-01"]` + `fallbacks: "default"`，它按拒绝类别路由，因此你无需维护模型清单。（较旧的数组形式——`betas: ["server-side-fallback-2026-06-01"]` + `fallbacks: [{"model": "claude-opus-4-8"}]`——仍然有效；Claude API 与 Claude Platform on AWS 之上——在 Bedrock、Vertex 与 Foundry 上，使用 SDK 的客户端 `BetaRefusalFallbackMiddleware` + `BetaFallbackState`）。请告知用户你已启用该参数；仅当用户拒绝时才移除。完整语义（计费、流中拒绝、额度重定价）见 `shared/model-migration.md` -> refusal 一节。**`{lang}/claude-api/README.md` § Refusal Fallbacks 中的各语言代码示例仅覆盖数组形式**——对于 `"default"` 模式，请遵循 `shared/model-migration.md` -> Migrating to Claude Opus 5 -> New API features 中的原始 HTTP 形态，并把 `fallbacks: [{...}]` 换成 `fallbacks: "default"` 加上 `-2026-07-01` 头；请求其余部分不变。
- **No assistant prefill** - same as the rest of the 4.6+ family.
  **不支持 assistant 预填充**——与 4.6+ 家族其余模型相同。
- **30-day data retention required** - Claude Fable 5.1 is not available under zero data retention unless expressly authorized by Anthropic; requests from an org whose retention configuration doesn't meet the requirement return `400 invalid_request_error`.
  **要求 30 天数据保留**——除非获得 Anthropic 明确授权，否则 Claude Fable 5.1 不支持零数据保留；保留配置不满足要求的组织发起的请求会返回 `400 invalid_request_error`。
- **Longer turns, different prompting** - single requests on hard tasks can run many minutes (plan timeouts/streaming/progress UX); effort sweeps should include low/medium for routine work; prompts written for prior models are often too prescriptive and reduce output quality. See `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> Behavioral shifts (prompt-tunable) for the recommended prompt snippets.
  **单轮更长，提示方式不同**——困难任务上的单次请求可能运行数分钟（请规划超时/流式/进度 UX）；力度扫描应包含面向常规工作的 low/medium；为旧模型撰写的提示词往往过于指令化并会降低输出质量。推荐的提示词片段见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> Behavioral shifts (prompt-tunable)。
- **Successor to Claude Fable 5 (`claude-fable-5`, still served) in the same tier at the same per-token price.** Same surface as Claude Fable 5 with three breaking changes - forced tool use (`tool_choice` `any` / `tool`) returns a 400 (use `auto` + a prompt instruction, `strict: true` for schema-valid arguments, or structured outputs); thinking blocks are bound to the producing model (other models drop them, unbilled); and editing earlier turns invalidates thinking blocks ("preserved thinking"; new accounts created on/after 2026-08-31 get a 400 on edited history on every platform, and enforcement scope is decided per model, and Claude Mythos 5.1 doesn't run this check. Make every harness append-only and run the three-step check; the opt-in controls beta is on the Claude API, Claude Platform on AWS, Bedrock, and Vertex - Foundry unconfirmed, see `shared/platform-availability.md`) - plus per-message `effort` (beta `mid-conversation-output-config-2026-07-01`, also on Claude Opus 5 and Claude Opus 5.5), turn-scoped `clear_at: "next_user_message"` system messages (beta), `thinking.display: "updates"` progress notes (beta, all platforms), cache reads at $0.25/MTok, and content provenance. Covered Model - ZDR orgs get `400 invalid_request_error` as on Claude Fable 5 (ZDR only if expressly authorized by Anthropic); no Priority Tier. Same tokenizer as Claude Fable 5. See `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5.
  **同为 Claude Fable 5（`claude-fable-5`，仍在服务）同档、同每 token 单价的后继者。**API 表面与 Claude Fable 5 相同，但有三个破坏性变更——强制工具使用（`tool_choice` `any` / `tool`）返回 400（改用 `auto` + 提示词指令、以 `strict: true` 保证参数符合 schema，或使用结构化输出）；思考块与产生它的模型绑定（其他模型静默丢弃，不计费）；以及编辑较早轮次会使思考块失效（"preserved thinking"；2026-08-31 及之后创建的新账号在每个平台上编辑历史都会得到 400，强制范围按模型决定，且 Claude Mythos 5.1 不运行该检查。让每个执行框架都只追加，并运行三步检查；可选控制的 beta 在 Claude API、Claude Platform on AWS、Bedrock 与 Vertex 上可用——Foundry 未确认，见 `shared/platform-availability.md`）——另有每消息 `effort`（beta `mid-conversation-output-config-2026-07-01`，也支持 Claude Opus 5 与 Claude Opus 5.5）、轮次作用域的 `clear_at: "next_user_message"` 系统消息（beta）、`thinking.display: "updates"` 进度说明（beta，所有平台）、$0.25/MTok 的缓存读取，以及内容溯源。Covered Model——ZDR 组织与 Claude Fable 5 一样得到 `400 invalid_request_error`（ZDR 仅在 Anthropic 明确授权时可用）；无 Priority Tier。分词器与 Claude Fable 5 相同。见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5。

### Claude Opus 5.5 (`claude-opus-5-5`) - the current Opus and the default model / Claude Opus 5.5（`claude-opus-5-5`）——当前 Opus 与默认模型

Successor to Claude Opus 5 in the Opus line at a lower price ($4 / $20 per MTok, cache reads $0.20), same 1M context / 128K output / tokenizer / feature set. Four breaking changes for code running on Claude Opus 5: **thinking can't be disabled** (`{type: "disabled"}` and `budget_tokens` both 400 at every effort level - effort is the only control, and its **default is `medium`**, one level below Claude Opus 5's `high`, so set it explicitly); **forced `tool_choice` `any`/`tool` returns a 400** (use `auto` + `strict: true` and steer from the prompt, or structured outputs); **thinking blocks are tied to the model and the conversation** (preserved thinking: only Claude Fable 5.1 / Claude Mythos 5.1 on the Claude API read its blocks, so a fallback to Claude Opus 5 runs without them; accounts created on or after 2026-08-31 are enforced on the history-editing check); and **on the Claude API and Google Cloud, computer use only through `computer_toolset_20260801`** (`computer_20251124` 400s there; Amazon Bedrock still accepts it). Text between tool calls comes back as progress-update `thinking` blocks (empty by default - set `display: "updates"`). Broader safety classifiers: `bio` and `reasoning_extraction` join `cyber`. Fast mode is Claude API only, $8 / $40 per MTok (2x standard). See `shared/model-migration.md` -> Migrating to Claude Opus 5.5.

作为 Opus 线中 Claude Opus 5 的后继者，价格更低（每 MTok $4 / $20，缓存读取 $0.20），上下文/输出/分词器/功能集同为 1M / 128K / 相同 / 相同。对运行在 Claude Opus 5 上的代码而言有四个破坏性变更：**思考无法关闭**（`{type: "disabled"}` 与 `budget_tokens` 在每个 effort 级别都返回 400——effort 是唯一的控制手段，且其**默认值为 `medium`**，比 Claude Opus 5 的 `high` 低一级，因此请显式设置）；**强制 `tool_choice` `any`/`tool` 返回 400**（改用 `auto` + `strict: true` 并通过提示词引导，或使用结构化输出）；**思考块与模型和对话绑定**（preserved thinking：在 Claude API 上只有 Claude Fable 5.1 / Claude Mythos 5.1 能读取其思考块，因此后备到 Claude Opus 5 时将在没有思考块的情况下运行；2026-08-31 及之后创建的账号会强制执行历史编辑检查）；以及**在 Claude API 与 Google Cloud 上，计算机使用只能通过 `computer_toolset_20260801`**（`computer_20251124` 在那里返回 400；Amazon Bedrock 仍接受它）。工具调用之间的文本以进度更新型 `thinking` 块返回（默认为空——设置 `display: "updates"`）。安全分类器范围扩大：在 `cyber` 之外新增 `bio` 与 `reasoning_extraction`。快速模式仅限 Claude API，每 MTok $8 / $40（标准的 2 倍）。见 `shared/model-migration.md` -> Migrating to Claude Opus 5.5。

### Claude Sonnet 5.5 (`claude-sonnet-5-5`) - the current Sonnet: speed and capability for everyday coding, agent, and enterprise work (Claude Opus 5.5 stays the default) / Claude Sonnet 5.5（`claude-sonnet-5-5`）——当前 Sonnet：为日常编码、智能体与企业工作提供速度与能力（默认模型仍为 Claude Opus 5.5）

Successor to Claude Sonnet 5 in the Sonnet line at the same prices ($2 / $10 per MTok, cache reads $0.20), with the same tokenizer, 1M context and 128K output. Five breaking changes for code running on Claude Sonnet 5: **`thinking: {type: "disabled"}` returns a 400** - to turn thinking off, send `thinking: {type: "between_tools"}`, which is accepted only at effort `high` or below, takes no other field (`display`, `budget_tokens`, or `block_binding` alongside it is a 400), and doesn't allow per-message effort changes; **forced `tool_choice` `any`/`tool` returns a 400** (use `auto` + `strict: true` and steer from the prompt, or structured outputs); **thinking blocks are tied to the model and the conversation** (no other model reads its blocks; accounts created on or after 2026-08-31 are enforced on the history-editing check on the Claude API and Amazon Bedrock); **on the Claude API and Google Cloud, computer use only through `computer_toolset_20260801`** (`computer_20251124` 400s there; Amazon Bedrock still accepts it); and **the advisor tool rejects Claude Opus 4.8, Claude Opus 4.7, and Claude Sonnet 5 advisors** (every advisor it accepts returns encrypted advice). Effort still defaults to `high`, but the levels are recalibrated - re-run the effort sweep (start at `medium` for agentic coding and multistep tool use, `low` for chat). Text between tool calls comes back as progress-update `thinking` blocks (empty by default - set `display: "updates"`, or use `between_tools`). Safety classifiers decline in five `stop_details` categories: `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`. See `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5.

作为 Sonnet 线中 Claude Sonnet 5 的后继者，价格相同（每 MTok $2 / $10，缓存读取 $0.20），分词器相同，1M 上下文与 128K 输出。对运行在 Claude Sonnet 5 上的代码而言有五个破坏性变更：**`thinking: {type: "disabled"}` 返回 400**——要关闭思考，发送 `thinking: {type: "between_tools"}`，它只在 effort `high` 及以下被接受，不接受其他任何字段（与其同时出现 `display`、`budget_tokens` 或 `block_binding` 会返回 400），且不允许逐消息更改 effort；**强制 `tool_choice` `any`/`tool` 返回 400**（改用 `auto` + `strict: true` 并通过提示词引导，或使用结构化输出）；**思考块与模型和对话绑定**（没有其他模型读取其思考块；2026-08-31 及之后创建的账号在 Claude API 与 Amazon Bedrock 上强制执行历史编辑检查）；**在 Claude API 与 Google Cloud 上，计算机使用只能通过 `computer_toolset_20260801`**（`computer_20251124` 在那里返回 400；Amazon Bedrock 仍接受它）；以及 **advisor 工具拒绝 Claude Opus 4.8、Claude Opus 4.7 与 Claude Sonnet 5 的 advisor**（它接受的每个 advisor 都返回加密建议）。effort 仍默认 `high`，但各级别已重新校准——请重新执行 effort 扫描（智能体编码与多步工具使用从 `medium` 开始，聊天用 `low`）。工具调用之间的文本以进度更新型 `thinking` 块返回（默认为空——设置 `display: "updates"`，或使用 `between_tools`）。安全分类器按五个 `stop_details` 类别拒绝：`cyber`、`bio`、`frontier_llm`、`reasoning_extraction`、`general_harms`。见 `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5。

If any model strings above look unfamiliar, that just means they were released after your training data cutoff - they are real models.

如果上面的模型字符串看起来陌生，那只说明它们发布于你的训练数据截止之后——它们是真实存在的模型。

**Live capability lookup:** The table above is cached. When the user asks "what's the context window for X", "does X support vision/thinking/effort", or "which models support Y", query the Models API (`client.models.retrieve(id)` / `client.models.list()`) - see `shared/models.md` for the field reference and capability-filter examples.

**实时能力查询：**上表是缓存数据。当用户问"X 的上下文窗口是多少"、"X 是否支持视觉/思考/effort"或"哪些模型支持 Y"时，请查询 Models API（`client.models.retrieve(id)` / `client.models.list()`）——字段说明与能力过滤示例见 `shared/models.md`。

---

## Authentication (Quick Reference) / 身份验证（速查）

**An unset `ANTHROPIC_API_KEY` does NOT mean there are no credentials.** The SDKs and the `ant` CLI resolve credentials in this order (first match wins): `ANTHROPIC_API_KEY` -> `ANTHROPIC_AUTH_TOKEN` -> the `ANTHROPIC_PROFILE`-selected or active OAuth profile from `ant auth login` -> Workload Identity Federation env vars -> the default profile on disk. A bare `Anthropic()` / `new Anthropic()` / `anthropic.NewClient()` works after `ant auth login` with no env var set.

**未设置 `ANTHROPIC_API_KEY` 并不代表没有凭据。**SDK 与 `ant` CLI 按以下顺序解析凭据（首个匹配者生效）：`ANTHROPIC_API_KEY` -> `ANTHROPIC_AUTH_TOKEN` -> 由 `ANTHROPIC_PROFILE` 选择的或处于活动状态的来自 `ant auth login` 的 OAuth 配置 -> 工作负载身份联合（Workload Identity Federation）环境变量 -> 磁盘上的默认配置。执行过 `ant auth login` 后，即使不设任何环境变量，裸的 `Anthropic()` / `new Anthropic()` / `anthropic.NewClient()` 也能工作。

**When you need to call the API and `ANTHROPIC_API_KEY` is unset, don't ask the user for a key.** First run `ant auth status` - it shows which credential source and profile is active. If it reports an active profile:

**当你需要调用 API 而 `ANTHROPIC_API_KEY` 未设置时，不要向用户索要密钥。**先运行 `ant auth status`——它会显示当前活动的凭据来源与配置。如果它报告存在活动配置：

- **SDK code or `ant` CLI:** just run it. The zero-arg client constructor and every `ant ...` subcommand pick up the profile automatically - no env var needed.
  **SDK 代码或 `ant` CLI：**直接运行即可。零参数客户端构造函数与每个 `ant ...` 子命令都会自动读取该配置——无需环境变量。
- **Raw `curl` / HTTP:** get a short-lived token with `ant auth print-credentials --access-token` and send it as `Authorization: Bearer <token>` **plus** the header `anthropic-beta: oauth-2025-04-20` (OAuth tokens go on `Authorization: Bearer`, not `x-api-key:` - converting a curl from an API key is a header change, not a key swap). Always pass `--access-token`; the no-flag form prints JSON, not a bare token.
  **原始 `curl` / HTTP：**用 `ant auth print-credentials --access-token` 获取短期令牌，并作为 `Authorization: Bearer <token>` 发送，**同时**加上 `anthropic-beta: oauth-2025-04-20` 头（OAuth 令牌放在 `Authorization: Bearer` 上，而不是 `x-api-key:`——把 curl 从 API 密钥改过来是改头部，而不是换密钥）。务必传 `--access-token`；不带该旗标的形式输出的是 JSON，不是裸令牌。

Only ask the user for a key if `ant auth status` reports no active credential source (or `ant` itself isn't installed). Suggest `ant auth login` as the first option - it stores a profile under `~/.config/anthropic/` that the SDKs read automatically - and an exported `ANTHROPIC_API_KEY` as the alternative.

只有当 `ant auth status` 报告没有活动的凭据来源（或 `ant` 本身未安装）时才向用户索要密钥。首选建议 `ant auth login`——它会在 `~/.config/anthropic/` 下存储一个 SDK 会自动读取的配置——导出的 `ANTHROPIC_API_KEY` 作为备选。

Full auth details (named profiles, scopes, the API-key-shadows-profile trap, refresh-token expiry): `shared/anthropic-cli.md`.

完整的身份验证细节（命名配置、作用域、"API 密钥遮蔽配置"陷阱、刷新令牌过期）：`shared/anthropic-cli.md`。

---

## Thinking & Effort (Quick Reference) / 思考与力度（速查）

Use adaptive thinking (`thinking: {type: "adaptive"}`) on every current model except Haiku 4.5, which still takes `budget_tokens` (table below) - Claude dynamically decides when and how much to think. Per-model rules:

在除 Haiku 4.5 之外的每个当前模型上使用自适应思考（`thinking: {type: "adaptive"}`）；Haiku 4.5 仍接受 `budget_tokens`（见下表）——Claude 会动态决定何时思考、思考多少。各模型规则：

| Model | Thinking config | Omitting `thinking` | `budget_tokens` | Sampling (`temperature`/`top_p`/`top_k`) | Effort levels |
|---|---|---|---|---|---|
| Fable 5 / Claude Fable 5.1 (and the Mythos counterparts) | `{type: "adaptive"}` or omit; explicit `{type: "disabled"}` returns 400 - omit the param instead (Claude Fable 5.1 / Claude Mythos 5.1 also 400 on forced `tool_choice` `any`/`tool`; Claude Fable 5.1 runs preserved thinking's history-editing check on replayed thinking blocks, Claude Mythos 5.1 does not) | Runs adaptive (thinking is always on) | Removed - `{type: "enabled", budget_tokens: N}` returns 400 | Removed - 400 | `low`/`medium`/`high`/`xhigh`/`max` |
| Claude Opus 5.5 | `{type: "adaptive"}` or omit; `{type: "disabled"}` and `{type: "enabled", budget_tokens}` return 400 at **every** effort level - omit the param and lower effort instead (also 400s on forced `tool_choice` `any`/`tool`, and runs preserved thinking - see `shared/model-migration.md` -> Migrating to Claude Opus 5.5) | Runs **adaptive** | Removed - 400 | Removed - 400 | `low`/`medium`/`high`/`xhigh`/`max` - **default `medium`** (not `high`); per-message effort (beta) supported |
| Claude Opus 5 | `{type: "adaptive"}` or omit; `{type: "disabled"}` accepted **only at effort `high` or below** - 400 at `xhigh`/`max`, and see the disabled-thinking pitfall below | Runs **adaptive** (thinking is on by default - unlike Opus 4.8/4.7) | Removed - 400 | Removed - 400 | `low`-`max` (all five) |
| Opus 4.8 / 4.7 | `{type: "adaptive"}` is the only on-mode; `{type: "disabled"}` accepted | Runs **without** thinking - set `{type: "adaptive"}` explicitly | Removed - 400 | Removed - 400 | `low`/`medium`/`high`/`xhigh`/`max` |
| Claude Sonnet 5.5 | `{type: "adaptive"}` or omit; `{type: "disabled"}` returns 400 - to turn thinking off send `{type: "between_tools"}` (no other field; 400 at `xhigh`/`max`; effort can't change mid-conversation with it) (also 400s on forced `tool_choice` `any`/`tool`, and runs preserved thinking - see `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5) | Runs **adaptive** | Removed - 400 | Non-default values - 400 | `low`/`medium`/`high`/`xhigh`/`max` - default `high`, levels recalibrated from Claude Sonnet 5; per-message effort (beta) supported with thinking on |
| Sonnet 5 | `{type: "adaptive"}` is the only on-mode; `{type: "disabled"}` accepted | Runs adaptive | Removed - 400 | Removed - 400 | `low`/`medium`/`high`/`xhigh`/`max` |
| Opus 4.6 / Sonnet 4.6 | `{type: "adaptive"}` (recommended; auto-enables interleaved thinking, no beta header) | Set `{type: "adaptive"}` explicitly | Deprecated - do not use in new code; transitional escape hatch only (see below) | Allowed | `low`/`medium`/`high`/`max` (`xhigh` arrived with Opus 4.7) |
| Haiku 4.5; older models (Sonnet 4.5, ...) only if explicitly requested | `{type: "enabled", budget_tokens: N}` | No thinking | Required for thinking; must be less than `max_tokens`, minimum 1024 - errors otherwise | Allowed | `effort` works on Opus 4.5 (`low`/`medium`/`high` only - no `xhigh`/`max`); errors on Sonnet 4.5 / Haiku 4.5 |

| 模型 | 思考配置 | 省略 `thinking` 时 | `budget_tokens` | 采样（`temperature`/`top_p`/`top_k`） | Effort 级别 |
|---|---|---|---|---|---|
| Fable 5 / Claude Fable 5.1（及对应的 Mythos 型号） | `{type: "adaptive"}` 或省略；显式 `{type: "disabled"}` 返回 400——应省略该参数（Claude Fable 5.1 / Claude Mythos 5.1 在强制 `tool_choice` `any`/`tool` 时也返回 400；Claude Fable 5.1 对回传的思考块运行 preserved thinking 的历史编辑检查，Claude Mythos 5.1 则不运行） | 运行自适应（思考始终开启） | 已移除——`{type: "enabled", budget_tokens: N}` 返回 400 | 已移除——返回 400 | `low`/`medium`/`high`/`xhigh`/`max` |
| Claude Opus 5.5 | `{type: "adaptive"}` 或省略；`{type: "disabled"}` 与 `{type: "enabled", budget_tokens}` 在**每个** effort 级别都返回 400——省略该参数并降低 effort 代替（强制 `tool_choice` `any`/`tool` 时同样 400，且运行 preserved thinking——见 `shared/model-migration.md` -> Migrating to Claude Opus 5.5） | 运行**自适应** | 已移除——400 | 已移除——400 | `low`/`medium`/`high`/`xhigh`/`max`——**默认 `medium`**（不是 `high`）；支持逐消息 effort（beta） |
| Claude Opus 5 | `{type: "adaptive"}` 或省略；`{type: "disabled"}` **仅在 effort `high` 及以下被接受**——`xhigh`/`max` 时返回 400，另见下方"关闭思考的陷阱" | 运行**自适应**（思考默认开启——不同于 Opus 4.8/4.7） | 已移除——400 | 已移除——400 | `low`-`max`（全部五档） |
| Opus 4.8 / 4.7 | `{type: "adaptive"}` 是唯一的开启方式；`{type: "disabled"}` 可接受 | 运行时**没有**思考——请显式设置 `{type: "adaptive"}` | 已移除——400 | 已移除——400 | `low`/`medium`/`high`/`xhigh`/`max` |
| Claude Sonnet 5.5 | `{type: "adaptive"}` 或省略；`{type: "disabled"}` 返回 400——要关闭思考请发送 `{type: "between_tools"}`（不带其他字段；`xhigh`/`max` 时 400；使用它时 effort 无法在对话中途更改）（强制 `tool_choice` `any`/`tool` 时同样 400，且运行 preserved thinking——见 `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5） | 运行**自适应** | 已移除——400 | 非默认值——400 | `low`/`medium`/`high`/`xhigh`/`max`——默认 `high`，各级别自 Claude Sonnet 5 起重新校准；开启思考时支持逐消息 effort（beta） |
| Sonnet 5 | `{type: "adaptive"}` 是唯一的开启方式；`{type: "disabled"}` 可接受 | 运行自适应 | 已移除——400 | 已移除——400 | `low`/`medium`/`high`/`xhigh`/`max` |
| Opus 4.6 / Sonnet 4.6 | `{type: "adaptive"}`（推荐；自动启用交错思考，无需 beta 头） | 请显式设置 `{type: "adaptive"}` | 已弃用——新代码不要使用；仅作过渡逃生门（见下文） | 允许 | `low`/`medium`/`high`/`max`（`xhigh` 随 Opus 4.7 引入） |
| Haiku 4.5；旧模型（Sonnet 4.5 等）仅在明确要求时使用 | `{type: "enabled", budget_tokens: N}` | 无思考 | 开启思考时必需；必须小于 `max_tokens`，最小 1024——否则报错 | 允许 | `effort` 在 Opus 4.5 上有效（仅 `low`/`medium`/`high`——无 `xhigh`/`max`）；在 Sonnet 4.5 / Haiku 4.5 上报错 |

Opus 4.8 keeps the same request surface as 4.7 (no new breaking changes) - see `shared/model-migration.md` -> Migrating to Opus 4.8 for the behavioral re-tuning, and -> Migrating to Opus 4.7 for the full breaking-change list when coming from 4.6 or earlier. With `thinking` disabled, Opus 4.8 may write longer reasoning into the visible response - leave adaptive thinking on, or add a final-answer-only instruction (see the migration guide).

Opus 4.8 保持了与 4.7 相同的请求表面（没有新的破坏性变更）——行为上的重新调优见 `shared/model-migration.md` -> Migrating to Opus 4.8；从 4.6 或更早版本迁移时的完整破坏性变更清单见 -> Migrating to Opus 4.7。关闭 `thinking` 时，Opus 4.8 可能把更长的推理写进可见响应——请保持自适应思考开启，或添加"只输出最终答案"的指令（见迁移指南）。

- **Effort (GA, no beta header):** `output_config: {effort: "low"|"medium"|"high"|"xhigh"|"max"}` - inside `output_config`, not top-level; default `high` (equivalent to omitting it) on every current model except Claude Opus 5.5, whose default is `medium` (thinking table above) - set it explicitly there. Controls thinking depth and overall token spend; combine with adaptive thinking for the best cost-quality tradeoffs. `xhigh` (added on Opus 4.7, between `high` and `max`) is the best setting for most coding and agentic use cases on Fable 5 / Opus 4.7/4.8 / Sonnet 5, and the default in Claude Code; effort matters more on those models than on any prior model in their tier - re-tune it when migrating, and run long-horizon/agentic tasks at `high`/`xhigh` with the full task spec given up front. Use a minimum of `high` for intelligence-sensitive work, `max` when correctness matters more than cost, and `low` for subagents or simple tasks - lower effort means fewer and more-consolidated tool calls, less preamble, and terser confirmations (`high` is often the sweet spot balancing quality and token efficiency).
  **Effort（已正式发布，无需 beta 头）：**`output_config: {effort: "low"|"medium"|"high"|"xhigh"|"max"}`——放在 `output_config` 内部，不是顶层；除 Claude Opus 5.5（默认 `medium`，见上方思考表）外，每个当前模型的默认值为 `high`（等同于省略它）——在该模型上请显式设置。它控制思考深度与整体 token 消耗；与自适应思考组合可获得最佳成本-质量权衡。`xhigh`（随 Opus 4.7 引入，介于 `high` 与 `max` 之间）是 Fable 5 / Opus 4.7/4.8 / Sonnet 5 上大多数编码与智能体用例的最佳设置，也是 Claude Code 中的默认值；effort 在这些模型上的影响超过其同档任何先前模型——迁移时请重新调优，并以 `high`/`xhigh` 运行长程/智能体任务，同时在请求中 upfront 给出完整任务说明。对智能敏感度高的工作至少用 `high`；正确性比成本更重要时用 `max`；子智能体或简单任务用 `low`——更低的 effort 意味着更少、更聚合的工具调用、更少的前言与更简洁的确认（`high` 通常是质量与 token 效率的平衡点）。
- **Choosing an effort level (cost tuning):** Effort is the first quality-trading lever, after the free wins (caching first) - it trades thoroughness against token spend within one model, and the top of the range earns its cost only on hard problems (raise to `max` only when measurement shows headroom at the level below). Which workloads repay higher effort is a property of the workload: coding and long-horizon agentic work respond strongly; chat, classification, and high-volume or latency-sensitive routes often don't and do well at `low`, with `medium` as the cost-saving step-down where quality holds (the per-level defaults above cover the rest). Measure on a sample of real requests before raising a default, and tune per route rather than globally. Before building a multi-model cost cascade, measure the simpler alternative first - the most capable model at lower effort on the same tasks: lower effort on the newest models often matches or exceeds prior-generation performance at high effort (on Fable 5, lower effort often exceeds `xhigh` on prior models), and one model means one cache namespace (caches are model-scoped, so a cascade forfeits cache reuse across its models; a mid-conversation top-level `effort` change still invalidates the messages cache, though the per-message effort system message avoids that on Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Opus 5 / Claude Sonnet 5.5 (with adaptive thinking) - `shared/prompt-caching.md` § Invalidation hierarchy). Judge cost per completed task, not per request - a cheaper request that needs more turns or retries to finish the job isn't cheaper. For the measured effort/cost tradeoffs by workload and the full lever order, `shared/cost-optimization.md` § 2.6.
  **选择 effort 级别（成本调优）：**effort 是第一个以质量换成本的杠杆，排在免费收益之后（先做缓存）——它在单个模型内以彻底性换取 token 消耗，且该范围顶端只有在难题上才配得上其成本（只有当测量显示下一档仍有余量时才升到 `max`）。哪些工作负载能从更高 effort 中获益是工作负载自身的属性：编码与长程智能体工作响应强烈；聊天、分类以及高并发或延迟敏感的路由往往不能，且在 `low` 下表现良好，在质量仍达标时以 `medium` 作为省钱的降档（其余由上述各级别默认值覆盖）。在提高默认值之前，先在真实请求样本上测量，并按路由调优而非全局调优。在构建多模型成本级联之前，先测量更简单的替代方案——同一任务上以更低 effort 运行能力最强的模型：最新模型上的低 effort 往往等同甚至超过上一代模型的高 effort 表现（在 Fable 5 上，更低 effort 常常超过旧模型的 `xhigh`），且单一模型意味着单一缓存命名空间（缓存按模型划分，级联会牺牲跨模型的缓存复用；对话中途更改顶层 `effort` 仍会使 messages 缓存失效，不过逐消息 effort 系统消息在 Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Opus 5 / Claude Sonnet 5.5（开启自适应思考时）上可避免该问题——见 `shared/prompt-caching.md` § Invalidation hierarchy）。按完成的任务评估成本，而不是按请求——一个需要更多轮次或重试才能完成任务的"更便宜"的请求并不便宜。各工作负载的实测 effort/成本权衡与完整的杠杆顺序见 `shared/cost-optimization.md` § 2.6。
- **Thinking display - `"omitted"` by default on Fable 5 / Claude Fable 5.1 / Mythos 5 / Claude Mythos 5.1 / Opus 5.5 / 5 / 4.8 / 4.7 / Sonnet 5 / Claude Sonnet 5.5:** `display: "summarized"` returns a readable summary of the reasoning; `"omitted"` (the default on all ten - a silent change from Opus 4.6 and Sonnet 4.6, where it was `"summarized"`) streams `thinking` blocks with empty text. `display` controls visibility only - thinking happens and is billed the same under every setting; the raw chain of thought is never exposed on any model. If you stream reasoning to users, the default looks like a long pause before output - set `thinking: {type: "adaptive", display: "summarized"}` explicitly. (Independent of display, echo thinking blocks back unchanged when continuing on the same model; other models silently ignore them (Claude Fable 5.1 / Claude Mythos 5.1 read them, and Claude Sonnet 5.5 reads Claude Sonnet 5, Opus 4.8, Haiku 4.5, and earlier models' blocks) - see the migration guide.) On Claude Fable 5.1 / Claude Mythos 5.1 / Claude Fable 5 / Claude Opus 5.5 / Claude Sonnet 5.5, `display: "updates"` (beta `thinking-display-updates-2026-08-18`, every platform) hides reasoning like `"omitted"` but returns the model's between-tool-call progress notes as short `thinking` block summaries - see `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features.
  **思考展示——Fable 5 / Claude Fable 5.1 / Mythos 5 / Claude Mythos 5.1 / Opus 5.5 / 5 / 4.8 / 4.7 / Sonnet 5 / Claude Sonnet 5.5 上默认为 `"omitted"`：**`display: "summarized"` 返回推理的可读摘要；`"omitted"`（全部十个模型的默认值——相较 Opus 4.6 与 Sonnet 4.6 的 `"summarized"` 是一次静默变更）则流式返回文本为空的 `thinking` 块。`display` 只控制可见性——无论哪种设置，思考都会发生且计费相同；任何模型都不会暴露原始思维链。如果你把推理流式展示给用户，默认值看起来像输出前的一段长暂停——请显式设置 `thinking: {type: "adaptive", display: "summarized"}`。（与 display 无关：在同一模型上继续对话时，原样回传思考块；其他模型会静默忽略它们（Claude Fable 5.1 / Claude Mythos 5.1 会读取它们，Claude Sonnet 5.5 能读取 Claude Sonnet 5、Opus 4.8、Haiku 4.5 及更早模型的思考块）——见迁移指南。）在 Claude Fable 5.1 / Claude Mythos 5.1 / Claude Fable 5 / Claude Opus 5.5 / Claude Sonnet 5.5 上，`display: "updates"`（beta `thinking-display-updates-2026-08-18`，所有平台）像 `"omitted"` 一样隐藏推理，但把模型在工具调用之间的进度说明作为简短的 `thinking` 块摘要返回——见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features。
- **When the user asks for "extended thinking", a "thinking budget", or `budget_tokens`:** always use Fable 5/5.1, Opus 5.5, 5, 4.8, 4.7, or 4.6 with `thinking: {type: "adaptive"}` - the fixed thinking-token-budget concept is deprecated and adaptive thinking replaces it. Do NOT use `budget_tokens` for new 4.6/4.7/4.8 code and do NOT switch to an older model just because the user mentions it. *Gradual-migration carve-out:* `budget_tokens` is still functional on Opus 4.6 and Sonnet 4.6 only, as a transitional escape hatch for existing code that needs a hard token ceiling before you've tuned `effort` - see `shared/model-migration.md` -> Transitional escape hatch. It is fully removed on Fable 5/5.1, Opus 5.5/5/4.7/4.8, and Sonnet 5.
  **当用户要求"extended thinking"、"thinking budget"或 `budget_tokens` 时：**一律在 Fable 5/5.1、Opus 5.5、5、4.8、4.7 或 4.6 上使用 `thinking: {type: "adaptive"}`——固定思考 token 预算的概念已弃用，由自适应思考取代。新写的 4.6/4.7/4.8 代码**不要**使用 `budget_tokens`，也**不要**仅因用户提及就切换到旧模型。*渐进迁移豁免：*`budget_tokens` 仅在 Opus 4.6 与 Sonnet 4.6 上仍然可用，作为在你调优好 `effort` 之前、需要硬性 token 上限的现有代码的过渡逃生门——见 `shared/model-migration.md` -> Transitional escape hatch。它在 Fable 5/5.1、Opus 5.5/5/4.7/4.8 与 Sonnet 5 上已被完全移除。

---

## Compaction (Quick Reference) / 压缩（速查）

**Beta, Fable 5/5.1, Opus 5.5, Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5.5, Sonnet 5, and Sonnet 4.6.** For long-running conversations that may exceed the 1M context window, enable server-side compaction. The API automatically summarizes earlier context when it approaches the trigger threshold (default: 150K tokens). Requires beta header `compact-2026-01-12`.

**Beta，适用于 Fable 5/5.1、Opus 5.5、Opus 5、Opus 4.8、Opus 4.7、Opus 4.6、Sonnet 5.5、Sonnet 5 与 Sonnet 4.6。**对于可能超出 1M 上下文窗口的长时对话，启用服务器端压缩。当上下文接近触发阈值（默认：150K token）时，API 会自动摘要较早的上下文。需要 beta 头 `compact-2026-01-12`。

**Critical:** Append `response.content` (not just the text) back to your messages on every turn. Compaction blocks in the response must be preserved - the API uses them to replace the compacted history on the next request. Extracting only the text string and appending that will silently lose the compaction state.

**关键：**每一轮都要把 `response.content`（而不仅仅是文本）追加回你的消息。响应中的压缩块必须被保留——API 用它们在下一次请求中替换已压缩的历史。只提取文本字符串并追加它会静默丢失压缩状态。

See `{lang}/claude-api/README.md` (Compaction section) for code examples. Full docs via WebFetch in `shared/live-sources.md`.

代码示例见 `{lang}/claude-api/README.md`（Compaction 一节）。完整文档可通过 WebFetch 获取，见 `shared/live-sources.md`。

---

## Prompt Caching (Quick Reference) / 提示词缓存（速查）

**Prefix match.** Any byte change anywhere in the prefix invalidates everything after it. Render order is `tools` -> `system` -> `messages`. Keep stable content first (frozen system prompt, deterministic tool list), put volatile content (timestamps, per-request IDs, varying questions) after the last `cache_control` breakpoint.

**前缀匹配。**前缀中任何位置的字节变化都会使其后所有内容的缓存失效。渲染顺序为 `tools` -> `system` -> `messages`。把稳定内容放在前面（固定的系统提示词、确定性的工具清单），把易变内容（时间戳、每请求 ID、各不相同的问题）放在最后一个 `cache_control` 断点之后。

**Mid-conversation operator instructions** (Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, Claude Sonnet 5.5; not Claude Sonnet 5; no beta header): append `{"role": "system", ...}` to `messages[]` instead of editing top-level `system`. Preserves the cached history prefix and is the prompt-injection-safe operator channel. See `shared/prompt-caching.md` § Mid-conversation system messages.

**对话中途的操作员指令**（Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1、Claude Sonnet 5.5；不含 Claude Sonnet 5；无需 beta 头）：向 `messages[]` 追加 `{"role": "system", ...}`，而不是编辑顶层 `system`。这样既保留已缓存的历史前缀，也是防提示词注入的操作员通道。见 `shared/prompt-caching.md` § Mid-conversation system messages。

**Top-level auto-caching** (`cache_control: {type: "ephemeral"}` on `messages.create()`) is the simplest option when you don't need fine-grained placement. Max 4 breakpoints per request. Minimum cacheable prefix is model-dependent (512-4096 tokens - see `shared/prompt-caching.md` § API reference) - shorter prefixes silently won't cache.

**顶层自动缓存**（在 `messages.create()` 上设置 `cache_control: {type: "ephemeral"}`）是不需要精细放置时最简单的选项。每个请求最多 4 个断点。最小可缓存前缀依模型而定（512-4096 token——见 `shared/prompt-caching.md` § API reference）——更短的前缀会静默地不缓存。

**Verify with `usage.cache_read_input_tokens`** - if it's zero across repeated requests, a silent invalidator is at work (`datetime.now()` in system prompt, unsorted JSON, varying tool set).

**用 `usage.cache_read_input_tokens` 验证**——如果重复请求中它始终为零，说明有静默失效因素在作怪（系统提示词中的 `datetime.now()`、未排序的 JSON、变化的工具集）。

For placement patterns, architectural guidance, and the silent-invalidator audit checklist: read `shared/prompt-caching.md`. Language-specific syntax: `{lang}/claude-api/README.md` (Prompt Caching section).

放置模式、架构指导与静默失效因素审计清单：请阅读 `shared/prompt-caching.md`。各语言的语法：`{lang}/claude-api/README.md`（Prompt Caching 一节）。

---

## Fast Mode (Quick Reference) / 快速模式（速查）

**Research preview, Claude Opus 5 / Claude Opus 5.5 / Opus 4.8 only** - Claude API and Managed Agents, not Bedrock / Google Cloud / Foundry. Opus 4.7 fast mode has been removed: `speed: "fast"` on 4.7 returns an error. Fast mode on Claude Opus 5 is priced at $10 / $50 per MTok; on Claude Opus 5.5, $8 / $40. Fast mode runs the same model at up to 2.5x higher output tokens per second, at premium pricing. Three things are required on every request: use the **beta** messages endpoint (`client.beta.messages....`), pass the beta flag `fast-mode-2026-02-01`, and set `speed: "fast"` as a top-level request parameter (not a header, not in `extra_body`).

**研究预览版，仅限 Claude Opus 5 / Claude Opus 5.5 / Opus 4.8**——仅 Claude API 与 Managed Agents，不支持 Bedrock / Google Cloud / Foundry。Opus 4.7 的快速模式已移除：在 4.7 上 `speed: "fast"` 会返回错误。快速模式在 Claude Opus 5 上定价为每 MTok $10 / $50；在 Claude Opus 5.5 上为 $8 / $40。快速模式以最高 2.5 倍的每秒输出 token 速度运行同一模型，按溢价计费。每个请求都需要三件事：使用 **beta** messages 端点（`client.beta.messages....`）、传入 beta 旗标 `fast-mode-2026-02-01`，并把 `speed: "fast"` 设为顶层请求参数（不是头部，也不在 `extra_body` 里）。

```python
client.beta.messages.create(
    model="claude-opus-5-5", max_tokens=4096,
    speed="fast", betas=["fast-mode-2026-02-01"],
    messages=[...],
)
```

| Language | Beta flag | Speed parameter |
|---|---|---|
| Python | `betas=["fast-mode-2026-02-01"]` | `speed="fast"` |
| TypeScript / Ruby | `betas: ["fast-mode-2026-02-01"]` | `speed: "fast"` |
| Go | `[]anthropic.AnthropicBeta{anthropic.AnthropicBetaFastMode2026_02_01}` | `Speed: anthropic.BetaMessageNewParamsSpeedFast` |
| Java | `.addBeta(AnthropicBeta.FAST_MODE_2026_02_01)` | `.speed(MessageCreateParams.Speed.FAST)` |
| C# | `Betas = ["fast-mode-2026-02-01"]` | `Speed = Speed.Fast` (`Anthropic.Models.Beta.Messages`) |
| PHP | `betas: ['fast-mode-2026-02-01']` | `speed: 'fast'` |
| cURL | `anthropic-beta: fast-mode-2026-02-01` header | `"speed": "fast"` in body |

| 语言 | Beta 旗标 | 速度参数 |
|---|---|---|
| Python | `betas=["fast-mode-2026-02-01"]` | `speed="fast"` |
| TypeScript / Ruby | `betas: ["fast-mode-2026-02-01"]` | `speed: "fast"` |
| Go | `[]anthropic.AnthropicBeta{anthropic.AnthropicBetaFastMode2026_02_01}` | `Speed: anthropic.BetaMessageNewParamsSpeedFast` |
| Java | `.addBeta(AnthropicBeta.FAST_MODE_2026_02_01)` | `.speed(MessageCreateParams.Speed.FAST)` |
| C# | `Betas = ["fast-mode-2026-02-01"]` | `Speed = Speed.Fast`（`Anthropic.Models.Beta.Messages`） |
| PHP | `betas: ['fast-mode-2026-02-01']` | `speed: 'fast'` |
| cURL | `anthropic-beta: fast-mode-2026-02-01` 头 | 请求体中的 `"speed": "fast"` |

`response.usage.speed` reports which speed was used. Fast mode has its own rate limit separate from standard Opus; on 429, either retry after the `retry-after` delay or drop `speed` and fall back to standard (note: switching speed invalidates prompt cache). Not available with Batch API, Priority Tier, Claude Platform on AWS, or third-party platforms.

`response.usage.speed` 报告实际使用的速度。快速模式拥有独立于标准 Opus 的限流；遇到 429 时，要么按 `retry-after` 延迟重试，要么去掉 `speed` 回退到标准速度（注意：切换速度会使提示词缓存失效）。Batch API、Priority Tier、Claude Platform on AWS 与第三方平台均不支持。

**Priority Tier is not supported on every current model.** It is supported on Claude Fable 5, Opus 4.8, and the older current models, but Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5, Claude Sonnet 5.5, Claude Fable 5.1, Claude Mythos 5.1, Claude Mythos 5, and Mythos Preview are excluded - a Priority Tier request naming one of them fails validation.

**并非每个当前模型都支持 Priority Tier。**它支持 Claude Fable 5、Opus 4.8 以及更早的当前模型，但 Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5、Claude Sonnet 5.5、Claude Fable 5.1、Claude Mythos 5.1、Claude Mythos 5 与 Mythos Preview 被排除在外——指名其中之一的 Priority Tier 请求会校验失败。

---

## Task Budgets (Quick Reference) / 任务预算（速查）

**Beta, Claude Opus 5 / Claude Opus 5.5 / Fable 5 / Claude Fable 5.1 (confirm at launch) / Claude Sonnet 5.5 / Opus 4.8 / 4.7 (not Claude Sonnet 5).** A task budget gives Claude a token ceiling for an agentic loop so it paces itself and finishes gracefully instead of being cut off - distinct from `max_tokens`, which is an enforced per-response ceiling the model is not aware of. Minimum `total`: 20,000. Set `task_budget` inside `output_config` on `client.beta.messages.stream(...)` with beta flag `task-budgets-2026-03-13` - use streaming so the large `max_tokens` doesn't hit HTTP timeouts (full details: `shared/model-migration.md` -> Task Budgets):

**Beta，适用于 Claude Opus 5 / Claude Opus 5.5 / Fable 5 / Claude Fable 5.1（发布时确认）/ Claude Sonnet 5.5 / Opus 4.8 / 4.7（不含 Claude Sonnet 5）。**任务预算为 Claude 的智能体循环设定一个 token 上限，使其自我调节节奏并体面收尾，而不是被硬性切断——这与 `max_tokens` 不同，后者是模型无从知晓的、按响应强制执行的上限。`total` 最小值：20,000。通过 `client.beta.messages.stream(...)` 的 `output_config` 内设置 `task_budget`，并带 beta 旗标 `task-budgets-2026-03-13`——请使用流式，以免过大的 `max_tokens` 触发 HTTP 超时（完整细节：`shared/model-migration.md` -> Task Budgets）：

```python
with client.beta.messages.stream(
    model="claude-opus-5-5", max_tokens=128000,
    output_config={"effort": "high", "task_budget": {"type": "tokens", "total": 64000}},
    betas=["task-budgets-2026-03-13"],
    messages=[...], tools=[...],
) as stream:
    response = stream.get_final_message()
```

`task_budget` fields: `type` (always `"tokens"`), `total`, and optional `remaining` (defaults to `total`). The server injects a countdown marker Claude sees during generation; the budget counts what Claude generates and the tool results it reads this turn - **not** the full history you resend each request. Not the same thing as **Managed Agents session budgets** - those are hard, dollar-denominated, platform-enforced caps on one CMA session (`shared/managed-agents-core.md` § Session budgets); a task budget is advisory and token-denominated.

`task_budget` 字段：`type`（恒为 `"tokens"`）、`total` 与可选的 `remaining`（默认等于 `total`）。服务器会注入一个倒计时标记，Claude 在生成过程中可以看到；预算统计的是 Claude 本轮生成的内容与其读取的工具结果——**而不是**你每次请求重发的完整历史。这与 **Managed Agents 会话预算**不是一回事——后者是对单个 CMA 会话的硬性、以美元计价、由平台强制执行的上限（`shared/managed-agents-core.md` § Session budgets）；任务预算是建议性的、以 token 计价。

**Observing spend:** accumulate `response.usage.output_tokens` (plus the token count of the tool-result blocks you append) across loop iterations if you want to display progress. Leave `remaining` unset in the normal loop - the server tracks the countdown itself, and passing a client-computed `remaining` while also resending full history under-reports the budget. **Only pass `remaining`** when you compact or rewrite history between requests and the server can no longer derive prior spend.

**观察消耗：**如果想展示进度，可在循环迭代间累计 `response.usage.output_tokens`（加上你追加的工具结果块的 token 数）。常规循环中请让 `remaining` 保持未设置——服务器自己跟踪倒计时，在重发完整历史的同时传入客户端计算的 `remaining` 会少报预算。**仅当**你在请求之间压缩或重写历史、服务器无法再推导先前消耗时，才传入 `remaining`。

---

## Provider Clients (Quick Reference) / 厂商客户端（速查）

When targeting Claude on a third-party platform, use that platform's dedicated client class - not the first-party `Anthropic()` client with a `base_url` override. After construction the client exposes the same `messages.create` / `.stream` surface as the first-party SDK.

当目标第三方平台上的 Claude 时，请使用该平台专用的客户端类——而不是带 `base_url` 覆盖的第一方 `Anthropic()` 客户端。构造完成后，客户端暴露与第一方 SDK 相同的 `messages.create` / `.stream` 表面。

### Amazon Bedrock

Use the **Mantle** client (Messages-API Bedrock endpoint). Bedrock model IDs take an `anthropic.` prefix (e.g. `"anthropic.claude-opus-5-5"`). Region is required.

使用 **Mantle** 客户端（Messages-API 的 Bedrock 端点）。Bedrock 模型 ID 带 `anthropic.` 前缀（例如 `"anthropic.claude-opus-5-5"`）。区域为必填。

| Language | Client |
|---|---|
| Python | `from anthropic import AnthropicBedrockMantle` -> `AnthropicBedrockMantle(aws_region="...")` |
| TypeScript | `import { AnthropicBedrockMantle } from "@anthropic-ai/bedrock-sdk"` -> `new AnthropicBedrockMantle({ awsRegion: "..." })` |
| Go | `bedrock.NewMantleClient(ctx, bedrock.MantleClientConfig{ AWSRegion: "..." })` |
| Java | `AnthropicOkHttpClient.builder().backend(BedrockMantleBackend.fromEnv()).build()` (from `com.anthropic.bedrock.backends`) |
| C# | `new AnthropicBedrockMantleClient(new() { AwsRegion = "..." })` (package `Anthropic.Bedrock`) |
| PHP | `use Anthropic\Bedrock\MantleClient;` -> `new MantleClient(awsRegion: '...')` |
| Ruby | `Anthropic::BedrockMantleClient.new(aws_region: "...")` |

| 语言 | 客户端 |
|---|---|
| Python | `from anthropic import AnthropicBedrockMantle` -> `AnthropicBedrockMantle(aws_region="...")` |
| TypeScript | `import { AnthropicBedrockMantle } from "@anthropic-ai/bedrock-sdk"` -> `new AnthropicBedrockMantle({ awsRegion: "..." })` |
| Go | `bedrock.NewMantleClient(ctx, bedrock.MantleClientConfig{ AWSRegion: "..." })` |
| Java | `AnthropicOkHttpClient.builder().backend(BedrockMantleBackend.fromEnv()).build()`（来自 `com.anthropic.bedrock.backends`） |
| C# | `new AnthropicBedrockMantleClient(new() { AwsRegion = "..." })`（包 `Anthropic.Bedrock`） |
| PHP | `use Anthropic\Bedrock\MantleClient;` -> `new MantleClient(awsRegion: '...')` |
| Ruby | `Anthropic::BedrockMantleClient.new(aws_region: "...")` |

`AnthropicBedrock` / `BedrockClient` / `BedrockBackend` (without `Mantle`) are the legacy `bedrock-runtime` InvokeModel path - prefer the Mantle client for new code.

`AnthropicBedrock` / `BedrockClient` / `BedrockBackend`（不带 `Mantle`）是旧的 `bedrock-runtime` InvokeModel 路径——新代码请优先使用 Mantle 客户端。

### Microsoft Foundry

| Language | Client |
|---|---|
| Python | `from anthropic import AnthropicFoundry` -> `AnthropicFoundry(api_key=..., resource="...")` |
| TypeScript | `import AnthropicFoundry from "@anthropic-ai/foundry-sdk"` -> `new AnthropicFoundry({ ... })` |
| Java | `AnthropicOkHttpClient.builder().backend(FoundryBackend.fromEnv()).build()` (from `com.anthropic.foundry.backends`) |
| C# | `new AnthropicFoundryClient(new AnthropicFoundryApiKeyCredentials(...))` (package `Anthropic.Foundry`) |
| PHP | `Foundry\Client::withCredentials(...)` |

| 语言 | 客户端 |
|---|---|
| Python | `from anthropic import AnthropicFoundry` -> `AnthropicFoundry(api_key=..., resource="...")` |
| TypeScript | `import AnthropicFoundry from "@anthropic-ai/foundry-sdk"` -> `new AnthropicFoundry({ ... })` |
| Java | `AnthropicOkHttpClient.builder().backend(FoundryBackend.fromEnv()).build()`（来自 `com.anthropic.foundry.backends`） |
| C# | `new AnthropicFoundryClient(new AnthropicFoundryApiKeyCredentials(...))`（包 `Anthropic.Foundry`） |
| PHP | `Foundry\Client::withCredentials(...)` |

The Go and Ruby SDKs do not currently support Foundry. For Ruby, use the standard `Anthropic::Client.new(base_url: "<foundry endpoint>")` as a fallback (Entra ID auth is not built in). For Claude Platform on AWS, see `shared/claude-platform-on-aws.md`.

Go 与 Ruby SDK 目前不支持 Foundry。Ruby 可退而使用标准的 `Anthropic::Client.new(base_url: "<foundry endpoint>")`（未内置 Entra ID 认证）。Claude Platform on AWS 见 `shared/claude-platform-on-aws.md`。

### Google Cloud Vertex AI

Two required constructor args: GCP `project_id` and `region`. Vertex model IDs take **no prefix** - current-generation models (Opus 5.5/5/4.8/4.7/4.6, Sonnet 5.5, Sonnet 5, Sonnet 4.6) use the bare first-party ID (e.g. `"claude-opus-5-5"`); dated-snapshot models use an `@` version separator (e.g. `claude-opus-4-5@20251101`, **not** `claude-opus-4-5-20251101`). Auth is GCP ADC (`gcloud auth application-default login`); no Anthropic API key. `region` can be `"global"` (recommended), a multi-region (`"us"`/`"eu"`), or a specific region. After construction, use the same `messages.create` / `.stream` surface.

两个必填构造参数：GCP `project_id` 与 `region`。Vertex 模型 ID **不带前缀**——当前世代模型（Opus 5.5/5/4.8/4.7/4.6、Sonnet 5.5、Sonnet 5、Sonnet 4.6）使用裸的第一方 ID（例如 `"claude-opus-5-5"`）；带日期快照模型使用 `@` 版本分隔符（例如 `claude-opus-4-5@20251101`，**而不是** `claude-opus-4-5-20251101`）。认证为 GCP ADC（`gcloud auth application-default login`）；无需 Anthropic API 密钥。`region` 可为 `"global"`（推荐）、多区域（`"us"`/`"eu"`）或特定区域。构造完成后，使用相同的 `messages.create` / `.stream` 表面。

| Language | Client |
|---|---|
| Python | `from anthropic import AnthropicVertex` -> `AnthropicVertex(project_id="...", region="...")` (install `"anthropic[vertex]"`) |
| TypeScript | `import { AnthropicVertex } from "@anthropic-ai/vertex-sdk"` -> `new AnthropicVertex({ projectId, region })` |
| Go | `import "github.com/anthropics/anthropic-sdk-go/vertex"` -> `anthropic.NewClient(vertex.WithGoogleAuth(ctx, region, projectID))` |
| Java | `AnthropicOkHttpClient.builder().backend(VertexBackend.builder().region("...").project("...").build()).build()` (from `com.anthropic.vertex.backends`) |
| C# | `new AnthropicClient { Backend = new VertexBackend(projectId, region) }` (package `Anthropic.Vertex`) |
| PHP | `use Anthropic\Vertex;` -> `Vertex\Client::fromEnvironment(location: '...', projectId: '...')` - note `location`, not `region` |
| Ruby | `Anthropic::VertexClient.new(region: "...", project_id: "...")` |

| 语言 | 客户端 |
|---|---|
| Python | `from anthropic import AnthropicVertex` -> `AnthropicVertex(project_id="...", region="...")`（安装 `"anthropic[vertex]"`） |
| TypeScript | `import { AnthropicVertex } from "@anthropic-ai/vertex-sdk"` -> `new AnthropicVertex({ projectId, region })` |
| Go | `import "github.com/anthropics/anthropic-sdk-go/vertex"` -> `anthropic.NewClient(vertex.WithGoogleAuth(ctx, region, projectID))` |
| Java | `AnthropicOkHttpClient.builder().backend(VertexBackend.builder().region("...").project("...").build()).build()`（来自 `com.anthropic.vertex.backends`） |
| C# | `new AnthropicClient { Backend = new VertexBackend(projectId, region) }`（包 `Anthropic.Vertex`） |
| PHP | `use Anthropic\Vertex;` -> `Vertex\Client::fromEnvironment(location: '...', projectId: '...')`——注意是 `location`，不是 `region` |
| Ruby | `Anthropic::VertexClient.new(region: "...", project_id: "...")` |

---

## Context Editing (Quick Reference) / 上下文编辑（速查）

**Beta.** Context editing **clears** old tool results or thinking blocks from the conversation before the model sees it; it is **not compaction** (which summarizes). On `client.beta.messages.*` with beta `context-management-2025-06-27`, pass `context_management.edits` with a strategy type:

**Beta。**上下文编辑会在模型看到对话之前**清除**旧的工具结果或思考块；它**不是压缩**（压缩是摘要）。在 `client.beta.messages.*` 上配合 beta `context-management-2025-06-27`，传入带策略类型的 `context_management.edits`：

```python
client.beta.messages.create(
    model="claude-opus-5-5", max_tokens=4096,
    betas=["context-management-2025-06-27"],
    context_management={"edits": [{"type": "clear_tool_uses_20250919"}]},
    tools=[...], messages=[...],
)
```

Strategy types: `clear_tool_uses_20250919` (clears old tool results; optional `clear_tool_inputs: true` also clears the tool_use params) and `clear_thinking_20251015` (clears thinking blocks). Do **not** use `compact_20260112` or beta `compact-2026-01-12` - those are the separate compaction feature.

策略类型：`clear_tool_uses_20250919`（清除旧工具结果；可选的 `clear_tool_inputs: true` 还会清除 tool_use 参数）与 `clear_thinking_20251015`（清除思考块）。**不要**使用 `compact_20260112` 或 beta `compact-2026-01-12`——那是独立的压缩功能。

---

## Mid-Conversation System Messages (Quick Reference) / 对话中途系统消息（速查）

**Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, and Claude Sonnet 5.5; not Claude Sonnet 5; no beta header.** Append `{"role": "system", "content": "..."}` to the `messages` array (not the top-level `system` field) to add an operator instruction mid-conversation without invalidating the cached prefix. Use the regular `client.messages.create` - there is no beta. A mid-conversation system message must follow a `user` message (or an `assistant` message ending in server-tool use), and must be either the last entry in `messages` or be followed by an `assistant` turn - it cannot be `messages[0]`. Availability: `shared/platform-availability.md`. See `shared/prompt-caching.md` § Mid-conversation system messages. A beta extension shipped with Claude Fable 5.1: `output_config: {effort: ...}` with `content: []` changes effort from that point on without a cache reset (beta `mid-conversation-output-config-2026-07-01`; Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5.5, Claude Opus 5, and Claude Sonnet 5.5 with thinking on; Claude API and Google Cloud). An effort-only message (empty `content`) is exempt from the placement rules above - it can sit anywhere in `messages`, including first or between an assistant turn and the next user turn; the rules apply to text and `clear_at` messages. For a per-turn reminder, give the message `clear_at: "next_user_message"` (beta `mid-conversation-system-clear-at-2026-08-21`): it renders for one turn, then stays in the transcript cleared - never delete earlier copies (on Claude Fable 5.1, Claude Opus 5.5, and Claude Sonnet 5.5 deleting one invalidates later thinking blocks); without the beta, a text block after the tool results, earlier copies kept. See `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features.

**Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1 与 Claude Sonnet 5.5；不含 Claude Sonnet 5；无需 beta 头。**向 `messages` 数组（而非顶层 `system` 字段）追加 `{"role": "system", "content": "..."}`，即可在对话中途加入操作员指令而不使已缓存前缀失效。使用常规的 `client.messages.create`——没有 beta。对话中途系统消息必须紧跟在 `user` 消息之后（或以服务器工具使用结尾的 `assistant` 消息之后），并且必须是 `messages` 的最后一项，或其后紧跟一个 `assistant` 轮次——不能是 `messages[0]`。可用性：`shared/platform-availability.md`。见 `shared/prompt-caching.md` § Mid-conversation system messages。随 Claude Fable 5.1 推出的 beta 扩展：`content: []` 的 `output_config: {effort: ...}` 可从该点起更改 effort 而不重置缓存（beta `mid-conversation-output-config-2026-07-01`；支持 Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Opus 5 与开启思考的 Claude Sonnet 5.5；限 Claude API 与 Google Cloud）。仅含 effort 的消息（`content` 为空）豁免于上述放置规则——它可以位于 `messages` 的任何位置，包括首位或 assistant 轮次与下一个 user 轮次之间；这些规则适用于文本与 `clear_at` 消息。若需要每轮提醒，给消息设置 `clear_at: "next_user_message"`（beta `mid-conversation-system-clear-at-2026-08-21`）：它只渲染一轮，然后以已清除状态留在记录中——绝不要删除较早的副本（在 Claude Fable 5.1、Claude Opus 5.5 与 Claude Sonnet 5.5 上，删除会使之后的思考块失效）；没有该 beta 时，则在工具结果之后放一个文本块，并保留较早的副本。见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features。

---

## Managed Agents (Beta) / Managed Agents（Beta）

**Managed Agents** is a third surface: server-managed stateful agents with Anthropic-hosted tool execution. You create a persisted, versioned Agent config (`POST /v1/agents`), then start Sessions that reference it. Each session provisions a container as the agent's workspace - bash, file ops, and code execution run there; the agent loop itself runs on Anthropic's orchestration layer and acts on the container via tools. The session streams events; you send messages and tool results back.

**Managed Agents** 是第三种接入层：由服务器管理的有状态智能体，工具执行由 Anthropic 托管。你先创建一个持久化、带版本的 Agent 配置（`POST /v1/agents`），然后启动引用它的 Session。每个会话会提供一个容器作为智能体的工作区——bash、文件操作与代码执行都在其中运行；智能体循环本身运行在 Anthropic 的编排层上，并通过工具作用于该容器。会话以流式推送事件；你则回传消息与工具结果。

Availability: `shared/platform-availability.md`. For agents on Bedrock / Vertex / Foundry (where Managed Agents is unsupported), use Claude API + tool use.

可用性：`shared/platform-availability.md`。Bedrock / Vertex / Foundry 上的智能体（这些平台不支持 Managed Agents）请使用 Claude API + tool use。

**Mandatory flow:** Agent (once) -> Session (every run). `model`/`system`/`tools` live on the agent, never the session. See `shared/managed-agents-overview.md` for the full reading guide, beta headers, and pitfalls.

**强制流程：**先 Agent（一次）-> 再 Session（每次运行）。`model`/`system`/`tools` 挂在 agent 上，绝不挂在 session 上。完整的阅读指南、beta 头与陷阱见 `shared/managed-agents-overview.md`。

**Beta headers:** `managed-agents-2026-04-01` - the SDK sets this automatically for all `client.beta.{agents,environments,sessions,vaults,deployments,deployment_runs}.*` calls. Memory stores use `agent-memory-2026-07-22` instead, which the SDK sets on `client.beta.memory_stores.*` calls; sending both headers on a memory store request returns a 400. Files API and Skills API are out of beta - no beta header needed (see the API Drift table above for the migration guides).

**Beta 头：**`managed-agents-2026-04-01`——SDK 会在所有 `client.beta.{agents,environments,sessions,vaults,deployments,deployment_runs}.*` 调用上自动设置。记忆存储（memory stores）改用 `agent-memory-2026-07-22`，SDK 会在 `client.beta.memory_stores.*` 调用上设置它；在记忆存储请求上同时发送两个头会返回 400。Files API 与 Skills API 已结束 beta——无需 beta 头（迁移指南见上方的 API Drift 表）。

**Subcommands** - invoke directly with `/claude-api <subcommand>`:

**子命令**——通过 `/claude-api <subcommand>` 直接调用：

| Subcommand | Action |
|---|---|
| `managed-agents-onboard` | Walk the user through setting up a Managed Agent from scratch. **Read `shared/managed-agents-onboarding.md` immediately** and follow its interview script: **describe -> configure the agent (propose, don't interrogate) -> environment -> session** (same arc as the Console quickstart, auth deferred to the session step) - defaults and inline suggestions do the work, with a silent viability gate (job vs tools/credentials/data) before any code is emitted. Do not summarize - run the interview. |

| 子命令 | 动作 |
|---|---|
| `managed-agents-onboard` | 引导用户从零开始设置一个 Managed Agent。**立即阅读 `shared/managed-agents-onboarding.md`** 并遵循其访谈脚本：**描述 -> 配置 agent（提供建议，不要连环追问）-> 环境 -> 会话**（与 Console 快速入门相同的脉络，认证推迟到会话步骤）——由默认值与内联建议完成工作，在产出任何代码之前有一个静默的可行性门（任务对工具/凭据/数据的要求核对）。不要概述——要执行访谈。 |

**Reading guide:** Start with `shared/managed-agents-overview.md`, then the topical `shared/managed-agents-*.md` files (core, environments, tools, events, outcomes, multiagent, webhooks, memory, scheduled-deployments, client-patterns, onboarding, api-reference). For Python, TypeScript, Go, Ruby, PHP, and Java, read `{lang}/managed-agents/README.md` for code examples. For cURL, read `curl/managed-agents.md`. **Agents are persistent - create once, reference by ID.** Define agents and environments as version-controlled files synced with `ant apply` - this is the recommended flow (see `shared/anthropic-cli.md`): the CLI owns the control plane (creating and updating agents), your code owns the data plane (`sessions.create` with the stored agent ID). Call `agents.create()` in code only when you must provision programmatically; either way, store the returned agent ID and pass it to every subsequent `sessions.create`; never call `agents.create()` in the request path. If a binding you need isn't shown in the language README, WebFetch the relevant entry from `shared/live-sources.md` rather than guess. C# has beta Managed Agents support via `client.Beta.Agents` and related namespaces - see `csharp/claude-api/README.md` for details, or `curl/managed-agents.md` for raw HTTP reference.

**阅读指南：**从 `shared/managed-agents-overview.md` 开始，然后阅读各主题的 `shared/managed-agents-*.md` 文件（core、environments、tools、events、outcomes、multiagent、webhooks、memory、scheduled-deployments、client-patterns、onboarding、api-reference）。Python、TypeScript、Go、Ruby、PHP 与 Java 的代码示例见 `{lang}/managed-agents/README.md`。cURL 见 `curl/managed-agents.md`。**Agent 是持久的——创建一次，按 ID 引用。**把 agent 与 environment 定义为受版本控制的文件，用 `ant apply` 同步——这是推荐流程（见 `shared/anthropic-cli.md`）：CLI 负责控制平面（创建与更新 agent），你的代码负责数据平面（用存储的 agent ID 调用 `sessions.create`）。只有在必须以编程方式供给时才在代码里调用 `agents.create()`；无论哪种方式，都要保存返回的 agent ID 并传给后续每次 `sessions.create`；绝不要在请求路径中调用 `agents.create()`。如果需要的绑定没有出现在语言 README 中，请从 `shared/live-sources.md` WebFetch 相应条目，而不是猜测。C# 通过 `client.Beta.Agents` 及相关命名空间提供 beta 版 Managed Agents 支持——详情见 `csharp/claude-api/README.md`，原始 HTTP 参考见 `curl/managed-agents.md`。

**When the user wants to set up a Managed Agent from scratch** (e.g. "how do I get started", "walk me through creating one", "set up a new agent"): read `shared/managed-agents-onboarding.md` and run its interview - same flow as the `managed-agents-onboard` subcommand.

**当用户想从零设置一个 Managed Agent 时**（例如 "how do I get started"、"walk me through creating one"、"set up a new agent"）：阅读 `shared/managed-agents-onboarding.md` 并执行其访谈——与 `managed-agents-onboard` 子命令相同的流程。

**When the user asks "how do I write the client code for X":** reach for `shared/managed-agents-client-patterns.md` - covers lossless stream reconnect, `processed_at` queued/processed gate, interrupt, `tool_confirmation` round-trip, the correct idle/terminated break gate, post-idle status race, stream-first ordering, file-mount gotchas, etc. For credentials, lead with vault `environment_variable` credentials - the first-class mechanism; secrets are substituted at egress and never enter the sandbox (`shared/managed-agents-tools.md` -> Vaults). Keeping credentials host-side via custom tools is the fallback where vault credentials don't fit (e.g. self-hosted sandboxes).

**当用户问"X 的客户端代码怎么写"时：**查阅 `shared/managed-agents-client-patterns.md`——涵盖无损流重连、`processed_at` 排队/已处理门、中断、`tool_confirmation` 往返、正确的 idle/terminated 中断门、idle 之后的状态竞争、流优先排序、文件挂载的坑等。凭据方面，优先使用 vault `environment_variable` 凭据——这是一等机制；密钥在出口处替换，绝不进入沙箱（`shared/managed-agents-tools.md` -> Vaults）。通过自定义工具把凭据留在宿主侧是 vault 凭据不适用时（例如自托管沙箱）的后备方案。

**When the task is a deliverable - default the kickoff to an outcome, not a plain message.** If the session's job is to produce something checkable (an artifact, a report, a PR, a dataset, a fixed set of changes), read `shared/managed-agents-outcomes.md` and kick off with `user.define_outcome` plus a starter rubric you draft from the task (5-10 concrete, independently gradeable criteria; comment it as a starter to tune). Reserve plain `user.message` for genuinely conversational sessions. Trigger on intent, not just the word: "keep working until it's right", "make sure the output is actually good", "don't stop at a first draft" all mean outcomes.

**当任务是一份交付物时——启动会话时默认采用 outcome，而不是普通消息。**如果会话的职责是产出可检验的东西（一个工件、一份报告、一个 PR、一个数据集、一组确定的变更），阅读 `shared/managed-agents-outcomes.md`，并以 `user.define_outcome` 加上你根据任务起草的初始评分规则启动（5-10 条具体、可独立评分的标准；以起始版本注释之，便于调优）。普通 `user.message` 留给真正对话型的会话。按意图触发，而不是只看字眼："keep working until it's right"、"make sure the output is actually good"、"don't stop at a first draft" 都意味着 outcome。

**When the user asks about tool approvals, permission policies, or "auto mode"** (which tool calls need a human, letting the server evaluate calls, `evaluated_permission` / `evaluation` on tool-use events): read `shared/managed-agents-tools.md` § Permission Policies - `always_allow` / `always_ask` / `auto` and the three `auto` outcomes (runs, denied as high-risk, pauses when indeterminate). For attaching a terminal to a live session (`ant beta:sessions connect`): `shared/anthropic-cli.md`.

**当用户问及工具审批、权限策略或"auto mode"时**（哪些工具调用需要人参与、让服务器评估调用、工具使用事件上的 `evaluated_permission` / `evaluation`）：阅读 `shared/managed-agents-tools.md` § Permission Policies——`always_allow` / `always_ask` / `auto` 以及 `auto` 的三种结果（运行、作为高风险被拒绝、不确定时暂停）。为活跃会话附加终端（`ant beta:sessions connect`）：见 `shared/anthropic-cli.md`。

**When the user wants the agent to run on a schedule** (cron, "every night", "weekly report"): read `shared/managed-agents-scheduled-deployments.md` - deployments fire sessions autonomously on a cron cadence, with per-firing run records and lifecycle controls (pause/unpause/archive).

**当用户想让智能体按计划运行时**（cron、"每晚"、"每周报告"）：阅读 `shared/managed-agents-scheduled-deployments.md`——deployment 按 cron 节奏自主触发会话，带每次触发的运行记录与生命周期控制（暂停/恢复/归档）。

**When the agent's work fans out** (research across several sources, per-file or per-record work, "look into N things, then summarize") **or one loop would fill its context with reading:** read `shared/managed-agents-multiagent.md` and recommend a multiagent session - start with just `{"type": "self"}` in the roster so the agent can delegate to copies of itself, then move reading-heavy sub-tasks to a cheaper worker agent (e.g. Claude Haiku 4.5, or Claude Sonnet 5.5 when the worker needs more judgment) referenced by ID.

**当智能体的工作需要扇出时**（跨多个来源的研究、按文件或按记录的工作、"调查 N 件事然后汇总"）**或单个循环会因读取而塞满其上下文时：**阅读 `shared/managed-agents-multiagent.md` 并推荐多智能体会话——先只在名册中放 `{"type": "self"}`，让智能体可以把任务委派给自身的副本，再把阅读密集的子任务交给按 ID 引用的更便宜的 worker 智能体（例如 Claude Haiku 4.5，或 worker 需要更多判断力时的 Claude Sonnet 5.5）。

---

## Server Tools (Quick Reference) / 服务器端工具（速查）

Server-side tools run on Anthropic's infrastructure - no client-side execution loop. Declare in `tools`; results arrive as content blocks in the same response. **No beta header** unless noted. **Prefer the latest type variant your model supports.** The `_20260209` web search / web fetch variants below (dynamic filtering) require Opus 5.5/5/4.8/4.7/4.6, Sonnet 5.5, Sonnet 5, or Sonnet 4.6; the basic variants for older models are listed after the table.

服务器端工具运行在 Anthropic 的基础设施上——无需客户端执行循环。在 `tools` 中声明；结果作为内容块随同一响应返回。除非另有说明，**无需 beta 头**。**优先使用你的模型支持的最新类型变体。**下方的 `_20260209` 网络搜索 / 网络抓取变体（动态过滤）要求 Opus 5.5/5/4.8/4.7/4.6、Sonnet 5.5、Sonnet 5 或 Sonnet 4.6；面向旧模型的基础变体列于表后。

| Tool | `type` | `name` | Key optional params | Result block type |
|---|---|---|---|---|
| Web search | `web_search_20260209` | `web_search` | `max_uses`, `allowed_domains`/`blocked_domains`, `user_location` | `web_search_tool_result` -> `.content` is a list of `web_search_result` |
| Web fetch | `web_fetch_20260209` | `web_fetch` | `max_uses`, `allowed_domains`/`blocked_domains`, `citations`, `max_content_tokens` | `web_fetch_tool_result` -> `.content` is a `web_fetch_result` with a `document` block |
| Code execution | `code_execution_20260521` | `code_execution` | none | `bash_code_execution_tool_result` -> `.content.stdout` / `.stderr` / `.return_code` |
| Tool search (regex) | `tool_search_tool_regex_20251119` | `tool_search_tool_regex` | mark other tools `defer_loading: true` | `tool_search_tool_result` |
| Tool search (BM25) | `tool_search_tool_bm25_20251119` | `tool_search_tool_bm25` | mark other tools `defer_loading: true` | `tool_search_tool_result` |

| 工具 | `type` | `name` | 关键可选参数 | 结果块类型 |
|---|---|---|---|---|
| 网络搜索 | `web_search_20260209` | `web_search` | `max_uses`、`allowed_domains`/`blocked_domains`、`user_location` | `web_search_tool_result` -> `.content` 是 `web_search_result` 的列表 |
| 网络抓取 | `web_fetch_20260209` | `web_fetch` | `max_uses`、`allowed_domains`/`blocked_domains`、`citations`、`max_content_tokens` | `web_fetch_tool_result` -> `.content` 是带 `document` 块的 `web_fetch_result` |
| 代码执行 | `code_execution_20260521` | `code_execution` | 无 | `bash_code_execution_tool_result` -> `.content.stdout` / `.stderr` / `.return_code` |
| 工具搜索（正则） | `tool_search_tool_regex_20251119` | `tool_search_tool_regex` | 把其他工具标记为 `defer_loading: true` | `tool_search_tool_result` |
| 工具搜索（BM25） | `tool_search_tool_bm25_20251119` | `tool_search_tool_bm25` | 把其他工具标记为 `defer_loading: true` | `tool_search_tool_result` |

`web_search_20260209` / `web_fetch_20260209` have built-in dynamic filtering - code execution runs under the hood, so do **not** separately declare `code_execution` in `tools` (a second execution environment confuses the model). For models older than Opus 4.6 / Sonnet 4.6, use the basic variants `web_search_20250305` / `web_fetch_20250910` instead; on Vertex AI only basic `web_search_20250305` is available. `code_execution_20260120` (REPL persistence + programmatic tool calling) runs on Opus 4.5+ / Sonnet 4.5+. **Go SDK only**: `code_execution_20260521` lives under `client.Beta.Messages.New` with `Betas: []anthropic.AnthropicBeta{"code-execution-2025-08-25"}` (other languages use plain `client.messages.create`); `code_execution_20260120` uses the non-beta `client.Messages.New` in Go like everywhere else. Web fetch only fetches URLs already present in the conversation. Provider availability varies by tool - see `shared/platform-availability.md`. See `shared/tool-use-concepts.md` for `pause_turn` handling.

`web_search_20260209` / `web_fetch_20260209` 内置动态过滤——底层会运行代码执行，因此**不要**在 `tools` 中另行声明 `code_execution`（第二个执行环境会干扰模型）。对早于 Opus 4.6 / Sonnet 4.6 的模型，请改用基础变体 `web_search_20250305` / `web_fetch_20250910`；Vertex AI 上只有基础的 `web_search_20250305` 可用。`code_execution_20260120`（REPL 持久化 + 程序化工具调用）运行于 Opus 4.5+ / Sonnet 4.5+。**仅 Go SDK**：`code_execution_20260521` 位于 `client.Beta.Messages.New` 之下，并需 `Betas: []anthropic.AnthropicBeta{"code-execution-2025-08-25"}`（其他语言使用普通的 `client.messages.create`）；`code_execution_20260120` 在 Go 中与其他语言一样使用非 beta 的 `client.Messages.New`。web fetch 只抓取对话中已存在的 URL。各工具的厂商可用性不同——见 `shared/platform-availability.md`。`pause_turn` 处理见 `shared/tool-use-concepts.md`。

## Document & File Input (Quick Reference) / 文档与文件输入（速查）

**PDF (base64, no beta):** `{"type": "document", "source": {"type": "base64", "media_type": "application/pdf", "data": <b64 string>}}` in user content, placed before the text block. Base64 string must have no newlines. Limits: 32 MB request, 600 pages (100 for 200k-context models). Java: `ContentBlockParam.ofDocument(DocumentBlockParam... Base64PdfSource.builder().data(...))`.

**PDF（base64，无 beta）：**在用户内容中放入 `{"type": "document", "source": {"type": "base64", "media_type": "application/pdf", "data": <b64 string>}}`，并置于文本块之前。Base64 字符串不得含换行。限制：请求 32 MB，600 页（200k 上下文模型为 100 页）。Java：`ContentBlockParam.ofDocument(DocumentBlockParam... Base64PdfSource.builder().data(...))`。

**Files API (no beta):** upload via `client.files.upload(...)` -> response `id` is the `file_id`. Reference it as `{"type": "document", "source": {"type": "file", "file_id": "..."}}` for PDF/text, or `{"type": "image", ...}` for images - the content-block type must match the file's MIME type. To migrate code off `files-api-2025-04-14`, WebFetch the Files API row in `shared/live-sources.md`. Availability: `shared/platform-availability.md`.

**Files API（无 beta）：**通过 `client.files.upload(...)` 上传 -> 响应中的 `id` 即 `file_id`。PDF/文本以 `{"type": "document", "source": {"type": "file", "file_id": "..."}}` 引用，图片以 `{"type": "image", ...}` 引用——内容块类型必须与文件的 MIME 类型匹配。要把代码从 `files-api-2025-04-14` 迁走，请 WebFetch `shared/live-sources.md` 中的 Files API 行。可用性：`shared/platform-availability.md`。

**Citations (no beta):** set `citations: {enabled: true}` on each `document` content block (all or none). Response splits into multiple `text` blocks; cited blocks carry a `citations` array. Each citation has `cited_text`, `document_index`, `document_title`, and a location by `type`: `char_location` (`start_char_index`/`end_char_index`) for plain text, `page_location` (`start_page_number`/`end_page_number`, 1-indexed) for PDF, `content_block_location` for custom content. Incompatible with `output_config.format` (returns a 400).

**引用（citations，无 beta）：**在每个 `document` 内容块上设置 `citations: {enabled: true}`（要么全开要么全关）。响应会拆分为多个 `text` 块；被引用的块带有 `citations` 数组。每个引用含 `cited_text`、`document_index`、`document_title`，以及按 `type` 划分的位置：纯文本用 `char_location`（`start_char_index`/`end_char_index`），PDF 用 `page_location`（`start_page_number`/`end_page_number`，从 1 开始），自定义内容用 `content_block_location`。与 `output_config.format` 不兼容（返回 400）。

## Tool Use Patterns (Quick Reference) / 工具使用模式（速查）

**Strict tool use (no beta):** set `strict: true` as a top-level field on the tool definition (alongside `name`/`description`/`input_schema`), **not** on `tool_choice`. Schema must have `additionalProperties: false` + `required`. Guarantees `tool_use.input` validates exactly. Go: `Strict: anthropic.Bool(true)` + `additionalProperties` via `InputSchema.ExtraFields`; Java: `.strict(true)` + `.putAdditionalProperty("additionalProperties", JsonValue.from(false))`.

**严格工具使用（无 beta）：**把 `strict: true` 设为工具定义的顶层字段（与 `name`/`description`/`input_schema` 并列），**而不是**设在 `tool_choice` 上。schema 必须含 `additionalProperties: false` + `required`。这保证 `tool_use.input` 精确通过校验。Go：`Strict: anthropic.Bool(true)` + 通过 `InputSchema.ExtraFields` 传 `additionalProperties`；Java：`.strict(true)` + `.putAdditionalProperty("additionalProperties", JsonValue.from(false))`。

**Parallel tool use (default on):** one assistant message may contain multiple `tool_use` blocks. Execute them concurrently, then return **all** `tool_result` blocks in a **single** user message - splitting them across multiple messages silently trains Claude to stop making parallel calls. For a failed tool, return `tool_result` with `is_error: true` - don't drop it.

**并行工具使用（默认开启）：**一条 assistant 消息可含多个 `tool_use` 块。并发执行它们，然后在**单条** user 消息中返回**全部** `tool_result` 块——把它们拆到多条消息会静默地训练 Claude 停止发起并行调用。工具失败时，返回 `is_error: true` 的 `tool_result`——不要丢弃它。

**Tool Runner (SDK beta helper):** drives the tool-call loop for you via `client.beta.messages.*`. Python: `@beta_tool` decorator + `client.beta.messages.tool_runner(...)` -> `runner.until_done()`. TypeScript: `betaZodTool({...})` from `@anthropic-ai/sdk/helpers/beta/zod` + `client.beta.messages.toolRunner(...)` -> `await runner`. Go: `toolrunner.NewBetaToolFromJSONSchema(...)` + `client.Beta.Messages.NewToolRunner(...)` -> `.RunToCompletion(ctx)`. Java requires `.addBeta("structured-outputs-2025-11-13")`. Ruby: `Anthropic::BaseTool` subclass + `client.beta.messages.tool_runner(...)`. PHP: `BetaRunnableTool` + `->toolRunner(...)`. C#: raw JSON-schema tools + `BetaToolRunner` via `client.Beta.Messages.ToolRunner(...)`.

**Tool Runner（SDK beta 辅助器）：**通过 `client.beta.messages.*` 替你驱动工具调用循环。Python：`@beta_tool` 装饰器 + `client.beta.messages.tool_runner(...)` -> `runner.until_done()`。TypeScript：来自 `@anthropic-ai/sdk/helpers/beta/zod` 的 `betaZodTool({...})` + `client.beta.messages.toolRunner(...)` -> `await runner`。Go：`toolrunner.NewBetaToolFromJSONSchema(...)` + `client.Beta.Messages.NewToolRunner(...)` -> `.RunToCompletion(ctx)`。Java 需要 `.addBeta("structured-outputs-2025-11-13")`。Ruby：`Anthropic::BaseTool` 子类 + `client.beta.messages.tool_runner(...)`。PHP：`BetaRunnableTool` + `->toolRunner(...)`。C#：原始 JSON-schema 工具 + 经由 `client.Beta.Messages.ToolRunner(...)` 的 `BetaToolRunner`。

**Programmatic tool calling (no beta header):** Claude calls your custom tool from inside code execution. Add `{"type": "code_execution_20260120", "name": "code_execution"}` **and** set `"allowed_callers": ["code_execution_20260120"]` on your custom tool. Opus 4.5+ / Sonnet 4.5+ (availability: `shared/platform-availability.md`). When responding to a pending programmatic call, the user message must contain **only** `tool_result` blocks (no text). Not compatible with `strict: true`, `disable_parallel_tool_use`, forced `tool_choice`, or MCP tools.

**程序化工具调用（无 beta 头）：**Claude 在代码执行内部调用你的自定义工具。添加 `{"type": "code_execution_20260120", "name": "code_execution"}`，**并且**在自定义工具上设置 `"allowed_callers": ["code_execution_20260120"]`。Opus 4.5+ / Sonnet 4.5+（可用性：`shared/platform-availability.md`）。响应待处理的程序化调用时，user 消息必须**只**包含 `tool_result` 块（不含文本）。与 `strict: true`、`disable_parallel_tool_use`、强制 `tool_choice` 或 MCP 工具不兼容。

## Other API Surfaces (Quick Reference) / 其他 API 表面（速查）

**Message Batches (no beta; availability: `shared/platform-availability.md`):** `client.messages.batches.create(requests=[{custom_id, params}, ...])` -> poll `client.messages.batches.retrieve(id).processing_status` until `"ended"` -> stream `client.messages.batches.results(id)`. Each result has `.custom_id` + `.result.type` (`succeeded`/`errored`/`canceled`/`expired`); on success read `.result.message.content`. Python wraps requests as `Request(custom_id=..., params=MessageCreateParamsNonStreaming(...))`. Results arrive in **any order** - key by `custom_id`, never by position.

**消息批处理（无 beta；可用性：`shared/platform-availability.md`）：**`client.messages.batches.create(requests=[{custom_id, params}, ...])` -> 轮询 `client.messages.batches.retrieve(id).processing_status` 直至 `"ended"` -> 以流式读取 `client.messages.batches.results(id)`。每个结果含 `.custom_id` + `.result.type`（`succeeded`/`errored`/`canceled`/`expired`）；成功时读取 `.result.message.content`。Python 把请求包装为 `Request(custom_id=..., params=MessageCreateParamsNonStreaming(...))`。结果以**任意顺序**到达——按 `custom_id` 作键，绝不要按位置。

**Models API (no beta; availability: `shared/platform-availability.md`):** `client.models.list()` (auto-paginates) and `client.models.retrieve("claude-opus-5-5")`. Each model object has `id`, `display_name`, `created_at`, and - since Mar 2026 - `max_input_tokens` (the context window), `max_tokens` (the output cap), and `capabilities`. There is no `context_window` field.

**Models API（无 beta；可用性：`shared/platform-availability.md`）：**`client.models.list()`（自动分页）与 `client.models.retrieve("claude-opus-5-5")`。每个模型对象含 `id`、`display_name`、`created_at`，以及自 2026 年 3 月起的 `max_input_tokens`（上下文窗口）、`max_tokens`（输出上限）与 `capabilities`。没有 `context_window` 字段。

**Stop details (GA, Opus 4.7+):** `response.stop_details` is populated **only when `stop_reason == "refusal"`** (fields: `type: "refusal"`, `category` - an open set, e.g. `"cyber"`, `"bio"`, `"reasoning_extraction"`, `"frontier_llm"`, or `null`; see the docs for the full list - and `explanation`). It is `null` for every other `stop_reason` (`end_turn`, `max_tokens`, `tool_use`, `pause_turn`, ...) - always guard before reading.

**停止详情（正式发布，Opus 4.7+）：**`response.stop_details` **仅在 `stop_reason == "refusal"` 时被填充**（字段：`type: "refusal"`、`category`——一个开放集合，例如 `"cyber"`、`"bio"`、`"reasoning_extraction"`、`"frontier_llm"` 或 `null`；完整清单见文档——以及 `explanation`）。对其他任何 `stop_reason`（`end_turn`、`max_tokens`、`tool_use`、`pause_turn` 等）它都是 `null`——读取前务必先做保护判断。

**Admin API (beta, since 2026-08-26):** organization management - members, invites, workspaces and workspace members, API keys, rate limit reports, service accounts, federation issuers/rules, CMEK external keys - under `client.beta.organization` in all seven SDKs and `ant beta:organization` in the CLI. Requires an admin credential: an Admin API key (`sk-ant-admin...`, read from `ANTHROPIC_API_KEY`) or an `org:admin` OAuth token (`ANTHROPIC_AUTH_TOKEN`); regular API keys are rejected. Usage and cost reports and the Claude Enterprise user-management/analytics endpoints are **not** in the SDKs - raw HTTP only. See `shared/admin-api.md`.

**Admin API（beta，自 2026-08-26 起）：**组织管理——成员、邀请、工作区与工作区成员、API 密钥、限流报告、服务账号、联合签发者/规则、CMEK 外部密钥——位于全部七个 SDK 的 `client.beta.organization` 与 CLI 的 `ant beta:organization` 之下。需要管理员凭据：Admin API 密钥（`sk-ant-admin...`，从 `ANTHROPIC_API_KEY` 读取）或 `org:admin` OAuth 令牌（`ANTHROPIC_AUTH_TOKEN`）；普通 API 密钥会被拒绝。用量与成本报告以及 Claude Enterprise 的用户管理/分析端点**不在** SDK 中——只能用原始 HTTP。见 `shared/admin-api.md`。

**Client config (no beta):** `timeout` default 10 min; **units differ by SDK** - Python/Ruby: seconds; TypeScript: **milliseconds**; Go `option.WithRequestTimeout(time.Duration)`; Java `Duration`; C# `TimeSpan`. TS scales the default up to 60 min for large `max_tokens` on non-streaming requests; Java does so for streaming requests (Java non-streaming scales 30s-10 min). `max_retries`/`maxRetries` default 2 (retries 408/409/429/5xx + connection errors). `base_url` (or `ANTHROPIC_BASE_URL` env). Per-request override: Python `client.with_options(timeout=5.0).messages.create(...)`; TS `client.messages.create({...}, {timeout: 5_000})`; Ruby `request_options: {timeout: 5}`. Timeouts are retried - wall-clock can reach `timeout × (max_retries+1)`.

**客户端配置（无 beta）：**`timeout` 默认 10 分钟；**各 SDK 单位不同**——Python/Ruby：秒；TypeScript：**毫秒**；Go `option.WithRequestTimeout(time.Duration)`；Java `Duration`；C# `TimeSpan`。TS 会在非流式请求的 `max_tokens` 较大时把默认值放大到 60 分钟；Java 对流式请求如此（Java 非流式在 30 秒-10 分钟间缩放）。`max_retries`/`maxRetries` 默认 2（重试 408/409/429/5xx + 连接错误）。`base_url`（或 `ANTHROPIC_BASE_URL` 环境变量）。按请求覆盖：Python `client.with_options(timeout=5.0).messages.create(...)`；TS `client.messages.create({...}, {timeout: 5_000})`；Ruby `request_options: {timeout: 5}`。超时会重试——墙钟时间可达 `timeout × (max_retries+1)`。

## Workload Identity Federation (Quick Reference) / 工作负载身份联合（速查）

**GA, no beta header.** Construct the normal zero-arg client (`Anthropic()` / `new Anthropic()` / `anthropic.NewClient()` / `AnthropicOkHttpClient.fromEnv()`); the SDK auto-detects WIF when **all** of `ANTHROPIC_FEDERATION_RULE_ID`, `ANTHROPIC_ORGANIZATION_ID`, `ANTHROPIC_SERVICE_ACCOUNT_ID`, and `ANTHROPIC_IDENTITY_TOKEN_FILE` (or `ANTHROPIC_IDENTITY_TOKEN`) are set, exchanges the JWT at `/v1/oauth/token`, and auto-refreshes. `ANTHROPIC_WORKSPACE_ID` does not gate activation - required only when the federation rule spans multiple workspaces (else 400 `workspace_id_required`), optional for single-workspace rules. `ANTHROPIC_API_KEY` or `ANTHROPIC_AUTH_TOKEN` (even empty) outrank WIF, and a set `ANTHROPIC_PROFILE` also wins over the federation env vars (a missing named profile is an error, not a fall-through) - unset all three.

**正式发布，无需 beta 头。**构造普通的零参数客户端（`Anthropic()` / `new Anthropic()` / `anthropic.NewClient()` / `AnthropicOkHttpClient.fromEnv()`）；当 `ANTHROPIC_FEDERATION_RULE_ID`、`ANTHROPIC_ORGANIZATION_ID`、`ANTHROPIC_SERVICE_ACCOUNT_ID` 与 `ANTHROPIC_IDENTITY_TOKEN_FILE`（或 `ANTHROPIC_IDENTITY_TOKEN`）**全部**设置时，SDK 会自动检测 WIF，在 `/v1/oauth/token` 交换 JWT，并自动刷新。`ANTHROPIC_WORKSPACE_ID` 不影响激活——仅当联合规则跨多个工作区时必填（否则 400 `workspace_id_required`），单工作区规则可选。`ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN`（即使是空值）优先级高于 WIF，已设置的 `ANTHROPIC_PROFILE` 也胜过联合环境变量（缺失的命名配置是错误，不是回退）——请把这三者全部取消设置。

---

## Reading Guide / 阅读指南

After detecting the language, read the relevant files based on what the user needs. Every `{lang}/...`, `shared/...`, and `curl/...` path cited in this document is relative to this skill's base directory, and none of those files' content is included above - Read each one on demand before relying on what it covers.

检测语言之后，根据用户需要阅读相关文件。本文中引用的每个 `{lang}/...`、`shared/...` 与 `curl/...` 路径都相对于本技能的基目录，且上述内容并未包含这些文件的内容——在依赖某文件所覆盖的内容之前，请按需先 Read 它。

**All SDK languages use the same multi-file layout** - directory `{lang}/claude-api/` containing `README.md` (install, client init, basic request, thinking, caching, stop details, misc), `tool-use.md` (tool definitions, agentic loop, Anthropic-defined tools, structured outputs), `streaming.md`, `batches.md`, `files-api.md`. Not every language has every file (e.g., Ruby has no `batches.md`); if a file is absent, that feature's example is not yet documented for that language - fall back to the cURL shape or WebFetch the SDK repo from `shared/live-sources.md`. **cURL** -> `curl/examples.md`.

**所有 SDK 语言使用相同的多文件布局**——目录 `{lang}/claude-api/` 包含 `README.md`（安装、客户端初始化、基础请求、思考、缓存、停止详情、杂项）、`tool-use.md`（工具定义、智能体循环、Anthropic 定义的工具、结构化输出）、`streaming.md`、`batches.md`、`files-api.md`。并非每种语言都有每个文件（例如 Ruby 没有 `batches.md`）；若某文件缺失，说明该功能的示例尚未为该语言撰写——回退到 cURL 形态，或从 `shared/live-sources.md` WebFetch SDK 仓库。**cURL** -> `curl/examples.md`。

The Quick Task Reference below uses the `{lang}/claude-api/FILE.md` path notation for all languages.

下方的快速任务参考对所有语言使用 `{lang}/claude-api/FILE.md` 路径记法。

### Quick Task Reference / 快速任务参考

**Single text classification/summarization/extraction/Q&A:**  
**单次文本分类/摘要/抽取/问答：**  
-> Read only `{lang}/claude-api/README.md` - **always read the README first** for any task (installation, quick start, common patterns, error handling)
-> 只读 `{lang}/claude-api/README.md`——任何任务都**先读 README**（安装、快速上手、常见模式、错误处理）

**Chat UI or real-time response display:**  
**聊天 UI 或实时响应展示：**  
-> Read `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`
-> 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**Long-running conversations (may exceed context window):**  
**长时对话（可能超出上下文窗口）：**  
-> Read `{lang}/claude-api/README.md` - see Compaction section
-> 阅读 `{lang}/claude-api/README.md`——见 Compaction 一节

**Migrating to a newer model (Sonnet 5.5 / Opus 5.5 / Fable 5.1 / Fable 5 / Opus 5 / Opus 4.8 / Opus 4.7 / Opus 4.6 / Sonnet 5 / Sonnet 4.6), replacing a retired model, or translating `budget_tokens` / prefill patterns to the current API:**  
**迁移到更新的模型（Sonnet 5.5 / Opus 5.5 / Fable 5.1 / Fable 5 / Opus 5 / Opus 4.8 / Opus 4.7 / Opus 4.6 / Sonnet 5 / Sonnet 4.6）、替换已退役的模型，或把 `budget_tokens` / 预填充模式改写为当前 API：**  
-> Read `shared/model-migration.md`
-> 阅读 `shared/model-migration.md`

**Upgrading the Anthropic SDK package itself across a major version (`anthropic` 0.x -> 1.x: `httpx2`, awaited async `.with_raw_response`, removed deprecated parameters / aliases / Text Completions, Python >= 3.10) - or writing new code against a project already on 1.x:**  
**跨主版本升级 Anthropic SDK 包本身（`anthropic` 0.x -> 1.x：`httpx2`、可 await 的异步 `.with_raw_response`、移除已弃用的参数/别名/Text Completions、要求 Python >= 3.10）——或针对已在 1.x 上的项目编写新代码：**  
-> Read `{lang}/claude-api/sdk-upgrade.md` (currently Python only; other SDKs have no bundled major-version guide yet - use that SDK's CHANGELOG via `shared/live-sources.md`)
-> 阅读 `{lang}/claude-api/sdk-upgrade.md`（目前仅 Python；其他 SDK 尚无附带的主版本升级指南——通过 `shared/live-sources.md` 查阅该 SDK 的 CHANGELOG）

**Building an eval set for a Claude app (or "how do I know if my change helped"):**  
**为 Claude 应用构建评测集（或"我怎么知道我的改动有没有帮助"）：**  
-> Read `shared/evals/build-eval.md` - it loads `shared/evals/eval-audit.md` (the health checklist every eval must satisfy) before Step 0.
-> 阅读 `shared/evals/build-eval.md`——它在 Step 0 之前会加载 `shared/evals/eval-audit.md`（每个评测都必须满足的健康清单）。

**Checking whether an existing eval is trustworthy ("is my eval any good?"):**  
**检查现有评测是否可信（"我的评测靠谱吗"）：**  
-> Read `shared/evals/eval-audit.md` and run it against the eval; report per its section 6.
-> 阅读 `shared/evals/eval-audit.md` 并对照该评测执行；按其第 6 节报告。

**Iteratively improving an app against an eval (prompt tuning, hill-climbing):**  
**对照评测迭代改进应用（提示词调优、爬山法）：**  
-> Read `shared/evals/eval-hillclimb.md` - runs Step 0 -> Step 5 with a train/test split; test is scored every round and is the headline.
-> 阅读 `shared/evals/eval-hillclimb.md`——按训练/测试切分运行 Step 0 -> Step 5；每轮都对测试集评分，并以它为准。

**Rendering an eval-hillclimb HTML report:**  
**渲染 eval-hillclimb HTML 报告：**  
-> Run `shared/evals/report/build-report.mjs` when it is on disk (EAP install), else `shared/evals/report/build-report-lite.mjs` (always extracted with this skill) - both consume the `_state.json` / `vN/` layout produced by the hillclimb guide and write the same `trajectory/scores.tsv`. Don't write a parallel one.
-> 若 `shared/evals/report/build-report.mjs` 在磁盘上（EAP 安装）则运行它，否则运行 `shared/evals/report/build-report-lite.mjs`（随本技能必然解包）——二者都消费 hillclimb 指南产出的 `_state.json` / `vN/` 布局，并写出相同的 `trajectory/scores.tsv`。不要另写一个平行实现。

**Migrating to, prompting, or tuning Claude Opus 5.5 (thinking can't be disabled, effort tuning and the `medium` default, forced tool use, computer toolset, progress updates, safeguard false positives, visual inputs / design outputs):**  
**迁移到 Claude Opus 5.5、为其撰写提示词或调优（思考无法关闭、effort 调优与 `medium` 默认值、强制工具使用、计算机工具集、进度更新、安全防护误报、视觉输入 / 设计类输出）：**  
-> Read `shared/model-migration.md` -> Migrating to Claude Opus 5.5; the preserved-thinking mechanics it points at are under Migrating to Claude Fable 5.1 from Claude Fable 5
-> 阅读 `shared/model-migration.md` -> Migrating to Claude Opus 5.5；其指向的 preserved-thinking 机制位于 Migrating to Claude Fable 5.1 from Claude Fable 5 之下

**Migrating to, prompting, or tuning Claude Sonnet 5.5 (`between_tools` instead of disabled thinking, recalibrated effort, forced tool use, computer toolset, advisor pairings, progress updates, tool use in chat, mid-turn user messages, verification at low effort, safeguard categories):**  
**迁移到 Claude Sonnet 5.5、为其撰写提示词或调优（以 `between_tools` 代替关闭思考、重新校准的 effort、强制工具使用、计算机工具集、advisor 配对、进度更新、聊天中的工具使用、轮中 user 消息、低 effort 下的验证、安全防护类别）：**  
-> Read `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5
-> 阅读 `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5

**Prompting or tuning Fable 5/5.1 (long turns, effort, verbosity, autonomous runs, sub-agents):**  
**为 Fable 5/5.1 撰写提示词或调优（长轮次、effort、冗长度、自主运行、子智能体）：**  
-> Read `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> Behavioral shifts (prompt-tunable) + Long-running agent recommendations
-> 阅读 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> Behavioral shifts (prompt-tunable) + Long-running agent recommendations

**Prompting or tuning Claude Fable 5.1 (progress updates, parallel tool calls, writing density / formatting, autonomy, test sprawl, whole-file rewrites) or making a harness compatible with preserved thinking's history-editing check (history edits, compaction, per-turn reminders):**  
**为 Claude Fable 5.1 撰写提示词或调优（进度更新、并行工具调用、写作密度 / 格式、自主性、测试蔓延、整文件重写），或让执行框架兼容 preserved thinking 的历史编辑检查（历史编辑、压缩、每轮提醒）：**  
-> Read `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features + Behavioral shifts (prompt-tunable); for the history-editing check itself (the three-step check, the append-only edit table, compaction shapes), Breaking change 3 in the same section; to find, measure and fix the edits an *existing* harness makes (capture, diff, replay with `drop_block`, one fix per cause, model switches), run `preserved-thinking-migration` (Subcommands table) - it reads `shared/preserved-thinking-migration.md`
-> 阅读 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features + Behavioral shifts (prompt-tunable)；历史编辑检查本身（三步检查、只追加编辑表、压缩形态）见同一节中的 Breaking change 3；要发现、测量并修复*现有*执行框架所做的编辑（捕获、差分、以 `drop_block` 重放、每个成因一个修复、模型切换），运行 `preserved-thinking-migration`（子命令表）——它会读取 `shared/preserved-thinking-migration.md`

**Prompt caching / optimize caching / "why is my cache hit rate low":**  
**提示词缓存 / 优化缓存 / "为什么我的缓存命中率低"：**  
-> Read `shared/prompt-caching.md` (prefix-stability design, breakpoint placement, anti-patterns that silently invalidate cache) + `{lang}/claude-api/README.md` (Prompt Caching section)
-> 阅读 `shared/prompt-caching.md`（前缀稳定性设计、断点放置、静默使缓存失效的反模式）+ `{lang}/claude-api/README.md`（Prompt Caching 一节）

**Auditing or cleaning up prompts, tool descriptions, skills, or agent configuration files such as `CLAUDE.md` ("is this prompt outdated", "remove the cruft", "this was written for an older model"):**  
**审计或清理提示词、工具描述、技能或诸如 `CLAUDE.md` 的智能体配置文件（"这个提示词过时了吗"、"清除陈旧残留"、"这是为旧模型写的"）：**  
-> Read `shared/prompt-audit.md` - dated-pattern tables with greppable signals, the keep list (what NOT to delete), and the report + proposed-diff output contract
-> 阅读 `shared/prompt-audit.md`——带可 grep 信号的过时模式表、保留清单（哪些**不能**删），以及报告 + 建议 diff 的输出契约

**Count tokens in a file / prompt / diff ("how many tokens is X"):**  
**统计文件 / 提示词 / diff 的 token 数（"X 有多少 token"）：**  
-> Read `shared/token-counting.md` - use `messages.count_tokens`, never `tiktoken`
-> 阅读 `shared/token-counting.md`——使用 `messages.count_tokens`，绝不要用 `tiktoken`

**Reducing or reviewing API spend ("the bill is too high", "make this cheaper", "am I overspending", cost per completed task, cheapest model or effort that holds quality):**  
**降低或审查 API 开销（"账单太高了"、"让这个更便宜"、"我是不是花超了"、按完成任务计算的成本、在保住质量前提下最便宜的模型或 effort）：**  
-> Read `shared/cost-optimization.md` - baseline and token profile first, then the levers in order (free wins before tradeoffs) with measured expectations, and a workload-shape -> lever mapping table
-> 阅读 `shared/cost-optimization.md`——先建立基线与 token 画像，再按顺序使用各杠杆（免费收益先于权衡）并给出实测预期，另有工作负载形态 -> 杠杆的映射表

**Function calling / tool use / agents:**  
**函数调用 / 工具使用 / 智能体：**  
-> Read `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` (conceptual foundations: function calling, code execution, memory, structured outputs) + `{lang}/claude-api/tool-use.md` (language-specific code examples: tool runner, manual loop, code execution, memory, structured outputs)
-> 阅读 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md`（概念基础：函数调用、代码执行、记忆、结构化输出）+ `{lang}/claude-api/tool-use.md`（各语言代码示例：tool runner、手写循环、代码执行、记忆、结构化输出）

**Agent design (tool surface, context management, caching strategy):**  
**智能体设计（工具表面、上下文管理、缓存策略）：**  
-> Read `shared/agent-design.md` (bash vs. dedicated tools, programmatic tool calling, tool search/skills, context editing vs. compaction vs. memory, caching principles)
-> 阅读 `shared/agent-design.md`（bash 对比专用工具、程序化工具调用、工具搜索/skills、上下文编辑 对比 压缩 对比 记忆、缓存原则）

**Batch processing (non-latency-sensitive; runs asynchronously at 50% cost):**  
**批处理（对延迟不敏感；以 50% 成本异步运行）：**  
-> Read `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`
-> 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**File uploads across multiple requests (same file without re-uploading):**  
**跨多个请求复用上传的文件（同一文件无需重复上传）：**  
-> Read `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`
-> 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**Organization administration (members, invites, workspaces, API keys, rate limit reports, service accounts, WIF resources, CMEK):**  
**组织管理（成员、邀请、工作区、API 密钥、限流报告、服务账号、WIF 资源、CMEK）：**  
-> Read `shared/admin-api.md` - `client.beta.organization` endpoint/method table, admin credentials, per-language naming and pagination, what stays curl-only
-> 阅读 `shared/admin-api.md`——`client.beta.organization` 端点/方法表、管理员凭据、各语言命名与分页、哪些仍只能用 curl

**Debugging HTTP errors or implementing error handling:**  
**调试 HTTP 错误或实现错误处理：**  
-> Read `shared/error-codes.md` - per-SDK typed exception class table and the Go `errors.As` pattern
-> 阅读 `shared/error-codes.md`——各 SDK 的类型化异常类表与 Go 的 `errors.As` 模式

**Latest official documentation:**  
**最新官方文档：**  
-> WebFetch the URLs in `shared/live-sources.md`
-> WebFetch `shared/live-sources.md` 中的 URL

**Managed Agents (server-managed stateful agents with workspace):**  
**Managed Agents（带工作区的服务器管理有状态智能体）：**  
-> See the reading guide in the `## Managed Agents (Beta)` section above - it lists every `shared/managed-agents-*.md` file and the language-specific READMEs (`{lang}/managed-agents/README.md`, `curl/managed-agents.md`).
-> 见上方 `## Managed Agents (Beta)` 章节中的阅读指南——它列出了每个 `shared/managed-agents-*.md` 文件与各语言 README（`{lang}/managed-agents/README.md`、`curl/managed-agents.md`）。

---

## When to Use WebFetch / 何时使用 WebFetch

Use WebFetch to get the latest documentation when:

在以下情况使用 WebFetch 获取最新文档：

- User asks for "latest" or "current" information
  用户要求"最新"或"当前"信息时
- Cached data seems incorrect
  缓存数据看起来不正确时
- User asks about features not covered here
  用户询问此处未覆盖的功能时

Live documentation URLs are in `shared/live-sources.md`.

实时文档 URL 见 `shared/live-sources.md`。

## Common Pitfalls / 常见陷阱

- Don't truncate inputs when passing files or content to the API. If the content is too long to fit in the context window, notify the user and discuss options (chunking, summarization, etc.) rather than silently truncating.
  向 API 传递文件或内容时不要截断输入。如果内容太长放不进上下文窗口，请通知用户并讨论选项（分块、摘要等），而不是静默截断。
- **Prefill removed (Fable 5, Claude Fable 5.1, Opus 5, Claude Opus 5.5, Sonnet 5, Claude Sonnet 5.5, and the 4.6/4.7/4.8 family):** Assistant message prefills (last-assistant-turn prefills) return a 400 error on Fable 5, Claude Fable 5.1, Opus 5, Claude Opus 5.5, Sonnet 5, Claude Sonnet 5.5, Opus 4.6, Opus 4.7, Opus 4.8, and Sonnet 4.6. Use structured outputs (`output_config.format`) or system prompt instructions to control response format instead. (One exception: the fallback-credit prefill claim - when redeeming a credit with `fallback_has_prefill_claim: true`, the server accepts the echoed assistant message; see the migration guide's refusal section.)
  **预填充已移除（Fable 5、Claude Fable 5.1、Opus 5、Claude Opus 5.5、Sonnet 5、Claude Sonnet 5.5 及 4.6/4.7/4.8 家族）：**assistant 消息预填充（最后一轮 assistant 预填充）在 Fable 5、Claude Fable 5.1、Opus 5、Claude Opus 5.5、Sonnet 5、Claude Sonnet 5.5、Opus 4.6、Opus 4.7、Opus 4.8 与 Sonnet 4.6 上返回 400 错误。请改用结构化输出（`output_config.format`）或系统提示词指令来控制响应格式。（一个例外：后备额度预填充主张——以 `fallback_has_prefill_claim: true` 兑换额度时，服务器接受回传的 assistant 消息；见迁移指南的 refusal 一节。）
- **Confirm migration scope before editing:** When a user asks to migrate code to a newer Claude model without naming a specific file, directory, or file list, **ask which scope to apply first** - the entire working directory, a specific subdirectory, or a specific set of files. Do not start editing until the user confirms. Imperative phrasings like "migrate my codebase", "move my project to X", "upgrade to Sonnet 4.6", or bare "migrate to Opus 4.8" are **still ambiguous** - they tell you what to do but not where, so ask. Proceed without asking only when the prompt names an exact file, a specific directory, or an explicit file list ("migrate `app.py`", "migrate everything under `services/`", "update `a.py` and `b.py`"). See `shared/model-migration.md` Step 0.
  **编辑前先确认迁移范围：**当用户要求把代码迁移到更新的 Claude 模型而未指明具体文件、目录或文件清单时，**先询问要应用的范围**——整个工作目录、某个特定子目录，还是一组特定文件。用户确认之前不要开始编辑。"migrate my codebase"、"move my project to X"、"upgrade to Sonnet 4.6" 或光秃秃的 "migrate to Opus 4.8" 这类命令式说法**仍然含糊**——它们告诉你要做什么，却没说在哪里做，所以要问。只有当提示词点名了确切的文件、特定的目录或明确的文件清单（"migrate `app.py`"、"migrate everything under `services/`"、"update `a.py` and `b.py`"）时才可不问而行。见 `shared/model-migration.md` Step 0。
- **`max_tokens` defaults:** Don't lowball `max_tokens` - hitting the cap truncates output mid-thought and requires a retry. For non-streaming requests, default to `~16000` (keeps responses under SDK HTTP timeouts). For streaming requests, default to `~64000` (timeouts aren't a concern, so give the model room). Only go lower when you have a hard reason: classification (`~256`), cost caps, deliberately short outputs, or **`max_tokens: 0`** for cache pre-warming (see `shared/prompt-caching.md` -> Pre-warming).
  **`max_tokens` 默认值：**不要把 `max_tokens` 报低——触及上限会在思路中途截断输出并需要重试。非流式请求默认 `~16000`（让响应保持在 SDK HTTP 超时之内）。流式请求默认 `~64000`（超时不再是问题，给模型留足空间）。只有确有硬性理由时才更低：分类（`~256`）、成本上限、刻意短输出，或用于缓存预热的 **`max_tokens: 0`**（见 `shared/prompt-caching.md` -> Pre-warming）。
- **Disabling thinking on Claude Opus 5 has two failure modes - prefer low/medium effort instead.** (On Claude Opus 5.5 it can't be disabled at all - `{type: "disabled"}` is a 400 at every effort level; use `low` effort. On Claude Sonnet 5.5, `{type: "disabled"}` is also a 400 - try thinking on at `low` effort first, and if a route must stay thinking-off, send `{type: "between_tools"}` at `high` effort or below.) Only affects code that explicitly opts out; thinking is on by default, so watch for a disabled-thinking setting carried forward from Opus 4.8. With `thinking: {type: "disabled"}`, the model occasionally writes a tool call into its **visible text** instead of a `tool_use` block: the turn succeeds, the call never runs, no error is raised, and in an agentic loop that text pollutes later turns. It can also leak `<thinking>` tags into the response. Turning thinking on and lowering `effort` fixes both and still cuts cost. If a route must stay thinking-off: **delete** any don't-think/don't-reason rule (it makes tag leakage worse), don't name thinking tags, and add the combined instruction *"When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response."* Details: `shared/model-migration.md` -> Two failure modes when thinking is disabled.
  **在 Claude Opus 5 上关闭思考有两种失败模式——更推荐改用 low/medium effort。**（在 Claude Opus 5.5 上它完全无法关闭——`{type: "disabled"}` 在每个 effort 级别都是 400；请用 `low` effort。在 Claude Sonnet 5.5 上 `{type: "disabled"}` 同样是 400——先尝试以 `low` effort 开启思考，若某条路由必须保持无思考，则在 `high` effort 或以下发送 `{type: "between_tools"}`。）这只影响显式选择退出的代码；思考默认开启，所以要留意从 Opus 4.8 沿袭下来的关闭思考设置。设置 `thinking: {type: "disabled"}` 时，模型偶尔会把工具调用写进**可见文本**而不是 `tool_use` 块：该轮成功、调用从未运行、不抛任何错误，而在智能体循环中这段文本会污染后续轮次。它还可能把 `<thinking>` 标签泄漏进响应。开启思考并调低 `effort` 能同时解决这两个问题且仍然省成本。如果某条路由必须保持无思考：**删除**任何"不要思考/不要推理"的规则（它会使标签泄漏更严重）、不要点名思考标签，并添加组合指令 *"When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response."* 详情：`shared/model-migration.md` -> Two failure modes when thinking is disabled。
- **128K output tokens:** Fable 5, Claude Fable 5.1, Opus 5, Claude Opus 5.5, Opus 4.6, Opus 4.7, Opus 4.8, Claude Sonnet 5.5, Sonnet 5, and Sonnet 4.6 support up to 128K `max_tokens`, but the SDKs require streaming for values that large to avoid HTTP timeouts. Use `.stream()` with `.get_final_message()` / `.finalMessage()`.
  **128K 输出 token：**Fable 5、Claude Fable 5.1、Opus 5、Claude Opus 5.5、Opus 4.6、Opus 4.7、Opus 4.8、Claude Sonnet 5.5、Sonnet 5 与 Sonnet 4.6 支持最高 128K 的 `max_tokens`，但 SDK 要求这么大的值必须使用流式以避免 HTTP 超时。请使用 `.stream()` 配合 `.get_final_message()` / `.finalMessage()`。
- **Forced tool use removed (Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5):** `tool_choice: {type: "any"}` and `{type: "tool", name: ...}` return a 400 (`tool_choice: type "tool" and "any" are not supported for this model.`), on `count_tokens` and Batches too. Use `{type: "auto"}` plus an explicit instruction naming the tool, `strict: true` on the tool to keep schema-valid arguments, or structured outputs (`output_config.format`) when the forced call only existed to get JSON back. `{type: "none"}` is unaffected; `disable_parallel_tool_use` still works with `auto` (at most one call).
  **强制工具使用已移除（Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5）：**`tool_choice: {type: "any"}` 与 `{type: "tool", name: ...}` 返回 400（`tool_choice: type "tool" and "any" are not supported for this model.`），在 `count_tokens` 与 Batches 上同样如此。请使用 `{type: "auto"}` 加上点名工具的明确指令、在工具上设置 `strict: true` 以保证参数符合 schema，或在强制调用只是为了拿回 JSON 时使用结构化输出（`output_config.format`）。`{type: "none"}` 不受影响；`disable_parallel_tool_use` 配合 `auto` 仍然有效（至多一次调用）。
- **Tool call JSON parsing (Fable 5, Claude Fable 5.1, Opus 5, Claude Opus 5.5, and the 4.6/4.7/4.8 family):** Fable 5, Claude Fable 5.1, Opus 5, Claude Opus 5.5, Opus 4.6, Opus 4.7, Opus 4.8, and Sonnet 4.6 may produce different JSON string escaping in tool call `input` fields (e.g., Unicode or forward-slash escaping). Always parse tool inputs with `json.loads()` / `JSON.parse()` - never do raw string matching on the serialized input.
  **工具调用 JSON 解析（Fable 5、Claude Fable 5.1、Opus 5、Claude Opus 5.5 及 4.6/4.7/4.8 家族）：**Fable 5、Claude Fable 5.1、Opus 5、Claude Opus 5.5、Opus 4.6、Opus 4.7、Opus 4.8 与 Sonnet 4.6 在工具调用 `input` 字段中可能产生不同的 JSON 字符串转义（例如 Unicode 或正斜杠转义）。务必用 `json.loads()` / `JSON.parse()` 解析工具输入——绝不要对序列化后的输入做原始字符串匹配。
- **Structured outputs (all models):** Use `output_config: {format: {...}}` instead of the deprecated `output_format` parameter on `messages.create()`. This is a general API change, not 4.6-specific.
  **结构化输出（所有模型）：**在 `messages.create()` 上使用 `output_config: {format: {...}}` 而不是已弃用的 `output_format` 参数。这是通用的 API 变更，不限于 4.6。
- **Don't reimplement SDK functionality:** The SDK provides high-level helpers - use them instead of building from scratch. Specifically: use `stream.finalMessage()` instead of wrapping `.on()` events in `new Promise()`; use typed exception classes (`Anthropic.RateLimitError`, etc.) instead of string-matching error messages; use SDK types (`Anthropic.MessageParam`, `Anthropic.Tool`, `Anthropic.Message`, etc.) instead of redefining equivalent interfaces.
  **不要重新实现 SDK 功能：**SDK 提供了高层辅助方法——使用它们而不是从零造轮子。具体而言：用 `stream.finalMessage()` 而不是把 `.on()` 事件包进 `new Promise()`；用类型化异常类（`Anthropic.RateLimitError` 等）而不是对错误消息做字符串匹配；用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.Message` 等）而不是重新定义等价接口。
- **Error handling - catch a chain, not one broad class.** A single `except APIStatusError` / `catch (AnthropicServiceException)` / `rescue APIError` loses the distinction between retryable (429, >=500, network) and non-retryable (400/404) failures. Write a most-specific-first chain - e.g. `NotFoundError` -> `RateLimitError` -> `APIStatusError` -> `APIConnectionError` (or the Go equivalent: `errors.As` into `*anthropic.Error` then `switch apierr.StatusCode { case 404: ...; case 429: ...; default: ... }`). Per-language class names and namespaces are in `shared/error-codes.md`.
  **错误处理——捕获一条链，而不是一个宽泛的类。**单一的 `except APIStatusError` / `catch (AnthropicServiceException)` / `rescue APIError` 会丢失可重试（429、>=500、网络）与不可重试（400/404）失败之间的区分。写一条最具体优先的链——例如 `NotFoundError` -> `RateLimitError` -> `APIStatusError` -> `APIConnectionError`（Go 的等价写法：用 `errors.As` 转成 `*anthropic.Error`，然后 `switch apierr.StatusCode { case 404: ...; case 429: ...; default: ... }`）。各语言的类名与命名空间见 `shared/error-codes.md`。
- **Don't research SDK types - write first.** If a type name isn't shown in the documentation included in this skill, write the code file from the namespace/package tables in the language-specific doc and let the compiler's error point you to the right name. Do not spend turns on WebFetch, SDK-repo clones, or compiling-and-running a separate reflection program to discover type names before writing - produce the source file first, then fix what the compiler reports. A quick `strings` / `jar tf` / `javap` against the installed SDK is acceptable for locating names (it returns in seconds), but don't escalate beyond that. A file with a wrong type name is recoverable; a session spent on discovery with no file written is not.
  **不要去调研 SDK 类型——先写代码。**如果本技能文档中没有展示某个类型名，就依据语言专属文档中的命名空间/包表写出代码文件，让编译器报错为你指到正确的名字。不要在写之前把轮次花在 WebFetch、clone SDK 仓库或编译运行一个反射程序上来发现类型名——先产出源文件，再修编译器报告的问题。对已安装的 SDK 快速跑一下 `strings` / `jar tf` / `javap` 来定位名字是可以的（几秒即返回），但不要升级到更多手段。类型名写错的文件是可以挽回的；一个没写出任何文件、全花在探索上的会话则不然。
- **Bash and text editor tools are Anthropic-defined, schema-less.** Declare `{"type": "bash_20250124", "name": "bash"}` / `{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}` - no `input_schema`. A custom tool with your own schema named `"bash"` is a different tool. Handler paths and security checks are in `shared/tool-use-concepts.md` § Client-Side Tools.
  **bash 与文本编辑器工具是 Anthropic 定义的无 schema 工具。**声明 `{"type": "bash_20250124", "name": "bash"}` / `{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}`——不带 `input_schema`。你自己定义的、名为 `"bash"` 且带自有 schema 的自定义工具是另一个工具。处理路径与安全检查见 `shared/tool-use-concepts.md` § Client-Side Tools。
- **Advisor tool model pairing.** The advisor tool's `model` must be at least as capable as the request's top-level `model` - e.g. executor `claude-sonnet-5-5` -> advisor `claude-opus-5-5`. An invalid pair returns 400; a `claude-sonnet-5-5` executor accepts only the advisors its row in the pairing table lists (not Claude Opus 4.8 / 4.7 / 4.6, Claude Sonnet 5, or Sonnet 4.6). Pairing table (and which advisors return plaintext vs encrypted `advisor_redacted_result` advice) in `shared/tool-use-concepts.md` § Advisor. Availability: `shared/platform-availability.md`.
  **Advisor 工具的模型配对。**advisor 工具的 `model` 能力必须不低于请求的顶层 `model`——例如执行器 `claude-sonnet-5-5` -> advisor `claude-opus-5-5`。无效配对返回 400；`claude-sonnet-5-5` 执行器只接受配对表中其所在行列出的 advisor（不接受 Claude Opus 4.8 / 4.7 / 4.6、Claude Sonnet 5 或 Sonnet 4.6）。配对表（以及哪些 advisor 返回明文、哪些返回加密的 `advisor_redacted_result` 建议）见 `shared/tool-use-concepts.md` § Advisor。可用性：`shared/platform-availability.md`。
- **Agent Skills != Managed Agents.** To have Claude generate a `.pptx`/`.xlsx`/etc. via Agent Skills, call `client.beta.messages.create` with `container={"skills": [...]}`, the `code_execution_20260521` tool, and the `code-execution-2025-08-25` beta (Skills is out of beta - no `skills-2025-10-02` header needed). Do not use `client.beta.agents` / `sessions` / `environments` here - those are the Managed Agents surface, not Agent Skills.
  **Agent Skills != Managed Agents。**要让 Claude 通过 Agent Skills 生成 `.pptx`/`.xlsx` 等文件，调用 `client.beta.messages.create` 时带 `container={"skills": [...]}`、`code_execution_20260521` 工具与 `code-execution-2025-08-25` beta（Skills 已结束 beta——无需 `skills-2025-10-02` 头）。此处不要用 `client.beta.agents` / `sessions` / `environments`——那些是 Managed Agents 的表面，不是 Agent Skills。
- **MCP connector needs both halves.** `mcp_servers=[{type:"url", url, name}]` alone is rejected as a validation error - also add `tools=[{type:"mcp_toolset", mcp_server_name:<same name>}]` with beta `mcp-client-2025-11-20`. Availability: `shared/platform-availability.md`.
  **MCP 连接器需要两部分。**只有 `mcp_servers=[{type:"url", url, name}]` 会被当作校验错误拒绝——还要加上 `tools=[{type:"mcp_toolset", mcp_server_name:<同名>}]` 并带 beta `mcp-client-2025-11-20`。可用性：`shared/platform-availability.md`。
- **`inference_geo` is a direct top-level request parameter** - `client.messages.create(..., inference_geo="us")` / `.inferenceGeo("us")`. Do not put it in `extra_body` / `putAdditionalBodyProperty`. (Messages API only - on Managed Agents, `inference_geo` instead nests inside the agent's `model` object, never top-level; see `shared/managed-agents-core.md` § Pinning inference geography.) Supported on Opus 4.6 / Sonnet 4.6 and later; availability: `shared/platform-availability.md`. `response.usage.inference_geo` reports where inference ran.
  **`inference_geo` 是直接的顶层请求参数**——`client.messages.create(..., inference_geo="us")` / `.inferenceGeo("us")`。不要放进 `extra_body` / `putAdditionalBodyProperty`。（仅 Messages API——在 Managed Agents 上，`inference_geo` 改为嵌套在 agent 的 `model` 对象内，绝不在顶层；见 `shared/managed-agents-core.md` § Pinning inference geography。）Opus 4.6 / Sonnet 4.6 及之后版本支持；可用性：`shared/platform-availability.md`。`response.usage.inference_geo` 报告推理实际运行的位置。
- **Fine-grained tool streaming is not a beta feature; this skill's default is to turn it on for streaming + client tools (the API itself still defaults to buffered).** Set `eager_input_streaming: true` on the tool definition and call the regular `client.messages.stream(...)`. There is no beta header and no `client.beta.*` path. Do not also send the legacy `fine-grained-tool-streaming-2025-05-14` beta header. Python's `@beta_tool(eager_input_streaming=True)` accepts it directly; TypeScript's `betaZodTool()` does not, so spread it on: `{ ...betaZodTool({...}), eager_input_streaming: true }`. With the field on, the API no longer coerces or validates the input, so the accumulated `partial_json` may be incomplete (`max_tokens`) or invalid - guard the parse (`shared/tool-use-concepts.md` -> Eager input streaming).
  **细粒度工具流式不是 beta 特性；本技能的默认做法是在"流式 + 客户端工具"时开启它（API 本身仍默认缓冲）。**在工具定义上设置 `eager_input_streaming: true` 并调用常规的 `client.messages.stream(...)`。没有 beta 头，也没有 `client.beta.*` 路径。不要同时发送旧的 `fine-grained-tool-streaming-2025-05-14` beta 头。Python 的 `@beta_tool(eager_input_streaming=True)` 直接接受它；TypeScript 的 `betaZodTool()` 不接受，需要展开传入：`{ ...betaZodTool({...}), eager_input_streaming: true }`。开启该字段后，API 不再对输入做强制转换或校验，因此累积的 `partial_json` 可能不完整（`max_tokens`）或无效——解析时要做防护（`shared/tool-use-concepts.md` -> Eager input streaming）。
- **Cache diagnostics is beta.** Use `client.beta.messages.*` with beta `cache-diagnosis-2026-04-07`. Pass `diagnostics: {previous_message_id: null}` on the first turn and `diagnostics: {previous_message_id: <previous response id>}` on subsequent turns; the result is on `response.diagnostics`. Availability: `shared/platform-availability.md`.
  **缓存诊断是 beta。**使用 `client.beta.messages.*` 并带 beta `cache-diagnosis-2026-04-07`。第一轮传 `diagnostics: {previous_message_id: null}`，后续轮次传 `diagnostics: {previous_message_id: <先前响应 id>}`；结果在 `response.diagnostics` 上。可用性：`shared/platform-availability.md`。
- **Memory tool type is `memory_20250818`.** Declare `{"type": "memory_20250818", "name": "memory"}`. Go uses the beta-namespace type `{OfMemoryTool20250818: &anthropic.BetaMemoryTool20250818Param{}}` on `client.Beta.Messages.New`; Python/TypeScript/Ruby/PHP/C# use the non-beta `client.messages.create`; Java has both a non-beta `MemoryTool20250818` and a beta tool-runner path. Python/TypeScript provide `BetaAbstractMemoryTool` / `betaMemoryTool` helpers for implementing the backend.
  **记忆工具类型是 `memory_20250818`。**声明 `{"type": "memory_20250818", "name": "memory"}`。Go 在 `client.Beta.Messages.New` 上使用 beta 命名空间的类型 `{OfMemoryTool20250818: &anthropic.BetaMemoryTool20250818Param{}}`；Python/TypeScript/Ruby/PHP/C# 使用非 beta 的 `client.messages.create`；Java 既有非 beta 的 `MemoryTool20250818`，也有 beta tool-runner 路径。Python/TypeScript 提供 `BetaAbstractMemoryTool` / `betaMemoryTool` 辅助器用于实现后端。
- **Use a model the feature actually supports.** Some features are restricted to specific model tiers - fast mode is Claude Opus 5 / Claude Opus 5.5 / Opus 4.8 only (and Claude API only), task budgets (Messages API only - Managed Agents session budgets have no model-tier restriction) are Claude Opus 5 / Claude Opus 5.5 / Fable 5 / Claude Fable 5.1 (confirm at launch) / Claude Sonnet 5.5 / Opus 4.8 / 4.7 only (not Claude Sonnet 5), and the advisor tool requires a valid executor<->advisor pair. If the user's prompt names a model that the feature doesn't support, use a supported model instead and note the substitution in the output.
  **使用功能真正支持的模型。**有些功能限定于特定模型档位——快速模式仅限 Claude Opus 5 / Claude Opus 5.5 / Opus 4.8（且仅限 Claude API），任务预算（仅 Messages API——Managed Agents 会话预算没有模型档位限制）仅限 Claude Opus 5 / Claude Opus 5.5 / Fable 5 / Claude Fable 5.1（发布时确认）/ Claude Sonnet 5.5 / Opus 4.8 / 4.7（不含 Claude Sonnet 5），而 advisor 工具要求有效的 执行器<->advisor 配对。如果用户的提示词点名的模型不支持该功能，请改用受支持的模型，并在输出中注明该替换。
- **Don't define custom types for SDK data structures:** The SDK exports types for all API objects. Use `Anthropic.MessageParam` for messages, `Anthropic.Tool` for tool definitions, `Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam` for tool results, `Anthropic.Message` for responses. Defining your own `interface ChatMessage { role: string; content: unknown }` duplicates what the SDK already provides and loses type safety.
  **不要为 SDK 数据结构自定义类型：**SDK 为所有 API 对象导出了类型。消息用 `Anthropic.MessageParam`，工具定义用 `Anthropic.Tool`，工具结果用 `Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam`，响应用 `Anthropic.Message`。自定义 `interface ChatMessage { role: string; content: unknown }` 会重复 SDK 已提供的内容并丧失类型安全。
- **Report and document output:** For tasks that produce reports, documents, or visualizations, the code execution sandbox has `python-docx`, `python-pptx`, `matplotlib`, `pillow`, and `pypdf` pre-installed. Claude can generate formatted files (DOCX, PDF, charts) and return them via the Files API - consider this for "report" or "document" type requests instead of plain stdout text.
  **报告与文档输出：**对产出报告、文档或可视化的任务，代码执行沙箱预装了 `python-docx`、`python-pptx`、`matplotlib`、`pillow` 与 `pypdf`。Claude 可以生成带格式的文件（DOCX、PDF、图表）并通过 Files API 返回——对"报告"或"文档"类请求请考虑用它代替纯 stdout 文本。
- **Server-tool errors don't raise.** Web search and web fetch errors return HTTP 200 with a `web_search_tool_result` / `web_fetch_tool_result` block whose `content` is a single error object (e.g. `{error_code: "max_uses_exceeded"}`) - not a raised exception. For web search, a success `content` is a *list*; an error `content` is an *object* - branch on that before indexing.
  **服务器端工具的错误不会抛异常。**网络搜索与网络抓取的错误以 HTTP 200 返回，带一个 `web_search_tool_result` / `web_fetch_tool_result` 块，其 `content` 是单个错误对象（例如 `{error_code: "max_uses_exceeded"}`）——而不是抛出的异常。对网络搜索而言，成功的 `content` 是*列表*，错误的 `content` 是*对象*——索引之前先按此分支。
- **Managed Agents web tools ignore the environment's `networking`.** `web_search` / `web_fetch` run on Anthropic's servers in cloud *and* self-hosted environments, and Console org-level web settings apply to the Messages API only. Restrict them per tool with `allowed_domains` **or** `blocked_domains` (never both; 1-64 plain hostnames per list, subdomains covered; IPs, bare TLDs, single-label and `localhost`-style names rejected on both tools; a path suffix is allowed only on `web_search`) on the toolset `configs` entry - `shared/managed-agents-tools.md` § Web search & web fetch settings.
  **Managed Agents 的网络工具忽略环境的 `networking` 设置。**`web_search` / `web_fetch` 在云端*和*自托管环境中都运行于 Anthropic 的服务器上，且 Console 的组织级网络设置只作用于 Messages API。要在工具层面限制它们，在工具集的 `configs` 条目上设置 `allowed_domains` **或** `blocked_domains`（绝不同时设置；每个列表 1-64 个纯主机名，覆盖子域；两个工具都拒绝 IP、裸 TLD、单标签与 `localhost` 风格的名称；路径后缀仅 `web_search` 允许）——见 `shared/managed-agents-tools.md` § Web search & web fetch settings。
- **Eval / hillclimb work has dedicated guides:** If the user says "hillclimb", "improve my eval score", "iterate on my prompt against an eval", or "build me an eval" - load `shared/evals/eval-hillclimb.md` or `shared/evals/build-eval.md` rather than improvising. The bundled HTML report builder is `shared/evals/report/build-report.mjs` when it is on disk (EAP install), else `shared/evals/report/build-report-lite.mjs` (always extracted with this skill); don't write a parallel one.
  **Eval / hillclimb 工作有专属指南：**如果用户说 "hillclimb"、"improve my eval score"、"iterate on my prompt against an eval" 或 "build me an eval"——加载 `shared/evals/eval-hillclimb.md` 或 `shared/evals/build-eval.md`，不要即兴发挥。附带的 HTML 报告构建器在磁盘上存在时（EAP 安装）为 `shared/evals/report/build-report.mjs`，否则为 `shared/evals/report/build-report-lite.mjs`（随本技能必然解包）；不要另写平行实现。
- **Code execution output block type:** `code_execution_20260521` returns `bash_code_execution_tool_result` (with `.content.stdout`), **not** the legacy bare `code_execution_tool_result`. Iterate `response.content` and match on the correct type.
  **代码执行输出块类型：**`code_execution_20260521` 返回 `bash_code_execution_tool_result`（带 `.content.stdout`），**而不是**旧的裸 `code_execution_tool_result`。遍历 `response.content` 并按正确的类型匹配。
- **Tool search: never defer everything.** The search tool itself must not have `defer_loading: true`, and at least one tool in `tools` must be non-deferred, or the API returns 400 `All tools have defer_loading set`.
  **工具搜索：绝不要全部延迟。**搜索工具自身不得设置 `defer_loading: true`，且 `tools` 中至少有一个工具必须是未延迟的，否则 API 返回 400 `All tools have defer_loading set`。
