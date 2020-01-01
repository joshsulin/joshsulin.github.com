---
layout: post
title:  "Rust中的模块和文件"
date:   2020-01-01 20:19:20 +0800
categories: Rust
tags: ['Rust']
---

在开发一个复杂的应用程序的时候，我们需要把各个功能拆分、封装到不同的文件，在需要的时候引用该文件。没人会写一个几万行代码的文件，这样在可读性、复用性和维护性上都很差，几乎所有的编程语言都有自己的模块组织方式，比如Java中的包、C#中的程序集等。Rust也不例外, 但是Rust的模块文档是从顶部设计开始写的，很多概念，有些复杂，我记得刚接触 Rust 时模块让我痛苦挣扎，所以我尝试用一种我认为说得通的方式解释它。

### 什么是 crate

一个crate通常来说是一个项目. Rust创建项目(crate)通常使用包管理器cargo, cargo new hello_cargo, 这样就会新建一个文件夹, 并且文件夹内有一个 Cargo.toml 配置文件，这个文件用于声明依赖，入口，构建选项等项目元数据。 每个 crate 可以独立地在 https://crates.io/ 上发表。

假设我们要创建一个二进制（可执行）项目：
> cargo new --bin

> 项目入口为 src/main.rs

对于二进制项目，src/main.rs 是项目主模块的常用路径。

默认情况下，我们的可执行项目的 src/main.rs 如下：

```
fn main() {
    println!("Hello world!");
}
```

我们可以通过 cargo run 构建和运行这个项目，若只想构建项目，则运行 cargo build

构建一个 crate 的时候，cargo 下载并编译所有所需依赖，默认情况下把临时文件和最终生成文件放入 ./target/ 目录下。 cargo 既是包管理器又是构建系统。

### crate 依赖

让我们向刚才创建的 crate 添加 time 依赖来看看命名空间是怎么工作的。如果你的 Cargo.toml，还没有 [dependencies] 部分，添加它，然后列出您要使用的包名称和版本。这个例子增加了一个 time 箱 (crate) 依赖:

```Rust
[dependencies]
time = "0.1.12"
```

如果我们还想添加一个 regex crate子依赖，我们不需要为每个箱子都添加 [dependencies]。下面就是你的 Cargo.toml 文件整体，看起来像依赖于 time 和 regex 箱:

```Rust
[package]
name = "my-rust"
version = "0.1.0"
authors = ["sulin <723719990@qq.com>"]
edition = "2018"

[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

你现在可以使用 regex crate了.

现在让我们在 src/main.rs 里使用 regex, src/main.rs 如下：

```Rust
fn main() {
    let re = regex::Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    println!("Did our date match? {}", re.is_match("2020-01-01"));
}
```

请注意：

我们不需要使用 use 指令来使用 regex - 它在项目下的文件全局可用，因为它在 Cargo.toml 中被声明为依赖（rust 2018之前的版本则不是这样）

我们完全没必要使用 mod （稍后讲述）

为了明白这篇博客的余下部分，你需要明白 rust 模块仅仅是命名空间 - 他们让你把相关符号组合在一起并保证可见性规则。

我们的 crate 有一个主模块（我们现在所在），它的源在 src/main.rs

regex crate 也有一个入口。因为他是一个库，默认情况下其主入口为 src/lib.rs

在我们主模块范围，我们可以在主模块通过依赖名称使用依赖

总之，我们现在只处理两个模块：我们项目主入口还有 regex 的入口。

### use 指令

如果我们不喜欢一直这样写 regex::Regex::new()，我们可以把 regex::Regex 注入主模块范围。

```Rust
use regex::Regex;
// 我们可以通过 `Regex::new()`

fn main() {
    let re = Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    println!("Did our date match? {}", re.is_match("2020-01-01"));
}
```

如果希望将一个路径下所有公有项引入作用域，可以指定路径后跟 * 通配符. 使用通配符时请多加小心！会使得我们难以推导作用域中有什么名称和它们是在何处定义的。`所以不推荐使用`.

### 模块不需要在分开的文件里

模块让我们可以将一个crate中的代码进行分组，以提高可读性与重用性。模块还可以控制项的私有性，即项是可以被外部代码使用的（public），还是作为一个内部实现的内容，不能被外部代码使用（private）。模块是一个让你组合相关符号的语言结构.

你不需要把他们放在不同的文件下.

下面我们以餐饮业为例，餐馆中会有一些地方被称之为 前台（front of house），还有另外一些地方被称之为 后台（back of house）。前台是招待顾客的地方，在这里，店主可以为顾客安排座位，服务员接受顾客下单和付款，调酒师会制作饮品。后台则是由厨师工作的厨房，洗碗工的工作地点，以及经理做行政工作的地方组成。

我们可以将函数放置到嵌套的模块中，来使我们的 crate 结构与实际的餐厅结构相同。

让我们修改下 src/main.rs 来证明这个观点：

```Rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}

        pub fn seat_at_table() {}
    }
    pub mod serving {
        pub fn take_order() {}

        pub fn server_order() {}

        pub fn take_payment() {}
    }
}

