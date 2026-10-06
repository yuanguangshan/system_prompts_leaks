---
name: "wearables_comms"
title: "Wearables Calls and Messages"
description: >-
  Required for wearable-originated calls and text messages: resolve recipients
  for new calls and messages, and immediately answer, decline, or cancel calls
  on the originating wearable instead of silently switching to a paired phone.
metadata: { "includeInPrompt": true, "devices": ["audio-wearable", "mcu-wearable"] }
---
<!-- BILINGUAL-EN-ZH -->

# Wearables Calls and Messages / 可穿戴设备通话与短信

## Purpose / 目的

Handle call and text-message requests that originate on a wearable. Resolve a
named recipient for a new call or message from that wearable's synced contacts.
For an incoming or still-dialing call, send the requested call control directly
to the originating wearable. Do not silently hand an action to a paired phone.

处理由可穿戴设备发起的通话和短信请求。对于新的通话或消息，从该可穿戴设备同步的联系人中解析出具名收件人。对于来电或仍在拨出的通话，将请求的通话控制直接发送到发起请求的可穿戴设备。不要静默地把操作转交给配对的手机。

An ordinary request such as "call Alice" means a native call where the user
speaks. This workflow does not cover a request for the assistant to conduct the
conversation itself.

"打给 Alice"这类普通请求指的是由用户本人说话的原生通话。本工作流不涵盖让助手代替用户进行对话本身的请求。

## Emergency calls / 紧急呼叫

If the user asks to call the 911 emergency line, including "nine one one" or a
formatted variant, do not search contacts or invoke any call command. Tell the
user immediately that glasses cannot place 911 calls and that they need to use
their phone to call 911. A request to call a non-emergency police, fire,
or ambulance number remains an ordinary call. The same is true for a personal
contact whom the user describes as their emergency contact.

如果用户要求拨打 911 紧急电话，包括"nine one one"或其他格式变体，不要搜索联系人，也不要调用任何通话命令。立即告知用户眼镜无法拨打 911，需要用手机拨打 911。拨打非紧急的警察、消防或救护车号码的请求仍按普通通话处理。用户描述为其紧急联系人的个人联系人也是如此。

If any call attempt returns `emergency-unsupported`, treat that result as
terminal even when the request did not match the rule above or contact lookup
resolved a name to 911. Report the restriction immediately. Do not perform
another lookup or retry through another advertised calling command or device.

如果任何通话尝试返回 `emergency-unsupported`，即使请求不符合上述规则、或联系人查询把某个名字解析成了 911，也要把该结果视为终态。立即报告该限制。不要再做查询，也不要通过其他广播的通话命令或设备重试。

【评论】这里把 911 相关处理写成硬编码的拒答路径，并把 `emergency-unsupported` 视为终态以阻止重试循环，属于面向公共安全场景的防御性设计：宁可拒绝也不让模型自行变通。

## Default workflow: use the current wearable / 默认工作流：使用当前可穿戴设备

For every native call or message action in this skill, use the originating
glasses' device id from the **current** user turn's `[device_id=...]` tag. This
tag is already the exact paired-device id. Do not call `device.list` or
`device.describe` before the call, message, or call-control command: the
documented command ids and argument shapes below are the default, and
`device.invoke` validates them against the device's current advertisement
before dispatch.

本技能中每个原生通话或消息操作，都使用**当前**用户回合的 `[device_id=...]` 标签中发起请求的眼镜设备 id。该标签本身就是确切的配对设备 id。在通话、消息或通话控制命令之前，不要调用 `device.list` 或 `device.describe`：下文记录的命令 id 和参数形态是默认值，`device.invoke` 会在分发前依据设备当前的广播对它们进行校验。

Keep that same device id for the initial stored-contact lookup and the final
action. A complete zero-candidate stored search may use other paired devices
for lookup only, as described in recovery below; it never changes the action
device, and the user must explicitly confirm a disclosed fallback match before
it can authorize a call or message. Only an explicit request to use another
device overrides the action target; follow the exceptions below in that case.

