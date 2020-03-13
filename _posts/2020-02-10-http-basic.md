---
layout: post
title: "Http 基础"
date: 2020-02-10 00:00:00
categories: frontend
comments: true
---

# TCP/IP协议族

通常使用的网络（包括互联网）是在TCP/IP协议族的基础上运作的。而HTTP属于它内部的一个子集。

TCP/IP协议族按层次分别分为以下4层:
- 应用层
- 传输层
- 网络层
- 数据链路层。

## 应用层

应用层决定了向用户提供应用服务时通信的活动。TCP/IP协议族内预存了各类通用的应用服务。
- FTP（FileTransferProtocol，文件传输协议）
- DNS（DomainNameSystem，域名系统）
- HTTP协议

## 传输层
传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据传输。

在传输层有两个性质不同的协议：
- TCP TransmissionControlProtocol，传输控制协议
- UDP UserDataProtocol，用户数据报协议

## 网络层
网络层用来处理在网络上流动的数据包。数据包是网络传输的最小数据单位。该层规定了通过怎样的路径（所谓的传输路线）到达对方计算机，并把数据包传送给对方。

与对方计算机之间通过多台计算机或网络设备进行传输时，网络层所起的作用就是在众多的选项内选择一条传输路线。

## 数据链路层。
用来处理连接网络的硬件部分。包括控制操作系统、硬件的设备驱动、NIC（NetworkInterfaceCard，网络适配器，即网卡），及光纤等物理可见部分（还包括连接器等一切传输媒介）。

硬件上的范畴均在链路层的作用范围之内。

![TCP/IP layers](/assets/posts/2020-02-10/tcpip-layers.jpg)


## HTTP相关协议

### IP协议
IP协议的作用是把各种数据包传送给对方。而要保证确实传送到对方那里，则需要满足各类条件。其中两个重要的条件是IP地址和MAC地址（MediaAccessControlAddress）。

IP地址指明了节点被分配到的地址，MAC地址是指网卡所属的固定地址。IP地址可以和MAC地址进行配对。IP地址可变换，但MAC地址基本上不会更改。

> ARP: Address Resolution Protocol 是一种用以解析地址的协议，根据通信方的IP地址就可以反查出对应的MAC地址。

### TCP协议

按层次分，TCP位于传输层，提供可靠的字节流服务。所谓的字节流服务（ByteStreamService）是指，为了方便传输，将大块数据分割成以报文段（segment）为单位的数据包进行管理。而可靠的传输服务是指，能够把数据准确可靠地传给对方。

一言以蔽之，TCP协议为了更容易传送大数据才把数据分割，而且TCP协议能够确认数据最终是否送达到对方。


### DNS协议

负责域名解析的DNS服务

DNS（DomainNameSystem）服务是和HTTP协议一样位于应用层的协议。它提供域名到IP地址之间的解析服务。


## URL 和 URI

URI用字符串标识某一互联网资源，而URL表示资源的地点（互联网上所处的位置）。可见URL是URI的子集。

# HTTP 协议

通过请求和响应的交换达成通信

## 请求报文
请求报文是由请求方法、请求URI、协议版本、可选的请求首部字段和内容实体构成的。

```
GET / HTTP/1.1
Host: blog.jasonheylon.com
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
...
```

## 响应报文

响应报文基本上由协议版本、状态码（表示请求成功或失败的数字代码）、用以解释状态码的原因短语、可选的响应首部字段以及实体主体构成。
```
HTTP/1.1 200 OK
Cache-Control: private
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html
...
```

## HTTP是无状态

HTTP/1.1虽然是无状态协议，但为了实现期望的保持状态功能，于是引入了Cookie技术。有了Cookie再用HTTP协议通信，就可以管理状态了。

## HTTP方法(verb)

- `GET`: GET方法用来请求访问已被URI识别的资源
- `POST`: POST方法用来传输实体的主体。
- `PUT`: PUT方法用来传输文件
- `HEAD`: HEAD方法和GET方法一样，只是不返回报文主体部分。用于确认URI的有效性及资源更新的日期时间等。
- `DELETE`:  DELETE方法用来删除文件，是与PUT相反的方法。DELETE方法按请求URI删除指定的资源。
- `OPTIONS`: OPTIONS方法用来查询针对请求URI指定的资源支持的方法。
- `TRACE`: TRACE方法是让Web服务器端将之前的请求通信返回给客户端的方法。
- `CONNECT`: CONNECT方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信。主要使用SSL（SecureSocketsLayer，安全套接层）和TLS（TransportLayerSecurity，传输层安全）协议把通信内容加密后经网络隧道传输。


