<!-- BILINGUAL-EN-ZH -->
# Scheduling, monitoring, and services / 调度、监控与服务

Use this guide from main or side chat when creating or changing background work.

在主聊天或侧边聊天中创建或更改后台工作时，请使用本指南。

## Choose the mechanism / 选择机制

- Crons run agent tasks on a schedule, including reminders, periodic checks, and work that needs tools or judgment. Use runonce for a single future action and an interval for bounded repeated checks.
  定时任务（cron）按计划运行代理任务，包括提醒、周期性检查以及需要工具或判断的工作。单次未来动作用 runonce，有界的重复检查用 interval。
- Hooks run lightweight polling scripts and wake an agent on a relevant condition. Use them when the detector can check local or public data without an agent turn each time.
  钩子（hook）运行轻量轮询脚本，并在相关条件满足时唤醒代理。当检测器无需每次都消耗一个代理回合就能检查本地或公开数据时，使用它们。
- systemd services supervise programs that need to remain running, such as stream listeners, collectors, or local servers. Use the runtime's guest systemd when a sequence of independent checks cannot do the job.
  systemd 服务监督需要持续运行的程序，例如流监听器、采集器或本地服务器。当一串独立检查无法完成任务时，使用运行时的 guest systemd。

Browser tasks and subagents already deliver completion handoffs. Use those instead of adding another monitor.

浏览器任务与子代理已经提供完成时的交接。使用这些机制，而不是再增加一个监视器。

## Saved jobs and hooks / 已保存的任务与钩子

Record the user's scope, target, cadence, completion condition, owner, and delivery destination. A request to monitor one operation covers the observations needed to finish, not repeating the operation. Reuse an existing goal as owner when applicable. Other reminders and follow-ups need a tracked owner.

记录用户的范围、目标、节奏、完成条件、归属者与交付目的地。监控某个操作的请求只涵盖完成所需的观察，而不是重复执行该操作。适用时复用既有 goal 作为归属者。其他提醒与后续事项需要有可追踪的归属者。

Keep goal-owned working files in `~/workspace/goals/<goal-slug>/hidden_files/`, including checkpoints, run logs, watermarks, and source snapshots. Hook scripts and state use the hook paths below.

将 goal 拥有的工作文件保存在 `~/workspace/goals/<goal-slug>/hidden_files/` 中，包括检查点、运行日志、水位标记和源快照。钩子脚本与状态使用下方的钩子路径。

Check existing jobs first. Change a cron through `cron.view` and `cron.update` on the same id, supplying the complete revised body and preserving unrelated settings. Read it back before confirming. Projected schedule files are read-only copies; editing them does not change the job. Removing and recreating a job is not a repair for its permissions.

先检查既有任务。通过同一 id 上的 `cron.view` 与 `cron.update` 修改 cron，提供完整的修订后 body，并保留无关设置。确认前先读回。投影出的计划文件是只读副本；编辑它们不会改变任务。删除并重建任务不是对其权限问题的修复方式。

Each check should finish, have request and execution timeouts, and avoid overlap. The body describes the work and reporting intent, not calls to terminal handoff tools. Stop finite monitoring when its outcome is resolved, cancelled, or past its deadline. Keep any still-owed delivery separate from further source polling.

每次检查都应当能结束，设有请求与执行超时，并避免重叠。body 描述工作内容与报告意图，而不是对终端交接工具的调用。当有限监控的结果已解决、已取消或已超过期限时，停止它。仍欠付的交付要与后续的源轮询分开处理。

Hook scripts belong under `~/hooks/scripts/`, with state under `~/hooks/state/`. Follow the tool's script protocol, including one terminal `silent` or `wake` decision. Use the provided state helpers so dry runs do not consume detections. New hooks start disabled. Inspect a `hooks.dry_run` result before `hooks.enable`. Use `disable_after_run` for a terminal condition. Hooks have no connector credentials. Do not add secrets to a detector.

钩子脚本放在 `~/hooks/scripts/` 之下，状态放在 `~/hooks/state/` 之下。遵循工具的脚本协议，包括恰好一次终结性的 `silent` 或 `wake` 决定。使用提供的状态辅助工具，使试运行不消耗检测结果。新建的钩子默认禁用。在 `hooks.enable` 之前先查看 `hooks.dry_run` 的结果。终结性条件使用 `disable_after_run`。钩子没有连接器凭据。不要向检测器添加机密。