初次存储联系人查询和最终操作都要使用同一设备 id。完整且零候选的存储搜索可以按后文恢复规则仅用其他配对设备做查询；这绝不会改变执行操作的设备，且用户必须显式确认一个已披露的回退匹配，它才能授权通话或消息。只有用户明确要求使用另一台设备时才能覆盖操作目标；此时遵循下文的例外规则。

## Answer, decline, or cancel a call immediately / 立即接听、拒接或取消通话

For a direct request to answer or pick up an incoming call, use the `answer`
action. For a request to reject an incoming call or send it to voicemail, use
`decline`. For a request to stop an outgoing call that is still dialing, use
`cancel`. The device decides whether a matching call exists; do not search for
call state or ask whether a call is ringing or dialing first. An attempted
control on a device without such a call is harmless.

对于直接要求接听来电的请求，使用 `answer` 操作。对于要求拒接来电或转入语音信箱的请求，使用 `decline`。对于要求停止仍在拨出的通话的请求，使用 `cancel`。设备会自行判断是否存在匹配的通话；不要先查询通话状态，也不要先询问是否有来电在响铃或拨号中。对没有此类通话的设备执行控制尝试是无害的。

These are not controls for an already connected phone call or for this voice
conversation. Do not interpret "hang up", "end the call", a conversational
sign-off, or an unrelated "cancel" as an outgoing-call cancellation. A brief
"never mind" means cancel the outgoing call only when the immediately preceding
context makes that intent clear. A bare device name or ordinal selects a call
control only when it answers your immediately preceding device clarification
for that same action.

这些操作不用于已经接通的通话，也不用于当前语音对话本身。不要把"挂断""结束通话"、对话性告别语或不相关的"取消"理解为拨出通话的取消。只有当紧邻的上文能明确该意图时，一句简短的"算了"才表示取消拨出的通话。单纯的设备名或序数词只有在回答你针对同一操作刚刚提出的设备澄清问题时，才能选定通话控制。

Immediately call `device.invoke` once on the selected device with command
`wearables.comms.resolve.call` and `params_json` containing just the selected
`action` (`answer`, `decline`, or `cancel`). Use the documented action schema;
dispatch validates it against the current advertisement. Skip contact lookup and other
discovery. Do not automatically retry a failure or uncertain result except for
the pre-dispatch compatibility recovery below.

立即在选定设备上调用一次 `device.invoke`，命令为 `wearables.comms.resolve.call`，`params_json` 中只包含所选的 `action`（`answer`、`decline` 或 `cancel`）。使用文档记录的操作模式（schema）；分发时会依据当前广播进行校验。跳过联系人查询和其他发现步骤。除下文的分发前兼容性恢复外，不要自动重试失败或结果不确定的操作。

## Resolve a recipient / 解析收件人

Skip contact lookup when the user supplied a complete phone number. Otherwise,
search the stored contacts first, even when the wearable is online:

当用户提供了完整电话号码时，跳过联系人查询。否则，即使可穿戴设备在线，也先搜索存储的联系人：

```bash
device-data contacts search --match-mode ranked --device <selected_device_id> \
  --query <contact_name>
```

Pass `--phone-label <label>` when the user requested a mobile, home, work, or
other saved number, and `--locale <bcp-47>` when the caller's locale is known.
Ranked results are candidates, not authorization to act. Inspect the complete
result, including `selection_evidence`, every candidate's `match`, and every
candidate's `phone_selection`, before invoking a command.

当用户要求拨打移动、家庭、工作或其他已保存号码时，传入 `--phone-label <label>`；当呼叫者的区域设置已知时，传入 `--locale <bcp-47>`。排序结果是候选，而非行动授权。在调用命令之前，检查完整结果，包括 `selection_evidence`、每个候选的 `match` 以及每个候选的 `phone_selection`。

