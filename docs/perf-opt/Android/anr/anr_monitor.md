# ANR 监控

Android M(6.0) 版本之后，应用侧无法直接通过监听 `data/anr/trace` 文件，监控是否发生 ANR，那么又有什么其它手段去判定 ANR 呢？目前了解到的方案(思路)主要有下面 2 种：

## 1. 主线程 watchdog 机制

核心思想是在应用层定期向主线程设置探测消息，并在异步设置超时监测，如在规定的时间内没有收到发送的探测消息状态更新，则判定可能发生 ANR，为什么是可能发生 ANR？因为还需要进一步从系统服务获取相关数据(可通过`ActivityManagerService.getProcessesInErrorState()`方法获取进程的ANR信息)，进一步判定是否真的发生 ANR。

[ANR-WatchDog](https://github.com/SalomonBrys/ANR-WatchDog/)就是使用该这个方案, 核心代码如下：

```java

private final Handler _uiHandler = new Handler(Looper.getMainLooper());
private final int _timeoutInterval;
private volatile long _tick = 0;
private volatile boolean _reported = false;

private final Runnable _ticker = new Runnable() {
    @Override public void run() {
        _tick = 0;
        _reported = false;
    }
};

@Override
public void run() {
    setName("|ANR-WatchDog|");

    //_timeoutInterval为设定的超时时长
    long interval = _timeoutInterval;
    while (!isInterrupted()) {
        //_tick为标志，主线程执行了下面发送的_ticker这个Runnable, 那么_tick就会被置为0
        boolean needPost = _tick == 0;
        //在子线程里面需要把标志改为非0，待会儿主线程执行了才知道
        _tick += interval;
        if (needPost) {
            //发个消息给主线程
            _uiHandler.post(_ticker);
        }

        //子线程睡一段时间，起来的时候要是标志位_tick没有被改成0，说明主线程太忙了，或者卡顿了，没来得及执行该消息
        try {
            Thread.sleep(interval);
        } catch (InterruptedException e) {
            _interruptionListener.onInterrupted(e);
            return ;
        }

        // If the main thread has not handled _ticker, it is blocked. ANR.
        if (_tick != 0 && !_reported) {
            //noinspection ConstantConditions
            //排除debug的情况
            if (!_ignoreDebugger && (Debug.isDebuggerConnected() || Debug.waitingForDebugger())) {
                Log.w("ANRWatchdog", "An ANR was detected but ignored because the debugger is connected (you can prevent this with setIgnoreDebugger(true))");
                _reported = true;
                continue ;
            }

            //可以自定义一个Interceptor告诉watchDog，当前上下文环境是否可以进行上报
            interval = _anrInterceptor.intercept(_tick);
            if (interval > 0) {
                continue;
            }

            //上报线程堆栈
            final ANRError error;
            if (_namePrefix != null) {
                error = ANRError.New(_tick, _namePrefix, _logThreadsWithoutStackTrace);
            } else {
                error = ANRError.NewMainOnly(_tick);
            }
            //回调
            _anrListener.onAppNotResponding(error);
            interval = _timeoutInterval;
            _reported = true;
        }
    }
}
```

## 2. 监听 SIGNALQUIT 信号

我们在[ANR原理](https://blog.adison.top/perf-opt/Android/anr/anr)提到了虚拟机是通过注册和监听 SIGNALQUIT 信号的方式执行请求的，我们也可以在应用层参考此方式注册相同信号去监听。只要我们能监控到系统发送的SIGQUIT信号，也就能够监控到发生了ANR。

当接收到该信号时，过滤场景，确定是发生用户可感知的 ANR 之后，从 Java 层获取各线程堆栈，或通过反射方式获取到虚拟机内部 Dump 线程堆栈的接口，在内存映射的函数地址，强制调用该接口，并将数据重定向输出到本地。

**该方案从思路上来说优于第一种方案，并且遵循系统信息获取方式，获取的线程信息及虚拟机信息更加全面，但缺点是对性能影响比较大，对于复杂的 App 来说，统计其耗时，部分场景一次 Dump 耗时可能要超过 10S。**

Linux系统提供了两种监听信号的方法，一种是SignalCatcher线程使用的 *sigwait* 方法进行同步、阻塞地监听，另一种是使用 *sigaction* 方法注册signal handler进行异步监听，我们都来试试。

### 2.1 sigwait

```c++
static void *mySigQuitCatcher(void* args) {
    while (true) {
        int sig;
        sigset_t sigSet;
        sigemptyset(&sigSet);
        sigaddset(&sigSet, SIGQUIT);
        sigwait(&sigSet, &sig);
        if (sig == SIGQUIT) {
            //Got SIGQUIT
        }
    }
}
pthread_t pid;
pthread_create(&pid, nullptr, mySigQuitCatcher, nullptr);
pthread_detach(pid);
```

这个时候就有了两个不同的线程 *sigwait* 同一个SIGQUIT，具体会走到哪个呢，我们在 *sigwait* 的文档中找到了这样的描述（ *sigwait* 方法是由 *sigwaitinfo* 方法实现的）

![](./assets/sigwait.png)

原来 **当有两个线程通过 *sigwait* 方法监听同一个信号时，具体是哪一个线程收到信号时不能确定的**。不确定可不行，当然不满足我们的需求

### 2.2  Signal Handler

那我们再试下另一种方法是否可行，我们通过可以sigaction方法，建立一个Signal Handler：

```c++
void signalHandler(int sig, siginfo_t* info, void* uc) {
  if (sig == SIGQUIT) {
    //Got An ANR
  }
}

struct sigaction sa;
sa.sa_sigaction = signalHandler;
sa.sa_flags = SA_ONSTACK | SA_SIGINFO | SA_RESTART;
sigaction(SIGQUIT, &sa, nullptr);
```

建立了Signal Handler之后，我们发现在同时有 *sigwait* 和signal handler的情况下，信号没有走到我们的signal handler而是依然被系统的Signal Catcher线程捕获到了，这是什么原因呢？

原来是Android默认把SIGQUIT设置成了BLOCKED，所以只会响应 *sigwait* 而不会进入到我们设置的handler方法中。我们通过 *pthread_sigmask* 或者 *sigprocmask* 把SIGQUIT设置为UNBLOCK，那么再次收到SIGQUIT时，就一定会进入到我们的handler方法中。需要这样设置：

```c++
sigset_t sigSet;
sigemptyset(&sigSet);
sigaddset(&sigSet, SIGQUIT);
pthread_sigmask(SIG_UNBLOCK, &sigSet, nullptr);
```

最后需要注意，我们通过Signal Handler抢到了SIGQUIT后，原本的Signal Catcher线程中的 *sigwait* 就不再能收到SIGQUIT了，原本的dump堆栈的逻辑就无法完成了，**我们为了ANR的整个逻辑和流程跟原来完全一致，需要在Signal Handler里面重新向Signal Catcher线程发送一个SIGQUIT**：

```c++
int tid = getSignalCatcherThreadId(); //遍历/proc/[pid]目录，找到SignalCatcher线程的tid
tgkill(getpid(), tid, SIGQUIT);
```

> *（如果缺少了重新向SignalCatcher发送SIGQUIT的步骤，AMS就一直等不到ANR进程写堆栈，直到20秒超时后，才会被迫中断，而继续之后的流程。直接的表现就是ANR弹窗非常慢（20秒超时时间），并且/data/anr目录下无法正常生成完整的 ANR Trace文件。）*



## 3. 完善的ANR监控方案

监控到SIGQUIT信号并不等于就监控到了ANR。



### 3.1 误报

**充分非必要条件1：发生ANR的进程一定会收到SIGQUIT信号；但是收到SIGQUIT信号的进程并不一定发生了ANR。**

考虑下面两种情况：

- **其他进程的ANR**：上面提到过，发生ANR之后，发生ANR的进程并不是唯一需要dump堆栈的进程，系统会收集许多其他的进程进行dump，也就是说当一个应用发生ANR的时候，其他的应用也有可能收到SIGQUIT信号。**进一步，我们监控到SIGQUIT时，可能是监听到了其他进程产生的ANR，从而产生误报。**
- **非ANR发送SIGQUIT**：发送SIGQUIT信号其实是很容易的一件事情，开发者和厂商都可以很容易的发送一个SIGQUIT（java层调用 *android.os.Process.sendSignal* 方法；Native层调用 *kill或者tgkill* 方法），**所以我们可能会收到非ANR流程发送的SIGQUIT信号，从而产生误报。**

怎么解决这些误报的问题呢，我重新回到ANR流程开始的地方:

```java
void appNotResponding(String activityShortComponentName, ApplicationInfo aInfo,
        String parentShortComponentName, WindowProcessController parentProcess,
        boolean aboveSystem, String annotation, boolean onlyDumpSelf) {
    //......
    synchronized (mService) {
        if (isSilentAnr() && !isDebugging()) {
            kill("bg anr", ApplicationExitInfo.REASON_ANR, true);
            return;
        }

        // Set the app's notResponding state, and look up the errorReportReceiver
        makeAppNotRespondingLocked(activityShortComponentName,
                annotation != null ? "ANR " + annotation : "ANR", info.toString());

        // show ANR dialog ......
    }
}

private void makeAppNotRespondingLocked(String activity, String shortMsg, String longMsg) {
    setNotResponding(true);
    // mAppErrors can be null if the AMS is constructed with injector only. This will only
    // happen in tests.
    if (mService.mAppErrors != null) {
        notRespondingReport = mService.mAppErrors.generateProcessError(this,
                ActivityManager.ProcessErrorStateInfo.NOT_RESPONDING,
                activity, shortMsg, longMsg, null);
    }
    startAppProblemLocked();
    getWindowProcessController().stopFreezingActivities();
}
```

在ANR弹窗前，会执行到 *makeAppNotRespondingLocked* 方法中，在这里会给发生ANR进程标记一个 *NOT_RESPONDING* 的flag。而这个flag我们可以通过ActivityManager来获取。

```java
private static boolean checkErrorState() {
    try {
        Application application = sApplication == null ? Matrix.with().getApplication() : sApplication;
        ActivityManager am = (ActivityManager) application.getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.ProcessErrorStateInfo> procs = am.getProcessesInErrorState();
        if (procs == null) return false;
        for (ActivityManager.ProcessErrorStateInfo proc : procs) {
            if (proc.pid != android.os.Process.myPid()) continue;
            if (proc.condition != ActivityManager.ProcessErrorStateInfo.NOT_RESPONDING) continue;
            return true;
        }
        return false;
    } catch (Throwable t){
        MatrixLog.e(TAG,"[checkErrorState] error : %s", t.getMessage());
    }
    return false;
}
```

监控到SIGQUIT后，我们在20秒内（20秒是ANR dump的timeout时间）不断轮询自己是否有NOT_RESPONDING对flag，一旦发现有这个flag，那么马上就可以认定发生了一次ANR。

>  *（你可能会想，有这么方便的方法，监控SIGQUIT信号不是多余的吗？直接一个死循环，不断轮训这个flag不就完事了？是的，理论上确实能这么做，但是这么做过于的低效、耗电和不环保外，更关键的是，下面漏报的问题依然无法解决）*

另外，Signal Handler回调的第二个参数siginfo_t，也包含了一些有用的信息，该结构体的第三个字段si_code表示该信号被发送的方法，SI_USER表示信号是通过kill发送的，SI_QUEUE表示信号是通过sigqueue发送的。但在Android的ANR流程中，高版本使用的是sigqueue发送的信号，某些低版本使用的是kill发送的信号，并不统一。

而第五个字段（极少数机型上是第四个字段）si_pid表示的是发送该信号的进程的pid，这里适用几乎所有Android版本和机型的一个条件是：如果发送信号的进程是自己的进程，那么一定不是一个ANR。可以通过这个条件排除自己发送SIGQUIT，而导致误报的情况。

### 3.2 漏报

**充分非必要条件2：进程处于NOT_RESPONDING的状态可以确认该进程发生了ANR。但是发生ANR的进程并不一定会被设置为NOT_RESPONDING状态。**

考虑下面两种情况：

- **后台ANR（SilentAnr）**：之前分析ANR流程我们可以知道，如果ANR被标记为了后台ANR（即SilentAnr），那么杀死进程后就会直接return，并不会走到产生进程错误状态的逻辑。这就意味着，后台ANR没办法捕捉到，而后台ANR的量同样非常大，并且后台ANR会直接杀死进程，对用户的体验也是非常负面的，这么大一部分ANR监控不到，当然是无法接受的。
- **闪退ANR**：除此之外，我们还发现相当一部分机型（例如OPPO、VIVO两家的高Android版本的机型）修改了ANR的流程，即使是发生在前台的ANR，也并不会弹窗，而是直接杀死进程，即闪退。这部分的机型覆盖的用户量也非常大。并且，确定两家今后的新设备会一直维持这个机制。

所以我们需要一种方法，在收到SIGQUIT信号后，能够非常快速的侦查出自己是不是已处于ANR的状态，进行快速的dump和上报。很容易想到，我们可以通过主线程是否处于卡顿状态来判断。那么怎么最快速的知道主线程是不是卡住了呢？**我们反射过主线程Looper的mMessage对象，该对象的when变量，表示的就是当前正在处理的消息入队的时间，我们可以通过when变量减去当前时间，得到的就是等待时间，如果等待时间过长，就说明主线程是处于卡住的状态**，这时候收到SIGQUIT信号基本上就可以认为的确发生了一次ANR：

```java

private static boolean isMainThreadStuck(){
    try {
        MessageQueue mainQueue = Looper.getMainLooper().getQueue();
        Field field = mainQueue.getClass().getDeclaredField("mMessages");
        field.setAccessible(true);
        final Message mMessage = (Message) field.get(mainQueue);
        if (mMessage != null) {
            long when = mMessage.getWhen();
            if(when == 0) {
                return false;
            }
            long time = when - SystemClock.uptimeMillis();
            long timeThreshold = BACKGROUND_MSG_THRESHOLD;
            if (foreground) {
                timeThreshold = FOREGROUND_MSG_THRESHOLD;
            }
            return time < timeThreshold;
        }
    } catch (Exception e){
        return false;
    }
    return false;
}
```

我们通过上面几种机制来综合判断收到SIGQUIT信号后，是否真的发生了一次ANR，最大程度地减少误报和漏报，才是一个比较完善的监控方案。



### 3.3 额外收获：获取ANR Trace

![](./assets/640 (2).png)

回到之前画的ANR流程示意图，Signal Catcher线程写Trace（圈2）也是一个边界，并且是通过socket的write方法来写Trace的，**如果我们能够hook到这里的write，我们甚至就可以拿到系统dump的ANR Trace内容**。这个内容非常全面，包括了所有线程的各种状态、锁和堆栈（包括native堆栈），对于我们排查问题十分有用，**尤其是一些native问题和死锁等问题**。Native Hook我们采用PLT Hook 方案。

```java

int (*original_connect)(int __fd, const struct sockaddr* __addr, socklen_t __addr_length);
int my_connect(int __fd, const struct sockaddr* __addr, socklen_t __addr_length) {
    if (strcmp(__addr->sa_data, "/dev/socket/tombstoned_java_trace") == 0) {
        isTraceWrite = true;
        signalCatcherTid = gettid();
    }
    return original_connect(__fd, __addr, __addr_length);
}

int (*original_open)(const char *pathname, int flags, mode_t mode);
int my_open(const char *pathname, int flags, mode_t mode) {
    if (strcmp(pathname, "/data/anr/traces.txt") == 0) {
        isTraceWrite = true;
        signalCatcherTid = gettid();
    }
    return original_open(pathname, flags, mode);
}

ssize_t (*original_write)(int fd, const void* const __pass_object_size0 buf, size_t count);
ssize_t my_write(int fd, const void* const buf, size_t count) {
    if(isTraceWrite && signalCatcherTid == gettid()) {
        isTraceWrite = false;
        signalCatcherTid = 0;
        char *content = (char *) buf;
        printAnrTrace(content);
    }
    return original_write(fd, buf, count);
}

void hookAnrTraceWrite() {
    int apiLevel = getApiLevel();
    if (apiLevel < 19) {
        return;
    }
    if (apiLevel >= 27) {
        plt_hook("libcutils.so", "connect", (void *) my_connect, (void **) (&original_connect));
    } else {
        plt_hook("libart.so", "open", (void *) my_open, (void **) (&original_open));
    }

    if (apiLevel >= 30 || apiLevel == 25 || apiLevel ==24) {
        plt_hook("libc.so", "write", (void *) my_write, (void **) (&original_write));
    } else if (apiLevel == 29) {
        plt_hook("libbase.so", "write", (void *) my_write, (void **) (&original_write));
    } else {
        plt_hook("libart.so", "write", (void *) my_write, (void **) (&original_write));
    }
}
```

其中有几点需要注意：

- **只Hook ANR流程**：有些情况下，基础库中的connect/open/write方法可能调用的比较频繁，我们需要把hook的影响降到最低。所以我们只会在接收到SIGQUIT信号后（重新发送SIGQUIT信号给Signal Catcher前）进行hook，ANR流程结束后再unhook。
- **只处理Signal Catcher线程open/connect后的第一次write**：除了Signal Catcher线程中的dump trace的流程，其他地方调用的write方法我们并不关心，并不需要处理。例如，dump trace的流程会在在write方法前，系统会先使用connet方法链接一个path为“*/dev/socket/tombstoned_java_trace*”的socket，我们可以hook connect方法，拿到这个socket的name，我们只处理connect这个socket后，相同线程（即Signal Catcher线程）的第一次write，这次write的内容才是我们唯一关心的。
- **Hook点因API Level而不同**：需要hook的write方法在不同的Android版本中，所在的so也不尽相同，不同API Level需要分别处理，hook不同的so和方法。

这个Hook Trace的方案，不仅仅可以用来查ANR问题，任何时候我们都可以手动向自己发送一个SIGQUIT信号，从而hook到当时的Trace。**Trace的内容对于我们排查线程死锁，线程异常，耗电等问题都非常有帮助**。



## 参考

[微信Android客户端的ANR监控方案](https://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=2649288031&idx=1&sn=91c94e16460a4685a9c0c8e1b9c362a6&chksm=8334c9ddb44340cb66e6ce512ca41592fb483c148419737dbe21f9bbc2bfc2f872d1e54d1641&scene=178&cur_album_id=1955379809983741955#rd)

