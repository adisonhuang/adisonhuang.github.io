# Bitmap: 从出生到死亡
## 基本概念

* 像素（`Pixel`）：
指可以表现亮度甚至色彩变化的一个点，是构成数字图像的最小单位。像素具有大小相同、明暗和颜色的变化。特点是有固定的位置和特定的颜色值。

* `color depth`、`bit depth`
每个像素RBG若各用8位表示，`bit depth`就是8bit，那么这个像素就用24位表示，`color depth`就是24bit。一个像素`color depth`越深，像素表达的颜色和亮度的位数越多，文件就越大。一个`color depth`中每个`channel`的深度就是`bit depth`。用32位表示一个像素的话，RBG占用24位，还有8位称为`alpha channel`。

* `alpha composite`、`alpha blend`、`alpha channel`
渲染图片的时候，图片有时有很多图层，然后再将多个图层组合起来，这叫做`alpha composite`。在这个过程中，多个图层每个对应像素合成的过程叫做`alpha blend`。透过看到下一图层，就需要记录一些哪里透明、哪里不透明的信息，这些信息就被存在一个`alpha channel`中了。

* 图片分辨率
计算机显示的图像是由像素点组成的，图片尺寸为640 x 480，代表图片水平有640个像素点，垂直有480像素点。

> 各种图片格式目的是在网络传输和存储的时候使用更少的字节，即起到压缩的作用，或者支持多张图片组合成一张动态图。在图片格式解码后，无论哪种图片的格式，图片数据都是像素数组。

* Bitmap
位图，又称为点阵图像、像素图或栅格图像，是由像素（图片元素）的单个点组成。这些点可以进行不同的排列和染色以构成图样。Bitmap的承载容器是`jpg`、`png`等格式的文件，是对bitmap的压缩。当jpg、png等文件需要展示在手机上的控件时，就会解析成Bitmap并绘制到view上。
> Bitmap同时也表示一种数据结构，用一个bit位来标记某个元素对应的Value， 而Key即是该元素。由于采用了Bit为单位来存储数据，因此在存储空间方面，可以大大节省。具体参见[什么是Bit-map](https://wizardforcel.gitbooks.io/the-art-of-programming-by-july/content/06.07.html)


## Android Bitmap
App开发不可避免的要和图片打交道，由于其占用内存非常大，管理不当很容易导致内存不足，最后OOM，图片的背后其实是Bitmap，Bitmap 占内存多是因为其像素数据(pixels)大。Bitmap 像素数据的存储在不同 Android 版本之间有所不同，具体来说

| 版本                         |                           内存分布                           |
| :--------------------------- | :----------------------------------------------------------: |
| ~ Android 2.3.3(API 10)      | 1. 位图的后备像素数据存储在 Native 堆中。Native 内存中的像素数据并不以可预测的方式释放，可能会导致应用短暂超出其内存限制并崩溃。 2. Android 2.2(API 8) 无并发垃圾回收功能, 当发生垃圾回收时，应用的线程会停止。这会导致延迟，从而降低性能。 3. Android 2.3 添加了并发垃圾回收功能，这意味着系统不再引用位图后，很快就会回收内存 |
| 3.0（API 11）~ 7.1（API 25） |      像素数据会与关联的位图一起存储在 Dalvik/ART 堆上。      |
| 8.0（API 26）~               | 位图像素数据存储在 Native 堆中。配合 NativeAllocationRegistry 进行垃圾回收 |

**为什么 Android 2.3.3 中 Native 的内存释放不可预测？在 Java 对象的 finalize 被调用时直接释放的方案有何不妥?Android 8.0 的 NativeAllocationRegistry 的引入是为了解决什么问题?**

带着疑问，我们去了解下Bitmap其内存是如何被分配和销毁以及学习一下 NativeAllocationRegistry 的技术。

### 总览

先从整体上看一下 Bitmap。
![](./assets/68747470733a2f2f626c6f672d313235313638383530342e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f3230313930362f6269746d61702d6372656174696f6e2d617263682e706e67.png)

- 用户在手机屏幕上看到的是一张张图片
- App 中这些图片实际上是 BitmapDrawable
- BitmapDrawable 是对 Bitmap 的包装
- Bitmap 是对 SkBitmap 的包装。具体说来， Bitmap 的具体实现包括 Java 层和 JNI 层，JNI 层依赖 [Skia](https://github.com/google/skia)。
- SkBitmap 本质上可简单理解为内存中的一个字节数组
所以说 Bitmap 其实是一个字节数组。创建 Bitmap 是在内存中分配一个字节数组，销毁 Bitmap 则是回收这个字节数组。

### 创建
创建 Bitmap 的方式很多，

- 可以通过 SDK 提供的 API 来创建 Bitmap
- 加载某些布局或资源时会创建 Bitmap
- [Glide](https://github.com/bumptech/glide) 等第三方图片库会创建 Bitmap

先说通过 API 创建 Bitmap。SDK 中创建 Bitmap 的 API 很多，分成三大类：

- **创建** Bitmap - `Bitmap.createBitmap()` 方法在内存中从无到有地创建 Bitmap
- **拷贝** Bitmap - `Bitmap.copy()` 从已有的 Bitmap 拷贝出一个新的 Bitmap
- **解码** - 从文件或字节数组等资源解码得到 Bitmap，这是最常见的创建方式
- BitmapFactory.decodeResource()
- [ImageDecoder.decodeBitmap](https://developer.android.com/reference/android/graphics/ImageDecoder)。ImageDecoder 是 Android 9.0 新加的类


## 图片文件大小和其占用内存是一致的吗
图片占用内存的大小与图片本身的大小没有直接关系，内存的数据和文件的数据不一样，内存主要是解码后的每个点的数据；而文件数据要看你的格式、压缩比、文件头、附加信息等等；所以WebP格式的图片虽然小，但占用的内存和其他格式无差别。