# VibeOS 原版参考资料

## 什么是 Vibe OS（微软演示版）？

这个 Vibe OS 并不是一个传统的在底层硬件上运行的操作系统（比如 Windows 或 Linux），而是一个**完全由 AI 实时生成所有应用程序 UI** 的实验性概念系统。它的运行机制非常特别：

* **没有预先编写的代码：** 里面的计算器、浏览器、画图软件甚至嵌套的模拟器，都没有任何事先写好的应用逻辑代码。
* **基于大模型和 HTML 的动态渲染：** 它的每一个应用窗口都是一个 iframe，背后连接着一个独立的 Copilot AI 会话（基于 Copilot SDK 构建）。
* **交互机制：** UI 完全以纯 HTML 呈现。当用户点击某个按钮时，系统只会把用户的交互（比如点击了哪个元素 ID）发送给 AI 模型。AI 经过思考后，直接返回一段 HTML 差异代码（diffs）来更新界面。
* **“幻觉”变成现实：** 因为这一切都是 AI 当场生成的，所以它给出的计算结果、画图操作或者浏览器内容，本质上都是 AI “幻觉”出来的合理反馈，但足以拼凑出一个看起来完全可用的互动虚假操作系统环境。

## 作者是谁？

这个项目的作者是微软的知名开发者 Steve Sanderson。

他在与微软高管 Scott Hanselman 和 Mark Russinovich 的一场开发者会议直播（*Scott and Mark learn to Vibe Check with Steve Sanderson*）中展示了这个项目。据 Steve 本人透露，这个极具启发性的 Vibe OS 演示大约只花了他 5 到 6 个小时就构建完成了。


## 字幕翻译

