<!-- BILINGUAL-EN-ZH -->
---
name: "consolidate-memory"
description: "Reflective pass over your memory files — merge duplicates, fix stale facts, prune the index."
---

# Memory Consolidation / 记忆整合

You're doing a reflective pass over what you've learned about this user and their work. The goal: a future session should be able to orient quickly — who they work with, what they're focused on, how they like things done — without re-asking.

你正在对已了解的该用户及其工作进行一次回顾性梳理。目标是：未来的会话应能快速进入状态——了解用户与谁共事、正在专注什么、偏好怎样的做事方式——而无需重新询问。

Your system prompt's auto-memory section defines the directory, file format, and memory types. Follow it.

系统提示词中的自动记忆（auto-memory）部分定义了目录、文件格式和记忆类型，请遵循该部分的规定。

## Phase 1 — Take stock / 第一阶段——盘点

- List the memory directory and read the index (`MEMORY.md`)
  - 列出记忆目录并读取索引（`MEMORY.md`）
- Skim each topic file. Note which ones overlap, which look stale, which are thin.
  - 粗读每个主题文件，记下哪些内容重叠、哪些已过时、哪些过于单薄。

## Phase 2 — Consolidate / 第二阶段——整合

**Separate the durable from the dated.** Preferences, working style, key relationships, and recurring workflows are durable — keep and sharpen them. Specific projects, deadlines, and one-off tasks are dated — if the date has passed or the work is done, retire the file or fold the lasting takeaway (e.g. "user prefers X format for launch docs") into a durable one.

**区分持久信息与时效信息。**偏好、工作风格、关键人际关系和周期性工作流属于持久信息——保留并提炼它们。具体项目、截止日期和一次性任务属于时效信息——如果日期已过或工作已完成，就废弃该文件，或把有长期价值的要点（例如"用户偏好发布文档采用 X 格式"）并入持久类文件。

**Merge overlaps.** If two files describe the same person, project, or preference, combine into one and keep the richer file's path.

**合并重叠内容。**如果两个文件描述的是同一个人、项目或偏好，则合并为一个，并保留内容更丰富文件的路径。

**Fix time references.** Convert "next week", "this quarter", "by Friday" to absolute dates so they stay readable later.

**修正时间表述。**把"下周"、"本季度"、"周五前"转换为绝对日期，以便日后阅读时仍然可理解。

**Drop what's easy to re-find.** If a memory just restates something you could pull from the user's calendar, docs, or connected tools on demand, cut it. Keep what's hard to re-derive: stated preferences, context behind a decision, who to go to for what.

**舍弃易于重新获取的内容。**如果某条记忆只是复述了随时可以从用户日历、文档或已连接工具中查到的信息，就删掉它。保留难以重新推导的内容：明确表达的偏好、决策背后的背景、什么事该找什么人。

## Phase 3 — Tidy the index / 第三阶段——整理索引

Update `MEMORY.md` so it stays under 200 lines and ~25KB. One line per entry, under ~150 chars: `- [Title](file.md) — one-line hook`.

更新 `MEMORY.md`，使其保持在 200 行以内、约 25KB 以内。每条占一行、不超过约 150 个字符：`- [Title](file.md) — one-line hook`。

- Remove pointers to retired memories
  - 移除指向已废弃记忆的条目
- Shorten any line carrying detail that belongs in the topic file
  - 缩短任何承载了本应属于主题文件细节的行
- Add anything newly important
  - 补充新近变得重要的内容

Finish with a short summary: how many files you touched and what changed.

最后给出一段简短总结：你改动了多少个文件以及发生了什么变化。
