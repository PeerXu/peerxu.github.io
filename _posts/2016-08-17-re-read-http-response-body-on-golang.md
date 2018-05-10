---
layout: post
title: 使golang中的http response body可重读
---

## 前言

首先来介绍一下今天遇到的问题: 在读取`net/http.Response.Body`时, 发现只能用`ioutil.ReadAll`读取一遍, 然后再读取就会失败了.

这就相当的尴尬了, 因为我读了一遍, 只是`Middleware`做检查, 后面还需要将`Response`对象传递到下个使用者上.

## 问题分析

当我分析`net/http/response.go`文件时, 发现`Response`对象的`Body`属性定义如下:

<pre>
type Response struct {
...
	// Body represents the response body.
	//
	// The http Client and Transport guarantee that Body is always
	// non-nil, even on responses without a body or responses with
	// a zero-length body. It is the caller's responsibility to
	// close Body. The default HTTP client's Transport does not
	// attempt to reuse HTTP/1.0 or HTTP/1.1 TCP connections
	// ("keep-alive") unless the Body is read to completion and is
	// closed.
	//
	// The Body is automatically dechunked if the server replied
	// with a "chunked" Transfer-Encoding.
	Body io.ReadCloser
...
}
</pre>

上面我们能看见, `Body`是一个`ReadCloser`对象, 那么意味着, 不能采用`Seek`方法将`Body`锚定到目标位置.

那么, 唯有采取更加极端的方式了.

## 解决方案

上面也说过了, 因为`Body`是`ReadCloser`, 不能`Seek`, 所以, 只能把`Body`读出来, 保存到`buffer`里面, 然后再将`buffer`包装成`io.ReadCloser`, 然后再绑定到`Response.Body`上.

直接上代码吧.

<pre>
import (
	"bytes"
	"io"
	"io/ioutil"
	"net/http"
)

func ReadAndAssignResponseBody(res *http.Response) (io.Reader, error) {
	buf, err := ioutil.ReadAll(res.Body)
	if err != nil {
		return nil, err
	}

	res.Body = ioutil.NopCloser(bytes.NewReader(buf))
	return bytes.NewReader(buf), nil
}
</pre>

enjoy it~
