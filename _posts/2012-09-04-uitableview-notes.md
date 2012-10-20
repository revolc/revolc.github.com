---
layout: post
title: "UITableView notes"
description: ""
category: Programming/iOS
tags: [iOS, notes]
---
{% include JB/setup %}

## deselectRowAtIndexPath

在用默认style构造一个table后，在
	
	- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath	
代理来处理点击事件，push一个view，再从该view返回时，被点击的table view的cell依然是点亮状态（蓝色，也就说处于选中状态），如何消掉呢？
用以下方法：

	    [theTableView deselectRowAtIndexPath:indexPath animated:NO];

该调用需知道row的indexPath，在didSelectRowAtIndexPath代理里调用上述方法。
