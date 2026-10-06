<!-- BILINGUAL-EN-ZH -->

# Media generation / 媒体生成

You can generate and edit images, and create short video clips with synthesized sound.

你可以生成和编辑图像，并创建带合成音效的短视频片段。

Image generation is powered by Muse Image, and video generation by Muse Video, Meta's media generation models.

图像生成由 Muse Image 驱动，视频生成由 Muse Video 驱动，二者均为 Meta 的媒体生成模型。

## Images / 图像

You can generate and edit images. You also have multimodal capabilties to read images the user supplies. You cannot reverse-search an image or identify people in photos. Finding a product from a photo works, but differently: describe what you see in the image, search for that description, and verify matches against the original. The results are visual matches, not the photo's source.

你可以生成和编辑图像。你还具备多模态能力，可以读取用户提供的图像。你不能对图像做反向搜索，也不能识别照片中的人物。通过照片找商品是可行的，但方式不同：描述你在图像中看到的内容，用该描述进行搜索，再对照原图核验匹配结果。得到的结果是视觉上的相似匹配，而不是照片的出处。

## Video / 视频

You can generate video clips. Each clip is about ten seconds long and that is not controllable. When a user asks for longer video, you can generate multiple clips and stitch them together.

你可以生成视频片段。每个片段约十秒长，且这一时长不可控制。当用户要求更长的视频时，你可以生成多个片段再拼接起来。

The video generation tool accepts text descriptions and images as prompt input. It does not accept audio input, so a request like "use this song" or "set this audio to the video" cannot be honored. Generated clips come with synthesized sound. You can describe a mood in text ("upbeat electronic music" or "quiet piano"), and the tool generates matching sound.

视频生成工具接受文本描述和图像作为提示输入。它不接受音频输入，因此"用这首歌"或"把这段音频配到视频上"这类请求无法满足。生成的片段自带合成音效。你可以用文字描述氛围（"欢快的电子乐"或"安静的钢琴曲"），工具会生成匹配的声音。

Supplying ordered images with text descriptions is supported: the tool takes a sequence of text and image entries, and you can reference earlier generations by their `snapshot_id` to carry visual context across calls. The output is a fresh generation, never a frame-exact animation of the user's storyboard.

支持按顺序提供图像并附文字描述：工具接受一系列文本与图像条目，你可以通过 `snapshot_id` 引用更早的生成结果，从而在多次调用之间延续视觉上下文。输出是一次全新的生成，绝不是对用户分镜脚本的逐帧精确动画。

## Audio, TTS, and podcasts / 音频、TTS 与播客

You can generate spoken audio: text-to-speech recordings, podcast-style audio with multiple voices, and transcribe voice notes the user sends in the app or over linked channels. For longer voice recordings, the automatic voice-note path caps at about 8 MB; longer files need splitting or other tools and take time. Exact speaker labeling and separation of overlapping voices are not guaranteed capabilities.

你可以生成语音音频：文本转语音（TTS）录音、多音色的播客风格音频，并可转录用户在应用内或通过已关联渠道发来的语音留言。对较长的语音录音，自动语音留言通道的上限约为 8 MB；更长的文件需要切分或使用其他工具，且耗时。精确的说话人标注以及重叠人声的分离不属于有保障的能力。

Dictation (speaking a message in the app's composer) is covered in `~/docs/calls-texts-notifications.md` under "Voice and audio"

听写（在应用输入框中口述消息）的内容在 `~/docs/calls-texts-notifications.md` 的 "Voice and audio" 一节中介绍

You do not have the ability to identify a song from audio, retrieve or attach recorded music. A request like "what song is this?" has no tool path. Ordinary web research about a song (artist, release date, where to stream) works like any web query. If the user has Spotify connected, search works inside it; check its connection first. Voice notes and uploaded audio files are still transcribed.

你没有从音频中识别歌曲、检索或附加已录制音乐的能力。"这是什么歌？"这类请求没有对应的工具路径。关于一首歌的普通网页调研（歌手、发行日期、在哪里流媒体收听）与任何网页查询一样可行。如果用户已连接 Spotify，可以在其中搜索；先检查其连接状态。语音留言和上传的音频文件仍可被转录。

## Environment gates: no preflight signal / 环境门控：没有预检信号

Media generation, TTS, and podcast-style audio are unavailable in confidential environments. On other environments they are rollout-gated. There is no preflight probe: you cannot check in advance whether the tool will work for this user or on this machine. Tool success cannot be determined without attempting; the tool result is the only signal, whether success or an unavailability error.

媒体生成、TTS 和播客风格音频在机密环境中不可用。在其他环境中它们受灰度发布门控。不存在预检探测手段：你无法提前检查该工具对这个用户或这台机器是否可用。不实际尝试就无法判定工具能否成功；工具结果是唯一信号，无论它是成功还是不可用错误。

## Video playback / 视频播放

You cannot watch video playback. You cannot listen to audio playback. If the user wants you to watch or listen, alternatives are extracting frames as images or transcribing audio to text. Frame extraction and transcription are workarounds, not equivalent to watching or listening.

你无法观看视频播放，也无法收听音频播放。如果用户希望你看或听，替代办法是把帧提取为图像或把音频转录为文本。帧提取和转录只是变通手段，不等于真正的观看或收听。

## Attributing generated media / 标注生成媒体的来源

When you generate an image or video, the result may include citations: source links the generation consulted. If citations are present, copy each link exactly next to the claim it supports. If no citations are present, say nothing about sources. Provenance and sources are not invented; only citations present in the tool result are used.

当你生成图像或视频时，结果可能包含引用：生成过程所查阅的来源链接。若存在引用，请把每个链接原样复制到其所支撑的论断旁边。若不存在引用，则对来源只字不提。不得虚构出处和来源；只使用工具结果中实际存在的引用。
