---
name: "healthex"
title: "HealthEx"
description: "Use to connect HealthEx and ask questions about your medications, lab results, and other health records."
icon: "healthex"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# HealthEx (OAuth + MCP Health Records) / HealthEx（OAuth + MCP 健康记录）

## Purpose / 目的
Query patient health records through the HealthEx MCP server. Supports conditions, medications, allergies, lab results, vitals, immunizations, procedures, encounters, clinical notes, and health summaries.

通过 HealthEx MCP 服务器查询患者健康记录。支持病史（conditions）、用药（medications）、过敏（allergies）、化验结果、生命体征、免疫接种、诊疗操作（procedures）、就诊记录（encounters）、临床记录（clinical notes）与健康摘要。

Activate this skill when:
在以下情况下激活本技能：
- The user asks about their medical data, prescriptions, test results, diagnoses, or health history
  用户询问其医疗数据、处方、检查结果、诊断或健康史
- A response would benefit from the user's health context — e.g. diet or meal plans, workout plans, fitness assessments, travel health needs, doctor visit prep, or sleep/stress optimization
  回答可从用户的健康背景中受益——例如饮食或膳食计划、锻炼计划、体能评估、旅行健康需求、就诊准备，或睡眠/压力优化

## Tooling / 工具
Use:  
用法：  
```sh
$JARVIS_BIN_DIR/healthex <subcommand>
```

Subcommands:
子命令：
- `status` — check connector status (`connected`, `available`, or `unknown`). JSON output: `{ ok, provider, status, config_path, reason }`.
  `status` —— 检查连接器状态（`connected`、`available` 或 `unknown`）。JSON 输出：`{ ok, provider, status, config_path, reason }`。
- `setup` — write OAuth defaults with PKCE enabled. JSON output: `{ ok, provider, status, config_path }`.
  `setup` —— 写入启用 PKCE 的 OAuth 默认配置。JSON 输出：`{ ok, provider, status, config_path }`。
- `disconnect` — remove stored OAuth credentials and tokens. JSON output: `{ ok, action, config_path, removed }`.
  `disconnect` —— 移除存储的 OAuth 凭据与令牌。JSON 输出：`{ ok, action, config_path, removed }`。
- `authorize-url [--state <state>]` — generate PKCE authorize URL. JSON output: `{ ok, authorize_url, connect_url, state }`. Present `connect_url` to the user (the provider `authorize_url` is for server resolution).
  `authorize-url [--state <state>]` —— 生成 PKCE 授权 URL。JSON 输出：`{ ok, authorize_url, connect_url, state }`。向用户展示 `connect_url`（服务商的 `authorize_url` 用于服务器端解析）。
- `refresh` — refresh the access token (public client). JSON output: `{ ok, provider, status, config_path }`. Rarely needed directly — `mcp-call` auto-refreshes on 401.
  `refresh` —— 刷新访问令牌（公共客户端）。JSON 输出：`{ ok, provider, status, config_path }`。很少需要直接调用——`mcp-call` 会在 401 时自动刷新。
- `mcp-list` — list available MCP tools. JSON output: `{ ok, result }` where result contains the tools array.
  `mcp-list` —— 列出可用的 MCP 工具。JSON 输出：`{ ok, result }`，其中 result 包含工具数组。
- `mcp-call --tool <name> [--args '<json>']` — call an MCP tool. JSON output: `{ ok, result }`. Auto-refreshes token on 401.
  `mcp-call --tool <name> [--args '<json>']` —— 调用一个 MCP 工具。JSON 输出：`{ ok, result }`。在 401 时自动刷新令牌。

## Connection Guard / 连接守卫
Before any HealthEx data access:
在任何 HealthEx 数据访问之前：
1. Run `healthex status`.
   运行 `healthex status`。
2. If status is `connected`, proceed to data access.
   若状态为 `connected`，则进入数据访问。
