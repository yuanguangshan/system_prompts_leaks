<!-- BILINGUAL-EN-ZH -->
# Prompt Caching - Design & Optimization / 提示词缓存 - 设计与优化

This file covers how to design prompt-building code for effective caching. For language-specific syntax, see the `## Prompt Caching` section in each language's README or single-file doc.

本文件介绍如何设计提示词构建代码以实现有效缓存。各语言特定的语法请参见每种语言的 README 或单文件文档中的 `## Prompt Caching` 一节。

## The one invariant everything follows from / 所有结论都源自的那条不变式

**Prompt caching is a prefix match. Any change anywhere in the prefix invalidates everything after it.**

**提示词缓存是前缀匹配。前缀中任何位置的任何改动都会使其之后的一切失效。**

The cache key is derived from the exact bytes of the rendered prompt up to each `cache_control` breakpoint. A single byte difference at position N - a timestamp, a reordered JSON key, a different tool in the list - invalidates the cache for all breakpoints at positions >= N.

缓存键由渲染后的提示词到每个 `cache_control` 断点为止的精确字节派生。位置 N 上的一个字节差异——一个时间戳、一个被重新排序的 JSON 键、列表中一个不同的工具——会使位置 >= N 的所有断点的缓存失效。

Render order is: `tools` -> `system` -> `messages`. A breakpoint on the last system block caches both tools and system together.

渲染顺序为：`tools` -> `system` -> `messages`。最后一个 system 块上的断点会把工具和系统提示词一起缓存。

Design the prompt-building path around this constraint. Get the ordering right and most caching works for free. Get it wrong and no amount of `cache_control` markers will help.

请围绕这一约束设计提示词构建路径。顺序排对了，大部分缓存就自动生效；排错了，再多的 `cache_control` 标记也无济于事。

---

## Workflow for optimizing existing code / 优化既有代码的工作流

When asked to add or optimize caching:

当被要求添加或优化缓存时：

1. **Trace the prompt assembly path.** Find where `system`, `tools`, and `messages` are constructed. Identify every input that flows into them.
   **追踪提示词组装路径。**找到 `system`、`tools` 与 `messages` 在何处构造，识别流入其中的每一个输入。
2. **Classify each input by stability:**
   **按稳定性对每个输入分类：**
   - Never changes -> belongs early in the prompt, before any breakpoint
     从不变化 -> 应放在提示词前部，位于任何断点之前
   - Changes per-session -> belongs after the global prefix, cache per-session
     每个会话变化 -> 应放在全局前缀之后，按会话缓存
   - Changes per-turn -> belongs at the end, after the last breakpoint
     每轮变化 -> 应放在末尾，最后一个断点之后
   - Changes per-request (timestamps, UUIDs, random IDs) -> **eliminate or move to the very end**
     每个请求变化（时间戳、UUID、随机 ID）-> **消除它或移到最末尾**
3. **Check rendered order matches stability order.** Stable content must physically precede volatile content. If a timestamp is interpolated into the system prompt header, everything after it is uncacheable regardless of markers.
   **检查渲染顺序与稳定性顺序一致。**稳定内容必须在物理位置上先于易变内容。如果时间戳被插值进系统提示词头部，那么无论加什么标记，其后的一切都无法缓存。
4. **Place breakpoints at stability boundaries.** See placement patterns below.
   **在稳定性边界处放置断点。**参见下文的放置模式。
5. **Audit for silent invalidators.** See anti-patterns table.
   **排查静默失效源。**参见反模式表。

---

## Placement patterns / 放置模式

### Large system prompt shared across many requests / 跨多个请求共享的大型系统提示词

Put a breakpoint on the last system text block. If there are tools, they render before system - the marker on the last system block caches tools + system together.

在最后一个 system 文本块上放置断点。如果有工具，它们渲染在 system 之前——最后一个 system 块上的标记会把工具 + 系统提示词一起缓存。

```js
"system": [
  {"type": "text", "text": "<large shared prompt>", "cache_control": {"type": "ephemeral"}}
]
```

### Multi-turn conversations / 多轮对话

Put a breakpoint on the last content block of the most-recently-appended turn. Each subsequent request reuses the entire prior conversation prefix. Earlier breakpoints remain valid read points, so hits accrue incrementally as the conversation grows.

在最近追加那一轮的最后一个内容块上放置断点。之后的每个请求都会复用此前整个对话前缀。更早的断点仍是有效的读取点，因此随着对话增长，命中会逐步累积。

```js
// Last content block of the last user turn
messages[-1].content[-1].cache_control = {"type": "ephemeral"}
```

### Shared prefix, varying suffix / 共享前缀，变化后缀

Many requests share a large fixed preamble (few-shot examples, retrieved docs, instructions) but differ in the final question. Put the breakpoint at the end of the **shared** portion, not at the end of the whole prompt - otherwise every request writes a distinct cache entry and nothing is ever read.

许多请求共享一大段固定前言（few-shot 示例、检索到的文档、指令），只是最后的问题不同。把断点放在**共享**部分的末尾，而不是整个提示词的末尾——否则每个请求都会写入一个互不相同的缓存条目，而永远读不到缓存。

```js
"messages": [{"role": "user", "content": [
  {"type": "text", "text": "<shared context>", "cache_control": {"type": "ephemeral"}},
  {"type": "text", "text": "<varying question>"}  // no marker - differs every time
]}]
```

### Mid-conversation system messages / 对话中途的 system 消息

**Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, and Claude Sonnet 5.5; no beta header. Not available on Claude Sonnet 5** - use top-level `system` there. (Sources conflict on Claude Sonnet 5: the model config marks it supported, but every canonical docs page omits it. Treat it as unsupported and catch the 400.) When an operator instruction arrives mid-conversation - a mode switch, updated context, dynamically injected state - send it as `{"role": "system", "content": "..."}` appended to `messages[]`, rather than editing top-level `system`. Editing top-level `system` changes the prefix ahead of the entire conversation history, so every cached turn is re-processed uncached; a `role: "system"` message sits after the history and leaves the cached prefix intact.

**支持 Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1 与 Claude Sonnet 5.5；无 beta 头。Claude Sonnet 5 上不可用**——在该模型上请使用顶层 `system`。（关于 Claude Sonnet 5，各来源说法不一：模型配置标记其支持，但所有正式文档页面均未列出。应将其视为不支持并捕获那个 400 错误。）当操作者指令在对话中途到达——模式切换、上下文更新、动态注入的状态——应将其作为 `{"role": "system", "content": "..."}` 消息追加到 `messages[]`，而不是修改顶层 `system`。修改顶层 `system` 会改变位于整个对话历史之前的前缀，因此每个已缓存的轮次都要以未缓存方式重新处理；而 `role: "system"` 消息位于历史之后，缓存前缀保持原样。

【评论】文档在此处如实记录了自身来源之间的矛盾（模型配置与正式文档不一致），并给出保守的工程处置：当作不支持、捕获 400 后回退——这是典型的"宁可保守"的 API 兼容性写法。

```js
// Top-level system stays byte-identical; new instruction goes after the cached history
"system": [{"type": "text", "text": "<stable core>", "cache_control": {"type": "ephemeral"}}],
"messages": [
  ...history,
  {"role": "user", "content": "..."},
  {"role": "system", "content": "Terse mode enabled - keep responses under 40 words."}
]
```

