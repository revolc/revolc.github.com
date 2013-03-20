---
layout: post
title: "使用git flow进行源码管理"
description: ""
category: SoftwareEngineer/Practice
tags: [Git, git flow, source code control, commands, Continuous Integration]
---
{% include JB/setup %}

以前的团队里使用[`git flow`][1]的方式进行开发，DVCS的方式，每个人各自开发很happy，最后轻松一合代码，一出build，一切都这么顺畅！但是到了新团队，因为条件有限，却要使用古老的SVN来进行源码管理，经常有合并的冲突，还不好解决，Xcode里有些文件又加不上，很蛋疼。

用过好的工具，再用糟的工具真的各种不适，正所谓“曾经沧海难为水, 除却巫山不是云”。决定在小团队里试用，最后再向高层推广`git flow`这种开发过程。

这里先整理一些有用的资源，对你也许也是个好的参考。

### 1. 概念

git flow模型由Vincent Driessen提出，[这里][1]是官方的日志。

### 2. 工具

首先，git flow是git的插件，要使用git flow这一流程，先要有git，git是Linus大神为了更好的开发Linux而做的版本控制系统，官网在[这里][2]。发现没，真的大神为了做一件事，往往会产生很强大的副产物，就像knuth为了写TAOCP而产生了TeX一样。不过因为GFW，官网始终访问不了，Mac用户如果装了Xcode，一般就会带着git，倒是少了一个负担。

其次，git flow的命令行需要有git-completion，其源码放在[github仓库上][3]了，安装教程参见[这里][4]。

最后，就是安装git flow的completion，[仓库在这里][5]

### 3. CheatSheet

如果看了上面这些还是不甚明白的话，网上有个开发都整理出的cheatsheet的东西很有价值，可以[参考下][6]。

[1]: http://nvie.com/posts/a-successful-git-branching-model/ "Official site"

[2]: git-scm.com/	"git official site"

[3]: https://github.com/git/git/blob/master/contrib/completion/git-completion.bash		"git completion source"

[4]: https://github.com/bobthecow/git-flow-completion/wiki/Install-Bash-git-completion	"git-completion install guide"

[5]: https://github.com/bobthecow/git-flow-completion	"git flow completion repo"


[6]: http://danielkummer.github.com/git-flow-cheatsheet/ "cheatsheet"