---
description: Art direction for every media.generate_image prompt in a deck, so imagery reads as one consistent visual system.
---
<!-- BILINGUAL-EN-ZH -->
# Image generation directive / 图像生成指令

Every `media.generate_image` prompt for a deck must be **art-directed**, not a bare subject, so the
imagery reads as one consistent, magazine-grade visual system instead of random stock. The
palette values (`primary`, `accent`, `paper`) and the deck's `image_style` come from the
`StylePlan` you were handed.

幻灯片组的每一个 `media.generate_image` 提示词都必须经过**艺术指导**，而不是一个光秃秃的主题词，这样图像才能呈现为一个统一的、杂志级的视觉系统，而不是随机的素材图。配色值（`primary`、`accent`、`paper`）和幻灯片组的 `image_style` 来自交给你的 `StylePlan`。

Build each prompt from these parts, in order, then combine them into one sentence. Set
`media.generate_image`'s `output_dir` to the exact `artifact_media_dir` string from your contract
(never a bare `.src/media/`, which would resolve against the home directory), then embed the
result as a `data:` URI.

按顺序用以下要素构建每个提示词，然后把它们组合成一句话。把 `media.generate_image` 的 `output_dir` 设为你的契约中确切的 `artifact_media_dir` 字符串（绝不要用裸的 `.src/media/`，那会相对主目录解析），然后把结果作为 `data:` URI 嵌入。

1. **Medium**: pick the one that fits the deck and keep it consistent across every slide:
   polished editorial photography for real-world or business topics (pitch, QBR, travel,
   product, food, real estate); a single illustration style (flat vector, watercolor,
   isometric, or 3D render, choose one) for playful, kids, gaming, wellness, or conceptual
   topics. Honor the deck's mood: the `StylePlan.image_style` line.

   **媒介（Medium）**：挑选适合该幻灯片组的一种，并在每张幻灯片上保持一致：现实或商业主题（融资演讲、QBR、旅行、产品、美食、房地产）使用精致的编辑摄影；活泼、儿童、游戏、健康或概念性主题使用单一插画风格（平面矢量、水彩、等轴测或 3D 渲染，选择其一）。遵循幻灯片组的情绪基调：即 `StylePlan.image_style` 一行。

2. **Subject**: the slide's concrete subject. For a how-to, educational, or concept slide,
   the image must clearly **demonstrate** that specific concept (an actual photo exhibiting
   "leading lines" or "rule of thirds", not a generic related scene) so the slide teaches at a
   glance.

   **主题（Subject）**：幻灯片的具体主题。对于教程类、教学类或概念类幻灯片，图像必须清楚地**展示**该具体概念（一张真正体现"引导线"或"三分法"的照片，而不是泛泛的相关场景），让幻灯片一目了然地完成讲解。

3. **Composition**: decide the image's **role** first, because it sets the framing:

   **构图（Composition）**：先决定图像的**角色**，因为它决定了取景方式：

   - **Full-bleed / hero** (cover, section, statement, closing, or full-screen image slide
     where the title/text *overlays* the photo): keep the subject and visual detail to one
     side or the upper area and leave large, calm, low-detail **negative space** along at
     least one side and the lower edge (the usual title zones) so the overlaid title reads
     cleanly. Rule-of-thirds; never a busy subject dead-center where a title lands.

     **满版/主视觉（Full-bleed / hero）**（封面、章节、宣言、结尾或全屏图像幻灯片，标题/文字*叠加*在照片上）：把主体和视觉细节放在一侧或上部，沿至少一侧和下缘（通常是标题区）留出大面积、平静、细节稀少的**负空间**，使叠加的标题清晰可读。遵循三分法；绝不在标题落点处以正中位置放置繁杂主体。

   - **Content / cell** (fills a two-column half, a bento/gallery cell, or a lockup image
     column, where the text sits in a *separate panel beside* the image): do the **opposite**,
     fill the frame with the subject, prominent and roughly centered, and do **not** reserve
     empty negative space. No text overlays it, so reserved space just crops into the cell as
     dead space; a tightly-composed, subject-filling frame crops cleanly under `object-fit:cover`.

     **内容/单元格（Content / cell）**（填充双栏的一半、bento/画廊单元格或组合图列，文字位于图像*旁边的独立面板中*）：做法**相反**，让主体充满画面，突出且大致居中，**不要**预留空白负空间。没有文字叠加在它上面，预留的空间只会成为单元格里的死角；构图紧凑、主体充满的画面在 `object-fit:cover` 裁切下效果干净。

