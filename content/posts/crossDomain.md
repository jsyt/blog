---
title: 浏览器常见跨域方式梳理
date: 2018-06-29T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript", "Browser"]
toc: true
featured_image : ""
keywords: ["浏览器", "跨域"]
description : "浏览器常见跨域方式梳理"
---


跨域是由于[浏览器同源策略](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)导致的，所以跨域只存在于浏览器端，非浏览器端不存在跨域问题，浏览器对跨域的请求、应答都能正常发送接收，只是浏览器在接收跨域应答时，将应答拦截了，所以我们需要一些额外的处理或设置让浏览器将跨域的应答返回给我们。


#### 常见的跨域处理方式有：
* jsonp
* CORS
* iframe + postMessage
* iframe + window.name
* iframe + location.hash
* iframe + domain
* nginx代理
* Nodejs中间件
* WebSocket
<!--more-->
### jsonp 跨域
jsonp 跨域是利用`script`标签天生具备跨域的特性，`script`的`src`属性发起的请求不受浏览器同源策略的限制，所以我们可以动态生成一个`script`标签对象，将要请求数据的`url`赋值给`script`标签的`src`属性，然后将此`script`标签`append`到`body`中。但是服务端怎么返回数据呢，返回的数据又如何处理呢？此时我们还需要预先写好一个解析数据的函数`analyzeData`，并将这个函数名通过请求的`url`一并传给后端，后端收到后，直接请求的数据放在解析函数`analyzeData`的参数中`analyzeData({a:1,b:2,c:3})`并返回，当次`script`标签加载完毕后就会直接执行`analyzeData({a:1,b:2,c:3})`方法。

```js
// 预先写好一个解析数据的函数
function analyzeData(data) {
  console.log(data)
}

var sct = document.createElement('script')
sct.src = 'http://goyth.com/json?callback=analyzeData'
document.body.appendChild(sct)
```

jsop 的优点是兼容性好，能兼容低版本的浏览器，缺点是只支持`get`请求，不支持其他方式的请求，并且对回调函数的错误处理不太友好。

## CORS(跨域资源共享)
CORS 全称为跨域资源共享（Cross-origin resource sharing），它是 W3C 用来允许`XMLHttpRequest`请求跨域的一个标准，也是现在主流的跨域方案。
CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。IE8+：IE8/9需要使用XDomainRequest对象来支持CORS。

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/crossOrigin-cors.png)


整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

跨域请求只需要在服务端应答报文头增加一个`Access-Control-Allow-Origin`字段，该字段是的值要么是请求时`Origin`字段的值，要么是一个 * ，`Access-Control-Allow-Origin=Origin字段的值` 表示只接受该域的请求，`Access-Control-Allow-Origin=*` 表示接受任意域名的请求。此时的请求是不带`Cookie`的，如果需要带上`Cookie`，则需要在发起请求时将 `XMLHttpRequest` 实例的 `withCredentials` 属性设置为 `true`，在服务端将`Access-Control-Allow-Credentials`字段设置为`true`。

以下内容直接参考[阮一峰老师的微博](http://www.ruanyifeng.com/blog/2016/04/cors.html)
CORS 分为简单请求和非简单请求，简繁请求符合以下要求：
（1) 请求方法是以下三种方法之一：
* HEAD
* GET
* POST

（2）HTTP的头信息不超出以下几种字段：

* Accept
* Accept-Language
* Content-Language
* Last-Event-ID
* Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

凡是不同时满足上面两个条件，就属于非简单请求。

### 简单请求
#### 基本流程
对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个`Origin`字段。

下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个`Origin`字段。

