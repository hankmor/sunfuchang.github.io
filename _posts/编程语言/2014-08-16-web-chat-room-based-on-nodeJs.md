---
layout: post
category : 编程语言
tagline: 
tags : [nodeJs, 聊天室, socket]
excerpt : 
title_cn: node学习第一课——基于nodeJs的web聊天室程序 
description: 本文为node学习的第一课，通过实现一个web聊天室程序，对nodeJs做一个整体认识和体验，用到socket.io和mime组建、node核心http模块、node核心fs文件系统模块。
---
{% include JB/setup %}

## 源码地址

[https://github.com/sunfuchang/nodeJs-study/tree/master/lesson2/chat](https://github.com/sunfuchang/nodeJs-study/tree/master/lesson2/chat)

## 一、所需组建

* socket.io组件库([http://en.wikipedia.org/wiki/WebSocket](http://en.wikipedia.org/wiki/WebSocket))

为实时通讯而设计的轻量的双向通讯协议。

* mime组件库([http://en.wikipedia.org/wiki/MIME](http://en.wikipedia.org/wiki/MIME))

用于根据文件扩展名获取文件的MIME类型。

## 二、文件定义

该文件为项目的包描述文件，定义了项目的名称、版本、描述、依赖组建等内容，例如：
{% highlight javascript %}
{
    "name": "chatrooms",
    "version": "0.0.1",
    "description": "聊天程序",
    "dependencies": {
        "socket.io": "0.9.6",
        "mime": "1.2.7"
    }
}
{% endhighlight %}
dependencies定义了项目所需要的第三方依赖库，包括依赖库得每次和需要的版本号，定义完成后，可以使用node的包管理器npm来安装这些依赖。

## 三、使用`npm install`安装依赖

进入项目的根目录，执行`npm install`命令，node会自动下载dependencies定义的库对应的版本，下载完成后的依赖库会放到项目根目录的node_modules文件夹中。

## 四、模块的引入：

使用`require`方法将模块引入，例如：
{% highlight javascript %}
// 加载socket.io模块
var socketIo = require('socket.io');
{% endhighlight %}

## 五、模块方法的导出

模块放到项目根目录下的lib文件夹中，模块定义的方法不能被外部访问，可以通过`exports`对象将模块的方法到处到外部，例如：
{% highlight javascript %}
exports.listen = function (server) {
    // some code here
}
{% endhighlight %}
那么外部在引入模块后就可以使用该模块的listen方法。

## 六、node的事件驱动

node采用的是单线程的，而其非阻塞I/O依赖于它的事件驱动机制。事件驱动编程底层依赖于事件循环（event loop），事件循环基本上是事件检测和事件处理器触发这两种函数不断循环调用的一个结构。在每次循环里，事件循环机制需要检测发生了哪些事件，当事件发生时，它找到对应的回调函数并调用它。例如：

传统的数据库查询执行：
{% highlight javascript %}
result = query('SELECT * FROM posts WHERE id = 1');
do_something_with(result); // 必须等到result结果返回才能继续执行，此时I/O被阻塞
{% endhighlight %}

而如果采用事件驱动：
{% highlight javascript %}
query_finished = function(result) {
    // 查询结束后回调执行
    do_something_with(result);
}
query('SELECT * FROM posts WHERE id = 1', query_finished);
// 不必等到query完成，而代码可以继续，非阻塞I/O
do_something_other
{% endhighlight %}

## 七、连接的建立

通过自定义模块的`listen`方法，传入http server来对其进行监听：
{% highlight javascript %}
// 将listen方法倒出以供外部调用
exports.listen = function (server) {
    // 启动socket.io服务器，搭载在server服务器上
    io = socketIo.listen(server);
    io.set('log level', 1);
    // 处理每个用户连接到聊天室的逻辑
    io.sockets.on('connection', function(socket) {
        ...
    }
    ...
}

/**
 * 创建服务器
 */
 var server = http.createServer(function (req, resp) {
    ...
 }).listen(3000, function () {
    console.log('Server is running on port 3000.');
 });

// 启动socket.io服务器
chatServer.listen(server);
{% endhighlight %}

客户端连接，需要引入`socket.io.js`文件：
{% highlight html %}
<script type="text/javascript" src="/socket.io/socket.io.js"></script>
{% endhighlight %}
然后连接到服务器上：
{% highlight javascript %}
var socket = io.connect('http://127.0.0.1:3000');
{% endhighlight %}
这样就拿到了客户端的socket，我们根据注册事件，供服务端触发，同时也可以触发服务端的事件。

## 八、事件触发

在socket.io中，通过`on`方法来注册事件，而通过`emit`方法来发射事件：

服务端注册一个`message`事件：
{% highlight javascript %}
// 注册服务端message事件，广播服务器响应的消息
socket.on('message', function (msg) {
    // 触发客户端socket的message事件，广播msg
    socket.broadcast.to(msg.room).emit('message', {
        text : nickNames[socket.id] + ' : ' + msg.text
    });
});
{% endhighlight %}

客户端的`socket`通过`emit`方法调用：
{% highlight javascript %}
// 消息发送方法
Chat.prototype.sendMessage = function (room, text) {
    var msg = {
        room : room,
        text : text
    };
    // 触发客户端message事件
    this.socket.emit('message', msg);
}
{% endhighlight %}

## 九、用到的socket api

* 建立socket.io服务器<br>
`socketIo.listen(server);`<br>
* 注册事件<br>
`socket.on`<br>
* 触发事件<br>
`socket.emit`<br>
socket.emit信息传输对象为当前socket对应的client，各个client socket相互不影响。
* 加入房间，分组<br>
`socket.join`<br>
* 发送广播<br>
`socket.broadcast.to`<br>
* 不知道是什么对象？<br>
`io.sockets.manager`<br>
* 建立连接<br>
`io.connect`<br>
* `socket.broadcast.emit`<br>
socket.broadcast.emit信息传输对象为所有client，排除当前socket对应的client。
* io.sockets.emit<br>
信息传输对象为所有client。<br>
* key为房间名，value为房间名对应的socket ID数组<br>
`io.sockets.manager.rooms`<br>
* 获取particular room中的客户端，返回所有在此房间的socket实例<br>
`io.sockets.clients('particular room')`<br>
* 通过socket.id来获取此socket进入的房间信息<br>
`io.sockets.manager.roomClients[socket.id]`<br>

详见：[https://github.com/sunfuchang/nodeJs-study/wiki](https://github.com/sunfuchang/nodeJs-study/wiki)