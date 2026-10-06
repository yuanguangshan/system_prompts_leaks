---
name: "skill_creator"
description: "Create or update a workspace skill: its description, structure, instructions, and supporting files."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Skill Creator / 技能创建器

## Purpose / 目的
Create or update a skill that is easy to trigger, concise to load, and backed by references or helper code only when they materially improve reliability.

创建或更新一个易于触发、加载精简的技能；只有当参考资料或辅助代码能实质提升可靠性时才引入它们。

## Workflow / 工作流程
1. Clarify the capability, likely trigger phrases, and the target workspace skill path (`~/workspace/skills/<name>/`).
   明确能力本身、可能的触发短语，以及目标工作区技能路径（`~/workspace/skills/<name>/`）。
2. Choose a narrow scope. Prefer one clear job per skill. Split unrelated jobs into separate skills.
   选择窄范围。每个技能最好只做一件清晰的事。把不相关的工作拆成独立技能。
3. Plan the file layout before editing:
   编辑之前先规划文件布局：
   - Keep only the operational core in `SKILL.md`.
     `SKILL.md` 中只保留操作性核心内容。
   - Put bulky docs, examples, schemas, or tutorials in `references/`.
     把大篇幅文档、示例、模式（schema）或教程放进 `references/`。
   - Put templates or output assets in `assets/` only when the final output uses them.
     只有当最终输出会用到模板或输出资产时，才把它们放进 `assets/`。
   - Prefer helper binaries or checked-in helpers in `bin/` over prompt-side protocol or auth instructions.
     优先使用 `bin/` 中的辅助二进制或入库的辅助工具，而不是在提示词侧编写协议或认证说明。
4. Draft or update frontmatter. Required: `name`, `description`.
   起草或更新 frontmatter。必填项：`name`、`description`。
5. Draft or update the body:
   起草或更新正文：
   - Tool-backed skills: `Purpose`, `Tooling`, `Auth`, `Operating Rules`
     工具支撑型技能：`Purpose`、`Tooling`、`Auth`、`Operating Rules`
   - Workflow-only skills: `Purpose`, `Workflow`, `Output Contract`, `Operating Rules`
     纯工作流技能：`Purpose`、`Workflow`、`Output Contract`、`Operating Rules`
   - Keep examples short and directly executable
     示例保持简短且可直接执行
6. Trim aggressively. Remove long API docs, schema dumps, and setup essays from `SKILL.md`. If a detail is useful but not needed on every trigger, move it to `references/`.
   大刀阔斧地精简。把冗长的 API 文档、模式倾倒和安装长文从 `SKILL.md` 中移除。某个细节有用但不是每次触发都需要时，移到 `references/`。
7. Sanity-check the result:
   对结果做合理性检查：
   - The description should say what the skill does and when it should trigger.
     description 应说明技能做什么、何时应触发。
   - The body should tell the model what to do next, not explain the whole domain.
     正文应告诉模型下一步做什么，而不是解释整个领域。
   - Commands, paths, and auth flows must match real repo/runtime behavior.
     命令、路径和认证流程必须与真实的仓库/运行时行为一致。

## Connector Credentials / 连接器凭据
Collecting a provider's credential is `credentials.request_api_access`, not a file you write. It is a sequence with external dependencies, and the tool enforces the order and refuses the schemes Muse cannot express.

收集提供商凭据使用的是 `credentials.request_api_access`，而不是你自己写一个文件。它是一个带外部依赖的序列，工具会强制执行顺序，并拒绝 Muse 无法表达的方案。

Using that credential is authored here, but do not start from an empty file. Once the connector is connected, scaffold it:

使用该凭据的部分在这里编写，但不要从空文件起步。连接器连上之后，先用脚手架生成：

```
/opt/hatch/skills/skill-creator/bin/scaffold-connector-skill --provider <provider>
```

It reads the connector from authd and writes a `SKILL.md` whose `Tooling` and `Auth` sections already carry the credential mechanics: which helper to import, where the value goes, which hosts are allowed, and how to replace a credential that stops working. Write the CLIs into the `bin/` it creates, and leave those two sections as generated.

它会从 authd 读取连接器并生成 `SKILL.md`，其 `Tooling` 和 `Auth` 两节已带有凭据机制：导入哪个辅助工具、值放在哪里、允许哪些主机，以及如何更换失效的凭据。把 CLI 写进它创建的 `bin/` 中，这两节保持生成时的原样。

A 401 or 403 from the provider is a question about the request before it is a question about the key. Check that the credential was attached at all: a request built without the helper carries nothing, and that looks exactly like a wrong or under-scoped token.

提供商返回 401 或 403 时，先怀疑请求本身，再怀疑密钥。先检查凭据是否根本没有附上：不经辅助工具构建的请求什么都不会携带，其表现与错误或权限不足的令牌一模一样。

【评论】这条排障次序（先查请求是否携带凭据、再查令牌是否有效）针对的是模型容易把所有 401/403 都归因于密钥错误的误判倾向。

## Operating Rules / 操作规则
1. Preserve working commands and repo conventions; do not invent binaries, paths, or auth flows.
   保留可用的命令和仓库约定；不要凭空编造二进制、路径或认证流程。
2. Prefer minimal frontmatter and on-demand loading. Only add metadata the skill actually needs.
   frontmatter 从简，按需加载。只添加技能真正需要的元数据。
3. Give auth its own section instead of burying it in operating rules.
   认证单独成节，而不要埋在操作规则里。
4. Use existing setup/auth helpers when they exist. Do not tell the model to hand-write config files if a bundled helper already owns that flow.
   已有的安装/认证辅助工具要优先使用。如果内置辅助工具已经负责某个流程，就不要让模型手写配置文件。
5. Create `references/` only when it materially shortens `SKILL.md`; avoid duplicating the same guidance in both places.
   只有当它能显著缩短 `SKILL.md` 时才创建 `references/`；避免在两处重复同一指导。
6. If you create or edit Python CLIs, compile them with `python3 -m py_compile ~/workspace/skills/<skill-name>/bin/*.py` before reporting success.
   如果创建或修改了 Python CLI，先执行 `python3 -m py_compile ~/workspace/skills/<skill-name>/bin/*.py` 编译通过后再报告成功。

## Reference Guide / 参考指南
For naming rules, resource-splitting heuristics, templates, and a review checklist, read [references/authoring_guide.md](references/authoring_guide.md).

命名规则、资源拆分经验法则、模板和审查清单，请阅读 [references/authoring_guide.md](references/authoring_guide.md)。
