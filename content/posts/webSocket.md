---
title: WebSocket知识点梳理
date: 2018-07-09T21:53:54+08:00
categories: ["Network"]
tags : ["Network", "webSocket"]
toc: true
featured_image : ""
keywords: ["WebSocket", "Network"]
description : "WebSocket知识点梳理"
---


## 什么是WebSocket
WebSocket是一种在单个TCP连接上进行全双工通讯的协议。它与HTTP协一样，同属于应用层协议。

## WebSocket解决了什么问题
WebSocket使得客户端和服务器之间的数据交换变得更加简单，**允许服务端主动向客户端推送数据**。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建**持久性的连接**，并进行**双向数据传输**。
简单说就是解决了浏览器和服务器之间双向数据传输的问题。

<!--more-->
## HTTP协议可以实现双向数据传输吗
答案肯定是可以的，在HTTP协议中我们通常使用**轮询**来实现双向通信。
轮询是通过在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

## HTTP1.1长连接与WebSocket长连接有什么区别
HTTP1.1默认启用"Connection: Keep-Alive"，使得在发送完http请求和应答后，不会立刻将连接关闭，在后续的http请求和应答可以继续使用这个连接，避免创建新的TCP连接时三次握手及断开连接时四次挥手的额外消耗。这个keep-alive一般会有固定的时间限制。如Apache是5s，而nginx默认是75s，超过这个时间服务器就会主动把TCP连接关闭了，因为不关闭的话会有大量的TCP连接占用系统资源。所以这个keep-alive并不是为了长连接设计的，只是为了提高http请求的效率。而WebSocket长连接的关闭可以由通过调用相应的API，主动控制。
HTTP1.1长连接是无状态的，每一个请求对应一个应答，并且每个请求和应答里面都包含了完整的头部信息；而WebSocket长连接是有状态的，在建立连接后，WebSocket只用携带少量头部字段信息（如数据包长度、掩码），不用携带状态信息。

## WebSocket的优点

- 较少的控制开销。在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小。在不包含扩展的情况下，对于服务器到客户端的内容，此头部大小只有2至10字节（和数据包长度有关）；对于客户端到服务器的内容，此头部还需要加上额外的4字节的掩码。相对于HTTP请求每次都要携带完整的头部，此项开销显著减少了。
- 更强的实时性。由于协议是全双工的，所以服务器可以随时主动给客户端下发数据。相对于HTTP请求需要等待客户端发起请求服务端才能响应，延迟明显更少；即使是和Comet等类似的长轮询比较，其也能在短时间内更多次地传递数据。
- 保持连接状态。与HTTP不同的是，Websocket需要先创建连接，这就使得其成为一种有状态的协议，之后通信时可以省略部分状态信息。而HTTP请求可能需要在每个请求都携带状态信息（如身份认证等）。
- 更好的二进制支持。Websocket定义了二进制帧，相对HTTP，可以更轻松地处理二进制内容。
- 可以支持扩展。Websocket定义了扩展，用户可以扩展协议、实现部分自定义的子协议。如部分浏览器支持压缩等。
- 更好的压缩效果。相对于HTTP压缩，Websocket在适当的扩展支持下，可以沿用之前内容的上下文，在传递类似的数据时，可以显著地提高压缩率。


## WebSocket兼容性情况
![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/webSocket-ws01.png)



<!-- ## WebSocket的缺点 -->
## WebSocket握手协议
WebSocket 是独立的、创建在 TCP 上的协议。

Websocket 通过 HTTP/1.1 协议的101状态码进行握手。

为了创建Websocket连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（handshaking）。

### 例子
一个典型的Websocket握手请求如下：

客户端请求

```js
GET / HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Host: example.com
Origin: http://example.com
Sec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==
Sec-WebSocket-Version: 13
```

服务器回应

```js
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: fFBooB7FAkLlXgRSz0BT3v4hq5s=
Sec-WebSocket-Location: ws://example.com/
```

### 字段说明
- Connection必须设置Upgrade，表示客户端希望连接升级。
- Upgrade字段必须设置Websocket，表示希望升级到Websocket协议。
- Sec-WebSocket-Key是随机的字符串，服务器端会用这些数据来构造出一个SHA-1的信息摘要。把“Sec-WebSocket-Key”加上一个特殊字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，然后计算SHA-1摘要，之后进行BASE-64编码，将结果做为“Sec-WebSocket-Accept”头的值，返回给客户端。如此操作，可以尽量避免普通HTTP请求被误认为Websocket协议。
- Sec-WebSocket-Version 表示支持的Websocket版本。RFC6455要求使用的版本是13，之前草案的版本均应当弃用。
- Origin字段是可选的，通常用来表示在浏览器中发起此Websocket连接所在的页面，类似于Referer。但是，与Referer不同的是，Origin只包含了协议和主机名称。
- 其他一些定义在HTTP协议中的字段，如Cookie等，也可以在Websocket中使用。

