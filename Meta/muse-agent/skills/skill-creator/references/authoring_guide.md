<!-- BILINGUAL-EN-ZH -->
# Skill Authoring Guide / 技能编写指南

Use this reference when the task needs detail beyond the main workflow in `SKILL.md`.

当任务需要超出 `SKILL.md` 主工作流的细节时，使用本参考。

## Naming / 命名
- Directory name: `kebab-case`
  目录名：`kebab-case`
- Frontmatter `name`: `snake_case`
  front matter 的 `name`：`snake_case`
- Keep names short, concrete, and capability-based
  名称要简短、具体、基于能力
- Namespace by provider or domain when it improves trigger clarity, for example `google-calendar` or `outlook-calendar`
  当按提供方或领域加命名空间能提升触发清晰度时就这么做，例如 `google-calendar` 或 `outlook-calendar`

## Resource Split / 资源拆分
Choose the smallest structure that carries the skill reliably.

选择能可靠承载该技能的最小结构。

- Keep instructions in `SKILL.md` when they are short, stable, and required on every trigger.
  当指令简短、稳定且每次触发都需要时，保留在 `SKILL.md` 中。
- Use `references/` when the detail is useful but conditional: long examples, schemas, variant-specific notes, or extended workflows.
  当细节有用但属条件性内容（长示例、模式、特定变体的说明或扩展工作流）时，放入 `references/`。
- Use `assets/` only for files that become part of the delivered output.
  `assets/` 只用于会成为交付输出一部分的文件。
- For Jarvis bundled skills, prefer `bin/` helpers for repeated protocol, credential, or parsing work. Do not leave those mechanics in prompt text if a helper can own them.
  对于 Jarvis 内置技能，重复的协议、凭据或解析工作优先交给 `bin/` 辅助脚本完成。如果辅助脚本能承担，就不要把这些机制留在提示词文本里。

## Frontmatter Template / Front matter 模板
```yaml
---
name: "my_skill"
description: "One-line description of what the skill does and when to use it."
---
```

Notes:

说明：

- `name` and `description` are the core trigger surface.
  `name` 和 `description` 是核心触发面。
- Bundled skills should usually not set `metadata.includeInPrompt`.
  内置技能通常不应设置 `metadata.includeInPrompt`。

## Body Templates / 正文模板

### Tool-backed skill / 基于工具的技能
```markdown
# Skill Title

## Purpose
One line.

## Tooling
Exact commands, key flags, and the response fields the model should parse.

## Auth
Where auth lives, what setup to do first, and what not to print.

## Operating Rules
Short numbered constraints the tool itself does not enforce.
```

### Workflow-only skill / 纯工作流技能
```markdown
# Skill Title

## Purpose
One line.

## Workflow
Ordered steps for the agent.

## Output Contract
What the final result should contain.

## Operating Rules
Short numbered constraints.
```

## What to Move Out of `SKILL.md` / 应从 `SKILL.md` 移出的内容
- Long API endpoint catalogs
  冗长的 API 端点目录
- Full response schema dumps
  完整的响应模式转储
- Repeated auth/token extraction snippets
  反复出现的认证/令牌提取片段
- Lengthy tutorials or background essays
  冗长的教程或背景长文
- Large blocks of variant-specific guidance that only apply sometimes
  只在部分情况适用的大段特定变体指导

Move that material to `references/` and link it from the relevant section in `SKILL.md`.

把这些材料移到 `references/`，并从 `SKILL.md` 的相关章节链接过去。

## Review Checklist / 检查清单
- Does the description clearly state both capability and trigger context?
  description 是否同时清楚说明了能力与触发场景？
- Is the skill scoped to one coherent job?
  技能是否限定于一个连贯的职责？
- Does `SKILL.md` tell the model what to do next, instead of teaching the whole subject?
  `SKILL.md` 是在告诉模型下一步做什么，而不是讲授整个主题？
- Are commands and paths real for this repo/runtime?
  命令与路径在本仓库/运行时中是否真实存在？
- Are auth expectations explicit when needed?
  需要时认证预期是否写明确切？
- Did you remove `includeInPrompt` unless there is a strong reason to keep it?
  除非有充分理由保留，你是否已移除 `includeInPrompt`？
- If the skill uses helpers, does the prompt rely on them instead of duplicating their work?
  如果技能使用辅助脚本，提示词是否依赖它们而不是重复其工作？
