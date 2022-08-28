# 为啥Android子线程抛出异常主线程会崩溃

## 1. UncaughtExceptionHandler

UncaughtExceptionHandler是Thread内部的一个接口：

```java
public interface UncaughtExceptionHandler {
        /**
         * Method invoked when the given thread terminates due to the given uncaught exception.
         * 方法在给定线程由于给定未捕获异常而终止时调用。
         * Any exception thrown by this method will be ignored by the Java Virtual Machine.
         * 此方法引发的任何异常都将被Java虚拟机忽略。 
         */
        void uncaughtException(Thread t, Throwable e);
    }

```

### 1.1 官方注释

```shell
Interface for handlers invoked when a Thread abruptly terminates due to an uncaught exception.
When a thread is about to terminate due to an uncaught exception the Java Virtual Machine will query the thread for its UncaughtExceptionHandler using getUncaughtExceptionHandler and will invoke the handler's uncaughtException method,  passing the thread and the exception as arguments. 
If a thread has not had its UncaughtExceptionHandler explicitly set, then its ThreadGroup object acts as its UncaughtExceptionHandler. 
If the ThreadGroup object has no special requirements for dealing with the exception, it can forward the invocation to the default uncaught exception handler.

在线程由于未捕获异常而突然终止时调用的处理程序的接口。
调用时通过查询getUncaughtExceptionHandler 
如果一个线程还没有显式设置它的UncaughtExceptionHandler，那么它的ThreadGroup对象就充当它的UncaughtExceptionHandler。
如果ThreadGroup对象没有处理异常的特殊要求，它可以将调用转发给默认的未捕获异常处理程序。
```

后面两句主要是描述了使用UncaughtExceptionHandler的顺序逻辑。
一般有三种UncaughtExceptionHandler：

|     UncaughtExceptionHandler种类      | 优先级 |
| :-----------------------------------: | :----: |
|  线程私有的UncaughtExceptionHandler   |  最高  |
| ThreadGroup的UncaughtExceptionHandler |  其次  |
|  静态默认的UncaughtExceptionHandler   |  最后  |

如果最后都没有UncaughtExceptionHandler处理这种未捕获的异常，就默认处理,打印错误堆栈异常。

## 2.Java和Android的不同处理

###  2.1 Java测试

设置当前线程的未捕获异常处理器，通过setUncaughtExceptionHandler。然后再线程中抛出异常，测试结果

```java
private static void testSetUncaughtExceptionHandler() {
  Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
      int y = new Random().nextInt(2); //这里有可能 y = 0
      System.out.println(y);
      int x = 10 / y;  //抛出异常
      System.out.println("testChild end");
    }
  });
  thread.setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
      System.out.println("uncaughtException:" + Thread.currentThread() + " " + t + " " + e);
    }
  });
  thread.start();

  try {
    Thread.sleep(2 * 1000);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  System.out.println("Main end " + Thread.currentThread());
}

```

输出结果：

```java
Java getDefaultUncaughtExceptionHandler:null
0
uncaughtException:Thread[Thread-0,5,main] Thread[Thread-0,5,main] java.lang.ArithmeticException: / by zero
Main end Thread[main,5,main]
```

当我们不设置UncaughtExceptionHandler的时候，结果如下：

```java
Java getDefaultUncaughtExceptionHandler:null
0
Exception in thread "Thread-0" java.lang.ArithmeticException: / by zero
	at com.thread.UncaughtExceptionHandlerTest$1.run(UncaughtExceptionHandlerTest.java:22)
	at java.lang.Thread.run(Thread.java:748)
Main end Thread[main,5,main]
```

### 2.2 结论

1. 默认没有UncaughtExceptionHandler的时候，系统会打印出崩溃异常路径

2. 子线程崩溃，main线程是不受影响，继续执行，直到休眠结束。

### 2.3 为啥Android子线程崩溃后，整个进程都跟着崩溃

#### 2.3.1 获取默认UncaughtExceptionHandler

* java

  ```java
  public static void main(String[] args) {
    Thread.UncaughtExceptionHandler uncaughtExceptionHandler = Thread.getDefaultUncaughtExceptionHandler();
    System.out.println("Java getDefaultUncaughtExceptionHandler:" + uncaughtExceptionHandler);
  }
  ```
  
  输出结果：
  
  ```she
  Java getDefaultUncaughtExceptionHandler:null
  ```
  
* Android

  ```java
  public void getDefaultUncaughtExceptionHandler(View view) {
    Thread.UncaughtExceptionHandler uncaughtExceptionHandler = Thread.getDefaultUncaughtExceptionHandler();
    Log.d("Test", "Android getDefaultUncaughtExceptionHandler:" + uncaughtExceptionHandler);
  }
  ```
  
  输出结果
  
  ```shell
  Android getDefaultUncaughtExceptionHandler:com.android.internal.os.RuntimeInit$KillApplicationHandler@45c7407
  ```
  
  > **Java没有默认UncaughtExceptionHandler ，Android有默认的UncaughtExceptionHandler (RuntimeInit$KillApplicationHandler)**

