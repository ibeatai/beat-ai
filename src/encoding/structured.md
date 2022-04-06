# 结构化数据

### 序列和反序列非结构化的JSON
[serde_json]() 是一个高性能的 JSON 包，它支持我们在不声明结构体的情况下，去解析 JSON。

```rust,editable
use serde_json::json;
use serde_json::{Value, Error};

fn main() -> Result<(), Error> {
    let j = r#"{
                 "userid": 103609,
                 "verified": true,
                 "access_privileges": [
                   "user",
                   "admin"
                 ]
               }"#;

    let parsed: Value = serde_json::from_str(j)?;

    let expected = json!({
        "userid": 103609,
        "verified": true,
        "access_privileges": [
            "user",
            "admin"
        ]
    });

    assert_eq!(parsed, expected);

    Ok(())
}
```

### 解析 TOML 文件
[toml](https://docs.rs/toml/latest/toml/) 包可以将 TOML 文件的内容解析为一个 `toml::Value` 值，该值能代表任何合法的 TOML 数据。

```rust,editable
use toml::{Value, de::Error};

fn main() -> Result<(), Error> {
    let toml_content = r#"
          [package]
          name = "your_package"
          version = "0.1.0"
          authors = ["You! <you@example.org>"]

          [dependencies]
          serde = "1.0"
          "#;

    let package_info: Value = toml::from_str(toml_content)?;

    assert_eq!(package_info["dependencies"]["serde"].as_str(), Some("1.0"));
    assert_eq!(package_info["package"]["name"].as_str(),
               Some("your_package"));

    Ok(())
}
```

还可以配合 [serde]() 将 TOML 解析到我们自定义的结构体中：
```rust,editable
use serde::Deserialize;

use toml::de::Error;
use std::collections::HashMap;

#[derive(Deserialize)]
struct Config {
    package: Package,
    dependencies: HashMap<String, String>,
}

#[derive(Deserialize)]
struct Package {
    name: String,
    version: String,
    authors: Vec<String>,
}

fn main() -> Result<(), Error> {
    let toml_content = r#"
          [package]
          name = "your_package"
          version = "0.1.0"
          authors = ["You! <you@example.org>"]

          [dependencies]
          serde = "1.0"
          "#;

    let package_info: Config = toml::from_str(toml_content)?;

    assert_eq!(package_info.package.name, "your_package");
    assert_eq!(package_info.package.version, "0.1.0");
    assert_eq!(package_info.package.authors, vec!["You! <you@example.org>"]);
    assert_eq!(package_info.dependencies["serde"], "1.0");

    Ok(())
}
```

### 使用小端字节序来读写整数
[byteorder](https://docs.rs/byteorder/latest/byteorder/) 在自行接收或发送网络字节流时会非常有用( 除非性能要求高，否则还是建议使用 JSON 等数据协议，不要自己做字节流解析 )。

```rust,editable

use byteorder::{LittleEndian, ReadBytesExt, WriteBytesExt};
use std::io::Error;

#[derive(Default, PartialEq, Debug)]
struct Payload {
    kind: u8,
    value: u16,
}

fn main() -> Result<(), Error> {
    let original_payload = Payload::default();
    let encoded_bytes = encode(&original_payload)?;
    let decoded_payload = decode(&encoded_bytes)?;
    assert_eq!(original_payload, decoded_payload);
    Ok(())
}

fn encode(payload: &Payload) -> Result<Vec<u8>, Error> {
    let mut bytes = vec![];
    bytes.write_u8(payload.kind)?;
    bytes.write_u16::<LittleEndian>(payload.value)?;
    Ok(bytes)
}

fn decode(mut bytes: &[u8]) -> Result<Payload, Error> {
    let payload = Payload {
        kind: bytes.read_u8()?,
        value: bytes.read_u16::<LittleEndian>()?,
    };
    Ok(payload)
}
```