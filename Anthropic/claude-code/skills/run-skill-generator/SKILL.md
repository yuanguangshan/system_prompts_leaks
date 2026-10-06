---
name: run-skill-generator
description: Author or improve the run-<unit> skill - a per-project skill that tells agents how to build, launch, and drive this project's app. Use when the user asks to set up the project, get it running, write run instructions, or verify build/run steps work from a clean environment.
disable-model-invocation: true
---
<!-- BILINGUAL-EN-ZH -->

Your job is to produce a **skill** at `<unit>/.claude/skills/run-<unit-name>/`
that lets a future agent build, launch, and **drive** this project from
a clean machine.

你的任务是在 `<unit>/.claude/skills/run-<unit-name>/` 下产出一个 **skill**，让未来的代理能够在干净的机器上构建、启动并**操控（drive）**这个项目。

The skill has two parts that live together:

这个 skill 包含共处一处的两个部分：

```
<unit>/.claude/skills/run-<unit-name>/
  SKILL.md      <- agent-facing instructions - SHORT. Points at the driver.
  driver.mjs    <- (or driver.py, smoke.sh, ... - or none: web apps use
                   chromium-cli off-the-shelf, and the heredoc in
                   SKILL.md is the script)
```

That almost always means **writing code**, not just prose. If the app
has any interactive surface (GUI, TUI, long-running server, REPL), the
future agent needs a programmatic way to poke it. A markdown file by
itself cannot click a button - but sometimes the button-clicker
already exists: for web apps it's `chromium-cli`, for servers it's
`curl`. You build (or script) that harness now, commit it alongside
the skill, and the `SKILL.md` documents how to use it.

这几乎总是意味着要**编写代码**，而不只是写文字。如果应用有任何可交互的界面（GUI、TUI、长期运行的服务器、REPL），未来的代理就需要一种以编程方式与之交互的手段。光靠一个 markdown 文件点不了按钮——但有时候"点按钮的人"已经存在：对 Web 应用来说是 `chromium-cli`，对服务器来说是 `curl`。你现在就要构建（或编写脚本）这个驱动工具（harness），把它与 skill 一起提交，并在 `SKILL.md` 中写明如何使用它。

## Definition of done / 完成标准

You are done when **all** of these are true:

当以下条件**全部**满足时，你才算完成：

1. **You launched the app in this container and interacted with it** -
   not its test suite, the actual running app. For anything with a GUI,
   that means you have a screenshot file on disk that you took.
   **你在本容器中启动了这个应用并与之交互**——不是它的测试套件，而是实际运行中的应用。对任何带 GUI 的东西来说，这意味着磁盘上存有一张你亲手截取的屏幕截图文件。
2. **The interaction harness is committed** next to the skill. A driver
   script, a REPL wrapper, a smoke test, or the `chromium-cli` heredoc
   inline in `SKILL.md` - whatever you used to drive the app in step 1.
   (Graduated into `scripts/`/`e2e/`? - fine, point at it. Web app with
   `chromium-cli` off-the-shelf? - the inline script is the harness; no
   separate file.)
   **交互驱动工具（harness）已提交**在 skill 旁边。驱动脚本、REPL 包装器、冒烟测试，或内联在 `SKILL.md` 中的 `chromium-cli` heredoc——即你在第 1 步中用来驱动应用的任何东西。（已经"毕业"进 `scripts/`/`e2e/`？——可以，指向它即可。Web 应用直接用现成的 `chromium-cli`？——内联脚本就是 harness，无需单独的文件。）
3. **The `SKILL.md` documents the harness** as the primary agent path -
   the section a future agent reads first is "run this driver / pipe
   these commands to `chromium-cli`," not "run `npm start` and a window
   opens."
   **`SKILL.md` 把该 harness 记录为主代理路径**——未来代理最先读到的章节应是"运行这个驱动程序 / 把这些命令喂给 `chromium-cli`"，而不是"运行 `npm start` 然后窗口打开"。
4. **Every code block in `SKILL.md` is a command you ran that worked.**
   This session. This container. Not from the README, not inferred.
   **`SKILL.md` 中的每个代码块都是你实际运行成功过的命令。**就在本次会话、本容器中。不是抄自 README，也不是推断出来的。

