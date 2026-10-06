<!-- BILINGUAL-EN-ZH -->
**CRITICAL: You MUST complete these steps in order. Do not skip ahead to writing code.**

**关键：你必须按顺序完成这些步骤。不得跳过直接去写代码。**

【评论】开头的强制顺序约束是针对模型倾向直接写代码而跳过前置探测（是否为可填写表单）的行为纠偏。

If you need to fill out a PDF form, first check to see if the PDF has fillable form fields. Run this script from this file's directory:
 `python scripts/check_fillable_fields <file.pdf>`, and depending on the result go to either the "Fillable fields" or "Non-fillable fields" and follow those instructions.

如果你需要填写 PDF 表单，首先检查该 PDF 是否具有可填写的表单字段。在本文件所在目录运行此脚本：
 `python scripts/check_fillable_fields <file.pdf>`，然后根据结果进入"Fillable fields"（可填写字段）或"Non-fillable fields"（不可填写字段）部分并遵循相应指示。

# Fillable fields / 可填写字段

If the PDF has fillable form fields:

如果 PDF 具有可填写的表单字段：

- Run this script from this file's directory: `python scripts/extract_form_field_info.py <input.pdf> <field_info.json>`. It will create a JSON file with a list of fields in this format:

- 在本文件所在目录运行此脚本：`python scripts/extract_form_field_info.py <input.pdf> <field_info.json>`。它会创建一个 JSON 文件，以下述格式列出各字段：

```
[
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "rect": ([left, bottom, right, top] bounding box in PDF coordinates, y=0 is the bottom of the page),
    "type": ("text", "checkbox", "radio_group", or "choice"),
  },
  // Checkboxes have "checked_value" and "unchecked_value" properties:
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "type": "checkbox",
    "checked_value": (Set the field to this value to check the checkbox),
    "unchecked_value": (Set the field to this value to uncheck the checkbox),
  },
  // Radio groups have a "radio_options" list with the possible choices.
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "type": "radio_group",
    "radio_options": [
      {
        "value": (set the field to this value to select this radio option),
        "rect": (bounding box for the radio button for this option)
      },
      // Other radio options
    ]
  },
  // Multiple choice fields have a "choice_options" list with the possible choices:
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "type": "choice",
    "choice_options": [
      {
        "value": (set the field to this value to select this option),
        "text": (display text of the option)
      },
      // Other choice options
    ],
  }
]
```

