# Fancy Rust
在设计上，我们并没有参考其它awesome-XXX常用的方式：在首页放非常非常长的列表，什么都囊括在内。而是选择了一个**用户友好的方式**来呈现内容:
- 主页Readme.md放置日常开发常用的Rust库、Rust精品学习资料、优秀开源库精选推荐, 其中**Rust库只推荐最优秀且持续更新的**，那些几年不更新的统统剔除掉，少数精品的呈现才不会让用户过于纠结，甚至不知道如何选择
- 其它各种库，统统使用单独的.md文件存放，例如**图像处理.md**，**游戏开发.md**等
- 绝大部分子项中的库按照推荐度优先级排序，越靠前，推荐度越高

## 目录
- 日常开发常用的Rust库: 
  - [Web/HTTP](#webhttp),  [SQL客户端](#SQL客户端), [NoSql客户端](#NoSql客户端)， [网络通信协议](#网络通信协议), [异步网络编程](#异步网络编程)
  - [服务发现](#服务发现), [消息队列](#消息队列), [搜索平台](#搜索平台)
  - [编解码](#编解码), [Email](#Email), [常用正则模版](#常用正则模版)
  - [日志监控](#日志监控), [代码Debug](#代码Debug), [性能优化](#性能优化)
- [精选中文学习资料](#精选中文学习资料)
- [精选Rust开源项目](#精选Rust开源项目)

## 日常开发常用Rust库
### Web/HTTP
* HTTP客户端
  * [reqwest](https://github.com/seanmonstar/reqwest)  一个简单又强大的HTTP客户端，`reqwest`是目前使用最多的HTTP库 

* Web框架
  * [axum](https://github.com/tokio-rs/axum)  基于Tokio和Hyper打造，模块化设计较好，目前口碑很好，值得使用Ergonomic and modular web framework built with Tokio, Tower, and Hyper
  * [Rocket](https://github.com/SergioBenitez/Rocket)  功能强大，API简单的Web框架，但是主要开发者目前因为个人原因无法进行后续开发，未来存在不确定性
  * [actix-web](https://github.com/actix/actix-web)  性能极高的Web框架，就是团队内部有些问题，未来存在一定的不确定性
  * 总体来说，上述三个web框架都有很深的用户基础，其实都可以选用，如果让我推荐，顺序如下: `axum` > `Rocket` > `actix-web`。 不过如果你不需要多么完善的web功能，只需要一个性能极高的http库，那么`actix-web`是非常好的选择，它的性能非常非常非常高！

### 日志监控
* 日志
[[crates.io](https://crates.io/keywords/log)] [[github](https://github.com/search?q=rust+log)]
  * [tokio-rs/tracing](https://github.com/tokio-rs/tracing)  强大的日志框架，同时还支持OpenTelemetry格式，无缝打通未来的监控
  * [rust-lang/log](https://github.com/rust-lang/log)  官方日志库，事实上的API标准, 但是三方库未必遵循
  * [estk/log4rs](https://github.com/estk/log4rs)  模仿JAVA `logback`和`log4j`实现的日志库, 可配置性较强
  * 在其它文章中，也许会推荐slog，但是我们不推荐，一个是因为近半年未更新，一个是`slog`自己也推荐使用`tracing`。
* 监控
  * [OpenTelemetry](https://github.com/open-telemetry/opentelemetry-rust)  `OpenTelemetry`是现在非常火的可观测性解决方案，提供了协议、API、SDK等核心工具，用于收集监控数据，最后将这些metrics/logs/traces数据写入到`prometheus`, `jaeger`等监控平台中。最主要是，它后台很硬，后面有各大公司作为背书，未来非常看好！
  * [vectordotdev/vector](https://github.com/vectordotdev/vector)  一个性能很高的数据采集agent，采集本地的日志、监控等数据，发送到远程的kafka、jaeger等数据下沉端，它最大的优点就是能从多种数据源(包括Opentelemetry)收集数据，然后推送到多个数据处理或者存储等下沉端。

### SQL客户端
* 通用
  * [launchbadge/sqlx](https://github.com/launchbadge/sqlx) 异步实现、高性能、纯Rust代码的SQL库，支持`PostgreSQL`, `MySQL`, `SQLite`,和 `MSSQL`.

* ORM
  * [rbatis/rbatis](https://github.com/rbatis/rbatis)  国内团队开发的ORM，异步、性能高、简单易上手
  * [diesel-rs/diesel](https://github.com/diesel-rs/diesel)  安全、扩展性强的Rust ORM库，支持`Mysql`、`Postgre`、`SqlLite`


* Mysql
  * [blackbeam/rust-mysql-simple](https://github.com/blackbeam/rust-mysql-simple)  纯Rust实现的Mysql驱动,提供连接池
  * [blackbeam/mysql_async](https://github.com/blackbeam/mysql_async)  基于Tokio实现的异步Mysql驱动
  * 上面两个都是一个团队出品，前者文档更全、star更多，建议使用前者


* Postgre
  * [sfackler/rust-postgres](https://github.com/sfackler/rust-postgres) 纯Rust实现的Postgre客户端

* Sqlite
  * [rusqlite](https://github.com/rusqlite/rusqlite) 用于[Sqlite3](https://www.sqlite.org/index.html)的Rust客户端

### NoSql客户端

* Redis
  * [mitsuhiko/redis-rs](https://github.com/mitsuhiko/redis-rs) 虽然最近更新不太活跃，但是它依然是最好的redis客户端，说实话，我期待更好的，可能这也是Rust生态的未来可期之处吧

* Canssandra
  * [krojew/cdrs-tokio](https://github.com/krojew/cdrs-tokio) [[cdrs-tokio](https://crates.io/crates/cdrs-tokio)] 生产可用的Cassandra客户端，异步、纯Rust实现，就是个人项目 + star较少，未来不确定会不会不维护
  * [scylla-rust-driver](https://github.com/scylladb/scylla-rust-driver)  ScyllaDB提供的官方库，支持cql协议，由于背靠大山，未来非常可期


* MongoDB
  * [mongodb/mongo-rust-driver](https://github.com/mongodb/mongo-rust-driver) 官方MongoDB客户端，闭着眼睛选就对了

### 分布式
#### 服务发现
- [luncj/etcd-rs](https://github.com/luncj/etcd-rs) 异步实现的Rust etcd客户端，优点是有一定的文档、作者较为活跃,意味着你提问题他可能会回答，不过，如果你不放心，还是考虑使用HTTP的方式访问ETCD

#### 消息队列
* Kafka
  * [fede1024/rust-rdkafka](https://github.com/fede1024/rust-rdkafka)  Rust Kafka客户端，基于C版本的Kafka库[librdkafka]实现，文档较全、功能较为全面
  * [kafka-rust/kafka-rust](https://github.com/kafka-rust/kafka-rust)  相比上一个库，它算是纯Rust实现，文档还行，支持Kafka0.8.2及以后的版本，但是对于部分0.9版本的特性还不支持。同时有一个问题：最初的作者不维护了，转给了现在的作者，但是感觉好像也不是很活跃
* Nats
  * [nats-io/nats.rs](https://github.com/nats-io/nats.rs) Nats官方提供的客户端

### 网络、通信协议
* Websocket
  * [snapview/tokio-tungstenite](https://github.com/snapview/tokio-tungstenite) 更适合Web应用使用的生产级Websocket库，它是异步非阻塞的，基于基于下下面的`tungstenite-rs`库和tokio实现
  * [rust-websocket](https://github.com/websockets-rs/rust-websocket)  老牌Websocket库，提供了客户端和服务器端实现，但是。。。很久没更新了
  * [snapview/tungstenite-rs](https://github.com/snapview/tungstenite-rs) 轻量级的Websocket流实现，该库更偏底层，例如，你可以用来构建其它网络库
* gRPC
  * [hyperium/tonic](https://github.com/hyperium/tonic) 纯Rust实现的gRPC客户端和服务器端，支持async/await异步调用，文档和示例较为清晰
  * [tikv/grpc-rs](https://github.com/tikv/grpc-rs) 国产开源之光Tidb团队出品的gRPC框架, 基于C的代码实现, 就是最近好像不是很活跃
  * 其实这两个实现都很优秀，把`tonic`放在第一位，主要是因为它是纯Rust实现，同时社区也更为活跃，但是并不代表它比`tikv`的更好！
* QUIC
  * [cloudflare/quiche](https://github.com/cloudflare/quiche) 大名鼎鼎`cloudflare`提供的QUIC实现，据说在公司内部重度使用，有了大规模生产级别的验证，非常值得信任，同时该库还实现了HTTP/3
  * [quinn-rs/quinn](https://github.com/quinn-rs/quinn) 提供异步API调用，纯Rust实现，同时提供了几个有用的网络库
* MQTT
  * [bytebeamio/rumqtt](https://github.com/bytebeamio/rumqtt)  MQTT3.1.1/5协议库，同时实现了客户端与服务器端broker
  * [ntex-rs/ntex-mqtt](https://github.com/ntex-rs/ntex-mqtt)  客户端与服务端框架，支持MQTT3.1.1与5协议
  * [eclipse/paho.mqtt.rust](https://github.com/eclipse/paho.mqtt.rust)  老牌MQTT框架，对MQTT支持较全, 其它各语言的实现也有

### 异步网络编程

* [tokio-rs/tokio](https://github.com/tokio-rs/tokio) 最火的异步网络库，除了复杂上手难度高一些外，没有其它大的问题。同时tokio团队提供了多个非常优秀的Rust库，整个生态欣欣向荣，用户认可度很高
* [async-std](https://async.rs/) 跟标准库API很像的异步网络库，相对简单易用，但是貌似开发有些停滞，还有就是功能上不够完善。但是对于普通用户来说，这个库非常值得一试，它在功能和简单易用上取得了很好的平衡
* [actix](https://github.com/actix/actix) 基于Actor模型的异步网络库，但这个库的开发貌似已经停滞，他们团队一直在专注于`actix-web`的开发
* [mio](https://github.com/tokio-rs/mio) 严格来说，MIO与之前三个不是同一个用途的，MIO = Meta IO，是一个底层IO库，往往用于构建其它网络库，当然如果你对应用网络性能有非常极限的要求， 可以考虑它，因为它的层次比较低，所带来的抽象负担小，所以性能损耗小
* 如果你要开发生产级别的项目，我推荐使用`tokio`，稳定可靠，功能丰富，控制粒度细；自己的学习项目或者没有那么严肃的开源项目，我推荐`async-std`，简单好用，值得学习；当你确切知道需要Actor网络模型时，就用`actix`


### 搜索平台

* ElasticSearch
  * [elastic/elasticsearch](https://github.com/elastic/elasticsearch-rs) 
  * [benashford/rs-es](https://github.com/benashford/rs-es) [[rs-es](https://crates.io/crates/rs-es)] — A Rust client for the [Elastic](https://www.elastic.co/) REST API 

* tantivy
  * [Tantivy](https://github.com/quickwit-inc/tantivy) - Tantivy is a full-text search engine library inspired by Apache Lucene and written in Rust

* MeiliSearch
  * [MeiliSearch](https://github.com/meilisearch/MeiliSearch) - Powerful, fast, and an easy to use search engine
 
### 代码Debug

* GDB
  * [gdbgui](https://github.com/cs01/gdbgui) — Browser based frontend for gdb to debug C, C++, Rust, and go. [![build badge](https://api.travis-ci.org/cs01/gdbgui.svg?branch=master)](https://travis-ci.org/cs01/gdbgui)
* LLDB
  * [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) — A LLDB extension for [Visual Studio Code](https://code.visualstudio.com/).

### 性能优化
* [bheisler/criterion.rs](https://github.com/bheisler/criterion.rs) — Statistics-driven benchmarking library for Rust
* [Bytehound](https://github.com/koute/bytehound) — A memory profiler for Linux
* [ellisonch/rust-stopwatch](https://github.com/ellisonch/rust-stopwatch) — A stopwatch library 
* FlameGraphs
  * [llogiq/flame](https://github.com/llogiq/flame) 
  * [mrhooray/torch](https://github.com/mrhooray/torch) — generates FlameGraphs based on DWARF Debug Info
* [sharkdp/hyperfine](https://github.com/sharkdp/hyperfine) — A command-line benchmarking tool 


### 编解码
* CSV
  * [BurntSushi/rust-csv](https://github.com/BurntSushi/rust-csv) — A fast and flexible CSV reader and writer, with support for Serde 
* JSON
  * [importcjj/rust-ajson](https://github.com/importcjj/rust-ajson) [[ajson]](https://crates.io/crates/ajson) —  Get JSON values quickly 
  * [maciejhirsz/json-rust](https://github.com/maciejhirsz/json-rust) [[json](https://crates.io/crates/json)] — JSON implementation in Rust 
  * [pikkr/pikkr](https://github.com/pikkr/pikkr) [[pikkr](https://crates.io/crates/pikkr)] — JSON parser which picks up values directly without performing tokenization in Rust
  * [serde-rs/json](https://github.com/serde-rs/json) [[serde\_json](https://crates.io/crates/serde_json)] — JSON support for [Serde](https://github.com/serde-rs/serde) framework 
  * [simd-lite/simd-json](https://github.com/simd-lite/simd-json) [[simd-json](https://crates.io/crates/simd-json)] — High performance JSON parser based on a port of simdjson
* MsgPack
  * [3Hren/msgpack-rust](https://github.com/3Hren/msgpack-rust) — A pure Rust low/high level MessagePack implementation 
* ProtocolBuffers
  * [stepancheg/rust-protobuf](https://github.com/stepancheg/rust-protobuf) 
  * [tokio-rs/prost](https://github.com/tokio-rs/prost) 
* TOML
  * [alexcrichton/toml-rs](https://github.com/alexcrichton/toml-rs)
* XML
  * [Florob/RustyXML](https://github.com/Florob/RustyXML) — an XML parser written in Rust
  * [media-io/yaserde](https://github.com/media-io/yaserde) — Yet Another Serializer/Deserializer specialized for XML 
  * [netvl/xml-rs](https://github.com/netvl/xml-rs) — A streaming XML library 
  * [shepmaster/sxd-document](https://github.com/shepmaster/sxd-document) — An XML library in Rust 
  * [shepmaster/sxd-xpath](https://github.com/shepmaster/sxd-xpath) — An XPath library in Rust 
  * [tafia/quick-xml](https://github.com/tafia/quick-xml) — High performance XML pull reader/writer 
* YAML
  * [chyh1990/yaml-rust](https://github.com/chyh1990/yaml-rust) — The missing YAML 1.2 implementation for Rust. 
  * [dtolnay/serde-yaml](https://github.com/dtolnay/serde-yaml) [[serde\_yaml](https://crates.io/crates/serde_yaml)] — YAML support for [Serde](https://github.com/serde-rs/serde) framework 
  * [vitiral/stfu8](https://github.com/vitiral/stfu8) [[stfu8](https://crates.io/crates/stfu8)] — Sorta Text Format in UTF-8 

### Email
* [lettre/lettre](https://github.com/lettre/lettre) — an SMTP-library for Rust 

### 常用正则模版

## 精选中文学习资料

## 精选Rust开源项目