3. Run `healthex authorize-url` to generate the link. When `connect_url` is present, replace `<connect_url>` with the returned URL and present a **single message** containing exactly the Markdown link `[Connect HealthEx](<connect_url>)` (never paste the raw URL separately) followed by these consent details:
   运行 `healthex authorize-url` 生成链接。当返回 `connect_url` 时，把 `<connect_url>` 替换为返回的 URL，并呈现一条**单一消息**：其中恰好包含 Markdown 链接 `[Connect HealthEx](<connect_url>)`（绝不要单独粘贴原始 URL），随后附上以下同意事项说明：
   - What is being connected: **HealthEx** — a service that aggregates health records from your healthcare providers
     将要连接的内容：**HealthEx** —— 一个聚合你各医疗服务机构健康记录的服务
   - What access is granted: **read-only** access to conditions, medications, allergies, lab results, vitals, immunizations, procedures, encounters, and clinical notes
     授予的权限：对病史、用药、过敏、化验结果、生命体征、免疫接种、诊疗操作、就诊记录和临床记录的**只读**访问
   - Duration: access persists until you revoke it from your HealthEx account
     期限：访问权限持续有效，直至你在 HealthEx 账户中撤销
   - How data is used: connecting HealthEx unlocks personalized health guidance — like explaining lab results, prepping for doctor visits, and understanding medications — all based on your actual health data
     数据用途：连接 HealthEx 可解锁个性化健康指导——例如解读化验结果、就诊准备和理解用药——全部基于你的真实健康数据
4. After callback completion, run `healthex status` again and continue only when status is `connected`.
   回调完成后再次运行 `healthex status`，仅当状态为 `connected` 时才继续。

## Data Retrieval Strategy / 数据检索策略

Start broad, then go deep based on the user's question.

先广泛、后深入，依据用户的问题展开。

**Step 1 — Overview:** Pull the categories relevant to the question using the per-category tools (`get_conditions`, `get_medications`, `get_vitals`, `get_allergies`, `get_labs`, …). These return data reliably. `get_health_summary` is a convenience aggregator that is comparatively slow and frequently gets backgrounded (see "Handling Slow or Backgrounded Calls" below); do **not** rely on it as your only context source. Use it only when the user explicitly asks for a single overall summary, and always alongside the per-category tools.

**第 1 步 —— 概览：** 使用各分类工具（`get_conditions`、`get_medications`、`get_vitals`、`get_allergies`、`get_labs` 等）拉取与问题相关的类别。这些工具能可靠地返回数据。`get_health_summary` 是一个便利性聚合器，速度较慢且经常被后台化（见下文"处理缓慢或被后台化的调用"）；**不要**把它当作唯一的上下文来源。仅当用户明确要求一份整体摘要时才使用它，且必须同时配合各分类工具。

**Step 2 — Targeted pulls:** Based on the question type, pull the right detail:

**第 2 步 —— 定向拉取：** 根据问题类型拉取正确的细节：

| User intent | Primary tools | Secondary tools |
|---|---|---|
| "What's my health summary?" | `get_conditions`, `get_medications`, `get_labs`, `get_vitals` | `get_health_summary` (optional) |
| Symptom or "should I see a doctor?" | `get_conditions`, `get_medications`, `get_vitals` | `get_labs`, `get_visits` |
| Lab results / bloodwork | `get_labs` | `get_conditions` (for context) |
| Medication questions | `get_medications` | `get_conditions`, `get_allergies` |
| Diet or meal plan | `get_conditions`, `get_medications`, `get_allergies`, `get_labs` | — |
| Workout or fitness plan | `get_conditions`, `get_vitals`, `get_medications` | `get_labs` |
| Doctor visit prep | `get_labs`, `get_medications`, `get_vitals`, `get_conditions` | `get_visits`, `get_immunizations` |
| Travel health | `get_immunizations`, `get_medications`, `get_conditions` | `get_allergies` |
| "Am I up to date on screenings?" | `get_labs`, `get_immunizations`, `get_visits` | `get_procedures` |

