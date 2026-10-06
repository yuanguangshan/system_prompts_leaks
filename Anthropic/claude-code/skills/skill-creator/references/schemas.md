<!-- BILINGUAL-EN-ZH -->
# JSON Schemas / JSON 模式（Schema）

This document defines the JSON schemas used by skill-creator.

本文档定义 skill-creator 所使用的 JSON 模式（schema）。

---

## evals.json / evals.json（评估定义）

Defines the evals for a skill. Located at `evals/evals.json` within the skill directory.

定义某项技能的评估用例（evals）。位于技能目录内的 `evals/evals.json`。

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's example prompt",
      "expected_output": "Description of expected result",
      "files": ["evals/files/sample1.pdf"],
      "expectations": [
        "The output includes X",
        "The skill used script Y"
      ]
    }
  ]
}
```

**Fields:**

**字段：**

- `skill_name`: Name matching the skill's frontmatter
  `skill_name`：与技能 frontmatter 匹配的名称
- `evals[].id`: Unique integer identifier
  `evals[].id`：唯一的整数标识符
- `evals[].prompt`: The task to execute
  `evals[].prompt`：要执行的任务
- `evals[].expected_output`: Human-readable description of success
  `evals[].expected_output`：对成功结果的人类可读描述
- `evals[].files`: Optional list of input file paths (relative to skill root)
  `evals[].files`：可选的输入文件路径列表（相对于技能根目录）
- `evals[].expectations`: List of verifiable statements
  `evals[].expectations`：可验证断言的列表

---

## history.json / history.json（历史记录）

Tracks version progression in Improve mode. Located at workspace root.

在改进（Improve）模式下追踪版本演进。位于工作区根目录。

```json
{
  "started_at": "2026-01-15T10:30:00Z",
  "skill_name": "pdf",
  "current_best": "v2",
  "iterations": [
    {
      "version": "v0",
      "parent": null,
      "expectation_pass_rate": 0.65,
      "grading_result": "baseline",
      "is_current_best": false
    },
    {
      "version": "v1",
      "parent": "v0",
      "expectation_pass_rate": 0.75,
      "grading_result": "won",
      "is_current_best": false
    },
    {
      "version": "v2",
      "parent": "v1",
      "expectation_pass_rate": 0.85,
      "grading_result": "won",
      "is_current_best": true
    }
  ]
}
```

**Fields:**

**字段：**

- `started_at`: ISO timestamp of when improvement started
  `started_at`：改进开始时间的 ISO 时间戳
- `skill_name`: Name of the skill being improved
  `skill_name`：被改进技能的名称
- `current_best`: Version identifier of the best performer
  `current_best`：表现最佳版本的标识符
- `iterations[].version`: Version identifier (v0, v1, ...)
  `iterations[].version`：版本标识符（v0、v1……）
- `iterations[].parent`: Parent version this was derived from
  `iterations[].parent`：派生出该版本的父版本
- `iterations[].expectation_pass_rate`: Pass rate from grading
  `iterations[].expectation_pass_rate`：评分得出的通过率
- `iterations[].grading_result`: "baseline", "won", "lost", or "tie"
  `iterations[].grading_result`："baseline"、"won"、"lost" 或 "tie"
- `iterations[].is_current_best`: Whether this is the current best version
  `iterations[].is_current_best`：该版本是否为当前最佳版本

---

## grading.json / grading.json（评分结果）

Output from the grader agent. Located at `<run-dir>/grading.json`.

评分代理（grader agent）的输出。位于 `<run-dir>/grading.json`。

```json
{
  "expectations": [
    {
      "text": "The output includes the name 'John Smith'",
      "passed": true,
      "evidence": "Found in transcript Step 3: 'Extracted names: John Smith, Sarah Johnson'"
    },
    {
      "text": "The spreadsheet has a SUM formula in cell B10",
      "passed": false,
      "evidence": "No spreadsheet was created. The output was a text file."
    }
  ],
  "summary": {
    "passed": 2,
    "failed": 1,
    "total": 3,
    "pass_rate": 0.67
  },
  "execution_metrics": {
    "tool_calls": {
      "Read": 5,
      "Write": 2,
      "Bash": 8
    },
    "total_tool_calls": 15,
    "total_steps": 6,
    "errors_encountered": 0,
    "output_chars": 12450,
    "transcript_chars": 3200
  },
  "timing": {
    "executor_duration_seconds": 165.0,
    "grader_duration_seconds": 26.0,
    "total_duration_seconds": 191.0
  },
  "claims": [
    {
      "claim": "The form has 12 fillable fields",
      "type": "factual",
      "verified": true,
      "evidence": "Counted 12 fields in field_info.json"
    }
  ],
  "user_notes_summary": {
    "uncertainties": ["Used 2023 data, may be stale"],
    "needs_review": [],
    "workarounds": ["Fell back to text overlay for non-fillable fields"]
  },
  "eval_feedback": {
    "suggestions": [
      {
        "assertion": "The output includes the name 'John Smith'",
        "reason": "A hallucinated document that mentions the name would also pass"
      }
    ],
    "overall": "Assertions check presence but not correctness."
  }
}
```

**Fields:**

**字段：**

- `expectations[]`: Graded expectations with evidence
  `expectations[]`：带证据的已评分断言
- `summary`: Aggregate pass/fail counts
  `summary`：通过/失败计数汇总
- `execution_metrics`: Tool usage and output size (from executor's metrics.json)
  `execution_metrics`：工具使用与输出规模（来自执行器的 metrics.json）
- `timing`: Wall clock timing (from timing.json)
  `timing`：墙上时钟计时（来自 timing.json）
- `claims`: Extracted and verified claims from the output
  `claims`：从输出中提取并验证的论断
- `user_notes_summary`: Issues flagged by the executor
  `user_notes_summary`：执行器标记的问题
- `eval_feedback`: (optional) Improvement suggestions for the evals, only present when the grader identifies issues worth raising
  `eval_feedback`：（可选）对评估用例的改进建议，仅在评分器发现值得提出的问题时出现

---

## metrics.json / metrics.json（执行指标）

Output from the executor agent. Located at `<run-dir>/outputs/metrics.json`.

执行器代理（executor agent）的输出。位于 `<run-dir>/outputs/metrics.json`。

```json
{
  "tool_calls": {
    "Read": 5,
    "Write": 2,
    "Bash": 8,
    "Edit": 1,
    "Glob": 2,
    "Grep": 0
  },
  "total_tool_calls": 18,
  "total_steps": 6,
  "files_created": ["filled_form.pdf", "field_values.json"],
  "errors_encountered": 0,
  "output_chars": 12450,
  "transcript_chars": 3200
}
```

**Fields:**

**字段：**

- `tool_calls`: Count per tool type
  `tool_calls`：按工具类型统计的调用次数
- `total_tool_calls`: Sum of all tool calls
  `total_tool_calls`：所有工具调用的总和
- `total_steps`: Number of major execution steps
  `total_steps`：主要执行步骤的数量
- `files_created`: List of output files created
  `files_created`：已创建输出文件的列表
- `errors_encountered`: Number of errors during execution
  `errors_encountered`：执行期间遇到的错误数量
- `output_chars`: Total character count of output files
  `output_chars`：输出文件的总字符数
- `transcript_chars`: Character count of transcript
  `transcript_chars`：执行记录（transcript）的字符数

---

## timing.json / timing.json（计时数据）

Wall clock timing for a run. Located at `<run-dir>/timing.json`.

一次运行的墙上时钟计时。位于 `<run-dir>/timing.json`。

**How to capture:** When a subagent task completes, the task notification includes `total_tokens` and `duration_ms`. Save these immediately — they are not persisted anywhere else and cannot be recovered after the fact.

**如何采集：** 子代理任务完成时，任务通知中包含 `total_tokens` 和 `duration_ms`。请立即保存这些值——它们不会持久化到其他任何地方，事后无法恢复。

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3,
  "executor_start": "2026-01-15T10:30:00Z",
  "executor_end": "2026-01-15T10:32:45Z",
  "executor_duration_seconds": 165.0,
  "grader_start": "2026-01-15T10:32:46Z",
  "grader_end": "2026-01-15T10:33:12Z",
  "grader_duration_seconds": 26.0
}
```

