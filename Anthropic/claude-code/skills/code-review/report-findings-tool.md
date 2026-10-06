<!-- BILINGUAL-EN-ZH -->
# ReportFindings tool / ReportFindings 工具

Report code-review findings as a typed list so the host UI can render them. Use this only when the active code-review instructions tell you to report findings with this tool; otherwise follow whatever output format those instructions specify. When reporting a review's results, call it once with the verified findings ranked most-severe first (empty array if nothing survived verification) and do not also print the findings as text. When re-reporting after applying fixes (only if the apply instructions ask for it), set `outcome` on each finding to what actually happened.

以带类型的列表形式上报代码评审发现，以便宿主 UI 进行渲染。仅当当前生效的代码评审指令要求你用本工具上报发现时才使用它；否则遵循那些指令指定的任何输出格式。上报评审结果时，只调用一次，传入经验证的发现并按严重程度从高到低排序（若没有任何发现通过验证则传空数组），并且不要再把发现以文本形式打印一遍。在应用修复后重新上报时（仅当应用修复的指令要求时），把每个发现的 `outcome` 字段设置为实际发生的结果。

## input_schema / input_schema（输入模式）

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "level": {
      "description": "Effort level the review ran at",
      "type": "string",
      "enum": [
        "low",
        "medium",
        "high",
        "xhigh",
        "max"
      ]
    },
    "findings": {
      "description": "Verified findings, most-severe first; empty if none survived",
      "maxItems": 32,
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "file": {
            "description": "Repo-relative path of the file the finding is in",
            "type": "string"
          },
          "line": {
            "description": "1-indexed line the finding anchors to",
            "type": "integer",
            "minimum": -9007199254740991,
            "maximum": 9007199254740991
          },
          "summary": {
            "description": "One-sentence statement of the defect",
            "type": "string"
          },
          "short_summary": {
            "description": "Compressed label for compact UI (≤60 chars): the claim alone, no rationale or consequence clause",
            "type": "string",
            "maxLength": 60
          },
          "failure_scenario": {
            "description": "Concrete inputs/state → wrong output/crash",
            "type": "string"
          },
          "category": {
            "description": "Short kebab-case slug of the finding type, e.g. \"correctness\", \"simplification\", \"efficiency\", \"test-coverage\"",
            "type": "string",
            "maxLength": 40
          },
          "verdict": {
            "description": "Set when a verify pass ran; absent on inline-only reviews",
            "type": "string",
            "enum": [
              "CONFIRMED",
              "PLAUSIBLE"
            ]
          },
          "outcome": {
            "description": "Set ONLY when re-reporting after applying fixes: what happened to this finding",
            "type": "string",
            "enum": [
              "fixed",
              "skipped",
              "no_change_needed"
            ]
          }
        },
        "required": [
          "file",
          "summary",
          "failure_scenario"
        ],
        "additionalProperties": false
      }
    }
  },
  "required": [
    "findings"
  ],
  "additionalProperties": false
}
```