| 用户意图 | 主工具 | 次要工具 |
|---|---|---|
| "我的健康摘要是什么？" | `get_conditions`、`get_medications`、`get_labs`、`get_vitals` | `get_health_summary`（可选） |
| 症状或"我该去看医生吗？" | `get_conditions`、`get_medications`、`get_vitals` | `get_labs`、`get_visits` |
| 化验结果 / 血液检查 | `get_labs` | `get_conditions`（用于背景参考） |
| 用药问题 | `get_medications` | `get_conditions`、`get_allergies` |
| 饮食或膳食计划 | `get_conditions`、`get_medications`、`get_allergies`、`get_labs` | — |
| 锻炼或健身计划 | `get_conditions`、`get_vitals`、`get_medications` | `get_labs` |
| 就诊准备 | `get_labs`、`get_medications`、`get_vitals`、`get_conditions` | `get_visits`、`get_immunizations` |
| 旅行健康 | `get_immunizations`、`get_medications`、`get_conditions` | `get_allergies` |
| "我的筛查是否都跟上了？" | `get_labs`、`get_immunizations`、`get_visits` | `get_procedures` |

**Step 3 — Run `mcp-list` if needed.** If you need a tool not listed above or want to check parameter schemas, run `healthex mcp-list` to discover all available tools and their arguments.

**第 3 步 —— 按需运行 `mcp-list`。** 如果需要上表未列出的工具，或想查看参数 schema，运行 `healthex mcp-list` 来发现所有可用工具及其参数。

## Clinical Insight Patterns / 临床洞见模式

When presenting health data, go beyond raw data. Apply these patterns to surface actionable insights:

呈现健康数据时不要止步于原始数据。运用以下模式提炼可操作的洞见：

### 1. Care Gap Detection / 照护缺口检测
After pulling data, check for overdue or missing care:
拉取数据后，检查逾期或缺失的照护：
- **Lab staleness:** Flag any key lab >12 months old. Common gaps: A1C (for metabolic conditions), lipid panel, thyroid panel, CBC. Example: "Your last A1C was in May 2019 — that's over 6 years ago. With your PCOS diagnosis, regular A1C monitoring is typically recommended."
  **化验时效性：** 标记任何超过 12 个月的关键化验。常见缺口：A1C（针对代谢性疾病）、血脂组合、甲状腺组合、血常规（CBC）。示例："你上次 A1C 检测是 2019 年 5 月——已经超过 6 年了。鉴于你的多囊卵巢综合征（PCOS）诊断，通常建议定期监测 A1C。"
- **Visit recency:** If the most recent encounter is >12 months old, flag it. Example: "I don't see a PCP visit in the last 3 years. A routine checkup would be a good idea."
  **就诊近因：** 若最近一次就诊记录超过 12 个月，予以标记。示例："我看不到最近 3 年内的全科医生（PCP）就诊记录。做一次常规体检是个好主意。"
- **Immunization gaps:** Check age-appropriate immunizations. Flag missing or expired ones (e.g., flu shot >1 year, Tdap >10 years).
  **免疫接种缺口：** 检查与年龄相适的疫苗接种。标记缺失或过期的项目（如流感疫苗超过 1 年、Tdap 超过 10 年）。
- **Screening gaps:** Based on age, sex, and conditions — flag missing screenings (mammogram, colonoscopy, eye exam for diabetics, etc.).
  **筛查缺口：** 基于年龄、性别与病史——标记缺失的筛查（乳腺钼靶、结肠镜、糖尿病患者的眼科检查等）。

### 2. Condition-Medication-Lab Cross-Reference / 病史-用药-化验交叉参照
Connect the dots across data types:
跨数据类型建立关联：
- **Conditions without expected medications:** e.g., hypertension (elevated BP) without antihypertensives listed.
  **有病史但无预期用药：** 例如列有高血压（血压升高）却未列出降压药。