### Restful
做web开发时在定义API时都会遵循一些标准，Restful一直是一个广泛使用的标准尤其对于Rails开发者们，其中Http verb就表示了对资源做什么样的操作。
- `GET`: 获取资源信息
- `POST`: 创建资源
- `PUT` / `PATCH`: 对资源进行更新
- `DELETE`: 删除资源

### CROS & `OPTIONS`

在做前端开发时，我们经常发现在发送一个`POST`的Ajax请求前总会有一个`OPTIONS`。这个请求是浏览器自动发起的，用于检测服务器是否开启了CROS来支持跨域请求的。

### 其他

- HTTP/1.1的DELETE方法本身和PUT方法一样不带验证机制，所以一般的Web网站也不使用DELETE方法。
- TRACE方法本来就不怎么常用，再加上它容易引发XST（CrossSiteTracing，跨站追踪）攻击，通常就更不会用到了。

CONNECT方法的格式如下所示
```bash
# CONNECT 代理服务器名:端口号 HTTP版本

CONNECT blog.jasonheylon.com:8080 HTTP/1.1
HOST blog.jasonheylon.com
```

## 持久连接

HTTP协议的初始版本中，每进行一次HTTP通信就要断开一次TCP连接。

比如，使用浏览器浏览一个包含多张图片的HTML页面时，在发送请求访问HTML页面资源的同时，也会请求该HTML页面里包含的其他资源。因此，每次的请求都会造成无谓的TCP连接建立和断开，增加通信量的开销。

为解决上述TCP连接的问题，HTTP/1.1和一部分的HTTP/1.0想出了持久连接（HTTPPersistentConnections，也称为HTTPkeepalive或HTTPconnectionreuse）的方法。持久连接的特点是，只要任意一端没有明确提出断开连接，则保持TCP连接状态。

在HTTP/1.1中，所有的连接默认都是持久连接，

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求。

## COOKIE
Cookie会根据从服务器端发送的响应报文内的一个叫做SetCookie的首部字段信息，通知客户端保存Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入Cookie值后发送出去。


# HTTP报文内容

- 请求行： 请求行包含用于请求的方法，请求URI和HTTP版本。
- 状态行: 状态行包含表明响应结果的状态码，原因短语和HTTP版本。
- 首部字段: 首部字段包含表示请求和响应的各种条件和属性的各类首部。一般有4种首部，分别是：通用首部、请求首部、响应首部和实体首部。其他可能包含HTTP的RFC里未定义的首部（Cookie等）。

常用的内容编码有以下几种。gzip（GNUzip）compress（UNIX系统的标准压缩）deflate（zlib）identity（不进行编码）


# HTTP 状态码

## 状态码的类别

|状态码|类别|原因短语|
|---|---|---|
|1XX|Informational（信息性状态码）|接收的请求正在处理|
|2XX|Success（成功状态码）|请求正常处理完毕|
|3XX|Redirection（重定向状态码）|需要进行附加操作以完成请求|
|4XX|clientError（客户端错误状态码）|服务器无法处理请求|
|5XX|ServerError（服务器错误状态码）|服务器处理请求出错|


## 2XX

### 200 OK
表示从客户端发来的请求在服务器端被正常处理了。

### 204 No Content
该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。另外，也不允许返回任何实体的主体。比如，当从浏览器发出请求处理后，返回204响应，那么浏览器显示的页面不发生更新。

### 206 PartialContent
该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的GET请求。响应报文中包含由ContentRange指定范围的实体内容。

## 3XX

### 301MovedPermanently

永久性重定向。该状态码表示请求的资源已被分配了新的URI，以后应使用资源现在所指的URI。也就是说，如果已经把资源对应的URI保存为书签了，这时应该按Location首部字段提示的URI重新保存。像下方给出的请求URI，当指定资源路径的最后忘记添加斜杠“/”，就会产生301状态码。

### 302Found

临时性重定向。该状态码表示请求的资源已被分配了新的URI，希望用户（本次）能使用新的URI访问。

