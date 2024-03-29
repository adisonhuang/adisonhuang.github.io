# 线程池总结

## 1. 为什么使用线程池

线程池解决的核心问题就是资源管理问题。在并发环境下，系统不能够确定在任意时刻中，有多少任务需要执行，有多少资源需要投入。这种不确定性将带来以下若干问题：

1. 频繁申请/销毁资源和调度资源，将带来额外的消耗，可能会非常巨大。
2. 对资源无限申请缺少抑制手段，易引发系统资源耗尽的风险。
3. 系统无法合理管理内部的资源分布，会降低系统的稳定性。

为解决资源分配这个问题，线程池采用了 **“池化”（Pooling）** 思想。池化，顾名思义，是为了最大化收益并最小化风险，而将资源统一在一起管理的一种思想。

### 1.1 使用线程池的好处：

- **降低资源消耗**：通过池化技术重复利用已创建的线程，降低线程创建和销毁造成的损耗。
- **提高响应速度**：任务到达时，无需等待线程创建即可立即执行。
- **提高线程的可管理性**：线程是稀缺资源，如果无限制创建，不仅会消耗系统资源，还会因为线程的不合理分布导致资源调度失衡，降低系统的稳定性。使用线程池可以进行统一的分配、调优和监控。
- **提供更多更强大的功能**：线程池具备可拓展性，允许开发人员向其中增加更多的功能。比如延时定时线程池ScheduledThreadPoolExecutor，就允许任务延期执行或定期执行。

!!! note ""

    “池化”思想不仅仅能应用在计算机领域，在金融、设备、人员管理、工作管理等领域也有相关的应用。
    在计算机领域中的表现为：统一管理IT资源，包括服务器、存储、和网络资源等等。通过共享资源，使用户在低投入中获益。除去线程池，还有其他比较典型的几种使用策略包括：
    
    1. 内存池(Memory Pooling)：预先申请内存，提升申请内存速度，减少内存碎片。
    2. 连接池(Connection Pooling)：预先申请数据库连接，提升申请连接的速度，降低系统的开销。
    3. 实例池(Object Pooling)：循环使用对象，减少资源在初始化和释放时的昂贵损耗。

   






## 3.线程池使用
### 3.1 创建
我们可以通过`ThreadPoolExecutor`来创建一个线程池。
```java
new ThreadPoolExecutor(corePoolSize, maximumPoolSize,
keepAliveTime, milliseconds,runnableTaskQueue, threadFactory,handler);
```
#### 参数说明
- **corePoolSize（线程池的基本大小）**：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。**如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程**。

- **maximumPoolSize（线程池最大大小）**：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。**值得注意的是如果使用了无界的任务队列这个参数就没什么效果**。

- **keepAliveTime（线程活动保持时间）**：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。

- **TimeUnit（线程活动保持时间的单位）**：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

- **ThreadFactory**：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常又帮助。

