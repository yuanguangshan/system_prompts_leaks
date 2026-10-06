---
name: create-plugin
description: Create and validate a new native Muse plugin package in the current workspace. Use ONLY when the user explicitly asks to create a Muse plugin or invokes the create-plugin skill. Do NOT use for application/library plugin classes, third-party plugin systems, or ordinary code changes.
---
<!-- BILINGUAL-EN-ZH -->
# Create Plugin / 创建插件

Create one new native Muse plugin in the current workspace. Use this skill only
for Muse native plugin package creation, not for implementing plugin classes or
plugin features inside another project. Produce the
smallest complete plugin that satisfies the request, validate it with the installed
muse CLI, and leave installation to the user.

在当前工作区中创建一个新的原生 Muse 插件。本技能只用于 Muse 原生插件包的创建，不用于在另一个项目中实现插件类或插件功能。产出满足请求的最小完整插件，用已安装的 muse CLI 验证，并把安装留给用户。

## Boundaries / 边界

- Work only in one new destination inside the current workspace.
  只在当前工作区内一个新的目标目录中工作。
- Never modify, merge into, delete, or replace an existing path.
  绝不修改、合并进、删除或替换既有路径。
- Do not install, enable, trust, execute, or fetch the generated plugin or its
  dependencies.
  不安装、不启用、不信任、不执行、不拉取生成的插件或其依赖。
- Do not update an existing plugin. Explain that update behavior needs a separate
  workflow.
  不更新既有插件。要说明更新行为需要单独的工作流。
- Do not publish or write global configuration.
  不发布，也不写全局配置。
- Ask before writing when the plugin ID, destination, requested capability, or
  required command is unclear.
  当插件 ID、目标目录、所请求的能力或所需命令不清晰时，先问再写。
- Reject unsupported capability families instead of inventing a schema.
  拒绝不受支持的能力族，而不是凭空发明 schema。

The supported capability families are exactly `skills`, `commands`, `hooks`,
`mcpServers`, and `reminders`. Reject `tools`, `agents`, `outputStyles`, `settings`,
and `apps`. A custom model tool belongs behind an `mcpServers` entry, not a direct
`tools` capability.

受支持的能力族恰好是 `skills`、`commands`、`hooks`、`mcpServers` 和 `reminders`。拒绝 `tools`、`agents`、`outputStyles`、`settings` 和 `apps`。自定义模型工具应放在某个 `mcpServers` 条目之后，而不是直接的 `tools` 能力。

## Optional References / 可选参考

This skill is complete without its references. If `read_skill` returned a physical
`SKILL.md` path and more detail would help, use `read_file` to inspect these siblings:

没有这些参考文件，本技能也是完整的。若 `read_skill` 返回了物理 `SKILL.md` 路径且更多细节会有帮助，用 `read_file` 查看这些同级文件：

- `references/native-plugin-contract.md`
- `references/capability-examples.json`

Treat them as read-only guidance. Do not fail merely because a reference was not
read.

把它们当作只读指引。不要仅仅因为某个参考文件未被阅读就判定失败。

## Clarify First / 先澄清

Before tools, determine:

在动用任何工具之前，先确定：

1. the portable plugin ID and human display name;
   可移植的插件 ID 和面向人的显示名称；
2. the destination relative to the current workspace;
   相对当前工作区的目标目录；
3. the requested capability families and IDs;
   所请求的能力族及 ID；
4. every required artifact and command;
   每个所需的工件和命令；
5. whether the request would require an unsupported family, dependency fetch, or
   modification of an existing path.
   该请求是否需要不受支持的族、依赖拉取，或修改既有路径。

Ask a focused question for missing facts. Refuse before mutation when the request is
outside this skill's boundaries.

就缺失的事实提出一个聚焦的问题。当请求超出本技能边界时，在任何变更之前拒绝。

## Portable IDs And Paths / 可移植的 ID 与路径

Plugin and capability IDs must:

插件和能力 ID 必须：

- contain 1 through 80 ASCII bytes;
  含 1 到 80 个 ASCII 字节；
- begin with a lowercase ASCII letter or digit;
  以小写 ASCII 字母或数字开头；
