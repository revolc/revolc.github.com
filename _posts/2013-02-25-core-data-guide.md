---
layout: post
title: "Core Data的使用"
description: ""
category: Programming/iOS
tags: [Objective-C, API, SQLite, Xcode]
---
{% include JB/setup %}

#### Core Data是什么

Core Data是Apple提供的一套数据持久化层，包含数据建模、数据操作等一系列技术，其底层的技术实现采用sqlite。既然是sqlite，为什么不用FMDB等框架直接操作数据库呢？原因在于，Core Data对数据操作做了优化，大大减少了资源消耗，例如，当列表的数据移出显示区域后，可以把这些表项所对应的数据从内存中释放，这样的应用对移动设备更友好。

#### 模型创建

一般使用Core Data