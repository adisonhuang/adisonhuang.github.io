# 设计模式概述

## 1. 设计模式是什么？

**设计模式** 是软件设计中常见问题的典型解决方案。 它们就像能根据需求进行调整的预制蓝图， 可用于解决代码中反复出现的设计问题。

设计模式与方法或库的使用方式不同， 你很难直接在自己的程序中套用某个设计模式。 模式并不是一段特定的代码， 而是解决特定问题的一般性概念。 你可以根据模式来实现符合自己程序实际所需的解决方案。

人们常常会混淆模式和算法， 因为两者在概念上都是已知特定问题的典型解决方案。 但算法总是明确定义达成特定目标所需的一系列步骤， 而模式则是对解决方案的更高层次描述。 同一模式在两个不同程序中的实现代码可能会不一样。

算法更像是菜谱： 提供达成目标的明确步骤。 而模式更像是蓝图： 你可以看到最终的结果和模式的功能， 但需要自己确定实现步骤。

## 2. 为什么学习设计模式？

或许你已从事程序开发工作多年， 却完全不知道单例模式是什么。 很多人都是这样。 即便如此， 你可能也在不自知的情况下已经使用过一些设计模式了。 所以为什么不花些时间来更进一步学习它们呢？

- 设计模式是针对软件设计中常见问题的工具箱， 其中的工具就是各种 **经过实践验证的解决方案**。 即使你从未遇到过这些问题， 了解模式仍然非常有用， 因为它能指导你如何使用面向对象的设计原则来解决各种问题。
- 设计模式定义了一种让你和团队成员能够更高效沟通的通用语言。 你只需说 “哦， 这里用单例就可以了”， 所有人都会理解这条建议背后的想法。 只要知晓模式及其名称， 你就无需解释什么是单例。


## 3. 封装和设计模式区别 

封装可以理解为由程序组合而成的可复用的具有特定功能的组件。而设计模式则表示组件内部或者组件之间是如何组装的。举个例子，汽车是由发动机、底盘、车身、电气设备组成，它们都有各自的作用，但我们在讨论汽车是什么的的时候，并不关心用的是哪个品牌的发动机，什么颜色的车身，更重要的是它们的“关系”，它们之间相互关联互相作用，构成了真正的汽车，这是个工程化的过程。

!!! note ""

    **设计模式就是表示这种组件相互关系的工程化抽象。**

## 3. 关于模式的争议

设计模式自其诞生之初似乎就饱受争议， 所以让我们来看看针对模式的最常见批评吧。

* **一种针对不完善编程语言的蹩脚解决方案**

通常当所选编程语言或技术缺少必要的抽象功能时， 人们才需要设计模式。 在这种情况下， 模式是一种可为语言提供更优功能的蹩脚解决方案。

例如， 策略模式在绝大部分现代编程语言中可以简单地使用匿名 （lambda） 函数来实现。

* **低效的解决方案**

模式试图将已经广泛使用的方式系统化。 许多人会将这样的统一化认为是某种教条， 他们会 “全心全意” 地实施这样的模式， 而不会根据项目的实际情况对其进行调整。

* **不当使用**

> 如果你只有一把铁锤， 那么任何东西看上去都像是钉子。

这个问题常常会给初学模式的人们带来困扰： 在学习了某个模式后， 他们会在所有地方使用该模式， 即便是在较为简单的代码也能胜任的地方也是如此。

## 4. 设计模式分类

不同设计模式的复杂程度、 细节层次以及在整个系统中的应用范围等方面各不相同。 我喜欢将其类比于道路的建造： 如果你希望让十字路口更加安全， 那么可以安装一些交通信号灯， 或者修建包含行人地下通道在内的多层互通式立交桥。

最基础的、 底层的模式通常被称为 *惯用技巧*。 这类模式一般只能在一种编程语言中使用。

最通用的、 高层的模式是 *构架模式*。 开发者可以在任何编程语言中使用这类模式。 与其他模式不同， 它们可用于整个应用程序的架构设计。

此外， 所有模式可以根据其*意图*或目的来分类。 主要有三种模式类别：

- **创建型模式**  提供创建对象的机制， 增加已有代码的灵活性和可复用性。
- **结构型模式**  介绍如何将对象和类组装成较大的结构， 并同时保持结构的灵活和高效。
- **行为模式**  负责对象间的高效沟通和职责委派。
