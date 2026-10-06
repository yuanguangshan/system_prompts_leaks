---
name: fewer-permission-prompts
description: Scan your transcripts for common read-only Bash and MCP tool calls, then add a prioritized allowlist to project .claude/settings.json to reduce permission prompts.
---
<!-- BILINGUAL-EN-ZH -->

# Fewer Permission Prompts / 减少权限提示

Look through my transcripts' MCP and bash tool calls, and based on those, make a prioritized list of patterns that I should add to my permission allowlist to reduce permission prompts. Focus on read-only commands.

查阅我的会话记录中的 MCP 和 bash 工具调用，据此制定一份按优先级排列的模式清单，供我加入权限允许列表以减少权限提示。只关注只读命令。

The format for permissions is: `Bash(foo*)`, `Bash(foo)`, `Bash(foo bar *)`, `mcp__slack__slack_read_thread`, etc.

权限的格式为：`Bash(foo*)`、`Bash(foo)`、`Bash(foo bar *)`、`mcp__slack__slack_read_thread` 等。

Then, add these to the project `.claude/settings.json` under `permissions.allow`.

然后，把这些条目添加到项目 `.claude/settings.json` 的 `permissions.allow` 之下。

## Steps / 步骤

1. **Locate transcripts.** Session transcripts live at `~/.claude/projects/<sanitized-cwd>/*.jsonl`. Each line is a JSON object. Tool calls appear as `assistant` messages with `message.content[]` entries of `type: "tool_use"`. The `name` field identifies the tool (e.g. `"Bash"`, `"mcp__slack__slack_read_thread"`); for Bash, `input.command` is the shell string.

   **定位会话记录。** 会话记录位于 `~/.claude/projects/<sanitized-cwd>/*.jsonl`。每行是一个 JSON 对象。工具调用以 `assistant` 消息的形式出现，其 `message.content[]` 条目的 `type: "tool_use"`。`name` 字段标识工具（例如 `"Bash"`、`"mcp__slack__slack_read_thread"`）；对 Bash 而言，`input.command` 是 shell 命令字符串。

   Scan the recent transcripts across the user's projects dir — not just the current project — so the allowlist reflects their actual usage. Cap the scan at a reasonable number of recent sessions (e.g. 50 most-recently-modified JSONL files) so this stays fast.

   扫描用户 projects 目录下所有项目的近期会话记录——而不仅仅是当前项目——使允许列表反映其真实使用情况。把扫描限制在合理数量的近期会话内（例如最近修改的 50 个 JSONL 文件），以保持速度。

2. **Extract tool-call frequencies.**
   **统计工具调用频率。**
   - For `Bash` calls: parse `input.command`, take the leading command token (handling `sudo`, `timeout`, pipes, `&&`, env-var prefixes). Record the command + first subcommand pair (e.g. `git status`, `gh pr view`, `ls`, `cat`).
     对 `Bash` 调用：解析 `input.command`，取开头的命令 token（处理 `sudo`、`timeout`、管道、`&&`、环境变量前缀）。记录"命令 + 首个子命令"对（例如 `git status`、`gh pr view`、`ls`、`cat`）。
   - For MCP calls: record the full tool name (e.g. `mcp__slack__slack_read_thread`).
     对 MCP 调用：记录完整工具名（例如 `mcp__slack__slack_read_thread`）。
   - Count occurrences across the scanned transcripts.
     统计在已扫描会话记录中的出现次数。

