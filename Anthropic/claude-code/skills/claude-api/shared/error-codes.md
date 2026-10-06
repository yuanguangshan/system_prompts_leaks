<!-- BILINGUAL-EN-ZH -->
# HTTP Error Codes Reference / HTTP 错误码参考

This file documents HTTP error codes returned by the Claude API, their common causes, and how to handle them. For language-specific error handling examples, see the `python/` or `typescript/` folders.

本文件记录 Claude API 返回的 HTTP 错误码、其常见原因及处理方法。各语言的错误处理示例参见 `python/` 或 `typescript/` 文件夹。

## Error Code Summary / 错误码汇总

| Code | Error Type              | Retryable | Common Cause                         |
| ---- | ----------------------- | --------- | ------------------------------------ |
| 400  | `invalid_request_error` | No        | Invalid request format or parameters |
| 401  | `authentication_error`  | No        | Invalid or missing API key           |
| 402  | `billing_error`         | No        | Billing or payment problem           |
| 403  | `permission_error`      | No        | Not allowed for this credential      |
| 404  | `not_found_error`       | No        | Unknown endpoint, or model not found or not available to your org |
| 413  | `request_too_large`     | No        | Request exceeds size limits          |
| 429  | `rate_limit_error`      | Yes       | Too many requests                    |
| 500  | `api_error`             | Yes       | Anthropic service issue              |
| 529  | `overloaded_error`      | Yes       | API is temporarily overloaded        |

| 代码 | 错误类型 | 可重试 | 常见原因 |
| ---- | --- | --- | --- |
| 400 | `invalid_request_error` | 否 | 请求格式或参数无效 |
| 401 | `authentication_error` | 否 | API 密钥无效或缺失 |
| 402 | `billing_error` | 否 | 计费或支付问题 |
| 403 | `permission_error` | 否 | 该凭证无权执行此操作 |
| 404 | `not_found_error` | 否 | 未知端点，或模型不存在或对你的组织不可用 |
| 413 | `request_too_large` | 否 | 请求超过大小限制 |
| 429 | `rate_limit_error` | 是 | 请求过多 |
| 500 | `api_error` | 是 | Anthropic 服务问题 |
| 529 | `overloaded_error` | 是 | API 暂时过载 |

## Detailed Error Information / 详细错误信息

### 400 Bad Request / 400 错误请求

**Causes:**

**原因：**

- Malformed JSON in request body
  请求体中的 JSON 格式错误
- Missing required parameters (`model`, `max_tokens`, `messages`)
  缺少必需参数（`model`、`max_tokens`、`messages`）
- Invalid parameter types (e.g., string where integer expected)
  参数类型无效（如应为整数处传了字符串）
- Empty messages array
  messages 数组为空
- Messages not alternating user/assistant
  消息未按 user/assistant 交替
- An `anthropic-beta` value that does not exist or is not enabled for your organization. Both cases return the same message: ``Unexpected value(s) `<value>` for the `anthropic-beta` header.``
  一个不存在或未对你的组织启用的 `anthropic-beta` 值。两种情况返回相同的消息：``Unexpected value(s) `<value>` for the `anthropic-beta` header.``

**Example error:**

**错误示例：**

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

**Fix:** Validate request structure before sending. Check that:

**修复：**发送前校验请求结构。检查：

- `model` is a valid model ID
  `model` 是有效的模型 ID
- `max_tokens` is a positive integer
  `max_tokens` 是正整数
- `messages` array is non-empty and alternates correctly
  `messages` 数组非空且正确交替

---

### 401 Unauthorized / 401 未授权

**Causes:**

**原因：**

- Missing `x-api-key` header or `Authorization` header
  缺少 `x-api-key` 头或 `Authorization` 头
- Invalid API key format
  API 密钥格式无效
- Revoked or deleted API key
  API 密钥已被吊销或删除
- OAuth bearer token sent via `x-api-key` instead of `Authorization: Bearer`
  OAuth bearer 令牌通过 `x-api-key` 而非 `Authorization: Bearer` 发送
