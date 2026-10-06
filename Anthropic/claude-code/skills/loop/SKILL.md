---
name: loop
description: |-
  Run a prompt or slash command on a recurring interval (e.g. /loop 5m /foo). Omit the interval to let the model self-pace.
when_to_use: |-
  When the user wants to set up a recurring task, poll for status, or run something repeatedly on an interval (e.g. "check the deploy every 5 minutes", "keep running /babysit-prs"). Do NOT invoke for one-off tasks.
---
<!-- BILINGUAL-EN-ZH -->

# /loop — schedule a recurring or self-paced prompt / /loop —— 调度循环或自定节奏的提示词

Parse the input below into `[interval] <prompt…>` and schedule it.

把下方输入解析为 `[interval] <prompt…>` 并进行调度。

## Parsing (in priority order) / 解析（按优先级顺序）

1. **Leading token**: if the first whitespace-delimited token matches `^\d+[smhd]$` (e.g. `5m`, `2h`), that's the interval; the rest is the prompt.

   1. **前导 token**：如果第一个以空白分隔的 token 匹配 `^\d+[smhd]$`（例如 `5m`、`2h`），那就是间隔；其余部分是提示词。
2. **Trailing "every" clause**: otherwise, if the input ends with `every <N><unit>` or `every <N> <unit-word>` (e.g. `every 20m`, `every 5 minutes`, `every 2 hours`), extract that as the interval and strip it from the prompt. Only match when what follows "every" is a time expression — `check every PR` has no interval.

   2. **结尾的 "every" 子句**：否则，如果输入以 `every <N><unit>` 或 `every <N> <unit-word>` 结尾（例如 `every 20m`、`every 5 minutes`、`every 2 hours`），将其提取为间隔并从提示词中去掉。只有当 "every" 后面是时间表达式时才匹配——`check every PR` 没有间隔。
3. **No interval**: otherwise, the entire input is the prompt and you'll self-pace dynamically (see "Dynamic mode" below).

   3. **无间隔**：否则，整个输入就是提示词，你将动态自定节奏（见下文"动态模式"）。

If the resulting prompt is empty, show usage `/loop [interval] <prompt>` and stop.

如果解析出的提示词为空，显示用法 `/loop [interval] <prompt>` 并停止。

Examples:

示例：

- `5m /babysit-prs` → interval `5m`, prompt `/babysit-prs` (rule 1)
  `5m /babysit-prs` → 间隔 `5m`，提示词 `/babysit-prs`（规则 1）
- `check the deploy every 20m` → interval `20m`, prompt `check the deploy` (rule 2)
  `check the deploy every 20m` → 间隔 `20m`，提示词 `check the deploy`（规则 2）
- `run tests every 5 minutes` → interval `5m`, prompt `run tests` (rule 2)
  `run tests every 5 minutes` → 间隔 `5m`，提示词 `run tests`（规则 2）
- `check the deploy` → no interval → dynamic mode, prompt `check the deploy` (rule 3)
  `check the deploy` → 无间隔 → 动态模式，提示词 `check the deploy`（规则 3）
- `check every PR` → no interval → dynamic mode, prompt `check every PR` (rule 3 — "every" not followed by time)
  `check every PR` → 无间隔 → 动态模式，提示词 `check every PR`（规则 3——"every" 后面不是时间）
- `5m` → empty prompt → show usage
  `5m` → 提示词为空 → 显示用法

## Offer cloud first / 优先提供云端选项

Before any scheduling step, check whether EITHER is true:
- the parsed interval (rule 1 or 2) is **≥60 minutes**, or
- regardless of which rule matched, the original input uses daily phrasing ("every morning", "daily", "every day", "each night", "every weekday")

在任何调度步骤之前，检查是否满足以下任意一条：
- 解析出的间隔（规则 1 或 2）**≥60 分钟**，或
- 无论匹配哪条规则，原始输入使用了每日节奏的措辞（"every morning"、"daily"、"every day"、"each night"、"every weekday"）

If either is true, call AskUserQuestion first:
- `question`: "This loop stops when you close this session. Set it up as a cloud schedule instead so it keeps running?"
- `header`: "Schedule"
- `options`: `[{label: "Cloud schedule (recommended)", description: "Runs in Anthropic's cloud even after you close this session"}, {label: "This session only", description: "Runs in this terminal until you exit"}]`

