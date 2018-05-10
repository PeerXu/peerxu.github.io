---
layout: post
title: 从主机复制文件到Docker的几种方法
---

[Docker](https://www.docker.io)是个Linux Container管理软件.

今天我们来讲解一下从主机复制文件到Docker的几种方法.

在分享之前, 我们看看Docker社区对这个[问题](https://github.com/dotcloud/docker/issues/905)的需求是有多么强(ju)烈(jin).

下面开始今天高(tu)大(yuan)上(fei)的分享.


### 1. 通过Build Docker Image添加文件

Docker Image是通过Dockerfile来创建的. 具体的创建过程可以参考[这里](https://www.docker.io/learn/dockerfile/).

我们可以在编写Dockerfile的时候, 将需要的文件通过 `ADD` 关键字添加文件到Docker Image里面.

    FROM 3scale/openresty

    ## add your supervisor openresty config
    ADD openresty.conf /etc/supervisor/conf.d/

    # Add your app
    ADD . /var/www

    CMD ["supervisor"]

引用自 [3scale/openresty](https://index.docker.io/u/3scale/openresty/)

这个Dockerfile中的`ADD` 关键字是将本机添加到Docker Image中的`/var/www` 文件夹中.

### 2. 通过docker run命令的-v/--volume参数

假设我们需要将本机的/data 目录分享到Docker的/mnt 目录下, 我们可以通过这样的命令:

    $ touch /data/bilibala
    $ docker run -v /data:/mnt -i -t ubuntu bash
    root@c039a83c35d0:/# ls /mnt
    bilibala

这个命令可以在启动container中绑定文件夹.

### 3. 通过API绑定目录

其实这个方法本质上跟2是一样的, 但是唯一不同的就是, API将`docker run` 这个命令分成两步了, 分别是: `create_container` 和 `start`
在`create_container` 中, 通过`volumes` 参数定义需要挂载的目录.
在`start` 中, `binds` 参数绑定.

下面是一个简单的example:

    #!/usr/bin/env python2.7
    import docker
    c = docker.Client()

    container = c.create_container('ubunt',
                                   command='bash', volumes=['/mnt'],
                                   tty=True, stdin_open=True)

    c.start(container['Id'], binds={'/data':'/mnt'})

这里就创建了一个挂载了`/data`目录的container.


### 4. 通过环境变量传递文件

这个是我自己发明的小技巧, 因为在利用`volumes` 参数的时候, 发现docker有些不稳定. 经常无法删除. 所以就通过创建的时候通过环境变量传输文件.

先将文件通过`base64`编码, 然后通过`create_container` 方法的 `environment`参数传递变量到container中, 在container中再解码放入到合适的路径下即可.

### 5. 总结

总的来说, 有三种不同的方式, 将host中的文件传递到container.

分别是:

1. 创建Image时, 添加文件到Image
1. 创建Container时, 通过volumes参数传递文件
1. 创建Container时, 通过environment参数传递文件
