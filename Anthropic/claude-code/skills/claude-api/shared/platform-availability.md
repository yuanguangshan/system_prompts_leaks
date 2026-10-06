<!-- BILINGUAL-EN-ZH -->
# Platform Availability / 平台可用性

Which features work on which provider platform. **This table is the single source of truth in this skill** - per-feature sections elsewhere point here instead of restating availability. When writing code for a third-party platform (Bedrock, Vertex, Foundry) or Claude Platform on AWS, check this table first; a feature not supported there means use the first-party Claude API surface or a different approach.

哪些功能在哪个提供商平台上可用。**本表是本技能中唯一的权威来源**——其他按功能展开的小节直接指向这里，而不是重复说明可用性。为第三方平台（Bedrock、Vertex、Foundry）或 Claude Platform on AWS 编写代码时，先查此表；某功能在该平台不受支持即意味着应改用第一方 Claude API 接口或其他方案。

Columns: **1P** = first-party Claude API, **P-AWS** = Claude Platform on AWS (Anthropic-operated, same-day parity), **Bedrock** = Amazon Bedrock, **Vertex** = Google Cloud Vertex AI, **Foundry** = Microsoft Foundry. Yes = GA, beta = beta, No = not supported, unconfirmed = not verified either way when this was written.

各列含义：**1P** = 第一方 Claude API，**P-AWS** = Claude Platform on AWS（由 Anthropic 运营、同日对齐），**Bedrock** = Amazon Bedrock，**Vertex** = Google Cloud Vertex AI，**Foundry** = Microsoft Foundry。Yes = 正式发布（GA），beta = 测试版，No = 不支持，unconfirmed = 截至撰写时两种情况均未得到证实。

