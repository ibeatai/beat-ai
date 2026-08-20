---
title: 你家的猫比 ChatGPT 更懂这个世界——一位 AI 教父刚刚为此离开了 Meta
author: Deepak Kumar
url: https://devdeepakkumar.medium.com/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over-78af3beb53e4
translated: 2026-06-03
excerpt: 我叫 Deepak Kumar，是个软件工程师，老是在技术的各个角落里反复爱上又厌倦。我之所以写东西，是因为只有把一件事讲给陌生人听，才能真正搞清楚自己到底有没有弄懂。所以别把这篇当成讲座，就当是你坐到我对面，问了一句"等等，现在 AI 圈到底在吵什么？"，我们随便聊聊。
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/01.thumb.webp
---

# 你家的猫比 ChatGPT 更懂这个世界——一位 AI 教父刚刚为此离开了 Meta

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/01.webp)

我叫 Deepak Kumar，是个软件工程师，老是在技术的各个角落里反复爱上又厌倦。我之所以写东西，是因为只有把一件事讲给陌生人听，才能真正搞清楚自己到底有没有弄懂。所以别把这篇当成讲座，就当是你坐到我对面，问了一句"等等，现在 AI 圈到底在吵什么？"，我们随便聊聊。

因为确实有一场争吵，一场真刀真枪的争吵。2025 年 11 月，一位差不多是亲手缔造了如今所谓深度学习这门学科的人，收拾好自己的桌子，走出了全世界最大的 AI 实验室，要从外面把这场仗打下去。

先从一只猫说起。

## 一岁小孩做得到，地表最强模型却做不到的事

把一杯水放在桌子边缘，旁边坐着一个一岁的小孩。这孩子还不会说话，不识字，不会数数，没法告诉你重力是什么。但你看，当他把杯子往边缘推的时候会发生什么。他身体里有什么东西已经知道了答案。他预料到杯子会掉，预料到水会洒。水花溅起来时他一点都不惊讶，只是觉得好玩。

想想这个小不点完成了多了不起的一件事。没人给他塞过物理课本，没人喂给他一千万个标好"杯子要掉"和"杯子不掉"的样本。仅仅是看了这个世界一年，大半时间还在流口水，他就学会了物体行为的那套粗略规则。

家猫做的事更厉害。它盯着两件家具之间的缝隙，估算自己的身子能不能挤过去，判断要跳多高，把即将落脚那块湿滑表面也算进去，然后一气呵成地完成整套动作。没有奖励函数，没有训练流程，也没有一座在俄勒冈嗡嗡作响的数据中心。

接下来是让人不太舒服的部分。地球上最先进的那些大语言模型——刚写出一首过得去的十四行诗、帮你 debug 了代码、还考过了律师资格考试的那些——做不到猫做的事。它们根本不知道没东西托着的杯子会掉。它们能给你写一段关于重力的漂亮文字，引用牛顿的话，可它们脑子里并没有一个能用来预测真实物理世界下一步会怎样的、可运作的重力模型。

[Yann LeCun](https://en.wikipedia.org/wiki/Yann_LeCun) 因为帮忙搭建了驱动这些模型的神经网络，拿过计算机科学的最高荣誉。他反复强调的正是这一点。2026 年 1 月接受 MIT Technology Review 采访时他指出，我们至今都没有一个像家猫那样灵巧的家用机器人，也没有真正意义上的自动驾驶汽车——原因恰恰在于，这些系统缺少一个关于世界的模型。（[MIT Technology Review](https://www.technologyreview.com/2026/01/22/1131661/yann-lecuns-new-venture-ami-labs/)）

一个能谈论世界的模型，和一个能真正预测、驾驭世界的模型，这中间的鸿沟，就是整篇文章的全部。后面所有内容都挂在这条裂缝上。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/02.webp)

## 那么，世界模型到底是什么

剥掉术语，世界模型其实就这么回事：对事物如何运转的一幅内在图景，好到足以让你想象出接下来会发生什么。

你就有一个。此刻，不用刻意去想，你就能回答这类问题：我松开这支笔，它往哪去？这只杯子倾斜得太过，会洒出什么？我推这扇门，它朝哪边开？这些答案不是你从记忆里调出来的。你是在脑子里跑了一个小小的模拟，用的是一辈子磕磕碰碰积累出来的世界模型。

扫地机器人有个很粗糙的版本。它存着家里大致的地图，所以撞到沙发后不会一直没完没了地撞下去。自动驾驶汽车需要的要丰富得多：它得在那个追着球往马路冲的孩子真正踏进车道**之前**就预判到——因为等孩子上了路再刹车，已经来不及了。

世界模型之所以特别，之所以不同于霸占头条的那些聊天机器人，靠的是三件事。我想把每一件都真正讲清楚，而不是罗列一下了事。

