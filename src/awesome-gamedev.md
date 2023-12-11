# 游戏开发
我在这里大胆预言：Rust未来会成为和`C++`同级别的游戏开发语言，特别是在游戏引擎方面，会大放异彩。


## 目录索引

- [**游戏引擎**](#游戏引擎)： [bevy](#bevy)， [fyrox](#fyrox前rg3d)， [ggez](#ggez)， [oxygengine](#oxygengine)， [macroquad](#macroquad)， [godot-rust](#godot-rust)， [piston](#piston)， [amethyst](#amethyst)
- [**GPU和图形渲染**](#gpu和图形渲染)： [wgpu](#wgpu)， [rust-gpu](#rust-gpu)，[kajiya](#kajiya)， [lyon](#lyon)， [ash](#ash)， [vulkano](#vulkano)， [rend3](#rend3)， [rafx](#rafx)， [gfx](#gfx)， [luminance](#luminance)， [miniquad](#miniquad)， [glow](#glow)
- [**学习资料和新闻**](#学习资料)

## 游戏引擎
### Bevy
[bevy](https://github.com/bevyengine/bevy)是一个数据驱动的游戏引擎，支持2D和3D图形开发，优点是社区活跃、更新快、模块化设计优秀、性能高，缺点是还处于快速开发中，并不适合生产使用。

同时`bevy`的文档齐全，官方示例很多，非常适合学习和使用。

<img alt="fyrox screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/bevy.jpg?raw=true" class="center"  />

### Fyrox(前rg3d)
[fyrox](https://github.com/FyroxEngine/Fyrox)是一个`2D`和`3D`游戏图形化引擎，功能丰富，生产可用(官方宣称)。

该项目前身是`rg3d`，但是被收购后，更名为`fyrox`，潜力应该是相当好的，下面截图来源于基于该引擎开发的游戏[`StationIapetus`](https://github.com/mrDIMAS/StationIapetus)。

<img alt="fyrox screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/fyrox.jpg?raw=true" class="center"  />

### ggez
[ggez](https://github.com/ggez/ggez)是一个轻量级的`2D`游戏图形引擎，它的目标是让游戏开发尽量的简单，因此它的功能并不是很强大，例如如果你想要强大且真实的物理引擎，它可能无能为力，但你可以选择在它的基础上构建自己的更高级的引擎。

<img alt="ggez screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/ggez.png?raw=true" class="center"  />

### oxygengine
[oxygengine](https://github.com/PsichiX/oxygengine)是一个`2D` HTML5游戏引擎，支持编译成WASM在浏览器中运行。

<img alt="oxygengine screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/oxygengine.gif?raw=true" class="center"  />

### macroquad
[macroquad](https://github.com/not-fl3/macroquad)是一个`2D`游戏引擎，特点是简单易用，例如它试图让使用者不会遇到Rust生命周期的难题。

### godot-rust
[godot-rust](https://github.com/godot-rust/godot-rust)是大名鼎鼎的`godot`引擎的`Rust`绑定，[`godot`](https://github.com/godotengine/godot)是`c++`开发的游戏`2D/3D`引擎，但是对Rust语言提供了很好的支持。

<img alt="godot screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/godot.jpg?raw=true" class="center"  />


### piston
[piston](https://github.com/PistonDevelopers/piston)是前两年较火的模块化的游戏引擎，但是最近半年开发速度缓慢，我调查了一番，但不清楚发生了什么。

### Amethyst
[Amethyst](https://github.com/amethyst/amethyst), 前几年较火的Rust游戏引擎，但是最近开发已经停滞，经过我调查，是因为作者团队转型Rust游戏开发知识分享，因此项目被[放弃](https://amethyst.rs/posts/amethyst--starting-fresh)。

## GPU和图形渲染
### wgpu
[wgpu](https://github.com/gfx-rs/wgpu)是一个纯Rust实现的图形化API库，具有安全、可移植等优点，如果你使用基于`wgpu`构建的库，那该库可以很多平台上运行：Linux, windows, MacOS, Android和IOS。

它可以原生的运行在`Vulkan`, `Metal`等主流平台上，且可以使用`wasm`的方式运行在`WebGPU`上，同时API兼容`WebGPU`标准。

总之，如果你要使用`WebGPU`, 选它就对了。

### rust-gpu
[rust-gpu](https://github.com/EmbarkStudios/rust-gpu)的目标是让Rust成为GPU编程的第一梯队语言，由大名鼎鼎的`Embark`公司开发，后台较硬。

如果需要通用的`GPU`编程，选它就对了。

### kajiya
[kajiya](https://github.com/EmbarkStudios/kajiya)是一个实时的、全局光照渲染系统，由`Embark`公司开发，该公司在秘密研究基于Rust的游戏引擎，据说准备应用在新游戏上，有朝一日它可能会是推动Rust游戏引擎爆发式发展的功臣。

`kajiya`应用了非常先进的论文和设计理念，因此非常值得有志于游戏引擎开发的同学学习。但目前还不适用于生产级使用，具体见[这里](https://medium.com/embarkstudios/homegrown-rendering-with-rust-1e39068e56a7)。

<img alt="kajiya screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/kajiya.jpg?raw=true" class="center"  />

### lyon
[lyon](https://github.com/nical/lyon)可以使用GPU进行向量路径渲染，例如高效渲染复杂的`svg`等。

### ash
[ash](https://github.com/MaikKlein/ash)是一个轻量级的`Vulkan`绑定。

<img alt="ash screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/ash.png?raw=true" class="center"  />

### vulkano
[vulkano](https://github.com/vulkano-rs/vulkano)是一个安全、特性丰富的`Vulkan`绑定。

### rend3
[rend3](https://github.com/BVE-Reborn/rend3)是一个简单易用、可定制性强、高效的3D渲染库，基于`wgpu`开发。

<img alt="rend3 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/rend3.jpg?raw=true" class="center"  />

### rafx
[rafx](https://github.com/aclysma/rafx)是一个多后端渲染器，目标是性能、扩展性和生产力。

<img alt="rafx screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/rafx.jpg?raw=true" class="center"  />

### gfx
[gfx](https://github.com/gfx-rs/gfx)是一个底层的图形库，目前已经不怎么活跃，主要原因是：它的核心组件`gfx-hal`最开始的目标是为`wgpu`提供功能，但是后面`wgpu`实现了自己的`wgpu-hal`，因此`gfx-hal`目前仅处于维护状态。

### luminance
[luminance](https://github.com/phaazon/luminance-rs)是一个类型安全、无状态的图形框架，目标是让图形渲染变得简单和优雅，最开始是通过`Haskell`语言实现，然后在`2016`年移植到`Rust`上。

它很简单，功能也不够强大，如果你没有`OpenGL`、`Vulkan`的经验，可以使用它做一些简单的图形渲染项目试试。

### miniquad
[miniquad](https://github.com/not-fl3/miniquad)是一个安全和跨平台的图形渲染库，它提供了较为底层的API，如果需要抽象层次更高的API，可以使用之前提到的[macroquad](https://github.com/not-fl3/macroquad),后者是基于`miniquad`封装实现。

<img alt="miniquad screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/miniquad.gif?raw=true" class="center"  />



### glow
[glow](https://github.com/grovesNL/glow)提供了各种`GL`绑定(OpenGL, WebGL), 提供了一定的抽象，避免你写平台相关的特定代码实现。


## 学习资料
### 游戏开发最新新闻

- [gamedev](https://gamedev.rs)

### 一些学习资料(英文)

- [Hands-on Rust](https://pragprog.com/titles/hwrust/hands-on-rust/)
- 使用[bracket-lib](https://github.com/amethyst/bracket-lib)和其[配套书籍](https://bfnightly.bracketproductions.com/rustbook/)进行学习
- 想要没有困难的开发一个跨平台的2D游戏？使用[`macroquad`](https://macroquad.rs)，并且可以参考用它开发的两个游戏: [fish fight](https://github.com/fishfight/FishFight)和[zemeroth](https://github.com/ozkriff/zemeroth)
- 想要开发一个简单的3D游戏并且需要一个编辑器？可以试试[`fyrox(rg3d)`](https://github.com/rg3dengine/rg3d)
- 想要开发一个复杂的游戏或者想要做一个demo，未来可以基于该demo继续开发，最终完成一个复杂游戏？可以试试`godot`引擎提供的`Rust`绑定：[godot-rust](https://godot-rust.github.io)
- 喜欢钻研前沿技术？试试[`bevy`](https://bevyengine.org)，它拥有最好的`ECS`实现和最先进的设计理念(可能)

### ECS(Entity Component System)和DOD(面向数据设计)资料
我们在上面提到的很多系统都使用了`ECS`和`DOD`，因此这两者对于游戏开发是极其重要的，下面是一些相关的英文资料(部分需要翻墙)，可以帮助大家理解相关概念。

- [hecs](https://github.com/Ralith/hecs), 一个用Rust实现的ECS世界
- [Understanding data-oriented design for entity component systems - Unity at GDC 2019](https://www.youtube.com/watch?v=0_Byw9UMn9g)
- [CppCon 2018: Stoyan Nikolov “OOP Is Dead, Long Live Data-oriented Design”](https://www.youtube.com/watch?v=yy8jQgmhbAU)
- [RustConf 2018 - Closing Keynote - Using Rust For Game Development by Catherine West](https://www.youtube.com/watch?v=aKLntZcp27M)
- ["Data-Oriented Design" web book by Richard Fabian](https://dataorienteddesign.com/dodbook/)


### 一些游戏开发的生产力工具

- [Blender](https://www.blender.org)用于3D建模
- [Krita](https://krita.org/en)用于创建2D图片




