# 字符编码

### 百分号编码( Percent encoding )
[百分号编码](https://en.wikipedia.org/wiki/Percent-encoding)又称 URL 编码。

[percent-encoding](https://docs.rs/crate/percent-encoding/2.1.0) 包提供了两个函数：`utf8_percent_encode` 函数用于编码、`percent_decode` 用于解码。

```rust,editable
use percent_encoding::{utf8_percent_encode, percent_decode, AsciiSet, CONTROLS};
use std::str::Utf8Error;

/// https://url.spec.whatwg.org/#fragment-percent-encode-set
const FRAGMENT: &AsciiSet = &CONTROLS.add(b' ').add(b'"').add(b'<').add(b'>').add(b'`');

fn main() -> Result<(), Utf8Error> {
    let input = "confident, productive systems programming";

    let iter = utf8_percent_encode(input, FRAGMENT);
    // 将元素类型为 &str 的迭代器收集为 String 类型
    let encoded: String = iter.collect();
    assert_eq!(encoded, "confident,%20productive%20systems%20programming");

    let iter = percent_decode(encoded.as_bytes());
    let decoded = iter.decode_utf8()?;
    assert_eq!(decoded, "confident, productive systems programming");

    Ok(())
}
```

该编码集定义了哪些字符( 特别是非 ASCII 和控制字符 )需要被百分比编码。具体的选择取决于上下文，例如 `url` 会对 URL 路径中的 `?` 进行编码，但是在路径后的查询字符串中，并不会进行编码。


### 将字符串编码为 application/x-www-form-urlencoded

使用 [form_urlencoded::byte_serialize](https://docs.rs/form_urlencoded/1.0.1/form_urlencoded/fn.byte_serialize.html) 函数将一个字符串编码成 [application/x-www-form-urlencoded](https://url.spec.whatwg.org/#application/x-www-form-urlencoded) 格式，然后再使用 [form_urlencoded::parse](https://docs.rs/form_urlencoded/1.0.1/form_urlencoded/fn.parse.html) 对其进行解码。

```rust,editable
use url::form_urlencoded::{byte_serialize, parse};

fn main() {
    let urlencoded: String = byte_serialize("What is ❤?".as_bytes()).collect();
    assert_eq!(urlencoded, "What+is+%E2%9D%A4%3F");
    println!("urlencoded:'{}'", urlencoded);

    let decoded: String = parse(urlencoded.as_bytes())
        .map(|(key, val)| [key, val].concat())
        .collect();
    assert_eq!(decoded, "What is ❤?");
    println!("decoded:'{}'", decoded);
}
```

### 十六进制编解码

[data_encoding](https://docs.rs/data-encoding/*/data_encoding/) 可以将一个字符串编码成十六进制字符串，反之亦然。

下面的例子将 `&[u8]` 转换成十六进制等效形式，然后与期待的值进行比较。

```rust,editable
use data_encoding::{HEXUPPER, DecodeError};

fn main() -> Result<(), DecodeError> {
    let original = b"The quick brown fox jumps over the lazy dog.";
    let expected = "54686520717569636B2062726F776E20666F78206A756D7073206F76\
        657220746865206C617A7920646F672E";

    let encoded = HEXUPPER.encode(original);
    assert_eq!(encoded, expected);

    let decoded = HEXUPPER.decode(&encoded.into_bytes())?;
    assert_eq!(&decoded[..], &original[..]);

    Ok(())
}
```

### Base64 编解码
[base64](https://docs.rs/base64/0.13.0/base64/index.html) 可以把一个字节切片编码成 `base64` String。

```rust,editable
use error_chain::error_chain;

use std::str;
use base64::{encode, decode};

error_chain! {
    foreign_links {
        Base64(base64::DecodeError);
        Utf8Error(str::Utf8Error);
    }
}

fn main() -> Result<()> {
    // 将 `&str` 转换成 `&[u8; N]` 
    let hello = b"hello rustaceans";
    let encoded = encode(hello);
    let decoded = decode(&encoded)?;

    println!("origin: {}", str::from_utf8(hello)?);
    println!("base64 encoded: {}", encoded);
    println!("back to origin: {}", str::from_utf8(&decoded)?);

    Ok(())
}
```