- Convert the PDF to PNGs (one image for each page) with this script (run from this file's directory):
`python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>`
Then analyze the images to determine the purpose of each form field (make sure to convert the bounding box PDF coordinates to image coordinates).

- 用此脚本将 PDF 转换为 PNG（每页一张图片）（在本文件所在目录运行）：
`python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>`
然后分析这些图片，确定每个表单字段的用途（注意务必把边界框的 PDF 坐标转换为图片坐标）。

- Create a `field_values.json` file in this format with the values to be entered for each field:

- 按此格式创建一个 `field_values.json` 文件，写入每个字段待填入的值：

```
[
  {
    "field_id": "last_name", // Must match the field_id from `extract_form_field_info.py`
    "description": "The user's last name",
    "page": 1, // Must match the "page" value in field_info.json
    "value": "Simpson"
  },
  {
    "field_id": "Checkbox12",
    "description": "Checkbox to be checked if the user is 18 or over",
    "page": 1,
    "value": "/On" // If this is a checkbox, use its "checked_value" value to check it. If it's a radio button group, use one of the "value" values in "radio_options".
  },
  // more fields
]
```

- Run the `fill_fillable_fields.py` script from this file's directory to create a filled-in PDF:
`python scripts/fill_fillable_fields.py <input pdf> <field_values.json> <output pdf>`
This script will verify that the field IDs and values you provide are valid; if it prints error messages, correct the appropriate fields and try again.

- 在本文件所在目录运行 `fill_fillable_fields.py` 脚本，生成已填写的 PDF：
`python scripts/fill_fillable_fields.py <input pdf> <field_values.json> <output pdf>`
该脚本会校验你提供的字段 ID 和值是否有效；如果打印了错误信息，请修正相应字段后重试。

# Non-fillable fields / 不可填写字段

If the PDF doesn't have fillable form fields, you'll add text annotations. First try to extract coordinates from the PDF structure (more accurate), then fall back to visual estimation if needed.

如果 PDF 没有可填写的表单字段，你需要添加文本注释（annotation）。首先尝试从 PDF 结构中提取坐标（更精确），如有需要再退回到目测估计。

## Step 1: Try Structure Extraction First / 第 1 步：优先尝试结构化提取

Run this script to extract text labels, lines, and checkboxes with their exact PDF coordinates:
`python scripts/extract_form_structure.py <input.pdf> form_structure.json`

运行此脚本，提取文本标签、线条和复选框及其精确 PDF 坐标：
`python scripts/extract_form_structure.py <input.pdf> form_structure.json`

This creates a JSON file containing:

它会创建一个包含以下内容的 JSON 文件：

- **labels**: Every text element with exact coordinates (x0, top, x1, bottom in PDF points)

- **labels**：每个文本元素及其精确坐标（x0、top、x1、bottom，单位为 PDF 点）

- **lines**: Horizontal lines that define row boundaries

- **lines**：定义行边界的水平线

- **checkboxes**: Small square rectangles that are checkboxes (with center coordinates)

- **checkboxes**：作为复选框的小方块矩形（含中心坐标）

- **row_boundaries**: Row top/bottom positions calculated from horizontal lines

- **row_boundaries**：由水平线计算出的行顶部/底部位置

**Check the results**: If `form_structure.json` has meaningful labels (text elements that correspond to form fields), use **Approach A: Structure-Based Coordinates**. If the PDF is scanned/image-based and has few or no labels, use **Approach B: Visual Estimation**.

**检查结果**：如果 `form_structure.json` 包含有意义的标签（与表单字段对应的文本元素），使用**方案 A：基于结构的坐标**。如果 PDF 是扫描件/纯图像且几乎没有标签，使用**方案 B：目测估计**。

---

## Approach A: Structure-Based Coordinates (Preferred) / 方案 A：基于结构的坐标（首选）

Use this when `extract_form_structure.py` found text labels in the PDF.

当 `extract_form_structure.py` 在 PDF 中找到了文本标签时使用此方案。

### A.1: Analyze the Structure / A.1：分析结构

Read form_structure.json and identify:

阅读 form_structure.json 并识别：

1. **Label groups**: Adjacent text elements that form a single label (e.g., "Last" + "Name")

1. **标签组**：构成单个标签的相邻文本元素（例如 "Last" + "Name"）

2. **Row structure**: Labels with similar `top` values are in the same row

2. **行结构**：`top` 值相近的标签位于同一行

3. **Field columns**: Entry areas start after label ends (x0 = label.x1 + gap)

3. **字段列**：填写区域从标签结束处之后开始（x0 = label.x1 + 间距）

4. **Checkboxes**: Use the checkbox coordinates directly from the structure

4. **复选框**：直接使用结构中的复选框坐标

**Coordinate system**: PDF coordinates where y=0 is at TOP of page, y increases downward.

**坐标系**：PDF 坐标，y=0 位于页面顶部，y 向下增大。

### A.2: Check for Missing Elements / A.2：检查缺失元素

The structure extraction may not detect all form elements. Common cases:

结构化提取可能无法检测到所有表单元素。常见情况：

- **Circular checkboxes**: Only square rectangles are detected as checkboxes

- **圆形复选框**：只有方形矩形会被检测为复选框

- **Complex graphics**: Decorative elements or non-standard form controls

- **复杂图形**：装饰性元素或非标准表单控件

- **Faded or light-colored elements**: May not be extracted

- **褪色或浅色元素**：可能无法被提取

If you see form fields in the PDF images that aren't in form_structure.json, you'll need to use **visual analysis** for those specific fields (see "Hybrid Approach" below).

如果你在 PDF 图片中看到了 form_structure.json 里没有的表单字段，需要对那些特定字段使用**视觉分析**（见下文"混合方案"）。

### A.3: Create fields.json with PDF Coordinates / A.3：用 PDF 坐标创建 fields.json

For each field, calculate entry coordinates from the extracted structure:

对每个字段，从提取的结构中计算填写坐标：

**Text fields:**

**文本字段：**

- entry x0 = label x1 + 5 (small gap after label)

- 填写区 x0 = label x1 + 5（标签后留小间距）

- entry x1 = next label's x0, or row boundary

- 填写区 x1 = 下一个标签的 x0，或行边界

- entry top = same as label top

- 填写区 top = 与标签 top 相同

- entry bottom = row boundary line below, or label bottom + row_height

- 填写区 bottom = 下方行边界线，或 label bottom + row_height

**Checkboxes:**

**复选框：**

- Use the checkbox rectangle coordinates directly from form_structure.json

- 直接使用 form_structure.json 中的复选框矩形坐标

- entry_bounding_box = [checkbox.x0, checkbox.top, checkbox.x1, checkbox.bottom]

- entry_bounding_box = [checkbox.x0, checkbox.top, checkbox.x1, checkbox.bottom]

Create fields.json using `pdf_width` and `pdf_height` (signals PDF coordinates):

使用 `pdf_width` 和 `pdf_height` 创建 fields.json（表示使用 PDF 坐标）：

```json
{
  "pages": [
    {"page_number": 1, "pdf_width": 612, "pdf_height": 792}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "Last name entry field",
      "field_label": "Last Name",
      "label_bounding_box": [43, 63, 87, 73],
      "entry_bounding_box": [92, 63, 260, 79],
      "entry_text": {"text": "Smith", "font_size": 10}
    },
    {
      "page_number": 1,
      "description": "US Citizen Yes checkbox",
      "field_label": "Yes",
      "label_bounding_box": [260, 200, 280, 210],
      "entry_bounding_box": [285, 197, 292, 205],
      "entry_text": {"text": "X"}
    }
  ]
}
```

**Important**: Use `pdf_width`/`pdf_height` and coordinates directly from form_structure.json.

**重要**：使用 `pdf_width`/`pdf_height`，坐标直接取自 form_structure.json。

### A.4: Validate Bounding Boxes / A.4：校验边界框

Before filling, check your bounding boxes for errors:
`python scripts/check_bounding_boxes.py fields.json`

填写前，检查边界框是否有误：
`python scripts/check_bounding_boxes.py fields.json`

This checks for intersecting bounding boxes and entry boxes that are too small for the font size. Fix any reported errors before filling.

该检查会发现相交的边界框，以及相对字号过小的填写框。填写前先修复所有报告的错误。

---

## Approach B: Visual Estimation (Fallback) / 方案 B：目测估计（后备）

Use this when the PDF is scanned/image-based and structure extraction found no usable text labels (e.g., all text shows as "(cid:X)" patterns).

当 PDF 是扫描件/纯图像、且结构化提取未找到可用文本标签时使用此方案（例如所有文本都显示为 "(cid:X)" 模式）。

### B.1: Convert PDF to Images / B.1：将 PDF 转换为图片

`python scripts/convert_pdf_to_images.py <input.pdf> <images_dir/>`

### B.2: Initial Field Identification / B.2：初步字段识别

Examine each page image to identify form sections and get **rough estimates** of field locations:

检查每一页图片，识别表单分区，并对字段位置做**粗略估计**：

- Form field labels and their approximate positions

- 表单字段标签及其大致位置

- Entry areas (lines, boxes, or blank spaces for text input)

- 填写区域（用于文本输入的线条、方框或空白处）

- Checkboxes and their approximate locations

- 复选框及其大致位置

For each field, note approximate pixel coordinates (they don't need to be precise yet).

对每个字段，记下近似的像素坐标（此时不需要精确）。

### B.3: Zoom Refinement (CRITICAL for accuracy) / B.3：放大精修（对准确性至关重要）

For each field, crop a region around the estimated position to refine coordinates precisely.

对每个字段，在估计位置周围裁剪一个区域，以精确细化坐标。

**Create a zoomed crop using ImageMagick:**

**使用 ImageMagick 创建放大裁剪图：**

```bash
magick <page_image> -crop <width>x<height>+<x>+<y> +repage <crop_output.png>
```

Where:

其中：

- `<x>, <y>` = top-left corner of crop region (use your rough estimate minus padding)

- `<x>, <y>` = 裁剪区域左上角（用你的粗略估计值减去余量）

- `<width>, <height>` = size of crop region (field area plus ~50px padding on each side)

- `<width>, <height>` = 裁剪区域尺寸（字段区域加上每侧约 50px 的余量）

**Example:** To refine a "Name" field estimated around (100, 150):

**示例：** 要精修估计位于 (100, 150) 附近的"Name"字段：

```bash
magick images_dir/page_1.png -crop 300x80+50+120 +repage crops/name_field.png
```

(Note: if the `magick` command isn't available, try `convert` with the same arguments).

（注意：如果 `magick` 命令不可用，请尝试用相同参数使用 `convert`）。

**Examine the cropped image** to determine precise coordinates:

**检查裁剪出的图片**以确定精确坐标：

1. Identify the exact pixel where the entry area begins (after the label)

1. 确定填写区域开始处的精确像素（标签之后）

2. Identify where the entry area ends (before next field or edge)

2. 确定填写区域结束的位置（下一个字段之前或页面边缘）

3. Identify the top and bottom of the entry line/box

3. 确定填写线/框的顶部和底部

**Convert crop coordinates back to full image coordinates:**

**把裁剪图坐标换算回完整图片坐标：**

- full_x = crop_x + crop_offset_x

- full_x = crop_x + crop_offset_x

- full_y = crop_y + crop_offset_y

- full_y = crop_y + crop_offset_y

Example: If the crop started at (50, 120) and the entry box starts at (52, 18) within the crop:

示例：如果裁剪起点为 (50, 120)，而填写框在裁剪图内起点为 (52, 18)：

- entry_x0 = 52 + 50 = 102

- entry_x0 = 52 + 50 = 102

- entry_top = 18 + 120 = 138

- entry_top = 18 + 120 = 138

**Repeat for each field**, grouping nearby fields into single crops when possible.

**对每个字段重复此过程**，可能的话把相邻字段合并到同一次裁剪中。

### B.4: Create fields.json with Refined Coordinates / B.4：用精修后的坐标创建 fields.json

Create fields.json using `image_width` and `image_height` (signals image coordinates):

使用 `image_width` 和 `image_height` 创建 fields.json（表示使用图片坐标）：

```json
{
  "pages": [
    {"page_number": 1, "image_width": 1700, "image_height": 2200}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "Last name entry field",
      "field_label": "Last Name",
      "label_bounding_box": [120, 175, 242, 198],
      "entry_bounding_box": [255, 175, 720, 218],
      "entry_text": {"text": "Smith", "font_size": 10}
    }
  ]
}
```

**Important**: Use `image_width`/`image_height` and the refined pixel coordinates from the zoom analysis.

**重要**：使用 `image_width`/`image_height` 以及放大分析得到的精修像素坐标。

### B.5: Validate Bounding Boxes / B.5：校验边界框

Before filling, check your bounding boxes for errors:
`python scripts/check_bounding_boxes.py fields.json`

填写前，检查边界框是否有误：
`python scripts/check_bounding_boxes.py fields.json`

This checks for intersecting bounding boxes and entry boxes that are too small for the font size. Fix any reported errors before filling.

该检查会发现相交的边界框，以及相对字号过小的填写框。填写前先修复所有报告的错误。

---

## Hybrid Approach: Structure + Visual / 混合方案：结构 + 视觉

Use this when structure extraction works for most fields but misses some elements (e.g., circular checkboxes, unusual form controls).

当结构化提取对大多数字段有效、但遗漏了部分元素（例如圆形复选框、非标准表单控件）时使用此方案。

1. **Use Approach A** for fields that were detected in form_structure.json

1. 对 form_structure.json 中检测到的字段**使用方案 A**

2. **Convert PDF to images** for visual analysis of missing fields

2. **将 PDF 转换为图片**，对缺失字段做视觉分析

3. **Use zoom refinement** (from Approach B) for the missing fields

3. 对缺失字段**使用放大精修**（来自方案 B）

4. **Combine coordinates**: For fields from structure extraction, use `pdf_width`/`pdf_height`. For visually-estimated fields, you must convert image coordinates to PDF coordinates:

4. **合并坐标**：对来自结构化提取的字段，使用 `pdf_width`/`pdf_height`。对目测估计的字段，必须把图片坐标转换为 PDF 坐标：

   - pdf_x = image_x * (pdf_width / image_width)

   - pdf_x = image_x * (pdf_width / image_width)

   - pdf_y = image_y * (pdf_height / image_height)

   - pdf_y = image_y * (pdf_height / image_height)

5. **Use a single coordinate system** in fields.json - convert all to PDF coordinates with `pdf_width`/`pdf_height`

5. 在 fields.json 中**使用单一坐标系** —— 用 `pdf_width`/`pdf_height` 把所有坐标统一转换为 PDF 坐标

---

## Step 2: Validate Before Filling / 第 2 步：填写前校验

**Always validate bounding boxes before filling:**
`python scripts/check_bounding_boxes.py fields.json`

**填写前务必校验边界框：**
`python scripts/check_bounding_boxes.py fields.json`

This checks for:

该检查会发现：

- Intersecting bounding boxes (which would cause overlapping text)

- 相交的边界框（会导致文字重叠）

- Entry boxes that are too small for the specified font size

- 相对指定字号过小的填写框

Fix any reported errors in fields.json before proceeding.

继续之前，先修复 fields.json 中所有报告的错误。

## Step 3: Fill the Form / 第 3 步：填写表单

The fill script auto-detects the coordinate system and handles conversion:
`python scripts/fill_pdf_form_with_annotations.py <input.pdf> fields.json <output.pdf>`

填写脚本会自动检测坐标系并处理转换：
`python scripts/fill_pdf_form_with_annotations.py <input.pdf> fields.json <output.pdf>`

## Step 4: Verify Output / 第 4 步：验证输出

Convert the filled PDF to images and verify text placement:
`python scripts/convert_pdf_to_images.py <output.pdf> <verify_images/>`

把填写后的 PDF 转换为图片并验证文字位置：
`python scripts/convert_pdf_to_images.py <output.pdf> <verify_images/>`

If text is mispositioned:

如果文字位置不对：

- **Approach A**: Check that you're using PDF coordinates from form_structure.json with `pdf_width`/`pdf_height`

- **方案 A**：检查你是否在用 form_structure.json 的 PDF 坐标并配合 `pdf_width`/`pdf_height`

- **Approach B**: Check that image dimensions match and coordinates are accurate pixels

- **方案 B**：检查图片尺寸是否匹配、坐标是否为精确像素

- **Hybrid**: Ensure coordinate conversions are correct for visually-estimated fields

- **混合方案**：确保目测估计字段的坐标换算正确
