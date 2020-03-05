>在阅读极客时间《透视http协议》中的学习笔记，好记性不如烂笔头。

## http是什么
http是**超文本传输协议**,要深入的理解就得把定义拆分出来逐个击破.
1. 超文本：表示`HTTP`传输的不是 `TCP/UDP` 这些底层协议里被切分的杂乱无章的二进制包`（datagram）`，而是完整的、有意义的数据，可以被浏览器、服务器这样的上层应用程序处理。
2. 传输：传输的条件是需要两方进行交流传输，http是一个在计算机世界里专门用来在两点之间传输数据的约定和规范。
3. 协议：协议指的是一种规范，双方都必须遵守，不然的话协议不生效。

## “四层”和“七层”
`TCP/IP` 协议实际上是一系列网络通信协议的统称，`tcp`处于传输层,`ip`处于网络层，其余层次也十分重要，这会在后面讲到。

`ip`是解决寻址和路由问题，以及如何在两点间传送数据包。`tcp`在`ip`之上，基于 IP 协议提供可靠的、字节流形式的通信，是 HTTP 协议得以实现的基础。

### TCP/IP 网络分层模型

`TCP/IP` 协议总共有四层，就像搭积木一样，每一层需要下层的支撑，同时又支撑着上层，任何一层被抽掉都可能会导致整个协议栈坍塌.  
1. 链接层
2. 网络层，ip层就是在这里
3. 传输层，tcp就是在这里，还有tcp的另外一个伙伴udp
4. 应用层，http就是在这里

### OSI 网络分层模型
1. 第一层：物理层，网络的物理形式，例如电缆、光纤、网卡、集线器等等；
2. 第二层：数据链路层，它基本相当于 TCP/IP 的链接层；
3. 第三层：网络层，相当于 TCP/IP 里的网际层；
4. 第四层：传输层，相当于 TCP/IP 里的传输层；
5. 第五层：会话层，维护网络中的连接状态，即保持会话和同步；
6. 第六层：表示层，把数据转换为合适、可理解的语法和语义；
7. 第七层：应用层，面向具体的应用传输数据。

## 域名
相对于访问服务器的ip地址而言，用户更喜欢访问一串相对好几的名字，这就是域名的产生。  

### 域名的形式
1. 根域名：我们常见的`.com` `.com.cn` `.org`都是根域名.
2. 顶级域名：我们可以买到的域名如博客地址`carrotwu.com` `baidu.com`等
3. 二级域名：...依次类推，没在前面加一个前缀就加上一级。

### 域名的解析
访问一个域名时，如果没有缓存的话会从根域名开始依次向上直到找到相应的服务器ip地址，这一过程叫做域名解析。  
域名解析会耗费大量的时长，所以可以通过**缓存**的思路去进行优化。大致过程是：
** 浏览器缓存->操作系统缓存->hosts->dns->根域名->顶级域名....持续下去**

## 标准请求方法
目前 HTTP/1.1 规定了八种方法，单词都必须是大写的形式。
1. GET：获取资源，可以理解为读取或者下载数据；
2. HEAD：获取资源的元信息；
3. POST：向资源提交数据，相当于写入或上传数据；
4. PUT：类似 POST；
5. DELETE：删除资源；
6. CONNECT：建立特殊的连接隧道；
7. OPTIONS：列出可对资源实行的方法；
8. TRACE：追踪请求 - 响应的传输路径。

## URI 和 URL
需要获取指定服务器的文件或者请求需要用到URI 和 URL。 URI，也就是统一资源标识符。URL，统一资源定位符。两者之间差距不大URL只是URI的一种具体表现形式。

