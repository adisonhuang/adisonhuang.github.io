# Android 内存分析工具(命令行)

## 1. dumpsys meminfo

`dumpsys meminfo` 是 Android 中最常用的查看内存的一个命令, 记录应用内存在不同类型的 RAM 分配之间的划分情况：

```shell
adb shell dumpsys meminfo package_name|pid [-d]

`-d` 标记会输出更多与 Dalvik 和 ART 内存占用情况相关的信息。
输出以KB为单位。
```
### 1.1 查看整体情况

```she
adb shell dumpsys meminfo
```

输入如下：

```shell
Applications Memory Usage (in Kilobytes):
Uptime: 890323212 Realtime: 4220902678

Total PSS by process:
    606,685K: com.ss.android.ugc.aweme (pid 21983 / activities)
    576,831K: system (pid 2141)
    489,218K: com.huawei.android.launcher (pid 5272 / activities)
    412,969K: com.android.systemui (pid 4303)
    378,407K: com.tencent.mm (pid 11384 / activities)
    316,658K: android.hardware.graphics.allocator@2.0-service (pid 748)
    280,486K: com.ss.android.lark (pid 16232 / activities)
    243,692K: com.ss.android.auto (pid 3409)
    196,695K: com.ss.android.ugc.aweme:miniappX (pid 22878)
		...
		...
		...

Total PSS by OOM adjustment:
    991,733K: Native
        316,658K: android.hardware.graphics.allocator@2.0-service (pid 748)
         81,071K: media.codec (pid 1075)
         44,014K: CameraDaemon (pid 1073)
         42,074K: surfaceflinger (pid 788)
         37,548K: zygote (pid 695)
         37,292K: com.android.calendar (pid 6449)
         37,047K: com.huawei.intelligent:intelligentService (pid 6375)
         34,675K: android.hardware.graphics.composer@2.2-service (pid 749)
         25,172K: zygote64 (pid 1047)
    576,831K: System
        576,831K: system (pid 2141)
    788,253K: Persistent
        412,969K: com.android.systemui (pid 4303)
         60,482K: com.huawei.HwOPServer (pid 5104)
         50,241K: com.android.phone (pid 5180)
         45,691K: com.huawei.systemserver (pid 4629)
         38,344K: com.huawei.hiview (pid 5159)
         37,893K: com.huawei.iaware (pid 4710)
         37,002K: com.huawei.nearby (pid 4872)
         29,056K: com.huawei.harmonyos.foundation (pid 5130)
         21,070K: com.huawei.securityserver (pid 5029)
         20,552K: com.android.nfc (pid 14751)
         17,482K: com.huawei.hiaction (pid 15913)
         17,471K: com.huawei.android.pushagent.PushService (pid 18531)
     15,409K: Persistent Service
         15,409K: com.android.bluetooth (pid 22380)
    597,783K: Foreground
        489,218K: com.huawei.android.launcher (pid 5272 / activities)
        108,565K: com.huawei.intelligent (pid 14886)
    523,458K: Visible
        158,410K: com.huawei.hwid.core (pid 16290)
         46,251K: com.huawei.ohos.photos (pid 10932)
         45,385K: com.google.android.gms.persistent (pid 3757)
         41,585K: com.huawei.hwid.persistent (pid 20615)
         36,724K: com.huawei.android.hwouc (pid 25447)
         35,977K: com.huawei.recsys (pid 14892)
         33,336K: com.huawei.android.totemweather (pid 14347)
         31,143K: com.google.android.gms.persistent (pid 3727)
         28,301K: com.huawei.hiai.engineservice (pid 10680)
         27,966K: com.huawei.profile (pid 12498)
         21,669K: com.huawei.lbs (pid 25766)
          8,618K: com.android.se (pid 19437)
          8,093K: com.huawei.vassistant:interactor (pid 31087)
     89,465K: Perceptible
         72,022K: com.google.android.inputmethod.latin (pid 29819)
         17,443K: com.huawei.vassistant:wakeup (pid 31110)
    281,443K: Perceptible Low
        108,049K: com.huawei.hwid.core (pid 30441)
         96,076K: com.huawei.android.launcher (pid 19806 / activities)
         36,923K: com.google.android.gms (pid 4003)
         30,465K: com.huawei.hwid.persistent (pid 14251)
          9,930K: com.huawei.lbs (pid 4129)
    127,204K: A Services
         46,686K: com.huawei.health:DaemonService (pid 18124)
         44,862K: com.huawei.wallet (pid 29714)
         35,656K: com.huawei.systemmanager:service (pid 23031)
  1,241,580K: Previous
        606,685K: com.ss.android.ugc.aweme (pid 21983 / activities)
        196,695K: com.ss.android.ugc.aweme:miniappX (pid 22878)
  1,145,459K: B Services
        378,407K: com.tencent.mm (pid 11384 / activities)
         90,070K: com.huawei.hwid.core (pid 477)
         80,241K: com.tencent.mm (pid 4289)
         74,925K: com.tencent.mobileqq:MSF (pid 3657)
         71,197K: com.tencent.mm:push (pid 12201)
         67,011K: com.tencent.mm:push (pid 2392)
  2,185,840K: Cached
        280,486K: com.ss.android.lark (pid 16232 / activities)
        243,692K: com.ss.android.auto (pid 3409)
        161,188K: com.ss.android.lark:sandboxed_process1 (pid 17018)
        151,325K: com.tencent.mm:toolsmp (pid 13044)
         76,070K: com.tencent.mm:tools (pid 11367)
         74,482K: com.android.mms (pid 10051 / activities)
         71,043K: com.ss.android.lark:wschannel (pid 2494)
         64,189K: com.ss.android.auto:sandboxed_process1 (pid 4941)
         63,318K: com.tencent.mm:support (pid 30592)
         59,377K: com.tencent.mm:exdevice (pid 11322)
         58,457K: com.huawei.appmarket (pid 3972)
         58,029K: com.android.gallery3d (pid 3027)
         56,591K: com.huawei.hiai (pid 29576)
         55,151K: com.huawei.appmarket (pid 1448)

Total PSS by category:
    861,742K: Native
    747,786K: Dalvik
        674,320K: .Heap
         44,887K: .Zygote
         18,755K: .LOS
          9,824K: .NonMoving
    492,577K: .art mmap
        324,296K: .App art
        168,281K: .Boot art
    452,769K: .dex mmap
        324,266K: .App vdex
        119,845K: .App dex
          8,658K: .Boot vdex
    423,432K: EGL mtrack
    357,649K: .so mmap
    247,011K: .apk mmap
    244,812K: GL mtrack
    241,900K: Unknown
    202,968K: Dalvik Other
        167,209K: .LinearAlloc
         19,303K: .GC
         16,456K: .IndirectRef
              0K: .JITCache
              0K: .CompilerMetadata
    105,374K: Other mmap
     52,039K: .jar mmap
     35,442K: .oat mmap
      4,476K: Stack
      3,415K: Other dev
      2,477K: .ttf mmap
        932K: Cursor
        756K: Ashmem
          0K: Gfx dev
          0K: Other mtrack

Total RAM: 7,789,196K (status normal)
 Free RAM: 2,771,304K (2,185,840K cached pss +   476,624K cached kernel +   108,840K free)
 Used RAM: 7,358,542K (6,378,618K used pss +   979,924K kernel)
 Lost RAM:   409,255K
     ZRAM: 1,336,996K physical used for 4,163,888K in swap (4,194,300K total swap)
   Tuning: 384 (large 512), oom   322,560K, restore limit   107,520K (high-end-gfx)
```

