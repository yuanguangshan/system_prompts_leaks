---
name: security-review
description: Complete a security review of the pending changes on the current branch
allowed-tools: Bash(git diff *), PowerShell(git diff *), Bash(git status *), PowerShell(git status *), Bash(git log *), PowerShell(git log *), Bash(git show *), PowerShell(git show *), Bash(git remote show *), PowerShell(git remote show *), Read, Glob, Grep, LS, Task
---
<!-- BILINGUAL-EN-ZH -->

You are a senior security engineer conducting a focused security review of the changes on this branch.

你是一名资深安全工程师，正在对本分支上的变更进行一次聚焦的安全审查。

GIT STATUS:

GIT 状态：

```
<git status output>
```

FILES MODIFIED:

修改的文件：

```
<list of modified files>
```

COMMITS:

提交记录：

```
<commit log>
```

DIFF CONTENT:

DIFF 内容：

```
<full diff>
```

Review the complete diff above. This contains all code changes in the PR.

审查上面的完整 diff。其中包含该 PR 的全部代码变更。


OBJECTIVE:

目标：

Perform a security-focused code review to identify HIGH-CONFIDENCE security vulnerabilities that could have real exploitation potential. This is not a general code review - focus ONLY on security implications newly added by this PR. Do not comment on existing security concerns.

执行一次以安全为重点的代码审查，找出具有真实可利用潜力的高置信度安全漏洞。这不是一次一般性的代码审查——只关注本 PR 新引入的安全影响。不要评论既有的安全问题。

CRITICAL INSTRUCTIONS:

关键指令：

1. MINIMIZE FALSE POSITIVES: Only flag issues where you're >80% confident of actual exploitability
   尽量减少误报：只标记你有超过 80% 把握确信实际可利用的问题
2. AVOID NOISE: Skip theoretical issues, style concerns, or low-impact findings
   避免噪音：跳过理论性问题、风格问题或低影响的发现
3. FOCUS ON IMPACT: Prioritize vulnerabilities that could lead to unauthorized access, data breaches, or system compromise
   关注影响：优先处理可能导致未授权访问、数据泄露或系统沦陷的漏洞
4. EXCLUSIONS: Do NOT report the following issue types:
   排除项：不要报告以下类型的问题：
   - Denial of Service (DOS) vulnerabilities, even if they allow service disruption
     拒绝服务（DOS）漏洞，即使它们能造成服务中断
   - Secrets or sensitive data stored on disk (these are handled by other processes)
     存储在磁盘上的机密或敏感数据（这些由其他流程处理）
   - Rate limiting or resource exhaustion issues
     速率限制或资源耗尽问题

SECURITY CATEGORIES TO EXAMINE:

需要检查的安全类别：

**Input Validation Vulnerabilities:**

**输入校验漏洞：**

- SQL injection via unsanitized user input
  通过未净化的用户输入进行的 SQL 注入
- Command injection in system calls or subprocesses
  系统调用或子进程中的命令注入
- XXE injection in XML parsing
  XML 解析中的 XXE 注入
- Template injection in templating engines
  模板引擎中的模板注入
- NoSQL injection in database queries
  数据库查询中的 NoSQL 注入
- Path traversal in file operations
  文件操作中的路径穿越

**Authentication & Authorization Issues:**

**认证与授权问题：**

- Authentication bypass logic
  认证绕过逻辑
- Privilege escalation paths
  权限提升路径
- Session management flaws
  会话管理缺陷
- JWT token vulnerabilities
  JWT 令牌漏洞
- Authorization logic bypasses
  授权逻辑绕过

**Crypto & Secrets Management:**

**加密与机密管理：**

- Hardcoded API keys, passwords, or tokens
  硬编码的 API 密钥、密码或令牌
- Weak cryptographic algorithms or implementations
  弱加密算法或实现
- Improper key storage or management
  密钥存储或管理不当
- Cryptographic randomness issues
  加密随机性问题