- **runnableTaskQueue（任务队列）**：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。

  |         名称          |                             描述                             |
  | :-------------------: | :----------------------------------------------------------: |
  |  ArrayBlockingQueue   | 一个用数组实现的有界阻塞队列，此队列按照先进先出（ FIFO ）的原则对元素进行排序。支持公平锁和非公平锁。 |
  |  LinkedBlockingQueue  | 一个由链表结构组成的有界队列，此队列按照先进先出（ FIFO ）的原则对元素进行排序。此队列的默认长度为 Integer.MAX_VALUE ，所以默认创建的该队列有容量危险。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。 |
  |   SynchronousQueue    | 一个不存储元素的阻塞队列，每一个 put 操作必须等待 take 操作，否则不能添加元素。支持公平锁和非公平锁。  Executors.newCachedThreadPool(）就使用了 SynChronousoueue ，这个线程池根据需要（新任务到来时）创建新的线程，如果有空闲线程则会重复使用，线程空闲了 60 秒后会被回收 |
  | PriorityBlockingQueue |                一个支持线程优先级排序的无界队列，默认自然序进行排序，也可以自定义实现 compareTo(）方法来指定元素排序规则，不能保证同优先级元素的顺序             |

  

- **RejectedExecutionHandler（饱和策略）**：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK提供的四种策略

  |        名称         |                             描述                             |
  | :-----------------: | :----------------------------------------------------------: |
  |  AbortPolicy   |                 丢弃任务并抛出 RejectedExecutionException 异常。这是线程池默认的拒绝策略．在任务不能再提交的时候，抛出异常，及时反馈程序运行状态。如果是比较关键的业务．推荐使用此拒绝策略，这样子在系统不能承载更大的并发量的时候，能够及时的通过异常发现                |
  |  CallerRunsPolicy   |                 由调用线程（提交任务的线程）处理该任务。这种情况是需要让所有任务都执行完毕，那么就适合大量计算的任务类型去执行，多线程仅仅是增大吞吐觅的手段，最终必须要让每个任务都执行完毕。            |
  | DiscardOldestPolicy |          弃队列最前面的任务，然后重新提交被拒绝的任务。是否要采用此种拒绝策略，还得根据实际业务是否允许丢弃老任务来认真衡量        |
  |    DiscardPolicy    |                       丢弃任务，但是不抛出异常。使用此策略，可能会使我们无法发现系统的异常状态。建议是一些无关紧要的业务采用此策略                       |
  |     自定义策略      | 根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务 |

### 3.1 提交任务
我们可以使用execute提交的任务，但是execute方法没有返回值，所以无法判断任务知否被线程池执行成功。通过以下代码可知execute方法输入的任务是一个Runnable类的实例。  
```java
threadsPool.execute(new Runnable() {
@Override

public void run() {

// TODO Auto-generated method stub

}

});
```
我们也可以使用submit 方法来提交任务，它会返回一个future,那么我们可以通过这个future来判断任务是否执行成功，通过future的get方法来获取返回值，get方法会阻塞住直到任务完成，而使用get(long timeout, TimeUnit unit)方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。
```java
try {

Object s = future.get();

} catch (InterruptedException e) {

// 处理中断异常

} catch (ExecutionException e) {

// 处理无法执行任务异常

} finally {

// 关闭线程池

executor.shutdown();

}
```
### 3.3 关闭线程池
我们可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，但是它们的实现原理不同，shutdown的原理是只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。shutdownNow的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。shutdownNow会首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。

只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。

## 3. 线程池核心设计与实现
### 3.1 总体设计
线程池在内部实际上构建了一个 **生产者消费者模型，将线程和任务两者解耦，并不直接关联，从而良好的缓冲任务，复用线程**。
线程池的运行主要分成两部分：任务管理、线程管理。

* 任务管理部分充当生产者的角色，当任务提交后，线程池会判断该任务后续的流转：（1）直接申请线程执行该任务；（2）缓冲到队列中等待线程执行；（3）拒绝该任务。
* 线程管理部分是消费者，它们被统一维护在线程池内，根据任务请求进行线程的分配，当线程执行完任务后则会继续获取新的任务去执行，最终当线程获取不到任务的时候，线程就会被回收。

总体运行机制如下
![](./77441586f6b312a54264e3fcf5eebe2663494.png)

### 3.2 任务执行机制

所有任务的调度都是由execute方法完成的，这部分完成的工作是：检查现在线程池的运行状态、运行线程数、运行策略，决定接下来执行的流程，是直接申请线程执行，或是缓冲到队列中执行，亦或是直接拒绝该任务。其执行过程如下：

1. 首先检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。
2. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务。
3. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
5. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

其执行流程如下图所示：

![](./31bad766983e212431077ca8da92762050214.png)

通过源代码来看看是如何实现的
```java
public void execute(Runnable command) {

if (command == null)

throw new NullPointerException();

//如果线程数小于基本线程数，则创建线程并执行当前任务

if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {

//如线程数大于等于基本线程数或线程创建失败，则将当前任务放到工作队列中。

if (runState == RUNNING && workQueue.offer(command)) {

if (runState != RUNNING || poolSize == 0)

ensureQueuedTaskHandled(command);

}

//如果线程池不处于运行中或任务无法放入队列，并且当前线程数量小于最大允许的线程数量，则创建一个线程执行任务。

else if (!addIfUnderMaximumPoolSize(command))

//抛出RejectedExecutionException异常

reject(command); // is shutdown or saturated

}

}
```

### 3.3 工作线程
线程池创建线程时，会将线程封装成工作线程Worker，Worker在执行完任务后，还会无限循环获取工作队列里的任务来执行。我们可以从Worker的run方法里看到这点：
```java
public void run() {

     try {

           Runnable task = firstTask;

           firstTask = null;

            while (task != null || (task = getTask()) != null) {

                    runTask(task);

                    task = null;

            }

      } finally {

             workerDone(this);

      }

}
```



## 5. 业务实践
### 5.3 线程监控

**通过线程池提供的参数进行监控**。线程池里有一些属性在监控线程池的时候可以使用

- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
- largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
- getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不减。
- getActiveCount：获取活动的线程数。

**通过扩展线程池进行监控**。通过继承线程池并重写线程池的beforeExecute，afterExecute和terminated方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。

### 5.2 线程收拢
对于项目本身业务来说，可以要求开发者使用统一的线程池来管理线程；但对于第三方 SDK 中的线程要怎么管理呢？这里我们可以在编译期间 **通过字节码修改的方式将所有创建线程的指令在编译期间替换成自定义的方法调用**，从而达到统一管理的目的。*Android* 创建线程主要是通过以下几种方式：

- `Thread` 及其子类
- `TheadPoolExecutor` 及其子类、`Executors`、`ThreadFactory` 实现类
- `AsyncTask`（已过时，可以不考虑）
- `Timer` 及其子类

字节码修改我们可以使用[ASM](https://asm.ow2.io/)，核心代码如下

**字节码修改**
```java
    private fun MethodInsnNode.transformInvokeSpecial(
        klass: ClassNode,
        method: MethodNode
    ) {
        if (this.name != "<init>") {
            return
        }
        when (this.owner) {
            //java.lang.Thread
            THREAD -> transformThreadInvokeSpecial(klass, method)
            //android.os.HandlerThread
            HANDLER_THREAD -> transformHandlerThreadInvokeSpecial(klass, method)
            //java.util.Timer
            TIMER -> transformTimerInvokeSpecial(klass, method)
            //java.util.concurrent.ThreadPoolExecutor
            THREAD_POOL_EXECUTOR -> transformThreadPoolExecutorInvokeSpecial(klass, method)
        }
    }
    
    
    // transformHandlerThreadInvokeSpecial
    private fun MethodInsnNode.transformHandlerThreadInvokeSpecial(
        klass: ClassNode,
        method: MethodNode,
        init: MethodInsnNode = this
    ) {
        when (this.desc) {
            // HandlerThread(String)
            "(Ljava/lang/String;)V" -> {
                method.instructions.apply {
                    insertBefore(init, LdcInsnNode(makeThreadName(klass.className)))
                    insertBefore(
                        init,
                        MethodInsnNode(
                            Opcodes.INVOKESTATIC,
                            SHADOW_THREAD,
                            "makeThreadName",
                            "(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;",
                            false
                        )
                    )
                }
                context.logger.i(" + $SHADOW_THREAD.makeThreadName(Ljava/lang/String;Ljava/lang/String;) => ${owner}.${name}${desc}: ${klass.name}.${method.name}${method.desc}")
            }
            // HandlerThread(String, int)
            "(Ljava/lang/String;I)V" -> {
                method.instructions.apply {
                    // ..., name, priority => ..., priority, name
                    insertBefore(init, InsnNode(Opcodes.SWAP))
                    // ..., priority, name => ..., priority, name, prefix
                    insertBefore(init, LdcInsnNode(makeThreadName(klass.className)))
                    // ..., priority, name, prefix => ..., priority, name
                    insertBefore(
                        init,
                        MethodInsnNode(
                            Opcodes.INVOKESTATIC,
                            SHADOW_THREAD,
                            "makeThreadName",
                            "(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;",
                            false
                        )
                    )
                    // ..., priority, name => ..., name, priority
                    insertBefore(init, InsnNode(Opcodes.SWAP))
                }
                context.logger.i(" + $SHADOW_THREAD.makeThreadName(Ljava/lang/String;Ljava/lang/String;) => ${owner}.${name}${desc}: ${klass.name}.${method.name}${method.desc}")
            }
        }
    }

```
**hook 入口代码片段**

线程接管之后可以自行做一些定制和优化，当然这里最好通过动态配置来控制，避免一些无法预见的问题。
```java
    public static ScheduledExecutorService newScheduledThreadPool(final int corePoolSize,
                                                                  final ThreadFactory factory,
                                                                  final String name, final String tag) {
        // 1. 直接换成业务线程池
        ScheduledExecutorService hookExecutorService =
            ThreadHookHelper.newScheduledThreadPool(corePoolSize, factory, tag);
        if (hookExecutorService != null) return hookExecutorService;
        if (ThreadHookHelper.allowCoreThreadTimeOut()) {
            final ScheduledThreadPoolExecutor executor =
                new ScheduledThreadPoolExecutor(min(max(1, corePoolSize), MAX_POOL_SIZE),
                    new NamedThreadFactory(factory, name));
            executor.setKeepAliveTime(DEFAULT_KEEP_ALIVE, TimeUnit.MILLISECONDS);
            // 2. 做一些优化：如允许核心线程销毁
            executor.allowCoreThreadTimeOut(true);
            return executor;
        }
        // 3. 重命名线程，方便查找问题：name为对应三方库名称
        return Executors.newScheduledThreadPool(corePoolSize, new NamedThreadFactory(factory, name));
    }

```

## 5.3.  如何合理配置线程池参数？

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：

- 任务的性质：CPU密集型任务，IO密集型任务和混合型任务。

  **任务性质不同的任务可以用不同规模的线程池分开处理**。CPU密集型任务配置尽可能少的线程数量，如配置`Ncpu+1`个线程的线程池。IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如`2*Ncpu`。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数。

- 任务的优先级：高，中和低。

  **优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理**。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

- 任务的执行时间：长，中和短。

  **执行时间不同的任务可以交给不同规模的线程池来处理**，或者也可以使用优先级队列，让执行时间短的任务先执行。

- 任务的依赖性：是否依赖其他系统资源，如数据库连接。

  依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

**建议使用有界队列**，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点。如果我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而使用有界队列可以在队列和线程池全满的时候可以抛出抛弃任务的异常，不会导致系统崩溃，然后我们可以通过监控发现问题并解决。

### 5.3.1 参数动态化
在实践过程中，即使像上面那样做了任务的区分，仍然不能够保证一次计算出来合适的参数，我们可以对于针对不同性能的手机和不同的任务线程池动态下发线程池配置；从而提高线程池的灵活性;

线程池构造参数有8个，但是最核心的是3个：`corePoolSize`、`maximumPoolSize`，`workQueue`，它们最大程度地决定了线程池的任务分配和线程分配策略。考虑到在实际应用中我们获取并发性的场景主要是两种：（1）并行执行子任务，提高响应速度。这种情况下，应该使用同步队列，没有什么任务应该被缓存下来，而是应该立即执行。（2）并行执行大批次任务，提升吞吐量。这种情况下，应该使用有界队列，使用队列去缓冲大批量的任务，队列容量必须声明，防止任务无限制堆积。**所以线程池只需要提供这三个关键参数的配置，并且提供两种队列的选择，就可以满足绝大多数的业务需求**，***Less is More***。

### 参考
[Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
[Java线程池的分析和使用](http://ifeve.com/java-threadpool/)



