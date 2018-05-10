---
layout: post
title: 利用Docker跨平台编译Windows平台的qemu-ga.exe
---

## 介绍

今天, 我们介绍一下怎么在Linux环境下编译Windows的可执行文件.

首先, 我们要先熟悉一下[mingw](https://zh.wikipedia.org/wiki/MinGW). <del>基本上就是一个能够编译出windows可执行文件的gcc吧.</del>

其次, 我们用一个已经搭好mingw环境的docker容器来做今天的展示.

[centos-mingw-base](https://hub.docker.com/r/e3rp4y/centos-mingw-base/)是在[Centos](https://hub.docker.com/_/centos/)上构建的mingw编译环境.

[Dockerfile](https://hub.docker.com/r/e3rp4y/centos-mingw-base/~/dockerfile/) ([github](https://github.com/PeerXu/centos-mingw-base)) 可以自己添加组件.

## Glance

我们先运行一下`centos-mingw-base`容器看看.

    $ docker run --rm -t -i e3rp4y/centos-mingw-base /bin/bash
    
    [root@4ad795647521 /]# rpm -qa|grep mingw
    mingw-binutils-generic-2.25-1.el7.x86_64
    mingw64-filesystem-100-1.el7.noarch
    mingw32-crt-4.0.2-1.el7.noarch
    mingw64-crt-4.0.2-1.el7.noarch
    mingw64-headers-4.0.2-1.el7.noarch
    mingw32-cpp-4.9.1-3.el7.x86_64
    mingw32-gcc-4.9.1-3.el7.x86_64
    mingw64-binutils-2.25-1.el7.x86_64
    mingw32-pkg-config-0.28-2.el7.x86_64
    mingw32-zlib-1.2.8-2.el7.noarch
    mingw64-zlib-static-1.2.8-2.el7.noarch
    mingw32-win-iconv-0.0.6-1.el7.noarch
    mingw32-gcc-c++-4.9.1-3.el7.x86_64
    mingw64-gcc-c++-4.9.1-3.el7.x86_64
    mingw64-gettext-0.18.3.2-1.el7.noarch
    mingw64-glib2-2.40.0-3.el7.noarch
    mingw32-termcap-1.3.1-15.el7.noarch
    mingw32-glib2-2.40.0-3.el7.noarch
    mingw32-glib2-static-2.40.0-3.el7.noarch
    mingw-filesystem-base-100-1.el7.noarch
    mingw32-filesystem-100-1.el7.noarch
    mingw32-winpthreads-4.0.2-1.el7.noarch
    mingw64-winpthreads-4.0.2-1.el7.noarch
    mingw32-headers-4.0.2-1.el7.noarch
    mingw32-binutils-2.25-1.el7.x86_64
    mingw64-cpp-4.9.1-3.el7.x86_64
    mingw64-gcc-4.9.1-3.el7.x86_64
    mingw64-pkg-config-0.28-2.el7.x86_64
    mingw64-zlib-1.2.8-2.el7.noarch
    mingw32-zlib-static-1.2.8-2.el7.noarch
    mingw64-termcap-1.3.1-15.el7.noarch
    mingw64-libffi-3.0.13-4.el7.noarch
    mingw64-win-iconv-0.0.6-1.el7.noarch
    mingw64-gettext-static-0.18.3.2-1.el7.noarch
    mingw32-libffi-3.0.13-4.el7.noarch
    mingw32-gettext-0.18.3.2-1.el7.noarch
    mingw32-gettext-static-0.18.3.2-1.el7.noarch
    mingw64-glib2-static-2.40.0-3.el7.noarch
    [root@4ad795647521 /]#

我们能够直观的看见, 安装了mingw环境的相关组件.

下面我们先来简单的编译一个Windows平台的"Hello World".

    [root@4ad795647521 /]# cd /tmp
    [root@4ad795647521 tmp]# cat hello.c
    #include <stdio.h>
    int main(void) {
        printf("hello world\n");
        return 0;
    }
    [root@4ad795647521 tmp]# i686-w64-mingw32-gcc -o hello.exe hello.c
    [root@4ad795647521 tmp]# ls -lh hello.exe
    -rwxr-xr-x 1 root root 54K Aug 12 10:53 hello.exe
    [root@4ad795647521 tmp]# file hello.exe
    hello.exe: PE32 executable (console) Intel 80386, for MS Windows
    
这样我们就成功地编译出一个windows平台 32位系统的`hello.exe`.

备注: 如果想要编译64位的话, 将编译器换成`x86_64-w64-mingw32-gcc`即可.

## 编译qemu-ga.exe

下面, 我们来编译一下qemu-ga.exe

首先, 我们需要将源代码放置到容器里面.

我们可以采用docker的`--volume`参数.

    docker run --rm --volume=/Users/peer/tmp/qemu-2.3.0/:/tmp/qemu -t -i e3rp4y/centos-mingw-base
    
这里, 我们用`--volume`参数, 将本机的`/Users/peer/tmp/qemu-2.3.0/`目录映射到容器的`/tmp/qemu/`目录上.

然后我们可以开始编译了.

    [root@035df45036cc /]# cd /tmp/qemu
    [root@035df45036cc qemu]# mkdir build
    [root@035df45036cc qemu]# cd build
    [root@035df45036cc build]# ../confug --target-list=x86_64-softmmu --cross-prefix=i686-w64-mingw32-
    [root@035df45036cc build]# make V=1 VL_LDFLAGS=-Wl,--build-id qemu-ga.exe
    
拷贝走你的`qemu-ga.exe` happy去吧.

enjoy it.