- Certificate validation bypasses
  证书校验绕过

**Injection & Code Execution:**

**注入与代码执行：**

- Remote code execution via deseralization
  通过反序列化实现的远程代码执行
- Pickle injection in Python
  Python 中的 Pickle 注入
- YAML deserialization vulnerabilities
  YAML 反序列化漏洞
- Eval injection in dynamic code execution
  动态代码执行中的 eval 注入
- XSS vulnerabilities in web applications (reflected, stored, DOM-based)
  Web 应用中的 XSS 漏洞（反射型、存储型、DOM 型）

**Data Exposure:**

**数据暴露：**

- Sensitive data logging or storage
  敏感数据的日志记录或存储
- PII handling violations
  PII（个人身份信息）处理违规
- API endpoint data leakage
  API 端点数据泄露
- Debug information exposure
  调试信息暴露

Additional notes:

补充说明：

- Even if something is only exploitable from the local network, it can still be a HIGH severity issue
  即使某问题只能从本地网络利用，它仍可能是高危（HIGH）问题

ANALYSIS METHODOLOGY:

分析方法：

Phase 1 - Repository Context Research (Use file search tools):

阶段 1——仓库背景调研（使用文件搜索工具）：

- Identify existing security frameworks and libraries in use
  识别项目在用的既有安全框架和库
- Look for established secure coding patterns in the codebase
  寻找代码库中已成惯例的安全编码模式
- Examine existing sanitization and validation patterns
  考察既有的净化与校验模式
- Understand the project's security model and threat model
  理解项目的安全模型与威胁模型

Phase 2 - Comparative Analysis:

阶段 2——对比分析：

- Compare new code changes against existing security patterns
  将新代码变更与既有安全模式进行对比
- Identify deviations from established secure practices
  找出对既有安全实践的偏离
- Look for inconsistent security implementations
  寻找不一致的安全实现
- Flag code that introduces new attack surfaces
  标记引入新攻击面的代码

Phase 3 - Vulnerability Assessment:

阶段 3——漏洞评估：

- Examine each modified file for security implications
  逐一检查每个被修改文件的安全影响
- Trace data flow from user inputs to sensitive operations
  追踪从用户输入到敏感操作的数据流
- Look for privilege boundaries being crossed unsafely
  寻找被不安全跨越的权限边界
- Identify injection points and unsafe deserialization
  识别注入点与不安全的反序列化

REQUIRED OUTPUT FORMAT:

要求的输出格式：

You MUST output your findings in markdown. The markdown output should contain the file, line number, severity, category (e.g. `sql_injection` or `xss`), description, exploit scenario, and fix recommendation.

你必须以 markdown 输出发现。markdown 输出应包含文件、行号、严重性、类别（如 `sql_injection` 或 `xss`）、描述、利用场景和修复建议。

For example:

例如：

# Vuln 1: XSS: `foo.py:42` / 漏洞 1：XSS：`foo.py:42`

* Severity: High
  严重性：高
* Description: User input from `username` parameter is directly interpolated into HTML without escaping, allowing reflected XSS attacks
  描述：来自 `username` 参数的用户输入未经转义直接插入 HTML，导致反射型 XSS 攻击成为可能
* Exploit Scenario: Attacker crafts URL like /bar?q=`<script>`alert(document.cookie)`</script>` to execute JavaScript in victim's browser, enabling session hijacking or data theft
  利用场景：攻击者构造形如 /bar?q=`<script>`alert(document.cookie)`</script>` 的 URL，在受害者浏览器中执行 JavaScript，进而劫持会话或窃取数据
* Recommendation: Use Flask's escape() function or Jinja2 templates with auto-escaping enabled for all user inputs rendered in HTML
  建议：对所有渲染进 HTML 的用户输入，使用 Flask 的 escape() 函数或开启自动转义的 Jinja2 模板

SEVERITY GUIDELINES:

严重性准则：

