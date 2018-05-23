---
layout: post
title: 通过IP组播发送WIFI配置到未联网设备上
---

## 前言

物联网设备, 顾名思义, 是需要设备连接到网络当中去. 但是, 怎么使设备连接到网络当中去, 是一件极其复杂的事情.

有输入设备的设备, 比较好办. 例如可以通过摄像头扫描二维码, 通过麦克风监听带配置信息的音频, 或者直接通过蓝牙发送配置信息都可以.

但是, 以上的设备都是带有某种输入设备的, 但是很多物联网设备, 除了WIFI网卡就没有别的联网设备了.

有人会说, 很简单啊, 设备设置成AP模式, 手机连接过去, OJBK.

这当然也是可以的, 但是, 需要手机连接进去设备的AP啊, 这这这, 这手机就会断网啊, 你能忍? 反正我不能忍了.

那么有没有办法, 手机已经连接WIFI的情况下, 把WIFI配置发送给设备呢?

答案当然是肯定的, 有同事在微博中发现, 这个[trick](https://weibo.com/1829606830/GffX2osBk?type=comment#_rnd1527069968397).

下面就来分析一下这个trick的原理.

## IP组播(IP Multicast)

[IP组播](https://en.wikipedia.org/wiki/IP_multicast)是TCP/IP协议中, 由IP层提供的特性. 目的是提供一对多, 多对多的基于IP网络的实时通信特性.

IP组播有特定的地址段, 从 `224.0.0.0` 到 `239.255.255.255`.

当然, IP组播有很多技术特性. 我们今天就使用其中的一个[IP多播地址到MAC地址映射](http://www.cloudstructured.com/network/multicast/multicast-part-ii-ip-multicast/)来实现我们的目的.

关于这个特性, 有个有趣的[小插曲](https://showipprotocols-tw.blogspot.sg/2009/05/ip-multicast-mac-address-23.html).

我们继续看这个映射到底是怎么回事.

```
                     224  .   128  .   0    .   0

                  11100000 10000000 00000000 00000000    IP地址
                            ||||||| |||||||| ||||||||
                            VVVVVVV VVVVVVVV VVVVVVVV
00000001 00000000 01011110 00000000 00000000 00000000    MAC地址

   01   -   00   -   5E   -   00   -   00   -   00

                          图1

```

我们先看看上面的*图1*. 箭头指向的, 就是会从IP地址映射成MAC地址的位.

所以, 如果我们现在有一个 `224.232.55.231`的IP地址, 会映射成 `01:00:5E:68:37:E7`

```
                     224  .   232  .   55   .   231

                  11100000 11101000 00110111 11100111    IP地址
                            ||||||| |||||||| ||||||||
                            VVVVVVV VVVVVVVV VVVVVVVV
00000001 00000000 01011110 01101000 00110111 11100111    MAC地址

   01   -   00   -   5E   -   68   -   37   -   E7

                          图2
```

当我们熟练掌握了这个技巧以后, 我们就有一种传输数据的手段.

虽然只能一次传输23bits. 但是, 这也是质的飞跃啊.

但是说了这么多, 到底这个特性有什么用啊?

别急别急, 让我慢慢说.

## 加密的80211和未加密的MAC地址

通常情况下, 我们连接的WIFI都是加密的, 那么我们连接到WIFI的设备发送的数据, 也是经过加密的. 我们可以捕获到加密的数据包, 但是无法解密.

所以就没有办法用正常的手段发送数据给物联网设备了.

(这不废话吗? 如果可以我还写这么一大段po文做甚?

但是, 凡事总有但是.

我们会发现, 捕捉到的数据包, 802.11层的MAC地址, 是以明文的形式发送的. 这就给了我们足够的想象空间了.

我们可以采用刚刚的*IP多播: IP地址映射MAC地址*特性, 来传输消息, 就是我们需要传递的WIFI配置信息.

## POC

我们先在设备上打开一个监听程序(tcpdump).

这里需要把网卡设置成监听模式, 因为我采用的是 `Raspberry Pi Zero W`, 所以需要用[nexmon](https://github.com/seemoo-lab/nexmon).

暂时我们先假设读者已经知道怎么启用监听模式.

```
# Pi Zero W Terminal

$ sudo tcpdump -i mon0 ether dst 01:00:5E:68:37:E7
```

```
# Sender Terminal

$ cat sender.go
package main

import (
	"net"
)

func main() {
	addr, _ := net.ResolveUDPAddr("udp", "224.232.55.231:240")
	cli, _ := net.DialUDP("udp", nil, addr)
	cli.Write(nil)
}
$ go run sender.go
```

```
# Pi Zero W Terminal
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on mon0, link-type IEEE802_11_RADIO (802.11 plus radiotap header), capture size 262144 bytes
15:12:58.633212 44578627us tsft 2427 MHz 0dBm signal Data IV:5e9e Pad 20 KeyID 0
15:12:58.703203 44652100us tsft 2427 MHz -54dBm signal Data IV:d7b1 Pad 20 KeyID 1

2 packets captured
3 packets received by filter
0 packets dropped by kernel
```

## 总结

剩下的事情, 就是如何把WIFI配置塞进去那23bits的问题了.

那么这个问题, 我们下集再说~

enjoy it~
