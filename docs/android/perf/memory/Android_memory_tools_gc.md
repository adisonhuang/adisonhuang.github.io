# Android 内存分析工具(查看GC)

## 1. Logcat读取GC消息

发生垃圾回收事件时，相应消息会输出到 Logcat 中。

### 1.1. Dalvik 日志消息

在 Dalvik 中，每个 GC 消息都会将以下信息输出到 logcat 中：

```shell
D/dalvikvm(PID): GC_Reason Amount_freed, Heap_stats, External_memory_stats, Pause_time
```

示例：

```shell
D/dalvikvm( 9050): GC_CONCURRENT freed 2049K, 65% free 3571K/9991K, external 4703K/5261K, paused 2ms+2ms
```

- **GC_Reason**(GC 原因)
  什么触发了 GC 以及是哪种回收。可能的原因包括：
  
  	* `GC_CONCURRENT`在堆开始占用内存时释放内存的并发 GC。
  	* `GC_FOR_MALLOC` 堆已满而系统不得不停止应用并回收内存时，应用尝试分配内存而引起的 GC。
  	* `GC_HPROF_DUMP_HEAP`当请求创建 HPROF 文件来分析堆时发生的 GC。
  	* `GC_EXPLICIT`显式 GC，例如调用 `gc()` 时（应避免调用它，而应信任 GC 会根据需要运行）。
  	* `GC_EXTERNAL_ALLOC`这仅适用于 API 级别 10 及更低级别（更新的版本会在 Dalvik 堆中分配任何内存）。外部分配内存的 GC（例如存储在原生内存或 NIO 字节缓冲区中的像素数据）。
  
- **Amount_freed**(释放量)

  从此次 GC 中回收的内存量。

- **Heap_stats**(堆统计数据)

  堆的可用空间百分比与（活动对象数量）/（堆总大小）。

- **External_memory_stats**(外部内存统计数据)

  API 级别 10 及更低级别的外部分配内存（已分配内存量）/（发生回收的限值）。

- **Pause_time**(暂停时间)

  堆越大，暂停时间越长。并发暂停时间显示两个暂停：一个出现在回收开始时，另一个出现在回收快要完成时。

在此类日志消息积聚时，请注意堆统计数据（上面示例中的 `3571K/9991K` 值）的增大情况。如果此值继续增大，可能会出现内存泄露。

### 1. 2. ART 日志消息

与 Dalvik 不同，ART 不会为未明确请求的 GC 记录消息。只有在系统认为 GC 速度较慢时才会输出 GC 消息。更确切地说，仅在 GC 暂停时间超过 5 毫秒或 GC 持续时间超过 100 毫秒时。如果应用未处于可察觉到暂停的状态（例如应用在后台运行时，这种情况下，用户无法察觉 GC 暂停），则其所有 GC 都不会被视为速度较慢。系统一直会记录显式 GC。

ART 会在其垃圾回收日志消息中包含以下信息：

```shell
I/art: GC_Reason GC_Name Objects_freed(Size_freed) AllocSpace Objects,
    Large_objects_freed(Large_object_size_freed) Heap_stats LOS objects, Pause_time(s)
```

示例：

```shell
I/art : Explicit concurrent mark sweep GC freed 104710(7MB) AllocSpace objects,
    21(416KB) LOS objects, 33% free, 25MB/38MB, paused 1.230ms total 67.216ms
```