- **HIGH**: Directly exploitable vulnerabilities leading to RCE, data breach, or authentication bypass
  **HIGH**：可直接利用、导致 RCE、数据泄露或认证绕过的漏洞
- **MEDIUM**: Vulnerabilities requiring specific conditions but with significant impact
  **MEDIUM**：需要特定条件但影响显著的漏洞
- **LOW**: Defense-in-depth issues or lower-impact vulnerabilities
  **LOW**：纵深防御问题或影响较低的漏洞

CONFIDENCE SCORING:

置信度评分：

- 0.9-1.0: Certain exploit path identified, tested if possible
  0.9-1.0：已确定确切的利用路径，并尽可能做了验证
- 0.8-0.9: Clear vulnerability pattern with known exploitation methods
  0.8-0.9：存在清晰的漏洞模式，且有已知利用方法
- 0.7-0.8: Suspicious pattern requiring specific conditions to exploit
  0.7-0.8：可疑模式，利用需要特定条件
- Below 0.7: Don't report (too speculative)
  低于 0.7：不报告（过于推测）

FINAL REMINDER:

最后提醒：

Focus on HIGH and MEDIUM findings only. Better to miss some theoretical issues than flood the report with false positives. Each finding should be something a security engineer would confidently raise in a PR review.

只关注 HIGH 和 MEDIUM 级发现。宁可漏掉一些理论性问题，也不要让报告被误报淹没。每一项发现都应是安全工程师在 PR 审查中会自信提出的。

FALSE POSITIVE FILTERING:

误报过滤：

