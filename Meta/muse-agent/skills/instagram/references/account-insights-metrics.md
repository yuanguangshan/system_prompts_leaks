<!-- BILINGUAL-EN-ZH -->
# Account insights metrics / 账户洞察指标

Use these names, units, and definitions when reporting `account-insights`
results or explaining an Instagram metric. Definitions come from Meta's
external metric catalog. Explain them in plain language, but keep the meaning
and the unit.

在报告 `account-insights` 结果或解释 Instagram 指标时，使用这些名称、单位和定义。定义来自 Meta 的对外指标目录。用通俗语言解释它们，但要保持含义与单位不变。

## Interpretation / 解读规则

- When explaining or comparing metrics, state each table definition's counting
  unit, counting condition, included content types, and estimate status.
  在解释或比较指标时，说明每条表格定义的计数单位、计数条件、所含内容类型以及估算状态。
- The catalog does not specify how Accounts Center accounts map to Instagram
  accounts, guarantee an ordering of these totals, or link the accounts and
  events across metrics. Treat these relationships as unknown.
  该目录没有说明 Accounts Center 账户如何映射到 Instagram 账户，不保证这些总数的排序，也不跨指标关联账户与事件。把这些关系视为未知。
- For interpretations beyond the reported counts and arithmetic, identify the
  catalog statement or independent evidence supporting the claim. If neither
  establishes it, state what remains unknown. When answering whether a gap
  proves a cause, stop there. Do not volunteer possible or likely causes.
  对于超出已报告计数和算术的解读，要指出支持该论断的目录表述或独立证据。若两者都不能确立它，就说明仍有何未知。回答差距是否能证明某个原因时，到此为止。不要主动提供可能或大概率的原因。
- Use the Viewers definition below. Do not add a video-only scope or a minimum
  watch duration.
  使用下方的 Viewers 定义。不要附加仅限视频的范围或最短观看时长。
- Describe a calculated ratio by its numerator and denominator. Ratios of
  visits or taps do not establish the fraction of distinct accounts that took
  an action.
  用分子和分母来描述计算出的比率。访问或点按的比率并不能确立采取该行动的去重账户占比。
- Visit counts can include repeat visits and do not identify distinct visitors.
  Distinct visiting accounts can equal or be fewer than the number of visits.
  访问计数可能包含重复访问，且不区分去重访客。去重的到访账户数可能等于或少于访问次数。
- `account-insights` has no follower count or follower growth. Do not derive
  one from demographic totals.
  `account-insights` 没有粉丝数或粉丝增长指标。不要从人口统计总数中推导它们。
- `account-insights` does not return an engagement rate. When calculating one,
  state the chosen formula and inputs.
  `account-insights` 不返回互动率。自行计算时，要说明所选的公式和输入。

## `account-insights` fields / `account-insights` 字段

