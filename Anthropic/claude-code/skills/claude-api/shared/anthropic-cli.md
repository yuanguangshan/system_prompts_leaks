<!-- BILINGUAL-EN-ZH -->

# Anthropic CLI (`ant`) / Anthropic CLI（`ant`）

The `ant` CLI exposes every Claude API resource as a shell subcommand. Compared to `curl`: request bodies are built from typed flags or piped YAML instead of hand-written JSON, `@path` inlines file contents into any string field, `--transform` extracts fields with a GJSON path (no `jq`), list endpoints auto-paginate (cap total results with `--max-items N`; `--limit` only sets the server page size), and the `beta:` prefix auto-sets the right `anthropic-beta` header.

`ant` CLI 把每个 Claude API 资源都暴露为 shell 子命令。与 `curl` 相比：请求体由带类型的旗标或管道输入的 YAML 构建，而非手写 JSON；`@path` 可把文件内容内联进任何字符串字段；`--transform` 用 GJSON 路径提取字段（无需 `jq`）；列表端点自动分页（用 `--max-items N` 限制总结果数；`--limit` 只设置服务器页大小）；`beta:` 前缀自动设置正确的 `anthropic-beta` 请求头。

## When to use the CLI vs the SDK / 何时用 CLI 而非 SDK

**CLI for the control plane, SDK for the data plane.** Agents and environments are relatively static resources you define, configure, and debug with `ant` - keep them as files in your repo, sync them with `ant apply` (by hand or from CI), inspect from a terminal. Sessions are dynamic and driven by your application through the SDK - create per task, stream events, react to tool calls, integrate into your product. Both hit the same API; the split is about where the call lives, not what's possible.

**CLI 用于控制面，SDK 用于数据面。** 代理（agents）和环境（environments）是相对静态的资源，用 `ant` 定义、配置和调试 — 以文件形式保存在仓库中，用 `ant apply` 同步（手动或从 CI），在终端里查看。会话（sessions）是动态的，由你的应用通过 SDK 驱动 — 按任务创建、流式接收事件、响应工具调用、集成进产品。两者调用同一个 API；区分在于调用发生在哪里，而不在于能做到什么。

【评论】"控制面/数据面"的划分把基础设施即代码（IaC）的理念套用到代理资源管理上：静态声明入仓库版本控制，动态运行时交互走 SDK。

| | Control plane -> `ant` | Data plane -> SDK |
|---|---|---|
| Resources | agents, environments, skills, vaults, files | sessions, events |
| Cadence | Once per deploy / ad-hoc | Every task / every turn |
| Lives in | `agents/`, `environments/`, `claude-lock.json` in your repo + CI + terminal | Application code |
| Typical calls | `ant apply`, `list`, `retrieve`, `archive`, `--debug` | `sessions.create()`, `events.stream()`, `events.send()` |

| | 控制面 -> `ant` | 数据面 -> SDK |
|---|---|---|
| 资源 | agents、environments、skills、vaults、files | sessions、events |
| 节奏 | 每次部署一次 / 临时操作 | 每个任务 / 每一轮 |
| 存放于 | 仓库中的 `agents/`、`environments/`、`claude-lock.json` + CI + 终端 | 应用代码 |
| 典型调用 | `ant apply`、`list`、`retrieve`、`archive`、`--debug` | `sessions.create()`、`events.stream()`、`events.send()` |

## Install and auth / 安装与认证

```sh
# macOS
brew install anthropics/tap/ant
xattr -d com.apple.quarantine "$(brew --prefix)/bin/ant"

# Linux / WSL - pick the release from github.com/anthropics/anthropic-cli/releases
curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_$(uname -s | tr A-Z a-z)_$(uname -m | sed -e s/x86_64/amd64/ -e s/aarch64/arm64/).tar.gz" \
  | sudo tar -xz -C /usr/local/bin ant

# Or from source (Go 1.25+)
go install github.com/anthropics/anthropic-cli/cmd/ant@latest
```

**Auth** - the CLI resolves credentials the same way the SDKs do (first match wins): explicit flags, then `ANTHROPIC_API_KEY`, then `ANTHROPIC_AUTH_TOKEN`, then the `ANTHROPIC_PROFILE`-selected or active profile, then Workload Identity Federation env vars, then the default profile on disk. Override the host with `ANTHROPIC_BASE_URL` or `--base-url`.

**认证** - CLI 解析凭据的方式与 SDK 相同（取第一个匹配者）：显式旗标，然后是 `ANTHROPIC_API_KEY`，然后是 `ANTHROPIC_AUTH_TOKEN`，然后是 `ANTHROPIC_PROFILE` 选定的或处于活动状态的 profile，然后是 Workload Identity Federation 环境变量，最后是磁盘上的默认 profile。用 `ANTHROPIC_BASE_URL` 或 `--base-url` 覆盖主机地址。