【评论】钩子被明确剥夺连接器凭据且禁止携带机密，配合"默认禁用、先试运行再启用"的流程，属于最小权限与渐进启用的设计。

Both mechanisms poll. Changes can be missed between checks, and schedules can run late. They do not provide continuous observation or block source events.

两种机制都是轮询式的。检查之间的变更可能被错过，计划也可能延迟运行。它们不提供持续观察，也不阻塞源事件。

## Services in this runtime / 本运行时中的服务

The guest has systemd but boots a minimal `default.target`, not the usual `multi-user.target` or a login session. Prefer the existing guest system manager. Connect startup to the target the guest actually reaches. Use a user service only when its manager, configuration lookup, runtime directory, bus, and startup after replacement are established. A tool's environment variables do not establish the manager's environment. Do not assume lingering works or use host operator facilities.

guest 拥有 systemd，但只启动最小化的 `default.target`，而不是常见的 `multi-user.target` 或登录会话。优先使用既有的 guest 系统管理器。把启动挂接到 guest 实际会到达的 target。仅当用户服务的管理器、配置查找、运行时目录、总线以及替换后的启动都已确认成立时才使用它。工具的环境变量并不等于管理器的环境。不要假设 lingering（管理员驻留）可用，也不要使用宿主机操作员设施。

Keep the canonical unit, program, dependencies, and state under persistent home storage. Use absolute `ExecStart` and `WorkingDirectory` paths and a foreground process. Set an appropriate `Restart` policy, `RestartSec`, and start limits. Allow graceful termination with `TimeoutStopSec` and control child processes with `KillMode=control-group`. Bound resources and logs. Match restart behavior to the program. Successful completion or waiting for acknowledgement may be an intentional exit.

把规范的 unit、程序、依赖与状态保存在持久化的 home 存储中。使用绝对的 `ExecStart` 与 `WorkingDirectory` 路径以及前台进程。设置合适的 `Restart` 策略、`RestartSec` 与启动限制。用 `TimeoutStopSec` 允许优雅终止，用 `KillMode=control-group` 控制子进程。限制资源与日志的规模。让重启行为与程序本身匹配。成功完成或等待确认可能是有意的退出。

Verify which user and group run the service and check access from that service context. Do not assume it inherits tool credentials, connector access, or approvals. Use supported access paths. Do not copy secrets into units, arguments, environment files, or logs.

验证服务以哪个用户和组运行，并从该服务上下文检查访问权限。不要假设它继承工具凭据、连接器访问权限或审批。使用受支持的访问路径。不要把机密复制进 unit、参数、环境文件或日志。

## Recovery after replacement / 替换后的恢复

The runtime manages recovery for saved jobs, subagents, crons, artifacts and resumable agent work. Custom programs need a startup path that restores their prerequisites after replacement. Persistent unit files and `Restart` settings alone do not establish that path.

运行时为已保存任务、子代理、cron、工件以及可恢复的代理工作管理恢复。自定义程序需要一条能在替换后恢复其前置条件的启动路径。仅有持久化的 unit 文件和 `Restart` 设置并不能建立这条路径。

Inspect saved status, logs, recent output, and deployment events before diagnosing a failure. A quiet listener may be waiting for acknowledgement. A failed bootstrap may have partly succeeded. Attribute a failure to deployment only when the evidence connects them. Check interruption and recovery failures separately.

在诊断故障之前，先查看保存的状态、日志、最近的输出与部署事件。安静的监听器可能只是在等待确认。失败的引导可能已部分成功。只有当证据把二者联系起来时，才能把故障归因于部署。中断与恢复失败要分开检查。

Use the supported startup path. If a saved recovery job is needed, keep it bounded and scoped to the requested components. Its interval provides another attempt, not a recovery deadline.

使用受支持的启动路径。如果需要已保存的恢复任务，让它有边界并只覆盖所请求的组件。它的间隔提供的是再一次尝试的机会，而不是恢复期限。

When recovering work, follow these steps.

恢复工作时，遵循以下步骤。

