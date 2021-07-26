top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，包括进程ID、内存占用率、CPU占用率等,类似于Windows的任务管理器。top命令提供了实时的对系统处理器的状态监视.它将显示系统中CPU最“敏感”的任务列表.该命令可以按CPU使用.内存使用和执行时间对任务进行排序。在Android中，我们可以通过`adb shell`来执行linux相关命令。

Android 提供了大多数常见的 Unix 命令行工具。如需查看可用工具的列表，请使用以下命令:
``` shell 
adb shell ls /system/bin
```

## 语法
``` shell
[adb shell] top [option]
```

### 全局选项

| 全局选项 | 说明     |
|------|--------|
| -h/--help  | 显示帮助消息。 |


### 命令和命令选项

| 选项  | 说明  |
|:---:|:---:|
| -b  | 批处理 |
| -H  | 显示线程信息 |
| -k -field.. | 修改默认排序字段 （默认是-S,-%CPU,-ETIME,-PID）|
| -o field.. | 修改展示字段 （默认是PID,USER,PR,NI,VIRT,RES,SHR,S,%CPU,%MEM,TIME+,CMDLINE）|
| -d n | 设置n秒更新一次信息，默认是3秒 |
| -p pid  | 显示指定pid的进程信息 |
| -m n | 只显示n个任务信息 |
| -u user  | 指定用户名user |
| -n m  | 循环显示m次 |

## 命令输出说明
### 命令
```shell
top
```
### 输出
``` shell
Tasks: 829 total,   1 running, 828 sleeping,   0 stopped,   0 zombie
  Mem:  3665268K total,  3060140K used,   605128K free,   1585152 buffers
 Swap:  2097148K total,   622800K used,  1474348K free,  1160664K cached
800%cpu  11%user   0%nice  13%sys 771%idle   0%iow   2%irq   1%sirq   0%host
   PID USER         PR  NI VIRT  RES  SHR S[%CPU] %MEM     TIME+ ARGS
   679 system       -3  -8  11G  13M 8.2M S  8.3   0.3   7:13.28 surfaceflinger
  2801 u0_a197      20   0  13G  57M  36M S  6.6   1.5   2:56.09 com.breel.wall+
 28115 shell        20   0  10G 3.5M 1.8M R  3.6   0.1   0:45.73 top
   681 system       -3  -8  10G 3.9M 2.8M S  3.0   0.1   2:12.51 android.hardwa+
 28686 root         20   0    0    0    0 S  1.6   0.0   0:00.57 [kworker/u16:1]
 28388 root         20   0    0    0    0 S  1.0   0.0   0:08.10 [kworker/u16:8]
 18994 u0_a159      10 -10  47G 170M 107M S  1.0   4.7   0:27.44 com.google.and+
  1696 system       18  -2  16G 370M 278M S  1.0  10.3   5:16.30 system_server
   312 root         RT   0    0    0    0 S  1.0   0.0   0:54.34 [crtc_commit:1+
  3002 radio        20   0  14G  73M  43M S  0.6   2.0   0:26.95 com.android.ph+
    10 root         20   0    0    0    0 S  0.6   0.0   0:11.32 [rcuop/0]
 28236 u0_a195      20   0  15G 107M  61M S  0.3   2.9   0:02.06 com.google.and+
 27789 root         20   0    0    0    0 S  0.3   0.0   0:02.76 [kworker/3:0]
  1218 wifi         20   0  10G 2.7M 2.0M S  0.3   0.0   0:02.35 wificond
  1032 wifi         20   0  10G  10M 1.9M S  0.3   0.2   0:28.13 vendor.google.+
   651 root         RT   0    0    0    0 S  0.3   0.0   0:21.95 [sugov:0]
   264 root         -3   0    0    0    0 S  0.3   0.0   0:18.47 [kgsl_worker_t+
    53 root         20   0    0    0    0 S  0.3   0.0   0:02.24 [rcuop/5]
    45 root         20   0    0    0    0 S  0.3   0.0   0:04.84 [rcuop/4]
    29 root         20   0    0    0    0 S  0.3   0.0   0:05.73 [rcuop/2]

```
### 输出说明
* 第一行: `Tasks` — 任务（进程），具体信息说明如下：
    
   | 展示项 | 说明|
   |:---:|:----:|
   | 829 total | 共有829个进程 |
   | 1 running | 处于运行中的有1个 |
   | 828 sleeping | 828个在休眠（sleep） |
   |  0 stopped| stoped状态的有0个 |
   |  0 zombie| zombie状态（僵尸）的有0个 |

* 第二行: 内存状态，具体信息说明如下：

   | 展示项 | 说明|
   |:---:|:----:|
   | 3665268K total | 物理内存总量（约3.5GB) |
   | 3060140K used | 使用中的内存总量（约11.7GB） |
   | 605128K free | 空闲内存总量（约590M） |
   | 1585152 buffers | 缓存的内存量（约1.5GB） |

