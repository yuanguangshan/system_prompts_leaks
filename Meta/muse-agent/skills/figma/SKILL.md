---
name: "figma"
description: >-
  Inspect Figma designs and generate implementation context for screens and
  interfaces. Use to read component variants, spacing and design tokens, inspect
  layouts and design-system libraries, and extract image assets for implementing
  a design through Figma's official MCP server.
icon: "figma"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Figma

Use the installed `figma` CLI. Start with `figma status`. If it reports
`not_connected`, run `figma authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `figma` CLI。先从 `figma status` 开始。如果它报告
`not_connected`，则运行 `figma authorize-url`，并且只把返回的
`connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. Figma issues a
per-registration confidential client; authd stores its generated secret outside
the runtime cell, so credentials must never be requested in chat.

OAuth 通过 authd 使用动态客户端注册和 PKCE。Figma 会为每次注册签发一个机密客户端；authd 将其生成的密钥存储在运行时单元之外，因此绝不能在聊天中请求凭据。

Run `figma list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `figma list-tools` 查看实时的提供方目录与模式（schema），然后用以下方式调用已通告的工具：

```text
figma call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Figma tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. Read permissions follow
the user's connector settings; design, file, asset, Code Connect, plugin, and
shader changes require granular approval. Do not retry a failed or timed-out
write automatically because its side effect may have completed.

`list-tools` 只暴露经过审核的 Figma 工具，并包含每个工具的
`hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或新的提供方工具在经过审核前保持不可用。读取权限遵循用户的连接器设置；设计、文件、资产、Code Connect、插件和着色器的更改需要细粒度批准。不要自动重试失败或超时的写操作，因为其副作用可能已经完成。

【评论】该条款禁止自动重试失败的写操作，是为避免重复执行可能已生效的副作用，属于典型的工具调用安全设计。

## Load Figma's provider skills / 加载 Figma 的提供方技能

Before calling a tool whose description requires a Figma skill, fetch that
skill with `get_figma_skill` and follow its prerequisites. These `skill://`
URIs refer to resources on Figma's MCP server. For example, before
`create_new_file`, read:

在调用描述中要求具备某项 Figma 技能的工具之前，先用 `get_figma_skill` 获取该技能并遵循其前置条件。这些 `skill://`
URI 指向 Figma MCP 服务器上的资源。例如，在执行
`create_new_file` 之前，先读取：

```text
figma call-tool --name get_figma_skill --arguments-json '{"uri":"skill://figma/figma-create-new-file/SKILL.md"}'
```

Use the skill URI advertised by the tool. If the relevant URI is unknown,
fetch `skill://index.json` with the same tool and select the matching skill.
Fetch supporting references only as needed, resolving relative paths against
the parent skill URI. Reuse guidance already loaded for the current task;
there is no need to install or copy these provider skills into the local
skill directory. Skill reads use the `designs.read` permission.

使用该工具所通告的技能 URI。如果相关 URI 未知，则用同一工具获取
`skill://index.json` 并选择匹配的技能。仅在需要时获取支持性引用，相对路径相对于父技能 URI 解析。复用当前任务中已加载的指引；无需将这些提供方技能安装或复制到本地技能目录。技能读取使用 `designs.read` 权限。

Call `get_figma_skill` only when advertised by `list-tools`. If the tool is
unavailable or Figma confirms that a skill is missing, continue using the
live tool description, schema, and local guides only when the skill is
optional (for example, "if it exists"). If the skill is required
unconditionally, stop the dependent operation and report the missing
guidance. Authentication failures, permission denials, timeouts, and other
read errors do not establish that a skill is missing. Avoid repeated failed
lookups; continue work that does not depend on the unavailable guidance.

只有在 `list-tools` 通告时才调用 `get_figma_skill`。如果该工具不可用，或 Figma 确认某技能缺失，则仅当该技能是可选的（例如"如果存在"）时，才继续使用实时的工具描述、模式（schema）和本地指南。如果该技能被无条件要求，则停止依赖它的操作并报告缺失的指引。身份验证失败、权限拒绝、超时及其他读取错误并不能证明技能缺失。避免反复进行注定失败的查找；继续执行不依赖该不可用指引的工作。

## Choose the workflow / 选择工作流

Read only the guide that matches the task before calling provider tools:

在调用提供方工具之前，只阅读与任务匹配的指南：

- Implement a Figma design in code: [design-to-code](references/design-to-code.md)
  在代码中实现 Figma 设计：[design-to-code](references/design-to-code.md)
- Create or edit canvas content with `use_figma`:
  [canvas editing](references/canvas-editing.md)
  使用 `use_figma` 创建或编辑画布内容：
  [canvas editing](references/canvas-editing.md)
- Generate a FigJam diagram: [diagrams](references/diagrams.md)
  生成 FigJam 图表：[diagrams](references/diagrams.md)
- Create Code Connect mappings: [Code Connect](references/code-connect.md)
  创建 Code Connect 映射：[Code Connect](references/code-connect.md)

For a Figma URL, extract its `fileKey` and `node-id`; convert node IDs from URL
form (`123-456`) to API form (`123:456`). Inspect the live schema before
constructing arguments because Figma can add optional fields without changing
this skill.

对于 Figma URL，提取其 `fileKey` 和 `node-id`；将节点 ID 从 URL 形式（`123-456`）转换为 API 形式（`123:456`）。在构造参数之前先查看实时模式（schema），因为 Figma 可能在不更改此技能的情况下新增可选字段。

Avoid redundant catalogue, context, screenshot, and validation calls. Figma
applies account- and seat-dependent usage limits, and repeated reads can consume
a user's small monthly allowance. Reuse results gathered earlier in the task.

避免冗余的目录、上下文、截图和校验调用。Figma 实施与账户和席位相关的用量限制，重复读取可能耗尽用户每月的少量配额。请复用任务中早先收集的结果。

【评论】该技能强调节省调用配额并复用结果，反映出 Figma MCP 接口存在按账户与席位计量的用量限制，这是外部服务集成中常见的成本约束。