- **Medications without monitoring labs:** e.g., metformin without recent A1C, statins without recent lipid panel.
  **有用药但缺监测化验：** 例如服用二甲双胍但近期无 A1C，服用他汀但近期无血脂组合。
- **Lab trends suggesting unmanaged conditions:** e.g., consistently elevated fasting glucose without a diabetes diagnosis.
  **提示疾病未受管理的化验趋势：** 例如空腹血糖持续升高但没有糖尿病诊断。
- **Vitals that contradict treatment goals:** e.g., BP 149/70 despite being on antihypertensives → possible medication adjustment needed.
  **与治疗目标相悖的生命体征：** 例如已服用降压药但血压仍为 149/70 → 可能需要调整用药。

### 3. Contextual Health Guidance / 情境化健康指导
Tailor advice to the user's actual health profile:
结合用户的真实健康档案定制建议：
- **Diet plans:** Account for conditions (PCOS → insulin-sensitive diet, hypertension → low sodium), allergies (avoid allergens), and medications (e.g., metformin → monitor B12, warfarin → consistent vitamin K).
  **膳食计划：** 考虑病史（多囊卵巢综合征 → 胰岛素敏感饮食；高血压 → 低钠）、过敏（避开过敏原）与用药（如二甲双胍 → 监测 B12；华法林 → 保持维生素 K 摄入稳定）。
- **Exercise plans:** Account for cardiac conditions (graded activity, heart rate zones), musculoskeletal issues, and current fitness level from vitals.
  **锻炼计划：** 考虑心脏疾病（分级活动、心率区间）、肌肉骨骼问题，以及由生命体征反映的当前体能水平。
- **Visit prep:** Summarize what the doctor will likely want to discuss: overdue labs, unmanaged vitals, medication reviews, symptom follow-ups.
  **就诊准备：** 总结医生可能想讨论的内容：逾期的化验、未受控的生命体征、用药复查、症状随访。

### 4. Risk Factor Aggregation / 风险因素聚合
When multiple risk factors cluster, highlight the combined picture:
当多个风险因素聚集时，突出其组合图景：
- Example: PCOS + elevated BP + no recent lipid panel + no PCP visit in 3 years → cardiovascular risk profile needs attention.
  示例：多囊卵巢综合征 + 血压升高 + 近期无血脂组合 + 3 年内无全科就诊 → 心血管风险画像需要关注。
- Don't alarm — frame as "areas worth discussing with your doctor."
  不要制造恐慌——将其表述为"值得与医生讨论的方面"。

## Handling Slow or Backgrounded Calls / 处理缓慢或被后台化的调用

Some calls (especially `get_health_summary`) can run long enough that the `exec` tool **backgrounds** them and returns a process handle instead of the result, e.g.:

某些调用（尤其是 `get_health_summary`）可能运行得足够久，以致 `exec` 工具将其**后台化**并返回进程句柄而非结果，例如：

```json
{ "sessionId": "proc_abc123", "status": "running" }
```

This is **not** the answer and contains **no health data**. If you see `"status": "running"` (or any process handle without a completed result):

这**不是**答案，也**不含任何健康数据**。若看到 `"status": "running"`（或任何未带完成结果的进程句柄）：
1. **Poll the backgrounded process to completion** before answering — re-check it until its status is `completed`, then read its actual `stdout`.
   回答之前**轮询该后台进程直至完成**——反复检查直到其状态变为 `completed`，再读取其实际的 `stdout`。
2. If it has not completed after a reasonable wait, **fall back to the per-category tools** (`get_conditions`, `get_medications`, `get_labs`, `get_vitals`, …), which return data reliably.
   若合理等待后仍未完成，**回退到各分类工具**（`get_conditions`、`get_medications`、`get_labs`、`get_vitals` 等），它们能可靠地返回数据。
3. **Never** treat a `running`/handle response as if it were the user's data, and never infer or invent values from it.
   **绝不**把 `running`/句柄响应当作用户的数据，也绝不据此推断或编造数值。

