---
name: durable-test-collateral
description: Load the body before the first implementation edit for a bug fix or behavior change, or when the user or governing specification makes reusable tests a deliverable. It keeps the smallest focused test with the delivered project and updates coverage when behavior changes. Do not load for greenfield work unless reusable tests are explicitly required, behavior-preserving refactors, verification-only work, plan/review/explanation/documentation/formatting/generated-output-only work, explicit no-change or no-tool work, or when the user declines tests.
user-invocable: false
---

<!-- BILINGUAL-EN-ZH -->
# Durable Test Collateral / 持久测试资产

Verification is part of the delivered project when it can prevent the same
behavior from breaking again. Use the project's existing framework and
conventions.

当验证能够防止同一行为再次被破坏时，它就是交付项目的一部分。使用项目已有的框架与约定。

## Keep the check with the project / 把检查留在项目里

For a bug fix or behavior change, add or update the smallest maintained test in
the repository's normal test layout that exercises the changed behavior. If a
specification makes a reusable suite or test category a deliverable, implement
that suite instead of treating it as private verification.

对于缺陷修复或行为变更，在仓库常规测试布局中添加或更新能覆盖所改行为的最小维护中测试。如果某规范把可复用测试套件或测试类别定为交付物，就实现那个套件，而不是把它当作私有验证。

An inline interpreter command, heredoc, scratch script, temporary-directory
test, demo, or manual probe may help diagnose the problem, but it does not
replace the maintained test. Keep a useful self-authored test unless the user
explicitly asks for a disposable probe.

内联解释器命令、heredoc、临时脚本、临时目录测试、演示或手动探查可能有助于诊断问题，但不能替代维护中的测试。保留有用的自写测试，除非用户明确要求一次性探查。

Do not add a new test framework solely to satisfy this skill. If the repository
has no maintained harness and the request does not require one, disclose when
durable coverage would be disproportionate or infeasible and use the narrowest
honest verification instead.

不要只为满足本技能而引入新的测试框架。如果仓库没有维护中的测试框架且请求也不要求，则在持久化覆盖会不成比例或不可行时如实说明，并改用范围最窄的诚实验证。

On a later turn, update the maintained test even when the old suite remains
green and the user does not repeat the test request. Cover the new behavior,
not merely an unrelated cleanup in the test file.

在后续轮次中，即使旧套件保持通过、用户也没有再次提出测试要求，也要更新维护中的测试。覆盖新行为，而不只是在测试文件里做无关的清理。

## Prove the maintained check / 证明维护测试有效

Observe the authentic failure before the implementation change when it is safe
and runnable. Then make the smallest implementation change and run the focused
maintained test after the final relevant source or test edit.

在安全且可运行的前提下，先在实现变更之前观察到真实失败。然后做最小的实现改动，并在最后一次相关源码或测试编辑之后运行聚焦的维护测试。

Do not weaken or delete a real failing test to obtain green. Fix the product,
or explain why the test's expectation is wrong before changing that
expectation. Do not repeat an unchanged check solely to collect evidence.

不要为了让测试变绿而弱化或删除真实失败的测试。要么修复产品，要么在修改该预期之前解释为什么测试的预期是错的。不要仅为了收集证据而重复运行未改动的检查。

Run a broad or full suite only when the user, a repository gate, or
proportionate risk requires it. A focused regression test comes first; broad
proof does not substitute for missing focused coverage.

只有当用户、仓库门禁或相称的风险要求时，才运行宽范围或完整套件。聚焦的回归测试优先；宽泛的证明不能替代缺失的聚焦覆盖。

If the real runner is unavailable but the focused test can still be authored
honestly, leave the focused test in the repository and say it was not run. Do
not fabricate a passing result.

如果真实的测试运行器不可用，但聚焦测试仍可诚实编写，就把聚焦测试留在仓库中，并说明它未运行。绝不伪造通过的结果。
【评论】"先看真实失败再修"对应测试工程中的红-绿流程，用于防止修复代码并未真正命中缺陷。
