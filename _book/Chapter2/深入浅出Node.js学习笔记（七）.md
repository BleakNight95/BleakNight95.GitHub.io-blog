# 深入浅出Node.js学习笔记（七）

## 网络编程

利用Node可以十分方便的搭建网络服务器。

Node提供了net、dgram、http、https等4个模块，分别用于处理TCP、UDP、HTTP、HTTPS，适用于服务器端和客户端。

##  1. 构建TCP服务

TCP服务在网络应用中十分常见，大多数的应用都是基于TCP搭建而成的。

### 1.1 TCP

TCP全名为传输控制协议，在OSI模型(物理层、数据链路层、网络层、传输层、会话层、表示层、应用层)中属于传输层协议。

TCP是面向连接的协议，其显著的特征是传输之前需要3次握手形成会话。

只有会话形成后，服务器端和客户端之间才能相互发送数据。在创建会话的过程中，服务器端和客户端分别提供一个套接字，这两个套接字共同形成一个连接。服务器端和客户端则通过套接字实现两者之间的操作。

### 1.2 创建TCP服务器端

通过net.createServer(listener)即可创建一个TCP服务器，listener是连接事件connection的侦听器。

可以利用Telnet工具作为客户端对创建的服务器进行会话交流。

### 1.3 TCP服务的事件

TCP服务的事件分为服务器事件和连接事件。

1. **服务器事件**

   对于通过net.createServer()创建的服务器而言，它是一个EventEmitter实例，自定义事件有：

   - listening
   - connection
   - close
   - error

2. **连接事件**

   服务器可以同时与多个客户端保持连接，对于每个连接而言是典型的可写可读Stream对象。Stream对象可以用于服务器端和客户端之间的通信，既可以通过data事件从一端读取另一端发来的数据，也可以通过write()方法从一端向另一端发送数据。自定义事件：

   - data
   - end
   - connect
   - drain
   - error
   - close
   - timeout

   TCP套接字是可写可读的Stream对象，可以利用pipe()方法巧妙地实现管道操作。

   TCP针对网络中的小数据也有一定的优化策略：Nagle算法。

   Nagle算法：

   要求缓冲区的数据达到一定数量或者一定时间后才将其发出，所以小数据包会被Nagle算法合并，因此来优化网络。虽然网络带宽被有效地使用，但是数据有可能被延迟发送。

## 2. 构建UDP服务

UDP全称为用户数据包协议，与TCP一样属于网络传输层。UDP和TCP最大的不同是UDP不是面向连接的。

### 2.1创建UDP套接字

创建UDP套接字十分的简单，UDP套接字一旦被创建，既可以作为客户端发送数据，也可以作为服务器端接收数据。

```
var dgram = rerquire('dgram');
var socket = dgram.createSocket("udp4");
```

### 2.2 创建UDP服务器端

想要UDP套接字接收网络信息，只要调用dgram.bind(port,[address])方法对网卡和端口进行绑定即可。

创建UDP服务器端示例：

```
var dgram = require("dgram");
var server = dgram.createSocket("udp4");
server.on("message", function (msg, rinfo) {
    console.log("server got: " + msg + " from " +
        rinfo.address + ":" + rinfo.port);
});
server.on("listening", function () {
    var address = server.address();
    console.log("server listening " +
        address.address + ":" + address.port);
});
server.bind(41234);
```

### 2.3 创建UDP客户端

创建UDP客户端示例：

```
var dgram = require('dgram');
var message = new Buffer("深入浅出Node.js");
var client = dgram.createSocket("udp4");
client.send(message, 0, message.length, 41234, "localhost", function (err, bytes) {
    client.close();
});
```

### 2.4 UDP套接字事件

UDP套接字只是一个EventEmitter的实例。自定义事件：

- message
- listening
- close
- error

## 3. 构建HTTP服务

Node提供了基本的http和https模块用于HTTP和HTTPS的封装。

HTTP服务器的实现：

```
var http = require('http');
http.createServer(function (req, res) {
	res.writeHead(200, {'Content-Type': 'text/plain'});
	res.end('Hello World\n');
}).listen(1337, '127.0.0.1');
console.log('Server running at http://127.0.0.1:1337/');
```

###  3.1 HTTP

1. 初始HTTP

   HTTP的全称是超文本传输协议(HyperText Transfer Protocol)。

   HTTP构建在TCP之上，属于应用层协议。

2. HTTP报文

   HTTP是基于请求响应式的，以一问一答的方式实现服务。

   HTTP服务只做两件事：处理HTTP请求和发送HTTP响应。

   无论是HTTP请求报文还是HTTP响应报文，报文内容都包含两个部分：报文头和报文体。

### 3.2 HTTP模块

Node的http模块包含对HTTP处理的封装。

在Node中，HTPP服务继承自TCP服务器(net模块)，它能够与多个客户端保持连接，由于其采用事件驱动的形式，并不是为每一个连接创建额外的线程或进程，保持很低的内存占用，所以能够实现高并发。

