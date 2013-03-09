---
layout: post
title: "用Jenkins搭建iOS项目的CI服务器"
description: ""
category: SoftwareEngineer/Practice
tags: [iOS, Continuous Integration, Xcode, Jenkins]
---
{% include JB/setup %}

## 什么是CI

Continuous Integration(CI)是软件开发的一种方法论。在开发过程中，关注的是每一个小小的改进，如果这个改进编写完成，并由开发人员测试验证通过，在保证不破坏编译的前提下，便尽早提交到版本控制。这样做，可以保证新的feature开发后能被更快地送到测试人员手中进行测试；减少重复的过程，如测试人员对同一问题不反复验证；时刻保证有可部署的软件。

具体的概念见[wiki]("en.wikipedia.org/wiki/Continuous_integration")，还有跟CI相关的[一本书]("http://book.douban.com/subject/2580604/")推荐给大家。

## CI实践之iOS篇

### CI工具Jenkins

#### 选型

市面上有很多CI工具了已经，商业的，开源的都有。其他的具体都不了解，以前团队一直用Hudson，最近，Hudson项目又分出一个分支: Jenkins，决定尝试一下。

#### 安装与部署

Jenkins的使用so easy，去其[官网]("http://jenkins-ci.org/")上下载jenkins.war文件，目前的版本是v1.504。下载页上还有针对各个系统的native package，但对于我来说，用war文件的方式就很不错了，Java所谓的"一次编写，到处运行"，还是很方便的。

部署也很简单，在下载完成后，把war文件存在一个长久的位置，然后用java来运行这个war就OK了：

	$> mv ~/Downloads/jenkins.war /where/i/want/to/store/jenkins.war
	$> java -jar jenkins.war --httpPort=8080

这样，通过游览器访问`localhost:8080`这个URL，就可以看到Jenkins的界面了。
	
#### 插件

默认下载的Jenkins是没git和Xcode的支持的，而我们要使用git来做版本控制，编译iOS项目又需要Xcode命令行，这时候，Jenkins的插件化设计就很贴心了，可以通过插件来扩展Jenkins的功能。

在Jenkins界面下选"Manange Jenkins"->"Manage Plugin"，可以列出插件界面，在这里面选相要的插件，安装后重启Jenkins就OK了。

#### Job的创建

Jenkins里一个编译任务就是一个Job，配置好一个Job，当编码完成时，启动它，Jenkins就帮我们把编译、打包、发布的任务都做了。

在Jenkins界面左上角点击"New Job"开始建立Job。在Job的Configure里最重要的是git仓库的URL，以及Xcode的设置。

1. 取一个"Job Name"，最好单词间不要有空格。
2. 在配置界面里“Source Code Management”项目里，配置要使用的SCM工具，这里我们选Git，再配置其URL(如，"git@github.com:revolc/Jenkins-iOS-Demo.git")和分支(*/master、test等)
3. 在"Build"项目里，点选"Add build step"，在下拉列表里选"XCode"，出来Xcode编译的配置:
	A. 填一个"Target"，如"Jenkins-iOS-Demo"，这个就是Xcode工程最后要编译输出的Target，一般是跟工程同名，要填对，不然无法找到target就无法编译；
	B. 填一个"Configure"，如"Debug"
	C. 勾选"Build IPA?"及"Clean before build?"选项
4. 最后，点"Save"按钮就保存了这个Job。

#### 运行构建任务

当Job创建完成，就可以在Job列表里选择一个要执行的任务，在Job详细界面，点击左侧的"Build Now"按钮即开始构建。

构建完成会有对每个具体构建任务的详细信息，点击一个构建进去，可以查看这个构建的状态"Status"；还有这次构建相对上次构建的改进"Changes"；构建默认只有Build号，通过"Edit Build Information"，可以给构建一个名字及描述，更多贴心功能，期待发现...