- Both `ANTHROPIC_API_KEY` and `ANTHROPIC_AUTH_TOKEN` set - the SDK sends both headers and the API rejects the request
  同时设置了 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN`——SDK 会同时发送两个头，API 拒绝该请求

**Fix:** Set `ANTHROPIC_API_KEY`, or run `ant auth login` and leave the client constructor empty. For raw HTTP with an OAuth token, use `Authorization: Bearer <token>` (not `x-api-key:`).

**修复：**设置 `ANTHROPIC_API_KEY`，或运行 `ant auth login` 并让客户端构造函数留空。对携带 OAuth 令牌的原始 HTTP，使用 `Authorization: Bearer <token>`（而非 `x-api-key:`）。

---

### 403 Forbidden / 403 禁止访问

**Causes:**

**原因：**

- The credential's organization or workspace is not allowed to perform this operation.
  凭证所属的组织或工作区无权执行此操作。
- The request was blocked by an access requirement, such as a region restriction or identity verification, for a model your organization can otherwise use. The message says what to do.
  对于你的组织本可使用的某个模型，请求被访问要求（如区域限制或身份验证）拦截。消息会说明该怎么做。
- Rarely, the model server denies a request that passed the API's access check. The message is `Access to this model requires an access grant your request does not have.`
  少数情况下，模型服务器会拒绝一个已通过 API 访问检查的请求。消息为 `Access to this model requires an access grant your request does not have.`

A model your organization cannot use is normally a 404, not a 403 (see below). A beta header your organization is not enabled for is a 400.

你的组织无法使用的模型通常是 404 而非 403（见下文）。未对你的组织启用的 beta 头则是 400。

**Fix:** Check your organization's access and workspace settings in the Console.

**修复：**在 Console 中检查你组织的访问权限与工作区设置。

---

### 404 Not Found / 404 未找到

**Causes:**

**原因：**

- Typo in model ID (e.g., `claude-sonnet-4.6` instead of `claude-sonnet-4-6`)
  模型 ID 拼写错误（如 `claude-sonnet-4.6` 而非 `claude-sonnet-4-6`）
- Using deprecated model ID
  使用了已弃用的模型 ID
- A model ID that exists but is not available to your organization
  模型 ID 存在但对你的组织不可用
- Invalid API endpoint
  API 端点无效

A model that does not exist and a model your organization cannot use return the same response, `not_found_error` with a message that starts with `model: <id>`. The API does not reveal whether a model exists to callers who cannot use it.

不存在的模型与你的组织无法使用的模型返回相同的响应：`not_found_error`，消息以 `model: <id>` 开头。对无法使用某模型的调用方，API 不会透露该模型是否存在。

【评论】对无权使用的调用方不区分"模型不存在"与"模型不可用"，属于避免向外界泄露产品与可用性信息的信息披露控制。

**Fix:** Use exact model IDs from the models documentation. You can use aliases (e.g., `claude-opus-5-5`). To see which models your organization can use, call `GET /v1/models`.

**修复：**使用模型文档中的确切模型 ID。可以使用别名（如 `claude-opus-5-5`）。要查看你的组织可使用哪些模型，调用 `GET /v1/models`。

---

### 413 Request Too Large / 413 请求过大

**Causes:**

**原因：**

- Request body exceeds maximum size
  请求体超过最大尺寸
- Too many tokens in input
  输入中的 token 过多
- Image data too large
  图像数据过大

**Fix:** Reduce input size - truncate conversation history, compress/resize images, or split large documents into chunks.

**修复：**减小输入体积——截断会话历史、压缩/缩放图像，或将大文档分块。

---

### 400 Validation Errors / 400 校验错误

Some 400 errors are specifically related to parameter validation:

有些 400 错误专门与参数校验有关：

- `max_tokens` exceeds model's limit
  `max_tokens` 超过模型上限
- Invalid `temperature` value (must be 0.0-1.0)
  `temperature` 值无效（必须为 0.0-1.0）
- `budget_tokens` >= `max_tokens` in extended thinking
  扩展思考中 `budget_tokens` >= `max_tokens`
- Invalid tool definition schema
  工具定义 schema 无效

**Model-specific 400s on Claude Opus 5.5 / Claude Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7:**

**Claude Opus 5.5 / Claude Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 上特定于模型的 400：**

- `temperature`, `top_p`, `top_k` are removed - sending any of them returns 400. Delete the parameter; see `shared/model-migration.md` -> Per-SDK Syntax Reference.
  `temperature`、`top_p`、`top_k` 已被移除——发送其中任何一个都会返回 400。删除该参数；参见 `shared/model-migration.md` 的 Per-SDK Syntax Reference。
- `thinking: {type: "enabled", budget_tokens: N}` is removed - sending it returns 400. Use `thinking: {type: "adaptive"}` instead.
  `thinking: {type: "enabled", budget_tokens: N}` 已被移除——发送它会返回 400。改用 `thinking: {type: "adaptive"}`。
- **Claude Opus 5:** `thinking: {type: "disabled"}` returns 400 when `effort` is `xhigh` or `max` - it is accepted at `high` or below. Thinking is on by default, so omitting the param runs adaptive rather than disabling it.
  **Claude Opus 5：**当 `effort` 为 `xhigh` 或 `max` 时，`thinking: {type: "disabled"}` 返回 400——在 `high` 或以下则可接受。思考默认开启，因此省略该参数会运行 adaptive 模式而非禁用思考。
- **Fable 5/5.1 only:** an explicit `thinking: {type: "disabled"}` returns 400 at any effort (it is accepted on Opus 4.8/4.7). Omit the `thinking` param entirely instead.
  **仅 Fable 5/5.1：**显式的 `thinking: {type: "disabled"}` 在任何 effort 下都返回 400（Opus 4.8/4.7 上可接受）。应完全省略 `thinking` 参数。
- **Fable 5/5.1, Mythos 5/5.1:** if the organization or workspace is set to zero data retention (ZDR) - or any retention below the required 30 days - then **all** requests to these models return `400 invalid_request_error` ("In order to access this model, your organization or workspace must have data retention enabled."), even with a perfectly valid payload; ZDR only if expressly authorized by Anthropic. Check the retention configuration before debugging the request body.
  **Fable 5/5.1、Mythos 5/5.1：**如果组织或工作区被设为零数据保留（ZDR）——或任何低于要求的 30 天的保留期——那么对这几个模型的**所有**请求都会返回 `400 invalid_request_error`（"In order to access this model, your organization or workspace must have data retention enabled."），即使载荷完全有效；ZDR 仅在 Anthropic 明确授权时可用。调试请求体之前，先检查保留期配置。
  【评论】"账户级数据保留配置会让所有请求以 400 失败、且与载荷内容无关"说明：排错并不总在代码层，组织/工作区配置同样会以请求错误的形式暴露。
- **Claude Opus 5.5:** `thinking: {type: "disabled"}` or `{type: "enabled", budget_tokens: N}` returns 400 `"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.` (`"thinking.type.enabled"` for the budget form) at every effort level - omit `thinking` and lower `output_config.effort` instead. A `tools` entry of type `computer_20251124` returns 400 `'claude-opus-5-5' does not support tool types: computer_20251124.` followed by `Did you mean one of` and the accepted types - declare `{type: "computer_toolset_20260801"}` instead (no beta header, no `name` / display size). See `shared/model-migration.md` -> Migrating to Claude Opus 5.5.
  **Claude Opus 5.5：**在每个 effort 级别上，`thinking: {type: "disabled"}` 或 `{type: "enabled", budget_tokens: N}` 都返回 400 `"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.`（预算形式对应 `"thinking.type.enabled"`）——应省略 `thinking` 并调低 `output_config.effort`。类型为 `computer_20251124` 的 `tools` 条目返回 400 `'claude-opus-5-5' does not support tool types: computer_20251124.`，后跟 `Did you mean one of` 和可接受的类型——应改声明 `{type: "computer_toolset_20260801"}`（无需 beta 头，无 `name`/显示尺寸）。参见 `shared/model-migration.md` 的 Migrating to Claude Opus 5.5。