If you're about to write the skill and you don't have (1), **stop.** You
are about to paraphrase existing docs. That document already exists -
it's called the README, and the whole reason you're here is that it
wasn't enough.

如果你正准备写这个 skill 却还不满足条件 (1)，**停下来。**你正要做的只是改写现有文档。那份文档早就存在——它叫 README，而你之所以会接到这个任务，正是因为它不够用。

【评论】本 skill 的核心设计是反"照抄文档"：要求一切结论来自本容器内的真实执行（截图、驱动脚本、可复现命令），用以防止代理生成看似完备实则未经验证的说明。

## The deliverables are code AND docs / 交付物是代码加文档

Typical output is a skill directory containing both:

典型的产出是一个同时包含两者的 skill 目录：

```
<unit>/.claude/skills/run-<unit>/
  SKILL.md         <- SHORT. Points at the driver. Has the frontmatter
                     that lets Claude auto-load it when someone asks
                     to "run <unit>" or "screenshot <unit>".
  driver.mjs       <- (or driver.py, smoke.sh, ... - or none: web apps
                     use chromium-cli off-the-shelf, and the heredoc
                     in SKILL.md is the script)
```

The driver lives **inside the skill directory** by default. They are a
pair - the skill's instructions and the code that implements them. A
driver that lives here is allowed to be a bit messier than production
code; it's agent tooling, not product surface.

驱动程序默认放在 **skill 目录内部**。两者是一对——skill 的说明文字，以及实现这些说明的代码。住在这里的驱动程序允许比生产代码略显杂乱；它是代理的工具，不是产品表面。

**Graduation:** if the driver grows into something the project's own
test suite wants to reuse - shared launch helpers, a real e2e harness -
move it to `scripts/` or `e2e/` and update `SKILL.md` to reference the
new path. The skill stays; the driver finds a better home.

**毕业（Graduation）：**如果驱动程序成长到项目自己的测试套件也想复用它——共享的启动辅助函数、真正的 e2e harness——就把它移到 `scripts/` 或 `e2e/`，并更新 `SKILL.md` 指向新路径。skill 留在原地；驱动程序搬去更合适的家。

The exact shape depends on the project, but the principle is constant:
**the driver is the deliverable.** The `SKILL.md` is its man page. For
a web app, the driver already exists - `chromium-cli`
([examples/playwright.md](examples/playwright.md)) - and the skill is
the script that runs it. For a desktop app
([examples/electron.md](examples/electron.md)), the driver is a custom
REPL under tmux that exposes `launch`/`ss`/`click`/`eval`. For a server,
the driver is `curl`. Whatever shape it takes, without something that
reaches into the running app, the skill is a description of a window
nobody can touch.

具体形态取决于项目，但原则恒定不变：**驱动程序才是交付物。**`SKILL.md` 只是它的 man 手册页。对 Web 应用来说，驱动程序已经现成——`chromium-cli`（[examples/playwright.md](examples/playwright.md)）——skill 就是运行它的脚本。对桌面应用（[examples/electron.md](examples/electron.md)）来说，驱动程序是 tmux 下暴露 `launch`/`ss`/`click`/`eval` 命令的自定义 REPL。对服务器来说，驱动程序就是 `curl`。无论何种形态，如果没有任何东西能伸进正在运行的应用里，这个 skill 就只是对一个谁也碰不到的窗口的描述。

## Where the skill goes / skill 放在哪里

The skill lives at `<unit>/.claude/skills/run-<unit-name>/`, where
`<unit>` is the directory for **one deployable thing** - an app, a
service, a library.

skill 位于 `<unit>/.claude/skills/run-<unit-name>/`，其中 `<unit>` 是**一个可部署物**——一个应用、一个服务或一个库——所在的目录。

