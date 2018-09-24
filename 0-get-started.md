# 开始

网络中的 Socket 和 Socket API 是用来跨网络传送消息的，它提供了 [进程间通信(IPC)](https://en.wikipedia.org/wiki/Inter-process_communication) 的一种形式。网络可以是逻辑的、本地的电脑网络，或者是可以物理连接到外网的网络，并且可以连接到其它网络。英特网就是一个明显的例子，就是那个你通过 ISP 连接到的网络

本篇教程有三个不同的迭代阶段，来展示如何使用 Python 构建一个 Socket 服务器和客户端

1. 我们将以一个简单的 Socket 服务器和客户端程序来开始本教程
2. 当你看完 API 了解例子是怎么运行起来以后，我们将会看到一个具有同时处理多个连接能力的例子的改进版
3. 最后，我们将会开发出一个更加完善且具有完整的自定义头信息和内容的 Socket 应用

教程结束后，你将学会如何使用 Python 中的 [socket 模块](https://docs.python.org/3/library/socket.html) 来写一个自己的客户端/服务器应用。以及向你展示如何在你的应用中使用自定义类在不同的端之间发送消息和数据

所有的例子程序都使用 Python 3.6 编写，你可以在 Github 上找到 [源代码](https://github.com/realpython/materials/tree/master/python-sockets-tutorial)

网络和 Socket 是个很大的话题。网上已经有了关于它们的字面解释，如果你还不是很了解 Socket 和网络。当你你读到那些解释的时候会感到不知所措，这是非常正常的。因为我也是这样过来的

尽管如此也不要气馁。 我已经为你写了这个教程。 就像学习 Python 一样，我们可以一次学习一点。用你的浏览器保存本页面到书签，以便你学习下一部分时能找到

让我们开始吧！