- **Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5:** `tool_choice: {type: "any"}` or `{type: "tool", name: ...}` returns 400 `tool_choice: type "tool" and "any" are not supported for this model.` - also on `count_tokens` and Batches. Use `{type: "auto"}` plus a prompt instruction (`strict: true` for schema-valid arguments), or structured outputs.
  **Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5：**`tool_choice: {type: "any"}` 或 `{type: "tool", name: ...}` 返回 400 `tool_choice: type "tool" and "any" are not supported for this model.`——在 `count_tokens` 和 Batches 上同样如此。使用 `{type: "auto"}` 加提示词指示（需要 schema 合法参数时加 `strict: true`），或使用结构化输出。
- **Claude Sonnet 5.5:** `thinking: {type: "disabled"}` returns 400 `"thinking.type.disabled" is not supported for this model. Use "thinking.type.between_tools" for the lowest thinking setting, or "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.` - send `{type: "between_tools"}` to turn thinking off, or leave thinking on at a lower effort. `between_tools` has its own 400s: at effort `xhigh` / `max` (`output_config.effort 'xhigh' is not supported when thinking is disabled on this model. Use effort 'high' or below, or enable thinking.`), with `display`, `budget_tokens`, or `block_binding` beside it, on a per-message effort change (`messages.N: output_config.effort 'low' differs from the 'high' in effect before it; ...`), and on any other model (`"thinking.type.between_tools" is not supported for this model.`). On the Claude API and Google Cloud a `computer_20251124` tool returns 400 `'claude-sonnet-5-5' does not support tool types: computer_20251124.` - declare `{type: "computer_toolset_20260801"}` (Amazon Bedrock still accepts the earlier tool). An advisor tool `model` of Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, Claude Sonnet 5, or Sonnet 4.6 returns 400 with a Claude Sonnet 5.5 executor. The history-editing check in the next bullet also applies to Claude Sonnet 5.5 thinking blocks - enforced by default for new accounts on the Claude API and Amazon Bedrock - and `block_binding` is accepted only with thinking on. See `shared/model-migration.md` -> Migrating to Claude Sonnet 5.5.
  **Claude Sonnet 5.5：**`thinking: {type: "disabled"}` 返回 400 `"thinking.type.disabled" is not supported for this model. Use "thinking.type.between_tools" for the lowest thinking setting, or "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.`——发送 `{type: "between_tools"}` 可关闭思考，或保持思考开启并调低 effort。`between_tools` 自身也有多种 400 场景：effort 为 `xhigh`/`max` 时（`output_config.effort 'xhigh' is not supported when thinking is disabled on this model. Use effort 'high' or below, or enable thinking.`）、旁边带 `display`、`budget_tokens` 或 `block_binding` 时、按消息修改 effort 时（`messages.N: output_config.effort 'low' differs from the 'high' in effect before it; ...`），以及在任何其他模型上（`"thinking.type.between_tools" is not supported for this model.`）。在 Claude API 和 Google Cloud 上，`computer_20251124` 工具返回 400 `'claude-sonnet-5-5' does not support tool types: computer_20251124.`——应声明 `{type: "computer_toolset_20260801"}`（Amazon Bedrock 仍接受较早的工具）。在 Claude Sonnet 5.5 执行器下，advisor 工具的 `model` 为 Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5 或 Sonnet 4.6 时返回 400。下一条中的历史编辑检查也适用于 Claude Sonnet 5.5 的思考块——对 Claude API 和 Amazon Bedrock 上的新账户默认强制——且 `block_binding` 只在思考开启时被接受。参见 `shared/model-migration.md` 的 Migrating to Claude Sonnet 5.5。
