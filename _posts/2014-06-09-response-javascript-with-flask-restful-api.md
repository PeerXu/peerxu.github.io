---
layout: post
title: 采用Flask-Restful插件返回Javascript脚本的两种方法
---

[Flask](https://github.com/mitsuhiko/flask) 是Python的一个轻量级的Web框架.

[Flask-Restful](https://github.com/twilio/flask-restful) 是将Flask应用构建为Restful API的一个插件.

一个Flask-Restful插件的demo:

    $ cat app.py
    #! /usr/bin/env python2.7
    # filename: app.py

    from flask import Flask
    from flask.ext.restful import Api, Resource

    app = Flask(__name__)
    api = Api(app)

    class User(Resource):
      def get(self, user_id):
        return {'user_id': user_id}

    api.add_resource(User, '/users/<int:user_id>')

    if __name__ == '__main__':
      app.run(port=8888, debug=True)

    $ python app.py
     * Running on http://127.0.0.1:8888/
     * Restarting with reloader

这里我们可以看见Flask Demo已经成功运行.

我们可以通过`curl`这个工具来进行简单测试.

    $ curl -v http://localhost:8888/users/1
    * Adding handle: conn: 0x22f5a40
    * Adding handle: send: 0
    * Adding handle: recv: 0
    * Curl_addHandleToPipeline: length: 1
    * - Conn 0 (0x22f5a40) send_pipe: 1, recv_pipe: 0
    * About to connect() to localhost port 8888 (#0)
    *   Trying 127.0.0.1...
    * Connected to localhost (127.0.0.1) port 8888 (#0)
    > GET /users/1 HTTP/1.1
    > User-Agent: curl/7.32.0
    > Host: localhost:8888
    > Accept: */*
    >
    * HTTP 1.0, assume close after body
    < HTTP/1.0 200 OK
    < Content-Type: application/json
    < Content-Length: 21
    < Server: Werkzeug/0.9.4 Python/2.7.5+
    < Date: Mon, 09 Jun 2014 15:25:42 GMT
    <
    {
        "user_id": 1
    }
    * Closing connection 0

curl的Response部分, 包含了一些信息.

其中Content-Type字段表示的是, 返回的内容是json格式的.

如果我们需要返回的值是javascript脚本的话, 那么我们就需要对demo进行一些改造.

有两种办法:

### 1. 通过Flask的make_response函数

    $ cat app.py
    #! /usr/bin/env python2.7
    # filename: app.py

    import simplejson as json
    from flask import Flask, make_response
    from flask.ext.restful import Api, Resource

    app = Flask(__name__)
    api = Api(app)

    class User(Resource):
      def get(self, user_id):
        user = {'user_id': user_id}
        data = json.dumps(user)
        response = make_response(data)
        response.headers.extend({
          'Content-Type': 'application/javascript'
        })
        return response

    api.add_resource(User, '/users/<int:user_id>')

    if __name__ == '__main__':
      app.run(port=8888, debug=True)

添加Content-Type字段到response的headers中, 可以将返回的内容类型设置为javascript.

### 2. 通过Request Headers限制Response类型

HTTP协议中规定, 在Request Header中指定Accept字段, 可以指定客户端接受的Response类型.

例如在curl中指定Accept字段:

    $ curl -H 'Accept: application/javascript' http://localhost:8888/users/1

但是在Flask-Restful插件中, 貌似是不支持javascript作为返回值类型.

所以我们需要做一些patch.


    #! /usr/bin/env python2.7
    # filename: app.py

    import simplejson as json
    from flask import Flask, make_response
    from flask.ext.restful import Api, Resource

    # monkey patch: application/javascript mimetype

    from flask.ext import restful
    def output_javascript(data, code, headers=None):
      response = make_response(data, code)
      response.headers.extend(headers or {})
      return response

    restful.DEFAULT_REPRESENTATIONS['application/javascript'] = output_javascript

    # end monkey patch

    app = Flask(__name__)
    api = Api(app)

    class User(Resource):
      def get(self, user_id):
        return {'user_id': user_id}

    api.add_resource(User, '/users/<int:user_id>')

这种方式显得更加优雅.

但是需要在发起Request的时候设置Request Header的`Accept`字段为`application/javascript`.

the end.