#### 1.1.1 Total RAM

这个数值是当前系统的所有内存大小。它是读取` /proc/meminfo` 节点中的 MemTotal 获取的。它一般并不等于机器的物理内存大小，那是因为 linux kernel 都会有预留内存（reserved）。物理内存 - kernel reserved = 系统实际内存总和。

#### 1.1.2 Free RAM

这个数值是当前系统可用内存大小。它是后面括号内那3项之和：

* **cached pss**

  是 `Total PSS by OOM adjustment` 里面 Cached 分类的总和。OOM score 是 Android 根据进程的类型给 lmk 打的分数，lmk 从高分开始回收进程。其中 Cached 的是 OOM 900 以上的，一般都是切到后台的进程，表示可以随时回收。

* **cached kernel**

  是 /proc/meminfo 中下面几项计算出来的： Buffers + Cached + SReclaimable - Mapped。

  * **Buffers:** 是 kernel 中块设备（block）申请的内存大小。
  * **Cached:** 这部分是文件缓存，例如进程加载的 so，jar，ttf，dex 等资源。它们的原始数据都在磁盘中，在内存紧缺的时候可以很方便将占用的内存交换出来使用。
  * **SReclaimable:** 是 Slab 中的可回收的部分。Slab 是 linux 中的一种内存分配机制。一般都是 kernel 模块使用的。在 Android 7 的时候 dumpsys meminfo 还是将所有的 Slab 计算到 Free RAM 中的，在 10 中改成可以回收的部分了
  * **Mapped:** 是 Cached 中已经被 mapped 的部分。Cached 中有一部分是被 unmapped 的，但是没有马上释放。可以理解 Mapped 就是进程中的一些文件资源已经被 load 进内存里面的。

