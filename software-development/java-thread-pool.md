# Java 线程池

[toc]  

## 类继承模型

```java
interface Executor{}

interface ExecutorService extends Executor{}

class ThreadPoolExecutor extends AbstractExecutorService{}
```

以上是三个核心类.

最后真正使用的构造方法
```java
/**
 * Creates a new {@code ThreadPoolExecutor} with the given initial
 * parameters.
 *
 * @param corePoolSize the number of threads to keep in the pool, even
 *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
 * @param maximumPoolSize the maximum number of threads to allow in the
 *        pool
 * @param keepAliveTime when the number of threads is greater than
 *        the core, this is the maximum time that excess idle threads
 *        will wait for new tasks before terminating.
 * @param unit the time unit for the {@code keepAliveTime} argument
 * @param workQueue the queue to use for holding tasks before they are
 *        executed.  This queue will hold only the {@code Runnable}
 *        tasks submitted by the {@code execute} method.
 * @param threadFactory the factory to use when the executor
 *        creates a new thread
 * @param handler the handler to use when execution is blocked
 *        because the thread bounds and queue capacities are reached
 * @throws IllegalArgumentException if one of the following holds:<br>
 *         {@code corePoolSize < 0}<br>
 *         {@code keepAliveTime < 0}<br>
 *         {@code maximumPoolSize <= 0}<br>
 *         {@code maximumPoolSize < corePoolSize}
 * @throws NullPointerException if {@code workQueue}
 *         or {@code threadFactory} or {@code handler} is null
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

下面对这几个关键参数使用的解释

> corePoolSize
> 核心线程数, 就是线程池保持的线程数. 空闲了也不会变少

> maximumPoolSize
> 最大线程数

> keepAliveTime & unit
> 当线程数超过了`corePoolSize`之后, 超过的线程保持的空闲时间. 超过后会被释放. 如果线程池中的线程数少于等于核心线程数 `corePoolSize`，那么这些线程不会因为空闲太长时间而被关闭，当然，也可以通过调用 `allowCoreThreadTimeOut(true)`使核心线程数内的线程也可以被回收。 `unit`是`keepAliveTime`参数的单位.

> workQueue
> 任务队列. 只保持通过`execute`方法提交上来的`Runnable`提交的任务.

> threadFactory
> 线程生成类. 会调用`new Thread`来生成线程. 会给线程按照规则起名字.

> handler
> 线程超过了最大线程数和最大队列数量之后的handle方法
> 
> 1. 拒绝并抛异常
```java
/**
 * A handler for rejected tasks that throws a
 * {@code RejectedExecutionException}.
 */
AbortPolicy
```
> 2. 拒绝不抛异常
```java
/**
 * A handler for rejected tasks that silently discards the
 * rejected task.
 */
DiscardPolicy
```
> 3. 丢弃队列最早的且未被开始处理的task, 然后重试. 注释解释: 除非 the discarded task executor is shut down. 我理解是线程池已经把这个task shut down了.
```java
/**
 * A handler for rejected tasks that discards the oldest unhandled
 * request and then retries {@code execute}, unless the executor
 * is shut down, in which case the task is discarded.
 */
DiscardOldestPolicy
```
> 4. 让提交线程的线程来执行该任务
```java
/**
 * A handler for rejected tasks that runs the rejected task
 * directly in the calling thread of the {@code execute} method,
 * unless the executor has been shut down, in which case the task
 * is discarded.
 */
CallerRunsPolicy
```
>
> 以上是几种策略, 也可以自己实现`RejectedExecutionHandler`接口 来自己定义策略.







