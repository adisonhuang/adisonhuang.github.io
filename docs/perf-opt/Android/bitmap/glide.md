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

