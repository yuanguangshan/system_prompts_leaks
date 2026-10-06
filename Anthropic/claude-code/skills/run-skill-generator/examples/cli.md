<!-- BILINGUAL-EN-ZH -->
# Example: CLI tool / 示例：CLI 工具

CLIs are the simplest case - there's usually no background process to
manage, no ports, no lifecycle. The skill focuses on **installation**,
**representative invocations**, and **testing**.

CLI 是最简单的情况——通常没有需要管理的后台进程、没有端口、没有生命周期。该技能关注的重点是**安装**、**代表性调用**和**测试**。

## What matters / 重点关注

- **How to get the binary on `PATH`.** Installed globally? Run via
  `npx`/`uv run`? Built to `./target/release/foo`? Be explicit.
  **如何把二进制文件放进 `PATH`。**是全局安装？通过 `npx`/`uv run` 运行？还是构建到了 `./target/release/foo`？要写明确。
- **Two or three example invocations** that cover the main use cases.
  Include expected output so a reader can tell it worked.
  **两到三个覆盖主要用例的示例调用。**附上预期输出，让读者能判断它确实运行成功。
- **Exit codes** if they're meaningful (e.g. linter returns 1 on findings).
  **退出码**（如果有意义的话，例如 linter 发现问题时返回 1）。
- **Stdin behavior** if the tool reads from stdin.
  **标准输入行为**（如果该工具会从 stdin 读取）。

## Example snippet / 示例片段

> ---
> name: run-mytool
> description: Build, install, and run mytool. Use when asked to run mytool, test it, or verify it's installed correctly.
> description：构建、安装并运行 mytool。当被要求运行 mytool、测试它或验证其已正确安装时使用。
> ---
>
> ## Setup / 环境搭建
>
> ```bash
> pip install -e .
> ```
>
> This puts `mytool` on PATH. Verify:
> 这会把 `mytool` 放到 PATH 上。验证方式：
>
> ```bash
> mytool --version
> # -> mytool 0.3.1
> ```
>
> ## Run / 运行
>
> Process a single file:
> 处理单个文件：
>
> ```bash
> mytool process input.json
> # -> Processed 42 records, wrote output.json
> ```
>
> Read from stdin, write to stdout:
> 从 stdin 读取，写入 stdout：
>
> ```bash
> cat input.json | mytool process -
> ```
>
> Lint a directory (exits non-zero on problems):
> 对目录执行 lint（发现问题时以非零值退出）：
>
> ```bash
> mytool lint ./src
> echo $?  # 0 if clean, 1 if issues found
> ```
>
> ## Test / 测试
>
> ```bash
> pytest
> ```

## Keep it short / 保持精简

A CLI's run skill can be very compact. Don't pad it with every flag -
the `--help` output covers that. Just show enough that an agent can
(a) build it, (b) confirm it works, (c) run the tests.

CLI 的 run 技能可以写得很紧凑。不要把每一个旗标都塞进去——`--help` 的输出已经覆盖了这些内容。只要展示出足够的信息，让智能体能够（a）构建它、（b）确认它能正常工作、（c）运行测试即可。
