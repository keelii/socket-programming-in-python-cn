# TCP Sockets

就如你马上要看到的，我们将使用 `socket.socket()` 创建一个类型为 `socket.SOCK_STREAM` 的 socket 对象，默认将使用 [Transmission Control Protocol(TCP) 协议][1]，这基本上就是你想使用的默认值

为什么应该使用 TCP 协议？

* **可靠的**：网络传输中丢失的数据包会被检测到并重新发送
* **有序传送**：数据按发送者写入的顺序被读取

相反，使用 `socket.SOCK_DGRAM` 创建的 [用户数据报协议(UDP)][2] Socket 是 **不可靠** 的，而且数据的读取写发送可以是 **无序的**

为什么这个很重要？网络总是会尽最大的努力去传输完整数据（往往不尽人意）。没法保证你的数据一定被送到目的地或者一定能接收到别人发送给你的数据

网络设备（比如：路由器、交换机）都有带宽限制，或者系统本身的极限。它们也有 CPU、内存、总线和接口包缓冲区，就像我们的客户端和服务器。TCP 消除了你对于丢包、乱序以及其它网络通信中通常出现的问题的顾虑

下面的示意图中，我们将看到 Socket API 的调用顺序和 TCP 的数据流：

![TCP Socket 流][image-1]

左边表示服务器，右边则是客户端

左上方开始，注意服务器创建「监听」Socket 的 API 调用：

* `socket()`
* `bind()`
* `listen()`
* `accept()`

「监听」Socket 做的事情就像它的名字一样。它会监听客户端的连接，当一个客户端连接进来的时候，服务器将调用 `accept()` 来「接受」或者「完成」此连接

客户端调用 `connect()` 方法来建立与服务器的链接，并开始三次握手。握手很重要是因为它保证了网络的通信的双方可以到达，也就是说客户端可以正常连接到服务器，反之亦然

上图中间部分往返部分表示客户端和服务器的数据交换过程，调用了 `send()` 和 `recv()`方法

下面部分，客户端和服务器调用 `close()` 方法来关闭各自的 socket

[1]:	https://en.wikipedia.org/wiki/Transmission_Control_Protocol
[2]:	https://en.wikipedia.org/wiki/User_Datagram_Protocol

[image-1]:	https://files.realpython.com/media/sockets-tcp-flow.1da426797e37.jpg