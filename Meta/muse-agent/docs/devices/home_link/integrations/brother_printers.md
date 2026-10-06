<!-- BILINGUAL-EN-ZH -->
# Brother Printers: IPP Printing / Brother 打印机：IPP 打印

**Last verified:** 2026-09-14 on a Brother MFC-J880DW

**最后验证：** 2026-09-14，Brother MFC-J880DW

Use this guide when fresh discovery identifies a Brother printer with IPP and
the user asks to print.

当新发现的设备识别出支持 IPP 的 Brother 打印机、且用户请求打印时，请使用本指南。

## Recommended Path / 推荐路径

1. Query current IPP capabilities. When advertised, prefer
   `image/pwg-raster` before PDF or JPEG.
   查询当前的 IPP 能力。若打印机已声明支持，优先使用 `image/pwg-raster`，其次才是 PDF 或 JPEG。
2. Generate PWG Raster with standard CUPS tools such as `imagetoraster` and  
   `rastertopwg`.
   使用标准 CUPS 工具（如 `imagetoraster` 与 `rastertopwg`）生成 PWG Raster。
3. Match media, resolution, printable area, and other print options to the
   printer's advertised values. Scale the layout and text for that resolution.
   将介质、分辨率、可打印区域及其他打印选项与打印机声明的值相匹配，并按该分辨率缩放版面与文字。
4. Submit it through Home Link as an IPP `Print-Job` with document format
   `image/pwg-raster`. Check the job status and output before retrying.
   通过 Home Link 以文档格式 `image/pwg-raster` 提交 IPP `Print-Job`。重试前先检查任务状态与输出。