---

## benchmark.json / benchmark.json（基准测试结果）

Output from Benchmark mode. Located at `benchmarks/<timestamp>/benchmark.json`.

基准测试（Benchmark）模式的输出。位于 `benchmarks/<timestamp>/benchmark.json`。

```json
{
  "metadata": {
    "skill_name": "pdf",
    "skill_path": "/path/to/pdf",
    "executor_model": "claude-sonnet-4-20250514",
    "analyzer_model": "most-capable-model",
    "timestamp": "2026-01-15T10:30:00Z",
    "evals_run": [1, 2, 3],
    "runs_per_configuration": 3
  },

  "runs": [
    {
      "eval_id": 1,
      "eval_name": "Ocean",
      "configuration": "with_skill",
      "run_number": 1,
      "result": {
        "pass_rate": 0.85,
        "passed": 6,
        "failed": 1,
        "total": 7,
        "time_seconds": 42.5,
        "tokens": 3800,
        "tool_calls": 18,
        "errors": 0
      },
      "expectations": [
        {"text": "...", "passed": true, "evidence": "..."}
      ],
      "notes": [
        "Used 2023 data, may be stale",
        "Fell back to text overlay for non-fillable fields"
      ]
    }
  ],

  "run_summary": {
    "with_skill": {
      "pass_rate": {"mean": 0.85, "stddev": 0.05, "min": 0.80, "max": 0.90},
      "time_seconds": {"mean": 45.0, "stddev": 12.0, "min": 32.0, "max": 58.0},
      "tokens": {"mean": 3800, "stddev": 400, "min": 3200, "max": 4100}
    },
    "without_skill": {
      "pass_rate": {"mean": 0.35, "stddev": 0.08, "min": 0.28, "max": 0.45},
      "time_seconds": {"mean": 32.0, "stddev": 8.0, "min": 24.0, "max": 42.0},
      "tokens": {"mean": 2100, "stddev": 300, "min": 1800, "max": 2500}
    },
    "delta": {
      "pass_rate": "+0.50",
      "time_seconds": "+13.0",
      "tokens": "+1700"
    }
  },

  "notes": [
    "Assertion 'Output is a PDF file' passes 100% in both configurations - may not differentiate skill value",
    "Eval 3 shows high variance (50% ± 40%) - may be flaky or model-dependent",
    "Without-skill runs consistently fail on table extraction expectations",
    "Skill adds 13s average execution time but improves pass rate by 50%"
  ]
}
```

