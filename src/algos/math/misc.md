# 杂项

### 大整数 Big int
使用 [BitInt](https://docs.rs/num/0.2.0/num/struct.BigInt.html) 可以对超过 128bit  的整数进行计算。

```rust,editable
use num::bigint::{BigInt, ToBigInt};

fn factorial(x: i32) -> BigInt {
    if let Some(mut factorial) = 1.to_bigint() {
        for i in 1..=x {
            factorial = factorial * i;
        }
        factorial
    }
    else {
        panic!("Failed to calculate factorial!");
    }
}

fn main() {
    println!("{}! equals {}", 100, factorial(100));
}
```