| Field | Instagram name | Unit | Definition |
|---|---|---|---|
| `accounts_reached` (or `reach`) | Accounts reached | accounts | The number of unique accounts that have seen your content at least once, including promoted posts and stories. Different from Views, which may include multiple views by the same accounts. Estimated. |
| `viewers` | Viewers | Accounts Center accounts | The number of Accounts Center accounts that have viewed your content at least once. Content includes reels, posts, stories, videos, live videos, and ads. Estimated. |
| `content_views` | Views | views | The number of times your content was played or displayed, including repeat views. Content includes reels, posts, stories, videos, live videos, and ads. |
| `content_views_by_follow_type` | Views by followers and non-followers | views | Views split by `1` followers, `2` non-followers, `0` unknown. **Views from non-followers** is the percentage of views that came from non-followers. |
| `engaged_accounts` | Accounts engaged | accounts | The number of accounts that have interacted with your content, including in ads. Interactions include likes, saves, comments, shares, and replies. Estimated. |
| `total_interactions` | Content interactions | interactions | The total number of post, story, reel, video, and live video interactions, including interactions on boosted content. |
| `interactions_by_follow_type` | Content interactions by followers and non-followers | interactions | Interactions split by `1` followers, `2` non-followers. |
| `interactions_by_media_type` | Content interactions by content type | interactions | Interactions split by internal content-type codes. Do not name the content types. Report the total or omit it. |
| `likes` | Likes | likes | The number of likes on your content. |
| `comments` | Comments | comments | The number of comments on your content minus deleted comments. |
| `saves` | Saves | saves | The number of saves of your content minus unsaves. |
| `shares` | Shares | shares | The number of shares of your content. |
| `replies` | Replies | replies | Replies to your stories, including text replies and quick reaction replies. |
| `profile_visits` | Profile visits | visits | The number of times your profile was visited. |
| `bio_link_taps` | External link taps | taps | Taps on links on your profile, excluding taps on your connected Facebook profile. |
| `contact_button_taps` | Contact button taps | taps | Taps on your business address, call, email, or text buttons. |
| `follower_demographics_by_*` | Follower demographics | followers | Your followers by age, gender (`F`, `M`, `U` unknown), country, or city, based on information people provide in their profiles. |
| `online_followers` | Most active times | followers | Followers online by day and hour range. |

| 字段 | Instagram 名称 | 单位 | 定义 |
|---|---|---|---|
| `accounts_reached`（或 `reach`） | Accounts reached | accounts | 至少一次看过你的内容的去重账户数，包括推广帖子和快拍。与 Views 不同，后者可能包含同一账户的多次观看。为估算值。 |
| `viewers` | Viewers | Accounts Center 账户 | 至少一次观看过你内容的 Accounts Center 账户数。内容包括 reels、帖子、快拍、视频、直播视频和广告。为估算值。 |
| `content_views` | Views | views | 你的内容被播放或展示的次数，包括重复观看。内容包括 reels、帖子、快拍、视频、直播视频和广告。 |
| `content_views_by_follow_type` | Views by followers and non-followers | views | 观看按 `1` 粉丝、`2` 非粉丝、`0` 未知拆分。**Views from non-followers** 指来自非粉丝的观看所占的百分比。 |
| `engaged_accounts` | Accounts engaged | accounts | 与你的内容发生过互动的账户数，包括广告中的互动。互动包括点赞、收藏、评论、分享和回复。为估算值。 |
| `total_interactions` | Content interactions | interactions | 帖子、快拍、reel、视频和直播视频互动的总数，包括对加热（boosted）内容的互动。 |
| `interactions_by_follow_type` | Content interactions by followers and non-followers | interactions | 互动按 `1` 粉丝、`2` 非粉丝拆分。 |
| `interactions_by_media_type` | Content interactions by content type | interactions | 互动按内部内容类型代码拆分。不要说出内容类型的名称。报告总数或省略。 |
| `likes` | Likes | likes | 你的内容获得的点赞数。 |
| `comments` | Comments | comments | 你的内容获得的评论数减去已删除的评论。 |
| `saves` | Saves | saves | 你的内容被收藏的次数减去取消收藏的次数。 |
| `shares` | Shares | shares | 你的内容被分享的次数。 |
| `replies` | Replies | replies | 对你快拍的回复，包括文字回复和快速表情回复。 |
| `profile_visits` | Profile visits | visits | 你的主页被访问的次数。 |
| `bio_link_taps` | External link taps | taps | 对你主页上链接的点按，不包括对你关联的 Facebook 主页的点按。 |
| `contact_button_taps` | Contact button taps | taps | 对你的商家地址、致电、电子邮件或发短信按钮的点按。 |
| `follower_demographics_by_*` | Follower demographics | followers | 按年龄、性别（`F`、`M`、`U` 为未知）、国家或城市划分的粉丝分布，基于人们在个人资料中提供的信息。 |
| `online_followers` | Most active times | followers | 按日期和小时区间划分的在线粉丝。 |

## Other Instagram metrics / 其他 Instagram 指标

`account-insights` does not return these. Use the definitions to answer
questions about them.

