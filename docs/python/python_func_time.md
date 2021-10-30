# python性能优化之函数执行时间分析

最近发现项目API请求比较慢，通过抓包发现主要是response时间太长，于是就开始进行优化工作。优化工作的关键一步是 **定位出问题的瓶颈** ，对于优化速度来说，从优化函数执行时间这个维度去切入是一个不错的选择。

> 本文侧重分析，不展开如何优化

## 利器

工欲善其事，必先利其器，我们需要一套方便高效的工具记录函数运行时间。说是一套工具，但对于一个简单项目或者日常开发来说，实现一个工具类足矣，由于实现比较简单，直接上代码：

```python
from functools import wraps

import cProfile
from line_profiler import LineProfiler

import time


def func_time(f):
    """
    简单记录执行时间
    :param f:
    :return:
    """

    @wraps(f)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = f(*args, **kwargs)
        end = time.time()
        print f.__name__, 'took', end - start, 'seconds'
        return result

    return wrapper


def func_cprofile(f):
    """
    内建分析器
    """

    @wraps(f)
    def wrapper(*args, **kwargs):
        profile = cProfile.Profile()
        try:
            profile.enable()
            result = f(*args, **kwargs)
            profile.disable()
            return result
        finally:
            profile.print_stats(sort='time')

    return wrapper



try:
    from line_profiler import LineProfiler


    def func_line_time(follow=[]):
        """
        每行代码执行时间详细报告
        :param follow: 内部调用方法
        :return:
        """
        def decorate(func):
            @wraps(func)
            def profiled_func(*args, **kwargs):
                try:
                    profiler = LineProfiler()
                    profiler.add_function(func)
                    for f in follow:
                        profiler.add_function(f)
                    profiler.enable_by_count()
                    return func(*args, **kwargs)
                finally:
                    profiler.print_stats()

            return profiled_func

        return decorate

except ImportError:
    def func_line_time(follow=[]):
        "Helpful if you accidentally leave in production!"
        def decorate(func):
            @wraps(func)
            def nothing(*args, **kwargs):
                return func(*args, **kwargs)

            return nothing

        return decorate
```

原始代码可以参考[gist](https://gist.github.com/adisonhuang/77f94b304c66a2c251dcbc8b5866c5b9)

如下，实现了3个装饰器函数`func_time`,`func_cprofile`,`func_line_time`,分别对应

1. 简单输出函数的执行时间
2. 利用python自带的内置分析包`cProfile` 分析，它主要统计函数调用以及每个函数所占的cpu时间
3. 利用[`line_profiler`](https://github.com/rkern/line_profiler)开源项目,它可以统计每行代码的执行次数和执行时间。

### 使用说明

我们以一个简单的循环例子来说明一下,

```python
def test():
    for x in range(10000000):
        print x
```

* `func_time`

关于`func_time`我觉得没什么好说的，就是简单输出下函数调用时间，这个在我们粗略统计函数执行时间的时候可以使用

如下：

```python
@func_time
def test():
    for x in range(10000000):
        print x
# 输出
test took 6.10190296173 seconds
```

* `func_cprofile`

`cProfile`是python内置包，基于lsprof的用C语言实现的扩展应用，运行开销比较合理，没有外部依赖，适合做快速的概要测试

```python
@func_cprofile
def test():
    for x in range(10000000):
        print x
# 输出
         3 function calls in 6.249 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    6.022    6.022    6.249    6.249 test.py:41(test)
        1    0.227    0.227    0.227    0.227 {range}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```
输出说明：

> 单位为秒

1. 第一行告诉我们一共有3个函数被调用。

   > 正常开发过程，第一行更多是输出类似`194 function calls (189 primiive calls) in 0.249 seconds`，(189 primiive calls)表示189个是原生（primitive）调用，表明这些调用不涉及递归

2. ncalls表示函数的调用次数，如果这一列有两个数值，表示有递归调用，第一个是总调用次数，第二个是原生调用次数。

3. tottime是函数内部消耗的总时间（不包括调用其他函数的时间）。

4. percall是tottime除以ncalls，表示每次调用平均消耗时间。

5. cumtime是之前所有子函数消耗时间的累积和。

6. percall是cumtime除以原生调用的数量，表示该函数调用时，每个原生调用的平均消耗时间。

7. filename:lineno(function)为被分析函数所在文件名、行号、函数名。

* `func_line_time`

[`line_profiler`](https://github.com/rkern/line_profiler)可以生成非常直接和详细的报告，但它系统开销很大，会比实际执行时间慢一个数量级

```python
@func_line_time()
def test():
    for x in range(10000000):
        print x
# 输出
Timer unit: 1e-06 s

Total time: 14.4183 s
File: /xx/test.py
Function: test at line 41

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    41                                           @func_line_time()
    42                                           def test():
    43  10000001    4031936.0      0.4     28.0      for x in range(10000000):
    44  10000000   10386347.0      1.0     72.0          print x
```

输出说明：

> 单位为微秒

1. Total Time：测试代码的总运行时间

2. Line:代码行号
3. Hits：表示每行代码运行的次数
4. Time：每行代码运行的总时间
5. Per Hits：每行代码运行一次的时间
6. % Time：每行代码运行时间的百分比

## 总结

日常开发中，可以使用`func_time`,`func_cprofile`做基本检查，定位大致问题，使用`func_line_time`进行更细致的深入检查。

> 注：`func_line_time` 还可以检查函数内部调用的函数执行时间，通过`follow`参数指定对应的内部调用的函数声明即可，该参数是个数组，也就是说可以检查多个内部调用的函数