4. **Color and value**: grade the image toward the palette (`primary`, `accent`) so it
   harmonizes, **but** keep a clear, well-exposed focal subject with strong contrast and tonal
   depth. **Never wash the image out**: no foggy, hazy, overexposed, blown-out, near-white,
   milky, flat, or low-contrast results, and never let the subject fade into the background.
   Match the image's brightness family to the slide background (rich low-key on a dark slide;
   bright with real tonal depth on a light slide), but a visible high-contrast subject always
   wins over blending. Avoid colors that clash with the palette.

   **色彩与明暗（Color and value）**：让图像向配色（`primary`、`accent`）调色以保持和谐，**但**要保持清晰、曝光良好的焦点主体，具有强对比和影调深度。**绝不要把图像洗白**：不要出现雾蒙蒙、朦胧、过曝、死白、接近白色、奶白、平淡或低对比的效果，绝不让主体淡入背景。使图像的明暗基调与幻灯片背景匹配（深色幻灯片用浓郁的低调影调；浅色幻灯片用明亮且有真实影调深度的画面），但清晰可见的高对比主体永远优先于融合。避免使用与配色冲突的颜色。

5. **Quality**: polished, never a generic stock look. Photography: soft
   intentional lighting, shallow depth of field, fine detail, sharp focus. Illustration: clean
   shapes, consistent line weight, balanced layout. Request a **high-resolution** image (long
   edge ≥1536px) that stays crisp filling a full-bleed slide, never soft, blurry, upscaled,
   pixelated, or low-detail. A **sourced** photo meets the same bar: verify the downloaded
   file's pixel size (`gm identify`), and a candidate below that long-edge floor is rejected
   for hero or full-bleed use rather than upscaled.

   **质量（Quality）**：精致，绝不要平庸的素材图观感。摄影：柔和而有意图的布光、浅景深、细节丰富、对焦锐利。插画：形状干净、线宽一致、布局均衡。请求**高分辨率**图像（长边 ≥1536px），在铺满满版幻灯片时依然清晰，绝不发软、模糊、放大凑数、像素化或细节不足。**检索来源**的照片要达到同样标准：验证下载文件的像素尺寸（`gm identify`），长边低于该下限的候选图直接弃用于主视觉或满版用途，而不是放大凑数。

6. **Aspect ratio**: request the panel's ratio: tall/portrait for a side panel, 16:9 for a
   full-bleed cover, so the image fills its container under `object-fit:cover` with minimal
   cropping.

   **宽高比（Aspect ratio）**：按面板的宽高比请求：侧栏用竖幅，满版封面用 16:9，使图像在 `object-fit:cover` 下以最小裁切填满容器。

7. **Exclusions**: no text, letters, numbers, equations, formulas, axis labels, legends,
   captions, watermarks, logos, dashboards, or UI in the image (render charts with the
   matplotlib path and any labels as styled HTML instead); no clutter in the text zone.

   **排除项（Exclusions）**：图像中不出现文字、字母、数字、等式、公式、坐标轴标签、图例、说明文字、水印、徽标、仪表盘或 UI（图表改用 matplotlib 路径渲染，标签用带样式的 HTML 呈现）；文字区内不留杂乱元素。

## Examples / 示例

**Hero** (title overlays the photo, so leave negative space):  
**主视觉（Hero）**（标题叠加在照片上，因此要留负空间）：  
> editorial photograph of a misty mountain lake at dawn, subject in the upper-right third with  
> calm empty water and sky in the lower-left for the title, soft golden side light, muted  
> palette grading toward the deck's paper tone, shallow depth of field, fine detail, 16:9

> 一张黎明时分薄雾笼罩的山间湖泊的编辑风摄影照片，主体位于右上三分区，左下留出平静空旷的水面和天空用于放置标题，柔和的金色侧光，调色偏向幻灯片组的纸色调，浅景深，细节丰富，16:9

**Content cell** (text sits beside it, so fill the frame):  
**内容单元格（Content cell）**（文字位于图像旁边，因此要充满画面）：  
> editorial photograph of a golden retriever puppy in a sunlit meadow, subject filling the  
> frame and centered, soft natural light, shallow depth of field, fine detail, tall 4:5  
> portrait for a side column

> 一张阳光照耀的草地上金毛幼犬的编辑风摄影照片，主体充满画面且居中，柔和的自然光，浅景深，细节丰富，供侧栏使用的 4:5 竖幅

## When to generate / 何时生成

Generate the cover hero by default when its subject depicts nothing real; when the cover subject
is a real thing (a company, product, place, or person), source it with the image-search skill
instead. Generate content images where they illustrate a concept (see `authoring.md` → Images). Metric, statement, and financial slides stay typographic /
data-driven; do not generate an image just to raise the count.

当封面主视觉的主体不描绘任何真实事物时，默认使用生成；当封面主体是真实事物（公司、产品、地点或人物）时，改用 image-search 技能获取。在内容图像用来说明概念之处生成它们（参见 `authoring.md` → Images）。指标页、宣言页和财务页保持排版/数据驱动；不要为了凑数量而生成图像。

【评论】排除条款要求生成图像中不出现任何文字与 UI，是对图像生成模型易产生乱码文字这一常见缺陷的针对性规避；真实事物改用图片检索则是可用性与事实准确性的折中。
