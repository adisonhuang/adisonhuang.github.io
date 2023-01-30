# Android 四大组件
## Context
!!! question "说说对Context的理解"
??? note "回答"
    - **Context是什么** Context是一个抽象基类。翻译过来就是上下文，也可以理解为环境，顾名思义就是在某一个场景中本身所包含的一些潜在信息，通过它可以获取程序运行的一些环境基础信息。
    - **Context类图关系**  Context下有两个子类，`ContextWrapper`是上下文功能的封装类，而`ContextImpl`则是上下文功能的实现类。ContextWrapper又有三个直接的子类，`ContextThemeWrapper`、`Service`和`Application`。其中，`ContextThemeWrapper`是一个带主题的封装类，而它有一个直接子类就是`Activity`.类图如下
    ``` mermaid
    classDiagram
      Context <|-- ContextWrapper
      Context <|-- ContextImpl
      ContextWrapper <|-- Application
      ContextWrapper <|-- Service
      ContextWrapper <|-- ContextThemeWrapper
     ContextThemeWrapper <|-- Activity
     class ContextWrapper{
        ~mBase Context
        ~attachBaseContext()
        +getBaseContext()
     }
      class Application{
        +LoadedApk: LoadedApk
        ~attach()
     }
      class Service{
        -mApplication: Application
        ~attach()
        +getApplication()
      }

     class ContextThemeWrapper{
        -mTheme:Resources.Theme
        ~attachBaseContext()
     }
      class Activity{
        -mApplication: Application
        ~attach()
        +getApplication()
      }
      class ContextImpl{
        ~mMainThread ActivityThead
        +getApplicationContext()
      }
      class Context{
        +getApplicationInfo()
        +getApplicationContext()
      }
    ```
    - **Context使用特殊场景** 在绝大多数场景下，Activity、Service和Application这三种类型的Context都是可以通用的，因为它们具体Context的功能则是由`ContextImpl`类去实现的。不过有几种场景比较特殊：
        - startActivity
            * 当为`Activity Context`则可直接使用;
            * 当为其他Context, 则必须带上`FLAG_ACTIVITY_NEW_TASK` flags才能使用;
        - Dialog则必须在一个Activity上面弹出（除非是系统级别吐司）
        - 在`Receiver #onReceive`函数中取得的Context不允许执行`bindService()`操作,因为其实际为[ReceiverRestrictedContext](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/ContextImpl.java),其对`bindService`做了限制。
    - **`getApplication()`和`getApplicationContext()`区别** 绝大多数情况下, `getApplication()`和`getApplicationContext()`这两个方法完全一致, 返回值也相同; 那么两者到底有什么区别呢? 
        * 对于`Activity/Service`来说, `getApplication()`和`getApplicationContext()`的返回值完全相同; 除非厂商修改过接口;
        * `BroadcastReceiver`在`onReceive`的过程, 能使用`getBaseContext().getApplicationContext`获取所在Application, 而无法使用`getApplication`;
        * `ContentProvider`能使用`getContext().getApplicationContext()`获取所在Application. 绝大多数情况下没有问题, 但是有可能会出现空指针的问题, 情况如下:**当同一个进程有多个apk的情况下, 对于第二个apk是由provider方式拉起的, 前面介绍过provider创建过程并不会初始化所在application, 此时执行 getContext().getApplicationContext()返回的结果便是NULL. 所以对于这种情况要做好判空.**，参见[理解Android Context](http://gityuan.com/2017/04/09/android_context/#54-getapplicationcontext)
## Activity相关
!!! question "说下Activity的生命周期"
??? note "回答"
    - onCreate()表示Activity 正在创建，常做初始化工作，如setContentView界面资源、初始化数据
    - onStart()表示Activity 正在启动，这时Activity 可见但不在前台，无法和用户交互，可以做一些资源准备工作，如开启轮播图自动无限轮播
    - onResume()表示Activity 获得焦点，此时Activity 可见且在前台并开始活动
    - onPause()表示Activity 正在停止，可做 数据存储、停止动画等操作
    - onStop()表示activity 即将停止，可做稍微重量级回收工作，如取消网络连接、注销广播接收器，暂停轮播图无限轮播等
    - onDestroy()表示Activity 即将销毁，常做回收工作、资源释放
    - onRestart()表示当Activity由后台切换到前台，由不可见到可见时会调用，表示Activity 重新启动


		

!!! question "说一下onSaveInstanceState()和onRestoreInstanceState()"
??? note "回答"
    * Activity的 onSaveInstanceState()和onRestoreInstanceState()并不是生命周期方法，它们不同于onCreate()、onPause()等生命周期方法，它们并不一定会被触发。
	* **什么时候会触发走这两个方法？** 当应用遇到意外情况（如：内存不足、用户直接按Home键）由系统销毁一个Activity，`onSaveInstanceState()` 会被调用。但是当用户主动去销毁一个Activity时，例如在应用中按返回键，`onSaveInstanceState()`就不会被调用。通常onSaveInstanceState()只适合用于保存一些临时性的状态，而onPause()适合用于数据的持久化保存。
	* **onSaveInstanceState()被执行的场景有哪些？**
      * 长按HOME键，选择运行其他的程序时
      * 锁屏时
      * 从activity A中启动一个新的activity时
      * 屏幕方向切换时
    * **onRestoreInstanceState** 只有在activity被系统回收，重新创建activity的情况下才会被调用(如横竖屏切换)。 
    * onCreat()和onRestoreInstanceState()恢复数据区别
      * 因为onSaveInstanceState 不一定会被调用，所以onCreate()里的Bundle参数可能为空，如果使用onCreate()来恢复数据，一定要做非空判断。而onRestoreInstanceState的Bundle参数一定不会是空值，因为它只有在上次activity被回收了才会调用。
      * onRestoreInstanceState是在onStart()之后被调用的。有时候我们需要onCreate()中做的一些初始化完成之后再恢复数据，用onRestoreInstanceState会比较方便。

!!! question "app切换到后台，当前activity会走onDestory方法吗？"
??? note "回答"
    不会走onDestory方法，会先后走onPause和onStop方法。
    
!!! question "activity被系统杀死，是否走destory流程？"
??? note "回答"
    * **主动杀死(从最近任务划掉)** 只会走启动Activity的onDestroy的生命周期，其他activity的onDestroy不会回调到（测试机器：Nexus 6p） 
    * **意外杀死(系统资源不足、强制停止）** 应用只有在进程存活的情况下才会按照正常的生命周期进行执行，如果进程突然被kill掉，相当于System.exit(0); 进程被杀死，根本不会走（activity，fragment）生命周期。只有在进程不被kill掉，正常情况下才会执行ondestory（）方法
    
!!! question "说下Activity的四种启动模式？"
??? note "回答"
    * Activity的四种启动模式
        * **standard标准模式**：每次启动一个Activity就会创建一个新的实例
        * **singleTop栈顶复用模式**：如果新Activity已经位于任务栈的栈顶，就不会重新创建，并回调 onNewIntent(intent) 方法；为防止快速点击时多次startActivity，可以将目标Activity设置为singleTop，如APP接收到多条推送消息，点开不同消息，均由同一实例展示。
        * **singleTask栈内复用模式**：只要该Activity在一个任务栈中存在，都不会重新创建，并回调 onNewIntent(intent) 方法。如果不存在，系统会先寻找是否存在需要的栈，如果不存在该栈，就创建一个任务栈，并把该Activity放进去；如果存在，就会创建到已经存在的栈中，并且会clearTop，即会把栈顶所有Activity清除；常用于主页和登陆页，无论哪种业务场景下再次回到此页，都不应保留之上Activity。**但当我们主页同时也是启动页时，这样会导致无论我们打开多少页面，应用进入后台再启动会直接回到主页。这种情况有两种解决方面：1.增加闪屏页作为启动页，桥接到主页（大部分app做法），2.主页设置为SingleTop，自行建立栈管理系统（这样的好处是减少一个Activity，提高启动速度**
        * **singleInstance单实例模式**：具有此模式的Activity只能单独位于一个任务栈中，且此任务栈中只有唯一一个实例。如APP经常调用的拨打电话、系统通讯录、地图类APP 等页面，不同APP调用此类Activity 时，首次创建实例，之后其他APP只能复用此实例。
        
!!! question "Activity任务栈的作用是什么？同一程序不同的Activity是否可以放在不同的Task任务栈中？"	
??? note "回答"
    * 什么是Activity Stack
        * Activity承担了大量的显示和交互工作，从某种角度上将，我们看见的应用程序就是许多个Activity的组合。为了让这许多 Activity协同工作而不至于产生混乱，Android平台设计了一种堆栈机制用于管理Activity，其遵循先进后出的原则，系统总是显示位于栈 顶的Activity 
    * 什么是Task
        * Task是指将相关的Activity组合到一起，以Activity Stack的方式进行管理，Task实际上是一个Activity栈，通常用户感受的一个Application就是一个Task，但是从根本上讲，一个Task是可以有一个或多个Android Application组成的。（如拍照流程，先跳转到相机应用Activity，当完成拍照功能后再返回当前Activity获取结果，这是一个`task`）。
    * 同一程序不同的Activity是否可以放在不同的Task任务栈中？
        * 可以的。比如：启动模式里有个Singleinstance，可以运行在另外的单独的任务栈里面。用这个模式启动的activity，在内存中只有一份，这样就不会重复的开启。
        * 也可以在激活一个新的activity时候,给intent设置flag，Intent的flag添加`FLAG_ACTIVITY_NEW_TASK`，这个被激活的activity就会在新的task栈里面

        
!!! question "两个Activity之间怎么传递数据？intent和bundle有什么区别？为什么有了intent还要设计bundle？"
??? note "回答"
    * 两个Activity之间怎么传递数据？
        * 基本数据类型可以通过Intent传递数据
        * 把数据封装至intent对象中
        ```java
        Intent intent = new Intent(content, MeActivity.class);
        intent.putExtra("goods_id", goods_id);
        content.startActivity(intent);
        ```
        * 把数据封装至bundle对象中
        * 把bundle对象封装至intent对象中
        ```java
        Bundle bundle = new Bundle();
        bundle.putString("malename", "李志");
        intent.putExtras(bundle);
        startActivity(intent); 
        ```
    * intent和bundle有什么区别？
        * Intent传递数据和Bundle传递数据是一回事，Intent传递时内部还是调用了Bundle。
        ```java
        public @NonNull Intent putExtra(String name, String value) {
            if (mExtras == null) {
                mExtras = new Bundle();
            }
            mExtras.putString(name, value);
            return this;
        }
        ```
    * 为什么有了intent还要设计bundle？
        * Bundle只是一个信息的载体，内部其实就是维护了一个Map<String,Object>。
        * Intent虽然内部也是通过持有一个Bundle来存储信息的，但Intent一方面作为一个高频使用的组件，更关注易用性，提供了可以直接操作Bundle的系列函数`putExtra(xxx)`，另一方面Intent作为一个消息传递对象，负责组件之间的数据通信，相当于组件之间的协议，有其特殊的”协议“语言，如可以使用 `setComponent()、setClass()、setClassName()`设置要启动的组件名称，另外还有`setAction() `、`setData()`、`setflags()`等
     
## Service相关
!!! question " 介绍一下Service，启动Service有几种方式，生命周期是怎样的？"
??? note "回答"
    * Service分为两种
        * 本地服务，属于同一个应用程序，通过startService来启动或者通过bindService来绑定并且获取代理对象。如果只是想开个服务在后台运行的话，直接startService即可，如果需要相互之间进行传值或者操作的话，就应该通过bindService。
        * 远程服务（不同应用程序之间），通过bindService来绑定并且获取代理对象。
    * 对应的生命周期如下：
        * `context.startService()` -> `onCreate()`-> `onStartCommand()`->**Service running**----- 调用`context.stopService()` ->`onDestroy()`->**Service stop**
        * `context.bindService()`->`onCreate()`->`onBind()`->**Service running**----调用`context.unbindService`->`onUnbind()` -> `onDestroy()`->**Service stop**
    * Service生命周期解释
        - onCreate（）：服务第一次被创建时调用
        - onStartComand（）：服务启动时调用
        - onBind（）：服务被绑定时调用
        - onUnBind（）：服务被解绑时调用
        - onDestroy（）：服务停止时调用

!!! question "service如何杀不死?"
??? note "回答"   
    - onStartCommand方法，返回START_STICKY（粘性）当service因内存不足被kill，当内存又有的时候，service又被重新创建
    - 设置优先级，在服务里的ondestory里发送广播 在广播里再次开启这个服务,双进程守护     

!!! question "先start后bind操作service，此时如何做才能回调Service的destory()方法"
??? note "回答" 
    一个服务只要被启动或者被绑定了之后，就会一直处于运行状态，必须要让以上两种条件同时不满足，服务才能被销毁。这种情况下要同时调用`stopService()`和`unbindService()`方法，onDestroy()方法才会执行  

!!! question "是否能在Service进行耗时操作?"
??? note "回答"   
    * 默认情况,如果没有显示的指定service所运行的进程,Service和Activity是运行在当前app所在进程的mainThread(UI主线程)里面。
    * service里面不能执行耗时的操作(网络请求,拷贝数据库,大文件)，在Service里执行耗时操作，有可能出现主线程被阻塞（ANR）的情况  
## BroadcastReceiver相关
!!! question "广播有哪些注册方式？有什么区别？"
??? note "回答"
    - **静态注册**：常驻系统，不受组件生命周期影响，即便应用退出，广播还是可以被接收，耗电、占内存。
    - **动态注册**：非常驻，跟随组件的生命变化，组件结束，广播结束。在组件结束前，需要先移除广播，否则容易造成内存泄漏。
!!! question "广播使用注意事项"
??? note "回答"
    - BroadCastReceiver 的生命周期很短暂，当接收到广播的时候创建，当onReceive()方法结束后销毁
    - 因为BroadCastReceiver的声明周期很短暂，所以不要在广播接收器中去创建子线程做耗时的操作，因为广播接受者被销毁后，这个子进程就会成为空进程，很容易被杀死
    - BroadCastReceiver是运行在主线程的，所以不能直接在BroadCastReceiver中去做耗时的操作，否则就会出现ANR异常 
## ContentProvider相关
!!! question "Android 设计 ContentProvider 的目的是什么呢？"
??? note "回答"
    * 隐藏数据的实现方式，对外提供统一的数据访问接口；
    * 更好的数据访问权限管理。ContentProvider 可以对开发的数据进行权限设置，不同的 URI 可以对应不同的权限，只有符合权限要求的组件才能访问到 ContentProvider 的具体操作。
    * ContentProvider 封装了跨进程共享的逻辑，我们只需要 Uri 即可访问数据。由系统来管理 ContentProvider 的创建、生命周期及访问的线程分配，简化我们在应用间共享数据（ 进程间通信 ）的方式。我们只管通过 ContentResolver 访问 ContentProvider 所提示的数据接口，而不需要担心它所在进程是启动还是未启动。

!!! question "运行在主线程的 ContentProvider 为什么不会影响主线程的 UI 操作?"
??? note "回答"
    * `ContentProvider` 的 `onCreate()` 是运行在 `UI` 线程的，而 `query()` ，`insert()` ，`delete()` ，`update()` 访问实质上是通过`Binder`方式调用，运行在binder线程池，所以调用这个方法并不会阻塞 `ContentProvider` 所在进程的主线程，但可能会阻塞调用者所在的进程的 `UI` 线程！所以，调用 `ContentProvider` 的操作仍然要放在子线程中去做。    