```
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
上面的头信息中，`Origin`字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果`Origin`指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含`Access-Control-Allow-Origin`字段（详见下文），就知道出错了，从而抛出一个错误，被`XMLHttpRequest`的`onerror`回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果`Origin`指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

```
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```
上面的头信息之中，有三个与CORS请求相关的字段，都以Access-Control-开头。

（1）Access-Control-Allow-Origin

该字段是必须的。它的值要么是请求时`Origin`字段的值，要么是一个*，表示接受任意域名的请求。

（2）Access-Control-Allow-Credentials

该字段可选。它的值是一个布尔值，表示是否允许发送`Cookie`。默认情况下，`Cookie`不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送`Cookie`，删除该字段即可。

（3）Access-Control-Expose-Headers

该字段可选。CORS请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。上面的例子指定，`getResponseHeader('FooBar')`可以返回FooBar字段的值。

#### withCredentials 属性
上面说到，CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定`Access-Control-Allow-Credentials`字段。

```
Access-Control-Allow-Credentials: true
```
另一方面，开发者必须在AJAX请求中打开withCredentials属性。

```
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```
否则，即使服务器同意发送Cookie，浏览器也不会发送。或者，服务器要求设置Cookie，浏览器也不会处理。

但是，如果省略`withCredentials`设置，有的浏览器还是会一起发送Cookie。这时，可以显式关闭withCredentials。

```
xhr.withCredentials = false;
```
需要注意的是，如果要发送Cookie，`Access-Control-Allow-Origin`就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。

### 非简单请求
#### 预检请求
非简单请求是那种对服务器有特殊要求的请求，比如请求方法是`PUT`或`DELETE`，或者`Content-Type`字段的类型是`application/json`。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

下面是一段浏览器的JavaScript脚本。

```
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```
上面代码中，HTTP请求的方法是`PUT`，并且发送一个自定义头信息`X-Custom-Header`。

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的HTTP头信息。

```
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
"预检"请求用的请求方法是`OPTIONS`，表示这个请求是用来询问的。头信息里面，关键字段是`Origin`，表示请求来自哪个源。

除了`Origin`字段，"预检"请求的头信息包括两个特殊字段。

（1）Access-Control-Request-Method

该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。

（2）Access-Control-Request-Headers

该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是X-Custom-Header。

#### 预检请求的回应
服务器收到"预检"请求以后，检查了`Origin`、`Access-Control-Request-Method`和`Access-Control-Request-Headers`字段以后，确认允许跨源请求，就可以做出回应。

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```
上面的HTTP回应中，关键的是`Access-Control-Allow-Origin`字段，表示http://api.bob.com可以请求数据。该字段也可以设为星号，表示同意任意跨源请求。

```
Access-Control-Allow-Origin: *
```
如果浏览器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。控制台会打印出如下的报错信息。

```
XMLHttpRequest cannot load http://api.alice.com.
Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.
```
服务器回应的其他CORS相关字段如下。

```
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```
（1）Access-Control-Allow-Methods

该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

（2）Access-Control-Allow-Headers

如果浏览器请求包括`Access-Control-Request-Headers`字段，则`Access-Control-Allow-Headers`字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

（3）Access-Control-Allow-Credentials

该字段与简单请求时的含义相同。

（4）Access-Control-Max-Age

该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

#### 浏览器的正常请求和回应
一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个`Origin`头信息字段。服务器的回应，也都会有一个`Access-Control-Allow-Origin`头信息字段。

下面是"预检"请求之后，浏览器的正常CORS请求。

```
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
上面头信息的`Origin`字段是浏览器自动添加的。

下面是服务器正常的回应。

```
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
```
上面头信息中，`Access-Control-Allow-Origin`字段是每次回应都必定包含的。

### 与JSONP的比较
CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。

## iframe + postMessage
postMessage是HTML5 XMLHttpRequest Level 2中的API，且是为数不多可以跨域操作的window属性之一，它可用于解决以下方面的问题：
a.） 页面和其打开的新窗口的数据传递
b.） 多窗口之间消息传递
c.） 页面与嵌套的iframe消息传递
d.） 上面三个场景的跨域数据传递

以 a.com 域下的 a.html 页面向 b.com 域下 b.html 页面通讯为例

a.com/a.html

