<!-- BILINGUAL-EN-ZH -->
---
name: workflow-authoring
description: Use when authoring a non-trivial Workflow for research, review, migration, or other multi-agent work, especially when the task needs multiple evidence sources, verification, or synthesis.
user-invocable: false
---

# Workflow Authoring / 工作流编写

Load this reference exactly once per parent session before the first non-trivial Workflow.
After a successful load, reuse that result for later Workflow authoring.
Do not call `read_skill` again after validation errors or for retries and resumes.
Select exactly one profile section from the active Workflow guidance.
Never mix symbols across profiles, and never use a call that the active
ToolSpec does not advertise.

在每个父会话中，只在第一个非平凡 Workflow 之前加载本参考一次。成功加载后，后续的 Workflow 编写复用该结果。校验出错后或重试与恢复时不要再调用 `read_skill`。从当前生效的 Workflow 指引中恰好选择一个 profile 部分。绝不在不同 profile 之间混用符号，也绝不调用当前 ToolSpec 未声明的能力。

## Shared research contract / 共享研究契约

A fixed batch count never proves completion. Scale the number and diversity of
children to the request, then stop on evidence state or an explicit caller or
runtime boundary.

固定的批处理数量永远不能证明已完成。子任务的数量与多样性应随请求规模伸缩，然后在证据状态或调用方/运行时的明确边界处停止。

For a review of a change the user has already identified, first take stock inline
of which files it touches and how large it is, then size the workflow to that
list: a small change gets a few focused children plus one verify vote, not the
full research shape.

对于用户已经指明的变更评审，先在会话内盘点它涉及哪些文件、规模多大，再据此确定工作流的规模：小变更只需要少数几个聚焦的子任务加一次校验投票，而不必套用完整的研究形态。

Discovery pointers are not inspected evidence when the relevant body is
readable. Require each research child to open every implementation or test body
it cites before `submit_result` when readable; search and grep output only
locate candidates. Back every assigned claim with inspected evidence or name
it as unresolved. Ask research children for
`complete:boolean`, `evidence:string[]`, and `unresolved:string[]`. Handle the
V2 spill marker below before checking for missing data. For inline results,
treat a profile-specific unsuccessful result envelope, missing or wrong-typed
data, `complete !== true`, or nonempty `unresolved` as incomplete.

当相关文件本体可读时，检索到的指针不等于已查验的证据。要求每个研究子任务在 `submit_result` 之前打开它引用的每一个可读的实现或测试文件；搜索与 grep 输出只能用于定位候选。每一条分派的主张都要有已查验的证据支撑，否则明确标为未决。要求研究子任务返回 `complete:boolean`、`evidence:string[]` 与 `unresolved:string[]`。在检查缺失数据之前，先按下文处理 V2 溢出标记。对内联结果而言，出现 profile 特定的失败结果信封、数据缺失或类型错误、`complete !== true`，或 `unresolved` 非空，都视为不完整。

【评论】“检索命中不算证据、必须打开被引用文件”的硬性要求，针对的是模型把搜索结果当作已验证事实的常见失败模式。

Never use data from an unsuccessful envelope as evidence or a gap disposition.
Critic unavailability is a synthesis note, never a research gap. Preserve
compact evidence, provenance refs, and every unresolved item in synthesis.
Synthesize from compact `result.data`, not from uninspected summaries. Disclose
omitted scope.

绝不把失败信封中的数据当作证据或缺口处置。评审者（critic）不可用属于综合阶段备注，绝不是研究缺口。在综合中保留精简证据、来源 ref 与每一个未决项。基于精简的 `result.data` 做综合，而不是基于未查验的摘要。披露被省略的范围。

Use independent verification when a claim has materially different failure
modes. Repeating the same prompt is not independent coverage.

当某条主张存在实质不同的失败模式时，使用相互独立的验证。重复同一提示词不构成独立覆盖。

Keep these reusable patterns when they fit the request:

当以下可复用模式适合请求时，予以采用：

- Multi-angle sweep: split initial researchers across genuinely different
  evidence surfaces, such as implementation, tests, design records, and
  operational traces; different role names do not increase coverage when they
  use the same search plan.
  多角度扫查：把初始研究者分到真正不同的证据面上，如实现、测试、设计记录与运维痕迹；若使用同一套搜索计划，不同的角色名称并不能提高覆盖。
- Adversarial verification: give a skeptic a concrete falsification target for
  each material claim; retain only claims that survive inspected counterevidence,
  and mark an unavailable or invalid verdict unresolved.
  对抗式验证：为每条重要主张给怀疑者一个具体的证伪目标；只保留经受住已查验反证的主张，把无法给出或无效的裁定标为未决。
