# Java 线程池

[toc]  

## 类继承模型

```java
interface Executor{}

interface ExecutorService extends Executor{}

class ThreadPoolExecutor extends AbstractExecutorService{}
```

以上是三个核心类.

## ThreadPoolExecutor类详情

在线程池中真正使用的构造方法

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
> 3. 丢弃队列最早的且未被开始处理的task, 然后重试. 注释解释: 除非 the discarded task executor is shut down. 我理解是线程池状态已经 shut down了(因为shutdown是不再接受新的任务提交了, 但是会继续处理队列中的任务).
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

## ThreadPoolExecutor 细节

```java
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

### 状态变量

- COUNT_BITS

这个是一个常量, `Integer.SIZE`是32, 减去3是29. 这个数值是一个分割线. 其中3位用于表示线程池状态, 29位表示线程池线程数量.

- CAPACITY

`CAPACITY   = (1 << COUNT_BITS) - 1;` 数值是: `000 11111111111111111111111111111` 3个0, 29个1. 最大线程数也就是这么多.

- RUNNING
`111 00000000000000000000000000000`

- SHUTDOWN
`000 00000000000000000000000000000`

- STOP
`001 00000000000000000000000000000`

- TIDYING
`010 00000000000000000000000000000`

- TERMINATED
`011 00000000000000000000000000000`


> 
> RUNNING：正常的状态：接受新的任务，处理等待队列中的任务
> SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务
> STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程
> TIDYING：所有的任务都已经终止，workCount 为 0。线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()
> TERMINATED：terminated() 方法结束后，线程池的状态就会变成这个
>


### 状态转换

```
RUNNING ---shutdown()---> SHUTDOWN
RUNNING ---shutdownNow()---> STOP
SHUTDOWN ---所有任务完成后---> TIDYING
STOP ---执行的线程全部完成---> TIDYING
TIDYING ---terminated()完成后---> TERMINATED
```


### execute方法

这个方法并不是现在执行的意思, 是提交一个任务给到线程池. 具体运行时间需要看具体线程池的状态.

```java
/**
 * Executes the given task sometime in the future.  The task
 * may execute in a new thread or in an existing pooled thread.
 *
 * If the task cannot be submitted for execution, either because this
 * executor has been shutdown or because its capacity has been reached,
 * the task is handled by the current {@code RejectedExecutionHandler}.
 *
 * @param command the task to execute
 * @throws RejectedExecutionException at discretion of
 *         {@code RejectedExecutionHandler}, if the task
 *         cannot be accepted for execution
 * @throws NullPointerException if {@code command} is null
 */
public void execute(Runnable command) {
	// 这里很容易理解, null的任务不需要执行. 抛出NPE
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
    	// 如果小于corePoolSize
    	
    	// 新建线程去执行
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
    	// 如果是正在运行状态的线程池 并且 成功将任务放到队列中的话进入此
    	
    	//二次确认线程池状态 ctl是原子的
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
        	// 如果线程池在二次确认的时候, 发现状态已经变了. 把刚才加进来的任务在移除掉 并且执行reject方法.
            reject(command);
        else if (workerCountOf(recheck) == 0)
        	// 如果现在线程数是0 那么新建一个线程
        	// 1. 为啥不直接addWorker(command, true) ?
        	// 答: 因为已经加到Queue了 不能把command直接扔给线程
        	// 2. 为啥是false
        	// 答: 因为如果corePoolSize设置0, 那么新建会失败. false的时候,线程数量边界是按照max来比的.
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
    	// 如果队列满了, 新建线程执行对比maxPoolSize
    	// 如果还是失败, 执行reject
        reject(command);
}
```

### reject 方法

这个其实就是最开始实例化ThreadPoolExecutor的时候传递进来的策略.

```java
/**
 * Invokes the rejected execution handler for the given command.
 * Package-protected for use by ScheduledThreadPoolExecutor.
 */
final void reject(Runnable command) {
    handler.rejectedExecution(command, this);
}
```


### addWorker 新建线程



```java
/**
 * Checks if a new worker can be added with respect to current
 * pool state and the given bound (either core or maximum). If so,
 * the worker count is adjusted accordingly, and, if possible, a
 * new worker is created and started, running firstTask as its
 * first task. This method returns false if the pool is stopped or
 * eligible to shut down. It also returns false if the thread
 * factory fails to create a thread when asked.  If the thread
 * creation fails, either due to the thread factory returning
 * null, or due to an exception (typically OutOfMemoryError in
 * Thread.start()), we roll back cleanly.
 *
 * @param firstTask the task the new thread should run first (or
 * null if none). Workers are created with an initial first task
 * (in method execute()) to bypass queuing when there are fewer
 * than corePoolSize threads (in which case we always start one),
 * or when the queue is full (in which case we must bypass queue).
 * Initially idle threads are usually created via
 * prestartCoreThread or to replace other dying workers.
 *
 * @param core if true use corePoolSize as bound, else
 * maximumPoolSize. (A boolean indicator is used here rather than a
 * value to ensure reads of fresh values after checking other pool
 * state).
 * @return true if successful
 */
private boolean addWorker(Runnable firstTask, boolean core) {
	// goto
    retry:
    for (;;) {
    	// 乐观锁使用方法
    	// 
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        // 如果线程池状态不对  &&  ! (线程池是SHUTDOWN && 没提交任务,仅仅新建线程 && 队列里有任务在等待) 意思就是线程池不是一个即将关闭的状态.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                // 线程数量校验失败, 太大了或者超过bounds了 就返回失败
                return false;
            if (compareAndIncrementWorkerCount(c))
            	// 成功的话, 直接break掉
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
            	// 状态有问题的话, 外层循环重新走
            	// 说明线程池被其他线程给操作了
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
    	// 新建一个Worker
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // 添加worker到hashset中
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
            	// 启动线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

这个是启动线程的操作. 终点是各种状态检查以及回滚. 整体就是一个很小心的方法 检查->操作->检查 这种.

### Executors 类

这个类是属于线程池的工具类, 都是一些静态方法用于线程池的工具使用.

例如: 

```java
/**
 * Creates a thread pool that reuses a fixed number of threads
 * operating off a shared unbounded queue.  At any point, at most
 * {@code nThreads} threads will be active processing tasks.
 * If additional tasks are submitted when all threads are active,
 * they will wait in the queue until a thread is available.
 * If any thread terminates due to a failure during execution
 * prior to shutdown, a new one will take its place if needed to
 * execute subsequent tasks.  The threads in the pool will exist
 * until it is explicitly {@link ExecutorService#shutdown shutdown}.
 *
 * @param nThreads the number of threads in the pool
 * @return the newly created thread pool
 * @throws IllegalArgumentException if {@code nThreads <= 0}
 */
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```



