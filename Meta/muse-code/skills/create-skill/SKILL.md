---
name: create-skill
description: Create and validate a new Muse skill — project-local in the current workspace by default, or a personal skill staged for `muse skills install` into the managed personal root. Use ONLY when the user explicitly asks to create a Muse skill or invokes the create-skill skill. Do NOT use for ordinary skill usage, code changes, benchmark tasks, or third-party skill/plugin systems.
---
<!-- BILINGUAL-EN-ZH -->

# Create Skill / 创建技能

Create one new Muse skill. Use this skill only for explicit Muse skill creation
requests, not for ordinary skill usage, code changes, benchmark tasks, or
third-party skill/plugin systems.

创建一个新的 Muse 技能。此技能仅用于明确的 Muse 技能创建请求，不用于普通技能使用、代码修改、基准测试任务或第三方技能/插件系统。

Two scopes exist (ADR 8975):

存在两种作用域（ADR 8975）：

- **Project scope (the default)**: the skill lives in the current workspace at
  `.agents/skills/<skill-id>/`.
  **项目作用域（默认）**：技能位于当前工作区的 `.agents/skills/<skill-id>/`。
- **Personal scope** (the user asked for a "personal", "user", or
  "cross-project" skill): the skill belongs in the managed personal root
  `$CONFIG_DIR/skills/<skill-id>` (`$XDG_CONFIG_HOME/muse`, else
  `$HOME/.config/muse`). You stage and validate the draft in the workspace,
  then hand the user one `muse skills install` command — the store performs the
  managed install (files + provenance), so `skills update` and
  `skills uninstall` keep working on it.
  **个人作用域**（用户要求"个人""用户"或"跨项目"技能）：技能应属于受管的个人根目录 `$CONFIG_DIR/skills/<skill-id>`（`$XDG_CONFIG_HOME/muse`，否则为 `$HOME/.config/muse`）。你先在工作区暂存并校验草稿，然后交给用户一条 `muse skills install` 命令——由存储库执行受管安装（文件 + 来源记录），因此 `skills update` 和 `skills uninstall` 对它仍然有效。

Never write to a foreign harness root (`~/.codex/skills`, `~/.claude/skills`)
or to `$HOME/.agents/skills` — those are import-only sources, never write
targets. Never write directly into `$CONFIG_DIR/skills` either: the store owns
that write, through the install command below.

绝不要写入外部 harness 根目录（`~/.codex/skills`、`~/.claude/skills`）或 `$HOME/.agents/skills`——它们只是仅限导入的来源，绝不是写入目标。也绝不要直接写入 `$CONFIG_DIR/skills`：该写入由存储库通过下面的安装命令独占完成。

【评论】"仅限导入、绝不写入"的约束用于避免与其他 AI 工具的技能目录发生写入冲突，属于跨工具共存的边界设计。

## Scope / 作用域

- Create exactly one directory: `.agents/skills/<skill-id>/` (project scope) or
  the staging directory `.agents/skill-drafts/<skill-id>/` (personal scope —
  deliberately OUTSIDE `.agents/skills/`, so the draft is not loaded as a
  project skill).
  只创建一个目录：`.agents/skills/<skill-id>/`（项目作用域）或暂存目录 `.agents/skill-drafts/<skill-id>/`（个人作用域——刻意放在 `.agents/skills/` 之外，以免草稿被当作项目技能加载）。
- Create exactly one required file: `<that directory>/SKILL.md`.
  只创建一个必需文件：`<该目录>/SKILL.md`。
- Do not create a plugin, install a plugin, enable a skill, trust a plugin, execute
  the generated skill, fetch remote content, or write outside the current workspace.
  不要创建插件、安装插件、启用技能、信任插件、执行生成的技能、获取远程内容，或写入当前工作区之外的任何位置。
- Do not add scripts, assets, references, or extra files unless the user explicitly
  asks for them and the target remains inside the new skill directory.
  除非用户明确要求且目标仍位于新技能目录内，否则不要添加脚本、资源、参考文件或额外文件。

## Inputs / 输入

Before writing files, identify:

写文件之前，先确定：

- `scope`: project (default) or personal — personal only when the user asked for
  a personal/user/cross-project skill.
  `scope`：project（默认）或 personal——仅当用户要求个人/用户/跨项目技能时才用 personal。
- `skill-id`: a portable lowercase identifier for the directory and frontmatter
  `name`.
  `skill-id`：用于目录和 frontmatter `name` 的可移植小写标识符。
- `description`: one clear sentence for the frontmatter.
  `description`：用于 frontmatter 的一句清晰描述。
- `body`: concise instructions that make the generated skill useful on its own.
  `body`：让生成的技能自身即可用的简洁指令。

Ask a short clarification question if the user did not provide enough information
to choose a safe `skill-id` and useful behavior.

如果用户提供的信息不足以选择一个安全的 `skill-id` 和有用的行为，则提出一个简短的澄清问题。

## Safety Checks / 安全检查

Reject the request before writing when:

出现以下情况时，在写入之前拒绝请求：

- the destination is not under `.agents/skills/` (project scope) or
  `.agents/skill-drafts/` (personal staging) in the current workspace;
  目标位置不在当前工作区的 `.agents/skills/`（项目作用域）或 `.agents/skill-drafts/`（个人暂存）之下；