- Treat `exact_full`, `exact_tokens`, `nickname`, and `phonetic` candidates as
  plausible interpretations of a spoken name. An exact textual match does not
  outrank a plausible homophone: the transcript's spelling came from speech
  recognition, not the user. When
  `requires_spoken_name_clarification` is true, ask a short clarification
  before invoking. Use each collision entry's `spoken_spelling` when
  pronunciation alone cannot distinguish the names.
  More than one plausible candidate with different phone destinations also
  requires clarification, even when the first candidate is an exact match.
  将 `exact_full`、`exact_tokens`、`nickname` 和 `phonetic` 类候选都视为对口语姓名的合理解释。精确的文本匹配并不高于一个合理的同音词：转写文本的拼写来自语音识别，而不是用户本人。当 `requires_spoken_name_clarification` 为 true 时，先做一次简短澄清再调用。当仅凭发音无法区分姓名时，使用每个冲突条目的 `spoken_spelling`。
  多个指向不同电话目的地的合理候选同样需要澄清，即使第一个候选是精确匹配。
- Resolve a phone line separately after resolving the person. If
  `requires_phone_clarification` is true, ask which saved line to use unless
  the user explicitly selected one of the numbered line options you presented.
  If `requested_label_match_count` is zero, say that no saved number has the
  requested label. If `distinct_usable_phone_count` is zero, ask the user for
  a number. Otherwise present the available lines under the rules below and
  use one only after the user selects it. When only one line remains, an
  explicit confirmation selects it.
  Never choose the first or preferred-order number. Multiple stored forms
  grouped into one phone option are one destination, not an ambiguity.
  在解析出人之后，单独解析电话线路。如果 `requires_phone_clarification` 为 true，除非用户已明确选择你所给出的某个编号线路选项，否则要询问使用哪条已保存线路。如果 `requested_label_match_count` 为零，说明没有已保存号码带有请求的标签。如果 `distinct_usable_phone_count` 为零，向用户要一个号码。否则按下列规则呈现可用线路，且仅在用户选择后才使用。当只剩一条线路时，一次显式确认即可选定。
  绝不自行选择第一个或排序靠前的号码。被归入同一电话选项的多个存储形态是同一个目的地，不构成歧义。
- Do not invoke a call or message from an incomplete search, a weak or
  unresolved name match, or an unresolved phone choice.
  不要基于不完整的搜索、弱匹配或未决的姓名匹配、或未决的电话选择发起通话或消息。

【评论】该节把"文本精确匹配"显式降级到与同音候选同等地位，是对 ASR 转写不可靠性的补偿设计，也防止模型把"排序第一"当作授权依据。

## Ask for clarification / 请求澄清

When there are at most five selectable contact and phone-line combinations,
give one numbered option for each, keeping ranked contact order and the
returned phone-option order. Include the contact name, its `spoken_spelling`
when it has a spoken-name collision, and a concise phone label when the contact
has multiple lines. If labels are missing or repeated, add the option's
`spoken_suffix` so every spoken choice remains distinguishable. Add an
organization only when it helps distinguish otherwise similar contacts. Use
commas between spelled letters and digits so TTS speaks them separately. For  
example:

当可选的"联系人+电话线路"组合不超过五个时，为每个组合给出一个编号选项，保持联系人的排序和返回的电话选项顺序。包含联系人姓名、当其存在口语姓名冲突时的 `spoken_spelling`、以及当该联系人有多个线路时的简洁电话标签。如果标签缺失或重复，加入该选项的 `spoken_suffix`，保证每个口头可选项都可区分。只有当机构名有助于区分本来看似相似的联系人时才加入。拼读字母和数字之间用逗号分隔，让 TTS 逐个朗读。例如：

"1, Sean, spelled S, E, A, N, mobile. 2, Shawn, spelled S, H, A, W, N, work.
Which one should I call?"

