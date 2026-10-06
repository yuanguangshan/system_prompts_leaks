<!-- BILINGUAL-EN-ZH -->
---
name: verify
description: Verify that a code change actually does what it's supposed to by exercising it end-to-end and observing behavior — drive the affected flow, not just tests or typecheck. Run before committing nontrivial changes; bootstraps this repo's project verify skill if none exists yet. Don't invoke it on a diff that only touches tests, docs, or other code with no runtime surface to drive (a change to product source always has one) — there's nothing to observe.
disable-model-invocation: true
---

**Verification is runtime observation.** You build the app, run it,
drive it to where the changed code executes, and capture what you
see. That capture is your evidence. Nothing else is.

**验证就是运行时观察。** 你要构建应用、运行它、把应用驱动到被改动代码执行之处，并记录下你所看到的内容。这份记录就是你的证据。除此之外的都不是。

**Don't run tests. Don't typecheck.** Running them here proves you
can run CI — not that the change works. Not as a warm-up,
not "just to be sure," not as a regression sweep after. The time
goes to running the app instead.

**不要跑测试。不要做类型检查。** 在这里运行它们只能证明你会跑 CI——不能证明改动可用。不是作为热身，不是"只是求个安心"，也不是事后补一轮回归。这些时间应该用在运行应用上。

【评论】该技能把"验证"严格定义为对运行时行为的观察，刻意排除测试与类型检查，以避免把"CI 能通过"误当作"改动有效"的证据——这是对验证语义的一种强观点。

**Don't import-and-call.** `import { foo } from './src/...'` then
`console.log(foo(x))` is a unit test you wrote. The function did what
the function does — you knew that from reading it. The app never ran.
Whatever calls `foo` in the real codebase ends at a CLI, a socket, or
a window. Go there.

**不要"导入即调用"。** `import { foo } from './src/...'` 然后 `console.log(foo(x))` 是你自己写的单元测试。函数做了函数一直在做的事——你读代码早就知道了。应用从未真正运行。真实代码库中调用 `foo` 的那条链路，最终会落在某个 CLI、某个 socket 或某个窗口上。去那里验证。

## Find the change / 找到改动

The scope is what you're verifying — usually a diff, sometimes just
"does X work." In a git repo, establish the full range (a branch may
be many commits, or the change may still be uncommitted):

范围就是你正在验证的东西——通常是一个 diff，有时只是"X 是否能用"。在 git 仓库中，先确定完整范围（一个分支可能包含多次提交，或改动可能尚未提交）：

```bash
git log --oneline @{u}..              # count commits (if upstream set)
git diff @{u}.. --stat                # full range, not HEAD~1
git diff origin/HEAD... --stat        # no upstream: committed vs base
git diff HEAD --stat                  # uncommitted: working tree vs HEAD
gh pr diff                            # if in a PR context
```

State the commit count. Large diff truncating? Redirect to a file
then Read it. Repo but no diff from any of these → say so, stop.
**No repo → the scope is whatever the user named; ask if they
didn't.**

报告提交数量。大 diff 被截断？重定向到文件再用 Read 读取。有仓库但以上命令都没有产出 diff → 说明情况，停止。**没有仓库 → 范围就是用户指名的东西；如果用户没说，就去问。**

**The diff is ground truth. Any description is a claim about it.**
Read both. If they disagree, that's a finding.

**diff 是事实基准。任何描述都只是对它的断言。** 两者都要读。如果两者不一致，这本身就是一项发现。

## Surface / 作用面

The surface is where a user — human or programmatic — meets the
change. That's where you observe.

作用面（surface）是用户——人类或程序——与改动相遇之处。那是你进行观察的地方。

| Change reaches | Surface | You |
|---|---|---|
| CLI / TUI | terminal | type the command, capture the pane — [example](examples/cli.md) |
| Server / API | socket | send the request, capture the response — [example](examples/server.md) |
| GUI | pixels | drive it under xvfb/Playwright, screenshot |
| Library | package boundary | sample code through the public export — `import pkg`, not `import ./src/...` |
| Prompt / agent config | the agent | run the agent, capture its behavior |
| CI workflow | Actions | dispatch it, read the run |

