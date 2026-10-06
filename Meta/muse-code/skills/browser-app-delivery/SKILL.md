---
name: browser-app-delivery
description: Deliver a runnable local browser app so the user can operate it on the first handoff. This delivery skill does not choose whether a requested local system should be a browser app. On a current turn that explicitly says to implement a brand-new browser project now, load bundled:greenfield-project-scaffolding before project work only after no-peek or permitted top-level placement facts prove that no named or suitable current-folder target resolves the root; never hand off from planning or answer-only turns, existing work, a named or suitable target, or standalone or single-file delivery. Resolve user-blocking product or interface choices with an available interaction surface; resolving those choices alone never authorizes implementation. Always call read_skill for bundled:browser-app-delivery before creating or materially changing a local browser app that owns its start path or a playable browser game. Also load it when the user requests a paste-ready, single-file, offline, or immediately playable browser artifact, even if no local start path should remain, unless the user says they will open or check it themselves or explicitly declines verification. On completed local-server delivery, include exact start commands, a concrete local URL, and explicit browser-open wording. On completed standalone delivery, hand off the exact artifact path or paste URL and smoke result without inventing a server. Do not load it for a self-contained static file that has no owned start path if the user either says they will open or check it themselves or explicitly declines verification. Do not load it for component/style-only edits, deployed sites, backend/API-only work, browser QA, review, explanation, plan-only, or explicit stop/no-tool turns.
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Browser App Delivery / 浏览器应用交付

For a brand-new browser project, apply this hard boundary before project work.
Do NOT call `read_skill` for `bundled:greenfield-project-scaffolding` on a
requirements-guidance, planning, option-selection, multi-decision, or answer-only
turn; from prior-turn answers, authorization, or reminders without new
implementation-now words; for existing work, server start/restart, verification,
or handoff-only work; for a named path or a confirmed-suitable here/current-folder
target; or for a requested standalone, paste-ready, single-file, or snippet artifact.

对于全新的浏览器项目，在开展项目工作之前先套用这条硬边界。在以下情况下，绝不要为 `bundled:greenfield-project-scaffolding` 调用 `read_skill`：当前是需求指导、规划、选项选择、多重决策或仅回答的回合；仅凭此前回合的回答、授权或提醒，而没有新的"现在就实现"的措辞；处理的是既有工作、服务器启动/重启、验证或仅交付的工作；目标是一个已被点名的路径或确认合适的此处/当前文件夹目标；或者请求的是独立、可直接粘贴、单文件或代码片段形式的工件。

【评论】该文件用大量限定词把"加载脚手架技能"的触发条件收得很紧，目的是防止代理在规划或答疑回合擅自开始实现——一种约束代理越权的阀门式设计。

Only when this same user turn explicitly says to start, implement, build,
scaffold, or go ahead with a new project now may unresolved placement trigger
the handoff. Do not load the greenfield skill merely to decide whether it is
eligible. If inspection is forbidden, do not list or read the workspace; use
only the request and `pwd`. Otherwise inspect only top-level placement facts
first. Call the greenfield skill before any project-content inspection or write
only when no named or suitable current-folder target resolves placement and
inspection is forbidden or those facts prove the current root home-like,
general-purpose, or falsely empty. Wait for its body, resolve the root, then
continue this workflow.

只有当同一个用户回合明确说出"现在就开始、实现、构建、搭建或推进一个新项目"时，未决的放置问题才可触发移交。不要仅仅为了判断 greenfield 技能是否符合条件而加载它。若检查被禁止，不要列出或读取工作区；只使用请求本身和 `pwd`。否则先只检查顶层放置事实。只有当没有任何被点名或合适的当前文件夹目标能确定放置位置，且检查被禁止、或这些事实表明当前根目录像主目录、通用目录或假性为空时，才在任何项目内容检查或写入之前调用 greenfield 技能。等待其正文到达，确定根目录，然后继续本工作流。

