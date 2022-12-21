# Android 内存综合治理

## 1. 内存基础监控

### 1.1 简介
内存基础监控主要负责对客户端的整体内存状态进行监控，借助线上海量用户的复杂案例来帮助我们发现在开发及测试阶段难以发现的问题，同时也能够帮助我们对客户端的内存使用情况进行分析，从而帮助我们优化客户端的内存使用情况。
``
监控模块主要实现对`虚拟内存`、`Java堆`、`FD数量`、`线程数量`、`PSS`、`Native Heap`的监控，根据系统限制或设置固定数值，定期查看其状态，同时我们把页面跳转时记录的内存数据记录在链路中进行上报，从而统计出应用运行过程中各个业务场景的内存状态。

### 1.2. 核心源码
```kotlin
// FD数量,耗时一般
fun getFds(): Int {
    lastFDCount = 0
    runCatching {
        if (appFdFile == null) {
            appFdFile = File("/proc/self/fd")
        }
        appFdFile?.listFiles { _, name -> TextUtils.isDigitsOnly(name) }
            ?.size ?: 0
    }.getOrDefault(0).also { lastFDCount = it }
    return lastFDCount
}


/**
 * Get Threads.
 * 耗时一般
 */
fun getProcessStatus(): ProcessStatus {
    val processStatus = ProcessStatus()
    runCatching {
        if (appStatusFile == null) {
            appStatusFile = File("/proc/self/status")
        }
        appStatusFile?.useLines {
            it.forEach { line ->
                when {
                    line.startsWith("VmSize") -> processStatus.vssKbSize = VSS_REGEX.matchValue(line)
                    line.startsWith("VmRSS") -> processStatus.rssKbSize = RSS_REGEX.matchValue(line)
                    line.startsWith("Threads") ->
                        processStatus.threadsCount = THREADS_REGEX.matchValue(line)
                }
            }
        }
    }
    lastProcessStatus = processStatus
    return processStatus
}


/**
 *  获取内存信息 比较耗时
 */
fun getProcessMemoryInfo(): ProcessMemInfo {
    val memInfo = ProcessMemInfo()
    runCatching {
        val mi = Debug.MemoryInfo()
        Debug.getMemoryInfo(mi)
        if (Build.VERSION.SDK_INT >= 23) {
            memInfo.javaHeapInKb = mi.getMemoryStat("summary.java-heap").toLong()
            memInfo.nativeHeapInKb = mi.getMemoryStat("summary.native-heap").toLong()
            memInfo.code = mi.getMemoryStat("summary.code").toLong()
            memInfo.stack = mi.getMemoryStat("summary.stack").toLong()
            memInfo.graphics = mi.getMemoryStat("summary.graphics").toLong()
            memInfo.system = mi.getMemoryStat("summary.system").toLong()
            memInfo.totalPssInKb = mi.getMemoryStat("summary.total-pss").toLong()
            memInfo.totalSwap = mi.getMemoryStat("summary.total-swap").toLong()
        } else {
            memInfo.javaHeapInKb = mi.dalvikPrivateDirty.toLong()
            memInfo.nativeHeapInKb = mi.nativePrivateDirty.toLong()
//            val otherPrivateDirty = mi.otherPrivateDirty
            memInfo.system = if (Build.VERSION.SDK_INT >= 19) {
                mi.totalPss - mi.totalPrivateDirty - mi.totalPrivateClean
            } else {
                mi.totalPss - mi.totalPrivateDirty
            }.toLong()
            memInfo.totalPssInKb = mi.totalPss.toLong()
        }
    }
    lastProcessMemInfo = memInfo
    return lastProcessMemInfo
}


```

## 2. 图片检测&监控

