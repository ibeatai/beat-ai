# 关掉你的编译检查
Rust的项目一向都是比较严肃的，虽不至于讲大道理，但是总归与有趣关系不大。但是这两天出现了一个搅局者，一起来看看。

该项目的名字很直白 [`turn_off_the_borrow_checker`](https://github.com/jeremyBanks/you-can) ，作用也很直接：通过引入一个宏关掉Rust的编译检查，哦呦，那是不是意味着以后再也没有人对着咱的代码指手画脚了？结论会在文末给出，先来看个例子。

## 一个绕过编译检查的例子
```rust
#[you_can::turn_off_the_borrow_checker]
fn main() {
    let mut source = 1;
    let mutable_alias = &mut source;
    source = 2; // 这里本该报错，但是引入宏后，不再会报错
    *mutable_alias = 3;
    println!("{source}");
}
```

`&%$#.*`，我的大脑一片混乱，这到底是如何做到的？宏还可以影响编译行为？闻所未闻啊，好在官方文档给出了解释。

## 原理解释
`turn_off_the_borrow_checker` 宏会在你的代码中寻找通过 `&` 、`&mut` 或 `ref`、`ref mut` 创建的引用，然后通过 [`::unbounded::reference()`](https://docs.rs/unbounded/0.0.2022/unbounded/fn.reference.html) 将其包裹起来，以实现[无界生命周期](https://course.rs/advance/lifetime/advance.html#无界生命周期)的效果.

这样编译器就会无视这些小透明，效果如同以下代码：
```rust
fn main() {
    let mut source = 1;
    let mutable_alias = ::unbounded::reference(&mut source);
    source = 2;
    *mutable_alias = 3;
    println!("{source}");
}
```

## 局限性
如果这个宏能可选的绕过我们在某些时候不希望有的编译检查就好了，可惜世事总是不遂人意。 它不是万能的，它无法消除由非法代码创建的生命周期或隐式创建的引用而导致的错误。

有时你可以通过引入前缀 `&*` 来强制一个引用成为显式的，这在某种程度上可以解决上述的局限性：

```rust
#[you_can::turn_off_the_borrow_checker]
fn main() {
    let mut source = Some(1);
    let inner_mut = &*source.as_ref().unwrap();
    let mutable_alias = &mut source;

    source = None;
    *mutable_alias = Some(2);

    if let Some(ref mut inner_a) = source {
        match source {
            Some(ref mut inner_b) => {
                *inner_b = inner_mut + 1;
                *inner_a = inner_mut + 2;
            },
            None => {
                println!("none");
            },
        }
    }

    println!("{source:?}");
}
```

以上代码使用宏后展开如下:
```rust
fn main() {
    let mut source = Some(1);
    let inner_mut = ::unbounded::reference(&*source.as_ref().unwrap());
    let mutable_alias = ::unbounded::reference(&mut source);

    source = None;
    *mutable_alias = Some(2);

    if let Some(ref mut inner_a) = source {
        let inner_a = ::unbounded::reference(inner_a);

        match source {
            Some(ref mut inner_b) => {
                let inner_b = ::unbounded::reference(inner_b);

                *inner_b = inner_mut + 1;
                *inner_a = inner_mut + 2;
            },
            None => {
                println!("none");
            },
        }
    }

    println!("{source:?}");
}
```

## 使用场景
其实这篇文章是 [Fancy Rust](https://fancy.rs) 的其中一篇，所属的目录是**有趣的 Rust 项目**，从这里大概就能猜出我们完全不推荐在生产环境使用该项目。

作者也是这个态度：此项目仅适用于教学或耍帅的目的，如果要在生产使用，那勇士，你走好：）
