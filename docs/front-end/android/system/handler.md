# Android消息机制--Handler

## 1、概述


Android有大量的消息驱动方式来进行交互，比如Android的四剑客`Activity`, `Service`, `Broadcast`, `ContentProvider`的启动过程的交互，都离不开消息机制，Android某种意义上也可以说成是一个以消息驱动的系统。消息机制涉及MessageQueue/Message/Looper/Handler这4个类。

### 1.1 模型

消息机制主要包含：

- **Message**：消息分为硬件产生的消息(如按钮、触摸)和软件生成的消息；
- **MessageQueue**：消息队列的主要功能向消息池投递消息(`MessageQueue.enqueueMessage`)和取走消息池的消息(`MessageQueue.next`)；
- **Handler**：消息辅助类，主要功能向消息池发送各种消息事件(`Handler.sendMessage`)和处理相应消息事件(`Handler.handleMessage`)；
- **Looper**：不断循环执行(`Looper.loop`)，按分发机制将消息分发给目标处理者。

### 1.2 架构图

![](./assets/Main.jpeg)

- **Looper** 有一个MessageQueue消息队列；
- **MessageQueue** 有一组待处理的Message；
- **Message **中有一个用于处理消息的Handler；
- **Handler **中有Looper和MessageQueue。

## 2、Message

每个消息用`Message`表示，`Message`主要包含以下内容：

| 数据类型 | 成员变量 | 解释         |
| :------- | :------- | :----------- |
| int      | what     | 消息类别     |
| long     | when     | 消息触发时间 |
| int      | arg1     | 参数1        |
| int      | arg2     | 参数2        |
| Object   | obj      | 消息内容     |
| Handler  | target   | 消息响应方   |
| Runnable | callback | 回调方法     |

创建消息的过程，就是填充消息的上述内容的一项或多项。



`Message` 是个单链表结构，静态变量`sPool`，通过next成员变量，维护一个消息池，静态变量`MAX_POOL_SIZE`代表消息池的可用大小；消息池的默认大小为50。通过`obtain()`方法从消息池中获取消息，`recycle`方法把不再使用的消息加入消息池。**这样的好处是，当消息池不为空时，可以直接从消息池中获取Message对象，而不是直接创建，提高效率。**



**obtain**

```java
public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null; //从sPool中取出一个Message对象，并消息链表断开
            m.flags = 0; // 清除in-use flag
            sPoolSize--; //消息池的可用大小进行减1操作
            return m;
        }
    }
    return new Message(); // 当消息池为空时，直接创建Message对象
}
```

**recycle**
```java
public void recycle() {
    if (isInUse()) { //判断消息是否正在使用
        if (gCheckRecycle) { //Android 5.0以后的版本默认为true,之前的版本默认为false.
            throw new IllegalStateException("This message cannot be recycled because it is still in use.");
        }
        return;
    }
    recycleUnchecked();
}

//对于不再使用的消息，加入到消息池
void recycleUnchecked() {
    //将消息标示位置为IN_USE，并清空消息所有的参数。
    flags = FLAG_IN_USE;
    what = 0;
    arg1 = 0;
    arg2 = 0;
    obj = null;
    replyTo = null;
    sendingUid = -1;
    when = 0;
    target = null;
    callback = null;
    data = null;
    synchronized (sPoolSync) {
        if (sPoolSize < MAX_POOL_SIZE) { //当消息池没有满时，将Message对象加入消息池
            next = sPool;
            sPool = this;
            sPoolSize++; //消息池的可用大小进行加1操作
        }
    }
}
```

## 3. MessageQueue

### 3.1 创建MessageQueue

```java
MessageQueue(boolean quitAllowed) {
    mQuitAllowed = quitAllowed;
    //通过native方法初始化消息队列，其中mPtr是供native代码使用
    mPtr = nativeInit();
}
```

