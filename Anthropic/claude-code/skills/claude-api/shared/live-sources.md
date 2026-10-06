<!-- BILINGUAL-EN-ZH -->
# Live Documentation Sources / 活文档来源

This file contains WebFetch URLs for fetching current information from platform.claude.com and Agent SDK repositories. Use these when users need the latest data that may have changed since the cached content was last updated.

本文件包含用于从 platform.claude.com 与 Agent SDK 仓库获取最新信息的 WebFetch URL。当用户所需的数据自缓存内容上次更新后可能已发生变化时，请使用这些 URL。

## When to Use WebFetch / 何时使用 WebFetch

- User explicitly asks for "latest" or "current" information
  用户明确要求"最新"或"当前"信息
- Cached data seems incorrect
  缓存数据看起来不正确
- User asks about features not covered in cached content
  用户询问缓存内容未覆盖的功能
- User needs specific API details or examples
  用户需要具体的 API 细节或示例

## Claude API Documentation URLs / Claude API 文档 URL

### Models & Pricing / 模型与定价

| Topic           | URL                                                                          | Extraction Prompt                                                               |
| --------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Models Overview | `https://platform.claude.com/docs/en/about-claude/models/overview.md`        | "Extract current model IDs, context windows, and pricing for all Claude models" |
| Migration Guide | `https://platform.claude.com/docs/en/about-claude/models/migration-guide.md` | "Extract breaking changes, deprecated parameters, and per-model migration steps when moving to a newer Claude model" |
| Introducing Claude Fable 5 | `https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5.md` | "Extract capabilities, API changes, and availability stages for Claude Fable 5 and Claude Mythos 5" |
| Pricing         | `https://platform.claude.com/docs/en/about-claude/pricing.md`                | "Extract current pricing per million tokens for input and output"               |
| Cost Optimization | `https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md` | "Extract measured cost levers, cache and batch savings, effort and model cost-per-task comparisons, budget controls, and multi-model guidance" |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 模型总览 | `https://platform.claude.com/docs/en/about-claude/models/overview.md` | "提取所有 Claude 模型当前的模型 ID、上下文窗口与定价" |
| 迁移指南 | `https://platform.claude.com/docs/en/about-claude/models/migration-guide.md` | "提取迁移到较新 Claude 模型时的破坏性变更、弃用参数与逐模型迁移步骤" |
| Claude Fable 5 介绍 | `https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5.md` | "提取 Claude Fable 5 与 Claude Mythos 5 的能力、API 变更与可用性阶段" |
| 定价 | `https://platform.claude.com/docs/en/about-claude/pricing.md` | "提取当前输入与输出每百万 token 的定价" |
| 成本优化 | `https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.md` | "提取实测的成本杠杆、缓存与批处理节省、effort 与模型的每任务成本对比、预算控制以及多模型指引" |

### Core Features / 核心功能

| Topic             | URL                                                                          | Extraction Prompt                                                                      |
| ----------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Extended Thinking | `https://platform.claude.com/docs/en/build-with-claude/extended-thinking.md` | "Extract extended thinking parameters, budget_tokens requirements, and usage examples" |
| Adaptive Thinking | `https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking.md` | "Extract adaptive thinking setup, effort levels, and Claude Opus 5.5 usage examples"         |
| Effort Parameter  | `https://platform.claude.com/docs/en/build-with-claude/effort.md`            | "Extract effort levels, cost-quality tradeoffs, and interaction with thinking"        |
| Tool Use          | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview.md`  | "Extract tool definition schema, tool_choice options, and handling tool results"       |
| Streaming         | `https://platform.claude.com/docs/en/build-with-claude/streaming.md`         | "Extract streaming event types, SDK examples, and best practices"                      |
| Prompt Caching    | `https://platform.claude.com/docs/en/build-with-claude/prompt-caching.md`    | "Extract cache_control usage, pricing benefits, and implementation examples"           |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 扩展思考 | `https://platform.claude.com/docs/en/build-with-claude/extended-thinking.md` | "提取扩展思考参数、budget_tokens 要求与使用示例" |
| 自适应思考 | `https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking.md` | "提取自适应思考的设置、effort 级别与 Claude Opus 5.5 使用示例" |
| Effort 参数 | `https://platform.claude.com/docs/en/build-with-claude/effort.md` | "提取 effort 级别、成本-质量权衡及其与思考功能的交互" |
| 工具使用 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview.md` | "提取工具定义 schema、tool_choice 选项与工具结果处理" |
| 流式传输 | `https://platform.claude.com/docs/en/build-with-claude/streaming.md` | "提取流式事件类型、SDK 示例与最佳实践" |
| 提示词缓存 | `https://platform.claude.com/docs/en/build-with-claude/prompt-caching.md` | "提取 cache_control 用法、定价收益与实现示例" |

