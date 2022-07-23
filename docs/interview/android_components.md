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
		test
!!! question "如何避免配置改变时Activity重建？优先级低的Activity在内存不足被回收后怎样做可以恢复到销毁前状态"
??? note "回答"
		test
!!! question "app切换到后台，当前activity会走onDestory方法吗？一般在onstop方法里做什么？什么情况会导致app会被杀死？"
??? note "回答"
		test
!!! question "说下Activity的四种启动模式？singleTop和singleTask的区别以及应用场景？"
??? note "回答"
		test
!!! question "任务栈的作用是什么？同一程序不同的Activity是否可以放在不同的Task任务栈中？"	
??? note "回答"
		test
!!! question "两个Activity之间怎么传递数据？intent和bundle有什么区别？为什么有了intent还要设计bundle？"
??? note "回答"
		test