* 第三行，swap交换分区信息，具体信息说明如下：

   | 展示项 | 说明|
   |:---:|:----:|
   | 2097148K total | 交换区总量（约2GB） |
   | 622800K used | 使用的交换区总量（约608M） |
   | 1474348K free | 空闲交换区总量（约1.4GB） |
   | 1160664K cached | 缓冲的交换区总量（约1.1GB） |

* 第四行，cpu状态信息，具体属性说明如下：

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

* 第五行以下：各进程（任务）的状态监控，项目列信息说明如下：
    
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

    

### 补充说明

1. swap分区：当系统的物理内存不够用的时候，就需要将物理内存中的一部分空间释放出来，以供当前运行的程序使用。 那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到Swap空间中，等到那些程序要运行时，再从Swap中恢复保存的数据到内存中。
   > 关于swap分区，可以参考[swap 分区](https://cloud.tencent.com/developer/article/1648858)

2. 第二行中使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在android上free内存会越来越少，但不用为此担心。

3. 如果出于习惯去计算可用内存数，这里有个近似的计算公式：第二行的free + 第二行的buffers + 第三行的cached，按这个公式此台设备的可用内存：605128K +1585152K +1160664K = 3.2GB左右。

4. 对于内存监控，在top里我们要时刻监控第三行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。

5. nice值表示进程可被执行的优先级的修正数值。进程优先级(PR)值越小越快被执行，加入nice值后，将会使得PR变为：`PR(new)=PR(old)+nice`。在LINUX系统中，Nice值的范围从-20到+19
6. 不同的Android系统，top命令得到的结果有所不同。上述是Android O（8.0系统，level = 26） 及之后top命令得到的结果。Android N（7.1系统，level = 25） 及之前top命令得到结果如下：


```shell
//CPU占用率：User 用户进程；System 系统进程；IOW IO等待时间；IRQ 硬中断时间
User 0%, System 4%, IOW 0%, IRQ 0%
// CPU使用情况：
// User    处于用户态的运行时间，不包含优先值为负进程
// Nice    优先值为负的进程所占用的CPU时间
// Sys     处于核心态的运行时间
// Idle    除IO等待时间以外的其它等待时间 
// IOW     IO等待时间
// IRQ     硬中断时间
// SIRQ    软中断时间
User 2 + Nice 0 + Sys 12 + Idle 284 + IOW 0 + IRQ 0 + SIRQ 0 = 298
//// 进程属性：
// PID     进程在系统中的ID
// CPU%    当前瞬时所以使用CPU占用率
// S       进程的状态，其中S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值是负数。
// #THR    程序当前所用的线程数
// VSS     Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
// RSS     Resident Set Size 实际使用物理内存（包含共享库占用的内存）
// PCY     Policy系统对这个进程/线程的调度策略，bg 后台；fg 前台
// Name    程序名称
  PID USER     PR  NI CPU% S  #THR     VSS     RSS PCY Name
 6498 root     20   0   3% R     1   9116K   1888K  fg top
  253 root     RT   0   0% S     1      0K      0K  fg nanohub
    3 root     20   0   0% S     1      0K      0K  fg ksoftirqd/0
  387 root     20   0   0% S     1      0K      0K  fg kworker/u16:4
```
    
## 代码通过top命令获取cpu使用率
在Android O之前可以通过读取伪文件`/proc/stat`和`/proc/<pid>/stat`，然后计算得到cpu使用率，Android O之后有权限限制，但可以通过`top -n 1`获取cpu使用率，代码如下
```kotlin
    private fun getCpuDataForO(): Float {
        var process: java.lang.Process? = null
        var cpuRate = 0.0F
        try {
            process = Runtime.getRuntime().exec("top -n 1")
            val reader =
                BufferedReader(InputStreamReader(process.inputStream))
            var line: String? = null
            var cpuIndex = -1
            while (reader.readLine().also { line = it } != null) {
                line = line!!.trim { it <= ' ' }
                if (TextUtils.isEmpty(line)) {
                    continue
                }
                val tempIndex = getCPUIndex(line!!)
                if (tempIndex != -1) {
                    cpuIndex = tempIndex
                    continue
                }
                if (line!!.startsWith(android.os.Process.myPid().toString())) {
                    if (cpuIndex == -1) {
                        continue
                    }
                    val param = line!!.split("\\s+".toRegex()).toTypedArray()
                    if (param.size <= cpuIndex) {
                        continue
                    }
                    var cpu = param[cpuIndex]
                    if (cpu.endsWith("%")) {
                        cpu = cpu.substring(0, cpu.lastIndexOf("%"))
                    }
                    cpuRate =  cpu.toFloat() / Runtime.getRuntime().availableProcessors()
                }
            }
        } catch (e: Exception) {
            Log.e(TAG, "getCpuDataForO fail, error: %s", e.toString())
        } finally {
            process?.destroy()
        }
        return cpuRate
    }
```

## 参考链接
[每天一个linux命令（44）：top命令](https://www.cnblogs.com/peida/archive/2012/12/24/2831353.html)