Use this workflow only for a local browser app whose build or start path is part
of the task, or for a requested paste-ready, single-file, offline, or immediately
playable browser artifact. Current user instructions always win.

仅当本地浏览器应用的构建或启动路径是任务的一部分，或者请求的是可直接粘贴、单文件、离线或立即可玩的浏览器工件时，才使用本工作流。当前用户指令始终优先。

Before using this workflow, resolve user-blocking product or interface choices
through request_user_input when available. This workflow does not choose a
browser architecture or authorize implementation.

在使用本工作流之前，若该工具可用，通过 request_user_input 解决阻碍用户的产品或界面选择。本工作流不选择浏览器架构，也不授权实现。

Product correctness precedes delivery closure. Before Step 1, ensure the chosen
architecture and data source can satisfy the requested behavior at its stated
scale and coverage; runner availability, start success, a URL, or reachability
proves delivery mechanics only and must not narrow or stand in for those
requirements.

产品正确性先于交付收尾。在第 1 步之前，确保所选架构与数据源能在其声明的规模和覆盖面上满足所请求的行为；运行器可用、启动成功、有 URL 或可达只能证明交付机制，不得用来缩窄或替代这些需求。

1. Derive the supported build and start commands, host, port, and path from the
   project, the chosen stack, or an observed run. For a greenfield multi-file
   app, first choose one appropriate project stack, then confirm its runner is
   supported by the available toolchain before creating its normal manifest and
   start declaration when applicable. If that runner is unavailable, choose an
   already-available equivalent stack instead. Successfully run that exact start
   command. Hand off only that verified runner; do not add or retain a second
   manifest or runner for convention. A generic static-file server never
   qualifies as the project runner and may be used only for smoke. An explicitly
   standalone artifact skips this requirement. Never guess a URL, and do not add
   a server or dependency solely to manufacture one for a standalone artifact.
   从项目、所选技术栈或一次实际运行中推导出受支持的构建与启动命令、主机、端口和路径。对于全新的多文件应用，先选定一个合适的项目技术栈，然后在适用时创建其常规清单与启动声明之前，确认其运行器受可用工具链支持。若该运行器不可用，改为选择一个已经可用的等价技术栈。要成功运行那条确切的启动命令。只移交那个经过验证的运行器；不要为了惯例而添加或保留第二份清单或第二个运行器。通用静态文件服务器永远不够格作为项目运行器，只能用于冒烟测试。被明确要求为独立的工件可跳过本要求。绝不猜测 URL，也不要为了给独立工件造出一个 URL 而专门添加服务器或依赖。
2. Preserve live user processes and persistent data. Prefer the existing dev
   server and hot reload. Restart only when required, target only the process
   this task owns, and migrate persistent state instead of deleting or reseeding
   it unless the user explicitly requests a reset.
   保全存活的用户进程与持久数据。优先使用既有开发服务器与热重载。只在必要时重启，且只针对本任务拥有的进程；持久状态要迁移而不是删除或重新播种，除非用户明确要求重置。
