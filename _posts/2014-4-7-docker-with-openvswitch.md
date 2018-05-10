---
layout: post
title: pipework -- docker网络增强工具
---

[pipework](https://github.com/jpetazzo/pipework)是[Docker](https://www.docker.io/)的一个网络增强功能插件.

Docker创建的时候, 默认是接入docker0(linux bridge)的. 所以只能单主机工作, 不能多台主机联动工作. 这时候, pipework应运而生.

我们先来介绍一下pipework的简单用法, 之后再深入探讨关于pipework与ovs的结合. 最后基于一个案例分析来结束.

### pipework入门

    # brctl addbr br0
    # ip link set dev br0 up
    # ip addr add 192.168.2.1/24 dev br0
    # MYSQL=$(docker run -d -p 3306:3306 -e MYSQL_PASS=admin tutum/mysql)
    # pipework br0 $MYSQL 192.168.2.100/24

1. 创建一个br0(linux bridge).
1. 配置IP地址给br0.
1. 启动一个mysql docker.
1. 使用```pipework```为docker添加IP地址.

经过这几个步骤后, 创建的docker就有一个设置指定的IP地址了.
pipework的具体工作原理就不在这里详细分析, 有兴趣的同学可以自己去读源代码.

### pipework进阶

上面入门介绍中, 我们使用pipework工具, 通过linux bridge的模式指定IP地址给docker容器.
其实, 除了linux bridge, pipework还支持现在流行的虚拟交换机(Open vSwitch).
下面简单的示范一下如何将docker与Open vSwitch(下称ovs)连通.

    # ovs-vsctl add-br ovsbr0
    # ip link set dev ovsbr0 up
    # ip addr add 192.168.3.1/24 dev ovsbr0
    # MYSQL=$(docker run -d -p 3306:3306 -e MYSQL_PASS=admin tutum/mysql)
    # pipework ovsbr0 $MYSQL 192.168.3.100/24

对于pipework来说, 底层使用什么虚拟网络设备是透明的.
pipework会智能地判断使用的是linux bridge还是ovs.
所以除了创建桥的方式不同, 其他并没有改变.

### pipework案例分析

下面就具体分析一个pipework的应用场景.

#### docker容器与VLAN

docker容器启动时, 默认是接入到docker0网桥上的. 这样就容易给大家造成一个假象, docker容器只能运行在单机的情况下.
*但是显示并非如此.*
我们可以通过pipework扩展docker的网络功能.

我们假设现在有2台服务器A和B, 交换机S.
现在我们同时在A,B上创建虚拟网桥ovsbr0. 并且在服务器A上, 将网桥启用和配置IP地址.
那么我们现在如果在A,B上,分别启动一个docker.
同时通过pipework将docker接入到ovsbr0上, 并且配置与A服务器上ovs相同网段的IP地址.
那么*两个docker互相之间就能够通信*了.
很神奇有木有!!!

但是万恶的需求是不断进化的(呵呵). 不会都让你们的docker运行在一个网络环境下, 有的应用是需要互相隔离的.
那么就有可能需要VLAN的划分. 这里多说点, 其实在现今的情况下, 除了划分VLAN还有其他的方式达到二层网络隔离的效果, 如VXLAN, NVGRE等技术.
但是我们现在还是只谈论VLAN隔离吧.

我们先在服务器A上做一下操作:

    # D1V10=$(docker run -i -t learn/ping /bin/bash)
    # pipework ovsbr0 -i d1v10 $D1V10 10.1.1.10/24
    # D2V10=$(docker run -i -t learn/ping /bin/bash)
    # pipework ovsbr0 -i d2v10 $D2V10 10.1.1.11/24
    # D3V20=$(docker run -i -t learn/ping /bin/bash)
    # pipework ovsbr0 -i d3v20 $D3V20 10.1.1.20/24

这里定义了3个Docker容器, 并且将其添加到ovsbr0这个桥上.
下面开始将不同的网卡设置vlan id.
我们将 D1V10 和 D2V10 设置vlan id为10. 将D3V20 设置vlan id为20.

我们可以通过```ovs-vsctl list-ports <BRIDGE>```找出桥设备下面的所有端口.
但是刚刚我们一口气连续创建了3个docker, 会看见3个对应的虚拟网卡.

然后我们需要做的是, 找出与Docker容器想对应的虚拟网卡.
刚刚在pipework执行时, 添加了参数```-i```, 意思是创建的虚拟网卡结尾对应的字符串.

比如:

    # pipework ovsbr0 -i d1v10 $D1V10 10.1.1.10/24

这命令执行后, ovsbr0设备会添加一个port, port的名字应该是```plxxxxd1v10```.
当中的xxxx是docker容器的pid.

这时我们就可以通过```ovs-vsctl set port <port> tag=<vlan id>```的方式设置vlan id了.

下面是范例:

    # ovs-vsctl list-ports ovsbr0
    pl23016d1v10
    pl23124d2v10
    pl23654d3v20
    # ovs-vsctl set port pl23016d1v10 tag=10
    # ovs-vsctl set port pl23124d2v10 tag=10
    # ovs-vsctl set port pl23654d3v20 tag=20

这样就完成了设置VLAN的步骤了.
下面我们通过简单的ping命令来验证一下vlan隔离的效果.

    # docker attach $D1V10
    root@f94dc85c44b1:/# ping 10.1.1.11  # reachable
    root@f94dc85c44b1:/# ping 10.1.1.20  # unreachable

#### 多服务器的vlan划分

可能有同学以为, docker只能在单机上工作(docker与docker之间通信只能在本机进行).
但是使用pipework增强插件后, 我们可以通过Linux Bridge或Open vSwitch来进行数据交换.
这样就可以不仅仅局限于Docker默认的隔离桥(原谅我的破翻译, 我真不知道怎么形容这货 --> docker0)

### 总结

pipework是个强大的docker网络管理工具. 具体的实现也比较简单. 可以配合ovs来实现强大的功能.
剩下就是发挥想象力的时间了...
