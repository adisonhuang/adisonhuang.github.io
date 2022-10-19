# Android 视图相关

!!! question "说下对Window的理解"
??? note "回答"
    Window 是一个窗口，是一个抽象的概念，并不真实存在，他只是以 View 的形式存在。每一个 Window 都对应一个 View 和一个 ViewRootImpl，Window 和 View 通过 ViewRootImpl 来建立联系。

    Window 是一个类，他的实现类是 PhoneWindow，他是用来创建我们页面中所需要的 View 的。所以这个 Window 可以称之为 View 的直接管理者。PhoneWindow 中的 DecorView 最终也是附加到 Window(窗口)上面的。

    Window有几种类型：
        * 应用Window：对应一个Activity。
        * 子Window：不能单独存在，需附属特定的父Window。如Dialog。
        * 系统Window： 需申明权限才能创建。如Toast。

!!! question "Activity、View、Window三者之间的关系"
??? note "回答"
    在Activity启动过程其中的attach()方法中初始化了PhoneWindow，而PhoneWindow是Window的唯一实现类，然后Activity通过setContentView将View设置到了PhoneWindow上，而View通过WindowManager的addView()、removeView()、updateViewLayout()对View进行管理。

!!! question "说下对ViewRootImpl的理解"
??? note "回答"
    ViewRootImpl是一个View树的根节点，它是一个View树的管理者，它负责View树的绘制、事件分发、窗口管理等工作。

    它是连接WindowManagerService和DecorView的纽带，View的三大流程（测量（measure），布局（layout），绘制（draw））均通过ViewRoot来完成。
    
    它既非View的子类，也非View的父类，但是，它实现了ViewParent接口，这让它可以作为View的名义上的父视图

!!! question "说下DecorView的理解"
??? note "回答" 
    * 什么是DecorView
        * D DecorView是顶级View，本质就是一个FrameLayout，它可以被认为是Android视图树的根节点视图。
        * 包含了两个部分，标题栏和内容栏, 内容栏id是content，也就是activity中setContentView所设置的部分，最终将布局添加到id为content的FrameLayout中

    * DecorView的职责
        Activity就像个控制器，不负责视图部分。Window像个承载器，装着内部视图。DecorView就是个顶层视图，是所有View的最外层布局。ViewRoot像个连接器，负责沟通，通过硬件的感知来通知视图，进行用户之间的交互。

!!! question "Dialog和Window有什么关系"
??? note "回答" 

!!! question "PopupWindow和Dialog有什么区别"
??? note "回答"    

!!! question "PopupWindow和Dialog有什么区别"
??? note "回答"   

!!! question "说下对View的理解"
??? note "回答"
    View 实际上是一个抽象类，他负责对**渲染、布局以及触摸事件进行抽象**
    
    * **布局抽象**
    布局，是 View 最重要的特性，诸如层级控制、矩形大小、Matrix 变换都属于布局抽象的范畴，布局是渲染、触摸两者的基础。
    * **触摸事件抽象**
    所谓触控，有触才有控，一方面 View 要负责接收触摸事件，另一方面 View 要负责反馈接收到的触摸事件。简单来说就是主要有两个过程，**冒泡、向上递归。**

    冒泡的主要作用是为了找出触摸点所在的 View，在找的过程中会形成响应链，一旦确定好响应者和响应链，触摸的过程就开始了。可以通过重写 `dispatchTouchEvent(MotionEvent event)` 验证。

    冒泡过程完成后，我们会得到响应者 A，紧接着 `touchstart / touchmove / touchend / touchcancel` 事件就会分发到这个响应者身上。 A 会根据事件类型，调用 `onTouchEvent(MotionEvent event)` 或者 `onInterceptTouchEvent(MotionEvent event)` 来处理事件。如果 A 没有处理事件，那么事件就会向上递归，一直到根 View，如果根 View 也没有处理事件，那么事件就会被丢弃。

    * **渲染抽象**
    渲染，是 View 最基本的特性，View 要负责把自己绘制到屏幕上，这个过程就是渲染。一般来说 View 不会直接面向 底层图形库（如OpenGL） 进行封装，而是通过中间层，在 iOS 上，使用的是 `CALayer(CoreGraphics)`，而在 Android 上，使用的是 `Canvas (Skia)`。

!!! question "简单说下View的绘制流程"
??? note "回答"

!!! question "说下MeasureSpec的理解"
??? note "回答"

!!! question "OpenGL、Vulkan、Skia的区别"
??? note "回答"

!!! question "说下硬件加速"
??? note "回答"

!!! question "requestLayout()、invalidate()与postInvalidate()有什么区别？"
??? note "回答"

!!! question "getWidth()方法和getMeasureWidth()区别呢？为什么有时候用getWidth()或者getMeasureWidth()得到0？"
??? note "回答"


!!! question "获得View宽高的办法"
??? note "回答"

!!! question "自定义View的注意事项"
??? note "回答"


