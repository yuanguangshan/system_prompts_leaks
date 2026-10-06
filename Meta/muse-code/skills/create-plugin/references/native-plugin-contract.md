<!-- BILINGUAL-EN-ZH -->
# Native Muse Plugin Contract / 原生 Muse 插件契约

This reference summarizes the manifest accepted by the current Muse validator.
The Plugin Creator instructions remain authoritative for reservation, mutation,
validation order, correction limits, and reporting.

本参考文档概述当前 Muse 校验器所接受的清单（manifest）。关于占位、变更、校验顺序、修正次数上限和报告，仍以插件创建器（Plugin Creator）指令为准。

## Package Layout / 包结构

A native plugin is a new directory in the current workspace. Its manifest is:

原生插件是当前工作区中的一个新目录。其清单文件为：

```text
.muse-plugin/plugin.json
```

Every file named by a capability is relative to the plugin root. A minimal manifest
contains:

能力（capability）所引用的每个文件都相对于插件根目录。最小清单包含：

```json
{
  "schemaVersion": 1,
  "name": "example-plugin",
  "displayName": "Example Plugin",
  "version": "0.1.0",
  "description": "One plain-language sentence.",
  "compat": {
    "source": "native",
    "manifestDir": ".muse-plugin"
  },
  "capabilities": {
    "skills": [],
    "commands": [],
    "hooks": [],
    "mcpServers": [],
    "reminders": []
  }
}
```

Use only fields required by the request. The installed validator decides whether an
omitted optional field is acceptable.

只使用请求所要求的字段。某个可选字段省略后是否可接受，由已安装的校验器决定。

## Identifiers / 标识符

Plugin and capability IDs use this portable grammar:

插件与能力 ID 使用以下可移植语法：

```text
^[a-z0-9][a-z0-9._-]{0,79}$
```

The basename before the first dot must not case-fold to `CON`, `PRN`, `AUX`, `NUL`,
`COM1` through `COM9`, or `LPT1` through `LPT9`. Plugin IDs `loop` and `muse-core`
are reserved by the product bundle.

第一个点号之前的基本名在大小写折叠后不得等于 `CON`、`PRN`、`AUX`、`NUL`、`COM1` 到 `COM9`、或 `LPT1` 到 `LPT9`。插件 ID `loop` 和 `muse-core` 由产品捆绑包保留。

## Paths / 路径

- Use relative UTF-8 paths and `/` separators in the manifest.
- 在清单中使用相对的 UTF-8 路径和 `/` 分隔符。
- Reject absolute paths, parent traversal, backslashes, and blank paths.
- 拒绝绝对路径、父目录穿越、反斜杠和空白路径。
- Create every referenced file before validation.
- 在校验之前创建所有被引用的文件。
- Canonicalize containment; a symlink must not escape the plugin or workspace root.
- 规范化包含关系；符号链接不得逃逸出插件根目录或工作区根目录。
- Keep generated component names portable across macOS, Linux, and Windows.
- 保持生成的组件名称在 macOS、Linux 和 Windows 之间可移植。

## Capability Fields / 能力字段

### Skills

```json
{"id":"review","path":"skills/review/SKILL.md","enabledDefault":false}
```

The target is a UTF-8 `SKILL.md` with valid frontmatter. `enabledDefault` is
optional; omit it only when the requested activation behavior is clear.

目标是一个带有效 frontmatter 的 UTF-8 `SKILL.md`。`enabledDefault` 是可选的；仅当所要求的激活行为已经明确时才省略它。

### Commands

```json
{"id":"summarize","path":"commands/summarize.md","enabledDefault":true}
```

The target is a UTF-8 Markdown command template.

目标是一个 UTF-8 的 Markdown 命令模板。

### Hooks

```json
{
  "id":"pre-check",
  "event":"PreToolUse",
  "command":["sh","hooks/pre-check.sh"],
  "timeoutMs":1000,
  "statusMessage":"Checking plugin policy"
}
```

The command is structured argv, not a shell string. If an argv element names a
relative source path, that regular file must exist beneath the plugin root. Hook
source paths cannot be shared by two hook IDs.

