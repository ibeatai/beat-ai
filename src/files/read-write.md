# 文件读写

## 迭代文件中的内容行

```rust,editable
use std::fs::File;
use std::io::{Write, BufReader, BufRead, Error};

fn main() -> Result<(), Error> {
    let path = "lines.txt";

    // 创建文件
    let mut output = File::create(path)?;
    // 写入三行内容
    write!(output, "Rust\n💖\nFun")?;

    let input = File::open(path)?;
    let buffered = BufReader::new(input);

    // 迭代文件中的每一行内容，line 是字符串
    for line in buffered.lines() {
        println!("{}", line?);
    }

    Ok(())
}
```

## 避免对同一个文件进行读写
[same_file](https://docs.rs/same-file/latest/same_file/) 可以帮我们识别两个文件是否是相同的。

```rust,editable
use same_file::Handle;
use std::fs::File;
use std::io::{BufRead, BufReader, Error, ErrorKind};
use std::path::Path;

fn main() -> Result<(), Error> {
    let path_to_read = Path::new("new.txt");

    // 从标准输出上获取待写入的文件名
    let stdout_handle = Handle::stdout()?;
    // 将待写入的文件名跟待读取的文件名进行比较
    let handle = Handle::from_path(path_to_read)?;

    if stdout_handle == handle {
        return Err(Error::new(
            ErrorKind::Other,
            "You are reading and writing to the same file",
        ));
    } else {
        let file = File::open(&path_to_read)?;
        let file = BufReader::new(file);
        for (num, line) in file.lines().enumerate() {
            println!("{} : {}", num, line?.to_uppercase());
        }
    }

    Ok(())
}
```

以下代码会报错，因为待写入的文件名也是 *new.txt*，跟待读取的文件名相同
```shell
cargo run >> ./new.txt
```

### 使用内存映射访问文件
[memmap](https://docs.rs/memmap/) 能创建一个文件的内存映射( memory map )，然后模拟一些非顺序读。

使用内存映射，意味着你将相关的索引加载到内存中，而不是通过 [seek](https://doc.rust-lang.org/std/fs/struct.File.html#method.seek) 的方式去访问文件。

[Mmap::map](https://docs.rs/memmap/*/memmap/struct.Mmap.html#method.map) 函数会假定待映射的文件不会同时被其它进程修改。

```rust,editable
use memmap::Mmap;
use std::fs::File;
use std::io::{Write, Error};

fn main() -> Result<(), Error> {
    write!(File::create("content.txt")?, "My hovercraft is full of eels!")?;

    let file = File::open("content.txt")?;
    let map = unsafe { Mmap::map(&file)? };

    let random_indexes = [0, 1, 2, 19, 22, 10, 11, 29];
    assert_eq!(&map[3..13], b"hovercraft");
    let random_bytes: Vec<u8> = random_indexes.iter()
        .map(|&idx| map[idx])
        .collect();
    assert_eq!(&random_bytes[..], b"My loaf!");
    Ok(())
}
```