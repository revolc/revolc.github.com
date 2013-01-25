---
layout: post
title: "关于NSUserDefaults"
description: ""
category: Programming/iOS
tags: [Objective-C, API]
---
{% include JB/setup %}

#### 概述

NSUserDefaults（[官方文档][1]）普遍用在存储用户定制数值，或者程序中需要存一个数据的场景。当NSUserDefaults写入文件后，其值会存在应用沙盒下的`Library/Preferences/com.xxx.AppName.plist`文件里（`com.xxx.AppName`是应用的标识符）。可以看到，这个文件是个plist文件，因此，它可以存plist可以存的所有数据类型：NSData, NSString, NSNumber, NSDate, NSArray, 或者NSDictionary。

它是针对每个应用适用的，即，只作用于自身应用内。sandbox的安全机制保证，它不能修改其他应用的值，也不会被其他应用修改。在程序里使用时，一般通过下面语句获取NSUserDefaults的引用：

	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	
这是个单例，而且是线程安全的([参见这里][2])。

#### 性能

既然，本质上说，它是个plist文件，那么为什么要用它来存一些程序里要用到的key值，而不直接用一个plist文件来存取呢？因为我们可以容易想见，如果程序的各个类都往一个文件中读写（别忘了，NSUserDefaults是个单例，而且它访问的文件也只有一个），那么当其中的key变多后，性能岂不是要变差？而如果各个类各自读写自己的plist文件，则不会使自己的文件里存了别人的key而导致文件变大、性能变差。

从文件的角度看，确实如此。但是，NSUserDefaults帮我们做了一层优化，NSUserDefaults是带缓存的。NSUserDefaults会把访问到的key缓存到内存里，下次再访问时，如果内存中命中就直接访问，如果未命中再从文件中载入。应用会时不时调用`[defaults synchronize]`方法来保证内存与文件中的数据的一致性，有时在写入一个值后也最好调用下这个方法来保证数据真正写入文件。

这样的机制带来了一定的性能提升，实测在10万个key值的情况下，通过NSUserDefaults来读取value是1ms级别的，而如果从自己写的plist里读取则是0.1s级别的开销。但是，如果用plist文件自己在应用里写入1个10万条记录的文件是1s级别的开销，而同时写入10万条NSUserDefaults键值对则是10s级别的延迟。原因在于，在创建key/value pair时，要在内存中也创建一个相应的映射，而系统时不时地调用`synchronize`方法会导致创建key/value pair被阻塞。

总之，读取NSUserDefaults是比较高效的，而程序不同地方对NSUserDefaults的写入操作也不会带来严重性能影响，但是，如果在一个地方大规模写入NSUserDefaults值则会是性能灾难。用plist文件读入内存在访问的瓶颈主要在读入的过程中，如果文件很大会比较耗时，但是一旦载好，在内存中读取就很快；但是，对内存值的写入则会造成内存与文件数据不一致，这时为了保证数据一致性就要写入文件，写入后又要读入内存，这就导致延迟。

#### Trap

在浏览NSUserDefaults的API时，发现有一个这个方法`initWithUser:`，本来以为这个方法会根据自己提供的username来创建个不同于系统维护的唯一的`standardUserDefaults`，结果发现是自己太天真，[这个SO帖][3]就解释了这个问题。

其实这个方法是`Available in OS X v10.0 and later`的，是给超级用户以一个用户来载入UserDefaults用的。

[1]: http://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/Reference/Reference.html "NSUserDefaults Class Reference"

[2]: http://appleparty.diandian.com/post/2012-05-24/9098104219 "Property List和NSUserDefaults"

[3]: http://stackoverflow.com/questions/9338915/am-i-misunderstanding-nsuserdefaults-initwithuser "Am I misunderstanding NSUserDefaults initWithUser?"