和301 Moved Permanently状态码相似，但302状态码代表的资源不是被永久移动，只是临时性质的。换句话说，已移动的资源对应的URI将来还有可能发生改变。比如，用户把URI保存成书签，但不会像301状态码出现时那样去更新书签，而是仍旧保留返回302状态码的页面对应的URI。

### 303 See Other

该状态码表示由于请求对应的资源存在着另一个URI，应使用GET方法定

>当301、302、303响应状态码返回时，几乎所有的浏览器都会把POST改成GET，并删除请求报文内的主体，之后请求会自动再次发送。301、302标准是禁止将POST方法改变成GET方法的，但实际使用时大家都会这么做。

### 304 Not Modified

大名鼎鼎的304，该状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但因发生请求未满足条件的情况后，直接返回304NotModified（服务器端资源未改变，可直接使用客户端未过期的缓存）。304状态码返回时，不包含任何响应的主体部分。304虽然被划分在3XX类别中，但是和重定向没有关系。

## 4XX

### 400 Bad Request
该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。另外，浏览器会像200OK一样对待该状态码。

### 401 Unauthorized
该状态码表示发送的请求需要有通过HTTP认证（BASIC认证、DIGEST认证）的认证信息。另外若之前已进行过1次请求，则表示用户认证失败。

返回含有401的响应必须包含一个适用于被请求资源的WWWAuthenticate首部用以质询（challenge）用户信息。当浏览器初次接收到401响应，会弹出认证用的对话窗口。

### 403 Forbidden
该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。

### 404 Not Found
该状态码表明服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用。


## 5XX

### 500
该状态码表明服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用。

**一般500为服务端发生了未捕获的异常**

### 503

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。如果事先得知解除以上状况需要的时间，最好写入RetryAfter首部字段再返回给客户端。

**一般需要检查一下nginx服务器配置等问题，也可能是服务程序没有启动等**


# WEB服务器

## 通信数据转发程序：代理、网关、隧道
HTTP通信时，除客户端和服务器以外，还有一些用于通信数据转发的应用程序，例如代理、网关和隧道。它们可以配合服务器工作。

HTTP通信时，除客户端和服务器以外，还有一些用于通信数据转发的应用程序，例如代理、网关和隧道。它们可以配合服务器工作。

### 代理
代理是一种有转发功能的应用程序，它扮演了位于服务器和客户端“中间人”的角色，接收由客户端发送的请求并转发给服务器，同时也接收服务器返回的响应并转发给客户端。

### 网关
网关是转发其他服务器通信数据的服务器，接收从客户端发送来的请求时，它就像自己拥有资源的源服务器一样对请求进行处理。有时客户端可能都不会察觉，自己的通信目标是一个网关。

### 隧道
隧道是在相隔甚远的客户端和服务器两者之间进行中转，并保持双方通信连接的应用程序。


## 代理

### 透明代理
透明代理转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理（TransparentProxy）。反之，对报文内容进行加工的代理被称为非透明代理。

### 缓存代理
代理转发响应时，缓存代理（CachingProxy）会预先将资源的副本（缓存）保存在代理服务器上。

当代理再次接收到对相同资源的请求时，就可以不从源服务器那里获取资源，而是将之前缓存的资源作为响应返回。


# HTTP 首部

HTTP/1.1规范定义了如下47种首部字段。

## 首部字段

|首部|字段名说明|
|---|---|
|CacheControl|控制缓存的行为|
|Connection|逐跳首部、连接的管理|
|Date|创建报文的日期时间|
|Pragma |报文指令|
|Trailer |报文末端的首部一览|
|TransferEncoding |指定报文主体的传输编码方式|
|Upgrade|升级为其他协议|
|Via|代理服务器的相关信息|
|Warning |错误通知|
|Accept|用户代理可处理的媒体类型|
|AcceptCharset|优先的字符集|
|AcceptEncoding|优先的内容编码|
|AcceptLanguage|优先的语言（自然语言）|
|AuthorizationWeb|认证信息|
|Expect|期待服务器的特定行为|
|From|用户的电子邮箱地址|
|Host|请求资源所在服务器|
|IfMatch|比较实体标记（ETag）|
|IfModifiedSince|比较资源的更新时间|
|IfNoneMatch|比较实体标记（与IfMatch相反）|
|IfRange|资源未更新时发送实体|
|Byte|的范围请求|
|IfUnmodifiedSince|比较资源的更新时间（与IfModifiedSince相反）|
|MaxForwards|最大传输逐跳数|
|ProxyAuthorization|代理服务器要求客户端的认证信息|
|Range|实体的字节范围请求|
|Referer|对请求中|
|URI|的原始获取方|
|TE|传输编码的优先级|
|UserAgentHTTP|客户端程序的信息|