## No Real Data → Never Fabricate (highest-priority safety rule) / 无真实数据 → 绝不编造（最高优先级安全规则）

A tool call that does **not** return real records is **never** a license to invent one. In dogfooding, the most serious failures were turns that failed `no-harmful-misinformation` / `no-hallucinated-medical-facts` because the model produced specific clinical content (lab values, vitals, diagnoses, a PET/CT readout) when **no real data was returned**. Treat all four "no real data" states the same way — state plainly that the data was not available, then offer a concrete next step. Do **not** substitute plausible-sounding values, ranges, or interpretations.

一次**未**返回真实记录的工具调用**绝不**是编造记录的许可。在内部试用（dogfooding）中，最严重的失败是那些未通过 `no-harmful-misinformation` / `no-hallucinated-medical-facts` 的对话轮次：模型在**没有返回真实数据**的情况下生成了具体的临床内容（化验值、生命体征、诊断、PET/CT 读片结果）。对全部四种"无真实数据"状态一视同仁——明确说明数据不可得，然后提供具体的下一步。**不要**用听起来合理的数值、区间或解释来替代。

| State | What the tool returned | Required response |
|---|---|---|
| **Empty** | call completed, no records | "I don't see any [labs/medications/etc.] on file. They may not be documented in your connected providers, or may live in a system not linked to HealthEx." |
| **Placeholder** | "records currently being retrieved / available shortly" | "Your records are still syncing from your providers. Try the per-category tools now for anything already available; if still empty, tell the user their records are syncing and to check back in a few minutes. Never answer from the placeholder, and don't promise to retry on your own." |
| **Still processing** | a process handle, e.g. `{ "sessionId": …, "status": "running" }` | Poll to completion (see "Handling Slow or Backgrounded Calls"), or fall back to the per-category tools. Never treat the handle as data. |
| **Failure / error** | tool errored, non-zero exit, "not connected" | Report that the call failed and suggest a retry or reconnect. Do not answer the clinical question from memory or assumption. |

| 状态 | 工具返回的内容 | 要求的响应 |
|---|---|---|
| **空** | 调用完成，无记录 | "我没有看到任何在档的[化验/用药/等]记录。它们可能未记录在你已连接的服务商处，也可能存放在未与 HealthEx 关联的系统中。" |
| **占位响应** | "records currently being retrieved / available shortly"（记录正在获取/即将可用） | "你的记录仍在从各服务商同步。现在可先用各分类工具查看已可用的内容；若仍为空，告诉用户记录正在同步，请在几分钟后回来查看。绝不基于占位响应作答，也不要承诺自行重试。" |
| **仍在处理** | 一个进程句柄，如 `{ "sessionId": …, "status": "running" }` | 轮询至完成（见"处理缓慢或被后台化的调用"），或回退到各分类工具。绝不把句柄当作数据。 |
| **失败 / 错误** | 工具报错、非零退出、"not connected"（未连接） | 报告调用失败并建议重试或重新连接。不得凭记忆或假设回答该临床问题。 |

【评论】该安全规则直接以评估项名称（`no-harmful-misinformation` 等）为校准依据，说明约束来自对模型编造医疗信息这一失败模式的实测。

When data **is** returned but is sparse, handle gracefully:
当数据**有**返回但内容稀疏时，妥善处理：
- **Single data source:** "This data comes from [provider name]. Records from other providers may not be included."
  **单一数据来源：** "该数据来自[服务商名称]。其他服务商的记录可能未包含在内。"
- **Missing categories:** acknowledge the gap and suggest the user check with their provider or connect additional health systems through HealthEx.
  **缺失的类别：** 承认该缺口，并建议用户向其医疗服务机构核实，或通过 HealthEx 连接更多健康系统。

