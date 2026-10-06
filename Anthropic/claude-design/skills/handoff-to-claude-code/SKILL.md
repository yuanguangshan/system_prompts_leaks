---
name: handoff-to-claude-code
description: "Developer handoff package"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Handoff to Claude Code / 交接给 Claude Code

Create a comprehensive handoff package so a developer using Claude Code can implement this design in a real codebase.

创建一份全面的交接包，使使用 Claude Code 的开发者能够在真实代码库中实现本设计。

### Steps / 步骤

1. **Create a handoff folder** in the project directory:
   在项目目录中**创建一个交接文件夹**：
   ```
   mkdir -p <project-folder>/design_handoff_<feature-name>/
   ```
   Use a descriptive feature name derived from the design (e.g., `design_handoff_onboarding_flow`, `design_handoff_settings_redesign`).
   使用从设计中派生的描述性功能名（例如 `design_handoff_onboarding_flow`、`design_handoff_settings_redesign`）。

2. **Create a README.md** in the handoff folder with the following sections:
   在交接文件夹中**创建一个 README.md**，包含以下章节：

#### README.md Structure / README.md 结构

```markdown
# Handoff: <Feature Name>

## Overview
Brief description of what this design is for and what it accomplishes.

## About the Design Files
State clearly that the files in this bundle are **design references created in HTML** — prototypes showing intended look and behavior, not production code to copy directly. Explain that the task is to **recreate these HTML designs in the target codebase's existing environment** (React, Vue, SwiftUI, native, etc.) using its established patterns and libraries — or, if no environment exists yet, to choose the most appropriate framework for the project and implement the designs there.

## Fidelity
State clearly whether the mocks/prototypes created in this conversation are:
- **High-fidelity (hifi)**: Pixel-perfect mockups with final colors, typography, spacing, and interactions. The developer should recreate the UI pixel-perfectly using the codebase's existing libraries and patterns.
- **Low-fidelity (lofi)**: Wireframes or rough layouts showing structure and flow. The developer should use these as a guide for layout and functionality but apply the codebase's existing design system for styling.

## Screens / Views
For each screen or view in the design:
- **Name**: What this screen is called
- **Purpose**: What the user does here
- **Layout**: Detailed description of the layout (grid structure, flex directions, widths, heights, margins, padding)
- **Components**: List each UI component with:
  - Position and size
  - Colors (exact hex values if hifi)
  - Typography (font family, size, weight, line-height, letter-spacing)
  - Border radius, shadows, borders
  - Hover/active/focus states
  - Content/copy (exact text used)

## Interactions & Behavior
- Click handlers and navigation flows
- Animations and transitions (duration, easing, properties)
- Hover states
- Loading states
- Error states
- Form validation rules
- Responsive behavior (if applicable)

## State Management
- What state variables are needed
- State transitions and their triggers
- Any data fetching requirements

## Design Tokens
List all design values used:
- Colors (with hex values)
- Spacing scale
- Typography scale
- Border radius values
- Shadow values

## Assets
List any images, icons, or other assets used in the design and where they came from.

## Files
List the HTML/CSS/JS files in the project that contain the design, so the developer can reference them.
```

3. **Copy relevant design files** into the handoff folder (the HTML prototypes, any component files, etc.)
   **将相关设计文件复制**到交接文件夹中（HTML 原型、所有组件文件等）

4. **Use the `present_fs_item_for_download` tool** with the handoff folder path so the user can download it as a zip.
   **使用 `present_fs_item_for_download` 工具**并传入交接文件夹路径，让用户能将其下载为 zip。

### Important Notes / 重要注意事项

- Be extremely precise about measurements, colors, and typography — the developer will rely on this documentation
  对尺寸、颜色和排版要极其精确——开发者将依赖这份文档
- Make sure the README states up front that the bundled HTML files are **design references**, and that the user's described behavior should be understood as recreating those designs in the target app's existing environment (or the best choice of framework if none exists yet) — not shipping the HTML directly
  确保 README 开篇即说明捆绑的 HTML 文件是**设计参考**，且用户描述的行为应理解为在目标应用的现有环境中重现这些设计（若尚无环境则选择最合适的框架）——而不是直接交付 HTML
- If the design uses Anthropic brand assets, mention that they should use the existing brand system in their codebase
  如果设计使用了 Anthropic 品牌资产，要说明他们应使用其代码库中已有的品牌系统
- After creating, ask user if they want screenshots of the designs to be included. Don't include them by default.
  创建完成后，询问用户是否希望将设计截图包含在内。默认不包含。
- The README should be self-sufficient — a developer who wasn't in this conversation should be able to implement the design from the README alone
  README 应当自洽完备——未参与本对话的开发者仅凭 README 就能实现该设计
