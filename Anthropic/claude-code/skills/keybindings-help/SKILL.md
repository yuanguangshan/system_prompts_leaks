---
name: keybindings-help
description: |-
  Use when the user wants to customize keyboard shortcuts, rebind keys, add chord bindings, or modify ~/.claude/keybindings.json. Examples: "rebind ctrl+s", "add a chord shortcut", "change the submit key", "customize keybindings".
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Keybindings Skill / 键盘快捷键技能

Create or modify `~/.claude/keybindings.json` to customize keyboard shortcuts.

创建或修改 `~/.claude/keybindings.json` 以自定义键盘快捷键。

## CRITICAL: Read Before Write / 关键：先读后写

**Always read `~/.claude/keybindings.json` first** (it may not exist yet). Merge changes with existing bindings — never replace the entire file.

**始终先读取 `~/.claude/keybindings.json`**（它可能尚不存在）。将更改与既有绑定合并——绝不整体替换该文件。

- Use **Edit** tool for modifications to existing files
  修改既有文件时使用 **Edit** 工具
- Use **Write** tool only if the file does not exist yet
  仅当文件尚不存在时才使用 **Write** 工具

## File Format / 文件格式

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor"
      }
    }
  ]
}
```

Always include the `$schema` and `$docs` fields.

始终包含 `$schema` 和 `$docs` 字段。

## Keystroke Syntax / 按键语法

**Modifiers** (combine with `+`):

**修饰键**（用 `+` 组合）：

- `ctrl` (alias: `control`)
  `ctrl`（别名：`control`）
- `alt` (aliases: `opt`, `option`) — note: `alt` and `meta` are identical in terminals
  `alt`（别名：`opt`、`option`）——注意：在终端中 `alt` 与 `meta` 完全相同
- `shift`
  `shift`
- `meta` (aliases: `cmd`, `command`)
  `meta`（别名：`cmd`、`command`）

**Special keys**: `escape`/`esc`, `enter`/`return`, `tab`, `space`, `backspace`, `delete`, `up`, `down`, `left`, `right`

**特殊键**：`escape`/`esc`、`enter`/`return`、`tab`、`space`、`backspace`、`delete`、`up`、`down`、`left`、`right`

**Chords**: Space-separated keystrokes, e.g. `ctrl+k ctrl+s` (1-second timeout between keystrokes)

**组合键（chord）**：以空格分隔的按键序列，例如 `ctrl+k ctrl+s`（两次按键之间超时 1 秒）

**Examples**: `ctrl+shift+p`, `alt+enter`, `ctrl+k ctrl+n`

**示例**：`ctrl+shift+p`、`alt+enter`、`ctrl+k ctrl+n`

## Unbinding Default Shortcuts / 解绑默认快捷键

Set a key to `null` to remove its default binding:

将某个键设为 `null` 即可移除其默认绑定：

```json
{
  "context": "Chat",
  "bindings": {
    "ctrl+s": null
  }
}
```

## How User Bindings Interact with Defaults / 用户绑定与默认绑定的交互方式

- User bindings are **additive** — they are appended after the default bindings
  用户绑定是**追加式**的——它们被添加在默认绑定之后
- To **move** a binding to a different key: unbind the old key (`null`) AND add the new binding
  要把绑定**移动**到另一个键：先解绑旧键（设为 `null`），再添加新绑定
- A context only needs to appear in the user's file if they want to change something in that context
  只有当用户想更改某个上下文中的内容时，该上下文才需要出现在用户文件中
  【评论】"追加式 + 显式解绑"的模型让用户文件只需声明与默认的差异，避免覆盖式配置互相吞并。

## Common Patterns / 常见模式

### Rebind a key / 重新绑定一个键
To change the external editor shortcut from `ctrl+g` to `ctrl+e`:
要把外部编辑器快捷键从 `ctrl+g` 改为 `ctrl+e`：
```json
{
  "context": "Chat",
  "bindings": {
    "ctrl+g": null,
    "ctrl+e": "chat:externalEditor"
  }
}
```

### Add a chord binding / 添加组合键绑定
```json
{
  "context": "Global",
  "bindings": {
    "ctrl+k ctrl+t": "app:toggleTodos"
  }
}
```

## Behavioral Rules / 行为规则

1. Only include contexts the user wants to change (minimal overrides)
   只包含用户想更改的上下文（最小化覆盖）
2. Validate that actions and contexts are from the known lists below
   校验动作和上下文是否来自下方的已知列表
3. Warn the user proactively if they choose a key that conflicts with reserved shortcuts or common tools like tmux (`ctrl+b`) and screen (`ctrl+a`)
   如果用户选择的键与保留快捷键或 tmux（`ctrl+b`）、screen（`ctrl+a`）等常用工具冲突，应主动提醒
4. When adding a new binding for an existing action, the new binding is additive (existing default still works unless explicitly unbound)
   为既有动作添加新绑定时，新绑定是追加式的（既有默认绑定仍然生效，除非显式解绑）
5. To fully replace a default binding, unbind the old key AND add the new one
   要完全替换某个默认绑定，需解绑旧键并添加新绑定

## Validation / 校验

Claude Code validates `~/.claude/keybindings.json` when it loads; warnings go to the debug log. After editing the file, re-check it against the rules below and fix anything that matches.

Claude Code 在加载时会校验 `~/.claude/keybindings.json`；警告会写入调试日志。编辑该文件后，按下方规则重新检查并修复所有匹配的问题。

### Common Issues and Fixes / 常见问题与修复

| Issue | Cause | Fix |
| --- | --- | --- |
| `keybindings.json must have a "bindings" array` | Missing wrapper object | Wrap bindings in `{ "bindings": [...] }` |
| `"bindings" must be an array` | `bindings` is not an array | Set `"bindings"` to an array: `[{ context: ..., bindings: ... }]` |
| `Unknown context "X"` | Typo or invalid context name | Use exact context names from the Available Contexts table |
| `Duplicate key "X" in Y bindings` | Same key defined twice in one context | Remove the duplicate; JSON uses only the last value |
| `"X" may not work: ...` | Key conflicts with terminal/OS reserved shortcut | Choose a different key (see Reserved Shortcuts section) |
| `Invalid action for "X"` | Action value is not a string or null | Actions must be strings like `"app:help"` or `null` to unbind |

| 问题 | 原因 | 修复方法 |
| --- | --- | --- |
| `keybindings.json must have a "bindings" array` | 缺少外层包装对象 | 把绑定包在 `{ "bindings": [...] }` 中 |
| `"bindings" must be an array` | `bindings` 不是数组 | 将 `"bindings"` 设为数组：`[{ context: ..., bindings: ... }]` |
| `Unknown context "X"` | 上下文名拼写错误或无效 | 使用"可用上下文"表中的确切上下文名 |
| `Duplicate key "X" in Y bindings` | 同一键在同一上下文中定义了两次 | 移除重复项；JSON 只使用最后一个值 |
| `"X" may not work: ...` | 该键与终端/操作系统保留快捷键冲突 | 换一个键（见"保留快捷键"一节） |
| `Invalid action for "X"` | 动作值不是字符串或 null | 动作必须是 `"app:help"` 这类字符串，或为解绑而设为 `null` |

### Example validation warnings (debug log) / 校验警告示例（调试日志）

```
[keybindings] Found 2 validation issue(s)
[keybindings] [error] Unknown context "chat" — Valid contexts: Global, Chat, Autocomplete, ...
[keybindings] [warning] "ctrl+c" may not work: Terminal interrupt (SIGINT)
```

**Errors** prevent bindings from working and must be fixed. **Warnings** indicate potential conflicts but the binding may still work.

**错误（error）**会使绑定无法工作，必须修复。**警告（warning）**表示存在潜在冲突，但绑定可能仍然有效。

## Reserved Shortcuts / 保留快捷键

### Non-rebindable (errors) / 不可重绑定（报错）
- `ctrl+c` — Cannot be rebound - used for interrupt/exit (hardcoded)
  `ctrl+c` —— 不可重绑定——用于中断/退出（硬编码）
- `ctrl+d` — Cannot be rebound - used for exit (hardcoded)
  `ctrl+d` —— 不可重绑定——用于退出（硬编码）
- `ctrl+m` — Cannot be rebound - identical to Enter in terminals (both send CR)
  `ctrl+m` —— 不可重绑定——在终端中与 Enter 相同（都发送 CR）
- `ctrl+[` — Cannot be rebound - identical to Escape in terminals
  `ctrl+[` —— 不可重绑定——在终端中与 Escape 相同
- `ctrl+i` — Cannot be rebound - identical to Tab in terminals
  `ctrl+i` —— 不可重绑定——在终端中与 Tab 相同
- `ctrl+h` — Cannot be rebound - identical to Backspace in terminals
  `ctrl+h` —— 不可重绑定——在终端中与 Backspace 相同
- `capslock` — Caps Lock is not delivered to terminal applications
  `capslock` —— Caps Lock 不会传递给终端应用

### Terminal reserved (errors/warnings) / 终端保留（报错/警告）
- `ctrl+z` — Unix process suspend (SIGTSTP) (may conflict)
  `ctrl+z` —— Unix 进程挂起（SIGTSTP）（可能冲突）
- `ctrl+\` — Terminal quit signal (SIGQUIT) (will not work)
  `ctrl+\` —— 终端退出信号（SIGQUIT）（不会生效）

### macOS reserved (errors) / macOS 保留（报错）
- `cmd+c` — macOS system copy
  `cmd+c` —— macOS 系统复制
- `cmd+v` — macOS system paste
  `cmd+v` —— macOS 系统粘贴
- `cmd+x` — macOS system cut
  `cmd+x` —— macOS 系统剪切
- `cmd+q` — macOS quit application
  `cmd+q` —— macOS 退出应用
- `cmd+w` — macOS close window/tab
  `cmd+w` —— macOS 关闭窗口/标签页
- `cmd+tab` — macOS app switcher
  `cmd+tab` —— macOS 应用切换器
- `cmd+space` — macOS Spotlight
  `cmd+space` —— macOS Spotlight

## Available Contexts / 可用上下文

| Context | Description |
| --- | --- |
| `Global` | Active everywhere, regardless of focus |
| `Chat` | When the chat input is focused |
| `Autocomplete` | When autocomplete menu is visible |
| `Confirmation` | When a confirmation/permission dialog is shown |
| `Help` | When the help overlay is open |
| `Transcript` | When viewing the transcript |
| `HistorySearch` | When searching command history (ctrl+r) |
| `Task` | When a task/agent is running in the foreground |
| `ThemePicker` | When the theme picker is open |
| `Settings` | When the settings menu is open |
| `Tabs` | When tab navigation is active |
| `Attachments` | When navigating image attachments in a select dialog |
| `Footer` | When footer indicators are focused |
| `AbovePrompt` | When a plugin's panel above the prompt, or a button in it, has keyboard focus |
| `AbovePromptInput` | When a plugin's input field above the prompt has keyboard focus |
| `AbovePromptSelect` | When a plugin's select above the prompt has keyboard focus |
| `Pane` | When a plugin's pane has keyboard focus |
| `PaneField` | When an input field or select in a plugin's pane has keyboard focus |
| `MessageSelector` | When the message selector (rewind) is open |
| `DiffDialog` | When the diff dialog is open |
| `DiffPanel` | When the diff sidebar panel is open |
| `ModelPicker` | When the model picker is open |
| `EffortSlider` | When the effort slider is open |
| `Select` | When a select/list component is focused |
| `Plugin` | When the plugin dialog is open |
| `Scroll` | When a scrollable view is focused (fullscreen layout) |
| `Agents` | When the agents view (`claude agents`) is open |

| 上下文 | 说明 |
| --- | --- |
| `Global` | 在任何地方都生效，与焦点无关 |
| `Chat` | 聊天输入框获得焦点时 |
| `Autocomplete` | 自动补全菜单可见时 |
| `Confirmation` | 显示确认/权限对话框时 |
| `Help` | 帮助浮层打开时 |
| `Transcript` | 查看会话记录时 |
| `HistorySearch` | 搜索命令历史时（ctrl+r） |
| `Task` | 有任务/智能体在前台运行时 |
| `ThemePicker` | 主题选择器打开时 |
| `Settings` | 设置菜单打开时 |
| `Tabs` | 标签页导航激活时 |
| `Attachments` | 在选择对话框中浏览图片附件时 |
| `Footer` | 底部状态栏指示器获得焦点时 |
| `AbovePrompt` | 提示框上方的插件面板或其中的按钮获得键盘焦点时 |
| `AbovePromptInput` | 提示框上方的插件输入框获得键盘焦点时 |
| `AbovePromptSelect` | 提示框上方的插件下拉框获得键盘焦点时 |
| `Pane` | 插件面板获得键盘焦点时 |
| `PaneField` | 插件面板中的输入框或下拉框获得键盘焦点时 |
| `MessageSelector` | 消息选择器（rewind）打开时 |
| `DiffDialog` | 差异对话框打开时 |
| `DiffPanel` | 差异侧边栏面板打开时 |
| `ModelPicker` | 模型选择器打开时 |
| `EffortSlider` | 努力程度滑块打开时 |
| `Select` | 下拉/列表组件获得焦点时 |
| `Plugin` | 插件对话框打开时 |
| `Scroll` | 可滚动视图获得焦点时（全屏布局） |
| `Agents` | 智能体视图（`claude agents`）打开时 |

## Available Actions / 可用动作

| Action | Default Key(s) | Context |
| --- | --- | --- |
| `app:interrupt` | `ctrl+c` | Global |
| `app:exit` | `ctrl+d` | Global |
| `app:toggleTodos` | `ctrl+t` | Global |
| `app:toggleTranscript` | `ctrl+o` | Global |
| `app:toggleBrief` | `ctrl+shift+b` | Global |
| `app:toggleReplTab` | (none) | Global |
| `app:toggleDiffNoiseFilter` | (none) | Global |
| `app:diffFileListUp` | `meta+up` | Global |
| `app:diffFileListDown` | `meta+down` | Global |
| `app:toggleDiffPreSession` | (none) | Global |
| `app:cycleDiffBase` | `ctrl+x b` | DiffPanel |
| `app:toggleTerminal` | (none) | Global |
| `app:redraw` | (none) | Global |
| `app:openArtifact` | `ctrl+]` | Global |
| `history:search` | `ctrl+r` | Global |
| `history:previous` | `up` | Chat |
| `history:next` | `down` | Chat |
| `chat:cancel` | `escape` | Chat |
| `chat:killAgents` | `ctrl+x ctrl+k` | Chat |
| `chat:cycleMode` | `shift+tab` | Chat |
| `chat:modelPicker` | `meta+p` | Chat |
| `chat:fastMode` | `meta+o` | Chat |
| `chat:thinkingToggle` | `meta+t` | Chat |
| `chat:workflowKeywordToggle` | `meta+w` | Chat |
| `chat:submit` | `enter` | Chat |
| `chat:queueSubmit` | `ctrl+x enter` | Chat |
| `chat:newline` | `ctrl+j` | Chat |
| `chat:undo` | `ctrl+_`, `ctrl+-`, `ctrl+shift+-`, `ctrl+shift+_` | Chat |
| `chat:externalEditor` | `ctrl+x ctrl+e`, `ctrl+g` | Chat |
| `chat:stash` | `ctrl+s` | Chat |
| `chat:imagePaste` | `ctrl+v` | Chat |
| `chat:clearInput` | `ctrl+l` | Chat |
| `chat:clearScreen` | `cmd+k` | Chat |
| `autocomplete:accept` | `tab` | Autocomplete |
| `autocomplete:dismiss` | `escape` | Autocomplete |
| `autocomplete:previous` | `up` | Autocomplete |
| `autocomplete:next` | `down` | Autocomplete |
| `confirm:yes` | `y`, `enter` | Confirmation |
| `confirm:no` | `escape`, `n`, `escape` | Settings |
| `confirm:previous` | `up` | Confirmation |
| `confirm:next` | `down` | Confirmation |
| `confirm:nextField` | `tab` | Confirmation |
| `confirm:previousField` | (none) | Confirmation |
| `confirm:cycleMode` | `shift+tab` | Confirmation |
| `confirm:toggle` | `space` | Confirmation |
| `tabs:next` | `tab`, `right` | Tabs |
| `tabs:previous` | `shift+tab`, `left` | Tabs |
| `transcript:toggleShowAll` | `ctrl+e` | Transcript |
| `transcript:exit` | `ctrl+c`, `escape`, `q` | Transcript |
| `historySearch:next` | `ctrl+r` | HistorySearch |
| `historySearch:accept` | `escape`, `tab` | HistorySearch |
| `historySearch:cancel` | `ctrl+c` | HistorySearch |
| `historySearch:execute` | `enter` | HistorySearch |
| `historySearch:cycleScope` | `ctrl+s` | HistorySearch |
| `task:background` | `ctrl+x ctrl+b`, `ctrl+b` | Task |
| `theme:toggleSyntaxHighlighting` | `ctrl+t` | ThemePicker |
| `theme:editCustom` | `ctrl+e` | ThemePicker |
| `help:dismiss` | `escape` | Help |
| `attachments:next` | `right` | Attachments |
| `attachments:previous` | `left` | Attachments |
| `attachments:remove` | `backspace`, `delete` | Attachments |
| `attachments:exit` | `down`, `escape` | Attachments |
| `footer:up` | `up`, `ctrl+p` | Footer |
| `footer:down` | `down`, `ctrl+n` | Footer |
| `footer:next` | `right` | Footer |
| `footer:previous` | `left` | Footer |
| `footer:openSelected` | `enter` | Footer |
| `footer:clearSelection` | `escape` | Footer |
| `footer:close` | `x` | Footer |
| `footer:dismiss` | `backspace`, `delete` | Footer |
| `abovePrompt:toggle` | `ctrl+x ctrl+a` | Chat |
| `abovePrompt:focus` | `ctrl+x tab` | Chat |
| `abovePrompt:next` | `tab`, `right`, `tab`, `down`, `tab`, `tab` | AbovePrompt |
| `abovePrompt:previous` | `shift+tab`, `left`, `shift+tab`, `up`, `shift+tab`, `shift+tab` | AbovePrompt |
| `abovePrompt:press` | `enter`, `space`, `enter`, `enter`, `enter` | AbovePrompt |
| `abovePrompt:leave` | `escape`, `escape`, `escape`, `escape` | AbovePrompt |
| `abovePrompt:highlightNext` | `down` | AbovePromptSelect |
| `abovePrompt:highlightPrevious` | `up` | AbovePromptSelect |
| `pane:scrollUp` | `up`, `up` | AbovePrompt |
| `pane:scrollDown` | `down`, `down` | AbovePrompt |
| `pane:pageUp` | `pageup`, `pageup` | AbovePrompt |
| `pane:pageDown` | `pagedown`, `pagedown` | AbovePrompt |
| `pane:top` | `home`, `home` | AbovePrompt |
| `pane:bottom` | `end`, `end` | AbovePrompt |
| `pane:grow` | `ctrl+x left`, `ctrl+x up` | Pane |
| `pane:shrink` | `ctrl+x right`, `ctrl+x down` | Pane |
| `pane:close` | `ctrl+x x`, `ctrl+x x` | Pane |
| `pane:next` | (none) | Pane |
| `pane:previous` | (none) | Pane |
| `messageSelector:up` | `up`, `k`, `ctrl+p` | MessageSelector |
| `messageSelector:down` | `down`, `j`, `ctrl+n` | MessageSelector |
| `messageSelector:top` | `ctrl+up`, `shift+up`, `meta+up`, `shift+k` | MessageSelector |
| `messageSelector:bottom` | `ctrl+down`, `shift+down`, `meta+down`, `shift+j` | MessageSelector |
| `messageSelector:select` | `enter` | MessageSelector |
| `diff:dismiss` | `escape` | DiffDialog |
| `diff:previousSource` | `left` | DiffDialog |
| `diff:nextSource` | `right` | DiffDialog |
| `diff:back` | (none) | DiffDialog |
| `diff:viewDetails` | `enter` | DiffDialog |
| `diff:previousFile` | `up`, `k` | DiffDialog |
| `diff:nextFile` | `down`, `j` | DiffDialog |
| `modelPicker:decreaseEffort` | `left` | ModelPicker |
| `modelPicker:increaseEffort` | `right` | ModelPicker |
| `modelPicker:thisSessionOnly` | `s` | ModelPicker |
| `effortSlider:thisSessionOnly` | `s` | EffortSlider |
| `select:next` | `down`, `j`, `ctrl+n`, `down`, `j`, `ctrl+n` | Settings |
| `select:previous` | `up`, `k`, `ctrl+p`, `up`, `k`, `ctrl+p` | Settings |
| `select:pageUp` | `pageup` | Select |
| `select:pageDown` | `pagedown` | Select |
| `select:first` | `home` | Select |
| `select:last` | `end` | Select |
| `select:accept` | `space`, `enter`, `enter` | Settings |
| `select:cancel` | `escape` | Select |
| `plugin:toggle` | `space` | Plugin |
| `plugin:install` | `i` | Plugin |
| `plugin:favorite` | `f` | Plugin |
| `permission:toggleDebug` | (none) | Confirmation |
| `settings:search` | `/` | Settings |
| `settings:retry` | `r` | Settings |
| `settings:periodDay` | `d` | Settings |
| `settings:periodWeek` | `w` | Settings |
| `settings:sortByTokens` | `t` | Settings |
| `voice:pushToTalk` | `space` | Chat |
| `scroll:previousPrompt` | `ctrl+up`, `ctrl+up` | Transcript |
| `scroll:nextPrompt` | `ctrl+down`, `ctrl+down` | Transcript |
| `scroll:pageUp` | `pageup`, `pageup` | Scroll |
| `scroll:pageDown` | `pagedown`, `pagedown` | Scroll |
| `scroll:lineUp` | `ctrl+p`, `k`, `up`, `wheelup` | Transcript |
| `scroll:lineDown` | `ctrl+n`, `j`, `down`, `wheeldown` | Transcript |
| `scroll:top` | `g`, `home`, `ctrl+home`, `g`, `home` | Transcript |
| `scroll:bottom` | `shift+g`, `end`, `ctrl+end`, `shift+g`, `end` | Transcript |
| `scroll:halfPageUp` | `ctrl+u`, `ctrl+u` | Settings |
| `scroll:halfPageDown` | `ctrl+d`, `ctrl+d` | Settings |
| `scroll:fullPageUp` | `ctrl+b`, `b`, `shift+space`, `b` | Transcript |
| `scroll:fullPageDown` | `ctrl+f`, `space`, `space` | Transcript |
| `selection:copy` | `ctrl+shift+c`, `cmd+c` | Scroll |
| `selection:clear` | (none) | Unknown |
| `selection:extendLeft` | `shift+left` | Scroll |
| `selection:extendRight` | `shift+right` | Scroll |
| `selection:extendUp` | `shift+up` | Scroll |
| `selection:extendDown` | `shift+down` | Scroll |
| `selection:extendLineStart` | `shift+home` | Scroll |
| `selection:extendLineEnd` | `shift+end` | Scroll |
| `agents:switchView` | `ctrl+s` | Agents |
| `agents:togglePin` | `ctrl+t` | Agents |

| 动作 | 默认按键 | 上下文 |
| --- | --- | --- |
| `app:interrupt` | `ctrl+c` | Global |
| `app:exit` | `ctrl+d` | Global |
| `app:toggleTodos` | `ctrl+t` | Global |
| `app:toggleTranscript` | `ctrl+o` | Global |
| `app:toggleBrief` | `ctrl+shift+b` | Global |
| `app:toggleReplTab` | （无） | Global |
| `app:toggleDiffNoiseFilter` | （无） | Global |
| `app:diffFileListUp` | `meta+up` | Global |
| `app:diffFileListDown` | `meta+down` | Global |
| `app:toggleDiffPreSession` | （无） | Global |
| `app:cycleDiffBase` | `ctrl+x b` | DiffPanel |
| `app:toggleTerminal` | （无） | Global |
| `app:redraw` | （无） | Global |
| `app:openArtifact` | `ctrl+]` | Global |
| `history:search` | `ctrl+r` | Global |
| `history:previous` | `up` | Chat |
| `history:next` | `down` | Chat |
| `chat:cancel` | `escape` | Chat |
| `chat:killAgents` | `ctrl+x ctrl+k` | Chat |
| `chat:cycleMode` | `shift+tab` | Chat |
| `chat:modelPicker` | `meta+p` | Chat |
| `chat:fastMode` | `meta+o` | Chat |
| `chat:thinkingToggle` | `meta+t` | Chat |
| `chat:workflowKeywordToggle` | `meta+w` | Chat |
| `chat:submit` | `enter` | Chat |
| `chat:queueSubmit` | `ctrl+x enter` | Chat |
| `chat:newline` | `ctrl+j` | Chat |
| `chat:undo` | `ctrl+_`, `ctrl+-`, `ctrl+shift+-`, `ctrl+shift+_` | Chat |
| `chat:externalEditor` | `ctrl+x ctrl+e`, `ctrl+g` | Chat |
| `chat:stash` | `ctrl+s` | Chat |
| `chat:imagePaste` | `ctrl+v` | Chat |
| `chat:clearInput` | `ctrl+l` | Chat |
| `chat:clearScreen` | `cmd+k` | Chat |
| `autocomplete:accept` | `tab` | Autocomplete |
| `autocomplete:dismiss` | `escape` | Autocomplete |
| `autocomplete:previous` | `up` | Autocomplete |
| `autocomplete:next` | `down` | Autocomplete |
| `confirm:yes` | `y`, `enter` | Confirmation |
| `confirm:no` | `escape`, `n`, `escape` | Settings |
| `confirm:previous` | `up` | Confirmation |
| `confirm:next` | `down` | Confirmation |
| `confirm:nextField` | `tab` | Confirmation |
| `confirm:previousField` | （无） | Confirmation |
| `confirm:cycleMode` | `shift+tab` | Confirmation |
| `confirm:toggle` | `space` | Confirmation |
| `tabs:next` | `tab`, `right` | Tabs |
| `tabs:previous` | `shift+tab`, `left` | Tabs |
| `transcript:toggleShowAll` | `ctrl+e` | Transcript |
| `transcript:exit` | `ctrl+c`, `escape`, `q` | Transcript |
| `historySearch:next` | `ctrl+r` | HistorySearch |
| `historySearch:accept` | `escape`, `tab` | HistorySearch |
| `historySearch:cancel` | `ctrl+c` | HistorySearch |
| `historySearch:execute` | `enter` | HistorySearch |
| `historySearch:cycleScope` | `ctrl+s` | HistorySearch |
| `task:background` | `ctrl+x ctrl+b`, `ctrl+b` | Task |
| `theme:toggleSyntaxHighlighting` | `ctrl+t` | ThemePicker |
| `theme:editCustom` | `ctrl+e` | ThemePicker |
| `help:dismiss` | `escape` | Help |
| `attachments:next` | `right` | Attachments |
| `attachments:previous` | `left` | Attachments |
| `attachments:remove` | `backspace`, `delete` | Attachments |
| `attachments:exit` | `down`, `escape` | Attachments |
| `footer:up` | `up`, `ctrl+p` | Footer |
| `footer:down` | `down`, `ctrl+n` | Footer |
| `footer:next` | `right` | Footer |
| `footer:previous` | `left` | Footer |
| `footer:openSelected` | `enter` | Footer |
| `footer:clearSelection` | `escape` | Footer |
| `footer:close` | `x` | Footer |
| `footer:dismiss` | `backspace`, `delete` | Footer |
| `abovePrompt:toggle` | `ctrl+x ctrl+a` | Chat |
| `abovePrompt:focus` | `ctrl+x tab` | Chat |
| `abovePrompt:next` | `tab`, `right`, `tab`, `down`, `tab`, `tab` | AbovePrompt |
| `abovePrompt:previous` | `shift+tab`, `left`, `shift+tab`, `up`, `shift+tab`, `shift+tab` | AbovePrompt |
| `abovePrompt:press` | `enter`, `space`, `enter`, `enter`, `enter` | AbovePrompt |
| `abovePrompt:leave` | `escape`, `escape`, `escape`, `escape` | AbovePrompt |
| `abovePrompt:highlightNext` | `down` | AbovePromptSelect |
| `abovePrompt:highlightPrevious` | `up` | AbovePromptSelect |
| `pane:scrollUp` | `up`, `up` | AbovePrompt |
| `pane:scrollDown` | `down`, `down` | AbovePrompt |
| `pane:pageUp` | `pageup`, `pageup` | AbovePrompt |
| `pane:pageDown` | `pagedown`, `pagedown` | AbovePrompt |
| `pane:top` | `home`, `home` | AbovePrompt |
| `pane:bottom` | `end`, `end` | AbovePrompt |
| `pane:grow` | `ctrl+x left`, `ctrl+x up` | Pane |
| `pane:shrink` | `ctrl+x right`, `ctrl+x down` | Pane |
| `pane:close` | `ctrl+x x`, `ctrl+x x` | Pane |
| `pane:next` | （无） | Pane |
| `pane:previous` | （无） | Pane |
| `messageSelector:up` | `up`, `k`, `ctrl+p` | MessageSelector |
| `messageSelector:down` | `down`, `j`, `ctrl+n` | MessageSelector |
| `messageSelector:top` | `ctrl+up`, `shift+up`, `meta+up`, `shift+k` | MessageSelector |
| `messageSelector:bottom` | `ctrl+down`, `shift+down`, `meta+down`, `shift+j` | MessageSelector |
| `messageSelector:select` | `enter` | MessageSelector |
| `diff:dismiss` | `escape` | DiffDialog |
| `diff:previousSource` | `left` | DiffDialog |
| `diff:nextSource` | `right` | DiffDialog |
| `diff:back` | （无） | DiffDialog |
| `diff:viewDetails` | `enter` | DiffDialog |
| `diff:previousFile` | `up`, `k` | DiffDialog |
| `diff:nextFile` | `down`, `j` | DiffDialog |
| `modelPicker:decreaseEffort` | `left` | ModelPicker |
| `modelPicker:increaseEffort` | `right` | ModelPicker |
| `modelPicker:thisSessionOnly` | `s` | ModelPicker |
| `effortSlider:thisSessionOnly` | `s` | EffortSlider |
| `select:next` | `down`, `j`, `ctrl+n`, `down`, `j`, `ctrl+n` | Settings |
| `select:previous` | `up`, `k`, `ctrl+p`, `up`, `k`, `ctrl+p` | Settings |
| `select:pageUp` | `pageup` | Select |
| `select:pageDown` | `pagedown` | Select |
| `select:first` | `home` | Select |
| `select:last` | `end` | Select |
| `select:accept` | `space`, `enter`, `enter` | Settings |
| `select:cancel` | `escape` | Select |
| `plugin:toggle` | `space` | Plugin |
| `plugin:install` | `i` | Plugin |
| `plugin:favorite` | `f` | Plugin |
| `permission:toggleDebug` | （无） | Confirmation |
| `settings:search` | `/` | Settings |
| `settings:retry` | `r` | Settings |
| `settings:periodDay` | `d` | Settings |
| `settings:periodWeek` | `w` | Settings |
| `settings:sortByTokens` | `t` | Settings |
| `voice:pushToTalk` | `space` | Chat |
| `scroll:previousPrompt` | `ctrl+up`, `ctrl+up` | Transcript |
| `scroll:nextPrompt` | `ctrl+down`, `ctrl+down` | Transcript |
| `scroll:pageUp` | `pageup`, `pageup` | Scroll |
| `scroll:pageDown` | `pagedown`, `pagedown` | Scroll |
| `scroll:lineUp` | `ctrl+p`, `k`, `up`, `wheelup` | Transcript |
| `scroll:lineDown` | `ctrl+n`, `j`, `down`, `wheeldown` | Transcript |
| `scroll:top` | `g`, `home`, `ctrl+home`, `g`, `home` | Transcript |
| `scroll:bottom` | `shift+g`, `end`, `ctrl+end`, `shift+g`, `end` | Transcript |
| `scroll:halfPageUp` | `ctrl+u`, `ctrl+u` | Settings |
| `scroll:halfPageDown` | `ctrl+d`, `ctrl+d` | Settings |
| `scroll:fullPageUp` | `ctrl+b`, `b`, `shift+space`, `b` | Transcript |
| `scroll:fullPageDown` | `ctrl+f`, `space`, `space` | Transcript |
| `selection:copy` | `ctrl+shift+c`, `cmd+c` | Scroll |
| `selection:clear` | （无） | Unknown |
| `selection:extendLeft` | `shift+left` | Scroll |
| `selection:extendRight` | `shift+right` | Scroll |
| `selection:extendUp` | `shift+up` | Scroll |
| `selection:extendDown` | `shift+down` | Scroll |
| `selection:extendLineStart` | `shift+home` | Scroll |
| `selection:extendLineEnd` | `shift+end` | Scroll |
| `agents:switchView` | `ctrl+s` | Agents |
| `agents:togglePin` | `ctrl+t` | Agents |
