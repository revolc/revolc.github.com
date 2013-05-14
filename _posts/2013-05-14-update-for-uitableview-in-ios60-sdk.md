---
layout: post
title: "UITableView更新在iOS6.0"
description: ""
category: Programming/iOS
tags: [UITableView,　iOS, API, Xcode]
---
{% include JB/setup %}

## UITableView更新在iOS6.0

0. 使用UITableView，或者UITableViewController基本步骤都差不多，后者只不是一个包装了UITableView的UIViewController，意义在于更方便写代码吧。
使用UITableView都需要设置下<UITableViewDataSource>，用来给tableview喂数据，这个<UITableViewDataSource>里要做的事是：

	a. 告诉上层的UITableView我有几个section，几行数据要展示：`- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;`
	
	b. 具体要怎么展示数据：`- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;`

	在上面b的方法里，如果每行都要实例化一个UITableViewCell对象，那么由于设备内存有限，如果数据多，那程序要crash的，所以采用的机制是实例化一屏（大概十来个）的UITableViewCell对象，当用户滑动要查看后面的UITableViewCell时，前面的对象就会被复用，而不会再初始化新的对象。这样就算有1000行数据，但实际只有10个对象，翻到后面时候，新的数据被绑定到复用的UITableViewCell对象上，从而保证内存不爆。

	如果有不明了处，一定是我没表达好，可以参见很多现有的博客，[关于UITableViewCell的重用初探](http://blog.csdn.net/likendsl/article/details/7356944)就可以一看。


1. iOS6.0前，使用的这种机制编程的范式如下

		UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
		
		//如果没有可回收的cell,　就实例化一个对象

		if (!cell) {
	        cell=[[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier]autorelease];

    	}

	    //为cell对象绑定数据
	    cell.textLabel.text=[NSString stringWithFormat:@"第%d行",indexPath.row];
 

2. 由于SDK升级，6.0提供了更多的功能

	`- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0);`，这个方法从这方法签名上看，似乎可以提供异质的cell，在6.0前，dequeueReusableCellWithIdentifier没有indexPath参数，可以想见，所有的row对应同样的cell结构，而6.0则可以为不同indexPath缓存不同的cell类型，所以是提供了更多的功能。
	
	至于兼容性问题，如果只用`- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier`方法，不传indexPath参数，那么iOS6.0前的设备都可以跑；而使用`- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath`则只能跑在iOS6.0后的设备。

	同时，如果要用`- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath`还是需要一番配置的，要使用下面2种方法中的一个来为cell对象绑定layout布局：
	
		- (void)registerNib:(UINib *)nib forCellReuseIdentifier:(NSString *)identifier NS_AVAILABLE_IOS(5_0);
		- (void)registerClass:(Class)cellClass forCellReuseIdentifier:(NSString *)identifier NS_AVAILABLE_IOS(6_0);
		
	在UITableView初始化后，获得UITableView对象的引用，调用上述方法中的任一个，视乎采用xib方式或者代码的方式。