- Judge panel: for an open solution space, generate candidates from different
  angles, score them against explicit criteria with independent judges, and
  synthesize the winner with useful runner-up ideas by provenance ref.
  A failed or invalid judge is not an affirmative vote.
  评审团：对开放的解空间，从不同角度生成候选，由相互独立的评审按明确标准打分，并按来源 ref 把胜者与有用的落选想法综合起来。失败或无效的评审员不构成赞成票。

## V2 spilled child results / V2 子任务结果溢出

Schema-valid `submit_result` custom data above 4096 canonical UTF-8 bytes
(excluding `notes`) is accepted and stored in full. Exactly 4096 bytes stays
inline. An oversize result arrives with `dataSpilledForSize: true`,
`submittedPayloadBytes`, `submittedPayloadChars`, and `ref`, with no inline
`data`. Size facts describe the full canonical submission, including non-null
`notes`; they are not the custom-only spill measurement.

超过 4096 规范化 UTF-8 字节（不含 `notes`）但符合 schema 的 `submit_result` 自定义数据会被接受并完整存储。恰好 4096 字节仍保持内联。超限结果会带有 `dataSpilledForSize: true`、`submittedPayloadBytes`、`submittedPayloadChars` 与 `ref`，且没有内联 `data`。这些大小事实描述的是包含非空 `notes` 在内的完整规范化提交，而不是仅针对自定义数据的溢出测量值。

Scripts MUST carry `dataSpilledForSize`, both size facts, and `ref` through to
their returned results. Preserve the envelope, or copy those fields explicitly
as in the V2 examples below. Never collapse a result to `data ?? null`. You
MUST NOT treat a spilled result as empty or failed, or repeat completed work
merely because inline data is absent. Continue to respect the envelope's actual
status and errors. Submission completion does not prove that uninspected
evidence is complete; preserve existing unresolved gaps.

脚本必须把 `dataSpilledForSize`、两个大小事实与 `ref` 一直传递到其返回结果中。保留信封，或按下文 V2 示例那样显式复制这些字段。绝不要把结果折叠成 `data ?? null`。绝不得把溢出结果当作空结果或失败结果，也不得仅因缺少内联数据就重复已完成的工作。继续尊重信封的实际状态与错误。提交完成并不证明未查验的证据是完整的；保留既有的未决缺口。

No script-side or parent-side API currently returns the full value. The full
submission is retained under its `ref` in durable session storage and exposed
through the MSP subagent view. Repeating a result observation or passing the
ref to another child supplies no lossless fetch; ref context is bounded.

目前没有任何脚本侧或父侧 API 能取回完整值。完整提交以其 `ref` 保存在持久会话存储中，并通过 MSP 子代理视图暴露。重复观察结果或把 ref 传给另一个子任务都无法无损取回；ref 上下文是有界的。

Ask children to keep custom data under 4096 canonical bytes. When file tools
are available, write large artifacts (test modules, reports) to files and
return paths plus a compact summary. Otherwise split the work or ask for a
compact schema. Report a spilled submission as complete but large, include its
`ref` and size facts, say where the full submission is retained, and disclose
that its contents remain uninspected by this Workflow. Do not claim a file was
written unless the child actually returned that artifact path.

要求子任务把自定义数据控制在 4096 规范化字节以内。文件工具可用时，把大型产物（测试模块、报告）写入文件，返回路径加精简摘要。否则拆分工作或要求使用精简 schema。对溢出的提交，应报告为“已完成但体量大”，附上其 `ref` 与大小事实，说明完整提交保存在何处，并披露本 Workflow 未查验其内容。除非子任务确实返回了该产物路径，否则不要声称文件已写入。

【评论】溢出机制把超大结果改为“持久存储 + 引用”的方式传递，迫使脚本显式处理“内容未查验”状态，而不是默认拿到全量数据。

## Two convergence rules / 两条收敛规则

Open-ended discovery and a known evidence gap are different jobs. Do not apply
one loop rule to both.

开放式发现与已知证据缺口是两种不同的工作。不要把同一条循环规则套在两者身上。

### Example: open discovery / 示例：开放式发现

Maintain `seen` and `dryRounds` in deterministic Workflow state. In each round,
ask complementary finders for items not already in `seen`. Add every reported
item to `seen` before judging it:

在确定性 Workflow 状态中维护 `seen` 与 `dryRounds`。每一轮让互补的发现者寻找 `seen` 中还没有的条目。在判定之前，先把每个上报条目加入 `seen`：

- deduplicate against all seen items, including rejected findings;
  对照所有已见条目去重，包括被否决的发现；
- If a round adds any fresh item, reset the dry count to zero;
  若某一轮加入任何新条目，把空转计数清零；
- if it adds none, increment the dry count; and
  若未加入任何条目，空转计数加一；且
- After two consecutive dry rounds, stop discovery.
  连续两轮空转后，停止发现。

