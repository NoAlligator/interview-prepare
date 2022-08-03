# 1.`URL`和`URI`的联系和区别

[参考](https://segmentfault.com/a/1190000019487884)

`URI`用字符串标识某一互联网资源。

`URL`表示资源的地点（在互联网上所处的位置）。

![clipboard.png](https://github.com/NoAlligator/pico/blob/main/img/bVbtVQR?raw=true)

# 2.说一说简单请求和预检请求的区别？

### ⭐**简单请求**：满足以下几种情况（日常开发基本上只会注意前两种）

1. 使用`GET、POST、HEAD`其中一种方法。
2. 只使用了如下的安全首部字段，不得**人为设置其他首部字段**：
   - `Accept`
   - `Accept-Language`
   - `Content-Language`
   - `Content-Type` 仅限以下三种，注意不包含`application/json`：
     - `text/plain`
     -  `multipart/form-data`
     -  `application/x-www-form-urlencoded`

- `HTML`头部`header field`字段：`DPR、Download、Save-Data、Viewport-Width、WIdth`。

1. 请求中的任意`XMLHttpRequestUpload` 对象均没有注册任何事件监听器；`XMLHttpRequestUpload` 对象可以使用 `XMLHttpRequest.upload` 属性访问。
2. 请求中没有使用 `ReadableStream` 对象。

### ⭐预检（非简单）请求，满足以下几种情况：

1. 使用了`PUT、DELETE、CONNECT、OPTIONS、TRACE、PATCH`方法。
2. **人为设置了非规定内的其他首部字段**，参考上面简单请求的安全字段集合，还要特别注意`Content-Type`的类。
3. `XMLHttpRequestUpload` 对象注册了任何事件监听器。
4. 请求中使用了`ReadableStream`对象。

### ⭐请求附带身份凭证

发起请求时设置`withCredentials` 标志设置为`true`，从而向服务器发送`cookie`， 但是**如果服务器端的响应中未携带`Access-Control-Allow-Credentials: true`，浏览器将不会把响应内容返回给请求的发送者**。

 对于**附带身份凭证的请求**，服务器**不得设置`Access-Control-Allow-Origin` 的值为`*`**， 必须是某个具体的域名。

简单的`GET`请求不会触发预检，如果**对此类带有身份凭证请求的响应中不包`Access-Control-Allow-Credentials: true`，这个响应将被忽略掉，并且浏览器也不会将相应内容返回给网页**。

# 3.说一说`HEAD请求` 和 `OPTIONS`请求

`HEAD`方法跟`GET`方法相同，只不过服务器响应时**不会返回响应体**。一个`HEAD`请求的响应中，响应头中包含的元信息应该和一个`GET`请求的响应消息相同。这种方法可以用来**获取请求中隐含的元信息，而不用传输实体本身**。也经常用来测试超链接的有效性、可用性和最近的修改。`HEAD`请求能够**在有限的速度和带宽下提供很多有用的信息**：只**请求资源的首部**、检查`URI`的**有效性**、检查网页**是否被篡改**、检测何时更新。如果 `HEAD` 请求的结果显示在上一次 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET) 请求后缓存的资源已经过期了, 即使没有发出[`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)请求，缓存也会失效。可以使用`HEAD`请求代替一次具体的`GET`请求向服务器发送一次头部的验证进行资源协商。

| 请求是否有正文                                               | 否   |
| :----------------------------------------------------------- | ---- |
| 成功的响应是否有正文                                         | 否   |
| 安全                                                         | 是   |
| [幂等](https://developer.mozilla.org/zh-CN/docs/Glossary/Idempotent) | 是   |
| 可缓存                                                       | 是   |
| [`HTML`表单](https://developer.mozilla.org/en-US/docs/Learn/Forms) 是否支持 | 否   |

> 跨域资源共享标准新增了一组`HTTP`首部字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，**对那些可能对服务器数据产生副作用的 `HTTP `请求方法（特别是 `GET`以外的 `HTTP` 请求，或者搭配某些 `MIME` 类型的 `POST` 请求），浏览器必须首先使用`OPTIONS`方法发起一个预检请求（`preflight request`），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 `HTTP` 请求。**在预检请求的返回中，服务器端也可以通知客户端，**是否需要携带身份凭证**（包括 `Cookies` 和 `HTTP` 认证相关数据）。

`OPTIONS`请求即预检请求，可用于检测服务器允许的`http`方法。当发起**跨域请求**时，由于安全原因，触发一定条件时浏览器会在正式请求之前自动先发起`OPTIONS`请求，即**`CORS`预检请求**，服务器若**接受该跨域请求**，浏览器才继续发起正式请求。

**预检请求**(`preflight request`)头**关键**字段：

| Request Header                     | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| **Access-Control-Request-Method**  | 告诉服务器实际请求所使用的 HTTP 方法                         |
| **Access-Control-Request-Headers** | 告诉服务器实际请求所携带的自定义首部字段，本次实际请求首部字段中content-type为自定义 |
| **Origin**                         | 由浏览器主动带上，代表当前请求源                             |

服务器**基于从预检请求头部获得的信息来判断**，是否接受接下来的实际请求。

| response header                      | 作用                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| **Access-Control-Allow-Methods**     | 返回了服务端允许的请求，包含GET/HEAD/PUT/PATCH/POST/DELETE   |
| **Access-Control-Allow-Credentials** | 允许跨域携带cookie（跨域请求要携带cookie必须设置为true）     |
| **Access-Control-Allow-Origin**      | 允许跨域请求的域名，这个可以在服务端配置一些信任的域名白名单 |
| **Access-Control-Request-Headers**   | 客户端请求所携带的自定义首部字段content-type                 |

此次`options`请求返回了响应头的内容，但**不返回响应实体`response body`内容**。

> `OPTIONS`预检请求的结果可以被**缓存**。`Access-Control-Max-Age`这个响应首部表示 `preflight request`（预检请求）的返回结果（即 `Access-Control-Allow-Methods` 和`Access-Control-Allow-Headers` 提供的信息） 可以被缓存的最长时间，单位是秒。

如何避免触发，一般是**修改请求的具体细节**，使其满足一个简单请求的要求（例如**更改`Content-Type`**）。

# 4.`Trace`请求

[参考](https://segmentfault.com/a/1190000013182974)

客户端发起一个请求时，这个请求可能要穿过防火墙、代理、网关或其他一些应用程序。每个中间节点都可能会修改原始的 `HTTP`请求。`TRACE` 方法允许**客户端在 最终将请求发送给服务器时，看看它变成了什么样子**。

`TRACE` 请求会在目的服务器端发起一个 `环回` 诊断。行程最后一站的服务器会弹回一条 `TRACE` 响应，并在响应主体中携带它收到的原始请求报文。这样客户端就可以查看在所有中间 `HTTP` 应用程序组成的请求 / 响应链上，原始报文是否，以及如何被毁坏或修改过。

**TRACE 方法主要用于诊断**；也就是说，**用于验证请求是否如愿穿过了请求 / 响应链。它也是一种很好的工具，可以用来查看代理和其他应用程序对用户请求所产生 效果**。

`Trace`请求中不能带有实体的主体部分。`TRACE `响应的实体主体部分包含了响应服务器收到的请求的精确副本。

# 5.`Connect`请求

`CONNECT`方法是`HTTP/1.1`协议预留的，能够将连接改为**隧道方式**的代理服务器。通常用于`SSL、TLS`加密服务器的链接与非加密的`HTTP`代理服务器的通信。

> CONNECT 方法可以开启一个**客户端与所请求资源之间的双向沟通的通道**。它可以用来创建隧道

# 6.说一说`HTTP`的长连接？

## TCP是长连接，UDP是短连接？

答：**长/短连接都是在TCP的前提下来说的，因为UDP没有连接的概念。**UDP Client不需要与Server建立连接，它只需要在需要的时候发一个包出去就可以了。所以，更准确地说法应该是：TCP长连接、TCP短连接。

## TCP是长连接，HTTP是短连接？

答：误以为TCP只能是长连接。我们再说一遍，TCP的连接有长有短。另外，HTTP 1.0确实是短连接，但是，**HTTP 1.0加入了keepalive之后，也可以实现长连接（本质还是TCP长连接）**。再到后来，出现了websocket，就是默认长连接的协议。

[参考](https://segmentfault.com/a/1190000015821798)

> - HTTP1.0默认不开启KeepAlive，因此要使用的话需要浏览器支持，在发送HTTP请求时主动携带 Connection: Close头。 
> - HTTP1.1规范明确规定了要默认开启KeepAlive，所以支持HTTP1.1的浏览器不需要显式指定，发送请求时会自动携带该头部，只有在想关闭时可以通过设置 Connection: Close。

> 之所以网络上说HTTP分为长连接和短连接，其实本质上是说的TCP连接。TCP连接是一个双向的通道，它是可以保持一段时间不关闭的，因此TCP连接才有真正的长连接和短连接这一说。

`HTTP`协议本身是无连接的，他的长连接乃至短链接是依赖传输层`TCP`协议实现的，早期`HTTP/1.0`默认使用短连接，也初步实现了长连接，主流的的`HTTP/1.1`默认使用长连接，确保连接的复用来减少服务器重复建立和销毁连接带来的巨大开销，同时也能提升服务端获取资源的效率。

## 如何实现长连接？

`HTTP/1.0`和`HTTP/1.1`通过使用`Connection:keep-alive`进行长连接（使用长连接的`HTTP`协议，会在响应头加入这行代码：`Connection:keep-alive`）。在一次 `TCP` 连接中可以完成多个`HTTP`请求，但是对每个请求仍然要单独发`header`，`Keep-Alive`不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如`Apache`）中设定这个时间。这种长连接是一种“**伪链接**”，而且**只能由客户端发送请求，服务端响应**。

## 关于`WebSocket`的长连接？

`Websocket`是`html5`提出的一个协议规范，是为解决客户端与服务端实时通信。本质上是一个**基于`TCP`**，先**通过`HTTP/HTTPS`协议**发起一条特殊的`HTTP`请求进行握手后创建一个用于换数据的`TCP`连接。

`WebSocket`是一个全双工的连接，可由服务端主动发起信息。长连接第一次`TCP`链路建立之后，后续数据可以双方都进行发送，不需要发送请求头。

## `WebSocket`长连接和`HTTP/1.1`长连接的区别？

[参考](https://www.cnblogs.com/ricklz/p/11108320.html)

`WebSocket`的长连接，是一个**真正的全双工**。`WbSocket`长连接第一次`TCP`链路建立之后，后续数据可以**双方都进行发送，不需要发送请求头或只需要发送很少的请求头**。

`keep-alive`双方**并没有建立正真的双向连接会话**，服务端可以在任何一次请求完成后关闭。它本身就规定了是**真正的、双工的**长连接，两边都必须要维持住连接的状态。

总的来说，`keep-alive`是服务端和客户端的一种**复用`TCP`连接的约定**，使得服务端维持了原先建立的`TCP`，而客户端也**被告知**可以**复用原先的`TCP`连接进行资源的请求**，但是这种请求依然是单向的，只能主动从客户端向服务端发起资源的请求，而且每一个请求依然**必须携带上单独的请求头**。而`WebSocket`请求也是基于`TCP`西协议建立的**长连接**，但是最大的不同是建立了**真正意义上的双向连接**，在建立`TCP`连接之后后续发起的的请求请求头不是必须的，所以**客户端和服务端**之间交换的头信息很少。（类比：一问一答`keep-alive`& 互相问答`WebSocket`）

# 7.说一说`HTTP/2`?

[什么是HTTP/2?](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650120831&idx=1&sn=2bcafb931eff897e2f866bd02dcda680&chksm=f36bbf5ec41c36481c2f5da1f74cf48fca374bb5b5922d26ac4ace7858cf8ede4fc2cefc1584&scene=21#wechat_redirect)

⭐什么是`pipelining`？

`HTTP/1.1`时期，持久连接（长连接）的**弊端**被提出 —— `HOLB（Head of Line Blocking，队头阻塞）`即持久连接下一个连接中的请求仍然是**串行**的，如果**某个请求出现网络阻塞等问题，会导致同一条连接上的后续请求被阻塞。**因此，`HTTP/1.1`中提出了`pipelining`概念，即客户端可以**在一个请求发送完成后不等待响应便直接发起第二个请求，服务端在返回响应时会按请求到达的顺序依次返回**，这样就极大地降低了延迟。然而`pipelining`并没有解决`HOLB`的问题，因为响应依然是**串行返回**的，`pipelining`也因此没有被广泛接受。

⭐什么是`multiplexing`?

`multiplexing`即**多路复用**，连接共享，在`SPDY`中提出，同时也在`HTTP/2`中实现。`multiplexing`技术能够让**多个请求和响应的传输完全混杂在一起进行**，通过`streamId`（流序号）来互相区别。这彻底解决了`HOLB`问题，同时**还允许给每个请求设置优先级，服务端会先响应优先级高的请求**。

⭐`HTTP/2`的最大区别？

- 二进制分帧。在`HTTP/2`中，在应用层（`HTTP2.0`）和传输层（`TCP`或者`UDP`）之间加了一层：二进制分帧层。在二进制分帧层中， `HTTP/2`会将所有传输的信息分割为**更小的消息和帧**（`frame`），并对它们采用二进制格式的编码。这种单连接多资源的方式，**减少了服务端的压力，使得内存占用更少，连接吞吐量更大。**而且，`TCP`连接数的减少**使得网络拥塞状况得以改善，同时慢启动时间的减少，使拥塞和丢包恢复速度更快**。
- 多路复用。多路复用允许同时通过单一的`HTTP/2.0`连接发起**多重的请求-响应消息**。在`HTTP1.1`协议中，**浏览器客户端在同一时间，针对同一域名下的请求有一定数量的限制，超过了这个限制的请求就会被阻塞。**而多路复用允许同时通过单一的`HTTP/2.0`连接**发起多重的“请求-响应”消息**。`HTTP/2.0`的请求的`TCP`的连接一旦建立，后续请求以`stream`的方式发送。每个`stream`的基本组成单位是`frame`（二进制帧）。客户端和服务器可以把`HTTP`消息**分解为互不依赖的帧，然后乱序发送**，最后再在另一端把它们**重新组合**起来。
- 头部压缩。`HTTP/1.1`的头部带有大量信息，而且每次都要重复发送。`HTTP/2`为了减少这部分开销，采用了`HPACK`头部压缩算法对`header`进行压缩。
- 服务端推送。**当用户的浏览器和服务器在建立连接后，服务器主动将一些资源推送给浏览器并缓存起来的机制。**有了缓存，当浏览器想要访问已缓存的资源的时候就可以**直接从缓存中读取**。

⭐`HTTP/2.0`是如何使用解决请求管道（`pipelining`）的头阻塞的问题的？

**HTTP/2废弃了管道化的方式**，而是创新性的引入了**帧、消息和数据流**等概念。客户端和服务器可以把 HTTP 消息分解为互不依赖的帧，然后乱序发送，最后再在另一端把它们重新组合起来。因为没有顺序了，所以就不需要阻塞了，就有效的解决了HTTP对头阻塞的问题。

但是，`HTTP/2`**仍然会存在队头阻塞的问题**，那是因为`HTTP/2`其实还是依赖`TCP`协议实现的。TCP传输过程中会把数据拆分为一个个**按照顺序**排列的数据包，这些数据包通过网络传输到了接收端，接收端再**按照顺序**将这些数据包组合成原始数据，这样就完成了数据传输。但是如果其中的某一个数据包没有按照顺序到达，接收端会一直保持连接等待数据包返回，这时候就会阻塞后续请求。这就发生了**TCP队头阻塞**。

`HTTP/1.1`的管道化持久连接也是使得同一个`TCP`链接可以被多个`HTTP`使用，但是`HTTP/1.1`中规定一个域名**只可以有`6`个`TCP`连接**。而`HTTP/2`中，**同一个域名只用一个`TCP`连接**。所以，**在HTTP/2中，TCP队头阻塞造成的影响会更大**，因为`HTTP/2`的多个请求都会受到影响，但是减少`TCP`连接的直接好处就是减少了服务器压力，降低了网络拥塞的可能。

# 8.`HTTP`无状态的优缺点？

`HTTP`是无状态协议，也就是他**不会对之前发生过的请求和响应的状态进行管理**，也就是**无其他手段帮助的情况**下无法根据之前的状态进行本次的请求处理。

优点：不必保存状态、不必建立维持连接、协议更加简单，使得能够减少服务器的资源负担。

缺点：无法维持、检测用户状态和辨别用户，必须依靠其他手段（`Cookie`、`Token`）。

# 9.`Cookie`的基本原理？

因为`HTTP`协议本身是**无状态的**，这就是的客户端发往服务器的请求都是**独立且不相互影响的**，服务器也无法辨别发送该请求的客户端是谁，在这种情况下**每次发送请求都需要额外携带完整认证信息**。保留无状态协议的同时又要解决该问题，就需要引入一种`Cookie`机制来**变通地实现用户辨别以及状态管理**，`Cookie`是在客户端经过服务端通过某种方式认证之后颁发的，他被携带在服务器认证后发送的响应报文的响应头`Set-Cookie`字段，通知客户端会保存该`Cookie`，当后续客户端发送请求的时候会在请求头`Cookie`字段上自动携带上该`Cookie`，服务端接收到该请求之后会读取`Cookie`，并以此为依据来辨别用户并获取用户之前的状态。

`Cookie`通常是有过期（`expire`）时间的。

# 10.说一说`HTTP`的分块传输编码？

**分块传输编码**（`Chunked transfer encoding`）是超文本传输协议（`HTTP`）中的一种数据传输机制，允许`HTTP`由应用服务器发送给客户端应用（ 通常是网页浏览器）的数据分成多个部分。分块传输编码只在`HTTP`协议`1.1`版本（`HTTP/1.1`）中提供。

分块传输编码的优点：

- **`HTTP`分块传输编码允许服务器为动态生成的内容维持`HTTP`持久连接。**通常，持久链接需要服务器在开始发送消息体前发送`Content-Length`消息头字段，但是对于动态生成的内容来说，在内容创建完之前是不可知的。**[动态内容，`content-length`无法预知]**
- **分块传输编码允许服务器在最后发送消息头字段。**对于那些头字段值在内容被生成之前无法知道的情形非常重要，例如消息的内容要使用散列进行签名，散列的结果通过HTTP消息头字段进行传输。没有分块传输编码时，服务器必须缓冲内容直到完成后计算头字段的值并在发送内容前发送这些头字段的值。[散列签名，需缓冲完成才能计算]
- `HTTP`服务器有时**使用压缩 （`gzip`或`deflate`）以缩短传输花费的时间**。分块传输编码可以用来分隔压缩对象的多个部分。在这种情况下，块不是分别压缩的，而是整个负载进行压缩，压缩的输出使用本文描述的方案进行分块传输。在压缩的情形中，**分块编码有利于一边进行压缩一边发送数据，而不是先完成压缩过程以得知压缩后数据的大小。**[`gzip`压缩，压缩与传输同时进行]

# 11.说一说`MIME`?

`MIME`是最早是用于**扩展邮件的媒体类型**，早期邮件使用`SMTP`只能够支持`ASCII`字符内容的传输，在引入`MIME`之后扩展了传输各种类型媒体内容的能力，后来也被逐渐运用在`HTTP`协议中。`MIME`使得客户端可以根据相应的头字段来判断媒体类型进行对应的解析。

`MIME`，全称`Multipurpose Internet Mail Extensions`，即多用途因特网邮件扩展机制。它是一种标准，用来表示文档、文件或字节流的性质和格式。在最早的`HTTP`协议中没有附加的数据类型信息，所有传送的数据都被客户程序解释为超文本标记语言，而之后为了支持数据类型，`HTTP`协议就是用来了附加在文档之前的`MIME`数据类型信息来标识数据类型。在实际应用中，就是`HTTP`请求头中的`Content-Type`字段的设置。

每一个`MIME`都由两部分和一个斜杠组成，前面是数据的大类（`audio、image...`），后面是定义大类里面的具体种类（`midi、png`）。

# 12.常见的请求头

`Content-Type`：请求资源的`MIME`类型。

`Range`：获取部分内容的范围请求。

# 13.内容协商请求头

> 在 [HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP) 协议中，内容协商是这样一种机制，通过**为同一 URI 指向的资源提供不同的展现形式**，可以使用户代理选择与用户需求相适应的最佳匹配（例如，文档使用的自然语言，图片的格式，或者内容编码形式）。

常见的用于内容协商的字段：

| 首部名称          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `Accept`          | [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept) 首部列举了用户代理希望接收的媒体资源的 `MIME` 类型。其中不同的 `MIME `类型之间用逗号分隔，同时每一种 `MIME` 类型会配有一个品质因数（`quality factor`），该参数明确了不同 `MIME` 类型之间的相对优先级。 |
| `Accept-Charset`  | [`Accept-Charset`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Charset)首部用于告知服务器该客户代理可以理解何种形式的字符编码。按照传统，不同地区用户使用的的浏览器会被赋予不同的值。如今 UTF-8 编码已经得到了广泛的支持，成为首选的字符编码类型。[为了通过减少基于配置信息的信息熵来更好地保护隐私信息](https://www.eff.org/deeplinks/2010/01/primer-information-theory-and-privacy)， 大多数浏览器会将 `Accept-Charset` 首部移除。 |
| `Accept-Encoding` | [`Accept-Encoding`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Encoding) 首部明确说明了（接收端）**可以接受的内容编码形式**（所支持的压缩算法）。该首部的值是一个`Q`因子清单（例如 `br, gzip;q=0.8`），用来提示不同编码类型值的优先级顺序。默认值 `identity` 则优先级最低（除非声明为其他优先级）。 |
| `Accept-Language` | [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language) 首部用来提示用户期望获得的自然语言的优先顺序。该首部的值是一个`Q`因子清单（例如 `"de, en;q=0.7"`）。用户代理的图形界面上所采用的语言通常可以用来设置为默认值，但是大多数浏览器**允许设置不同优先级的语言选项**。 |

内容协商技术的三种类型：

- 服务器驱动协商：**由服务器端进行协商。以请求的首部字段为参考，在服务端自动处理。**
- 客户端驱动协商：**由客户端进行内容协商的方式。用户可以从浏览器显示的可选列表中手动选择（用户体验比较好）。**
- 透明协商：**服务器驱动和客户端驱动的结合体，由服务器端和客户端各自进行内容协商的一种方法。**

# 14.说一说状态码？

状态码的职责是当客户端向服务器端发送请求时，**描述返回的请求结果**。借助状态码，用户端可以知道服务器端是正常处理了请求，还是出现了错误。**状态码是`HTTP/1.1`规范的一部分，`RFC`规范定义了响应的五种状态，并为五种状态细分了许多具体的状态码用于根据具体的情况来响应指定的状态码。**也可以自定义状态码，但是最好是根据五大类别的属性来进行定义。

五大类状态码简述：

| 状态码 | 类别             | 原因短语                       |
| ------ | ---------------- | ------------------------------ |
| 1XX    | 信息性状态码     | 接收的请求**正在处理**         |
| 2XX    | 成功状态码       | 请求**正常处理完毕**           |
| 3XX    | 重定向状态码     | 需要进行**附加操作**以完成请求 |
| 4XX    | 客户端错误状态码 | 服务器**无法处理请求**         |
| 5XX    | 服务器错误状态码 | 服务器处理请求**出错**         |

常见状态码：

| 状态码 | 短语                    | 描述                                                         |
| ------ | ----------------------- | ------------------------------------------------------------ |
| ⭐200   | `OK`                    | 状态码 **`200 OK`** 表明**请求已经成功**， 默认情况下状态码为200的响应可以被缓存。 |
| ⭐201   | `Created`               | 在`HTTP`协议中，**`201 Created`** 是一个代表成功的应答状态码，表示**请求已经被成功处理，并且创建了新的资源**。新的资源在应答返回之前已经被创建。**同时新增的资源会在应答消息体中返回**，其地址或者是原始请求的路径，或者是 [`Location`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Location) 首部的值。这个状态码的**常规使用场景是作为 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST) 请求的返回值**。 |
| ⭐204   | `No Content`            | `HTTP` **`204 No Content `**成功状态响应码，表示**该请求已经成功了，但是客户端客户不需要离开当前页面**。默认情况下 `204`响应是可缓存的。使用惯例是，**在 [`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 请求中进行资源更新，但是不需要改变当前展示给用户的页面，那么返回 `204 No Content`。**如果创建了资源，则返回 [`201`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/201) `Created` 。如果应将页面更改为新更新的页面，则应改用 [`200`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/200) 。 |
| ⭐206   | `Partial Content`       | `HTTP` **`206 Partial Content`** 成功状态响应代码**表示请求已成功，并且主体包含所请求的数据区间，该数据区间是在请求的 [`Range`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Range) 首部指定的**。如果只包含一个数据区间，那么整个响应的 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 首部的值为所请求的文件的类型，同时包含 [`Content-Range`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Range) 首部。如果包含多个数据区间，那么整个响应的 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 首部的值为 `multipart/byteranges` ，其中一个片段对应一个数据区间，并提供  [`Content-Range`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Range) 和 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 描述信息。 |
| ⭐301   | `Moved Permanently`     | `HTTP` `301`**` 永久重定向`** 说明**请求的资源已经被移动到了由 [`Location`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Location) 头部指定的`url`上，是固定的不会再改变**。搜索引擎会根据该响应修正。尽管标准要求浏览器在收到该响应并进行重定向时不应该修改`http method`和`body`，但是有一些浏览器可能会有问题。所以最好是在应对[`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET) 或 [`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD) 方法时使用`301`，其他情况使用[`308`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/308) 来替代`301`。 |
| 302    | `Found`                 | `HTTP` `302`**` Found`** 重定向状态码**表明请求的资源被暂时的移动到了由该HTTP响应的响应头[`Location`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Location) 指定的 `URL` 上**。浏览器会**重定向到这个`URL`， 但是搜索引擎不会对该资源的链接进行更新**。在确实需要将重定向请求的方法转换为 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)的场景下，可以使用 [`303`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/303) `See Other`。例如在使用 [`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 方法进行文件上传操作时，需要返回确认信息（例如“你已经成功上传了`xyz`”）而不是上传的资源本身，就可以使用这个状态码。 |
| 303    | `See Other`             | `HTTP` **`303 See Other`** 重定向状态码，通常作为 [`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 或 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST) 操作的返回结果，它**表示重定向链接指向的不是新上传的资源，而是另外一个页面，比如消息确认页面或上传进度页面。而请求重定向页面的方法要总是使用 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)。** |
| ⭐304   | `Not Modified`          | `HTTP` `304`**` 未改变`**说明**无需再次传输请求的内容，也就是说可以使用缓存的内容**。 |
| 307    | `Temporary Redirect`    | `HTTP` `307 Temporary Redirect`，临时重定向响应状态码，表示请求的资源暂时地被移动到了响应的 [`Location`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Location) 首部所指向的 `URL` 上。**状态码 `307` 与 [`302`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/302) 之间的唯一区别在于，当发送重定向请求的时候，`307` 状态码可以确保请求方法和消息主体不会发生变化。**如果使用 `302` 响应状态码，一些旧客户端会错误地将请求方法转换为 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)：也就是说，在`Web`中，如果使用了 `GET` 以外的请求方法，且返回了 `302` 状态码，则重定向后的请求方法是不可预测的；但如果使用 `307` 状态码，之后的请求方法就是可预测的。对于 `GET` 请求来说，两种情况没有区别。 |
| 308    | `Permanent Redirect`    | 在 `HTTP` 协议中， **`308 Permanent Redirect`**（永久重定向）是表示重定向的响应状态码，说明请求的资源已经被永久的移动到了由 [`Location`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Location) 首部指定的`URL`上。**浏览器会进行重定向，同时搜索引擎也会更新其链接（用`SEO`的行话来说，意思是“链接汁”（`link juice`）被传递到了新的 `URL`）。** |
| ⭐400   | `Bad Request`           | `HTTP` **`400 Bad Request`** 响应状态码表示**由于语法无效，服务器无法理解该请求。**（例如，格式错误的请求语法，太大的大小，无效的请求消息或欺骗性路由请求） 客户端**不应该在未经修改的情况下重复此请求**。 |
| ⭐401   | `Unauthorized`          | 状态码 **`401 Unauthorized`** 代表**客户端错误**，指的是**由于缺乏目标资源要求的身份验证凭证，发送的请求未得到满足**。（认证失败，期望客户端重试） |
| ⭐403   | `Forbidden`             | 状态码 **`403 Forbidden`** 代表客户端错误，指的是**服务器端有能力处理该请求，但是拒绝授权访问**。这个状态类似于 [`401`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/401)，但进入该状态后不能再继续进行验证。该访问是**长期禁止的，并且与应用逻辑密切相关（例如不正确的密码使得没有权限访问某一资源）**。（没有权限，不期望客户端重试） |
| ⭐404   | `Not Found`             | 状态码 `404 Not Found` 代表客户端错误，指的是**服务器端无法找到所请求的资源。**返回该响应的链接通常称为坏链（`broken link`）或死链（`dead link`），它们会导向链接出错处理页面。`404` 状态码**并不能说明请求的资源是临时还是永久丢失**。**如果服务器知道该资源是永久丢失，那么应该返回 [`410`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/410) (`Gone`) 而不是 `404 `。** |
| ⭐405   | `Method Not Allowed`    | 状态码 `405 Method Not Allowed` 表明服务器禁止了使用当前 HTTP 方法的请求。 |
| 410    | `Gone`                  | HTTP `410 丢失` 说明请**求的目标资源在原服务器上不存在了，并且是永久性的丢失。**如果不清楚是否为永久或临时的丢失，应该使用[`404`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/404) 。 |
| ⭐500   | `Internal Server Error` | 在`HTTP` 协议中，`500 Internal Server Error` 是表示**服务器端错误的响应状态码，意味着所请求的服务器遇到意外的情况并阻止其执行请求**。这个错误代码是一个通用的“万能”响应代码。有时候，对于类似于 `500 `这样的错误，服务器管理员会更加详细地记录相关的请求信息来防止以后同样错误的出现。 |
| ⭐502   | `Bad Gateway`           | `502 Bad Gateway`是一种`HTTP`协议的服务器端错误状态代码，它表示作为网关或代理角色的服务器，从上游服务器（如`tomcat`、`php-fpm`）中接收到的响应是无效的。`502 `错误通常不是客户端能够修复的，而是需要由途径的`Web`服务器或者代理服务器对其进行修复。由于互联网[协议](https://developer.mozilla.org/zh-CN/docs/Glossary/Protocol)是非常明确的，因此 502 通常意味着（网关和上游服务器）其中一方或双方并没有被正确地，完整地进行编码。 |
| 503    | `Service Unavailable`   | `503 Service Unavailable`是一种`HTTP`协议的服务器端错误状态代码，它表示服务器尚未处于可以接受请求的状态。通常造成这种情况的原因是由于服务器停机维护或者已超载。注意在发送该响应的时候，应该同时发送一个对用户友好的页面来解释问题发生的原因。该种响应应该用于临时状况下，与之同时，在可行的情况下，应该在 [`Retry-After`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Retry-After) 首部字段中包含服务恢复的预期时间。缓存相关的首部在与该响应一同发送时应该小心使用，因为 `503`状态码通常应用于临时状况下，而此类响应一般不应该进行缓存。 |
| 504    | `Gateway Timeout`       | `504 Gateway Timeout`是一种`HTTP`协议的服务器端错误状态代码，表示扮演网关或者代理的服务器无法在规定的时间内获得想要的响应。`504 `错误通常不是在客户端可以修复的，而是需要由途径的`Web`服务器或者代理服务器对其进行修复。 |

[状态码301和302的区别](https://github.com/Ray1993/MyBlog/issues/3)

[304状态码及缓存](https://juejin.cn/post/6844903512946507790)

[401和403的区别](https://www.cnblogs.com/qiqi715/p/12982102.html)

# 15.说一说`HTTP`首部（请求头）有哪些常见的字段？

## 通用首部字段

请求报文和响应报文**双方都会使用的首部**。

## 请求首部字段

从**客户端向服务端发送请求报文时使用的首部**。补充了请求的附加内容、客户端信息、响应内容相关优先级等信息。

## 响应首部字段

从**服务器向客户端返回响应报文时使用的首部**。补充了响应的附加内容，也会要求客户端附加额外的内容信息。

## 实体首部字段

针对请求报文和响应报文的**实体部分使用的首部**。补充了资源内容更新时间等与实体有关的信息。

![image-20220105223752493](https://github.com/NoAlligator/pico/blob/main/img/image-20220105223752493.png?raw=true)

![image-20220105223803330](https://github.com/NoAlligator/pico/blob/main/img/image-20220105223803330.png?raw=true)

![image-20220105223931183](https://github.com/NoAlligator/pico/blob/main/img/image-20220105223931183.png?raw=true)

![image-20220105223941165](https://github.com/NoAlligator/pico/blob/main/img/image-20220105223941165.png?raw=true)

## 端到端首部和逐跳首部的概念

`HTTP`首部字段将定义成**缓存代理**和**非缓存代理**的行为，分成`2`种类型。

- 端到端首部(`End-to-end Header`)：它们被**发送到请求或响应的最终接收者**。响应中的端到端报头必须存储为缓存条目的一部分，并且必须在由缓存条目形成的任何响应中传输。
- 逐跳首部( `Hop-by-hop Header`)：它们只对**单个传输层连接**有意义，并且不由缓存存储或由代理转发。

以下首部为`HTTP/1.1`的逐跳首部字段：

●`Connection`
●`Keep-Alive`
●`Proxy-Authenticate`
●`Proxy-Authorization`
●`Trailer`
●`TE`
●`Transfer-Encoding`
●`Upgrade`

## 请求头`Connection`字段？

`Connection`首部字段具备如下两个作用：

- 控制**不再转发给代理**的首部字段；
- 管理**持久连接**；

### 控制转发

![image-20220106160846915](https://github.com/NoAlligator/pico/blob/main/img/image-20220106160846915.png?raw=true)

### 管理持久连接

![image-20220106160945841](https://github.com/NoAlligator/pico/blob/main/img/image-20220106160945841.png?raw=true)

## 请求头`Date`字段

> 首部`Date`字段表明创建`HTTP`报文的日期和时间。

```http
HTTP/1.1协议使用在RFC1123中规定的日期时间的格式，如下：
Date: Tue, 03 Jul 2012 04:40:59 GMT
之前的HTTP协议版本使用RFC850定义的更格式：
Date: Tue, 03-Jul-12 04:40:59 GMT
```

## 请求头`Trailer`字段？

**`Trailer`** 是一个**响应首部**，允许发送方在**分块发送的消息后面添加额外的元信息**，这些元信息可能是随着消息主体的发送动态生成的，比如消息的完整性校验，消息的数字签名，或者消息经过处理之后的最终状态等。

以下用例中，指定首部字段 `Trailer` 的值为 `Expires` ，在报文主体之后（分块长度0之后）出现的首部字段 `Expires`。

```
HTTP/1.1200 OK
Date:Tue,03Jul201204:40:56 GMT
Content-Type: text/html
···
Transfer-Encoding: chunked
Trailer:Expiress
···
···（报文主体）···
0
Expires:Tue,28Sep200423:59:59 GMT
```

## 请求头`Transfer-Encoding`字段？

首部字段`Transfer-Encoding`字段规定了**传输报文主体时采用的编码方式**。`HTTP/1.1`的传输编码方式仅对分块传输编码有效。

## 请求头`Upgrade`字段

首部`Upgrade`字段用于检测`HTTP`协议及其他协议**是否可使用更高的版本进行通信**，其参数可以用来指定一个完全不同的通信协议。

![image-20220106164258626](https://github.com/NoAlligator/pico/blob/main/img/image-20220106164258626.png?raw=true)![image-20220106164355268](https://github.com/NoAlligator/pico/blob/main/img/image-20220106164355268.png?raw=true)



## 请求头`Via`字段

> `Via` 是一个通用首部，是由代理服务器添加的，适用于正向和反向代理，在**请求和响应首部中均可出现**。这个消息首部可以用来**追踪消息转发情况**，**防止循环请求**，以及识别在请求或响应传递链中消息发送者对于协议的支持能力。

使用首部`Via`是为了**追踪客户端与服务器之间的请求和响应报文的传输路径**。报文经过代理或网关时，会**先在首部字段`Via`中附加该服务器的信息，然后再进行转发**。首部字段`Via`不仅用于追踪报文的转发，**还可避免请求回环的发生**。所以必须在经过代理时附加该首部字段内容。

![image-20220106174832773](https://github.com/NoAlligator/pico/blob/main/img/image-20220106174832773.png?raw=true)



上图用例中，在经过代理服务器`A`时，`Via`首部添加了相应的代理服务器信息。行头的`1.0`是指代理服务器上应用的`HTTP`协议版本。接下来经过代理服务器`B`亦是如此，在`Via`首部附加服务器信息，也可以增加一个新的`Via`首部写入服务器信息。

`Via`首部为了**追踪传输路径**，所以经常会和`TRACE`方法一起使用。比如，**代理服务器接收到`TRACE`方法发送过来的请求（其中`Max-Forward: 0`）时，代理服务器就不能转发该请求了。这种情况下，代理服务器会将自身的信息附加到`Via`首部后，返回该请求的响应。**

## 请求头`Warning`字段

`HTTP/1.1`的`Warning`首部是从`HTTP/1.0`的响应首部（`Retry-After`）演变过来的。**该首部通常会告知用户一些缓存相关的问题警告**。

`Warning`首部的格式如下：

```
Warning: [警告码] [警告的主机: 警告码] "[警告内容]" ([日期时间])
```

![image-20220106180553487](https://github.com/NoAlligator/pico/blob/main/img/image-20220106180553487.png?raw=true)

## 请求的请求头

### `Accept`

**`Accept`** 请求头用来告知（服务器）客户端**可以处理的内容类型**，这种内容类型用[`MIME类型`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)来表示。借助[内容协商机制](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation), **服务器可以从诸多备选项中选择一项进行应用，并使用 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)来表示**。借助[内容协商机制](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation), 服务器可以从诸多备选项中选择一项进行应用，并使用 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 应答头通知客户端它的选择。浏览器会基于请求的上下文来为这个请求头设置合适的值，比如获取一个CSS层叠样式表时值与获取图片、视频或脚本文件时的值是不同的。

```http
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```

若想要给显示的媒体类型增加优先级，**则使用`q=`来额外表示权重值，用分号`;`进行分隔。**权重值`q`的范围是`0 ~ 1`（可以精确到小数点后三位），且`1`为最大值。不指定权重`q`值时，默认权重为`q=1.0`。**当服务器提供多种内容时，将会首先返回权重值最高的媒体类型。**

### `Accept-Charset`

被废弃，原因：

`UTF-8`得到了良好的支持，是字符编码的首选。为了通过更少的基于配置的熵来保证更好的隐私，所有的浏览器都省略了`Accept-Charset`报头。`Chrome、Firefox、Internet Explorer、Opera、Safari`都放弃了这个头文件。

### `Accept-Encoding`

`HTTP` 请求头 **`Accept-Encoding`** 会将客户端能够理解的内容编码方式——通常是某种压缩算法——进行通知（给服务端）。通过内容协商的方式，服务端会选择一个客户端提议的方式，使用并在**响应头** [`Content-Encoding`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Encoding) 中通知客户端该选择。也可以采用权重`q`值来表示相对优先级，这一点与首部字段`Accept`相同。另外也可以使用`*`作为通配符，指定任意的编码格式。

### `Accept-Language`

**`Accept-Language`** 请求头允许客户端声明它可以理解的自然语言，以及优先选择的区域方言。借助[内容协商机制](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation)，服务器可以从诸多备选项中选择一项进行应用， 并使用 [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language) 应答头通知客户端它的选择。

### `Authorizarion`

该首部字段是用来**告知服务器，用户代理的认证信息（证书值）**。通常，想要通过服务器认真的用户代理会在接收到返回的`401`状态码响应后，把首部字段`Authorization`加入请求中。**共用缓存在接收到含有`Authorization`首部字段的请求时的操作处理会略有差异。**

语法如下：

```http
Authorization: <type> <credentials>
```

### `Host`

首部字段`Host`会告知服务器，**请求的资源所处的互联网主机名和端口号**。`Host`首部在`HTTP/1.1`规范中是**唯一一个必须被包含在请求内的首部字段**。（注意，**仅仅在`HTTP/1.1`是唯一必须的**）。

请求被发送至服务器时，请求中的主机名会用`IP`地址直接替换解决。但如果这时，相同的`IP`地址下部署运行着**多个域名**，那么**服务器就会无法理解究竟是那个域名对应的请求**。因此，就需要使用首部字段`Host`来明确指出**请求的主机名**。若服务器未设定主机名，那**直接发送一个空值即可**。

### `If-Range`

`If-Range`告知服务器若指定的`If-Range`字段值（`ETag`值或者时间）和请求资源的`ETag`值或事件相一致时，则作为范围请求处理。反之，返回全体资源。

![image-20220106184422528](https://github.com/NoAlligator/pico/blob/main/img/image-20220106184422528.png?raw=true)

如果不适用首部字段`If-Range`发送请求的情况：**服务器的资源如果更新，那客户端持有资源中的一部分也会随之无效，当然，范围请求作为前提是无效的。**这时，服务器会暂且以状态码`412`作为响应返回，其目的是催促客户端再次发送请求，这样一来，**与使用首部字段`If-Range`相比，需要多一次请求消耗。**

![image-20220106184446286](https://github.com/NoAlligator/pico/blob/main/img/image-20220106184446286.png?raw=true)

### `Max-Forward`

![image-20220106184955551](https://github.com/NoAlligator/pico/blob/main/img/image-20220106184955551.png?raw=true)

通过`TRACE`方法或`OPTIONS`方法，**发送包含首部字段`Max-Forwards`的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。**服务器在往下一个服务器转发请求之前，`Max-Forwards`的值减
`1`后重新赋值。当服务器接收到`Max-Forwards`值为`0`的请求时，则不再进行转发，而是直接返回响应。使用`HTTP`协议通信时，请求可能会经过代理等多台服务器。**途中，如果代理服务器由于某些原因导致请求转发失败，客户端也就等不到服务器返回的响应了。对此，我们无从可知。可以灵活使用首部字段`Max-Forwards`，针对以上问题产生的原因展开调查。**由于当`Max-Forwards`字段值为`0`时，服务器就会立即返回响应，**由此我们至少可以对以那台服务器为终点的传输路径的通信状况有所把握**。

![image-20220106185213313](https://github.com/NoAlligator/pico/blob/main/img/image-20220106185213313.png?raw=true)

### `Range`

```
Range: bytes=5001-10000
```

对于**只需获取部分资源的范围请求，包含首部字段`Range`即可告知服务器资源的指定范围**。上面的示例表示请求获取从第 `5001`字节至第`10000`字节的资源。接收到附带 `Range` 首部字段请求的服务器，会在处理请求之后返回状态码为 `206 Partial Content`的响应。无法处理该范围请求时，则会返回状态码`200 OK`的响应及全部资源。

### `User-Agent`

![image-20220106185501173](https://github.com/NoAlligator/pico/blob/main/img/image-20220106185501173.png?raw=true)

首部字段`User-Agent`会将**创建请求的浏览器和用户代理名称等信息传达给服务器**。

## 响应的响应头

### `Accept-Ranges`

### ![image-20220106185707924](https://github.com/NoAlligator/pico/blob/main/img/image-20220106185707924.png?raw=true)`Age`

![image-20220106185743137](https://github.com/NoAlligator/pico/blob/main/img/image-20220106185743137.png?raw=true)

### `ETag`

![image-20220106185820999](https://github.com/NoAlligator/pico/blob/main/img/image-20220106185820999.png?raw=true)

![image-20220106185830719](https://github.com/NoAlligator/pico/blob/main/img/image-20220106185830719.png?raw=true)

### `Location`

使用首部字段`Location`可以**将响应接收方引导至某个与请求`URI`位置不同的资源**。基本上，该字段会配合`3xx：Redirection`的响应，提供重定向的`URI`。几乎所有的浏览器在接收到包含首部字段`Location`的响应后，都会强制性地尝试对已提示的重定向资源的访问。

### `Retry-After`

首部字段`Retry-After`告知**客户端应该在多久之后再次发送请求**。主要配合状态码`503 Service Unavailable`响应，或`3xx Redirect`响应一起使用。字段值可以指定为具体的日期时间（`Wed, 04 Jul 2012 06:34:24 GMT`等格式），**也可以是创建响应后的秒数**。

### `Server`

首部字段`Server`告知客户端当前服务器上安装的`HTTP`服务器应用程序的信息。不单单会标出服务器上的软件应用名称，**还有可能包括版本号和安装时启用的可选项**。

## 实体首部字段

### `Allow`

首部字段`Allow`用于**通知客户端能够支持`Request-URI`指定资源的所有`HTTP`方法**。当服务器接收到不支持的`HTTP`方法时，会以状态码`405 Method Not Allowed`作为响应返回。与此同时，**还会把所有能支持的`HTTP`方法写入首部字段`Allow`后返回。**

`Content-Encoding` `Content-Language` `Content-Encoding` `Content-Length` `Content-Location` `Content-MD5` `Content-Range` `Content-Type` `Expire` `Last-Modified`。

# 16.`HTTP`的缺点以及`HTTPS`的基本概念？

## [HTTPS最详细的参考](https://zhuanlan.zhihu.com/p/43789231)

## `HTTP`的缺点

- 通信**使用明文**（协议本身不具备加密），可能会被窃听。
- **协议本身不验证通信方身份**，因此有可能遭遇伪装。
- 无法证明报文的完整性，因此**有可能已遭篡改**。

## `HTTP`的隐患

- 无法确定请求发送至目标的`Web`服务器是否是按真实意图返回响应的那台服务器。**有可能是已伪装的 `Web`服务器**。（源服务器不确定性）；
- 无法确定响应返回到的客户端是否是按真实意图接收响应的那个客户端。有可能是已伪装的客户端。（客户端不确定性）；
- 无法确定正在通信的对方是否具备访问权限。因为某些`Web`服务器上保存着重要的信息，只想发给特定用户通信的权限。（权限不确定性）；
- 无法判断请求是出自何方、出自谁手；
- 即使是无意义的请求也会照单全收，因为部分资源的请求必须是面向所有用户的，无法阻止海量请求下的`Dos`攻击。

## 加密方式与`HTTPS`

### 通信加密（`HTTPS`）

一种方式就是将**通信加密**。`HTTP`协议中没有加密机制，但**可以通过和`SSL`（`Secure Socket Layer`，安全套接层）或`TLS`（`Transport Layer Security`，安全层传输协议）的组合使用，加密`HTTP`的通信内容。**用`SSL`建立安全通信线路之后，就可以在这条线路上进行`HTTP`通信了。与`SSL`组合使用的 `HTTP`被称为`HTTPS`（`HTTP Secure`，超文本传输安全协议）或`HTTP over SSL`。

### 内容加密

将`HTTP`报文里所含的内容进行加密处理（仍有被篡改风险）。

## `HTTPS`的证书

`SSL`不仅提供加密处理，还使用了一种被称为证书的手段，**可用于用户端来确认请求的目标服务器是否可靠**（可靠的网站并开启了`HTTPS`加密的网站会提供相应的证书）。

## 中间人攻击

请求在响应或者传输途中，遭攻击者拦截并篡改内容的攻击称为**中间人攻击**。为了有效防止这种攻击，有必要使用`HTTPS`。**`SSL`提供认证和加密处理及摘要功能。**

## 什么是`HTTPS`？

`HTTPS = HTTP + 加密处理 + 认证 + 完整性保护`

![image-20220106193616513](https://github.com/NoAlligator/pico/blob/main/img/image-20220106193616513.png?raw=true)

## `HTTPS`是怎么实现的？

`HTTPS`**并非是应用层的一种新协议**。只是`HTTP`**通信接口部分用`SSL`（`Secure Socket Layer`）和`TLS`（`Transport Layer Security`）协议代替而已。**通常，`HTTP`直接和`TCP`通信。当使用`SSL`时，则演变成先和`SSL`通信，再由`SSL`和`TCP`通信了。简言之，所谓`HTTPS`，其实就是身披`SSL`协议这层外壳的`HTTP`。

## `HTTPS`的两种加密方式？

- 对称密匙加密： **加密和解密**同用一个密匙。
- 不对称加密系统：使用一对非对称的密匙。一把叫做**私有密钥**，另一把叫做**公开密钥**。私有密钥不能让其他方知道，而公开密钥可以随意发布，用于进行加密。发送密文的一方使用对方的公开密钥进行加密处理，对方收到被加密的信息后，再使用自己的私有密钥进行解密。利用这种方式，不需要发送用来解密的私有密钥，也不必担心密钥被攻击者窃听而盗走。

`HTTPS`采用**共享密钥加密和公开密钥加密**两者并用的**混合加密机制**（公开密钥加密系统）。若密钥需要实现**安全交换**，那么仅使用公开密钥来通信。但是公开密钥加密与共享密钥相比，其处理速度要慢。

## `HTTPS`之`SSL、TLS`建立连接的过程

> - TCP 协议 — 通信双方通过三次握手建立 TCP 连接；
> - TLS 协议 — 通信双方通过四次握手建立 TLS 连接；
>
> 综上，总共七次握手，此外：**一般情况下，不管 TLS 握手次数如何，都得先经过 TCP 三次握手后才能进行**，因为 HTTPS 都是基于 TCP 传输协议实现的，得先建立完可靠的 TCP 连接才能做 TLS 握手的事情。

[参考](https://zhuanlan.zhihu.com/p/43789231)

[为什么 HTTPS 需要 7 次握手以及 9 倍时延](https://draveness.me/whys-the-design-https-latency/)

[给面试官上一课：HTTPS是先进行TCP三次握手，再进行TLS四次握手 ...](https://www.hackbase.net/article-262445-1.html)

[TLS1.3基本原理](https://www.itread01.com/content/1545708792.html)

![image-20220106232039637](https://github.com/NoAlligator/pico/blob/main/img/image-20220106232039637.png?raw=true)

![img](https://pic2.zhimg.com/v2-22c7091dfa8fb587d9f37d7c3edc387d_r.jpg)

1. 客户端**发送`Client Hello`报文开始`SSL`通信**。报文中包含客户端支持的`SSL`的制定版本、加密组件（`Cipher Suite`）列表（所使用的加密算法及密钥长度等）。
2. 服务器可以进行`SSL`通信时，**回以`Server Hello`报文作为应答**。和客户端一样，在报文中包含`SSL`版本以及加密组件。服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的。
3. 服务器**发送`Certificate`报文**，报文中包含公开密钥证书。
4. 服务器**发送`Server Hello Done`报文通知客户端**，最初阶段的`SSL` 握手协商结束。
5. `SSL`第一次握手结束之后，**客户端以`Client Key Exchange`报文作为回应**。报文中**包含通信加密中使用的一种被称为`Pre-master secret`的随机密码串**。该报文**已用步骤`3`中的公开密钥进行加密**。
6. 客户端**继续发送`Change Cipher Spec`报文**。该报文会提示服务器：**在次之后的通信会采用`Pre-master secret`密钥加密。**
7. 客户端**发送`Finished`报文**。该报文包含**连接至今全部报文的整体校验值**，这次**握手协商能否成功**，要以**服务器是否能够正确解密该报文作为判定标准**。
8. 服务器**同样发送`Change Cipher Spec`报文**。
9. 服务器**同样发送`Finshed`报文**。
10. 服务器和客户端的`Finshed`报文交换完毕之后，**`SSL`连接就算建立完成**。当然，通信会受到`SSL`的保护。从此处开始进行应用层的通信。
11. 应用层协议通信，及发送`HTTP`响应。
12. 最后由客户端断开连接。**断开连接时，发送`close_notify`报文**（这步之后会发送`TCP FIN`报文来关闭于`TCP`的通信）。
13. 在以上流程中，应用层发送数据时会附加一种叫做`MAC(Message Authetication Code)`的报文摘要。`MAC`报文摘要能够查知报文是否遭到篡改，从而保护报文完整性。

![image-20220107110131244](https://github.com/NoAlligator/pico/blob/main/img/image-20220107110131244.png?raw=true)

精简版的核心流程：

1. 客户端主动发起`SSL`握手协商，服务端发送证书；
2. 客户端验证服务端发送的证书；
3. 客户端生成随机数（对称加密的密钥）；
4. 客户端生成握手信息（生成随机数加密握手信息及哈希 + 公钥加密随机数）；
5. 服务端接收随机数加密的信息，并解密得到随机数，验证握手信息是否被篡改；
6. 服务端同样使用客户端生成的随机字符串加密握手信息和握手信息`hash`值发回给到客户端；
7. 客户端验证服务端发送回来的握手信息，完成握手；
8. 服务端和客户端开始用随机数进行对称加密通信。

## 为什么需要客户端生成随机密码串？

客户端生成的随机密钥，用来进行**对称加密**：

首先，客户端在验证证书无误之后生成该随机数，并用服务端发送的公钥进行加密，这样使得**只有掌握了私钥的服务器才能够解开密码**。

用证书中的签名`hash`算法取握手信息的`hash`值，然后**用生成的随机数对[ 握手信息和握手信息的`hash`值 ]进行加密，然后用公钥将随机数进行加密后，一起发送给服务端**。其中计算握手信息的`hash`值，目的是为了保证传回到服务端的握手信息没有被篡改。（随机数加密、公匙加密）

服务端收到客户端传回来的用随机数加密的信息后，**先用私钥解密随机数，然后用解密得到的随机数解密握手信息，获取握手信息和握手信息的`hash`值，计算自己发送的握手信息的`hash`值，与客户端传回来的进行对比验证**。如果验证无误，**同样使用随机字符串加密握手信息和握手信息`hash`值发回给到客户端**。

客户端收到服务端发送过来的握手信息后，用开始自己生成的随机数进行解密，验证被随机数加密的握手信息和握手信息`hash`值。验证无误后，握手过程就完成了，从此服务端和客户端就开始用那串随机数进行对称加密通信了。

[参考教程](https://zhuanlan.zhihu.com/p/107573461)

## `HTTPS`使用`SSL`会导致通信变慢吗？

`HTTPS`也存在一些问题，当使用`SSL`时，他的处理速度会变慢：

- 通信慢：必须进行`SSL`通信，因此整体上处理通信量不可避免会增加。
- 处理速度慢：`SSL`必须进行加密处理，在服务器和客户端都需要进行加密的运算处理。从结果上讲，比起`HTTP`会更多地消耗服务器和客户端的硬件资源，导致负载增强。

可以使用`SSL`加速器这种（专用服务器）硬件来改善该问题。该硬件为`SSL`通信专用硬件，相对软件来讲，能够提高数倍`SSL`的计算速度。仅在`SSL`处理时发挥`SSL`加速器地功效，以分担负载。

# 17.说一说`WebSocket`？

`WebSocket`是`HTML5`开始提供的一种在单个`TCP `连接上进行**全双工通讯**的协议。`WebSocket`是另一个独立的基于`TCP`协议的通信协议，不是`HTTP`协议的增强功能，但是会依靠`HTTP`协议。

在`WebSocket`之前，实现实时更新服务器数据的功能都是**通过轮询的方式**来实现的，这些实现方式有一个共同点，**都是由客户端主动向服务器端发起请求**，不难想象，轮询的方式就有一个轮询时间间隔的问题，间隔长了服务端的数据不能及时更新到客户端，间隔短了就会有许多无用的请求，增加服务器压力，浪费服务器以及带宽资源，`WebSocket`可以满足这个需求。

![img](https://github.com/NoAlligator/pico/blob/main/img/bg2017051502.png?raw=true)

一旦 **`Web` 服务器与客户端**之间建立起 `WebSocket` 协议的通信连接，之后所有的通信都**依靠这个专用协议进行**。通信过程中可互相发送`JSON、XML、HTML`或图片等任意格式的数据。由于是**建立在`TCP`长连接基础上的协议，因此连接的发起方仍是客户端**，而一旦确立`WebSocket`通信连接，**不论服务器还是客户端，任意方都可直接向对方发送报文**。

## `WebSocket`协议的主要特点

1. 双向推送功能：服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种；
2. 减少通信量：只要建立起`WebSocket`连接，就希望一致保持连接状态。和`HTTP`相比，不但每次建立连接的总开销减少，而且**由于`WebSocket`的首部信息很小，通信量也相应减少**；
3. 建立在`TCP`协议之上，服务器端的实现比较容易；
4. 与 `HTTP`协议有着良好的兼容性。默认端口也是`80`和`443`，并且握手阶段采用 `HTTP` 协议，因此握手时不容易屏蔽，能通过各种 `HTTP` 代理服务器；
5. 可以发送文本，也可以发送二进制数据；
6. 没有同源限制，客户端可以与任意服务器通信；
7. 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是`URL`；

![img](https://github.com/NoAlligator/pico/blob/main/img/bg2017051503.jpg?raw=true)

## `WebSocket`通信的建立

简化流程：

1. 客户端发送一个 `upgrade` 的`HTTP`请求，请求协议升级为`WebSocket`。
2. 服务器返回一个`HTTP 101 Switching Protocol`，代表请求切换协议成功。
3. 服务端和客户端通过`WebSocket`协议进行双工通信。

![握手过程](https://www.timefly.cn/learn-websocket-protocol-1/ws-shake-hand.png)

细节：

1. 首先客户端会发送一个握手包**请求协议升级**。握手包的报文格式必须符合`HTTP`报文格式的规范。并且遵循以下条件：

- 方法必须为`GET`方法
- `HTTP`版本不能低于`1.1`
- 必须包含`Host`头部
- 必须包含`Origin`头部
- 必须包含`Upgrade`头部，值必须为`websocket`
- 必选包含`Connection`头部，值必须为`Upgrade`
- 必须包含`Sec-WebSocket-Key`头部，值是一个`Base64`编码的`16`字节随机字符串
- 必须包含`Sec-WebSocket-Version`头部，值必须为`13`

2. 服务端**验证客户端的握手包符合规范之后也会发送一个握手包给客户端**。格式如下：

- 必须包含`Upgrade`头部，值必须为`websocket`
- 必选包含`Connection`头部，值必须为`Upgrade`
- 必须包含一个`Sec-Websocket-Accept`头部，值是根据如下规则计算的：
  - 首先将一个固定的字符串`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`拼接到`Sec-WebSocket-Key`对应值的后面。
  - 对拼接后的字符串进行一次`SHA-1`计算
  - 将计算结果进行`Base-64`编码

3. **客户端收到服务端的握手包之后，验证报文格式时候符合规范**，以`2.`中同样的方式计算`Sec-WebSocket-Accept`并与服务端握手包里的值进行比对。
4. 建立`WebSocket`连接。
5. 以上任意一步不通过都无法建立`WebSocket`连接。

![image-20220107125356076](https://github.com/NoAlligator/pico/blob/main/img/image-20220107125356076.png?raw=true)

[WebSocket的RFC标准](https://datatracker.ietf.org/doc/html/rfc6455#section-4.1)

# 18.`Referer`和`Origin`的区别？

[参考](https://www.dazhuanlan.com/mrpuchi/topics/1159115)

`Referer` 指示了请求来自于哪个具体页面，包含**服务器名和路径的详细`URL`**，浏览器自动添加到`HTTP`请求`Header`中，无需手动设置。`Referer` 首部**可能暴露用户的浏览历史，涉及到用户的隐私问题。**

`Origin` 指示了请求来自于哪个站点，只有服务器名，**不包含路径信息**，浏览器自动添加到`HTTP`请求 `Header` 中，无需手动设置。跨域请求会自动携带`Origin`。

因此`origin`较`referer`更安全，多用于防范`CSRF`攻击。

# 19.`PUT`、`POST`、`PATCH`的区别？

## 幂等方法

**HTTP 幂等方法是指无论调用多少次都不会有不同结果的 HTTP 方法。**它无论是调用一次，还是十次都无关紧要。结果仍应相同。再次强调， **它只作用于结果而非资源本身。**它仍可能被操纵（如一个更新的 timestamp），提供这一信息并不影响（当前）资源的表现形式。

`POST`：非幂等地创建整个资源。`Create`

`PUT`：幂等地创建或更新整个资源。`Create or Update`

`PATCH`：非幂等性地更新**局部**资源。`Update field`（语义上是非幂等性的，但是设计上也可以设置成幂等性的`PATCH`请求）。

| HTTP Method | Idempotent（幂等性） | Safe |
| :---------- | :------------------- | :--- |
| OPTIONS     | yes                  | yes  |
| GET         | yes                  | yes  |
| HEAD        | yes                  | yes  |
| PUT         | yes                  | no   |
| POST        | no                   | no   |
| DELETE      | yes                  | no   |
| PATCH       | no                   | no   |

# 20.HTTP的历史

## HTTP1.0和HTTP1.1的一些区别

HTTP1.0最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在1999年才开始广泛应用于现在的各大浏览器网络请求中，同时HTTP1.1也是当前使用最为广泛的HTTP协议。 主要区别主要体现在

1. **缓存处理**（增加了`Etag`），在HTTP1.0中主要使用header里的If-Modified-Since，Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since，If-Match，If-None-Match等更多可供选择的缓存头来控制缓存策略。
2. **带宽优化及网络连接的使用**（支持部分传输），HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（`Partial Content`），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
3. **错误通知的管理**（新的语义化状态码），在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
4. **Host头处理**（支持主机名），在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
5. **长连接**（默认开启了长连接），HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

## HTTP2.0和HTTP1.X相比的新特性

**二进制分帧。**在`HTTP/2`中，在应用层（`HTTP2.0`）和传输层（`TCP`或者`UDP`）之间加了一层：二进制分帧层。在二进制分帧层中， `HTTP/2`会将所有传输的信息分割为**更小的消息和帧**（`frame`），并对它们采用二进制格式的编码。这种单连接多资源的方式，**减少了服务端的压力，使得内存占用更少，连接吞吐量更大。**而且，`TCP`连接数的减少**使得网络拥塞状况得以改善，同时慢启动时间的减少，使拥塞和丢包恢复速度更快**。

**多路复用。**多路复用允许同时通过单一的`HTTP/2.0`连接发起**多重的请求-响应消息**。在`HTTP1.1`协议中，**浏览器客户端在同一时间，针对同一域名下的请求有一定数量的限制，超过了这个限制的请求就会被阻塞。**而多路复用允许同时通过单一的`HTTP/2.0`连接**发起多重的“请求-响应”消息**。`HTTP/2.0`的请求的`TCP`的连接一旦建立，后续请求以`stream`的方式发送。每个`stream`的基本组成单位是`frame`（二进制帧）。客户端和服务器可以把`HTTP`消息**分解为互不依赖的帧，然后乱序发送**，最后再在另一端把它们**重新组合**起来。

**头部压缩。**`HTTP/1.1`的头部带有大量信息，而且每次都要重复发送。`HTTP/2`为了减少这部分开销，采用了`HPACK`头部压缩算法对`header`进行压缩。

**服务端推送。**当用户的浏览器和服务器在建立连接后，服务器主动将一些资源推送给浏览器并缓存起来的机制。有了缓存，当浏览器想要访问已缓存的资源的时候就可以直接从缓存中读取。

[参考](https://mp.weixin.qq.com/s/GICbiyJpINrHZ41u_4zT-A?)