- **Claude Fable 5.1 / Claude Opus 5.5 - preserved thinking / history-editing check (new accounts created on/after 2026-08-31 on every platform, or any request that sets `prefix_mismatch_behavior`; Claude Mythos 5.1 doesn't run it):** ``messages.N.content.M: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block".`` (plus a sentence naming the beta header when it wasn't sent, and optionally one naming the first message that changed) means the system prompt, tool list, or an earlier message changed since that thinking block was produced. Retrying the same body never clears it; `count_tokens` returns the same 400. (In the Message Batches API the *unset* default drops the failing blocks instead of failing the item - a Batches item fails as `errored` only with `prefix_mismatch_behavior: "error"` set.) Strip the named block and every thinking block after it and retry once, or resend with `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` under beta `thinking-binding-controls-2026-08-01` (the beta is available on the Claude API, Claude Platform on AWS, Bedrock, and Vertex; Foundry unconfirmed - `shared/platform-availability.md`; without the header that field is a 400 ending `block_binding: Extra inputs are not permitted`); then fix the harness so it stops editing history (see `shared/model-migration.md` -> Migrating to Claude Fable 5.1 from Claude Fable 5). The same leading clause with *no* "bound to a different conversation" sentence is a tampered signature - always a 400, regardless of the setting.
  **Claude Fable 5.1 / Claude Opus 5.5——保留思考/历史编辑检查（2026-08-31 及之后在各平台创建的新账户，或任何设置了 `prefix_mismatch_behavior` 的请求；Claude Mythos 5.1 不运行该检查）：**``messages.N.content.M: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block".``（若未发送 beta 头还会附加一句指名该头的说明，并可能附加一句指名第一个被更改消息的说明）意味着自该思考块生成以来，系统提示词、工具列表或更早的某条消息发生了变化。重试相同请求体永远无法消除该错误；`count_tokens` 返回同样的 400。（在 Message Batches API 中，*未设置*时的默认行为是丢弃出错的块而不是使该条目失败——只有显式设置 `prefix_mismatch_behavior: "error"` 时，Batches 条目才以 `errored` 失败。）剥除被指名的块及其后的所有思考块并重试一次，或在 beta `thinking-binding-controls-2026-08-01` 下携带 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` 重新发送（该 beta 在 Claude API、Claude Platform on AWS、Bedrock 和 Vertex 上可用；Foundry 上未确认——见 `shared/platform-availability.md`；不带该头时该字段报 400，以 `block_binding: Extra inputs are not permitted` 结尾）；然后修复调用框架，使其停止编辑历史（参见 `shared/model-migration.md` 的 Migrating to Claude Fable 5.1 from Claude Fable 5）。同样的开头句但*没有*"bound to a different conversation"一句，则是签名被篡改——无论设置如何都一律 400。

**Common mistake with extended thinking on older models (Opus 4.6 and earlier):**