```js
var iframe = document.createElement('iframe')
iframe.src = 'http://b.com/b.html'
iframe.style.display = 'none'
document.body.appendElement(iframe)

iframe.onload = function() {
  let msg = {domain : 'a.html'}
  iframe.contentWindow.postMessage(msg, 'http://b.com')
}
window.addEventListener('message', function(msg) {
  let origin = msg.origin  // 消息来源地址 'http://b.com'
  let data = msg.data  // 传送过来的数据 this message is from b.html
  let source = msg.source  // 源window对象
  console.log(data)  //this message is from b.html
}, false)

```

b.com/b.html
```js
window.addEventListener('message', function(msg) {
  let origin = msg.origin  // 消息来源地址 'http://a.com'
  let data = msg.data  // 传送过来的数据 {domain : 'a.html'}
  let source = msg.source  // 源window对象
  console.log(data)  //{domain : 'a.html'}
}, false)
let data = 'this message is from b.html'
window.parent.postMessage(data, 'http://a.html')

```

### postMessage的使用方法：
- otherWindow.postMessage(message, targetOrigin);
   - otherWindow
    其他窗口的一个引用，比如iframe的contentWindow属性、执行window.open返回的窗口对象、或者是命名过或数值索引的window.frames。
   - message
    不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化。
   - targetOrigin
    通过窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符串"*"（表示无限制）或者一个URI。在发送消息的时候，如果目标窗口的协议、主机地址或端口这三者的任意一项不匹配targetOrigin提供的值，那么消息就不会被发送；只有三者完全匹配，消息才会被发送。这个机制用来控制消息可以发送到哪些窗口；例如，当用postMessage传送密码时，这个参数就显得尤为重要，必须保证它的值与这条包含密码的信息的预期接受者的origin属性完全一致，来防止密码被恶意的第三方截获。如果你明确的知道消息应该发送到哪个窗口，那么请始终提供一个有确切值的targetOrigin，而不是*。不提供确切的目标将导致数据泄露到任何对数据感兴趣的恶意站点。

### message 的属性有:

- data
  从其他 window 中传递过来的对象。
- origin
  调用 postMessage  时消息发送方窗口的 origin . 这个字符串由 协议、“://“、域名、“ : 端口号”拼接而成。例如 “https://example.org (隐含端口 443)”、“http://example.net (隐含端口 80)”、“http://example.com:8080”。请注意，这个origin不能保证是该窗口的当前或未来origin，因为postMessage被调用后可能被导航到不同的位置。
- source
  对发送消息的窗口对象的引用; 您可以使用此来在具有不同origin的两个窗口之间建立双向通信。


## iframe + window.name

window.name 有一个特性就是，在一个窗口（window）下的所有的页面都是共享一个 window.name 对象，只要是同域的情况下，每个页面都可以对window.name 进行读写，不同域是读取不到的。因此我们可以用一个代理iframe将这个iframe的域设置为要请求数据的域，此时是同域请求，是可以请求到数据的，然后将请求的数据赋值给 window.name 对象，再将iframe的域跳转到当前域，然后读取window.name 的值，就实现了跨域通讯。
我们以http://a.com域下的a.html页面向http://b.com域发起一个http://b.com/json请求为例

我们首先创建一个http://b.com域下的代理页面proxy.html，通过proxy.html 去发起http://b.com/json请求，注意此时是同域的所以可以请求成功

```html
<!DOCTYPE html>
<html>
<head>
</head>
<body>
<script>
let xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
  if(xhr.readystate === '4' && xhr.status === '200') {
    window.name = xhr.responseText  // 将获取到的数据赋值给window.name
    location.href = 'http://a.com/index.html'  // 将iframe 的域设置回http://a.com
  }
}
xhr.open('http://b.com/json')
xht.send()
</script>
</body>
</html>
```

通过iframe将代理页面proxy.html与a.html放到一个窗口下，这样就可以读取到 proxy.html 页面设置的 window.name 的值。