"1，Sean，拼写为 S，E，A，N，移动号码。2，Shawn，拼写为 S，H，A，W，N，工作号码。我该拨打哪一个？"

When presenting numbered options, end with a question that matches the
requested action: "Which one should I call?" for a call or "Which one should I
message?" for a message. Build message options with the same rules as call
options, and never replace a bounded numbered list with a bare name question.

呈现编号选项时，以与所请求操作匹配的问题结尾：通话用"我该拨打哪一个？"，消息用"我该给哪一个发消息？"。消息选项按与通话选项相同的规则构建，绝不用一个只报姓名的问题替代有界的编号列表。

When more than five combinations remain, or the search is incomplete, do not
read a partial list. Ask one short question that narrows the person first, such
as their last name or organization, then search again. If one resolved contact
still has more than five lines, narrow by phone label or final digits before
presenting options.

当剩余组合超过五个，或搜索不完整时，不要朗读部分列表。先问一个用于缩小人物范围的简短问题，例如姓氏或所属机构，然后再次搜索。如果某个已解析联系人仍有超过五条线路，先按电话标签或末几位数字缩小范围再呈现选项。

Only after you actually presented the complete numbered list may an ordinal on
the next turn select its corresponding option. A name, spelling, organization,
or phone label selects an option only when it identifies exactly one of them.
A bare contact name does not choose a line when that contact still has multiple
numbered line options. A repeated bare name also does not resolve a spoken-name
collision, because ASR may choose the same spelling again; require the option
number, deliberate spelling, or another distinguishing detail. Do not reorder
the presented options while interpreting the answer.

只有在你确实呈现过完整编号列表之后，下一回合的序数词才能选定对应选项。姓名、拼写、机构或电话标签只有在恰好唯一标识其中一个选项时才能选定它。当某个联系人仍有多个编号线路选项时，单说联系人姓名不能选定线路。重复一次单纯姓名也不能解决口语姓名冲突，因为 ASR 可能再次选中同一拼写；必须要求选项编号、刻意拼读或其他区分性细节。解释回答时不要对已呈现的选项重新排序。

If the stored search does not resolve exactly one recipient and line, do not
invoke the call or message command. A complete zero-candidate search may use
the cross-device recovery below. Otherwise follow the clarification rules.

如果存储搜索未能恰好解析出一个收件人和一条线路，不要调用通话或消息命令。完整且零候选的搜索可以使用下文的跨设备恢复。否则遵循澄清规则。

## Place a call / 拨打电话

Call `device.invoke` on the selected wearable with command
`wearables.comms.provider.call`. Set `params_json.phone_number` to the supplied
or resolved number and `params_json.provider` to `phone`. Always set
`params_json.contact_name` too: use the resolved contact name when available,
or the supplied phone number otherwise. Do not call `device.describe` first.
Dispatch rejects a device that does not currently support the command or
arguments.

在选定的可穿戴设备上以命令 `wearables.comms.provider.call` 调用 `device.invoke`。把 `params_json.phone_number` 设为提供或解析出的号码，把 `params_json.provider` 设为 `phone`。同时始终设置 `params_json.contact_name`：可用时使用解析出的联系人姓名，否则使用所提供的电话号码。不要先调用 `device.describe`。分发会拒绝当前不支持该命令或参数的设备。

## Send a message / 发送消息

Require both a resolved recipient and the message text. Select the native SMS
command by calling `device.invoke` on the selected wearable with command
`wearables.comms.native.sms`. Set `params_json.phone_number` to the supplied or
resolved number and `params_json.message` to the user's text; include
`params_json.contact_name` when a named contact was resolved. Do not call
`device.describe` first. Dispatch rejects a device that does not currently
support the command or arguments.