**旧模型（Opus 4.6 及更早）上扩展思考的常见错误：**

```
# Wrong: budget_tokens must be < max_tokens
thinking: budget_tokens=10000, max_tokens=1000  -> Error!

# Correct
thinking: budget_tokens=10000, max_tokens=16000
```

---

### 429 Rate Limited / 429 触发限流

**Causes:**

**原因：**

- Exceeded requests per minute (RPM)
  超出每分钟请求数（RPM）
- Exceeded tokens per minute (TPM)
  超出每分钟 token 数（TPM）
- Exceeded tokens per day (TPD)
  超出每天 token 数（TPD）

**Headers to check:**

**需要查看的响应头：**

- `retry-after`: Seconds to wait before retrying
  `retry-after`：重试前需等待的秒数
- `x-ratelimit-limit-*`: Your limits
  `x-ratelimit-limit-*`：你的限额
- `x-ratelimit-remaining-*`: Remaining quota
  `x-ratelimit-remaining-*`：剩余配额

**Fix:** The Anthropic SDKs automatically retry 429 and 5xx errors with exponential backoff (default: `max_retries=2`). For custom retry behavior, see the language-specific error handling examples.

**修复：**Anthropic SDK 会以指数退避自动重试 429 和 5xx 错误（默认 `max_retries=2`）。自定义重试行为参见各语言的错误处理示例。

---

### 500 Internal Server Error / 500 服务器内部错误

**Causes:**

**原因：**

- Temporary Anthropic service issue
  Anthropic 服务的临时问题
- Bug in API processing
  API 处理中的缺陷

