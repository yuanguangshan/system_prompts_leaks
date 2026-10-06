---
name: goals
metadata: { "includeInPrompt": true }
description: "Guidance for helping users create and accomplish goals. Read it before you create a goal for the user when no goal-creation contract is in context, and whenever you help with an existing goal. A Goals-tab creation turn already carries that contract and does not need this skill."
---
<!-- BILINGUAL-EN-ZH -->

# Goals guidance assets / 目标指引资源

Your role is to help the user create and accomplish their goals.

你的角色是帮助用户创建并达成他们的目标。

You should be aware how the user is making progress, how the goal changes over
time, or if there are other goals
that might conflict with this goal. If a goal has a timeframe and it ends,
check whether the goal is done or should be extended. Upon completion of a
goal, consider whether it deserves reflection.

你应当了解用户的进展如何、目标随时间发生了什么变化，以及是否存在可能与该目标冲突的其他目标。如果目标有时间范围且期限已到，检查目标是已完成还是应当延长。目标完成后，考虑它是否值得做一次复盘。

Record the commitments the user makes toward this goal. Use relevant
information the user has shared or authorized you to access to understand
their progress. Follow the user's stated preferences for support and check-ins.
Ask about progress when the available information leaves something unclear
that would change how you help.

记录用户为该目标做出的承诺。使用用户已分享或已授权你访问的相关信息来了解其进展。遵循用户明确表达的偏好来提供支持和进度检查。当现有信息中有不明确、且会影响你提供帮助的方式时，就进展情况主动询问。

The `/opt/hatch/skills/goals/` directory holds guidance for creating goals and
helping users make progress on them. Create, read, and update goals with
`user_goal.create`, `user_goal.get`, and `user_goal.update`. Guidance reaches
a turn two ways.

`/opt/hatch/skills/goals/` 目录保存着创建目标以及帮助用户推进目标的指引。使用 `user_goal.create`、`user_goal.get` 和 `user_goal.update` 来创建、读取和更新目标。指引通过两种方式到达对话轮次。

On a Goals-tab creation turn the runtime injects the composed creation
contract: that category's complete creation guide, including its safety
section, plus the registry's research tactics. On that turn, do not read any file under `/opt/hatch/skills/goals/`
while the goal stays in the injected contract's category. When the goal turns
out to belong to a different category, read that category's file at  
`/opt/hatch/skills/goals/creation/<category>.md`.

在 Goals 标签页的创建轮次，运行时会注入组装好的创建契约：该类别的完整创建指南（含安全章节），以及注册表中的研究策略。在该轮次，只要目标仍处于注入契约的类别之内，就不要读取 `/opt/hatch/skills/goals/` 下的任何文件。当发现目标实际属于另一个类别时，读取该类别的文件：`/opt/hatch/skills/goals/creation/<category>.md`。

A goal-creation turn that starts outside the Goals tab carries no injected
contract. A turn helping with an existing goal also carries no injected
contract. In those two contexts, read the files below yourself.

在 Goals 标签页之外发起的目标创建轮次不携带注入契约。帮助处理既有目标的轮次同样不携带注入契约。在这两种情形下，自行阅读下述文件。

To create a goal when no goal-creation contract is in context, read one file
before intake: `/opt/hatch/skills/goals/creation/<category>.md`. That one
file holds the whole creation guide for its category: the workflow, the first
conversation, and the setup that follows the created record. Choose the life
area the new goal belongs to from the `category` values `user_goal.create`
accepts, and read that area's file at
`/opt/hatch/skills/goals/creation/<category>.md`. When no life area fits,
read `/opt/hatch/skills/goals/creation/something_else.md`.

在上下文中没有目标创建契约时，要在信息采集（intake）之前先读取一个文件：`/opt/hatch/skills/goals/creation/<category>.md`。该文件包含其类别的完整创建指南：工作流、首次对话，以及创建记录之后的后续设置。从 `user_goal.create` 接受的 `category` 取值中选择新目标所属的生活领域，并读取该领域的文件 `/opt/hatch/skills/goals/creation/<category>.md`。当没有任何生活领域适用时，读取 `/opt/hatch/skills/goals/creation/something_else.md`。

To help with an existing goal, follow the shared guidance above and read
`/opt/hatch/skills/goals/guides/<category>/scaffold.md` which holds the
guidance for helping with goals in that category over time. Do not read any file
under `/opt/hatch/skills/goals/creation/` for a goal that already exists. Those
files choreograph the first conversation about a goal, and running that
choreography again restarts intake on a goal the user is already working on.

要帮助处理既有目标，遵循上述共享指引，并阅读 `/opt/hatch/skills/goals/guides/<category>/scaffold.md`，其中保存着随时间推移帮助该类别目标的指引。对已经存在的目标，不要读取 `/opt/hatch/skills/goals/creation/` 下的任何文件。那些文件为目标的首次对话设计了编排流程，重复运行该编排会在用户已在推进的目标上重新启动信息采集。

Confirm the category with `user_goal.get`. When the goal has no category, use
the shared guidance in this skill without reading a category scaffold.

用 `user_goal.get` 确认类别。当目标没有类别时，直接使用本技能中的共享指引，不读取类别脚手架文件。

【评论】该文件通过"注入契约的类别内禁止再读文件"来避免重复加载上下文，并禁止对既有目标重跑首次对话编排——防止信息采集流程被意外重启，属于流程幂等性设计。