## 帧协议
客户端、服务端数据的交换，离不开数据帧格式的定义。因此，在实际讲解数据交换之前，我们先来看下WebSocket的数据帧格式。

WebSocket客户端、服务端通信的最小单位是帧（frame），由1个或多个帧组成一条完整的消息（message）。

发送端：将消息切割成多个帧，并发送给服务端；
接收端：接收消息帧，并将关联的帧重新组装成完整的消息；
本节的重点，就是讲解数据帧的格式。详细定义可参考 [RFC6455 5.2节](https://tools.ietf.org/html/rfc6455#section-5.2) 。
![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/webSocket-ws02.webp)


**FIN**：1个比特。

如果是1，表示这是消息（message）的最后一个分片（fragment），如果是0，表示不是是消息（message）的最后一个分片（fragment）。

**RSV1, RSV2, RSV3**：各占1个比特。

一般情况下全为0。当客户端、服务端协商采用WebSocket扩展时，这三个标志位可以非0，且值的含义由扩展进行定义。如果出现非零的值，且并没有采用WebSocket扩展，连接出错。

**Opcode**: 4个比特。

操作代码，Opcode的值决定了应该如何解析后续的数据载荷（data payload）。如果操作代码是不认识的，那么接收端应该断开连接（fail the connection）。可选的操作代码如下：

- %x0：表示一个延续帧。当Opcode为0时，表示本次数据传输采用了数据分片，当前收到的数据帧为其中一个数据分片。
- %x1：表示这是一个文本帧（frame）
- %x2：表示这是一个二进制帧（frame）
- %x3-7：保留的操作代码，用于后续定义的非控制帧。
- %x8：表示连接断开。
- %x9：表示这是一个ping操作。
- %xA：表示这是一个pong操作。
- %xB-F：保留的操作代码，用于后续定义的控制帧。

**Mask**: 1个比特。

表示是否要对数据载荷进行掩码操作。从客户端向服务端发送数据时，需要对数据进行掩码操作；从服务端向客户端发送数据时，不需要对数据进行掩码操作。

如果服务端接收到的数据没有进行过掩码操作，服务端需要断开连接。

如果Mask是1，那么在Masking-key中会定义一个掩码键（masking key），并用这个掩码键来对数据载荷进行反掩码。所有客户端发送到服务端的数据帧，Mask都是1。

掩码的算法、用途在下一小节讲解。

**Payload length**：数据载荷的长度，单位是字节。为7位，或7+16位，或1+64位。

假设数Payload length === x，如果

- x为0~126：数据的长度为x字节。
- x为126：后续2个字节代表一个16位的无符号整数，该无符号整数的值为数据的长度。
- x为127：后续8个字节代表一个64位的无符号整数（最高位为0），该无符号整数的值为数据的长度。

此外，如果payload length占用了多个字节的话，payload length的二进制表达采用网络序（big endian，重要的位在前）。

**Masking-key**：0或4字节（32位）

所有从客户端传送到服务端的数据帧，数据载荷都进行了掩码操作，Mask为1，且携带了4字节的Masking-key。如果Mask为0，则没有Masking-key。

备注：载荷数据的长度，不包括mask key的长度。

**Payload data**：(x+y) 字节

载荷数据：包括了扩展数据、应用数据。其中，扩展数据x字节，应用数据y字节。

扩展数据：如果没有协商使用扩展的话，扩展数据数据为0字节。所有的扩展都必须声明扩展数据的长度，或者可以如何计算出扩展数据的长度。此外，扩展如何使用必须在握手阶段就协商好。如果扩展数据存在，那么载荷数据长度必须将扩展数据的长度包含在内。

应用数据：任意的应用数据，在扩展数据之后（如果存在扩展数据），占据了数据帧剩余的位置。载荷数据长度 减去 扩展数据长度，就得到应用数据的长度。

### 掩码算法
掩码键（Masking-key）是由客户端挑选出来的32位的随机数。掩码操作不会影响数据载荷的长度。掩码、反掩码操作都采用如下算法：

首先，假设：

- original-octet-i：为原始数据的第i字节。
- transformed-octet-i：为转换后的数据的第i字节。
- j：为i mod 4的结果。
- masking-key-octet-j：为mask key第j字节。
-
算法描述为： original-octet-i 与 masking-key-octet-j 异或后，得到 transformed-octet-i。

```js
j = i MOD 4
transformed-octet-i = original-octet-i XOR masking-key-octet-j
```

### 数据掩码的作用
WebSocket协议中，数据掩码的作用是增强协议的安全性。但数据掩码并不是为了保护数据本身，因为算法本身是公开的，运算也不复杂。除了加密通道本身，似乎没有太多有效的保护通信安全的办法。

那么为什么还要引入掩码计算呢，除了增加计算机器的运算量外似乎并没有太多的收益（这也是不少同学疑惑的点）。

答案还是两个字：安全。但并不是为了防止数据泄密，而是为了防止早期版本的协议中存在的代理缓存污染攻击（proxy cache poisoning attacks）等问题。

### 跳动检测

在握手之后的任何时候，客户端或者服务器都可以选择向对方发送 ping 帧。 当收到一个 ping 帧，收件人必须尽快发回一个 pong 帧。 这是一次跳动。 你可以使用它来确保客户端保持着连接。

ping 帧或 pong 帧只是一个常规的帧，但它是一个控制帧。 ping 帧具有 0x9 的操作码，并且 pong 帧具有 0xA 的操作码。 当你得到一个 ping 帧，发回一个 pong 帧与 ping 帧完全相同的有效载荷数据（对于 pings 和 pongs ，最大有效载荷长度是 125 ）。 你也可能会得到一个 pong 帧返回，而无需再发送一个 ping 帧。如果它发生就忽略它。

跳动检测可能是非常有用的。 有些服务（如负载均衡器）会终止空闲连接。 另外，接收方无法查看远端是否已经终止。 只有在下一个发送时你会意识到出了问题。

### Sec-WebSocket-Key/Accept的作用
前面提到了，`Sec-WebSocket-Key/Sec-WebSocket-Accept`在主要作用在于提供基础的防护，减少恶意连接、意外连接。

作用大致归纳如下：

- 避免服务端收到非法的websocket连接（比如http客户端不小心请求连接websocket服务，此时服务端可以直接拒绝连接）
- 确保服务端理解websocket连接。因为ws握手阶段采用的是http协议，因此可能ws连接是被一个http服务器处理并返回的，此时客户端可以通过Sec-WebSocket-Key来确保服务端认识ws协议。（并非百分百保险，比如总是存在那么些无聊的http服务器，光处理Sec-WebSocket-Key，但并没有实现ws协议。。。）
- 用浏览器里发起ajax请求，设置header时，Sec-WebSocket-Key以及其他相关的header是被禁止的。这样可以避免客户端发送ajax请求时，意外请求协议升级（websocket upgrade）
- 可以防止反向代理（不理解ws协议）返回错误的数据。比如反向代理前后收到两次ws连接的升级请求，反向代理把第一次请求的返回给cache住，然后第二次请求到来时直接把cache住的请求给返回（无意义的返回）。
- Sec-WebSocket-Key主要目的并不是确保数据的安全性，因为Sec-WebSocket-Key、Sec-WebSocket-Accept的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）。

**强调：Sec-WebSocket-Key/Sec-WebSocket-Accept 的换算，只能带来基本的保障，但连接是否安全、数据是否安全、客户端/服务端是否合法的 ws客户端、ws服务端，其实并没有实际性的保证。**


### 数据传递
一旦WebSocket客户端、服务端建立连接后，后续的操作都是基于数据帧的传递。

WebSocket根据`opcode`来区分操作的类型。比如`0x8`表示断开连接，`0x0`-`0x2`表示数据交互。

#### 1、数据分片
WebSocket的每条消息可能被切分成多个数据帧。当WebSocket的接收方收到一个数据帧时，会根据`FIN`的值来判断，是否已经收到消息的最后一个数据帧。

FIN=1表示当前数据帧为消息的最后一个数据帧，此时接收方已经收到完整的消息，可以对消息进行处理。FIN=0，则接收方还需要继续监听接收其余的数据帧。

此外，`opcode`在数据交换的场景下，表示的是数据的类型。`0x01`表示文本，`0x02`表示二进制。而`0x00`比较特殊，表示延续帧（continuation frame），顾名思义，就是完整消息对应的数据帧还没接收完。

#### 2、数据分片例子
直接看例子更形象些。下面例子来自[MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers)，可以很好地演示数据的分片。客户端向服务端两次发送消息，服务端收到消息后回应客户端，这里主要看客户端往服务端发送的消息。

**第一条消息**

FIN=1, 表示是当前消息的最后一个数据帧。服务端收到当前数据帧后，可以处理消息。opcode=0x1，表示客户端发送的是文本类型。

**第二条消息**

1. FIN=0，opcode=0x1，表示发送的是文本类型，且消息还没发送完成，还有后续的数据帧。
2. FIN=0，opcode=0x0，表示消息还没发送完成，还有后续的数据帧，当前的数据帧需要接在上一条数据帧之后。
3. FIN=1，opcode=0x0，表示消息已经发送完成，没有后续的数据帧，当前的数据帧需要接在上一条数据帧之后。服务端可以将关联的数据帧组装成完整的消息。

```js
Client: FIN=1, opcode=0x1, msg="hello"
Server: (process complete message immediately) Hi.
Client: FIN=0, opcode=0x1, msg="and a"
Server: (listening, new message containing text started)
Client: FIN=0, opcode=0x0, msg="happy new"
Server: (listening, payload concatenated to previous message)
Client: FIN=1, opcode=0x0, msg="year!"
Server: (process complete message) Happy new year to you too!
```

## Websocket API

### WebSocket 的用法示例

```js
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) {
  console.log("Connection open ...");
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};
```

### WebSocket 构造函数

WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。

```js
var ws = new WebSocket('ws://localhost:8080');
```
执行上面语句之后，客户端就会与服务器进行连接。

实例对象的所有属性和方法清单，参见这里。

### webSocket.readyState
readyState属性返回实例对象的当前状态，共有四种。

- CONNECTING：值为0，表示正在连接。
- OPEN：值为1，表示连接成功，可以通信了。
- CLOSING：值为2，表示连接正在关闭。
- CLOSED：值为3，表示连接已经关闭，或者打开连接失败。

下面是一个示例。

```js
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

### webSocket.onopen
实例对象的onopen属性，用于指定连接成功后的回调函数。

```js
ws.onopen = function () {
  ws.send('Hello Server!');
}
```
如果要指定多个回调函数，可以使用addEventListener方法。

```js
ws.addEventListener('open', function (event) {
  ws.send('Hello Server!');
});
```
### webSocket.onclose
实例对象的onclose属性，用于指定连接关闭后的回调函数。

```js
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

### webSocket.onmessage
实例对象的onmessage属性，用于指定收到服务器数据后的回调函数。

```js
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};

ws.addEventListener("message", function(event) {
  var data = event.data;
  // 处理数据
});
```
注意，服务器数据可能是文本，也可能是二进制数据（blob对象或Arraybuffer对象）。

```js
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
除了动态判断收到的数据类型，也可以使用binaryType属性，显式指定收到的二进制数据类型。

```js
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

### webSocket.send()
实例对象的send()方法用于向服务器发送数据。

发送文本的例子。

```js
ws.send('your message');
```

发送 Blob 对象的例子。

```js
var file = document
  .querySelector('input[type="file"]')
  .files[0];
ws.send(file);
```

发送 ArrayBuffer 对象的例子。

```js
// Sending canvas ImageData as ArrayBuffer
var img = canvas_context.getImageData(0, 0, 400, 320);
var binary = new Uint8Array(img.data.length);
for (var i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);
```

### webSocket.bufferedAmount
实例对象的bufferedAmount属性，表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束。

```js
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```
### webSocket.onerror
实例对象的onerror属性，用于指定报错时的回调函数。

```js
socket.onerror = function(event) {
  // handle error event
};

socket.addEventListener("error", function(event) {
  // handle error event
});
```


---
*参考链接：
https://zh.wikipedia.org/zh-cn/WebSocket
https://juejin.im/post/5b0a31f851882538bb0cfae2
https://cloud.tencent.com/document/product/214/4150?fromSource=gwzcw.93403.93403.93403
https://www.cnblogs.com/chyingp/p/websocket-deep-in.html
https://www.zhihu.com/question/20215561
https://mp.weixin.qq.com/s/7aXMdnajINt0C5dcJy2USg
https://www.oschina.net/translate/how-does-javascript-actually-work-part-5
http://www.ruanyifeng.com/blog/2017/05/websocket.html*