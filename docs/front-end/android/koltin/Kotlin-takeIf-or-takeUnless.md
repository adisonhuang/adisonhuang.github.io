# 使用Kotlin takeIf（或takeUnless）

在Kotlin的[标准函数](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt)，有两大函数，即`takeIf`和`takeUnless`，乍一看，有什么特别之处呢？这几乎就是`if`？

或者极端点，把每一个`if`语句改成类似下面（**不推荐**）。

```kotlin
//原始代码
if（status）{doThis（）}
//修改后的代码
takeIf {status}？apply {doThis（）}
```

### 深入探讨

像其他任何东西一样，`takeIf`（或`takeUnless`）确实有它的使用场景。我通过不同情况分享我对他们的理解。在此之前，让我们看看它的实现。

#### 函数签名

```kotlin
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? 
    = if (predicate(this)) this else null
```

从函数签名，我们注意到

1. 它是从T对象本身调用的。即`T.takeIf`，
2. `predicate`函数以T对象为参数
3. 等待`predicate`评估后它返回`this`或`null`。

### 合理的使用情况

基于以上特点，我可以推导出它相对于`if`的使用情况，如下：

#### 1.它是从T对象本身调用的。即`T.takeIf`，

它可以很好处理可空性检查。一个例子如下

```kotlin
//原始代码
if（someObject！= null && status）{ 
   doThis（）
}
//改进的代码
someObject？.takeIf {status}？apply {doThis（）}
```

#### 2.`predicate`函数以T对象为参数

由于将T作为`predicate`的参数，所以可以进一步简化`takeIf`代码

```kotlin
//原始代码
if（someObject！= null && someObject.status）{ 
   doThis（）
} 
//更好的代码
if（someObject？.status == true）{ 
   doThis（）
}
//改进的代码
someObject？.takeIf {it.status} ?. apply {doThis（）}
```

`更好的代码`的确还可以，但需要显式的`true`关键词，所以并不理想。

#### 3.等待`predicate`评估后它返回`this`或`null`

既然它返回`this`，那就可以用来进行链式调用。因此，下面代码可以优化

```kotlin
//原始代码
if（someObject！= null && someObject.status）{ 
   someObject.doThis（）
}
//改进的代码
someObject？.takeIf {status}？doThis（）
```

或者实现获取数据或退出的更好方式（例子从[Kotlin Doc](https://kotlinlang.org/docs/reference/whatsnew11.html#also-takeif-and-takeunless)中摘取）

```kotlin
val index 
   = input.indexOf（keyword）.takeIf {it> = 0}？：error（“Error”）
val outFile 
   = File（outputDir.path）.takeIf {it.exists（）}？：return  false
```

### 注意

看看下面的代码。

```kotlin
//语法上仍然正确。但逻辑错误！
someObject？.takeIf {status} .apply {doThis（）}

//正确的（注意可空性检查？）
someObject？.takeIf {status} ？.apply {doThis（）}
```

`doThis()`在第一行中不管`status`true 还是 false 都会执行。因为 即使`takeIf`返回`null`，它仍然会被调用。（这里假设`doThis()`不是`someObject`的函数）

所以在这里，第二行的`?`  是非常微妙且重要的。