### Media & Files / 媒体与文件

| Topic       | URL                                                                    | Extraction Prompt                                                 |
| ----------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Vision      | `https://platform.claude.com/docs/en/build-with-claude/vision.md`      | "Extract supported image formats, size limits, and code examples" |
| PDF Support | `https://platform.claude.com/docs/en/build-with-claude/pdf-support.md` | "Extract PDF handling capabilities, limits, and examples"         |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 视觉 | `https://platform.claude.com/docs/en/build-with-claude/vision.md` | "提取支持的图片格式、大小限制与代码示例" |
| PDF 支持 | `https://platform.claude.com/docs/en/build-with-claude/pdf-support.md` | "提取 PDF 处理能力、限制与示例" |

### API Operations / API 操作

| Topic            | URL                                                                         | Extraction Prompt                                                                                       |
| ---------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Batch Processing | `https://platform.claude.com/docs/en/build-with-claude/batch-processing.md` | "Extract batch API endpoints, request format, and polling for results"                                  |
| Files API        | `https://platform.claude.com/docs/en/build-with-claude/files.md`            | "Extract file upload, download, referencing in messages, supported types, and the migration steps from files-api-2025-04-14" |
| Token Counting   | `https://platform.claude.com/docs/en/build-with-claude/token-counting.md`   | "Extract token counting API usage and examples"                                                         |
| Rate Limits      | `https://platform.claude.com/docs/en/api/rate-limits.md`                    | "Extract current rate limits by tier and model"                                                         |
| Usage and Cost Admin API | `https://platform.claude.com/docs/en/manage-claude/usage-cost-api.md` | "Extract the usage_report and cost_report endpoints, Admin API key requirements, filter and group_by dimensions, token fields, and granularity limits" |
| Errors           | `https://platform.claude.com/docs/en/api/errors.md`                         | "Extract HTTP error codes, meanings, and retry guidance"                                                |
| Amazon Bedrock   | `https://platform.claude.com/docs/en/build-with-claude/claude-on-amazon-bedrock.md` | "Extract the AnthropicBedrockMantle client per language, `anthropic.`-prefixed model IDs, auth paths, feature availability, and regions" |
| Claude Platform on AWS | `https://platform.claude.com/docs/en/build-with-claude/claude-platform-on-aws.md` | "Extract the AnthropicAWS client per language, SigV4 auth, credential precedence, short-term API keys, workspace_id, and region requirements" |
| Claude Platform on AWS - IAM actions | `https://platform.claude.com/docs/en/api/claude-platform-on-aws-iam-actions.md` | "Extract the IAM action names, resource ARNs, and policy examples required for each API capability" |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 批处理 | `https://platform.claude.com/docs/en/build-with-claude/batch-processing.md` | "提取批处理 API 端点、请求格式与结果轮询" |
| Files API | `https://platform.claude.com/docs/en/build-with-claude/files.md` | "提取文件上传、下载、在消息中引用、支持的类型，以及从 files-api-2025-04-14 迁移的步骤" |
| Token 计数 | `https://platform.claude.com/docs/en/build-with-claude/token-counting.md` | "提取 token 计数 API 的用法与示例" |
| 速率限制 | `https://platform.claude.com/docs/en/api/rate-limits.md` | "提取按层级与模型划分的当前速率限制" |
| 用量与成本 Admin API | `https://platform.claude.com/docs/en/manage-claude/usage-cost-api.md` | "提取 usage_report 与 cost_report 端点、Admin API 密钥要求、filter 与 group_by 维度、token 字段及粒度限制" |
| 错误 | `https://platform.claude.com/docs/en/api/errors.md` | "提取 HTTP 错误码、含义与重试指引" |
| Amazon Bedrock | `https://platform.claude.com/docs/en/build-with-claude/claude-on-amazon-bedrock.md` | "提取各语言的 AnthropicBedrockMantle 客户端、`anthropic.` 前缀的模型 ID、认证路径、功能可用性与区域" |
| AWS 上的 Claude 平台 | `https://platform.claude.com/docs/en/build-with-claude/claude-platform-on-aws.md` | "提取各语言的 AnthropicAWS 客户端、SigV4 认证、凭证优先级、短期 API 密钥、workspace_id 与区域要求" |
| AWS 上的 Claude 平台 - IAM 操作 | `https://platform.claude.com/docs/en/api/claude-platform-on-aws-iam-actions.md` | "提取每个 API 能力所需的 IAM 操作名称、资源 ARN 与策略示例" |

