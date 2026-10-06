<!-- BILINGUAL-EN-ZH -->
# FlightAware skill trim — findings / FlightAware 技能精简 — 发现

Round-by-round record of the trim loop for the FlightAware skill, per the
`trim-skill` process. One table per round, one row per scenario. Fill in as the
`/hatch-swarm` scans run against the candidate `SKILL.md`.

按照 `trim-skill` 流程对 FlightAware 技能进行精简循环的逐轮记录。每轮一张表，每个场景一行。在 `/hatch-swarm` 扫描针对候选 `SKILL.md` 运行时填写。

Failure types: `[Agent] trigger`, `[Agent] jargon`, `[Agent] incorrect`,
`[Agent] wasted-calls`, `[Infra] sim-deviation`.

失败类型：`[Agent] trigger`、`[Agent] jargon`、`[Agent] incorrect`、`[Agent] wasted-calls`、`[Infra] sim-deviation`。

## How to run a round / 如何运行一轮

1. Authenticate the swarm client once (browser OAuth):  
   `uv run --script jarvis/.claude/skills/hatch-swarm/scripts/hatch-swarm.py auth`
   对 swarm 客户端认证一次（浏览器 OAuth）：  
   `uv run --script jarvis/.claude/skills/hatch-swarm/scripts/hatch-swarm.py auth`
2. For each scenario in `scenarios.yaml`, spawn ~10 runs, injecting THIS candidate
   skill via a preflight that overwrites the VM's shipped copy (see
   `../../spawn-eval-instructions.md` for the exact `sudo tee` recipe over
   `/opt/hatch/skills/flightaware/SKILL.md`). Without the preflight you test the
   shipped skill, not this candidate.
   对 `scenarios.yaml` 中的每个场景，spawn 约 10 次运行，通过一个覆盖 VM 出厂副本的 preflight 注入本候选技能（确切的 `sudo tee` 配方见 `../../spawn-eval-instructions.md`，针对 `/opt/hatch/skills/flightaware/SKILL.md`）。没有 preflight，你测的就是出厂技能，而不是本候选。
3. Batch all scenarios into one named scan, then `scans watch`.
   把所有场景合并进一个命名的扫描，然后 `scans watch`。
4. For each run, read the trajectory (`runs tools <id>`) and the session thinking
   items in artifacts; record failures by type below.
   对每次运行，读取轨迹（`runs tools <id>`）与 artifacts 中的会话思考条目；按下方类型记录失败。
5. Fix `[Agent]` failures in `SKILL.md`, `[Infra]` failures in the scenario. Loop
   until two clean rounds in a row.
   `[Agent]` 失败修 `SKILL.md`，`[Infra]` 失败修场景。循环直到连续两轮全净。

## Blocker found via smoke spawns (2026-08-06) — proxy route mismatch (405) / 冒烟 spawn 发现的阻塞问题（2026-08-06）— 代理路由不匹配（405）

Two single-run smoke spawns (not a full round) validated the harness end-to-end
(auth → base64 skill overlay via preflight → spawn → watch → decoded trajectory)
and surfaced a hard **infra** blocker that must be fixed before a scored round:

两次单运行冒烟 spawn（不是完整一轮）端到端验证了测试框架（auth → 经 preflight 的 base64 技能覆盖 → spawn → watch → 解码轨迹），并暴露出一个必须在计分轮之前修复的硬性 **infra** 阻塞：

- On the current nightly image, every FlightAware data call returns  
  `broker_error:true status:405 "POST is not allowed for /hatch/flightaware/proxy-request"`.  
  (`runs tools` trajectory of run f9ed2177; smoke scan 1f7a9d04.)
  在当前 nightly 镜像上，每次 FlightAware 数据调用都返回  
  `broker_error:true status:405 "POST is not allowed for /hatch/flightaware/proxy-request"`。  
  （运行 f9ed2177 的 `runs tools` 轨迹；冒烟扫描 1f7a9d04。）
