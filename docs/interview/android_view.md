# Android 视图相关
!!! question "说下对Window的理解"
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
!!! question "OpenGL、Vulkan、Skia的区别"
??? note "回答"

!!! question "OpenGL、Vulkan、Skia的区别"
??? note "回答"

!!! question "说下硬件加速"
??? note "回答"

!!! question "requestLayout()、invalidate()与postInvalidate()有什么区别？"
??? note "回答"

!!! question "getWidth()方法和getMeasureWidth()区别呢？为什么有时候用getWidth()或者getMeasureWidth()得到0？"
??? note "回答"


!!! question "getWidth()方法和getMeasureWidth()区别呢？为什么有时候用getWidth()或者getMeasureWidth()得到0？"
??? note "回答"