* **free**

​		是 /proc/meminfo 下的 MemFree

从上面的统计内容来看，和一般 linux 理解的 free RAM 并不一样，一般的 linux 认为 Free RAM = free + Cached。也就是说 Cached 在内存充足的情况，缓存了进程运行所需要的资源，能加快进程的运行（对于app启动速度尤为明显）；当内存不足的情况，可以快速的交换出来。linux 认为这可以算作可用内存，所以 linux 的 Cached 一般都比较大，真正的 free 内存都很少。而 dumpsys meminfo 则认为 lmk 可以随时回收的 Pss 是可用内存，而 load 进内存的文件 Cached 则不算。

#### 1.1.3 Used RAM

这个数值是当前系统已经使用内存的大小。它也是后面括号内那2项之和：

* **used pss:** 是 Total PSS by OOM adjustment 除了 Cached 分类之外的进程的 Pss 之和。

* **kernel**: 分2部分：

  第一部分是 /proc/meminfo 中： Shmem + SUnreclaim+ PageTables + KernelStack：

  *  **Shmem:** Shared memory 即 kernel 中的共享内存，tmpfs 也会被统计为 Shmem。
  * **SUnreclaim:** Slab 中的不可回收部分。
  * **PageTables:** kernel 中用来转化虚拟地址和物理地址的。
  *  **KernelStack:** 每个用户线程都会在 kernel 有个内核栈（kernel stack）。kernel stack 虽然属于用户态线程，但是用户态无法访问，只有 syscall、异常等进入到内核态才会调用到。所以这部分内存是内核代码使用的。

  第二部分是 `VM_ALLOC_USED`，它是 kernel 中的模块通过 vmalloc 函数申请的内存，通过统计 /proc/vmallocinfo 中 “pages=” 数值之和得到（注意这里统计的单位是页面，一般Android上跑的 linux 一个页面都是 4Kb）

#### 1.1.4 ZRAM

zram 是 linux 的一项内存压缩技术，Android 里面用来当 Swap 用。当配置开启了 Zram，AMS 会把一些优先级低线程标记为可以放入 Swap 分区（例如说一些 Cached 进程）。这个 Swap 是基于内存的（Zram 支持写回磁盘，但是Android没支持），线程数据放入 Swap 的时候，会进行压缩（可以配置压缩算法），然后把之前占用的内存就可以给其他线程使用了。当再次使用这个线程数据的时候，再从 Swap 解压换回正常内存。这样能在小内存设备上挤出更多的内存空间给前台进程使用。

* **physical used for**:是读取 /sys/block/zram0/mm_stat 统计的。mm_stat 一共有8个数值，含义可以参看参考资料里面的 kernel 官方文档。这里读取的是第三个数值，也就是 mem_used_total。这里代表是实际物理内存使用的大小（经过压缩算法）。
* **in swap**: 是 /proc/meminfo 里面 SwapTotal - SwapFree。代表 swap 里存放实际内存的大小（解压之后的） 。
* **total swap**: 是 /proc/meminfo 里面的 SwapTotal。可以由方案配置



### 1.2 查看应用内存情况

```shell
adb shell dumpsys meminfo pid
```

输出：

```shell
Applications Memory Usage (in Kilobytes):
Uptime: 891059163 Realtime: 4221638629

** MEMINFO in pid 11384 [com.tencent.mm] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    51333    51304        0   143586   265728   234533    31194
  Dalvik Heap    71657    71652        0    12801    88052    63476    24576
 Dalvik Other    19908    19908        0      944
        Stack       56       56        0       24
       Ashmem        4        4        0        0
    Other dev       56        0       56        0
     .so mmap     6214      808     1144     8905
    .jar mmap      672        0        0        0
    .apk mmap    20208       32      140      276
    .ttf mmap     2963        0     2540        0
    .dex mmap       83       12        0     4392
    .oat mmap      283        0        0        0
    .art mmap     9057     8848        0     6925
   Other mmap     6005     2736      748        0
    GL mtrack    16708    16708        0        0
      Unknown     5345     5344        0    20302
        TOTAL   408707   177412     4628   198155   353780   298009    55770

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    80500
         Native Heap:    51304
                Code:     4676
               Stack:       56
            Graphics:    16708
       Private Other:    28796
              System:   226667

               TOTAL:   408707       TOTAL SWAP PSS:   198155

 Objects
               Views:     1369         ViewRootImpl:        1
         AppContexts:        9           Activities:        1
              Assets:       16        AssetManagers:        0
       Local Binders:      222        Proxy Binders:      137
       Parcel memory:       45         Parcel count:      182
    Death Recipients:       56      OpenSSL Sockets:        0
            WebViews:        0

 SQL
         MEMORY_USED:      376
  PAGECACHE_OVERFLOW:       57          MALLOC_SIZE:      117

 DATABASES
      pgsz     dbsz   Lookaside(b)          cache  Dbname
         4      108            109       26/30/15  /data/user/0/com.tencent.mm/databases/google_app_measurement.db
         4       28             35         2/18/3  /data/user/0/com.tencent.mm/databases/Scheduler.db

 Asset Allocations
    : 7965K
    : 7965K
```

