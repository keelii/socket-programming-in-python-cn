# Socket API 概览

Python 的 socket 模块提供了使用 Berkeley sockets API 的接口。这将会在我们这个教程里使用和讨论到

主要的用到的 Socket API 函数和方法有下面这些：

* `socket()`
* `bind()`
* `listen()`
* `accept()`
* `connect()`
* `connect_ex()`
* `send()`
* `recv()`
* `close()`

Python 提供了和 C 语言一致且方便的 API。我们将在下面一节中用到它们

作为标准库的一部分，Python 也有一些类可以让我们方便的调用这些底层 Socket 函数。尽管这个教程中并没有涉及这部分内容，你也可以通过 [socketserver 模块](https://docs.python.org/3/library/socketserver.html) 中找到文档。当然还有很多实现了高层网络协议（比如：HTTP, SMTP）的的模块。做为了解，可以查看这个链接中的内容 [Internet Protocols and Support](https://docs.python.org/3/library/internet.html)

