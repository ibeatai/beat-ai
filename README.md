# fancy-rust
使用我们精心挑选的开源代码，让你的Rust项目fancy起来!

# 工具类Rust项目
## Web
* HTTP客户端
  * [seanmonstar/reqwest](https://github.com/seanmonstar/reqwest) — an ergonomic HTTP Client for Rust. 

* Web服务器框架
  * [actix/actix-web](https://github.com/actix/actix-web) — A lightweight async web framework for Rust with websocket support 
  * [Rocket](https://github.com/SergioBenitez/Rocket) — Rocket is web framework for Rust (nightly) with a focus on ease-of-use, expressability, and speed 
  * [axum](https://github.com/tokio-rs/axum) - Ergonomic and modular web framework built with Tokio, Tower, and Hyper

## 日志监控
* 日志
[[crates.io](https://crates.io/keywords/log)] [[github](https://github.com/search?q=rust+log)]
  * [slog-rs/slog](https://github.com/slog-rs/slog) — Structured, composable logging for Rust 
  * [tokio-rs/tracing](https://github.com/tokio-rs/tracing) — An application level tracing framework for async-aware structured logging, error handling, metrics, and more 
  * [estk/log4rs](https://github.com/estk/log4rs) — highly configurable logging framework modeled after Java's Logback and log4j libraries 
  * [jesusprubio/leg](https://github.com/jesusprubio/leg) — Elegant print for lazy devs. Make your CLIs nicer with minimal effort.
  * [rust-lang/log](https://github.com/rust-lang/log) — Logging implementation for Rust 
  * [seanmonstar/pretty-env-logger](https://github.com/seanmonstar/pretty-env-logger) — A pretty, easy-to-use logger for Rust. 

* 监控
  * [OpenTelemetry](https://crates.io/crates/opentelemetry) — OpenTelemetry provides a single set of APIs, libraries, agents, and collector services to capture distributed traces and metrics from your application. You can analyze them using Prometheus, Jaeger, and other observability tools.
  * [vectordotdev/vector](https://github.com/vectordotdev/vector) — A High-Performance, Logs, Metrics, & Events Router.

## SQL客户端
* 通用
  * [launchbadge/sqlx](https://github.com/launchbadge/sqlx) - async PostgreSQL/MySQL/SQLite connection pool with strong typing support

* ORM
  * [diesel-rs/diesel](https://github.com/diesel-rs/diesel) — an ORM and Query builder for Rust
  * [ivanceras/rustorm](https://github.com/ivanceras/rustorm) — an ORM for Rust 
  * [rbatis/rbatis](https://github.com/rbatis/rbatis) — Rust ORM Framework High Performance(JSON based) 
  * [SeaQL/sea-orm](https://github.com/SeaQL/sea-orm) — an async & dynamic ORM for Rust 

* Mysql
  * [blackbeam/mysql_async](https://github.com/blackbeam/mysql_async) — asyncronous Rust Mysql driver based on Tokio.
  * [blackbeam/rust-mysql-simple](https://github.com/blackbeam/rust-mysql-simple) — A native MySql client 

* Postgre
  * [sfackler/rust-postgres](https://github.com/sfackler/rust-postgres) [[postgres](https://crates.io/crates/postgres)] — A native [PostgreSQL](https://www.postgresql.org/) client

* Sqlite
  * [rusqlite](https://github.com/rusqlite/rusqlite) — [Sqlite3](https://www.sqlite.org/index.html) bindings

## NoSql客户端

* Redis
  * [mitsuhiko/redis-rs](https://github.com/mitsuhiko/redis-rs) — [Redis](https://redis.io/) library in Rust 

* Canssandra
  * [AlexPikalov/cdrs](https://github.com/AlexPikalov/cdrs) [[cdrs](https://crates.io/crates/cdrs)] — native client written in Rust 
  * [krojew/cdrs-tokio](https://github.com/krojew/cdrs-tokio) [[cdrs-tokio](https://crates.io/crates/cdrs-tokio)] - production-ready async Apache Cassandra driver written in pure Rust 
  * [Metaswitch/cassandra-rs](https://github.com/Metaswitch/cassandra-rs) —  bindings to the DataStax C/C++ client 

* MongoDB
  * [mongodb/mongo-rust-driver](https://github.com/mongodb/mongo-rust-driver) 

## 分布式
#### 服务发现
- [jimmycuadra/rust-etcd](https://github.com/jimmycuadra/rust-etcd) [[etcd](https://crates.io/crates/etcd)] — A client library for CoreOS's etcd.
- [luncj/etcd-rs](https://github.com/luncj/etcd-rs) — An asynchronous etcd client for rust 
#### 消息队列
* Kafka
  * [fede1024/rust-rdkafka](https://github.com/fede1024/rust-rdkafka) [[rdkafka](https://crates.io/crates/rdkafka)] — [librdkafka](https://github.com/edenhill/librdkafka) bindings 
  * [gklijs/schema_registry_converter](https://github.com/gklijs/schema_registry_converter) [[schema_registry_converter](https://crates.io/crates/schema_registry_converter)] — to integrate with [confluent schema registry](https://www.confluent.io/product/confluent-platform/data-compatibility/) 
  * [kafka-rust/kafka-rust](https://github.com/kafka-rust/kafka-rust) 
* Nats
  * [nats-io/nats.rs](https://github.com/nats-io/nats.rs) — Rust client for NATS, the cloud native messaging system. 

## 网络、通信协议
* Grpc
  * [tikv/grpc-rs](https://github.com/tikv/grpc-rs) — The gRPC library for Rust built on C Core library and futures
* QUIC
  * [cloudflare/quiche](https://github.com/cloudflare/quiche) — cloudflare implementation of the QUIC transport protocol and HTTP/3 
* MQTT
  * [](https://github.com/bytebeamio/rumqtt) - MQTT3.1.1/5
## 内容搜索

* ElasticSearch
  * [benashford/rs-es](https://github.com/benashford/rs-es) [[rs-es](https://crates.io/crates/rs-es)] — A Rust client for the [Elastic](https://www.elastic.co/) REST API 
  * [elastic-rs/elastic](https://github.com/elastic-rs/elastic) [[elastic](https://crates.io/crates/elastic)] — elastic is an efficient, modular API client for Elasticsearch written in Rust 

* tantivy
  * [Tantivy](https://github.com/quickwit-inc/tantivy) - Tantivy is a full-text search engine library inspired by Apache Lucene and written in Rust

* MeiliSearch
  * [MeiliSearch](https://github.com/meilisearch/MeiliSearch) - Powerful, fast, and an easy to use search engine
 
## Rust代码Debug

* GDB
  * [gdbgui](https://github.com/cs01/gdbgui) — Browser based frontend for gdb to debug C, C++, Rust, and go. [![build badge](https://api.travis-ci.org/cs01/gdbgui.svg?branch=master)](https://travis-ci.org/cs01/gdbgui)
* LLDB
  * [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) — A LLDB extension for [Visual Studio Code](https://code.visualstudio.com/).


## 编解码
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

## Email
* [lettre/lettre](https://github.com/lettre/lettre) — an SMTP-library for Rust 
# 精选Rust开源项目(值得学习)
