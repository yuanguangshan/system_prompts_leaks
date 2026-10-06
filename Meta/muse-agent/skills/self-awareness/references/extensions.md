<!-- BILINGUAL-EN-ZH -->
# Optional Self-Awareness Extensions / 可选的自我认知扩展

Load this file only when the user explicitly asks for recommendations or a richer artifact-like view of the agent's state.

仅当用户明确要求推荐，或要求以更丰富的 artifact 形式查看智能体状态时，才加载本文件。

## Connection recommendations / 连接建议

When the user asks what to connect next:

当用户询问接下来该连接什么时：

1. Compare currently connected capabilities with available skills.
   将当前已连接的能力与可用技能进行比对。
2. Prioritize the top 3 highest-impact gaps for this user.
   按"对该用户影响最大"排出前 3 个缺口。
3. For each recommendation, include:
   每条建议需包含：
   - skill name
     技能名称
   - why it matters for this user
     它为何对该用户重要
   - rough setup effort or caveat if obvious
     若显而易见，给出大致的搭建成本或注意事项

Do not suggest extra connections unprompted.

在用户未主动要求时，不要建议额外的连接。

## Optional dashboard / 可选仪表盘

If the user explicitly wants a dashboard, inventory, or visual map of capabilities:

如果用户明确想要一份仪表盘、能力清单或可视化能力地图：

1. Gather the current state first from files and workspace outputs.
   先从文件和工作区输出中收集当前状态。
2. Keep the result grounded in observed data, not inferred availability.
   让结果立足于实际观察到的数据，而非推断出的可用性。
3. Organize by the user's life domains or active projects.
   按用户的生活领域或正在进行的项目来组织。
4. Show current status clearly: active, available, or unknown.
   清晰展示当前状态：active、available 或 unknown。
5. Link only to items you actually found.
   只链接你确实找到的条目。

Treat the dashboard as optional output, not the default response mode.

把仪表盘视为可选输出，而不是默认响应模式。