- **API key**: set `ANTHROPIC_API_KEY` in the environment.
  **API 密钥**：在环境中设置 `ANTHROPIC_API_KEY`。
- **OAuth profile** (no static key to manage): `ant auth login` opens a browser, exchanges for a short-lived token, and stores a profile under `$ANTHROPIC_CONFIG_DIR` (default `~/.config/anthropic/` on Linux/macOS, `%APPDATA%\Anthropic` on Windows - `configs/<profile>.json` for settings, `credentials/<profile>.json` for tokens). Subsequent `ant` (and SDK) calls pick it up automatically - a bare `Anthropic()` client works after login, but scripts that read `ANTHROPIC_API_KEY` directly do not. Claude Code and the Claude Agent SDK honor the same profile resolution. `ant auth status` shows which credential source and profile won (it reports status only - don't script against its exit code as a health check); `ant auth logout` clears the active profile (`--all` for every profile). On a remote host without a browser, `ant auth login --no-browser` prints the authorize URL and accepts the code back in the terminal.
  **OAuth profile**（无需管理静态密钥）：`ant auth login` 会打开浏览器、兑换短时效令牌，并在 `$ANTHROPIC_CONFIG_DIR`（Linux/macOS 下默认 `~/.config/anthropic/`，Windows 下为 `%APPDATA%\Anthropic` — `configs/<profile>.json` 存设置，`credentials/<profile>.json` 存令牌）下保存一个 profile。后续的 `ant`（和 SDK）调用会自动采用它 — 登录后裸的 `Anthropic()` 客户端即可工作，但直接读取 `ANTHROPIC_API_KEY` 的脚本不行。Claude Code 与 Claude Agent SDK 遵循相同的 profile 解析。`ant auth status` 显示哪个凭据来源和 profile 胜出（它只报告状态 — 不要把它的退出码当健康检查写进脚本）；`ant auth logout` 清除活动 profile（`--all` 清除所有 profile）。在没有浏览器的远程主机上，`ant auth login --no-browser` 会打印授权 URL 并在终端接收授权码。
- **Non-interactive workloads** (CI, servers, containers): interactive login is for development on your own machine - use Workload Identity Federation instead (see the authentication docs via `shared/live-sources.md`).
  **非交互式工作负载**（CI、服务器、容器）：交互式登录只适用于在你自己的机器上开发 — 请改用 Workload Identity Federation（认证文档经 `shared/live-sources.md` 获取）。

> **The #1 auth trap:** profiles are only consulted when no API key is set. A stale exported `ANTHROPIC_API_KEY` silently overrides every profile - requests hit whatever org/workspace that key is scoped to. `ant auth status` shows which source won; unset the key (or per-command: `env -u ANTHROPIC_API_KEY ant ...`) before relying on a profile. Truly **unset** it - an empty `ANTHROPIC_API_KEY=""` still wins its precedence slot and authenticates with an empty key. The same shadowing applies in reverse to Claude Code: after `ant auth login`, Claude Code may warn about an auth conflict between the profile and its own `/login` credential - keep one (use the profile and `/logout` in Claude Code, or `ant auth logout` to keep Claude Code's own login).

> **头号认证陷阱：** 只有在未设置 API 密钥时才会查询 profile。一个残留的已导出 `ANTHROPIC_API_KEY` 会静默覆盖所有 profile — 请求会打到该密钥所属的 org/workspace。`ant auth status` 显示哪个来源胜出；在依赖 profile 之前先取消该密钥（或按命令：`env -u ANTHROPIC_API_KEY ant ...`）。要真正**取消**它 — 空的 `ANTHROPIC_API_KEY=""` 仍会占据其优先级位置并以空密钥认证。同样的遮蔽关系反向也适用于 Claude Code：`ant auth login` 之后，Claude Code 可能警告 profile 与其自身 `/login` 凭据存在认证冲突 — 二者保留其一（用 profile 并在 Claude Code 中 `/logout`，或 `ant auth logout` 以保留 Claude Code 自己的登录）。

【评论】凭据解析按优先级链短路，这类"环境变量残留静默改变目标租户"的陷阱在多组织工作区场景中很典型，文档以最高优先级警示加以强调。

**Named profiles** - an interactive-login token is bound to a single org+workspace, and the API only shows resources belonging to that workspace. If an agent, session, or file you created "disappears", the usual cause is a token scoped to a different workspace than the one that created it (`ant auth status` shows the active workspace). Multi-workspace work means one profile per workspace:

**命名 profile** - 交互式登录令牌绑定到单一 org+workspace，API 只显示属于该 workspace 的资源。如果你创建的代理、会话或文件"消失"了，常见原因是令牌所属的 workspace 与创建它的 workspace 不同（`ant auth status` 显示活动 workspace）。多 workspace 工作意味着每个 workspace 一个 profile：

```sh
ant auth login --profile <name>                  # creates the profile if it doesn't exist; org/workspace picker in browser
ant auth login --profile <name> --workspace-id wrkspc_01...   # bind directly, skip the picker
ant profile activate <name>                      # switch the default profile
ant --profile <name> models list                 # one-off; equivalent: ANTHROPIC_PROFILE=<name> ant models list
ant profile list                                 # inspect
ant profile set workspace_id wrkspc_01... --profile <name>    # edit config keys (workspace_id, base_url, organization_id, ...)
```

`ant profile set` edits an existing profile's config - it never creates one, and it does **not** rebind already-issued credentials; run `ant auth login` again under that profile to mint a token for the new target. Pointing `ANTHROPIC_PROFILE` at a profile that doesn't exist is an error, not a fall-through. Refresh tokens eventually hard-expire (they don't slide with use) - when a previously working profile starts failing auth, re-run `ant auth login` before debugging anything else.

`ant profile set` 编辑既有 profile 的配置 — 它从不创建 profile，也**不会**重新绑定已签发的凭据；请在该 profile 下重新运行 `ant auth login` 来为新目标铸造令牌。把 `ANTHROPIC_PROFILE` 指向不存在的 profile 是错误，而非向下兜底。刷新令牌最终会硬过期（不会随使用而顺延）— 当一个之前正常的 profile 开始认证失败时，先重新运行 `ant auth login`，再调试其他任何东西。

**Scopes** - a profile's OAuth scope set is requested at login (`--scope`) and persists on the profile (`scope` is also a `profile set` config key; like other config edits, changing it requires a fresh `ant auth login` to take effect). Privileged scopes - e.g. `org:admin` for organization-administration endpoints - are **not** in the default scope set: pass the full set you want explicitly (`ant auth login --profile admin --scope "... org:admin"`), and the server grants a privileged scope only if your role actually has it. Because the scope set rides on every token the profile mints, keep privileged work on a dedicated profile (`admin` vs `default`) and do day-to-day inference on the unprivileged one, switching with `--profile`/`ANTHROPIC_PROFILE`. Check `ant auth login --help` for the current scope list, and `ant auth status` to see what the active token carries.

**权限范围（Scopes）** - profile 的 OAuth 范围集在登录时申请（`--scope`）并持久保存在 profile 上（`scope` 也是 `profile set` 的配置键；与其他配置修改一样，修改它需要重新 `ant auth login` 才能生效）。特权范围 — 例如用于组织管理端点的 `org:admin` — **不在**默认范围集中：请显式传入你想要的完整范围集（`ant auth login --profile admin --scope "... org:admin"`），且只有当你的角色确实拥有该特权范围时服务器才会授予。由于范围集附着在 profile 铸造的每一枚令牌上，应把特权工作放在专用 profile（`admin` 与 `default` 分开），日常推理用非特权 profile，通过 `--profile`/`ANTHROPIC_PROFILE` 切换。用 `ant auth login --help` 查看当前范围列表，用 `ant auth status` 查看活动令牌携带的范围。

To hand the active credential to a subprocess or raw-HTTP script:

要把活动凭据交给子进程或原始 HTTP 脚本：

```sh
# Bare access token - for curl's Authorization header
curl https://api.anthropic.com/v1/messages \
  -H "Authorization: Bearer $(ant auth print-credentials --access-token)" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: oauth-2025-04-20" \
  -H "content-type: application/json" \
  -d '{"model": "claude-opus-5-5", "max_tokens": 1024, "messages": [{"role": "user", "content": "Hello"}]}'

# .env format - sets ANTHROPIC_AUTH_TOKEN (and ANTHROPIC_BASE_URL if the profile has one).
# Output is bare KEY=value (no `export`), so use `set -a` to auto-export for child processes:
set -a; eval "$(ant auth print-credentials --env)"; set +a
python my_script.py   # SDK picks up ANTHROPIC_AUTH_TOKEN
```

OAuth tokens go on `Authorization: Bearer` (not `x-api-key:`) **plus the `anthropic-beta: oauth-2025-04-20` header** - converting a raw curl/httpx script from an API key is a header change, not a key swap. The beta header requirement is endpoint-dependent (some endpoints happen to work without it; `/v1/messages` does not) - always send it so requests don't break when you switch endpoints. The token is short-lived and not auto-refreshed when passed via env var, so re-run `print-credentials` before it expires for long-running scripts (`print-credentials` itself refreshes the token if needed). If both `ANTHROPIC_API_KEY` and `ANTHROPIC_AUTH_TOKEN` are set, the SDKs send both and the API rejects the request - unset `ANTHROPIC_API_KEY` before `eval`ing the `--env` output.

OAuth 令牌放在 `Authorization: Bearer`（而非 `x-api-key:`）上，**还要加 `anthropic-beta: oauth-2025-04-20` 请求头** — 把原始 curl/httpx 脚本从 API 密钥迁移过来是改请求头，不是换密钥。beta 请求头的要求取决于端点（有些端点恰好没有它也能工作；`/v1/messages` 不行）— 请始终发送它，以免切换端点时请求出问题。令牌短时效，且通过环境变量传入时不会自动刷新，因此长时间运行的脚本应在过期前重新运行 `print-credentials`（`print-credentials` 本身会在需要时刷新令牌）。如果 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 同时设置，SDK 会两个都发送而 API 会拒绝请求 — 在 `eval` `--env` 输出之前先取消 `ANTHROPIC_API_KEY`。

**Foot-gun:** `ant auth print-credentials` with **no flags** prints the entire credentials JSON, not the bare token - putting that in an `Authorization` header yields an empty response or HTTP/2 protocol error. Always use `--access-token` for headers (it always reads the named/active profile; a set `ANTHROPIC_API_KEY` doesn't override credential printing).

**易踩的坑：** 不带任何旗标的 `ant auth print-credentials` 打印的是完整凭据 JSON，而非裸令牌 — 把它放进 `Authorization` 请求头会得到空响应或 HTTP/2 协议错误。构造请求头时务必用 `--access-token`（它始终读取命名/活动 profile；已设置的 `ANTHROPIC_API_KEY` 不会覆盖凭据打印）。

## Command structure / 命令结构

```
ant <resource>[:<subresource>] <action> [flags]
```

Beta resources (agents, sessions, environments, deployments, skills, vaults, memory stores) live under `beta:` - the CLI auto-sends the right `anthropic-beta` header, so don't pass it yourself unless overriding with `--beta <header>`. For self-hosted environments, `ant beta:worker poll/run` and `ant beta:environments:work stats/stop` drive and monitor the work queue - see `shared/managed-agents-self-hosted-sandboxes.md`.

Beta 资源（agents、sessions、environments、deployments、skills、vaults、memory stores）位于 `beta:` 之下 — CLI 会自动发送正确的 `anthropic-beta` 请求头，除非要用 `--beta <header>` 覆盖，否则不要自己传。对自托管环境，`ant beta:worker poll/run` 和 `ant beta:environments:work stats/stop` 用于驱动和监控工作队列 — 见 `shared/managed-agents-self-hosted-sandboxes.md`。

```sh
ant models list
ant messages create --model claude-opus-5-5 --max-tokens 1024 --message '{role: user, content: "Hello"}'
ant beta:agents retrieve --agent-id agent_01...
ant beta:sessions:events list --session-id session_01...
```

`ant --help` lists resources; append `--help` to any subcommand for its flags.

`ant --help` 列出资源；在任何子命令后加 `--help` 查看其旗标。

## Global flags / 全局旗标

| Flag | Purpose |
| --- | --- |
| `--format` | `auto` (default: pretty if TTY, compact if piped), `json`, `jsonl`, `yaml`, `pretty`, `raw`, `explore` (interactive TUI) |
| `--transform` | GJSON path applied to the response (per-item on list endpoints). Not applied when `--format raw`. |
| `-r`, `--raw-output` | If the transformed result is a string, print it without quotes (jq semantics). Pair with `--transform` for scalar capture. |
| `--max-items` | Cap total results returned from auto-paginating list endpoints (distinct from `--limit`, which is the server page size). |
| `--format-error` / `--transform-error` | Same as `--format`/`--transform`, applied to error responses. `-r` does not apply to the error path - use `--format-error yaml` for unquoted error scalars. |
| `--base-url` | Override API host |
| `--debug` | Print full HTTP request + response to stderr (API key redacted) |

| 旗标 | 用途 |
| --- | --- |
| `--format` | `auto`（默认：TTY 时 pretty，管道时 compact）、`json`、`jsonl`、`yaml`、`pretty`、`raw`、`explore`（交互式 TUI） |
| `--transform` | 应用于响应的 GJSON 路径（列表端点上按每项应用）。`--format raw` 时不应用。 |
| `-r`, `--raw-output` | 若转换结果是字符串，去掉引号打印（jq 语义）。与 `--transform` 搭配用于捕获标量。 |
| `--max-items` | 限制自动分页列表端点返回的总结果数（区别于 `--limit`，后者是服务器页大小）。 |
| `--format-error` / `--transform-error` | 与 `--format`/`--transform` 相同，但应用于错误响应。`-r` 不作用于错误路径 — 错误标量要去引号请用 `--format-error yaml`。 |
| `--base-url` | 覆盖 API 主机 |
| `--debug` | 向 stderr 打印完整 HTTP 请求 + 响应（API 密钥已脱敏） |

## Output - `--transform` + `--format` / 输出 - `--transform` + `--format`

`--transform` takes a [GJSON path](https://github.com/tidwall/gjson/blob/master/SYNTAX.md). On list endpoints it runs **per item**, not on the envelope.

`--transform` 接受一个 [GJSON 路径](https://github.com/tidwall/gjson/blob/master/SYNTAX.md)。在列表端点上它**按每项**运行，而非作用于整个信封。

```sh
ant beta:agents list --transform '{id,name,model}' --format jsonl
```

**Extract a scalar for shell use:** pair `--transform` with `-r` (`--raw-output` - prints strings unquoted, jq-style):

**提取标量供 shell 使用：** 把 `--transform` 与 `-r`（`--raw-output` — 打印字符串时去掉引号，jq 风格）搭配：

```sh
AGENT_ID=$(ant beta:agents create --name "My Agent" --model '{id: claude-sonnet-5-5}' \
  --transform id -r)
```

## Input - flags, stdin, `@file` / 输入 - 旗标、stdin、`@file`

**Flags** - scalar fields map directly. Structured fields accept relaxed-YAML syntax (unquoted keys) or strict JSON. Repeatable flags build arrays (each `--tool`, `--event`, `--message` appends one element):

**旗标（Flags）** - 标量字段直接映射。结构化字段接受宽松 YAML 语法（键可不加引号）或严格 JSON。可重复旗标构建数组（每个 `--tool`、`--event`、`--message` 追加一个元素）：

```sh
ant beta:agents create \
  --name "Research Agent" \
  --model '{id: claude-opus-5-5}' \
  --tool '{type: agent_toolset_20260401}' \
  --tool '{type: custom, name: search_docs, input_schema: {type: object, properties: {query: {type: string}}}}'
```

**Stdin** - pipe a full JSON or YAML body. Merged with flags; flags win on conflict (for array fields, any flag **replaces** the stdin array entirely - it does not append). Quote the heredoc delimiter (`<<'YAML'`) to disable shell expansion inside the body:

**Stdin** - 通过管道传入完整 JSON 或 YAML 请求体。与旗标合并；冲突时旗标优先（对数组字段，任何旗标都会**整体替换** stdin 数组 — 而非追加）。给 heredoc 定界符加引号（`<<'YAML'`）以禁用请求体内的 shell 展开：

```sh
ant beta:agents create <<'YAML'
name: Research Agent
model: claude-opus-5-5
system: |
  You are a research assistant. Cite sources for every claim.
tools:
  - type: agent_toolset_20260401
YAML
```

**`@file` references** - inline a file's contents into any string-valued field. Inside structured flag values, quote the path. Binary files are auto-base64'd; force with `@file://` (text) or `@data://` (base64). Escape a literal leading `@` as `\@`.

**`@file` 引用** - 把文件内容内联进任何取字符串值的字段。在结构化旗标值内，请给路径加引号。二进制文件自动 base64；可用 `@file://`（文本）或 `@data://`（base64）强制指定。字面 `@` 开头需转义为 `\@`。

```sh
ant beta:agents create --name "Researcher" --model '{id: claude-sonnet-5-5}' --system @./prompts/researcher.txt

ant messages create --model claude-opus-5-5 --max-tokens 1024 \
  --message '{role: user, content: [
    {type: document, source: {type: base64, media_type: application/pdf, data: "@./scan.pdf"}},
    {type: text, text: "Extract the text from this scanned document."}
  ]}' \
  --transform 'content.0.text' -r
```

Flags that natively take a file path (e.g. `--file` on `beta:files upload`) accept a bare path without `@`.

原生接受文件路径的旗标（如 `beta:files upload` 上的 `--file`）接受不带 `@` 的裸路径。

## Version-controlled Managed Agents resources (`ant apply`) / 版本受控的 Managed Agents 资源（`ant apply`）

This is the recommended flow for defining agents, environments, skills, memory stores and deployments: one file (or skill directory) per resource in your repo, synced with `ant apply` (needs `ant` 1.30.0 or later - check `ant --version`). It prints a plan, creates or updates what differs, and records each resource's ID in `claude-lock.json`. See `shared/managed-agents-core.md` for the field reference, and the `ant apply` page in `shared/live-sources.md` for `--force`, `--prune`, `--lock-file`, renamed or deleted files and CI setup (written for a person at a terminal; the rules below still apply).

这是定义 agents、environments、skills、memory stores 和 deployments 的推荐流程：仓库中每个资源一个文件（或技能目录），用 `ant apply` 同步（需要 `ant` 1.30.0 或更高版本 — 用 `ant --version` 检查）。它会打印计划，创建或更新有差异的部分，并把每个资源的 ID 记录在 `claude-lock.json` 中。字段参考见 `shared/managed-agents-core.md`；`--force`、`--prune`、`--lock-file`、重命名或删除的文件以及 CI 设置见 `shared/live-sources.md` 中的 `ant apply` 页面（该文档面向终端前的人类；以下规则仍然适用）。

```
agents/summarizer.md          # YAML frontmatter = agent config, Markdown body = system prompt
environments/cloud.yaml       # the environment create body
skills/pr-summary/SKILL.md    # a skill is a directory with SKILL.md at its root
memory_stores/notes.yaml
deployments/nightly.md        # frontmatter = deployment create body, Markdown body = the message that starts each run
claude-lock.json              # written by ant apply - commit it
```

```markdown
---
# agents/summarizer.md
name: Summarizer
model: claude-sonnet-5-5
tools:
  - type: agent_toolset_20260401
---

You are a helpful assistant that writes concise summaries.
```

```yaml
# environments/cloud.yaml
name: summarizer-env
config: {type: cloud, networking: {type: unrestricted}}
```

```sh
ant apply --dry-run -v agents/summarizer.md environments/cloud.yaml   # print the plan with every field, change nothing
ant apply agents/summarizer.md environments/cloud.yaml   # print the plan, then ask (y)es / (n)o / (d)etails - needs a terminal
ant apply   # later: reconcile every file claude-lock.json already tracks
```

- **Name the files you wrote; pass `.` or a directory only when the user asks for the whole tree.** A directory is walked to any depth and everything that looks like a resource is applied: any file that has a top-level `type:`, sits directly in `agents/`, `environments/`, `memory_stores/` or `deployments/`, or is named after one of them (`environment_staging.yaml`), plus any directory holding a `SKILL.md`. Claude Code plugins, conda (`environment.yml`) and Kubernetes (`deployments/`) use the same names, and a cloned repo can hold files its user never read.
  **指明你写过的文件；只在用户要求处理整棵目录树时才传 `.` 或目录。** 目录会被递归走查到任意深度，一切看起来像资源的东西都会被应用：任何含顶层 `type:` 的文件、直接位于 `agents/`、`environments/`、`memory_stores/` 或 `deployments/` 中的文件、以上述名字命名的文件（`environment_staging.yaml`），以及任何含 `SKILL.md` 的目录。Claude Code 插件、conda（`environment.yml`）和 Kubernetes（`deployments/`）使用相同的名字，克隆来的仓库可能包含其用户从未读过的文件。

【评论】此条针对"资源声明文件与其他工具的配置文件重名"的误判风险：目录递归加宽松的资源识别规则可能把无关文件一并供应出去，属于防误操作的护栏条款。

- **Without a terminal (a coding agent's shell), `ant apply` prints the plan and exits; it applies only with `--yes`.** If you are a coding agent running this for a user, that flag is their approval, not yours: show them the dry-run plan and add `--yes` (or answer the prompt) only once they say go ahead. The plan also covers whatever `claude-lock.json` already tracks: if it would create or change anything you did not write, or a file you did not write sits at a path you need, stop and ask; never add `--force` or `--prune` on your own.
  **没有终端时（如编码代理的 shell），`ant apply` 打印计划即退出；只有加 `--yes` 才会应用。** 如果你是替用户执行此命令的编码代理，该旗标代表的是用户的批准，而不是你的：先向用户展示 dry-run 计划，只有当用户说继续时才加 `--yes`（或应答提示）。计划还覆盖 `claude-lock.json` 已跟踪的一切：如果它会创建或更改任何非你编写的资源，或一个非你编写的文件位于你需要的位置，停下来询问；绝不要擅自加 `--force` 或 `--prune`。

【评论】"--yes 是用户的批准而非你的"明确禁止代理代行人类授权，并禁止自行使用破坏性更强的 `--force`/`--prune`，是典型的人机权限边界条款。

- **Reference other resources by path, not ID** (relative to the file that names it): `skills: [../skills/pr-summary]` on an agent; `agent: ../agents/summarizer.md` and `environment_id: ../environments/cloud.yaml` on a deployment. `ant apply` also applies whatever the files you pass reference, in dependency order, and fills in the IDs. For a resource these files don't manage, write its ID (`agent_01...`, `env_01...`); anything else is sent as written.
  **用路径而非 ID 引用其他资源**（相对于引用它的文件）：代理上写 `skills: [../skills/pr-summary]`；deployment 上写 `agent: ../agents/summarizer.md` 和 `environment_id: ../environments/cloud.yaml`。`ant apply` 还会按依赖顺序应用你传入文件所引用的资源，并填入 ID。对这些文件不管理的资源，写其 ID（`agent_01...`、`env_01...`）；其余内容按原样发送。
- **Commit `claude-lock.json`** (the first run writes it where you run the command - use the repo root). The next run uses it to update the same resources instead of creating duplicates. A resource created any other way (Console, `ant beta:agents create`, an SDK) cannot be adopted: a file describing it creates a second one.
  **提交 `claude-lock.json`**（首次运行时写在命令执行目录 — 请用仓库根目录）。下一次运行会用它更新同一批资源而不是创建重复项。以其他方式创建的资源（Console、`ant beta:agents create`、SDK）无法被收编：描述它的文件会创建出第二个。
- **To change a resource, edit its file and run `ant apply` again** (an agent gets a new version; whatever references it is updated in the same run).
  **要修改资源，编辑其文件后再次运行 `ant apply`**（代理会获得新版本；引用它的内容会在同一次运行中更新）。
- **CI in the user's own repository:** run from the directory that holds `claude-lock.json` (normally the repo root) and name the resource directories the project has, not `.` (a walk of `.` also applies look-alike files elsewhere in the repo): `ant apply --dry-run agents environments` on pull requests, `ant apply --yes agents environments` only on push to the default branch (there the merge is the approval), then commit `claude-lock.json`.
  **用户自己仓库中的 CI：** 从存放 `claude-lock.json` 的目录（通常是仓库根目录）运行，并写明项目实际拥有的资源目录名，不要用 `.`（对 `.` 的走查也会应用仓库中其他地方的相似文件）：在拉取请求上运行 `ant apply --dry-run agents environments`，仅在推送到默认分支时运行 `ant apply --yes agents environments`（此时合并即批准），然后提交 `claude-lock.json`。
- **Not managed:** vaults and credentials (`ant beta:vaults`, `ant beta:vaults:credentials`, or an SDK), uploaded files, sessions.
  **不受管理：** vault 与凭据（`ant beta:vaults`、`ant beta:vaults:credentials` 或 SDK）、上传的文件、会话。

**One-off provisioning** can still use `ant beta:agents create <<'YAML'` (see Input above) and `ant beta:agents update --agent-id ... --version N`; you keep track of the IDs yourself.

**一次性供应**仍可使用 `ant beta:agents create <<'YAML'`（见上文"输入"一节）和 `ant beta:agents update --agent-id ... --version N`；ID 由你自己跟踪。

Start a session with the IDs from `claude-lock.json` (each `resources` key is the file's path as the plan prints it):

用 `claude-lock.json` 中的 ID 启动会话（每个 `resources` 键就是计划打印的文件路径）：

```sh
AGENT_ID=$(jq -r '.resources["./agents/summarizer.md"].id' claude-lock.json)
ENV_ID=$(jq -r '.resources["./environments/cloud.yaml"].id' claude-lock.json)
SID=$(ant beta:sessions create --agent "$AGENT_ID" --environment-id "$ENV_ID" --title "Task" --transform id -r)
ant beta:sessions:events send --session-id "$SID" \
  --event '{type: user.message, content: [{type: text, text: "Summarize X"}]}'
ant beta:sessions:events list --session-id "$SID" --transform 'content.0.text' -r
ant beta:sessions:events stream --session-id "$SID"   # live event stream
```

### Attach a terminal to a session (`ant beta:sessions connect`) / 将终端接入会话（`ant beta:sessions connect`）

`ant beta:sessions connect <session-id>` attaches your terminal to an existing session: it loads the transcript, follows it live, and lets you step in - send a message, interrupt, or allow/deny a tool call that is waiting for approval. Ctrl+C detaches; the session keeps running, and reconnecting reloads the full history. Read-only if the session is `terminated` or archived.

`ant beta:sessions connect <session-id>` 把你的终端接入一个既有会话：它会加载对话记录、实时跟随，并允许你介入 — 发送消息、打断，或批准/拒绝一个等待审批的工具调用。Ctrl+C 分离；会话继续运行，重连时会重新加载完整历史。会话处于 `terminated` 或已归档状态时为只读。

```sh
ant beta:sessions connect sesn_011CZkZAtmR3yMPDzynEDxu7          # terminal view
ant beta:sessions connect sesn_011CZkZAtmR3yMPDzynEDxu7 --web    # Console session viewer, served locally
```

| Key | Action |
|---|---|
| Enter | Send input as a `user.message` (Alt+Enter / Ctrl+J for a newline) |
| Esc | Interrupt the running agent (`user.interrupt`) |
| Ctrl+O | Toggle detail: tool inputs/results, token usage, status events (`--verbose` / `-v` starts expanded) |
| PgUp / PgDn | Scroll; scrolling up pauses following, End resumes |
| Ctrl+C (or Ctrl+D on empty input) | Detach |

| 按键 | 动作 |
|---|---|
| Enter | 以 `user.message` 发送输入（Alt+Enter / Ctrl+J 换行） |
| Esc | 打断运行中的代理（`user.interrupt`） |
| Ctrl+O | 切换详情：工具输入/结果、token 用量、状态事件（`--verbose` / `-v` 时默认展开） |
| PgUp / PgDn | 滚动；向上滚动暂停跟随，End 恢复 |
| Ctrl+C（或空输入时 Ctrl+D） | 分离 |

When a call is waiting for approval (`always_ask`, or `auto` with no determination), the input line becomes **Allow tool call?** with **Yes** / **No** / **No, and tell the agent why** - the CLI sends `user.tool_confirmation`, with your typed reason as `deny_message`. In multiagent sessions the terminal view follows the primary thread only (which includes coordinator<->subagent messages).

当某个调用等待审批时（`always_ask`，或无判定结果的 `auto`），输入行会变成 **是否允许工具调用？**，可选 **是** / **否** / **否，并告知代理原因** — CLI 发送 `user.tool_confirmation`，你输入的理由作为 `deny_message`。在多代理会话中，终端视图只跟随主线程（包括 coordinator<->subagent 消息）。

`--web` serves the Console's session viewer from a local server on `127.0.0.1`, prints the URL, and opens the browser (`--no-browser` to skip). The URL works once, within two minutes (reloading that tab is fine; to open it elsewhere, run the command again). The page talks only to the local `ant` process, which makes the API calls, so credentials never leave the CLI; the server runs until Ctrl+C. Unlike the terminal view, the browser viewer follows every thread of a multiagent session.

`--web` 由 `127.0.0.1` 上的本地服务器提供 Console 的会话查看器，打印 URL 并打开浏览器（`--no-browser` 跳过打开）。该 URL 一次性有效、时限两分钟（刷新该标签页没问题；要在别处打开需重新运行命令）。页面只与本地 `ant` 进程通信，由后者发起 API 调用，因此凭据不会离开 CLI；服务器一直运行到 Ctrl+C 为止。与终端视图不同，浏览器查看器会跟随多代理会话的每一条线程。

Needs an interactive terminal (except `--web`) - for scripts use `ant beta:sessions:events stream` / `send`, below.

需要交互式终端（`--web` 除外）— 脚本请用下文的 `ant beta:sessions:events stream` / `send`。

### Interactive session loop (stream-before-send) / 交互式会话循环（先建流后发送）

`ant beta:sessions:events stream` only delivers events emitted *after* the stream opens - so open it **before** sending the kickoff to avoid missing early events. Use process substitution to hold the stream on a file descriptor, send, then read:

`ant beta:sessions:events stream` 只传递在流打开*之后*发出的事件 — 因此要在发送启动消息**之前**先打开流，以免错过早期事件。用进程替换把流挂在一个文件描述符上，发送，然后读取：

```sh
exec {stream}< <(ant beta:sessions:events stream --session-id "$SID" \
  --transform '{type,text:content.#(type=="text").text,err:error.message}' --format yaml)

ant beta:sessions:events send --session-id "$SID" > /dev/null <<'YAML'
events:
  - type: user.message
    content:
      - type: text
        text: Summarize the repo README
YAML

type=
while IFS= read -r -u "$stream" line; do
  case "$line" in
    type:\ session.status_idle) break ;;
    type:\ session.error)
      IFS= read -r -u "$stream" next || next=
      case "$next" in err:\ *) msg=${next#err: } ;; *) msg=unknown ;; esac
      printf '\n[Error: %s]\n' "$msg"; break ;;
    type:\ *) type=${line#type: } ;;
    text:*)
      [[ $type == agent.message ]] || continue
      val=${line#text: }
      case "$val" in '|-'|'|') ;; *) printf '%s' "$val" ;; esac ;;
    \ \ *)
      if [[ $type == agent.message ]]; then printf '%s\n' "${line#  }"; fi ;;
  esac
done
exec {stream}<&-
```

This works for interactive exploration and demos. For application code that needs to react to `agent.tool_use` / `agent.custom_tool_use` events, reconnect after drops, or dedup against `events.list`, use the SDK - see `shared/managed-agents-client-patterns.md`.

这适用于交互式探索和演示。对于需要响应 `agent.tool_use` / `agent.custom_tool_use` 事件、断线后重连、或与 `events.list` 去重的应用代码，请使用 SDK — 见 `shared/managed-agents-client-patterns.md`。

## Scripting patterns / 脚本模式

`--transform id -r` on a list endpoint emits one bare ID per line - compose with `xargs`, or use `--max-items N` to bound the result set without piping through `head`:

列表端点上的 `--transform id -r` 每行输出一个裸 ID — 可与 `xargs` 组合，或用 `--max-items N` 限制结果集而无需经过 `head` 管道：

```sh
FIRST=$(ant beta:agents list --transform id -r --max-items 1)
ant beta:agents:versions list --agent-id "$FIRST" --transform '{version,created_at}' --format jsonl
```

Error shaping mirrors the success path (note: `-r` does not apply to error output - use `--format-error yaml` for an unquoted scalar here):

错误输出的整形与成功路径对称（注意：`-r` 不作用于错误输出 — 此处要去引号的标量请用 `--format-error yaml`）：

```sh
ant beta:agents retrieve --agent-id bogus --transform-error error.message --format-error yaml 2>&1
```

Shell completion: `ant @completion {zsh|bash|fish|powershell}`.

Shell 补全：`ant @completion {zsh|bash|fish|powershell}`。

For the full, always-current reference (including per-endpoint flags), WebFetch the **Anthropic CLI** URL in `shared/live-sources.md`.

完整且始终最新的参考（含各端点旗标），请 WebFetch `shared/live-sources.md` 中的 **Anthropic CLI** URL。