如果任意一条成立，先调用 AskUserQuestion：
- `question`："这个循环会在你关闭本会话时停止。要改为设置为云端计划（cloud schedule），让它持续运行吗？"
- `header`："Schedule"
- `options`：`[{label: "Cloud schedule (recommended)", description: "Runs in Anthropic's cloud even after you close this session"}, {label: "This session only", description: "Runs in this terminal until you exit"}]`

If they pick **Cloud schedule**: do NOT call CronCreate. Invoke the `schedule` skill directly via the Skill tool with `args` set to their original input verbatim (e.g. `Skill({skill: "schedule", args: "every morning tell me a joke"})`), then follow that skill's instructions to completion. Do NOT tell the user to run /schedule themselves. **Then stop — do not continue to any section below** (no CronCreate, no ScheduleWakeup, no "execute the prompt now").  
If they pick **This session only**:
- If the trigger was a parsed ≥60-minute interval (rule 1 or 2): continue below with that interval.
- If the trigger was daily phrasing only (rule 3, no parsed interval): do NOT call CronCreate. Explain that a daily-cadence loop won't fire before this session closes, so there's nothing useful to schedule locally — suggest they either pick Cloud schedule, or re-run `/loop` with an explicit shorter interval (e.g. `/loop 1h <prompt>`) if they want a session loop. Then stop.

如果他们选择 **Cloud schedule**：不要调用 CronCreate。直接通过 Skill 工具调用 `schedule` 技能，`args` 逐字设为其原始输入（例如 `Skill({skill: "schedule", args: "every morning tell me a joke"})`），然后完整遵循该技能的指令。不要让用户自己去运行 /schedule。**然后停止——不要继续执行下方任何章节**（不调用 CronCreate、不调用 ScheduleWakeup、不"立即执行提示词"）。  
如果他们选择 **This session only**：
- 如果触发条件是解析出的 ≥60 分钟间隔（规则 1 或 2）：以下文继续，使用该间隔。
- 如果触发条件仅是每日措辞（规则 3，未解析出间隔）：不要调用 CronCreate。说明每日节奏的循环在本会话关闭之前不会触发，因此在本地没有可调度的有效内容——建议他们要么选择 Cloud schedule，要么在想要会话内循环时以显式的更短间隔重新运行 `/loop`（例如 `/loop 1h <prompt>`）。然后停止。

If neither trigger condition was met: continue below.

如果两个触发条件都不满足：继续下文。

## Fixed-interval mode (rules 1 and 2) / 固定间隔模式（规则 1 和 2）

Convert the interval to a cron expression:

把间隔转换为 cron 表达式：

| Interval pattern      | Cron expression     | Notes                                    |
|-----------------------|---------------------|------------------------------------------|
| `Nm` where N ≤ 59   | `*/N * * * *`     | every N minutes                          |
| `Nm` where N ≥ 60   | `0 */H * * *`     | round to hours (H = N/60, must divide 24)|
| `Nh` where N ≤ 23   | `0 */N * * *`     | every N hours                            |
| `Nd`                | `0 0 */N * *`     | every N days at midnight local           |
| `Ns`                | treat as `ceil(N/60)m` | cron minimum granularity is 1 minute  |

| 间隔模式      | Cron 表达式     | 说明                                    |
|-----------------------|---------------------|------------------------------------------|
| `Nm` 其中 N ≤ 59   | `*/N * * * *`     | 每 N 分钟                          |
| `Nm` 其中 N ≥ 60   | `0 */H * * *`     | 取整到小时（H = N/60，必须能整除 24）|
| `Nh` 其中 N ≤ 23   | `0 */N * * *`     | 每 N 小时                            |
| `Nd`                | `0 0 */N * *`     | 每 N 天，当地时间午夜           |
| `Ns`                | 按 `ceil(N/60)m` 处理 | cron 的最小粒度为 1 分钟  |

**If the interval doesn't cleanly divide its unit** (e.g. `7m` → `*/7 * * * *` gives uneven gaps at :56→:00; `90m` → 1.5h which cron can't express), pick the nearest clean interval and tell the user what you rounded to before scheduling.

