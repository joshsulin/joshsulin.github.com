---
layout: post
title:  "JDK1.5为什么要引入Future API, 以及它的实现类FutureTask底层源码分析"
date:   2018-11-07 21:14:12 +0800
categories: Java
tag: Java
---

[TOC]

### Future API 定位
在JDK1.5版本中, 引入Future API用于帮助我们可以自由的控制异步任务: 可以通过它来查询异步任务的执行状态, 取消任务, 同时获得异步计算的结果。

### 疑问->难道在JDK1.5之前, 没有提供API可以获取异步计算的结果?
既然存在疑问, 就需要解除疑惑, 用己掌握的知识梳理出为什么在JDK1.5之前, 没有提供API获取异步计算的结果。

#### 第一: 解释什么是异步计算(异步编程)
就是启动一个线程(Thread)执行单独的任务, 不阻塞主线程(Main)的执行

#### 第二: Java中如何启动一个线程用于异步计算
简单的讲, 就是通过Thread类。Thread使用支持两种方式: 
1、继承Thread类, 重写run方法, 新的对象调用start()方法。
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("我将异步执行");
    }
}
MyThread myThread = new MyThread();
myThread.start();
```
2、implements Runnable接口, 实现run()方法, 构造Thread对象, 调用start()方法。
```java
class MyThread implements Runnable {
    public void run() {
        System.out.println("我将异步执行");
    }
}
```
new Thread(new MyThread()).start();

通过上面两种方式可知, 不管哪种实现方式, 都实现了run方法。并且调用thread.start()方法。

#### 第三: Thread的start()方法与run()有什么关系? run方法如何调用的? 明明实现的是run()方法, 为什么调用的是start()方法。

我们知道, new一个Thread, 调用它的start的方法, 就可以创建一个线程, 然后执行该线程需要执行的业务逻辑, 那么run方法是怎么被执行的呢?

一个Java线程的创建本质上就对应了一个本地线程(native thread)的创建, 两者是一一对应的。

关键问题是: 本地线程执行的应该是本地代码, 而Java线程提供的线程函数(run)是Java方法, 编译出的是Java字节码。所以, Java线程其实提供了一个统一的线程函数, 该线程函数通过Java虚拟机调用Java线程方法, 这是通过Java本地方法调用来实现的。

通过源码分析, 调用关系如下:

<a href='/img/post/20181107/01.png' target="_blank"><img src='/img/post/20181107/01.png' /></a>


总结: JVM虚拟机规范约定, 由JVM虚拟机回调run方法。

#### 第四: 通过前面三步, 可以得出一个结论, Java异步任务是通过Thread类来完成的, 具体说是通过 start() 和 run()方法来完成的。

总结: 既然Java异步任务是通过Thread类来完成的, 通过start()方法启动一个线程, 该线程再执行run()方法, 完成相应的业务逻辑执行。start()、run()方法都是void, 肯定是没有返回值的。

### 列举一些场景, 主线程需要依赖子线程执行结果

#### 场景1
假如你突然想做饭，但是没有厨具，也没有食材。网上购买厨具比较方便，食材去超市买更放心。

实现分析: 在快递员送厨具的期间，我们肯定不会闲着，可以去超市买食材。所以，在主线程里面另起一个子线程去网购厨具。

但是，子线程执行的结果是要返回厨具的，而run方法是没有返回值的。所以，这才是难点。

#### 场景2
假如整个家庭成员去体检, 每位家庭成员都启动一个子线程去完成，主线程负责启动子线程以及等待子线程完成, 收集执行结果并进行分析。

......
......
......

这样的场景非常多, 所以JDK1.5提出了Future API规范, 用于获取异步计算的结果.

### Future API规范

```java
public interface Future<V> {
    /**
     * 取消方法
     */
    boolean cancel(boolean mayInterruptIfRunning);
    
    /**
     * 计算是否被取消：如果计算在正常结束前被取消了，则返回true
     */
    boolean isCancelled();
    