fn main() {
    crate::front_of_house::hosting::add_to_waitlist();
    crate::front_of_house::hosting::seat_at_table();
    crate::front_of_house::serving::take_order();
    crate::front_of_house::serving::server_order();
    crate::front_of_house::serving::take_payment();
}
```

从范围角度，我们项目结构如下：

```Rust
crate  主模块
 └── front_of_house 
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

从文件角度, 主模块和 front_of_house 模块都在同一个文件 src/main.rs 下。

### 模块可以在可分开的文件中

现在, 如果我们如下修改项目：

src/front_of_house.rs

```Rust
pub mod hosting {
    pub fn add_to_waitlist() {}
    pub fn seat_at_table() {}
}
pub mod serving {
    pub fn take_order() {}
    pub fn server_order() {}
    pub fn take_payment() {}
}
```

src/main.rs

```Rust
fn main() {
    crate::front_of_house::hosting::add_to_waitlist();
    crate::front_of_house::hosting::seat_at_table();
    crate::front_of_house::serving::take_order();
    crate::front_of_house::serving::server_order();
    crate::front_of_house::serving::take_payment();
}
```

然而这行不通.

```Rust
   Compiling my-rust v0.1.0 (/Users/sulin/project/rust/my-rust)
error[E0433]: failed to resolve: maybe a missing crate `front_of_house`?
 --> src/main.rs:2:12
  |
2 |     crate::front_of_house::hosting::add_to_waitlist();
  |            ^^^^^^^^^^^^^^ maybe a missing crate `front_of_house`?
```

虽然 src/main.rs 和 src/lib.rs（二进制和库项目）会被 cargo 自动识别为程序入口，其他文件则需要在文件中明确声明。

我们的错误在于仅仅创建了 src/front_of_house.rs 文件，希望 cargo 会在构建时找到它，但事实上并不是这样的。cargo 甚至不会解析它。 cargo check 命令也不会报错，因为 src/front_of_house.rs 现在还不是 crate 源文件的一部分。

为了改正这个错误，可以如下修改 src/main.rs（因为它是项目入口，这是 cargo 已知的）:

```Rust
mod front_of_house {
    include!("front_of_house.rs");
}
// 注意: 这不是符合 rust 风格的写法，仅作 mod 学习用

fn main() {
    crate::front_of_house::hosting::add_to_waitlist();
    crate::front_of_house::hosting::seat_at_table();
    crate::front_of_house::serving::take_order();
    crate::front_of_house::serving::server_order();
    crate::front_of_house::serving::take_payment();
}
```

现在 crate 可以编译和运行了，因为：

我们定义了一个名为 front_of_house 的模块

我们告诉编译器复制/粘贴其他文件（front_of_house.rs）到模块代码块中

参考 include! 文档

但这不是通常导入模块的方式。按照惯例，如果使用不跟随代码块的 mod 指令，效果上述一样。

所以也可以这样写：

```Rust
mod front_of_house;

fn main() {
    crate::front_of_house::hosting::add_to_waitlist();
    crate::front_of_house::hosting::seat_at_table();
    crate::front_of_house::serving::take_order();
    crate::front_of_house::serving::server_order();
    crate::front_of_house::serving::take_payment();
}
```

就是这么简单。但容易混淆之处在于，根据 mod 之后是否有代码块，它可以内联定义模块，或者导入其他文件。

这也解释了为什么在 src/front_of_house.rs 里不用再定义另一个 mod math {}。因为 src/front_of_house.rs 已经在 src/main.rs 中导入，它已经说 src/front_of_house.rs 的代码存在于一个名为 front_of_house 的模块中。

#### 那 use 呢

现在我们几乎了解了 mod，那 use 呢？

use 的唯一目的是将符号带入命名空间，让符号使用更加简短。

特别是，use 永远不会告诉编译器去编译 mod 导入文件之外的其他文件。

在 main.rs/front_of_house.rs 例子中，在 src/main.rs 写下如下语句时：

```Rust
mod front_of_house;
```

