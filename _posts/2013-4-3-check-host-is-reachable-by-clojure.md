---
layout: post
title: InetAddress类中isReachable方法检测原理
---

今日, 有一同事问我, 为什么我用ping命令可以连通的一台主机, 用java的InetAddress类中的isReachable方法就测试失败呢?

当时我就感觉奇怪了, 难道isReachable用的不是ping?

刚刚测试了一下, 果然真不是用ping... 用的是 [echo][1]协议.

那么不可达的原因就不言而喻了, 因为目的主机没有启动echo服务.

附上clojure的测试代码:

	(.. java.net.InetAddress (getByName "192.168.1.1") (isReachable 1000))
	
吐槽: 如果是java的话要多长才能完成这个简单的测试呢?

[1]: http://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers "echo"