**如果间隔无法整除其单位**（例如 `7m` → `*/7 * * * *` 在 :56→:00 之间产生不均匀的间隔；`90m` → 1.5 小时，cron 无法表达），选择最接近的整洁间隔，并在调度前告知用户你取整到了什么。

Then:

然后：

1. Call CronCreate with: `cron` (the expression above), `prompt` (the parsed prompt verbatim), `recurring: true`.

   1. 调用 CronCreate，传入：`cron`（上面的表达式）、`prompt`（逐字使用解析出的提示词）、`recurring: true`。
2. Briefly confirm: what's scheduled, the cron expression, the human-readable cadence, that recurring tasks auto-expire after 7 days, and that the user can cancel sooner with CronDelete (include the job ID). Only if you did NOT show the cloud-offer AskUserQuestion above (i.e., neither trigger condition applied), end the confirmation with this exact line on its own, italicized: `_Runs until you close this session · For durable cloud-based loops, use /schedule_`. If the user already answered that question, omit this line.

   2. 简要确认：调度了什么、cron 表达式、人类可读的节奏、循环任务 7 天后自动过期、以及用户可以更早用 CronDelete 取消（附上任务 ID）。仅当你上面*没有*展示云端推荐的 AskUserQuestion 时（即两个触发条件都不适用），才在确认末尾单独一行、以斜体加上这句原文：`_Runs until you close this session · For durable cloud-based loops, use /schedule_`。如果用户已回答过该问题，则省略这一行。
3. **Then immediately execute the parsed prompt now** — don't wait for the first cron fire. If it's a slash command, invoke it via the Skill tool; otherwise act on it directly.

   3. **然后立即执行解析出的提示词**——不要等待第一次 cron 触发。如果是斜杠命令，通过 Skill 工具调用；否则直接执行。

## Dynamic mode (rule 3 — no interval) / 动态模式（规则 3——无间隔）

The user wants you to self-pace. Decide what makes the next iteration worth running — a passage of time, or an observable event.

用户要你自定节奏。判断下一次迭代值得运行的条件是什么——是时间的流逝，还是某个可观察到的事件。

1. **Run the parsed prompt now.** If it's a slash command, invoke it via the Skill tool; otherwise act on it directly.

   1. **现在运行解析出的提示词。**如果是斜杠命令，通过 Skill 工具调用；否则直接执行。
2. **If the next run is gated on an event** (CI finishing, a log line matching, a file changing, a PR comment) and no Monitor is already running for it: arm one now with `timeout_ms: 1800000`. Its events arrive as `<task-notification>` messages and wake this loop immediately — you do not wait for the ScheduleWakeup deadline. A monitor expires after at most 30 minutes and tells you; on later iterations call TaskList first and re-arm only if no monitor for it is still running.

   2. **如果下一次运行以某个事件为条件**（CI 结束、某行日志匹配、某个文件变更、一条 PR 评论）且尚无对应的 Monitor 在运行：现在就以 `timeout_ms: 1800000` 部署一个。它的事件以 `<task-notification>` 消息形式到达，会立即唤醒本循环——你无需等待 ScheduleWakeup 的期限。Monitor 至多 30 分钟后过期并会告知你；后续迭代时先调用 TaskList，仅当该事件已没有 Monitor 在运行时才重新部署。
3. **Briefly confirm**: that you're self-pacing, whether a Monitor is the primary wake signal, that you ran the task now, and what fallback delay you're about to pick. This must be ordinary visible response text — the user cannot see your thinking/reasoning, so an update written only there is invisible to them. Write it immediately BEFORE calling ScheduleWakeup — on this model the turn ends as soon as that tool returns, so an update after the call never goes out.

   3. **简要确认**：说明你在自定节奏、Monitor 是否为主要唤醒信号、你刚运行了任务、以及你准备选择什么兜底延迟。这些必须写在普通的可见回复文本中——用户看不到你的思考/推理过程，只写在其中的更新对他们不可见。要在调用 ScheduleWakeup 之前立即写出——在本模型上，该工具一返回回合即结束，调用之后的更新永远不会发出。