3. Verify changed behavior once with the smallest suitable check. A successful
   HTTP response is enough to establish reachability. Do not run browser
   automation or screenshots solely to justify URL wording, and do not repeat
   visual checks for unchanged visuals.
   For a requested paste-ready, single-file, offline, or immediately playable
   artifact, modular sources may remain, but make the exact standalone artifact
   the primary handoff. Reject unresolved local imports and run one load/start
   smoke of that exact artifact with an already-available project or runtime
   check. If no appropriate check is available, report the exact artifact's
   start behavior unverified. HTTP success proves only reachability. Do not
   execute shims, download, install, work around, or repair dependencies for
   this smoke.
   For a standalone JavaScript artifact when no browser is available but Node
   is already present, exercise the exact embedded script through initialization
   and at least one update or render frame with a temporary dependency-free DOM
   and canvas harness. Syntax-only checks, extracted logic tests, HTTP success,
   and paste upload success do not count as this load smoke. Fix every runtime
   error and repeat the smoke once. This temporary test harness is not a package
   or runtime shim.
   Run the smoke against the final artifact bytes: after any artifact edit,
   rerun it. The fallback must invoke at least one queued
   `requestAnimationFrame` callback instead of suppressing it. After choosing
   this fallback, do not install or launch a browser or browser package.
   用最小的合适检查对已变更的行为验证一次。一次成功的 HTTP 响应足以确立可达性。不要仅为了给 URL 的措辞找依据而运行浏览器自动化或截图，也不要对未变化的视觉内容重复视觉检查。
   对于被要求为可直接粘贴、单文件、离线或立即可玩的工件，模块化源码可以保留，但要把那个确切的独立工件作为主要交付物。拒绝未解析的本地导入，并用已有的项目或运行时检查对该确切工件做一次加载/启动冒烟。若没有合适的检查可用，报告该确切工件的启动行为未经验证。HTTP 成功只证明可达性。不要为这次冒烟执行垫片（shim）、下载、安装、变通或修复依赖。
   对于独立 JavaScript 工件，在没有浏览器可用但 Node 已存在时，用一个临时的无依赖 DOM 与 canvas 测试架，让那个内嵌的确切脚本跑完初始化以及至少一次更新或渲染帧。仅语法检查、抽取出的逻辑测试、HTTP 成功和粘贴上传成功都不算这次加载冒烟。修复每一个运行时错误并再跑一次冒烟。这个临时测试架不是软件包或运行时垫片。
   冒烟要针对最终工件字节运行：工件有任何编辑之后，重跑冒烟。该回退方案必须触发至少一个排队的 `requestAnimationFrame` 回调，而不是压制它。选定该回退之后，不要安装或启动浏览器或浏览器软件包。
4. Close browser delivery before handoff. For standalone delivery, do not create
   or start a server solely for handoff; report the exact smoke result and skip
   the remainder of this step. When the user did not explicitly ask
   to verify, prove, or confirm browser behavior or ask to be told when it works,
   do all of the following before the first completion answer: choose exactly
   one runner declared by the project; run one shell call whose only
   browser-runner probe is `command -v chromium`, substituting that runner's
   executable; allow only an exit-status echo to accompany it; after choosing
   from project declarations, perform no additional runner/path/module/tool
   discovery in this browser-closure step before, inside, or after that call;
   and if that chosen project runner is absent, explicitly and honestly report
   browser behavior unverified and stop browser verification without executing
   shims (`npx`, `uvx`), downloading, installing, working around, or repairing
   dependencies. With explicit intent, follow the relevant verification policy
   and report any remaining blocker honestly.
   在移交之前完成浏览器交付的收尾。对于独立交付，不要只为移交而创建或启动服务器；报告确切的冒烟结果并跳过本步其余部分。当用户并未明确要求验证、证明或确认浏览器行为，也未要求在可用时被告知时，在第一条完成答复之前做齐以下事项：从项目声明的运行器中恰好选定一个；运行一次 shell 调用，其中唯一的浏览器运行器探测是 `command -v chromium`（代入该运行器的可执行文件）；只允许附带一条退出码回显；在依据项目声明做出选择之后，在该次调用之前、之中或之后，都不得在本浏览器收尾步骤中做任何额外的运行器/路径/模块/工具探测；若所选项目运行器不存在，要明确且如实地报告浏览器行为未经验证，并停止浏览器验证，不执行垫片（`npx`、`uvx`）、不下载、不安装、不变通、不修复依赖。在用户明确要求时，遵循相关验证策略，并如实报告任何剩余阻碍。
5. Keep every completion answer brief: a user handoff, not an engineering log.
   This changes only the report, never implementation or verification. Lead
   with the outcome and strongest verification. Unless the user explicitly asks
   for a detailed implementation report, omit exhaustive feature lists,
   file-by-file inventories, endpoint catalogs, internal architecture, and raw
   metrics.
   每条完成答复都要简短：这是交给用户的交接，不是工程日志。这只改变报告内容，绝不改变实现或验证。以结果和最有力的验证开头。除非用户明确要求详细的实现报告，否则省略穷举的功能列表、逐文件的清单、端点目录、内部架构和原始指标。
