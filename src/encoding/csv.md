# CSV

## 读取 CSV 记录
我们可以将标准的 CSV 记录值读取到 [csv::StringRecord](https://docs.rs/csv/*/csv/struct.StringRecord.html) 中，但是该数据结构期待合法的 UTF8 数据行，你还可以使用 [csv::ByteRecord](https://docs.rs/csv/1.1.6/csv/struct.ByteRecord.html) 来读取非 UTF8 数据。

```rust,editable
use csv::Error;

fn main() -> Result<(), Error> {
    let csv = "year,make,model,description
        1948,Porsche,356,Luxury sports car
        1967,Ford,Mustang fastback 1967,American car";

    let mut reader = csv::Reader::from_reader(csv.as_bytes());
    for record in reader.records() {
        let record = record?;
        println!(
            "In {}, {} built the {} model. It is a {}.",
            &record[0],
            &record[1],
            &record[2],
            &record[3]
        );
    }

    Ok(())
}
```

还可以使用 [`serde`](https://docs.rs/serde/1.0.136/serde/) 将数据反序列化成一个强类型的结构体。

```rust,editable
use serde::Deserialize;
#[derive(Deserialize)]
struct Record {
    year: u16,
    make: String,
    model: String,
    description: String,
}

fn main() -> Result<(), csv::Error> {
    let csv = "year,make,model,description
1948,Porsche,356,Luxury sports car
1967,Ford,Mustang fastback 1967,American car";

    let mut reader = csv::Reader::from_reader(csv.as_bytes());

    for record in reader.deserialize() {
        let record: Record = record?;
        println!(
            "In {}, {} built the {} model. It is a {}.",
            record.year,
            record.make,
            record.model,
            record.description
        );
    }

    Ok(())
}
```

### 读取使用了不同分隔符的 CSV 记录
下面的例子将读取使用了 `tab` 作为分隔符的 CSV 记录。

```rust,editable
use csv::Error;
use serde::Deserialize;
#[derive(Debug, Deserialize)]
struct Record {
    name: String,
    place: String,
    #[serde(deserialize_with = "csv::invalid_option")]
    id: Option<u64>,
}

use csv::ReaderBuilder;

fn main() -> Result<(), Error> {
    let data = "name\tplace\tid
        Mark\tMelbourne\t46
        Ashley\tZurich\t92";

    let mut reader = ReaderBuilder::new().delimiter(b'\t').from_reader(data.as_bytes());
    for result in reader.deserialize::<Record>() {
        println!("{:?}", result?);
    }

    Ok(())
}
```

### 基于给定条件来过滤 CSV 记录

```rust,editable
use error_chain::error_chain;

use std::io;

error_chain!{
    foreign_links {
        Io(std::io::Error);
        CsvError(csv::Error);
    }
}

fn main() -> Result<()> {
    let query = "CA";
    let data = "\
City,State,Population,Latitude,Longitude
Kenai,AK,7610,60.5544444,-151.2583333
Oakman,AL,,33.7133333,-87.3886111
Sandfort,AL,,32.3380556,-85.2233333
West Hollywood,CA,37031,34.0900000,-118.3608333";

    let mut rdr = csv::ReaderBuilder::new().from_reader(data.as_bytes());
    let mut wtr = csv::Writer::from_writer(io::stdout());

    wtr.write_record(rdr.headers()?)?;

    for result in rdr.records() {
        let record = result?;
        if record.iter().any(|field| field == query) {
            wtr.write_record(&record)?;
        }
    }

    wtr.flush()?;
    Ok(())
}
```

### 序列化为 CSV

下面例子展示了如何将 Rust 类型序列化为 CSV。

```rust,editable
use std::io;

fn main() -> Result<()> {
    let mut wtr = csv::Writer::from_writer(io::stdout());

    wtr.write_record(&["Name", "Place", "ID"])?;

    wtr.serialize(("Mark", "Sydney", 87))?;
    wtr.serialize(("Ashley", "Dublin", 32))?;
    wtr.serialize(("Akshat", "Delhi", 11))?;

    wtr.flush()?;
    Ok(())
}
```

### 使用 serde 序列化为 CSV
下面例子将自定义数据结构通过 `serde` 序列化 CSV。
```rust,editable
use error_chain::error_chain;
use serde::Serialize;
use std::io;

error_chain! {
   foreign_links {
       IOError(std::io::Error);
       CSVError(csv::Error);
   }
}

#[derive(Serialize)]
struct Record<'a> {
    name: &'a str,
    place: &'a str,
    id: u64,
}

fn main() -> Result<()> {
    let mut wtr = csv::Writer::from_writer(io::stdout());

    let rec1 = Record { name: "Mark", place: "Melbourne", id: 56};
    let rec2 = Record { name: "Ashley", place: "Sydney", id: 64};
    let rec3 = Record { name: "Akshat", place: "Delhi", id: 98};

    wtr.serialize(rec1)?;
    wtr.serialize(rec2)?;
    wtr.serialize(rec3)?;

    wtr.flush()?;

    Ok(())
}
```

### CSV 列转换
下面代码将包含有颜色名和十六进制颜色的 CSV 文件转换为包含颜色名和 rgb 颜色。这里使用 `csv` 包对 CSV 文件进行读写，然后用 `serde` 进行序列化和反序列化。

```rust,editable
#use error_chain::error_chain;
use csv::{Reader, Writer};
use serde::{de, Deserialize, Deserializer};
use std::str::FromStr;

#error_chain! {
#   foreign_links {
#       CsvError(csv::Error);
#       ParseInt(std::num::ParseIntError);
#       CsvInnerError(csv::IntoInnerError<Writer<Vec<u8>>>);
#       IO(std::fmt::Error);
#       UTF8(std::string::FromUtf8Error);
#   }
#}

#[derive(Debug)]
struct HexColor {
    red: u8,
    green: u8,
    blue: u8,
}

#[derive(Debug, Deserialize)]
struct Row {
    color_name: String,
    color: HexColor,
}

impl FromStr for HexColor {
    type Err = Error;

    fn from_str(hex_color: &str) -> std::result::Result<Self, Self::Err> {
        let trimmed = hex_color.trim_matches('#');
        if trimmed.len() != 6 {
            Err("Invalid length of hex string".into())
        } else {
            Ok(HexColor {
                red: u8::from_str_radix(&trimmed[..2], 16)?,
                green: u8::from_str_radix(&trimmed[2..4], 16)?,
                blue: u8::from_str_radix(&trimmed[4..6], 16)?,
            })
        }
    }
}

impl<'de> Deserialize<'de> for HexColor {
    fn deserialize<D>(deserializer: D) -> std::result::Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        let s = String::deserialize(deserializer)?;
        FromStr::from_str(&s).map_err(de::Error::custom)
    }
}

fn main() -> Result<()> {
    let data = "color_name,color
red,#ff0000
green,#00ff00
blue,#0000FF
periwinkle,#ccccff
magenta,#ff00ff"
        .to_owned();
    let mut out = Writer::from_writer(vec![]);
    let mut reader = Reader::from_reader(data.as_bytes());
    for result in reader.deserialize::<Row>() {
        let res = result?;
        out.serialize((
            res.color_name,
            res.color.red,
            res.color.green,
            res.color.blue,
        ))?;
    }
    let written = String::from_utf8(out.into_inner()?)?;
    assert_eq!(Some("magenta,255,0,255"), written.lines().last());
    println!("{}", written);
    Ok(())
}
```

