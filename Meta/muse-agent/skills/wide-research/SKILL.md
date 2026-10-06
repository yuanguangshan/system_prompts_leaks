---
name: "wide_research"
description: "Use when the user needs broad parallel research across many independent inputs with a shared output schema."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Wide Research / 广域研究

## Purpose / 目的
Coordinate one manager subagent that fans out the same research task across many independent inputs and returns a single normalized result set.

协调一个管理者（manager）子智能体，把同一研究任务扇出到多个相互独立的输入上，并返回一个统一的规范化结果集。

## When to Use / 何时使用
- The task can be split into many independent subtasks (for example, one company/person/topic per input).
  任务可以拆分为多个相互独立的子任务（例如每个输入对应一家公司/一个人物/一个主题）。
- Each subtask should return the same structured fields.
  每个子任务应返回相同的结构化字段。
- The user asks for "wide research", "parallel research", or high-volume lookup/screening.
  用户要求"wide research"（广域研究）、"parallel research"（并行研究）或大批量查询/筛选。

Do not use this workflow for single-item tasks or when subtasks depend on each other.

单项任务或子任务之间存在依赖时，不要使用此工作流。

## Tooling / 工具
Use exactly one manager subagent for the overall operation.

整个操作只使用一个管理者子智能体。

Manager input contract:

管理者输入契约：

- `operation_brief`: one concise sentence.
  `operation_brief`：一句简洁的说明。
- `inputs`: one independent item per element.
  `inputs`：每个元素对应一个独立条目。
- `output_schema`: required fields and allowed types.
  `output_schema`：必需字段及允许的类型。
- `worker_prompt_template`: per-input instructions.
  `worker_prompt_template`：针对每个输入的指令。
- `completion_format`: exact JSON object shape for the manager's final response.
  `completion_format`：管理者最终响应的精确 JSON 对象结构。

Manager output contract:

管理者输出契约：

- `total`
  `total`（总数）
- `success_count`
  `success_count`（成功数）
- `failure_count`
  `failure_count`（失败数）
- `results`
  `results`（结果列表）
- `failures`
  `failures`（失败列表）
- `notes`
  `notes`（备注）

## Operating Rules / 操作规则
1. Deduplicate and normalize the input list before spawning the manager.
   在启动管理者之前，先对输入列表去重并规范化。
2. Keep the output schema minimal and explicit; avoid optional or free-form fields unless the user asked for them.
   输出 schema 保持最小且明确；除非用户要求，避免可选字段或自由格式字段。
3. Use one manager coordinator for one user goal; do not fan out multiple sibling root-level subagents.
   一个用户目标只使用一个管理者协调器；不要扇出多个平行的根级子智能体。
4. Instruct the manager to assign one worker per input and keep each worker scoped to its own item.
   指示管理者为每个输入分配一个 worker，并让每个 worker 只处理自己的条目。
5. If completeness matters and some inputs fail, retry only the failed inputs, once, when feasible.
   如果完整性很重要且部分输入失败，在可行时只对失败的输入重试一次。
6. Reply to the user immediately that wide research has started; do not block on completion.
   广域研究开始后立即回复用户；不要阻塞等待其完成。
7. In the final user-facing output, always report coverage (`success_count/total`) and unresolved gaps.
   在最终面向用户的输出中，始终报告覆盖率（`success_count/total`）和未解决的缺口。

【评论】"一个目标只用一个管理者、不扇出多个根级子智能体"是对并行规模的约束，用于控制成本与编排复杂度；"开始即回复、不阻塞等待完成"则是降低感知延迟的交互设计。