### Admin API (Organization Management) / Admin API（组织管理）

| Topic                | URL                                                                     | Extraction Prompt                                                                     |
| -------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Admin API Guide      | `https://platform.claude.com/docs/en/manage-claude/admin-api.md`        | "Extract Admin API authentication, SDK/CLI usage, and member/invite/key management"   |
| Admin API Reference  | `https://platform.claude.com/docs/en/api/admin.md`                      | "Extract endpoint parameters, responses, and pagination for the Admin API"            |
| Workspaces           | `https://platform.claude.com/docs/en/manage-claude/workspaces.md`       | "Extract workspace create/list/archive and member management via API"                  |
| Rate Limits API      | `https://platform.claude.com/docs/en/manage-claude/rate-limits-api.md`  | "Extract org and workspace rate limit report endpoints and filters"                    |
| WIF Admin            | `https://platform.claude.com/docs/en/manage-claude/wif-admin-api.md`    | "Extract service account, federation issuer, and federation rule management"           |
| Usage & Cost Reports | `https://platform.claude.com/docs/en/manage-claude/usage-cost-api.md`   | "Extract usage and cost report endpoints (curl-only, not in the SDKs)"                 |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| Admin API 指南 | `https://platform.claude.com/docs/en/manage-claude/admin-api.md` | "提取 Admin API 认证、SDK/CLI 用法与成员/邀请/密钥管理" |
| Admin API 参考 | `https://platform.claude.com/docs/en/api/admin.md` | "提取 Admin API 的端点参数、响应与分页" |
| 工作区 | `https://platform.claude.com/docs/en/manage-claude/workspaces.md` | "提取通过 API 进行的工作区创建/列表/归档与成员管理" |
| 速率限制 API | `https://platform.claude.com/docs/en/manage-claude/rate-limits-api.md` | "提取组织与工作区速率限制报告端点及过滤器" |
| WIF 管理 | `https://platform.claude.com/docs/en/manage-claude/wif-admin-api.md` | "提取服务账号、联合身份签发方与联合规则管理" |
| 用量与成本报告 | `https://platform.claude.com/docs/en/manage-claude/usage-cost-api.md` | "提取用量与成本报告端点（仅 curl，SDK 中不提供）" |

### Tools / 工具

