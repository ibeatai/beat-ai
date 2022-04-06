# 字符串解析

### 访问 Unicode 字符

[unicode-segmentation]() 包的 [UnicodeSegmentation::graphemes]() 函数可以将 UTF-8 字符串收集成一个 Unicode 字符组成的数组。这样我们就可以通过索引的方式来访问对应的字符了。

```rust,editable
use unicode_segmentation::UnicodeSegmentation;

fn main() {
    let name = "José Guimarães\r\n";
    let graphemes = UnicodeSegmentation::graphemes(name, true)
    	.collect::<Vec<&str>>();
    assert_eq!(graphemes[3], "é");
}
```

### 为自定义结构体实现 FromStr 特征
为我们的 RGB 结构体实现 `FromStr` 特征后，就可以将一个十六进制的颜色表示字符串转换成 RGB 结构体。

```rust,editable
use std::str::FromStr;

#[derive(Debug, PartialEq)]
struct RGB {
    r: u8,
    g: u8,
    b: u8,
}

impl FromStr for RGB {
    type Err = std::num::ParseIntError;


    // 将十六进制的颜色码解析为 `RGB` 的实例
    fn from_str(hex_code: &str) -> Result<Self, Self::Err> {
    
        // u8::from_str_radix(src: &str, radix: u32) 将一个字符串切片按照指定的基数转换为 u8 类型
        let r: u8 = u8::from_str_radix(&hex_code[1..3], 16)?;
        let g: u8 = u8::from_str_radix(&hex_code[3..5], 16)?;
        let b: u8 = u8::from_str_radix(&hex_code[5..7], 16)?;

        Ok(RGB { r, g, b })
    }
}

fn main() {
    let code: &str = &r"#fa7268";
    match RGB::from_str(code) {
        Ok(rgb) => {
            println!(
                r"The RGB color code is: R: {} G: {} B: {}",
                rgb.r, rgb.g, rgb.b
            );
        }
        Err(_) => {
            println!("{} is not a valid color hex code!", code);
        }
    }

    // 测试 from_str 是否按照预期工作
    assert_eq!(
        RGB::from_str(&r"#fa7268").unwrap(),
        RGB {
            r: 250,
            g: 114,
            b: 104
        }
    );
}
```


### 实现 Display 特征
@todo