Claude Code **natively discovers** skills from nested `.claude/skills/`
directories: an agent working anywhere inside `<unit>` will see
`/run-<unit-name>` as an available skill, and it auto-loads when the
request matches its description (e.g. "run the desktop app," "take a
screenshot of billing").

Claude Code 会**原生发现**嵌套 `.claude/skills/` 目录中的 skill：任何在 `<unit>` 内工作的代理都会看到 `/run-<unit-name>` 是一个可用的 skill，当请求匹配其描述时它会自动加载（例如"运行桌面应用""给 billing 截图"）。

- **Single-project repo:** `.claude/skills/run-<repo-name>/` at repo root.
  **单项目仓库：**仓库根目录下的 `.claude/skills/run-<repo-name>/`。
- **Large repo with many apps:** one per app, colocated -
  `apps/billing/.claude/skills/run-billing/`,
  `apps/desktop/.claude/skills/run-desktop/`.
  **含多个应用的大型仓库：**每个应用一个，与应用同处一地——`apps/billing/.claude/skills/run-billing/`、`apps/desktop/.claude/skills/run-desktop/`。
- **App with multiple binaries:** still **one** skill at the app's
  root with a section per binary. They share setup. Start from the
  closest single-binary example and add a `## Run: <name>` section
  per binary.
  **含多个可执行文件的应用：**仍在应用根目录放**一个** skill，为每个可执行文件设一个章节。它们共享安装配置。从最接近的单可执行文件示例出发，为每个可执行文件添加一个 `## Run: <name>` 章节。

If you're not sure where the unit boundary is, **ask the user.**

如果你不确定 unit 边界在哪里，**问用户。**

Slugify the directory name: lowercase, dashes for spaces, no slashes
(`run-billing-api`, not `run-billing/api`). The directory name and
the frontmatter `name:` should match - that's the slash command.

目录名做 slug 化处理：小写、空格换成连字符、不含斜杠（`run-billing-api`，而不是 `run-billing/api`）。目录名应与 frontmatter 中的 `name:` 一致——那就是斜杠命令名。

## Process / 流程

### 0. Find any existing skill about running this app / 0. 寻找已有的关于运行本应用的 skill

List the project's skills with their descriptions (same probe `/run`
uses - users name these variously, so match on description, not name):

列出项目的各个 skill 及其描述（与 `/run` 使用的探查方式相同——用户对它们的命名五花八门，所以按描述匹配，而不是按名字）：

```bash
d=$PWD; while :; do
  grep -Hm1 '^description:' "$d"/.claude/skills/*/SKILL.md 2>/dev/null
  [ -e "$d/.git" ] || [ "$d" = / ] && break
  d=$(dirname "$d")
done
```

If one is about launching/driving this app - whatever it's named -
**refine, don't rewrite**: verify its claims, fix what's wrong, add
what's missing, preserve what works. Re-run the driver if there is
one. Keep its existing name.

如果其中某个 skill 与启动/驱动本应用有关——无论它叫什么名字——**改进它，不要重写**：验证它的说法、修正错误、补上缺失、保留有效的部分。如果有驱动程序就重新运行一遍。保留它现有的名字。

(Also check for a legacy `.claude/run.md` - earlier versions of this
tool produced those. If you find one, migrate it: the body becomes
the skill's `SKILL.md` content, any referenced scripts move into the
skill dir, and delete the old file.)

（另外检查是否有旧式的 `.claude/run.md`——本工具的早期版本生成的是这种文件。如果找到，就迁移它：正文变成 skill 的 `SKILL.md` 内容，被引用的脚本移入 skill 目录，然后删除旧文件。）

If none exists, decide where to create it (see above) and continue.

如果一个都没有，就决定在哪里创建（见上文）并继续。

### 1. Discover - and treat every claim as disprovable / 1. 探索——把每个论断都当作可证伪的

Figure out what you're authoring for:

弄清楚你是在为哪种情况编写：

- Manifest right here (`package.json`, `go.mod`, `pyproject.toml`...) and
  it's one self-contained thing -> this is the unit.
  清单文件就在眼前（`package.json`、`go.mod`、`pyproject.toml`……）且它是一个自包含的整体 -> 这就是 unit。
- Looks like a mega-repo root (`apps/`, `packages/`, `services/`) ->
  **ask which one.** List candidates, let them pick, `cd` there.
  看起来像超大仓库的根目录（`apps/`、`packages/`、`services/`）-> **问清是哪一个。**列出候选，让对方选，然后 `cd` 过去。