| Topic          | URL                                                                                    | Extraction Prompt                                                                        |
| -------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Code Execution | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool.md` | "Extract code execution tool setup, file upload, container reuse, and response handling" |
| Computer Use   | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool.md`   | "Extract the computer_toolset_20260801 setup (configs, member tools, batch actions, toolset_name on results), the Compatibility matrix, and the migration steps from computer_20251124"             |
| Bash Tool      | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/bash-tool.md`           | "Extract bash tool schema, reference implementation, and security considerations"        |
| Text Editor    | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/text-editor-tool.md`    | "Extract text editor tool commands, schema, and reference implementation"                |
| Memory Tool    | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`         | "Extract memory tool commands, directory structure, and implementation patterns"         |
| Tool Search    | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool.md`    | "Extract tool search setup, when to use, and cache interaction"                          |
| Programmatic Tool Calling | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling.md` | "Extract PTC setup, script execution model, and tool invocation from code"    |
| Skills         | `https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview.md`        | "Extract skill folder structure, SKILL.md format, and loading behavior"                  |
| Skills Guide   | `https://platform.claude.com/docs/en/build-with-claude/skills-guide.md`                | "Extract the Skills API (`/v1/skills`) usage and the migration steps from skills-2025-10-02" |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 代码执行 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool.md` | "提取代码执行工具的设置、文件上传、容器复用与响应处理" |
| 计算机使用 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool.md` | "提取 computer_toolset_20260801 的设置（configs、成员工具、批量操作、结果中的 toolset_name）、兼容性矩阵，以及从 computer_20251124 迁移的步骤" |
| Bash 工具 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/bash-tool.md` | "提取 bash 工具 schema、参考实现与安全注意事项" |
| 文本编辑器 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/text-editor-tool.md` | "提取文本编辑器工具的命令、schema 与参考实现" |
| 记忆工具 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md` | "提取记忆工具命令、目录结构与实现模式" |
| 工具搜索 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool.md` | "提取工具搜索的设置、适用时机与缓存交互" |
| 程序化工具调用 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling.md` | "提取 PTC 设置、脚本执行模型与从代码中调用工具的方式" |
| 技能 | `https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview.md` | "提取技能目录结构、SKILL.md 格式与加载行为" |
| 技能指南 | `https://platform.claude.com/docs/en/build-with-claude/skills-guide.md` | "提取 Skills API（`/v1/skills`）的用法以及从 skills-2025-10-02 迁移的步骤" |

### Advanced Features / 高级功能

| Topic              | URL                                                                           | Extraction Prompt                                   |
| ------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------- |
| Structured Outputs | `https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md` | "Extract output_config.format usage and schema enforcement"                           |
| Compaction         | `https://platform.claude.com/docs/en/build-with-claude/compaction.md`         | "Extract compaction setup, trigger config, and streaming with compaction"             |
| Context Editing    | `https://platform.claude.com/docs/en/build-with-claude/context-editing.md`    | "Extract context editing thresholds, what gets cleared, and configuration"            |
| Citations          | `https://platform.claude.com/docs/en/build-with-claude/citations.md`          | "Extract citation format and implementation"        |
| Context Windows    | `https://platform.claude.com/docs/en/build-with-claude/context-windows.md`    | "Extract context window sizes and token management" |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 结构化输出 | `https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md` | "提取 output_config.format 用法与 schema 强制机制" |
| 压缩（Compaction） | `https://platform.claude.com/docs/en/build-with-claude/compaction.md` | "提取压缩设置、触发配置与带压缩的流式传输" |
| 上下文编辑 | `https://platform.claude.com/docs/en/build-with-claude/context-editing.md` | "提取上下文编辑阈值、被清除的内容与配置方式" |
| 引用 | `https://platform.claude.com/docs/en/build-with-claude/citations.md` | "提取引用格式与实现" |
| 上下文窗口 | `https://platform.claude.com/docs/en/build-with-claude/context-windows.md` | "提取上下文窗口大小与 token 管理" |

### Managed Agents / 托管智能体

Use these when a managed-agents binding, behavior, or wire-level detail isn't covered in the cached `shared/managed-agents-*.md` concept files or in `{lang}/managed-agents/README.md`.

当托管智能体的绑定、行为或线上层细节未被缓存的 `shared/managed-agents-*.md` 概念文件或 `{lang}/managed-agents/README.md` 覆盖时，请使用这些 URL。