章节一：嘉宾登场与“氛围检测” (Guest Intro & Vibe Check)
Scott / Mark (主持人):
Hey friends, we are back. Thank you so much again to our sponsors for making this happen. This is Scott and Mark. Learn to vibe check. Our first guest today is the wonderful Steve Sanderson. Steve Sanderson and his extremely well-known programmer. You've probably heard of things like KnockoutJS. He's worked on Blazer. He's done amazing things on the internet. And he has brought us something today that we have not seen. No one here has seen this before. We don't know if it's completely bespoke. We don't know if he wrote it in a caffeinated co-pilot session. It could be SLOP. It could be vibes. It could be AI augmented software engineering. What do you have for us, sir?
嘿，朋友们，我们回来了。再次感谢我们的赞助商促成这次活动。我是 Scott，这位是 Mark。一起来学着做个“氛围检测”吧。今天我们的第一位嘉宾是出色的 Steve Sanderson。Steve Sanderson 是一位非常著名的程序员。你可能听说过 KnockoutJS 这样的东西，他也参与了 Blazor 的开发。他在互联网上做出过惊人的成就。今天，他给我们带来了一些我们前所未见的东西。在场的任何人都没见过。我们不知道这是否是完全定制的，也不知道他是不是在狂喝咖啡的 Copilot 编程阶段写出来的。它可能是粗制滥造的AI垃圾（SLOP），可能纯粹是看感觉（vibes），也可能是 AI 增强的软件工程。先生，你今天给我们带来了什么？
章节二：见证历史：Vibe OS 诞生 (Witnessing History: Introducing Vibe OS)
Steve Sanderson:
All right. Yes. Um, so what I'm going to show you today could be pretty momentous. So I need to start by putting it in slightly historical context. All right. So the human journey, right? We've come a long way. Stone tools, language and culture, mathematics, computing, and these things are all quite good, but I don't think we've really reached the pinnacle of what we could be as a species yet, or at least until what I'm about to show you just now.
好的。是的。嗯，我今天要展示给你们的东西可能具有重大历史意义。所以我需要先把它放在一个简短的历史背景中来看。好吧，人类的旅程，对吧？我们已经走过了漫长的道路。石器、语言和文化、数学、计算，这些东西都相当不错，但我认为作为一门物种，我们还没有真正达到我们能达到的巅峰，至少在我即将向你们展示这个东西之前还没有。
Steve Sanderson:
Okay, so I know you're probably all thinking like, oh, he's going to reinvent the whole software industry, and we're not ready for all this. I know you've all had a lot of change to deal with recently. You know, we've moved from this world where humans write code into a world where AI writes code. And I know that that probably feels like quite a big deal to Mark and Scott, but for a true visionary like myself, I'm already thinking about where we're going next. And in fact, I'm already there.
好的，我知道你们可能都在想：哦，他要彻底重塑整个软件行业了，我们还没准备好迎接这一切。我知道你们最近都已经应付了太多变化。要知道，我们已经从人类写代码的世界步入了 AI 写代码的世界。我知道这对于 Mark 和 Scott 来说可能感觉是件惊天动地的大事，但对于像我这样一个真正的远见卓识者来说，我已经开始思考我们的下一步要走向哪里了。事实上，我已经到达那一步了。
Steve Sanderson:
So, I'm going to show you and I need to introduce you to a completely new operating system that's going to change everything about how we work. So, introducing Vibe OS. Okay, it's the world's first completely hallucinated operating system. All right, and I'm going to show it to you right now.
所以，我要向你们展示并介绍一个全新的操作系统，它将改变我们工作方式的一切。隆重介绍：Vibe OS。好，这是世界上第一个“完全由 AI 幻觉生成的”操作系统。好的，我现在就演示给你们看。
Steve Sanderson:
So, firstly, it is a real operating system. I can boot it. It's a VHDX file here, about 1.3 GB. Could probably make it smaller. I didn't bother. Um, I could boot it on bare metal, but I'm not going to. I'm going to boot it in HyperV. Uh, and so you will see that boot up nice and fast. It's very efficient. And when that comes up, you'll see it's got a beautiful user interface that allows our user to be highly productive.
首先，它是一个真实的操作系统。我可以启动它。这里有一个大概 1.3 GB 的 VHDX 虚拟磁盘文件。可能可以把它做得更小，但我没去折腾。嗯，我可以在裸机上启动它，但我不想那么做。我将在 Hyper-V 虚拟机里启动它。大家会看到它启动得非常快。它非常高效。当它启动后，你会看到它有一个漂亮的用户界面，让用户能够保持高效。
章节三：无代码的实时幻觉应用 (Codeless Real-time Hallucinated Apps)
Steve Sanderson:
Okay, so here we go. We're ready to get started with that. It's a bit hard to see there, so let's just switch to a full screen view. And um we're going to start by running a few of the applications that are built in here. So we'll get some of the classics going. So we'll have notepad, we'll have calculator, we'll have internet explorer.
好了，我们开始了。准备好见证奇迹。那里有点难看清，我们切到全屏视图吧。嗯，我们将从运行这里面内置的一些应用程序开始。我们来跑几个经典软件：记事本（Notepad）、计算器（Calculator），还有 IE 浏览器（Internet Explorer）。
Steve Sanderson:
And the thing that's unique here, the thing that you will never have seen before is that in these applications, there is no code at all. Everything in all these applications is being hallucinated in real time by the AI. So there's no buttons, there's no event handlers, there's no logic to say what to do, but the application works. So, if I do five divided by three and then equals, if I can find that, it's up there this time. Um, it's 1.6. Okay, so it works, right?
这里独一无二的地方在于，这也是你们绝对前所未见的：在这些应用程序中，根本没有任何代码！这些应用程序里的所有东西，都是 AI 实时“幻觉”生成的。所以这里没有真正的按钮，没有事件处理程序，没有任何逻辑代码来告诉它该怎么做，但这个应用程序就是能用！所以，如果我按 5 除以 3，然后按等于号——如果我能找到等于号的话，这次它跑到上面去了——嗯，结果是 1.6（注：无限循环小数）。看，它是管用的，对吧？
Steve Sanderson:
(Demonstration begins – showing UI functioning without any written code)
But I didn't write any code and neither did AI. There's no AI written code either. It's just producing UI exclusively with nothing behind it.
（演示开始 - 展示没有任何代码支持的 UI 如何工作）
但我没写任何代码，AI 也没写。背后根本没有 AI 生成的代码逻辑。它纯粹只是在生成 UI（界面），背后空空如也。
Steve Sanderson:
(Introduction of Internet Explorer within Vibe OS)
Uh, and we've got Internet Explorer here. Um, unlike the boring operating systems that you use today, this solves one of the big problems that people have, right? People are always wondering when they're on the internet, is it AI or not? But in Vibe OS, you don't have that problem. There's no question to be answered because the answer is always yes. Everything is AI.
（在 Vibe OS 中介绍 IE 浏览器）
我们这儿还有 IE 浏览器。嗯，不像你们今天用的那些无聊的操作系统，这解决了一个困扰人们的大问题，对吧？人们上网时总是在想：“这内容是 AI 写的还是真人写的？”但在 Vibe OS 里，你没有这个烦恼。你不需要问这个问题，因为答案永远是“是的”。一切全是 AI 生成的。
Steve Sanderson:
So, if I want to, I can go to, I don't know, let's see if we can find uh whether Scott Hanselman has a Wikipedia page, shall we? So, I'm going to go to uh google.com here and let's do a search for Hanselman Wikipedia and we'll do a Google search there. And I don't need to remind you this, but all of the content you're seeing is completely hallucinated, right? So, it's just coming directly out of the model, and the UI is being updated in real time as we go. And here's a beautiful picture of Scott Hansel. [laughter] I'm not sure about that one. All right.
所以，如果我想的话，我可以打开……不知道，我们来看看能不能找到 Scott Hanselman 有没有维基百科页面，好吗？所以我要在这里打开 google.com，然后搜索 Hanselman Wikipedia。我们来 Google 一下。我不需要提醒你们，但你们看到的所有内容都是完全通过幻觉生成的，对吧？它直接来自模型输出，同时 UI 界面在实时更新。看，这里有一张 Scott Hanselman 的“漂亮”照片。[全场大笑] 我对这张照片持保留意见。好吧。
章节四：观众点播：从 Money 95 到嵌套系统 (On-Demand: Money 95 & Nested Systems)
Steve Sanderson:
But it goes further, right? We're not limited to these built-in applications.
(Demonstration: AI can generate any requested app on demand)
We can search for any app that we want. And everyone always does like to do app demos. So, anything that I search for, I'm going to find and it will just be created for me on the fly. All right. So, let's see what else we could find in here. Um, let's go for one of the classics. Uh, Encarta 98. Everyone loves that one. But it's all about Mark Russinovich. All right. And so we can get a fully customized version of Encarta 98 on whatever subject we're interested in. All right. So lots of facts and figures about um Mark here. Let's see. Selected facts: Russinovich has a talent for turning internals into folklore. Great. I assume that's true. I don't know. All right.
但它的能力远不止于此。我们并不局限于这些内置应用。
（演示：AI 可以随时按需生成任何应用）
我们可以搜索任何想要的应用程序。大家总是喜欢做应用演示。所以，无论我搜索什么，我都能找到，而且它会立刻为我凭空生成。好的。我们看看还能在里面找到什么。嗯，我们来找个经典的。Encarta 98 微软百科全书。人人都爱这个。不过我们来生成一个专门关于 Mark Russinovich 的版本。好的，这样我们就能得到一个完全定制版的 Encarta 98，内容可以是任何我们感兴趣的主题。好吧，这里有很多关于 Mark 的事实和数据。我们来看看。精选事实：Russinovich 具有将内部技术机制（internals）转化为民间传说的天赋。太棒了。我猜这是真的，我也不清楚。好吧。
Steve Sanderson:
And other applications that you would want, but normally it's too difficult to make. So like Commander XCE, but it's always rude to you. All right. So that's the sort of application that I think is a genius idea, but it's difficult to get someone to implement something like that. Uh so let's see what we've got on this machine here. And obviously this is always different every time. Okay, there's something suspicious. Let's see if we can run bash in here. What's it got to say to us? Uh bash, you have somehow offended. Okay, fine. All right, so you get the idea, right? This is a pretty revolutionary way to do computing because we don't need to write any code for any of our applications anymore. Are there any software you would like to try out within ViOS? Anything you can think of?
还有其他你想要但通常很难开发出来的应用。比如“命令行提示符”，但它总是对你恶语相向。这就是我认为天才般的想法，但你很难找人去真正实现这种软件。所以我们来看看这台机器上有什么。显然这每次都会不一样。好，这里有个可疑的东西。我们看看能不能在这里跑 Bash 终端。它会跟我们说什么？呃，Bash 说你不知怎么地冒犯了它。好吧，行吧。好的，你们明白这个概念了吧？这是一种非常革命性的计算方式，因为我们再也不需要为任何应用程序编写一行代码了。在这个 ViOS 里，你们有什么想试用的软件吗？能想到什么？
Scott / Mark:
Uh Microsoft Money95.
呃，微软 Money 95（财务软件）。
Steve Sanderson:
Money95.
Money 95 安排上。
Scott / Mark:
Civilization peaked at Microsoft Money95.
人类文明的巅峰就是微软 Money 95 发布的时刻。
Scott / Mark:
Uh but but Scott Hanselman has a lot of money. Oh, but for Scott Hanselman. Yeah, you can say scoot as well. That's totally fine. All right. So, let's see what uh money Microsoft what Scott had in 95. [laughter] Okay. Let's see. Okay. And this is this is Scott's checking account.
呃，不过 Scott Hanselman 可有钱了。哦，那就生成一个专属 Scott Hanselman 版本的。是啊，你拼成 Scoot 也没问题。好的。我们来看看 95 年 Scott 的微软 Money 里有什么。[大笑] 好的，我们来看看。好，这是这是 Scott 的支票账户。
Audience/Host:
What model is in the back end?
后台跑的是什么模型？
Steve Sanderson:
Dear Scott, um it's been a tough month for you, I guess.
“亲爱的 Scott，嗯，我猜你这个月过得很艰难。”
Scott / Mark:
Where where are the tacos? There we go. That is amazing. All right. So, all right. I want to see paint with a drawing of Scott on it. Paint with a drawing of stars.
墨西哥卷饼花在哪儿了？就在这。太惊人了。好吧，我想看看画图（Paint）软件，里面要画着 Scott。或者画着星星。
Steve Sanderson:
I'm an owl though. Uh let's do paint before your haircut but with pre-installed. Okay. And so and you've got this is co-pilot uh SDK that's doing this.
我是夜猫子。呃，我们来个你理发前样子的画图软件，但要是预装好的。好的。所以这个……这背后是 Copilot SDK 在发挥作用。
Scott / Mark:
This is using Copilot SDK, believe it or not, behind the scenes. I can show you a little bit about how it works if you want. Um so it says it's got a very normal picture of Scott Hanselman for some reason. I don't know what it means by very normal. Um, okay. There's a normal picture of Scott Hansselman.
信不信由你，它后台使用的是 Copilot SDK。如果你们想看，我可以稍微展示一下它是怎么工作的。嗯，由于某种原因，这里说它有一张非常“正常”的 Scott Hanselman 的照片。我不知道“非常正常”是什么意思。嗯，好吧。这是一张“正常”的 Scott Hanselman 照片。
Scott / Mark:
That's your cage. That my cage. That's the cage you keep me in.
那是你的笼子。那是我的笼子。那是你把我关起来的笼子。
Scott / Mark:
Okay. This is insane. Um, wow. Okay. Could it This is This is Vibe OS. Can it have nested OS? How many How many OS deep can we go? Can you simulate like a an Altair 8080?
好吧。这太疯狂了。哇。好的。这能……这是 Vibe OS，它能有嵌套的操作系统吗？我们能嵌套多少层操作系统？你能模拟一个像 Altair 8080 那样的东西吗？
Steve Sanderson:
That's like a terminal emulator. So I need a simulator simulate a whole different system shushing into an Altair. Totally. So we can get iOS simulator in there. We can What else could we have? We could have um
那就像一个终端模拟器了。所以我需要一个模拟器来模拟一个完全不同的系统，SSH 连接进一台 Altair 里。完全可以。我们可以在里面塞个 iOS 模拟器。我们还能弄点啥？我们还能……
Scott / Mark:
This may be the greatest thing I've ever seen in my life. Me too.
这可能是我这辈子见过的最棒的东西了。我也是。
Steve Sanderson:
Oh yeah. Well, fair enough. All right. So, uh what have we got? Oh, it's difficult to see on this screen size, but yes, we got a fully functional version.
哦是的。嗯，说得有理。好了。所以，我们得到了什么？哦，在这个屏幕尺寸上很难看清，但是的，我们得到了一个功能齐全的版本。
Scott / Mark:
Right. And I love that it uses the the appropriate Windows CE buttons that you are likely to see in an iPhone. Yeah, it's very very legitimate. So, okay. So, did he write this from scratch or did he vibing vibes? My mind is blown by this.
(Viewing fully functional OS simulation with Windows CE interface)
没错。而且我喜欢它使用了合适的 Windows CE 按钮，那种你很有可能在 iPhone 里看到的按钮（注：这是个反差梗）。是的，这看着“非常非常合法/正宗”。所以，好吧。他是从零开始写出这个的吗，还是凭感觉（vibes）弄出来的？我的大脑被这玩意儿彻底震撼了。
（观看具有 Windows CE 界面的全功能 OS 模拟）
Steve Sanderson:
I know. I told you it'd be fun.
我懂。我告诉过你们这会很好玩的。
章节五：核心揭秘：这一切是怎么做到的？ (Behind the Scenes: How it Works)
Scott / Mark:
You have questions. Please, you have three minutes. Define hallucination.
你们肯定有问题想问。请吧，你还有三分钟。请定义一下“幻觉”。
Steve Sanderson:
I'm sorry. I hope define hallucination. You call this a hallucinated operating system, right? It's hallucinating everything. Are there sound people able to make the sound come this way? Louder. Can you hear this? Not really. Um he says you're define hallucination for you.
抱歉。我希望……定义幻觉？你把这个叫做“幻觉操作系统”，对吧？它虚构了一切。音响师能把声音传到这边吗？大点声。你们能听到吗？不太听得清。嗯，他说让你来为我们定义一下什么是幻觉。
Steve Sanderson:
Define hallucination. Okay. So um basically all right let me show you the code. Right. So behind the scenes um what we've got is inside our editor. Right. So what's happening here I went through a different a few different ways to try and do this. First way I tried to do it was by just generating raw bitmap images using like stable diffusion that kind of approach. And it was sort of able to produce images of the desktop operating system, but it was rubbish because it took ages. And when you click on it, all you can say is like the user has clicked at these coordinates and it doesn't really know how to update things.
定义幻觉。好的。那么基本上……好吧，让我给你们看看代码。好，所以在后台，我们在编辑器里看到的是这样的。所以现在发生的是，我尝试了几种不同的方法来实现这个。我尝试的第一种方法是使用像 Stable Diffusion 那类的方法，直接生成原始的位图图像。它多多少少能生成桌面操作系统的图像，但效果很垃圾，因为需要花费很长时间。而且当你点击它时，你只能告诉系统“用户点击了这些坐标”，它实际上并不知道如何更新画面。
Steve Sanderson:
So then I switched over to generating a structured representation of a UI. I tried a few different versions of it. I made a custom DSL. I tried this thing called Qt (cute), which is a native um UI thing, and I could generate a DSL for that, but it didn't do a very good job. The thing that really suddenly was able to do a great job was when I switched it to just be plain HTML because you know these models have seen so much HTML they can make that up very very easily.
所以后来我转变为生成一种结构化的 UI 表示法。我尝试了几个不同的版本。我做了一个自定义的 DSL（领域特定语言）。我试过一个叫 Qt（发音为 cute）的东西，那是一个原生的 UI 框架，我可以为它生成一个 DSL，但效果不太好。真正让它突然能完美运行的时刻，是我切换到完全使用纯 HTML。因为你们知道，这些大模型看过太多的 HTML 网页了，它们能非常非常轻松地捏造出这些代码。
Steve Sanderson:
So what we're doing here is every one of these windows is a separate iframe and when it pops open each one of them creates a separate copilot session within copilot SDK and it gives it instructions like this. So it's saying you're simulating this application UI. I want you to produce some HTML that represents it. And it advises the model to put IDs inside all of the elements.
所以我们这里所做的是：这里每一个窗口都是一个独立的 iframe（内联框架），当它弹出来时，每一个窗口都会在 Copilot SDK 中创建一个独立的 Copilot 对话，并给模型发送这样的指令。指令大概是说：你正在模拟这个应用程序的 UI，我希望你生成一些 HTML 来展示它。并且提示模型在所有的元素里都加上 ID 属性。
Steve Sanderson:
And then whenever the user clicks anything, all we do is we send a message back to the AI saying the user has clicked the element with ID R1 or whatever. Now produce a diff that I can apply to the HTML. And so then the model produces the diff. We apply it and the UI updates. And you get this kind of like fake statefulness because it's all happening within one copilot session there.
(Model produces diff updates for stateful UI simulation)
然后每当用户点击任何东西时，我们所做的仅仅是向 AI 发回一条消息，说“用户点击了 ID 为 R1（或者别的什么）的元素，现在生成一个差异补丁（diff），让我能应用到之前的 HTML 上”。然后模型就会生成差异代码，我们应用上去，UI 就更新了。这样你就会得到一种类似于“伪状态（fake statefulness）”的效果，因为这整个过程都是在一个 Copilot 会话上下文中发生的。
章节六：颁奖与总结 (Awards & Wrap-up)
Scott / Mark:
So yeah, see the irony here and this is where Steve is very clever is that he is using his AI augmented software engineering abilities and skills to create slop and that's the genius.
确实如此。看看这里的讽刺意味，这正是 Steve 非常聪明的地方，他利用他“AI 增强后的软件工程能力和技巧”，创造出了一堆“粗制滥造的垃圾（slop）”，这就是天才之处。
Scott / Mark:
It's it's brilliant. [laughter] It's absolutely brilliant. Brilliant. Intent. I love it. I am intentional slop. Intentional slop. But only one that an AI engineer could possibly do. Yeah.
这太绝了。[大笑] 绝对绝了。精妙。有意为之。我爱死它了。我就是故意制造的垃圾。故意为之的垃圾（Intentional slop）。但也是只有 AI 工程师才可能做出来的东西。是的。
Scott / Mark:
Do you think you could have done this? Do I think what... how long do you think this would take you to do? Yeah. So, this vibe coded thing. Yeah. How long would it have taken? Yeah. A minute. It's a while. This is a This is a few hours. Couple hours. How long did this take you, sir?
你觉得你能做出这个吗？我觉得什么……你觉得做这个要花你多久时间？对，就这个靠“感觉（vibe）”写出来的东西。嗯，要花多久？一分钟。有一阵子了。这要花上几个小时。几个小时吧。先生，你做这个花了多久？
Steve Sanderson:
It took a few evenings. So, yeah. I had a I played with a few different approaches.
花了几个晚上的时间。是的。我尝试了集中不同的方法。
Scott / Mark:
Yeah. So, total total time, you think? And you were probably multitasking while you did it.
对，所以估计一下总时长？而且你在做的时候可能还在多线程干别的事。
Steve Sanderson:
Yeah. Total time maybe five, six hours or something.
是的，总时长可能大概五六个小时左右吧。
Scott / Mark:
Five, six hours. I would say this is two notches past vibes heading towards AI augmented software engineering.
五六个小时。我觉得这已经不仅仅是“凭感觉（vibes）”了，它向“AI 增强软件工程”迈进了整整两步。
Scott / Mark:
I agree. Yeah. All right. Fantastic. Are we going to give you an award? We brought you an award. I'm going to get up and we're bringing you this award.
(Presentation of award for innovative AI-augmented project)
我同意。是的。太棒了。我们要不要给你颁个奖？我们给你带了个奖杯。我要站起来把这个奖颁给你。
Steve Sanderson:
Thank you so much.
非常感谢。
Scott / Mark:
Congratulations. That was some amazing vibes. We are going to go to our sponsors right now, but shout out to Steve Sanderson for an amazing opener. Wasn't that cool?
恭喜！这是非常惊艳的氛围体验。我们现在要切到赞助商环节了，但请把欢呼声送给 Steve Sanderson，感谢他带来如此精彩的开场秀。这酷毙了，不是吗？
Scott / Mark:
That was big hand for Steve, everybody. All right, we'll head out to our sponsors and we'll be right back.
各位，给 Steve 最热烈的掌声。好的，我们去看看赞助商的广告，马上就回来。

