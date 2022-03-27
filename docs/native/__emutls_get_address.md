> 转自：https://zhuanlan.zhihu.com/p/392288963

最近遇到一个比较诡异的问题，有一个动态链接库，之前是静态链接的STL（c++_static），改成动态链接STL（c++_shared）之后，在>=Android 11的手机上会报dlopen报找不到__emutls_get_address符号。

首先用nm查看下这个动态链接库里的符号：

```shell
$ nm libxxx.so |grep __emutls_get_address
         U __emutls_get_address
```

这个U表示未定义，也就是符号的定义在其他库中。我们用的是ndk21b，到ndk的目录看下libc++_shared.so里的符号：

```shell
$ cd toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/lib/arm-linux-androideabi
$ nm libc++_shared.so |grep __emutls_get_address                                                        
00070364 t __emutls_get_address
```

这个小写的t表示是内部符号，不暴露给外部使用（暴露给外部的话应该是大写的T）。这个就有点奇怪了，我们的动态库里没有这个符号，然后依赖的libc++_shared.so里的符号又不对外暴露，肯定找不到啊。。。

最后查出来的原因是：

我们编译自己的动态链接库的时候，使用的是老版本ndk里的ibc++_shared.so，而打包进apk的，是ndk21b里的libc++_shared.so。

这个老版本的libc++_shared.so是会暴露这个__emutls_get_address的符号的。由于老版本的库没有带符号，因此不能用nm命令查看，但是我们可以用readelf查看：

```shell
$ readelf -s libc++_shared.so | grep __emutls_get_address
  2110: 000846a0   316 FUNC    GLOBAL DEFAULT   11 __emutls_get_address
```

可以发现，这个符号的可见性是GLOBAL DEFAULT。

我们可以readelf再看下ndk21b的libc++_shared.so里的符号：

```shell
$ readelf -s libc++_shared.so| grep __emutls_get_address                                  s
 37081: 00070364   324 FUNC    LOCAL  HIDDEN    13 __emutls_get_address
```

发现区别没，这里的可见性是LOCAL HIDDEN，所以不会对外暴露。事实上，在这种情况下，链接器会自动帮我们链接 libgcc_real.a，而libgcc_real.a里是包含这个符号的：

```shell
$ cd toolchains/llvm/prebuilt/darwin-x86_64/lib/gcc/arm-linux-androideabi/4.9.x/armv7-a
$ nm libgcc_real.a|grep __emutls_get_address
0000020c T __emutls_get_address
```

结论：

编译时用的stl版本一定要跟打包到apk里的stl版本保持一致，否则就会出现一些莫名其妙的版本兼容问题。

P.S. 如果不清楚符号在哪个库中，可以lldb attach到app上，然后用`image lookup --symbol 'xxx'`在所有依赖库中进行搜索。