| Topic                 | URL                                                                              | Extraction Prompt                                                                               |
| --------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Overview              | `https://platform.claude.com/docs/en/managed-agents/overview.md`                 | "Extract the high-level architecture and how agents/sessions/environments/vaults fit together" |
| Quickstart            | `https://platform.claude.com/docs/en/managed-agents/quickstart.md`               | "Extract the minimal end-to-end agent -> environment -> session -> stream code path"              |
| Agent Setup           | `https://platform.claude.com/docs/en/managed-agents/agent-setup.md`              | "Extract agent create/update/list-versions/archive lifecycle and parameters"                   |
| Define Outcomes       | `https://platform.claude.com/docs/en/managed-agents/define-outcomes.md`          | "Extract outcome definitions, evaluation hooks, and success criteria configuration"             |
| Sessions              | `https://platform.claude.com/docs/en/managed-agents/sessions.md`                 | "Extract session lifecycle, status transitions, idle/terminated semantics, and resume rules"    |
| Environments          | `https://platform.claude.com/docs/en/managed-agents/environments.md`             | "Extract environment config (cloud/networking), management endpoints, and reuse model"          |
| Self-Hosted Sandboxes | `https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes.md`    | "Extract config:{type:self_hosted}, ANTHROPIC_ENVIRONMENT_KEY, EnvironmentWorker.run/handle_item, environments.work.poller(drain), beta_agent_toolset, ant beta:worker poll/run, webhook-driven wake, memory stores (ANTHROPIC_WORK_SECRET, memory_sync_interval/memory_sync_deletes)" |
| Self-Hosted Sandboxes - Security | `https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security.md` | "Extract what the customer owns (hardening, egress, key custody, trust boundaries) vs what Anthropic cannot do" |
| Events and Streaming  | `https://platform.claude.com/docs/en/managed-agents/events-and-streaming.md`     | "Extract event stream types, stream-first ordering, reconnect/dedupe, and steering patterns"    |
| Tools                 | `https://platform.claude.com/docs/en/managed-agents/tools.md`                    | "Extract built-in toolset, custom tool definitions, and tool result wire format"                |
| Files                 | `https://platform.claude.com/docs/en/managed-agents/files.md`                    | "Extract file upload, mount paths, session resources, and listing/downloading session outputs"  |
| Permission Policies   | `https://platform.claude.com/docs/en/managed-agents/permission-policies.md`      | "Extract permission policy types (`always_allow` / `always_ask` / `auto`), the three `auto` outcomes, the `evaluated_permission` + `evaluation` event fields, and per-tool config" |
| Multi-Agent           | `https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration.md` | "Extract multi-agent composition patterns, sub-agent invocation, and result handoff"            |
| Observability         | `https://platform.claude.com/docs/en/managed-agents/observability.md`            | "Extract logging, tracing, and usage telemetry exposed by managed agents"                       |
| Webhooks              | `https://platform.claude.com/docs/en/managed-agents/webhooks.md`                 | "Extract webhook endpoint registration, HMAC signature verification, supported event types, and delivery semantics" |
| GitHub                | `https://platform.claude.com/docs/en/managed-agents/github.md`                   | "Extract github_repository resource shape, multi-repo mounting, and token rotation"             |
| MCP Connector         | `https://platform.claude.com/docs/en/managed-agents/mcp-connector.md`            | "Extract MCP server declaration on agents and vault-based credential injection at session"     |
| Vaults                | `https://platform.claude.com/docs/en/managed-agents/vaults.md`                   | "Extract vault create, credential add/rotate, OAuth refresh shape, and archive"                 |
| Skills                | `https://platform.claude.com/docs/en/managed-agents/skills.md`                   | "Extract skill packaging and loading model for managed agents"                                  |
| Memory                | `https://platform.claude.com/docs/en/managed-agents/memory.md`                   | "Extract memory resource shape, scoping, and lifecycle"                                         |
| Onboarding            | `https://platform.claude.com/docs/en/managed-agents/onboarding.md`               | "Extract first-run setup, prerequisites, and account/region requirements"                      |
| Cloud Containers      | `https://platform.claude.com/docs/en/managed-agents/cloud-containers.md`         | "Extract cloud container runtime, image config, and network/storage knobs"                     |
| Migration             | `https://platform.claude.com/docs/en/managed-agents/migration.md`                | "Extract migration paths from earlier APIs/preview shapes to GA managed agents"                 |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| 概览 | `https://platform.claude.com/docs/en/managed-agents/overview.md` | "提取高层架构以及 agents/sessions/environments/vaults 如何协同" |
| 快速入门 | `https://platform.claude.com/docs/en/managed-agents/quickstart.md` | "提取最小的端到端 agent -> environment -> session -> stream 代码路径" |
| 智能体设置 | `https://platform.claude.com/docs/en/managed-agents/agent-setup.md` | "提取智能体创建/更新/列出版本/归档的生命周期与参数" |
| 定义成果 | `https://platform.claude.com/docs/en/managed-agents/define-outcomes.md` | "提取成果定义、评估钩子与成功标准配置" |
| 会话 | `https://platform.claude.com/docs/en/managed-agents/sessions.md` | "提取会话生命周期、状态转换、idle/terminated 语义与恢复规则" |
| 环境 | `https://platform.claude.com/docs/en/managed-agents/environments.md` | "提取环境配置（云/网络）、管理端点与复用模型" |
| 自托管沙箱 | `https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes.md` | "提取 config:{type:self_hosted}、ANTHROPIC_ENVIRONMENT_KEY、EnvironmentWorker.run/handle_item、environments.work.poller(drain)、beta_agent_toolset、ant beta:worker poll/run、webhook 驱动的唤醒、记忆存储（ANTHROPIC_WORK_SECRET、memory_sync_interval/memory_sync_deletes）" |
| 自托管沙箱 - 安全 | `https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security.md` | "提取客户负责的部分（加固、出口流量、密钥保管、信任边界）与 Anthropic 无法做到的部分" |
| 事件与流式传输 | `https://platform.claude.com/docs/en/managed-agents/events-and-streaming.md` | "提取事件流类型、流优先排序、重连/去重与转向模式" |
| 工具 | `https://platform.claude.com/docs/en/managed-agents/tools.md` | "提取内置工具集、自定义工具定义与工具结果线上格式" |
| 文件 | `https://platform.claude.com/docs/en/managed-agents/files.md` | "提取文件上传、挂载路径、会话资源以及会话输出的列举/下载" |
| 权限策略 | `https://platform.claude.com/docs/en/managed-agents/permission-policies.md` | "提取权限策略类型（`always_allow` / `always_ask` / `auto`）、三种 `auto` 结果、`evaluated_permission` + `evaluation` 事件字段以及按工具配置" |
| 多智能体 | `https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration.md` | "提取多智能体组合模式、子智能体调用与结果交接" |
| 可观测性 | `https://platform.claude.com/docs/en/managed-agents/observability.md` | "提取托管智能体暴露的日志、追踪与用量遥测" |
| Webhook | `https://platform.claude.com/docs/en/managed-agents/webhooks.md` | "提取 webhook 端点注册、HMAC 签名校验、支持的事件类型与投递语义" |
| GitHub | `https://platform.claude.com/docs/en/managed-agents/github.md` | "提取 github_repository 资源结构、多仓库挂载与令牌轮换" |
| MCP 连接器 | `https://platform.claude.com/docs/en/managed-agents/mcp-connector.md` | "提取智能体上的 MCP 服务器声明以及会话时基于 vault 的凭证注入" |
| Vault | `https://platform.claude.com/docs/en/managed-agents/vaults.md` | "提取 vault 创建、凭证添加/轮换、OAuth 刷新结构与归档" |
| 技能 | `https://platform.claude.com/docs/en/managed-agents/skills.md` | "提取托管智能体的技能打包与加载模型" |
| 记忆 | `https://platform.claude.com/docs/en/managed-agents/memory.md` | "提取记忆资源的结构、作用域与生命周期" |
| 入门引导 | `https://platform.claude.com/docs/en/managed-agents/onboarding.md` | "提取首次运行设置、前置条件与账户/区域要求" |
| 云容器 | `https://platform.claude.com/docs/en/managed-agents/cloud-containers.md` | "提取云容器运行时、镜像配置与网络/存储配置项" |
| 迁移 | `https://platform.claude.com/docs/en/managed-agents/migration.md` | "提取从早期 API/预览形态迁移到 GA 托管智能体的路径" |

