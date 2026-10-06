<!-- BILINGUAL-EN-ZH -->

# Code Connect / Code Connect

Use Code Connect only for published components on an Organization or Enterprise
plan. The Figma URL must identify a node.

Code Connect 仅适用于 Organization 或 Enterprise 套餐下已发布的组件。Figma URL 必须能定位到某个具体节点。

1. Call `get_code_connect_suggestions` with `excludeMappingPrompt` enabled to
   discover published, unmapped components in the selection.
   调用 `get_code_connect_suggestions` 并启用 `excludeMappingPrompt`，以发现选区中已发布但尚未映射的组件。
2. For each returned main component ID, call
   `get_context_for_code_connect`. Supply the project's actual language and
   framework, inferred from its source and `figma.config.json`.
   对返回的每个主组件（main component）ID，调用 `get_context_for_code_connect`。提供项目的实际语言和框架，需从项目源代码和 `figma.config.json` 推断得出。
3. Inspect the codebase and match each Figma component to a real code component
   by purpose, property types, variants, and import configuration. If more than
   one candidate is plausible, confirm the match with the user before writing.
   检查代码库，按用途、属性类型、变体（variant）和导入配置，将每个 Figma 组件匹配到真实的代码组件。如果存在多个合理的候选，先与用户确认匹配结果再写入。
4. Account for every Figma property that has a legitimate code equivalent.
   Map every variant value exhaustively; omit properties with no corresponding
   code prop instead of inventing one. Resolve configurable nested components
   dynamically rather than hardcoding their output.
   覆盖每个在代码中有合理对应物的 Figma 属性。穷尽映射每个变体取值；没有对应代码 prop 的属性应直接省略，而不是凭空编造。可配置的嵌套组件要动态解析，而不是硬编码其输出。
5. Validate the mapping against the code component's real prop types and the
   complete Figma property set before publishing it with an advertised Code
   Connect management tool.
   在用已公布的 Code Connect 管理工具发布映射之前，先对照代码组件真实的 prop 类型和完整的 Figma 属性集合进行校验。

For parserless templates, create `ComponentName.figma.ts`, use the `figma.code`
tagged-template form, and do not replace an existing parser-based `.figma.tsx`
mapping. Follow the project's existing Code Connect format when it has already
standardized on another supported workflow.

对于无解析器（parserless）模板，创建 `ComponentName.figma.ts`，使用 `figma.code` 标签模板形式，并且不要替换已有的基于解析器的 `.figma.tsx` 映射。当项目已标准化采用另一种受支持的工作流时，应遵循其现有的 Code Connect 格式。