A caller limit, capacity boundary, or runtime budget may stop it earlier; that
stop is partial unless all requested scope is covered.

调用方限制、容量边界或运行时预算可能使其更早停止；除非所请求的全部范围都已覆盖，否则该停止属于部分完成。

### Example: explicit gap follow-up / 示例：明确缺口的跟进

Track the lineage of each concrete unresolved gap.

追踪每一个具体未决缺口的谱系。

- Dispatch exactly one focused follow-up for that gap lineage.
  为该缺口谱系恰好分派一次聚焦的跟进。
- After that attempt, carry the narrowed, reworded, or still-unresolved descendant unchanged into synthesis.
  该次尝试之后，把收窄、改写或仍未解决的后续缺口原样带入综合阶段。
- Do not make its new wording look like a new gap and dispatch it again.
  不要把改写后的说法伪装成新缺口再次分派。

### Example: verification and omitted scope / 示例：验证与省略范围

For a claim involving behavior, abuse resistance, and a reported failure:

对于涉及行为、抗滥用能力与某次已报告失败的主张：

- use separate correctness, security, and reproduction lenses;
  分别使用正确性、安全性与复现三个视角；
- give each verifier a distinct falsification target;
  给每个验证者一个不同的证伪目标；
- State omitted scope for top-N, sampling, no-retry, capacity, caller limit, and runtime budget boundaries; and
  对 top-N、抽样、不重试、容量、调用方限制与运行时预算边界，声明被省略的范围；且
- never describe a bounded sample as exhaustive.
  绝不把有界样本描述成穷尽式覆盖。

## Workflow API V1 / 工作流 API V1

The API is available as bare globals - agent, parallel, pipeline, phase, log,
args, budget - and through the legacy host object. Use the V1 globals or their
`host` aliases described by the active ToolSpec. Read caller input from
`host.args`, the only advertised caller-input spelling.
For example, fan out independent research with `host.parallel(` and use
`host.agent` for the critic, one gap follow-up, and final synthesis:

该 API 以裸全局变量形式提供——agent、parallel、pipeline、phase、log、args、budget——也可通过旧式 host 对象使用。使用 V1 全局变量或当前 ToolSpec 所描述的 `host` 别名。调用方输入从 `host.args` 读取，这是唯一声明的调用方输入拼写。例如，用 `host.parallel(` 扇出独立研究，并用 `host.agent` 执行评审者、一次缺口跟进与最终综合：

```javascript
export default async function workflow(host) {
  const evidenceSchema = {
    type: "object",
    required: ["complete", "evidence", "unresolved"],
    properties: {
      complete: { type: "boolean" },
      evidence: { type: "array", items: { type: "string" } },
      unresolved: { type: "array", items: { type: "string" } },
    },
  };
  const compact = (result, scope, missing = `${scope}: missing complete evidence result`) => {
    const failed = result === null || result.error_kind;
    const data = !failed && result.data && typeof result.data === "object" ? result.data : null;
    const evidence = !failed && Array.isArray(data?.evidence) ? data.evidence.filter(Boolean) : [];
    const declared = !failed && Array.isArray(data?.unresolved) ? data.unresolved.filter(Boolean) : [];
    const complete = data?.complete === true && evidence.length > 0 && declared.length === 0;
    return {
      scope,
      ref: result?.ref ?? null,
      complete,
      evidence,
      unresolved: complete ? [] : (declared.length ? declared : [missing]),
    };
  };

  const reports = await host.parallel([
    { input: "Inspect the implementation body; return complete/evidence/unresolved.", schema: evidenceSchema },
    { input: "Inspect the tests; return complete/evidence/unresolved.", schema: evidenceSchema },
  ]);
  const compactReports = reports.map((result, index) => compact(result, `primary-${index}`));
  const synthesisNotes = [];
  const critic = await host.agent({
    input: `Find concrete gaps in this compact evidence: ${JSON.stringify(compactReports)}.`,
    schema: evidenceSchema,
  });
  const criticData = critic !== null && !critic.error_kind && critic.data && typeof critic.data === "object" ? critic.data : null;
  const criticEvidence = Array.isArray(criticData?.evidence) ? criticData.evidence.filter(Boolean) : [];
  const criticUnresolved = Array.isArray(criticData?.unresolved) ? criticData.unresolved.filter(Boolean) : [];
  const criticHasUsableDisposition = (criticData?.complete === true && criticEvidence.length > 0 && criticUnresolved.length === 0)
    || criticUnresolved.length > 0;
  const compactCritic = criticHasUsableDisposition
    ? compact(critic, "critic")
    : (synthesisNotes.push("completeness critic unavailable"), { scope: "critic", ref: critic?.ref ?? null, complete: true, evidence: [], unresolved: [] });
  const open = [...compactReports, compactCritic].flatMap((report) => report.unresolved);
  const firstGap = open[0];
  const followup = firstGap ? await host.agent({
    input: `Resolve this exact gap once, or return it unchanged: ${firstGap}`,
    schema: evidenceSchema,
  }) : null;
  const followupReport = firstGap ? compact(followup, firstGap, firstGap) : null;
  const all = followupReport ? [...compactReports, compactCritic, followupReport] : [...compactReports, compactCritic];
  const unresolved = firstGap ? [...open.slice(1), ...followupReport.unresolved] : open;
  const evidence = all.flatMap((report) => report.evidence.map((value) => ({ source: report.scope, ref: report.ref, value })));
  const refs = [...new Set(all.map((report) => report.ref).filter(Boolean))];
  const synthesis = await host.agent({
    input: `Synthesize only this compact evidence: ${JSON.stringify({ evidence, refs, unresolved, notes: synthesisNotes })}`,
    schema: evidenceSchema,
  });
  const synthesisFailed = synthesis === null || synthesis.error_kind;
  const synthesisData = !synthesisFailed && synthesis.data && typeof synthesis.data === "object" ? synthesis.data : null;
  const synthesisUnresolved = Array.isArray(synthesisData?.unresolved) ? synthesisData.unresolved.filter(Boolean) : [];
  const synthesisComplete = synthesisData?.complete === true
    && Array.isArray(synthesisData.evidence)
    && synthesisData.evidence.length > 0
    && synthesisUnresolved.length === 0;
  if (!synthesisComplete) synthesisNotes.push("synthesis unavailable or incomplete");
  return { status: unresolved.length || synthesisUnresolved.length || synthesisNotes.length > 0 ? "partial" : "complete", ref: synthesis?.ref ?? null, unresolved: [...unresolved, ...synthesisUnresolved], notes: synthesisNotes };
}
```

