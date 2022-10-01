# Dart 简介

## Dart 是什么？

Dart 是一种面向对象的、类定义的、单继承的语言，它使用 C 语言风格的语法来清晰地表达代码，同时也支持泛型、函数式编程和面向对象编程。Dart 代码可以编译成本地代码（使用 [dart2native](https://dart.dev/tools/dart2native) 命令），也可以编译成 JavaScript 代码（使用 [dart2js](https://dart.dev/tools/dart2js) 命令）。

Dart 代码可以在服务器端运行，也可以在浏览器端运行。Dart 代码可以使用 [Flutter](https://flutter.dev/) 框架编写移动应用程序。


## Dart 核心特性

### JIT 与 AOT

借助于先进的工具链和编译器，Dart 是少数同时支持 `JIT`（Just In Tim，即时编译）和 `AOT`（Ahead of Time，运行前编译）的语言之一。

语言在运行之前通常都需要编译，JIT 和 AOT 则是最常见的两种编译模式。

* JIT 在运行时即时编译，在开发周期中使用，可以动态下发和执行代码，开发测试效率高，但运行速度和执行性能则会因为运行时即时编译受到影响。

* AOT 即提前编译，可以生成被直接执行的二进制代码，运行速度快、执行性能表现好，但每次执行前都需要提前编译，开发测试效率低。

在开发期使用 JIT 编译，可以缩短产品的开发周期。Flutter 最受欢迎的功能之一热重载，正是基于此特性。而在发布期使用 AOT，就不需要像 React Native 那样在跨平台 JavaScript 代码和原生 Android、iOS 代码之间建立低效的方法调用映射关系。所以说，Dart 具有运行速度快、执行性能好的特点。

AOT 的典型代表是 C/C++，它们必须在执行前编译成机器码；而 JIT 的代表，则包括了如 JavaScript、Python 等几乎所有的脚本语言。

### 内存分配与垃圾回收

Dart VM 的内存分配策略比较简单，创建对象时只需要在堆上移动指针，内存增长始终是线性的，省去了查找可用内存的过程。

Dart 的垃圾回收，则是采用了多生代算法。新生代在回收内存时采用“半空间”机制，触发垃圾回收时，Dart 会将当前半空间中的“活跃”对象拷贝到备用空间，然后整体释放当前空间的所有内存。回收过程中，Dart 只需要操作少量的“活跃”对象，没有引用的大量“死亡”对象则被忽略，这样的回收机制很适合 Flutter 框架中大量 Widget 销毁重建的场景。

### 单线程模型

Dart 是单线程模型， 天然不存在资源竞争和状态同步的问题，这样可以让 Dart 语言的开发者更加专注于业务逻辑的实现，而不需要考虑多线程的问题。 

在 Dart 中，并发是通过 Isolate 实现的。Isolate 是类似于线程但不共享内存，独立运行的 worker。这样的机制，就可以让 Dart 实现无锁的快速分配。

## 语言特性
>  我们可以在[dartpad](https://dartpad.cn/) 或者 [replit](https://replit.com/languages/dart) 上体验 Dart 语言的特性。

Dart 在静态语法方面和 Java 非常相似，如类型定义、函数声明、泛型等，而在动态特性方面又和 JavaScript 很像，如函数式特性、异步支持等。除了融合 Java 和 JavaScript 语言之所长之外，Dart 也具有一些其他很有表现力的语法，如可选命名参数、..（级联运算符）和?.（条件成员访问运算符）以及??（判空赋值运算符）。

Dart 是一门强类型语言，但是它的类型系统比较灵活，支持类型推断，可以让开发者不用显式地声明变量的类型，而是由 Dart VM 自动推断。

在 Dart 中，我们可以用 var 或者具体的类型来声明一个变量。当使用 var 定义变量时，表示类型是交由编译器推断决定的，当然你也可以用静态类型去定义变量，更清楚地跟编译器表达你的意图，这样编辑器和编译器就能使用这些静态类型，向你提供代码补全或编译警告的提示了。

Dart 是类型安全的语言，并且所有类型都是对象类型，都继承自顶层类型 Object，因此一切变量的值都是类的实例（即对象），甚至数字、布尔值、函数和 null 也都是继承自 Object 的对象。

### 基本语法

#### **`var`**

var 是 Dart 中的关键字，用于声明变量，它的作用是让编译器自动推断变量的类型，因此 var 定义的变量是没有类型的，它的类型是由编译器推断出来的。

Dart 中 var 变量一旦赋值，类型便会确定，则不能再改变其类型，如：

```dart
 var t = "hi world";
// 下面代码在dart中会报错，因为变量t的类型已经确定为String，
// 类型一旦确定后则不能再更改其类型。
t = 1000;
```

#### **`final和const`**

在 Dart 中，final 和 const 都可以用来定义常量，但是它们之间有一些细微的区别。 

* const，表示变量在编译期间即能确定的值；

* final 则不太一样，用它定义的变量可以在运行时确定值，而一旦确定后就不可再变。

``` dart
//可以省略String这个类型声明
final str = "hi world";
//final String str = "hi world"; 
const str1 = "hi world";
//const String str1 = "hi world";
```

> const 适用于定义编译常量（字面量固定值）的场景，而 final 适用于定义运行时常量的场景。

#### **`空安全（null-safety）`**

Dart 2.12 版本引入了空安全（null-safety）特性，它是一种静态类型检查，可以让你在编译时就发现空指针异常，而不是在运行时才发现。

```dart
int i = 8; //默认为不可空，必须在定义时初始化。
// 如果我们预期变量不能为空，但在定义时不能确定其初始值，则可以加上late关键字，
// 表示会稍后初始化，但是在正式使用它之前必须得保证初始化过了，否则会报错
late int k;
k=9;


String? str ;  // 定义为可空类型，对于可空变量，我们在使用前必须判空。
print(str!.length); //如果str为null，这里会报错
if(str !=null){
    print(str!.length); //因为已经判过空，所以能走到这 i 必不为null
}

print(str?.hashCode); // 如果我们预期变量可能为空，但是在使用前我们又不想判空，那么可以使用?操作符，

str = "test";

print(str!.length); // 正确，str不为空，可以使用！
```

#### **`String`**

`Dart 的 String 由 UTF-16 的字符串组成`。和 JavaScript 一样，构造字符串字面量时既能使用单引号也能使用双引号，还能在字符串中嵌入变量或表达式：你可以使用 ${express} 把一个表达式的值放进字符串。而如果是一个标识符，你可以省略{}。

```dart
var s = 'cat';
var s1 = 'this is a uppercased string: ${s.toUpperCase()}';

// 字符串拼装

var s2 = 'Hello' + ' ' + 'World!' ;

// 对于多行字符串的构建，你可以通过三个单引号或三个双引号的方式声明，这与 Python 是一致的：

var s3 = """This is a
multi-line string.""";

```

### 运算符

运算符Dart 和绝大部分编程语言的运算符一样。

不过，Dart 多了一些额外的运算符，比如 `?.`、`??=`、`??`、`..`、`...`、`...?`


* **?. 运算符**：如果?.左侧的操作数为 null，则返回 null，否则就对?.右侧的操作数求值。
    ```dart
    var a;
    print(a?.length); // null
    ```

* **??= 运算符**：如果 p 为 null，那么 p = Point(1, 2) 就会报错。但是，p ??= Point(1, 2) 就不会报错，因为 ??= 运算符的作用是，如果 p 为 null，那么就把 Point(1, 2) 赋值给 p。

* **?? 运算符**：如果 p 为 null，那么 p ?? Point(1, 2) 就会返回 Point(1, 2)。如果 p 不为 null，那么 p ?? Point(1, 2) 就会返回 p。

* **.. 运算符**：.. 运算符可以用于级联调用（`cascade notation`），它允许你在同一个对象上执行多个操作。

    ```dart
    var sb = StringBuffer()
      ..write('foo')
      ..write('bar');
    print(sb); // foobar
    ```

* **.../ ...?  运算符**  ：... 运算符可以用于展开一个集合的元素，比如：

    ```dart
    
    var list = [1, 2, 3];
    var list2 = [0, ...list];
    print(list2); // [0, 1, 2, 3]

    // ...? 运算符可以用于展开一个集合的元素，但是如果集合为 null，那么就不展开
    var list3;
    var list4 = [0, ...?list3];
    print(list4); // [0]

    ```


### 函数

Dart 语言中的函数是一等公民，可以作为参数传递，也可以作为返回值返回。

```dart
// 定义一个函数
int add(int a, int b) {
  return a + b;
}

// 箭头函数
int add1(int a, int b) => a + b;

// 可选参数
int add2(int a, int b, [int c]) {
  return a + b + (c ?? 0);
}

// 可选命名参数

int add3(int a, int b, {int c}) {
  return a + b + (c ?? 0);
}


// 调用函数
add(1, 2);

// 可选参数

add2(1, 2, 3);

// 命名参数

add3(1, 2, c: 3);



//函数作为变量
var say = (str){
  print(str);
};
say("hi world");

// 函数作为参数传递
void execute(var callback) {
    callback();
}
execute(() => print("xxx"))

    
```

**Dart 认为重载会导致混乱，因此从设计之初就不支持重载，而是提供了可选命名参数和可选参数**。

在声明函数时：

* 给参数增加{}，以 paramName: value 的方式指定调用参数，也就是可选命名参数；
* 给参数增加[]，则意味着这些参数是可以忽略的，也就是可选参数。

> 可选参数和命名参数都可以设置默认值，但是可选参数的默认值只能通过位置来确定，而命名参数的默认值只能通过名称来确定。

**可选命名参数在Flutter中使用非常多。注意，不能同时使用可选的位置参数和可选的命名参数**

### 类

Dart 中并没有 public、protected、private 这些关键字，我们只要在声明变量与方法时，在前面加上“_”即可作为 private 方法使用。如果不加“_”，则默认为 public。不过，**“_”的限制范围并不是类访问级别的，而是库访问级别**。


#### **类的构造**

```dart
class Person {
  String name;
  int age;

  // 默认构造函数
  Person(this.name, this.age);

  // 命名构造函数
  Person.withName(String name) : this(name, 10);

  // 命名构造函数
  Person.withNameAndAge(this.name, this.age);

  void printInfo() {
    print('name: $name, age: $age');
  }
}

void main() {
  var p = Person.withName("test");
  var p1 = Person.withNameAndAge("dd",100);
  p.printInfo(); // 输出(name: test, age: 10)
  
  p1.printInfo(); // 输出(name: dd, age: 100)
}


```

####  **类的混入(Mixin)**

Dart 是不支持多继承的，但是它支持 mixin，简单来讲 mixin 可以 “组合” 多个类的功能，而不是继承它们。

这样一来，不仅可以解决 Dart 缺少对多重继承的支持问题，还能够避免由于多重继承可能导致的歧义（菱形问题）。

!!!note ""
    继承歧义，也叫菱形问题，是支持多继承的编程语言中一个相当棘手的问题。当 B 类和 C 类继承自 A 类，而 D 类继承自 B 类和 C 类时会产生歧义。如果 A 中有一个方法在 B 和 C 中已经覆写，而 D 没有覆写它，那么 D 继承的方法的版本是 B 类，还是 C 类的呢？

要使用混入，只需要 `with` 关键字即可

```dart

abstract class Person {
  String name;
  int age;

  Person(this.name, this.age);

  void printInfo();
}

abstract class Student {
  void study();
}

class PersonStudent extends Person with Student {
  PersonStudent(String name, int age) : super(name, age);

  @override
  void printInfo() {
    print('name: $name, age: $age, type: PersonStudent');
  }

  @override
  void study() {
    print('name: $name, age: $age, type: PersonStudent, study');
  }
}

```




### 异步

Dart 语言中的异步，是通过 `Future` 和 `Stream` 实现的。`Isolate` 也是一种异步的实现方式。 

`async`和`await`关键词支持了异步编程，允许您写出和同步代码很像的异步代码。

#### **Future**

`Future`与JavaScript中的`Promise`非常相似 ， 表示一个异步操作的最终完成（或失败）及其结果值的表示。一个Future只会对应一个结果，要么成功，要么失败。

**Future 的所有API的返回值仍然是一个Future对象，所以可以很方便的进行链式调用。**

```dart
Future<String> lookUpVersion() => Future.delayed(Duration(seconds: 2), () => '1.0.0');

void main() {
  print('start');
  lookUpVersion().then((value) => print(value));
  print('end');
}

// 输出
start
end
1.0.0
```


#### **`async`和`await`**

`async`和`await`关键词支持了异步编程，允许您写出和同步代码很像的异步代码。

`async`关键字用于声明一个异步函数，`await`关键字用于等待一个异步函数执行完成。


```dart
import 'dart:async';

void main() async {
  print('start');
  await printAsync();
  print('end');
}

Future printAsync() async {
  await Future.delayed(Duration(seconds: 2));
  print('printAsync');
}
```



#### **Stream**

`Stream`是一个异步事件序列，简单理解，其实就是一个异步数据队列， 它可以是单订阅或者广播。

```dart
import 'dart:async';

void main() {
 void main(){
  test();
}

test() async{
  // 使用 periodic 创建流，第一个参数为间隔时间，第二个参数为回调函数
  Stream<int> stream = Stream<int>.periodic(Duration(seconds: 1), callback);
  // await for循环从流中读取
//   await for(var i in stream){
//     print(i);
//   }

    // 使用listen监听流
  stream.listen((value) => print(value));
}

int callback(int value){
  return value;
}

// 输出
1
2
3
....
```

单订阅的Stream只能被监听一次，而广播的Stream可以被监听多次。

```dart
import 'dart:async';

void main() {
  Stream<int> stream = Stream<int>.periodic(Duration(seconds: 1), callback);
  stream.listen((value) => print(value));
 // stream.listen((value) => print(value)); // 报错，因为流只能被监听一次 "Unhandled exception:Bad state: Stream has already been listened to."

  Stream<int> stream = Stream<int>.periodic(Duration(seconds: 1), callback).asBroadcastStream(); // 转换成广播的Stream
  stream.listen((value) => print(value));
  stream.listen((value) => print(value)); // 正常输出 ，广播的Stream可以被监听多次
}
```
#### **Isolate**

`Isolate`是Dart中的一个轻量级的并发执行单元，但是它不是一个线程。它是一个线程的模拟，它可以在单个线程中运行多个`Isolate`，并且它们之间不会共享内存。


```dart

import 'dart:isolate';

void main() {
  Isolate.spawn(foo, 'hello');
  Isolate.spawn(foo, 'world');
}

void foo(String message) {
  print(message);
}

// 输出

hello
world
```
















