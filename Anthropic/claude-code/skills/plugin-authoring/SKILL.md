---
name: plugin-authoring
description: |-
  Make a mod: a live pane, band, status line, toast or hook inside Claude Code (terminal or desktop Code tab), written as a plugin of function hooks that hot-reloads in this session. Load before writing or debugging a hooks module.
---
<!-- BILINGUAL-EN-ZH -->

WHERE TO WRITE IT. Write each mod in its own child folder of `/Users/asgeirtj/.claude/dev-mods/6c004cfe-1d2e-49dc-904a-37f8db6d0a99`: `/Users/asgeirtj/.claude/dev-mods/6c004cfe-1d2e-49dc-904a-37f8db6d0a99/<mod-name>/`, three files written directly:

写到哪。把每个 mod 写在 `/Users/asgeirtj/.claude/dev-mods/6c004cfe-1d2e-49dc-904a-37f8db6d0a99` 下它自己的子文件夹中：`/Users/asgeirtj/.claude/dev-mods/6c004cfe-1d2e-49dc-904a-37f8db6d0a99/<mod-name>/`，直接写入三个文件：

- `.claude-plugin/plugin.json`: `{ "name": "<mod-name>", "version": "0.1.0", "description": "<one line>" }`
  `.claude-plugin/plugin.json`：`{ "name": "<mod-name>", "version": "0.1.0", "description": "<one line>" }`
- `hooks/hooks.json`: `{ "modules": ["./register.tsx"] }`, one path, relative to that file
  `hooks/hooks.json`：`{ "modules": ["./register.tsx"] }`，一个路径，相对于该文件
- `hooks/register.tsx` (or `.ts`): the hooks module, `export const register: Register = (on, options) => { ... }`, the type imported from `'claude-code'`
  `hooks/register.tsx`（或 `.ts`）：hooks 模块，`export const register: Register = (on, options) => { ... }`，类型从 `'claude-code'` 导入

A mod that keeps values in `$.state` has a fourth file, `types/index.d.ts`: its type contract, declaring each value in `interface PluginState` under the mod's name, named in `plugin.json` as `"types": "./types/index.d.ts"`. The module imports its value types from `'../types'`, and `claude plugin validate` holds every `$.state` key the module names to that contract.

把值保存在 `$.state` 中的 mod 还有第四个文件 `types/index.d.ts`：即它的类型契约，以 mod 的名字在 `interface PluginState` 中声明每个值，并在 `plugin.json` 中以 `"types": "./types/index.d.ts"` 指明。模块从 `'../types'` 导入其值类型，`claude plugin validate` 会把模块提到的每个 `$.state` 键约束到该契约上。

WHERE THE TYPES ARE. The engine writes them; there is no command to run. Before the mod has loaded: `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/types/claude-code.d.ts`, this build's API and built-in tools, written as this skill loaded (this process's own folder: after a restart the next load of this skill writes and names a new one). Once the engine has loaded the mod (hot reloading enabled for this session, or a `--plugin-dir` folder), `<mod folder>/.claude-plugin/types/` holds the same for the editor: `claude-code/index.d.ts`, the API; `claude-code-tools/index.d.ts`, so `e.tool === "Bash"` narrows `e`; `claude-code-mcp/index.d.ts`, the MCP tools connected when the mod last reloaded; each `dependencies` plugin's contract; and a `tsconfig.json` the mod's own extends, so `tsc -p <mod folder>` type-checks it. The API file carries every event's input and result, every noun and method on `$` with its doc comment and an example, and every element's props, in about 14,000 lines: grep it for the name at hand (`'tool.call'`, `Pane: {`, `export type ToolCallResult`) and read the declaration it lands on.

类型在哪。由引擎写入；没有需要运行的命令。mod 加载之前：`/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/types/claude-code.d.ts`，即本构建的 API 与内置工具，在本技能加载时写入（本进程自己的文件夹：重启后下一次加载本技能会写入并命名一个新文件）。一旦引擎加载了 mod（本会话启用了热重载，或是 `--plugin-dir` 文件夹），`<mod folder>/.claude-plugin/types/` 会为编辑器保存同样的内容：`claude-code/index.d.ts`，API 本体；`claude-code-tools/index.d.ts`，使 `e.tool === "Bash"` 能收窄 `e` 的类型；`claude-code-mcp/index.d.ts`，mod 最近一次重载时连接的 MCP 工具；每个 `dependencies` 插件的契约；以及一个供 mod 自己 extends 的 `tsconfig.json`，因此 `tsc -p <mod folder>` 可对它做类型检查。API 文件以约 14,000 行的篇幅承载每个事件的输入与结果、`$` 上每个名词与方法及其文档注释和示例、以及每个元素的 props：对眼前的名字 grep 它（`'tool.call'`、`Pane: {`、`export type ToolCallResult`），然后阅读落点处的声明。

