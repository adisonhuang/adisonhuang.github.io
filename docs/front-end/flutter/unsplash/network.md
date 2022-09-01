# Flutter 现学现用 之unsplash客户端实战（二）

## unsplash api注册

在https://unsplash.com/developers注册app，获取Access Key和Secret key



https://unsplash.com/documentation api 文档



## 网络请求

在 Flutter 中，Http 网络编程的实现方式主要分为三种：`dart:io 里的 HttpClient 实现`、`Dart 原生 http 请求库实现`、`第三方库 dio 实现`。

### 依赖管理

Dart 提供了包管理工具 Pub，用来管理代码和资源。从本质上说，包（package）实际上就是一个包含了 pubspec.yaml 文件的目录，其内部可以包含代码、资源、脚本、测试和文档等文件。包中包含了需要被外部依赖的功能抽象，也可以依赖其他包。与 Android 中的 JCenter/Maven、iOS 中的 CocoaPods、前端中的 npm 库类似，Dart 提供了官方的包仓库 Pub。通过 Pub，我们可以很方便地查找到有用的第三方包。

在 Dart 中，库和应用都属于包。pubspec.yaml 是包的配置文件，包含了包的元数据（比如，包的名称和版本）、运行环境（也就是 Dart SDK 与 Fluter SDK 版本）、外部依赖、内部配置（比如，资源管理）。	

```dart
name: funsplash #应用名称
description: A unsplash client project. #应用描述
https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html
#应用版本及构建number
version: 1.0.0+1
#Dart运行环境区间
environment:
  sdk: ">=2.17.6 <3.0.0"
#Flutter依赖库    
dependencies:
  flutter:
    sdk: flutter
  dio: ^4.0.6  
```

~~运行环境和依赖库 cupertino_icons 冒号后面的部分是版本约束信息，由一组空格分隔的版本描述组成，可以支持指定版本、版本号区间，以及任意版本这三种版本约束方式。比如上面的例子中，cupertino_icons 引用了大于 0.1.1 的版本。需要注意的是，由于元数据与名称使用空格分隔，因此版本号中不能出现空格；同时又由于大于符号“>”也是 YAML 语法中的折叠换行符号，因此在指定版本范围的时候，必须使用引号， 比如">=2.1.0 < 3.0.0"。~~



对于包，我们通常是指定版本区间，而很少直接指定特定版本，因为包升级变化很频繁，如果有其他的包直接或间接依赖这个包的其他版本时，就会经常发生冲突。而对于运行环境，如果是团队多人协作的工程，建议将 Dart 与 Flutter 的 SDK 环境写死，统一团队的开发环境，避免因为跨 SDK 版本出现的 API 差异进而导致工程问题。



### dio

HttpClient 和 http 使用方式虽然简单，但其暴露的定制化能力都相对较弱，很多常用的功能都不支持（或者实现异常繁琐），比如取消请求、定制拦截器、Cookie 管理等。因此对于复杂的网络请求行为，我推荐使用目前在 Dart 社区人气较高的第三方 dio 来发起网络请求。

我们首先需要把 dio 加到 pubspec 中的依赖里：

```dart

dependencies:
  dio: ^4.0.6 
```





## JSON解析



由于 Flutter 不支持运行时反射，因此并没有提供像 Gson、Mantle 这样自动解析 JSON 的库来降低解析成本。在 Flutter 中，JSON 解析完全是手动的，开发者要做的事情多了一些，但使用起来倒也相对灵活。









写dart会遇到一个代码风格问题：In new code, use`lowerCamelCase`for constant variables，强制要你使用小骆驼拼写法去定义常量，比如：

```dart
const String ACCOUNTPW = 'account_pw';
```

会出现警告：Prefer using lowerCamelCase for constant names

如果想消除这个警告，有以下几种方式：

1，将ACCOUNTPW 改成accountPw（当然这种方式说了等于白说）

2，在这行代码上面添加一行注释：// ignore: constant_identifier_names，也可以消除警告，但是这种做法只针对注释下面的这一行代码生效，总不能每个地方都添加一遍吧。

3，在项目根目录增加一个配置文件：analysis_options.yaml

```text
include: package:lints/recommended.yaml
linter:
  rules:
    constant_identifier_names: false
```

添加：constant_identifier_names: false配置， 即可对整个项目生效，关闭常量命名校验。