## Diagnostic Workflow API V2 / 诊断工作流 API V2

Keep each V2 `input` within 4096 UTF-8 bytes, including task text and refs.
Deferred commands also obey their whole-command size limit. The refs below
request bounded prior-result context; inspect needed bodies and return
`complete: false` with unresolved gaps if required evidence is unavailable.
Refs do not carry lossless `result.data` or parent-local gap state. Include
known unresolved items and synthesis notes as essential compact context.

每个 V2 `input` 保持在 4096 UTF-8 字节以内，含任务文本与 ref。延迟命令同样受其整条命令大小限制约束。下文的 ref 会请求有界的前序结果上下文；应查验所需文件本体，若所需证据不可得，返回 `complete: false` 并列出未决缺口。ref 不携带无损的 `result.data`，也不携带父侧本地缺口状态。把已知未决项与综合备注作为必要的精简上下文包含进去。

The diagnostic script surface is exactly Agent, Phase, Pipeline, ParallelGroup,
WorkflowCommandError, log, args, and budget. This profile is fresh-run and
terminal-only. Use deferred work inside the diagnostic containers for
parallelism, then read immutable results:

诊断脚本可用的接口恰好是 Agent、Phase、Pipeline、ParallelGroup、WorkflowCommandError、log、args 与 budget。该 profile 为全新运行且仅有终态。在诊断容器内部使用延迟任务实现并行，然后读取不可变结果：