第一，它预测未来，而不是过去。语言模型训练出来是猜句子里的下一个词。世界模型训练出来是猜**现实的下一个状态**。听着像，其实是两件天差地别的任务。预测"猫坐在 \_\_\_ 上"是个文字游戏。预测"这摞积木接下来要 \_\_\_"则要求你知道塔会倒。

第二，它主要靠看来学，而不是靠人教。世界不会自带标签。你小时候看着一只玻璃杯摔碎，没人跳出来在这件事上盖个章"玻璃，易碎，受撞即破"。你就这么看着，然后更新了自己的理解。这种系统从原始观察中自学的方式，研究者叫它自监督学习，它是大多数正经世界模型工作引擎盖底下的那台发动机。

这正是 Yann LeCun 念叨了将近十年的一个类比的核心，一旦听过，你就再也忘不掉。他说，假如智能是一块蛋糕，那么自监督学习——光靠观察世界来学的那种——就是整块蛋糕的海绵蛋糕主体。监督学习——人替你仔细标注的那种——只是薄薄一层糖霜。而强化学习——靠偶尔的奖励来学的那种——不过是顶上孤零零的那颗樱桃。他 2016 年在一场重要的 AI 大会上头一回端出这块蛋糕，2019 年又调了调配方。（[SyncedReview](https://medium.com/syncedreview/yann-lecun-cake-analogy-2-0-a361da560dae)）这个说法残酷在它的简单。心智里绝大部分东西，是从看里搭起来的，不是从标签里，也不是从奖励里。可过去这几年，我们一门心思盯着糖霜和樱桃，却几乎不知道怎么把蛋糕本身烤出来。

LeCun 还喜欢用一个粗略估算的数字把这件事说实，我头一回听到时整个人愣住了。一个普通的四岁小孩，仅仅是醒着、四处张望，通过视觉系统过的原始信息量，就已经超过最大的语言模型从人类有史以来数字化的全部文字里吸收到的总和。一个孩子盯着世界看四年，胜过整个互联网的文字。如果这哪怕大体属实，那么"你能光靠读书读出一颗完整的心智，不需要眼睛、不需要手、不需要跟现实有任何接触"这种想法，就不太像一场大胆的押注，反而更像是搞错了类别。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/03.webp)

第三，它让你能做规划。一旦你能想象出后果，你就能挑选动作。走高速还是走小路？你不会两条都开一遍再说。你在脑子里把两条都模拟一遍，挑更好的那条。一个有着良好世界模型的系统也能这么干：想象出几个可能的动作，预测每一个会把你带到哪里，然后选。没有世界模型的系统，本质上就是闭着眼瞎反应，碰运气。

所以下次听到"世界模型"，别脑补成一个普通聊天机器人。脑补成你脑子里那个早就知道杯子会掉的东西。

## 让整件事跑起来的那个安静的小把戏

现在到了精妙的部分，而且是真精妙，所以跟紧我。

搭世界模型最直接的办法，似乎应该是把未来一五一十地预测出来。给系统看一段视频，让它生成下一帧，每一个像素都生成出来，就像 ChatGPT 预测下一个词那样。人们就这么实打实试了好几年。结果是一场灾难，而搞清楚为什么值得，因为它能解释后面整个故事。

设想一段视频：一个球朝桌沿滚去。在训练数据里，球有时弹向左边，有时弹向右边。一个语言模型补完"球弹向了 \_\_\_"这句话很轻松。它给"左"和"右"各留一份小小的概率，同时握着两种答案，心安理得。但一个被逼着画出唯一一帧下一画面的模型没法这么含糊其辞。面对一个左右皆可的球，它能做的最稳妥的事就是取个折中，画出两个未来的平均——结果就是一个糊成一团、像鬼影一样同时出现在两处的球。把这种预测往前多推几步，整段视频就化成了一锅灰汤。

把可能性数一数，情况更糟。语言模型从大概五万个词的词表里挑。一帧高清视频有几百万个像素，每个像素能取几百个值，算下来，可能的下一帧比可观测宇宙里的原子还多。你根本枚举不过来。于是模型只能含糊，而含糊看起来就是一片模糊。LeCun 在这里有个他最爱举的例子。他说，训练一个模型去预测行车记录仪的视频，它会把几乎全部力气都耗在预测路边树叶的随机颤动上——这种运动不带任何有用信息，却有海量的像素在动。

LeCun 对这个问题的回应，是他在 Meta 研发的一种架构，叫 JEPA，全称 Joint Embedding Predictive Architecture（联合嵌入预测架构）。这名字太拗口，所以我直接把直觉递给你。

JEPA 不去用像素预测未来。它用**想法**预测未来。

