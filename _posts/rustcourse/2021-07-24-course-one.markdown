---
layout: post
title:  "第一课: Rust是什么, 为什么强烈推荐大家学习它!"
date:   2021-07-24 20:19:20 +0800
categories: Rust
tags: ['Rust']
---

2021 年，Stack Overflow 发布了他们的年度开发者报告，报告中一个很有意思的数据是 Rust 一骑绝尘，打败一众编程语言，夺得了`“最受程序员喜爱的编程语言”`这个称号，而且这已经是` Rust 连续第五年蝉联`这个荣誉了。

那么，Rust究竟是一门怎样的语言? 为什么连续五年夺得"最受程序员喜爱的编程语言"称号。

今天由我和大家一起聊聊, 同时欢迎大家来参加我的公开课

今天给大家带来的主题是：认识面向基础架构语言Rust, 推荐大家使用Rust来编写健壮的应用程序.

PPT

首先我来简单的介绍一下自己.

我叫苏林, 是一名从事于互联网研发的程序员, 也是一名技术爱好者, 在互联网行业沉浮十余年, 先后效力于电商、SaaS领域, 对底层系统级开发比较感兴趣, 也才促使我学习和探索Rust语言. 


PPT

我今天想给大家分享的内容一共包含四部分：

第一: Rust诞生背景
	任何一门技术的诞生，它都是为了解决一个问题。那么 Rust 是为了解决什么问题而存在？这是我们面对Rust语言的时候，必须要先搞清楚的一个问题。

第二: Rust语言设计思想
	通过了解Rust语言的设计思想，来看看 Rust 语言如何解决它想解决的那些问题。

第三: Rust语言的发展前景
	我们每个人的精力都有限, 所以选择一门语言来学习, 你很可能会考虑这样一个问题, 前景如何, 

第四: 通过一个Hello world理解Rust语言如何执行
	通过一个Hello world的程序, 带领大家了解Rust代码如何执行.

PPT

我们先来了解第一部分，Rust 诞生背景。

Rust 语言到底想解决什么问题呢？接下来就带领大家一起回顾一下

### Rust诞生背景
PPT

最初，Rust 是 Mozilla 公司员工 Graydon Hoare (格雷顿 霍尔) 的私人项目。Hoare 是一名职业的编程语言工程师，他的日常工作就是给其他的语言开发编译器和工具集，但并不会参与背后的设计工作。老干这种活儿，自然而然的，Hoare 就萌生了开发一门编程语言的想法，这门语言就是 Rust。

2006 年，Hoare 开始在业余时间设计并开发 Rust 的早期版本。任何一门新技术的诞生，都是为了解决某些问题，Rust 也不例外。

自操作系统诞生以来，系统级主流编程语言从汇编语言到 C 到 C++，已经发展了近 50 个年头，但依然存在两个难题：

- 很难编写内存安全的代码；
+ 很难编写线程安全的代码。

这两个难题存在的本质原因是 C/C++ 属于类型不安全的语言，它们薄弱的内存管理机制导致了很多常见的漏洞。其实 20 世纪 80 年代也出现过非常优秀的语言，比如 Ada(ei 达) 语言。Ada 拥有诸多优秀的特性：可以在编译期进行类型检查、没有 GC 式确定性内存管理、内置安全并发模型、无数据竞争、系统级硬实时编程等。但它的性能和同时期的 C/C++ 相比确实是有差距的。那个时代计算资源匮乏，大家追求的是性能。所以，大家都宁愿牺牲安全性来换取性能。这也是 C/C++ 得以普及的原因。

但随着硬件演进的速度不断加快，Hoare 认为，未来的互联网除了关注性能，一定还会高度关注安全性和并发性，以前那些青睐于 C 和 C++ 的设计方式将会不断的发生改变。

基于这样的认知，Hoare 在设计时对 Rust 的期望是：性能可以和 C/C++ 媲美，还能保证安全性，同时可以提供高效的开发效率，代码还得容易维护。