```javascript
const evidenceSchema = {
  type: "object",
  required: ["complete", "evidence", "unresolved"],
  properties: {
    complete: { type: "boolean" },
    evidence: { type: "array", items: { type: "string" } },
    unresolved: { type: "array", items: { type: "string" } },
  },
};
const synthesisNotes = [];
const spilledResults = [];
const recordSpill = (result, scope) => {
  if (result?.dataSpilledForSize !== true) return false;
  spilledResults.push({
    scope, ref: result.ref, status: result.status, ok: result.ok, error: result.error,
    dataSpilledForSize: result.dataSpilledForSize,
    submittedPayloadBytes: result.submittedPayloadBytes,
    submittedPayloadChars: result.submittedPayloadChars,
  });
  synthesisNotes.push(`${scope}: large submission retained at ${result.ref} in session storage / MSP subagent view; contents uninspected.`);
  return result.status === "completed" && result.ok === true && result.error == null;
};
const group = await ParallelGroup.start({
  members: [
    Agent.defer.start({ input: "Inspect the implementation body; return complete/evidence/unresolved.", schema: evidenceSchema }),
    Agent.defer.start({ input: "Inspect the tests; return complete/evidence/unresolved.", schema: evidenceSchema }),
  ],
});
const reports = await group.result();
const fromAttemptOutcome = (outcome, scope) => {
  if (outcome?.kind !== "attempt" || !outcome.result?.ref) {
    return { scope, ref: null, complete: false, evidence: [], unresolved: [`${scope}: no completed attempt result`] };
  }
  const result = outcome.result;
  const ref = outcome.result.ref;
  if (recordSpill(result, scope)) {
    return { scope, ref: result.ref, complete: false, evidence: [], unresolved: [] };
  }
  const data = result?.data && typeof result.data === "object" ? result.data : null;
  const terminalOk = result.status === "completed" && result.ok === true && result.error == null && data !== null;
  const evidence = terminalOk && Array.isArray(data.evidence) ? data.evidence.filter(Boolean) : [];
  const declared = terminalOk && Array.isArray(data.unresolved) ? data.unresolved.filter(Boolean) : [];
  const complete = terminalOk && data.complete === true && evidence.length > 0 && declared.length === 0;
  return {
    scope,
    ref,
    complete,
    evidence: terminalOk ? evidence : [],
    unresolved: complete ? [] : (declared.length ? declared : [`${scope}: no completed attempt result`]),
  };
};
const compactReports = reports.map((outcome, index) => fromAttemptOutcome(outcome, `parallel-${index}`));
const pipeline = await Pipeline.start({
  items: reports,
  stages: [{
    title: "Check",
    run: ({ item, index }) => {
      if (item?.kind !== "attempt" || !item.result?.ref) {
        return { complete: false, evidence: [], unresolved: [`parallel-${index}: missing result ref`] };
      }
      return Agent.defer.start({ input: `Verify the evidence behind ${item.result.ref}; return complete/evidence/unresolved.`, schema: evidenceSchema });
    },
  }],
});
const checked = await pipeline.result();
const checkedReports = checked.map((item, index) => {
  if (item?.kind !== "completed" || !item.output?.ref) {
    return { scope: `pipeline-${index}`, ref: null, complete: false, evidence: [], unresolved: [`pipeline-${index}: not completed`] };
  }
  if (recordSpill(item.output, `pipeline-${index}`)) {
    return { scope: `pipeline-${index}`, ref: item.output.ref, complete: false, evidence: [], unresolved: [] };
  }
  const data = item.output.data && typeof item.output.data === "object" ? item.output.data : null;
  const outputOk = item.output.status === "completed" && item.output.ok === true && item.output.error == null && data !== null;
  const evidence = outputOk && Array.isArray(data.evidence) ? data.evidence.filter(Boolean) : [];
  const declared = outputOk && Array.isArray(data.unresolved) ? data.unresolved.filter(Boolean) : [];
  const complete = outputOk && data.complete === true && evidence.length > 0 && declared.length === 0;
  return {
    scope: `pipeline-${index}`,
    ref: item.output.ref,
    complete,
    evidence: outputOk ? evidence : [],
    unresolved: complete ? [] : (declared.length ? declared : [`pipeline-${index}: missing structured output`]),
  };
});
const unresolved = [...compactReports, ...checkedReports].flatMap((report) => report.unresolved);
const refs = [...compactReports, ...checkedReports].map((report) => report.ref).filter(Boolean);
const evidence = [...compactReports, ...checkedReports].flatMap((report) => report.evidence.map((value) => ({ source: report.scope, ref: report.ref, value })));
const critic = await Agent.start({ input: `Inspect relevant bodies and find concrete gaps in reports ${refs.join(" ")}. Known unresolved items: ${JSON.stringify(unresolved)}. Return complete/evidence/unresolved; set complete:false and name unresolved gaps when evidence is unavailable.`, schema: evidenceSchema });
const gaps = await critic.latestAttempt.result();
const criticSpilled = recordSpill(gaps, "critic");
const criticData = gaps?.data && typeof gaps.data === "object" ? gaps.data : null;
const criticTerminalOk = gaps?.status === "completed"
  && gaps?.ok === true
  && gaps?.error == null
  && criticData !== null;
const criticGaps = criticTerminalOk && Array.isArray(criticData.unresolved) ? criticData.unresolved.filter(Boolean) : [];
if (criticTerminalOk && Array.isArray(criticData.evidence)) {
  evidence.push(...criticData.evidence.filter(Boolean).map((value) => ({ source: "critic", ref: gaps.ref, value })));
}
const criticComplete = criticTerminalOk
  && criticData?.complete === true
  && Array.isArray(criticData.evidence)
  && criticData.evidence.length > 0
  && Array.isArray(criticData.unresolved)
  && criticData.unresolved.length === 0;
if (!criticComplete && criticGaps.length === 0 && !criticSpilled) {
  synthesisNotes.push("completeness critic unavailable");
}
const open = [...unresolved, ...criticGaps];
const firstGap = open[0];
let gapResult = null;
let gapDescendants = [];
if (firstGap) {
  const resolver = await Agent.start({ input: `Resolve this exact gap once, or return it unchanged: ${firstGap}`, schema: evidenceSchema });
  gapResult = await resolver.latestAttempt.result();
  recordSpill(gapResult, firstGap);
  const gapData = gapResult?.data && typeof gapResult.data === "object" ? gapResult.data : null;
  const gapTerminalOk = gapResult?.status === "completed"
    && gapResult?.ok === true
    && gapResult?.error == null
    && gapData !== null;
  const reportedDescendants = gapTerminalOk && Array.isArray(gapData.unresolved) ? gapData.unresolved.filter(Boolean) : [];
  if (gapTerminalOk && Array.isArray(gapData.evidence)) {
    evidence.push(...gapData.evidence.filter(Boolean).map((value) => ({ source: firstGap, ref: gapResult.ref, value })));
  }
  const gapComplete = gapTerminalOk
    && gapData?.complete === true
    && Array.isArray(gapData.evidence)
    && gapData.evidence.length > 0
    && reportedDescendants.length === 0;
  if (!gapComplete) gapDescendants = reportedDescendants.length > 0 ? reportedDescendants : [firstGap];
}
const finalUnresolved = firstGap ? [...open.slice(1), ...gapDescendants] : open;
const synthesisRefs = [...refs, gaps?.ref, gapResult?.ref].filter(Boolean);
const synthesisAgent = await Agent.start({
  input: `Synthesize reports ${synthesisRefs.join(" ")} using inspected bodies only. Preserve this known state: ${JSON.stringify({ unresolved: finalUnresolved, notes: synthesisNotes })}. Keep unavailable-critic notes separate from research gaps. Inspect relevant bodies as needed; preserve unresolved items and omitted scope. Set complete:false when needed evidence is unavailable.`,
  schema: evidenceSchema,
});
const synthesis = await synthesisAgent.latestAttempt.result();
const synthesisSpilled = recordSpill(synthesis, "synthesis");
const synthesisData = synthesis?.data && typeof synthesis.data === "object" ? synthesis.data : null;
const synthesisTerminalOk = synthesis?.status === "completed"
  && synthesis?.ok === true
  && synthesis?.error == null
  && synthesisData !== null;
const synthesisGaps = synthesisTerminalOk && Array.isArray(synthesisData.unresolved) ? synthesisData.unresolved.filter(Boolean) : [];
const synthesisComplete = synthesisTerminalOk
  && synthesisData?.complete === true
  && Array.isArray(synthesisData.evidence)
  && synthesisData.evidence.length > 0
  && synthesisGaps.length === 0;
if (!synthesisComplete && !synthesisSpilled) synthesisNotes.push("synthesis unavailable or incomplete");
return { status: finalUnresolved.length || synthesisGaps.length || synthesisNotes.length > 0 ? "partial" : "complete", ref: synthesis?.ref ?? null, reports: synthesisRefs, spilledResults, unresolved: [...finalUnresolved, ...synthesisGaps], notes: synthesisNotes };
```

