---
layout: post
title: "给iOS工程添加Build号"
description: ""
category: SoftwareEngineer/Practice
tags: [Continuous Integration, iOS, script, Xcode, Git]
---
{% include JB/setup %}

几乎所有的工程都有版本号，但是并不是所以的工程都有build号。有时候，build号其实比版本号更有用。因为，版本号是对外发布时用的，是用户看的，而build号则不直接与版本号相关，它自己单调递增，用来标记出的包的第N次build的id性质的东西。以前的工程里一直只有版本号没有build号，导致与测试人员交流不畅，不知测试说的bug在哪个版本的代码上产生，深深困扰，直到，把build号整进来。。。


### 1. 从配置文件中的累加

每个iOS工程都有个`${PROJECT_NAME}-Info.plist`文件，其中`${PROJECT_NAME}`是build时的环境变量，可以在build的某个阶段中跑脚本来进行处理。

	buildPlist=$PRODUCT_SETTINGS_PATH
	buildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBuildNumber" "$buildPlist")
	buildNumber=$(($buildNumber+1))
	/usr/libexec/PlistBuddy -c "Set CFBuildNumber $buildNumber" "$buildPlist"
	
上面的脚本需要在`${PROJECT_NAME}-Info.plist`文件中，加入一个键值对，CFBuildNumber及其对应的值。

上面说，在某个阶段，那么，具体可以有哪些阶段？一种是在Build时，在Build Phrase里做；一种是直接编辑Xcode的编译的scheme，在`Archive`这一scheme里添加`Post-actions`。

####方式1、添加Build Phrase

在Xcode的Project Navigator里选工程文件，在Build Phrase这一tab里，点右下角的"Add Build Phrase" -> "Add Run Script"，把上述脚本贴进去就行。Run Script的阶段选在Copy Files后面，是可行的，其他的顺序没有验证过。

####方式2、scheme里添加Post-actions

在Xcode菜单里，选"Product" -> "Edit Scheme…" -> 展开左栏的"Archive" -> "Post-actions"里点击"+"号 -> "New Run Script Aciton"

备注：工程的配置文件，终极是用${PRODUCT_SETTINGS_PATH}来配置的 

### 2. 直接从版本库取

####ver.1

`git log`加`-n`选项，可以限制最多显示几行log，由于log是按时间逆序排的，所以`git log -n 1`即显示最近的一次提交的log

	version=`git log -n 1 | head -n 1 | cut -d ' ' -f2 | cut -c1-6`
		
cut命令用来载取输入流中的某些字段，选项`-d ' '`表示以空格作为分隔符，`-f2`表示选择第2个字段，即commit的hash，再用cut命令来substring前6个字符(一般来说，6个字符就够区分不同的hash了)。
			
####ver.2

上面的方案只能说解决了需求，但是还是不完美，通过`git log --help`查`git log`的选项，发现可以通过设置选项来格式化`git log`的输出样式，于是有了下面这行：

	version=`git log -n 1 --pretty=oneline --abbrev-commit | cut -d ' ' -f1`
	
备注：如果更喜欢大写字母的Build号，像图中那样的，可以再在管道后加一个大写转换：`|tr [a-z] [A-Z]`

####ver.3

前面的命令可以进一步化简：

	version=`git log -1 --pretty=format:"%h"

