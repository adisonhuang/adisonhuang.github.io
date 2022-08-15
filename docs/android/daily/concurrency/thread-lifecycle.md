# 线程的生命周期，真的没那么简单

> 摘自 https://developer.huawei.com/consumer/cn/forum/topic/0202779877806640557?fid=0101592429757310384

今天，我们就结合**操作系统线程和编程语言线程**再次深入探讨线程的生命周期问题，线程的生命周期其实没有我们想象的那么简单！！

理解线程的生命周期本质上理解了**生命周期中各个节点的状态转换机制**就可以了。

接下来，我们分别就**通用线程生命周期和Java语言的线程生命周期**分别进行详细说明。

## 通用的线程生命周期

通用的线程生命周期总体上可以分为五个状态：**初始状态、可运行状态、运行状态、休眠状态和终止状态。**

我们可以简单的使用下图来表示这五种状态。

![image.png](./0080086000029274683.20220119151547.60970209371553221864883953456566.png)

### 初始状态

线程已经被创建，但是不允许分配CPU执行。**需要注意的是：这个状态属于编程语言特有，这里指的线程已经被创建，仅仅指在编程语言中被创建，在操作系统中，并没有创建真正的线程。**

### 可运行状态

线程可以分配CPU执行。此时，**操作系统中的线程被成功创建，可以分配CPU执行。**

### 运行状态

当操作系统中存在空闲的CPU，操作系统会将这个空闲的CPU分配给一个处于可运行状态的线程，被分配到CPU的线程的状态就转换成了运行状态

### 休眠状态

运行状态的线程调用一个阻塞的API（例如，以阻塞的方式读文件）或者等待某个事件（例如，等待某个条件变量等），线程的状态就会转换到休眠状态。**此时线程会释放CPU资源，休眠状态的线程没有机会获得CPU的使用权。**一旦等待的条件出现，线程就会从休眠状态转换到可运行状态。

### 终止状态

线程执行完毕或者出现异常就会进入终止状态，终止状态的线程不会切换到其他任何状态，这也意味着**线程的生命周期结束了。**

以上就是通用的线程生命周期，下面，我们再看对比看下Java语言中的线程生命周期。

## Java中的线程生命周期

Java中的线程生命周期总共可以分为六种状态：**初始化状态（NEW）、可运行/运行状态（RUNNABLE）、阻塞状态（BLOCKED）、无时限等待状态（WAITING）、有时限等待状态（TIMED_WAITING）、终止状态（TERMINATED）。**

**需要大家重点理解的是：虽然Java语言中线程的状态比较多，但是，其实在操作系统层面，Java线程中的阻塞状态（BLOCKED）、无时限等待状态（WAITING）、有时限等待状态（TIMED_WAITING）都是一种状态，即通用线程生命周期中的休眠状态。也就是说，只要Java中的线程处于这三种状态时，那么，这个线程就没有CPU的使用权。**

理解了这些之后，我们就可以使用下面的图来简单的表示Java中线程的生命周期。

![image.png](./0080086000029274683.20220119151559.52889579688648851370402326236988.png)

我们也可以这样理解阻塞状态（BLOCKED）、无时限等待状态（WAITING）、有时限等待状态（TIMED_WAITING），它们是导致线程休眠的三种原因！

接下来，我们就看看Java线程中的状态是如何转化的。

### RUNNABLE与BLOCKED的状态转换/

只有一种场景会触发这种转换，就是线程等待synchronized隐式锁。synchronized修饰的方法、代码块同一时刻只允许一个线程执行，其他的线程则需要等待。此时，等待的线程就会从RUNNABLE状态转换到BLOCKED状态。当等待的线程获得synchronized隐式锁时，就又会从BLOCKED状态转换到RUNNABLE状态。

这里，需要大家注意：线程调用阻塞API时，在操作系统层面，线程会转换到休眠状态。但是在JVM中，Java线程的状态不会发生变化，也就是说，Java线程的状态仍然是RUNNABLE状态。JVM并不关心操作系统调度相关的状态，在JVM角度来看，等待CPU使用权（操作系统中的线程处于可执行状态）和等待IO操作（操作系统中的线程处于休眠状态）没有区别，都是在等待某个资源，所以，将其都归入了RUNNABLE状态。

我们平时所说的Java在调用阻塞API时，线程会阻塞，指的是操作系统线程的状态，并不是Java线程的状态。

### RUNNABLE与WAITING状态转换

线程从RUNNABLE状态转换成WAITING状态总体上有三种场景。

**场景一**

获得synchronized隐式锁的线程，调用无参的Object.wait()方法。此时的线程会从RUNNABLE状态转换成WAITING状态。

**场景二**

调用无参数的Thread.join()方法。其中join()方法是一种线程的同步方法。例如，在threadA线程中调用threadB线程的join()方法，则threadA线程会等待threadB线程执行完。而threadA线程在等待threadB线程执行的过程中，其状态会从RUNNABLE转换到WAITING。当threadB执行完毕，threadA线程的状态则会从WAITING状态转换成RUNNABLE状态。

**场景三**

调用LockSupport.park()方法，当前线程会阻塞，线程的状态会从RUNNABLE转换成WAITING。调用LockSupport.unpark(Thread thread)可唤醒目标线程，目标线程的状态又会从WAITING状态转换到RUNNABLE。

### RUNNABLE与TIMED_WAITING状态转换

总体上可以分为五种场景。

**场景一**

调用带超时参数的Thread.sleep(long millis)方法；

**场景二**

获得synchronized隐式锁的线程，调用带超时参数的Object.wait(long timeout)参数；

**场景三**

调用带超时参数的Thread.join(long millis)方法；

**场景四**

调用带超时参数的LockSupport.parkNanos(Object blocker, long deadline)方法；

**场景五**

调用带超时参数的LockSuppor.parkUntil(long deadline)方法。

### 从NEW到RUNNABLE状态

Java刚创建出来的Thread对象就是NEW状态，创建Thread对象主要有两种方法，一种是继承Thread对象，重写run()方法；另一种是实现Runnable接口，重写run()方法。

**注意：这里说的是创建Thread对象的方法，而不是创建线程的方法，创建线程的方法包含创建Thread对象的方法。**

**继承Thread对象**

```java
public class ChildThread extends Thread{
    @Override
    public void run(){
        //线程中需要执行的逻辑
    }
}
//创建线程对象
ChildThread childThread = new ChildThread();复制
```

**实现Runnable接口**

```java
public class ChildRunnable implements Runnable{
    @Override
    public void run(){
        //线程中需要执行的逻辑
    }
}
//创建线程对象
Thread childThread = new Thread(new ChildRunnable());复制
```

**处于NEW状态的线程不会被操作系统调度，因此也就不会执行。Java中的线程要执行，就需要转换到RUNNABLE状态。从NEW状态转换到RUNNABLE状态，只需要调用线程对象的start()方法即可。**

```
//创建线程对象
Thread childThread = new Thread(new ChildRunnable());
//调用start()方法使线程从NEW状态转换到RUNNABLE状态
childThread.start();复制
```

### RUNNABLE到TERMINATED状态

线程执行完run()方法后，或者执行run()方法的时候抛出异常，都会终止，此时为TERMINATED状态。如果我们需要中断run()方法，可以调用interrupt()方法。