我们在主模块导入一个名为 front_of_house 模块，这个模块导出 其他 模块.

从范围角度，结构如下：

```Rust
crate 主模块(我们在这儿)
 └── front_of_house 模块
     ├── hosting 模块
     │   ├── add_to_waitlist 函数
     │   └── seat_at_table 函数
     └── serving 模块
         ├── take_order 函数
         ├── serve_order 函数
         └── take_payment 函数
```

这就是为什么我们要使用 add_to_waitlist 函数时要这样引用 front_of_house::hosting::add_to_waitlist，即从主模块到 add_to_waitlist 函数的正确路径。

`请注意`，如果我们从另一个模块调用 add_to_waitlist，那么 front_of_house::hosting::add_to_waitlist 可能不是有效路径。 然而，add_to_waitlist 有一个更长的添加路径，即 crate::front_of_house::hosting::add_to_waitlist - 它在我们的 crate 中的任何位置都有效（只要 front_of_house 模块保持原样）。

所以，如果我们不想每次都使用 front_of_house::hosting:: 前缀调用 add_to_waitlist 可以用 use 指令：

```Rust
mod front_of_house;
use front_of_house::hosting;
use front_of_house::serving;

fn main() {
    hosting::add_to_waitlist();
    hosting::seat_at_table();
    serving::take_order();
    serving::server_order();
    serving::take_payment();
}
```

### 那 mod.rs 又是什么呢？

好吧，我说谎了 - 我们还没完全了解 mod。

目前，crate 有一个漂亮又扁平的文件结构：

```Rust
src/
    main.rs
    front_of_house.rs
```

其实这不是很有道理的，因为 front_of_house 里有很多小模块，我们可以这样改变它的结构, 让代码结构更清晰:

```Rust
src/
    main.rs
    front_of_house/
        mod.rs
```

就命名空间/范围而言，两种结构都是等价的。我们的新 src/front_of_house/mod.rs 与src/front_of_house.rs具有完全相同的内容， 并且我们的 src/main.rs 完全不变。

事实上，如果我们定义了 front_of_house  模块的子模块， front_of_house/mod.rs 结构更加易于理解。

我们现在的文件结构如下：

```Rust
src/
    main.rs
    front_of_house/
        mod.rs
        serving.rs (新文件!)
        hosting.rs (也是新文件!)
```

概念上而言，命名空间树如下：

```Rust
crate (src/main.rs)
    `front_of_house` 模块 (src/front_of_house/mod.rs)
        `serving` 模块 (src/front_of_house/serving.rs)
        `hosting` 模块 (src/front_of_house/hosting.rs)
```

我们的 src/main.rs 不需要做很大改动 - front_of_house 仍在相同位置。我们只是让它使用 serving 和 hosting：

```Rust
// 保证 front_of_house 在 `./front_of_house.rs` 或 `./front_of_house/mod.rs` 中定义
mod front_of_house;

// 将两个符号带入范围，在 `front_of_house` 模块中保证都已导出
use front_of_house::{hosting, serving};

fn main() {
    hosting::add_to_waitlist();
    hosting::seat_at_table();
    serving::take_order();
    serving::server_order();
    serving::take_payment();
}
```

我们的 src/front_of_house/hosting.rs 正如我们在 front_of_house.rs 模块做的一样：定义一个函数，并用 pub 将其导出。

```Rust
pub fn add_to_waitlist() {}
pub fn seat_at_table() {}
```

类似地，src/front_of_house/serving.rs 文件如下：

```Rust
pub fn take_order() {}
pub fn server_order() {}
pub fn take_payment() {}
```

现在来看 src/front_of_house/mod.rs。我们知道 cargo 知道 front_of_house 这个模块存在， 因为 src/main.rs 中的 mod front_of_house; 语句已将其导入。 但我们需要让 cargo 也知道 hosting 和 serving 模块。

所以我们需要在 src/front_of_house/mod.rs 添加如下语句；

```Rust
pub mod hosting;
pub mod serving;
```

现在 cargo 知晓所有源文件。

crate 能编译成功了.

这样改变后，从 src/front_of_house/mod.rs 角度看，模块结构如下：

```Rust
`front_of_house` 模块
    `hosting` 模块（公开）
        `add_to_waitlist` 函数（公开）
        `seat_at_table` 函数（公开）
    `serving` 模块（公开）
        `take_order` 函数（公开）
        `server_order` 函数（公开）
        `take_payment` 函数（公开）
```

### 结论

希望这些能澄清 rust 的模块和文件, 如果有任何疑问, 可以在社区进行交流.
