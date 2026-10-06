<!-- BILINGUAL-EN-ZH -->
# Claude Model Catalog / Claude 模型目录

**Only use exact model IDs listed in this file.** Never guess or construct model IDs - incorrect IDs will cause API errors. Use aliases wherever available. For the latest information, WebFetch the Models Overview URL in `shared/live-sources.md`, or query the Models API directly (see Programmatic Model Discovery below).

**只使用本文件中列出的精确模型 ID。** 绝不猜测或拼造模型 ID——错误的 ID 会导致 API 错误。凡是可用别名之处都使用别名。要获取最新信息，用 WebFetch 抓取 `shared/live-sources.md` 中的 Models Overview URL，或直接查询 Models API（见下文的 Programmatic Model Discovery 程序化模型发现）。

【评论】首段以强制口吻限定模型只能使用清单内的 ID，是针对模型幻觉式拼造模型名的防护设计。

## Programmatic Model Discovery / 程序化模型发现

For **live** capability data - context window, max output tokens, feature support (thinking, vision, effort, structured outputs, etc.) - query the Models API instead of relying on the cached tables below. Use this when the user asks "what's the context window for X", "does model X support vision/thinking/effort", "which models support feature Y", or wants to select a model by capability at runtime.

要获取**实时**能力数据——上下文窗口、最大输出 token、功能支持（思考、视觉、effort、结构化输出等）——请查询 Models API，而不要依赖下方缓存表格。当用户询问"X 的上下文窗口是多少"、"模型 X 是否支持视觉/思考/effort"、"哪些模型支持功能 Y"，或想在运行时按能力选择模型时，使用此方法。

```python
m = client.models.retrieve("claude-opus-4-8")
m.id                 # "claude-opus-4-8"
m.display_name       # "Claude Opus 4.8"
m.max_input_tokens   # context window (int)
m.max_tokens         # max output tokens (int)

# capabilities is an untyped nested dict - bracket access, check ["supported"] at the leaf
caps = m.capabilities
caps["image_input"]["supported"]                       # vision
caps["thinking"]["types"]["adaptive"]["supported"]     # adaptive thinking
caps["effort"]["max"]["supported"]                     # effort: max (also low/medium/high)
caps["structured_outputs"]["supported"]
caps["context_management"]["compact_20260112"]["supported"]

# filter across all models - iterate the page object directly (auto-paginates); do NOT use .data
[m for m in client.models.list()
 if m.capabilities["thinking"]["types"]["adaptive"]["supported"]
 and m.max_input_tokens >= 200_000]
```

Top-level fields (`id`, `display_name`, `max_input_tokens`, `max_tokens`) are typed attributes. `capabilities` is a dict - use bracket access, not attribute access. The API returns the full capability tree for every model with `supported: true/false` at each leaf, so bracket chains are safe without `.get()` guards. TypeScript SDK: same method names, also auto-paginates on iteration.

顶层字段（`id`、`display_name`、`max_input_tokens`、`max_tokens`）是带类型的属性。`capabilities` 是一个 dict——使用方括号访问，而不是属性访问。API 为每个模型返回完整的能力树，每个叶子节点都有 `supported: true/false`，因此方括号链不需要 `.get()` 防护也是安全的。TypeScript SDK：方法名相同，迭代时同样自动分页。

### Raw HTTP / 原生 HTTP

```bash
curl https://api.anthropic.com/v1/models/claude-opus-4-8 \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01"
```

```js
{
  "id": "claude-opus-4-8",
  "display_name": "Claude Opus 4.8",
  "max_input_tokens": 1000000,
  "max_tokens": 128000,
  "capabilities": {
    "image_input": {"supported": true},
    "structured_outputs": {"supported": true},
    "thinking": {"supported": true, "types": {"enabled": {"supported": false}, "adaptive": {"supported": true}}},
    "effort": {"supported": true, "low": {"supported": true}, ..., "max": {"supported": true}},
    ...
  }
}
```