- **GC_Reason**(GC 原因)
	什么触发了 GC 以及是哪种回收。可能的原因包括：
	* `Concurrent` 不会挂起应用线程的并发 GC。此 GC 在后台线程中运行，而且不会阻止分配。
	* `Alloc` 应用在堆已满时尝试分配内存而引起的 GC。在这种情况下，垃圾回收在分配线程中发生。
	* `Explicit` 由应用明确请求的垃圾回收，例如，通过调用 `gc()` 或 `gc()`。与 Dalvik 一样，在 ART 中，最佳做法是信任 GC 并避免请求显式 GC（如果可能）。不建议请求显式 GC，因为它们会阻止分配线程并不必要地浪费 CPU 周期。此外，如果显式 GC 导致其他线程被抢占，则也可能会导致卡顿（应用出现卡顿、抖动或暂停）。
	* `NativeAlloc` 原生分配（例如位图或 RenderScript 分配对象）导致出现原生内存压力，进而引起的回收。
	* `CollectorTransition` 由堆转换引起的回收；这由在运行时变更 GC 策略引起（例如应用在可察觉到暂停的状态之间切换时）。回收器转换包括将所有对象从空闲列表空间复制到碰撞指针空间（反之亦然）。回收器转换仅在以下情况下出现：在 Android 8.0 之前的低内存设备上，应用将进程状态从可察觉到暂停的状态（例如应用在前台运行时，这种情况下，用户可以察觉 GC 暂停）更改为察觉不到暂停的状态（反之亦然）。
	* `HomogeneousSpaceCompact` 同构空间压缩是空闲列表空间到空闲列表空间压缩，通常在应用进入到察觉不到暂停的进程状态时发生。这样做的主要原因是减少内存使用量并对堆进行碎片整理。
	* `DisableMovingGc` 这不是真正的 GC 原因，但请注意，由于在发生并发堆压缩时使用了 GetPrimitiveArrayCritical，回收遭到阻止。一般情况下，强烈建议不要使用 GetPrimitiveArrayCritical，因为它在移动回收器方面存在限制。
	* `HeapTrim` 这不是 GC 原因，但请注意，在堆修剪完成之前，回收会一直受到阻止。


- **GC_Name**(GC 名称)
  ART 具有可以运行的多种不同的 GC。
  	*  `Concurrent mark sweep (CMS)` 整个堆回收器，会释放和回收除映像空间以外的所有其他空间。
  	*  `Concurrent partial mark sweep` 几乎整个堆回收器，会回收除映像空间和 Zygote 空间以外的所有其他空间。
  	*  `Concurrent sticky mark sweep` 分代回收器，只能释放自上次 GC 后分配的对象。此垃圾回收比完整或部分标记清除运行得更频繁，因为它更快速且暂停时间更短。
  	* `Marksweep + semispace` 非并发、复制 GC，用于堆转换以及同构空间压缩（对堆进行碎片整理）。

- **Objects_freed**(释放的对象)

  此 GC 从非大型对象空间回收的对象数量。

- **Size_freed**(释放的大小)

  此 GC 从非大型对象空间回收的字节数量。

- **Large objects freed**(释放的大型对象)

  此垃圾回收从大型对象空间回收的对象数量。

- **Large object size freed**(释放的大型对象大小)

  此垃圾回收从大型对象空间回收的字节数量。

- **Heap stats**(堆统计数据)

  可用空间百分比与（活动对象数量）/（堆总大小）。

- **Pause times**(暂停时间)

  通常情况下，暂停时间与 GC 运行时修改的对象引用数量成正比。当前，ART CMS GC 仅在 GC 即将完成时暂停一次。 移动 GC 的暂停时间较长，会在 GC 的大部分时间持续。

如果在 logcat 中看到大量 GC，请注意堆统计数据（上面示例中的 `25MB/38MB` 值）的增大情况。如果此值继续增大，且始终没有变小的趋势，可能会出现内存泄漏。或者，如果看到原因为“Alloc”的 GC，则已快要达到堆容量上限，并且很快会出现 OOM 异常。

## 2. 通过`Debug`类读取GC消息

在实验室或者内部试用环境，我们也可以通过 `Debug.startAllocCounting` 来监控 Java 内存分配和 GC 的情况，需要注意的是这个选项对性能有一定的影响，虽然目前还可以使用，但已经被 Android 标记为 deprecated。

通过监控，我们可以拿到内存分配的次数和大小，以及 GC 发起次数等信息。

```java
long allocCount = Debug.getGlobalAllocCount();
long allocSize = Debug.getGlobalAllocSize();
long gcCount = Debug.getGlobalGcInvocationCount();
```

上面的这些信息似乎不太容易定位问题，在 Android 6.0 之后系统可以拿到更加精准的 GC 信息。

```java
// 运行的GC次数
Debug.getRuntimeStat("art.gc.gc-count");
// GC使用的总耗时，单位是毫秒
Debug.getRuntimeStat("art.gc.gc-time");
// 阻塞式GC的次数
Debug.getRuntimeStat("art.gc.blocking-gc-count");
// 阻塞式GC的总耗时
Debug.getRuntimeStat("art.gc.blocking-gc-time");
```

需要特别注意阻塞式 GC 的次数和耗时，因为它会暂停应用线程，可能导致应用发生卡，我们也可以更加细粒度地分应用场景统计。

## 参考

[android 高手课](https://time.geekbang.org/column/article/71610)