http://a.com/a.html
```html
<!DOCTYPE html>
<html>
<head>
</head>
<body>
<script>
let iframe = document.createElement('iframe')
let time = 0
iframe.onload = function() {
  if(++time = 2) {
    var data  = iframe.contentWindow.name  // 取出window.name
    console.log(data)
  }
}
iframe.src = 'http://b.com/proxy.html'
document.body.appendChild(iframe)

</script>
</body>
</html>


```

## iframe + location.hash
与 iframe + window.name 一样，此方法也是通过一个与请求同域的代理页面去发起请求，然后将请求到的数据放在a.com 域 的hash 部分，赋值给 `parent.location.href`，这样就可以触发 a.com 域下的 onhashchange 事件，然后通过 `location.hash` 就可以读取到 hash 部分的数据了

代理页面 http://b.com/proxy.html
```html
<!DOCTYPE html>
<html>
<head>
</head>
<body>
<script>
let xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
  if(xhr.readystate === '4' && xhr.status === '200') {
    let hash = JSON.stringify(xhr.responseText)  // 将获取到的数据赋值给window.name
    parent.location.href = 'http://a.com/index.html#'+ hash // 此时会触发父页面的onhashchange事件
    // 注意这个地方不能使用parent.location.hash，子页面需要与父页面同域才能修改父页面的location.hash，此时是不同域的
  }
}
xhr.open('http://b.com/json')
xht.send()
</script>
</body>
</html>
```


http://a.com/a.html
```html
<!DOCTYPE html>
<html>
<head>
</head>
<body>
<script>
let iframe = document.createElement('iframe')
iframe.src = 'http://b.com/proxy.html'
document.body.appendChild(iframe)
window.onhashchange = function() {
  let data = location.hash.slice(1)
  console.log(data)
}
</script>
</body>
</html>

```


## iframe + domain

这种跨域方式只适用与主域相同的情况下的跨域通信，假如我们现在有两个页面a.html和b.html分别在a.goyth.com和b.goyth.com域下，此时他们的主域都是goyth.com，我们现在只用把两个页面下的 `document.domain` 都设置成一样，就可以跨域通信了，可以相互读取对方window对象下的数据。但要注意的是，document.domain的设置是有限制的，我们只能把document.domain设置成自身或更高一级的父域，且主域必须相同。

a.goyth.com/a.html
```html
<iframe id = "iframe" src="http://b.goyth.com/b.html" onload = "test()"></iframe>
<script type="text/javascript">
    function test(){
        var iframe = document.getElementById('￼ifame');
        var win = document.contentWindow;//可以获取到iframe里的window对象，但该window对象的属性和方法几乎是不可用的
        var doc = win.document;//这里获取不到iframe里的document对象
        var name = win.name;//这里同样获取不到window对象的name属性
    }
</script>
```

此时将a.html和b.html的`document.domain`都设置为`goyth.com`

a.goyth.com/a.html
```html
<iframe id = "iframe" src="http://b.goyth.com/b.html" onload = "test()"></iframe>
<script type="text/javascript">
  document.domain = 'goyth.com'
  function test(){
      var iframe = document.getElementById('￼ifame');
      var win = document.contentWindow;//可以获取到iframe里的window对象，但该window对象的属性和方法几乎是不可用的
      var doc = win.document;//这里获取不到iframe里的document对象
      var name = win.name;//这里同样获取不到window对象的name属性
  }
</script>
```

b.goyth.com/b.html
```html

<script type="text/javascript">
  document.domain = 'goyth.com'
</script>
```

## nginx代理跨域

### nginx配置解决iconfont跨域
浏览器跨域访问js、css、img等常规静态资源被同源策略许可，但iconfont字体文件(eot|otf|ttf|woff|svg)例外，此时可在nginx的静态资源服务器中加入以下配置。
```
location / {
  add_header Access-Control-Allow-Origin *;
}
```
### nginx反向代理接口跨域
跨域原理： 同源策略是浏览器的安全策略，不是HTTP协议的一部分。服务器端调用HTTP接口只是使用HTTP协议，不会执行JS脚本，不需要同源策略，也就不存在跨越问题。