**Fields:**

**字段：**

- `metadata`: Information about the benchmark run
  `metadata`：关于本次基准测试运行的信息
  - `skill_name`: Name of the skill
    `skill_name`：技能名称
  - `timestamp`: When the benchmark was run
    `timestamp`：基准测试运行的时间
  - `evals_run`: List of eval names or IDs
    `evals_run`：评估名称或 ID 的列表
  - `runs_per_configuration`: Number of runs per config (e.g. 3)
    `runs_per_configuration`：每种配置的运行次数（例如 3）
- `runs[]`: Individual run results
  `runs[]`：单次运行的结果
  - `eval_id`: Numeric eval identifier
    `eval_id`：评估的数字标识符
  - `eval_name`: Human-readable eval name (used as section header in the viewer)
    `eval_name`：人类可读的评估名称（在查看器中用作小节标题）
  - `configuration`: Must be `"with_skill"` or `"without_skill"` (the viewer uses this exact string for grouping and color coding)
    `configuration`：必须为 `"with_skill"` 或 `"without_skill"`（查看器用这个精确字符串进行分组与配色）
  - `run_number`: Integer run number (1, 2, 3...)
    `run_number`：整数运行编号（1、2、3……）
  - `result`: Nested object with `pass_rate`, `passed`, `total`, `time_seconds`, `tokens`, `errors`
    `result`：嵌套对象，包含 `pass_rate`、`passed`、`total`、`time_seconds`、`tokens`、`errors`
- `run_summary`: Statistical aggregates per configuration
  `run_summary`：每种配置的统计汇总
  - `with_skill` / `without_skill`: Each contains `pass_rate`, `time_seconds`, `tokens` objects with `mean` and `stddev` fields
    `with_skill` / `without_skill`：各自包含 `pass_rate`、`time_seconds`、`tokens` 对象，均带 `mean` 与 `stddev` 字段
  - `delta`: Difference strings like `"+0.50"`, `"+13.0"`, `"+1700"`
    `delta`：形如 `"+0.50"`、`"+13.0"`、`"+1700"` 的差异字符串
