title: 内存诊断
date: 2016-05-04 15:26:28
tags: [memory, DevTools]
author: jiliang

---

原文出自: https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/?memory-diagnosis?hl=en
原文作者: Meggin Kearney

---

内存泄漏是可用计算机内存的逐渐丢失，发生在程序临时使用内存后不能释放内存时。当你怀疑自己造成内存泄漏时，按照以下目录表中列出的调查步骤提纲测试。

> 目录 
>> 
   [检查页面是否用了太多内存]()
   [发现没有被垃圾回收清理的对象]()
   [减少内存泄漏的因素]()
   [限定垃圾回收频率]()
   [内存分析资源]()
  
---

> TL;DR(Too Long; Don,t Read)
>> 
   通过Chrome Task Manager监控内存条目快速查看页面是否消耗太多内存。
   使用[Chrome Develop Timeline](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/timeline-tool)中的momory view确认内存使用是否在增长。
   使用Chrome DevTools heap profiler确认已分离的节点是否仍然占用内存。
   注意频繁的垃圾收集和垃圾收集暂停，二者都会影响性能。

---

### 检查页面是否用了太多内存

检查一个页面是否使用太多内存，首先要做的就是确认你怀疑的一系列行为是否正在泄漏内存。这可能是网址导航的任何事件，鼠标悬浮，点击，或者其他的某些随着时间推移影响性能的页面交互。
一旦你怀疑内存性能问题，当你第一次注意到你的页面长时间使用后变慢时，使用[Chromr Develop Timeline](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/timeline-tool)诊断过度内存使用。

> 注意
>> 
   第一次听说内存管理？开始查看[内存术语](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/memory-101)基础。

#### 使用Chrome任务管理器监控内存

用Chrome任务管理器监控一个页面的活动内存使用。
从Chrome菜单进入任务管理器>Tools>Task Manager(快捷键Shift+Esc)。
打开后，右键点击条目的头部区域，能看到JavaScript memory选项。运行的事件可能用太多内存，监控活动内存使用的变化。

![task-manager.npg](task-manager.png)

#### 使用memory视图确认内存使用是否在增加

要诊断是否是内存问题，要转到Timeline面板Memory视图。点击Record按钮，与你的应用程序进行交互，重复任何你认为可能导致泄漏的步骤。停止记录。
你看到的图表显示你的应用程序分配到的内存。如果它恰巧在这段时间消耗大量增长(而且没有下降)，这就是内存泄漏的迹象。
![normal-sawtooth.png](normal-sawtooth.png)
   
正常的应用程序形状应该更像一个锯齿状曲线，因为垃圾回收器的介入使内存分配后释放。不必担心这个——运行JavaScript总会有额外消耗，即便是一个空的`requestAnimationFrame`也会产生这种锯齿状，这无法避免。
只要锯齿不是尖锐的，那就说明很多内存被分配了，另一方面也产生了很多垃圾。你必须注意曲线陡峭度增加的比例。
![steep-sawtooth.png](steep-sawtooth.png)

Memory视图中的DOM node counter，Document counter 和Event listener count在诊断过程中会很有用。DOM节点使用本地，不直接影响JavaScript内存图。
一旦你怀疑有内存泄漏，用heap profiler和object allocation tracer找泄漏的原因。

**例子：**尝试这个[内存增长](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example1.html)的例子，你可以练习高效使用Timeline memory模式。


### 发现没有被垃圾回收机制清理的对象

用Profiles面板的Chrome DevTools Heap profiler发现没有被垃圾回收机制清理的对象。

Heap profiler通过页面的JavaScript对象和相关DOM节点显示内存分配。这有助于发现另外的不可见的泄漏，这些泄漏的发生是由于被遗忘的分离的DOM子树的游离。

用heap profiler能获取JS heap snapshots，分析内存图表，比较snapshots,探测DOM泄漏(详见[How to Record Heap Snapshots](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/heap-snapshots))。
在constructor和retained视窗有很多数据。以最短距离保留的对象通常是导致内存泄漏的第一个候选者。内存泄漏侦查从你的DOM树中的第一个保留对象开始，因为保留的对象按到窗口的距离排序。

![first-retained.jpg](first-retained.jpg)

当心你的heap snapshots中的黄色和红色的对象。红色节点(深色背景)没有从JavaScript到他们的直接引用，但是他们却是活动的，因为他们是分离的DOM树的一部分。也可能是一个引用自JavaScript树中的节点(可能是作为一个闭包或者变量)，但恰巧阻止了整个DOM树被垃圾回收。

![red-yellow-objects.jpg](red-yellow-objects.jpg)

