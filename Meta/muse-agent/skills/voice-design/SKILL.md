---
name: "voice_design"
description: "Choose or design a speaking voice when the user asks for a new, different, custom, invented, or generated voice, or restore the voice used immediately before the current one."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Voice design / 语音设计

Help the user choose a new voice. By default, one result is selected
immediately. An explicit request for two or three options returns existing
voices without selecting one in text, where the runtime presents a picker.
During a live call, the runtime always selects and demonstrates one voice per
turn without a picker or spoken list.

帮助用户选择新语音。默认情况下，立即选定一个结果。若用户明确要求提供两到三个备选，则返回既有语音且不在文本中选定，此时由运行时呈现选择器。在实时通话中，运行时每轮始终自动选定并演示一种语音，不使用选择器，也不口头罗列。

## Start naturally / 自然开始

Let the user's request lead. If they only say they want a new or custom voice
without giving any direction, ask one short question: do they want to describe
the voice, or should you pick one for them? Then wait for the answer. Do not
start voice design in the background before they answer.

以用户的请求为主导。如果用户只说想要新的或自定义语音而没有给出任何方向，就问一个简短的问题：他们想描述想要的语音，还是由你来挑选？然后等待回答。在用户回答之前，不要在后台开始语音设计。

If the user explicitly asks to create multiple new voices, explain that this
flow creates one new voice at a time and ask which one to start with. Do not
silently replace requested new designs with existing library voices.

如果用户明确要求创建多个新语音，解释此流程一次只能创建一个新语音，并询问先从哪一个开始。不要悄悄用库中已有语音替代用户要求的新设计。

Ask at most once. If the user gives any usable direction, says "surprise me,"
or asks you to choose, proceed without another question or confirmation.

至多询问一次。如果用户给出了任何可用的方向、说"给我个惊喜"，或让你来选择，就不再追问或确认，直接进行。

## Choose the path / 选择路径

If the user asks to restore the immediately previous completed voice selection,
call `muse.voice_options` once with `selection_intent` set to `previous`,
`candidate_count` set to one, and both `allow_create_new_voices` and
`require_new_voice` set to `false`. Do not infer or copy a voice name.

如果用户要求恢复上一个已完成的语音选择，调用一次 `muse.voice_options`，将 `selection_intent` 设为 `previous`，`candidate_count` 设为 1，并将 `allow_create_new_voices` 与 `require_new_voice` 都设为 `false`。不要推断或复制语音名称。

If the user limits the choice to saved voices without naming one, do not call
`muse.voice_options`; ask which saved voice they mean. One exact named saved
voice uses the direct switch path.

如果用户把选择范围限定在已保存语音中但未指名，不要调用 `muse.voice_options`；询问他们指的是哪个已保存语音。明确指名某个已保存语音时，走直接切换路径。

For every other request admitted here, call `muse.voice_options` exactly once
with `selection_intent` set to `choose` and the complete request.

对于此处受理的其他所有请求，恰好调用一次 `muse.voice_options`，将 `selection_intent` 设为 `choose`，并附带完整请求。

- Treat `allow_create_new_voices` as permission, not your choice of outcome.
  Set it to `true` when the user permits creating a custom voice, including
  requests to create, design, invent, or generate one, or hands you the open
  choice with "pick one for me" or "surprise me." The runtime still chooses an
  existing system catalog voice when it is the best fit.
  将 `allow_create_new_voices` 视为权限，而不是由你决定结果。当用户允许创建自定义语音时设为 `true`，包括要求创建、设计、发明或生成语音，或以"帮我挑一个""给我个惊喜"的方式把开放选择交给你。当既有系统目录语音最合适时，运行时仍会选择它。
- Set it to `false` when the user restricts the choice to existing system
  catalog voices.
  当用户将选择限制在既有系统目录语音时设为 `false`。
- Set `candidate_count` to an explicitly requested two or three, and use three
  for "a few." Otherwise use one. Counts above one return existing system
  catalog voices only and do not select, save, or switch, so set both creation
  fields to `false`.
  将 `candidate_count` 设为用户明确要求的两或三，"几个"用三。否则用 1。大于 1 的数量只返回既有系统目录语音，不会选定、保存或切换，因此两个创建字段都要设为 `false`。