- use only lowercase ASCII letters, digits, `.`, `_`, and `-` afterward;
  之后只使用小写 ASCII 字母、数字、`.`、`_` 和 `-`；
- not have a case-insensitive basename before the first dot equal to `CON`, `PRN`,
  `AUX`, `NUL`, `COM1` through `COM9`, or `LPT1` through `LPT9`;
  第一个点之前、不区分大小写的基名不得等于 `CON`、`PRN`、`AUX`、`NUL`、`COM1` 至 `COM9` 或 `LPT1` 至 `LPT9`；
- not use a product-reserved plugin ID: `loop` or `muse-core`.
  不使用产品保留的插件 ID：`loop` 或 `muse-core`。

Artifact paths must be relative UTF-8 paths beneath the plugin root. Manifest paths
use `/`. Reject absolute paths, `..`, backslashes in manifest values, symlink
escapes, and any path whose canonical parent leaves the current workspace.

工件路径必须是插件根之下的相对 UTF-8 路径。清单路径使用 `/`。拒绝绝对路径、`..`、清单值中的反斜杠、符号链接逃逸，以及任何规范化父目录离开当前工作区的路径。

## Required Manifest Base / 必需的清单基座

Start from this native manifest shape and fill only requested capabilities:

从这个原生清单形状出发，只填入所请求的能力：

