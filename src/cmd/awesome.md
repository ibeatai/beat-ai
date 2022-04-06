## 命令行工具
对于每一个程序员而言，命令行工具都非常关键。你对他越熟悉，在使用计算机、处理工作流程等越是高效。

下面我们收集了一些优秀的Rust所写的命令行工具，它们相比目前已有的其它语言的实现，可以提供更加现代化的代码实现、更加高效的性能以及更好的可用性。

### 索引目录
| 新工具 | 替代的目标或功能描述 | 
| ------ | ----------------- |
| [bat](#bat) | cat |
| [exa](#exa) | ls |
| [lsd](#lsd) | ls |
| [fd](#fd) | find |
| [procs](#procs) | ps | 
| [sd](#sd) | sed |
| [dust](#dust) | du |
| [starship](#starship) | 现代化的命令行提示 | 
| [ripgrep](#ripgrep) | grep |
| [tokei](#tokei) | 代码统计工具 | 
| [hyperfine](#hyperfine) | 命令行benchmark工具 | 
| [bottom](#bottom) | top |
| [teeldear](#tealdear) | tldr |
| [grex](#grex) | 根据文本示例生成正则 | 
| [bandwitch](#bandwhich) | 显示进程、连接网络使用情况|
| [zoxide](#zoxide) | cd |
| [delta](#delta) | git可视化 |
| [nushell](#nushell) | 全新的现代化shell |
| [mcfly](#mcfly) | 替代`ctrl + R`命令搜索 | 
| [fselect](#fselect) | 使用SQL语法查找文件 |
| [pueue](#pueue) | 命令行任务管理工具 | 
| [watchexec](#watchexec) | 监视目录文件变动并执行命令 | 
| [dura](#dura) | 更加安全的使用git | 
| [alacritty](#alacritty) | 强大的基于OpenGL的终端 | 
| [broot](#broot) | 可视化访问目录树 | 

### bat
[bat](https://github.com/sharkdp/bat)`克隆了**cat**的功能并提供了语法高亮和Git集成，它支持`Windows`，`MacOS`和`Linux`。同时，它默认提供了多种文件后缀的语法高亮。

<img alt="bat screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/command-line/bat.png?raw=true" class="center"  />



### exa
[exa](https://github.com/ogham/exa)是`ls`命令的现代化实现，后者是目前`Unix/Linux`系统的默认命令，用于列出当前目录中的内容。

<img alt="exa screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/exa.jpg?raw=true" class="center"  />

### lsd
[lsd](https://github.com/Peltoche/lsd) 也是 `ls` 的新实现，同时增加了很多特性，例如：颜色标注、icons、树形查看、更多的格式化选项等。

<img alt="lsd screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/lsd.png?raw=true" class="center"  />

### fd
[fd](https://github.com/sharkdp/fd) 是一个更快、对用户更友好的**find**实现，后者是 `Unix/Linux` 内置的文件目录搜索工具。之所以说它用户友好，一方面是 `API` 非常清晰明了，其次是它对最常用的场景提供了有意义的默认值：例如，想要通过名称搜索文件：

- `fd`: `fd PATTERN`
- `find`: `find -iname 'PATTERN'`

同时 `fd` 性能非常非常高，还提供了非常多的搜索选项，例如允许用户通过 `.gitignore` 文件忽略隐藏的目录、文件等。

<img alt="fd screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/fd.svg?raw=true" class="center"  />

### procs
[procs](https://github.com/dalance/procs) 是 **ps** 的默认实现，后者是 `Unix/Linux` 的内置命令，用于获取进程( `process` )的信息。`proc` 提供了更便利、可读性更好的格式化输出。

<img alt="procs screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/procs.jpg?raw=true" class="center"  />

 ### sd
[sd](https://github.com/chmln/sd) 是 **sed** 命令的现代化实现，后者是 `Unix/Linux` 中内置的工具，用于分析和转换文本。

 `sd` 拥有更简单的使用方式，而且支持方便的正则表达式语法，`sd` 拥有闪电般的性能，比 `sed` 快 **2x-11x** 倍。

以下是其中一个性能测试结果：

*对1.5G大小的 JSON 文本进行简单替换*

`hyperfine -w 3 'sed -E "s/\"/\'/g" *.json >/dev/null' 'sd "\"" "\'" *.json >/dev/null' --export-markdown out.md`

| Command | Mean [s] | Min…Max [s] |
|:---|---:|---:|
| `sed -E "s/\"/'/g" *.json >/dev/null` | 2.338 ± 0.008 | 2.332…2.358 |
| `sed "s/\"/'/g" *.json >/dev/null` | 2.365 ± 0.009 | 2.351…2.378 |
| `sd "\"" "'" *.json >/dev/null` | **0.997 ± 0.006** | 0.987…1.007 |

结果: ~2.35 times faster

### dust
[dust](https://github.com/bootandy/dust) 是一个更符合使用习惯的**du**，后者是 `Unix/Linux` 内置的命令行工具，用于显示硬盘使用情况的统计。

<img alt="dust screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/dust.png?raw=true" class="center"  />

### starship
[starship](https://github.com/starship/starship) 是一个命令行提示，支持任何 `shell` ，包括 `zsh` ，简单易用、非常快且拥有极高的可配置性, 同时支持智能提示。

<img alt="starship screenshot" width="100%"  src="https://raw.githubusercontent.com/starship/starship/master/media/demo.gif" class="center"  />

### [ripgrep](https://github.com/BurntSushi/ripgrep)
[ripgrep](https://github.com/BurntSushi/ripgrep) 是一个性能极高的现代化 `grep` 实现，后者是 `Unix/Linux` 下的内置文件搜索工具。该项目是 Rust 的明星项目，一个是因为性能极其的高，另一个就是源代码质量很高，值得学习, 同时 `Vscode` 使用它作为内置的搜索引擎。

从功能来说，除了全面支持 `grep` 的功能外，`repgre` 支持使用正则递归搜索指定的文件目录，默认使用 `.gitignore` 对指定的文件进行忽略。

<img alt="ripgrep screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/ripgrep.png?raw=true" class="center"  />


### tokei
[tokei](https://github.com/XAMPPRocky/tokei) 可以分门别类的统计目录内的代码行数，速度非常快！

<img alt="tokei screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/tokei.png?raw=true" class="center"  />

### hyperfine
[hyperfine](https://github.com/sharkdp/hyperfine) 是命令行benchmark工具，它支持在多次运行中提供静态的分析，同时支持任何的 `shell` 命令，准确的 `benchmark` 进度和当前预估等等高级特性。


<img alt="hyperfine screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/hyperfine.gif?raw=true" class="center"  />

### bottom
[bottom](https://github.com/ClementTsang/bottom) 是一个现代化实现的 `top`，可以跨平台、图形化的显示进程/系统的当前信息。

<img alt="bottom screenshot" width="100%"  src="https://github.com/ClementTsang/bottom/raw/master/assets/demo.gif" class="center"  />

### tealdear
[tealdear](https://github.com/dbrgn/tealdeer) 是一个更快实现的**tldr**, 一个用于显示 `man pages` 的命令行程序，简单易用、基于例子和社区驱动是主要特性。

<img alt="teeldear screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/teeldear.gif?raw=true" class="center"  />

### bandwhich
[bandwhich](https://github.com/imsnif/bandwhich) 是一个客户端实用工具，用于显示当前进程、连接、远程 IP( hostname ) 的网络信息。

<img alt="bandwhich screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/bandwhich.gif?raw=true" class="center"  />

### grex
[grex](https://github.com/pemistahl/grex) 既是一个命令行工具又是一个库，可以根据用户提供的文本示例生成对应的正则表达式，非常强大。

<img alt="grex screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/grex.gif?raw=true" class="center"  />

### zoxide
[zoxide](https://github.com/ajeetdsouza/zoxide) 是一个智能化的 `cd` 命令，它甚至会记忆你常用的目录。

<img alt="zoxide screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/zoxide.webp?raw=true" class="center"  />

### delta
[delta](https://github.com/dandavison/delta) 是一个 `git` 分页展示工具，支持语法高亮、代码比对、输出 `grep` 等。


<img alt="delta screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/delta.png?raw=true" class="center"  />

### nushell
[nushell](https://github.com/nushell/nushell) 是一个全新的 `shell` ，使用 `Rust` 实现。它的目标是创建一个现代化的 `shell` ：虽然依然基于 `Unix` 的哲学，但是更适合现在的时代。例如，你可以使用 `SQL` 语法来选择你想要的内容！

<img alt="delta screenshot" width="100%"  src="https://github.com/nushell/nushell/raw/main/images/nushell-autocomplete5.gif" class="center"  />

### mcfly
[mcfly](https://github.com/cantino/mcfly) 会替换默认的 `ctrl-R`，用于在终端中搜索历史命令， 它提供了智能提示功能，并且会根据当前目录中最近执行过的上下文命令进行提示。`mcfly` 甚至使用了一个小型的神经网络用于智能提示！

<img alt="mcfly screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/mcfly.png?raw=true" class="center"  />

### fselect
[fselect](https://github.com/jhspetersson/fselect) 允许使用 SQL 语法来查找系统中的文件。它支持复杂查询、聚合查询、.gitignore 忽略文件、通过宽度高度搜索图片、通过 hash 搜索文件、文件属性查询等等，相当强大！

```shell
# 复杂查询
fselect "name from /tmp where (name = *.tmp and size = 0) or (name = *.cfg and size > 1000000)"

# 聚合函数
fselect "MIN(size), MAX(size), AVG(size), SUM(size), COUNT(*) from /home/user/Downloads"

# 格式化函数
fselect "LOWER(name), UPPER(name), LENGTH(name), YEAR(modified) from /home/user/Downloads"
```

### pueue
[pueue](https://github.com/nukesor/pueue) 是一个命令行任务管理工具，它可以管理你的长时间运行的命令，支持顺序或并行执行。简单来说，它可以管理一个命令队列。

<img alt="pueue screenshot" width="100%"  src="https://github.com/studyrs/cookbook-images/blob/main/command-line/pueue.gif?raw=true" class="center"  />

### watchexec
[watchexec](https://github.com/watchexec/watchexec) 可以监视指定的目录、文件的改动，并执行你预设的命令，支持多种配置项和操作系统。

```shell
# 监视当前目录/子目录中的所有js、css、html文件，一旦发生改变，运行`npm run build`命令
$ watchexec -e js,css,html npm run build

# 当前目录/子目录下任何python文件发生改变时，重启`python server.py`
$ watchexec -r -e py -- python server.py
```

### dura
[dura](https://github.com/tkellogg/dura) 运行在后台，监视你的 `git` 目录，提交你未提交的更改但是并不会影响 `HEAD`、当前的分支和 `git` 索引(staged文件)。

如果你曾经遇到过**"完蛋, 我这几天的工作内容丢了"**的情况，那么就可以尝试下 `dura`，`checkout dura brach`，然后代码就可以顺利恢复了：）

**恢复代码**
1. 你可以使用 `dura` 分支来恢复

```console
$ echo "dura/$(git rev-parse HEAD)"
```

1. 也可以手动恢复

```console
# Or, if you don't trust dura yet, `git stash`
$ git reset HEAD --hard
# get the changes into your working directory
$ git checkout $THE_HASH
# last few commands reset HEAD back to master but with changes uncommitted
$ git checkout -b temp-branch
$ git reset master
$ git checkout master
$ git branch -D temp-branch
```

## alacritty 
[alacritty](https://github.com/alacritty/alacritty) 是一个跨平台、基于OpenGL的终端，性能极高的同时还支持丰富的自定义和可扩展性，可以说是非常优秀的现代化终端。

目前已经是 `beta` 阶段，可以作为日常工具来使用。

<img alt="alacritty screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/superstar/alacritty.png?raw=true" class="center"  />

## broot
[`broot`](https://github.com/Canop/broot) 允许你可视化的去访问一个目录结构。

<img alt="broot screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/command-line/broot.png?raw=true" class="center"  />