HTTP服务和TCP服务模型的区别在于，在开启keepalive后，一个TCP会话可以用于多次请求和响应。TCP服务以connection为单元进行服务，HTTP服务以request为单位进行服务。

http模块将连接所用套接字的读写抽象为ServerRequest和ServerResponse对象，分别对应请求和响应操作。

1. HTTP请求

   对于TCP连接的读操作，http模块将其封装为ServerRequest对象。

2. HTTP响应

   HTTP响应对象封装了对底层连接的写操作，可以将其看做一个可写的流对象。

   响应结束后，HTTP服务器可能会将当前的连接用于下一个请求，或者关闭连接。

3. HTTP服务的事件

   HTTP服务也是个EventEmitter实例：

   - connection事件
   - request事件
   - close事件
   - checkContinue事件
   - connect事件
   - upgrade事件
   - clientError事件

### 3.3 HTTP客户端

HTTP客户端是服务器服务模型的另一部分，处在HTTP的另一端，在整个报文的参与中，报文头和报文体由它产生。

1. HTTP响应

   HTTP客户端在CLientRequest对象中，它的事件叫做response。

2. HTTP代理

   http提供的ClientRequest对象是基于TCP层实现的，在keepalive的情况下，一个底层会话连接可以多次用于请求。为了重用TCP连接，http模块包含一个默认的客户端代理对象http。globalAgent.t它对每个服务器端(host+port)创建的连接进行了管理，默认情况下，通过ClientRequest对象对同一个服务器发起的HTTP请求最多可以创建5个连接。它的实质是一个连接池。

   调用HTTP客户端同时对一个服务器发起10次请求时，其实质只有5个请求处于并发状态，后续的请求完成服务后才真正发出。这与浏览器对同一个域名有下载连接数的限制是相同的行为。

3. HTTP客户端事件

   - response
   - socket
   - upgrade
   - continue

## 4. 构建WebSocket服务

WebSocket与Node之间的配合堪称完美的理由：

- WebSocket客户端基于时间的编程模型与Node自定义事件相差无几；
- WebSocket实现了客户端和服务器端之间的长连接，而Node事件驱动的方式十分擅长于大量的客户端保持高并发连接；

WebSocket与传统HTTP的好处：

- 客户端与服务器端只建立一个TCP连接，可以使用更少的连接；
- WebSocket服务器端可以推送数据到客户端，这远比HTTP请求响应模式更加灵活、更加高效；
- 有更轻量级的协议头，减少数据传送量；

使用WebSocket,网页客户端只需要一个TCP连接即可完成双向通信，在服务器与客户端频繁通信时，无须频繁断开连接和重发请求。

WebSocket与HTTP的区别：

相比HTTP，WebSocket更接近于传输层协议，并没有在HTTP的基础上模拟服务器端的推送，而是在TCP上定义的独立的协议。

WebSocket协议主要分为两个部分:握手和数据传输。

### 4.1 WebSocket握手

一旦WebSocket握手成功，服务器端与客户端将会呈现对等的效果，都能接收和发送消息。

### 4.2 WebSocket数据传输

在握手顺利完成后，当前连接不再进行HTTP的交互，而是开始WebSocket的数据帧协议。实现客户端与服务器端的数据交换。

## 5. 网络服务与安全

### 5.1 TLS/SSL

1. 密钥

   TLS/SSL是一个公钥/私钥的结构，它是一个非对称的结构，每个服务器端和客户端都有自己的公私钥。

   公钥用来加密要传输的数据，私钥用来解密接收到的数据。

   公钥和私钥是配对的，通过公钥加密的数据，只有通过私钥才能解密，所以在建立安全传输之前，客户端和服务器端之间需要互换公钥。客户端发送数据时要通过服务器端的公钥进行加密，服务器端发送数据时则需要客户端的公钥进行加密。

   公私钥的非对称加密虽好，但是网络中依然可能存在窃听的情况，典型的例子就是中间人攻击。

   客户端和服务器端在交换公钥的过程中，中间人对客户端扮演服务器端的角色，对服务器端扮演客户端的角色，因此客户端和服务器端几乎感受不到中间人的攻击。

   为了解决中间人攻击，数据传输过程中还需要对得到的公钥进行认证，以确认得到的公钥是出自目标服务器。

2. 数字证书

   为了确保数据的安全，引入第三方：CA(Certificate Authority,数字证书认证中心)。

   CA的作用是为站点颁发证书，且这个证书中具有CA通过自己的公钥和私钥实现的签名。

   为了得到签名证书，服务器端需要通过自己的私钥生成CSR(Certificate  Signing Request，证书签名请求)文件。CA机构将通过这个文件颁发属于该服务器的签名证书，只要通过CA机构就能验证证书是否合法。

### 5.2 TLS服务

1. 创建服务器端
2. TLS客户端

与普通的TCP服务器端和客户端相比，TLS的服务器端和客户端仅仅只在证书的配置上有差别，其余部分基本相同。

### 5.3 HTTPS服务

HTTPS服务就是工作在TLS/SSL上的HTTP。

1. 准备证书
2. 创建HTTPS服务
3. HTTP客户端