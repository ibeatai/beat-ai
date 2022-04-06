# 统计

### 测量中心趋势

下面的一些例子为 Rust 数组中的数据计算它们的中心趋势。

#### 平均值
首先计算的是平均值。

```rust,editable
fn main() {
    let data = [3, 1, 6, 1, 5, 8, 1, 8, 10, 11];

    let sum = data.iter().sum::<i32>() as f32;
    let count = data.len();

    let mean = match count {
       positive if positive > 0 => Some(sum  / count as f32),
       _ => None
    };

    println!("Mean of the data is {:?}", mean);
}
```

#### 中位数
下面使用快速选择算法来计算中位数。该算法只会对可能包含中位数的数据分区进行排序，从而避免了对所有数据进行全排序。

```rust,editable
use std::cmp::Ordering;

fn partition(data: &[i32]) -> Option<(Vec<i32>, i32, Vec<i32>)> {
    match data.len() {
        0 => None,
        _ => {
            let (pivot_slice, tail) = data.split_at(1);
            let pivot = pivot_slice[0];
            let (left, right) = tail.iter()
                .fold((vec![], vec![]), |mut splits, next| {
                    {
                        let (ref mut left, ref mut right) = &mut splits;
                        if next < &pivot {
                            left.push(*next);
                        } else {
                            right.push(*next);
                        }
                    }
                    splits
                });

            Some((left, pivot, right))
        }
    }
}

fn select(data: &[i32], k: usize) -> Option<i32> {
    let part = partition(data);

    match part {
        None => None,
        Some((left, pivot, right)) => {
            let pivot_idx = left.len();

            match pivot_idx.cmp(&k) {
                Ordering::Equal => Some(pivot),
                Ordering::Greater => select(&left, k),
                Ordering::Less => select(&right, k - (pivot_idx + 1)),
            }
        },
    }
}

fn median(data: &[i32]) -> Option<f32> {
    let size = data.len();

    match size {
        even if even % 2 == 0 => {
            let fst_med = select(data, (even / 2) - 1);
            let snd_med = select(data, even / 2);

            match (fst_med, snd_med) {
                (Some(fst), Some(snd)) => Some((fst + snd) as f32 / 2.0),
                _ => None
            }
        },
        odd => select(data, odd / 2).map(|x| x as f32)
    }
}

fn main() {
    let data = [3, 1, 6, 1, 5, 8, 1, 8, 10, 11];

    let part = partition(&data);
    println!("Partition is {:?}", part);

    let sel = select(&data, 5);
    println!("Selection at ordered index {} is {:?}", 5, sel);

    let med = median(&data);
    println!("Median is {:?}", med);
}
```

#### 众数( mode )
下面使用了 `HashMap` 对不同数字出现的次数进行了分别统计。

```rust,editable
use std::collections::HashMap;

fn main() {
    let data = [3, 1, 6, 1, 5, 8, 1, 8, 10, 11];

    let frequencies = data.iter().fold(HashMap::new(), |mut freqs, value| {
        *freqs.entry(value).or_insert(0) += 1;
        freqs
    });

    let mode = frequencies
        .into_iter()
        .max_by_key(|&(_, count)| count)
        .map(|(value, _)| *value);

    println!("Mode of the data is {:?}", mode);
}
```

### 标准偏差

下面一起来看看该如何计算一组测量值的标准偏差和 z-score。

```rust,editable
fn mean(data: &[i32]) -> Option<f32> {
    let sum = data.iter().sum::<i32>() as f32;
    let count = data.len();

    match count {
        positive if positive > 0 => Some(sum / count as f32),
        _ => None,
    }
}

fn std_deviation(data: &[i32]) -> Option<f32> {
    match (mean(data), data.len()) {
        (Some(data_mean), count) if count > 0 => {
            let variance = data.iter().map(|value| {
                let diff = data_mean - (*value as f32);

                diff * diff
            }).sum::<f32>() / count as f32;

            Some(variance.sqrt())
        },
        _ => None
    }
}

fn main() {
    let data = [3, 1, 6, 1, 5, 8, 1, 8, 10, 11];

    let data_mean = mean(&data);
    println!("Mean is {:?}", data_mean);

    let data_std_deviation = std_deviation(&data);
    println!("Standard deviation is {:?}", data_std_deviation);

    let zscore = match (data_mean, data_std_deviation) {
        (Some(mean), Some(std_deviation)) => {
            let diff = data[4] as f32 - mean;

            Some(diff / std_deviation)
        },
        _ => None
    };
    println!("Z-score of data at index 4 (with value {}) is {:?}", data[4], zscore);
}
```