## Current Models (recommended) / 当前模型（推荐）

| Friendly Name     | Alias (use this)    | Full ID                       | Context        | Max Output | Status |
|-------------------|---------------------|-------------------------------|----------------|------------|--------|
| Claude Fable 5.1    | `claude-fable-5-1`      | -                             | 1M             | 128K       | Active |
| Claude Mythos 5.1   | `claude-mythos-5-1`     | -                             | 1M             | 128K       | Active (Project Glasswing only) |
| Claude Fable 5 | `claude-fable-5` | -                             | 1M             | 128K       | Active |
| Claude Mythos 5 | `claude-mythos-5` | -                          | 1M             | 128K       | Active (Project Glasswing only) |
| Claude Opus 5.5 | `claude-opus-5-5` | -                             | 1M             | 128K       | Active |
| Claude Opus 5     | `claude-opus-5`       | -                             | 1M             | 128K       | Active |
| Claude Opus 4.8   | `claude-opus-4-8`   | -                             | 1M             | 128K       | Active |
| Claude Opus 4.7   | `claude-opus-4-7`   | -                             | 1M             | 128K       | Active |
| Claude Opus 4.6   | `claude-opus-4-6`   | -                             | 1M             | 128K       | Active |
| Claude Sonnet 5.5 | `claude-sonnet-5-5` | -                     | 1M             | 128K       | Active |
| Claude Sonnet 5 | `claude-sonnet-5` | -                         | 1M             | 128K       | Active |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | -                             | 1M             | 128K       | Active |
| Claude Haiku 4.5  | `claude-haiku-4-5`  | `claude-haiku-4-5-20251001`   | 200K           | 64K        | Active |

| 友好名称 | 别名（使用此别名） | 完整 ID | 上下文 | 最大输出 | 状态 |
|-------------------|---------------------|-------------------------------|----------------|------------|--------|
| Claude Fable 5.1 | `claude-fable-5-1` | - | 1M | 128K | 活跃 |
| Claude Mythos 5.1 | `claude-mythos-5-1` | - | 1M | 128K | 活跃（仅限 Project Glasswing） |
| Claude Fable 5 | `claude-fable-5` | - | 1M | 128K | 活跃 |
| Claude Mythos 5 | `claude-mythos-5` | - | 1M | 128K | 活跃（仅限 Project Glasswing） |
| Claude Opus 5.5 | `claude-opus-5-5` | - | 1M | 128K | 活跃 |
| Claude Opus 5 | `claude-opus-5` | - | 1M | 128K | 活跃 |
| Claude Opus 4.8 | `claude-opus-4-8` | - | 1M | 128K | 活跃 |
| Claude Opus 4.7 | `claude-opus-4-7` | - | 1M | 128K | 活跃 |
| Claude Opus 4.6 | `claude-opus-4-6` | - | 1M | 128K | 活跃 |
| Claude Sonnet 5.5 | `claude-sonnet-5-5` | - | 1M | 128K | 活跃 |
| Claude Sonnet 5 | `claude-sonnet-5` | - | 1M | 128K | 活跃 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | - | 1M | 128K | 活跃 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | `claude-haiku-4-5-20251001` | 200K | 64K | 活跃 |

