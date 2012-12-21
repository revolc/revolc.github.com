---
layout: post
title: "Opaque Type"
description: ""
category: Programming/iOS
tags: [objective-C, notes]
---
{% include JB/setup %}


# Opaque Types - 不透明类型

看Apple的文档只会不经意间就遇到这个词，于是就会好奇到底是什么东东？官方的解释在这：<http://developer.apple.com/library/mac/#documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/OpaqueTypes.html>：

> 	The individual fields of an object based on an opaque type 
> 	are hidden from clients, but the type’s functions 
> 	offer access to most values of these fields.

下面是我的理解：

首先，它是个type，本质上，是C语言的structure。它想像`class`那样对数据和方法进行封装，但C语言没有能力提供这样的机制，所以，它更多的是对数据进行封装。有时候，会`class`和`type`混用，是取`type`有`class`的封装这一含义。

其次，它是不透明的。`opaque`是“不透明的；无光泽的，晦暗的；不传导性的；含糊的，迟钝的”，这里应该是指它对外屏蔽了内部的成员。

文档里用CFString进行举例说明：


![release](/assets/images/opaque_type/opaquetypes.png)

如上图所示，CFString这一structure/type里存在相应的field，但操作这一class，包括初始化、读取属性、复制、销毁等，都是用CFString打头的函数来的（而不是对CFString“对象”发消息来的）。

不禁要问，这样做除了增加开销，还有什么意义？

官方给出了答案：

1. 封装
 
如果用C语言的`.`来直接对访问CFString的成员，可以想见，如果哪天Apple更改了CFString成员的名字，你的代码就不可用了。
	
2. 屏蔽细节

CFString一般是16bit的Unicode字符的数组，但也会有优化，有时，只有8bit的ASCII字符。如果我们直接用数组下标来访问里面的元素，你就不知道什么时候要以2字节为间距来跳，什么时候要以1字节来跳。
	
3. 性能上也不差

官方的说法是CPU的cost不会高于、内存则会少于直接用C语言来操作。以CFString这个例子来说，如果用C的数组方式，我们可能要开一个数组，以2字节的数据为元素；而且CFString会针对只用ASCII就可表示的字符采用8bit的ASCII来存储，这样，内存消耗就小。