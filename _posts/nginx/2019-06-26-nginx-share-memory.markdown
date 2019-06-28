---
layout: post
title:  "简单梳理基于共享内存实现的lua-resty-lock"
date:   2019-06-28 15:35:15 +0800
categories: OpenResty 
tag: OpenResty
---

### 了解Nginx的进程模型

众所周知, nginx在启动后, 会有`一个master进程`和`多个worker进程`。master进程主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。而基本的网络事件，则是放在worker进程中来处理了。多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求，只可能在一个worker进程中处理，一个worker进程，不可能处理其它进程的请求。worker进程的个数是可以设置的, 一般我们会设置与机器cpu核数一致, 这里面的原因与nginx的进程模型分不开的。nginx的进程模型, 可以由下图来表示:

<a href="/img/post/nginx/1.png" target="_blank"><img src="/img/post/nginx/1.jpg" /></a>

### worker进程之间如何通信

Nginx有多个worker进程, 通过`共享内存`来进行通信.

### 什么是共享内存

顾名思义, `共享内存`就是允许两个不相关的进程访问同一个逻辑内存。共享内存是在两个正在运行的进程之间共享和传递数据的一种非常有效的方式。不同进程之间共享的内存通常安排为同一段物理内存。进程可以将同一段共享内存连接到它们自己的地址空间中, 所有进程都可以访问共享内存中的地址, 就好像它们是由用C语言函数malloc分配的内存一样。而如果某个进程向共享内存写入数据, 所做的改动将立即影响到可以访问同一段共享内存的任何其他进程.

`特别提醒: `共享内存并未提供同步机制, 也就是说, 在第一个进程结束对共享内存的写操作之前, 并无自动机制可以阻止第二个进程开始对它进行读取。所以我们通常需要用其他的机制来同步对共享内存的访问。

`Nginx操作共享内存, 通过锁来保证同步`。这里的锁与下面要提及的lua-resty-lock不是同一回事, 一个是Nginx内部的锁(也叫原子操作), lua-resty-lock 是基于Nginx内部的原子操作来实现的锁.

### lua_shared_dict 的作用

`如何使用`: Nginx作如下配置

```nginx
# 分配一个共享内存
lua_shared_dict dshop_lock 5m;
```

`作用`: 创建全局共享的table（多个worker进程共享）, 存放在 共享内存 里, 数据结果为 红黑树, 使用 自旋锁 来保证同步.

### 基于lua_shared_dict add方法, 用于判断是否获取锁

`lua代码操作共享内存`

```Lua
local dict = ngx.shared['dshop_lock'];
local ok, err = dict:add(key, true, exptime);
```

语法: success, err, forcible = ngx.shared.DICT:add(key, value, exptime?, flags?)

类似`set`方法, 但仅当存储字典 `ngx.shared.DICT` 中 不存在 该key时执行存储 key-value 对。

如果参数 `key` 在字典中己经存在(且没有过期), `success` 返回值为 `false`, 同时 `err` 返回 `"exist"` (己存在).

`基于这个特性, add成功, 表示获取到锁, 不成功表示没有获取到锁.`

### 总结

这篇博客只是简单梳理一下, 如果大家想深入了解, 可以继续分析 `lualib/resty/lock.lua` 这个文件, 在lock方法中还存在很多思想, 由于时间精力能力有限, 不再深入分析。
