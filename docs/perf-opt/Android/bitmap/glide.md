# Glide 核心原理速读
> 不陷于茫茫源码之中，只取其核心

## 前言
一般来说，一个优化的图片加载库，都会有以下几个特点：
* 支持多种图片格式，包括但不限于：`jpg`、`png`、`webp`、`gif`、`bmp`、`svg`等
* 支持多种图片尺寸，包括但不限于：`原图`、`缩略图`、`自定义尺寸`等
* 支持多种图片加载方式，包括但不限于：`同步加载`、`异步加载`、`预加载`、`缓存`等
* 支持多种图片显示方式，包括但不限于：`ImageView`、`Bitmap`、`Drawable`、`View`等
* 支持多种图片加载策略，包括但不限于：`内存缓存`、`磁盘缓存`、`网络加载`等
* 支持多种图片加载优化，包括但不限于：`线程池`、`缓存策略`、`内存优化`、`网络优化`等

## 总览
### 代码结构
Glide 的代码结构非常清晰，主要分为以下几个部分：
* `Glide`：Glide 的入口类，负责初始化、配置、加载图片等
* `RequestManager`：Glide 的请求管理类，负责管理图片加载请求
* `RequestBuilder`：Glide 的请求构建类，负责构建图片加载请求
* `RequestOptions`：Glide 的请求配置类，负责配置图片加载请求
* `Target`：Glide 的请求目标类，负责指定图片加载的目标
* `Engine`：Glide 的引擎类，负责图片加载的核心逻辑
* `EngineJob`：Glide 的引擎任务类，负责图片加载的具体逻辑
* `Resource`：Glide 的资源类，负责管理图片加载的资源
* `Cache`：Glide 的缓存类，负责管理图片加载的缓存
* `DiskCache`：Glide 的磁盘缓存类，负责管理图片加载的磁盘缓存
* `MemoryCache`：Glide 的内存缓存类，负责管理图片加载的内存缓存
* `BitmapPool`：Glide 的位图池类，负责管理图片加载的位图池
* `Transformation`：Glide 的转换类，负责管理图片加载的转换
* `TransformationUtils`：Glide 的转换工具类，负责管理图片加载的转换工具

一般来说，我们使用如下代码加载一张网络图片：
``` java
Glide.with(this).load(url).into(imgView);
 ```       
流程大概如下：
![](./assets/glide_process.png)

### 加载源码简述

1. 通过 `RequestManager` 构建 `RequestBuilder`
``` java
public static RequestManager with(@NonNull Context context) {  
     return getRetriever(context).get(context);
    }

 public RequestBuilder<Drawable> load(@Nullable String string) { 
    return asDrawable().load(string);
    }

```
2.  通过 `RequestBuilder` 构建 `Target`
``` java
public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {    
    ....
    return into(     
         glideContext.buildImageViewTarget(view, transcodeClass),      
         /*targetListener=*/ null,     
          requestOptions,     
           Executors.mainThreadExecutor());
    }

  private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> options,
      Executor callbackExecutor) {
    ...
    requestManager.track(target, request);

    return target;
  }   

```
3. 通过 `Target` 构建 `EngineJob`
`requestManager#track`最终会调用到`SingleRequest#begin`, 而真正的请求 是在 `SingleRequest#onSizeReady` 开始的
``` java
  public void onSizeReady(int width, int height) {
	  ....
      loadStatus =
          engine.load(
              glideContext,
              model,
              requestOptions.getSignature(),
              this.width,
              this.height,
              requestOptions.getResourceClass(),
              transcodeClass,
              priority,
              requestOptions.getDiskCacheStrategy(),
              requestOptions.getTransformations(),
              requestOptions.isTransformationRequired(),
              requestOptions.isScaleOnlyOrNoTransform(),
              requestOptions.getOptions(),
              requestOptions.isMemoryCacheable(),
              requestOptions.getUseUnlimitedSourceGeneratorsPool(),
              requestOptions.getUseAnimationPool(),
              requestOptions.getOnlyRetrieveFromCache(),
              this,
              callbackExecutor);
		...
   
    }

  public class LoadStatus {
    private final EngineJob<?> engineJob;
    private final ResourceCallback cb;
	...

  }
```
4. 通过`Engine`执行`EngineJob`
`Engine`
```java
 public <R> LoadStatus load(...) {
    ...
    synchronized (this) {
   // 尝试从内存缓存中获取
      memoryResource = loadFromMemory(key, isMemoryCacheable, startTime);

      if (memoryResource == null) {
        // 没有缓存，执行加载job
        return waitForExistingOrStartNewJob(...);
      }
    }
    ...
    return null;
  } 

private <R> LoadStatus waitForExistingOrStartNewJob(cb, ....) {

   ...
    //创建一个执行工作
    EngineJob<R> engineJob =
        engineJobFactory.build(
         ...);
    //创建一个解码工作
    DecodeJob<R> decodeJob =
        decodeJobFactory.build(...);

    jobs.put(key, engineJob);
     // 注册ResourceCallback接口，就是在成功获取图片后，需要显示到ImageView 上的回调，这个接口回调到SingleRequest 中
    engineJob.addCallback(cb, callbackExecutor);
    //开始执行
    engineJob.start(decodeJob);


    return new LoadStatus(cb, engineJob);
  }  
```
5. 通过`DecodeJob`执行`DecodePath`