### URI 的格式
URI 本质上是一个字符串，这个字符串的作用是唯一地标记资源的位置或者名字.  
下面的这张图显示了 URI 最常用的形式，由 `scheme`、`host:port`、`path` 和 `query` 四个部分组成，但有的部分可以视情况省略。    
![alt](http://img.carrotwu.com/Fk8saVU2ODPqvHIgbN00LIPBCFpn)  
1. scheme 叫“方案名”或者“协议名”，表示资源应该使用哪种协议来访问(`http` 和 `https`)；
2. “host:port”表示资源所在的主机名和端口号；
3. path 标记资源所在的位置；
4. query 表示对资源附加的额外要求

## 相应状态码
服务器通过http获取客户端请求的数据之后，会返回一个数据报文给客户端，不同的请求结果会对应不同的状态码.  

1. 1××：提示信息，表示目前是协议处理的中间状态，还需要后续的操作；
2. 2××：成功，报文已经收到并被正确处理；
3. 3××：重定向，资源位置发生变动，需要客户端重新发送请求；
4. 4××：客户端错误，请求报文有误，服务器无法处理；
5. 5××：服务器错误，服务器在处理请求时内部发生了错误。

### 一些常用需要记住的状态码

1. “200 OK”是最常见的成功状态码，表示一切正常，服务器如客户端所期望的那样返回了处理结果，如果是非 HEAD请求，通常在响应头后都会有 body 数据。
2. “204 No Content”是另一个很常见的成功状态码，它的含义与“200 OK”基本相同，但响应头后没有 body 数据。所以对于 Web 服务器来说，正确地区分 200 和 204是很必要的。
3. “301”永久重定向, “302”临时重定向.
4. “304 Not Modified” 是一个比较有意思的状态码，它用于 If-Modified-Since 等条件请求，表示资源未修改，用于缓存控制。它不具有通常的跳转含义，但可以理解成“重定向已到缓存的文件”（即“缓存重定向”）。
5. 401代表需要登录, 403代表没有响应的权限, 404查找到响应的文件, 405请求方法不合理。

## http的实体数据
在http的请求头中，可以通过设置响应的类别（`content-type`）来声明需要获取的文件格式.  
1. `text`：即文本格式的可读数据，我们最熟悉的应该就是`text/html` 了，表示超文本文档，此外还有纯文本`text/plain`、样式表 `text/css` 等。
2. `image`：即图像文件，有 `image/gif`、`image/jpeg`、`image/png` 等。
3. `audio/video`：音频和视频数据，例如 `audio/mpeg`、`video/mp4` 等。
4. `application`：数据格式不固定，可能是文本也可能是二进制，必须由上层应用程序来解释。常见的有`application/json`，`application/javascript`、`application/pdf` 等，另外，如果实在是不知道数据是什类型，像刚才说的“黑盒”，就会是`application/octet-stream`，即不透明的二进制数据。

HTTP在传输时为了节约带宽，有时候还会压缩数据，为了不要让浏览器继续“猜”，还需要有一个“`Encoding type`”，告诉数据是用的什么编码格式，这样对方才能正确解压缩，还原出原始的数据。  
1. gzip：GNU zip 压缩格式，也是互联网上最流行的压缩格式；
2. deflate：zlib（deflate）压缩格式，流行程度仅次于gzip；
3. br：一种专门为 HTTP 优化的新压缩算法（Brotli）。

## http传输大文件的方法
1. 压缩 HTML 等文本文件是传输大文件最基本的方法；
2. 分块传输可以流式收发数据，节约内存和带宽，使用响应头字段“`Transfer-Encoding: chunked`”来表示，分块的格式是 16 进制长度头 + 数据块；
3. 范围请求可以只获取部分数据，即“分块请求”，实现视频拖拽或者断点续传，使用请求头字段“`Range`”和响应头字段“`Content-Range`”，响应状态码必须是 206；
4. 也可以一次请求多个范围，这时候响应报文的数据类型是“`multipart/byteranges`”，body 里的多个部分会用`boundary` 字符串分隔。

## HTTP的连接管理
![alt](http://img.carrotwu.com/FkZXP3semwjK-wh1bMwR6zWfDjma)  

### 短连接
因为客户端与服务器的整个连接过程很短暂，不会与服务器保持长时间的连接状态，所以就被称为“短连接”`（shortlived connections）`。早期的 HTTP 协议也被称为是“无连接”的协议。

### 长连接
针对短连接暴露出的缺点，HTTP 协议就提出了“长连接”的通信方式，也叫“持久连接”`（persistentconnections）`、“连接保活”`（keep alive）`、“连接复用”`（connection reuse）`。长链接的请求会复用之前已经连接过的tcp连接而不会重新创建一个新的http连接。

### 连接相关的头字段
我们可以在请求头里明确地要求使用长连接机制，使用的字段是`Connection`，值是`“keep-alive”`。  
在客户端，可以在请求头里加上`“Connection: close”`字段，告诉服务器：“这次通信后就关闭连接”。  
服务器端通常不会主动关闭连接，但也可以使用一些策略。拿 Nginx 来举例，它有两种方式：  
1. 使用`“keepalive_timeout”`指令，设置长连接的超时时间，如果在一段时间内连接上没有任何数据收发就主动断开连接，避免空闲连接占用系统资源。
2. 使用`“keepalive_requests”`指令，设置长连接上可发送的最大请求次数。比如设置成 1000，那么当 Nginx 在这个连接上处理了 1000 个请求后，也会主动断开连接。

### 队头阻塞
看完了短连接和长连接，接下来就要说到著名的“队头阻塞”（`Head-of-line blocking`，也叫“队首阻塞”）了。  
因为 HTTP 规定报文必须是“一发一收”，这就形成了一个先进先出的“串行”队列。队列里的请求没有轻重缓急的优先级，只有入队的先后顺序，排在最前面的请求被最优先处理。  

如果队首的请求因为处理的太慢耽误了时间，那么队列里后面的所有请求也不得不跟着一起等待，结果就是其他的请求承担了不应有的时间成本。  

在TCP传输过程中，由于单个数据包的丢失⽽造成的阻塞称为TCP上的队头阻塞。

### 性能优化
1. 并发连接:也就是同时对一个域名发起多个长连接，用数量来解决质量的问题。
2. 域名分片:后台有多个服务器，可以通过分发到不同的服务器解决，用数量来解决质量。

## HTTP的Cookie机制

1. Cookie 是服务器委托浏览器存储的一些数据，让服务器有了“记忆能力”；
2. 响应报文使用 `Set-Cookie` 字段发送`“key=value”`形式的 Cookie 值；
3. 请求报文里用 Cookie 字段发送多个 Cookie 值；
4. 为了保护 Cookie，还要给它设置有效期、作用域等属性，常用的有 `Max-Age、Expires、Domain、HttpOnly` 等；  
“HttpOnly”会告诉浏览器，此 Cookie 只能通过浏览器 HTTP 协议传输，禁止其他方式访问，浏览器的 JS 引擎就会禁用 document.cookie 等一切相关的 API，脚本攻击也就无从谈起了。  

另一个属性“SameSite”可以防范“跨站请求伪造”（XSRF）攻击，设置成“SameSite=Strict”可以严格限定 Cookie 不能随着跳转链接跨站发送，而“SameSite=Lax”则略宽松一点，允许 GET/HEAD 等安全方法，但禁止POST 跨站发送。  

还有一个属性叫“Secure”，表示这个 Cookie 仅能用 HTTPS 协议加密传输，明文的HTTP 协议会禁止发送。但 Cookie 本身不是加密的，浏览器里还是以明文的形式存在。
5. Cookie 最基本的用途是身份识别，实现有状态的会话事务。

## https
由于 HTTP 天生“明文”的特点，整个传输过程完全透明，任何人都能够在链路中截获、修改或者伪造请求 / 响应报文，数据不具有可信性。所以我们需要另外一种协议来确保数据的安全,https就应运而生了。  
HTTPS 其实是一个“非常简单”的协议，RFC 文档很小，只有短短的 7 页，里面规定了新的协议名“https”，默认端口号 443，至于其他的什么请求 - 应答模式、报文结构、请求方法、URI、头字段、连接管理等等都完全沿用 HTTP，没有任何新的东西。https只是在http的基础上加了一层，把原有的传输协议从下层的传输协议由 `TCP/IP` 换成了`SSL/TLS`  
![alt](http://img.carrotwu.com/FkfghsuX5zkRgJKEL4B_3Czq14uV)

### ssl 
TLS 由记录协议、握手协议、警告协议、变更密码规范协议、扩展协议等几个子协议组成，综合使用了对称加密、非对称加密、身份认证等许多密码学前沿技术。  

浏览器和服务器在使用 TLS 建立连接时需要选择一组恰当的加密算法来实现安全通信，这些算法的组合被称为“密码套件”（cipher suite，也叫加密套件）。

### 对称加密与非对称加密
1. 对称加密:加密和解密都用同一个秘钥,对称加密本身并不能解决秘钥在传输过程中被劫持的问题。
2. 非对称加密:加密用公钥,解密用秘钥。公钥由权威证书机构颁发，秘钥自己保存。因为每次传输数据都会使用不一样的秘钥导致非对称加密速度确实很慢  
所以现在都是通过使用混合加密的方式进行传输，第一次传输的过程中使用非对称加密传输秘钥，之后使用相同的公钥和秘钥进行堆成加密。（SSL 就是通信双方通过非对称加密协商出一个用于对称加密的密钥。）

## http的一些缺点
1. 队头阻塞
2. 明文传输
3. header每次传输都带上，头部数据大
4. 只能主动向服务端请求数据，不能被动推送 

## http2
1. 头部压缩 对臃肿的header进行压缩处理
2. 二进制格式
3. 通过流的形式解决队头阻塞和多路复用
4. 服务端可推送数据  

### http2的请求方式
1. ⾸先，浏览器准备好请求数据，包括了请求⾏、请求头等信息，如果是POST⽅法，那么还要有请求体。
2. 这些数据经过⼆进制分帧层处理之后，会被转换为⼀个个带有请求ID编号的帧，通过协议栈将这些帧发送给服务器。
3. 服务器接收到所有帧之后，会将所有相同ID的帧合并为⼀条完整的请求信息。
4. 然后服务器处理该条请求，并将处理的响应⾏、响应头和响应体分别发送⾄⼆进制分帧层。
5. 同样，⼆进制分帧层会将这些响应数据转换为⼀个个带有请求ID编号的帧，经过协议栈发送给浏览器。
6. 浏览器接收到响应帧之后，会根据ID编号将帧的数据提交给对应的请求。

### http3
虽然`HTTP/2`解决了应⽤层⾯的队头阻塞问题，不过和`HTTP/1.1`⼀样，`HTTP/2`依然是基于TCP协议的，⽽TCP最初就是为了单连接⽽设计的。所以http2依然还会有队友阻塞的问题，因为假设tcp中的一个数据包2发生了错误，这时候整个http需要重新发送数据包2，这样子回导致数据包1任然需要等待数据包2的完成。即时`http2`实现了多路复用，依然会出现这样的问题。  
![alt](http://img.carrotwu.com/Fq_lEoMtIoAqUrll2s60Shv5Lyce)  

因此，`HTTP/3`选择了⼀个折衷的⽅法⸺`UDP`协议，基于UDP实现了类似于 TCP的多路数据流、传输可靠性等功能，我们把这套功能称为`QUIC`协议。  
![alt](http://img.carrotwu.com/FrH9n_zf9BO-UbpYmrm1MuyB5d8a)  

1. 实现了类似TCP的流量控制、传输可靠性的功能。虽然UDP不提供可靠性的传输，但QUIC在UDP的基础之上增加了⼀层来保证数据可靠性传输。它提供了数据包重传、拥塞控制以及其他⼀些TCP中存在的特性。
2. 集成了TLS加密功能。⽬前QUIC使⽤的是TLS1.3，相较于早期版本TLS1.3有更多的优点，其中最重要的⼀点是减少了握⼿所花费的RTT个数。
3. 实现了HTTP/2中的多路复⽤功能。和TCP不同，QUIC实现了在同⼀物理连接上可以有多个独⽴的逻辑数据流。实现了数据流的单独传输，就解决了TCP中队头阻塞的问题。
4. 实现了快速握⼿功能。由于QUIC是基于UDP的，所以QUIC可以实现使⽤0-RTT或者1-RTT来建⽴连接，这意味着QUIC可以⽤最快的速度来发送和接收数据，这样可以⼤⼤提升⾸次打开⻚⾯的速度。