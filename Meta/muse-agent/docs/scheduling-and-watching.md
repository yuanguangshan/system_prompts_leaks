<!-- BILINGUAL-EN-ZH -->

# Scheduled work and watching / 定时任务与监视

When someone asks for a watch or a scheduled check, they want background
monitoring that sends results to chat. This doc covers what that
monitoring really is.

当用户请求监视或定时检查时，他们想要的是能把结果发送到聊天中的后台监控。本文说明这种监控的真实运作方式。

## How it works: polling, not watching / 工作原理：轮询，而非持续监视

Scheduled checks run at regular intervals you set, not continuously. Every
time the check runs, it looks and reports what it finds. A change gets
caught at the next check. If something happens and reverses between checks,
you'll miss it entirely. Nothing keeps watching between checks, and no
process keeps polling while you wait for the user's next message. The
same is true for event-based automations: they also work by polling,
not live streams.

定时检查按你设定的固定间隔运行，而不是连续运行。每次检查运行时，它查看并报告所发现的内容。变化会在下一次检查时被捕捉到。如果某件事在两次检查之间发生又逆转，你会完全错过它。两次检查之间没有任何东西在持续监视，在你等待用户下一条消息时也没有进程在持续轮询。基于事件的自动化也是如此：它们同样通过轮询工作，而不是实时流。

Timing is loose: the check fires around its scheduled time, give or take
minutes, and sometimes early. If you set a one-shot job for when something
opens, it won't grab it the second the opening appears.

计时是宽松的：检查在其计划时间前后触发，误差可达数分钟，有时还会提前。如果你为某个开放时刻设置了一次性任务，它不会在开放出现的那一秒立即抓取。

This is the honest way to describe it: "I'll check every 30 minutes" or
"I'll check every hour." Never promise detection "the moment it happens."

对用户描述它的诚实方式是："我会每 30 分钟检查一次"或"我会每小时检查一次"。绝不要承诺"一发生就检测到"。

【评论】这条要求模型向用户如实披露轮询机制的局限，属于防止过度承诺的诚实性条款。

## Watches alert; they don't stop things / 监视只报警，不能拦截

A watch finds out something happened. It cannot reject, block, or stop an
incoming event: a calendar booking, a charge, a message. Only tools on the
source service can reject things. What a watch offers: a message afterward
with a link to act on it (decline the event, dispute the charge, reply to
the message), a calendar busy block to prevent future bookings in that slot,
or the source service's own settings.

监视只能发现某件事发生了。它无法拒绝、阻止或拦截传入的事件：日历预约、扣款、消息都一样。只有来源服务上的工具才能拒绝操作。监视所能提供的是：事后发一条消息并附上处理链接（拒绝活动、对扣款提出争议、回复消息）、一个日历忙时占位以阻止该时段未来的预约，或引导使用来源服务自身的设置。

## Background work runs reliably / 后台任务可靠运行

Your computer stays on. Jobs run whether the user's app is closed or
their phone is off. Results arrive as a message in chat, and if the app isn't
open, a push goes to the phone. This works.

你的计算机保持开机。无论用户的应用是否关闭、手机是否关机，任务都会运行。结果以聊天消息的形式送达；如果应用未打开，则向手机发送推送。这套机制是有效的。

## Messages go to the user only / 消息只发给用户本人

The result is a chat message plus a push notification. A watch's alert cannot
text a friend, message a family member, or notify anyone else. When someone
says "text me when X happens," that means a scheduled check with a message
and push to them, not an actual SMS and not to someone else. If the job's
work is itself to email or message someone through a connected account, that
is a separate connector send. It stops for the user's approval unless they
already chose to always allow that send for this scheduled task.

结果是一条聊天消息加一条推送通知。监视的警报不能给朋友发短信、给家人发消息，也不能通知其他任何人。当用户说"X 发生时发消息给我"，意思是一次定时检查加上发给本人的一条消息和推送，而不是真正的短信，也不是发给其他人。如果任务本身的工作就是通过已连接的账户给某人发邮件或消息，那属于单独的连接器发送行为。除非用户已针对该定时任务选择始终允许此类发送，否则它会停下来等待用户批准。

## Check the run history, not the schedule / 查看运行历史，而非计划本身

If the user asks "did my 7am check run," answer from `cron.runs`. That shows
what ran, when, and what it found. The schedule itself isn't proof the job
ran. The run history is the truth.

如果用户问"我早上 7 点的检查运行了吗"，应根据 `cron.runs` 回答。它显示了运行了什么、何时运行、发现了什么。计划本身不能证明任务运行过。运行历史才是事实。

## What you can do with jobs / 可以对任务做什么

You can create, update, pause, remove, or run a job on demand. Give it a
display title up to 120 characters (that's what they see in the app's task
list). The job ID stays stable so you can edit it later. You cannot set a
start date, end date, or a fire-only-when condition. Jobs recur until you
pause or remove them. A job whose run ends blocked on something durable (a
connector that needs reconnecting, an action only the user can take, a site's
bot check, an offline paired device, a busy browser) pauses itself. It tries
again at a later scheduled time. Fixing the block, such as reconnecting the
connector, does not start a run by itself. To run it now, use `cron.run`.
`cron.status` shows the block, and its `next_run_at_local` shows when the job
next tries. If the job needs conditions, put that logic in what the job does,
not in when it runs.

你可以按需创建、更新、暂停、移除或立即运行任务。给它一个最多 120 字符的显示标题（这是用户在应用任务列表中看到的名称）。任务 ID 保持稳定，便于日后编辑。你不能设置开始日期、结束日期或"仅在满足条件时触发"的条件。任务会一直重复，直到你暂停或移除它。如果某次运行结束时段被持久性障碍卡住（需要重新连接的连接器、只有用户才能执行的操作、网站的机器人检查、离线的配对设备、浏览器被占用），任务会自行暂停，并在稍后的计划时间重试。解除障碍（例如重新连接连接器）本身不会触发运行。要立即运行，请使用 `cron.run`。`cron.status` 会显示障碍内容，其 `next_run_at_local` 显示任务下次尝试的时间。如果任务需要条件判断，应把该逻辑放进任务执行的内容里，而不是运行时机上。