3. **Filter to read-only.** Keep only commands that don't mutate state. Examples of read-only: `ls`, `cat`, `pwd`, `git status`, `git log`, `git diff`, `git show`, `git branch`, `rg`, `grep`, `find`, `head`, `tail`, `wc`, `file`, `which`, `echo`, `date`, `gh pr view`, `gh pr list`, `gh pr diff`, `gh issue view`, `gh issue list`, `gh run list`, `gh run view`, `gh api` (GET), `bun run typecheck`, `bun run lint`, `bun run test` (for tests that don't mutate), `docker ps`, `docker logs`, `kubectl get`, `kubectl describe`, `ps`, `top`, `df`, `du`, `env`, `printenv`, any MCP tool with `read`/`get`/`list`/`search`/`view` in its name.

   **只保留只读命令。** 仅保留不改变状态的命令。只读命令示例：`ls`、`cat`、`pwd`、`git status`、`git log`、`git diff`、`git show`、`git branch`、`rg`、`grep`、`find`、`head`、`tail`、`wc`、`file`、`which`、`echo`、`date`、`gh pr view`、`gh pr list`、`gh pr diff`、`gh issue view`、`gh issue list`、`gh run list`、`gh run view`、`gh api`（GET）、`bun run typecheck`、`bun run lint`、`bun run test`（针对无副作用的测试）、`docker ps`、`docker logs`、`kubectl get`、`kubectl describe`、`ps`、`top`、`df`、`du`、`env`、`printenv`，以及名称中含 `read`/`get`/`list`/`search`/`view` 的任何 MCP 工具。

   Drop anything that writes, deletes, renames, pushes, merges, installs, or runs a build/test that has side effects. When in doubt, leave it out.

   丢弃任何会写入、删除、重命名、推送、合并、安装或运行有副作用的构建/测试的命令。拿不准时，宁可不放。

   **Never allowlist a pattern that grants arbitrary code execution.** A wildcard rule for any of these (e.g. `Bash(python3:*)`) is equivalent to allowing arbitrary code execution. This list is not exhaustive — apply the same rule to anything in the same category:
   **绝不允许把授予任意代码执行能力的模式加入允许列表。** 针对以下任何一项的通配规则（例如 `Bash(python3:*)`) 都等价于允许任意代码执行。此列表并不详尽——对同类事物一概适用同样的规则：
   - Interpreters: `python`/`python3`, `node`, `bun`, `deno`, `ruby`, `perl`, `php`, `lua`, etc.
     解释器：`python`/`python3`、`node`、`bun`、`deno`、`ruby`、`perl`、`php`、`lua` 等。
   - Shells: `bash`, `sh`, `zsh`, `fish`, `eval`, `exec`, `ssh`, etc.
     Shell：`bash`、`sh`、`zsh`、`fish`、`eval`、`exec`、`ssh` 等。
   - Package runners: `npx`, `bunx`, `uvx`, `uv run`, etc.
     包运行器：`npx`、`bunx`、`uvx`、`uv run` 等。
   - Task-runner wildcards: `npm run *`, `yarn run *`, `pnpm run *`, `bun run *`, `make *`, `just *`, `cargo run *`, `go run *`, etc. — an exact `Bash(bun run typecheck)` is fine, `Bash(bun run *)` is not
     任务运行器通配：`npm run *`、`yarn run *`、`pnpm run *`、`bun run *`、`make *`、`just *`、`cargo run *`、`go run *` 等——精确的 `Bash(bun run typecheck)` 没问题，`Bash(bun run *)` 则不行。
   - `gh api *`, `docker run`/`exec`, `kubectl exec`, `sudo`, and similar
     `gh api *`、`docker run`/`exec`、`kubectl exec`、`sudo` 及类似命令。

   【评论】这一条是典型的权限收敛设计：把"任意解释器/Shell 通配"等同于任意代码执行来禁止，防止允许列表成为绕过权限体系的旁路。

