##1. 使用共享C++ 运行时库

在NDK中使用 C++ 时，请使用最新 NDK，并选择使用 libc++ 共享STL 作为 C++ STL，这样可以使多个 so 共享一个 C++ STL，因为使用静态STL将会在每个 so 库中出现重复代码，增加应用大小，并且由于全局数据和静态构造函数在内的 STL 将同时存在于两个库中。此应用的运行时行为未定义，因此在实际运行过程中，应用会经常崩溃。例如：内存在一个库中分配，而在另一个库中释放，从而导致内存泄漏或堆损坏。

###1.1. 解决方案

- ndk-build

在 Application.mk 中添加

```cmake
APP_STL := c++_shared
```

- cmake

在 build.gradle 中添加

```cmake
externalNativeBuild {
    cmake {
        arguments "-DANDROID_STL=c++_shared"
  }
}
```

###1.2. libc++_shared.so 冲突解决

使用  c++_shared 编译后 aar 中将附带 libc++_shared.so，如果有多个aar 都附带了  libc++_shared.so，将导致APP工程中编译不过，此时可以采取下面两种办法：

- APP 工程中 build.gradle 中添加编译配置

```cmake
packagingOptions {
    pickFirst 'lib/*/libc++_shared.so'
}
```

- 在编译 so 库的工程 build.gradle 中添加配置排除  libc++_shared.so,  由App工程负责添加 libc++_shared.so ：

```cmake
libraryVariants.all { variant ->
    def bundleTask = variant.packageLibrary
    def reBundleTask = tasks.create("reBundle${variant.name.capitalize()}Aar", Zip) {
        def dir = bundleTask.destinationDir
        def path = bundleTask.archivePath
        def name = bundleTask.archiveName
        archiveName name + ".bak"
        destinationDir dir
        from(zipTree(path)) {
            exclude "jni/*/libc++_shared.so"
        }
        doLast {
            delete path
            file("$dir/$name" + ".bak").renameTo(path)
        }
    }
    bundleTask.finalizedBy reBundleTask
    variant.assemble.dependsOn reBundleTask
}
```

##2. 开启编译优化

如果使用 GCC 可以 -Os 打开优化，如果使用 Clang 可以 -Oz 打开优化

###2.1. 解决方案

- ndk-build

在 Android.mk 中添加

```cmake
LOCAL_CFLAGS += -Os -Oz
LOCAL_CPPFLAGS += -Os -Oz
```

- cmake

在 build.gradle 中添加

```cmake
externalNativeBuild {
    cmake {
        cFlags "-Os -Oz"
        cppFlags "-Os -Oz"
    }
}
```

##3. 隐藏符号

```cmake
-fvisibility=hidden
```

隐藏elf符号表，可以减少 so 文件大小，提升性能 注意：需要在提供给 java 层暴露的JNI函数可以 在方法上添加 **JNIEXPORT** 宏或者添加属性 **attribute** ((visibility ("default")))

```cmake
-fvisibility-inlines-hidden
```

隐藏所有内联函数，从而减小导出符号表的大小，既能缩减文件的大小，还能提高运行性能

###3.1. 解决方案

- ndk-build

在 Android.mk 中添加

```cmake
LOCAL_CFLAGS += -fvisibility=hidden
LOCAL_CPPFLAGS += -fvisibility=hidden -fvisibility-inlines-hidden
```

- cmake 在 build.gradle 中添加

```cmake
externalNativeBuild {
    cmake {
        cFlags "-fvisibility=hidden"
 		cppFlags "-fvisibility=hidden -fvisibility-inlines-hidden"
 	}
}
```

##4. 删除无用函数

使用 --gc-section 编译选项减小程序体积

解决方案

- ndk-build

在 Android.mk 中添加

```cmake
LOCAL_CPPFLAGS += -ffunction-sections -fdata-sections
LOCAL_CFLAGS += -ffunction-sections -fdata-sections 
LOCAL_LDFLAGS += -Wl,--gc-sections
```

- cmake

在 build.gradle 中添加

```cmake
externalNativeBuild {
    cmake {
        cFlags "-ffunction-sections -fdata-sections"
        cppFlags "-ffunction-sections -fdata-sections"
    }
}
```

##5. 不使用异常和RTTI

在C++ 中使用异常和 RTTI 会显著增加文件大小，因此尽量不要在 C++中 使用异常和 RTTI，如果代码中没有使用到 异常和 RTTI 请删除相关 feature 和 flags

###5.1. 解决方案

- ndk-build

```cmake
从 LOCAL_CPP_FEATURES 中 删除 exceptions 和 rtti
从 APP_CPPFLAGS 中 删除 -frtti 和 -fexceptions
从 LOCAL_CPPFLAGS 中 删除 -frtti 和 -fexceptions
```

- cmake

```cmake
删除 cppFlags "-frtti"
删除 cppFlags "-fexceptions"
```

##6. 不使用iostream

尽量不要在 C++ 代码中使用 iostream，如：

```cmake
#include<iostream>
std::cout << "test" <<std::endl;
```

其他类似用法：`std::cin`，`std::cout`，`std::cerr`，`std::clog`，`std::wcin`，`std::wcout`，`std::wcerr`，`std::wclog`

这样没有什么作用并且会显著增加 so 文件大小

###6.1. 解决方案

删除 iostream 相关代码调用，如果有打日志需要可以使用 __android_log_print

```
#define LOG_TAG "native_log"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
LOGE("AndroidBitmap_getInfo() failed ! error=%d", ret);
```
##参考
https://juejin.cn/post/6844903889683103757