| 改动到达之处 | 作用面 | 你要做的事 |
|---|---|---|
| CLI / TUI | 终端 | 输入命令，捕获面板输出 — [示例](examples/cli.md) |
| 服务器 / API | socket | 发送请求，捕获响应 — [示例](examples/server.md) |
| GUI | 像素 | 在 xvfb/Playwright 下驱动它，截图 |
| 库 | 包边界 | 通过公共导出运行示例代码 — `import pkg`，而不是 `import ./src/...` |
| 提示词 / 代理配置 | 代理本身 | 运行代理，捕获其行为 |
| CI 工作流 | Actions | 触发它，读取运行结果 |

**Internal function? Not a surface.** Something in the repo calls it
and that caller ends at one of the rows above. Follow it there. A
bash security gate's surface isn't the function's return value — it's
the CLI prompting or auto-allowing when you type the command.

**内部函数？那不是作用面。** 仓库中有代码调用它，而那个调用方最终落在上表某一行。顺着链路走到那里。一个 bash 安全门控的作用面不是该函数的返回值——而是你输入命令时 CLI 进行提示或自动放行的那个行为。

**No runtime surface at all** — docs-only, type declarations with no
emit, build config that produces no behavioral diff — report
**SKIP — no runtime surface: (reason).** Don't run tests to fill
the space.

**完全没有运行时作用面**——纯文档、不产出发产物的类型声明、不产生行为差异的构建配置——报告 **SKIP — no runtime surface: (reason)。** 不要靠跑测试来填充空白。

**Tests in the diff are the author's evidence, not a surface.** CI
runs them. You'd be re-running CI. Tests-only PR → SKIP, one line.
Mixed src+tests → verify the src, ignore the test files. Reading a
test to learn what to check is fine — it's a spec. But then go run
the app. Checking that assertions match source is code review.

**diff 中的测试是作者的证据，不是作用面。** 测试由 CI 运行；你再跑就等于重复运行 CI。仅含测试的 PR → SKIP，一行说明即可。源码与测试混合 → 验证源码部分，忽略测试文件。读测试来了解要检查什么是可以的——它是规格说明。但读完仍要去运行应用。检查断言与源码是否一致属于代码审查。

## Get a handle / 找到抓手

**Check `.claude/skills/` first — even if you already know how to
build and run.** A matching `verifier-*` skill is the repo's
evidence-capture protocol: it wraps the session so a reviewer can
replay what you saw (recording, screenshots). Drive the surface
without it and you get a verdict with no replay.

**先检查 `.claude/skills/`——即使你已经知道如何构建和运行。** 匹配的 `verifier-*` 技能是该仓库的证据采集协议：它会把会话包装起来，让审查者能够回放你看到的内容（录像、截图）。不带它驱动作用面，你得到的将是一个没有回放的结论。

Skills live at the repo root **and** in the package/app dirs the
diff touches — in a monorepo the unlock for `apps/desktop/` is
usually `apps/desktop/.claude/skills/`, not the root. Probe both:

技能既位于仓库根目录，**也**位于 diff 涉及的包/应用目录——在 monorepo 中，`apps/desktop/` 的解锁钥匙通常是 `apps/desktop/.claude/skills/`，而不是根目录。两处都探测：

```bash
ls .claude/skills/                    # repo root
ls <touched-dir>/.claude/skills/      # each dir level the diff names
```

- **`verifier-*` matching your surface** (CLI verifier for a CLI
  change, etc.) → invoke it with the Skill tool and follow its
  setup. Mismatched surface → skip that one, try the next. Stale
  verifier (fails on mechanics unrelated to the change) → ask the
  user whether to patch it; don't FAIL the change for verifier rot.
  **与作用面匹配的 `verifier-*`**（CLI 改动对应 CLI verifier 等）→ 用 Skill 工具调用它并按其设置执行。作用面不匹配 → 跳过它，试下一个。过时的 verifier（在与改动无关的机制上失败）→ 询问用户是否修补；不要因为 verifier 年久失修而给改动判 FAIL。
- **`run-*` but no matching verifier** → use its build/launch
  primitives as your handle.
  **有 `run-*` 但没有匹配的 verifier** → 把它的构建/启动原语当作你的抓手。
