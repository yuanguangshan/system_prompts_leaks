<!-- BILINGUAL-EN-ZH -->
---
name: design-sync
description: Push a React design system to claude.ai/design. This runs a converter that bundles the real component code (from Storybook or a bare package) and uploads it. Use when the user runs /design-sync or says "sync my design system to Claude Design".
disable-model-invocation: true
---

# Sync a design system to claude.ai/design / 将设计系统同步到 claude.ai/design

## What this is for / 本技能的用途

**Claude Design** (claude.ai/design) is Claude's design tool: users prompt a design agent and it builds working UI - screens, flows, prototypes - rendered live in the browser from real React code. Out of the box it designs with generic components. This skill changes that: it converts the user's design-system repo into the format Claude Design consumes and uploads it, so from then on **the design agent builds with the customer's actual components** - every design it produces is on-brand, made of their real parts, and maps 1:1 onto code their engineers can ship.

**Claude Design**（claude.ai/design）是 Claude 的设计工具：用户向一个设计智能体下达提示，它便会构建可用的 UI——屏幕、流程、原型——由真实的 React 代码在浏览器中实时渲染。开箱即用时，它使用通用组件进行设计。本技能改变了这一点：它把用户的设计系统仓库转换为 Claude Design 所消费的格式并上传，从此**设计智能体用客户的真实组件进行构建**——它产出的每个设计都符合品牌、由客户的真实部件构成，并与工程师可直接发布的代码一一对应。

That framing should drive every judgment call in this skill, because each uploaded artifact is an input to that agent (or to the humans steering it):

这一框架应当驱动本技能中的每一个判断，因为每个上传的产物都是该智能体（或指导它的人类）的输入：

| Uploaded artifact | Consumed by | For |
|---|---|---|
| `_ds_bundle.js` + `_vendor/` | the design agent's runtime | every design it produces renders these real compiled components from `window.<globalName>.*` |
| `styles.css`, `fonts/`, `tokens/`, `_ds_bundle.css` | every rendered design | the look - tokens, fonts, and component styles, all reachable from `styles.css`'s `@import` closure (designs receive only that closure) |
| `<Name>.d.ts` (`<Name>Props`) | the design agent | the API contract it codes against |
| `<Name>.prompt.md` | the design agent | its usage reference - how to compose the component, with examples |
| `<Name>.html` preview card | humans in the component picker | how they find components and trust the sync |
| `_ds_sync.json` | future syncs | the sync anchor - content hashes that let a re-sync (any machine) skip re-verifying unchanged components AND compute exactly what to upload/delete |

| 上传的产物 | 由谁消费 | 用途 |
|---|---|---|
| `_ds_bundle.js` + `_vendor/` | 设计智能体的运行时 | 它产出的每个设计都从 `window.<globalName>.*` 渲染这些真实编译后的组件 |
| `styles.css`、`fonts/`、`tokens/`、`_ds_bundle.css` | 每一个被渲染的设计 | 外观——令牌、字体与组件样式，全部可从 `styles.css` 的 `@import` 闭包获取（设计只接收该闭包） |
| `<Name>.d.ts`（`<Name>Props`） | 设计智能体 | 它编码时依据的 API 契约 |
| `<Name>.prompt.md` | 设计智能体 | 它的使用参考——如何组合该组件，附示例 |
| `<Name>.html` 预览卡片 | 组件选择器中的人类 | 他们如何发现组件并信任本次同步 |
| `_ds_sync.json` | 未来的同步 | 同步锚点——内容哈希，让再次同步（任意机器）既能跳过对未变更组件的重新校验，又能精确计算需要上传/删除的内容 |

This is why fidelity is the whole game: a component that renders wrong here renders wrong in **every design the agent ever builds with it**, and a wrong `.d.ts` or misleading `.prompt.md` makes the agent misuse the API everywhere. The verification loops in the sub-skills exist because of this - they are not bureaucracy.

这就是为什么保真度是全部关键：一个在这里渲染错误的组件，会在**智能体用它构建的每一个设计**中都渲染错误；而一个错误的 `.d.ts` 或有误导性的 `.prompt.md` 会让智能体在各处误用该 API。子技能中的校验循环正因此存在——它们不是官僚流程。

【评论】本技能把上传产物视为下游设计智能体的输入，因此反复强调校验与保真度——这是一种"供应链"式的质量观：上游失真会在下游所有产出中被放大。

The converter builds all of the above deterministically from the repo's own `dist/`. With a Storybook, previews come from the repo's stories and are verified against its own storybook render (kept as a local reference, never uploaded). Without one, every component still ships fully functional, and rich previews are authored from the repo's own usage examples for the components the user scopes in, graded on an absolute rubric. **Core principle: ship what the customer already built** - the bundle is their compiled `dist/`, never a reimplementation.

转换器从仓库自身的 `dist/` 确定性地构建上述所有产物。若有 Storybook，预览来自仓库的 stories，并对照其自身的 Storybook 渲染进行校验（仅作为本地参考保留，从不上传）。若没有 Storybook，每个组件仍会以完整功能交付，而富预览则基于仓库自身的用法示例、为用户圈定的组件撰写，并按绝对化评分标准打分。**核心原则：交付客户已经构建好的东西**——打包内容是它们编译后的 `dist/`，绝不是重新实现。

You have a `DesignSync` tool that reads and writes the user's claude.ai/design projects. If a tool call fails with an authorization error, relay its guidance to the user verbatim - the tool's message is environment-aware (in an interactive terminal it names `/design-login`; in headless sessions like claude.ai/code it points at a path that works there) - and retry after they've acted on it.

你有一个可读写用户 claude.ai/design 项目的 `DesignSync` 工具。如果某次工具调用因授权错误而失败，请把工具给出的指引原样转达给用户——该工具的消息是感知环境的（在交互式终端中它会指明 `/design-login`；在 claude.ai/code 这类无头会话中，它会指向在那里可行的路径）——并在用户处理后重试。

## 0. First sync? Set expectations before any work / 0. 首次同步？先设定预期再动手