可以说，Rust 诞生之初就是奔着 C/C++ 去的，Hoare 在一次采访中就曾说过：
> 纵观周围，大部分堆栈级的系统代码都是用 C 或者 C++ 编写的，而那正是我们的目标所在。同时，我们的目标人群正是那些纠结的 C/C++ 程序员，实际上就是我们自己。如果你也和我们一样，不断重复地迫使自己因为 C++ 的高效和部署特性而选择它来进行系统级的开发，却又希望可以编写一些更加安全而省心的程序的话，希望我们可以给你一些帮助。

正是因为 Hoare 以这种观点作为基石，才使得今天的`Rust成为了一门同时追求安全、并发和性能的现代系统级编程语言`。

### Rust语言设计思想

为了达成目标，Rust 语言遵循了四条设计哲学：

#### 安全

来看安全。Rust 是静态的，拥有丰富的类型系统和所有权语义模型，保证了内存安全性和线程安全性。 举个例子，C 语言中很容易出现整数溢出，如果被黑客利用，就会出现安全问题，而 Rust 中的每个值都只能被一个所有者拥有，所以 C 语言常遇到的这类问题，对 Rust 来说都不是问题。

同时，借助类型系统的强大，Rust 编译器可以在编译期就对类型进行检查，及时发现内存不安全的问题，使开发者能在编译阶段就将诸多类型错误扼杀于萌芽之中。当然，凡事都有两面性，之前，我一位从 C++ 转向 Rust 的程序员朋友就吐槽过，“C++ 是调试的时候想撞墙，而 Rust 是编译的时候想撞墙。”

#### 并发

来看并发。并发和并行是技术圈内永不会过时的话题，而 Rust 在设计层面就提供了一整套机制来保证并发的安全性。举个例子，数据并发在多线程程序中是一个常见的危险因素，而 Rust 通过所有权模型，非常清晰地定义了一个安全的边界，保证你的代码不会出问题。

#### 高效

来看高效。Rust 没有运行时机制，也没有垃圾回收机制，所以它非常快并且内存效率极高。同时，Rust 可以为关键性能服务提供支持，还可以轻松地与其他语言集成。

#### 零成本抽象

除了安全、并发、高效，Rust 还追求高效开发和性能。

编程语言如果想做到高效开发，就必须拥有一定的抽象表达能力。关于抽象表达能力，最具代表性的语言就是Ruby。Ruby 代码和 Rust 代码的对比示意如代码清单 1-1 所示。

```Ruby
# Ruby代码
5.times{ puts "Hello Ruby"}
2.days.from_now;
```

```Rust
# Rust代码
5.times(|| println!(“Hello Rust”));
2.days().from_now();
```

代码清单 1-1：Ruby 代码和 Rust 代码对比示意

在代码清单 1-1 中，代码第 2 行和第 3 行是 Ruby 代码，分别表示“输出 5 次"Hello Ruby"”和“从现在开始两天之后”，代码的抽象表达能力已经非常接近自然语言。再看第 5 行和第 6 行的 Rust 代码，它和 Ruby 语言的抽象表达能力是不相上下的。

但是 Ruby 的抽象表达能力完全是靠牺牲性能换来的。而 Rust 的抽象是零成本的，Rust 的抽象并不会存在运行时性能开销，这一切都是在编译期完成的。代码清单 1-1 中的迭代 5 次的抽象代码，在编译期会被展开成和手写汇编代码相近的底层代码，所以不存在运行时因为解释这一层抽象而产生的性能开销。对于一门系统级编程语言而言，运行时零成本是非常重要的。这一点，Rust 做到了。Rust 中零成本抽象的基石就是泛型和 trait。

### Rust发展前景

今天的Rust成为了一门同时追求安全、并发和性能的现代系统级编程语言, 正是得益于这些特性，Rust 语言近年来接连获得国内外多家大厂的公开支持。

微软宣布将探索使用 Rust 编程语言作为 C、C++ 和其他语言的替代方案，以此来改善应用程序的安全状况，并进行了一些使用 Rust 重写 Windows 系统组件的实验。

