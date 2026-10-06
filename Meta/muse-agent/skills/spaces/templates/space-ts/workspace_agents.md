<!-- BILINGUAL-EN-ZH -->

# Building this web artifact / 构建此 Web 工件

This directory is a web artifact — a TypeScript space: a React client in
`client/`, server actions in `server/src/actions.ts`, the schema in
`server/src/schema.ts`, and Drizzle SQL migrations in `drizzle/` (see
`space.json` for its runtime and slug).

该目录是一个 Web 工件——一个 TypeScript space：React 客户端位于 `client/`，服务端动作位于 `server/src/actions.ts`，模式（schema）位于 `server/src/schema.ts`，Drizzle SQL 迁移位于 `drizzle/`（其运行时和 slug 见 `space.json`）。

Build, audit, and ship it only through the web-artifact builder interface your
session provides — the exact plan → build → audit → submit flow, how to edit or
inspect an existing artifact, and the schema/migration commands are all in your
builder instructions and the artifacts skill, which stay current if that
interface ever changes. Do not hand-edit the
built bundle under `.space-build/`, and do not `bun run build`: neither
publishes the artifact.

只能通过会话提供的 Web 工件构建器界面来构建、审计和发布它——确切的"计划 → 构建 → 审计 → 提交"流程、如何编辑或检查已有工件、以及模式/迁移命令，都写在构建器说明和 artifacts 技能中，即使该界面发生变化它们也能保持最新。不要手动编辑 `.space-build/` 下已构建的产物，也不要运行 `bun run build`：这两种方式都不会发布工件。

If you are not the builder subagent (for example, the main assistant landed
here), do not build from this directory. List the existing artifacts and
request a change by describing the edit — that spawns a builder to do the
work.

如果你不是构建器子代理（例如主助手进入了本目录），不要从此目录进行构建。应列出现有工件，并通过描述修改内容来请求变更——这会派生一个构建器来完成工作。

## This artifact's data / 此工件的数据

This artifact's data lives in `app.db`, managed by the app: read it with the
artifact inspect data operations and change it through the app's own actions
(`artifact.invoke_action`) or an artifact edit, never by running sqlite or
scripts against the file.

此工件的数据存放在 `app.db` 中，由应用本身管理：应通过工件检查（inspect）数据操作来读取，通过应用自身的动作（`artifact.invoke_action`）或工件编辑来修改，绝不要直接对该文件运行 sqlite 或脚本。

【评论】本文件是典型的"受管生命周期"护栏：把构建、发布和数据访问都限制在专用界面内，禁止绕过流程直接改产物或数据库，以避免绕过审计环节。