然而黄色节点(黄色背景)是真有JavaScript引用。在同一个分离的DOM树中寻找黄色节点，定位JavaScript引用。应该有一个从DOM window到元素的属性链(比如window.foo.bar[2].baz)。

观察这个动画，理解分离的节点从哪里融入整个图片：

![detached-nodes.gif](detached-nodes.gif)

**例子：**尝试这个[分离节点](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example4.html)的例子，你能在Timeline中看到节点演变过程然后获得heap snapshots找到分离节点。

###减少内存泄漏的因素
用object allocation tracker通过看JS对象实时分配减少内存泄漏的原因。**object tracker**结合了heap profiler的详细快照信息和Timeline面板增加的更新和跟踪。

跟踪对象的堆分配包括开始一个记录，展示一系列行为，停止记录分析。object tracker在整个记录期间定时获得heap snapshots(如每50ms一次)，在记录末尾还有一次最后快照。堆分配配置资料展示对象在哪里创建，确认保留路径：

![allocation-tracker.png](allocation-tracker.png)

在[How to Use the Allocation Profiler Tool](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/allocation-profiler)学习怎样使用这个工具。


### 限定垃圾回收频率

垃圾回收器(比如在V8引擎中的这个)需要能在应用程序中定位活动的对象，也能定位死的对象(垃圾)和不能获得的对象。如果在做频繁的垃圾回收，你可能分配内存太过频繁了。

也要注意有趣的垃圾回收暂停。如果垃圾回收(GC)丢失了任何在你的JavaScript中由于逻辑错误导致的死对象，这些对象的内存消耗不能被收回。这种情形会随着时间最终降低你的应用程序的速度(详见[How to Use the Allocation Profiler Tool](https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/allocation-profiler))。

这经常发生在当你不需要的变量和事件监听器依然被一些代码引用时。维护这些引用时，这些对象不能被垃圾回收机制正确清理。

记得检查并注销包含DOM元素引用的变量，这些变量可能会在你的app的生命周期内更新或者摧毁。检查可能引用其他对象(或者其他的DOM元素)的对象属性。注意可能随时间推移累积的变量缓存。



### 内存分析资源

下面是从增长的内存泄漏DOM节点开始的一系列连续的测试变化的内存的例子:

- [Example 1: 增长的内存](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example1.html)
- [Example 2: 垃圾回收行为](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example2.html)
- [Example 3: 散落的对象](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example3.html)
- [Example 4: 分离的节点](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example4.html)
- [Example 5: 内存和隐藏类](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example5.html)
- [Example 6: 泄漏的DOM节点](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example6.html)
- [Example 7: Eval是魔鬼(几乎总是)](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example7.html)
- [Example 8: 记录对堆分配](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example8.html)
- [Example 9: DOM泄漏比预期的要更大](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example9.html)
- [Example 10: 保留路径](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/demos/memory/example10.html)

从下面的地址可以获得更多的demos：

- [收集散落的对象](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/heap-profiling-summary.html) 
- [判定行为清洁](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/heap-profiling-comparison.html)
- [探索堆内容](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/heap-profiling-containment.html)
- [揭秘DOM泄漏](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/heap-profiling-dom-leaks.html)
- [发现累积点](https://github.com/GoogleChrome/devtools-docs/blob/master/docs/heap-profiling-dominators.html)

#### 额外资源

[内存管理大师](https://youtu.be/LaxbdIyBkL0)Addy Osmani给你一个调试内存问题的速成课。[幻灯片展示](https://speakerdeck.com/addyosmani/javascript-memory-management-masterclass)和[样例代码](https://github.com/addyosmani/memory-mysteries)都是可以获得的。
视频地址：https://youtu.be/LaxbdIyBkL0

#### 社区资源
有许多社区成员写的用Chrome DevTools查找和处理web app内存问题的精彩资源。以下是有用的选择：

- [用Chrome DevTools发现和调试内存泄漏](http://slid.es/gruizdevilla/memory)
- [用DevTools进行JavaScript分析](http://coding.smashingmagazine.com/2012/06/12/javascript-profiling-chrome-developer-tools/)
- [Gmail级有效的内存管理](http://www.html5rocks.com/en/tutorials/memory/effectivemanagement/)
- [Chrome Devtools 2013大改版](http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/)
- [用DevTools渲染和内存分析](http://www.slideshare.net/matenadasdi1/google-chrome-devtools-rendering-memory-profiling-on-open-academy-2013)
- [用DevTools Timeline和Profile进行性能优化](http://addyosmani.com/blog/performance-optimisation-with-timeline-profiles/)




