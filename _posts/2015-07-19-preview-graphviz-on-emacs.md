---
layout: post
title: 在Emacs上预览graphviz图
---

[graphviz](http://www.graphviz.org/)是AT&T发明的一个结构化图片描述语言.

接下来就是介绍如何在Emacs上编辑Graphviz文件并且预览.

### 安装Graphviz

安装Graphviz, 这里安装的是带GUI的, 不用Emacs预览的使用也能用Graphviz.app 预览.

    brew install Caskroom/cask/graphviz
    
### 下载graphviz-dot-mode.el

[graphviz-dot-mode.el](https://raw.githubusercontent.com/ppareit/graphviz-dot-mode/master/graphviz-dot-mode.el)是Emacs的graphviz插件, 能够编译和预览graphviz图片.

    $ cd ~/.emacs.d/  // 放置在Emacs配置目录下, 你可以放在别的地方, 下面的地址跟着修改就可以了.
    $ wget https://raw.githubusercontent.com/ppareit/graphviz-dot-mode/master/graphviz-dot-mode.el
    
### 在init.el中载入插件

打开`init.el`, 在最后载入`graphviz-dot-mode.el`插件.


    $ cat ~/.emacs.d/init.el
    
    ...
    ;; Graphviz Mode
    ;; 如果之前保存插件地址更改了, 这里也需要作出相应的修改.
    (load-file "~/.emacs.d/graphviz-dot-mode.el")
    
添加后, 每次启动Emacs都会自动载入`graphviz-dot-mode.el`这个插件了.

如果不想重启Emacs的话, 可以在句末按下, `<C-x> <C-e>` 这样就会动态载入`graphviz-dot-mode.el`插件了.
    
### 品尝成功的果实

好了, 现在万事俱备, 只欠`graphviz`了.

我们随便编辑个`graphviz`文件.

    $ cat /tmp/test.dot
    
    digraph structs {
        node [shape=record];
        struct1 [label="<f0> left|<f1> middle|<f2> right"];
        struct2 [label="<f0> one|<f2> two"];
        struct3 [label="hello\nworld |{ b |{ c|<here> d|e}| f}| g | h"];
        struct1: f1 -> struct2:f0;
        struct1: f2 -> struct3:here;
    }
    
用Emacs打开, 然后按下 `<C-c c>`, 这样就成功编译`test.dot`文件.

之后只需要按下 `<C-c p>`, 就能够预览`test.dot`编译成png的文件.

enjoy it~