## 响应首部字段

|首部|字段名说明|
|---|---|
|AcceptRanges|是否接受字节范围请求|
|Age|推算资源创建经过时间|
|ETag|资源的匹配信息|
|Location|令客户端重定向至指定|
|URIProxyAuthenticate|代理服务器对客户端的认证信息|
|RetryAfter|对再次发起请求的时机要求|
|ServerHTTP|服务器的安装信息|
|Vary|代理服务器缓存的管理信息|
|WWWAuthenticate|服务器对客户端的认证信息|

## 实体首部字段


|首部|字段名说明|
|---|---|
|Allow|资源可支持的HTTP方法|
|ContentEncoding|实体主体适用的编码方式|
|ContentLanguage|实体主体的自然语言|
|ContentLength|实体主体的大小（单位：字节）|
|ContentLocation|替代对应资源的URI|
|ContentMD5|实体主体的报文摘要|
|ContentRange|实体主体的位置范围|
|ContentType|实体主体的媒体类型|
|Expires|实体主体过期的日期时间|
|LastModified|资源的最后修改日期时间|


# HTTP缓存

![http-request](/assets/posts/2020-02-10/http-request.png)

## 通过 ETag 验证缓存的响应
- 服务器使用 ETag HTTP 标头传递验证令牌。
- 验证令牌可实现高效的资源更新检查：资源未发生变化时不会传送任何数据。

假定在首次提取资源 120 秒后，浏览器又对该资源发起了新的请求。 首先，浏览器会检查本地缓存并找到之前的响应。 遗憾的是，该响应现已过期，浏览器无法使用。 此时，浏览器可以直接发出新的请求并获取新的完整响应。 不过，这样做效率较低，因为如果资源未发生变化，那么下载与缓存中已有的完全相同的信息就毫无道理可言！

这正是验证令牌（在 ETag 标头中指定）旨在解决的问题。 服务器生成并返回的随机令牌通常是文件内容的哈希值或某个其他指纹。 客户端不需要了解指纹是如何生成的，只需在下一次请求时将其发送至服务器。 如果指纹仍然相同，则表示资源未发生变化，您就可以跳过下载。

```
ETag: W/"<etag_value>"
ETag: "<etag_value>"
```

> 'W/'(大小写敏感) 表示使用弱验证器。 弱验证器很容易生成，但不利于比较。 强验证器是比较的理想选择，但很难有效地生成。 相同资源的两个弱Etag值可能语义等同，但不是每个字节都相同。

### 避免“空中碰撞”
在ETag和 If-Match 头部的帮助下，您可以检测到"空中碰撞"的编辑冲突。

例如，当编辑MDN时，当前的wiki内容被散列，并在响应中放入Etag：
```
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4
```
将更改保存到Wiki页面（发布数据）时，POST请求将包含有ETag值的If-Match头来检查是否为最新版本。
```
If-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```
如果哈希值不匹配，则意味着文档已经被编辑，抛出412前提条件失败错误。

### 缓存未更改的资源

ETag头的另一个典型用例是缓存未更改的资源。 如果用户再次访问给定的URL（设有ETag字段），显示资源过期了且不可用，客户端就发送值为`ETag`的`If-None-Match` header字段：

