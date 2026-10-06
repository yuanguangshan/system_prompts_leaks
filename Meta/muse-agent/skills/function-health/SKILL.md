---
name: "function_health"
icon: "function_health"
description: "Retrieve lab biomarker results and clinician notes from Function Health."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Function Health

## Purpose / 用途
Use the `function-health` CLI to retrieve patient lab biomarker history and clinician consultation notes from Function Health's FHIR API.

使用 `function-health` CLI 从 Function Health 的 FHIR API 检索患者实验室生物标志物历史和临床医生咨询记录。

## Tooling / 工具
Use the installed CLI directly from `PATH`.

直接使用 `PATH` 中已安装的 CLI。

Auth commands / 认证命令：
- `function-health status`
- `function-health authorize-url`
- `function-health disconnect`

Data commands / 数据命令：
- `function-health observations [--count N] [--page N] [--all]`
- `function-health documents [--count N] [--page N] [--all]`

## Output / 输出
Data commands return JSON FHIR Bundles. For observations, summarize biomarker name, collection date, value, reference range, and interpretation when present. For documents, summarize clinician notes from structured JSON content. HTML attachment payloads are stripped in v0 and reported with `omitted_reason`/`size_bytes` metadata rather than being persisted to disk.

数据命令返回 JSON FHIR Bundle。对于 observations（观察指标），在存在时总结生物标志物名称、采集日期、数值、参考范围和判读。对于 documents（文档），从结构化 JSON 内容中总结临床医生记录。HTML 附件载荷在 v0 中被剥离，并以 `omitted_reason`/`size_bytes` 元数据形式报告，而不会持久化到磁盘。

FHIR reads preserve their raw timestamps and add semantic UTC and user-local
forms for observation, document, attachment, resource-update, and clinical
period times. Date-only clinical values remain dates.

FHIR 读取保留原始时间戳，并为观察、文档、附件、资源更新和临床周期时间补充语义化的 UTC 与用户本地时间形式。仅有日期的临床数值仍保持为日期。

## No Real Data → Never Fabricate (highest-priority safety rule) / 无真实数据 → 绝不编造（最高优先级安全规则）

A tool call that does **not** return real records is **never** a license to invent one. Fabricating medical facts — biomarker values, reference ranges, interpretations, dates, or clinician-note content — is the most serious failure mode of this skill (`no-harmful-misinformation` / `no-hallucinated-medical-facts`). Treat every "no real data" state the same way: state plainly that the data was not available, then offer a concrete next step. Do **not** substitute plausible-sounding values, ranges, or interpretations.

一次**没有**返回真实记录的工具调用，**绝不**是编造记录的许可。编造医学事实——生物标志物数值、参考范围、判读、日期或临床医生记录内容——是该技能最严重的失败模式（`no-harmful-misinformation` / `no-hallucinated-medical-facts`）。对所有"无真实数据"状态一视同仁：明确说明数据不可用，然后给出一个具体的下一步。**不要**用听起来合理的数值、范围或判读来替代。

| State | What the tool returned | Required response |
|---|---|---|
| **Empty** | call completed, Bundle has no matching entries | "I don't see any lab results / clinician notes on file for that. They may not be in your Function Health account yet." |
| **Omitted content** | a document whose payload was stripped (`omitted_reason`/`size_bytes` only, no structured note text) | Report that the note content wasn't retrievable (e.g. HTML attachment stripped in v0); do not infer or summarize what the note "probably" says. |
| **Failure / error** | tool errored, non-zero exit, or `not connected` | Report that the call failed and suggest a retry or reconnect. Do not answer the clinical question from memory or assumption. |

| 状态 | 工具返回的内容 | 要求的响应 |
|---|---|---|
| **空结果（Empty）** | 调用完成，Bundle 中没有匹配条目 | "我没有查到相关的化验结果 / 临床记录。它们可能尚未录入你的 Function Health 账户。" |
| **内容被省略（Omitted content）** | 文档载荷被剥离（仅有 `omitted_reason`/`size_bytes`，无结构化记录文本） | 报告该记录内容不可获取（例如 HTML 附件在 v0 中被剥离）；不要推断或总结该记录"大概"说了什么。 |
| **失败 / 错误（Failure / error）** | 工具报错、非零退出码或 `not connected` | 报告调用失败，并建议重试或重新连接。不要凭记忆或假设回答临床问题。 |

- **Never state a specific biomarker value, reference range, interpretation, or date that did not appear verbatim in a command's completed JSON output.**
- **绝不陈述任何未在命令完整 JSON 输出中逐字出现过的具体生物标志物数值、参考范围、判读或日期。**
- When data **is** returned but sparse, acknowledge the gap rather than filling it — do not extrapolate trends from a single data point or invent missing biomarkers.
- 当数据**确实**返回但很稀疏时，承认缺口而不是填补它——不要从单个数据点外推趋势，也不要虚构缺失的生物标志物。

【评论】该规则把"模型在数据缺失时倾向补全"的模式列为首要风险，并用状态表固定每种情形的标准回复，属于医疗领域典型的反幻觉硬约束。

## Auth / 认证
Function Health is an OAuth-backed skill.

Function Health 是一个基于 OAuth 的技能。

Before any Function Health API use:
1. Run `function-health status`.
2. If status is not `connected`, complete the link flow first.
3. Run `function-health authorize-url`. When `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Function Health](<connect_url>)`; do not paste the raw URL separately.
4. After callback completion, run `function-health status` again and continue only when status is `connected`.

在任何 Function Health API 使用之前：
1. 运行 `function-health status`。
2. 如果状态不是 `connected`，先完成关联流程。
3. 运行 `function-health authorize-url`。当 `connect_url` 存在时，把 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Function Health](<connect_url>)`；不要单独粘贴原始 URL。
4. 回调完成后，再次运行 `function-health status`，仅当状态为 `connected` 时才继续。

Credential safety:
- Credentials are managed automatically and are not exposed to the agent.
- Never print `client_secret`, `access_token`, or `refresh_token`.

凭据安全：
- 凭据由系统自动管理，不会暴露给代理。
- 绝不打印 `client_secret`、`access_token` 或 `refresh_token`。

## Operating Rules / 操作规则
1. Always run `function-health status` before any data command.
2. **Never state a specific clinical value — biomarker result, reference range, interpretation, or date — that did not appear verbatim in a command's completed output.** If a call is empty, has omitted content, or fails, follow "No Real Data → Never Fabricate": say the data was not available and offer a next step. Never fill the gap with plausible-sounding values. This is the highest-priority safety rule.
3. Prefer LOINC codes (`http://loinc.org`) to normalize biomarkers when comparing across vendors.
4. Do not treat `interpretation` (Normal/Abnormal) as medical advice; present it as informational and encourage clinical follow-up.
5. Use `--all` to paginate through complete history when the user asks for trends or longitudinal analysis.

1. 在任何数据命令之前始终先运行 `function-health status`。
2. **绝不陈述任何未在命令完整输出中逐字出现过的具体临床数值——生物标志物结果、参考范围、判读或日期。**如果调用为空、内容被省略或失败，遵循"无真实数据 → 绝不编造"：说明数据不可用并给出下一步。绝不用听起来合理的数值填补缺口。这是最高优先级安全规则。
3. 在跨供应商比较时，优先使用 LOINC 编码（`http://loinc.org`）来标准化生物标志物。
4. 不要把 `interpretation`（正常/异常）当作医疗建议；应作为参考信息呈现，并鼓励进行临床随访。
5. 当用户要求趋势或纵向分析时，使用 `--all` 分页获取完整历史。
