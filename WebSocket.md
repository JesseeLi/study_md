## WebSocket 教程

HTTP 协议有一个缺陷：通信只能由客户端发起。

服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

### WebSocket 与 HTTP 的关系
- WebSocket 与 HTTP 协议一样都是基于 TCP 的，二者均为可靠的通信协议。
- WebSocket 与 HTTP 协议均为应用层协议，但 WebSocket 是一种独立于 HTTP 的协议。
- WebSocket 在建立握手连接时，数据是通过 HTTP 协议传输的。
- WebSocket 建立好连接后，真正通信阶段的数据传输不依赖于 HTTP 协议。

#### WebSocket 与 HTTP 轮询技术的对比

HTTP 实现实时 “推送” 用到的轮询技术主要分为两种：`短轮询` 与 `长轮询`。

短轮询：客户端每间隔特定时间向服务器发送 HTTP 请求拉取数据，而服务器收到请求后不管是否有新的数据信息都直接响应请求。可见短轮询有如下缺点：不必要请求过多，浪费带宽与服务器资源；并不能获得真正实时信息，除非服务器数据更新的间隔时间固定且等于设置的轮询间隔时间，否则服务器响应相对于数据更新必定存在一定的延迟时间。

长轮询：客户端向服务器发起 HTTP 请求后，服务器一直保持连接打开并阻塞，直到服务器有新数据更新并可以发送，或者等待一定时间直至超时后才会返回。收到服务器的响应后浏览器关闭连接，随即又向服务器发起一个新的请求。上述过程在浏览器页面打开期间一直持续不断。长轮询虽然减少了请求量，并且不再存在延时；但在未有数据更新时会长时间霸占服务器资源，造成服务器资源浪费，实时交互能力也不够强。


### WebSocket 特点

1. 建立在 TCP 协议之上，服务器端的实现比较容易。
1. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
1. 数据格式比较轻量，性能开销小，通信高效。
1. 可以发送文本，也可以发送二进制数据。
1. 没有同源限制，客户端可以与任意服务器通信。
1. 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

```markup
ws://example.com:80/some/path
```
### WebSocket 客户端API

#### WebSocket 构造函数
WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。

```javascript
var ws = new WebSocket('ws://localhost:8080');
```

#### webSocket.readyState

`readyState`属性返回实例对象的当前状态，共有四种。

- CONNECTING：值为0，表示正在连接。（connecting）
- OPEN：值为1，表示连接成功，可以通信了。（open）
- CLOSING：值为2，表示连接正在关闭。（closing）
- CLOSED：值为3，表示连接已经关闭，或者打开连接失败。（closed）

```javascript
switch (ws.readyState) {
  case WebSocket.CONNECTING:
    // do something
    break;
  case WebSocket.OPEN:
    // do something
    break;
  case WebSocket.CLOSING:
    // do something
    break;
  case WebSocket.CLOSED:
    // do something
    break;
  default:
    // this never happens
    break;
}
```

#### webSocket.onopen

实例对象的`onopen`属性，用于指定连接成功后的回调函数。

```javascript
ws.onopen = function () {
  ws.send('Hello Server!');
}
```

如果要指定多个回调函数，可以使用`addEventListener`方法。

```javascript
ws.addEventListener('open', function (event) {
  ws.send('Hello Server!');
});
```

#### webSocket.onclose

实例对象的`onclose`属性，用于指定连接关闭后的回调函数。

```javascript
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};

ws.addEventListener("close", function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
});
```

#### webSocket.onmessage

实例对象的`onmessage`属性，用于指定收到服务器数据后的回调函数。

```javascript
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};

ws.addEventListener("message", function(event) {
  var data = event.data;
  // 处理数据
});
```

注意，服务器数据可能是文本，也可能是二进制数据（`blob`对象或`Arraybuffer`对象）。

```javascript
ws.onmessage = function(event){
  if(typeof event.data === String) {
    console.log("Received data string");
  }

  if(event.data instanceof ArrayBuffer){
    var buffer = event.data;
    console.log("Received arraybuffer");
  }
}
```

除了动态判断收到的数据类型，也可以使用`binaryType`属性，显式指定收到的二进制数据类型。

```javascript
// 收到的是 blob 数据
ws.binaryType = "blob";
ws.onmessage = function(e) {
  console.log(e.data.size);
};

// 收到的是 ArrayBuffer 数据
ws.binaryType = "arraybuffer";
ws.onmessage = function(e) {
  console.log(e.data.byteLength);
};
```

#### webSocket.send()

实例对象的`send()`方法用于向服务器发送数据。

发送文本的例子。

```javascript
ws.send('your message');
```
发送 Blob 对象的例子。
```javascript
var file = document
  .querySelector('input[type="file"]')
  .files[0];
ws.send(file);
```
发送 ArrayBuffer 对象的例子。
```javascript
// Sending canvas ImageData as ArrayBuffer
var img = canvas_context.getImageData(0, 0, 400, 320);
var binary = new Uint8Array(img.data.length);
for (var i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);
```

#### webSocket.bufferedAmount

实例对象的`bufferedAmount`属性，表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束。

```javascript
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```

#### webSocket.onerror

实例对象的`onerror`属性，用于指定报错时的回调函数。

```javascript
socket.onerror = function(event) {
  // handle error event
};

socket.addEventListener("error", function(event) {
  // handle error event
});
```