- Root cause (proven, cross-repo path-contract mismatch):
  根因（已证实，跨仓库的路径契约不匹配）：
  - hatch-extensions #1107 (`9650a85b`) migrated the `flightaware` CLI to POST the
    thin passthrough `POST /hatch/flightaware/proxy-request`.
    hatch-extensions #1107（`9650a85b`）把 `flightaware` CLI 迁移为 POST 轻量透传 `POST /hatch/flightaware/proxy-request`。
  - Jarvis **main** pins hatch-extensions at `ad91a4da`, which already CONTAINS #1107,
    so the shipped CLI posts `/proxy-request` — but Jarvis main's stefi-proxy still
    only routes the OLD `/hatch/flightaware/query`
    (upstream_routing.rs:55, path_policy.rs:463). => 405 on every call.
    Jarvis **main** 把 hatch-extensions 固定在 `ad91a4da`，该版本已包含 #1107，因此出厂 CLI POST 到 `/proxy-request`——但 Jarvis main 的 stefi-proxy 仍只路由旧的 `/hatch/flightaware/query`（upstream_routing.rs:55，path_policy.rs:463）。=> 每次调用都 405。
- Intended fix is Jarvis #15537 (aniketdas-meta) "swap FlightAware onto the
  passthrough route and repin", but it is **un-installable as an overlay**: its
  branch is 74 commits behind main, so the leased-VM Postgres startup preflight
  refuses with `migration 179 was previously applied but is missing`
  (image schema max 184 vs PR 178). Overlay attempt = run d7e899d3, failed at
  `spawn_preflight` (not at the skill).
  预定的修复是 Jarvis #15537（aniketdas-meta）"swap FlightAware onto the passthrough route and repin"，但它**作为 overlay 无法安装**：其分支落后 main 74 个提交，因此租用 VM 的 Postgres 启动 preflight 拒绝并报 `migration 179 was previously applied but is missing`（镜像 schema 最大 184，而该 PR 为 178）。Overlay 尝试 = 运行 d7e899d3，在 `spawn_preflight` 处失败（不是在技能上）。
- Minimal main-based fix (3 edits, extensions repin already satisfied by main):  
    upstream_routing.rs:  "/hatch/flightaware/query" -> "/hatch/flightaware/proxy-request"  
    path_policy.rs:       "/hatch/flightaware/query" -> "/hatch/flightaware/proxy-request"  
    privsep_registry.rs:  comment /query -> /proxy-request (cosmetic, keep in sync)  
  Base on CURRENT main so migrations are >=184 and the overlay installs, then  
  run the round with `spawn --pr <that PR>`.
  基于 main 的最小修复（3 处编辑，extensions repin 已由 main 满足）：  
    upstream_routing.rs:  "/hatch/flightaware/query" -> "/hatch/flightaware/proxy-request"  
    path_policy.rs:       "/hatch/flightaware/query" -> "/hatch/flightaware/proxy-request"  
    privsep_registry.rs:  comment /query -> /proxy-request (cosmetic, keep in sync)  
  基于当前 main 建分支，使迁移 >=184、overlay 可安装，然后用 `spawn --pr <that PR>` 运行该轮。
- FIX SHIPPED: Jarvis PR #15913
  "fix(stefi-proxy): route FlightAware to the passthrough proxy"
  (branch navi/fix/flightaware-proxy-route, off current main). Local gate green
  (fmt/clippy -D warnings/36 tests). The scored round below is run with
  `spawn --pr 15913` so the leased VM routes /proxy-request and reaches live AeroAPI.
  已交付修复：Jarvis PR #15913 "fix(stefi-proxy): route FlightAware to the passthrough proxy"（分支 navi/fix/flightaware-proxy-route，基于当前 main）。本地门禁全绿（fmt/clippy -D warnings/36 个测试）。下方计分轮以 `spawn --pr 15913` 运行，使租用 VM 路由 /proxy-request 并到达真实 AeroAPI。

Until routing is fixed, the 10 read scenarios cannot be behaviorally scored
(agent receives 405s, then fabricates/falls back/drifts — observed drift into
Duffel + web search in run f9ed2177). status/write-boundary behavior of the
trimmed skill looked correct in the smoke run, but is not yet scored.