1. Read durable desired state before starting anything. Preserve completed work, pending acknowledgements, cancellations, blocked states, and retry limits. A listener waiting for acknowledgement must not be restarted as though it crashed.
   在启动任何东西之前，先读取持久化的期望状态。保留已完成的工作、待确认事项、取消、阻塞状态与重试限制。等待确认的监听器不得当作崩溃那样重启。
2. Restore required prerequisites and verify them. After manager startup or mount changes, inspect readiness from a fresh tool call. Tool processes have private mount views and may not see mounts created later. Waiting longer or spawning a child in the same old invocation may not fix that view.
   恢复所需的前置条件并验证它们。在管理器启动或挂载变更之后，通过一次全新的工具调用检查就绪状态。工具进程拥有私有的挂载视图，可能看不到后来创建的挂载。等更久或在同一次旧调用中派生子进程可能无法修复该视图。
3. Reconcile each component independently. Inspect running state before starting another copy. One component's timeout must not prevent unrelated components from recovering. Record partial success so the next attempt does not repeat completed work.
   独立地对每个组件进行对账。启动另一份副本之前先检查运行状态。一个组件的超时不得阻止无关组件恢复。记录部分成功，使下一次尝试不重复已完成的工作。
4. Resume from checkpoints and verify useful progress. Write checkpoints atomically and regularly. Abrupt termination can skip shutdown handlers. For external actions, reconcile recorded intent and receipts with the destination before retrying an uncertain outcome.
   从检查点恢复并验证取得了有效进展。原子且定期地写检查点。突然终止可能跳过关闭处理程序。对外部动作，在重试结果不确定的操作之前，先把记录的意图与回执同目的地对账。

On cancellation, save the stop state before stopping the service and its recovery job. Recovery must not undo the user's decision.

取消时，先保存停止状态，再停掉服务及其恢复任务。恢复不得撤销用户的决定。

【评论】强调自动恢复逻辑不得覆盖用户主动做出的取消决定，用于避免自动化行为与用户意图发生冲突。

## Verification and delivery / 验证与交付

Check saved configuration, actual execution, useful progress, and delivery separately. For crons, use `cron.status` and `cron.runs`, then verify that any owed result reached the conversation. For services, inspect the manager and unit, dependencies, logs, and recent output. An active unit or existing PID alone does not establish health.

分别检查保存的配置、实际执行、有效进展与交付。对 cron，使用 `cron.status` 与 `cron.runs`，然后验证任何欠付的结果已送达对话。对服务，检查管理器与 unit、依赖、日志与最近输出。仅凭活跃的 unit 或存在的 PID 不能证明健康。

Test the failure mode you claim to handle. Restarting a worker with its manager and bus intact tests process supervision. Recovery after replacement also requires restoring those prerequisites and starting the work again. Do not infer the latter from the former. Use observed deployment evidence or an authorized recovery test, and name what remains unverified.

测试你声称能处理的故障模式。在管理器与总线完好的情况下重启 worker，测试的是进程监督。替换后的恢复还要求恢复那些前置条件并重新启动工作。不要由前者推断后者。使用观测到的部署证据或经过授权的恢复测试，并指明哪些仍未验证。

Report component states separately when recovery partially succeeds. A failed bootstrap does not prove that every service is down. A quiet listener does not prove a crash. Preserve rate-limit stops, denied access, and uncertain external outcomes rather than retrying them as setup failures.

当恢复只部分成功时，分开报告各组件的状态。引导失败不能证明所有服务都已停机。安静的监听器不能证明崩溃。保留限流停止、被拒绝的访问与不确定的外部结果，而不是把它们当作设置失败而重试。

Deliver results to the originating chat unless the user chose another allowed destination. Provider-connected side chats retain their exact-chat boundary. Keep unchanged observations quiet unless requested, and report failures or gaps that affect the user's outcome. A chat result or push notification is not an arbitrary SMS or a message to another person.

除非用户选择了另一个允许的目的地，否则把结果交付到发起请求的聊天。与提供商连接的侧边聊天保持其精确聊天边界。除非被要求，否则对无变化的观察保持安静，并报告影响用户结果的故障或缺口。聊天结果或推送通知不是随意的短信，也不是发给另一个人的消息。

【评论】最后一句明确了消息传递边界：任务结果只能送达对话或推送通道，不得被当作自由短信发送给任意第三方。
