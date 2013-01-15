---
layout: post
title: "lipo命令使用"
description: ""
category: Objective-C/tools
tags: [lipo, Objective-C, commands]
---
{% include JB/setup %}

#lipo命令使用

Mac下的`lipo`命令是用来产生`universal`文件的。所谓的`universal`是从Intel版本的Mac出现后（大概是06年左右，从Mac OS X Tiger时代开始）才有的提法.因为先前的Mac只支持PowerPC架构，后来Mac产品全线转到Intel，这就有一个过渡阶段，因为对老版本的PowerPC Mac还得做一些支持，因此就可以用`universal binary`的方式，用同一个二进制可执行文件对多个架构进行支持。当然，还有一种技术，苹果也申请了商标，叫`Rosetta`，相当于用Intel来运行出一个PowerPC模拟器，来跑那些还没对Intel架构重新编译的软件。（`Rosetta`其实是个隐喻，解释请移步这里<http://en.wikipedia.org/wiki/Rosetta_Stone>）

言归正传，lipo是对`universal`二进制进行CRUD的命令，以下引自`lipo`的man page：


	The lipo command creates or operates on ``universal'' (multi-architec-
       ture) files. 
       
### Read

列出一个二进制文件的架构，这里有Mac自带软件Photo Booth为例：

	lipo -info /Applications/Photo\ Booth.app/Contents/MacOS/Photo\ Booth 

还可以列出更全的信息：

	lipo -detailed_info /Applications/Photo\ Booth.app/Contents/MacOS/Photo\ Booth 
	
验证某个架构是否包含在文件中：

	lipo  /Applications/Photo\ Booth.app/Contents/MacOS/Photo\ Booth -verify_arch x86_64
		
### Create

这里的产生一个新的二进制文件的方式一般不是平空产生（拿个编辑器直接写出二进制的机器码，那obj-c，编译器都可以下岗了），一般是从几个架构的文件揉成一个新文件：

	lipo -create Debug-iphonesimulator/libMAMapKit.a Release-iphoneos/libMAMapKit.a -output libMAMapKit.a	

### Update

#### 从输入文件中删除支持的一个架构

有时候，软件是对多个架构编译的，而如果你确定只用Intel的64bit架构的话，可以对其瘦身，用`-thin`参数。

### Delete

##### 删除操作对象里的一种架构

	lipo Photo\ Booth -remove x86_64 -output test_out
	
能删除的前提是这个二进制文件是含多种架构的`fat`的，也就是相对于只含有一种架构的`thin`的，这点上，苹果的东西都比较哨皮，把复杂的概念平实化，而微软总要把简单的东西高深化。。。

