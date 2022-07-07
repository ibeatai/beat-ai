# 使用Rust增强Javascript
`Javascript`是目前全世界使用最广的语言(TIOBE排行榜比较迷，JS并没有排在第一位，我个人并不认同它的排名)。在过去这么多年中，围绕着`Javascript`已经建立了庞大的基础设施生态：例如使用`webpack`来将多个`js`文件打包成一个；使用`Babel`允许你用现代化的`js`语法编写兼容旧浏览器的代码；使用`Eslint`帮助开发找出代码中潜在的问题，类似`cargo clippy`。

以上的种种都在帮助`js`成为更好的语言和工具，它们是`Web`应用程序得以顺利、高效的开发和运行的基石。这些工具往往使用`Javascript`语言编写，一般来说，是没有问题的，但是在某些时候，可能会存在性能上的瓶颈或者安全隐患，因此阴差阳错、机缘巧合下，`Rust`成为了一个搅局者。

## Javascript基建库
### deno
首先出场的自然是咖位最重的之一，可以说正是因为`deno`和`swc`的横空出世，才让一堆观望的大神对于Rust实现`Javascript`基建有了更强的信心。

`deno`是`node`半逆转后的字序，从此可以看出`deno`是`Node.js`的替代，它的目标是为`Typescript/Javascript`提供一个更现代化、更安全、更强大    的运行时，同时内置了很多强大的工具，可以用于打包、编译成可执行文件、文档、测试、lint等。

值得一提的是，`deno`的不少工具都使用了`swc`进行建造，包括代码审查、格式化、文档生成等。

通过包引入的方式来对比下`deno`和`node`，大家可以自己品味下。

```js
// node
const koa = require("koa" );
const logger = require("@adesso/logger")

// deno
import { Application } from "https://deno.land/x/oak/mod.ts";
import { Logger } from "https://adesso.de/lib/logger.ts"
```


