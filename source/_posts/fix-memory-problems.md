title: 处理内存问题
date: 2016-05-04 14:16:49
tags: [chrome, DevTools, profile]
author: jiliang

---


原文出自: https://developers.google.com/web/tools/chrome-devtools/profile/memory-problems/?hl=en
原文作者: Meggin Kearney

---


程序临时使用内存后不能释放内存时，会发生内存丢失。注意内存泄漏，溢出，和强制垃圾回收。

---

现在的JavaScript引擎在许多情况下可以高度自动化地清理我们的代码产生的垃圾。也就是说，它们也只能做到这样，我们的应用程序仍然容易发生逻辑错误造成的内存泄漏。用这些工具来找出你的瓶颈，记住，不要猜测，去测试。

---

### 主题
- [内存诊断]()
- [内存术语]()
- [如何记录堆快照]()
- [如何使用分配分析器工具]() 
