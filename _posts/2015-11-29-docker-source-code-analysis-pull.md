---
layout: post
title: Docker源代码分析 -- pull repository
---

本系列文章是从源代码入手分析Docker部分功能或者模块的具体实现

今天我们就先来看看在我们输入了`docker pull debian`之后到底发生了些什么事情

注意: 本文是基于docker v0.5.1版本分析

## 切入点

熟悉docker的同学都知道, 我们在运行一个container之前是需要下载container的image的.

我们可以通过以下的命令实现下载镜像的功能:

    docker pull debian
    
然后我们就可以运行基于`debian`的容器:

    docker run -i -t --rm --name debian debian /bin/bash
    
那么现在我们就看看, 到底在`pull`后面发生了多少不为人知的秘密.

我们知道, `commands.go`是命令行参数的入口:

## commands.go:

`CmdPull`作为客户端的入口, 调用`utils.ParseRepositoryTag`处理参数, 分离出`fromImage`和`tag`两个参数, 然后调用服务端的`/images/create`接口.

## api.go:

`api.go`作为服务端路由解析器, 将`/images/create`转发到`PostImagesCreate`函数上.

`PostImagesCreate`进行一些参数处理, 之后开始调用获取镜像函数`server.go/ImagePull`.

## server.go:

    func (srv *Server) ImagePull(localName string, tag string, out io.Writer, sf *utils.StreamFormatter, authConfig *auth.AuthConfig) error

这是`ImagePull`的方法声明. 

`ImagePull`方法的重要步骤:

1. 通过`registry.NewRegistry`创建`Registry`对象`r`.
1. 为获取镜像这个行为上锁(`srv.poolAdd`)并且在结束时释放锁(`srv.poolRemove`).
1. 通过`registry.ResolveRepositoryName`解析出仓库(`repository`)结构.
1. 通过`srv.pullRepository`获取仓库结构.
1. 如果失败了, 则通过`srv.pullImage`下载镜像.

下面我们继续来分析`srv.pullRepository`和`srv.pullImage`方法.

    func (srv *Server) pullRepository(r *registry.Registry, out io.Writer, localName, remoteName, askedTag, indexEp string, sf *utils.StreamFormatter) error

这是`pullRepository`的方法声明.

`pullRepository`方法的重要步骤:

1. 通过上面创建的`Registry`对象`r`的方法`GetRepositoryData`获取仓库的数据`repoData`.
1. 这时候通过`graph`对象的`UpdateChecksums`方法更新仓库上所有镜像的校验码.
1. 接下来是通过r的方法`GetRemoteTags`获取`repoData`的仓库列表上, 每个仓库的`tag`.
1. 接下来会判断一下在`CmdPull`解析到的`tag`是否在上面的`tags`列表上, 如果存在则下载该`tag`和相关联的`images`, 否则就下载所有`tag`.
1. 接下来就是通过`srv.pullImage`方法来下载由步骤4中指定的`tag`.

接下来我们分析一下`pullImage`函数

    func (srv *Server) pullImage(r *registry.Registry, out io.Writer, imgID, endpoint string, token []string, sf *utils.StreamFormatter) error

这是`pullImage`的函数声明

`pullImage`方法的重要步骤:

1. 通过`r.GetRemoteHistory`获取镜像的父节点`history`对象, *注意: 这里的父节点是包含自己的*.
1. 然后就是遍历`history`对象, 逐个下载镜像相关内容.
> 1. 通过`r.GetRemoteImageJSON`获取镜像的`json`内容`imgJSON`, 还有镜像大小`imgSize`
> 1. 由`imgJSON`通过`NewImgJSON`方法创建`Image`对象`img`
> 1. 通过`img`获取到镜像`layer结构`
> 1. 最后通过`graph.Register`下载并保存整个镜像的内容及其相关的上下文.

最后, 我们来分析一下`graph.Register`方法.

## graph.go

这是`Register`的方法声明.

    func (graph *Graph) Register(layerData Archive, store bool, img *Image) error

这是`Register`方法的重要步骤:

1. 先通过`ValidateID`验证一下`镜像ID`的有效性.
1. 通过`StoreImage`保存下载的镜像在临时目录.
1. 然后将下载完成的镜像保存在对应的镜像目录.
1. 最后再添加镜像到`graph`的缓存里面.

that's all