Wrap the shared discovery and gap-lineage state machine around these calls when
needed. Do not invent durable control or recovery methods in this profile.

需要时在这些调用外层包上共享的发现与缺口谱系状态机。在该 profile 中不要发明持久化的控制或恢复方法。

## Live Workflow API V2 / 实时工作流 API V2

Budget the complete caller-supplied `input` or `message` string to at most
4096 UTF-8 bytes before runtime-added prior-result context. This applies to
`Agent.start`, `agent.followup`, and `agent.send`. Count task text, JSON syntax,
evidence, refs, and unresolved items together; JavaScript `string.length` is
not a UTF-8 byte count.

调用方提供的完整 `input` 或 `message` 字符串，在运行时追加前序结果上下文之前，预算不超过 4096 UTF-8 字节。这适用于 `Agent.start`、`agent.followup` 与 `agent.send`。任务文本、JSON 语法、证据、ref 与未决项一并计数；JavaScript 的 `string.length` 并不是 UTF-8 字节数。

【评论】提醒 `string.length` 统计的是 UTF-16 码元而非 UTF-8 字节数，中文等多字节文本尤其容易因此超出限制。

Build critic and synthesis handoffs as a short task and accessible `result.ref`
tokens plus only essential compact context. The runtime adds bounded context
for those refs; children must inspect needed bodies and return `complete: false`
with unresolved gaps when required evidence is unavailable.
Refs do not carry lossless `result.data` or parent-local gap state. Include
known unresolved items and synthesis notes as essential compact context.
Keep every unresolved item unchanged in workflow state and in the final outcome.
If essential context will not fit, split work within the remaining budget or
disclose the omitted scope; do not truncate JSON or silently drop gap lists.

把评审者与综合的交接构建为简短任务加可访问的 `result.ref` 令牌，外加仅必要的精简上下文。运行时会为这些 ref 追加有界上下文；子任务必须查验所需文件本体，在所需证据不可得时返回 `complete: false` 并列出未决缺口。ref 不携带无损的 `result.data`，也不携带父侧本地缺口状态。把已知未决项与综合备注作为必要的精简上下文包含进去。每个未决项在工作流状态与最终结果中都保持原样。若必要的上下文放不下，在剩余预算内拆分工作或披露被省略的范围；不要截断 JSON，也不要悄悄丢弃缺口列表。

