<!-- BILINGUAL-EN-ZH -->
# Example: TUI / interactive terminal app / 示例：TUI / 交互式终端应用

Interactive terminal apps (text editors, REPLs, curses-based UIs) can't
be driven directly by an agent's bash tool - they take over the terminal.
The skill must show how to wrap them in `tmux` so the agent can send
input, capture output, and take screenshots.

交互式终端应用（文本编辑器、REPL、基于 curses 的界面）无法被 agent 的 bash 工具直接驱动——它们会接管整个终端。技能必须展示如何用 `tmux` 包装它们，使 agent 能够发送输入、捕获输出并截取屏幕内容。

## The tmux pattern / tmux 模式

This is the standard approach:

这是标准做法：

1. Start the TUI inside a detached tmux session
  1. 在一个分离（detached）的 tmux 会话中启动 TUI
2. Send keystrokes with `tmux send-keys`
  2. 用 `tmux send-keys` 发送按键
3. Read screen contents with `tmux capture-pane`
  3. 用 `tmux capture-pane` 读取屏幕内容
4. Clean up with `tmux kill-session`
  4. 用 `tmux kill-session` 清理

The skill's `SKILL.md` should present this as the primary way to drive
the app. A small `driver.sh` that wraps the launch+attach sequence can
live in the skill directory, but for most TUIs the raw tmux commands in
the skill body are enough.

技能的 `SKILL.md` 应当把这种方式作为驱动应用的主要方法。技能目录里可以放一个包装启动加连接（launch+attach）流程的小型 `driver.sh`，但对大多数 TUI 来说，技能正文中直接写 tmux 命令就足够了。

## Example snippet / 示例代码片段

> ## Run (interactive, for agents)
>
> ## 运行（交互式，供 agent 使用）
>
> Start the TUI inside tmux:
>
> 在 tmux 中启动 TUI：
>
> ```bash
> tmux new-session -d -s app -x 120 -y 40 './myapp'
> ```
>
> Poll until the ready marker appears (faster + more reliable than a fixed sleep -
> returns the instant the app is up, fails loudly if it isn't):
>
> 轮询直到就绪标记出现（比固定 sleep 更快、更可靠——应用一就绪立即返回，未就绪则明确报错）：
>
> ```bash
> timeout 10 bash -c 'until tmux capture-pane -t app -p | grep -q "Ready"; do sleep 0.2; done'
> tmux capture-pane -t app -p
> ```
>
> Send input (this example navigates to the Settings screen and toggles
> an option):
>
> 发送输入（本例导航到 Settings 界面并切换一个选项）：
>
> ```bash
> tmux send-keys -t app 's'
> timeout 5 bash -c 'until tmux capture-pane -t app -p | grep -q "Settings"; do sleep 0.2; done'
> tmux send-keys -t app 'Down' 'Down' 'Space'  # navigate + toggle
> timeout 5 bash -c 'until tmux capture-pane -t app -p | grep -qF "[x]"; do sleep 0.2; done'
> tmux capture-pane -t app -p
> ```
>
> If you find yourself writing more than a couple of these poll lines, pull
> them into a `wait_for()` helper in a `driver.sh` next to the skill.
>
> 如果你发现自己写了不止两三行这样的轮询语句，就把它们抽取为技能旁 `driver.sh` 中的 `wait_for()` 辅助函数。
>
> Quit:
>
> 退出：
>
> ```bash
> tmux send-keys -t app 'q'
> tmux kill-session -t app 2>/dev/null || true
> ```
>
> ### Key reference / 按键参考
>
> | Key | Action |
> |---|---|
> | `j` / `k` or `Down` / `Up` | Navigate list |
> | `Enter` | Select |
> | `s` | Settings |
> | `q` | Quit |
>
> | 按键 | 操作 |
> |---|---|
> | `j` / `k` 或 `Down` / `Up` | 在列表中导航 |
> | `Enter` | 选择 |
> | `s` | 设置 |
> | `q` | 退出 |

## Details worth documenting / 值得记录的细节

- **Terminal size.** Some TUIs break or hide content at small widths.
  Specify a known-good size in the `tmux new-session -x -y` args.
  - **终端尺寸。** 某些 TUI 在窄宽度下会错乱或隐藏内容。在 `tmux new-session -x -y` 参数中指定一个已知可用的尺寸。
- **Startup time.** Poll for a ready marker (`until tmux capture-pane | grep -q X`)
  rather than a fixed `sleep N` - returns the instant the app is up, and fails
  usefully when it never does. Say what string means ready.
  - **启动时间。** 用轮询就绪标记（`until tmux capture-pane | grep -q X`）代替固定 `sleep N`——应用一起来的瞬间即返回，始终不就绪时也能给出有效的失败信息。要说明哪个字符串代表"就绪"。
- **Keybinding reference.** A table of the main keys. This is the "API"
  of a TUI - an agent needs it to drive the app.
  - **快捷键参考。** 一张主要按键的表格。这是 TUI 的"API"——agent 需要它才能驱动应用。
- **Exit cleanly.** Show the quit keystroke *and* `tmux kill-session` as
  a fallback.
  - **干净退出。** 同时展示退出按键*和*作为兜底的 `tmux kill-session`。
- **Color/unicode quirks.** If `capture-pane` output is hard to read,
  note flags that help (`-e` for escape sequences, `-J` to join wrapped
  lines).
  - **颜色/Unicode 怪癖。** 如果 `capture-pane` 的输出难以阅读，记录有帮助的标志（`-e` 用于转义序列，`-J` 用于合并折行）。

## Also document the direct invocation / 同时记录直接调用方式

For a human running the app interactively, tmux is overkill. Include
the one-liner too:

对以交互方式运行应用的人类来说，tmux 属于杀鸡用牛刀。也要附上那一行单行命令：

> ## Run (direct, for humans)
>
> ## 运行（直接运行，供人类使用）
>
> ```bash
> ./myapp
> ```
>
> Press `q` to quit.
>
> 按 `q` 退出。