- Genuinely ambiguous -> ask.
  确实含糊 -> 问。

Survey the usual places: `README.md`, `package.json` scripts,
`Dockerfile`, `Makefile`, `.github/workflows/`, `CONTRIBUTING.md`. CI
configs are often more accurate than READMEs.

调查那些常见位置：`README.md`、`package.json` 脚本、`Dockerfile`、`Makefile`、`.github/workflows/`、`CONTRIBUTING.md`。CI 配置往往比 README 更准确。

**Every claim in existing docs is a hypothesis.** Especially the
negative ones:

**现有文档中的每个论断都是一个假设。**尤其是那些否定性的论断：

| When docs say... | What you do |
|---|---|
| "Requires macOS/Windows" | Launch it on Linux anyway. Apps rarely refuse to start - they crash on a missing `.so`, which `apt-get` fixes. Native modules for *your host's* keychain/notifications may no-op; the core usually runs. |
| "Requires a GPU" | Try software rendering. Electron/Chrome fall back with `--disable-gpu`. |
| "Requires a paid account / feature flag" | The gate is code you can read. Find it (env var? build define? SSR-embedded JSON?) and patch it for your local run. Document the patch. |
| "Run `npm start`" | That's the human path (spawns a window, waits forever). Find or build the *programmatic* path - `electron-forge start` to build then launch via Playwright, or equivalent. |

| 文档说…… | 你要做的 |
|---|---|
| "需要 macOS/Windows" | 照样在 Linux 上启动它。应用很少会拒绝启动——它们只是因缺少某个 `.so` 而崩溃，`apt-get` 就能解决。针对*你所在宿主机*的钥匙串/通知的原生模块可能变成空操作；核心通常能跑起来。 |
| "需要 GPU" | 试试软件渲染。Electron/Chrome 会用 `--disable-gpu` 自动回退。 |
| "需要付费账户 / 功能开关" | 那道门槛就是你能读到的代码。找到它（环境变量？构建期 define？SSR 内嵌的 JSON？），为本地运行打上补丁，并把补丁记录下来。 |
| "运行 `npm start`" | 那是人类路径（启动一个窗口，然后永远等待）。找到或构建*程序化*路径——用 `electron-forge start` 构建再经 Playwright 启动，或类似方案。 |

【评论】该表要求把文档中的否定性论断（"需要 GPU""仅限 macOS"）当作待证伪的假设并通过实验推翻，其中甚至包括为绕过付费门槛而打补丁的做法，仅限于本地测试环境使用。

"Not supported on Linux" in a README written by a macOS developer
means "I never tried." You're about to try. **If you give up here, the
skill you write is the README with extra steps.**

由 macOS 开发者写下的 README 里那句"不支持 Linux"，意思是"我从没试过"。而你就要去试了。**如果你在这里放弃，你写出的 skill 就只是加了若干步骤的 README。**

### 2. Execute - and BUILD the harness you need / 2. 执行——并构建你需要的 harness

You're in a headless Linux container. The app is going to fight you.
That fight is the content of the skill.

你在一个无头（headless）Linux 容器里。应用会跟你较劲。这场较劲正是 skill 的内容所在。

Keep a running `NOTES.md` as you go. Every error -> every fix -> every
command that finally worked. This scratchpad becomes the
Troubleshooting section.

边做边维护一份 `NOTES.md`。每个错误 -> 每次修复 -> 每条最终奏效的命令。这份草稿日后会成为故障排除（Troubleshooting）章节。

**Work up to a real interaction:**

**逐步做到一次真实的交互：**

- **Install + build.** When something's missing, note the exact
  `apt-get` / `npm install` that fixed it.
  **安装 + 构建。**缺什么的时候，把恰好解决问题的那条 `apt-get` / `npm install` 命令记下来。