4. **Drop commands Claude Code already auto-allows.** These don't need an allowlist entry — they never prompt. If you see any of these in the transcripts, skip them; don't suggest them to the user.

   **剔除 Claude Code 已自动放行的命令。** 这些命令不需要加入允许列表——它们从不触发提示。如果在会话记录中看到它们，直接跳过；不要向用户建议。

   - **Always auto-allowed (any args):** `cal`, `uptime`, `cat`, `head`, `tail`, `wc`, `stat`, `strings`, `hexdump`, `od`, `nl`, `id`, `uname`, `free`, `df`, `du`, `locale`, `groups`, `nproc`, `basename`, `dirname`, `realpath`, `cut`, `paste`, `tr`, `column`, `tac`, `rev`, `fold`, `expand`, `unexpand`, `fmt`, `comm`, `cmp`, `numfmt`, `readlink`, `diff`, `true`, `false`, `sleep`, `which`, `type`, `expr`, `seq`, `tsort`, `pr`, `echo`, `ls`, `cd`.
     **始终自动放行（任意参数）：** `cal`、`uptime`、`cat`、`head`、`tail`、`wc`、`stat`、`strings`、`hexdump`、`od`、`nl`、`id`、`uname`、`free`、`df`、`du`、`locale`、`groups`、`nproc`、`basename`、`dirname`、`realpath`、`cut`、`paste`、`tr`、`column`、`tac`、`rev`、`fold`、`expand`、`unexpand`、`fmt`、`comm`、`cmp`、`numfmt`、`readlink`、`diff`、`true`、`false`、`sleep`、`which`、`type`、`expr`、`seq`、`tsort`、`pr`、`echo`、`ls`、`cd`。
   - **Auto-allowed with zero args only:** `pwd`, `whoami`, `alias`.
     **仅在无参数时自动放行：** `pwd`、`whoami`、`alias`。
   - **Auto-allowed exact forms:** `claude -h`, `claude --help`, `node -v`, `node --version`, `python --version`, `python3 --version`, `ip addr`.
     **精确形式自动放行：** `claude -h`、`claude --help`、`node -v`、`node --version`、`python --version`、`python3 --version`、`ip addr`。
   - **Auto-allowed with safe flags only (validated):** `xargs`, `file`, `sed` (read-only expressions), `sort`, `man`, `help`, `netstat`, `ps`, `base64`, `grep`, `egrep`, `fgrep`, `sha256sum`, `sha1sum`, `md5sum`, `tree`, `date`, `hostname`, `lsof`, `pgrep`, `tput`, `ss`, `fd`, `fdfind`, `aki`, `rg`, `jq`, `uniq`, `history`, `arch`, `ifconfig`, `pyright`, `find` (blocks `-delete`/`-exec`/`-execdir`/`-ok`/`-okdir`/`-fprint*`/`-fls`/`-files0-from`), `printf` (blocks any `-flag`), `test` (blocks `-v`/`-R`/`-a`/`-o`).
     **仅带安全标志时自动放行（经校验）：** `xargs`、`file`、`sed`（只读表达式）、`sort`、`man`、`help`、`netstat`、`ps`、`base64`、`grep`、`egrep`、`fgrep`、`sha256sum`、`sha1sum`、`md5sum`、`tree`、`date`、`hostname`、`lsof`、`pgrep`、`tput`、`ss`、`fd`、`fdfind`、`aki`、`rg`、`jq`、`uniq`、`history`、`arch`、`ifconfig`、`pyright`、`find`（拦截 `-delete`/`-exec`/`-execdir`/`-ok`/`-okdir`/`-fprint*`/`-fls`/`-files0-from`）、`printf`（拦截任何 `-flag`）、`test`（拦截 `-v`/`-R`/`-a`/`-o`）。
   - **All git read-only subcommands:** `git status`, `git log`, `git diff`, `git show`, `git blame`, `git branch`, `git tag`, `git remote`, `git ls-files`, `git ls-remote`, `git config --get`, `git rev-parse`, `git describe`, `git stash list`, `git reflog`, `git shortlog`, `git cat-file`, `git for-each-ref`, `git worktree list`, etc.
     **全部 git 只读子命令：** `git status`、`git log`、`git diff`、`git show`、`git blame`、`git branch`、`git tag`、`git remote`、`git ls-files`、`git ls-remote`、`git config --get`、`git rev-parse`、`git describe`、`git stash list`、`git reflog`、`git shortlog`、`git cat-file`、`git for-each-ref`、`git worktree list` 等。
   - **All gh read-only subcommands:** `gh pr view`, `gh pr list`, `gh pr diff`, `gh pr checks`, `gh pr status`, `gh issue view`, `gh issue list`, `gh issue status`, `gh run view`, `gh run list`, `gh workflow list`, `gh workflow view`, `gh repo view`, `gh release view`, `gh release list`, `gh api` (GET), `gh auth status`, etc.
     **全部 gh 只读子命令：** `gh pr view`、`gh pr list`、`gh pr diff`、`gh pr checks`、`gh pr status`、`gh issue view`、`gh issue list`、`gh issue status`、`gh run view`、`gh run list`、`gh workflow list`、`gh workflow view`、`gh repo view`、`gh release view`、`gh release list`、`gh api`（GET）、`gh auth status` 等。
   - **Docker read-only subcommands:** `docker ps`, `docker images`, `docker logs`, `docker inspect`.
     **Docker 只读子命令：** `docker ps`、`docker images`、`docker logs`、`docker inspect`。

   Source of truth: `src/tools/BashTool/readOnlyValidation.ts` (`READONLY_COMMANDS`, `READONLY_NOARGS`, `READONLY_EXACT`, `COMMAND_ALLOWLIST`) and `src/utils/shell/readOnlyCommandValidation.ts` (`GIT_READ_ONLY_COMMANDS`, `GH_READ_ONLY_COMMANDS`, `DOCKER_READ_ONLY_COMMANDS`, `RIPGREP_READ_ONLY_COMMANDS`, `PYRIGHT_READ_ONLY_COMMANDS`). If the user is in this repo and you're unsure whether a command is covered, grep these files rather than guessing.

   权威来源：`src/tools/BashTool/readOnlyValidation.ts`（`READONLY_COMMANDS`、`READONLY_NOARGS`、`READONLY_EXACT`、`COMMAND_ALLOWLIST`）和 `src/utils/shell/readOnlyCommandValidation.ts`（`GIT_READ_ONLY_COMMANDS`、`GH_READ_ONLY_COMMANDS`、`DOCKER_READ_ONLY_COMMANDS`、`RIPGREP_READ_ONLY_COMMANDS`、`PYRIGHT_READ_ONLY_COMMANDS`）。如果用户就在本仓库中，而你不确定某命令是否已被覆盖，请 grep 这些文件，不要靠猜。

