---
name: "github"
title: "GitHub"
description: "Search and work with the user's GitHub repositories through GitHub's official MCP server."
icon: "github"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# GitHub

Use the installed `github` CLI. Start with `github status` unless this is the
successful OAuth follow-up, which already establishes authorization. A
successful status verifies OAuth and MCP connectivity, not installation or
repository access.

使用已安装的 `github` CLI。从 `github status` 开始，除非当前是 OAuth 成功后的跟进回合——那种情况下授权已经建立。状态成功验证的是 OAuth 与 MCP 的连通性，而不是安装状态或仓库访问权限。

If OAuth is disconnected, run `github authorize-url` and present its returned
`connect_url`. Each user authorizes Muse under their own GitHub identity through
the shared Connect flow. Give only the authorization step before connection;
the successful OAuth follow-up handles repository installation.

如果 OAuth 已断开，运行 `github authorize-url` 并展示其返回的 `connect_url`。每个用户都通过共享的 Connect 流程，以自己的 GitHub 身份授权 Muse。在连接之前只给出授权步骤；仓库安装由 OAuth 成功后的跟进回合处理。

After authorization, run `github install-url` without `--state` before replying.
Do not construct the URL yourself or reuse OAuth state. If the command fails or
returns no `install_url`, explain that the installation link could not be
generated; do not create a widget, invent a link, or claim setup is complete.

授权之后，在回复前运行不带 `--state` 的 `github install-url`。不要自己构造该 URL，也不要复用 OAuth state。如果命令失败或未返回 `install_url`，说明无法生成安装链接；不要创建组件，不要编造链接，也不要声称设置已完成。

Briefly confirm authorization and explain that an account owner or organization
admin selects which repositories Muse may access. Present the exact returned
`install_url` with `widget.create`, using `kind: "list"`. In `data.items`, include
exactly one row with `title: "Select repositories"`,
`subtitle: "Choose which repositories Muse can access"`, `type: "link"`, and
`data.url` copied from `install_url`. Set the row's `image_url` to GitHub's
official icon: `https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png`.
Omit the list heading. Place the returned `embed_token` on its own line in the
reply; do not also print the URL, icon, or a Markdown link. The row opens GitHub
to install or configure repository access.

简要确认授权已完成，并说明由账户所有者或组织管理员选择 Muse 可以访问哪些仓库。用 `widget.create` 展示返回的 `install_url` 原文，使用 `kind: "list"`。在 `data.items` 中恰好包含一行，其 `title: "Select repositories"`、`subtitle: "Choose which repositories Muse can access"`、`type: "link"`，且 `data.url` 复制自 `install_url`。将该行的 `image_url` 设为 GitHub 官方图标：`https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png`。省略列表标题。把返回的 `embed_token` 单独放在回复中一行；不要再打印 URL、图标或 Markdown 链接。该行会打开 GitHub 以安装或配置仓库访问。

If `widget.create` is unavailable or creation fails, use a Markdown link labeled  
**Select repositories** within a sentence, with words after the link on the same
line (for example, "Open `[Select repositories](URL)` to choose which repositories
Muse can access.", substituting the exact returned URL). Do not put the link on
its own line or at the end of a line, where it becomes a large preview card.

如果 `widget.create` 不可用或创建失败，则在一个句子中使用标注为 **Select repositories** 的 Markdown 链接，链接后面在同一行还有文字（例如 "Open `[Select repositories](URL)` to choose which repositories Muse can access."，其中替换为返回的确切 URL）。不要把链接单独放在一行或放在行尾，否则它会变成一张大的预览卡片。

If OAuth is already connected and repository setup is needed, give this
repository-selection step without restarting OAuth. For an existing
installation, GitHub opens its settings and may not redirect back to Muse.
Ask the user to return to chat after configuring repository access.

如果 OAuth 已连接且需要配置仓库，直接给出这一仓库选择步骤，不要重启 OAuth。对于既有安装，GitHub 会打开其设置页面，且可能不会重定向回 Muse。请用户在配置完仓库访问后返回对话。

After the user finishes, verify access to the repository needed for their task
with a reviewed read tool. Do not claim private-repository access from OAuth
status or a click on the installation link alone. The user token is limited by
both that user's access and the app installation's repository selection.
Installation does not require repeating OAuth.

用户完成后，用一个已审查的读取工具验证其任务所需仓库的访问权限。不要仅凭 OAuth 状态或对安装链接的一次点击就宣称拥有私有仓库访问权。用户令牌同时受该用户自身权限和应用安装时的仓库选择所限。安装无需重复 OAuth。

Do not disconnect or repeat OAuth just because a repository is missing. Check
the signed-in identity and installation's repository selection first; reconnect
when authentication needs repair or the configured GitHub App identity changes.

不要仅仅因为缺少某个仓库就断开连接或重复 OAuth。先检查已登录身份和安装的仓库选择；当认证需要修复或已配置的 GitHub App 身份发生变化时，才重新连接。

GitHub uses one fixed, Muse-owned GitHub App identity for every user, like the
fixed Slack and QuickBooks app identities. The app must permit installation on
the intended accounts; this does not require a GitHub Marketplace listing. During
validation, install it only on disposable test repositories. Broader production
rollout remains gated on review and explicit organization installation.

GitHub 对每个用户都使用一个固定的、由 Muse 持有的 GitHub App 身份，与固定的 Slack 和 QuickBooks 应用身份类似。该应用必须允许在预期账户上安装；这不要求有 GitHub Marketplace 上架。验证期间，只把它安装在一次性测试仓库上。更大范围的生产部署仍以审查通过和显式的组织级安装为前提。

Run `github list-tools` to inspect the live provider catalogue and schemas. Each tool includes a local `hatch_command` classification:

运行 `github list-tools` 检查提供方的实时目录与 schema。每个工具都带有一个本地 `hatch_command` 分类：

- `call-read-tool` is limited to Muse's reviewed read-only allowlist and uses the connector's read permission.
- `call-read-tool` 仅限于 Muse 已审查的只读允许列表，使用连接器的读取权限。
- `call-tool` is the write-capable fallback for changes and every new or unknown tool; it requires approval every time.
- `call-tool` 是面向变更操作以及所有新的或未知工具的、具备写入能力的回退通道；它每次都需要批准。

Call the tool using the returned command:

使用返回的命令调用工具：

```text
github call-read-tool --name <reviewed-read-tool> --arguments-json '<json-object>'
github call-tool --name <write-or-unknown-tool> --arguments-json '<json-object>'
```

GitHub App user tokens do not use classic OAuth scopes, so there is no honest OAuth scope-upgrade flow to expose. Progressive access comes from repository selection during App installation and Muse's separate read/write controls. Never infer read safety from GitHub's live catalogue: the binary rejects unreviewed names on `call-read-tool`, and newly advertised tools remain write-capable until reviewed. Never request OAuth tokens or app credentials in chat.

GitHub App 用户令牌不使用经典 OAuth scope，因此不存在可以诚实呈现的 OAuth scope 升级流程。渐进式访问来自应用安装期间的仓库选择，以及 Muse 独立的读/写控制。绝不要从 GitHub 的实时目录推断读取安全性：二进制程序会在 `call-read-tool` 上拒绝未经审查的名称，而新通告的工具在审查之前始终按具备写入能力对待。绝不要在对话中索要 OAuth 令牌或应用凭据。
