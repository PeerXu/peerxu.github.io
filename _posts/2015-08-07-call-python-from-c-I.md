---
layout: post
title: 在C中调用Python程序(I)
---

## 废话

通常, 我们会在终端上启动Python程序.

    $ python -c 'print "Hello, World"'
    Hello, World
    $ python hello.py
    Hello, World
    
但是我们今天来介绍一下, 怎么在C语言中调用Python程序.

<del>我们当然不是在C中调用`system`这么low的方法啦</del>

## 文档导读

首先, 先把我们的[官方文档](https://docs.python.org/2/c-api/index.html)祭出来. 让大家膜拜一下.

我们先来个文档导读.

[Introduction](https://docs.python.org/2/c-api/intro.html)是介绍Python的C语言API接口的. 大概有什么接口啦, 要怎么调用啦, 需要define什么头文件啦, etc.

[he Very High Level Layer](https://docs.python.org/2/c-api/veryhigh.html)是我们今天的重头戏, 介绍Python的执行表达式API.

[Concrete Objects Layer](https://docs.python.org/2/c-api/concrete.html)是介绍每种Python原生对象C API接口, 基本上就是怎么创建对象, 修改对象, 读取对象, etc.

其他文档重要吗? 当然重要啊, 回去读10遍回来比看我这篇破文章靠谱多了.

## 当然我们是从最简单的开始

我们先来看看这个介绍[Embedding Python](https://docs.python.org/2/c-api/intro.html#embedding-python).

里面详细介绍了Python初始化环境的流程, 我们先看一个最简单的环境.

    // file: py.c
    
    // 导入Python的头文件, 这里我采用的是Python2.7版本
    #include <python2.7/Python.h>
    
    int main() {
      // 初始化Python环境
      Py_Initialize();
      // 完了
      return 0;
    }
    
写完上面惊为天人的代码以后, 我们就来编译它.

    $ gcc -lpython2.7 -o py py.c
    
`-lpython2.7`意思是告诉gcc编译器采用`python2.7`的代码库.

注意: 在编译之前请安装相应平台的python开发库.

例如: ubuntu就安装`python-dev`, centos就安装`python-devel`, osx貌似直接`brew install python`就可以了.

编译完成后, 会生成一个`py`的可执行文件.

     $ ./py  // 当然什么都没有显示啊笨蛋!
     
## 别bb了, 快入正题

我们上面的程序写了那么长, 然而并没有什么卵用.

我们下面来点实际点的, 调用python的print来输出点什么东西.

    // file: py.c
    
    // 导入Python的头文件, 这里我采用的是Python2.7版本
    #include <python2.7/Python.h>
    
    int main() {
      // 初始化Python环境
      Py_Initialize();
      
      // 执行Python代码, 调用Python的print
      PyRun_SimpleString("print \"Hello, C! -- from Python\"");
      
      // 清理Python环境
      Py_Finalize();
      return 0;
    }
    
再编译后执行试试?

    $ gcc -lpython2.7 -o py py.c
    $ ./py
    Hello, C! -- from Python

## bb完要总结点东西吧

现在我们已经能够在C里面通过Python API来创建Python运行环境和执行Python代码了.

是不是如此的激动人心?

但是还有一些问题我们是需要解决的, 例如, 我怎么获取Python的变量和返回值, 我怎么调用Python的函数, etc.

那句话怎么说来着的?

欲知后事如何, 请听下回分解!

enjoy it~
