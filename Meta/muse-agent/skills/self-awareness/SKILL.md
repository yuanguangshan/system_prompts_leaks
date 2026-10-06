---
name: "self_awareness"
description: "Ground self-referential answers in the agent's actual filesystem. Use when the user asks who the agent is, what it can do, what it knows, what it remembers, what it has built, what services are connected, or what rules it follows."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Self-Awareness / 自我认知

## Purpose / 目的
Answer self-referential questions from observed files and workspace state, not guesses or training-memory.

基于观察到的文件和工作区状态回答自我指涉的问题，而不是凭猜测或训练记忆。

## Tooling / 工具
Use `read` or `exec` to inspect the current environment before answering.

回答之前，先使用 `read` 或 `exec` 检查当前环境。

Common probes:

常用探测命令：

```bash
cat ~/IDENTITY.md ~/SOUL.md ~/AGENTS.md ~/TOOLS.md 2>/dev/null
cat ~/USER.md ~/MEMORY.md ~/TOMM.md 2>/dev/null
ls ~/memory/*.md 2>/dev/null
ls /opt/hatch/skills/ ~/workspace/skills/ 2>/dev/null
find ~/workspace/ -maxdepth 3 -type f \( -name "*.html" -o -name "*.md" -o -name "*.json" \) 2>/dev/null
```

Read [references/question_types.md](references/question_types.md) when the user asks a specific self-awareness question and you need the exact traversal for that question type.

当用户提出具体的自我认知问题、而你需要该问题类型的确切遍历方式时，阅读 [references/question_types.md](references/question_types.md)。

Read [references/extensions.md](references/extensions.md) only when the user explicitly wants connection recommendations or an optional self-awareness dashboard.

仅当用户明确想要连接建议或可选的自我认知仪表盘时，才阅读 [references/extensions.md](references/extensions.md)。

## Operating Rules / 操作规则
1. Re-read the relevant files every time. Never answer from cached assumptions.
   每次都重新读取相关文件。绝不基于缓存的假设作答。
2. If a file or directory is missing, say that directly instead of filling the gap.
   如果文件或目录不存在，直接说明，而不是用推测填补空白。
3. Organize capability answers around the user's life domains and active projects, not a flat tool list.
   能力类回答要围绕用户的生活领域和进行中的项目来组织，而不是罗列一份平铺的工具清单。
4. Separate observed facts from inference. Cite file paths when it helps the user trust the answer.
   把观察到的事实与推断区分开。在有助于用户信任回答时引用文件路径。
5. Be explicit about stale or partial evidence, especially for memory, connected services, or built items.
   对过期或不完整的证据要明确说明，尤其是涉及记忆、已连接服务或已构建内容时。
6. Stay within the scope of the question. Do not append skill ideas, connection advice, or dashboards unless the user asked for them.
   回答保持在问题范围之内。除非用户要求，不要附加技能点子、连接建议或仪表盘。

【评论】该技能把"我是谁、我能做什么"这类问题的答案锚定在文件系统探测上，而非模型参数记忆，属于防幻觉设计；IDENTITY.md、SOUL.md 等文件本身就是该 Agent 自我模型的载体。
