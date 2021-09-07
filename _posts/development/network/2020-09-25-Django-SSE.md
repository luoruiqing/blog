---
title: "HTTP SSE"
subtitle: "HTTP SSE | 长连接 | 单向通信 | 服务器推送 | 非Websocket | 网络 | Django" 
layout: post
author: "luoruiqing"
header-style: text
tags:
  - Network
  - Django
---


## SSE

> EventSource 是服务器推送的一个网络事件接口。一个EventSource实例会对HTTP服务开启一个持久化的连接，以`text/event-stream` 格式发送事件, 会一直保持开启直到被要求关闭。
一旦连接开启，来自服务端传入的消息会以事件的形式分发至你代码中。如果接收消息中有一个事件字段，触发的事件与事件字段的值相同。如果没有事件字段存在，则将触发通用事件。
与 WebSockets,不同的是，服务端推送是单向的。数据信息被单向从服务端到客户端分发. 当不需要以消息形式将数据从客户端发送到服务器时，这使它们成为绝佳的选择。例如，对于处理社交媒体状态更新，新闻提要或将数据传递到客户端存储机制（如IndexedDB或Web存储）之类的，EventSource无疑是一个有效方案
-- [MDN - EventSource](https://developer.mozilla.org/zh-CN/docs/Server-sent_events/EventSource)


想要了解更多细节可以查看 [Server-Sent Events 教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)


## 格式

`SSE` 本身是基于 `HTTP` 协议的一种规则, 与 `文件流` 相似, 通俗点可以叫 **事件流**, 浏览器在接受到响应后会 **持续不断的拉取** 响应体中的内容, 同时在浏览器上请求和响应的行为还是会正常生效的, 例如传递 **Cookie**, **gzip** 压缩的解压过程等等.

#### 响应头

```text
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

需要将 `Cache-Control` 设置为不缓存, 同时 `Connection` 持续连接.

#### 响应体(流式)

每条消息可能包含 `retry` , `event`, `id` , `data` 以及 `:` 在数据行的 **开头**, 以表示不同的含义和操作, `\n\n` 代表一条信息的结束. 这里只说一下最简单的实现:

```
retry: 1000\n\n

data: 推送的消息.\n\n

data: 推送的消息.\n\n
```

## gzip

当服务端启用了 `GZip` 压缩时, 需要保证返回数据内容都是经过 **gzip压缩的**, 否则将响应头设置如下关闭解析压缩:

```
Content-Encoding: none  
```

## Nginx 支持

**Nginx** 作为代理服务时, 将会代理请求到后端服务, 但是内部存在 **缓存器**, 这会导致 **连接创建后浏览器不能直接获取到响应体** 也就是消息内容, 这时需要在 `服务端设置` 响应头来告知 **Nginx** 代理采用什么样的策略代理, 如下:

```
X-Accel-Buffering: no # 由上游控制是否缓存上游的响应
```


## 服务端

服务端可以结合 `Redis` 来定制更多的功能, 例如基于 `发布订阅` 去实现聊天室, 或者数据编辑锁定状态, 大屏实时数据等等实时性较高的场景.

#### 例子

这是一个基于 **Python** **Django** 框架简陋的 **SSE** 实现, 其他语言遵寻规则即可实现. 这里包含一个 **结束状态** 的捕获实现.

```py
from django.views import View as DjangoView
from django.http.response import StreamingHttpResponse


class SSECloseStatus:
    ''' 用于关闭状态的检测 '''

    def __init__(self, request, callback=lambda: None):
        self.request = request
        self.callback = callback

    def __del__(self):
        return self.callback(self.request)


class SSEStreamBaseView(DjangoView):
    ''' SSE 事件流视图实现 '''
    retry = 3000  # 重试间隔时间(毫秒)

    def iterator(self, request):
        ''' 每次推送的数据方法, 生成器 '''
        raise NotImplementedError()

    def close(self, request):
        ''' 关闭客户端触发的方法 '''
        raise NotImplementedError()

    def sse_iterator(self, request):
        ''' 符合协议的推送数据 '''
        _ = SSECloseStatus(request, self.close)  # 检测状态, 被释放时将触发close方法
        yield f'retry: {self.retry}\n\n'

        for data in self.iterator(request):
            yield f'data: {data}\n\n'

    def get(self, request):
        ''' 流响应数据 '''
        response = StreamingHttpResponse(self.sse_iterator(request), content_type='text/event-stream')
        response['Cache-Control'] = 'no-cache'
        response['Connection'] = 'keep-alive'
        response['Access-Control-Allow-Origin'] = '*'
        response['Content-Encoding'] = 'none'  # 不使用压缩
        response['X-Accel-Buffering'] = 'no'  # Nginx适应
        return response


class SSEStreamView(SSEStreamBaseView):
    def close(self, request):
        ''' 关闭客户端不处理 '''

```