### Model Descriptions / 模型描述
- **Claude Fable 5.1** - Anthropic's most capable widely released model, for the most demanding reasoning and long-horizon agentic work. Successor to Claude Fable 5 in the same tier at the same per-token price ($10/$50 per MTok; cache reads $0.25/MTok - 0.025x, a quarter of Claude Fable 5's; batch $5/$25); stronger long-running agentic coding, knowledge work with documents/spreadsheets/slides, multistep research, vision, long-context retrieval, and computer use. Same API surface as Claude Fable 5 (thinking always on, no prefill, no sampling params, `refusal` stop reason, 512-token cache minimum) with three breaking changes: forced tool use (`tool_choice` `any` / `tool`) returns a 400; thinking blocks are bound to the producing model (only Claude Mythos 5.1 can read them - other models drop them); and editing earlier turns invalidates thinking blocks ("preserved thinking"; new accounts created on/after 2026-08-31 get a 400 on edited history on every platform, and enforcement scope is decided per model; the opt-in controls beta is on the Claude API, Claude Platform on AWS, Bedrock, and Vertex - Foundry unconfirmed, `shared/platform-availability.md`). Adds per-message `effort`, turn-scoped `clear_at` system messages, `thinking.display: "updates"` progress updates, and content provenance. Same tokenizer as Claude Fable 5; 1M context (default), 128K max output. Covered Model: 30-day retention required (ZDR only if expressly authorized by Anthropic) - ZDR orgs get `400 invalid_request_error`, as on Claude Fable 5. No Priority Tier; shares the Fable 5.x rate-limit pool. See `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5.
  **Claude Fable 5.1**——Anthropic 能力最强、已广泛发布的模型，面向要求最高的推理与长程智能体工作。同一层级中 Claude Fable 5 的后继者，每 token 价格相同（每百万 token $10/$50；缓存读取 $0.25/MTok——0.025 倍，为 Claude Fable 5 的四分之一；批量 $5/$25）；在长时运行的智能体编码、涉及文档/电子表格/幻灯片的知识工作、多步研究、视觉、长上下文检索和计算机使用方面更强。API 接口与 Claude Fable 5 相同（思考始终开启、无预填充、无采样参数、`refusal` 停止原因、512 token 缓存下限），另有三处破坏性变更：强制工具使用（`tool_choice` 为 `any` / `tool`）返回 400；思考块绑定到产生它的模型（只有 Claude Mythos 5.1 能读取——其他模型会丢弃它们）；编辑较早回合会使思考块失效（"preserved thinking"保留思考；2026-08-31 及之后创建的新账户在所有平台上对被编辑过的历史都会收到 400，执行范围按模型决定；opt-in controls beta 在 Claude API、AWS 上的 Claude Platform、Bedrock 和 Vertex 上提供——Foundry 未确认，见 `shared/platform-availability.md`）。新增每消息 `effort`、回合范围的 `clear_at` 系统消息、`thinking.display: "updates"` 进度更新以及内容溯源。与 Claude Fable 5 使用相同的分词器；1M 上下文（默认），128K 最大输出。Covered Model：要求 30 天保留期（仅当 Anthropic 明确授权时才可零数据保留 ZDR）——ZDR 组织会收到 `400 invalid_request_error`，与 Claude Fable 5 上相同。无 Priority Tier；与 Fable 5.x 共享速率限制池。参见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5。
- **Claude Fable 5** / **Claude Mythos 5** (`claude-fable-5` / `claude-mythos-5`) - the previous Fable / Mythos release: same tier, limits and per-token pricing as Claude Fable 5.1, which adds three breaking API changes over them (see above; cache reads here are $1/MTok rather than Claude Fable 5.1's $0.25); still served and selectable by id. Claude Mythos 5 ran no safety classifiers, so `stop_reason: "refusal"` does not occur on it. Prefer claude-fable-5-1 for new work.
  **Claude Fable 5** / **Claude Mythos 5**（`claude-fable-5` / `claude-mythos-5`）——上一代 Fable / Mythos 发布版本：层级、限制和每 token 价格与 Claude Fable 5.1 相同，后者相对它们增加了三处破坏性 API 变更（见上文；此处的缓存读取为 $1/MTok，而非 Claude Fable 5.1 的 $0.25）；仍提供服务，可按 id 选择。Claude Mythos 5 不运行任何安全分类器，因此不会出现 `stop_reason: "refusal"`。新工作优先使用 claude-fable-5-1。
- **Claude Mythos 5.1** - The same model as Claude Fable 5.1 (same capabilities, limits, per-token pricing, API behavior - except it does not run the history-editing check), offered only to approved Project Glasswing customers; successor to Claude Mythos 5 (which itself succeeded the invitation-only `claude-mythos-preview`). Unlike Claude Mythos 5 it runs safeguards that depend on the access program, so handle `stop_reason: "refusal"`. Not offered on Claude Platform on AWS. Use it only when the org participates in Project Glasswing; otherwise use `claude-fable-5-1`.
  **Claude Mythos 5.1**——与 Claude Fable 5.1 相同的模型（能力、限制、每 token 价格、API 行为都相同——除了它不运行历史编辑检查），仅提供给获得批准的 Project Glasswing 客户；Claude Mythos 5 的后继者（而 Claude Mythos 5 又接替了仅限邀请的 `claude-mythos-preview`）。与 Claude Mythos 5 不同，它运行依赖访问计划的安全防护，因此需要处理 `stop_reason: "refusal"`。不在 AWS 上的 Claude Platform 提供。仅当组织参与 Project Glasswing 时使用；否则使用 `claude-fable-5-1`。
- **Claude Opus 5.5** - Successor to Claude Opus 5 in the Opus line for long-running agentic coding and knowledge work, at a lower price ($4 / $20 per MTok; cache reads $0.20). Same 1M context, 128K output, tokenizer, and feature set as Claude Opus 5, with four breaking changes: thinking can't be disabled (effort is the only control, default `medium`), forced `tool_choice` 400s, thinking blocks are tied to the model and the conversation, and computer use needs the `computer_toolset_20260801` toolset. Broader safety classifiers (`bio` and `reasoning_extraction` join `cyber`). The current Opus and the default model; see `shared/model-migration.md` -> Migrating to Claude Opus 5.5.
  **Claude Opus 5.5**——Opus 系列中 Claude Opus 5 的后继者，面向长时运行的智能体编码与知识工作，价格更低（每百万 token $4 / $20；缓存读取 $0.20）。与 Claude Opus 5 相同的 1M 上下文、128K 输出、分词器和功能集，但有四处破坏性变更：思考无法禁用（effort 是唯一控制手段，默认 `medium`）、强制 `tool_choice` 返回 400、思考块绑定到模型和对话、计算机使用需要 `computer_toolset_20260801` 工具集。安全分类器覆盖更广（`bio` 和 `reasoning_extraction` 加入 `cyber`）。当前的 Opus 与默认模型；参见 `shared/model-migration.md` -> Migrating to Claude Opus 5.5。
- **Claude Opus 5** - For complex agentic coding and enterprise work; a step-change over Claude Opus 4.8, strongest on deep reasoning, agentic and long-horizon work, and test-time compute scaling, at half the cost of Claude Fable 5.1 (Claude Fable 5.1 remains the highest-capability tier). Safety classifiers can return `stop_reason: "refusal"` - handle it before reading `content`. A drop-in upgrade at Opus 4.8's pricing ($5/$25 per MTok) with the same feature set. Thinking is on by default (omitting `thinking` runs adaptive; `{type: "adaptive"}` is equivalent), and `thinking: {type: "disabled"}` is available only at effort `high` or lower - pairing it with `xhigh`/`max` returns a 400. Raw thinking tokens are never returned. Full effort ladder through `max`; 512-token prompt-cache minimum (down from 1024 on Opus 4.8); fast mode on the Claude API only. Elevated cybersecurity safeguards. Separate rate-limit bucket from the combined Opus 4.x pool. 1M context window (default and maximum), 128K max output. See `shared/model-migration.md` -> Migrating to Claude Opus 5.
  **Claude Opus 5**——面向复杂智能体编码和企业工作；相比 Claude Opus 4.8 是一次跨越式提升，在深度推理、智能体与长程工作以及测试时计算扩展方面最强，成本为 Claude Fable 5.1 的一半（Claude Fable 5.1 仍是能力最高的层级）。安全分类器可能返回 `stop_reason: "refusal"`——在读取 `content` 之前先处理它。以 Opus 4.8 的价格（每百万 token $5/$25）即可无缝升级，功能集相同。思考默认开启（省略 `thinking` 时运行为自适应；`{type: "adaptive"}` 等效），且 `thinking: {type: "disabled"}` 仅在 effort 为 `high` 或更低时可用——与 `xhigh`/`max` 搭配会返回 400。原始思考 token 永不返回。完整 effort 阶梯直至 `max`；提示词缓存下限 512 token（从 Opus 4.8 的 1024 下调）；快速模式仅在 Claude API 上提供。网络安全防护升级。速率限制桶独立于合并的 Opus 4.x 池。1M 上下文窗口（默认和最大），128K 最大输出。参见 `shared/model-migration.md` -> Migrating to Claude Opus 5。
- **Claude Opus 4.8** - The most capable model in the Opus 4 series - highly autonomous, state-of-the-art on long-horizon agentic work, knowledge work, and memory; clearer, warmer writing. Same API surface as Opus 4.7 (adaptive thinking only; sampling parameters and `budget_tokens` removed). 1M context window at standard API pricing (no long-context premium). See `shared/model-migration.md` -> Migrating to Opus 4.8 - a 4.7 -> 4.8 move is a model-ID swap plus prompt re-tuning, no new breaking changes.
  **Claude Opus 4.8**——Opus 4 系列中能力最强的模型——高度自主，在长程智能体工作、知识工作和记忆方面达到最先进水平；文笔更清晰、更温暖。API 接口与 Opus 4.7 相同（仅支持自适应思考；采样参数和 `budget_tokens` 已移除）。以标准 API 价格提供 1M 上下文窗口（无长上下文溢价）。参见 `shared/model-migration.md` -> Migrating to Opus 4.8——从 4.7 迁移到 4.8 只需替换模型 ID 并重新调整提示词，没有新的破坏性变更。
- **Claude Opus 4.7** - Previous-generation Opus. Highly autonomous; strong on long-horizon agentic work, knowledge work, vision, and memory. Adaptive thinking only; sampling parameters and `budget_tokens` removed. 1M context window. See `shared/model-migration.md` -> Migrating to Opus 4.7.
  **Claude Opus 4.7**——上一代 Opus。高度自主；在长程智能体工作、知识工作、视觉和记忆方面表现出色。仅支持自适应思考；采样参数和 `budget_tokens` 已移除。1M 上下文窗口。参见 `shared/model-migration.md` -> Migrating to Opus 4.7。
- **Claude Opus 4.6** - Older Opus. Supports adaptive thinking (recommended), 128K max output tokens (requires streaming for large outputs). 1M context window.
  **Claude Opus 4.6**——更早的 Opus。支持自适应思考（推荐），最大输出 128K token（大输出需要流式传输）。1M 上下文窗口。
- **Claude Sonnet 5** - The previous Sonnet; near-Opus quality on coding and agentic work. Adaptive thinking on by default (omitting `thinking` runs adaptive); manual `budget_tokens` removed; non-default sampling parameters rejected. `effort` supports `low`/`medium`/`high`/`xhigh`/`max`. New tokenizer (~30% more tokens for the same text vs Sonnet 4.6). High-resolution vision (2576px). 1M context window, 128K max output. See `shared/model-migration.md` -> Migrating to Claude Sonnet 5.
  **Claude Sonnet 5**——上一代 Sonnet；编码与智能体工作接近 Opus 水准。自适应思考默认开启（省略 `thinking` 时运行为自适应）；手动 `budget_tokens` 已移除；非默认采样参数会被拒绝。`effort` 支持 `low`/`medium`/`high`/`xhigh`/`max`。新分词器（与 Sonnet 4.6 相比，同样文本约多 30% token）。高分辨率视觉（2576px）。1M 上下文窗口，128K 最大输出。参见 `shared/model-migration.md` -> Migrating to Claude Sonnet 5。
- **Claude Sonnet 5.5** - Successor to Claude Sonnet 5 in the Sonnet line, at the same prices ($2 / $10 per MTok; cache reads $0.20). Same tokenizer as Claude Sonnet 5; 1M context, 128K max output. Adaptive thinking on by default; effort default `high`, with recalibrated levels. Five breaking changes: `thinking: {type: "disabled"}` returns a 400 (send `{type: "between_tools"}` at effort `high` or below to turn thinking off), forced `tool_choice` 400s, thinking blocks are tied to the model and the conversation, computer use on the Claude API and Google Cloud needs the `computer_toolset_20260801` toolset, and the advisor tool rejects Claude Opus 4.8, Claude Opus 4.7, and Claude Sonnet 5 advisors. See `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5.
  **Claude Sonnet 5.5**——Sonnet 系列中 Claude Sonnet 5 的后继者，价格相同（每百万 token $2 / $10；缓存读取 $0.20）。与 Claude Sonnet 5 相同的分词器；1M 上下文，128K 最大输出。自适应思考默认开启；effort 默认 `high`，各档位经过重新校准。五处破坏性变更：`thinking: {type: "disabled"}` 返回 400（在 effort 为 `high` 或更低时发送 `{type: "between_tools"}` 来关闭思考）、强制 `tool_choice` 返回 400、思考块绑定到模型和对话、Claude API 和 Google Cloud 上的计算机使用需要 `computer_toolset_20260801` 工具集，以及 advisor 工具拒绝 Claude Opus 4.8、Claude Opus 4.7 和 Claude Sonnet 5 的 advisor。参见 `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5。
- **Claude Sonnet 4.6** - Previous-generation Sonnet. Supports adaptive thinking (recommended). 1M context window. 128K max output tokens.
  **Claude Sonnet 4.6**——上一代 Sonnet。支持自适应思考（推荐）。1M 上下文窗口。最大输出 128K token。
- **Claude Haiku 4.5** - Fastest and most cost-effective model for simple tasks.
  **Claude Haiku 4.5**——面向简单任务最快、性价比最高的模型。

## Legacy Models (still active) / 旧模型（仍然活跃）

| Friendly Name     | Alias (use this)    | Full ID                       | Status |
|-------------------|---------------------|-------------------------------|--------|
| Claude Opus 4.5   | `claude-opus-4-5`   | `claude-opus-4-5-20251101`    | Active |
| Claude Opus 4.1   | `claude-opus-4-1`   | `claude-opus-4-1-20250805`    | Deprecated (retires 2026-08-05 - migrate to `claude-opus-5-5`) |
| Claude Sonnet 4.5 | `claude-sonnet-4-5` | `claude-sonnet-4-5-20250929`  | Active |

| 友好名称 | 别名（使用此别名） | 完整 ID | 状态 |
|-------------------|---------------------|-------------------------------|--------|
| Claude Opus 4.5 | `claude-opus-4-5` | `claude-opus-4-5-20251101` | 活跃 |
| Claude Opus 4.1 | `claude-opus-4-1` | `claude-opus-4-1-20250805` | 已弃用（2026-08-05 退役——迁移到 `claude-opus-5-5`） |
| Claude Sonnet 4.5 | `claude-sonnet-4-5` | `claude-sonnet-4-5-20250929` | 活跃 |

## Deprecated Models (retiring soon) / 已弃用模型（即将退役）

| Friendly Name     | Alias (use this)    | Full ID                       | Status     | Retires      |
|-------------------|---------------------|-------------------------------|------------|--------------|
| Claude Sonnet 4   | `claude-sonnet-4-0` | `claude-sonnet-4-20250514`    | Deprecated | TBD          |
| Claude Opus 4     | `claude-opus-4-0`   | `claude-opus-4-20250514`      | Deprecated | TBD          |
| Claude Haiku 3    | -                   | `claude-3-haiku-20240307`     | Deprecated | Apr 19, 2026 |

| 友好名称 | 别名（使用此别名） | 完整 ID | 状态 | 退役时间 |
|-------------------|---------------------|-------------------------------|------------|--------------|
| Claude Sonnet 4 | `claude-sonnet-4-0` | `claude-sonnet-4-20250514` | 已弃用 | 待定 |
| Claude Opus 4 | `claude-opus-4-0` | `claude-opus-4-20250514` | 已弃用 | 待定 |
| Claude Haiku 3 | - | `claude-3-haiku-20240307` | 已弃用 | 2026 年 4 月 19 日 |

## Retired Models (no longer available) / 已退役模型（不再可用）

| Friendly Name     | Full ID                       | Retired     |
|-------------------|-------------------------------|-------------|
| Claude Sonnet 3.7 | `claude-3-7-sonnet-20250219`  | Feb 19, 2026 |
| Claude Haiku 3.5  | `claude-3-5-haiku-20241022`   | Feb 19, 2026 |
| Claude Opus 3     | `claude-3-opus-20240229`      | Jan 5, 2026 |
| Claude Sonnet 3.5 | `claude-3-5-sonnet-20241022`  | Oct 28, 2025 |
| Claude Sonnet 3.5 | `claude-3-5-sonnet-20240620`  | Oct 28, 2025 |
| Claude Sonnet 3   | `claude-3-sonnet-20240229`    | Jul 21, 2025 |
| Claude 2.1        | `claude-2.1`                  | Jul 21, 2025 |
| Claude 2.0        | `claude-2.0`                  | Jul 21, 2025 |

| 友好名称 | 完整 ID | 退役时间 |
|-------------------|-------------------------------|-------------|
| Claude Sonnet 3.7 | `claude-3-7-sonnet-20250219` | 2026 年 2 月 19 日 |
| Claude Haiku 3.5 | `claude-3-5-haiku-20241022` | 2026 年 2 月 19 日 |
| Claude Opus 3 | `claude-3-opus-20240229` | 2026 年 1 月 5 日 |
| Claude Sonnet 3.5 | `claude-3-5-sonnet-20241022` | 2025 年 10 月 28 日 |
| Claude Sonnet 3.5 | `claude-3-5-sonnet-20240620` | 2025 年 10 月 28 日 |
| Claude Sonnet 3 | `claude-3-sonnet-20240229` | 2025 年 7 月 21 日 |
| Claude 2.1 | `claude-2.1` | 2025 年 7 月 21 日 |
| Claude 2.0 | `claude-2.0` | 2025 年 7 月 21 日 |

## Resolving User Requests / 解析用户请求

When a user asks for a model by name, use this table to find the correct model ID:

当用户按名称要求某个模型时，使用此表找到正确的模型 ID：

| User says...                              | Use this model ID              |
|-------------------------------------------|--------------------------------|
| "fable", "most capable model"             | `claude-fable-5-1`                 |
| "most powerful"                           | `claude-fable-5-1`                 |
| "mythos", "mythos 5.1"                    | `claude-mythos-5-1` (Project Glasswing participants only; otherwise use `claude-fable-5-1`) |
| "fable 5", "mythos 5" (previous version) | `claude-fable-5` / `claude-mythos-5` (still served; prefer `claude-fable-5-1` for new work) |
| "mythos preview"                          | `claude-mythos-5-1` (successor to `claude-mythos-preview` - see migration guide) |
| "opus"                                    | `claude-opus-5-5`                   |
| "opus 5"                                  | `claude-opus-5`             |
| "opus 5.5"                                | `claude-opus-5-5` |
| "opus 4.8"                                | `claude-opus-4-8`              |
| "opus 4.7"                                | `claude-opus-4-7`              |
| "opus 4.6"                                | `claude-opus-4-6`              |
| "opus 4.5"                                | `claude-opus-4-5`              |
| "opus 4.1"                                | `claude-opus-4-1` (deprecated, retires 2026-08-05 - suggest `claude-opus-5-5`) |
| "opus 4", "opus 4.0"                      | `claude-opus-4-0` (deprecated - suggest `claude-opus-5-5`) |
| "sonnet", "balanced"                      | `claude-sonnet-5-5`           |
| "sonnet 5"                                | `claude-sonnet-5`           |
| "sonnet 5.5"                              | `claude-sonnet-5-5` |
| "cheapest sonnet", "newest sonnet", "latest sonnet" (any attribute phrasing) | `claude-sonnet-5-5` |
| "sonnet 4.6"                              | `claude-sonnet-4-6`            |
| "sonnet 4.5"                              | `claude-sonnet-4-5`            |
| "sonnet 4", "sonnet 4.0"                  | `claude-sonnet-4-0` (deprecated - suggest `claude-sonnet-5-5`) |
| "sonnet 3.7"                              | Retired - suggest `claude-sonnet-5-5` |
| "sonnet 3.5"                              | Retired - suggest `claude-sonnet-5-5` |
| "haiku", "fast", "cheap"                  | `claude-haiku-4-5`             |
| "haiku 4.5"                               | `claude-haiku-4-5`             |
| "haiku 3.5"                               | Retired - suggest `claude-haiku-4-5` |
| "haiku 3"                                 | Deprecated - suggest `claude-haiku-4-5` |

| 用户说…… | 使用此模型 ID |
|-------------------------------------------|--------------------------------|
| "fable"、"最有能力的模型" | `claude-fable-5-1` |
| "最强大" | `claude-fable-5-1` |
| "mythos"、"mythos 5.1" | `claude-mythos-5-1`（仅限 Project Glasswing 参与者；否则使用 `claude-fable-5-1`） |
| "fable 5"、"mythos 5"（上一个版本） | `claude-fable-5` / `claude-mythos-5`（仍在服务；新工作优先使用 `claude-fable-5-1`） |
| "mythos preview" | `claude-mythos-5-1`（`claude-mythos-preview` 的后继者——见迁移指南） |
| "opus" | `claude-opus-5-5` |
| "opus 5" | `claude-opus-5` |
| "opus 5.5" | `claude-opus-5-5` |
| "opus 4.8" | `claude-opus-4-8` |
| "opus 4.7" | `claude-opus-4-7` |
| "opus 4.6" | `claude-opus-4-6` |
| "opus 4.5" | `claude-opus-4-5` |
| "opus 4.1" | `claude-opus-4-1`（已弃用，2026-08-05 退役——建议 `claude-opus-5-5`） |
| "opus 4"、"opus 4.0" | `claude-opus-4-0`（已弃用——建议 `claude-opus-5-5`） |
| "sonnet"、"均衡" | `claude-sonnet-5-5` |
| "sonnet 5" | `claude-sonnet-5` |
| "sonnet 5.5" | `claude-sonnet-5-5` |
| "最便宜的 sonnet"、"最新的 sonnet"（任何按属性描述的说法） | `claude-sonnet-5-5` |
| "sonnet 4.6" | `claude-sonnet-4-6` |
| "sonnet 4.5" | `claude-sonnet-4-5` |
| "sonnet 4"、"sonnet 4.0" | `claude-sonnet-4-0`（已弃用——建议 `claude-sonnet-5-5`） |
| "sonnet 3.7" | 已退役——建议 `claude-sonnet-5-5` |
| "sonnet 3.5" | 已退役——建议 `claude-sonnet-5-5` |
| "haiku"、"快"、"便宜" | `claude-haiku-4-5` |
| "haiku 4.5" | `claude-haiku-4-5` |
| "haiku 3.5" | 已退役——建议 `claude-haiku-4-5` |
| "haiku 3" | 已弃用——建议 `claude-haiku-4-5` |