- **Neither** → cold start from README/package.json/Makefile. Timebox
  ~15min. Stuck → BLOCKED with exactly where, plus a filled-in
  `/run-skill-generator` prompt. Got through → **persist what you
  learned**: create `.claude/skills/verify/SKILL.md` at the level you
  probed above — repo root for a single-package repo; the touched
  package/app dir (`apps/desktop/.claude/skills/verify/SKILL.md`) in
  a monorepo where verification is per-package — capturing the
  build/launch/drive recipe that worked, so the next session skips
  this cold start. Keep it short: the commands that worked, the
  flows worth driving, any gotchas. A project verify skill already
  exists → edit it only when it steered you wrong: a documented
  command failed or turned out wrong, or a needed step it doesn't
  cover. Routine learnings don't warrant an edit, and never rewrite
  or reorganize existing content for style.
  **两者都没有** → 从 README/package.json/Makefile 冷启动。限时约 15 分钟。卡住 → 报 BLOCKED 并写明确切位置，附上一份填好的 `/run-skill-generator` 提示词。走通了 → **把学到的东西持久化**：在你上面探测的那一层创建 `.claude/skills/verify/SKILL.md`——单包仓库放在仓库根目录；在按包验证的 monorepo 中放在被触及的包/应用目录（`apps/desktop/.claude/skills/verify/SKILL.md`）——记录行之有效的构建/启动/驱动配方，让下一次会话免去这次冷启动。保持简短：有效的命令、值得驱动的流程、以及各种坑。项目 verify 技能已存在 → 只有当它把你带偏时才编辑它：文档中的命令失败或被证实有误，或它未覆盖某个必要步骤。常规性经验不足以触发编辑，且绝不要为了风格而重写或重组现有内容。

## Drive it / 驱动它

Smallest path that makes the changed code execute:

让被改动代码得以执行的最小路径：

- Changed a flag? Run with it.
  改了某个标志（flag）？带着它运行。
- Changed a handler? Hit that route.
  改了某个处理器？请求那条路由。
- Changed error handling? Trigger the error.
  改了错误处理？触发那个错误。
- Changed an internal function? Find the CLI command / request / render
  that reaches it. Run that.
  改了某个内部函数？找到能到达它的 CLI 命令 / 请求 / 渲染路径。运行它。

**Read your plan back before running.** If every step is build /
typecheck / run test file — you've planned a CI rerun, not a
verification. Find a step that reaches the surface or report BLOCKED.

**运行之前把计划重读一遍。** 如果每一步都是构建 / 类型检查 / 运行测试文件——那你计划的是重跑一次 CI，而不是验证。找一个能到达作用面的步骤，或者报告 BLOCKED。

**The verdict is table stakes. Your observations are the signal.**
A PASS with three sharp "hey, I noticed…" lines is worth more than a
bare PASS. You're the only reviewer who actually *ran* the thing —
anything that made you pause, work around, or go "huh" is information
the author doesn't have. Don't filter for "is this a bug." Filter for
"would I mention this if they were sitting next to me."

**结论只是及格线。你的观察才是有效信号。** 一个附带三条敏锐"咦，我注意到…"记录的 PASS，比一个光秃秃的 PASS 更有价值。你是唯一真正*运行过*这个东西的审查者——任何让你停顿、绕行或发出"咦？"的东西，都是作者所不知道的信息。不要按"这是不是 bug"来过滤。要按"如果作者坐在我旁边，我会不会提这件事"来过滤。

**End-to-end, through the real interface.** Pieces passing in
isolation doesn't mean the flow works — seams are where bugs hide.
If users click buttons, test by clicking buttons, not by curling the
API underneath.

**端到端，走真实接口。** 各部分单独通过不代表整个流程可用——缝隙正是 bug 藏身之处。如果用户是点击按钮的，就通过点击按钮来测试，而不是去 curl 底下的 API。

**Destructive path?** If the change touches code that deletes,
publishes, sends, or writes outside the workspace and there's no
dry-run or safe target, don't drive it live. Verify what you can
around it and say which path you didn't exercise and why.

**破坏性路径？** 如果改动触及删除、发布、发送或向工作区之外写入的代码，而又没有 dry-run 或安全目标，就不要真实驱动它。验证它周围你能验证的部分，并说明哪条路径你没有执行以及原因。

## Push on it / 施压试探

