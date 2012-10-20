---
layout: post
title: "Hello World"
description: ""
category: Blogging
tags: [Jekyll, notes]
---

This is a 'Hello World' for Jekyll, MarkDown stuff.

# 配置github和Jekyll环境来写博客

在可以用Jekyll写博客，并让github来host博客前，先配环境。

## 配置github

1. 当然，先得有github账号
2. 之后，在github的管理界面里创建一个USERNAME.github.com的仓库，其中，USERNAME为github账号名。

## 安装Jekyll

Jekyll依赖Ruby环境，幸运的是，Mac里已经安装好了，安装Jekyll只需一条命令

	$>gem install Jekyll

有了Jekyll就可以在本地预览博客效果

	# remember to change USERNAME to your GitHub username.
	$ cd USERNAME.github.com 
	$ Jekyll --server
	 
Jekyll用的是4000端口：http://localhost:4000/

## 添加博客

	$ rake post title="Hello World"

rake需要有rakefile才能进行博客的生成，不然会报错:

	No Rakefile found (looking for: rakefile, Rakefile, rakefile.rb, Rakefile.rb)


也就是说，上述的文件名中任意一个都可以做rakefile。

rakefile是一个ruby脚本，我推测是，rake根据它为模板，生成一个post。

## metadata

用rake生成的博客会有在这样的头:


	---
	layout: post
	title: "Hello world"
	description: ""
	category: 
	tags: []
	---
	{% include JB/setup %}

1. description里可以定义keywords来做SEO
2. category只能有一个，标明博客所属的类别，会在category页里显示
3. tags可以有多个，用`,`隔开，用`[`和`]`括起
