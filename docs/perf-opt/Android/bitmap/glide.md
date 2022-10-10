# Glide 核心原理速读
> 不陷于茫茫源码之中，只取其核心

## 总览

一般来说，我们使用如下代码加载一张网络图片：
``` java
Glide.with(this).load(url).into(imgView);
 ```       
流程大概如下：
![](./assets/glide_process.png)



### Glide 特点

一般来说，一个优秀的图片加载库，都会有以下几个特点：
* 支持多种图片格式，如：`jpg`、`png`、`webp`、`gif` 等
* 支持多种数据源，如：`网络`、`本地`、`assets` 等
* 支持多种缓存策略，如：`内存`、`磁盘`、`网络` 等

而Glide处理具备以上特点外，还有以下特点：

- 生命周期感知（根据Activity或者Fragment的生命周期管理图片加载请求）
- 高效处理Bitmap（bitmap的复用和主动回收，减少系统回收压力）
- 更高效的缓存策略，加载速度快且内存开销小

## 核心原理

### Glide是如何感知生命周期的

1. 传入Activity/Fragment/Context
**`Glide#with`** 
``` java
 public static RequestManager with(@NonNull Activity activity) {
    return getRetriever(activity).get(activity); #1
  }
```
2. 关联生命周期
**`RequestManagerRetriever#get`**
```java
  public RequestManager get(@NonNull Activity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else if (activity instanceof FragmentActivity) {
      return get((FragmentActivity) activity);  #2
    } else {
      ....
    }
  }

    @NonNull
  public RequestManager get(@NonNull FragmentActivity activity) {
    ...
    //Glide 终于支持官方的lyfecycle方案了，虽然是实验性的，但不影响分析，就以这个为例吧
    if (useLifecycleInsteadOfInjectingFragments()) {
      Context context = activity.getApplicationContext();
      Glide glide = Glide.get(context);
      #4
      return lifecycleRequestManagerRetriever.getOrCreate(
          context,
          glide
          activity.getLifecycle(),
          activity.getSupportFragmentManager(),
          isActivityVisible);
    } else {
      ...
    }
  }
```
**`LifecycleRequestManagerRetriever#getOrCreate`**
```java
  RequestManager getOrCreate(
      Context context,
      Glide glide,
      final Lifecycle lifecycle,
      FragmentManager childFragmentManager,
      boolean isParentVisible) {
    Util.assertMainThread();
    RequestManager result = getOnly(lifecycle);
    if (result == null) {
      
      //LifecycleLifecycle 是关联Activity生命周期和RequestManager的桥梁
      #5 
      LifecycleLifecycle glideLifecycle = new LifecycleLifecycle(lifecycle);
      // 这里会构建RequestManager并且和Lifecycle关联
      #6
      result =
          factory.build(
              glide,
              glideLifecycle,
              new SupportRequestManagerTreeNode(childFragmentManager),
              context);
      ...
    }
    return result;
  }
```
2. 感知生命周期
**`RequestManager`**
```java
  RequestManager(
      Glide glide,
      Lifecycle lifecycle,
      RequestManagerTreeNode treeNode,
      RequestTracker requestTracker,
      ConnectivityMonitorFactory factory,
      Context context) {
   ...
    // 关联生命周期
    #7
      lifecycle.addListener(this);
   ...
  }

  // 感知生命周期处理请求.
  @Override
  public synchronized void onStart() {
    resumeRequests();
    targetTracker.onStart();
  }
  // 感知生命周期处理请求.
  @Override
  public synchronized void onStop() {
    pauseRequests();
    targetTracker.onStop();
  }
// 感知生命周期处理请求.
  @Override
  public synchronized void onDestroy() {
    targetTracker.onDestroy();
    for (Target<?> target : targetTracker.getAll()) {
      clear(target);
    }
    targetTracker.clear();
    requestTracker.clearRequests();
    lifecycle.removeListener(this);
    lifecycle.removeListener(connectivityMonitor);
    Util.removeCallbacksOnUiThread(addSelfToLifecycle);
    glide.unregisterRequestManager(this);
  }
```

**`LifecycleLifecycle`** 是关联Activity生命周期和RequestManager的桥梁

```java
final class LifecycleLifecycle implements Lifecycle, LifecycleObserver {
  @NonNull
  private final Set<LifecycleListener> lifecycleListeners = new HashSet<LifecycleListener>();

  @NonNull private final androidx.lifecycle.Lifecycle lifecycle;
  //这里是#4的 activity.getLifecycle()
  LifecycleLifecycle(androidx.lifecycle.Lifecycle lifecycle) {
    this.lifecycle = lifecycle;
    // LifecycleLifecycle 作为 LifecycleObserver 注册到 Activity Lifecycle
    lifecycle.addObserver(this);
  }

  @OnLifecycleEvent(Event.ON_START)
  public void onStart(@NonNull LifecycleOwner owner) {
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onStart();
    }
  }

  @OnLifecycleEvent(Event.ON_STOP)
  public void onStop(@NonNull LifecycleOwner owner) {
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onStop();
    }
  }

  @OnLifecycleEvent(Event.ON_DESTROY)
  public void onDestroy(@NonNull LifecycleOwner owner) {
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onDestroy();
    }
    owner.getLifecycle().removeObserver(this);
  }
  // RequestManager 作为 LifecycleListener 注册到 LifecycleLifecycle，从而间接注册到 Activity Lifecycle
  @Override
  public void addListener(@NonNull LifecycleListener listener) {
    lifecycleListeners.add(listener);

    if (lifecycle.getCurrentState() == State.DESTROYED) {
      listener.onDestroy();
    } else if (lifecycle.getCurrentState().isAtLeast(State.STARTED)) {
      listener.onStart();
    } else {
      listener.onStop();
    }
  }

  @Override
  public void removeListener(@NonNull LifecycleListener listener) {
    lifecycleListeners.remove(listener);
  }
}
```
#### 小结
1.获取对应Activity的`Lifecycle`, 把`LifecycleLifecycle`作为 LifecycleObserver 注册到 `Activity Lifecycle`；
2.在创建`RequestManager`时，RequestManager 作为 `LifecycleListener` 注册到 LifecycleLifecycle，从而间接注册到 Activity Lifecycle;
3.这样当Activity生命周期变化的时候，就能通过接口回调去通知RequestManager处理请求.

> 以上是Glide支持官方的lyfecycle方案的处理流程，其实默认Glide是自己实现了一套生命周期感知方案，这里不做分析，感兴趣的可以自己去看源码或者查看网上相关文章。

### 缓存策略
### 内存优化
### 扩展支持




