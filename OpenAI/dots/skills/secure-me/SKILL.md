---
name: secure-me
description: "Find compromised passwords in the user's personal accounts, prepare official reset flows, hand over before the user enters and submits each new credential, and help verify the change and authorized password-manager save."
---
<!-- BILINGUAL-EN-ZH -->

# dot Secure Me / dot 安全加固

Find exposed passwords, prepare each provider's official reset flow, and help the user verify what was changed and saved. The user enters, confirms, and submits every new credential.

查找已泄露的密码，准备各服务商的官方重置流程，并帮助用户核实更改和保存了什么。每个新凭据都由用户亲自输入、确认并提交。

## Find compromised passwords / 查找已泄露的密码

- Start with the chosen password manager, such as Chrome, and check its current security report and what the available tools can inspect. Use the user's existing account choices and connections. Report which accounts were covered and any gaps.
  从所选的密码管理器（例如 Chrome）入手，查看其当前的安全报告以及可用工具能检查的范围。使用用户已有的账户选择和连接。报告覆盖了哪些账户以及存在哪些缺口。
- Look for a current compromised-password warning, a verified provider notice, or a disclosure the user confirms. Check whether the warning concerns an old vault entry or a password already changed. An email address in an old breach is not enough to conclude that the current password was exposed. Separate confirmed and uncertain findings.
  寻找当前的密码泄露警告、经过验证的服务商通知，或用户自行确认的泄露披露。核查该警告针对的是旧的保险库条目还是已经改过的密码。电子邮件地址出现在旧泄露事件中，不足以断定当前密码已被暴露。把已确认的发现和不确定的发现分开。
- Include other accounts the manager reports as using the same exposed password, without extracting or comparing secret values. Missing, weak, or reused passwords alone are outside this cleanup. Start with compromised email, identity-provider, and password-manager accounts that could unlock others, then financial and other sensitive accounts.
  把管理器报告为使用了同一泄露密码的其他账户也纳入进来，但不得提取或比对保密值。仅有缺失、薄弱或重复使用的密码不属于本次清理范围。先处理可能解锁其他账户的电子邮件、身份提供商和密码管理器账户，再处理金融及其他敏感账户。

## Prepare the change and hand over / 准备更改并移交操作

- If the user asked for a check, report affected accounts and explain the next step. If they asked for help fixing them, prepare the official flow account by account. Follow `<confirmation_policy>`: ask the user to take over before any new credential is entered. The user must enter it, confirm it, and submit the change themselves, even if they asked the user's dot to fix everything.
  如果用户要求的是检查，报告受影响的账户并说明下一步。如果用户请求帮助修复，则逐个账户准备官方流程。遵循 `<confirmation_policy>`：在任何新凭据被输入之前，请用户接管操作。用户必须亲自输入、确认并提交更改，即使他们要求用户的 dot 代办一切。

【评论】"凭据只经用户之手"是密码类工作流的核心防滥用边界：代理可代为导航，但不接触任何秘密值，避免凭据进入对话或日志。

- Keep passwords and authentication codes inside a protected manager or official provider flow. Never ask for them or a vault export in chat, or expose them in page inspection, screenshots, logs, or notes. Say "I can't see your passwords" only when the integration enforces that. The user's dot only needs account references and non-secret status.
  把密码和验证码保存在受保护的密码管理器或服务商官方流程之内。绝不在聊天中索要它们或保险库导出文件，也不要在页面检查、截图、日志或笔记中暴露它们。只有在集成机制确实强制这一点时，才说"我看不到你的密码"。用户的 dot 只需要账户引用和非机密的状态信息。
- Reach the provider's genuine account settings independently of a warning email. Prepare its supported change or reset flow, show the user how to generate a unique password in their manager, and make sure they can recover it before they submit. Keep the last working session open. The user also handles any required sign-in, biometric, or multifactor step.
  不依赖警告邮件，独立到达服务商真正的账户设置页面。准备其受支持的更改或重置流程，向用户演示如何在密码管理器中生成唯一密码，并确保他们在提交之前能够找回该密码。保持上一个可用的会话处于打开状态。任何必需的登录、生物识别或多因素步骤也由用户完成。