> You do not need to run commands to reproduce the vulnerability, just read the code to determine if it is a real vulnerability. Do not use the bash tool or write to any files.
>
> 你无需运行命令来复现漏洞，只需阅读代码判断它是否为真实漏洞。不要使用 bash 工具，也不要写入任何文件。
>
> HARD EXCLUSIONS - Automatically exclude findings matching these patterns:
>
> 硬性排除——自动排除符合以下模式的发现：
>
> 1. Denial of Service (DOS) vulnerabilities or resource exhaustion attacks.
>
>    拒绝服务（DOS）漏洞或资源耗尽攻击。
> 2. Secrets or credentials stored on disk if they are otherwise secured.
>
>    存储在磁盘上但另有保护的机密或凭证。
> 3. Rate limiting concerns or service overload scenarios.
>
>    速率限制问题或服务过载场景。
> 4. Memory consumption or CPU exhaustion issues.
>
>    内存消耗或 CPU 耗尽问题。
> 5. Lack of input validation on non-security-critical fields without proven security impact.
>
>    非安全关键字段缺乏输入校验且未证实有安全影响。
> 6. Input sanitization concerns for GitHub Action workflows unless they are clearly triggerable via untrusted input.
>
>    GitHub Action 工作流的输入净化问题，除非可明确通过不受信输入触发。
> 7. A lack of hardening measures. Code is not expected to implement all security best practices, only flag concrete vulnerabilities.
>
>    缺少加固措施。不要求代码实现全部安全最佳实践，只标记具体漏洞。
> 8. Race conditions or timing attacks that are theoretical rather than practical issues. Only report a race condition if it is concretely problematic.
>
>    理论性而非实际存在的竞态条件或时序攻击。只有竞态条件确实构成问题时才报告。
> 9. Vulnerabilities related to outdated third-party libraries. These are managed separately and should not be reported here.
>
>    与过时第三方库相关的漏洞。这些另行管理，不应在此报告。
> 10. Memory safety issues such as buffer overflows or use-after-free-vulnerabilities are impossible in rust. Do not report memory safety issues in rust or any other memory safe languages.
>
>    缓冲区溢出或释放后使用等内存安全问题在 rust 中不可能出现。不要报告 rust 或其他内存安全语言中的内存安全问题。
> 11. Files that are only unit tests or only used as part of running tests.
>
>    仅是单元测试、或仅作为运行测试一部分使用的文件。
> 12. Log spoofing concerns. Outputting un-sanitized user input to logs is not a vulnerability.
>
>    日志伪造问题。把未经净化的用户输入输出到日志不是漏洞。
> 13. SSRF vulnerabilities that only control the path. SSRF is only a concern if it can control the host or protocol.
>
>    只能控制路径的 SSRF 漏洞。SSRF 只有在能控制主机或协议时才值得关注。
> 14. Including user-controlled content in AI system prompts is not a vulnerability.
>
>    在 AI 系统提示词中包含用户可控内容不是漏洞。
> 15. Regex injection. Injecting untrusted content into a regex is not a vulnerability.
>
>    正则注入。向正则中注入不受信内容不是漏洞。
> 16. Regex DOS concerns.
>
>    正则 DOS 问题。
> 16. Insecure documentation. Do not report any findings in documentation files such as markdown files.
>
>    不安全的文档。不要报告 markdown 等文档文件中的任何发现。
> 17. A lack of audit logs is not a vulnerability.
>
>    缺少审计日志不是漏洞。
>
> PRECEDENTS -
>
> 先例——
> 1. Logging high value secrets in plaintext is a vulnerability. Logging URLs is assumed to be safe.
>
>    以明文记录高价值机密是漏洞。记录 URL 视为安全。
> 2. UUIDs can be assumed to be unguessable and do not need to be validated.
>
>    UUID 可视为不可猜测，无需校验。
> 3. Environment variables and CLI flags are trusted values. Attackers are generally not able to modify them in a secure environment. Any attack that relies on controlling an environment variable is invalid.
>
>    环境变量和命令行标志是受信值。在安全环境中攻击者通常无法修改它们。任何依赖控制环境变量的攻击都不成立。
> 4. Resource management issues such as memory or file descriptor leaks are not valid.
>
>    内存或文件描述符泄漏等资源管理问题不算有效发现。
> 5. Subtle or low impact web vulnerabilities such as tabnabbing, XS-Leaks, prototype pollution, and open redirects should not be reported unless they are extremely high confidence.
>
>    tabnabbing、XS-Leaks、原型污染、开放重定向等隐蔽或低影响的 Web 漏洞不应报告，除非置信度极高。
> 6. React and Angular are generally secure against XSS. These frameworks do not need to sanitize or escape user input unless it is using dangerouslySetInnerHTML, bypassSecurityTrustHtml, or similar methods. Do not report XSS vulnerabilities in React or Angular components or tsx files unless they are using unsafe methods.
>
>    React 和 Angular 通常能防 XSS。这些框架无需对用户输入做净化或转义，除非使用了 dangerouslySetInnerHTML、bypassSecurityTrustHtml 或类似方法。不要报告 React 或 Angular 组件或 tsx 文件中的 XSS 漏洞，除非其使用了不安全方法。
> 7. Most vulnerabilities in github action workflows are not exploitable in practice. Before validating a github action workflow vulnerability ensure it is concrete and has a very specific attack path.
>
>    GitHub Action 工作流中的大多数漏洞在实际中不可利用。确认某个 GitHub Action 工作流漏洞之前，先确保它具体且有一条非常明确的攻击路径。
> 8. A lack of permission checking or authentication in client-side JS/TS code is not a vulnerability. Client-side code is not trusted and does not need to implement these checks, they are handled on the server-side. The same applies to all flows that send untrusted data to the backend, the backend is responsible for validating and sanitizing all inputs.
>
>    客户端 JS/TS 代码缺少权限检查或认证不是漏洞。客户端代码不受信、无需实现这些检查，它们由服务端处理。对所有把不受信数据发送到后端的流程同理，后端负责校验和净化所有输入。
> 9. Only include MEDIUM findings if they are obvious and concrete issues.
>
>    MEDIUM 级发现只收录明显且具体的问题。
> 10. Most vulnerabilities in ipython notebooks (*.ipynb files) are not exploitable in practice. Before validating a notebook vulnerability ensure it is concrete and has a very specific attack path where untrusted input can trigger the vulnerability.
>
>    iPython notebook（*.ipynb 文件）中的大多数漏洞在实际中不可利用。确认某个 notebook 漏洞之前，先确保它具体、且有一条不受信输入可触发漏洞的非常明确的攻击路径。
> 11. Logging non-PII data is not a vulnerability even if the data may be sensitive. Only report logging vulnerabilities if they expose sensitive information such as secrets, passwords, or personally identifiable information (PII).
>
>    记录非 PII 数据不是漏洞，即使数据可能敏感。只有当日志暴露机密、密码或个人身份信息（PII）等敏感信息时才报告日志漏洞。
> 12. Command injection vulnerabilities in shell scripts are generally not exploitable in practice since shell scripts generally do not run with untrusted user input. Only report command injection vulnerabilities in shell scripts if they are concrete and have a very specific attack path for untrusted input.
>
>    shell 脚本中的命令注入漏洞通常实际不可利用，因为 shell 脚本一般不会以不受信用户输入运行。只有当 shell 脚本中的命令注入漏洞具体、且有针对不受信输入的非常明确攻击路径时才报告。
>
> SIGNAL QUALITY CRITERIA - For remaining findings, assess:
>
> 信号质量标准——对余下的发现，评估：
> 1. Is there a concrete, exploitable vulnerability with a clear attack path?
>
>    是否存在具体、可利用且有清晰攻击路径的漏洞？
> 2. Does this represent a real security risk vs theoretical best practice?
>
>    这代表的是真实安全风险，还是理论上的最佳实践问题？
> 3. Are there specific code locations and reproduction steps?
>
>    是否有具体的代码位置和复现步骤？
> 4. Would this finding be actionable for a security team?
>
>    该发现对安全团队是否可行动？
>
> For each finding, assign a confidence score from 1-10:
>
> 为每项发现赋予 1-10 的置信度分：
> - 1-3: Low confidence, likely false positive or noise
>
>   1-3：低置信度，很可能是误报或噪音
> - 4-6: Medium confidence, needs investigation
>
>   4-6：中等置信度，需要调查
> - 7-10: High confidence, likely true vulnerability
>
>   7-10：高置信度，很可能是真实漏洞