### swc
[`swc`](https://github.com/swc-project/swc)是`Typescript/Javascript`编译器，它可以用来编译、压缩和打包JS，同时支持使用插件进行扩展，例如做代码变换等。

`swc`目前正在被一些知名项目所使用，包括`Next.js`,`Parcel`和`Deno`，还有些著名的公司也在使用它，例如`Vercel`、字节跳动、腾讯等。

它的性能非常非常高，官方号称，在单线程下比`Babel`快20倍，在4核心下比`Babel`快70倍！

几个使用案例：

- [nextjs 12](http://nextjs.org/12), 通过使用`swc`获得了更好的扩展性、性能以及wasm的支持，其中性能方面提升了3倍刷新速度、5倍打包速度
- [Parcel](https://parceljs.org/)，通过使用`swc`改善了10倍的性能

<img alt="parcel screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/parcel.png?raw=true" class="center"  />



官方还提供了一个在线运行的[demo](https://swc.rs/playground)，功能齐全，可以试试。

<img alt="swc screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/swc.jpg?raw=true" class="center"  />

### Rome
[`Rome`](https://github.com/rome/tools)可以用来对`JavaScript`、`TypeScript`、`HTML`、`JSON`、`Markdown` 和 `CSS` 进行lint、编译、打包等功能，它的目标是替代`Babel`、`ESLint`、`webpack`、`Prettier`、`Jest`等。

一开始`Rome`是使用`Typescript`开发，目前正在用`Rust`进行重写。有趣的是: `Rome`的作者也是`Babel`的作者, 后者还是他在学习编译原理时做的。


### fnm
[`fnm`](https://github.com/Schniz/fnm)是一个简单易用、高性能的`Node`版本管理工具，还支持`.nvmrc`文件(`nvm`的`node`版本描述文件)

<img alt="fnm screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/fnm.svg?raw=true" class="center"  />

### boa
[`boa`](https://github.com/boa-dev/boa)是一个高性能的`javascript`词法分析器，解析器和解释器，目前还是实验性质的。

<img alt="boa screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/boa.gif?raw=true" class="center"  />

### napi
[`napi`](https://github.com/napi-rs/napi-rs)可以用于构建基于`Node API`的`Nodejs`插件，目前由`nextjs`主导开发。

### volt
[`volt`](https://github.com/voltpkg/volt)是一个现代化的、高性能、安全可靠的`Javascript`包管理工具。目前该库正处于活跃开发阶段，只供学习使用。

<img alt="volt screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/volt.png?raw=true" class="center"  />

### neon
[`neon`](https://github.com/neon-bindings/neon)可以用于写安全、高性能的原生`Nodejs`模块。

```rust
fn make_an_array(mut cx: FunctionContext) -> JsResult<JsArray> {
    // 创建一些值:
    let n = cx.number(9000);
    let s = cx.string("hello");
    let b = cx.boolean(true);

    // 创建一个新数组:
    let array: Handle<JsArray> = cx.empty_array();

    // 将值推入数组中
    array.set(&mut cx, 0, n)?;
    array.set(&mut cx, 1, s)?;
    array.set(&mut cx, 2, b)?;

    // 返回数组
    Ok(array)
}

register_module!(mut cx, {
    cx.export_function("makeAnArray", make_an_array)
})
```

### resvg-js
[resvg-js](https://github.com/yisibl/resvg-js)是一个高性能`svg`渲染库，使用Rust + Typescript实现。下面的图片通过`svg`实现(羞～～～): 

<img alt="resvg screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/resvg.svg?raw=true" class="center"  />

### deno_lint
[deno_lint](https://github.com/denoland/deno_lint), 由`deno`团队出品的`lint`工具，支持`Javascript/Typescript`，支持`Deno`也支持`Node`。

优点之一就是极致的快:
```shell
[
  {
    "name": "deno_lint",
    "totalMs": 105.3750100000002,
    "runsCount": 5,
    "measuredRunsAvgMs": 21.07500200000004,
    "measuredRunsMs": [
      24.79783199999997,
      19.563640000000078,
      20.759051999999883,
    ]
  },
  {
    "name": "eslint",
    "totalMs": 11845.073306000002,
    "runsCount": 5,
    "measuredRunsAvgMs": 2369.0146612000003,
    "measuredRunsMs": [
      2686.1039550000005,
      2281.501061,
      2298.6185210000003,
    ]
  }
]
```

### rslint
[rslint](https://github.com/rslint/rslint)是一个高性能、可定制性强、简单易用的`Javascript/Typescript` lint分析工具。

```shell
$ echo "let a = foo.hasOwnProperty('bar');" > foo.js
$ rslint ./foo.js
error[no-prototype-builtins]: do not access the object property `hasOwnProperty` directly from `foo`
  ┌─ ./foo.js:1:9
  │
1 │ let a = foo.hasOwnProperty('bar');
  │         ^^^^^^^^^^^^^^^^^^^^^^^^^
  │
help: get the function from the prototype of `Object` and call it
  │
1 │ let a = Object.prototype.hasOwnProperty.call(foo, 'bar');
  │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  │
  ╧ note: the method may be shadowed and cause random bugs and denial of service vulnerabilities

Outcome: 1 fail, 0 warn, 0 success

help: for more information about the errors try the explain command: `rslint explain <rules>`
```


### rusty_v8
[rusty_v8](https://github.com/denoland/rusty_v8)是`v8`的Rust语言绑定，底层封装了`c++ API`。


## 用WASM增强JS

### wasm
[wasm(web assembly)](https://webassembly.org/docs/use-cases/)是一种低级语言，它运行在浏览器中，可以和`javascript`相互调用，几乎所有浏览器都支持， 而且目前有多种高级语言都可以直接编译成`wasm`，更是大大增强了它的地位。

目前来说Rust可以编译成`wasm`，虽然还不够完美，但是它正在以肉眼可见的速度快速发展中。因此同时使用`Rust`和`Javascript`成为了一种可能：将`Rust`编译成`wasm`，再跟`js`进行交互，两者共生共存，各自解决擅长的场景(`wasm`性能高，`js`开发速度快)。

### yew
[`yew`](https://github.com/yewstack/yew)是一个正在活跃开发的`Rust/Wasm`框架，用于构建`Web`客户端应用。

<img alt="yew screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/yew.jpg?raw=true" class="center"  />

### gloo
[gloo]是一个模块化的工具，使用`Rust/WASM`构建快速、可靠的`Web`应用。

```rust
use gloo::{events::EventListener, timers::callback::Timeout};
use wasm_bindgen::prelude::*;

pub struct DelayedHelloButton {
    button: web_sys::Element,
    on_click: events::EventListener,
}

impl DelayedHelloButton {
    pub fn new(document: &web_sys::Document) -> Result<DelayedHelloButton, JsValue> {
        // 创建 `<button>` 元素.
        let button = document.create_element("button")?;

        // 监听button上的`click`事件
        let button2 = button.clone();
        let on_click = EventListener::new(&button, "click", move |_event| {
            // 一秒后，更新button中的文本
            let button3 = button2.clone();
            Timeout::new(1_000, move || {
                button3.set_text_content(Some("Hello from one second ago!"));
            })
            .forget();
        });

        Ok(DelayedHelloButton { button, on_click })
    }
}
```

### wasm-bindgen
[wasm-bindgen](https://github.com/rustwasm/wasm-bindgen)可以让`WASM`模块和`Javascript`模块进行更好的交互。

```rust
use wasm_bindgen::prelude::*;

// 从Web导入 `window.alert` 函数
#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

// 从Rust导出一个`greet`函数到Javascript，该函数会`alert`一条欢迎信息
#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("Hello, {}!", name));
}
```

### wasm-pack
[wasm-pack](https://github.com/rustwasm/wasm-pack)是一站式的解决方案，用于构建和使用Rust生成的WASM，支持在浏览器中或后台的`Node.js`中与`Javascript`进行交互。

<img alt="wasm-pack screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/wasm-pack.gif?raw=true" class="center"  />



### wasmer
[wasmer](https://github.com/wasmerio/wasmer)是业界领先的`WASM`运行时，支持`WASI`和`Emscripten`。

```shell
$ wasmer qjs.wasm
QuickJS - Type "\h" for help
qjs > const i = 1 + 2;
qjs > console.log("hello " + i);
hello 3
```
### wasmtime
[wasmtime](https://github.com/bytecodealliance/wasmtime)是一个为`WASM`设计的`JIT`风格的独立运行时。

```rust
fn main() {
    println!("Hello, world!");
}
```

```shell
$ rustup target add wasm32-wasi
$ rustc hello.rs --target wasm32-wasi
$ wasmtime hello.wasm
Hello, world!
```

### trunk
[trunk](https://github.com/thedodd/trunk)是一个`WASM`构建、打包、Web发布工具。

### photon
[photon]()是高性能的、跨平台的图片处理库，使用`Rust`开发，编译成`WASM`运行，为你的Web应用和`Node.js`应用提供无与伦比的图片处理速度，当然，它既然使用`Rust`开发，也可以作为一个库被你的后台程序所使用。

<img alt="photon screenshot" width="100%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/photon.jpg?raw=true" class="center"  />

### tinysearch
[tinysearch](https://github.com/tinysearch/tinysearch)是一个搜索工具，用于静态网站中的内容搜索，使用`Rust`和`WASM`构建。优点是体积小(适用于浏览器)、性能高、全文索引。

<img alt="tinysearch screenshot" width="80%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/tinysearch.gif?raw=true" class="center"  />

### wasm-pdf
[wasm-pdf](https://github.com/jussiniinikoski/wasm-pdf)通过`Javascript`和`WASM`来生成`PDF`，可以直接在浏览器中使用。


### makepad
[makepad](https://github.com/makepad/makepad)是一个充满创意的Rust开发平台，支持编译成`wasm`，并使用`webGL`进行渲染。

<img alt="makepad screenshot" width="80%" src="https://github.com/studyrs/cookbook-images/blob/main/javascript/makepad.jpg?raw=true" class="center"  />

## Rust + Javascript学习教程
### wasm-book
[wasm-book](https://github.com/rustwasm/book)是一本讲述`Rust`和`wasm`的书，篇幅不算长，但是值得学习，还包含了几个很酷的例子。


### wasm-learning
[`wasm-learning`](https://github.com/second-state/wasm-learning)是一个英文教程，用于学习`Rust`, `wasm`和`Node.js`，你可以学会如何使用`Rust`来为`Nodejs`构建函数，可以同时利用`Rust`的性能、`wasm`的安全性和可移植性、`js`的易用性。

### rust-js-snake-game
[`rust-js-snake-game`](https://github.com/RodionChachura/rust-js-snake-game)是一个用`rust + js + wasm`构建的贪食蛇游戏。