### Anthropic CLI / Anthropic CLI

The `ant` CLI provides terminal access to the Claude API. Every API resource is exposed as a subcommand. It is the recommended way to keep agents, environments, skills, memory stores and deployments as version-controlled files (`ant apply` - see `shared/anthropic-cli.md`), and also exposes sessions and every other API resource for scripting and interactive inspection.

`ant` CLI 提供对 Claude API 的终端访问。每个 API 资源都以子命令的形式暴露。它是把智能体、环境、技能、记忆存储与部署保存为版本受控文件（`ant apply`——见 `shared/anthropic-cli.md`）的推荐方式，同时也为脚本化和交互式检查暴露 sessions 及其他所有 API 资源。

| Topic         | URL                                                     | Extraction Prompt                                                                                  |
| ------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Anthropic CLI | `https://platform.claude.com/docs/en/cli-sdks-libraries/cli/quickstart.md` | "Extract CLI install, authentication, command structure, and sending a first request" |
| `ant apply` | `https://platform.claude.com/docs/en/cli-sdks-libraries/cli/apply.md` | "Extract the file layout per resource kind, how a file's kind is inferred, path references between files, `claude-lock.json`, the flags (`--dry-run`, `--yes`, `--force`, `--prune`, `--upgrade`, `--lock-file`), and the CI setup" |
| `ant beta:sessions connect` | `https://platform.claude.com/docs/en/cli-sdks-libraries/cli/sessions-connect.md` | "Extract the interactive session viewer: keybindings, tool-call allow/deny prompt, `--web` local viewer and its URL/lifetime rules" |
| Authentication overview | `https://platform.claude.com/docs/en/manage-claude/authentication.md` | "Extract the credential options (API keys, interactive OAuth login, Workload Identity Federation) and when to use each" |
| WIF reference | `https://platform.claude.com/docs/en/manage-claude/wif-reference.md`  | "Extract credential precedence order, the profile configuration file schema, and the configuration directory layout" |