#### 2.3.2 RuntimeInit$KillApplicationHandler

```java
public class RuntimeInit {
    ...
    /**
     * Logs a message when a thread encounters an uncaught exception. By
     * default, {@link KillApplicationHandler} will terminate this process later,
     * but apps can override that behavior.
     */
    private static class LoggingHandler implements Thread.UncaughtExceptionHandler {
        public volatile boolean mTriggered = false;

        @Override
        public void uncaughtException(Thread t, Throwable e) {
            //...打印log信息...
            }
        }
    }

    /**
     * Handle application death from an uncaught exception.  The framework
     * catches these for the main threads, so this should only matter for
     * threads created by applications. Before this method runs, the given
     * instance of {@link LoggingHandler} should already have logged details
     * (and if not it is run first).
     */
    private static class KillApplicationHandler implements Thread.UncaughtExceptionHandler {
        private final LoggingHandler mLoggingHandler;

        /**
         * Create a new KillApplicationHandler that follows the given LoggingHandler.
         * If {@link #uncaughtException(Thread, Throwable) uncaughtException} is called
         * on the created instance without {@code loggingHandler} having been triggered,
         * {@link LoggingHandler#uncaughtException(Thread, Throwable)
         * loggingHandler.uncaughtException} will be called first.
         *
         * @param loggingHandler the {@link LoggingHandler} expected to have run before
         *     this instance's {@link #uncaughtException(Thread, Throwable) uncaughtException}
         *     is being called.
         */
        public KillApplicationHandler(LoggingHandler loggingHandler) {
            this.mLoggingHandler = Objects.requireNonNull(loggingHandler);
        }

        @Override
        public void uncaughtException(Thread t, Throwable e) {
            try {
                ensureLogging(t, e); //打印log
                ...
            } catch (Throwable t2) {
                ...
            } finally {
                // Try everything to make sure this process goes away.
                Process.killProcess(Process.myPid());  // 杀死当前进程
                System.exit(10);
            }
        }
        ......
    }
}
```

我们可以看到在KillApplicationHandler 是的接口中，uncaughtException方法在finally的地方，Kill掉了整个进程。这也就是为啥子线程崩溃整个进程也崩溃的原因。

#### 2.3.3 如何做到Android是子线程崩溃后，整个进程不崩溃

**崩溃处理源码**：

```java
public final void dispatchUncaughtException(Throwable e) {
  Thread.UncaughtExceptionHandler initialUeh =
    Thread.getUncaughtExceptionPreHandler();
  if (initialUeh != null) {
    try {
      initialUeh.uncaughtException(this, e);
    } catch (RuntimeException | Error ignored) {
      // Throwables thrown by the initial handler are ignored
    }
  }
  getUncaughtExceptionHandler().uncaughtException(this, e);
}
```

```java
public UncaughtExceptionHandler getUncaughtExceptionHandler() {
  return uncaughtExceptionHandler != null ?
    uncaughtExceptionHandler : group; //group是线程的ThreadGroup，默认继承自parent
}
```
`ThreadGroup` 实现`UncaughtExceptionHandler`，处理如下

```java
public void uncaughtException(Thread t, Throwable e) {
  if (parent != null) {
    parent.uncaughtException(t, e);
  } else {
    Thread.UncaughtExceptionHandler ueh =
      Thread.getDefaultUncaughtExceptionHandler();
    if (ueh != null) {
      ueh.uncaughtException(t, e); //DefaultUncaughtExceptionHandler处理
    } else if (!(e instanceof ThreadDeath)) {
      System.err.print("Exception in thread \""//log输出
                       + t.getName() + "\" ");
      e.printStackTrace(System.err);
    }
  }
}
```

从源码我们可以知道，要想进程不崩溃，有三种办法：

1. 线程可以设置的私有UncaughtExceptionHandler
2. 设置线程ThreadGroup
3. 设置DefaultUncaughtExceptionHandler（崩溃sdk一般使用这种做法）

## 3. 总结

1. UncaughtExceptionHandler的获取顺序（当前线程-》ThreadGroup -》DefaultUncaughtExceptionHandler）
2. 默认没有UncaughtExceptionHandler的时候，系统会打印出崩溃异常路径
3. Java中子线程崩溃，main线程是不受影响，继续执行，直到休眠结束。
4. Android是子线程崩溃后，整个进程都崩溃
5. Java没有默认UncaughtExceptionHandler ，Android有默认的UncaughtExceptionHandler（RuntimeInit$KillApplicationHandler）
6. Android是子线程崩溃后，整个进程都崩溃，崩溃原因是：KillApplicationHandler中kill掉了整个进程
   