### 2.1. 简介
假设屏幕分辨率 `1080x1920`，一张 Bitmap 占满屏幕时内存大小是`1080x1920x4`，大约 8MB。所以一个不起眼的 Bitmap 占用的内存也远远大于多数对象。可见，如果应用内有许多不合理的图片，就会拉高内存水位。如下图， 目前88%以上的用户的手机是8.0及更高系统版本，它的内存申请是在Native层的。因此，图片内存的分析重点放在8.0以及更高的系统版本上。
![](./assets/20221219-110541.jpeg)

从[Bitmap: 从出生到死亡](https://blog.adison.top/perf-opt/Android/bitmap/bitmap-from-birth-to-death/#3-bitmap) 可以知道， 


无论哪种Bitmap创建方式，最终殊途同归。都会通过`Bitmap.cpp`中的`Bitmap::createBitmap`方法创建Bitmap对象。

所以，我们只需要hook这个`createBitmap`p函数，就能够拿到每次图片创建时的bitmap的Java对象。通过该对象，可以获得`图片的尺寸大小`、`内存占用大小`，`堆栈`等信息，将这些信息上报到性能平台也就达到了检测与监控图片创建的目标。

这里hook的方案采用爱奇艺开源的`plt-hook`开源方案[xHook](https://github.com/iqiyi/xHook)来完成。关于`plt-hook`具体原理见[Native-Hook](https://blog.adison.top/perf-opt/Android/memory/Native-Hook/)。


那怎么查找到这个函数呢？

首先需要通过 `adb pull system/lib/libandroid_runtime.so` 拿到系统的so文件，然后通过`arm-linux-androideabi-nm -D libandroid_runtime.so | grep bitmap` ，可以查找到对应的函数名，也就是我们要hook的函数，系统版本不同可能会有差异，注意兼容。

![](./assets/38p4h4ny9u.png)

### 2.2. 实现

核心源码如下：

```c++
  jobject (*old_create_bitmap_fun)(JNIEnv *env, void *bitmap,
                                     int bitmapCreateFlags, jbyteArray ninePatchChunk,
                                     jobject ninePatchInsets,
                                     int density);
    void Hooker::InitHook() {
        bool hooked = false;
        xhook_clear();
        int result = xhook_register(hook_lib,
                                    GetCreateBitmapSymbol(),
                                    reinterpret_cast<void *>(HookBitmapCreate),
                                    reinterpret_cast<void **>(&old_create_bitmap_fun));
        result != 0 ? hooked = false : hooked = true;
        if (hooked) {
            int result = xhook_refresh(0);
        }
    }

    const char *Hooker::GetCreateBitmapSymbol() {
        if (android_api >= __ANDROID_API_O__) {
            return "_ZN7android6bitmap12createBitmapEP7_JNIEnvPNS_6BitmapEiP11_jbyteArrayP8_jobjecti";
        } else if (android_api >= __ANDROID_API_M__) {
            return "_ZN11GraphicsJNI12createBitmapEP7_JNIEnvPN7android6BitmapEiP11_jbyteArrayP8_jobjecti";
        } else if (android_api >= __ANDROID_API_L__) {
            return "_ZN11GraphicsJNI12createBitmapEP7_JNIEnvP8SkBitmapP11_jbyteArrayiS5_P8_jobjecti";
        } else if (android_api >= __ANDROID_API_K__) {
            return "_ZN11GraphicsJNI12createBitmapEP7_JNIEnvP8SkBitmapP11_jbyteArrayiS5_P10_jintArrayi";
        } else {
            bitmapProfiler::Log::error(TAG, "SDK :%d not support", android_api);
            return "_ZN7android6bitmap12createBitmapEP7_JNIEnvPNS_6BitmapEiP11_jbyteArrayP8_jobjecti";
        }
    }

     jobject Hooker::HookBitmapCreate(JNIEnv *env, void *bitmap, int bitmapCreateFlags,
                                     jbyteArray ninePatchChunk, jobject ninePatchInsets,
                                     int density) {

        jobject obj = old_create_bitmap_fun(env, bitmap, bitmapCreateFlags,
                                            ninePatchChunk, ninePatchInsets, density);

        if (hookEnabled()) {
           ...
           // 这里把hook到的bitmap传递到java层处理
        }

        return obj;
    }
   
 ```       

这样我们就得知经过app创建的所有Bitmap的内存大小。

我们还可以更进一步，获取图片所在view的宽高，有时由于一些错误处理，会导致一个很小的view里面承载了一个内存宽高比它本身宽高大得多的图片，这样的话，解码图片时按照内存宽高来解码就很浪费了；另外，我们也可以获取图片文件大小。

我们可以通过字节码插桩技术来实现。

以hook `Okhttp`和`Glide`为例，核心源码如下:

**ASM 相关代码**
```java
public class ImageProfilerClassVisitor extends BaseClassVisitor {
    
    ...

    @Override
    public MethodVisitor visitMethod(int access, String methodName, String desc, String signature, String[] exceptions) {
        MethodVisitor mv = cv.visitMethod(access, methodName, desc, signature, exceptions);
   
        if(mClassName.equals("com/bumptech/glide/request/SingleRequest") && (methodName.equals("<init>") || methodName.equals( "init")) && desc!=null){
            return mv == null ? null : new GlideMethodAdapter(mv,access,methodName,desc);
        }

        if(mClassName.equals("okhttp3/OkHttpClient$Builder") && methodName.equals("<init>") && desc.equals("()V")){
            return mv == null ? null : new OkHttpMethodAdapter(mv,access,methodName,desc);
        }

        return mv;
    }

}

public class OkHttpMethodAdapter extends AdviceAdapter {

    ...

    @Override
    protected void onMethodExit(int opcode) {
        super.onMethodExit(opcode);
        //添加应用拦截器
        mv.visitVarInsn(ALOAD, 0);
        mv.visitFieldInsn(GETFIELD, "okhttp3/OkHttpClient$Builder", "interceptors", "Ljava/util/List;");
        mv.visitMethodInsn(INVOKESTATIC, "[pageageName]/instrument/image/ImageTracer", "getInstance", "()L[pageageName]/instrument/image/ImageTracer;", false);
        mv.visitMethodInsn(INVOKEVIRTUAL, "[pageageName]/instrument/image/ImageTracer", "getOkHttpInterceptors", "()Ljava/util/List;", false);
        mv.visitMethodInsn(INVOKEINTERFACE, "java/util/List", "addAll", "(Ljava/util/Collection;)Z", true);
        mv.visitInsn(POP);
    }
}

public class GlideMethodAdapter extends AdviceAdapter {

     ...
    @Override
    protected void onMethodExit(int opcode) {
        super.onMethodExit(opcode);
        mv.visitVarInsn(ALOAD, 0);
        mv.visitMethodInsn(INVOKESTATIC, "[pageageName]/instrument/image/aop/glide/GlideHook", "proxy", "(Ljava/lang/Object;)V", false);
    }
}

```

**代理实现**

```java

public class ImageInterceptor implements Interceptor {


    @Override
    public Response intercept(Chain chain) throws IOException {
        //发出请求时不拦截
        Request request = chain.request();
        Response response = chain.proceed(request);
        //拦截响应
        String header = response.header("Content-Type");
        //如果是图片类型则拦截
        try {
            if(isImage(header) && ImageMonitorManager.INSTANCE.getInstalled()){
                process(response);
            }
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return response;
    }

    private void process(Response response){
        String header = response.header("Content-Length");
         ...
        //获取图片大小
        ImageDataManager.INSTANCE.saveImageSizeInfo(response.request().url().toString(), Integer.parseInt(header));
    }

  ...
}


public class GlideHook  {

    public static void proxy(Object singleRequest) {
        try {
        ...
        requestListeners.add(new GlideImageListener());
        ...
    }
}

public class GlideImageListener<R> implements RequestListener<R> {


    @Override
    public boolean onResourceReady(R resource, Object model, Target<R> target, DataSource dataSource, boolean isFirstResource) {
         if(target instanceof CustomViewTarget || target instanceof ViewTarget){
                View view;
                if (target instanceof ViewTarget){
                    view = ((ViewTarget) target).getView();
                }else {
                    view = ((CustomViewTarget) target).getView();
                }
                target.getSize(new SizeReadyCallback() {
                    @Override
                    public void onSizeReady(int width, int height) {
                          if (resource == null)return;
                          // 获取view的宽高
                          ImageDataManager.INSTANCE.saveImageInfo(resource,theadName,width,height,"Glide",callStack,view,model.toString());
                    }
                });
            }
        return false;
    }
}

```
> 对于通过图片控件方法直接设置图片的情况，如`setImageDrawable`，`setImageBitmap`等，我们可以通过hook控件的相关方法来实现。这里要求项目最好有一套自己的控件系统，这样可以方便统一处理。

### 2.3. 小结

这样，我们就能得到应用创建的`Bitmap的内存大小`，`加载的view的大小`、`图片文件大小`等信息，通过这些信息，我们可以对图片进行优化，比如：
1. 是否存在过大内存的图片
    在清晰度要求不高的场景下，可以适当的缩小原始原图, 譬如一些磨砂模糊图片
2. 是否存在Bitmap的内存宽高远大于view大小的情况
    譬如，Glide是根据控件大小来为 transform 自动计算出具体的 width 和 height。对于宽高为 wrap_content 的控件，自动计算出的结果是当前屏幕大小，所以使用Glide加载的时候，控件布局尽量设置宽高，避免出现过大的图片
3. 是否存在过大的图片文件
   可以直接从云端获取指定宽高的图片 
4. 是否存在重复Bitmap
    可以考虑复用
5. Bitmap是否存在内存泄漏，是否正常回收
    譬如 `bitmap拷贝行为，通过源bitmap对象获得变换后的bitmap对象，这里需要考虑源bitmap是否可以立即释放。`

我们可以做成一个工具，方便查看定位问题，效果如下：

![](./assets/e9e03e8a-a0fc-4803-92a3-30367261def6.png)

同时，上报性能监控平台数据，可以做版本数据对比以及一键提bug

!!! note "关于上面提到设置为wrap_content导致Glide加载图片内存过大问题源码"

    ```java
    # ViewTarget.java
    void getSize(@NonNull SizeReadyCallback cb) {
        int currentWidth = this.getTargetWidth();
        int currentHeight = this.getTargetHeight();
        ...
    }

    private int getTargetWidth() {
        ...
        return this.getTargetDimen(this.view.getWidth(), layoutParamSize, horizontalPadding);
    }

    private int getTargetDimen(int viewSize, int paramSize, int paddingSize) {
        int adjustedParamSize = paramSize - paddingSize;
        if (adjustedParamSize > 0) {
            return adjustedParamSize;
        } else if (this.waitForLayout && this.view.isLayoutRequested()) {
            return 0;
        } else {
            int adjustedViewSize = viewSize - paddingSize;
            if (adjustedViewSize > 0) {
                return adjustedViewSize;
            } else if (!this.view.isLayoutRequested() && paramSize == -2) {
                if (Log.isLoggable("ViewTarget", 4)) {
                    Log.i("ViewTarget", "Glide treats LayoutParams.WRAP_CONTENT as a request for an image the size of this device's screen dimensions. If you want to load the original image and are ok with the corresponding memory cost and OOMs (depending on the input size), use override(Target.SIZE_ORIGINAL). Otherwise, use LayoutParams.MATCH_PARENT, set layout_width and layout_height to fixed dimension, or use .override() with fixed dimensions.");
                }
                // 当设置LayoutParams.WRAP_CONTENT会走进这里，这里会返回当前屏幕的宽高，上面log也有明确提示
                return getMaxDisplayLength(this.view.getContext());
            } else {
                return 0;
            }
        }
    }
    ```

## 3. 资源泄露/触顶

### 3.1. 线程泄露

### 3.2. 资源使用触顶监控


## 参考