> **PSS**
>
> 进程独占的 RAM 页会直接计入其 PSS 值，而与其他进程共享的 RAM 页则仅会按相应比例计入 PSS 值

#### 1.2.1 横列项参数：

* **Pss Total**: /proc/pid/smaps vma 里面 Pss 之和
* **Private Dirty**: /proc/pid/smaps vma 里面 Private_Dirty 之和
* **Private Clean**: /proc/pid/smaps vma 里面 Private_Clean 之和
* **SwapPss Dirty**: /proc/pid/smaps vma 里面 SwapPss 之和（这是开了 Zram 的，如果没开就是 Swap 之和，同时显示会变为 Swap Dirty）
* **Heap Size**: Native Heap 是 mallinfo() 里面 usmblks 的值。mallinof() 这个是一个 glic 函数，用来获取当前 native heap 的信息，返回的是一个结构体，里面的字段可以 man mallinfo 查看。usmblks 是 heap 能申请的最大值。Dalvik Heap 是 java 类 Runtime 里面的 totalMemory() 函数获取的，是当前虚拟机的 heap 大小，这值会动态调整，初始大小和调整策略由 dalvik.vm.heapgrowthlimit 和 dalvik.vm.heapsize 属性控制。
*  **Heap Alloc**: Native Heap 是 mallinfo() 里面 uordblks 的值，表示当前 native heap 已经申请的内存大小。Dalvik Heap 是 Runtime 里面 totalMemory() - freeMemory() 的值。此值大于 `Pss Total` 和 `Private Dirty`，这是因为您的进程是从 Zygote 派生的，且包含您的进程与所有其他进程共享的分配。
*  **Heap Free**: Native Heap 是 mallinfo() 里面 fordblks 的值，表示当前 native heap 空闲内存的大小。Dalvik Heap 是 Runtime 里面 freeMemory() 的值，表示虚拟机 heap 空闲内存大小。虚拟机 heap 的空闲内存小到一定程度会有一定程度的策略扩大 heap 的 total size，策略的阀值由上面说的属性控制

 #### 1.2.2 竖行项参数

只有 Pss Total、Private Dirty、Private Clean、SwapPss Dirty 区分竖直的分类（Heap Size、Heap Alloc、Heap Free 只区分 Native Heap 和 Dalvik Heap）

