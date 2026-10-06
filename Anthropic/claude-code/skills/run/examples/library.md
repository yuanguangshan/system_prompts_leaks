<!-- BILINGUAL-EN-ZH -->
# Example: Library / SDK / 示例：库 / SDK

Libraries don't have a "run" step in the process sense - there's no
server to start, no CLI to invoke. For libraries, the run skill is about:

库在流程意义上没有"运行"环节——没有需要启动的服务器，也没有需要调用的 CLI。对库而言，run 技能的内容是：

1. **Building** the library from source
  1. **从源码构建**该库
2. **Running the test suite**
  2. **运行测试套件**
3. **A minimal working example** that exercises the library and proves
   it's installed correctly
  3. 一个能实际调用该库并证明其安装正确的**最小可运行示例**

Keep it brief. The template's Build and Test sections do most of the work.

保持简短。模板的 Build 和 Test 部分已承担大部分工作。

## The smoke-test example / 冒烟测试示例

The main library-specific addition is a tiny program (or REPL snippet)
that imports the library and does one real thing. This is how an agent
confirms "yes, the library is usable":

最主要的库特定补充内容是一个小程序（或 REPL 片段），它导入该库并完成一件真实的事情。这就是代理确认"是的，这个库可用"的方式：

> ## Verify
>
> 验证
>
> ```bash
> python -c '
> from mylib import Client
> c = Client()
> print(c.ping())
> '
> # -> pong
> ```

Or for a compiled language:

对于编译型语言：

> ```bash
> cat > /tmp/smoke.go <<GO
> package main
> import "example.com/mylib"
> func main() { println(mylib.Version()) }
> GO
> go run /tmp/smoke.go
> # -> v1.2.3
> ```

## Example snippet / 示例片段

> ---
> name: run-mylib
> description: Build, install, and test mylib from source. Use when asked to verify mylib works, run its tests, or build a distribution.
> ---
>
> `mylib` is a Python library - "running" it means building from source
> and executing the test suite.
>
> `mylib` 是一个 Python 库——"运行"它意味着从源码构建并执行测试套件。
>
> ## Setup
>
> ## 环境配置
>
> ```bash
> pip install -e '.[dev]'
> ```
>
> ## Verify
>
> ## 验证
>
> ```bash
> python -c 'import mylib; print(mylib.__version__)'
> # -> 2.1.0
> ```
>
> ## Test
>
> ## 测试
>
> ```bash
> pytest
> ```
>
> Subset of tests: `pytest tests/unit/`. With coverage: `pytest --cov=mylib`.
>
> 测试子集：`pytest tests/unit/`。带覆盖率：`pytest --cov=mylib`。
>
> ## Build (distribution)
>
> ## 构建（发行包）
>
> ```bash
> pip install build
> python -m build
> # -> dist/mylib-2.1.0-py3-none-any.whl
> ```

## Things to consider documenting / 值得记录的内容

- **Development mode vs installed mode.** `pip install -e .` vs
  `pip install .` - if behavior differs, say which to use for what.
  - **开发模式与安装模式。**`pip install -e .` 与 `pip install .` 的对比——如果行为有差异，说明各自适用于什么场景。
- **Optional dependencies.** `[dev]`, `[test]`, `[docs]` extras and when
  each is needed.
  - **可选依赖。**`[dev]`、`[test]`、`[docs]` 等附加依赖组，以及各自何时需要。
- **Generated code.** If there's a codegen step (protobuf, OpenAPI clients),
  document it - it's almost always missing from READMEs.
  - **生成的代码。**如果存在代码生成步骤（protobuf、OpenAPI 客户端），要把它记录下来——README 中几乎总是缺失这一步。