- the requested final destination is a foreign harness root (`~/.codex/skills`,
  `~/.claude/skills`), `$HOME/.agents/skills`, or any absolute path outside the
  two sanctioned roots and the personal staging destination (the workspace
  `.agents/skills/` tree, `$CONFIG_DIR/skills/<skill-id>`, and
  `.agents/skill-drafts/<skill-id>`) — explain the sanctioned path instead;
  请求的最终目标是外部 harness 根目录（`~/.codex/skills`、`~/.claude/skills`）、`$HOME/.agents/skills`，或两个许可根目录与个人暂存目标（工作区 `.agents/skills/` 树、`$CONFIG_DIR/skills/<skill-id>`、`.agents/skill-drafts/<skill-id>`）之外的任何绝对路径——改为说明许可路径；
- the ID is empty, `.`, `..`, contains `/` or `\`, starts with `-`, or contains
  anything except ASCII lowercase letters, digits, hyphen, or underscore;
  ID 为空、为 `.` 或 `..`、包含 `/` 或 `\`、以 `-` 开头，或含有 ASCII 小写字母、数字、连字符、下划线以外的任何字符；
- the ID is a Windows reserved stem such as `con`, `prn`, `aux`, `nul`, `com1`,
  `com2`, `com3`, `com4`, `com5`, `com6`, `com7`, `com8`, `com9`, `lpt1`,
  `lpt2`, `lpt3`, `lpt4`, `lpt5`, `lpt6`, `lpt7`, `lpt8`, or `lpt9`;
  ID 是 Windows 保留名，如 `con`、`prn`、`aux`、`nul`、`com1`、`com2`、`com3`、`com4`、`com5`、`com6`、`com7`、`com8`、`com9`、`lpt1`、`lpt2`、`lpt3`、`lpt4`、`lpt5`、`lpt6`、`lpt7`、`lpt8` 或 `lpt9`；
- the destination already exists, is a symlink, or any parent resolves outside the
  current workspace.
  目标已存在、是符号链接，或任一父目录解析到当前工作区之外。

## Creation Steps / 创建步骤

Use normal file and shell tools with the current workspace as the base. `<dir>`
below is `.agents/skills/<skill-id>` (project scope) or
`.agents/skill-drafts/<skill-id>` (personal scope).

使用普通文件和 shell 工具，以当前工作区为基准。下文的 `<dir>` 指 `.agents/skills/<skill-id>`（项目作用域）或 `.agents/skill-drafts/<skill-id>`（个人作用域）。

1. Check that `<dir>` does not exist.
   检查 `<dir>` 不存在。
2. Create its parent (`.agents/skills` or `.agents/skill-drafts`) if needed.
   按需创建其父目录（`.agents/skills` 或 `.agents/skill-drafts`）。
3. Reserve the leaf directory with a no-replace operation. If another process wins
   the race, stop and report incomplete.
   用不可替换操作预留叶子目录。如果另一进程赢得竞争，停止并报告未完成。
4. Recheck that the reserved directory resolves inside the current workspace and is
   not a symlink.
   复查预留目录解析后位于当前工作区内且不是符号链接。
5. Write `<dir>/SKILL.md`.
   写入 `<dir>/SKILL.md`。
6. Run:
   运行：

   ```sh
   muse skills validate <dir> --json
   ```

7. Treat the draft as complete only when the validator succeeds, returns
   `valid: true`, and reports zero diagnostics or warnings.
   仅当校验器成功、返回 `valid: true` 且报告零诊断或警告时，才视为草稿完成。
8. If validation fails, correct the same draft and revalidate. Stop after three
   correction rounds and report the remaining validator output as incomplete.
   如果校验失败，修正同一草稿并重新校验。三轮修正后停止，并将剩余的校验器输出报告为未完成。

## Generated `SKILL.md` / 生成的 `SKILL.md`

Use this shape:

使用以下形态：

```markdown
---
name: <skill-id>
description: <one sentence>
---

# <Readable Skill Name>

<Instructions for when and how to use the skill.>
```

Keep the generated instructions direct and self-contained. Include only behavior the
user asked for or that is necessary for the skill to work.

保持生成的指令直接且自包含。只包含用户要求的行为或技能正常工作所必需的行为。

## Completion Report / 完成报告

On success, report:

成功时，报告：

- the canonical path to the created directory;
  所创建目录的规范路径；
- the validator command and clean result;
  校验器命令及其干净的结果；
- that no install, enable, trust, or execution step was run;
  未运行任何安装、启用、信任或执行步骤；
- **personal scope only**: the one command that finishes the managed install
  into the personal root — run by the user, so the skills store records the
  install provenance itself:
  **仅个人作用域**：完成向个人根目录受管安装的那一条命令——由用户运行，这样技能存储库会自行记录安装来源：

  ```sh
  muse skills install .agents/skill-drafts/<skill-id>
  ```

  and that the staging directory can be deleted after the install succeeds.
  并说明安装成功后可以删除暂存目录。

On failure, report:

失败时，报告：

- what operation failed;
  哪个操作失败；
- the destination if it was reserved;
  如果已预留，则报告目标位置；
- the remaining diagnostics or tool error;
  剩余的诊断信息或工具错误；
- which checks were not completed.
  哪些检查未完成。