- **Launch the app.** Not the test suite - the app. A desktop GUI
  (Electron, native) needs `xvfb-run` and a handful of `lib*`
  packages; a web app driven by `chromium-cli` runs headless and
  needs neither. Launch timeouts and cryptic crashes are normal at
  this stage. Read the stack trace, install the missing thing, try
  again.
  **启动应用。**不是测试套件——是应用。桌面 GUI（Electron、原生）需要 `xvfb-run` 和几个 `lib*` 包；由 `chromium-cli` 驱动的 Web 应用以无头方式运行，两者都不需要。启动超时和莫名的崩溃在这个阶段很正常。读堆栈跟踪，装上缺的东西，再试一次。
- **Build a harness to drive it.** You need a handle on the running
  app that lets you send input and observe output programmatically.
  The shape depends on the project (see table below).
  **构建驱动它的 harness。**你需要一个作用于运行中应用的"把手"，让你能以编程方式发送输入、观察输出。形态取决于项目（见下表）。

  **Cover the layer(s) PRs actually touch.** A tmux driver that pokes
  the CLI's user surface is the right handle for UI changes - and the
  wrong one for a PR that touches one internal function. For the
  latter an agent wants `NODE_ENV=test bun run script.ts` (or
  equivalent): import the function, call it, observe. If most PRs
  here touch internals, that direct-invocation path is the driver's
  main entry point, and the tmux launch is secondary. Look at recent
  merged PRs: what layer do they touch? Cover that.
  **覆盖 PR 实际触及的层。**对 UI 改动而言，戳 CLI 用户界面的 tmux 驱动是对症的把手——对触及单个内部函数的 PR 则不然。对后者，代理想要的是 `NODE_ENV=test bun run script.ts`（或等价方式）：导入函数、调用、观察。如果这里的大多数 PR 触及内部实现，那条直接调用路径就是驱动程序的主入口，tmux 启动退居其次。看看最近合并的 PR：它们触及哪一层？就覆盖那一层。

  For a **web** app, `chromium-cli` is the driver - you script it,
  you don't write it (see [examples/playwright.md](examples/playwright.md)).
  For a **desktop** GUI (Electron), write a REPL driver (stdin
  commands -> click/type/screenshot), run it inside tmux, and use
  `send-keys` / `capture-pane`. You will iterate on that driver - it
  starts minimal (`launch`, `ss`, `quit`) and grows whatever commands
  you need to reach the interesting part of the app.
  对 **web** 应用，`chromium-cli` 就是驱动程序——你编写的是驱动它的脚本，而不是它本身（见 [examples/playwright.md](examples/playwright.md)）。对 **desktop** GUI（Electron），写一个 REPL 驱动（stdin 命令 -> 点击/输入/截图），在 tmux 内运行它，并使用 `send-keys` / `capture-pane`。你会不断迭代这个驱动——它从最小集（`launch`、`ss`、`quit`）起步，逐渐长出触达应用中有意思部分所需的任何命令。
- **Do one real user flow end-to-end.** Click the button. Fill the
  form. See the result in the DOM. Take a screenshot. **Actually look
  at the screenshot.** If it's blank or showing an error page, you're
  not done.
  **端到端地走完一个真实用户流程。**点那个按钮。填那个表单。在 DOM 里看到结果。截一张图。**真的去看那张截图。**如果它是空白的或显示错误页，你就还没完成。
- **Then run the tests.** Unit tests are a sanity check, not the main
  event.
  **然后跑测试。**单元测试是健全性检查，不是重头戏。
- **Stop cleanly.**
  **干净地收尾。**

**Obstacles are content.** You will hit weird ones - coordinate systems
that don't line up, APIs that return empty on this Electron version,
feature gates that hide the thing you need to test. Each of these gets
a bullet in Gotchas and (often) a helper in your driver. The gold
standard is a Gotchas section full of things nobody could have guessed.

**障碍本身就是内容。**你会撞上各种怪事——对不齐的坐标系、在这个 Electron 版本上返回空的 API、把你需要测试的东西藏起来的功能开关。每一条都值得在 Gotchas（坑点）章节里加一个条目，并且（通常）在你的驱动程序里加一个辅助函数。黄金标准是：Gotchas 章节里全是没人能预先猜到的东西。

