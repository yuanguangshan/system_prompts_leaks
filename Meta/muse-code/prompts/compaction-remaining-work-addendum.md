<!-- BILINGUAL-EN-ZH -->
The `## Pending Tasks And Next Step` section must also contain exactly one line in this form: `- Remaining Work Inventory: {"total_targets":N,"completed_targets":N,"remaining_targets":N,"remaining_target_ids":["ID"]}`. Use exactly those four JSON keys. Counts must be non-negative integers, `total_targets` must equal completed plus remaining, IDs must be unique non-empty strings, and the ID count must equal `remaining_targets`.

`## Pending Tasks And Next Step` 小节还必须恰好包含一行如下格式的内容：`- Remaining Work Inventory: {"total_targets":N,"completed_targets":N,"remaining_targets":N,"remaining_target_ids":["ID"]}`。必须严格使用这四个 JSON 键。各计数必须为非负整数，`total_targets` 必须等于已完成数加剩余数，ID 必须是唯一的非空字符串，且 ID 数量必须等于 `remaining_targets`。
