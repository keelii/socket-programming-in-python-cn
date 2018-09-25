# 故障排查

某些东西运行不了是很常见的，你可能不知道应该怎么做，不用担心，所有人都会遇到这种问题，希望你借助本教程、调试器和万能的搜索引擎解决问题并且继续下去

如果还是解决不了，你的第一站应该是 python 的 [socket](https://docs.python.org/3/library/socket.html) 模块文档，确保你读过文档中每个我们使用到的方法、函数。同样的可以从引用一节中找到一些办法，尤其是错误一节中的内容

有的时候问题并不是由你的源代码引起的，源代码可能是正确的。有可能是不同的主机、客户端和服务器。也可能是网络原因，比如路由器、防火墙或者是其它网络设备扮演了中间人的角色

对于这些类型的问题，额外的一些工具是必要的。下面这些工具或者集可能会帮到你或者至少提供一些线索

## pin

ping 命令通过发送一个 [ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol) 报文来检测主机是否连接到了网络，它直接与操作系统上的 TCP/IP 协议栈通信，所以它在主机上是独立于任何应用程序运行的

下面是一段在 macOS 上执行 ping 命令的结果

```bash
$ ping -c 3 127.0.0.1
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.058 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.165 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.164 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.058/0.129/0.165/0.050 ms
```

注意后面的统计输出，这对你排查间歇性的连接问题很有帮助。比如说，是否有数据包丢失？网络延迟怎么样（查看消息的往返时间）

如果你与主机之间有防火墙的话，ping 发送的请求可能会被阻止。防火墙管理员定义了一些规则强制阻止一些请求，主要的原因就是他们不想自己的主机是可以被发现的。如果你的机器也出现这种情况的话，请确保在规则中添加了允许 ICMP 包的发送

ICMP 是 ping 命令使用的协议，但它也是 TCP 和其他底层用于传递错误消息的协议，如果你遇到奇怪的行为或缓慢的连接，可能就是这个原因

ICMP 消息通过类型和代号来定义。下面有一些重要的信息可以参考：

ICMP 类型 | ICMP 代码 | 说明
--        | --        | --
8         | 0         | 打印请求
0         | 0         | 打印回复
3         | 0         | 目标网络不可达
3         | 1         | 目标主机不可达
3         | 2         | 目标协议不可达
3         | 3         | 目标端口不可达
3         | 4         | 需要分片，但是 DF(Don't fragmentation) 标识已被设置
11        | 0         | 网络存在环路

查看 [Path MTU Discovery](https://en.wikipedia.org/wiki/Path_MTU_Discovery#Problems_with_PMTUD) 更多关于分片和 ICMP 消息的内容，里面遇到的问题就是我前面提及的一些奇怪行为

## netstat

在 [查看 socket 状态](#查看 socket 状态) 一节中我们已经知道如何使用 netstat 来查看 socket 及其状态的信息。这个命令在 macOS, Linux, Windows 上都可以使用

在之前的示例中我并没有提及 `Recv-Q` 和 `Send-Q` 列。这些列表示发送或者接收队列中网络缓冲区数据的字节数，但是由于某些原因这些字节还没被远程或者本地应用读写

换句话说，这些网络中的字节还在操作系统的队列中。一个原因可能是应用程序受 CPU 限制或者无法调用 `socket.recv()`、`socket.send()` 方法处理，或者因为其它一些网络原因导致的，比如说网络的拥堵、失败、硬件及电缆的问题

为了复现这个问题，看看到底在错误发生前我应该发送多少数据。我写了一个测试客户端可以连接到测试服务器，并且重复的调用 `socket.send()` 方法。测试服务端永远不调用 `socket.recv()` 或者 `socket.send()` 方法来处理客户端发送的数据，它只接受连接请求。这会导致服务器上的网络缓冲区被填满，最终会在客户端上报错

首先运行服务端：

```bash
$ ./app-server-test.py 127.0.0.1 65432 listening on ('127.0.0.1', 65432)
```

然后运行客户端，看看发生了什么：

```bash
$ ./app-client-test.py 127.0.0.1 65432 binary test
error: socket.send() blocking io exception for ('127.0.0.1', 65432):
BlockingIOError(35, 'Resource temporarily unavailable')
```

下面是用 `netstat` 命令在错误发生时执行的结果：

```bash
$ netstat -an | grep 65432
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4  408300      0  127.0.0.1.65432        127.0.0.1.53225        ESTABLISHED
tcp4       0 269868  127.0.0.1.53225        127.0.0.1.65432        ESTABLISHED
tcp4       0      0  127.0.0.1.65432        *.*                    LISTEN
```

第一行就表示服务端（本地端口是 65432）

```bash
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4  408300      0  127.0.0.1.65432        127.0.0.1.53225        ESTABLISHED
```

注意 Recv-Q: 408300

第二行表示客户端（远程端口是 65432）

```python
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0 269868  127.0.0.1.53225        127.0.0.1.65432        ESTABLISHED
```

注意 Send-Q: 269868

显然，客户端试着写入字节，但是服务端并没有读取他们。这导致服务端网络缓冲队列中应该保存的数据被积压在接收端，客户端的网络缓冲队列积压到发送端

## windows

如果你使用的是 windows 电脑，有一个工具套件绝对值得安装 [Windows Sysinternals](https://docs.microsoft.com/en-us/sysinternals/)

里面有个工具叫 `TCPView.exe`，它是 windows 下的一个可视化的 netstat 工具。除了地址、端口号和 socket 状态之外，它还会显示发送和接收的数据包以及字节数。就像 Unix 工具集 `lsof` 命令一样，你也可以看见进程名和 ID，可以在菜单中查看更多选项

![TCPView](https://files.realpython.com/media/tcpview.53c115c8b061.png)

## Wireshark

有时候你可能想查看网络底层发生了什么，忽略应用程序的输出或者外部库调用，想看看网络层面到底收发了什么内容，就像调试器一样，当你需要看清这些的时候，没有别的办法

[Wireshark](https://www.wireshark.org/) 是一款可以运行在 macOS, Linux, Windows 以及其它系统上的网络协议分析、流量捕获工具，GUI 版本的程序叫做
`wireshark`，命令 行的程序叫做 `tshark`

流量捕获是一个非常好用的方法，它可以让你看到网络上应用程序的行为，收集到关于收发消息多少、频率等信息，你也可以看到客户端或者服务端如何关闭/取消连接，或者停止响应，当你需要排除故障的时候这些信息非常的有用

网上还有很多关于 `wireshark` 和 `TShark` 的基础使用教程

这有一个使用 wireshark 捕获本地网络数据的例子：

![wireshark](https://files.realpython.com/media/wireshark.529c058891dc.png)

还有一个和上面一样的使用 tshark 命令输出的结果：

```bash
$ tshark -i lo0 'tcp port 65432'
Capturing on 'Loopback'
    1   0.000000    127.0.0.1 → 127.0.0.1    TCP 68 53942 → 65432 [SYN] Seq=0 Win=65535 Len=0 MSS=16344 WS=32 TSval=940533635 TSecr=0 SACK_PERM=1
    2   0.000057    127.0.0.1 → 127.0.0.1    TCP 68 65432 → 53942 [SYN, ACK] Seq=0 Ack=1 Win=65535 Len=0 MSS=16344 WS=32 TSval=940533635 TSecr=940533635 SACK_PERM=1
    3   0.000068    127.0.0.1 → 127.0.0.1    TCP 56 53942 → 65432 [ACK] Seq=1 Ack=1 Win=408288 Len=0 TSval=940533635 TSecr=940533635
    4   0.000075    127.0.0.1 → 127.0.0.1    TCP 56 [TCP Window Update] 65432 → 53942 [ACK] Seq=1 Ack=1 Win=408288 Len=0 TSval=940533635 TSecr=940533635
    5   0.000216    127.0.0.1 → 127.0.0.1    TCP 202 53942 → 65432 [PSH, ACK] Seq=1 Ack=1 Win=408288 Len=146 TSval=940533635 TSecr=940533635
    6   0.000234    127.0.0.1 → 127.0.0.1    TCP 56 65432 → 53942 [ACK] Seq=1 Ack=147 Win=408128 Len=0 TSval=940533635 TSecr=940533635
    7   0.000627    127.0.0.1 → 127.0.0.1    TCP 204 65432 → 53942 [PSH, ACK] Seq=1 Ack=147 Win=408128 Len=148 TSval=940533635 TSecr=940533635
    8   0.000649    127.0.0.1 → 127.0.0.1    TCP 56 53942 → 65432 [ACK] Seq=147 Ack=149 Win=408128 Len=0 TSval=940533635 TSecr=940533635
    9   0.000668    127.0.0.1 → 127.0.0.1    TCP 56 65432 → 53942 [FIN, ACK] Seq=149 Ack=147 Win=408128 Len=0 TSval=940533635 TSecr=940533635
   10   0.000682    127.0.0.1 → 127.0.0.1    TCP 56 53942 → 65432 [ACK] Seq=147 Ack=150 Win=408128 Len=0 TSval=940533635 TSecr=940533635
   11   0.000687    127.0.0.1 → 127.0.0.1    TCP 56 [TCP Dup ACK 6#1] 65432 → 53942 [ACK] Seq=150 Ack=147 Win=408128 Len=0 TSval=940533635 TSecr=940533635
   12   0.000848    127.0.0.1 → 127.0.0.1    TCP 56 53942 → 65432 [FIN, ACK] Seq=147 Ack=150 Win=408128 Len=0 TSval=940533635 TSecr=940533635
   13   0.001004    127.0.0.1 → 127.0.0.1    TCP 56 65432 → 53942 [ACK] Seq=150 Ack=148 Win=408128 Len=0 TSval=940533635 TSecr=940533635
^C13 packets captured
```