`account-insights` 不返回这些指标。回答相关问题时使用这些定义。

| Metric | Definition |
|---|---|
| Impressions | The number of times your content was on screen, including in ads. Instagram now reports Views for organic content. |
| Initial plays | The number of times your reel starts to play for the first time in a reel session. Counts plays of at least 1 millisecond and excludes replays. |
| Replays | The number of times your reel starts to play again after an initial play in the same reel session. |
| Watch time | The total time your reel was played, including time spent replaying it. |
| Average watch time | Watch time divided by initial plays. A viewer-based version divides by viewers. |
| Story views | The number of views of your story, including repeat views. |
| Accounts reached (live video) | The number of unique accounts that saw at least 3 seconds of your live video. Estimated. |
| 3-second video plays | The number of times your video played for at least 3 seconds, or nearly its full length if shorter. |
| Returning viewers | Viewers who saw your content in the last 90 days and are returning to see this reel. |
| Overall followers | Accounts that followed you minus accounts that unfollowed you or left Instagram in the period. Net growth, not new followers. |
| Active followers | Followers who were active on Instagram in the period. |
| Hook rate | The percentage of views that watched past the first 3 seconds of recent reels. Calculated as views of at least 3 seconds divided by initial views. |
| Like rate, save rate | Each rate is the number of a reel's viewers who took the corresponding action divided by initial views, expressed as a percentage. |
| Comment rate, share rate | Each rate is the number of a reel's viewers who took the corresponding action divided by initial views, expressed as a percentage. Estimated and in development. |
| Profile activity | Actions taken when engaging with your profile. |

| 指标 | 定义 |
|---|---|
| Impressions（曝光） | 你的内容出现在屏幕上的次数，包括广告中。Instagram 现在对自然流量内容报告 Views。 |
| Initial plays（首次播放） | 你的 reel 在一次 reel 会话中首次开始播放的次数。计入至少 1 毫秒的播放，不包括重播。 |
| Replays（重播） | 在同一 reel 会话中，你的 reel 在首次播放之后再次开始播放的次数。 |
| Watch time（观看时长） | 你的 reel 被播放的总时长，包括重播花费的时间。 |
| Average watch time（平均观看时长） | 观看时长除以首次播放。基于观看者的版本则除以观看人数。 |
| Story views（快拍观看） | 你的快拍被观看的次数，包括重复观看。 |
| Accounts reached (live video)（触达账户，直播视频） | 至少观看你直播视频 3 秒的去重账户数。为估算值。 |
| 3-second video plays（3 秒视频播放） | 你的视频播放至少 3 秒的次数；若视频更短，则为接近完整时长的播放次数。 |
| Returning viewers（回访观看者） | 在过去 90 天内看过你的内容、此次回来看这条 reel 的观看者。 |
| Overall followers（粉丝总数） | 关注你的账户减去该时期内取关或离开 Instagram 的账户。是净增长，不是新增粉丝。 |
| Active followers（活跃粉丝） | 该时期内在 Instagram 上活跃过的粉丝。 |
| Hook rate（开场留存率） | 观看近期 reel 超过前 3 秒的观看所占百分比。计算方式为至少 3 秒的观看数除以初始观看数。 |
| Like rate, save rate（点赞率、收藏率） | 每个比率都是采取相应动作的 reel 观看者数量除以初始观看数，以百分比表示。 |
| Comment rate, share rate（评论率、分享率） | 每个比率都是采取相应动作的 reel 观看者数量除以初始观看数，以百分比表示。为估算值且仍在开发中。 |
| Profile activity（主页活动） | 与你的主页互动时采取的动作。 |

For a metric not listed here, use `social_content_performance`, which resolves
catalog definitions with `instagram-cli analytics-metric-metadata`. Do not
invent a definition.

对未列于此处的指标，使用 `social_content_performance`，它通过 `instagram-cli analytics-metric-metadata` 解析目录定义。不要自行发明定义。
