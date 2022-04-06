# 调用系统命令

### 调用一个外部命令并处理输出内容

下面的代码将调用操作系统中的 `git log --oneline` 命令，然后使用 [regex](https://docs.rs/regex/*/regex/struct.Regex.html) 对它输出到 `stdout` 上的调用结果进行解析，以获取哈希值和最后 5 条提交信息( commit )。

```rust,editable
#use error_chain::error_chain;

use std::process::Command;
use regex::Regex;

#error_chain!{
#    foreign_links {
#        Io(std::io::Error);
#        Regex(regex::Error);
#        Utf8(std::string::FromUtf8Error);
#    }
#}

#[derive(PartialEq, Default, Clone, Debug)]
struct Commit {
    hash: String,
    message: String,
}

fn main() -> Result<()> {
    let output = Command::new("git").arg("log").arg("--oneline").output()?;

    if !output.status.success() {
        error_chain::bail!("Command executed with failing error code");
    }

    let pattern = Regex::new(r"(?x)
                               ([0-9a-fA-F]+) # commit hash
                               (.*)           # The commit message")?;

    String::from_utf8(output.stdout)?
        .lines()
        .filter_map(|line| pattern.captures(line))
        .map(|cap| {
                 Commit {
                     hash: cap[1].to_string(),
                     message: cap[2].trim().to_string(),
                 }
             })
        .take(5)
        .for_each(|x| println!("{:?}", x));

    Ok(())
}
```

### 调用 python 解释器运行代码并检查返回的错误码

```rust,editable
#use error_chain::error_chain;

use std::collections::HashSet;
use std::io::Write;
use std::process::{Command, Stdio};

#error_chain!{
#    errors { CmdError }
#    foreign_links {
#        Io(std::io::Error);
#        Utf8(std::string::FromUtf8Error);
#    }
#}

fn main() -> Result<()> {
    let mut child = Command::new("python").stdin(Stdio::piped())
        .stderr(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()?;

    child.stdin
        .as_mut()
        .ok_or("Child process stdin has not been captured!")?
        .write_all(b"import this; copyright(); credits(); exit()")?;

    let output = child.wait_with_output()?;

    if output.status.success() {
        let raw_output = String::from_utf8(output.stdout)?;
        let words = raw_output.split_whitespace()
            .map(|s| s.to_lowercase())
            .collect::<HashSet<_>>();
        println!("Found {} unique words:", words.len());
        println!("{:#?}", words);
        Ok(())
    } else {
        let err = String::from_utf8(output.stderr)?;
        error_chain::bail!("External command failed:\n {}", err)
    }
}
```

### 通过管道来运行外部命令
下面的例子将显示当前目录中大小排名前十的文件和子目录，效果等效于命令 `du -ah . | sort -hr | head -n 10`。

[`Command`](https://doc.rust-lang.org/std/process/struct.Command.html) 命令代表一个进程，其中父进程通过 [Stdio::piped](https://doc.rust-lang.org/std/process/struct.Stdio.html) 来捕获子进程的输出。

```rust,editable
#use error_chain::error_chain;

use std::process::{Command, Stdio};

#error_chain! {
#    foreign_links {
#        Io(std::io::Error);
#        Utf8(std::string::FromUtf8Error);
#    }
#}

fn main() -> Result<()> {
    let directory = std::env::current_dir()?;
    let mut du_output_child = Command::new("du")
        .arg("-ah")
        .arg(&directory)
        .stdout(Stdio::piped())
        .spawn()?;

    if let Some(du_output) = du_output_child.stdout.take() {
        let mut sort_output_child = Command::new("sort")
            .arg("-hr")
            .stdin(du_output)
            .stdout(Stdio::piped())
            .spawn()?;

        du_output_child.wait()?;

        if let Some(sort_output) = sort_output_child.stdout.take() {
            let head_output_child = Command::new("head")
                .args(&["-n", "10"])
                .stdin(sort_output)
                .stdout(Stdio::piped())
                .spawn()?;

            let head_stdout = head_output_child.wait_with_output()?;

            sort_output_child.wait()?;

            println!(
                "Top 10 biggest files and directories in '{}':\n{}",
                directory.display(),
                String::from_utf8(head_stdout.stdout).unwrap()
            );
        }
    }

    Ok(())
}
```

### 将子进程的 stdout 和 stderr 重定向到同一个文件
下面的例子将生成一个子进程，然后将它的标准输出和标准错误输出都输出到同一个文件中。最终的效果跟 Unix 命令 `ls . oops >out.txt 2>&1` 相同。

[File::try_clone](https://doc.rust-lang.org/std/fs/struct.File.html#method.try_clone) 会克隆一份文件句柄的引用，然后保证这两个句柄在写的时候会使用相同的游标位置。

```rust,editable
use std::fs::File;
use std::io::Error;
use std::process::{Command, Stdio};

fn main() -> Result<(), Error> {
    let outputs = File::create("out.txt")?;
    let errors = outputs.try_clone()?;

    Command::new("ls")
        .args(&[".", "oops"])
        .stdout(Stdio::from(outputs))
        .stderr(Stdio::from(errors))
        .spawn()?
        .wait_with_output()?;

    Ok(())
}
```

### 持续处理子进程的输出
下面的代码会创建一个管道，然后当 `BufReader` 更新时，就持续从 `stdout` 中读取数据。最终效果等同于 Unix 命令 `journalctl | grep usb`。

```rust,editable
use std::process::{Command, Stdio};
use std::io::{BufRead, BufReader, Error, ErrorKind};

fn main() -> Result<(), Error> {
    let stdout = Command::new("journalctl")
        .stdout(Stdio::piped())
        .spawn()?
        .stdout
        .ok_or_else(|| Error::new(ErrorKind::Other,"Could not capture standard output."))?;

    let reader = BufReader::new(stdout);

    reader
        .lines()
        .filter_map(|line| line.ok())
        .filter(|line| line.find("usb").is_some())
        .for_each(|line| println!("{}", line));

     Ok(())
}
```

### 读取环境变量

使用 [std::env::var](https://doc.rust-lang.org/std/env/fn.var.html) 可以读取系统中的环境变量。

```rust,editable
use std::env;
use std::fs;
use std::io::Error;

fn main() -> Result<(), Error> {
    // 读取环境变量 `CONFIG` 的值并写入到 `config_path` 中。
    // 若 `CONFIG` 环境变量没有设置，则使用一个默认的值 "/etc/myapp/config"
    let config_path = env::var("CONFIG")
        .unwrap_or("/etc/myapp/config".to_string());

    let config: String = fs::read_to_string(config_path)?;
    println!("Config: {}", config);

    Ok(())
}
```

