---
layout: post
title: "Bundle Programming Guide笔记"
description: ""
category: Programming/iOS
tags: [Bundle, App, iOS, Mac, API, Objective-C]
---
{% include JB/setup %}

本篇是苹果的《Bundle Programming Guide》文档的笔记，原文档在[这里](http://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFBundles/CFBundles.pdf)。


## Bundle vs. Package

package，只要Finder把其当成单个文件的目录。

bundle, 是一个目录，包含定义好的层次化结构并把可执行代码与资源文件一同保存。

Mac里的一个应用就是一个package，一般情况下是系统将其当成一个文件来处理，但是，如果在右键菜单里选择"Show Package Contents"，用户就可以把它当成目录来进行相应处理(当然，修改程序包里的内容是极不推荐的行为)。因此，package是一个提升用户体验的抽象。

而bundle则主要用来方便应用开发者，比如，用预先定义好的结构生成文件，那么l10n(什么是l10n？戳[这里][l10n definition])就比较方便。具体的bundle结构取决于应用针对的平台或者插件的类型。

## 如何识别

1. 以特定的后缀结尾的文件: .app, .bundle, .framework, .plugin, .kext等
2. 某些应用可以声明一个后缀名来表示package
3. 目录的package标识位写入了值

*一些注意点：*

1. 绝大多数可执行文件可以bundle起来，比如Application, framework(shared libraries), plug-in等；但是静态/动态库、shell脚本、UNIX命令行工具不用bundle结构
2. bundle起来的应用可以直接从服务器跑，不需要依赖本地系统的共享库、插件、资源等。

主要bundle类型

1. Application
	
	用来管理可执行的进程所需的代码与资源文件，具体结构取决于面向的平台，iOS或者Mac OS X。
	
2. Framework
	
	用来管理动态共享库及其相应资源，比如头文件，这样，应用就可以用它来链接出最后的可执行文件。
	
3. Plug-in

	主要针对OS X。
	
## Bundle的创建

一般而言，用户不用自己创建bundle，开个Xcode工程，最后Xcode会帮我们生成bundle。但是，不是所有的Xcode模板都会生成bundle，如果是要创建一个command line tool或者static libraries，就不会生成bundle。同时，如果使用make file这一套工程管理方式，也不会生成bundle，而是直接生成二进制可执行文件。

Bundle文件的一些guideline

1. 一定要包含Info.plist文件，而且其中最好设置了bundle类型相应的一些重要的key值。详见《Runtime Configuration Guidelines》

2. 应用运行所依赖的资源文件，如图片、字符串文件、l10n资源及插件等，应该都包含进bundle，不重要的资源可以考虑存储在磁盘上。

3. 如果要调包含Java代码的bundle，使用NSBundle类，而不要使用CFBundleRef的C引用。

4. OC代码的bundle可以用2种方式来装载，但效果不同。CFBundleRef类型的是立即绑定符号，但是是私有的；NSBundle类型的是惰性加载，而且是全局装载的，并且在装载完后会发NSBundleDidLoadNotification。

## bundle的通用文件结构

#### Info.plist
必需的，用来配置应用的运行时，系统依赖这个文件来获得应用的相关信息，比如应用要用到的权限等。

#### 可执行文件
必需的，是应用的起点，必须对所面向的平台静态编译。

#### 资源文件
程序运行所需的图片、图标、声音、nib文件、字符串文件、配置文件及数据文件等，具体位置因平台而异。

####其他支援文件
Mac应用可以包含更高层的资源，比如，私有的framework，插件，文档模板及其他自定义的数据资源；而iOS应用只能包含数据资源，而不能包含framework及plug-in等

### iOS应用的bundle解剖

iOS应用的bundle结构定义更多地为移动设备的限制考虑，使用扁平化的结构，而不用多层次的结构，以节省磁盘空间。

	MyApp.app
		MyApp
		MyAppIcon.png
		MySearchIcon.png
		Info.plist
		Default.png
		MainWindow.nib
		Settings.bundle //在Settings中显示应用的设置
		MySettingsIcon.png
		iTunesArtwork
		en.lproj
			MyImage.png
		fr.lproj
			MyImage.png
			
### Info.plist文件主要键介绍

#### CFBundleDisplayName
(Bundle display name)

在应用ICON下显示的名字，最好国际化之

#### CFBundleIdentifier
(Bundle identifier)

用来标识应用，一般是反域名形式给出

#### CFBundleVersion
(Bundle version)

单调递增的字符串，由多个点分的整数构成，不能将这个值国际化。

#### CFBundleIconFiles
iOS 3.2后支持。

#### LSRequiresIPhoneOS
(Application requires iOS environment)

标识应用只能在iOS环境中运行，创建工程时Xcode会自动加上并设置为true，不能修改这个值！

#### UIRequiredDeviceCapabilities
用来申明应用的权限，值可为NSArray或者NSDictionary，其中NSDictionary中的键为所需的权限，值全为true，NSArray中的各个元素均是权限名。

### Info.plist常见键

#### UIStatusBarHidden
布尔值，用来标识是否显示状态栏

#### UIStatusBarStyle
具体值为UIApplication.h定义的。

#### UIInterfaceOrientation
UIApplication.h中定义的常量。

#### UIPrerenderedIcon
布尔值，默认为false，表示需要系统帮我们生成高光和斜角效果，如果自己设计的icon不需要系统的任何效果，设为true

##### UIRequiresPersistentWiFi
布尔值，默认为false。为true时表示应用始终需要WiFi连接，否则系统会在30分钟后关闭WiFi以省电。但是，如果设备屏幕关闭后，就算为tru有也没有用，苹果把关闭屏幕的状态当成应用不活跃的状态。

##### UILaunchImageFile
启动图片的文件名，如果为空则用Default

### Framework Bundle

Framework是用来封闭动态共享库及相应的资源文件的层次化目录，framework通过定义头文件来向外界提供该框架的符号以进行链接。Framework的意义是用来提供通用的、共有的代码，以减少应用的内存消耗、提升系统性能。

OS X里可访问的framework在`/System/Library/Frameworks/`目录下，而iOS的framework放在相应版本的SDK的`System/Library/Frameworks/`目录下。

Framework里Versions子目录下包含了每个framework版本对应的代码与资源，用符号链接的方式指当最新的版本。

	MyFramework.framework/	   MyFramework  -> Versions/Current/MyFramework	   Resources    -> Versions/Current/Resources	   Versions/    	  A/        	 MyFramework	         Headers/    	        MyHeader.h        	 Resources/            	English.lproj/               		InfoPlist.strings           		Info.plist			Current -> A

### Loadable Bundle

Plug-in及其他类型的loadable bundle可以用来动态改变应用的行为，具体见"Code Loading Programming Topics"

### Bundle的l10n

bundle的本地化一般是由`language_region .lproj`这一目录里的资源来进行的，其中`language`部分是一个2字符的语言代码，由ISO639标准来定义；`region`部分是一个2字符的地区代码，由ISO3166标准来定义。

### Bundle的API

在获取bundle中的文件，先要获得bundle的引用，具体有如下方式：

##### 获得mainBundle的引用：

	// ObjC API: Get the main bundle for the app.
	NSBundle* mainBundle;	mainBundle = [NSBundle mainBundle];
	// C API: Get the main bundle for the app	CFBundleRef mainBundle;	mainBundle = CFBundleGetMainBundle();
##### 获得具体路径下的bundle：
	// ObjC API: Obtain a reference to a loadable bundle.
	NSBundle* myBundle;		
	myBundle = [NSBundle bundleWithPath:@"/Library/MyBundle.bundle"];
	// C API: Get the main bundle for the app	CFURLRef bundleURL;	CFBundleRef myBundle;
	// Make a CFURLRef from the CFString representation of the	// bundle’s path.	bundleURL = CFURLCreateWithFileSystemPath(                kCFAllocatorDefault,                CFSTR("/Library/MyBundle.bundle"),                kCFURLPOSIXPathStyle,                true );	// Make a bundle instance using the URLRef.	myBundle = CFBundleCreate( kCFAllocatorDefault, bundleURL );	
	// You can release the URL now.	CFRelease( bundleURL );	
	// Use the bundle...	// Release the bundle when done.	CFRelease( myBundle );
##### 用Bundle Identifier来获取：

	// ObjC API:
	NSBundle* myBundle = [NSBundle bundleWithIdentifier:@"com.apple.myPlugin"];
	
	// C API: Look for a bundle using its identifier
	CFBundleRef requestedBundle;	requestedBundle = CFBundleGetBundleWithIdentifier(        CFSTR("com.apple.Finder.MyGetInfoPlugIn") );
        
###获取Bundle中的资源

NSBundle的一大意义是不用知道资源文件的具体位置，只要遵循预先定义好的规则就能根据地区、语言等设置取出正确的文件。设计初衷的实现，有赖于预先定义搜索资源文件的规则：

1. 全局的资源文件（没有l10n的）
2. 指定地区的资源文件
3. 指定语言的资源文件
4. bundle开发者所用语言（由Info.plist里CFBundleDevelopmentRegion的键设定，当无法决定bundle所处环境的语言时，这个值是最后一根稻草）

资源文件与设备相关

自iOS4.0以后，可以标记某个特定的资源文件只适用于某一类型的设备，用下面的方式来命名资源文件：

	<basename><device>.<filename_extension>
	
`<basename>`是一个资源文件的标识符，用来区分不同的资源文件；`<filename_extension>`是文件后缀名，用来标识文件类型；`<device>`是大小写敏感的字符串，目前合法的值有`~ipad`、`iphone`。

获取资源文件路径

有了NSBundle引用，就可以用下列的方法来获取要查找的资源文件的具体路径：

	pathForResource:ofType:
	pathForResource:ofType:inDirectory:
	pathForResource:ofType:inDirectory:forLocalization:
	pathsForResourcesOfType:inDirectory:
	pathsForResourcesOfType:inDirectory:forLocalization:

注意，上面的方法中有获取单个文件路径的，也有获取一系列文件路径的，区别在方法是以path还是以paths开始。

一个OC的示例：

	NSBundle* myBundle = [NSBundle mainBundle];	NSString* myImage = [myBundle pathForResource:@"Seagull" ofType:@"jpg"];

	NSBundle* myBundle = [NSBundle mainBundle];	NSArray* myImages = [myBundle pathsForResourcesOfType:@"jpg"                              inDirectory:nil];	
同样的，也有CoreFoundation的API：

	CFBundleCopyResourceURL
	CFBundleCopyResourceURLInDirectory
	CFBundleCopyResourceURLsOfType
	CFBundleCopyResourceURLsOfTypeInDirectory
	CFBundleCopyResourceURLsOfTypeForLocalization
	
一个C的示例：

	CFURLRef    seagullURL;	// Look for a resource in the main bundle by name and type.	seagullURL = CFBundleCopyResourceURL( mainBundle,    	            CFSTR("Seagull"),        	        CFSTR("jpg"),            	    NULL );
	CFArrayRef  birdURLs;	// Find all of the JPEG images in a given directory.	birdURLs = CFBundleCopyResourceURLsOfType( mainBundle,    	            CFSTR("jpg"),        	        CFSTR("BirdImages") );

获取bundle的Info.plist数据

每个bundle必有一个Info.plist文件，这是一个XML的文本文件，包含了特定的键值对，DTD[在此][Info.plist DTD]，具体配置参见《Runtime Configuration Guidelines》。里面的值也可以用2种API来获取。

在NSBundle类里有`objectForInfoDictionaryKey:`和`infoDictionary`2种方法来获取，前者会给出所查键所对应的l10n后的值；
而后者只以NSDictionary的形式给出Info.plist的所有键值对，再去查询相应的键，获取的值是raw值，也即没有l10n的

同样，C的API也有对应的2种方式，函数名分别为`CFBundleGetValueForInfoDictionaryKey`和`CFBundleGetInfoDictionary`，一个示例如下：

	// This is the ‘vers’ resource style value for 1.0.0	#define kMyBundleVersion1 0x01008000	UInt32  bundleVersion;	// Look for the bundle’s version number.	bundleVersion = CFBundleGetVersionNumber( mainBundle );	// Check the bundle version for compatibility with the app.	if (bundleVersion < kMyBundleVersion1)    	return (kErrorFatalBundleTooOld);
一个获取没有l10n的字符串的示例：
	CFDictionaryRef bundleInfoDict;    CFStringRef     myPropertyString;  	
	// Get an instance of the non-localized keys.      bundleInfoDict = CFBundleGetInfoDictionary( myBundle );	// If we succeeded, look for our property.	if ( bundleInfoDict != NULL ) {		myPropertyString = CFDictionaryGetValue( bundleInfoDict,                      CFSTR("MyPropertyKey") );	}  
后文涉及可执行文件的装载与卸载，bundle的卸载，文档包等，在iOS里用不到，这里且按下不表。	
[l10n definition]: http://en.wikipedia.org/wiki/Internationalization_and_localization
[Info.plist DTD]: http://www.apple.com/DTDs/PropertyList-1.0.dtd