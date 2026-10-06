<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Onboarding Flow / 托管代理 - 上手引导流程

> **Invoked via `/claude-api managed-agents-onboard`?** You're in the right place. Run the interview below - don't summarize it back to the user, ask the questions.

> **是通过 `/claude-api managed-agents-onboard` 触发的吗？** 你来对地方了。请执行下面的访谈流程——不要把内容总结复述给用户，而是逐条提问。

Claude Managed Agents is a hosted agent: Anthropic runs the agent loop and provisions a sandboxed container per session where the agent's tools execute (or your own worker, with a `self_hosted` environment - see `shared/managed-agents-self-hosted-sandboxes.md`). You supply an **agent config** (tools, skills, model, system prompt - reusable, versioned) and an **environment config** (the sandbox - reusable across agents). Each run is a **session**.

Claude Managed Agents 是一种托管式代理：Anthropic 负责运行代理循环，并为每个会话预配一个沙箱容器，代理的工具在其中执行（使用 `self_hosted` 环境时则由你自己的 worker 执行——参见 `shared/managed-agents-self-hosted-sandboxes.md`）。你提供**代理配置**（工具、技能、模型、系统提示词——可复用、带版本）和**环境配置**（沙箱——可跨代理复用）。每一次运行就是一个**会话（session）**。

The flow is four beats - **describe -> agent -> environment -> session** - the same arc as the Console quickstart, and the same philosophy: **value before credentials**. The user goes from idea to a runnable session before any auth ask; each credential is *flagged* at the moment the design makes it relevant (§2) and *collected* once, at session setup (§4), where it binds (`sessions.create()`) and gets exercised (smoke-test). Read `shared/managed-agents-core.md` alongside this - it has full detail for each knob; this doc is the interview script.

整个流程分四拍：**描述 -> 代理 -> 环境 -> 会话**——与 Console 快速上手相同的叙事线，也是相同的理念：**先交付价值，后索取凭据**。用户在任何鉴权请求之前就能完成从想法到可运行会话的过程；每个凭据都在设计使其产生关联的时刻被*标记*（§2），并在会话搭建阶段（§4）一次性*收集*——在那里它被绑定（`sessions.create()`）并得到实际验证（冒烟测试）。请同时阅读 `shared/managed-agents-core.md`——那里有每个配置项的完整细节；本文档则是访谈脚本。

---

## 1. Describe the task / 描述任务

**Open with a one-breath signpost and a single open prompt - don't guess, don't questionnaire.** In your own words:

**开场先给一句话路标，再提一个开放式问题——不要靠猜，也不要做成问卷。** 用你自己的话：

> Managed Agents is hosted - Anthropic runs the agent loop, the sandbox, and the infrastructure; you just define the agent. We'll do this in three moves: the agent, the environment it runs in, then a live test session. So: describe the agent you want - what should it do, and what kicks it off (a person, an event, a schedule)?

> Managed Agents 是托管的——代理循环、沙箱和基础设施都由 Anthropic 运行；你只需定义代理。我们分三步进行：定义代理、定义它的运行环境，然后做一次实际测试会话。那么：请描述你想要的代理——它应该做什么，由什么触发（某个人、某个事件，还是某个日程）？

Let them answer in full before configuring anything.

先让用户完整作答，再开始任何配置。

## 2. Configure the agent - propose, don't interrogate / 配置代理——提出方案，而非连环追问

Their description does the interview's work. Draft the agent config from it and **present it as a proposal with your suggestions inline** - the user reacts to a concrete config instead of answering a question list. At most one batched follow-up for true gaps. Suggest where the description gives you an opening:

用户的描述已经承担了访谈的工作。据此起草代理配置，并**以方案形式呈现、把你的建议内联其中**——让用户针对具体配置作出反应，而不是回答一长串问题。对真正的信息缺口至多做一次集中追问。在描述给你留出切入点的位置提出建议：

