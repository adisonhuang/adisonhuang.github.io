# Android 内存综合治理
## 图片检测&监控
假设屏幕分辨率 `1080x1920`，一张 Bitmap 占满屏幕时内存大小是`1080x1920x4`，大约 8MB。所以一个不起眼的 Bitmap 占用的内存也远远大于多数对象。可见，如果应用内有许多不合理的图片，就会拉高内存水位。如下图， 目前88%以上的用户的手机是8.0及更高系统版本，它的内存申请是在Native层的。因此，图片内存的分析重点放在8.0以及更高的系统版本上。
![](./assets/20221219-110541.jpeg)

从[Bitmap: 从出生到死亡](https://blog.adison.top/perf-opt/Android/bitmap/bitmap-from-birth-to-death/#3-bitmap) 可以知道， 


无论哪种Bitmap创建方式，最终殊途同归。都会通过`Bitmap.cpp`中的`Bitmap::createBitmap`方法创建Bitmap对象。

所以，我们只需要hook这个`createBitma`p函数，就能够拿到每次图片创建时的bitmap的Java对象。通过该对象，可以获得`图片的尺寸大小`、`内存占用大小`，`堆栈`等信息，将这些信息上报到性能平台也就达到了检测与监控图片创建的目标。

这里hook的方案采用开源的inline-hook开源方案来完成。原理在本篇中不做描述。怎么查找到这个函数呢？

首先需要通过adb pull system/lib/libandroid_runtime.so 拿到系统的so文件，然后通过arm-linux-androideabi-nm -D libandroid_runtime.so | grep bitmap ，可以查找到对应的函数名，也就是我们要hook的函数，系统版本不同可能会有差异，注意兼容。



## 参考

[全民K歌内存篇3——native内存分析与监控](https://cloud.tencent.com/developer/article/1817357)