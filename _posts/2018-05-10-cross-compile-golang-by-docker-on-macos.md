---
layout: post
title: 在macOS下通过Docker交叉编译GO程序
---

## 前言

*TL;DR*

1. 普通情况下, 只需要设置环境变量 `GOOS`和 `GOARCH`即可编译成目标平台的二进制文件.
2. 特殊情况下(需要采用Golang的 `plugin`包或其他情况), 因为macOS安装 `arm`平台的编译工具链不方便, 所以在Docker里面编译.

Golang对交叉编译的支持是很棒的, 通常情况下, 我们只需要配置 `GOOS`和 `GOARCH`这两个环境变量, 就能够正常交叉编译了.

```
$ cat main.go
package main
import "fmt"
func main() { fmt.Println("hello, world") }
$ GOOS=linux GOARCH=arm GOARM=6 go build -o hello main.go
$ file hello
hello: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, with debug_info, not stripped
```

把上面的`hello`程序复制到Raspberry Pi Zero W(下文都是采用这个平台作为测试)上, 可以运行.

接下来提出这篇文章需要解决的问题.

Golang 1.8中, 添加了一个新的包: [plugin](https://golang.org/pkg/plugin/). 支持Golang代码以插件形式执行.

下面是演示代码.

```
// file: plugin/plugin.go
package main

import "fmt"

func Hello() { fmt.Println("hello, world") }
```

```
// file: main.go
package main

import "plugin"

func main() {
        p, err := plugin.Open("plugin.so")
        if err != nil {
                panic(err)
        }

        HelloFn, err := p.Lookup("Hello")
        if err != nil {
                panic(err)
        }

        HelloFn.(func())()  // prints "hello, world"
}
```

```
$ go build -o plugin.so -buildmode=plugin plugin/plugin.go
$ go build -o hello main.go
$ ./hello
hello, world
```

以上是编译成本地可执行二进制文件的流程.

但是, 如果采用交叉编译, 就会出现以下错误.

```
$ GOOS=linux GOARCH=arm GOARM=6 go build -o plugin.so -buildmode=plugin plugin/plugin.go
# command-line-arguments
loadinternal: cannot find runtime/cgo
/usr/local/opt/go/libexec/pkg/tool/darwin_amd64/link: running clang failed: exit status 1
clang: error: invalid linker name in argument '-fuse-ld=gold'
```

## 解决问题

前言有点多, 后面就长话短说.

因为没有在网上找到太多的解决方案, 决定在linux环境下编译试试. 得到如下结果.

```
$ GOOS=linux GOARCH=arm GOARM=6 go build -o plugin.so -buildmode=plugin plugin/plugin.go
# command-line-arguments
loadinternal: cannot find runtime/cgo
/usr/local/go/pkg/tool/linux_amd64/link: running gcc failed: exit status 1
gcc: error: unrecognized command line option '-marm'; did you mean '-mabm'?
```

然后通过搜索得到[结果](https://github.com/golang/go/issues/16801)

需要指定 `CC`环境变量, 指向 `arm-linux-gnueabihf-gcc`.

```
$ CC=arm-linux-gnueabihf-gcc GOOS=linux GOARCH=arm GOARM=6 go build -o hello main.go
$ CC=arm-linux-gnueabihf-gcc GOOS=linux GOARCH=arm GOARM=6 go build -o plugin.so -buildmode=plugin plugin/plugin.go
$ file hello plugin.so
hello:     ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 2.6.32, not stripped
plugin.so: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, not stripped
```
编译成功

## 总结

如果觉得上面都太复杂了的话, 可以试试这个[docker镜像](https://hub.docker.com/r/e3rp4y/rpi-cross-compiler-platform/).

    $ docker run -it --rm e3rp4y/rpi-cross-compiler-platform /bin/bash
    
已经配置好 `GOOS`, `GOARCH`, `GOARM`, `CC`和 `CGO_ENABLED`几个环境变量.

## 备注

1. RPi有自己的[编译工具](git://github.com/raspberrypi/tools.git)