| Feature | 1P | P-AWS | Bedrock | Vertex | Foundry | Notes |
|---|---|---|---|---|---|---|
| Messages, streaming, tool use | Yes | Yes | Yes | Yes | Yes | Core API |
| PDF input | Yes | Yes | Yes | Yes | Yes | |
| Structured outputs / strict tool use | Yes | Yes | Yes | Yes | Yes | |
| Adaptive thinking / effort | Yes | Yes | Yes | Yes | Yes | |
| Extended thinking | Yes | Yes | Yes | Yes | Yes | |
| Prompt caching (5m, 1h) | Yes | Yes | Yes | Yes | Yes | |
| Automatic prompt caching | Yes | Yes | Yes | Yes | Yes | The legacy Bedrock integration (Opus 4.6 and earlier) rejects top-level `cache_control` with a 400 - explicit breakpoints only there |
| Token counting | Yes | Yes | Yes | Yes | Yes | |
| Citations | Yes | Yes | Yes | Yes | Yes | |
| Search results content blocks | Yes | Yes | Yes | Yes | Yes | |
| Fine-grained tool streaming | Yes | Yes | Yes | Yes | Yes | Bedrock: `eager_input_streaming` on the newer serving stack only (Opus 4.7/4.8/5, Fable 5, Sonnet 4.6/5); older deployments (Opus 4.5/4.6, Sonnet 4.0/4.5, Haiku 4.5) 400 on the field |
| Compaction | beta | beta | beta | beta | beta | |
| Context editing | beta | beta | beta | beta | beta | |
| Context windows (1M) | Yes | Yes | Yes | Yes | Yes | |
| `inference_geo` (data residency) | Yes | Yes | No | No | No | |
| **Server-side tools** | | | | | | |
| &nbsp;&nbsp;Web search | Yes | Yes | No | Yes | Yes | Vertex: basic `web_search_20250305` only (no `_20260209` dynamic filtering). Foundry Hosted on Azure: basic `web_search_20250305` only |
| &nbsp;&nbsp;Web fetch | Yes | Yes | No | No | Yes | Foundry Hosted on Azure: basic `web_fetch_20250910` only |
| &nbsp;&nbsp;Code execution | Yes | Yes | No | No | Yes | Foundry: Hosted on Anthropic deployments only - Hosted on Azure returns a 400 |
| &nbsp;&nbsp;Tool search | Yes | Yes | Yes | Yes | Yes | Bedrock: InvokeModel API only, not Converse |
| &nbsp;&nbsp;Advisor tool | beta | beta | No | No | No | |
| **Client-implemented tools** | | | | | | |
| &nbsp;&nbsp;Bash, text editor, memory | Yes | Yes | Yes | Yes | Yes | |
| &nbsp;&nbsp;Computer use | beta | beta | beta | beta | beta | `computer_20251124` and older versions: beta on all five platforms. Claude Opus 5.5 accepts only `computer_toolset_20260801` (GA, no beta header) on the Claude API and Google Cloud, and still accepts `computer_20251124` on Amazon Bedrock (`shared/model-migration.md` -> Migrating to Claude Opus 5.5, breaking change 4). Claude Sonnet 5.5 accepts only the toolset on the Claude API and Google Cloud, but still accepts `computer_20251124` on Amazon Bedrock, and rejects `computer_20250124` everywhere (`shared/model-migration.md` -> Migrating to Claude Sonnet 5.5, breaking change 4) |
| **Agentic / orchestration** | | | | | | |
| &nbsp;&nbsp;Agent Skills (Messages API) | Yes | Yes | No | No | beta | Foundry: Hosted on Anthropic deployments only - Hosted on Azure returns a 400 |
| &nbsp;&nbsp;Programmatic tool calling | Yes | Yes | No | No | Yes | Foundry: Hosted on Anthropic deployments only - Hosted on Azure returns a 400 |
| &nbsp;&nbsp;MCP connector | beta | beta | No | No | beta | |
| &nbsp;&nbsp;Managed Agents | beta | beta | No | No | No | Foundry: No (inferred; not in Foundry docs either way) |
| &nbsp;&nbsp;Self-hosted sandboxes | beta | beta | No | No | No | P-AWS: worker authenticates with IAM/SigV4 or an AWS-Console API key + `AnthropicSelfHostedEnvironmentAccess` (Console environment keys don't work there); sessions on self-hosted environments cannot attach memory stores; `GET /v1/environments/{id}/work` list endpoint not supported, other work endpoints OK |
| **API endpoints** | | | | | | |
| &nbsp;&nbsp;Message Batches | Yes | Yes | No | No | No | |
| &nbsp;&nbsp;Files API | Yes | Yes | No | No | beta | Foundry: Hosted on Anthropic deployments only - Hosted on Azure returns a 400 |
| &nbsp;&nbsp;Models API | Yes | Yes | No | No | No | |
| **Other** | | | | | | |
| &nbsp;&nbsp;Mid-conversation system messages | Yes | Yes | Yes | Yes | No | Claude Opus 5, Claude Opus 5.5, Claude Opus 4.8, Claude Fable 5, Claude Fable 5.1, Claude Mythos 5, Claude Mythos 5.1, Claude Sonnet 5.5; not Claude Sonnet 5. Bedrock: InvokeModel passthrough, not ARN-versioned models |
| &nbsp;&nbsp;Mid-conversation tool changes | beta | beta | beta | beta | No | Same models as mid-conversation system messages; beta `mid-conversation-tool-changes-2026-07-01` |
| &nbsp;&nbsp;Turn-scoped (`clear_at`) system messages | beta | beta | beta | beta | No | Same models as mid-conversation system messages; beta `mid-conversation-system-clear-at-2026-08-21` (on Bedrock/Vertex pass the value as a beta) |
| &nbsp;&nbsp;Per-message `effort` (system message `output_config`) | beta | unconfirmed | unconfirmed | beta | unconfirmed | Claude Fable 5.1, Claude Mythos 5.1, Claude Opus 5, Claude Opus 5.5, Claude Sonnet 5.5 (thinking on only - a 400 with `between_tools`); beta `mid-conversation-output-config-2026-07-01`; on the Claude API and Google Cloud, open to any organization that sends the header (Claude Platform on AWS/Bedrock/Foundry unconfirmed; Claude Opus 5 excluded on Bedrock) |
| &nbsp;&nbsp;`thinking.display: "updates"` | beta | beta | beta | beta | beta | Claude Fable 5.1, Claude Mythos 5.1, Claude Fable 5, Claude Opus 5.5, Claude Sonnet 5.5 (with adaptive thinking); beta `thinking-display-updates-2026-08-18` (pass the beta value per platform); without it `"updates"` is rejected as an unknown `display` value |
| &nbsp;&nbsp;Thinking block-binding controls | beta | beta | beta | beta | unconfirmed | `thinking.block_binding` + `input_transformations`; beta `thinking-binding-controls-2026-08-01` (the same beta name on the Claude API, Claude Platform on AWS, Bedrock, and Vertex - Bedrock: the `anthropic_beta` body field, Vertex: the `anthropic-beta` HTTP header); Foundry unconfirmed; wherever the header is rejected, use strip-and-retry; the history-editing enforcement itself follows the account-age rule in `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 |
| &nbsp;&nbsp;Server-side `fallbacks` | beta | beta | No | No | No | `"default"` -> beta `server-side-fallback-2026-07-01`; array form -> beta `server-side-fallback-2026-06-01` |
| &nbsp;&nbsp;Fast mode | beta | No | No | No | No | Research preview, beta `fast-mode-2026-02-01`, first-party API only (Claude Opus 5 / Opus 4.8 at $10 / $50; Claude Opus 5.5 at $8 / $40) |
| &nbsp;&nbsp;Cache diagnostics | beta | No | No | No | No | First-party API only |
| &nbsp;&nbsp;Task budgets | beta | beta | No | No | No | Beta header `task-budgets-2026-03-13`; 3P availability not documented - assume unsupported |

| 功能 | 1P | P-AWS | Bedrock | Vertex | Foundry | 备注 |
|---|---|---|---|---|---|---|
| 消息、流式输出、工具使用 | 是 | 是 | 是 | 是 | 是 | 核心 API |
| PDF 输入 | 是 | 是 | 是 | 是 | 是 | |
| 结构化输出 / 严格工具使用 | 是 | 是 | 是 | 是 | 是 | |
| 自适应思考 / effort | 是 | 是 | 是 | 是 | 是 | |
| 扩展思考 | 是 | 是 | 是 | 是 | 是 | |
| 提示词缓存（5 分钟、1 小时） | 是 | 是 | 是 | 是 | 是 | |
| 自动提示词缓存 | 是 | 是 | 是 | 是 | 是 | 旧版 Bedrock 集成（Opus 4.6 及更早）会以 400 拒绝顶层 `cache_control`——该平台只能使用显式断点 |
| Token 计数 | 是 | 是 | 是 | 是 | 是 | |
| 引用（Citations） | 是 | 是 | 是 | 是 | 是 | |
| 搜索结果内容块 | 是 | 是 | 是 | 是 | 是 | |
| 细粒度工具流式输出 | 是 | 是 | 是 | 是 | 是 | Bedrock：仅较新服务栈支持 `eager_input_streaming`（Opus 4.7/4.8/5、Fable 5、Sonnet 4.6/5）；较旧部署（Opus 4.5/4.6、Sonnet 4.0/4.5、Haiku 4.5）对该字段返回 400 |
| 压缩（Compaction） | beta | beta | beta | beta | beta | |
| 上下文编辑 | beta | beta | beta | beta | beta | |
| 上下文窗口（1M） | 是 | 是 | 是 | 是 | 是 | |
| `inference_geo`（数据驻留） | 是 | 是 | 否 | 否 | 否 | |
| **服务器端工具** | | | | | | |
| &nbsp;&nbsp;网页搜索 | 是 | 是 | 否 | 是 | 是 | Vertex：仅支持基础的 `web_search_20250305`（不支持 `_20260209` 动态过滤）。Foundry Hosted on Azure：仅支持基础的 `web_search_20250305` |
| &nbsp;&nbsp;网页抓取 | 是 | 是 | 否 | 否 | 是 | Foundry Hosted on Azure：仅支持基础的 `web_fetch_20250910` |
| &nbsp;&nbsp;代码执行 | 是 | 是 | 否 | 否 | 是 | Foundry：仅 Hosted on Anthropic 部署可用——Hosted on Azure 返回 400 |
| &nbsp;&nbsp;工具搜索 | 是 | 是 | 是 | 是 | 是 | Bedrock：仅 InvokeModel API，不支持 Converse |
| &nbsp;&nbsp;Advisor 工具 | beta | beta | 否 | 否 | 否 | |
| **客户端实现的工具** | | | | | | |
| &nbsp;&nbsp;Bash、文本编辑器、记忆 | 是 | 是 | 是 | 是 | 是 | |
| &nbsp;&nbsp;计算机使用 | beta | beta | beta | beta | beta | `computer_20251124` 及更早版本：在全部五个平台上均为 beta。Claude Opus 5.5 在 Claude API 和 Google Cloud 上只接受 `computer_toolset_20260801`（GA，无需 beta 头），在 Amazon Bedrock 上仍接受 `computer_20251124`（`shared/model-migration.md` -> Migrating to Claude Opus 5.5，破坏性变更 4）。Claude Sonnet 5.5 在 Claude API 和 Google Cloud 上只接受该工具集，在 Amazon Bedrock 上仍接受 `computer_20251124`，且在所有平台都拒绝 `computer_20250124`（`shared/model-migration.md` -> Migrating to Claude Sonnet 5.5，破坏性变更 4） |
| **智能体 / 编排** | | | | | | |
| &nbsp;&nbsp;Agent Skills（Messages API） | 是 | 是 | 否 | 否 | beta | Foundry：仅 Hosted on Anthropic 部署可用——Hosted on Azure 返回 400 |
| &nbsp;&nbsp;程序化工具调用 | 是 | 是 | 否 | 否 | 是 | Foundry：仅 Hosted on Anthropic 部署可用——Hosted on Azure 返回 400 |
| &nbsp;&nbsp;MCP 连接器 | beta | beta | 否 | 否 | beta | |
| &nbsp;&nbsp;Managed Agents | beta | beta | 否 | 否 | 否 | Foundry：否（推断得出；Foundry 文档两种说法均无记载） |
| &nbsp;&nbsp;自托管沙箱 | beta | beta | 否 | 否 | 否 | P-AWS：worker 使用 IAM/SigV4 或 AWS 控制台 API 密钥 + `AnthropicSelfHostedEnvironmentAccess` 进行认证（控制台环境密钥在该平台不可用）；自托管环境上的会话无法挂接记忆存储；`GET /v1/environments/{id}/work` 列表端点不受支持，其余 work 端点正常 |
| **API 端点** | | | | | | |
| &nbsp;&nbsp;Message Batches | 是 | 是 | 否 | 否 | 否 | |
| &nbsp;&nbsp;Files API | 是 | 是 | 否 | 否 | beta | Foundry：仅 Hosted on Anthropic 部署可用——Hosted on Azure 返回 400 |
| &nbsp;&nbsp;Models API | 是 | 是 | 否 | 否 | 否 | |
| **其他** | | | | | | |
| &nbsp;&nbsp;对话中途系统消息 | 是 | 是 | 是 | 是 | 否 | Claude Opus 5、Claude Opus 5.5、Claude Opus 4.8、Claude Fable 5、Claude Fable 5.1、Claude Mythos 5、Claude Mythos 5.1、Claude Sonnet 5.5；不支持 Claude Sonnet 5。Bedrock：以 InvokeModel 透传方式提供，不支持 ARN 版本化模型 |
| &nbsp;&nbsp;对话中途工具变更 | beta | beta | beta | beta | 否 | 与对话中途系统消息相同的模型；beta `mid-conversation-tool-changes-2026-07-01` |
| &nbsp;&nbsp;轮次范围（`clear_at`）系统消息 | beta | beta | beta | beta | 否 | 与对话中途系统消息相同的模型；beta `mid-conversation-system-clear-at-2026-08-21`（在 Bedrock/Vertex 上将该值作为 beta 传入） |
| &nbsp;&nbsp;按消息 `effort`（系统消息 `output_config`） | beta | 未确认 | 未确认 | beta | 未确认 | Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5、Claude Opus 5.5、Claude Sonnet 5.5（仅限思考开启时——`between_tools` 返回 400）；beta `mid-conversation-output-config-2026-07-01`；在 Claude API 和 Google Cloud 上，对发送该头的任何组织开放（Claude Platform on AWS/Bedrock/Foundry 未确认；Bedrock 上不含 Claude Opus 5） |
| &nbsp;&nbsp;`thinking.display: "updates"` | beta | beta | beta | beta | beta | Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Opus 5.5、Claude Sonnet 5.5（需配合自适应思考）；beta `thinking-display-updates-2026-08-18`（按平台传入 beta 值）；若无该头，`"updates"` 会作为未知 `display` 值被拒绝 |
| &nbsp;&nbsp;思考块绑定控制 | beta | beta | beta | beta | 未确认 | `thinking.block_binding` + `input_transformations`；beta `thinking-binding-controls-2026-08-01`（在 Claude API、Claude Platform on AWS、Bedrock 和 Vertex 上为同一 beta 名称——Bedrock：`anthropic_beta` 请求体字段，Vertex：`anthropic-beta` HTTP 头）；Foundry 未确认；凡该头被拒绝之处，采用剥离重试（strip-and-retry）；历史编辑强制机制本身遵循 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5 中的账户年龄规则 |
| &nbsp;&nbsp;服务器端 `fallbacks` | beta | beta | 否 | 否 | 否 | `"default"` 形式 -> beta `server-side-fallback-2026-07-01`；数组形式 -> beta `server-side-fallback-2026-06-01` |
| &nbsp;&nbsp;快速模式 | beta | 否 | 否 | 否 | 否 | 研究预览，beta `fast-mode-2026-02-01`，仅第一方 API（Claude Opus 5 / Opus 4.8 为 $10 / $50；Claude Opus 5.5 为 $8 / $40） |
| &nbsp;&nbsp;缓存诊断 | beta | 否 | 否 | 否 | 否 | 仅第一方 API |
| &nbsp;&nbsp;任务预算 | beta | beta | 否 | 否 | 否 | Beta 头 `task-budgets-2026-03-13`；第三方可用性未记载——应视为不支持 |

【评论】用单一表格集中维护跨平台功能矩阵、并让其他小节引用而非复述，是一种减少文档漂移（drift）的单源真相（single source of truth）写法；表中"unconfirmed"条目也被显式标注而非留空。