This is also the prompt-injection-safe replacement for embedding operator instructions as text inside a user turn (the `<system-reminder>` pattern): both have the same caching profile, but `role: "system"` is the non-spoofable operator channel, whereas text inside user/tool content can be forged by anything that writes to user-visible input.

这也是把操作者指令以文本形式嵌入用户轮次（即 `<system-reminder>` 模式）的防提示词注入替代方案：两者的缓存特征相同，但 `role: "system"` 是不可伪造的操作者通道，而 user/tool 内容中的文本可以被任何能写入用户可见输入的东西伪造。

【评论】这里点明了一个安全设计：`role: "system"` 被定位为"操作者专用、不可伪造"的通道，与普通用户输入中的 `<system-reminder>` 文本（可被提示词注入伪造）在信任级别上有本质区别，二者只是缓存特征恰好相同。

Must follow a `role: "user"` message (or an `assistant` message ending in server-tool use), and must be either the last entry in `messages` or be followed by an `assistant` turn; cannot be `messages[0]` - use top-level `system` for the initial prompt. Content is text-only. Unsupported models return a 400 (`BadRequestError`: `role 'system' is not supported on this model`); catch that error and fall back to putting the instruction in a user-turn `<system-reminder>` block.

它必须紧跟在 `role: "user"` 消息之后（或紧跟在以服务端工具调用结尾的 `assistant` 消息之后），并且必须是 `messages` 的最后一个条目，或者后面跟着一个 `assistant` 轮次；不能是 `messages[0]`——初始提示词请使用顶层 `system`。内容只能为纯文本。不支持的模型会返回 400（`BadRequestError`：`role 'system' is not supported on this model`）；捕获该错误并回退为把指令放进用户轮次的 `<system-reminder>` 块中。

