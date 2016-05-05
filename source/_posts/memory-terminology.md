title: 内存术语
date: 2016-05-05 15:07:30
tags: [DevTools, memory, terminology]
author: jiliang
---


原文出自：https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/memory-101?hl=en
原文作者：Meggin Kearney

---

本节介绍了用于内存分析的常用术语，和适用于不同语言的很多内存分析工具。

---

这些术语和概念指的是[Chrome DevTools Heap Profiler](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/heap-snapshots)。如果你用过Java,.NET或者其他的内存分析器，那么这将是一次复习提升。

> 目录
>> 对象大小
>> 对象保留树
>> 支配者
>> V8特性


##对象大小
把内存当做一个基本类型(如number，string)和对象(含数组)组成的图。它被形象地表示为有很多相互关联的点，如下图：

![thinkgraph.png](thinkgraph.png)

对象占用内存有两种方式：

- 对象自己直接占用。
- 隐式地通过其他对象的引用占用，因而阻止那些对象被垃圾回收机制(简称**GC**)自动处理。

当用DevTools的Heap Profiler(Profiles下用于审查内存问题的工具)时，你可能会发现你正在看一些不同的信息栏。两个突出的是**Shallow Size**和**Retained Size**，这些是什么？

![shallow-retained.png](shallow-retained.png)

###Shallow size

这是对象自己占用的内存大小。

典型的Javascript对象保留一些内存来存储它的描述和值。一般情况下，只有数组和字符串会有明显的shallow size。然而，在渲染器内存中，字符串和外部数组经常有他们的主仓库，在JavaScript堆上只暴露一个小的封装对象。

渲染内存是检查渲染页面的进程的所有内存:本地内存+当前页JS堆内存+页面开始后的所有专用JS堆内存。然而，小对象也可以通过阻止其他对象被垃圾回收进程自动处理间接占用大量内存。

###Retained size

这是被删除的对象释放的内存大小，这个对象伴随着它所依赖的不能从**GC roots**得到的对象被删除时释放内存。

**GC roots**由(本地或全局的)引用本地代码到V8外的JavaScript对象时创建的handles组成。这些handles会在**GC roots** > **Handle scope**和**GC roots** > **Global handles**下的heap snapshot找到。只在文档中描述handles而不深入浏览器实现细节会很迷惑。GC roots和handles都是比不用担心的。

大部分内部GC roots用户并不关心。从应用的角度讲有以下几种roots：

- Window全局对象(在每个iframe里)。在heap snapshots中有一个distance区域，展示的是在到window的最短保留路径上属性引用的数量。
- 通过遍历文档可以获得由所有原生DOM节点组成的文档DOM树。并不是所有的节点都有JS封装，但是如果有的话，文档活动的时候这些节点也活动。
- 有时对象会被调试环境和DevTools控制台保留(例如控制台赋值之后)。清空控制台关闭调试器的断点后再创建heap snapshots。

内存图从根开始，这个根可能是浏览器的window对象或者Node.js模块的全局对象。你不能控根对象怎样进行垃圾回收。

![dontcontrol.png](dontcontrol.png)

不能达到根节点的节点被作为垃圾回收。

> 注意
>> Shallow和Retained size都用字节表示。

##对象保留树

堆是一个相互连接的对象组成的网络。在数学上，这种结构叫图或者记忆图。图由被边连接的节点构成，边和节点相互对应。

- **节点**(或者说对象)用构造函数的名字指定，构造函数用来创造节点。
- **边**用属性的名字指定。

学习[怎样用Heap Profiler记录profile](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/heap-snapshots)。从Heap Profiler记录下看到的明显的东西包括距离：到GC root的距离。如果几乎所有相同类型的对象距离也一样，其他的少量对象距离比较大，那就需要探查一下了。

![root.png](root.png)

##支配者

支配者对象由一个树形结构组成，因为每个对象都有一个支配者。一个对象的支配者可能没有直接引用它所支配的对象；也就是说，支配者的树不是图的生成树。

在下图中：

- 节点1支配节点2
- 节点2支配节点3，4，6
- 节点3支配节点5
- 节点5支配节点8
- 节点6支配节点7

![dominatorsspanning.png](dominatorsspanning.png)

在下面的例子中，节点#3是#10的支配者，但是#7存在于每个从GC到#10的简单路径中。因此，如果B对象B存在于每个从根到对象A的简单路径中，那么B是A的支配者。

![dominators.gif](dominators.gif)

##V8特性

分析内存时，理解为什么用特定的方法看heap snapshots是很有帮助的。本节描述V8 JavaScript虚拟机(V8 VM或VM)内存相关的问题。

###JavaScript对象表示
有三种基本类型:

- Numbers(如3.14159...)
- Booleans(true,false)
- Strings(如"Werner Heisenberg")

他们不能引用其他的值，他们是叶子节点或者终端节点。

**Numbers**可以被存储为：

- 一个被叫做**小整数**(SMIs)的31位整数值，或者
- 堆对象，称作**heap numbers**。Heap numbers是用来存储不适合SMI格式的值，比如doubles，或者当一个值需要被封装的时候，比如给对象设置属性时。

**Strings**可以被存储在：

- **Vm堆**，或者
- **渲染内存**的外部。一个封装对象被创建和用于访问外部存储，如脚本源和从Web接收的 其他内容存储的位置，而不是复制到VM堆。

新的JavaScript对象的内存由一个专用的JavaScript堆(或者VM堆)分配。这些对象被V8的垃圾回收器管理，因此，只要有一个强引用指向他们，他们就会一直存活。

**内置对象**不在JavaScript堆中。内置对象，不同于堆对象，整个生命周期都不被V8垃圾回收机制管理，只能被JavaScript用它的封装对象访问。

**Cons string**是由多对字符串存储后连接组成的，是一系列相互关联的事物的结果。只在需要时发生*cons string*内容的连接。当一个连接的字符串的子字符串需要被创建时生成一个实例。

例如，如果你连接**a**和**b**，你得到一个字符串(a,b)代表连接结果。如果你再把d连接到这个结果上，你会得到另一个cons string((a,b),d)。

**数组**——是带有数字键值的对象。数组在V8 VM中被广泛用于存储大量数据。键值对集合像数组支持的字典一样使用。

典型的JavaScript对象可以用两种数组类型之一存储：

- 命名属性
- 数字元素

很少数的属性存在JavaScript对象内部。

**Map**——描述对象自身和它的布局的对象。例如，maps用来描述[快速属性访问](https://developers.google.com/v8/design.html#prop_access)的隐式对象层次。

###对象组

内置对象组由占用相同引用的对象组成。想一下，例如，一个DOM子树每一个节点有一个到父节点的连接和多个到下一个子节点及其兄弟节点的连接，这样就形成了一个连通图。注意，内置对象不在JavaScript堆中——所以他们的大小是零。创建的封装的对象则相反。

封装的对象占用与内置对象相对应的引用，用于对它重定向命令。对象组最终占据封装对象。然而这并不能创建一个不可回收的循环，因为GC很快就能释放封装没有被引用的对象组。但是，如果忘记释放一个的封装，那么全部组和相关的封装都将被占用。