6. 通过 `Resource` 构建 `Bitmap`
7. 通过 `Bitmap` 构建 `Drawable`
8. 通过 `Drawable` 构建 `ImageView`
9. 通过 `ImageView` 显示图片




<!-- Glide 的加载流程大致如下：
1. 通过 `RequestManager` 构建 `RequestBuilder`，并通过 `RequestBuilder` 构建 `RequestOptions`，最后通过 `RequestOptions` 构建 `Target`，最后通过 `Target` 构建 `EngineJob`，最后通过 `EngineJob` 构建 `Engine`，最后通过 `Engine` 构建 `Resource`，最后通过 `Resource` 构建 `Drawable`，最后通过 `Drawable` 构建 `Bitmap`，最后通过 `Bitmap` 构建 `ImageView`，最后通过 `ImageView` 显示图片。 -->

## 图片缓存策略

### 内存缓存
Glide 的内存缓存策略是通过 `MemoryCache` 来实现的，它的实现类是 `LruResourceCache`，它的内部实现是通过 `LinkedHashMap` 来实现的，它的缓存策略是 `LRU` 算法，即最近最少使用的缓存会被移除。它的缓存大小是通过 `MemorySizeCalculator` 来计算的，它的计算公式如下：
``` java
int maxSize = (int) (maxSizeMultiplier * maxMemory);
maxSize = Math.max(maxSize, minSize);
maxSize = Math.min(maxSize, maxSize);
 ```

### 磁盘缓存
Glide 的磁盘缓存策略是通过 `DiskCache` 来实现的，它的实现类是 `DiskLruCacheWrapper`，它的内部实现是通过 `DiskLruCache` 来实现的，它的缓存策略是 `LRU` 算法，即最近最少使用的缓存会被移除。它的缓存大小是通过 `DiskCache.Factory` 来计算的，它的计算公式如下：
``` java
long size = 0;
try {
    StatFs statFs = new StatFs(directory.getAbsolutePath());
    size = (long) statFs.getBlockCount() * (long) statFs.getBlockSize();
} catch (Throwable ignored) {
}
size = Math.max(size, MIN_DISK_CACHE_SIZE);
size = Math.min(size, MAX_DISK_CACHE_SIZE);
return size;
 ```
其中，`MIN_DISK_CACHE_SIZE` 的默认值是 5MB，`MAX_DISK_CACHE_SIZE` 的默认值是 250MB。

### 位图池
Glide 的位图池策略是通过 `BitmapPool` 来实现的，它的实现类是 `LruBitmapPool`，它的内部实现是通过 `LruPoolStrategy` 来实现的，它的缓存策略是 `LRU` 算法，即最近最少使用的缓存会被移除。它的缓存大小是通过 `MemorySizeCalculator` 来计算的，它的计算公式如下：
``` java
int maxSize = (int) (maxSizeMultiplier * maxMemory);
maxSize = Math.max(maxSize, minSize);
maxSize = Math.min(maxSize, maxSize);
 ```
其中，`maxSizeMultiplier` 的默认值是 0.4，`minSize` 的默认值是 4MB，`maxSize` 的默认值是 32MB。