5. **Pick the pattern form.** Use the narrowest pattern that still covers the observed usage:
   **选择模式形式。** 使用在覆盖已观测用法的前提下最窄的模式：
   - If the user runs many variants (`git log`, `git log --oneline`, `git log main..HEAD`): use `Bash(git log *)` — note the space before `*`, which is required for prefix matching to work correctly.
     如果用户运行多种变体（`git log`、`git log --oneline`、`git log main..HEAD`）：使用 `Bash(git log *)`——注意 `*` 前的空格，前缀匹配要正确工作必须有它。
   - If a single exact invocation is common: use `Bash(foo)` with no wildcard.
     如果某个精确调用很常见：使用不带通配符的 `Bash(foo)`。
   - For MCP: use the full tool name verbatim (no wildcard needed; they're already specific).
     对 MCP：逐字使用完整工具名（无需通配符；它们本身已足够具体）。
   - Never widen a pattern to the point that it conflicts with the rules above (no arbitrary code execution, no mutation/side effects).
     绝不把模式放宽到与上述规则冲突的程度（不允许任意代码执行，不允许变更/副作用）。

6. **Prioritize.** Rank by count descending. Drop anything that appeared fewer than ~3 times — not worth the allowlist entry. Cap the list at the top ~20 so the user can skim it.
   **排定优先级。** 按出现次数降序排列。丢弃出现少于约 3 次的命令——不值得加入允许列表。列表上限为前 20 条左右，方便用户快速浏览。

7. **Present the prioritized list to the user** as a markdown table with columns: rank, pattern, count, one-line description. Example:
   **向用户展示这份按优先级排列的清单**，使用 markdown 表格，列包括：排名、模式、次数、一行描述。示例：

   | # | Pattern | Count | Notes |
   |---|---------|-------|-------|
   | 1 | `Bash(git status *)` | 142 | repo status checks |
   | 2 | `Bash(gh pr view *)` | 87 | PR inspection |
   | 3 | `mcp__slack__slack_read_thread` | 54 | Slack thread reads |

   | # | 模式 | 次数 | 备注 |
   |---|---------|-------|-------|
   | 1 | `Bash(git status *)` | 142 | 仓库状态检查 |
   | 2 | `Bash(gh pr view *)` | 87 | PR 查看 |
   | 3 | `mcp__slack__slack_read_thread` | 54 | Slack 线程读取 |

8. **Merge into `.claude/settings.json`** in the current project (not `~/.claude/settings.json`, not `.claude/settings.local.json`). Create the file if it doesn't exist. Preserve existing keys and existing entries in `permissions.allow`; de-duplicate against what's already there; don't remove anything; don't reorder unrelated fields.
   **合并进当前项目的 `.claude/settings.json`**（不是 `~/.claude/settings.json`，也不是 `.claude/settings.local.json`）。文件不存在则创建。保留既有键和 `permissions.allow` 中的既有条目；与已有内容去重；不删除任何内容；不重排无关字段。

9. **Report back.** Tell the user what you added (count + a few examples), what was already in the allowlist, and what you skipped and why (e.g. "dropped `rm` and `git push` — not read-only; dropped `cat`/`ls`/`git status` — already auto-allowed, no rule needed").
   **汇报结果。** 告诉用户你添加了什么（数量 + 几个示例）、允许列表中已有什么、跳过了什么以及原因（例如"丢弃了 `rm` 和 `git push`——非只读；丢弃了 `cat`/`ls`/`git status`——已自动放行，无需规则"）。

Do not add anything to `permissions.deny` or `permissions.ask`. Do not touch any other settings field.

不要向 `permissions.deny` 或 `permissions.ask` 添加任何内容。不要触碰任何其他设置字段。