想想你实际上是怎么预判世界的。看着别人去拿一只咖啡杯时，你不会去预测从陶瓷上反射的每一颗光子的精确位置。你预测的是某种抽象的、高层次的东西："手伸向杯子，手指扣住把手，杯子被端起来。"你扔掉了那些你没法知道的细节，攥住了真正要紧的大意。JEPA 做的恰恰是这个。它学着把一个场景表示成一束压缩过的意义，然后在这个意义的空间里做预测，忽略掉那些无法预测的噪声。（[MIT Technology Review](https://www.technologyreview.com/2026/01/22/1131661/yann-lecuns-new-venture-ami-labs/)）

LeCun 把这描述成一个系统在学习世界底层的规则——就像一个婴儿学会重力那样，单凭看，一遍又一遍，直到那套规律沉淀下来。

这个想法里藏着一个有名的陷阱，研究者花了几十年才攻克。假设你简单地告诉模型："看同一个场景的两个视角，让你对它们的内部描述彼此吻合。"听上去挺合理。但模型会找到一条偷懒的捷径。它可以学着把所有东西——整个世界里的每一个场景——都映射成一模一样的一段空白描述，比方说全是 1 的一串数。这样两个视角永远完美吻合，数学上交了差，模型却什么都没学到。研究者把这叫表征坍缩。想象一个学生，每道考题都答"看情况"。严格来说，这学生永远不会错。可他也彻底没用。

这不是新问题，而它的解法有个特别有人情味的来历。早在 1990 年代初的贝尔实验室，LeCun 造了个叫 Siamese network（孪生网络）的东西来抓伪造签名：喂进两个签名，训练系统在它们出自同一只手时产出吻合的指纹特征、不是同一只手时产出冲突的特征。同样的把戏——让模型在两个视角间跟自己达成一致——正是现代世界模型背后的动力。难的一直是怎么阻止坍缩。突破出现在 2020 年前后，LeCun 和一位名叫 Stephane Deny 的博士后落到一个他们称为 Barlow Twins 的办法上，借用了神经科学家 Horace Barlow 六十年前关于眼睛里的神经元如何避免在冗余信息上白费力气的一个想法。像 Barlow Twins 和 DINO 这类方法，骨子里都是些聪明的法子，逼着模型把自己对世界的内部图景保持得丰富而具体，而不是坍缩成一摊糊。这些东西上不了头条。可在那些激动人心的部分能跑起来之前，它们全都得先被解决掉。

这不是个思想实验。Meta 真把它造出来了。他们的模型叫 V-JEPA 2，在超过一百万小时的互联网视频上训练，没有任何人工标注，它学会了预判物理场景如何展开。然后，在上面叠加不到六十二小时的机器人录像，同一个模型就能在一间它从没见过的实验室里操控一只真实的机械臂，抓取并摆放物体，对那个环境没有做过任何针对性的任务训练。（[Meta AI](https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks/)，[arXiv 论文](https://arxiv.org/abs/2506.09985)）

停下来体会一秒。一百万小时纯看，再加一丁点上手练习，这东西就能在一个它从未踏足的房间里行动。这比我们接下来要讲的那套蛮力打法，要接近人学一项新任务的方式得多。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/04.webp)

## 为什么现代 AI 里最聪明的点子，同时也最懒

接下来我得给另一方说句公道话，因为另一方有一个非常有力的论点，外加十年的实绩。

2019 年，一位名叫 Richard Sutton 的研究者——他是整整一个 AI 分支强化学习的奠基人之一——写了一篇短文，叫《The Bitter Lesson》（苦涩的教训）。全文勉强一千字，却塑造了这个领域里几乎超过任何一份文档的、价值数十亿美元的决策。用他自己的话说，核心主张是：利用算力的通用方法终究是最有效的，而且优势巨大。（[Richard Sutton, "The Bitter Lesson"](https://www.cs.utexas.edu/~eunsol/courses/data/bitter_lesson.pdf)）

让我把这句从学术腔里翻出来，因为要理解今天的 AI 为什么长成这副模样，这是头一等重要的想法。

七十年来，聪明的研究者一直试图手工搭建智能。他们会细心地把好棋的原则教给一个国际象棋程序。他们会满怀爱意地把语法规则编码进一套翻译系统。可每一次，时间拉得够长，总会冒出一个更笨、更通用、单纯往问题上猛砸海量数据和算力的打法，把那个手工精雕的版本碾碎。聪明的、人类设计的方案会触顶停滞。蛮力的、堆规模的方案则一路往上爬。

这就是苦涩之处。苦涩在于，它暗示我们的聪明、我们关于思考该如何运作的那些优雅理论，大多只是碍事。历史反复教的这一课是：别耍聪明了，加更多算力，喂更多数据，然后让开路。

大语言模型就是苦涩教训被推到逻辑极致的产物。没人坐下来教过 GPT 什么是隐喻、怎么写 Python。人们拿了荒唐多的文字——差不多是可读的整个互联网——训练一个模型去做一件蠢到极点的简单事：预测下一个词，亿万亿万次。就从这个无脑的猜词游戏里，长出了某种乍看吓人地像流畅表达和推理的东西。

所以当批评者管 LLM 叫"蛮力"时，他们没全说错，但这个词在偷偷耍花招。是的，它确实是蛮力，在它靠的是惊人规模而非优雅设计这个意义上。可蛮力**赢了**。一次又一次。苦涩教训的全部要点就是：那个不优雅、没优化、贪吃算力的打法，恰恰是那个不断击败漂亮打法的赢家。

这是把对方论点立得最稳的版本。这也是为什么那些最大公司里最聪明的人押注于它，并不是傻。历史站在他们这边。

有意思的问题是，历史是不是即将做一件它从没做过的事——停下来。

## 没人料到的剧情反转

接下来这部分，我头一回读到时是真的笑出了声。

Richard Sutton，写下《The Bitter Lesson》——那篇被整个扩规模教派当圣经引用的文章——的那个人，认为大语言模型是条死路。

2025 年底，在 Dwarkesh Podcast 的一场长谈里，Sutton 论证说，要抵达真正的智能，LLM 是一条"死路"。（[Dwarkesh Podcast](https://www.dwarkesh.com/p/richard-sutton)）他的论证很锋利，直插整场辩论的骨头。

Sutton 的看法是：真正的智能，动物和人所拥有的那种，来自在**活着**中学习。你做一个动作，看会发生什么，得到一点关于这动作好坏的信号，然后调整。接着再来一遍。一遍又一遍。永不停歇。你从不停止学习。一个学走路的孩子，没有一个会结束的"训练阶段"，结束之后就被冻结、出厂。他们就是一直走下去，摔倒，调整，进步，往后一辈子都这样。

大语言模型不是这么干的。它们被一次性地、在一大堆人写的文字上训练好，然后冻结、部署。它们学会的是模仿人类已经说过的话。而 Sutton 最深的反对意见是：一个纯粹建立在模仿人类文字之上的智能，永远封顶在人类水平。它能把我们已知的东西重新混搭，却没办法从它自己对世界的经验里学到真正全新的东西——因为它压根没有对世界的经验。它只有我们关于世界的文字。

看看这个论点奇怪的形状。苦涩教训说：别再编码人类知识了，让机器自己学。而 Sutton 现在指出，LLM 尽管规模庞大，在某种意义上却恰恰是它的**反面**。它们浸泡在人类知识里。它们是模仿机器。那个告诉我们别再依赖人类先验的人，眼看着行业造出了史上最大的人类模仿引擎，还管它叫通往超级智能的路，而他不买账。

LeCun 和 Sutton 在很多事上意见相左。但在这一件事上他们一致，而且两位脾性大不相同的图灵奖得主竟落到同一个疑心上，这本身就重要：把猜词这件事扩大规模，不会一路把我们走到一颗心智那里。缺了点什么，而缺的那个东西，是与现实的接触。

## 那为什么每家万亿美元的公司还是一头扎进它

如果这个领域里几位最受推崇的头脑都认为 LLM 是条弯路，那为什么微软、Google、OpenAI——还有直到不久前的 Meta——往里砸了几百亿美元？

我觉得有四个诚实的理由，没有一个是愚蠢。

第一个理由是，LLM **现在**就能用，而世界模型大多还不能。这是最关键的一条。研究中的梦想没法发货。当世界模型还在实验室里捡木块时，ChatGPT 已经有了好几亿用户。一家公司这个季度要对客户和股东交代，而不是对一个或许十年后才回本的愿景交代。LeCun 自己估计，基于世界模型路线建成的人类水平机器智能，可能要花十年甚至更久。在科技圈，十年是一个地质纪元。（[Nasdaq / RTTNews](https://www.nasdaq.com/articles/metas-chief-ai-scientist-yann-lecun-depart-and-launch-ai-start-focused-world-models)）

第二个理由就是我们刚讲过的苦涩教训本身。扩规模赢了太多回，赌它输反倒显得鲁莽。每次有人说"扩规模就要撞墙了"，下一个更大的模型就跨过了那堵墙。如果你整个职业生涯都因为加更多算力而被嘉奖，你就会继续加更多算力。

第三个理由是纯粹的经济账。把一个 LLM 扩大规模的配方，如今已经摸得很透。你大致知道多加数据、多加芯片，模型能变好多少。这是一笔已知的、可融资的、可规划的押注。相比之下，世界模型的研究突破，按定义就是不可预测的。你没法把"取得一次概念性突破"写进季度路线图。投资人和高管钟爱一个能拧的旋钮。扩规模就是个旋钮。天才不是。

第四个理由最具人性，而且是一个跟专业能力一样古老的陷阱。当你非常擅长某件事，你往往会以为你做这件事的方式就是那个方式。AI 里有一种被记录在案的模式：团队高估自己的聪明和现成的打法，抗拒一个无聊的真相——换个路子也许更好。一旦整个行业都在一种架构上标准化了，雇了成千上万专精于它的人，还围着它建起了芯片和数据中心，这个行业就对"你押错了"这种说法产生出强烈的免疫反应。

所以这些公司偏向一种"不智能、未优化的算法"，不是出于愚蠢。它们偏向的是今天能赚钱、可预测地扩规模、又契合他们已有技能的那个东西。这是一笔理性的押注。它也可能是错的。两者可以同时成立，而这份张力，正是此刻这出戏的全部。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/05.webp)

## 他不是被赶走的，他是腻了

现在说回这一切的中心人物，以及那一步——把一场缓慢的研究争论变成头版头条的那一步。

Yann LeCun 是现代 AI 所谓的几位教父之一。他帮忙开发了让计算机视觉跑起来的卷积神经网络，这也是你手机能从一张照片里认出你的脸的部分原因。他在 Meta 还叫 Facebook 的年代创办并搭建了它的基础研究实验室 FAIR。他分享了图灵奖，这是计算机科学里最接近诺贝尔奖的东西。无论从哪个角度看，他都是 AI 王室。

而在 2025 年 11 月，他宣布要离开 Meta，去创办自己的公司。（[CNBC](https://www.cnbc.com/2025/11/19/meta-chief-ai-scientist-yann-lecun-is-leaving-the-company-.html)）

把这件事说成一场闹翻会很干净利落，但真实的故事更有意思，也带点伤感。过去这一两年，Meta 把它的 AI 战略重塑得恰好围着 LeCun 所怀疑的那个东西。它的 Llama 4 模型在与对手的较量中砸了锅之后，Mark Zuckerberg 全押在公司所谓的超级智能上。他花了超过一百四十亿美元把 Scale AI 创始人 Alexandr Wang 招了进来，组建了一支新的精英部队，而 LeCun 的研究实验室越来越被指向出产品，而不是去追他在意的那种长周期科学。（[Nasdaq / RTTNews](https://www.nasdaq.com/articles/metas-chief-ai-scientist-yann-lecun-depart-and-launch-ai-start-focused-world-models)）

LeCun 在采访里对此一直很有风度。他说 FAIR 是一次研究上的成功，公司只是在该追什么这件事上做了不同的选择。他甚至放话，Meta 也许最终会成为他新公司的第一个客户。这里没有公开的对骂。（[MIT Technology Review](https://www.technologyreview.com/2026/01/22/1131661/yann-lecuns-new-venture-ami-labs/)）字里行间有的，是一位创办了一个地方来做耐心、基础研究的科学家，眼看着那个地方被改造成一座产品工厂，然后悄悄决定，他宁可去亲手造一个他真正相信的未来。

那个未来有名字。他的新公司叫 AMI Labs，是 Advanced Machine Intelligence（先进机器智能）的缩写，总部设在巴黎。他取这个名字有一部分是因为"ami"在法语里是"朋友"的意思——这种细小而温暖的细节，能让你窥见一个人的某种底色。他出任执行董事长，由一位多年的同事 Alex LeBrun 担任 CEO 负责运营，同时他保留着自己在 NYU 的教授职位。这家公司押的全部就是面向物理世界的世界模型，也就是 JEPA 那条路，刻意**不**去跟其他所有人在跑的聊天机器人竞赛较劲。（[MIT Technology Review](https://www.technologyreview.com/2026/01/22/1131661/yann-lecuns-new-venture-ami-labs/)）公司落地页上那一行字，把整套哲学压缩成了一句：真正的智能不始于语言，它始于世界。

而最让我意外的是这部分，因为它不是你会猜到的。AMI 的头几个目标不是聊天机器人，甚至不是家用机器人。在最近一次采访里，LeCun 说，未来一两年的计划是把世界模型对准那些复杂的工业系统——它们的行为太纠缠，没法塞进一组干净利落的方程里。一台喷气发动机。一座化工厂。一张电网。最让人吃惊的是，一个人类病人。他提出，如果你能给一个糖尿病人的身体建一个良好的预测模型，你就能规划一套疗程去调控他的血糖，甚至琢磨出怎么诱导一颗干细胞变成那种能分泌胰岛素的细胞。同样的思路指向发现新材料、新催化剂、新电池。他的逻辑一以贯之到底。一个能预测某个动作后果的系统，在任何后果重要的地方都有用——也就是说，在哪儿都有用。在他的讲法里，聊天机器人是配角。他在同一次采访里直白地说，今天的语言模型擅长摆弄语言，但基本上别的什么都不行。（[Welch Labs interview, part one](https://www.youtube.com/watch?v=kYkIdXwW2AE)）

还有一个细节我觉得意味深长。2025 年 12 月 LeCun 在巴黎发言时明确表示，Meta 不会是这家新公司的出资方。（[Bloomberg](https://www.bloomberg.com/news/articles/2025-12-04/ai-pioneer-lecun-says-meta-won-t-invest-in-world-model-startup)）他正从地球上最有钱的公司之一的资源里抽身离开，去追一个或许要十年才结果的想法。你不会为了一份薪水这么做。你这么做，是因为你认为整个行业正自信满满地朝错误的方向狂奔，而你再也受不了在里面眼睁睁看着。

## 走向门口的，不止他一个

如果这只是一个有名的唱反调者，你大可把它当成一位天才的古怪十字军。但它不止一个人。

Fei-Fei Li，这位常因在计算机视觉上的奠基性工作而被称为 AI 教母的计算机科学家，正从另一个角度提出一个惊人相似的论点。她把自己的版本叫"空间智能"，她的公司 World Labs 在 2025 年 11 月发布了一个叫 Marble 的模型，能从一句简单的文字或图像提示生成可探索的 3D 世界。她在网上发布的一份宣言里把论点说得很直白：要造出真正智能的机器，需要某种比语言模型更有野心的东西，因为今天的 LLM 并不真正理解我们身处的这个物理的、三维的世界。（[TechCrunch](https://techcrunch.com/2025/11/12/fei-fei-lis-world-labs-speeds-up-the-world-model-race-with-marble-its-first-commercial-product/)，[Radical Ventures](https://radical.vc/building-spatially-intelligent-ai/)）

她的说法一直在我脑子里挥之不去。她把空间智能描述为人类思维的脚手架。早在你有词语之前，你就理解空间。你知道妈妈在门**后面**，玩具在沙发**底下**，那条缝你能**钻过去**。语言是后来才有的，是搭在那个空间地基之上的。她的论点是，我们一直在用错误的顺序造 AI——从语言起步，指望对世界的理解会从中莫名其妙地掉出来——而在活物身上，它的运作方向恰恰相反。

而且这不只是两位带着创业公司的学者。Google DeepMind 也一直在造世界模型，包括一个能生成可交互、可导航环境的系统 Genie 3。NVIDIA 和其他一些公司也在绕着同一片地盘打转。如今已经有一个虽小但在壮大的群体——其中还有些人就在最大的实验室里——把"给世界建模，而不只是给词语建模"当成下一个真正的前沿。（[Themesis survey of world model approaches](https://themesis.com/2026/01/07/world-models-five-competing-approaches/)）

所以这已经不再是个边缘观点。它正在成为一个派系。而范式的转移，正是靠派系完成的。

## 头条略去的那一段

LeCun 那笔十亿美元押注的耸动报道，往往会跳过下面这件事，而我觉得略去它就是不诚实，所以我不略。眼下，今天，他的路线正在输。

正当世界模型要花六十秒才能把一只杯子挪过桌面时，主流的语言驱动型机器人正干着真正叫人瞠目结舌的事。一家叫 Physical Intelligence 的创业公司展示过会削西葫芦、会叠纸风车、会倒垃圾的机器人。这些跑在这个领域所谓的视觉-语言-动作模型上，而这种机器人大脑，恰恰建在 LeCun 说是死路的那些大语言模型之上。按你今天下午会去看的任何一个公平计分牌，基于 LLM 的路线正在赢下这场机器人竞赛，而且不是赢一点，是赢一大截。

LeCun 自己也清楚。当 Welch Labs 团队直接问他，他的世界模型路线会不会超过那些机器人时，他的回答没有半点含糊。他说那些模型注定要完，说它们基本上跑不好。（[Welch Labs interview, part two](https://www.youtube.com/watch?v=v_jDvpEGTIg)）体会一下这有多奇怪。一个自己的演示一分钟才挪动一只杯子的人，看着那些会叠衣服的机器人，却管这些会叠衣服的机器人叫死路。他不是在说自己的路线现在更好。他是在说，那个亮眼的东西是个陷阱，而那个不亮眼的东西才是真正的路，他愿意拿一家公司和十年光阴来押。

他对自身短板的坦诚，反倒是他身上最有说服力的地方。当下的世界模型机器人，只能可靠地往前规划大约五步，再往后它想象的未来就漂离现实太远、不可信了。他提出的解法，是他称之为分层世界模型的东西，他用一个谁都能体会的例子来解释。如果你坐在纽约的办公室里，想明天到巴黎，你不会把这趟行程当成一连串以毫秒为单位的肌肉抽动来规划。你在顶层用大块的抽象单元来规划——到机场，赶上航班——只有当你真的从椅子上站起来时，才去填那些精细的运动细节。一套分层的世界模型也会这么干，在顶上做粗略的远程规划，在底下做精确的近程规划。这套东西到了大规模上是否真行得通，照他自己承认，还是个悬而未决的研究问题。

这把我带到一个我老是忍不住去想的对比。大约三十年前，LeCun 造出了能从支票上读出手写数字的神经网络。当时那些感觉像是可爱、狭窄的玩具。没有人看着一台眯着眼认邮政编码的机器，会猜到同样这套基本想法，放大上百万倍，有一天会写论文、考过律师资格考试。所以当他那些挪杯子的机器人今天看着像玩具时，诚实的问题不是"它们怎么这么烂？"，而是"这是不是又一个 1995 年？"。也许这些是会扩张成一切的玩具。又或许，这一回，玩具就只是玩具。没人真的知道，而任何告诉你他知道的人，都是在虚张声势。

![](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/your-cat-understands-the-world-better-than-chatgpt-and-one-of-ais-godfathers-just-quit-meta-over/06.webp)

## 这整个故事里，没人能告诉你智能到底是什么

我想用一个藏在一切底下的问题来收尾，那个在这些辩论里几乎从不被说出口的问题，因为它令人难堪。

我们其实并不知道**智能到底是什么**。

我们成天用这个词。我们争论机器有没有它、它们什么时候会有它、它会不会超过我们的。我们围着追逐"通用人工智能"建起了一个万亿美元的产业。可是，要是你把十位专家关进一个房间，让他们各写下一条精确的智能定义，你会得到十一个答案外加一场打架。LeCun 本人就认为 AGI 是个定义如此糟糕的词，以至于他拒绝使用——这也是他的公司为什么叫 Advanced **Machine** Intelligence（先进机器智能）的部分原因。

有一个事实应该让我们所有人谦卑下来。有个东西叫莫拉维克悖论，以机器人专家 Hans Moravec 命名，是他早在 1988 年指出的。（[Moravec's paradox](https://en.wikipedia.org/wiki/Moravec%27s_paradox)）这个悖论是说：我们以为是人类智能巅峰的那些事，比如下棋、做微积分、考试，对计算机来说反倒相对容易。而我们斥为琐碎的那些事，任何一个幼童或动物都做得来的，比如穿过一间杂乱的屋子、认出一张脸、捡起一只杯子而不把它捏碎，对机器来说反倒难得吓人。

LeCun 在那次 MIT 采访里直指这个悖论，把它当成 LLM 为什么显得比实际聪明得多的原因。它们攻克了"智能"里我们恰好觉得了不起的那些部分——言语的、符号的，却在那些其实花了进化好几亿年才搞定的部分上无能为力。我们造出了能辩论哲学却没法可靠地装好一台洗碗机的机器，然后我们就被它们到底有多聪明这件事搞糊涂了。

想想这揭示了什么。我们关于什么算"智能"的整套直觉，是反着的。在某个想象中的智力层级里，我们把象棋特级大师排在清洁工之上，可从工程的角度看，清洁工在一个混乱物理世界里毫不费力的穿行，才是难得多的问题。窗台上的那只猫，正在解决地球上还没有任何机器完全攻克的难题。

所以当人们信心十足地宣布 AI"几乎和人一样聪明"或者"就要变成超级智能了"，我诚实的反应是：聪明在**哪件事**上？按**谁的**定义？怎么**量**的？我们正用一份自己写的考卷给机器打分，这卷子恰好奖励机器本就擅长的东西，然后又装作很意外它们拿了满分。

我不是说智能是个虚构。显然，你脑子里、那只猫脑子里，确实有某种真实的东西在发生。我想说的是，"智能"不是一件你能放在单个旋钮上往上拧的、干净利落的东西。它是一束乱糟糟的、彼此非常不同的能力，其中一些我们已经出色地自动化了，另一些我们才刚刚开始触碰。整场 AGI 对话，常常感觉像一群人在为一个连坐标都谈不拢的目的地激烈争吵。

## 我自己到底怎么想，仅供参考

我不知道谁对。我想对这点坦白，因为在这场辩论里向你兜售确定性的人，都是在向你兜售点什么。

但把这一切都琢磨过后，我落在这里。

LLM 这一派说得对：扩规模创造过奇迹，你绝不该轻率地赌它输。苦涩教训是真的，产品是真的，而把这十年最有用的技术斥为一种"蠢算法"，是那种当下显得很聪明、过后却很难看的事。

世界模型这一派也说得对：前方某处有一堵墙，一个只懂我们的文字、从没碰过世界的系统缺了某种本质性的东西，而一个谈论重力的模型和一只活在重力之中的猫之间的鸿沟，不是一个可以靠扩规模抹平的细节。它就是整个问题本身，仍旧搁在那里，没解决。

最打动我的，是 LeCun 所做的这个选择的形状。一个站在领域顶端、有一切理由继续待在舒适区里的人，看着整个行业围着组织起来的那份共识，平静地说，我觉得你们想错了，而我要拿我接下来十年的人生去证明这一点。他可能会失败。世界模型也许会变成那条优雅的死路，而蛮力扩规模会成为那个丑陋的赢家，恰如苦涩教训所预言。历史不站在他这边。

但科学里的每一个范式，都曾显得不可战胜，直到它被战胜的那一刻为止；而最先看见裂缝的那个人，通常会先被当成怪人一阵子，然后才像个先知。

我会盯着那只猫看。

*谢谢你读到这里。如果你觉得我哪里说错了，我宁可听到也不愿不知道，所以来跟我争吧。我聊过的最棒的几场 AI 对话，都是从有人告诉我"你没抓住重点"开始的。*

## 资料来源与延伸阅读

本文中所有事实性内容都取自下面的报道与研究。我建议你读读原文，尤其是那篇 MIT Technology Review 的采访，它是 LeCun 用自己的话对其思路最清晰的一次陈述。

1.  Caiwei Chen, "Yann LeCun's new venture is a contrarian bet against large language models," *MIT Technology Review*, January 22, 2026. [https://www.technologyreview.com/2026/01/22/1131661/yann-lecuns-new-venture-ami-labs/](https://www.technologyreview.com/2026/01/22/1131661/yann-lecuns-new-venture-ami-labs/)
2.  "Meta chief AI scientist Yann LeCun is leaving the company," *CNBC*, November 19, 2025. [https://www.cnbc.com/2025/11/19/meta-chief-ai-scientist-yann-lecun-is-leaving-the-company-.html](https://www.cnbc.com/2025/11/19/meta-chief-ai-scientist-yann-lecun-is-leaving-the-company-.html)
3.  "AI Pioneer LeCun Says Meta Won't Invest in 'World Model' Startup," *Bloomberg*, December 4, 2025. [https://www.bloomberg.com/news/articles/2025-12-04/ai-pioneer-lecun-says-meta-won-t-invest-in-world-model-startup](https://www.bloomberg.com/news/articles/2025-12-04/ai-pioneer-lecun-says-meta-won-t-invest-in-world-model-startup)
4.  "Meta's Chief AI Scientist Yann LeCun To Depart And Launch AI Start-Up Focused On 'World Models'," *RTTNews via Nasdaq*, November 2025. [https://www.nasdaq.com/articles/metas-chief-ai-scientist-yann-lecun-depart-and-launch-ai-start-focused-world-models](https://www.nasdaq.com/articles/metas-chief-ai-scientist-yann-lecun-depart-and-launch-ai-start-focused-world-models)
5.  "Introducing the V-JEPA 2 world model and new benchmarks for physical reasoning," *Meta AI*, June 11, 2025. [https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks/](https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks/)
6.  Mido Assran et al., "V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning," *arXiv:2506.09985*, June 2025. [https://arxiv.org/abs/2506.09985](https://arxiv.org/abs/2506.09985)
7.  Richard S. Sutton, "The Bitter Lesson," 2019. [https://www.cs.utexas.edu/~eunsol/courses/data/bitter\_lesson.pdf](https://www.cs.utexas.edu/~eunsol/courses/data/bitter_lesson.pdf) （概览：[https://en.wikipedia.org/wiki/Bitter\_lesson)](https://en.wikipedia.org/wiki/Bitter_lesson\))
8.  "Richard Sutton: Father of RL thinks LLMs are a dead end," *Dwarkesh Podcast*, September 2025. [https://www.dwarkesh.com/p/richard-sutton](https://www.dwarkesh.com/p/richard-sutton)
9.  "Fei-Fei Li's World Labs speeds up the world model race with Marble," *TechCrunch*, November 12, 2025. [https://techcrunch.com/2025/11/12/fei-fei-lis-world-labs-speeds-up-the-world-model-race-with-marble-its-first-commercial-product/](https://techcrunch.com/2025/11/12/fei-fei-lis-world-labs-speeds-up-the-world-model-race-with-marble-its-first-commercial-product/)
10.  Fei-Fei Li, "Building Spatially Intelligent AI" (manifesto excerpt), *Radical Ventures*, November 17, 2025. [https://radical.vc/building-spatially-intelligent-ai/](https://radical.vc/building-spatially-intelligent-ai/)
11.  "World Models: Five Competing Approaches," *Themesis*, January 2026 (survey covering JEPA, World Labs, and DeepMind's Genie 3). [https://themesis.com/2026/01/07/world-models-five-competing-approaches/](https://themesis.com/2026/01/07/world-models-five-competing-approaches/)
12.  "Moravec's paradox," background reference. [https://en.wikipedia.org/wiki/Moravec%27s\_paradox](https://en.wikipedia.org/wiki/Moravec%27s_paradox)
13.  Welch Labs, "Yann LeCun's Billion Dollar Bet" (part one of a two-part series on JEPA and world models, featuring interview footage with LeCun), YouTube, 2026. [https://www.youtube.com/watch?v=kYkIdXwW2AE](https://www.youtube.com/watch?v=kYkIdXwW2AE)
14.  Welch Labs, "Yann LeCun's $1B Bet Against LLMs (Part 2)" (covers V-JEPA 2, VL-JEPA, world-model planning, and LeCun's near-term roadmap for AMI Labs), YouTube, 2026. [https://www.youtube.com/watch?v=v\_jDvpEGTIg](https://www.youtube.com/watch?v=v_jDvpEGTIg)
15.  "Yann LeCun Cake Analogy 2.0," *SyncedReview*, February 22, 2019 (origin and 2019 update of the "intelligence is a cake" analogy). [https://medium.com/syncedreview/yann-lecun-cake-analogy-2-0-a361da560dae](https://medium.com/syncedreview/yann-lecun-cake-analogy-2-0-a361da560dae)
