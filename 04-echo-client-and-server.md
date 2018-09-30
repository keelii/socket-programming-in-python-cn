# 客户端/服务器打印程序

你现在已经了解了基本的 socket API 以及客户端和服务器是如何通信的，让我们来创建一个客户端和服务器。我们将会以一个简单的实现开始。服务器将打印客户端发送回来的内容

## 打印程序的服务端

下面就是服务器代码，`echo-server.py`：

```python
#!/usr/bin/env python3

import socket

HOST = '127.0.0.1'  # 标准的回环地址 (localhost)
PORT = 65432        # 监听的端口 (非系统级的端口: 大于 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024)
            if not data:
                break
            conn.sendall(data)
```

> 注意：上面的代码你可能还没法完全理解，但是不用担心。这几行代码做了很多事情，这只是一个起点，帮你看见这个简单的服务器是如何运行的教程后面有引用部分，里面有很多额外的引用资源链接，这个教程中我将把链接放在那儿

让我们一起来看一下 API 调用以及发生了什么

`socket.socket()` 创建了一个 socket 对象，并且支持 [上下文管理器](https://docs.python.org/3/reference/datamodel.html#context-managers)，你可以使用 [with 语句](https://docs.python.org/3/reference/compound_stmts.html#with)，这样你就不用再手动调用 `s.close()` 来关闭 socket 了

```python
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    pass  # Use the socket object without calling s.close().
```

调用 `socket()` 时传入的 socket 地址族参数 `socket.AF_INET` 表示因特网 IPv4 [地址族](https://realpython.com/python-sockets/#socket-address-families)，`SOCK_STREAM` 表示使用 TCP 的 socket 类型，协议将被用来在网络中传输消息


`bind()` 用来关联 socket 到指定的网络接口（IP 地址）和端口号：

```python
HOST = '127.0.0.1'
PORT = 65432

# ...

s.bind((HOST, PORT))
```

`bind()` 方法的入参取决于 socket 的地址族，在这个例子中我们使用了 `socket.AF_INET` (IPv4)，它将返回两个元素的元组：(host, port)

host 可以是主机名称、IP 地址、空字符串，如果使用 IP 地址，host 就应该是 IPv4 格式的字符串， `127.0.0.1` 是标准的 IPv4 回环地址，只有主机上的进程可以连接到服务器，如果你传了空字符串，服务器将接受本机所有可用的 IPv4 地址

端口号应该是 1-65535 之间的整数（0是保留的），这个整数就是用来接受客户端链接的 TCP 端口号，如果端口号小于 1024，有的操作系统会要求管理员权限

使用 `bind()` 传参为主机名称的时候需要注意：

> 如果你在 host 部分 **主机名称** 作为 IPv4/v6 socket 的地址，程序可能会产生非确定性的行为，因为 Python 会使用 DNS 解析后的 **第一个** 地址，根据 DNS 解析的结果或者 host 配置 socket 地址将会以不同方式解析为实际的 IPv4/v6 地址。如果想得到确定的结果传入的 host 参数建议使用数字格式的地址 [引用](https://docs.python.org/3/library/socket.html)

我稍后将在 [使用主机名](#使用主机名) 部分讨论这个问题，但是现在也值得一提。目前来说你只需要知道当使用主机名时，你将会因为 DNS 解析的原因得到不同的结果

可能是任何地址。比如第一次运行程序时是 10.1.2.3，第二次是 192.168.0.1，第三次是 172.16.7.8 等等

继续看上面的服务器代码示例，`listen()` 方法调用使服务器可以接受连接请求，这使它成为一个「监听中」的 socket

```python
s.listen()
conn, addr = s.accept()
```

`listen()` 方法有一个 `backlog` 参数。它指定在拒绝新的连接之前系统将允许使用的 _未接受的连接_ 数量。从 Python 3.5 开始，这是可选参数。如果不指定，Python 将取一个默认值

如果你的服务器需要同时接收很多连接请求，增加 backlog 参数的值可以加大等待链接请求队列的长度，最大长度取决于操作系统。比如在 Linux 下，参考 [/proc/sys/net/core/somaxconn](https://serverfault.com/questions/518862/will-increasing-net-core-somaxconn-make-a-difference/519152)

`accept()` 方法阻塞并等待传入连接。当一个客户端连接时，它将返回一个新的 socket 对象，对象中有表示当前连接的 conn 和一个由主机、端口号组成的 IPv4/v6 连接的元组，更多关于元组值的内容可以查看 [socket 地址族](#socket 地址族) 一节中的详情

这里必须要明白我们通过调用 `accept()` 方法拥有了一个新的 socket 对象。这非常重要，因为你将用这个 socket 对象和客户端进行通信。和监听一个 socket 不同的是后者只用来授受新的连接请求

```python
conn, addr = s.accept()
with conn:
    print('Connected by', addr)
    while True:
        data = conn.recv(1024)
        if not data:
            break
        conn.sendall(data)
```

从 `accept()` 获取客户端 socket 连接对象 conn 后，使用一个无限 while 循环来阻塞调用 `conn.recv()`，无论客户端传过来什么数据都会使用 `conn.sendall()` 打印出来

如果 `conn.recv()` 方法返回一个空 byte 对象（`b''`），然后客户端关闭连接，循环结束，with 语句和 conn 一起使用时，通信结束的时候会自动关闭 socket 链接

## 打印程序的客户端

现在我们来看下客户端的程序，` echo-client.py`：

```python
#!/usr/bin/env python3

import socket

HOST = '127.0.0.1'  # 服务器的主机名或者 IP 地址
PORT = 65432        # 服务器使用的端口

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    s.sendall(b'Hello, world')
    data = s.recv(1024)

print('Received', repr(data))
```

与服务器程序相比，客户端程序简单很多。它创建了一个 socket 对象，连接到服务器并且调用 `s.sendall()` 方法发送消息，然后再调用 `s.recv()` 方法读取服务器返回的内容并打印出来

## 运行打印程序的客户端和服务端

让我们运行打印程序的客户端和服务端，观察他们的表现，看看发生了什么事情

> 如果你在运行示例代码时遇到了问题，可以阅读 [如何使用 Python 开发命令行命令](https://dbader.org/blog/how-to-make-command-line-commands-with-python)，如果你使用的是 windows 操作系统，请查看 [Python Windows FAQ](https://docs.python.org/3.6/faq/windows.html)

打开命令行程序，进入你的代码所在的目录，运行打印程序的服务端：

```bash
$ ./echo-server.py
```

你的命令行将被挂起，因为程序有一个阻塞调用

```python
conn, addr = s.accept()
```

它将等待客户端的连接，现在再打开一个命令行窗口运行打印程序的客户端：

```bash
$ ./echo-client.py
Received b'Hello, world'
```

在服务端的窗口你将看见：

```bash
$ ./echo-server.py
Connected by ('127.0.0.1', 64623)
```

上面的输出中，服务端打印出了 `s.accept()` 返回的 addr 元组，这就是客户端的 IP 地址和 TCP 端口号。示例中的端口号是 64623 这很可能是和你机器上运行的结果不同

## 查看 socket 状态

想查找你主机上 socket 的当前状态，可以使用 `netstat` 命令。这个命令在 macOS, Window, Linux 系统上默认可用

下面这个就是启动服务后 netstat 命令的输出结果：

```bash
$ netstat -an
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0      0  127.0.0.1.65432        *.*                    LISTEN
```

注意本地地址是 127.0.0.1.65432，如果 `echo-server.py` 文件中 `HOST` 设置成空字符串 `''` 的话，netstat 命令将显示如下：

```bash
$ netstat -an
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0      0  *.65432                *.*                    LISTEN
```

本地地址是 `*.65432`，这表示所有主机支持的 IP 地址族都可以接受传入连接，在我们的例子里面调用 `socket()` 时传入的参数 `socket.AF_INET` 表示使用了 IPv4 的 TCP socket，你可以在输出结果中的 `Proto` 列中看到(tcp4)

上面的输出是我截取的只显示了咱们的打印程序服务端进程，你可能会看到更多输出，具体取决于你运行的系统。需要注意的是 Proto, Local Address 和 state 列。分别表示 TCP socket 类型、本地地址端口、当前状态

另外一个查看这些信息的方法是使用 `lsof` 命令，这个命令在 macOS 上是默认安装的，Linux 上需要你手动安装

```bash
$ lsof -i -n
COMMAND     PID   USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
Python    67982 nathan    3u  IPv4 0xecf272      0t0  TCP *:65432 (LISTEN)
```

isof 命令使用 `-i` 参数可以查看打开的 socket 连接的 COMMAND, PID(process id) 和 USER(user id)，上面的输出就是打印程序服务端

`netstat` 和 `isof` 命令有许多可用的参数，这取决于你使用的操作系统。可以使用 man page 来查看他们的使用文档，这些文档绝对值得花一点时间去了解，你将受益匪浅，macOS 和 Linux 中使用命令 `man netstat` 或者 `man lsof` 命令，windows 下使用 `netstat /?` 来查看帮助文档

一个通常会犯的错误是在没有监听 socket 端口的情况下尝试连接：

```bash
$ ./echo-client.py
Traceback (most recent call last):
  File "./echo-client.py", line 9, in <module>
    s.connect((HOST, PORT))
ConnectionRefusedError: [Errno 61] Connection refused
```

也可能是端口号出错、服务端没启动或者有防火墙阻止了连接，这些原因可能很难记住，或许你也会碰到 `Connection timed out` 的错误，记得给你的防火墙添加允许我们使用的端口规则

引用部分有一些常见的 [错误信息](10-reference.md#错误信息)

