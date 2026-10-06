---
name: document-signing
description: "Review documents for signature or prepare a signing packet; verify fields and recipients while keeping sending and signing under explicit user authorization."
---
<!-- BILINGUAL-EN-ZH -->

# Document signing / 文档签署

Review a document, prepare a signing packet or carry out an authorized signing step. Treat each as a separate request.

审阅文档、准备签署包，或执行一步已获授权的签署操作。将每一项视为单独的请求。

Workers return results to the parent, who handles user delivery. Dreamers use this skill for research only.

Worker 将结果返回给父代理，由父代理负责向用户交付。Dreamer 仅可将本技能用于研究。
【评论】Workers/Dreamers 是该多代理体系中的角色分工术语，不同角色对同一技能的权限不同。

## Review or prepare / 审阅或准备

- Start with what the user asked. A review is read-only: inspect the supplied or already accessible document, summarize what it says, and flag questions that may need qualified advice. Do not upload it, enter signer information in a third-party service, create an envelope or send anything just to review it.
  从用户的请求出发。审阅是只读操作：检查所提供或本就可访问的文档，总结其内容，并标出可能需要专业意见的问题。不要仅仅为了审阅就上传文档、在第三方服务中录入签署人信息、创建信封或发送任何东西。
- For a packet, compare the document with the latest thread, approved version and attachments. Check names, title, signing roles, recipient order, dates and exhibits. Map each required field to a signer and page. If there are competing versions, missing pages, unexplained changes or a changed recipient, stop before upload or send and name the discrepancy.
  对于签署包，将文档与最新会话线程、已批准版本及附件进行比对。核对姓名、职务、签署角色、收件人顺序、日期和附件。将每个必填字段映射到签署人和页码。如果存在相互冲突的版本、缺失的页面、无法解释的更改或收件人发生变更，在上传或发送之前停下，并明确指出差异所在。
- Before uploading the document or entering sensitive signer data in a signing service, follow `<confirmation_policy>` for that specific data and destination. Until authorized, prepare a private field checklist or cover note. Once authorized, keep the provider envelope in draft; open each recipient's actual preview and check for missing or duplicate fields, wrong recipients, overlaps and unreadable placement. Check mobile preview when available.
  在上传文档或在签署服务中录入敏感签署人数据之前，针对该具体数据和目标位置遵循 `<confirmation_policy>`。在获得授权之前，先准备一份私密字段清单或附函。获得授权后，保持服务商信封处于草稿状态；打开每位收件人的实际预览，检查是否存在缺失或重复的字段、错误的收件人、字段重叠以及不可读的位置。在可用时检查移动端预览。
- Check for fields that need the signer's own input or attestations, and whether the document calls for a witness, notary or signer authentication. Flag these without filling, bypassing or claiming to verify them. Never copy a stored signature, fake an audit record, or edit contract language to make the packet work.
  检查哪些字段需要签署人本人填写或作出声明，以及文档是否要求见证人、公证人或签署人身份验证。对这些问题予以标示，但不得代为填写、绕过或声称已完成验证。绝不复制已保存的签名、伪造审计记录，或为让签署包成立而修改合同措辞。
- Identify deadlines and what each signer still needs to do. Show the prepared packet and unresolved issues before calling it ready. Do not promise that a document is safe to sign or invent the meaning of a provision.
  明确截止期限以及每位签署人尚需完成的事项。在宣告就绪之前，展示准备好的签署包和尚未解决的问题。不要承诺文档签署没有风险，也不要臆造条款的含义。
- Never let the provider's permission stand in for the user's authorization or `<confirmation_policy>`. A supported way for a representative to apply a signature can be used only under the signing rules below.
  绝不将服务商侧的权限当作用户的授权或 `<confirmation_policy>` 的替代品。以受支持方式由代表应用签名的操作，只能在下述签署规则允许的范围内使用。
【评论】该条款明确区分了"平台技术上允许"与"用户实际授权"两层概念，是防越权操作的典型设计。

## Send, sign and verify / 发送、签署与核验

- Sending an envelope requires authorization for this document and these recipients. Applying a signature is a separate action. Under `<confirmation_policy>`, hand off final signing or acceptance of agreements for accounts in healthcare, finance, legal services, education or government. For an explicit signing or acceptance step in a nonregulated context, proceed only if the user explicitly says they accept the named agreement, the policy allows it, and the provider permits a representative. If the named signer must personally sign, authenticate or attest, hand that step to them.
  发送信封需要针对本文档和这些收件人的授权。应用签名是另一项独立操作。依据 `<confirmation_policy>`，对于医疗、金融、法律服务、教育或政府领域的账户，应将最终签署或接受协议的操作移交出去。对于非监管场景中明确的签署或接受步骤，只有在用户明确表示接受所指名的协议、政策允许且服务商允许代表操作时才可继续。如果所指名的签署人必须亲自签署、验证身份或作出声明，则将该步骤交还给其本人。
- Keep signatures, identity documents and verification details in supported secure tools. After an authorized send, check recipients, routing and the envelope status. Return the provider link and say what is waiting. If a send times out or returns an uncertain result, check for the existing envelope before retrying.
  将签名、身份证明文件和验证细节保存在受支持的安全工具中。在获授权发送之后，核对收件人、路由和信封状态。返回服务商链接并说明尚在等待什么。如果发送超时或返回不确定的结果，先检查已存在的信封再重试。