中文版表格：

| 主题 | URL | 抽取提示词 |
|---|---|---|
| Anthropic CLI | `https://platform.claude.com/docs/en/cli-sdks-libraries/cli/quickstart.md` | "提取 CLI 安装、认证、命令结构以及发送首个请求" |
| `ant apply` | `https://platform.claude.com/docs/en/cli-sdks-libraries/cli/apply.md` | "提取每种资源类型的文件布局、文件类型的推断方式、文件间路径引用、`claude-lock.json`、各标志（`--dry-run`、`--yes`、`--force`、`--prune`、`--upgrade`、`--lock-file`）与 CI 设置" |
| `ant beta:sessions connect` | `https://platform.claude.com/docs/en/cli-sdks-libraries/cli/sessions-connect.md` | "提取交互式会话查看器：按键绑定、工具调用允许/拒绝提示、`--web` 本地查看器及其 URL/生命周期规则" |
| 认证概览 | `https://platform.claude.com/docs/en/manage-claude/authentication.md` | "提取凭证选项（API 密钥、交互式 OAuth 登录、工作负载身份联合）及各自的适用时机" |
| WIF 参考 | `https://platform.claude.com/docs/en/manage-claude/wif-reference.md` | "提取凭证优先级顺序、profile 配置文件 schema 与配置目录布局" |

---

## Claude API SDK Repositories / Claude API SDK 仓库

WebFetch these when a binding (class, method, namespace, field) isn't covered in the cached `{lang}/` skill files or in the managed-agents docs above. The SDKs include beta managed-agents support for `/v1/agents`, `/v1/sessions`, `/v1/environments`, and related resources - search the repo for `BetaManagedAgents`, `beta.agents`, `beta.sessions`, or the equivalent namespace for that language.

当某个绑定（类、方法、命名空间、字段）未被缓存的 `{lang}/` 技能文件或上文的托管智能体文档覆盖时，请对这些仓库使用 WebFetch。各 SDK 均包含针对 `/v1/agents`、`/v1/sessions`、`/v1/environments` 及相关资源的 beta 托管智能体支持——可在仓库中搜索 `BetaManagedAgents`、`beta.agents`、`beta.sessions` 或该语言的等价命名空间。

| SDK        | URL                                                      | Extraction Prompt                                                                                                       |
| ---------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Python     | `https://github.com/anthropics/anthropic-sdk-python`     | "Extract beta managed-agents namespaces, classes, and method signatures (`client.beta.agents`, `client.beta.sessions`)" |
| TypeScript | `https://github.com/anthropics/anthropic-sdk-typescript` | "Extract beta managed-agents namespaces, classes, and method signatures (`client.beta.agents`, `client.beta.sessions`)" |
| Java       | `https://github.com/anthropics/anthropic-sdk-java`       | "Extract beta managed-agents classes, builders, and method signatures (`client.beta().agents()`, `BetaManagedAgents*`)" |
| Go         | `https://github.com/anthropics/anthropic-sdk-go`         | "Extract beta managed-agents types and method signatures (`client.Beta.Agents`, `BetaManagedAgents*` event types)"      |
| Ruby       | `https://github.com/anthropics/anthropic-sdk-ruby`       | "Extract beta managed-agents methods and parameter shapes (`client.beta.agents`, `client.beta.sessions`)"               |
| C#         | `https://github.com/anthropics/anthropic-sdk-csharp`     | "Extract beta managed-agents classes and method signatures (NuGet package, `BetaManagedAgents*` types)"                 |
| PHP        | `https://github.com/anthropics/anthropic-sdk-php`        | "Extract beta managed-agents classes and method signatures (`$client->beta->agents`, `BetaManagedAgents*` params)"      |