- After the user completes the change, follow `<confirmation_policy>` for saving the specific credential in the chosen manager. If that save is already explicitly authorized and a protected tool can do it without exposing the secret, use it; otherwise ask immediately before saving or guide the user through it. Check the exact account and domain. If the provider accepted the change but the save failed, stop work on other accounts until this one can be recovered.
  在用户完成更改之后，遵循 `<confirmation_policy>` 决定是否把该具体凭据保存到所选的密码管理器。如果该保存已被明确授权，且某个受保护的工具能在不暴露机密的情况下完成，就使用它；否则在保存前立即询问，或引导用户自行完成。核对确切的账户和域名。如果服务商已接受更改但保存失败，则暂停处理其他账户，直到这个账户可以恢复为止。

## Verify and finish / 验证并收尾

- Track **changed**, **saved**, and **sign-in verified** separately. Use provider or manager status where it is available and label anything reported only by the user. Where supported, guide a fresh sign-in without risking lockout; an already open session is not proof. Say which checks still need the user.
  把**已更改**、**已保存**和**登录已验证**分开跟踪。在可得时使用服务商或管理器的状态，并把仅由用户口头报告的内容加以标注。在受支持时，引导一次全新登录且不冒锁定风险；已经打开的会话并不是证据。说明哪些检查仍需要用户完成。
- Keep a minimal private checkpoint with account references, findings, authorization, results, and next steps; never include secrets. After an interruption or unclear response, inspect the current provider and manager status before suggesting another reset.
  保留一份最小化的私人检查点，包含账户引用、发现、授权、结果和下一步；绝不包含机密。在中断或响应不明确之后，先检查当前服务商和管理器的状态，再建议另一次重置。
- Report how many distinct accounts were changed, saved, and verified, and what remains. If nothing was flagged, say which sources you checked. If there is evidence of stolen sessions or a compromised device, explain that a password change may leave that risk unresolved.
  报告有多少个不同账户被更改、保存和验证，以及还剩什么。如果没有发现任何问题，说明检查了哪些来源。如果有证据表明会话被盗或设备已失陷，要说明更改密码可能无法解决该风险。
- Keep the completion message short. A little celebration is fine after the work is verified. Only add personal details supported by evidence, and stay within the requested cleanup.
  完成消息保持简短。在工作经过验证之后，略微庆祝一下是可以的。只加入有证据支持的个人细节，并保持在所要求的清理范围之内。

## Examples / 示例

- **User:** "Chrome says four passwords were exposed. Help me fix them and save the replacements in Chrome."
  Chrome 说有四个密码已泄露。帮我修复它们并把替代密码保存在 Chrome 里。
  - dot: "I found four current warnings. I'll open each official reset page. You'll need to enter, confirm, and submit each new password. I'll keep track of what's changed and saved"
    dot："我发现了四条当前的警告。我会打开每一个官方重置页面。每个新密码都需要你亲自输入、确认并提交。我会跟踪哪些已更改、哪些已保存"
  - dot: When the evidence supports it: "All four changes and Chrome saves are confirmed 🎉 Three fresh sign-ins worked but the fourth still needs you."
    dot：当证据支持时："全部四项更改和 Chrome 保存均已确认 🎉 三次全新登录成功，第四次仍需要你来完成。"
- **User:** "A breach alert says my email was leaked. Do I need to reset this password?"
  一条泄露警报说我的邮箱被泄露了。我需要重置这个密码吗？
  - dot: "The alert names your email, but I haven't found evidence that your current password was exposed. I'll take a look at your password manager's current report before recommending a reset"
    dot："警报提到了你的邮箱，但我还没有找到当前密码已被暴露的证据。在建议重置之前，我会先查看你的密码管理器的当前报告"
- **User:** "The site accepted my new password, but Chrome didn't save it. Can we do the next one?"
  网站接受了我的新密码，但 Chrome 没有保存它。我们可以进行下一个了吗？
  - dot: "Let's recover this account in Chrome first. Keep the site open and I'll guide you through the protected save. Then we'll continue."
    dot："我们先在 Chrome 中把这个账户恢复好。保持该网站处于打开状态，我会引导你完成受保护的保存。然后我们再继续。"
- **User:** "Just do the last reset for me. I can send you the code."
  最后这一次重置就替我做了吧。我可以把验证码发给你。
  - dot: "Don't share the code with me. I can open the official page and show you what to do, but you'll need to enter, confirm, and submit the new password yourself"
    dot："不要把验证码发给我。我可以打开官方页面并告诉你怎么做，但你需要亲自输入、确认并提交新密码"

【评论】最后一个示例拒绝了"用户主动交出验证码"的代办请求：即使出于善意，凭据一旦进入对话就会落入日志与模型上下文，该条款堵住了社工式移交的口子。
