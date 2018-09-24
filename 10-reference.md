# 引用

这一节主要用来引用一些额外的信息和外部资源链接

## Python 文档

* Python’s [socket module](https://docs.python.org/3/library/socket.html)
* Python’s [Socket Programming HOWTO](https://docs.python.org/3/howto/sockets.html#socket-howto)

## 错误信息

下面这段话来自 python 的 socket 模块文档：

> 所有的错误都会触发异常，像无效参数类型和内存不足的常见异常可以被抛出；从
> Python 3.3 开始，与 socket 或地址语义相关的错误会引发 OSError 或其子类之一的异
> 常 [引用](https://docs.python.org/3/library/socket.html)

异常                   | `errno` 常量 | 说明
--                     | --          | --
BlockingIOError        | EWOULDBLOCK  | 资源暂不可用，比如在非阻塞模式下调用 `send()` 方法，对方太繁忙面没有读取，发送队列满了，或者网络有问题
OSError                | EADDRINUSE   | 端口被战用，确保没有其它的进程与当前的程序运行在同一地址/端口上，你的服务器设置了 `SO_REUSEADDR` 参数
ConnectionResetError   | ECONNRESET   | 连接被重置，远端的进程崩溃，或者 socket 意外关闭，或是有防火墙或链路上的设配有问题
TimeoutError           | ETIMEDOUT    | 操作超时，对方没有响应
ConnectionRefusedError | ECONNREFUSED | 连接被拒绝，没有程序监听指定的端口

## socket 地址族

`socket.AF_INET` 和 `socket.AF_INET6` 是 `socket.socket()` 方法调用的第一个参数
，表示地址协议族，API 使用了一个期望传入指定格式参数的地址，这取决于是
`AF_INET` 还是 `AF_INET6`

 地址族         | 协议 | 地址元组                        | 说明
--              | --   | --                              | --
socket.AF_INET  | IPv4 | (host, port)                    | host 参数是个如 `www.example.com` 的主机名称，或者如 `10.1.2.3` 的 IPv4 地址
socket.AF_INET6 | IPv6 | (host, port, flowinfo, scopeid) | 主机名同上，IPv6 地址 如：`fe80::6203:7ab:fe88:9c23`，flowinfo 和 scopeid 分别表示 C 语言结构体 `sockaddr_in6` 中的 `sin6_flowinfo` 和 `sin6_scope_id` 成员

注意下面这段 python socket 模块中关于 host 值和地址元组文档

> 对于 IPv4 地址，使用主机地址的方式有两种：`''` 空字符串表示 `INADDR_ANY`，字符
> `'<broadcast>'` 表示 `INADDR_BROADCAST`，这个行为和 IPv6 不兼容，因此如果你的
> 程序中使用的是 IPv6 就应该避免这种做法。[源文档
> ](https://docs.python.org/3/library/socket.html#socket-families)

我在本教程中使用了 IPv4 地址，但是如果你的机器支持，也可以试试 IPv6 地址。`socket.getaddrinfo()` 方法会返回五个元组的序列，这包括所有创建 socket 连接的必要参数，`socket.getaddrinfo()` 方法理解并处理传入的 IPv6 地址和主机名

下面的例子中程序将返回一个通过 TCP 连接到 `example.org` 80 端口上的地址信息：

```python
>>> socket.getaddrinfo("example.org", 80, proto=socket.IPPROTO_TCP)
[(<AddressFamily.AF_INET6: 10>, <SocketType.SOCK_STREAM: 1>,
 6, '', ('2606:2800:220:1:248:1893:25c8:1946', 80, 0, 0)),
 (<AddressFamily.AF_INET: 2>, <SocketType.SOCK_STREAM: 1>,
 6, '', ('93.184.216.34', 80))]
```

如果 IPv6 可用的话结果可能有所不同，上面返回的值可以被用于 `socket.socket()` 和
`socket.connect()` 方法调用的参数，在 python socket 模块文档中的 [示例
](https://docs.python.org/3/library/socket.html#example) 一节中有客户端和服务端
程序

## 使用主机名

这一节主要适用于使用 `bind()` 和 `connect()` 或 `connect_ex()` 方法时如何使用主机名，然而当你使用回环地址做为主机名时，它总是会解析到你期望的地址。这刚好与客户端使用主机名的场景相反，它需要 DNS 解析的过程，比如 `www.example.com`

下面一段来自 python socket 模块文档

> 如果你主机名称做为 IPv4/v6 socket 地址的 host 部分，程序可能会出现非预期的结果
> ，由于 python 使用了 DNS 查找过程中的第一个结果，socket 地址会被解析成与真正的
> IPv4/v6 地址不同的其它地址，这取决于 DNS 解析和你的 host 文件配置。如果想得到
> 确定的结果，请使用数字格式的地址做为 host 参数的值 [源文档
> ](https://docs.python.org/3/library/socket.html)

通常回环地址 `localhost` 会被解析到 `127.0.0.1` 或 `::1` 上，你的系统可能就是这么设置的，也可能不是。这取决于你系统配置，与所有 IT 相关的事情一样，总会有例外的情况，没办法完全保证 localhost 被解析到了回环地址上

比如在 Linux 上，查看 `man nsswitch.conf` 的结果，域名切换配置文件，还有另外一个 macOS 和 Linux 通用的配置文件地址是：`/etc/hosts`，在 windows 上则是`C:\Windows\System32\drivers\etc\hosts`，hosts 文件包含了一个文本格式的静态域名地址映射表，总之 DNS 也是一个难题

有趣的是，在撰写这篇文章的时候（2018 年 6 月），有一个关于 [让 localhost 成为真正的 localhost](https://tools.ietf.org/html/draft-ietf-dnsop-let-localhost-be-localhost-02)的 RFC 草案，讨论就是围绕着 localhost 使用的情况开展的

最重要的一点是你要理解当你在应用程序中使用主机名时，返回的地址可能是任何东西，如果你有一个安全性敏感的应用程序，不要使用主机名。取决于你的应用程序和环境，这可能会困扰到你

**注意：** 安全方面的考虑和最佳实践总是好的，即使你的程序不是安全敏感型的应用。如果你的应用程序访问了网络，那它就应该是安全的稳定的。这表示至少要做到以下几点：

* 经常会有系统软件升级和安全补丁，包括 python，你是否使用了第三方的库？如果是的话，确保他们能正常工作并且更新到了新版本
* 尽量使用专用防火墙或基于主机的防火墙来限制与受信任系统的连接
* DNS 服务是如何配置的？你是否信任配置内容及其配置者
* 在调用处理其他代码之前，请确保尽可能地对请求数据进行了清理和验证，还要为此添加测试用例，并且经常运行

无论是否使用主机名称，你的应用程序都需要支持安全连接（加密授权），你可能会用到 [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)，这是一个超越了本教程的范围的话题。可以从 python 的 [SSL](https://docs.python.org/3/library/ssl.html) 模块文档了解如何开始使用它，这个协议和你的浏览器使用的安全协议是一样的

考虑到接口、IP 地址、域名解析这些「变量」，你应该怎么应对？如果你还没有网络应用程序审查流程，可以使用以下建议：

应用程序 | 使用       | 建议
--       | --         | --
服务端   | 回环地址   | 使用 IP 地址 127.0.0.1 或 ::1
服务端   | 以太网地址 | 使用 IP 地址，比如：10.1.2.3，使用空字符串表示本机所有 IP 地址
客户端   | 回环地址   | 使用 IP 地址 127.0.0.1 或 ::1
客户端   | 以太网地址 | 使用统一的不依赖域名解析的 IP 地址，特殊情况下才会使用主机地址，查看上面的安全提示

对于客户端或者服务端来说，如果你需要授权连接到主机，请查看如何使用 TLS

## 阻塞调用

如果一个 socket 函数或者方法使你的程序挂起，那么这个就是个阻塞调用，比如 accept(), connect(), send(), 和 recv() 都是 **阻塞** 的，它们不会立即返回，阻塞调用在返回前必须等待系统调用 (I/O) 完成。所以调用者 —— 你，会被阻止直到系统调用结束或者超过延迟时间或者有错误发生

阻塞的 socket 调用可以设置成非阻塞的模式，这样他们就可以立即返回。如果你想做到这一点，就得重构并重新设计你的应用程序

由于调用直接返回了，但是数据确没就绪，被调用者处于等待网络响应的状态，没法完成它的工作，这种情况下，当前 socket 的状态码 errno 应该是 `socket.EWOULDBLOCK`。`setblocking()` 方法是支持非阻塞模式的

默认情况下，socket 会以阻塞模式创建，查看 [socket 延迟的注意事项](https://docs.python.org/3/library/socket.html#notes-on-socket-timeouts) 中三种模式的解释

## 关闭连接

有趣的是 TCP 连接一端打开，另一端关闭的状态是完全合法的，这被称做 TCP「半连接」，是否需要这种保持状态是由应用程序决定的，通常来说不需要。这种状态下，关闭方将不能发送任何数据，它只能接收数据

我不是在提倡你采用这种方法，但是作为一个例子，HTTP 使用了一个名为「Connection」的头来标准化规定应用程序是否关闭或者保持连接状态，更多内容请查看 [RFC 7230 中 6.3 节, HTTP 协议 (HTTP/1.1): 消息语法与路由](https://tools.ietf.org/html/rfc7230#section-6.3)

当你在设计应用程序及其应用层协议的时候，最好先了解一下如何关闭连接，有时这很简单而且很明显，或者采取一些可以实现的原型，这取决于你的应用程序以及消息循环如何被处理成期望的数据，只要确保 socket 在完成工作后总是能正确关闭

## 字节序

查看维基百科 [字节序](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F) 中关于不同的 CPU 是如何在内存中存储字节序列的，处理单个字节时没有任何问题，但是当把多个字节处理成单个值（四字节整型）时，如果和你通信的另一端使用了不同的字节序时字节顺序需要被反转

字节顺序对于字符文本来说也很重要，字符文本通过表示为多字节的序列，就像 Unicode 一样。除非你只使用 `true` 和 ASCII 字符来控制客户端和服务端的实现，否则使用 utf-8 格式或者支持字节序标识(BOM) 的 Unicode 字符集会比较合适

在应用层协议中明确的规定使用编码格式是很重要的，你可以规定所有的文本都使用 utf-8 或者用「content-encoding」头指定编码格式，这将使你的程序不需要检测编码方式，当然也应该尽量避免这么做

当数据被调用存储到了文件或者数据库中而且又没有数据的元信息的时候，问题就很麻烦了，当数据被传到其它端，它将试着检测数据的编码方式。有关讨论，请参阅 Wikipedia 的 [Unicode](https://en.wikipedia.org/wiki/Unicode) 文章，它引用了 [RFC 3629:UTF-8, a transformation format of ISO 10646](https://tools.ietf.org/html/rfc3629#page-6)

> 然而 UTF-8 的标准 RFC 3629 中推荐禁止在 UTF-8 协议中使用标记字节序 (BOM)，但是
> 讨 论了无法实现的情况，最大的问题在于如何使用一种模式在不依赖 BOM 的情况下区分
> UTF-8 和其它编码方式

避开这些问题的方法就是总是存储数据使用的编码方式，换句话说，如果不只用 utf-8 格式的编码或者其它的带有 BOM 的编码就要尝试以某种方式将编码方式存储为元数据，然后你就可以在数据上附加编码的头信息，告诉接收者编码方式

TCP/IP 使用的字节顺序是 [big-endian](https://en.wikipedia.org/wiki/Endianness#Big)，被称做网络序。网络序被用来表示底层协议栈中的整型数字，好比 IP 地址和端口号，python 的 socket 模块有几个函数可以把这种整型数字从网络字节序转换成主机字节序

函数            | 说明
--              | --
socket.ntohl(x) | 把 32 位的正整型数字从网络字节序转换成主机字节序，在网络字节序和主机字节序相同的机器上这是个空操作，否则将是一个 4 字节的交换操作
socket.ntohs(x) | 把 16 位的正整型数字从网络字节序转换成主机字节序，在网络字节序和主机字节序相同的机器上这是个空操作，否则将是一个 2 字节的交换操作
socket.htonl(x) | 把 32 位的正整型数字从主机字节序转换成网络字节序，在网络字节序和主机字节序相同的机器上这是个空操作，否则将是一个 4 字节的交换操作
socket.htons(x) | 把 16 位的正整型数字从主机字节序转换成网络字节序，在网络字节序和主机字节序相同的机器上这是个空操作，否则将是一个 2 字节的交换操作

你也可以使用 [struct](https://docs.python.org/3/library/struct.html) 模块打包或者解包二进制数据（使用格式化字符串）：

```python
import struct
network_byteorder_int = struct.pack('>H', 256)
python_int = struct.unpack('>H', network_byteorder_int)[0]
```
