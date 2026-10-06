<!-- BILINGUAL-EN-ZH -->

# Compose the story visually / 用视觉讲故事

Choose the subject of each scene before choosing its container. Use an actual
artifact for a result, a spatial diagram for a relationship, and a bubble for
an exchange. Give the focal element most of the card's area. Keep the title
short and remove labels that repeat what the viewer can already see.

先确定每个场景的主体，再选择它的容器。结果用真实的产物（artifact）呈现，关系用空间示意图呈现，交流用气泡呈现。把卡片的大部分面积留给焦点元素。标题保持简短，删掉重复观众已能看见内容的标签。

Use the product palette and Optimistic fonts from
`/opt/hatch/skills/magic-moment/reference/design.md`. Keep a consistent corner
radius and padding rhythm across the video. Change composition when the subject
changes. Use a deliberate dark card for a technical mechanism or a hero metric
when it strengthens the sequence. Give supporting text less weight than the
focal element. Do not use identical title-plus-two-row cards for every topic.

使用 `/opt/hatch/skills/magic-moment/reference/design.md` 中的产品色板和 Optimistic 字体。在整个视频中保持一致的圆角半径和留白节奏。主体变化时构图随之变化。当深色卡片能强化叙事序列时，用它来呈现技术机制或核心指标。辅助文字的视觉分量要低于焦点元素。不要对每个话题都使用一模一样的"标题加两行"卡片。

Use these compositions when they fit the evidence:

当以下构图与证据相符时使用它们：

- Calendar: show a small week grid or time lane with the supported events in
  position. Highlight the available slot or planned activity once. Include
  dates and clock times only when sourced. Use an undated rhythm diagram when
  the narration supports a recurring pattern but no exact schedule.
  - 日历：展示一个小的周网格或时间轴，把有依据的事件放到位。可用时段或计划活动只高亮一次。日期和钟点只在有出处时才标注。当叙述支持"周期性规律"但没有确切日程时，使用不带日期的节奏示意图。
- Route: give the route line and endpoints room to read. Draw a real map only
  from sourced coordinates. Without coordinates, use a visibly abstract path
  beside the supported route name and distance or duration; omit map tiles,
  street names, compass marks, and a fake location pin.
  - 路线：给路线线和端点留出可读空间。只有基于有出处的坐标才绘制真实地图。没有坐标时，在可支持的路线名称和距离或时长旁边使用明显抽象的路径；省略地图瓦片、街道名、方位标记和伪造的定位图钉。
- Progress: use a chart only when its values are sourced. Distinguish completed
  activity from recommendations with solid versus outlined marks and short
  state labels. Do not turn an intended progression into a performance gain.
  - 进度：只有数值有出处时才使用图表。用实心与空心标记以及简短的状态标签区分"已完成的活动"与"建议"。不要把"打算做的事"画成"已经取得的成效"。
- Sync or automation: show the source, a connecting path, and the destination.
  Animate one transfer when the source supports a completed transfer. For a
  skill that was built but has no evidenced execution, reveal the connection
  without a success check or fabricated activity data.
  - 同步或自动化：展示来源、连接路径和目的地。当来源能证明一次已完成的传输时，动画演示这一次传输。对于已构建但没有可证实执行记录的技能，只呈现连接本身，不加成功对勾，也不编造活动数据。
- Artifact: reveal a large complete block of the actual artifact. Crop to the
  relevant content rather than fitting a whole desktop page into a thumbnail.
  Let its own visual identity carry the scene. Use a second crop when the
  narration moves from past records to upcoming work.
  - 产物：大幅展示真实产物的一个完整区块。裁剪到相关内容，而不是把整个桌面页面塞进缩略图。让产物自身的视觉识别特征承载场景。当叙述从过往记录转向即将进行的工作时，使用第二次裁剪。

Use the HTML examples in  
`/opt/hatch/skills/magic-moment/reference/design-system/story-compositions.html`  
for spatial layout and motion. Adapt their sample content from evidence. These
are explanatory diagrams, not replicas of product screens. Keep recognizable
product controls in the main kit's anatomy.

空间布局与动效参考 `/opt/hatch/skills/magic-moment/reference/design-system/story-compositions.html` 中的 HTML 示例。其示例内容要根据证据改写。这些是解释性图示，不是产品界面的复制品。可识别的产品控件保留在主套件的结构中。

Animate the narrated change once, then hold the result long enough to read.
Draw a path, highlight a slot, or reveal a recorded entry when that action is
supported. Keep decorative motion still while the viewer reads. Use the
renderer for entrance, exit, and tap motion; do not add another pop to the
whole HTML card.

把叙述到的变化动画演示一次，然后停住结果足够长的时间供人阅读。当动作有依据支持时，绘制路径、高亮时段或展示已记录的条目。观众阅读期间装饰性动效保持静止。入场、退场和点击动效交给渲染器处理；不要给整个 HTML 卡片再叠加一个弹出效果。

Plan the sequence as scene, source, focal element, motion, and hold. Preview
with `/opt/hatch/skills/magic-moment/mm preview <screenplay.json>`. Read the
initial, middle, and final card states at their returned phone size. Revise
weak hierarchy, clipped text, and tiny artifact content before the full render.
Review the final encoded video separately because standalone previews do not
prove face clearance, transition spacing, or audio timing.

按"场景、出处、焦点元素、动效、停顿"规划序列。用 `/opt/hatch/skills/magic-moment/mm preview <screenplay.json>` 预览。按返回的手机尺寸查看卡片的初始、中间和最终状态。在完整渲染之前修正层级薄弱、文字被裁切、产物内容过小等问题。最终编码后的视频要单独复查，因为独立预览并不能证明人脸避让、转场间距或音频时序是否合格。

【评论】"只画有出处的数据、无坐标时改用明显抽象的路径、不把计划画成成效"等条款，是针对生成式视频内容"以视觉暗示虚构事实"风险的诚实性护栏，禁止用地图瓦片或假定位图钉营造真实感。
