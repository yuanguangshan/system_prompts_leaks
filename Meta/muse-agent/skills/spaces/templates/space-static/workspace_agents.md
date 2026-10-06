<!-- BILINGUAL-EN-ZH -->
# Building this web artifact / 构建这个 web artifact

This directory is a web artifact — a lite static page: one self-contained
`index.html` plus any binary assets under `./assets/`, with no server, database,
or migrations (see `space.json` for its runtime and slug).

本目录是一个 web artifact——一个轻量静态页面：一个自包含的 `index.html`，加上 `./assets/` 下的二进制资源，没有服务器、数据库或迁移（其运行时与 slug 见 `space.json`）。

Build, audit, and ship it only through the web-artifact builder interface your
session provides — the exact plan → build → audit → submit flow, plus how to
edit or inspect an existing artifact, are in your builder instructions and the
artifacts skill, which stay current if that interface ever changes. Do not hand-edit the built bundle under `.space-build/`,
and do not `bun run build`: neither publishes the artifact.

只能通过你的会话所提供的 web-artifact 构建器界面来构建、审计和发布它——确切的 plan → build → audit → submit 流程，以及如何编辑或检查已有 artifact，都在你的构建器指令和 artifacts 技能中；即使该界面将来发生变化，这些说明也会保持最新。不要手工编辑 `.space-build/` 下已构建的产物，也不要运行 `bun run build`：这两者都不会发布 artifact。

If you are not the builder subagent (for example, the main assistant landed
here), do not build from this directory. List the existing artifacts and
request a change by describing the edit — that spawns a builder to do the
work.

如果你不是构建器子智能体（例如主助手进到了这里），不要从这个目录执行构建。请列出现有的 artifact，并通过描述修改内容来请求变更——这会派生出一个构建器来完成工作。

【评论】模板刻意不写死构建流程细节，而是指向独立的构建器指令，以免模板随构建器界面演进而过时；同时把构建权限隔离到专门的子智能体。