- **Tools** - enable the full prebuilt toolset by default (`agent_toolset_20260401`: `bash`, `read`, `write`, `edit`, `glob`, `grep`, `web_fetch`, `web_search`). **Suggest MCP servers** for any third-party service the job names (GitHub, Linear, Slack, ...) - and flag the credential each one implies as you suggest it ("Linear MCP -> you'll need a Linear API token at kickoff"), so §4's auth step is a formality, not a surprise. Collection itself waits for §4. Custom tools only if the user's own app must answer calls (name, description, input schema - their handler code is theirs; don't generate it).
  **工具（Tools）**——默认启用完整的预构建工具集（`agent_toolset_20260401`：`bash`、`read`、`write`、`edit`、`glob`、`grep`、`web_fetch`、`web_search`）。对任务中提到的任何第三方服务（GitHub、Linear、Slack 等）**建议相应的 MCP 服务器**——并在建议的同时点出各自隐含的凭据（"Linear MCP -> 启动时你需要一个 Linear API token"），这样 §4 的鉴权步骤就只是走个流程，而不会出人意料。凭据收集本身留到 §4。只有当用户自己的应用必须响应工具调用时才使用自定义工具（名称、描述、输入 schema——处理代码归用户所有；不要代为生成）。
- **Skills** - **suggest** prebuilt `xlsx`/`docx`/`pptx`/`pdf` when the job produces those artifacts; custom by `skill_id` (max 20 total per agent, prebuilt + custom combined).
  **技能（Skills）**——当任务会产出相应文件时**建议**使用预构建的 `xlsx`/`docx`/`pptx`/`pdf`；自定义技能通过 `skill_id` 引用（每个代理总计最多 20 个，预构建与自定义合并计算）。
- **Outcome - the default kickoff for any job with a deliverable.** If the job produces something checkable (an artifact, a report, a PR, a dataset), draft a starter rubric from the description - explicit, independently gradeable criteria: not "a good report" but "a CSV with a numeric `price` column per SKU" - and propose it inline with the config; the harness grades and iterates against it (`shared/managed-agents-outcomes.md`). The user not having a rubric is not a reason to skip this - drafting one is your job; mark it as a starter to tune. Fall back to a conversational kickoff only when the job is genuinely interactive (a chat surface, human-in-the-loop steering).
  **Outcome（成果）——任何有交付物的任务的默认启动方式。** 如果任务会产出可检验的东西（一个 artifact、一份报告、一个 PR、一个数据集），就根据描述起草一份初始评分标准（rubric）——明确、可独立评定的标准：不是"一份好报告"，而是"一份每个 SKU 都带数值型 `price` 列的 CSV"——并随配置一并内联提出；执行框架会据此评分并迭代（`shared/managed-agents-outcomes.md`）。用户没有评分标准不是跳过这一步的理由——起草标准是你的职责；把它标注为待调优的初稿。只有当任务本质上是交互式的（聊天界面、人在回路中引导）时才退回到对话式启动。
- **On-hand resources** - repos on disk (`github_repository`: URL, optional `mount_path`/`checkout`; token comes in §4), files to seed (Files API upload -> `{type: "file", file_id, mount_path}`; read-only), if the job references them.
  **在手的资源**——磁盘上已有的仓库（`github_repository`：URL，可选 `mount_path`/`checkout`；token 在 §4 提供）、需要预置的文件（经 Files API 上传 -> `{type: "file", file_id, mount_path}`；只读），以任务提及者为限。
- **Model** - default `claude-opus-5-5`; `claude-fable-5-1` for the hardest long-horizon work (`shared/model-migration.md` -> Migrating to Claude Fable 5.1).
  **模型（Model）**——默认 `claude-opus-5-5`；最困难的长程任务用 `claude-fable-5-1`（`shared/model-migration.md` -> Migrating to Claude Fable 5.1）。

> Important: **PR creation needs the GitHub MCP server too** - a `github_repository` mount is filesystem-only. Edit in the mount -> push branch via `bash` -> open the PR via the MCP `create_pull_request` tool.

> 重要：**创建 PR 还需要 GitHub MCP 服务器**——`github_repository` 挂载只有文件系统能力。在挂载中编辑 -> 通过 `bash` 推送分支 -> 通过 MCP `create_pull_request` 工具发起 PR。

Full detail per knob: `shared/managed-agents-tools.md` (toolset, MCP, custom tools, skills), `shared/managed-agents-environments.md` (repos, files).

每个配置项的完整细节：`shared/managed-agents-tools.md`（工具集、MCP、自定义工具、技能）、`shared/managed-agents-environments.md`（仓库、文件）。

## 3. Environment / 环境

Usually zero or one question:

通常零个或一个问题：

- **Reuse or create?** Environments are shared across agents - check for an existing one first.
  **复用还是新建？** 环境可跨代理共享——先检查是否已有可用环境。
- **Networking** - default unrestricted egress. Switch to `limited` only if the user wants egress control - then set `allow_mcp_servers: true` or list every MCP server domain in `allowed_hosts`, or those tools fail silently.
  **网络（Networking）**——默认完全出站。仅当用户想要出站控制时切换到 `limited`——此时需设置 `allow_mcp_servers: true` 或把每个 MCP 服务器域名列入 `allowed_hosts`，否则这些工具会静默失败。
- **Suggest `self_hosted`** when the signals are there: tools must run on their own infra, secrets can't leave it, or they need binaries/data the cloud container won't have (`shared/managed-agents-self-hosted-sandboxes.md`; on Claude Platform on AWS the worker authenticates with IAM instead of an environment key and sessions there can't attach memory stores). Otherwise `cloud` - don't raise it unprompted for simple jobs.
  当出现以下信号时**建议 `self_hosted`**：工具必须运行在用户自己的基础设施上、机密不能离开该设施、或需要云端容器不具备的二进制文件/数据（`shared/managed-agents-self-hosted-sandboxes.md`；在 Claude Platform on AWS 上，worker 用 IAM 而非环境密钥进行身份验证，且那里的会话无法挂载记忆存储）。否则用 `cloud`——对简单任务不要主动提起自托管。

## 4. Session - auth, then test run / 会话——先鉴权，再试运行

**Auth happens here - collect the credentials flagged in §2, now that the config is settled:** a vault (existing or `vaults.create()`) + `vaults.credentials.create()` for each MCP server declared in §2, `environment_variable` credentials for API keys the job uses (substituted at egress; the sandbox sees a placeholder), and the `authorization_token` for each repo mount. Credentials are write-only; MCP credentials match servers by URL and auto-refresh. See `shared/managed-agents-tools.md` -> Vaults.

**鉴权发生在这一步——配置敲定之后，收集 §2 中标记过的凭据：** 一个 vault（已有的，或经 `vaults.create()` 创建），加上为 §2 中声明的每个 MCP 服务器执行 `vaults.credentials.create()`；任务用到的 API key 使用 `environment_variable` 凭据（在出站时替换；沙箱只能看到占位符）；每个仓库挂载提供对应的 `authorization_token`。凭据均为只写；MCP 凭据按 URL 与服务器匹配并自动刷新。参见 `shared/managed-agents-tools.md` -> Vaults。

**Silent viability gate - run this yourself before emitting anything; surface only the gaps.** Walk the job clause by clause: every verb maps to an enabled tool or MCP server ("open a PR" -> GitHub MCP, not just the mount); every MCP server and repo mount has its credential from the auth step; every external host is reachable under the networking choice; every file/repo/dataset the job references is mounted; "done" is checkable. If something's missing, say so and resolve it - don't emit a config you already know is under-resourced.

**静默可行性闸门——在输出任何内容之前先自行执行这项检查；只呈现缺口。** 逐条走查任务：每个动词都对应一个已启用的工具或 MCP 服务器（"发一个 PR" -> GitHub MCP，而不只是仓库挂载）；每个 MCP 服务器和仓库挂载都持有鉴权步骤提供的凭据；每个外部主机在所选网络策略下可达；任务引用的每个文件/仓库/数据集都已挂载；"完成"是可检验的。若有缺失，直说并解决它——不要输出一个你明知资源不足的配置。

【评论】该条款要求模型在生成配置前先自检可行性、只向用户呈现缺口，属于把验证责任前置到生成阶段的防错设计。

**Kickoff - pick one, never both. Outcome is the default:**

**启动（Kickoff）——二选一，绝不同时使用。默认用 Outcome：**

- `user.define_outcome` + rubric - the default whenever the job has a deliverable (§2 drafts the rubric); the harness iterates and grades until the rubric passes.
  `user.define_outcome` + 评分标准——只要任务有交付物即为默认（§2 已起草评分标准）；执行框架据此迭代并评分，直到标准通过。
- `user.message` - only for genuinely conversational sessions.
  `user.message`——仅用于真正的对话式会话。
- **Scheduled shape?** Skip per-session kickoff entirely - create a **deployment** (`deployments.create()` with `schedule` + `initial_events`); each firing creates the session autonomously. See `shared/managed-agents-scheduled-deployments.md`.
  **定时形态？** 完全跳过逐会话启动——创建一个**部署（deployment）**（`deployments.create()`，带 `schedule` + `initial_events`）；每次触发都会自主创建会话。参见 `shared/managed-agents-scheduled-deployments.md`。

Mechanics to bake into the runtime code: session creation resolves resources (a bad mount surfaces there, before tokens) but does not itself provision the sandbox; open the event stream *before* sending the kickoff; break on `session.status_terminated`, or `session.status_idle` with any non-`requires_action` `stop_reason` - terminal, or `budget_reached`, which is not terminal (only a budget change/removal resumes it) (`shared/managed-agents-client-patterns.md` Pattern 5); usage lands on `span.model_request_end`; artifacts land in `/mnt/session/outputs/` (`files.list({scope_id: session.id, ...})`).

需要固化到运行时代码里的机制：会话创建会解析资源（错误的挂载在这一步、消耗 token 之前即暴露），但其本身不会预配沙箱；要在发送启动事件*之前*打开事件流；在 `session.status_terminated`，或 `session.status_idle` 且 `stop_reason` 非 `requires_action` 时跳出循环——终态，或 `budget_reached`（非终态，只有预算变更/移除才能恢复）（`shared/managed-agents-client-patterns.md` Pattern 5）；用量记录在 `span.model_request_end`；产物落在 `/mnt/session/outputs/`（`files.list({scope_id: session.id, ...})`）。

## 5. Integrate - emit the code / 集成——输出代码

Go straight from the last answer to the code - no preamble, no lecture about setup-vs-runtime; the two-block structure shows it. Generate **two clearly-separated blocks**:

从最后一个回答直接进入代码——不要开场白，也不要长篇讲解 setup 与 runtime 的区别；两段式结构本身就能说明。生成**两个明确分隔的代码块**：

**Block 1 - Setup (files + `ant apply`; the IDs land in `claude-lock.json`).** Agents and environments are version-controlled definitions - write them as files and sync them with `ant apply` (`shared/anthropic-cli.md` -> Version-controlled Managed Agents resources):

**块 1 —— 设置（Setup：文件 + `ant apply`；ID 落入 `claude-lock.json`）。** 代理和环境都是受版本控制的定义——把它们写成文件，并用 `ant apply` 同步（`shared/anthropic-cli.md` -> Version-controlled Managed Agents resources）：

1. `agents/<name>.md` - YAML frontmatter (`name`, `model`, `tools`, `mcp_servers`, `skills`) with the system prompt as the Markdown body - and `environments/<name>.yaml`. Reusing an existing environment (§3)? Write no environment file (it would create a second one), leave it out of the commands below, and use the existing `env_...` ID wherever an environment is named (Block 2, a deployment file's `environment_id`).
   `agents/<name>.md` —— YAML frontmatter（`name`、`model`、`tools`、`mcp_servers`、`skills`），系统提示词作为 Markdown 正文——外加 `environments/<name>.yaml`。如果要复用已有环境（§3）：那就不要写环境文件（否则会创建出第二个环境），别把它放进下面的命令，并在任何需要指名环境的地方（块 2、部署文件的 `environment_id`）改用已有的 `env_...` ID。
2. ```sh
   ant apply --dry-run -v agents/<name>.md environments/<name>.yaml   # prints the full plan, every field; changes nothing
   ant apply agents/<name>.md environments/<name>.yaml                # asks, then creates; run it again after any edit to update
   ```
   Name the files you just wrote - never `.` or a directory, which is walked and also creates whatever else in the repo looks like a resource (a Claude Code plugin's `agents/*.md` and `skills/*/SKILL.md`, files the user never read). Without a terminal (a coding agent's shell) the second command prints the plan and exits; it applies only with `--yes`, which is the user's approval, not yours: show them the dry-run plan and add it only once they say go ahead. If the plan would create or change anything you did not write, or a file you did not write sits at a path you need, stop and ask; never add `--force` or `--prune` on your own.
   指明你刚写的那几个文件——绝不要用 `.` 或整个目录，那会被递归遍历，还会把仓库里其他长得像资源的东西一并创建出来（比如某个 Claude Code 插件的 `agents/*.md` 和 `skills/*/SKILL.md`——用户从未读过这些文件）。在没有终端的环境（编码代理的 shell）下，第二条命令只打印计划然后退出；只有加 `--yes` 才会真正应用，而 `--yes` 代表的是用户的批准，不是你的：先把 dry-run 计划展示给用户，等他们说继续之后才加上它。如果计划会创建或更改任何不是你写的文件，或一个不是你写的文件恰好占据你需要的路径，停下来询问；绝不擅自添加 `--force` 或 `--prune`。
   
   【评论】"`--yes` 是用户的批准，不是你的"这一条款把破坏性操作的人工确认权明确保留给人类用户，属于对编码代理自主权限的典型约束。
3. Keep `claude-lock.json` beside the files (commit both if this is a repo) - it holds the IDs, and without it the next `ant apply` creates duplicates. Copy the IDs Block 2 needs (agent, environment; scheduled shape: the deployment) into the app's own config or env vars once - `resources["./agents/<name>.md"].id` and so on, keyed by the path the plan printed - so the running app does not depend on the lockfile.
   把 `claude-lock.json` 和这些文件放在一起（如果是仓库就一并提交）——它保存着 ID，缺了它下一次 `ant apply` 会创建重复资源。把块 2 需要的 ID（代理、环境；定时形态还有部署）一次性复制进应用自己的配置或环境变量——`resources["./agents/<name>.md"].id` 等，以计划打印的路径为键——这样运行中的应用就不依赖锁文件。

If `ant` is missing or older than 1.30.0 (`ant --version`) - and the user is not on Claude Platform on AWS (below) - say so and offer to install or upgrade it (`shared/anthropic-cli.md` -> Install and auth). Ask before running an installer, and do not silently fall back to the SDK; use the SDK fallback below only if the user declines or it cannot be installed.

如果 `ant` 缺失或版本低于 1.30.0（`ant --version`）——且用户不在 Claude Platform on AWS 上（见下）——如实说明并提议安装或升级（`shared/anthropic-cli.md` -> Install and auth）。运行安装程序前要先询问，不要静默退回 SDK；只有当用户拒绝安装或无法安装时，才使用下方的 SDK 兜底方案。

SDK fallback if the user asks - and **required on Claude Platform on AWS**, where auth is SigV4 and the `ant` CLI has no SigV4 mode (use the platform client from `shared/claude-platform-on-aws.md`): label it `# ONE-TIME SETUP - run once, save the IDs` and call `environments.create()` -> `agents.create()`.

SDK 兜底方案：在用户要求时提供——并且在 Claude Platform on AWS 上是**必选项**（那里的鉴权是 SigV4，而 `ant` CLI 没有 SigV4 模式；使用 `shared/claude-platform-on-aws.md` 中的平台客户端）：加注 `# ONE-TIME SETUP - run once, save the IDs` 标签，并调用 `environments.create()` -> `agents.create()`。

> Warning: **Deployments are newer than the rest of the MA surface.** Before emitting `ant beta:deployments ...` or `client.beta.deployments` / `client.beta.deployment_runs` calls, verify the user's installed CLI/SDK exposes them (`ant beta:deployments --help`; `hasattr(client.beta, "deployments")`). If not, emit raw HTTP against `POST /v1/deployments` with the `managed-agents-2026-04-01` beta header (plus `oauth-2025-04-20` when authenticating with a Bearer token from `ant auth print-credentials`), and leave an upgrade note marking what simplifies to SDK calls.

> 警告：**Deployments（部署）比 MA（Managed Agents）其余部分更新。** 在生成 `ant beta:deployments ...` 或 `client.beta.deployments` / `client.beta.deployment_runs` 调用之前，先验证用户安装的 CLI/SDK 是否支持（`ant beta:deployments --help`；`hasattr(client.beta, "deployments")`）。若不支持，则生成针对 `POST /v1/deployments` 的原始 HTTP 调用，带上 `managed-agents-2026-04-01` beta 请求头（若使用来自 `ant auth print-credentials` 的 Bearer token 鉴权，还需加 `oauth-2025-04-20`），并留下一条升级说明，标注哪些部分日后可简化为 SDK 调用。

**Scheduled shape? The deployment is setup, not runtime.** Create it in Block 1. With `ant apply`: write `deployments/<name>.md` and add it to the same `ant apply` command. It names the agent and environment by path (a reused environment by its `env_...` ID); the frontmatter is `schedule` plus the rest of the create body, and the Markdown body becomes a `user.message` kickoff (for an Outcome kickoff put `initial_events` in the frontmatter and leave the body empty, not both). With the SDK: `deployments.create()` with `schedule` + `initial_events` after the agent/environment IDs exist. Block 2 is then **not** a session loop - there is no per-run kickoff to send. Emit instead: a manual-run trigger (`POST /v1/deployments/{id}/run`) so the user can test now rather than wait for the first firing - the manual run doubles as the smoke test - plus a fetch helper (latest `deployment_runs` entry -> `session_id` -> Console URL + `files.list(scope_id=session_id)` for the artifacts).

**定时形态？部署属于设置（setup），而非运行时（runtime）。** 在块 1 中创建。使用 `ant apply` 时：写一个 `deployments/<name>.md`，并把它加进同一条 `ant apply` 命令。它按路径指名代理和环境（复用的环境用其 `env_...` ID）；frontmatter 是 `schedule` 加上创建请求体的其余部分，Markdown 正文则成为 `user.message` 启动事件（若用 Outcome 启动，则把 `initial_events` 放进 frontmatter 并把正文留空，两者不可兼得）。使用 SDK 时：在代理/环境 ID 存在之后，调用带 `schedule` + `initial_events` 的 `deployments.create()`。此时块 2 就**不是**会话循环——没有需要逐次发送的启动事件。应改为输出：一个手动运行触发器（`POST /v1/deployments/{id}/run`），让用户现在就能测试而不必等首次定时触发——手动运行可兼作冒烟测试——外加一个结果获取辅助函数（取最新的 `deployment_runs` 条目 -> `session_id` -> Console URL + 用 `files.list(scope_id=session_id)` 取产物）。

**Block 2 - Runtime (every invocation; conversational and Outcome shapes).** SDK code in the detected language (Python/TS/cURL - SKILL.md -> Language Detection); don't emit shell loops here:

**块 2 —— 运行时（Runtime：每次调用；对话与 Outcome 两种形态）。** 用检测到的语言（Python/TS/cURL——见 SKILL.md -> Language Detection）生成 SDK 代码；不要在这里输出 shell 循环：

1. Load `agent_id` + `env_id` from config/env (where Block 1 put them)
   从配置/环境变量中加载 `agent_id` + `env_id`（即块 1 存放它们的位置）
2. `sessions.create(agent=AGENT_ID, environment_id=ENV_ID, resources=[...], vault_ids=[...])`, then print the Console URL so the user can watch live: `https://platform.claude.com/workspaces/default/sessions/{session.id}` (swap `default` for their workspace slug)
   调用 `sessions.create(agent=AGENT_ID, environment_id=ENV_ID, resources=[...], vault_ids=[...])`，然后打印 Console URL 供用户实时观看：`https://platform.claude.com/workspaces/default/sessions/{session.id}`（把 `default` 换成其 workspace slug）
3. **Smoke-test when the job depends on MCP servers, credentials, or locked-down hosts** - those failures don't surface at `sessions.create()`, only on first use. One cheap probe turn ("Confirm you can reach `<service>` and list 1-2 items; don't start the task"), verify, then send the real kickoff. Skip when there are no external dependencies.
   **当任务依赖 MCP 服务器、凭据或受限主机时做冒烟测试**——这些故障不会在 `sessions.create()` 时暴露，只在首次使用时出现。先来一轮低成本的探测（"确认你能访问 `<service>` 并列出 1-2 个条目；先别开始任务"），验证通过后再发送真正的启动事件。没有外部依赖时跳过。
4. Open stream -> send the §4 kickoff -> loop with the terminal gate from §4.
   打开流 -> 发送 §4 的启动事件 -> 按 §4 的终态闸门循环。

> Warning: **Never emit `agents.create()` and `sessions.create()` in the same unguarded block** - that teaches creating a new agent per run, the #1 anti-pattern. Single-script requests: wrap creation in `if not os.getenv("AGENT_ID"):`.

> 警告：**绝不要把 `agents.create()` 和 `sessions.create()` 放在同一个无保护的代码块里**——那是在教用户每次运行都新建一个代理，这是头号反模式。用户要求单脚本时：把创建逻辑包在 `if not os.getenv("AGENT_ID"):` 里。

【评论】该警告针对自动化生成代码场景中最常见的资源滥用反模式（每次运行都新建代理），并给出了条件保护的具体写法。

Pull exact syntax from `{lang}/managed-agents/README.md` for your detected language (cURL and C#: use `curl/managed-agents.md` as the wire-level reference). Don't invent field names.

按检测到的语言从 `{lang}/managed-agents/README.md` 获取准确的语法（cURL 和 C#：以 `curl/managed-agents.md` 作为线上协议级参考）。不要凭空编造字段名。
