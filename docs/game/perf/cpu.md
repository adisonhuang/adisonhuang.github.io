# Unity3D性能优化——CPU篇

## **CPU性能优化**

CPU 优化主要以性能分析为引，根据分析所得的数据，找到性能问题，以便快速并定向的优化项目。在我们之前的文章中有提到，CPU usage profiler会统计渲染、 物理、 脚本、 GC、UI、垂直同步以及全局光照等模块的 CPU 使用情况。

首先我们再了解一个工具，在我们的项目中，可以在Game视图点击stats开启Statistics窗口（渲染统计窗口），该窗口显示游戏运行时，渲染、声音、网络状况等多种统计信息，帮助我们分析游戏性能。

![img](./assets/v2-10138cdb510ba1ca1c36e9429b21b325_1440w.jpeg)



- **首先我们来了解几个概念性的问题**



- **垂直同步**
  关于垂直同步的问题，我们在之前的工具篇中就有提到，想要了解的读者可以看看之前的文章，如果要深入了解，可能需要参考一些教程，在本文中就不做过多的阐述。
	[Unity3D性能优化——工具篇](https://blog.adison.top/game/perf/tools/)

- **渲染**
  在unity中GPU和CPU渲染也是一个很大的话题，在之后的文章中，会提到渲染问题，在这里我们只需要大致的了解什么是Batches和Draw Call。
  在我们进行游戏优化时，常会听到要减少Draw Call。Draw Call实际上就是一个命令，它的发起方是CPU，接收方是GPU，这个命令仅仅会指向一个需要被渲染的图元列表，而不会再包含任何材质信息。 当给定一个Draw Call时，GPU就会根据渲染状态和所有输入的顶点数据来进行计算，最终输出成屏幕上显示的像素。
  引擎每对一个物体进行一次DrawCall，就会产生一个Batch，这个Batch里包含着该物体所有的网格和顶点数据，当渲染另一个相同的物体时，引擎会直接调用Batch里的信息，将相关顶点数据直接送到GPU，从而让渲染过程更加高效，即Batching技术是将所有材质相近的物体进行合并渲染。
  Batches其实就是Unity内置的Draw Call Batching



- **GC(垃圾回收)**

在提起这个问题时，我们首先要了解一下GC。
而想要了解什么是GC我们就要考虑到unity的内存管理机制。

> Unity主要采用自动内存管理的机制，开发者不需要详细地告诉unity如何进行内存管理，unity内部自身会进行内存管理。
>
> Unity内部有两个内存管理池：堆内存和栈内存。
>
> Unity中的变量只会在栈或堆内存上进行内存分配。只要变量处于激活状态，则其占用的内存会被标记为使用状态，则该部分的内存处于被分配的状态。一旦变量不再激活，则其所占用的内存不再需要，该部分内存可以被回收到内存池中被再次使用，这样的操作就是内存回收。处于栈上的内存回收及其快速，处于堆上的内存并不是及时回收的，此时其对应的内存依然会被标记为使用状态。垃圾回收主要是指堆上的内存分配和回收，unity中会定时对堆内存进行GC操作。
>
> 每次运行GC的时候，会检查堆内存上的每个存储变量，然后对每个变量会检测其引用是否处于激活状态，如果变量的引用不再处于激活状态，则会被标记为可回收，被标记的变量会被移除，其所占有的内存会被回收到堆内存上。GC操作是一个极其耗费的操作，堆内存上的变量或者引用越多则其运行的操作会更多，耗费的时间越长。
> 在了解GC在unity内存管理中的作用后，我们需要考虑其带来的问题。最明显的问题是GC操作会需要大量的时间来运行，如果堆内存上有大量的变量或者引用需要检查，则检查的操作会十分缓慢，这就会使得游戏运行缓慢。其次GC可能会在关键时候运行，例如在CPU处于游戏的性能运行关键时刻，此时任何一个额外的操作都可能会带来极大的影响，使得游戏帧率下降。
>
> 另外一个GC带来的问题是堆内存的碎片化。当一个内存单元从堆内存上分配出来，其大小取决于其存储的变量的大小。当该内存被回收到堆内存上的时候，有可能使得堆内存被分割成碎片化的单元。也就是说堆内存总体可以使用的内存单元较大，但是单独的内存单元较小，在下次内存分配的时候不能找到合适大小的存储单元，这也会触发GC操作或者堆内存扩展操作。
>
> 堆内存碎片会造成两个结果，一个是游戏占用的内存会越来越大，一个是GC会更加频繁地被触发。

在了解了以上的几个概念后，我们就可以从实际运用中来探索，如何对unity进行相关操作时引起的性能问题进行优化。



## **1.缓存**

**我们可以把一些必要的对象缓存起来，** 在Unity中，类似于GameObject.Find ， GetComponent，transform这类的函数，会产生较大的消耗，比如下列代码：

```text
void Update(){
    this.transform.Translate(0, 1, 0);
    this.GetComponent<Rigidbody>().AddForce(Vector3.forward);
}
```

在update中，每帧都去访问这些函数是非常耗时的，我们可以在 Awake 或者 Start 函数中，获取一次组件引用，把引用缓存在当前 class 中，供 Update 等函数使用 这样可以减少每帧获取组件带来的开销。

```text
Rigidbody mRigi;
private Transform mTransform;

void Awake(){
    myRigi = this.GetComponent<Rigidbody>();
    myTransform = this.transform;
}

void Update(){
    myRigidbody.AddForce(Vector3.forward);
    myTransform.Translate(0, 1f, 0f);
}
```

在 Awake 函数中获取刚体组件和 transform 组件的引用，它们缓存在当前的 class 的字段中，然后在 update 函数中通过私有字段，来使用刚体和 transform 组件。

- **注意，GetComponent如果返回空值会调用GC如下所示**：

![img](./assets/v2-49609c39845279407f9c71ff4697cb49_1440w.jpeg)


在这里，cube中并没有添加刚体，我们在update中调用如下代码：

```
this.GetComponent<Rigidbody>().AddForce(Vector3.forward);
```

可以看到在Profiler中，BehaviourUpdate有近16k的GC

![img](./assets/v2-296e69577d26b83729f01e4e4d8f30b1_1440w.jpeg)


所以，我们要避免出现空的组件获取。

- 同时我们在项目中可能会用到，`unityEngine.Object == null`这样的比较方法，它和GetComponent()的情况类似，也会出现cpu消耗以及GC，这里都涉及到引擎方面的调用机制。对于大多数的测试功能，我们都应该少做null比较，而是使用**断言（Assert）**来代替。



## **2. 对象池**

- 对象池是我们在实际的游戏开发中，运用比较广泛的重要技术。

在项目中频繁地使用 Instantiate 和 Destroy 函数，会为脚本执行和垃圾回收带来很大的性能开销。如下图所示，我们使用Instantiate函数大量的创建物体，并使用Destroy销毁，这些代码占用了大量的cpu时间。

![img](./assets/v2-6567fe29da6848e800673b04304cd567_1440w.jpeg)


我们可以使用对象池优化游戏对象的创建和销毁

对象池的含义很简单，我们将对象储存在一个“池”中，当需要它时可以重复使用，而不是创建一个新的对象，尽可能的复用内存中已经驻留的资源来减少频繁的IO耗时操作。有经验的开发者在程序设计时就会做一个规范，其中包含了角色池，怪物池，特效池，经验池等。

- **注意：使用对象池时，应当可以支持把物体移出屏幕，连续使用的物体可以只是移出屏幕，只有长时间不使用的物体才隐藏；因为频繁的Activate使用，也会引起不必要的性能消耗。**



## **3.冗余代码**

**删除不必要的函数、无用的代码段及测试代码**

在我们的项目开发中，可能会增加很多测试代码，或者有一些空的回调函数或许还会有一些不需要的继承。
比如说一些控制台输出的的测试或者空的start、update，这些都会消耗CPU性能。
所以在项目开始时，应该在编辑器中设置不要自动继承MonoBehaviour，在需要使用时自行添加。同时测试代码要做好标记，不再需要其时一点要删除。



## **4.CPU峰值**

**避免实例化对象时造成cpu峰值**

如果我们在某一个特定时间，集中创建很多对象，会造成这一时间段的cpu峰值，我们的游戏就会在这一时间段就会出现卡顿，这里可以使用协程做一些间隔。
打个比方，我们开发一个rpg游戏，当我们切换到一个新的地图，地图中有大量的怪物，如果我们在进入地图时集中创建所有的怪物，那么在初始进入地图的瞬间，必然会非常的卡顿。我们可以做的就是在加载进度或进入地图后，使用协程，如每一帧创建一个怪物，这样可以在我们完成其他动作（如行走）的每帧同时(之后)，逐渐创建游戏对象，这样我们基本不会感觉到游戏的卡顿，并且同样实现了我们创建游戏对象的需求。

> Unity中的协程方法通过yield这个特殊的属性，可以在任何位置、任意时刻暂停。也可以在指定的时间或事件后继续执行，而不影响上一次执行的就结果，提供了极大地便利性和实用性。
>
> 协程在每次执行时都会新建一个（伪）新线程来执行，而不会影响主线程的执行情况。

```text
private int instanceCount;
void Start(){
    insCount = 10;
    //启动协程
    StartCoroutine(SpawnInstance());
}

IEnumerator SpawnInstance(){
    while(insCount > 0){
        //这里创建游戏物体
        insCount --;
        //协程返回，在固定时间后继续执行后面的代码
        yield return new WaitForSeconds(1f);
    }
}
```

- **注意：在使用协程时，yield本不会产生堆内存分配，但是如果yield带有参数返回，则会造成不必要的内存垃圾** 例如：`yield return 0;`由于需要返回0，引发了装箱操作，所以会产生内存垃圾。这种情况下，为了避免内存垃圾，我们可以这样返回：`yield return null;`如果我们每次返回时，都会new一个相同的变量， 例如我们上边的代码`yield return new WaitForSeconds(1f);`我们可以采用缓存来避免这样的内存垃圾产生：

```text
WaitForSeconds spawnWait = new WaiForSeconds(1f);

IEnumerator SpawnInstance(){
    while(insCount > 0){
        //这里创建游戏物体
        insCount --;
        //协程返回，在spawnWait时间后继续执行后面的代码
        yield return spawnWait;
    }
}
```



## **5.GC(垃圾回收)**

**尽量少的使用会调用GC的代码**

首先我们来看一下，在我们写代码时，哪些操作会调用GC：
1） 在堆内存进行内存分配操作，而内存不够时便会触发垃圾回收来利用闲置内存；
2） GC会自动的触发，不同平台频率不同；
3） GC可以手动执行。

其中最主要的就是第一点，如果GC造成性能问题，我们就需要知道哪部分代码会造成GC，内存垃圾在变量不再激活时产生，所以首先我们需要知道堆内存上分配的是什么变量。

即使是初学者也应该了解在C#中，值类型变量都在栈上进行内存分配（如int = 7 其对应的变量在函数调用完后会立即回收），引用类型都在堆内存上分配（如List myList = new List() 其对应的变量在GC的时候才回收）。

在了解了这些后，我们可以配合之前所学的profier工具来定位造成大量内存分配的函数，一旦定位该函数，我们就可以分析解决其造成问题的原因从而减少内存垃圾的产生。

我们在之前已经提到了部分会调用GC的操作，并且了解了GC的基本原理，这里我们再说一些会调用GC的操作。

- 比如String的相加操作， 会频繁申请内存并释放，导致GC频繁（在c#中，字符串是引用类型，也就意味着，每次我们操纵一个字符串（比如这里所说的相加操作），Unity将创建一个包含更新值的新字符串，我们创建和销毁字符串都会产生垃圾。），我们可以使用System.Text.StringBuilder。



- 在unity中，很多函数调用也会造成内存垃圾，比如之前所说到的GetComponent，还有GameObject.name或GameObject.tag，Input.touches，Physics.SphereCastAll()等。
  我们可以如上文第一点描述的方法，先对其进行缓存。而且unity也提供了相关的函数来替换他们，比如GameObject.tag我们可以使GameObject.CompareTag()来替代，Input.touches可以使用Input.GetTouch()，或者用Physics.SphereCastNonAlloc()来代替Physics.SphereCastAll()。

------

以上是我们在实际的unity项目中进行cpu优化的主要注意方向，当然，在实际项目的优化中，还有更多的方法和细节需要我们注意，例如

**1）减少对粒子系统Play()的调用；**

**2）处理Rigidbody时，使用FixedUpdate，设置Fixed timestep，减少物理计算次数；**

**3）如果可以，尽量不用MeshCollider，如果不能避免的话，尽量用减少Mesh的面片数，或用较少面片的物体来代替；**

**4）使用内建数组如使用Vector3.zero而不是new Vector(0,0,0);**

**5）每次访问Transform.position/rotation都有相应的消耗。能cache就cache其返回结果。**

**6）如果可能，尽量用Queue/Stack来代替List**

**7） 同一脚本中频繁使用的变量建议声明其为全局变量，脚本之间频繁调用的变量或方法建议声明为全局静态变量或方法；**

在本文中，只能用一些简单的例子来讲述一些常见的性能问题及优化方法，而更多的优化问题，需要读者在具体的项目中发现，并根据实际的应用场景来选择合适的优化方法

## 参考

摘自https://zhuanlan.zhihu.com/p/39998137