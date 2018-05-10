---
layout: post
title: 让你的Docker运行在Open vSwitch上
---

docker创建的container只能运行在docker创建的docker0上(linux bridge).
所以就有了[pipework](https://github.com/PeerXu/pipework)这个项目.

## pipework范例


    # ovs-vsctl add-br br0
    # ip link set br0 up
    # ip addr add 10.0.0.1/24 dev br0
    # pipework br0 $(docker run -d mysql /usr/sbin/mysqld_safe) 10.0.0.200/24

以上几行脚本的意思是:

1. 创建一个Open vSwitch的桥br0
1. 设置br0的ip地址为10.0.0.1
1. 在br0上创建一个docker并绑定IP地址

## pipework工作原理

创建一对[veth](http://wangcong.org/blog/archives/1704), 将一头绑定到docker内, 另外一头绑定到Open vSwitch.
然后通过_iproute_ 绑定IP地址等等操作.

然后, 就没有然后了.