!!!note ""

		在整个消息机制中，而`MessageQueue`是连接Java层和Native层的纽带，换言之，Java层可以向MessageQueue消息队列中添加消息，Native层也可以向MessageQueue消息队列中添加消息，`MessageQueue`大部分核心方法都交给native层来处理，其中MessageQueue类中涉及的native方法如下：
	
		```java
		private native static long nativeInit(); //初始化
		private native static void nativeDestroy(long ptr);//清理回收
		private native void nativePollOnce(long ptr, int timeoutMillis);//提取消息队列中的消息
		private native static void nativeWake(long ptr); //唤醒
		private native static boolean nativeIsPolling(long ptr);
		private native static void nativeSetFileDescriptorEvents(long ptr, int fd, int events);
		```
		> 具体参阅[Android消息机制](http://gityuan.com/2015/12/27/handler-message-native/)

### 3.2 添加一条消息到消息队列

**enqueueMessage**

```java
boolean enqueueMessage(Message msg, long when) {
    // 每一个普通Message必须有一个target
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }
    synchronized (this) {
        if (mQuitting) {  //正在退出时，回收msg，加入到消息池
            msg.recycle();
            return false;
        }
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            //p为null(代表MessageQueue没有消息） 或者msg的触发时间是队列中最早的， 则进入该该分支
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked; //当阻塞时需要唤醒
        } else {
            //将消息按时间顺序插入到MessageQueue。一般地，不需要唤醒事件队列，除非
            //消息队头存在barrier，并且同时Message是队列中最早的异步消息。
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p;
            prev.next = msg;
        }
        //消息没有退出，我们认为此时mPtr != 0
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

`MessageQueue`是按照Message触发时间的先后顺序排列的，队头的消息是将要最早触发的消息。当有消息需要加入消息队列时，会从队列头开始遍历，直到找到消息应该插入的合适位置，以保证所有消息的时间顺序。

### 3.3 提取下一条message

**next()**

```java
Message next() {
    final long ptr = mPtr;
    if (ptr == 0) { //当消息循环已经退出，则直接返回
        return null;
    }
    int pendingIdleHandlerCount = -1; // 循环迭代的首次为-1
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        //阻塞操作，当等待nextPollTimeoutMillis时长，或者消息队列被唤醒，都会返回
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            //当消息的Handler为空时，则查询异步消息
            if (msg != null && msg.target == null) {
                //当查询到异步消息，则立刻退出循环
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    //当异步消息触发时间大于当前时间，则设置下一次轮询的超时时长
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 获取一条消息，并返回
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    //设置消息的使用状态，即flags |= FLAG_IN_USE
                    msg.markInUse();
                    return msg;   //成功地获取MessageQueue中的下一条即将要执行的消息
                }
            } else {
                //没有消息
                nextPollTimeoutMillis = -1;
            }
            //消息正在退出，返回null
            if (mQuitting) {
                dispose();
                return null;
            }
            //当消息队列为空，或者是消息队列的第一个消息时
            if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                //没有idle handlers 需要运行，则循环并等待。
                mBlocked = true;
                continue;
            }
            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }
        //只有第一次循环时，会运行idle handlers，执行完成后，重置pendingIdleHandlerCount为0.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; //去掉handler的引用
            boolean keep = false;
            try {
                keep = idler.queueIdle();  //idle时执行的方法
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }
            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }
        //重置idle handler个数为0，以保证不会再次重复运行
        pendingIdleHandlerCount = 0;
        //当调用一个空闲handler时，一个新message能够被分发，因此无需等待可以直接查询pending message.
        nextPollTimeoutMillis = 0;
    }
}
```

`nativePollOnce`是阻塞操作，其中`nextPollTimeoutMillis`代表下一个消息到来前，还需要等待的时长；当nextPollTimeoutMillis = -1时，表示消息队列中无消息，会一直等待下去。

当处于空闲时，往往会执行`IdleHandler`中的方法。当nativePollOnce()返回后，next()从`mMessages`中提取一个消息。

## 4. Looper

### 4.1 prepare()

```java
private static void prepare(boolean quitAllowed) {
    //每个线程只允许执行一次该方法，第二次执行时线程的TLS已有数据，则会抛出异常。
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //创建Looper对象，并保存到当前线程的TLS区域
    sThreadLocal.set(new Looper(quitAllowed));
}
```

Looper.prepare()在每个线程只允许执行一次，该方法会创建Looper对象，Looper的构造方法中会创建一个MessageQueue对象，再将Looper对象保存到当前线程TLS。

### 4.2 loop()

```java
public static void loop() {
    final Looper me = myLooper();  //获取TLS存储的Looper对象 
    final MessageQueue queue = me.mQueue;  //获取Looper对象中的消息队列

    Binder.clearCallingIdentity();
    //确保在权限检查时基于本地进程，而不是调用进程。
    final long ident = Binder.clearCallingIdentity();

    for (;;) { //进入loop的主循环方法
        Message msg = queue.next(); //可能会阻塞 
        if (msg == null) { //没有消息，则退出循环
            return;
        }

        //默认为null，可通过setMessageLogging()方法来指定输出，用于debug功能
        Printer logging = me.mLogging;  
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        msg.target.dispatchMessage(msg); //用于分发Message 
        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        //恢复调用者信息
        final long newIdent = Binder.clearCallingIdentity();
        msg.recycleUnchecked();  //将Message放入消息池 
    }
}
```

loop()进入循环模式，不断重复下面的操作，直到没有消息时退出循环

- 读取MessageQueue的下一条Message；
- 把Message分发给相应的target；
- 再把分发后的Message回收到消息池，以便重复利用。

## 5. Handler

### 5.1 消息发送

![](./assets/java_sendmessage.png)

从上图，可以发现所有的发消息方式，最终都是调用`MessageQueue.enqueueMessage()`;

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis); 
}
```