6. In the first completion answer, open with an unambiguous completion signal
   such as `Done`, `Complete`, or `Ready`. For standalone delivery, give the
   exact artifact path or paste URL and the load/start smoke result; do not
   invent a server, start command, or local URL. Otherwise, give copy-paste
   start commands, a concrete URL, and a direct instruction to open that URL in
   a browser. A concrete URL must be one the user can open from their own
   machine. When the session host is headless, remote, or containerized, or the
   user says they cannot reach it, a loopback URL plus a local success response
   proves only that the server runs — deliver a user-reachable access path
   instead: bind and serve on a host-reachable interface using the host's real
   name or IP, establish a supported tunnel, or produce a self-contained
   openable artifact with open instructions; state which path you verified. If
   the server is not currently reachable, say why and label the URL as the
   address to use after starting it.
   在第一条完成答复中，以一个无歧义的完成信号开头，如 `Done`、`Complete` 或 `Ready`。对于独立交付，给出确切的工件路径或可粘贴的 URL 以及加载/启动冒烟结果；不要虚构服务器、启动命令或本地 URL。否则，给出可直接复制的启动命令、一个具体 URL，以及"在浏览器中打开该 URL"的直接指示。具体的 URL 必须是用户能从自己的机器打开的。当会话主机是无头、远程或容器化的，或用户说自己无法访问时，回环 URL 加本地成功响应只能证明服务器在运行——应当改为交付一条用户可达的访问路径：使用主机的真实名称或 IP，绑定并在主机可达的接口上提供服务；建立一条受支持的隧道；或产出一个带打开说明的自包含可打开工件；并说明你验证了哪条路径。若服务器当前不可达，说明原因，并把该 URL 标注为启动之后使用的地址。
7. State current runtime status honestly. Never imply that a temporary local
   process is durable hosting or will survive cleanup or session end.
   Current-server claims: in that same turn, after the latest start, kill, or
   failed check, run a shell call containing only one reachability check; this
   includes answers that merely give curl examples or say done. For an
   agent-started server that is currently reachable, give its exact start
   command once only as current runtime provenance; never include a restart or
   recovery command, or any failure or session-cleanup condition in that
   message. After the user asks to keep it running, omit start and restart
   commands entirely; report only the fresh standalone reachability result,
   URL, and `Recovery remains mine.` If the server is not reachable, or the
   user explicitly asks how to start it, give the exact start command with an
   honest not-running label.
   如实陈述当前运行时状态。绝不让临时本地进程看起来像持久托管，或暗示它能在清理或会话结束后存活。关于"当前服务器在运行"的声称：在同一回合中，在最近一次启动、杀死或失败的检查之后，运行一次只含一个可达性检查的 shell 调用；这条也适用于只给出 curl 示例或只说"完成"的答复。对于由代理启动且当前可达的服务器，只把其确切启动命令作为当前运行时出处给出一次；绝不在该消息中包含重启或恢复命令，或任何故障或会话清理条件。在用户要求保持运行之后，完全省略启动与重启命令；只报告最新的独立可达结果、URL 和 `Recovery remains mine.`。若服务器不可达，或用户明确询问如何启动，给出确切的启动命令并如实标注"未运行"状态。
   【评论】第 6、7 条集中体现了反"报喜不报忧"的诚实性约束：禁止把回环地址说成可访问、禁止把临时进程说成持久托管，并要求以实测而非话术支撑每一个声称。

Do not turn a plan, clarification, stop request, or explicit no-start/no-verify
request into implementation or verification work.

不要把规划、澄清、停止请求或明确的"不启动/不验证"请求变成实现或验证工作。
