# dumpsys

`dumpsys`命令功能很强大,可以获取设备上运行的所有系统服务的状态信息，如查看内存占用情况、帧率信息、收集网络使用情况统计信息等等。

## 语法
使用 dumpsys 的一般语法如下：
```shell
 adb shell dumpsys service [arguments]
```
例如：查看包名为`com.xx.xx`的内存情况

```shell
adb shell dumpsys meminfo com.xx.xx
```

可以通过下面任一命令查看与 `dumpsys` 配合使用的系统服务的完整列表

```shell
adb shell dumpsys -l
adb shell service list
```

## 系统服务

| 服务名       |  功能         |
| :-----------: | :-----------:|
| activity     | AMS相关信息  |
| package      |  PMS相关信息  |
| window       |  WMS相关信息  |
| processinfo  | 进程服务         |
| procstats    |  进程统计     |
| cpuinfo      | CPU          |
| meminfo      |  内存         |
| gfxinfo      | 图像         |
| SurfaceFlinger  | 图像相关   |
| input        | IMS相关信息  |
| power        |  PMS相关信息  |
| batterystats |  电池统计信息 |
| battery      | 电池信息     |
| alarm        |  闹钟信息     |
| dropbox      | 调试相关     |
| dbinfo       | 数据库       |
| appops               | app使用情况      |
| permission           | 权限             |
| batteryproperties    | 电池相关         |
| audio                | 查看声音信息     |
| netstats             | 查看网络统计信息 |
| diskstats            | 查看空间free状态 |
| jobscheduler         | 查看任务计划     |
| wifi                 | wifi信息         |
| diskstats            | 磁盘情况         |
| usagestats           | 用户使用情况     |
| devicestoragemonitor | 设备信息         |
| …                    | …                |