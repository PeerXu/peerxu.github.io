---
layout: post
title: 在C中调用Python程序(II)
---

## 承上启下的段落

[上一节](http://blog.e3rp4y.me/2015/08/07/call-python-from-c-I/)我们已经介绍了, 怎么在C语言中, 采用Python API调用Python程序.

上一节采用的是`PyRun_SimpleString`这个API函数. 

这个函数的优点就是方便, 不用初始化环境变量就能够正常运行了.

缺点也是相当明显, 便利带来的就是灵活性不够高. 最简单的C-Python交互都无法正常完成.

## 摊开你的双手, 来迎接新挑战吧!

首先, 我们还是得先看看文档.

[The Very High Level Layer](https://docs.python.org/2/c-api/veryhigh.html)告诉我们, 除了`PyRun_SimpleString`之外, 还有`PyRun_String`, `PyRun_File`, etc.

我们今天就来介绍一下`PyRun_String`的用法.

这个是`PyRun_String`的函数签名:

    PyObject*             // PyRun_String的返回值类型
    PyRun_String(         // 函数名
      const char *str,    // 执行的Python代码
      int start,          // 载入类型, 分别有: Py_eval_input, Py_file_input, Py_single_input
      PyObject *globals,  // 全局对象
      PyObject *locals    // 局部对象
    )

上面我们可以知道, `PyRun_String`和`PyRun_SimpleString`是相似的功能, 都是运行一段Python代码.

`start`参数是指执行的这段代码, 到底是怎么导入的, 这里分了三种模式:

* [Py_eval_input](https://docs.python.org/2/c-api/veryhigh.html?highlight=py_single_input#c.Py_eval_input), 是执行隔离的表达式用的.
* [Py_file_input](https://docs.python.org/2/c-api/veryhigh.html?highlight=py_single_input#c.Py_file_input), 是最常用的模式, 表示表达式
* [Py_single_input](https://docs.python.org/2/c-api/veryhigh.html?highlight=py_single_input#c.Py_single_input), 这种模式, 是表示输入的代码是从交互接口输入的.

`globals`参数是表示Python的`globals`对象

`locals`参数是表示Python的`locals`对象

好了, 了解了`PyRun_String`函数的各个变量含义后, 我们就来写点代码吧.

## 代码解析

    // file: py.c
    
    #include <python2.7/Python.h>

    int main() {
      // 这里声明了三个环境变量和一个返回值.
      PyObject *pMainModule,   // __main__模块的指针对象, 等下会初始化为__main__模块.
          *pGlobalDict,        // globals对象指针
          *pLocalDict,         // locals对象指针
          *pResult;            // 用来获取返回值

      // 这是一段Python代码          
      char *cmd = ""
        "import math\n"
        "x = 3\n"
        "y = 2\n"
        "result = math.pow(x, y)";

      // 用来接受返回值
      double result;

      Py_Initialize();

      // 初始化__main__模块
      pMainModule = PyImport_AddModule("__main__");
      // 在__main__模块中获取globals对象
      pGlobalDict = PyModule_GetDict(pMainModule);
      // 初始化一个空的locals对象
      pLocalDict = PyDict_New();

      // !! 这里是整个程序的核心, 就在这里执行了cmd的Python程序
      PyRun_String(cmd, Py_file_input, pGlobalDict, pLocalDict);

      // 获取locals对象上面的result变量
      pResult = PyDict_GetItemString(pLocalDict, "result");
      // 讲PyFloat类型转换成C语言的double类型
      result = PyFloat_AsDouble(pResult);

      // 释放引用计数
      Py_DECREF(pResult);
      Py_DECREF(pLocalDict);
      Py_DECREF(pGlobalDict);

      // 打印计算得出来的值
      printf("result=%lf\n", result);

      Py_Finalize();

      return 0;
    }

编译一下看看结果?

    $ gcc -lpython2.7 -o py py.c
    $ ./py
    result=9.000000
    
## 总结点什么吧

好吧, 上面的注释应该写得够清楚了, 还有什么不明白的话, 就是我表达能力有问题了...

实在不行就回去多看看文档吧.

enjoy it~