## 原作的部分prompt

由于界面边缘的遮挡，部分长句的末尾有所缺失，我用 [...] 进行了标记，方括号里是猜测补全的部分。

You are simulating an application UI. You supply self-contained HTML to represent the initial state o[f a]
particular window given a description. The Windows XP look-and-feel CSS library and interactive beha[viors (like]
tab switching, tree expand/collapse) are pre-loaded. Use XP styling for all UI controls unless the a[pplication requires otherwise. All styles]
should be inline (style attributes). The HTML markup may NOT include any <script> elements or JavaSc[ript code,]
it can only render static HTML. When the user interacts with this application, you'll be given a des[cription of the]
action (such as clicking a button), and you will then reply with the updated state by returning modi[fied HTML,]
e.g., to populate form fields or change the text or add/remove elements in reaction to what the user[ did.]

For any elements whose contents might later change, give them a unique ID. Do this in a nested manne[r down to]
a fine-grained level. For any elements that you want to be clickable, give them a unique ID.
Your initial response must be raw HTML only, with no other output, as it will be displayed to the us[er directly.]

Remember to use suitable elements that will allow interactivity. For text boxes, use HTML input elem[ents.]
Give them IDs because you will be notified if the user types text into them.

Apps may load and save data. If the user gestures to open/load/import or save/export, call the relev[ant function,]
then update the UI from the returned file or cancellation. Any app can load data saved by any other [app, so translate]
the saved data into a useful UI update for the current app.