**The driver script gets committed alongside the skill.** It is not
scaffolding. It is the way future agents (and humans) will drive this
app. It defaults to living inside the skill directory (for a web app
using `chromium-cli`, that means inline in `SKILL.md` - the heredoc
is the script). If it outgrows that - if the project's real test
suite wants to import from it - move it to `scripts/` or `e2e/` and
update `SKILL.md` to point there.

**驱动脚本要与 skill 一起提交。**它不是脚手架。它就是未来的代理（以及人类）驱动这个应用的方式。它默认放在 skill 目录内（对使用 `chromium-cli` 的 Web 应用来说，这意味着内联在 `SKILL.md` 里——heredoc 就是脚本）。如果它长大超出了这个范围——如果项目真正的测试套件想从它那里导入——就把它移到 `scripts/` 或 `e2e/`，并更新 `SKILL.md` 指向那里。

### 3. Write SKILL.md / 3. 撰写 SKILL.md

Short. Point at the driver. Use [template.md](template.md) as the
starting structure - it has the frontmatter shape.

简短。指向驱动程序。用 [template.md](template.md) 作为起始结构——它带有 frontmatter 的样式。

**The frontmatter matters.** The `name:` becomes the slash command
(`/run-billing`). The `description:` is what Claude scans to decide
whether to auto-load this skill - put the **verbs an agent would
actually type** in it: "run," "start," "build," "test," "screenshot."
Generic descriptions ("helpful utilities for billing") won't match.

**frontmatter 很重要。**`name:` 会成为斜杠命令（`/run-billing`）。`description:` 是 Claude 用来判断是否自动加载这个 skill 的依据——把**代理真正会输入的动词**写进去："run""start""build""test""screenshot"。泛泛的描述（"billing 的实用工具"）匹配不上。

Body structure:

正文结构：

1. One-paragraph intro: what this app is, how it's driven -
   `<driver-path>` under xvfb/tmux for desktop, `chromium-cli` for
   web, `curl` for a server.
   一段话的简介：这个应用是什么、如何驱动它——desktop 用 xvfb/tmux 下的 `<driver-path>`，web 用 `chromium-cli`，服务器用 `curl`。
2. **Prerequisites** - the exact `apt-get install` line you ran.
   **先决条件**——你实际运行过的那条 `apt-get install` 命令。
3. **Build** - the exact commands, in order. Include any patches you
   had to apply (feature gates, config overrides) with the exact `sed`
   or edit.
   **构建**——按顺序列出的确切命令。包含你不得不打的任何补丁（功能开关、配置覆盖），附上确切的 `sed` 或编辑内容。
4. **Run (agent path)** - FIRST. How to launch the driver, what
   commands it accepts, where screenshots land. If it's a REPL, show
   the tmux wrapping. This is the section the next agent will actually
   use.
   **运行（代理路径）**——放在第一。如何启动驱动程序、它接受哪些命令、截图落在哪。如果是 REPL，展示 tmux 包装。这是下一个代理真正会使用的章节。
5. **Run (human path)** - SECOND, if different. `npm start` -> window
   opens -> Ctrl-C. Brief. Note that it's useless headless.
   **运行（人类路径）**——放在第二，如果与代理路径不同的话。`npm start` -> 窗口打开 -> Ctrl-C。简短。注明它在无头环境下没用。
6. **Gotchas** - the battle scars. The things that look like they
   should work but don't, and the workaround. If this section is
   generic, you didn't fight hard enough.
   **Gotchas（坑点）**——战斗留下的伤疤。那些看起来应该行得通却行不通的事，以及绕过办法。如果这个章节写得平淡无奇，说明你搏斗得不够狠。
7. **Troubleshooting** - symptom -> fix. Only errors you actually hit.
   **故障排除**——症状 -> 解法。只收录你真正遇到过的错误。

Keep it **verified** (you ran it), **prescriptive** (one path, not
options), **honest** (flaky? slow? say so).

保持**经过验证**（你运行过）、**规定动作**（一条路径，不摆选项）、**诚实**（不稳定？慢？直说）。

