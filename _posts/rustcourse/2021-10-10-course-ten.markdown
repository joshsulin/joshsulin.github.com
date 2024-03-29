---
layout: post
title:  "第十课: 使用Redis给短链服务加速"
date:   2021-10-10 20:19:20 +0800
categories: Rust
tags: ['Rust']
---

大家好, 欢迎大家来参加本次公开课. 今天是第10次公开课, 过去两次主要围绕着Rust Web领域相关的实战, 主要以一个简单的短链服务为项目背景来进行实战讲解, 带领大家有信心的将Rust落地, 选用的框架为 auxm+sqlx, 当然还有其它的一些其它框架比如poem、ORM领域的 Diesel、SeaORM.

打开知乎上张汉东老师的文章: https://www.zhihu.com/people/blackanger/posts 可以看一下, 有两篇都对Rust web生态做了比较多的介绍, 大家在技术选择上应该有所帮助

今天的公开课的主题是, 使用Redis给短链服务加速.

今天的公开课的按以下几部份来进行交流.

1、上节课开发的短链服务部署到生产，并进行压测，分析性能瓶颈的思路

2、在项目中使用 Redis 及组件介绍, 让大家熟悉Redis库的使用.

3、使用 Redis 给短链 API 服务加速, 给大家展示实现的代码.

4、压测带 Redis 的短链服务和原来对比, 看看性能提升了多少.

5、大型架构扩展的思路, 业界通用手法是什么, 提供一些解决问题的思路.

期望公开课达到的目的.
1、这个实战也是连续做了3次公开课了, 核心目的希望大家将Rust落地生产环境, 提升大家学习Rust的兴趣, 告别学习Rust始终在一个main文件里面写一些demo.
2、Rust用于web领域开发也是非常便利的.
3、熟悉大型架构优化的通用手法.

接下来开始第一部分 的讲解. "上节课开发的短链服务部署到生产，并进行压测，分析性能瓶颈的思路"

首先我们回顾一下上次公开课的内容.

什么是短链(短地址)服务
-> 将长地址缩短到一个很短的地址, 用户访问这个短地址可以重定向到原本的长地址. 经常在微博、手机短信上看到比较短的URL, 叫你访问.

为了实现短链服务, 我们简单的实现了几个API, 因为实战短链服务不是我们的目的, 主要是以短链服务为项目背景来讲解Rust web领域相关开发, 所以我们实现的短链服务非常简单, 但是麻雀虽小, 但是五脏巨腹, 该用的核心接口还是有的.

接下来看看代码, 看看实现的这些接口.

接口1: /api/create_shortlink post请求, 将长地址转化成短地址, 并存入数据库, 接口的实现比较简单, 将长地址链接存在数据, 用数据库的自增来当成短地址.

接口2: /api/delete_shortlink post请求, 删除短地址记录.

接口3: /api/:id get请求, 302跳转到相应的长地址网址, 通过id查询到长地址, 并进行跳转.

接口4: /api/not_found, 找不到短地址对应的长地址, 跳转的链接.

接下来聊一下, 部署生产环境.

首先购买了一台云主机, 配置信息如下. Rust的环境安装非常简单.

接下来我们就使用Linux自带的ab性能测试工具来压测一下. 简单的看一下ab性能测试工具的用法.

接下来, 我们来思考如何提升性能, 目前是请求进来, 就直接查询数据库(mysql), 我们都知道 MySQL 能承担的并发读写的量是有上限的, 当系统并发达到一定量时, 单台MySQL就很难应付了.

绝大多数据互联网系统, 都使用 MySQL 加上 Redis 这对儿经典的组合来解决这个问题.

接下来在我们系统中引入redis, 来提升系统的性能, 先来明确如何优化接口.

Rust生态中, 选用哪个Redis库. 做为初学者如何来进行选择? 可以在github上面来进行搜索. 经过各种指标的对比, 我们选用 redis-rs

简单介绍一下 redis-rs.

Redis-rs是一个用于Rust的高级Redis库, 它通过一个非常灵活但低级别的API提供了对所有Redis功能的方便访问。它使用了一个可定制的类型转换特性，因此任何操作都可以返回你所期望的类型的结果。这使得开发体验非常愉快。

redis-rs 暴露了两个 API 级别：一个低级和一个高级部分。高层部分并没有暴露 redis 的所有功能，并且可能在如何讲协议方面采取一些自由的做法。API的低级部分允许你在redis层面上表达任何请求。你可以在任何时候在两个API级别之间流畅地切换。

除了上面解释的同步接口外，还存在一个基于future和tokio的异步接口。

大型互联网架构演进通用思路.

首先, 为什么要进行架构升级, 最大要应对的业务场景就是 高并发挑战, 高并发、大流量的背景. 网上也有很多相关的文章, 各大互联网公司的架构演进之路.

通用的优化思路肯定都是一样的, 万变不离其中, 水平扩展和垂直扩展.

现在我们的服务是一台服务器安装所有应用. 












