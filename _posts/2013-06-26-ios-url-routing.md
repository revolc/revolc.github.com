---
layout: post
title: "iOS URL routing"
description: ""
category: programming/ios
tags: [iOS, URL]
---
{% include JB/setup %}

### 问题的提出

在iOS应用中，应用内的某界面调用其他界面时，传统的做法是在调用处初始化另一个界面的实例，然后不管用`pushViewController:animated:`或者是`presentModalViewController:animated:`等方法来展示。如在A界面中使用下面2行代码：

	...
	UserViewController *viewController = [[]UserViewController alloc] init];
    [[self navigationController] pushViewController:viewController animated:YES];

这样做固然简单，但是造成了A代码与UserViewController的耦合；而且因为是写死的，不利于扩展。如果这里采用的是打开一个URL，同时，在应用程序其他部分能对这个URL响应并进行相应的处理，则对这块逻辑解耦了。如：

	...
	[[UIApplication sharedApplication] openURL:userUrl];

### URL是什么？

根据[RFC1808](http://www.ietf.org/rfc/rfc1808.txt):

	A Uniform Resource Locator (URL) is a compact representation of the
	   location and access method for a resource available via the Internet.

URL是用来表示互联网上资源的位置及访问方法的。


### iOS中URL处理逻辑

在iOS应用中，当你在应用中触发一个事件，或者在UIWebView点击web页面上的一个超链接时，都可以打开一个URL。

原生应用可以用下面代码实现：

	[[UIApplication sharedApplication] openURL:someUrl];

HTML文件中，可以用下面代码定义一个URL：

	<a href="mailto:frank@wwdcdemo.example.com">John Frank</a>

上例中的"mailto"是URL的scheme，iPhone应用支持的URL scheme可以参见这个[官方文档](http://developer.apple.com/library/ios/#featuredarticles/iPhoneURLScheme_Reference/Introduction/Introduction.html)。

当然，这些仅仅是系统支持的，如果一个应用想要定义一个自己的URL scheme，也可以实现，需要如下步骤：

1. 在Info.plist文件里加CFBundleURLTypes这一Array
2. CFBundleURLTypes添加元素，Dictionary类型，需有CFBundleURLName键，一般对应反向域名；还有CFBundleURLSchemes键，其值为Array，Array里的每个元素就是自定义的scheme

示例如下：

	<key>CFBundleURLTypes</key>
	<array>
	    <dict>
	        <key>CFBundleURLName</key>
	        <string>me.dayujun.demo</string>
	        <key>CFBundleURLSchemes</key>
	        <array>
	            <string>myapp</string>
	        </array>
	    </dict>
	</array>

定义完后，一旦打开"myapp://..."的URL，根据系统的版本，就会调用应用AppDelegate里的`application:openURL:sourceApplication:annotation:`(4.2及以后)或者`application:handleOpenURL:`方法(4.2以前)。

在这里实现你对自己定义的URL scheme的逻辑，这样就实现了URL的路由。

### 第3方库

当然，已经有不少第3方库实现了URL的路由，只需在代码里做相应的配置即可。在github上看比较有潜力的是下面这2个库[JLRoutes](https://github.com/joeldev/JLRoutes)和[Routable](https://github.com/routable/routable-ios)。今天花时间使用并比较了2个库，结果如下：

#### Routable

Routable的使用很简单，在AppDelegate的`application: didFinishLaunchingWithOptions:`方法里调用JLRoutes的`addRoute:handler:`方法里做URL映射，注意，要给router设一个UINavigationController的实例，以便router对ViewController进入跳转，如：

	[[Routable sharedRouter] map:@"stuff://users/:id" toController:[UserCardViewController class]];
    [[Routable sharedRouter] setNavigationController:self.naviController];
    
可以给不同的URL以不同的UPRouterOptions对象的跳转选项，来决定ViewController打开的方式。

还有独特的匿名映射的方式：

	[[Routable sharedRouter] map:@"invalidate/:id" toCallback:^(NSDictionary *params) {
	  [Cache invalidate: [params objectForKey:@"id"]]];
	}];
	
	[[Routable sharedRouter] open:@"invalidate/5h1b2bs"];

可以在想跳转一个callback的地方临时映射个URL。

然后在映射上的类里实现下列方法：

	- (id)initWithRouterParams:(NSDictionary *)params
	
该方法将获得URL传来的参数，用来进行业务相关的操作。

调用URL的地方用：

    [[Routable sharedRouter] open:urlStr];
	
传入可识别的URL对应的NSString，即可进行相应处理。如果传入的URL不能处理，则会抛出异常，所以最好用`openExternal:`的方法来打开外部的URL：

	[[Routable sharedRouter] openExternal:@"http://www.youtube.com/watch?v=oHg5SJYRHA0"];


#### JLRoutes

JLRoutes的使用也很简单，而且使用了block来传递业务逻辑。在AppDelegate的`application: didFinishLaunchingWithOptions:`方法里调用JLRoutes的`addRoute:handler:`方法，handler里传入的参数是个block: `(BOOL (^)(NSDictionary *parameters))handlerBlock`，可以看见，这个block以parameters字典为入参，返回一个BOOL值。这个parameters字典就是从URL里解析的参数，返回参数如果为YES，则表示这个URL在这层处理完了，如果为NO，则可能递交下一层次处理。实际上这个block就是URL映射对应的业务逻辑的处理部分。

parameters字典里至少有下列键值：

	{
	  "JLRouteURL":  "(the NSURL that caused this block to be fired)",
	  "JLRoutePattern": "(the actual route pattern string)",
	  "JLRouteNamespace": "(the route namespace, defaults to JLRoutesGlobalNamespace)"
	}

JLRoutes有命名空间的概念，默认的是全局空间：`static NSString *const kJLRoutesGlobalNamespaceKey = @"JLRoutesGlobalNamespace";`。可以用下列调用指定命名空间：

	[[JLRoutes routesForScheme:@"stuff"] addRoute:@"/foo" handler:^BOOL(NSDictionary *parameters) {
	  // This block is called for stuff://foo
	  return YES;
	}];
	
如果一个URL在给定的scheme上不match，但是如果在全局空间上可以match，根据需要，可以对一个空间设置是否退回到全局空间上进行匹配：

    [JLRoutes routesForScheme:@"thing"].shouldFallbackToGlobalRoutes = YES;

调用的地方用：

	[[UIApplication sharedApplication] openURL:someUrl];
	
传入可识别的URL，即可进行相应处理。

#### 比较

通过上面2种库的集成，得出结论：

1. Routable会用系统提供的ViewController跳转方式进行界面跳转，而JLRoutes则完全要自己写跳转逻辑看，但这样粒度更小，给用户更多的可能性，未必不是个优点；
2. Routable的匿名映射URL方式调用block的方式，跟直接调用个block相比，几乎没有什么先进性；
3. Routable传递URL是用NSString，而JLRoutes是个NSURL，而且有更多URL参数解析的功能；Routable需要侵入已有代码，添加`initWithRouterParams:`实现，其实代码耦合度更大。

各个方面看，JLRoutes都完胜Routable，唯一不好一点是，JLRoutes如果在`application: didFinishLaunchingWithOptions:`里实现映射的URL对应的逻辑，会导致AppDelegate类越来越大，但是这个问题可以通过把这些逻辑挪到另一个类中轻松解决。

综上，JLRoutes似乎是更好的选择，而且Hacker News上[一个帖子](https://l2exilium.net/item?id=5504404)的一句话`Checkout https://github.com/joeldev/JLRoutes by a former Apple iOS employee.
`更坚定了我的选择
