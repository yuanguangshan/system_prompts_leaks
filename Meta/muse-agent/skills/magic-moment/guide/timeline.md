<!-- BILINGUAL-EN-ZH -->

# Source and timing / 素材来源与时间轴

Run `/opt/hatch/skills/magic-moment/mm transcribe <video> --run <unique-name>`. Keep its source digest, duration, original ASR text, and word timestamps in `~/workspace/.output/<unique-name>/mm_transcript.json`. Use a new run for different footage. Do not edit the ASR file. Supply user corrections through the screenplay's `transcript_correction` object described in `/opt/hatch/skills/magic-moment/guide/story_and_canon.md`.

运行 `/opt/hatch/skills/magic-moment/mm transcribe <video> --run <unique-name>`。把素材摘要（digest）、时长、原始 ASR 文本和词级时间戳保存在 `~/workspace/.output/<unique-name>/mm_transcript.json` 中。不同的素材要使用新的运行（run）。不要编辑 ASR 文件。用户的修正通过剧本的 `transcript_correction` 对象提供，参见 `/opt/hatch/skills/magic-moment/guide/story_and_canon.md`。

Anchor each beat to the narrated claim it depicts. Use word timestamps when the wording is unchanged. Inspect the source to time corrected text; do not reuse the old ASR words as if they aligned with the correction. Do not show a result before the narration reaches it.

把每个节拍（beat）锚定到它所描绘的旁白论断上。措辞未变时使用词级时间戳。修正后的文本要检查素材来重新定时；不要把旧的 ASR 词当作与修正文本对齐来复用。旁白尚未讲到某个结果之前，不要把它显示出来。

Use finite nonnegative seconds for `start` and `end`. End each content beat within the measured source duration. The renderer preserves the source and appends the Muse close after the footage ends. The close lasts through the full logo animation and one avatar reaction, then holds the final frame briefly. Do not cut off the speaker's conclusion to make room for the close. Do not promise a thirty-second result from longer footage without a separate user-requested edit.

`start` 和 `end` 使用有限的非负秒数。每个内容节拍都必须在实测素材时长之内结束。渲染器保留原素材，并在素材播完后追加 Muse 片尾。片尾会完整播放 logo 动画和一次虚拟形象反应，然后短暂定格最后一帧。不要为了给片尾腾出空间而切断讲话者的结论。素材长于三十秒时，除非用户另行要求剪辑，否则不要承诺产出三十秒的成片。

Treat `end` as the end of the beat's active hold and animation. Content remains in the thread after `end`; later entries push it backward along the arc, shrinking it and increasing its transparency. Typing indicators disappear at `end`. Coverage measures active intervals; it is informational and is not a reason to add filler. Inspect actual visible holds in the rendered video.

把 `end` 视为节拍活动保持与动画的结束点。`end` 之后内容仍保留在线程中；后续条目会把它沿弧线向后推移，使其缩小并提高透明度。输入指示符（typing indicators）在 `end` 时消失。覆盖率（coverage）衡量的是活动区间；它仅作参考信息，不能成为添加填充内容的理由。要检查渲染后视频中实际可见的保持区间。

Allow a visual at least 0.8 seconds after the preceding beat. Answer a typing indicator with its supported Muse bubble. Keep at most three active beats at once. Inspect every tap transition: no new beat may enter during a tap. Validation resolves eligible image, video, and browser taps before applying this rule; use `tap: false` to opt out.

前一个节拍之后，视觉元素至少要间隔 0.8 秒才允许出现。输入指示符要用其支持的 Muse 气泡来回应。同时最多保持三个活动节拍。检查每一次点按（tap）过渡：点按期间不得有新节拍进入。校验流程会先处理符合条件的图片、视频和浏览器点按，再应用此规则；可用 `tap: false` 选择退出。

【评论】"素材更长时不得承诺三十秒成片"是防过度承诺条款；"结果不得早于旁白出现"则体现了音画同步的时间轴约束。
