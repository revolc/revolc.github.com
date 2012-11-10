---
layout: post
title: "cocos2d Intro"
description: ""
category: Programming/cocos2d
tags: [cocos2d, notes, iOS, Game Development]
---
{% include JB/setup %}


# cocos2d是什么

cocos2d是一个游戏引擎，所谓的游戏引擎就是提供给编程人员一些好用的API，来处理图像、动画、音效、音乐等，帮助您方便地创建富媒体的游戏。当然，你聪明的完全可以自己调用设备、平台上的native的API来自己完成这些功能，但是，请相信我，这是一个痛苦而低效的过程，干嘛要重复创造轮子呢？

cocos2d一开始是实现在python上的，cocos2d-python，借助pyglet来实现的引擎，而后随着iOS平台的升温，开发了cocos2d-iphone。再然后，随着移动平台的火爆，cocos2d-x（以C++开发来实现跨平台的cocos2d版本）也出来了。BTW，貌似华人对cocos2d-x的贡献很大，（Ref: <http://paralaxer.com/what-is-cocos2d-x/>）。

>The Cocos2d-X project was started around July 4th, 2010. Primary contributors to the project have been Walzer Wang, Minggo Zhang and James Chen. Many other contributors have authored the project as well.

真是值得华夏儿女骄傲，但具体这3人是两岸三地或者东南亚哪的，待考。

总之，cocos2d是一系列不同平台上的游戏引擎的总称，cocos2d-python是第一个实现，然后又有了cocos2d-iphone，之后又有了cocos2d-x想要实现跨平台，最近又有了cocos2d-html5，是一个活跃的开发社区。

# cocos2d-iphone的安装

本文着重于cocos2d在iOS上的安装。

1. 获取源码

		$>git clone https://github.com/cocos2d/cocos2d-iphone.git

	或者去这个链接<https://github.com/cocos2d/cocos2d-iphone>下zip包，获取cocos2d-iphone的源码。

2. 安装模板
	
	cocos2d-iphone的源码里带了xcode的工程模板，使用模板可以快速方便地创建游戏工程。

	下载完成后，进入到cocos2d-iphone目录下，执行安装模板的脚本
		
		$> cd cocos2d-iphone
		$> ./install-templates.sh -f
		
		
3. 跑个Hello World!

	打开xcode，新建一个工程，选择相应的cocos2d模板（纯粹cocos2d、cocos2d带Box2D物理引擎的、cocos2d带chipmunk物理引擎的），会创建一个帮你引用了相关cocos2d或者物理引擎库文件的工程，而且这个工程直接运行会有个小黑屏显示个`Hello World`。