```
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

服务器将客户端的ETag（作为`If-None-Match`字段的值一起发送）与其当前版本的资源的ETag进行比较，如果两个值匹配（即资源未更改），服务器将返回不带任何内容的304未修改状态，告诉客户端缓存版本可用（新鲜）。

### If-None-Match
If-None-Match 是一个条件式请求首部。对于 GET 和 HEAD 请求方法来说，当且仅当服务器上没有任何资源的 ETag 属性值与这个首部中列出的相匹配的时候，服务器端会才返回所请求的资源，响应码为  200  。对于其他方法来说，当且仅当最终确认没有已存在的资源的  ETag 属性值与这个首部中所列出的相匹配的时候，才会对请求进行相应的处理。

对于  GET 和 HEAD 方法来说，当验证失败的时候，服务器端必须返回响应码 304 （Not Modified，未改变）。对于能够引发服务器状态改变的方法，则返回 412 （Precondition Failed，前置条件失败）。需要注意的是，服务器端在生成状态码为 304 的响应的时候，必须同时生成以下会存在于对应的 200 响应中的首部：Cache-Control、Content-Location、Date、ETag、Expires 和 Vary 。

ETag 属性之间的比较采用的是弱比较算法，即两个文件除了每个比特都相同外，内容一致也可以认为是相同的。例如，如果两个页面仅仅在页脚的生成时间有所不同，就可以认为二者是相同的。

当与  If-Modified-Since  一同使用的时候，If-None-Match 优先级更高（假如服务器支持的话）。

以下是两个常见的应用场景：

- 采用 GET 或 HEAD  方法，来更新拥有特定的ETag 属性值的缓存。
- 采用其他方法，尤其是  PUT，将 If-None-Match used 的值设置为 * ，用来生成事先并不知道是否存在的文件，可以确保先前并没有进行过类似的上传操作，防止之前操作数据的丢失。这个问题属于更新丢失问题的一种。

```
If-None-Match: "bfc13a64729c4290ef5b2c2730249c88ca92d82d"

If-None-Match: W/"67ab43", "54ed21", "7892dd"

If-None-Match: *
```

## 修改时间
###  Last-Modified

The Last-Modified  是一个响应首部，其中包含源头服务器认定的资源做出修改的日期及时间。 它通常被用作一个验证器来判断接收到的或者存储的资源是否彼此一致。由于精确度比  ETag 要低，所以这是一个备用机制。包含有  If-Modified-Since 或 If-Unmodified-Since 首部的条件请求会使用这个字段。

```
Last-Modified: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```

### If-Modified-Since
If-Modified-Since 是一个条件式请求首部，服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回，状态码为 200  。如果请求的资源从那时起未经修改，那么返回一个不带有消息主体的  304  响应，而在 Last-Modified 首部中会带有上次修改时间。 不同于  If-Unmodified-Since, If-Modified-Since 只可以用在 GET 或 HEAD 请求中。

当与 If-None-Match 一同出现时，它（If-Modified-Since）会被忽略掉，除非服务器不支持 If-None-Match。
```
If-Modified-Since: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
```




## Cache-Control
![cache-control](/assets/posts/2020-02-10/http-cache-control.png)

### `no-cache`和`no-store`
`no-cache`表示必须先与服务器确认返回的响应是否发生了变化，然后才能使用该响应来满足后续对同一网址的请求。 因此，如果存在合适的验证令牌 (ETag)，`no-cache` 会发起往返通信来验证缓存的响应，但如果资源未发生变化，则可避免下载。

相比之下，`no-store`则要简单得多。 它直接禁止浏览器以及所有中间缓存存储任何版本的返回响应，例如，包含个人隐私数据或银行业务数据的响应。 每次用户请求该资产时，都会向服务器发送请求，并下载完整的响应。

### `public`与 `private`
如果响应被标记为`public`，则即使它有关联的 HTTP 身份验证，甚至响应状态代码通常无法缓存，也可以缓存响应。 大多数情况下，`public`不是必需的，因为明确的缓存信息（例如`max-age`）已表示响应是可以缓存的。

相比之下，浏览器可以缓存`private`响应。 不过，这些响应通常只为单个用户缓存，因此不允许任何中间缓存对其进行缓存。 例如，用户的浏览器可以缓存包含用户私人信息的 HTML 网页，但 CDN 却不能缓存。

### `max-age`
指令指定从请求的时间开始，允许提取的响应被重用的最长时间（单位：秒）。 例如，“max-age=60”表示可在接下来的 60 秒缓存和重用响应。

HTTP/1.1定义的 Cache-Control 头用来区分对缓存机制的支持情况， 请求头和响应头都支持这个属性。通过它提供的不同的值来定义缓存策略。


refs:
- [HTTP 缓存](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=zh-CN)
- [Etag spec](https://tools.ietf.org/html/rfc7232#section-2.3)
