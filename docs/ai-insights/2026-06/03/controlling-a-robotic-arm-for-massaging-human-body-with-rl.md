---
title: 用强化学习控制机械臂为人体按摩
author: Yasin Yousif, Ph.D
url: https://levelup.gitconnected.com/controlling-a-robotic-arm-for-massaging-human-body-with-rl-e9df3eb2ea3d
translated: 2026-06-03
excerpt: 用交互式强化学习实现最优行为
tags:
  - Machine Learning
  - Technology
cover: https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/01.thumb.webp
---

# 用强化学习控制机械臂为人体按摩

用交互式强化学习实现最优行为

![图1：示意图（作者用 AI 生成）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/01.jpg)

*本文探讨强化学习（RL）如何大幅提升机械臂的能力。重点在于 RL 怎样帮助落地真实世界的机器人方案——也许不是直接落地，而是作为一种额外的增强学习技术，叠加在一个基础的手工方案之上。*

*把直接方案（这里是通过检测并规划末端执行器路径来做按摩）跟作为增强技术的 RL（用来应对引入的误差）结合起来——这个思路被证明非常实用，不只在本例里，在我见过的其他应用里也是。*

> *本文基于我参与的一个项目，隶属* [*Fellowship.ai*](https://www.fellowship.ai/) *计划。* [*源码在这里*](https://github.com/fellowship/massage_robot)*！*

### **相关往期文章：**

-   RL 入门

-   Max-Ent IRL 深度指南

-   训练 RL 模型的最佳实践

## 引言

腰背和颈部疼痛是极普遍的状况，约 60% 到 80% 的成年人[一生中](https://journals.lww.com/md-journal/fulltext/2017/05190/trends_in_diagnosis_of_painful_neck_and_back.3.aspx)都会遇到。这些不适来源各异，小到长时间工作或姿势不良带来的单纯肌肉紧张，大到脊柱移位、与年龄相关的骨质疏松等更严重的潜在病症——后者在老年人身上尤为常见。

很多有这类困扰的人，会通过专业按摩理疗来缓解。熟练的按摩师确实能带来显著效果，但所需的专业能力往往稀缺又昂贵。**这给机器人自动化的介入留出了机会**，具体说，就是用专门的机械臂来做按摩。这个想法激发了不少创新，[aescape](https://www.aescape.com/) 这样的初创公司就在开发商用按摩机器人按摩床，要提供稳定、可及的治疗，效率可与人类按摩师相当。

![图2：本项目表现最佳模型的结果](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/02.jpg)

尽管机器人与人工智能进展斐然，要开发一套真正有效的机器人按摩系统仍是不小的挑战。因为复刻按摩人体肌肉所需的精确动作和恰当力度、确保安全、规避潜在的失效点，都是复杂的任务。这里的工作聚焦于在仿真环境里设计并测试一套基础的机器人按摩系统。

为了说明我们的思路，设想一台装备了 3 自由度（3-DoF）臂和柔性末端执行器的机械臂，位于一个人体模型上方。系统起步的几步是：定位人体模型，到达一个指定起点，然后执行一段经标定的按摩动作。一个持续的反馈回路必不可少——既要主动防住潜在问题，也要适应模型的反应。

哪怕是这个简化的演示，也牵出好几个繁复的步骤。因此我们提出一个混合方案，把基于规则的路径跟随方法，跟在仿真环境里训练的 RL 方法结合起来。我们的框架用上了 [Pybullet](https://pybullet.org/wordpress/) 这类工具，并仔细定义了状态、动作和奖励的公式——这些是 RL 训练成功的关键组件。

为了严格评估这套设计，我们在多种环境配置下对几种 RL 方法做了基准对比。其中包括有 RL 与无 RL 的系统对比，以及缺少路径生成能力的系统。这些对比的结果会在我们的发现里详细讨论。

测试得出的结论是：这套结合方案——尤其在采用 Deep Deterministic Policy Gradient（DDPG）这一 RL 方法时——整体表现最佳，展现出更优的力度控制和更高的回合奖励。总的来说，本项目取得了以下几项关键成果：

-   提出一套训练机器人按摩臂的有效方法论。
-   对多种配置做基准对比，探索不同的状态/动作/奖励公式与 RL 方法论。
-   在多样的按摩路径和不同条件下（包括引入随机噪声）对训练好的模型做大量测试。

## 方法论

下面，我们拆解一套基于 PyBullet 的机器人按摩仿真的核心组件，它集成了一个细致的人体仿真模型、一台 UR5 机械臂、一套用于自然运动的路径生成，以及一个状态、动作、奖励都定义清楚的 RL 框架。

## 仿真环境

仿真环境建在 PyBullet 上，这是个强大的物理引擎，能做实时的刚体动力学和碰撞检测。环境初始化时载入一个平面、一个立方体，以及一台从 URDF 描述加载的 UR5 机械臂。重力和时间步设置模拟真实世界的物理，确保机器人与人体的交互逼真。一个人体仿真模型被载入并摆进场景，充当"病人"。环境管理机器人的受控关节和末端执行器，逐帧推进仿真，并收集详细的接触数据。这套搭建为训练和评估机器人按摩策略提供了一个丰富的平台。

仿真的核心是一个精巧的人体仿真模型，从项目 [gym-assistant](https://github.com/Healthcare-Robotics/assistive-gym) 导入，由代表肢体和关节的胶囊体与球体构成。模型支持男女两种变体，质量、尺寸、肤色可配，增强了真实感。

这套仿真里的机器人理疗师是一台 UR5 臂，一款广泛使用的工业机械手，以精度和灵活性著称。它从一个 URDF 文件加载，基于 [robot-description](https://github.com/robot-descriptions/robot_descriptions.py)。这台臂有六个受控关节，以及一个为执行按摩手法而设计的末端执行器。逆运动学算出关节角度，以沿生成路径到达目标位置；带力限制的位置控制确保动作安全、平滑。这套精确的控制框架让机器人能在贴近人体模型的近距离下，执行复杂、自适应的按摩动作。

![图3：完整的仿真场景](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/03.jpg)

## 混合控制（RL + 路径生成）

为了模仿人类按摩的流畅，系统在人体模型表面附近的点之间生成正弦轨迹。机器人的末端执行器周期性地跟随这些轨迹，并由 RL 模型做经标定的更新，以适应人体的位置和表面轮廓。这套方法产出连续、拟自然的运动模式，从而维持稳定的接触和力度——这对有效按摩至关重要。完整流程如下图所示。

![图4：按摩系统的完整流程](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/04.webp)

在 RL 这一步，需要定义一个全面的状态空间、动作空间和奖励函数。在我们的情形下，下面这套设置很高效：

![图5：我们 RL 训练步骤的状态/动作/奖励设置。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/05.webp)

-   状态：囊括接触点坐标、力的大小、法向距离和方向、末端执行器位姿与朝向，以及关节位置和速度。
-   动作：指定机器人末端执行器相对生成路径的**目标位置变化量**，用来调节轨迹的力度。
-   奖励：鼓励机器人施加恰当的接触力——对低于最大阈值（50 牛顿）的力给奖励，对过大的力或错误的接触点给惩罚。这套奖励结构把 agent 引向安全、有效的按摩行为。

这套基于 PyBullet 的机器人按摩仿真，把基于物理的建模、细致的人体解剖、精确的机器人控制和 RL 原理优雅地结合在一起。

在 RL 任务上，我们探索了三种应用于该仿真的前沿 RL 算法：

-   **DDPG（Deep Deterministic Policy Gradient）**
-   **PPO（Proximal Policy Optimization）**
-   **TD3（Twin Delayed Deep Deterministic Policy Gradient）**。

我们也点出了它们训练过程和实现细节里的关键之处。这些算法的实现基于 [CleanRL 代码库](https://github.com/vwxyzjn/cleanrl)。

每种算法各有独到的强项，它们的实现体现了训练、探索和稳定性方面的最佳实践。实践中，明智的做法是把它们全部基准对比一遍，挑出最适合自己用例的那个——这正是我们这里所做的。

## 评估与结果

下面我们展示本项目全部实验得到的两类曲线。第一类是 tensorboard 日志得出的训练曲线。第二类是力度曲线，刻画臂施加在模型上的法向力。

最后给出一张所有方法的完整性能表，用来确定表现最佳的模型。

## 训练曲线

DDPG-Y 模型的训练曲线，刻画的是机器人按摩系统在沿仿真模型 Y 轴跟随按摩路径这一情形下的学习进展。图的 Y 轴表示回合回报，衡量 agent 在每个训练回合中获得的累计奖励。X 轴表示训练迭代次数。下面这条曲线让人看清，在这项具体任务上，模型随时间提升性能的成效如何。

![图6：DDPG 沿模型高度方向按摩的训练曲线](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/06.webp)

下面这条训练曲线，对比 DDPG 和 PPO 算法在沿仿真环境 X 轴的按摩路径上训练时的表现，训练过程中不引入噪声。图凸显了两种方法在理想无噪声条件下的学习效率与稳定性。

![图7：DDPG 和 PPO 沿模型宽度方向按摩的训练曲线。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/07.webp)

下面这两条噪声条件下 DDPG 和 PPO 算法的训练曲线，展示了环境噪声对沿 X 轴按摩路径（身体宽度）学习速度的影响。图的 X 轴同样表示回合回报，但训练中存在噪声导致回报普遍偏低，说明模型在适应不确定、多变条件时面对的难度上升了。

![图8：DDPG 和 PPO 沿模型宽度方向、带随机初始化按摩的训练曲线。](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/08.webp)

> *总体而言，训练曲线表明* ***DDPG 取得了最快、最稳的学习进展****，在无噪声环境里尤为明显；而引入随机初始化对 DDPG 和 PPO 都构成显著挑战，压低了它们的回合回报，也拖慢了收敛。*

## 力度曲线

下面展示的这条跨 360 个仿真步的力度曲线，显示机器人末端执行器施加在人体模型上的接触力。在回合的大部分时间里，力都保持在 50 牛顿以下，处于按摩安全又舒适的范围内。临近结尾（超过 360 步范围时）出现一处明显的尖峰，说明施加力度有过短暂上升，提示这里还有改进空间，以避免不适或受伤。

此外，这里的图在力度曲线正下方还显示了：一张接触图（末端执行器索引 7 表示直接接触）、人体模型部位的接触索引，以及规划路径（蓝色）和修改后路径（橙色）的实际 Z 值。

![图9：DDPG 跨 720 步的力度与接触曲线（沿模型宽度、无噪声）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/09.webp)

DDPG-Y 模型跨 360 步的力度曲线显示，机械臂把接触力大体维持在 50 牛顿以下，偶有中断。这些波动暗示了机器人调节力度、或短暂失去接触的时刻，反映出沿 Y 轴按摩路径的动态特性。

![图10：DDPG 跨 360 步的力度与接触曲线（沿身体高度、无噪声）](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/10.webp)

下表给出各种 RL 方法在稳定与噪声两种环境条件下、100 个样本回合回报的均值和标准差。参与基准对比的方法有：无 RL（No-RL）、PPO、DDPG、沿 Y 轴训练的 DDPG（DDPG-Y），以及无路径引导的 TD3。

**结果表明，DDPG 和 DDPG-Y 在稳定环境里取得最高回报，而噪声条件普遍拉低了所有方法的性能。值得注意的是，DDPG 即便带噪声也维持了相对强的表现，体现出鲁棒性。TD3 在不以正弦波为基础时表现失败，均值回报只到 3300。这显示出我们上面提出的混合模型的高效。**

各按摩控制变体 100 个样本（回报）的均值和标准差

![表11：各方法回合回报对比](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/11.png)

最后，下面我们展示两段演示视频，分别是利用 DDPG 沿模型高度方向（第一段）、再沿模型宽度方向（第二段）的仿真场景，对应上面无噪声训练好的模型。

![图12：DDPG 模型沿人体模型高度方向（世界 Y 轴）做按摩](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/12.jpg)

![图13：DDPG 模型沿人体模型宽度方向（世界 X 轴）做按摩](https://cdn.jsdelivr.net/gh/genesislab-io/beatai-assets@5c58ae79cd94bcca8c6b9858f61f9652fa606578/ai-insights/2026-06/03/images/controlling-a-robotic-arm-for-massaging-human-body-with-rl/02.jpg)

## 结论

实验结果验证了我们这套方法的有效性，在与 **DDPG** 这类基于价值的方法搭配时尤其如此。不过，在噪声环境里训练时（人体模型初始位置随机，按摩路径的频率和幅度也随机），我们观察到学习有一段暂时的放缓，性能也略有下滑。为应对这点，我们认为 **课程学习**——训练中逐步引入噪声，从稳定条件起步、再渐进增加复杂度——大有潜力去加速学习过程。

尽管前景可期，我们当前的模型仍有*局限*。它们的训练没有接触过多变的环境条件，因此对模型或臂位置的变化有些敏感。此外，一项关键的安全特性——执行紧急规避动作以防对人体造成潜在伤害的能力——在本工作中仍未实现。一个直接的解法是：一旦施加的力超过设定阈值（如 50 牛顿），就触发一个预先编程好的规避动作。

放眼未来，几条改进方向已经清晰可见。

-   探索柔性球以外的其他末端执行器，会拓宽可行按摩手法的范围。
-   比如一只多自由度的拟人手，就能提供更具适应性的方案。
-   再者，采用双臂构型、对标 Aesape 按摩床这类系统，是未来研究一个很有吸引力的方向。
-   最后，根据个人偏好对最大施加力做个性化，必不可少。这要么需要开发一个更定制化的模型，要么需要为不同力度等级训练多个 RL 模型。

来联系吧：[Medium](https://medium.com/@mryasinusif)| [Linkedin](https://www.linkedin.com/in/engyasinyousif/?originalSubdomain=de)| [Twitter](https://x.com/YasinYousif001) | [联系表单](https://www.rlbyexample.net/pages/contact-form/index.html)

*除非另有说明，所有图片均由作者生成*