#### 大厂都在使用Rust

- 微软宣布将探索使用 Rust 编程语言作为 C、C++ 和其他语言的替代方案
- AWS 在其开源博客上发文陆续使用 Rust 编写了多款产品, AWS 更是将 Rust 编译器团队负责人纳入麾下，进一步壮大了其内部的 Rust 团队。
- 国内PingCAP公司的TiKV已经快3.0了，使用Rust开发。
- 国内区块链公司秘猿，Nervos公链，也使用Rust开发。
- 知乎搜索引擎用Rust构建。
- 阿里蚂蚁金服时序数据库、淘宝广告推荐算法都已经使用了Rust。
- 腾讯云也在开始布局Rust
- Parity、以太坊区块链。
- Atlassian，在后端使用Rust。
- Dropbox，在前后端均使用了Rust。
- Facebook，使用Rust 重写了源码管理工具。
- Google，在Fuchsia 项目中部分使用了Rust。
- Microsoft，在Azure IoT 网络上部分使用了Rust。
- npm，在其核心服务上使用了Rust。
- RedHat，使用Rust 创建了新的存储系统。
- Reddit，使用Rust 处理评论。
- Twitter，在构建团队中使用Rust。

#### 重点提一下区块链领域

Rust己经成为区块链领域第一开发语言, 越来越多的著名区块链项目已经选择使用Rust作为其开发语言，包括但不限于 Parity, Polkadot, Substrate, Grin, Ethereum classic, Holochain, Cardano-rust, Exonum, Lighthouse, Nimiq, Nervos, Conflux-rust, Codechain, Witnet. Rust语言正在IT工业各个领域快速发展，而由于区块链本身的特质，区块链领域是较早接纳Rust的领域之一。在区块链领域，Rust正以势如破竹之势占领区块链新兴项目市场，很多著名的老项目也在考虑转向使用Rust重写。

### Rust语言如何执行

Rust 从诞生伊始，就考虑到了平台移植性问题。通常编译阶段被分为前端和后端两部分，Rust 作为编译语言，也是这样划分的。Rust 编译器是一个编译前端，它的工作是对代码进行词法分析、语法分析、类型检查、生成中间代码、进行独立于目标机器的优化等工作。使用 LLVM 作为编译器后端代码生成框架，则可以利用 LLVM 兼容多个目标机器的特性，实现跨平台编译和优化等工作。所以，用户在使用 Rust 时，大多数时候无须考虑各个目标机器平台的特有性质，基本上可以做到一次编写，到处运行。而当用户在需要处理跨平台兼容性问题的时候，Rust 也以第三方 crate 的形式提供了诸多辅助。

Rust 源码经过分词和解析，生成 AST（抽象语法树）。然后把 AST 进一步简化处理为 HIR（High-level IR），目的是让编译器更方便地做类型检查。HIR 会进一步被编译为 MIR（Middle IR），这是一种中间表示，它在 Rust1.12 版本中被引入，主要用于以下目的。

缩短编译时间。MIR 可以帮助实现增量编译，当你修改完代码重新编译的时候，编译器只计算更改过的部分，从而缩短了编译时间。

缩短执行时间。MIR 可以在 LLVM 编译之前实现更细粒度的优化，因为单纯依赖 LLVM 的优化粒度太粗，而且 Rust 无法控制，引入 MIR 就增加了更多的优化空间。

更精确的类型检查。MIR 将帮助实现更灵活的借用检查，从而可以提升 Rust 的使用体验。

最终，MIR 会被翻译为 LLVM IR，然后被 LLVM 的处理编译为能在各个平台上运行的目标机器码。

### 本课小结

Rust 的产生看似偶然，其实是必然。未来的互联网注重安全和高性能是必然的趋势。GH 看到了这一点，Mozilla 也看到了这一点，所以两者才能一拍即合，创造出 Rust。

Rust 从 2006 年诞生之日开始，目标就很明确——追求安全、并发和高性能的现代系统级编程语言。

所以，你准备好学习 Rust 了吗？