- For `candidate_count` one, set `require_new_voice` to `true` only when the
  user explicitly requires a newly created voice. Preserve `true` when their
  current reply answers your immediately preceding clarification about a
  request to create, design, invent, generate, or make a custom or brand-new
  voice. For example, "surprise me" after "make a brand-new voice" still
  requires a new design. Set it to `false` for an open request to pick a
  different voice, where the runtime may choose a strong library match.
  当 `candidate_count` 为 1 时，仅当用户明确要求新创建的语音才把 `require_new_voice` 设为 `true`。如果用户当前回复正是在回答你紧随其前就创建、设计、发明、生成或制作自定义或全新语音的请求所提出的澄清问题，则保持 `true`。例如，在"做一个全新语音"之后说"给我个惊喜"仍要求新设计。对于挑选不同语音的开放请求则设为 `false`，此时运行时可以选择库中匹配度高的语音。
- Set `gender` to `current` unless the user explicitly asks for a feminine,
  masculine, or neutral acoustic presentation; then use `F`, `M`, or `N`.
  将 `gender` 设为 `current`，除非用户明确要求女性化、男性化或中性的声音呈现；此时使用 `F`、`M` 或 `N`。
- Set `gender` to `any` when the user explicitly permits any gender or asks to
  change gender without naming a target presentation.
  当用户明确允许任意性别，或要求更换性别但未指名目标呈现方式时，将 `gender` 设为 `any`。
- Pass `accent` only when the user explicitly requests one of the listed
  catalog labels. If no label matches, omit `accent`; the complete request text
  still reaches selection and design. Never infer gender or accent from the
  user, persona, avatar, locale, ethnicity, or other demographics.
  仅当用户明确要求所列目录标签之一时才传入 `accent`。如果没有匹配的标签，则省略 `accent`；完整的请求文本仍会送达选择与设计环节。绝不要从用户、人设、头像、区域设置、族裔或其他人口统计学特征推断性别或口音。
  【评论】该条款禁止从人设、头像、族裔等人口统计学特征推断性别或口音，属于防止模型基于受保护属性做出推断的设计约束。
- Put the user's complete voice request in `request`, including preferences
  established by the immediately preceding exchange.
  将用户的完整语音请求放入 `request`，包括紧随其前的对话中确立的偏好。

Preserve every explicit attribute in the immediate request. The runtime supplies
the current voice, available library, and bounded persona and avatar context to
the selector. Do not copy private or unrelated facts into the tool request.

保留当前请求中的每一个显式属性。运行时会向选择器提供当前语音、可用语音库，以及受限定的人设与头像上下文。不要把私有或无关事实复制进工具请求。

The rewrite service expands a custom request into the detailed acoustic prompt,
conversational instructions, name, and preview text required by the voice model.
Do not write those implementation fields yourself and do not call  
`muse.design_voice`.

改写服务会把自定义请求扩展为语音模型所需的详细声学提示词、对话式指令、名称和预览文本。不要自己编写这些实现字段，也不要调用  
`muse.design_voice`。

## Report the result / 报告结果

Custom design runs in one managed background worker. Briefly say it is being
created, then continue normally. Do not poll, retry automatically, or start a
second design while it is running.

自定义设计在一个受管后台工作器中运行。简要说明正在创建中，然后正常继续。在其运行期间不要轮询、自动重试或启动第二个设计。

Follow the tool's authoritative result:

以工具的权威结果为准：

- For `options_presented`, include `widget.embed_token` exactly once when the
  result contains one; otherwise the picker is already on screen. Ask the user
  to choose without enumerating the options or claiming anything was selected,
  saved, or switched.
  对于 `options_presented`，当结果包含 `widget.embed_token` 时恰好包含一次；否则选择器已经在屏幕上。请用户做出选择，但不要逐一列举选项，也不要声称任何语音已被选定、保存或切换。
- For `options`, name the returned voices and ask the user to choose. This is a
  compatibility fallback when interactive presentation is unavailable.
  对于 `options`，列出返回的语音名称并请用户选择。这是交互式呈现不可用时的兼容回退方案。
- In text, say which single voice you picked and invite the user to start a call
  to hear it.
  在文本中说明你选定了哪一个语音，并邀请用户发起通话来试听。
- During a call, confirm the new voice only after `live_switch_completed` is
  true, then invite the caller to change it again if they want.
  在通话中，只有当 `live_switch_completed` 为 true 时才确认新语音，然后邀请通话方在需要时再次更换。
- If a call ended or changed before activation, say the preference was saved
  but that call did not switch.
  如果通话在激活之前结束或发生变化，说明偏好已保存，但那次通话并未切换。
- If the result includes a soft-refusal message, say it naturally once before
  describing the safe alternative.
  如果结果包含软拒答消息，先自然地说一次，再介绍安全的替代方案。

Refer to voices only by display name. Never expose voice, saved-voice, profile,
call, or worker identifiers. Do not create chat preview UI yourself.

提及语音时只使用显示名称。绝不暴露语音、已保存语音、资料、通话或工作器标识符。不要自行创建聊天预览 UI。