```json
{
  "schemaVersion": 1,
  "name": "<portable-id>",
  "displayName": "<human name>",
  "version": "0.1.0",
  "description": "<plain description>",
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

The manifest lives at `.muse-plugin/plugin.json`. Keep empty capability arrays
unless the installed validator accepts their omission with zero diagnostics.

清单位于 `.muse-plugin/plugin.json`。保留空的能力数组，除非已安装的验证器接受省略它们且零诊断输出。

For each requested capability, create every path referenced by the manifest:

对每个所请求的能力，创建清单引用的每个路径：

- `skills`: `{id, path, enabledDefault?}` and a UTF-8 `SKILL.md` with valid
  frontmatter;
  `skills`：`{id, path, enabledDefault?}`，以及一个带有效 frontmatter 的 UTF-8 `SKILL.md`；
- `commands`: `{id, path, enabledDefault?}` and a UTF-8 Markdown template;
  `commands`：`{id, path, enabledDefault?}`，以及一个 UTF-8 Markdown 模板；
- `hooks`: `{id, event, command, timeoutMs?, statusMessage?}` and any relative
  command source named by its argv;
  `hooks`：`{id, event, command, timeoutMs?, statusMessage?}`，以及其 argv 所指名的任何相对命令源文件；
- `mcpServers`: a stdio `{id, transport?, command}` or HTTP
  `{id, transport:"http", url}` entry and any relative command source named by its
  argv;
  `mcpServers`：一个 stdio `{id, transport?, command}` 或 HTTP `{id, transport:"http", url}` 条目，以及其 argv 所指名的任何相对命令源文件；
- `reminders`: `{id, path, tools?, blocking?, decision, defaultPriority?,
  maxPriority?, maxChildSteps?, maxInstallsPerRun?, reasoningEffort?, context?}`
  and a UTF-8 duty file. Use the executable decision object, including its V1
  `envelope` and `deliveryRole`, from `references/capability-examples.json`.
  `reminders`：`{id, path, tools?, blocking?, decision, defaultPriority?,
  maxPriority?, maxChildSteps?, maxInstallsPerRun?, reasoningEffort?, context?}`，以及一个 UTF-8 职责文件。使用 `references/capability-examples.json` 中可执行的决策对象，包括其 V1 的 `envelope` 和 `deliveryRole`。

## Reserve The Destination / 预留目标目录

No artifact write may occur before all reservation steps succeed:

在所有预留步骤成功之前，不得发生任何工件写入：

1. Resolve the current workspace and requested parent to canonical paths.
   把当前工作区和所请求的父目录解析为规范路径。
2. Require the parent and proposed destination to remain inside the workspace.
   要求父目录和拟议目标目录始终位于工作区内。
3. Reject a symlinked destination or a parent whose canonical identity escapes.
   拒绝符号链接的目标目录，或规范化身份逃逸出工作区的父目录。
4. Confirm the destination does not exist.
   确认目标目录不存在。
5. Through the managed shell tool advertised by this session, use one
   platform-native atomic no-replace directory creation operation. Do not use a
   check-then-overwrite command.
   通过本会话宣告的受管 shell 工具，使用一次平台原生的、原子的、不可替换（no-replace）目录创建操作。不要使用"先检查再覆盖"的命令。
6. If creation reports occupied or fails, stop. Never delete or reuse the path.
   若创建报告已被占用或失败，停止。绝不删除或复用该路径。
7. Canonicalize the new directory and its parent again. Require the same expected
   identity and workspace containment before the first `write_file` call.
   再次把新目录及其父目录规范化。在第一次 `write_file` 调用之前，要求具有同样的预期身份和工作区包含关系。

Use `write_file` and `edit_file` for UTF-8 artifacts. Do not use shell redirection
to bypass file-tool containment. After reservation, mutate only the new directory.

UTF-8 工件使用 `write_file` 和 `edit_file`。不要用 shell 重定向绕过文件工具的包含约束。预留之后，只变更新目录。

## Validate And Correct / 验证与修正

Validate every generated skill directory first. Invoke the installed program with
the equivalent structured argv. This is the `muse skills validate` operation;
keep its arguments structured:

先验证每个生成的技能目录。用等价的结构化 argv 调用已安装的程序。这就是 `muse skills validate` 操作；保持其参数结构化：

```json
["muse", "skills", "validate", "<skill-directory>", "--json"]
```

Then run the `muse plugins validate` operation with structured arguments:

然后用结构化参数运行 `muse plugins validate` 操作：

```json
["muse", "plugins", "validate", "<plugin-directory>", "--json"]
```

A validation result is clean only when the process starts, exits zero, stdout is
parseable JSON, top-level `valid` is `true`, and `diagnostics` is present and empty.
Warnings are not clean. A missing command, malformed JSON, timeout, denial,
cancellation, non-zero exit, `valid:false`, or any diagnostic leaves the draft
incomplete.

只有当进程启动、退出码为零、stdout 可解析为 JSON、顶层 `valid` 为 `true`、且 `diagnostics` 存在并为空时，验证结果才算干净。警告不算干净。命令缺失、JSON 畸形、超时、被拒绝、被取消、非零退出、`valid:false` 或任何诊断输出，都会使草稿处于未完成状态。

For each validation layer, make at most three correction rounds after its first
failed result. Edit only the new draft and rerun the same layer. The fourth failed
result is terminal for that layer. Never continue to whole-plugin validation while
a nested skill is invalid.

对每个验证层，在其首次失败结果之后最多进行三轮修正。只编辑新草稿并重跑同一层。第四次失败结果对该层是终局的。只要嵌套技能仍无效，就绝不继续整插件验证。

## Report / 报告

On success, report:

成功时，报告：

- the canonical artifact path;
  规范化的工件路径；
- the capability and file inventory;
  能力与文件清单；
- the number and outcome of nested skill validations;
  嵌套技能验证的次数与结果；
- the whole-plugin validation outcome;
  整插件验证的结果；
- an unexecuted install proposal as structured `program` and `argv` values:
  一个未执行的安装提案，以结构化的 `program` 和 `argv` 值表示：

```json
{
  "program": "muse",
  "argv": ["plugins", "install", "<canonical-plugin-path>"],
  "executed": false
}
```

Explicitly state that installation was not run.

明确声明未运行安装。

On failure or denial after reservation, report the draft path, exact failed
operation, remaining diagnostics, and checks not completed. Do not claim the draft
is created, valid, complete, or ready to install. Cancellation ends the run without
a later report; preserve all pre-existing bytes and let runtime cancellation remain
authoritative.

预留之后若失败或被拒绝，报告草稿路径、确切的失败操作、剩余诊断，以及未完成的检查。不要声称草稿已创建、有效、完整或可安装。取消会终结本次运行且不再有后续报告；保留所有既有字节，让运行时的取消保持权威。
