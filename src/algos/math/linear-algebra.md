# 线性代数

### 矩阵相加

使用 [ndarray::arr2](https://docs.rs/ndarray/*/ndarray/fn.arr2.html) 可以创建二阶矩阵，并计算它们的和。

```rust,editable
use ndarray::arr2;

fn main() {
    let a = arr2(&[[1, 2, 3],
                   [4, 5, 6]]);

    let b = arr2(&[[6, 5, 4],
                   [3, 2, 1]]);

    // 借用 a 和 b，求和后生成新的矩阵 sum
    let sum = &a + &b;

    println!("{}", a);
    println!("+");
    println!("{}", b);
    println!("=");
    println!("{}", sum);
}
```

### 矩阵相乘

[ndarray::ArrayBase::dot](https://docs.rs/ndarray/0.15.4/ndarray/struct.ArrayBase.html#method.dot-1) 可以用于计算矩阵乘法。

```rust,editable
use ndarray::arr2;

fn main() {
    let a = arr2(&[[1, 2, 3],
                   [4, 5, 6]]);

    let b = arr2(&[[6, 3],
                   [5, 2],
                   [4, 1]]);

    println!("{}", a.dot(&b));
}
```

### 标量、向量、矩阵相乘

在 `ndarry`中，1 阶数组根据上下文既可以作为行向量也可以作为列向量。如果对你来说，这个行或列的方向很重要，可以考虑使用一行或一列的 2 阶数组来表示。

在下面例子中，由于 1 阶数组处于乘号的右边位置，因此 `dot` 会把它当成列向量来处理。

```rust,editable
use ndarray::{arr1, arr2, Array1};

fn main() {
    let scalar = 4;

    let vector = arr1(&[1, 2, 3]);

    let matrix = arr2(&[[4, 5, 6],
                        [7, 8, 9]]);

    let new_vector: Array1<_> = scalar * vector;
    println!("{}", new_vector);

    let new_matrix = matrix.dot(&new_vector);
    println!("{}", new_matrix);
}
```

### 向量比较

浮点数通常是不精确的，因此比较浮点数不是一件简单的事。[approx](https://docs.rs/approx/*/approx/index.html) 提供的 [assert_abs_diff_eq!](https://docs.rs/approx/0.5.1/approx/macro.assert_abs_diff_eq.html) 宏提供了方便的按元素比较的方式。为了使用 `approx` ，你需要在 `ndarray` 的依赖中开启相应的 feature：例如，在 `Cargo.toml` 中修改 `ndarray` 的依赖引入为 `ndarray = { version = "0.13", features = ["approx"] }`。

```rust,editable
use approx::assert_abs_diff_eq;
use ndarray::Array;

fn main() {
  let a = Array::from(vec![1., 2., 3., 4., 5.]);
  let b = Array::from(vec![5., 4., 3., 2., 1.]);
  let mut c = Array::from(vec![1., 2., 3., 4., 5.]);
  let mut d = Array::from(vec![5., 4., 3., 2., 1.]);
  
  // 消耗 a 和 b 的所有权
  let z = a + b;
  // 借用 c 和 d
  let w =  &c + &d;

  assert_abs_diff_eq!(z, Array::from(vec![6., 6., 6., 6., 6.]));

  println!("c = {}", c);
  c[0] = 10.;
  d[1] = 10.;

  assert_abs_diff_eq!(w, Array::from(vec![6., 6., 6., 6., 6.]));

}
```

### 向量范数( norm )

需要注意的是 `Array` 和 `ArrayView` 都是 `ArrayBase` 的别名。因此一个更通用的参数应该是 `&ArrayBase<S, Ix1> where S: Data`，特别是在你提供一个公共 API 给其它用户时，但由于咱们是内部使用，因此更精准的 `ArrayView1<f64>` 会更适合。

```rust,editable
use ndarray::{array, Array1, ArrayView1};

fn l1_norm(x: ArrayView1<f64>) -> f64 {
    x.fold(0., |acc, elem| acc + elem.abs())
}

fn l2_norm(x: ArrayView1<f64>) -> f64 {
    x.dot(&x).sqrt()
}

fn normalize(mut x: Array1<f64>) -> Array1<f64> {
    let norm = l2_norm(x.view());
    x.mapv_inplace(|e| e/norm);
    x
}

fn main() {
    let x = array![1., 2., 3., 4., 5.];
    println!("||x||_2 = {}", l2_norm(x.view()));
    println!("||x||_1 = {}", l1_norm(x.view()));
    println!("Normalizing x yields {:?}", normalize(x));
}
```

### 矩阵的逆变换
例子中使用 [nalgebra::Matrix3](https://docs.rs/nalgebra/*/nalgebra/base/type.Matrix3.html) 创建一个 3x3 的矩阵，然后尝试对其进行逆变换，获取一个逆矩阵。

```rust,editable
use nalgebra::Matrix3;

fn main() {
    let m1 = Matrix3::new(2.0, 1.0, 1.0, 3.0, 2.0, 1.0, 2.0, 1.0, 2.0);
    println!("m1 = {}", m1);
    match m1.try_inverse() {
        Some(inv) => {
            println!("The inverse of m1 is: {}", inv);
        }
        None => {
            println!("m1 is not invertible!");
        }
    }
}
```

### 序列/反序列化一个矩阵

下面将展示如何将矩阵序列化为 JSON ，然后再反序列化为原矩阵。

```rust,editable
extern crate nalgebra;
extern crate serde_json;

use nalgebra::DMatrix;

fn main() -> Result<(), std::io::Error> {
    let row_slice: Vec<i32> = (1..5001).collect();
    let matrix = DMatrix::from_row_slice(50, 100, &row_slice);

    // 序列化矩阵
    let serialized_matrix = serde_json::to_string(&matrix)?;

    // 反序列化
    let deserialized_matrix: DMatrix<i32> = serde_json::from_str(&serialized_matrix)?;

    // 验证反序列化后的矩阵跟原始矩阵相等
    assert!(deserialized_matrix == matrix);

    Ok(())
}
```