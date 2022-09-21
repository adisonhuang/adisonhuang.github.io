# Bitamp 内存占用分析
## 1. 前言
阅读本篇之前，先来想一些问题：
* Q1：一张图片的占用空间大小和其内存占用大小有关系吗？
* Q3：一张图片，分别放在drawable-nodpi,drawable-mdpi,drawable-hdpi, drawable-xxhdpi,drawable-xxxhdpi这几个资源目录中，加载后占用内存相同吗？
* Q4：一张图片放在ImageView中，占用内存大小和ImageView的大小有关系吗？

围绕上述问题，我们探讨下Bitmap的内存占用问题。
## 2. 基础概念
这里先要搞清楚 DisplayMetrics 的两个变量，摘录官方文档的解释：

- **density**：The logical density of the display. This is a scaling factor for the Density Independent Pixel unit, where one DIP is one pixel on an approximately 160 dpi screen (for example a 240x320, 1.5”x2” screen), providing the baseline of the system’s display. Thus on a 160dpi screen this density value will be 1; on a 120 dpi screen it would be .75; etc. This value does not exactly follow the real screen size (as given by xdpi and ydpi, but rather is used to scale the size of the overall UI in steps based on gross changes in the display dpi. For example, a 240x320 screen will have a density of 1 even if its width is 1.8”, 1.3”, etc. However, if the screen resolution is increased to 320x480 but the screen size remained 1.5”x2” then the density would be increased (probably to 1.5).
- **densityDpi**：The screen density expressed as dots-per-inch.

简单来说，可以理解为 density 的数值是 1dp=density px；densityDpi 是屏幕每英寸对应多少个点（不是像素点），它们之间公式是 `px = dp * (densityDpi / 160)`，也就是`density = 160/densityDpi`，在 DisplayMetrics 当中，这两个的关系是线性的：

| density(密度)    | 1    | 1.5  | 2    | 3    | 3.5  | 4    |
| :--------- | :--- | :--- | :--- | :--- | :--- | :--- |
| densityDpi(密度值) | 160  | 240  | 320  | 480  | 560  | 640  |
| 资源目录    | mdpi    | hdpi | xhdpi    | xxhdpi    | xxxhdpi  | 4    |
| 代表分辨率 | 320*480  |480*800  | 720*1280  | 1080*1920  | 560  | 640  |

为了不引起混淆，本文所有提到的密度除非特别说明，都指的是 densityDpi，当然如果你愿意，也可以用 density 来说明问题。


