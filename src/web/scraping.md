# 提取网络链接( 爬虫 )

### 从目标网页 HTML 中提取出所有链接

下面代码使用 [reqwest::get](https://docs.rs/reqwest/*/reqwest/fn.get.html) 发起一次 http 请求，然后通过 `select` 包的 [Document::from_read](https://docs.rs/select/*/select/document/struct.Document.html#method.from_read) 将请求的结果解析为 HTML 文档。


```rust,editable
#use error_chain::error_chain;
use select::document::Document;
use select::predicate::Name;

#error_chain! {
#      foreign_links {
#          ReqError(reqwest::Error);
#          IoError(std::io::Error);
#      }
#}

#[tokio::main]
async fn main() -> Result<()> {
  let res = reqwest::get("https://www.rust-lang.org/en-US/")
    .await?
    .text()
    .await?;

  Document::from(res.as_str())
    .find(Name("a"))
    .filter_map(|n| n.attr("href"))
    .for_each(|x| println!("{}", x));

  Ok(())
}
```