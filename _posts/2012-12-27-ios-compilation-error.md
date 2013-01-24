---
layout: post
title: "ios编译错误总结"
description: ""
category: Programming/iOS
tags: [Objective-C, Xcode, notes, compilation error ]
---
{% include JB/setup %}


这篇blog记录了开发过程中遇到的编译时错误。

# 1. 系统库的头文件被改过

#### Xcode错误输出:
	
	fatal error: file '/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/../lib/clang/2.1/include/stdint.h' has been modified since the precompiled header was built
		
#### 原因:
	
如果打开系统库头文件，比如`stdint.h`并不小心改动，虽然改动不会影响系统里的头文件，但改动操作会被Xcode记录，再编译时就出这恶心的错误，重启Xcode还不能解决。
		
#### 解决：
		
删除~/Library/Developer/Xcode/DerivedData/{project name + gobly-gook}文件夹，CLEAN & CLEAR。

#### REF
	
<http://stackoverflow.com/questions/7071523/Xcode-4-1-fatal-error-stdlib-modified-since-the-precompiled-header-was-built>
		
# 2. 缺少依赖的`.o`文件，链接错误

#### Xcode错误输出:

	Undefined symbols for architecture i386   // 模拟器
	Undefined symbols for architecture armv7  // 真机调试
			
#### 原因：
	
发生这种错误通常是project.pbxproj这个文件引起的，尤其在多人合作开发的时候，svn提交不规范可能导致project.pbxproj发生错误，导致文件的引用不在project.pbxproj文件中。 Xcode项目import文件会根据project.pbxproj来查找，查找不到文件的引用则会有上述的错误。
		
#### 解决：
	
点击工程，在主界面中点击Build Phases，根据提示信息“XXX”来判断缺少什么文件，一般如果缺少自定义的文件，XXX会是缺少的类名，那么就在Complie Sources中加入该文件。如果缺少类库，则在Link Binary With Libraries中加入该类库。