**Paths in SKILL.md are relative to `<unit>/`,** not to the skill
directory. State this at the top if there's any ambiguity. When the
driver lives inside the skill, its path from `<unit>` is
`.claude/skills/run-<unit-name>/driver.mjs` - it's long, but explicit.

**SKILL.md 中的路径相对于 `<unit>/`，**而不是相对于 skill 目录。如有任何歧义，在文件顶部说明这一点。当驱动程序住在 skill 内部时，它相对于 `<unit>` 的路径是 `.claude/skills/run-<unit-name>/driver.mjs`——很长，但明确。

### 4. Verify / 4. 验证

Fresh shell, `cd` into the unit, follow the skill's `SKILL.md`
line-by-line without deviating. Any improvisation = a gap. Fix it.

开一个全新的 shell，`cd` 进入该 unit，逐行照着 skill 的 `SKILL.md` 执行，毫不偏离。任何即兴发挥都等于一个缺口。修掉它。

## Project-type patterns / 项目类型模式

Pick a starting shape for your driver. These examples are shared with
the `/run` skill (same per-project-type patterns are used as the
fallback when no project-specific run skill exists) - if you're
authoring a new one, the example is your starting template.

为你的驱动程序挑一个起始形态。这些示例与 `/run` skill 共享（当不存在项目专属的 run skill 时，同样的按项目类型划分的模式被用作回退）——如果你正在编写新的模式，示例就是你的起始模板。

| Project type | Driver shape | Example |
|---|---|---|
| Web server / API | Background-launch + `curl`-based smoke script | [examples/server.md](examples/server.md) |
| CLI tool | Representative-args smoke script, check exit codes + output | [examples/cli.md](examples/cli.md) |
| TUI / interactive terminal | tmux wrapper: `send-keys` / `capture-pane` | [examples/tui.md](examples/tui.md) |
| Electron / desktop GUI | Playwright `_electron` REPL driver under xvfb, screenshots, tmux-wrapped | [examples/electron.md](examples/electron.md) |
| Browser-driven | dev server + `chromium-cli` script | [examples/playwright.md](examples/playwright.md) |
| Library / SDK | Import-and-call smoke script | [examples/library.md](examples/library.md) |

| 项目类型 | 驱动形态 | 示例 |
|---|---|---|
| Web 服务器 / API | 后台启动 + 基于 `curl` 的冒烟脚本 | [examples/server.md](examples/server.md) |
| CLI 工具 | 具代表性参数的冒烟脚本，检查退出码 + 输出 | [examples/cli.md](examples/cli.md) |
| TUI / 交互式终端 | tmux 包装：`send-keys` / `capture-pane` | [examples/tui.md](examples/tui.md) |
| Electron / 桌面 GUI | xvfb 下的 Playwright `_electron` REPL 驱动，带截图、tmux 包装 | [examples/electron.md](examples/electron.md) |
| 浏览器驱动 | 开发服务器 + `chromium-cli` 脚本 | [examples/playwright.md](examples/playwright.md) |
| 库 / SDK | 导入并调用的冒烟脚本 | [examples/library.md](examples/library.md) |

For a web app, start from [examples/playwright.md](examples/playwright.md)
-- drive it with `chromium-cli`, no custom driver needed. For a
desktop app, start from [examples/electron.md](examples/electron.md)
-- it has the full `_electron` REPL driver skeleton, the tmux wrapping,
and the catalog of obstacles you'll hit.

对 Web 应用，从 [examples/playwright.md](examples/playwright.md) 出发——用 `chromium-cli` 驱动它，无需自定义驱动程序。对桌面应用，从 [examples/electron.md](examples/electron.md) 出发——它包含完整的 `_electron` REPL 驱动骨架、tmux 包装，以及你会撞上的障碍清单。

## What to include / 应当包含什么

- **Prerequisites** - OS packages, runtimes, tools. Ubuntu `apt-get`
  lines. The exact ones.
  **先决条件**——操作系统包、运行时、工具。Ubuntu `apt-get` 命令行。要一条不差。
- **Setup** - install deps, configure, any patches.
  **安装配置**——装依赖、做配置、打补丁。
- **Build** - compile/bundle.
  **构建**——编译/打包。
