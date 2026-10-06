---
name: git
description: Source-control safety for Git. Two rules apply whether or not you read the body. First, never commit, amend, push, tag, rebase, cherry-pick, revert, or reset --hard unless the user asked for that exact write in this session. An explicit request to create a new commit counts anywhere in the user's own task text, but it is not a request to amend, push, or tag. Finishing a task without that request is not authorization, so leave your work uncommitted for review. Second, when a Git lock file or corrupt index blocks you, never kill the holding process or bulldoze through with deletions - wait, retry, use the holder's own stop mechanism, or stop and report. Load the body only before writing Git history or recovering Git state.
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Git source-control safety / Git 源码控制安全

The working tree belongs to the user. Make the change, leave it visible, and
let them decide what becomes history.

工作树属于用户。完成更改，让它保持可见，由用户决定什么进入历史。

## Never without an explicit ask / 没有明确要求绝不执行

The user must have asked for that exact write in this session, in their own
words. An explicit request to create a new commit counts anywhere in the user's
own task text, but it is not a request to amend, push, or tag. Finishing a task
without that request does not authorize a history write.

用户必须在本会话中用自己的话要求过那个确切的写入操作。用户任务文本中任何位置出现的"创建新提交"的明确请求都算数，但它不构成修改（amend）、推送或打标签的请求。没有该请求就完成任务并不构成对历史写入的授权。

- `commit`, `commit --amend`, `push`, `tag`, `rebase`, `cherry-pick`, `revert`,
  `merge`, branch deletion, `reset --hard`, path restore, and `clean -f` all
  require an explicit ask.
  `commit`、`commit --amend`、`push`、`tag`、`rebase`、`cherry-pick`、`revert`、
  `merge`、删除分支、`reset --hard`、路径恢复和 `clean -f` 都需要明确要求。
- Never publish merely because publication would be useful.
  绝不因为发布会有用就发布。

【评论】该技能把"完成任务"与"写入历史"的授权严格分离，即使任务本身暗示需要提交，也要求用户显式开口，是针对代理越权改写 Git 历史的防护设计。

## Before an authorized discard / 在获准的丢弃操作之前

Recovery is insurance, not authorization. A save never expands what the user
allowed you to delete or overwrite.

恢复是保险，不是授权。做保存绝不会扩大用户允许你删除或覆盖的范围。

When the user explicitly authorized an operation that may discard or broadly
overwrite workspace changes, and the workspace is an ordinary Git checkout:

当用户明确授权了一个可能丢弃或大范围覆盖工作区更改的操作，且工作区是一个普通的 Git 检出时：

1. After `read_skill` gives the physical Git skill package path, run:

   在 `read_skill` 给出 Git 技能包的物理路径之后，运行：

   ```bash
   bash <git-skill-dir>/scripts/workspace-recovery.sh save before-discard
   ```

2. Keep the printed `refs/tbh/recovery/...` value in your handoff, then perform
   only the exact authorized operation.
   把打印出来的 `refs/tbh/recovery/...` 值记入交接内容，然后只执行那个获得确切授权的操作。
3. If the save fails or the workspace is unsupported, stop before destruction
   and ask the user.
   如果保存失败或工作区不受支持，在破坏发生之前停下并询问用户。

To recover that saved tree later, run:

之后要恢复该保存的树，运行：

```bash
bash <git-skill-dir>/scripts/workspace-recovery.sh restore <recovery-ref>
```

Restore first saves the current workspace, leaves HEAD, branch, and the real
index in place, and restores the selected snapshot into the worktree. The ref
also retains distinct staged bytes when the same path had later worktree edits.
Read those staged bytes with `git show '<recovery-ref>^2:<path>'`; the recovery
commit's second parent is the saved index tree. Ignored files are not saved;
restore refuses an ignored-path collision rather than overwrite data it could
not save. Use only the exact printed ref, not a revision expression.

恢复操作会先保存当前工作区，保持 HEAD、分支和真实索引不动，并把选定的快照恢复到工作树中。当同一路径在之后还有工作树编辑时，该引用还会保留不同的暂存字节。用 `git show '<recovery-ref>^2:<path>'` 读取那些暂存字节；恢复提交的第二个父提交是已保存的索引树。被忽略的文件不会被保存；restore 在遇到被忽略路径冲突时会拒绝执行，而不是覆盖它无法保存的数据。只使用打印出的确切引用，不要使用修订表达式。

## Always fine / 始终允许

Read-only inspection such as `status`, `log`, `show`, `diff`, `blame`,
`branch -v`, `show-ref`, `rev-parse`, `for-each-ref`, and `merge-base` is fine.
Staging named files to inspect `diff --cached`, and a temporary stash around a
tool that requires a clean tree, are also fine when the original state is put
back afterward.

只读检查如 `status`、`log`、`show`、`diff`、`blame`、
`branch -v`、`show-ref`、`rev-parse`、`for-each-ref` 和 `merge-base` 是允许的。为查看 `diff --cached` 而暂存指定文件，以及在需要干净树的工具周围做临时 stash，只要事后恢复原状，也都是允许的。

## The rest / 其余规则

- Do not use a broad add in a tree you did not clean; name the files you changed.
  在你未清理过的树中不要使用宽泛的 add；指名你更改过的文件。
- Do not initialize a repository inside vendored, build, data, or existing
  source-control trees.
  不要在 vendored 目录、构建目录、数据目录或已有源码控制树内初始化仓库。
- Do not hand-edit generated files; change their source of truth.
  不要手工编辑生成文件；去修改它们的真实来源。
- Scratch repositories under a temporary directory are yours to experiment in.
  临时目录下的草稿仓库可供你自由实验。

## Locks and corrupt state / 锁与损坏状态

A Git lock usually means another process is writing. Never kill the holder on
your own initiative. Wait, inspect the holder read-only, use that tool's normal
stop mechanism, or report the block. Never delete a lock without proving no
live process owns it, and never respond to index corruption by deleting state
and committing anyway.

Git 锁通常意味着另一个进程正在写入。绝不要主动杀死持锁进程。可以等待、以只读方式检查持锁者、使用该工具的正常停止机制，或报告阻塞。在没有证明没有存活进程持有锁之前绝不要删除锁，也绝不要以删除状态并强行提交来应对索引损坏。

【评论】"不动锁、不硬闯"的规则承认代理对并发写入者的判断有限，宁可阻塞上报也不冒数据损坏风险，是保守的错误处理策略。

Never commit or push while status reports changes you did not make and cannot
explain.

当 status 显示存在你不曾做出且无法解释的更改时，绝不要提交或推送。

## If you already made an unwanted write / 如果你已经做了不想要的写入

Say so plainly and stop. Do not attempt more history surgery without direction.

如实说明并停下。在没有指示的情况下不要再尝试对历史做手术。