A completed sync always leaves `.design-sync/config.json` holding both a `projectId` and a `pkg`. If both are present, this is a re-sync - skip this section (§2 covers honoring prior state). (If `design-sync.config.json` exists instead - the config's old name and location - move it: `mkdir -p .design-sync && mv -n design-sync.config.json .design-sync/config.json`, commit the move, then apply the same test.) Anything less - no config at all, or a partial one left by a run that never finished - gets first-time treatment: tell the user up front, before doing anything else:

一次已完成的同步总会在 `.design-sync/config.json` 中同时留下 `projectId` 和 `pkg`。若两者都在，这就是一次再次同步——跳过本节（§2 说明如何尊重既有状态）。（如果存在的是 `design-sync.config.json`——该配置的旧名称与旧位置——先移动它：`mkdir -p .design-sync && mv -n design-sync.config.json .design-sync/config.json`，提交这次移动，然后套用同样的判断。）任何不及上述的状态——完全没有配置，或某次未完成运行留下的残缺配置——都按首次导入处理：在做任何事之前先告知用户：

- No completed sync was found - this is a first-time import.
  未发现已完成的同步——这是一次首次导入。
- This skill attempts a **high-fidelity** import of their design system: by default that means iterating on the build and visually verifying the quality of every component preview, which can take **up to a few hours** on a large repo.
  本技能会对用户的设计系统做**高保真**导入：默认意味着反复迭代构建并对每个组件预览的质量做视觉校验，在大型仓库上可能需要**长达数小时**。
- They can interrupt at any time - a message mid-run to check progress or redirect the effort is welcome and won't break anything.
  用户可随时打断——运行中途发消息查看进度或调整方向都是受欢迎的，且不会破坏任何东西。
- A first-time import goes into a **new Claude Design project created for it** (§1). Everything that needs their approval happens **near the start** - creating that project, and one approval that covers this run's uploads into it. After that, **verified components appear in the project as the run progresses**: they can open the project at any time and watch it fill in, and nothing waits on their approval at the end.
  首次导入会进入**为其新建的 Claude Design 项目**（§1）。所有需要用户批准的事项都发生在**开头附近**——创建该项目，以及一份覆盖本次运行全部上传的一次性批准。此后，**通过校验的组件会随运行推进陆续出现在项目中**：用户可以随时打开项目看着它逐步填满，结尾不会有任何事项等待批准。
- The run records config and notes as it goes, so future syncs are faster and mostly deterministic.
  运行过程中会随做随记配置与备注，因此未来的同步更快且基本是确定性的。

(If §1 routes this run into an existing project - the user re-adopting one, or a `projectId` left pinned by an aborted run - parts of this won't apply; scale the expectations to what §1 routes them to.)

（若 §1 把本次运行路由进一个既有项目——用户重新认领某个项目，或被中断运行留下的 `projectId` 固定项——其中部分内容不适用；请按 §1 的路由结果调整预期。）

Then confirm they want to proceed - this process can use a significant number of tokens (`AskUserQuestion`: proceed with the full high-fidelity sync, or adjust scope first). If their request already acknowledged the time/cost, note that and continue without re-asking.

然后确认用户是否想继续——此过程可能消耗大量 token（用 `AskUserQuestion` 询问：按完整高保真同步继续，还是先调整范围）。如果用户的请求已经认可了时间/成本，说明这一点并直接继续，无需再次询问。

## 1. Pick the target project / 选择目标项目

If `DesignSync` isn't already in your tool list, load it via `ToolSearch(query: "select:DesignSync")` first. A target gets picked one of three ways, in precedence order:

如果 `DesignSync` 尚不在你的工具列表中，先通过 `ToolSearch(query: "select:DesignSync")` 加载它。目标项目按以下三种方式、依优先级选定：

- **Pinned**: `.design-sync/config.json` has a `projectId` -> that's the target. `DesignSync(get_project)` to confirm it still exists and is `PROJECT_TYPE_DESIGN_SYSTEM`, mention which project you're syncing to, and re-ask only if it's gone or the user redirects.
  **已固定**：`.design-sync/config.json` 中有 `projectId` -> 它就是目标。用 `DesignSync(get_project)` 确认它仍存在且类型为 `PROJECT_TYPE_DESIGN_SYSTEM`，说明你正在向哪个项目同步，仅当它已消失或用户改变方向时才再次询问。
- **Fresh - the first-time default**: no pin -> **create a new project**. A fresh project is the only target whose entire contents this run owns; that ownership is what makes the incremental upload (§3) safe to approve in one shot, and it's why existing projects are never offered here - pouring a first import into a project that already has files would show a half-imported mix to anyone using it, with no sync anchor to tell its files apart from this run's. Use `DesignSync(list_projects)` to pick a NON-colliding name (a duplicate gets rejected and costs a round-trip), confirm the name via `AskUserQuestion`, and only then call `DesignSync(create_project)` - it raises its own permission prompt, and an unconfirmed creation can stall an unattended session. If that prompt is denied, stop and ask the user what to do differently; never retry unasked, never continue without a target. One salvage case: a project evidently left by a prior aborted run of this repo (it has the name this skill would propose - `list_files` it to confirm it's actually empty, since `list_projects` shows no file counts) may be offered for reuse instead of creating another, or noted as safe to delete.
  **全新——首次导入的默认选项**：没有固定项 -> **创建新项目**。全新项目是唯一一个其全部内容都归本次运行所有的目标；正是这种所有权使增量上传（§3）可以一次性安全批准，也是这里从不提供既有项目的原因——把首次导入灌进一个已有文件的项目，会让任何使用者看到半导入的混杂状态，且没有同步锚点来区分哪些文件属于本次运行。用 `DesignSync(list_projects)` 选一个不冲突的名称（重名会被拒绝并浪费一次往返），通过 `AskUserQuestion` 确认名称，然后才调用 `DesignSync(create_project)`——它会触发自己的权限提示，未经确认的创建可能让无人值守的会话卡住。若该提示被拒绝，停下来问用户想如何调整；绝不要未经要求就重试，也绝不要在没有目标的情况下继续。一种补救情形：一个明显由本仓库此前被中断的运行遗留的项目（名称正是本技能会提议的那个——用 `list_files` 确认它确实为空，因为 `list_projects` 不显示文件数）可以作为复用选项提出以避免另建一个，或标注为可安全删除。
- **Re-adopted - on the user's explicit ask only**: the user names an existing project (by name or UUID; typically re-adopting the project a previous sync uploaded to, after the config was lost). `DesignSync(get_project)`, check `type` is `PROJECT_TYPE_DESIGN_SYSTEM`, then warn them in plain language (no tool jargon) that syncing can overwrite or delete files already in it - e.g. "Heads up: syncing into that existing project means I may replace or remove files it already contains so it ends up matching this repo. If anything in there isn't from this repo, it could be lost - want me to continue, or create a fresh project instead?" - and proceed only on their confirmation. This explicit ask is the ONLY way an unpinned run ends up in a pre-existing project.
  **重新认领——仅限用户明确要求时**：用户点名一个既有项目（按名称或 UUID；通常是在配置丢失后，重新认领此前某次同步上传到的项目）。调用 `DesignSync(get_project)`，检查 `type` 是否为 `PROJECT_TYPE_DESIGN_SYSTEM`，然后用平实语言（不用工具行话）警告用户：同步可能覆盖或删除其中已有的文件——例如："注意：向那个既有项目同步意味着我可能替换或移除它已包含的文件，使其最终与这个仓库一致。如果里面有任何不是来自这个仓库的内容，都可能丢失——要我继续，还是改为创建一个全新项目？"——只有在用户确认后才继续。这一明确要求是无固定项的运行进入既有项目的唯一途径。

**Record the pin at settlement.** The moment the target is settled - created, reused, or re-adopted - **record its `projectId` in `.design-sync/config.json`**, before anything uploads. This is the skill's one recording rule: a death at any later point leaves a pinned config, so the retry repairs the SAME project through the atomic path instead of creating a duplicate and orphaning the original. (The post-upload record step in the sub-skills' atomic sections is just the backstop for this rule.)

**目标敲定即记录固定项。**目标一经敲定——无论新建、复用还是重新认领——就在任何上传发生之前，**把它的 `projectId` 记入 `.design-sync/config.json`**。这是本技能唯一的记录规则：此后任何时点的中断都会留下已固定的配置，于是重试会经由原子路径修复同一个项目，而不是创建重复项目并遗弃原项目。（子技能原子章节中上传后的记录步骤只是这条规则的兜底。）

**Route the upload path.** A `projectId` pinned **before this run started** always takes the **atomic path** (the sub-skill's upload section) - even when its project turns out empty; a bulk re-upload is fine there, and one rule beats a special case. Otherwise the remote decides, via a prompt-free `DesignSync(list_files)` on the target:

**划分上传路径。**在本运行**开始之前**就已固定的 `projectId` 总是走**原子路径**（子技能的上传章节）——即使该项目最终是空的；在那里做一次批量重传没有问题，一条规则胜过一个特例。否则由远端决定，方式是对目标做一次无提示的 `DesignSync(list_files)`：

- **Empty** (the normal case - this run just created it) -> **incremental path** (§3): one upfront approval, then verified components upload as the run progresses.
  **空**（正常情况——项目由本次运行刚创建）-> **增量路径**（§3）：一次事前批准，此后通过校验的组件随运行推进陆续上传。
- **Non-empty** (a re-adopted project) -> **atomic path**: it may be in active use, so it updates in one pass at the end of the run, after everything is verified.
  **非空**（重新认领的项目）-> **原子路径**：它可能正在被使用，因此在运行结束时、一切校验通过之后一次性整体更新。

The router decides only the **upload** path. **Verification** scope is the anchor's job: a project with `_ds_sync.json` lets the re-sync driver skip unchanged components; no anchor means everything gets verified, whichever upload path applies.

路由器只决定**上传**路径。**校验**范围由锚点决定：带有 `_ds_sync.json` 的项目可让再次同步驱动器跳过未变更的组件；没有锚点则一切都要校验，无论适用哪条上传路径。

## 2. Explore, then write config / 探查，然后写配置

The workflow is **explore the repo -> write `.design-sync/config.json` (§1's pin has already created the directory and the file - read it and add to it, never dropping `projectId`; `mkdir -p .design-sync` stays as a harmless safety net for legacy states) -> run the converter deterministically from it**. The converter's discovery is heuristic-based; each heuristic has a config override (after the sub-skill stages the scripts: `grep -r ASSUMPTION .ds-sync/*.mjs .ds-sync/lib/*.mjs` lists them) so repos that don't match the defaults write config, not code. Edit `lib/*.mjs` only as a last resort (see the sub-skill's escape-hatch section: storybook §5, package §Troubleshooting).

工作流是**探查仓库 -> 写 `.design-sync/config.json`（§1 的固定项已创建目录和文件——读取它并在其上追加，绝不丢掉 `projectId`；`mkdir -p .design-sync` 作为应对遗留状态的无害兜底保留）-> 从配置出发确定性地运行转换器**。转换器的发现机制基于启发式；每个启发式都有配置覆盖项（子技能暂存脚本之后：`grep -r ASSUMPTION .ds-sync/*.mjs .ds-sync/lib/*.mjs` 可列出它们），因此与默认值不符的仓库只需写配置，而不是改代码。只有万不得已才编辑 `lib/*.mjs`（见子技能的逃生舱章节：storybook §5、package §Troubleshooting）。

**The upload format is the contract; the converter is the deterministic path to it, not the only path.** What the app consumes is fully specified by the output layout: `_ds_bundle.js` + `@ds-bundle` header, `styles.css`, `components/<group>/<Name>/{.html,.jsx,.d.ts,.prompt.md}` with the `@dsCard` first line, `_preview/`, `_vendor/`, `fonts/`, `_ds_sync.json` (see the sub-skill's layout and upload sections).

**上传格式才是契约；转换器只是通往该格式的确定性路径，而非唯一路径。**应用所消费的内容完全由输出布局规定：`_ds_bundle.js` + `@ds-bundle` 头、`styles.css`、带 `@dsCard` 首行的 `components/<group>/<Name>/{.html,.jsx,.d.ts,.prompt.md}`、`_preview/`、`_vendor/`、`fonts/`、`_ds_sync.json`（见子技能的布局与上传章节）。

An off-script layout should also produce `_ds_sync.json` when it can. For the package shape, `lib/sync-hashes.mjs` gives `styleShaFor`/`renderHashFor`/`sourceKeyFor`; the envelope is `{shape, styleSha, renderHashes, sourceKeys, keyRecipe, scriptsSha, sourceHashes, auxSha, bundleSha12}` (see the sidecar block in `package-build.mjs` - `sourceHashes` itself comes from `stampHeader` in `lib/bundle.mjs`; `sourceKeys` may be omitted, which just means changed artifacts re-verify). The storybook shape's recipe needs story facts an off-script generator may not have; omitting the sidecar is then the honest choice - the next sync simply has no anchor and re-verifies everything, which is correct.

非标准（off-script）布局在能力所及时也应产出 `_ds_sync.json`。对于 package 形态，`lib/sync-hashes.mjs` 提供 `styleShaFor`/`renderHashFor`/`sourceKeyFor`；信封结构为 `{shape, styleSha, renderHashes, sourceKeys, keyRecipe, scriptsSha, sourceHashes, auxSha, bundleSha12}`（见 `package-build.mjs` 中的 sidecar 块——`sourceHashes` 本身来自 `lib/bundle.mjs` 的 `stampHeader`；`sourceKeys` 可省略，那只是意味着有变化的产物需重新校验）。storybook 形态的配方需要非标准生成器可能不具备的 story 事实；此时省略 sidecar 才是诚实的选择——下一次同步只是没有锚点、重新校验全部内容，这是正确的行为。

One invariant that's easy to miss when producing the layout by hand: rendered designs receive only `styles.css`'s transitive `@import` closure. Any real component CSS (`_ds_bundle.css`) must be `@import`ed from `styles.css` - a card linking it directly proves nothing about designs.

手工产出布局时容易漏掉的一条不变式：被渲染的设计只会收到 `styles.css` 的传递 `@import` 闭包。任何真正的组件 CSS（`_ds_bundle.css`）都必须从 `styles.css` 被 `@import`——一张直接链接它的卡片对设计说明不了任何问题。

For a repo genuinely outside the converter's envelope (non-esbuild-bundlable builds, exotic toolchains), produce the layout by whatever means the repo allows. The gates don't move: `package-validate.mjs` must exit clean, and every story must be graded before upload - from true screenshot pairs in the storybook shape, on the absolute rubric in the package shape. Off-script generation is legitimate; off-script *verification* is not.

对于真正超出转换器适用范围的仓库（无法用 esbuild 打包的构建、另类工具链），用该仓库允许的任何方式产出布局。门槛不变：`package-validate.mjs` 必须干净退出，且每条 story 在上传前都必须被打分——storybook 形态用真实截图配对，package 形态按绝对化评分标准。非标准*生成*是合法的；非标准*校验*不是。

**State from prior runs.** If `.design-sync/config.json` or `.design-sync/NOTES.md` already exist, Read both first and honor what's there - they hold corrections from earlier syncs. **Whenever the user tells you about an issue mid-run** (a path, a build flag, a component to skip, a package-manager quirk), persist it immediately so the next sync doesn't need telling again: a value that maps to a `cfg.*` field goes into `.design-sync/config.json`; anything else goes as a bullet in `.design-sync/NOTES.md`. Both get committed at the end (the sub-skill says when).

**先前运行的既有状态。**如果 `.design-sync/config.json` 或 `.design-sync/NOTES.md` 已存在，先 Read 两者并尊重其中内容——它们保存着早期同步的修正。**每当用户在运行中途指出一个问题**（某条路径、某个构建标志、某个要跳过的组件、某个包管理器的怪癖），立即持久化它，让下次同步无需再被告知：能映射到某个 `cfg.*` 字段的值写入 `.design-sync/config.json`；其余内容作为条目写入 `.design-sync/NOTES.md`。两者都会在结尾被提交（子技能说明了时机）。

1. **Faithful install with the repo's own package manager.** Use the repo's pinned node version (`.nvmrc` / `engines.node`), then detect via lockfile: `yarn.lock` -> `yarn install --immutable`; `pnpm-lock.yaml` -> `pnpm i --frozen-lockfile`; `bun.lockb`/`bun.lock` -> `bun install --frozen-lockfile`; `package-lock.json` -> `npm ci`.
   **用仓库自己的包管理器做忠实安装。**使用仓库固定的 node 版本（`.nvmrc` / `engines.node`），然后通过锁文件检测：`yarn.lock` -> `yarn install --immutable`；`pnpm-lock.yaml` -> `pnpm i --frozen-lockfile`；`bun.lockb`/`bun.lock` -> `bun install --frozen-lockfile`；`package-lock.json` -> `npm ci`。
2. **Determine the source shape.** If `.design-sync/config.json` already exists and has a `"shape"` field, use that. Otherwise `Glob` for `**/.storybook/main.*` and `**/storybook/main.*` (some repos drop the dot; exclude `node_modules`) - monorepo DSes keep it in a subpackage, so never assume it's at repo root:
   **判定源形态。**如果 `.design-sync/config.json` 已存在且带有 `"shape"` 字段，就用它。否则 `Glob` 查找 `**/.storybook/main.*` 与 `**/storybook/main.*`（有些仓库不带点；排除 `node_modules`）——monorepo 中的设计系统把它放在子包里，所以绝不要假定它在仓库根目录：
   - Any match -> `shape = 'storybook'`. The match's grandparent is the package to run from. Found several -> `AskUserQuestion` which one is the design system's; that dir becomes `storybookConfigDir`. **Do not fall back to package just because `.storybook` isn't at repo root.**
     有匹配 -> `shape = 'storybook'`。匹配项的祖父目录即运行所在包。找到多个 -> 用 `AskUserQuestion` 问哪个才是设计系统的；该目录即成为 `storybookConfigDir`。**不要仅因 `.storybook` 不在仓库根目录就退回 package 形态。**
   - Found `*.stories.*` files but no `.storybook/` dir in the target -> `AskUserQuestion`: "Found story files but no `.storybook/` here - is there a Storybook config elsewhere in this repo (e.g. `apps/storybook/.storybook` in a monorepo)?" If they point at one -> `shape = 'storybook'`, record that path as `storybookConfigDir`. If they say no -> `shape = 'package'`.
     找到 `*.stories.*` 文件但目标处没有 `.storybook/` 目录 -> 用 `AskUserQuestion` 询问："这里找到了 story 文件但没有 `.storybook/`——本仓库其他地方（比如 monorepo 中的 `apps/storybook/.storybook`）是否有 Storybook 配置？"若用户指了一处 -> `shape = 'storybook'`，把该路径记为 `storybookConfigDir`。若用户说没有 -> `shape = 'package'`。
   - No `.storybook/` and no `*.stories.*` -> `AskUserQuestion` whether a Storybook exists at all. If they point at one, record it as `storybookConfigDir` and `shape = 'storybook'`. If no, `shape = 'package'`.
     既没有 `.storybook/` 也没有 `*.stories.*` -> 用 `AskUserQuestion` 询问到底是否存在 Storybook。若用户指了一处，把它记为 `storybookConfigDir` 且 `shape = 'storybook'`。若没有，`shape = 'package'`。

Then `Read` `<skill-base-dir>/storybook/SKILL.md` or `<skill-base-dir>/non-storybook/SKILL.md` and follow it from there (the storybook one points back into the package one's shared tables where they overlap). Record `"shape"` (and `"storybookConfigDir"` when set) in `.design-sync/config.json` when you write it so re-sync skips detection. Both shapes run `<skill-base-dir>/package-build.mjs` as the converter entry and `<skill-base-dir>/resync.mjs` as the single re-sync driver (build -> diff -> validate -> scoped capture, one verdict JSON); shared adapters live at `<skill-base-dir>/lib/`, and `<skill-base-dir>/storybook/` holds the storybook-only harness (`compare.mjs` - preview-vs-storybook matching; `probe.mjs` - provider inference fallback).

然后 `Read` `<skill-base-dir>/storybook/SKILL.md` 或 `<skill-base-dir>/non-storybook/SKILL.md` 并从那里继续（两者重叠之处，storybook 版会指回 package 版的共享表格）。写 `.design-sync/config.json` 时把 `"shape"`（以及设置了的 `"storybookConfigDir"`）记录进去，让再次同步跳过探测。两种形态都以 `<skill-base-dir>/package-build.mjs` 为转换器入口，以 `<skill-base-dir>/resync.mjs` 为唯一的再次同步驱动器（构建 -> 差异 -> 校验 -> 定向采集，输出一份裁定 JSON）；共享适配器位于 `<skill-base-dir>/lib/`，`<skill-base-dir>/storybook/` 则存放仅 storybook 使用的工具（`compare.mjs`——预览与 Storybook 的比对；`probe.mjs`——provider 推断的回退）。

## 3. The incremental upload sequence (first syncs into an empty project) / 增量上传序列（首次同步进空项目）

On the incremental path (§1), the user approves the upload once, early, and then watches verified components appear in their project while the run is still going - instead of waiting hours for one bulk upload at the end. This section is the shared mechanics; the sub-skill says **when** each step fires (its own build and verification gates, marked "incremental path" there). The sub-skill upload section's mechanics apply to every write here too: <=256 files per `write_files` call and smaller chunks for binary-heavy dirs, upload hygiene, and the what-stays-local list.

在增量路径（§1）上，用户只在早期批准一次上传，然后在运行仍在进行时就看着通过校验的组件出现在自己的项目里——而不是等上几个小时等结尾的一次性批量上传。本节是共享机制；子技能规定每一步**何时**触发（其自身的构建与校验门槛，在那里标记为 "incremental path"）。子技能上传章节的机制同样适用于此处的每次写入：每次 `write_files` 调用不超过 256 个文件、二进制密集目录用更小的块、上传卫生，以及留在本地不传的清单。

### Open the upload channel - at the sub-skill's first-clean-build gate / 打开上传通道——在子技能首次干净构建的门槛处

1. **Explain the approval in plain language first.** Before asking, tell the user what they're about to approve, with no tool jargon (no "plan", "glob", or tool-method names): e.g. *"I'll ask for one approval now that covers uploading everything this run produces into the new project - and cleaning up any files a later rebuild drops. You won't be prompted again; components will appear in the project as they're verified."* The approval dialog shows a structured path list on its own; this message is what makes that dialog make sense to someone who's never synced before.
   **先用平实语言解释这次批准。**在请求之前，告诉用户将要批准什么，不用工具行话（不说 "plan"、"glob" 或工具方法名）：例如：*"我现在会请求一次批准，覆盖把本次运行产出的全部内容上传到新项目——以及清理后续重建删掉的文件。之后不会再提示你；组件通过校验后会出现在项目里。"* 批准对话框本身会显示结构化路径列表；这条消息是让从未同步过的人理解那个对话框的关键。
2. `DesignSync(finalize_plan)` with `localDir: "./ds-bundle"`, `writes: ["components/**", "tokens/**", "fonts/**", "_vendor/**", "_preview/**", "guidelines/**", "_ds_bundle.js", "_ds_bundle.css", "styles.css", "README.md", "_ds_sync.json", "_ds_needs_recompile"]`, and `deletes: ["components/**", "tokens/**", "fonts/**", "_vendor/**", "_preview/**", "guidelines/**"]`. The delete globs are what make the end-of-run reconciliation below prompt-free - and they're consent-trivial here: the project started empty, so anything deletable is something this same run uploaded. The returned `planId` serves the whole run (it lives for the session). Lost mid-run to a context reset -> `finalize_plan` again, one fresh approval, before uploading anything more. A whole-session death doesn't resume this path at all: the retry arrives pinned (§1) and correctly goes atomic - expected, not a bug to work around.
   调用 `DesignSync(finalize_plan)`，参数 `localDir: "./ds-bundle"`、`writes: ["components/**", "tokens/**", "fonts/**", "_vendor/**", "_preview/**", "guidelines/**", "_ds_bundle.js", "_ds_bundle.css", "styles.css", "README.md", "_ds_sync.json", "_ds_needs_recompile"]` 以及 `deletes: ["components/**", "tokens/**", "fonts/**", "_vendor/**", "_preview/**", "guidelines/**"]`。正是这些删除 glob 让下文运行结束时的对账无需再提示——而且在这里同意几乎不构成负担：项目开始时为空，所以任何可删除的东西都是本次运行自己上传的。返回的 `planId` 服务整个运行（它在会话期内有效）。若运行中途因上下文重置而丢失 -> 再次 `finalize_plan`，重新获得一次批准，然后才能继续上传。整个会话的死亡则完全不在此路径上恢复：重试到来时带有固定项（§1）并正确地走原子路径——这是预期行为，不是需要绕过的 bug。
3. **If the approval is denied, stop and ask - never continue silently, never re-prompt unasked.** Say in plain language what was denied and what it covered ("the one-time approval for uploading this run's output into the new project"), then offer: try the approval again; target a different project; or finish the build and verification locally with no upload. Local-only -> the run proceeds normally except nothing uploads, and the end-of-run report hands over both the `ds-bundle/` path and the project's URL (`https://claude.ai/design/p/<projectId>` - the pin is already recorded, so a later sync finds this project rather than orphaning it). A different project -> it goes through §1's re-adoption ask and the router like any other explicit choice, pin included: non-empty -> atomic path, this plan abandoned; empty -> resume here with a fresh approval.
   **如果批准被拒绝，停下来询问——绝不悄悄继续，绝不未经要求再次弹提示。**用平实语言说明被拒绝的是什么、它覆盖了什么（"把本次运行产出上传到新项目的一次性批准"），然后提供选项：再试一次批准；换一个目标项目；或只在本地完成构建与校验、不上传。仅本地 -> 运行照常进行，只是什么都不上传，运行结束的报告同时交出 `ds-bundle/` 路径与项目 URL（`https://claude.ai/design/p/<projectId>`——固定项已记录，之后的同步会找到这个项目而不是遗弃它）。换项目 -> 它与任何其他明确选择一样走 §1 的重新认领询问与路由，包括固定项：非空 -> 原子路径，放弃本计划；为空 -> 在这里用一次新的批准继续。

### Push each verified batch / 推送每一批通过校验的组件

Nothing uploads until the first batch of components passes the sub-skill's done-bar. **The first push carries the shared base files together with that first batch**: `_ds_bundle.js`, `_ds_bundle.css`, `styles.css`, `README.md`, `_vendor/**`, `tokens/**`, `fonts/**`, `guidelines/**`, plus the batch's `components/<group>/<Name>/` dirs and `_preview/<Name>.*` files. Two reasons they travel together: the first thing the user sees in the project is real components, not an empty shell that claims something was uploaded - and by first-batch time the shared files have earned their place, because grading those components exercised the very same bundle, CSS, and fonts. This first push is the project's first content and its largest, so it takes the full fence: sentinel first (`write_files` `_ds_needs_recompile` - it fences the app's manifest/copy machinery against a half-uploaded state), then the files, then the sentinel re-write (every push on this path ends by re-writing the sentinel - that's what makes the app refresh its view of the project next time it's opened). Output the project URL prominently with this push - `https://claude.ai/design/p/<projectId>` - it's the moment the project first has something to see.

在第一批组件通过子技能的完成线之前，什么都不上传。**第一次推送把共享基础文件与第一批组件一起携带**：`_ds_bundle.js`、`_ds_bundle.css`、`styles.css`、`README.md`、`_vendor/**`、`tokens/**`、`fonts/**`、`guidelines/**`，外加该批次的 `components/<group>/<Name>/` 目录和 `_preview/<Name>.*` 文件。它们同行有两个理由：用户在项目中最先看到的应是真实组件，而不是一个声称已上传内容的空壳——而且到第一批时，共享文件已经挣得自己的位置，因为给那些组件打分所检验的正是同一套 bundle、CSS 和字体。这第一次推送是项目的首批内容也是最大的一次，所以要加完整围栏：先哨兵（`write_files` 写入 `_ds_needs_recompile`——它为应用的清单/复制机制加上防半上传状态的围栏），然后是文件，然后重写哨兵（此路径上的每次推送都以重写哨兵收尾——正是它让应用在下次打开时刷新对项目的视图）。随这次推送醒目地输出项目 URL——`https://claude.ai/design/p/<projectId>`——这是项目第一次有东西可看的时刻。

【评论】`_ds_needs_recompile` 哨兵是典型的两阶段提交式写保护：先立标志、写入内容、再解除标志，避免消费方读到半套状态；锚点文件 `_ds_sync.json` 则刻意放在最后一步提交。

Every later batch that passes the done-bar: `write_files` its `components/<group>/<Name>/` dirs and `_preview/<Name>.*` files, then re-write the sentinel - the new cards appear next time the user opens or refreshes the project. When you report batch progress, include the project URL so the new cards are one click away. If a full rebuild has run since the last push (a global config fix landed), include the shared base files again: the fix rewrote the bundle/CSS/fonts locally, and without re-pushing them every component verified after it renders against stale remote versions until close-out. They're in the approved plan and idempotent, so the re-push costs nothing.

之后每一批通过完成线的批次：`write_files` 其 `components/<group>/<Name>/` 目录与 `_preview/<Name>.*` 文件，然后重写哨兵——新卡片会在用户下次打开或刷新项目时出现。报告批次进度时附上项目 URL，让新卡片一键可达。如果自上次推送后发生过一次完整重建（落地了某个全局配置修复），要再次包含共享基础文件：那次修复在本地重写了 bundle/CSS/字体，若不重推，其后校验的每个组件在收尾之前都渲染在过时的远端版本上。它们已在批准的计划内且操作幂等，重推没有额外代价。

Later batch pushes need no leading fence - they're short and always end re-armed, so the unfenced window is negligible (the first push above and the long close-out below are the ones that fence first). And batches are progressive visibility, not the correctness mechanism: the close-out guarantees the final state, so don't agonize over batch composition - a component pushed early then reworked later simply gets re-pushed.

后续批次推送无需前置围栏——它们短小且总是以重新武装的状态收尾，无围栏窗口可以忽略不计（上文第一次推送与下文冗长的收尾才是先加围栏的）。批次提供的是渐进可见性，不是正确性机制：最终状态由收尾保证，因此不必为批次组成纠结——先推送、后来返工的组件重推一次即可。

### Close out - after the sub-skill's final gate / 收尾——在子技能的最终门槛之后

1. **Sentinel first, then full content writes.** Re-write `_ds_needs_recompile` before anything else - the app clears the sentinel whenever the user opens the project (which this path invites mid-run), and the close-out is the longest write+delete stretch, so re-fencing here is what keeps a half-applied state from ever being consumed. Then everything in the plan's writes EXCEPT `_ds_sync.json`, chunked. Re-uploading unchanged files is idempotent and cheap; this pass covers anything the batches missed and anything the final rebuild changed, so the project ends up exactly matching the final verified build no matter how the batches went.
   **先哨兵，再全量内容写入。**做任何事之前先重写 `_ds_needs_recompile`——用户每次打开项目，应用都会清除哨兵（本路径主动邀请运行中途打开），而收尾是最长的写入+删除阶段，因此在这里重新加围栏才能确保半套状态永远不会被消费。然后按块写入计划 writes 中除 `_ds_sync.json` 外的一切。重传未变化的文件是幂等且廉价的；这一遍覆盖批次遗漏的与最终重建改动的所有内容，因此无论批次进展如何，项目最终都精确匹配最终校验过的构建。
2. **Reconciliation deletes - mandatory, not conditional.** `DesignSync(list_files)` the project and `delete_files` every remote path under `components/`, `_preview/`, `tokens/`, `fonts/`, `_vendor/`, `guidelines/` that the final `ds-bundle/` does not contain (the plan's delete globs cover them - no new prompt). Why this pass exists: a component uploaded by an earlier batch and then dropped, renamed, or regrouped later in the run is invisible to every future re-sync diff - anchor-based diffs only see what the anchor records - so this is the only moment it can ever be cleaned up; skip it and the orphan is permanent. The deletes also retire the orphan's card: the app rebuilds its component index from the currently-uploaded files, so the card disappears once the sentinel is re-armed (next step) and the project is opened.
   **对账删除——强制执行，不是可选。**用 `DesignSync(list_files)` 列出该项目，并 `delete_files` 删除 `components/`、`_preview/`、`tokens/`、`fonts/`、`_vendor/`、`guidelines/` 之下最终 `ds-bundle/` 不包含的每个远端路径（计划的删除 glob 已覆盖它们——无需新提示）。这一遍存在的原因：早先批次上传、随后在运行中被丢弃、改名或重组的组件，对未来所有再次同步的差异比对都不可见——基于锚点的差异只能看到锚点记录的内容——所以这是它唯一能被清理的时机；跳过则孤儿永久留存。这些删除还会让孤儿的卡片退役：应用从当前已上传文件重建组件索引，因此哨兵重新武装（下一步）且项目被打开后，卡片即消失。
3. **Sentinel re-arm, then `_ds_sync.json` absolutely last**, in its own `write_files` call - same rule, same reason as the atomic path: the anchor must only ever vouch for a fully-applied state, and it goes after the deletes so a failed delete can't leave remote files the anchor no longer sees. Then output the project URL - `https://claude.ai/design/p/<projectId>` - with the final summary.
   **哨兵重新武装，然后 `_ds_sync.json` 绝对最后**，单独一次 `write_files` 调用——与原子路径同样的规则、同样的理由：锚点只可为完全应用后的状态作保，且它放在删除之后，这样一次失败的删除就不会留下锚点再也看不到的远端文件。然后在最终总结中输出项目 URL——`https://claude.ai/design/p/<projectId>`。

A mid-run abort anywhere on this path (user stops the run, session dies) leaves the project **un-anchored** - the documented safe state: the next sync re-verifies everything and re-uploads, nothing silently rots. And as in the sub-skill upload sections, any write/delete failure that retries don't clear means **STOP** - no sentinel re-arm, no `_ds_sync.json`.

此路径上任何位置的中途中止（用户停止运行、会话死亡）都会让项目处于**无锚点**状态——这是文档化的安全状态：下一次同步会重新校验一切并重新上传，不会有东西悄悄腐烂。与子技能上传章节一样，任何重试都无法消除的写入/删除失败都意味着**停止**——不重新武装哨兵，不写 `_ds_sync.json`。

## Author the conventions header / 撰写约定头（conventions header）

You've just spent real effort making this design system's previews render - working out how components must be wrapped, what provider and theme setup they need, what load order matters, and which mistakes silently produce unstyled output. That knowledge evaporates when the sync ends unless you write it down here, for a very specific reader.

你刚刚花了真功夫让这套设计系统的预览得以渲染——弄清组件必须怎样包裹、需要什么 provider 与主题设置、加载顺序何者重要、以及哪些错误会悄悄产出无样式输出。除非你在此处为一位非常特定的读者把这些知识写下来，否则它们会随同步结束而蒸发。

**Who reads it.** The file you author is prepended to the generated README (via the `readmeHeader` config key) and inlined into the system prompt of a *design agent* - a model that builds apps WITH this component library, hundreds of times, for users who never see this file. It won't make storybook previews, run this repo's build, or read its source; it gets the README and the bound artifacts, nothing else. An agent in that position follows concrete, enumerated guidance and cannot follow guidance that isn't there: name the tokens and it uses tokens; leave the class vocabulary unnamed and it won't guess at yours - it will invent its own. Say to wrap in the provider and it wraps; don't, and it mostly won't. So every sentence must pass one test: *could the design agent act on this without guessing?* ("Follow the design system's conventions" fails that test; delete it and write the convention.)

**谁来读它。**你撰写的文件会被前置到生成的 README（经由 `readmeHeader` 配置键），并内联进一个*设计智能体*的系统提示词——这是一个用这套组件库构建应用的模型，成百上千次，为永远看不到这个文件的用户服务。它不会做 Storybook 预览、不会跑这个仓库的构建、也不会读源码；它拿到的只有 README 与绑定的产物，别无其他。处于这种位置的智能体只遵循具体、逐条列出的指引，无法遵循不存在的指引：说出令牌的名字，它就用令牌；不点明类名词汇表，它不会去猜你的——它会发明自己的一套。说要包在 provider 里它就包；不说，它多半不包。因此每句话都要通过一个测试：*设计智能体能否不靠猜测就据此行动？*（"遵循设计系统的约定"通不过这个测试；删掉它，把约定本身写出来。）

【评论】约定头会被内联进另一个智能体的系统提示词，本节实质上是在要求为下游模型撰写"可直接执行、可机械验证"的提示词内容，而非给人看的泛泛说明。

**What to write** - four concerns, in whatever structure serves this DS:

**写什么**——四个关注点，用任何适合这套设计系统的结构组织：

- **Wrapping and setup.** If components need a provider/root wrapper to be styled (it's usually where the tokens and theme live), name it, say what breaks without it, and show the wrap in a minimal snippet - plus theme setup, load order, and any gotcha that cost you a preview debugging cycle. Filter by the reader's job: it builds apps, not previews - harness-specific setup (storybook quirks, scaffolding) goes to NOTES.md; what matters for building with the components goes here.
  **包裹与设置。**如果组件需要 provider/根包裹器才能获得样式（令牌与主题通常就在那里），点名它、说明没有它会坏什么，并用一个最小片段展示包裹方式——外加主题设置、加载顺序，以及任何让你耗掉一轮预览调试的坑。按读者的职责过滤：它构建应用，不做预览——特定于测试架的设置（Storybook 怪癖、脚手架）归 NOTES.md；与用这些组件构建应用相关的才写在这里。
- **The styling idiom, with its actual vocabulary.** Teach THIS system's idiom, never a generic one: utility-class systems get a compact family table with real names from the styling source (a Tailwind preset enumerates them exactly); prop/theme systems get "no CSS classes - style via props" with the props that carry the design language; token systems get the `var(--*)` pattern with real names. Never import an idiom the DS doesn't have.
  **样式惯用法，连同它真实的词汇。**教这一个系统的惯用法，绝不教通用套路：工具类系统给一张紧凑的家族表，用样式源里的真实名称（Tailwind preset 会精确枚举它们）；prop/主题系统写"没有 CSS 类——通过 props 设样式"，并列出承载设计语言的那些 props；令牌系统写 `var(--*)` 模式加真实名称。绝不引入这套设计系统没有的惯用法。
- **Where the truth lives.** Name the stylesheet/source files the agent should read before styling (the bound copies it will have, e.g. `_ds/<folder>/styles.css` and its imports) and the per-component docs. An agent that reads the real files beats any summary - your job is making sure it knows where to look.
  **真相在何处。**点名智能体在写样式前应读的样式表/源文件（它将拥有的绑定副本，例如 `_ds/<folder>/styles.css` 及其导入）以及每个组件的文档。读真实文件的智能体胜过任何摘要——你的职责是让它知道去哪里找。
- **One idiomatic build snippet.** A short, real example - a library component for the control, the DS's styling idiom for the agent's own layout glue. Adapt one of your verified previews: it's code you know renders.
  **一段地道的构建片段。**一个简短、真实的例子——控件用库组件，智能体自己的布局粘合代码用这套设计系统的样式惯用法。改编自你某个已校验的预览：那是你确认过能渲染的代码。

Across different kinds of systems that looks like (illustrative, not exhaustive): a Tailwind-preset DS -> family table (`bg-surface-1`, `gap-md`, `text-body`...) + root wrapper; a grommet-style DS -> no classes, `pad`/`background`/`tone` props + ThemeProvider; a chakra-style DS -> theme-token strings (`color="red.500"`); a CSS-modules/BEM DS -> the exported class maps and whether new names are ever legitimate; a web-components DS -> slots, attributes, and registration order.

跨不同种类的系统，这看起来像（示例性，非穷尽）：Tailwind-preset 设计系统 -> 家族表（`bg-surface-1`、`gap-md`、`text-body`……）+ 根包裹器；grommet 风格设计系统 -> 无类，`pad`/`background`/`tone` props + ThemeProvider；chakra 风格设计系统 -> 主题令牌字符串（`color="red.500"`）；CSS-modules/BEM 设计系统 -> 导出的类映射，以及新名称何时才算合法；web-components 设计系统 -> slots、attributes 与注册顺序。

**Validate before shipping.** A conventions file that names things which don't exist is worse than none - the agent will trust it, write vocabulary that doesn't resolve, and ship silently unstyled output. Before committing: every class, token, prop, and component you enumerated must exist in the built artifacts - grep classes/tokens against the compiled stylesheets in the output dir; check named components against the `components/<group>/<Name>/` directories in the output dir (the build you just ran emits one per component - that tree is the sync-time name index; `.ds-build-meta.json` carries only counts), then the bundle text (authoritative - e.g. a provider like the root wrapper ships in the bundle without a component folder) before cutting a claim. Verifies in neither -> fix the name or cut it; documented in source but absent from the build -> that's a NOTES.md finding, not header content.

**交付前先验证。**一份点名了不存在之物的约定文件比没有更糟——智能体会信任它，写出解析不到的词汇，悄悄交付无样式的输出。提交之前：你枚举的每一个类、令牌、prop 和组件都必须存在于构建产物中——用 grep 在输出目录的编译后样式表中核对类/令牌；对照输出目录中的 `components/<group>/<Name>/` 目录核对点名的组件（你刚跑的构建为每个组件生成一个——这棵树就是同步时的名称索引；`.ds-build-meta.json` 只带数量），再对照 bundle 文本（权威来源——例如根包裹器这类 provider 会随 bundle 发布而没有组件目录），然后才落下一个断言。两处都验证不到 -> 修正名称或删掉它；源码有文档但构建中没有 -> 那是 NOTES.md 的发现，不是头部内容。

**Budget.** Be terse - 2-4k characters covers all four concerns, and real names beat vagueness. If the build's size warning fires, read which side it names. Header-side (the header alone exceeds ~31.9k): shorten the header - it survives inline truncation only while it itself fits the ~32k window; past that, its own tail is cut and the body contributes nothing. Body-side: your conventions are safe (prepended, within-window); what's lost is the END of the generated body - typically the component index's tail. Accept that loss deliberately, or reduce the synced surface (package shape: `componentSrcMap` exclusions, a narrower `tokensGlob`; storybook shape: sync fewer stories) - there is no body-section trim knob.

**预算。**要简练——2-4k 字符即可覆盖全部四个关注点，且真实名称胜过含糊其辞。如果构建的尺寸警告触发了，看清它点的是哪一侧。头部一侧（仅头部就超过约 31.9k）：缩短头部——只有当它自身装进约 32k 窗口时，它才能在内联截断下幸存；超过之后，它自己的尾部被截掉，而正文毫无贡献。正文一侧：你的约定是安全的（前置且在窗口内）；损失的是生成正文的末尾——通常是组件索引的尾部。要么有意识地接受这一损失，要么缩小同步面（package 形态：`componentSrcMap` 排除项、更窄的 `tokensGlob`；storybook 形态：同步更少的 story）——不存在针对正文分节的裁剪旋钮。

**Where it lives, and reruns.** Write `.design-sync/conventions.md`, set `"readmeHeader": ".design-sync/conventions.md"`, commit both - it's deliberately human-editable. Then rebuild so the README actually carries the header - it's stitched at build time. **The rebuild rule:** the post-authoring rebuild is a fresh DRIVER run on every path - first syncs omit `--remote` - because the closing receipt and the upload plan must both describe the header-bearing build; a bare converter run wipes `.sync-diff.json` and the receipt artifacts, leaving the uploaded build unreceipted. (Every other mention of the post-authoring rebuild defers to this rule.) Whenever the file already exists - regardless of how this run was classified (re-sync, re-adoption after a lost config, recovery from a partial one): never rewrite it - re-run the validation pass against the fresh build and report any name that no longer verifies (NOTES.md + user), proposing edits. Authoring happens only when no `.design-sync/conventions.md` exists. Content belongs to its authors; your standing job is keeping it true.

**它放在哪里，以及重跑。**写 `.design-sync/conventions.md`，设置 `"readmeHeader": ".design-sync/conventions.md"`，两者都提交——它被刻意设计为人类可编辑。然后重新构建，让 README 真正带上这个头部——它在构建时缝合。**重建规则：**撰写后的重建在任何路径上都是一次全新的 DRIVER 运行——首次同步省略 `--remote`——因为收尾回执与上传计划都必须描述携带头部的构建；一次纯粹的转换器运行会清掉 `.sync-diff.json` 与回执产物，让已上传的构建没有回执。（其他所有提到撰写后重建之处都以这条规则为准。）只要该文件已存在——无论本次运行如何归类（再次同步、配置丢失后的重新认领、从残缺状态恢复）：绝不重写它——对照新构建重跑校验那一遍，报告任何不再能通过验证的名称（写进 NOTES.md 并告知用户），并提出修改建议。只有当 `.design-sync/conventions.md` 不存在时才撰写。内容属于其作者；你的长期职责是让它保持为真。


## Hint / 提示

```
$ARGUMENTS
```
