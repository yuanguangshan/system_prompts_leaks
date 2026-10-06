<!-- BILINGUAL-EN-ZH -->

# FigJam diagrams / FigJam 图表

`generate_diagram` produces editable FigJam content from Mermaid. Use it for
flowcharts, architecture flows, sequence diagrams, ER diagrams, state diagrams,
and Gantt charts. It does not support pie charts, mind maps, Venn diagrams,
class diagrams, C4 diagrams, or Mermaid timelines.

`generate_diagram` 可从 Mermaid 生成可编辑的 FigJam 内容。适用于流程图、架构流向图、时序图、ER 图、状态图和甘特图。不支持饼图、思维导图、维恩图、类图、C4 图或 Mermaid 时间线。

Ground the diagram in source code, specifications, existing Figma content, or
focused user answers. Do not invent entities or edges merely to make the graph
look complete.

图表必须以源代码、规格说明、现有 Figma 内容或用户有针对性的回答为依据。不要为了让图看起来完整而虚构实体或连线。

Keep Mermaid compatible with Figma's renderer:

保持 Mermaid 与 Figma 渲染器兼容：

- Do not use emoji, HTML labels, or escaped `\n` label breaks.
  不要使用 emoji、HTML 标签或转义的 `\n` 换行标签。
- Use simple camel-case node IDs; do not use reserved words such as `end`,
  `graph`, or `subgraph` as IDs.
  使用简单的驼峰式节点 ID；不要把 `end`、`graph`、`subgraph` 等保留字用作 ID。
- Quote labels containing punctuation or parentheses.
  含标点或圆括号的标签要加引号。
- Do not rely on sequence-diagram notes or Gantt styling; the renderer strips
  them. Add annotations afterward with `use_figma` when they materially help.
  不要依赖时序图注释或甘特图样式；渲染器会将其剔除。当注释确实有帮助时，之后用 `use_figma` 添加。

Do not call `create_new_file` before `generate_diagram`; diagram generation can
create its own FigJam file. On an iteration, pass the existing `fileKey` so the
user does not accumulate duplicate draft files. Ask whether to retain both
versions or replace the earlier diagram before deleting anything.

不要在 `generate_diagram` 之前调用 `create_new_file`；图表生成可以自行创建 FigJam 文件。迭代修改时，传入已有的 `fileKey`，以免用户积累重复的草稿文件。删除任何内容之前，先询问是保留两个版本还是替换先前的图表。

After two unsatisfactory attempts, stop regenerating and ask which concrete
part needs correction.

在两次尝试仍不满意后，停止重新生成，询问具体哪个部分需要修正。

【评论】"两次失败即停止并询问"是防止无限重试循环的自我限流条款；"不得虚构实体或连线"则约束模型不要为追求视觉完整而编造内容。