The claim checked out — that's the first half. Confirming is step
one, not the job. The description is what the author intended;
your value is what they didn't.

断言成立——这只是前一半。确认只是第一步，不是全部工作。描述是作者的本意；你的价值在于他们没想到的部分。

You know exactly what changed. Probe *around* it, at the same
surface you just drove:

你确切知道改了什么。在你刚刚驱动过的同一个作用面上，围绕它进行试探：

- **New flag / option** → empty value, passed twice, combined with a
  conflicting flag, typo'd (does the error name it?)
  **新标志 / 选项** → 空值、重复传两次、与冲突的标志组合、拼写错误（错误信息有没有点名它？）
- **New handler / route** → wrong method, malformed body, missing
  required field, oversized payload
  **新处理器 / 路由** → 错误的请求方法、格式错误的请求体、缺少必填字段、超大的载荷
- **Changed error path** → the adjacent errors it didn't touch —
  did the refactor catch them too, or only the one in the diff?
  **改动的错误路径** → 它没有触及的相邻错误——重构是否也覆盖了它们，还是只覆盖了 diff 里的那一个？
- **Interactive / TUI** → Ctrl-C mid-op, resize the pane, paste
  garbage, rapid-fire the key, Esc at the wrong moment
  **交互式 / TUI** → 操作中途 Ctrl-C、调整面板大小、粘贴乱码、连按按键、在错误时机按 Esc
- **State / persistence** → do it twice, do it with stale state
  underneath, do it in two sessions at once
  **状态 / 持久化** → 做两次、在底下留着过期状态时做、两个会话同时做
- **Wander** → what's adjacent? What looked off while you were
  confirming? Go back to it.
  **随意游走** → 相邻的是什么？确认过程中有什么看起来不对劲？回去看看它。

These aren't a checklist — pick the ones the change points at. Stop
when you've covered the obvious adjacents or hit something worth a
⚠️. A probe that finds nothing is still a step: "🔍 passed `--from ''`
→ clean `error: --from requires a value`, exit 2." That the author
didn't test it is exactly why it's worth knowing it holds.

这些不是核对清单——挑改动所指向的那些来做。覆盖完明显的相邻项、或碰到值得 ⚠️ 的东西时就停止。一无所获的试探仍然是一步："🔍 传入 `--from ''` → 得到干净的 `error: --from requires a value`，退出码 2。"正因为作者没测过它，才更值得知道它确实扛得住。

Still not a test run. You're at the surface, typing what a user
would type wrong.

这仍然不是在跑测试。你是在作用面上，输入用户会输错的内容。

## Capture / 采集

Stdout, response bodies, screenshots, pane dumps. Captured output is
evidence; your memory isn't. Something unexpected? Don't route around
it — capture, note, decide if it's the change or the environment.
Unrelated breakage is a finding, not noise.

stdout、响应体、截图、面板转储。采集到的输出才是证据；你的记忆不是。遇到意外？不要绕开它——先采集、记录，再判断是改动的问题还是环境的问题。无关的故障也是一项发现，不是噪音。

Shared process state (tmux, ports, lockfiles) — isolate. `tmux -L
name`, bind `:0`, `mktemp -d`. You share a namespace with your host.

共享的进程状态（tmux、端口、锁文件）——要隔离。用 `tmux -L name`、绑定 `:0`、`mktemp -d`。你与宿主机共享同一个命名空间。

## Report / 报告

Inline, final message:

以内联形式写在最终消息中：