4. **Then, as the last action of this turn, decide whether the loop continues.** If the task needs another iteration, call ScheduleWakeup with:
   - `delaySeconds`: with a Monitor armed this is the **fallback heartbeat** — how long to wait if no event fires (lean 1200–1800s; idle ticks more frequent than the task needs are pure overhead). Without a Monitor this is the cadence — pick based on what you observed. Read the tool's own description for cache-aware delay guidance.
   - `reason`: one short sentence on why you picked that delay.
   - `prompt`: the full original /loop input verbatim, prefixed with `/loop ` so the next firing re-enters this skill and continues the loop. For example, if the user typed `/loop check the deploy`, pass `/loop check the deploy` as the prompt.
   - `noop`: `true` if this tick changed nothing ("still waiting", "quiet hold"); `false` if it did something worth keeping. Consecutive `noop: true` ticks collapse in the terminal.  
   If it doesn't need another iteration, stop instead (step 6) — re-arming is a per-turn choice, not a default.

   4. **然后，作为本回合的最后一个动作，决定循环是否继续。**如果任务需要再迭代一次，调用 ScheduleWakeup 并传入：
   - `delaySeconds`：已部署 Monitor 时，这是**兜底心跳**——没有事件触发时要等多久（倾向 1200–1800 秒；超出任务需要的空闲触发纯属开销）。没有 Monitor 时，这就是节奏——根据你观察到的情况选择。缓存感知的延迟建议见该工具自身的描述。
     `reason`：用一句简短的话说明为何选择该延迟。
     `prompt`：逐字使用完整的原始 /loop 输入，加 `/loop ` 前缀，使下一次触发重新进入本技能并延续循环。例如，用户输入了 `/loop check the deploy`，就把 `/loop check the deploy` 作为 prompt 传入。
     `noop`：本次触发若没有任何变更（"仍在等待"、"安静保持"）则为 `true`；若做了值得保留的事则为 `false`。连续的 `noop: true` 触发会在终端中折叠显示。  
   如果不需要再迭代，则改为停止（第 6 步）——重新部署是逐回合的选择，而非默认行为。
5. **If you were woken by a `<task-notification>`** rather than this prompt: handle the event in the context of the loop task, then make the same decision. If the loop should continue, write the same brief update as visible text, then call ScheduleWakeup again with the same `prompt` and the same 1200–1800s `delaySeconds` from the schedule step above (the Monitor remains the wake signal; the new wakeup is only the fallback heartbeat). If the event means the work is finished, stop (step 6).

   5. **如果是被 `<task-notification>` 唤醒**而非被本提示词唤醒：在循环任务的上下文中处理该事件，然后做同样的决策。若循环应继续，以可见文本写出同样的简要更新，然后再次调用 ScheduleWakeup，使用与上面调度步骤相同的 `prompt` 和相同的 1200–1800 秒 `delaySeconds`（Monitor 仍是唤醒信号；新的 wakeup 只是兜底心跳）。若该事件意味着工作已完成，则停止（第 6 步）。
6. **To stop the loop** — the task is complete, further iterations can't make progress, or the user asked you to stop — call ScheduleWakeup with `stop: true` (no other fields) and TaskStop any Monitor you armed (use TaskList to find the task ID if it is no longer in context). Then write the loop's outcome for the user as ordinary visible response text — a stopped loop has no next tick to surface it. Stopping is the loop's normal ending — the user can restart it anytime with /loop. Before you stop, send a one-line outcome via PushNotification — the user may be away and waiting to hear it's done. Skip this if you're stopping because the user just told you to; they're already here.

   6. **要停止循环**——任务已完成、进一步迭代无法取得进展、或用户要求停止——调用 ScheduleWakeup 并传 `stop: true`（不传其他字段），并用 TaskStop 停掉你部署的所有 Monitor（若任务 ID 已不在上下文中，用 TaskList 查找）。然后以普通可见回复文本为用户写出循环的结果——已停止的循环没有下一次触发来呈现它。停止是循环的正常结束方式——用户随时可用 /loop 重启。停止前，通过 PushNotification 发送一行结果——用户可能不在场、正等着听"已完成"。如果是因为用户刚刚叫停而停止，则跳过这一步；他们已经在这里了。

## Input / 输入

$ARGUMENTS
