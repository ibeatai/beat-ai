# 复数

### 创建复数
[num::complex::Complex](https://autumnai.github.io/cuticula/num/complex/struct.Complex.html) 可以帮助我们创建复数，其中实部和虚部必须是一样的类型。

```rust,editable
fn main() {
    let complex_integer = num::complex::Complex::new(10, 20);
    let complex_float = num::complex::Complex::new(10.1, 20.1);

    println!("Complex integer: {}", complex_integer);
    println!("Complex float: {}", complex_float);
}
```

### 复数相加
复数计算和 Rust 基本类型的计算并无区别。

```rust,editable
fn main() {
    let complex_num1 = num::complex::Complex::new(10.0, 20.0); // Must use floats
    let complex_num2 = num::complex::Complex::new(3.1, -4.2);

    let sum = complex_num1 + complex_num2;

    println!("Sum: {}", sum);
}
```

### 数学函数
在 [num::complex::Complex](https://autumnai.github.io/cuticula/num/complex/struct.Complex.html) 中定义了一些内置的数学函数，可用于对复数进行数学运算。

```rust,editable
use std::f64::consts::PI;
use num::complex::Complex;

fn main() {
    let x = Complex::new(0.0, 2.0*PI);

    println!("e^(2i * pi) = {}", x.exp()); // =~1
}
```

