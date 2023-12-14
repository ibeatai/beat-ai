# 明星项目
> 滚滚长江东逝水，浪花淘尽英雄，是非成败转头空 - 临江仙·滚滚长江东逝水

经过大浪淘沙留下来的才是真金白银，对于开源项目也是如此。对于明星项目，本文不仅仅以`star`数的多少作为评判维度，还会结合项目规模、影响力、活跃度、社区活跃度等多个方面进行评定，希望大家能喜欢。

需要注意，本文列出的几乎都是平台级项目，因此并不是star多，就能名列其中，例如很多`star`很多的工具、Rust库、书籍都没有列入，如果大家想要看更多的子类项目，请访问对应的文件进行查看。

## deno
首先出场的自然是咖位最重的之一，可以说正是因为`deno`和`swc`的横空出世，才让一堆观望的大神对于Rust实现`Javascript`基建有了更强的信心。

[`deno`](https://github.com/topics/rust?l=rust&o=desc&s=stars)是`node`半逆转后的字序，从此可以看出`deno`是`Node.js`的替代，它的目标是为`Typescript/Javascript`提供一个更现代化、更安全、更强大    的运行时，同时内置了很多强大的工具，可以用于打包、编译成可执行文件、文档、测试、lint等。

## alacritty
[alacritty](https://github.com/alacritty/alacritty)是一个跨平台、基于OpenGL的终端，性能极高的同时还支持丰富的自定义和可扩展性，可以说是非常优秀的现代化终端。

目前已经是`beta`阶段，可以作为日常工具来使用。

<img alt="alacritty screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/alacritty.png?raw=true" class="center"  />

## starship
[starship](https://github.com/starship/starship)是一个命令行提示，支持任何`shell`，包括`zsh`，简单易用、非常快且拥有极高的可配置性。

<img alt="starship screenshot" width="100%"  src="https://raw.githubusercontent.com/starship/starship/master/media/demo.gif" class="center"  />

## MeiliSearch 
[MeiliSearch](https://github.com/meilisearch/MeiliSearch)是一个搜索平台，但是跟`ElasticSearch`不同，`MeiliSearch`并不是通用目的的，它的目标是为终端用户提供边输入边提示的即刻搜索功能，因此是一个轻量级搜索平台，不适用于数据量大时的搜索目的。

总之，如果你需要在网页端或者APP为用户提供一个搜索条，然后支持输入容错、前缀搜索时，就可以使用它。

<img alt="meilisearch screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/meilisearch.gif?raw=true" class="center"  />

## swc 🌟19.5k
[`swc`](https://github.com/swc-project/swc)是`Typescript/Javascript`编译器，它可以用来编译、压缩和打包JS，同时支持使用插件进行扩展，例如做代码变换等。

`swc`目前正在被一些知名项目所使用，包括`Next.js`,`Parcel`和`Deno`，还有些著名的公司也在使用它，例如`Vercel`、字节跳动、腾讯等。

它的性能非常非常高，官方号称，在单线程下比`Babel`快20倍，在4核心下比`Babel`快70倍！

几个使用案例：

- [nextjs 12](http://nextjs.org/12), 通过使用`swc`获得了更好的扩展性、性能以及wasm的支持，其中性能方面提升了3倍刷新速度、5倍打包速度
- [Parcel](https://parceljs.org/)，通过使用`swc`改善了10倍的性能

<img alt="parcel screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/parcel.png?raw=true" class="center"  />

## tauri
[tauri](https://tauri.studio)可以用来更小、更快、更安全的桌面应用，它想要替代的是`electron.js`。

下面是援引自[官网](https://tauri.studio/benchmarks)的性能对比图:

<img alt="tauri1 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/tauri1.png?raw=true" class="center"  />

<img alt="tauri2 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/tauri2.png?raw=true" class="center"  />

## yew
[`yew`](https://github.com/yewstack/yew)是一个正在活跃开发的`Rust/Wasm`框架，用于构建`Web`应用。

<img alt="yew screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/yew.jpg?raw=true" class="center"  />


## firecracker 
[`firecracker`](https://github.com/firecracker-microvm/firecracker)是一个安全、高性能的无服务计算虚拟机(FaaS)，支持多租户、资源隔离等高级特性，由Amazon公司开发，为AWS部分云计算服务提供了强力有的支持。BTW，亚马逊Amazon公司对于Rust语言的喜爱是众所周知的，几乎已经成了Rust的形象大使之一了:）

## nushell
[`nushell`](https://github.com/nushell/nushell)是一个全新的`shell`，使用`Rust`实现。它的目标是创建一个现代化的`shell`：虽然依然基于`Unix`的哲学，但是更适合现在的时代。例如，你可以使用`SQL`语法来选择你想要的内容！

<img alt="delta screenshot" width="100%"  src="https://github.com/nushell/nushell/raw/main/images/nushell-autocomplete5.gif" class="center"  />


## tokio
[`tokio`](https://github.com/tokio-rs/tokio)的名声可以说是如雷贯耳，如果学过Rust但是没有听说过它，那我觉得可能要回炉重造下:)

`tokio`是一个异步IO的运行时，提供了`I/O`、网络、调度、定时器等等异步编程所必须的功能和工具，性能和功能都异常强大。

## AppFlowy
[AppFlowy](https://github.com/AppFlowy-IO/appflowy)是[`Notion`](https://www.notion.so/zh-cn)的开源实现，使用`Rust`和`Flutter`进行开发，用于用户文档和数据的管理，支持丰富的自定义特性。

<img alt="appflowy screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/appflowy.png?raw=true" class="center"  />

## Bevy
[bevy](https://github.com/bevyengine/bevy)是一个数据驱动的游戏引擎，支持2D和3D图形开发，优点是社区活跃、更新快、模块化设计优秀、性能高，缺点是还处于快速开发中，并不适合生产使用。

同时`bevy`的文档齐全，官方示例很多，非常适合学习和使用。

<img alt="bevy screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/gamedev/bevy.jpg?raw=true" class="center"  />

## actix-web
[actix-web]()是全世界最快的web框架之一，甚至可以把之一去掉，因为排在它前面的看上去像是一个专为跑分而生的轻量级框架，而`actix-web`可是功能相当多的！

下面给出`actix`和Go语言`Gin`框架的性能对比：

<img alt="actix screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/actix.png?raw=true" class="center"  />

## iced
[`iced`](https://github.com/iced-rs/iced)是一个跨平台GUI库，具有简单易用、模块化设计、响应式布局等优点。

<img alt="iced screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/iced.gif?raw=true" class="center"  />

## cube.js
[`cube.js`](https://github.com/cube-js/cube.js)是一个数据分析API平台，可以用于构建内部的BI或为现有的应用增加客户数据统计等功能，使用`Rust`和`Typescript`构建。

<img alt="cubejs screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/cubejs.png?raw=true" class="center"  />

## wasmer
[wasmer](https://github.com/wasmerio/wasmer)是业界领先的`WASM`运行时，支持`WASI`和`Emscripten`。

```shell
$ wasmer qjs.wasm
QuickJS - Type "\h" for help
qjs > const i = 1 + 2;
qjs > console.log("hello " + i);
hello 3
```

## tikv
[`tikv`](https://github.com/tikv/tikv)相信大家都已知道，`tidb`的底层存储服务，国人之光项目，在数据之外，还做了大量的技术知识普及工作，值得敬佩！

`tikv`是分布式`KV`数据库，支持分布式事务。

<img alt="tikv screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/tikv.png?raw=true" class="center"  />

## ruffle
[`ruffle`](https://github.com/ruffle-rs/ruffle)是用Rust写的`Flash Player`模拟器，同时支持桌面端和Web端，其中后者通过WASM提供支持。

## rustdesk
[`rustdesk`](https://github.com/rustdesk/rustdesk)是国内团队开发的一款远程桌面软件。

<img alt="rustdesk screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/rustdesk.jpg?raw=true" class="center"  />

## RustPython
[`RustPython`]是使用`Rust`实现的`Python`解释器, 支持`Python3`（CPython >= 3.9.0）。

大家可以通过官方提供的[在线网址](https://rustpython.github.io/demo/)进行尝试。

<img alt="rustpython screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/rustpython.jpg?raw=true" class="center"  />

## vector
[`vector`](https://github.com/vectordotdev/vector)是一个性能很高的数据采集agent，采集本地的日志、监控等数据，发送到远程的kafka、jaeger等数据下沉端，它最大的优点就是能从多种数据源(包括Opentelemetry)收集数据，然后推送到多个数据处理或者存储等下沉端。

<img alt="vector screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/vector.jpg?raw=true" class="center"  />

## mdbook
[`mdbbok`](https://github.com/rust-lang/mdBook)可以基于`markdown`文件自动创建在线电子书，非常简单好用，目前的问题就是缺乏章节内部的目录跳转和中文搜索。

<img alt="mdbook screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/mdbook.jpg?raw=true" class="center"  />

## zola
[`zola`](https://github.com/getzola/zola)是一个静态网站生成器，类似`hugo`。

<img alt="zola screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/zola.jpg?raw=true" class="center"  />


## gitui
[`gitui`](https://github.com/extrawurst/gitui)是一个奇快无比的Git终端UI，无需浏览器即可使用。

<img alt="gitui screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/gitui.gif?raw=true" class="center"  />

## solana
[`solana`](https://github.com/solana-labs/solana)是知名的区块链平台，快速、安全、去中心化，还自带应用市场。

## ripgrep
[`ripgrep`](https://github.com/BurntSushi/ripgrep)是一个性能极高的现代化`grep`实现，后者是`Unix/Linux`下的内置文件搜索工具。该项目是Rust的明星项目，一个是因为性能极其的高，另一个就是源代码质量很高，值得学习, 同时`Vscode`使用它作为内置的搜索引擎。

从功能来说，除了全面支持`grep`的功能外，`repgre`支持使用正则递归搜索指定的文件目录，默认使用`.gitignore`对指定的文件进行忽略。

<img alt="ripgrep screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/ripgrep.png?raw=true" class="center"  />

## citybound
[`citybound`](https://github.com/citybound/citybound)是一个多人在线模拟游戏，使用Rust + WASM + JS开发。

<img alt="citybound screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/game/citybound.png?raw=true" class="center"  />


## bottlerocket
[`bottlerocket`](https://github.com/bottlerocket-os/bottlerocket)是一个基于`Linux`的操作系统，它的目标是为容器提供宿主环境。

## lemmy
[`lemmy`](https://github.com/LemmyNet/lemmy)是一个`reddit`克隆，可以通过连接聚合的方式来构建社区，支持桌面和移动端。

<img alt="lemmy screenshot" width="100%" src="/superstar/lemmy.jpg?raw=true" class="center"  />


## tantivy
[`tantivy`](https://github.com/quickwit-inc/tantivy)是Rust实现的本地搜索库，功能对标`lucene`，如果你不需要分布式，那么引入tantivy作为自己本地Rust服务的一个搜索，是相当不错的选择，该库作者一直很活跃，而且最近还创立了搜索引擎公司，感觉大有作为. 该库的优点在于纯Rust实现，性能高(lucene的2-3倍)，资源占用低(对比java自然不是一个数量级)，社区活跃。

## sled
[`sled`](https://github.com/spacejam/sled)是本地嵌入式的数据库。

```rust
let tree = sled::open("/tmp/welcome-to-sled")?;

// insert and get, similar to std's BTreeMap
let old_value = tree.insert("key", "value")?;

assert_eq!(
  tree.get(&"key")?,
  Some(sled::IVec::from("value")),
);

// range queries
for kv_result in tree.range("key_1".."key_9") {}

// deletion
let old_value = tree.remove(&"key")?;

// atomic compare and swap
tree.compare_and_swap(
  "key",
  Some("current_value"),
  Some("new_value"),
)?;

// block until all operations are stable on disk
// (flush_async also available to get a Future)
tree.flush()?;
```

### redox
[`Redox`](https://github.com/redox-os/redox)是一个`Unix`风格的微内核操作系统，使用`Rust`实现。`redox`的目标是安全、快速、免费、可用，它在内核设计上借鉴了很多优秀的内核，例如：`SeL4`, `MINIX`, `Plan 9`和`BSD`。

但`redox`不仅仅是一个内核，它还是一个功能齐全的操作系统，提供了操作系统该有的功能，例如：内存分配器、文件系统、显示管理、核心工具等等。你可以大概认为它是一个`GNU`或`BSD`生态，但是是通过一门现代化、内存安全的语言实现的。

> 不过据我仔细观察，redox目前的开发进度不是很活跃，不知道发生了什么，未来若有新的发现会在这里进行更新 - Sunface

<img alt="redox1 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/os/redox1.jpg?raw=true" class="center"  />

<img alt="redox2 screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/os/redox2.jpeg?raw=true" class="center"  />

### youki
[`youki`](https://github.com/containers/youki)是一个容器运行时，实现了`OCI`标准，性能非常好的同时具备非常高的安全性, 目前来说，它的性能跟`crun`差不多，比`runc`快50%以上。


### slint
[`slint`](https://github.com/slint-ui/slint)是一个GUI工具集，原名sixtyfps, 同时适用于嵌入式系统、桌面系统、移动端、浏览器(WASM)，支持使用多种语言进行开发，背后有商业公司的支持，未来前景看好。

slint已于2023年4月发布1.0版本，标志着结束开发模式并已准备好在生产环境中使用。

<img alt="sixtyfps screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/sixtyfps.png?raw=true" class="center"  />

### wasmtime
[wasmtime](https://github.com/bytecodealliance/wasmtime)是一个为`WASM`设计的`JIT`风格的独立运行时，支持`WASI`。

```rust
fn main() {
    println!("Hello, world!");
}
```

```shell
$ rustup target add wasm32-wasi
$ rustc hello.rs --target wasm32-wasi
$ wasmtime hello.wasm
Hello, w
```

### polkadot
[`polkadot`](https://github.com/paritytech/polkadot)是知名的区块链平台，它是从[`Substrate`](https://github.com/paritytech/substrate)抽离出来，后者是下一代区块链开发框架。

### lapce
[`lapce`](https://github.com/lapce/lapce)是一款性能极高、功能强大、基于`wgpu`渲染的代码编辑器，基于`Xi-Editor`开发，后者`Xi-Editor`曾经也红极一时，可惜不再维护了，但是依然非常适合做一个编辑器内核。

<img alt="lapce screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/lapce.jpg?raw=true" class="center"  />

### rust-gpu
[rust-gpu](https://github.com/EmbarkStudios/rust-gpu)的目标是让Rust成为GPU编程的第一梯队语言，由大名鼎鼎的`Embark`公司开发，后台较硬。

如果需要通用的`GPU`编程，选它就对了。