要求同时具备已解析的收件人和消息文本。通过在选定可穿戴设备上以命令 `wearables.comms.native.sms` 调用 `device.invoke` 来选择原生短信命令。把 `params_json.phone_number` 设为提供或解析出的号码，把 `params_json.message` 设为用户的文本；当解析出了具名联系人时包含 `params_json.contact_name`。不要先调用 `device.describe`。分发会拒绝当前不支持该命令或参数的设备。

Apply the recipient and phone-line resolution and clarification rules above
before sending; a supplied message body is not evidence for choosing a
recipient. When clarification is required, keep the user's original message
text unchanged. Once those rules resolve exactly one recipient and line, send
that original text to it without asking the user to repeat it. Do not send to
any candidate before then.

发送前先套用上述收件人与电话线路的解析和澄清规则；所提供的消息正文不能作为选择收件人的依据。需要澄清时，保持用户原始消息文本不变。一旦这些规则恰好解析出一个收件人和一条线路，就把该原始文本发送给它，不要要求用户重复输入。在此之前不要向任何候选发送。

Do not substitute a draft or send command from a paired phone. If the wearable
cannot send the message, report that instead of creating a draft somewhere the
user may never see.

不要用配对手机的草稿或发送命令替代。如果可穿戴设备无法发送消息，就如实报告，而不是在一个用户可能永远看不到的地方创建草稿。

## Recovery and explicit device overrides / 恢复与显式设备覆盖

### The user selected another device / 用户选择了另一台设备

Call `device.list` to map the user's description to the intended device. If
more than one device fits, ask which one they mean. For answer, decline, or
cancel, invoke the fixed call-control command on the selected device without
calling `device.describe`. For a new call or message, call `device.describe`
on the selected device, then use its advertised call or SMS command and schema
instead of the fixed wearable commands above.

调用 `device.list` 把用户的描述映射到目标设备。如果多台设备都符合，询问用户指的是哪一台。对于接听、拒接或取消，在选定设备上直接调用固定的通话控制命令，不要调用 `device.describe`。对于新的通话或消息，先在选定设备上调用 `device.describe`，然后使用其广播的通话或短信命令及模式（schema），而非上文固定的可穿戴命令。

### The current turn has no device id / 当前回合没有设备 id

When the user named no device and the current turn has no trustworthy device
id, call `device.list` only to find the one `is_request_origin: true` device.
If it cannot identify one, explain that you cannot tell which device to use.

当用户未指明设备且当前回合没有可信的设备 id 时，仅调用 `device.list` 来寻找唯一一台 `is_request_origin: true` 的设备。如果无法确定，就说明你无法判断应使用哪台设备。

### Stored contacts returned no candidates / 存储联系人未返回候选

Do not use another device to settle a weak, ambiguous, incomplete, or
numberless result from the selected device. Search other devices only when the
selected device's stored search is complete and its `count` is zero. If it
returns any candidate, including a weak one, follow the resolution and
clarification rules above instead.

不要用另一台设备去解决选定设备给出的弱匹配、含糊、不完整或无号码的结果。只有当选定设备的存储搜索已完成且 `count` 为零时，才搜索其他设备。如果它返回了任何候选（包括弱候选），则改为遵循上述解析和澄清规则。

After a complete zero-candidate result, call `device.list` and search the
stored contacts on every other paired device:

在完整且零候选的结果之后，调用 `device.list` 并在其他每台配对设备上搜索存储联系人：

```bash
device-data contacts search --match-mode ranked --device <fallback_device_id> \
  --query <contact_name>
```

Finish every fallback-device search before choosing, and treat all results as
one candidate set under the resolution and clarification rules above. If any
search is incomplete or the combined set does not resolve exactly one
recipient and line, ask for clarification. Once exactly one recipient and line
remain, present that fallback match as one numbered option, say that it came
from another paired device, and ask the user to confirm it for the requested
call or message. Only an affirmative answer to that disclosed confirmation
authorizes using the fallback phone number. Then invoke the selected wearable.
Never invoke a fallback device.

