---
layout: post
title: "以依赖工程的方式配置cocos2d库"
description: ""
category: Programming/cocos2d
tags: [cocos2d, notes, iOS, Game Development]
---
{% include JB/setup %}

#以依赖工程的方式配置cocos2d库

安装cocos2d很简单，装好后，可以用模板的方式创建一个基于cocos2d的工程，但这样就失了灵活性，比如，cocos2d不支持ARC，那么在工程里要对一堆cocos2d的源文件设`-fno-objc-arc`,挺烦的。当然，这些限制不限于此，当你工程进展到一定程度时才发现这些限制很致命的话，就很抓狂了。

所以，这篇要讲怎么用把cocos2d工程作为依赖工程的方式来建立新的游戏工程。

[上文](http://revolc.github.com/Programming/cocos2d/2012/11/11/cocos2d-intro/)已经讲了cocos2d-iphone的安装，书接上回，其实下载的cocos2d-iphone目录下有几个工程文件是可资利用的，`cocos2d-ios.xcodeproj`





2. Build Settings -> User Header Search Paths添加cocos2d.h所在目录，这样就可以引用`cocos2d.h`了。


就这么简单，编译，运行！