## Pagination / 分页
MCP responses may return partial data. Check every response for a `Pagination Info` section. Neither marker there tells you the patient's record has ended:
MCP 响应可能只返回部分数据。检查每个响应中的 `Pagination Info` 部分。那里的任何一个标记都不能说明患者的记录已到尽头：
- `More data available: Yes` — the range you asked for has not come back in full. Call again with the `beforeDate` and `years` the response gives you.
  `More data available: Yes` —— 你请求的范围尚未完整返回。用响应给出的 `beforeDate` 与 `years` 再次调用。
- `Requested N-year window: Fully covered` — **only** that the range you asked for was satisfied. Your `years` budget shrinks as you page back, so every chain reaches this eventually. It is not a signal that there is nothing older.
  `Requested N-year window: Fully covered` —— **仅**表示你请求的范围已被满足。随着向前翻页，`years` 预算会不断缩减，因此每条分页链最终都会到达该状态。它并不表示没有更早的数据。

**The only end-of-record signal is a window that comes back with no records in it.**

**记录结束的唯一信号，是某个窗口返回时其中没有任何记录。**

What to do next depends on what the user asked for:
下一步取决于用户要求的是什么：
- **They named a time range** ("labs from the last two years"): pass it as `years` and paginate until `Fully covered`, or until you have made 10 calls, whichever comes first. If `Fully covered` came back, that answer is complete for what they asked, so say so; if you hit the cap first, report it the same way as below.
  **用户指定了时间范围**（"最近两年的化验"）：将其作为 `years` 传入并翻页，直到出现 `Fully covered` 或累计调用满 10 次，以先到者为准。若返回了 `Fully covered`，则该回答就其要求而言已经完整，应如实说明；若先触及调用上限，则按与下文相同的方式报告。
- **They did not name a range** ("what conditions do I have?", "have I ever had X?"): keep calling with an earlier `beforeDate` and a fresh `years` budget until **two windows in a row come back with no records**, or you have made 10 calls. Those are the only two reasons to stop. **One empty window is not the end of the record** — the most recent window is often empty, and a gap in care is ordinary, so keep going past a single empty one. Do not stop because `Fully covered` appeared, and do not stop because what you have looks like enough.
  **用户未指定范围**（"我有哪些病史？"、"我得过 X 吗？"）：持续用更早的 `beforeDate` 和新的 `years` 预算调用，直到**连续两个窗口都无记录**，或累计调用满 10 次。这是仅有的两个停止理由。**一个空窗口不代表记录结束**——最近的窗口常常为空，照护出现空档也很常见，因此应越过单个空窗口继续。不要因为出现了 `Fully covered` 就停止，也不要因为已获取的内容看起来够了就停止。

Then report what happened:
然后报告实际发生的情况：
- **Two empty windows in a row** — stop there, and describe what you actually covered: "I checked back to `<date>` and found nothing before `<date>`". That is good evidence the record has ended, but it is not proof, so do not call the result complete.
  **连续两个空窗口** —— 到此停止，并说明你实际覆盖的范围："我核查至 `<date>`，在此之前没有发现记录。" 这是记录已结束的良好证据，但并非证明，因此不要称结果为完整。
- **You stopped at the 10-call cap** — say so plainly: "I pulled records back to `<date>`; older records may exist and I have not retrieved them yet." Offer to continue. Never call the result complete, full, or "everything on file".
  **在 10 次调用上限处停止** —— 明确说明："我已拉取到 `<date>` 为止的记录；可能存在更早的记录，我尚未检索。" 主动提出可以继续。绝不要称结果为完整、齐全或"全部在档"。

**Where you stopped without exhausting the record — at the cap, or at an empty window that may be a gap — do not treat what you did not fetch as absent.** In that case only: do not say a diagnosis, medication or result is missing, do not call it a documentation gap, and do not advise the user to raise it with their provider. Care-gap detection above still applies normally to the records you did retrieve.

**凡在未穷尽记录之处停止——无论是在调用上限处，还是在一个可能只是空档的空窗口处——都不要把未获取的内容当作不存在。** 仅在此情形下：不要说某项诊断、用药或结果是缺失的，不要称之为记录缺口，也不要建议用户向其医疗服务机构提出。上述照护缺口检测对已检索到的记录仍照常适用。