实现思路：通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。

nginx具体配置：
```
#proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;

    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
```
1.) 前端代码示例：
```js
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问nginx中的代理服务器
xhr.open('get', 'http://www.domain1.com:81/?user=admin', true);
xhr.send();
```
2.) Nodejs后台示例：
```js
var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var params = qs.parse(req.url.substring(2));

    // 向前台写cookie
    res.writeHead(200, {
        'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'   // HttpOnly:脚本无法读取
    });

    res.write(JSON.stringify(params));
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');
```


## Nodejs中间件代理跨域
node中间件实现跨域代理，原理大致与nginx相同，都是通过启一个代理服务器，实现数据的转发。

### 非vue框架的跨域（2次跨域）
利用node + express + http-proxy-middleware搭建一个proxy服务器。

1.）前端代码示例：
```js
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问http-proxy-middleware代理服务器
xhr.open('get', 'http://www.domain1.com:3000/login?user=admin', true);
xhr.send();
```
2.）中间件服务器：
```js
var express = require('express');
var proxy = require('http-proxy-middleware');
var app = express();

app.use('/', proxy({
    // 代理跨域目标接口
    target: 'http://www.domain2.com:8080',
    changeOrigin: true,

    // 修改响应头信息，实现跨域并允许带cookie
    onProxyRes: function(proxyRes, req, res) {
        res.header('Access-Control-Allow-Origin', 'http://www.domain1.com');
        res.header('Access-Control-Allow-Credentials', 'true');
    },

    // 修改响应信息中的cookie域名
    cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
}));

app.listen(3000);
console.log('Proxy server is listen at port 3000...');
```
3.）Nodejs后台同（六：nginx）

### vue框架的跨域（1次跨域）
利用node + webpack + webpack-dev-server代理接口跨域。在开发环境下，由于vue渲染服务和接口代理服务都是webpack-dev-server同一个，所以页面与代理接口之间不再跨域，无须设置headers跨域信息了。

webpack.config.js部分配置：
```js
module.exports = {
    entry: {},
    module: {},
    ...
    devServer: {
        historyApiFallback: true,
        proxy: [{
            context: '/login',
            target: 'http://www.domain2.com:8080',  // 代理跨域目标接口
            changeOrigin: true,
            cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
        }],
        noInfo: true
    }
}
```



## WebSocket协议跨域
WebSocket protocol是HTML5一种新的协议。它实现了浏览器与服务器全双工通信，同时允许跨域通讯，是server push技术的一种很好的实现。原生WebSocket API使用起来不太方便，我们使用Socket.io，它很好地封装了webSocket接口，提供了更简单、灵活的接口，也对不支持webSocket的浏览器提供了向下兼容。

1.）前端代码：
```html
<div>user input：<input type="text"></div>
<script src="./socket.io.js"></script>
<script>
var socket = io('http://www.domain2.com:8080');

// 连接成功处理
socket.on('connect', function() {
    // 监听服务端消息
    socket.on('message', function(msg) {
        console.log('data from server: ---> ' + msg);
    });

    // 监听服务端关闭
    socket.on('disconnect', function() {
        console.log('Server socket has closed.');
    });
});

document.getElementsByTagName('input')[0].onblur = function() {
    socket.send(this.value);
};
</script>
```
2.）Nodejs socket后台：
```js
var http = require('http');
var socket = require('socket.io');

// 启http服务
var server = http.createServer(function(req, res) {
    res.writeHead(200, {
        'Content-type': 'text/html'
    });
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');

// 监听socket连接
socket.listen(server).on('connection', function(client) {
    // 接收信息
    client.on('message', function(msg) {
        client.send('hello：' + msg);
        console.log('data from client: ---> ' + msg);
    });

    // 断开处理
    client.on('disconnect', function() {
        console.log('Client socket has closed.');
    });
});
```

