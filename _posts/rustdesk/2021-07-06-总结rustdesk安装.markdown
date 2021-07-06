---
layout: post
title:  "Mac下源码编译RustDesk注意事项"
date:   2021-07-06 08:10:20 +0800
categories: RustDesk
tags: ['RustDesk', 'Rust']
---
对于我而言, 一开始不习惯仔仔细细的阅读文档, 直接 git clone、Cargo run '干', 遇到了很多问题. 

以下是我在Mac下源码编译RustDesk所遇到的问题.

### 问题1:
<a href='/img/rustdesk/01.jpg' target="_blank"><img src='/img/rustdesk/01.jpg' /></a>

打开文件 `libs/magnum-opus/build.rs:7:50`, 会看到这行代码.

```rust
let vcpkg_root = std::env::var("VCPKG_ROOT").unwrap();
```

这行代码是读取 VCPKG root 环境变量。如果不存在这个变量，就引起了上面的错，因此请确保按照构建脚本的指示正确地设置了这个变量。

`以下是官方文档明确写了的, 这就是不爱仔细看文档的结果`

<a href='/img/rustdesk/02.jpeg' target="_blank"><img src='/img/rustdesk/02.jpeg' /></a>

### 问题2:

为了解决问题1, 需要安装`vcpkg`包管理器, 要求Mac系统为10.15及以上.

<a href='/img/rustdesk/03.jpeg' target="_blank"><img src='/img/rustdesk/03.jpeg' /></a>

以上是我在Mac下源码编译RustDesk所遇到的2个问题, 希望对大家有所帮忙.