**Fix:** Retry with exponential backoff. If persistent, check [status.anthropic.com](https://status.anthropic.com).

**修复：**以指数退避重试。若持续出现，查看 [status.anthropic.com](https://status.anthropic.com)。

---

### 529 Overloaded / 529 过载

**Causes:**

**原因：**

- High API demand
  API 需求过高
- Service capacity reached
  服务容量已达上限

**Fix:** Retry with exponential backoff. Consider using a different model (Haiku is often less loaded), spreading requests over time, or implementing request queuing.

**修复：**以指数退避重试。考虑换用其他模型（Haiku 的负载通常较低）、把请求在时间上摊开，或实现请求排队。

---

## Common Mistakes and Fixes / 常见错误与修复

| Mistake                         | Error            | Fix                                                     |
| ------------------------------- | ---------------- | ------------------------------------------------------- |
| `temperature`/`top_p`/`top_k` on Claude Opus 5.5 / Claude Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 | 400 | Remove the parameter (see `shared/model-migration.md`)  |
| `budget_tokens` on Claude Opus 5.5 / Claude Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 | 400  | Use `thinking: {type: "adaptive"}`                      |
| `thinking: {type: "disabled"}` on Fable 5/5.1 | 400    | Omit the `thinking` param entirely (accepted on Opus 4.8/4.7) |
| Org set to ZDR / retention below 30 days (Fable 5/5.1, Mythos 5/5.1) | 400 on every request | Fix the org's data-retention configuration - the payload isn't the problem |
| `thinking: {type: "disabled"}` or `budget_tokens` on Claude Opus 5.5 | 400 `"thinking.type.disabled" is not supported for this model` | Omit `thinking`; control depth with `output_config.effort` (default `medium`) |
| `computer_20251124` tool on Claude Opus 5.5 | 400 `does not support tool types: computer_20251124` | `{type: "computer_toolset_20260801"}` - no beta header, no `name` / display size; update the agent loop for member tool calls |
| `thinking: {type: "disabled"}` on Claude Sonnet 5.5 | 400 `"thinking.type.disabled" is not supported for this model` | `{type: "between_tools"}` at effort `high` or below (no other `thinking` field, no per-message effort change), or thinking on at a lower effort |
| `thinking: {type: "between_tools"}` on any other model, or at `xhigh` / `max` | 400 | Send it only to Claude Sonnet 5.5 at effort `high` or below; otherwise omit `thinking` |
| `tool_choice` `any` / `tool` on Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5 | 400 | `{type: "auto"}` + name the tool in the prompt (`strict: true` for schema-valid args), or structured outputs |
| Edited history replayed with thinking blocks (Claude Fable 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5, preserved thinking; Claude Mythos 5.1 doesn't run this check) | 400 `Invalid signature in thinking block ... bound to a different conversation` | Stop editing history - keep the transcript append-only, using mid-conversation `role: "system"` / tool-change messages, turn-scoped `clear_at` reminders that are never deleted, server-side context editing, and summary-only compaction instead of edits; recover once by stripping the named block and every thinking block after it (text and tool calls stay), or `prefix_mismatch_behavior: "drop_block"` (thinking on only - not with Claude Sonnet 5.5's `between_tools`) |
| `thinking.block_binding` without `thinking-binding-controls-2026-08-01` | 400 `block_binding: Extra inputs are not permitted` | Send the beta header where the controls beta is offered (`shared/platform-availability.md`); elsewhere remove `block_binding` and use strip-and-retry |
| `budget_tokens` >= `max_tokens` (older models) | 400 | Ensure `budget_tokens` < `max_tokens`                  |
| Typo in model ID                | 404              | Use valid model ID like `claude-opus-5-5`               |
| First message is `assistant`    | 400              | First message must be `user`                            |
| Consecutive same-role messages  | 400              | Alternate `user` and `assistant`                        |
| API key in code                 | 401 (leaked key) | Use environment variable                                |
| Custom retry needs              | 429/5xx          | SDK retries automatically; customize with `max_retries` |

| 常见错误 | 错误码 | 修复 |
| --- | --- | --- |
| 在 Claude Opus 5.5 / Claude Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 上使用 `temperature`/`top_p`/`top_k` | 400 | 删除该参数（见 `shared/model-migration.md`） |
| 在 Claude Opus 5.5 / Claude Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 上使用 `budget_tokens` | 400 | 使用 `thinking: {type: "adaptive"}` |
| 在 Fable 5/5.1 上使用 `thinking: {type: "disabled"}` | 400 | 完全省略 `thinking` 参数（Opus 4.8/4.7 上可接受） |
| 组织被设为 ZDR / 保留期低于 30 天（Fable 5/5.1、Mythos 5/5.1） | 每个请求都 400 | 修复组织的数据保留配置——问题不在载荷 |
| 在 Claude Opus 5.5 上使用 `thinking: {type: "disabled"}` 或 `budget_tokens` | 400 `"thinking.type.disabled" is not supported for this model` | 省略 `thinking`；用 `output_config.effort` 控制深度（默认 `medium`） |
| 在 Claude Opus 5.5 上使用 `computer_20251124` 工具 | 400 `does not support tool types: computer_20251124` | `{type: "computer_toolset_20260801"}`——无需 beta 头，无 `name`/显示尺寸；并更新智能体循环以适配成员工具调用 |
| 在 Claude Sonnet 5.5 上使用 `thinking: {type: "disabled"}` | 400 `"thinking.type.disabled" is not supported for this model` | 在 effort `high` 或以下使用 `{type: "between_tools"}`（不带其他 `thinking` 字段、不按消息改 effort），或保持思考开启并调低 effort |
| 在任何其他模型上、或在 `xhigh`/`max` 下使用 `thinking: {type: "between_tools"}` | 400 | 只发给 effort `high` 或以下的 Claude Sonnet 5.5；否则省略 `thinking` |
| 在 Claude Fable 5.1 / Claude Mythos 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5 上使用 `tool_choice` 的 `any`/`tool` | 400 | `{type: "auto"}` + 在提示词中点名工具（需要 schema 合法参数时加 `strict: true`），或结构化输出 |
| 编辑过的历史连同思考块一起重放（Claude Fable 5.1 / Claude Opus 5.5 / Claude Sonnet 5.5 的保留思考；Claude Mythos 5.1 不运行该检查） | 400 `Invalid signature in thinking block ... bound to a different conversation` | 停止编辑历史——保持转录只追加，改用会话中途的 `role: "system"`/工具变更消息、从不删除的回合级 `clear_at` 提醒、服务端上下文编辑以及仅摘要式压缩来代替编辑；一次性恢复：剥除被指名的块及其后的所有思考块（文本与工具调用保留），或使用 `prefix_mismatch_behavior: "drop_block"`（仅思考开启时——不适用于 Claude Sonnet 5.5 的 `between_tools`） |
| 没有 `thinking-binding-controls-2026-08-01` 时使用 `thinking.block_binding` | 400 `block_binding: Extra inputs are not permitted` | 在提供该 controls beta 的平台发送 beta 头（`shared/platform-availability.md`）；其他平台移除 `block_binding` 并改用"剥除后重试" |
| `budget_tokens` >= `max_tokens`（较旧模型） | 400 | 确保 `budget_tokens` < `max_tokens` |
| 模型 ID 拼写错误 | 404 | 使用有效的模型 ID，如 `claude-opus-5-5` |
| 首条消息为 `assistant` | 400 | 首条消息必须是 `user` |
| 连续同角色消息 | 400 | `user` 与 `assistant` 交替 |
| API 密钥写在代码里 | 401（密钥已泄露） | 使用环境变量 |
| 自定义重试需求 | 429/5xx | SDK 自动重试；用 `max_retries` 定制 |

## Typed Exceptions in SDKs / SDK 中的类型化异常

**Always use the SDK's typed exception classes** instead of checking error messages with string matching. Each HTTP status code maps to a specific exception class per SDK.

**始终使用 SDK 的类型化异常类**，而不是用字符串匹配检查错误消息。每个 HTTP 状态码在每个 SDK 中都映射到特定的异常类。

【评论】"按类型化异常而非错误消息字符串匹配"是官方 SDK 的通用设计：每个状态码对应独立异常类，使调用方能以语言原生的异常机制做可靠分支。

### Exception class names by language / 各语言的异常类名

| HTTP | Python (`anthropic.*`) / TypeScript (`Anthropic.*`) | Ruby (`Anthropic::Errors::*`) | Java (`com.anthropic.errors.*`) | C# | PHP (`Anthropic\Core\Exceptions\*`) |
|---|---|---|---|---|---|
| 400 | `BadRequestError` | `BadRequestError` | `BadRequestException` | `AnthropicBadRequestException` | `BadRequestException` |
| 401 | `AuthenticationError` | `AuthenticationError` | `UnauthorizedException` | `AnthropicUnauthorizedException` | `AuthenticationException` |
| 403 | `PermissionDeniedError` | `PermissionDeniedError` | `PermissionDeniedException` | `AnthropicForbiddenException` | `PermissionDeniedException` |
| 404 | `NotFoundError` | `NotFoundError` | `NotFoundException` | `AnthropicNotFoundException` | `NotFoundException` |
| 422 | `UnprocessableEntityError` | `UnprocessableEntityError` | `UnprocessableEntityException` | `AnthropicUnprocessableEntityException` | `UnprocessableEntityException` |
| 429 | `RateLimitError` | `RateLimitError` | `RateLimitException` | `AnthropicRateLimitException` | `RateLimitException` |
| >=500 | `InternalServerError` | `InternalServerError` | `InternalServerException` | `Anthropic5xxException` | `InternalServerException` |
| net | `APIConnectionError` | `APIConnectionError` | `AnthropicIoException` | `AnthropicIOException` | `APIConnectionException` |
| base | `APIError` (both); `APIStatusError` (Python only) | `APIStatusError` / `APIError` | `AnthropicServiceException` | `AnthropicApiException` | `APIStatusException` / `APIException` |

| HTTP 状态码 | Python (`anthropic.*`) / TypeScript (`Anthropic.*`) | Ruby (`Anthropic::Errors::*`) | Java (`com.anthropic.errors.*`) | C# | PHP (`Anthropic\Core\Exceptions\*`) |
|---|---|---|---|---|---|
| 400 | `BadRequestError` | `BadRequestError` | `BadRequestException` | `AnthropicBadRequestException` | `BadRequestException` |
| 401 | `AuthenticationError` | `AuthenticationError` | `UnauthorizedException` | `AnthropicUnauthorizedException` | `AuthenticationException` |
| 403 | `PermissionDeniedError` | `PermissionDeniedError` | `PermissionDeniedException` | `AnthropicForbiddenException` | `PermissionDeniedException` |
| 404 | `NotFoundError` | `NotFoundError` | `NotFoundException` | `AnthropicNotFoundException` | `NotFoundException` |
| 422 | `UnprocessableEntityError` | `UnprocessableEntityError` | `UnprocessableEntityException` | `AnthropicUnprocessableEntityException` | `UnprocessableEntityException` |
| 429 | `RateLimitError` | `RateLimitError` | `RateLimitException` | `AnthropicRateLimitException` | `RateLimitException` |
| >=500 | `InternalServerError` | `InternalServerError` | `InternalServerException` | `Anthropic5xxException` | `InternalServerException` |
| 网络 | `APIConnectionError` | `APIConnectionError` | `AnthropicIoException` | `AnthropicIOException` | `APIConnectionException` |
| 基类 | `APIError`（两者）；`APIStatusError`（仅 Python） | `APIStatusError` / `APIError` | `AnthropicServiceException` | `AnthropicApiException` | `APIStatusException` / `APIException` |

The Ruby and PHP classes live in a dedicated errors namespace - write `Anthropic::Errors::RateLimitError` and `Anthropic\Core\Exceptions\RateLimitException` (not bare `Anthropic::RateLimitError`). All 4xx C# exceptions also inherit from `Anthropic4xxException`.

Ruby 与 PHP 的类位于专门的错误命名空间——应写 `Anthropic::Errors::RateLimitError` 和 `Anthropic\Core\Exceptions\RateLimitException`（而不是裸的 `Anthropic::RateLimitError`）。所有 4xx 的 C# 异常还继承自 `Anthropic4xxException`。

### Catch most-specific first, in a chain / 按从最具体到基类的顺序链式捕获

Order `catch`/`except`/`rescue` clauses from the most specific subclass to the base class, with a separate clause for each category you handle differently - retryable (429, >=500, network) vs. non-retryable (4xx). The SDK defines a distinct class per status for exactly this reason; a single broad catch-all discards that information.

把 `catch`/`except`/`rescue` 子句从最具体的子类排到基类，对需要区别处理的每个类别各写一个子句——可重试（429、>=500、网络）与不可重试（4xx）。SDK 为每个状态定义单独的类正是为此；一个宽泛的兜底 catch 会丢弃这些信息。

```python
try:
    msg = client.messages.create(...)
except anthropic.NotFoundError as e:          # 404 - e.g. bad model ID
    ...
except anthropic.RateLimitError as e:         # 429 - back off and retry
    ...
except anthropic.APIStatusError as e:         # any other non-2xx HTTP response
    print(e.status_code, e.message)
except anthropic.APIConnectionError as e:     # network failure before a response
    ...
```

The same chain shape applies in every SDK: TypeScript `instanceof Anthropic.NotFoundError` -> `RateLimitError` -> `APIConnectionError` -> `APIError` (check `APIConnectionError` before `APIError` - in the TypeScript SDK it's a subclass of `APIError`, unlike Python where it's a sibling); Ruby `rescue Anthropic::Errors::NotFoundError` -> `...::RateLimitError` -> `...::APIStatusError`; Java `catch (NotFoundException) ... catch (RateLimitException) ... catch (AnthropicServiceException)`; C# `catch (AnthropicNotFoundException) ... catch (AnthropicRateLimitException) ... catch (AnthropicApiException)`; PHP `catch (NotFoundException) ... catch (RateLimitException) ... catch (APIStatusException)`.

同样的链式结构适用于每个 SDK：TypeScript 用 `instanceof Anthropic.NotFoundError` -> `RateLimitError` -> `APIConnectionError` -> `APIError`（先查 `APIConnectionError` 再查 `APIError`——在 TypeScript SDK 中它是 `APIError` 的子类，不像 Python 中是并列关系）；Ruby 用 `rescue Anthropic::Errors::NotFoundError` -> `...::RateLimitError` -> `...::APIStatusError`；Java 用 `catch (NotFoundException) ... catch (RateLimitException) ... catch (AnthropicServiceException)`；C# 用 `catch (AnthropicNotFoundException) ... catch (AnthropicRateLimitException) ... catch (AnthropicApiException)`；PHP 用 `catch (NotFoundException) ... catch (RateLimitException) ... catch (APIStatusException)`。

### Go - `errors.As` then branch on status / Go——先 `errors.As` 再按状态码分支

The Go SDK returns a single `*anthropic.Error` for all non-2xx responses. Unwrap it with `errors.As`, then branch on `StatusCode`:

Go SDK 对所有非 2xx 响应返回单一的 `*anthropic.Error`。用 `errors.As` 解包，然后按 `StatusCode` 分支：

```go
_, err := client.Messages.New(ctx, params)
if err != nil {
    var apierr *anthropic.Error
    if errors.As(err, &apierr) {
        switch apierr.StatusCode {
        case 404:
            // bad model ID / resource
        case 429:
            // back off and retry
        default:
            // other API error - apierr.StatusCode, apierr.RequestID
        }
    } else {
        // transport-level error (*url.Error wrapping *net.OpError, etc.)
    }
}
```

### Error `.type` Field / 错误的 `.type` 字段

All `APIStatusError` subclasses now expose a `.type` property (Python: `.type`, TypeScript: `.type`, Java: `.errorType()`, Go: `.Type()`, Ruby: `.type`, PHP: `.type`) that returns the API error type string (e.g., `"invalid_request_error"`, `"authentication_error"`, `"rate_limit_error"`, `"overloaded_error"`). Use this to classify errors by type name instead of by status code. `"billing_error"` is a 402 and `"permission_error"` is a 403.

所有 `APIStatusError` 的子类现在都暴露一个 `.type` 属性（Python：`.type`，TypeScript：`.type`，Java：`.errorType()`，Go：`.Type()`，Ruby：`.type`，PHP：`.type`），返回 API 错误类型字符串（如 `"invalid_request_error"`、`"authentication_error"`、`"rate_limit_error"`、`"overloaded_error"`）。用它按类型名而不是状态码对错误分类。`"billing_error"` 对应 402，`"permission_error"` 对应 403。

```python
except anthropic.APIStatusError as e:
    if e.type == "rate_limit_error":
        # handle rate limiting
    elif e.type == "overloaded_error":
        # handle overload
```
