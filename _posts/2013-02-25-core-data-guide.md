---
layout: post
title: "Core Data的使用"
description: ""
category: Programming/iOS
tags: [Objective-C, API, SQLite, Xcode]
---
{% include JB/setup %}

### Core Data是什么

Core Data是Apple提供的一套数据持久化层，包含数据建模、数据操作等一系列技术，其底层的技术实现采用sqlite。既然是sqlite，为什么不用[FMDB]等框架直接操作数据库呢？原因在于，Core Data对数据操作做了优化，大大减少了资源消耗，例如，当列表的数据移出显示区域后，可以把这些表项所对应的数据从内存中释放，这样的应用对移动设备更友好。

### 模型创建

Core Data用`.xcdatamodeld`文件来描述数据模型之间的关系，`.xcdatamodeld`文件的产生可以是：1)新建一个使用Core Data的Xcode工程时就带的；2)用Xcode新建文件操作，选 _"iOS->Core Data->Data Model"_ 来产生，需要。

有了`.xcdatamodeld`文件就可以在图形化界面里进行模型的属性和关系的编辑。可以操作的对象有`Entity`和`Attribute`，对应于数据库中的表和字段。与`Entity`平行的概念有`Fetch Request`，可以理解为一个视图，本质是个SQL语句；与`Attribute`平行的概念有`Relation`，可以理解为一个外键联系。

Apple推荐的Entity1到Entity2有个Relation，该Relation最好能从Entity2指回Entity1。在关系的Inspector中，设置`Inverse`值即可。如果关系设为了一一对应，那么须把`Delete Rule`设为`Cascade`，这样一个Entity1对象被删了，那么相应的Entity2对象也被删，确保不会出来脏数据。


### CRUD里的CR



#### 数据填充





### Trouble Shooting

1. 模型更新

有时，更新了Model文件，重新跑应用进就会崩溃。原因很简单，旧的应用有persistent store，但不知道怎么样来读新的Model，自然就崩溃了。解决办法有2个：1)删除旧应用，全新安装；2)数据迁移。第1种方式简单粗暴，但就意味着用户不能更新应用，所以，后面会讲怎样进行数据迁移。

2. 打开sql调试

既然Core Data的底层是sqlite，那么有时出现问题时，我们就有必要查看它执行的sql语句来定位问题。可以通过如下方式打开sql调试：在 _"Product->Edit Scheme…->Run->Arguments"_ 面板添加选项:“-com.apple.CoreData.SQLDebug 1”。再次运行就可以在console里看到sql语句了。


[FMDB]: https://github.com/ccgus/fmdb