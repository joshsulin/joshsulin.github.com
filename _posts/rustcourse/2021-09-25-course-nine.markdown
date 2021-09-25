---
layout: post
title:  "第八课: 利用Tokio实现一个高性能Mini Http Server"
date:   2021-09-12 20:19:20 +0800
categories: Rust
tags: ['Rust']
---

大家好, 欢迎大家来参加本次公开课. 

今天的公开课的主题是, 利用Tokio实现一个高性能Mini Http Server. Rust web框架建议选用Axum

公开课正式开始前首先我来简单的介绍一下自己.

我叫苏林, 是一名从事于互联网研发的程序员, 也是一名技术爱好者, 在互联网行业有十余年, 先后效力于电商、SaaS领域, 对底层系统级开发比较感兴趣, 也才促使我学习和探索Rust语言. 

现在开始今天正式的公开课, 今天的公开课的按以下几部份来进行分享.

1、高性能Http Server实战课程开篇
=> 主要跟大家聊一聊, 接下来几次公开课的安排. 会以短链服务为背景, 如何一步一步的来实现.

2、回顾Tokio是什么
=> 因为高性能Http Server离不开Tokio, 需要借助于异步框架Tokio来构建.

3、利用Tokio实现一个Mini Http Server
=> 通过带领大家实操来实现一个简单版的Http Server, 了解Http Server的本质是什么? 

4、压测Mini Http Server
=> 对比Java, 选用基于Netty的SprintBoot框架来进行压测, 让大家感受Rust异步框架Tokio的性能.

5、Rust Web框架生态介绍及推荐使用Axum框架
=> 这一小节会给大家聊一聊为何会使用Web框架, 以及Rust web框架生态如何, 在后面的公开课里, 会采用Axum框架来进行后面的开发.

期望公开课达到的目的.
1、实战: 学会从0开始设计架构短链服务.
2、Http Server的本质是什么.
3、了解Rust Web框架生态, 为我们后面技术选型提供一些参考.

接下来开始第一部分 "高性能Http Server实战课程开篇"

讲一讲接下来公开课的安排, 介绍短链服务

