# Kotlin中inline, noinline, crossinline的区别

我们在Kotlin中使用[高阶函数](https://www.kotlincn.net/docs/reference/lambdas.html)会带来一些运行时的效率损失：每一个函数都是一个对象，并且会捕获一个闭包。 即那些在函数体内会访问到的变量。 内存分配（对于函数对象和类）和虚拟调用会引入运行时间开销。

调用一个方法是一个压栈和出栈的过程，调用方法时将栈针压入方法栈，然后执行方法体，方法结束时将栈针出栈，这个压栈和出栈的过程会耗费资源，这个过程中传递形参也会耗费资源。

```kotlin
fun <T> lock(l: Lock, body: () -> T): T {
      l.lock()
      try {
          return body()
      } finally {
          l.unlock()
      }
  }
```

这个lock方法的方法体中，不会将它的形参再传递给其他方法。我们调用一下lock方法：

```kotlin
lock(l, {"do something!"})//l是一个Lock对象
```

对于编译器来说，调用lock方法就要将参数l和lambda表达式{“do something!”}进行传递，还要将lock方法进行压栈出栈处理，这个过程就会耗费资源。如果我们把lock方法删除，直接执行lock方法的方法体：

```kotlin
l.lock()
try {
    return "do something!"
} finally {
    l.unlock()
}
```

这样做的效果和调用lock方法是一样的，而且不需要压栈出栈了，但是如果代码中频繁调用lock方法，必然要复制粘贴大量重复代码，还不如抽取出来一个lock方法，但是如果lock方法被频繁调用，压栈出栈将会带来性能问题。

那有什么方法，让程序避免写冗余代码，又能够提高效率呢？针对这个问题，kotlin引入了`inline`关键字。

### inline

在许多情况下通过内联化 lambda 表达式可以消除压栈出栈的开销。 下述函数是这种情况的很好的例子。即 lock() 函数可以很容易地在调用处内联。 考虑下面的情况：

```kotlin
lock(l) { foo() }
```

编译器没有为参数创建一个函数对象并生成一个调用。取而代之，编译器可以生成以下代码：

```kotlin
l.lock()
try {
    foo()
 } finally {
    l.unlock()
}
```

这个不是我们从一开始就想要的吗？为了让编译器这么做，我们需要使用 inline 修饰符标记 lock() 函数：

```kotlin
inline fun <T> lock(l: Lock, body: () -> T): T { …… }
```

inline 修饰符影响函数本身和传给它的 lambda 表达式：所有这些都将内联到调用处。但是内联可能导致生成的代码增加，不过如果我们使用得当（即避免内联过大函数），性能上会有所提升。

另外`inline`也有个缺点，我们可以在 lambda 表达式 中调用return返回，可能会导致`inline`之后的代码无法执行 ，比如下面的代码 ：

```kotlin
inline fun higherOrderFunction(aLambda: () -> Unit) {
    aLambda()
    println("after invoke lambda") // 1
}

fun callingFunction() {
    higherOrderFunction {
        println("invoke lambda...") // 2
        return
    }
    println("after callingFunction...") // 3
}

// 打印结果 
// "invoke lambda..."
```



当我们调用 `callingFunction()` ，只会打印”invoke lambda…”，说明 #1 、#3 的代码被跳过了，所以在使用`inline`函数中包含lambda静态式的时候，要避免在lambda中使用return ，应该使用`return@higherOrderFunction`的方式:

```kotlin
inline fun higherOrderFunction(aLambda: () -> Unit) {
    aLambda()
    println("after invoke lambda")
}

fun callingFunction() {
    higherOrderFunction {
        println("invoke lambda...")
        return@higherOrderFunction
    }
    println("after callingFunction...")
}

// 打印结果 
// invoke lambda...
// after invoke lambda
// after callingFunction...
```



使用`return@higherOrderFunction`的方式返回得到的结果正确。不过kotlin也有另一种方式来限制在lambda中直接return，那就是使用 noinline 或 crossinline 。

### noinline

如果你只想被（作为参数）传给一个内联函数的 lamda 表达式中只有一些被内联，你可以用 `noinline` 修饰符标记某些lambda表达式禁止内联：

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { …… }
```

可以内联的 lambda 表达式只能在内联函数内部调用或者作为可内联的参数传递， 但是 `noinline` 的可以以任何我们喜欢的方式操作：存储在字段中、传送它等等。

另外，定义了`noinline` 的lambda表达式将不能直接调用 return ，否则会报错 ：

```kotlin
inline fun higherOrderFunction(noinline aLambda: () -> Unit) {
    aLambda()
    println("after invoke lambda")
}

fun callingFunction() {
    higherOrderFunction {
        println("invoke lambda...")
        return  // 编译不通过，`noinline` 的lambda表达式不能直接 return
    }
    println("after callingFunction...")
}
```



需要注意的是，如果一个内联函数没有可内联的函数参数并且没有[具体化的类型参数](https://www.kotlincn.net/docs/reference/inline-functions.html#具体化的类型参数)，编译器会产生一个警告，因为内联这样的函数很可能并无益处（如果你确认需要内联，则可以用 `@Suppress("NOTHING_TO_INLINE")` 注解关掉该警告）。

### crossinline

使用`crossinline`也可以限制lambda表达式将不能直接调用 return ，它与 `noinline` 的区别在于使用`crossinline`的lambda仍然是`inline`的，比如下面使用`noinline` 的代码：

```kotlin
inline fun higherOrderFunction(aLambda1: () -> Unit, noinline aLambda2: () -> Unit) {
    aLambda1()
    aLambda2()
    println("after invoke lambda")
}

fun callingFunction() {
    higherOrderFunction({
        println("invoke lambda1...")
        return@higherOrderFunction
    }, {
        println("invoke lambda2...")
        return@higherOrderFunction
    })
    println("after callingFunction...")
}
```



其中aLambda1是`inline`的，而aLambda2是`noinline`的，查看编译后的代码：

```kotlin
public static final void higherOrderFunction(@NotNull Function0 aLambda1, @NotNull Function0 aLambda2) {
      Intrinsics.checkParameterIsNotNull(aLambda1, "aLambda1");
      Intrinsics.checkParameterIsNotNull(aLambda2, "aLambda2");
      aLambda1.invoke();
      aLambda2.invoke();
      String var3 = "after invoke lambda";
      System.out.println(var3);
   }

   public static final void callingFunction() {
      Function0 aLambda2$iv = (Function0)null.INSTANCE;
      String var1 = "invoke lambda1...";
      System.out.println(var1);
      aLambda2$iv.invoke();
      var1 = "after invoke lambda";
      System.out.println(var1);
      String var4 = "after callingFunction...";
      System.out.println(var4);
   }
```



从上面的代码我们可以发现aLambda1被内联到调用函数`callingFunction`中，而aLambda2并没有被内联到调用函数`callingFunction`中。如果我们把`noinline`修改成`crossinline`，如下所示：

```kotlin
inline fun higherOrderFunction(aLambda1: () -> Unit, crossinline aLambda2: () -> Unit) {
    aLambda1()
    aLambda2()
    println("after invoke lambda")
}

fun callingFunction() {
    higherOrderFunction({
        println("invoke lambda1...")
        return@higherOrderFunction
    }, {
        println("invoke lambda2...")
        return@higherOrderFunction
    })
    println("after callingFunction...")
}
```

同样aLambda1是`inline`的，而aLambda2是`crossinline`的，查看编译后的代码：

```kotlin
public static final void higherOrderFunction(@NotNull Function0 aLambda1, @NotNull Function0 aLambda2) {
     Intrinsics.checkParameterIsNotNull(aLambda1, "aLambda1");
     Intrinsics.checkParameterIsNotNull(aLambda2, "aLambda2");
     aLambda1.invoke();
     aLambda2.invoke();
     String var3 = "after invoke lambda";
     System.out.println(var3);
  }

  public static final void callingFunction() {
     String var0 = "invoke lambda1...";
     System.out.println(var0);
     var0 = "invoke lambda2...";
     System.out.println(var0);
     var0 = "after invoke lambda";
     System.out.println(var0);
     var0 = "after callingFunction...";
     System.out.println(var0);
  }
```



从上面的代码我们可以发现aLambda1和 aLambda2都被内联到调用函数`callingFunction`中了。

### 总结

本文主要介绍`inline`, `noinline`, `crossinline`的区别。为了减少使用高阶函数带来的一些运行时的效率损失，可以使用`inline`来标记一个函数为内联函数，内联函数在编译后会把代码都插入到调用函数的地方，所以可能导致最终的代码增加，所以必须避免内联过大函数才能使内联函数性能提升。另外我们可以用`noinline`和`crossinline`来限制内联函数中的lambda直接调用return返回，避免出现预料之外的结果。

### 参考

- [Kotlin中inline, noinline, crossinline的区别](https://zhooker.github.io/2018/10/15/Kotlin%E4%B8%ADinline-noinline-crossinline%E7%9A%84%E5%8C%BA%E5%88%AB/)