* **Natvie Heap**: /proc/pid/smaps 中： 所有 [heap]、[anon:libc_malloc] 的 vma 中各自字段之和。例如说 Natvie Heap 的 Pss Total 就是所有的 [heap]、[anon:libc_malloc] 的 vma 中 Pss 之和。从 vma 的名字来看，应该是 natvie malloc 申请的内存。
* **Dalvik Heap**: /proc/pid/smaps 中所有以 [anon:dalvik-alloc space、[anon:dalvik-main space、[anon:dalvik-large object space、[anon:dalvik-free list large object space、[anon:dalvik-non moving space、[anon:dalvik-zygote space 开头的 vma 中各字段之和。从 vma 的名字来看，应该是 java 虚拟机申请的内存。
*  **Dalvik Other:** /proc/pid/smaps 中 vma 中一些以 anon:dalvik 开头的字段之和（大部分是除了上面 Dalvik Heap 之外的），有点多我就不一一列的，详细的可以自己去 android/framework/base/core/jni/android_os_Debug.cpp 里面的 load_maps() 函数里面看。这里看 vma 名字像是虚拟机的一些引用计数、字节码缓存之类的。
* **Stack:** /proc/pid/smaps 中所有以 [stack 开头的 vma 各字段之和。应该是栈申请的内存，一般也不是很大，几十k这样。
* **Ashmem:** /proc/pid/smaps 中所有以 /dev/ashmem 开头的 vma 各字段之和。应该是进程间的共享内存。
* **Other dev:** /proc/pid/smaps 中所有以 /dev 开头的 vma 各字段之和。应该是一些设备驱动映射的内存。
*  **.so mmap:** /proc/pid/smaps 中所有以 .so 结尾的 vma 各字段之和。是 load so 映射的内存。
*  **.jar mmap:** /proc/pid/smaps 中所有以 .jar 结尾的 vma 各字段之和。是 load jar 映射的内存。
*  **.apk mmap:** /proc/pid/smaps 中所有以 .apk 结尾的 vma 各字段之和。是 load apk 映射的内存。
* **.ttf mmap:** /proc/pid/smaps 中所有以 .ttf 结尾的 vma 各字段之和。是 load ttf 映射的内存。
*  **.dex mmap:** /proc/pid/smaps 中所有以 .odex、.dex、.vdex、结尾的 vma 各字段之和。应该是加载虚拟机字节码、预编译的字节码映射的内存。
*  **.oat mmap:** /proc/pid/smaps 中所有以 .oat 结尾的 vma 各字段之和。应该是加载虚拟机预编译字节码映射的内存。好像是 system 的 jar dex2oat 出来的是 oat，其他的是 odex。
* **.art mmap:** /proc/pid/smaps 中所有以 .art、art] 结尾的 vma 各字段之和。这个好像是 odex 的一些索引，又叫启动镜像。也是虚拟机预编译出来的内容加载到内存里面所占用的。
* **Other mmap:** 除了上面分类， vma 中有名字的各字段之和。
*   **Unknown:** 除了上面分类， vma 中没有名字的各字段之和。
* **Total:** 分别等于各自列上的项目之和，Pss Total 还需要额外加上 SwapPss Dirty 这一列的求和。

#### 1.2.3 **App Summary**    

* **Java Heap:** Dalvik Heap 的 Private Dirty + .art mmap 的 (Private Dirty + Private Clean)。Android 认为这些代表进程的虚拟机占用的内存。
* **Native Heap:** Native Heap 中的 Private Dirty。Android 认为这个数值代表进程的 native 占用的内存。
* **Code:** .so mmap、.jar mmap、.apk mmap、ttf mmap、.dex mmap、oat mmap 的 (Private Dirty + Private Clean )。Android 认为这些是一些资源类（例如字节码文件、图片、字体等）占用的内存。
*  **Stack:** Stack 中 Private Dirty 的值。
* **Graphcis:** 这个是 GPU 里面 texture、buffer 占用的内存，需要 gpu 驱动支持才行。0 的话代表 gpu 驱动还没支持。
* **Private Other:** TOTAL(Private Dirty + Private Clean) - Java Heap - Native Heap - Code - Stack - Graphcis
* **System:** TOTAL(Pss) - TOTAL(Private Dirty + Private Clean)
* **TOTAL:** TOTAL(Pss)
* **TOTAL SWAP PSS:** TOTAL(SwapPss Dirty)

#### 1.2.4 **OBjects**

这些统计的是进程里面一些 Android Java 对象的引用信息，对于排查 Java 内存泄漏比较有用

#### 1.2.5 **SQL**

应该是 sql 申请的内存，暂时没研究过，不过一般这部分占用内存都不高。

## 2. /proc/xxx

### 2.1 /proc/pid/maps

每个进程都会有这个文件节点，表示进程的内存映射信息。例如说我们 cat 一下 systemui 的：

```shell
12c00000-12cc0000 rw-p 00000000 00:00 0          [anon:dalvik-main space (region space)]
12cc0000-12e40000 ---p 00000000 00:00 0          [anon:dalvik-main space (region space)]
12e40000-12f00000 rw-p 00000000 00:00 0          [anon:dalvik-main space (region space)]
12f00000-12f80000 rw-p 00000000 00:00 0          [anon:dalvik-main space (region space)]
12f80000-13280000 rw-p 00000000 00:00 0          [anon:dalvik-main space (region space)]
13280000-14140000 ---p 00000000 00:00 0          [anon:dalvik-main space (region space)]
14140000-22c00000 rw-p 00000000 00:00 0          [anon:dalvik-main space (region space)]
6fef2000-700cb000 rw-p 00000000 fd:03 1476       /system/framework/arm/boot.art
700cb000-70185000 rw-p 00000000 fd:03 1455       /system/framework/arm/boot-core-libart.art
70185000-701ad000 rw-p 00000000 fd:03 1467       /system/framework/arm/boot-okhttp.art
701ad000-701e2000 rw-p 00000000 fd:03 1452       /system/framework/arm/boot-bouncycastle.art
701e2000-701ee000 rw-p 00000000 fd:03 1449       /system/framework/arm/boot-apache-xml.art
701ee000-7087f000 rw-p 00000000 fd:03 1461       /system/framework/arm/boot-framework.art
7087f000-708ab000 rw-p 00000000 fd:03 1458       /system/framework/arm/boot-ext.art
708ab000-70966000 rw-p 00000000 fd:03 1470       /system/framework/arm/boot-telephony-common.art
70966000-70970000 rw-p 00000000 fd:03 1473       /system/framework/arm/boot-voip-common.art
70970000-70980000 rw-p 00000000 fd:03 1464       /system/framework/arm/boot-ims-common.art
70980000-70983000 rw-p 00000000 fd:03 1446       /system/framework/arm/boot-android.test.base.art
70983000-70a37000 r--p 00000000 fd:03 1477       /system/framework/arm/boot.oat
70a37000-70c5a000 r-xp 000b4000 fd:03 1477       /system/framework/arm/boot.oat
70c5a000-70c5b000 rw-p 00000000 00:00 0          [anon:.bss]
70c5b000-70c5d000 r--s 00000000 fd:03 1491       /system/framework/boot.vdex
```

上面只是截图了前面一部分，因为是整个进程的内存页面，所以整个文件会比较大（和进程复杂度有关，systemui 挺复杂的）。里面每一行代表一个 vma（virtual memory areas：虚拟内存区域）。下面我们来看看 vma 每一列代表什么含义：

* **12c00000-12cc0000:** 这个是该虚拟内存段的开始和结束地址。通过地址 x page_size 可以计算出该段内存的大小。
* **rw-p:** 该段内存的权限。前3位分别是：读、写、执行，最后一位 p 代表私有，s 代表共享。
* **00000000:** 该虚拟内存段起始地址在对应的映射文件中以页为单位的偏移量，对匿名映射，它等于0或者vm_start/PAGE_SIZE。
* **00:00:** 文件的主设备号和次设备号。对匿名映射来说，因为没有文件在磁盘上，所以没有设备号，始终为00:00。对有名映射来说，是映射的文件所在设备的设备号。上面有映射号的都是一些文件资源。
* **0:** 被映射到虚拟内存的文件的索引节点号，通过该节点可以找到对应的文件，对匿名映射来说，因为没有文件在磁盘上，所以没有节点号，始终为00:00。
* **[anon:dalvik-main space (region space)]:** 被映射到虚拟内存的文件名称。后面带(deleted)的是内存数据，可以被销毁。对有名来说，是映射的文件名。对匿名映射来说，是此段虚拟内存在进程中的角色。dumpsys meminfo 是通过这个字段来进行分类统计的。

### 2.2 /proc/pid/smaps

smaps 是 map 的扩展，它显示的信息更加详细，还是拿 systemui 的来举例：

 ```shell
 12c00000-12dc0000 rw-p 00000000 00:00 0          [anon:dalvik-main space (region space)]
 Name:           [anon:dalvik-main space (region space)]
 Size:               1792 kB
 KernelPageSize:        4 kB
 MMUPageSize:           4 kB
 Rss:                 480 kB
 Pss:                 480 kB
 Shared_Clean:          0 kB
 Shared_Dirty:          0 kB
 Private_Clean:         0 kB
 Private_Dirty:       480 kB
 Referenced:          480 kB
 Anonymous:           480 kB
 AnonHugePages:         0 kB
 ShmemPmdMapped:        0 kB
 Shared_Hugetlb:        0 kB
 Private_Hugetlb:       0 kB
 Swap:                  0 kB
 SwapPss:               0 kB
 Locked:                0 kB
 VmFlags: rd wr mr mw me ac 
 70c5b000-70c5d000 r--s 00000000 fd:03 1491       /system/framework/boot.vdex
 Size:                  8 kB
 KernelPageSize:        4 kB
 MMUPageSize:           4 kB
 Rss:                   8 kB
 Pss:                   2 kB
 Shared_Clean:          8 kB
 Shared_Dirty:          0 kB
 Private_Clean:         0 kB
 Private_Dirty:         0 kB
 Referenced:            8 kB
 Anonymous:             0 kB
 AnonHugePages:         0 kB
 ShmemPmdMapped:        0 kB
 Shared_Hugetlb:        0 kB
 Private_Hugetlb:       0 kB
 Swap:                  0 kB
 SwapPss:               0 kB
 Locked:                0 kB
 VmFlags: rd mr me ms
 ```

* **Size:** vma 空间的大小。可以由地址计算出来。等于 Vss，进程有时候 malloc 了一大块内存，但是并没有真正使用。但是还是会计算这块内存的大小
* **KernelPageSize:** kernel page size 的大小，一般是 4kb。
* **MMUPageSize:** MMU page size 大小，一般等于 KernelPageSize。
* **Rss:**表示进程实际占用内存的情况（包含共享资源）
* **Pss:**表示进程实际占用内存的情况（共享资源按比例均分）
* **hared/Private:** 该 vma 是私有的还是共享的。对照前面的权限标志位的最后一位，确实 p 的 Shared 有数值，并且不等于 Pss（被均分了）；而 s 的就是 Private 的有数值，并等于 Pss（因为是私有的）
* **Dirty/Clean:** 在页面被淘汰的时候，就会把该脏页面回写到交换分区（换出，swap out）。有一个标志位用于表示页面是否dirty。
* **Referenced:** 当前页面被标记为已引用或者包含匿名映射。
* **Anonymous:** 匿名映射的物理内存，这部分内存不来自于文件的内存大小。
* **AnonHugePages** 统计的是Transparent HugePages (THP)
* **Shared/Private_Hugetlb:** 由hugetlbfs页面支持的内存使用量，由于历史原因，该页面未计入“ RSS”或“ PSS”字段中。 并且这些没有包含在Shared/Private_Clean/Dirty 字段中。
*  **Swap:** 开启 Zram 后，Swap 到 Zram 分区的大小。这里是包含了共享资源部分内存的。
* **SwapPss:** Swap 共享部分按比例均分。
*  **Locked:** 常驻物理内存的大小，这些页不会被换出。
* **VmFlags:** 表示与特定虚拟内存区域关联的内核标志。标志如下：

```shell
rd  - readable
wr  - writeable
ex  - executable
sh  - shared
mr  - may read
mw  - may write
me  - may execute
ms  - may share
gd  - stack segment growns down
pf  - pure PFN range
dw  - disabled write to the mapped file
lo  - pages are locked in memory
io  - memory mapped I/O area
sr  - sequential read advise provided
rr  - random read advise provided
dc  - do not copy area on fork
de  - do not expand area on remapping
ac  - area is accountable
nr  - swap space is not reserved for the area
ht  - area uses huge tlb pages
ar  - architecture specific flag
dd  - do not include area into core dump
sd  - soft-dirty flag
mm  - mixed map area
hg  - huge page advise flag
nh  - no-huge page advise flag
mg  - mergable advise flag
```

  ### 1.3 /proc/meminfo

前面的 dumpsys meminfo  total 统计的部分的基础就来源于这个节点的数值：

  ```shell
  MemTotal:         486028 kB
  MemFree:           26124 kB
  MemAvailable:     181240 kB
  Buffers:             788 kB
  Cached:           172652 kB
  SwapCached:         6428 kB
  Active:           145524 kB
  Inactive:         159336 kB
  Active(anon):      61084 kB
  Inactive(anon):    74240 kB
  Active(file):      84440 kB
  Inactive(file):    85096 kB
  Unevictable:        2892 kB
  Mlocked:            2892 kB
  HighTotal:             0 kB
  HighFree:              0 kB
  LowTotal:         486028 kB
  LowFree:           26124 kB
  SwapTotal:        364516 kB
  SwapFree:         295852 kB
  Dirty:                20 kB
  Writeback:             0 kB
  AnonPages:        132388 kB
  Mapped:           108676 kB
  Shmem:              1644 kB
  Slab:              41300 kB
  SReclaimable:      13752 kB
  SUnreclaim:        27548 kB
  KernelStack:        5792 kB
  PageTables:        14332 kB
  NFS_Unstable:          0 kB
  Bounce:                0 kB
  WritebackTmp:          0 kB
  CommitLimit:      607528 kB
  Committed_AS:   11034652 kB
  VmallocTotal:     507904 kB
  VmallocUsed:           0 kB
  VmallocChunk:          0 kB
  CmaTotal:           8192 kB
  CmaFree:             248 kB
  ```

* **MemTotal:** 总共系统可用的物理内存。你会发现这个并不等于实际贴在机器上的内存大小。是因为还有一部分内存要作为 linux kernel 的保留内存（reserved）。

* **MemFree:** 这个就是真正意义上没有被使用的内存了。这值在 linux 系统上会比较低，其实有一部分内存是各个进程正在使用的，还有一大部分是 Cached。linux 的设计本意就是尽量多的使用内存，来保证进程间切换的平滑性（多任务）。

* **MemAvailable:** 这个值会比 MemFree 多一些，kernel 根据一些可以回收的内存（例如 cached、buffer 之类的）估算出一个能用的。

* **Buffers:** 块设备(block)申请的内存。

* **Cached/Mapped/AnonPages:** 进程的内存页面分为2种： file-backed pages（与文件对应的内存页）和 anonymous pages（匿名页）。file-backed pages 是指例如说进程的代码段、so、jar 库之类的，映射的都是 file-backed，而进程的堆、栈是不与文件相对应的，就属于匿名页。file-backed pages 在内存不足的时候可以直接写回硬盘的文件（叫 page-out），而不需要用到交互区（swap）；而匿名页在内存不足的时候就只能写到硬盘的交换区里（叫 swap-out）。

  * **AnonPages:** 是前面提到的 anonymous pages。它是进程的私有堆栈。
  * **Mapped:** 是前面提到的 file-backed pages。
  * **Cached:** Cached 是 Mapped 的超集，除了包含 Mapped 之外，Cached 还包括了 unmapped 的页面。进程中的 file-backed pages 被 unmapped 后不会马上回收，而是当做缓冲计算在 Cached 当中。但是进程中的 anonymous pages 一旦进程退出，则会马上回收。

* **SwapCached:** 这个好像是 Swap 分区的 page cached。SwapCached 并没有包含在 Cached 里面。

* **Active/Inactive/Active(anon)/Inactive(anon)/Active(file)/Inactive(file):** linux kernel 有个 LRU 的页面回收算法。LRU list 包括：LRU_INACTIVE_ANON（Inactive(anon)）、LRU_ACTIVE_ANON（Active(anon)）、LRU_INACTIVE_FILE（Inactive(file)）、 LRU_ACTIVE_FILE（Active(anon)）、LRU_UNEVICTABLE（Unevictable）。Active 就是对应正在使用的内存页面，一般这种是无法回收的。而 Inactive 则是最近没在使用的页面，一般这种是可以回收（reclaimed）的。而 Active(anon/file)则分别代表了正在使用的 anonymous pages 和 file-backed pages。Inactive(anon/file) 则是最近没在使用的 anonymous pages 和 file-backed pages。从上面定义来看一般： Active(anon) + Inactive(anon) = AnonPages ，但是发现并不相等，一个会受 Shmem 状态的影响，一个是有点小偏差。同理 Active(file) + Inactive(flie) 与不完全等于 Cached，一个也是受 Shmem 状态影响，一个是 Activie(flie) + Inactivie(file) 还包含 Buffers。

* **Unevictable:** 是不能被 page-out/swap-out 的页面。

* **SwapTotal/SwapFree:** 配置了 Zram 后，swap 分区的总大小和空闲大小。

* **Shmem:** 包括 shared memory 和 tmpfs。android 上一般也不大。

* **Slab/SReclaimable/SUnreclaim:** Slab 是 linux kernel 上的一种内存分配管理机制，一般是内核模块使用。SReclaimable 和 SUnreclaim 一个是可以回收的，一个是不能回收的 Slab 内存，而 Slab 则这2个加一起。dumpsys meminfo 把 SUnreclaim 算成是 kernel Used 。

* **KernelStack:** 内核的调用堆栈。内核栈是常驻内存的，既不包括在 LRU lists 里，也不包括在进程的 RSS/PSS 内存里，所以可以认为它是 kernel 消耗的内存。dumpsys meminfo 把 KernelStack 也计算在 Kernel Used 里面

* **PageTables:** `Page Table`用于将内存的虚拟地址翻译成物理地址，随着内存地址分配得越来越多，`Page Table`会增大。请把`Page Table`与`Page Frame（页帧）`区分开，物理内存的最小单位是 `page frame`，每个物理页对应一个描述符(struct page)，在内核的引导阶段就会分配好、保存在 mem_map[] 数组中，mem_map[] 所占用的内存被统计在 dmesg 显示的 reserved （kernel 预留内存）中，/proc/meminfo 的 MemTotal 是不包含它们的。而Page Table的用途是翻译虚拟地址和物理地址，它是会动态变化的，要从 MemTotal 中消耗内存。dumpsys meminfo 把 PageTables 计算在 Kernel Used 里面。

* **VmallocUsed:** kernel 模块使用 vmalloc 申请的内存。它也是算 kernel 使用的内存的，dumpsys meminfo 把 VmallocUsed 也计算在 Kernel Used 里面。然而在 Android 上 /proc/meminfo 这里的 VmallocUsed 统计不到数值。dumpsys meminfo 是通过 /proc/vmallocinfo 统计的。/proc/vmallocinfo 里不仅有 vmalloc 的，还有 ioremap（这个是IO地址映射的）、vmap 的，这2个是不占内存的。vmalloc 的每一行有 “pages=x” 的页面数量信息，所以直接计算 pages 的总数，然后 x page_size 就能统计到 VmallocUsed 了（这也是前面 dumpsys meminfo kernel used 那里说计算 pages= 的理由）。

* **CmaTotal/CmaFree:** 配置的 cma 总大小和 free 大小。一般有 iommu 的方案 cma 都会比较小。

## 参考资料
[Android 内存优化方法](http://light3moon.com/2020/12/07/Android%20%E5%86%85%E5%AD%98%E4%BC%98%E5%8C%96%E6%96%B9%E6%B3%95/)

  

  

  

  

  
