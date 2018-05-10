---
layout: post
title: kombu 入门教程(上)
---

## 前言

本文是[kombu框架](https://github.com/celery/kombu)(一个基于python的消息队列框架)的入门教程. 文章会简单介绍消息队列中的基本概念和kombu框架的简单使用方法.

## Hello World

我们来了解一下, 一个简单的消息队列里面, 应该包含以下的对象.

1. **Producer** -- 负责消息的发送
1. **Consumer** -- 负责消息的接收
1. **Queue** -- 负责消息的传输
1. **Exchange** -- 负责消息的分发

在我们的**Hello World**中, 我们只看到了创建**Queue**的过程, 其实一个完整消息队列应用程序是会包含上面所说的所有实体的.

但是kombu很贴心的为我们封装了一个便于使用的**SimpleQueue**类, 让我们能够快速地开发.

下面我们就围绕着这个**SimpleQueue**类来进行简单的试验:

    import kombu
    conn = kombu.Connection('redis://localhost')

导入kombu模块, 并且创建一个连接, 连接到本地的Redis服务上(本文的介绍都是基于Redis作为消息队列后端)

    def message_process(body, message):
        print "[x] %s" body
        message.ack()
    queue = conn.SimpleQueue('demo')
    queue.consumer.callbacks = [message_process]

这里定义了一个**message_process**函数, 用于处理消息.
需要注意的是, 当消息处理完之后, 要发送调用message对象的ack方法来确保消息已经被正确处理.
如果没有确认的话, 那么消息依然放置在队列当中, 当作没有被处理.

之后创建了一个消息队列, 命名为**demo**. 并且给队列的consumer绑定了一个消息处理函数.

以上就是关于消息接收的一些过程, 下面继续介绍一下消息发送的过程.

    queue.put('hello,world')

创建消息队列的过程与消息接收的相同, 要注意的就是不需要为消息队列指定consumer的处理函数.
直接调用queue的put方法就可以发送消息了.

完整的代码如下:

_receiver.py_

    import kombu

    def message_process(body, message):
        print "[x] %s" % body
        message.ack()

    conn = kombu.Connection('redis://localhost')
    queue = conn.SimpleQueue('demo')
    queue.consumer.callbacks = [message_process]

    queue.get()

_sender.py_

    import kombu

    conn = kombu.Connection('redis://localhost')
    queue = conn.SimpleQueue('demo')
    queue.put('hello, world')

启动receiver:

    $ python receiver.py

发送消息:

    $ python sender.py

之后会在receiver的终端上看见:

    $ python receiver.py
    [x] hello world

简单的消息队列应用就这样.

感谢阅读~