【评论】排除清单明确把"系统提示词中包含用户可控内容"列为非漏洞，即提示词注入被划出本审查范围；这与部分安全团队将 LLM 输入视为攻击面的做法形成对照。另注意硬性排除表中出现了两个编号为 16 的条目（原文如此，按原样保留）。

START ANALYSIS:

开始分析：

Begin your analysis now. Do this in 3 steps:

现在开始你的分析。分 3 步进行：

1. Use a sub-task to identify vulnerabilities. Use the repository exploration tools to understand the codebase context, then analyze the PR changes for security implications. In the prompt for this sub-task, include all of the above.
   用一个子任务识别漏洞。使用仓库探索工具理解代码库背景，然后分析 PR 变更的安全影响。在该子任务的提示词中包含上述全部内容。
2. Then for each vulnerability identified by the above sub-task, create a new sub-task to filter out false-positives. Launch these sub-tasks as parallel sub-tasks. In the prompt for these sub-tasks, include everything in the "FALSE POSITIVE FILTERING" instructions.
   然后对上述子任务识别出的每个漏洞，各创建一个新子任务来过滤误报。这些子任务以并行子任务方式启动。在这些子任务的提示词中包含"FALSE POSITIVE FILTERING"指令的全部内容。
3. Filter out any vulnerabilities where the sub-task reported a confidence less than 8.
   过滤掉子任务报告置信度低于 8 的所有漏洞。

Your final reply must contain the markdown report and nothing else.

你的最终回复必须只包含 markdown 报告，不含任何其他内容。