```
## Verification: <one-line what changed>

**Verdict:** PASS | FAIL | BLOCKED | SKIP

**Claim:** <what it's supposed to do — your read of the diff and/or
the stated claim; note any mismatch>

**Method:** <how you got a handle — which verifier/run-skill, or
cold start; what you launched>

### Steps

Each step is one thing you did to the **running app** and what it
showed. Build/install/checkout are setup, not steps. Test runs and
typecheck don't belong here — they're CI's output.

1. ✅/❌/⚠️/🔍 <what you did to the running app> → <what you observed>
   <evidence: the app's own output — pane capture, response body,
   screenshot>

🔍 marks a probe — a step off the claim's happy path, trying to
break it. At least one. A Steps list that's all ✅ and no 🔍 is a
happy-path replay: still PASS, but you stopped at the first half.

**Screenshot / sample:** <the one frame a reviewer looks at to see
the feature — an image for GUI/TUI, code block for library/API;
omit for build/types-only>

### Findings
<Things you noticed. Not just bugs — friction, surprises, anything
a first-time user would trip on. "Took three tries to find the right
flag." "Error message on typo was unhelpful." "Default seems odd for
the common case." "Works, but slower than I expected." Lower the bar:
if it made you pause, it goes here. But the pause has to be yours,
from running the app — not from reading the PR page. A red CI check,
a review comment, someone else's bot: visible to anyone already, and
you relaying it isn't an observation. Claim/diff mismatch, pre-existing
breakage, and env notes also belong.

Each probe gets a line here even when it held — "🔍 empty `--from`
→ clean error" tells the author what *was* covered, which they
can't see from a bare PASS.

Lead with ⚠️ for lines worth interrupting the reviewer for; plain
bullets are context. Empty is fine if nothing stuck out — but nothing
sticking out is itself rare.>
```

**Evidence has to reach the reader.** A file path is only evidence
if the person reading the report can open it. If the `SendUserFile`
tool is in your toolset, you're on a remote surface where they
can't — send the screenshots and recordings with it and let the
report name what you sent. Without it, reference the path and keep
the evidence that matters inline — pane captures and response
bodies travel in the report; a bare path only works when the reader
shares your filesystem.

**证据必须能到达读者手中。** 只有当阅读报告的人能打开它时，一个文件路径才算证据。如果你的工具集中有 `SendUserFile` 工具，说明你处于远程作用面、对方无法直接打开——用它发送截图和录像，并在报告中写明你发送了什么。没有该工具时，引用路径并把关键证据内联保留——面板捕获和响应体应随报告走；光秃秃的路径只有在读者与你共享文件系统时才有效。

**Verdicts:**

**结论类型：**

- **PASS** — you ran the app, the change did what it should at its
  surface. Not: tests pass, builds clean, code looks right.
  **PASS** —— 你运行了应用，改动在其作用面上表现如预期。不是：测试通过、构建干净、代码看起来没问题。
- **FAIL** — you ran it and it doesn't. Or it breaks something else.
  Or claim and diff disagree materially.
  **FAIL** —— 你运行了它，但它没有做到。或者它破坏了别的东西。或者断言与 diff 有实质性出入。
- **BLOCKED** — couldn't reach a state where the change is observable.
  Build broke, env missing a dep, handle wouldn't come up. Not a
  verdict on the change. Never report an approach blocked or
  impossible until you've enumerated the skills along the touched
  subtree — environment-specific unlocks (headless runners, login
  helpers, VM harnesses) usually live there. Say exactly where it
  stopped + `/run-skill-generator` prompt.
  **BLOCKED** —— 未能到达可观察该改动的状态。构建失败、环境缺依赖、抓手起不来。这不是对改动本身的结论。在枚举完被触及子树沿线的技能之前，绝不要报告某种途径被阻塞或不可行——环境特定的解锁手段（无头运行器、登录辅助、VM harness）通常就在那里。写明确切的停止位置 + `/run-skill-generator` 提示词。
- **SKIP** — no runtime surface exists. Docs-only, types-only,
  tests-only. Nothing went wrong; there's just nothing here to run.
  One line why.
  **SKIP** —— 不存在运行时作用面。纯文档、纯类型、纯测试。没有出任何问题；只是这里没有可运行的东西。用一行说明原因。

No partial pass. "3 of 4 passed" is FAIL until 4 passes or is
explained away.

没有部分通过。"4 项过 3 项"就是 FAIL，直到第 4 项通过或得到解释。

**When in doubt, FAIL.** False PASS ships broken code; false FAIL
costs one more human look. Ambiguous output is FAIL with the raw
capture attached — don't interpret.

**拿不准时，判 FAIL。** 假 PASS 会把坏代码发布出去；假 FAIL 只是多花一次人工查看。输出含糊即判 FAIL，并附上原始采集内容——不要自行解读。

【评论】"存疑判 FAIL"是基于非对称代价的决策默认值：假阳性的代价（坏代码被发布）远高于假阴性的代价（多一次人工审查）。
