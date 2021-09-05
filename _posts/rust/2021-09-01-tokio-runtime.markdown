---
layout: post
title:  "Tokio运行时介绍"
date:   2021-09-01 22:03:20 +0800
categories: Rust
tags: ['Rust']
---

```Rust
#[derive(Debug)]
pub struct Runtime {
		/// Task executor
		kind: Kind,

		/// Handle to runtime, also contains driver handles
		handle: Handle,

		/// Blocking pool handle, used to signal shutdown
		blocking_pool: BlockingPool,
}
```


