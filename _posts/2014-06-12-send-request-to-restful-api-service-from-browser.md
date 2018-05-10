---
layout: post
title: 在浏览器中调用RESTful API Service
---

**以下内容请自备梯子**

[RESTful](http://zh.wikipedia.org/wiki/REST) API Service是以REST风格提供的API Service.

通常情况下会采用到HTTP的GET, POST, DELETE, PUT等方法.

因为浏览器的正常访问手段(直接通过连接访问或Form提交), 无法向Service发起DELETE, PUT等方法的请求.

所以只能通过[`XMLHttpRequest`](https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest)对象来提供完整交互操作.

那么最后的问题就回归到, 如果使用`XMLHttpRequest`对象上了.

参考上面给出的[`XMLHttpRequest`](https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest)的链接, 我们学习到使用方法.

    // javascript
    var xhr = new XMLHttpRequest();
    xhr.open("DELETE", "http://localhost:8888/messages/1");
    xhr.send();

上面就是`XMLHttpRequest`的简单使用方法. 当然, 这里只是发送了, 并完成后续的处理. 我们现在只关心发送.

那么在这里也奉上Server端的源代码.

    #! /usr/bin/env python2.7
    # filename: app.py

    from flask import Flask, make_response

    app = Flask(__name__)

    @app.route('/messages/<int:msg_id>', methods=['DELETE'])
    def delete_message(msg_id):
      return 'delete message #%s' % msg_id

    app.run(debug=True, port=8888)

上面就是服务端的代码.

但是通过浏览器console发起请求的时候, 我们可以观察到一下错误提示:

    OPTIONS http://localhost:8888/messages/1 No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'https://developer.mozilla.org' is therefore not allowed access.
    XMLHttpRequest cannot load http://localhost:8888/messages/1. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'https://developer.mozilla.org' is therefore not allowed access.

这两行错误提示表明, 我们需要在服务端上, 在返回`Response`对象时候, 在`Response Header`上提供`Access-Control-Allow-Origin`字段.

并且通过Chrome Developer Tools的Network中观察到, `XMLHttpRequest`会发送HTTP的`OPTIONS` Request到服务端上. 通过这次Request的Response的`Access-Control-Allow-Origin`和`Access-Control-Allow-Methods`这两个字段来决定, xhr是否有对该域发起请求的权限.

其中, `Access-Control-Allow-Origin`字段代表的是, 是否允许本域向API Service发起请求. `*`代表所有域都能发起请求.

而`Access-Control-Allow-Methods`则代表的允许请求的方法有那些.

那么这时我们就**将就地**添加`Access-Control-Allow-Origin`字段为`*` **(注意: 请在生产环境中谨慎添加允许跨域的主机), 添加`Access-Control-Allow-Methods`字段为`POST,DELETE,PUT,GET`.

再次修改服务端代码:

    #! /usr/bin/env python2.7
    # filename: app.py

    from flask import Flask, make_response

    app = Flask(__name__)

    @app.route('/messages/<int:msg_id>', methods=['OPTIONS'])
    def options_message(msg_id):
      resp = make_response('')
      resp.headers.extend({
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST,DELETE,PUT,GET'
      })
      return resp

    @app.route('/messages/<int:msg_id>', methods=['DELETE'])
    def delete_message(msg_id):
      resp = make_response('delete message #%s' % msg_id)
      resp.headers['Access-Control-Allow-Origin'] = '*'
      return resp

    app.run(debug=True, port=8888)

在`OPTIONS`处理函数中, 添加Response Header的`Access-Control-Allow-Origin`和`Access-Control-Allow-Methods`字段. 来声明权限和允许的方法.

在`DELETE`处理函数中, 添加`Access-Control-Allow-Origin`字段来声明权限.

that's all.