The live Workflow API V2 surface in this activation slice is the Agent and
AgentAttempt path. Start each independent worker, then observe the exact
attempt. Use a follow-up only for an explicit gap lineage that has not already
received one:

本激活切片中实时 Workflow API V2 的接口是 Agent 与 AgentAttempt 路径。启动每个独立的工作进程，然后观察确切的尝试。仅对尚未获得过跟进的明确缺口谱系使用一次跟进：

```javascript
const evidenceSchema = {
  type: "object",
  required: ["complete", "evidence", "unresolved"],
  properties: {
    complete: { type: "boolean" },
    evidence: { type: "array", items: { type: "string" } },
    unresolved: { type: "array", items: { type: "string" } },
  },
};
const synthesisNotes = [];
const spilledResults = [];
const recordSpill = (result, scope) => {
  if (result?.dataSpilledForSize !== true) return false;
  spilledResults.push({
    scope, ref: result.ref, status: result.status, ok: result.ok, error: result.error,
    dataSpilledForSize: result.dataSpilledForSize,
    submittedPayloadBytes: result.submittedPayloadBytes,
    submittedPayloadChars: result.submittedPayloadChars,
  });
  synthesisNotes.push(`${scope}: large submission retained at ${result.ref} in session storage / MSP subagent view; contents uninspected.`);
  return result.status === "completed" && result.ok === true && result.error == null;
};
const agent = await Agent.start({ input: "Inspect the implementation body; return complete/evidence/unresolved.", schema: evidenceSchema });
const tests = await Agent.start({ input: "Inspect the tests; return complete/evidence/unresolved.", schema: evidenceSchema });
const workers = [agent, tests];
const reports = await Promise.all([
  agent.latestAttempt.result(),
  tests.latestAttempt.result(),
]);
const compact = (result, scope) => {
  if (recordSpill(result, scope)) {
    return { scope, ref: result.ref, complete: false, evidence: [], unresolved: [] };
  }
  const data = result?.data && typeof result.data === "object" ? result.data : null;
  const terminalOk = result.status === "completed" && result.ok === true && result.error == null && data !== null;
  const evidence = terminalOk && Array.isArray(data.evidence) ? data.evidence.filter(Boolean) : [];
  const declared = terminalOk && Array.isArray(data.unresolved) ? data.unresolved.filter(Boolean) : [];
  const complete = terminalOk && data.complete === true && evidence.length > 0 && declared.length === 0;
  return {
    scope,
    ref: result?.ref ?? null,
    complete,
    evidence: terminalOk ? evidence : [],
    unresolved: complete ? [] : (declared.length ? declared : [`${scope}: missing complete evidence result`]),
  };
};
const compactReports = reports.map((result, index) => compact(result, `primary-${index}`));
const primaryRefs = compactReports.map((report) => report.ref).filter(Boolean);
const evidence = compactReports.flatMap((report) => report.evidence.map((value) => ({ source: report.scope, ref: report.ref, value })));
const primaryGaps = compactReports.flatMap((report) => report.unresolved);
const critic = await Agent.start({
  input: `Inspect relevant bodies and find concrete gaps in reports ${primaryRefs.join(" ")}. Known unresolved items: ${JSON.stringify(primaryGaps)}. Return complete/evidence/unresolved; set complete:false and name unresolved gaps when evidence is unavailable.`,
  schema: evidenceSchema,
});
const criticResult = await critic.latestAttempt.result();
const criticSpilled = recordSpill(criticResult, "critic");
const criticData = criticResult?.data && typeof criticResult.data === "object" ? criticResult.data : null;
const criticTerminalOk = criticResult?.status === "completed"
  && criticResult?.ok === true
  && criticResult?.error == null
  && criticData !== null;
const criticGaps = criticTerminalOk && Array.isArray(criticData.unresolved) ? criticData.unresolved.filter(Boolean) : [];
if (criticTerminalOk && Array.isArray(criticData.evidence)) {
  evidence.push(...criticData.evidence.filter(Boolean).map((value) => ({ source: "critic", ref: criticResult.ref, value })));
}
const criticComplete = criticTerminalOk
  && criticData?.complete === true
  && Array.isArray(criticData.evidence)
  && criticData.evidence.length > 0
  && criticGaps.length === 0;
if (!criticComplete && criticGaps.length === 0 && !criticSpilled) {
  synthesisNotes.push("completeness critic unavailable");
}
const open = [...primaryGaps, ...criticGaps];
const gapOwner = compactReports.findIndex((report) => report.unresolved.length > 0);
const firstGap = gapOwner >= 0 ? compactReports[gapOwner].unresolved[0] : (criticGaps[0] ?? null);
const gapAgent = gapOwner >= 0 ? workers[gapOwner] : critic;
const gapDescendants = [];
let followupRef = null;
if (firstGap) {
  const followupAttempt = await gapAgent.followup({ input: `Resolve this exact gap once, or return it unchanged: ${firstGap}` });
  const followupResult = await followupAttempt.result();
  recordSpill(followupResult, firstGap);
  followupRef = followupResult?.ref ?? null;
  const data = followupResult?.data && typeof followupResult.data === "object" ? followupResult.data : null;
  const followupTerminalOk = followupResult?.status === "completed"
    && followupResult?.ok === true
    && followupResult?.error == null
    && data !== null;
  const reportedDescendants = followupTerminalOk && Array.isArray(data.unresolved) ? data.unresolved.filter(Boolean) : [];
  if (followupTerminalOk && Array.isArray(data.evidence)) {
    evidence.push(...data.evidence.filter(Boolean).map((value) => ({ source: firstGap, ref: followupResult.ref, value })));
  }
  const descendants = reportedDescendants.length > 0 ? reportedDescendants : [firstGap];
  const followupComplete = followupTerminalOk
    && data?.complete === true
    && Array.isArray(data.evidence)
    && data.evidence.length > 0
    && reportedDescendants.length === 0;
  if (!followupComplete) gapDescendants.push(...descendants);
}
const unresolved = firstGap ? [...open.slice(1), ...gapDescendants] : open;
const synthesisRefs = [...primaryRefs, criticResult?.ref, followupRef].filter(Boolean);
const synthesis = await Agent.start({
  input: `Synthesize reports ${synthesisRefs.join(" ")} using inspected bodies only. Preserve this known state: ${JSON.stringify({ unresolved, notes: synthesisNotes })}. Keep unavailable-critic notes separate from research gaps. Inspect relevant bodies as needed; preserve unresolved items and omitted scope. Set complete:false when needed evidence is unavailable.`,
  schema: evidenceSchema,
});
const final = await synthesis.latestAttempt.result();
const finalSpilled = recordSpill(final, "synthesis");
const finalData = final?.data && typeof final.data === "object" ? final.data : null;
const finalTerminalOk = final?.status === "completed"
  && final?.ok === true
  && final?.error == null
  && finalData !== null;
const finalUnresolved = finalTerminalOk && Array.isArray(finalData.unresolved) ? finalData.unresolved.filter(Boolean) : [];
const finalOk = finalTerminalOk
  && finalData?.complete === true
  && Array.isArray(finalData.evidence)
  && finalData.evidence.length > 0
  && Array.isArray(finalData.unresolved)
  && finalData.unresolved.length === 0;
if (!finalOk && !finalSpilled) synthesisNotes.push("synthesis unavailable or incomplete");
return { status: unresolved.length || finalUnresolved.length || synthesisNotes.length > 0 || !finalOk ? "partial" : "complete", ref: final?.ref ?? null, spilledResults, unresolved: [...unresolved, ...finalUnresolved], notes: synthesisNotes };
```

