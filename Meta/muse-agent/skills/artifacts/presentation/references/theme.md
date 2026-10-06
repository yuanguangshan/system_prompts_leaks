---
description: The theme JSON design system stored under ~/workspace/themes/, its schema, and how to generate or apply a theme.
---
<!-- BILINGUAL-EN-ZH -->

# Theme schema and generation / 主题模式（schema）与生成

A theme is a small JSON design system stored under `~/workspace/themes/`. The artifact builder's
theme decision lives in `/opt/hatch/skills/artifacts/pdf/references/visual.md`; read this file when generating or applying
a theme.

主题是存储在 `~/workspace/themes/` 下的一个小型 JSON 设计系统。工件构建器的主题决策位于 `/opt/hatch/skills/artifacts/pdf/references/visual.md`；在生成或应用主题时阅读本文件。

## Theme file schema / 主题文件模式

Each theme JSON has two blocks: `ui` (document styling) and `visual` (image-generation guidance). Example:

每个主题 JSON 包含两个块：`ui`（文档样式）和 `visual`（图像生成指引）。示例：

```json
{
  "name": "Paper & Ink", "mode": "light",
  "ui": {
    "colors": { "bg": "#FAF9F6", "surface": "#F5F0EB", "text": "#1A1A1A", "accent": "#3D3229", "textMuted": "#8B7355", "danger": "#9B2C2C", "success": "#16A34A", "warning": "#D97706" },
    "font": "Georgia, 'Times New Roman', serif", "fontSans": "system-ui, sans-serif",
    "radius": { "card": "6px", "button": "4px" },
    "shadows": { "md": "0 1px 3px rgba(61,50,41,.08)" },
    "notes": "Editorial, bookish. Uppercase headers, thin dividers, generous whitespace."
  },
  "visual": {
    "style": "1970s vintage film photography", "mood": "warm, nostalgic",
    "lighting": "golden hour", "palette": "warm golds, sun-bleached pastels",
    "texture": "35mm film grain", "references": "Slim Aarons",
    "orientation": "landscape", "filter": "contrast(1.05) saturate(0.85) sepia(0.1)"
  }
}
```

### `ui` block fields / `ui` 块字段

| Field | Purpose |
|-------|---------|
| `colors` | Full palette: bg, surface, borders, text, accent, textMuted, and semantic colors (danger, success, warning) |
| `font` / `fontSans` / `fontMono` | Typography stacks for body, UI, and code |
| `fontWeight` | Weight mapping for headings, body, and labels |
| `radius` | Border-radius tokens per element type |
| `shadows` | Box-shadow tokens (or "none" for flat themes) |
| `notes` | Theme personality and special component patterns |

| 字段 | 用途 |
|-------|---------|
| `colors` | 完整调色板：bg、surface、边框、text、accent、textMuted 以及语义色（danger、success、warning） |
| `font` / `fontSans` / `fontMono` | 正文、UI 和代码的字体栈 |
| `fontWeight` | 标题、正文和标签的字重映射 |
| `radius` | 各元素类型的圆角令牌 |
| `shadows` | 盒阴影令牌（扁平主题用 "none"） |
| `notes` | 主题个性与特殊组件样式 |

Maintain a 4.5:1 minimum contrast between text and background.

文字与背景之间保持最低 4.5:1 的对比度。

### `visual` block fields / `visual` 块字段

| Field | Purpose | Examples |
|-------|---------|---------|
| `style` | Art direction | "3D Pixar render", "watercolor", "vintage film" |
| `mood` | Emotional tone | "cozy and warm", "dark and mysterious" |
| `lighting` | Light direction | "golden hour", "dramatic chiaroscuro" |
| `palette` | Image color guidance | "jewel tones", "neon on black" |
| `texture` | Surface quality | "clean digital", "film grain" |
| `references` | Style references | "Wes Anderson", "Studio Ghibli" |
| `orientation` | Default image orientation | "landscape", "square", "vertical" |
| `filter` | CSS filter applied to `<img>` tags | "contrast(1.05) sepia(0.1)" or "none" |

| 字段 | 用途 | 示例 |
|-------|---------|---------|
| `style` | 美术方向 | "3D Pixar render"（3D 皮克斯渲染）、"watercolor"（水彩）、"vintage film"（复古胶片） |
| `mood` | 情绪基调 | "cozy and warm"（温馨暖意）、"dark and mysterious"（黑暗神秘） |
| `lighting` | 光线方向 | "golden hour"（黄金时刻）、"dramatic chiaroscuro"（戏剧性明暗对照） |
| `palette` | 图像配色指引 | "jewel tones"（宝石色）、"neon on black"（黑底霓虹） |
| `texture` | 表面质感 | "clean digital"（干净数字感）、"film grain"（胶片颗粒） |
| `references` | 风格参照 | "Wes Anderson"、"Studio Ghibli" |
| `orientation` | 默认图像方向 | "landscape"（横版）、"square"（方形）、"vertical"（竖版） |
| `filter` | 应用于 `<img>` 标签的 CSS 滤镜 | "contrast(1.05) sepia(0.1)" 或 "none" |

## Generating a theme / 生成主题

1. **Infer the `ui` block**: colors, fonts, and shapes matching the vibe. Ensure a 4.5:1 minimum contrast. Pick light or dark mode.
   **推断 `ui` 块**：与整体气质匹配的颜色、字体和形状。确保最低 4.5:1 的对比度。选定浅色或深色模式。
2. **Infer the `visual` block**: art style, mood, references, CSS filter.
   **推断 `visual` 块**：美术风格、情绪、风格参照、CSS 滤镜。
3. **Save** to `~/workspace/themes/<kebab-case-name>.json`.
   **保存**到 `~/workspace/themes/<kebab-case-name>.json`。
4. **Confirm**: "I created 'Wes Anderson': pastel pinks, Futura, cinematic film stills. Want me to use this?"
   **确认**："我创建了 'Wes Anderson' 主题：柔和粉色调、Futura 字体、电影剧照质感。需要我使用它吗？"

Capture the *feel* (color temperature, density, typography personality), not an exact layout clone.

捕捉的是*感觉*（色温、密度、排版个性），而不是对布局的精确克隆。

## Applying a theme (builder side) / 应用主题（构建器侧）

- **Fonts render offline.** Map every theme font to an installed local family (verify with `fc-list`) before using it in CSS. Do not rely on Google Fonts imports or Microsoft fonts; see `/opt/hatch/skills/artifacts/pdf/references/workflow.md` for reliable local families.
  **字体离线渲染。** 在 CSS 中使用主题字体之前，先把每个字体映射到已安装的本地字族（用 `fc-list` 验证）。不要依赖 Google Fonts 导入或 Microsoft 字体；可靠的本地字族参见 `/opt/hatch/skills/artifacts/pdf/references/workflow.md`。
- **Imagery.** When generating hero or illustrative imagery with `media.generate_image`, build prompts from the `visual` block (style, mood, lighting, palette, references), set `output_dir` to `artifact_media_dir`, and apply the theme's `filter` to the embedded images.
  **图像。** 用 `media.generate_image` 生成主视觉或插图时，从 `visual` 块（style、mood、lighting、palette、references）构建提示词，把 `output_dir` 设为 `artifact_media_dir`，并对嵌入图像应用主题的 `filter`。
