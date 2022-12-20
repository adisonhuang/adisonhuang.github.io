# Native Hook

## 1. 什么是HOOK？

Hook 直译过来就是“钩子”的意思，是指截获进程对某个 API 函数的调用，使得 API 的执行流程转向我们实现的代码片段，从而实现我们所需要得功能，这里的功能可以是监控、修复系统漏洞，也可以是劫持或者其他恶意行为。

## 2. 为什么要HOOK？

Native Hook技术一直在解决一些疑难问题时特别有效，由于不仅可以修改我们自己的代码，也可以修改系统的代码，所以用处非常广泛，在性能监控方面可以做锁阻塞，IO ，Binder call，fd泄漏等的监控，在稳定性方面可以修复特殊机型或者系统的一些native bug。

## 3. 如何HOOK？

`Native Hook`技术又分为`GOT/PLT Hook`，`Inline Hook`，`Trap Hook` 等，`GOT/PLT Hook`兼容性比较好，可以达到上线标准，但是只能Hook基于GOT表的一些函数，`Inline Hook`能Hook几乎所有函数，但是兼容性较差，不能达到上线标准，`Trap Hook`兼容性也能达到上线标准而且Hook范围比Inline Hook还广，但是由于它是基于信号处理的，性能比较差。

|              | 兼容性 | Hook范围 | 性能 |
| ------------ | ------ | -------- | ---- |
| GOT/PLT Hook | 优     | 差       | 优   |
| Inline Hook  | 差     | 优       | 优   |
| Trap Hook    | 优     | 优       | 差   |

可以看出三种方案都是各有优缺点，我们可以根据自己的需要选择合适的Hook方案，发挥他们的长处，目前来说基于兼容性和性能两方面考虑，GOT/PLT Hook通常是我们线上项目的最优选择。

### 3.1 GOT/PLT Hook

GOT/PLT Hook 主要是用于替换某个 SO 的外部调用，通过将外部函数调用跳转成我们的目标函数。

那 GOT/PLT Hook 的实现原理究竟是什么呢？你需要先对 SO 库文件的 ELF 文件格式和动态链接过程有所了解。

#### ELF 格式

ELF（Executableand Linking Format）是可执行和链接格式，它是一个开放标准，各种 UNIX 系统的可执行文件大多采用 ELF 格式。虽然 ELF 文件本身就支持三种不同的类型（重定位、执行、共享），不同的视图下格式稍微不同，不过它有一个统一的结构，这个结构如下图所示。

![](./assets/c291aa63-3256-492e-b2c2-8f4b5999b8d5.jpg)

我们主要关心“.plt”和“.got”这两个节区：

* **.plt**。该节保存过程链接表（Procedure Linkage Table）。
* **.got**。该节保存着全局的偏移量表。

我们也可以使用`readelf -S`来查看 ELF 文件的具体信息。