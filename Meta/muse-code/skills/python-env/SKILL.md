---
name: python-env
description: Setting up or installing into a Python environment. One rule applies whether or not you read the body - the environment belongs with the project, so create it inside the project directory (.venv) or let uv manage it, and never in a scratch directory like /tmp and never by forcing an install into the system interpreter. Load the body before creating a virtualenv, choosing an installer, or writing run instructions for a Python project.
user-invocable: false
---

<!-- BILINGUAL-EN-ZH -->
# Python environments / Python 环境

The environment is part of the project, not scratch space. A user who opens the
project tomorrow, or clones it on another machine, should find the environment
where their editor, their tooling, and their habits expect it.

环境是项目的一部分，而不是临时空间。明天再次打开这个项目、或在另一台机器上克隆它的用户，应当能在其编辑器、工具链和使用习惯所预期的位置找到环境。

## Where it goes / 环境放在哪里

Create it **inside the project directory** — `.venv` at the project root is the
convention nearly every editor and tool auto-detects:

把它创建在**项目目录内部** —— 项目根目录下的 `.venv` 是几乎所有编辑器和工具都会自动识别的约定：

```
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

For a greenfield project, prefer `uv`, which manages a project-local `.venv` for
you and is much faster:

对于全新项目，优先使用 `uv`，它会替你管理项目本地的 `.venv`，而且速度快得多：

```
uv venv
uv pip install -e .      # or `uv sync` when there is a lockfile
uv run python -m yourpkg # runs in the project env without activating
```

Never put the environment in `/tmp`, `~/envs`, or any other scratch location
outside the project. It is invisible to the user's tooling, it is not what they
will look for, and on `/tmp` it is deleted out from under them.

绝不要把环境放在 `/tmp`、`~/envs` 或项目之外的任何临时位置。它对用户的工具链不可见，也不是用户会去找的地方，而且在 `/tmp` 上它会随时被系统清掉。

## PEP 668: "externally-managed-environment" / PEP 668："externally-managed-environment"

Homebrew, Debian, and Ubuntu mark the system interpreter as externally managed,
so `pip install` outside a virtualenv refuses with:

Homebrew、Debian 和 Ubuntu 把系统解释器标记为"外部管理"，因此在虚拟环境之外运行 `pip install` 会报错拒绝：

```
error: externally-managed-environment
```

That is the signal to create the project environment — not an obstacle to work
around. Do **not** pass `--break-system-packages`, set
`PIP_BREAK_SYSTEM_PACKAGES`, or delete the `EXTERNALLY-MANAGED` marker: those
mutate an interpreter the OS owns, leave the project with no environment of its
own, and can break other software on the machine.

这正是创建项目环境的信号 —— 不是需要绕开的障碍。**不要**传 `--break-system-packages`、设置 `PIP_BREAK_SYSTEM_PACKAGES` 或删除 `EXTERNALLY-MANAGED` 标记：这些做法会改动操作系统拥有的解释器，让项目没有自己的环境，还可能破坏机器上的其他软件。
【评论】该规则把 PEP 668 的拦截视为设计意图而非故障，可避免以破坏系统级 Python 为代价换取便利的常见误操作。

## Hand off commands the user can run / 交给用户可运行的命令

Write the run instructions against the project environment, so they work from a
fresh shell in the project directory:

运行说明要面向项目环境编写，使其在项目目录下的全新 shell 中即可运行：

```
source .venv/bin/activate
python -m yourpkg ...
```

or, with uv, `uv run python -m yourpkg ...`.

或者在使用 uv 时用 `uv run python -m yourpkg ...`。

Do not hand back absolute paths into an environment outside the project
(`/tmp/whatever/bin/python -m yourpkg`). Even when they work right now, they
tell the user their project has no environment of its own.

不要把指向项目之外环境的绝对路径交还给用户（例如 `/tmp/whatever/bin/python -m yourpkg`）。即使它们当前能运行，也在告诉用户其项目没有自己的环境。

## Respect what is already there / 尊重既有配置

If the project already has an environment or a declared tool — a `.venv`, a
`uv.lock`, Poetry, Pipenv, conda — use it rather than introducing a second one.
Check before creating.

如果项目已有环境或已声明的工具 —— `.venv`、`uv.lock`、Poetry、Pipenv、conda —— 就使用它，而不是再引入第二个。创建之前先检查。