Check `dataSpilledForSize` first. For inline results, read structured child
data from `result.data`, including `data.unresolved`; the top-level result
object is only the envelope. To stop a whole launched run
from the parent conversation, call `work_stop` with `work_id` set to the
`workId` from the launch result when that tool is available. `interrupt()` stops
only one live child attempt: check `agent.latestAttempt.getStatus()` and call
`agent.latestAttempt.interrupt()` before awaiting
`agent.latestAttempt.result()`. After `result()` resolves, the attempt is
terminal.

先检查 `dataSpilledForSize`。对内联结果，从 `result.data` 读取子任务的结构化数据，包括 `data.unresolved`；顶层结果对象只是信封。要从父会话停止整个已启动的运行，在该工具可用时调用 `work_stop`，把 `work_id` 设为启动结果中的 `workId`。`interrupt()` 只停止一个存活的子任务尝试：先检查 `agent.latestAttempt.getStatus()` 并调用 `agent.latestAttempt.interrupt()`，然后再等待 `agent.latestAttempt.result()`。`result()` 决议完成后，该尝试即进入终态。

For later-owner recovery, invoke the Workflow tool with the returned
`scriptPath` and `resumeFromRunId`; do not add those fields to the script API.
Wrap the shared discovery and gap-lineage state machine around the Agent calls,
and preserve unresolved descendants unchanged at the final boundary.

后续所有者恢复时，用返回的 `scriptPath` 与 `resumeFromRunId` 调用 Workflow 工具；不要把这些字段加进脚本 API。在 Agent 调用外层包上共享的发现与缺口谱系状态机，并在最终边界处让未决的后续缺口保持原样。