    /**
     * 计算是否完成:不管是正常完成、异常结束、还是被取消了，都返回true
     */
    boolean isDone();
    
    /**
     * 检索返回结果，如果计算未完成，则等待任务完成。
     */
    V get() throws InterruptedException, ExecutionException;
    
    /**
     * 和get()方法类似，我们可以通过参数timout和unit指定等待的时间上限，如果时间结束了，计算还未完成，就会抛出TimeOutException异常
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

V是实际计算结果的类型, 也就是get()方法返回的类型。在其中, 一共五个方法。两个状态查询方法(isCancelled和isDone方法), 一个取消计算方法(cancel), 两个检索结果方法 get()和get(long, TimeUnit)方法。

对于己经结束的任务、己经取消过的任务、不能被取消的任务, 调用cancel会失败并返回false; 如果任务还未开始, 调用cancel后, 任务将不会再被执行, 并返回true; 如果任务正在进行中, 参数mayInterruptIfRunning为true, 则中断执行此任务的线程, false 任务则继续执行, 直到完毕。

总结: 到这儿, Future之所以被设计的原因己经很明了了, 其实就是帮助我们可以自由的控制异步任务: 可以通过它来查询异步任务的执行状态, 取消任务, 也可以获得正常的结果。

截止目前, 应该己经阐明了为什么要引入FutureAPI, 以及要解决的问题, 接下来继续分析Future的实现 -> FutureTask

### FutureTask 源码分析

#### 构造函数
```java
// 构造函数一
public FutureTask(Callable<V> callable) {
    if (callable == null) {
        throw new NullPointerException();
    }
    this.callable = callable;
    this.state = NEW;
}
// 构造函数二
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}
```

源码显示, 不管我们使用哪个构造函数, 其内部都是把将传入的参数保存为callable, 并且把状态置为NEW。FutureTask一共声明了7个状态。

```java
// 初始状态
private static final int NEW          = 0;
// 运行中状态, 表示正在设置结束。很短暂的一个状态
private static final int COMPLETING   = 1;
// 正常结束的状态
private static final int NORMAL       = 2;
// 异常状态, 任务异常结束
private static final int EXCEPTIONAL  = 3;
// 任务成功被取消的状态
private static final int CANCELLED    = 4;
// 很短暂的状态, 当在NEW状态下, 调用了cancel(true), 则状态就会转换为INTERRUPTING, 直到执行了Thread#interrupt()方法, 状态转换为INTERRUPTED
private static final int INTERRUPTING = 5;
任务被中断后的状态
private static final int INTERRUPTED  = 6;
```

#### run()方法讲解
FatureTask被创建后, 可以交给Thread调用start()方法, 然后由JVM调用run()方法
```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```
在run()方法中, 先进行状态检查, 检查是否处理NEW状态, 然后将执行线程的引用保存在runner变量中。FutureTask的runner变量用来引用任务执行所在的线程。然后执行Callable的call方法, 进行任务执行。接下来会出现两种情况:

情况一: 如果执行顺利完成, 则调用set(result)的方法。在set()方法中, 先将状态置为COMPLETING, 然后调用finishCompletion()方法, 通知所有等待结果的线程, 并调用done()。

情况二: 如果执行出现了异常。则执行setException()方法。在setException()方法中, 操作基本和set()方法一样, 只是outcome保存的是Throwable。

全局outcome变量
```java
private Object outcome;
```

#### FutureTask对Future接口API的实现

isCancelled() 和 isDone()

```java
public boolean isCancelled() {
    return state >= CANCELLED;
}

public boolean isDone() {
    return state != NEW;
}
```
当前任务的状态保存在全局变量state中。这里检查是否取消和是否完成, 只要检查一下state的值即可。

cancel(boolean)方法

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    if (!(state == NEW && UNSAFE.compareAndSwapInt(this, stateOffset, NEW,mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {
        if(mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally {
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();
    }
}
```

cancel方面会直接检测当前状态是否是NEW, 如果不是, 说明任务己经完成或取消或中断, 所以直接返回。当符合条件后, 检查mayInterruptIfRunning的值.

1、如果mayInterruptIfRunning == false, 则直接将状态设置为 CANCELLED, 并且调用finishCompletion()方法, 通知正在等待结果的线程。

2、如果mayInterruptIfRunning == true, 则暂时将状态设置为INTERRUPTING, 然后试着中断线程, 完成后将状态设置为INTERRUPTED, 最后调用finishCompletion()方法。

get()

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}

private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```

get()方法首先还是进行状态监测, 如果现在正处于NEW和COMPLETING状态, 则会调用awaitDone(), 直到状态转变为其他状态, 然后调用report()方法, 在report()方法中, 首先检测状态: 如果是NORMAL状态, 直接返回保存在outCome中的结果; 如果是CANCELLED、INTERRUPTING、INTERRUPTED状态，则抛出CancellationException()；如果处于其他状态则抛出ExecutionException（比如调用了get(true,atime)方法，时间到期后状态可能还处于NEW状态)。

通过阅读FutureTask的源码, 其实逻辑并不复杂, 但是里面用到了非常多的重要知识, 比如并发情况下, 如何保证线程安全? 如何灵活的在线程生命周期中进行切换? 接下来我们继续探究更深层次的知识。

### FutureTask源码里关于线程安全的知识

#### state 变量

先上一段示例代码片段:

```java
Callable<Chuju> onlineShopping = new Callable<Chuju>() {
    @Override
    public Chuju call() throws Exception {
        System.out.println("第一步: 下单");
        System.out.println("第一步: 等待送货");
        Thread.sleep(5000);
        System.out.println("第一步: 快递送到");
        return new Chuju();
    }
};
FutureTask<Chuju> task = new FutureTask<>(onlineShopping);
new Thread(task).start();
Chuju chuju = task.get();
```

上面这一段代码, 涉及两个线程, 一个是new Thread启动的"Thread-0"线程, 一个是"Main"线程需要执行的 task.get(), 通过之前的源码分析, 它们都是靠一个状态变量来控制 state, 画一个图。

<a href='/img/post/20181107/02.png' target="_blank"><img src='/img/post/20181107/02.png' /></a>

Java内存模型规定了所有的变量都存储在主内存(Main Memory)中, 每条线程还有自己的工作内存(Working Memory, 可与前面讲的处理器高速缓存类比), 线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝, 线程对变量的所有操作(读取、赋值等)都必须在工作内存中进行, 而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量, 线程间变量值的传递均需要通过主内存来完成。如果state是一个普通的成员变量, 肯定存在线程安全问题。

```java
// FutureTask里面定位的state加了volatile关键字
private volatile int state;
```

声明变量是volatile的, JVM保证了每次读变量都从内存中读, 跳过工作内存这一步。当一个线程修改了这个变量的值, volatile保证了新值能立即同步到主内存, 以及每次使用前立即从主内存刷新。volatile的读性能消耗与普通变量几乎相同, 但是写操作稍慢, 因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。

总结: FutureTask中针对全局通用的变量state, 用了volatile关键字来修饰, 保证了线程安全。

### FutureTask源码里关于线程状态的切换

#### finishCompletion()方法, 如何实现通知所有等待结果的线程？

核心思想->使用到了LockSupport这个类, LockSupport为每个线程准备了一个许可, 如果许可可用, 那么park()函数会立即返回, 并且消费这个许可(也就是将许可变得不可用), 如果许可不可用, 那么就会阻塞。而unpark()则使得一个许可变为可用(但是和信号量不同的是, 许可不能累加, 你不可能拥有超过一个许可, 它永远只有一个)。

其实要把这个源码和底层分析清楚, 还需要一篇博客专门来讲LockSupport, 以及与Thread.suspend相比, 为什么LockSupport比Thread.suspend更有优势。以及FutureTask源码里面是如何利用LockSupport.unpark和LockSupport.park来实现通知所有等待结果的线程。敬请期待。
