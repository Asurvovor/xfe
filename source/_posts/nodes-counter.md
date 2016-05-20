title: Timeline面板记录节点数
date: 2016-05-20 17:50:24
tags: [memory, nodes, GC, timeline]
author: jiliang

---

直接举例子说明nodes的统计方法，以及删除节点时的一个注意事项。

先上源代码：

```javascript
<!doctype html>
<html>
<head>
	<meta charset="utf-8" />
</head>
<body>
<ul id="a">
	<li></li>
	<li></li>
	<li><span id="b">xxx</span></li>
</ul>
<input type="button" onclick="removeA();" value="移除"/>
<input type="button" onclick="gca();" value="gca"/>
<input type="button" onclick="gcb();" value="gcb"/>

<script>
var a = document.querySelector('#a');
var b = document.querySelector('#b');
function removeA() {
	a.parentNode.removeChild(a);
};
function gca() {
	a = null;
};
function gcb() {
	b = null;
}
</script>
</body>
</html>

```

------

以下是操作步骤：

首先，测试原始的节点数量。
在Timeline面板勾选Memory，然后开始录制，先点一下强制垃圾回收，然后点finish。具体如图：
![init.png](init.png)
![gc-record.npg](gc-record.png)
可以测得初始节点数为36.

第二部，测试removeA执行时后的节点数。
录制过程中点一次按钮“移除”。操作过程如下图：
![event-record.npg](event-record.png)
可得此时节点数仍为36.没有变化。
这时页面里已经看不到整个列表，但是节点依然存在，依然占用内存。

第三步，测试`a = null;`后的节点数。
录制过程中点一次“gca”按钮。
最终可得节点数仍为36.说明节点依然存在。

最后，测试`b = null;`执行后的节点数。
录制过程中点一次“gcb”按钮。
节点数为26.`ul`列表的所有节点清楚完毕。

----

以上实验说明：
- 当子节点有JavaScript引用时，removeChild方法不能删除节点，节点都还占用内存。
- 需要把所有的带有JavaScript引用的节点全部设置为null，才能释放内存。


-----