### 5.2 消息分发机制

```java
 msg.target.dispatchMessage(msg); //msg.target为handler
```

在Looper.loop()中，当发现有消息时，调用消息的目标handler，执行dispatchMessage()方法来分发消息。

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        //当Message存在回调方法，回调msg.callback.run()方法；
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //当Handler存在Callback成员变量时，回调方法handleMessage()；
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //Handler自身的回调方法handleMessage()
        handleMessage(msg);
    }
}
```

> `Handler`是消息机制中非常重要的辅助类，更多的实现都是`MessageQueue`, `Message`中的方法，Handler的目的是为了更加方便的使用消息机制。

## 6 . 总结

最后用一张图，来表示整个消息机制

![handler_java](./assets/handler_java.jpeg)

**图解：**

- Handler通过sendMessage()发送Message到MessageQueue队列；
- Looper通过loop()，不断提取出达到触发条件的Message，并将Message交给target来处理；
- 经过dispatchMessage()后，交回给Handler的handleMessage()来进行相应地处理。
- 将Message加入MessageQueue时，往管道写入字符，可以会唤醒loop线程；如果MessageQueue中没有Message，并处于Idle状态，则会执行IdelHandler接口中的方法，往往用于做一些清理性地工作。

**消息分发的优先级：**

1. Message的回调方法：`message.callback.run()`，优先级最高；
2. Handler的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；
3. Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。

**消息缓存：**

为了提供效率，提供了一个大小为50的Message缓存队列，减少对象不断创建与销毁的过程。



## 7. Handler 在系统中的应用

### 7.1 HandlerThread

HandlerThread 起到的作用就是 **方便了主线程和子线程之间的交互，主线程可以直接通过 Handler 来声明耗时任务并交由子线程来执行** 。使用 HandlerThread 也方便在多个线程间共享，主线程和其它子线程都可以向 HandlerThread 下发任务，且 HandlerThread 可以保证多个任务执行时的有序性

```java
class MainActivity : AppCompatActivity() {

    private val handlerThread = HandlerThread("I am HandlerThread")