**Per-turn reminders in a tool loop: turn-scoped messages, never deleted.** A reminder injected into history and removed on the next request is a history edit - the cache misses from that point and, on Claude Fable 5.1, Claude Opus 5.5, and Claude Sonnet 5.5, every later thinking block is invalidated. Instead give the `role: "system"` message `clear_at: "next_user_message"` (beta `mid-conversation-system-clear-at-2026-08-21`; same models and platforms as mid-conversation system messages): it renders for one turn, then stays in the transcript cleared - costing no input tokens, not cache-eligible (`cache_control` on it is a 400; put the breakpoint on the preceding user turn), and still part of the prefix. Append a fresh copy after each `tool_result` message and leave earlier copies in place; without the beta, a `text` block after the `tool_result` blocks in the same user message, earlier copies kept. Separately, per-message effort (beta `mid-conversation-output-config-2026-07-01`; Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5, Claude Opus 5.5, and Claude Sonnet 5.5 with thinking on; Claude API and Google Cloud): a `role: "system"` message with `content: []` and `output_config: {effort: ...}` changes effort from the next user turn on **without** the messages-cache invalidation that a top-level `effort` change causes, and is exempt from the placement rules (it can sit anywhere) - see the Invalidation hierarchy below and `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features.

**工具循环中的每轮提醒：使用轮次作用域消息，绝不删除。**注入历史、又在下一次请求中被移除的提醒属于历史编辑——缓存从那一点起失效，并且在 Claude Fable 5.1、Claude Opus 5.5 与 Claude Sonnet 5.5 上，其后每一个思考块都会失效。正确做法是给 `role: "system"` 消息设置 `clear_at: "next_user_message"`（beta `mid-conversation-system-clear-at-2026-08-21`；与对话中途 system 消息相同的模型和平台）：它渲染一轮，随后以已清除状态留在对话记录中——不消耗输入 token、不参与缓存（对其使用 `cache_control` 会返回 400；请把断点放在前面的用户轮次上），且仍是前缀的一部分。在每条 `tool_result` 消息后追加一份新副本，并保留更早的副本；在没有该 beta 的情况下，则使用同一条用户消息中位于 `tool_result` 块之后的 `text` 块，同样保留更早的副本。另外，还有按消息设置 effort 的能力（beta `mid-conversation-output-config-2026-07-01`；Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5、Claude Opus 5.5 与开启思考的 Claude Sonnet 5.5；Claude API 与 Google Cloud）：带有 `content: []` 与 `output_config: {effort: ...}` 的 `role: "system"` 消息从下一个用户轮次起更改 effort，**而不会**像顶层 `effort` 变更那样使消息缓存失效，且不受放置规则约束（可以放在任何位置）——参见下文的 Invalidation hierarchy（失效层级）与 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 -> New API features。

### Prompts that change from the beginning every time / 每次都从头变化的提示词

Don't cache. If the first 1K tokens differ per request, there is no reusable prefix. Adding `cache_control` only pays the cache-write premium with zero reads. Leave it off.

不要缓存。如果每个请求的前 1K token 都不同，就不存在可复用的前缀。添加 `cache_control` 只会支付缓存写入溢价而读取为零。不要加。

---

## Architectural guidance / 架构层面指导

These are the decisions that matter more than marker placement. Fix these first.

这些是比标记放置更重要的决策。先解决它们。

**Keep the system prompt frozen.** Don't interpolate "current date: X", "mode: Y", "user name: Z" into the system prompt - those sit at the front of the prefix and invalidate everything downstream. Inject dynamic context later in `messages` instead - as a `{"role": "system", ...}` message where supported (see § Mid-conversation system messages above), or as text in a user message otherwise. A message at turn 5 invalidates nothing before turn 5.

**保持系统提示词冻结。**不要把"current date: X"、"mode: Y"、"user name: Z"这类内容插值进系统提示词——它们位于前缀前端，会使下游一切失效。应改为把动态上下文在 `messages` 中较晚注入——在支持的地方作为 `{"role": "system", ...}` 消息（参见上文 § Mid-conversation system messages），否则作为用户消息中的文本。第 5 轮的消息不会使第 5 轮之前的任何内容失效。

**Don't change tools or model mid-conversation.** Tools render at position 0; adding, removing, or reordering a tool invalidates the entire cache. Same for switching models (caches are model-scoped). If you need "modes", don't swap the tool set - give Claude a tool that records the mode transition, or pass the mode as message content. Serialize tools deterministically (sort by name).

**对话中途不要更换工具或模型。**工具渲染在位置 0；添加、移除或重排工具会使整个缓存失效。切换模型同理（缓存按模型隔离）。如果需要"模式"，不要更换工具集——给 Claude 一个记录模式切换的工具，或者把模式作为消息内容传入。以确定性的方式序列化工具（按名称排序）。

**Fork operations must reuse the parent's exact prefix.** Side computations (summarization, compaction, sub-agents) often spin up a separate API call. If the fork rebuilds `system` / `tools` / `model` with any difference, it misses the parent's cache entirely. Copy the parent's `system`, `tools`, and `model` verbatim, then append fork-specific content at the end.

**分叉操作必须复用父调用的精确前缀。**旁路计算（摘要、压缩、子智能体）常常会另起一个独立的 API 调用。如果分叉在重建 `system` / `tools` / `model` 时存在任何差异，它将完全错过父调用的缓存。应逐字复制父调用的 `system`、`tools` 与 `model`，再把分叉特有的内容追加到末尾。

---

## Silent invalidators / 静默失效源

When reviewing code, grep for these inside anything that feeds the prompt prefix:

审查代码时，请在一切进入提示词前缀的内容中 grep 以下模式：

| Pattern | Why it breaks caching |
|---|---|
| `datetime.now()` / `Date.now()` / `time.time()` in system prompt | Prefix changes every request |
| `uuid4()` / `crypto.randomUUID()` / request IDs early in content | Same - every request is unique |
| `json.dumps(d)` without `sort_keys=True` / iterating a `set` | Non-deterministic serialization -> prefix bytes differ |
| f-string interpolating session/user ID into system prompt | Per-user prefix; no cross-user sharing |
| Conditional system sections (`if flag: system += ...`) | Every flag combination is a distinct prefix |
| `tools=build_tools(user)` where set varies per user | Tools render at position 0; nothing caches across users |

| 模式 | 为何破坏缓存 |
|---|---|
| 系统提示词中的 `datetime.now()` / `Date.now()` / `time.time()` | 前缀每个请求都变化 |
| 内容前部的 `uuid4()` / `crypto.randomUUID()` / 请求 ID | 同理——每个请求都唯一 |
| 不带 `sort_keys=True` 的 `json.dumps(d)` / 迭代 `set` | 序列化不确定 -> 前缀字节不同 |
| 把会话/用户 ID 用 f-string 插值进系统提示词 | 前缀按用户隔离；无法跨用户共享 |
| 条件化的 system 分段（`if flag: system += ...`） | 每种标志组合都是一个不同的前缀 |
| 工具集合因用户而异的 `tools=build_tools(user)` | 工具渲染在位置 0；跨用户无法命中缓存 |

Fix by moving the dynamic piece after the last breakpoint, making it deterministic, or deleting it if it's not load-bearing.

修复方式：把动态部分移到最后一个断点之后、使其变得确定，或者如果它并非真正起作用的内容就删掉。

---

## API reference / API 参考

```js
"cache_control": {"type": "ephemeral"}              // 5-minute TTL (default)
"cache_control": {"type": "ephemeral", "ttl": "1h"} // 1-hour TTL
```

- Max **4** `cache_control` breakpoints per request.
  每个请求最多 **4** 个 `cache_control` 断点。
- Goes on any content block: system text blocks, tool definitions, message content blocks (`text`, `image`, `tool_use`, `tool_result`, `document`).
  可放在任何内容块上：system 文本块、工具定义、消息内容块（`text`、`image`、`tool_use`、`tool_result`、`document`）。
- Top-level `cache_control` on `messages.create()` auto-places on the last cacheable block - simplest option when you don't need fine-grained placement (§ Automatic vs explicit breakpoints).
  `messages.create()` 上的顶层 `cache_control` 会自动放置在最后一个可缓存块上——不需要细粒度放置时这是最简单的选项（§ Automatic vs explicit breakpoints）。
- Caches are isolated per workspace on the Claude API, Claude Platform on AWS, and Microsoft Foundry (per organization on Amazon Bedrock and Google Cloud), and never shared across organizations. Traffic for the same prompt split across workspaces writes and reads separate entries - check this before blaming a low hit rate on the prompt.
  在 Claude API、AWS 上的 Claude Platform 与 Microsoft Foundry 上，缓存按工作区隔离（在 Amazon Bedrock 与 Google Cloud 上按组织隔离），且绝不在组织之间共享。同一提示词的流量若分散在多个工作区，写入和读取的都是各自独立的条目——在把低命中率归咎于提示词之前，先检查这一点。
- Minimum cacheable prefix is model-dependent. Shorter prefixes silently won't cache even with a marker - no error, just `cache_creation_input_tokens: 0`:
  最小可缓存前缀因模型而异。更短的前缀即使加了标记也会静默地不缓存——没有报错，只是 `cache_creation_input_tokens: 0`：

| Model | Minimum |
|---|---:|
| Claude Opus 5.5, Claude Opus 5, Claude Fable 5, Claude Mythos 5, Claude Fable 5.1, Claude Mythos 5.1, Claude Sonnet 5.5 (check the prompt caching docs before relying on its value) | 512 tokens |
| Opus 4.8, Claude Sonnet 5, Sonnet 4.6, Sonnet 4.5, Opus 4.1, Opus 4, Sonnet 4 | 1024 tokens |
| Opus 4.7, Mythos Preview, Haiku 3.5 | 2048 tokens |
| Opus 4.6, Opus 4.5, Haiku 4.5 | 4096 tokens |

| 模型 | 最小值 |
|---|---:|
| Claude Opus 5.5、Claude Opus 5、Claude Fable 5、Claude Mythos 5、Claude Fable 5.1、Claude Mythos 5.1、Claude Sonnet 5.5（在依赖其数值前请查阅提示词缓存文档） | 512 tokens |
| Opus 4.8、Claude Sonnet 5、Sonnet 4.6、Sonnet 4.5、Opus 4.1、Opus 4、Sonnet 4 | 1024 tokens |
| Opus 4.7、Mythos Preview、Haiku 3.5 | 2048 tokens |
| Opus 4.6、Opus 4.5、Haiku 4.5 | 4096 tokens |

**The minimum is not monotonic across generations** - 512 on the newest models, but 4096 on Opus 4.6/4.5 and Haiku 4.5. A 3K-token prompt caches on Claude Opus 5, Opus 4.8, and Sonnet 4.5, and silently won't on Opus 4.6 or Haiku 4.5. Claude Opus 5 halves the Opus 4.8 minimum (1024 -> 512), so prompts previously too short to cache now create entries with no code change.

**该最小值在各代之间并非单调**——最新模型是 512，而 Opus 4.6/4.5 与 Haiku 4.5 是 4096。一个 3K token 的提示词在 Claude Opus 5、Opus 4.8 与 Sonnet 4.5 上可以缓存，在 Opus 4.6 或 Haiku 4.5 上则会静默失败。Claude Opus 5 把 Opus 4.8 的最小值减半（1024 -> 512），因此以前短到无法缓存的提示词现在无需改代码即可创建缓存条目。

These minimums apply on **every** platform where the model is available - the old Amazon Bedrock override for Claude Fable 5.1 was removed, and no per-platform exception remains.

这些最小值适用于该模型可用的**每一个**平台——Amazon Bedrock 上针对 Claude Fable 5.1 的旧覆盖已被移除，不再存在任何按平台的例外。

**Economics:** Cache reads cost ~0.1× base input price - **0.025× on Claude Fable 5.1** ($0.25/MTok, on Claude Mythos 5.1 too) and 0.05× on Claude Opus 5.5 ($0.20/MTok), which moves every break-even below proportionally. Cache writes cost **1.25× for 5-minute TTL, 2× for 1-hour TTL**. Break-even depends on TTL: with 5-minute TTL, two requests break even (1.25× + 0.1× = 1.35× vs 2× uncached); with 1-hour TTL, you need at least three requests (2× + 0.2× = 2.2× vs 3× uncached). The 1-hour TTL keeps entries alive across gaps in bursty traffic, but the doubled write cost means it needs more reads to pay off.

**经济账：**缓存读取约为基础输入价格的 0.1×——**Claude Fable 5.1 上为 0.025×**（$0.25/MTok，Claude Mythos 5.1 亦然），Claude Opus 5.5 上为 0.05×（$0.20/MTok），这使得下文每个盈亏平衡点都按比例下移。缓存写入**5 分钟 TTL 为 1.25×，1 小时 TTL 为 2×**。盈亏平衡取决于 TTL：5 分钟 TTL 时两个请求即可打平（1.25× + 0.1× = 1.35×，对比不缓存的 2×）；1 小时 TTL 时至少需要三个请求（2× + 0.2× = 2.2×，对比不缓存的 3×）。1 小时 TTL 能让缓存条目在突发流量的间隙中存活，但写入成本翻倍意味着需要更多读取才划算。

### Choosing the TTL / 选择 TTL

A cache read refreshes the entry's timer at no additional cost, on either TTL. The lifetime is measured from the **start** of the request that writes or reads the entry - generation time counts against it, so a 4-minute generation leaves about 1 minute for the next request to start before a 5-minute entry expires. Requests that share a prefix and start less than 5 minutes apart keep the 5-minute cache warm indefinitely - the 1-hour TTL buys nothing there except the doubled write price. Choose by the start-to-start gap between requests that share the prefix:

无论哪种 TTL，缓存读取都会以零额外成本刷新条目的计时器。生存期从写入或读取该条目的请求**开始**时刻算起——生成时间也计入其中，因此一次 4 分钟的生成之后，5 分钟条目只给下一个请求留了约 1 分钟的启动时间。共享同一前缀且启动间隔小于 5 分钟的请求能让 5 分钟缓存无限期保温——在这种情况下，1 小时 TTL 除了让写入价格翻倍之外毫无收益。按共享前缀的请求之间的启动间隔来选择：

| Start-to-start gap between requests sharing the prefix | TTL |
|---|---|
| Under 5 minutes (continuous traffic; agent loops whose turns generate well under 5 minutes) | 5-minute - every request refreshes it; strictly cheaper |
| 5-60 minutes (a user who replies after 20 minutes; an agentic side-task or a generation that runs past 5 minutes between reads) | 1-hour - the only window where the 2× write pays off |
| Over an hour | Neither helps directly - re-warm on a schedule (§ Pre-warming the cache) or accept the cold miss |

| 共享前缀的请求之间的启动间隔 | TTL |
|---|---|
| 不到 5 分钟（持续流量；每轮生成远低于 5 分钟的智能体循环） | 5 分钟——每个请求都会刷新它；严格更便宜 |
| 5-60 分钟（用户 20 分钟后才回复；智能体旁路任务或一次生成在两次读取之间超过 5 分钟） | 1 小时——2× 写入唯一能回本的区间 |
| 超过一小时 | 两者都无直接帮助——按计划重新保温（§ Pre-warming the cache）或接受冷失效 |

**Claude Fable 5.1 / Claude Mythos 5.1: a keep-alive is usually cheaper than the 1-hour TTL.** With cache reads at 0.025x on Claude Fable 5.1 and Claude Mythos 5.1 (versus 0.05x on Claude Opus 5.5 and 0.1x elsewhere - see Economics above) a miss is much more expensive *relative to a hit*, and a read is nearly free - so for the 5-60 minute gap, instead of paying the 2x write for the 1-hour TTL, stay on the default 5-minute TTL and, while idle, re-send the previous request with `max_tokens: 0` shortly before the entry would expire. That request refreshes the entry's timer and bills only a cheap cache read (no output tokens). At Claude Fable 5.1 prices this beats the 1-hour TTL unless pauses regularly approach an hour. `max_tokens: 0` follows § Pre-warming's rejected combinations; on these models the ones that can arise are `stream: true`, structured outputs, and Batches (forced `tool_choice` and `thinking.type: "enabled"` are already 400s here). Send the keep-alive with `stream` off - streaming is a transport option, not part of the cached prefix, so dropping it for this one request costs nothing - and where the request can't be reshaped that way, with structured outputs (`output_config.format`) or inside a Message Batches request, use the 1-hour TTL instead. The prompt-caching page (`shared/live-sources.md`) has a cost comparison on a sample workload and an example keep-alive request.

**Claude Fable 5.1 / Claude Mythos 5.1：保活请求通常比 1 小时 TTL 更便宜。**由于 Claude Fable 5.1 与 Claude Mythos 5.1 上的缓存读取为 0.025x（相比之下 Claude Opus 5.5 为 0.05x、其他模型为 0.1x——见上文 Economics），未命中*相对于命中*的代价高得多，而一次读取几乎免费——因此在 5-60 分钟的间隙场景，与其为 1 小时 TTL 支付 2x 写入，不如留在默认的 5 分钟 TTL 上，并在条目即将过期前用 `max_tokens: 0` 重发上一个请求以保持活跃。该请求会刷新条目的计时器，且只计费一次廉价的缓存读取（无输出 token）。按 Claude Fable 5.1 的价格计算，除非暂停经常接近一小时，否则这比 1 小时 TTL 更划算。`max_tokens: 0` 遵循 § Pre-warming 的被拒绝组合；在这些模型上实际可能遇到的是 `stream: true`、结构化输出与 Batches（强制 `tool_choice` 与 `thinking.type: "enabled"` 在这里本来就会返回 400）。发送保活请求时请关闭 `stream`——流式是传输选项，不属于被缓存的前缀，因此仅这一次请求去掉它没有任何代价；而在请求无法这样改造、带结构化输出（`output_config.format`）或位于 Message Batches 请求内的情况下，请改用 1 小时 TTL。提示词缓存页面（`shared/live-sources.md`）给出了一个示例负载上的成本对比和一个保活请求示例。

On the Claude API, cache reads also do not count toward input-token rate limits on most models (Haiku 3.5 is the documented exception - see the rate-limits doc), so keeping entries alive across gaps can raise effective throughput as well as cut cost.

在 Claude API 上，对大多数模型而言缓存读取也不计入输入 token 速率限制（Haiku 3.5 是文档记载的例外——参见速率限制文档），因此让条目跨间隙存活既能降低成本，也能提升有效吞吐。

---

## Automatic vs explicit breakpoints / 自动与显式断点

Automatic caching is a top-level `cache_control` field on the request, not on any content block. The system places the breakpoint on the last cacheable block and moves it forward as the conversation grows; if the last block isn't an eligible target it silently walks backward to the nearest eligible one, and skips caching if none is found. The automatic breakpoint defaults to the 5-minute TTL (the top-level field accepts `ttl: "1h"`) and consumes one of the 4 breakpoint slots. It composes with explicit markers in the same request, with two documented 400s: all 4 slots already taken by explicit markers, and an explicit marker on the last block whose TTL differs from the top-level field's (an explicit marker there with the same TTL makes automatic caching a no-op).

自动缓存是请求上的顶层 `cache_control` 字段，而不是任何内容块上的字段。系统会把断点放在最后一个可缓存块上，并随对话增长向前移动；若最后一个块不是合格目标，它会静默回退到最近的合格块，若一个也找不到则跳过缓存。自动断点默认 5 分钟 TTL（顶层字段接受 `ttl: "1h"`），并占用 4 个断点槽位之一。它与同一请求中的显式标记可以组合使用，但有两种文档记载的 400：4 个槽位已被显式标记占满，以及最后一个块上的显式标记与顶层字段的 TTL 不同（若该处的显式标记与顶层字段 TTL 相同，则自动缓存成为无操作）。

Automatic is the right default for multi-turn conversations - the multi-turn placement pattern above with no marker bookkeeping. Use explicit breakpoints when:

对于多轮对话，自动是正确的默认——即上文的多轮放置模式，且无需人工管理标记。以下情况使用显式断点：

| Situation | Why automatic is the wrong tool |
|---|---|
| The prompt ends in unique per-request content (retrieved rows, per-request context, the one-off question) | The automatic breakpoint lands after the unique tail, so every request pays the write premium on bytes that are never read back - a pure surcharge. The signature: `cache_creation_input_tokens` on every request while `cache_read_input_tokens` never covers the full shared prefix. Put an explicit marker at the end of the shared portion instead (§ Shared prefix, varying suffix). |
| Sections change at different frequencies (tools never, context daily, conversation per-turn) | Automatic places exactly one breakpoint; multiple stability boundaries need explicit markers. |
| One block should be 1-hour TTL and another 5-minute | Per-block TTL requires explicit markers - and entries with the longer TTL must appear before shorter ones (a 1-hour entry must appear before any 5-minute entries). |
| A single turn appends more than 20 positions (consecutive tool_use runs, and tool_result runs, each collapse to one position) | The lookback can miss the previous entry - § 20-block lookback window. |
| A platform or integration without automatic caching (check `shared/platform-availability.md`) | The top-level field is rejected there - use explicit markers only. |

| 情形 | 为何自动缓存不是合适的工具 |
|---|---|
| 提示词以每个请求独有的内容结尾（检索到的行、按请求的上下文、一次性问题） | 自动断点落在独特尾部之后，每个请求都要为永远不会被读回的字节支付写入溢价——纯粹的额外开销。特征签名：每个请求都有 `cache_creation_input_tokens`，而 `cache_read_input_tokens` 始终覆盖不了完整的共享前缀。应改为在共享部分末尾放显式标记（§ Shared prefix, varying suffix）。 |
| 各部分变化频率不同（工具从不变、上下文按天变、对话按轮变） | 自动只放置一个断点；多个稳定性边界需要显式标记。 |
| 一个块需要 1 小时 TTL，另一个需要 5 分钟 | 按块设置 TTL 需要显式标记——且长 TTL 的条目必须出现在短 TTL 之前（1 小时条目必须出现在任何 5 分钟条目之前）。 |
| 单轮追加超过 20 个位置（连续的 tool_use 段与 tool_result 段各自折叠为一个位置） | 回看可能错过上一个条目——§ 20-block lookback window。 |
| 不支持自动缓存的平台或集成（查 `shared/platform-availability.md`） | 顶层字段在那里会被拒绝——只能使用显式标记。 |

**The robust combination for agent loops:** one explicit breakpoint on the last block of the static system prefix - the expensive shared part gets a guaranteed read point that survives whatever happens later in `messages` - plus top-level automatic caching for the growing conversation tail (where automatic caching is available - `shared/platform-availability.md`). If § Choosing the TTL puts you on the 1-hour TTL, set `ttl: "1h"` on the explicit marker as well as on the top-level field. Both default to 5 minutes, and a 1-hour automatic entry after a 5-minute marker breaks the ordering rule above: longer TTLs must come first. The reverse, a 1-hour marker with a 5-minute tail, is allowed.

**智能体循环的稳健组合：**在静态系统前缀的最后一个块上放一个显式断点——昂贵的共享部分由此获得一个保证存在的读取点，无论 `messages` 之后发生什么——再加上顶层自动缓存用于不断增长的对话尾部（在自动缓存可用的平台上——`shared/platform-availability.md`）。如果 § Choosing the TTL 让你落在 1 小时 TTL 上，请同时在显式标记和顶层字段上设置 `ttl: "1h"`。两者默认都是 5 分钟，而 5 分钟标记之后的 1 小时自动条目会违反上述排序规则：长 TTL 必须在前。反过来——1 小时标记配 5 分钟尾部——是允许的。

---

## Verifying cache hits / 验证缓存命中

The response `usage` object reports cache activity:

响应的 `usage` 对象报告缓存活动：

| Field | Meaning |
|---|---|
| `cache_creation_input_tokens` | Tokens written to cache this request (you paid the ~1.25× write premium) |
| `cache_read_input_tokens` | Tokens served from cache this request (you paid ~0.1×) |
| `input_tokens` | Tokens processed at full price (not cached) |

| 字段 | 含义 |
|---|---|
| `cache_creation_input_tokens` | 本次请求写入缓存的 token（你支付了 ~1.25× 的写入溢价） |
| `cache_read_input_tokens` | 本次请求由缓存提供的 token（你支付了 ~0.1×） |
| `input_tokens` | 以全价处理的 token（未缓存） |

If `cache_read_input_tokens` is zero across repeated requests with identical prefixes, a silent invalidator is at work - diff the rendered prompt bytes between two requests to find it.

如果前缀完全相同的重复请求中 `cache_read_input_tokens` 始终为零，说明存在静默失效源——对两个请求的渲染后提示词字节做 diff 即可找到它。

**`input_tokens` is the uncached remainder only.** Total prompt size = `input_tokens + cache_creation_input_tokens + cache_read_input_tokens`. If your agent ran for hours but `input_tokens` shows 4K, the rest was served from cache - check the sum, not the single field.

**`input_tokens` 只是没有被缓存的那部分。**提示词总大小 = `input_tokens + cache_creation_input_tokens + cache_read_input_tokens`。如果你的智能体运行了数小时而 `input_tokens` 只显示 4K，说明其余部分都由缓存提供——请看总和，而不是单个字段。

Language-specific access: `response.usage.cache_read_input_tokens` (Python/TS/Ruby), `$message->usage->cacheReadInputTokens` (PHP), `resp.Usage.CacheReadInputTokens` (Go/C#), `.usage().cacheReadInputTokens()` (Java).

各语言的访问方式：`response.usage.cache_read_input_tokens`（Python/TS/Ruby）、`$message->usage->cacheReadInputTokens`（PHP）、`resp.Usage.CacheReadInputTokens`（Go/C#）、`.usage().cacheReadInputTokens()`（Java）。

**Verify after every change, not just at setup.** The costliest caching failure in production is silent: requests keep succeeding, the bill is just higher - no error, nothing announces it. The typical shape is a regression, not a bad first implementation: caching works when written, then a later change to prompt assembly (a new dynamic field in the system prompt, a history-rewriting feature, a tool list that stopped being deterministic) misses on every request and goes unnoticed for months. The `usage` fields are the only ground truth that caching is working. Re-check them whenever prompt-assembly code changes, and prefer a standing check - an integration-test assertion that a second identical request shows `cache_read_input_tokens > 0`, or monitoring on the usage fields - over a one-time look.

**每次变更之后都要验证，而不只在搭建时。**生产中最昂贵的缓存故障是静默的：请求继续成功，只是账单更高——没有报错，没有任何东西提示你。典型形态是回归，而不是最初的实现不佳：写代码时缓存正常工作，之后对提示词组装的某次修改（系统提示词中新增的动态字段、重写历史的功能、不再确定性的工具列表）让每个请求都不命中，并数月无人察觉。`usage` 字段是缓存是否正常工作的唯一事实来源。提示词组装代码每次变更都要重新检查它们，并且比起一次性查看，更应建立常设检查——例如集成测试断言"第二次相同请求的 `cache_read_input_tokens > 0`"，或对 usage 字段做监控。

**The healthy-loop signature.** Writes bill only the delta past the highest cache hit, so in a steady multi-turn loop each request should read everything accumulated so far and write only what the last turn added:

**健康循环的特征。**写入只对最高缓存命中之后的增量计费，因此在稳定的多轮循环中，每个请求应读取到目前为止积累的全部内容，只写入上一轮新增的部分：

- `cache_read_input_tokens` - the whole prior prefix; grows turn over turn
  `cache_read_input_tokens`——此前的整个前缀；逐轮增长
- `cache_creation_input_tokens` - roughly the previous assistant output plus the newly appended input; small relative to the conversation
  `cache_creation_input_tokens`——大致等于上一条助手输出加上新追加的输入；相对整个对话很小
- `input_tokens` - just the tail after the last breakpoint
  `input_tokens`——只是最后一个断点之后的尾部

If `cache_creation_input_tokens` is instead near the full conversation size on every request, either the prefix is being rewritten upstream of the breakpoint, or the write is happening for a reason payload diffing and cache diagnostics can't localize - with thinking enabled on a model that strips prior-turn thinking blocks the invalidation is server-side (§ Invalidation hierarchy), and a single turn that appends more than 20 positions (parallel tool-call runs collapse to one position - § 20-block lookback window) pushes the previous entry out of the lookback so every request rewrites the whole conversation with byte-identical payloads (§ 20-block lookback window). Rule both show-nothing cases out first from the model and the turn shape. Reads can only land on positions where a previous request wrote a breakpoint, so the usage fields say *that* the prefix broke (reads collapse, often to zero) but not where - the payload diff or cache diagnostics below localizes the exact point.

如果 `cache_creation_input_tokens` 反而在每个请求上都接近整个对话的规模，要么是断点上游的前缀被重写了，要么是写入由于某种负载 diff 与缓存诊断都无法定位的原因而发生——在启用了思考且模型会剥离先前轮次思考块的情况下，失效发生在服务端（§ Invalidation hierarchy）；而单轮追加超过 20 个位置（并行的工具调用段折叠为一个位置——§ 20-block lookback window）会把上一个条目推出回看窗口，导致每个请求都以字节完全相同的负载重写整个对话（§ 20-block lookback window）。请先根据模型与轮次形态排除这两种"读数全无"的情形。读取只能落在此前请求写过断点的位置上，因此 usage 字段只能说明前缀*确实*断了（读取坍缩，常常归零），却说不出断在哪里——用下文的负载 diff 或缓存诊断来定位确切位置。

**Finding the invalidator.** Log several consecutive request payloads (the full JSON body) and diff adjacent pairs. In a growing conversation, adjacent payloads legitimately differ at the end (the newly appended turn); what must be byte-identical is the overlap - the previous request's prompt should reappear unchanged as a prefix of the next. Strip `cache_control` markers before diffing: the moving marker always differs between adjacent requests and is not an invalidator (previously-marked blocks are still cache hits). The first remaining divergence inside the overlapping region is the invalidation point. This catches the class of bug code review misses - nondeterministic serialization, a library reordering keys or fields, a value that changes between requests but not within one. On the Claude API, cache diagnostics (beta header `cache-diagnosis-2026-04-07`) does this comparison server-side once you opt in: send the header on **every** request - fingerprints are stored only for requests that carried it, so a one-shot retrofit fails with `previous_message_not_found` - then pass the previous response's `id` as `diagnostics.previous_message_id` and the response's `diagnostics` object names where the two requests diverged (model, system, tools, or message history). No payload logging needed. Availability: `shared/platform-availability.md`.

**找出失效源。**记录若干连续请求的负载（完整 JSON body），对相邻对做 diff。在增长的对话中，相邻负载在末尾出现差异是合理的（新追加的那一轮）；必须字节相同的是重叠部分——上一个请求的提示词应原封不动地作为下一个请求的前缀重新出现。diff 之前先剥掉 `cache_control` 标记：移动的标记在相邻请求之间必然不同，但它不是失效源（此前被标记过的块仍然是缓存命中）。重叠区域内第一处残余分歧就是失效点。这能抓住代码审查漏掉的一类 bug——不确定的序列化、重新排列键或字段的库、在请求之间变化但单个请求内不变的值。在 Claude API 上，缓存诊断（beta 头 `cache-diagnosis-2026-04-07`）在你选择加入后由服务端完成这一比较：在**每个**请求上都带上该头——指纹只为携带它的请求存储，因此一次性的补加会以 `previous_message_not_found` 失败——然后把上一个响应的 `id` 作为 `diagnostics.previous_message_id` 传入，响应的 `diagnostics` 对象会指明两个请求在哪里分歧（模型、system、工具或消息历史）。无需记录负载。可用性：`shared/platform-availability.md`。

**Unexplained writes:** `usage.cache_creation` breaks `cache_creation_input_tokens` down by TTL (`ephemeral_5m_input_tokens` / `ephemeral_1h_input_tokens`). Server tools such as web search automatically insert a 5-minute cache write after tool results when the request already uses caching - writes at a position you didn't mark; expected behavior, not an invalidator.

**无法解释的写入：**`usage.cache_creation` 按 TTL 细分 `cache_creation_input_tokens`（`ephemeral_5m_input_tokens` / `ephemeral_1h_input_tokens`）。当请求已经使用缓存时，网页搜索等服务端工具会在工具结果之后自动插入一次 5 分钟缓存写入——写入发生在你未标记的位置；这是预期行为，不是失效源。

---

## Invalidation hierarchy / 失效层级

Not every parameter change invalidates everything. The API has three cache tiers, and changes only invalidate their own tier and below:

并非每个参数变更都会使一切失效。API 有三个缓存层级，变更只使其自身层级及以下失效：

| Change | Tools cache | System cache | Messages cache |
|---|:---:|:---:|:---:|
| Tool definitions (add/remove/reorder) | No | No | No |
| Model switch | No | No | No |
| `speed`, web-search, citations toggle | Yes | No | No |
| System prompt content | Yes | No | No |
| `tool_choice`, images | Yes | Yes | No |
| `thinking` or `effort` change | model-specific | model-specific | No |
| Message content | Yes | Yes | No |

| 变更 | 工具缓存 | 系统缓存 | 消息缓存 |
|---|:---:|:---:|:---:|
| 工具定义（添加/移除/重排） | 否 | 否 | 否 |
| 切换模型 | 否 | 否 | 否 |
| `speed`、网页搜索、引用开关 | 是 | 否 | 否 |
| 系统提示词内容 | 是 | 否 | 否 |
| `tool_choice`、图片 | 是 | 是 | 否 |
| `thinking` 或 `effort` 变更 | 因模型而异 | 因模型而异 | 否 |
| 消息内容 | 是 | 是 | 否 |

Implication: you can change `tool_choice` per-request without losing the tools+system cache, and message-content changes never touch it. Thinking and `effort` changes always invalidate the messages cache, and on models that render the thinking configuration ahead of tools and system they invalidate those caches too - pin thinking and effort settings per route rather than varying them per request. Only tool-definition and model changes force a full rebuild on every model.

推论：你可以按请求更改 `tool_choice` 而不丢失工具+系统缓存，消息内容变更也从不触及它。thinking 与 `effort` 变更总会使消息缓存失效，并且在把思考配置渲染在工具与系统之前的模型上，它们也会使那些缓存失效——请按路由固定 thinking 与 effort 设置，而不是按请求变化。只有工具定义与模型变更会在所有模型上强制完全重建。

**Three of these rows have a cache-preserving escape hatch** - the tools row, the system-prompt row, and (on Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Opus 5) the `effort` row - each by moving the change out of the top-level request and into a system message inside `messages[]`, after the cached prefix. The inject-then-delete reminder pattern has its own hatch: a text block appended after the `tool_result` blocks in the user message, never deleted. **Availability differs per row** - they are not gated together:

**其中三行有保留缓存的逃生通道**——工具行、系统提示词行，以及（在 Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Opus 5 上的）`effort` 行——方法都是把变更从顶层请求移入 `messages[]` 内、缓存前缀之后的 system 消息。"注入后删除"的提醒模式有自己的通道：在用户消息中 `tool_result` 块之后追加一个 text 块，并且从不删除。**每行的可用性不同**——它们的门控并不绑定在一起：

| Top-level change that invalidates | Cache-preserving form | Available on |
|---|---|---|
| Tool definitions (add/remove) | `tool_addition` / `tool_removal` blocks - see `shared/tool-use-concepts.md` § Mid-conversation tool changes | Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, Claude Sonnet 5.5 (not Claude Sonnet 5), behind `mid-conversation-tool-changes-2026-07-01` |
| System prompt content | A `{"role": "system", "content": "..."}` message - see § Mid-conversation system messages above | Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1 - **already available today** (Claude Sonnet 5.5 at launch), no beta header |
| Per-turn reminder (inject, then delete next request) | A turn-scoped `clear_at: "next_user_message"` system message, left in the transcript - see § Mid-conversation system messages above (without the beta: a text block after the `tool_result` blocks, earlier copies kept) | Same models as mid-conversation system messages, behind `mid-conversation-system-clear-at-2026-08-21` |
| `effort` change | A `{"role": "system", "content": [], "output_config": {"effort": ...}}` message - see `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 | Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5 (thinking on only), behind `mid-conversation-output-config-2026-07-01` |
| Dropped thinking blocks (a Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5 block replayed to a model that can't read it - only Claude Fable 5.1 / Claude Mythos 5.1 on the Claude API read Claude Opus 5.5's, and no other model reads Claude Sonnet 5.5's - or a history-editing-check `drop_block`) | None - the API drops the block on that request and the messages cache changes from its position onward; tools and system caches are intact. Blocks the receiving model can read, passed back unchanged, keep the cache intact | - |

| 使缓存失效的顶层变更 | 保留缓存的替代形式 | 可用范围 |
|---|---|---|
| 工具定义（添加/移除） | `tool_addition` / `tool_removal` 块——参见 `shared/tool-use-concepts.md` § Mid-conversation tool changes | Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1、Claude Sonnet 5.5（不含 Claude Sonnet 5），位于 `mid-conversation-tool-changes-2026-07-01` 之后 |
| 系统提示词内容 | `{"role": "system", "content": "..."}` 消息——参见上文 § Mid-conversation system messages | Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1——**现已可用**（Claude Sonnet 5.5 发布即包含），无 beta 头 |
| 每轮提醒（注入后于下一请求删除） | 轮次作用域的 `clear_at: "next_user_message"` system 消息，留在对话记录中——参见上文 § Mid-conversation system messages（无 beta 时：`tool_result` 块之后的 text 块，保留更早副本） | 与对话中途 system 消息相同的模型，位于 `mid-conversation-system-clear-at-2026-08-21` 之后 |
| `effort` 变更 | `{"role": "system", "content": [], "output_config": {"effort": ...}}` 消息——参见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 | Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5（仅限开启思考），位于 `mid-conversation-output-config-2026-07-01` 之后 |
| 被丢弃的思考块（Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5 的思考块被回放给无法读取它的模型——在 Claude API 上只有 Claude Fable 5.1 / Claude Mythos 5.1 能读 Claude Opus 5.5 的思考块，且没有其他模型能读 Claude Sonnet 5.5 的——或触发历史编辑检查的 `drop_block`） | 无——API 会在该请求上丢弃该块，消息缓存自其位置起失效；工具与系统缓存保持完好。接收模型能读取的思考块若原样传回，则缓存保持完好 | - |

Model switch has no escape hatch: caches are model-scoped. Keep the main loop on one model and spawn a subagent for cheaper sub-tasks (see `agent-design.md` § Caching for Agents).

切换模型没有逃生通道：缓存按模型隔离。让主循环固定在一个模型上，为更便宜的子任务派生子智能体（参见 `agent-design.md` § Caching for Agents）。

**Thinking blocks and the messages cache (model-specific).** On Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, Mythos Preview, Opus 4.5 and later (Claude Opus 5.5 included), and Sonnet 4.6 and later, previous-turn thinking blocks are preserved by default, so passing a regular (non-tool-result) user message with thinking enabled leaves the messages cache valid. On earlier Opus and Sonnet models and all Haiku models through Haiku 4.5, that same request strips previously-cached thinking blocks from context, and every message after the first stripped block falls out of cache - in an agent loop this shows up as a `cache_creation_input_tokens` spike on turns where a plain user message follows tool use. (Toggling thinking on/off between requests is a separate, all-models invalidator of the messages cache - see the hierarchy table above. Changing `output_config.effort` behaves the same as changing thinking parameters; setting the model's default effort explicitly is equivalent to omitting it, so pinning the default costs nothing.)

**思考块与消息缓存（因模型而异）。**在 Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1、Mythos Preview、Opus 4.5 及之后（含 Claude Opus 5.5）、以及 Sonnet 4.6 及之后的模型上，先前轮次的思考块默认被保留，因此在开启思考的情况下传入普通（非工具结果）用户消息不会使消息缓存失效。在更早的 Opus 与 Sonnet 模型以及直到 Haiku 4.5 的所有 Haiku 模型上，同样的请求会把先前缓存的思考块从上下文中剥离，第一个被剥离块之后的每条消息都会掉出缓存——在智能体循环中，这表现为"普通用户消息跟在工具使用之后"的那些轮次上出现 `cache_creation_input_tokens` 尖峰。（在请求之间开关 thinking 是另一个针对所有模型的、使消息缓存失效的因素——见上方层级表。更改 `output_config.effort` 与更改 thinking 参数的表现相同；显式设置模型默认 effort 等价于省略它，因此固定默认值没有任何代价。）

---

## 20-block lookback window / 20 块回看窗口

Each breakpoint walks backward **at most 20 positions** to find a prior cache entry. On the Claude API a run of consecutive `tool_use` blocks counts as one position, and so does a run of consecutive `tool_result` blocks, so a turn with many *parallel* tool calls doesn't push the previous request's entry out of the window; a turn that adds more than 20 positions of other content (long sequential tool loops, many text/image blocks) still can - the next request's breakpoint won't find the previous cache and silently misses.

每个断点会向后回看**最多 20 个位置**以寻找先前的缓存条目。在 Claude API 上，连续的 `tool_use` 块段算一个位置，连续的 `tool_result` 块段也算一个位置，因此包含许多*并行*工具调用的轮次不会把上一个请求的条目推出窗口；但追加超过 20 个位置其他内容的轮次（很长的顺序工具循环、大量文本/图片块）仍可能如此——下一个请求的断点将找不到先前的缓存并静默失效。

Fix: place an intermediate breakpoint every ~15 positions in long turns, or put the marker on a block that's within 20 positions of the previous turn's last cached block.

修复：在长轮次中每隔约 15 个位置放一个中间断点，或者把标记放在距上一轮最后一个已缓存块 20 个位置以内的块上。

---

## Concurrent-request timing / 并发请求时序

A cache entry becomes readable only after the first response **begins streaming**. N parallel requests with identical prefixes all pay full price - none can read what the others are still writing.

缓存条目只有在第一个响应**开始流式输出**之后才可读。N 个前缀相同的并行请求全都支付全价——谁也读不到其他请求正在写的内容。

For fan-out patterns: send 1 request, await the first streamed token (not the full response), then fire the remaining N-1. They'll read the cache the first one just wrote.

对于扇出（fan-out）模式：先发 1 个请求，等它的第一个流式 token（而不是完整响应）返回，再发出其余 N-1 个。它们会读到第一个请求刚写入的缓存。

The same arithmetic shapes multi-agent designs: N parallel workers each assembling a slightly different prompt over the same context write N separate cache entries and read none of each other's. When input cost dominates, fewer lanes over a byte-identical shared prefix - or one worker making N sequential passes - turn those writes into reads.

同样的算术也塑造着多智能体设计：N 个并行 worker 若各自基于同一上下文组装略有不同的提示词，会写入 N 个互不相通的缓存条目，且彼此都读不到。当输入成本占主导时，让更少的通道运行在字节完全相同的共享前缀上——或者由一个 worker 做 N 次顺序处理——就能把这些写入变成读取。

## Pre-warming the cache / 预热缓存

To eliminate the cache-miss latency on the *first* real request, send a **`max_tokens: 0`** request at startup (or on an interval). The API runs prefill - writing the cache at your `cache_control` breakpoint - and returns immediately with `content: []`, `stop_reason: "max_tokens"`, and a populated `usage` block (zero output tokens billed; normal cache-write charge on `cache_creation_input_tokens`).

为消除*第一个*真实请求上的缓存未命中延迟，在启动时（或按间隔）发送一个 **`max_tokens: 0`** 请求。API 会执行预填充（prefill）——在你的 `cache_control` 断点处写入缓存——并立即返回 `content: []`、`stop_reason: "max_tokens"` 以及填充好的 `usage` 块（不计费输出 token；`cache_creation_input_tokens` 上照常收取缓存写入费用）。

**When to pre-warm** - pre-warming trades a cache-write charge *now* for lower TTFT on the *next* real request. It's worth it when all three hold: (a) first-request latency is user-visible (chat/voice/interactive - not background jobs), (b) the shared prefix is large enough that a cold write is noticeably slow, and (c) there's a moment *before* traffic to fire it - app startup, worker boot, post-deploy, start of a scheduled window.

**何时预热**——预热是用*现在*的一次缓存写入费用，换取*下一个*真实请求更低的 TTFT。当以下三点同时成立时才值得：(a) 首请求延迟对用户可见（聊天/语音/交互——而非后台任务），(b) 共享前缀足够大以致冷写入明显偏慢，(c) 在流量到来*之前*存在一个可以触发预热的时机——应用启动、worker 启动、部署完成后、计划窗口开始时。

| Skip pre-warming when... | Because |
|---|---|
| Traffic is continuous (requests <= TTL apart) | The first real request warms the cache and every subsequent one hits it; a separate warm call is a pure extra write |
| The prefix is small or below the cacheable minimum | The cold-write penalty is negligible |
| The prefix varies per request/user | Nothing shared to pre-warm |
| You'd pre-warm many distinct prefixes speculatively | Each is a ~1.25× write; cost can exceed the latency you save |

| 何时跳过预热 | 原因 |
|---|---|
| 流量连续（请求间隔 <= TTL） | 第一个真实请求就会把缓存焐热，后续每个请求都命中；单独的预热调用是纯粹的额外写入 |
| 前缀很小或低于可缓存最小值 | 冷写入的代价可以忽略 |
| 前缀因请求/用户而异 | 没有共享内容可预热 |
| 你想投机性地预热许多个不同前缀 | 每个都是一次 ~1.25× 写入；成本可能超过你省下的延迟 |

**Scheduled re-warms:** only needed when traffic has gaps longer than the TTL. If real requests arrive more often than every 5 minutes, they keep the cache warm on their own - don't add an interval re-warm. For bursty traffic with long idle gaps, either re-warm just under the TTL or switch to `ttl: "1h"` and re-warm less often.

**定时重新保温：**只在流量的间隙长于 TTL 时才需要。如果真实请求的到达频率高于每 5 分钟一次，它们自己就能保持缓存温热——不要额外加定时重保温。对于带长空闲间隙的突发流量，要么在略小于 TTL 的时点重保温，要么改用 `ttl: "1h"` 并降低重保温频率。

```python
client.messages.create(
    model="claude-opus-5-5",
    max_tokens=0,
    # Example values - send the same thinking and effort settings as your real traffic (see below)
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    system=[{
        "type": "text",
        "text": SYSTEM_PROMPT,
        "cache_control": {"type": "ephemeral"},
    }],
    messages=[{"role": "user", "content": "warmup"}],
)
```

**Breakpoint placement:** put `cache_control` on the **last block shared with the real request** (the system prompt or tool definitions) - **not** on the placeholder user message, and **not** via top-level automatic caching (which would key the cache to the placeholder). The placeholder can be any non-whitespace string; it's read during prefill but never answered.

**断点放置：**把 `cache_control` 放在**与真实请求共享的最后一个块**（系统提示词或工具定义）上——**不要**放在占位用户消息上，也**不要**用顶层自动缓存（那会把缓存键关联到占位消息上）。占位内容可以是任何非空白字符串；它在预填充期间被读取，但永远不会被回答。

**Match the real traffic's thinking and effort settings.** Both are rendered into the prompt (see § Invalidation hierarchy), so a pre-warm with different settings can write a cache entry your real traffic never hits. For adaptive-thinking traffic, send the same `thinking` and `effort` values in the pre-warm. Traffic that uses extended thinking (`thinking.type: "enabled"`, accepted only on Claude 4.6 and earlier models and on Claude Mythos Preview) can't be matched, because `max_tokens: 0` rejects that setting (see Rejected combinations below). Whether a pre-warm still helps that traffic depends on where the model renders the thinking configuration, which the docs don't state per model - check `cache_read_input_tokens` on the first real request.

**匹配真实流量的 thinking 与 effort 设置。**两者都会渲染进提示词（参见 § Invalidation hierarchy），因此设置不一致的预热可能写出一个真实流量永远命中不了的缓存条目。对于自适应思考流量，请在预热中发送相同的 `thinking` 与 `effort` 值。使用扩展思考的流量（`thinking.type: "enabled"`，仅在 Claude 4.6 及更早模型与 Claude Mythos Preview 上被接受）无法匹配，因为 `max_tokens: 0` 拒绝该设置（见下文 Rejected combinations）。预热是否仍对这种流量有帮助，取决于模型把思考配置渲染在哪里——文档没有按模型说明这一点——请在第一个真实请求上检查 `cache_read_input_tokens`。

**Rejected combinations:** `max_tokens: 0` is an `invalid_request_error` with `stream: true`, `thinking.type: "enabled"`, `output_config.format`, `tool_choice` of `{"type":"tool"}` or `{"type":"any"}`, or inside a Message Batches request.

**被拒绝的组合：**当出现 `stream: true`、`thinking.type: "enabled"`、`output_config.format`、`tool_choice` 为 `{"type":"tool"}` 或 `{"type":"any"}`、或位于 Message Batches 请求内时，`max_tokens: 0` 会报 `invalid_request_error`。

**TTL still applies** - re-warm at least every 5 minutes for the default cache, or use the 1-hour TTL. This replaces the older `max_tokens: 1` workaround (no single-token reply to discard, no output tokens billed, intent is unambiguous).

**TTL 仍然适用**——默认缓存至少每 5 分钟重保温一次，或使用 1 小时 TTL。这取代了旧的 `max_tokens: 1` 变通办法（没有需要丢弃的单 token 回复、不计费输出 token、意图明确无歧义）。