- `notes`: Freeform observations from the analyzer
  `notes`：来自分析器的自由格式观察记录

**Important:** The viewer reads these field names exactly. Using `config` instead of `configuration`, or putting `pass_rate` at the top level of a run instead of nested under `result`, will cause the viewer to show empty/zero values. Always reference this schema when generating benchmark.json manually.

**重要：** 查看器按精确名称读取这些字段。用 `config` 代替 `configuration`，或把 `pass_rate` 放在运行的顶层而不是嵌套在 `result` 之下，都会导致查看器显示空值/零值。手动生成 benchmark.json 时务必参照本模式（schema）。

【评论】此处强调字段名采用精确字符串匹配而非宽松解析，是查看器类工具的常见设计；用整段警告加以突出，说明实践中此类命名偏差较易发生。

---

## comparison.json / comparison.json（对比结果）

Output from blind comparator. Located at `<grading-dir>/comparison-N.json`.

盲评对比器（blind comparator）的输出。位于 `<grading-dir>/comparison-N.json`。

```json
{
  "winner": "A",
  "reasoning": "Output A provides a complete solution with proper formatting and all required fields. Output B is missing the date field and has formatting inconsistencies.",
  "rubric": {
    "A": {
      "content": {
        "correctness": 5,
        "completeness": 5,
        "accuracy": 4
      },
      "structure": {
        "organization": 4,
        "formatting": 5,
        "usability": 4
      },
      "content_score": 4.7,
      "structure_score": 4.3,
      "overall_score": 9.0
    },
    "B": {
      "content": {
        "correctness": 3,
        "completeness": 2,
        "accuracy": 3
      },
      "structure": {
        "organization": 3,
        "formatting": 2,
        "usability": 3
      },
      "content_score": 2.7,
      "structure_score": 2.7,
      "overall_score": 5.4
    }
  },
  "output_quality": {
    "A": {
      "score": 9,
      "strengths": ["Complete solution", "Well-formatted", "All fields present"],
      "weaknesses": ["Minor style inconsistency in header"]
    },
    "B": {
      "score": 5,
      "strengths": ["Readable output", "Correct basic structure"],
      "weaknesses": ["Missing date field", "Formatting inconsistencies", "Partial data extraction"]
    }
  },
  "expectation_results": {
    "A": {
      "passed": 4,
      "total": 5,
      "pass_rate": 0.80,
      "details": [
        {"text": "Output includes name", "passed": true}
      ]
    },
    "B": {
      "passed": 3,
      "total": 5,
      "pass_rate": 0.60,
      "details": [
        {"text": "Output includes name", "passed": true}
      ]
    }
  }
}
```

---

## analysis.json / analysis.json（分析结果）

Output from post-hoc analyzer. Located at `<grading-dir>/analysis.json`.

事后分析器（post-hoc analyzer）的输出。位于 `<grading-dir>/analysis.json`。

```json
{
  "comparison_summary": {
    "winner": "A",
    "winner_skill": "path/to/winner/skill",
    "loser_skill": "path/to/loser/skill",
    "comparator_reasoning": "Brief summary of why comparator chose winner"
  },
  "winner_strengths": [
    "Clear step-by-step instructions for handling multi-page documents",
    "Included validation script that caught formatting errors"
  ],
  "loser_weaknesses": [
    "Vague instruction 'process the document appropriately' led to inconsistent behavior",
    "No script for validation, agent had to improvise"
  ],
  "instruction_following": {
    "winner": {
      "score": 9,
      "issues": ["Minor: skipped optional logging step"]
    },
    "loser": {
      "score": 6,
      "issues": [
        "Did not use the skill's formatting template",
        "Invented own approach instead of following step 3"
      ]
    }
  },
  "improvement_suggestions": [
    {
      "priority": "high",
      "category": "instructions",
      "suggestion": "Replace 'process the document appropriately' with explicit steps",
      "expected_impact": "Would eliminate ambiguity that caused inconsistent behavior"
    }
  ],
  "transcript_insights": {
    "winner_execution_pattern": "Read skill -> Followed 5-step process -> Used validation script",
    "loser_execution_pattern": "Read skill -> Unclear on approach -> Tried 3 different methods"
  }
}
```
