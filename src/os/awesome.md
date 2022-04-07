# 操作系统

操作系统范畴很大，本章节中精选的内容聚焦在用Rust实现的操作系统以及用Rust写操作系统的教程。

## 目录
| 系统 | 描述 |
| -------- | --------- |
| [redox](#redox) | `Unix`风格的微内核OS |
| [tock](#tock) | 嵌入式操作系统 |
| [theseus](#theseus) | 独特设计的OS | 
| [writing os in rust](#writing-an-os-in-rust) | 使用Rust开发简单的操作系统 |
| [rust-raspberrypi-OS-tutorials](#rust-raspberrypi-os-tutorials) |  Rust嵌入式系统开发教程 |
| [rcore-os](#rcore-os) | 清华大学提供的`rcore`操作系统教程 | 
| [edu-os](#edu-os) | 亚琛工业大学操作系统课程的配套项目 |

### redox
[redox](https://github.com/redox-os/redox) 是一个 `Unix` 风格的微内核操作系统，使用 `Rust` 实现。`redox` 的目标是安全、快速、免费、可用，它在内核设计上借鉴了很多优秀的内核，例如：`SeL4`, `MINIX`, `Plan 9`和`BSD`。

但 `redox` 不仅仅是一个内核，它还是一个功能齐全的操作系统，提供了操作系统该有的功能，例如：内存分配器、文件系统、显示管理、核心工具等等。你可以大概认为它是一个 `GNU` 或 `BSD` 生态，但是是通过一门现代化、内存安全的语言实现的。

> 不过据我仔细观察，redox目前的开发进度不是很活跃，不知道发生了什么，未来若有新的发现会在这里进行更新 - Sunface

<img alt="redox1 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/os/redox1.jpg?raw=true" class="center"  />

<img alt="redox2 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/os/redox2.jpeg?raw=true" class="center"  />

### tock
[tock](https://github.com/tock/tock) 是一个嵌入式操作系统，设计用于在低内存和低功耗的微控制器上运行多个并发的、相互不信任的应用程序，例如它可在 `Cortex-M` 和 `RISC-V` 平台上运行。

`Tock` 使用两个核心机制保护操作系统中不同组件的安全运行：

- 内核和设备驱动全部使用Rust编写，提供了很好安全性的同时，还将内核和设备进行了隔离
- 使用了内存保护单元技术，让应用之间、应用和内核之间实现了安全隔离

具体可通过这本书了解: [The Tock Book](https://book.tockos.org/introduction.html).

<img alt="tock screenshot" width="100%"  src="https://book.tockos.org/imgs/imix.svg" class="center"  />

### Theseus
[Theseus](https://github.com/theseus-os/Theseus) 是从零开始构建的操作系统，完全使用Rust进行开发。它使用了新的操作系统结构、更好的状态管理，以及利用语言内设计原则将操作系统的职责(如资源管理)转移到编译器中。

该OS目前尚处于早期阶段，但是看上去作者很有信心未来可以落地，如果想要了解，可以通过官方提供的[在线书籍](https://theseus-os.github.io/Theseus/book/index.html)进行学习。

### Writing an OS in Rust
[Writing an OS in Rust](https://os.phil-opp.com) 是非常有名的博客系列，专门讲解如何使用Rust来写一个简单的操作系统，配套源码在[这里](https://github.com/phil-opp/blog_os)，目前已经发布了第二版。

以下是`async/await`的目录截图：
<img alt="writing-os screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/os/writing-os.jpg?raw=true" class="center"  />

### rust-raspberrypi-OS-tutorials
[rust-raspberrypi-OS-tutorials](https://github.com/rust-embedded/rust-raspberrypi-OS-tutorials) 教大家如何用Rust开发一个嵌入式操作系统，可以运行在树莓派上。这个教程讲得很细，号称手把手教学，而且是从零实现，因此很值得学习。

<img alt="rrot screenshot" width="50%" height="400px" src="https://github.com/studyrs/cookbook-images/blob/main/os/rrot.jpg?raw=true" class="center"  />
<img alt="rrot1 screenshot" width="49%" height="400px" src="https://github.com/studyrs/cookbook-images/blob/main/os/rrot1.gif?raw=true" class="center"  />

### rcore-os
[rcore-os](https://github.com/rcore-os) 是由清华大学开发的操作系统，用 Rus t实现, 与 `linux` 相兼容，主要目的目前还是用于教学，因为还有相关的配套教程，非常值得学习。目前支持的功能不完全列表如下：`linux` 兼容的 `syscall` 接口、网络协议栈、简单的文件系统、信号系统、异步IO、内核模块化。

- [内核实现](https://github.com/rcore-os/rCore)
- [配套教程](https://github.com/rcore-os/rCore-Tutorial-Book-v3) 


以下是在树莓派上运行的图：
<img alt="rcore screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/os/rcore.jpg?raw=true" class="center"  />



### edu-os
[edu-os](https://github.com/RWTH-OS/eduOS-rs) 是 `Unix` 风格的操作系统，用于教学目的，它是亚琛工业大学（RWTH Aachen University)操作系统课程的配套大项目，但是我并没有找到对应的课程资料，根据作者的描述，上面部分的**Writing an OS in Rust**对他有很大的启发。

<img alt="eduos screenshot" width="100%"  src="https://github.com/RWTH-OS/eduOS-rs/raw/master/img/demo.gif?raw=true" class="center"  />