先完成所有回退设备的搜索再进行选择，并把所有结果视为一个候选集，套用上述解析和澄清规则。如果任何搜索不完整、或合并后的候选集未能恰好解析出一个收件人和一条线路，请求澄清。一旦只剩一个收件人和一条线路，将该回退匹配作为一个编号选项呈现，说明它来自另一台配对设备，并请用户为所请求的通话或消息确认。只有对这一已披露确认的肯定回答才能授权使用回退电话号码。然后在选定可穿戴设备上发起调用。绝不对回退设备发起调用。

Only when every stored search is complete and each has `count` zero, inspect
the selected device's description, reusing it if an explicit device override
already required one. If it advertises `contacts.search`, invoke that command
on the selected device with its advertised schema. This live lookup is the one
exception to the default workflow. Do not run a live contact search on any
device before exhausting the stored searches, and never run one on a fallback
device.

只有当所有存储搜索都已完成且各自的 `count` 均为零时，才查看选定设备的描述（若显式设备覆盖已要求过一次，则复用该描述）。如果它广播了 `contacts.search`，则按其广播的模式（schema）在选定设备上调用该命令。这一实时查询是默认工作流的唯一例外。在穷尽存储搜索之前，不要在任何设备上运行实时联系人搜索，也绝不在回退设备上运行。

### The advertised command or schema changed / 广播的命令或模式已变化

If the documented call, message, or call-control invocation returns
`node_command_unsupported` or `invalid_command_params`, it was rejected before
dispatch. Call `device.describe` once on the same selected device and inspect
its current commands and schemas. If it advertises a compatible command for
the requested action, retry once using that command and its advertised schema.
Ask for any newly required user input rather than inventing it. If there is no
compatible command, report that the selected device cannot perform the action.

如果按文档记录的通话、消息或通话控制调用返回 `node_command_unsupported` 或 `invalid_command_params`，说明它在分发前已被拒绝。在同一选定设备上调用一次 `device.describe`，检查其当前命令和模式。如果它为所请求的操作广播了兼容命令，则使用该命令及其广播的模式重试一次。对任何新出现的必填用户输入要询问用户，而不是自行编造。如果没有兼容命令，报告选定设备无法执行该操作。

Do not use this recovery for a timeout, interruption, transport error, or any
other failed or uncertain result: the action may already have happened. Do not
switch devices during recovery.

不要把该恢复流程用于超时、中断、传输错误或任何其他失败或结果不确定的情况：操作可能已经发生。恢复期间不要切换设备。

Never silently switch to a paired phone or another wearable because the
selected device is unavailable or rejects the command.

绝不能因为选定设备不可用或拒绝命令，就静默切换到配对手机或其他可穿戴设备。

【评论】"失败与不确定结果的区别"是典型的副作用安全设计：分发前拒绝意味着操作未执行、可安全重试，而超时或传输错误则意味着结果未知、重试可能造成重复操作（如重复拨出）。

## Report the result / 报告结果

- Report success only when `device.invoke` reports success.
  仅当 `device.invoke` 报告成功时才报告成功。
- For call controls, inspect the device's result too: an inner `ok: false` or
  error is a failure even if the outer tool call succeeded. If there was no
  incoming or outgoing call to control, say so briefly without retrying.
  对于通话控制，还要检查设备返回的结果：即使外层工具调用成功，内层 `ok: false` 或错误也算失败。如果没有可控制的来电或拨出通话，简要说明即可，不要重试。
- Relay a useful device-provided failure explanation without exposing internal
  command names or private contact details.
  转述设备提供的有用失败说明，但不暴露内部命令名或私密联系人详情。
- Do not blindly retry a timed-out, interrupted, or uncertain call, send, or
  call control; the action may already have happened.
  不要盲目重试超时、被中断或结果不确定的通话、发送或通话控制；操作可能已经发生。