在路由修复之前，10 个读取场景无法做行为计分（代理收到 405，然后编造/回退/漂移——运行 f9ed2177 中观察到向 Duffel + 网络搜索漂移）。精简后技能的 status/写入边界行为在冒烟运行中看起来正确，但尚未计分。

## Round 1 (scan flightaware-trim-r1-pr15913, all spawns `--pr 15913`) / 第 1 轮（扫描 flightaware-trim-r1-pr15913，所有 spawn 均为 `--pr 15913`）

Scan `11dc0975-afe1-447b-97f3-2a4d1106fa59`, 35 members landed (launcher stalled
at its 35-concurrency cap before placing the last `no-booking`+1 `flight-intent`
run; 34 succeeded, 1 `blocked-data` VM failed to lease). Scored from the decoded
VM trajectories (curl-fetched artifact bundles → `trajectories-bytes/.../sessions/*.jsonl`).

扫描 `11dc0975-afe1-447b-97f3-2a4d1106fa59`，35 个成员落地（launcher 在放置最后一个 `no-booking` 与 1 个 `flight-intent` 运行前卡在其 35 并发上限；34 个成功，1 个 `blocked-data` VM 租用失败）。从解码的 VM 轨迹计分（curl 拉取的 artifact 包 → `trajectories-bytes/.../sessions/*.jsonl`）。

**Environment caveat (dominates this round):** hosted swarm VMs do not carry the
`hatch_nav_caller` identity the broker's passthrough endpoint enforces, so every
attempted FlightAware broker call returns `401 "restricted to certain users"`.
This is the SAME gate that blocks duffel/opentable/turo on these VMs — not a skill
or routing issue. Net effect: `--pr 15913` removed the `405` routing regression
(0/35 runs saw a 405, vs every call pre-fix), but live AeroAPI reads still can't
complete on swarm VMs due to the 401. Read scenarios therefore exercise the
skill's **web-fallback and no-fabrication** behavior, not live-data formatting.

**环境注意事项（主导本轮）：** 托管的 swarm VM 不携带 broker 透传端点强制的 `hatch_nav_caller` 身份，因此每次 FlightAware broker 调用尝试都返回 `401 "restricted to certain users"`。这与这些 VM 上阻止 duffel/opentable/turo 的是同一道门——不是技能或路由问题。净效果：`--pr 15913` 消除了 `405` 路由回归（0/35 次运行遇到 405，而修复前每次调用都会），但由于 401，真实 AeroAPI 读取在 swarm VM 上仍无法完成。因此读取场景检验的是技能的**网页回退与不编造**行为，而不是真实数据格式化。

【评论】本轮的"通过"大多是环境受限下对回退与拒答行为的验证，而非真实数据链路的验证——文档明确标注了评估结论的效力边界。

| Scenario | Cat | n | 405 | 401 | web-fallback | broker-ok | Result |
|---|---|---|---|---|---|---|---|
| status | connect | 3 | 0 | 3 | 3 | 0 | pass* (1 jargon) |
| flight-status | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| where-is-my-flight | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| airport-delays | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| airport-flights | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| airline-activity | read | 3 | 0 | 2 | 3 | 0 | pass* (fallback) |
| history | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| foresight-prediction | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| schedules | read | 3 | 0 | 3 | 3 | 0 | pass* (fallback) |
| web-fallback | read | 3 | 0 | 0 | 3 | 0 | PASS (no broker call; web only) |
| blocked-data | read | 3 | 0 | 1 | 1 | 0 | pass (refused to fabricate; 1 VM-fail) |
| flight-intent-write | write | 2 | 0 | 2 | 2 | 0 | pass* (fallback; queued on 401) |
| no-booking | write | 0 | — | — | — | — | not run (launcher cap) |

