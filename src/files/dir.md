# 目录访问

### 获取24小时内被修改过的文件
通过遍历读取目录中文件的 [Metadata::modified](https://doc.rust-lang.org/std/fs/struct.Metadata.html#method.modified) 属性，来获取目标文件名列表。

```rust,editable
use error_chain::error_chain;

use std::{env, fs};

error_chain! {
    foreign_links {
        Io(std::io::Error);
        SystemTimeError(std::time::SystemTimeError);
    }
}

fn main() -> Result<()> {
    let current_dir = env::current_dir()?;
    println!(
        "Entries modified in the last 24 hours in {:?}:",
        current_dir
    );

    for entry in fs::read_dir(current_dir)? {
        let entry = entry?;
        let path = entry.path();

        let metadata = fs::metadata(&path)?;
        let last_modified = metadata.modified()?.elapsed()?.as_secs();

        if last_modified < 24 * 3600 && metadata.is_file() {
            println!(
                "Last modified: {:?} seconds, is read only: {:?}, size: {:?} bytes, filename: {:?}",
                last_modified,
                metadata.permissions().readonly(),
                metadata.len(),
                path.file_name().ok_or("No filename")?
            );
        }
    }

    Ok(())
}
```

### 获取给定路径的 loops

使用 [same_file::is_same_file]() 可以检查给定路径的 loops，loop 可以通过以下方式创建:
```shell
mkdir -p /tmp/foo/bar/baz
ln -s /tmp/foo/  /tmp/foo/bar/baz/qux
```

```rust,editable
use std::io;
use std::path::{Path, PathBuf};
use same_file::is_same_file;

fn contains_loop<P: AsRef<Path>>(path: P) -> io::Result<Option<(PathBuf, PathBuf)>> {
    let path = path.as_ref();
    let mut path_buf = path.to_path_buf();
    while path_buf.pop() {
        if is_same_file(&path_buf, path)? {
            return Ok(Some((path_buf, path.to_path_buf())));
        } else if let Some(looped_paths) = contains_loop(&path_buf)? {
            return Ok(Some(looped_paths));
        }
    }
    return Ok(None);
}

fn main() {
    assert_eq!(
        contains_loop("/tmp/foo/bar/baz/qux/bar/baz").unwrap(),
        Some((
            PathBuf::from("/tmp/foo"),
            PathBuf::from("/tmp/foo/bar/baz/qux")
        ))
    );
}
```

### 递归查找重复的文件名

[walkdir](https://docs.rs/walkdir/latest/walkdir/) 可以帮助我们遍历指定的目录。

```rust,editable
use std::collections::HashMap;
use walkdir::WalkDir;

fn main() {
    let mut filenames = HashMap::new();

    // 遍历当前目录
    for entry in WalkDir::new(".")
            .into_iter()
            .filter_map(Result::ok)
            .filter(|e| !e.file_type().is_dir()) {
        let f_name = String::from(entry.file_name().to_string_lossy());
        let counter = filenames.entry(f_name.clone()).or_insert(0);
        *counter += 1;

        if *counter == 2 {
            println!("{}", f_name);
        }
    }
}
```

### 递归查找满足条件的所有文件

下面的代码通过 [walkdir](https://docs.rs/walkdir/latest/walkdir/) 来查找当前目录中最近一天内发生过修改的所有文件。

[follow_links](https://docs.rs/walkdir/*/walkdir/struct.WalkDir.html#method.follow_links) 为 `true` 时，那软链接会被当成正常的文件或目录一样对待，也就是说软链接指向的文件或目录也会被访问和检查。若软链接指向的目标不存在或它是一个 loops，就会导致错误的发生。

```rust,editable
#use error_chain::error_chain;

use walkdir::WalkDir;

#error_chain! {
#    foreign_links {
#        WalkDir(walkdir::Error);
#        Io(std::io::Error);
#        SystemTime(std::time::SystemTimeError);
#    }
#}

fn main() -> Result<()> {
    for entry in WalkDir::new(".")
            .follow_links(true)
            .into_iter()
            .filter_map(|e| e.ok()) {
        let f_name = entry.file_name().to_string_lossy();
        let sec = entry.metadata()?.modified()?;

        if f_name.ends_with(".json") && sec.elapsed()?.as_secs() < 86400 {
            println!("{}", f_name);
        }
    }

    Ok(())
}
```

### 遍历目录跳过隐藏文件

下面例子使用 [walkdir](https://docs.rs/walkdir/latest/walkdir/) 来遍历一个目录，同时跳过隐藏文件 `is_not_hidden`。

```rust,editable
use walkdir::{DirEntry, WalkDir};

fn is_not_hidden(entry: &DirEntry) -> bool {
    entry
         .file_name()
         .to_str()
         .map(|s| entry.depth() == 0 || !s.starts_with("."))
         .unwrap_or(false)
}

fn main() {
    WalkDir::new(".")
        .into_iter()
        .filter_entry(|e| is_not_hidden(e))
        .filter_map(|v| v.ok())
        .for_each(|x| println!("{}", x.path().display()));
}
```

### 递归计算给定深度的文件大小
递归访问的深度可以使用 [WalkDir::min_depth](https://docs.rs/walkdir/*/walkdir/struct.WalkDir.html#method.min_depth) 和 [WalkDir::max_depth](https://docs.rs/walkdir/2.3.2/walkdir/struct.WalkDir.html#method.max_depth) 来控制。

```rust,editable
use walkdir::WalkDir;

fn main() {
    let total_size = WalkDir::new(".")
        .min_depth(1)
        .max_depth(3)
        .into_iter()
        .filter_map(|entry| entry.ok())
        .filter_map(|entry| entry.metadata().ok())
        .filter(|metadata| metadata.is_file())
        .fold(0, |acc, m| acc + m.len());

    println!("Total size: {} bytes.", total_size);
}
```

### 递归查找所有 png 文件
例子中使用了 [glob](https://docs.rs/glob/) 包，其中的 `**` 代表当前目录及其所有子目录，例如，`/media/**/*.png` 代表在 `media` 和它的所有子目录下查找 png 文件.

```rust,editable
#use error_chain::error_chain;

use glob::glob;

#error_chain! {
#    foreign_links {
#        Glob(glob::GlobError);
#        Pattern(glob::PatternError);
#    }
#}

fn main() -> Result<()> {
    for entry in glob("**/*.png")? {
        println!("{}", entry?.display());
    }

    Ok(())
}
```

### 查找满足给定正则的所有文件且忽略文件名大小写

[glob_with](https://docs.rs/glob/*/glob/fn.glob_with.html) 函数可以按照给定的正则表达式进行查找，同时还能使用选项来控制一些匹配设置。

```rust,editable
use error_chain::error_chain;
use glob::{glob_with, MatchOptions};

error_chain! {
    foreign_links {
        Glob(glob::GlobError);
        Pattern(glob::PatternError);
    }
}

fn main() -> Result<()> {
    let options = MatchOptions {
        case_sensitive: false,
        ..Default::default()
    };

    for entry in glob_with("/media/img_[0-9]*.png", options)? {
        println!("{}", entry?.display());
    }

    Ok(())
}
```