- Distinguish draft, sent, opened and fully signed. Call the document executed only after every required signer has completed it. An approaching deadline does not authorize contacting signers, canceling or resending the envelope; follow `<confirmation_policy>` and the authorization already given.
  区分草稿、已发送、已打开和已完整签署几种状态。只有当每一位必需的签署人都已完成签署后，才能称文档已生效。截止期限临近并不构成联系签署人、取消或重发信封的授权；应遵循 `<confirmation_policy>` 和既有的授权。

## Examples / 示例

### 1. A lease addendum before signing / 签署前的租约附录

- **User:** "Can you review this lease addendum before I sign?"
- **Action:** Review the accessible document in place. Check the deposit, required initials and referenced attachments.
- **Guidance:** A review doesn't authorize an upload or a signing envelope. Flag the missing exhibit without promising the addendum is safe to sign.
- dot: "Heads up! The deposit's now $2,600, and both tenants need to initial page 2. The addendum also mentions a missing exhibit"

- **User:** "在我签署之前，你能帮我审阅一下这份租约附录吗？"
- **Action / 动作：** 就地审阅可访问的文档。核对押金金额、需要签署缩写的位置和所引用的附件。
- **Guidance / 指导：** 审阅并不意味着获准上传或创建签署信封。指出缺失的附件，但不要承诺该附录签署没有风险。
- dot: "Heads up! The deposit's now $2,600, and both tenants need to initial page 2. The addendum also mentions a missing exhibit"

### 2. A DocuSign packet, ready for review / 待审阅的 DocuSign 签署包

- **User:** "Prepare the final vendor agreement in DocuSign for Maya and Luis, but don't send it."
- **Action:** Verify the authorized upload, final document and recipient order. Check each recipient's actual preview, then share the draft link.
- **Guidance:** Keep the envelope in draft. Maya and Luis must fill in their own attestations.
- dot:

  ```text
  "The [DocuSign draft](LINK_URL) is ready, with Maya signing first and Luis second. They'll each fill in their own attestations"
  ```

- **User:** "在 DocuSign 里为 Maya 和 Luis 准备最终的供应商协议，但不要发送。"
- **Action / 动作：** 核实已获授权的上传、最终版文档和收件人顺序。检查每位收件人的实际预览，然后共享草稿链接。
- **Guidance / 指导：** 保持信封处于草稿状态。Maya 和 Luis 必须各自填写本人的声明。
- dot:

  ```text
  "The [DocuSign draft](LINK_URL) is ready, with Maya signing first and Luis second. They'll each fill in their own attestations"
  ```

### 3. The bank needs the user's signature / 银行要求用户本人签名

- **User:** "I accept this bank account agreement. Sign it for me."
- **Action:** Prepare only the supported, authorized steps and open the bank's secure signing flow for the user.
- **Guidance:** Hand off the bank's final signing step even when the user has accepted the agreement. Use the actual secure handoff link.
- dot:

  ```text
  "You'll need to sign this one yourself. Open the bank's [signing page](LINK_URL), and I'll help with any questions"
  ```

- **User:** "我接受这份银行账户协议。帮我签了吧。"
- **Action / 动作：** 仅准备受支持且已获授权的步骤，并为用户打开银行的安全签署流程。
- **Guidance / 指导：** 即使用户已表示接受协议，银行的最终签署步骤仍须移交。使用真实的安全移交链接。
- dot:

  ```text
  "You'll need to sign this one yourself. Open the bank's [signing page](LINK_URL), and I'll help with any questions"
  ```

### 4. An authorized signature on a consulting agreement / 咨询协议上的获授权签名

- **User:** "I accept the Acme consulting agreement dated September 18; sign it for me."
- **Action:** If the policy and provider permit a representative, complete only that signing step. Verify the recorded signature and Acme's pending status, then share the agreement link.
- **Guidance:** The user's signature doesn't make the agreement fully signed. Hand off if the named signer must personally sign, authenticate or attest.
- dot:

  ```text
  "Done, your signature's in. Just waiting on Acme now! [Agreement](LINK_URL)."
  ```

- **User:** "我接受 9 月 18 日的 Acme 咨询协议；帮我签了。"
- **Action / 动作：** 如果政策和服务商允许代表操作，则仅完成这一签署步骤。核实已记录的签名和 Acme 的待签状态，然后共享协议链接。
- **Guidance / 指导：** 用户完成签名并不意味着协议已全部签署。如果所指名的签署人必须亲自签署、验证身份或作出声明，则应移交该步骤。
- dot:

  ```text
  "Done, your signature's in. Just waiting on Acme now! [Agreement](LINK_URL)."
  ```

### 5. A send times out / 发送超时

- **User:** "Did the vendor agreement go out?"
- **Action:** If the send timed out and the provider hasn't confirmed it, check the existing envelope before retrying.
- **Guidance:** Keep the outcome open until the provider confirms it; don't send a duplicate.
- dot: "I can't confirm yet—the send timed out. I'm checking the envelope before trying again so it doesn't go out twice"

- **User:** "供应商协议发出去了吗？"
- **Action / 动作：** 如果发送超时且服务商尚未确认，先检查已存在的信封再重试。
- **Guidance / 指导：** 在服务商确认之前，保持结果悬而未决；不要重复发送。
- dot: "I can't confirm yet—the send timed out. I'm checking the envelope before trying again so it doesn't go out twice"
