---
layout: post
title: 一次无意识的DNS劫持
---

0x00 前言

发现DNS劫持的缘由在于部署Dnsmasq的过程中, 同网段内的Windows主机的DNS地址被改变成Dnsmasq服务所在的主机IP地址. 所有就有了这篇分析报告.

0x01 现象

在某个子网下部署dnsmasq服务, 过了一会儿, 其他用Windows的同事就大喊: XXX, 为什么我不能上网!

经过一轮排查, 发现是使用DHCP获取IP地址的Windows用户, DNS地址被改成了部署Dnsmasq服务的服务器IP地址.

以为是Dnsmasq的BUG(其实还真算Dnsmasq的BUG). 那么就添加了```--port=0```的启动参数, 表示不提供DNS服务, 然后再添加```--dhcp-option=6,114.114.114.114```指定DNS服务器IP地址(离真相已经很近了).

但是因为公司默认的DHCP并非```114.114.114.114```, 而是某个不知名的DNS服务器地址. 所以又有人大喊为什么我的DNS变成了```114.114.114.114```.

到现在我才意识到, 问题的严重性.

我成功地将DNS劫持了.

别问我劫持了DNS能够做什么.

0x02 分析

发生劫持事件后, 我添加Dnsmasq启动参数```--no-daemon```, 表示调试模式.

然后启动通过```tcpdump -n -i eth0 -vvv port 68```来捕捉DHCP请求.

经过观察, 发现Windows DHCP 客户端会定时广播```DHCPINFORM```, 而Dnsmasq会响应并非在```dhcp-hostsfile```列表里面的主机. 响应的结果就是发送```DHCPACK```. 而```DHCPACK```上又带上了```Domain Name Server Option```字段. 

所以, Windows DHCP Client会将Dnsmasq返回的DHCPACK包中带上的DNS地址设置为自己的DNS地址.

0x03 结论

你还敢用Windows的DHCP吗?

当你无法保证所在网络绝对安全的情况下, 请手动设置DNS地址.