- **Run (agent path)** - the driver. Commands. Screenshot location.
  **运行（代理路径）**——驱动程序。命令。截图位置。
- **Direct invocation** - if callable: how to import and run internal
  code without the full app. The env var / flag that bypasses init
  guards. Many PRs need only this.
  **直接调用**——如果可调用：如何在不起完整应用的情况下导入并运行内部代码。绕过初始化守卫的环境变量 / 开关。许多 PR 只需要这个。
- **Run (human path)** - if meaningfully different.
  **运行（人类路径）**——如果有实质差异的话。
- **Test** - the test suite command.
  **测试**——测试套件命令。
- **Gotchas** - non-obvious traps you hit.
  **Gotchas（坑点）**——你撞上的那些不明显的陷阱。
- **Troubleshooting** - error -> fix.
  **故障排除**——错误 -> 解法。
- **The driver itself** - committed in the skill dir (or graduated
  to `scripts/`/`e2e/`), or inline in `SKILL.md` for `chromium-cli`
  web apps; referenced from `SKILL.md` either way.
  **驱动程序本身**——提交在 skill 目录里（或"毕业"到 `scripts/`/`e2e/`），对 `chromium-cli` Web 应用则内联在 `SKILL.md` 中；无论哪种方式都要在 `SKILL.md` 中引用。

## What to leave out / 应当省略什么

- **Anything you didn't run.** If the README says `yarn start:prod` and
  you never ran it, it's not in the skill. Full stop.
  **任何你没运行过的东西。**如果 README 写着 `yarn start:prod` 而你从没运行过它，它就不能进 skill。没有例外。
- **Documented happy paths for platforms you're not on.** You're in a
  Linux container. A macOS-only section you can't verify is
  speculation. Mention it exists; don't elaborate.
  **你所在平台之外有文档记录的理想路径。**你在 Linux 容器里。无法验证的 macOS 专属章节只是猜测。提一句它存在即可；不要展开。
- **Exhaustive options.** One working path.
  **穷举所有选项。**一条走得通的路径就够。
- **Architecture prose.** That's other docs.
  **架构论述。**那是别的文档的事。
- **Generic troubleshooting.** "If the build fails, check your Node
  version" - useless. Only include errors you actually hit and fixed.
  **泛泛的故障排除。**"如果构建失败，检查你的 Node 版本"——没用。只收录你真正遇到并修好的错误。

## Red flags - you are about to ship the wrong thing / 危险信号——你即将交付错误的东西

Stop and reconsider if:

出现以下情况时，停下来重新考虑：

- **You haven't taken a screenshot** of a GUI app. You didn't run it.
  **你没有截过图**——对一个 GUI 应用而言。那就是你没运行过它。
- **Your skill has no driver/smoke script** to point at, and the app
  is interactive. The next agent has no way to drive it. (Web app
  using `chromium-cli`? - the heredoc in `SKILL.md` is the driver;
  no separate file needed.)
  **你的 skill 没有可指向的驱动/冒烟脚本**，而应用是可交互的。下一个代理将无法驱动它。（用 `chromium-cli` 的 Web 应用？——`SKILL.md` 里的 heredoc 就是驱动程序；无需单独文件。）
- **Your skill reads like the README.** Same structure, same
  commands, same caveats. You paraphrased.
  **你的 skill 读起来像 README。**相同的结构、相同的命令、相同的注意事项。你只是在改写。
- **Your Troubleshooting section is generic.** Real execution produces
  specific, weird errors. Generic errors = you didn't execute.
  **你的故障排除章节是泛泛而谈的。**真实的执行会产生具体而古怪的错误。错误泛泛 = 你没有执行过。
- **You wrote "not supported on this platform"** without trying to
  launch it. The README author was on a Mac. You are not. Try.
  **你写下了"此平台不支持"**却没有尝试启动它。README 的作者用的是 Mac。你不是。试试看。
- **Everything worked first try.** Either this project is trivially
  simple, or you ran the test suite and called it done.
  **一切一次就成功。**要么这个项目简单到不值一提，要么你只跑了测试套件就宣布完工。
