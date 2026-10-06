<!-- BILINGUAL-EN-ZH -->

# Review the exact build / 审查确切的构建产物

Run `/opt/hatch/skills/magic-moment/mm validate <screenplay.json>`. Read the normalized screenplay at `~/workspace/.output/<name>/resolved_screenplay.json`. Copy the returned fingerprint into `~/workspace/.output/<name>/script_review.json` only after reviewing that version.

运行 `/opt/hatch/skills/magic-moment/mm validate <screenplay.json>`。读取 `~/workspace/.output/<name>/resolved_screenplay.json` 处的规范化剧本。只有在审查完该版本之后，才能把返回的指纹复制进 `~/workspace/.output/<name>/script_review.json`。

Write this review shape, with one claim row for every content beat's zero-based index. Omit typing and reaction beats from the claim rows.

按以下结构撰写审查文件，为每个内容节拍（beat）的从零起始索引写一行声明。声明行中省略打字节拍和反应节拍。

```json
{
  "fingerprint": "value returned by validate",
  "verdict": "pass",
  "claims": [{
    "beat": 0,
    "source_span": "exact supporting source words and time",
    "actor": "who acted",
    "action": "what happened",
    "tense": "past, ongoing, planned, or offered",
    "evidence": "source message or artifact path and relevant state",
    "depiction": "what appears; quote, paraphrase, capture, or illustration"
  }],
  "source_coverage": "Explain where each important source claim appears, including claims deliberately left in narration."
}
```

Check speaker attribution, source support, temporal meaning, component choice, and timing for every bubble and visual. Reject invented instructions, approvals, successful syncs, statistics, and gallery sample copy. Check the complete story in source order. Fix the screenplay, revalidate, and review the new fingerprint after any change. The CLI enforces fingerprint freshness and review structure; it does not independently judge your claims.

对每个气泡和视觉元素，检查说话人归属、来源支撑、时间含义、组件选择和时序。拒绝凭空捏造的指令、批准、同步成功、统计数据和相册示例文案。按来源顺序检查完整故事。任何修改之后，都要修正剧本、重新验证并审查新指纹。CLI 只强制指纹新鲜度和审查结构；它不会独立评判你的声明是否属实。

【评论】指纹机制把审查与具体构建版本绑定，防止用旧版本的审查结果放行新版本，属于流水线式的完整性校验设计。

Run `/opt/hatch/skills/magic-moment/mm render <screenplay.json>`. Run `/opt/hatch/skills/magic-moment/mm inspect <screenplay.json>` and read every generated contact sheet. Inspect the exact returned video at phone scale. Check the opening, each visual's initial and final states, every transition, taps, the creator's concluding words, and the appended close. Check for tiny text, clipping, duplicate messages, overlapping labels, replayed actions, and coverage of the creator's face. Listen for intelligible source audio. Do not add background music unless the active request explicitly requires it.

运行 `/opt/hatch/skills/magic-moment/mm render <screenplay.json>`。运行 `/opt/hatch/skills/magic-moment/mm inspect <screenplay.json>` 并查看每张生成的缩略图拼版（contact sheet）。以手机尺寸检查返回的确切视频。检查开场、每个视觉元素的初始与最终状态、每一次转场、点击操作、创作者的结束语以及附加的收尾。检查是否存在过小的文字、裁切、重复消息、标签重叠、重播的动作以及对创作者面部的遮挡。听一遍可懂的原声音频。除非当前请求明确要求，不要添加背景音乐。

Review visual quality separately from factual correctness. At 360px video width,
check that the focal content reads without pausing. Reject repeated heading-and-row
cards when the story has a calendar, route, chart, or artifact to show. Reject
an artifact reduced to an unreadable thumbnail, audit labels in the artwork,
or old cards obscuring the next scene. Compare every card's rendered states
with the scene plan and revise weak compositions before passing the layout
review. Check the full heading and every status label against the card edges.
For each artifact reveal, identify the result the viewer can read at phone
scale. Reject the reveal when only its title is legible.
Record what you saw, including the visual focus and readability, in each
checked-state observation. A valid hash proves identity, not quality.

视觉质量与事实正确性分开审查。在 360px 视频宽度下，确认焦点内容无需暂停即可读清。当故事有日历、路线、图表或工件可展示时，拒绝重复使用"标题+行"式卡片。拒绝被缩成无法阅读的缩略图的工件；审查画面中的标签，拒绝旧卡片遮挡下一场景的情况。将每张卡片的渲染状态与场景计划逐一对比，在通过版式审查之前修正薄弱的构图。对照卡片边缘检查完整标题和每个状态标签。对每一次工件揭示，找出观众在手机尺寸下能读清的结果。若揭示画面中只有标题可辨认，则予以拒绝。
在每条已检查状态的观察记录中写下你看到的内容，包括视觉焦点和可读性。有效的哈希只能证明身份一致，不能证明质量。

Write `~/workspace/.output/<name>/layout_review.json` with `output_sha256` copied from the render manifest, `verdict: "pass"`, and `checked_states` containing one `{ "time": 1.1, "observation": "what you inspected" }` row for every time returned by `mm inspect`. Record failures and repair them before passing. Render again after any repair; do not post-process the checked MP4.

写入 `~/workspace/.output/<name>/layout_review.json`，其中 `output_sha256` 从渲染清单复制，`verdict: "pass"`，且 `checked_states` 为 `mm inspect` 返回的每个时间点各含一行 `{ "time": 1.1, "observation": "what you inspected" }`。记录失败项并在通过前修复。任何修复之后都要重新渲染；不要对已检查的 MP4 做后期处理。

Run `/opt/hatch/skills/magic-moment/mm publish <screenplay.json>` after both reviews pass. Deliver only its `DONE` path. Publication checks the source, assets, screenplay, review, and final output digests. Keep the run manifest as the provenance record.

两项审查都通过后，运行 `/opt/hatch/skills/magic-moment/mm publish <screenplay.json>`。只交付其 `DONE` 路径。发布环节会校验来源、素材、剧本、审查文件和最终输出的摘要。保留运行清单作为溯源记录。