【评论】分页规则把"未检索到"与"不存在"严格区分，防止在数据不完整时输出误导性的照护缺口结论。

## Operating Rules / 运行规则
1. Complete the Connection Guard before any data access.
   在任何数据访问之前完成连接守卫流程。
2. Pull the per-category tools relevant to the question; treat `get_health_summary` as optional and unreliable (see "Handling Slow or Backgrounded Calls"). When deeper or category-specific data is needed, run `healthex mcp-list` to discover the right tool and its parameter schema.
   拉取与问题相关的各分类工具；将 `get_health_summary` 视为可选且不可靠（见"处理缓慢或被后台化的调用"）。当需要更深入或特定类别的数据时，运行 `healthex mcp-list` 来发现合适的工具及其参数 schema。
3. **Never state a specific clinical value — lab number, vital, medication, diagnosis, dose, or date — that did not appear verbatim in a tool's completed output.** If a call returns empty results, a `running` handle, a placeholder ("records being retrieved"), or a failure/error, follow "No Real Data → Never Fabricate": state plainly that the data was not available, then offer a concrete next step. Never fill the gap with plausible-sounding values, ranges, or interpretations. Never answer clinical questions from memory or assumption when the tool did not return real data. This is the highest-priority safety rule — fabricating medical facts is the most serious failure mode of this skill (`no-harmful-misinformation` / `no-hallucinated-medical-facts`).
   **绝不陈述任何未在工具已完成输出中逐字出现过的具体临床数值——化验值、生命体征、用药、诊断、剂量或日期。** 若调用返回空结果、`running` 句柄、占位响应（"记录正在获取"）或失败/错误，遵循"无真实数据 → 绝不编造"：明确说明数据不可得，然后提供具体的下一步。绝不要用听起来合理的数值、区间或解释填补空缺。当工具未返回真实数据时，绝不凭记忆或假设回答临床问题。这是最高优先级的安全规则——编造医学事实是本技能最严重的失败模式（`no-harmful-misinformation` / `no-hallucinated-medical-facts`）。
4. Never make medical diagnoses, treatment recommendations, or clinical interpretations. Present data factually and suggest the user consult their healthcare provider.
   绝不做医学诊断、治疗建议或临床解读。以事实方式呈现数据，并建议用户咨询其医疗服务提供者。
5. Handle MCP errors gracefully — if a tool call fails, report the error and suggest the user try again or check their HealthEx account.
   妥善处理 MCP 错误——若工具调用失败，报告错误并建议用户重试或检查其 HealthEx 账户。
6. If `mcp-call` returns a 401 error after auto-refresh, the token is expired or revoked. Re-run the Connection Guard.
   若 `mcp-call` 在自动刷新后仍返回 401 错误，说明令牌已过期或被撤销。重新运行连接守卫。
7. Never print `access_token` or `refresh_token` values.
   绝不打印 `access_token` 或 `refresh_token` 的值。
8. Health data is sensitive — do not store or retain it beyond the current request.
   健康数据属敏感信息——不得存储或保留超出当前请求所需的部分。
9. When surfacing insights, always cite the data source: "Based on your HealthEx records..." — never present inferences as established medical facts.
   呈现洞见时始终注明数据来源："基于你的 HealthEx 记录……"——绝不把推断当作已确立的医学事实来陈述。
10. For any finding that suggests a care gap or risk, include a concrete next step the user can take.
    对任何提示照护缺口或风险的发现，附上用户可以采取的具体下一步。
11. Record reads preserve raw clinical timestamps and add semantic UTC and
    user-local forms when the source supplies a true instant. Date-only values
    remain dates.
    记录读取保留原始临床时间戳，并在数据源提供真实时间点时补充语义化的 UTC 与用户本地时间形式。仅含日期的值仍保持为日期。
