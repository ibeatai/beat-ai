# 处理器

### 获取逻辑CPU的核心数

[num_cpus](https://docs.rs/num_cpus/latest/num_cpus/) 可以用于获取逻辑和物理的 CPU 核心数，下面的例子是获取逻辑核心数。

```rust,editable
fn main() {
    println!("Number of logical cores is {}", num_cpus::get());
}
```