    private val handler by lazy {
        object : Handler(handlerThread.looper) {
            override fun handleMessage(msg: Message) {
                Thread.sleep(2000)
                Log.e("MainActivity", "这里是子线程，可以用来执行耗时任务：" + Thread.currentThread().name)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        btn_test.setOnClickListener {
            handler.sendEmptyMessage(1)
        }
        handlerThread.start()
    }

}

```

HandlerThread 是 Thread 的子类，其作用就是为了用来执行耗时任务，其 `run()`方法会自动为自己创建一个 Looper 对象并保存到 `mLooper`，之后就主动开启消息循环，这样 HandlerThread 就会来循环处理 Message 了。

```java
 		public void run() {
        mTid = Process.myTid();
        //触发当前线程创建 Looper 对象
        Looper.prepare();
        synchronized (this) {
            //获取 Looper 对象
            mLooper = Looper.myLooper();
            //唤醒所有处于等待状态的线程
            notifyAll();
        }
        //设置线程优先级
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        //开启消息循环
        Looper.loop();
        mTid = -1;
    }

```



### 7.2 IntentService

ntentService 是系统提供的 Service 子类，用于在后台串行执行耗时任务，在处理完所有任务后会自动停止，不必来手动调用 `stopSelf()` 方法。而且由于IntentService 是四大组件之一，拥有较高的优先级，不易被系统杀死，因此适合用于执行一些高优先级的异步任务

**Google 官方以前也推荐开发者使用 IntentService，但是在 Android 11 中已经被标记为废弃状态了，新版本让考虑使用workmanager或者JobIntentService**

### 7.3 同步屏障

**同步屏障机制是一套为了让某些特殊的消息得以更快被执行的机制**。

这里我们假设一个场景：我们向主线程发送了一个UI绘制操作Message，而此时消息队列中的消息非常多，那么这个Message的处理可能会得到延迟，绘制不及时造成界面卡顿。同步屏障机制的作用，是让这个绘制消息得以越过其他的消息，优先被执行。


正常来说，每一个Message在被插入到MessageQueue中的时候，会强制其`target`属性不能为null。

```java
boolean enqueueMessage(Message msg, long when) {
  // Hanlder不允许为空
  if (msg.target == null) {
      throw new IllegalArgumentException("Message must have a target.");
  }
  ...
}
```

而 **target==null的特殊Message就是同步屏障**

通过`MessageQueue`中`postSyncBarrier`方法可以插入同步屏障

```java
private int postSyncBarrier(long when) {
    synchronized (this) {
        final int token = mNextBarrierToken++;
        final Message msg = Message.obtain();
        msg.markInUse();
        msg.when = when;
        msg.arg1 = token;

        Message prev = null;
        Message p = mMessages;
        // 把当前需要执行的Message全部执行
        if (when != 0) {
            while (p != null && p.when <= when) {
                prev = p;
                p = p.next;
            }
        }
        // 插入同步屏障（没有给Message赋值target属性，且插入到Message队列头部）
        if (prev != null) { 
            msg.next = p;
            prev.next = msg;
        } else {
            msg.next = p;
            mMessages = msg;
        }
        return token;
    }
}
```



MessageQueue在获取下一个Message的时候，如果碰到了同步屏障，那么不会取出这个同步屏障，而是会遍历后续的Message，找到第一个异步消息取出并返回。这里跳过了所有的同步消息，直接执行异步消息。为什么叫同步屏障？因为它可以屏蔽掉同步消息，优先执行异步消息。

```java
Message next() {
    ···
    if (msg != null && msg.target == null) {
        // 同步屏障，找到下一个异步消息
        do {
            prevMsg = msg;
            msg = msg.next;
        } while (msg != null && !msg.isAsynchronous());
    }
    ···
}
```

**注意，同步屏障不会自动移除，使用完成之后需要手动进行移除，不然会造成同步消息无法被处理**。

```java
Message next() {
    ...
    // 阻塞时间
    int nextPollTimeoutMillis = 0;
    for (;;) {
        // 阻塞对应时间 
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // 同步屏障，找到下一个异步消息
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            // 如果上面有同步屏障，但却没找到异步消息，
            // 那么msg会循环到链表尾，也就是msg==null
            if (msg != null) {
                ···
            } else {
                // 没有消息，进入阻塞状态
                nextPollTimeoutMillis = -1;
            }
            ···
        }
    }
}
```

通过`removeSyncBarrier`方法移除同步屏障

```java
public void removeSyncBarrier(int token) {
     synchronized (this) {
         Message prev = null;
         Message p = mMessages;
         //从消息队列找到 target为空,并且token相等的Message
         while (p != null && (p.target != null || p.arg1 != token)) {
             prev = p;
             p = p.next;
         }
         final boolean needWake;
         if (prev != null) {
             prev.next = p.next;
             needWake = false;
         } else {
             mMessages = p.next;
             needWake = mMessages == null || mMessages.target != null;
         }
         p.recycleUnchecked();

         if (needWake && !mQuitting) {
             nativeWake(mPtr);
         }
     }
 }
```



虽然`postSyncBarrier() `和 `removeSyncBarrier`被标记位hide，但是我们还是可以通过反射调用的。而且还可以通过系统调用这个方法添加屏障（在ViewRootImpl中requestLayout时会使用到）之后，我们发送异步消息，悄悄上车。

添加异步消息时有两种办法：

* 使用异步类型的Handler发送的全部Message都是异步的
* 给Message标记异步
  给Message标记异步可以通过它的setAsynchronous()进行，非常简单。

**调用requestLayout()方法之后，并不会马上开始进行绘制任务，而是会给主线程设置一个同步屏幕，并设置Vsync信号监听。当Vsync信号的到来，会发送一个异步消息到主线程Handler，执行我们上一步设置的绘制监听任务，并移除同步屏障。**

```java

//ViewRootImpl.java
@Override
public void requestLayout() {
    if (!mHandlingLayoutInLayoutRequest) {
        checkThread();
        mLayoutRequested = true;
        scheduleTraversals();
    }
}
void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        //插入屏障
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
        //监听Vsync信号，然后发送异步消息 -> 执行绘制任务
        mChoreographer.postCallback(
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        if (!mUnbufferedInputDispatch) {
            scheduleConsumeBatchedInput();
        }
        notifyRendererOfFramePending();
        pokeDrawLockIfNeeded();
    }
}
```

在等待Vsync信号的时候主线程什么事都没干，这样的好处是保证在Vsync信号到来时，绘制任务可以被及时执行，不会造成界面卡顿。但这样也带来了相对应的代价：

- 我们的同步消息最多可能被延迟一帧的时间，也就是16ms，才会被执行
- 主线程Looper造成过大的压力，在VSYNC信号到来之时，才集中处理所有消息

改善这个问题的办法是使用异步消息，发送异步消息之后，即时是在等待Vsync期间也可以执行我们的任务，让我们设置的任务可以更快得被执行（**如有必要才这样搞，UI绘制高于一切**）且减少主线程的Looper压力。

> MessageQueue之所有将同步屏障的接口都变成隐藏接口是不想普通的开发者向主线程队列投递同步屏障影响VSync消息的正常执行，开发过程中尽量不要使用异步消息和同步屏障。

## 参考

[Android消息机制](http://gityuan.com/2015/12/26/handler-message-framework/)

[关于Handler同步屏障你可能不知道的问题](https://juejin.cn/post/6940607471153053704)