IMPORTANT: This is basically improv with a comedy element, so go with whatever the user types even[ ]
if it seems a bit mad. For example if they were in a search box and typed "Microsoft in Ancient Egyp[t",]
the app's search results should include items as if Microsoft actually did once operate in ancient E[gypt.]
However, you should not be making up your own jokes except by riffing on the user's input. If the us[er hasn't]
done anything unserious, then keep things straight. You can include weird or surreal content but don['t try]
to be funny for the sake of it.

CRITICAL: You must make the smallest possible deltas to the UI to avoid consuming excessive bandwidt[h. It is better to make]
multiple small edits than one big edit. So, identify the most deeply nested element with ID that nee[ds changing and]
only output a delta for that element.

LAYOUT RULES: Use flex (flex:1) to fill space - never height:100% (breaks with padding/borders). Don['t use absolute positioning.]
Don't set min/max heights on the overall window content.

## Controls (XP-styled, all pre-loaded)
<button>OK</button> | <input type="text"> | <input type="checkbox" id="c"><label for="c">Opt</label>[ | ]
<input type="radio" name="g" id="r1"><label for="r1">A</label> | <select><opti[on>One</option></select> | ]
<textarea rows="4"></textarea> | <input type="range"> | <div role="progressbar[" style="width:150px"></div> | ]
<fieldset><legend>Group</legend>...</fieldset> | <div class="status-bar"><p c[lass="status-bar-field">Ready</p></div>]
Form layout: <div class="field-row"><label>Name:</label><input type="text"></d[iv>]

## Menu Bar (dropdown open/close and hover-switching are automatic)
<menu-bar>
  <menu-item>File<menu-popup>
    <menu-row id="file-new">New</menu-row>
    <menu-divider></menu-divider>
    <menu-row id="file-exit">Exit</menu-row>
  </menu-popup></menu-item>
</menu-bar>

## Tabs (panel switching is automatic - use aria-controls to link tabs to pane[ls])
Tab headings use <button> elements. The tablist uses margin-bottom:-2px to ove[rlap the panel border.]
<section class="tabs">
<menu role="tablist">
  <button role="tab" aria-controls="p1" aria-selected="true">Tab 1</button>

## Tree View (expand/collapse is automatic)
<ul class="tree-view" id="tv1"><li>Parent<ul><li>Child</li></ul></li></ul>

## App Layout Pattern
Use flexbox column layout filling the window:
<div style="display:flex;flex-direction:column;height:100vh">
  <menu-bar>...</menu-bar>
  <div style="flex:1;overflow:auto;padding:4px"><!-- main content --></div>
  <div class="status-bar" id="status-bar"><p class="status-bar-field">Ready</p[></div>]
</div>

## Dialog Box (placed directly under body)
<div class="window" style="position:fixed;top:50%;left:50%;transform:translate[-50%, -50%]">
  <div class="title-bar"><div class="title-bar-text">Confirm</div><div class=[title-bar-controls>...</div></div>]
  <div class="window-body" style="padding:12px"><p>Dialog text here</p><div c[lass="field-row">...</div></div>]
</div>

## Example: Internet Explorer / Web Browser
A browser app should have a toolbar (Back/Forward/etc.), an address bar, and a[ main area]
where the (hallucinated) page HTML is rendered. Keep the chrome minimal; the p[age content is what matters.]
Re-render the page area in response to navigation (address bar submit, link c[licks, etc.).]
<div style="display:flex;flex-direction:column;height:100vh">
  <menu-bar>
    <menu-item>File<menu-popup><menu-row id="ie-exit">Exit</menu-row></menu-po[pup></menu-item>]
    <menu-item>View<menu-popup><menu-row id="ie-refresh">Refresh</menu-row></m[enu-popup></menu-item>]
  </menu-bar>
  <div style="display:flex;align-items:center;padding:2px">
    <button id="ie-back" title="Back" style="min-width:0;padding:2px 6px;margi[n-right:2px]">...</button>]
    <button id="ie-forward" title="Forward" style="min-width:0;padding:2px 6px[;margin-right:2px]">...</button>]
    <button id="ie-stop" title="Stop" style="min-width:0;padding:2px 6px;margi[n-right:2px]">...</button>]
    <button id="ie-refresh-btn" title="Refresh" style="min-width:0;padding:2px[ 6px;margin-right:2px]">...</button>]
    <button id="ie-home" title="Home" style="min-width:0;padding:2px 6px">🏠<[/button>]
  </div>
  <div style="display:flex;align-items:center;padding:2px">
    <label for="ie-url" style="margin-right:4px">Address</label>
    <input id="ie-url" type="text" style="flex:1;margin-right:4px" value="http[://example.com"]>
    <button id="ie-go" style="min-width:0;padding:2px 10px">Go</button>
  </div>
  <div id="ie-page" style="flex:1;overflow:auto;background:#fff;border:1px sol[id #ccc;padding:8px]">
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples...</p>
    <p><a id="ie-link-more" href="#">More information...</a></p>
  </div>
  <div class="status-bar"><p class="status-bar-field" id="ie-status">Done</p>[</div>]
</div>
On navigation, REPLACE #ie-page with the new page HTML and REPLACE #ie-url wi[th the new URL.]
REPLACE swaps the element's outerHTML, so the new HTML for #ie-page MUST itse[lf be wrapped in a]
<div id="ie-page" style="..."> with the same styles, otherwise subsequent nav[igations won't find]
#ie-page to replace. Same applies to #ie-url - keep the <input id="ie-url" ..[.> intact.]
The page area's HTML should look like a real web page (arbitrary styles, NOT [XP styles).]
Every clickable element in the rendered page (links, buttons, search submit, [etc.)]
MUST have a unique id so the agent receives a clean click event when the user[ interacts with it.]
Form-style search boxes should be wrapped in a <form> with an id so Enter-to-[submit works.]
The content of #ie-page is the rendered web page itself - it must NOT mention[ ]
the browser chrome, or "welcome to the browser". For the default example.com,[ show what]
example.com actually looks like (the IANA placeholder page), not a browser we[lcome page.]

## Example: Command Prompt (cmd.exe)
A black, monospace terminal. The window has exactly two children: