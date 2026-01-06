---
title: HTTP协议详解(HTTP、HTTP2、HTTP3)
author:  "DeepBlue"
tags: ['计算机网络']
date: 2022/7/1 20:46:25
categories: '算法'
header-img:  "https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/logo.png"
---

# HTTP协议详解

为什么会出现 HTTP 协议，从 HTTP1.0 到 HTTP3 经历了什么？HTTPS 又是怎么回事？

HTTP 是一种用于获取类似于 HTML 这样的资源的 **应用层通信协议**， 他是万维网的基础，是一种 CS 架构的协议，通常来说，HTTP 协议一般由浏览器等 “客户端” 发起，发起的这个请求被称为 Request, 服务端接受到客户端的请求后，会返回给客户端所请求的资源，这一过程被称为 Response，在大部分情况下，客户端和服务器之间还可能存在许多 [proxies](https://developer.mozilla.org/en-US/docs/Glossary/Proxy)，他们的作用可能各不相同，有些可能作为网关存在，有些可能作为缓存存在。

![](https://cdn.jsdelivr.net/gh/520MianXiangDuiXiang520/cdn@master/img/1609752900346-1609752900153.png)

HTTP 协议有三个基本的特性：

1. 简单：HTTP 的协议和报文是简单，易于理解和阅读的（HTTP/2 已经改用二进制传输数据，但 HTTP 整体还是简单的）
2. 可拓展的：请求和响应都包括 “Header” 和 “Body” 两部分，我们可以通过添加头部字段轻松的拓展 HTTP 的功能
3. 无状态的：服务端不保存客户端状态，也就是说每一次请求的服务端来说都是唯一无差别的，我们只能通过 Cookie 等技术创建有状态的会话。

## HTTP 的历史

HTTP 的历史可以追溯到万维网刚被发明的时候，1989年， Tim Berners-Lee 博士写了一份关于建立一个**通过网络传输超文本系统**的报告。该系统起初被命名为 *Mesh*，在随后的1990年项目实施期间被更名为万维网（World Wide Web）。他以现有的 TCP IP 协议为基础建造， 由四个部分组成：

* 用来表示超文本文档的文本格式，即超文本标记语言（HTML）
* 一个用来传输超文本的简单应用层协议，即超文本传输协议（HTTP）
* 一个用来显示或编辑超文本文档的客户端，即网络浏览器，而第一个浏览器则被称为 *WorldWideWeb*
* 一个用于提供可访问文档的服务，[httpd](https://zh.wikipedia.org/wiki/Httpd) 的前身.

这四部分在 1990 年底完成，这时候的 HTTP 协议还很简单，后来为了于其他版本的协议区分，最初的 HTTP 协议被记为 HTTP/0.9，

后来，随着计算机技术的发展，HTTP 协议也随着 HTTP/1.0, HTTP/1.1, HTTP/2 等关键版本更迭变得更加高效实用。

### HTTP/0.9 on-line

最初的 0.9 版本也被称为单行协议（on-line）, 基于 TCP 协议，该版本下只有一个可用的请求方法：GET， 请求格式也相当简单：

```http
GET /index.html
```

它表示客户端请求 `index.html` 的内容，0.9 版本的 HTTP 响应也同样简单，他只允许响应 HTML 格式的字符串，如：

```html
<html>
    <h1> ..... </h1>
</html>
```

这一阶段的响应甚至没有响应头，也没有响应码或错误代码，一旦出现问题，服务端会响应一段特殊的 HTML 字符串以便客户端查看。 服务端在发送完数据后，就会立刻关闭 TCP 连接。

### HTTP/1.0

0.9 版本的 HTTP 协议太过于简单甚至是简陋，而随着浏览器和服务器的应用被扩展到越来越多的领域，0.9 版本的协议已经不能适应，直到 1996年11月，[RFC 1945](https://tools.ietf.org/html/rfc1945) 定义了 HTTP/1.0, 但他并不是官方标准，该版本的 HTTP 协议较 0.9 版本有了一下改变：

1. 版本号被添加到了请求头上，像下面这样：

   ```http
   GET /mypage.html HTTP/1.0
   ```

2. 引入了 HTTP头的概念，无论是请求还是响应，允许传输元数据，这使得协议更加灵活和具有拓展性。

3. 请求方法拓展到了 GET，HEAD，POST

4. 在新 HTTP 头（`Content-Type`）的帮助下，可以传输不止 HTML 的任意格式的数据。

5. 响应时带上了状态码，使得浏览器能够知道响应的状态并作出响应的处理。

6. ...

### HTTP/1.1

同 0.9 版本一样，1.0 版本下，TCP 连接是不能复用的，数据发送完后服务端会立刻关闭连接，但由于建立 TCP 连接的代价较大，所以 1.0 版本的 HTTP 协议并不是足够高效，加上 HTTP/1.0 多种不同的实现方式在实际运用中显得有些混乱，自1995年就开始了 HTTP 的第一个标准化版本的修订工作，到1997年初，HTTP1.1 标准发布。

1.1 版本的改进包括：

1. 支持长连接：在 HTTP1.1 中默认开启 Connection： keep-alive，允许在一个 TCP 连接上传输多个 HTTP 请求和响应，减少了建立和关闭连接造成的性能消耗。

2. 支持 `pipline`: HTTP/1.1 还支持流水线（pipline）工作，流水线是指在同一条长连接上发出连续的请求，而不用等待应答返回。这样可以避免连接延迟。

3. 支持响应分块：对于比较大的响应，HTTP/1.2 通过 `Transfer-Encoding` 首部支持将其分割成多个任意大小的分块，每个数据块在发送时都会附上块的长度，最后用一个零长度的块作为消息结束的标志。

4. 新的缓存控制机制：HTTP/1.1定义的 [`Cache-Control`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control) 头用来区分对缓存机制的支持情况，同时，还提供 [`If-None-Match`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match), [`ETag`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag) ,  [`Last-Modified`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Last-Modified), [`If-Modified-Since`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-Modified-Since) 等实现缓存的验证等工作。

5. 允许不同域名配置到同一IP的服务器上：在 HTTP/1.0 时，认为每台服务器绑定一个唯一的 IP，但随着技术的进步，一台服务器的多个虚拟主机会共享一个IP，为了区分同一服务器上的不同服务，HTTP/1.1 在请求头中加入了 `HOST` 字段，它指明了请求将要发送到的服务器主机名和端口号，这是一个必须字段，请求缺少该字段服务端将会返回 400.

6. 引入内容协商机制，包括语言，编码，类型等，并允许客户端和服务器之间约定以最合适的内容进行交换。

7. 使用了 100 状态码：HTTP/1.0 中，定义：

   ```txt
   o 1xx: Informational - Not used, but reserved for future use
   ```

   在 2.0 版本时，使用了这个保留的状态码，用来表示临时响应。

### HTTPS

HTTP/1.1 之后，对 HTTP 协议的拓展变得更加简单，但 HTTP 依然存在一个天然的缺陷就是明文传输数据，直到 1994 年底，网景公司在 TCP/IP 协议栈的基础上添加了 SSL 层用来加密传输，后来，在标准化的过程中， SSL 成了  TLS （Transport Layer Security 传输层安全协议），基于 HTTPS 通信的客户端和服务器在建立完 TCP 连接之后会协商通信密钥，在之后的通信过程中， 客户端和服务器会使用该密钥对数据进行对称加密，以防数据被窃取或篡改。（密钥协商阶段会使用非对称加密）。

### HTTP/2

HTTP/1.1 虽然允许连接复用和以流水线方式运作，但在一个 TCP 连接里面，所有数据依然还是按序发送的，服务器只能处理完一个请求再去处理另一个请求，如果第一个请求非常慢，就会造成后面的请求长时间阻塞，这被称为 **队头阻塞（Head-of-line blocking）**，2009 年，谷歌公开了自行研发的 SPDY 协议，它基于 HTTPS，并采用多路复用解决了队头阻塞的问题，同时，它还使用了 Header 压缩等技术大大降低了延时并提高了带宽利用率，在之后的 2015 到 2019 年间，谷歌在自家浏览器上实践和证明了这个协议，SPDY 也成了 HTTP/2 的基石。

2015 年 5 月， HTTP/2 正式标准化，他与 1.x 版本 不同在于：

1. 1.x 版的 HTTP 协议传输的是文本信息，这对开发者很友好，但却浪费了计算机的性能，HTTP/2 改成了基于二进制而不再是基于文本的协议，
2. 这是一个复用协议。并行的请求能在同一个链接中处理，移除了HTTP/1.x中顺序和阻塞的约束。
3. 压缩了headers。因为headers在一系列请求中常常是相似的，其移除了重复和传输重复数据的成本。
4. 其允许服务器在客户端缓存中填充数据，通过一个叫服务器推送的机制来提前请求。

虽然 HTTP/2 2015 年就被标准化，在到目前为止，HTTP/1.1 任然被广泛使用，据 [MySSL](https://myssl.com/https_reports.html) 的最新统计，截至 2020 年 12 月，已有 65.84% 的站点支持了 HTTP/2. HTTP2在HTTP1.1的基础上做了很多性能上的优化，如果你想看看具体优化的结果，请访问[这里](https://http2.akamai.com/demo)

### HTTP/3

HTTP/3 是即将到来的第三个主要版本的 HTTP 协议，在 HTTP/3 中，将弃用 TCP 协议，改为使用基于 UDP 的 [QUIC](https://zh.wikipedia.org/wiki/快速UDP网络连接) 协议实现。QUIC（快速UDP网络连接）是一种实验性的网络传输协议，由Google开发，该协议旨在使网页传输更快。

在2018年10月28日的邮件列表讨论中，[IETF(互联网工程任务组)](https://zh.wikipedia.org/wiki/IETF) HTTP和QUIC工作组主席 [Mark Nottingham](https://zh.wikipedia.org/w/index.php?title=Mark_Nottingham&action=edit&redlink=1) 提出了将 HTTP-over-QUIC 更名为 HTTP/3 的正式请求，以“明确地将其标识为HTTP语义的另一个绑定……使人们理解它与 QUIC 的不同”，并在最终确定并发布草案后，将 QUIC 工作组继承到 HTTP 工作组, 在随后的几天讨论中，Mark Nottingham 的提议得到了 IETF 成员的接受，他们在2018年11月给出了官方批准，认可 HTTP-over-QUIC 成为 HTTP/3。

2019年9月，HTTP/3支持已添加到 CloudFlare 和 Chrome 上。Firefox Nightly 也将在2019年秋季之后添加支持。

## HTTP/1.1 细节

### HTTP报文

HTTP 的报文都由消息头和消息体两部分组成，两者之间以 `CRLF(回车换行)` 分割。


#### 请求头格式

请求头第一行为**请求行**，其余为请求头字段：如下：

```http
POST /api/article/list HTTP/1.1
Host: junebao.top:8888
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Content-Type: application/json;charset=utf-8
Content-Length: 32
Origin: https://junebao.top
Connection: keep-alive
Referer: https://junebao.top/
Cache-Control: max-age=0
```

请求行由三部分组成：

1. 请求方法
2. 请求资源的 url
3. 协议版本

他们以空格分隔，[RFC2068](https://tools.ietf.org/html/rfc2068#section-5.1.1) 定义了其中不同的请求方法，他们分别为  OPTIONS， GET， HEAD， POST， PUT， DELETE， TRACE,除此之外，后来还添加了一个 PATCH 方法。

| 方法    | 基本用法                                                     | 请求                                | 响应                                                    | 幂等性 | 缓存                     | 安全性 |
| ------- | ------------------------------------------------------------ | ----------------------------------- | ------------------------------------------------------- | ------ | ------------------------ | ------ |
| OPTIONS | 获取目的资源所支持的通信选项,如检测服务器所支持的请求方法或CORS预检请求 | 不能携带请求体或数据                | 可以携带响应体，但一般有效数据被放在头部如 Allow 等字段 | 幂等   | 不可缓存                 | 安全   |
| GET     | 用于获取某个资源                                             | 参数一般携带在 URL 后面，没有请求体 | 有响应体                                                | 幂等   | 可缓存                   | 安全   |
| HEAD    | 用于请求资源的头部信息，如下载前获取大文件的大小             | 没有请求体                          | 没有响应体，响应头应该与使用 GET 请求时的一样           | 幂等   | 可缓存                   | 安全   |
| POST    | 将数据发送给服务器                                           | 数据放在请求体中                    | 有响应体                                                | 不幂等 | 可缓存（包含新鲜信息时） | 不安全 |
| PUT     | 使用请求中的负载创建或替换目标资源                           | 数据放在请求体中                    | 有响应体                                                | 幂等   | 不可缓存                 | 不安全 |
| DELETE  | 删除指定资源                                                 | 可以由请求体                        | 可以由响应体                                            | 幂等   | 不可缓存                 | 不安全 |
| TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                   | 无请求体                            | 无响应体                                                | 幂等   | 不可缓存                 | 不安全 |
| PATCH   | 作为 PUT 的补充，用于修改已知资源的部分                      | 有请求体                            | 无响应体                                                | 非幂等 | 不可缓存                 | 不安全 |

##### 请求头字段

[RFC 2068](https://tools.ietf.org/html/rfc2068#section-5.3) 提供了 17 种请求头字段，但 HTTP 协议是易于拓展的，我们可以根据自己的需要添加自己的请求头，常见的请求头字段包括：

| 字段       | 作用                                                         | 示例                |
| ---------- | ------------------------------------------------------------ | ------------------- |
| HOST       | 指明了要发送到的服务器的主机号和端口号，这是一个必须字段，缺失服务器一般会返回 400,<br />端口号默认 80 和 443 | Host: www.baidu.com |
| ACCEPT     | 告知服务器客户端可以处理的内容类型，用[MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)来表示。 | Accept: text/html   |
| User-Agent | 用户代理标识                                                 |                     |
| Cookies    | 用于维持会话                                                 |                     |
| ...        | ...                                                          | ...                 |

#### 响应头格式

```txt
 Response      = Status-Line
                 *( general-header
                 | response-header
                 | entity-header )
                 CRLF
                 [ message-body ]
```

类似于请求头，响应头包括状态行和响应头字段两部分组成。

状态行包括协议版本，状态码，状态描述三部分组成，类似：

```http
http/2 200 ok
```

目前 http 使用的状态码分为 5 类：

* 1xx: 信息响应类
* 2xx: 正常响应类
* 3xx: 重定向类
* 4xx: 客户端错误类
* 5xx: 服务端错误类

##### 常见状态码

| 状态码 | 描述                       | 作用                                                         |
| ------ | -------------------------- | ------------------------------------------------------------ |
| 100    | Continue                   | 迄今为止的所有内容都是可行的，客户端应该继续请求             |
| 200    | Ok                         | 请求成功                                                     |
| 201    | Created                    | 该请求已成功，并因此创建了一个新的资源。这通常是在POST请求，或是某些PUT请求之后返回的响应。 |
| 301    | Moved Permanently          | 永久重定向                                                   |
| 302    | Found                      | 临时重定向                                                   |
| 400    | Bad Request                | 请求参数错误或语义错误                                       |
| 401    | Unauthorized               | 请求未认证                                                   |
| 403    | Forbidden                  | 拒绝服务                                                     |
| 404    | Not Found                  | 资源不存在                                                   |
| 429    | Too Many Requests          | 超过请求速率限制（节流）                                     |
| 500    | Internal Server Error      | 服务端未知异常                                               |
| 501    | Not Implemented            | 此请求方法不被服务端支持                                     |
| 502    | Bad Gateway                | 网关错误                                                     |
| 503    | Service Unavailable        | 服务不可用                                                   |
| 504    | Gateway Timeout            | 网关超时                                                     |
| 505    | HTTP Version Not Supported | HTTP 版本不被支持                                            |

### 无状态的 HTTP

HTTP 是一个无状态的协议，为了维持会话，每客户端请求时，都应该携带一个 “凭证”，证明 who am i, 目前维持会话常用的技术有：cookie, session, token, 等

#### cookie

[RFC 6265](https://tools.ietf.org/html/rfc6265) 定义了 Cookie 的工作方式, Cookie 是服务器发送给客户端并存储在本地的一小段数据，在用户第一次登录时，服务器生成 Cookie 并在响应头里添加 `Set-Cookie` 字段，客户端收到响应后，将 `Set-Cookie` 字段的值（Cookie）存储在本地，以后每次请求时，客户端会自动通过 `Cookie` 字段携带 Cookie。

Cookie 以键值的形式储存，除了必须的 Name 和 Value，还可以为 Cookie 设置以下属性：

* Domain：指定了哪写主机可以接收该 Cookie，默认为 Origin， 不包含子域名。
* Path：规定了请求主机下的哪些路径时要携带该 Cookie。
* Expires/Max-Age: 规定该 Cookie 过期时间或最大生存时间，该时间只与客户端有关。
* HttpOnly: JavaScript [`Document.cookie`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) API 无法访问带有 HttpOnly 属性的cookie,用于预防 XSS 攻击；用于持久化会话的 Cookie 一般应该设置 HttpOnly 。
* Secure：标记为 Secure 的 Cookie 只能使用 HTTPS 加密传输给服务器，因此可以防止中间人攻击，但 Cookie 天生具有不安全性，任何敏感数据都不应该使用 Cookie 传输，哪怕标记了 secure.
* Priority：
* SameSite：要求该 Cookie 在跨站请求时不会被发送，用来阻止 CSRF 攻击，它有三种可选的值：
  * None：在同站请求和跨站请求时都会携带上 Cookie
  * Strict：只会在访问同站请求时带上 Cookie
  * Lex：与 Strict 类似，但用户从外部站点导航至URL时（例如通过链接）除外，新版浏览器一般以 Lex 为默认选项。

Cookie 被完全保存在客户端，对客户端用户来说是透明的，用户可以自己创建和修改 Cookie，所以将敏感信息（如用于持久化会话的用户身份信息等）存放在 Cookie 中是十分危险的，如果不得已需要使用 Cookie 来存储和传递这类信息，应该考虑使用 JWT 等类似机制。

由于 Cookie 的不安全性，绝大部分 Web 站点已经开始停止使用 Cookie 持久化会话，但 Cookie 在一些对安全性要求不高的场景下依然被广泛使用，如：

* 个性化设置
* 浏览器用户行为跟踪。

> 了解更多：
>
> [超级 Cookie 和僵尸 Cookie](https://www.sohu.com/a/259750790_185201)
>
> [决战僵尸 Cookie](https://yq.aliyun.com/articles/299884/)

#### SESSION

Cookie 不安全的根源在于它将会话信息保存在了客户端，为此，就有了使用 Session 持久化会话的方案，用户在第一次登录时，服务器会将用户会话状态信息保存在服务器内存中，同时会为这段信息生成一串唯一索引，将这个索引作为 Cookie （Name 一般为 SESSION_IDSESSION_ID）返回给客户端，客户端下一次请求时，会自动携带这个 SESSION_ID, 服务器只需要根据 SESSION_ID 的值找到对应的状态信息就可以知道这次请求是谁发起的。

SESSION 很大程度上还是依赖于 Cookie，但这时 Cookie 中保存的已经是一段对客户端来说无意义的字符串了，因此使用 Session 能安全的实现会话持久化，但 Session 信息被保存在服务器内存中，可能造成服务器压力过大，并且在分布式和前后端分离的环境下，Session 并不容易拓展。

#### TOKEN

Cookie 和 Session 都是开箱即用的 API，因此，他们不可避免地缺少灵活性，在一般开发中，往往采用更灵活地 Token，Token 与 Session 原理一致，都是将会话信息保存到服务器，然后向客户端返回一个该信息的索引（token），但 Token 完全由开发者实现，可以根据需要将会话信息存储在内存，数据库，文件等地方，而对于该信息的索引，也可以根据具体需要选择使用请求头，请求体或者 Cookie 传递，也不必拘束于只 Cookie 传递。

#### JWT

全称 json web token, 是一种客户端存储会话状态的技术，它使用数字签名技术防止了负荷信息被篡改，jwt 包含三部分信息：

- Header：包含 token 类型和算法名称
- Payload：存储的负载信息（敏感信息不应该明文存放在此）
- Signature：服务端使用私钥对 Header 和 Payload 的签名，防止信息被篡改。

这三部分原本都是 json 字符串，最终他们会经过 Base64 编码后拼接到一起，使用 `.` 分割。

#### 分布式解决方案

在分布式场景下，同一用户的不同次请求可能会被打到不同的服务器上，这时如果还像单机时那样存储，就会出问题，一般的解决方案包括：

* 粘性 session：将用户绑定到一台服务器上，如 Nginx 负载均衡策略使用 ip_hash, 但这样如果当前服务器发生故障，可能导致分配到这台服务器上的用户登录信息失效，容错度低。
* session 复制：一台服务器的 session 改变，就广播给所有服务器，但会影响服务器性能
* session 共享：把所有服务器的 session 放在一起，如使用 redis 等分布式缓存做 session 集群。
* 客户端记录状态：使用诸如 JWT 之类的方法。

### 连接管理

连接管理是一个 HTTP 的关键话题：打开和保持连接在很大程度上影响着网站和 Web 应用程序的性能。在 HTTP/1.x 里有多种模型：**短连接** **长连接**和**HTTP 流水线**

#### 短连接

HTTP 最早期的模型，也是  HTTP/1.0 的默认模型，是短连接。每一个 HTTP 请求都由它自己独立的连接完成；这意味着发起每一个 HTTP 请求之前都会有一次 TCP 握手，而且是连续不断的。

TCP 协议握手本身就是耗费时间的，所以 TCP 可以保持更多的热连接来适应负载。短连接破坏了 TCP 具备的能力，新的冷连接降低了其性能。

#### 长连接

短连接有比较大的问题：

* 创建新连接耗费的时间尤为明显，另外 TCP 连接的性能只有在该连接被使用一段时间后(热连接)才能得到改善。为了缓解这些问题，*长连接* 的概念便被设计出来了，甚至在 HTTP/1.1 之前。或者这被称之为一个 *keep-alive* 连接。

一个长连接会保持一段时间，重复用于发送一系列请求，节省了新建 TCP 连接握手的时间，还可以利用 TCP 的性能增强能力。当然这个连接也不会一直保留着：连接在空闲一段时间后会被关闭(服务器可以使用 [`Keep-Alive`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive) 协议头来指定一个最小的连接保持时间)。

> 长连接也还是有缺点的；就算是在空闲状态，它还是会消耗服务器资源，而且在重负载时，还有可能遭受 [DoS attacks](https://developer.mozilla.org/en-US/docs/Glossary/DoS_attack) 攻击。这种场景下，可以使用非长连接，即尽快关闭那些空闲的连接，也能对性能有所提升。
>
> HTTP/1.0 里默认并不使用长连接。把 [`Connection`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Connection) 设置成 `close` 以外的其它参数都可以让其保持长连接，通常会设置为 `retry-after。`

#### 流水线

默认情况下，[HTTP](https://developer.mozilla.org/en/HTTP) 请求是按顺序发出的。下一个请求只有在当前请求收到应答过后才会被发出。由于会受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

流水线是在同一条长连接上发出连续的请求，而不用等待应答返回。这样可以避免连接延迟。理论上讲，性能还会因为两个 HTTP 请求有可能被打包到一个 TCP 消息包中而得到提升。就算 HTTP 请求不断的继续，尺寸会增加，但设置 TCP 的 [MSS](https://en.wikipedia.org/wiki/Maximum_segment_size)(Maximum Segment Size) 选项，仍然足够包含一系列简单的请求。

并不是所有类型的 HTTP 请求都能用到流水线：只有 [idempotent](https://developer.mozilla.org/en-US/docs/Glossary/idempotent) 方式，比如 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)、[`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD)、[`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 和 [`DELETE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/DELETE) 能够被安全的重试：如果有故障发生时，流水线的内容要能被轻易的重试。

![image](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/1460000013594854)

### CORS

**跨源资源共享** ([CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)) （或通俗地译为跨域资源共享）是一种基于[HTTP](https://developer.mozilla.org/zh-CN/docs/Glossary/HTTP) 头的安全机制，该机制通过允许服务器标示除了它自己以外的其它[origin](https://developer.mozilla.org/zh-CN/docs/Glossary/源)（域，协议和端口），这样浏览器可以访问加载这些资源。出于安全性，浏览器限制脚本内发起的跨源HTTP请求。 例如，XMLHttpRequest和Fetch API遵循同源策略。 这意味着使用这些API的Web应用程序只能从加载应用程序的同一个域请求HTTP资源，除非响应报文包含了正确CORS响应头。

跨源HTTP请求的一个例子：运行在 http://domain-a.com 的JavaScript代码使用ajax来发起一个到 https://domain-b.com/data.json 的请求。

![CORS-DEMO](https://mdn.mozillademos.org/files/14295/CORS_principle.png)

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

只要同时满足以下两大条件，就属于简单请求。

（1) 请求方法是以下三种方法之一：

- HEAD
- GET
- POST

（2）除了被用户代理自动设置的首部字段（例如 `Connection`，`User-Agent`）和在 Fetch 规范中定义为 禁用首部名称 的其他首部，允许人为设置的字段为 Fetch 规范定义的 对 CORS 安全的首部字段集合。该集合为：

- [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept)
- [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)
- [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language)
- [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) （需要注意额外的限制）
- `DPR`
- `Downlink`
- `Save-Data`
- `Viewport-Width`
- `Width`

- `Content-Type`的值仅限于下列三者之一：
  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`

#### 简单请求

对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个`Origin`字段。

下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个`Origin`字段。

```http
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

如果`Origin`指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含`Access-Control-Allow-Origin`字段（详见下文），就知道出错了，从而抛出一个错误，被`XMLHttpRequest`的`onerror`回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果`Origin`指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

> ```http
> Access-Control-Allow-Origin: http://api.bob.com
> Access-Control-Allow-Credentials: true
> Access-Control-Expose-Headers: FooBar
> Content-Type: text/html; charset=utf-8
> ```

上面的头信息之中，有三个与CORS请求相关的字段，都以`Access-Control-`开头。

#### 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是`PUT`或`DELETE`，或者`Content-Type`字段的类型是`application/json`。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的`XMLHttpRequest`请求，否则就报错。

```javascript
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

上面代码中，HTTP请求的方法是`PUT`，并且发送一个自定义头信息`X-Custom-Header`。

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的HTTP头信息。

```http
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

**（1）Access-Control-Request-Method**

该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是`PUT`。

**（2）Access-Control-Request-Headers**

该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是`X-Custom-Header`。

服务器收到"预检"请求以后，检查了`Origin`、`Access-Control-Request-Method`和`Access-Control-Request-Headers`字段以后，确认允许跨源请求，就可以做出回应。否则不作出回应，也就是我们会经常性的出现的一种开发服务时候的错误。

### HTTP 缓存

**HTTP/1.1**定义的 [`Cache-Control`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control) 头用来区分对缓存机制的支持情况， 请求头和响应头都支持这个属性。通过它提供的不同的值来定义缓存策略。

#### 缓存策略

- ```html
  Cache-Control: no-store  无缓存
  ```

- ``` html
  Cache-Control: no-cache  缓存但重新验证，此方式下，每次有请求发出时，缓存会将此请求发到服务器,服务器端会验证请求中所描述的缓存是否过期，若未过期（注：实际就是返回304），则缓存才使用本地缓存副本。
  ```

- ```html
  Cache-Control: private
  Cache-Control: public  "public" 指令表示该响应可以被任何中间人（译者注：比如中间代理、CDN等）缓存。若指定了"public"，则一些通常不被中间人缓存的页面（译者注：因为默认是private）（比如 带有HTTP验证信息（帐号密码）的页面 或 某些特定状态码的页面），将会被其缓存。
  ```

#### 缓存的过期机制

缓存的过期机制中，最重要的指令是 "`max-age=<seconds>`"，表示资源能够被缓存（保持新鲜）的最大时间。相对[Expires](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Expires)而言，max-age是距离请求发起的时间的秒数。针对应用中那些不会改变的文件，通常可以手动设置一定的时长以保证缓存有效，例如图片、css、js等静态资源。

```html
Cache-Control: max-age=31536000
```

## HTTPS 细节

### HTTPS的请求过程

1、浏览器发起往服务器的 443 端口发起请求，请求携带了浏览器支持的加密算法和哈希算法。
2、服务器收到请求，选择浏览器支持的加密算法和哈希算法。
3、服务器下将数字证书返回给浏览器，这里的数字证书可以是向某个可靠机构申请的，也可以是自制的。（注释：证书包括以下这些内容：1. 证书序列号。2. 证书过期时间。3. 站点组织名。4. 站点DNS主机名。5. 站点公钥。6. 证书颁发者名。7. 证书签名。因为证书就是要给大家用的，所以不需要加密传输）
4、浏览器进入数字证书认证环节，这一部分是浏览器内置的 TSL 完成的：
	**4.1** 首先浏览器会从内置的证书列表中索引，找到服务器下发证书对应的机构，如果没有找到，此时就会提示用户该证书是不是由权威机构颁发，是不可信任的。如果查到了对应的机构，则取出该机构颁发的公钥。
	**4.2** 用机构的证书公钥解密得到证书的内容和证书签名，内容包括网站的网址、网站的公钥、证书的有效期等。浏览器会先验证证书签名的合法性。签名通过后，浏览器验证证书记录的网址是否和当前网址是一致的，不一致会提示用户。如果网址一致会检查证书有效期，证书过期了也会提示用户。这些都通过认证时，浏览器就可以安全使用证书中的网站公钥了。
	**4.3** 浏览器生成一个随机数 R，并使用网站公钥对 R 进行加密。
5、浏览器将加密的 R 传送给服务器。
6、服务器用自己的私钥解密得到 R。
7、服务器以 R 为密钥使用了对称加密算法加密网页内容并传输给浏览器。
8、浏览器以 R 为密钥使用之前约定好的解密算法获取网页内容。

对其中细节的深究可以参见：https://zhuanlan.zhihu.com/p/43789231

## HTTP/2 细节

### 实现

2015 年，HTTP/2 发布。HTTP/2 是现行 HTTP 协议（HTTP/1.x）的替代，但它不是重写，HTTP 方法/状态码/语义都与 HTTP/1.x 一样。HTTP/2 基于 SPDY3，专注于**性能**，最大的一个目标是在用户和网站间只用一个连接（connection）。

**那么SPDY3是什么呢？**

SPDY是谷歌自行研发的 SPDY 协议，主要解决 HTTP/1.1 效率不高的问题。谷歌推出 SPDY，才算是正式改造 HTTP 协议本身。降低延迟，压缩 header 等等，SPDY 的实践证明了这些优化的效果，也最终带来 HTTP/2 的诞生。

HTTP/2 由两个规范（Specification）组成：

1. Hypertext Transfer Protocol version 2 - [RFC7540](https://tools.ietf.org/html/rfc7540)
2. HPACK - Header Compression for HTTP/2 - [RFC7541](https://tools.ietf.org/html/rfc7541)

**那么HTTP2在HTTP1.1的基础上做了哪些改进**

- 二进制传输
- 请求和响应复用
- Header压缩
- Server Push（服务端推送）

#### 二进制传输

HTTP/2 采用二进制格式传输数据，而非 HTTP 1.x 的文本格式，二进制协议解析起来更高效。 HTTP / 1 的请求和响应报文，都是由起始行，首部和实体正文（可选）组成，各部分之间以文本换行符分隔。**HTTP/2 将请求和响应数据分割为更小的帧，并且它们采用二进制编码**。

新的二进制分帧机制改变了客户端与服务器之间交换数据的方式。 为了说明这个过程，我们需要了解 HTTP/2 的三个概念：

- *数据流*：已建立的连接内的双向字节流，可以承载一条或多条消息。
- *消息*：与逻辑请求或响应消息对应的完整的一系列帧。
- *帧*：HTTP/2 通信的最小单位，每个帧都包含帧头，至少也会标识出当前帧所属的数据流。

这些概念的关系总结如下：

- 所有通信都在一个 TCP 连接上完成，此连接可以承载任意数量的双向数据流。
- 每个数据流都有一个唯一的标识符和可选的优先级信息，用于承载双向消息。
- 每条消息都是一条逻辑 HTTP 消息（例如请求或响应），包含一个或多个帧。
- 帧是最小的通信单位，承载着特定类型的数据，例如 HTTP 标头、消息负载等等。 来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

![HTTP/2 数据流、消息和帧](https://developers.google.com/web/fundamentals/performance/http2/images/streams_messages_frames01.svg?hl=zh-cn)

#### 请求和响应复用

在 HTTP/1.x 中，如果客户端要想发起多个并行请求以提升性能，则必须使用多个 TCP 连接这是 HTTP/1.x 交付模型的直接结果，该模型可以保证每个连接每次只交付一个响应（响应排队）。 更糟糕的是，这种模型也会导致队首阻塞，从而造成底层 TCP 连接的效率低下。

HTTP/2 中新的二进制分帧层突破了这些限制，实现了完整的**请求和响应复用**：客户端和服务器可以将 HTTP 消息分解为互不依赖的帧，然后交错发送，最后再在另一端把它们重新组装起来。

在 HTTP/2 中，有了二进制分帧之后，HTTP /2 不再依赖 TCP 链接去实现多流并行了，在 HTTP/2 中：

- 同域名下所有通信都在单个连接上完成。
- 单个连接可以承载任意数量的双向数据流。
- 数据流以消息的形式发送，而消息又由一个或多个帧组成，多个帧之间可以乱序发送，因为根据帧首部的流标识可以重新组装。

这一特性，使性能有了极大提升：

- 同个域名只需要占用一个 TCP 连接，使用一个连接并行发送多个请求和响应,消除了因多个 TCP 连接而带来的延时和内存消耗。
- 并行交错地发送多个请求，请求之间互不影响。
- 并行交错地发送多个响应，响应之间互不干扰。
- 在 HTTP/2 中，每个请求都可以带一个 31bit 的优先值，0 表示最高优先级， 数值越大优先级越低。有了这个优先值，客户端和服务器就可以在处理不同的流时采取不同的策略，以最优的方式发送流、消息和帧。

![](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/2019-03-06-4.png)

#### Header压缩

在 HTTP/1 中，我们使用文本的形式传输 header，在 header 携带 cookie 的情况下，可能每次都需要重复传输几百到几千的字节。为了减少这块的资源消耗并提升性能，HTTP/2 使用 HPACK 压缩格式压缩请求和响应标头元数据，这种格式采用两种强大的技术：

1. 这种格式支持通过**静态霍夫曼代码对传输的标头字段进行编码**，从而减小了各个传输的大小。
2. 这种格式要求客户端和服务器同时维护和更新一个包含之前见过的标头字段的**索引列表**（换句话说，它可以建立一个共享的压缩上下文），此列表随后会用作参考，对之前传输的值进行有效编码。

利用霍夫曼编码，可以在传输时对各个值进行压缩，而利用之前传输值的索引列表，我们可以通过传输索引值的方式对重复值进行编码，索引值可用于有效查询和重构完整的标头键值对。

![HPACK：HTTP/2 的标头压缩](https://developers.google.com/web/fundamentals/performance/http2/images/header_compression01.svg?hl=zh-cn)

作为一种进一步优化方式，HPACK 压缩上下文包含一个静态表和一个动态表：静态表在规范中定义，并提供了一个包含所有连接都可能使用的常用 HTTP 标头字段（例如，有效标头名称）的列表；动态表最初为空，将根据在特定连接内交换的值进行更新。 因此，为之前未见过的值采用静态 Huffman 编码，并替换每一侧静态表或动态表中已存在值的索引，可以减小每个请求的大小。

注：在 HTTP/2 中，请求和响应标头字段的定义保持不变，仅有一些微小的差异：所有标头字段名称均为小写，请求行现在拆分成各个 `:method`、`:scheme`、`:authority` 和 `:path` 伪标头字段。

如需了解有关 HPACK 压缩算法的完整详情，请参阅 [IETF HPACK - HTTP/2 的标头压缩](https://tools.ietf.org/html/draft-ietf-httpbis-header-compression)。

#### Server Push

HTTP/2 新增的另一个强大的新功能是，服务器可以对一个客户端请求发送多个响应。 换句话说，除了对最初请求的响应外，服务器还可以向客户端推送额外资源如下图所示，而无需客户端明确地请求。

![服务器为推送资源发起新数据流 (promise) ](https://developers.google.com/web/fundamentals/performance/http2/images/push01.svg?hl=zh-cn)

为什么在浏览器中需要一种此类机制呢？一个典型的网络应用包含多种资源，客户端需要检查服务器提供的文档才能逐个找到它们。 那为什么不让服务器提前推送这些资源，从而减少额外的延迟时间呢？ 服务器已经知道客户端下一步要请求什么资源，这时候服务器推送即可派上用场。

事实上，如果您在网页中内联过 CSS、JavaScript，或者通过数据 URI 内联过其他资产（请参阅[资源内联](https://hpbn.co/http1x/#resource-inlining)），那么您就已经亲身体验过服务器推送了。 对于将资源手动内联到文档中的过程，我们实际上是在将资源推送给客户端，而不是等待客户端请求。 使用 HTTP/2，我们不仅可以实现相同结果，还会获得其他性能优势。 推送资源可以进行以下处理：

- 由客户端缓存
- 在不同页面之间重用
- 与其他资源一起复用
- 由服务器设定优先级
- 被客户端拒绝

##### 服务端推送如何实现

所有服务器推送数据流都由 `PUSH_PROMISE` 帧发起，表明了服务器向客户端推送所述资源的意图，并且需要先于请求推送资源的响应数据传输。 这种传输顺序非常重要：客户端需要了解服务器打算推送哪些资源，以免为这些资源创建重复请求。 满足此要求的最简单策略是先于父响应（即，`DATA` 帧）发送所有 `PUSH_PROMISE` 帧，其中包含所承诺资源的 HTTP 标头。

在客户端接收到 `PUSH_PROMISE` 帧后，它可以根据自身情况选择拒绝数据流（通过 `RST_STREAM` 帧）。 （例如，如果资源已经位于缓存中，便可能会发生这种情况。） 这是一个相对于 HTTP/1.x 的重要提升。 相比之下，使用资源内联（一种受欢迎的 HTTP/1.x“优化”）等同于“强制推送”：客户端无法选择拒绝、取消或单独处理内联的资源。

使用 HTTP/2，客户端仍然完全掌控服务器推送的使用方式。 客户端可以限制并行推送的数据流数量；调整初始的流控制窗口以控制在数据流首次打开时推送的数据量；或完全停用服务器推送。 这些优先级在 HTTP/2 连接开始时通过 `SETTINGS` 帧传输，可能随时更新。

### 过渡到 HTTP/2

上面说了这么多，我们要如何启用HTTP2呢？

对应Nginx服务器参见：

https://jkzhao.github.io/2018/01/16/Nginx%E9%85%8D%E7%BD%AEHTTP-2-0/

spring boot使用的话如果你是使用的内置的Tomcat服务器，那么只需要在配置文件中添加配置：

```yaml
server:
  http2:
    enabled: on
```

Tomcat 服务器：

只有Tomcat 9 版本之后版本才支持HTTP2协议。在 conf/server.xml 中增加内容：

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol" maxThreads="150" SSLEnabled="true">
<UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol"/>
<SSLHostConfig honorCipherOrder="false">
<Certificate certificateKeyFile="conf/ca.key" certificateFile="conf/ca.crt"/>
</SSLHostConfig>
</Connector>
```

其余服务器的话大家应该网上已经有很多文章去介绍了，大家去查一下吧。

## HTTP/3 细节

### 为什么要出现HTTP3

虽然 HTTP/2 解决了很多之前旧版本的问题，但是它还是存在一个巨大的问题，主要是底层支撑的 `TCP 协议`造成的。

上文提到 HTTP/2 使用了多路复用，一般来说同一域名下只需要使用一个 TCP 连接。但当这个连接中出现了丢包的情况，那就会导致 HTTP/2 的表现情况反倒不如 HTTP/1 了。

因为在出现丢包的情况下，整个 TCP 都要开始等待重传，也就导致了后面的所有数据都被阻塞了。但是对于 HTTP/1.1 来说，可以开启多个 TCP 连接，出现这种情况反到只会影响其中一个连接，剩余的 TCP 连接还可以正常传输数据。

那么可能就会有人考虑到去修改 TCP 协议，其实这已经是一件不可能完成的任务了。因为 TCP 存在的时间实在太长，已经充斥在各种设备中，并且这个协议是由操作系统实现的，更新起来不大现实。

基于这个原因，**Google 就更起炉灶搞了一个基于 UDP 协议的 QUIC 协议，并且使用在了 HTTP/3 上**，HTTP/3 之前名为 HTTP-over-QUIC，从这个名字中我们也可以发现，HTTP/3 最大的改造就是使用了 QUIC（快速 UDP Internet 连接）。

### QUIC 功能

#### 0-RTT

通过使用类似 TCP 快速打开的技术，缓存当前会话的上下文，在下次恢复会话的时候，只需要将之前的缓存传递给服务端验证通过就可以进行传输了。**0RTT 建连可以说是 QUIC 相比 HTTP2 最大的性能优势**。

这里面有两层含义:

- 传输层 0 RTT 就能建立连接。
- 加密层 0 RTT 就能建立加密连接。

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/4d8ca515daf3e1956250032f998086a1.png)

上图中间是HTTPS的一次完全握手的建连过程，需要 3 个 RTT。就算是会话复用也需要至少 2 个 RTT。但是HTTP3使用的QUIC由于建立在 UDP 的基础上，同时又实现了 0 RTT 的安全握手，所以在大部分情况下，只需要 0 个 RTT 就能实现数据发送，在实现前向加密的基础上，并且 0RTT 的成功率相比 TLS 的会话记录单要高很多。

#### 多路复用

虽然 HTTP/2 支持了多路复用，但是 TCP 协议终究是没有这个功能的。QUIC 原生就实现了这个功能，并且传输的单个数据流可以保证有序交付且不会影响其他的数据流，这样的技术就解决了之前 TCP 存在的问题。

同 HTTP2.0 一样，同一条 QUIC 连接上可以创建多个 stream，来发送多个 HTTP 请求，但是，QUIC 是基于 UDP 的，一个连接上的多个 stream 之间没有依赖。比如下图中 stream2 丢了一个 UDP 包，不会影响后面跟着 Stream3 和 Stream4，不存在 TCP 队头阻塞。虽然 stream2 的那个包需要重新传，但是 stream3、stream4 的包无需等待，就可以发给用户。

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/2019-03-06-8.png)

#### 加密认证的报文

TCP 协议头部没有经过任何加密和认证，所以在传输过程中很容易被中间网络设备篡改，注入和窃听。比如修改序列号、滑动窗口。这些行为有可能是出于性能优化，也有可能是主动攻击。

但是 QUIC 的 packet 可以说是武装到了牙齿。除了个别报文比如 PUBLIC_RESET 和 CHLO，所有报文头部都是经过认证的，报文 Body 都是经过加密的。

这样只要对 QUIC 报文任何修改，接收端都能够及时发现，有效地降低了安全风险。

#### 向前纠错机制

QUIC 协议有一个非常独特的特性，称为向前纠错 (Forward Error Correction，FEC)，每个数据包除了它本身的内容之外，还包括了部分其他数据包的数据，因此少量的丢包可以通过其他包的冗余数据直接组装而无需重传。向前纠错牺牲了每个数据包可以发送数据的上限，但是减少了因为丢包导致的数据重传，因为数据重传将会消耗更多的时间(包括确认数据包丢失、请求重传、等待新数据包等步骤的时间消耗)

假如说这次我要发送三个包，那么协议会算出这三个包的异或值并单独发出一个校验包，也就是总共发出了四个包。当出现其中的非校验包丢包的情况时，可以通过另外三个包计算出丢失的数据包的内容。**当然这种技术只能使用在丢失一个包的情况下，如果出现丢失多个包就不能使用纠错机制了，只能使用重传的方式了**。

### 写在最后

可见HTTP3在效率上和安全性上都有了很大程度上的修改，但是由于目前这个标准还在论证中，Nginx等也只是在测试版中加入了对HTTP3的支持，等到技术真正的论证实现完成，我们就可以使用上快速且安全的HTTP3协议了，期待着这一天的到来。

## 参考

[mozilla 开发文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)

[谷歌开发文档 HTTP2 简介](https://developers.google.com/web/fundamentals/performance/http2?hl=zh-cn)

[一文读懂HTTP2和HTTP3](https://blog.fundebug.com/2019/03/07/understand-http2-and-http3/)

[WIKI](https://zh.wikipedia.org/wiki/%E5%BF%AB%E9%80%9FUDP%E7%BD%91%E7%BB%9C%E8%BF%9E%E6%8E%A5)