中文版表格：

| SDK | URL | 抽取提示词 |
|---|---|---|
| Python | `https://github.com/anthropics/anthropic-sdk-python` | "提取 beta 托管智能体命名空间、类与方法签名（`client.beta.agents`、`client.beta.sessions`）" |
| TypeScript | `https://github.com/anthropics/anthropic-sdk-typescript` | "提取 beta 托管智能体命名空间、类与方法签名（`client.beta.agents`、`client.beta.sessions`）" |
| Java | `https://github.com/anthropics/anthropic-sdk-java` | "提取 beta 托管智能体类、构建器与方法签名（`client.beta().agents()`、`BetaManagedAgents*`）" |
| Go | `https://github.com/anthropics/anthropic-sdk-go` | "提取 beta 托管智能体类型与方法签名（`client.Beta.Agents`、`BetaManagedAgents*` 事件类型）" |
| Ruby | `https://github.com/anthropics/anthropic-sdk-ruby` | "提取 beta 托管智能体方法与参数形态（`client.beta.agents`、`client.beta.sessions`）" |
| C# | `https://github.com/anthropics/anthropic-sdk-csharp` | "提取 beta 托管智能体类与方法签名（NuGet 包、`BetaManagedAgents*` 类型）" |
| PHP | `https://github.com/anthropics/anthropic-sdk-php` | "提取 beta 托管智能体类与方法签名（`$client->beta->agents`、`BetaManagedAgents*` 参数）" |

Each SDK repo also ships runnable programs under `examples/` - including the refusal-fallback / `fallbacks` examples (client-side middleware registration, fallback state, server-side `fallbacks` param). Fetch those for exact per-language syntax instead of translating another language's example.

每个 SDK 仓库还在 `examples/` 下附带可运行的程序——包括拒答回退 / `fallbacks` 示例（客户端中间件注册、回退状态、服务器端 `fallbacks` 参数）。请获取这些示例以获得准确的各语言语法，而不是改写另一门语言的示例。

### SDK major-version upgrade guides / SDK 大版本升级指南

Authoritative change lists for upgrading the SDK package itself across a major version. The bundled `{lang}/claude-api/sdk-upgrade.md` is the executable form; when the two disagree, the repository guide wins.

用于跨大版本升级 SDK 包本身的权威变更清单。随附的 `{lang}/claude-api/sdk-upgrade.md` 是其可执行形式；两者不一致时，以仓库指南为准。

【评论】这里显式声明了优先级规则：在线仓库指南优先于随技能打包的缓存文档，属于"活文档覆盖静态缓存"的设计。

| SDK                | URL                                                                         | Extraction Prompt                                                                                                   |
| ------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Python (0.x -> 1.x) | `https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md` | "Extract every breaking change with its before/after code, the new minimum Python version, and the upgrade command" |

中文版表格：

| SDK | URL | 抽取提示词 |
|---|---|---|
| Python（0.x -> 1.x） | `https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md` | "提取每项破坏性变更及其前后代码、新的最低 Python 版本与升级命令" |

---

## Fallback Strategy / 回退策略

If WebFetch fails (network issues, URL changed):

如果 WebFetch 失败（网络问题、URL 变更）：

1. Use cached content from the language-specific files (note the cache date)
   使用各语言专属文件中的缓存内容（注意缓存日期）
2. Inform user the data may be outdated
   告知用户数据可能已过时
3. Suggest they check platform.claude.com or the GitHub repos directly
   建议用户直接查看 platform.claude.com 或 GitHub 仓库
