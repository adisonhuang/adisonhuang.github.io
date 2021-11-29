# meminfo命令

查看指定进程的内存使用情况
## 语法
``` shell
[adb shell] dumpsys meminfo package_name|pid 
```

## 输出如下
``` shell
Applications Memory Usage (in Kilobytes):
Uptime: 103140608 Realtime: 103140608

** MEMINFO in pid 14533 [package_name] **
                   Pss  Private  Private  SwapPss      Rss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty    Total     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------   ------
  Native Heap    94226    94144        0     2485    94396   112740    89386    13924
  Dalvik Heap    98179    98096        0      312    98392    92380    43228    49152
 Dalvik Other    17886    13624        0       56    22248
        Stack     4736     4736        0      450     4740
       Ashmem      182      180        0        0      196
      Gfx dev    39520    38224     1296        0    39520
    Other dev      169        0      168        0      396
     .so mmap    34230     3280    27940      272    39776
    .jar mmap     2616        0      524        0    20780
    .apk mmap    18513      208    13484        0    36812
    .ttf mmap      869        0      812        0      980
    .dex mmap    59773       20    59372       24    60168
    .oat mmap      442        0      212        0      840
    .art mmap     8749     7752       92     1849    11856
   Other mmap     4105      380     3336        1     5308
   EGL mtrack    27744    27744        0        0    27744
    GL mtrack    25240    25240        0        0    25240
      Unknown     5250     5200        0       40     5364
        TOTAL   447918   318828   107236     5489   447918   205120   132614    63076

 App Summary
                       Pss(KB)                        Rss(KB)
                        ------                         ------
           Java Heap:   105940                         110248
         Native Heap:    94144                          94396
                Code:   105920                         167936
               Stack:     4736                           4740
            Graphics:    92504                          92504
       Private Other:    22820
              System:    21854
             Unknown:                                   24932

           TOTAL PSS:   447918            TOTAL RSS:   494756       TOTAL SWAP PSS:     5489

 Objects
               Views:     1510         ViewRootImpl:        1
         AppContexts:       11           Activities:        1
              Assets:       27        AssetManagers:        0
       Local Binders:      130        Proxy Binders:       51
       Parcel memory:       51         Parcel count:      206
    Death Recipients:        4      OpenSSL Sockets:       12
            WebViews:        2

 SQL
         MEMORY_USED:      803
  PAGECACHE_OVERFLOW:      248          MALLOC_SIZE:      117

 DATABASES
      pgsz     dbsz   Lookaside(b)          cache  Dbname
         4       84            100     4055/41/18  /data/user/0/com.yy.hiyo/databases/xxxx.db
         4       20             95     1235/497/8  /data/user/0/com.yy.hiyo/databases/xxxx.db
         4       32             31         1/30/2  /data/user/0/com.yy.hiyo/databases/com.google.android.datatransport.events
         4      112            100       54/44/21  /data/user/0/com.yy.hiyo/databases/xxxx.db
```
## 输出说明

### 内存指标概念

| item  | 全称                  | 含义     | 等价                         |
| :---- | :-------------------- | :------- | :--------------------------- |
| USS   | Unique Set Size       | 物理内存 | 进程独占的内存               |
| PSS   | Proportional Set Size | 物理内存 | PSS= USS+ 按比例包含共享库   |
| RSS   | Resident Set Size     | 物理内存 | RSS= USS+ 包含共享库         |
| VSS   | Virtual Set Size      | 虚拟内存 | VSS= RSS+ 未分配实际物理内存 |
| Dirty |                       |          |                              |
| Clean |                       |          |                              |

* MEMINFO：
  
   | 展示项 | 说明|
   |:---:|:----:|
   | 829 total | 共有829个进程 |
   | 1 running | 处于运行中的有1个 |
   | 828 sleeping | 828个在休眠（sleep） |
   |  0 stopped| stoped状态的有0个 |
   |  0 zombie| zombie状态（僵尸）的有0个 |

* App Summary：

   | 展示项 | 说明|
   |:---:|:----:|
   | 3665268K total | 物理内存总量（约3.5GB) |
   | 3060140K used | 使用中的内存总量（约11.7GB） |
   | 605128K free | 空闲内存总量（约590M） |
   | 1585152 buffers | 缓存的内存量（约1.5GB） |

* Objects：

   | 展示项 | 说明|
   |:---:|:----:|
   | 2097148K total | 交换区总量（约2GB） |
   | 622800K used | 使用的交换区总量（约608M） |
   | 1474348K free | 空闲交换区总量（约1.4GB） |
   | 1160664K cached | 缓冲的交换区总量（约1.1GB） |

* SQL：

   | 展示项 | 说明|
   |:---:|:----:|
   | 800%cpu | CPU总量 |
   | 11%user | 用户空间占用CPU的百分比(不包含nice值为负的进程) |
   | 0%nice | 改变过优先级的进程占用CPU的百分比(nice值为负的进程)|
   | 13%sys | 内核空间占用CPU的百分比|
   | 771%idle | 空闲CPU百分比| 
   | 0%iow | IO等待占用CPU的百分比| 
   | 2%irq | 硬中断（Hardware IRQ）占用CPU的百分比| 
   |1%sirq | 软中断（Software Interrupts）占用CPU的百分比| 
   |0%host | ？|

* DATABASES：
  
    | 展示项 | 说明|
    |:---:|:----:|
    | PID | 进程id |
    | USER | 进程所有者 |
    | PR | 进程优先级 |
    | NI | nice值。负值表示高优先级，正值表示低优先级 |
    | VIRT | 进程使用的虚拟内存总量。VIRT=SWAP+RES |
    | RES | 程使用的、未被换出的物理内存大小，RES=CODE+DATA |
    | SHR | 共享内存大小 |
    | S | 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程 |
    | %CPU  | 上次更新到现在的CPU时间占用百分比 |
    | %MEM   | 进程使用的物理内存百分比 |
    | TIME+ ARGS   | 进程使用的CPU时间总计和进程名|

