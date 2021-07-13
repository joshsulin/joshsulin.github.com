---
layout: post
title:  "解析datafuse中select version()语句执行过程"
date:   2021-07-13 06:40:00 +0800
categories: Datafuse
tags: ['Datafuse', 'Rust']
---

分析方法参考官方文档 [https://datafuse.rs/development/tracing/](https://datafuse.rs/development/tracing/)

一条简单的SQL(select version())在datafuse中经历了如下几个步骤:

### fuse_query::servers::mysql::mysql_handler
第一行日志: `DEBUG fuse_query::servers::mysql::mysql_handler] Received connect from 127.0.0.1:61928`

作用就是接收客户端连接, 这是典型的socket服务器端和客户端通信. 采用Tokio实现, Tokio构建于Rust之上，提供极快的性能，使其成为高性能服务器应用程序的理想选择。

以下这段代码为服务端启动的代码, 启动后, 一直监控接收客户端的连接请求. 不细讲下面这段代码, 后面会开专题给大家聊tokio, 以及如何用tokio来实现类似datafuse里的start方法, 同时带领大家阅读datafuse的源码.

```Rust
async fn start(&self, args: (String, u16)) -> Result<SocketAddr> {
        let abort_registration = self.abort_parts.lock().1.take();
        if let Some(abort_registration) = abort_registration {
            let sessions = self.session_manager.clone();
            let aborted = self.aborted.clone();
            let aborted_notify = self.aborted_notify.clone();
            let rejected_executor = Runtime::with_worker_threads(1)?;

            let (stream, addr) = Self::listener_tcp(&args.0, args.1).await?;

            tokio::spawn(async move {
                let mut listener_stream = Abortable::new(stream, abort_registration);

                loop {
                    match listener_stream.next().await {
                        None => break,
                        Some(Err(error)) => {
                            log::error!("Unexpected error during process accept: {}", error)
                        }
                        Some(Ok(tcp_stream)) => {
                            if let Ok(addr) = tcp_stream.peer_addr() {
                                log::debug!("Received connect from {}", addr);
                            }

                            match sessions.create_session::<Session>() {
                                Err(error) => {
                                    Self::reject_session(tcp_stream, &rejected_executor, error)
                                }
                                Ok(runnable_session) => {
                                    if let Err(error) = runnable_session.start(tcp_stream).await {
                                        log::error!(
                                            "Unexpected error occurred during start session: {:?}",
                                            error
                                        );
                                    };
                                }
                            }
                        }
                    }
                }

                aborted.store(true, Ordering::Relaxed);
                aborted_notify.notify_waiters();
            });

            return Ok(addr);
        }

        Err(ErrorCode::LogicalError("MySQLHandler already running."))
    }
```

### fuse_query::servers::mysql::mysql_interactive_worker
日志 `DEBUG fuse_query::servers::mysql::mysql_interactive_worker] select hello()`

通过查看源码, 这一步主要是创建一个session, 将本次查询放在单独的session上下文里执行. 通过SessionManager来管理封装Session的.

```Rust
pub struct SessionManager {
    conf: Config,
    cluster: ClusterRef,
    datasource: Arc<DataSource>,

    max_mysql_sessions: usize,
    sessions: RwLock<HashMap<String, Arc<Box<dyn ISession>>>>,
    // TODO: remove queries_context.
    queries_context: RwLock<HashMap<String, FuseQueryContextRef>>,

    notifyed: Arc<AtomicBool>,
    aborted_notify: Arc<tokio::sync::Notify>,
}
```

通过SessionManager的结构体, 大体也能看出datafuse的一次query需要的上下文信息.

通过以上两步, 可以得出, datafuse服务端的工作原理, 监控客户端请求, 将每一次请求(查询)都放在一个session里去完成, 相当于fuse_query的壳子. 

接下来的步骤是fuse_query核心流程 `Planner` -> `optimizer` -> `Executor` ......

### fuse_query::sql::plan_parser
日志
```Rust
DEBUG fuse_query::sql::plan_parser: query="select hello()"
[DEBUG sqlparser::parser] parsing expr
[DEBUG sqlparser::parser] prefix: Function(Function { name: ObjectName([Ident { value: "hello", quote_style: None }]), args: [], over: None, distinct: false })
[DEBUG sqlparser::parser] get_next_precedence() EOF
[DEBUG sqlparser::parser] next precedence: 0
```
这一步主要是将 SQL 解析出的语法树转化成逻辑计划，逻辑优化，物理计划. 是fuse_query的核心.

比如: `select database()` 它会解析成这样一个结构.

```Rust
"Projection: database():Utf8\n  Expression: database(default):Utf8 (Before Projection)\n    ReadDataSource: scan partitions: [1], scan schema: [dummy:UInt8], statistics: [read_rows: 1, read_bytes: 1]"
```

如下这段代码, 大体可以看清步骤.

```Rust
pub fn build_from_sql(&self, query: &str) -> Result<PlanNode> {
    tracing::debug!(query);
    DfParser::parse_sql(query).and_then(|(stmts, _)| {
        stmts
            .first()
            .map(|statement| self.statement_to_plan(statement))
            .unwrap_or_else(|| {
                Result::Err(ErrorCode::SyntaxException("Only support single query"))
            })
    })
}
```

### Executor
日志
```Rust
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}: fuse_query::optimizers::optimizer: Before ProjectionPushDown
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}: fuse_query::optimizers::optimizer: After ProjectionPushDown
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}: fuse_query::optimizers::optimizer: Before Scatters
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}: fuse_query::optimizers::optimizer: After Scatters
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}: fuse_query::optimizers::optimizer: Before StatisticsExact
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}: fuse_query::optimizers::optimizer: After StatisticsExact
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}:build: fuse_query::pipelines::processors::pipeline_builder: Received plan:
DEBUG execute{ctx.id="9fd10e99-94b7-49d1-943c-1e078db1a30a"}:build: fuse_query::pipelines::processors::pipeline_builder: Pipeline:
```

通过日志, 可以得出execute的过程是要经过6个执行优化的过程. `Before ProjectionPushDown`、`After ProjectionPushDown`、`Before Scatters`、`After Scatters`、`Before StatisticsExact`、`After StatisticsExact`

接下来这些代码都比较复杂, 其实也不叫复杂, 必要要解相应理论才知道为什么会这样来编写.

有一个好的办法是, 运行程序中相应代码的测试, 来加深对以上过程的理解.