WHAT HAPPENS WHEN THE TURN ENDS. Loading this skill through the Skill tool or its slash command starts the engine's watch on `/Users/asgeirtj/.claude/dev-mods/6c004cfe-1d2e-49dc-904a-37f8db6d0a99`. The first file written there makes the engine ask the person, once, right then, while the turn goes on: "Enable hot reloading for this session?", with `How does this work?` first, then `Enable for this session` and `Not now`. That question is the switch, and the person alone answers it: no permission mode, rule or hook does. On `Enable for this session` the folder joins the session's plugin folders and the mod loads when the turn ends, whole, and each later edit reloads it when the turn that made the edit ends. The answer reaches you as a notice at the start of your next turn, one of: enabled, with what the load came to; declined (the mods are written and load the next time this session starts; the person can ask for the question again); still open (a new prompt from the person takes the question down, and the engine asks again when that turn ends); or off, with the reason (nobody could be asked, as under `claude -p`; an organization's policy; an untrusted workspace). A process that restarted loads an enabled folder again by itself; otherwise its watch starts the next time this skill loads, and a manifest already in the folder raises the question when that turn ends.

回合结束时会发生什么。通过 Skill 工具或其斜杠命令加载本技能，会启动引擎对 `/Users/asgeirtj/.claude/dev-mods/6c004cfe-1d2e-49dc-904a-37f8db6d0a99` 的监视。写入该处的第一个文件会让引擎当场、且仅一次地在回合进行中询问用户："Enable hot reloading for this session?"，先有 `How does this work?`，然后是 `Enable for this session` 与 `Not now`。这个问题就是开关，且只有用户本人能回答：任何权限模式、规则或 hook 都不能代答。选择 `Enable for this session` 后，该文件夹加入会话的插件文件夹，mod 在回合结束时整体加载；此后每次编辑都会在做出该编辑的回合结束时重载。答案会在你下一个回合开始时以通知的形式到达，可能是以下之一：已启用，并附加载结果；已拒绝（mod 已写入，将在本会话下次启动时加载；用户可以再次要求弹出该问题）；仍然未决（用户的新提示会让该问题消失，引擎会在该回合结束时再次询问）；或已关闭，并附原因（无法询问任何人，如在 `claude -p` 下；组织策略；不受信任的工作区）。重启过的进程会自行再次加载已启用的文件夹；否则其监视会在本技能下次加载时启动，而文件夹中已存在的 manifest 会在该回合结束时再次引出那个问题。

【评论】把启用热重载的决定权排他性地交给人类用户，并明确排除权限模式、规则或 hook 代答，是一种针对"代码自动执行自身"风险的防护设计。

A reload is a fresh load of the module: `register` runs again and `session.start` fires again. Values in `$.state` (the session's) and `$.store` (across sessions) are the host's and stay; the module's own variables start over.

一次重载就是模块的一次全新加载：`register` 重新运行，`session.start` 重新触发。`$.state`（会话内的）与 `$.store`（跨会话的）中的值属于宿主并得以保留；模块自己的变量则从头开始。

## A mod in one paragraph / 一段话讲完一个 mod

`on(event, matcher?, hook)` adds a hook, and every hook is `($, e, next)`: `$` is the engine interface, each call spelled noun then method; `e` is the event's input, a plain frozen value; `next(e)` runs the plugins beneath and then the engine's own behaviour, resolving to the event's result. A hook that returns without `next` answers for itself; `next({ ...e, x })` rewrites what the rest sees. The module runs in an environment of its own, with no DOM and no Node: `$` reaches everything outside it. JSX compiles against the global `h`, and the elements come from the drawing surface's own table, `const { Box, Text, Button } = $.ui.resolve(e)`, where `e.surface` is `terminal`, `desktop`, `vscode` or `mobile`.

`on(event, matcher?, hook)` 添加一个 hook，每个 hook 都是 `($, e, next)`：`$` 是引擎接口，每次调用都按"名词后接方法"拼写；`e` 是事件的输入，一个普通的冻结值；`next(e)` 运行其下的插件、再运行引擎自身的行为，最终解析为该事件的结果。不调用 `next` 直接返回的 hook 由自己给出答案；`next({ ...e, x })` 则改写后续环节所见。模块运行在自己的环境中，没有 DOM 也没有 Node：`$` 可触达其外的一切。JSX 针对全局 `h` 编译，元素来自绘制表面自己的表：`const { Box, Text, Button } = $.ui.resolve(e)`，其中 `e.surface` 为 `terminal`、`desktop`、`vscode` 或 `mobile`。

## From the ask to the shape / 从需求到形态

Each example is one complete hooks module; with the two JSON files above, and its contract where it has one, it is a mod that loads, validates and type-checks on this build.

每个示例都是一个完整的 hooks 模块；配合上面的两个 JSON 文件、以及（如有）它的契约，它就是一个能在本构建上加载、通过校验并通过类型检查的 mod。

| The person asks for | What it is | Shown in |
| --- | --- | --- |
| a pane, panel, sidebar, live view | `$.ui.open({ id, title })`, drawn by a `ui.render` hook on `{ component: 'Pane', requestId: id }`; opened by something the person did (a command they typed, a Button they pressed) it seats at any width; opened unasked (from `session.start`, a timer) it seats from 144 terminal columns and waits below that | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane.tsx`, its contract `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane-state.d.ts` |
| a band or row above the prompt | a `ui.render` hook on `{ component: 'AbovePrompt' }` returning a tree, or `next(e)` with nothing to show | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band.tsx`, its contract `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band-state.d.ts` |
| a status line entry | `$.ui.status(text)` from any hook; `undefined` clears it | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/tool-call.ts` |
| a toast | `$.ui.toast(text)` from any hook | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band.tsx` |
| block, rewrite or react to a tool call | `on('tool.call', { tool }, hook)`: return `{ deny }`, call `next({ ...e, ... })`, or `await next(e)` and act on the result | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/tool-call.ts` |
| change or react to a prompt | `on('prompt.submit', hook)`: `next({ ...e, text })` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band.tsx` |
| change the system prompt | `on('prompt.compose', hook)`: answer `next(e)`'s `{ sections }` with one added last (`scope: 'session'`), replaced or dropped | its doc in the types |
| play a sound | `$.audio.play({ asset })` from any hook, the asset a file of the mod's | its doc in the types |
| a slash command | `$.command.register({ name, description })` in `session.start`, answered by a `command.run` hook returning `{ text }` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane.tsx` |
| values a drawing reads | `atom(ref, initial)`, `read($, atom)` while drawing, `update($, atom, fn)` from a handler or another event; the write redraws the readers; each value declared in the contract | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane-state.d.ts` |
| work on a timer, a tool the model calls, a subagent type, model calls, files, processes | `$.clock`, `$.tool`, `$.agent`, `$.model`, `$.fs`, `$.process` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/reference.md` |

| 用户想要 | 它是什么 | 示例见 |
| --- | --- | --- |
| 面板（pane）、panel、侧边栏、实时视图 | `$.ui.open({ id, title })`，由一个订阅 `{ component: 'Pane', requestId: id }` 的 `ui.render` hook 绘制；由用户的行为打开（他们输入的命令、按下的 Button）时可以任意宽度驻留；未被请求而打开（来自 `session.start`、定时器）时从 144 终端列起驻留，低于该值则等待 | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane.tsx`，其契约见 `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane-state.d.ts` |
| 提示符上方的带状区或行 | 一个订阅 `{ component: 'AbovePrompt' }` 的 `ui.render` hook，返回一棵树；或无可显示时调用 `next(e)` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band.tsx`，其契约见 `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band-state.d.ts` |
| 状态行条目 | 任意 hook 中调用 `$.ui.status(text)`；传 `undefined` 清除 | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/tool-call.ts` |
| toast 提示 | 任意 hook 中调用 `$.ui.toast(text)` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band.tsx` |
| 阻止、改写或响应工具调用 | `on('tool.call', { tool }, hook)`：返回 `{ deny }`、调用 `next({ ...e, ... })`，或 `await next(e)` 后对结果采取行动 | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/tool-call.ts` |
| 修改或响应提示 | `on('prompt.submit', hook)`：`next({ ...e, text })` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/band.tsx` |
| 修改系统提示词 | `on('prompt.compose', hook)`：应答 `next(e)` 的 `{ sections }`，可最后追加一个（`scope: 'session'`）、替换或丢弃 | 类型文件中的文档 |
| 播放声音 | 任意 hook 中调用 `$.audio.play({ asset })`，asset 为 mod 自己的文件 | 类型文件中的文档 |
| 斜杠命令 | 在 `session.start` 中调用 `$.command.register({ name, description })`，由一个返回 `{ text }` 的 `command.run` hook 应答 | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane.tsx` |
| 绘制所读取的值 | `atom(ref, initial)`，绘制时 `read($, atom)`，从处理器或其他事件中 `update($, atom, fn)`；写入会触发读取者重绘；每个值都在契约中声明 | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/examples/pane-state.d.ts` |
| 定时器上的工作、模型调用的工具、子智能体类型、模型调用、文件、进程 | `$.clock`、`$.tool`、`$.agent`、`$.model`、`$.fs`、`$.process` | `/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/reference.md` |

## Checking it and reading what the engine refused / 校验它，并读懂引擎拒绝了什么

`claude plugin validate <mod folder>` reads the manifest and the module's source the way the engine will, and reports what the module hooks and calls and all the engine would refuse. Type-check the mod: once it has loaded, `tsc -p <mod folder>`; where nothing is laid (the first write, or `claude -p`), with the `tsconfig.json` from the types file's header, kept outside the mod folder, its `include` naming that file and the mod's `hooks`. Write at least one `*.test.ts` for the behaviour asked for and run `claude plugin test <mod folder>`. Then give the person the mod's full path and how it loads: enabled, nothing to run; off as nobody could be asked, in a terminal, `claude --plugin-dir` and that path; off otherwise, nothing loads it until that changes.

`claude plugin validate <mod folder>` 会以引擎将来读取的方式读取 manifest 与模块源码，并报告模块的 hook 与调用中、以及一切会被引擎拒绝之处。对 mod 做类型检查：加载之后，`tsc -p <mod folder>`；在没有任何落盘之处（首次写入，或 `claude -p`）则使用 types 文件头部给出的 `tsconfig.json`，保存在 mod 文件夹之外，其 `include` 指明该文件与 mod 的 `hooks`。针对所要求的行为至少写一个 `*.test.ts`，并运行 `claude plugin test <mod folder>`。然后把 mod 的完整路径及其加载方式告诉用户：已启用，无需任何操作；因无法询问而关闭时，在终端用 `claude --plugin-dir` 加该路径；其他关闭情形下，在情况改变前不会有任何东西加载它。

While the session hot-reloads a plugin folder the transcript carries one dim line naming the plugin, the event and the reason when a hook fails or a module does not load, and says when a hook's tree did not validate and the engine drew its own instead: `<plugin>: ui.render (<Component>) refused: <reason>; the engine drew its own`, `<plugin>` being the one plugin whose hook could have drawn the tree, `hooks` otherwise. In any other session those lines go to the debug log alone, as `<plugin>: <line>`. The debug log (`claude --debug`) carries a line for every occurrence and every result the engine refused; in every session such a tree has a line there beginning `ui.render (<Component>): a hook returned a tree that does not validate`, followed by the reason.

当会话热重载某个插件文件夹时，若某个 hook 失败或模块未加载，记录（transcript）中会有一条暗色行，点名插件、事件与原因；当某个 hook 的树未通过校验、引擎改画自己的树时也会说明：`<plugin>: ui.render (<Component>) refused: <reason>; the engine drew its own`，其中 `<plugin>` 是其 hook 本可能画出该树的那个插件，否则为 `hooks`。在任何其他会话中，这些行只进入调试日志，形如 `<plugin>: <line>`。调试日志（`claude --debug`）为每一次出现、以及每一个被引擎拒绝的结果都保留一行；在每个会话中，这样的树在日志里都有一行以 `ui.render (<Component>): a hook returned a tree that does not validate` 开头、后接原因。

`/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/reference.md` is the long form: the full event list and streaming events, `ui.render` in depth, `$.state` contracts, `--plugin-dir` and `CLAUDE_CODE_PLUGIN_DIRS`, `userConfig` options, and what a test holds.

`/private/tmp/claude-501/bundled-skills/2.1.288/90955cd459120f476f04c5ab6d943d7b/plugin-authoring/reference.md` 是长篇版本：完整事件列表与流式事件、`ui.render` 深度解析、`$.state` 契约、`--plugin-dir` 与 `CLAUDE_CODE_PLUGIN_DIRS`、`userConfig` 选项，以及测试所应覆盖的内容。
