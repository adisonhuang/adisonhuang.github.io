# native 崩溃信号介绍

## 一、概念说明

在应用崩溃的时候，我们将会获取到两个信息:

• signal: 信号量，下文将会详细的说明不同的信号量及其含义

• code: 错误码, 除了几个所有信号量(signal) 公共的错误码(code)，一般不同信号量(signal)有特定的错误码(code)，可以看做是信号量(signal)的补充说明。

## 二、信号量(signal) 和 错误码(code)说明

### 1. SIGILL

SIG是信号名的通用前缀，ILL是 illegal instruction（ 非法指令 ） 的缩写。

对应的数值为 4。

SIGILL 是当一个进程尝试执行一个非法指令时发送给它的信号。

常见原因有:

1. CPU架构不匹配
2. .so 文件被破坏，或者代码段被破坏导致；
3. 主动崩溃，如[ _builtintrap()](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html#index-_005f_005fbuiltin_005ftrap) 也会使用非法指令来实现。

si_addr 为出错的指令。

该信号量中常见的错误码说明：

| Code       | 说明                            |
| :--------- | :------------------------------ |
| ILL_ILLOPC | 非法的操作码(opcode)            |
| ILL_ILLOPN | 非法的操作数(operand)           |
| ILL_ILLADR | 非法的寻址模式                  |
| ILL_ILLTRP | 非法的trap                      |
| ILL_PRVOPC | 特权操作码(Privileged opcode)   |
| ILL_PRVREG | 特权寄存器(Privileged register) |
| ILL_COPROC | 协处理器错误                    |
| ILL_BADSTK | 内部堆栈错误                    |

### 2. SIGBUS

SIG是信号名的通用前缀，BUS是bus error (总线错误 ) ，意味着系统检测到硬件问题后发送给进程的信号。

对应的数值为7。

通常该信号的产生不是因为硬件有物理上的损坏，而是代码实现有 bug 导致 ，如地址不对齐，或者不存在的物理地址等。

si_addr 为所访问的非法地址。

该信号量中常见的错误码说明：

| Code          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| BUS_ADRALN    | 访问的地址不对齐。32位处理器一般要求指针是4字节对齐的        |
| BUS_ADRERR    | 访问不存在的物理地址。一般是由于 mmap 的文件发生 truncated 导致。常见于文件访问过程中，被删除或者替换；或 mmap 到内存后，继续向文件写入且导致文件 truncated，再读取时就会出现该错误；另外， mmap 且访问超过文件实际大小的空间时，也可能会出现该错误 |
| BUS_OBJERR    | 特定对象的硬件错误                                           |
| BUS_MCEERR_AR | BUS_MCEERR_AR                                                |
| BUS_MCEERR_AQ | BUS_MCEERR_AQ                                                |

### 3. SIGSEGV

SIG 是信号名的通用前缀, SEGV 是 segmentation violation 的缩写。

对应的数值为 11。

该信号意味着一个进程执行了一个无效的内存引用，或发生段错误。

si_addr 为所访问的无效地址。

该信号量中常见的错误码说明：

| Code        | 说明                           |
| :---------- | :----------------------------- |
| SEGV_MAPERR | 地址不在 /proc/self/map 映射中 |
| SEGV_ACCERR | 没有访问权限                   |

### 4. SIGFPE

SIG是信号名的通用前缀。FPE是floating-point exception（浮点异常）的首字母缩写拼接而成。

对应的数值为 8。

该信号一般是算术运算相关问题导致的。

si_addr 为失败的指令。

该信号量中常见的错误码说明：

| Code       | 说明             |
| :--------- | :--------------- |
| FPE_INTDIV | 整数除以 0       |
| FPE_INTOVF | 整数溢出         |
| FPE_FLTDIV | 浮点数除以 0     |
| FPE_FLTOVF | 浮点数向下溢出   |
| FPE_FLTRES | 浮点数结果不精确 |
| FPE_FLTINV | 无效的浮点运算   |
| FPE_FLTSUB | 下标超出范围     |

### 5. SIGABRT

SIG是信号名的通用前缀。ABRT是abort program的缩写。

对应的数值为 6。

该信号意味着异常退出；通常是调用[ abort()](https://www.mkssoftware.com/docs/man3/abort.3.asp), [raise()](https://www.mkssoftware.com/docs/man3/raise.3.asp), [kill()](https://www.mkssoftware.com/docs/man3/kill.3.asp), [pthread_kill()](https://www.mkssoftware.com/docs/man3/pthread_kill.3.asp) 或者被系统进程杀死时出现。

当错误码为 SI_USER 时表示是被其它程序杀死，一般情况是由于ANR被 system_server 杀死；

其他错误码一般是业务自己调用 abort() 等函数退出，此时错误码一般认为无效。

### 6. SIGSTKFLT

SIG是信号名的通用前缀，STKFLT是stack fault 的缩写。

对应的数值为 16。

按照官方文档说明，该信号量意味着协处理器栈故障。

根据网上的部分问题结论说明，在内存耗尽时，一般 malloc 返回 NULL 且设置 errno 为 ENOMEM，但有些系统可能会使用 SIGSTKFLT 信号代替。

### 7. SIGPIPE

该信号对应的数值为 13。

该信号意味着管道错误，通常在进程间通信产生，比如采用FIFO(管道)通信的两个进程，读管道没打开或者意外终止仍继续往管道写，写进程就会收到 SIGPIPE 信号。

### 8. SIGUSR2

为 12。用户自定义信号2。

### 9. SIGTERM

SIG是信号名的通用前缀，TERM 是 Termination 的缩写。该信号对应的数值为 15。

与 SIGKILL (signal 9) 类似，不过 SIGKILL 不可捕获，而 SIGTERM 可被捕获。一般常见于 APP 处于后台时，被系统 vold 进程使用 SIGTERM 杀死，该情形基本没有意义，可忽略。

另外，chromium 多进程架构中，经常也会使用 SIGTERM 杀死出现了异常的子进程。

### 10. SIGSYS

该信号对应的数值为 31，通常是因为无效的 linux 内核系统调用而产生。

在 android O (android 8.0) 中，部分不安全的系统调用被移除，若代码中仍然使用它们，则会出现 SIGSYS。

### 11. SIGTRAP

该信号对应的数值为 5。gdb 调试设置断点等操作使用的信号。

| Code        | 说明        |
| :---------- | :---------- |
| TRAP_BRKPT  | TRAP_BRKPT  |
| TRAP_TRACE  | TRAP_TRACE  |
| TRAP_BRANCH | TRAP_BRANCH |
| TRAP_HWBKPT | TRAP_HWBKPT |

参考文献：

1. [siginfo_t — data structure containing signal information](https://www.mkssoftware.com/docs/man5/siginfo_t.5.asp)
2. [linux系统编程之信号（一）：中断与信号](https://www.cnblogs.com/mickole/p/3189156.html)
3. [linux系统编程之信号（六）：信号发送函数sigqueue和信号安装函数sigaction](https://www.cnblogs.com/mickole/p/3191804.html)
4. [SIGSTKFLT](https://stackoverflow.com/questions/21104107/fatal-signal-16-sigstkflt-has-been-occured-unexpectedly)