---
layout: post
title: OpenvSwitch gre隧道配置
---

本文是讲述如何在OpenvSwitch交换机上配置gre隧道的. 不废话, 马上开始吧.

本文要求读者对Openvswitch, Mininet, Openflow, Pox有一定了解.

实验环境如下:

1. Host * 2
    * 主机上安装了OpenvSwitch与mininet
    * 分别命名2台主机为H0和H1(程序员都是从0开始计数的)
    * H0: ip地址: 192.168.122.150 eth0绑定到br1上 (绑定过程参考实验过程)
    * H1: ip地址: 192.168.122.151 eth0绑定到br1上
1. Openflow 控制器主机 (可以是上面其中一台)
    * 命名为C0
    * 选用pox控制器

实验目的: 通过构建gre隧道, 让2个隔离的datapath(fake bridge)虚拟机通信

实验简介:

1. 在C0上启动pox控制器
1. 在H0上创建br1, 将eth0绑定到br1上, 完成相关配置.
1. 在H0上以对应参数启动mininet, 并且配置dp0, 与设置mininet的虚拟机h2, h3.
1. 为H0上创建gre通道
1. H1同理
1. 在H0的miminet控制台中, 使用h2 pingH1上的h2.

实验过程:

    # 启动pox控制器
    C0 $ ./pox.py forwarding.l2_learning

    # 配置H0
    # 配置br1
    H0 # ovs-vsctl add-br br1
    H0 # ovs-vsctl add-port br1 eth0
    H0 # ip addr add 192.168.122.150/24 dev br1
    H0 # ip route add default via 192.168.122.1 dev br1
    # 启动mininet, 选择远程控制器, 连接到192.168.122.1的控制器上.
    H0 # mn --switch=ovsk --controller=remote --ip=192.168.122.1
    # 添加gre隧道, 连接到192.168.122.151
    H0 # ovs-vsctl add-port dp0 gre0 -- set interface gre0 type=gre options:remote_ip=192.168.122.151

    # 配置H1
    # 配置br1
    H1 # ovs-vsctl add-br br1
    H1 # ovs-vsctl add-port br1 eth0
    H1 # ip addr add 192.168.122.151/24 dev br1
    H # ip route add default via 192.168.122.1 dev br1
    # 启动mininet, 选择远程控制器, 连接到192.168.122.1的控制器上.
    H1 # mn --switch=ovsk --controller=remote --ip=192.168.122.1
    # 添加gre隧道, 连接到192.168.122.150
    H1 # ovs-vsctl add-port dp0 gre0 -- set interface gre0 type=gre options:remote_ip=192.168.122.150
    # H1的mininet控制台
    mininet> h2 ip addr replace 10.0.0.4/24 dev h2-eth0
    mininet> h3 ip addr replace 10.0.0.5/24 dev h3-eth0

    # H0的mininet控制台
    mininet> h2 ping -c 2 10.0.0.4
    PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
    64 bytes from 10.0.0.4: icmp_req=1 ttl=64 time=54.5 ms
    64 bytes from 10.0.0.4: icmp_req=2 ttl=64 time=0.499 ms
    --- 10.0.0.4 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1002ms
    rtt min/avg/max/mdev = 0.499/27.526/54.554/27.028 ms

参考文章: <http://networkstatic.net/open-vswitch-gre-tunnel-configuration/>

enjoy it.
