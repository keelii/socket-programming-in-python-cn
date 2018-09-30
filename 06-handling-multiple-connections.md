# 处理多个连接

打印程序的服务端肯定有它自己的一些局限。这个程序只能服务于一个客户端然后结束。打印程序的客户端也有它自己的局限，但是还有一个问题，如果客户端调用了下面的方法`s.recv()` 方法将返回 `b'Hello, world'` 中的一个字节 `b'H'`

```python
data = s.recv(1024)
```

1024 是缓冲区数据大小限制最大值参数 `bufsize`，并不是说 `recv()` 方法只返回 1024个字节的内容

`send()` 方法也是这个原理，它返回发送内容的字节数，结果可能小于传入的发送内容，你得处理这处情况，按需多次调用 `send()` 方法来发送完整的数据

> 应用程序负责检查是否已发送所有数据；如果仅传输了一些数据，则应用程序需要尝试传递剩余数据 [引用](https://docs.python.org/3/library/socket.html#socket.socket.send)

我们可以使用 `sendall()` 方法来回避这个过程

> 和 send() 方法不一样的是，`sendall()` 方法会一直发送字节，只到所有的数据传输完成或者中途出现错误。成功的话会返回 None [引用](https://docs.python.org/3/library/socket.html#socket.socket.sendall)

到目前为止，我们有两个问题：

* 如何同时处理多个连接请求
* 我们需要一直调用 `send()` 或者 `recv()` 直到所有数据传输完成

应该怎么做呢，有很多方式可以实现并发。最近，有一个非常流程的库叫做 [Asynchronous I/O](https://docs.python.org/3/library/asyncio.html) 可以实现，asyncio 库在 Python 3.4 后默认添加到了标准库里面。传统的方法是使用线程

并发的问题是很难做到正确，有许多细微之处需要考虑和防范。可能其中一个细节的问题都会导致整个程序崩溃

我说这些并不是想吓跑你或者让你远离学习和使用并发编程。如果你想让程序支持大规模使用，使用多处理器、多核是很有必要的。然而在这个教程中我们将使用比线程更传统的方法使得逻辑更容易推理。我们将使用一个非常古老的系统调用：`select()`

`select()` 允许你检查多个 socket 的 I/O 完成情况，所以你可以使用它来检测哪个 socket I/O 是就绪状态从而执行读取或写入操作，但是这是 Python，总会有更多其它的选择，我们将使用标准库中的 [selectors](https://docs.python.org/3/library/selectors.html) 模块，所以我们使用了最有效的实现，不用在意你使用的操作系统：

> 这个模块提供了高层且高效的 I/O 多路复用，基于原始的 `select` 模块构建，推荐用户使用这个模块，除非他们需要精确到操作系统层面的使用控制 [引用](https://docs.python.org/3/library/selectors.html)

尽管如此，使用 `select()` 也无法并发执行。这取决于您的工作负载，这种实现仍然会很快。这也取决于你的应用程序对连接所做的具体事情或者它需要支持的客户端数量

[asyncio](https://docs.python.org/3/library/asyncio.html) 使用单线程来处理多任务，使用事件循环来管理任务。通过使用 `select()`，我们可以创建自己的事件循环，更简单且同步化。当使用多线程时，即使要处理并发的情况，我们也不得不面临使用 CPython 或者 PyPy 中的「全局解析器锁 GIL」，这有效地限制了我们可以并行完成的工作量

说这些是为了解析为什么使用 `select()` 可能是个更好的选择，不要觉得你必须使用 asyncio、线程或最新的异步库。通常，在网络应用程序中，你的应用程序就是 I/O 绑定：它可以在本地网络上，网络另一端的端，磁盘上等待

如果你从客户端收到启动 CPU 绑定工作的请求，查看 [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html) 模块，它包含一个 ProcessPoolExecutor 类，用来异步执行进程池中的调用

如果你使用多进程，你的 Python 代码将被操作系统并行地在不同处理器或者核心上调度运行，并且没有全局解析器锁。你可以通过
Python 大会上的演讲 [John Reese - Thinking Outside the GIL with AsyncIO and Multiprocessing - PyCon 2018](https://www.youtube.com/watch?v=0kXaLh8Fz3k) 来了解更多的想法

在下一节中，我们将介绍解决这些问题的服务器和客户端的示例。他们使用 `select()` 来同时处理多连接请求，按需多次调用 `send()` 和 `recv()`
