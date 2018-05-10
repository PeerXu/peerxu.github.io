---
layout: post
title: Git Patch简单教程
---

今天我们来学习一下关于git怎么创建与应用patch.

### 预备资料

[patch](http://zh.wikipedia.org/zh-cn/Patch)是用来对现有的源代码进行更新的一种手段.

patch可以通过diff, git, svn, hg等工具生成.

diff与git产成的patch文件有点差异(下面会简单的介绍patch文档的结构).

下面我们就来介绍怎么创建一个patch.

### 前戏

首先, 你得有一个好环(ji)境(you), 才能开展patch的工作.

所以我在这里简单的创(ji)建(you)一个环境给大家使用. :P

    $ mkdir Hello
    $ cd hello
    $ git init
    $ cat << EOF > hello.py
    #!/usr/bin/env python2.7
    print "hello, world"
    EOF
    $ git add .
    $ git commit -m "initial commit" -a

至此, 大家已经有一个可以使用的好环(ji)境(you).(怎么感觉节奏不对)

### 动作片播放ing...

好了, 我们已经创建好了一个可以用的基友, 那么就开始干吧!

    $ git checkout -b my-feature
    $ vi hello.py
    $ cat hello.py
    #!/usr/bin/env python2.7
    def hello(name):
        print "hello, %s" % name

    hello("world")
    $ git commit -m "add feature" -a
    $ git format-patch master
    $ ls
    0001-add-feature.patch  hello.py

下面来为大家讲解一下上面的_动作_要领.

首先我们很大众的开局, 先创建一个my-feature.

然后啪啪啪地编辑了一下文档, 添加了一个相当犀利的特性.

然后按照正常套路去commit咱们牛逼的特性.

当全世界都以为我们要merge到master的时候, 我只能说你们, 图样图森普.

我们的目标是没有蛀牙. 咳咳, 是创建patch.

所以我们最后使用 ```git format-patch master``` 创建一个patch.

好了, 通常动作片的中场都是要休息一下的.

### 动作片继续播放ing...

我们既然已经生成了patch文件, 咱们就看看这文件到底是什么样子的.

    From a9706e72ba4976f69d1c3ca5e93ab9a00757c9fd Mon Sep 17 00:00:00 2001
    From: Peer Xu <pppeerxu@gmail.com>
    Date: Thu, 21 Nov 2013 23:37:41 +0800
    Subject: [PATCH] add feature

    ---
     hello.py | 5 ++++-
     1 file changed, 4 insertions(+), 1 deletion(-)

    diff --git a/hello.py b/hello.py
    index 6635083..c314e13 100644
    --- a/hello.py
    +++ b/hello.py
    @@ -1,2 +1,5 @@
     #!/usr/bin/env python2.7
    -print "hello,world"
    +def hello(name):
    +    print "hello, %s" % name
    +
    +hello("world")
    --
    1.8.3.2

以我阅片无数的经验, 这是一封邮件.

嗯, 剩下的就留给各位看官自己去YY吧.

### 动作片通常都是分开三部播放的...

这里, 我传授大家怎么将git生成的patch应用到项目中去.

还是咱们的好环境.

    $ cp 0001-add-feature.patch /dev/shm/
    $ rm 0001-add-feature.patch # 删除生成的patch, 保证环境干净 (这是你洁癖的证据!)
    $ git checkout master
    $ git am /dev/shm/0001-add-feature.patch
    $ cat hello.py
    #!/usr/bin/env python2.7
    def hello(name):
        print "hello, %s" % name

    hello("world")
    $ git log # 这里可以看到应用patch的日志

我们先将patch复制到另外一个地方(假装是从很远的地方下载下来的)

然后checkout到master(假装在别的环境中...)

然后使用 ```git am``` 这命令使patch应用到环境中去.

然后, 就没有然后了...

### 动作片再好, 也不能经常看...
