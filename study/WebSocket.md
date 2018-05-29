## WebSocket
学习一门新的技术，首先要善用搜索引擎，一般看个几篇文章就会对一个知识点有一个宏观的了解（面向搜索引擎编程）。首先通过搜索引擎我们应该能知道`WebSocket`的基本定义：
> **WebSocket**是一种在单个TCP连接上进行全双工通讯的协议。`WebSocket`使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在`WebSocket API`中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

通过以上我们可以了解到：

- `WebSocket`是一种网络通信协议
- 建立在`TCP`协议之上
- 允许服务端主动向客户端推送数据。


#### 为什么需要WebSocket


那么为什么要新增这样一个协议呢？ 因为`HTTP`协议只能由客户端主动发起请求。在`WebSocket`协议之前，我们要实现即时通讯的功能只能使用`AJAX`轮询（每隔一段时间，就发出一个请求）等方式。轮询的效率是非常低的，而且也非常浪费资源。

轮询就类似于我们追电视剧的时候，每隔一天就需要去`APP`上查看是否有更新，而 `WebSocket`协议则是 `APP` 在电视剧更新的时候主动给我们发送一条消息。

#### 基本介绍

协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是`URL`，如：

```
ws://example.com/wsapi
wss://secure.example.com/
```

`Websocket`使用和`HTTP`相同的`TCP`端口，可以绕过大多数防火墙的限制。默认情况下，`Websocket`协议使用80端口；运行在`TLS`之上时，默认使用443端口。`Websocket`通过`HTTP/1.1`协议的101状态码进行握手。	

#### 客户端API

`WebSocket`对象作为一个构造函数，用于新建`WebSocket`实例

```javascript
var webSocket = new WebSocket('ws://localhost:8080');
```

`WebSocket.readyState`属性，只读属性`readyState`表示连接状态，可以是以下值：

- `WebSocket.CONNECTING`：值为0，表示正在连接。
- `WebSocket.OPEN`：值为1，表示连接成功，可以通信了。
- `WebSocket.CLOSING`：值为2，表示连接正在关闭。
- `WebSocket.CLOSED`：值为3，表示连接已经关闭，或者打开连接失败。

`webSocket.onopen`属性，用于指定连接成功后的回调函数。

```javascript
webSocket.onopen = function(evnt) {
	console.log("链接服务器成功!")
};
```

` webSocket.onclose`属性，用于指定连接关闭后的回调函数。

```javascript
webSocket.onclose = function(evnt) {
    console.log("与服务器断开了链接!")
}
```

`webSocket.onmessage`属性，用于指定收到服务器数据后的回调函数。

```javascript
webSocket.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};
```

`webSocket.onerror`属性，用于指定报错时的回调函数。

```javascript
webSocket.onerror = function(evnt) {
     // handle error event
};
```

`webSocket.send()`方法，用于向服务器发送数据。

```javascript
webSocket.send("Hello World!");
```

#### 服务端的实现

由于`WebSocket`是一个协议，服务器具体怎么实现，取决于所用编程语言和框架本身。Node.js、Java、C++、Python 等多种语言都有自己的解决方案。

> `Java`的`web`一般都依托于`servlet`容器。其中`Tomcat7`、`Jetty7`及以上版本均开始支持`WebSocket`

本篇文章采用`node.js`来实现服务端，略过安装`node.js`的过程，在项目空间下使用命令安装`nodejs-websocket`模块：

```
npm install --save nodejs-websocket
```

服务器端代码如下：

```javascript
var log = console.log.bind(console);
var ws = require("nodejs-websocket"); //引入websocket模块
var server  = ws.createServer(function(conn){
    //服务端接收到消息后触发
    conn.on("text",function(msg){
        log("Recevied:"+msg); // 后台打印接受的信息
        // conn.sendText(msg); 向当前连接的客户端发生消息
        // 服务器就向全部客户端广播一条消息
        server.connections.forEach(function(connection) { 
            connection.sendText(msg);
        });
    });

    conn.on("close",function(code,reason){
        log("Connection closed");
    });

    conn.on("error",function(err){
        log("Handle Error");
        log(err);
    });
}).listen(8009);
```

保存为`index.js`文件，然后运行命令：`node index.js`，此时服务器运行成功。

#### 实现客户端

客户端提供一个输入框用来输入消息，以及一个向服务端发生消息的按钮：

```javascript
<!DOCTYPE>
<html>
  <head>
    <meta charset="utf-8">
    <title>Socket.IO charset</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font: 13px Helvetica, Arial; }
        form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
        form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
        form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
        #messages { list-style-type: none; margin: 0; padding: 0; }
        #messages li { padding: 5px 10px; }
        #messages li:nth-child(odd) { background: #eee; }
      </style>
</head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
    <script src="js/jquery.min.js"></script>
    <script>
      var log = console.log.bind(console);

      var ws;
      if ('WebSocket' in window) {
        ws = new WebSocket("ws://127.0.0.1:8009"); // 注意换成实际的ip地址
      } else if ('MozWebSocket' in window) {
        ws = new MozWebSocket("ws://127.0.0.1:8009");
      }
      ws.onopen = function(evnt) {
        log("connected server");
      };
      ws.onclose = function(evnt) {
        log("connect closed");
      };
      
      ws.onmessage = function(evnt){
        log(evnt.data);
        $('#messages').append($('<li>').text(evnt.data));
      };

      ws.onerror = function(evnt) {
        // handle error event
      };
      //向服务器发生消息
      $("form").submit(function(){
        ws.send($("#m").val());
        $("#m").val(""); //清空消息框
        return false; //阻止form提交
      });
  </script>
  </body>
</html> 
```

这时我们就完成了我们的聊天应用了。写完这样一个简单的`Demo`，我们就可以在简历上加上[了解XXX技术了]

### 参考文章

[WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561/answer/40316953)

[WebSocket 教程](http://www.ruanyifeng.com/blog/2017/05/websocket.html)

[nodejs-websocket](https://www.npmjs.com/package/nodejs-websocket)