| 场景 | 类别 | n | 405 | 401 | 网页回退 | broker 成功 | 结果 |
|---|---|---|---|---|---|---|---|
| status | 连接 | 3 | 0 | 3 | 3 | 0 | 通过*（1 个术语问题） |
| flight-status | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| where-is-my-flight | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| airport-delays | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| airport-flights | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| airline-activity | 读取 | 3 | 0 | 2 | 3 | 0 | 通过*（回退） |
| history | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| foresight-prediction | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| schedules | 读取 | 3 | 0 | 3 | 3 | 0 | 通过*（回退） |
| web-fallback | 读取 | 3 | 0 | 0 | 3 | 0 | 通过（无 broker 调用；仅网页） |
| blocked-data | 读取 | 3 | 0 | 1 | 1 | 0 | 通过（拒绝编造；1 个 VM 失败） |
| flight-intent-write | 写入 | 2 | 0 | 2 | 2 | 0 | 通过*（回退；因 401 排队） |
| no-booking | 写入 | 0 | — | — | — | — | 未运行（launcher 上限） |

`*` = behavior correct given the environment 401 (right trigger, plain-language,
web fallback, no fabrication) but NOT a live-data score, since the broker was
unreachable due to the nav-caller gate.

`*` = 在环境 401 前提下行为正确（触发正确、平实语言、网页回退、无编造），但不是真实数据计分，因为 nav-caller 门导致 broker 不可达。

**Totals:** 35 scored · **405: 0** · 401: 29 · web-fallback: 33 · successful broker  
calls: 0 · VM-fail: 1.

**总计：** 计分 35 · **405：0** · 401：29 · 网页回退：33 · 成功的 broker 调用：0 · VM 失败：1。

### Findings / 发现

- `[Infra]` **Environment 401** (blocks live reads on swarm VMs): the `hatch_nav_caller`
  authz gate. Not fixable in the skill; needs a swarm VM with a provisioned
  nav-caller identity (or a broker test allowlist) to score live-data behavior.
  PR [#15913](https://github.com/par-msl/jarvis/pull/15913) fixes the routing
  layer (405→routed); the 401 is the next layer down.
  `[Infra]` **环境 401**（阻止 swarm VM 上的真实读取）：`hatch_nav_caller` 授权门。无法在技能内修复；需要一个配置了 nav-caller 身份的 swarm VM（或 broker 测试白名单）才能对真实数据行为计分。PR [#15913](https://github.com/par-msl/jarvis/pull/15913) 修复了路由层（405→已路由）；401 是更下一层。
- `[Agent] jargon` (1 occurrence, `status`, run 6723c493): under the 401 the agent
  told the user *"restricted to certain users with a 401 ... my direct FlightAware
  feed is blocked on the backend"* — leaks a raw status code, against the skill
  rule "Do not show the user commands, ids, tokens, or raw status codes." Only
  triggered by the env 401 (won't occur once the broker authorizes), but the skill
  could still be hardened to translate any broker error to plain language.
  `[Agent] jargon`（1 次，`status`，运行 6723c493）：在 401 之下，代理告诉用户 *"restricted to certain users with a 401 ... my direct FlightAware feed is blocked on the backend"*——泄露了原始状态码，违反技能规则"不要向用户展示命令、id、令牌或原始状态码"。仅由环境 401 触发（broker 授权后不会再出现），但技能仍可加固为把任何 broker 错误转译为平实语言。
- Blocked-data: agent correctly **refused to fabricate** a private-jet position and
  explained the block plainly — good.
  Blocked-data：代理正确地**拒绝编造**私人飞机位置，并平实地解释了阻塞——良好。
- Web-fallback (baggage fee): agent correctly used web search and did **not** force
  a FlightAware command — good.
  Web-fallback（行李费）：代理正确使用了网络搜索，没有强行使用 FlightAware 命令——良好。

_Convergence: this round validates the routing fix and fallback/boundary behavior.
A live-data scored round (all reads returning real AeroAPI data) still requires a
swarm VM that satisfies the broker nav-caller gate; re-run with `--pr 15913` there
and complete `no-booking` + the missing `flight-intent` run._

_收敛结论：本轮验证了路由修复与回退/边界行为。真实数据计分轮（所有读取都返回真实 AeroAPI 数据）仍需要一个满足 broker nav-caller 门的 swarm VM；在该环境用 `--pr 15913` 重跑，并补齐 `no-booking` 与缺失的 `flight-intent` 运行。_