command 是结构化的 argv，而不是 shell 字符串。如果某个 argv 元素指名了一个相对源路径，该普通文件必须存在于插件根目录之下。hook 源路径不能被两个 hook ID 共享。

### MCP Servers

For a local stdio server / 对于本地 stdio 服务器：

```json
{
  "id":"workspace-index",
  "transport":"stdio",
  "command":["python3","mcp/server.py"]
}
```

`transport` defaults to `stdio`. An HTTP transport instead requires a non-empty
`url`; do not invent an endpoint. A custom model tool is exposed by an MCP server,
not by a direct `tools` capability.

`transport` 默认为 `stdio`。若使用 HTTP 传输，则改为要求一个非空的 `url`；不要凭空编造端点。自定义模型工具由 MCP 服务器暴露，而不是由直接的 `tools` 能力暴露。

### Reminders

```json
{
  "id":"review-policy",
  "path":"reminders/review-policy.md",
  "tools":["read_file"],
  "blocking":false,
  "decision":{
    "version":1,
    "envelope":{"version":1,"template":"<system-reminder>\n{text}\n</system-reminder>"},
    "deliveryRole":"developer",
    "...":"copy the remaining executable fields from capability-examples.json"
  }
}
```

The duty file must exist and be UTF-8. `decision` is required, including its V1
`envelope` and `deliveryRole` authority. Optional policy
fields are `defaultPriority`, `maxPriority`, `maxChildSteps`,
`maxInstallsPerRun`, `reasoningEffort`, and `context`. Add them only when
requested and after checking the executable examples.

职责文件必须存在且为 UTF-8 编码。`decision` 为必填，包括其 V1 `envelope` 和 `deliveryRole` 权限主体。可选的策略字段有 `defaultPriority`、`maxPriority`、`maxChildSteps`、`maxInstallsPerRun`、`reasoningEffort` 和 `context`。仅在请求要求时、且在核对可执行示例之后再添加它们。

## Unsupported Families / 不支持的能力族

The validator rejects direct capability keys `tools`, `agents`, `outputStyles`,
`settings`, and `apps`. Do not translate them into guessed fields. Ask the user to
restate a custom tool as an MCP server when that matches their intent.

校验器拒绝直接的能力键 `tools`、`agents`、`outputStyles`、`settings` 和 `apps`。不要把它们翻译成猜测出来的字段。当符合用户意图时，请用户把自定义工具改述为一个 MCP 服务器。

## Validation / 校验

Run each generated skill first / 先运行每个生成的技能：

```json
["muse","skills","validate","<skill-directory>","--json"]
```

Then run the plugin validator / 然后运行插件校验器：

```json
["muse","plugins","validate","<plugin-directory>","--json"]
```

A result is clean only when the process starts, exits zero, returns parseable JSON,
sets top-level `valid` to `true`, and returns an empty `diagnostics` array. Warnings
are failures for creation. Correct one validation layer at a time, with no more than
three correction rounds after its first failed result.

只有当进程启动、以零退出、返回可解析的 JSON、把顶层 `valid` 置为 `true`、并返回空的 `diagnostics` 数组时，结果才算干净。对于创建而言，警告即视为失败。每次只修正一个校验层，自其首次失败结果起，修正轮数不超过三轮。

## Creation Boundary / 创建边界

Reserve one absent destination with a native atomic no-replace directory creation
before writing artifacts. Recheck canonical workspace containment immediately after
reservation. Stop on occupancy, denial, cancellation, or any failed containment
check. Never install, enable, trust, execute, fetch, update, or publish the draft.

在写入工件之前，先用原生的原子性"不可替换"目录创建占位一个尚不存在的目标位置。占位之后立即重新检查规范化的工作区包含关系。一旦遇到占用、拒绝、取消或任何包含关系检查失败，立即停止。绝不对草稿执行安装、启用、信任、执行、获取、更新或发布操作。

【评论】"绝不安装/执行/发布草稿"与原子占位等约束表明该契约将插件生成流程严格限定在纯文件写入层面，防止生成中的代码获得执行机会。
